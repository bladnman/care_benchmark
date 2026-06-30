## System-level intent

1. **The server computes meaning, the client computes appearance.** This is stated as the plan's "organizing principle" and repeated through the client/server split: "Personality, mood, perch assignment, weather, and settled-state are server-authoritative facts," while "Pixel position, pose, audio, and narration text" are client-rendered consequences. The snapshot boundary in §2.3 carries the same intent: everything left of the snapshot is server-owned; everything right of it is a pure function of "latest snapshot, wall-clock time, per-entity render seed."

2. **Aliveness must feel continuous, not announced.** The plan makes "No entry animation, ever" and "No announcement surface, ever" non-negotiable implementation rules in §2.4. The first frame is phase-aligned to the tick timestamp so the bird "resumes mid-cycle"; session-start, return, and milestone events are not toasts, banners, or modals.

3. **Architectural absences are part of the product.** The non-goals are not treated as dormant features: "architectural absences, not feature flags." Gamification, Tamagotchi mechanics, and broad social-network surfaces are kept out structurally, with "no kill-switch" for turning gamification on and "no event type, API field, or data-model column" that can become a streak/count surface.

4. **Single-writer truth over client-to-client reconciliation.** The plan frames sync as "a property, not a feature." Personality and mood have one writer, the tick; clients write only event facts. This is why "no last-write-wins" is a core principle rather than a conflict-resolution algorithm.

5. **The product should be boring operationally, but exact about invariants.** The architecture chooses a modular monolith, Postgres event log, edge cache, polling, and no WebSocket. The plan calls this the "boring with teeth" register: ordinary infrastructure, but with strict boundaries around server-authored deltas, event idempotency, and tick transactionality.

6. **Privacy and non-exposure are enforced at serialization and schema boundaries.** Raw personality values are "never serialized to any client payload," email is "stored once" and "never used as a key/identifier," metrics carry no account or bird dimension, and synthetic UUIDs are the "ONLY cross-service identifier." The plan wants friction against later leakage, not just UI restraint.

7. **Accessibility is the same aviary, not a stripped fallback.** Reduced-motion mode is "a second rendering strategy consuming the identical data layer"; narration shares the same structured facts as the notebook; keyboard access uses real focusable elements while keeping the canvas visually chrome-free. The plan explicitly says a "v1.1 accessibility patch is itself a launch failure."

8. **Voice consistency comes from shared facts, not prose mimicry.** The `StateFactExtractor` is the shared source for notebook and narration so a screen-reader user hears "the same product, not two products with different personalities glued together." Captions similarly come from the same `CallSpec` as audio, so text matches what played.

9. **Procedural systems are preferred when they preserve budget, variability, and testability.** Procedural audio avoids recorded assets, rigged SVG parts avoid frame-by-frame sprite sheets, pose curves avoid canned clips, and templates avoid LLM hallucination. The plan repeatedly favors data/grammar/curve systems that are "trivially testable" and fit the bundle and memory budgets.

10. **Calibration and gates are product safeguards, not polish.** Drift pace, audio quality, screen-reader behavior, bundle size, first-bird timing, and 30-minute memory growth are all launch gates or CI checks. The plan names calibration as a "build deliverable, not a deploy-time guess."

## Per-feature whys

### 1. Scope

- **Single-user accounts, one canonical aviary, and multi-device sync**: The plan's rationale is that server-authoritative state lets every device read the same record. Later in §6 it says there is "no client-to-client sync to design" because both devices read snapshots from the single writer.

- **Magic-link auth**: NOT RECOVERABLE FROM PLAN

- **Two starter birds at adoption**: NOT RECOVERABLE FROM PLAN

- **Seven-bird cap**: The plan ties the cap to the "audio-recognizability ceiling" and calls it "a hard constraint, not a rollout lever." It also keeps rendering and resource pools bounded.

- **Age-gated bird unlock schedule**: The plan chooses placeholder thresholds stored as "a single config table" so they can be "retuned without a deploy" and flags them for calibration.

- **A ~6-species pool**: NOT RECOVERABLE FROM PLAN

- **Final mood enum adding `settled`**: The plan adds `settled` because the layout and interactions describe "a distinct settled visual/behavioral state" that "doesn't cleanly collapse into `drowsy`."

- **Personality vector with monotonic-toward-expressive drift**: The rationale is that presence and interactions should produce slow, saturating, instrument-detectable change without a visible single-session jump. Monotonicity is enforced where delta is applied so "a bug in pressure computation can never produce negative drift."

