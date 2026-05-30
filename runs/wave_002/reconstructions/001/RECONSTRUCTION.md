## System-level intent

1. Server authority and canonical state ownership

The plan repeatedly prefers a "server-authoritative snapshot plus append-only event-log model" and later restates that "server is the sole writer of personality and mood state," "clients write only interaction events," and "visitors have no write path into host state." This principle shows up in the planning frame, client/server split, event rules, sync model, and risk mitigation for multi-device sync. The intent is to protect canonical personality and mood from stale clients, peer sync, replay, and visitor writes.

2. Privacy-preserving deterministic authoredness

The plan wants notebook prose, narration, and captions to "feel authored" while avoiding "an online LLM dependency." It calls for a "shared deterministic prose system" and a "deterministic prose compiler" to protect "privacy, consistency, latency, and tone." This intent appears in the planning frame, text generation approach, notebook generation, screen-reader narration, captions, and notebook-quality mitigations.

3. Monotonic long-term care without punishment for absence

The plan separates "permanent personality drift" from temporary "quiet after absence" behavior so that "personality drift only moves toward more expressive states and never reverses on neglect" while returning users still find birds "quieter and more ambient." This appears in the planning frame, permanent drift vs ambient quietness, drift function, temporary expressivity reservoir, and simulation correctness tests.

4. Slow, read-heavy coherence over realtime complexity

The plan chooses "revisioned snapshot polling and visibility-triggered refresh" and says "Do not use websockets for v1." The rationale is that "the canonical simulation cadence is slow and read-heavy," state updates are "low-volume," and polling is "simpler, easier to harden, easier to cache," and less likely to create "invisible realtime race conditions." This appears in the planning frame, sync model choice, API snapshots, polling cadence, and visit consistency.

5. Ambient, non-gamified product philosophy

The plan explicitly excludes "Gamification of any kind" and "Tamagotchi mechanics," including hunger, decay, visible distress, negative drift, streaks, counters, achievements, levels, leaderboards, profiles, and discovery feeds. Notebook rules also say to "never mention numerical trait changes or visit-count behavior," and editorial tests ban "achievements, streaks, scores, or trait numbers." This principle shows up in out-of-scope decisions, notebook generation, accessibility narration, validation, and risk mitigation.

6. Accessibility as a first-class product surface

The plan includes "Accessibility surfaces from day one" and later warns against accessibility shipping "as fallback instead of designed surface." It requires narration, reduced-motion mode, call captions, keyboard navigation, focus treatment, and WCAG AA contrast. The plan says focus styling is "a design deliverable" and accessibility acceptance criteria should be "equal to core feature criteria."

7. Matter-of-fact system copy separated from naturalist voice

The plan asks for "naturalist voice" in authored observation surfaces while requiring "matter-of-fact voice" for "auth, error, revocation, unsupported-browser, and settings-system copy." This appears in text generation, field notebook UI, screen-reader narration, captions, and matter-of-fact system surfaces. The intent is to keep evocative bird observation separate from operational surfaces.

8. Performance through explicit client/server and render boundaries

The plan draws a "render pipeline boundary" and says not to build the aviary scene as "ordinary DOM animation." It separates React shell, dedicated scene renderer, and WebAudio engine so top bar and accessibility surfaces stay straightforward while protecting "runtime performance for continuous animation and audio." Performance budgets, build-time controls, and rollout gates reinforce this.

9. Honest presence and integrity of drift

The plan uses the PRD's "three-signal conjunction" and bounded server-accepted pings because the server "does not trust client-declared continuous duration." It drops stale pings and says this "keeps presence honest," "prevents clients from retroactively claiming hours of idle attention," and "preserves integrity of drift over convenience." This appears in presence accounting, event rules, offline behavior, and drift risk mitigation.

10. Quiet sociality with host control and no visitor influence

The plan frames visits as a "Quiet social feature" with "per-invite read-only visits, revocation, visit log, optional visit notifications off by default." Visitors poll with a "dedicated read-only token," revocation or expiration takes effect on the next polling response, and visitor presence "does not enter the host simulation ledger." The intent is social visibility without co-presence, collaboration, chat, or host-state mutation.

