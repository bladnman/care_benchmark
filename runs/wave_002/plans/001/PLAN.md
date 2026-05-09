# Pocket Aviary — v1 Implementation Plan

This plan turns the Pocket Aviary PRD into an executable build. It is opinionated where the PRD is opinionated and explicit where the PRD leaves engineering discretion. Where the PRD names a calibration target without numbers (e.g. "a few minutes"), this plan picks a defensible value, marks it as a calibration target, and notes how we'll tune it during build.

The plan is organized so a separate engineering team can pick up any section and execute it without further clarification. Sections cross-reference each other where coupling is real (the simulation tick and the sync model are one design problem; the audio pipeline and the call grammar are one design problem); they are kept separate where decisions can be made independently.

A small set of load-bearing rules from the PRD are restated at the top of each section that depends on them, because they are the kind of rule a contributor working in isolation will violate by default.

---

## 1. Scope

### 1.1 In scope for v1

- A single horizontal browser-rendered aviary scene per account, sized to fit any reasonable viewport, with two starter birds at adoption and a hard cap of seven.
- Six bird species in the species pool, each with: distinct silhouette (SVG), default plumage palette (parameterized for plumage-saturation drift), and a procedural call-grammar motif library.
- Bird engine: hidden 5-dim personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity); 5-state mood enum (wary, content, curious, drowsy, alert); presence-driven low-pass drift toward expressive (monotonic, never negative); mood transitions on session events, time of day, weather, and bird-to-bird interaction.
- Server-side simulation tick at 60-second cadence, sole writer of personality state.
- Append-only client-to-server interaction event log; clients never write personality directly.
- Multi-device sync as a property of the server-canonical model — no client-side merge layer.
- Single-user accounts via email magic link (15-minute expiry, single-use); per-device revocable session tokens; verified email change; soft delete with 30-day grace, then hard delete.
- Synthetic UUID for all internal identifiers; email stored once on the account record, encrypted at rest.
- Account export (JSON snapshot, emailed download link).
- Aviary scene: three perch zones (front/middle/back), local-time day/night cycle, ambient weather (rain, wind), ambient leaf/feather drift, no in-scene chrome, top bar with notebook/offer/settle/settings, top-bar fade on cursor stillness.
- Interactions: return-greeting (per-bird, mood- and absence-shaped, procedurally varied), listen-in with gradual mix ramp, three offer types (seed, song fragment, still pool) with per-bird cooldown, settle gesture with 5-second undo.
- Field notebook: auto-generated naturalist entries with sparsity guarantees, read-only, infinite scrollback.
- Visit invitation feature: email-based invite, one-time link, read-only ambient view, revocable, default off, 30-day expiry, optional opt-in visit notification, visit log in settings.
- Procedural call synthesis via WebAudio; chorus mixing; listen-in mix decay; graceful silent fallback with auto-captions if WebAudio unavailable.
- Accessibility as designed surface: screen-reader narration in naturalist prose at 30–60s cadence; reduced-motion mode as a different rendering (cross-fades, not animation-off); call captions generated from the call grammar; full keyboard navigation; WCAG AA contrast on all user copy.
- Performance: <2MB initial JS bundle (gzipped); time-to-first-bird <500ms on mid-tier mobile + 4G; 60fps idle motion on 5-year-old laptop; bounded memory over 30-minute session (CI test).
- Synthetic perf monitoring fleet; aggregate-only RUM; simulation-tick p99 latency alarm at 5s.
- Browser support: last two majors of Chrome, Safari, Firefox, Edge; matter-of-fact unsupported-browser surface for older.
- Privacy: per-bird interaction events never aggregated, never used for training or recommendation, never shared. Architectural separation between simulation DB and analytics warehouse.

### 1.2 Explicitly out of scope (v1)

These are restated from the PRD's `non_goals.md`, `social_optional.md`, and `product_brief.md` so they are visible to anyone implementing this plan:

- Native iOS/Android apps. No native client; no provisioning of the data model for native constraints.
- Any gamification surface: streaks, achievements, badges, levels, scores, "days visited," XP, tiers, milestone celebrations, calendar of green dots. No "harmless" version of any of these.
- Tamagotchi mechanics: no death, no hunger, no decaying happiness meter, no distress states, no negative drift on neglect.
- Social network surfaces beyond the single visit affordance: no profiles, follows, public discovery, leaderboards, comments, friend-of-friend chains, "show off" mode, "your friend visited!" push.
- Push, email, or in-app notifications that summon the user back to the aviary. No "you've been gone" surface anywhere.
- Shared aviaries, multi-aviary accounts, household profiles, customizable scenes, payments.
- "Welcome back" toasts, banners, modals, or any textual welcome on return. The bird greeting is the entire welcome.
- Any user-visible numeric exposure of personality vectors. Not in stats panels, not in debug views, not under any toggle, not for power users.
- Recorded audio fallback for WebAudio. Silence with captions only.

These exclusions are architectural where possible: the aggregate telemetry pipeline is structured so per-bird state cannot be sent through it; the simulation database is not connected to the analytics warehouse; there is no leaderboard table to be quietly exposed later. Refusing these features at the data layer is more durable than refusing them at the UI layer.

---

## 2. Architecture

### 2.1 Service shape

Five services, each with a single responsibility. Smaller is the goal; we resist temptation to merge for "simplicity," because the simulation/sync correctness story leans on each boundary being clean.

1. **Edge / Web** (Node + Fastify behind a CDN). Serves the static client bundle (HTML, JS, CSS, SVGs, fonts). Inlines the initial state-snapshot for the signed-in user as a `<script type="application/json">` block in the HTML response on authenticated requests, so the client can render the first bird without a second round trip. Static assets are versioned and cached at the CDN edge.

2. **Auth** (Node + Fastify). Handles magic-link issuance, link consumption, session token issuance, token revocation, email change verification, account creation and soft/hard delete. Owns the `accounts` table. Stateless except for short-lived magic-link records.

3. **Simulation** (Rust + Tokio). The bird engine. Owns the canonical aviary state per account: birds, personality vectors, mood, mood timers, call timing, perch positions, weather state. Runs the tick scheduler. Sole writer to the `birds` and `aviary_state` tables. Consumes the interaction event log. Exposes a small read API for snapshots and a write API for events.

   Rust is the right choice here, not Node, because the tick is a hot path and we want a memory-bounded long-lived process. Hundreds of thousands of accounts × one tick per minute × non-trivial per-account computation is exactly the workload Rust suits, and the cost of GC pauses on Node would show up first as p99 tick-latency alarms.

4. **API Gateway** (Node + Fastify). Public-facing client API. Validates session tokens with Auth, forwards reads to Simulation (snapshot pulls), writes to the event log (append-only), proxies notebook reads, handles visit-invitation lifecycle, account-export trigger, settings reads/writes. Rate-limited per session.

5. **Notebook Worker** (Python). Periodic job that reads the recent simulation state and the event log for each active account, emits naturalist field-notebook entries against the sparsity guarantee, writes them to the `notebook_entries` table. Python, not Rust, because the entry-generation logic is text-template-heavy and benefits from a flexible templating layer; performance is non-critical (one job per account every few hours at most).

Plus three pieces of shared infrastructure:

- **Postgres** (managed; replica + read replica). Holds: `accounts`, `birds`, `aviary_state`, `personality_vectors`, `interaction_events` (append-only), `notebook_entries`, `visits`, `sessions`. Partitioning on `interaction_events` by `account_uuid` hash.
- **Redis** (managed). Magic-link tokens, session-token verification cache, per-account presence-window state for fast read in the tick.
- **Object storage** (S3-compatible). Static assets (CDN origin), account-export JSON files (signed time-limited URLs).

Telemetry sinks (Prometheus, Loki) are wired to Edge, Auth, API Gateway, Simulation, Notebook Worker. They never receive per-account or per-bird state — see §11.

### 2.2 Client/server split — what runs where

| Concern | Runs on | Rationale |
|---|---|---|
| Personality vector storage and update | Server (Simulation) | Hard rule. Client never owns. |
| Mood storage and transition computation | Server (Simulation) | Server-authoritative state. |
| Call timing decision (when to call) | Server (Simulation) | Driven by mood, vocal frequency, weather, time of day. Client renders the timing it's told. |
| Call audio synthesis (the actual sound) | Client (WebAudio) | Cannot fit recorded audio in 2MB; chorus depends on per-call variation. |
| Idle micro-motion frame interpolation | Client | High frequency; would be wasteful to push from server. Animations are mood-keyed but the local frames are deterministic given mood + bird seed. |
| Day/night palette | Client (uses local timezone) | Local time is the user's time, not the server's. Client computes from `Date()`. Server tags state with `tick_at_utc` and the client maps to local time on render. |
| Weather state | Server | Aviary-global; needs to be the same across the user's devices. |
| Ambient leaf/feather drift | Client | Pure rendering ornaments; no per-leaf state to sync. |
| Presence accounting | Client emits, server validates | Client is the only thing that can observe `visibilityState`, focus, pointer/key activity. Sends presence-pings on a low-frequency cadence; server consumes. |
| Field notebook entries | Server (Notebook Worker) | Sparsity is a global property; entry quality depends on access to the full state. |
| Return-greeting choreography | Server decides which bird greets, client renders | Bird selection (boldness × mood × absence length) is engine-resident; once selected, the client has enough state to render. |

