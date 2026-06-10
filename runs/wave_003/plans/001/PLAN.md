# Pocket Aviary — v1 Implementation Plan

This is the execution plan for Pocket Aviary v1: a browser-based virtual aviary of two-to-seven birds whose personalities drift over weeks in response to the user's presence, with procedural calls, a server-canonical simulation, a field notebook, opt-in read-only visits, and accessibility as a designed surface. The plan interprets the PRD into buildable systems; where the PRD leaves a call open, this plan makes the call and records it in the Decision Ledger (§16).

How to read this plan: §1–§2 fix scope and architecture; §3–§6 specify the server (data, API, simulation, sync); §7–§9 specify the client (rendering, audio, accessibility); §10–§13 are cross-cutting systems (voice, security/privacy, performance/observability, testing); §14–§17 cover rollout, risks, decisions, and staffing. Default numeric values appear throughout; every one of them is a named, server-tunable constant (§5.1.4) unless marked as a hard PRD constraint.

---

## 1. Scope

### 1.1 In scope for v1

- One aviary per account; two starter birds; cap of seven (hard PRD constraint, built into the engine).
- Single horizontal scene, one screen, no pan/zoom/scroll; three perch zones; day/night cycle on the user's local time; rare ambient weather; ambient leaf/feather drift.
- Bird engine: hidden five-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), fast-timescale mood (wary, content, curious, drowsy, alert), monotonic-upward drift driven primarily by presence, procedural per-bird call signatures, bird-to-bird interaction, mood-shaped idle motion.
- Interactions: return-greeting, idle presence (a first-class interaction), listen-in, offer (seed / song fragment / still pool), settle (with 5-second undo), field notebook (read-only, sparse, naturalist prose).
- Accounts: email magic-link sign-in, per-device revocable sessions, email change with verification, JSON export by emailed link, soft deletion (30 days) then hard deletion.
- Sync: server-side simulation tick is the only writer of canonical state; clients pull snapshots and interpolate; clients submit interaction events to an append-only log; multi-device coherence is a property of the architecture, not a feature.
- Social: visit invitations by email — read-only ambient view, per-invite opt-in, revocable, 30-day expiry, silent visit log, opt-in (default off) visit notification toggle. Nothing else.
- Accessibility, shipped with v1, not after: screen-reader narration in naturalist prose, reduced-motion mode as a designed rendering register, runtime-generated call captions, WCAG AA contrast on all user copy, full keyboard navigation.
- Performance, treated as launch gates: initial JS bundle <2MB gzipped; first bird visible <500ms on mid-tier mobile over 4G; 60fps idle motion on a five-year-old mid-range laptop for a 30-minute session; zero client memory growth over 30 minutes (CI-enforced); simulation-tick p99 latency alarm at 5s.
- Browser support: last two major versions of Chrome, Safari, Firefox, Edge; matter-of-fact unsupported-browser surface for older.

### 1.2 Out of scope (and engineered to stay out)

Per `non_goals.md` and the brief: no native apps; no gamification of any kind (no streaks, achievements, levels, scores, badges, counters, green-dot calendars, "you've been here X days" surfaces, anywhere, including settings); no Tamagotchi mechanics (no death, hunger, distress, decaying meters); no social-network surfaces (no profiles, follows, feeds, discovery, comments, leaderboards, co-presence); no push/ping/email notifications about the aviary (the single exception: the off-by-default visit-notification toggle); no payments; no shared or multiple aviaries; no customizable scenes.

Two of these are enforced structurally, not just by review:

- **No leaderboard-able data.** We never compute cross-account aggregates of bird or interaction state (also a privacy rule, §11). The stats a leaderboard would need do not exist in any pipeline.
- **No user-behavior observations.** The notebook/narration generator's detector inputs are aviary-state features only; templates are linted to contain no second-person pronouns and no visit-frequency vocabulary (§10.3).

### 1.3 Product invariants as engineering constraints

The five principles in the brief become these testable rules, referenced throughout:

| Invariant | Enforcement |
|---|---|
| First frame is the aviary mid-motion; no spinner, no entry animation | Boot path budget + visual-regression test of the cold-load sequence (§7.1); loading state is the "quiet field" |
| No announcement surfaces (toasts, banners, welcome text, badges) | Component-library omission (no toast primitive exists in the repo) + copy lint + design review checklist |
| Calls are procedural; no recorded audio anywhere, ever, including fallback | No audio assets allowed in the bundle (CI rule: build fails on `.mp3/.ogg/.wav/.m4a`); WebAudio-unavailable path is silence + captions |
| Personality numbers never rendered in any UI | Snapshot API carries derived behavior parameters, not raw traits (§4.2); the client never receives the five trait values (exception: export file, §16-D7) |
| Naturalist voice on product surfaces; matter-of-fact on system surfaces | Single copy system with a register field per surface; lint + writer-owned golden tests (§10) |
| Drift is monotonic toward expressive; absence never penalized | Property-based tests on the drift function: for all event histories, traits are non-decreasing (§13.2) |

---

## 2. System architecture

### 2.1 Shape

A deliberately small system: **one modular-monolith API service**, **a tick-worker fleet**, **one Postgres database**, **Redis** (scheduling, rate limits, queues), **an edge layer** (static delivery + authenticated snapshot inlining), and **a transactional email adapter**. No microservices at v1; module boundaries inside the monolith are the future seams.

```
                ┌──────────────────────────── Edge (CDN + edge functions + edge KV) ───────────┐
 Browser ◄──────┤  HTML shell + inlined bootstrap snapshot; static assets; session validation  │
   │            └───────────────┬───────────────────────────────────────────────▲──────────────┘
   │ events / API                │ (KV miss / dynamic)                           │ snapshot write-through
   ▼                             ▼                                               │
┌─────────────────────────── API service (modular monolith) ────────────┐   ┌────┴──────────────┐
│ auth/accounts │ aviary state read │ event ingest │ visits │ notebook  │   │  Tick workers      │
│ settings/export/deletion │ email adapter │ admin/config               │   │  (simulation)      │
└──────────────┬─────────────────────────────┬──────────────────────────┘   └────┬───────────────┘
               ▼                             ▼                                   │
        Postgres (canonical state, event log, notebook, invites)  ◄──────────────┘
               ▲                             ▲
        Redis (tick schedule, rate limits, job queues)
```

Modules (one repo, packages): `auth`, `aviary-api`, `sim-engine` (tick), `voice-grammar` (shared with client), `call-grammar` (shared with client), `notebook`, `visits`, `lifecycle` (export/deletion), `email`, `proto` (shared types/schemas), and client packages `renderer`, `perform` (client behavior layer), `audio`, `chrome-ui`, `boot-kernel`.

### 2.2 Authority boundaries — the render-pipeline boundary

This is the most important line in the system. Everything on one side is canonical and slow; everything on the other side is expressive and fast; exactly two artifacts cross it.

- **Server (canonical, slow).** Owns: personality vectors, mood, perch zone, weather, arrival offers, notebook, accounts, invites. The simulation tick (§5) is the **only writer** of personality and mood — no client code path mutates them, ever (hard PRD constraint).
- **Client (performance layer, fast).** Owns: rendering, idle micro-motion, call scheduling and synthesis, greeting execution, interpolation between snapshots, captions, narration delivery. The performance layer is *interpolation in behavior space*: it elaborates the snapshot's derived parameters into moment-to-moment behavior, holds no state beyond the current snapshot window, and has zero authority. Clients never tick.
- **Crossing the boundary, downward:** the **render snapshot** — a small (≤8KB typical, 32KB hard cap) JSON document of positions, moods, derived behavior parameters, call-propensity parameters, weather, active transitions, and seeds.
- **Crossing the boundary, upward:** the **interaction event log** — append-only, idempotent, typed events (presence pings, listen-in start/end, offers, settle, greeting-performed, tz-observed). Clients send observations of user intent; the server decides what they mean (additive deltas; never absolute values — hard PRD constraint).

Resilience property worth naming: because the client performs autonomously from the latest snapshot, a server blip of several minutes is invisible — birds keep moving, calling, and reacting on stale-but-valid parameters. The aviary degrades to "slightly behind," never to "frozen."

### 2.3 Technology choices

- **Language:** TypeScript end-to-end. Rationale: `voice-grammar` and `call-grammar` must run on both server (notebook) and client (captions, narration, synthesis), and shared types across the snapshot/event boundary remove a class of drift bugs.
- **Server:** Node 22 + Fastify; tick workers are plain Node processes consuming a Redis-sorted-set schedule. Postgres 16 (primary store; event log as range-partitioned tables). Redis 7.
- **Edge:** Cloudflare Workers + KV (or functional equivalent; the design uses only "run code at edge, read KV, validate a signed cookie" and is portable).
- **Client:** no framework in the aviary render loop (hand-rolled scene system on **Canvas 2D**, §7.2); **Preact** for chrome surfaces (top bar popovers, settings, notebook panel, auth screens), all code-split out of the boot kernel.
- **Email:** provider-agnostic adapter (Postmark or SES behind one interface).
- **Infra:** containers on a managed orchestrator, multi-AZ, single region at v1; IaC from day one; staging environment that runs the full tick fleet against synthetic accounts.