- **Mood as fast-timescale state**: Mood "drives idle-motion and audio params" and "must be consistent across devices," so it is computed server-side and rendered client-side.

- **Procedural call synthesis, chorus mixing, and listen-in mix re-balance**: Procedural audio helps stay inside the bundle budget and avoid recorded-audio fallback. Listen-in is a "re-balance, not a mute," with other birds kept above silence; chorus overlap uses variation to avoid phase-cancellation.

- **Return-greeting**: NOT RECOVERABLE FROM PLAN

- **Offer reaction**: The plan resolves the tension between immediate watching and slow tick authority by separating "behavioral reaction" from "state mutation." The client immediately renders a deterministic reaction; durable drift/mood effects land on the next tick.

- **Settle state**: The plan gives `settled` a distinct state because settled birds have their own visual/behavioral posture and because settle closes the presence window cleanly.

- **5s undo for settle**: NOT RECOVERABLE FROM PLAN

- **Field notebook**: The notebook is server-written because it needs "cross-session history and sparsity control that only the server can see." It is also where product voice is "most concentrated and most visible," so entries are sparse, retrospective, and not generated from client-only state.

- **Day/night cycle**: Day/night is "not simulated or persisted" because it is a pure function of time and timezone hint. Computing it identically in tick and client avoids disagreement while keeping client-local rendering.

- **Ambient weather**: Weather is persisted because it "must be identical across a user's devices"; a phone and laptop should show "the same passing rain."

- **Ambient leaf/feather ornaments**: The plan keeps ornaments client-only because the PRD is explicit that they are "not driven by the simulation tick" and have "no per-leaf state."

- **Visit invitations**: Read-only, revocable, opt-in, off-by-default visits are designed to keep social scope narrow. The `visits` module is isolated from aggregate-stats paths so future leaderboard-like features cannot "just" reuse existing infrastructure.

- **Screen-reader narration, reduced motion, captions, contrast, and keyboard navigation**: These ship in v1 because "a v1.1 accessibility patch is itself a launch failure." The implementation makes them alternate render/access layers over the same aviary facts, not a stripped fallback.

- **Performance budgets**: The budgets exist to protect first-bird aliveness, sustained idle motion, bundle size, and memory stability. The plan maps them to specific levers: edge-inlined snapshots, code-splitting, procedural assets, bounded bird count, simple curves, and reused audio resources.

- **Account export**: NOT RECOVERABLE FROM PLAN

- **Soft-delete with 30-day recovery then hard delete**: NOT RECOVERABLE FROM PLAN

- **Native apps, gamification, Tamagotchi mechanics, and broad social surfaces excluded from v1**: The plan says these are "architectural absences" and later calls gamification a different product. The reason is to prevent reasonable-looking additions like streak counters, visit calendars, visible distress, or public feeds from entering through latent fields or switches.

- **Visit token lifetime**: The plan reads "one-time link" as uniquely generated per invite, not single-use-then-dead, because visit duration tracking and "revoke an active invite" imply "a session-lived, revocable credential."

### 2. Architecture

- **Modular monolith for request-path backend**: The plan rejects microservices because this is "one product with tightly coupled invariants" such as server-only personality writes, honest presence, and email never leaking as an identifier. A modular monolith keeps those invariants enforceable through code review and internal module boundaries.

- **Independently scaled tick worker fleet**: Tick load is "steady background compute" while request traffic is "bursty, diurnal, latency-sensitive." Separate scaling avoids over-provisioning request servers or letting tick latency spike during traffic peaks.

- **Edge layer with inlined current snapshot**: The edge inlines the account snapshot so "the first bird-bearing paint doesn't wait on an origin round trip," making the `<500ms time-to-first-bird` budget reachable.

- **Postgres append-only event log instead of Kafka/Kinesis for v1**: The plan chooses Postgres because it gives transactional consistency with tick cursor advancement "for free," and v1 event volume does not justify a dedicated log system.

- **Per-account monotonic event sequence**: The sequence makes additive event processing ordered per account and lets the tick read rows with `sequence > last_processed_sequence`.

- **Edge snapshot cache**: The cache supports first paint and later polls from the edge, falling through to `aviary-read` only on miss or staleness.

- **Transactional email behind `EmailSender`**: The internal interface makes the provider "swappable without touching call sites."

- **Separate `visits` module**: Visits are isolated from `aviary-read` and aggregate-stats code paths to reduce "social scope creep."

