## System-level intent

- **V1 discipline and explicit boundaries.** The plan repeatedly defines what is "in scope for v1" and "explicitly out of scope." It keeps the first version to a "browser-only client," "single-user accounts," a capped aviary, and account management, while excluding native apps, payments, shared aviaries, customizable scenes, and broad social surfaces.

- **A calm product that refuses gamification and custodial obligation.** The plan excludes "achievements, streaks, levels, scores, badges, calendars, XP, tiers, ranks" and also excludes "Tamagotchi mechanics: death, hunger, distress, happiness decay, custodial obligation." The adoption ramp is "age-based, not engagement-based" because it "refuses the gamification trap of 'earn more birds by visiting more.'" Drift on neglect is "0.0," not punishment.

- **Slow, additive, expressive change.** The drift design is a "low-pass filter" with "measurable drift" around one week and "visible drift" around three weeks. Personality deltas are "non-negative," "additive," and "monotonic toward expressive." The risk section names the desired feel as a "slow current."

- **Server-canonical correctness rather than client reconciliation.** The "key invariant" is that "the client never writes personality state" and "never computes drift." The server holds the "single canonical state," clients post events, and "personality state conflicts are architecturally impossible." This carries through the service split, event log, tick scheduler, and sync model.

- **Privacy and safety enforced by architecture.** Email is encrypted and hashed; the "synthetic UUID is the only identifier used anywhere outside the account record itself." The analytics pipeline may not read the simulation database, and the separation is enforced by "network policies, separate credentials," not just application discipline. The visitor path is also isolated so a "visitor never writes."

- **A read-only, opt-in social edge instead of a social network.** The plan excludes "profiles, follows, public feeds, discovery, leaderboards, comments, co-presence." The only visitor surface is "per-invite, opt-in, read-only ambient view, revocable, 30-day expiry," backed by a Visit Service that is a "thin read-only projection."

- **Naturalist field-notebook voice as product voice.** Notebook prose is "naturalist voice, lowercase, present-tense." Screen-reader narration uses the "same naturalist field-notebook voice" and is "never announcement-style." The plan rejects LLM-generated notebook prose because it would risk "voice inconsistency and hallucinated details."

- **Accessibility as a designed surface, not a fallback.** The plan includes "screen-reader narration," "reduced-motion mode," "call captions," "WCAG AA contrast," and "full keyboard navigation." Reduced motion is explicitly a "designed aesthetic, not a degraded fallback," with cross-fades that have "their own calm quality."

- **Recognizable procedural variation without repetition.** Calls come from species motif libraries plus each bird's `call_grammar_seed`, so a call is recognizable as "Pip's call" but "never identical twice." Chorus behavior, greeting hashing, pitch jitter, timing jitter, and motif selection all reinforce variation without breaking identity.

- **Fast, lightweight, observable execution.** The plan gives budgets for "<2 MB initial JS bundle," "<500 ms time-to-first-bird," "60 fps idle motion," and "zero memory growth over 30 min." It chooses Canvas 2D, embedded initial snapshots, progressive loading, code-splitting, RUM, synthetic tests, and deploy gates to keep the aviary responsive.

## Per-feature whys

### Scope and exclusions

- **Browser-only client** - NOT RECOVERABLE FROM PLAN

- **Single-user accounts** - NOT RECOVERABLE FROM PLAN

- **Synthetic UUID account IDs** - The synthetic UUID is the "only identifier used anywhere outside the account record itself" so email never appears in "logs, partition keys, telemetry, or inter-service messages."

- **Per-device session tokens and session revocation** - These live in the isolated Auth Service because auth is "security-critical" and benefits from "secrets, rate-limiting, brute-force protection."

- **Server-side aviary simulation** - The server owns personality, mood, drift, notebook generation, and tick scheduling so "multi-device sync" is correct and "no last-write-wins" is enforceable.

- **No numeric personality-vector surface** - NOT RECOVERABLE FROM PLAN

- **Two starter birds** - NOT RECOVERABLE FROM PLAN

- **Age-gated adoption and birds-per-aviary ramp** - The ramp is "age-based, not engagement-based" to refuse the "gamification trap" of earning birds by visiting more.

