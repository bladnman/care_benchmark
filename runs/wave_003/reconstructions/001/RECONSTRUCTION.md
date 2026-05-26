## System-level intent

- **Keep the aviary quiet, ambient, and non-gamified.** This shows up in the v1 non-goals: "No gamification of any kind" and "No Tamagotchi mechanics," plus the observability boundary that deliberately does not measure "per-bird engagement funnels," "individual user session duration," or "which bird is most popular." The plan also names the cultural risk of a "harmless welcome toast, a streak counter, or a notification."

- **Make care accumulative, not punitive.** The drift function says "No negative deltas" and "A bird ignored for weeks stays at its current trait values; it does not regress." This aligns with the non-goal that birds "do not die, hunger, show distress, or have decaying happiness meters."

- **Aim for slow expressive change, not a stat-management game or a screensaver.** The personality-vector calibration target is "measurable change in instruments after ~1 week of regular visits" and "user-visible change after ~3 weeks." The drift calibration risk says that if drift is too fast, users feel they are "playing a stat-management game"; if too slow, the product "feels like a screensaver."

- **Keep the client a viewer and event emitter, not a simulation participant.** The boundary rule says the client "renders snapshots," "never computes drift," "never writes personality values," and "never invents mood transitions." Sync repeats the same principle: "there is no client-side state to merge" and "Clients are dumb renderers."

- **Protect the canonical simulation from race conditions and last-write-wins behavior.** The Simulation Service is "the only writer of personality vectors, mood state, and notebook entries." Conflict prevention says there is "No last-write-wins on personality," because clients send events like "user listened in to Pip for 3 minutes" and the tick computes the delta.

- **Use naturalist prose and a specific, lowercase, present-tense voice for aviary meaning.** Field Notebook entries are "naturalist prose, lowercase, present-tense." Screen-reader narration has the "Same voice as field notebook," and the accessibility risk says narration must not become a "state-list readout."

- **Make accessibility part of the product's aliveness, not an afterthought.** Accessibility is in v1 scope and in Phase 5, and the risk section says accessibility features are "in the critical path, not a Phase-2 add-on." Reduced-motion mode must not ship as just "animations off"; its acceptance criteria are framed as a screen-reader user being able to describe "the aviary's mood after 5 minutes."

- **Treat performance as part of the first impression and ongoing spell.** The plan targets "time-to-first-bird <500ms," "60fps idle," and "no memory growth over 30min." The loading state rejects a spinner and instead uses a "quiet field" so the first bird appears "mid-motion as soon as the snapshot arrives."

- **Allow only opt-in, read-only, low-social viewing.** Visit invitations are "opt-in," "read-only ambient viewing," "revocable," and expire after 30 days. The feature "defaults OFF for every new account, with no onboarding prompt to share," and there are "No social network surfaces."

## Per-feature whys

### Scope

- **Single-user accounts**: The plan keeps one aviary per account and explicitly excludes "shared or multi-user aviaries," aligning account identity with a private aviary rather than a social surface.

- **Email magic-link sign-in**: NOT RECOVERABLE FROM PLAN

- **Multi-device session tokens**: The rationale is multi-device coherence: laptop and phone use "the same account credentials," sessions can be listed and revoked, and both devices read from the same canonical snapshot.

- **Account settings with export and deletion**: Settings persist accessibility preferences such as `reduced_motion`, `captions`, and `visit_notifications` server-side. Export, soft deletion, and 30-day recovery support account lifecycle operations without putting email anywhere except the Account record.

- **Two starter birds per new account**: NOT RECOVERABLE FROM PLAN

- **Seven-bird cap**: The cap is hard because "audio recognizability collapses above this threshold."

- **Single horizontal scene**: NOT RECOVERABLE FROM PLAN

- **Three perch zones**: NOT RECOVERABLE FROM PLAN

- **Personality vectors**: The vectors carry the product's slow-change promise while remaining hidden: they are "never exposed numerically to users," and calibration aims for instrument-visible change before user-visible change so expression can emerge without a stats surface.

