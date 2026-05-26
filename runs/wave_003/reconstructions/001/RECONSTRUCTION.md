## System-level intent

- The product is an ambient, non-gamified aviary, not an engagement loop. This shows up in Scope's "Gamification" non-goal: "no achievements, streaks, levels, scores, badges, green-dot calendars, XP, ranks, tiers, or any engagement counter"; the plan calls this "absolute and non-negotiable." It reappears in the gamification creep risk as an "absolute refusal of gamification" and a "load-bearing design decision."

- The birds should feel alive through continuity, variation, and subtlety rather than explicit game mechanics. This appears in "ambient micro-motion," the "Return-Greeting Implementation," the "Idle Micro-Motion System" where motion is "never perfectly periodic," and the risks section's "Feels Alive" principle: calls must "never repeat identically" and the "first-frame-already-running conceit must be flawless."

- The relationship is long-term, stable, and non-punitive. Scope includes "stable identity," "monotonic drift," and "New bird offers at age-based intervals"; the Tamagotchi non-goal says birds do not "die, get hungry, show distress, or have decaying happiness meters" and that neglect produces "ambient quietness, not visible suffering."

- The client is presentation, not authority. Architecture states that the client is a "renderer and event emitter" and that the server is the "sole state authority." The render pipeline "never writes to the simulation state," and the plan says this split is "non-negotiable" because it makes "multi-device sync correct" and prevents "personality vector corruption."

- Sync should be simple because the architecture makes conflict unlikely. The Sync Model says it is "architecturally simple because it is designed to be simple": one canonical record, simulation as "sole writer," clients as "read-only consumers," append-only events, and "no CRDT, no operational transform, no last-write-wins arbitration on personality state."

- Product voice splits between naturalist surfaces and system surfaces. The field notebook, narration, captions, and offer prompts use "naturalist voice": "lowercase," "present-tense," "specific to the bird and the moment," with "no exclamation marks," "no 'you'," and "no announcement framing." Sign-in, errors, account settings, accessibility settings, and sync conflicts use "matter-of-fact voice": "Direct and clear" with "No warmth pretending to be useful."

- Accessibility is part of the core experience, not a late add-on. Scope includes "screen-reader narration," "reduced-motion mode," "call captions," "WCAG AA contrast," and "full keyboard navigation." The accessibility regression risk says accessibility is "in the critical path from day one," and reduced-motion is a rendering flag that "cannot be 'partially' implemented."

- Privacy minimization is architectural. Account data uses a "synthetic" UUID "never derived from email"; the indexes section says email is "never used as a key, partition value, or log field." Observability is "aggregate-only" and explicitly does not measure "per-bird personality values," "per-account interaction patterns," "which birds users listen in to or offer to," "notebook entry content," or "visit patterns between specific accounts."

- Social and growth surfaces should stay quiet and opt-in. Social is limited to visit invitations "off by default," "per-invite opt-in," "revocable," "30-day expiry," a "read-only ambient visitor view," and "no co-presence." Bird offers are "gentle" and "non-intrusive," appearing as a naturalist-voice notebook message with no expiry.

- Performance is part of the illusion. Scope sets "<500ms time-to-first-bird," "60fps idle motion," and "no memory growth over 30 min." Initial state is inlined in HTML so the client hydrates "without a separate API call"; the loading state avoids a spinner, and the "first-frame-already-running conceit" is called out as something that must not break.

## Per-feature whys

### Scope

- Single horizontal scene: NOT RECOVERABLE FROM PLAN

- 2 starter birds with cap 7: The plan ties bird count to gradual, age-based growth: all accounts start at 2 birds, beta withholds third-bird offers, and later offers appear at 90, 180, 270, 365, and 450 days.

- 3 perch zones: The plan uses front, middle, and back perches to support visual layering, bird focus order, perch preference by mood, adjacency for bird-to-bird effects, and stereo position in the audio mix.

