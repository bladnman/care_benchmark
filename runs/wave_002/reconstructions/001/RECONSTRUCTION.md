## System-level intent

- Architecturally enforced non-goals. The plan treats "No gamification of any kind" as an "absolute constraint" and says the non-goals are "enforced architecturally, not just by omission from the UI." This shows up again in the event log and telemetry schemas having "no fields" for streak counters, visit-frequency surfaces, or leaderboards, and in the risk section where gamification friction is "intentional and load-bearing."

- Server-as-sole-writer authority. The plan repeatedly centers the "server-authoritative bird engine," the "Simulation Tick Worker" as "the only writer of personality vectors and mood, full stop," and the sync model's "single writer." The client is intentionally a "thin renderer and event emitter" and never computes personality, mood, or drift.

- Slow, ambient, non-game pacing. The plan resists instantaneous causality: `POST /aviary/events` returns "accepted, not yet simulated," "no single session moves things visibly," and propagation latency is "acceptable and arguably correct" because "no interaction should feel instantaneous-and-causal in a way that resembles a button-press game." The "quiet-field loading state" and no-spinner first frame carry the same product tempo.

- Monotonic expressive growth, not decay. The plan uses "monotonic-toward-expressive drift," says drift is "asymmetric (up-only) by construction," and requires "No subtraction path" in the tick code. This is tied to rejecting "Tamagotchi mechanics" such as death, hunger, decay, or visible distress.

- Privacy, PII minimization, and aggregate-only observation. The data model stores email "exactly once, encrypted," avoids email in "any other table, log line, partition key, or telemetry event," and keeps operational telemetry "aggregate-only." The metrics-emission layer "simply not accepting an `account_id` parameter" is the same principle expressed operationally.

- Accessibility as a designed v1 surface. Accessibility is "shipped with v1, not retrofitted"; reduced motion is a "designed surface, not animations-off"; and launch gating says not to launch without reduced motion and screen-reader narration "complete and reviewed." The plan treats accessibility as part of the product surface, not as a fallback.

- Naturalist voice and specificity. The plan asks for "naturalist prose," "naturalist screen-reader narration," captions in a "naturalist voice," and notebook templates "parameterized by the specific bird names/states involved." It also says the template library is content the "writing/design team owns," keeping voice load-bearing.

- One canonical aviary across clients, visitors, visuals, audio, and narration. The client/server split makes multi-device sync "a property of the architecture," visitors receive the "actual canonical state" with no special visitor rendering, and the accessibility layer reads the same snapshot so narration and visuals "can never drift into two different products."

- Procedural media over static loops. The audio pipeline rejects recorded loops: call grammar is data, calls are synthesized with WebAudio, "no two calls are byte-identical," and chorus mixing is meant to produce "a real chorus" rather than a phase-cancellation artifact from stacked recorded loops.

- Measurable, tunable, CI-gated quality. The plan repeatedly converts intent into checks: 7-day and 3-week drift calibration queries, bundle-size CI, synthetic 60fps checks, heap-snapshot memory tests, production synthetic checks, and launch gates. Calibration constants are "not hardcoded guesses" and are expected to be tuned.

- Narrow, auditable boundaries and capability absence. Services are "deliberately small," REST is chosen because fixed endpoints are easier to keep "auditable," and several constraints are enforced by missing capability: visitor tokens have no event-log write capability, clients have no endpoint accepting mood or personality values, and no audio asset pipeline exists for recorded fallback files.

## Per-feature whys

### Scope

- Single-user accounts: NOT RECOVERABLE FROM PLAN

- Magic-link auth and session tokens: The plan gives security and privacy rationale around the flow: magic-link requests must not reveal whether an email exists, tokens have 15-minute expiry, and verification tokens are single-use via a `token_hash` consumed-flag pattern.

- One canonical aviary per account: The rationale is that all clients read the same canonical store; "two clients reading the same canonical store via the same snapshot endpoint cannot diverge."

- Two starter birds at adoption: The plan says this is "fixed at launch" and "a day-one product decision, not a rollout lever."