11. Operational configurability before and after launch

The plan repeatedly keeps sensitive thresholds configurable: drift coefficients as "config, not constants," bird unlock thresholds "operationally configurable," and public thresholds "config-driven" so the team can slow or pause unlocks. This appears in drift calibration risks, rollout phases, public v1 thresholds, and mitigations for recognizability, performance, and notebook quality.

12. Build correctness constraints before polish, then make polish first-class

The recommended implementation sequence says "the hardest correctness constraints land before polish," while "the polish surfaces still have time to be built as first-class product features rather than bolt-ons." This intent also appears in the delivery workstreams, validation strategy, and rollout gates.

## Per-feature whys

### V1 scope

- Web-only product for modern browsers: NOT RECOVERABLE FROM PLAN
- Single-user accounts with magic-link sign-in: NOT RECOVERABLE FROM PLAN
- One canonical aviary per account: The plan supports this through a "one server-owned canonical state" and server authority, which is meant to avoid peer sync, client-owned simulation state, and consistency ambiguity.
- Two starter birds at account creation, with age-based unlock support up to seven total birds: The plan later says unlock thresholds should remain config-driven so the team can slow or pause unlocks if "recognizability, performance, or notebook quality degrade at higher bird counts."
- Server-side simulation tick: The plan says it advances mood and personality "whether or not the client is open," supporting continuity across sessions and preventing the client from owning simulation state.
- Core interactions: return-greeting, listen-in, offer, settle, field notebook: NOT RECOVERABLE FROM PLAN
- Presence accounting using the PRD's three-signal conjunction: The plan's presence design keeps presence "honest," survives packet loss, and prevents clients from "retroactively claiming hours of idle attention."
- Multi-device sync through one server-owned canonical state: The plan uses this to keep state coherent and prevent stale sessions from overwriting or corrupting "canonical personality."
- Accessibility surfaces from day one: The plan wants accessibility acceptance criteria "equal to core feature criteria" and warns against accessibility shipping as fallback.
- Quiet social feature: The plan implements read-only visits, revocation, visit log, and notifications off by default to preserve quiet sociality without visitor writes, co-presence, chat, or host-state mutation.
- Operational telemetry limited to aggregate health and performance metrics: The plan uses aggregate-only metrics to enforce the privacy boundary and avoid per-bird interaction history, per-account drift dashboards, and identity-keyed observability.

### Product behavior decisions that unblock implementation

- Separate `personality_vector` and `recent_presence_reservoir`: The plan does this to satisfy two truths at once: monotonic permanent drift and "quieter after absence." A bird can become "permanently bolder and warmer" while still sounding quieter after time away.
- Deterministic prose compiler for notebook, narration, and captions: The plan says this preserves "consistent naturalist voice," low latency, no raw per-bird interaction history leaving the simulation boundary, and "fully testable copy generation."
- HTTP snapshot polling with revision IDs, ETags, and visibility-triggered refreshes: The plan chooses this because the tick cadence is about once per minute, updates are low-volume, visits tolerate "next snapshot pull," and polling is simpler, easier to harden, easier to cache, and less prone to race conditions than realtime channels.

### Recommended system architecture