- Day/night cycle: The plan uses local time and smooth phase interpolation so the aviary changes with the user's day; evening and night also quiet calls and settle most birds, except the nightjar.

- Ambient weather: The plan uses weather as an input to mood, notebook entries, rendering, and ambient audio, giving the aviary a naturalistic state beyond direct user actions.

- Ambient micro-motion: The plan's rationale is the "feels alive" principle: subtle, non-periodic movement prevents the scene from reading as robotic or canned.

- Personality vector: The plan uses five normalized traits as the source for mood, call frequency, plumage intensity, curiosity, social warmth, and rendering hints, while keeping raw trait values off the client.

- Monotonic drift: The plan implements "monotonic toward expressive" by clamping deltas to non-negative values. The rationale is long-term visible change without traits decreasing; the risk section also notes monotonicity lets the team slow drift "without corrupting existing state."

- Mood state machine: Mood is weighted by time of day, recent interaction, weather, nearby bird mood, and personality so bird behavior reflects context rather than arbitrary animation.

- Procedural call grammar: The plan refuses recorded audio and uses procedural calls so sounds can vary by species, mood, greeting, idle state, and chorus without downloading audio files or repeating identically.

- Idle motion system: Idle states are keyed to mood and jittered so the eye cannot detect a loop; this directly supports the "feels alive" intent.

- Bird-to-bird interaction: Mood contagion, chorus emergence, and call response make the aviary behave as a small social system rather than isolated animated birds.

- Stable identity: The plan treats identity as durable: bird IDs "never change," call signatures use deterministic seeds, and the sync risk says personality corruption is catastrophic because "the bird the user knows is gone."

- Return-greeting: The greeting is selected from absence duration, mood, boldness, and social warmth so returning feels acknowledged but varied; the "feels alive" risk says the return-greeting must be "genuinely varied."

- Listen-in: Listen-in raises the focused bird's audio, lowers others, and adds a subtle highlight "just enough to guide the eye," creating focused attention without stopping the ambient scene.

- Offer: Offers create a seed, song fragment, or still pool that nearby birds react to based on mood and curiosity; the same interactions feed drift deltas for curiosity, boldness, and vocal frequency.

- Settle: Settle gives the user a way to transition the aviary into quiet: light dims, calls fade, birds enter `settled`, and a 5-second undo window lets any click reverse the transition.

- Field notebook: The notebook is for "naturalist observation, not event logging." It captures periodic and noteworthy moments in lowercase, present-tense prose and is also where gentle bird offers appear.

- Presence accounting: Presence pings are aggregated into presence-time for drift computation; day-one instrumentation also checks "presence accuracy" to detect presence inflation.

- Magic-link email auth: NOT RECOVERABLE FROM PLAN

- Synthetic UUID account ID: The plan says account IDs are "never derived from email," and email is never used as a key, partition value, or log field, preserving the privacy boundary.

- Per-device session tokens: Per-device tokens support active session listing and revocation while using a browser/OS heuristic that is "not PII."

- Account export: NOT RECOVERABLE FROM PLAN

- Soft-then-hard deletion: The 30-day soft-delete window allows account recovery before hard deletion is scheduled.

- Server-side simulation tick: The simulation runs "whether or not any client is connected," so the aviary continues to advance and all clients later read the canonical result.

- Snapshot-based client consumption: Snapshots let the client consume server-authoritative state without mutating it; initial snapshots are also inlined for first-bird performance.

- Multi-device coherence via single canonical record: The plan says all devices read the same canonical record, so multi-device coherence is a "natural consequence."

- Visit invitations: Invitations are off by default, per-invite opt-in, revocable, and expire after 30 days, keeping social access bounded and intentional.

- Read-only ambient visitor view: Visitor snapshots are filtered to exclude interaction capabilities, preserving a quiet "ambient visitor view" rather than shared control.

- Visit log: NOT RECOVERABLE FROM PLAN