- Species drawn from a ~6-species pool: NOT RECOVERABLE FROM PLAN

- Cap of 7 birds: NOT RECOVERABLE FROM PLAN

- Age-gated growth beyond starter birds: The rationale is pacing. The threshold table lets the team tune whether "few months -> third bird, year -> five or six" feels right, using config rather than hardcoded tick-worker logic.

- Server-authoritative bird engine: This exists so personality, mood, and drift have one source of truth; clients "never write personality state directly," and sync becomes architectural rather than a client merge problem.

- Five raw personality traits: NOT RECOVERABLE FROM PLAN

- Five-state mood enum: Mood is intentionally a "small state machine" with hysteresis so it "doesn't flicker tick-to-tick on noise."

- Monotonic-toward-expressive drift: The rationale is to make growth asymmetric and avoid decay mechanics; the plan explicitly ties this to "No Tamagotchi mechanics" and rejects any negative `PersonalityDelta`.

- Presence accounting: Presence is the dominant drift input because it represents "attention to the aviary as a whole." Settle closes the presence window so the worker does not continue accruing presence after the user has left.

- Bird-to-bird interaction as a feature: NOT RECOVERABLE FROM PLAN

- Return-greeting: NOT RECOVERABLE FROM PLAN

- Listen-in: The plan makes listen-in both a simulation signal and an audio focus behavior: it contributes to social warmth and vocal frequency for the listened-to bird, while gain ramps avoid the "channel switch" feeling.

- Offer, including seed, song fragment, and still pool: Offers create drift and mood signals. An accepted offer biases toward `content` and `curiosity`; offering near a bird at all nudges `boldness`.

- Settle: Settle has no drift contribution; its rationale is to "close the presence window cleanly" and prevent presence-time from continuing into later ticks after departure.

- Field notebook: The notebook exists as sparse, specific "naturalist prose." Trigger cooldowns implement "roughly one entry every few days," templates avoid generic event-log strings, and the evaluator excludes visit cadence so it cannot become a disguised streak counter.

- Single horizontal scene: NOT RECOVERABLE FROM PLAN

- Three perch zones: The plan uses perch zones as a rendering of personality and mood: `current_perch_zone` is derived from updated personality plus mood, with boldness and mood mapping birds toward front, middle, or back.

- Day/night cycle on local time: The rationale is continuity when no one is present. The server caches the last client timezone so the aviary can keep advancing through the night "whether or not anyone is watching."

- Rare ambient weather: The plan uses weather as a mood, rendering, and notebook input: rain can bias the aviary toward `wary` or `drowsy`, wind can bias birds toward `alert` or `wary`, and weather-adjacent observations can trigger notebook prose.

- Idle micro-motion: Idle motion lets the "user read mood from motion without a label." Mood-weighted randomness avoids a fixed loop while keeping behavior recognizably mood-shaped.

- Top-bar chrome with fade: The fade keeps chrome cosmetic and unobtrusive; the plan also stresses that this UI state is "independent of the presence-accounting system" and "not a presence signal."

- Multi-device sync via server-side simulation tick: Sync is "conflict-free by construction" because concurrent device events land in an ordered append-only log and the tick consumes both; nothing is overwritten.

- Visit-invitation social feature: The rationale is a narrowly bounded social affordance: "per-invite opt-in," off by default, and no profiles, follows, discovery, leaderboards, or comments.

- Read-only ambient visits: Visitor tokens are read-only at the capability level, with no event-log write access. Visitors see the "actual canonical state" and no special visitor rendering.

- Revocable, 30-day visit invite expiry: NOT RECOVERABLE FROM PLAN

- Naturalist screen-reader narration: Narration is generated from canonical snapshot data so screen-reader output stays aligned with the aviary. Queue depth is capped so updates do not flood the screen reader.

- Reduced-motion mode: Reduced motion is a "structurally separate render path" and a "designed visual register," scheduled inside v1 so it is not bolted on afterward.

- Call captioning: Captions are generated from the same call-grammar runtime as synthesis, so they describe what was actually played rather than guessing.

