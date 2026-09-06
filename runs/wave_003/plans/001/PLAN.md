# Pocket Aviary — v1 Implementation Plan

- **Run:** wave_003 / plan slot 001
- **Phase:** 1 (planning only — this document is the deliverable; no product code is written here)
- **Source spec:** `prd/1-START_HERE.md` and the nine PRD files it lists (product_brief, concepts, bird_engine, interactions, aviary_layout, accounts_sync, social_optional, accessibility_perf, non_goals)
- **Audience:** the engineering team executing v1. This plan is written to be executable without further clarification; where the PRD left a value open ("calibrate during build"), this plan pins an initial value, marks it **[CALIBRATE]**, and names the harness that tunes it.

---

## Table of contents

1. Executive summary
2. Scope — what v1 is and isn't
3. Architecture — service shape, client/server split, render-pipeline boundary
4. Data model
5. API surface
6. Simulation engine design — tick, drift, mood, call grammar
7. Sync model — one canonical aviary, zero client-side truth
8. Frontend rendering pipeline
9. Audio pipeline
10. Accessibility surfaces
11. Accounts, auth, and privacy engineering
12. Social — visit invitations
13. Performance budgets and observability
14. Testing and calibration strategy
15. Rollout plan
16. Risks and mitigations
17. Recorded judgment calls (ambiguity log)
18. Workstreams, sequencing, and milestones
19. Appendices (snapshot example, event examples, voice rules for engineers)

---

## 1. Executive summary

Pocket Aviary is a browser-only, single-user virtual aviary whose entire product claim is affective: birds that feel alive because a **server-side simulation advances whether or not anyone is watching**, and a client that renders snapshots of that simulation with procedural motion and procedural audio. The architecture falls out of that claim:

- **The server is the only writer of canonical state.** Personality vectors, moods, positions, notebook entries, and presence accounting live server-side, advanced by a ~60-second tick per aviary. Clients append interaction events to an append-only log and pull snapshots; they never own or merge state. This makes multi-device sync a property of the architecture rather than a feature, and makes last-write-wins corruption structurally unreachable.
- **The client is a renderer and an event source, nothing more.** It interpolates between snapshots, synthesizes calls from a per-species motif grammar via WebAudio, drives mood-shaped idle micro-motion at 60fps, and generates ornaments (leaves, feathers) locally. The first painted frame shows birds mid-action because the snapshot carries each bird's current action and phase — there is no entry animation and no spinner.
- **Two clocks, two layers.** The slow clock (personality drift over weeks, monotonic toward expressive, presence-time dominant) is server-only. The fast clock (mood, calls, motion) is server-canonical at tick granularity and client-presentational at frame granularity. A third, purely local layer (ornaments, exact call notes, micro-motion timing) is deliberately non-canonical so no two devices ever need to agree on a waveform.
- **Privacy is an architectural boundary, not a policy.** Email is PII stored once, encrypted, on the account record; every other reference is a synthetic UUID. Per-bird interaction data never enters telemetry pipelines; aggregate operational metrics carry no per-account dimension. The raw personality vector never leaves the simulation service in any client-facing payload — clients receive derived presentation parameters only (the account export is the single, user-initiated exception).
- **The refusals are load-bearing and get engineering enforcement.** No toasts, no streaks, no badges, no gamification, no notifications about the aviary, no numeric personality readouts, no recorded audio, no native app. Section 2.3 lists the concrete guardrails (copy lints, PR checklists, API schema shape) that keep these refusals from eroding under well-meaning contributions.

The plan is organized into six workstreams (simulation backend; client platform; rendering; audio; accessibility & design system; infrastructure & observability) with a vertical-slice milestone at week 6 (one bird, end to end: snapshot → render → idle motion → procedural call → presence ping → drift delta visible in staging instruments) and GA gated on a drift-calibration lab, an audio-uncanniness protocol, and an accessibility CI matrix.

---

## 2. Scope

### 2.1 In scope for v1

| Area | Ships in v1 |
|---|---|
| Accounts | Email magic-link sign-in (15-min expiry, single-use), per-device revocable session tokens, email change with verification, account export (JSON, emailed link), soft-delete 30 days → hard delete |
| Aviary | One canonical aviary per account; starts with 2 system-selected birds; cap 7; age-gated adoption offers; user-assigned renameable names; stable internal bird identity forever |
| Simulation | Server-side tick (~60s), personality drift (5 traits, monotonic toward expressive), mood system (5–6 states, persists across sessions), bird-to-bird interaction, ambient weather, local-time day/night signals |
| Interactions | Return-greeting (server-directed, client-rendered, never identical twice), listen-in (slow mix re-balance, others never silent), offers (seed / song fragment / still pool, per-bird cooldown), settle (with 5-second undo), presence accounting (three-signal conjunction) |
| Field notebook | Auto-generated naturalist entries, rare (≈1 per few days at regular use), read-only, infinite scrollback, no user-behavior observations |
| Social | Visit invitations: host-initiated, per-invite opt-in, email one-time link, read-only ambient visitor session, revocable, 30-day expiry, visit log in settings, visit-notification toggle (off by default) |
| Accessibility | Screen-reader narration (naturalist prose, slow cadence), call captions generated from the call grammar, reduced-motion mode as a designed cross-fade rendering, full keyboard navigation, WCAG AA contrast on all user copy |
| Performance | <2MB gzipped initial JS, <500ms time-to-first-bird on mid-tier mobile/4G, 60fps idle on a 5-year-old laptop sustained over 30 minutes, no client memory growth over 30 minutes (CI-enforced) |
| Platforms | Last two major versions of Chrome, Safari, Firefox, Edge; matter-of-fact unsupported-browser surface for the rest |

### 2.2 Out of scope for v1 (and the plan does not design for them)

Per `non_goals.md` and the brief's scope statement — these are not deferred features, they are refusals, and nothing in this architecture reserves space for them:

- Native apps (iOS/Android) — web only; no protocol or data-model concessions to native clients.
- Gamification of any flavor: achievements, badges, levels, scores, streaks, XP, "days visited," green-dot calendars, "birds adopted: N" counters, milestone celebrations. No opt-in dashboard, no quiet setting toggle.
- Tamagotchi mechanics: birds never die, never get hungry, never show distress, no decaying happiness meter. Neglect produces ambient quietness (see the `rapport` term, §6.3.4), never visible suffering and never negative trait drift.
- Social-network surfaces: profiles, follows, feeds, public discovery, leaderboards, comments, friend-of-friend chains, co-presence during visits. We do not even compute the cross-account statistics such features would need.
- Payments, shared/multi-aviary accounts, customizable scenes, multi-aviary accounts, push notifications, any email about aviary state ("your birds miss you" class of message — forever out).
- Recorded audio in any form, including as a fallback path. The no-recorded-audio rule is unconditional; the WebAudio fallback is graceful silence with captions on by default.
- Numeric exposure of personality vectors anywhere in the product UI. No debug view, no tier, no toggle. (Account export is the one user-initiated exception the PRD itself specifies.)

### 2.3 Engineering guardrails that make the refusals stick

The PRD is explicit that the biggest threat to this product is a well-meaning contributor adding "just one" announcement or engagement surface. Guardrails:

1. **Copy lint (CI).** A lint pass over all user-facing strings and template grammars fails the build on: exclamation marks in naturalist-voice surfaces, second-person "you" in notebook/narration templates, gamification vocabulary (`streak`, `level`, `score`, `badge`, `achievement`, `unlock`, `congratulations`, `welcome back`) anywhere, and naturalist phrasing in system-voice surfaces (error/settings copy is checked for the inverse: it must be plain, capitalized, direct).
2. **No-toast rule (code-level).** The UI kit contains no toast/snackbar/banner/modal-greeting component. Adding one requires deleting a lint rule, which is visible in review. The only transient surfaces that exist are captions (opt-in), narration (screen-reader path), and the settle-undo window (which is a scene change, not chrome).
3. **API schema shape.** No client-facing endpoint accepts a trait value, a mood value, or any absolute simulation state. Event payloads describe *observations* ("user listened in to bird X for N ms"), never conclusions ("set boldness to 0.62"). A schema-level rule, enforced by types, not discipline.
4. **PR checklist** derived from `non_goals.md`: does this change announce? does this change measure the user rather than the aviary? does this change make absence costly? does this change expose a number that should be felt? Any "yes" blocks merge.
5. **Telemetry allowlist.** Metric definitions live in one registry file; a CI check fails on any new metric carrying an account, bird, or email dimension (§13.3).

---

## 3. Architecture

### 3.1 Service shape

Four deployables plus datastores. Deliberately few services: v1 scale is small, and every service boundary is a place canonical state could leak or lag.

```
                         ┌────────────────────────────────────────────┐
                         │                CDN / Edge                  │
                         │  static shell (HTML/CSS/JS, immutable)     │
                         │  edge function: session check + snapshot   │
                         │  injection into HTML (first-paint path)    │
                         └───────────────┬────────────────────────────┘
                                         │
        ┌────────────────────────────────┼────────────────────────────────┐
        │                                │                                │
┌───────▼────────┐              ┌────────▼─────────┐            ┌─────────▼────────┐
│  api service   │              │  sim service     │            │  auth service    │
│  (stateless)   │              │  (tick workers + │            │  (magic links,   │
│  snapshots,    │◄────────────►│  engine library) │            │   sessions,      │
│  event ingest, │   reads/writes│  tick scheduler, │            │   invites,       │
│  notebook read,│   canonical DB│  drift/mood/     │            │   export/delete) │
│  settings,     │              │  greeting/note-  │            └─────────┬────────┘
│  offer resolve │              │  book generators │                      │
└───────┬────────┘              └────────┬─────────┘            ┌─────────▼────────┐
        │                                │                      │  mail delivery   │
        │                                │                      │  (transactional  │
┌───────▼────────────────────────────────▼───────┐              │  provider)       │
│              PostgreSQL (primary + replica)    │              └──────────────────┘
│  accounts, birds, event_log (partitioned,      │
│  append-only), snapshots, notebook_entries,    │
│  invitations, visit_log, settings, sessions    │
└────────────────────────────────────────────────┘
        Redis: snapshot hot cache, edge revocation list (30s TTL),
               tick scheduler queue (next_tick_at), rate-limit counters
```

- **api service** (stateless HTTP): serves snapshot pulls, ingests events, resolves offers synchronously (§6.9), serves notebook/settings/account surfaces, hosts the visitor read-only endpoints. Horizontally scalable; no in-memory canonical state.
- **sim service**: owns the engine library (drift, mood, call-activity, weather, greeting director, notebook generator) and runs the tick workers. The *only* code path that writes personality vectors, moods, or canonical positions. The api service links the engine library read-only for the synchronous offer-reaction fast path, which writes its result through a single `apply_reaction` routine that appends to the event log and updates only reaction/cooldown fields — never traits (traits are folded in by the next tick, preserving single-writer discipline for drift).
- **auth service**: magic-link issuance/consumption, session tokens (signed, revocable), email-change verification, invitation tokens, export generation, deletion state machine. Separate deployable so credential-handling code has its own review and deploy bar.
- **mail delivery**: transactional provider (SES/Postmark class). Emails contain links and system copy only — never aviary state, never bird names, never per-account data beyond the recipient address. This keeps the third-party boundary clean against the privacy commitment.
- **PostgreSQL 16** (primary + one replica): canonical store. Chosen over a document store because the invariants that matter (append-only event log with per-account sequence, single-writer tick with advisory locks, transactional snapshot writes) are exactly what Postgres makes boring. The event log is a partitioned table (§4.3).
- **Redis**: hot snapshot cache (api reads through it), the edge revocation list, the tick scheduler's `next_tick_at` sorted set, and rate-limit counters. All Redis data is derivable from Postgres; Redis loss degrades latency, never correctness.
- No message broker in v1. At v1 scale (one tick per account per minute, tiny payloads), a Postgres/Redis-backed scheduler is simpler and removes an entire class of ordering bugs. The event log *is* the queue: the tick consumes it by sequence cursor. If volume later demands it, the log's shape (append-only, per-account ordered, cursor-consumed) ports to Kafka without changing the engine — and the synthetic-UUID rule means the partition key is never PII.

### 3.2 Client/server split — the canonicality table

The single most important table in this plan. Every behavior in the product is classified into exactly one of three tiers; code review enforces the classification.

| Tier | Owner | Cadence | Contents |
|---|---|---|---|
| **Canonical simulation state** | sim service (server) | tick (~60s) | personality vectors, mood + mood timers, rapport (recency EMA), perch targets/positions at tick granularity, weather state, settled flag, offer cooldowns, notebook entries, adoption availability, greeting *intent* |
| **Render-time behavior** | client | 60fps / audio clock | position interpolation between snapshots, idle micro-motion selection and execution, call scheduling within server-provided activity parameters, exact motif notes and timing, bird-to-bird audible responses, greeting *execution* from the directive, listen-in mix ramps, settle lighting transition, caption text rendering |
| **Pure ornaments** | client, no state | ambient | leaf/feather drift, parallax, water shimmer, rain/wind particle rendering, top-bar fade |

Rules that fall out:

- **Clients never tick.** A client that is closed, suspended, or backgrounded has no effect on canonical state except through the events it managed to upload.
- **Two devices never need to agree on a waveform or a frame.** Call note choices and micro-motion timings are device-local by design; canonical state is the call *activity* (rate, energy, motif weights), not the audio. This is why "the same aviary in the same mood" holds across devices without any device-to-device sync.
- **The server decides *who/what/why*; the client decides *exactly how it looks and sounds*.** Greeting: server picks the bird, the form class, and the variation seed (§6.8); the client renders a unique instance. Offer: server resolves the reaction type from mood/personality (§6.9); the client animates it.

### 3.3 Render-pipeline boundary

The boundary between "simulation" and "rendering" is the **snapshot** (§4.4). Everything above the snapshot line is deterministic, persisted, and identical for every device; everything below is stochastic, ephemeral, and device-local. The snapshot is small (2–6KB for 7 birds), versioned monotonically, and self-sufficient for first paint: it carries per-bird position, current action, action phase, mood, call-activity parameters, lighting phase inputs, weather state, greeting directive (when due), and scheduled narration lines.

### 3.4 Technology choices (with rationale)

| Choice | Decision | Rationale |
|---|---|---|
| Language | TypeScript everywhere (client + server) | One language across the engine/client boundary; the engine library's types (mood enum, event taxonomy) are shared verbatim, killing drift-between-layers bugs |
| Server runtime | Node.js 22 LTS | Mature, hireable, fine for a tick-per-minute workload |
| Client framework | Preact for chrome/panels (top bar, notebook, settings); hand-rolled scene layer for the canvas | The scene is a game-loop problem, not a DOM-diffing problem; chrome is a small DOM problem. Keeps the initial bundle far under budget |
| Scene rendering | Canvas 2D, layered offscreen canvases | 7 birds + particles at 60fps is comfortably within Canvas2D on a 5-year-old laptop *if* static layers are cached; WebGL buys headroom we don't need and costs bundle + driver-compat risk. Cross-fades for reduced-motion are trivial in Canvas2D |
| Bird visuals | Procedural pose/skeleton system drawn from species parameter sets; compact SVG path data for silhouettes | "Bird visual assets are generated procedurally where possible" (PRD). Procedural poses are also what make mood-shaped idle motion and mid-action first frames possible |
| Audio | Raw WebAudio API (no framework) | We need sample-accurate lookahead scheduling and per-bird gain nodes; libraries add bundle and abstraction we'd fight |
| Database | PostgreSQL 16 | §3.1 |
| Edge | CDN with edge functions (Cloudflare Workers class) | PRD requires the initial snapshot delivered from edge with the HTML (§8.1) |
| Infra | Single cloud region + CDN; containers on a managed compute layer (ECS/Cloud Run class) | v1 scale is tiny; no k8s ceremony |
| IaC / CI | Terraform; GitHub Actions with the gates in §13.1 and §14 | Standard |

### 3.5 Environments