- No co-presence: The plan groups this with no social network surfaces; it prevents visits from becoming profiles, follows, comments, feeds, leaderboards, or "show-off" mode.

- Screen-reader narration: Narration gives screen-reader users the same naturalist, moment-specific aviary information through an ARIA live region.

- Reduced-motion mode: Reduced motion is a designed cross-fade surface: motion is replaced with still poses, fades, static overlays, and slower transitions while audio, captions, narration, notebook, and interactions remain unchanged.

- Call captions: Captions turn call grammar into prose descriptions such as "a warm, soft three-note rise," making audio information available visually and by default if WebAudio fails.

- WCAG AA contrast: Text, captions, notebook, settings, and errors have explicit contrast targets so they remain readable across day/night sky states.

- Full keyboard navigation: The plan defines focus order, key bindings, and visible focus indicators so birds, notebook, offer, settle, and listen-in are usable without a pointer.

- Performance budgets: The budgets protect the first-bird moment, idle smoothness, memory stability, and operational health: <2MB initial bundle, <500ms first bird, 60fps, and no memory growth over 30 minutes.

- System-selected starter birds: NOT RECOVERABLE FROM PLAN

- User-assigned names: NOT RECOVERABLE FROM PLAN

- Rename at any time: NOT RECOVERABLE FROM PLAN

- New bird offers at age-based intervals: The rationale is gradual, non-intrusive growth over months and a year; offers appear as naturalist notebook messages and do not expire.

### Architecture

- CDN Edge with inlined state: The edge serves static assets and an initial state snapshot in HTML to support the time-to-first-bird goal.

- API Gateway: It centralizes auth verification, rate limiting, and request routing before requests reach the specific services.

- Auth Service: It issues and verifies magic links, manages session revocation, and rate-limits magic link requests per email.

- Aviary API Service: It serves state snapshots, accepts interaction events into the append-only event log, serves paginated notebook entries, and caches snapshots at the edge for fast initial load.

- Simulation Service: It owns ticks, drift, mood transitions, call scheduling, notebook entry generation, and canonical state writes; account UUID partitioning lets it scale horizontally.

- Social Service: It keeps visit invitations bounded by create, revoke, expire, read-only snapshots, visit logs, and background cleanup.

- Account Service: It owns account CRUD, verified email changes, 30-day deletion, exports, and settings.

- Client/server split: The client renders, synthesizes audio, detects presence, and emits events; the server stores and mutates personality, mood, drift, calls, notebook entries, and snapshots. The plan says this prevents multi-device sync errors and personality corruption.

- Render pipeline boundary: Snapshot -> interpolator -> scene graph -> renderer/audio scheduler keeps visual and audio output downstream of server state and prevents the render pipeline from writing simulation state.

### Data model

- Account email encryption and hash: Email is encrypted for storage, hashed for uniqueness checks without decrypting, and never used as key, partition value, or log field.

- Bird stable internal identifier: Bird IDs "never change," preserving stable identity across names, moods, perches, and drift.

- Personality vector drift record: Drift records are separate to maintain a "full drift audit trail"; the bird vector is the running sum of seed values and applied deltas.

- Interaction event append-only log: Events are additive facts, processed once by ticks, and can be replayed if corruption is detected.

- Simulation tick record: It records consumed events, resulting snapshot, drift deltas, mood transitions, generated notebook entries, execution time, and duration so ticks can be audited and measured.

- Notebook entry prose: Entries use "naturalist voice, lowercase, present-tense" and trigger from periodic, greeting, offer, drift, weather, or bird-to-bird events.

- Presence session: The client flushes valid presence every 30 seconds as `presence_ping` events so the server can aggregate presence-time for drift.

- Visit invitation token and status: One-time tokens, statuses, and 30-day expiry bound visitor access and support revocation, use, and cleanup.

- Visit log entry: NOT RECOVERABLE FROM PLAN

