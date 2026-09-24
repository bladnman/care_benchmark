## System-level intent

- Canonical server truth over client-side ownership. This appears in the product invariants ("the server is the only writer of personality and mood; clients render snapshots and submit events"), the architecture section ("The database is authoritative"), the API flow ("The scheduled tick remains the only canonical simulation writer"), and the rendering section ("the server tick owns continuity"). The intent is that clients present and request, while state, sequencing, drift, and mood are advanced in one serialized place.

- Account-private, privacy-minimal operation. The plan repeats "account-private initial snapshot," "personalized snapshots must be private," "synthetic UUIDs," "Encrypt email," "never raw movement or key values," and "RUM carries no account dimension, bird ID/state, event payload, email, or interaction history." Telemetry, logs, analytics, visits, and presence are all shaped to avoid exposing account behavior.

- Gentle expressiveness without distress. The core product invariant is that "drift is one-way toward expressiveness and neglect never lowers traits or produces distress." The simulation section repeats this with "Apply only nonnegative deltas" and "Absence does not become distress." The risks section rejects changes that make absence lower traits or make "one session legible as stat movement."

- Naturalist prose instead of game language. The plan calls for "concise, specific, lowercase naturalist prose" in the aviary, notebook, narration, captions, and offer prompts, while banning a "welcome toast, badge, streak, arrival banner, or visit badge." Notebook guidance rejects generic "achievement" copy and session or visit summaries.

- A quiet one-screen scene, not a configurable or gamified surface. The boundary says the aviary has "no scrolling/panning or embedded scene controls" and excludes "scene customization," "public ranking," and "gamification." The client section reinforces a "one-screen, responsive horizontal scene" where "users cannot arrange" birds.

- Stable bird identity with hidden mechanics. This shows up in "bird IDs survive rename and migrations," server-only "five-trait personality vector," hidden exact values, stable "species motif identity and per-bird call seed," and recognizable call signatures. Identity is allowed to persist; internal numbers are not exposed.

- Procedural sensory output from shared structured events. The plan says "calls are generated procedurally, never downloaded loops," and later specifies "a structured call event" so "audio, captions and narration derive from the same generated call." The audio section repeats that audio and captions "share one call event source to prevent mismatches."

- Accessibility is v1 release scope, not a later add-on. The plan includes "reduced-motion rendering, captions" in v1, calls reduced motion "a designed alternate renderer," says "Ship accessibility with v1," and later states narration, settings, screen-reader and contrast review are "release scope, not post-launch accessibility work."

- Reliability and performance are launch gates, measured operationally. Architecture calls for leases, revision checks, atomic commits, migration and rollback procedures. Performance budgets are explicit release gates, and rollout pauses on auth errors, cross-account access, tick latency, rendering budget, and audio failure. The plan insists on operational measurements while saying "Track no engagement or ranking metrics."

- Visits are explicit, read-only, and non-present. Across product, data, API, rollout, and risks, visitors are "read-only," "never contribute presence or drift," use "one-time visitor link" access, have revocation and expiry, and have "no badge." Optional notification is "off by default."

## Per-feature whys

### 1. Product boundary and invariants

- Browser-only aviary: NOT RECOVERABLE FROM PLAN

- One user and one canonical aviary per account: the plan ties this to canonical state by requiring one unique account-owned `Aviary`, saying the client "never merges two aviary states," and making launch readiness depend on state being "identical across two signed-in devices."

- Two system-selected starter birds with user-assigned and renameable names: NOT RECOVERABLE FROM PLAN

- Birds added as the aviary ages: the plan later says later offers are based "only on aviary age" and rollout should configure "age-based new-bird offers" without visit counters, so added birds are meant to avoid visit count or score mechanics.

- Hard maximum of seven birds: NOT RECOVERABLE FROM PLAN

- Animated horizontal scene: the client pipeline describes a "one-screen, responsive horizontal scene" with front, middle, and back perch zones so the aviary remains a single scene with no pan, zoom, scroll, or embedded scene controls.