The line is: anything that affects the canonical relationship the user has with their birds — personality, mood, drift, who-greets-first — runs server-side. Anything ephemeral — frame interpolation, ambient ornaments, palette mapping by local time — runs client-side. The split is not "fat client vs thin client"; it's "canonical state vs presentation," and we hold it strictly.

### 2.3 Render pipeline boundary

The client receives a state snapshot, places birds at their canonical positions and motions, and renders forward at 60fps using deterministic per-bird animation generators seeded by `(bird_id, mood, current_motion_phase)`. Between snapshots, the client interpolates positions for moving birds.

Crucially, the client does not extrapolate canonical state — it does not infer "the bird probably moved to the front perch by now"; it waits for the next snapshot. Canonical state moves on the server's tick; the client's job between ticks is presentational continuity, not speculation.

Snapshot delivery is push-on-change rather than pure pull where possible (see §5 — server-sent events for snapshot deltas), with a low-frequency keepalive pull as a fallback for environments where SSE is unreliable.

---

## 3. Data model

All identifiers are synthetic UUIDs generated at row creation. Email appears on `accounts` only.

### 3.1 Tables

```sql
-- accounts: one row per user account
CREATE TABLE accounts (
  uuid UUID PRIMARY KEY,
  email_encrypted BYTEA NOT NULL,         -- encrypted with KMS-managed key
  email_hash BYTEA NOT NULL UNIQUE,        -- HMAC for lookup; not reversible
  created_at TIMESTAMPTZ NOT NULL,
  deleted_at TIMESTAMPTZ,                  -- soft delete; hard delete clears row
  pending_email_encrypted BYTEA,           -- email change in flight
  pending_email_hash BYTEA,
  notify_on_visit BOOLEAN NOT NULL DEFAULT FALSE,
  reduced_motion_pref VARCHAR(16) NOT NULL DEFAULT 'auto',  -- 'auto' | 'on' | 'off'
  captions_pref VARCHAR(8) NOT NULL DEFAULT 'auto',         -- 'auto' | 'on' | 'off'
  audio_pref VARCHAR(8) NOT NULL DEFAULT 'on',              -- 'on' | 'off'
  timezone VARCHAR(64),                                     -- IANA TZ; client sends, server caches
  schema_version INT NOT NULL DEFAULT 1
);

-- birds: bird identity is stable for the life of the account
CREATE TABLE birds (
  uuid UUID PRIMARY KEY,
  account_uuid UUID NOT NULL REFERENCES accounts(uuid) ON DELETE CASCADE,
  species VARCHAR(32) NOT NULL,            -- e.g. 'wren', 'warbler', 'finch', 'thrush', 'sparrow', 'nightjar'
  name VARCHAR(64) NOT NULL,
  adopted_at TIMESTAMPTZ NOT NULL,
  visual_seed BIGINT NOT NULL,             -- per-bird seed for visual variation within species
  call_seed BIGINT NOT NULL,               -- per-bird seed for procedural call grammar
  retired_at TIMESTAMPTZ                   -- never set in v1; reserved for hypothetical future migrations
);

-- personality_vectors: hidden, never client-visible numerically
CREATE TABLE personality_vectors (
  bird_uuid UUID PRIMARY KEY REFERENCES birds(uuid) ON DELETE CASCADE,
  boldness REAL NOT NULL,                  -- normalized [0.0, 1.0]
  social_warmth REAL NOT NULL,
  vocal_frequency REAL NOT NULL,
  plumage_saturation REAL NOT NULL,
  curiosity REAL NOT NULL,
  updated_at TIMESTAMPTZ NOT NULL,
  drift_history_compact BYTEA              -- bounded-size compact summary of historical deltas; NOT raw history
);

-- aviary_state: per-account scene-level state
CREATE TABLE aviary_state (
  account_uuid UUID PRIMARY KEY REFERENCES accounts(uuid) ON DELETE CASCADE,
  weather VARCHAR(16) NOT NULL DEFAULT 'clear',  -- 'clear' | 'rain' | 'wind'
  weather_until TIMESTAMPTZ,
  last_tick_at TIMESTAMPTZ NOT NULL,
  next_third_bird_offer_eligible_at TIMESTAMPTZ
);

-- bird_state: per-bird mood/perch/call state, mutated on tick
CREATE TABLE bird_state (
  bird_uuid UUID PRIMARY KEY REFERENCES birds(uuid) ON DELETE CASCADE,
  mood VARCHAR(16) NOT NULL,                -- 'wary' | 'content' | 'curious' | 'drowsy' | 'alert'
  mood_set_at TIMESTAMPTZ NOT NULL,
  perch VARCHAR(8) NOT NULL,                -- 'front' | 'middle' | 'back'
  perch_set_at TIMESTAMPTZ NOT NULL,
  next_call_at TIMESTAMPTZ,                 -- next scheduled call; NULL when no call planned
  current_action VARCHAR(16) NOT NULL DEFAULT 'idle'  -- 'idle' | 'preening' | 'scanning' | 'tilting' | 'fluffed'
);

-- interaction_events: append-only, source of truth for what the user did
CREATE TABLE interaction_events (
  uuid UUID PRIMARY KEY,
  account_uuid UUID NOT NULL,
  bird_uuid UUID,                           -- NULL for aviary-level events (settle, weather observed)
  event_type VARCHAR(32) NOT NULL,          -- 'presence_ping' | 'listen_in_start' | 'listen_in_end' | 'offer' | 'settle' | 'session_start' | 'session_end' | 'return_greeting_seen'
  event_data JSONB NOT NULL DEFAULT '{}'::jsonb,
  occurred_at TIMESTAMPTZ NOT NULL,
  consumed_by_tick_at TIMESTAMPTZ           -- set when the simulation tick has folded this event in
) PARTITION BY HASH (account_uuid);

CREATE INDEX ON interaction_events (account_uuid, occurred_at);
CREATE INDEX ON interaction_events (consumed_by_tick_at) WHERE consumed_by_tick_at IS NULL;

-- presence_windows: derived from presence_ping events, kept compact
CREATE TABLE presence_windows (
  uuid UUID PRIMARY KEY,
  account_uuid UUID NOT NULL REFERENCES accounts(uuid) ON DELETE CASCADE,
  window_started_at TIMESTAMPTZ NOT NULL,
  window_ended_at TIMESTAMPTZ NOT NULL,
  duration_seconds INT NOT NULL,
  consumed_by_drift_at TIMESTAMPTZ
);

-- notebook_entries: naturalist-prose observations
CREATE TABLE notebook_entries (
  uuid UUID PRIMARY KEY,
  account_uuid UUID NOT NULL REFERENCES accounts(uuid) ON DELETE CASCADE,
  occurred_at TIMESTAMPTZ NOT NULL,
  body_md TEXT NOT NULL                    -- naturalist prose; lowercase; no announcement framing
);
CREATE INDEX ON notebook_entries (account_uuid, occurred_at DESC);

-- visits: invitation lifecycle
CREATE TABLE visits (
  uuid UUID PRIMARY KEY,
  host_account_uuid UUID NOT NULL REFERENCES accounts(uuid) ON DELETE CASCADE,
  visitor_email_encrypted BYTEA NOT NULL,
  visitor_email_hash BYTEA NOT NULL,
  invite_token_hash BYTEA NOT NULL UNIQUE,
  created_at TIMESTAMPTZ NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL,         -- created_at + 30 days
  revoked_at TIMESTAMPTZ,
  last_visited_at TIMESTAMPTZ,
  visit_count INT NOT NULL DEFAULT 0
);

-- sessions: per-device session tokens
CREATE TABLE sessions (
  uuid UUID PRIMARY KEY,
  account_uuid UUID NOT NULL REFERENCES accounts(uuid) ON DELETE CASCADE,
  token_hash BYTEA NOT NULL UNIQUE,
  device_label VARCHAR(64),                 -- best-effort UA-derived; not authoritative
  created_at TIMESTAMPTZ NOT NULL,
  last_used_at TIMESTAMPTZ NOT NULL,
  revoked_at TIMESTAMPTZ
);
```

### 3.2 Personality vector ranges and seeds

- All five traits normalized to `[0.0, 1.0]`. Internal computation uses `f32`; storage `REAL`.
- Newly adopted bird's seed values are drawn from species-specific Gaussians (mean and stddev defined per species in a config file), clipped to `[0.05, 0.55]`. The clip ceiling at 0.55 leaves headroom for drift-up over weeks; the floor at 0.05 keeps the bird from being entirely flat at adoption.
- Plumage saturation is special: it starts low (0.15–0.30) for all species and is the most visibly drift-driven trait. The PRD explicitly says it drifts up with sustained attention; the visible plumage shift over weeks is the headline drift signal, so we calibrate to make this trait the most legible.

### 3.3 Bird-state legality matrix (mood × perch × action)

Not every combination is plausible. The simulation enforces:

- `wary` mood → perch defaults to `back` or `middle`; preferred actions: `scanning`, `fluffed`. Not `preening`.
- `drowsy` mood → preferred actions: `fluffed`, `idle`. Calls are infrequent and short.
- `alert` mood → perch defaults to `middle` or `front`; action: `scanning`, `tilting`. Higher call probability.
- `curious` mood → action: `tilting`. Approaches offers more readily.
- `content` mood → action: `preening` or `idle`. Greets first more readily if also high-warmth.