- **Server ownership of personality vector values**: The plan's why is "No-last-write-wins correctness" and the rule that raw personality is "never serialized to any client payload."

- **Server-derived perch zone assignment**: Perch zone is "a user-visible signal the user reads, never controls," so it belongs with mood/personality meaning rather than client appearance.

- **Client ownership of pixel position and pose**: Continuous 60fps motion "can't be tick-cadence-limited (~60s) without looking frozen," so the client tweens and animates from server facts.

- **Client-side WebAudio call synthesis**: The plan says bundle budget and chorus quality both require client-side synthesis.

- **Client-side narration/caption prose**: This avoids a server round trip on every narration cadence tick and keeps voice consistent by sharing the fact-extraction module.

- **Server-written notebook entries**: Notebook entries need cross-session history and sparsity control; only the server has enough context.

- **Presence detection split**: The client owns DOM signals like visibility, focus, and pointer/key activity; the server is the trust boundary for what counts toward drift.

- **Snapshot as render pipeline boundary**: The snapshot boundary lets the renderer ignore why a bird is `wary` and only consume mood, perch zone, and seed. This is what makes reduced-motion mode "a second rendering strategy over the same data."

- **No entry animation**: The plan's reason is aliveness: birds should appear to have been there already, with preen/call cycles resumed mid-cycle.

- **No announcement surface**: The plan prohibits toasts, banners, and modals for session-start, return, or milestone events so the product does not turn natural changes into announcements.

### 3. Data model

- **Server-generated UUIDs as primary keys**: UUIDs are the "ONLY cross-service identifier," protecting the rule that email is not a key or identifier elsewhere.

- **UTC timestamps and timezone hint**: UTC is the persisted truth; client-local time is only for day/night rendering and a last-known timezone hint.

- **Encrypted email stored once**: The plan stores email encrypted and says it is "never used as a key/identifier anywhere else," reducing PII leakage risk.

- **Distinct `Aviary` entity**: The plan keeps it separate from `Account` for "clean ownership of aviary-wide state."

- **Personality vector as jsonb blob**: The tick is the only writer and always rewrites the whole vector; jsonb also keeps species trait-range config and live vector in the same shape for calibration diffing.

- **`InteractionEvent.received_at` as authoritative time**: Server time controls drift math so a client with a fast or slow clock cannot inflate or deflate drift.

- **`NotebookEntry.related_bird_ids` not exposed**: These ids exist for generation-side dedup only, never client exposure.

### 4. API surface

- **Deliberately small, mostly-generic API surface**: The plan wants clients to report "what happened," never "what the new state should be," avoiding state-mutation endpoints that would undermine server-authored deltas.

- **One generic `POST /events` write path**: One endpoint for five event types matches the additive event model and avoids growing per-event-type partial mutations.

- **`AviarySnapshot` as the one payload shape that matters**: The renderer, audio engine, and narration generator all consume this contract, keeping the client side a pure consequence of snapshot facts.

- **Derived rendering parameters instead of raw personality in snapshots**: The plan enforces non-exposure at the API boundary so personality cannot leak through network inspection, and a future stats feature would require a deliberate server decision.

- **Visitor-scoped read-only snapshot endpoint**: The plan keeps visits on a separate auth path so visitor access stays read-only and distinct from signed-in aviary reads.

### 5. Simulation engine design

- **Lease/claim tick scheduling with `SELECT ... FOR UPDATE SKIP LOCKED`**: The scheduler can claim due accounts without contending on the account row and scales horizontally by adding workers.

- **Tick jitter**: Jitter spreads load and avoids thundering-herd ticking of accounts that signed up at the same moment.

- **Per-account tick transaction and edge publish**: Persisting bird/aviary updates, advancing cursors, marking events, and publishing the snapshot in the same transaction-adjacent pass makes "no client mutates personality directly" true end-to-end.

- **Scored-candidate mood model**: A scored model keeps recent interaction, time of day, ambient weather, and personality "composable and independently tunable," unlike a hand-written if/else state machine.

- **Mood hysteresis and dwell floor**: Hysteresis prevents flapping tick-to-tick while still allowing same-session shifts after the dwell floor passes.

- **Monotonic personality drift application**: Applying `max(0, alpha * pressure)` at the final delta step means upstream bugs cannot create negative drift.

- **Drift calibration testbed**: The offline replay testbed tunes pace constants to named targets: detectable movement after about one week, user-noticeable movement after about three weeks, and no single-session visible jump.

- **Bird-to-bird propagation second pass**: Propagation applies on the next scoring pass, not retroactively, to keep causality one-directional and avoid oscillation.

