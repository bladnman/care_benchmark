## System-level intent

- The aviary should feel "continuously alive" rather than like an app that starts and stops. This appears in the product contract, the "canonical server-side simulation tick" that runs "whether or not a client is open," the first-frame rule that there is "No spinner, no wake-up animation," and the exit criterion that "two starter birds feel alive immediately on load."

- Attention is rewarded, but absence is not punished. The plan says the product "rewards attention without punishing absence," excludes "hunger, decay, death, or punitive neglect mechanics," defines drift as "monotonic toward expressive," and says absence becomes "lower greeting likelihood" and "reduced call participation," "not negative vector drift."

- The product must avoid game loops, pet-care obligations, and social-network dynamics. This is stated directly in the core affective contract and repeated through exclusions of "streaks, scores, badges, XP, counters, leaderboards," "Tamagotchi-style" mechanics, "shared aviaries," "chat," "comments," "profiles," and "public feeds."

- Server authorship is a hard boundary. The plan says "Server is the only writer of bird personality vectors and canonical mood state," "Clients append events," "they never submit authoritative bird state," and "No last-write-wins state merges are allowed for personality or mood."

- Personality continuity is treated as the worst failure boundary. The planning assumptions say "personality loss is the worst failure mode"; the sync risk says device races or replay issues could "silently erase personality evolution"; the mitigation is append-only events, server-only vector writes, locks, idempotent ingest, and replayable audit cursors.

- Privacy is architectural, not just policy. The plan says the "Privacy boundary is enforced architecturally," simulation data and telemetry use "separate pipelines and schemas," analytics receive "only aggregate metrics," and there should be "No telemetry that reconstructs a specific user's relationship with their birds."

- The product voice is quiet, sparse, and naturalist. The plan requires that the aviary "notices the user without announcement UI," notebook prose uses "naturalist lowercase present-tense style," notebook generation enforces "sparsity," and system errors are "matter-of-fact" rather than naturalist prose.

- Accessibility surfaces are part of the product, not fallback copies. The plan calls them "first-class product features," says reduced-motion is "a full experience, not a passive CSS flag," and readiness requires captions, narration, keyboard paths, and reduced motion to deliver "the actual product rather than a degraded surrogate."

- The interface should be calm and scene-first. The top bar is "sparse," fades "near-transparent," has "No inline UI chrome in scene," and audio mixing should be "subtle enough to preserve calm."

- The architecture should stay small enough for one team while keeping boundaries around simulation and privacy. The planning assumptions say to "minimize service count while preserving hard boundaries," and the recommended stack "keeps the platform small enough for one team while still separating interactive API concerns from simulation execution."

## Per-feature whys

### Scope and product contract

- Browser-only product for modern Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN

- Email magic-link authentication: NOT RECOVERABLE FROM PLAN

- One aviary per account: NOT RECOVERABLE FROM PLAN

- Two starter birds at account creation: the plan ties this to immediate aliveness, with readiness requiring that "two starter birds feel alive immediately on load" and rollout beginning with bird count "capped at two while drift/audiovisual calibration stabilizes."

- Age-based expansion up to a hard cap of seven birds: the why is calibration, recognizability, and performance. The rollout enables "age-based third-bird offers" only after greeting, call recognizability, and notebook sparsity are healthy, then ramps "from 2 to 3 to 5 to 7" while validating chorus recognizability and performance.

- Canonical server-side simulation tick: it keeps mood, drift, time-of-day behavior, weather, greetings, and notebook-worthy events moving "whether or not a client is open," and preserves server authorship of personality and mood.

- Read-only field notebook: it records "noteworthy moments, not from every session," in "naturalist" prose, while sparsity thresholds and rate limits avoid spam.

- Passive presence: it is the dominant positive input for drift, so attention can matter without introducing meters or obligations.

- Listen-in: it is "second-order and bird-specific," and the mix is meant to feel like "attentive listening, not channel switching."

- Offer: offers apply "small capped adjustments," supporting positive interaction while ensuring "No single session" moves visible behavior enough to infer a meter.

