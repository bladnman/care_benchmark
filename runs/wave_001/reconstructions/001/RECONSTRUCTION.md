## System-level intent

- "Feels alive, not robotic" is the primary product test. It shows up in the product framing as the "primary acceptance criterion," in the quiet loading/first-frame instructions, in continuous mood-shaped motion, in procedural call variation, and in the requirement that "the server simulation must advance while no client is connected."
- The product is built around "measured presence" and "quiet interactions," not pressure. This shows up in presence-time as the "dominant input," listen-in/offers/settle as small gestures, "neglect never applies negative drift," and the explicit rejection of "any mechanic that punishes absence."
- Canonical aviary state is server-authored. The plan repeats that there is "no client ownership of canonical state," "clients read snapshots and append events; they never merge state," and there is "no last-write-wins for personality, mood, or canonical bird state."
- Continuity matters more than instant reaction. This appears in persistent bird identity, persistent personality vectors, persistent mood, mood that "resets on a daily-ish cadence, not on open," snapshots with transition metadata, and the rule that the first frame appears already in progress rather than waking up.
- The product voice is naturalist, calm, and non-gamified. The plan calls for "naturalist prose," "lowercase, present tense, specific, no exclamation," "matter-of-fact" system surfaces, no "welcome back," no streak language, no scores, no badges, and no exposed personality labels.
- The aviary scene should carry the experience through behavior. This shows up in "no UI chrome inside the scene," no instructional in-app prose explaining the charm, return-greeting with "no UI copy," top-bar controls outside the scene, and the field notebook as a reading surface outside the aviary.
- Accessibility is part of the actual aviary experience, not an alternate static mode. The plan says "Accessibility ships in v1 as part of the core product," that screen-reader narration comes from "the same canonical state as the visual scene," and that reduced motion should "preserve calls, captions, drift, mood, notebook, and interaction semantics."
- Privacy boundaries are product boundaries. This shows up in synthetic IDs, encrypted email stored "exactly once," "aggregate-only operational telemetry," the rule that the simulation database is not an analytics warehouse source, and disallowed telemetry that could reconstruct "a user's relationship with their birds."
- Sharing is constrained to read-only visits, not social-network creep. This shows up in named email invites, one-time links, expiry, revocation, "No event endpoint" for visitors, no visitor presence in host simulation tables, no public discovery, no co-presence markers, and "off-by-default visit notifications."
- The plan treats subtle correctness as the hard part. It says the product's hardest problems are "subtle correctness problems rather than breadth of feature surface," then invests in simulation fixtures, accelerated multi-week calibration, accessibility QA, performance soak tests, privacy review, and tone/prohibited-language checks.

## Per-feature whys

### 1. Product framing and v1 scope

