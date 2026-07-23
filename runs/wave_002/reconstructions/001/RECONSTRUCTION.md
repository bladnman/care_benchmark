## System-level intent

- Server-authored canonical state with a hard client/server boundary. This shows up in "server-side simulation tick" as "the only writer of canonical aviary state," in Architecture as "clients never write personality state," and in Sync as "One canonical record" and "No last-write-wins." The intent is to make divergent simulation and absolute client writes unreachable by construction.

- Drift should be monotonic, expressive, and never punitive. This shows up in Scope's "No Tamagotchi mechanics" and "Drift is monotonic toward expressive," and in the drift function's "Deltas are non-negative" and "ambient quietness, not regression." The plan treats neglect as "absence of signal," not decay.

- The product voice is quiet, sparse, and non-gamified. This shows up in "No gamification of any flavor," "no streaks, counters, badges," the "quiet field," the passive notebook dot as "noticing, not announcing," and age offers that are "never a modal, never a badge count." The plan repeatedly protects against "announcements/gamification."

- The aviary should feel alive without faking liveness. This shows up in "the aviary has been living," "no cached-aviary rendering that could present stale state as live," the "honest quiet field," server ticks that continue while rendering stops, and first-frame rendering from an inlined snapshot.

- Observation should be naturalist prose, not metrics or user surveillance. This shows up in the field notebook as "read-only, sparse, naturalist prose," notebook entries that are "never numeric" and "never about the user's behavior," and screen-reader narration from the same "naturalist prose" generator family.

- Privacy is a hard boundary enforced by schema and infrastructure. This shows up in "Email appears only on the account record," visitor sessions producing "no rows in interaction_events," and observability's "Hard boundary" where no metric carries `account_id`, `bird_id`, or per-account interaction dimensions.

- Social access is optional, read-only, and deliberately not a network. This shows up in Visits as "per-invite opt-in," "read-only ambient view," "revocable," "off by default," and in non-goals as "No social-network surfaces."

- Accessibility is part of v1, not a later patch. This shows up in "Accessibility ships in v1, not after" and "Reduced-motion, narration, captions are launch-blocking features." The canvas is `aria-hidden`; semantics live in narration and DOM chrome.

- Procedural variation should preserve recognizable identity. This shows up in call grammar and audio: "no two plays identical, signature preserved," "recognizability survives drift," and no recorded audio fallback. Variation changes micro-timing and pitch, while fixed per-bird pitch offset preserves signature.

- Quality and boundaries are enforced by tests, schemas, lint, and CI gates rather than policy alone. This shows up in monotonicity fuzz tests, template lint banning "you visited," metrics field allowlists, "CI-enforced, not guidelines" budgets, and schema-level constraints where no endpoint accepts trait values.

## Per-feature whys

### Scope

- Single-user accounts, email magic-link sign-in, one aviary per account: NOT RECOVERABLE FROM PLAN

- Two starter birds at adoption: NOT RECOVERABLE FROM PLAN

- Growth from starter birds to a hard cap of seven birds via age-based offers: The plan makes growth "age-gated" and says age offers begin "organically," so no ramp is needed at general availability. The quiet affordance, never a modal or badge count, supports the non-gamified "notice-never-announce" product voice.

- Server-side simulation tick as the only writer of canonical aviary state: The rationale is sync correctness and trait integrity. Clients "write events; the tick computes deltas," and "there is no API that accepts an absolute trait value."

- Client rendering of snapshots with interpolation: The rationale is that "nothing in the visual scene requires server round-trips per frame." The client draws from the latest snapshot plus elapsed wall-clock time, while interpolation avoids teleports.

- Presence accounting by the three-signal conjunction: The rationale appears in Risks: "Any laxer presence definition silently inflates population drift." The conjunction prevents false presence from distorting drift.

- Return-greeting: The plan grounds this as an absence-aware ambient cue: greeting form comes from "absence-length bucket" and mood, exactly one bird greets first, and the greeting is "never as text."

- Listen-in: The rationale is focus without breaking ambience. The focused bird ramps up, others ramp down to "ambient, never silent," and the mixer has "No hard cuts anywhere."

- Offer of seed, song fragment, or still pool: The rationale is that offers become interaction signals for drift: accepted offers lift `curiosity`, any offer lifts `boldness` slightly, and offer engagement is consumed by the tick rather than writing state directly.