These constraints are encoded in the simulation's mood-action transition table; they prevent the engine from producing nonsensical states (a wary bird preening unbothered) that would read as broken.

### 3.4 What is deliberately *not* in the data model

- No `streak_days` column. No `total_visits` column. No `xp` column. No `level` column. No `engagement_score` column. We do not compute these elsewhere either.
- No `mood_history` table that lets us replay every mood transition for analytics. The aggregate telemetry boundary in §11 forbids that pipeline; the absence of the table makes its accidental construction harder.
- No "average drift across all accounts" view. Refused at the schema level.
- No `share_publicly` flag on accounts or birds; no `is_featured` column. Public discovery is out of scope.

---

## 4. API surface

All client-facing endpoints are under `https://api.pocketaviary.example/v1/`. The API is JSON-over-HTTPS for control-plane calls, plus one server-sent-events stream for snapshot delivery. We considered WebSocket; the bidirectional model isn't needed (clients write events via POST, reads are server→client), so SSE keeps the surface smaller.

### 4.1 Authentication

```
POST /auth/magic-link
Body: { "email": string }
→ 202 Accepted (always; no leak of "is this email registered")
```

Server creates an `accounts` row if none exists (a new account starts here, with two starter birds adopted on first sign-in completion — see §4.7), generates a 128-bit random token, stores its hash in Redis with 15-minute TTL, emails the link to the user. Rate-limited per email at 3/hour, 10/day.

```
GET /auth/consume?token=<token>&continue=<path>
→ 302 Redirect to /<path> with Set-Cookie: session=<session-token>; Secure; HttpOnly; SameSite=Lax
```

Token is single-use. On consumption: token hash is deleted from Redis, a `sessions` row is created, the session token is set as a `Secure HttpOnly` cookie. If this is the user's first sign-in, the adoption flow triggers (§4.7).

```
POST /auth/sign-out
→ 204
```

Marks the current session revoked. Future requests with that cookie fail.

```
POST /auth/email-change
Body: { "new_email": string }
→ 202 Accepted
```

Sends a verification email to `new_email`. Old email continues to work until verification.

```
POST /account/delete
→ 202 Accepted
```

Sets `accounts.deleted_at = NOW()`. A user signing in during the next 30 days sees a "restore your account" prompt; clicking it clears `deleted_at`. After 30 days a hard-delete worker removes the row and cascades.

### 4.2 State snapshot — pull and stream

The client gets canonical aviary state via two paths:

**Inlined initial snapshot** — On the authenticated `GET /` Edge response, the HTML includes the latest snapshot as JSON. This is what makes time-to-first-bird <500ms achievable: there is no second round trip before the first bird renders.

**SSE stream for updates** —
```
GET /aviary/stream
→ text/event-stream
```
Events on the stream are typed:
- `snapshot:full` — full snapshot, sent on initial connect and on reconnect
- `snapshot:delta` — bird-state changes, weather changes; pushed on tick when something changed
- `event:greeting` — the chosen greeting bird and its greeting form, pushed on session start
- `event:notebook` — a new notebook entry was written
- `keepalive` — sent every 25 seconds so middleboxes don't close idle connections

**Fallback pull** — for environments where SSE is blocked (corporate proxies, etc.):
```
GET /aviary/snapshot
→ 200 { ... full snapshot ... }
```

### 4.3 Snapshot shape

```json
{
  "snapshot_id": "uuid",
  "tick_at_utc": "2026-05-08T14:32:00Z",
  "weather": { "type": "clear", "until_utc": null },
  "next_third_bird_offer_eligible_at": "2026-08-01T00:00:00Z",
  "birds": [
    {
      "id": "uuid",
      "species": "wren",
      "name": "pip",
      "visual_seed": 7142,
      "call_seed": 4120,
      "mood": "content",
      "mood_set_at_utc": "2026-05-08T14:14:00Z",
      "perch": "front",
      "perch_set_at_utc": "2026-05-08T14:30:00Z",
      "current_action": "preening",
      "next_call": {
        "scheduled_utc": "2026-05-08T14:32:18Z",
        "motif_seed": 88291,
        "duration_ms": 1400,
        "intensity": 0.6
      }
    }
  ],
  "render_hints": {
    "plumage_saturation_normalized": [0.42, 0.31]   // ordered to match birds[]; opaque scalar; mapped to a visual ramp client-side
  }
}
```

Notes:
- Personality vectors are not in the snapshot. The only plumage-related field is a normalized scalar that drives the visual ramp; the four other traits affect the snapshot's *contents* (which mood the bird is in, when it next calls, where it perches) but are not numerically exposed. A user inspecting devtools sees no boldness/curiosity/social-warmth/vocal-frequency numbers.
- `render_hints.plumage_saturation_normalized` is the one trait that has to be exposed to render correctly; we expose it as an opaque scalar named for what it visually controls, never as the trait name. This is a hairline judgment call: the rule "personality is never exposed" is held by the absence of all four other traits and by never naming this scalar `plumage_saturation_trait`.
- The SSE delta envelope contains only the changed birds and changed scene-level fields, keyed by `snapshot_id`. Clients reconcile by `snapshot_id` ordering.

### 4.4 Interaction events — write

```
POST /events
Body: {
  "events": [
    { "type": "presence_ping", "occurred_at_utc": "...", "data": { "windowed_active": true } },
    { "type": "listen_in_start", "occurred_at_utc": "...", "bird_id": "uuid" },
    { "type": "offer", "occurred_at_utc": "...", "bird_id": "uuid", "data": { "kind": "seed" } },
    { "type": "settle", "occurred_at_utc": "..." }
  ]
}
→ 202 Accepted { "accepted_count": N }
```

Events are batched. The client buffers and flushes on visibility change, on `beforeunload` (best-effort via `navigator.sendBeacon`), and on a low-frequency timer (every 30 seconds while visible). Server appends to `interaction_events`; the simulation tick consumes them in order on the next pass.

The endpoint never returns "applied" state, only "accepted." The client must rely on the next snapshot from the SSE stream to learn what the simulation made of its events. This enforces server-authoritative state — the client is not allowed to infer "I just sent an offer, the bird must now be content."

### 4.5 Notebook

```
GET /notebook?before=<entry-uuid>&limit=20
→ 200 { "entries": [...], "next": "..." }
```

Read-only. No write endpoint. No edit. No delete.

### 4.6 Visits

```
POST /visits/invitations
Body: { "email": string }
→ 201 { "invitation_id": "uuid", "expires_at_utc": "..." }
```
(Sends an email to the visitor with the invite link.)

```
DELETE /visits/invitations/{id}
→ 204
```
(Revokes the invitation; an active visitor session is terminated at next snapshot pull.)

```
GET /visits/log
→ 200 { "visits": [...], "outstanding_invitations": [...] }
```

```
GET /visit?token=<invite-token>
→ HTML, with inlined read-only snapshot of the host's aviary
```

Visitor's session is read-only at the data layer: the visitor's session token is flagged `readonly`; any POST to `/events` with a read-only token is rejected with 403. The server does not record presence-pings or any interactions from a visitor's session; visitors don't appear in `interaction_events`.

```
GET /visit/stream?token=<invite-token>
→ SSE, snapshot:full + snapshot:delta as for the host
```

### 4.7 Adoption flow (first sign-in)

When a brand-new account completes their first magic-link sign-in:

1. Server selects two species from the pool by deterministic-with-jitter rule (see §5.5).
2. Two `birds` rows are created with stable UUIDs, default name suggestions per species (e.g. "pip" for a small bird, "wren" for a wren).
3. Two `personality_vectors` rows are created with seed values per §3.2.
4. The user is shown the naming UI: two birds, default names pre-filled, editable.
5. On confirm, the aviary scene appears with the empty-aviary quiet field briefly, then each bird flies in to its starting perch with a soft animation.
6. Subsequent sessions never show empty-aviary again.

### 4.8 Account export

```
POST /account/export
→ 202 Accepted
```

Async. A worker generates a JSON snapshot (birds, names, personality vectors, moods, notebook entries, settings), uploads it to S3, and emails the user a signed URL valid for 7 days.

Yes, the export contains personality vectors. The PRD explicitly lists them in the export contents. The rule is "the user never sees them in product surfaces"; the export is a data-portability surface, not a product surface, and exporting them is correct because they are the user's data. The export is the one place the numbers leak — and they leak to the user, into a JSON file they downloaded — which is consistent with the PRD's stance.

### 4.9 Settings

```
GET /settings
PUT /settings
Body: { "audio": "on"|"off", "captions": "on"|"off"|"auto", "reduced_motion": "on"|"off"|"auto", "notify_on_visit": bool }
```

```
GET /sessions  → list of devices
DELETE /sessions/{id}  → revoke that device's session
```

### 4.10 Voice-of-API

All system-side responses (`4xx`, `5xx`, settings surfaces, sign-in, visit-revocation surfaces) are written in matter-of-fact tone. The client renders error messages from the body's `message` field; we deliberately do not let backend developers slip naturalist phrasing into error responses. Lint rule + a small string-style guide enforces this.

---

## 5. Simulation engine design

This section is the longest because it is the load-bearing core. The PRD calls out specific calibration targets and rules; this plan turns them into a concrete tick algorithm.

### 5.1 Tick cadence and execution model