- **Mood system**: The mood system lets time-of-day, weather, recent interactions, personality, and nearby birds shape what the user perceives. The plan's accessibility acceptance criterion asks whether a screen-reader user can describe "the aviary's mood after 5 minutes."

- **Procedural calls**: Procedural synthesis is required because the plan explicitly excludes a "recorded audio fallback." The audio risk frames believable procedural calls as part of the "aliveness spell," warning that a "robotic chirp" can break trust.

- **Idle motion**: Idle motion supports aliveness by keeping birds from moving in lockstep: clips get randomized start offsets so birds "never sync," and procedural perturbation ensures two content birds "never preen identically."

- **Drift function**: The drift function is named "the load-bearing algorithm" and "the product's central promise." It must move slowly enough to avoid a stat-management game, fast enough to avoid a screensaver, and never punish absence.

- **Bird-to-bird interaction**: NOT RECOVERABLE FROM PLAN

- **Return-greeting**: NOT RECOVERABLE FROM PLAN

- **Listen-in**: Listen-in gives attention to one bird without erasing the rest of the aviary. The focused bird ramps up while others drop but "Never to 0," and transitions use exponential ramps "for natural feel." It also shapes `social_warmth` and `vocal_frequency` through drift.

- **Offer seed/song/still pool**: Offers are interaction events that feed the simulation: an accepted offer nudges `curiosity`, an offer made nearby nudges `boldness`, and recent acceptance can nudge mood toward `content`.

- **Settle gesture**: NOT RECOVERABLE FROM PLAN

- **Presence accounting**: Presence is measured by the "three-signal rule" of visibility, focus, and recent pointer/key activity because presence-time is the base signal for drift.

- **Field Notebook**: Notebook entries are auto-generated, read-only aviary observations so they do not become a user-behavior ledger. The plan says entries are "never about user behavior" and "only about aviary observations."

- **Visit Invitations**: The rationale is constrained ambient sharing without building a social network: visits are opt-in, email-based, read-only, revocable, expire after 30 days, default off, and have "no onboarding prompt to share."

- **Server-side canonical state and no client-side state merging**: The rationale is sync correctness. There is "no merge" because clients do not own simulation state; this avoids last-write-wins corruption of personality, mood, and notebook state.

- **Screen-reader narration**: Narration gives non-visual access to the aviary in the same product voice as the Field Notebook: "lowercase, present-tense, specific," with idle updates suppressed if a previous utterance is still speaking.

- **Reduced-motion mode**: Reduced motion preserves the feeling of aliveness while replacing frame-by-frame animation with cross-fades, removing ambient leaf drift, and slowing palette shifts.

- **Call captioning**: Captions are generated from the same procedural call grammar as audio so text matches what played. In silent mode, captions are enabled by default.

- **Keyboard navigation**: Keyboard support makes bird focus, listen-in, offers, panels, and settle reachable without pointer input, with a focus indicator visible against all aviary states.

- **Performance budgets**: The budgets protect the first-bird moment and long-running ambient experience: initial bundle under 2MB gzipped, time-to-first-bird under 500ms, 60fps idle, and no memory growth over 30 minutes.

### Architecture

- **Web Client**: The client owns rendering, audio synthesis, input handling, presence detection, animation interpolation, captions, and narration because it is the "viewer and event emitter" side of the boundary.

- **API Service**: The API Service is stateless so it can handle auth, snapshot delivery, event appends, and account operations while remaining horizontally scalable.

- **Simulation Service**: The Simulation Service is stateful because it is the only writer for personality vectors, mood state, and notebook entries, and it runs the tick processor that turns events into canonical state.

- **Static Asset CDN**: The CDN serves JS bundles, species asset packs, and the HTML shell from the edge to support the quiet field and time-to-first-bird target.

- **Client/server split**: The split exists to enforce the simulation boundary: the server stores and mutates personality, mood, drift, notebook entries, and canonical snapshots; the client renders snapshots and emits events.

- **Render pipeline boundary**: Client-side interpolation allows 60fps scene rendering while the server ticks at about a 1-minute cadence. Snapshot pulls on visibility changes, post-sleep frame gaps, and keepalive protect coherence without making the client authoritative.

### Data Model

