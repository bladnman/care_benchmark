## System-level intent

- **Notice, never announce.** The plan repeatedly protects a quiet product register where the aviary is perceived through subtle changes rather than explicit system messaging. This shows up in the offline behavior, where an offline banner is rejected because it would be an "offline" announcement and "breaking the felt-aliveness." It also appears in loading ("quiet-field loading state persists (no spinner)"), bird addition ("a subtle cue in the aviary scene" rather than "a modal popup"), rollout gates ("Zero 'Welcome back' text"), and the risk section's "notice audit."

- **Alive, not gamified or custodial.** The plan excludes "streaks, scores, achievements, badges, levels, calendars, visit-frequency counters" and also excludes hunger, distress, death, and "decay-on-neglect." Drift is "monotonic toward expressive." This intent appears in the scope exclusions, the non-negative drift function, the two-week absence calibration target where "personality plateaus, doesn't regress," and the risk framing that too-fast drift would make the product "feel like a Tamagotchi."

- **Slow, cumulative expression.** The product is meant to reward attention without becoming a direct progress meter. The drift target is "measurable in instruments at ~1 week of regular visits; visible to the user at ~3 weeks." Notebook entries are "rare," roughly "one every few days." Bird offers are tied to "aviary age, not visit count." These choices make change legible over time while avoiding visit-frequency pressure.

- **Server-canonical, single-writer state.** The plan's core architecture is that "the server is the only writer of personality state" and "the client renders snapshots, never owns state." Multi-device sync is deliberately described as "not sync" but "two clients independently pulling the same canonical server state." Conflict handling is "prevented by design, not resolved," with the simulation tick as the serial writer.

- **Privacy by data-boundary, not just policy.** The plan uses a synthetic account UUID "never derived from email," stores email encrypted, avoids email as a join key, and bounds telemetry to "aggregate operational only." It explicitly refuses metrics that could answer "what is this account's bird doing?" and places the simulation database on a separate PostgreSQL instance from any analytics warehouse with "no ETL job" reading from it.

- **Accessibility is first-class product texture.** The plan names "first-class accessibility" in scope and rejects accessibility as a separate checklist in the risk section: "it's the same rendering pipeline, the same prose generation, just with different output modes." This intent shows up in naturalist screen-reader narration, procedural call captions, reduced-motion cross-fades, keyboard navigation, focus management, and WCAG AA contrast.

- **Performance protects the central conceit.** The plan treats speed as part of the experience: "First bird renders (<500ms target)," TTFBird is a named metric, and bundle creep is framed as causing "the aviary already in motion" to collapse "into a loading screen." The CDN, critical bundle split, Canvas2D choice, CI bundle gates, and synthetic monitoring all serve that product feeling.

- **Naturalist vocabulary and matter-of-fact voice.** The plan enforces a specific language register. Error responses use "the matter-of-fact voice." Notebook and screen-reader prose are "naturalist prose." Appendix A says Bird is never "creature," "pet," "animal," or "character"; Listen-in is never "solo," "select," "highlight," or "pin." The "Welcome back" lint and prohibited strings protect this voice.

## Per-feature whys

### Scope

- **Two starter birds per new account**: The plan makes the starting aviary feel assigned rather than selected: "the user does not pick from a catalog." The deterministic-random assignment also preserves reproducibility from the creation seed.

- **Aviary grows to a max of seven birds**: NOT RECOVERABLE FROM PLAN

- **New species offers on a time-gated cadence**: The plan specifies "aviary age, not visit count," aligning this feature with the anti-gamification stance against calendars and visit-frequency counters.

- **Adoption flow showing the arriving bird entering the scene**: The arrival is staged as part of the aviary rather than as a detached transaction; later detail says the offer is "a subtle cue in the aviary scene" instead of "a modal popup."

- **Single-user accounts**: The plan keeps the product away from shared aviaries and social network surfaces. It explicitly excludes "multi-aviary accounts, shared aviaries" and profiles, follows, public discovery, leaderboards, comments, and friend-of-friend chains.

