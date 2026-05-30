# Pocket Aviary v1 Implementation Plan

## Planning frame

This plan translates the PRD into an executable v1 delivery blueprint for a separate engineering team. It intentionally makes a few architecture calls where the PRD is directional but not prescriptive:

1. The system will use a server-authoritative snapshot plus append-only event-log model, not peer sync or client-owned simulation state.
2. Permanent personality drift and temporary "quiet after absence" behavior will be modeled separately so we preserve monotonic long-term drift while still letting absent birds feel more ambient on return.
3. Notebook prose, screen-reader narration, and call captions will be generated from a shared deterministic prose system rather than an online LLM dependency, to protect privacy, consistency, latency, and tone.
4. Multi-device freshness will use revisioned snapshot polling and visibility-triggered refresh rather than a continuously open realtime channel, because the canonical simulation cadence is slow and read-heavy.

## V1 scope

### In scope

- Web-only product for modern browsers.
- Single-user accounts with magic-link sign-in.
- One canonical aviary per account.
- Two starter birds at account creation, with age-based unlock support up to seven total birds.
- Server-side simulation tick that advances bird mood and personality whether or not the client is open.
- Core interactions: return-greeting, listen-in, offer, settle, field notebook.
- Presence accounting using the PRD's three-signal conjunction.
- Multi-device sync through one server-owned canonical state.
- Accessibility surfaces from day one: screen-reader narration, reduced-motion mode, call captions, keyboard navigation, WCAG AA contrast for copy surfaces.
- Quiet social feature: per-invite read-only visits, revocation, visit log, optional visit notifications off by default.
- Operational telemetry limited to aggregate health and performance metrics.

### Explicitly out of scope

- Native iOS or Android clients.
- Shared aviaries, multi-user simulation, co-presence, or collaborative visits.
- Gamification of any kind: streaks, counters, achievements, levels, leaderboards, profiles, discovery feeds.
- Tamagotchi mechanics: hunger, decay, visible distress, negative drift on absence.
- Public discovery, comments, chat, avatars, or visitor interaction with host birds.
- Offline-first simulation or durable offline replay. If connectivity drops, the app may continue rendering the last known state briefly, but presence accrual and authoritative interactions pause until the server connection is restored.

## Product behavior decisions that unblock implementation

### Permanent drift vs ambient quietness

The PRD requires two truths at once:

- personality drift only moves toward more expressive states and never reverses on neglect
- a user returning after absence should find birds quieter and more ambient

To satisfy both, the engine will maintain two separate layers:

- `personality_vector`: permanent, monotonic, slow-moving trait values
- `recent_presence_reservoir`: temporary, decaying expressivity fuel derived from recent host presence and recent meaningful interaction

Greeting likelihood, chorus density, notebook frequency, and near-term vocal activity will depend on both layers. A bird can become permanently bolder and warmer over months while still sounding quieter after two weeks away because its temporary presence reservoir has decayed.

### Text generation approach

The notebook, narration, and captions should feel authored, but the privacy and latency constraints argue against per-request model inference. Use a deterministic prose compiler fed by structured state summaries, event patterns, time-of-day context, and species-specific phrasing tables. This preserves:

- consistent naturalist voice
- low latency on every client surface
- no raw per-bird interaction history leaving the simulation boundary
- fully testable copy generation

### Sync model choice

Use HTTP snapshot polling with revision IDs, ETags, and visibility-triggered refreshes. Do not use websockets for v1. The tick cadence is about once per minute, state updates are low-volume, and the read-only visit flow already tolerates "next snapshot pull" semantics. Polling is simpler, easier to harden, easier to cache, and less likely to create invisible realtime race conditions around session pause/resume.

## Recommended system architecture

### High-level shape

Use a TypeScript monorepo with five production units:

1. `web-app`: React/Next.js application serving the aviary shell, settings, notebook, invite flow, and magic-link surfaces.
2. `api-service`: authenticated HTTP API for snapshots, events, account settings, exports, invite management, and visit bootstrap.
3. `simulation-worker`: background worker fleet that owns the canonical tick, notebook generation, weather scheduling, bird unlock scheduling, and deletion processing.
4. `shared-domain`: shared type contracts, state schemas, phrase templates, and validation rules used by both API and client.
5. `email-worker`: outbound mail delivery for magic links, invites, and export links.