- **Account**: `account_id` is used everywhere internally while encrypted `email` lives only on the Account record, which supports the privacy boundary around identity.

- **Session**: Sessions support multi-device use and revocation through active session listing and deletion.

- **Aviary**: NOT RECOVERABLE FROM PLAN

- **Bird**: Stable `bird_id`, species, name, birth/adoption date, and perch position give each bird durable identity and snapshot-renderable placement.

- **Personality Vector**: The vector is server-authoritative and never client-written because it is the canonical long-term state that drift mutates.

- **Mood State**: Mood state tracks fast-timescale `current_mood`, entry time, expiry, and next scheduled tick so weather, time, and interaction effects can be timed.

- **Event Log**: The log is append-only because clients submit events while the Simulation Service reads and processes them. Server insertion order prevents race conditions between devices.

- **Field Notebook Entry data**: `trigger_events` are internal and not user-visible so generated observations can be grounded in events without exposing a behavioral log to the user.

- **Visit Invitation**: Status, expiration, revocation, and last-used fields support opt-in viewing that can be revoked and that expires.

- **Visit Session**: NOT RECOVERABLE FROM PLAN

- **Snapshot computed on read**: NOT RECOVERABLE FROM PLAN

- **Species Pool**: NOT RECOVERABLE FROM PLAN

### API Surface

- **Authentication endpoints**: NOT RECOVERABLE FROM PLAN

- **Snapshot polling endpoint**: Polling serves the current canonical state on initial load, visibility return, post-sleep gaps, and low-frequency keepalive so clients stay coherent without owning state.

- **Event submission endpoint**: Batched event submission returns 202 immediately because events are fire-and-forget into the queue for later simulation processing.

- **Account operations endpoints**: Export, deletion, recovery, session listing, and settings updates provide account lifecycle control and persistent preferences.

- **Visit endpoints**: Invite creation, invite listing, revocation, and visitor snapshot access support read-only visiting. The visitor endpoint allows "No event submission" from this session type.

- **Notebook endpoint**: The notebook has only a read endpoint because entries are "server-generated only."

### Simulation Engine Design

- **Tick architecture**: The tick turns unprocessed events into canonical state by computing presence-time, applying drift, transitioning moods, updating positions, generating notebook entries, advancing weather, writing state, and marking events processed.

- **Presence-time aggregation**: Presence-time is derived from `presence_ping` events using the three-signal rule because it is the base drift input.

- **Interaction multipliers**: Listen-in, accepted offers, and nearby offers change different traits so user interactions shape expression without direct stat writes.

- **Monotonic drift toward expressive traits**: The rationale is that "neglect" means `delta` is about zero, not regression; trait growth slows as traits approach ceiling.

- **Mood transition logic**: Mood transitions combine dawn, dusk, night, weather, recent interactions, personality gating, and bird-to-bird influence so mood is contextual rather than randomly assigned.

- **Call-grammar runtime**: Species motifs, `vocal_frequency`, mood, personality, and random seed generate varied calls; the server sends scheduled call time and motif seed, and the client synthesizes at trigger time.

- **Calibration guardrails**: Guardrails detect drift that is too fast or too slow, and service-configuration tuning allows post-launch calibration without deploy.

### Sync Model

- **Canonical source of truth**: The simulation database is the only canonical store for personality, moods, positions, and notebook entries, which keeps all devices coherent.

- **Multi-device coherence**: Multiple clients submit to the same event log and pull from the same snapshot endpoint, so there is "no client-side state to merge."

- **Conflict prevention**: Clients send events, not absolute values. Server insertion order and tick processing prevent race conditions between devices.

- **Mood sync**: Mood is snapshot-based so simultaneous clients render the same mood from the last tick, and neither client "owns" mood.

- **Offline behavior**: Offline clients keep rendering from the last snapshot with ambient motion; audio falls to ambient silence when cached schedules run out; reconnection pulls a fresh snapshot for a seamless update.

### Frontend Rendering Pipeline

- **Scene composition layer order**: NOT RECOVERABLE FROM PLAN

- **Bird rendering composite**: The bird rendering pipeline lets species silhouette, plumage fill, eye/beak detail, and feather texture express bird identity, with `plumage_saturation` tinting the visible bird.