- **Email magic-link auth**: NOT RECOVERABLE FROM PLAN

- **Synthetic account UUID**: The account ID is "never derived from email," and email is "never used as a join key or identifier anywhere else," supporting identity separation and privacy hygiene.

- **Email change requiring new-address verification**: NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick**: The tick exists so the aviary continues to change "regardless of client connectivity" and so only the server computes drift, moods, mood timers, ambient events, and notebook entries.

- **Client-side rendering**: The client renders snapshots and interpolates for smooth motion while never owning personality state. This keeps visual aliveness local without moving canonical simulation authority to the browser.

- **Day/night cycle anchored to browser local time**: Local time drives the aviary's lighting and mood context; the data model says timezone "drives day/night cycle for the aviary," and mood transitions use "user local" morning, dusk, and night.

- **Ambient weather, leaf drift, and feather drift**: The plan uses ambient elements to sustain felt aliveness without server state for each element; leaves and feathers are spawned client-side with "No per-element server state."

- **Return-greeting**: NOT RECOVERABLE FROM PLAN

- **Listen-in**: Listen-in is an attention surface. The audio design says the focused bird calls more often and other birds less often, which "reinforces the feeling of paying attention to one bird."

- **Offer**: Offers are one of the product's interaction signals into the simulation: offer counts and types are aggregated by the tick, influence drift deltas, and can bias mood toward `content` or `curious`.

- **Settle**: Settle is a soft session-end gesture. It appears as a presence-end reason, biases all birds toward `drowsy`, and ramps all audio gains down to about 0.1.

- **Field notebook**: The notebook gives rare, read-only naturalist observations generated by the simulation tick, such as weather, chorus, quiet periods, unusual greetings, and weekly summaries.

- **Keyboard navigation**: The plan grounds keyboard navigation in first-class accessibility, with focusable canvas, bird focus, Enter/Space listen-in, Escape exit, dialog focus trap, and restored focus after listen-in.

- **Multi-device sync**: The why is the "single-canonical-server-state architecture." Devices do not sync with each other; they independently pull the same canonical state.

- **Read-only visit invitations**: Visits let a guest see the aviary "as the host sees it" while preventing social drift. The plan forbids co-presence, chat, comments, and "visitor-driven drift."

- **Revocable visit invitations expiring after 30 days**: NOT RECOVERABLE FROM PLAN

- **First-class accessibility**: Accessibility is product-complete behavior, not a separate track. The plan requires narration, reduced motion, procedural captions, WCAG AA contrast, and full keyboard navigation in scope.

- **Export JSON snapshot of aviary state**: NOT RECOVERABLE FROM PLAN

- **Soft-delete with 30-day recovery window**: The rationale present in the plan is recoverability: deletion is soft at first, and restore cancels deletion "within 30-day window."

- **Session token management and revocation**: The feature limits session lifetime and account access. Token hashes are stored, refresh rotates the token, and revocation invalidates specific sessions.

- **Aggregate operational telemetry**: Telemetry is bounded so operations can track counts, latencies, errors, and session-duration histograms without reading per-bird or per-account interaction state.

### Architecture

- **Static CDN**: The CDN delivers only static shell, bundle, CSS, SVG assets, and WebAssembly modules with "No dynamic content," supporting fast delivery and keeping account state behind the API.

- **API Gateway**: The gateway is deliberately thin: it handles auth, session validation, state-pull, event append, rate limits, and event-log writes.

- **Simulation Service**: The simulation service exists to centralize tick running, drift compute, mood transitions, ambient weather, and notebook generation.

- **Single PostgreSQL primary**: The plan explicitly rejects microservice-database-per-service because tick processing reads events and writes personality, mood, and notebook "in one short transaction per account." Splitting this would create distributed-transaction coordination for an operation "that must be serial."

- **Partitioning by account UUID**: Partitioning is the preferred scaling path if tick processing bottlenecks: run parallel tick workers by account UUID with "no shared state" instead of splitting the schema.