### Infrastructure

- Postgres as the system of record for accounts, birds, canonical aviary state, event log, notebook entries, invites, sessions, and audit data.
- Redis or a managed queue for due-tick scheduling, outbound email jobs, and export jobs.
- CDN/edge caching for static assets, initial HTML, JS bundles, and immutable species visual/audio motif data.
- Object storage for generated export payloads and optional internal build artifacts.
- One primary region for canonical data writes. Read caching at the edge is allowed for anonymous assets only; authoritative aviary state stays region-local to avoid consistency ambiguity.

### Client/server split

Server responsibilities:

- canonical personality state
- canonical mood state
- event ingestion and ordering
- simulation tick and conflict prevention
- notebook entry creation
- visit authorization and revocation
- account lifecycle and privacy workflows

Client responsibilities:

- rendering the current scene from snapshots
- smooth interpolation between snapshots
- local-only ambient ornaments such as drifting leaves and feathers
- WebAudio synthesis and mixing
- screen-reader queue playback from structured state
- accessibility UI, keyboard handling, and top-bar behavior

### Render pipeline boundary

Do not build the aviary scene as ordinary DOM animation. Split the frontend into:

- React shell for chrome, settings, notebook, auth, accessibility controls, and visit UI
- dedicated scene renderer for the aviary itself, using canvas or WebGL-backed 2D rendering
- dedicated audio engine using WebAudio plus AudioWorklet for deterministic scheduling

This keeps the top bar and accessibility surfaces straightforward while protecting runtime performance for continuous animation and audio.

## Core data model

### Account and identity

`accounts`

- `account_id` UUID primary key
- `email_ciphertext`
- `email_hash` for lookup and rate limiting
- `created_at`
- `status` enum: active, pending_deletion
- `pending_delete_at`
- `timezone`
- `default_accessibility_prefs`
- `visit_notifications_enabled`

`sessions`

- `session_id` UUID
- `account_id`
- `device_label`
- `issued_at`
- `expires_at`
- `revoked_at`
- `last_seen_at`

`magic_links`

- `token_hash`
- `account_id` or pending email
- `expires_at`
- `consumed_at`
- `request_ip_hash`

### Aviary and birds

`aviaries`

- `aviary_id` UUID
- `account_id`
- `created_at`
- `state_revision` bigint
- `last_tick_at`
- `current_weather_state`
- `current_day_phase`
- `recent_presence_reservoir`
- `last_presence_end_at`
- `settled_until`
- `next_bird_unlock_at`
- `bird_cap` default 7

`birds`

- `bird_id` UUID
- `aviary_id`
- `species_id`
- `display_name`
- `adopted_at`
- `position_zone` enum: front, middle, back
- `pose_state`
- `motion_seed`
- `call_seed`
- `is_active`

`bird_personality_vectors`

- `bird_id`
- `boldness`
- `social_warmth`
- `vocal_frequency`
- `plumage_saturation`
- `curiosity`
- `last_drift_at`

`bird_runtime_state`

- `bird_id`
- `mood_state`
- `mood_intensity`
- `mood_started_at`
- `call_cadence_phase`
- `current_transition`
- `greeting_weight`
- `last_offer_reaction_at`
- `offer_cooldown_until`
- `last_listen_in_at`

### Events and notebook

`interaction_events`

- `event_id` UUID
- `aviary_id`
- `bird_id` nullable
- `account_id`
- `session_id`
- `event_type`
- `event_payload` JSONB
- `client_event_id`
- `client_occurred_at`
- `server_received_at`
- `processed_in_tick_at` nullable

Allowed event types:

- `presence_ping`
- `presence_end`
- `listen_in_started`
- `listen_in_ended`
- `offer_seed`
- `offer_song_fragment`
- `offer_still_pool`
- `settle_started`
- `settle_reversed`

`notebook_entries`

- `entry_id` UUID
- `aviary_id`
- `entry_date`
- `headline_fragment`
- `body_text`
- `source_tick_at`
- `source_signals` JSONB

The `source_signals` field exists for explainability and testing only. It is not surfaced to users and it is not exported into aggregate telemetry.

### Social and visits

`visit_invites`

