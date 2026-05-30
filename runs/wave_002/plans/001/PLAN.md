# Pocket Aviary v1 Implementation Plan

## 1. Scope and product boundaries

Pocket Aviary v1 is a browser-only product that delivers a single canonical aviary per account, with two starter birds and a gradual expansion path up to seven birds based on aviary age rather than engagement. The implementation must preserve the product's core affective contract: the aviary feels alive, continues without the viewer, notices rather than announces, and never turns the relationship into a game, duty, or social performance.

### In scope

- Browser-based client for modern Chrome, Safari, Firefox, and Edge
- Email magic-link authentication with revocable per-device sessions
- One canonical aviary per account, synced across devices
- Server-side simulation tick that advances moods, drift, call timing, weather windows, greeting readiness, and notebook candidate events whether or not a client is open
- Two starter birds at onboarding, age-gated additions up to seven
- Hidden personality vectors and visible mood expression
- Presence accounting based on visibility + focus + recent input conjunction
- Return greeting, listen-in, offer, settle, and field notebook
- Quiet visit invitations with read-only rendering for visitors
- Accessibility surfaces: narration, reduced-motion mode, captions, keyboard support
- Operational telemetry limited to aggregate health and performance metrics

### Explicitly out of scope

- Native mobile apps
- Gamification of any kind: streaks, counters, badges, levels, achievements, leaderboards, visit-frequency surfaces
- Tamagotchi mechanics: hunger, illness, decay, punishment for absence, negative drift on neglect
- Public social surfaces: profiles, follows, comments, discovery, feed, co-presence
- Shared aviaries or multi-user ownership
- Scene customization, multi-aviary accounts, payments, or recommendation features

### v1 success criteria

- The first render feels like an already-running aviary rather than a loading UI
- Birds maintain continuity across devices and across weeks
- The product remains emotionally calm even in error and edge states
- Accessible modes feel like the real product, not stripped-down fallbacks
- The data architecture makes privacy promises technically enforceable rather than policy-only

## 2. Architecture

### High-level system shape

Use a thin-client, server-canonical architecture with four runtime domains:

1. Web client
2. API and auth service
3. Simulation service
4. Persistence and asynchronous delivery infrastructure

The architectural rule is simple: clients render and emit events; the server owns truth. Personality state, mood state, notebook state, and visit authorization are canonical on the server. The client never computes durable bird state and never writes trait values directly.

### Recommended deployment topology

- `web-app`: SSR/edge-delivered shell plus client-side rendering app
- `api-service`: authenticated HTTP API for snapshots, events, notebook access, account surfaces, visit flows
- `simulation-worker`: scheduled tick processor that advances canonical aviary state
- `email-worker`: magic links, export links, and visit invites
- `postgres`: source of truth for accounts, birds, moods, notebook entries, visit records, invites, and session tokens
- `event-log`: append-only interaction event table in Postgres initially; keep schema compatible with later queue extraction if needed
- `cache/cdn`: edge-cached static assets and optionally edge-delivered boot snapshot envelope

This should be implemented as a modular monolith for v1 rather than microservices. Separate code boundaries matter; distributed runtime complexity does not. The engineering risk is in simulation correctness and feel, not service sprawl.

### Client/server split

Server responsibilities:

- Authenticate users and visitors
- Persist account, aviary, bird, notebook, visit, and settings data
- Record interaction events
- Compute presence-time windows from client pings
- Run tick-driven simulation updates
- Author all personality and mood mutations
- Materialize snapshot payloads for rendering
- Enforce visit read-only permissions
- Generate notebook entries and screen-reader text payloads from canonical state

Client responsibilities:

- Render current scene and transitions from server snapshot
- Synthesize procedural audio from server-provided call grammar state
- Interpolate motion between snapshots
- Detect local presence signals and submit pings/events
- Manage accessibility presentation layers
- Handle optimistic transient UI only where it cannot alter canonical bird state

### Render pipeline boundary

The render boundary must sit at "scene directives," not at raw simulation internals and not at finished animation clips.

The snapshot should provide:

- Bird identities, names, species, mood, perch zone, pose family, animation phase seed, and greeting eligibility
- Ambient scene state: local-time phase, settled/not settled, weather event window, palette mode
- Call runtime state: motif family, tempo envelope, current mix hints, next-call window seeds
- Accessibility narrative surface inputs
- Notebook excerpts and metadata when requested

The snapshot should not expose:

- Numeric personality vector values
- Drift deltas
- Hidden calibration coefficients
- Internal conflict-resolution metadata unless needed for sync integrity

This boundary allows clients to feel alive while keeping drift and mood logic server-owned.

## 3. Data model

### Core entities

#### Account

- `account_id` UUID primary key
- `email_encrypted`
- `email_verified_at`
- `status` enum: active, pending_deletion, deleted
- `created_at`, `updated_at`, `deleted_at`
- `settings_json` containing accessibility, audio, and notification preferences
- `aviary_id`

#### Session token

- `session_id` UUID
- `account_id`
- `device_label`
- `created_at`, `last_seen_at`, `revoked_at`, `expires_at`

#### Aviary

- `aviary_id` UUID
- `account_id`
- `created_at`
- `current_scene_phase` enum-ish field for dawn/morning/day/evening/night
- `is_settled`
- `active_weather_state`
- `last_tick_at`
- `next_bird_unlock_at`
- `version` monotonically increasing integer for snapshot sequencing

#### Bird

- `bird_id` UUID
- `aviary_id`
- `species_id`
- `display_name`
- `adopted_at`
- `position_zone` enum: front/middle/back
- `pose_state`
- `mood_state`
- `mood_entered_at`
- `last_greeted_at`
- `last_offer_at`
- `last_listen_in_at`
- `call_signature_seed`
- `animation_seed`
- `is_nocturnal_variant` boolean if species-specific

#### Personality vector

Separate table from bird row so durable identity and tunable traits can evolve independently.

- `bird_id`
- `boldness`
- `social_warmth`
- `vocal_frequency`
- `plumage_saturation`
- `curiosity`
- `updated_at`

#### Presence window / session presence aggregate

- `presence_window_id`
- `account_id`
- `session_id`
- `started_at`
- `ended_at`
- `qualified_seconds`
- `source_visibility_state`
- `source_focus_state`
- `source_recent_input_state`

Store raw-ish qualified windows rather than only cumulative totals so calibration can be adjusted without reinterpreting ambiguous logs.

#### Interaction event

- `event_id` UUID
- `account_id`
- `aviary_id`
- `session_id`
- `bird_id` nullable for global actions
- `event_type` enum
- `event_payload_json`
- `occurred_at`
- `ingested_at`
- `processed_at`
- `tick_id` nullable once consumed

Event types:

- `presence_ping`
- `listen_in_started`
- `listen_in_ended`
- `offer_seed`
- `offer_song_fragment`
- `offer_still_pool`
- `settle_started`
- `settle_undone`
- `session_visible`
- `session_hidden`
- `visitor_view_started` and `visitor_view_ended` in visit log only, excluded from simulation inputs

#### Notebook entry

- `entry_id` UUID
- `aviary_id`
- `bird_id` nullable
- `entry_text`
- `entry_type` enum: greeting, quiet_stretch, weather, social, perch_change, offer_reaction, seasonal_observation
- `observed_at`
- `created_at`

#### Visit invite

- `invite_id` UUID
- `host_account_id`
- `visitor_email_encrypted`
- `visitor_email_hash` for lookup/rate limiting
- `token_hash`
- `created_at`
- `expires_at`
- `revoked_at`
- `last_used_at`
- `status`
- `notifications_enabled_snapshot` optional

#### Visit session / visit log

- `visit_id`
- `invite_id`
- `host_account_id`
- `visitor_email_encrypted`
- `started_at`
- `ended_at`
- `approx_duration_seconds`

### Modeling rules

- Bird identity is immutable and separate from species and name
- Personality vectors are persisted state, never derived on read
- Mood is persisted state and survives session boundaries
- Visitor activity never enters simulation inputs
- Account email is never used as an internal identifier outside encrypted storage and hashed lookup helpers

