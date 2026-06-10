## System-level intent

- **Executable, v1-bounded engineering judgment.** The plan presents itself as an "executable engineering plan" that "makes a defensible call" where decisions are open. This shows up in the repeated **[Call]** markers, the small-team service shape "rather than a microservice fleet," and the final "Open calls made in this plan" list, where most choices are meant to be reversible by config or module.

- **Absence is architectural.** The plan repeatedly treats omitted mechanics as product intent, not missing backlog: "no streaks, achievements, levels, counters," no Tamagotchi decay, no social-network surfaces, and "no user-visible personality numbers in any form." It says "nothing in the schema, API, or telemetry computes or stores cross-account comparative stats," making the absence structural.

- **One canonical aviary, one writer.** The central invariant is "server is the only writer of canonical state." It appears in scope, architecture, data model, API, sync, and risk mitigation: the client is "a renderer and an event reporter," the simulation service is the "sole writer," and last-write-wins on personality is "unreachable."

- **Logical continuity without waste.** The plan wants the user-observable property that "the aviary you return to is the aviary that has been running," while avoiding useless work for dormant aviaries. That intent shows in the "logically continuous" hot/lazy tick, deterministic catch-up, and the equivalence invariant `catchUp(S, E, T) === composeTicks(S, E, T)`.

- **Determinism as trust, debugging, and calibration.** Determinism shows up in the tick, lazy catch-up, replay debugging, property tests, seeded RNG, and calibration harness. The plan says determinism makes "lazy mode safe, replay debugging possible, and the drift calibration harness trustworthy."

- **Events, not values.** Clients submit additive facts like "listened in on Pip for 3 minutes," never values like "boldness = 0.62." This appears in the API, event log, sync contract, and risk section as a way to make client-side corruption and merge UI structurally unnecessary.

- **Slow, monotonic, non-punitive growth.** The bird engine is "monotonic toward expressive"; neglect produces "zero drift, never reversal." The plan ties this to weeks-long pacing, day-7 and day-21 calibration gates, "no visible single-session change," and the lapse case being "quieter, not punished."

- **Qualitative expression, not exposed numbers.** Raw vectors exist for the engine, but traits reach product surfaces only as "already-shaped qualitative parameters," quantized tiers, motion profiles, call envelopes, and naturalist observations. The plan explicitly says the raw vector "never leaves the server outside account export."

- **A living scene, not a UI dashboard.** Across rendering, load states, motion, audio, and top bar behavior, the plan preserves the aviary as a continuous scene: birds start "mid-action," no spinner, no fly-in except first adoption, micro-motion runs continuously, and the top bar fades out of the way.

- **Procedural recognizability.** Audio and motion are meant to feel alive without recorded assets: "zero recorded audio," per-bird `call_seed`, "same bird, with the same voice," and dogfood tests where someone can tell "Pip from Wren by ear, blind."

- **One product voice with two registers.** The plan makes prose a system boundary: `@aviary/prose` owns "every user-facing string," with `naturalist()` and `system()` registers. It frames this as "the same product, not two products glued together."

- **Accessibility as a designed surface.** Accessibility is not a retrofit: the plan names "slow-cadence naturalist screen-reader narration," a "distinct cross-fade rendering register," runtime captions, keyboard navigation, visible focus, and launch-blocking manual screen-reader passes.

- **Privacy as topology and schema, not policy text.** The plan repeats that synthetic UUIDs, encrypted email, aggregate-only telemetry, CI lints, metric allowlists, and physically separate pipelines enforce privacy "as network topology and schema review, not policy text."

- **Performance as part of the conceit.** The first-bird and idle-motion budgets are launch-blocking because latency makes "the conceit" leak. The quiet-field load state exists so cold-path misses degrade "as a designed surface, not a failure."

- **Ambient, opt-in, read-only sharing.** Social is constrained to "per-invite, opt-in, revocable, read-only ambient visits." Visitors generate no presence or interaction signal, no visitor event route exists, and visit logs are scoped to sharing transparency rather than social mechanics.

## Per-feature whys

### Scope

- **Email magic-link sign-in** - NOT RECOVERABLE FROM PLAN

- **Per-device revocable session tokens** - NOT RECOVERABLE FROM PLAN

- **Email change with verification** - NOT RECOVERABLE FROM PLAN

- **Account export as an emailed JSON snapshot** - The plan treats export as "the user's data" and "a data-portability surface." This is also the rationale for the account-export exception to the no-numbers rule.

- **Soft-then-hard account deletion with a 30-day window** - NOT RECOVERABLE FROM PLAN