- Settle: it ends presence but "does not impose negative trait change," preserving the no-punishment absence contract.

- Notebook browsing: NOT RECOVERABLE FROM PLAN

- Account/settings surface: NOT RECOVERABLE FROM PLAN

- Accessibility settings and surfaces: the plan treats screen-reader narration, reduced motion, captions, keyboard support, and contrast as "first-class product features" and as "the actual product rather than a degraded surrogate."

- Multi-device sync: the why is that canonical server state plus append-only interaction events prevent conflicts and preserve personality history across devices.

- Invite-by-email read-only visits: the feature is "quiet optional social," "off by default," "revocable," and "expiring" so the product can allow visits without shared aviaries, co-presence, visitor interactions, or social-network dynamics.

- Aggregate operational telemetry and performance instrumentation only: this protects privacy by excluding "per-bird interaction history" from analytics and forbidding dashboards keyed by account, bird, or interaction history.

- Silence plus captions when audio synthesis is unavailable: the plan prefers this to "introducing a lower-fidelity recorded-audio path."

### System architecture

- Single authoritative application platform with `web-client`, `api-app`, and `simulation-worker`: the why is to keep the platform "small enough for one team" while separating interactive API concerns from simulation execution.

- `web-client`: it renders the aviary, synthesizes audio, captures qualified interaction events, interpolates between snapshots, and presents notebook/settings/accessibility surfaces while leaving authoritative state to the server.

- `api-app`: it owns auth flows, snapshot reads, event ingestion, visit-link resolution, account/settings mutations, export/delete workflows, and prose endpoints where needed, keeping HTTP concerns stateless.

- `simulation-worker`: it owns scheduled ticks, drift, mood, weather, notebook-entry generation, stale-session cleanup, and snapshot materialization so canonical simulation continues independently of clients.

- Primary relational database: the plan uses it for durable accounts, birds, snapshots, notebook entries, invites, visit logs, and the durable event log.

- Job queue / scheduler: it supports simulation ticks, email sends, exports, deletion windows, per-account scheduling, and retry semantics.

- Object storage, CDN/edge delivery, and email provider: object storage keeps exports and compact assets out of bundle code; CDN/edge supports HTML, JS, CSS, assets, and snapshot bootstrap payloads; email supports magic links, invites, exports, and deletion confirmation.

- TypeScript + React frontend, TypeScript backend, PostgreSQL, managed queue, transactional email: this stack keeps shared domain models between API and worker packages and keeps the platform small for one product team.

- Initial load path optimized for "first bird visible": this serves the affective contract that the aviary is already alive, with at least one bird placed quickly rather than waiting for full hydration.

### Domain and data model

- Synthetic account identifiers and encrypted verified email: the why is privacy, with synthetic UUIDs used everywhere outside the encrypted account email field.

- Locale and preferred timezone: these feed local day-part derivation so time-of-day behavior matches the account timezone and local clock.

- Current session registry: it exists for revocable device sessions.

- Bird personality vector fields: the plan makes these canonical server data with durable migrations and recovery safeguards because "personality loss is the worst failure mode."

- Bird call signature seed / motif library references: these let birds of the same species remain individually recognizable through stable seed variation.

- Interaction event log: clients append presence, listen-in, offer, settle, visibility, and session events so the worker can interpret drift later from ordered evidence.

- Event payload constrained to non-PII operational data: the why is the privacy boundary.

- Presence session accumulator: it derives qualified active presence from visibility, focus, recent input, and accumulated seconds, supporting presence-time as the dominant drift input.

- Notebook source tags and significance score: tags support internal generation logic, testing, and tuning; significance preserves sparsity and avoids spam.

- Invite and visit records: they support revocable, expiring, read-only visits and record approximate duration without affecting host drift.

- Materialized aviary snapshot: it makes snapshot reads cheap and includes render, ambient, audio, narration, and caption inputs needed for immediate client presentation.

- Data retention rules: canonical state persists until deletion, event logs stay long enough for replay and audits but are bounded, and analytics receives only aggregate metrics with no account or bird dimensions.

### API surface