- Browser-only app for current major Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN
- Magic-link email sign-in: NOT RECOVERABLE FROM PLAN
- Per-device sessions and session revocation: The plan wants account/session surfaces where revocation "must invalidate the token immediately," so device sessions are controllable and not just passive login artifacts.
- Verified email changes: NOT RECOVERABLE FROM PLAN
- Export and deletion: NOT RECOVERABLE FROM PLAN
- One canonical aviary per account: The why is canonical continuity. There is "one canonical aviary state per account," clients "never merge state," and multi-device conflict handling avoids competing aviary states.
- Two starter birds and a cap that grows toward seven: The plan wants the aviary to start with life already present, then ramp count only after "performance, recognizability, and chorus-mix checks" stay healthy.
- Persistent bird identity: The rationale is continuity and recognizability over time; bird IDs, species, names, call seeds, mood, perch intent, and snapshots are server-owned canonical state.
- Persistent personality vectors: The rationale is gradual expressive drift driven by measured presence and interactions, while keeping trait values hidden from normal product surfaces.
- Persistent mood: The rationale is continuity across sessions; mood "persists across sessions" and is computed through ticks rather than defaulting on open.
- Server-authored simulation tick: The rationale is that the aviary must advance while no client is connected and avoid the client becoming "an alternate simulation source."
- Event-log-driven interaction processing: The rationale is ordered, idempotent processing; events are append-only, processed by tick checkpoint, and safe to retry after a crashed tick.
- No client ownership of canonical state: The rationale is sync correctness and continuity; the client renders and interpolates, while simulation code is the only writer of personality, mood, perch intent, call schedule, weather, and notebook observations.
- Single-scene responsive aviary: The rationale is a focused horizontal scene with "no panning, scrolling, or zooming," where the aviary itself carries the experience.
- Three perch zones: The rationale is to map server perch intent to "visual depth, scale, and parallax" without letting the user drag birds or assign perches.
- Local-time day/night: The rationale is mood and visual continuity; day/night phase is based on local timezone and server snapshot phase, with gradual palette shifts.
- Rare ambient weather: The rationale is subdued ambient life; weather modulates posture and mood but is "rare and subdued."
- Ambient visual motion: The rationale is that the scene feels alive, with client-owned leaf/feather drift that remains non-stateful and removable for reduced motion.
- Top-bar controls: The rationale is to keep affordances outside the scene; the top bar contains account/settings, accessibility, notebook, and offer controls while the aviary has no chrome.
- Quiet loading field: The rationale is avoiding "spinner, progress bar, skeleton card, or machine-like loading UI."
- First-frame in-progress rendering: The rationale is that the aviary appears already underway, with no normal-load wake-up animation.
- Return-greeting: The rationale is that a returning user is noticed by behavior, "never by a welcome toast," with at most one primary greeter and no all-birds-in-unison effect.
- Listen-in: The rationale is attention rather than control; it rebalances the audio mix, posts attention events, and adds bird-specific drift toward social warmth and vocal frequency.
- Offer: The rationale is a quiet gesture with small drift effects, server cooldowns, and acceptance criteria that it is "not repeatable stat buttons."
- Settle: The rationale is to quiet mood and end presence cleanly without punishment; it is optional, undoable for five seconds, and equivalent to tab close at the engine level.
- Field notebook: The rationale is rare naturalist observations, "not logs," not a feed, not badges, and not user-behavior scoring.
- Precise presence accounting: The rationale is honest drift processing; presence windows are server-derived so background tabs, hidden windows, or unattended laptops do not accrue presence-time.
- Adoption and naming: The rationale is gradual arrival without gamified framing; new birds are age-based/server-selected, users name birds, and there is no species catalog or rarity list.
- Procedural WebAudio call synthesis: The rationale is varied, recognizable bird identity without recorded loops; calls remain identifiable across mood and drift.
- Listen-in mix: The rationale is that listen-in should "feel like attention, not channel switching," so other birds ramp down but are never silent.
- Call captions: The rationale is captions grounded in the "actual call grammar event," not static species text, and usable as fallback when WebAudio is unavailable.
- Graceful silence: The rationale is that if WebAudio is unavailable or denied, the aviary still loads alive and plays in silence with captions on by default.
- Screen-reader narration: The rationale is to carry the actual aviary experience through prose from canonical state without exposing mood labels, perch indices, or trait numbers.
- Reduced-motion rendering: The rationale is accessibility without losing semantics; motion becomes slow cross-fades while calls, captions, drift, mood, notebook, and interactions remain.
- Keyboard navigation: The rationale is that every v1 interaction remains operable through focus order, arrows within the bird group, Enter, and Escape.
- Optional read-only visit invitations: The rationale is constrained sharing: named email invite, one-time link, 30-day expiry, revocation, visit log, and visitor activity that cannot affect host drift.
- Aggregate-only telemetry: The rationale is operational and performance observability while excluding per-bird and per-account relationship data from analytics.

### 2. Target architecture

- Three-part architecture of web client, API service, and simulation worker: The rationale is a clean boundary between rendering/interactions, authenticated service surfaces, and server-side behavioral state.
- TypeScript browser client: The rationale is to render the scene, synthesize calls, collect events, present account/settings flows, and interpolate snapshots while never computing or writing personality drift.
- API service: The rationale is to centralize auth, accounts, visit permissions, event ingestion, exports, settings, and state reads.
- Simulation worker: The rationale is to consume append-only events, apply drift and mood deltas, advance ambient state, write canonical snapshots, and emit notebook candidates.
- Keeping API and simulation in the same deployable initially: The rationale is reduced operational complexity, provided the clean module boundary is preserved.
- Server-owned canonical state: The rationale is continuity for bird IDs, species, names, personality, mood, perch zones, weather, settled state, notebook, settings, invites, and interaction checkpoints.
- Client-owned ephemeral presentation details: The rationale is local rendering freedom for frame interpolation, pose curves, ornament generation, audio execution, focus state, and local accessibility display before persistence.
- CDN/edge-served shell and compact assets: The rationale is fast initial delivery and first-bird rendering.
- Edge-cached bootstrap envelope where possible: The rationale is fast first state, while avoiding sensitive state at CDN unless authenticated edge logic is used.
- Relational database for accounts, birds, aviaries, events, snapshots, notebook, sessions, invites, and jobs: NOT RECOVERABLE FROM PLAN
- Queue or database-backed lease table for ticks: The rationale is that each tick must be idempotent and safe to retry.
- Object storage for exports with short-lived signed links: The rationale is generated export delivery through temporary links emailed to verified addresses.
- Transactional email provider: The rationale is to send magic links, invite links, export links, and account email verification.

