## System-level intent

- Treat product scope as release invariants, not flexible goals. This appears in "Treat these as release invariants, not aspirational design notes" and is reinforced by launch-gate language in performance, privacy, sync, and accessibility sections.
- Keep the aviary quiet, non-gamified, and free of punishment. The plan repeatedly excludes "quests, scoring, streaks," "distress or hunger state," "bird death," and "engagement scoring"; it also says "absence may leave birds ambient or quiet but never distressed" and "no absence penalty."
- Make the server the canonical source of truth. The plan says "the server is the sole canonical state writer," "client state is presentation/interpolation only," and the tick worker is "the only writer for canonical simulation outputs."
- Preserve privacy boundaries and minimize identifiers. The plan uses "synthetic UUID," encrypted email only where required, telemetry that "cannot query or join the per-account simulation store," and aggregate-only metrics with no email, account UUID, bird IDs, event payloads, personality, mood, or histories.
- Keep personality implicit and natural, never ranked or quantified. This shows up in "Never expose personality numbers," "no user surface ranks or quantifies bird personality," no trait deltas in event responses, and notebook/narration rules that avoid trait numbers.
- Use specific voice by surface. The plan distinguishes "matter-of-fact system copy" for account, error, sync, and accessibility-settings surfaces from "specific lowercase naturalist prose" for the aviary and notebook.
- Treat accessibility as part of the core aviary, not a fallback. The plan says "Both visual and audio accessibility paths ship at launch," "reduced motion remains a designed aviary," and warns against "Accessibility becomes a second-rate fallback."
- Make simulation slow, deterministic, and auditable. The plan calls for seeded pseudo-randomness, versioned configuration, reproducible restart/retry behavior, "small measurable vector movement" after about one week, and no visibly moving trait in one session.
- Keep social access bounded and read-only. The plan allows "per-invite read-only visits" while excluding shared aviary, public discovery, chat, comments, social profile, feed, and show-off rendering; a visitor "cannot affect the host simulation."

## Per-feature whys

### 1. Product scope and invariants

- One browser-based aviary per single-user account: the plan ties this to a single account ownership boundary, no shared aviary, no public discovery, and an auditable state/privacy boundary.
- Two system-selected starter birds: NOT RECOVERABLE FROM PLAN
- Seven-bird maximum: the plan says "all seven-or-fewer birds remain in frame," requires profiling at "two and seven birds," and says to preserve the "seven-bird cap if optimization is needed."
- Email magic-link auth: the rationale is one-time verified access with token hashes, expiry, single use, per-email rate limits, and no logged link contents.
- Stable named birds: the plan says rename must not alter "identity, vector, mood, or call identity," preserving stable identity and recognizable signatures.
- Server-simulated moods and personality drift: the plan makes this server-owned so clients cannot supply personality values or authoritative timestamps and so multi-device state is serialized.
- Animated single-screen scene: the plan wants all birds visible with no pan, zoom, scroll, drag-to-place, inline labels, badges, or bird overlays.
- Procedural calls: the plan says calls are "generated procedurally," avoids downloadable or looped call assets, and keeps stable bird signatures across mood and drift.
- Listen-in: the plan uses listen-in to focus one bird while others remain audible at ambient level and to provide a validated attention signal for social warmth and vocal frequency.
- Seed, song-fragment, and still-pool offers: the plan uses offers as bounded secondary signals that nudge curiosity, boldness, or vocality, with cooldowns so one session cannot saturate curiosity.
- Settle: the plan says settle "quiets mood," terminates the presence window, fades light and calls toward evening, and adds "no directional drift."
- Sparse read-only field notebook: the plan frames notebook entries as rare observations, not a client event feed; entries preserve chronological history without edit/delete and avoid visit counts, trait values, and achievement language.
- Presence accounting: the plan makes presence "the primary drift input, not click count," and requires visible, focused, recently active intervals so an open or background tab does not count.
- Account settings: the plan uses settings for timezone, export/deletion, invitation notice preference, accessibility preferences, and session/account lifecycle control.
- Accessibility settings: the plan requires OS preference plus explicit setting support so reduced motion, captions, narration, keyboard, focus, and contrast are product acceptance work.
- Multi-device sync: the plan requires server receipt ordering, database locks/versioning, event cursors, and no last-write-wins so personality history is not lost or duplicated.
- Export: the plan says export is on demand, snapshot-based, delivered to verified email by expiring URL, and includes sensitive vectors because "the user owns their copy."
- Deletion: the plan uses immediate soft deletion, 30-day recovery, then hard delete with deletion propagation across aviary, vectors, notebook, events, sessions, invitations, telemetry, exports, and backups.
- Per-invite read-only visits: the plan limits visits to deliberate host invitations, read-only capability, revocation, and transparency logs; visitor activity cannot enter drift or affect the host scene.
- Visual and audio accessibility paths at launch: the plan makes these launch gates and says not to gate reduced motion, captions, narration, or keyboard support behind later rollout.
- No quests, scoring, streaks, or visit-frequency display: the plan rejects engagement scoring, per-account usage dashboards, attendance language, achievement language, and gamified notebook output.
- No distress, hunger, bird death, or neglect penalty: the plan says absence may leave birds ambient or quiet but never distressed, and personality traits never decrease due to neglect.
- No textual welcome or return announcement: the plan repeats this as a rule, and instead uses procedural return greeting behavior with no absence-duration UI.
- No native client: NOT RECOVERABLE FROM PLAN
- No payments: NOT RECOVERABLE FROM PLAN
- No customizable scene: NOT RECOVERABLE FROM PLAN

