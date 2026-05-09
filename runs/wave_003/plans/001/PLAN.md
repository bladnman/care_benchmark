# Pocket Aviary — v1 Implementation Plan

This is an executable plan for a frontier engineering team to build Pocket Aviary v1. It interprets the PRD into concrete architecture, data models, protocols, simulation math, rendering and audio pipelines, accessibility surfaces, performance budgets, rollout, and risks. Where the PRD leaves room, this plan makes a defensible call and names it.

The plan inherits the PRD's load-bearing rules without restating them: presence is the conjunction of three signals; drift is monotonic toward expressive; the server is the only writer of personality state; calls are procedural; there is no gamification; the aviary appears already in motion. Any line below that appears to relax one of those rules should be read as a mistake and corrected — they are invariants, not defaults.

---

## 1. Scope

### In scope for v1

- Single-user accounts via email magic link; one canonical aviary per account.
- Two starter birds at adoption; cap of seven; aviary-age-paced offers for additional birds.
- Six-species pool with distinct silhouettes, default plumage palettes, and call-grammar motif libraries.
- Server-side simulation tick (~1 minute cadence) advancing personality vectors, mood, and call timing.
- Slow personality drift across five traits (boldness, social warmth, vocal frequency, plumage saturation, curiosity), monotonic toward expressive.
- Mood as a fast-timescale enumerated state (wary, content, curious, drowsy, alert), persisted across sessions.
- Procedural call synthesis client-side via WebAudio, with per-bird recognizable signatures and real chorus mixing.
- Single horizontal scene with three perch zones, day/night cycle anchored to the user's local time, ambient weather and micro-motion.
- Return-greeting that varies by absence length, bird boldness, and current mood — never canned.
- Listen-in (focus a bird, slow mix re-balance), Offer (seed, song fragment, still pool with per-bird cooldown), Settle (slow evening lighting, undo within 5s).
- Field notebook (auto-generated naturalist prose, sparse cadence, read-only, infinite scrollback).
- Multi-device sync as a property of server-authoritative state.
- Visit invitations (host-issued, per-invite opt-in, read-only ambient view, revocable, 30-day expiration on unused invites; 30-day soft account deletion; per-account session list with revocation).
- Accessibility as designed surface: screen-reader narration, reduced-motion mode (cross-fade rendering, not stripped fallback), procedural call captions, keyboard navigation, WCAG AA contrast on all user copy.
- Performance: <2MB initial JS bundle (gzipped), <500ms time-to-first-bird on mid-tier mobile over 4G, 60fps idle motion on a 5-year-old laptop, no measurable memory growth over 30 minutes.
- Account export (JSON, on demand, emailed link).
- Aggregate-only telemetry; no per-bird or per-account interaction state in any analytics pipeline.

### Out of scope (PRD-named non-goals; respected absolutely)

- No native mobile apps; no plan to build them.
- No gamification of any flavor: no streaks, achievements, badges, levels, scores, ranks, tiers, "days visited," green-dot calendars, XP, milestones, "you've been here every day this week" surface, hidden engagement counters, or quiet opt-in dashboards equivalent to any of these. This rule is absolute and rules out adjacent disguises (visit-frequency widgets, exportable visit logs framed as engagement, notebook entries observing user behavior rather than the aviary).
- No Tamagotchi mechanics: birds do not die, get hungry, become distressed, mistrust the user, or drift in a negative direction on neglect. Neglect produces ambient quietness, not punishment.
- No social-network surfaces: no profiles, follows, public feed, discovery, comments, leaderboards, friend-of-friend chains, co-presence, "show-off" rendering for visitors, or "your friend visited!" notifications by default.
- No notifications by default (push, email, in-product). The aviary lives where the user visits it.
- No personality-vector exposure to the user, ever — no debug toggle, no stats panel, no inspector, no advanced setting. Hard rule at the API layer, not just the UI.
- No client-side simulation tick; clients never write personality state.
- No recorded-audio fallback if WebAudio fails; silence with captions is the documented fallback.
- No "Welcome back!" toast, banner, modal, or text on return. The bird greeting is the entire welcome.
- No streak counter, days-visited surface, or any visit-frequency aggregate exposed to the user, in any tier or version.
- No public discovery, leaderboards, or any cross-account aggregate visible inside the product.

### Defensible interpretive calls

Where the PRD leaves room, this plan commits:

- **Tick cadence:** 60 seconds. Catch-up ticks process up to 24 hours of accumulated mood/drift in a single batched compute when the simulation has been idle (e.g., a fresh login on an account with no live ticking; see §6 on tick scheduling and dormancy).
- **Presence activity window:** 4 minutes. A pointer-or-key event within the last 240 seconds satisfies the activity condition, alongside `visibilityState === 'visible'` and `document.hasFocus()`.
- **Presence ping cadence:** 15 seconds while all three presence conditions hold; pings stop the moment any condition fails. Pings carry a monotonic client clock and are server-clamped on receipt to bound clock-skew abuse.
- **Snapshot pull cadence:** on visibility change, on render-frame gaps >2s (suspend detection), and a 30-second keepalive while visible.
- **Listen-in mix ramp:** 1.2s linear-in-amplitude crossfade on engage and disengage.
- **Offer per-bird cooldown:** 5 minutes per bird per offer kind.
- **Settle undo window:** 5 seconds (PRD-named).
- **Notebook entry cadence:** at most one entry per 36 hours under typical use; a sparsity scheduler caps dense periods.
- **Magic link expiry:** 15 minutes (PRD-named); used links invalidated immediately.
- **Soft-delete window:** 30 days (PRD-named).
- **Invite expiry:** 30 days (PRD-named); revoked at next snapshot pull on visitor side.
- **Bundle budget:** 2MB gzipped at first paint (PRD-named); route-split anything past the first-bird path.
- **First-bird budget:** 500ms p75 on a mid-tier mobile device over 4G, on a warm CDN; 800ms p95.
- **Tick latency:** p99 under 5s before alarm (PRD-named).

Where the PRD says "implementation detail," that detail lives in the spec sections that follow.

---

## 2. Architecture

### High-level shape

A small set of services, deliberately fewer than a typical microservice diagram. The product is small; the architecture should be small.

```
+-------------------+        +----------------------+        +--------------------+
|  Web client       |  HTTPS |  Edge / CDN          |  HTTPS |  API gateway       |
|  (React app +     | <----> |  (static assets,     | <----> |  (REST + WS)       |
|   WebAudio +      |        |   snapshot edge      |        |                    |
|   Canvas/SVG)     |        |   cache, magic-link  |        +-----+--------+-----+
+-------------------+        |   redirect)          |              |        |
                             +----------------------+              |        |
                                                                   v        v
                                                 +-----------------+--+   +-+----------------+
                                                 |  Aviary service     |  |  Auth service    |
                                                 |  (snapshots,        |  |  (magic link,    |
                                                 |   event ingest,     |  |   sessions,      |
                                                 |   visit reads)      |  |   email change)  |
                                                 +-----------+---------+  +------------------+
                                                             |
                                              +--------------+--------------+
                                              v                             v
                                  +-----------+----------+      +-----------+------------+
                                  |  Simulation worker   |      |  Postgres (canonical   |
                                  |  (server-side tick,  |<---->|  aviary state, events, |
                                  |   notebook writer)   |      |  accounts, invites)    |
                                  +----------------------+      +------------------------+
                                              |
                                              v
                                  +-----------+----------+
                                  |  Email service (SES) |
                                  |  (magic links,       |
                                  |   exports, invites)  |
                                  +----------------------+

+----------------------+        +---------------------------+
|  Aggregate telemetry |  push  |  Operational data store    |
|  pipeline            | -----> |  (no per-account fields)   |
+----------------------+        +---------------------------+
```

### Services

- **Web client.** React + TypeScript SPA. Renders aviary in a single `<canvas>` (with SVG/CSS chrome layered above). Runs WebAudio. Holds presence state and emits ping events. Pulls state snapshots; never writes personality state. Code-splits non-first-bird surfaces (settings, notebook drawer, accessibility settings, visit-invite flow, account deletion).
- **API gateway.** Thin authn/routing layer. Verifies session token, rate-limits by account UUID, attaches request-scope context. Handles WebSocket upgrade for the ambient stream.
- **Aviary service.** Reads canonical state from Postgres. Returns snapshots. Accepts interaction events into the append-only event log. Serves visit-mode read-only state. No write to personality state lives here.
- **Auth service.** Magic-link issuance, redemption, session token issuance and revocation, email change verification, soft-delete state machine.
- **Simulation worker.** The only writer of personality state. Runs the tick loop per account on a per-second scheduler; catches up when an account has been dormant. Generates notebook entries. Computes mood transitions and drift deltas. Reads the event log; writes canonical state and the notebook.
- **Email service.** A thin wrapper over a transactional email provider (SES at v1) for magic links, export download links, invitations, deletion confirmations.
- **Postgres.** One operational database for accounts, birds, personality vectors, mood, the event log, the notebook, invites, sessions. Single-region primary with a read replica for snapshot serving. Logical separation of the `simulation` schema from the `account` schema enforces the privacy-pipeline boundary at the access-layer level.
- **Aggregate telemetry pipeline.** Operational metrics only. No per-account dimension. Hard separation from the simulation database — telemetry collectors do not have credentials to read it.