- **TypeScript and Node.js or Bun for API Gateway**: The rationale is shared types with the frontend and a thin gateway whose work is routing auth, event append, and state pull.

- **TypeScript and Node.js for Simulation Service**: The plan says simulation math is floating-point and state-machine logic, "not CPU-bound," and shared type definitions matter across layers.

- **PostgreSQL 16+**: The plan chooses it for relational modeling, strong JSON support, mature partitioning, and because access is gated in the service layer rather than through row-level security.

- **Preact or Solid frontend**: The rationale is bundle size: the 2MB gzipped budget "rules out larger frameworks," while WebAudio can be accessed directly without an audio dependency.

- **Transactional email service**: NOT RECOVERABLE FROM PLAN

- **Server-only writer of personality state**: This prevents clients from mutating the hidden simulation directly; clients append events, and only the tick computes drift and mutates personality vectors.

- **Client renders snapshots, never owns state**: The client can animate smoothly while personality state remains canonical and is "never cached client-side beyond the current rendering frame."

- **No client-to-client communication**: This keeps multi-device behavior as canonical state pull rather than peer sync.

- **Tick independent of client sessions**: This lets the aviary continue even when no tab is open, with fast-forward behavior for long-dormant accounts.

### Data Model

- **Encrypted account email stored once**: The plan separates identity from address: email is encrypted and never used as a join key or identifier elsewhere.

- **Pending email field**: NOT RECOVERABLE FROM PLAN

- **Account timezone**: Timezone is stored because it "drives day/night cycle for the aviary."

- **Account settings for reduced motion, captions, and visit notifications**: These settings persist user preferences for accessibility and visit behavior.

- **Stable bird ID**: Bird identity is durable: the ID is "never regenerated, never reassigned" and all bird identities are stable across migrations, species-pool updates, or name changes.

- **Bird display order**: The plan uses stable display order for rendering, so the scene can sort birds consistently from 1 to 7.

- **Personality vectors as stored canonical rows**: The plan avoids runtime recomputation from event history. The stored value is canonical, and only the simulation tick writes it.

- **Hidden personality floats**: The plan keeps numerical personality values out of the client response; snapshots return mood labels and procedural rendering parameters instead of raw floats.

- **Plumage saturation monotonic non-decreasing**: This preserves the "monotonic toward expressive" product stance and ensures plumage never regresses.

- **Mood rows with transition reason**: The mood table carries current mood, entry time, and last transition reason so the tick can handle duration-based transitions and debugging.

- **Append-only interaction events**: The event log is never modified or deleted because it is "the audit trail for the simulation."

- **Notebook entries with tags**: Notebook entries capture naturalist prose and optional internal tags such as `greeting`, `weather`, `chorus`, and `quiet_morning`.

- **Visit invitations**: The data model supports one-time, revocable, expiring read-only visit links by storing token, status, creation, expiration, and revocation.

- **Visit log**: NOT RECOVERABLE FROM PLAN

- **Session tokens stored as hashes**: Hashing plus revocation supports session control; the sync section says replaying a token is limited by rotation on refresh.

- **Magic links with `used_at`**: First-use invalidation prevents magic-link replay; replay returns 401.

### API Surface

- **Auth request-link returning 200 always**: The plan states the rationale directly: "to avoid email enumeration."

- **Auth verify-link invalidating token on use**: This prevents magic-link replay by setting `used_at` when consumed.

- **Auth refresh rotating session token**: Rotation limits the token replay window.

- **Auth signout and session revocation endpoints**: They let the user revoke active sessions and specific devices.

- **Aviary snapshot endpoint**: This is the canonical state pull used by clients; optional `since` can support delta-only responses but the full payload is small enough that delta is not load-bearing at launch.

- **Event append endpoint with idempotency key**: The endpoint gives clients one way to record interaction events while leaving personality writes to the server tick.

- **Notebook pagination**: NOT RECOVERABLE FROM PLAN