- **Chorus eligibility as client-side call scheduling hints**: Server state does not become chorus state; `energyLevel` only makes overlapping calls more likely while timing/variation remains client-side.

- **Day/night pure function and weather persisted**: Day/night need not be stored because tick and client can compute it identically; weather must be stored so all devices show the same event.

### 6. Sync model

- **Sync as a property, not a feature**: Since the tick is the only writer and clients read snapshots, the plan says the only work is keeping the single-writer property true under concurrent event-log writes.

- **No last-write-wins**: Clients never submit absolute state, so the morning-laptop/lunch-phone race is "structurally unreachable."

- **Clock skew handling**: The plan uses `received_at` for drift-affecting interval math and keeps `client_occurred_at` for debugging/display only, closing a gaming/skew vector.

- **Idempotency by `clientEventId`**: Retries are no-ops on second arrival, preventing double-counting, especially for periodic `presence_ping`.

- **Pull-based snapshot freshness**: Polls happen on visibility change, resume, and low-frequency keepalive. This matches the explicit pull model and keeps the architecture simple while edge cache handles most reads.

- **Immediate interaction rendering with delayed durable mutation**: The client may render an immediate deterministic reaction, but the state mutation remains on the slow authoritative tick cadence.

### 7. Audio pipeline

- **Species motif libraries as data, not code**: This keeps six species inside the bundle budget and lets sound design iterate without an engineering release.

- **Poisson-ish call scheduler with seeded jitter**: Seeded randomness makes output reproducible for testing but not literally identical in practice.

- **Listen-in gain rebalancing**: Focused bird audio ramps up while others remain at a "quieted-but-nonzero ambient floor," enforcing "a re-balance, not a mute."

- **Chorus stagger and per-call variation**: Small random offsets and separate jitter prevent birds from playing the same call at the same phase, avoiding phase-cancellation artifacts.

- **Captions generated from `CallSpec`**: The same object feeds audio and caption text, guaranteeing captions match what was actually played.

- **Bounded reusable audio resources**: A pool sized to the 7-bird cap avoids per-call allocation churn and supports the no-memory-growth budget.

- **Audio failure as caption-only mode**: If `AudioContext` fails or is blocked, scheduling still runs and calls become captions with no recorded-audio fallback, matching "graceful silence with captions on by default."

### 8. Frontend rendering pipeline

- **Single canvas full-redraw renderer**: With the scene bounded at seven birds, full redraw is simpler and cheap enough; dirty-rect bookkeeping would cost bundle size.

- **No general-purpose game engine dependency**: A hand-rolled renderer keeps engine weight out of the bundle and avoids carrying unused physics, tilemaps, and particle systems.

- **SVG rig parts rasterized to an offscreen atlas**: Rigged parts allow continuously parametrized motion and runtime `plumageSaturation` adjustment without pre-baked art per saturation level.

- **Procedural pose system**: Mood-shaped curve generators avoid canned clips and use `render_seed` so birds in the same mood do not move in lockstep.

- **Phase-aligned first frame**: Evaluating functions from `tickTimestamp` removes the "reset to a default pose" tell and implements "appears already in motion."

- **Quiet-field loading state with no spinner**: If the snapshot is late, the client renders a quiet field; under the time target this is rarely perceptible, and the codebase has no spinner.

- **Reduced-motion as a rendering-strategy swap**: The same pose outputs and snapshot data are sampled at a slow cadence and cross-faded, making it "a different rendering of the same aviary."

- **Responsive perch layout**: Percentage positions and a minimum-spacing solver compress spacing rather than cropping at narrow widths.

- **Invisible accessible bird buttons**: Real DOM buttons mirror canvas bird coordinates so keyboard and assistive-tech users get full navigation while the aviary has no visible chrome.

- **Code-splitting**: Critical-path code stays near the internal target, with settings, visits, export/delete, and notebook views lazy-loaded to leave headroom under 2MB.

### 9. Naturalist text generation

- **Shared `StateFactExtractor`**: Notebook and narration consume the same structured observations, enforcing voice consistency through shared input.

- **Sparse notebook grammar**: Noteworthiness scoring plus a 36-72h minimum gap hits "roughly one entry every few days" even for active users.

- **Present-tense narration grammar**: Narration is frequent, live, and single-moment, with no comparative/historical framing, fitting the live aviary rather than the retrospective notebook.