- **One canonical aviary per account advanced by a server-side tick** - The why is multi-device continuity: "the aviary you return to is the aviary that has been running," and all devices read the same canonical state.

- **Hidden five-trait personality vector** - The vector gives the engine expressive inputs while preserving the product rule that users never see numbers; the plan says traits reach the client only as qualitative motion, call, plumage, and greeting parameters.

- **Presence-time as the dominant drift input** - The plan reads presence as attention: "one human has one attention," and "watching birds without moving is the actual product." Presence drives growth without turning interaction into a meter.

- **Fast-timescale mood persisting across sessions** - The plan wants no reset or snap on return: the snapshot carries whatever the hot or lazy tick computed, so mood continuity is "by construction."

- **Bird-to-bird interaction and chorus emergence** - The plan uses bird-to-bird spread and co-occurring call windows so birds influence each other rather than acting as isolated sprites; chorus emergence feeds audio envelopes and notebook detectors.

- **Stable bird identity** - The why is affective and technical: stable IDs and call seeds preserve "same bird, same voice," while immutable rows and vector checksum canaries protect against the named worst failure of deleting drift.

- **Species pool of about six** - NOT RECOVERABLE FROM PLAN

- **Two system-selected starter birds** - The plan chooses complementary call registers and palettes so the starters are distinct, then presents them as "the birds that arrived" rather than as a user-optimized selection.

- **Age-gated growth to a cap of seven birds** - Growth is age-based so it is not gamified, and the cap protects recognizability; the risk section says the cap stays at 7 unless the recognizability test says otherwise.

- **User naming and renaming** - NOT RECOVERABLE FROM PLAN

- **Return-greeting** - The greeting is the "anchor moment" and is computed server-side so it is canonical and consistent across refresh, while still shaped by absence length, boldness, and mood.

- **Listen-in** - Listen-in "re-balances, never mutes" so focus is attentive rather than isolating. It makes one bird more audible while keeping the others as an ambient bed.

- **Offer interaction** - Offers are quiet, cooldown-protected inputs. The cooldown and synchronous outline response support pacing: the client can begin an approach animation, but the canonical mood consequence lands at the next tick.

- **Settle interaction** - Settle is an "opt-in soft session end." The visual effect is session-scoped because remote-settling another open device "would be surprising," while the canonical engine effect stays identical.

- **Three-condition presence accounting** - The conjunction of visible, focused, and recent input prevents tab-open or multi-device inflation. The plan calls presence the dominant drift input, so it receives clamps and audited sampling.

- **Read-only field notebook** - The notebook exists to record "observations of the aviary, never of the user's behavior." Sparse output and read-only grants keep it from becoming a journal, metric, or editable reward log.

- **Single horizontal one-screen scene with no in-scene UI chrome** - The scene is meant to stay an aviary rather than a dashboard. The top bar is outside the scene, sparse, and fading; birds must remain the first-order surface.

- **Load-with-motion-in-progress and quiet-field loading state** - The why is continuity: no entry animation, no spinner, no progress element. If state is late, quiet field covers the gap without making the moment feel like app loading.

- **Responsive layout that never crops a bird** - The plan wants the actual birds to remain inspectable on every viewport; the slot solver compresses spacing first and guarantees containment.

- **Fully procedural WebAudio with no recorded audio** - Procedural synthesis avoids sample-identical repetition and phase artifacts, supports per-bird recognizable signatures, and keeps the no-recorded-audio rule unconditional.

- **Per-bird call signatures and real chorus mixing** - The plan says recognizability is the "affective spine." `call_seed` fixes identity while mood and drift modulate rate, ornamentation, and intensity.

- **Graceful silence fallback with captions on by default** - If audio is unavailable, the aviary still runs and the scheduler still produces caption timing; captions reflect real calling behavior rather than canned fallback strings.

- **Screen-reader narration, reduced motion, captions, keyboard navigation, focus, and contrast** - The plan treats these as v1 surfaces because accessibility is "designed," not checklist work added after the fact.

- **Per-invite ambient visits** - Visits are opt-in, revocable, read-only, and generate no presence or interaction signal. This keeps sharing ambient and avoids social-network mechanics.

- **Visit notifications off by default with per-account opt-in** - NOT RECOVERABLE FROM PLAN

- **Synthetic UUIDs, encrypted email, and telemetry separation** - The why is privacy: email lives in one encrypted field, every other durable identifier is synthetic, and aggregate metrics have no account-ID dimension.

- **Performance budgets** - The budgets protect the feeling that the aviary is alive: first bird visible fast, 60fps idle motion, and zero memory growth over a 30-minute session.

- **Unsupported-browser surface** - NOT RECOVERABLE FROM PLAN