- TypeScript monorepo: NOT RECOVERABLE FROM PLAN
- `web-app`: NOT RECOVERABLE FROM PLAN
- `api-service`: NOT RECOVERABLE FROM PLAN
- `simulation-worker`: The plan assigns it canonical tick, notebook generation, weather scheduling, bird unlock scheduling, and deletion processing because those responsibilities belong server-side and should continue whether or not the client is open.
- `shared-domain`: The plan uses shared type contracts, state schemas, phrase templates, and validation rules so API and client share contracts and deterministic prose assets.
- `email-worker`: NOT RECOVERABLE FROM PLAN
- Postgres as system of record: NOT RECOVERABLE FROM PLAN
- Redis or managed queue: NOT RECOVERABLE FROM PLAN
- CDN/edge caching for static assets and immutable motif data: The plan uses caching for static assets and anonymous assets while keeping authoritative aviary state region-local to avoid consistency ambiguity.
- Object storage for generated export payloads and optional internal build artifacts: NOT RECOVERABLE FROM PLAN
- One primary region for canonical data writes: The plan uses one primary region and region-local authoritative state to avoid "consistency ambiguity."
- Server responsibilities for canonical personality, mood, events, tick, notebook, visits, lifecycle, privacy: The rationale is that the server owns canonical state, conflict prevention, privacy workflows, and interaction ordering.
- Client responsibilities for rendering, interpolation, local-only ambient ornaments, WebAudio, screen-reader playback, accessibility UI: The rationale is that clients render from snapshots, synthesize smooth local presentation, and keep ambient ornaments local while not owning canonical simulation state.
- Dedicated scene renderer instead of ordinary DOM animation: The plan says this protects runtime performance for continuous animation and audio.
- Dedicated audio engine using WebAudio plus AudioWorklet: The plan uses this for deterministic scheduling and procedural call synthesis.

### Core data model

- `accounts`: NOT RECOVERABLE FROM PLAN
- `sessions`: NOT RECOVERABLE FROM PLAN
- `magic_links`: NOT RECOVERABLE FROM PLAN
- `aviaries`: NOT RECOVERABLE FROM PLAN
- `birds`: NOT RECOVERABLE FROM PLAN
- `bird_personality_vectors`: The plan's rationale is to store permanent, monotonic, slow-moving trait values separately from temporary expressivity.
- `bird_runtime_state`: The plan uses runtime state for mood, call cadence, greeting weight, offer cooldowns, and listen-in recency so near-term behavior can change without overwriting permanent personality.
- `interaction_events`: The plan uses an append-only event-log model for event ingestion, ordering, idempotency, and processing in ticks rather than client-owned simulation state.
- Allowed interaction event types: NOT RECOVERABLE FROM PLAN
- `notebook_entries`: The plan stores sparse authored observations generated by the simulation worker; `source_signals` exist for "explainability and testing only" and are not surfaced or exported into aggregate telemetry.
- `visit_invites`: NOT RECOVERABLE FROM PLAN
- `visit_sessions`: NOT RECOVERABLE FROM PLAN
- `visit_logs`: NOT RECOVERABLE FROM PLAN
- `export_jobs`: NOT RECOVERABLE FROM PLAN
- `deletion_jobs`: NOT RECOVERABLE FROM PLAN
- Static domain assets as versioned code or immutable content assets: The plan wants semantic versions so snapshots can reference which species asset version the client should render against.

### API surface

- Idempotency keys on all write endpoints: The plan uses idempotency and dedupe to prevent duplicate retries and stale writes from corrupting canonical state.
- Monotonic `state_revision` on state-bearing reads: The plan uses revisioning so clients can request deltas or fresh snapshots from the last seen revision and stay coherent across devices.
- `POST /api/auth/request-magic-link`: NOT RECOVERABLE FROM PLAN
- `POST /api/auth/consume-magic-link`: NOT RECOVERABLE FROM PLAN
- `GET /api/account`: NOT RECOVERABLE FROM PLAN
- `POST /api/account/email-change`: NOT RECOVERABLE FROM PLAN
- `POST /api/account/export`: NOT RECOVERABLE FROM PLAN
- `POST /api/account/delete`: The plan marks pending deletion and terminates future background unlocks while preserving a recovery window; this supports soft delete and account lifecycle privacy workflows.
- `POST /api/account/delete/cancel`: NOT RECOVERABLE FROM PLAN
- `GET /api/aviary/bootstrap`: The plan includes a snapshot and greeting plan so the client can render current state and immediate return greetings on load.
- `GET /api/aviary/snapshot?since_revision=...`: The plan uses 304 when unchanged and snapshot refresh on visibility regain, acknowledged interaction, long frame gaps, and keepalive to keep state coherent without realtime channels.
- `POST /api/aviary/events:batch`: The plan batches bounded interaction events and enqueues them for the next tick so clients submit events, not personality values, mood values, or absolute positions.
- Dropping old presence pings: The plan drops stale pings rather than replaying them "to keep presence honest."
- Accepting non-presence interactions after brief retry windows: NOT RECOVERABLE FROM PLAN
- Clients never submit personality values, mood values, or absolute bird positions: The rationale is server authority over canonical simulation state and conflict prevention.
- `GET /api/notebook?cursor=...`: The plan makes notebook entries paginated and read-only, with "no mutation endpoint," preserving notebook as simulation-generated observation rather than user-authored or edited state.
- `POST /api/visits/invites`: NOT RECOVERABLE FROM PLAN
- `GET /api/visits/invites`: NOT RECOVERABLE FROM PLAN
- `POST /api/visits/invites/{invite_id}/revoke`: The plan uses revocation for host control over visits.
- `GET /api/visit/{token}/bootstrap`: The plan returns a read-only snapshot and allowed host-display metadata with no event-write routes, preserving visitor read-only access.
- `GET /api/visit/{token}/snapshot?since_revision=...`: The plan uses read-only polling and matter-of-fact unavailable payloads so revocation or expiration takes effect on the next polling response.