- **Six-species pool** - Six species provide "enough variety for a 7-bird aviary" without "diluting per-species design quality."

- **All gamification out of scope** - The plan excludes scores, streaks, badges, tiers, ranks, and related surfaces to keep the aviary away from achievement and progression mechanics.

- **Tamagotchi mechanics out of scope** - The plan excludes death, hunger, distress, happiness decay, and "custodial obligation"; neglect produces "zero drift, not negative drift."

- **Social-network surfaces out of scope** - The plan keeps social behavior to an opt-in, read-only visitor surface and excludes profiles, follows, public feeds, discovery, leaderboards, comments, and co-presence.

- **Push notifications and aviary-state email notifications out of scope** - NOT RECOVERABLE FROM PLAN

- **Shared aviaries, multi-aviary accounts, customizable scenes, payments, and billing out of scope** - NOT RECOVERABLE FROM PLAN

### Architecture and sync

- **Three services rather than one monolith** - Auth is isolated because it is "security-critical"; Aviary owns the hot simulation path; Visit is isolated to prevent visitor traffic from impacting the host and to enforce "visitor never writes" at the network boundary.

- **Shared Postgres cluster with separate schemas** - This keeps "operational complexity low" at v1 scale while allowing the simulation DB to split later "without schema changes."

- **Client/server split** - The client renders, animates, synthesizes audio, detects presence, and posts events; the server mutates personality and mood. This preserves the invariant that the client "never writes personality state" and "never computes drift."

- **StateSnapshot render boundary** - The snapshot contains "everything the client needs to render the current state and schedule the next few seconds of audio" while excluding raw personality values.

- **Visit Service as read-only projection** - It prevents visitor traffic from impacting the host's simulation path and enforces the "visitor never writes" rule.

- **SSE for real-time updates** - SSE is preferred over WebSocket because the flow is "unidirectional (server -> client)" and has "simpler reconnection semantics."

- **Snapshot pull triggers** - Initial page load supports fast first paint; visibility change, SSE updates, keepalive, and resume-from-suspend keep the rendered state current after tab changes, disconnects, and suspended animation frames.

- **Snapshot interpolation and missed-snapshot behavior** - Interpolation smooths positions, lighting, and motion transitions; if a snapshot is missed, the client keeps the last state with idle motion and then "snaps to the correct state with a smooth transition."

### Data model

- **Encrypted email plus email hash** - Encryption protects the address, while the hash is "for uniqueness check only"; the email is kept out of logs, telemetry, partition keys, and inter-service messages.

- **Session token hashes** - Tokens are stored as SHA-256 hashes so bearer tokens are not stored directly in the session record.

- **Stable bird IDs** - NOT RECOVERABLE FROM PLAN

- **`call_grammar_seed`** - The seed deterministically selects motif material so each bird's call signature is "stable and recognizable."

- **Append-only interaction event log** - Append-only events make simultaneous device posts safe, avoid overwrite conflicts, and give the tick a durable input stream to mark as consumed.

- **`consumed_by_tick` event marking** - Events are marked consumed only after successful processing so a failed tick can re-process unconsumed events.

- **Notebook entries** - Entries convert noteworthy simulation events into "naturalist voice, lowercase, present-tense" prose and preserve which interaction events contributed.

- **Visit invitation expiry, revocation, and token hashing** - Invitations are "per-invite, opt-in, read-only," revocable, and expire after 30 days, bounding visitor access.

- **Visit log** - NOT RECOVERABLE FROM PLAN

- **Additive, monotonic drift** - Drift adds non-negative deltas and clamps traits to [0, 1], so birds grow "toward expressive" and neglect never reduces traits.

- **Mood persistence across sessions** - Persisted mood makes session-end mood the starting point next time, with server-side ticks modifying it during the interim.

### API surface

- **Magic-link request and consume** - Tokens are single-use, expire after 15 minutes, are hashed, and requests are rate-limited, supporting the security-critical auth boundary.

- **Snapshot ETag for conditional requests** - NOT RECOVERABLE FROM PLAN

- **Aviary event posting** - Clients append events to the log rather than writing simulation state, preserving the server-canonical tick model.