- Settle with 5s undo: NOT RECOVERABLE FROM PLAN

- Field notebook: The rationale is sparse observation over real state. Entries are "template-composed naturalist prose," "never numeric," "never about the user's behavior," and capped to preserve "sparsity."

- Single horizontal scene, perch zones, day/night, rare weather, ambient micro-motion, fading top bar: The rationale is mostly ambient continuity and quiet chrome. The top bar fades, weather is scheduled server-side, and day/night uses local time so the aviary reads as a living place rather than a static UI.

- Visits as opt-in, read-only, revocable, expiring, silent, and off by default: The rationale is to allow a limited "ambient view" without creating social-network surfaces or simulation effects. Visitors cannot call events, and visitor sessions create no interaction rows or presence windows.

- Screen-reader narration, reduced motion, captions, contrast, and keyboard navigation: The rationale is launch-blocking accessibility. The plan says these are v1 features, not "post-launch fixes," and the surfaces are generated from the same state as the rendering.

- Account export: The rationale for including personality vectors is explicit: export is "to the user themselves," so vectors are included.

- Soft-then-hard deletion: The rationale is recoverability followed by complete scrub. Soft delete lasts 30 days; hard delete scrubs every table including events, visit logs, and telemetry keys.

- Aggregate-only operational telemetry: The rationale is the "hard privacy boundary." Metrics cannot carry account, bird, or per-account interaction dimensions, and the telemetry warehouse has no path to the simulation DB.

- Web-only, no native apps: NOT RECOVERABLE FROM PLAN

- No gamification: The rationale is to prevent scope creep toward "announcements/gamification" and avoid streaks, counters, badges, calendars, or engagement surfaces.

- No Tamagotchi mechanics: The rationale is that absence should not become punishment. The plan says there is no death, hunger, distress, decay, and drift is "monotonic toward expressive."

- No social-network surfaces: The rationale is to keep visits from becoming profiles, follows, feeds, discovery, leaderboards, comments, co-presence, chat, or avatars.

- No recorded audio anywhere: NOT RECOVERABLE FROM PLAN

### Architecture

- Four services behind a single API gateway with one relational store: NOT RECOVERABLE FROM PLAN

- Auth service: NOT RECOVERABLE FROM PLAN

- Simulation service: The rationale is ownership of canonical aviary state. It runs ticks, consumes the event log, computes deltas and moods, and is "the only writer of personality vectors."

- State/read service: The rationale is fast read access. It serves snapshots and keeps a per-account edge cache for the "fast first load."

- Visit service: The rationale for separation is that visits have "their own authorization surface," even if small enough to fold into state service.

- Append-only interaction event log using a partitioned log table rather than Kafka: The rationale is v1 scale: "a simple partitioned log table is sufficient" and "we do not need Kafka."

- Snapshot cache: The rationale is first-load performance through edge-cached JSON and inlined first snapshot rendering.

- Email delivery dependency: NOT RECOVERABLE FROM PLAN

- Server-owned personality, mood, positions, calls, presence time, drift, weather, and day/night: The rationale is canonical consistency. These are simulation state, not client state.

- Client-owned rendering, interpolation, procedural calls, audio mixing, ornaments, presence detection, and event submission: The rationale is that the client can render smoothly from snapshots while only submitting events, not canonical traits.

- API schema with no absolute trait values: The rationale is enforcement "with teeth"; no request type includes a field that could overwrite personality state.

- Render pipeline from snapshot plus elapsed wall-clock time: The rationale is no per-frame server dependency. Ambient ornaments carry "no simulation state," and rendering stops entirely when hidden.

### Data model

- Synthetic UUIDs and email only on the account record: The rationale is privacy isolation. Email is encrypted at rest and "Nothing else in the schema references email."

- Personality seed values in the mid-low range: The rationale is explicit: "mid-low seeds leave headroom for three weeks of visible drift."

- Static species catalog with motifs, palettes, silhouettes, and nightjar flag: NOT RECOVERABLE FROM PLAN

- Immutable bird identity with rename touching only `name`: NOT RECOVERABLE FROM PLAN

- Personality vectors only in `birds.personality`, never recomputed from logs or exposed in client payloads: The rationale is to keep personality as canonical server state and prevent client-facing or derived trait paths.

- Append-only `interaction_events`: The rationale is ordered tick consumption and deterministic additive updates; sync correctness depends on the tick folding ordered events into one vector.