### Simulation engine design

- Due-aviary scheduler every minute: The plan uses a minute-level scheduler because canonical simulation cadence is slow, about once per minute.
- Idempotent tick jobs keyed by `(aviary_id, tick_window_end)`: The plan uses idempotency to prevent duplicate processing and support catch-up after worker delay.
- Catch-up by iterating missed tick windows in order, bounded per run: The plan says this prevents one stalled aviary from starving the queue.
- Per-tick phases: NOT RECOVERABLE FROM PLAN
- Presence pings only when visible, focused, and recently active: The plan uses the three-signal conjunction to avoid crediting idle or hidden tabs as meaningful presence.
- Server conversion of pings into bounded presence windows: The plan says this keeps presence honest, survives packet loss, and prevents retroactive claims of idle attention.
- Drift function as small, saturating, additive deltas: The plan uses this so traits never decrement due to absence, per-session effect is capped, diminishing returns apply, and change is measurable after one week but felt after about three weeks.
- Offline calibration harness: The plan says to build it before public release to verify the one-week and three-week thresholds against explicit goldens.
- Temporary expressivity reservoir: The plan uses it to influence return greetings, call density, chorus likelihood, notebook frequency, and perch preference so the product can get "quieter after absence" without violating monotonic permanent drift.
- Mood enum plus intensity: NOT RECOVERABLE FROM PLAN
- Mood persistence across tab open and session end: The plan preserves mood so it does not reset on tab open, and time-of-day transitions can soften or intensify between sessions.
- Greeting selection computed on session bootstrap: The plan says not to precompute on tick so the system can incorporate "true absence length and the most recent state."
- Primary greeter plus optional staggered secondary follow-up: The plan uses this to "suppress simultaneous greetings."
- Greeting plan in bootstrap payload: The plan includes motion cue, call cue, and timing offsets so the client can render it immediately.
- Offers as authored interaction types, not free-form inputs: The plan uses this to select immediate reactions, make short-lived runtime changes, contribute small drift, and enforce cooldowns without free-form client state.
- Weather as sparse aviary-level ambient state: The plan says weather exists to "lightly perturb mood and ambience, not to become a feature system."
- Sparse notebook generation: The plan says notebook creation should be "sparse and observation-like," average every few days, with no entry per session, specific observational prose, and no trait numbers or visit-count behavior.

### Sync model and conflict prevention

- Server as sole writer of personality and mood: The plan uses this to protect canonical state from stale clients and corruption.
- Clients write only interaction events: The plan uses this to keep clients from submitting personality, mood, or positions while still enabling interactions.
- Visitors have no write path into host state: The plan uses this to keep visits read-only and out of the host simulation ledger.
- Revisioning with `state_revision`: The plan uses revisions so clients request deltas or fresh snapshots from the last seen revision and remain coherent.
- Polling on initial load, visibility regain, suspend/resume, visible interval, and post-write acknowledgement: The plan says this keeps state coherent while respecting slow tick cadence.
- Event dedupe by `(session_id, client_event_id)`: The plan uses dedupe to handle retries safely.
- Processing by `server_received_at`: The plan uses server receive time for ordering and treats client occurrence time only as supporting context.
- Reject stale session writes: The plan uses this to prevent old sessions from affecting newer canonical state.
- Degraded network behavior without durable offline replay: The plan preserves drift integrity over convenience: last-snapshot animation may continue, but presence is not credited until fresh valid pings arrive and stale pings are dropped.
- Visitor polling with read-only token: The plan uses this so revocation or expiration takes effect on the next response and visitor presence does not emit interaction events.