- **Templates instead of a generative model**: The plan avoids LLM dependency because voice must not drift, facts must not hallucinate, entries must stay rare, and narration must run client-side at low latency.

### 10. Accessibility surfaces

- **Screen-reader narration through `aria-live`**: Updates replace, rather than append to, the live region so the assistive-technology queue does not accumulate stale narration.

- **Reduced-motion defaulting from `prefers-reduced-motion` with override**: The rationale is that reduced motion is built into v1 and uses the same data/rendering strategy described in §8.4.

- **Captioning defaults on when WebAudio is unavailable**: This implements graceful silence while still surfacing call events.

- **Keyboard navigation through top bar and bird layer**: Standard button/menu semantics ensure offer and settle are reachable without canvas-only interaction.

- **Contrast token checker**: WCAG AA is enforced in CI for new chrome surfaces rather than left as manual design review.

- **Build-in, not bolt-on accessibility**: The plan makes reduced-motion and captioning launch gates because deferring them would itself be a launch failure.

### 11. Performance budgets and observability

- **Bundle-size CI check from the first bundler commit**: The plan enforces the budget early so the team does not discover overage near launch.

- **30-minute memory soak in CI**: The PRD wanted "a real test in CI, not a guideline," so a scheduled simulated idle session asserts heap does not trend upward.

- **Synthetic first-bird timing fleet**: Common-geography synthetic browsers measure staging/production so the <500ms target is verified in conditions closer to users.

- **Aggregate-only RUM**: The metric schema itself has no account-id field, making privacy an architectural boundary rather than a downstream filtering policy.

- **Simulation-tick p99 alarm**: A p99 over 5s pages on-call to catch degradation before users perceive that the aviary is slow to update.

### 12. Rollout

- **Internal dogfood**: Team accounts catch "obvious aliveness failures" like canned motion and audio uncanniness that automated tests cannot.

- **Calibration pass as blocking gate**: Drift constants, motif libraries, and real assistive-technology behavior are validated before beta/GA.

- **Invite-only beta cohort**: A small cohort exposes real multi-day drift behavior, which the plan says is hardest to validate purely offline.

- **Gradual percentage rollout to GA**: Wider rollout is gated on performance budgets holding under real traffic and the tick-latency alarm staying quiet.

- **Launch gate checklist before GA**: The plan requires first-bird timing, reduced motion, captions, bundle margin, memory soak, and drift calibration before GA, not staged into v1.1.

- **Bird-count ramp**: The 7-bird cap is enforced from day one because it protects audio recognizability; only the age-gated unlock schedule ramps by config.

- **Narrow kill-switches**: Call engine, weather, and notebook generator can be disabled independently for incidents without disabling presence/drift/sync, "the product's actual spine."

- **No gamification kill-switch**: The plan says a streak counter is "a different product" and refuses to build the door for it to slip in.

### 13. Risks

- **Drift calibration risk**: Too fast reads as Tamagotchi; too slow reads as a screensaver. The mitigation is offline calibration plus aggregate-only post-launch drift-curve instrumentation.

- **Sync/tick correctness risk**: A double-apply bug would silently corrupt drift, so cursor advancement and personality writes happen in the same transaction and event ingestion is idempotent.

- **Audio uncanniness risk**: Procedural synthesis can feel "MIDI-ish," so motif libraries are a dedicated sound-design deliverable with a listening-test gate.

- **Notebook/narration repetition risk**: Template text can feel formulaic, so the plan calls for a generous per-fact-type template library, no-repeat-recently logic, and sustained writing investment.

- **Voice-divergence risk**: If the shared extractor is bypassed, notebook and narration voices can drift apart; CI should assert both consumers import the same extractor module.

- **Bundle-budget creep risk**: Procedural audio, rig/pose, and accessibility all compete for the 2MB ceiling, so CI enforces the budget from the first commit.

- **PII-leakage risk**: Engineers may reach for email as a convenient key; a lint rule flags email-field references outside the `accounts` boundary and documents UUID-only as a standing contract.

- **Social-feature scope-creep risk**: Visit notifications, frequent visitor surfaces, and aggregate visit stats are predictable pressures; isolating `visits` makes accidental leaderboard-style reuse harder.

- **Tick scalability risk**: Lease/claim scheduling and Postgres event logging are v1-scale choices, but the claim model is designed to scale horizontally because ordering guarantees are hard to retrofit.

- **Gamification-creep risk**: The mitigation is structural: no event type, API field, telemetry table, or data-model column tracks streaks, visit frequency, or user action counts that could be surfaced as counters.