- WCAG AA contrast: The rationale is that overlaid text has to remain readable against both "the brightest (midday)" and "darkest (night)" aviary background states.

- Full keyboard navigation: Birds are canvas-rendered but receive real DOM hit-targets so Tab, arrow-key focus, Enter, Escape, and the offer panel remain keyboard-operable.

- Account export: NOT RECOVERABLE FROM PLAN

- Soft deletion with 30-day undo: The plan articulates the undo rationale through the account endpoint labeled "I changed my mind," available before hard delete.

- Aggregate-only operational telemetry: The rationale is privacy and non-exploitation: per-bird and per-account interaction data is "never aggregated, never used for ML, never shared," and metrics do not accept account IDs.

### Architecture

- Four deliberately small services: The plan separates account lifecycle, aviary reads/event writes, independent simulation ticks, and visit tokens/invites so each service has a narrow responsibility.

- Scheduled Simulation Tick Worker: The tick worker runs independently of requests so "the aviary continues without the viewer" and remains the sole writer of personality and mood.

- Thin client renderer and event emitter: Keeping the client thin makes multi-device sync architectural; the client renders snapshots and emits events but never owns personality, mood, or drift.

- Cosmetic ambient ornaments generated client-side: Leaf and feather drift avoid server round-trips because the plan explicitly treats them as "purely cosmetic, non-simulated ambient ornaments."

- Pure render pipeline boundary: "Snapshot in, pixels/audio out" prevents rendering code from becoming simulation code and keeps the renderer a pure function of snapshot, elapsed time, and accessibility settings.

- Accessibility layer reading the same snapshot: This prevents narration and visuals from drifting into "two different products."

### Data model

- Synthetic UUIDs and email stored exactly once: Synthetic IDs keep email and PII out of table keys, logs, partitions, and telemetry; encrypted email exists only on the account record.

- Separate `Aviary` entity under one account: Even though v1 has one aviary per account, the separate entity means the schema "doesn't have to be reshaped if multi-aviary ever ships."

- Session list, revocation, and device labels: The plan grounds device labels as "user-visible in session list" entries such as "Chrome on Mac," supporting account settings and session revocation.

- Stable bird IDs: Stable `bird_id` values support the "identity-continuity rule"; birds are "never regenerated, never replaced."

- Renameable bird name with no simulation effect: NOT RECOVERABLE FROM PLAN

- Public bird projection with no raw personality fields: The `BirdPublicView` projection has no path to raw personality columns so a future debug panel cannot accidentally leak them.

- Append-only `InteractionEvent` log: The rationale is structural conflict avoidance: the log has "no mutable aggregate fields," so "no last-write-wins" is true because there is nothing to overwrite.

- `consumed_by_tick_id` watermark: The consumed flag makes event folding "idempotent-safe" and gives the tick worker "a natural watermark."

- `PersonalityDelta` audit trail: Deltas make the "measurable in instruments after ~1 week" target queryable and give support/debugging a way to answer why a trait moved without exposing numbers to the user.

- `NotebookEntry` stored verbatim: Entries are generated once and never regenerated on read so historical entries do not change voice if the generator changes later.

- Internal `trigger_kind`: The trigger kind is never exposed; it exists to throttle entry frequency.

- Approximate visit duration in visit log: Duration is bucketed, not precise, "to avoid building a behavioral-tracking surface for visitors."

- Indexing, partitioning, and cold archive: Partitioning by synthetic `account_id` avoids email-derived keys; indexes make tick scans cheap; old consumed events can be archived because the tick has already folded them into deltas and mood state.

### API surface

- REST over HTTPS instead of GraphQL: The plan says access patterns are narrow and a fixed endpoint set is easier to keep "PII/exposure boundaries auditable."

- Snapshot endpoint with no personality fields: The snapshot gives birds, mood, perch zone, call grammar, and aviary state, but "No personality fields," preserving the raw-trait exposure boundary.

- Separate narration endpoint: A screen-reader-only user should not need the visual poll path, so narration can be polled independently at its own cadence.