### Architecture

- **Three deployable server components plus the web client** - The plan says this shape is "sized for a small team rather than a microservice fleet," while still separating auth/API, simulation, and mail responsibilities.

- **API service without simulation logic** - The API terminates auth, serves snapshots, and appends events, but "never computes drift"; this supports the single-writer canonicality rule.

- **Simulation service as tick runner and sole writer** - The simulation service owns drift, moods, weather, timers, notebook candidates, and fresh snapshots so canonical state changes in exactly one place.

- **Mail worker and Postmark call** - Postmark is chosen for "deliverability quality on low-volume transactional mail," with a `Mailer` interface so the provider can be swapped.

- **PostgreSQL as source of truth and Redis for short-lived coordination** - The rationale is durability and simplicity: accounts, birds, event log, notebook, and invites live in Postgres; "nothing durable lives in Redis."

- **Cloudflare Workers plus KV edge snapshot cache** - The plan chooses this because it has "the simplest write-through API for our shape" and helps the first-snapshot path.

- **Client/server split** - The split is the core invariant: a bird's "being" is server state, while a bird's "appearance this frame" is client interpolation.

- **Snapshot as render pipeline boundary** - Snapshot separates deterministic server state from ephemeral client realization: drift, mood, and weather are server-side; positions, animation curves, audio realization, and ambient ornaments are client-side.

- **Hot mode plus lazy catch-up ticking** - The plan avoids ticking every dormant aviary forever because it is "wasteful and adds nothing observable," while preserving the running-aviary property exactly.

- **Deterministic tick and catch-up equivalence test** - Determinism makes lazy ticking safe, enables replay debugging, and makes the calibration harness trustworthy.

- **Application-layer encrypted email plus CI lint** - The rationale is to keep email out of every table, log, queue, metric, and cache key except sanctioned account/invite fields.

- **Physically separate telemetry pipeline** - The why is hard separation between aggregate operational telemetry and per-account simulation state; metrics infrastructure has no simulation database credentials.

### Data model and API surface

- **Email fingerprint for lookup** - The HMAC fingerprint supports lookup-by-email "without decryption."

- **Magic-link token consumption with atomic compare-and-set** - The plan says this makes replay "impossible."

- **Raw personality vectors in account export** - The plan reads export as data portability, not a UI surface; it also flags this for product confirmation before GA.

- **Append-only interaction event log** - The event log gives a server-assigned total order, idempotent retries, and replayable history for deterministic debugging.

- **Server timestamps as authoritative for engine math** - Client timestamps are diagnostics only, preventing client clock drift or manipulation from changing simulation outcomes.

- **Presence rollups and raw ping retention** - Rollups give drift the input it needs while raw pings older than seven days are deleted; the notebook cannot read presence totals.

- **Four-minute activity window** - The plan leans long because "watching birds without moving is the actual product," while keeping the constant configurable for calibration.

- **No notebook UPDATE/DELETE grants for the API role** - The rationale is the "read-only contract."

- **No streak, visit-count, achievement, or behavioral aggregate tables** - The plan makes anti-gamification architectural and prevents notebook or telemetry surfaces from becoming disguised counters.

- **Magic-link request returns 202 always** - The rationale is "no account-existence oracle."

- **Snapshot returns qualitative render-ready state** - The snapshot omits personality numbers so the no-numbers rule holds even against a network-tab reader.

- **Server-side greeting block in snapshot** - Greeting selection is canonical and consistent across refresh; the client handles timing and animation.

- **Polling instead of WebSockets** - The plan says the aviary changes on a roughly minute cadence, so sockets "buy nothing the product can feel" and add operational surface.

- **Event write endpoint with idempotency keys** - Idempotency makes retries safe over flaky mobile networks while preserving append-only event semantics.

- **Offer acknowledgement before next tick** - The client can start the approach animation promptly, but canonical mood consequences remain in the tick.

- **Visit snapshot endpoint with no event writes** - Visitor presence and interactions are "unrecordable by construction."

### Simulation engine

- **Pure deterministic `@aviary/engine` tick module** - Sharing the module between simulation service and calibration harness keeps authoritative execution and calibration aligned.

- **Drift formula with `(1 - x)` saturation** - Natural saturation lets long-tenured birds keep drifting ever more slowly instead of clipping, and with no negative term neglect cannot reverse progress.

- **Presence, listen-in, offers, and settle drift weights** - The plan uses presence as the dominant input, listen-in to warm and vocalize the focused bird, offers to add curiosity or boldness, and settle as mood-only so ending a session is not growth fuel.