- **Cadence**: 60 seconds. Configurable; we expect to tune in the 30–90 second range during build.
- **Execution**: a global tick scheduler partitions accounts across N tick workers (target N=8 for v1). Each worker holds a batch of accounts in memory for its tick pass and processes them sequentially.
- **Tick is idempotent**: re-running a tick for the same `(account_uuid, tick_at_utc)` produces the same state. Achieved by writing `aviary_state.last_tick_at` and rejecting older ticks; combined with the consumed-event-watermark approach (§5.3).

### 5.2 What the tick computes, in order

For each account:

1. **Consume new interaction events.** Fetch events with `consumed_by_tick_at IS NULL` ordered by `occurred_at`. Apply the deltas (see §5.3, §5.4). Mark them `consumed_by_tick_at = now`.
2. **Compute presence windows.** Roll up consecutive presence-pings into windows; persist to `presence_windows`. (See §5.6.)
3. **Apply drift deltas.** For each completed presence window not yet consumed by drift, compute the drift contribution and apply to personality vectors. Mark `consumed_by_drift_at`.
4. **Mood transitions.** Re-evaluate each bird's mood given recent events, time-of-day, weather, and personality. Update `bird_state.mood` and `mood_set_at` if changed. (See §5.7.)
5. **Schedule calls.** For each bird with `next_call_at IS NULL` or `next_call_at <= now`, decide whether to call and, if so, schedule the next `next_call_at` and pick a `motif_seed`. (See §5.8.)
6. **Bird-to-bird interaction.** For each bird that just had something happen (a call, a mood shift), decide if any other bird responds — a chorus join, a wary spread, a curious head-tilt. (See §5.9.)
7. **Weather.** Roll a random check against the weather table. (See §5.10.)
8. **Perch decisions.** A small fraction of birds per tick reconsider perch given mood and personality. Most birds stay put.
9. **Write `aviary_state.last_tick_at`** and the `bird_state` rows.
10. **Emit a snapshot delta** to the SSE stream subscribers for this account.