### Frontend rendering pipeline

- Server-rendered shell and account surfaces with fast hydration: NOT RECOVERABLE FROM PLAN
- First bird visibility inside 500ms: The plan targets this by embedding or edge-delivering the initial snapshot, code-splitting non-essential settings, and rendering the aviary as soon as scene payload is ready.
- Quiet field loading surface and never a spinner: NOT RECOVERABLE FROM PLAN
- Stable scene graph layers: NOT RECOVERABLE FROM PLAN
- Keep birds fully on-screen with responsive spacing: The plan explicitly wants no cropping, widening spacing on desktop and compressing on mobile.
- Bird render packet with species, asset version, perch, pose, seeds, transition and idle weights: The plan says the client uses this packet to synthesize "alive-feeling micro-motion between snapshots" without server streaming every visual step.
- Reduced-motion path using same state packet and swapped animator: The plan uses this so no separate simulation path exists while motion changes to cross-fades, removed drift, and slower day/night shifts.
- Top bar fade and reappear behavior: NOT RECOVERABLE FROM PLAN
- Top bar remains keyboard accessible while visually faded: The plan supports keyboard accessibility as a first-class surface.
- Field notebook side panel or modal sheet, read-only, paginated, keyboard navigable, lowercase naturalist formatting: The plan supports read-only authored observation, viewport-appropriate display, keyboard access, and preservation of naturalist style.

### Audio pipeline

- WebAudio plus AudioWorklet procedural call synthesis: The plan uses this for per-bird procedural calls, deterministic scheduling, and smooth playback without stored long audio buffers.
- Species motif library, timing rules, pitch contours, timbral parameters, caption descriptors: The plan uses these as species defaults so bird instances can apply seeds and personality modifiers.
- Rolling 2-to-4-second scheduler: The plan says this gives smooth playback without storing long audio buffers.
- Listen-in changes only the mix, not the simulation: The plan uses this so focus affects listening presentation while leaving canonical simulation untouched.
- Smooth gain ramps for listen-in: The plan uses ramps so the focused bird rises gradually, other birds reduce to ambient without muting fully, and disengage returns with the same profile.
- Persistent call seed and motif weighting profile: The plan uses these to preserve per-bird identity; drift may alter density and variation but not erase "recognizable signature contours."
- WebAudio fallback to visual aviary and default captions, with no recorded audio fallback: NOT RECOVERABLE FROM PLAN

### Accessibility surfaces

- Shared screen-reader narration composer: The plan uses structured state to generate "slow naturalist prose updates" and avoid exposing trait numbers, raw pose IDs, or debug state.
- Separate ARIA live region queues: The plan separates low-priority ambient narration from higher-priority user-triggered observations such as greeting or offer reaction.
- Narration cadence: NOT RECOVERABLE FROM PLAN
- Captions generated from call grammar inputs: The plan uses the same call grammar inputs as synthesis so captions match the sound envelope and calling bird.
- Caption placement near calling bird with accessible contrast and fade timing: The plan supports accessibility and sound-envelope alignment.
- Keyboard support flows: The plan requires full keyboard access through top bar, modal surfaces, bird focus, listen-in, transient surfaces, offer menu, and notebook.
- Focus rings with contrast across morning, dusk, and night palettes: The plan treats focus styling as "a design deliverable" so focus remains visible in all scene palettes.
- Matter-of-fact system surfaces: The plan uses matter-of-fact copy for auth, error, revocation, unsupported-browser, and settings-system surfaces and explicitly avoids reusing naturalist phrasing there.

### Performance budgets and observability