- `GET /aviary/snapshot`: it returns current materialized state plus minimal bootstrap data for immediate render.

- `GET /aviary/notebook`: it provides paginated, read-only notebook entries, newest first, matching the read-only field notebook surface.

- `GET /aviary/birds`: it exposes names/species/settings surfaces only and withholds raw personality numbers from product clients.

- `POST /aviary/events`: batched append-only ingest acknowledges quickly while leaving drift interpretation to the tick; validation checks event type, session, timestamps, and bird ownership.

- `POST /aviary/presence`: the plan includes it only if presence pings need tighter validation or rate limiting than general events.

- Visit invite and read-only visit endpoints: they resolve host snapshot authorization, support revocation, record duration only, and do not affect host drift.

- Accessibility/settings endpoints: they support explicit settings for accessibility surfaces that the plan treats as first-class product features.

- Export and delete endpoints: they support export generation, deletion confirmation, a 30-day soft-delete recovery window, cancellation, and hard-delete cleanup.

- Email-change endpoints: NOT RECOVERABLE FROM PLAN

- Logout and session revocation endpoints: they support session revocation, which is immediate for future requests.

### Simulation engine design

- Roughly 60-second per-account tick cadence with jitter: ticks continue regardless of client connectivity, and jitter avoids thundering herds.

- Per-aviary lock and atomic persistence: the why is to prevent concurrent drift writes and keep canonical state, event cursors, and snapshots consistent.

- Additive deltas over bounded scalar traits: this allows regular presence and interactions to become measurable without unbounded or meter-like movement.

- Presence-time, listen-in, offers, and settle as drift inputs: presence is dominant, listen-in is bird-specific, offers are capped, and settle has no negative trait change.

- Monotonic drift toward expressive: traits only increase from positive interaction and never decrease due to neglect, matching the no-punishment product contract.

- Ambient quietness on absence: reduced greeting likelihood, reduced call participation, and mood defaults make absence quiet without negative vector drift.

- One-week measurable and three-week perceptible calibration targets: these guard against birds feeling static or feeling gameable, and prevent users from inferring a meter from one session.

- Mood engine: weighted transitions among wary, content, curious, drowsy, alert, and night variants make mood respond to offers, listen-in attention, time of day, weather, nearby bird interactions, and personality modifiers while persisting across sessions and devices.

- Greeting selection: a primary greeter is chosen on visible return from boldness, social warmth, mood, and absence duration; secondary greetings are staggered so they are never synchronized.

- Precomputed greeting plan in snapshot hints: it lets the client render greetings within the first seconds without extra round trips.

- Call grammar runtime: motif libraries, stable seed variation, mood, vocal-frequency traits, and chorus timing make calls individually recognizable, while captions derive from the same call plan so text matches actual played calls.

- Notebook generation: the worker generates entries from noteworthy moments, constrained to "naturalist lowercase present-tense style," with significance thresholds and per-aviary rate limits to avoid spam.

- Weather and day-part: day-part derives from account timezone and local clock; rare, short-lived weather windows add low-intensity mood effects; night stays "quiet but not dead."

### Sync and multi-device model

- One server-authored snapshot lineage: clients always read canonical state, and event ordering is authoritative.

- No last-write-wins merges for personality or mood: this prevents device races from overwriting personality history.

- Snapshot pulls on initial load, visibility regain, long frame-gap recovery, keepalive, and significant user actions: the why is to stay current and reflect actions quickly.

- Snapshot version numbers: they skip redundant state application.

- Local interpolation between snapshots: it smooths render positions and motion phases without client-side authorship.

- Session tokens scoped per device and idempotent event ingest: these prevent conflicts and replay issues.

- Last processed event cursor and per-aviary tick lock: they prevent duplicate or concurrent drift writes.

- Invite revocation and session expiry through snapshot authorization checks: they propagate access changes on the next pull.

- Failure handling: snapshot failures show "matter-of-fact system error" surfaces, event append failures retry with bounded retention without client-side drift, and expired sessions stop presence capture and prompt re-auth.

### Frontend rendering pipeline