- `dev` (engineers' machines; docker-compose: Postgres, Redis, api, sim, mailhog).
- `staging` — full stack at production shape, plus the **calibration lab**: synthetic persona accounts driven by a headless-client harness that replays weeks of presence/interaction patterns in accelerated time (§14.2). Staging is where drift, mood, notebook rarity, and greeting behavior are tuned. Production user data is *never* used for this (§13.3).
- `prod` — canary + full deployment lanes; config flags in §15.5 gate every tunable.

---

## 4. Data model

### 4.1 Identity and PII rules (non-negotiable)

1. `account_id` is a synthetic UUIDv7 generated at account creation. It is the identifier in every table, every log line, every metric label that needs one (it doesn't — §13.3), every cache key, every queue entry.
2. Email exists in exactly one place: `accounts.email_encrypted` (envelope encryption; KEK in the cloud KMS, DEK per account). Lookup by email goes through a blinded index: `accounts.email_hash = HMAC(email, server_pepper)`, used only by the auth service for sign-in and invitation matching. No other service can resolve email → account or account → email except through the auth service's export/invite paths.
3. `bird_id` is a synthetic UUIDv7, immutable for the life of the account. Name and species are mutable attributes; identity is the UUID. No migration, pool change, or rename ever mints a new bird_id or reuses a dead one. A CI migration test asserts bird_id set invariance across every schema migration (§14.1).
4. Visitor emails (invitations) follow the same rule: stored hashed + encrypted on the invitation record; the visit log displays them to the host because the host supplied them (that is the host's own data, shown back to the host).

### 4.2 Schemas

SQL-sketch; types are the contract, column lists are complete for v1.

```sql
-- ============ accounts & auth ============
CREATE TABLE accounts (
  account_id      uuid PRIMARY KEY,               -- synthetic, the only identifier
  email_encrypted bytea NOT NULL,                 -- envelope-encrypted
  email_hash      bytea NOT NULL UNIQUE,          -- HMAC blind index (auth service only)
  timezone        text NOT NULL DEFAULT 'UTC',    -- IANA; account-level, updated from client (§6.6.4)
  created_at      timestamptz NOT NULL DEFAULT now(),
  aviary_created_at timestamptz,                  -- "aviary age" clock for adoption pacing; null until onboarding completes
  status          text NOT NULL DEFAULT 'active', -- active | pending_deletion | deleted
  deletion_requested_at timestamptz,              -- soft-delete marker; hard delete at +30d
  settings        jsonb NOT NULL DEFAULT '{}'     -- accessibility/audio/notification settings (§4.2.9)
);

CREATE TABLE sessions (
  session_id     uuid PRIMARY KEY,
  account_id     uuid NOT NULL REFERENCES accounts(account_id),
  token_hash     bytea NOT NULL UNIQUE,           -- we store hashes, never tokens
  device_label   text,                            -- user-agent-derived, shown in settings list
  created_at     timestamptz NOT NULL DEFAULT now(),
  last_seen_at   timestamptz NOT NULL,
  expires_at     timestamptz NOT NULL,            -- 30 days sliding
  revoked_at     timestamptz
);

CREATE TABLE magic_links (
  token_hash   bytea PRIMARY KEY,
  email_hash   bytea NOT NULL,
  created_at   timestamptz NOT NULL DEFAULT now(),
  expires_at   timestamptz NOT NULL,              -- created_at + 15 min
  consumed_at  timestamptz                        -- single-use; consumed atomically
);

CREATE TABLE email_changes (
  account_id        uuid PRIMARY KEY REFERENCES accounts(account_id),
  new_email_encrypted bytea NOT NULL,
  new_email_hash    bytea NOT NULL,
  token_hash        bytea NOT NULL,
  requested_at      timestamptz NOT NULL,
  expires_at        timestamptz NOT NULL,         -- 24h; old email works until verified
  completed_at      timestamptz
);

-- ============ birds & aviary state ============
CREATE TABLE birds (
  bird_id       uuid PRIMARY KEY,                 -- stable forever (§4.1.3)
  account_id    uuid NOT NULL REFERENCES accounts(account_id),
  species_key   text NOT NULL,                    -- from the species pool (§6.5.2)
  name          text NOT NULL,                    -- user-assigned; renameable; ≤24 chars
  adopted_at    timestamptz NOT NULL DEFAULT now(),
  -- personality vector: the slow clock. Written ONLY by the tick (§6.1).
  p_boldness    real NOT NULL,                    -- [0,1]
  p_warmth      real NOT NULL,                    -- [0,1] social warmth
  p_vocal       real NOT NULL,                    -- [0,1] vocal frequency
  p_plumage     real NOT NULL,                    -- [0,1] plumage saturation
  p_curiosity   real NOT NULL,                    -- [0,1]
  -- fast clock:
  mood          text NOT NULL,                    -- alert|curious|content|wary|drowsy|resting (§6.4)
  mood_since    timestamptz NOT NULL,
  rapport       real NOT NULL DEFAULT 0,          -- recency EMA of presence, half-life ~3d (§6.3.4)
  -- presentation state at tick granularity:
  perch_zone    text NOT NULL,                    -- front|middle|back
  perch_anchor  smallint NOT NULL,                -- anchor index within zone
  pos_x         real NOT NULL,                    -- normalized [0,1] scene coordinate
  action        text NOT NULL,                    -- idle action class at tick time (§8.4)
  action_phase  real NOT NULL,                    -- [0,1) phase within action — mid-action first frames
  call_rate     real NOT NULL,                    -- calls/min activity parameter for client grammar
  call_energy   real NOT NULL,                    -- [0,1] amplitude/liveliness parameter
  offer_cooldown_until timestamptz,
  UNIQUE (account_id, name)                       -- names unique within an aviary (case-insensitive index)
);
CREATE INDEX birds_account ON birds(account_id);

CREATE TABLE aviary_state (                        -- one row per account; scene-level canonical state
  account_id     uuid PRIMARY KEY REFERENCES accounts(account_id),
  bird_count     smallint NOT NULL DEFAULT 0,      -- denormalized; cap 7 enforced in engine
  settled        boolean NOT NULL DEFAULT false,
  settled_at     timestamptz,
  weather        text NOT NULL DEFAULT 'clear',    -- clear|rain|wind (§6.7)
  weather_until  timestamptz,
  state_version  bigint NOT NULL DEFAULT 0,        -- monotonic; bumped every snapshot write
  last_tick_at   timestamptz NOT NULL,
  last_tick_index bigint NOT NULL,                 -- floor(unix_ts/60) of last applied tick (§6.2)
  event_cursor   bigint NOT NULL DEFAULT 0,        -- last consumed event_log.account_seq
  last_presence_end timestamptz,                   -- for absence bands in greeting (§6.8)
  arrival_id     uuid                              -- current arrival window id; greeting fires once per arrival
);

-- ============ event log (append-only) ============
CREATE TABLE event_log (
  account_seq  bigint GENERATED ALWAYS AS IDENTITY,-- per-account ordering is (account_id, seq);
  event_id     uuid NOT NULL,                      -- client-generated ULID/UUIDv7 — idempotency key
  account_id   uuid NOT NULL,
  actor        text NOT NULL,                      -- 'host' | 'system' (visitor events are NOT written here, §12.2)
  type         text NOT NULL,                      -- taxonomy §4.3
  client_ts    timestamptz NOT NULL,
  received_at  timestamptz NOT NULL DEFAULT now(),
  payload      jsonb NOT NULL DEFAULT '{}',
  PRIMARY KEY (account_id, account_seq),
  UNIQUE (account_id, event_id)                    -- dedupe on replay
) PARTITION BY RANGE (received_at);                -- monthly partitions; retention §4.5

-- ============ snapshots ============
CREATE TABLE snapshots (                           -- latest materialized snapshot per account (hot copy in Redis)
  account_id   uuid PRIMARY KEY REFERENCES accounts(account_id),
  state_version bigint NOT NULL,
  payload      jsonb NOT NULL,                     -- the §4.4 document, minus greeting/narration which are computed at pull time
  created_at   timestamptz NOT NULL DEFAULT now()
);

-- ============ notebook ============
CREATE TABLE notebook_entries (
  entry_id    uuid PRIMARY KEY,
  account_id  uuid NOT NULL REFERENCES accounts(account_id),
  entry_ts    timestamptz NOT NULL,                -- simulated observation time
  text        text NOT NULL,                       -- immutable once written; naturalist voice
  salience    smallint NOT NULL,                   -- generator's noteworthiness score (internal)
  template_key text NOT NULL                       -- which grammar template produced it (voice QA, §6.10)
);
CREATE INDEX notebook_account_ts ON notebook_entries(account_id, entry_ts DESC);

-- ============ visits ============
CREATE TABLE visit_invitations (
  invite_id      uuid PRIMARY KEY,
  host_account_id uuid NOT NULL REFERENCES accounts(account_id),
  visitor_email_encrypted bytea NOT NULL,
  visitor_email_hash bytea NOT NULL,
  token_hash     bytea NOT NULL UNIQUE,            -- one-time link token
  created_at     timestamptz NOT NULL DEFAULT now(),
  expires_at     timestamptz NOT NULL,             -- +30 days if unused
  first_used_at  timestamptz,
  revoked_at     timestamptz
);

CREATE TABLE visit_sessions (                      -- exchanged from a one-time link; read-only scope
  visit_session_id uuid PRIMARY KEY,
  invite_id      uuid NOT NULL REFERENCES visit_invitations(invite_id),
  token_hash     bytea NOT NULL UNIQUE,
  created_at     timestamptz NOT NULL DEFAULT now(),
  expires_at     timestamptz NOT NULL,             -- 7 days from exchange
  revoked_at     timestamptz,                      -- set when host revokes; checked every snapshot pull
  last_seen_at   timestamptz
);

CREATE TABLE visit_log (                           -- host-visible transparency record (§12.4)
  visit_id     uuid PRIMARY KEY,
  invite_id    uuid NOT NULL REFERENCES visit_invitations(invite_id),
  started_at   timestamptz NOT NULL,
  ended_at     timestamptz,                        -- approximate; last snapshot pull + grace
  notified     boolean NOT NULL DEFAULT false      -- opt-in email notification sent (§12.4)
);
```

Notes:

- **4.2.9 `accounts.settings` (jsonb)** holds: `reduced_motion: 'auto'|'on'|'off'` (auto = follow `prefers-reduced-motion`), `captions: bool`, `audio_muted: bool`, `narration verbosity: 'normal'|'sparse'`, `visit_notifications: bool` (default false), `privacy_policy_ack: ts`. Settings are system-surface data; the settings UI uses matter-of-fact voice.
- There is deliberately **no table for streaks, visit counts, or engagement scores**. Presence exists only as `event_log` rows and the derived `rapport`/personality values. Nothing in the schema can answer "how many days in a row did this user visit" without reconstructing it from the raw log — and the log prunes at 90 days (§4.5), so the gamification temptation is schema-hostile, which is the point.
- **No `mood_history` table.** Mood transitions are ephemeral canonical state; the notebook is the only long-term record of moments, by design.

### 4.3 Event taxonomy

Every row in `event_log` has one of these types. Payloads describe observations, never conclusions (§2.3.3).

| type | payload | written by | consumed for |
|---|---|---|---|
| `presence_start` | `{arrival_id}` | client (gate opens) | greeting absence band; presence integration |
| `presence_ping` | `{arrival_id, gate:{vis,focus,act}}` | client, every 30s while gate holds | presence minutes (dominant drift input) |
| `presence_end` | `{arrival_id, reason:'settle'|'hidden'|'closed'|'lapsed'}` | client (best-effort, `sendBeacon`) | ends presence window; server infers end from last ping if absent |
| `listen_in_start` | `{bird_id}` | client | drift: warmth+vocal for focused bird |
| `listen_in_end` | `{bird_id, duration_ms}` | client | drift integration (duration is the signal) |
| `offer` | `{kind:'seed'|'song'|'pool', song_key?, reaction_id}` | api (on offer resolve, §6.9) | drift: curiosity (accepted) / boldness (offered near) |
| `settle` | `{}` | client | mood-quieting signal; ends presence window cleanly |
| `settle_undo` | `{settle_event_id}` | client (≤5s) | reverses settled flag if window valid |
| `rename` | `{bird_id, name}` | api (fast-path write + log) | snapshot name update; notebook voice |
| `session_meta` | `{tz, audio_available, reduced_motion_pref, muted}` | client at session start | timezone update; audio-fallback context; mute multiplier (§6.3.3) |
| `adopt` | `{bird_id, species_key, name}` | api (adoption flow) | new bird record; fly-in directive |
| `system_note` | `{text_key, params}` | sim (rare, e.g., weather extremes for notebook salience) | notebook generator input |

Visitor sessions write **nothing** to `event_log` — visitor activity lands only in `visit_log` (start/approximate end). This is the structural guarantee that "a visitor sitting and watching for an hour does not drift the host's birds."

### 4.4 Snapshot payload (the sync contract)

Computed at pull time from `snapshots.payload` + live additions (greeting directive, narration lines). Target size 2–6KB for 7 birds. Full example in Appendix C; shape:

```jsonc
{
  "v": 1,                          // payload schema version
  "state_version": 81234,          // monotonic per account
  "tick_at": "2026-09-06T07:31:00Z",
  "account": { "timezone": "Europe/Berlin", "settings_digest": { "captions": false, "reduced_motion": "auto", "muted": false } },
  "aviary": {
    "lighting": { "phase": "morning", "sun_t": 0.31 },   // sun_t = normalized position in local day, client renders gradient
    "weather": { "state": "clear", "until": null },
    "settled": false,
    "bird_count": 2,
    "adoption_offer_available": false                    // pull-only surface (§6.11); never pushed
  },
  "birds": [
    {
      "id": "0192c…", "name": "pip", "species": "warbler",
      "mood": "content", "mood_since": "…",
      "pos": { "zone": "front", "anchor": 2, "x": 0.38, "y": 0.62 },
      "action": { "type": "preen", "phase": 0.42, "since": "…" },   // mid-action first frame
      "move": null,                                                  // or { "to": {...}, "depart_at": "…", "mode": "fly"|"hop" }
      "call": { "rate": 1.7, "energy": 0.6, "motif_seed": 41 },      // activity params for client grammar
      "render": { "plumage_sat": 0.52, "fluff": 0.2 },               // derived presentation params ONLY
      "offer_cooldown_until": null,
      "listen_in": false
    }
  ],
  "greeting": null,                 // or directive, §6.8 — present only on a fresh arrival window
  "narration": { "lines": [ { "at_offset_s": 0, "text": "…" }, { "at_offset_s": 45, "text": "…" } ] },
  "fly_in": null                    // or { "bird_id": "…", "from": "offscreen_left" } — once, post-adoption (§8.1.4)
}
```

**Enforcement point:** the snapshot contains no raw personality values. `render.plumage_sat` is a derived presentation parameter (a monotonic function of `p_plumage` with per-species offset, computed by the engine's presentation mapper). Boldness/warmth/curiosity/vocal appear only as already-resolved behavior: position, action, call rate, greeting directive. The API layer's response types do not contain trait fields; a unit test snapshots the OpenAPI schema and fails if any field matching `p_*|trait|boldness|warmth|vocal|curiosity|plumage` (beyond the derived `plumage_sat`) appears in any client-facing response except the account export.

### 4.5 Retention and deletion

- `event_log`: monthly partitions; raw events pruned 90 days after consumption by the tick. Drift is already folded into vectors; the notebook is the user-facing record; nothing downstream needs raw events after integration. Pruning is privacy-positive and is named in the privacy policy.
- `snapshots`: latest only (single row per account, overwritten).
- Notebook entries: kept for account lifetime, never archived or hidden (PRD: indefinite scrollback).
- Account deletion: `status='pending_deletion'`, tick stops, snapshot pulls return the recovery surface; at +30 days a hard-delete job removes every row keyed by `account_id` across all tables and prunes Redis keys. Backup rotation is ≤35 days so deleted data ages out of PITR windows; the privacy policy states this plainly.
- Export files: generated to object storage with a 72-hour signed-URL lifetime, deleted after expiry or download, whichever comes first.

---

## 5. API surface

### 5.1 Conventions

- JSON over HTTPS, versioned path prefix `/v1/`. Auth via `HttpOnly; Secure; SameSite=Lax` session cookie (signed, Ed25519; 30-day sliding expiry; revocation via server-side `sessions` rows + edge-cached revocation list, 30s TTL).
- **Error model:** `{ "code": "magic_link_expired", "message": "We couldn't sign you in. The link may have expired. Try requesting a new link." }` — `message` is user-displayable, matter-of-fact voice, never naturalist (§19.3 maps voice per surface). `code` is the machine key. Errors never contain email addresses or account UUIDs in `message`.
- **Idempotency:** every state-changing client request carries a client-generated `event_id` (UUIDv7). Replays are deduped by the `UNIQUE (account_id, event_id)` constraint and return the original result. This makes the offline replay path (§7.5) safe.
- **Conditional pulls:** `GET /v1/aviary/snapshot` accepts `If-None-Match: "<state_version>"` and returns `304` when unchanged — cheap keepalives.
- No client-submitted state fields anywhere: grep the OpenAPI spec for request bodies — they contain ids, kinds, durations, names, settings, and timestamps. That is the whole writable surface.

### 5.2 Endpoint catalog

**Auth (auth service; matter-of-fact voice on every response):**

| Method & path | Purpose | Notes |
|---|---|---|
| `POST /v1/auth/link` | Request magic link `{email}` | Always 204 (no account enumeration). Rate limit: 5/hour/email, 10/day/email. Link TTL 15 min |
| `GET /v1/auth/verify?token=…` | Consume link, create session, 302 to `/` | Atomic single-use consumption. New email → account creation → onboarding route |
| `POST /v1/auth/signout` | Revoke current session | 204 |
| `GET /v1/sessions` | List active sessions (device label, last seen) | Account settings surface |
| `POST /v1/sessions/{id}/revoke` | Revoke a session | Immediate; edge revocation list updated |
| `POST /v1/account/email-change` | Start email change | Verification email to *new* address; old email works until verified |
| `GET /v1/account/email-change/verify?token=…` | Complete email change | 24h TTL |
| `POST /v1/account/export` | Generate JSON export, email download link | Includes birds, names, **current personality vectors**, moods, notebook, settings — the one sanctioned numeric exposure, user-initiated, system-voice email |
| `POST /v1/account/delete` | Soft-delete (30d) | Returns recovery instructions copy |
| `POST /v1/account/restore` | "I changed my mind" | Only valid while `pending_deletion` |

**Aviary (api service):**

| Method & path | Purpose | Notes |
|---|---|---|
| `GET /v1/aviary/snapshot` | Pull canonical state (§4.4) | `If-None-Match` support; `?tz=` hint updates account timezone (§6.6.4); computes greeting directive + narration lines at pull time; also served (pre-injected) by the edge function for first paint (§8.1) |
| `POST /v1/events` | Batch-append interaction events `{events:[…]}` | 202 `{accepted, duplicates, rejected:[{event_id, code}]}`. Server validates plausibility (§6.12.2); rejected events get codes like `presence_rate_implausible` |
| `POST /v1/birds/{bird_id}/offers` | Offer, synchronous reaction `{kind, song_key?, event_id}` | 200 `{reaction, cooldown_until, narration}` (§6.9). Reaction `type` may be `approach`, `wait_then_approach`, `ignore`, `join_song`, `call_against`, `go_quiet`, `drink`, `bathe`, `watch`. Cooldown active → 200 `{reaction:{type:'cooldown'}, retry_after}` (not an error; the client normally prevents this via snapshot state) |
| `PUT /v1/birds/{bird_id}` | Rename `{name, event_id}` | ≤24 chars, unique per aviary, sanitized; direct canonical write (names are not simulation state) + event log row |
| `POST /v1/aviary/adopt` | Accept an age-gated adoption offer `{name, event_id}` | 409 `no_offer_available` if gate unmet or count=7. Server assigns species (§6.11); response includes `fly_in` directive |
| `GET /v1/notebook?before=<entry_id>&limit=50` | Notebook scrollback | `{entries:[{id, ts, text}], next_cursor}`; immutable, read-only; no edit/delete endpoints exist |
| `GET/PUT /v1/settings` | Accessibility/audio/notification settings | System-voice surface |

**Visits (host side, auth+api; visitor side, api):**

| Method & path | Purpose | Notes |
|---|---|---|
| `POST /v1/visits/invitations` | Invite `{email, event_id}` | Emails one-time link `/visit/{token}`; 30-day expiry |
| `GET /v1/visits/invitations` | Outstanding invitations | Settings surface |
| `POST /v1/visits/invitations/{id}/revoke` | Revoke (outstanding or active) | Immediate: invalidates invite + all visit sessions minted from it; revocation list to edge |
| `GET /v1/visits/log` | Visit log (who, when, approximate duration) | Ordered most-recent-first; no badge, no push (§12.4) |
| `GET /v1/visits/{token}/exchange` | Visitor: exchange one-time link → visit session cookie | Marks `first_used_at`; 7-day visit session |
| `GET /v1/visits/snapshot` | Visitor: read-only snapshot pull | Same snapshot minus `account`, minus `greeting` (visitors never trigger greetings), minus `adoption_offer_available`; revoked/expired → 410 with matter-of-fact "visit no longer available" surface copy. No event endpoints exist for visitor sessions — the API surface itself cannot record visitor presence |

### 5.3 Visit-invitation flow (sequence)

```
host                api/auth                 mail            visitor
 │ POST invitations   │                        │                 │
 │ {email}───────────►│ create invite (token   │                 │
 │                    │ hashed, 30d expiry)    │                 │
 │                    │───────────────────────►│ "name invited   │
 │                    │                        │  you to visit   │
 │                    │                        │  their aviary"  │
 │                    │                        │ (matter-of-fact,│
 │                    │                        │  no bird data)──►│ opens /visit/{token}
 │                    │◄────────────────────────────────────────── │ exchange → visit session
 │                    │ visitor snapshot pulls (read-only, ~30s)   │
 │ GET visits/log     │                        │                 │
 │───────────────────►│ shows visit + duration │                 │
 │ POST revoke ──────►│ sessions revoked; next visitor pull → 410 surface
```

### 5.4 Rate limiting and abuse

- Per-IP + per-account limits on auth endpoints (above); `POST /v1/events`: max 100 events/batch, 10 batches/min/account; presence_ping server-side plausibility caps (§6.12.2); offers: enforced by per-bird cooldown + 20/hour/account ceiling; invitations: 10 outstanding/host, 50 lifetime/day/host (anti-spam; invitation emails are the only abuse vector with third-party impact).
- Visitor snapshot pulls: 10/min/visit-session.
- All limits return 429 with matter-of-fact copy and `Retry-After`.

---

## 6. Simulation engine design

The engine is a pure-function library (`@aviary/engine`, shared TypeScript package) plus the tick worker that applies it to canonical state. Every function in the engine is **deterministic given (state, events, tick_index, PRNG seed)** — this is what makes lazy catch-up identical to eager ticking (§6.1.3) and what makes the calibration harness trustworthy (§14.2).

### 6.1 The tick

#### 6.1.1 Cadence and scheduling

- Canonical cadence: one tick per 60 seconds of simulated time. `tick_index = floor(unix_seconds / 60)` — ticks are indexed by **absolute wall-clock time**, not by per-account counters, so the state of an aviary at time T is a pure function of its state at any earlier tick plus the events and tick indices in between. This is the determinism backbone.
- Scheduler: Redis sorted set `tick_queue` keyed by `next_tick_at`. Sim workers pop due accounts, tick them, re-enqueue.
- **Eager set:** accounts with any event, presence, or snapshot pull in the last 72 hours tick every 60s whether or not a client is connected (the PRD's "the tick runs whether or not any client is connected" — honored for every account that is plausibly being visited).
- **Dormant set:** accounts idle >72 hours are swept every 6 hours, and receive an **on-demand catch-up** at snapshot-pull time (§6.1.3) before the response is served.

#### 6.1.2 Tick body (pseudocode)

```
tick(account, now):
  lock = pg_advisory_xact_lock(hash(account_id))     # single writer; skip if held (another worker mid-tick)
  st   = aviary_state[account_id]
  from_index = st.last_tick_index + 1
  to_index   = floor(now / 60s)
  for idx in from_index..to_index:                   # catch-up loop; normally exactly one iteration
    events = event_log[account_id] where account_seq > st.event_cursor
                                     and received_at <= tick_end(idx)
    # 1. presence integration (validated pings only, §6.12)
    presence_minutes = integrate_presence(events, idx)
    # 2. fast clock: rapport EMA update (half-life 3 days)
    rapport' = rapport * exp(-Δt/τ) + k * presence_minutes   (capped at 1.0)
    # 3. slow clock: additive drift deltas (§6.3) — server-authored, monotonic ≥ 0
    for bird in birds:
      Δ = drift_deltas(bird, events, presence_minutes, idx)
      bird.p_* = clamp(bird.p_* + Δ, 0, 1)           # traits only ever increase (§6.3.2)
    # 4. mood transitions (§6.4), deterministic order: birds by adopted_at
    weather = weather_scheduler(account_id, idx)      # (§6.7) pure function of (account, day, idx)
    for bird in birds (in order):
      bird.mood = transition(bird.mood, bird.p_*, bird.rapport', time_of_day(account.tz, idx),
                             weather, recent_events(bird), neighbor_moods)
      bird.mood_since = changed ? tick_time(idx) : bird.mood_since
    # 5. positions & actions: perch selection from mood × boldness × rapport (§6.6)
    for bird in birds:
      (zone, anchor, x) = perch_choice(bird, weather, time_of_day)   # may equal current (birds don't fidget perches every tick)
      action, phase     = action_at(bird, idx)                        # idle action class + phase for mid-action snapshots
      call_rate, call_energy = call_activity(bird.p_vocal, bird.mood, time_of_day, weather, bird.rapport')
    # 6. settled flag decay: settled clears on any new arrival or interaction event
    # 7. notebook: candidate observation scoring + rarity budget (§6.10)
    maybe_write_notebook(account, birds, events, weather, idx)
    # 8. persist: single transaction — birds, aviary_state (state_version+1, cursor, last_tick_index=idx), snapshots row, Redis hot copy
```

Per-account advisory lock + single-transaction persist = exactly one writer, no torn states. Tick latency target: p50 <150ms, p99 <1s per account-tick (alarm at p99 5s per §13.5).

#### 6.1.3 Lazy catch-up ≡ eager ticking (invariant + test)

When a dormant account pulls a snapshot, the api service asks sim to catch up: the same `tick()` runs the `from_index..to_index` loop over elapsed absolute-time indices before responding. Because every stochastic draw is seeded by `(bird_id, tick_index)` (§6.2) and weather is a pure function of `(account_id, day, idx)`, **catch-up produces bit-identical state to what eager ticking would have produced**. A CI property test runs both schedules over 30 simulated days for fixture accounts and asserts byte-equal final state (§14.7). This preserves the PRD's guarantee — "personality drifts during the user's absence based on inputs from before they left, not on inputs invented at the moment they return" — while not burning a worker-year ticking accounts nobody opens.

Bounded catch-up: if `to_index - from_index > 44,640` (~31 days), the loop is collapsed using the closed-form integral of the presence-independent terms (rapport decay is exponential; mood re-anchors at each local dawn anyway; weather only affects mood transiently). The collapse is derived from the same formulas and covered by the same equivalence test with a 90-day fixture.

### 6.2 Determinism and PRNG discipline

- One PRNG implementation (xoshiro256** via a tiny TS port), never `Math.random()` in engine code (lint-enforced: `no-restricted-syntax` on `Math.random` in `packages/engine`).
- All draws are keyed: `rng(account_id, bird_id, tick_index, purpose_tag)`. Purpose tags: `mood_transition`, `perch_choice`, `action_pick`, `weather`, `greeting_pick`, `greeting_form`, `notebook_pick`, `notebook_fill`.
- Client-side render/audio variation uses device-local RNG deliberately (calls are ephemeral, §3.2); the server supplies integer seeds (`motif_seed`, greeting `variation_seed`) so the *character* of a moment is canonical even though the exact notes are not.

### 6.3 Drift function (the slow clock)

#### 6.3.1 Shape

Drift is a low-pass filter over presence-and-interaction signals, implemented per tick as **additive, saturating, non-negative deltas**:

```
Δtrait(tick) = η_trait · Σ_sources [ w_source · s_source(tick) ] · (1 − trait)
trait ← trait + Δtrait            # clamp [0,1]; Δ ≥ 0 always
```

- The `(1 − trait)` factor makes growth asymptotic: fast-ish early, slow near the ceiling — no trait ever exceeds 1, and late drift can't overshoot.
- `s_source(tick)` is the normalized signal strength in this tick's window:
  - `s_presence = min(validated_presence_minutes, 5) / 5` — dominant input, capped per tick so a single marathon session can't spike drift.
  - `s_listenin(bird) = min(listen_in_minutes_for_bird, 3) / 3` — applies to the focused bird's `p_warmth` and `p_vocal` only.
  - `s_offer_accept(bird) = 0.5 per accepted offer` (capped 1.0/tick) → `p_curiosity`; `s_offer_near(bird) = 0.3 per offer where bird was within one perch zone` → `p_boldness`.
  - `s_settle = 0` for drift direction (settle is mood-quieting only, per PRD); it does end the presence window cleanly.
- Initial learning rates **[CALIBRATE]**: `η_presence = 0.00035/tick` (all five traits receive presence signal, weighted: plumage ×1.0, boldness ×0.8, warmth ×0.8, vocal ×0.6, curiosity ×0.4); `η_listenin = 0.0009/tick`; `η_offer = 0.0012/tick`. With the regular-visitor persona (below), these land week-1 Δ ≈ 0.02–0.04 (instruments-visible) and week-3 cumulative Δ ≈ 0.08–0.15 (user-visible), matching the PRD's calibration target. The values live in server config, not code, and the calibration lab (§14.2) owns them.
- Seed values for newly-adopted birds **[CALIBRATE]**: per-trait `N(species_baseline, 0.08)` clamped to `[0.15, 0.55]` — new birds start modest and unformed; the ceiling-on-seed guarantees drift has somewhere to go and that the first weeks of a relationship are the fastest-feeling (within the invisible-per-session rule).

#### 6.3.2 Monotonicity (the anti-Tamagotchi invariant)

`Δ ≥ 0` on every trait, every tick, unconditionally. There is no code path that decreases a personality value — not neglect, not muting, not settling, not migration, not support tooling (the admin surface has no trait editor; the only write is "restore from backup"). Enforcement: a DB trigger rejects any UPDATE that decreases a `p_*` column (belt-and-braces; the engine never emits one), and a CI test fuzzes event sequences asserting monotonicity. This is the load-bearing implementation of "punishing absence is the central mistake" — it is a database-level fact, not a convention.

#### 6.3.3 Mute handling (judgment call)

The brief lists "whether you mute the calls or let them play" among the small ways users show up. Monotonicity forbids penalizing muting. Resolution: sessions with `audio_muted=false` apply a ×1.10 multiplier to the presence signal for `p_vocal` only (listening to calls is attention to calls); muted sessions apply ×1.00 — never less. Subtle, monotonic, defensible. Recorded in §17.

#### 6.3.4 Rapport — why neglected birds get "quieter" without negative drift

The PRD requires that a user returning after two weeks finds birds "quieter than they were, not birds that have learned to mistrust them." Personality can't do this (monotonic), so the engine carries a separate fast-decaying term:

```
rapport ∈ [0,1],  rapport' = rapport · exp(−Δt / τ) + k · presence_minutes,  τ = 3 days [CALIBRATE], k = 0.02 [CALIBRATE]
```

Rapport modulates **behavioral expressiveness**, not personality: greeting propensity (§6.8), call_energy, perch-forwardness, and offer-approach latency. Two weeks absent → rapport ≈ 0.15 → birds still call, still live, but greet less and sit back more; one good session restores rapport noticeably (half-life days, not weeks). Rapport is canonical (ticked, synced), decays toward zero but never below it, and never touches the `p_*` columns. This cleanly implements "becomes ambient: still alive, still calling, but greeting less often because less often is what's been observed."

#### 6.3.5 Trait → observable mapping (the presentation mapper)

One engine module maps traits to derived presentation/behavior parameters. This is the *only* place traits influence anything a client can see, and its outputs are what appear in snapshots:

| Trait | Canonical behavior effects | Derived render param |
|---|---|---|
| boldness | perch-zone distribution weights (front bias), greeting approach distance, offer-approach latency | — (baked into `pos`, `action`) |
| social warmth | P(greets first), P(calls back in response), perch proximity to other birds | — (baked into greeting directive, `pos`) |
| vocal frequency | `call_rate` base, chorus join propensity | `call.rate` |
| plumage saturation | — | `render.plumage_sat` (palette saturation multiplier 0.75→1.15 and feather-detail layer count 1→3) |
| curiosity | P(investigates offer) by kind, head-tilt rate toward novel events (weather start, falling leaf near perch, song fragment) | — (baked into offer reaction, action weights) |

### 6.4 Mood system (the fast clock)

#### 6.4.1 States

Final v1 enum (PRD delegates finalization to implementation): **`alert`, `curious`, `content`, `wary`, `drowsy`, `resting`**. `resting` is the night state (eyes closed, low on perch); the nightjar-like species substitutes `alert`/`curious` at night (§6.6.3). Mood is stored per bird with `mood_since`.

#### 6.4.2 Transition model

Each tick, per bird, transition propensity to each candidate state is a weighted sum, then a deterministic sampled choice (seeded per §6.2), with strong self-loop bias (moods are sticky; changes read as changes):

```
score(m') = base(m' | species)
          + w_p · personality_term(m', p_*)        # e.g., wary score reduced by boldness; curious raised by curiosity
          + w_t · time_of_day_term(m', local_hour) # drowsy↑ at dusk, alert↑ at dawn, resting↑ at deep night
          + w_r · rapport_term(m', rapport)        # low rapport slightly favors wary/drowsy over curious/alert
          + w_i · interaction_term(m', recent)     # offer accepted → content↑; listen-in → curious↑ briefly; settle → drowsy↑
          + w_a · ambient_term(m', weather)        # rain → vocal damp (content/drowsy↑, alert↓); wind → alert↑ or wary↑ per boldness
          + w_b · contagion_term(m', neighbors)    # neighbor wary → wary↑ (this is the "alarm call" spread); neighbor content → content↑ weakly
P(m') ∝ exp(score(m') / T),  T = 0.6 [CALIBRATE]   # softmax; T tunes how restless the aviary feels
```

Weights table lives in engine config (`mood_weights.json`), versioned, tuned in the calibration lab. The PRD's named cases are test fixtures: high-boldness bird is measurably less likely to enter wary on identical input; passing rain dampens vocal frequency across the aviary for the event's duration + ~30 min; one bird's alarm-class call shifts nearby birds toward wary (contagion applies within one perch zone adjacency).

#### 6.4.3 Daily re-anchor

At each local dawn (account timezone), mood re-anchors: `mood ← sample(personality_baseline)` — the "daily-ish reset cadence." This prevents multi-day mood ruts and makes mornings feel like mornings.

#### 6.4.4 Persistence across sessions

Mood is canonical state; it is exactly what the tick last wrote. A tab opened after a day away shows the mood the interim ticks produced (dusk → drowsy → resting → dawn re-anchor → alert/content). There is no reset-to-neutral path anywhere in the codebase; a CI test asserts the snapshot after simulated absence differs from the seed mood per the time-of-day model ("mood never snaps").

### 6.5 Calls — server activity, client grammar

#### 6.5.1 The split

Calls are procedural and synthesized client-side (non-negotiable, both for aliveness and for the 2MB budget). The server's canonical contribution is **call activity**: per tick, per bird, `call_rate` (calls/minute) and `call_energy` (0–1), computed from `p_vocal`, mood, time-of-day, weather, and rapport:

```
call_rate = species_base_rate · (0.4 + 1.2·p_vocal) · mood_factor(mood) · tod_factor(local_hour) · weather_factor · (0.5 + 0.5·rapport)
```

with `mood_factor`: alert 1.2, curious 1.1, content 1.0, wary 0.5 (and wary calls skew to the species' alarm motif class), drowsy 0.3, resting 0.05 (nightjar-like: 1.0 at night). `tod_factor` peaks at early morning, troughs at midday, secondary peak at dusk, near-zero at deep night (except nightjar-like). `weather_factor`: rain 0.4, wind 0.8, clear 1.0. The client's grammar runtime (§9) turns these parameters into actual calls with device-local variation. Two devices hearing slightly different renderings of the same canonical activity is correct behavior, not a sync bug — the waveform was never canonical.

#### 6.5.2 Species pool (6 species, v1)

Coherent set, "birds you might see in one place." Final art/voice pass is design-owned; engine-relevant parameters per species: silhouette params, palette base, motif library, behavioral quirks.

| key | character | call grammar character | quirk |
|---|---|---|---|
| `warbler` | small, grey-olive, high-perch sitter | bright rising motifs, 2–4 notes | referenced in PRD narration samples; default starter candidate |
| `wren` | small, round, warm brown, low perches | rich fast trills with pauses | default starter candidate |
| `finch` | conical bill, ochre/olive | short rhythmic chips, often doubled | joins choruses readily (high base vocal) |
| `thrush` | larger, warm-breasted, middle perch | fluted phrases, slower, lower rate | strong dawn-calling peak |
| `nightjar` | soft-mottled, ground-low, crepuscular | soft churring/purring motifs | **the night species**: active and calling into late hours when others rest (§6.6.3) |
| `swallow` | slim, forked tail, wire-perch posture | high twittering bursts, aerial | most likely to make short flight transitions between anchors |

Starter selection: two **distinct** species sampled by `rng(account_id, -, adoption, 'starters')`, weighted toward `warbler`/`wren` (the PRD's named example pair) so the marketing/voice samples match common reality. Rarity is not a feature: all six are equally available to the adoption offer.

#### 6.5.3 Recognizability (the seven-bird ceiling's engineering basis)

Each bird's **signature** = stable per-bird timbre parameters derived once at adoption from `rng(bird_id, 'signature')` and never re-rolled: base pitch offset (±15% of species base), formant/filter ratios, rhythm bias (note-spacing multiplier), ornament propensity. Signature is independent of mood and drift — mood/drift modulate *what and how often* the bird calls, never *who it sounds like*. This is the invariant behind "a user who has spent two weeks with Pip should know Pip's call by ear." Per-species motif libraries must contain ≥12 motifs across ≥3 classes (contact, alarm, song-fragment-response) so signatures have material to be recognizable through; the recognizability test protocol is §14.4.

### 6.6 Positions, perches, and the day/night signal

#### 6.6.1 Perch choice

Three zones (front/middle/back) × 3–5 anchors each, defined in a scene manifest shared by server and client (single JSON, versioned — positions must mean the same thing on both sides). Per tick, a bird keeps its perch unless a transition is sampled:

```
P(move) = 0.15/tick base · mood modifiers (wary ↑, drowsy ↓, curious ↑) · species modifier (swallow ↑)
zone weights ∝ [front: 0.2 + 0.6·p_boldness·(0.4+0.6·rapport), middle: 0.5, back: 1.1 − 0.5·p_boldness + wary_bonus]
```

Night: all birds bias to their species' roost anchor (low, back-ish) except the nightjar-like species (§6.6.3). Moves are emitted in the snapshot as `move: {to, depart_at, mode}` so clients animate flight/hop rather than teleport (§8.5); `depart_at` is spread across the tick window so birds don't move in lockstep.

#### 6.6.2 Day/night

Lighting follows **account-local time**. The snapshot carries `sun_t` (normalized 0–1 position in the local solar day, computed from account timezone + date) and the client renders the palette gradient from it (§8.6). Canonical mood/call effects use the same `sun_t` server-side, so lighting and behavior always agree. Phases: dawn (0.20–0.27), morning (0.27–0.42), midday (0.42–0.58), afternoon (0.58–0.73), dusk (0.73–0.80), evening (0.80–0.87), night (0.87–0.20). Settle overlays the evening palette regardless of `sun_t` until cleared (§8.5.3).

#### 6.6.3 Night is not dead

During night phase, most birds are `resting`; the nightjar-like species runs its normal activity curve phase-shifted (peak at `sun_t` ≈ 0.9–0.1). If an aviary contains one, the client keeps rendering/animating it and its calls continue at low rate — the scene always has at least one living motion source at night (ambient ornaments also continue).

#### 6.6.4 Timezone semantics (judgment call)

One account-level IANA timezone, used for *both* lighting and mood signals, so "the laptop in the morning and the phone at night show the same aviary in the same mood" is literally true — two devices in different timezones see the same scene, not two local-time scenes. The client sends its device tz with snapshot pulls (`?tz=`) and in `session_meta`; the server updates `accounts.timezone` when the reported tz differs, with 24h hysteresis (a value can change at most once per 24h unless two consecutive sessions agree) to prevent device-hopping flip-flop. Recorded in §17.

### 6.7 Weather scheduler

Pure function of `(account_id, local_day, tick_index)`: each local day, `rng(account_id, day, 'weather')` draws 0–1 rain events (expected ~2/week: per-day P(rain)=0.28) of 5–15 minutes, and soft-wind events (expected ~3/week) of 10–30 minutes. Never thunder, never snow, never anything assertive. Weather state lives in `aviary_state`, appears in snapshots, feeds `ambient_term` in mood transitions (§6.4.2) and `weather_factor` in call activity (§6.5.1), and drives client-side particle rendering (§8.7). Because it's a pure function of the day seed, catch-up replays weather identically — a user who was away during a rain sees its mood after-effects exactly as if they'd watched it fall, and the notebook can honestly say "a passing rain" (§6.10).

### 6.8 Greeting director (the anchor moment)

The greeting is **server-directed, client-executed**. Direction happens at snapshot-pull time (not in the tick) because it must land within the first second or two of the session.

#### 6.8.1 Arrival windows

An *arrival* opens when a snapshot pull occurs and `now − last_presence_end > 10 minutes` **[CALIBRATE]** (or no presence ever). The server mints an `arrival_id` (stored on `aviary_state`), emits `presence_start` bookkeeping, and includes a greeting directive in that pull's response **only**. Subsequent keepalive pulls within the same arrival window return `greeting: null` — a refresh or a second device joining mid-session does not re-greet. The client executes a directive exactly once per `arrival_id` (local dedupe set).

#### 6.8.2 Absence bands (from `last_presence_end`)

| band | absence | greeting form classes |
|---|---|---|
| short | 10min–1h | glance-up-from-preening; brief head-tilt; weight-shift-and-look (no call, or single soft note) |
| medium | 1h–24h | quiet two-note call + look; head-tilt + step toward front perch; soft call, no movement |
| long | >24h | re-orientation: approach flight/hop to front perch + longer call; longer call followed by a second bird's response; full-body turn + call + settle-into-front |

#### 6.8.3 Bird selection and staggering

Greeter weight `∝ (0.3 + p_boldness)² · (0.5 + rapport) · mood_mult` where `mood_mult`: alert 1.3, curious 1.2, content 1.0, wary 0.4, drowsy 0.2, resting 0.0 (except nightjar-like at night: 1.0). Highest-weight bird greets; with P = 0.35·(second bird's warmth) a **responder** is also directed (its response is part of the same directive, not a second greeting). When multiple birds would greet, the directive carries per-bird `offset_ms` — randomized 800–4000ms stagger, never unison (a simultaneous chorus-on-cue would *announce* the arrival; staggering makes it the aviary noticing, one bird at a time). The directive carries `variation_seed`; the client's grammar composes the actual call/motion instance from it, so the greeting is never identical twice — real procedural variation, not three canned variants in rotation. Anti-repeat: the server keeps the last 3 form-classes used per bird (small ring on `aviary_state`) and re-samples on collision.

#### 6.8.4 Directive shape (in snapshot)

```jsonc
"greeting": {
  "arrival_id": "…",
  "absence_band": "long",
  "greeter": { "bird_id": "…", "form": "approach_front_long_call", "offset_ms": 300, "variation_seed": 7714 },
  "responder": { "bird_id": "…", "form": "soft_answer_call", "offset_ms": 2600, "variation_seed": 902 }
}
```

The client executes within 0.3–2s of first paint (the directive is in the first-paint snapshot), and the greeting narration line is queued promptly to the screen-reader path (§10.1). **No textual welcome accompanies it, ever** (§2.3.2).

### 6.9 Offer reaction resolver (synchronous fast path)

Offers need a reaction *now*, not at the next tick, so `POST /v1/birds/{id}/offers` resolves synchronously in the api service using current canonical state — but writes only through the engine's `resolve_offer` pure function and appends the outcome to the event log; the *drift* consequence is folded in by the next tick (single-writer discipline preserved).

Resolution per offer kind, receiving bird = the offer is to the aviary, so the engine picks which bird(s) react: weight `∝ (0.3 + p_curiosity) · mood_mult · proximity` — nearest-and-most-curious bird is the primary reactor; a second bird may watch (no reaction event, just client-side attention: head-tilt, per curiosity).

| kind | mood-shaped outcomes (primary reactor) |
|---|---|
| `seed` | curious/content → `approach` (hop to front perch, peck, 4–8s); wary → `wait_then_approach` (10–25s hesitation, then approach — the PRD's "waits and eventually comes near"); drowsy/resting → `ignore` (may not approach at all); alert → `approach` with faster latency |
| `song` (fragment from library of 6 **[CALIBRATE]** short motifs) | by `p_vocal` × mood: `join_in` (bird answers the fragment, grammar-composed), `call_against` (overlapping counter-call), `go_quiet` (listen posture, call_rate suppressed ~2min), `watch` (head-tilts only) |
| `pool` | `drink`, `bathe`, or `watch` — weighted by curiosity and mood; pool persists in scene ~3–5 min as a render element (snapshot field) so all devices see it |

Cooldown: per-bird, 4 minutes **[CALIBRATE: "a few minutes"]**, stored canonically (`offer_cooldown_until`), surfaced in snapshots so the UI can quiet the affordance without an error. The cooldown exists so curiosity drift can't saturate in a session — with `s_offer` capped per tick (§6.3.1) it's belt-and-braces against button-mashing. Reaction responses include a `narration` line (naturalist, immediate priority — §10.1) and, for caption users, a caption for any call the reaction includes.

### 6.10 Field notebook generator

#### 6.10.1 Voice and content rules

Server-side template grammar (not an LLM, not a third-party service — the privacy commitment forbids sharing per-account interaction data with third parties, and a template grammar is auditable for voice). Every entry: lowercase, present-tense frame (past tense allowed for the day's events per the PRD samples: "pip greeted before wren today"), bird-named, specific to the moment, no exclamation, **no second person**, no numeric trait references, no observations of the *user's* behavior (the interactions.md line: the notebook may write that Pip greeted first today; it may never write that the user has been here every day this week — template subjects are restricted to birds, weather, light, and the scene; a generator lint (§2.3.1) enforces the ban on "you" and on user-activity template slots).

#### 6.10.2 Candidate observations and salience

Each tick, the generator scans a candidate set (cheap, bounded):

| candidate | example output | salience |
|---|---|---|
| greeting-order first-of-week | "tuesday — pip greeted before wren today, first time this week." | 7 |
| weather passed (rain/wind) during or just after presence | "a short rain moved through after lunch. neither bird looked up for long." | 6 |
| long quiet stretch (no calls > 20min during presence) | "a long stretch of quiet this morning. pip preened for several minutes without looking up." | 5 |
| mood-visible day (wary bird, fluffed against cold) | "wren is fluffed against the cool air, watching the back perch. low calls only." | 5 |
| offer reaction worth noting | "the still pool lasted an hour. wren drank twice; pip only watched." | 4 |
| new bird arrived (adoption) | "a third bird came in from the east this evening, and sat back, and watched." | 9 |
| first nightjar night-call of the season / first frost-light morning etc. (seasonal set pieces, ≤1/month) | — | 6 |
| perch-preference shift over the week (slow-trait *felt* observation, phrased behaviorally) | "pip has been sitting forward this week, near the glass of light." | 4 |

Templates are parameterized slots (bird names, species descriptors, zone names, day names, weather nouns) with per-species adjective pools curated by the design owner. Each template has a **cooldown: same template_key may not fire within 7 days** for the same account, and same (template_key, bird) not within 14 days.

#### 6.10.3 Rarity budget

Target: ≈1 entry per 2–4 days for a regularly-visited aviary; more when genuinely noteworthy; sparse-but-not-dead for absent accounts (weather/seasonal candidates still fire occasionally — the notebook is the aviary's life continuing, not a reward for visiting). Mechanism: per tick, highest-salience candidate above threshold enters a lottery with `P(write) = salience_weight · daily_budget_remaining`; daily budget = 1 (hard cap 2/day, and ≥7 total/30 days floor via a "quiet-days" booster that promotes ambient candidates). Parameters **[CALIBRATE]** in the lab against persona runs asserting the entry-rate distribution (§14.2). Entries are written in-tick (same transaction), immutable, never edited or deleted except by account deletion.

### 6.11 Adoption pacing (age-gated offers)

- Gate: `adoption_offer_available = (bird_count < 7) AND (aviary_age ≥ gate_schedule[bird_count])`.
- `gate_schedule` **[CALIBRATE]** initial: bird 3 at 90 days, bird 4 at 180, bird 5 at 270, bird 6 at 365, bird 7 at 450 — matching the PRD's rhythm ("a few months old offers a third bird; a year-old aviary may have grown to five or six"). Server config; changes are design decisions, never engagement-metric-driven (§13.3 forbids the metrics that would tempt it).
- Surface: **pull-only**. The flag rides in snapshots; the account/settings area shows an "aviary" section with the offer when available (user goes to it; it never comes to the user — no toast, no badge, no email, no notebook "announcement"). Optionally, the first notebook entry after availability may use the `new bird arrived` template's *precursor* ("a warbler has been sitting on the far wire these mornings, watching.") at most once — an observation, not a notification. Adoption itself: user names the bird (default suggestions offered), `POST /v1/aviary/adopt`, server assigns species by seeded draw (distinct from existing species where pool allows), bird record created with seeded personality (§6.3.1), response carries the `fly_in` directive — the one and only entrance animation in the product's life (§8.1.4).

### 6.12 Presence accounting

#### 6.12.1 Client gate (exact per concepts.md)

The client evaluates the three-signal conjunction every 15s:
1. `document.visibilityState === 'visible'`, AND
2. `document.hasFocus()`, AND
3. at least one `pointermove` or `keydown` within the activity window **W = 180s [CALIBRATE — "a few minutes, leaning longer"; the lab tunes W against the drift harness]**.

While the gate holds: `presence_ping` every 30s (batched with other events, §7.2). Gate opens → `presence_start` (with the arrival_id from the greeting flow). Gate closes → `presence_end` with reason (`hidden`, `closed`, `lapsed`, `settle`), best-effort via `navigator.sendBeacon` on `visibilitychange`/`pagehide`. Settle and tab-close both end presence identically at the engine level — no penalty asymmetry, no "you didn't settle" anything.

#### 6.12.2 Server validation (honesty defense-in-depth)

The three-signal gate is client-evaluated (only the client can see focus/visibility), so the server validates plausibility and integrates conservatively:
- Ping rate: ≤1 per 20s accepted per account (excess marked `presence_rate_implausible`, dropped — a buggy or malicious client cannot inflate).
- Presence minutes per tick = wall-clock span covered by accepted pings, capped at the tick's real duration; gaps > 90s between pings break the span (a laptop sleeping mid-session doesn't bank phantom minutes).
- Cross-device dedupe: two devices pinging simultaneously contribute the **union** of coverage, never the sum.
- Late uploads: events with `client_ts` older than 6h are rejected (`stale_event`) — presence from a spotty connection is accepted within 6h, beyond that it's indistinguishable from replay.
- Integration into drift uses only validated minutes (§6.3.1), and per-tick signal is capped, so even a perfectly-spoofed client drifts at the calibrated maximum rate — the abuse ceiling is "a very attentive user."
- The overnight-laptop scenario is a named test fixture (§14.3): tab visible, window unfocused, no input → zero presence minutes banked.

---

## 7. Sync model

### 7.1 The whole model in one paragraph

There is one canonical aviary state, in one database, written by one code path (the tick, plus the two sanctioned fast-paths: offer-reaction fields and renames, which never touch traits). Devices are readers and event-sources. A device uploads observations ("I listened in on bird X for 3 minutes"), pulls snapshots ("here is the aviary as of state_version N"), and renders. There is no client-side canonical state, therefore no merge, no reconciliation, no last-write-wins, and no conflict surface in the data layer — the PRD's "there is nothing to sync" is implemented literally. The only "conflict" surfaces users ever see are session/auth errors (expired link, timed-out session), which are matter-of-fact system copy (§5.1), not data conflicts.

### 7.2 Event upload path

- Client maintains an in-memory queue, mirrored to IndexedDB (bounded ring, 500 events) for crash/offline survival.
- Flush triggers: every 5s if non-empty, immediately on high-salience events (offer, settle), and on `visibilitychange→hidden`/`pagehide` via `sendBeacon`.
- Every event carries `event_id` (UUIDv7, client-minted), `type`, `client_ts`, payload. `POST /v1/events` responses report `{accepted, duplicates, rejected}`; duplicates are silent successes (idempotent by constraint); rejected events are dropped from the queue except `rate_limited` (retried with backoff).
- Server assigns `account_seq` at insert (identity column) — **the tick consumes strictly by this sequence**, so processing order is server-receipt order, identical regardless of which device sent what when.

### 7.3 Snapshot pull path

Pull triggers (per PRD): (a) initial page load — via edge-injected first-paint snapshot (§8.1); (b) `visibilitychange → visible`; (c) render-frame gap detection — if `requestAnimationFrame` delta exceeds 30s (laptop suspend/resume), pull; (d) low-frequency keepalive while visible: every 30s **[CALIBRATE]** with `If-None-Match` (304s are the common case and cost ~nothing); (e) after any synchronous interaction response that bumped state (offer reactions return the affected bird's fresh fields inline — no extra pull).

**Interpolation contract:** the client treats snapshots as keyframes. On receiving version N+1: birds whose `pos` changed and carry a `move` block animate along the move (flight/hop path, §8.5); birds whose `action` changed cross-blend over ~1.5s; `call` parameters ramp over ~5s (no audible step in call rate); lighting `sun_t` is continuous by construction. Nothing ever pops. If a pull returns a version jump >2 ticks (resume from suspend), the client skips interpolation and *eases* birds to new positions over 2–3s (a re-orientation, not a teleport) — the aviary "has been running," and the ease reads as the viewer's eye catching up.

### 7.4 Why conflicts are structurally impossible (the no-LWW proof sketch)

The PRD's failure scenario — laptop writes morning drift, phone's stale write clobbers it — requires a client that can write personality state. In this system: (1) no API request body contains a trait field (§5.1, schema-enforced); (2) the only writer of `p_*` columns is the tick under an advisory lock, applying additive deltas computed from the event log in `account_seq` order; (3) deltas are functions of *events*, and events are immutable, deduped, and totally ordered per account — so any two ticks processing the same log prefix produce identical results, and no ordering of device uploads can lose information (a phone event that arrives late is simply consumed by a later tick; it is never "stale" in a way that overwrites anything). The DB trigger of §6.3.2 makes even a compromised server path unable to decrease a trait. Sync correctness therefore reduces to: event log append correctness (Postgres constraints), tick single-writer (advisory lock), and cursor advancement (same transaction as state write). All three are testable invariants (§14.1), not distributed-systems luck.

### 7.5 Failure modes and recovery

| failure | behavior | recovery |
|---|---|---|
| Client offline during session | events queue in IndexedDB; snapshot pulls fail silently; client keeps rendering last snapshot (simulation "continues" visually from last-known parameters — acceptable: canonical truth resumes on reconnect) | flush queue on reconnect; server dedupes; ≤6h-old events accepted |
| Snapshot pull 5xx / timeout | keep rendering last snapshot; retry with backoff (5s, 15s, 45s, then keepalive cadence); after 3 consecutive failures show a dismissible matter-of-fact inline status in the top-bar account area only ("Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch.") — the only error surface inside the aviary view, and it is system-voice, never a toast over the scene | next successful pull resumes; version-jump easing (§7.3) |
| Tick worker crash mid-tick | transaction rolls back; advisory lock releases; scheduler re-enqueues; catch-up loop covers the missed indices | automatic; no state torn |
| Tick backlog (outage > 1h) | scheduler prioritizes accounts with active sessions; dormant accounts catch up lazily on pull (§6.1.3) | backlog drains; equivalence invariant means no correctness debt |
| Redis loss | snapshots re-read from Postgres (slower pulls, ~10min); revocation list rebuilt from DB; scheduler queue rebuilt from `aviary_state.last_tick_at` scan | automatic degradation, no data loss |
| Session revoked mid-session | next snapshot pull 401 → matter-of-fact "Your session timed out. Sign in again to keep watching." Scene freezes on last snapshot behind the auth surface (no event uploads accepted) | re-auth |
| Duplicate tabs (same device) | both pull, both upload; presence dedupe (§6.12.2) prevents double-banking; greeting fires per tab arrival (each tab is a viewing surface — acceptable and unannounced) | none needed |

---

## 8. Frontend rendering pipeline

### 8.1 Boot sequence and the first frame

The central conceit, made concrete: **the first painted frame is the aviary, mid-action — never a load state.**

#### 8.1.1 Edge-injected first paint (<500ms budget path)

1. Browser requests `/`. CDN edge serves the immutable HTML shell with critical CSS inlined (sky gradient + scene silhouette colors, so even byte-one paints a *quiet field*, not white).
2. An edge function reads the session cookie, verifies its Ed25519 signature and expiry locally (no origin round-trip), checks the revocation list (Redis-replicated to edge, 30s TTL), and on success fetches the account's snapshot from the regional edge cache (populated by the api service on every write; stale-while-revalidate 60s) — then **inlines it as `<script>window.__AVIARY_INITIAL__=…</script>` in the HTML**. Authenticated-user HTML is `Cache-Control: private, no-store` at the browser; the shell assets remain immutable-cached.
3. The main bundle (code-split: `core` = engine-types + renderer + audio + scene manifest, everything else lazy) parses, reads `__AVIARY_INITIAL__`, and paints the first frame from it: birds at snapshot positions, in snapshot actions, **at snapshot action phases** (`action_phase: 0.42` means the first frame shows a bird 42% through a preen stroke — mid-action by construction), ambient ornaments already seeded mid-flight (ornament RNG is seeded from `state_version` so leaves are scattered through their paths at t=0, not spawned at the top edge).
4. If a greeting directive is present, it executes on the 0.3–2s timeline (§6.8.4). Audio attempts start on first gesture (§9.6).
5. The full app (notebook, settings, event queue, keepalive) hydrates *after* first paint; none of it blocks the scene.

Budget math: shell HTML + inlined snapshot ≈ 15–25KB; `core` bundle target ≤ 450KB gz (§13.1); edge snapshot fetch p95 < 80ms (regional cache). On mid-tier mobile/4G: DNS+TLS ~150ms, HTML+snapshot ~100ms, core JS parse+first frame ~200ms → **first bird ≈ 450ms p75**, inside the 500ms affective threshold with headroom. Time-to-first-bird is measured as `navigationStart → first bird pixels on canvas` via a renderer callback into the RUM beacon (§13.4).

#### 8.1.2 Cold/slow path — the quiet field

If the edge can't inline a snapshot (auth check slow, cache miss + origin slow, JS still parsing on a low-end device), the screen shows the **quiet field**: the sky gradient at the correct local `sun_t` (computable from device clock + coarse geo tz before any data arrives), soft background foliage, and at most one or two faint motion cues (a single slow leaf). No spinner, no skeleton, no progress bar, no logo animation. The quiet field *is* the loading state and is indistinguishable from a distant view of a calm aviary. When the snapshot lands, birds are simply already there (cross-fade ≤300ms at their scene positions — a fade-in-place, not an entrance).

#### 8.1.3 Signed-out / new-account path

Marketing-free: the sign-in page is a system-voice surface (email field, plain copy). After magic-link verification for a **new** account, onboarding runs: naming screen (naturalist voice — "two birds arrived this morning. they'll want names." with default suggestions [pip, wren, juniper, ash, bramble, sorrel, teal, merrin]) → `POST /v1/aviary/adopt` ×2 → first snapshot returns with `fly_in` directives.

#### 8.1.4 Empty-aviary state and the one fly-in

Between adoption and first render, the aviary is empty — the same quiet field. Then, once in the account's lifetime, the birds enter with a soft fly-in to their starting perches (snapshot `fly_in` directive; ~2.5s eased bezier path from offscreen; no sound beyond a soft wing/call accent if audio is live). After that, the client treats an empty bird list as a rendering error state (quiet field + matter-of-fact reload surface) — the user never sees an empty aviary again.

### 8.2 Scene composition (layers)

Single `<canvas>` (device-pixel-ratio-aware, capped at 2× for fill-rate), composited from cached offscreen layers:

| layer | content | update policy |
|---|---|---|
| L0 sky | vertical gradient from `sun_t` palette keyframes; settle overlay | repaint on lighting change (≈1×/min + continuous slow lerp) |
| L1 background | soft foliage silhouettes, distant branches | static offscreen canvas; parallax factor 0.2 |
| L2 midground | three perch zones + anchors (scene manifest), still pool (when active), birds | birds repaint every frame; perches static-cached; parallax 1.0 |
| L3 foreground | occasional branch/leaf pass-throughs, rain/wind particles | every frame; parallax 1.4; object-pooled |
| L4 DOM overlays | captions, focus rings/proxies, top bar | DOM, outside canvas |

Parallax is pointer-position-driven at ±6px max displacement, heavily damped (lerp 0.03/frame) — "gentle, not parallax-heavy." Disabled entirely in reduced-motion mode. Dirty-rect strategy: L0/L1 cached; per-frame drawing is L2 birds (≤7 × ~40 draw ops each with the pose system) + L3 particles (cap 60) — comfortably 60fps on integrated graphics five years old (§14.5 soak verifies).

**Scene manifest** (`scene@v1.json`, shared with the server, §6.6.1): normalized coordinates for zone bands, anchor points with per-species roost flags, perch geometry (branch paths), foliage placement seeds, responsive rules (§8.10). One source of truth so server `pos` values and client pixels never disagree.

### 8.3 Bird rendering (procedural, personality-keyed)

Each bird is drawn by a **pose system**, not a sprite sheet:

- **Skeleton:** per-species parametric skeleton (body ellipse ratios, neck, head, beak profile, tail fan, leg/talon paths) defined as SVG-path-derived control sets (~4–8KB/species in the manifest).
- **Poses:** each species ships ~14 key poses (perch-stance, preen variants ×3, scan left/right, head-tilt, weight-shift, drowsy-low, resting-eyes-closed, drink, bathe, peck, call-posture ×2, takeoff, land). Poses are control-point sets; the renderer blends between poses (catmull-rom on control points) — this is what makes cross-fade reduced-motion rendering (§8.9) and mid-action phases (§8.1) the same mechanism at different speeds.
- **Plumage:** palette = species base palette modified by `render.plumage_sat` (saturation multiplier 0.75→1.15) and feather-detail layer count (1→3 by plumage). Drift is *visible on the bird* — richer color, more defined feather strokes — without any number surfacing.
- **Signature details:** per-bird deterministic markings (cheek patch, wing bar emphasis) from `rng(bird_id,'markings')` — subtle visual individuality so two same-species birds are distinguishable at a glance (supports the recognizability promise visually as well as aurally).
- **Call posture:** when the client grammar emits a call (§9.3), the pose system plays the call-posture (beak open, body lift) synchronized to the audio envelope; caption (if on) fades in near the bird (§10.5).

### 8.4 Idle micro-motion (the aliveness layer)

Birds are never still in a way that reads as paused. Per bird, a lightweight action scheduler (client-side, frame-driven, seeded device-local but *weighted by canonical state*):

- **Action classes** with mood-shaped weights (the PRD's mapping, as a weight table): wary → scan ×3, weight-shift ×2, preen ×0.5, back-perch bias (position already canonical); content → preen ×3, slow scan, occasional soft call posture; curious → head-tilt ×3, watch-passing-leaf (ornament hook: a falling leaf within the bird's zone triggers a tracked head-tilt), investigate-pool; drowsy → sit-low, fluff, micro-nods (slow), eyes at half; resting → eyes closed, breath-rate sway only; alert → upright scan ×2, quick head turns.
- **Continuous jitter:** every bird carries 2–3 layered noise oscillators (breath: 0.25Hz amplitude 1px; balance: 0.08Hz amplitude 1.5px; head micro-tremor: 0.9Hz amplitude 0.5px) — never zero, never periodic-looking (incommensurate frequencies).
- **Action durations** 3–12s, gaps 1–8s, mood-scaled (drowsy stretches gaps ×2). Between actions, the bird is in perch-stance + jitter, which is itself motion.
- The canonical snapshot's `action`/`phase` anchors the *current* action at pull time (first frame mid-action, and cross-device the same bird is doing the same kind of thing); between snapshots the client scheduler free-runs within the mood weights. This is the render-time-behavior tier (§3.2): canonical enough to read the mood, local enough to never need sync.

### 8.5 Transitions

1. **Perch moves:** snapshot `move` blocks drive eased flight (bezier arc, 1.2–2.5s by distance, species-modified: swallow fast/direct, thrush slower/heavier) or hop (short anchor-to-anchor, 0.4s). Takeoff/land poses bookend the path. `depart_at` staggering (§6.6.1) prevents lockstep.
2. **Mood changes:** no discrete visual event — the action-weight table and posture offsets (wary: body compressed, back-biased; content: relaxed fluff; drowsy: low sit) cross-blend over ~10s. Mood is read from motion, never labeled: **no status icons, no tooltips, no mood text anywhere in the scene** (the moment the user has to be told, the contract fails — the only mood *words* live in narration/captions/notebook prose).
3. **Settle:** lighting lerps to the evening palette over 4–6s, `call_rate`/`call_energy` ramp down (audio §9.5), birds transition to drowsy-weighted actions and drift to roost anchors over ~30s. The aviary "acknowledges the goodbye" as one soft collective settling motion — no text, no sound cue beyond the quieting. **Undo window:** any click/tap/key in the scene within 5s reverses (lighting lerps back over 2s, `settle_undo` event uploaded); after 5s the settled state persists until presence ends or a new interaction event occurs (offer/listen-in clears it, with the reverse transition).
4. **Weather transitions:** 60–90s ease-in of particles + palette desaturation (rain: cool grey wash, leaf stillness; wind: leaf-ripple bursts, sway amplitude up on foliage layers), hold, 60s ease-out. Never abrupt, never assertive.

### 8.6 Day/night palette engine

Palette = keyframed gradient stops at the phase boundaries of §6.6.2 (dawn warm-pink horizon → midday soft blue → dusk amber-rose → night deep blue-grey with warm window-light accents), interpolated continuously from `sun_t`. All scene colors are lerped in OKLCH space (perceptually smooth dawns; avoids the muddy-grey midpoints RGB lerping produces). Night dims to a floor (never black — the scene stays legible; resting birds are visible silhouettes). The palette engine also drives the top-bar chrome tint (§8.11) so chrome never fights the scene. All palette pairs are checked at design-token time for caption/focus contrast (§10.4, §10.6).

### 8.7 Weather rendering

Rain: pooled particle system (≤200 streaks, low-alpha, slight wind slant) + perch-drip accents + a soft darkening wash; birds shift to sheltered anchors (canonical position already handles the move; client adds a subtle "hunched" posture offset while `weather=rain`). Wind: leaf-ripple waves through L1/L2 foliage + increased ornament spawn rate + occasional faster leaf pass-throughs. Both are pure ornaments tier (§3.2) — no per-particle state leaves the device, no tick involvement, matching "no per-leaf state" in the PRD.

### 8.8 Ambient ornaments

Leaves and feathers spawn at slow random intervals (Poisson, λ ≈ 1 per 40–90s **[CALIBRATE]** visually), drift on eased noise paths through L3, exit or settle. Feather "catches the light" via a rotation-dependent highlight. Ornaments can trigger curious-bird head-tilts (§8.4) — the scene and the birds acknowledge each other without any canonical event. Ornaments are the first thing the frame-budget governor sheds (§13.2) on weak hardware; the birds' micro-motion is never shed.

### 8.9 Reduced-motion mode (a designed surface, not a fallback)

Detection: `matchMedia('(prefers-reduced-motion: reduce)')` OR settings override (`on`/`off` beats OS; `auto` follows OS). When active, the renderer switches motion strategy — same scene graph, same canonical state, different visual register:

- **Micro-motion → pose cross-fades:** the jitter oscillators and frame-blended motion stop; birds hold still poses that cross-fade to the next pose over 3–5s (preen becomes a slow dissolve through 3 preen key-poses). The bird is never frozen — it moves like a slow photograph sequence, which is its own quiet aesthetic.
- **Flight → cross-fade between perches:** bird dissolves out at anchor A (0.8s), dissolves in at anchor B (0.8s). No animated paths.
- **Ornaments:** leaf/feather drift removed entirely. Weather: rain reduced to a static soft wash + slow palette shift (no streaks); wind reduced to foliage tint/stillness change.
- **Parallax:** removed.
- **Day/night color shifts:** retained, slowed ×2.
- **Settle transition:** retained as a pure lighting cross-fade (6–8s).
- **Calls/audio:** unchanged, full quality (or captions per settings) — reduced-motion is a *visual* register; the audio spine is untouched.
- **Greeting:** form classes remap to their motionless equivalents (glance → pose cross-fade to head-tilt pose; approach-call → cross-fade to front perch + call posture + call audio/caption). The greeting still *happens*, still varies, still notices — it just doesn't traverse the screen.

This mode ships **with** v1, in the same release as everything else (the PRD is explicit that a later "v1.1 fix" is a failed launch). The pose-cross-fade system is the same blending machinery as normal mode at different time constants — reduced-motion is a rendering *strategy object*, exercised by the same tests, not a bolted-on degradation path.

### 8.10 Responsive scene rules

- The scene is one horizontal composition, no pan/scroll/zoom, all seven birds visible at all viewports, never cropped.
- Layout: the scene manifest defines a 16:9 reference composition; the renderer fits it to the viewport with `contain` semantics horizontally — on narrow viewports (phone portrait, ≥320px), zone bands compress (front/middle/back anchors converge toward center, x-positions re-normalized by a per-zone compression curve) and bird scale steps up slightly (birds stay readable); on wide viewports (≥1440px), inter-perch spacing widens up to a max scene width (beyond which the scene centers with quiet margins — the aviary doesn't become a panorama).
- Top bar height is fixed (44px touch target zone); the scene occupies the remainder. Phone portrait: top bar compresses to icon-only (it already is) with 48px spacing.
- Orientation change / resize: re-fit within one frame (manifest math is closed-form; no re-layout jank), positions re-normalize — birds never leave frame mid-resize.
- Minimum supported: 320×480 viewport; maximum tested: 3440×1440 ultrawide.

### 8.11 Top bar chrome and fade

- Contents, exactly: **offer** affordance, **field notebook** icon, **accessibility settings**, **account/settings**. Nothing else, ever (guardrail: the top-bar component's prop surface is a closed enum; adding an item requires touching the enum, which review will see).
- No badges, no dots, no counts on any icon (§2.3.2 — the notebook doesn't advertise new entries; the visit log doesn't advertise visits).
- Fade: after 3s of cursor stillness, bar opacity lerps to 0.12 over 1.5s; any pointermove/keydown restores to 1.0 over 0.3s. **Keyboard-focus rule:** if focus is within the bar (or a panel is open), the bar is pinned at 1.0 — fading must never hide focus (§10.3). Touch: bar shows on tap near the top edge, auto-fades after 5s.
- Icons are line-art in the palette's warm-neutral, tinted by the palette engine (§8.6); labels appear on hover/focus only (tooltip-style, AA contrast, system-voice microcopy: "Offer", "Field Notebook", "Accessibility", "Account").

### 8.12 Hidden-tab and lifecycle behavior

- `visibilitychange → hidden`: stop `requestAnimationFrame` (rendering halts — battery), suspend the audio graph (ramp master gain to 0 over 150ms first, no click), flush event queue via sendBeacon, emit `presence_end` if the gate closes.
- `→ visible`: pull snapshot (§7.3 trigger b), resume rAF, ease into the fresh state (version-jump rule), audio resumes on next gesture if the browser suspended the context.
- Laptop suspend/resume: rAF gap detector (>30s delta) triggers the same pull-and-ease path.
- The simulation never depended on any of this — the server ticked regardless; the client's job on return is only to *catch the eye up* to a place that has been continuing.

---

## 9. Audio pipeline

### 9.1 WebAudio graph

```
                       ┌──────────── per bird (≤7) ────────────┐
 motif scheduler ─────►│ voice chain: osc bank (2–3 osc) +     │
 (lookahead, §9.3)     │ noise src → species formant filters → │──► birdGain[i] ─┐
                       │ per-note ADSR envelope                │                 │
                       └───────────────────────────────────────┘                 │
 ambient bed (soft room tone, wind when weather=wind) ──────────► ambientGain ────┤
                                                                                 ▼
                                                     masterBus → dynamics(comp) → masterGain → destination
                                                     (analyser tap: level meter for caption sync + diagnostics)
```

- One `AudioContext`, created lazily, resumed per autoplay policy (§9.6). All gains are `AudioParam`s ramped with `setTargetAtTime`/`linearRampToValueAtTime` — never `.value` jumps (no clicks, and the listen-in ramp is a first-class graph operation).
- Voice budget: ≤ 7 birds × 2 simultaneous motifs + 1 ambient bed + 1 song-fragment voice = ≤ 17 concurrent chains. Node lifecycle: every oscillator/source gets `stop()` scheduled at creation and `onended` → disconnect (the no-leak rule, §9.9).
- No `AudioBuffer` sample libraries — synthesis is oscillator+noise+filter only (bundle budget and the no-recorded-audio rule both demand it).

### 9.2 Motif grammar and per-bird signature

- **Species motif library** (data, in the manifest): ≥12 motifs/species across classes `contact`, `alarm`, `song_response`, `chatter`, `night` (nightjar only). A motif = a note sequence: per note `{pitch_ratio, dur_ms, gap_ms, envelope_shape, ornament? (trill/slur/flick), filter_mod?}` — a compact DSL (~30–60 bytes/motif), not audio.
- **Per-bird signature** (stable forever, §6.5.3): `base_pitch_mult` (±15%), `formant_ratios` (2 poles, ±10%), `tempo_mult` (±12%), `ornament_bias` (which ornaments this bird favors), `timbre_mix` (osc blend: sine/triangle/saw + noise ratio). Derived once from `rng(bird_id,'signature')` at adoption, shipped in the manifest extension of the snapshot (tiny), never re-rolled. A user learns *this* bird's voice; mood and drift change delivery, not identity.
- **Mood/drift modulation at emit time:** `call_rate`/`call_energy` from the snapshot (§6.5.1) set the scheduler's rate and amplitude; mood selects the motif-class distribution (wary → alarm-weighted, short; content → contact/song; drowsy → low-energy single notes; curious → chatter + rising contours); energy scales envelope peak and ornament density.
- **Variation (never identical twice):** at emit, each note gets jitter: pitch ±1.5%, dur ±8%, gap ±12%, ornament substitution with P=0.15 from the bird's bias set, and a per-call micro-transposition (±3%) — all device-local RNG. A rolling per-bird history of the last 8 emitted (motif_id, transposition, ornament-mask) fingerprints rejects near-repeats (re-sample, max 3 attempts, then shift transposition) — the "once the user hears the same call twice, exactly the same way, the spell breaks" rule gets a data structure.

### 9.3 Scheduler

Standard WebAudio lookahead pattern: a 25ms-interval scheduler (driven by `setTimeout` but scheduling against `AudioContext.currentTime`) emits notes up to 400ms ahead — sample-accurate timing immune to main-thread jank (frame drops never garble calls). Per bird, call events are drawn from an inhomogeneous Poisson process with rate `call_rate` (per snapshot parameters, ramped smoothly on snapshot updates), time-of-day-shaped within the tick window, plus:

- **Chorus coupling (client-local):** when bird A emits, each other bird with `p_vocal`-derived propensity may join within 0.4–2.0s (rate-weighted); join probability rises with existing overlap (chorus events emerge when two or more high-vocal birds are calling in the same window — the PRD's emergence rule as a coupling term). Coupling is capped (≤3 simultaneous callers) so the chorus stays a chorus, not a wall.
- **Audible call-and-response:** a joining bird prefers `song_response`-class motifs and transposes toward the joiner's signature-relative pitch of the initiator's last note ( Interval ±0 or a species-consonant third) — responses *sound* like responses.
- **Greeting calls** execute from the directive's `variation_seed` (§6.8.4) — the seed feeds the emit-time jitter draw, so the greeting call is unique but canonical in character.

### 9.4 Chorus mixing

Per-bird `birdGain[i]` into a shared `masterBus`. No panning in v1 (the scene is one horizontal plane; stereo position follows x-position subtly — actually implemented: constant-power pan from `pos.x`, ±25% width, because "the warbler on the left sounds slightly left" is free spatial aliveness within the no-panning *scene* rule, which is about the camera, not the stereo field; recorded as a judgment call §17). A gentle bus compressor (2:1, slow attack) glues overlapping calls without pumping. Two procedural voices overlapping never phase-cancel the way two looped recordings do — each note's frequency/timing jitter (§9.2) decorrelates them naturally; this is the engineering content of "stacked loops produce an artifact the ear catches."

### 9.5 Listen-in mix (the re-balance, never a mute)

- Engage (click/tap/Enter on a bird): focused `birdGain` ramps **up** toward +4dB relative over 2.5s (`setTargetAtTime`, time constant 0.8s — the ramp is audible as a slow rise, not a switch); every other `birdGain` ramps **down to the ambient floor** (−12dB, never −∞/mute) over the same curve; `ambientGain` unchanged. The focused bird's scheduler also gets a modest rate lift (×1.15 — listening invites calling; canonical drift separately rewards the attention via `listen_in_*` events).
- Disengage triggers: second activation on the same bird, focus/click on a different bird (listen-in transfers: old ramp down, new ramp up, simultaneously), click on empty scene space, keyboard focus leaving the scene, Escape. All disengages ramp back to the ambient balance over 2.5s.
- The client emits `listen_in_start`/`listen_in_end(duration_ms)` events (§4.3) — the drift signal is the *duration*, integrated server-side.
- Hard cuts are a review-blocking bug: every gain change in the codebase goes through a `rampTo(param, target, seconds)` helper; direct `.value` assignment is lint-banned in the audio module.

### 9.6 Autoplay policy (the honest constraint)

Browsers forbid audible audio before a user gesture on the page — "calls already audible on the first frame" cannot be honored literally on a cold navigation with no prior interaction, and the plan says so plainly rather than pretending. Behavior:

1. `AudioContext` created suspended at boot; the scene renders fully alive visually from frame one.
2. Resume is attempted on the **first** `pointerdown`/`keydown`/`touchstart` anywhere in the document (which in practice is within seconds — the presence gate's activity signals are the same events). On resume: ambient bed fades in over 1.5s, call scheduler starts at its current canonical rate (birds are mid-conversation, not starting from silence — the first call typically lands within a few seconds).
3. Returning users with browser autoplay allowances (MEI/high engagement) get audio from frame one where the browser permits — the resume attempt simply succeeds immediately after a prior-session gesture pattern.
4. If the context is blocked or unavailable → §9.8 fallback. **No interstitial, no "tap to enable sound" modal, no announcement** — the mute/audio state lives in accessibility settings (system-voice), and the top bar never grows an audio icon (§8.11 closed enum). The tradeoff (audio begins at first touch rather than at first paint) is recorded in §17 as a platform-forced deviation with the smallest possible affective cost.

### 9.7 Song-fragment library (the `song` offer)

Six short composed fragments **[CALIBRATE: library size]** (~2–4s each: e.g., "three rising notes, pause, two falling"; a fragment is data in the same motif DSL, tagged `offer_song`). On a song offer, the fragment plays softly through a dedicated voice (gentle lowpass, −6dB below bird level) from the scene's center; reacting birds' responses (§6.9) are grammar-composed against it (`join_in` uses `song_response` motifs transposed to the fragment's key). Captions for fragments follow §9.7→§10.5 (caption text generated from the same DSL descriptor: "a soft three-note rise").

### 9.8 Fallback: graceful silence + captions

If `AudioContext` is unavailable (feature-detect at boot), construction throws, or the context enters a permanent error state: the audio subsystem reports `audio_available=false` (also sent in `session_meta`), **captions default to ON** (overriding the stored setting, with the accessibility-settings surface showing why in matter-of-fact copy: "Audio isn't available in this browser. Captions are on."), and the aviary runs in graceful silence. Call scheduling continues to run *visually*: call postures, caption emission, and narration lines all fire on the same scheduler clock — the caption path is driven by the scheduler, not by the audio graph, so silence loses the sound but keeps the *events*. No recorded-audio fallback exists in any form (§2.2). Audio-context runtime errors are counted in aggregate telemetry (§13.4).

### 9.9 Memory discipline (the 30-minute rule, audio side)

- Zero per-call allocation of buffers: there are no buffers — oscillator/filter/gain nodes are cheap, created per note, and destroyed deterministically via `onended → disconnect()` (a node-count assertion runs in the soak test: live audio nodes ≤ 60 at any sample point, flat over 30 minutes).
- The scheduler's lookahead queue is a fixed-size ring (32 slots). Motif history ring buffers (§9.2) are fixed (8/bird). No closures capturing scene state per note (lint rule + heap-snapshot diff in CI, §14.5).
- One `AudioContext` for the page lifetime; suspend (not close) on hidden tab; close only on full teardown.

---

## 10. Accessibility surfaces

Design stance (from the PRD, restated once because it governs every decision below): accessibility here is a **designed surface, not a parity checklist**. A screen-reader user gets an aviary that feels alive in prose; a reduced-motion user gets a calmer rendering with its own charm (§8.9); an audio-off user gets captions in the product's voice. All of it ships with v1.

### 10.1 Screen-reader narration

- **Content:** running naturalist prose describing the scene as a place, generated **server-side** by the same template-grammar family as the notebook (§6.10) from the same canonical state the visual surface reads — one voice everywhere. Judgment call (§17): server generation guarantees voice consistency and keeps the client thin; the snapshot carries the next 2–3 scheduled lines with offsets.
  - Example lines: "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
  - Never state-lists, never "Pip is at perch 2," never mood labels as labels ("Wren mood: content" is banned; "wren sits fluffed against the cool air, quiet today" is the register).
- **Cadence:** one prose update per 30–60s at idle (snapshot `narration.lines` carry `at_offset_s`; the client paces them into the live region). The line generator describes what has *been happening*, composed from current canonical state + recent render-time activity the client reports nothing about — the server writes from tick state alone; the client only schedules.
- **Priority bumps:** user-initiated events narrate promptly (queue-jump, not interrupt): the return-greeting on session start ("pip looks up from the low perch and calls once, toward the glass"), offer reactions (delivered inline in the offer response), settle ("the light goes to evening. the birds settle."). These are observations, not confirmations — no "offer accepted" anywhere.
- **Mechanism:** a single `aria-live="polite"` region (visually hidden unless the user enables "show narration text" in accessibility settings — a real option, since some users read better than they listen). Polite, never `assertive`: the product does not interrupt people; the priority bump is queue ordering. One line per region update; the client rate-limits to ≥20s between updates regardless of event burst (a queue absorbs bursts — rapid offers can't machine-gun the SR).
- **Verbosity setting:** `normal` / `sparse` (≈1 line per 2–3 min) in accessibility settings.

### 10.2 The accessible scene model (focus proxies)

Canvas content is invisible to assistive tech, so the scene gets a DOM proxy layer:

- Per bird, an invisible `<button>` positioned over the bird's current scene location (updated at 4Hz from the render loop — cheap, and pointer-target accuracy only needs to be roughly right since keyboard users never use these positions; screen-reader users get labels, mouse users get the canvas hit-testing).
- `aria-label` composed at snapshot cadence (not per frame — label churn is SR noise): "pip — a small grey warbler on the front perch, calling softly" / "wren — perched further back, feathers fluffed, quiet." Composed from the same presentation mapper outputs the narration uses (§6.3.5), so label, narration, and caption never contradict.
- The canvas itself: `role="img"` with a static long-description ("the aviary: a horizontal morning scene with three perches…") plus `aria-describedby` pointing at the live narration region. The proxy buttons live in a `role="group" aria-label="birds"` container.
- Activating a proxy button (click or Enter) = listen-in (§10.3). The proxy is the bird, interaction-wise; the canvas is its picture.

### 10.3 Keyboard navigation map

| context | keys | behavior |
|---|---|---|
| page | `Tab` | skip-link ("Skip to aviary") → top bar items (offer → notebook → accessibility → account, left-to-right) → scene group (first bird proxy, in scene x-order) |
| top bar | `Enter`/`Space` | activate item (opens panel; panels trap focus, `Esc` closes and returns focus to the opener) |
| top bar | any key / pointer move | bar pins to full opacity while focus is inside (§8.11) |
| scene | `←`/`→` (and `↑`/`↓` as aliases) | move focus between bird proxies in scene order, wrapping |
| scene | `Enter` | listen-in on the focused bird (toggle: second Enter disengages) |
| scene | `Esc` | exit listen-in; focus stays on the bird |
| offer panel | `Tab`/arrows | choose offer kind (seed / song fragment ×6 / still pool) and, for songs, the fragment; `Enter` offers; reaction narrated + captioned; panel stays open (multiple offers are fine, cooldowns quiet the control with an accessible reason: "wren isn't interested right now" — matter-of-fact-adjacent but in-voice since it's an aviary observation) |
| notebook panel | `Tab`/`PgDn`/arrows | scroll entries (each entry an `<article>` with timestamp); infinite scrollback paginates on demand |
| anywhere | — | every interactive element is reachable without a pointer; no interaction is canvas-only (hit-testing on canvas is a mouse *convenience*, mirrored by proxies) |

### 10.4 Focus indicators

Double-ring treatment on both chrome and scene: inner 2px warm off-white (`#F5EFDD`) + outer 3px deep bark (`#2E2A22`), 2px offset — chosen so the composite reads ≥3:1 against both the brightest midday sky and the dimmest night palette (verified per palette keyframe in the design-token contrast suite, §10.6). On birds, the ring is drawn as a soft ellipse around the proxy position (DOM, so it's the same in reduced-motion — it appears/disappears without animation there). Focus is never removed, never `:focus-visible`-suppressed on the scene proxies (keyboard users are the point).

### 10.5 Call captions

- Opt-in (accessibility settings), auto-enabled on audio fallback (§9.8).
- Text generated **from the motif descriptor at emit time** — the same structured data that schedules the notes renders the words, so the caption always matches what actually played (or what would have played, in silent mode): "a soft three-note rise" / "a low trill, paused, low trill again" / "a single sharp call from the back perch." Templates per motif class × contour × energy, naturalist voice, lowercase.
- Placement: small text near the calling bird (DOM overlay, L4), fades in with the call's attack, out ~1.5s after its end; max 2 simultaneous captions (oldest fades early if exceeded — captions must not become subtitles-for-everything).
- Contrast: caption text on a soft scrim pill (`rgba(24,26,22,0.55)` behind warm off-white text) — ≥4.5:1 against every palette keyframe (token suite verifies).
- Captions are content, not chrome: they appear inside the scene area, but only when enabled, and they carry no interaction.

### 10.6 Contrast and the design system

- All user-copy text (top-bar labels/tooltips, settings, account surfaces, errors, captions, displayed narration) passes **WCAG AA** (4.5:1 body, 3:1 large) — the floor, with the design-system spec (separate doc, visual designer owns) naming actual ratios per surface. This plan's obligation: the token pipeline *tests* ratios automatically — every (foreground, background) token pair used for text is asserted in CI against every dynamic background state (day/night keyframes × weather washes × settle overlay), because the scene behind chrome changes under the product's own logic.
- The aviary scene itself carries no user copy except the top bar and opt-in captions — the contrast surface is chrome-over-scene, and the scrim/pin rules above handle it.
- Reduced-motion, captions, narration-visibility, and verbosity are all persisted in `accounts.settings` and apply across devices (canonical settings; a phone session honors what the laptop set).

### 10.7 Accessibility settings surface

Lives behind the top-bar accessibility icon; system-voice (matter-of-fact) copy throughout — it is a settings surface, the named exception. Controls: reduced motion (auto/on/off), call captions (on/off), show narration text (on/off), narration verbosity (normal/sparse), audio muted (on/off — muting is an accessibility control, not a media chrome), plus a link to the privacy policy (plain text, naming the aggregate telemetry categories and explicitly excluding per-bird interaction state). Changes apply immediately, persist server-side, and are reflected in `session_meta`/settings digest on all devices.

### 10.8 Accessibility CI matrix (regression defense — ships from day one)

1. **axe-core** scan on every route/state (scene, panels, settings, auth surfaces, visitor surface) in Playwright — zero violations gate.
2. **Focus-path assertion tests:** scripted keyboard walks (the §10.3 table as test cases) asserting focus order, listen-in engagement via Enter, Esc exit, panel focus traps, and that focus is never inside a faded top bar.
3. **Live-region tests:** narration updates land in the polite region, ≥20s apart, one line each; priority events jump the queue; verbosity setting changes cadence.
4. **Label-stability tests:** proxy `aria-label`s change only at snapshot cadence; label/narration/caption consistency (same presentation-mapper source asserted by unit tests on the composer).
5. **Contrast-token suite** (§10.6) across palette keyframes.
6. **Manual SR matrix per release** (cannot be automated honestly): NVDA+Firefox, JAWS+Chrome, VoiceOver+Safari (macOS), VoiceOver+iOS Safari — a 20-minute scripted session (arrive, hear greeting narration, tab to a bird, listen in, offer a seed, read two notebook entries, settle) run against the release candidate by a rotating pair; findings block release at severity ≥ major. Budget for this in every release plan — it is the single most-skipped accessibility practice and the PRD names accessibility regressions as a top risk.

---

## 11. Accounts, auth, and privacy engineering

### 11.1 Magic-link sign-in (flow details)

1. `POST /v1/auth/link {email}` → always 204 (no enumeration). Rate limits §5.4. Email normalized (lowercase, unicode NFKC, gmail-style plus-addressing preserved — no provider-specific canonicalization in v1, documented as a known sharp edge).
2. Auth service mints a 256-bit token; stores only `HMAC(token)` with 15-min expiry; emails the link. Email copy is matter-of-fact: subject "Your Pocket Aviary sign-in link"; body: plain sentence, the link, expiry note, "If you didn't request this, you can ignore it."
3. `GET /v1/auth/verify?token=…`: atomic consume (`UPDATE … SET consumed_at=now() WHERE consumed_at IS NULL AND expires_at>now()` — row-level atomicity makes replay impossible); create/load account; issue session (signed cookie + `sessions` row with device label); 302 to `/`. Failure → matter-of-fact surface: "We couldn't sign you in. The link may have expired. Try requesting a new link."
4. New account → account_id UUIDv7 minted, email stored encrypted + hash-indexed, onboarding (§8.1.3).

### 11.2 Sessions

- Cookie: `aviary_session`, HttpOnly, Secure, SameSite=Lax, 30-day sliding expiry; value = signed token (Ed25519) binding `session_id`. Server-side `sessions` rows enable revocation; the edge function checks signature + expiry locally and the revocation list at 30s staleness (§8.1.1) — a revoked session loses first-paint injection within 30s and API access immediately.
- Settings shows active sessions (device label, last seen, revoke button). Revoking another device is immediate for API, ≤30s for edge-injected paint.
- Session timeout mid-write (the PRD's named rare case): event uploads 401 → client queues, shows the matter-of-fact re-auth surface ("Your session timed out. Sign in again to keep watching."), scene freezes on last snapshot behind it; after re-auth, queue flushes (≤6h staleness rule applies).

### 11.3 Email change

`POST /v1/account/email-change` → verification token to the **new** address (24h). Old email remains the sign-in credential until `GET …/email-change/verify` completes the swap transactionally (encrypt new, re-hash, update). If the new address collides with an existing account hash → matter-of-fact refusal ("That email is already used for a Pocket Aviary account.").

### 11.4 Export

`POST /v1/account/export` → job builds JSON (account settings, birds with **current personality vectors** — the one sanctioned numeric exposure, because it is the user's own data leaving through the user's own hands — moods, names, species, notebook entries, visit invitations/log summary), uploads to object storage, emails a 72h signed link. Export emails are system-voice. The export file itself has a header comment block in plain language describing what each field is (the vectors get one honest sentence: "these numbers shape how your birds behave; the product itself never shows them to you").

### 11.5 Deletion (soft → hard)

- `POST /v1/account/delete`: `status='pending_deletion'`, `deletion_requested_at=now()`. Tick stops for the account; snapshot pulls return the recovery surface; any signed-in page shows a persistent, dismissible matter-of-fact bar: "Your account is scheduled for deletion on {date}. [I changed my mind]" — this bar is the *account surface* exception to no-banners (it is system chrome about system state, not an aviary announcement; it exists only in the 30-day window).
- Sign-in during the window → account loads normally with the bar; `POST /v1/account/restore` clears the mark.
- +30 days: hard-delete job (transactional per account): all rows keyed by account_id across every table, Redis keys, object-storage exports; partition-level event-log pruning handles the rest. Backup rotation ≤35 days bounds PITR residue; stated in the privacy policy.

### 11.6 Privacy architecture (the boundary as plumbing)

The PRD's privacy commitment is implemented as structural rules, each with an enforcement point:

| commitment | structural rule | enforcement |
|---|---|---|
| Per-bird interaction events drive only that user's simulation | `event_log` is readable only by the sim service role and the api ingest path; no other DB role has SELECT | Postgres role grants; infra-as-code test asserts grants |
| Never aggregated for model training / population analysis | No ETL, no warehouse connector, no analytics SDK touches the simulation DB. Aggregate metrics originate in two places only: (a) service-level RED metrics (api/sim), (b) client RUM beacons with a fixed schema (§13.4) that contains no account/bird identifiers — beacons are anonymous, sampled, and schema-frozen in code review | metric registry + CI allowlist (§2.3.5); DB grant audit |
| Never shared with third parties | The mail provider sees email addresses and nothing else (no bird data in any email — template lint asserts); no error-reporting payload may include request bodies (scrubber middleware); no third-party scripts on the page (CSP `script-src 'self'` + the edge-injected snapshot script) | CSP, template lints, dependency audit |
| Email is PII, one place only | §4.1: encrypted column + HMAC index, auth-service-only access; every other table/log/queue references `account_id` UUID | schema review rule; a CI grep fails builds on `email` appearing in any non-auth service's code or SQL |
| Privacy policy is plain and specific | Settings links a plain-text policy naming the aggregate categories collected and explicitly excluding per-bird interaction state | content owned by design+legal, shipped with v1 |

Telemetry that *is* collected is §13.4's list, and its schemas are the allowlist's positive half. "Is this account having errors" (allowed: error counts, latency) vs "what is this account's bird doing" (never: no per-account dimension on anything behavioral) is the line, and it lives in the metric registry file, not in anyone's memory.

---

## 12. Social — visit invitations

### 12.1 Invitation lifecycle

```
created (host enters email) ──► outstanding (30d expiry) ──► used (link exchanged → visit session, 7d)
        │                              │                              │
        └── revoked (host) ◄───────────┴──────────── revoked ◄────────┘
                                     expired (unused, 30d) / session expired (7d)
```

- Host: settings → "visits" section (system-voice surface): invite form (email field), outstanding invitations list with revoke buttons, visit log (visitor email as the host typed it, date, approximate duration, most-recent-first).
- Invitation email (matter-of-fact): "{host first-identifier — the host's account display name, which is just the name they signed up with; no email shown} invited you to visit their aviary. [Visit link] — this link works once and expires {date}. Visiting is read-only: you can watch and listen; nothing you do changes the aviary."
- One-time link → `GET /v1/visits/{token}/exchange` marks `first_used_at`, mints a 7-day visit-session cookie scoped to `visit:read:{host_account_id}`. The link itself dies on exchange; the session carries access. (Judgment call §17: strict one-use links would break page refreshes; exchange-once + short session honors "one-time link" in spirit — the *link* is one-time — while not making a visit fragile. Revocation kills sessions instantly regardless.)
- Defaults: visits OFF — nothing exists until a host sends an invite. No onboarding mention, no discovery, no prompts.

### 12.2 The visitor session (render-only)

- Visitor client loads the same shell; edge injects the **host's** snapshot via the visit-session scope (same cache path, minus `account`, minus `greeting`, minus `adoption_offer_available` — §5.2). The visitor sees exactly what the host would see right now: same birds, moods, drift-derived rendering, day/night, weather. No prettification pass exists in the renderer (there is no visitor-mode flag in the scene code at all — the only differences are data omissions and interaction disablement).
- Interaction disablement: the visitor build path hides the top bar except a minimal chrome: a system-voice line "visiting — read-only" and the accessibility icon (visitors get captions/reduced-motion/narration — accessibility is not host-gated). No offer panel, no notebook, no settle, no listen-in (listen-in is an interaction; visitors get the ambient mix only). Scene click/tap does nothing; bird proxies are `aria-disabled` with the read-only line as description; narration still runs (the visitor's screen reader gets the same prose — observation is the whole feature).
- **Zero simulation footprint:** the visitor client uploads no events (the event endpoint rejects visit-scoped cookies at the middleware level — a 403 by construction, not by convention); visitor presence/integration is impossible because the API surface for it doesn't exist in their session. The host's drift comes from the host's presence, period.
- Visitor snapshot pulls every 30s (read-only keepalive); `visit_log.ended_at` ≈ last pull + 60s grace (approximate duration, as specified).

### 12.3 Revocation and expiration surfaces

- Host revokes (outstanding or active): `revoked_at` set on invitation + all its visit sessions; revocation list pushed to edge immediately. Visitor's next snapshot pull (≤30s) → 410 → matter-of-fact surface: "This visit is no longer available. The host has ended it." Unused link revoked → same surface on exchange. Expired (30d unused / 7d session) → "This visit link has expired. Ask your friend to send a new one."
- No host notification on revocation success (absence from the log is the confirmation); no visitor appeal path; expired invitations cannot be revived (host issues a new one).

### 12.4 Visit log and the notification toggle

- Visit log: settings surface, pull-only, no badge anywhere (§8.11). Rows: visitor email (host-supplied), date, approximate duration; plus outstanding invitations with revoke.
- `visit_notifications` toggle (default **off**, never surfaced in onboarding): when on, each *new* visit (first exchange of an invitation) sends the host one email: "{email} visited your aviary on {date}." — matter-of-fact, no bird data, no frequency, no re-engagement copy. Per-visit, not digest (a digest is a product decision we don't need at v1). When off: nothing, ever. There is no in-product visit indicator of any kind.

---

## 13. Performance budgets and observability

### 13.1 Budget table (CI-gated; a red gate blocks merge/deploy)

| budget | limit | measurement | gate |
|---|---|---|---|
| Initial JS bundle (gzipped, `core` chunk set at first paint) | **< 2MB hard cap**; internal target ≤ 600KB, warn at 800KB | bundlesize check on every PR build; asset manifest diff comment | PR check (hard fail at cap; warn at target) |
| Time to first bird (mid-tier mobile profile, throttled 4G) | **< 500ms** (p75 of synthetic fleet; Moto G-class CPU throttle 4×, Fast 4G profile) | synthetic fleet (§13.4) marks `navigationStart → first bird pixels` (renderer callback); RUM confirms in field (aggregate histogram) | synthetic dashboard alarm; release-blocker if p75 > 500ms for 3 consecutive days |
| Idle frame rate (5-year-old mid-range laptop profile) | **60fps sustained over 30 minutes** (frame-time p95 < 20ms, no monotonic degradation) | Playwright soak with CPU throttle 2× + WebGL-less Canvas2D path; frame-timing histogram from rAF deltas | nightly CI; release-blocker on p95 regression > 2ms |
| Client memory (30-minute session) | **no growth** (heap slope < 0.5MB per 30min after 5-min warmup; live audio nodes ≤ 60 flat; DOM node count flat ±5%) | CDP heap sampling in the same soak; heap-snapshot diff (detached-node check) | nightly CI, hard fail — the PRD makes this "a real test in CI, not a guideline" |
| Snapshot payload | ≤ 8KB p99 (7 birds + directives) | api-side size histogram | alarm at 12KB |
| Tick latency (per account-tick, server) | p50 < 150ms, **p99 < 1s; alarm at p99 > 5s** (the PRD's named error budget) | sim-service histogram | pager alarm §13.5 |
| Snapshot delivery (edge-injected first paint) | p95 < 80ms edge-cache hit; p99 < 300ms incl. origin fallback | edge function timing logs (aggregate) | alarm |
| Event ingest | p99 < 200ms; duplicate rate < 2% (higher means client retry bugs) | api histogram | alarm |

Code-splitting plan (keeps `core` small): `core` (shell, scene manifest, renderer, pose data for the 6 species, audio grammar, event queue, snapshot sync); lazy chunks: `notebook-panel`, `settings` (accessibility + account), `visits` (invitation flow + log), `onboarding`, `export`. The offer panel is in `core` (it's a primary interaction). Species pose data ships complete in `core` (it's small, §8.3, and a first-paint bird must never wait on a lazy chunk).

### 13.2 Runtime frame-budget governor (weak-hardware graceful degradation)

A rolling 2s average of frame times drives a three-step governor: p95 frame > 14ms → shed ornaments (particles ×0.5, parallax off); > 18ms sustained 10s → shed more (particles ×0.25, plumage detail layer −1, backdrop foliage static); > 22ms sustained 10s → minimum mode (ornaments off, micro-motion amplitude halved — **bird motion is never fully shed**; a frozen aviary is worse than a slow one). Recovery hysteresis: one step up per 30s below threshold. Governor state is device-local, not telemetry-identified (only aggregate step-histograms are reported).

### 13.3 What we measure — and what we deliberately don't

**Measure (aggregate-only, no account/bird/email dimension anywhere):**

- Operational RED metrics per service: request counts, error rates, latency histograms (snapshot pulls, event ingest, tick compute, offer resolve, edge injection).
- Client RUM (sampled 10%, anonymous beacon, fixed schema): time-to-first-bird histogram, frame-time histograms, governor-step histogram, audio-context error counts, autoplay-resume latency, snapshot pull failure rate, event queue depth at flush, memory-soak opt-in diagnostics (dev/beta builds only).
- Synthetic fleet results (§13.4) from 5 geographies.
- Email deliverability: bounce/complaint rates per template (aggregate, provider-side).
- Product health, qualitatively and consented: support-contact volume and themes; beta-cohort interviews; the team's own dogfood accounts (consented internal accounts, analyzed in the calibration lab like any persona).

**Deliberately don't measure (the privacy boundary at metric-definition level, and the anti-gamification boundary at the same time):**

- No per-account or per-bird telemetry of any kind. No "average drift across accounts" dashboards — population-level analysis of how birds are interacted with is forbidden by the privacy commitment, so **drift calibration runs exclusively on synthetic persona accounts in staging** (§14.2). This is a real constraint on how we tune, and the plan embraces it: the lab, not production, owns calibration.
- No visit-frequency analytics, no session-count-per-user, no retention cohorts keyed to behavior, no funnel analysis through onboarding that could resurrect "days visited" through the back door of a dashboard. Session-duration histograms are anonymized with no per-account dimension (explicitly allowed by the PRD) and are used for capacity planning only.
- No A/B experimentation framework in v1. Every mechanism that could "optimize engagement" is a mechanism that could optimize against the product's soul; v1 tunes via the staging lab and design judgment. (If experimentation is ever reconsidered, it would be a PRD-level decision, not an engineering one.)
- No cross-account statistics that a leaderboard could be built from later — we don't compute them even internally (the PRD's "the architectural absence makes the feature's reappearance harder" is honored by not having the data).
- Data-integrity monitoring stays inside the simulation boundary: the tick verifies per-account invariants (traits monotonic vs. last tick, mood in enum, positions in manifest) and emits **only aggregate counts** ("N accounts failed vector-integrity check this hour" → pager). The per-account detail lives in the sim service's own logs (access-restricted, not telemetry), because "is this account having errors" is allowed and silent vector corruption is the worst failure mode (§16, risk R6).

### 13.4 Synthetic check fleet + RUM pipeline

- **Synthetic fleet:** Playwright runners on a schedule (hourly prod, per-deploy canary) from 5 geographies × 2 device profiles (mid-tier mobile/4G-throttled; 5-year-laptop/2×-CPU-throttle). Each run: cold load → assert first-bird < budget → 5-minute session (greeting observed, listen-in, offer, settle) → frame/memory histograms → beacon to the metrics pipeline. Also runs the visitor path (invite fixture account) and the auth path (magic-link fixture via mailhog-class capture).
- **RUM:** tiny first-party beacon (`/v1/rum`, no cookies, no identifiers, sampled 10%, schema-frozen in the metric registry). CSP allows only first-party. No third-party analytics, ever (§11.6).
- **Dashboards:** operational health only (RED + budgets + alarms). One page, no drill-down by account because there is no account dimension to drill by.

### 13.5 Alarms and error budgets

| alarm | condition | response |
|---|---|---|
| tick-latency | p99 > 5s over 15min (the PRD's named budget) | page on-call; likely causes: backlog after outage, DB contention; runbook: scale workers, check lock contention |
| tick-backlog | queue depth > 10min of ticks for eager set | page; scale workers; lazy catch-up covers dormant accounts automatically |
| snapshot-5xx | error rate > 1% over 10min | page; clients degrade gracefully (§7.5) but this is users seeing the error surface |
| first-bird-synthetic | p75 > 500ms, 3 consecutive hourly runs | ticket (release-blocker if within 48h of deploy — roll back) |
| audio-error-rate | context failures > 5% of sessions (aggregate) | ticket; check browser-version breakdown (fallback path should be catching these — verify caption-default-on rate rose in lockstep) |
| integrity-check | aggregate vector-integrity failures > 0 for 2 consecutive hours | page; sim logs (restricted) for detail; halt deploys of sim service until triaged |
| email-deliverability | bounce > 5% or spam-complaint > 0.1% per template | ticket; auth/magic-link deliverability is the front door |
| memory-soak-CI | nightly slope gate fails | blocks release train (not a page — a gate) |

Error budgets: snapshot availability 99.9%/month; tick freshness (eager set ticked within 2× cadence) 99.5%. Budget exhaustion → feature freeze, reliability sprint. The product tolerates brief degradation gracefully by design (clients render last snapshot), which is why the budgets can be modest rather than heroic.

---

## 14. Testing and calibration strategy

### 14.1 Unit / integration (per workstream)

- **Engine (pure functions — the best-tested code in the repo):** drift deltas (monotonicity fuzz: 10k random event sequences, assert no trait ever decreases; saturation: traits bounded ≤1; calibration points §14.2), mood transitions (fixture assertions for every PRD-named case: boldness dampens wary, rain dampens vocal, alarm contagion within zone adjacency, dusk→drowsy, dawn re-anchor), call-activity curves (golden-value tests per mood×tod×weather cell), weather scheduler (distribution tests over 10k account-days: ~2 rains/week, never assertive durations), greeting director (bird-selection distribution honors boldness×rapport×mood; staggering never unison; anti-repeat ring works; arrival-window dedupe: second pull in window → no directive), notebook generator (voice lints §2.3.1 as unit tests on 10k generated entries: lowercase, no "you", no exclamation, no numbers-as-traits; rarity budget distribution §14.2; template cooldowns), presence integration (dedupe, caps, gap-breaking, union-across-devices).
- **API:** contract tests (OpenAPI frozen schema; the no-trait-fields test §4.4), idempotency (replayed event_ids), rate limits, auth middleware (visit-scope cookies 403 on event endpoints — the structural visitor-footprint test), error-copy voice tests (system surfaces match the matter-of-fact fixtures verbatim from the PRD).
- **Data layer:** migration harness — every migration runs against a fixture DB with 1k synthetic accounts and asserts: bird_id sets invariant, `p_*` columns never altered by migration code, event_log partition integrity. PITR restore drill scripted quarterly.
- **Client:** snapshot interpolation (version-jump easing, no-teleport assertions on position deltas per frame), event queue (offline replay, sendBeacon on hide, dedupe), presence gate (the three-signal conjunction truth table — all 8 combinations, only `vis∧focus∧act` banks), reduced-motion strategy (every motion class has its cross-fade counterpart; a test walks the motion inventory), top-bar fade + focus-pin rule, governor steps.

### 14.2 The drift calibration lab (the PRD's named calibration target, as a harness)

Staging-only. Persona driver replays accelerated sessions against the real tick engine (time compression: 1 simulated day per ~2 real minutes; ticks run at compressed cadence with identical code paths — the engine is time-indexed, §6.1.1, so compression is honest).

**Personas (initial set):**

| persona | pattern |
|---|---|
| `regular` | 10–15min validated presence/day, 6–7 days/week; 1–2 listen-ins (2–4min) most days; offer every 2–3 days |
| `casual` | 3–5min presence, 3–4 days/week; occasional listen-in |
| `devoted` | 30–60min/day presence, heavy listen-in, daily offers (cooldown-bound) |
| `absent_then_back` | regular for 2 weeks, zero for 2 weeks, regular again |
| `night_owl` | presence 22:00–01:00 local |
| `weekend_only` | 45min Sat+Sun, nothing weekdays |
| `muter` | regular, but `audio_muted=true` |
| `background_laptop` (adversarial) | tab visible 12h/day, no focus, no input → **must bank zero presence** |

**Assertions (the calibration targets, executable):**

1. `regular`: ≥0.02 cumulative Δ on the dominant trait by day 7 (**measurable in instruments**); ≥0.06 on ≥2 traits by day 21, with behavioral readouts crossing perceptibility thresholds — greeting-form distribution shifts toward longer forms, front-perch occupancy +15pp, plumage detail layer steps at least once (**visible to the user**).
2. No persona produces per-session visible change: max Δ within any single simulated day < 0.01 on every trait (the "never session-by-session" rule).
3. `absent_then_back`: traits at return == traits at departure (monotonicity, bit-exact); rapport at return ≈ 0.1–0.2; greeting propensity measurably lower than at departure; one week back restores rapport but traits resume from the departure value (no lost drift, no penalty).
4. `devoted` vs `regular` at day 21: devoted ahead, but sub-linearly (saturation working); devoted never exceeds the day-21 regular by more than ~2× (drift must not become a grind optimization).
5. `background_laptop`: zero drift beyond seed; test fails the build on any Δ > 0 (the presence-honesty tripwire at the engine level).
6. Notebook entry rate per persona within target bands (regular: 1 per 2–4 days; devoted: not more than ~2× regular — sparsity preserved even for very active users, the PRD's explicit tuning requirement; absent: ≥1 per 30 days floor via ambient candidates).
7. Mood-life checks: no bird stuck in one mood > 3 simulated days (excluding resting-at-night patterns); wary-contagion events resolve within ~2 simulated hours absent fresh triggers.

Lab outputs a weekly calibration report (trait-velocity curves per persona, entry-rate histograms, mood occupancy) reviewed by design+eng together; η/weights/W/budgets are config (§15.5) tuned against the report. **Production data never enters this loop** (§13.3) — the lab is how we honor "calibration is part of this PRD" without violating the privacy boundary.

### 14.3 Presence-honesty tests (named scenario suite)

The three-signal gate's failure modes, as executable scenarios (Playwright + engine integration): overnight background laptop (zero banked); visible-but-unfocused second monitor (zero); focused-but-idle 4 minutes (banks until W elapses, then lapses — presence "is not lost the moment the user stops moving the mouse"); active watching 45min without clicks but with occasional mouse drift (banks fully — watching with minimal movement is the actual product); two devices simultaneously (union, not sum); sleeping laptop mid-session (gap-break at 90s, no phantom minutes); spoofed client at 10× ping rate (capped at calibrated max); 8-hour-late upload (rejected stale).

### 14.4 Audio recognizability & uncanniness protocol

- **Objective tests:** per-species motif coverage (≥12 motifs, ≥3 classes — manifest lint); variation-space audit (a generator run of 10k calls per bird fixture asserts zero exact-duplicate fingerprints and near-repeat rate < 0.5% within any 100-call window — the §9.2 history ring works); chorus decorrelation (spectral analysis of 1k simulated two-bird overlaps: no sustained comb-filtering signature above threshold — the phase-artifact class the PRD names); signature stability (calls from the same bird fixture across mood/personality sweeps cluster by bird in a timbre-feature space — an embedding-distance test: within-bird distance < between-bird distance at 7-bird density, which is the machine-checkable proxy for "seven is the cap").
- **Human protocol (per release, beta onwards):** blind listening sessions — 5 listeners, 7-bird-density aviary audio rendered from fixtures, tasks: (a) identify which of two birds is calling (recognizability), (b) rate naturalness 1–5 with free-text (uncanniness), (c) spot repeated calls over a 10-minute exposure (loop-detection by ear). Release gates: (a) ≥80% correct at pair level, (b) mean ≥3.5 with no "robotic/repetitive" theme in >1 report, (c) ≤1 spontaneous repeat call-out. Findings feed motif-library and jitter-parameter work.

### 14.5 Memory & frame soak (the 30-minute CI test)

Nightly: 3 profiles (baseline, reduced-motion, audio-fallback) × 30-minute automated sessions (scripted presence: slow pans of attention via listen-in cycling, offers at cooldown cadence, notebook scrolling every 5min to exercise entry virtualization). CDP heap sampling every 30s → linear-regression slope gate (<0.5MB/30min post-warmup); detached-DOM-node count flat; audio-node census flat (§9.9); frame-time histogram gate (§13.1). Notebook scrollback uses windowed rendering (50-entry DOM window, entries released on scroll-out — the PRD's "do not retain references after scroll-out" as an explicit virtualization requirement). Fails → release train blocked.

### 14.6 Accessibility regression suite

§10.8's six layers, all in CI except the manual SR matrix (per release candidate, rotating pair, blocking at severity ≥ major). Additionally: narration-voice lint runs over the *generated output* of the narration grammar in staging nightly (1k generated lines through the §2.3.1 lints) — voice regressions in generated prose are caught as test failures, not by luck.

### 14.7 Determinism / equivalence tests

- **Eager ≡ lazy:** fixture accounts run 30 simulated days under (a) continuous eager ticking and (b) dormant-with-catch-up-at-random-pull-times; final canonical state byte-equal; notebook entry sets equal (entry ids differ, `(ts, template_key, text)` tuples equal). Also the 90-day collapsed-catch-up variant (§6.1.3).
- **Cross-device convergence:** two simulated clients pulling the same account see identical canonical fields at identical `state_version`s (snapshot equality test); their device-local render/audio divergence is asserted *not* to leak into any uploaded event beyond observations.
- **Replay safety:** re-running a tick with the same inputs (crash-recovery simulation) produces identical writes (advisory-lock + cursor semantics tested under injected failures).

---

## 15. Rollout plan

### 15.1 Milestones (assumes ~8–10 engineers, six workstreams §18; weeks from kickoff)

| milestone | weeks | exit criteria |
|---|---|---|
| **M0 foundations** | 0–2 | repos/monorepo + CI gates live (bundle, lints incl. copy-lint and `Math.random` ban, a11y axe pass on a hello-world scene); infra (Terraform) for dev/staging; scene manifest v1 drafted; design tokens + palette keyframes + voice guide (§19.3) published; schema v1 migrated in dev |
| **M1 engine core** | 2–6 | tick loop + drift + mood + presence integration in staging; event ingest + snapshot APIs; auth service (magic link E2E); determinism tests green; calibration lab v0 running `regular`/`background_laptop` personas |
| **M2 client core — the vertical slice** | 3–8 | **week-6 gate: one bird, end to end** — edge-injected snapshot → first frame mid-action < 500ms (synthetic) → mood-shaped idle motion at 60fps → procedural calls with signature + variation → presence pings banking → drift delta visible in lab instruments after accelerated week. Listen-in mix ramps. This slice is the product's thesis in one bird; everything after is breadth |
| **M3 full aviary + surfaces** | 7–12 | 7-bird density (chorus, governor, soak gates); greeting director + execution; offers + reactions + cooldowns; settle + undo; notebook generator + panel; day/night + weather; adoption flow + age gate; onboarding; settings surfaces; multi-device sync verification (two real devices, one account) |
| **M4 accessibility + social** | 10–14 | narration + live regions + verbosity; captions; reduced-motion strategy complete (§8.9 inventory walked); keyboard map complete + focus-path tests; SR matrix first full pass; visits E2E (invite → exchange → read-only → revoke → expire); export/delete/sessions |
| **M5 hardening + calibration** | 13–16 | full persona suite green with tuned config; audio human protocol pass #1; 30-min soaks green on 3 profiles; synthetic fleet from 5 geos green; failure-injection week (kill tick workers mid-tick, drop Redis, expire sessions mid-write, replay events); privacy-boundary audit (DB grants, CSP, metric registry, email templates) signed off |
| **Beta** | 16–20 | closed beta (below); calibration report from 4 weeks of dogfooding + beta support themes; audio protocol pass #2 on release candidate |
| **GA** | ~20–22 | all gates green; runbooks + on-call rotation live; privacy policy published; launch |

### 15.2 Launch stages

1. **Internal dogfood (from M2 onward):** every team member's account is real and daily. The team is the first calibration instrument — the PRD's "feels alive" is a judgment call, and it gets made by people living with it for months, not by a dashboard.
2. **Closed beta (weeks 16–20):** 200–500 invited users (waitlist-free: personal-invite style, matching the product's temperament). Beta builds ship the RUM beacon + opt-in diagnostics. Feedback channels: a single plain-text feedback link in account settings (system-voice) and scheduled interviews (consented, qualitative — §13.3's allowed product-health path). Watch: audio uncanniness reports (the human protocol's themes, in the wild), greeting repetition complaints ("it does the same thing every time" is the canary for variation-budget bugs), narration verbosity complaints, presence-gate surprises (birds feeling "off" after background-tab periods would indicate honesty bugs).
3. **GA:** public sign-up (still magic-link only). No launch marketing mechanics that would strain the no-announcement ethos (no launch-day counters, obviously; no "join N others" social proof — that's a discovery surface wearing a hat).

### 15.3 Birds-per-aviary ramp

- **Per-account:** everyone starts at 2 (system-selected, §6.5.2); the age-gated schedule (§6.11: 3rd at ~90d … 7th at ~450d) is server config. Launch with the conservative schedule; relax toward the PRD's stated rhythm ("a few months → third; a year → five or six") by design judgment across the first year, never by engagement metrics (which we don't collect per-account anyway — §13.3 makes the temptation structurally absent).
- **Fleet-level:** the cap of 7 is an engine invariant (enforced at adopt-time, tested at 7-bird density in every gate — chorus recognizability §14.4, soak, governor). The cap does not ramp; it is the ceiling from day one. The species pool (6) can grow post-v1 via manifest additions — identity continuity is unaffected (species_key is an attribute; bird_id is forever, §4.1.3).
- **Tick-cadence ramp:** launch with eager-set window 72h / dormant sweep 6h; if fleet size surprises us, tune windows via config (the equivalence invariant §6.1.3 means cadence changes are correctness-neutral by construction — a genuinely rare property, and the reason the determinism work is front-loaded in M1).

### 15.4 Instrumented from day one (GA-minus-zero)

Everything in §13.3's "measure" list is live at beta start (not GA): RED metrics, tick latency + backlog, snapshot delivery (edge + origin), RUM (first-bird, frames, governor, audio errors, autoplay-resume latency), synthetic fleet (hourly, 5 geos), email deliverability, integrity-check aggregates, support-contact themes (manual log). The nightly gates (soak, a11y, determinism, calibration lab) run from M1/M2 onward — observability of the *product's promises* (drift calibration, no-memory-growth, 60fps) is built as CI before there are users to disappoint.

### 15.5 Config flags and kill switches (server-side config; no client release needed)

| flag/config | default | purpose |
|---|---|---|
| `tick_cadence_s`, `eager_window_h`, `dormant_sweep_h` | 60 / 72 / 6 | scheduler tuning (correctness-neutral, §15.3) |
| `drift_eta_*`, `drift_weights`, `seed_distribution` | §6.3.1 values | calibration lab owns |
| `presence_activity_window_s` (W) | 180 | the "few minutes, lean longer" knob |
| `presence_ping_interval_s`, caps | 30 / §6.12.2 | honesty tuning |
| `mood_weights.json`, `mood_softmax_T` | §6.4.2 | mood-life tuning |
| `rapport_tau_d`, `rapport_k` | 3 / 0.02 | quietness-on-return feel |
| `notebook_budget`, template cooldowns, salience table | §6.10 | rarity tuning |
| `adoption_gate_schedule` | §6.11 | pacing |
| `greeting_absence_bands`, anti-repeat ring size | §6.8 | anchor-moment feel |
| `audio_jitter_params`, motif history size | §9.2 | variation budget |
| `visits_enabled` | true | feature kill-switch (invitations stop; outstanding sessions honored or revoked — host comms in system voice if killed) |
| `rums_sampling`, `synthetic_fleet_enabled` | 10% / true | observability throttles |
| `reduced_motion_default_override` | auto | emergency a11y posture (e.g., a motion bug in the wild → default everyone to reduced-motion while hotfixing — the accessibility-first escape hatch) |

Every flag change is a reviewed config commit (flags are code-adjacent: versioned, tested against fixture snapshots in CI — a config typo must not be able to zero out η or invert monotonicity; the monotonicity DB trigger of §6.3.2 backstops even that).

---

## 16. Risks and mitigations

Ranked by (impact on the product's thesis × likelihood). The four PRD-named risk areas (drift calibration, sync correctness, audio uncanniness, accessibility regressions) get the deep treatment first.

### R1 — Drift miscalibration (too fast → Tamagotchi; too slow → screensaver) — **the product's central risk**

- *Failure look:* users report "my birds changed overnight" (too fast — the clickable-number feeling even without numbers) or beta interviews reveal nobody can articulate any change after two months (too slow — nothing they do seems to matter). Both kill the thesis; the band between is narrow and the PRD says so explicitly.
- *Why likely:* the signals are noisy human behavior; the instruments-vs-user gap (week 1 vs week 3) is a perceptual claim, not just a numeric one; config drift across a 20-week build.
- *Mitigations:* the calibration lab (§14.2) with executable week-1/week-3 assertions per persona, running from M1 and gated weekly; all η/weights in reviewed config (§15.5) so tuning is a data change, not a code change; asymptotic `(1−trait)` saturation bounds runaway; per-tick signal caps bound abuse and marathon sessions; the per-day Δ < 0.01 assertion makes "single session moves a trait" a build failure; dogfooding from M2 gives humans months of subjective read on the curve before beta; beta interview protocol asks the perception questions directly ("has anything about the birds changed since you started?" — unaided recall is the visible-drift test).
- *Residual:* perception is subjective; if beta says "too slow," the lever is η + rapport responsiveness (fast-feedback term) — tunable without schema change. Deliberate choice: when in doubt, err slow (a screensaver can be sped up in a config commit; a Tamagotchi teaches users the wrong relationship and un-teaching is months).

### R2 — Sync correctness (lost/duplicated/misordered drift; the silent-deletion scenario) — **existential but structurally defended**

- *Failure look:* the PRD's named scenario — morning drift silently clobbered by a stale lunch write; or double-banked presence from two devices inflating drift; or a torn tick leaving mood and position from different indices. All invisible to users until the relationship feels wrong, and invisible to logs by nature.
- *Mitigations (structural, not procedural):* no client write path for state exists (§5.1 schema shape — you cannot build LWW on an API that accepts no state); single writer via advisory lock + transactional cursor advance (§6.1.2); idempotent events by UUID constraint; server-assigned total order (`account_seq`) consumed strictly in order; union-not-sum presence dedupe; determinism + equivalence tests (§14.7) including crash-injection replay; monotonicity DB trigger as the last line (even a server bug cannot decrease a trait); integrity-check aggregates alarm on impossible transitions (§13.3). The failure mode isn't "handled" — it's made unreachable by construction, then tested for unreachability.
- *Residual:* event *loss* (client dies before flush beyond IndexedDB recovery) under-banks presence — acceptable direction (under- vs over-counting: under-counting drifts slower, which is the safe error per R1's err-slow rule).

### R3 — Audio uncanniness (repetition, chorus artifacts, "robotic" timbres) — **high likelihood, high thesis-impact**

- *Failure look:* the spell-break the PRD names — a user hears the same call twice identically, or two birds phase-cancel into a buzz, or a species just sounds wrong; aliveness collapses and "every other feature collapses" with it.
- *Mitigations:* the variation architecture (per-note jitter + micro-transposition + ornament substitution + 8-call fingerprint ring, §9.2) makes exact repeats impossible by construction and near-repeats rate-limited; chorus decorrelation is a spectral test (§14.4), not a hope; signature-stability embedding test guards recognizability at 7-bird density (the cap's machine-checkable basis); ≥12 motifs × 3 classes per species floors the material; the human blind-listening protocol (§14.4) gates every release from beta onward — machines check the invariants, ears check the feel; song-fragment library is *composed* (design-owned), not algorithmically generated (melodic quality is where algorithms betray themselves).
- *Residual:* species-level timbre misses (a species just sounds synthetic) — mitigation is the art process (audio designer owns per-species formant/envelope sets with review checkpoints at M2/M3), and the fallback position is graceful: fewer, better motifs beat more mediocre ones (variation comes from the jitter architecture, not library size alone).

### R4 — Accessibility regressions (narration drifts into state-lists; focus breaks; reduced-motion ships hollow) — **the PRD's named "worse than missing a contrast threshold" failure**

- *Failure look:* narration cadence creeps up until SR users silence it (the system pushing its own accessibility surface aside); a refactor desyncs focus proxies from bird positions; reduced-motion mode rots into "animations off" because nobody on the team uses it; captions stop matching calls after a grammar change.
- *Mitigations:* accessibility is a workstream with an owner from week 0 (§18), not a pass at the end; the CI matrix (§10.8) makes the mechanical layers regressions-proof (focus-path tests are scripted keyboard walks; live-region rate-limiter is unit-tested; label-stability tested); narration and caption text come from the same presentation-mapper source as labels (one composer — divergence is a unit-test failure, §14.6); generated-prose voice lints run nightly over 1k lines (§14.6); the manual SR matrix per release candidate catches what automation can't (blocking at severity ≥ major, budgeted in every release plan); reduced-motion is a strategy object sharing the pose-blend machinery (§8.9) — it cannot rot independently because it *is* the same code at different time constants, and the motion-inventory walk test enumerates every motion class against its counterpart; the `reduced_motion_default_override` kill switch (§15.5) is the emergency posture if a motion bug ships.
- *Residual:* SR-voice quality (prose that passes lints but reads flat) — mitigated by the same dogfooding/beta-interview loop as R1, with SR users explicitly recruited into the beta cohort (a11y community channels; the beta is invite-based, so this is a deliberate invitation list).

### R5 — Presence-honesty bugs (drift inflation across the population) — **silent, systemic, PRD-named**

- *Failure look:* the overnight-laptop class — a laxer gate than specified silently inflates everyone's drift; "feels alive over weeks" becomes "birds change between sessions" fleet-wide; no test catches it because the test suite encodes the same laxity.
- *Mitigations:* the gate is the exact three-signal conjunction with a truth-table test (all 8 combos, §14.1); the server never trusts the client's *conclusion* — it validates plausibility (rate caps, gap-breaks, staleness rejection, union dedupe, §6.12.2) so even a buggy client can't over-bank beyond the calibrated ceiling; the `background_laptop` adversarial persona fails the build on any drift (§14.2); the scenario suite (§14.3) encodes the PRD's named failure cases as executable tests; W (activity window) is config with the lab owning it — leaning longer (180s initial) per the PRD's guidance, since the failure direction of a too-short W (presence lapses while watching still) is *visible in dogfooding* while a too-long W's failure (banking away-time) is not — asymmetric visibility justifies the lean.
- *Residual:* a determined spoofer banks presence at the calibrated max rate — accepted (the abuse ceiling is "very attentive user"; drift saturation bounds the damage; this is a calm product, not an economy).

### R6 — Personality-vector loss / identity discontinuity — **low likelihood, worst-possible impact**

- *Failure look:* a migration, restore, or bug zeroes or swaps vectors; "losing a personality vector amounts to deleting the bird"; a user's three weeks silently un-happen, and — per the PRD — no unit test fails; the user just feels the bird un-reveal itself.
- *Mitigations:* vectors are append-verified every tick (integrity check vs. previous values: monotonic + bounded + not-identically-zero; aggregate alarm §13.3, detail in restricted sim logs); migration harness asserts bird_id-set and vector invariance (§14.1); PITR + quarterly scripted restore drills (restore a day-old vector state into staging and diff against the integrity chain); no admin tooling can write vectors (the only writes are tick deltas and restores); bird identity is UUID-forever with no code path that re-mints (§4.1.3) — species-pool changes, renames, and migrations all preserve it by schema design; backups bounded and deletion-honoring (§4.5) so privacy and durability don't fight.
- *Residual:* total-DB loss = total product loss for affected accounts — standard DR posture (multi-AZ primary, cross-region backup copies, tested restores) is the answer; there is no cleverer one.

### R7 — Autoplay policy vs "calls audible on the first frame" — **guaranteed, platform-imposed**

- *Failure look:* first-time visitors get a visually-alive but silent aviary until they touch something; the PRD's "calls already audible" first-frame claim is partially unachievable in browsers, full stop.
- *Mitigations:* §9.6's honest design — resume on first gesture (seconds away in practice, and the same events the presence gate needs), returning users often get immediate audio via browser autoplay allowances, captions-on-fallback keeps the *events* alive in silence, no interstitial/announcement ever. The deviation is recorded (§17) and the affective gap is minimized rather than papered over: the first *call* typically lands within 2–3s of the first gesture, and the greeting (0.3–2s post-paint) usually coincides with the user's first click/tap anyway — the greeting call is very often the first sound heard, which is the right first sound.
- *Residual:* none beyond the platform's; any plan claiming otherwise is planning fiction.

### R8 — First-frame budget missed on real-world low-end devices — **medium likelihood, thesis-impact**

- *Failure look:* time-to-first-bird exceeds 500ms on the long tail (old Android, slow networks), and the product reads as "loading" exactly where the conceit matters most.
- *Mitigations:* the edge-injection architecture removes the origin round-trip from first paint (§8.1.1); the quiet-field path makes slow starts *diegetically acceptable* (it reads as the aviary catching up, not the product loading — the failure is softened by design, not just prevented); `core` bundle target 600KB with an 800KB warn gate keeps headroom under the 2MB cap; synthetic fleet includes the mid-tier/4G profile hourly with a release-blocking alarm (§13.5); frame governor (§13.2) protects the *runtime* budget on the same tail devices; species pose data in `core` (no lazy chunk can delay a first-paint bird).
- *Residual:* bottom-decile devices/networks will exceed 500ms sometimes — the quiet field is the mitigation that degrades gracefully instead of breaking the conceit.

### R9 — Announcement/gamification creep (the well-meaning contributor) — **near-certain attempts, catastrophic if landed**

- *Failure look:* "just a small toast saying hi," a streak widget in settings, a "your friend visited!" email by default, a notebook entry about the user's visit pattern — each individually small, cumulatively a different product (the PRD's own analysis, which this plan adopts wholesale).
- *Mitigations:* §2.3's guardrails are the answer: no toast component exists in the UI kit; copy lints fail builds on the vocabulary; the top-bar enum is closed; the notebook template grammar has no user-activity slots (the generator *cannot* observe the user — schema-level, like the no-LWW rule); the PR checklist asks the four questions; the visit-notification toggle ships off-by-default with onboarding forbidden from mentioning it; this plan itself (§2.2–2.3) is the artifact a future contributor argues against, and it argues back with the PRD's reasoning embedded.
- *Residual:* product-level decisions above engineering's pay grade (a future PM with a growth mandate) — outside this plan's control; the PRD's "none will be added in any future version that still calls itself Pocket Aviary" is the recorded line in the sand.

### R10 — Notebook/narration voice quality (templates producing flat or awkward prose) — **medium likelihood, high charm-impact**

- *Failure look:* "specific charm engine" outputs read as Mad-Libs; repeated structures become noticeable over weeks (the notebook's own version of the audio-repetition problem); awkward slot fills ("a leaf drifted down past the back perch and neither bird looked up" is the bar; "bird did thing at place" is the failure).
- *Mitigations:* curated template pools with per-species/slot variant depth (≥6 phrasings per slot combination class), template cooldowns (§6.10.2) preventing structural repetition, salience-driven selection so entries cluster on *moments* (the charm is what's chosen, not just how it's phrased), voice lints as tests (§14.6), design-owner review of every template addition (copy is code-reviewed like code), and the dogfood months — the team reads their own notebooks daily from M3, which is the real quality gate.
- *Residual:* template grammar has a ceiling vs. free generation; the ceiling is accepted deliberately (an LLM path would violate the third-party privacy boundary with per-account data, and a local model blows the bundle/server budget — the judgment call is recorded §17).

### R11 — Email deliverability (magic links = the front door) — **medium likelihood, high friction-impact**

- *Failure look:* links land in spam; sign-in conversion craters; the calm first impression becomes "check your spam folder."
- *Mitigations:* reputable transactional provider, SPF/DKIM/DMARC from day one (M0 infra task), plain-text emails (no heavy HTML spam-heuristics), single-purpose templates, deliverability alarm (§13.5), support copy ready ("the link may have expired / check spam / request a new one" — matter-of-fact, per the PRD's own sample), rate limits that also protect sender reputation.
- *Residual:* consumer-provider spam whims; the re-request flow is the mitigation (15-min expiry + unlimited re-requests makes a lost email recoverable in one click).

### R12 — Tick-fleet operational surprises at scale — **low likelihood at v1 scale, medium impact**

- *Failure look:* eager-set grows; worker pool saturates; tick freshness budget erodes; users see stale-but-fine aviaries (graceful) or the p99 alarm pages constantly (not graceful for the team).
- *Mitigations:* the determinism invariant makes cadence a pure cost/feel knob (§15.3) — degrading dormant accounts to lazier sweeps is correctness-neutral; tick cost is tiny (p50 <150ms target, single-account transactions); backlog alarm + autoscaling workers; the collapsed catch-up (§6.1.3) bounds worst-case resume cost after outages; capacity model reviewed at each stage gate (beta → GA sizing from synthetic-fleet + beta load).
- *Residual:* a 100× surprise would push toward sharded workers / partitioned queues — the event-log shape ports (§3.1), so the migration path is known, just not built.

### R13 — Timezone semantics confusion (cross-device lighting/mood mismatch expectations) — **low likelihood, low-medium impact**

- *Failure look:* a user travels, their phone shows "morning" lighting while the account tz says evening; or hysteresis makes tz updates feel laggy.
- *Mitigations:* the account-level tz judgment call (§6.6.4) makes cross-device consistency the *defined* behavior ("same aviary, same mood" is the PRD's own multi-device promise and it wins over per-device local time); hysteresis prevents flip-flop; the settings surface shows the current timezone (system-voice, editable implicitly by travel or explicitly by tapping it — a small "your aviary follows {tz}" line in account settings); dogfooding includes a two-timezone household scenario (common in the team).
- *Residual:* genuinely bicontinental users get one canonical morning — accepted; the alternative (per-device lighting) breaks the shared-aviary promise, which is worse.

### R14 — Visit-link forwarding / host privacy surprise — **low likelihood, medium trust-impact**

- *Failure look:* a visitor forwards their exchanged link; a stranger watches the host's aviary; the host's transparency record (visit log) doesn't reflect who actually watched.
- *Mitigations:* the *link* is one-time (§12.1) — forwarding the email link post-exchange does nothing; the exchanged visit session is cookie-bound (forwarding a session requires forwarding browser cookies — out of threat scope for v1); 7-day session expiry bounds exposure; revocation is immediate and total; the visit log shows dates/durations so anomalies are host-visible; read-only scope means worst case is *being watched*, never being altered — the blast radius is bounded by the feature's own smallness.
- *Residual:* within-session cookie sharing during the 7 days — accepted (v1 threat model: accidental exposure, not adversarial targeting; the PRD's social feature is small and quiet, and so is its risk).

### R15 — Browser-support long tail (Canvas2D/WebAudio quirks across the supported matrix) — **medium likelihood, contained impact**

- *Failure look:* a Safari version's AudioParam ramp behaves differently; a Firefox canvas perf cliff; iOS memory-pressure kills contexts.
- *Mitigations:* the supported matrix (last two majors × 4 browsers) is CI-tested (synthetic fleet runs the matrix weekly; soak runs Chrome/Firefox/Safari profiles); audio-context error telemetry (aggregate) surfaces browser-version-correlated failures; the fallback path (§9.8) degrades audio failures to silence+captions, never to brokenness; the unsupported-browser surface (matter-of-fact, feature-detect at boot: canvas, WebAudio, ES2020, `visibilityState`) catches the rest cleanly.
- *Residual:* iOS Safari background-audio quirks — the hidden-tab suspend policy (§8.12) sidesteps most of it (we never fight for background audio; the product doesn't want it).

---

## 17. Recorded judgment calls (ambiguity log)

The PRD instructs: where something is ambiguous, make a defensible call and note it. Every call this plan makes beyond the spec text, in one place:

| # | ambiguity | call | defense |
|---|---|---|---|
| J1 | Tick runs "whether or not any client is connected" — literally every account every 60s forever? | Eager 60s ticking for accounts active within 72h; dormant accounts on 6h sweeps + deterministic on-demand catch-up at pull time | The equivalence invariant (§6.1.3, tested §14.7) makes the observable behavior identical to universal eager ticking — the aviary a user returns to *is* the aviary that has been running; only the compute bill differs |
| J2 | "The aviary follows the user's local time" — per-device or per-account? | Per-account IANA timezone (client-reported, 24h hysteresis), used for both lighting and mood | The PRD's own multi-device promise is "the same aviary in the same mood" on laptop and phone — per-device local time would show two different scenes and break it (§6.6.4, R13) |
| J3 | "Calls already audible" on the first frame vs browser autoplay policy | AudioContext resumed on first pointer/key gesture; visual aliveness from frame one; captions cover audio-unavailable states | Browsers forbid pre-gesture audio, full stop; the greeting usually coincides with the first gesture, so the first sound heard is typically the greeting call (§9.6, R7) — the smallest honest deviation |
| J4 | Notebook/narration generation: LLM or templates? | Curated server-side template grammars; no generative-ML path in v1 | The privacy commitment forbids sharing per-account interaction data with third parties (rules out hosted LLMs); a local model busts bundle/server budgets; templates are voice-auditable and testable (§6.10, R10) |
| J5 | Mood set "finalized in implementation" | `alert, curious, content, wary, drowsy, resting` | The PRD's four named examples + alert, plus `resting` for the night state the layout file requires ("eyes closed, low on the perch"); six states keep the transition table legible (§6.4.1) |
| J6 | Presence activity window "a few minutes… leaning longer" | W = 180s initial, config-owned by the calibration lab | The lean-longer instruction, plus asymmetric failure visibility: too-short W shows up in dogfooding (presence lapses while watching still), too-long W is silent — so start long and let the lab pull it down if needed (§6.12.1, R5) |
| J7 | Snapshot keepalive "low-frequency" | 30s while visible, with `If-None-Match` (304-dominant) | Small payloads make 30s nearly free; keeps mood/weather/call-parameter freshness well inside the tick cadence so the client never renders stale canonical parameters for long (§7.3) |
| J8 | Scene rendering technology | Canvas 2D with cached offscreen layers; no WebGL | ≤7 birds + ≤60 particles at 60fps is within Canvas2D on the budget hardware *with* the layer-caching discipline (§8.2); WebGL adds bundle, driver risk, and headroom nobody spends; cross-fades (reduced-motion) are native to Canvas2D compositing |
| J9 | Event transport | Postgres partitioned append-only table with per-account identity sequence; no broker in v1 | The log *is* the queue (cursor-consumed by the tick); one fewer moving part, transactional with state writes (no dual-write problem); shape ports to Kafka later with the UUID partition key already PII-free (§3.1) |
| J10 | Offer reaction timing (tick is too slow for a reaction "now") | Synchronous resolve in the api service via the engine's pure `resolve_offer`, writing reaction/cooldown fields + an event-log row; drift folded by the next tick | Reactions feel immediate; single-writer discipline for *traits* is preserved (the fast path cannot touch `p_*` — engine-level and trigger-level enforcement) (§6.9) |
| J11 | Greeting timing (tick can't fire within 1–2s of tab open) | Server *directs* at snapshot-pull time (arrival windows, `arrival_id` dedupe); client *executes* procedurally from the directive | Canonical who/what/why, sub-second when/how (§6.8) — the same canonicality split as everything else; keepalive pulls and second devices never re-greet |
| J12 | "One-time link" for visits vs page refreshes | Link is one-time (exchange consumes it); the exchanged visit session lives 7 days, cookie-bound, instantly revocable | Honors "one-time link" literally at the link layer while not making a visit fragile; revocation semantics unchanged (§12.1, R14) |
| J13 | Visit-notification channel when the opt-in toggle is on | One matter-of-fact email per new visit; no in-product indicator ever | "Not a notification surface" governs the aviary (no bird-state pings, ever); the toggle exists precisely so a host *can* be told about visits — email is the only channel that reaches outside the product, and the in-product path would need a badge, which is banned (§12.4) |
| J14 | Where the age-gated adoption offer surfaces | Pull-only: flag in snapshot → quiet "aviary" section in account settings; optionally one precursor notebook observation; never toast/badge/email | "Offers appear in the user's flow" must square with no announcements and no emails about the aviary — a pull surface with one in-voice observation is the compliant reading (§6.11) |
| J15 | Muting as a drift input (brief lists it; monotonicity forbids penalty) | Unmuted sessions multiply the presence signal for `p_vocal` by ×1.10; muted ×1.00, never less | Honors "whether you mute the calls" as an input while keeping drift monotonic — listening to calls is attention to calls; not listening is neutral, not negative (§6.3.3) |
| J16 | Stereo panning of calls by scene x-position (±25%, constant-power) | Implemented | The no-panning rule governs the *camera/scene* ("adding panning or scrolling would shift attention to geography") — subtle stereo placement is spatial aliveness within a fixed frame, not scene panning; it is removable by config if beta ears disagree (§9.4) |
| J17 | Raw event-log retention | Prune 90 days after tick consumption | Drift is folded into vectors and the notebook is the user-facing record; retention beyond integration serves no product purpose and is privacy-negative (§4.5) |
| J18 | Late-event acceptance window | 6h (older → `stale_event` rejection) | Covers spotty mobile connections and suspend/resume; beyond 6h, late presence is indistinguishable from replay and the honest default is to drop it (§6.12.2) |
| J19 | Personality seed distribution | `N(species_baseline, 0.08)` clamped `[0.15, 0.55]`; PRD delegates ranges/seeds to the simulation service | New birds start unformed so the first weeks feel (invisibly-per-session) formative; the ceiling guarantees drift headroom; the lab owns the values as config (§6.3.1) |
| J20 | Starter species selection | Two distinct species, seeded draw weighted toward warbler/wren | Distinct species maximize first-encounter recognizability (visual + audio); the weighting makes the PRD's own example pair a common reality in voice samples and marketing (§6.5.2) |
| J21 | Error surface inside the aviary view after repeated snapshot failures | A dismissible matter-of-fact inline status in the top-bar account area (not a toast over the scene) | The no-chrome rule governs the *scene*; a persistent silent failure would be worse; system-voice copy in the chrome layer is the named exception's own territory (§7.5) |
| J22 | No A/B experimentation framework in v1 | Deliberate absence | Experimentation infrastructure is engagement-optimization infrastructure; the privacy boundary forbids the per-account behavioral data it needs, and the calibration lab + design judgment are the sanctioned tuning paths (§13.3) |

---

## 18. Workstreams, sequencing, and team shape

Assumed team: 8–10 engineers + audio designer + visual designer + (shared) PM/design-owner-of-voice. Six workstreams; the milestone gates (§15.1) are the integration contracts between them.

| workstream | owns | headcount | critical dependencies |
|---|---|---|---|
| **A — Simulation backend** | engine library (drift, mood, call-activity, weather, greeting director, notebook grammar, presentation mapper), tick workers, scheduler, calibration lab | 2–3 | schema v1 (M0); scene manifest (with C); lab personas (with F's staging infra) |
| **B — Client platform** | app shell, snapshot sync + interpolation contract, event queue + presence gate, onboarding, settings surfaces, PWA-less offline behavior (§7.5) | 2 | snapshot schema frozen at M1 (A); edge function (F) |
| **C — Rendering** | scene manifest, pose system + species art data, layer compositor, idle-motion scheduler, transitions, day/night palette, weather/ornaments, reduced-motion strategy, responsive fit, governor | 2 | manifest joint-ownership with A (positions must mean the same thing both sides); palette tokens (E) |
| **D — Audio** | WebAudio graph, motif DSL + libraries (with audio designer), signature system, scheduler + chorus coupling, listen-in ramps, caption generator, fallback path | 1–2 (+audio designer) | motif DSL shared with caption templates (E reviews voice); call-activity parameters frozen at M1 (A) |
| **E — Accessibility & design system** | narration composer + live-region pacing, focus proxies + keyboard map, captions presentation, contrast token suite + CI checks, copy lints, voice guide (§19.3), SR matrix process | 1–2 (+visual designer) | presentation mapper (A) as the single text-composition source; every workstream consumes its lints |
| **F — Infrastructure & observability** | Terraform, Postgres/Redis, edge function + snapshot cache, auth service, mail integration, CI gates (bundle/soak/a11y/determinism), synthetic fleet, RUM pipeline, metric registry, alarms/runbooks | 1–2 | M0 is theirs; the metric registry is the privacy boundary's enforcement point (§13.3) |

Sequencing notes:

- **The engine library is the spine** — A publishes typed interfaces (snapshot schema, event taxonomy, presentation mapper) in week 2–3; B/C/D build against fixtures before the engine is done. The week-6 vertical slice (M2) is the forcing function: one bird, every layer, real.
- **E is not a phase.** Narration/captions/reduced-motion/keyboard land inside M3/M4 with their features, per the PRD's ship-together rule; E's lints and CI checks exist from M0 so voice and contrast never retrofit.
- **The calibration lab (A+F) starts at M1** with two personas and grows; by M5 it is the weekly tuning instrument (§14.2). No production data, ever.
- **Design-owner-of-voice** reviews: every notebook/narration/caption template, every email, every error string, every settings label. The voice split (§19.3) is a reviewable artifact, not a vibe.

---

## 19. Appendices

### 19.1 Appendix A — example snapshot (2 birds, morning, greeting due)

```json
{
  "v": 1,
  "state_version": 81234,
  "tick_at": "2026-09-06T07:31:00Z",
  "account": { "timezone": "Europe/Berlin", "settings_digest": { "captions": false, "reduced_motion": "auto", "muted": false, "narration": "normal" } },
  "aviary": {
    "lighting": { "phase": "morning", "sun_t": 0.31 },
    "weather": { "state": "clear", "until": null },
    "settled": false,
    "bird_count": 2,
    "adoption_offer_available": false,
    "pool": null
  },
  "birds": [
    {
      "id": "0192c4a1-7b3e-7a11-9f02-4c8d2e19a001",
      "name": "pip", "species": "warbler",
      "mood": "alert", "mood_since": "2026-09-06T05:02:00Z",
      "pos": { "zone": "middle", "anchor": 1, "x": 0.42, "y": 0.55 },
      "action": { "type": "scan", "phase": 0.42, "since": "2026-09-06T07:30:41Z" },
      "move": null,
      "call": { "rate": 2.1, "energy": 0.72, "motif_seed": 41 },
      "render": { "plumage_sat": 0.52, "fluff": 0.1, "pan": -0.1 },
      "signature": { "pitch": 1.06, "formants": [1.02, 0.97], "tempo": 0.94, "ornament_bias": "trill", "timbre_mix": 0.62 },
      "offer_cooldown_until": null,
      "listen_in": false
    },
    {
      "id": "0192c4a1-7b3e-7a11-9f02-4c8d2e19a002",
      "name": "wren", "species": "wren",
      "mood": "drowsy", "mood_since": "2026-09-06T06:40:00Z",
      "pos": { "zone": "back", "anchor": 3, "x": 0.71, "y": 0.48 },
      "action": { "type": "preen", "phase": 0.17, "since": "2026-09-06T07:30:12Z" },
      "move": { "to": { "zone": "middle", "anchor": 2, "x": 0.58, "y": 0.57 }, "depart_at": "2026-09-06T07:33:20Z", "mode": "hop" },
      "call": { "rate": 0.4, "energy": 0.3, "motif_seed": 17 },
      "render": { "plumage_sat": 0.44, "fluff": 0.55, "pan": 0.35 },
      "signature": { "pitch": 0.93, "formants": [0.98, 1.05], "tempo": 1.08, "ornament_bias": "flick", "timbre_mix": 0.41 },
      "offer_cooldown_until": "2026-09-06T07:36:10Z",
      "listen_in": false
    }
  ],
  "greeting": {
    "arrival_id": "0192c9ff-114b-7d22-a013-9e77c1a20b64",
    "absence_band": "medium",
    "greeter": { "bird_id": "0192c4a1-7b3e-7a11-9f02-4c8d2e19a001", "form": "quiet_two_note_call_and_look", "offset_ms": 400, "variation_seed": 7714 },
    "responder": null
  },
  "narration": {
    "lines": [
      { "at_offset_s": 1, "text": "pip is on the middle perch this morning, upright, watching the light come in." },
      { "at_offset_s": 48, "text": "wren preens on the back perch, fluffed against the cool air. a leaf drifts past and neither bird looks up." }
    ]
  },
  "fly_in": null
}
```

### 19.2 Appendix B — example event batch upload

```json
POST /v1/events
{
  "events": [
    { "event_id": "0192ca10-3f21-7b45-9c02-1e5a77d40a01", "type": "presence_start", "client_ts": "2026-09-06T07:31:02Z", "payload": { "arrival_id": "0192c9ff-114b-7d22-a013-9e77c1a20b64" } },
    { "event_id": "0192ca10-3f21-7b45-9c02-1e5a77d40a02", "type": "presence_ping",  "client_ts": "2026-09-06T07:31:32Z", "payload": { "arrival_id": "0192c9ff-…", "gate": { "vis": true, "focus": true, "act": true } } },
    { "event_id": "0192ca10-3f21-7b45-9c02-1e5a77d40a03", "type": "listen_in_start", "client_ts": "2026-09-06T07:32:05Z", "payload": { "bird_id": "0192c4a1-7b3e-7a11-9f02-4c8d2e19a001" } },
    { "event_id": "0192ca10-3f21-7b45-9c02-1e5a77d40a04", "type": "listen_in_end",   "client_ts": "2026-09-06T07:35:11Z", "payload": { "bird_id": "0192c4a1-…", "duration_ms": 186000 } },
    { "event_id": "0192ca10-3f21-7b45-9c02-1e5a77d40a05", "type": "session_meta",    "client_ts": "2026-09-06T07:31:01Z", "payload": { "tz": "Europe/Berlin", "audio_available": true, "reduced_motion_pref": false, "muted": false } }
  ]
}
→ 202 { "accepted": 5, "duplicates": 0, "rejected": [] }
```

Note what is absent: no mood, no trait, no position, no state of any kind. Observations only. (The `offer` event is written server-side by the offer resolver, §6.9, so its shape never traverses a client request body beyond `{kind, song_key?, event_id}`.)

### 19.3 Appendix C — voice rules for engineers (the two-register cheat sheet)

**Naturalist voice** — the aviary, notebook, narration, captions, offer panel microcopy, onboarding naming screen, greeting-adjacent text (of which there is none — the greeting is the bird):

- lowercase by default; present-tense frame; specific to bird and moment; bird-named.
- preferred verbs: notice, perch, settle, listen in, offer, preen, drift, watch.
- no exclamation marks; no second person ("you"); no announcement framing; no numbers-as-traits; no gamification vocabulary (lint-enforced, §2.3.1).
- the bar: "wren is on the low perch this morning, fluffed against the cool air." The floor (build-failing): "Achievement unlocked: First Greeter."

**Matter-of-fact voice** — sign-in, sessions, sync/error surfaces, account settings, accessibility settings, visit management, export/delete, unsupported-browser:

- normal English capitalization; direct; says what happened and what to do; no naturalist phrasing, no warmth pretending to be useful.
- fixtures (verbatim from the PRD, used as test golden strings): "We couldn't sign you in. The link may have expired. Try requesting a new link." / "Your session timed out. Sign in again to keep watching." / "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."
- the rule for new surfaces: any surface where the user engages with the system *as a system* (money, identity, errors, settings) is matter-of-fact. Everything else is naturalist. When in doubt, the copy lint and the voice owner decide — not the engineer's instinct.

### 19.4 Appendix D — definition of done for v1 (release checklist)

1. All CI gates green: bundle (≤ cap, ≤ warn), a11y (axe + focus paths + live regions + contrast tokens), soak (30-min, 3 profiles), determinism/equivalence, monotonicity fuzz, presence truth-table + scenario suite, migration harness, copy lints, metric-registry allowlist.
2. Calibration lab: full persona suite green against current config; weekly report reviewed by design+eng within the last 2 weeks of the release candidate.
3. Audio human protocol pass on the release candidate (§14.4 gates).
4. Manual SR matrix pass on the release candidate (NVDA/JAWS/VoiceOver×2), no unresolved majors (§10.8.6).
5. Synthetic fleet green from all 5 geographies × 2 device profiles for 72h (first-bird p75 < 500ms; frames; audio errors nominal).
6. Failure-injection drill done this release cycle (tick kill, Redis drop, session expiry mid-write, event replay) with runbooks updated.
7. Privacy audit signed: DB grants, CSP, email templates (no bird data), RUM schema, metric registry, privacy policy text current (§11.6).
8. Guardrail review: no toast component exists; top-bar enum unchanged; notebook grammar has no user-activity slots; no endpoint accepts state (§2.3).
9. On-call rotation staffed; alarms (§13.5) firing-tested in staging.
10. Voice review: every template, email, error string, and label reviewed by the design owner of voice this cycle.

---

*End of plan. The product this plan builds is the one the PRD describes: a small, quiet, browser-only aviary that has been continuing without the viewer, notices them when they arrive, and changes over weeks in ways they feel rather than read. Every architectural choice above is in service of that sentence, and every refusal in §2.2 is enforced by a mechanism, not a memo.*