### 2. System shape and ownership

- Modular service rather than per-feature services: the plan says explicit boundaries keep "the state writer and privacy boundary" auditable.
- Browser client: the rationale is presentation and interpolation only; it fetches snapshots, synthesizes calls, and submits typed events without calculating or persisting canonical moods, personality, locations, or drift.
- API: the rationale is authenticated ownership and validation; it appends events and manages account features while refusing client-supplied personality values or authoritative event timestamps.
- Tick worker: the plan makes it the only canonical simulation writer and has it process each account's unconsumed events in order, whether or not a browser is connected.
- Relational database: the plan uses it as a transactional source of truth for account, birds, state, events, notebook, invitations, and sessions.
- Account UUID references and partitioning: the plan uses UUIDs so email is not a key, log field, metric dimension, or inter-service identifier.
- Mail/auth adapter: the plan isolates mail and token handling so token hashes, expiry, single use, rate limits, and no logged links can be enforced.
- Operational telemetry: the plan keeps telemetry separate so it can track aggregate request, latency, error, render, and audio health without joining per-account simulation data.
- Shared compact snapshot contract for owned and visitor views: the plan uses the same state representation while limiting write authority to the owned route and keeping visitor routes read-only.
- Prompt HTML and small bootstrap snapshot: the plan wants a quiet field drawn promptly without spinner or wake-up animation while snapshot data arrives.

### 3. Persistent data and state boundaries

- UUID primary keys and explicit schema versions: the plan uses them for durable records, versioned simulation, and auditable state evolution.
- Account timezone: the plan says mood and day/night follow the user's local time and must be canonical so the worker and all devices agree.
- Device sessions: the rationale is revocable per-device access; revocation takes effect on subsequent requests.
- Aviary age and bird-offer eligibility: the plan derives offers from aviary age, not required attendance, engagement, click volume, or paid tier.
- Bird stable UUID, seed, vector, mood, perch, pose, and call grammar identity: the plan keeps bird identity stable while hiding server-only personality and preserving mood/call continuity.
- Versioned simulation configuration and trait clamping: the plan keeps trait ranges and starter seeds versioned and prevents neglect-driven decreases.
- Interaction event log: the plan uses append-only ordered events, server receive time, and idempotency so retries cannot apply drift twice and visitor activity never becomes host events.
- Notebook entries: the plan makes them immutable rare observations generated only from the aviary's state, with source-state references for audit or regeneration without exposing trait numbers.
- Invitation, visit session, and visit log records: the plan supports one-time links, 30-day unused expiry, revocation, approximate duration transparency, and no visitor presence in bird simulation.
- Notice preference off by default: the plan says notices are only explicitly opted in and control notices without affecting simulation or visit state.
- Export/deletion job records: the plan uses job state and one-time download token metadata for expiring export delivery and verifiable deletion completion.
- Atomic tick cursor and snapshot version commits: the plan requires transactional commits, optimistic version checks or row locks, and unique event IDs to serialize ticks and prevent duplicate drift.