### Client/server split (load-bearing)

This is the decision the rest of the architecture follows from:

- **Server is the only writer of personality state.** No client code path mutates a personality vector. The client emits interaction events; the simulation worker consumes them.
- **Client interpolates between server snapshots.** Bird positions, motion phases, and call timing are predicted forward by the client between snapshots, but the canonical state is always the server's. On snapshot arrival, the client gently reconciles its predicted state with the snapshot (ease toward the snapshot over ~250ms; never teleport unless the divergence exceeds a threshold).
- **Notebook is server-generated.** Entry-generation logic runs in the simulation worker, not on the client.
- **Audio is client-side.** WebAudio synthesis happens on the client because (a) bandwidth/latency cannot serve it, (b) the chorus mechanic requires per-call procedural variation in real time, (c) the bundle budget cannot afford recorded audio at the variation needed.
- **Visit-mode is a read-only client.** The visitor's client pulls snapshots like the host's, but every write path (event log, settle, listen-in side effects) is no-op'd at the API gateway based on session role.

### Render pipeline boundary

The boundary between simulation and rendering is deliberately strict:

- Simulation owns: personality vectors, mood states, mood-shaped behavior decisions (which bird greets, when a bird shifts perch, when a call is generated), call grammar parameter selection (which motif, which timing), notebook entry generation.
- Rendering owns: how a bird's idle micro-motion is drawn between snapshots, how perch movements are animated, how the day/night palette interpolates, how leaves and feathers drift, how the listen-in crossfade ramps, how the focus indicator is drawn.
- Calls cross the boundary at a defined seam: simulation emits a `call_event` in the snapshot ("Pip will start a call at server time T with motif M, mood-modulated parameter set P"); the client's audio engine renders that call from the motif library at the right time. The client never decides to *generate* a call; it decides only how to *play* a call the simulation already generated.

This seam is what keeps multi-device coherence: two devices receiving the same `call_event` synthesize the same call (modulo small per-device synthesis jitter that does not break recognizability).

### Repository / build shape

A single monorepo with three top-level packages: `client/`, `server/`, `shared/` (call grammar definitions, snapshot schema, type definitions). Build and deploy as separate artifacts but version the shared schema together to prevent client/server schema drift.

---

## 3. Data model

All identifiers are synthetic UUIDs. Email lives only on the account record, encrypted at rest, and is never used as a key, partition, log dimension, or telemetry attribute (PRD: "synthetic account ID" — non-negotiable).

### Postgres schemas

Two logical schemas: `account` and `simulation`. Telemetry has no read access to either.

#### `account.accounts`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | synthetic, generated at creation |
| `email_encrypted` | bytea | KMS-encrypted email; only column carrying email |
| `email_hash` | bytea | HMAC of email for lookup at sign-in (single-use, bounded scope) |
| `created_at` | timestamptz | |
| `deletion_initiated_at` | timestamptz null | sets the soft-delete clock |
| `deleted_at` | timestamptz null | hard-delete tombstone; row is purged from this table once children are purged |
| `notify_on_visit` | bool default false | per-account toggle; default off |
| `prefs_json` | jsonb | accessibility settings (reduced-motion override, captions on/off, audio on/off), aviary-name (non-essential) |

The `email_hash` is used only on the sign-in path to find the account UUID; it never appears in any other query plan, log, or analytics event.

#### `account.sessions`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | session token (HMAC-validated; the bare row id is the public token) |
| `account_id` | uuid fk | |
| `device_label` | text | derived from User-Agent at issuance, displayed in session list |
| `created_at` | timestamptz | |
| `last_seen_at` | timestamptz | refreshed on each request |
| `revoked_at` | timestamptz null | |

#### `account.magic_links`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | one-time link token |
| `email_hash` | bytea | for lookup; entry is short-lived |
| `created_at` | timestamptz | |
| `consumed_at` | timestamptz null | invalidated on consumption |
| `expires_at` | timestamptz | created_at + 15 minutes |
| `kind` | enum (`signin`, `email_change`, `export`, `invite`) | |
| `payload_json` | jsonb | per-kind payload (target email for email_change; aviary id for invite) |

#### `simulation.birds`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | stable bird identity; never reissued for any reason |
| `account_id` | uuid fk | |
| `species_id` | text fk | references the species pool |
| `name` | text | user-editable; renaming never affects identity |
| `adopted_at` | timestamptz | |
| `seed` | bytea(16) | random per-bird; seeds initial vector and per-bird call variation |
| `personality` | jsonb | the personality vector |
| `mood` | jsonb | current mood state with timers |
| `position` | jsonb | current perch and on-perch jitter |
| `last_tick_at` | timestamptz | last time the simulation worker advanced this bird |

The `personality` column shape:

```json
{
  "boldness": 0.42,
  "social_warmth": 0.55,
  "vocal_frequency": 0.38,
  "plumage_saturation": 0.30,
  "curiosity": 0.48,
  "drift_history": {
    "boldness_low_pass_state": 0.18,
    "social_warmth_low_pass_state": 0.22,
    ...
  },
  "version": 1
}
```

Each trait is a scalar in `[0, 1]`. The `*_low_pass_state` fields store the running low-pass filter state used by the drift function so that drift is not recomputed from scratch each tick. `version` allows safe schema migration without rebuilding personality from event logs.

The `mood` column shape:

```json
{
  "state": "content",
  "entered_at": "2026-05-08T06:12:33Z",
  "decay_to": "drowsy",
  "decay_at": "2026-05-08T07:30:00Z",
  "modulators": { "weather": "calm", "tod_phase": "morning" }
}
```

#### `simulation.events` (the append-only interaction log)

| Column | Type | Notes |
|---|---|---|
| `id` | bigserial pk | |
| `account_id` | uuid fk | |
| `bird_id` | uuid null | null for aviary-wide events |
| `kind` | enum | `presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `tab_focus`, `tab_blur`, `visit_open`, `visit_close` |
| `client_ts` | timestamptz | client's monotonic clock at event |
| `server_ts` | timestamptz | server-clamped on receipt |
| `payload_json` | jsonb | per-kind payload |
| `consumed_by_tick_at` | timestamptz null | set when the simulation has applied this event |

Append-only. Never updated except for setting `consumed_by_tick_at`. Retention is rolling 90 days; older rows are summarized into long-term `presence_minutes` aggregates per account-day (no per-bird dimension in the rolled-up form, since rolled-up data is for tick warm-up only).

#### `simulation.notebook_entries`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | |
| `account_id` | uuid fk | |
| `created_at` | timestamptz | |
| `prose` | text | naturalist prose, lowercase, present-tense |
| `referenced_birds` | uuid[] | for narration cross-reference if needed |
| `kind` | enum | `morning_observation`, `chorus`, `first_greeter`, `weather`, `mood_shift`, `quiet`, `aviary_age`, etc. |

Read-only from the user's side; only the simulation worker writes here.

#### `simulation.aviary_state`

Per-account aviary-wide state: weather state machine, day/night phase derived from the user's stored timezone, current chorus events, an aviary-age-based "next bird offer" timer.

#### `simulation.invites`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | invite token |
| `host_account_id` | uuid fk | |
| `visitor_email_encrypted` | bytea | |
| `created_at` | timestamptz | |
| `expires_at` | timestamptz | created_at + 30 days |
| `revoked_at` | timestamptz null | |
| `last_visited_at` | timestamptz null | for visit log |
| `total_visit_seconds` | int default 0 | for visit log; not used by the simulation |

#### `simulation.visit_events`

Aggregate "who visited and when" rows that power the host's visit log. Strictly local to the host's account; never used for any cross-account analytic.

### Personality vector — invariants

- **Range.** Each trait in `[0, 1]`, stored as `numeric(6,5)` to avoid floating-point drift across migrations.
- **Initialization.** Each starter bird's seed-derived vector is sampled from a species-conditional distribution, with a target mean offset to put new birds in the "shy / unestablished" range (boldness 0.30–0.45, plumage 0.25–0.35, vocal frequency 0.30–0.45, social warmth 0.30–0.50, curiosity 0.35–0.50). Concrete distributions are listed in §5.
- **Monotonicity.** A trait value never decreases as a result of a drift update. Idempotent enforcement: the drift update is `personality[trait] = max(personality[trait], personality[trait] + clamp(delta, 0, max_step))`. The `max(...)` is belt-and-suspenders — even if a future code path miscomputes a delta as negative, the floor saves the user's bird.
- **Bound.** A trait value never exceeds 1.0; the drift function uses a sigmoid-shaped diminishing-returns curve (§5) so that highly drifted traits drift more slowly, which keeps drift "feels measurable at week 1, visible at week 3" calibration intact at long timescales without ever flatlining.
- **Never exposed.** No API ever returns a personality vector value to a client. This is enforced at the snapshot serialization layer, with a unit test that asserts every public response shape excludes raw vector fields. Internal admin tooling that does expose the vector is a separate authenticated surface available only to the simulation team for calibration; it is not part of the product.

---

## 4. API surface

Two protocols: REST/HTTPS for state pulls, event submission, and account management; a thin WebSocket channel for ambient call-event push to the client. WebSocket is optional — the system also functions over polling, falling back automatically. We use WebSocket for affective fidelity (calls that line up tightly with the simulation's intent), not for correctness.

All endpoints require a session token except the auth endpoints. All responses are JSON. All endpoints are versioned `/v1/...`.

### Auth

```
POST /v1/auth/magic-link
  body: { email: string }
  201: {} (always returns 201; no signal that the email is registered, to avoid enumeration)
  rate-limit: 5/email/hour, 30/IP/hour