### 3. Domain model and persistence

- Synthetic UUIDs throughout: The rationale is privacy and boundary control; foreign keys, log fields, queue keys, and metric dimensions use synthetic account IDs or non-PII operational IDs.
- Email stored exactly once on the account record and encrypted at rest: The rationale is to keep email out of partition keys, metric dimensions, log fields, and service identifiers.
- `sessions`: The rationale is per-device session tracking with hashed tokens, labels, last seen, expiry, and immediate revocation.
- `magic_links`: NOT RECOVERABLE FROM PLAN
- `aviaries`: The rationale is to hold the canonical single-aviary state: settled state, timezone, weather, day/night phase, adoption eligibility, and bird cap.
- `birds`: The rationale is persistent identity and behavior: species, user name, mood, perch intent, motion phase, call signature seed, personality vector, drift accumulator, offer cooldowns, and version.
- `personality_snapshots`: The rationale is "server-side calibration and rollback," not user exposure or analytics.
- `interaction_events`: The rationale is append-only interaction processing with idempotency, validation, and processed tick tracking.
- `presence_windows`: The rationale is making drift processing "honest and debuggable" without treating raw pings as drift directly.
- `aviary_snapshots`: The rationale is a compact canonical state and short history for rendering, not per-frame animation data.
- `notebook_entries`: The rationale is stable stored prose tied to rare observation sources and visible order.
- `visit_invites` and `visit_sessions`: The rationale is opt-in read-only visit access with encrypted visitor email, hashed token, expiry/revocation, and visit-duration metadata.
- `outbox_email`: The rationale is transactional email queueing for auth, invites, exports, and verification.
- Personality vector internal ranges: The rationale is normalized server-side calibration while staying absent from user API, UI, accessible labels, export prose, and account settings except raw export JSON.
- Mood enum with stable values: The rationale is "enough states to support visible variation, not a sprawling emotional taxonomy."
- Mood transition metadata: The rationale is to avoid snapping and let clients crossfade or animate toward the new posture.
- `presence_ping`: The rationale is to record visibility, focus, and recent activity claims for precise presence accounting.
- `presence_end`: The rationale is to close presence on settle, hidden, blur, unload, or stale-window timeout.
- `listen_in_start` and `listen_in_end`: The rationale is to capture bird-specific attention duration.
- `offer`: The rationale is to record offer type and target context with idempotency so the tick can apply cooldowns and drift.
- `settle_start` and `settle_undo`: The rationale is to make settle reversible and tied to snapshot version.
- `adoption_name_set` and `bird_renamed`: The rationale is that names mutate user-facing bird names through API validation, not simulation drift.
- Settings events: The rationale is audit and multi-device consistency, not simulation input.
- Visitor logs separate from host simulation tables: The rationale is that visitor presence and interactions "never feed the bird engine."

### 4. API surface