- **Bird rename endpoint** - NOT RECOVERABLE FROM PLAN

- **Account export, soft-then-hard deletion, email change, and settings endpoints** - NOT RECOVERABLE FROM PLAN

- **Visitor snapshot endpoint with disabled affordances** - The visitor receives the same StateSnapshot but has no offer, listen-in, or settle controls and posts no events, preserving read-only visiting.

- **Reduced-motion, captions, and visit-notification settings** - Settings persist user accessibility and visit preferences to the account.

### Simulation engine design

- **Tick scheduler** - A roughly 60-second worker cadence processes each account in a transaction, with p99 latency under 5 seconds and enough headroom for "thousands of accounts."

- **Tick idempotency** - If a tick fails mid-processing, the next tick can re-process unconsumed events; this keeps the event-ingestion path safe.

- **Presence-ping validation** - The server checks visibility, focus, and recent input, then logs anomalies even though the client already filters.

- **Low-pass drift calibration** - The calibration targets make drift measurable in instruments after about one week, visible to the user after about three weeks, and imperceptible in a single session.

- **Per-trait drift modulation** - Traits respond differently to signals: boldness gets more from presence, vocal frequency from listen-in, curiosity from offer ratio, and plumage saturation more slowly.

- **Bird-to-bird interactions** - NOT RECOVERABLE FROM PLAN

- **Ambient weather frequency** - NOT RECOVERABLE FROM PLAN

- **Notebook noteworthy-event detectors and cooldown** - Entries are generated only for patterns like first-greeter change, long quiet stretch, chorus, mood stability, offer pattern, or drift milestone, with a minimum two-day cooldown to preserve sparsity.

- **Hand-written notebook templates instead of LLM prose** - Templates are curated and reviewed because LLM-generated prose risks "voice inconsistency and hallucinated details."

### Frontend rendering pipeline

- **Canvas 2D scene** - Canvas 2D is "sufficient for the visual complexity"; WebGL would add complexity without proportional benefit, and DOM rendering would not hit 60 fps with seven birds, ambient motion, and parallax.

- **Layered scene composition** - NOT RECOVERABLE FROM PLAN

- **Layered bird sprites with procedural variation** - Species silhouettes, plumage layers, poses, and motion let birds vary procedurally while keeping assets predictable and small.

- **Progressive bird asset loading** - First paint uses a single pose per species, keeping the aviary responsive before the full pose library loads.

- **Idle micro-motion from `motion_state` and `motion_phase`** - The client advances motion phase locally between snapshots for "continuous animation."

- **Day/night cycle** - Timezone-based lighting keeps the day/night cycle meaningful, with smooth gradients rather than discrete switches.

- **Reduced-motion rendering** - Motion is replaced with cross-fades, ambient drift is removed, and the surface is treated as a "designed aesthetic, not a degraded fallback."

- **Quiet field loading state** - When birds are not yet available, the sky and foliage appear with no spinner because it reads as "the aviary is here, the birds are just not visible yet."

- **Audio context on first user gesture** - The audio context waits for a user gesture because of browser autoplay policy.

- **Embedded initial snapshot and progressive hydration** - The first frame renders from the embedded snapshot so birds are already in position before the full JS bundle hydrates.

### Audio pipeline

- **Procedural WebAudio calls** - Calls are synthesized from motif selection, pitch and tempo transforms, envelopes, and a mixer rather than recorded audio files.

- **Per-bird call signature** - `call_grammar_seed` selects motif subsets and base pitch so calls are "recognizable across sessions."

- **Chorus mixing** - Separate audio channels and independent seeds create a "real chorus" without phase cancellation or stacking artifacts.

- **Listen-in mix ramps** - The focused bird ramps to 0 dB while others ramp to -12 dB, not muted, with 1-2 second transitions to avoid hard cuts.

- **Ambient audio and weather audio** - NOT RECOVERABLE FROM PLAN

- **WebAudio fallback** - If WebAudio is unavailable, the aviary is silent, captions are enabled by default, and there is "no recorded-audio fallback" because the no-recorded-audio rule is unconditional.