### 4. HTTP/API contracts

- Versioned API and typed schemas: the plan wants explicit contracts for owned and visitor routes and narrow request/response shapes.
- Owned-route authentication with revocable device sessions: the rationale is account ownership and later session revocation.
- Visitor-route short-lived scoped capabilities: the plan uses limited capabilities so visitor access is read-only and revocable.
- Magic-link creation and consume endpoints: the plan uses a 15-minute link, per-email rate limits, atomic single consume, and matter-of-fact expired/used errors.
- Session listing/revocation: the plan gives users control over device sessions.
- Verified email change: the plan requires the new address to work only after verification while the old remains active until then.
- Aviary snapshot endpoint: the plan returns canonical version, server time, timezone/day phase, weather, safe bird state, transitions, and call cues while excluding personality vectors and keeping payloads in kilobytes.
- Snapshot version cursors and `If-None-Match`: the plan uses them for unchanged state and efficient sync.
- Aviary events endpoint: the plan accepts only an idempotency key and narrow event union while deriving account, receipt time, duration caps, and authorization from the session.
- Event validation: the plan validates bird ownership, cooldowns, duration bounds, and settle undo windows so client claims cannot directly shape state.
- Event response without trait delta: the plan avoids user-visible personality values by returning sequence, cooldown, and undo state rather than drift amounts.
- Notebook paging endpoint: the plan provides immutable newest-first entries while excluding streaks and visit-frequency observations.
- Initial adoption naming endpoint: the plan completes naming for two server-selected starter birds.
- Age eligibility endpoint and adoption acceptance: the plan makes new-bird eligibility age-based, capped at seven, and not tied to visits, clicks, or paid tier.
- Bird rename endpoint: the plan allows name changes while preserving bird identity, vector, mood, and call identity.
- Invitation creation endpoint: the plan creates one-time invite links by target email for read-only visits.
- Visitor snapshot endpoint: the plan returns the same state representation without write authority and rechecks revocation on periodic pulls.
- Host-only invitation list, revoke, and visit log endpoints: the plan provides transparency and control in account settings.
- Absence of visitor event endpoints: the plan prevents visit authority from offering, listening in, settling, or producing drift.
- Account endpoints for settings, timezone, export, deletion, accessibility, and notices: the plan centralizes account lifecycle and preferences with normal system language.
- Explicit conflict, auth, expired, and revoked states: the plan avoids client conflict resolution by having clients re-fetch canonical state after reconnect or version mismatch.

### 5. Simulation and behavior engine