- Interaction events endpoint returning 202: The client does not receive a synchronous "new mood"; the effect appears after a tick, matching async simulation and the principle that no single session should visibly move state.

- Notebook cursor pagination: Cursor-based pagination supports "indefinite scrollback."

- Visitor-scoped snapshot endpoint: Visitors get the same response shape as the host and the actual canonical state, gated by a visitor token scope.

- No client-submitted absolute state endpoint: The plan's rationale is capability absence: no endpoint accepts mood, personality, or perch position, so there is "no code path to accidentally call."

### Simulation engine design

- Single transaction per aviary tick: The transaction makes crash recovery safe; a mid-tick crash cannot leave an aviary half-updated, and retry re-reads the same unconsumed events.

- Cached client timezone for local time: The server needs local time even when no client is connected, so it stores the most recently seen timezone and lets the aviary advance while everyone is asleep.

- Low-pass drift calibration: The filter is tuned so drift is instrument-measurable after about a week and visible after about three weeks, with constants adjusted through queries rather than "hardcoded guesses."

- Plumage saturation from sustained attention: The plan grounds this directly in sustained attention, combining presence-time and listen-in.

- Mood hysteresis, daily-ish reset, and persistence: Hysteresis avoids flicker, the daily-ish reset prevents yesterday's interaction bias from carrying indefinitely, and stored mood avoids any session-start reinitialization path.

- Call-response propagation: NOT RECOVERABLE FROM PLAN

- Wary-mood spread: The plan's rationale for the implementation is subtlety: the spread can "tip a close call" but will not override a strongly scored alternative.

- Chorus eligibility: The snapshot-level `chorus_active` flag lets the audio engine mix chorus without storing chorus history.

- Rule-based notebook generator instead of live LLM calls: The rationale is lower latency, lower cost, and "fully reviewable/auditable copy before ship" for a load-bearing voice.

- Notebook trigger cooldown and rarity budgets: Cooldowns directly implement the "roughly one entry every few days" target.

- No visit-frequency notebook or narration triggers: The rationale is structural enforcement of "no streak counter, no disguised version of one."

### Frontend rendering pipeline

- 2D canvas instead of WebGL: The plan says the described visual complexity does not need a GPU shader pipeline; 2D canvas keeps the bundle smaller and iteration faster.

- DOM overlays and invisible bird hit-targets: These preserve real focus, ARIA labels, and keyboard behavior while birds remain canvas-rendered.

- Mood-weighted idle motion state machine: The rationale is that behavior should be "recognizably mood-shaped" without a scripted fixed loop.

- Snapshot-to-snapshot interpolation and flight transitions: Interpolation prevents teleporting, and flight animations are distinct from idle motion.

- Mid-motion first frame and quiet-field loading: The renderer must avoid blank/loading frames; before a snapshot arrives, it renders a soft quiet-field state rather than a spinner.

- Reduced-motion render path: Reduced motion swaps continuous animation for discrete pose keyframes and slower color shifts because it is a designed mode, not an animations-off override.

- Top-bar fade as client UI state: The rationale is cosmetic polish without feeding the presence system.

### Audio pipeline

- Procedural call grammar and WebAudio synthesis: The plan rejects recorded audio for bundle-budget reasons and to avoid repeated loops; motif grammar and live synthesis make calls vary.

- Personality-derived call hints instead of raw personality values: The snapshot sends scaled hints such as `call_interval_hint`, preserving the rule that raw personality is never exposed numerically or informally.

- Chorus mixing through independent node graphs: Independent synthesis produces a "real chorus" and avoids fixed-phase cancellation from stacked recorded loops.

- Listen-in gain ramps: Ramping focused and ambient gains over one to two seconds avoids the hard "channel switch" the plan warns against.

- WebAudio fallback without recorded audio: If WebAudio is unavailable, the app proceeds normally, captions default on, and no recorded-audio fallback path exists to violate the "no recorded audio" rule.

- Caption generation from the same grammar as synthesis: The caption always describes what was actually played because it is produced from the same motif-sequence decision.

### Accessibility surfaces