- `POST /api/auth/magic-link` neutral success response: NOT RECOVERABLE FROM PLAN
- `POST /api/auth/magic-link/consume`: The rationale is one-time token consumption, account creation/resume, starter aviary creation when needed, and per-device session creation.
- `GET /api/account`: The rationale is to expose account settings, sessions, deletion state, notification preferences, and export/delete availability in one account surface.
- `PATCH /api/account/settings`: The rationale is system settings, accessibility preferences, visit notification preference, and timezone consistency.
- Email-change endpoints: NOT RECOVERABLE FROM PLAN
- Session revoke endpoint: The rationale is immediate device-session revocation.
- Account export endpoint: The rationale is asynchronous export with emailed link when ready.
- Account delete and cancel endpoints: The rationale is deletion with restoration inside the 30-day soft-delete window.
- `GET /api/aviary/bootstrap`: The rationale is the first fast state request, returning canonical snapshot, manifest references, server time, version, settings needed for rendering, and whether adoption naming is needed.
- `GET /api/aviary/snapshot`: The rationale is no-change/latest snapshot reads for visibility return, long frame gaps, and visible keepalive.
- `POST /api/aviary/events`: The rationale is batched event append with idempotency, per-event acceptance/rejection, and authoritative snapshot version.
- `GET /api/aviary/notebook`: The rationale is paginated read access to the notebook.
- Bird rename endpoint: The rationale is account-ownership validation before changing user-facing names.
- Dynamic offer library endpoint: NOT RECOVERABLE FROM PLAN
- Adoption availability endpoint: The rationale is to report age-based arrival "without gamified framing."
- Adoption creation endpoint: The rationale is accepting names for server-selected birds while avoiding species catalog or rarity list.
- Visit invite endpoints: The rationale is one-time named email invites, 30-day expiry, revocation, and a host account-settings view of invitations and visit log.
- Visitor consume endpoint: The rationale is a visitor session scoped to the host aviary.
- Visitor bootstrap/snapshot endpoints: The rationale is read-only visitor state, with no event endpoint except operational heartbeat for duration logging.
- Naturalist product error/message tone: The rationale is that product surfaces should use naturalist prose while avoiding "welcome back," streak language, score-like counts, and personality labels.
- Matter-of-fact system endpoint messages: The rationale is clear sign-in, sync, unsupported browser, account, visit expiration, and accessibility settings surfaces.

### 5. Server-side simulation engine

- Deterministic, testable modules with explicit seeds: The rationale is calibration, repeatability, and subtle behavioral correctness.
- Event ingestion normalizer: The rationale is to turn raw append-only events into valid tick inputs.
- Presence-window builder: The rationale is server-derived, trusted-bounds duration reconstruction.
- Drift calculator: The rationale is additive personality movement from presence, listen-in, and offers.
- Mood transition engine: The rationale is persistent, smoothed mood changes modulated by local time, weather, interactions, personality, and bird-to-bird signals.
- Perch and idle-intent planner: The rationale is server-owned continuity of bird placement and movement intent.
- Call schedule planner: The rationale is server-authored call intents and motif parameters without sending audio buffers.
- Weather/day-night updater: The rationale is ambient state continuity across time and sessions.
- Notebook selector and prose generator: The rationale is rare observation candidates and stable naturalist entries.
- Snapshot writer: The rationale is canonical rendering state with version/server time.
- Once-per-minute tick cadence: The rationale is slow simulation ownership consistent with measured presence and low-frequency state propagation.
- Tick lease: The rationale is to ensure only one worker processes an aviary at a time.
- Server-received event order: The rationale is authoritative ordering, with client timestamps used only inside trusted bounds.
- Idempotent tick processing: The rationale is that reprocessing after a crash must not double-apply drift.
- Presence-time as dominant drift input: The rationale is that birds "gradually evolve through the user's measured presence."
- Listen-in drift: The rationale is stronger bird-specific movement toward social warmth and vocal frequency, scaled and capped.
- Offer drift: The rationale is small curiosity/boldness movement with cooldowns so one session cannot saturate curiosity.
- Settle drift handling: The rationale is clean presence ending and quieted mood without material personality drift beyond already accrued presence.
- No negative drift on neglect: The rationale is "no mechanic that punishes absence"; absence cannot reduce vectors or create distress.
- Low-pass filters and daily caps: The rationale is user-visible drift only after regular use, with no single session moving a trait visibly.
- Calibration fixtures: The rationale is to inspect vector movement, mood distributions, greeting frequency, perch occupancy, and call frequency across representative users.
- Internal-only calibration instruments: The rationale is not to expose trait values through product dashboards.
- Daily-ish mood reset rather than reset on open: The rationale is continuous life over time, not session-based defaults.
- Bird-to-bird effects: The rationale is emergent aviary behavior such as alarm-like nudges, chorus, and warm birds perching nearer one another.
- Return-greeting greeter selection: The rationale is a behavior-based greeting shaped by absence length, boldness, mood, social warmth, and greeting history.
- Staggered secondary greetings: The rationale is to avoid triggering all birds in unison.
- Greeting represented in snapshot state: The rationale is server-authored continuity with procedural client rendering.
- Species motif libraries: The rationale is structured call variation through pitch contour, rhythm, timbre, and allowed ranges.
- Stable call seed per bird: The rationale is that a bird's call remains identifiable across mood and drift.
- Seven-bird mix recognizability: The rationale is each bird remains separable in the mix.
- Rare notebook observations: The rationale is notebook entries should be noteworthy, sparse, and not logs.
- Notebook trigger filtering: The rationale is to describe the aviary rather than score the user, excluding visit streaks, total time, and "you returned after X days."
- Stored notebook prose: The rationale is that old entries remain stable.

