# Pocket Aviary v1 — Implementation Plan

This document is the executable plan for shipping Pocket Aviary v1 to the open web. It is written for a frontier engineering team that has read the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`) and needs to know what to build, in what order, with which boundaries.

The plan deliberately treats the affective constraints from the PRD as engineering constraints. "Notice, never announce," "presence is real interaction," "drift is monotonic toward expressive," "personality is server-authored," and the seven-bird cap are all encoded into the architecture below — they are not stylistic notes that get translated by code review, they are properties the system has whether or not a contributor has internalized them.

Where the PRD leaves a knob unspecified, this plan picks a defensible value and says so in section 1.3. Where the PRD names an absolute rule, the plan honors it and routes around the developer instinct that would naturally violate it.

---

## 0. How to read this plan

- **Section 1 (Scope)** — what's in v1 and what isn't, including the non-goals as runtime invariants.
- **Section 2 (Architecture)** — the service topology, client/server split, and stack picks.
- **Section 3 (Data model)** — Postgres tables, personality vector encoding, event log shape.
- **Section 4 (API surface)** — REST endpoints, snapshot shape, event submission, visit invitations.
- **Section 5 (Simulation engine)** — the server tick, drift filter, mood transitions, call grammar runtime.
- **Section 6 (Sync model)** — how a single canonical aviary reaches multiple devices, why last-write-wins is unreachable.
- **Section 7 (Frontend render)** — scene composition, idle motion, the loading conceit, reduced-motion as a designed surface.
- **Section 8 (Audio)** — procedural call synthesis, chorus mixing, listen-in ramping, the WebAudio fallback.
- **Section 9 (Accessibility)** — screen-reader narration as a first-class surface, captions, keyboard nav, contrast.
- **Section 10 (Performance and observability)** — bundle budget, time-to-first-bird, runtime budgets, the telemetry that exists and the telemetry that explicitly doesn't.
- **Section 11 (Rollout)** — milestones from spike to GA, ramp policy for bird count, day-one instrumentation.
- **Section 12 (Risks)** — the failure modes most likely to silently corrupt the product, with the mitigation each one drives in earlier sections.
- **Section 13 (Open questions)** — items that need a designer, scientist, or stakeholder call before shipping.

A reader can skim sections 1, 2, and 11 to understand the shape of the work. A reader who needs to start a sprint should read 3–10 in full.

---

## 1. Scope

### 1.1 In scope (v1)

- **Single-user accounts** with email magic-link sign-in. Per-account synthetic UUID. Per-device session tokens, revocable. Email change with verify-then-switch.
- **Single canonical aviary per account**, two starter birds at adoption, capped at seven birds, third-and-beyond gated by aviary age.
- **Bird engine**: hidden personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), mood enum (wary, content, curious, drowsy, alert, settled), bird-to-bird interaction, mood persists across sessions.
- **Server-side simulation tick** at ~60s cadence, advancing canonical state regardless of client connectivity.
- **Drift function** monotonic toward expressive, low-pass filtered over presence-time and interaction signals, calibrated so a typical bird shows instrument-detectable drift after ~7 days and user-detectable drift after ~21 days of regular visits.
- **Procedural call synthesis** client-side via WebAudio, motif-grammar driven, mood-and-personality shaped, with a real chorus mechanic (two birds calling produce real interaction, not stacked loops).
- **Single horizontal scene** with three perch zones (front/middle/back), no panning or scrolling, day/night driven by user local time, ambient weather (rain, wind), ambient leaf/feather drift.
- **Top bar** with account, accessibility, field notebook, and offer affordance; fades after a few seconds of cursor stillness.
- **Return-greeting** anchor moment, varying by absence length, bird boldness, and bird mood; staggered across multiple greeters.
- **Listen-in** as a slow audio re-balance (not a mute), disengageable by re-click, focus elsewhere, or empty-aviary click.
- **Offer** flow (seed, song fragment, still pool of water) with per-bird cooldown and mood/personality-shaped reactions.
- **Settle** soft session-end gesture, with five-second undo.
- **Field notebook**: auto-generated naturalist prose, sparse cadence (roughly one entry per few days for a regular user), read-only, infinite scrollback.
- **Visits**: per-invite opt-in invitations to read-only ambient aviary view, revocable, no co-presence, no chat, no avatars, no notifications by default (opt-in toggle).
- **Visit log** in account settings.
- **Multi-device sync** as a property of the architecture (no separate sync feature).
- **Account export** (JSON, on-demand, emailed download link).
- **Account deletion**: soft for 30 days, then hard.
- **Accessibility**: screen-reader narration written in the same naturalist voice as the rest of the product; reduced-motion as a designed alternative rendering (cross-fades between still poses, not "animations off"); call captions generated from the procedural call grammar; full keyboard navigation; WCAG AA contrast.
- **Performance**: initial JS bundle ≤2MB gzipped, first-bird-visible <500ms on a mid-tier 4G mobile device, 60fps idle motion on a five-year-old laptop, no memory growth over 30 minutes (verified in CI).
- **Browser support**: last two major versions of Chrome, Safari, Firefox, Edge.

### 1.2 Out of scope (v1)

The non-goals from `non_goals.md` are encoded as runtime invariants, not just absences:

- **No gamification of any kind.** No streak counter. No "days visited." No calendar dot grid. No badges, achievements, levels, scores, ranks, tiers, XP, "birds adopted: 2" counter. The notebook may not write entries about user-frequency ("you visited every day this week"); that surface is forbidden at the prompt template level (section 5.6).
- **No Tamagotchi mechanics.** Birds do not die. Birds do not get hungry. Drift never moves toward less-expressive. There is no health bar, no happiness meter, no neglect surface.
- **No social network surfaces.** No profiles, follows, public feeds, leaderboards, discovery, comments, ratings, or featured aviaries. No metric is computed across accounts that could later be exposed (the absence is enforced at the data-pipeline level: no cross-account aggregation reads the simulation database).
- **No native apps.** Web only. The data model and protocols do not anticipate native clients.
- **No "Welcome back" toast or banner.** No textual welcome anywhere on return. The bird greeting is the entire welcome.
- **No "your friend visited" notification by default.** Opt-in, off out-of-the-box, never surfaced during onboarding.
- **No recorded audio fallback path.** WebAudio unavailable → graceful silence with captions on by default.
- **No personality numbers exposed to the user.** No stats panel, no debug view, no settings toggle, ever.
- **No client-side personality writes.** The client never writes personality state; only the simulation tick does.
- **No SSO, no password auth.** Magic link only at v1.
- **No customizable scenes, no shared aviaries, no multi-aviary accounts, no payments.**

### 1.3 Defensible calls made by this plan

The PRD leaves several knobs to implementation. This plan picks values and notes them so reviewers can argue with them rather than discover them.

| Knob | PRD says | This plan picks | Why |
| --- | --- | --- | --- |
| Tick cadence | "~once per minute" | 60s default; configurable to 30s under load-test | Round number, easy reasoning about durations, 60s is well within the affective tolerance for "the aviary is continuing" |
| Presence activity window | "a few minutes; lean longer" | 5 minutes since last pointermove/keypress | Balances "watching without moving" with "user walked away" |
| Snapshot keepalive cadence | unspecified | 30s while visible, on visibility change, on suspend resume | Cheap, keeps multi-device coherent without flooding |
| Notebook entry cadence | "roughly one entry every few days" | Target 1 entry / 4 days for a regularly-visited aviary; jitter ±2 days; sparsity guaranteed by per-account rate limit (max 3 entries / 7 days) | Sparse enough not to become a feed |
| Bird species pool | "about six species" | Six at v1: a wren, a warbler, a finch, a thrush, a chickadee, a nightjar (the night-active one) | Six gives diversity without diluting recognizability |
| Aviary-age pacing for new birds | "match deepening relationship" | First new bird offer at 6 weeks of aviary age; subsequent offers every 8–10 weeks; cap at 7 | Months not days; refuses gamification pacing |
| Personality trait scale | scalar, normalized small range | `[0.0, 1.0]` floats, stored as `numeric(5,4)` in Postgres | Bounded, instrument-friendly, easy clamping |
| Notebook prose generation | naturalist voice, specific | Server-side template engine with a controlled grammar over named state slots; *not* an LLM at v1 | Cost, latency, content moderation, determinism; keeps voice consistent |
| Mood enum | "wary, content, curious, drowsy, alert; finalize in implementation" | These five plus `settled` (the night/end-of-session state) | `settled` is named in `concepts.md` for the aviary; reuse for consistency |
| Drift filter | "low-pass over presence-and-interaction" | EMA per trait, time-constant ~72h on presence input, ~24h on listen-in/offer, with hard upper bound of `1.0` | EMA is the simplest filter that gives the calibration shape; constants are the seed for tuning |
| Tick implementation | server-side | Postgres-backed worker pool reading account shards via row-level lock; idempotent on event-log offset | Boring and correct; no Kafka at v1 |
| Snapshot delivery | small payload from edge | HTML + bundle + initial state snapshot delivered together via CDN edge with `Cache-Control: private, must-revalidate` | Minimizes round-trips for TTFB |
| Render pipeline | unspecified | 2D Canvas with sprite-atlas, layered DOM-free; WebGL is a v2 option behind a flag if profiling demands it | Canvas2D meets the budgets and is easier to keep accessible |
| Frontend framework | unspecified | Preact + signals; ~5KB framework cost; React-compatible JSX for ergonomics | Hits the 2MB budget with room to spare |
| State sync transport | unspecified | HTTP polling for snapshots; no WebSocket at v1; SSE used only for visit-revocation push | Avoids stateful infra; aligns with "client pulls" model in the PRD |
| Auth token | per-device session | JWT (short-lived 1h) + opaque refresh token (rotatable, server-revocable, 30-day) | Standard pattern, revocation list small |
| Background job tooling | unspecified | A single Postgres-LISTEN/NOTIFY-based job runner for tick scheduling; cron parity worker as cold-start backup | Boring infrastructure, no Redis at v1 |

These are starting points; section 12 tracks the calibration risk for the ones that affect felt-aliveness directly (drift filter, presence window, notebook cadence, listen-in ramp time).

---

## 2. Architecture

### 2.1 Service topology

```
                                  +----------------------+
                                  |   CDN (edge-cached)  |
                                  |  HTML + JS + initial |
                                  |   state snapshot     |
                                  +----------+-----------+
                                             |
                                             v
+------------------+       +-----------+    +----------------+
|   Browser tab    |<----->|  API tier |<-->|   Postgres     |
|  (Preact + WA)   |  TLS  |  (Node)   |    |  (single       |
|  - render        |       |  stateless|    |   primary,     |
|  - audio         |       |           |    |   logical      |
|  - presence      |       +-----+-----+    |   replication) |
+------------------+             ^          +-------+--------+
                                 |                  ^
                                 | LISTEN/NOTIFY    | row updates
                                 |                  |
                          +------+----------+       |
                          | Simulation      +-------+
                          | worker pool     |
                          | (Node, N=2 v1)  |
                          | - tick scheduler|
                          | - tick executor |
                          | - notebook      |
                          | - mailer queue  |
                          +-----------------+
                                  ^
                                  | SES / SMTP
                                  v
                          +-----------------+
                          |  Email provider |
                          |  (magic links,  |
                          |  visit invites, |
                          |  exports)       |
                          +-----------------+