GET /v1/auth/redeem?token=...
  302 to client app on success with session cookie set
  302 to client app with ?error=expired on expired token
  302 to client app with ?error=invalid on invalid/used token

POST /v1/auth/email-change
  body: { new_email: string }
  201: {}  (sends verify link to new email; old email continues to work until verified)

GET /v1/auth/email-change/verify?token=...
  302 to settings on success

POST /v1/auth/sessions/:session_id/revoke
  204

POST /v1/auth/sign-out
  204

GET /v1/auth/sessions
  200: [ { id, device_label, created_at, last_seen_at } ]

POST /v1/account/delete
  201: { soft_deleted_until }

POST /v1/account/cancel-delete
  204

POST /v1/account/export
  201: {}  (emails an export download link; honors current verified email)
```

### Aviary state

```
GET /v1/aviary/snapshot
  200: AviarySnapshot
```

The `AviarySnapshot` is the load-bearing read shape. Its schema is shared with the client (see `shared/`):

```ts
type AviarySnapshot = {
  schema_version: 1;
  server_now_ms: number;     // server clock at snapshot generation
  account_id: string;        // synthetic UUID; sanity-checked at client
  birds: BirdSnapshot[];     // 2..7 entries
  weather: WeatherState;
  day_night: DayNightPhase;  // derived from the account's tz at snapshot time
  upcoming_calls: CallEvent[];   // call events scheduled in the next ~120s
  upcoming_actions: ActionEvent[]; // mood-shaped non-call actions (perch shifts, head-tilts, fly-ins)
  notebook_recent_count: number;  // for the notebook drawer pip
  visit_mode: false;         // true only on the visitor read endpoint
  next_pull_hint_ms: number; // suggested keepalive interval
};

type BirdSnapshot = {
  id: string;
  species_id: string;
  name: string;
  perch: 'front' | 'middle' | 'back';
  perch_offset: { x_norm: number; y_norm: number }; // small jitter, normalized
  facing: 'left' | 'right';
  mood: MoodState;        // public summary; no raw modulator values
  pose_phase: 'preen' | 'scan' | 'tilt' | 'rest' | 'alert' | 'mid_call' | 'arrival' | 'departure';
  pose_phase_started_ms: number;
  // No personality vector. Ever.
};

type MoodState = {
  state: 'wary' | 'content' | 'curious' | 'drowsy' | 'alert';
  // No numeric modulators. Public summary only.
};

type CallEvent = {
  bird_id: string;
  start_at_server_ms: number;
  motif_id: string;
  // a small, capped set of public parameters that the client uses to synthesize:
  pitch_offset_semitones: number;  // bounded
  tempo_modifier: number;          // bounded
  intensity_norm: number;          // 0..1, mood/personality-shaped
  duration_ms: number;
  caption_prose: string;           // generated server-side from the same call grammar
};
```

`BirdSnapshot` carries a *public summary* of mood for two reasons: (a) the client needs mood for idle-motion shaping, (b) any richer numeric exposure leaks personality. The mapping from the canonical mood (with internal modulators and timers) to the public mood enum lives at the snapshot serializer.

### Interaction events

```
POST /v1/aviary/events
  body: { events: ClientEvent[] }   // batched; max 50 per request
  202: { accepted: int, rejected: ClientEventRejection[] }
```

Events are write-only on the client side; the response body never returns derived state. The client batches events on a 1-second tick to amortize request cost. Submitted events:

- `presence_ping` — body `{ at_client_ms, visibility, focus, last_input_ms_ago }`. Server validates the conjunction; if any condition is absent or implausible, the event is dropped silently and not counted toward presence-time. The client sends the raw signals for server-side validation rather than a boolean "I am present" so the client cannot misrepresent presence.
- `listen_in_start` / `listen_in_end` — body `{ at_client_ms, bird_id }`.
- `offer` — body `{ at_client_ms, bird_id, kind: 'seed' | 'song_fragment' | 'still_pool', song_motif_id?: string }`. Server enforces cooldown.
- `settle` — body `{ at_client_ms }`.
- `settle_undo` — body `{ at_client_ms }`.
- `tab_focus` / `tab_blur` — for presence accounting context, not as primary presence signal.

The server clamps `at_client_ms` against `server_now_ms` to prevent clock-skew abuse: if the client claims a timestamp more than 30s off the server clock, the event is normalized to `server_now`.

### Ambient stream (optional)

```
WS /v1/aviary/stream
  server -> client messages: { type: 'snapshot_delta' | 'call_event' | 'visit_revoked', ... }
  client -> server messages: heartbeats only; no event submission over WS
```

The WebSocket is a push channel only. All writes still go through `POST /v1/aviary/events` to keep the audit/allowlist surface narrow. If the WS drops, the client falls back to the 30s pull keepalive without any user-visible state change.

### Visit (host side)

```
POST /v1/visits/invite        body: { email }     201: { invite_id, expires_at }
GET  /v1/visits                                  200: [ { id, visitor_email, created_at, expires_at, last_visited_at, total_visit_seconds, revoked_at } ]
POST /v1/visits/:id/revoke                       204
```

### Visit (visitor side)

```
GET /v1/visits/:invite_token/snapshot
  200: AviarySnapshot with visit_mode=true and writable affordances stripped
  410: { reason: 'expired' | 'revoked' | 'invalid' }