- **Animation system**: The two-track system separates server-derived pose/position interpolation from idle micro-motion overlays, preserving canonical snapshots while keeping birds alive between snapshots.

- **Idle micro-motion categories**: Mood-specific motion categories make content, wary, drowsy, curious, and alert birds look different without exposing numeric personality.

- **Perch changes, mood changes, weather onset, and day/night transitions**: Smooth bezier paths, cross-fades, gradual weather opacity, and continuous palette interpolation avoid hard cuts and stepped changes.

- **Reduced-motion rendering**: Cross-fades, direct perch transitions, removed leaf drift, and slowed palette shifts preserve scene continuity for reduced-motion users.

- **Quiet field loading state**: The plan rejects a spinner and "fade-from-static" so slow loads still feel like a quiet aviary. The first bird appears mid-motion when the snapshot arrives.

- **Responsive layout**: Layout changes preserve birds across viewports, and aspect ratio locking prevents cropping birds by letterboxing with soft extended background if needed.

### Audio Pipeline

- **WebAudio procedural engine**: NOT RECOVERABLE FROM PLAN

- **Motif-to-sound mapping**: Motifs map to envelopes, filters, and durations so the same call grammar can produce actual synthesized calls.

- **Personality and mood shaping**: `vocal_frequency`, `boldness`, and mood change rate, complexity, amplitude, brightness, envelope, and pitch so calls express bird state.

- **Chorus mixing**: Separate gain nodes and compression prevent clipping; listen-in gives focus while keeping other birds audible.

- **Listen-in decay and cross-fade**: Fade-down and fade-up behavior avoids hard cuts when disengaging or switching focus.

- **WebAudio fallback**: Silent mode with captions on keeps the experience accessible if `AudioContext` fails or is denied, while honoring the no-recorded-audio boundary.

- **Audio memory management**: Pre-generated reusable motif buffers, per-call node cleanup, and suspending `AudioContext` when hidden support the no-memory-growth runtime budget.

### Accessibility Surfaces

- **Screen-reader narration generator**: The generator consumes the current snapshot and creates naturalist prose, preserving the same voice as the notebook while describing actual aviary state.

- **Narration cadence and queue behavior**: Idle narration every 30 to 60 seconds avoids chatter, while user-initiated events interrupt because they are more immediately relevant.

- **ARIA aviary region**: The aviary uses `role="region"`, `aria-live="polite"`, and `aria-label="aviary"` so the scene can be entered and narrated as a coherent region.

- **Reduced-motion toggle**: It responds to both `prefers-reduced-motion` and an explicit accessibility setting, with the preference stored server-side.

- **Call caption generation and display**: Caption strings come from the same motif-variation function as audio, then appear near the calling bird with high contrast and timed fade behavior.

- **Keyboard navigation and focus indicator**: Tab order, arrows, Enter, Escape, offer controls, settle shortcut, and a visible soft outline make the core interactions keyboard-reachable.

- **Contrast requirements**: User-copy text meets WCAG AA; scene elements are exempt because the plan says the scene itself has no user copy.

### Performance Budgets and Observability

- **Bundle budgets and code-splitting**: Code-splitting by route, preloading critical chunks, and compact species visuals are used to keep the initial bundle under 2MB gzipped.

- **Time-to-first-bird implementation**: Edge HTML, inlined critical CSS, async JS, and an embedded or very-low-latency snapshot exist to render the first bird within 500ms.

- **Runtime budgets**: `requestAnimationFrame` timing, heap profiling, buffer pools, and freed audio allocations protect 60fps idle motion and zero memory growth.

- **Synthetic monitoring**: Automated browsers check time-to-first-bird, snapshot latency, audio context success, and frame rate so regressions are caught from multiple geographies.

- **Real User Monitoring**: RUM is aggregate only and measures load timing, first bird render, frame timing, audio errors, and JS errors without per-account dimensions.

- **Server-side metrics**: Tick latency, event log lag, snapshot latency, and auth latencies are measured because slow ticks or stale snapshots would damage canonical state delivery.