- Initial JS bundle under 2MB gzipped: NOT RECOVERABLE FROM PLAN
- First bird visible under 500ms: The plan uses this as a performance budget and rollout gate for target devices.
- 60fps idle motion on a five-year-old mid-range laptop: NOT RECOVERABLE FROM PLAN
- No sustained memory growth over a 30-minute session: NOT RECOVERABLE FROM PLAN
- Simulation tick p99 under 5 seconds: The plan uses this as a budget and operational alert threshold.
- Code-splitting for settings, invite management, and export/deletion flows: The plan uses aggressive code-splitting to keep initial load fast and meet bundle/render budgets.
- Immutable CDN caching for species assets: The plan uses this to optimize static asset delivery.
- Bundle analysis gate in CI: The plan uses this to enforce build-time budget controls.
- Animation and audio stress test scenes in CI and nightly builds: The plan uses these to enforce performance and stability for continuous motion and audio.
- Allowed aggregate runtime metrics: The plan uses first-bird timing, load timing, frame aggregates, audio failures, API latency, snapshot sizes, tick lag, and revocation latency as aggregate health and performance metrics.
- Disallowed analytics dimensions and payloads: The plan disallows per-bird interaction history, per-account drift dashboards, model training inputs from notebook or interaction logs, and identity-keyed dimensions to enforce privacy.
- Physical separation of simulation database and analytics pipeline: The plan uses this to ensure only approved aggregate counters and latency histograms reach analytics sinks.
- Automated redaction checks: The plan uses redaction checks to ensure no behavioral event payload reaches analytics.

### Security and privacy implementation details

- Synthetic UUID account IDs outside encrypted account record: The plan supports privacy by avoiding direct identity exposure outside the encrypted account record.
- Encrypt stored emails and invited visitor emails at rest: The plan supports privacy for account and visitor identity.
- Hash tokens at rest: The plan supports security for magic links, invites, sessions, and export downloads.
- Short-lived signed visitor tokens with one-time bootstrap and bounded polling lifetime: The plan supports controlled read-only visits with bounded access.
- Per-device session revocation from settings: The plan supports account control over sessions.
- Soft delete immediately and hard delete after 30 days: The plan supports account lifecycle with immediate soft delete and a recovery window.
- Email exports to the verified address only, no direct browser download without re-authenticated access: The plan protects export delivery through verified identity and re-authenticated access.

### Delivery workstreams

- Platform and domain foundation: NOT RECOVERABLE FROM PLAN
- Simulation and notebook: NOT RECOVERABLE FROM PLAN
- Aviary client and rendering: NOT RECOVERABLE FROM PLAN
- Audio and accessibility: NOT RECOVERABLE FROM PLAN
- Social and account lifecycle: NOT RECOVERABLE FROM PLAN

### Validation strategy

- Personality traits never decrement due to absence: The plan tests this to enforce monotonic long-term drift.
- Ordered event-log replay tests: The plan uses replay tests to verify ordered event-log consumption.
- Catch-up tick tests after worker delay: The plan uses these to verify delayed workers do not break canonical tick processing.
- Calibration goldens for one-week and three-week drift: The plan uses goldens to verify measurable and felt change thresholds.
- Dual-device concurrent session tests: The plan uses these to prevent multi-device sync from corrupting canonical personality.
- Visibility suspend/resume tests: The plan uses these to verify freshness and polling behavior after lifecycle gaps.
- Stale event retry and dedupe tests: The plan uses these to verify retries do not duplicate or corrupt state.
- Revocation propagation tests for active visitors: The plan uses these to verify visit revocation correctness.
- First-bird synthetic tests on throttled devices: The plan uses these to verify first-bird performance on target profiles.
- 30-minute soak tests for memory growth: The plan uses these to verify no sustained memory growth over long sessions.
- Reduced-motion visual regression tests: The plan uses these to verify the reduced-motion visual system remains valid.
- Mobile responsive snapshot tests: The plan uses these to ensure no bird crops off-screen.
- Automated keyboard path coverage: The plan uses this to validate keyboard accessibility.
- Live-region queue behavior tests: The plan uses this to validate narration behavior.
- Caption contrast and placement checks: The plan uses this to validate caption accessibility.
- Screen-reader usability sessions before launch: The plan uses these to validate screen-reader surfaces as designed experiences.
- Notebook and narration tone review: The plan uses review against a tone rubric to protect editorial quality.
- Banned-language regression suite: The plan uses this to prevent achievements, streaks, scores, or trait numbers from entering copy.

