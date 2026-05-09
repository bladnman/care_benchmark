# Pocket Aviary — v1 Implementation Plan

This plan interprets the PRD into an executable build. A frontier engineering team should be able to ship v1 from this document without further clarification. Where the PRD left a defensible call open, I make the call here and mark it explicitly under "Open calls" at the end of each major section so the choice is visible to reviewers and not buried in code.

The plan is deliberately opinionated. The product's value is largely affective — felt-aliveness, naturalist voice, restraint — and "obvious" engineering shortcuts (recorded audio fallbacks, last-write-wins sync, ARIA-label automation, an entry spinner) are precisely the moves that destroy the product's center. The architectural rules below exist to make those shortcuts unreachable, not just discouraged.

---

## 1. Scope

### In scope for v1

- Single-user accounts, magic-link sign-in (15-minute link expiry, single-use), per-device session tokens with a revocation surface.
- Single canonical aviary per account, multi-device read-from-server sync.
- Two starter birds at adoption (system-chosen species; user-named); cap at seven; new species offered at age-tied milestones.
- Six-species pool, each with a visual silhouette, default plumage palette, and motif library for procedural calls.
- Hidden personality vector per bird (boldness, social warmth, vocal frequency, plumage saturation, curiosity); slow drift driven primarily by presence and listen-in; monotonic toward expressive.
- Mood layer: small enum (wary, content, curious, drowsy, alert), persisted across sessions, transitions on a daily-ish cadence shaped by recent interactions, time of day, weather, and personality.
- Server-side simulation tick at slow cadence (~60s); single writer of personality and mood state.
- Single horizontal scene rendered in the user's browser tab; three perch zones; day/night anchored to user's local time; rare ambient weather; ambient leaf and feather drift.
- Top-bar chrome with account/settings, accessibility settings, field notebook, offer affordance, settle gesture; fades on cursor stillness.
- Listen-in interaction (slow gain ramp, others quiet but never silent).
- Offer interaction (seed, song fragment, still pool) reachable from top bar with per-bird cooldown.
- Settle gesture with 5-second click-anywhere undo.
- Field notebook: auto-generated, sparse, naturalist-prose, read-only, scrollable indefinitely.
- Procedural client-side WebAudio synthesis of calls; chorus mixing; listen-in mix decay.
- Reduced-motion mode (designed surface, not a stripped fallback).
- Screen-reader narration in naturalist prose on a slow cadence with priority bumps for user-initiated events.
- Call captioning generated from the same call grammar.
- Full keyboard navigation; visible focus indicators on light/dark scenes; WCAG AA contrast on chrome and overlays.
- Visits: per-invite email-bound opt-in, read-only ambient view; revocable; visit log; 30-day invite expiry; off by default; visit-notify opt-in toggle off by default.
- Account export (on-demand JSON, emailed download link); soft-delete with 30-day recovery; hard delete after 30 days.
- Aggregate-only telemetry; per-bird simulation data never reaches analytics or training pipelines.
- Browser support: last two major versions of Chrome, Safari, Firefox, Edge. Older browsers see a matter-of-fact unsupported-browser surface.

### Explicitly out of scope (v1)

- Native iOS/Android apps. (Architecture and protocol design assume browser-only; we do not pre-shape the data model for a native client.)
- Any gamification surface. No achievements, streaks, levels, scores, badges, "birds adopted: N", green-dot calendars, XP, ranks, tiers, or "you've been here every day this week" surfaces — including disguises like quiet calendars in settings or exportable visit logs.
- Tamagotchi-style mechanics. Birds do not die, hunger, decay, or visibly suffer from neglect. Drift is monotonic toward expressive.
- Social-network surfaces. No profiles, follows, public feed, comments, leaderboards, "explore other aviaries," friend-of-friend chains, mutual visits, or visit chat.
- Push or email notifications about the aviary. The product does not pull the user back; the user comes when they come.
- Payments, multi-aviary accounts, shared aviaries, customizable scenes, scenery extensions.
- Recorded-audio fallback path. (Hard rule.)
- Streak counters of any kind. (Hard rule, including disguises.)

### Non-goals as architectural absences

The non-goals are honored in code, not just in policy. Specifically:

- Visit-frequency is never computed at the account level; the schema has no `visits_count`, no `daily_login_streak`, no `last_visited_streak_start`. The metric does not exist, so it cannot be exposed.
- Per-bird interaction events do not flow into any analytics warehouse table. Telemetry pipeline is on a separate network from the simulation DB; IAM denies analytics roles read access to simulation tables.
- Personality vectors have no client-write code path. The DB grant for personality tables is on the simulation worker only.

---

## 2. Architecture

### Service shape

Six logical services. They may collapse into fewer deployments at v1 — but the seams below are real and must be preserved at the data-flow level even when packaging.

1. **Edge / Web** — static client bundle served from CDN; thin edge HTML renderer that injects an initial state hint for signed-in users to support <500ms first-bird.
2. **Auth service** — magic-link issuance and verification, session token lifecycle, email change verification.
3. **Aviary API** — HTTPS/JSON endpoints for snapshot reads, event-log writes, settings, visits, exports, deletion.
4. **Simulation worker pool** — runs per-account ticks; sole writer of personality, mood, perch, schedule, and notebook entries.
5. **Mailer** — transactional email for magic links, visit invites, export download links, email-change verification.
6. **Telemetry pipeline** — aggregate-only operational metrics, on a network that has no read path to the simulation DB.

### Data stores

- **Primary state DB**: PostgreSQL (or equivalent ACID relational). Holds accounts, birds, personality vectors, moods, perch state, notebook entries, visit invites, sessions. Row-level security where supported; account_id partition key.
- **Event log**: an append-only Postgres table partitioned by account_id and ingest_seq. (Choice over Kafka at v1: simpler operations, single-writer-per-account is naturally enforced by an advisory lock on account_id; event volumes are low. If volumes grow past a tier, swap the log behind a read-through interface; the simulation worker only consumes a tail-cursor.)
- **Object storage** for export downloads (signed URLs, short TTL).
- **Cache**: a small Redis (or equivalent) for: snapshot hot path read-through cache (per account, short TTL), magic-link nonce store, rate-limit counters.
- **Analytics warehouse**: separate, isolated. Receives only aggregate metric streams.

### Client/server split (load-bearing)

- **Server is the only writer** of personality vectors, moods, perch state, and call schedules. No code path on the client mutates these.
- **Server is the only writer** of notebook entries. Notebook prose is generated server-side from a curated template grammar pinned to verified facts.
- **Client emits events** to an append-only log (presence pings, listen-in start/end, offers, settle, settle undo, return focus). It never sends absolute state.
- **Client renders from snapshots** (skeletal canonical state) and fills in micro-motion procedurally. The client owns: animation interpolation, audio synthesis, per-session listen-in mix, leaf/feather ornaments.
- **Snapshots do not contain numeric personality vector values.** They contain mood, perch, pose, scheduled call seeds, and shaping parameters needed to render — never the raw personality numbers. (Hard rule; protects "personality vector is never exposed numerically.")

### Render pipeline boundary

The render pipeline boundary is the snapshot. Everything below the snapshot is server-canonical. Everything above the snapshot is client-rendered. Specifically:

- Bird positions and pose categories are server-decided per tick; the client interpolates between them.
- Call schedule (next call seed, timestamp, motif params) is server-emitted; the client realizes the audio.
- Day/night state, weather state, weather decay schedule are server-emitted.
- Leaf and feather drift, idle micro-jitter, parallax tilt, transition easings — purely client-side ornaments. They do not reach the server.