```

There are exactly three components on the server side at v1: an API tier, a simulation worker pool, and Postgres. There is no Redis, no Kafka, no separate cache, no separate analytics store. The simplicity is part of the design — fewer moving parts means fewer ways to silently corrupt the personality vector, which is the worst possible failure mode.

### 2.2 Client/server boundary

The boundary is a single, sharp rule with engineering teeth:

> **The server is the only writer of canonical aviary state. The client renders snapshots and submits append-only events. Under no code path does a client write a personality vector value.**

This rule shows up everywhere in this plan (§3.2, §4.2, §5, §6) and is the implementation of the "no last-write-wins for personality" rule from the PRD. The boundary is enforced both at the API layer (the API service rejects any payload containing `personality.*` fields outside the simulation worker's internal endpoint) and at the database layer (a Postgres row-level policy revokes UPDATE on `bird.personality_*` columns for every role except the simulation worker's role).

### 2.3 Tech stack

- **Language (server)**: TypeScript on Node 22, single binary per service. TypeScript over Go because the simulation logic involves a lot of small numerical functions where the iteration speed of TypeScript and the type sharing with the client matter; Go was considered for the worker but ruled out by the desire to share the personality math (drift filter, mood transitions) between the test harness and production unchanged.
- **Language (client)**: TypeScript + Preact 10 + `@preact/signals`. Preact for bundle size; Signals for the small reactive state graph the top bar and notebook need.
- **Render (client)**: Canvas 2D for the aviary scene, WebGL deferred. The scene is small (one screen, ~7 birds, ambient ornaments) and the per-frame draw call count is low; the modern Canvas 2D path on a 5-year-old mid-range laptop hits 60fps comfortably.
- **Audio (client)**: WebAudio with a custom synthesis graph (oscillators + biquad filters + small wavetables). No third-party audio library at v1 — the synthesis is hand-rolled because we have to own the chorus mix.
- **Database**: Postgres 16. JSONB for personality history payloads where they need to be inspected; numeric columns for the live trait values. Logical replication enabled for read replicas later; single primary at v1.
- **Auth**: server-issued JWTs (RS256, 1-hour TTL) plus opaque refresh tokens stored in a `session` table; magic-link tokens stored as `pending_signin` rows with a 15-minute TTL.
- **Email**: SES with a fallback SMTP route via SendGrid; magic-link, visit-invitation, account-export emails. No marketing emails ever; no transactional email beyond these three paths.
- **CDN**: CloudFront in front of the API origin for the static bundle and the HTML; the initial state snapshot is delivered via the same edge with a short-lived signed URL embedded in the HTML.
- **Observability**: OpenTelemetry traces; Prometheus metrics; logs in structured JSON to a centralized destination. Per-account fields are filtered out at the OTel SDK layer (§10.5).
- **CI/CD**: Builds run on push; deploy via a release branch with a manual approval. Performance budget enforcement and the no-memory-growth test are CI gates (not warnings).

### 2.4 Why not these

- **Why not WebSocket / persistent server push for sync?** Snapshots are cheap (~3KB) and a 30s polling cadence with on-visibility-change pulls is sufficient for the felt-coherence we need across devices. WebSockets add server-state, reconnection logic, and a class of test scenarios that don't carry their weight at v1. Visit-revocation uses SSE (one-way, idempotent) precisely to avoid bidirectional state.
- **Why not Kafka for the event log?** Postgres is enough at v1 volume (single account event rate is sub-1Hz; aggregate is a few kHz at full rollout). The event log is `append-only` semantically but lives in Postgres `interaction_event` table. Re-evaluate at 100k accounts.
- **Why not Redis for snapshot caching?** The snapshot is regenerated cheaply on demand (a single Postgres read of an account's state). A Redis cache is an optimization we don't need yet; it also adds a class of stale-snapshot bugs that hurt felt-coherence.
- **Why not LLM for notebook prose at v1?** A controlled grammar generator over named state slots is deterministic, cheap, and consistent in voice. LLM costs and latency are real, content moderation is a real concern, and a small grammar with good slot phrasing produces prose indistinguishable from a tasteful field-notebook entry for our domain. Reconsider at v1.5 if entries feel templated; switch is a single-component swap.

---

## 3. Data model

The data model is described in the language Postgres uses. Migrations are forward-only; a schema change is a new migration file, never an in-place edit.

### 3.1 Tables

**`account`**

```
id              uuid primary key default gen_random_uuid()  -- synthetic; never derived from email
email_enc       bytea not null                              -- pgsodium-sealed, decrypted only at sign-in / export
email_hash      bytea not null unique                       -- HMAC-SHA256 with a server pepper; for lookup at sign-in
created_at      timestamptz not null default now()
deleted_at      timestamptz                                 -- soft-delete marker; null = active
hard_delete_at  timestamptz                                 -- 30 days after deleted_at; nightly job processes
settings        jsonb not null default '{}'                 -- a11y prefs, visit-notification toggle, captions, etc.
```

The synthetic `id` is the only identifier used in any other table, log line, telemetry record, or partition key. The `email_enc` column is read at exactly two paths: sign-in and account export. The `email_hash` column exists only to look up the account at sign-in by deterministic hash; the hash uses a server-side pepper so that a database leak does not expose the email-set even to a constant-time lookup attack. Email change verifies the new address via a separate `email_change_pending` row before swapping `email_enc`.

**`session`**

```
id              uuid primary key
account_id      uuid not null references account(id) on delete cascade
refresh_token_h text not null unique     -- argon2id of the opaque refresh token
device_label    text                     -- best-effort UA + IP-derived label; user-visible in settings
created_at      timestamptz not null default now()
last_used_at    timestamptz not null default now()
revoked_at      timestamptz
```

Per-device session, revocable from settings. `device_label` is best-effort and the user can rename it.

**`bird`**

```
id                       uuid primary key default gen_random_uuid()
account_id               uuid not null references account(id) on delete cascade
species                  text not null check (species in ('wren','warbler','finch','thrush','chickadee','nightjar'))
display_name             text not null
adopted_at               timestamptz not null default now()
boldness                 numeric(5,4) not null check (boldness between 0 and 1)
social_warmth            numeric(5,4) not null check (social_warmth between 0 and 1)
vocal_frequency          numeric(5,4) not null check (vocal_frequency between 0 and 1)
plumage_saturation       numeric(5,4) not null check (plumage_saturation between 0 and 1)
curiosity                numeric(5,4) not null check (curiosity between 0 and 1)
mood                     text not null check (mood in ('wary','content','curious','drowsy','alert','settled'))
mood_set_at              timestamptz not null default now()
mood_expires_at          timestamptz                  -- soft; tick may transition before
perch                    text not null check (perch in ('front','middle','back'))
last_call_at             timestamptz                  -- last time engine generated a call event for this bird
order_in_aviary          smallint not null            -- stable sort for rendering and for the cap of 7
```

The bird's identity (`id`) is stable across renames, sync conflicts, server migrations. `display_name` is renameable. The five personality columns are the trait scalars; `mood` is the fast-timescale state. The simulation worker is the only role with `UPDATE` permission on the personality columns, the `mood*` columns, and `perch`; the API tier has `UPDATE` permission only on `display_name`.

**`personality_history`**

```
bird_id      uuid not null references bird(id) on delete cascade
recorded_at  timestamptz not null
boldness     numeric(5,4) not null
social_warmth numeric(5,4) not null
vocal_frequency numeric(5,4) not null
plumage_saturation numeric(5,4) not null
curiosity    numeric(5,4) not null
primary key (bird_id, recorded_at)
```

Snapshot of the personality vector taken on every tick that produces a non-zero delta. Used by the test harness to verify drift calibration (§5.3) and by the export feature. Never exposed to the user as numbers; the data is internal.

**`interaction_event`**

```
id              bigserial primary key
account_id      uuid not null references account(id) on delete cascade
bird_id         uuid                                  -- nullable for non-bird-targeted events
client_id       uuid not null                         -- for idempotency; client generates UUIDv4
event_type      text not null check (event_type in (
                  'presence_ping','listen_in_start','listen_in_end',
                  'offer_seed','offer_song','offer_pool',
                  'settle_start','settle_undo'
                ))