- **Visit invitation endpoints**: They implement rate-limited host invitations, host revocation, logs, and read-only visitor state fetches while returning 403 for expired or revoked tokens.

- **Visitor view returning same snapshot shape with `read_only: true`**: The visitor sees the host aviary in the same state shape but cannot drive drift or interact as an owner.

- **Account settings endpoint**: It supports timezone and preference updates needed by day/night, reduced motion, captions, and visit notification behavior.

- **Account change-email endpoint**: NOT RECOVERABLE FROM PLAN

- **Account export endpoint emailing a download link**: NOT RECOVERABLE FROM PLAN

- **Account delete and restore endpoints**: They implement the 30-day soft-delete recovery behavior.

- **Matter-of-fact error response convention**: Error copy stays in the product voice: "Your session timed out. Sign in again to keep watching."

### Simulation Engine Design

- **Tick loop reading unprocessed events in order**: Ordered event consumption lets the tick aggregate presence, listen-in, offers, settle events, and ambient context before computing state changes.

- **One transaction per account in the tick**: Writing vectors, moods, notebook entries, and `last_ticked_at` together protects the serial account operation described in the architecture section.

- **`last_consumed_event_id` pointer**: This avoids event skipping from offset-based pagination and gives the tick a monotonic consumption boundary.

- **Drift as a low-pass filter**: The drift function is calibrated so change is instrument-detectable around week 1 and visible around week 3, avoiding both instant personality shifts and invisible stagnation.

- **Non-negative drift deltas**: This protects the no-neglect, no-regression design: "no negative deltas; personality plateaus, doesn't regress."

- **Per-bird drift accumulator with decay**: The mild exponential decay makes "very old presence-time" fade from influence without creating negative drift.

- **Calibration harness before launch**: The harness verifies week-1 sub-perception, week-3 visible deltas, no negative absence drift, and strictly monotonic plumage saturation.

- **Weighted mood transition rules**: Mood reflects time of day, interaction, weather, bird-to-bird signals, personality, and settle rather than a single trigger.

- **Mood hysteresis threshold**: The plan states the reason: a new mood must outscore current mood by 0.15 "to prevent oscillation."

- **Rare notebook entry generation**: Entries are rare, roughly "one every few days for an active aviary," so observations stay notable rather than becoming a feed.

- **Notebook template grammar and deduplication**: Templates keep the naturalist voice procedural, while deduplication prevents "identical entry within 48 hours."

- **Species pool with seed personality values**: Species have distinct silhouettes, palettes, call motifs, and personality ranges, giving each bird a species-shaped baseline.

- **Deterministic seed values at adoption**: Seeds use `(account_id + bird_id + adopted_at)` so actual seed personalities are deterministic across replays.

- **Bird-to-bird chorus interaction**: When one bird calls, a bird with high vocal frequency and social warmth may join, supporting organic chorus behavior.

- **Bird-to-bird wary propagation**: Nearby birds receive a wary-transition bump when another bird is wary, making mood local to the scene rather than isolated per bird.

- **Close-perch event**: When warm birds share the front perch, rendering can show them near each other, making social warmth visible without exposing the underlying numeric trait.

- **Bird addition by aviary age**: The plan uses age thresholds rather than visit count to avoid turning growth into a visit-frequency mechanic.

- **New bird cue as silhouette resolving over sessions**: The cue is subtle and in-scene, matching "notice, never announce" and avoiding modal interruption.

### Sync Model

- **Single canonical aviary state per account**: The state rows in PostgreSQL make sync a property of the single-writer architecture.

- **Snapshot pull on tab navigation, visibility, focus, and keepalive**: These pulls refresh the client from canonical server state when attention returns or enough time passes.

- **Delta-only snapshot as stretch goal**: The full snapshot is only about 2KB to 5KB, so deltas are an optimization rather than a launch dependency.

- **Client interpolation between snapshots**: Interpolation over about 500ms avoids teleporting birds; perch changes cross-fade, mood changes wait for idle-cycle expression, and calls adjust on call boundaries.