- Session token device fingerprint: The device fingerprint is a browser/OS heuristic "not PII," used with issued, expiry, revocation, and last-used timestamps for session management.

- Species definition: Static species config gives each species silhouette, palette, motif library, personality defaults, and nocturnal behavior.

- Indexes and partitioning: Indexes support uniqueness, tick consumption, sequential access, pagination, invitation management, and drift audit queries; partitioning by account UUID supports shardable event and tick processing.

### API surface

- Authentication endpoints: The endpoints cover requesting and verifying magic links, refreshing tokens, revoking sessions, and listing sessions; rate limiting is explicitly attached to magic-link requests.

- Full snapshot endpoint: The endpoint returns current birds, moods, positions, personality-derived rendering hints, day phase, weather, ambient state, and transitions in under 4KB for efficient rendering.

- Delta snapshot endpoint: Delta snapshots send only changes since a tick and fall back to full snapshots if the delta is too large, supporting efficient polling.

- `render_hints` and `call_hint`: They expose personality-derived behavior without exposing raw trait values; the client can render and schedule while "never knowing the underlying numbers."

- Interaction event endpoint: Events are fire-and-forget so the client does not wait for simulation results; the next snapshot reflects processed events.

- Batch event endpoint: It exists for flushing accumulated presence pings.

- Field notebook endpoint: NOT RECOVERABLE FROM PLAN

- Account settings endpoint: It lets users update reduced motion, captions, audio, and visit notification preferences.

- Account email-change endpoints: They require verification of the new address before completing the change.

- Account export endpoint: NOT RECOVERABLE FROM PLAN

- Account delete and recover endpoints: They initiate soft deletion and allow recovery within the 30-day window.

- Social invitation endpoints: They create, list, and revoke invitations and expose the visit log so hosts can manage bounded visitor access.

- Visitor endpoint: The token is the credential and serves a read-only snapshot without requiring auth.

- Adoption endpoints: They gate accepting a new bird on aviary age availability and allow naming on accept.

- Rename endpoint: NOT RECOVERABLE FROM PLAN

- Initial state delivery: Inlining `window.__AVIARY_STATE__` eliminates a separate API call and supports the <500ms time-to-first-bird target.

### Simulation engine design

- Tick loop: The tick loop consumes unprocessed events, computes presence-time and drift, applies deltas, evaluates moods, schedules calls, generates notebook entries, writes canonical state, and marks events consumed.

- Drift function: The low-pass drift function mixes presence, interactions, and bird-to-bird context so change accumulates slowly rather than jumping after individual events.

- Drift base rate: The stated rationale is calibration: ~0.014/week at 1hr/day and user-visible movement after ~3 weeks.

- Presence weights: Presence is the "dominant driver," and plumage drifts fastest "with attention."

- Interaction modifiers: Each offer and listen-in event changes related traits: listen-in affects social warmth and vocal frequency, seed affects curiosity and boldness, song affects vocal frequency, and pool affects curiosity.

- Monotonicity enforcement: Clamping deltas to non-negative values implements "monotonic toward expressive" and guarantees traits never decrease.

- Drift calibration tests: The tests verify 7-day movement, 21-day user-visible movement, and 30 days of zero presence without trait decrease.

- Mood state machine: Mood weights combine time, recent interactions, weather, nearby birds, and personality; if no strong signal exists, the current mood is retained.

- Settled override: Night or the settle gesture overrides other mood signals into `settled`, except for the nightjar at night.

- Mood transition cross-fade: Transitions are not instantaneous; the client cross-fades idle motion over about 2 seconds.

- Call grammar runtime: Species-specific motif libraries, timing params, pitch params, and variation rules let calls reflect species and personality while staying procedural.

- Server-side call scheduling: The server provides `next_call_at` so call timing stays consistent with canonical mood and personality state.

- Chorus mechanic: Birds with high vocal frequency and social warmth can respond within a chorus delay using harmonically related motifs, making calls relational.