```

The visitor endpoint enforces:

- No event submission accepted on the visitor's session — gateway-level reject.
- Visit-mode snapshots strip the `notebook_recent_count` field (visitors can't see it) and add a small "visit_session" bookkeeping tag for the host's visit log.
- Visitor presence is *not* counted toward the host's drift signal. This is enforced at event ingest, where visitor sessions cannot emit `presence_ping`.

### Notebook

```
GET /v1/notebook?cursor=...    200: { entries: NotebookEntry[], next_cursor: string | null }
```

Read-only; no write endpoint. Cursor is opaque (server-side pagination).

### Privacy and shape rules baked into the API

- No endpoint returns a personality vector value, anywhere, on any path, including admin-shaped paths in the public surface.
- No endpoint returns "days visited," "session count," "last visit," "streak," or any aggregate that a client could surface as a visit-frequency widget. The host's visit log is per-invite, not per-account-frequency, and it does not expose calendar shapes that resemble streak surfaces.
- The notebook never emits entries about the user's behavior frequency. The kind enum is finite and does not include any "you visited every day" shape; the prose generator's allowed-template set excludes user-behavior observations.
- The visitor endpoint never returns the host's notebook contents. Notebook is private to the host.

---

## 5. Simulation engine design

The simulation is the substantive core of the product. This section is specific because the calibration is what makes "feels alive over weeks" a property of the build, not a hope.

### 5.1 Tick cadence and scheduling

The simulation worker runs a tick loop per account with a target cadence of 60 seconds. The scheduler holds two queues:

- **Live queue.** Accounts with a session-pinged in the last 5 minutes. These tick on the 60s cadence to keep mood and call-event scheduling fresh for an active client.
- **Dormant queue.** Accounts not in the live queue. These tick on a 15-minute background cadence to advance day/night, weather, and slow drift. Dormant accounts also receive a *catch-up* tick when a client session resumes.

The catch-up tick takes the time delta since `last_tick_at` and applies it as a single composite update rather than spinning the tick N times. The composite-update formula for drift, mood, and ambient state is in §5.5 and §5.6.

The reason for this two-queue design: the live cadence keeps the user's experience continuous; the dormant cadence keeps the aviary continuing without the viewer (PRD: "the aviary the user comes back to is the aviary that has been running"); the catch-up tick keeps cost flat as users dip in and out without losing the felt-continuity.

### 5.2 Simulation idempotence

Every tick is deterministic given (state, time delta, event log slice). A re-run of the same input produces the same output. This matters for two reasons: testability (we can replay tick sequences in CI) and crash recovery (if a worker dies mid-tick, the partial state is rolled back and re-run cleanly).

Each tick:

1. Reads (with `FOR UPDATE`) the bird rows and the unconsumed events for the account.
2. Computes the new mood, drift, position, and upcoming call/action events.
3. Writes the new bird rows and `consumed_by_tick_at` on the events.
4. Commits.

A tick is not a stream; it is a transaction. The event log is consumed in `id` order. Two simultaneous ticks for one account are prevented by an account-level advisory lock at the worker.

### 5.3 The presence signal

Presence-time is the drift function's primary input, and its definition is the load-bearing precision named in `concepts.md`. The implementation:

- The client emits a `presence_ping` every 15 seconds while all three conditions hold:
  - `document.visibilityState === 'visible'`
  - `document.hasFocus()` (with a polyfill check on browsers where `hasFocus` is unreliable)
  - A `pointermove` or `keypress` event has occurred within the last 240 seconds.
- The server normalizes consecutive pings into presence intervals. Two pings within 20 seconds of each other (allowing one missed ping) extend the current interval; otherwise a new interval starts.
- **Visit-mode emits no pings.** The client suppresses pings when `visit_mode: true`; the server independently rejects pings from a visitor session even if the client is forged.
- A presence interval terminates on any of: `tab_blur`, settle, the 20s gap above, or session timeout.
- Per-account presence-minutes are summed into a rolling 30-day window, decayed exponentially so older minutes weigh less than recent ones (half-life: 7 days). This is `presence_recent` and feeds the drift function.

**Why this shape.** The conjunction is the precision the PRD names. The 15s/20s ping cadence absorbs network jitter without inflating presence on a flaky connection. The 4-minute activity window matches the PRD's "lean toward the longer side because watching birds without moving is the actual product." The 7-day half-life on `presence_recent` tunes the drift "measurable at week 1, visible at week 3" target.

### 5.4 Mood transitions

Mood is a small enumerated state machine per bird with five states and named transitions. Mood is updated every tick; transition probabilities depend on (a) recent interactions, (b) time-of-day phase in the account's stored timezone, (c) ambient events (weather, alarm calls), (d) the bird's own personality vector.

Transition table (from → to, condition; multiple matching conditions are evaluated by max priority and a small randomness budget):

| From | To | Condition |
|---|---|---|
| any | content | offer accepted; or first listen-in within last 60s on this bird |
| any | curious | song-fragment offer played within last 30s; or another bird's call within last 5s with high social warmth |
| any | drowsy | tod_phase ∈ {late_evening, night} for ≥10 minutes |
| any | alert | alarm-call event in last 5s; or weather:wind started in last 10s; lower probability for high-boldness birds |
| any | wary | wary-call from another bird; or weather:rain started in last 30s; lower probability for high-boldness birds |
| drowsy | content | tod_phase shifts to morning AND presence active |
| any | content | mood entered_at older than 30 minutes AND no triggering event AND tod_phase ∈ {morning, midday} |

Each tick, the mood state may transition at most once. The transition is sampled from the allowed transitions weighted by priority and personality. Boldness reduces wary/alert probabilities; social warmth raises content/curious probabilities.

Mood persists across sessions: the row in `simulation.birds.mood` is the canonical mood. The user never sees a "mood reset" on tab open because there is none — the mood the user sees is the mood the server already computed for now.

### 5.5 The drift function

Drift is a low-pass filter over presence-and-interaction signals, applied per trait, monotonic toward expressive.

Per tick, for each bird and each trait, the drift update is:

```
input_signal = w_presence * presence_recent_norm
             + w_listen_in * listen_in_recent_norm
             + w_offer * offer_recent_norm
             + w_settle * 0  # settle does not push drift; PRD-named