- ARIA live screen-reader narration: The plan uses polite live regions, present-moment naturalist prose, and queue depth capped at one to avoid backlog or flooding.

- Priority narration for user-initiated events: Immediate lightweight narration preserves "prompt-response feel" without making the idle narration cadence faster.

- Account-persisted reduced-motion setting: The rationale is that some users want reduced motion without an OS-wide default, and account persistence lets the setting follow them across devices.

- Contrast tokens: Tokens are used instead of ad hoc colors so text remains AA-compliant across bright and dim aviary states.

- Keyboard navigation patterns: Roving focus, real DOM controls, Enter/Escape behavior, and keyboard-operable offer selection make the canvas scene fully navigable.

### Performance budgets and observability

- Bundle-size CI and code splitting: CI fails regressions past the 2MB budget, while non-critical account, accessibility, and visit flows are split away from the first-paint path.

- Embedded first snapshot: Including or immediately attaching the first snapshot avoids a second network round-trip before the first bird can render.

- Synthetic 60fps checks: Idle performance is enforced in CI/staging on throttled profiles, not just spot-checked manually.

- Memory-growth CI, object pooling, and notebook virtualization: The plan treats "no memory growth over 30 minutes" as a real heap-snapshot test, supported by bounded audio node reuse and releasing scrolled-out notebook DOM.

- Aggregate RUM without account IDs: The metrics layer does not accept `account_id`, preventing accidental high-cardinality or per-account telemetry leaks.

- Simulation tick p99 alarm: A stalled tick worker silently breaks "the aviary continues without the viewer" for all accounts, making it the most important alarm.

- Unsupported-browser surface: Feature detection renders a matter-of-fact unsupported-browser surface instead of attempting an unspecified degraded experience.

### Rollout and risks

- Foundation first: Account Service, data model, and tick skeleton come first to de-risk the "server-as-sole-writer" piece that every other surface depends on.

- Core engine before rendering: Drift, mood, bird-to-bird interaction, and calibration queries are built and tunable before rendering, validated against synthetic event-log fixtures.

- Rendering and audio in parallel after engine: Both are gated on the core engine so snapshot polling, interpolation, and audio behavior have stable simulation inputs.

- Reduced-motion scheduled immediately after full-motion rendering: The rationale is to ship it in v1 with its own design review, "not bolted on afterward."

- Accessibility narration, captions, and keyboard nav before beta: These share snapshot/template infrastructure with the notebook and receive accessibility-focused QA before beta.

- Field notebook and offer/listen-in/settle interactions after stable engine and rendering: They are layered on top of the now-stable engine plus rendering.

- Visit-invitation feature last among in-scope features: Visits are additive and do not gate the core single-user experience, but still ship in v1.

- Dedicated performance hardening pass: Bundle, perf gates, memory testing, and monitoring are informed by real measurements, "not assumed correct from architecture alone."

- Hot-tunable birds-per-aviary threshold table: The team can adjust growth pacing after early-cohort behavior without a code change.

- Day-one instrumentation: Drift calibration, performance RUM, notebook generation rate, and visit funnel metrics exist from the first deployed environment so the team can verify promises rather than infer them.

- Launch gates: Reduced motion, narration, memory, bundle, and calibration gates are "explicit go/no-go gates" because retrofitting them post-launch would contradict the plan's own reasoning.

- Drift coefficient configuration: The mitigation for "screensaver" versus "Tamagotchi" failure is the audit table, calibration queries, and config-based coefficients.

- Sync reconciliation job: A periodic recompute from `PersonalityDelta` history can catch double-application bugs, and this is cheap because deltas are already an audit log.

- Early audio prototype spike: Procedural synthesis quality is the highest-uncertainty piece, so the plan asks for a prototype with real listeners and an explicit fallback design conversation.

- Narration and caption template coverage tests: Template coverage belongs in the engine's test suite so new mood/action/event combinations do not leave accessibility surfaces stale.

- Gamification creep friction: Future gamified surfaces require deliberate schema/API changes; the plan treats that friction as intentional rather than a gap.