- Return-greeting: the risks section says to "stagger return greetings" and avoid "repetitive canned greetings," so the articulated reason is to keep greetings from feeling canned while preserving the quiet bird identity.

- Presence accounting: the plan says to "never equate an open tab with presence," to send heartbeats only while visibility, focus, and recent activity hold, and to make presence time the dominant drift signal without collecting raw input.

- Listen-in: listen-in contributes more to "that bird's social warmth and vocal frequency" and ramps the selected bird up while other birds stay ambient, so its reason is both individual interaction drift and focused listening without silence.

- Offers: accepted offers "nudge curiosity," approaching or receiving an offer can nudge boldness, and offer reactions are made visible promptly while the tick remains canonical. Later bird offers are age-based, not visit-count or score-based.

- Settle: settlement "ends presence and quiets mood but adds no directional drift," gently lowers calls, and prevents settled/hidden/idle time from becoming eligible presence.

- Sparse read-only field notebook: observations are only for "noteworthy aviary events and sparse quiet observations," about one every few days, grounded in actual facts, and explicitly not session logs, vector deltas, visit-frequency summaries, or achievement copy.

- Magic-link accounts: tokens are one-time, expire in 15 minutes, have per-email rate limits, and support matter-of-fact expired/replayed errors, so the reason given is bounded, replay-resistant sign-in without exposing email-derived keys.

- Account export: export is a "point-in-time JSON snapshot" generated on demand and emailed through a "short-lived download link" to the verified address, so the reason is account data access bounded by verified delivery and link lifetime.

- Account deletion: soft deletion gives a 30-day recovery window, while hard deletion removes all account-linked stores and telemetry identifiers; the plan says deletion should be "reliable and auditable" with an "honest boundary" for backups.

- Multi-device sync: the client keeps only the latest snapshot revision and never merges states, while the server is canonical; launch readiness requires state identical across two signed-in devices.

- Optional per-invite visits: visits are designed for named-email, one-time, read-only snapshot access with expiry, revocation, transparent host log, and no contribution to host presence, drift, analytics identity, badges, or notifications by default.

- Narration: narration is derived from "the same snapshot as the scene," throttled and coalesced, and gives prompt but restrained priority to greeting, accepted offer, and settle so announcements do not overlap or become high-frequency.

- Reduced-motion rendering: it is a "designed alternate renderer" that replaces flight and drift with still/preen poses and cross-fades while not pausing bird drift, mood, or calls, honoring `prefers-reduced-motion` with a settings override.

- Captions: captions come from actual motif parameters and the shared call event source, appear near the calling bird, and exist so audio and captions do not mismatch; if WebAudio fails, captions turn on by default.

- Operational performance telemetry: the plan measures page/load timing, frame timing, audio-context failures, request errors, and tick latency so release gates and alerts can work, while RUM remains aggregate and excludes account, bird, event, email, and interaction history.

- Concise lowercase naturalist prose in aviary surfaces: the articulated reason is product voice consistency across aviary, notebook, narration, captions, and offer prompts, separate from "direct matter-of-fact system copy" for sign-in, account, settings, accessibility, and errors.

- Ban on welcome toast, badge, streak, arrival banner, and visit badge: the plan grounds this in its non-gamified, non-social product boundary and repeats "no badge" for visit logs and no "achievement" copy for notebook entries.

### 2. Architecture and service boundaries

- Web client, authenticated API, relational canonical-state store, and background simulation worker: these boundaries keep the browser rendering snapshots and submitting events while the database and worker own canonical state, ticks, retries, and account-scoped serialization.

- Durable queue or database-backed job table: it schedules ticks and retries so the worker can advance aviaries independently of connected clients.

- Account-scoped lease/lock and revision check: the stated reason is to prevent "two workers from advancing one aviary concurrently."

- Atomic state revision and consumed-event cursor commit: the reason is that bird/aviary changes and consumed event sequence advance together, avoiding lost or double-applied events.

- Compact private initial snapshot with immediate render: it lets the browser "render immediately" from an account-private snapshot while avoiding shared cache of personalized state.

- CDN for static shell/assets but private personalized snapshots: static assets may be shared, but snapshots are "private and keyed by account/session, never shared-cacheable across accounts."