- Visitor sessions producing no interaction events or presence windows: The rationale is read-only visits that cannot influence the host aviary's simulation.

- `notebook_entries` with `generation_key`: The rationale is idempotency so "re-ticks never duplicate an entry."

- `visit_log_entries` written silently on session boundaries: The rationale is that the host sees the log only "on-demand"; visits are not pushed as announcements.

- `aviary_age_offers` driven purely by aviary age: The rationale is age-gated, non-engagement-based growth.

### API Surface

- Magic-link issuance and verification: NOT RECOVERABLE FROM PLAN

- Single-use magic links with immediate invalidation and rate limits: NOT RECOVERABLE FROM PLAN

- Session token as httpOnly cookie plus bearer fallback: The rationale for the fallback is "Safari ITP edge cases."

- Email change where old email works until verification: NOT RECOVERABLE FROM PLAN

- Account export job that emails a download link: NOT RECOVERABLE FROM PLAN

- Account deletion with recover action on signed-in pages: The rationale is the soft-delete period being recoverable before hard deletion.

- Account settings for captions, reduced motion, visit notifications, and audio mute: NOT RECOVERABLE FROM PLAN

- Snapshot polling rather than websockets: The rationale is that polling is "simpler and cheaper than websockets," and "30s staleness is invisible" against a 60s tick.

- Snapshot schema with aviary-level state, per-bird animation parameters, call schedule, and notebook flag: The rationale is that the client can render everything from the snapshot without frame-by-frame server access.

- Passive notebook "new entries" dot: The rationale is discoverability. The plan says the notebook is otherwise undiscoverable, and the dot is "noticing, not announcing."

- Mood transmitted as opaque animation parameters: The rationale is to let the client render mood-shaped idle motion while preventing any path from rendering `"mood: content"` as text.

- Paginated notebook entries, newest first, unbounded history: NOT RECOVERABLE FROM PLAN

- Batched event submission with 202 responses and effects in the next snapshot: The rationale is preserving the server tick as the writer of effects; clients submit events and wait for canonical state.

- Presence pings sent only while the conjunction holds: The rationale is accurate presence accounting; pings stop "the moment any leg fails."

- Visit invite one-time link with no bulk: The rationale is per-invite opt-in rather than broad social sharing.

- Visit snapshot checked for expiry or revocation on every pull: The rationale is immediate revocation; active visitor sessions terminate on their next snapshot pull.

- Visitor role blocked from `/aviary/events`: The rationale is read-only access with no simulation effects.

### Simulation Engine Design

- Tick scheduler running roughly every 60s with jitter: The rationale for jitter is "to smooth load," with exact cadence calibrated in build.

- Tick consuming unconsumed events in sequence: The rationale is deterministic state updates from ordered events.

- Presence windows materialized from pings: The rationale is to turn heartbeat pings into bounded presence intervals that close on settle, stopped pings, or gaps.

- Personality deltas applied additively: The rationale is monotonic drift with no absolute overwrites.

- Non-negative deltas: The rationale is explicit: neglect yields "absence of drift - ambient quietness, not regression."

- Trait targeting from presence, listen-in, accepted offers, and nearby offers: The rationale is to map different interaction signals to different expressive traits while keeping presence as the "dominant weight."

- Drift calibration targets and headless harness: The rationale is to make drift measurable after about one week and user-visible after about three weeks, while keeping a single 30-minute session below perceptibility.

- Mood transitions via priors, interaction bias, ambient bias, personality modulation, and contagion: The rationale is a persistent mood state shaped by time, weather, personality, interactions, and nearby birds rather than resetting on tab open.

- Account-timezone time of day: The rationale is local-time day/night behavior, including dusk, morning, night, and nightjar differences.

- Server-scheduled call intents with client synthesis: The rationale is recognizable procedural calls with variation. Server controls intent; client produces micro-timing and pitch variation.

- Per-bird fixed pitch offset: The rationale is that recognizability survives drift because drift changes "how often" and "how eagerly," not base timbre.

- Chorus clusters when vocal birds overlap: The rationale is emergent bird-to-bird coupling, with staggered offsets so cues do not fire in unison.

- Bird-to-bird wary contagion: The rationale is coupling between nearby birds rather than isolated state transitions.