- `invite_id` UUID
- `host_account_id`
- `visitor_email_ciphertext`
- `visitor_email_hash`
- `invite_token_hash`
- `status` enum: pending, active, revoked, expired
- `created_at`
- `expires_at`
- `revoked_at`

`visit_sessions`

- `visit_session_id`
- `invite_id`
- `host_aviary_id`
- `started_at`
- `ended_at`
- `last_snapshot_at`

`visit_logs`

- `log_id`
- `host_account_id`
- `invite_id`
- `visitor_email_ciphertext`
- `started_at`
- `ended_at`
- `approx_duration_seconds`

### Privacy and lifecycle

`export_jobs`

- `export_job_id`
- `account_id`
- `requested_at`
- `completed_at`
- `download_token_hash`
- `expires_at`

`deletion_jobs`

- `account_id`
- `scheduled_hard_delete_at`
- `recovered_at`
- `completed_at`

### Static domain assets

Store species definitions, motif libraries, pose banks, and phrase tables as versioned code or immutable content assets, not mutable database rows. Each definition bundle gets a semantic version so snapshots can reference which species asset version the client should render against.

## API surface

All write endpoints require idempotency keys. All state-bearing read endpoints return a monotonic `state_revision`.

### Auth and account

`POST /api/auth/request-magic-link`

- input: email
- output: success envelope only
- behavior: create or reuse pending account, rate-limit by hashed email, enqueue email

`POST /api/auth/consume-magic-link`

- input: token
- output: session token, account summary

`GET /api/account`

- returns account settings, active sessions, visit preferences, deletion/export status

`POST /api/account/email-change`

- starts verified email replacement flow

`POST /api/account/export`

- enqueue JSON export and email delivery

`POST /api/account/delete`

- mark pending deletion and terminate future background unlocks while preserving recovery window

`POST /api/account/delete/cancel`

- recover soft-deleted account inside 30-day window

### Aviary bootstrap and snapshots

`GET /api/aviary/bootstrap`

- returns account summary, accessibility prefs, top-bar capabilities, notebook unread hint if any, and current aviary snapshot
- snapshot includes:
  - `state_revision`
  - `server_time`
  - `day_phase`
  - `weather_state`
  - `settled_state`
  - `recent_presence_reservoir`
  - per-bird render and audio seeds
  - per-bird mood and pose
  - `return_greeting_plan`
  - optional `notebook_highlight`

`GET /api/aviary/snapshot?since_revision=...`

- returns 304 if unchanged, otherwise the newest snapshot
- called on visibility regain, after acknowledged interaction, after long frame gaps, and on low-frequency keepalive

### Events

`POST /api/aviary/events:batch`

- accepts a bounded batch of interaction events
- each event includes `client_event_id`, `session_id`, `bird_id` if relevant, payload, and client timestamp
- server assigns receive time and enqueues for the next tick

Rules:

- presence pings older than a short threshold are dropped rather than replayed, to keep presence honest
- non-presence interaction events may be accepted after brief retry windows if the session is still valid
- clients never submit personality values, mood values, or absolute bird positions

### Notebook

`GET /api/notebook?cursor=...`

- paginated read-only notebook entries
- no mutation endpoint exists

### Visits

`POST /api/visits/invites`

- input: visitor email
- output: invite record and expiry
- side effects: enqueue email with one-time link

`GET /api/visits/invites`

- returns outstanding and historical invites for host settings

`POST /api/visits/invites/{invite_id}/revoke`

- immediately revokes the invite

`GET /api/visit/{token}/bootstrap`

- visitor bootstrap endpoint
- returns read-only snapshot plus host-display metadata allowed by policy
- no event-write routes are exposed from the visitor surface

`GET /api/visit/{token}/snapshot?since_revision=...`

- read-only polling endpoint
- revoked or expired tokens return the matter-of-fact unavailable surface payload

## Simulation engine design

### Tick scheduler

Run a due-aviary scheduler every minute. Each aviary stores `last_tick_at`; the worker claims due aviaries in batches and performs idempotent tick jobs keyed by `(aviary_id, tick_window_end)`.

If workers fall behind, the system performs catch-up by iterating missed tick windows in order, bounded to a maximum catch-up batch per run so one stalled aviary does not starve the queue.

### Per-tick phases