- Transient client rendering/audio state only: the reason is that the client "never merges two aviary states" and relies on the latest snapshot revision rather than becoming a state authority.

- Separate server modules for identity/auth, aviary state, snapshots, event intake, simulation, notebook generation, and visit authorization: NOT RECOVERABLE FROM PLAN

- Analytics/RUM pipeline with no read path to simulation tables: the reason is to keep operational measurements aggregate and separated from account behavior and simulation state.

- Small architecture decision record for database, queue, session-token format, and deployment platform: the plan places this before account data is admitted, so the reason is to settle those choices together with migration and rollback procedures.

- Boring managed components with transactions, encrypted storage, backups, regional edge delivery, and worker scheduling: the plan prefers these because they satisfy durability, privacy, delivery, and scheduling needs without adding a streaming platform unless scale measures require it.

- Schema migration and rollback procedures before account data: the reason is to admit account data only after storage evolution and rollback are established.

### 3. Persistent data model

- Synthetic UUIDs for account identifiers in every table, message, partition, log, and metric: the reason is to avoid email-derived keys and keep identifiers synthetic across storage, messages, logs, and metrics.

- Encrypted email on the account record, used only for sign-in, verified account changes, and required mail delivery: the reason is to restrict email to required account operations and avoid using it as an identifier.

- Account settings for captioning, reduced motion override, and visit-notification opt-in: NOT RECOVERABLE FROM PLAN

- Per-device revocable sessions: the plan says session tokens are "per-device and revocable" and later requires session list/revoke, so the reason is account control over device sessions.

- MagicLink records with token digest, expiry, consumption, and rate-limit metadata: the reason is one-time, 15-minute magic-link sign-in with replay and rate-limit handling.

- Aviary record with revision, tick cursor, last presence end, settled state, and event sequence cursor: the reason is to make the aviary's canonical revision, tick progress, presence accounting, and consumed event position persistent.

- Transactional initialization of two birds and the initial scene: the reason is that a new account receives its starter birds and scene together as a stable initial state.

- Bird stable UUID, species, display name, server-only traits, mood timer, perch/action/call state, adoption timestamp, and next-bird eligibility: the reason is stable identity across rename and migration, hidden personality mechanics, persistent mood/action rendering, and age-based bird eligibility.

- Bounded normalized traits and versioned seed/range configuration: the reason is server-side tuning without exposing exact values or rewriting history.

- InteractionEvent sequence, idempotency key, server receipt time, minimal payload, and processing cursor: the reason is to drive that account's simulation in order, support duplicate-safe processing, and avoid raw pointer paths or raw key data.

- Append-only InteractionEvents until retention/compaction policy exists: the plan says these events exist "only to drive that account's simulation," so append-only retention protects replay until safe compaction is defined.

- NotebookEntry records with observed time, structured observation kind, bird references, generated prose, and generation/version metadata: the reason is to preserve read-only notebook history while grounding prose in actual bird/time/scene facts and generation version.

- Read-only notebook history preserved indefinitely while account exists: NOT RECOVERABLE FROM PLAN

- Invite records with encrypted invitee email, one-time token digest, 30-day unused expiry, revocation, and first-use times: the reason is named-email, one-time, revocable, read-only visitor access.

- VisitSession / VisitLog with coarse duration and visitor email reference: the reason is a host transparency log while avoiding host presence, simulation, or analytics identity links.

- DeletionJob with scheduled hard-delete time and progress: the reason is to make the 30-day recovery window and final purge "reliable and auditable."

- Hard deletion across account, bird, event, notebook, invite, session, export artifacts, and per-account telemetry identifiers: the reason is complete account-linked purge at expiry, with backup expiry behavior explicitly bounded.

### 4. API and event flow

- Versioned, authenticated, idempotent mutating routes scoped from session account UUID: the reason is to avoid accepting a client-selected account ID and to handle duplicate mutations safely.

- `POST /v1/auth/magic-links` and `POST /v1/auth/sessions`: the reason is per-email rate-limited link request, one-time token consumption, device session creation, and clear expired/replayed-link errors.