This split keeps the simulation cheap (tick is decisions, not pixels) and keeps the client free to render at 60fps without round-trips.

### Open calls

- **DB**: Postgres. Reasoning: ACID semantics, advisory locks for tick single-writer, mature tooling; can carry the v1 audience comfortably. Alternative (DynamoDB / Spanner) explicitly rejected for v1 to avoid eventual-consistency debugging in a domain where additive correctness matters.
- **Edge HTML**: a thin Edge function (e.g., Cloudflare Worker, Vercel Edge, Lambda@Edge) renders the HTML with an inline initial snapshot for signed-in users. Sticky session cookie identifies the account at the edge.
- **Worker scheduling**: a per-account tick scheduler that enqueues jobs at ~60s cadence with per-account jitter (to spread load and avoid synchronized thundering herds). At small scale a single cron + worker pool suffices; at scale, use a queue with per-account ordering (e.g., partitioned consumer groups).

---

## 3. Data model

All identifiers below are UUIDs unless noted. All timestamps are stored UTC; user-facing time is rendered against the user's local timezone.

### `account`

- `account_id` (UUID, PK) — synthetic, primary identifier across every system.
- `email_encrypted` (bytes) — encrypted at rest with a per-table key, decrypted only by the auth and mailer services. Never used as an identifier outside this table.
- `email_verified_at` (timestamp, nullable).
- `created_at` (timestamp).
- `last_seen_at` (timestamp).
- `tz_iana` (text) — last-known IANA timezone; updated on snapshot pull.
- `soft_deleted_at` (timestamp, nullable). If non-null, account is hard-deleted at `soft_deleted_at + 30d`.
- `prefs_jsonb` — accessibility prefs (reduced motion override, captions on/off, audio mute), visit-notify toggle (default false).

### `bird`

- `bird_id` (UUID, PK) — stable for life of the bird.
- `account_id` (UUID, FK).
- `species_id` (text) — references species pool record.
- `name` (text) — user-assigned; renameable.
- `spawned_at` (timestamp).
- `personality` (jsonb) — `{ boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity }`, scalars in [0,1].
- `personality_filter_state` (jsonb) — internal low-pass filter state per trait (running averages, window decay accumulators).
- `last_tick_seq` (bigint) — last tick that updated this bird.

Indexed by `account_id` (partition key); no cross-account index ever exists.

### `mood`