```

Each `*_recent_norm` is the normalized recent-window signal for this bird-and-trait, with traits weighing different signals (Listen-in pushes social warmth and vocal frequency; Offer pushes curiosity and a small boldness; Presence pushes plumage saturation, vocal frequency at lower weight, and a slow general drift).

Then:

```
low_pass_state[trait] = (1 - alpha) * low_pass_state[trait] + alpha * input_signal
delta = beta * sigmoid_diminishing(personality[trait], low_pass_state[trait])
delta = clamp(delta, 0, max_step_per_tick[trait])  # monotonic
personality[trait] = clamp(personality[trait] + delta, 0, 1)
```

Calibration constants (initial, tunable in production with a guardrail that prevents online tuning from crossing the monotonicity invariant):

- `alpha = 0.02` per tick at 60s cadence — the low-pass time constant works out to roughly a week.
- `beta = 0.0007` per tick — a typical user gets `delta ≈ 5e-4` per tick on a presence-active session; over a week of regular visits this accumulates to ~0.02 of trait change, which is "measurable in instruments." Over three weeks ~0.06, which is "visible to the user" given that visible mood-shaped behavior thresholds are ~0.05 apart.
- `max_step_per_tick` per trait: 0.001 for plumage and social warmth, 0.0015 for vocal frequency and curiosity, 0.0008 for boldness. Boldness drifts more slowly because it is the most visible trait moment-to-moment (perch position) and we want cumulative-effect-style change there.
- `sigmoid_diminishing` flattens delta as `personality[trait]` approaches 1, so traits don't saturate to a flatline at long timescales.

**Calibration test.** A simulation harness runs 21 simulated days against a synthetic "regular user" (30 minutes of presence per day, an offer every other day, three listen-ins per session). Targets:

- Mean drift across traits at day 7: between 0.015 and 0.030 (measurable in instruments).
- Mean drift across traits at day 21: between 0.05 and 0.09 (visible to the user as mood-shaped behavior threshold crossings).
- Drift over 14 days of zero presence: 0 (no negative drift, no decay-to-baseline; bird becomes ambient by virtue of mood drifting toward drowsy/wary in low-presence conditions, not by its personality vector decreasing).

The harness output is a CI artifact; calibration drift in the harness output is alarmed.

**Asymmetry rule (load-bearing).** Drift is monotonic toward expressive. There is no negative drift on neglect. The implementation enforces this at the `clamp(delta, 0, max_step)` step. A code path that mistakenly produces a negative delta produces a 0 update, not a decrement. This rule is the implementation of the PRD's "no Tamagotchi" non-goal at the engine layer; everything else cosmetic depends on it.

**What "becomes ambient" means.** A bird that has not been visited often is not less colorful or less bold; it is in a mood that calls less often. Mood is the visible variable; personality is fixed-or-rising. The user who comes back after two weeks finds birds whose vocal frequency hasn't dropped, but whose mood has drifted to drowsy/content-low-energy because tod-phase progression and absence of attention-modulators have nothing pushing them toward alert/curious.

### 5.6 The call grammar runtime

Each species has a motif library — a small set of motif templates (typically 4–8 motifs per species), each described by:

- A pitch-contour curve (relative semitones over time).
- A tempo envelope (note durations).
- An intensity envelope (amplitude over time).
- A "joinability" score for the chorus mechanic.
- A mood-conditional usage table (which motifs are used in which moods).

A call is generated by:

1. Picking a motif from the species' library, conditioned on mood and personality. High vocal frequency raises the probability of "joining" motifs.
2. Sampling per-call variation: pitch offset (±2 semitones), tempo modifier (±15%), intensity (mood-shaped), small ornament insertions.
3. Constructing a concrete `CallEvent` with these parameters and a server timestamp.

**Recognizability.** Each species has a "signature" motif — the one a typical listener can identify across mood and personality drift. The signature is sampled with high probability across all moods (e.g., 60% of calls of a high-vocal-frequency species use its signature motif, with the remaining 40% drawn from the rest of its library). The recognizability calibration target is: in a blinded listening test, a user who has spent 2 weeks with a bird identifies its signature in ≥85% of trials when played alongside other species' signatures.

**Per-bird variation.** The bird's seed perturbs the species library deterministically — a slight pitch-base shift, a slight tempo bias — so that two birds of the same species are individually recognizable while clearly being the same species. This is a small but load-bearing detail: a user with two warblers should know "her warbler" by ear.

**Chorus.** When two or more birds with high `vocal_frequency` and `social_warmth` have call events scheduled in overlapping windows, the simulation snaps their start times to within a small jitter so the calls overlap musically rather than collide. The runtime mixer (client-side) then mixes them with phase-aware envelope so the result is a real chorus, not stacked tracks. Chorus events are notebook-eligible.

### 5.7 The aviary-wide state

Beyond birds, the simulation tracks:

- **Day/night phase.** Derived from the account's stored timezone and the server clock. Phases: `dawn`, `morning`, `midday`, `afternoon`, `evening`, `dusk`, `night`. Phase transitions feed mood (drowsy at dusk; alert in early morning).
- **Weather.** A slow Markov chain: `calm` → {calm, light_wind, rain} with low transition probabilities (so weather is rare). A weather event lasts 4–10 minutes.
- **Aviary age.** The account's `created_at` drives a "next bird offer" timer. Offers appear at ~3, ~10, ~28, ~80, ~200 days, spaced super-linearly so that early growth is more visible and later growth is rarer. The user must accept an offer for a bird to appear.
- **Notebook entry-generation logic.** Runs at the end of each tick on the live queue and once per dormant tick. Generation is sparsity-controlled: a hard floor of 36 hours between entries under typical conditions, raised to a soft floor of 18 hours when something genuinely noteworthy happens (a chorus, a first-greeter shift, an unusually long quiet). The generator prefers specific particulars (named birds, specific moments) over generic states. The prose templates are voice-checked offline; the generator picks a template family based on the eligible event kinds and fills it with concrete particulars.

### 5.8 Snapshot generation

A snapshot is the read view of the simulation at server time T. It includes:

- The current bird rows (mood public-mapped, no personality).
- The next ~120s of `CallEvent`s and `ActionEvent`s, scheduled by the simulation in advance.
- The current weather/day-night phase.
- A `next_pull_hint_ms` to guide the client's keepalive.

Snapshots are computed lazily on `GET /v1/aviary/snapshot` from the canonical state. They are *not* a separate cached document that can drift from the canonical state. We do cache snapshots for a few hundred ms at the edge for the case of a snapshot pulled by both the client and a hot-reload — this is a request-coalescing cache, not a stale-while-revalidate cache.

---

## 6. Sync model

Multi-device sync is not a feature; it is a property of the architecture (PRD, accounts_sync.md). This section spells out exactly how the property is enforced.

### 6.1 Single canonical record

Postgres holds the canonical state. The simulation worker is the only writer of personality vectors and mood. Every client of every device reads from snapshots derived from this single record.

### 6.2 No client writes to personality

The API gateway enforces this at the schema level: the request body schemas for client-facing endpoints do not include personality fields. The simulation worker is the only service with write access to `simulation.birds.personality`. This is enforced by the database role used by the aviary service (read-only on the personality column) versus the simulation worker's role (read-write).

### 6.3 Additive deltas, never absolute values

A client never sends "set boldness to X." A client sends "user listened in to bird Y for Z seconds." The simulation worker decides what that means for personality. This rules out the last-write-wins failure mode (PRD: "morning's drift is silently deleted").

The append-only event log is the substrate. The simulation tick processes events in `id` order, applies deltas, and marks events `consumed_by_tick_at`. Two devices submitting events in the same window produce one ordered event log; the tick consumes them in order; the resulting personality update is the composition of the deltas — never the overwrite of one by the other.

### 6.4 Conflict prevention vs. resolution

There is no conflict to resolve because there is no two-writer state. The PRD asks for "no last-write-wins for personality state" — the implementation answer is "personality state has one writer." This is a sharper property than conflict resolution: there is no algorithm to argue about because there is no conflict surface to argue over.

The places where multi-writer-shaped problems still appear, and how they're handled:

- **User-editable bird name.** Two devices renaming Pip to two different names within seconds. Resolution: last-write-wins by `server_ts`. Names are not load-bearing for identity; renaming is intentional and the user can re-rename. We do not model "name conflicts" because the user's last action is the right answer.
- **Account-level prefs (audio on/off, captions on/off, reduced-motion override).** Last-write-wins by `server_ts`. Same reasoning.
- **Listen-in across devices.** Listen-in is per-session, not persisted. Each session has its own listen-in; one user with two open sessions gets two independently-mixed audio streams. The simulation treats each `listen_in_start`/`listen_in_end` pair as an attention signal for drift purposes; there is no conflict.
- **Settle.** A settle event on one device is canonical: it shifts the simulation's lighting to evening for the account. A second device open at the same time receives the settled state on the next snapshot pull and renders it. (We considered scoping settle per-session; the PRD frames settle as a goodbye to the aviary, not to a session, so the canonical-state model is the right one.)

### 6.5 Snapshot freshness across devices

The 30s keepalive plus visibility-change pull plus optional WS push gives a typical multi-device user the feeling of co-state: a settle on the laptop reflects on the phone within a snapshot interval. We do not promise sub-second cross-device propagation; we promise "within the next visible snapshot."

### 6.6 Session-token hygiene

Per-device session tokens are independent. Revoking a token on device A does not revoke device B. The session list (`GET /v1/auth/sessions`) lets the user see and revoke individual devices.

### 6.7 Soft-deletion across devices

When an account enters soft-delete, all sessions remain valid (the user must be able to sign in to recover). The aviary serves snapshots normally. Hard-delete after 30 days revokes all sessions and purges every record tied to the account UUID, including notebook entries, event log rows, invites, and visit records.

### 6.8 Cross-device account export

An export is generated server-side by reading the canonical state at request time and emailing a download link. The export is a JSON snapshot — bird names, current personality vectors (this is the *only* surface where the personality vector is exposed, and it is exposed to the user about their own birds in an explicit "your data" context, never inside the product UI), current moods, notebook entries, account settings.

This is an intentional narrow exception to the never-expose-personality-numerically rule: the rule is about not turning the product into a stat-management exercise, and a JSON export of "your own data" is not the product surface. We considered redacting the vector from the export and rejected it: the user's data is theirs; redacting it from their own export would be the wrong shape.

---

## 7. Frontend rendering pipeline

The client is a React + TypeScript SPA whose main job is to render an aviary that feels alive. The rendering pipeline is shaped to honor "the aviary appears with motion already in progress" as a real property, not an aspiration.

### 7.1 Stack

- React 18 for the chrome (top bar, settings drawer, notebook drawer, account flow). React only manages chrome and event wiring; it does not render the aviary scene.
- Canvas2D for the aviary scene (single full-bleed `<canvas>` behind the chrome). We considered WebGL; Canvas2D is enough for what we draw and avoids a class of GPU/driver quirks on older laptops, which matters for the 5-year-old-laptop performance budget. The scene is composed of a small number of layers (sky, parallax background, perches, birds, foreground ornaments), each drawn at its own update cadence.
- WebAudio for calls.
- A small custom scene-graph layer that holds bird sprite state, lerps it between snapshots, and drives the per-frame draw.

### 7.2 The first-frame property

The PRD requires that the first frame the user sees has birds mid-action. The implementation:

1. The HTML response from the edge embeds an inline `<script>` with the snapshot fetch initiated as a critical preload, plus a tiny "quiet field" CSS background so that even before any bird draws, the page is the correct hue (sky-blue for day, evening-warm for evening, dim for night) — never white, never a spinner.
2. The client bundle is split: a `boot` chunk under 200KB gzipped that includes the scene-graph, sprite loader for the species in this snapshot, and the WebAudio bootstrapping. Larger features (notebook, settings, visit-invite flow) are route-split.
3. The boot chunk's first job is to draw the snapshot: read the bird positions and mood, read the day/night phase, and render the first frame with each bird at its current pose-phase. There is no fade-in. The first frame is the aviary.
4. WebAudio context creation is deferred until first user interaction in browsers that require gesture-to-resume; until then the visual aviary plays normally and captions display if enabled. The activation prompt, when shown, uses matter-of-fact tone ("tap to enable sound") and is never wrapped in naturalist voice.
5. While the snapshot fetch is in flight, the page is the quiet field — a soft sky color and one or two faint cues (slow ambient color drift). No spinner. No "Loading aviary…" text.

### 7.3 Scene composition

The scene draws in a fixed layer order:

1. **Sky.** A full-canvas background whose color is a function of day_night phase + weather. Updates ~1 Hz; interpolates color smoothly between phases.
2. **Background parallax.** Soft foliage and distant cues. Mostly static; occasional ambient drift.
3. **Perches.** Three perch zones: front (close to the viewer), middle, back. Drawn statically with subtle weather-conditional tint.
4. **Birds.** Drawn at their current perch with on-perch jitter and pose-phase animation (preen, scan, tilt, rest, alert, mid_call, arrival, departure).
5. **Foreground ornaments.** Occasional leaves and feathers drifting through frame at a low cadence (one ornament per 8–25s on average). Generated client-side; no per-ornament server state.
6. **Top bar.** HTML/CSS layer above the canvas. Fades to nearly transparent after ~3s of cursor stillness (PRD-named); returns to full opacity on cursor or keyboard activity.
7. **Listen-in indicator.** A soft halo around the focused bird; rendered only when listen-in is active.
8. **Captions overlay.** When enabled, short prose appears near the calling bird, fading in/out with the call.
9. **Focus ring.** Keyboard focus indicator on the currently-focused bird; high-contrast against the aviary palette in both bright and dim states.

### 7.4 Frame loop

The frame loop runs at the display's refresh rate (typically 60 Hz, sometimes 120 Hz on high-refresh displays). The frame budget at 60fps is 16.67ms; we target 8ms aviary-frame work to leave headroom.

Per frame:

1. Update simulation-time interpolation for each bird (between the last two snapshots).
2. Update pose-phase animation for each bird (driven by `pose_phase` and `pose_phase_started_ms`).
3. Sky color interpolation (1Hz update; per-frame lerp).
4. Foreground ornament tick (rare).
5. Listen-in mix ramp progress (if engaged or disengaging).
6. Composite the canvas in layer order.

The frame loop pauses when `document.visibilityState !== 'visible'` (browser typically does this anyway via rAF throttling, but we explicitly suspend the rAF to ensure no battery drain). The simulation continues server-side.

### 7.5 Idle micro-motion

Idle micro-motion is mood-shaped. The PRD names it: a wary bird scans more, a content bird preens, a curious bird tilts toward sounds, a drowsy bird sits low and fluffed.

Implementation: each `pose_phase` is a small parametric animation (3–8 keyframes) with mood-conditional dwell times. The client picks the next `pose_phase` from a mood-conditional Markov chain seeded by the bird's `id`. The result is per-bird-recognizable micro-motion that varies per session without ever looking canned.

### 7.6 Transitions

- **Perch shift.** A bird shifting from middle to front animates over ~600–1200ms with an easing curve that includes a small mid-flight hover; the simulation schedules these as `ActionEvent`s.
- **Arrival.** A new bird flies in from the edge of the scene with a soft fly-in over ~1500ms.
- **Departure.** Bird departure does not exist in v1 (no mortality, no "leaving"); arrival is one-way.
- **Greeting motion.** The greeting is a `pose_phase` change (`scan` → `tilt` toward viewer → small step forward) plus a `CallEvent`. The combination is what the user reads as "the bird noticed me."

### 7.7 Reduced-motion mode

Reduced-motion is a designed alternate rendering, not a stripped fallback (PRD).

- Per-bird pose-phase animations replaced by slow cross-fades between still poses (each pose held for 4–8s, cross-fades over ~800ms).
- Perch transitions become cross-fades between perches over ~800ms instead of animated paths.
- Ambient leaf and feather drift removed entirely.
- Sky color shifts (day/night) preserved but slowed by 50%.
- Calls preserved at full quality (or replaced by captions per audio settings).
- Day/night phase transitions and mood transitions preserved.
- Focus ring preserved with no animation.

The reduced-motion engine is selected on boot from `prefers-reduced-motion` *or* from the user's accessibility preference (which can override `prefers-reduced-motion` in either direction). The selection lives at the scene-graph level — there is one renderer for full-motion and one for reduced-motion, sharing the same scene-graph and snapshot interpolation but rendering differently.

### 7.8 Responsiveness

Single horizontal scene. Aspect-ratio-preserving, with a "reframe" layout that ranges from 16:9 wide-desktop to 4:3 phone-portrait without cropping any bird. The perches and bird positions are normalized in scene-coordinates `[0, 1]`; the renderer maps them to viewport pixels per the active layout.

---

## 8. Audio pipeline

The audio is the affective spine. Procedural call synthesis client-side via WebAudio; chorus mixing; listen-in mix decay; WebAudio fallback to silence-with-captions.

### 8.1 WebAudio graph

```
[Synth voices, one per bird]   ->   [Per-bird gain node]   ->   [Listen-in mix bus]   ->   [Master gain]   ->   [Destination]
                                                                           ^
                                                                           |
                                                                  [Ambient bus] (continuous low-level mix when no calls)