### 6. Client rendering pipeline

- Minimal shell capable of drawing quiet field and first birds quickly: The rationale is fast first-bird rendering.
- Bootstrap request or authenticated bootstrap envelope: The rationale is fast first canonical state.
- Quiet field while waiting: The rationale is avoiding machine-like loading UI.
- First bird frame as mid-preen, mid-call, settled posture, or another ongoing pose: The rationale is no wake-up animation after normal load.
- One horizontal responsive scene: The rationale is a constrained aviary view without panning, scrolling, or zooming.
- Aspect-ratio and responsive constraints: The rationale is to prevent birds being cropped on narrow phone or wide desktop.
- No bird dragging or perch assignment: The rationale is that users do not control layout.
- Gradual day/night palette shifts: The rationale is local-time continuity based on server snapshot phase.
- Non-stateful leaf/feather drift: The rationale is ambient life without adding canonical state.
- Pose/state-machine animation: The rationale is mood-shaped, continuous, species-specific visible behavior.
- Smooth blending to server target states: The rationale is avoiding abrupt snapshot changes.
- Pause rendering when hidden and fetch fresh snapshot on return: The rationale is performance plus canonical continuity.
- Reduced-motion cross-fade path: The rationale is accessible motion reduction while preserving the aviary experience.
- Top bar fade: The rationale is keeping controls available without scene chrome.
- Keyboard focus revealing the bar: The rationale is accessibility and visible focus treatment.
- No buttons, labels, badges, hover tooltips, or status icons inside the scene: The rationale is that the aviary itself carries experience through behavior.
- Return-greeting rendered from snapshot with no UI copy: The rationale is behavior-based recognition, not announcement text.
- Listen-in start/disengage routes: The rationale is local attention control through click, tap, keyboard, same bird, another bird, empty scene, or focus exit.
- Offer opened from the top bar: The rationale is keeping offer as a top-bar affordance rather than clicking a bird or placing controls in the scene.
- Offer optimistic placement only if it cannot contradict server state: The rationale is preserving canonical state authority.
- Settle lighting and five-second undo by any click in the aviary: The rationale is a reversible quieting gesture.
- Field notebook outside the scene: The rationale is a calm, read-only reading surface, not a badge feed.
- No instructional in-app prose explaining charm: The rationale is that behavior, affordances, labels, and matter-of-fact settings should carry the product.

### 7. Audio pipeline

- WebAudio for all bird calls: The rationale is procedural calls with no recorded-loop path and no recorded fallback.
- Global audio manager: The rationale is centralized AudioContext, gain buses, listen-in bus, limiting, and caption event bridge.
- Per-bird synthesizers from motif data: The rationale is identity-shaped procedural calls using oscillators, filters, envelopes, pitch curves, modulation, and seeded variation.
- Server call intents with client jitter: The rationale is server-authored timing and motif choices while preventing grid-like sound.
- Chorus mixer: The rationale is overlapping calls without phase artifacts and with per-bird recognizability.
- Listen-in gain ramp: The rationale is attention, not channel switching, with no abrupt cuts.
- Captions generated from actual call grammar: The rationale is caption text that matches the call event rather than static species strings.
- Captions near the calling bird: The rationale is readable association with the actual call while meeting contrast rules.
- Audio enablement on first permissible gesture without a modal: The rationale is that the visual aviary still loads alive and audio prompts stay quiet and matter-of-fact.
- Silence with captions if WebAudio is unavailable or denied: The rationale is graceful fallback without recorded audio.
- Reuse buffers, envelopes, and node pools: The rationale is avoiding per-call allocations and memory growth.
- 30-minute automated memory test with calls and listen-ins: The rationale is proving audio performance over a long session.

### 8. Accessibility plan