- Session list/revoke, new-email verification, export request, deletion, and recovery endpoints: NOT RECOVERABLE FROM PLAN

- `GET /v1/aviary/snapshot`: the reason is to return only renderable canonical scene state, revision, tick age, transitions, and event cursor while excluding personality values and private interaction history.

- Revision/ETag snapshot responses: the reason is low-frequency visible-tab keepalive and revision-aware refresh without re-sending unchanged state.

- Snapshot pulls on first open, visibility return, long frame gap, and keepalive: the reason is to rejoin server-owned continuity after startup, hidden periods, and suspended frames.

- `POST /v1/aviary/events` batch with client idempotency keys and advisory client time: the reason is narrow typed event intake where server receipt time and sequence order govern simulation.

- Event validation for bird membership, cooldowns, offer types, and session ownership: the reason is to enforce canonical account and bird constraints before the simulation consumes an event.

- Durable receipt and canonical revision from event submission: the reason is to acknowledge whether the event was accepted, duplicate, expired, cooldown, or unavailable against canonical state.

- Prompt offer or settle reaction through the serialized tick routine: the reason is to make reactions visible without waiting a full minute while preserving the scheduled tick as the only canonical simulation writer.

- Presence heartbeats with visibility, focus, recent activity boolean/time, and bounded interval only: the reason is privacy-preserving presence that never stores raw movement or key values.

- Heartbeats only while all three eligibility conditions hold: the plan's reason is that an open tab alone must not count as presence.

- Server caps, de-duplicates, and closes presence intervals on hidden, unfocused, idle, settle, or disconnect timeout: the reason is to avoid inflated or duplicated presence and to make settled/idle/hidden time ineligible.

- Privacy-preserving internal fixtures for the "few minutes" activity window: the reason is to calibrate presence without raw input collection.

- Listen-in start/end events for the focused bird while mix levels stay client-side: the reason is to record the interaction signal for simulation while keeping audio mixing a rendering concern.

- Offers as server-validated events with per-bird cooldown of a few minutes: the reason is canonical validation and repeated-offer throttling per bird.

- Five-second settle undo: NOT RECOVERABLE FROM PLAN

- Visit invitation by named email, pending/active/recent lists, and revoke: the reason is explicit per-invite access and host-visible management of visit state.

- One-time visitor link with 30-day unused expiry and revocation checked on every snapshot request: the reason is to keep visitor access bounded and enforceable on each pull.

- `GET /v1/visits/{token}/snapshot` with no interaction endpoints or notebook controls: the reason is same canonical renderable scene as the host, but read-only visitor access.

- Approximate visit duration logging with no badge and notification off by default: the reason is transparency without social display or opt-out notification pressure.

### 5. Simulation and behavioral engine

- Roughly one-minute server tick independent of connected clients: the reason is continuity even when no browser is connected and a single canonical simulation cadence.

- Deterministic ticks for state, ordered event range, time input, and stored random seed: the reason is reproducible advancement under retries, replay, and worker restarts.

- Account-scoped transaction/lease around loading state, events, compute, persist, and acknowledge: the reason is idempotent retries and no double-applied presence or offers.

- Bounded catch-up after downtime: the reason is to advance elapsed mood/day-phase state without one unbounded loop per missed minute or jumpy backlog behavior.

- Five bounded normalized personality scalars stored only in `Bird`: the reason is hidden server-side personality that can affect behavior without reaching snapshots, narration, or notebook.

- Slow low-pass accumulator for eligible presence and accepted interaction signals: the reason is calibrated drift: measurable after about a week, no visibly attributable single-session change, and perceptible difference only after about three weeks.

- Presence time as dominant drift input: NOT RECOVERABLE FROM PLAN

- Listen-in drift toward social warmth and vocal frequency: the plan says listen-in contributes more to those traits for that bird, tying focused listening to those dimensions.

- Accepted offers and approaching/receiving offers nudging curiosity and boldness: the plan ties those interaction signals to curiosity and boldness rather than visit count or score.