## 4. API surface

Use HTTP+JSON for v1, with server-sent events or lightweight polling reserved for future extension. The simulation cadence is slow enough that snapshot polling plus explicit refresh triggers is sufficient.

### Auth and account APIs

- `POST /v1/auth/magic-link/request`
- `POST /v1/auth/magic-link/consume`
- `POST /v1/auth/session/revoke`
- `GET /v1/account`
- `PATCH /v1/account/settings`
- `POST /v1/account/export`
- `POST /v1/account/delete`
- `POST /v1/account/delete/cancel`

### Aviary state APIs

- `GET /v1/aviary/snapshot`
  - Returns canonical scene snapshot plus `snapshot_version`, `generated_at`, and optional `next_refresh_hint_ms`
- `GET /v1/aviary/notebook?cursor=...`
- `POST /v1/aviary/events`
  - Accepts batched interaction events from the authenticated client
- `POST /v1/aviary/presence`
  - Specialized endpoint for frequent presence pings if event batching proves too noisy

### Social / visit APIs

- `POST /v1/visits/invites`
- `GET /v1/visits/invites`
- `POST /v1/visits/invites/:invite_id/revoke`
- `GET /v1/visits/log`
- `GET /v1/visit/:token/snapshot`
  - Visitor-only read-only aviary snapshot

### Accessibility APIs

Accessibility can be generated from the main snapshot for v1 rather than a separate endpoint, but keep room for:

- `GET /v1/aviary/narration`

only if screen-reader timing or queue control requires independent fetching.

### Event contract design

Clients should send semantic events, never calculated deltas. Example payloads:

```json
{
  "events": [
    {
      "event_type": "listen_in_started",
      "bird_id": "uuid",
      "occurred_at": "2026-05-30T12:00:00Z",
      "event_payload": {
        "input_mode": "pointer"
      }
    }
  ]
}
```

Presence ping example:

```json
{
  "event_type": "presence_ping",
  "occurred_at": "2026-05-30T12:03:00Z",
  "event_payload": {
    "visible": true,
    "focused": true,
    "recent_input_within_seconds": 120
  }
}
```

The server validates qualifying presence windows; it does not trust the client to precompute presence-time totals.

### Snapshot refresh model

Clients refresh snapshot on:

- Initial navigation
- Visibility becoming visible
- Resuming after large wall-clock gap
- Completing meaningful interaction that should surface promptly
- Periodic keepalive, likely 30-60 seconds while visible

Snapshots include a version number. If the client posts events against an older snapshot version, that is acceptable because the event log is append-only; versioning is for freshness, not mutation arbitration.

## 5. Simulation engine design

### Tick cadence

Target tick cadence: once per minute, with a processing SLA far below cadence and an alert threshold at p99 > 5s. The cadence is slow enough to feel ambient and cheap enough to run for all accounts regardless of active clients.

### Tick stages

1. Load accounts/aviaries due for tick
2. Pull unprocessed interaction events since previous tick
3. Update qualified presence windows from presence pings and session transitions
4. Compute per-bird drift deltas
5. Compute mood transitions
6. Compute weather/event transitions
7. Compute perch-zone and pose-family changes
8. Advance call scheduling seeds and greeting eligibility
9. Generate notebook entry candidates and persist sparse accepted entries
10. Advance aviary age unlock schedule if due
11. Persist new canonical state and mark events processed atomically

### Drift function

Drift must be monotonic toward expressive. That implies:

- Presence-time can increase traits; absence cannot decrease them
- Listen-in amplifies social warmth and vocal frequency for the focused bird
- Offers can nudge curiosity and boldness upward modestly
- Settle does not alter personality directly beyond ending qualifying presence

Implementation recommendation:

- Maintain each trait in normalized `[0, 1]`
- Apply low-pass additive deltas with diminishing returns near upper bounds
- Weight presence most heavily, then listen-in, then offers
- Use rolling time windows across days so one long session does not produce visible overnight change

Concrete behavior target:

- Measurable instrumentation-level movement after ~1 week of regular visits
- Human-noticeable change after ~3 weeks
- No visible trait jumps within a single session

Suggested drift formula family:

- For each bird trait, compute `delta = gain * signal_strength * headroom`
- `headroom = max(0, 1 - current_trait)` to enforce asymptotic saturation
- Smooth via capped per-tick accumulation to prevent bursty event clusters from spiking

This formula family preserves monotonicity, headroom, and slow expressivity without needing downward corrections.

### Presence accounting

Presence is the dominant drift input and must be processed as qualified intervals.

Client emits pings every 60-120 seconds while the page is visible, including current visibility, focus, and recent-input window. The server constructs presence windows only where all three are true. The recent-input threshold should be tuned during build, likely 2-5 minutes, with instrumentation to compare false-drop and false-keep behavior.

Important design choice:

- Presence is accumulated per account session but applied to birds at tick time, allowing bird-specific weighting later without redefining presence itself

### Mood transitions

Mood is a fast-timescale state machine with persistence across sessions. Recommended v1 mood set:

- wary
- content
- curious
- drowsy
- alert
- settled

Transition inputs:

- Local time of day
- Current weather
- Recent bird-specific interactions
- Spillover from nearby bird calls/alarm states
- Bird personality modifiers

Design rules:

- High boldness lowers entry likelihood into wary
- High curiosity raises responses to offers and ambient novelty
- Evening pulls toward drowsy or settled
- Rain dampens vocal frequency and can bias some birds toward quiet/content while others skew alert/wary

Persist `mood_entered_at` so transitions can respect dwell time and avoid rapid oscillation.

### Return greeting runtime

Greeting selection should be precomputed from canonical state at session-start refresh, not improvised entirely in the client.

The server should provide:

- which bird is primary greeter
- optional staggered secondary responders
- greeting style family based on absence length, boldness, and mood
- timing offsets and motion/audio seeds

This keeps the "noticed, never announced" moment coherent across devices and tied to true absence duration.

### Call grammar runtime

Each species has a motif library. Each bird has a stable `call_signature_seed` that selects and biases motifs so the bird remains recognizable over time.

Server-owned:

- Motif family assignment
- Call tempo window
- Chorus join propensity
- Mood and trait modifiers

Client-owned:

- WebAudio synthesis from motif instructions
- Mix balancing for ambient vs listen-in
- Envelope shaping and local playback scheduling

Represent calls as procedural instructions rather than audio assets:

- motif id
- pitch contour seed
- envelope type
- duration family
- pause spacing
- volume bias

### Bird-to-bird interaction

Birds are not independent. The tick should compute lightweight social coupling:

- Chorus probability rises when multiple high-vocal-frequency birds are in compatible moods
- Wary states can softly propagate to nearby birds
- Social warmth influences response likelihood to another bird's call

This should remain stochastic and sparse. The goal is a small social system, not a crowded rules engine.

### Notebook generation

Notebook entries should be generated from notable state patterns, not every event. Build an observation-ranking pass after tick updates:

- first greeting change after long streak of another bird greeting first
- unusual quiet stretch
- weather plus posture combination
- repeated perch preference shift
- offer reaction that reveals mood/personality

Apply sparsity rules:

- Base cadence around one entry every few days for regular users
- Permit higher density only for genuinely notable moments
- Never mirror raw telemetry language

Use templated naturalist grammar with variation slots rather than LLM generation in the live path. This gives deterministic tone control, low latency, and no privacy spill into model inference.

## 6. Sync model

### Canonical state model

The sync model is intentionally simple:

- one canonical aviary state per account
- only server tick mutates durable bird state
- clients only read snapshots and append events

This eliminates client reconciliation of personality and mood.

### Conflict prevention

Prevent conflicts structurally rather than resolving them cosmetically.

- No client-submitted absolute personality values
- No last-write-wins for bird state
- Event ingestion is append-only and idempotent by event id
- Tick processing transactionally marks which events were consumed
- Snapshot versions are monotonic for freshness tracking

### Multi-device behavior

If phone and laptop are both open:

- both post presence and interaction events tied to their own session ids
- both read the same canonical snapshots
- whichever device interacts contributes events; only authenticated account owner sessions count toward presence
- if simultaneous visible sessions exist, treat both as legitimate presence but cap aggregate presence accumulation per wall-clock minute to avoid double-counting attention

That cap is important. A user should not earn 2x drift by leaving the same aviary open on laptop and phone simultaneously.

### Offline and degraded handling

If the client loses connectivity:

- freeze to local ambient render for a short grace period
- queue interaction events locally with timestamps
- on reconnect, submit queued events if still within acceptable age window

Guardrail:

- queued events may inform recent listen-in/offer activity, but offline mode must not simulate durable bird state locally
- if offline duration is long, discard stale queued events that would misrepresent context and refresh from canonical state

### Session timeout and magic-link edge cases

Errors here should be matter-of-fact. Do not attempt to preserve naturalist tone in auth and sync failure surfaces. Engineering should centralize account/system voice separately from product-surface voice to prevent accidental copy drift.

## 7. Frontend rendering pipeline

### Rendering stack

Use a browser-native rendering stack that balances motion richness and bundle limits. Recommended approach:

- DOM/CSS/SVG or Canvas 2D hybrid for scene composition
- WebAudio for calls
- Minimal asset pipeline with species illustrations as compact vectors or sprite atlases

Avoid heavyweight 3D or WebGL-first choices in v1 unless profiling proves they are necessary. The scene complexity does not justify the bundle and compatibility cost.

### Scene composition

Render layers:

1. Sky and distant foliage
2. Back perch zone
3. Middle perch zone
4. Front perch zone
5. Foreground leaves/branches
6. Top bar chrome
7. Accessibility overlays like call captions and focus indicators

Bird layout should preserve visibility on all supported viewports and never crop birds offscreen.

### First-load experience

The client should request the initial snapshot as part of the first HTML/bootstrap response when possible. If a full snapshot cannot arrive within the first render budget, show the quiet field rather than a spinner. The first bird should appear already in pose/motion state, not via boot-up animation.

### Motion system

Normal mode:

- idle micro-motion loops with seeded variance
- perch repositioning via slow arcs or hops
- subtle ambient parallax
- weather overlays and leaf/feather drift

Reduced-motion mode:

- pose-to-pose cross-fades instead of continuous micro-animation
- no drifting leaf ornaments
- slower palette transitions preserved
- same canonical bird state and audio/caption behavior

### Listen-in transitions

Audio mix and visual focus should ease in and out over several hundred milliseconds to feel like attentive listening rather than channel switching. Other birds remain audible/visible, only softened.

### Top bar behavior

- sparse icon set only
- fades nearly transparent after inactivity
- restores on pointer or keyboard activity
- keyboard reachable with visible focus treatment in all light states

### Field notebook UI

- read-only
- chronological reverse order with infinite scroll or cursor pagination
- no badges, no unread counts, no "new entry!" celebration

## 8. Audio pipeline

### WebAudio-first synthesis

Implement procedural synthesis entirely client-side from motif instructions supplied by the snapshot. This keeps bundle weight low and maintains per-call variation.

Components:

- motif scheduler
- per-bird synth voice
- chorus mixer
- ambience bus
- listen-in focus bus
- caption generator linked to played motif instructions

### Recognizable per-bird identity

Each bird needs stable auditory identity. Achieve this by combining:

- stable motif family and signature seed per bird
- limited pitch/timing variation within species envelope
- mood-based modulation that changes expression without erasing identity

### Chorus mixing

- preserve per-bird recognizability
- avoid clipping and spectral crowding
- constrain simultaneous high-energy calls, especially near bird-count ceiling

At higher bird counts, the scheduler should explicitly thin overlap windows to preserve intelligibility.

### Listen-in mix

- focused bird gains presence and slight clarity boost
- others drop to ambient, never mute
- disengage restores ambient mix gradually

### Fallback behavior

If WebAudio is unavailable or blocked:

- switch to silent mode
- enable captions by default
- keep all interaction and rendering intact