- **Diminishing returns within a session** - A long session matters more than a short one but not linearly, protecting weeks-long pacing against outliers.

- **Day-7 and day-21 calibration gates** - The plan makes felt pacing executable: measurable drift by day 7, at least one expressed tier by day 21 plus or minus 4, and no visible single-session change.

- **Quantized plumage richness tiers** - Visual change arrives in "rare, look-back-noticeable steps" instead of per-session gradients.

- **Weighted mood scoring rather than hard FSM** - The same stimulus can yield different moods across birds because time of day, events, weather, personality, and inertia all contribute.

- **Mood inertia** - Stickiness keeps moods from flapping tick-to-tick and makes changes happen on a believable timescale.

- **Canonical call envelopes with client-side realization** - The server defines what kind of calling is happening, while individual call realization can differ per device as cosmetic variation.

- **Notebook detectors and sparsity controller** - Detectors observe bird and aviary facts; the token bucket keeps entries around one every two to four days so active accounts stay sparse.

- **Notebook privacy type boundary** - The generator's input type contains only bird and aviary observations, making "observations of the aviary, never of the user's behavior" a compile-time property.

- **Age-only adoption schedule** - The next bird is gated by aviary age rather than user behavior, keeping growth out of badges, streaks, or earned reward loops.

- **Immutable bird IDs and vector checksum canaries** - The plan uses immutability and deploy-time checks to prevent migrations or bugs from rewriting identity or accumulated drift.

### Sync model

- **Single canonical record** - Multi-device sync is just reading the same record; there is "no merge, no reconciliation, no client-resident authoritative state."

- **Additive, server-authored deltas** - Clients submit events, not trait values, making personality last-write-wins impossible.

- **Idempotent, replayable writes** - Unique `(account_id, idempotency_key)` makes retries safe and supports historical reproduction.

- **Concurrent sessions as normal** - Laptop and phone can both append events; the presence clamp collapses attention and the tick serializes effects.

- **Session-scoped settle visuals** - Settle affects the local device because changing another open screen would be surprising; canonical mood input is still logged.

- **Auth-level-only conflict surfaces** - There is no data-merge UI because the model has no data to merge.

- **Snapshot staleness bounds and suspend/resume pulls** - The polling cadence stays within the product's natural pace, while frame-gap detection handles tab restore and suspension.

### Frontend rendering pipeline

- **WebGL2 with a thin bespoke layer** - The plan rejects a general engine because the scene is one screen, up to seven birds, and a few depth layers; a general engine costs bundle budget and "buys nothing."

- **Preact DOM chrome** - Preact is small and gives accessibility semantics "where they belong" for top bar, settings, notebook, and dialogs.

- **Boot chunk and lazy chunks** - The boot chunk contains only what is needed to put a mid-action bird on screen; audio, notebook, settings, invite, export, and some registers load later to protect first paint.

- **Layered single-viewport scene** - The scene uses sky, foliage, perch planes, birds, and foreground passes to create depth while keeping birds contained and visible.

- **Client local-time day/night palette** - Day and night evolve continuously from local time, independent of snapshot cadence.

- **Procedural bird pose graph** - Parametric rigs and mood-shaped idle scheduling make birds feel alive without sprite-sheet bloat.

- **Continuous micro-motion** - Breathing, feather settle, and pose motion ensure stillness "never reads as paused."

- **Interpolation through plausible actions** - Perch changes play as hops or short flights, never teleports, preserving scene continuity between snapshots.

- **First frame from inlined or preloaded snapshot** - Birds begin mid-action and audio joins later, so the aviary does not feel like it starts when the app starts.

- **Reduced-motion render register** - The plan calls it "parallel" and "not degraded," replacing animation with slow cross-fades and its own QA pass.

- **Fading top bar that respects focus** - The bar gets out of the scene during quiet watching, but focus-visible suppresses fade for accessibility.

- **Bird hit targets as DOM overlays** - Focus, ARIA, keyboard, touch, and pointer handling share one system with target sizes at least 44px.

- **Offer tray without countdown timer** - Cooldown is shown as quiet unavailability because a timer would be "a meter."

### Audio pipeline

- **Parametric motif libraries** - Compact pitch, duration, timbre, and ornament data support procedural calls without recorded assets.

- **AudioWorklet voice pool** - A pre-allocated reused voice pool satisfies the no-memory-growth budget and avoids render-thread GC pressure.

- **Socially coupled audio scheduler** - High chorus-affinity birds answer one another in a beat-window, producing call-and-response and true chorus from independently synthesized voices.

- **Listen-in gain automation** - Smooth gain ramps prevent hard cuts and keep non-focused birds above silence.

