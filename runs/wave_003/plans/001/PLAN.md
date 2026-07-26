# Pocket Aviary — v1 Implementation Plan

**Status:** phase-1 plan, ready for engineering execution
**Audience:** a frontier engineering team (~7 people) building v1 end to end
**Source:** `prd/` (product_brief, concepts, bird_engine, interactions, aviary_layout, accounts_sync, social_optional, accessibility_perf, non_goals)

This plan interprets the PRD into an executable build. It does not restate the spec. Where the PRD leaves a value uncalibrated ("a few minutes," "slow cadence"), this plan picks a number, shows the arithmetic behind it, and marks it as a calibration target with a test that pins it. Where the PRD's affective rules imply an engineering constraint that would otherwise be invisible in code review, this plan converts the rule into a mechanism (a lint, a type, a CI gate) rather than leaving it as documentation.

---

## Table of contents

1. [Scope](#1-scope)
2. [Architecture](#2-architecture)
3. [Data model](#3-data-model)
4. [API surface](#4-api-surface)
5. [Simulation engine design](#5-simulation-engine-design)
6. [Sync model](#6-sync-model)
7. [Frontend rendering pipeline](#7-frontend-rendering-pipeline)
8. [Audio pipeline](#8-audio-pipeline)
9. [Voice kernel — notebook, narration, captions](#9-voice-kernel--notebook-narration-captions)
10. [Accessibility surfaces](#10-accessibility-surfaces)
11. [Performance budgets and observability](#11-performance-budgets-and-observability)
12. [Privacy and security engineering](#12-privacy-and-security-engineering)
13. [Testing strategy](#13-testing-strategy)
14. [Rollout](#14-rollout)
15. [Risks](#15-risks)
16. [Ambiguities resolved — decisions log](#16-ambiguities-resolved--decisions-log)

---

## 1. Scope

### 1.1 In scope for v1

| Area | Included |
| --- | --- |
| Accounts | Single-user accounts, magic-link sign-in (15-min expiry, single use), per-device revocable sessions, email change with verification, JSON export, soft-delete 30d → hard delete |
| Aviary | One aviary per account, 2 starter birds, cap 7, three perch zones, local-time day/night, ambient weather, ambient micro-motion, top-bar fade |
| Bird engine | Personality vector (5 traits), monotonic drift, mood state machine with cross-session persistence, procedural call grammar, mood-shaped idle motion, bird-to-bird interaction, ~6-species pool, age-gated new-bird arrivals |
| Interactions | Return-greeting, listen-in, offer (seed / song fragment / still pool) with per-bird cooldown, settle with 5s undo, field notebook (read-only, sparse), presence accounting |
| Sync | Server-authoritative simulation tick, snapshot pull, append-only client event log, additive server-authored personality deltas |
| Social | Visit invitations (off by default, per-invite opt-in, 30-day expiry, revocable), read-only ambient visitor view, silent visit log, opt-in visit notification toggle (default off) |
| Accessibility | Screen-reader narration in naturalist prose, reduced-motion mode as a designed surface, call captions, WCAG AA on all user copy, full keyboard navigation |
| Performance | <2MB gz initial bundle, <500ms time-to-first-bird (mid-tier mobile / 4G), 60fps idle on a 5-year-old laptop, no memory growth over 30 min |
| Platforms | Last two major versions of Chrome, Safari, Firefox, Edge; phone through desktop viewports |

### 1.2 Out of scope for v1 — and structurally refused

The four non-goals are not just unbuilt; several are made *harder to build later* by choices in this plan, which is the point of refusing them at the architecture layer rather than the backlog layer.

| Non-goal | How this plan refuses it structurally |
| --- | --- |
| Native mobile app | No shared protocol abstraction layer, no versioned mobile-friendly schema, no push-token infrastructure. The wire protocol is tuned for a browser client with an inlined bootstrap snapshot. |
| Gamification | No counter primitives exist anywhere: the event log stores no per-account session counts usable as a streak, the notebook generator's `subject` field is type-constrained to `bird` or `aviary` (never `user`), and a CI lint bans a lexicon (`streak`, `achievement`, `badge`, `level`, `score`, `days visited`, `you've been`, `welcome back`) from every user-facing string catalog. §13.6 describes the anti-announcement test suite. |
| Tamagotchi mechanics | Drift deltas are clamped to `[0, Δmax]` at the type level (`NonNegativeDelta`), so a negative-drift code path fails to compile. There is no hunger, health, or happiness field in the schema — the absence is in the DDL, not in a policy doc. |
| Social-network surfaces | No cross-account read path exists in the sim service except the single scoped visitor projection. There is no table that aggregates anything across accounts, so a leaderboard has no substrate to read. Telemetry (§12.3) cannot join to sim data by network topology. |

Also explicitly out: payments, shared/team aviaries, multi-aviary accounts, customizable scenes, public discovery, push notifications, email notifications about the aviary. The only emails the system sends are: magic link, email-change verification, export-ready link, visit invitation, and (only if the host opted in) visit notification.

### 1.3 Definition of done

V1 ships when every item in §14.6 (launch gates) is green. Two of those gates are unusual and are called out here because they will otherwise be treated as soft: **the drift calibration gate** (§5.3) and **the recognizability gate** (§8.6). Neither is a subjective sign-off; both are measured procedures with numeric thresholds.

---

## 2. Architecture

### 2.1 The one boundary that matters: discrete state vs. continuous presentation

Every other architectural question in this product falls out of one line: **the server owns discrete state; the client owns continuous presentation.**

- **Server owns:** personality vectors, mood and mood-entry time, perch assignment, call *intents* (which bird calls, when, from which motif, with which seed), weather schedule, greeting selection, notebook entries, narration text, aviary phase (light level).
- **Client owns:** interpolation between perch positions, idle micro-motion phase, ambient leaves and feathers, parallax, audio synthesis from call intents, mixing, caption layout, the top-bar fade.

The test for which side a thing belongs on: *would two devices showing the same aviary at the same moment disagree if this were client-owned?* Bird perch: yes → server. The exact phase of a leaf drifting through frame: no, nobody can tell → client. This one rule resolves the render-pipeline boundary, the sync model, and the visitor-view design without further argument, and it is what lets a snapshot be kilobytes.

### 2.2 Services

Seven deployable units. Small on purpose; the interesting complexity is in the sim, not in the topology.

```
                      ┌──────────────── CDN edge (static + edge render) ────────────────┐
  browser  ───────────►  aviary-edge:  serves HTML with INLINED bootstrap snapshot       │
                      └────────────────────────────┬───────────────────────────────────┘
                                                   │
        ┌──────────────┬─────────────────┬─────────┴────────┬─────────────────┐
        ▼              ▼                 ▼                  ▼                 ▼
   auth-svc       aviary-api        events-api         visits-api        mailer-svc
  (magic link,   (GET snapshot,   (POST events,      (invites, visit    (transactional
   sessions)      notebook,        append-only)       snapshot proj.)     email only)
        │              │                 │                  │                 │
        └──────────────┴────────┬────────┴──────────────────┘                 │
                                ▼                                             │
                     ┌──────────────────────┐                                 │
                     │  PRIMARY DB (PG)     │◄───── sim-worker (the tick) ─────┘
                     │  accounts, birds,    │       ONLY writer of personality
                     │  vectors, events,    │
                     │  notebook, invites   │
                     └──────────────────────┘
                                ▲
                                │  Redis: snapshot cache, rate limits, tick leases

        ══════════ HARD NETWORK BOUNDARY (no route, no credential) ══════════

                     telemetry pipeline ──► metrics store ──► dashboards
                     (aggregate only; see §12.3)
```

- **aviary-edge** — the only thing on the hot path for first paint. Serves the app HTML with the bootstrap snapshot inlined as a `<script type="application/json">` payload, so the client needs zero round-trips before drawing the first bird. Runs at CDN edge PoPs; authenticates via the session cookie and calls `aviary-api`'s snapshot endpoint (regional, cached in Redis) with a hard 120ms timeout. On timeout it serves the HTML *without* an inlined snapshot; the client then renders the quiet field (§7.9) and pulls normally. It never blocks the document.
- **aviary-api** — snapshot reads, notebook pagination, account settings, bird rename, export requests. Read-heavy, cacheable.
- **events-api** — the single write path for clients. Append-only. Deliberately separated from `aviary-api` so it can be rate-limited, scaled, and (if it ever fails) degraded independently: if `events-api` is down, the aviary still renders and only drift accrual pauses, which is the correct failure mode for this product.
- **sim-worker** — the tick. The only process in the system with write access to `bird_state` and `personality_vectors`. Enforced by database role: `aviary-api` and `events-api` connect as roles with no UPDATE grant on those tables. The rule "clients never write personality state" is thereby enforced by Postgres, not by code review.
- **auth-svc**, **visits-api**, **mailer-svc** — conventional.

### 2.3 Client/server split for the client itself

The web client is a single-page app with three route-level chunks:

| Chunk | Contents | Loading |
| --- | --- | --- |
| `boot` | Canvas setup, bird renderer, snapshot decoder, interpolation, micro-motion. | Inline-critical, in the initial HTML response |
| `aviary` | Audio engine, narration, captions, listen-in, offer, settle, notebook | Prefetched immediately after first paint |
| `system` | Account settings, accessibility settings, invite flow, visit log, export, sign-in | Lazy, on navigation only |

`system` is the "matter-of-fact voice" chunk and `aviary` is the "naturalist voice" chunk. That is not a coincidence — the voice split (§9.1) is enforced per-chunk by the string-catalog lint, and the code-splitting boundary makes the lint's scope unambiguous.

### 2.4 Technology choices

| Concern | Choice | Why (and what was rejected) |
| --- | --- | --- |
| Client framework | Preact + signals for chrome; **no framework inside the canvas** | React DOM reconciliation has no role in a canvas scene. Preact keeps the top bar, notebook, and settings cheap (~4KB) against the 2MB budget. Rejected: React (bundle), Svelte (fine, but Preact's ecosystem fit for a11y primitives is better here). |
| Scene rendering | Canvas 2D with layered offscreen canvases | At 7 birds × ~8 animated sub-parts plus ornaments, Canvas 2D holds 60fps comfortably on 2019 integrated graphics and costs ~0 bundle. Rejected: WebGL/three.js (200KB+, GPU wake cost hurts battery on a calm product, and we need zero of its capabilities); DOM/CSS animation (compositor layer explosion, and cross-fade reduced-motion mode is far harder). |
| Audio | Raw WebAudio, hand-built graph | Tone.js is ~150KB for scheduling we can write in ~6KB, and its transport model fights our server-scheduled intents. |
| Server language | TypeScript (Node) for API/edge; **TypeScript for sim-worker too** | One language for the shared `voice-kernel`, `call-grammar`, and `drift` packages, which must produce byte-identical results on server and client (captions must match audio; narration must match scene). A second language would force duplicate implementations of exactly the code where divergence is most damaging. |
| Database | PostgreSQL 16, partitioned event table | Single-writer sim + strong ordering + row-level locking is exactly Postgres's shape. Rejected: Kafka for the event log (operational weight for a per-account-ordered stream that Postgres gives us with a sequence). |
| Cache / leases | Redis | Snapshot cache, tick leases, rate limits. |
| Deploy | Containers on a managed platform; edge functions for `aviary-edge` | — |

### 2.5 Deterministic replay is the load-bearing systems trick

The PRD requires the tick to run "whether or not any client is connected" at ~once per minute. Taken literally at scale, one million accounts is ~16,700 ticks/second forever, the overwhelming majority computing state nobody will look at.

**Decision:** the tick is *logically* continuous at 60s and *physically* tiered, with exact catch-up. Every tick's output is a pure function of `(prior_state, events_in_window, wall_clock_tick_index, seeded_prng(bird_id, tick_index))`. Nothing in the tick reads `now()` directly, samples unseeded randomness, or depends on when it executed. Weather is pre-scheduled per aviary-day from a seed (§5.6), so it too replays exactly.

Given that purity, a cold account's state can be fast-forwarded on demand and the result is **bit-identical** to what a continuously-running tick would have produced. The user's claim — "the aviary the user comes back to is the aviary that has been running" — is therefore true in the strong sense, not approximated.

Tiers:

| Tier | Condition | Physical cadence |
| --- | --- | --- |
| Warm | A client pulled a snapshot in the last 10 min | Every 60s |
| Recent | Presence within 24h | Every 15 min (batched, replaying the intervening minute-ticks) |
| Cold | No presence in 24h | Lazy: replayed on next snapshot request, plus a nightly sweep to keep notebook entries and drift epochs current |

A cold-account catch-up of 30 days is 43,200 minute-ticks. Measured target: <200ms, achieved by (a) collapsing the mood evaluation to epoch boundaries when no events exist in a window — mood transitions in an empty window depend only on circadian/weather curves and can be solved analytically per segment rather than iterated — and (b) applying drift once per drift-epoch (a local day), not per minute. A hard cap: any catch-up exceeding 2s is completed asynchronously while the request returns the last-known state with a `stale: true` flag; the client renders it normally (the aviary is never a load state) and picks up the caught-up state on its next pull 30s later.

**Cost check.** At 1M accounts with 5% warm at any moment: 50k warm × 1/min = 833 ticks/s; recent tier ~200k × 1/15min = 222 batched runs/s. A tick for a 7-bird aviary is ~0.4ms of CPU. Total ≈ 0.5 vCPU-seconds/s for warm plus batching overhead — comfortably a handful of workers. Naive uniform ticking would have been ~40× that.

---

## 3. Data model

### 3.1 Core tables

```sql
-- Identity. Email appears in exactly one column in the entire system.
CREATE TABLE accounts (
  id                  uuid PRIMARY KEY DEFAULT gen_random_uuid(),  -- synthetic; the ONLY identifier used anywhere else
  email_ciphertext    bytea NOT NULL,          -- envelope-encrypted (KMS data key), see §12.1
  email_lookup_hash   bytea NOT NULL UNIQUE,   -- HMAC-SHA256(email, pepper) for sign-in lookup; not reversible
  pending_email_ct    bytea,                   -- email change awaiting verification
  timezone            text NOT NULL DEFAULT 'UTC',  -- IANA; client-provided hint, last-write-wins (see §6.5)
  created_at          timestamptz NOT NULL DEFAULT now(),
  aviary_seed         bigint NOT NULL,         -- deterministic seed for species selection, weather, PRNG streams
  max_birds           smallint NOT NULL DEFAULT 2,  -- raised by age-gate; hard ceiling 7 enforced by CHECK
  settings            jsonb NOT NULL DEFAULT '{}'::jsonb,  -- reduced_motion, captions, audio, visit_notify
  deletion_requested_at timestamptz,
  hard_delete_after   timestamptz,
  CHECK (max_birds BETWEEN 2 AND 7)
);

CREATE TABLE birds (
  id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),  -- STABLE FOREVER. never reissued. see §3.4
  account_id    uuid NOT NULL REFERENCES accounts(id),
  species       text NOT NULL,          -- references the species pool (code-side registry)
  name          text NOT NULL,          -- user-assigned; renameable; NEVER an identifier
  voice_seed    bigint NOT NULL,        -- fixes this bird's timbre/pitch-centre invariants forever (§8.3)
  adopted_at    timestamptz NOT NULL DEFAULT now(),
  retired_at    timestamptz             -- reserved; unused in v1. birds do not die.
);

-- Personality: the canonical, stored, server-only-written vector.
CREATE TABLE personality_vectors (
  bird_id       uuid PRIMARY KEY REFERENCES birds(id),
  boldness      real NOT NULL, social_warmth real NOT NULL, vocal_frequency real NOT NULL,
  plumage       real NOT NULL, curiosity     real NOT NULL,
  updated_at    timestamptz NOT NULL,
  epoch_hi      integer NOT NULL,   -- last drift-epoch applied; makes epoch application idempotent
  CHECK (boldness BETWEEN 0 AND 1 AND social_warmth BETWEEN 0 AND 1
         AND vocal_frequency BETWEEN 0 AND 1 AND plumage BETWEEN 0 AND 1 AND curiosity BETWEEN 0 AND 1)
);

-- Append-only audit/recovery ledger. NOT read at runtime (see §3.3).
CREATE TABLE drift_deltas (
  bird_id   uuid NOT NULL REFERENCES birds(id),
  epoch     integer NOT NULL,          -- local-day index
  d_bold real NOT NULL, d_warm real NOT NULL, d_vocal real NOT NULL, d_plum real NOT NULL, d_cur real NOT NULL,
  inputs    jsonb NOT NULL,            -- the signal values that produced this delta, for calibration forensics
  applied_at timestamptz NOT NULL DEFAULT now(),
  PRIMARY KEY (bird_id, epoch),
  CHECK (d_bold >= 0 AND d_warm >= 0 AND d_vocal >= 0 AND d_plum >= 0 AND d_cur >= 0)  -- monotonicity, in the DB
);

-- Fast-timescale state, rewritten by the tick.
CREATE TABLE bird_state (
  bird_id        uuid PRIMARY KEY REFERENCES birds(id),
  mood           text NOT NULL,        -- wary|alert|curious|content|drowsy|settled
  mood_since     timestamptz NOT NULL, -- persists across sessions; never reset on tab open
  perch_zone     smallint NOT NULL,    -- 0 front, 1 middle, 2 back
  perch_slot     smallint NOT NULL,    -- position within the zone
  next_call_at   timestamptz,
  last_greeted_at timestamptz,
  offer_cooldown_until timestamptz,
  gesture        text NOT NULL,        -- current idle gesture, so a returning client resumes mid-action
  gesture_phase  real NOT NULL
);

-- The append-only client write path.
CREATE TABLE interaction_events (
  account_id  uuid NOT NULL,
  seq         bigint NOT NULL,          -- server-assigned, per-account monotonic
  client_id   text NOT NULL,            -- idempotency key from the client
  type        text NOT NULL,            -- presence|listen_in_start|listen_in_end|offer|settle|session_open|session_close
  bird_id     uuid,
  occurred_at timestamptz NOT NULL,     -- CLAMPED server-side, see §6.3
  received_at timestamptz NOT NULL DEFAULT now(),
  payload     jsonb NOT NULL,
  consumed_by_tick bigint,              -- tick id that consumed this row
  PRIMARY KEY (account_id, seq),
  UNIQUE (account_id, client_id)
) PARTITION BY RANGE (received_at);     -- monthly partitions; retention §12.4

CREATE TABLE notebook_entries (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  account_id  uuid NOT NULL REFERENCES accounts(id),
  observed_at timestamptz NOT NULL,
  subject_kind text NOT NULL CHECK (subject_kind IN ('bird','aviary')),  -- never 'user'. see §1.2
  subject_bird uuid REFERENCES birds(id),
  generator   text NOT NULL,            -- detector id, for debugging and sparsity accounting
  gen_seed    bigint NOT NULL,
  prose       text NOT NULL             -- the realized text, stored (never regenerated — voice must be stable)
);

CREATE TABLE sessions (
  id uuid PRIMARY KEY, account_id uuid NOT NULL REFERENCES accounts(id),
  token_hash bytea NOT NULL UNIQUE, created_at timestamptz NOT NULL DEFAULT now(),
  last_seen_at timestamptz NOT NULL, user_agent_class text,   -- "Chrome on macOS", not a raw UA string
  revoked_at timestamptz
);

CREATE TABLE magic_links (
  token_hash bytea PRIMARY KEY, account_id uuid NOT NULL, purpose text NOT NULL,  -- signin|email_change
  expires_at timestamptz NOT NULL, consumed_at timestamptz
);

CREATE TABLE invites (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  host_account_id uuid NOT NULL REFERENCES accounts(id),
  visitor_email_ct bytea NOT NULL,        -- encrypted, same discipline as account email
  token_hash bytea NOT NULL UNIQUE,
  created_at timestamptz NOT NULL DEFAULT now(),
  expires_at timestamptz NOT NULL,        -- created_at + 30 days
  revoked_at timestamptz, first_used_at timestamptz
);

CREATE TABLE visit_sessions (
  id uuid PRIMARY KEY, invite_id uuid NOT NULL REFERENCES invites(id),
  started_at timestamptz NOT NULL, last_pull_at timestamptz NOT NULL   -- duration is approximate, by design
);
```

### 3.2 What is deliberately absent from the schema

No `health`, `hunger`, `happiness`, `energy`, `xp`, `level`, `visit_count`, `streak_days`, `last_visit_date_series`, `achievements`. No cross-account aggregate table of any kind. These absences are load-bearing: a future contributor who wants a streak counter has to add a migration, and a migration adding a per-user counter is a reviewable event in a way that a feature flag is not.

### 3.3 Personality: stored canonical, ledger for recovery

The PRD is explicit: the personality vector "is never derived from session history at runtime, never recomputed from event logs." This plan honors that literally — **runtime reads always read `personality_vectors`**, a single stored row, and no code path reconstructs a vector from events.

`drift_deltas` exists anyway, written in the same transaction as the vector update, because the PRD also names vector loss as the worst possible failure. The ledger is an append-only audit and disaster-recovery artifact:

- **Forensics** — when calibration looks wrong, the ledger tells us which day's signals produced which delta.
- **Recovery** — if a bad deploy corrupts vectors, the ledger plus the last good snapshot restores them exactly, without inventing values.
- **It is not a runtime dependency.** A CI architecture test asserts that no module in the request path imports the `drift_deltas` repository.

This distinction (canonical stored value vs. recovery ledger) is the defensible reading of a rule that would otherwise leave us with no way to un-corrupt the one piece of data the product cannot afford to lose.

### 3.4 Identity continuity

`birds.id` is generated once and never changes. Concretely, three rules with tests:

1. No code path deletes a `birds` row. Deletion happens only via account hard-delete, which removes the account entirely.
2. Species-pool changes (new species, retuned motifs) are additive; a species is never removed, and a bird's `species` value is never rewritten by a migration. A migration test asserts `species` is not in the SET clause of any migration.
3. `voice_seed` is fixed at adoption and never rewritten, so a bird's *identity by ear* (§8.3) is as stable as its identity by id.

The word "reset," "regenerate," or "swap" never appears in an operational runbook for birds. If a bird's state is ever unrecoverable, the incident procedure is to restore from PITR, not to reseed.

---

## 4. API surface

All endpoints under `/v1`. Auth via `HttpOnly; Secure; SameSite=Lax` session cookie. JSON everywhere. Errors use the matter-of-fact voice catalog (§9.1) and carry a stable `code` for the client to map.

### 4.1 Read: the snapshot

```
GET /v1/aviary/snapshot?since=<version>
→ 200, ETag, Cache-Control: private, no-store
```

```json
{
  "v": 184213,
  "server_time": 1785500000000,
  "window_ms": 90000,
  "next_pull_after_ms": 30000,
  "aviary": {
    "tz": "America/New_York",
    "light": { "level": 0.62, "warmth": 0.31, "phase": "morning",
               "curve": [[0,0.62],[900000,0.68],[1800000,0.73]] },
    "weather": { "kind": "none" },
    "weather_next": { "kind": "rain", "at": 1785503600000, "duration_ms": 420000, "intensity": 0.35 }
  },
  "birds": [
    {
      "id": "b_7f3c1e...", "name": "pip", "species": "warbler",
      "perch": { "zone": 0, "slot": 2, "since": 1785499880000 },
      "mood": "curious",
      "render": { "palette": "warbler.04", "feather_detail": 3, "scale": 1.0 },
      "motion": { "gesture": "preen", "phase": 0.37, "rate": 0.92, "seed": 55123 },
      "calls": [
        { "at": 1785500002400, "motif": "m3", "seed": 918273, "intensity": 0.8 },
        { "at": 1785500041900, "motif": "m1", "seed": 331902, "intensity": 0.6 }
      ]
    }
  ],
  "greeting": { "bird": "b_7f3c1e...", "form": "look_and_call", "at": 1785500001200, "seed": 44771 },
  "narration": { "text": "a small grey bird is on the front rail, calling softly. …", "at": 1785500000000 },
  "stale": false
}
```

Four things about this payload are deliberate:

- **`render` carries a palette identifier, not a plumage number.** No endpoint anywhere returns a personality trait value. The wire protocol carries *behavior* (which perch, which gesture, which palette bucket), never traits. This is what makes "the user never sees the numbers" true even for a user with devtools open — there is nothing to read. Plumage is quantized to 6 palette steps server-side; boldness, warmth, vocal frequency, and curiosity never leave the server in any form.
- **`window_ms` is a forward window.** The snapshot includes scheduled call intents and a light curve for the next 90 seconds. The client therefore always has material to render and never stalls waiting for a poll. This is also what makes the "already in motion" first frame trivially true: the bootstrap snapshot arrives with the bird mid-preen at a known phase and a call already scheduled 2.4 seconds out.
- **`greeting` is precomputed at snapshot time**, so the return-greeting fires within 1–2s with zero additional round trips (§5.7).
- **`since` is an optimization only.** If the client passes a version, the server may return `204 Not Modified`. It never returns a partial diff — snapshots are small and full-state replacement removes an entire class of merge bugs.

### 4.2 Write: events

```
POST /v1/aviary/events
```

```json
{ "events": [
  { "cid":"e_9f2a1", "type":"presence", "t":1785499940000,
    "payload": { "dur_ms":15000, "vis":true, "focus":true, "act_age_ms":42000 } },
  { "cid":"e_9f2a2", "type":"listen_in_start", "t":1785499951000, "bird":"b_7f3c1e..." },
  { "cid":"e_9f2a3", "type":"offer", "t":1785499990000, "bird":"b_7f3c1e...",
    "payload": { "item":"seed", "outcome":"approached" } }
]}
→ 202 { "accepted": 3, "duplicates": 0, "seq_hi": 448120 }
```

- Idempotent on `(account_id, cid)`; duplicates are counted and dropped.
- `t` is clamped server-side to `[now − 300s, now + 5s]`.
- Batched: the client flushes every 15s, on `visibilitychange`, and via `sendBeacon` on `pagehide`.
- `offer.outcome` is client-observed *presentation* of a server-decided reaction; the sim recomputes the authoritative outcome from bird state and ignores the client's claim for drift purposes. The field exists only for notebook detectors that want "what the user saw."

### 4.3 Remaining endpoints

| Method + path | Purpose | Notes |
| --- | --- | --- |
| `POST /v1/auth/magic-link` | Request sign-in link | Always returns 202 regardless of account existence (no account enumeration). Rate limit: 5/hour/email, 20/hour/IP. |
| `GET /auth/callback?t=…` | Consume link | 15-min expiry, single use, constant-time hash compare, sets session cookie, 302 to `/` |
| `GET /v1/auth/sessions` · `DELETE /v1/auth/sessions/:id` | List/revoke devices | Device shown as a UA *class*, never a raw UA string |
| `POST /v1/auth/signout` | Revoke current session | |
| `GET /v1/notebook?before=<cursor>&limit=30` | Paginated entries, newest first | Read-only; no PATCH/DELETE exist |
| `PATCH /v1/birds/:id` | `{ "name": "pippa" }` | Name only. Trait fields are not in the request schema. |
| `PATCH /v1/account/settings` | reduced_motion, captions, audio_enabled, visit_notify, timezone | |
| `POST /v1/account/export` | Queue export | Generated async, emailed as a signed 24h link |
| `POST /v1/account/delete` · `POST /v1/account/undelete` | Soft delete / recover | Undelete available on any signed-in page for 30 days |
| `POST /v1/invites` `{email}` · `GET /v1/invites` · `DELETE /v1/invites/:id` | Visit invitations | Max 20 outstanding; 30-day expiry |
| `GET /v1/visits/log` | Host's visit log | Pull-only; no badge, no count surfaced elsewhere |
| `GET /visit/:token` | Visitor entry point | Sets a scoped visitor cookie; no account created |
| `GET /v1/visit/snapshot` | Visitor projection | §4.4 |

### 4.4 The visit flow

1. Host `POST /v1/invites {email}` → row created with 256-bit token (stored hashed), 30-day expiry; `mailer-svc` sends the link. No confirmation toast in the aviary; the invite appears in the settings list, which is where the host already is.
2. Visitor opens `/visit/:token`. `visits-api` validates (not expired, not revoked, not used by a different browser beyond a reasonable session), creates a `visit_sessions` row, sets a scoped cookie bound to that invite, and serves the aviary shell.
3. Visitor polls `GET /v1/visit/snapshot`, which runs the **same snapshot generator** with a visitor projection:
   - **Included:** all bird state, calls, perches, moods-as-rendered, weather, day/night, narration (accessibility parity for visitors is not optional — §10.1).
   - **Excluded:** `greeting` (the visitor's arrival must not be noticed by the birds — the greeting belongs to the host), notebook, account data, any invite/visit metadata.
   - **No special rendering.** The visitor projection is derived by *deletion* from the host snapshot, never by an alternate code path, which structurally forecloses "show-off mode."
4. All writes from a visitor session return `403 visit_read_only`. The visitor's client does not ship listen-in, offer, or settle handlers at all — the `aviary` chunk reads a `readOnly` capability flag from bootstrap and mounts a reduced control set (top bar shows accessibility settings only).
5. **Visitor activity is never written to `interaction_events`.** `events-api` rejects visitor cookies at the edge of the handler, before any parsing. `visit_sessions.last_pull_at` is updated by `visits-api` in a different table for the visit log; it is never joined to sim inputs. A test asserts that a 60-minute visitor session produces zero change in the host's drift ledger.
6. Revocation: `DELETE /v1/invites/:id` sets `revoked_at`; the visitor's next poll (≤30s) returns `410` with `code: visit_unavailable`, and the client renders the matter-of-fact surface. Unused revoked links simply stop working with the same surface. No confirmation is shown to the host.

---

## 5. Simulation engine design

The sim is a pure function library (`packages/sim`) plus a thin worker that loads state, applies it, and persists. Purity is a hard rule (§2.5): the tick takes `(state, events, tickIndex, prngStreams)` and returns `(nextState, writes)`.

### 5.1 The tick

```
tick(accountId, tickIndex):
  lease  = redis.acquire("tick:"+accountId, ttl=30s)      # single-writer per account
  state  = load(account, birds, vectors, bird_state)
  events = fetch events with consumed_by_tick IS NULL, ORDER BY seq
  ctx    = { localTime(tz, tickIndex), weather(seed, day, tickIndex), prng(birdId, tickIndex) }

  for each bird:
     accumulateDriftSignals(bird, events, ctx)      # into per-epoch accumulator, not applied yet
     mood      = evaluateMood(bird, events, ctx, neighbors)
     perch     = choosePerch(bird, mood, ctx)
     calls     = scheduleCalls(bird, mood, ctx, window=90s)
     gesture   = chooseGesture(bird, mood, ctx)

  if crossedDriftEpochBoundary(ctx):  applyDrift(...)     # once per local day, idempotent on epoch_hi
  notebook  = runObservationDetectors(state, nextState, ctx)   # sparsity governor, §9.3
  narration = composeNarration(nextState, ctx)                  # §9.2
  persist(nextState, notebook, narration, mark events consumed)
  invalidate snapshot cache
```

Ordering guarantees: events are consumed in `seq` order; a tick either commits entirely or not at all (single transaction); `consumed_by_tick` makes reprocessing impossible; `personality_vectors.epoch_hi` makes drift application idempotent even if a tick is retried after a partial failure.

### 5.2 Drift: the exact function

Traits are `[0,1]`. Seeds at adoption: species prior mean ± jitter, clamped to `[0.20, 0.55]`, leaving deliberate headroom for years of monotonic upward drift.

**Per-epoch signals.** A drift epoch is one local day, boundary at 04:00 local (chosen so a late-night session belongs to the day it started).

```
S_presence   = min(1, presence_minutes_today / 20)
S_listen[i]  = min(1, listen_seconds_on_bird_i / 300)
S_offerNear  = min(1, offers_made_while_bird_i_within_one_zone / 3)
S_offerTake  = min(1, offers_bird_i_investigated / 3)
```

**Per-trait daily signal:**

| Trait | Signal |
| --- | --- |
| plumage | `S_presence` |
| social_warmth | `0.5·S_presence + 0.5·S_listen[i]` |
| vocal_frequency | `0.4·S_presence + 0.6·S_listen[i]` |
| boldness | `0.6·S_presence + 0.4·S_offerNear` |
| curiosity | `0.5·S_presence + 0.5·S_offerTake` |

Settle contributes nothing to drift — it closes the presence window cleanly and nudges mood, exactly as specified.

**The update** — a first-order low-pass approaching the ceiling, clamped non-negative:

```
Δt = clamp( α · (1 − t) · S , 0 , Δ_max )        α = 0.014 / epoch,  Δ_max = 0.02
t' = t + Δt
```

`(1 − t)` gives diminishing returns near the top, so a bird retains reserve indefinitely and no user "maxes out" a trait. `Δ_max` guarantees the single-session rule: even a 12-hour marathon moves any trait by at most 0.02.

**Calibration arithmetic.** Reference user: 15 min presence/day, 5 days/week, ~2 min listen-in on one bird, one offer investigated. Starting `t ≈ 0.35`.

```
plumage Δ/day = 0.014 × (1 − 0.35) × 0.75 = 0.00683
  7 days  (5 active): 0.0341     ← instrument-measurable, well above the 0.01 threshold
 21 days (15 active): 0.0985     ← crosses the 0.09 perceptual threshold
  1 session max:      0.0200     ← below perceptual threshold. no session moves a trait visibly.
```

The instruments-vs-user gap the PRD asks for falls out of the *behavior mapping*, not out of the drift rate. Behavior is a smooth function of traits (`P(front perch) = σ(k·(boldness − 0.5))`, `E[calls/hour] = base·(0.6 + 0.8·vocal)`, plumage → 6 quantized palette steps ≈ 0.09 apart in trait space). At day 7 a bird's front-perch probability has shifted ~3 percentage points — real in a histogram, invisible in a session. At day 21 it has shifted ~9 points and crossed a palette step, which is what "you notice when you look back" feels like.

**Named thresholds, as test constants:**

```
DRIFT_INSTRUMENT_THRESHOLD = 0.010   # must be exceeded by day 7 for the reference profile
DRIFT_PERCEPTUAL_THRESHOLD = 0.090   # must be exceeded by day 21; must NOT be by day 7
DRIFT_SESSION_CEILING      = 0.020   # must never be exceeded in one day, any input
```

**The drift observatory.** A permanent CI job runs four scripted profiles through the time-warp harness (§13.3) for 90 simulated days: `absent` (0 presence), `sparse` (2 days/wk × 5 min), `reference`, `heavy` (60 min/day, daily). It asserts the three thresholds, asserts `absent` produces exactly zero change in every trait, and stores the trajectory curves as artifacts. A code change that moves the 21-day reference trajectory by >10% fails the build and requires an explicit calibration review. This uses synthetic profiles only — never production data (§12.3).

### 5.3 Monotonicity, enforced three ways

1. **Type:** `applyDrift` accepts `NonNegativeDelta`, a branded type constructible only via `clampNonNegative`.
2. **Database:** `drift_deltas` has `CHECK (d_* >= 0)`.
3. **Property test:** for 10,000 randomly generated event sequences (including hostile ones — long absences, weather storms, rapid offer spam, clock skew), assert every trait is non-decreasing across every tick.

The behavioral consequence — "a neglected bird becomes ambient, not distressed" — is implemented in the *mood and scheduling* layer, not by lowering traits: with no recent presence, `S = 0`, traits hold, and the mood evaluator's `recent_interaction` term decays to zero, which raises the relative weight of circadian and ambient terms. The bird calls at its natural (unchanged) `vocal_frequency` rate but greets less often, because greeting selection is gated on recency of observation, not on a trait. That is exactly "quieter than they were, not sick."

### 5.4 Mood

Six states: `wary`, `alert`, `curious`, `content`, `drowsy`, `settled`.

Each tick, score every candidate mood and switch only under hysteresis:

```
score(m) =  w_circ · circadian(m, localTime)
          + w_evt  · recentEvents(m, events, τ = 8 min exponential decay)
          + w_wx   · weather(m, currentWeather)
          + w_soc  · contagion(m, neighborMoods, zoneAdjacency)
          + w_pers · personalityBias(m, vector)
          + w_iner · (m == current ? 1 : 0)

weights: w_circ 1.0, w_evt 1.4, w_wx 0.6, w_soc 0.5, w_pers 0.8, w_iner 0.6

switch iff  argmax(score) ≠ current
       AND  score(argmax) − score(current) > 0.15
       AND  now − mood_since ≥ 90s
```

- **Circadian** — piecewise curves over local time: `alert` peaks 06:00–09:00; `content` broad midday; `drowsy` rises from 18:00; `settled` dominates after civil dusk (approximated from timezone offset and date — no geolocation, no lat/long; §16).
- **Recent events** — an accepted offer pushes `content` and `curious`; a listen-in pushes `curious` and `alert` on the focused bird; settle pushes `drowsy`; a sudden ambient event pushes `wary`.
- **Weather** — rain suppresses `alert` and boosts `drowsy` mildly; wind boosts `alert` for high-boldness birds and `wary` for low.
- **Contagion** — a `wary` neighbour in an adjacent zone adds to this bird's `wary` score, scaled by `(1 − boldness)`. Contagion reads the *previous* tick's neighbour moods (double-buffered), so mood propagation cannot oscillate within a tick and evaluation order is irrelevant.
- **Personality bias** — `wary` gets `−1.4·(boldness − 0.5)`; `curious` gets `+1.2·(curiosity − 0.5)`; `settled` is species-gated (the nightjar-like species stays `alert` at night).
- **Persistence** — `bird_state.mood` and `mood_since` are simply the stored state. There is no session-start mood recomputation, no "reset to neutral" code path, and a test asserts that opening a session produces zero mood writes.

Ties break with `prng(bird_id, tick_index)`, so replay is deterministic.

### 5.5 Perch, gesture, and bird-to-bird interaction

- **Perch choice** — evaluated at most every 8 ticks per bird (and on strong mood change), giving birds a settled quality rather than constant hopping. `P(front) = σ(3.2·(boldness − 0.5) + moodOffset(mood))` where `moodOffset` is `+0.6 curious`, `+0.3 content`, `−0.8 wary`, `−0.5 drowsy`, `−1.2 settled`. Slots within a zone are assigned to avoid collisions; a high-warmth bird biases toward a slot adjacent to another bird. **The user cannot influence perch directly** — there is no API field for it, which is how "perch is a signal, not a layout" is enforced.
- **Gesture** — Poisson scheduling with mood-dependent rates: `content` → preen often; `wary` → scan often, preen rarely; `curious` → head-tilt toward the most recent call source; `drowsy` → fluff and low-sit; `alert` → quick scans, weight-shuffle. The chosen gesture and its phase are in the snapshot, so a client that opens mid-preen renders mid-preen.
- **Bird-to-bird** — three mechanisms: (a) **call-and-response** — when bird A calls, each other bird B rolls `P = 0.15 + 0.5·warmth_B·vocal_B` to schedule a reply 0.9–2.6s later; (b) **contagion** as above; (c) **chorus emergence** — the scheduler nudges the next-call time of high-`vocal_frequency` birds toward alignment with a probability of `0.25·vocal_A·vocal_B`, so choruses happen naturally and often enough to be a felt property, without being scripted events.

### 5.6 Weather

Weather is a **pre-generated per-aviary-day schedule**, computed as `weatherSchedule(aviary_seed, local_date)`: a deterministic function returning zero to two events per day with kind (`rain` | `wind`), start time, duration (4–12 min), and intensity (0.2–0.5). Target frequency: rain ~3×/week, wind ~5×/week; never both at once; never at intensity that reads as a storm.

Determinism matters twice over: it makes cold-account replay exact (§2.5), and it lets the snapshot ship `weather_next` so the client can pre-warm the rain layer and cross-fade it in rather than popping it on.

### 5.7 Return-greeting

Triggered by a `session_open` event, or — for the common case — computed inline when `aviary-edge` builds the bootstrap snapshot, so it costs zero round trips.

**Absence** is computed server-side as `now − last_presence_end`, never trusted from the client.

**Greeter selection** — weighted sample (seeded by session id, so all of a user's tabs agree):

```
weight(bird) = boldness^1.5 · (0.4 + 0.6·social_warmth) · moodReadiness(mood) · recencyPenalty(last_greeted_at)
moodReadiness: alert 1.0, curious 0.95, content 0.8, drowsy 0.35, wary 0.25, settled 0.05
recencyPenalty: 0.35 if this bird greeted the last two sessions, else 1.0
```

The recency penalty is what makes the notebook's "pip greeted before wren today, first time this week" a true observation rather than a coincidence — greeter identity genuinely varies, and it varies in a way that tracks boldness and warmth drift over weeks.

**Form ladder by absence bucket:**

| Absence | Form | Content |
| --- | --- | --- |
| < 5 min | `glance` | Head lifts from current gesture, brief look toward viewer. No call. |
| 5 min – 2 h | `glance_call` | Look plus a short 1–2 phrase call |
| 2 h – 24 h | `look_and_call` | Look, a fuller call, possible one-zone step forward if boldness > 0.45 |
| > 24 h | `approach` | Longer call, movement to the front zone, and a likely second-bird response |

**Variation is real, not a rotation of variants.** The call is a fresh grammar realization from `seed` (§8.2) — a new transform stack every time, not one of three recordings. The motion is a fresh blend over gesture primitives with jittered timing. A test asserts that 200 consecutive greetings for the same bird produce 200 distinct `(motif, transform-stack, timing)` tuples, and that no realized call's parameter vector is within an epsilon of any of the previous 20.

**Staggering** — when a second bird responds, its offset is drawn uniformly from 0.8–3.5s with an enforced minimum 400ms separation from any other greeting event. Simultaneous greeting is unreachable by construction.

### 5.8 Offers

Three items: `seed`, `song` (a fragment from a library of ~12 short motifs), `pool`.

Reaction is server-decided at the next tick, but — critically for feel — the client must react *immediately*. Resolution: the offer's outcome is computable client-side from data the client already has (mood, plus a server-supplied `offer_disposition` per bird in the snapshot, which is a bucketed reaction tendency: `approach` / `wait_then_approach` / `watch` / `ignore`). The client plays the corresponding reaction with zero latency; the tick records the authoritative event. Because disposition is server-supplied and mood is server-supplied, client and server cannot disagree.

- **Cooldown** — 4 minutes per bird, enforced server-side (`offer_cooldown_until`) *and* mirrored in the client so the affordance is quietly unavailable rather than erroring. The PRD's rationale is respected in the math: with `S_offerTake` saturating at 3/day and a 4-minute cooldown, curiosity cannot be saturated by button-mashing.
- **Song offer** — synthesized through the same audio engine as bird calls, at low gain, from the offer motif library. Bird response is `join` / `quiet` / `call_against`, weighted by `vocal_frequency` and mood.
- **Pool offer** — adds a reflective surface to the front plane for ~3 minutes with its own reflection shader-free render pass (Canvas 2D: a clipped, vertically-flipped, blurred redraw of the mid plane at low alpha); birds may `drink`, `bathe`, or `watch`.
- The offer affordance lives in the top bar only. There is no click-a-bird-to-offer path, matching the spec.

### 5.9 Settle

`settle` event → the tick sets an `aviary.settled` flag, ramps the light curve toward evening over 6 seconds (the client animates the ramp from the curve in the snapshot), biases every bird's mood scoring toward `drowsy`/`settled`, and lengthens call intervals by 2.5×.

**Undo:** any pointer/keyboard interaction within 5 seconds of the settle emits `settle_undo`, and the client reverses the light ramp locally and immediately. Because the settle event may already be at the server, `settle_undo` carries the same `cid` correlation and the tick treats the pair as a no-op. The client does not wait for confirmation — the undo is instantaneous locally, which is the only way a 5-second mercy window feels like a mercy.

Settle and tab-close are identical to the drift engine: both end the presence window. `session_close` (via `sendBeacon`) and settle both close it; if neither arrives, the window closes implicitly when presence pings stop, with the last ping's end as the boundary.

### 5.10 New-bird arrivals

`max_birds` is raised by aviary age: **90 days → 3, 210 → 4, 365 → 5, 545 → 6, 730 → 7**. Age only; no interaction inputs feed this, by construction (the age-gate function's signature takes `adopted_at` and `now` and nothing else).

The arrival is presented as an arrival, not a reward: on the first session after the gate opens, an unnamed bird of a new species appears on the back perch, keeping distance, in `wary`-leaning mood. A quiet naming affordance appears in the top bar (not a modal, not a celebration, no confetti — an anti-announcement test asserts no new live region or toast fires). If the user doesn't name it, it stays as a visiting bird with a default name and keeps its distance until they do. Species is drawn from the pool excluding species already present until all six are used.

Starter selection at adoption: deterministic from `aviary_seed`, constrained so the two starters occupy **different call registers** (one high, one mid) — recognizability has to work from day one, and two similar voices at the start would undercut the product's central affordance before the user ever learns it.

---

## 6. Sync model

### 6.1 Why there is nothing to reconcile

Both devices read one canonical record. There is no client-side authoritative state, so there is no merge. This section is short because the architecture did the work.

### 6.2 Single-writer discipline

- `sim-worker` is the only role with `UPDATE` on `personality_vectors` and `bird_state` (Postgres grants; verified by a startup assertion in the API services that `SELECT has_table_privilege(...)` returns false — the service refuses to boot if it has been over-granted).
- Per-account tick serialization via a Redis lease plus a Postgres advisory lock keyed on `hashtext(account_id)`. Two workers cannot tick one account concurrently.
- Deltas are additive and server-authored. The client cannot express an absolute trait value: `POST /v1/aviary/events` has a closed schema in which no event type carries a trait field. A last-write-wins personality bug is not merely avoided — it is unrepresentable.

### 6.3 Event ingestion and presence integrity

The presence definition (visible ∧ focused ∧ activity within window) is computed client-side because only the client can observe those signals. That makes it forgeable and makes it vulnerable to honest bugs. Server-side defenses:

1. **Clamp per ping** — a presence event credits at most `min(payload.dur_ms, 15_000, now − last_accepted_ping_end)`. Overlapping or replayed pings credit nothing extra.
2. **Clamp per day** — at most 6 hours of presence credited per account-day. This never binds for a real user (drift saturates at 20 minutes), and it caps the blast radius of a client bug.
3. **Clamp timestamps** — `occurred_at` into `[now − 300s, now + 5s]`, which prevents a suspended laptop from dumping a day of backdated presence on resume.
4. **Signal echo** — pings carry `vis`, `focus`, `act_age_ms`; the server rejects any ping asserting presence with `act_age_ms > activity_window`. This catches client regressions where one of the three conditions is dropped, which is the exact silent failure the PRD warns about.
5. **Activity window: 4 minutes** (§16). Chosen long because watching birds without moving is the product; chosen finite because an open laptop in an empty room is not presence.

A dedicated monitor tracks the **population presence-minutes-per-active-day distribution** and alarms on a >20% week-over-week shift in the median. Presence inflation is the failure mode with no user-visible symptom and no failing test; the only way to catch it is to watch the distribution.

### 6.4 Client pull policy

Pull a fresh snapshot on: initial load (inlined, no request), `visibilitychange → visible`, a render-frame gap > 5s (laptop resume), keepalive every 30s while visible, and after any locally-emitted interaction that the server will adjudicate (offer, settle) at +2s. Never poll while hidden.

Snapshots carry a monotonic `v`; the client discards any snapshot with `v` less than or equal to the current one, which makes out-of-order responses harmless.

**No WebSockets in v1.** The tick is 60s and the snapshot carries a 90s forward window, so a 30s poll is strictly more information than the client can consume. A persistent connection would add reconnection logic, sticky-session constraints, and per-connection server memory to buy nothing. Revisit only if a future feature needs sub-second server-initiated events.

### 6.5 The one place last-write-wins is correct

Personality: never. But three fields are last-write-wins and that is right, so the rule doesn't get cargo-culted into paralysis:

| Field | Policy | Why it's safe |
| --- | --- | --- |
| `accounts.timezone` | Last write wins | It's a hint about where the user is *now*; the newest device is the best answer. Travel and device disagreement resolve correctly. |
| `birds.name` | Last write wins | User-authored, low-stakes, and the user can see the result immediately. |
| `accounts.settings` | Last write wins per key (JSONB merge, not replace) | Per-key merge prevents a stale phone from reverting a laptop's reduced-motion choice. |

The distinction to hold: **derived state the user did not author is never LWW; state the user authored directly is fine as LWW.**

### 6.6 Multi-device coherence of *presentation*

Two devices must not just agree on state but *look* like they agree. Because calls carry `(at, motif, seed)` and gestures carry `(gesture, phase, seed)`, two devices synthesize identical audio at the same wall-clock moment and animate the same gesture from the same phase. Client clock skew is corrected with a lightweight offset estimate from `server_time` on each pull (median of the last 5 samples), so scheduled calls land within ~50ms across devices — inaudible as a discrepancy, and irrelevant since nobody hears two devices at once, but it also means a visitor and a host hear the same aviary.

---

## 7. Frontend rendering pipeline

### 7.1 Scene graph and layers

Four canvases, composited by the browser, each redrawn at its own rate:

| Layer | Contents | Redraw |
| --- | --- | --- |
| `sky` | Gradient sky, distant foliage silhouettes, day/night tint | On light-curve change only (~1 Hz), plus resize |
| `back` | Background foliage, back perch, parallax offset ×0.3 | 60 Hz while wind/ornaments active, else on change |
| `mid` | **Birds**, all three perches, pool offer surface | 60 Hz |
| `fore` | Foreground branch, drifting leaves/feathers, parallax ×1.4, caption anchors | 60 Hz |

Only `mid` and `fore` are genuinely per-frame. `sky` is the cheapest possible implementation of a continuous day/night cycle: interpolate a small palette LUT from the snapshot's `light.curve`, repaint a gradient once a second.

### 7.2 The loop

```
onFrame(t):
  dt = min(t − tPrev, 100)             # clamp after a suspend; never a giant catch-up step
  simLocal.advance(dt)                 # micro-motion phases, ornaments, interpolators, mix ramps
  audio.pump(t)                        # schedule any call intents entering the 200ms lookahead
  draw(mid); draw(fore); maybeDraw(back, sky)
```

Fixed-rate presentational updates at 60Hz with an accumulator; draw is once per rAF. The loop stops entirely on `visibilitychange → hidden` (battery, and there is nothing to see). On resume, phases are advanced by the elapsed wall time before the first draw, so the bird appears to have kept preening the whole time — the aviary continued, and the resume frame proves it.

### 7.3 Bird rendering

Each bird is a small skeletal rig (body, head, beak, two wings, tail, two feet, eye) driven by ~10 parameters. Geometry is compact path data per species (authored as SVG, compiled at build time into `Path2D`-constructible commands, ~2–4KB per species). Plumage is applied as a runtime palette so the same geometry renders a bird at any plumage step without new assets — which is what makes plumage drift free at the asset layer.

**Idle micro-motion** is layered continuous noise plus discrete gestures:

- **Continuous** — three seeded 1-D simplex noise streams per bird (head yaw, head pitch, body bob) at 0.15–0.6 Hz, amplitude scaled by mood (`drowsy` 0.3×, `alert` 1.4×). Seeded per `bird_id`, so a bird's fidget signature is its own and is identical across devices.
- **Discrete** — the server-chosen gesture (`preen`, `scan`, `fluff`, `shuffle`, `tilt`) plays as a short parameter animation with jittered duration (±18%) and a randomized entry blend. Blending between gestures is a 220ms ease so a bird never snaps between poses.
- **Never a paused-looking still.** A guard test: over a 30-second render capture, every bird's parameter vector must change by more than a threshold in every 500ms window. A bird that stops moving fails CI.

### 7.4 Perch transitions

On a perch change in a new snapshot, the client plans a flight: a quadratic Bezier with an upward mid-control point, duration 480–900ms scaled by distance and mood, with wing-beat animation phase-locked to the path. If a new snapshot arrives mid-flight with a different destination, the interpolator retargets by re-solving the curve from current position and velocity — no teleport, ever. If a bird appears in a snapshot at a perch the client didn't know about (first load), it is simply drawn there; there is no fly-in except in the empty-aviary case (§7.9).

### 7.5 Ambient ornaments

Leaves and feathers are pure client-side, as the PRD requires: a small pool (max 6 concurrent) of ornaments spawned by a Poisson process (λ ≈ 1 per 12s, modulated by wind), following a sine-perturbed fall path across `fore` or `back`. Pooled objects, zero allocation per spawn after warmup.

### 7.6 Parallax and responsiveness

Parallax responds to viewport-relative pointer position at ±8px maximum for `fore`, ±3px for `back`, heavily smoothed (time constant 400ms). It is subtle by mandate, and it is disabled entirely in reduced-motion.

Layout: the scene is defined in a virtual coordinate space with three perch bands at fixed vertical fractions and horizontal slot positions expressed as fractions with configurable margins. On narrow viewports, horizontal slot spacing compresses and perch bands move slightly closer; on wide viewports, spacing expands to a maximum, past which the scene is centered with extended sky. **Invariant, tested at 14 viewport sizes from 320×480 to 3440×1440: every bird's full bounding box is inside the viewport with ≥12px margin at all times, including mid-flight.** Never crop a bird.

### 7.7 Reduced-motion mode as a designed surface

Same scene graph, different *presenter*. The renderer takes a `Presenter` interface; `MotionPresenter` and `CrossfadePresenter` implement it. This is the mechanism that prevents reduced-motion from decaying into "animations off," because there is no `if (reducedMotion) return;` anywhere in the draw code — there is a second, deliberately-authored presenter.

| Element | Motion presenter | Cross-fade presenter |
| --- | --- | --- |
| Idle micro-motion | Continuous noise | Sequence of 3–5 authored still poses per gesture, cross-fading over 1.2s each, with 2–5s holds |
| Flight | Bezier path with wing beats | 900ms cross-dissolve between origin-perch pose and destination-perch pose |
| Day/night | Continuous | Continuous but slowed (stepped at 30s intervals, cross-faded) |
| Weather | Animated rain streaks, leaf ripple | Static overlay that fades in and out; no streak motion |
| Ornaments | Drifting leaves | Removed entirely |
| Parallax | ±8px | Off |
| Top-bar fade | Animated over 600ms | Instant opacity change (no transition) |
| Calls, drift, mood, notebook | Full | **Full — unchanged** |

Activated by `prefers-reduced-motion: reduce` (respected on first paint, read from the media query before the first draw so there is no motion flash) or the accessibility settings toggle, which can also *override* the media query in either direction.

The cross-fade aesthetic gets real design time: the pose sets are authored, not sampled from the animation. A vestibular user gets a calmer aviary, not a broken one.

### 7.8 Top bar

Four items: account/settings, accessibility, notebook, offer. Nothing else, ever — a test asserts the top-bar item registry has exactly four entries and fails on addition without an explicit test update, which forces the conversation.

Fade: after 4s of pointer stillness and no keyboard activity, opacity → 0.12 over 600ms. Restores instantly on `pointermove`/`keydown`. **Never fades while any top-bar element has focus, while a popover is open, or while a screen reader is detected as active** (approximated by: any programmatic focus into the bar, or `prefers-reduced-motion`, or the accessibility settings having been opened this session). The faded bar remains fully keyboard-reachable and fully hit-testable; fade is opacity only, never `pointer-events: none`.

### 7.9 First frame, loading, and the empty aviary

**The 500ms path.** No spinner exists in the codebase — there is no spinner component to accidentally use.

1. `aviary-edge` returns HTML containing: critical CSS (~3KB), the inlined bootstrap snapshot (~3–5KB JSON), and the `boot` module inline (~28KB gz). Compressed HTML target ≤ 45KB.
2. `boot` parses the snapshot, sizes the canvas, and draws frame one: birds at their current perches in their current gestures **at their current phases**, sky at the current light level, ornaments already mid-flight. No fade-in. No entry animation. The first painted frame is a frame from the middle of a continuous animation.
3. `aviary` chunk (audio, narration, captions, interactions) loads immediately after and initializes without any visible transition. Audio starts as soon as the chunk lands and the autoplay policy permits (§8.7).

Budget (mid-tier Android, 4G, cold cache):

| Stage | Budget |
| --- | --- |
| DNS + TLS + connection (edge PoP) | 120ms |
| TTFB for HTML+snapshot | 140ms |
| HTML/CSS parse + inline module eval | 90ms |
| Layout + first canvas draw | 70ms |
| **Total to first bird visible** | **420ms** (80ms headroom against the 500ms budget) |

**When the snapshot isn't inlined** (edge timeout, cold session, direct API failure): the client draws the **quiet field** — sky gradient at the correct light level for the client's local time, soft background foliage, one or two faint ambient motion cues, no birds, no text, no spinner. It holds until the snapshot arrives, then draws birds in place with no transition. This reads as the aviary catching up, which is the intent.

**Empty aviary** (post-adoption, pre-first-bird) is the same quiet field. Each starter bird then enters with a soft fly-in from offscreen to its starting perch, staggered by ~2.2s. This is the only fly-in-from-offscreen in the product, and after adoption the user never sees an empty aviary again.

### 7.10 What the client never does

No client-side tick. No client-authored personality. No client-side mood inference. No spinner. No toast. No modal on return. There is no toast/snackbar primitive in the design system, and the anti-announcement suite (§13.6) asserts that no `role="status"`/`role="alert"` region is created on the aviary route outside the narration live region.

---

## 8. Audio pipeline

### 8.1 Graph

```
per call:  wavetable osc (PeriodicWave from bird's harmonic profile)
           + filtered noise source (breath component)
             → syllable gain (amplitude envelope)
             → per-bird gain  ×  per-zone gain  ×  listen-in gain
             → [ dry ──────────────────────────────► ]
               [ send → algorithmic reverb (delay+allpass network, ~2KB code) → ]
             → master limiter → destination

separate:  ambient bed — low-level filtered noise + very sparse distant call layer,
           its own gain, unaffected by listen-in except for a slight lift
```

No convolution reverb (impulse responses cost bundle we don't have). A small Schroeder-style delay/allpass network gives the "outdoor space" cue at negligible size.

### 8.2 Call grammar

```
call       := phrase ( gap phrase ){0..k}
phrase     := motif ∘ transform*
motif      := syllable{1..5}                       -- from the species motif library
syllable   := { f0: [(t, hz)…]            -- pitch contour control points
              , dur_ms
              , amp: { attack, decay, sustain, release }
              , timbre: { harmonics: [w1..w8], noise_ratio }
              , vibrato: { rate_hz, depth_cents } }
transform  := transpose(semitones)
            | timeScale(ratio)
            | ornament(kind, position)              -- grace note, trill, terminal fall
            | repeat(n, amplitude_decay)
```

Each species ships 4–6 motifs. A realized call is produced by `realize(motifLibrary[species], voiceSeed, callSeed, moodParams, personalityParams)` — a pure function. The same function, given the same seeds, produces the same call on every device and inside the caption generator.

### 8.3 Recognizability: the invariant/variant split

This is the single most important design decision in the audio system, because "a user who has spent two weeks with Pip should know Pip's call by ear, even across mood and drift" is otherwise an aspiration with no mechanism.

| Fixed at adoption — **never** changes | Modulated by mood and personality |
| --- | --- |
| Species motif library (the melodic shapes) | Number of phrases per call (`k`) |
| Pitch centre offset (bird's own ±3 semitones within species range) | Gap length between phrases |
| Harmonic profile / timbre weights | Global `timeScale` (tempo) |
| Noise ratio (breathiness) | Ornament density |
| Vibrato rate | Amplitude / intensity |
| Syllable attack shape | Inter-call interval (λ) |

Timbre and pitch centre are the ear's identity cues; tempo, ornament, and density are the ear's *expression* cues. By fixing the former in `birds.voice_seed` and modulating only the latter, a bird sounds like itself when drowsy, when curious, and after three weeks of vocal-frequency drift. Drift changes **how often** and **how elaborately** a bird calls, never **what it sounds like**.

### 8.4 Scheduling

The server sends call intents with absolute times. The client keeps a 200ms lookahead: on each frame, any intent whose `at` (corrected for clock offset) falls within the lookahead is scheduled onto the WebAudio timeline with sample-accurate `start()` times. This gives sample-accurate timing without a high-frequency timer, and it is why a chorus sounds like a chorus rather than like two independent players.

### 8.5 Chorus and mixing

Per-bird gain is `zoneGain(perch) × intensity × listenInFactor`, where `zoneGain` is front 1.0 / middle 0.72 / back 0.5, paired with a rising reverb send (front 0.1, middle 0.2, back 0.35) and a gentle high-shelf cut on the back zone. Distance is therefore audible, and a bold bird coming forward is audible as coming forward — perch is a *sonic* signal as well as a visual one.

Chorus arises from genuine overlap of independently-realized calls, biased by the scheduler (§5.5). Because every call is realized fresh from the grammar, two birds calling simultaneously never phase-cancel the way two copies of a recording would — the artifact the PRD names is unreachable because there are no recordings.

Voice budget: max 12 concurrent syllable voices; if exceeded, the quietest back-zone voice is dropped. Realistically 7 birds × ~1.5 syllables in flight never approaches this.

### 8.6 Listen-in mix

- **Engage** — focused bird +5 dB and reverb send reduced to 0.06 (it moves forward in the space, not just up in volume); all other birds ramp to −9 dB; ambient bed +1.5 dB. Ramp: 1.4s, `setTargetAtTime` with a time constant of 0.45 so it eases rather than slides linearly.
- **Disengage** — the same ramp in reverse, 1.4s. This is the "mix decay."
- **Others never go silent.** The floor is −9 dB, which is a hard-coded constant with a test asserting no code path sets a non-focused bird's gain below it. Silence would make the aviary a set of soloable tracks, which is the failure the PRD names.
- **Disengage triggers:** clicking the focused bird again, focusing a different bird (which cross-ramps directly, no pass through neutral), clicking empty scene, `Escape`, or keyboard focus leaving the scene.
- Listen-in also emits `listen_in_start`/`listen_in_end` for drift, and drives narration priority.

**The recognizability gate** (a launch gate, §14.6): a listening study with 12 participants. Each learns two birds over three sessions, then identifies calls in a 7-bird chorus. Pass condition: ≥75% identification accuracy at 7 birds, and no accuracy cliff between 5 and 7. If it fails, the ship cap drops to the largest passing count and the audio-mix work to raise it becomes a post-v1 project. This is how the "seven is empirical" claim gets treated as empirical.

### 8.7 Autoplay, silence, and the WebAudio fallback

Browsers require a gesture before audio starts. This is a real conflict with "calls already audible on the first frame," and it cannot be engineered away — so it is handled without announcement:

1. On load, create the `AudioContext`. If it starts `running` (common on a return visit within the same session, or where the user has granted the origin sound), audio begins with the scene, mid-call, as intended.
2. If it is `suspended`, the aviary renders and animates normally with **no prompt, no unmute badge, no banner**. The first pointer or keyboard interaction anywhere resumes the context, and audio fades in over 800ms mid-call — it sounds like the sound was always there and the user just tuned in. Captions render during the silent period if the user has them on.
3. There is one exception, in accessibility settings only: a plain matter-of-fact line explaining that the browser requires an interaction before sound can play. It lives where a user goes to solve a problem, not on the aviary surface.

**True WebAudio unavailability** (unsupported, blocked, hardware failure): the aviary plays in silence with **captions forced on by default**, and a single matter-of-fact line in accessibility settings. No recorded-audio fallback path exists in the codebase — there is no audio file loader to fall back to.

### 8.8 Memory discipline

WebAudio source nodes are single-use by specification, so the pooling target is everything else: `PeriodicWave` objects are created once per species-timbre and cached; noise buffers are pre-generated and shared; envelope parameter arrays are pooled; the reverb network is built once. Scheduled-node references are dropped in an `onended` handler that also returns any pooled auxiliaries. The 30-minute no-growth test (§13.5) covers this specifically, since audio is the most likely source of leak in the product.

---

## 9. Voice kernel — notebook, narration, captions

### 9.1 One voice, enforced mechanically

Three surfaces (notebook entries, screen-reader narration, call captions) must sound like one observer, and the whole product must maintain the naturalist/matter-of-fact split. Doing this by style guide alone will fail on the tenth contributor. So:

**`packages/voice-kernel`** — shared server and client:

- A **lexicon**: allowed bird verbs (`notice`, `perch`, `settle`, `preen`, `call`, `fluff`, `drift`, `watch`, `tilt`, `scan`), adjectives, time phrases, weather phrases. Words not in the lexicon cannot be emitted by a generator.
- A **template graph** per observation type: slot-filled sentence structures with multiple realizations, chosen by seeded PRNG.
- An **anti-repetition memory**: per account, the last 40 realized phrase-template ids are stored; the generator will not reuse a template within that window. This is what stops the notebook reading as a mail merge after two months.
- Two **string catalogs**: `catalog.naturalist.ts` and `catalog.system.ts`.

**CI lints:**

| Lint | Applies to | Rule |
| --- | --- | --- |
| `voice/naturalist` | naturalist catalog + all generator templates | Lowercase start; present tense (banned past-tense auxiliaries); no `!`; no second person (`you`, `your`); no announcement lexicon; no numerals referring to counts of user behavior |
| `voice/system` | system catalog | Sentence case; no naturalist verbs used metaphorically; must state what happened and what to do |
| `voice/no-gamification` | every user-facing string, both catalogs | Bans `streak`, `achievement`, `badge`, `level`, `score`, `unlocked`, `days visited`, `welcome back`, `great to see you`, `you've been`, `keep it up`, `milestone` |
| `voice/no-trait-numbers` | every user-facing string and every template | Bans interpolation of any field originating from `personality_vectors` |
| `voice/catalog-boundary` | route chunks | `aviary` chunk may not import `catalog.system`; `system` chunk may not import `catalog.naturalist` (except the shared error surface) |

### 9.2 Screen-reader narration

Generated **server-side**, in the tick, from the same state that produces the visual scene — so the narration cannot drift from the scene, and so it is identical on every device.

Cadence: one update per 45s at idle (within the specified 30–60s), and prompt updates on user-initiated events: return-greeting (≤1.5s), offer reaction (≤1s), settle (≤1s), listen-in engage (≤1s), and a new notebook entry. A coalescing queue holds at most 2 pending updates; if a third arrives, the middle one is dropped rather than queueing a backlog the user has to wait through. Idle narration is *suppressed entirely* for 20s after any prioritized update, so the user is never talked over.

Composition, in order of what it mentions: the most salient bird (the one that most recently called, was greeted by, or is focused), then a second bird if its state is notable, then the aviary condition (light, weather). Prose, not a list:

> a small grey bird is on the front rail, calling softly. another sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Delivered into a single `aria-live="polite" aria-atomic="true"` region. Never `assertive` — an assertive aviary would interrupt the user's own work, which is the audio equivalent of a toast.

Narration is available to **visitors** too, with host-behavior references (e.g. greeting) absent. Accessibility parity does not stop at the host's own account.

### 9.3 Field notebook

Detectors run each tick and propose candidate observations with a novelty score:

| Detector | Fires on |
| --- | --- |
| `greeter_change` | Today's greeter differs from the recent modal greeter |
| `first_in_window` | A bird did something for the first time in N days (greeted first, came to the front perch, joined a chorus) |
| `long_quiet` | An unusually long interval with no calls |
| `chorus` | Three or more birds calling within a 6s window |
| `weather_moment` | Bird behavior notably coinciding with rain or wind |
| `perch_unusual` | A bird occupying a zone it rarely occupies |
| `proximity` | Two birds perching adjacent for an extended period |
| `night_caller` | The nightjar-like species calling late |

**Sparsity governor** — the mechanism that keeps the notebook from becoming a feed:

```
- Global cooldown: no entry within 36h of the previous entry
- Per-detector refractory: 10 days
- Novelty gate: score ≥ 0.55, where score falls with how recently a similar
  observation was written for the same subject
- Target rate: ~1 entry per 2–4 days for a regularly-visited aviary; asymptotic
  cap of 12 entries per 30 days regardless of activity
```

A CI test runs the heavy profile (60 min/day, every day) for 90 simulated days and asserts the entry count lands in 25–40 — i.e., a very active user gets *more* moments but not proportionally more, which is the sparsity the PRD asks for.

**Subject constraint** — `subject_kind` is `bird` or `aviary`. There is no code path that constructs an entry about the user, and the schema `CHECK` makes one unrepresentable. "Pip greeted first today" is expressible; "you visited every day this week" is not.

Entries store their realized prose permanently and are never regenerated (a voice change must not rewrite history). Read-only: no PATCH or DELETE endpoints exist. Pagination is cursor-based and unbounded backward — nothing is archived or hidden.

### 9.4 Captions

Generated client-side by the **same function that realizes the call**, so caption and audio cannot diverge:

```
realizeCall(...) → { audioParams, captionSpec }
captionSpec → prose via voice-kernel:  contour + syllable count + duration + intensity + zone
```

| Realized call | Caption |
| --- | --- |
| 3 syllables, rising contour, short, low intensity | "a soft three-note rise" |
| 2 phrases, trill motif, gap between | "a low trill, paused, low trill again" |
| 1 syllable, sharp attack, back zone | "a single sharp call from the back perch" |

Captions render as small text near the calling bird's projected position, fading in over 200ms and out 600ms after the call ends, positioned to avoid overlapping another caption or the bird itself, and constrained to stay inside the viewport. They pass WCAG AA against the aviary at every light level via a subtle scrim (a soft, low-opacity rounded backdrop tuned per light level rather than a hard box).

Captions are `aria-hidden="true"` when narration is active — otherwise a screen-reader user hears every call announced twice, once as narration and once as caption text, which is precisely the queue-flooding the PRD warns about. When narration is not in use (a hearing-impaired sighted user, audio off, noisy room), captions are exposed normally.

Captions default **on** when WebAudio is unavailable; otherwise off, toggled in accessibility settings.

---

## 10. Accessibility surfaces

### 10.1 Stance

Accessibility ships with v1 or v1 does not ship. It is in the launch gates (§14.6), not in a follow-up milestone. The three surfaces (narration §9.2, reduced-motion §7.7, captions §9.4) are designed surfaces with their own product work, not fallbacks. Two structural commitments back this up:

1. **No `aria-label` automation of state.** There is a lint that fails on any `aria-label` whose value interpolates a mood, trait, or perch value. State reaches the screen reader as prose, through the voice kernel, or not at all.
2. **Visitors get the same accessibility surfaces as hosts.** A read-only view is still a view.

### 10.2 Keyboard navigation

| Key | Action |
| --- | --- |
| `Tab` | Cycles top-bar items, then the aviary scene as a single composite widget |
| `Tab` (into scene) | Focuses the first bird (leftmost, front-most) |
| `←` `→` | Move focus between birds in visual order |
| `↑` `↓` | Move focus between perch zones |
| `Enter` / `Space` | Listen in on the focused bird |
| `Escape` | Exit listen-in; if not listening in, return focus to the top bar |
| `Tab` (out of scene) | Leaves the aviary as one stop, not seven |
| `O`, `N`, `S` | Shortcuts to offer, notebook, settle (documented in accessibility settings, not on the surface) |

The scene is a composite widget (`role="application"` with a labelled group and roving `tabindex`), so a keyboard user is not forced through seven tab stops to reach the settings icon. Every action reachable by pointer is reachable by keyboard, including the offer picker (a fully navigable popover with focus trap and restore) and settle (including its 5-second undo, which any keypress triggers).

### 10.3 Focus indication

A dual-stroke ring — a 2px light inner stroke and a 2px dark outer stroke — so it reads against both a bright noon sky and a dim night scene without needing to know the background. Drawn in the canvas for birds (following the bird's bounding path, offset 4px), in CSS for chrome. It is never suppressed by the top-bar fade: focus forces the bar to full opacity.

### 10.4 Contrast and motion preferences

All user copy (top bar, settings, account, errors, captions, any visually-displayed narration) meets WCAG AA (4.5:1 body, 3:1 large). Automated axe-core checks run in CI on every route in both light and night aviary states, and captions are contrast-checked programmatically against sampled backdrop luminance at 6 light levels.

`prefers-reduced-motion` is honored on the first paint, before any animation begins. `prefers-contrast: more` raises chrome contrast and strengthens the focus ring. `prefers-reduced-transparency` disables the top-bar fade (the bar stays at full opacity).

### 10.5 Accessibility settings surface

Matter-of-fact voice. Contents: reduced motion (auto / on / off), call captions (on / off), narration detail (standard / brief), audio (on / off), plus the plain explanation of the browser-audio-gesture requirement when relevant. Reachable by keyboard from the first tab stop, lazily loaded but prefetched when the top bar first receives focus (so a keyboard user never waits on a chunk).

---

## 11. Performance budgets and observability

### 11.1 Budgets and enforcement

| Budget | Target | Enforcement |
| --- | --- | --- |
| Initial JS bundle | **< 2MB gz** total; **< 60KB gz** on the critical first-paint path | `size-limit` per chunk in CI; PR fails on regression > 2% |
| Time to first bird | **< 500ms** p75 on mid-tier Android / 4G | Synthetic fleet on throttled real devices, per release; RUM p75 alarm |
| Idle frame rate | **60fps** on a 2019 mid-range laptop | Automated 5-minute render capture; fails if > 1% of frames exceed 16.7ms or any frame exceeds 50ms |
| Memory | **No growth over 30 min** | CI: 30-minute headless session; heap after forced GC must be within 5% of the 5-minute baseline; detached-node count must not grow |
| Snapshot latency | p95 < 120ms, p99 < 300ms | Server metric with alarms |
| Tick latency | p99 < 5s (alarm), target p50 < 20ms | Server metric with alarm exactly as specified |
| Tick freshness | p99 of "age of last applied tick for a warm account" < 180s | Server metric — see below |

**Chunk budget allocation (gz):**

| Chunk | Budget |
| --- | --- |
| `boot` (inline critical) | 28KB |
| Render engine + species geometry | 190KB |
| Audio engine + motif libraries | 150KB |
| Voice kernel + templates | 70KB |
| Chrome (Preact + top bar + notebook) | 90KB |
| `system` chunk (lazy) | 180KB |
| **Total** | **708KB — well under 2MB, with the headroom reserved for species geometry growth and reduced-motion pose sets** |

Coming in far under the 2MB cap is deliberate: 2MB is the cap at which the 500ms budget becomes unrecoverable, and the 500ms budget is the one that actually matters. We treat 2MB as a hard ceiling and ~800KB as the working target.

**Tick freshness over tick latency.** Latency alone can look healthy while a backlog silently starves accounts. Freshness — how stale is the newest tick for an account someone is watching — is the SLI that actually corresponds to the user's experience of a live aviary, and it is the primary sim alarm.

### 11.2 What we measure

- **Server:** request rate/latency/error by endpoint, tick latency and freshness, tick backlog depth, event ingestion rate, snapshot cache hit rate, DB pool saturation, magic-link delivery/bounce rate, mail queue depth.
- **Client RUM, aggregate only:** page load timing, first-bird-render timing, frame-time histogram, long-task count, audio-context state distribution and failure counts, snapshot fetch latency, JS error counts by class, reduced-motion/captions adoption rates as plain counters.
- **Synthetic:** a browser fleet from ~5 geographies running the full aviary hourly on throttled profiles, asserting first-bird time, frame rate, that audio produced non-silent output, and that no error surfaced.
- **Product health, aggregate only:** anonymized session-duration histogram (explicitly allowed by the PRD), account creation rate, sign-in success rate.
- **Calibration, synthetic accounts only:** drift trajectories from the observatory profiles (§5.2). Never from real accounts.

### 11.3 What we deliberately do not measure

Named here so that "we could just add a dimension" is a visible decision, not a quiet one:

- No per-account or per-bird dimension on any metric. Not even hashed. The metrics wrapper's type signature makes account id unattachable (§12.3).
- No cross-account drift, mood, or trait aggregates. No "average boldness" dashboard — that would convert the product into a data product and is exactly what the privacy section forbids.
- No per-user retention cohorts, no DAU/WAU-by-user tables, no visit-frequency series per account. There is deliberately no substrate from which a streak could later be computed, even internally.
- No visit graph across accounts. The visit log is per-host and is never aggregated.
- No content of notebook entries or narration in logs or telemetry.

### 11.4 Alerting and error budget

| Alarm | Threshold | Page? |
| --- | --- | --- |
| Tick freshness p99 | > 180s for 5 min | Yes |
| Tick latency p99 | > 5s for 5 min | Yes |
| Snapshot 5xx rate | > 1% for 5 min | Yes |
| Magic-link delivery failure | > 5% over 15 min | Yes |
| First-bird RUM p75 | > 500ms for 30 min | No — ticket |
| Audio-context failure rate | > 3% of sessions | No — ticket |
| Presence-minutes median shift | > 20% week over week | No — ticket, but investigate urgently (§6.3) |

---

## 12. Privacy and security engineering

### 12.1 Email lives in exactly one place

- `accounts.email_ciphertext` — envelope encryption with a KMS-managed data key, per-account IV.
- `accounts.email_lookup_hash` — `HMAC-SHA256(lowercased_email, pepper)` where the pepper is in KMS, not in the database. This is the only field sign-in queries against; it is not reversible and is not usable as a join key to anything else.
- **Every other reference to an account, anywhere — DB foreign keys, service calls, log lines, metric labels, queue keys, sharding, error reports — is `accounts.id` (a synthetic UUID).**

Enforcement, because this is the rule the PRD calls "impossible to retrofit":

1. A **log scrubber** in the shared logging package redacts anything matching an email pattern, and increments a counter when it does — a nonzero counter is a bug to fix, not a success.
2. A **CI grep** fails the build on any log/metric/span call whose arguments reference an `email` field.
3. A **type-level guard**: email is carried in a branded `Pii<string>` type that the logger and the metrics client refuse to accept as an argument (no overload exists).
4. The visitor email in `invites` follows the identical discipline.

### 12.2 Auth and tokens

| Token | Properties |
| --- | --- |
| Magic link | 256-bit CSPRNG, stored as SHA-256 hash, 15-min expiry, single use (`consumed_at` set in the same transaction that mints the session), constant-time compare, purpose-bound (`signin` vs `email_change` are not interchangeable) |
| Session | 256-bit, `HttpOnly; Secure; SameSite=Lax`, hashed at rest, 90-day sliding expiry, revocable per device |
| Visit | 256-bit, hashed at rest, 30-day expiry, revocable, scoped to a single host aviary read projection |

No account enumeration: `POST /auth/magic-link` responds identically for known and unknown emails. Rate limits: 5 links/hour/email, 20/hour/IP, and a global circuit breaker on the mailer. CSRF: `SameSite=Lax` plus an origin check on all mutating endpoints. Standard security headers, strict CSP with no inline script except the nonce-tagged bootstrap payload (which is `application/json`, not executable).

Email change: verification link to the *new* address; the old address keeps working until verification lands; a matter-of-fact notice is sent to the old address after the switch (a security notice, not a product notification — the distinction matters and is worth stating so the no-email rule isn't read as forbidding security mail).

### 12.3 The telemetry boundary as topology

The privacy commitment is implemented as network topology, not policy:

- The analytics/telemetry stack runs in a separate account/VPC with **no route** to the simulation database and no credential that could reach it. This is asserted by an infrastructure test in CI that attempts a connection from the telemetry subnet and requires it to fail.
- The metrics client's type signature makes a per-account dimension unrepresentable: `emit(name: MetricName, value: number, dims: AllowedDims)` where `AllowedDims` is a closed union of operational dimensions (region, endpoint, status class, browser class, device class). There is no `account_id` variant, and adding one is a change to a shared type that shows up in review.
- No ETL job reads `personality_vectors`, `bird_state`, `interaction_events`, or `notebook_entries`. A schema-permissions test asserts the analytics role has no grants on those tables.
- If ML ever exists for any feature, per-bird fields are not available to it — because the pipeline that would feed it cannot read them.

### 12.4 Deletion, export, retention

- **Soft delete:** `deletion_requested_at` set, `hard_delete_after = +30 days`. The account still signs in; any signed-in page shows a matter-of-fact recovery affordance ("This account is scheduled for deletion on <date>. Restore it."). During soft-delete the sim continues to tick, so a recovered account has not lost 30 days of its aviary — the aviary continued without the viewer, which is exactly the product's premise.
- **Hard delete:** a daily job removes every row keyed to the account across all tables (accounts, birds, vectors, drift ledger, bird state, events across all partitions, notebook, sessions, magic links, invites, visit sessions) plus queued mail. It writes an audit record containing the account UUID and timestamp only.
- **Backups:** PITR window and logical backup retention are both set to **30 days**, so hard-deleted data ages out of backups within the same window as the soft-delete promise. Documented in the privacy policy.
- **Event retention:** `interaction_events` partitions are dropped after 90 days. Drift has already been applied and is in the canonical vector; the raw events are not needed and holding them longer is holding a behavioral record we promised not to keep.
- **Export:** async job produces JSON — account settings, birds (id, name, species, adopted date), current moods, notebook entries, and current personality vectors — emailed as a signed 24-hour link. See §16 for the noted tension around including vectors.

---

## 13. Testing strategy

### 13.1 Determinism and golden replays

The sim's purity (§2.5) makes it exhaustively testable. A corpus of ~30 golden scenarios (each a fixed event log plus a fixed start state) is replayed on every CI run; the full resulting state — vectors, moods, perches, call intents, notebook prose — is byte-compared against a committed fixture. Any behavioral change surfaces as a fixture diff in review, which is the right place to notice that a refactor changed how birds behave.

A separate **replay-equivalence test** runs the same 30-day scenario two ways — 43,200 individual minute-ticks vs. one lazy catch-up — and asserts bit-identical output. This is the test that makes §2.5's cost optimization safe.

### 13.2 Property tests (fast-check, 10k cases each)

- Monotonicity: no event sequence decreases any trait.
- Traits stay in `[0,1]`.
- Daily delta ≤ `Δ_max` for every trait under every input, including hostile ones.
- Idempotency: replaying any event batch produces no additional state change.
- Presence clamping: no combination of overlapping, backdated, duplicated, or oversized pings credits more presence than wall-clock elapsed.
- Ordering: shuffling the arrival order of events with the same `occurred_at` produces identical final state.
- Mood hysteresis: no mood changes more than once per 90 seconds.

### 13.3 Time-warp harness

A test-only driver that runs the sim over simulated months in seconds, given a scripted presence/interaction profile. It powers the drift observatory (§5.2), the notebook sparsity test (§9.3), the age-gate tests (§5.10), and manual QA ("show me this aviary at day 60").

### 13.4 Rendering and audio

- **Frame-time capture:** headless Chrome with CPU throttling, 5 minutes, asserting the 60fps budget.
- **Never-still test:** every bird's parameter vector must change in every 500ms window (§7.3).
- **Viewport invariant:** 14 viewport sizes × mid-flight states, asserting no bird is cropped.
- **Reduced-motion test:** in cross-fade mode, no continuous-motion parameter changes more than once per 800ms; ornaments are absent; calls, drift, and mood updates still occur.
- **Audio variation test:** 500 consecutive realizations of one bird's call; assert no exact repeat, assert pairwise parameter distance exceeds a floor within any 20-call window, and assert the *invariant* parameters (timbre, pitch centre, vibrato rate) are identical across all 500. That single test encodes both "never canned" and "always recognizable."
- **Offline render check:** an `OfflineAudioContext` render of a 7-bird chorus, asserting no clipping, no silence, and a spectral centroid within an expected band.

### 13.5 Memory

30-minute headless session with continuous interaction (offers, listen-in, notebook open/close, tab hide/show cycles). Assert: post-GC heap within 5% of the 5-minute baseline; detached DOM node count flat; `AudioNode` live count bounded; ornament pool size bounded. Runs nightly, not per-PR (it's a 30-minute test), and is a launch gate.

### 13.6 The anti-announcement suite

These tests exist because the PRD identifies these as the violations most likely to be introduced by a well-meaning contributor. They are cheap and they never expire.

1. No `role="status"` or `role="alert"` node exists on the aviary route other than the single narration live region.
2. Session start produces zero text nodes containing any of: "welcome", "back", "hi", "hello", "you", "your", plus the gamification lexicon.
3. The top-bar item registry has exactly four entries.
4. No component named `Toast`, `Snackbar`, `Banner`, or `Confetti` exists in the repo.
5. The naturalist string catalog contains no second-person pronouns.
6. A new-bird arrival produces no live-region update and no modal.
7. No response body from any endpoint contains a field whose value derives from `personality_vectors`, verified by a tainting test that marks vector values and asserts they never reach a serializer (export excepted, and that exception is explicit and singular).

### 13.7 Accessibility

axe-core on every route in both day and night states in CI. Manual screen-reader script (VoiceOver/Safari, NVDA/Firefox) per release covering: sign-in, first session with return-greeting, listen-in via keyboard, offer via keyboard, settle plus undo, notebook navigation, and settings. Narration output is snapshot-tested against the voice lints. Keyboard-only traversal of every flow is automated.

### 13.8 Load

Target v1: 100k accounts, 5k concurrent warm. Soak tests on tick throughput with the tiering enabled, snapshot read throughput at cache hit rates from 0–95%, and event ingestion at 10× expected. Also a specific test for the cold-account thundering herd (a large population returning at once after a long quiet period, e.g. after an outage) — the catch-up path must degrade to `stale: true` rather than to timeouts.

---

## 14. Rollout

### 14.1 Team shape

| Role | Count | Focus |
| --- | --- | --- |
| Client/render engineer | 2 | Canvas pipeline, both presenters, scene, chrome |
| Audio engineer | 1 | Call grammar, synthesis, mixing, captions |
| Backend engineer | 2 | Sim worker, APIs, auth, sync, infra |
| Design (visual + motion) | 1 | Species art, palettes, reduced-motion pose sets, focus/contrast treatments |
| Generalist / a11y + QA | 1 | Voice kernel, accessibility surfaces, test harnesses |

~24 weeks to v1, six milestones. Accessibility work is inside each milestone, never after.

### 14.2 Milestones

| # | Weeks | Deliverable | Exit criteria |
| --- | --- | --- | --- |
| **M0 — Skeleton** | 1–3 | Repo, CI, schema, auth end-to-end, empty aviary renders a quiet field | A user can sign in with a magic link and see the quiet field. Voice lints and the anti-announcement suite are live *from day one* (before there is anything to violate). |
| **M1 — One bird alive** | 4–8 | Sim tick, one species, personality + mood + perch, procedural calls, idle micro-motion, snapshot pull | One bird lives on a server tick, calls procedurally with real variation, and looks alive on the client. Golden replay tests green. |
| **M2 — The aviary** | 9–13 | 6 species, 2–7 birds, bird-to-bird interaction, chorus, day/night, weather, listen-in, mixing | Seven birds at 60fps; chorus is emergent; listen-in ramps correctly; internal ear-test says the birds are distinguishable |
| **M3 — Interactions & voice** | 14–17 | Return-greeting, offers, settle + undo, field notebook, narration, captions, reduced-motion presenter | Full a11y pass on a real screen reader; reduced-motion mode reviewed as its own designed surface, not compared to the motion one |
| **M4 — Accounts & sync** | 18–20 | Multi-device, export, delete, sessions, visits end-to-end, telemetry boundary | Two devices provably show one aviary; visitor produces zero host drift; infra test proves the telemetry boundary |
| **M5 — Calibration & hardening** | 21–24 | Drift calibration, perf work to budget, memory, load, closed beta | All launch gates green (§14.6) |

### 14.3 Beta

- **Internal (week 18, ~15 people)** — real accounts, real weeks of drift. This is when three-week drift is first *felt* rather than measured, and it is why internal beta starts six weeks before launch: the calibration feedback loop is inherently three weeks long, so it cannot start in week 23.
- **Closed beta (week 21, 200 invited users)** — recruited to over-sample accessibility needs (screen-reader users, reduced-motion users, users who browse with audio off). Feedback collected by direct interview, not in-product prompts (an in-product survey would be an announcement surface).

### 14.4 Ramping birds per aviary

At launch, no account is old enough to reach the age gates, so ramping is about *readiness*, not about users' aviaries growing.

- Server-side config `MAX_BIRDS_CEILING`, launched at **3**, gates the age function's output regardless of aviary age. It is raised to 5, then 7, as the recognizability study (§8.6) and the seven-bird frame-rate budget hold on real traffic.
- The age gates themselves are fully implemented and tested via time-warp from M2, so no rushed work is needed when the first accounts turn 90 days old.
- A dedicated cohort of internal accounts is seeded with backdated `adopted_at` values to exercise 3-, 5-, and 7-bird aviaries in production from launch day, so we are not discovering seven-bird performance from real users at day 730.

### 14.5 Instrumented from day one

Everything in §11.2 ships at M0 or with its feature — not retrofitted. Specifically at launch: tick freshness and latency, first-bird RUM, frame-time histograms, audio-context state distribution, magic-link delivery, the presence-minutes distribution monitor, and the drift observatory on synthetic profiles.

### 14.6 Launch gates

Every one is measured, none is a vibe check.

1. Drift calibration: the three thresholds (§5.2) pass for all four observatory profiles.
2. Recognizability study: ≥75% at the shipping bird cap (§8.6).
3. First bird < 500ms p75 on the throttled synthetic fleet.
4. 60fps with 7 birds on the reference 2019 laptop.
5. 30-minute memory test green.
6. Bundle under budget, critical path under 60KB gz.
7. axe-core clean; manual screen-reader script passes on VoiceOver and NVDA.
8. Reduced-motion mode signed off by design as its own surface.
9. Anti-announcement suite green; voice lints green on 100% of user-facing strings.
10. Telemetry-boundary infra test green; PII grep clean.
11. Multi-device coherence test green; visitor-produces-no-drift test green.
12. Backup restore drill completed in staging: a personality vector corruption restored exactly from PITR + ledger.
13. Disaster runbook reviewed, containing no procedure that resets or reseeds a bird.

### 14.7 Post-launch watch (first 30 days)

Daily review of: tick freshness, presence-minutes distribution (inflation is silent), first-bird RUM, audio-context failures by browser, magic-link deliverability, and drift trajectories on synthetic profiles. Weekly review of the notebook entry-rate distribution — the sparsity governor is calibrated on simulated activity and real users will surprise it.

---

## 15. Risks

Ordered by expected damage. Each has a detection mechanism, because the characteristic failure mode of this product is *silence*: nearly every risk below degrades the experience without throwing an error.

| # | Risk | Why it's bad here | Mitigation | Detection |
| --- | --- | --- | --- | --- |
| 1 | **Drift miscalibration** | Too fast → Tamagotchi; too slow → screensaver. Either kills the core promise, and neither throws. | Explicit closed-form drift function with derived constants (§5.2); observatory in CI; three-week internal beta starting week 18 | Observatory trajectory diffs per build; alarm on >10% shift; internal beta qualitative check at 21 days |
| 2 | **Personality vector loss or corruption** | The worst failure the PRD names. A reset bird passes every unit test and quietly un-reveals itself to the user. | Sim-worker as sole writer via DB grants; append-only drift ledger; PITR; restore drill as a launch gate; no reset/reseed procedure exists in any runbook | A daily consistency job recomputes `vector_from_seed + Σ ledger` and compares to the stored canonical value, alarming on divergence — this catches silent corruption that nothing else would |
| 3 | **Presence inflation** | A laxer presence definition corrupts drift across the whole population, silently, with no failing test — exactly as the PRD warns. | Three-signal conjunction client-side; server-side clamps and signal echo (§6.3); per-day cap | Population presence-minutes median monitor with a 20% WoW alarm; a synthetic client that asserts a hidden tab accrues zero presence |
| 4 | **Audio uncanniness** | Once a user hears a repeat, the spell does not recover. | Grammar with a real transform space; no recorded audio anywhere in the repo; invariant/variant split | 500-call variation test; offline chorus render checks; ear-test panel each milestone |
| 5 | **Accessibility regression** | A narration or reduced-motion regression is invisible to sighted, motion-tolerant developers. | Both surfaces are product-owned, in the launch gates, with a second presenter rather than a conditional | axe in CI on every route/state; manual SR script per release; reduced-motion render assertions |
| 6 | **Announcement creep** | "Just a small toast" is the single most likely charm-destroying change, and it will arrive in a friendly PR. | Anti-announcement suite (§13.6); no toast primitive exists; voice lints on every string | CI, on every PR, forever |
| 7 | **Gamification creep** | Same shape as #6, arriving as a "harmless" counter. | No counter substrate in the schema; `subject_kind` constraint; lexicon lint | CI; migration review flags any new per-account counter column |
| 8 | **Perf regression eroding the 500ms budget** | Above 500ms the product becomes an app that loads, which is the failure the PRD names as affective, not technical. | Per-chunk size limits; ~800KB working target against a 2MB cap; inlined bootstrap snapshot | Size-limit in CI; synthetic fleet per release; RUM p75 alarm |
| 9 | **Cold-account catch-up divergence** | If the lazy path diverges from the continuous path, "the aviary kept running" becomes a lie in a way no user could report precisely. | Total purity of the tick; seeded PRNG keyed to `(bird_id, tick_index)`; pre-scheduled weather | Replay-equivalence test (§13.1) on every build |
| 10 | **Sync/ordering bug losing drift** | Silent partial data loss with no log line. | Additive server-authored deltas only; per-account tick lease; `consumed_by_tick`; `epoch_hi` idempotency; no client-writable trait field exists | Property tests on ordering and idempotency; the ledger-vs-canonical consistency job (#2) also catches lost deltas |
| 11 | **Autoplay policy breaks the "already audible" first frame** | The one place browser policy actively fights the product's central conceit. | Silent, unannounced resume on first interaction with an 800ms mid-call fade-in; captions cover the gap; explanation only in settings | Audio-context state distribution in RUM; synthetic checks assert non-silent output after a simulated interaction |
| 12 | **Magic-link deliverability** | If the link doesn't arrive, there is no password fallback — the user is simply locked out. | Reputable provider, SPF/DKIM/DMARC, dedicated sending domain, warmed IP, plain-text-friendly template, generous re-request rate limit | Delivery/bounce/complaint rate alarms; a synthetic sign-in probe every 15 minutes end-to-end through a real mailbox |
| 13 | **Notebook prose going stale or generic** | The notebook is where the voice is most concentrated; genericness there discredits the voice everywhere. | Template graph with anti-repetition memory; detector diversity; sparsity governor | Weekly sample review during beta; an automated check that the distribution of templates used across accounts is not concentrated above a threshold |
| 14 | **Seven-bird recognizability doesn't hold** | The cap is presented as empirical but has not been measured yet. | Study as a launch gate; ship cap is config-driven so it can launch at 3 or 5 without code changes | The study itself; ongoing internal listening checks as motif libraries change |
| 15 | **PII leakage into observability** | The PRD's named compliance failure mode; impossible to retrofit. | Synthetic UUID everywhere; `Pii<T>` branded type; log scrubber; CI grep; topology separation | Scrubber-hit counter (nonzero = bug); periodic log audit sampling |

---

## 16. Ambiguities resolved — decisions log

The PRD leaves these open. Each is decided here, with the reasoning, so an engineer never has to guess and a reviewer can overturn any of them by pointing at one line.

| # | Ambiguity | Decision | Reasoning |
| --- | --- | --- | --- |
| 1 | Presence activity window ("a few minutes") | **4 minutes** | Leans long, per the PRD's explicit instruction, because watching without moving is the product. Finite because an open laptop in an empty room must not count. |
| 2 | Tick cadence ("~once per minute") | **60s logically; tiered physically with exact replay** (§2.5) | Preserves the observable property exactly while making 1M accounts affordable. The equivalence is a test, not a claim. |
| 3 | Drift rate constant | **α = 0.014/epoch, Δ_max = 0.02** | Derived in §5.2 to satisfy all three stated calibration targets simultaneously. |
| 4 | Drift epoch boundary | **04:00 local time** | A late-night session belongs to the day it began. |
| 5 | Mood set ("exact set finalized in implementation") | **wary, alert, curious, content, drowsy, settled** | The five named in the PRD plus `settled`, which the PRD's night behavior ("eyes closed, low on the perch") requires as a distinct state from `drowsy`. |
| 6 | Offer cooldown ("a few minutes") | **4 minutes per bird** | With `S_offerTake` saturating at 3/day, this makes within-session curiosity saturation unreachable, which is the stated purpose of the cooldown. |
| 7 | Notebook entry rate ("one every few days") | **36h global cooldown, 10-day per-detector refractory, ≤12 entries/30 days** | Tested against a heavy-usage profile to confirm activity yields more moments but not proportionally more. |
| 8 | Narration cadence ("30–60s") | **45s idle; prioritized events ≤1.5s; 20s idle suppression after a prioritized update** | Center of the stated band, with an explicit rule against talking over the user. |
| 9 | Species pool ("about six") | **Six: warbler-like (high, fast), finch-like (mid, bright), wren-like (mid, rapid trill), thrush-like (low, melodic), tit-like (high, staccato), nightjar-like (low, nocturnal)** | Distinct registers and rhythms, chosen for by-ear separability. The nightjar is required by the PRD's night behavior. |
| 10 | Day/night computation | **Timezone offset + date only; no geolocation** | Location is PII we have no reason to hold. A civil-dusk approximation from date and timezone is more than accurate enough for a palette shift. |
| 11 | New-bird age gates | **90 / 210 / 365 / 545 / 730 days** | Matches the PRD's stated feel ("a few months → third bird; a year → five or six") using age only. |
| 12 | Realtime transport | **Polling, no WebSockets** | A 60s tick and a 90s forward window make a 30s poll strictly sufficient; a socket adds failure modes and buys nothing. |
| 13 | Autoplay policy conflict | **Silent resume on first interaction; explanation only in accessibility settings** | Any on-surface unmute prompt is an announcement. Silence plus captions is the honest fallback the PRD already endorses for the WebAudio case. |
| 14 | Wire exposure of traits | **The protocol carries behavior and a quantized palette bucket; trait scalars never leave the server** | "The user never sees the numbers" has to hold with devtools open, or it does not hold. |
| 15 | **Export includes personality vectors vs. never exposing numbers** | **Include them** — the PRD names them explicitly in the export — but as a generated file only: no UI renders them, no client code parses them, and this is the single documented exception to the tainting test (§13.6 item 7). | The two rules genuinely collide; the export rule is more specific and explicit, so it wins. Flagged for product review: bucketing vectors into qualitative descriptions in the export would satisfy both, at the cost of the export's fidelity. Low-cost change either way; recommend an explicit decision before launch. |
| 16 | Drift ledger vs. "never recomputed from event logs" | **Canonical stored vector is the only runtime read; the ledger is an append-only audit/recovery artifact, architecturally excluded from the request path** | The PRD's rule targets runtime derivation; the same PRD names vector loss as the worst failure. A recovery artifact satisfies both, and a CI architecture test keeps it out of the runtime. |
| 17 | Visitor accessibility surfaces | **Visitors get narration, captions, reduced-motion, and keyboard navigation** | The PRD's accessibility stance is about who the product is for, not about which session type they're in. Cost is near zero since the visitor projection reuses the host generator. |
| 18 | Security email to a changed address | **Sent** | The "not a notification surface" rule is about the aviary. An account-security notice is a system surface in the matter-of-fact register, and omitting it would be a security defect. |
| 19 | Bird cap at launch | **`MAX_BIRDS_CEILING` starts at 3, ramps to 7 as the recognizability study and perf hold** | Seven is presented as empirical; until it's measured, launching at the measured number and ramping is the honest sequence. |
| 20 | Offer reaction latency | **Client plays the reaction immediately from a server-supplied `offer_disposition`; the tick records the authoritative event** | A round-trip delay on a gesture would read as lag; a client-invented outcome would risk divergence. A server-supplied disposition gives instant feedback with zero divergence risk. |

---

## Appendix A — Repository layout

```
apps/
  web/                     # the client
    boot/                  # inline-critical: canvas, decoder, first draw
    render/                # scene graph, presenters (motion | crossfade), species geometry
    audio/                 # graph, grammar realization, mixing, captions
    chrome/                # top bar, notebook, settings (Preact)
  edge/                    # aviary-edge: HTML + inlined bootstrap snapshot
services/
  aviary-api/  events-api/  auth-svc/  visits-api/  sim-worker/  mailer-svc/
packages/
  sim/                     # PURE: drift, mood, perch, scheduling, weather, detectors
  call-grammar/            # PURE: motif libraries, transforms, realize(); shared client/server
  voice-kernel/            # PURE: lexicon, templates, anti-repetition, catalogs
  protocol/                # wire types; the ONLY place snapshot/event shapes are defined
  telemetry/               # metrics client with the closed AllowedDims union
tools/
  time-warp/               # months-in-seconds harness
  observatory/             # drift calibration profiles + trajectory artifacts
  lints/                   # voice, no-gamification, no-trait-numbers, catalog-boundary, pii
```

`packages/sim`, `packages/call-grammar`, and `packages/voice-kernel` have a hard dependency rule enforced in CI: **no I/O, no clock access, no unseeded randomness.** That single rule is what makes the golden replays, the time-warp harness, the drift observatory, the caption/audio equivalence, and the cold-account catch-up all work.

## Appendix B — Environments

| Env | Purpose | Data |
| --- | --- | --- |
| `dev` | Local; sim runs in-process with a fast tick (1s) and time-warp available | Seeded synthetic |
| `staging` | Full topology including the telemetry boundary test | Synthetic only, never production copies |
| `beta` | Internal + closed beta accounts | Real, under the same privacy rules |
| `prod` | — | Real |

Production data is never copied to a lower environment. Debugging a specific account is done through the drift ledger and structured logs keyed by account UUID, not by cloning the account's data elsewhere.