Do not add recorded audio fallback.

### Audio testing

- determinism tests for motif instruction generation
- browser compatibility tests for AudioContext startup and resume
- long-session leak tests for audio buffers and nodes
- perceptual QA sessions to verify identity recognizability and uncanniness thresholds

## 9. Accessibility surfaces

### Screen-reader narration

Provide a live-region narration layer fed by canonical scene state and user-triggered events. Narration must be slow, sparse, and prose-first.

Implementation approach:

- generate narration segments from snapshot diffs plus cadence rules
- idle updates every 30-60 seconds
- priority bump for greeting, successful offer reaction, settle
- deduplicate repetitive state to avoid queue spam

Use handcrafted template grammar in naturalist voice, not raw state dumps.

### Keyboard model

- Tab cycles top bar and aviary entry
- Arrow keys move bird focus
- Enter toggles listen-in on focused bird
- Escape exits listen-in and dismisses transient panels
- Offer and settle reachable without pointer

### Call captions

Generate caption text from actual procedural output. Caption anchoring near the bird should avoid obscuring the scene and respect contrast requirements.

### Reduced motion and audio-off users

- reduced-motion is a designed mode, not disabled animation
- captions support audio-off and hearing-difference cases
- narration and notebook should continue to communicate aliveness even without audio

### Contrast and focus

- WCAG AA minimum for copy-bearing surfaces
- visible focus rings in all palette/time-of-day states
- test dusk/night states specifically, since calm palettes can conceal focus cues

### Accessibility QA gates

- screen-reader pass on VoiceOver, NVDA, and at least one browser pairing per platform target
- keyboard-only full interaction pass
- reduced-motion visual QA as a first-class design review
- caption correctness spot checks against actual generated calls

## 10. Performance budgets and observability

### Budgets

- Initial JS bundle under 2MB gzipped
- First bird visible under 500ms on mid-tier mobile over 4G
- Idle motion at 60fps on 5-year-old mid-range laptop
- No measurable memory growth across a 30-minute session
- Tick latency p99 under 5s alarm threshold

### Engineering implications

- code-split settings, account, and invite management flows
- keep initial snapshot small and cache-friendly
- precompute scene directives server-side to reduce client boot work
- reuse audio nodes/buffers aggressively
- cap ornament complexity and simultaneous call density

### Observability

Collect only aggregate health metrics:

- page load timing
- first-bird render timing
- render frame timing
- memory usage samples
- audio-context failures
- tick latency
- request counts and error rates

Do not collect per-account bird state, per-bird drift history for analytics, or warehouse copies of simulation records.

### Debugging strategy

Use internal admin/debug tooling that accesses a specific account only under authenticated operational workflows, rather than mirroring simulation data into analytics systems. This preserves privacy boundaries while keeping incidents diagnosable.

## 11. Security and privacy implementation

### Privacy boundary

Make the PRD's privacy stance enforceable in code and infra:

- simulation database is separate from analytics sink access
- telemetry schemas ban bird ids, notebook text, personality values, and per-account interaction histories
- internal ids use synthetic UUIDs only
- emails stored encrypted and surfaced only where operationally required

### Visit privacy

- invites are per-email and revocable
- visitor activity is logged for transparency but excluded from drift inputs
- revoked/expired links fail closed
- hosts get optional notifications only if explicitly enabled

### Account lifecycle

- 15-minute magic-link expiry and one-time consumption
- soft delete for 30 days, then hard delete cascade
- export delivered via verified-email link

## 12. Rollout plan

### Phase A: internal prototype

- implement auth, canonical aviary, two birds, snapshot rendering, basic tick, and presence accounting
- no visits yet
- use internal QA accounts to validate first-bird timing, greeting feel, and sync continuity

Exit criteria:

- server-canonical simulation works across laptop/phone
- presence accounting behaves honestly under common focus/visibility edge cases
- no canned-feeling loops in greeting or calls

### Phase B: private alpha

- enable notebook, offers, settle, reduced motion, captions, and screen-reader narration
- introduce sparse weather and bird-to-bird interaction
- add internal calibration dashboards that expose aggregate, not per-user, drift-rate distributions