- **Audio bundle budget** - Motif definitions and synthesis code stay under about 50 KB to preserve the initial bundle budget.

### Accessibility surfaces

- **Screen-reader narration via ARIA live region** - The live region gives screen-reader users prose updates generated from the snapshot state.

- **Narration priority queue** - User-initiated events are narrated within 2 seconds, state changes within 15 seconds, and ambient updates within 60 seconds so important changes arrive sooner.

- **Narration voice** - Narration uses the same "naturalist field-notebook voice," "lowercase, present-tense, specific," and "never announcement-style."

- **Call captions** - Captions describe the actual motifs played and are auto-enabled when WebAudio is unavailable.

- **Keyboard navigation and focus ring** - Birds and top-bar actions are reachable with Tab, arrows, Enter, Escape, and shortcuts, with a focus ring that contrasts against bright and dim scene states.

- **WCAG AA contrast** - Text, top-bar labels, and captions are designed to pass contrast thresholds against variable aviary backgrounds.

### Performance budgets and observability

- **Bundle budget and code-splitting** - The initial bundle stays under 2 MB gzipped, while non-critical settings, visit, export, and deletion surfaces load on first access.

- **Time-to-first-bird strategy** - Embedded snapshots, CDN caching, progressive enhancement, critical CSS, and preconnect support the "<500 ms" target.

- **Runtime budgets and synthetic measurement** - Frame rate, frame time, memory growth, audio errors, and snapshot latency are measured to preserve 60 fps idle motion and zero memory growth over 30 minutes.

- **Aggregate telemetry** - Request counts, latencies, tick latency, frame timing, audio errors, anonymized session histograms, and bundle size are allowed for operations.

- **Forbidden per-account telemetry** - Bird states, personality values, mood transitions, event contents, notebook contents, and presence-time are excluded to preserve the privacy boundary.

- **Separate telemetry pipelines** - Aggregate metrics and simulation data are separated at the infrastructure level so the analytics pipeline never reads the simulation database.

- **Alerting thresholds** - Alerts on tick latency, snapshot latency, endpoint errors, and memory growth catch operational failures tied to the plan's budgets.

### Rollout, implementation order, and testing

- **Internal alpha** - The first phase measures drift trajectories, tunes constants, validates mood and notebook cadence, and checks performance budgets on the target device matrix.

- **Closed beta** - The second phase expands to about 200 users, enables third-bird adoption and visit invitations, tests notebook cadence, and collects screen-reader feedback.

- **General availability** - GA opens sign-up, ramps adoption by aviary age, enables the full species pool, and continues drift calibration.

- **Day-one instrumentation** - Drift, mood transitions, notebook rate, greeting variation, audio errors, and presence validity are measured from the start so calibration and regressions are visible.

- **Implementation order** - The build "de-risk[s] the load-bearing systems first"; simulation and rendering overlap because they communicate through an early, stable snapshot interface.

- **Unit tests** - Unit tests cover drift, mood transitions, presence accounting, and notebook cadence/voice at the level where each rule is isolated.

- **Integration tests** - Integration tests cover multi-device sync, tick idempotency, visit revocation, and auth lifecycle because those paths cross service and state boundaries.

- **Synthetic tests** - Automated browsers measure first bird, frame rate, memory growth, audio errors, and snapshot latency on every deploy and on a 6-hour schedule.

- **Drift calibration tests** - Ninety-day simulations verify measurable and visible drift targets, no negative drift on neglect, and trait bounds.

- **Accessibility tests** - ARIA cadence/voice, reduced-motion behavior, keyboard reachability, and contrast are tested to guard the designed accessibility surfaces.

### Open questions and defensible calls

- **Three-minute activity window for presence** - Three minutes is a "defensible middle ground": long enough for watching without moving, short enough that walking away does not accumulate drift.

- **Two-day notebook cadence** - The two-day minimum "preserves sparsity for active users" and can be tuned during alpha.

- **Hand-designed SVG bird assets instead of fully procedural bird generation** - Hand-designed assets are "more predictable and easier to quality-check."

- **Stored, overridable timezone** - Inferred-then-stored timezone avoids "the complexity of real-time timezone detection" while keeping the day/night cycle meaningful.