- Bird-to-bird mood contagion: Nearby birds can become wary after another bird becomes wary, giving the aviary shared ambient state.

- Bird-to-bird chorus emergence: Multiple vocal birds in content or alert moods can trigger a chorus event hint.

- Bird-to-bird call response: High social warmth increases probability of responding to another bird's call.

- Notebook entry generation: Entries are rare by default, boosted by noteworthy events, and suppressed if entries were generated recently, preserving the "roughly one entry every few days" cadence.

- Notebook template library: Templates are curated so entries read as "naturalist observation, not event logging" and avoid gamification language and user-behavior observations.

### Sync model

- Single canonical aviary record: One record per account makes multi-device coherence a natural consequence because all devices read the same state.

- No client-to-client sync: The plan avoids CRDTs, operational transform, and last-write-wins on personality state because clients only submit events and never own state.

- Snapshot pull triggers: The plan gives rationales for each pull: bootstrap, hidden tab state advancement, unfocused window state advancement, staying current with server ticks, confirming settle/unsettle, and catching laptop suspension.

- Snapshot interpolation: Current/next buffers and interpolation smooth 30-60 second snapshot gaps; discrete state changes use cross-fades.

- Event idempotency: Client-generated UUIDs deduplicate events on `(account_id, event.id)`.

- Event timestamps: Client timestamps order batches, while server timestamps control processing cadence.

- Conflict prevention: Personality and mood have single-writer simulation ticks, while interaction events are additive append-only facts.

- Account settings concurrency: Settings use last-write-wins with optimistic concurrency because settings are "low-stakes and rarely change simultaneously."

- Offline rendering: The client keeps rendering the last snapshot so the aviary appears frozen rather than broken.

- Offline event queue and reconnect: Queued events flush on reconnection, a fresh snapshot reflects background ticking, and the client catches up over about 5 seconds.

- Reconnection error surface: No error appears unless reconnection fails for more than 30 seconds; then the message is matter-of-fact.

### Frontend rendering pipeline

- HTML5 Canvas rendering: Canvas 2D is chosen instead of WebGL because the visual complexity does not require WebGL, Canvas is simpler to maintain, and it is more broadly supported.

- Lightweight scene graph library: The plan uses a custom ~50KB scene graph to keep rendering lightweight.

- Lightweight reactive framework for UI chrome: The framework handles top bar, notebook, and settings while the aviary scene remains imperative Canvas rendering.

- Procedurally generated SVG bird sprites: SVGs with plumage-saturation-driven color parameters avoid raster images in the critical path.

- Background SVG assets: Small SVGs and procedural ambient elements keep the critical path light.

- Scene composition layers: NOT RECOVERABLE FROM PLAN

- Day/night phase interpolation: Continuous sky gradients keep the user from noticing discrete palette changes.

- Idle state cadence and variation: Jitter and parameter variation prevent the eye from detecting repeated loops.

- Return-greeting selection: The greeting bird is the highest `boldness * social_warmth` among non-settled birds, matching greeting likelihood to personality.

- Return-greeting type by absence and mood: Short absences produce glances, day-length absences can produce quiet calls, and >24h absences can move a bird forward or create a response, scaling greeting intensity to absence.

- Listen-in visual and audio mix: The focused bird gets a subtle highlight and louder mix while others continue, maintaining ambient continuity while directing attention.

- Offer panel and scene element: The offer creates a visible seed, song fragment, or still pool so the interaction has an in-scene object rather than abstract UI only.

- Offer reaction by mood and curiosity: Reactions vary from quick approach to slow approach to head turn, so birds preserve their current state instead of always rewarding the action.

- Settle rendering: Lighting, calls, and idle states all transition into evening/settled over seconds, creating a full-scene quieting rather than a mode toggle.

- Settle undo window: Any click within 5 seconds reverses the transition, preventing an accidental settle from sticking.