- Settlement adds no directional drift: the reason is that settle ends presence and quiets mood without becoming a trait-change action.

- Nonnegative deltas only, with caps and no negative or neglect deltas: the reason is the invariant that drift is one-way toward expressiveness and absence never creates distress.

- Server-side versioned drift coefficients, presence window, bounds, seed distributions, and adoption thresholds: the reason is tunability while freezing the selected configuration version so later tuning does not rewrite history.

- No cross-account behavioral aggregation for calibration: the reason is privacy around behavioral data and account simulation.

- Persistent mood enum and timers: the reason is that reconnect does not reset mood to neutral and mood survives across sessions.

- Mood transitions from same-session events, local-time cycle, weather, calls/alarm effects, and personality weights: the reason is to ground mood in canonical scene facts and bird-to-bird behavior rather than absence distress.

- Rain and wind effects on calling, alertness, or wariness: NOT RECOVERABLE FROM PLAN

- Quiet/ambient absence behavior: the reason is that reduced recent presence is presented as quieter ambient behavior, not worsening mood or declining trait.

- Stable species motif identity and per-bird call seed: the reason is that each bird keeps a recognizable signature across mood and trait drift.

- Runtime call grammar with motif, timing, pitch, and envelope variation: the reason is procedural calls that vary without becoming downloaded loops or losing bird identity.

- Structured call event for motif parameters and caption template parameters: the reason is shared audio, caption, and narration derivation from the same generated call.

- Bird response and chorus: NOT RECOVERABLE FROM PLAN

- Six-species pool with no species rarity mechanic: the plan states the pool should stay coherent and species rarity is not a mechanic, so the reason is to avoid turning species into ranking or scarcity.

- Adoption flow selecting initial two species and later age-based offers capped at seven: the reason is to add birds by aviary age only, never visit count or score.

- Notebook generation from noteworthy aviary events and sparse quiet observations: the reason is a field notebook that remains sparse, actual, deduplicated, and not a feed or behavior log.

### 6. Client rendering pipeline

- One-screen responsive horizontal scene with front, middle, and back perch zones: the reason is a composed aviary scene that can compress or expand without cropping birds and without pan, zoom, or scroll.

- Compact scene graph with foliage/sky, perch plane, birds, foreground branch, and sparse top-bar DOM: the reason is low-cost compositing and a quiet visual hierarchy.

- Canvas 2D renderer with semantic HTML controls, text, focus targets, and narration: the plan says Canvas 2D is suitable for "low-cost compositing and stable frame pacing," while semantic HTML preserves controls, text, focus, and narration.

- Rendering choice validated against 60fps and browser budgets: the reason is to lock the renderer only after it meets frame and browser support budgets.

- Snapshot starts with active bird state, non-static pose, transition progress, call timing, and idle activity: the reason is first paint as an already-living scene, with "never" a wake-up or loading spinner.

- Quiet field and local ambient cues if state is late: the reason is to avoid a spinner while waiting for canonical state.

- One-time fly-in only from post-adoption empty aviary to first bird: NOT RECOVERABLE FROM PLAN

- Client interpolation and idle micro-motion from bird state, mood, and stable local seeds: the reason is inexpensive presentation between authoritative snapshots without client-owned simulation.

- Client-only leaves and feathers with no server event or personality effect: the reason is ornamental motion that cannot affect canonical state or personality.

- Birds choose perch and idle action; users cannot arrange them: the reason is server/canonical bird behavior and no scene customization.

- requestAnimationFrame while visible, stopped rendering while hidden, fresh snapshot on visibility or frame gap: the reason is performance and continuity without hidden client simulation.

- Sparse top bar with account/settings, accessibility, notebook, and offer: the reason is to limit embedded controls in the scene.

- Top bar fade after inactivity and restore on pointer/keyboard activity: the reason is a quiet scene while preserving keyboard discoverability and focus visibility.

- Reduced-motion still/preen poses, slow cross-fades, no leaf/feather drift, and slower palette shifts: the reason is a complete reduced-motion renderer rather than pausing the aviary's drift, mood, or calls.

### 7. Audio pipeline