- **Privacy boundary enforcement**: Telemetry is separated from simulation databases, and the audit query asserts no analytics warehouse reads from the simulation DB.

- **Metrics deliberately not measured**: The plan avoids metrics that could support gamification or privacy erosion, including per-bird funnels, individual session duration, and cross-account popularity.

### Rollout

- **Phase 0 infrastructure**: The phase establishes the CDN, services, databases, auth, event ingestion, synthetic monitoring, and RUM needed before product behavior depends on them.

- **Phase 1 bird engine core**: The phase comes early because personality vectors, drift, mood, tick scheduling, and calibration are the central simulation promise.

- **Phase 2 client rendering and audio**: Rendering, procedural calls, chorus mixing, listen-in, snapshot consumption, and interpolation are grouped to hit the 500ms first-bird target.

- **Phase 3 interactions**: Presence detection, return-greeting, listen-in, offer, settle, and notebook generation are grouped because they turn user events into simulation inputs and observations.

- **Phase 4 sync and multi-device**: Multi-device testing, no-last-write-wins validation, and tick-latency stress testing address the sync correctness risk.

- **Phase 5 accessibility and polish**: Narration, reduced motion, captions, keyboard navigation, WCAG, and performance audit are grouped before final QA because accessibility is in the critical path.

- **Phase 6 social and final QA**: Visit invitations, revocation, expiration, internal soft launch, calibration tuning, and bug fixing come late because sharing is constrained and optional.

- **Birds-per-aviary unlock cadence**: NOT RECOVERABLE FROM PLAN

- **Day-one instrumentation**: Day-one instrumentation measures time-to-first-bird, frame rate, load timing, tick latency, auth rates, audio initialization, and aggregate drift rates so calibration and performance can be tuned immediately.

### Risks and Mitigations

- **Drift-calibration harness**: The harness exists because drift has "no objective unit test for feels right" but is the central promise.

- **Configurable drift constants**: Keeping drift constants in service configuration allows hot-tuning during soft launch without deploying code.

- **Sync correctness invariant checks and chaos testing**: These protect against silent personality-vector corruption that users would feel but "couldn't name."

- **Procedural audio sound design and believability testing**: The plan calls for procedural audio expertise and repetition testing because robotic or repeated calls break the "aliveness spell."

- **Accessibility consultant, dogfooding, and acceptance criteria**: These mitigate the risk that accessibility becomes a state-list readout or that reduced motion breaks aliveness.

- **Performance CI gates and synthetic alarms**: These exist because bundle growth, slower time-to-first-bird, and frame-rate drops can regress as features accrete.

- **Telemetry credential separation, query auditing, and privacy review**: These prevent a "harmless" aggregate dashboard or analytics pipeline from reading per-bird interaction data or the simulation DB.

- **Non-goal onboarding, code review checklist, and temptation log**: These protect the product culture from welcoming, streak, notification, or engagement-metric ideas that look harmless but violate the boundaries.

- **Visit invite rate limits, revocation UI, expiration, and no public discovery**: These mitigate spam, confusing revocation, accumulated invitations, and amplification.

### Appendix Ambiguity Resolutions

- **Canvas 2D over WebGL**: Canvas 2D is preferred because the scene is not 3D, WebGL adds bundle size and compatibility risk, and Canvas can meet the required visual quality within budget.

- **Server framework choice**: The framework should be whatever the team knows well because the simulation tick, not HTTP framework choice, is the critical code.

- **PostgreSQL and optional append-optimized event store**: PostgreSQL is sufficient for v1 relational data and likely event-log scale, though a time-series or append-optimized store may help.

- **Tick scheduling with per-aviary jobs and retry**: A queue or cron-like scheduler gives each aviary its own tick job, with exponential backoff and alerts for failed ticks.

- **Client framework choice**: React or vanilla JS can work because the render loop is Canvas-based and mostly outside framework reconciliation; the team should choose based on expertise.

- **Client-side call caption generation**: Captions are generated from the same motif-variation function as audio so they "always match what was played."

- **Deterministic weather randomization**: Weather seeded by aviary creation date plus day-of-year simplifies testing and snapshot caching while still varying across accounts.