- One scene renderer: it owns birds, perches, ambient layers, top bar chrome, and accessibility overlays as one aviary surface.

- Snapshot bootstrap for immediate drawing: it includes enough pose and motion state to draw birds without waiting for non-critical surfaces.

- Initial quiet field, no spinner, no wake-up animation, no fade-from-static sequence: this protects the illusion that the aviary is already alive.

- Birds enter already mid-action: snapshot motion phase avoids an app-startup reveal.

- Separation of canonical state from render ornaments: bird perch, pose family, active greeting, mood, and weather stay canonical, while leaf drift, feather drift, and parallax remain ornamental.

- Bird motion clips or procedural pose states: preen, scan, tilt, shuffle, call posture, and settle posture express mood and action family.

- Render interpolation without teleporting: it bridges snapshots smoothly.

- Front/middle/back perch zones: they create scale, opacity, and depth cues without requiring 3D scene complexity.

- Sparse top bar above scene: account/settings, accessibility, notebook, offer, and settle affordances stay outside the scene, fade near-transparent after inactivity, and return on cursor or keyboard activity.

- Responsive behavior: all birds remain in frame across mobile and desktop, with wide screens increasing perch spacing and narrow screens compressing without cropping.

- Reduced-motion renderer variant: it uses pose cross-fades, removes drifting ornaments, slows palette transitions, and keeps simulation inputs identical so only presentation changes.

### Audio pipeline

- Bounded WebAudio engine with reusable nodes and motif generators: this supports procedural calls while protecting memory and CPU.

- Stable per-bird synthesis profile seeded from species and bird id: the why is individual recognizability.

- Chorus mixing: overlapping calls use subtle pan, volume, and spatial hints to preserve calm.

- Listen-in mix: focusing a bird ramps that mix up and others down to ambient, "never to zero," so it feels like attentive listening.

- Shared audio-state machine for keyboard and pointer: it keeps interaction modes consistent.

- Audio fallback to silence plus captions: if WebAudio init fails or permission is unavailable, call timing remains visual and textual without a pre-recorded audio path.

- Audio performance safeguards: reused nodes, bounded concurrent calls, and monitoring protect recognizability, CPU, audio context health, and voice-node growth.

### Accessibility surfaces

- Screen-reader narration: slow-cadence naturalist narration from snapshot state and prioritized events gives idle and prompt updates without flooding screen-reader queues.

- Captions: per-call text comes from the actual procedural call plan, is positioned near the calling bird, meets AA contrast, transitions calmly, and auto-enables when audio fallback engages.

- Keyboard and focus: predictable top-bar tab order, arrow-key bird traversal, Enter for listen-in, Escape to exit, and keyboard-reachable offer and settle make the aviary operable without pointer use.

- Focus rings for bright and dim aviary states: they keep focus visible across scene conditions.

- Reduced motion and contrast: the plan respects `prefers-reduced-motion`, allows explicit override, requires WCAG AA text, and treats reduced motion as a full experience.

- Accessibility QA: manual screen-reader testing and snapshot-driven narration tests ensure prose remains naturalist and non-metric.

### Privacy, security, and compliance shape

- Synthetic UUIDs and encrypted email: the why is to keep identity out of ordinary identifiers and fields.

- Logical separation of simulation database and operational telemetry stores: this enforces the privacy boundary.

- Exclusion of per-account and per-bird event streams from analytics warehouse ingestion: this prevents analytics from reconstructing a user's relationship with birds.

- Magic links that expire after 15 minutes and are single-use: the articulated why is security shape.

- Immediate session revocation for future requests: it makes session control effective.

- Account deletion soft for 30 days, then hard-delete with export/storage cleanup: this provides a recovery window and eventual cleanup.

- JSON snapshot export via emailed verified link: it ties export delivery to verified email.

### Performance budgets and observability

- Initial JS under 2MB, first bird visible under 500ms, 60fps idle motion, no 30-minute memory growth, and tick p99 under 5 seconds: these budgets preserve first-bird immediacy, calm motion, long-session stability, and timely simulation.