### 2.4 Why not X (pre-empting the debates)

- **No WebSockets at v1.** The canonical state changes at ~1/minute; polling at tick cadence with jitter, plus refetch-on-visibility and refetch-on-frame-gap, meets every PRD behavior including visit revocation "at the next snapshot pull." Polling is edge-cacheable and removes a stateful fleet. (Ledger D2.)
- **No game engine, no React in the scene.** The scene is ≤7 birds, ≤3 layers, gentle particles. An engine spends our 2MB budget on capabilities we must not use (physics, cameras, tweening UIs).
- **No LLM in the notebook/narration path.** Voice is the product's most concentrated surface; we need deterministic, auditable, writer-owned output at zero marginal latency/cost. A hand-built grammar with a large authored corpus gives us that (§10.2). (Ledger D5.)

---

## 3. Data model

Postgres, UUIDv7 keys. All times UTC `timestamptz`. The five trait values live **only** in `birds` and are never serialized to any client surface except the account-export file (Ledger D7).

```sql
-- Identity. Email exists in exactly one place, encrypted (KMS envelope).
-- email_bidx is a keyed blind index (HMAC-SHA256) used solely for sign-in lookup;
-- it is never logged, never used as a reference, never leaves this table.
accounts(
  id uuid PK,                      -- the synthetic account id; the ONLY cross-system identifier
  email_enc bytea NOT NULL,
  email_bidx bytea UNIQUE NOT NULL,
  settings jsonb NOT NULL DEFAULT '{}',   -- captions, reduced_motion_override, narration,
                                          -- visit_notifications (default false), tz_last_observed
  created_at timestamptz, deleted_at timestamptz NULL, purge_after timestamptz NULL
)

sessions(
  id uuid PK, account_id uuid FK,
  token_hash bytea NOT NULL,       -- SHA-256 of a 256-bit random token
  device_label text,               -- user-agent derived, user-editable
  created_at, last_seen_at, revoked_at NULL
)

magic_links(
  token_hash bytea PK, email_bidx bytea NOT NULL,
  account_id uuid NULL,            -- null until account exists (first sign-in creates account)
  purpose text CHECK (purpose IN ('sign_in','email_change','export_download')),
  expires_at timestamptz NOT NULL, -- 15 minutes for sign_in (PRD)
  consumed_at timestamptz NULL     -- single-use; consumption is an atomic UPDATE ... WHERE consumed_at IS NULL
)

aviaries(
  id uuid PK, account_id uuid UNIQUE FK,
  created_at timestamptz NOT NULL,         -- aviary age; drives arrivals
  tz text NOT NULL DEFAULT 'UTC',          -- last-observed IANA tz; tick uses for time-of-day
  weather jsonb NOT NULL DEFAULT '{}',     -- {kind, started_at, ends_at} | {}
  last_presence_at timestamptz NULL,       -- drives greeting absence-length and dormancy
  tick_seq bigint NOT NULL DEFAULT 0,
  last_event_seq bigint NOT NULL DEFAULT 0,-- event-log consumption cursor
  next_tick_at timestamptz NOT NULL,
  dormancy smallint NOT NULL DEFAULT 0,    -- 0 active(60s) / 1 dormant(30m) / 2 deep(6h); see §5.1.3
  greet_history jsonb NOT NULL DEFAULT '[]'-- rolling 14-day [(date, first_greeter_bird_id)]
)

birds(
  id uuid PK,                       -- stable identity for the life of the account (PRD-invariant)
  aviary_id uuid FK, species_id text NOT NULL,
  name text NOT NULL, adopted_at timestamptz NOT NULL,
  -- personality vector: server-only, [0,1] reals
  boldness real, social_warmth real, vocal_freq real, plumage real, curiosity real,
  mood text CHECK (mood IN ('wary','content','curious','drowsy','alert')),
  mood_state jsonb NOT NULL DEFAULT '{}',  -- dwell timer, pressure accumulators
  perch text CHECK (perch IN ('front','middle','back')),
  signature_seed bigint NOT NULL,          -- fixed at adoption; call-signature identity
  drift_day jsonb NOT NULL DEFAULT '{}',   -- per-day saturating credit accumulators (§5.3)
  offer_cooldowns jsonb NOT NULL DEFAULT '{}'
)

-- Append-only interaction log. Range-partitioned monthly on ingested_at; partitions
-- older than 90 days dropped (vectors are canonical; the log is never replayed — PRD).
events(
  seq bigserial,                    -- per-table monotonic; tick consumes in seq order
  aviary_id uuid NOT NULL,
  client_event_id uuid NOT NULL,    -- idempotency key; UNIQUE (aviary_id, client_event_id)
  session_id uuid NOT NULL,
  type text NOT NULL,               -- presence_ping | listen_in | offer | settle |
                                    -- greeting_performed | tz_observed | adopt | rename
  bird_id uuid NULL,
  at timestamptz NOT NULL,          -- client claim, sanity-clamped to ±5min of ingest
  payload jsonb NOT NULL DEFAULT '{}',
  ingested_at timestamptz NOT NULL DEFAULT now()
)

notebook_entries(
  id uuid PK, aviary_id uuid FK,
  written_at timestamptz, day_label text,   -- "tuesday" style label; no clock times (voice)
  body text NOT NULL,
  detector text NOT NULL,                   -- which observer produced it (internal only)
  dedupe_hash bytea NOT NULL                -- surface-form hash; entries never repeat verbatim
)                                            -- retained for account lifetime; user scroll-back is unbounded

arrivals(  -- the aviary-age-driven new-bird mechanic (§5.9)
  id uuid PK, aviary_id uuid FK, species_id text,
  appears_at timestamptz, expires_at timestamptz,
  status text CHECK (status IN ('lingering','adopted','departed'))
)

invites(
  id uuid PK, aviary_id uuid FK,
  visitor_email_enc bytea NOT NULL,        -- PII: encrypted; shown only in host's visit log
  token_hash bytea UNIQUE NOT NULL,
  created_at, expires_at,                  -- 30 days if unused (PRD)
  consumed_at NULL, revoked_at NULL
)

visit_sessions(
  id uuid PK, invite_id uuid FK,
  started_at, last_seen_at                  -- approximate duration = last_seen - started
)

sim_constants(version int PK, constants jsonb, applied_at, note text)  -- §5.1.4
export_jobs(id, account_id, requested_at, file_key, link_expires_at, status)
deletion_log(account_id, soft_deleted_at, purged_at, verified_at)      -- audit of hard-deletes
```

Species catalog (six species, including one nightjar-like night-active signature) ships as versioned code config, not DB rows: silhouette part-descriptors, default palette ramps, motif library id, activity phase. Stable `species_id` strings.

Derived stores: edge KV `snapshot:{aviary_id}` (written by tick, read by edge bootstrap), `revoked-sessions` KV set (seconds-level propagation; origin enforces strictly).

---

## 4. API surface

HTTPS JSON. Session cookie: httpOnly, Secure, SameSite=Lax; mutating routes additionally require a custom header (`x-aviary-client`) as CSRF belt-and-braces. All requests rate-limited per session and per IP. All system-surface error bodies use matter-of-fact voice and a stable `code`.

### 4.1 Auth and account

| Route | Behavior |
|---|---|
| `POST /auth/magic-link {email}` | Always `202` (no account enumeration). Creates account lazily on first consume. Rate-limited per email (default 5/hour) and per IP. |
| `GET /auth/consume?token=…` | Atomic single-use consume; ≤15 min old; sets session cookie; redirects to `/`. Expired/used → matter-of-fact error page ("We couldn't sign you in. The link may have expired. Try requesting a new link."). |
| `POST /auth/signout` | Revokes current session. |
| `GET /account/sessions` / `DELETE /account/sessions/{id}` | List and revoke device sessions. Revocation propagates to edge KV within seconds; origin checks are authoritative immediately. |
| `PUT /account/email {new_email}` | Sends verification link to the new address; switch commits only on consume; old email works until then. |
| `GET/PUT /account/settings` | Captions, reduced-motion override, narration, visit notifications, sound preference. Device-level `prefers-reduced-motion` always wins over account override when set (§9.2). |
| `POST /account/export` | `202`; async job snapshots aviary state to JSON, emails a 7-day download link to the verified address. |
| `POST /account/delete` / `POST /account/restore` | Soft delete now (sign-in still possible during window; any signed-in page shows the restore affordance); hard purge after 30 days via lifecycle job + verification sweep. |

### 4.2 Aviary state and events

`GET /aviary/snapshot` → the render snapshot. ETag = `tick_seq` (cheap 304s on poll). Shape:

```json
{
  "tick_seq": 48211,
  "server_time": "2026-06-09T14:02:11Z",
  "aviary": {
    "tz": "America/New_York",
    "weather": {"kind": "rain", "ends_at": "2026-06-09T14:04:30Z"},
    "last_presence_at": "2026-06-08T21:40:00Z",
    "settled_capable": true,
    "perf_epoch": 290532121
  },
  "birds": [{
    "id": "b_01HZ…", "name": "pip", "species": "sp_warbler",
    "perch": "front", "pos": {"x": 0.62, "y": 0.41},
    "mood": "curious", "pose_hint": "preen",
    "active_transition": null,
    "behavior": {
      "call_rate_pm": 1.4, "chorus_join": 0.45, "response": 0.5,
      "ornament": 0.3, "approach": 0.7, "greet_weight": 0.55,
      "tilt_rate": 0.6, "scan_rate": 0.2, "fluff": 0.1
    },
    "plumage_render": 0.42,
    "signature_seed": 221387
  }],
  "arrival": null
}
```

Notes with teeth: the five raw traits are **not** in this payload — `behavior` carries derived, quantized parameters (two decimal places) computed by the tick. This keeps the no-numeric-exposure rule enforceable at the API boundary, not just the UI, and blunts third-party "bird stats dashboard" tooling (Ledger D6). `pos` is normalized scene coordinates. `active_transition` carries in-flight perch moves so a freshly opened client renders a bird mid-flight, not teleported.

`POST /aviary/events` → `202`. Batched, idempotent (`client_event_id` dedupe), at-least-once from the client (retry with same ids; `navigator.sendBeacon` on pagehide for the final batch):

```json
{"events": [
  {"id": "01J…", "type": "presence_ping", "at": "…", "payload": {"span_s": 30}},
  {"id": "01J…", "type": "listen_in", "bird_id": "b_…", "at": "…", "payload": {"phase": "start"}},
  {"id": "01J…", "type": "offer", "at": "…", "payload": {"kind": "seed", "anchor": "front_left"}},
  {"id": "01J…", "type": "settle", "at": "…", "payload": {"undone": false}},
  {"id": "01J…", "type": "greeting_performed", "bird_id": "b_…", "at": "…",
   "payload": {"form": "two_note_call", "absence_s": 58800}},
  {"id": "01J…", "type": "tz_observed", "at": "…", "payload": {"tz": "America/New_York"}}
]}
```

Server-side validation: type whitelist, payload schema per type, per-session rate caps (presence pings ≤3/min; offers ≤40/day; listen-in pairs sane), timestamps clamped. A tampered client can only distort its own aviary (the privacy boundary makes this self-harm), and drift inputs saturate anyway (§5.3).

Other product routes: `GET /notebook?cursor=…` (paged, immutable entries); `PUT /birds/{id}/name` (rename any time; no effect on personality/mood/call — PRD); `GET /aviary/arrival` / `POST /aviary/arrival/adopt {name}` (arrival mechanic, §5.9).

### 4.3 Visits

| Route | Behavior |
|---|---|
| `POST /visits/invites {email}` | Creates invite, emails one-time link. Off-by-default feature; nothing in onboarding points at it. |
| `GET /visits/invites` / `DELETE /visits/invites/{id}` | List / revoke (revocation immediate). |
| `GET /visits/log` | Visit log: visitor email, date, approximate duration, outstanding invites. On demand only; no badges anywhere. |
| `GET /visit/{token}` | Consumes the one-time link **once** to bind a visitor cookie on that browser (forwarded links die after first use — Ledger D9); thereafter serves the visitor page while the invite is live. |
| `GET /visit/{token}/snapshot` | Visitor-scoped snapshot: identical render payload, host's `tz` (visitor sees the host's day/night — PRD), minus `last_presence_at`/`arrival`/anything account-ish. Polls at tick cadence. Revoked/expired → `410` + matter-of-fact surface ("This visit is no longer available."). |

Visitor sessions are a separate token namespace with **no event-ingest capability at the routing layer** — the deny is structural: the visitor service has no route to `POST /aviary/events`, and visitor cookies fail auth on the account API entirely. Visitor watching generates no presence, no drift, no events (PRD).

---

## 5. Simulation engine

### 5.1 The tick

#### 5.1.1 What one tick does

Per aviary, transactionally:

1. Read aviary row, birds, and `events` with `seq > last_event_seq` (in seq order — this ordering is what makes additive deltas safe).
2. Fold events into accumulators: presence seconds (union-bucketed, §5.2), listen-in seconds per bird, offers and reactions, settle, greeting-performed, tz updates.
3. Advance continuous-time processes for elapsed `Δt`: drift integrals (§5.3), mood hazard process (§5.4), weather schedule (§5.5), perch re-selection (§5.6), arrival schedule (§5.9).
4. Run notebook observation detectors; maybe emit ≤1 entry (§5.10).
5. Write: trait values (monotonic check enforced in code *and* by a DB trigger that rejects decreases — defense in depth), mood, perch, weather, `tick_seq+1`, `last_event_seq`, `next_tick_at`.
6. Render and push the derived snapshot to edge KV (`snapshot:{aviary_id}`).

Idempotency: the transaction either fully commits or retries; event consumption is cursor-based, so a crashed tick re-reads the same events and produces the same deltas (fold functions are deterministic; randomness is drawn from a PRNG seeded by `(aviary_id, tick_seq)`).

#### 5.1.2 Scheduling

Redis sorted set keyed by `next_tick_at`; N worker processes claim due aviaries with an atomic pop, sharded by `aviary_id` hash to keep per-aviary ordering. Target tick compute p50 <250ms; p99 alarm at 5s (PRD). A `tick_lag` gauge (now − next_tick_at of the most overdue aviary) alarms before users could notice.

#### 5.1.3 Adaptive cadence with continuous-time equivalence

The PRD requires the aviary to advance whether or not anyone is connected; it does not require burning a CPU core per dormant account. We reconcile these by defining **every time-driven process in continuous time** — drift as closed-form exponential integrals over credited input, mood as a continuous-time Markov process (hazard rates integrated over Δt), weather as a Poisson schedule — so that one 30-minute step is statistically identical to thirty 1-minute steps when the event log is empty (which, for a dormant aviary, it is by definition: no client, no events).

- **Active** (presence within 48h, or any connected client): tick every 60s.
- **Dormant** (>48h): every 30 min.
- **Deep-dormant** (>30d): every 6h.
- **Wake:** any snapshot request or event ingest enqueues an immediate tick and restores 60s cadence — so a returning user's first pull reflects a just-ticked aviary.

Dormant aviaries still accrue notebook-eligible moments (a rain that passed, a quiet stretch, the nightjar calling late) at a sparser cap (≤1 entry/week), so a returning user sees that the aviary continued — which is the product's central conceit doing work while nobody watches.

#### 5.1.4 Tuning constants

All engine constants (rate constants, saturation knees, cadences, cooldowns, weather frequency, notebook budgets) live in a versioned `sim_constants` row read at tick time, with an audited changelog. Tuning changes apply prospectively only; **no migration ever rewrites existing trait vectors** (identity-continuity rule: the bird the user knows is never "recalibrated" out from under them).

### 5.2 Presence accounting

Client side: a presence ping (`span_s: 30`) is sent every 30s **only while all three hold simultaneously** — `document.visibilityState === 'visible'`, `document.hasFocus() === true`, and a `pointermove`/`keydown` (or `touchstart`/`pointerdown`) occurred within the activity window. Activity window default **5 minutes**, calibration range 3–10, leaning long because sitting still and watching *is* the product (PRD). Any condition failing stops pings immediately; settle and pagehide both just stop pings (both are equivalent, unceremonious presence-ends — PRD).

Server side, per tick:

- **Union bucketing.** Presence is credited per wall-clock 30s bucket per aviary, regardless of how many sessions ping in that bucket. Two devices open side-by-side accrue presence at 1×, not 2×. (This closes the multi-device drift-inflation hole the PRD's calibration warning is about.)
- **Gap merging.** Gaps <90s between pings within one session are bridged (a dropped request must not chop presence).
- **Hard cap.** Credited presence ≤ elapsed wall-clock per tick window, structurally.

`last_presence_at` updates on credited presence and feeds greeting absence-length and dormancy.

### 5.3 Drift

Traits `v ∈ [0,1]`, server-only. Per tick, for each trait:

```
v ← v + η_t · I_t · (1 − v)        // exponential approach to 1; monotonic by construction
```

`I_t` is the tick's credited input for that trait, built from saturating accumulators:

- **Presence (dominant input; drifts all traits, weighted).** Daily credit follows diminishing returns: `credit(s) = 1 − exp(−s/τ_p)` with `τ_p = 20 min`, effectively saturating around ~45 min/day. Camping a tab (even legitimately) cannot exceed the design calibration; a laptop left open accrues nothing at all because pings stop (§5.2).
- **Listen-in** (strong attention signal): per-bird minutes → that bird's `social_warmth` and `vocal_freq`, saturating at ~10 min/bird/day.
- **Offers:** a bird *accepting* an offer → small `curiosity` credit; offering near a bird at all → smaller `boldness` credit. Per-bird drift-credit cooldown **4 minutes** (PRD "few minutes"): repeat offers inside the window produce reactions but zero credit, so curiosity cannot saturate within one session (the cooldown is functional, not punitive — PRD).
- **Settle:** ends the presence window cleanly and applies a mood-quieting nudge; **zero drift direction** (PRD).
- **Plumage saturation** drifts on overall sustained attention (presence-weighted), and — like every trait — never moves down.

**Negative drift does not exist in the codebase.** There is no code path, constant, or admin tool that decreases a trait. Neglect produces *ambient* birds because expression-mapping (§5.6, §7.3, §8.3) keys some behaviors to *recent* presence context (greeting eagerness, call rate modulation), not because traits decay. Property test: for every generatable event history, `v(t+1) ≥ v(t)` (§13.2).

**Calibration targets** (PRD, made testable): under the reference cohort "regular visits" (15 min/day, 5 days/week), per-trait Δv ≈ **+0.03 after week 1** (instrument-detectable in the harness) and ≈ **+0.12 after week 3**, where +0.12 is chosen to cross at least one *behavior band*. Behavior bands quantize traits into expression tiers — e.g. boldness perch priors `low: 60/30/10 back/mid/front`, `mid: 35/40/25`, `high: 15/40/45`; greeting-first weights; call-rate multipliers — with ±0.02 hysteresis so a bird doesn't flicker at a boundary. Bands are how drift becomes *visible without being told*: the user notices Pip on the front perch more often, greeting first more often. A single session moves no trait by more than 0.01 (hard clamp per tick window) — "no single session shifts a trait visibly" (PRD).

Calibration is validated **only** in the accelerated simulation harness and on explicit-consent internal dogfood accounts — never from production user aggregates, which the privacy commitment forbids (§11, Ledger D8).

### 5.4 Mood

Five states: `wary, content, curious, drowsy, alert`. Continuous-time Markov machine per bird: each tick integrates transition hazards over Δt with minimum dwell 10–30 min (no flapping at 60s ticks). Hazard modulators:

- **Recent interactions:** accepted offer → toward `content`; song-fragment play → toward `curious` for high-vocal birds; settle → general quieting.
- **Local time of day** (aviary `tz`): toward `drowsy` near dusk, `alert` in early morning; at night most birds enter a `settled/sleeping` *presentation* of drowsy (eyes closed, low posture) — except the nightjar-like species, which inverts its activity phase.
- **Ambient events:** rain damps vocal expression and nudges `alert/wary` slightly; an alarm call shifts *nearby* birds toward `wary` (contagion is canonical here, performed instantly client-side — §5.8).
- **Personality priors:** high boldness lowers `wary` entry hazard; high curiosity raises `curious` dwell.
- **Daily-ish reset, not a snap:** a circadian regression term pulls mood toward a personality-conditioned baseline, strongest in the user's early-morning hours. Mood persists across sessions and never visibly snaps on tab-open (PRD): the client only ever sees mood that the continuously-advancing server already had.

### 5.5 Weather

Server-canonical per aviary (both of a user's devices — and any visitor — see the same rain): Poisson schedule averaging 2–4 events/week; rain 90–240s, soft wind 60–180s; never assertive, no thunderstorms, no snow (PRD). Active weather rides the snapshot with `ends_at`; mood effects per §5.4; render effects per §7.4.

### 5.6 Perch selection

Each tick, per bird, sample perch zone from the boldness-band prior, modulated by mood (wary → back-shifted; curious/alert → forward-shifted), social warmth (probability mass toward perching near a warm bird's zone), and inertia (stay-put bias; birds shouldn't churn zones every minute). A zone change emits an `active_transition` (flight path + duration) in the snapshot so every client renders the same move. Perch is a signal the user reads, never a control they hold (PRD: no arranging birds).

### 5.7 Greeting

The return-greeting must land within 1–2s of tab-open — faster than any server round-trip chain — so it executes client-side from snapshot data, deterministically:

- Snapshot carries `last_presence_at` and per-bird `greet_weight` (derived from boldness, social warmth, mood, plus a recency-fairness term so the same bird doesn't *always* greet first unless its personality really says so).
- Client computes `absence = now − last_presence_at`. Tiers: `<10 min` → glance tier (look up from preening, small head-turn); `10 min–6h` → a quiet two-note call and a look; `6h–48h` → re-orientation tier (step toward front perch, longer call, possible second-bird response); `>48h` → fullest tier, unhurried (a bird coming closer, calling longer).
- Greeter selected by weighted draw seeded with `(perf_epoch, floor(now/10min))` — two devices opened near-simultaneously pick the same greeter. Form assembled from the greeting grammar (motif + motion phrase), procedurally varied — never a canned cue, never identical twice (PRD).
- Multiple would-be greeters stagger 1.5–4s randomized offsets; never a unison chorus on arrival (PRD).
- Client posts `greeting_performed {bird_id, form, absence_s}`; the notebook's first-greeter detector and `greet_history` consume it. Drift weight: none (greeting is the system's behavior, not user input).

And, restated as an absolute because the PRD does: **no textual welcome of any kind, anywhere** — no toast, banner, modal, or "gone X days" copy. The greeting *is* the welcome. The component library will not contain a toast primitive (§1.3).

### 5.8 Bird-to-bird interaction — split across the boundary

Canonical layer (tick): mood contagion (alarm → nearby wary), social-warmth proximity effects in perch choice, chorus *propensity* parameters. Performance layer (client): actual call-and-response timing (response within 2–6s at `behavior.response` probability), chorus emergence when overlapping calls occur, head-tilts toward a calling neighbor. The split keeps within-session liveness instant while the durable consequences flow only through the canonical tick.

### 5.9 Arrivals — bird 3 through 7

Driven by **aviary age only** (not visits, not interactions, not payment — PRD): third bird offered around day 90 (±14d jitter), then every ~90d (±21d) up to seven, which lands "a few months → third bird; a year → five or six" (PRD). Mechanics, designed to honor *notice-never-announce* (Ledger D4):

- At `appears_at`, an `arrival` activates: an unfamiliar bird of a server-chosen species begins lingering at the scene's back edge — present in some sessions, absent in others, rendered slightly tentative. No modal, no badge, no "NEW!".
- The notebook may observe it once: *"an unfamiliar bird has been lingering at the edge of the aviary, watching."*
- Clicking/focusing the unfamiliar bird opens a quiet naturalist surface: *"this one seems inclined to stay."* — with a naming field (default suggestion provided) and an adopt affordance. The user does not pick species from a catalog; this is the bird that arrived (PRD's adoption logic extended past the starters).
- Ignored arrivals depart after ~21 days (`status: departed`); another arrives at the next interval. Nothing is lost or penalized.
- Species selection biases toward the nightjar-like species by arrival 3–4 if the aviary lacks it (gives night a voice; soft choice, Ledger D12).

Starters: server picks two distinct species (seeded random), presented as "the birds that arrived"; user names them (suggestions offered, renameable forever; rename never touches personality, mood, call, or id).

### 5.10 Field notebook generation

Runs inside the tick. **Detectors** (aviary-state inputs only; the line is *observations of the aviary, never of the user* — PRD):

first-greeter change vs trailing week · long-quiet stretch (call-rate percentile) · weather pass + a bird's reaction to it · perch-habit shift (modal zone changed week-over-week) · notable offer reaction (a wary bird approached; first pool bathe) · chorus event (3+ birds overlapping) · arrival lingering / new bird settling in · plumage coming in richer (band crossing, phrased observationally) · nightjar calling late · a greeting form longer than that bird's usual.

**Sparsity controller:** noteworthiness scoring → token bucket (capacity 2, refill 1 per 60h ≈ one entry every 2–4 days for a regular aviary; dormant cap 1/week; per-detector cooldowns, e.g. perch-habit ≤1 per 2 weeks). The notebook is observations, not a feed (PRD).

**Prose:** generated by the shared voice engine (§10.2) — lowercase, present-tense, bird-named, specific; day-name labels, never clock times; `dedupe_hash` guarantees no entry ever repeats verbatim within an aviary. Banned by lint: second person, visit counts, durations, numbers about traits, gamification lexicon. Entries are immutable, never archived, scroll back forever; the notebook is read-only — no edit, delete, or annotate affordances exist (PRD).

---

## 6. Sync model

Mostly already implied by §2.2 and §5 — which is the point: sync is a property of the architecture (PRD), and this section is the checklist that keeps it that way.

1. **One canonical record.** Server-side Postgres rows are the aviary. Both devices (and visitors) read the same record via snapshots; "there is nothing to sync."
2. **Single writer.** Only the tick writes personality/mood/perch/weather. Enforced three ways: code structure (no other module imports the trait-write path), DB trigger rejecting trait decreases or non-tick writers (role-based), and audit (§13.2 invariant checks).
3. **No last-write-wins, structurally.** Clients submit *events*, never state. The tick folds events **in `seq` order** into additive deltas. The PRD's laptop-morning/phone-lunch overwrite scenario is unreachable: both sessions' events land in one log; both fold; nothing overwrites.
4. **Idempotent ingest.** `(aviary_id, client_event_id)` uniqueness + client retry with stable ids = at-least-once delivery, exactly-once effect.
5. **Snapshot freshness:** clients refetch on `visibilitychange→visible`, on render-frame gap >5s (laptop-lid suspend detection), and on a jittered 60–75s poll while visible (ETag/304). Snapshot staleness is bounded by tick cadence + one poll interval.
6. **Reconciliation on the client** is presentation-only: birds glide (or, reduced-motion, cross-fade) from rendered positions to snapshot positions; mood expression blends over ~2s; large divergence (rare) uses a soft scene cross-fade rather than teleporting birds. No client state survives reconciliation except in-flight user gestures.
7. **Clocks:** server time in every snapshot; client maintains an offset and never trusts local wall-clock for sim-relative decisions (greeting absence uses server-relative times).
8. **Conflict surfaces** that remain (auth races, expired links, revoked sessions, outage) all resolve to matter-of-fact system copy (§10.1); the aviary surface itself never shows sync chrome.

---

## 7. Frontend rendering pipeline

### 7.1 Boot — the <500ms first-bird path

Per the PRD, the first frame is the aviary mid-motion: no spinner, no entry animation, no fade-from-static. Budgeted cold-load sequence on the reference device (Moto G-class, 4G, cold cache):

1. **0ms** — Request hits edge. Edge function validates the session cookie *locally* (Ed25519-signed token; no origin round-trip; revocation KV checked best-effort), reads `snapshot:{aviary_id}` from edge KV (written by the tick, so it's ≤1 tick stale), and streams the HTML shell with: inline critical CSS, the **inlined bootstrap snapshot** in a `<script type="application/json">` island, and the boot-kernel `<script>` tag. Target TTFB ≤150ms.
2. **~250–450ms** — Boot kernel (≤150KB gz: scene system, Canvas 2D renderer, bird puppet, pose decode, day/night LUT) parses and executes; first paint draws sky/foliage layers from the local-time palette and **birds at their snapshot positions in their snapshot poses** — one mid-preen, one mid-scan — using a prebaked compact pose atlas per species (≤30KB total WebP) so nothing waits on procedural art. **First bird visible <500ms.**
3. **Immediately after first paint** — idle-motion generators take over from the static poses without a visible seam (generators initialize *from* the pose-hint); greeting executes per §5.7 (the greeting itself is the 1–2s "noticed you" beat); conditional snapshot revalidation fires.
4. **Lazily** — audio engine (~80KB) loads and waits for its gesture unlock (§8.5); voice/caption grammar (~40KB); Preact chrome (top bar popovers, settings, notebook) on first intent; full procedural plumage atlases render in idle time and replace the prebaked atlas invisibly.

Degraded paths: KV miss / signed-out / slow origin → the **quiet field** (soft sky gradient, one or two faint ambient motions, no spinner, no progress text); birds enter via the same fly-in used by the empty-aviary state. Signed-out users get the static sign-in shell (matter-of-fact voice). The cold-load sequence is pinned by a visual-regression + synthetic-throttle CI test (§13.4): the 500ms budget is a launch gate, not an aspiration.

### 7.2 Renderer

**Canvas 2D, layered, no framework in the loop** (Ledger D1):

- Four compositing layers, each its own canvas: **sky/background foliage** (repainted at ~2Hz and on palette LUT changes — day/night drift is minutes-scale), **midplane** (perches + birds; repainted every frame), **particles** (leaves, feathers, rain; pooled sprites), **foreground** (occasional branch/leaf pass). Subtle autonomous parallax between layers (layer drift, *not* cursor-tracking — gimmick avoidance); the scene is explicitly not parallax-heavy.
- `devicePixelRatio` capped at 2; offscreen-canvas caching for static geometry; dirty-region skip when the scene is fully settled at night.
- Frame budget: 16.6ms with ≤8ms main-thread target; a frame-budget monitor drives a **graceful degradation ladder** under sustained jank: drop particle counts → drop parallax → halve background repaint rate. Bird motion degrades last; chrome never gets jank precedence over birds.
- Hidden tab: cancel rAF, suspend audio context, presence pings stop naturally; a lightweight `visibilitychange` listener resumes everything and triggers snapshot refetch (PRD: client stops rendering when hidden; simulation continues server-side).
- WebGL2 remains a documented escape hatch behind the `SceneRenderer` interface if real-device profiling ever fails the 60fps gate; we do not expect to need it for ≤7 birds and light particles.

Responsive: one horizontal scene at every viewport; layout solver keeps all perch zones and **every bird in frame at all times** (no cropping a bird out — PRD); narrow viewports compress inter-perch spacing, wide viewports expand it; aspect handling lives in the layout solver with min/max clamps.

### 7.3 Bird puppet and idle motion

- Each species is a parametric vector descriptor (Path2D part set: body, head, beak, two wings, tail, legs; ~10 transform nodes). Per-bird palette = species ramp interpolated by `plumage_render` (richer saturation/feather detail as plumage drifts up; never down).
- **Idle micro-motion generators**, continuously running, mood-weighted: breath (chest scale ±0.5% at ~0.2Hz, Perlin-jittered), weight-shift shuffle, preen sequences, scene-scanning with saccade pauses, head-tilt toward sound-source vectors (real ones — a neighbor's call, a song-fragment offer), feather fluff (outline expansion), blink. A mood→generator-weight table makes mood legible from motion alone: wary = back-perched, high scan, low preen; content = preen-heavy; curious = tilt/investigate; drowsy = low posture, fluffed, slow breath. **No mood labels, tooltips, or status icons exist** — motion is the only mood surface (PRD).
- Perch transitions: parabolic hop/flight with banking and wing-flap cycle, 600–900ms, from `active_transition` so all clients agree.
- Birds are never still in a way that reads as paused; generator output never hits zero amplitude while awake, and sleeping birds still breathe.

### 7.4 Scene systems

- **Day/night:** palette LUT keyed to user-local time, continuous (dawn warm-up across early hours → brightest midday → warm evening → dim night); calls quiet in the evening via the behavior params; at night most birds render settled (eyes closed, low) while the nightjar species stays active. Night is alive, just quiet.
- **Weather render:** rain = light pooled particle pass + slight palette cool + audio damp; wind = foliage ripple + leaf-drift burst. Reduced-motion renders weather as slowed palette shifts + audio only, no particles.
- **Ambient drift:** leaves/feathers at slow random intervals, client-generated (pure rendering ornament, no sim state — PRD), pooled.
- **Top bar:** the only chrome — account/settings, accessibility, notebook, offer; nothing else, ever (the bar has four slots and no extension point). Fades to ~8% opacity after 4s of cursor stillness; restores on pointer/keyboard activity or when any bar item holds focus (fade is visual only; keyboard reachability is never lost). No UI inside the scene: no buttons, badges, tooltips, overlays, or labels (PRD).
- **Empty-aviary state:** post-adoption quiet field, then the first bird's soft fly-in; never seen again after that.

### 7.5 Reduced-motion mode

A designed rendering register, not a fallback (PRD), active when `prefers-reduced-motion` is set or the account opts in (device signal wins; §9.2):

- The same puppet/pose system runs with a different **motion driver**: micro-motion becomes slow cross-fades between held poses (4–8s intervals — a preen becomes a sequence of preen-poses dissolving into each other); perch changes become ~800ms cross-fades between perches; ambient leaf drift is removed; day/night color shifts remain, slowed.
- Calls play at full quality; captions, narration, notebook, drift, mood — all unchanged. The aviary is the aviary, in a calmer visual register with its own quiet charm.
- Reduced-motion has its own visual-regression suite and ships at launch (§9.5).

### 7.6 Interaction implementations

- **Listen-in:** click/tap/keyboard-focus a bird (Enter). Audio re-balance per §8.4; visually, nothing gamey — at most a subtle attention cue in the focused bird's behavior (it may glance over). Disengage: same bird again, another bird, empty-space click, or focus departure (Esc); slow ramp both ways. Never called solo/select/highlight/pin anywhere, including code identifiers (§10.4).
- **Offer:** from the top-bar affordance only (not by clicking birds — PRD). Choose seed / song fragment (small curated library, 5–7 motifs) / still pool. Seed and pool render at front-scene anchors; pool persists ~12 min with reflective shimmer, then fades; song fragment plays softly through the audio engine as a scheduler stimulus. Reactions are mood × curiosity shaped: curious-content birds approach; wary birds wait, then maybe come near; drowsy birds may not. Affordance soft-disables ~90s after an offer (gesture pacing, no countdown UI, no tooltip-scolding); per-bird drift-credit cooldown 4 min (§5.3).
- **Settle:** top-bar gesture; 4s lighting shift to evening, calls quiet, a small acknowledging response from the birds (a settling posture, one low call); the settled state holds until tab-close or re-engagement. **Undo:** any click in the aviary within 5s reverses it (accidental-click mercy — PRD). Emits `settle`; engine treats it as presence-end + mood-quieting only. Closing the tab without settling is exactly equal at the engine level; no recovery surface, no nudge ever mentions settling.
- **Notebook UI:** top-bar icon → code-split panel over the scene; virtualized infinite scroll (memory rule §12); entries grouped under day labels; read-only; no share/export/annotate affordances.

---

## 8. Audio pipeline

### 8.1 Synthesis engine

All calls synthesized client-side in an **AudioWorklet** (synthesis off the main thread; render loop unaffected): per voice — two oscillators (sine/triangle blend) + light FM for chirp character + filtered noise burst components + biquad bandpass + ADSR, driven by pitch-contour automation from motif specs. **Voice pool of 16, pre-allocated, reused; zero per-call allocation** (the no-memory-growth rule applies to audio first — §12). Worst case (7-bird chorus + ambience) budgeted ≤~5% CPU on the baseline laptop. Graph: per-bird voice chains → per-bird gain → subtle stereo pan by perch x-position → bird bus → gentle limiter → master. A very low procedural ambience bed (filtered air/leaf noise) sits under the calls so "quieting to ambient" lands on something (Ledger D11). No audio asset files exist in the repo (CI-enforced, §1.3).

### 8.2 Call grammar and per-bird signature

- Per-species **motif library**: parametric phrase atoms (contours, trill rates, harmonic stacks, envelopes), authored with a sound designer in a dedicated tuning workbench (§14, M0).
- A **phrase grammar** (weighted FSM over motifs with rhythm rules) generates each call as an AST; nothing is ever played from a stored sequence — every call is a fresh generation (PRD: never identical twice; no loops; real chorus, no phase-cancel artifacts).
- **Signature = identity:** `signature_seed` (fixed at adoption) derives stable motif weights, base pitch offset, and timbre params. Mood and drifted vocal-frequency modulate *rate, loudness, ornamentation density* — never the signature core. Pip's call stays Pip's across mood and drift (PRD recognizability rule; validated by listening tests, §13.5). Renames never touch the seed.
- The call AST is the shared source for **synthesis** and **captioning** (§9.3): the caption describes the call that actually played, by construction.

### 8.3 Scheduler, chorus, bird-to-bird

Client-side scheduler per bird: Poisson-ish call onsets at `call_rate_pm` with refractory periods, modulated by time-of-day and weather damping from the snapshot; response calls at `behavior.response` probability 2–6s after a neighbor's call; chorus joins at `chorus_join` when ≥2 birds are vocal in a window — chorus is emergent overlap of independent procedural voices, which is what makes it a chorus. Deterministic PRNG seeded `(signature_seed, perf_epoch, time-bucket)` keeps two same-account devices in approximate macro-agreement. Song-fragment offers inject a stimulus event: high-vocal birds may join or counter-call; others go quiet and tilt.

### 8.4 Listen-in mix

Per-bird gain automation: engage ramps the focused bird +6dB relative and the others down ~−12dB toward (never to) the ambient floor over **2.0s**; disengage ramps back over **2.5s**. A re-balance, not a mute — the others stay audible by design (PRD: a place, not soloable tracks). Mix state also feeds caption prominence (focused bird's captions slightly larger/longer).

### 8.5 Autoplay reality, and the fallback

Browsers block audio before a user gesture, and the PRD's "calls already audible" on first frame collides with that. The plan (Ledger D3): the aviary opens **visually** fully alive; the AudioWorklet context starts suspended and **resumes on the first qualifying gesture** (pointerdown/keydown/touch — pointermove doesn't qualify per browser policy), with calls fading up over ~2s rather than popping on. No "click to enable sound" banner (announcement); the top bar's accessibility item carries a quiet sound-state indicator, and if audio remains locked after a grace period, **captions render by default** so calls are perceivable before sound unlocks. Returning users with engagement-index autoplay grants get sound immediately where the browser allows it.

**WebAudio unavailable** (old browser, denied context, hardware): graceful silence + captions on by default. There is no recorded-audio fallback — unconditional rule (PRD); silence with captions beats canned audio.

---

## 9. Accessibility surfaces

Accessibility is a designed surface shipped with v1 (milestone M3, before beta — §14); a reduced-motion mode landing post-launch is a launch failure by definition (PRD).

### 9.1 Screen-reader narration

- A single ARIA live region (`aria-live="polite"`, swap-replace to avoid queue flooding) carries **running naturalist prose**, generated client-side by the shared voice engine (§10.2) from the same state the visuals render: *"a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."* Never a state list, never "Wren mood: content," never announcement diction.
- Cadence: one idle update per **45s ±15** (PRD 30–60s). User-initiated events (return-greeting, offer reaction, settle acknowledgment) jump the queue and narrate within 1–2s — still written as observations, not confirmations.
- Chrome surfaces (top bar, settings, notebook, auth) get conventional semantic structure — landmarks, headings, labels — in the appropriate register (naturalist for notebook content, matter-of-fact for system surfaces). Voice continuity between narration and notebook is a tested property (golden suite, §13.6): same product in the ears as in the eyes.

### 9.2 Reduced-motion

Covered in §7.5. Activation logic: OS `prefers-reduced-motion` always wins when set; account-level opt-in covers users whose OS setting is unavailable; the account override syncs across devices via settings.

### 9.3 Call captions

- Opt-in from accessibility settings; **on by default** whenever audio is unavailable or still gesture-locked (§8.5).
- Generated at runtime from the call's AST — *"a soft three-note rise," "a low trill, paused, low trill again"* — via a deterministic descriptor mapping with authored variation; the caption always matches what actually played (PRD: no fixed strings per call).
- Rendered as small text near the calling bird on a translucent scrim chip (the chip is what guarantees AA contrast over arbitrary scene pixels — a deliberate, minimal exception to "no chrome in the scene," Ledger D10); fade in/out 300ms (opacity steps in reduced-motion); per-bird caption queue with collision avoidance.

### 9.4 Keyboard and focus

Exactly per PRD: Tab traverses top-bar items; Tab into the scene focuses the first bird; arrow keys move between birds (left-to-right roving tabindex); **Enter** = listen-in on focused bird; **Escape** = exit listen-in; offer dialog opens from its top-bar item and is fully keyboard-navigable with a focus trap; settle reachable from the top bar. Focus indicator: soft 2px high-contrast outline with a 1px halo, tested against bright midday, dim night, and settled palettes. Top-bar fade never affects focusability (§7.4).

### 9.5 Contrast and CI

All user copy (top bar, settings, account/error surfaces, captions, visually-rendered narration) passes WCAG AA minimum via design tokens; automated contrast checks run against day, night, and settled palettes in CI; axe-core runs on every chrome surface; the manual screen-reader matrix (NVDA/Firefox, JAWS/Chrome, VoiceOver/Safari) is a release-gate checklist per milestone (§13.6).

---

## 10. Voice and copy system

The voice split is load-bearing product structure, so it gets a system, not a style guide.

### 10.1 Registers

One copy pipeline with a mandatory `register` field per surface: **naturalist** (aviary, notebook, narration, captions, offer prompts, arrival surface) — lowercase, present-tense, specific, bird-verbed, no exclamation, no "you"-announcements; **matter-of-fact** (sign-in, sessions, sync errors, account, settings, accessibility settings, unsupported-browser, visit-revoked, export/delete) — normal capitalization, direct, states what happened and what to do. The boundary rule for future surfaces ships in the engineering docs verbatim: anywhere the user engages the system *as a system* — money, identity, errors, settings — drops out of naturalist (PRD).

### 10.2 The voice engine

One shared TypeScript package (`voice-grammar`) generates notebook entries (server), narration (client), and caption descriptors (client): writer-authored template grammars with weighted variants and slot fillers; per-aviary anti-repetition memory (surface-form hashes — no verbatim repeats, ever); rhythm constraints (sentence-length variation, em-dash/comma cadence); lowercase enforcement in the naturalist register. The writer owns the corpus in reviewable files; golden-output tests snapshot a seeded generation run so any phrasing change is a deliberate, reviewed diff (§13.6).

### 10.3 The lint with teeth

CI fails on: gamification lexicon anywhere in product surfaces (`streak, achievement, badge, level, score, XP, rank, unlock, milestone, welcome back, you've been`); second-person pronouns in notebook/narration templates; exclamation marks in naturalist register; uppercase sentence-starts in naturalist register; any copy string not routed through the registered pipeline. The banned-lexicon list is append-only and owned by the writer.

### 10.4 Vocabulary in code

PRD glossary terms are the engineering vocabulary: event types, functions, and UI identifiers say `call`, `listen_in`, `offer`, `settle`, `presence`, `visit` — never `song`, `solo/select/pin`, `gift`, `logout-lite`, `engagement`, `friend-view`. Cheap rule, real effect: teams build the product their identifiers describe.

---

## 11. Security and privacy engineering

- **Synthetic ID rule (non-negotiable, PRD):** every internal reference — DB FKs, queue messages, telemetry, logs, shard keys, KV keys — uses the account UUID. Email is stored once, encrypted (KMS envelope), on `accounts`, plus a keyed blind index used solely for sign-in lookup. Enforcement: log-scrubbing middleware; a CI **canary test** that drives a synthetic account with a known email through sign-in/export/visit flows and asserts the address appears nowhere in logs, telemetry, or error output; schema-registry denylist for telemetry fields (`email`, `bird_id`, `aviary_id`, raw trait names).
- **Magic links:** 256-bit random tokens, SHA-256 at rest, 15-min TTL, atomic single-use consume; per-email and per-IP rate limits; `202` always (no enumeration).
- **Sessions:** 256-bit tokens hashed at rest; Ed25519-signed edge-verifiable wrapper for the bootstrap path; revocable from settings; revocation immediate at origin, seconds-eventual at edge KV.
- **Visitor tokens:** separate namespace; read-only by routing structure (§4.3); single-consumption link binding (Ledger D9).
- **Privacy boundary as pipeline architecture (PRD):** per-bird/per-account interaction state exists **only** in the simulation database and is read **only** by the simulation and that user's own surfaces. The telemetry SDK schema has no per-account dimensions; the analytics store has no connection — network-level — to the sim DB; no ML training path receives per-bird fields (none exists at v1, and the absence is documented as deliberate). Aggregate telemetry is the enumerated operational set in §12 and nothing else. Population-level analysis of bird interaction does not exist, which also structurally forecloses leaderboards (§1.2).
- **Export:** on-demand job → JSON (birds with names/species/adoption dates/current personality vectors/current moods, notebook entries, settings — contents per PRD; Ledger D7) → 7-day signed download link to the verified email; files purged after expiry.
- **Deletion:** soft-delete marks immediately (restore affordance on any signed-in page); lifecycle job hard-purges at 30 days — birds, vectors, events, notebook, invites, visit logs, sessions, KV snapshots, export files — then runs a verification sweep recorded in `deletion_log`. Backup retention is documented ≤30 days so purged data ages out of backups on a stated clock.
- Standard hygiene: SameSite=Lax + custom-header CSRF check, strict CSP (no third-party scripts in the product at all), dependency audit gates, secrets in managed KMS, least-privilege DB roles (the trait-write role belongs to the tick alone), pre-launch external security review (§14).

---

## 12. Performance budgets and observability

### 12.1 Budgets (launch gates, enforced in CI where possible)

| Budget | Value | Enforcement |
|---|---|---|
| Initial JS bundle | **<2MB gz** hard; boot kernel ≤150KB gz target | bundle-size CI gate, fails the build |
| First bird visible | **<500ms** (Moto G-class, 4G, cold cache), p75 | synthetic throttled-emulation CI + scheduled real-device lab runs |
| Idle frame rate | **60fps sustained 30 min** on 5-year-old mid-range laptop | CPU-throttled CI proxy + device-lab soak; frame-time p95 <16.6ms in the worst test scene (7 birds + rain + listen-in + captions) |
| Client memory | **No growth over 30 min** | CI soak: headless 30-min run, forced GC, heap delta <5MB; pools for particles/audio voices/captions; virtualized notebook |
| Tick latency | p50 <250ms; **p99 alarm at 5s** (PRD) | prod alerting + tick-lag gauge |
| Snapshot payload | ≤8KB typical, 32KB cap | schema test |

Bundle discipline that buys the budget: procedural bird art (vector descriptors, not image sets), procedural audio (no audio files — also the rule), aggressive code-splitting (settings/notebook/visits/auth all out of the kernel), Preact not React, no third-party UI/analytics SDKs in the product surface.

### 12.2 What we measure

Synthetic: scripted browser fleet from several geographies on a schedule — cold-load timing breakdown, first-bird-render, frame-rate sample, audio-context error probe, snapshot latency. RUM (aggregate-only, sampled, **no per-account dimensions**): page-load and first-bird timings, frame-time histograms, audio-pipeline error counts, snapshot fetch latency, JS error rates by release. Server: request rates/latencies/errors, tick p50/p99 + lag, event-ingest validity rates, email delivery success, queue depths, anonymized session-duration histograms (no account key). Releases gate on these dashboards staying green for 7 days pre-launch (§14).

### 12.3 What we deliberately don't measure

Named so nobody "helpfully" adds them: retention cohorts, engagement funnels, DAU/streak-shaped metrics, per-account session patterns, visit-frequency distributions, per-bird interaction analytics, population drift distributions (also privacy-forbidden), A/B engagement experiments on the aviary surface. Operational health only. The product's success metrics are qualitative beta signal plus the budgets above, and that is a decision, not an omission.

---

## 13. Testing and quality strategy

1. **Unit/integration:** standard coverage on auth flows (expiry, single-use, replay), event ingest (dedupe, validation, rate caps), visits (authz matrix: visitor cannot POST events, revocation at next pull), lifecycle (export/delete/restore, purge verification).
2. **Simulation property tests** (the engine's spine): monotonic drift under *all* generated event histories; trait bounds [0,1]; per-tick clamp ≤0.01; idempotent re-tick after simulated crash; union-bucketed presence never exceeds wall-clock; adaptive-cadence equivalence (30-min step ≡ thirty 1-min steps for empty logs, statistically, KS-tested); mood dwell minimums; mood-never-snaps (state at session boundaries is continuous).
3. **Calibration harness:** the engine running at ~1000× with synthetic presence cohorts (regular / sporadic / absent / camping / two-device); asserts week-1 instrument-detectability and week-3 band-crossing targets; regression-runs on every constants change. This harness is the *only* calibration instrument besides consenting dogfood accounts (privacy, §11).
4. **Performance CI:** the §12.1 table — bundle gate, throttled cold-load budget, frame-time scene test, 30-min memory soak. Plus the cold-load **visual regression** (first frame must contain birds mid-pose, quiet-field fallback must contain no spinner pixels — yes, really).
5. **Audio:** offline-render unit tests (render a call AST to a buffer; assert pitch contour, envelope, duration); chorus overlap renders without clipping; voice-pool exhaustion behavior; and human **listening protocols** — uncanniness panel each milestone, and the recognizability test: after a simulated two-week exposure script, listeners identify which of 7 birds is calling ≥80% of the time. The 7-bird cap is empirical (PRD); this test is its instrument.
6. **Voice/a11y:** golden-output snapshots of seeded notebook/narration/caption generations, writer-reviewed on diff; the §10.3 lints; axe-core CI; automated AA contrast on day/night/settled palettes; keyboard-nav integration tests; manual SR matrix (NVDA, JAWS, VoiceOver) per release; reduced-motion visual-regression suite.
7. **Security/privacy:** PII canary (§11); authz matrices; log-scrub verification; telemetry schema denylist test; deletion E2E with post-purge sweep.
8. **Chaos/sync:** concurrent two-device event storms with induced retries and out-of-order ingest — assert single-writer invariants, no lost deltas, no duplicate effects; tick-fleet kill/resume mid-transaction.

---

## 14. Rollout

### 14.1 Milestones

- **M0 — Foundations and the two scary spikes (3 wk).** Repo/monorepo, CI skeleton with the budget gates from day one, design tokens, edge bootstrap walking skeleton. **Spike A:** procedural call synthesis workbench with sound designer — kill the uncanniness question early. **Spike B:** cold-load path to <500ms with a stub snapshot. Both spikes are go/no-go gates on their respective architectures.
- **M1 — Vertical slice (4 wk).** One unauthenticated dev aviary: tick + Postgres + snapshot + event log; Canvas renderer + one-species puppet + idle motion; presence accounting; calls + listen-in mix; greeting path. Success: a dev sits with one bird for 10 minutes and it *feels alive* — this is the product hypothesis test, on purpose, before accounts exist.
- **M2 — Accounts and sync (4 wk).** Magic links, sessions, edge auth, multi-device, offers, settle, adoption/starter flow, rename; chaos/sync suite green; six species + perch system + day/night + weather.
- **M3 — Voice and accessibility (4 wk).** Voice engine + notebook (detectors, sparsity, UI); narration; captions; reduced-motion register; keyboard/focus; SR matrix pass; voice lints live. Accessibility ships *here*, pre-beta, by design (PRD).
- **M4 — Visits and lifecycle (3 wk).** Invites/visitor path/revocation/log/notification toggle; export; deletion/restore; unsupported-browser surface; external security review; privacy/telemetry audit.
- **M5 — Calibration and hardening (6+ wk, overlapping M3–M4).** Calibration harness cohorts + employee dogfood (consented) from M2 onward — drift needs weeks of wall-clock, so it starts as early as the engine exists; constants tuned via `sim_constants` changelog; perf budgets green 7 consecutive days; listening tests pass; closed beta (~200 invited accounts) for 4+ weeks; then open launch.

Indicative total: ~24–28 weeks to open launch.

### 14.2 Ramping birds-per-aviary

Natural ramp by construction: every account starts at two; arrivals unlock by aviary age (~day 90 for bird 3), so launch-day load is 2-bird aviaries and the population grows into 5–7 over the first year. Before the first arrival cohort lands, run the 5- and 7-bird recognizability listening tests (M5) and load-test the snapshot/tick path at 7; the cap stays 7 and is engine-enforced (arrivals simply stop scheduling).

### 14.3 Launch posture

Closed beta by invitation (matter-of-fact email, no waitlist gamification), single region, error budgets and on-call from beta day one. Runbooks for: tick stall (detection: tick-lag alarm; user impact bounded by client autonomy — birds keep performing on the last snapshot), event-log backlog, edge KV staleness, email-provider outage (auth-critical: queue + retry + status surface in matter-of-fact voice). No marketing surfaces, counters, or announcements inside the product at any phase.

### 14.4 Instrumented from day one

The §12.2 set, the tick alarms, the PII canary, deletion verification, and the budget dashboards — all live from M1, because retrofitting observability after calibration starts would leave the weeks-scale drift work blind.

---

## 15. Risks

| # | Risk | Why it's real | Mitigation |
|---|---|---|---|
| R1 | **Drift calibration wrong** (too fast = Tamagotchi; too slow = screensaver) — and we can't measure it in production (privacy boundary) | The product lives in a narrow band; prod aggregates are forbidden | Accelerated harness with cohort assertions (§13.3); behavior-band design makes visibility thresholds explicit; consented dogfood cohort from M2; ship conservative-slow; constants tunable prospectively with changelog, never retro-rewriting vectors |
| R2 | **Procedural calls sound synthetic/uncanny** — the affective spine fails | DSP birdsong is genuinely hard; "almost right" is worse than stylized | M0 spike with sound designer before architecture hardens; stylized-not-photoreal direction; uncanniness panels per milestone; recognizability protocol (§13.5); AudioWorklet perf floor on low-end devices |
| R3 | **Autoplay policy vs "calls already audible"** | Browsers gate audio on gesture; no banner allowed | Designed silent-start: visual aliveness unconditional, audio fades in on first gesture, captions default-on until unlock (§8.5, Ledger D3) |
| R4 | **First-bird <500ms misses on real networks** | Auth + state + render in half a second is tight | Edge-inlined snapshot, edge-local cookie validation, ≤150KB kernel, prebaked pose atlas, budget CI on throttled profile + real-device lab from M0 |
| R5 | **Sync correctness erodes** (lost drift, double-applied events) | Silent failures: a slow-drifting bird fails no test by default | Single-writer enforced at code/DB-role/trigger layers; idempotent cursor-ordered ingest; chaos suite (§13.8); monotonicity audit on every tick |
| R6 | **Presence signal inflates** (multi-device double-count, tampered clients, lax window) | Drift corruption is population-wide and silent (PRD's own warning) | Three-condition conjunction client-side; server union-bucketing, gap rules, hard caps, daily saturation; tampering is self-harm-only by privacy design |
| R7 | **Memory growth** (audio voices, captions, notebook scroll, particles) | 30-min no-growth is a hard gate | Pooling everywhere, virtualized notebook, CI soak with heap assertions |
| R8 | **Accessibility regresses into checklist parity** or slips past launch | The cheap version is always nearer | A11y is M3, pre-beta; narration/captions ride the same voice engine as the product; golden + SR-matrix gates; reduced-motion visual suite |
| R9 | **Tick fleet cost/scale** at dormant-account volume | 1/min × every account is wasteful; naive laziness violates the PRD | Continuous-time formulation + adaptive cadence with proven equivalence (§5.1.3); wake-on-read; tick-lag alarms |
| R10 | **PII leakage via convenience** (email as identifier, trait values in logs) | Named in the PRD as a seen failure mode | UUID-only references, blind index, log scrubbing, CI canary, telemetry denylist (§11) |
| R11 | **Voice drift / gamification creep** by well-meaning contributors | "Just a small toast" is the documented failure shape | Structural omissions (no toast primitive, no announce pipeline), banned-lexicon lint, register-typed copy system, writer ownership, design checklist tied to the five invariants |
| R12 | **Notebook prose feels templated** over months | Grammar repetition is detectable by attentive users | Large authored corpus, per-aviary anti-repetition memory, sparsity (≈1 entry/2–4 days) shrinks exposure, writer-reviewed goldens; post-v1 corpus growth is cheap |

---

## 16. Decision ledger — ambiguity resolutions

Calls this plan makes where the PRD is silent or in tension; each is reversible at the stated cost.

- **D1 — Canvas 2D over WebGL/engine.** ≤7 birds + light particles fits Canvas 2D with layer caching at 60fps on the baseline; smaller kernel serves the 500ms budget. WebGL2 escape hatch behind the renderer interface.
- **D2 — Polling, no WebSockets.** State changes at tick cadence; polling + visibility/frame-gap refetch meets every specified behavior (including visit revocation "at next pull"). Revisit only if a future feature needs sub-tick push.
- **D3 — Audio gesture-gating.** Visual-first aliveness, fade-in on first gesture, captions-by-default until unlock, no banner. The PRD's "calls already audible" reads as design intent constrained by platform reality; this is the closest conforming behavior.
- **D4 — Arrival UX for birds 3–7.** "Appears in the user's flow" interpreted as an in-scene lingering unfamiliar bird + one notebook observation + a quiet adopt surface on focus; declining-by-ignoring departs without penalty. Honors notice-never-announce and no-catalog adoption.
- **D5 — Authored grammar, not LLM, for all prose.** Determinism, auditability, writer ownership, zero latency/cost, no third-party data flow (privacy). Cost: corpus authoring effort (M3) and post-launch corpus growth.
- **D6 — Raw traits never serialized to clients.** Snapshots carry derived, quantized behavior parameters. Slightly larger derivation surface server-side; buys API-level enforcement of the no-numeric-exposure rule.
- **D7 — Export includes personality vectors.** `accounts_sync.md` explicitly enumerates them; `bird_engine.md` bans numeric *display surfaces*. Resolution: the export is a data-ownership artifact (system register), not a product surface; we follow the explicit enumeration. Flagged for product sign-off; trivial to drop the fields if overruled.
- **D8 — No production drift monitoring.** The privacy commitment ("never used … for population-level analysis") outranks calibration convenience. Calibration = harness + consented dogfood only.
- **D9 — Visit link semantics.** "One-time link" = single consumption binds one visitor browser; that browser may return while the invite lives; forwarded links die after first use. Reconciles "one-time" with a revocable *active* invite and a multi-visit log.
- **D10 — Caption scrim chip.** A minimal translucent backing behind captions inside the scene — a deliberate, bounded exception to "no chrome in the scene," because AA contrast over arbitrary scene pixels is otherwise unmeetable. Accessibility wins the tie.
- **D11 — Low ambience bed.** A very quiet procedural air/leaf layer under the calls, so listen-in's "others quiet to ambient" lands on a real floor and unlocked audio never sounds like a void. Consistent with "calls already audible"; trivially removable.
- **D12 — Numeric defaults.** Activity window 5 min; ping 30s; greeting tiers at 10 min/6h/48h; offer credit cooldown 4 min; affordance pacing 90s; pool lifetime ~12 min; listen-in ramps 2.0s/2.5s; dormancy 48h/30d; notebook bucket 1-per-60h; arrival ~day 90 ±14d then ~90d ±21d; nightjar bias by arrival 3–4. All `sim_constants`-tunable; none are hard PRD numbers except where cited (15-min links, 30-day invites/deletion, 5s undo, 7-bird cap, budgets).

---

## 17. Team and sequencing

Seven engineers + three part-time craft roles, mapped to the architecture seams: 2 client/rendering (boot kernel, scene, puppet, reduced-motion), 1 audio DSP (synthesis, grammar, mix — pairs with the sound designer), 2 server/simulation (tick, drift/mood, sync, calibration harness), 1 accounts/lifecycle/visits + edge, 1 a11y + design systems + voice-engine (pairs with the writer). Plus: visual designer (scene, species art, focus/caption treatments), sound designer (motif libraries, uncanniness panels), writer (voice corpus, goldens, lexicon ownership).

Critical path: M0 spikes (audio uncanniness, 500ms boot) → M1 vertical slice (the felt-aliveness test) → calibration clock starts at M2 and runs through launch. The two things that cannot be compressed are wall-clock drift validation and listening-test iteration — which is why both start as early as the plan can physically put them.

— end of plan —