```

- One synth voice per bird, allocated lazily when the bird first calls. Voices are reused for the bird's subsequent calls; we never allocate per-call.
- Per-bird gain node: governs that bird's mix level. When listen-in engages on bird X, X's gain ramps up linearly while every other bird's gain ramps down to ambient-level over 1.2s. Other birds never go silent (PRD: "Silencing them entirely would teach the user that the aviary is a set of things to switch between").
- Listen-in mix bus: a stable summing point so the listen-in ramp is implemented as gain changes at the per-bird node, not as bus reconfiguration.
- Ambient bus: a low-pass-filtered continuous quiet hum + occasional ambient sound at very low volume; provides the felt-not-heard "the aviary has its own air" presence even when no bird is calling. This is critical to felt-aliveness — a fully silent aviary between calls reads as paused.

### 8.2 Procedural synthesis

Each species has a motif library (§5.6); each bird's calls are synthesized from the library at runtime. Synthesis pipeline per call:

1. Server-side `CallEvent` arrives with motif id and parameters.
2. Client schedules the call at the WebAudio context time corresponding to `start_at_server_ms` (clock-aligned via the most recent snapshot's `server_now_ms`).
3. At the scheduled time, the synthesizer pulls the motif's pitch contour, tempo envelope, intensity envelope; applies the per-call parameters (pitch offset, tempo modifier, intensity); and renders the call into the synth voice using a small set of WebAudio building blocks (oscillators with envelopes, filtered noise for sibilance, a short reverb tail per species).

The synth keeps allocations bounded: a fixed pool of oscillator/filter nodes per voice, reused across calls. No per-call `new AudioContext`, no per-call `createBuffer` allocation. (The "no memory growth over 30 minutes" rule is partly enforced here; CI runs a 30-minute aviary and asserts heap and buffer stability.)

### 8.3 Chorus mixing

When the simulation snaps two calls into overlap (§5.6), the client renders both calls simultaneously through their own voices. Because each call is procedural and has a unique pitch contour and intensity envelope, the result is a chorus rather than a phase-cancellation mess. The mix is in real time at the destination node; we do not pre-render chorus events.

The mix has a small ducking compressor on the master to prevent peak clipping when ≥3 birds are calling simultaneously; this is the only dynamic-range processing on the chain and it sits at unity gain except in true peaks.

### 8.4 Listen-in mix decay

On `listen_in_start(bird_id)`:

- The focused bird's gain ramps to 1.0 over 1.2s.
- All other birds' gains ramp to ~0.25 (ambient level) over 1.2s.
- The ambient bus stays at its current level.

On `listen_in_end`:

- All bird gains ramp back to 1.0 over 1.2s.

Engagement and disengagement are triggered by user actions (focus on bird, click empty, keyboard focus change, focus a different bird, second click on focused bird).

### 8.5 The WebAudio fallback (silence + captions)

If the WebAudio context cannot be created (older browser, denied audio permission, hardware error), the audio engine enters a degraded state:

- All call events are dropped at the audio engine (no synthesis happens).
- Captions are forced on (regardless of the user's caption setting), so the user still sees what was about to be heard.
- A matter-of-fact one-time line in accessibility settings explains "audio isn't available; captions are showing what the birds are calling."
- The simulation is unaffected: birds still call (in the canonical state), the notebook still records them, drift still proceeds. The user's experience is silent-with-captions rather than silent-with-nothing.

We do not ship a recorded-audio fallback. The PRD names this rule and the reasoning is as PRD-stated: any quality below the procedural variation is canned, any quality above breaks the bundle budget, and silence-with-captions is the better fallback.

### 8.6 Audio safety and quality

- Master volume defaults at -6dB FS; users can adjust in accessibility settings.
- Calls are bounded in duration (≤5s) and intensity (≤0.7 normalized) to prevent jarring loudness; the synth never produces a click or peak above these bounds.
- Audio context state is observed; if it transitions to `suspended` (e.g., browser policy), the engine pauses scheduling and resumes on the next visibility-resume + user gesture.
- Output sample rate matches the audio context; we do not resample.

---

## 9. Accessibility surfaces

Accessibility is a designed surface, not a checklist (PRD). Each surface ships with v1.

### 9.1 Screen-reader narration

The narration is naturalist prose generated server-side from the same simulation state. The client surfaces it as a polite live-region (`aria-live="polite"`) under the canvas.

- Cadence: ~one prose update per 30–60s at idle, faster on user-initiated events (greeting, offer, settle).
- Voice: identical to the field notebook — lowercase, present-tense, specific, no announcement framing.
- Implementation: the simulation generates a `narration` field in the snapshot (and, for user-initiated events, a delta `narration_event`). The client routes these to a `<div aria-live="polite">` that screen readers announce.
- Priority bumps: greeting on session start, offer reactions, and settle are routed to the same live region with a small priority that ensures they're announced promptly without overwhelming the queue.
- Snapshots include a "narration cadence hint" so the client doesn't fire updates on every snapshot — only when the simulation provides new narration.

The `narration` text is a rolling description, not a list of state changes. Example sequence over 90 seconds:

> a small grey bird is perched on the front rail, calling softly. another sits further back with feathers fluffed.

> wren shifts on the back perch and tilts toward a passing leaf.

> pip greets — a soft three-note rise — and the morning warms a little.

A screen-reader user reading these one at a time gets a sense of the morning, not a stream of events.

### 9.2 Captions for procedural calls

Captions are short prose descriptions of what each call sounds like, generated server-side from the same call grammar. They appear as small text near the calling bird, fading in/out with the call. Captions and narration are independent: a user can have audio on and captions off (the default), audio on and captions on, or audio off and captions on.

- Captions display when audio is unavailable (default-on if WebAudio fails).
- Captions display when the user opts in explicitly.
- Caption text is generated from the actual procedural parameters: a "soft three-note rise" caption corresponds to a call of three notes ascending in pitch with low intensity.
- Captions are surfaces that can read against the aviary background; they pass WCAG AA contrast against both day and night palettes. The visual designer specifies the exact treatment (likely a soft-shadowed text on a near-transparent background pill).

### 9.3 Reduced-motion mode

§7.7 covers the rendering. The accessibility setting honors `prefers-reduced-motion` by default and lets the user override in either direction. The override is per-account in `prefs_json` and synced across devices.

### 9.4 Keyboard navigation

- `Tab` cycles through top-bar items in order: aviary (first), notebook, offer, settings, account.
- Focusing the aviary surface focuses the first bird (front perch, leftmost).
- Arrow keys move focus between birds (left/right cycles through birds by perch position).
- `Enter` toggles listen-in on the focused bird.
- `Esc` exits listen-in if active; otherwise removes focus from the aviary surface.
- The offer dropdown is reachable from the top bar via `Tab` and operable by `Enter`/arrow keys.
- Focus indicator: a soft, high-contrast outline drawn around the focused bird; persists across mood changes; readable against day, evening, and night palettes.

### 9.5 Settings surfaces

Accessibility settings live in a quiet drawer behind a top-bar icon. The drawer is matter-of-fact in tone: "Reduced motion: On/Off/Use system preference." "Captions: On/Off." "Audio: On/Off." "Master volume: …."

### 9.6 Contrast

All user copy passes WCAG AA. Contrast budget: 4.5:1 for body text, 3:1 for large text, against the aviary background's worst-case state (night). The visual designer specifies actual ratios per surface; the design system encodes them as tokens.

### 9.7 Aria roles and labels

- The aviary canvas has `role="img"` + `aria-label` set to a brief description ("aviary scene; <N> birds visible").
- Each bird is exposed as a focusable child element overlaid on the canvas with `role="button"` + `aria-label="<bird name>, <species>, <public mood>"`. The mood is the public mood enum, never numeric.
- Calls in flight are not separately announced (the narration covers them); captioned calls are announced via the captions live region only when captions are enabled.

### 9.8 Accessibility regression discipline

A CI a11y suite runs:

- axe-core scans on every page and drawer.
- Keyboard navigation flow tests via Playwright.
- Screen-reader narration cadence test (the live-region updates at the expected rate over a synthetic 5-minute session).
- Reduced-motion mode rendering test (no `requestAnimationFrame`-driven animations on bird sprites, only cross-fades).

A regression in any of these blocks deploy. The "we will fix accessibility in v1.1" failure mode is precluded because the suite runs at v1.

---

## 10. Performance budgets and observability

### 10.1 Budgets (PRD-named)

- **Initial JS bundle:** ≤2MB gzipped at first paint.
- **Time-to-first-bird:** <500ms p75 on mid-tier mobile / 4G; <800ms p95.
- **Idle motion:** 60fps on a 5-year-old mid-range laptop.
- **No memory growth over 30 minutes.**
- **Simulation tick latency:** p99 ≤ 5s (alarm threshold).

### 10.2 Bundle strategy

- Boot chunk (≤200KB gz): scene-graph, snapshot interpolation, audio bootstrapping, sprite loader for the visible species.
- Audio motif library: lazy-loaded after first paint; the first chunk includes only the species in this snapshot. Cached aggressively.
- Notebook drawer, settings, accessibility settings, account flow, visit-invite flow: route-split. Loaded on demand.
- Sprite assets: small SVGs / compact bitmaps; per-species generation where possible. Asset budget per species: <50KB.
- Tree-shake aggressive; no large UI library that can't be tree-shaken; bundle analyzer in CI.
- Polyfills: only what's needed for last-two-major-versions browsers. We deliberately do not maintain compatibility paths for very old browsers.

### 10.3 First-bird budget mechanics

- Inline the snapshot fetch initiation in the HTML (preload + critical hint).
- Snapshot endpoint is served from a CDN edge with a small, request-coalesced cache (under 500ms TTL); Postgres is hit only on cache miss.
- The snapshot is small (kilobytes; an AviarySnapshot for 4 birds plus 120s of upcoming calls is ~3-6KB).
- The boot chunk's first paint is the quiet field; the first bird draw fires as soon as the snapshot resolves; the audio bootstraps in parallel (deferred to user gesture if needed).

### 10.4 60fps idle motion

- Canvas2D with offscreen draw buffers per layer; only the bird layer redraws per frame at 60Hz, the rest at 1–10Hz.
- Sprite cache: pre-rendered sprite frames at the active viewport scale; no per-frame SVG-to-canvas re-rasterization.
- Day-night palette interpolation done on the GPU via CSS filters where feasible (background sky), Canvas2D tinting where not.
- Fast-path skip when the tab is hidden (rAF stopped, audio context paused).

### 10.5 No memory growth

- Audio: voice pool fixed at allocation; no per-call buffer creation.
- Sprites: cache size bounded; LRU eviction of unused sprite frames.
- Notebook: virtualized list — entries scrolled out of view release their DOM nodes.
- Snapshots: keep only the last two for interpolation; older snapshots are GC-eligible.
- A 30-minute soak test runs in CI on each release: opens an aviary, takes presence pings, scrolls the notebook, opens listen-in, reduces motion mode, returns to normal, exercises offer cooldowns. Heap snapshots at the start and end must be within a small ε; any growth fails the build.

### 10.6 Observability

Operational telemetry only — never per-bird, never per-account interaction state.

**What we measure:**

- Page load timings (TTFB, FCP, time-to-first-bird).
- Render frame timings (p50, p95, p99 frame duration; long-task counts).
- Audio context errors (failed creates, denied permissions).
- Simulation tick latencies (p50, p95, p99 per shard).
- API request counts and latencies, segmented by endpoint and HTTP status.
- WebSocket connection counts and disconnection causes.
- Email delivery rates and bounces (magic link, export, invite).
- Error rates by endpoint.
- Synthetic checks: a fleet of headless browsers running the aviary on a schedule from common geographies, validating first-bird budget and basic rendering.

**What we deliberately do not measure:**

- Per-bird interaction events as analytics dimensions.
- Per-account session duration (individual sessions; only anonymized session-duration histograms are aggregated, with no account dimension).
- Personality vector values across the population.
- Drift rates per account.
- Visit frequency surfaces of any kind.
- Anything that would let us answer "which user is most engaged."

The boundary is enforced architecturally:

- The aggregate telemetry pipeline does not have credentials to read the simulation database.
- Telemetry events are emitted through an in-process emitter that strips per-account dimensions at source. Emitting an event with `account_id` is a runtime error; CI lints it.
- The privacy claim ("we don't aggregate per-bird data") is implemented as an architectural property, not a policy. There is no place in the system that has both per-bird data and aggregate-pipeline access.

**Alarms:**

- Simulation tick p99 > 5s for 5 consecutive minutes.
- First-bird p75 > 500ms for 10 consecutive minutes (synthetic).
- Magic-link delivery success < 99% over 1 hour.
- Snapshot endpoint p99 > 250ms for 5 consecutive minutes.

### 10.7 Browser support

Last two major versions of Chrome, Safari, Firefox, Edge. Older browsers are caught at boot and shown a matter-of-fact unsupported-browser surface. The matrix is encoded in build config; we test on the named browsers in CI per release.

---

## 11. Rollout

### 11.1 v1 scope freeze

The features listed in §1 are the v1 surface. We do not ship a partial v1 that omits accessibility, the field notebook, or the visit feature — these are v1 substantive content. Reduced-motion ships day-one.

### 11.2 Phases

The product ships in three phases, internal-only until phase 3.

- **Phase α (internal, ~4 weeks).** Two-bird aviary with no notebook, no visits, no accessibility surfaces beyond keyboard nav. Goal: prove the simulation tick / snapshot / render loop end-to-end; calibrate drift constants against the synthetic harness; validate the chorus mechanic; close the bundle/first-bird budgets.
- **Phase β (closed beta, ~8 weeks).** Full v1 surface, invitation-only beta with ~200 accounts. Goal: real-world drift calibration over a 3-week window; sync correctness across devices; a11y surfaces validated against real users (including screen-reader testers); audio uncanniness stress-tested against a large procedural call sample.
- **Phase 1.0 (public).** Open sign-ups. Two-bird-cap maintained at start; the aviary-age-paced offers begin appearing at the named intervals.

### 11.3 Bird ramp

Per the PRD, the cap is 7. The ramp is a function of aviary age, not engagement:

- Day 0: 2 birds (starter pair).
- Day ~3: third-bird offer appears in the user's flow (a quiet notebook-style mention; the user accepts to add the bird).
- Day ~10: fourth-bird offer.
- Day ~28: fifth-bird offer.
- Day ~80: sixth-bird offer.
- Day ~200: seventh-bird offer (cap reached).

The pacing matches the PRD's "rhythm of a relationship deepening." The intervals are super-linear so growth slows at the cap. The user can decline an offer; the next offer is gated by the same age threshold (i.e., the user does not "miss" their fifth bird if they decline at day 28; the offer reappears later).

### 11.4 Day-one instrumentation

We instrument the operational metrics in §10.6 from day one. Specifically:

- Simulation tick latency, segmented by shard.
- First-bird p75/p95.
- Audio context error rate.
- Magic-link delivery success.
- Synthetic checks from 4 geographies.
- Drift calibration harness output (CI artifact, weekly review).
- A11y CI suite (regressions block deploy).

We deliberately do not instrument:

- Per-bird drift values across the population.
- Per-account interaction frequencies.
- Per-account session durations as an analytics dimension.

### 11.5 Feature flagging

A feature flag service for ops: ability to disable WebSocket push (fall back to polling), cap the simulation tick rate during incident, disable the visit feature account-wide, disable the email export. We do not feature-flag the affective surface (no "show streak counter for 1% of users" experiments — the PRD's gamification refusal forecloses this entirely).

### 11.6 Migrations

Schema migrations follow a "add column, backfill, switch reads, drop old" discipline. Personality vector migrations preserve the existing values — never recompute from the event log, never reset (PRD: identity continuity). The `personality.version` field allows additive trait expansion in future without rebuilding state.

---

## 12. Risks

### 12.1 Drift miscalibration (high)

A drift function tuned wrong — too fast, too slow, or asymmetric in a way that violates monotonicity — corrupts the central product promise. The risk is silent: no test fails, but birds change visibly between sessions or never seem to change at all.

**Mitigations:**

- Drift calibration harness in CI (§5.5). Targets are checked-in numerical bands; harness output drift alarms.
- The asymmetry rule is enforced at code level (`clamp(delta, 0, max_step)`), not at calibration. A negative delta is impossible to commit.
- Online tuning of constants is gated behind a guardrail that prevents crossing the monotonicity invariant.
- Phase β collects qualitative feedback from a small set of testers about whether their birds "feel like they're changing" at week 1 and week 3.
- A "personality drift dashboard" exists internally for the simulation team only (admin-authenticated, never exposed to users), surfacing aggregate drift trajectories from the calibration harness.

### 12.2 Sync correctness (high)

A bug in the simulation worker's event-log consumption — out-of-order processing, double-count, unconsumed events — would produce wrong personality state. Multi-device divergence (the PRD's named failure mode) would only show up as users feeling "my birds aren't drifting like they should be."

**Mitigations:**

- The event log is append-only and consumed in `id` order with idempotent `consumed_by_tick_at` marking.
- Account-level advisory lock prevents concurrent ticks per account.
- Tick replay test: re-running a tick on the same input produces the same output. Asserted in CI.
- Multi-device sync test: a synthetic harness simulates two devices interacting concurrently and asserts the personality vector is the additive composition of their effects, not the result of either's overwriting the other.
- The "no personality writes from the client" property is enforced at DB role level.

### 12.3 Audio uncanniness (medium-high)

The PRD names this directly: looped audio is the audible signature of dead software, and the chorus mechanic depends on real procedural variation. A subtle bug in the synth — a motif that sounds the same twice, a chorus that produces phase artifacts — breaks the spell across the entire product.

**Mitigations:**

- Synth variation test: 100 calls of the same species in the same mood produce a measurable diversity (in pitch contour, tempo, ornament insertion). If diversity is below threshold, build fails.
- Chorus quality test: synthetic two-bird chorus events are spectrally analyzed; presence of phase-cancellation artifacts above a threshold fails the build.
- Manual listening review at each release by the audio designer.
- A user-reportable "this bird sounded canned" is logged (without per-bird state in telemetry) so we can investigate aggregate trends.

### 12.4 Accessibility regression (medium)

A small change to the rendering pipeline that breaks reduced-motion, narration cadence, or keyboard navigation is the highest-likelihood regression on this product. The mitigation is design-level: accessibility surfaces are not afterthoughts.

**Mitigations:**

- A11y CI suite (§9.8) blocks deploy on regression.
- Reduced-motion mode is a separate renderer with its own tests.
- Narration is generated by the simulation, not the client — a renderer regression cannot break narration.
- Manual screen-reader testing in phase β with at least 5 testers, with a documented pass list for v1.0.

### 12.5 First-frame fidelity (medium)

The "aviary appears already in motion" property is fragile under slow networks, cold caches, and bundle bloat. A regression here reads as the product breaking, not as performance.

**Mitigations:**

- First-bird budget alarm at p75 > 500ms for 10 consecutive minutes.
- Bundle budget alarm in CI; over-budget fails the build.
- Synthetic checks from 4 geographies.
- Quiet-field loading state validated in design review (no spinner under any condition).

### 12.6 Privacy boundary leakage (high)

The privacy claim ("we don't aggregate per-bird data") is the kind of thing that's easy to honor at design time and easy to leak under the pressure of "just one analytics dashboard." A leaked boundary is non-recoverable trust damage.

**Mitigations:**

- Telemetry pipeline does not have credentials to read the simulation database. (Architectural; not a policy.)
- Telemetry emitter strips `account_id` and per-bird fields at source; emitting them is a runtime error.
- Quarterly privacy audit: an engineer outside the simulation team verifies the architectural property.
- The "personality vector never exposed" rule is unit-tested at the snapshot serializer.

### 12.7 Scope creep into gamification (high — cultural)

The PRD names this as the strongest temptation. A "harmless" streak counter, a notebook entry that observes user behavior, a "you visited every day this week" surface, a public discovery feed — any of these reverses the product's center of gravity.

**Mitigations:**

- The non-goals list (§1; PRD `non_goals.md`) is binding and explicit.
- Code review checklist includes: "Does this surface count, rank, or compare any user behavior?" If yes, reject.
- Notebook prose-template generation is constrained to bird-and-aviary observations; user-behavior observations are not in the template family.
- API surface design forecloses the "just expose this aggregate" path: the visit log is per-invite (not per-frequency), the notebook is per-bird (not per-user-behavior), the export is per-account (not per-tier).
- The non-goals section is part of the engineering onboarding doc, framed as the rules that must survive every reasonable-looking pitch to relax them.

### 12.8 Simulation cost at scale (medium)

The per-account tick is cheap individually; at N=1M accounts the live-queue cadence (60s) is 16,000 ticks/second. The dormant queue at 15 minutes is much lighter. The simulation worker must scale horizontally.

**Mitigations:**

- Per-account tick cost target: <10ms p95. The budget is set at the calibration harness; over-budget fails the build.
- Sharding by account UUID; horizontal scaling of simulation workers.
- The catch-up tick design absorbs dormant accounts cheaply.
- Postgres scaling: the canonical state has read replicas; the event log uses partitioned tables by month.

### 12.9 Visitor-as-attack-surface (medium)

A malicious visitor could try to impersonate the host's session, submit events under the host's account, or exfiltrate the host's notebook. The visitor surface must be hardened.

**Mitigations:**

- Visit-mode session tokens are scoped: the gateway rejects writes from a visit-token session.
- Visitor pings are independently rejected at event ingest.
- Visitor-mode snapshots strip notebook contents.
- Invite tokens are single-use-per-session and expire on revoke.
- Rate-limiting on visitor endpoints.

### 12.10 Email delivery as a single point of failure (low-medium)

Magic-link sign-in depends on email delivery. A provider outage blocks all sign-ins.

**Mitigations:**

- Multi-provider failover (SES primary, fallback provider).
- Synthetic email-delivery checks from CI environment.
- Magic-link delivery success rate alarm at <99% over 1 hour.
- A "having trouble signing in?" matter-of-fact recovery surface that explains the email may be delayed and offers re-request.

---

## 13. Open implementation details (deferred to specs)

A small set of items are intentionally out of scope for this plan and live in their own specs:

- The visual designer's palette and contrast specs (referenced by §7 and §9).
- The design-system spec for top-bar chrome, drawer surfaces, focus indicators.
- The species-pool visual designs (silhouettes, default plumage) and motif libraries.
- The exact prose templates for narration and the field notebook (voice review by the writer; not encoded in this plan).
- The exact email copy for magic links, exports, invitations, and deletion confirmations (matter-of-fact voice; copy review in onboarding).

These are referenced where they touch the architecture but are not specified here.

---

## 14. Summary of load-bearing rules (pinned)

A flat list of the rules that this plan is built around. Any change in the codebase that violates one of these is a defect.

1. The server is the only writer of personality state.
2. Personality drift is monotonic toward expressive (no negative drift on neglect).
3. Calls are procedural; no recorded audio fallback; silence-with-captions is the documented fallback.
4. Presence is the conjunction of `visibilityState === 'visible'` AND `document.hasFocus()` AND a pointer/key event in the last 4 minutes.
5. Per-bird interaction state is never aggregated for any cross-account purpose.
6. Personality vector values are never exposed to the user inside the product (the JSON export is the only exception, and only in the user's own data context).
7. The aviary appears with motion already in progress; no spinner, no "Welcome back," no entry animation.
8. Notice, never announce: no toasts, no visit notifications by default, no engagement surfaces.
9. No gamification of any kind: no streaks, achievements, badges, levels, calendars, public surfaces, or any aggregate visit-frequency widget.
10. Synthetic UUIDs are the only account identifier; email lives encrypted on one row.
11. Bird identity is stable across renames, sync events, and migrations.
12. Accessibility is a designed surface, not a stripped fallback; reduced-motion is its own renderer.
13. The visitor session is render-only; visitor presence does not drift the host's birds.
14. Notebook is read-only and observes the aviary, never the user's behavior.
15. Bundle budget 2MB gz; first-bird p75 <500ms; 60fps idle on a 5-year-old laptop; no memory growth over 30 minutes.

These rules are the substance of the product. The code, the schema, the API, and the build pipeline are organized to make them invariants, not aspirations.