- Notebook rule set over real state: The rationale is sparse entries about real observations such as greeter changes, quiet stretches, offer reactions, and weather responses.

- Notebook lint banning "you visited" patterns: The rationale is to keep notebook prose from becoming user-behavior tracking.

- Return-greeting greeter selection and stagger: The rationale is a single first greeter plus possible response so simultaneous cues "never fire in unison."

### Sync Model

- One canonical record: The rationale is "nothing to merge and no client-to-client path."

- No last-write-wins: The rationale is that additive, server-authored deltas in event-log order make divergent simulation "unreachable by construction."

- Per-account sequence assignment at event ingest: The rationale is total and deterministic tick consumption order.

- Matter-of-fact conflict surfaces: NOT RECOVERABLE FROM PLAN

- No offline mode and no cached live aviary rendering: The rationale is that stale rendering would "fake continuity," which the plan says is worse than an honest quiet field.

### Frontend Rendering Pipeline

- Canvas 2D renderer for scene with DOM for chrome and accessibility: The rationale is that 60fps idle micro-motion with pose blending is cheaper in immediate mode, while parallel DOM preserves semantics.

- Framework-light rendering and code splitting: The rationale is to protect the 2MB gzip budget.

- Single horizontal scene with three depth-scaled perch zones: The rationale is a responsive continuous scene where birds are never cropped or offscreen.

- Subtle parallax: NOT RECOVERABLE FROM PLAN

- First snapshot inlined in the edge HTML: The rationale is that the client "never waits for a round trip before drawing."

- First frame with birds mid-activity and ambient drift already running: The rationale is an immediate living scene rather than a static load.

- Audio starting only on first user gesture with captions covering pre-gesture window: The rationale is the browser autoplay policy, "a platform constraint we can't design away."

- Quiet-field slow loading state with no spinner, progress bar, or fade-from-static: The rationale is honest quietness rather than fake continuity or noisy UI.

- First bird visible under 500ms on mid-tier mobile over 4G: The rationale is fast first load, enforced by synthetic RUM gates.

- Idle animation state machine and interpolation: The rationale is mood-legible micro-motion and no teleports between snapshots.

- Return-from-hidden cross-fade: The rationale is so a day of server-side change reads as "the aviary has been living," not "a jump cut."

- Top bar icons fading to low opacity: The rationale is quiet chrome that recedes when the cursor is still.

- No badges except notebook dot: The rationale is to avoid counters and announcements while preserving notebook discoverability.

- Offer panel with seed, song-fragment library, still pool, and settle: NOT RECOVERABLE FROM PLAN

- Reduced-motion as a separate render path: The rationale is "not a degradation"; mood remains legible through slow cross-fades and unchanged audio, captions, notebook, and drift.

- Hidden-tab stopping render loop and audio scheduling: The rationale is resource hygiene and accurate presence; pings stop and fresh snapshot is pulled on return.

- Frame-gap watchdog after suspend: The rationale is resync after hidden or suspended time.

### Audio Pipeline

- WebAudio procedural synthesis from motif params: The rationale is compact, variable audio without recorded files; motif libraries are "kilobytes, not audio files."

- Variation seed perturbing timing, pitch, and amplitude: The rationale is "no two plays identical, signature preserved."

- Chorus mixing with per-bird gains and soft compressor: The rationale is balanced chorus audio by perch depth without hard artifacts.

- Chorus events from server-scheduled call intents: The rationale is that chorus is emergent from simulation intent, "not a client effect."

- Listen-in gain ramps: The rationale is focused listening while others remain "ambient, never silent."

- Rain low-pass/gain dip and night quiet curve: NOT RECOVERABLE FROM PLAN

- Captions generated from the same motif params being synthesized: The rationale is that the "caption always matches what played."

- Captions in naturalist voice with WCAG AA contrast: The rationale is accessible, scene-adjacent audio description.

- WebAudio fallback to graceful silence with captions force-on: The rationale is that the user can still follow calls, while "No recorded-audio fallback exists."

- Audio buffers preallocated, reused, and bounded by seven birds: The rationale is resource hygiene with no per-call allocation and a bounded node pool.

### Accessibility Surfaces

- Screen-reader narration via `aria-live="polite"` naturalist prose: The rationale is an accessible semantic parallel to the canvas scene, using the same snapshot state.

- Narration cadence and rate-limited queue: The rationale is to avoid flooding the screen reader while still bumping priority for user-initiated events.