- **Conflicts prevented by design**: The plan avoids conflict resolution by making clients unable to write personality and by ordering append-only events for the tick.

- **Presence deduplication across devices**: If two devices send conflicting presence starts, the tick picks the first and ignores nearby duplicates.

- **Late-arriving event cutoff**: Events older than two tick windows are dropped because they "cannot usefully contribute to drift" and old events have near-zero weight under exponential decay.

- **No offline mode**: The plan keeps the last-known snapshot rendered and queues a small number of in-memory events, but does not promise full offline behavior.

- **No offline indicator**: The plan states the rationale as a "notice, never announce" decision: an offline banner would announce system state and break felt-aliveness.

### Frontend Rendering Pipeline

- **Preact or Solid prototype decision**: The choice is deferred until animation-frame performance is tested with "7 birds + ambient effects at 60fps."

- **Critical bundle and lazy chunks**: The aviary shell and first-bird path stay critical; settings, accessibility settings, visit invitation flow, and account management are lazy-loaded to protect first render and bundle budget.

- **Quiet-field loading state**: The loading state avoids a spinner so the aviary begins as a quiet scene rather than a system loading screen.

- **Early snapshot request before bundle finish**: Starting the state request around 50ms after navigation helps first bird render as soon as the bundle is ready.

- **Edge-cached snapshot with short TTL**: The short cache absorbs "tab-reload storms" without making state freshness load-bearing for long.

- **First bird renders under 500ms**: This supports the plan's core aliveness target: the first bird should appear before the product collapses into a loading experience.

- **Canvas2D rendering**: Canvas2D is chosen because DOM/SVG could exceed the 60fps budget with animated birds and ambient effects, while WebGL is "overkill" and adds shader, text, and cross-browser complexity.

- **Single canvas scene draw order**: Drawing bottom-to-top lets the plan layer sky, foliage, perches, birds, foreground, and overlays coherently in one render loop.

- **Slow lighting transitions**: Eased lerp over several minutes makes day-to-evening a "slow cross-fade, not a hard cut."

- **Top bar as DOM overlay**: The top bar is DOM rather than canvas for "native accessibility tree exposure."

- **Call captions as DOM overlays**: Captions appear near calling birds and can use live-region behavior for screen-reader users.

- **Procedural bird rendering from species silhouettes**: Species silhouettes plus hue, saturation, pose, and position parameters make visual expression derive from snapshot state without exposing personality floats.

- **Perch transition Bezier arc**: The arc is explicitly to "read as a short flight rather than a slide."

- **Idle micro-motion scheduler**: Mood-adjusted random intervals and no-repeat penalty keep birds from feeling mechanical.

- **Breathing animation**: The subtle scale oscillation exists "to prevent the bird from reading as frozen."

- **Reduced-motion cross-fade path**: Reduced motion replaces frame-by-frame animation and flight paths with cross-fades while keeping simulation-affecting behavior unchanged.

- **Keyboard navigation implementation**: The focusable canvas, arrow-key bird navigation, Enter/Space listen-in, Escape exit, offer dialog, and focus return make the aviary operable without a mouse.

### Audio Pipeline

- **Client-side WebAudio synthesis**: The plan downloads no recorded call files; all calls are synthesized from motif data in the browser.

- **Species call grammar**: Motifs and transformation rules give each species a library of phrases without storing waveforms.

- **Per-bird timbre from personality vector**: Stable formant shifts mean "Pip always sounds like Pip" across sessions.

- **Independent random call timers**: Exponential timing and unsynchronized birds make choruses "feel organic."

- **Listen-in call scheduling**: The focused bird calls more often and other birds less often, reinforcing the feeling of attention.

- **Audio gain by listen-in state**: Gain ramps foreground the focused bird and quiet the rest during listen-in.

- **Audio gain by perch zone**: Front-perch birds are louder than back-perch birds, giving spatialization by volume without stereo complexity.

- **Evening and night quieter calls**: Time-of-day gain keeps calls consistent with the aviary's lighting and mood.