- Live region narration from canonical state: The rationale is the same aviary experience expressed as restrained naturalist prose.
- Narration cadence of roughly 30 to 60 seconds: The rationale is low-frequency narration that does not flood the screen reader.
- Prompt narration for user-initiated events: The rationale is responsiveness without losing restraint.
- Narration queue with debouncing: The rationale is preventing screen-reader flooding.
- Tab order through top bar, bird focus group, and open panel: The rationale is predictable keyboard operation.
- Arrow keys between birds, Enter for listen-in, Escape for exit/close: The rationale is complete keyboard operation of v1 interactions.
- Fully keyboard-operable menus and panels: The rationale is accessibility parity for offers, account settings, accessibility settings, and notebook.
- Focus indicators against all palettes: The rationale is visible focus treatment that does not look like game targeting UI.
- `prefers-reduced-motion` by default with override: The rationale is respecting user motion preference while allowing account-level choice.
- Removing leaf/feather drift in reduced motion: The rationale is reducing ambient motion while keeping slow color transitions.
- Captions optional unless audio fallback requires them: The rationale is user choice unless captions are needed for graceful silence.
- WCAG AA contrast for user-copy text: The rationale is readable account, error, sync, settings, and caption surfaces.
- Automated accessibility checks: The rationale is verifying focus order, ARIA roles, contrast, reduced motion, and keyboard behavior.
- Manual screen-reader QA: The rationale is checking narration cadence and prose quality beyond automated tests.

### 9. Sync and conflict model

- One canonical aviary state per account: The rationale is that clients append events and read snapshots rather than merging state.
- Snapshot request on navigation or visibility return: The rationale is resuming from canonical state.
- Low-frequency polling or lightweight stream: The rationale is modest traffic because tick cadence is slow.
- Snapshot fetch after long frame gaps or laptop resume: The rationale is discarding stale local interpolation assumptions.
- Multi-device idempotency keys and server order: The rationale is conflict prevention and ordered tick processing.
- No last-write-wins for personality, mood, or canonical bird state: The rationale is avoiding lost drift and conflicting aviary states.
- Optimistic concurrency for settings and names: The rationale is user-controlled fields can conflict, with matter-of-fact conflict surfaces.
- Server-side offer cooldowns: The rationale is consistent accept/reject decisions across devices.
- Listen-in local audio focus only: The rationale is attention as simulation signal without syncing "currently listening in" as global UI state.
- Settle as account-level session intent only for the active client: The rationale is other devices simply read the canonical state the tick computed.
- Pings only when visible, focused, and recently active: The rationale is that hidden tabs, background windows, and unattended laptops must not accrue presence-time.
- Activity timeout calibrated for quiet watching: The rationale is to allow quiet watching while still preventing unattended accrual.
- Visitor snapshot validity checks and revocation on next pull: The rationale is timely read-only access revocation.
- Visitor sessions write duration metadata only: The rationale is no greetings, presence, offers, listen-in, settle, notebook, or drift events from visitors.

### 10. Privacy, telemetry, and observability

- Per-bird interaction events and simulation history kept only for the user's aviary: The rationale is excluding them from analytics, model training, recommendation systems, and third parties.
- Simulation database not an analytics warehouse source: The rationale is preventing analytics from reading relationship data.
- Synthetic account IDs in operational logs only where necessary: The rationale is to prefer request IDs and aggregate counters.
- Email never as partition key, metric dimension, log field, or service identifier: The rationale is the encrypted email single-source rule.
- Allowed aggregate telemetry: The rationale is operational and performance health through counts, latencies, tick metrics, payload size, timing, memory, WebAudio errors, export/delete job counts, and anonymous histograms without account dimension.
- Explicitly disallowed telemetry: The rationale is preventing data products that could reconstruct a user's relationship with birds.
- Operational alarms: The rationale is catching tick latency, backlog, failed processing, stale snapshots, delivery failures, export failures, and hard-delete failures.
- Synthetic browser checks: The rationale is measuring first bird visible, idle FPS, WebAudio startup, reduced motion, and memory growth.
- Structured logs with scrubbing: The rationale is observability without emails, tokens, notebook prose, event payload details, or raw personality vectors in logs.

### 11. Performance plan