- Aggregate-only RUM: navigation timing, first-bird timing, frame pacing, audio errors, and snapshot latency can be observed without account or bird dimensions.

- Worker metrics: tick queue depth, duration, lock contention, notebook generation rate, and invite email failures expose operational health.

- Synthetic browser probes: signed-out shell and synthetic signed-in accounts verify real user paths from key geographies.

- What not to measure: the plan excludes account, bird, interaction-history dashboards, population ranking, comparative drift, and telemetry that reconstructs a specific relationship.

### Delivery plan, rollout plan, testing strategy, risks, and readiness

- Phase A foundations: auth, account model, sessions, aviary/bird schema, event log, starter birds, snapshot materialization, and tick scheduling come first so state and authorship foundations exist before richer behavior.

- Phase B core simulation: presence qualification, drift, mood, day-part, weather, greeting, and notebook work are paired with deterministic tests and one-week/three-week calibration harnesses.

- Phase C core client: first-frame shell, scene renderer, top bar, bird focus, interpolation, matter-of-fact system surfaces, interactions, notebook browsing, and accessibility settings establish the usable browser experience.

- Phase D audio and accessibility: procedural WebAudio, captions, narration, reduced-motion renderer, keyboard polish, and compatibility/performance passes are grouped because they must preserve product feel across supported browsers.

- Phase E social and lifecycle: invites, read-only visit sessions, visit log, export, deletion recovery, and account settings come after the core aviary, keeping social and lifecycle features contained.

- Phase F hardening and launch prep: privacy verification, migration rehearsal, load tests, synthetic monitoring, calibration tuning, and staged rollout prepare production-like readiness.

- Internal dogfood and closed beta: the plan uses synthetic and employee accounts first, then a fixed small cohort capped at two birds while drift and audiovisual calibration stabilizes.

- Age-based third-bird offers and maximum bird ramp: unlocks wait for healthy greeting, call recognizability, and notebook sparsity metrics, then ramp from 2 to 3 to 5 to 7 while validating chorus recognizability and performance.

- Day-one instrumentation: auth rates, snapshot latency, first-bird timing, tick health, audio fallback, accessibility adoption, screen-reader reports, and invite success rates observe launch-critical surfaces.

- Operational playbooks: if tick latency rises, pause new bird unlocks before changing the core experience; if audio errors spike, force silent-caption mode; if notebook density rises, adjust significance thresholds centrally.

- Simulation correctness tests: deterministic replay, monotonic drift properties, no negative-trait movement on neglect, and concurrent-device conflict tests directly cover drift and sync risks.

- Client, end-to-end, and performance tests: visual snapshots, interaction tests, accessibility tests, signup, multi-device validation, visit flows, export/deletion recovery, bundle gates, first-bird budget checks, and soak tests cover product readiness.

- Drift calibration mitigation: calibration harnesses, synthetic visit patterns, and rollout gates address the risk that birds feel static or gameable.

- Sync correctness mitigation: append-only events, server-only vector writes, per-aviary locks, idempotent event ingest, and audit cursors address the risk of erased personality evolution.

- Audio uncanniness mitigation: species motif libraries, stable per-bird seeding, recognizability tests, capped concurrent calls, and staged bird-count ramp address repetition, wrong synthetic feel, and mushy chorus.

- Accessibility regression mitigation: reduced-motion and narration are dedicated workstreams with acceptance criteria tied to product feel, not mere compliance.

- First-frame illusion mitigation: snapshot bootstrap, quiet-field shell, and the no-spinner rule keep loading behavior from revealing "app startup."

- Privacy boundary mitigation: schema separation, code review checklists, telemetry allowlist, and automated log scanning guard against routing per-bird interaction data into telemetry or logs.

- Notebook quality mitigation: constrained templates, significance scoring, cadence throttles, and editorial review reduce repetitive, generic, or too-frequent prose.

- v1 readiness criteria: the team must demonstrate immediate aliveness, canonical simulation across devices and absences, measurable and perceptible drift without trait math, sparse specific naturalist notebook entries, full accessibility paths, no gamification or accidental social-network surfaces, and verified privacy and telemetry boundaries.