client_time     timestamptz not null                  -- client wall-clock; used for ordering within session
server_time     timestamptz not null default now()    -- authoritative time; ordering across clients
payload         jsonb not null default '{}'
created_at      timestamptz not null default now()
unique (account_id, client_id)                        -- idempotency
```

Append-only by code path (no `UPDATE` or `DELETE` granted to the API tier). Partition by `created_at` monthly; old partitions detach to cold storage at 6 months and become unreadable to the API tier (see §10.6 retention).

**`tick_log`**

```
account_id        uuid not null references account(id) on delete cascade
tick_started_at   timestamptz not null
tick_completed_at timestamptz
events_consumed   bigint not null
status            text not null check (status in ('ok','retry','failed'))
last_event_id     bigint                              -- cursor; tick is idempotent on this offset
primary key (account_id, tick_started_at)
```

The simulation tick records its own progress per account so reruns are idempotent and a worker crash does not double-count events.

**`notebook_entry`**

```
id           uuid primary key default gen_random_uuid()
account_id   uuid not null references account(id) on delete cascade
created_at   timestamptz not null default now()
slots        jsonb not null     -- inputs to the prose generator; recorded for export and re-rendering
prose        text not null      -- the final rendered prose (canonical, what the user sees)
template_id  text not null      -- name of the grammar template used; lets us rev grammar safely
```

`prose` is canonical so a user reading an old entry sees what was written then, even if the grammar evolves.

**`visit`**

```
id                 uuid primary key default gen_random_uuid()
host_account_id    uuid not null references account(id) on delete cascade
visitor_email_enc  bytea not null
visitor_email_hash bytea not null
invite_token_h     text not null unique         -- argon2id of the link token
created_at         timestamptz not null default now()
expires_at         timestamptz not null         -- created_at + 30 days
revoked_at         timestamptz
last_used_at       timestamptz
visit_count        int not null default 0
visit_seconds      int not null default 0       -- aggregate, never per-second; for the host's visit log
```

Visitor email is encrypted at rest; the host sees the email in the visit log when they look at it (it's their invite).

**`account_export`**

```
id           uuid primary key
account_id   uuid not null references account(id) on delete cascade
requested_at timestamptz not null default now()
fulfilled_at timestamptz
download_url text                          -- short-lived signed URL
status       text not null check (status in ('pending','ready','expired'))
```

Exports are emailed; the URL expires in 24 hours.

**`pending_signin`** and **`email_change_pending`** are routine token tables and their shape is omitted here; they store argon2id-hashed tokens and per-row TTLs.

### 3.2 Personality vector encoding

Each trait is a `numeric(5,4)` in `[0.0, 1.0]`. The tick computes a delta `Δ ∈ [0, ε_max]` per tick per trait; clamps so the column never decreases (database trigger enforces `NEW.<trait> >= OLD.<trait>` on the simulation worker's role) and never exceeds `1.0`.

The monotonic-toward-expressive rule is a database-level invariant. A check trigger on `bird` rejects any UPDATE that decreases any of the five personality columns, no matter which role made the call. This is paranoia, and it's right. The trigger also rejects any update where more than one personality column changes by more than `0.01` in a single transaction — a guard against a future code path that batches a session's worth of drift incorrectly.

`mood` and `perch` are unconstrained between transitions; they may move freely. They are not subject to the monotonicity invariant.

### 3.3 Append-only event log invariants

- `INSERT`-only access for the API tier (no `UPDATE` or `DELETE` privilege on `interaction_event`).
- Each event carries a `client_id` UUID generated by the browser; the unique index `(account_id, client_id)` makes retried submissions idempotent. A client that submits the same event twice (network retry, page refresh mid-submit) produces one row.
- `server_time` is authoritative for ordering across clients of the same account. `client_time` is used only for within-session ordering and as a sanity-check input for sync conflict detection.
- The simulation tick consumes events in `(account_id, server_time, id)` order. The worker reads up through the latest committed event, applies them in order, and stamps `tick_log.last_event_id` to that row's `id`.

### 3.4 Soft delete and retention

- `account.deleted_at` set on user-initiated deletion; account is restorable for 30 days. UI shows a matter-of-fact "your account is scheduled for deletion in N days; click here to recover" surface on sign-in during the window.
- Nightly job processes accounts with `now() > hard_delete_at`: removes the account row and cascades through every related table. `interaction_event`, `notebook_entry`, `visit`, `personality_history`, `bird`, `session` are all `ON DELETE CASCADE`. The cascade is deliberate; the privacy commitment in `accounts_sync.md` is that deletion is total.
- Telemetry (separate database, §10.5) does not contain account-correlated rows, so it has nothing to delete on user deletion.

---

## 4. API surface

The API is a small, conservative HTTP+JSON surface. There is no GraphQL at v1; the endpoint count is low enough that GraphQL's flexibility doesn't pay for the cost of a typed schema layer.

All endpoints are versioned at `/v1/*`. All responses use `application/json` with `cache-control: no-store` unless otherwise noted. Errors return RFC 7807 problem-detail JSON. Authenticated endpoints require a `Authorization: Bearer <jwt>` header.

### 4.1 Auth

- **`POST /v1/auth/request-link`** — `{ email }` body. Always returns `204` regardless of whether the email exists (no account-enumeration). Rate-limited per email and per IP. Sends a magic-link email if the email maps to an account.
- **`POST /v1/auth/consume-link`** — `{ token, device_label? }` body. On success returns `{ access_token, refresh_token, account_id }`. Token is single-use; consumption invalidates it.
- **`POST /v1/auth/refresh`** — `{ refresh_token }`. Returns a new access token; rotates the refresh token (the old one is invalidated).
- **`POST /v1/auth/sign-out`** — Revokes the current device session. Returns `204`.
- **`GET /v1/account/sessions`** — Lists per-device sessions for the user; `device_label`, `created_at`, `last_used_at`.
- **`DELETE /v1/account/sessions/{id}`** — Revokes a specific session.

### 4.2 Aviary state

- **`GET /v1/aviary`** — Returns the snapshot:

  ```json
  {
    "snapshot_id": "<ulid>",
    "server_time": "2026-05-09T13:14:00Z",
    "next_keepalive_seconds": 30,
    "day_state": {
      "local_time_iso": "2026-05-09T08:14:00-05:00",
      "phase": "morning",
      "weather": { "kind": "rain", "started_at": "...", "ends_at": "...", "intensity": 0.4 }
    },
    "birds": [
      {
        "id": "...",
        "species": "wren",
        "name": "Pip",
        "perch": "front",
        "mood": "content",
        "render_hint": {
          "pose": "preen",
          "facing": "left",
          "anim_phase_seconds": 12.4,   // for client interpolation continuity
          "plumage_level": 3              // 1..5; mapped from plumage_saturation
        },
        "call_grammar_seed": "1f8a...",   // per-bird seed; client uses it to drive the call generator
        "call_schedule": [
          { "fires_at": "2026-05-09T13:14:08Z", "motif_id": "wren-rise-3" }
        ]
      }
    ]
  }
  ```

  No personality numbers are in the response. `plumage_level` is a quantized 1..5 visual hint derived server-side from `plumage_saturation`, so the client never sees the trait scalar even at the network boundary.

  The snapshot includes a forward window of `call_schedule` entries (typically 30 seconds) so the client can synthesize calls without round-tripping for each call.

- **`POST /v1/aviary/events`** — Submit one or more interaction events. Body:

  ```json
  {
    "events": [
      {
        "client_id": "<uuid>",
        "event_type": "presence_ping",
        "client_time": "...",
        "bird_id": null,
        "payload": {}
      }
    ]
  }
  ```

  Server returns `{ accepted: <count>, deduped: <count> }`. The endpoint never returns personality state; the client polls `GET /v1/aviary` for any updated render.

- **`GET /v1/aviary/notebook?cursor=...&limit=...`** — Paginated notebook entries, newest first. Cursor-based.

- **`POST /v1/aviary/birds/{id}/rename`** — `{ name }` body. Returns the updated bird block.

- **`GET /v1/aviary/narration?since=<snapshot_id>`** — A different rendering of the same state, in naturalist prose for screen readers; see §9. Paced server-side.

### 4.3 Visit invitations

- **`POST /v1/visits`** — `{ visitor_email }`. Creates a row, generates a signed link token, sends the email. Returns the invite metadata (no link in the response).
- **`GET /v1/visits`** — Host's visit log (outstanding + historical).
- **`DELETE /v1/visits/{id}`** — Revokes an invite. Active visitor sessions drop on next snapshot pull (§4.4).
- **`GET /v1/visit/{token}`** — Read-only ambient snapshot for visitor; same shape as `/v1/aviary` but stripped of `call_schedule` writes (visitor cannot affect host) and with a `read_only: true` flag the client honors.
- **SSE: `/v1/visit/{token}/stream`** — One-way push channel that emits `revoked` events when the host revokes mid-visit, so the visitor's session terminates within seconds rather than at the next polling tick.

### 4.4 Account management

- **`GET /v1/account`** — Account settings (a11y prefs, captions toggle, visit-notification toggle, current email).
- **`PATCH /v1/account`** — Partial update; visit-notification toggle, accessibility settings, captions toggle, motion settings.
- **`POST /v1/account/email-change`** — `{ new_email }`. Sends a confirm-link to the new address.
- **`POST /v1/account/email-change/confirm`** — `{ token }`. Swaps email atomically.
- **`POST /v1/account/export`** — Triggers an export. Returns `202`. Email arrives within minutes with a signed download link.
- **`POST /v1/account/delete`** — Marks for deletion (soft).
- **`POST /v1/account/recover`** — Reverses `delete` if within the 30-day window.

### 4.5 Internal endpoints (worker only)

- **`POST /internal/tick/run`** — Triggers a tick for an account. Called by the scheduler. Authenticated by mTLS + a separate JWT issuer.
- **`POST /internal/notebook/generate`** — Generates a notebook entry given current state slots. Internal only.
- **`POST /internal/narration/generate`** — Generates the running prose narration for the screen reader.

These endpoints are not exposed to the public DNS name. They are reachable only on the internal VPC with mTLS.

### 4.6 Errors and the matter-of-fact voice

The API returns RFC 7807 problem JSON. The user-facing error UI maps a small set of `type` values to the matter-of-fact phrasings from `accounts_sync.md`:

| `type` | User sees |
| --- | --- |
| `urn:pa:auth/expired-link` | "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| `urn:pa:auth/session-timeout` | "Your session timed out. Sign in again to keep watching." |
| `urn:pa:state/load-failed` | "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." |
| `urn:pa:visit/revoked` | "This visit is no longer available." |
| `urn:pa:visit/expired` | "This invitation has expired." |
| `urn:pa:browser/unsupported` | "Pocket Aviary needs a recent browser. Try the latest Chrome, Safari, Firefox, or Edge." |

The mapping table is owned by the client; the server returns the `type` and a `detail` for logging. The matter-of-fact phrasings are internationalized in a separate string table; the naturalist-voice strings (notebook prose, narration) are *generated*, not stored as translatable strings (§5.6).

### 4.7 Rate limits and abuse

- `request-link`: 5 requests per email per hour, 50 per IP per hour. Returns 204 always (no enumeration).
- `aviary/events`: 30 events / 10 seconds per session. The aggregate stays well below the realistic event rate; high rates are dropped with a 429 and the client backs off.
- `visits`: 20 invitations per account per day.
- `account/export`: 1 export per account per 24 hours.
- All endpoints behind a global rate limit at the edge (CloudFront → WAF) for crude DoS shedding.

---

## 5. Simulation engine

The simulation engine is the behavioral core. It runs in the worker pool, ticks on a schedule, and is the only writer of personality state. This section specifies its inputs, outputs, scheduler, and the math behind drift, mood, and call grammar.

### 5.1 The tick

A tick for one account does the following, in order:

1. **Acquire a row-lock on `tick_log` for the account** to ensure only one worker ticks any given account at a time. Use `SELECT ... FOR UPDATE SKIP LOCKED` to avoid blocking; another worker simply moves on to a different account.
2. **Read the cursor**: latest `tick_log.last_event_id` for the account.
3. **Read events**: `SELECT * FROM interaction_event WHERE account_id = $1 AND id > $cursor ORDER BY server_time, id` — the new events since the last tick.
4. **Read current state**: birds, current moods, perches, plumage levels.
5. **Compute deltas**:
   - Personality drift per bird (§5.2)
   - Mood transitions per bird (§5.4)
   - Perch reassignments where mood change implies (§5.5)
   - Call schedule for the next snapshot window (§5.7)
6. **Apply updates** in a single transaction:
   - `UPDATE bird` setting new personality scalars (subject to monotonicity trigger)
   - `INSERT INTO personality_history` if any trait moved more than `0.0001`
   - `UPDATE bird` setting new mood, mood timer, perch
   - Generate a new call schedule and store in a small `bird_call_schedule` cache table (cleared and rewritten each tick)
7. **Maybe write a notebook entry** (§5.6)
8. **Stamp `tick_log`** with the highest event id consumed and `tick_completed_at`
9. **Commit**

Tick cadence: every account gets ticked at least every 90 seconds at p99, with the average ~60 seconds. A scheduler enqueues account IDs in round-robin order; workers pop and process. The scheduler also enqueues an account immediately when an event arrives (via Postgres `NOTIFY`) so an interactive moment isn't waiting for the next slot.

When no client has touched an account for a long time, the tick still runs — every account ticks at least once per 5 minutes regardless of activity, so background drift (mood transitioning toward drowsy at dusk, the nightjar starting its calls) advances even for an empty session. The 5-minute idle cadence is calibrated to the visible day/night transition speed.

### 5.2 Personality drift

The drift function is an exponentially-weighted moving average per trait, with input weighted by event type, normalized for elapsed time since the last tick.

Pseudocode:

```
for each trait t in {boldness, warmth, vocal, plumage, curiosity}:
    input_t = 0.0
    for each event e in events:
        input_t += weight_t(e.type) * intensity(e)
    elapsed_seconds = now - last_tick
    α = 1 - exp(-elapsed_seconds / τ_t)
    delta_t = α * input_t * scale_t
    delta_t = max(0, delta_t)  // monotonic clamp
    new_t = min(1.0, current_t + delta_t)
    if new_t > current_t:
        record_history(bird_id, trait_t, new_t)
        bird.t = new_t
```

Where:

- `weight_t(e.type)` is a small matrix encoded once in the engine. Examples:
  - `presence_ping → boldness: 0.0` (presence shifts no specific trait directly), `→ all: small uniform contribution`
  - `listen_in_start (target=this bird): warmth: 1.0, vocal: 0.6` (the user is showing this bird specific attention)
  - `offer_seed (received by this bird): curiosity: 0.8, boldness: 0.3`
  - `offer_song (responded): vocal: 0.7`
  - `offer_pool (used): curiosity: 0.4, plumage: 0.3` (water on plumage is a specific charm)
  - `settle_start: 0` (no specific drift; mood-only)
- `intensity(e)` is normalized: a presence ping contributes `1.0` if the user was present in the last minute and decays; a listen-in is its duration in 30-second units capped at 10 (5 minutes of listening is the saturation point).
- `τ_t` is the time-constant. For traits driven by presence (boldness, curiosity), `τ = 72h`. For traits driven by direct attention (warmth, vocal, plumage), `τ = 24h`.
- `scale_t` is a per-trait calibration factor. Initial seed: all `0.001`. The values get tuned to the calibration target in §5.3.

The function is monotonic by construction: `delta_t = max(0, ...)` and `new_t = min(1.0, ...)`. Database trigger is the belt to the suspenders.

The tick only writes a new personality row if the trait actually moved more than `0.0001`. This prevents history-table bloat for accounts that are mostly idle but still being ticked.

### 5.3 Drift calibration

The PRD names a calibration target: instrument-detectable drift after ~7 days of regular visits, user-detectable drift after ~21 days. We translate that to a numeric target the test harness can verify:

- **Instrument target**: starting from `(0.5, 0.5, 0.5, 0.5, 0.5)` as the initial trait vector, simulate 7 days of "regular visits" — define this as 4 sessions per day, 6 minutes each, alternating mostly-idle presence with a daily listen-in and an offer every other day. The maximum trait should have moved by `>= 0.05` and the minimum by `>= 0.01`.
- **User target**: at 21 days of the same regimen, the maximum trait should have moved by `>= 0.20` (visible plumage step) and at least three traits should have moved by `>= 0.05`.

The test harness simulates a fake event stream and asserts the bands. The calibration knobs (`τ_t`, `scale_t`) are tuned until the simulation hits the targets. The harness runs in CI on every PR that touches the engine so a casual commit can't drift the calibration without showing up as a failure.

A second harness simulates the **neglect case**: 21 days of zero events, starting from a high vector. The drift must produce `delta_t == 0` for every trait (monotonicity). The bird still ages — its mood rotates through the day/night cycle — but its personality values are unchanged. This is the test that catches a regression of "negative drift on neglect" before it reaches a user.

### 5.4 Mood

Mood is a small enum with explicit transition rules. The state machine uses a current-mood × event lookup that nudges the mood probabilistically each tick.

Mood inputs:

- **Recent events in this tick window** (offers accepted → content; alarm-style call from another bird → wary)
- **Time of day in the user's local timezone** — the server stores `account.timezone` (set at first sign-in via the browser's `Intl.DateTimeFormat().resolvedOptions().timeZone`, updateable in settings). Late dusk biases drowsy; dawn biases alert; midday is neutral.
- **Ambient weather** — current rain dampens vocal frequency for the duration of the rain (this is a transient mood mod, not a personality drift).
- **Bird personality** — high boldness reduces the wary transition probability by a factor; high curiosity increases the curious transition probability.

The transitions are:

```
+---------+    user attention      +---------+
|  wary   | --------------------> | content |
+---------+                       +---------+
     ^                                 |
     |  ambient alarm                  | offer accepted
     |                                 v
+---------+                       +---------+
|  alert  | <-------------------- | curious |
+---------+    novel call          +---------+
     |
     |  dusk
     v
+---------+   late night     +----------+
| drowsy  | --------------> |  settled |
+---------+                 +----------+
                                ^
                                | settle_start
                                |
                            (any state)
```

The state machine is encoded as a per-tick sampling: given the current mood and the current inputs (events, time-of-day, weather, personality), each tick the mood may transition with some probability. Probabilities are picked so a content bird stays content for 20–60 minutes on average; mood doesn't flicker every tick.

`settled` is the absorbing state for night; only sunrise + a personality-shaped wake roll moves a bird back to alert/content.

### 5.5 Perch assignment

Perch is mood-and-personality-driven. The selection function:

```
front_score = boldness + 0.3*(mood == 'content' or 'curious')
middle_score = 0.5 + 0.4*(mood == 'alert')
back_score = (1 - boldness) + 0.5*(mood == 'wary' or 'drowsy')
```

The bird chooses a perch each tick by softmax over those scores, with a small bias for staying where it was last tick (to avoid unrealistic hopping). The softmax is parameterized by personality so a bold curious bird is much more likely to be on the front perch in any given tick than a wary timid bird.

The chosen perch is rendered immediately by the client at the next snapshot pull; the client interpolates (a soft fly between perches over ~1.5 seconds) so transitions don't teleport.

### 5.6 Notebook prose generation

A grammar-based generator produces field-notebook entries on a controlled schedule.

**Cadence**:

- Per-account rate limit: maximum 3 entries / 7-day window
- Average target: 1 entry / 4 days for a regularly-visited aviary
- Generation triggered on tick when interesting event detected: first time a bird has greeted before another in the past 7 days, first time vocal frequency has crossed a quantized threshold (visible to the user as a small change in call rhythm), an unusually long quiet stretch followed by a call, a chorus event with three or more birds.
- Sparsity guarantee: if the rate limit would be exceeded, the entry is suppressed and the trigger is *not* deferred. This is a hard floor on noise.

**Generator**:

The generator is a context-free grammar with named slots and a small synonym pool. Templates look like:

```
template "first-greeter":
  "{day_of_week_lc} — {bird_a_name_lc} greeted before {bird_b_name_lc} today, first time this {timeframe_lc}."
template "fluffed":
  "{bird_a_name_lc} is fluffed against the {weather_descriptor_lc}, watching the {perch_lc} perch."
template "long-quiet":
  "a long stretch of quiet this {time_of_day_lc}. {bird_a_name_lc} preened for several minutes without looking up."
template "chorus":
  "the {weather_descriptor_or_time_descriptor_lc} carried three voices this morning — {bird_list}."
```

Slots are filled from the current state (which bird greeted first, current weather, current time of day, perch positions). The synonym pool (`fluffed against {the cool air | the early chill | the evening cool}`) gives the entries variation without falling out of voice. The grammar is implemented as a small TypeScript module; the templates live in source and are reviewed for voice carefully.

**Banned phrases at the template layer**:

- No template may contain `you`, `your`, second person at all (the notebook does not address the user).
- No template may reference visit frequency, streaks, days-since-last-visit, "every day this week," or any pattern over the user's behavior. The notebook makes observations of the aviary, never of the user. The grammar lints templates against a small banned-token list at build time so a future contributor can't quietly add such a template.

**Why grammar, not LLM at v1**: deterministic, fast, voice-controlled, no moderation surface. We can swap to an LLM behind the same internal interface in v1.5 if entries feel too templated. The interface (input slots → entry text) is the same either way.

**Voice continuity**: the grammar shares its phrase pool with the screen-reader narration generator (§9.2) so a screen-reader user moving between the aviary surface and the notebook surface hears the same product.

### 5.7 Call grammar runtime

Calls are generated client-side from a small grammar shipped in the bundle, driven by per-bird seeds and motif IDs from the snapshot's `call_schedule`.

Server side, the simulation tick:

- Computes the next ~30 seconds of call events for each bird based on personality (`vocal_frequency` drives base call rate), mood (`alert` boosts rate, `drowsy` reduces, `settled` silences), and bird-to-bird interaction (a call from bird A may trigger a response from bird B; chorus events emerge naturally from overlapping high-vocal birds).
- Selects motif IDs from the species-specific motif library (each species has 6–10 motifs; mood selects a subset).
- Includes a per-call jitter seed so the client's procedural synthesis varies parameters (timing, pitch, repetition count) without server roundtrip per call.

Client side, the call schedule is consumed by the audio engine (§8). The client never invents calls outside the schedule — the schedule is canonical so two devices viewing the same account at the same time hear the same calls (modulo small floating-point variation in synthesis).

### 5.8 Bird-to-bird interaction

Within a tick, after computing per-bird mood transitions:

- A bird's mood-shift toward `wary` has a probability of spreading to nearby birds (perch-adjacent or recently-interacting).
- A bird's call event in the schedule has a probability of triggering a response in another bird with high `social_warmth`; the response is added to the schedule with a small offset.
- A chorus event is detected when three or more birds have call events overlapping within a 2-second window; the engine boosts the chorus's call durations slightly (a "joining in" effect) and biases the next round of moods toward `content` for the participants.

Bird-to-bird interaction is what keeps the aviary feeling like a small social system rather than parallel NPCs. The engine implements it explicitly because emergent behavior on independent birds doesn't produce real coupling.

### 5.9 Adoption of new birds (aviary-age pacing)

- Eligibility check on each tick: if `account.created_at + threshold` has elapsed AND the user has fewer than 7 birds AND no offer is currently pending, generate a `new_bird_offer` row.
- Thresholds: 6 weeks for offer 3; 14 weeks for offer 4; 22 weeks for offer 5; 30 weeks for offer 6; 38 weeks for offer 7. Each threshold has ±1-week jitter.
- The offer surfaces in the top bar as a small affordance (a single quiet icon) and presents the user with two species the system has selected, with default suggested names. The user accepts or dismisses; dismiss reschedules the offer to ~4 weeks later.
- The offer never pings, never emails, never highlights. It is a quiet affordance the user discovers when they look at the top bar.

### 5.10 Tick performance budget

- Tick wall-time per account: p50 50ms, p99 500ms, alarm at 2s.
- Worker pool sized to handle current account count × tick frequency × p95 wall-time × 2x headroom.
- Per-tick database queries: read events (1 query), read state (1 query), apply updates (1 transaction with N+M statements where N is birds and M is history rows). No N+1 patterns; all bird updates batch in a single statement using `unnest`.

---

## 6. Sync model

Multi-device sync is a property of the architecture, not a feature. Section 5 already describes how this works; this section makes the consequences explicit so a contributor cannot quietly violate them.

### 6.1 Single-writer invariant

The simulation worker is the only role with `UPDATE` privilege on:

- `bird` columns: `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`, `mood`, `mood_set_at`, `mood_expires_at`, `perch`, `last_call_at`
- `personality_history` (insert only, but writeable)
- `notebook_entry` (insert only)

Granted to API tier:

- `bird.display_name`
- `interaction_event` (insert only)
- `account.settings`
- `session.*`
- `visit.*` (host-mediated)

Granted to neither (insert/select only by specific stored procedures):

- `account.email_enc`, `account.email_hash` (sealed/hashed; rotation handled by procedure)

This is enforced at the Postgres role level. Each service uses a distinct database role with the smallest set of privileges it needs. The worker role can update personality columns; the API role cannot. A bug in the API service that tries to write personality fails at the database, not at code review.

### 6.2 Snapshot-pull semantics

Clients pull `GET /v1/aviary` at:

- Initial page load
- `visibilitychange → 'visible'`
- Every 30 seconds while visible (keepalive)
- After a long render-frame gap (laptop suspend/resume; gap >5 seconds)
- After submitting a batch of events that should be reflected (offer accepted, listen-in start, settle)

The snapshot is small (~3KB) so frequent polling is cheap.

### 6.3 No client merge logic

The client does not merge state from another device. There is no "your phone made changes; apply them?" surface, ever. Both devices read from the same canonical record, so there is nothing to merge.

The closest the client comes to handling cross-device state is: when the client pulls a new snapshot and a bird's perch differs from what's currently rendered, the client interpolates the bird from the rendered position to the snapshot position over ~1.5 seconds (the same interpolation used for in-tick perch transitions). The user sees the bird move; they do not see a "synced from another device" surface.

### 6.4 Event ordering across devices

Events from two devices for the same account are ordered by `server_time` (the API's `now()` at insert). The simulation worker consumes them in that order. There is no client-side timestamp that wins over the server.

If two events arrive within the same Postgres transaction (truly simultaneous), they are ordered by `id` (the bigserial). Because `id` is monotonic per partition and the `unique (account_id, client_id)` index dedupes retries, the ordering is deterministic.

### 6.5 Mid-air collision avoidance for `display_name`

Renaming a bird is a `PUT` of a single field; if both devices rename Pip simultaneously, the second `PUT` wins. This is acceptable: name is user-controlled, the user knows what they typed last, and the worst case is a 1-second stale name on one device. There is no snapshot-merge complexity for display name.

### 6.6 Visit revocation propagation

When a host revokes a visit, the `visit.revoked_at` is set. The visitor's client polls `GET /v1/visit/{token}` (visitor side); on next pull (or via the SSE channel within seconds), the response is `403` with the `urn:pa:visit/revoked` problem type. The visitor sees the matter-of-fact surface and the visit closes.

The revocation propagates to existing visit sessions within ~2 seconds when SSE is connected, within ~30 seconds when SSE is unavailable.

---

## 7. Frontend rendering pipeline

The frontend is a Preact + signals application that boots into the aviary scene as the first render, with no spinner, no fade-in, no entry animation.

### 7.1 Entry sequence

The HTML response from the edge contains:

1. The HTML skeleton with a single `<canvas>` element and a `<div>` for the top bar.
2. An inline `<style>` for the calm-field background color (matches the day-state from the server).
3. A small inline `<script>` (~5KB) that knows how to render a single first-frame Canvas drawing of the aviary with one or two birds at their snapshot positions, statically — no animation engine yet, just one draw call.
4. A `<link rel="modulepreload">` for the main bundle.
5. The initial state snapshot, embedded inline as JSON (~3KB).

The inline script reads the snapshot, draws the first frame, and then `import()`s the main bundle. By the time the bundle loads, the user has been seeing birds for a couple hundred milliseconds. The bundle then takes over the canvas, replaces the static frame with the animation engine, and the transition is invisible because the first frame the bundle renders is identical to the static frame.

This is the implementation of "the aviary appears with motion already in progress." The static first frame is not a load state — it's the aviary. The animation starts when the engine boots.

For slower connections (e.g., bundle takes >500ms to arrive after HTML), the static frame is unchanged for that time; the user sees an aviary, just one that isn't animating yet. The calm-field is the design's catch-all for slow paths, never a spinner.

### 7.2 Render loop

The render loop is a single `requestAnimationFrame` loop driven by the engine. Per frame:

1. Compute `delta_t` since last frame.
2. For each bird, run its current animation state machine forward by `delta_t`.
3. Run ambient ornaments forward (leaves drifting, weather effects).
4. Draw the scene in three layers:
   - Background layer: sky color (day-state-driven), distant foliage, soft parallax shadow.
   - Middle layer: perches + birds.
   - Foreground layer: occasional leaves, weather overlay (rain particles or wind blur).
5. Submit the canvas frame.

Frame cost target: <8ms on a 5-year-old mid-range laptop, leaving headroom for the audio context, JS GC, and the UI thread.

### 7.3 Bird animation state machines

Each bird has a small animation state machine driven by the snapshot's `render_hint` and the bird's mood. States:

- `perched_idle` — the default; subtle weight-shifting and breathing.
- `preen` — runs through preen poses; punctuated by short pauses.
- `tilt_listen` — head tilts toward a sound; triggered by another bird's call event in the schedule.
- `call` — body posture changes during a call (chest rise, beak open); synchronized with the audio engine via the call schedule.
- `flying_to_perch` — interpolation from old perch to new perch, ~1.5s, parametric path.
- `interacting_with_offer` — approaching a seed pile, drinking from the pool, etc.
- `settled` — drowsy posture with feathers fluffed.

Each state has its own timing, with parameters from the bird's personality so two birds in the same state still look slightly different. The state machine is mood-aware: a wary bird in `perched_idle` favors longer scan motions and shorter preens than a content bird.

### 7.4 Reduced-motion mode

Users with `prefers-reduced-motion: reduce` (matched by media query) or who opt in via the accessibility settings get the reduced-motion rendering. This is **not** a fallback; it's a parallel render path designed for charm at slower visual cadence.

In reduced-motion:

- Idle motion is replaced by slow cross-fades (3–5 seconds) between still poses representing the same state machine. A preening bird is rendered in a sequence of preen-poses; each pose holds for several seconds; the transition is a smooth alpha cross-fade.
- `flying_to_perch` becomes a cross-fade between perch-A and perch-B over the same ~1.5 seconds, with no in-between flight path.
- Ambient leaf drift is removed entirely; the foreground becomes still.
- Day-state color changes (sunrise to morning, dusk to evening) remain, slowed slightly so the transitions are visible without ever being abrupt.
- Audio is unchanged. Calls play; chorus mixes; listen-in ramps.

Reduced-motion is implemented as a per-state alternative renderer: each bird-animation state has a `motion: 'full' | 'reduced'` branch, and a top-level flag selects. The branch is selected at render time, not at boot — so a user toggling the accessibility setting mid-session transitions smoothly.

### 7.5 Top bar

The top bar is a Preact component above the canvas. It contains four icons: account, accessibility, notebook, offer. The bar is `position: absolute; top: 0; opacity: 1` by default; after 4 seconds of cursor stillness and no keyboard activity, opacity drops to `0.15` over 800ms. On mousemove, keypress, or focus event, opacity returns to `1` over 200ms. The fade is done via CSS opacity, not display:none, so the elements are still focusable when the user reaches them by Tab.

### 7.6 Accessibility tree

The aviary canvas has a parallel accessibility tree for screen readers (§9). The canvas itself has `role="img"` with a dynamic `aria-label` summarizing the current scene (a single-sentence current-state summary). Below the canvas, an `aria-live="polite"` region renders the running narration prose, paced by the server at one update per 30–60 seconds at idle (faster on user-initiated events).

Each bird is **not** an individual accessibility node; the prose narration treats them as named characters. The reason: making each bird an individual focusable node creates a list-of-things UI for screen readers, which is the announcement-style accessibility we are explicitly avoiding. The narration produces the affective surface; bird focus is reachable via the keyboard-mode listen-in path (§9.4) but that is a different surface.

### 7.7 Asset pipeline

Bird visuals are SVGs produced by the visual designer, exported as a single sprite atlas at build time. Each species has:

- 5 plumage levels (mapped from `plumage_saturation` quantized to 1..5)
- ~12 pose frames per state (preen, scan, tilt, call, fly-out, fly-in, settled)

The atlas is post-processed at build to a single PNG with positional metadata. Atlas size budget: 600KB compressed. Loading is single-fetch from CDN with a year-long cache header (filename hashed).

Sprite frames are drawn with per-bird tinting from the `plumage_level` (a small color shift in HSL space), so the same atlas serves all five plumage levels per species without 5× the assets.

### 7.8 Day/night and weather rendering

Day-state is a function of the server-provided `day_state.local_time_iso` and `day_state.phase`. The client renders the sky color, the sun/moon position, and the ambient lighting tint as continuous interpolations between four anchor points (sunrise, midday, dusk, midnight). Transitions between phases happen gradually; the user looking up two minutes apart at dusk sees a slightly warmer hue.

Weather is rendered as a particle system overlay: rain is tiny vertical streaks (50–200 active particles depending on intensity); wind is a CSS-filter ripple on the foliage layer. Weather events are short (typically 5–15 minutes); the server provides start and end times so the client can fade in and out smoothly.

### 7.9 Empty-aviary state

Briefly, between the end of the adoption flow and the first bird arriving in the scene, the canvas renders the calm field. The first bird then enters with a fly-in to its starting perch. After this moment, the scene is never empty again at v1 (deletion isn't a thing; the cap is 7 down to 2).

### 7.10 Ambient ornaments

Leaves, feathers, and the occasional small dust mote drift through the scene at random intervals. These are pure rendering ornaments — no per-leaf state ever leaves the client. They are generated client-side at idle cadence (every 8–30 seconds, jittered) and live for a few seconds before fading. They are skipped in reduced-motion mode.

---

## 8. Audio pipeline

Audio is the affective spine of Pocket Aviary, and the procedural-call rule is non-negotiable. The audio engine runs in a single shared `AudioContext`, with a graph that supports per-bird buses, the listen-in mix, and the chorus mechanic.

### 8.1 Audio graph

```
                                   +-------------------+
                                   |   Master gain     |
                                   +---------+---------+
                                             |
                            +----------------+----------------+
                            |                |                 |
                +-----------v---+   +--------v-------+  +------v--------+
                |  Bird bus 1   |   |  Bird bus 2    |  |  Bird bus N   |
                |  (gain/pan)   |   |  (gain/pan)    |  |  (gain/pan)   |
                +-------+-------+   +--------+-------+  +-------+-------+
                        |                    |                  |
                +-------v-------+    +-------v-------+   +------v--------+
                | Synth voice 1 |    | Synth voice 2 |   | Synth voice N |
                +---------------+    +---------------+   +---------------+

                +-------------------+
                | Ambient bed bus   | --> Master gain
                | (low rumble,      |
                |  wind, distant    |
                |  ambient texture) |
                +-------------------+

                +-------------------+
                | Limiter (master)  |
                +-------------------+
```

- One `AudioContext` per page. Resumed on first user gesture (tab focus + click anywhere).
- Each bird has a dedicated **bird bus** with a `GainNode` (mix level) and a `StereoPannerNode` (subtle stereo placement based on perch position).
- Each bird bus accepts one or more synth voices at a time (handles overlapping motifs within one bird).
- A master `DynamicsCompressorNode` (limiter) sits before output to prevent clipping when many birds call at once.
- An **ambient bed** is a low-volume background texture (distant ambient noise, wind in foliage) that runs continuously and gives the silence between calls a quiet floor.

### 8.2 Procedural call synthesis

Each motif is a small program over WebAudio nodes. A typical motif:

```
motif "wren-rise-3":
  for i in 0..2:
    osc = OscillatorNode(type='sine')
    osc.frequency = base_pitch * (1 + 0.04 * i + jitter())
    env = GainNode()
    env.gain.linearRampTo(peak, 0.04)
    env.gain.linearRampTo(0, 0.18)
    biquad = BiquadFilterNode(type='bandpass', f=2000, Q=2)
    osc -> biquad -> env -> bird_bus
    osc.start(t + i * 0.22 + jitter())
    osc.stop(t + i * 0.22 + 0.30)
```

The motif library has 6–10 motifs per species, each with motif-specific oscillator counts, filter shapes, envelope curves, and timing. At runtime, the client fills the motif with:

- The bird's `vocal_frequency`-shaped pitch (a mild scaling of `base_pitch`)
- Mood-shaped envelope (a `wary` bird's call is shorter and quieter; a `content` bird's is longer with a warmer envelope)
- Per-call jitter from the per-bird seed (so the same motif sounds slightly different each play)

Synthesis is done with `OscillatorNode`s (cheap), `BiquadFilterNode`s (also cheap), and small `AudioBufferSourceNode`s for occasional textural elements (a soft chiff, a leaf-rustle in the ambient bed). All nodes are short-lived: created at motif start, disconnected and garbage-collected at motif end. We carefully track that no node holds a long-lived reference; the no-memory-growth test in CI catches regressions.

### 8.3 Chorus mechanic

Two birds calling at overlapping times produce a real chorus because both are running through the synthesis path simultaneously, with their oscillators producing physically-summed audio at the master bus. The summed signal interacts in time and frequency in ways that two recorded loops layered cannot fake.

The simulation's bird-to-bird coupling (§5.8) ensures that overlapping calls happen often enough — when two birds with high `social_warmth` are both alert, their schedules are coupled so one calls in response to the other. Three birds joining produces a chorus event that the engine reinforces by extending durations slightly.

The master limiter prevents clipping when 5+ birds call simultaneously, but the limiter is gentle (slow attack, slow release, knee-soft) so the chorus doesn't pump audibly.

### 8.4 Listen-in mix

Listen-in is implemented as a per-bus gain ramp:

- On listen-in start (target = bird X):
  - Bird X's bus gain ramps from `0.5` (default) to `1.0` over 1500ms (smooth `setTargetAtTime`).
  - All other bird buses ramp from `0.5` to `0.18` over the same 1500ms.
  - Ambient bed gain ramps from `0.4` to `0.25`.
- On listen-in disengage:
  - All ramps reverse over the same 1500ms.

The 1500ms ramp is the listening tempo. Slower ramps make disengage feel sticky; faster ramps make engage feel like a switch. 1500ms is the calibration anchor; A/B test in beta.

**Other birds never go silent during listen-in.** The 0.18 floor is deliberately audible — the user still hears the aviary, just attenuated. This is a non-negotiable part of the affective design (per `interactions.md`); a contributor reaching for `gain = 0` is changing the product.

### 8.5 Per-call scheduling

The client receives `call_schedule` in the snapshot — a 30-second window of upcoming calls per bird. The audio engine schedules each call with the WebAudio `start(t)` API, where `t` is computed against `audioContext.currentTime`. Drift between server time and audio context time is reconciled at each snapshot by re-scheduling the next window relative to the new server time.

Calls that didn't fire (the user hid the tab, the audio context was suspended) are simply lost — they were ambient calls; nothing depends on every one playing. The simulation continues; the next snapshot brings new calls.

### 8.6 Fallback path

If WebAudio is unavailable:

- The audio engine refuses to initialize; sets a `audio_disabled = true` flag.
- The UI shows a small matter-of-fact note in the accessibility settings: "Audio is unavailable in this browser. Captions are on."
- Captions are forced on regardless of user setting (silence + no captions would be worse than silence + captions).
- The aviary plays in graceful silence; no recorded audio path is ever taken.

We instrument WebAudio errors at boot (§10.5) so we know whether this fallback is hitting more users than expected.

### 8.7 No memory growth

Every audio node created in a motif has a tracked lifetime: the motif registers the node in a per-motif set, and on motif end, all nodes in the set are disconnected. Disconnect releases the WebAudio internal references and allows GC.

A 30-minute test in CI runs the audio engine through 5000 calls and 200 chorus events, checking that the heap doesn't grow more than a small floor (~5MB allowance for normal JS heap drift). Above that, the test fails the build.

### 8.8 Pause when tab hidden

When `document.visibilityState === 'hidden'`:

- The audio context is suspended (`audioContext.suspend()`). This stops scheduling and stops audio output.
- The render loop pauses (no `requestAnimationFrame` while hidden).
- Presence pings are not generated (the user is not present per the PRD's three-condition rule).

When the tab returns to visible:

- A fresh snapshot is pulled.
- The audio context is resumed and the new call schedule is loaded.
- The render loop resumes.

The transition is smooth because the simulation has continued server-side (§5.1); the user comes back to the aviary that has been running, not a frozen one.

---

## 9. Accessibility

Accessibility surfaces in Pocket Aviary are designed for charm, not parity-by-checklist. The screen-reader user, the reduced-motion user, the user with audio off — all of them should experience an aviary that *feels alive*. This section specifies how.

### 9.1 Screen-reader narration

The narration is a stream of naturalist prose generated server-side and delivered to the client through `GET /v1/aviary/narration`. The client renders it into an `aria-live="polite"` region. The narration is paced server-side at one prose update per 30–60 seconds at idle; the client polls the endpoint at the same cadence as snapshot pulls (30s) and treats any new prose as a new entry to push to the live region.

Each prose entry is 1–3 sentences in the same voice as the field notebook:

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

The generator is the same template engine as the notebook (§5.6) with a different template set tuned for current-state description rather than retrospective observation. Templates produce narration consistent with what the visual surface shows, generated from the same state.

### 9.2 Narration cadence and event priority

Narration cadence rules:

- Idle: 1 prose update per 30–60s, jittered. Updates only fire when the state has changed enough to be worth describing — a small mood shift in one bird, a perch change, the start of a weather event, a dusk transition.
- User-initiated events get priority bumps:
  - **Return-greeting**: the narration prose for the greeting is generated as soon as the snapshot pull happens; the live region is updated within 2 seconds of the user landing on the page.
  - **Offer**: the narration describes the bird's reaction within 2 seconds of the offer being submitted.
  - **Settle**: the narration describes the lighting shift and the aviary going quiet within 2 seconds of the gesture.
  - **Listen-in**: when the user enters listen-in mode on a bird, a prose update names the focused bird and notes the audio re-balance: "you're listening to pip now. the others are quieter in the background."

The "you're listening" is the one place the narration can use second-person, because the narration is describing a user action they just took; the rest of the surface stays observational.

### 9.3 Captions for calls

Captions appear as small text near the calling bird, fading in and out with the call. Each caption is generated from the procedural call grammar at runtime: when a motif fires, the caption is the human-readable summary of that motif's shape:

```
motif "wren-rise-3" -> caption "a soft three-note rise"
motif "warbler-trill-low" -> caption "a low trill"
motif "thrush-flute-pause-flute" -> caption "a single flute, paused, then again"
```

Captions live in source as a data table mapping motif IDs to natural-language phrasings. The phrasings honor the naturalist voice (lowercase, present tense, specific to the call shape).

Caption rendering:

- Caption text appears in a small label near the bird's beak position, fading in over 200ms when the call starts.
- Caption fades out over 600ms when the call ends.
- Maximum 3 captions visible at once (chorus events with 4+ birds collapse into a single caption: "three voices joining").

### 9.4 Keyboard navigation

- **Tab order**: top bar (account, accessibility, notebook, offer), then aviary canvas, then any open modal.
- **Aviary canvas focus**: pressing Tab from the offer button focuses the canvas. The canvas then enters a "bird-cycle" mode: arrow keys move focus between birds (left/right cycles through the bird list ordered by `order_in_aviary`).
- **Listen-in**: Enter on the focused bird starts listen-in. Escape exits. Focus moves elsewhere → listen-in disengages.
- **Offer**: keyboard shortcut from anywhere in the page (e.g., `o` when focus is in the aviary, but not when typing). The offer affordance opens a small menu (seed, song, pool); arrow keys choose, Enter selects.
- **Settle**: a single keyboard shortcut (`s`) reachable from the aviary canvas.
- **Notebook**: opens with a keyboard shortcut from the top bar (`n`). The notebook is a scrollable list with arrow-key navigation.

Focus indicators are visible against any aviary background (bright morning, dim dusk, dark night) — a soft white-and-dark dual outline that reads against any palette. The visual designer specifies the exact shape; the engineering rule is that the outline is at least 3px wide and uses two-tone contrast.

### 9.5 Reduced-motion for screen readers

A screen-reader user who has not opted into reduced-motion still receives the same narration cadence; reduced-motion is a visual-rendering setting, not a narration setting. They are independent.

A user with both reduced-motion and a screen reader gets the cross-fade visual rendering, the same narration cadence, and the same captioning (if audio is on).

### 9.6 Captions on by default in fallback paths

If WebAudio is unavailable, captions are forced on. This is the only setting the system overrides without explicit opt-in; the alternative is silence with no captions, which is a worse failure than the user noticing the override.

### 9.7 ARIA structure

- `<main role="main">` wraps the aviary surface.
- `<canvas role="img" aria-label="...">` renders the aviary; the `aria-label` is the latest narration prose update (so a screen reader that doesn't read the live region still gets a current description).
- `<div role="status" aria-live="polite">` holds the running narration. Only the latest prose entry is in the DOM; older entries are removed (the screen reader has already read them).
- Top bar items each have `aria-label`s in matter-of-fact voice ("Account settings", "Accessibility settings", "Notebook", "Offer").
- The notebook is a `<section aria-label="Field notebook">` containing a scrollable list of `<article>` elements, each with the date as a `<time>` and the prose as paragraphs.

### 9.8 WCAG AA contrast

All user-copy text passes WCAG AA contrast:

- Top bar icons and labels: contrast ratio ≥ 4.5:1 against the aviary scene at any day-state (the visual designer provides per-state styling rules).
- Caption text: contrast ratio ≥ 4.5:1 against the bird sprite color (handled by a small text shadow / outline mechanic).
- Settings, error, and account surfaces: standard high-contrast UI surfaces, ≥ 4.5:1.

Contrast is enforced at the design-system level; CSS is reviewed against the design tokens and a pre-commit check (axe-core integrated into the test suite) catches regressions.

### 9.9 Internationalization at v1

V1 ships English. The matter-of-fact strings (sign-in, errors, settings) are externalized into a `messages.en.json` for future locales. The naturalist prose (notebook, narration, captions) is generated, not stored as translations — the grammar would need to be re-authored per locale, which is a significant effort. Internationalization of the naturalist surface is a v2 conversation.

Bird names are user-typed, so they pass through unchanged regardless of locale.

### 9.10 First-class accessibility process

- Accessibility lives in the same product team, not a separate "compliance" stream.
- Every new surface passes accessibility review before it ships, with a small checklist that catches the predictable issues (focus indicators, contrast, narration update, keyboard reach).
- Two manual screen-reader passes (NVDA on Windows, VoiceOver on macOS) on every release candidate.
- A small advisory group of users (paid for time; no marketing role) reviews the experience at major milestones and flags failures we would otherwise miss.

---

## 10. Performance budgets and observability

Performance in Pocket Aviary is the bridge between an engineering metric and a felt-aliveness property: above the budget, the user notices the load; below, they don't. The budgets are runtime invariants, not launch-day goals.

### 10.1 Initial JS bundle ≤2MB gzipped

Bundle inventory and budget:

| Component | Estimate (gzipped) | Notes |
| --- | --- | --- |
| Preact + signals | ~5KB | Framework |
| Aviary engine (render, animation, audio) | ~80KB | Hand-rolled |
| Sprite atlas (sprite metadata + decoder) | ~10KB | The atlas itself is a separate PNG, year-cached, not in the JS bundle |
| Procedural call grammar (motif library) | ~25KB | Motif programs |
| Caption phrase table | ~5KB | Per-motif phrasings |
| Scene assets (SVGs not in atlas) | ~20KB | Top bar icons, focus outline shape |
| Narration → DOM glue | ~3KB | A11y live region updates |
| Top bar UI (Preact components) | ~15KB | Account, settings, notebook, offer |
| Notebook viewer | ~10KB | Lazy-loaded; not in critical bundle |
| Visit-host UI (settings tab) | ~10KB | Lazy-loaded; not in critical bundle |
| Sign-in/account surfaces | ~15KB | Lazy-loaded; not in critical bundle |
| **Critical bundle total** | **~163KB** | Well under budget |
| **Total including lazy chunks** | **~400KB** | Well under budget |

The 2MB cap is comfortable. The cap exists to discourage drift and to leave headroom for v1.5 features (e.g., the LLM-narration swap). CI gates every PR on bundle size; a 5KB increase requires a justification line in the PR description.

### 10.2 Time to first bird visible <500ms

On a mid-tier 4G mobile device:

- DNS + TLS + first byte: ~150ms
- HTML + inline snapshot + inline script: 1 round-trip, ~50KB total, ~250ms
- Inline script renders the static first frame: ~30ms
- Total: ~430ms

The static-first-frame trick (§7.1) is what makes this budget achievable. Without it, the bundle (>200KB even when minimal) would gate the first paint at >700ms.

The budget is a CI gate: a synthetic mid-tier-4G test runs on every release; failure fails the build.

### 10.3 60fps idle motion on a 5-year-old laptop

Frame budget per frame: <16ms total, with the engine target of <8ms.

CI runs a 30-minute Playwright session against the engine on the 5-year-old laptop fleet (a small set of provisioned hardware kept for this test) and captures `requestAnimationFrame` timing. The p95 frame time must be <16ms; p99 must be <33ms (one dropped frame is acceptable; sustained drops are not).

### 10.4 No memory growth over 30 minutes

The CI test runs for 30 minutes simulating a typical session: presence pings, listen-in start/end, offers, calls, chorus events. The heap is sampled every 30 seconds. The slope of the heap-vs-time line must be <100KB/min; the absolute heap at 30 minutes must be <50MB.

The test is a hard gate. A regression here would be a real product failure at v1 scale.

### 10.5 Telemetry — what we measure

Aggregate, never per-account:

- HTTP request counts, latencies (p50, p95, p99), error rates, by endpoint.
- Simulation tick latency (p50, p95, p99) and tick failure rate.
- Snapshot pull latency and payload size.
- Browser-side: `Time to First Bird` (synthetic + RUM), frame timing summary (RUM histograms only, no per-session traces).
- Audio errors: count of WebAudio init failures, count of audio-context-suspend events.
- Sign-in success/failure rates (no email or account dimension).
- Visit invitation send/accept rates (count, no host or visitor dimension).
- Notebook-entry generation rate (count per hour, no account dimension).

Telemetry pipeline:

- Frontend → OpenTelemetry SDK with custom processor that strips any `account_id`, `bird_id`, `email`, and `name` fields before export.
- Backend → OpenTelemetry SDK; resource attributes do not include account fields. Span attributes filtered the same way.
- Telemetry destination: a separate observability database (Prometheus + Tempo or vendor equivalent). The simulation database is *never* read by analytics queries; this is enforced by network policy (the analytics service has no network path to the simulation database).

### 10.6 Telemetry — what we deliberately don't measure

- No per-bird interaction logs in telemetry.
- No per-account session-duration histograms with account dimensions; only anonymous aggregate buckets.
- No "how often is feature X used per account" metrics.
- No "engagement" metrics — DAU/MAU/retention as a per-account concept doesn't exist in our telemetry. We may compute aggregate DAU as a single number; we do not compute it per-cohort or per-feature.

The line is: operational health (allowed) vs. user behavior aggregation (not allowed). The simulation database holds the ground truth of user behavior; that database is never queried for analytics.

### 10.7 Synthetic monitoring

A fleet of automated Playwright browsers runs the aviary on a schedule (every 5 minutes) from common geographies (US East, US West, EU West, EU North, Asia Pacific). Each session:

- Signs in with a synthetic test account.
- Loads the aviary and verifies first-bird-render <500ms.
- Submits a test event and verifies the snapshot reflects it.
- Disconnects.

Results feed the same telemetry pipeline. Synthetic test accounts are flagged in the database and excluded from any aggregate computations (test accounts are not the user we are designing for).

### 10.8 Logs

- Server logs are structured JSON with no PII. `account_id` is logged (it's the synthetic UUID, not PII); email, names, prose are not.
- Client errors flow through Sentry with PII-stripping configured. Stack traces and minified-source maps are delivered; user input is not.
- Log retention: 30 days for hot, 90 days for cold, then deletion. Logs are not part of the export; logs are operational not user-facing.

### 10.9 Error budget and SLO

- API availability SLO: 99.9% monthly. Burn rate alerts at 2%/h sustained over 1h.
- Simulation tick latency SLO: p99 < 5 seconds. Alarm at p99 > 5 seconds for 5 minutes sustained.
- First-bird-render: p95 < 750ms (a slightly looser real-user-monitoring bound than the synthetic-CI 500ms gate, accounting for real-network variance). Alarm at p95 > 1.5s for 15 minutes sustained.

---

## 11. Rollout

### 11.1 Milestones

**M0 — Foundations (weeks 1–4)**

- Repo, CI, deploy infrastructure.
- API tier scaffold, Postgres schema (§3), migration tooling.
- Auth surfaces (magic link, sessions, refresh).
- Synthetic test accounts and CI gates for bundle size, first-bird-render, no-memory-growth.

**M1 — Single bird walking around (weeks 5–8)**

- One species (wren). Static personality vector, no drift.
- Render pipeline (Canvas + sprite atlas, basic preen/scan/tilt animations).
- Audio engine with one motif. No chorus yet.
- Snapshot pull, presence ping submission. Server-side mood transitions for one bird.
- Internal milestone: a single wren preens on a perch and calls when the user opens the tab.

**M2 — Two birds and chorus (weeks 9–12)**

- Add second species (warbler). Bird-to-bird interaction (§5.8) so chorus events happen.
- Listen-in mix with the 1500ms ramp.
- Offer flow (seed, song, pool) with mood-shaped reactions.
- Settle gesture with undo.
- First end-to-end demo: two birds, listen-in, offer, settle.

**M3 — Drift and the simulation tick (weeks 13–16)**

- Server-side simulation tick at 60s cadence.
- Drift filter (§5.2) with calibration tests (§5.3).
- Multi-device sync (snapshot polling from a second device shows the same state).
- Personality monotonicity invariant enforced (database trigger; CI test).
- First end-to-end test: a 21-day fast-forward simulated session shows a measurable drift.

**M4 — Notebook and narration (weeks 17–20)**

- Notebook entry generator with grammar templates.
- Screen-reader narration generator (same engine, current-state mode).
- Caption renderer for procedural calls.
- Voice review with a designer; templates polished.

**M5 — Day/night, weather, ambient (weeks 21–24)**

- Day/night cycle driven by user-local time.
- Ambient weather (rain, wind) with mood effects.
- Ambient leaves and feathers.
- Reduced-motion alternative renderer (§7.4).
- Idle motion at 60fps verified on the laptop fleet.

**M6 — Six species and adoption pacing (weeks 25–28)**

- Six species in the pool: wren, warbler, finch, thrush, chickadee, nightjar.
- New-bird offer flow (§5.9) at aviary-age thresholds.
- Adoption flow at sign-up: two starter birds, default suggested names, naming UI.
- Empty-aviary state for the brief window between adoption and first bird arrival.

**M7 — Visits (weeks 29–32)**

- Invitation flow: email send, accept, visit log.
- Visitor render (read-only ambient view).
- Revocation, expiration, and the matter-of-fact visit-revoked surface.
- SSE channel for in-flight revocation.

**M8 — Account features and exports (weeks 33–34)**

- Account settings UI (a11y, captions, motion, visit notifications, sessions).
- Email change with verify-then-switch.
- Account export (JSON, emailed download).
- Account deletion (soft → hard).

**M9 — Beta (weeks 35–38)**

- Closed beta with 100 paid invitees. Twice-weekly office hours, in-product feedback link.
- Calibration adjustments on drift, narration cadence, listen-in ramp, notebook cadence.
- Accessibility audits with the advisory group.

**M10 — GA launch (week 39+)**

- Open registration. Initial daily-active cap to ensure tick latency stays within budget; release the cap as the worker pool scales.
- Day-one observability: first-bird-render p95, tick latency p99, audio init failure rate, sign-in success rate.

### 11.2 Bird-count ramp

The cap is 7 at v1; the in-aviary count grows over aviary age (§5.9). For the first launch:

- All accounts start with 2 birds.
- The new-bird offer logic ships at GA, so accounts created at GA can grow to 3 by 6 weeks of age.
- The cap of 7 is enforced at the database (a check constraint counts birds per account).

### 11.3 Day-one observability

Dashboards exist on day one for:

- API request volume, latency, error rate.
- Simulation tick volume, latency, failure rate.
- First-bird-render p50/p95/p99 by geography.
- Sign-in funnel: link request, link consume, session start.
- Audio init success rate.
- A small "feels-alive" composite — a single number derived from snapshot-pull-success-rate × narration-update-on-time-rate × audio-context-resume-success-rate. The composite is informational, not an SLO; it gives oncall a single glance for "is the affective surface working."

A small set of synthetic visit invitations runs daily (one host account, one visitor account, both synthetic) to exercise the visit path.

### 11.4 Feature flags

A small flag system exists for:

- `tick_cadence_seconds` (default 60).
- `presence_window_minutes` (default 5).
- `narration_idle_min_seconds` / `narration_idle_max_seconds` (default 30/60).
- `notebook_cadence_target_days` (default 4).
- `listen_in_ramp_ms` (default 1500).
- `species_enabled` (default all six).
- `new_bird_offer_thresholds_weeks` (default the schedule in §5.9).

Flags are read from a config table, hot-reloaded by services without restart. The flag values are operational only (no feature on/off toggles for product surfaces; we don't ship features behind flags for "rollout safety" if the feature isn't ready).

### 11.5 Migration policy

- Forward-only migrations; no destructive migrations on shipped tables.
- Personality history is never deleted on a migration except as part of account hard-deletion (the user's drift history is the user's relationship with their birds; we do not lose it casually).
- Migrations that touch the `bird` table go through a four-step process: add new column nullable → backfill → switch reads → switch writes → drop old column (in a separate release after one full week of stability).

---

## 12. Risks

This section enumerates the failure modes most likely to silently corrupt the product, with the mitigations they drive in earlier sections. The pattern is: each risk is a way the product can fail in a way the user can't name, and each mitigation is an architectural property that makes the failure harder.

### 12.1 Drift miscalibration

**Failure mode**: Birds drift too fast (→ Tamagotchi feeling: "my bird changed because I clicked") or too slow (→ "nothing I do matters; this is a screensaver"). Users don't report this in those words; they just leave.

**Mitigations**:

- The instrument-and-user-detectable calibration targets are encoded in CI tests (§5.3). Every PR runs them; a calibration regression fails the build.
- The neglect-no-drift test is also in CI; a regression here is a hard fail.
- The drift filter constants (`τ_t`, `scale_t`) are feature-flagged so we can adjust without a deploy; a beta feedback loop tunes them in M9.
- A small panel of calibration users (paid for time, not marketing) reviews drift at 2 and 6 weeks of use; their qualitative reads anchor the numerical calibration.

### 12.2 Sync incoherence

**Failure mode**: Two devices show different aviary states; or a personality drift from one session is silently lost when another device writes. The user notices their bird "feels different" and can't explain why.

**Mitigations**:

- Server is the only writer of personality (§6.1). Database role privileges enforce it.
- No client merge logic exists — there's nothing to merge.
- Additive deltas in the simulation tick (§5.2). The tick consumes events in `(server_time, id)` order; reorder is impossible.
- A multi-device coherence test in CI: a simulated session with two clients (both submitting events to the same account) verifies the snapshot from each is byte-identical after the tick.

### 12.3 Audio uncanniness

**Failure mode**: Calls sound canned (the procedural variation isn't enough to escape the loop-feeling); or the chorus mechanic produces phase-canceling artifacts; or the listen-in ramp feels like a cut.

**Mitigations**:

- Procedural synthesis with per-call jitter (§8.2). The motif library has 6–10 motifs per species, and each motif fires with random parameter variation.
- The chorus mechanic uses summed oscillators, not stacked recordings (§8.3). Phase canceling is a real concern; the synthesis path is hand-tuned to avoid the worst pairs (we won't put two narrowly-detuned sines in the chorus), and the master limiter is gentle.
- Listen-in ramp duration calibrated in M9 with real users; current 1500ms is the seed.
- A small audio review at every milestone with the audio designer; the engineering team does not own the listening-quality call alone.

### 12.4 Accessibility regression

**Failure mode**: A new feature ships without accessibility surface, and screen-reader users get a degraded experience in that area; or the contrast on a new copy element fails AA without anyone noticing.

**Mitigations**:

- Screen-reader narration is generated from the same state as the visual surface (§9.1); a feature without narration coverage means the narration generator produces a no-op for that surface, which is detectable in the narration-generation test.
- Every new surface passes an accessibility checklist before shipping (§9.10).
- Contrast checks are in pre-commit (axe-core).
- The advisory group reviews the experience at every milestone.

### 12.5 PII leakage

**Failure mode**: Email shows up as an identifier in a log line, a Kafka partition key (we don't have Kafka but might in v2), an OpenTelemetry span attribute, or a dashboard. The leak is silent; nobody notices until a compliance audit.

**Mitigations**:

- Synthetic UUID is the only identifier in any non-account-record location (§3.1). The architectural rule is "email lives in `account.email_enc`; it is read at sign-in and at export only."
- OpenTelemetry SDK has a custom processor that strips any field matching `email`, `name`, or `display_name` (§10.5).
- A periodic check (weekly) grep-scans logs and traces for PII patterns; alerts on hits.
- The privacy rule is in CLAUDE.md-equivalent contributor docs and in code review checklists.

### 12.6 Engagement creep

**Failure mode**: A contributor adds a "harmless" engagement feature — a gentle streak nudge, a quiet calendar in settings, a notebook entry that says "you visited every day this week" — that violates the gamification ban. The product slowly converts to an engagement product.

**Mitigations**:

- The notebook template grammar lints templates against a banned-token list at build time (§5.6). A template containing `you`, `your`, "every day," "streak," etc. fails the build.
- The CLAUDE.md / contributor docs list the gamification ban explicitly with the reasoning (not just "no streaks" but *why* not).
- Code review for any new user-facing surface includes the question "does this announce or notice?" — the principle is named in the review template.
- A quarterly review (formally part of the product process) walks the entire product surface looking for surfaces that have drifted toward announcement style. The review is documented; deletions resulting from it are noted.

### 12.7 Silent personality reset

**Failure mode**: A migration, a sync bug, a deletion-and-recreation in the bird table (e.g., a contributor "fixes" a bird that's in a weird state) silently replaces the bird the user knows with a new bird that has the same name. The user can't name what's wrong; they just feel the bird is different.

**Mitigations**:

- Stable bird `id` invariant (§3.1). The bird's identity is the UUID; renames are name-only.
- No code path in the API tier deletes a bird (delete is reserved to account-deletion cascade). Renames update `display_name` only.
- A test in CI verifies that across a full migration cycle on a representative dataset, no bird IDs change.
- The personality history table is the audit: if a personality vector ever discontinuously moves (delta > 0.5 in a single tick), an alert fires and an oncall investigates before the user notices.

### 12.8 Tick worker outage

**Failure mode**: The simulation worker pool is offline for an extended window. Personality drift stops; mood transitions stop; new bird offers don't fire. Users open the aviary and see frozen birds.

**Mitigations**:

- Worker pool is N=2 at v1 with active-active redundancy; failover is automatic.
- The tick is idempotent on the event-log offset (§5.1), so a worker restart resumes correctly.
- If both workers are offline, the API tier still serves snapshot pulls from the last canonical state (no degradation visible to a single user). A long outage manifests as "the aviary has been unusually still"; the engine catches up on resume by processing accumulated events.
- An alert fires at >2 minutes of zero ticks across the fleet.

### 12.9 Notebook prose feels templated

**Failure mode**: The grammar's variation is too small; users read three notebook entries and recognize the templates. The voice loses charm.

**Mitigations**:

- The grammar has 30+ templates at v1, each with multiple synonym pools; the slot-fill produces enough variation that the same template is unlikely to repeat in any user's notebook within a year.
- Beta users review notebook entries qualitatively in M9; templates that read as canned are revised.
- The internal interface (slots → entry text) supports swapping in an LLM in v1.5 without changing callers.

### 12.10 Cap-of-seven misidentification

**Failure mode**: A future contributor reasons "we have audio mixing improvements; the cap could be 9 now." The cap rises silently; per-bird recognizability collapses; the per-bird relationship the product is selling thins.

**Mitigations**:

- The cap is a database check constraint, not just a config value. Raising it requires a migration.
- The rationale (audio recognizability ceiling) is in `bird_engine.md` and in the migration template's required justification.
- Any cap change requires a paired audio-mix-recognizability test that demonstrates `n+1` birds remain recognizable; without that test, the migration doesn't merge.

### 12.11 Weather event becomes a feature

**Failure mode**: Weather is meant to be quiet ambient; over time, contributors add a thunderstorm event, a snow event, a tornado feature, and weather becomes a thing the user looks for instead of something that happens to the aviary.

**Mitigations**:

- Weather kinds are a fixed enum in the engine: `rain`, `wind`, `none`. Adding a kind is a deliberate design conversation.
- Weather events are short and rare by configuration (1–3 per week per aviary). Tuning these is a calibration knob, not a feature surface.
- Weather is not surfaced in the notebook except as a subordinate descriptor in entries; "the rain came through this morning" is fine, "today's weather: thunderstorm" is not.

### 12.12 Visit feature creep

**Failure mode**: The simple "invite a friend" affordance accretes — a "popular aviaries" surface, a visitor count, a "frequent visitor" status, comments on visits. The visit feature converts into a social network in ten small commits.

**Mitigations**:

- The non-goals in §1.2 include the social-network surfaces explicitly.
- The data model does not include any cross-account aggregation that would feed a "popular aviaries" surface; adding such a surface requires adding the aggregation, which is a deliberate architectural change.
- The visit revocation, invite expiration, and per-invite-opt-in design make the feature self-limiting at the affordance level.

---

## 13. Open questions

A short list of items that need a stakeholder, designer, or scientist call before the relevant milestone. Each is named here so they don't get quietly resolved by an engineer making a defensible call.

1. **Exact narration cadence in different time-of-day contexts.** Does the cadence speed up at sunrise (a small "the aviary wakes up" moment) or stay constant? Designer + accessibility review.
2. **Narration for users with both `prefers-reduced-motion` and high-contrast modes.** Is the visual fallback for high-contrast different from the default reduced-motion render? Designer + accessibility review.
3. **Bird species visual style across the six species.** The visual designer owns this, but the engineering plan needs the five-plumage-level pose count finalized so the atlas can be sized.
4. **Weather event probability and duration distributions.** Calibration in M5; the seed values in §7.8 are placeholders.
5. **Initial drift constants (`τ_t`, `scale_t`).** Calibration in M3 with synthetic regressions, then in M9 with real-user feedback.
6. **Notebook prose grammar review.** The voice is the most visible product surface; a designer reviews every template before it ships.
7. **Aviary-age thresholds for the new-bird offer.** The values in §5.9 are placeholders; designer + product call before M6.
8. **Whether the narration should describe the listen-in mix re-balance or stay silent on it.** Currently planned to describe it (§9.2); accessibility review may change this.
9. **Privacy policy and terms of service.** Legal review; required for sign-up flow at GA.
10. **Pricing model and billing.** Out of scope for v1 engineering, but the team needs to know whether the product is free at GA or behind a paywall, since it affects the cap on registration ramp.

These resolve in the milestones noted; the plan is robust to changes in any of them.

---

## Appendix A — A reader's check

A reader who has internalized this plan should be able to answer these questions without re-reading:

- Why doesn't the client write personality? *Because last-write-wins on personality silently destroys drift; the server is the only writer.*
- Why is presence three-conjunctive? *Because any laxer definition silently inflates drift across the population.*
- Why no recorded audio fallback? *Because canned audio kills the spell, and silence + captions is a better fallback than that.*
- Why is the tick at 60 seconds, not on every event? *Because drift is a slow filter and tick-per-event would over-react; the slow tick is part of the calibration shape.*
- Why no Kafka? *Because Postgres is enough at v1 volume and adding Kafka adds failure modes that hurt felt-coherence.*
- Why no LLM for the notebook at v1? *Because deterministic, cheap, voice-controlled prose ships; LLM is the v1.5 swap behind the same interface.*
- Why does the server return `plumage_level` and not `plumage_saturation`? *Because the user never sees the personality scalar, even at the network boundary.*
- Why is `display_name` writable by the API tier but `boldness` isn't? *Because name is user-controlled and personality is server-authored.*
- Why is reduced-motion its own designed surface, not a fallback? *Because a stripped fallback would tell the user their preference cost them the product.*

If a future contributor can answer those without reading this plan, the plan has done its job. If they can't, this section is the cheat sheet.