- Due-work scheduler with roughly one-minute cadence and jitter: the plan spreads load while ticking aviaries whether or not browsers are connected.
- Transactional tick processing: the plan loads state, processes events by sequence, computes elapsed time, updates moods/effects/notebook, persists version and cursor, then commits.
- Bounded deterministic catch-up: the plan avoids replaying thousands of visual frames after long offline periods and uses elapsed-duration formulas or coarse buckets.
- Idempotent tick IDs and safe retry: the plan supports restart/retry behavior without duplicate simulation effects.
- Versioned deterministic simulation module: the plan uses seeded pseudo-randomness from aviary, tick, and bird IDs for reproducibility and variety.
- Offline calibration configuration: the plan keeps coefficients separate from API/client code and records the version in state.
- Slow low-pass personality drift: the plan targets small measurable movement after one week of regular presence, noticeable differences around three weeks, and no visible one-session trait movement.
- Presence-dominant drift: the plan says presence time dominates and click count should not be rewarded.
- Listen-in drift: the plan gives focused birds extra attention for social warmth and vocal frequency.
- Accepted offer and near-bird offer drift: the plan uses offers to nudge curiosity and boldness while cooldowns prevent saturation.
- Settle behavior in simulation: the plan quiets mood and terminates presence without directional drift.
- Per-tick and lifetime bounds plus monotonicity tests: the plan prevents personality decay and negative inference from inactivity.
- Separate mood state: the plan makes mood a faster daily-ish enum that persists across sessions and never resets on open.
- Mood transition inputs: the plan uses prior mood, recent events, local time phase, weather, bird-to-bird calls/alarm state, and personality.
- Night and species behavior: the plan generally settles birds at night while a nightjar-like species may remain active.
- Rain and wind behavior: the plan dampens calling in rain and affects alert/wariness in wind.
- Perch and idle pose choice: the plan derives these from mood and personality with stochastic variation constrained by stable identity.
- Per-species call motif grammar and stable bird signature: the plan keeps calls recognizable across mood and drift.
- Client WebAudio synthesis: the plan uses oscillators, noise, envelopes, bounded voices, and small timing/pitch variation instead of recorded assets.
- Server-derived response and chorus schedules: the plan ensures concurrent clients render the same aviary moments.

### 6. Presence, sessions, and synchronization

- Visibility, focus, and recent activity presence gate: the plan avoids equating an open tab with presence while allowing still watching through a calibrated lease.
- Presence lease renewal and termination: the plan terminates intervals on focus/visibility loss, settle, pagehide, or lease expiry to avoid background accumulation.
- Server validation and caps for intervals: the plan prevents clock tampering and excessive claimed duration.
- Low-frequency snapshot pull and presence push while visible: the plan keeps clients synchronized without making the client canonical.
- Immediate pull on visibility return, long frame gap, or device resume: the plan avoids simulating missed time on the client after suspension.
- Client interpolation between canonical snapshots: the plan allows smooth presentation while keeping durable state server-owned.
- Multi-device overlap: the plan serializes events on server receipt sequence and avoids last-write-wins for vectors.
- Transaction/locking and event cursors: the plan prevents lost updates and duplicate processing.

### 7. Frontend scene and interaction rendering

- Responsive horizontal scene with front, middle, and back perch zones: the plan keeps all birds visible on the smallest supported viewport with calm depth.
- No pan, zoom, scroll, drag-to-place, labels, badges, or overlays: the plan keeps the aviary a single calm scene rather than a manipulable dashboard.
- Top bar containing account/settings, accessibility, notebook, and offer only: the plan limits persistent controls on the scene.
- Top bar fade and restore: the plan quiets the visual surface while preserving keyboard discoverability and focus visibility.
- First paint as quiet field or snapshot-backed scene: the plan targets first-bird-visible under 500ms and avoids blocking on notebook, settings, audio permission, or secondary assets.
- Mood/personality-keyed idle motion: the plan uses preen, scan, tilt, and shuffle to express mood and personality without textual labels.
- Leaves, feathers, and subtle parallax as client ornaments: the plan keeps ornaments out of persistent simulation state.
- Day/night from account timezone: the plan keeps scene phase consistent across worker and devices.
- Weather and bird locations from canonical snapshots: the plan prevents the client from inventing durable state.
- Return greeting: the plan uses absence duration, boldness, mood, and deterministic variation, with one bird noticing first and others staggered when appropriate, instead of textual welcome or absence-duration UI.
- Listen-in engagement and disengagement: the plan supports click/tap and keyboard focus, gradually raises the focused bird, and gradually returns when focus leaves or target changes.
- Offers from top bar: the plan chooses offer type and eligible target through UI and renders reactions from mood, curiosity, and vocality without direct bird clicking.
- Settle undo window: the plan gives any aviary click within five seconds as undo for settle.
- Tab close equivalent to settle at simulation level: the plan avoids a recovery prompt and treats closing as ordinary quieting.
- Reduced motion: the plan replaces continuous movement with cross-fading still poses, removes leaf drift, slows ambient lighting, and preserves audio, mood, drift, and notebook behavior.
- Code-splitting account, accessibility, notebook, and invitation surfaces: the plan protects the initial JavaScript budget.
- Initial gzipped JavaScript below 2MB: the plan supports fast first draw and performance budgets.