- Reduced-motion rendering: Cross-fades, still poses, static overlays, and disabled drift replace animated paths while keeping the same interactions and audio.

- Fast loading path: Inlined snapshot and immediate Canvas first frame avoid showing a loading state under 500ms.

- Slow loading path: A quiet field with no spinner preserves the tone until the snapshot arrives.

- Error loading path: The message is matter-of-fact and tells the user to reload or get in touch.

- Responsive layout: The scene compresses without cropping birds; aspect ratio is maintained by adjusting inter-perch spacing rather than scaling the whole scene uniformly.

### Audio pipeline

- Audio scheduler from call hints: The scheduler consumes snapshot hints so audio timing follows canonical server state.

- WebAudio procedural call synthesis: Oscillators, noise, ADSR, and motif sequencing synthesize calls at runtime without downloaded audio files.

- Motif variation: Pitch shift, timing stretch, ornamentation, and envelope variation reduce repetition and support the "never identical twice" property.

- Waveform palette: Sine, triangle, FM pair, and filtered noise give pure whistles, softer breathy tones, warbles, and whisper components.

- Chorus mixer: Per-bird gain, master gain, and spatial panning allow focus, stereo placement, and controlled mixes.

- Gentle limiter: The limiter prevents clipping when multiple birds call simultaneously.

- Listen-in mix: Gain ramps focus the selected bird without abruptly muting the rest of the aviary.

- Hybrid call scheduling: Server hints keep calls synchronized, while client-side scheduling gives low latency and avoids waiting for the next snapshot.

- WebAudio fallback: If audio is unavailable, captions turn on by default, visual-only mode continues, and no error surface appears.

- Audio budget: Procedural synthesis and motif data stay within the 2MB bundle budget, with no audio files downloaded.

### Accessibility surfaces

- ARIA live region narration: A polite `role="log"` live region gives screen-reader users naturalist descriptions of idle state and user-initiated events.

- Narration cadence: Idle narration runs every 30-60 seconds, while user-initiated events preempt idle narration so direct actions are announced promptly.

- Narration queue management: The queue is capped at 3, idle narration replaces pending idle narration, and oldest non-priority items drop to prevent stacking.

- Reduced-motion detection: `prefers-reduced-motion` plus user override respects both system settings and explicit preference.

- Reduced-motion mode as visual-only change: The plan states audio, captions, narration, notebook, and all interactive functionality remain unchanged.

- Call captions: Captions map motif and mood to naturalist prose and appear near the calling bird with fade behavior.

- Keyboard focus order: The order moves from top bar into birds by perch and then notebook entries, giving predictable navigation.

- Keyboard shortcuts: Enter/Space, Escape, arrows, O, N, and S expose listen-in, navigation, offer, notebook, and settle without a pointer.

- Focus indicators: High-contrast outlines and underlines keep keyboard focus visible against bright and dim aviary states.

- Contrast and text: Explicit ratios protect readability for top bar, captions, notebook, settings, account surfaces, and errors.

### Performance budgets and observability

- Bundle budget: Component budgets and code-splitting keep the critical path around 665KB gzipped, "well under 2MB cap."

- Time-to-first-bird budget: CDN edge HTML with inlined snapshot, minimal critical JS, and Canvas first frame target 450ms total.

- Critical path optimization: Renderer and scene graph load first; audio, notebook, settings, and social defer until after first paint.

- Runtime performance budget: 60fps, p99 frame time under 16.6ms, memory growth near zero, one AudioContext, no per-call buffer allocation, small DOM, and <5ms snapshot parse protect long idle sessions.

- Simulation tick performance: Tick latency, cadence accuracy, and event lag targets ensure server ticks stay close to the 60-second model.

- Aggregate-only metrics: Observability intentionally excludes per-account and per-bird data to preserve privacy.

- Synthetic checks: Automated browser fleets from 5 geographies verify the aviary repeatedly.