### Rollout plan

- Phase 0 internal prototype with two birds, no visits, real auth and sync, full simulation, notebook, audio, captions, and reduced motion: The plan uses this gate to verify drift calibration, first-bird performance, and telemetry privacy before broader exposure.
- Phase 1 employee dogfood with visits, export/delete, and accessibility feedback loop: The plan uses dogfood to monitor tick lag, audio fallback rate, notebook quality, and visit revocation correctness.
- Phase 2 invite-only beta with bird unlock logic behind config and visits off by default: The plan uses beta to exercise external use while keeping visits default-off and host notifications off; seeded backdated accounts exercise third through seventh bird behavior before general users reach those ages.
- Phase 3 public v1 with full scope and config-driven unlock thresholds: The plan keeps thresholds configurable so the team can slow or pause unlocks if recognizability, performance, or notebook quality degrade at higher bird counts.

### Day-one instrumentation and operating thresholds

- First-bird render percentile distributions: The plan instruments these to monitor first-bird performance.
- Tick queue lag and duration: The plan instruments these to monitor simulation health and alert on tick p99 over 5 seconds.
- Snapshot payload sizes: The plan instruments these to detect p95 size regression beyond budget.
- WebAudio init success rate: The plan instruments this to monitor audio fallback risk.
- Unsupported-browser rate: NOT RECOVERABLE FROM PLAN
- Reduced-motion usage rate as aggregate count: The plan records aggregate usage without per-account behavioral detail.
- Active visit count and revocation latency aggregates: The plan monitors visit scale and revocation propagation.
- Operational alerts: The plan uses alerts for tick p99, first-bird p95, audio fallback rate, snapshot size regression, and memory soak regression to catch budget and reliability failures.

### Principal risks and mitigations

- Drift calibration misses the emotional target: The plan mitigates birds feeling unchanged or visibly changing between sessions with a calibration harness, config coefficients, and dogfood review of notebook output alongside drift traces.
- Multi-device sync corrupts canonical personality: The plan mitigates stale overwrites and event replay with no client personality writes, dedupe, server-receive ordering, tick idempotency, and dual-device concurrency testing.
- Audio feels canned or indistinct: The plan mitigates repetition and blurred bird identity with persistent seeds, motif variance, blinded listening tests, and configurable bird unlock thresholds.
- Accessibility ships as fallback instead of designed surface: The plan mitigates raw labels and "animation off" by using shared prose, a dedicated reduced-motion system, and equal accessibility acceptance criteria.
- Privacy boundaries erode over time: The plan mitigates event leakage with aggregate metric allowlists, simulation DB isolation, and schema review gates for observability changes.
- Notebook quality becomes generic: The plan mitigates templated, repetitive, or gamified prose with editor-reviewed phrase libraries, sparsity controls, banned phrasing tests, and observation triggers tied to meaningful state patterns.

### Recommended implementation sequence

- Account, session, aviary, bird, and event schemas first: NOT RECOVERABLE FROM PLAN
- Server-authoritative snapshot and event API second: The plan puts canonical state ownership and event ingestion before simulation and client polish.
- Simulation worker with reservoir, mood, drift, and notebook generation third: The plan brings hardest correctness constraints in before polish.
- Scene renderer with quiet field bootstrapping and responsive layout fourth: The plan adds the visual experience after canonical simulation exists.
- Audio synthesis and listen-in mixing fifth: The plan adds procedural audio after renderer basics.
- Narration, reduced motion, captions, and keyboard support sixth: The plan makes polish surfaces first-class rather than bolt-ons.
- Visit invites, read-only visitor bootstrap, and revocation seventh: The plan adds quiet social surfaces after core simulation and accessibility.
- Exports, deletion, observability, and rollout controls eighth: The plan hardens lifecycle, privacy, operations, and launch controls last.