### 8. Audio, captions, and accessible interaction

- Bounded WebAudio engine: the plan reuses one AudioContext and pooled voices to control resources and mix calls softly.
- Listen-in gain automation and mix floors: the plan lets one bird become prominent while other birds never become silent.
- Autoplay handling: the plan starts or resumes audio only after an allowed gesture and otherwise renders gracefully in silence.
- Captions default on when audio is unavailable or denied: the plan keeps audio information accessible.
- No recorded fallback audio: the plan relies on procedural calls and silence plus captions as the fallback.
- Captions generated from actual motif and performance parameters: the plan makes captions accurate to the call and places them near the caller.
- Caption readability across scene phases: the plan requires captions to remain readable in all palettes and phases.
- Parallel prose narration surface: the plan builds narration from the same snapshot in naturalist lowercase present tense, avoiding personality numbers and rapid state chatter.
- Live region queue and coalescing: the plan prevents screen-reader users from being flooded.
- Keyboard controls: the plan supports top-bar tab order, scene entry, arrow bird focus, Enter for listen-in, Escape to exit, offers, and settle.
- Visible high-contrast focus outlines: the plan keeps keyboard focus usable against all scene palettes.
- Accessible names without internal state: the plan keeps interaction accessible while preserving hidden personality and system measurements.
- WCAG AA, screen-reader, keyboard, reduced-motion, audio-blocked, and narrow-viewport testing: the plan treats these as product acceptance work.

### 9. Notebook, account lifecycle, and visits

- Notebook generation in simulation/domain layer: the plan avoids a client event feed and bases observations on state facts.
- Noteworthy observation templates and sparsity policy: the plan keeps entries rare and authored, with reviewed variants rather than generic state labels or freeform sensitive text.
- Banned notebook content: the plan excludes visit counts, user presence frequency, trait values, achievement language, and generic event-log phrasing.
- Chronological notebook history and pagination: the plan preserves long histories while making them readable.
- Account creation flow: the plan sends a one-time link, creates one aviary, chooses starter species server-side, asks for names, and follows quiet empty-state handling.
- Age-based bird offers over months: the plan makes growth depend on age, stops at seven, and never depends on engagement.
- Deletion lifecycle: the plan uses immediate soft deletion, 30-day recovery, then a verifiable hard-delete job.
- Export lifecycle: the plan makes export requested, snapshot-based, and delivered by expiring URL to verified email.
- Visits default off until host invitation: the plan requires deliberate host creation before any visit exists.
- Target email validation and one-time invite links: the plan limits visitor access to intended recipients and time-bounded links.
- Invitation listing, visit records, and revocation in settings: the plan gives the host transparency and control.
- Snapshot revocation recheck on each pull: the plan makes revocation effective on the visitor's immediate next pull.
- Visitor pages as ambient read-only: the plan bars host-state-changing notebook browsing, presence heartbeat, offers, listen-in, settle, and shared cursor.
- Approximate visit duration logging: the plan logs duration solely for host transparency, not drift.
- No default visit notification: the plan honors notices only when the optional setting is enabled.
- No social profile, feed, discovery, chat, comments, or show-off rendering: the plan keeps visits private, ambient, and non-social.

### 10. Performance, observability, and release