- RUM and APM: Real User Monitoring captures aggregate render/load timing, while server-side APM tracks latency, traces, and errors.

- Alerting: Alerts fire on p99 tick latency, snapshot response time, or 5xx rate thresholds.

### Rollout

- Internal alpha: The first phase validates the core loop, calibrates drift, fixes rendering/audio bugs, and verifies multi-device sync with 10-20 accounts.

- Closed beta: The plan uses 200-500 users to measure drift at scale, performance budgets, "feels alive" perception, notebook frequency, and prose quality while withholding third-bird offers.

- Open beta: The plan opens registration, enables 90-day third-bird offers and visit invitations, and monitors scaling.

- General availability: GA removes beta labeling and enables the full bird count ramp up to 7 at age-based intervals.

- Birds-per-aviary ramp: Max birds increase slowly with aviary age; offers are gentle notebook messages, can be accepted any time, and do not expire.

- Day-one drift instrumentation: Drift deltas and cumulative weekly drift are measured from internal alpha to hit 0.01-0.02/week at 1hr/day presence.

- Day-one performance instrumentation: The plan measures all §10 metrics with alerting active from the first internal alpha user.

- Day-one mood transition instrumentation: Mood transition frequency is measured against a 2-5 transitions per bird per hour target.

- Day-one notebook quality review: Sampling 50 entries per week checks voice compliance.

- Day-one audio health instrumentation: WebAudio creation success and synthesis errors catch audio failures.

- Day-one sync correctness checks: Random client-rendered state comparisons against server state verify snapshot consistency.

- Day-one presence accuracy checks: Comparing client pings against server session data detects presence inflation.

- Feature flags: Flags provide kill switches and ramp controls for visits, third-bird offers, procedural audio, forced reduced motion, simulation ticks, and notebook generation.

### Risks

- Drift calibration mitigation: Instrumentation, simulated tests, server-configurable `base_rate`, weekly population distributions, and monotonicity control address too-fast or too-slow drift.

- Sync correctness mitigation: Drift audit trail, seed-plus-delta reconstruction, append-only replay, and single-writer ticks protect against personality corruption.

- Audio uncanniness mitigation: Real bird call structures, deep variation, listening sessions, beta feedback, and musical training aim to prevent mechanical or unpleasant procedural calls.

- Audio hybrid contingency: Short recorded base samples with procedural variation preserve "never identical twice" if pure procedural calls fail.

- Accessibility regression mitigation: Critical-path inclusion, automated tests, screen-reader beta users, shared narration/notebook code, and scene-graph-wide reduced motion prevent late or partial accessibility.

- Simpler narration contingency: If prose quality is insufficient, the plan prefers simple state descriptions over a broken narration experience.

- Feels-alive mitigation: Varied return-greetings, non-looping idle motion, non-repeating calls, flawless first frame, and "Turing test" sessions target affective aliveness.

- Privacy boundary mitigation: Separate simulation and analytics databases, telemetry field allowlists, review checklist, and automated tests prevent per-bird or notebook data entering telemetry.

- Gamification creep mitigation: Documentation, review checklist, not computing leaderboard/streak metrics, and notebook restrictions prevent engagement counters, streaks, badges, or milestone surfaces from creeping in.

### Appendices

- Species pool: NOT RECOVERABLE FROM PLAN

- Mood to idle motion mapping: The mapping gives each mood primary/secondary idle expression and perch preference, grounding mood in visible behavior.

- Naturalist voice reference: It keeps aviary, notebook, narration, captions, and offer prompts lowercase, present-tense, specific, bird-centered, and free of announcement framing.

- Matter-of-fact voice reference: It keeps sign-in, errors, account settings, accessibility settings, and sync conflicts direct, clear, and useful without naturalist phrasing.

- Engineering milestones: Milestones sequence dependencies from simulation, rendering, audio, interactions, notebook, auth, sync, accessibility, social, and performance into alpha, beta, open beta, and GA targets.