- **Settle gain ramp**: Settle ramps all gains down and holds them low, supporting the soft session-end gesture.

- **Chorus soft limiter**: The limiter prevents clipping when independent calls are mixed.

- **Chorus events noted for notebook generation**: The audio remains client-side, but chorus events become material for naturalist notebook entries.

- **AudioContext failure fallback to captions**: If audio fails, the aviary remains visually complete, captions silently turn on for the session, and no error message is shown.

- **AudioContext created on first user gesture**: This satisfies autoplay policies while allowing the aviary to be visually alive before audio can start.

- **Call synthesis memoization**: Caching motif-transform buffers for about 60 seconds avoids repeated synthesis work.

- **AudioWorklet with ScriptProcessor fallback**: The plan uses off-main-thread audio when available and a browser fallback otherwise.

- **Bounded audio buffers**: The buffer cap prevents unbounded memory growth.

### Accessibility Surfaces

- **Screen-reader live-region narration**: Narration gives semantic aviary state as naturalist prose using bird species, names, moods, perch zones, calling state, time, weather, and settled state.

- **Narration queue with at most one pending update**: The plan states the reason: the user hears "the latest state, not a backlog."

- **Priority narration for user-initiated events**: Return greeting, offer response, and settle get immediate updates because they are user-initiated.

- **Call captions from call composition**: Caption text is generated from motif structure, pitch range, trill presence, and pause pattern so it describes the actual procedural call.

- **Chorus caption offsetting**: Captions for simultaneous birds are vertically offset "to avoid overlap."

- **Focus ring and virtual bird focus**: The focus indicators are designed to remain visible against both bright and dim aviary states.

- **Icon buttons with aria-labels**: The plan exposes top bar controls to assistive technology, including "field notebook" and "account and settings."

- **Offer dialog as accessible modal**: The `<dialog>` has `aria-modal`, focus trap, Escape close, and keyboard navigation.

- **Reduced-motion setting monitored and stored**: The product honors both `prefers-reduced-motion` and a server-stored user setting.

- **WCAG contrast requirements**: All user-copy surfaces, focus indicators, and interactive borders must meet AA or specified contrast thresholds.

### Performance Budgets and Observability

- **Bundle budgets**: CI fails when gzip thresholds are exceeded, preventing bundle size creep from harming the first-bird experience.

- **TTFBird performance mark**: The plan measures navigation start to first bird drawn because first visible bird is the core perceived-readiness metric.

- **60fps render budgets**: Idle and listen-in transitions must stay at 60fps so motion and audio focus changes do not break the illusion of a living scene.

- **30-minute memory no-growth budget**: Long sessions are tested for leaks by requiring heap growth slope to be at or below zero.

- **Server latency budgets**: State snapshot, event append, tick duration, and email delivery thresholds make the canonical-server model operationally measurable.

- **Aggregate API metrics**: API counts, latency, and errors are collected per endpoint and status code with "No per-account dimension."

- **Simulation tick metrics**: Tick phase timing and anonymized drift-delta distributions help detect calibration or event-loss issues without exposing specific accounts.

- **Client-side RUM limits**: RUM collects load timing, TTFBird, frame timing histograms, and audio errors but no per-account, per-bird, or interaction-event data.

- **Sanitized error tracking**: Account UUIDs, bird names, and emails are stripped from error context before submission.

- **Synthetic monitoring**: Headless browsers from multiple regions measure TTFBird, render consistency, and audio-context availability.

- **Deliberately unmeasured bird/account behavior**: The plan refuses metrics derivable from personality vectors or interaction events so telemetry cannot answer "what is this account's bird doing?"

- **No ETL from simulation database**: The data pipeline boundary enforces the privacy line mechanically.

### Rollout Plan

- **Alpha phase**: Internal launch validates full feature set, 2-bird aviaries, green CI, TTFBird, and tick p99 before outside users.

- **Closed beta phase**: Invited users validate drift calibration at the 1-week mark, personality-vector data safety, and absence of "Welcome back" text.