- Canvas `aria-hidden` with semantics in narration and DOM chrome: The rationale is that all accessibility semantics live outside the non-semantic canvas.

- Testing with NVDA, JAWS, and VoiceOver: NOT RECOVERABLE FROM PLAN

- Keyboard navigation through top bar, scene, birds, listen-in, and settle: The rationale is full keyboard operation for core interactions.

- Focus indicator validated against day and night scenes: The rationale is visible focus under both bright and dim scene conditions.

- AA copy, captions, and notebook backing scrim: The rationale is contrast that passes "regardless of scene brightness."

- Accessibility as a launch-blocking shipping rule: The rationale is explicit: accessibility ships "in v1, not after."

### Data Lifecycle Features

- Adoption sequence from signup to two named starters to first soft fly-in: The rationale is that the aviary becomes "never empty again."

- Age-based offers at roughly 3, 6, 10, 14, and 18 months: The rationale is controlled, age-gated growth to a seven-bird cap without engagement mechanics.

- Age offer as quiet top-bar affordance, never modal or badge count: The rationale is the same quiet, non-announcing voice used for the notebook dot.

- Export JSON with birds, vectors, moods, notebook, and settings: The rationale for vectors is user ownership of their own export.

- Export link expiring after 24h: NOT RECOVERABLE FROM PLAN

- Deletion hard job scrubbing all tables: The rationale is complete account deletion, while aggregate telemetry is accountless.

### Performance Budgets and Observability

- Initial JS bundle under 2MB gzip: The rationale is protected by bundle-size gates and framework-light/code-split frontend choices.

- Time to first bird under 500ms: The rationale is fast perceived arrival of the aviary, enforced by synthetic RUM and release gates.

- 60fps idle motion and no memory growth over 30 minutes: The rationale is sustained ambient performance, enforced by frame-timing and heap soak tests.

- Tick latency p99 under 5s alarm: The rationale is preventing aviaries from becoming visibly behind.

- Aggregate RUM and synthetic browser fleet: The rationale is operational visibility into performance and errors without per-account engagement tracking.

- Metrics field allowlist and warehouse isolation from simulation DB: The rationale is to enforce the telemetry privacy boundary "not by policy."

- Privacy policy naming aggregate categories in plain text: The rationale is user-readable transparency about what telemetry exists.

### Rollout

- Phase A internal flag for engine, renderer, and audio: The rationale is staff-only calibration before exposure.

- Headless drift calibration in Phase A: The rationale is to test the 1-week and 3-week targets before shipping tuning.

- Phase B invite beta: The rationale is controlled observation of tick p99, first-bird timing, audio errors, and caption/narration quality.

- Weekly human review of caption and narration samples: The rationale is quality review for naturalist prose and accessibility outputs.

- Phase C general availability with age-offers beginning organically: The rationale is that no ramp is needed because growth is age-gated.

- Visits off by default from day one: The rationale is preserving optional, opt-in social access.

- Day-one funnel-lite aggregate counts: The rationale is measuring signup, adoption, and day-7 return without per-account engagement dashboards.

- No per-account engagement dashboards: The rationale is making them "deliberately unbuildable under the telemetry allowlist."

### Risks and Mitigations

- Drift calibration harness, monotonicity tests, QA time-scale knob, and server-side retuning: The rationale is avoiding drift that is too fast ("Tamagotchi feel") or too slow ("screensaver").

- Presence module tests and anomalous-window filtering: The rationale is that corrupted presence "silently inflates population drift."

- Schema constraints, codeowner review, and property tests for sync: The rationale is preventing a future optimization from reintroducing absolute writes.

- Audio entropy, signature, and listening tests: The rationale is that synthetic-cheap calls "break the spell faster than silence."

- Shared snapshot source for renderer and narration plus regression suites: The rationale is that canvas plus live region is a "fragile pairing" that can desync.

- Scheduler p99 alarm, partitioning, and jittered cadence: The rationale is avoiding tick backlog that leaves aviaries "visibly behind."

- CONTRIBUTING non-goals, PR checkbox, and notebook-template lint against user-behavior observations: The rationale is preventing predictable scope creep toward "a well-meaning toast or streak."

- Metrics allowlist and synthetic UUID rule: The rationale is preventing a "harmless" per-account metric from eroding the privacy boundary.