- Initial JS under 2MB gzipped: The rationale is a strict bundle budget for fast load.
- First bird under 500ms on mid-tier mobile over 4G: The rationale is immediate alive rendering.
- 60fps idle motion for 30 minutes on older laptop: The rationale is continuous idle life without degraded performance.
- No client memory growth over 30 minutes: The rationale is long-session stability.
- Kilobyte-scale snapshots: The rationale is compact state transfer, not megabyte payloads.
- Code-splitting panels and flows out of the initial bundle: The rationale is to keep the first render path small.
- Initial render limited to quiet field, two bird renderers, essential fetch, motif references, and top-bar shell: The rationale is first-bird speed.
- Compact vector or sprite assets and data-driven visual variation: The rationale is avoiding large bitmaps.
- Lazy-loaded notebook and account panels: The rationale is keeping non-initial surfaces off the critical path.
- Renderer choice based on prototype profiling: The rationale is meeting 60fps and memory budgets with the least complexity; the product "does not require 3D."
- Object pools for leaf/feather drift and audio nodes: The rationale is avoiding allocation-driven memory growth.
- Pause rendering when hidden and resume from server snapshot: The rationale is performance and canonical correctness.
- Bounded and debounced captions/narration generation: The rationale is preventing unbounded accessibility work.

### 12. Rollout plan

- Phase 0 isolated simulation and prototypes: The rationale is proving simulation, rendering, reduced motion, WebAudio, recognizability, motif grammar, pose vocabulary, naturalist prose, and privacy/telemetry schema before product data exists.
- Phase 1 account/canonical state/internal accounts: The rationale is to establish magic-link sign-in, starter aviary, server tick, quiet loading, and first-frame rendering behind an internal gate.
- Phase 2 core interactions and drift: The rationale is to add greeting, listen-in, offers, settle, cooldowns, presence windows, drift, mood transitions, bird-to-bird calls, and notebook policy before outside opening.
- Accelerated calibration before opening outside the team: The rationale is validating multi-week drift and behavior before beta users.
- Phase 3 accessibility and performance hardening: The rationale is v1 launch gating for narration, captions, keyboard, reduced motion, focus, contrast, WebAudio fallback, unsupported browser, bundle, first-bird, FPS, and memory budgets.
- Phase 4 sync, privacy, export/delete, and visits: The rationale is hardening multi-device ordering, idempotency, stale snapshots, sessions, email change, deletion, and visitor read-only boundaries.
- Phase 5 private beta and bird-cap ramp: The rationale is to start at two birds and increase toward five to seven only after drift, audio recognizability, performance, and chorus-mix checks remain healthy.
- No engagement dashboards during rollout: The rationale is aggregate-only operational metrics and avoidance of engagement/gamified tracking.

### 13. Engineering workstreams

- Simulation and data workstream: The rationale is owning schema, event log, tick lease, idempotency, snapshots, drift/mood, notebook, calibration, and simulation tests.
- Frontend scene workstream: The rationale is owning responsive renderer, pose system, interpolation, top bar, quiet loading, adoption/naming, panels, and reduced-motion renderer.
- Audio workstream: The rationale is owning motif format, WebAudio synthesis, chorus mixer, listen-in ramps, captions, fallback, and memory/performance tests.
- Accessibility workstream: The rationale is owning narration, keyboard focus, captions, contrast, reduced-motion QA, and screen-reader QA.
- Auth/account/privacy workstream: The rationale is owning login, sessions, email changes, export, deletion, settings, encrypted email, log scrubbing, and privacy surface.
- Visits workstream: The rationale is owning invite issuance, visitor sessions, read-only snapshot API, revocation, expiration, visit log, and notification toggle.
- Observability/release workstream: The rationale is owning aggregate metrics, synthetic checks, alarms, CI budgets, beta controls, and runbooks.
- Product-language guardrails in acceptance criteria: The rationale is keeping every workstream aligned on no announcement toasts, gamified counters, exposed traits, distress states, or social-network creep.

### 14. Test strategy