A full tick for one account is bounded by ~5ms of CPU on average; the p99 latency alarm is at 5 seconds total (PRD's named threshold), which gives ample headroom.

### 5.3 Personality drift function

The drift function is a low-pass filter over presence and interaction signals.

For each presence window (duration `D` in seconds, completed and not yet consumed by drift):

```
delta_per_minute = 0.0006     # base rate, calibrated
duration_minutes = D / 60.0

# All trait deltas are non-negative. Monotonic-toward-expressive rule.
delta_boldness = 0.6 * delta_per_minute * duration_minutes
delta_social_warmth = 0.7 * delta_per_minute * duration_minutes
delta_vocal_frequency = 0.5 * delta_per_minute * duration_minutes
delta_plumage_saturation = 1.0 * delta_per_minute * duration_minutes   # most visible
delta_curiosity = 0.4 * delta_per_minute * duration_minutes

# Saturation (logistic-style soft cap so traits don't bunch at 1.0)
trait_new = trait_old + delta * (1.0 - trait_old)
```

For interaction-driven contributions (each is a small one-shot delta on top of the presence window):

- `listen_in` (per minute focused on bird X): adds to X's `social_warmth` and `vocal_frequency` at 0.5× the presence rate. Does not affect other birds.
- `offer accepted by bird X` (per acceptance): one-shot bumps `curiosity` of X by `0.002`, all-birds `boldness` by `0.0008` (offering near them at all is the boldness signal).
- `settle`: no drift contribution. Treated as a clean session-end marker.

#### 5.3.1 Calibration target (named so we can test against)

- A user with 30 minutes of presence per day for 7 days should produce a measurable shift in instruments: each trait moves by roughly 0.04–0.10 (from a 0.30 baseline that's about a 15–30% relative move). Test harness asserts this.
- The same user at 21 days should show shifts that produce *visible* changes — primarily the plumage ramp crossing a render threshold (every ~0.10 of plumage saturation moves the visual ramp by one perceptible step), and the bird's idle motion repertoire expanding (e.g. starts head-tilting at sounds it ignored before).
- A user with no presence for 14 days produces zero drift in either direction. This is the load-bearing asymmetry test: we explicitly assert that absence does not move traits down.

#### 5.3.2 No client-submitted personality state

Hard-enforced: the `POST /events` endpoint's input schema rejects any field that looks like a trait write. The simulation worker is the only writer of `personality_vectors`. A code review checklist item flags any future PR that imports the personality_vectors model in a non-simulation service.

#### 5.3.3 Bounded drift history

We store `drift_history_compact` as a fixed-size compact summary (tracking trait values at 1-week intervals, last 12 weeks; total ≤256 bytes). This is sufficient for the field notebook to write entries like "pip's plumage has filled in over the last fortnight" without us holding raw event history forever.

### 5.4 Mood transition rules

Mood is an enum over `{wary, content, curious, drowsy, alert}`. Transitions are computed at every tick using a small table.

**Per-bird mood priors based on time-of-day** (in user's local timezone, derived via `accounts.timezone`):
- Pre-dawn (5h before sunrise): drowsy with high probability.
- Morning (sunrise to ~3h after): alert / content.
- Midday: content / curious depending on personality.
- Late afternoon: content / curious.
- Dusk (1h before sunset to 1h after): drowsy / content.
- Night: drowsy / wary; nightjar species stays alert.

**Modifier inputs** (each shifts the mood-transition probability matrix):
- Recent events in the last 5 minutes:
  - offer accepted → +content
  - offer ignored (cooldown elapsed without bird approaching) → small +wary on this bird, but only if it was wary or drowsy before; we never push a content bird toward wary because of a missed offer
  - listen-in active → +content for the focused bird
  - return-greeting → +alert briefly (60s) for the greeting bird
- Weather:
  - rain → quiet, lower vocal frequency, slight +wary
  - wind → +alert
- Personality:
  - high boldness → bias against wary
  - high curiosity → bias toward curious in novel-event windows

**Mood transitions**: re-rolled each tick from the modified probability distribution. Sticky: once entered, mood persists for at least 90 seconds (enforced via `mood_set_at`) unless an event explicitly forces a transition (alarm call from another bird → wary forced). The stickiness is what makes mood feel like a *state*, not a per-tick guess.

**Mood persistence across sessions** (PRD requirement): mood is just a row in `bird_state` that the tick keeps updating. There is no client-side reset on tab open; the next snapshot the client pulls has whatever mood the server has decided.

### 5.5 Species assignment at adoption

At account creation:

- Pool: `{wren, warbler, finch, thrush, sparrow, nightjar}`. v1 ships six.
- Two starter species are picked: not the same species twice; weighted by a fixed distribution (more common species: wren, sparrow, finch; rarer: nightjar).
- Default name suggestions are species-appropriate (wren → "wren" or "pip"; sparrow → "sparrow" or "pip"; etc.).
- The user does not see the catalog. The two birds arrived; that's the framing.

### 5.6 Presence accounting

Per the PRD: presence-event recorded only when, **simultaneously**:
1. `document.visibilityState === 'visible'`
2. document has window focus (`document.hasFocus()`)
3. at least one `pointermove` or `keypress` in the last `T` minutes (calibration target: T = 4 minutes; we lean longer because watching birds without moving is the real product)

Client: a `PresenceTracker` module checks all three at a 5-second cadence; if all true, increments an in-memory window. On state change (any of the three flips false), it closes the current window and queues a `presence_ping` event with `window_started_at`, `window_ended_at`, `duration_seconds`. On flush, events go to `POST /events`.

Server: `presence_windows` rows are computed by rolling up `presence_ping` events. (We chose to send window-events rather than per-second pings to reduce request volume; the resolution is preserved in the event payload.)

Verification: presence is the dominant drift input, so the `PresenceTracker` is covered by extensive tests:
- Tab hidden but window focused → no presence.
- Window unfocused but tab visible → no presence.
- No mouse activity for 5+ minutes → no presence (after the calibration window elapses).
- All three true → presence accumulates at wall-clock rate.
- Long-suspended laptop → on resume, the `pointermove` watchdog has been reset; presence resumes from the next ping.

A red-team test runs the harness with a "tab open in background, laptop unattended for 8 hours" trace and asserts zero drift.

### 5.7 Call grammar runtime (server side)

Each species has a motif library — a small set of pitch/duration/articulation patterns that combine to form calls. Both server and client know the library; the server picks *which* motifs combine and *when* a call happens, and the client renders them.

Server-side picks: at each tick, for each bird:
- Probability of call in next 60s: `base_rate(species) × vocal_frequency_trait × mood_modifier × weather_modifier × time_of_day_modifier`
- If the roll succeeds, schedule `next_call_at` at a randomized point in the next 60s; pick a motif sequence (1–3 motifs, weighted by mood); compute `motif_seed` so the client can deterministically reproduce the same audio from the same seed.

Server-side outcome is `{scheduled_utc, motif_seed, duration_ms, intensity}`. The client uses `motif_seed` to drive the WebAudio synthesizer; same seed produces the same audio across devices and across sessions for a given snapshot. (Since the same call should sound the same to multiple devices viewing the same aviary at the same tick.)

### 5.8 Bird-to-bird interaction

After a bird's call or mood-shift this tick, for each other bird:
- Roll for chorus-join: if my `vocal_frequency × social_warmth` ≥ threshold, I may schedule a call within the next ~5s.
- Roll for wary-spread: if the originating bird went wary (e.g. alarm call), nearby birds (perched within 1 zone) re-roll mood with +wary bias.
- Roll for curious-attention: if the originating bird is calling melodically, high-curiosity birds may switch to `tilting`.

This produces emergent chorus and emergent quiet — the aviary feels like a small social system rather than independent NPCs, which is the PRD's stated goal.

### 5.9 Weather simulation

- Per-tick roll: `clear` → `clear` with high probability; small chance to enter `rain` (lasting 10–25 minutes) or `wind` (lasting 20–40 minutes).
- Frequency target: a few times per week (PRD wording); we calibrate to ~3 events per week.
- Effects: see §5.4 mood modifiers.
- Visitors see the same weather (server-canonical).

### 5.10 Third-bird offer cadence

Per PRD: new species offers appear based on aviary age, not visit count or score.

- First eligibility: ~60 days after account creation.
- Cadence: a fresh offer every 30–90 days (jittered) until cap of 7.
- The offer surface itself: a small naturalist-prose surface ("a small thrush has been spending time in the aviary lately. would you like to invite it to stay?"), with confirm or dismiss. Dismissal does not cost anything; the next offer comes around on schedule.
- The cadence is in `aviary_state.next_third_bird_offer_eligible_at`; reset on accept or dismiss.

The PRD makes a clear point that cadence is age-based, not effort-based. We resist any temptation to add "you visited every day so a bird arrives faster." The cadence is wall-clock.

### 5.11 What the simulation is forbidden from doing

- Writing personality vectors based on a client-supplied value.
- Producing negative drift on any trait. (Asserted in the drift-update code.)
- Resetting personality vectors. There is no `regenerate bird` code path.
- Replacing one bird with another. Renaming and species are decoupled; bird UUID is invariant.
- Telling the client "the user has visited 7 days in a row." There is no aggregator that produces this signal.
- Performing any computation that aggregates across accounts (e.g. "average drift"). This is a hard architectural separation: simulation database is not connected to analytics, by network policy and by service-account permission.

---

## 6. Sync model

### 6.1 The architectural property

There is no client-side sync engine. The sync property comes for free from "server is the only canonical writer."

- Both the user's laptop and phone open SSE streams to the same `/aviary/stream`.
- Both receive the same `snapshot:full` on connect and the same `snapshot:delta` on every tick.
- Both render the same scene from the same data.
- Both write events to the same `interaction_events` log via `POST /events`.
- The simulation tick consumes events in `occurred_at` order regardless of which device sent them.

### 6.2 No last-write-wins for personality

Restated as a section header because the PRD is explicit. Personality is server-written via additive deltas computed from the event log. Clients never submit personality values. There is no code path on the server that takes a personality state from a client and uses it.

### 6.3 Event log conflict resolution

Two devices may emit `listen_in_start` for the same bird at slightly different times. The tick processes them in `occurred_at` order; the one with the later `occurred_at` is the one "in effect" when the tick runs. The earlier one contributes a small drift signal for its observed-duration window before the second one starts.

Two devices that try to settle simultaneously: settle is idempotent at the `aviary_state` level — the lighting evening-shift is a derived UI state, not a stored boolean — so this is not a conflict.

### 6.4 What sync explicitly is not

- Not a CRDT. We considered this and rejected it: there is no client-side state that needs to merge.
- Not eventual consistency requiring reconciliation. Snapshots are consistent at every tick.
- Not last-write-wins on any state. The phrase "last-write-wins" should not appear in any code in this product.

### 6.5 Multi-device simultaneous interaction

User on laptop and phone listening in to different birds simultaneously: both events are recorded. The simulation tick attributes the listen-in time to each focused bird independently. Drift contributions accumulate to both birds. From the user's perspective, this is a slightly unusual pattern (two devices, two listen-ins), but the product is honest about it.

We considered whether to "lock" the aviary to one device. We chose not to: the user paying attention from two devices is still attention, and the product's central frame is "presence is real interaction." Locking would introduce a "switch active device" UI, which is announcement-style.

### 6.6 Visitor sessions and the simulation

Visitor's SSE stream is a fan-out of the host's `snapshot:delta`s. Visitor has no event-write capability (read-only at the API layer). Simulation does not see the visitor; nothing the visitor does affects drift, mood, or anything else.

### 6.7 Snapshot pull authentication

- Host pulls require a session cookie that authenticates as the account's owner.
- Visitor pulls require an `invite_token` query parameter; the server validates against `visits.invite_token_hash`, checks `revoked_at` is null and `expires_at` is in the future. On invalidate, the SSE stream returns a `visit:revoked` event; the client renders a matter-of-fact surface.

---

## 7. Frontend rendering pipeline

### 7.1 Stack

- **Framework**: Solid.js. Chosen over React for two reasons: smaller bundle (1–2KB runtime vs ~40KB), and fine-grained reactivity (no virtual DOM diff per tick, which matters at 60fps). Solid's reactivity model is also closer to the way our state actually flows (signal in, signal out).
- **Rendering**: a hybrid SVG + Canvas approach. The aviary scene background, perches, and most ambient elements are SVG (for sharpness and accessibility); birds are rendered as Canvas (for per-frame procedural micro-motion that would thrash SVG). Birds' static silhouettes are SVGs preloaded into off-screen `ImageBitmap`s once and drawn to Canvas.
- **State**: server-canonical, delivered via SSE; local UI state in Solid signals (e.g. "is the user currently listening in," "is the top bar visible").
- **Animation**: a small custom animation runtime (no GreenSock, no Framer; bundle budget). Mood-keyed easing curves. Deterministic seeded jitter so that two devices viewing the same bird draw the same micro-motion frame-for-frame within a render tick.
- **Build**: Vite + Rollup. Aggressive code-splitting: account-settings, accessibility-settings, visit-invitation flow are separate chunks loaded on navigate.

### 7.2 Boot sequence — time to first bird <500ms

The 500ms budget on a mid-tier mobile + 4G is the affective-perf threshold. The boot sequence:

1. **0ms**: HTML arrives from CDN. Includes inlined JSON snapshot in `<script type="application/json" id="initial-snapshot">`. Includes critical CSS inlined in `<style>` (the day/night palette, scene background, top bar layout). Includes a small bootstrap script (~3KB) that runs synchronously.
2. **0–100ms**: Bootstrap script reads the inlined snapshot, places bird sprites at their initial perches via positioned `<canvas>` elements inside the SVG scene container, and draws the first frame. **At this point the user sees the first bird.** We are aiming for this to land at ~150ms on a mid-tier mobile + 4G; it must land before 500ms.
3. **100–800ms**: The main JS bundle (≤2MB gzipped) finishes loading and hydrates. The PresenceTracker, the SSE connection, the audio context (gated on user gesture; we don't autoplay), and the animation runtime come online.
4. **800ms+**: Birds' next calls are scheduled per the snapshot's `next_call` fields. WebAudio activates on the first user interaction (click anywhere; required by browser autoplay policy). Until then, captions display (if `captions: 'auto'` or `'on'`) for any call the snapshot says is happening.

The first bird does not wait for JS to fully hydrate. The bootstrap can draw a single static-pose bird from the snapshot data alone. From the user's perspective, the aviary appears with birds in place.

### 7.3 Snapshot reconciliation

The client maintains a `Map<bird_id, BirdRenderState>`. Each `snapshot:delta` updates the matching entries. Position changes (perch transitions) trigger a smooth animated transition (cross-fade in reduced-motion mode; flight path in normal mode). Mood changes update the animation generator's parameters; the next idle-motion cycle uses the new parameters.

**No extrapolation rule**: if the client doesn't get a new snapshot for 90 seconds (e.g. SSE connection died), the client does not invent state. Birds continue their last-known animation cycle until reconnect; reconnect triggers a `snapshot:full` and the client snaps to the canonical state with a brief cross-fade.

### 7.4 Idle micro-motion

Implemented as deterministic generators per `(bird_id, mood, action)`:

- Preening: a short cycle (~3s) of head-to-wing motion with personality-keyed timing jitter.
- Scanning: head turns left and right with mood-keyed period.
- Tilting: head tilts toward sound source (driven by call events from other birds).
- Fluffed: minimal motion, slow body sway.
- Idle (default): tiny weight-shift on perch, occasional blink.

Each generator outputs interpolation parameters for limbs and head; the renderer composes them into a final draw. No looped frame animation; everything is computed.

### 7.5 Day/night and weather

- Sky background is a CSS gradient driven by current local time. Computed via CSS variables; updated every 5 minutes (scene transitions are slow).
- Weather overlays: rain is drawn as a thin canvas overlay with low-density droplet particles; wind is a leaves-drift particle multiplier and a slight foreground branch sway.
- All ambient effects are bounded particle counts; we cap to ensure 60fps headroom.

### 7.6 Reduced-motion mode

Triggered by `prefers-reduced-motion: reduce` or by `accounts.reduced_motion_pref === 'on'`. In this mode:

- The bird renderer switches to a different generator: cross-fades between still poses every 4–8 seconds instead of continuous micro-motion.
- Flight transitions become opacity cross-fades (bird disappears from old perch, appears at new perch over 1.5s).
- Ambient leaf drift is removed.
- Day/night palette transitions remain but are slowed.
- Calls play normally (or caption per audio settings).
- The reduced-motion mode is its own designed surface, not a stripped fallback. The cross-fade aesthetic is intentional — calmer, slower, with its own quiet charm. Visual designer to specify the cross-fade pose set per species.

### 7.7 Top-bar fade and chrome

- Top bar: 4 icon slots — notebook, offer, settle, settings.
- Position: top-left for notebook + offer + settle; top-right for settings (account, accessibility).
- Styling: small icons, monochrome-ish, low-contrast against scene. WCAG AA on hover/focus state.
- Fade: opacity goes from 1.0 → 0.15 over 600ms after 4 seconds of cursor stillness on the aviary surface; returns to 1.0 immediately on `pointermove` or `keydown`.
- Keyboard focus pulls the top bar to full opacity and remains.

### 7.8 No render path that contains UI chrome inside the aviary

A linter rule: components rendered as children of `<AviaryScene>` cannot import from `<TopBar>`, `<Toast>`, `<Modal>`, or any chrome-component module. Enforced at build time. Any future PR that renders a tooltip in-scene fails CI.

---

## 8. Audio pipeline

### 8.1 Procedural call synthesis

- WebAudio graph per bird: a small subtractive/FM synthesis chain that takes motif parameters (pitch, duration, articulation, breath, intensity) and produces a call.
- Motif library per species: ~5–10 base motifs each. A motif is a sequence of (pitch_pattern, duration_pattern, articulation) tuples, parameterized.
- Variation: the call grammar combines 1–3 motifs per call with personality-keyed timing/pitch perturbation. A bird with `vocal_frequency = 0.4` calls with consistent timing; a bird with `vocal_frequency = 0.8` calls with more variation in motif sequencing.
- Same-call determinism across devices: server provides `motif_seed` in the snapshot; client uses it to seed the variation RNG. Same seed, same output, on every device for a given snapshot.

### 8.2 Chorus mixing

- Each bird has its own AudioBufferSourceNode chain feeding into a per-bird GainNode, then into a master mix bus.
- Listen-in mix: the focused bird's gain ramps to 1.0 over 600ms; non-focused birds' gains ramp to 0.25 over the same window. Disengage reverses the ramp.
- Ramps are linear-then-eased (ease-out-cubic at the tail) to avoid the "switching channels" feel.
- Master output goes through a soft-knee compressor to prevent two simultaneous loud calls from clipping.
- Procedural means real-time mixing: two birds calling at once produce an actual chorus, not stacked loops. Phase relationships are organic.

### 8.3 No recorded audio fallback

If WebAudio is unavailable (older browser, Safari with audio context permission denied, hardware error):
- The aviary plays in silence.
- Captions auto-enable (regardless of `captions_pref`, with a small UI affordance to disable if the user prefers complete silence).
- A matter-of-fact one-time notice appears in settings: "audio is not available on this device. captions are on by default."

We do not ship recorded audio. The bundle would not fit the procedural variation we need; the failure mode of "canned audio that everyone immediately recognizes as canned" is worse than silence.

### 8.4 Audio context activation

Browser autoplay policies require a user gesture before WebAudio can produce sound. The first user interaction (click anywhere on the aviary, top-bar icon click, even a key press) activates the audio context. Before activation, captions cover any scheduled calls; this is a normal state, not an error state.

### 8.5 Captioning runtime

Captions are generated from the same call-grammar parameters that drive the audio. The caption generator maps motif sequence + intensity to short naturalist prose:
- 1 short motif at low intensity → "a soft single note from the back perch"
- 3 motifs at high intensity → "a long trill, a pause, two short notes"
- Chorus join → "another joins in"

Captions appear as small text near the bird (positioned just above the calling bird), fading in over 200ms, holding for the call duration + 1s, fading out over 400ms. WCAG AA contrast against any aviary background state.

The caption library is parameterized prose, not a fixed string per call. Every caption is generated per call, matching what the audio actually played.

### 8.6 Listen-in implementation details

- Engage via mouse click, tap, or keyboard Enter on focused bird.
- Disengage via second click on same bird, click anywhere not on a bird, focus shift to a different bird, or Escape key.
- Mix transition: 600ms in, 600ms out.
- Visual signal: focused bird has a subtle outline that intensifies very slightly. No "selected" badge, no "listen-in active" toast.
- Event log: emits `listen_in_start` on engage, `listen_in_end` on disengage (or session end / tab close).

### 8.7 Audio bundle budget

The motif library, the synthesis chain, and the caption generator together must fit in the bundle. Estimated cost: ~150–200KB gzipped (includes all 6 species' motif libraries). Within the 2MB envelope.

---

## 9. Accessibility surfaces

This section operationalizes the PRD's "first-class, not a checklist" stance. Every accessibility surface here is a designed surface, not a stripped fallback.

### 9.1 Screen-reader narration

- A live region (`aria-live="polite"` `aria-atomic="false"`) attached to the aviary scene container, hidden visually but read by AT.
- A `NarrationGenerator` module reads the same snapshot the visual renderer reads, and emits naturalist-prose updates at a 30–60s idle cadence.
- On user-initiated events (return-greeting, offer reaction, settle), the generator emits a prioritized update within 1–2 seconds; the live region's politeness is upgraded to `assertive` only for return-greeting (so it lands promptly), not for ongoing observations.
- Prose voice: same as the field notebook. Lowercase. Present-tense. Specific.

Examples (generated, not pre-written):
- Idle morning: "a small grey bird is perched on the front rail, calling softly. another sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
- Return-greeting: "pip steps forward and calls a soft two-note rise."
- Offer accepted: "the seed is offered; pip approaches and pecks once."

We avoid:
- "Pip is at perch 2." (announcement)
- "Pip mood: content." (state-list)
- "Welcome back!" (announcement at all)

### 9.2 Reduced-motion mode

(Cross-referenced with §7.6.) The reduced-motion mode is a different rendering of the same aviary. Calls still play, drift still happens, the field notebook still notices things. The aesthetic is calmer, slower, cross-fade-based.

A user who has set `prefers-reduced-motion` for vestibular reasons gets a Pocket Aviary that is calmer; not one that feels broken.

### 9.3 Captions

Captions for procedural calls (covered in §8.5). Default: `auto` — captions appear if WebAudio is unavailable or the user has audio off; can be forced on via settings.

Caption position is alongside the calling bird, positioned to avoid overlapping the bird itself. Visual designer specifies treatment.

### 9.4 Keyboard navigation

- Tab: moves focus through top bar items (notebook, offer, settle, settings) in left-to-right order, then into the aviary.
- In aviary: Tab focuses first bird; arrow keys (Left/Right) move focus between birds in perch order.
- Enter on a focused bird: triggers listen-in.
- Escape: disengages listen-in if active; otherwise moves focus back to top bar.
- Top-bar shortcuts: `o` for offer (opens the offer sheet), `s` for settle (with the 5-second undo affordance accessible via Escape), `n` for notebook.
- Focus indicator: high-contrast outline around focused bird (designer-specified).

### 9.5 Contrast

All user copy passes WCAG AA. Specifically:
- Top bar labels: 4.5:1 minimum against any aviary state.
- Captions: 4.5:1 minimum, with a subtle text shadow / scrim if the bird's local background is too varied.
- Settings, error surfaces, account UI: all meet AA; design system enforces.
- Aviary scene itself: no body text overlaid on the scene (the scene is just birds and place); contrast applies primarily to chrome.

### 9.6 Settings surface

Accessibility settings:
- Audio: on / off
- Captions: auto / on / off
- Reduced motion: auto / on / off
- (Plus account-level: `notify_on_visit` toggle, session list, account export, account delete.)

Settings UI uses matter-of-fact tone (PRD's named exception). System-clarity over naturalist warmth.

### 9.7 Test plan

- Automated: axe-core in CI, asserting no WCAG AA violations on top bar, settings, account flows, error surfaces, notebook view.
- Manual: VoiceOver + macOS, NVDA + Windows, TalkBack + Android, on every release. Real ATs, not just automated tests, because the *quality* of the narration matters.
- Reduced-motion test: a user with `prefers-reduced-motion: reduce` should see the cross-fade renderer, hear the same calls, get the same notebook entries, and feel the same drift over weeks. Nothing about their experience should be "lite."

---

## 10. Performance budgets and observability

### 10.1 Budgets

| Budget | Target | Test |
|---|---|---|
| Initial JS bundle (gzipped, first paint) | <2MB | CI assertion on bundle size; PR fails if exceeded. |
| Time to first bird visible | <500ms on mid-tier mobile + 4G | Synthetic Lighthouse run in CI; perf monitoring fleet asserts on prod. |
| Idle motion FPS | 60fps on 5-year-old mid-range laptop | Manual benchmark per release; client RUM reports frame-time histograms (anonymized). |
| Memory growth over 30 minutes | 0 net growth | CI test runs the client headlessly for 30min and asserts heap snapshot growth is below 5MB. |
| Simulation tick latency | p50 < 100ms; p99 < 5s | Prometheus alarm at p99 > 5s. |
| Snapshot delivery (SSE delta) | < 100ms from tick completion to client receipt | Synthetic monitoring with timestamped pings. |

### 10.2 Observability — what we measure

Server-side:
- Request counts, latencies, error rates per endpoint.
- Tick duration per account (histogram).
- Tick worker queue depth.
- SSE connection count, age distribution.
- DB query latencies (slow query log).

Client-side (RUM, aggregate-only):
- Page load timings (TTFB, FCP, time-to-first-bird).
- Render-frame timings (frame time histogram, dropped-frame count per session).
- Audio context errors, audio buffer underruns.
- SSE reconnect rate.

### 10.3 What we deliberately do not measure

- Per-account session duration. (Aggregate histograms ok; per-account, no.)
- Per-bird mood histograms. (Across-account aggregation forbidden by §11.)
- Per-account interaction counts. (Forbidden.)
- "Engagement metrics" of any kind. (Forbidden by intent and architecture.)

### 10.4 Telemetry boundary

The simulation database is on a separate VPC subnet from the analytics warehouse. No service account in the analytics path has read access to the simulation database. The metric-emission library on the server has a column denylist: any metric with a label that includes `bird_id` or `account_uuid` is dropped at the client side and a build-time linter flags the offending code. This is a hard architectural separation, not a policy.

### 10.5 Synthetic monitoring fleet

- 4 synthetic clients running on schedule from US-East, US-West, EU-West, AP-Southeast.
- Each runs the aviary for 5 minutes per check, asserting: TTFB, time-to-first-bird, first SSE event arrival, audio context activation, frame-rate sample.
- Failures page on-call with structured diagnostics.

### 10.6 Error budgets

- Client-side render errors: <0.1% of sessions show a render error.
- SSE disconnect rate: <2% per minute (excluding tab-hidden cases, which are expected to disconnect).
- Tick-latency p99: <5s (PRD-specified alarm).
- Audio context activation rate: >95% on supported browsers (the rest fall back to captions).

---

## 11. Privacy architecture

(Cross-references §10.4, §3.4, and the `accounts_sync.md` privacy rules.)

### 11.1 What we keep, where, and for how long

- **Email**: encrypted (KMS) on `accounts.email_encrypted`, plus an HMAC for lookup at `accounts.email_hash`. The HMAC key is per-environment; it is never used outside the auth lookup path. Email appears nowhere else. Emails in transit (notification, magic link, account-export delivery) are sent via the email provider and not persisted in app logs.
- **Per-bird interaction events**: persisted in `interaction_events` for the lifetime of the account. Used only for the simulation tick. Deleted on hard account delete.
- **Notebook entries**: persisted for the lifetime of the account. Deleted on hard delete.
- **Aggregate operational telemetry**: counts, latencies, error rates, anonymized session-duration histograms. Retention 90 days. No per-account dimension.
- **Logs**: server logs use `account_uuid` (the synthetic ID) only. No emails, no IPs beyond the standard request-log retention (30 days), no per-bird interaction details. PR linter flags any log statement that includes bird state.

### 11.2 What we do not keep

- We do not aggregate per-bird interaction state into any cross-account dataset.
- We do not train any model on per-bird interaction data. (No models in v1, but the rule applies forward.)
- We do not share per-bird interaction state with any third party.
- We do not compute "popular interaction patterns" or "average aviary mood" across accounts.

### 11.3 Account deletion

- Soft delete (immediate): `accounts.deleted_at` set; the user sees a "this account is scheduled for deletion in 30 days" banner if they sign in within the window, with a one-click restore.
- Hard delete (after 30 days): a daily worker hard-deletes accounts with `deleted_at + 30 days < now`. Cascades drop birds, personality vectors, bird state, interaction events, notebook entries, sessions, visits. Aggregate telemetry is anonymized and not affected.

### 11.4 Audit

Quarterly: a privacy review confirms the simulation/analytics separation hasn't drifted, the linter rules are still in place, no new code path has been introduced that violates the data-pipeline boundary.

---

## 12. Internationalization and localization

V1 is English-only on user-facing surfaces. The PRD does not call out i18n; we are not scoping it for v1.

However, two i18n-adjacent concerns must be built right from day one:
- **Local-time anchoring**: the aviary uses the user's local time, not server time. The client sends its IANA timezone on session start; the server caches it on `accounts.timezone`. Day/night palette and mood time-of-day priors derive from local time.
- **Timezone changes**: if the user signs in from a device whose timezone differs from the cached one, we update the cache. We don't try to be clever about "is this travel or a new device"; we just use the most recent observed timezone.

We are not localizing prose in v1. If we localize later, the field notebook, narration, and offer prompts would all need translator-passable templates; the procedural call-caption system would also need motif-name translations. We tag these surfaces in the codebase so a future i18n pass has clean boundaries.

---

## 13. Rollout

### 13.1 Phasing

**Phase 0 — internal prototype (weeks 1–6).** Build the simulation engine end-to-end with one species, two birds, on a single dev environment. Verify the tick correctly drives drift over a simulated week of presence. No frontend yet — drive it from a Python REPL that emits events and checks state.

**Phase 1 — closed alpha (weeks 6–14).** Bring the frontend online. Implement two species, the listen-in interaction, the offer interaction, the settle gesture, the field notebook, basic SSE delivery. ~20 internal users on isolated environment. Goal: validate the affective design — does the aviary feel alive? — not to validate scale.

**Phase 2 — invite-only beta (weeks 14–24).** All six species. Full interaction set. Multi-device sync. Reduced-motion mode. Screen-reader narration. Visit feature. ~500 invited users. Feature flags allow enabling features per cohort. Goal: validate drift calibration over 4–6 weeks of real use, validate sync correctness across devices, validate accessibility surfaces with real AT users.

**Phase 3 — public launch (week 24).** Open sign-up with invite-link sharing. Calibration values frozen. Bundle budget held to <2MB.

### 13.2 Birds-per-aviary ramp

We do not start the cap at 7. We start at 2 (fixed) for the closed alpha, raise to 3 in beta, and raise the cap to 7 only after we've tested chorus recognizability with real users. The cap-raise is a feature flag, not a deploy.

### 13.3 What we instrument from day one

- Drift correctness: does presence-time produce the calibrated trait shifts? Synthetic tests run a simulated user for 30 days and assert on trait ranges. Production telemetry tracks p50/p95 trait values across the user base and alarms if the distribution drifts (population-level only; no per-account exposure).
- Sync correctness: a multi-device test harness runs two clients against one account and asserts they see identical snapshots within the SSE delivery window.
- Audio quality: a daily synthetic test renders a 60-second chorus from a fixed seed and asserts the output's spectrum matches a reference (regression catch for changes that subtly alter audio).
- Accessibility regression: axe-core on every PR, plus monthly manual VoiceOver/NVDA/TalkBack walkthroughs.

### 13.4 Calibration windows

The drift function and the presence-window threshold are calibrated during beta. Concretely, we expect to:
- Start with the values in §5.3 and §5.6.
- Observe real users' traits over 2–4 weeks.
- Adjust the rate constants so that the "measurable in instruments at week 1, visible to user at week 3" target is hit.
- Lock the values at public launch.

Post-launch changes to the drift function are forward-only — we don't retroactively reshape anyone's traits — and require an explicit rollout doc.

### 13.5 Feature flags

- `enable_visits`: gates the visit-invitation flow.
- `enable_third_bird_offer`: gates the new-species offer cadence.
- `cap_birds`: integer; soft cap on birds-per-aviary, defaults to 2 in alpha, 3 in beta, 7 at launch.
- `enable_screen_reader_narration`: gates the narration generator (we want this on always but the flag exists so we can disable in case of regression).

Feature flags are server-controlled; the client receives the relevant subset on session start. No client-side flag overrides.

### 13.6 Migrations

- Schema migrations are forward-only in production. Down-migrations exist in dev/test for iteration, never run in prod.
- Personality-vector migrations: if we ever change the trait set (e.g. add a 6th trait), the migration must preserve all existing trait values, initialize the new trait at a low seed value (so it doesn't show up retroactively), and document the change in a forward-only ledger. We do not plan such a migration in v1; the rule is stated so future-us doesn't accidentally invalidate drift history.

### 13.7 Rollback

The simulation tick is the most fragile component. A rollback plan: keep the previous simulation worker version on standby; if the new version produces tick-latency p99 alarms or drift-correctness anomalies, swap workers behind a feature flag. The Postgres schema is the source of truth; bird-state writes are idempotent on `(bird_uuid, last_tick_at)` so a worker replacement does not corrupt state.

### 13.8 Day-1 launch checklist

- All performance budgets pass on synthetic monitoring from all four geographies.
- Accessibility audit complete for VoiceOver, NVDA, TalkBack on the launch device matrix.
- Drift calibration locked; calibration document signed off.
- Privacy boundary audit complete; simulation/analytics separation verified by network policy review.
- Email magic-link delivery tested through the production email provider with real DKIM/SPF.
- Account export end-to-end tested.
- Account hard-delete tested (in staging with synthetic accounts at 30+ days old).
- Visit revocation latency verified.
- WebAudio fallback path tested on a Safari version that denies audio context permission.

---

## 14. Risks

### 14.1 Drift calibration miscalibration

**Risk**: The drift function is too fast (Tamagotchi-feel) or too slow (screensaver-feel). The engine will silently produce the wrong product if the rate constants are off, and the user-felt failure ("it doesn't feel like anything is happening" or "my bird changed too fast") will be hard to diagnose because nothing throws an error.

**Mitigation**: The PRD names the calibration target ("measurable at week 1, visible at week 3"). We bake those targets into automated tests that simulate 30 days of presence and assert on trait deltas. We tune in beta with real users before locking. We monitor population-level trait distributions in production for drift in the drift function (the meta-drift).

**What we won't do**: ship a "drift speed" setting for users. The drift speed is a designed property; making it user-tunable would convert it into a number the user manages, which is exactly the failure mode this product is shaped to refuse.

### 14.2 Sync correctness — silent personality data loss

**Risk**: A bug in the simulation tick's event-consumption logic causes some interaction events to be skipped or double-counted, silently. The user's drift would be wrong but no error would fire.

**Mitigation**: 
- The `consumed_by_tick_at` watermark is the only thing that determines whether an event counts. It's set in the same transaction that writes the personality vector update.
- A daily reconciliation job spot-checks: for a sample of accounts, replay the event log from scratch and assert the trait values match the stored values (within rounding tolerance). Discrepancies page immediately.
- Property-based tests on the tick (Hypothesis-style): random event sequences, assert idempotency and order-independence-where-applicable.
- The "no last-write-wins on personality" rule means client conflicts can't corrupt state at all; this risk is mostly internal.

### 14.3 Audio uncanniness

**Risk**: The procedural calls sound like generated audio, not like birds. The "feels canned" failure the PRD warns about.

**Mitigation**:
- Hire or contract a sound designer who has done procedural audio for nature/ambient products. Don't have a generalist engineer design the synthesis chain.
- Per-call variation tested in user research during alpha and beta: do listeners distinguish two birds by ear after 2 weeks? If not, we recalibrate motif distinctiveness or reduce the species count.
- Reference comparison: build a reference mix of real-bird recordings (for internal calibration only, never shipped) and listen to procedural output A/B. Ear-test, not just spectral analysis.
- Chorus-mix realism tested with a phase-correlation analysis: stacked-loop artifact would show up as comb filtering; we assert the procedural output does not.

### 14.4 Accessibility regressions

**Risk**: A frontend change breaks the screen-reader experience, or the reduced-motion mode silently degrades. Accessibility regressions are easy to ship and hard to detect because most engineers aren't using AT routinely.

**Mitigation**:
- axe-core on every PR; CI fails on AA violations.
- Manual VoiceOver/NVDA/TalkBack tests on every release; logged in a release checklist.
- Reduced-motion mode has its own test harness that captures screenshots over a 30-second simulated session; visual diff catches regressions in the cross-fade rendering.
- Hire/contract an AT user as part of beta; their feedback gates launch.

### 14.5 The "harmless" gamification feature

**Risk**: A well-meaning contributor adds a "you've been here 5 days in a row" surface, a milestone celebration on first listen-in, a "Pip greeted you 3 days in a row" notebook entry. Each one looks innocuous; the cumulative effect is a different product.

**Mitigation**:
- The non_goals.md file is canonical. Every PR description is required to confirm it doesn't add a streak/score/level/celebration surface; a PR template enforces this.
- The schema has no `streak_days`, `total_visits`, `xp`, or any column that would seed such a feature. Adding such a feature requires a migration, which requires a rationale in the migration's forward-only ledger, which is reviewed.
- A linter rule forbids the strings `streak`, `achievement`, `unlocked`, `level up`, `you've been here`, `welcome back`, `your friend visited` (case-insensitive) from appearing in any rendered string. The rule is hard; bypassing it requires explicit comment with PR-author justification, which would then be reviewed.

### 14.6 Voice drift in notebook entries

**Risk**: The notebook entry generator produces entries that drift toward stock event-log style ("session started," "Pip's mood changed to content"), eroding the product's voice.

**Mitigation**:
- Notebook entries are template-generated with a constrained library; the templates are reviewed by a copywriter at launch.
- A test corpus of 100 generated entries is reviewed each release; the reviewer checks for voice drift and any forbidden patterns (announcement framing, "you" in observations, gamification language).
- Sparsity is enforced: target 1 entry per 3–5 days for a regularly-visited aviary; the generator's rate-limit is a hard cap per account per week.

### 14.7 Performance regressions over time

**Risk**: Bundle size creeps over 2MB; time-to-first-bird drifts over 500ms. These are slow regressions that don't fire alarms until they're well past the budget.

**Mitigation**:
- Bundle-size assertion in CI; PR fails if over budget.
- Synthetic monitoring fleet asserts time-to-first-bird every 5 minutes in production.
- A monthly perf review checks frame-time histograms from RUM and identifies any slow-burn regressions.

### 14.8 PII leakage via email-as-identifier

**Risk**: A future engineer adds a service that uses email as an identifier in logs/Kafka/shards. PII spreads everywhere it shouldn't.

**Mitigation**:
- The synthetic UUID rule is in the architecture doc and the code review checklist.
- The accounts table is the only table with `email_encrypted`. A linter rule flags any new table with a column matching `email`, `e_mail`, or `mail_address`.
- A quarterly grep across logs verifies no email patterns appear.

### 14.9 Visitor-traffic privacy edge cases

**Risk**: A visitor's IP/UA/session is logged in a way that exposes "X visited the host's aviary" to the wrong party.

**Mitigation**:
- The visit log shows the visitor's email (which is the host's input) and timing only; never the visitor's IP, device fingerprint, or any other detail.
- Visitor session tokens are scoped per-invitation; no cross-invitation correlation.
- Server logs scrub visitor IPs after 30 days.

### 14.10 Server-side simulation single point of failure

**Risk**: The tick worker fleet goes down. The aviary doesn't tick. Users opening the tab get stale state.

**Mitigation**:
- Multiple tick workers with account-partition fan-out; one worker's failure affects only its partition.
- Backfill on restart: the next tick after a worker restart processes accumulated events and produces a delayed-but-correct state.
- Read-only degradation: if all tick workers are down, clients still get the last-known snapshot from Postgres; the aviary appears slightly frozen but doesn't error. A matter-of-fact banner appears if the staleness exceeds 10 minutes.

### 14.11 Ambiguity calls (judgment notes)

A handful of decisions in this plan resolve PRD ambiguities; we're explicit about them so reviewers know what's decided vs what's interpreted.

- **Tick cadence**: PRD says "~once per minute." Picked 60 seconds with calibration range 30–90s.
- **Presence pointer/key window `T`**: PRD says "a few minutes." Picked 4 minutes; tuned in beta.
- **Drift rates per trait**: PRD names the calibration target but not the rate constants. Picked the §5.3 values; tuned in beta.
- **Plumage saturation render exposure**: PRD says "user never sees personality vector values." We expose a normalized `plumage_saturation_normalized` scalar in the render hint because the visual ramp can't be driven without it. The scalar is opaque (not labeled with the trait name) and is the only one of the five traits that has any client surface. This is a hairline call and we own it.
- **Magic-link rate limit**: PRD says "per-email at a reasonable threshold." Picked 3/hour, 10/day.
- **Snapshot delivery**: PRD says client pulls snapshots and interpolates. We added SSE for push delivery; pull remains as fallback. This is a strict superset of the PRD requirement.
- **Bird species pool**: PRD says "about six species." Picked exactly 6: wren, warbler, finch, thrush, sparrow, nightjar. The nightjar is required to satisfy "one species remains active into the night."
- **Third-bird offer cadence**: PRD says "an aviary a few months old offers a third bird." Picked first eligibility at 60 days; subsequent offers every 30–90 days.

---

## 15. Open questions for design / product (not blockers; flag for sign-off)

These are not implementation blockers — the plan above resolves each one defensibly — but they are worth explicit sign-off from product/design before launch.

1. The exact visual treatment of the `plumage_saturation_normalized` scalar's effect on the bird sprite. The plan exposes a normalized scalar; the visual designer specifies how it maps to plumage rendering.
2. The motif libraries for each species. Sound designer to deliver; voice-of-product approves.
3. The notebook-entry template library. Copywriter to deliver; voice-of-product approves.
4. The reduced-motion pose set for each species. Visual designer to deliver.
5. The default name suggestions per species. Copywriter to deliver.
6. The exact magic-link email copy. Matter-of-fact tone; copywriter approves.
7. The matter-of-fact unsupported-browser surface. Copywriter to deliver.

---

## 16. Plan summary (one paragraph)

Pocket Aviary is a server-canonical, multi-device, browser-rendered aviary with a small set of birds that drift in personality over weeks of presence. The implementation is shaped by five load-bearing rules: server-only personality writes (no client mutation, no last-write-wins), additive monotonic drift (no negative trait movement), procedural audio synthesis (no recorded fallback), accessibility-as-designed-surface (not stripped fallback), and architectural separation of per-bird state from analytics (data pipeline boundary, not policy). The product surface refuses every standard engagement feature (streaks, scores, levels, "welcome back"); the data model contains no columns that would seed those features, and the linter and CI rules forbid them being added later. Calibration of drift rates and presence thresholds happens in invite-only beta with real users; the bundle, time-to-first-bird, and tick-latency budgets are CI-asserted from day one. The plan ships in four phases over ~24 weeks, with feature flags gating bird-cap, visits, and third-bird-offer cadence so we can ramp safely; rollback is a worker-version swap with idempotent ticks. Risks are dominated by drift miscalibration, audio uncanniness, and accessibility regression — each has a named mitigation rather than a hopeful note. The plan deliberately resists the most reachable alternatives (CRDTs, recorded audio, "harmless" toasts) because each one would erode a load-bearing property of the product the PRD is asking us to build.