- Client-side WebAudio synthesis from compact motif library: the reason is procedural calls without downloaded loops and without delaying first bird paint.

- Recognizable bird signature across mood and trait drift: the reason is stable per-bird identity, with species motif family and bird seed preserving the motif while mood and frequency alter expression.

- At most seven voices and chorus interactions with reused nodes/buffers and capped contexts/workers: the reason is bounded performance and no memory growth while supporting the seven-bird cap.

- Audio context startup under browser gesture/autoplay rules: the reason is browser policy compliance while not gating scene rendering on audio.

- Listen-in mix ramp: the selected bird comes up and other birds go down to ambient, never silence, over a smooth envelope so focus does not erase the aviary.

- Listen-in disengagement on reselect, another bird, empty-space click, or keyboard focus leaving: the reason is consistent exit from the focused audio state across pointer and keyboard use.

- Settle gently lowers calls: the reason is to quiet mood and calls without making settle a directional drift signal.

- Offer-song fragments as soft procedural motifs: NOT RECOVERABLE FROM PLAN

- Silent fallback with captions on by default when WebAudio is unavailable or denied: the reason is graceful accessibility without recorded fallback audio.

- Captions from actual motif/parameters near the calling bird: the reason is to reflect the generated call itself and fade with it.

- Shared audio and caption call event source: the reason is to prevent mismatches.

- Accessible user audio controls and status text, with failed audio context avoiding repeated error announcements: the reason is accessibility without high-frequency error announcements.

### 8. Accessibility and performance budgets

- Slow naturalist prose narration from the same snapshot as the scene: the reason is narration grounded in canonical scene state rather than internal vectors or raw state labels.

- Idle narration update every 30-60 seconds with prompt restrained priority for greeting, accepted offer, and settle: the reason is timely narration for important moments without overwhelming output.

- Polite live region/queue that coalesces superseded narration: the reason is to prevent overlapping or high-frequency announcements.

- Keyboard access for top bar, scene entry, bird navigation, listen-in, offer menu, settle, and Escape exit: the reason is that all interactive elements work with keyboard and focus leaves listen-in consistently.

- Visible focus outlines on day and night backgrounds: the reason is focus visibility across scene palettes.

- Screen-reader prose, call captions, visual copy voice, and WCAG AA contrast checks: the reason is accessibility parity and copy consistency before release.

- Initial JavaScript under 2 MB gzipped with code-split settings, accessibility settings, and invite management: the reason is first-paint budget and keeping non-core panels out of the initial bundle.

- First bird visible under 500 ms on mid-tier mobile over 4G: the reason is that the authenticated snapshot path must still meet the immediate living-scene expectation.

- Idle motion at 60 fps on a five-year-old mid-range laptop for 30 minutes: the reason is sustained frame pacing for the animated scene.

- No client memory growth over 30 minutes: the reason is bounded audio contexts, workers, and retained notebook render nodes.

- Latest two major releases of Chrome, Safari, Firefox, and Edge, with unsupported-browser guidance outside that range: the reason is browser support coverage with direct matter-of-fact fallback guidance.

- Aggregate measurements for load, timing, failures, request errors, and tick latency with p99 alerting: the reason is operational release monitoring without product engagement metrics.

- Synthetic browsers across common geographies: NOT RECOVERABLE FROM PLAN

- Analytics credentials unable to reach simulation database: the reason is to keep simulation data isolated from analytics and warehouse access.

- Avoid presence totals and engagement funnels as product metrics: the reason is privacy and the plan's rejection of engagement/ranking metrics.

### 9. Delivery sequence and rollout

- Foundations phase before account data: the reason is to settle data contracts, threat model, schema/migrations, UUID rule, auth/session storage, idempotency, privacy review, and deletion semantics first.

- Canonical engine phase before real-user rollout: the reason is to validate calibration with synthetic accounts/events and fault injection before real users.

- Aviary surface phase before tuning release: the reason is to tune first-bird time and sustained frame/memory budgets alongside active-first render, scene composition, reduced motion, and keyboard/focus controls.