- Drift monotonicity tests: The rationale is additive, capped, low-pass drift and no negative drift on absence.
- Mood transition tests: The rationale is mood shaped by time of day, weather, interactions, and personality.
- Presence-window tests: The rationale is verifying visibility/focus/activity combinations.
- Event idempotency and ordered-processing tests: The rationale is preventing double application and preserving server order.
- Offer cooldown tests: The rationale is keeping offers from saturating curiosity or becoming repeatable stat buttons.
- Magic-link and session tests: The rationale is expiry, one-time consumption, and revocation correctness.
- Visit invite and visitor permission tests: The rationale is expiry, revocation, and read-only boundaries.
- Notebook sparsity and prohibited-language tests: The rationale is keeping entries rare and non-gamified.
- Multi-device integration tests: The rationale is appending events without last-write-wins state loss.
- Hidden-tab integration tests: The rationale is presence correctness.
- Tab close and settle integration tests: The rationale is both end presence without penalty.
- Visitor watch/no-event tests: The rationale is no visitor influence on host simulation.
- Export/delete integration tests: The rationale is allowed account/aviary data, no unrelated telemetry, restore inside 30 days, and dependent-record removal.
- End-to-end first account and aviary render tests: The rationale is confirming sign-in, starter birds, naming, and first render.
- Return-after-absence tests: The rationale is one varied bird greeting and no textual welcome.
- Listen-in tests: The rationale is audio ramps and all disengage routes.
- Offer tests: The rationale is mood/personality-shaped responses and cooldown behavior.
- Settle tests: The rationale is lighting shift and five-second undo.
- Notebook tests: The rationale is read-only sparse behavior.
- Keyboard-only tests: The rationale is every v1 interaction operable.
- Reduced-motion tests: The rationale is cross-fade aviary, not static fallback.
- WebAudio unavailable tests: The rationale is captions-on graceful silence.
- Visit flow tests: The rationale is invite, consume, watch, revoke, and expired-link behavior.
- Simulation calibration tests: The rationale is one-week measurable internal drift, three-week qualitative change, two-week absence without distress or negative traits, and seven-bird recognizability/performance.
- Content and tone checks: The rationale is keeping naturalist surfaces lowercase, present-tense, specific, and free of prohibited "you" announcement framing, while system surfaces remain clear, capitalized, matter-of-fact, and actionable.

### 15. Key risks and mitigations

- Drift calibration mitigation: The rationale is to avoid drift feeling like stat manipulation or irrelevance through fixtures, caps, low-pass filters, internal instruments, and two-bird beta holds.
- Presence correctness mitigation: The rationale is to prevent background tabs or unattended laptops from corrupting drift.
- Sync correctness mitigation: The rationale is to prevent lost drift or conflicting aviary states through append-only events, server-only personality writes, idempotency keys, leases, transactional checkpoints, and no last-write-wins.
- Audio uncanniness mitigation: The rationale is to avoid synthetic, repetitive, or blurry calls with motif libraries, stable seeds, mood/personality variation, chorus QA, recognizability tests, and bird-count ramp.
- Accessibility regression mitigation: The rationale is to prevent accessible paths becoming state lists or static fallbacks.
- Performance mitigation: The rationale is to protect first-bird timing and long-session memory health.
- Tone/product-creep mitigation: The rationale is to stop toasts, counters, notifications, or social loops through copy review, prohibited-language tests, design checklist, and architecture that does not compute gamified/social aggregates.
- Privacy mitigation: The rationale is to prevent emails or bird interaction data leaking into logs or analytics.
- Notebook quality mitigation: The rationale is to prevent generic event logs or overly frequent entries.
- Visit boundary mitigation: The rationale is to prevent visitor sessions influencing host birds or becoming a social-network foothold.

### 16. Acceptance checklist

- Birds already in motion on first normal session: The rationale is "not an app waking up."
- Returning user noticed by one bird through behavior: The rationale is no welcome toast.
- Presence requires visible document, focused window, and recent pointer/key activity: The rationale is precise presence accounting.
- Personality vectors persistent, server-owned, additive, hidden, and monotonic toward expressive: The rationale is gradual expressive drift without exposed stats or negative absence.
- Mood persists and changes through server ticks: The rationale is continuity across sessions.
- Procedural WebAudio calls recognizable per bird and captioned from grammar: The rationale is identity, variation, and accessible fallback.
- Listen-in rebalances without silencing the rest: The rationale is attention rather than channel switching.
- Offers as gestures with cooldowns: The rationale is avoiding repeatable stat buttons.
- Settle optional and undoable, equivalent to tab close at engine level: The rationale is no punishment or special drift exploit.
- Notebook sparse, read-only, naturalist, and not a user-behavior log: The rationale is rare observation prose rather than scoring.
- One horizontal scene with no placement controls or scene chrome: The rationale is constrained behavioral experience.
- Accessibility surfaces carry the actual aviary experience: The rationale is no lesser fallback.
- Performance budgets met: The rationale is first-bird speed, 60fps idle, and no memory growth.
- Visits named, opt-in, revocable, read-only, and unable to affect host drift: The rationale is controlled sharing without social creep.
- Aggregate telemetry excludes per-bird and per-account relationship data: The rationale is privacy boundary preservation.
- No gamification, Tamagotchi mechanics, social-network mechanics, push/ping behavior, or native-app scope in v1: The rationale is keeping v1 within the quiet aviary product boundary.