- `bird_id` (UUID, PK, FK).
- `mood` (enum: wary | content | curious | drowsy | alert).
- `mood_entered_at` (timestamp).
- `mood_modifiers` (jsonb) — list of `{source, decay_at, vector}` (e.g., from accepted offer, from rain, from a peer's alarm call); used by the next tick.

### `perch_state`

- `bird_id` (UUID, PK, FK).
- `perch_zone` (enum: front | middle | back).
- `perch_slot` (smallint, within zone).
- `pose` (enum: preening | scanning | head_tilting | resting | calling | flying).
- `pose_started_at`, `pose_ends_at` (timestamps).
- `next_call_seed_at` (timestamp).
- `next_call_seed_value` (bigint) — deterministic seed for upcoming call.

### `interaction_event` (append-only)

- `event_id` (UUID, PK).
- `account_id` (UUID, partition key, FK).
- `bird_id` (UUID, nullable, FK).
- `client_session_id` (UUID).
- `type` (enum: presence_ping | listen_in_start | listen_in_end | offer | settle | settle_undo | return_focus | adoption_named | rename).
- `payload` (jsonb) — event-specific (offer kind, target bird, listen-in duration, etc.).
- `client_occurred_at` (timestamp).
- `server_received_at` (timestamp).
- `ingest_seq` (bigint) — server-assigned sequence per `account_id`.
- `consumed_at` (timestamp, nullable) — set by the simulation worker when a tick has applied this event.

### `notebook_entry`

- `entry_id` (UUID, PK).
- `account_id` (UUID, FK).
- `occurred_at` (timestamp).
- `body_text` (text).
- `bird_id_refs` (uuid[]).
- `generation_source` (jsonb) — record of the facts that drove the prose (which template, which fact set), for audit and tuning.

### `visit_invite`

- `invite_id` (UUID, PK).
- `host_account_id` (UUID, FK).
- `visitor_email_encrypted` (bytes).
- `visitor_email_verified_hash` (text) — keyed hash for lookup at link consumption.
- `token_hash` (text) — hash of the invite token; the raw token is never stored.
- `status` (enum: pending | active | revoked | expired | consumed).
- `issued_at`, `expires_at` (`issued_at + 30d`), `last_used_at`.
- `revoked_at` (timestamp, nullable).

### `visit_session`

- `session_id` (UUID, PK).
- `invite_id` (UUID, FK).
- `started_at`, `last_pull_at`, `ended_at`.
- `visitor_user_agent_hash` (text) — for visit-log rendering only; not used for tracking.

### `magic_link`

- `link_id` (UUID, PK).
- `account_id` (UUID, FK, nullable until first verify on a new email).
- `token_hash` (text).
- `issued_at`, `expires_at` (`issued_at + 15min`), `used_at` (nullable).
- `purpose` (enum: sign_in | email_change_verify | export_download).

### `device_session`

- `session_token_id` (UUID, PK).
- `account_id` (UUID, FK).
- `device_label` (text) — derived from user agent for the revocation UI.
- `created_at`, `last_used_at`, `revoked_at`.

### Snapshot (transient; not persisted)

What the client receives:

```jsonc
{
  "snapshot_seq": 1234,
  "server_now": "2026-05-08T14:31:02Z",
  "tz_iana": "America/Los_Angeles",
  "daylight_state": { "phase": "morning", "phase_progress": 0.42 },
  "weather": { "kind": "clear" },
  "birds": [
    {
      "bird_id": "...",
      "name": "pip",
      "species": "warbler",
      "mood": "content",
      "perch": { "zone": "front", "slot": 1 },
      "pose": { "kind": "preening", "started_at": "...", "ends_at": "..." },
      "render_params": {
        "plumage_saturation_render": 0.78,   // a render value, not the raw trait
        "scan_speed_render": 0.6,            // mood/personality-shaped
        "tilt_bias_render": 0.3,
        "boldness_render": 0.7
      },
      "next_call": { "scheduled_at": "...", "seed": 9281736, "motif_set": "warbler_morning" }
    }
  ]
}
```

`render_params` are derived shaping values needed to render mood-flavored idle motion. They are not the personality vector. The client cannot reconstruct the personality vector from them. They look superficially like the same scalars; in fact they are functions of mood × personality × time-of-day, computed server-side, deliberately squashed and noise-padded so a curious user inspecting network traffic cannot read the personality vector off the wire.

### Open calls

- **Personality vector range**: scalars in [0,1] with a base seed at adoption sampled per-trait per-bird from a calibrated distribution per species. Specific seeds and ranges live in the simulation service config.
- **Mood enum**: wary | content | curious | drowsy | alert at v1. Locked. Adding moods later requires a calibration round.
- **Notebook source recording**: every entry stores its source facts, even though they're never shown to the user, so a future audit can verify the prose is grounded.

---

## 4. API surface

REST over HTTPS, JSON bodies. All responses are typed; errors use a small consistent envelope. Endpoints below assume an authenticated session cookie unless noted; the visit endpoints carry an explicit token.

Voice rule: every error returned by these endpoints is rendered (where shown to the user) in the matter-of-fact register, not the naturalist register. The voice split is honored at the surface, but the API itself returns structured codes; client-side voice mapping ensures a system-error never bleeds into naturalist UI.

### Auth

- `POST /auth/request_link { email }` — issues magic link. Always returns 200 with a uniform body whether or not the email is registered (avoids enumeration). Rate-limited per email and per IP.
- `GET /auth/verify?token=` — sets device session cookie if valid; redirects to the aviary; consumes the link.
- `POST /auth/sign_out` — revokes current device session.
- `GET /account/sessions` — list device sessions.
- `POST /account/sessions/:id/revoke` — revoke a session.
- `POST /account/email/change_request { new_email }` — issues verification email to new address.
- `POST /account/email/verify { token }` — switches the canonical email after verification.

### Aviary state

- `GET /aviary/snapshot?since=<seq>` — returns the current snapshot. If `since` is provided and recent, server may return a delta payload; otherwise a full snapshot. Response includes `Cache-Control: private, no-store`.
- `POST /aviary/events` — body: array of events. Server assigns `ingest_seq` and persists. 202 Accepted on success.
- `GET /aviary/birds/:id` — name, species, current mood, perch zone, last seen pose. No personality numbers.
- `POST /aviary/birds/:id/rename { name }` — updates name.
- `GET /aviary/notebook?cursor=<entry_id>&limit=<n>` — paginated, newest first.

### Settings

- `GET /account/settings`
- `POST /account/settings` — accessibility prefs, visit-notify toggle, audio mute, captions opt-in.
- `POST /account/timezone { tz_iana }` — explicit timezone update, called by the client when it detects a change.

### Export & deletion

- `POST /account/export` — queues an export; mailer sends a one-time, short-TTL signed download link.
- `POST /account/delete` — sets `soft_deleted_at = now`. Account is read-only-ish (sign-in works to recover) for 30 days.
- `POST /account/recover` — clears `soft_deleted_at`.

### Visits

- `POST /visits/invites { visitor_email }` — host issues invite; mailer sends visitor link.
- `GET /visits/invites` — list of host's invites.
- `POST /visits/invites/:id/revoke` — revokes; visitor's next pull returns visit-no-longer-available.
- `GET /visits/log` — host's visit history.
- `GET /visits/snapshot?token=<visit_token>` — read-only snapshot for an active visit. Identical schema to `/aviary/snapshot`. Visitor sessions are throttled and never accept events; any POST to events with a visit token is rejected.

### Operational

- `GET /healthz`, `GET /readyz` — service health.
- `GET /version` — build info.

### API rules

- All endpoints are scoped to the requester's `account_id`; cross-account reads are not possible because no endpoint accepts another account_id parameter.
- No analytics endpoints exist. There is no "ping us with telemetry" client API; aggregate telemetry comes from server logs and the synthetic perf fleet, not from the user's browser sending identifiable events.
- The events endpoint accepts only the named event types; unknown types are rejected. A new event type is a deliberate change with a calibration review.

### Open calls

- **Snapshot delta vs full**: the wire format supports both. The server decides per-request based on whether the client's `since` is recent enough to express as a delta; full snapshot at v1 default.
- **Long-poll vs fetch-on-event**: v1 is fetch-on-event (visibility change, long frame gap, periodic keepalive). Server-Sent Events deferred to v1.x — the client read pattern is bursty and the cost of opening a long-lived connection per tab is real on mobile.

---

## 5. Simulation engine design

The simulation tick is the architectural heart of the product. Its correctness properties are not "best effort"; they are tested.

### Tick cadence and shape

- Each account is ticked at a target cadence of ~60s with up to ±15s of jitter (to spread load and avoid synchronized waves). Calibrate cadence in alpha.
- A scheduler service enqueues per-account tick jobs. Each job acquires an advisory lock keyed on `account_id` (Postgres `pg_advisory_xact_lock(account_id_hash)`); conflicting ticks for the same account block until release. This guarantees single-writer-per-account.
- The job consumes events from the event log in `ingest_seq` order from the cursor (the tick stores its consumed cursor on a per-account row). The cursor advances atomically with the canonical-state write.

### Drift function

Personality drift is a low-pass filter with one-sided clamping per trait.

For trait `T_i` on bird `b`:

```
delta_i = sum over events e in window:
    presence_weight(e, b)             // how much "presence" this event represents
  * trait_weight_i(e, b, mood, personality)  // how much this event nudges trait i

T_i_new = max(T_i, T_i + delta_i * smoothing_i)
```

Notes:

- `presence_weight(e, b)` is dominated by presence_ping events (deduplicated to distinct seconds across overlapping sessions). Listen-in start/end produces a strong contribution to `social_warmth` and `vocal_frequency` for the focused bird. Offers contribute to `curiosity` and a small amount to `boldness`.
- `trait_weight_i(...)` is a sparse mapping from event types to traits. We start from a calibrated table and tune it during alpha based on the calibration harness output.
- `smoothing_i` is a small per-trait scalar tuned so that a "typical regular user" (defined via a synthetic event log: ~15 minutes of presence per day, occasional listen-in, occasional offer) yields measurable change in instruments at ~7 days and visible change to a user at ~21 days.
- `max(T_i, ...)` enforces monotonic-toward-expressive at the lowest level. This is asserted in unit tests for any input across the parameter space — there is no event log that drives any trait below its prior value.
- Plumage saturation drifts even more slowly than the others, and is the visible-most signal of long-term drift. Calibration target: visible at ~21 days.

### Mood transitions

Each tick reconsiders mood for each bird with probabilities shaped by:

- Personality (boldness reduces wary entry; high curiosity raises curious; high social warmth raises content on peer-call response).
- Recent events in the consumed window (offer accepted -> bias toward content; settle gesture for the user's current session -> bias toward drowsy at end of presence).
- Time of day in the user's local tz (drowsy bias near dusk; alert bias early morning; wary bias slight at deep night).
- Weather (rain dampens vocal scheduling and biases content/drowsy; wind biases bold birds toward alert and shy birds toward wary).
- Peer states (a peer in `wary` propagates a small wary modifier with distance and personality decay).

Mood transitions are explicit state-machine moves recorded with `mood_entered_at` and a list of decaying modifiers; they are persisted on the bird row, so they survive across sessions and multi-device sync without recomputation.

### Call grammar runtime

The simulation is responsible for *scheduling* calls; it never synthesizes audio.

For each bird, the tick may schedule one or more upcoming call events:

- A `next_call_seed_at` is assigned based on `vocal_frequency`, mood, time-of-day, and ambient state (chorus opportunities raise nearby birds' chances of joining).
- A `next_call_seed_value` is assigned (deterministic per call). This seed makes the realized audio identical across all clients of the same account watching the same time slice (multi-device coherence; visit coherence).
- The tick may also emit "responds-to" links between schedules (bird A's call cues bird B's response with a `responds_to: A_call_id`).

Snapshots include the upcoming call schedule for the next short horizon (e.g., next 60s). The client uses these to schedule its WebAudio render-graph events.

### Bird-to-bird interaction

Bird-to-bird interactions are computed in the tick after all per-bird mood updates:

- Alarm propagation: a bird transitioning to `wary` or `alert` emits a transient modifier that influences peers' next-tick mood transitions.
- Chorus formation: when two or more high-vocal-frequency birds in `content` or `curious` mood are scheduled to call within a small window, the tick widens that window slightly to encourage temporal overlap, and emits a `chorus_event` flag in the snapshot for the rendering layer to honor (subtle attenuation to make chorus comprehensible).
- Call-and-response: shy/social personality combinations produce response chains.

### Ambient events

- Day/night phase computed per-account from `tz_iana`. Phases: dawn, morning, midday, afternoon, evening, dusk, night. Phase transitions are smooth (not snap); the snapshot carries `phase` and `phase_progress` for client-side palette interpolation.
- Weather sampled at low Poisson rate per aviary (target: ~3 events/week at v1). Each event has duration and intensity; the tick records `weather` state and decays it over time.
- Leaf and feather drift: pure client ornaments. The tick does not schedule them. (Per `aviary_layout.md`.)

### Notebook entry generation

Notebook prose is generated server-side by a small template grammar. Generation runs as a sub-step of the tick.

Sparsity rules:

- Hard cap: at most one entry per account per day on average. Implementation: a token-bucket per account; an entry consumes a token; tokens refill at ~1/day. Bursting allowed for genuinely noteworthy days (e.g., the first chorus event ever).
- Eligibility filter: an entry is only considered when the consumed event window or the canonical state diff includes a "noteworthy pattern." Examples: first-greeting change ("pip greeted before wren today"), drift threshold crossed (no number named — the prose is grounded in a fact like "wren has been calling more often this week"), unusual chorus, weather-event reflection, a long quiet stretch.
- Voice grounding: prose is generated from a curated template grammar reviewed by a writer-of-record. Templates are pinned to fact slots; facts are pulled from the canonical state diff, never invented.
- No LLM at runtime in v1. The temptation to generate prose with a large model is real and rejected: the affective register of the notebook is the product's voice, and a runtime model introduces failure modes ("Pip's vocal frequency increased by 0.03"; "Achievement: First Chorus"; gamification leakage; hallucinated facts). Templates are slower to design but durable.
- Forbidden patterns are blocklisted in templates: any phrasing that implies streak, achievement, unlock, count, "you've been here," "your friend visited," or addresses the user as "you" or "your" except in matter-of-fact system surfaces.

Each entry is stored with its `generation_source` (which facts, which template); a quarterly audit samples entries to verify grounding and voice.

### Tick correctness invariants

Tested in CI:

1. **Single writer**: only the simulation worker writes the personality, mood, perch, and notebook tables. Enforced by Postgres role grants and asserted in a structural test that scans the codebase for writes outside the worker.
2. **Single-writer-per-account**: re-entrancy is impossible due to the advisory lock.
3. **Idempotent on event range**: replaying a tick over the same event range from the same prior state yields the same canonical state. Tested by capturing event ranges and snapshotting before/after.
4. **Monotonic toward expressive**: for any input, no tick decreases any personality trait. Tested via fuzz: 10,000 random event-log sequences, assert no trait ever decreases.
5. **Deterministic call seeds**: given the same canonical state and same tick wall-clock, the call schedule is identical. Used for multi-device coherence.
6. **No cross-account reads**: tick has only `account_id`-scoped queries; an integration test injects cross-account state and asserts the tick does not read it.
7. **Latency**: tick p99 < 5s under realistic load; alarms above.

### Open calls

- **Scheduler**: v1 uses a small distributed cron + per-account work queue. Spread accounts across a partitioned consumer pool keyed on `account_id` to preserve per-account ordering.
- **Notebook writer-of-record**: a named role on the team; templates land via PR review by that role. Set up in alpha.
- **LLM prose**: deferred. Could be reconsidered for v2 with a strict grounding harness; v1 is templates only.

---

## 6. Sync model

The model is "single canonical aviary, server is the only writer of canonical state." This is a sync architecture only in the loose sense — there is nothing to sync because there is one record.

### Read path

1. Client opens; sends `GET /aviary/snapshot` with credentials.
2. Server returns the current snapshot (built from the canonical state, the next-call schedule, and the daylight/weather state).
3. Client renders from the snapshot, interpolating motion to the next predicted state.
4. Client refreshes the snapshot:
   - On `visibilitychange` -> visible.
   - When a frame gap longer than ~500ms is detected (laptop suspend, browser throttle).
   - On a low-frequency keepalive (every 60–120s while visible).
   - After certain user actions (offer, listen-in start) where the server may have computed an immediate response.

### Write path

1. Client batches events and POSTs to `/aviary/events` at a small cadence.
   - Presence pings: every ~15s while presence conditions hold (visibility + focus + recent input).
   - User-initiated events: immediate.
2. Server appends to the event log; assigns `ingest_seq`.
3. The next tick consumes events and writes canonical state.
4. Client observes the effect on the next snapshot pull.

### Multi-device behavior

- Two devices signed into the same account see the same canonical aviary, in the same mood, with the same drift history, because both pull the same snapshots.
- Both devices can submit events simultaneously. They are interleaved by `server_received_at` and `ingest_seq` and consumed in order.
- Presence pings from overlapping sessions are deduplicated to distinct seconds at tick time. A user with two devices both showing the aviary does not get double-presence-time.
- Listen-in: each session's listen-in is recorded as that session's interaction. The tick deduplicates effective listen-in seconds across sessions per (account, bird) so drift contribution is summed once per second of attention.
- Settle: a session-end gesture, applied per session. A second device that's still actively presenting does not get a "settled" lighting state. This is a defensible call: settle is "presence ends here," and presence is per-session.

### Conflict prevention

There is no conflict resolution because there is no client-vs-client conflict surface:

- Clients never submit absolute personality state.
- Mood is computed server-side; clients cannot modify it.
- Perches and call schedules are server-decided.
- Settings (rename, prefs, accessibility prefs) are last-writer-wins on the small per-account settings record; the failure mode (accidentally overwriting a setting from another tab) is acceptable and reversible.

The "no last-write-wins for personality state" rule is enforced architecturally by the lack of any client-write code path.

### Identity continuity

- `bird_id` is never reused, never reset, never regenerated, never swapped. Every migration that touches the bird table preserves `bird_id` and the personality vector. A structural test on the migration runner asserts this on every PR that adds a migration.
- A "reset bird" operation does not exist as a feature, an admin tool, or a debug command.

### Open calls

- **Settle scope**: per-session, not global. Reasoning: "presence ends when presence ends" maps cleanly to per-session; a global settle would require client-to-client signaling.
- **Real-time push from server**: deferred to v1.x. v1 is pull-based on a low-frequency keepalive; this is good enough for the slow-tick cadence.

---

## 7. Frontend rendering pipeline

### Stack

- TypeScript throughout.
- **Aviary scene**: Canvas2D with a small hand-built scene graph. Reasoning: 2MB initial bundle cap is tight; Canvas2D draws our scene at 60fps on the 5-year-old laptop target; WebGL adds bundle and complexity for marginal benefit. (We will profile once in alpha; if Canvas2D fails, the swap point is small — the renderer is one module behind a stable interface.)
- **Chrome (top bar, settings, notebook, modals)**: Preact (or equivalent ultra-minimal React-compatible). Tokenized CSS variables for the design system. No heavy CSS framework.
- **State management on the client**: a single small reactive store for snapshots and accessibility prefs; no Redux or similar.
- **Code-splitting**: aggressive. The aviary surface is the only initial route. Account settings, accessibility settings, notebook, visit invitation flow, export, deletion are separate chunks fetched on demand.

### Initial paint (the <500ms target)

Hot-path pipeline:

1. User navigates; CDN serves edge-rendered HTML.
2. Edge HTML for signed-in users includes an inline `<script>` containing `__initialAviary` — a small JSON snapshot (≤ ~5KB) sufficient to draw 1–2 birds in starting poses.
3. The HTML imports a tiny critical chunk (≤ 50KB gzipped) that contains the renderer bootstrap, bird silhouette set, and initial pose rendering. This chunk paints the first bird.
4. The main bundle (the rest of the renderer, audio, chrome, route handling) loads in parallel and takes over after first paint.
5. The audio context is not created until first user gesture (browser autoplay policy); captions auto-enable until then if the user has audio enabled.

For unsigned-in users, the edge serves a minimal sign-in surface that does not fetch the aviary chunks; a signed-in user landing on the unsigned variant is rare and tolerable.

### Scene composition

The scene has three z-planes with very subtle parallax:

- **Background**: gradient sky (palette interpolated from `daylight_state`), soft foliage silhouettes, cloud whisps. Re-rendered on phase change.
- **Mid plane**: perches and birds. Perch rendering is static SVG-derived geometry; bird rendering is layered (silhouette + plumage + accents) per species template, parameterized by `plumage_saturation_render` and mood-shaping parameters.
- **Foreground**: occasional branch sweep, leaf and feather drift, occasional foreground bird-overlap (when a bird flies between foreground and mid).

### Idle micro-motion

Per-bird state machine running at 60fps client-side:

- States: preening, scanning, head_tilting, resting, calling. Transitions sampled on a per-bird local timer with mood-shaped probabilities.
- Motion paths are parametric (eased noise + small randomized phase offsets); never a looped key-frame sequence. The same motion is never replayed identically. (Hard rule.)
- Mood expression in motion:
  - `wary`: bird perches further back (a within-zone perch slot offset), scans more (faster head-position changes), pauses longer between micro-motions.
  - `content`: more preening, slower scans, eyes half-closed in rest.
  - `curious`: head-tilts more often, peeks toward sound sources, lighter weight-shifts.
  - `drowsy`: lowered body, fluffed feathers, fewer transitions.
  - `alert`: upright posture, faster scans, occasional wing-twitch.

### Transitions

- Bird perch-to-perch: server schedules; client renders flight along an eased path with an arc shaped by `boldness_render` (high boldness flies low and direct; low boldness flies higher and curved). If the client opens mid-transition (snapshot indicates a transition past midpoint), client picks up at the right percentage so the bird arrives at the right time.
- Mood transition: cross-fade in pose set; never a hard cut. New mood-shaping parameters interpolate over a few seconds.
- Day/night: gradual color-temperature shift over minutes (driven by `phase_progress` + client-local interpolation).

### Reduced-motion mode

- Trigger: `prefers-reduced-motion: reduce` media query OR explicit accessibility settings opt-in.
- Renderer swaps from continuous micro-motion to a sequence of distinct poses cross-fading with longer durations (e.g., 1.5–3s).
- Flight transitions become cross-fades between perches rather than animated paths.
- Leaf and feather drift removed entirely. Parallax tilt removed. Day/night color shifts retained (slowed).
- Calls and captions unaffected.
- Birds still drift, mood still changes, notebook still accrues. The reduced-motion render is its own designed surface, not a stripped fallback.
- Switching modes mid-session is supported (cross-fades into the new mode).

### Chrome

- Top bar: account/settings, accessibility, notebook, offer, settle. Sparse.
- Top bar fades to ~10% opacity after ~3s of cursor stillness; restores on cursor or keyboard activity. Fade and restore use easing, not snap.
- Chrome surfaces (settings, notebook) open as overlays that dim the aviary slightly; ESC closes; focus is trapped within them.
- The aviary scene itself never carries buttons, badges, hover-tooltips, overlay icons, or inline labels. (Hard rule.)

### Empty / loading / cold-cache state

A single quiet field design covers all three:

- Soft sky-color background (interpolated from current day/night phase if known; default to morning).
- One or two faint motion cues — a leaf passing, a soft breeze in foliage.
- No spinner. No "loading…" label. No progress bar.
- If the snapshot delays beyond ~1.5s, an additional motion cue appears (a feather drift).

The empty-aviary state (just-after-adoption, before first bird flies in) is the same quiet field; the first bird then enters with a soft fly-in to its starting perch.

### Performance enforcement

- Bundle audit on every PR: hard fail at 2MB gzipped initial route. Tracked per-route.
- Synthetic perf check: a headless-browser test loads the page on a throttled network, measures first-bird-render, and fails CI if > 500ms on the target profile.
- 60fps idle-motion test: a synthetic 30-min session in a headless browser captures frame timings; fail if 1% of frames exceed 16.7ms.
- Memory growth test: same 30-min session, capture heap before and after; fail if RSS grew beyond a calibrated noise floor.
- All perf tests are gated as required CI checks.

### Open calls

- **Renderer**: Canvas2D for v1. Documented swap point if profiling shows it can't hit 60fps on target hardware.
- **Chrome framework**: Preact. Light wrapper on top of CSS-variables-based design tokens. Re-evaluate at v1.x.
- **Inline initial state size budget**: ~5KB. Edge function size-checks before injection.

---

## 8. Audio pipeline

The audio pipeline is the affective spine; if it sounds canned, the product reads as theater. Procedural-only is non-negotiable.

### WebAudio context lifecycle

- Single `AudioContext` per page. Lazy-created on first user gesture (click, keypress) due to browser autoplay policies.
- Until the context is created (or if creation fails), the aviary plays in graceful silence with captions auto-enabled.
- The "tap to enable calls" affordance is a small icon in the top bar that briefly highlights on first session; fades back into the chrome as normal.
- AudioWorklet preferred for synthesis. (Available in all browsers in our support matrix; ScriptProcessor not used in v1.)

### Procedural synthesis

- Each species has a motif library: a small set of parameterized motifs (rises, falls, trills, calls, repeats, soft-breath fillers) implemented as small DSP graphs (oscillators, formant filters, envelopes, low noise).
- A call is rendered as a tree of motifs sampled at runtime from a per-call seed (`next_call_seed_value` in the snapshot).
- Variation comes from randomized selection within personality- and mood-shaped distributions: e.g., a high-vocal-frequency bird in `content` mood favors longer multi-motif calls; a `wary` bird favors short single-motif calls.
- Per-species recognizability: each species has fixed signature parameters (a characteristic pitch range, attack shape, motif weights) such that the user can identify the species across mood and drift. Per-bird differentiation within a species comes from a stable per-bird deviation seed (small offsets in pitch center, attack timing).

### Chorus mixing

- The simulation schedules calls; sometimes it widens scheduling windows to encourage temporal overlap (a chorus event).
- The client schedules each call in the AudioContext at its `scheduled_at` time relative to `server_now`.
- Mix levels per call: when multiple birds call within a small window, the per-call gains are scaled with a soft-knee compressor in the mix bus to preserve intelligibility while preserving the impression that several birds are calling at once.
- The chorus mechanic is the audible payoff of procedural synthesis. Calibrate so that two simultaneous calls produce clean co-articulation, not destructive interference.

### Listen-in mix decay

- Listen-in is a pure client-side mix change. Engaging listen-in on a bird ramps that bird's call gain up by ~+6 dB over ~400–600ms while ramping other birds' gains down by ~-9 dB to ambient.
- Disengaging uses the same ramp curve in reverse.
- Other birds never go silent. (Hard rule.)
- The listen-in event is reported to the server for drift; the server does not re-emit the mix.

### Captioning

- Captions are generated by the same call-grammar code path that drives the synth. The call descriptor object emits `audio_render_params` and `caption_text`; both are derived from the same seed.
- Captions render as small naturalist-prose chips near the calling bird, fading in over the call's attack and out over its decay. Caption text uses the same lowercase, present-tense, specific voice as the field notebook.
- Captions auto-enable when the audio context is unavailable, when the user has muted audio, or by explicit accessibility preference.

### WebAudio fallback

- If WebAudio cannot be initialized, the aviary plays in graceful silence with captions on by default.
- No recorded-audio fallback path is loaded or shipped. The audio chunk only contains procedural synthesis code.
- Users on browsers without WebAudio see a matter-of-fact unsupported-browser surface.

### Memory and CPU

- All synth nodes are pooled and reused. A bounded pool of motif graphs is preallocated; calls reuse them with parameter resets.
- Per-call allocation budget is zero in steady state; allocation profiling is part of the 30-minute memory test.
- CPU budget: 20% of one core during a chorus on the 5-year-old laptop target. Profiling is part of CI.

### Mute and audio prefs

- Top-bar settings include a mute toggle and a captions-on/off toggle. Captions are forced on when mute is on or audio is unavailable.
- Volume control: a single linear control mapping to the master gain.

### Open calls

- **Sound design**: a named role on the team. Motif libraries land as PRs reviewed by sound design. We commit to v1 with six species each having a well-tuned motif library; expansion is calibration work, not v1.
- **Call seed determinism**: deterministic across clients; this means a call rendered at the same `scheduled_at` on two devices is bit-identical (to the limits of float precision in the AudioWorklet). Visit coherence depends on this.

---

## 9. Accessibility surfaces

Accessibility is a designed surface, not a checklist.

### Screen-reader narration

- The aviary surface includes an `aria-live="polite"` container that updates with running naturalist prose.
- Cadence: roughly one update per 30–60s at idle. User-initiated events (return-greeting on session start, an offer reaction, a settle gesture) get priority bumps.
- Voice: same as the notebook — lowercase, present-tense, specific.

```
a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.
```

- Generation: server-side, from the same template grammar that drives the notebook (different template set, same voice). Client also has a fallback narration generator for snapshot-only contexts.
- Forbidden patterns blocklisted: ARIA-style state lists ("warbler perched at high branch"), gamification language, "you" address (except in matter-of-fact system surfaces), counts.

### Captioning for calls

See audio pipeline. Captions are short prose descriptions emitted by the call grammar, rendered near the calling bird, naturalist voice.

### Reduced-motion mode

See frontend pipeline. Designed surface; cross-fade pose-sets, ornament motion removed, slow color shifts retained.

### Keyboard navigation

- Tab order: top-bar items first (account/settings, accessibility, notebook, offer, settle), then a single Tab into the aviary scene focuses the first bird.
- Within the aviary: arrow keys move focus among birds. Enter triggers listen-in on the focused bird; Escape exits listen-in. Tab from the aviary returns to the top bar.
- Focus indicators: a soft white halo with a thin dark inner stroke that reads on both bright and dim aviary states. The visual designer tunes the exact treatment.
- Modals (settings, notebook, offer chooser) trap focus and return it to the trigger on close.
- All interactive surfaces are reachable by keyboard. Tested by an automated keyboard-only audit in CI.

### WCAG AA contrast

- All chrome text and overlay text passes WCAG AA against every backdrop it can appear on.
- Captions and narration overlays have a slim contrast-aware tint or text-shadow tuned to remain readable across day/night palettes.
- The design system enforces ratios per surface; a CI lint blocks PRs that introduce text overlays with insufficient contrast.

### Voice continuity across surfaces

- Naturalist voice: aviary surface, notebook, narration, captions.
- Matter-of-fact voice: auth flows, account settings, sync errors, accessibility settings, unsupported-browser surface, deletion confirmation, export-link emails.
- The voice split is enforced by code-organization: a `voice/naturalist/` module and a `voice/matter_of_fact/` module are the only sources of user copy. A lint rule forbids inlining user-visible strings outside these modules.

### Audio prefs surface

- A small accessibility settings page exposes: captions on/off (default: off; forced on when mute or no audio), mute, master volume, narration on/off, narration rate (slow/medium/fast), reduced-motion override.

### Screen-reader testing

- Manual review by an accessibility reviewer on every PR touching scene rendering, audio pipeline, narration, or captions.
- Automated checks: live-region update tests, focus order tests, contrast lints, aria-attribute presence on chrome.

### Open calls

- **Narration generation site**: server-side primary, with a client-side fallback. Server-side avoids running a heavier template engine on the client and keeps voice consistency between narration and notebook.
- **Caption-near-bird placement**: hover-style positioning that respects responsive scene layout. Implementation detail in the rendering spec.

---

## 10. Performance budgets and observability

### Budgets (non-negotiable)

- **Initial JS bundle ≤ 2MB gzipped** at first paint of the aviary route. CI hard fail.
- **First bird visible ≤ 500ms** on a mid-tier mobile device over 4G. Synthetic check.
- **60fps idle motion** on a 5-year-old mid-range laptop. Synthetic check.
- **No memory growth over 30 minutes** in client. Synthetic check.
- **Simulation tick p99 ≤ 5s.** Server alarm.

### Synthetic performance fleet

- A small fleet of automated browsers in a few common geographies runs the full session loop on a schedule (sign in, load aviary, watch for 30 minutes, settle).
- Reports first-bird-render distribution, frame-rate distribution, audio-context errors, simulation-tick-latency proxy (snapshot-pull latency).
- Failures alert; the synthetic fleet is the primary signal of perf regressions.

### Real User Monitoring (aggregate-only)

- Page load timings, first-bird-render timings, render-frame timings, audio-context errors, snapshot-pull latencies.
- All metrics are aggregate. No per-account dimension is permitted in any metric.
- Sampling strict: histograms anonymized; no per-event PII.

### Telemetry boundary (architectural)

- The simulation DB and the analytics warehouse live on different networks. Network policy denies analytics roles read access to the simulation DB.
- The telemetry library has a static schema enforced at build time; metric definitions are PR-reviewed by a privacy-domain owner. No metric definition can name `bird_id`, `personality_*`, `mood`, `notebook_*`, `interaction_event_*`, or any field that could reconstruct a user's relationship with their birds.
- Per-bird interaction events flow into the event log only; the event log has no read path to telemetry.

### Server observability

- Per-tick logs: hashed `account_id` for log-cardinality, tick duration, deltas applied (bucketed counts, never raw deltas), event counts by type. Email is never logged. Bird names are never logged.
- Tick correctness audits: a daily job samples accounts and replays their event logs through a test tick to confirm idempotency; alarms on drift.
- Magic-link issuance and verify ratios; abnormal ratios alarm on possible abuse.

### Privacy in observability (lints)

- Linter enforces the metric schema. PRs that add a metric naming a forbidden field fail CI.
- Annual external audit of metrics emitted (in addition to internal review).

### Browser support

- Chrome, Safari, Firefox, Edge — last two major versions each.
- Older browsers see a matter-of-fact unsupported-browser surface explaining what's needed.

### Open calls

- **Synthetic fleet provider**: pick a vendor or run our own; vendor is fine for v1 since the test surface is the public site under a test account.
- **Tick latency alarm**: p99 5s as the trip threshold, with a separate p50 1s alarm to catch shape changes before they become user-visible.

---

## 11. Rollout

### Phase 0 — internal alpha (~2 weeks)

- Single deployment, small invite list (team + close advisors).
- Two birds per aviary, hard-locked. No third-bird offering. Visit feature off.
- Goal: validate aliveness — does the first frame land, does the bird notice, does the call feel right? Calibration data for drift weights, mood transition probabilities, weather rates, notebook entry sparsity.
- Rendering fallback (Canvas2D vs WebGL) decided in this phase.
- Procedural call motif libraries reviewed by sound design.
- Reduced-motion mode reviewed by an accessibility reviewer.

### Phase 1 — closed beta (~4 weeks)

- Larger invite list, magic-link sign-in only.
- Cap birds at 3; allow third-bird offer at age milestone (compressed for testing).
- Visit feature available, off by default.
- Goal: stress sync, multi-device, accessibility surfaces. First time the synthetic perf fleet runs against production.
- Telemetry boundary verified end-to-end (privacy-domain owner audits the metric definitions).

### Phase 2 — public v1

- Open sign-up.
- Bird cap at 7; new species offered at age intervals.
- Notebook fully wired with reviewed templates.
- All accessibility surfaces shipped at launch (not deferred).

### Ramp policy

- Birds-per-aviary cap raised by month, based on observed call recognizability in user testing. Initial schedule:
  - Month 1: cap at 2 (no third-bird offers)
  - Month 2–3: cap at 3
  - Month 4–6: cap at 5
  - Month 7+: cap at 7
- Drift weights are held constant after public v1; changes go through a calibration review.
- Notebook templates expand based on writer-of-record output, never auto-generated.

### Instrumentation from day one

- First-bird-render timings.
- Simulation-tick latencies and counts.
- Audio-context errors.
- Magic-link issuance/verify ratios.
- Sync-failure counts.
- Accessibility-mode adoption (aggregate).
- Notebook entry rates (aggregate, per day, no per-account dimension).
- Call counts (aggregate, no per-account, no per-bird).

### What we deliberately don't measure at launch

- DAU, MAU, retention curves, churn — we don't operate the product on engagement metrics.
- Per-bird metrics aggregated across users — privacy boundary.
- Visit frequency at the user level — non-goal architectural absence.
- Anything that would imply a leaderboard or comparison surface — non-goal architectural absence.

We monitor health (errors, perf), not engagement. The product's success is felt; it does not have a dashboard.

### Versioning and migration

- Snapshot and event-log payloads are versioned (`v1`, `v1.1`, etc.).
- Server supports the previous schema for at least 30 days after a new version rolls out.
- Migrations on the bird table preserve `bird_id` and the personality vector on every row; a structural test asserts this on every migration PR.
- Data migrations are backed up before applying; restore drills run quarterly.

### Open calls

- **Calibration review process**: documented in alpha; includes the harness, the trait-change distributions, and a sign-off rubric.
- **Public launch communication**: written in matter-of-fact voice for all system surfaces; the site copy itself in naturalist voice.

---

## 12. Risks

A close reading of where this build is most likely to fail. Each risk has a concrete mitigation.

### A. Drift calibration drift (highest-impact)

The "1 week measurable / 3 weeks visible" target is a calibration claim integrated across many small weights. The risk: in production, drift is too fast (Tamagotchi-feeling), too slow (screensaver-feeling), or asymmetric (some traits drift, others don't), and we won't know until users report that their birds feel wrong.

Mitigation:
- Calibration harness that replays scripted event logs against the simulation and reports trait-change-per-week distributions. Run on every weight change.
- Lock weights behind a calibration review (sign-off required from the engine domain owner and the writer-of-record).
- Weights are a single config; rollback is an immediate config push, no migration.
- Regression test: reproduce production traffic shapes against alpha-tuned weights and confirm distributions match within tolerance.

### B. Sync correctness regressions

The hard rule "no client writes personality state" is easy to violate by accident — a refactor adding a "set boldness for testing" endpoint, an admin tool that bypasses the tick, a migration that recomputes drift from event logs.

Mitigation:
- DB grants: only the simulation worker role has `UPDATE/INSERT` on personality, mood, perch tables.
- Structural CI test: scans the codebase for any code path that writes those tables outside the worker.
- Migration review checklist: every migration PR confirms it doesn't reset or overwrite personality vectors.
- Code-search lint in pre-commit hooks for keywords like "set boldness," "reset bird," "regenerate personality."

### C. Audio uncanniness

Procedural calls can sound robotic if the motif library is too small or the parameterization too coarse. A user who hears the same call shape twice in a session is on the path to feeling the canned-software signal.

Mitigation:
- Sound design owns motif libraries and approves each species's library before release.
- CI test: synthesize 1000 calls per species and assert a minimum spectral-feature variety threshold (e.g., distinct fundamental-frequency contour count above a floor).
- Internal soak: A/B-style comparison of two motif configurations on the same account before promoting changes.
- User testing in alpha specifically asks "did any call sound the same as another?"

### D. Accessibility regressions

Reduced-motion mode and narration are easy to break with rendering refactors.

Mitigation:
- Dedicated CI tests: reduced-motion mode renders a known scene and snapshots the output; narration prose pipeline has unit tests.
- Manual review by an accessibility reviewer on every PR touching scene rendering, narration, or audio.
- Alpha includes an accessibility-only test pass with screen reader users.

### E. Privacy boundary erosion

The "per-bird events never enter telemetry" rule is structural, but nothing technical stops a future engineer from naming a per-bird metric. The risk is gradual.

Mitigation:
- Telemetry library with a static metric schema enforced at build time.
- Forbidden field-name lint on metric definitions.
- Privacy-domain owner is a required reviewer on metric-definition PRs.
- Quarterly audit of metrics emitted, including external review annually.

### F. Snapshot delivery latency

The 500ms first-bird budget is fragile. A slow API edge or a cold cache can blow it.

Mitigation:
- Edge-cached HTML with inline initial state per signed-in account.
- CDN-rendered fallback HTML for unsigned-in users that loads only the chrome (sign-in surface).
- Per-region edge presence covering the user base.
- Synthetic perf fleet alarms on first-bird-render regressions.

### G. Visit-feature abuse

Magic-link visit invites could be misused (token sharing beyond the named visitor, scraping aviary state).

Mitigation:
- Invites bound to a single visitor email (token issued for that email; verify hash of the email at consumption).
- Visit tokens single-use per device session; reuse from a different device requires the original visitor email.
- Revocation immediate on host action.
- Rate limit on invite issuance per host per day.
- Visit snapshots throttled at a lower rate than host snapshots.

### H. WebAudio support edge cases

Older browsers, browser autoplay policies, hardware audio failures.

Mitigation:
- Graceful silence with captions on by default.
- "Tap to enable calls" affordance for autoplay-blocked contexts.
- Matter-of-fact unsupported-browser surface for the long tail.
- Audio-context-error counts as an aggregate metric; degradation is observable.

### I. Personality-vector loss (catastrophic, silent)

A bug or migration that resets a user's personality vector silently destroys the user's relationship with their birds. The failure does not pop alarms — the user just feels something is wrong.

Mitigation:
- Per-tick personality state is journaled (write-ahead) before being applied; on tick failure, the prior state is restored.
- Daily backups of the simulation DB, tested by restore drills quarterly.
- Structural test: any migration touching the bird table preserves `bird_id` and personality vector on every row.
- Alarm: a daily job samples accounts and confirms personality vectors are within a tolerance of the previous day's snapshot (no large unexplained drops).

### J. Notebook prose drift

Auto-generated prose can fall into repetitive phrasing or break the voice contract.

Mitigation:
- Templated grammar reviewed by writer-of-record.
- Lint on prose templates for forbidden patterns ("achievement," "streak," "you've been here," "your friend," explicit number naming, "level up").
- Quarterly sampled human review of generated entries.
- Entries store `generation_source` for audit.

### K. Single-tenant isolation in the simulation worker

A bug that lets one account's tick read or write another's state.

Mitigation:
- `account_id` as a partition key in every query; row-level security where the DB supports it.
- Integration tests that try to read across accounts and assert failure.
- Worker job dispatch keyed on `account_id`; worker process scoped to one account at a time.

### L. Time-of-day computation failures

Wrong timezone for the user produces "morning calls at midnight," a small but real aliveness regression.

Mitigation:
- Timezone resolution on every snapshot pull (client passes its current tz; server stores last-known tz on account).
- Fall back to account-creation tz if present.
- Synthetic test: load the aviary with several tz fixtures and assert the day/night phase matches.

### M. Magic-link replay & email change attacks

Standard auth risk surface.

Mitigation:
- Token single-use; invalidated immediately on consumption.
- Token expiration at 15 minutes.
- Rate limit per email and per IP on issuance.
- Email change requires verification of the new address; old address continues to work until the new verifies.
- Sessions revocable; abnormal session creation rates alarm.

### N. The temptation to add a streak/achievement

Not a technical risk, but a real product risk. Six months in, a "harmless" engagement feature pitch arrives.

Mitigation:
- Non-goals are written explicitly in the PRD.
- A review checklist on PRs that touch any user-facing surface asks: does this surface visit-frequency, attention, or count? Does it address the user as "you" outside system surfaces? Does it compare aviaries?
- Architectural absence: the metrics for streaks/achievements don't exist; the schema has no `visits_count`, `streak_start`, `last_visited`. Re-introducing the feature is costly, not free.
- The non-goals file is required reading for any new contributor.

### O. Notebook entry rate calibration

If sparsity rules are too strict, the notebook feels dead; too loose, every session generates noise that dilutes the entries that matter.

Mitigation:
- Hard cap (1/account/day average) set in alpha and revisited based on reviewer judgment in beta.
- Sample of entries reviewed weekly during beta.
- A "first-week entries" set is hand-reviewed for new accounts to ensure the early experience lands.

### P. Procedural call seed determinism failure

If two clients of the same account synthesize calls slightly differently, multi-device coherence breaks. Visit coherence breaks similarly.

Mitigation:
- Deterministic seed-driven synthesis in AudioWorklet.
- Cross-platform synthesis test: same seed, same `scheduled_at`, two browsers, assert audio buffer is bit-identical to a tolerance.
- Coherence is a first-class test.

---

## 13. Cross-cutting concerns and conventions

### Voice handling in code

- All user copy lives in `voice/naturalist/*` or `voice/matter_of_fact/*`.
- Lint forbids inline user-visible strings outside these modules.
- Forbidden phrases ("welcome back," "great to see you," "achievement," "streak," "level up," "you've been here," "earn," "unlock") fail the lint anywhere in the codebase.

### Privacy in code

- The `analytics/` package cannot import from the `simulation/`, `birds/`, `notebook/`, or `events/` packages.
- The `mailer/` package handles email; no other package may decrypt the email field.
- Logging utilities scrub `bird_id`, `personality`, and `notebook` fields by default.

### Testing strategy

- Unit tests on drift function (monotonic, idempotent, calibrated).
- Integration tests on tick (single-writer, account-scoped).
- Property tests on call grammar (recognizability across mood, no exact repetition).
- E2E tests on auth flow, sync flow, visit flow, settle flow, listen-in flow, offer flow.
- Accessibility tests: keyboard navigation, narration cadence, reduced-motion mode rendering, contrast.
- Performance tests: bundle size, first-bird-render, 60fps idle, 30-min memory.
- Voice tests: forbidden-phrase lint, voice-module-scope lint.

### Security

- All endpoints HTTPS only.
- Session cookies HttpOnly, SameSite=Lax, Secure.
- Magic-link tokens random-256-bit, hashed at rest.
- Visit tokens random-256-bit, hashed at rest, bound to email and visit session.
- CSP locks down the script origins to first-party + audio worklet origin.
- Rate limits on auth endpoints, invite issuance, export requests.

### Open calls

- **Internationalization**: v1 is English-only. Voice modules are structured to accept locale variants; localization is a v1.x effort.
- **Privacy policy text**: lives in account settings, drafted in alpha alongside the engineering build.

---

## 14. Dependencies and team-shape implications

### Required roles

- Engine domain owner: drift function, mood transitions, tick correctness, calibration harness.
- Frontend rendering owner: scene composition, idle motion, transitions, reduced-motion mode.
- Audio designer: motif libraries, procedural synthesis tuning, chorus mixing, captioning voice.
- Writer-of-record: notebook templates, narration templates, voice continuity across surfaces.
- Accessibility reviewer: required reviewer on rendering, audio, narration, captions PRs.
- Privacy-domain owner: required reviewer on telemetry and metric-definition PRs.
- Backend infrastructure: auth, API, DB, simulation worker scheduler, mailer.

### External dependencies

- CDN with edge-function support (Cloudflare Workers / Vercel Edge / equivalent).
- Transactional email provider (with high deliverability for magic links).
- Object storage (signed URLs for export downloads).
- Monitoring and alarming infrastructure.
- Synthetic perf fleet (vendor or self-hosted).

### Deferred to v1.x or later

- Native mobile apps.
- Server-side push (SSE) for snapshot updates.
- Localization beyond English.
- Richer notebook prose generation (LLM-grounded with strict harness).
- Co-presence in visits.
- Aviary states beyond v1's scope (named scenes, customization).

---

## 15. Final framing

Pocket Aviary's value is felt-aliveness. The implementation rules in this plan exist to keep that feeling architecturally durable: server-only writes for personality state to keep drift coherent across devices and time; procedural-only audio to keep the chorus alive; designed-not-stripped accessibility surfaces to keep the affective core available to every user; aggregate-only telemetry to keep the relationship private; the absence of any streak/achievement metric to make their reintroduction structurally costly.

A frontier team executing this plan should hold three rules above everything else:

1. **The aviary appears already in motion.** The first frame has birds mid-action. There is no entry animation. There is no spinner. There is no "Welcome back" toast.
2. **Drift is monotonic toward expressive.** A bird that gets ignored becomes ambient, not distressed. The drift function never decreases a trait. The schema does not store data that would enable a "neglect" feature.
3. **The user never sees a number for their bird's personality.** Snapshots do not carry the personality vector. The render parameters look superficially like the vector and are not. There is no debug surface to expose them.

If those three rules hold, the rest of the product gets the room it needs to feel like what it claims to be: a small, calm, alive place that has been running without the viewer, and is glad — quietly, never said — that the viewer came back.