- Interaction and audio phase: the reason is to verify listen-in, offers, settle, procedural grammar, chorus, matching captions, graceful silence, and distinguishable bird signatures at seven voices together.

- Notebook and access phase as release scope: the reason is that notebook, narration queue, settings, export, session revoke, email change, deletion/recovery, and screen-reader/contrast review are not post-launch accessibility work.

- Visits phase kept disabled until access-control checks pass: the reason is to prove visitor access cannot write any event and visitor viewing does not affect host drift.

- Controlled launch through internal synthetic and staff accounts, small cohort, and progressive rollout: the reason is to pause or roll back on auth errors, cross-account access, tick latency, rendering budget, and audio failure, and to roll forward only when privacy, accessibility, and performance gates pass.

- Conservative server-side tuning of age-based new-bird offers without user-facing visit counters: the reason is to allow tuning while avoiding visit counters and engagement metrics.

### 10. Risks and mitigations

- Synthetic week/three-week drift fixtures: the reason is to tune uncertain low-pass coefficients, presence window, and caps while rejecting absence penalties or one-session-visible stat movement.

- Visibility/focus/idle transition tests for presence: the reason is to handle browser suspension and signal variation while collecting only eligibility heartbeats.

- Overlapping event, retry, worker crash, and replay tests: the reason is to prove append-only sequence, unique idempotency keys, serialized server writer, and atomic state/cursor commits under concurrency.

- DST/timezone edits and server downtime tests: the reason is to prevent tick backlog or clock changes from causing jumps.

- Listen-testing two through seven simultaneous birds: the reason is to keep birds or calls from feeling uncanny or canned while retaining seeded species/bird motifs.

- No recorded-audio fallback: the reason is that audio blocked by browser policy should not gate rendering, and captions should cover unsupported or denied WebAudio.

- Screen-reader user testing as a release requirement: the reason is to keep narration from overwhelming or flattening affect.

- Reduced-motion, keyboard, and contrast checks in every scene acceptance pass: the reason is to prevent accessibility regressions during motion or design changes.

- Invite authorization checks on every pull and mutation-route attack tests: the reason is to keep leaked or revoked invite links from becoming valid write access.

- Structured log scrubbing and event schema review: the reason is to prevent PII or relationship data from entering telemetry.

- Per-aviary sparsity, deduplication, and prose review for notebook entries: the reason is to prevent the notebook from becoming a feed or user-behavior record.

- Account-linked data store, export, mail artifact, backup, recovery, and purge checks: the reason is to prevent account deletion from leaving hidden state.

### 11. Definition of ready to launch

- New account receives two stable birds: NOT RECOVERABLE FROM PLAN

- Server restart/retry cannot lose or double-apply an event: the reason is canonical idempotent event processing and atomic state/cursor commits.

- Mood persists and drift calibration is within target: the reason is persisted mood timers and week/three-week drift calibration.

- No client API can mutate vectors: the reason is that personality values stay server-only and never enter event payloads.

- Hidden/background clients do not accrue presence: the reason is presence eligibility requires visible, focused, recent activity, with no hidden client simulation.

- Invite visitors are read-only and revocable: the reason is visitor access must not write events or affect host drift.

- Notebook output is sparse and specific: the reason is to avoid a feed, session log, vector log, visit-frequency summary, or achievement copy.

- Scene meets load/frame/memory targets: the reason is the release budgets for first paint, first bird, 60fps idle motion, and no memory growth.

- Calls are procedural and captions match them: the reason is shared generated call events and no downloaded loops.

- Narration, reduced motion, keyboard navigation, focus, and AA contrast are complete: the reason is accessibility is v1 release scope.

- Export, recovery, and hard deletion work: the reason is account data access, 30-day recovery, final purge, and deletion auditability.

- Operational dashboards exclude account/bird behavior: the reason is aggregate operational telemetry without account dimensions, bird state, event payloads, emails, interaction history, engagement, or ranking metrics.

- No out-of-scope social or gamified surface has entered the build: the reason is the product boundary excluding social profiles, chat/comments, public ranking, gamification, badges, streaks, and visit-frequency displays.