1. Load canonical aviary state plus unprocessed events since `last_tick_at`.
2. Aggregate events into per-bird and aviary-level influence signals.
3. Advance the temporary presence reservoir based on recent host presence and reservoir decay.
4. Compute mood transitions per bird from time of day, weather, recent interaction, bird-to-bird contagion, and personality modifiers.
5. Compute additive personality deltas for each bird.
6. Advance runtime state: perch preference, pose target, call cadence phase, offer cooldowns, settled state.
7. Decide whether a notebook-worthy observation occurred.
8. Persist the new canonical state and increment `state_revision`.

### Presence accounting implementation

The client emits periodic `presence_ping` events only while:

- document is visible
- window is focused
- recent pointer or keyboard activity falls inside the calibrated grace window

The server does not trust client-declared continuous duration. It converts accepted pings into bounded presence windows. Suggested implementation:

- client sends a ping every 30 seconds while present
- server credits up to 30 seconds of presence for each valid ping
- reservoir fill is derived from server-accepted windows, not raw client durations

This design keeps presence honest, survives packet loss, and prevents clients from retroactively claiming hours of idle attention.

### Drift function

Personality updates are small, saturating, additive deltas:

- `boldness`: increases from repeated proximity offers and consistent host presence
- `social_warmth`: increases from listen-in, greeting participation, and repeated positive sessions
- `vocal_frequency`: increases from sustained attention and voluntary call joining
- `plumage_saturation`: increases primarily from cumulative presence over time
- `curiosity`: increases from approach-and-engage reactions to offers

Rules:

- traits never decrement due to absence
- per-session effect is capped
- diminishing returns apply at higher values
- one week of regular visits should produce measurable but not user-visible change
- about three weeks of regular visits should produce felt change

Build an offline calibration harness before public release. Feed synthetic session patterns into the engine and verify the one-week and three-week thresholds against explicit goldens.

### Temporary expressivity reservoir

The reservoir decays on a rolling basis and influences:

- return-greeting probability and intensity
- call density
- likelihood of chorus events
- notebook entry frequency
- near-front perch preference among already-bold birds

This is how the product gets "quieter after absence" without violating monotonic permanent drift.

### Mood system

Represent mood as an enum plus intensity:

- wary
- content
- curious
- drowsy
- alert
- settled

Mood inputs:

- local time-of-day
- active weather
- recent offer outcome
- listen-in recency
- nearby bird contagion effects
- bird-specific personality modifiers

Mood persistence rules:

- mood does not reset on tab open
- session end preserves mood
- time-of-day transitions can soften or intensify mood between sessions

### Greeting selection

Do not precompute return greetings on the tick. Compute them on session bootstrap so the system can incorporate true absence length and the most recent state.

Greeting selection algorithm:

1. identify candidate birds weighted by boldness, social warmth, current mood, and reservoir level
2. suppress simultaneous greetings by selecting one primary greeter and optional staggered secondary follow-up
3. synthesize a greeting plan with motion cue, call cue, and timing offsets over the first 1 to 2 seconds
4. include the greeting plan in bootstrap payload so the client can render it immediately

### Offers

Offers are modeled as authored interaction types, not free-form inputs. Each offer produces:

- immediate reaction selection based on mood and curiosity
- short-lived runtime changes such as approaching a perch or pausing a call
- small drift contribution if accepted or investigated
- cooldown enforcement per bird

### Weather

Weather is an aviary-level ambient state scheduled by the simulation worker a few times per week. Use a sparse scheduler, not continuous meteorological simulation. Weather exists to lightly perturb mood and ambience, not to become a feature system.

### Notebook generation

Notebook creation should be sparse and observation-like. Use trigger classes such as:

- unusual greeting order
- long quiet stretch
- distinctive weather plus pose combination
- first approach to an offer after prior reluctance
- unusual chorus behavior

Rules:

- average cadence: every few days for regularly visited aviaries
- no notebook entry per session
- prose is specific and observational
- never mention numerical trait changes or visit-count behavior

## Sync model and conflict prevention

### State ownership

- server is the sole writer of personality and mood state
- clients write only interaction events
- visitors have no write path into host state

### Revisioning

Each canonical update increments `state_revision`. Clients store the last seen revision and request deltas or fresh snapshots from that point. If the server cannot cheaply provide a delta, it may return a full compact snapshot with the new revision.

### Polling cadence

Client refreshes occur:

- on initial load
- on document visibility regain
- after suspend/resume or large render-clock gap
- every 15 to 30 seconds while visible
- immediately after successful interaction write acknowledgements

This is enough to keep state coherent while respecting the slow tick cadence.

### Idempotency and ordering

Every client event includes:

- `client_event_id`
- `session_id`
- `event_type`
- `client_occurred_at`

Server behavior:

- dedupe retries on `(session_id, client_event_id)`
- order processing by `server_received_at`, using `client_occurred_at` only as supporting context
- reject stale session writes

### Offline and degraded network behavior

V1 does not promise durable offline interaction recording. If connectivity is briefly interrupted:

- the scene may continue animating from the last snapshot
- listen-in mix changes may continue locally until reset
- presence is not credited until the server receives fresh valid pings
- stale pings are dropped instead of replayed

This preserves integrity of drift over convenience.

### Visit consistency

Visitors poll the host snapshot using a dedicated read-only token. Revocation or expiration takes effect on the next polling response. Visitor presence does not emit host interaction events and does not enter the host simulation ledger.

## Frontend rendering pipeline

### Application shell

Use server-rendered HTML for the shell and account surfaces, with fast hydration for:

- top bar
- notebook drawer
- offer affordance
- accessibility settings
- account/settings pages

### Initial load behavior

Target first bird visibility inside 500ms by:

- embedding or edge-delivering the initial bootstrap snapshot with the HTML
- code-splitting non-essential settings surfaces
- rendering the aviary as soon as the first scene payload is ready

If the snapshot is late, show the quiet field loading surface. Never show a spinner.

### Scene graph

Structure the scene renderer into stable layers:

1. background sky and distant foliage
2. perch and branch planes
3. bird sprites or vector bodies
4. optional foreground leaf or branch pass
5. caption and focus overlay layer

Birds are always kept fully on-screen. Responsive rules widen spacing on desktop and compress spacing on mobile without cropping.

### Bird rendering

Each bird render packet should include:

- species identifier and asset version
- current perch zone
- current pose family
- motion seed
- call seed
- target transition or idle behavior weights

The client uses this packet to synthesize alive-feeling micro-motion between snapshots rather than waiting for the server to stream every visual step.

### Reduced-motion path

Reduced motion uses the same state packet but swaps the animator:

- cross-fade between still poses
- cross-fade perch transitions rather than animating flights
- remove leaf and feather drift
- keep day/night color shifts but slow them

No separate simulation path exists for reduced motion.

### Top bar and affordances

Top bar items:

- account/settings
- accessibility
- notebook
- offer
- settle

Behavior:

- fade nearly transparent after inactivity
- reappear on pointer or keyboard activity
- remain keyboard accessible even while visually faded

### Field notebook UI

Notebook opens in a side panel or modal sheet depending on viewport. It is read-only, paginated, and fully keyboard navigable. Entries use naturalist formatting and preserve lowercase style.

## Audio pipeline

### Synthesis architecture

Use WebAudio plus AudioWorklet for per-bird procedural call synthesis. Each species ships with:

- motif library
- timing rules
- pitch contour rules
- timbral parameters
- caption descriptor templates

Each bird instance applies its own seeds and personality modifiers on top of species defaults.

### Scheduling

Maintain a rolling short-horizon scheduler, around 2 to 4 seconds ahead, driven by:

- current bird mood
- vocal frequency trait
- reservoir level
- listen-in focus state
- chorus opportunities from nearby bird call windows

This gives smooth playback without storing long audio buffers.

### Listen-in mix

Listen-in changes only the mix, not the simulation. Implement with smooth gain ramps:

- focused bird rises gradually
- other birds reduce to ambient, never mute fully
- disengage returns with the same ramp profile

### Call recognizability

Preserve per-bird identity by tying each bird to a persistent call seed and motif weighting profile. Drift may alter density and variation, but not erase recognizable signature contours.

### WebAudio fallback

If WebAudio is unavailable or blocked:

- run the visual aviary normally
- enable captions by default
- do not attempt recorded audio fallback

Track aggregate fallback rates as an operational metric.

## Accessibility surfaces

### Screen-reader narration

Implement a shared narration composer that converts structured state into slow naturalist prose updates. Delivery approach:

- one ARIA live region queue for low-priority ambient narration
- one higher-priority queue for user-triggered observations such as greeting or offer reaction

Cadence:

- idle narration every 30 to 60 seconds
- prompt but still observational narration for user-triggered events

Do not expose trait numbers, raw pose IDs, or debug state through accessibility labels.

### Captions

Caption text is generated from the same call grammar inputs used for synthesis. Position captions near the calling bird, with accessible color contrast and fade timing that matches the sound envelope.

### Keyboard support

Required flows:

- tab through top bar and modal surfaces
- enter aviary bird focus from keyboard
- arrow-key move between birds
- Enter or Space to listen in
- Escape to exit listen-in or dismiss transient surfaces
- keyboard navigation through offer menu and notebook

### Focus treatment

Use clearly visible focus rings that hold contrast against morning, dusk, and night palettes. Treat focus styling as a design deliverable, not a browser default afterthought.

### Matter-of-fact system surfaces

All auth, error, revocation, unsupported-browser, and settings-system copy must use the matter-of-fact voice. Do not reuse naturalist phrasing there.

## Performance budgets and observability

### Enforced budgets

- initial JS bundle under 2MB gzipped
- first bird visible under 500ms on mid-tier mobile over 4G
- 60fps idle motion on a five-year-old mid-range laptop
- no sustained memory growth over a 30-minute session
- simulation tick p99 under 5 seconds

### Build-time controls

- aggressive code-splitting for settings, invite management, and export/deletion flows
- immutable CDN caching for species assets
- bundle analysis gate in CI
- animation and audio stress test scenes in CI and nightly builds

### Runtime instrumentation

Allowed aggregate metrics:

- first-bird render timing
- bundle and asset load timing
- render-frame timing aggregates
- audio initialization failures
- WebAudio fallback rate
- API latency
- snapshot size distribution
- tick duration and queue lag
- visit revocation propagation latency

Not allowed:

- per-bird interaction history in analytics
- per-account drift dashboards
- population-level model training inputs from notebook or interaction logs
- any observability dimension keyed by email or human-readable identity

### Privacy boundary enforcement

Physically separate the simulation database from the aggregate analytics pipeline. Export only approved aggregate counters and latency histograms. Add automated redaction checks to ensure no event payload containing bird or account behavioral detail reaches analytics sinks.

## Security and privacy implementation details

- Use synthetic UUID account IDs everywhere outside the encrypted account record.
- Encrypt stored emails and invited visitor emails at rest.
- Hash tokens at rest for magic links, invites, session export downloads.
- Use short-lived signed visitor tokens with one-time bootstrap and bounded polling lifetime.
- Support per-device session revocation from settings.
- Implement soft delete immediately, hard delete after 30 days via background job.
- Email exports to the verified address only; do not offer direct browser download without re-authenticated access.

## Delivery workstreams

### Workstream 1: platform and domain foundation

- account model
- auth and sessions
- Postgres schemas
- snapshot contracts
- event ingestion API
- privacy boundary setup

### Workstream 2: simulation and notebook

- tick scheduler
- personality and mood engine
- presence reservoir
- weather scheduler
- notebook prose compiler
- calibration harness

### Workstream 3: aviary client and rendering

- scene renderer
- responsive layout
- top bar fade behavior
- initial load and quiet field
- notebook UI

### Workstream 4: audio and accessibility

- procedural call synthesis
- listen-in mixer
- captions
- narration queue
- reduced-motion renderer
- keyboard navigation and focus treatment

### Workstream 5: social and account lifecycle

- invite creation and revocation
- visit bootstrap and read-only polling
- visit log
- export flow
- soft delete and recovery

## Validation strategy

### Simulation correctness

- property tests proving personality traits never decrement due to absence
- replay tests for ordered event-log consumption
- catch-up tick tests after simulated worker delay
- calibration goldens for one-week measurable drift and three-week felt drift

### Sync correctness

- dual-device concurrent session tests
- visibility suspend/resume tests
- stale event retry and dedupe tests
- revocation propagation tests for active visitors

### Rendering and performance

- first-bird synthetic tests on throttled devices
- 30-minute soak tests for memory growth
- reduced-motion visual regression tests
- mobile responsive snapshot tests to ensure no bird crops off-screen

### Accessibility