- **Settle and hidden-tab fade-outs** - Settle quiets the mix into night; hidden-tab fading respects that the user has departed and presence has ended.

- **Captions generated from realized calls** - Captions describe what the synthesizer actually built, so accessibility tracks the live procedural sound.

- **No recorded-audio fallback** - The fallback remains silence plus captions, preserving the zero-recorded-audio rule unconditionally.

- **No "click to enable sound" announcement** - Before a required user gesture, the product behaves like the silence path instead of demanding attention.

### Accessibility surfaces

- **Shared prose system** - Central ownership of strings enforces the naturalist/matter-of-fact split by imports and register types, not by review vigilance.

- **Screen-reader narration cadence** - Idle narration every 30 to 60 seconds, with dropping rather than backlog, keeps narration observational and slow.

- **Assertive return-greeting narration** - The plan makes return-greeting the only assertive event because it is "the anchor moment."

- **Per-bird accessible names with no mood labels and no numbers** - Accessible names use the same descriptors as the visual/narration systems while preserving the hidden-trait rule.

- **Keyboard navigation inside the scene** - Spatial arrow navigation, Enter for listen-in, Escape to exit, and Tab out make the scene operable without pointer assumptions.

- **Contrast and caption checks** - Design tokens and automated checks enforce WCAG AA across day and night palettes.

- **Voice-rule lints and manual screen-reader passes** - Banned words, no "you," no exclamations, and manual VoiceOver/NVDA checks keep accessibility and voice from drifting under deadline pressure.

### Performance, observability, and rollout

- **CI-enforced budgets** - Size, first-bird time, frame rate, memory, and tick latency are launch-blocking so performance remains part of the product surface.

- **Edge first-snapshot path** - Edge KV and preloaded boot chunks support the sub-500ms first-bird goal for hot accounts.

- **Quiet-field cold fallback** - When edge misses or lazy catch-up delays the snapshot, quiet field degrades gracefully as a designed surface.

- **Synthetic monitoring fleet** - Dedicated synthetic accounts measure load, frame timing, audio init, snapshot latency, and full auth without using real per-account state.

- **Aggregate-only RUM and server metrics** - Metrics provide operational health while excluding account IDs, per-bird state, and per-account interaction history.

- **Deliberately unmeasured engagement and retention mechanics** - The plan does not measure product-mechanic funnels because the metrics philosophy gives "nothing to A/B against."

- **Milestone build order** - Foundations and privacy lints come first, canonical vertical slice validates the core, audio and engine depth follow, and accessibility starts before hardening rather than at the end.

- **Drift calibration starting at M2** - Three-week drift targets require real wall-clock time, so calibration begins as soon as engine depth exists.

- **Config-driven coefficients** - Drift weights, activity window, cooldowns, mood scores, and notebook bucket rates can be recalibrated without client release or schema change.

- **Third-bird offers disabled at GA** - The plan waits for roughly 90-day-old beta aviaries to validate the offer flow before enabling growth in production.

- **Feature flags for operational fallback, not product experimentation** - Flags can disable edge KV, audio, or visits, but there is no A/B framework because experimentation conflicts with the product's metrics philosophy.

- **Support surfaces at GA** - Unsupported browser page, status link, and plain-text privacy policy help failures stay matter-of-fact and transparent.

### Risk-shaped implementation features

- **Drift harness and beta diaries** - The plan treats drift feel as the main risk and uses executable gates, beta observation, config coefficients, and diaries as the instrument of record.

- **Single-writer tick, append-only log, replay, and vector canaries** - These protect against "sync/canonicality bugs that delete drift," the named worst failure.

- **Blind audio recognizability tests** - A dogfooder "which bird called?" target of at least 80 percent supports the rationale for call signatures and the seven-bird cap.

- **Accessibility work in M3** - Placing prose, narration, captions, reduced motion, and keyboard nav before hardening keeps accessibility on the critical path.

- **Presence sampler tests and clamps** - The plan sees corrupted presence as population-wide drift inflation, so the sampler is audited and server-clamped.

- **Tick fleet alarms and lazy catch-up** - Lazy mode bounds hot-set cost, while overdue-aviary and scheduler-lag alarms guard against stalled aviaries.

- **Deliverability monitoring and resend affordance** - Since magic link is the only door, synthetic auth round trips and mail-domain setup mitigate lockout.

- **Calibration-consented accounts as the only per-bird dashboard exception** - The flag checked at the query layer gives calibration one sanctioned exception without eroding the broader privacy boundary.

- **Banned-pattern lints and PR-review checklist** - These prevent "Welcome back!", counters, toasts, and other announcement-register strings from changing the product voice.