- **Open beta phase**: Larger self-serve signup validates p99 tick latency, support ticket rate, and accessibility audit completeness.

- **GA launch gate**: GA waits until all beta gates are sustained for 2 weeks.

- **Bird-cap ramp**: The plan states the reason directly: validate audio-pipeline chorus behavior at increasing counts before exposing 7-bird choruses to everyone.

- **Launch instrumentation**: TTFBird, tick p99, audio-context failures, endpoint errors, and anonymized session-duration histogram are tracked from alpha to catch core reliability issues early.

- **Pre-launch drift calibration**: Synthetic presence patterns verify drift is measurable at week 1, visible at week 3, non-regressive after absence, and monotonic for plumage.

- **Pre-launch rendering test**: A throttled 5-year-old laptop profile verifies 60fps idle motion under constrained hardware.

- **Pre-launch audio stress test**: Seven max-vocal-frequency birds calling simultaneously verifies no clipping, no audio-context crashes, and no memory growth.

- **Pre-launch accessibility audit**: Narration quality, keyboard nav, and reduced-motion aesthetics are checked before alpha.

- **Pre-launch security review**: Magic-link replay protection, token rotation, email enumeration resistance, and synthetic UUID hygiene are reviewed before alpha.

### Risks

- **Drift calibration risk**: The plan's why is that too-fast drift makes the product feel like a Tamagotchi, while too-slow drift makes it feel like a screensaver.

- **Drift recalibration contingency**: Existing personality vectors are not retroactively adjusted because drift rates are hidden from users; new rates apply only to future ticks.

- **Sync correctness risk**: Silent event loss or concurrent tick overwrites would make drift slower than expected without obvious symptoms.

- **Single-writer-per-account mitigation**: `SELECT ... FOR UPDATE` prevents two tick workers from processing the same account concurrently.

- **Synthetic test account mitigation**: A known event sequence compared to expected personality values can alarm on hidden drift failures.

- **Audio uncanniness risk**: Bad procedural calls can break the spell and collapse the product's "affective spine."

- **Hand-tuned motif mitigation**: Motifs are designed by someone with synthesis expertise, with bounded pitch and tempo transformations to avoid unnatural variation.

- **Ship fewer species contingency**: If a species's audio is unsalvageable, the extensible species pool lets the team ship with fewer species and introduce the problematic one later.

- **Accessibility regression risk**: Shipping incomplete narration, captions, reduced motion, or keyboard nav would violate the "first-class, not a checklist" stance.

- **Accessibility definition of done**: A feature is not complete until keyboard nav, screen-reader narrative, reduced-motion path, and contrast pass.

- **Bundle size creep risk**: Size creep threatens TTFBird and the "aviary already in motion" conceit.

- **Bundle enforcement mitigation**: CI size gates, import lint rules, and bundle visualization catch oversized additions during review.

- **Server-side tick scalability risk**: If tick work exceeds the one-minute interval, the simulation falls behind and users see stale state.

- **Tick scalability mitigation**: UUID hash partitioning and stateless workers allow horizontal scaling by shrinking each worker's range.

- **Notice discipline erosion risk**: Seemingly harmless toasts, banners, spinners, or "Welcome back!" copy can degrade the product register incrementally.

- **Notice discipline mitigation**: Prohibited-string linting, PR checklist review, and a "notice audit" mechanically defend the product voice.

### Appendices

- **Shared vocabulary map**: The vocabulary map prevents domain drift by fixing terms such as Bird, Aviary, Call, Mood, Personality vector, Drift, Presence, Listen-in, Offer, Settle, Field notebook, Visit, and Tick.

- **Calibration harness design**: The standalone TypeScript harness makes drift behavior testable in CI with synthetic accounts, schedules, CSV outputs, and assertions for week 1, week 3, plumage, and absence.

- **Species SVG asset spec**: Single-path, small SVG assets support canvas fill colorization, pose variants, and the bundle budget.