- automated keyboard path coverage
- live-region queue behavior tests
- caption contrast and placement checks
- screen-reader usability sessions before launch

### Editorial quality

- notebook and narration review against tone rubric
- regression suite for banned language such as achievements, streaks, scores, or trait numbers

## Rollout plan

### Phase 0: internal prototype gate

Ship a closed internal build with:

- two birds only
- no visits yet
- real auth and sync
- full simulation tick
- notebook, audio, captions, and reduced-motion path active

Exit criteria:

- drift calibration looks right in the harness and in live dogfood
- first-bird perf meets budget on target devices
- no privacy boundary violations in telemetry review

### Phase 1: employee dogfood

Enable:

- visits for a small set of internal accounts
- export and delete flows
- accessibility feedback loop

Monitor:

- tick lag
- audio fallback rate
- notebook quality
- visit revocation correctness

### Phase 2: invite-only beta

Open external beta with:

- two starter birds
- age-based bird unlock logic enabled behind config
- default visits still off
- optional host visit notifications still off by default

Use internal seeded accounts with backdated aviary ages to exercise third through seventh bird behavior before general users naturally reach those ages.

### Phase 3: public v1

Launch with the full v1 scope. Keep bird unlock thresholds operationally configurable. Suggested initial thresholds:

- day 0: 2 birds
- day 30: 3 birds
- day 90: 4 birds
- day 180: 5 birds
- day 270: 6 birds
- day 365: 7 birds

These thresholds should remain config-driven so the team can slow or pause unlocks if recognizability, performance, or notebook quality degrade at higher bird counts.

## Day-one instrumentation and operating thresholds

Instrument from day one:

- first-bird render percentile distributions
- tick queue lag and duration
- snapshot payload sizes
- WebAudio init success rate
- unsupported-browser rate
- reduced-motion usage rate as an aggregate count
- active visit count and revocation latency aggregates

Operational alerts:

- tick p99 over 5 seconds
- first-bird p95 over 500ms on target synthetic profile
- audio fallback rate above agreed threshold
- snapshot p95 size regression beyond budget
- memory soak regression failure in CI

## Principal risks and mitigations

### Drift calibration misses the emotional target

Risk:

- birds feel unchanged for too long or visibly change between sessions

Mitigation:

- build calibration harness first
- treat coefficients as config, not constants compiled into code
- review live dogfood notebook output alongside numeric drift traces

### Multi-device sync corrupts canonical personality

Risk:

- stale client sessions overwrite newer state or replay old events

Mitigation:

- clients never write personality
- event dedupe and ordering by server receive time
- tick idempotency
- aggressive dual-device concurrency testing

### Audio feels canned or indistinct

Risk:

- repeated phrases break aliveness
- too many birds blur into undifferentiated ambience

Mitigation:

- persistent per-bird seeds plus species motif variance
- recognizability QA with blinded listening tests
- operationally configurable bird unlock thresholds

### Accessibility ships as fallback instead of designed surface

Risk:

- narration becomes raw state labels
- reduced-motion becomes "animation off"

Mitigation:

- shared prose compiler
- dedicated reduced-motion visual system
- accessibility acceptance criteria equal to core feature criteria

### Privacy boundaries erode over time

Risk:

- engineers leak per-bird event detail into logs or analytics

Mitigation:

- explicit allowlist of aggregate metrics
- simulation DB isolated from analytics warehouse
- schema review gate for all observability changes

### Notebook quality becomes generic

Risk:

- prose sounds templated, repetitive, or gamified

Mitigation:

- editor-reviewed phrase library
- sparsity controls
- regression tests for banned phrasing
- observation triggers tied to meaningful state patterns rather than every session

## Recommended implementation sequence

1. Stand up account, session, aviary, bird, and event schemas.
2. Build the server-authoritative snapshot and event API.
3. Implement the simulation worker with reservoir, mood, drift, and notebook generation.
4. Build the scene renderer with quiet field bootstrapping and responsive layout.
5. Add audio synthesis and listen-in mixing.
6. Add narration, reduced-motion, captions, and keyboard support.
7. Add visit invites, read-only visitor bootstrap, and revocation.
8. Harden exports, deletion, observability, and rollout controls.

If the team holds this order, the hardest correctness constraints land before polish, and the polish surfaces still have time to be built as first-class product features rather than bolt-ons.