- Initial JavaScript, first-bird, frame-rate, and memory budgets: the plan sets acceptance budgets for fast load, smooth idle motion, and no 30-minute memory growth.
- Audio and rendering resource bounds: the plan reuses audio buffers/voices, bounds workers/contexts, releases notebook references, pauses when hidden, and keeps ornaments allocation-light.
- Latest two major browser versions: the plan sets a support window for Chrome, Safari, Firefox, and Edge.
- Unsupported-browser page: the plan uses clear matter-of-fact copy for older browsers.
- Aggregate synthetic checks and RUM: the plan monitors navigation, snapshot/first-bird timing, frame timing, audio-context errors, API/tick latency, and error rates without per-account data.
- Tick p99 alarms: the plan pages/alarms when simulation tick p99 exceeds five seconds.
- Telemetry exclusions: the plan excludes bird ID, account dimension, state payload, interaction type history, and user email.
- Request correlation IDs not linkable to account UUID in analytics: the plan preserves operational debugging without account-linked analytics.
- Telemetry schema review as API review: the plan makes privacy review part of interface review.
- Controlled server-side rollout flags: the plan starts with deterministic fixtures and a small invited cohort, then broadens after auth, sync, privacy, and accessibility acceptance.
- Ramp age-based adoption eligibility and load toward seven birds: the plan separates initial quality gates from later load/adoption ramp.
- Accessibility support not gated behind later rollout: the plan requires reduced motion, captions, screen-reader narration, and keyboard support from launch.
- Aggregate operational and product-quality metrics: the plan tracks latency, tick p99, errors, frame budget, audio fallback incidence, and invite correctness without engagement scoring.
- Drift and call mix calibration: the plan uses internal fixtures and consented qualitative review without retaining per-bird telemetry for population analysis.
- Prelaunch concurrency and retry verification: the plan tests duplicate tick delivery, multi-device overlap, stale snapshots, event ordering, and deletion races.
- Browser/device profiling at two and seven birds: the plan verifies performance at the starter and cap sizes.
- Auth and invitation token/revocation checks: the plan treats token and revocation behavior as launch gates.
- Accessibility audits: the plan requires audits across visual, keyboard, screen-reader, reduced-motion, captions, and contrast.
- Launch gates not invitations to add engagement features: the plan says verification is for the stated product behaviors, not new engagement features.

### 11. Main risks and mitigations

- Drift calibration risk: the plan mitigates too-fast, too-slow, or click-rewarding drift with one-week and three-week targets, presence dominance, bounded contributions, simulated trajectories, monotonic nonnegative deltas, and no absence penalty.
- Presence risk: the plan mitigates inflated or missed presence with visible, focused, recent activity, lease intervals, capped durations, hide/settle/expiry termination, and browser tests.
- Sync risk: the plan mitigates lost or duplicated personality history with a single server writer, ordered append-only events, idempotency, transactional cursor/state commit, locking/versioning, and simultaneous-client recovery tests.
- Mood/scene restart risk: the plan mitigates mood jumps and "newly started" feel by persisting mood, absolute transition timing, timezone-driven phase, current bootstrap snapshot, deterministic catch-up, and no entry/loading animations.
- Call quality risk: the plan mitigates synthetic or blurred signatures by prototyping motifs early across species and simultaneous birds, testing recognizability, tuning envelopes/timing and mix floors, and using silence plus captions as fallback.
- Accessibility fallback risk: the plan mitigates it by designing narration, reduced-motion poses, captions, focus, and contrast in parallel with the core renderer and including assistive-technology users in acceptance review.
- Privacy leakage risk: the plan mitigates it with synthetic UUIDs except encrypted email, isolated telemetry, schema allowlists, token hashing, authorization tests, deletion propagation, and export-content audit.
- Repetitive or gamified notebook risk: the plan mitigates it with template review, cadence limits, and banned-category tests for attendance, trait values, achievement language, and generic event-log phrasing.
- Seven-bird and long-session performance risk: the plan mitigates it with early max-load and 30-minute profiling, bounded animation/audio resources, small bootstrap payload, and preserving the two-bird start and seven-bird cap.