Exit criteria:

- drift calibration hits one-week measurable / three-week noticeable target in qualitative and instrumented review
- accessibility modes reviewed as first-class product surfaces
- memory and audio leak tests pass

### Phase C: invite-only beta

- enable quiet visit invites and host visit log
- ramp max birds conservatively:
  - start with two starter birds only
  - enable third-bird age unlock for selected accounts
  - later expand toward higher caps after audio recognizability review

Exit criteria:

- read-only visitor mode does not affect host simulation
- invitation revoke/expire flows are reliable
- higher bird counts do not collapse call intelligibility or render performance

### Phase D: general availability v1

- launch with two starter birds and staged age-based expansion
- keep notifications off by default, including visit notifications
- ship operational alarms and privacy audits from day one

### Day-one instrumentation

- auth success/failure rates
- snapshot latency and first-bird render timing
- tick latency and backlog depth
- presence qualification rate by client platform in aggregate
- audio pipeline failure rate
- accessibility mode adoption counts in aggregate

## 13. Risks and mitigations

### Drift calibration risk

Risk:

- Birds change too quickly and feel game-like, or too slowly and feel static

Mitigation:

- build calibration harnesses with synthetic presence/interaction traces
- review weekly trend curves before broad launch
- keep coefficients configurable server-side without schema changes

### Presence honesty risk

Risk:

- Background tabs or dual-device sessions inflate drift

Mitigation:

- enforce conjunction rule server-side
- cap simultaneous-session accumulation per wall-clock minute
- audit presence qualification metrics for suspicious always-on patterns

### Sync correctness risk

Risk:

- Lost drift due to race conditions, duplicate events, or stale client state assumptions

Mitigation:

- append-only idempotent event ingestion
- transactional tick processing
- no client-authored durable state
- snapshot versioning for diagnostics

### Audio uncanniness risk

Risk:

- Calls sound canned, repetitive, or synthetic in the wrong way

Mitigation:

- stable motif families with runtime variation
- perceptual QA across repeated listening sessions
- constrain overlap and reuse
- prefer silence plus captions over low-quality fallback audio

### Accessibility regression risk

Risk:

- Accessible modes lag behind visual product and become stripped-down alternatives

Mitigation:

- treat narration/reduced-motion/captions as launch-blocking scope
- include accessibility reviews in product design sign-off, not only QA
- add regression tests for keyboard and reduced-motion paths

### Performance regression risk

Risk:

- Bundle bloat or render complexity pushes first-bird timing above the affective threshold

Mitigation:

- enforce bundle budget in CI
- synthetic first-bird timing tests
- strict asset review for new species and motion features

### Privacy boundary erosion risk

Risk:

- Convenience logging or analytics gradually absorbs simulation data

Mitigation:

- schema linting for telemetry events
- simulation DB access controls separate from analytics roles
- privacy review required for any new metric touching account-scoped behavior

## 14. Recommended implementation order for the engineering team

1. Establish schema, auth, session, and canonical aviary snapshot pipeline.
2. Implement birds, personality persistence, mood persistence, and simulation tick skeleton.
3. Add presence qualification and drift accumulation.
4. Add render pipeline with already-in-progress first frame and idle motion.
5. Add procedural call synthesis and listen-in mix.
6. Add offers, settle, and notebook generation.
7. Add accessibility surfaces in parallel with core interactions, not after.
8. Add quiet visit invitations and read-only visitor snapshots.
9. Harden telemetry, privacy controls, deletion/export, and operational alarms.
10. Run calibration, long-session, and cross-device QA before expanding bird count beyond the starter state.

## 15. Open implementation decisions to resolve during build

These are acceptable ambiguities to settle in implementation without changing the product:

- Exact recent-input threshold for presence qualification
- Final mood enum membership and transition coefficients
- Exact bird unlock schedule by aviary age
- Precise rendering technology split between SVG, DOM, and Canvas
- Exact template library for notebook and narration phrasing

The team should resolve these through calibration and prototyping while preserving the non-negotiable constraints above.
