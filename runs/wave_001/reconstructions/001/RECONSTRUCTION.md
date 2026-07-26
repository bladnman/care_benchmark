## System-level intent

1. **Affective requirements should become engineering mechanisms.** In "How to read this plan," the plan says the PRD is "unusually opinionated about feel" and the plan's job is to "convert those into structures that make the wrong thing hard to build." This shows up again in "Anti-drift guardrails," where product integrity is encoded as CI instead of left as "a code review habit."

2. **Feels alive, not robotic.** The plan carries this through the canonical server tick, deterministic catch-up, "no load-state code path," interpolation rather than teleporting, idle micro-motion floors, procedural call variation, and the aliveness harness. The plan repeatedly treats "reads as paused" as a bug and makes first-bird timing "an affective metric, not a perf metric."

3. **Notice, never announce.** The plan enforces this with no toast/banner/notification component, no textual welcome, polite rather than assertive live regions, pull-only visit logs with "no badge," quiet newcomer offers, and error surfaces that only appear after sustained failure. The recurring vocabulary is "no in-aviary error surface," "not a notification," "no badge," and "never announce."

4. **Personality should be felt by watching the bird, not exposed as data.** The plan strips trait values at the service boundary, projects only "qualitative, quantized fields," bans numeric trait surfacing, keeps `social_warmth` visible only through behavior, and says coarse banding "forces the client to express personality through behaviour rather than through a parameterized visual dial."

5. **No gamification, no engagement economy.** The plan makes "no `visit_count`, no `session_count`, no `streak`" load-bearing, bans counter-like schema additions, keeps notebook subjects from being the user, refuses "north-star engagement metric," and treats the "one screen" test as a tiebreaker: "does this make the user look at a bird, or at the product?"

6. **Monotonic drift means no Tamagotchi punishment.** Drift deltas are clamped non-negative at one write site, lapsed users get "no trait decrease," `settle` contributes nothing to drift, and pending deletion stops tick scheduling rather than creating decay. The plan explicitly calls this the "mechanical implementation of the no-Tamagotchi rule."

7. **One canonical record, server-owned bird reality.** The plan states the invariant: "The server owns what the bird is. The client owns what the bird looks like right now." This appears in the single Postgres store, event-only writes, server-only trait updates, one canonical aviary per account, and sync model where "there is nothing to sync, nothing to merge."

8. **Determinism is the way the aviary keeps running.** Hot/cold tick scheduling, catch-up, counter-based PRNG, pure step functions, advisory locks, replay property tests, and production canaries all protect the promise that "the aviary the user returns to must be the aviary that ran."

9. **Accessibility is the product, not a fallback.** The plan says "Accessibility ships with, not after" and asks in launch verification whether the aviary "felt alive" to screen-reader and reduced-motion users. Narration, captions, DOM mirror, and reduced-motion all consume the same scene model so the result is "same product, not two products glued together."

10. **Privacy boundaries should be structural.** The plan uses encrypted email, synthetic UUIDs, retention limits, no cross-account aggregation, a separate telemetry store, no analytics warehouse, and closed metric dimensions. It repeats that "a policy alone is not a boundary."

11. **Audio must be procedural, recognizable, and identity-preserving.** The plan bans audio files, fixes timbre identity from `voice_seed`, forbids mood/drift from modulating timbre or motif membership, and says the recognizability invariant is the mechanism behind knowing "Pip's call by ear."

12. **Small surface area protects the product.** The architecture is "small on purpose"; the plan rejects a "microservice-per-noun decomposition," panning/zooming, social-network surfaces, extra top-bar anchors, and product surfaces that compete with the aviary.

13. **System surfaces speak matter-of-factly; naturalist surfaces observe.** Error bodies, account settings, sign-in, and unsupported-browser surfaces use "matter-of-fact voice"; notebook, narration, and captions use lowercase, present tense, no second person, and observation rather than state-transition language.

14. **Long-lived product integrity needs guardrails, not memory.** The plan says refusals "erode by increments" and treats banned components, banned strings, schema guards, contract tests, and PR template questions as the form of refusal that "survives a year of staffing changes."

## Per-feature whys

### Scope and product boundary

- **Email + magic-link sign-in:** NOT RECOVERABLE FROM PLAN
- **Always-202 auth link request:** Why: the endpoint returns `202` regardless of account existence for "no account enumeration."
- **15-minute magic-link expiry:** NOT RECOVERABLE FROM PLAN
- **Single-use magic-link consumption:** Why: the plan pairs consumed links with `410` expired/consumed handling so replayed links do not sign users in again.
- **Per-email rate limiting for auth links and invites:** Why: the plan names per-email limits as part of preventing abuse and enumeration around link issuance.
- **Per-device session tokens listed and revocable from account settings:** NOT RECOVERABLE FROM PLAN
- **Revoking the current session signs out:** Why: session revocation is meant to be real account control, not only list cleanup.
- **Email change with new-address verification before commit:** NOT RECOVERABLE FROM PLAN
- **Old email remains valid until verification:** Why: the plan preserves the current address until the new one is verified, avoiding an uncommitted identity change.
- **Synthetic account UUID as the only identifier outside the account record:** Why: PII handling says every other reference uses the synthetic UUID so logs, queues, cache keys, metrics, and errors do not carry email.
- **JSON account export on demand via emailed download link:** NOT RECOVERABLE FROM PLAN
- **Soft deletion for 30 days with in-product recovery, then hard deletion:** Why: deletion has a recovery window, stops tick scheduling immediately, and then removes account-linked data while backups age out within the promised cycle.
- **One canonical aviary per account:** Why: the "one canonical record" model eliminates sync surfaces and conflict classes.
- **Two starter birds at adoption:** NOT RECOVERABLE FROM PLAN
- **Hard cap of 7 birds:** Why: the cap rests on call-signature recognizability and audio-mix clarity; before enabling the 5th bird, the plan requires listener evidence.
- **Species pool of 6:** NOT RECOVERABLE FROM PLAN
- **One nightjar-like species active at night:** Why: "Night is not a dead state"; the species ignores night vocal-damping and has an alert night prior.
- **Species silhouettes, plumage palettes, and motif libraries:** Why: the rendering and call grammar need species-specific visual and audio identity.
- **Hidden 5-trait personality vector per bird:** Why: personality is persisted server-side and used to shape behavior while remaining "server-write-only" and never serialized to the client.
- **Monotonic-toward-expressive drift driven primarily by presence-time:** Why: this implements growth without decay or punishment; the user notices a bird changed, not that a meter advanced.
- **5-state mood machine with daily-ish reset and cross-session persistence:** Why: moods provide fast-timescale behavioral variety while persisting across sessions and avoiding any "session start" reset path.
- **Server-side simulation tick at ~60s:** Why: the server owns bird reality, keeps devices coherent, and gives the aviary observable continuity independent of client connections.
- **Bird-to-bird call-and-response, wary contagion, and emergent chorus:** Why: interaction makes birds affect one another without scripting a show; chorus is "emergent, not scheduled."
- **New-species offers gated on aviary age only:** Why: age-only pacing avoids engagement gating; it stays outside gamification and interaction-count incentives.
- **Stable internal bird IDs:** Why: a bird is "never replaced, regenerated, reseeded, or migrated into a different row identity"; identity and personality survive names, species, and migrations.
- **Return-greeting:** Why: the plan calls greeting "the anchor moment" and gives it a dedicated subsystem so returns feel personal without using textual welcome.
- **Exactly one bird greets first:** Why: the first 1-2 seconds should have one focal bird; additional response is staggered so birds never greet in unison.
- **A wary or low-boldness bird may not greet:** Why: "Silence is a valid greeting outcome"; a guaranteed greeting would become "a canned greeting."
- **Listen-in by click, tap, or keyboard focus:** Why: listen-in is an act of attention at a device, and the mix should rebalance smoothly around that focus.
- **Other birds attenuate but never mute during listen-in:** Why: the aviary remains present; the change is "a listening change rather than a channel switch."
- **Offer reachable only from the top bar:** Why: keeping offers in the top bar preserves the no-UI-chrome-inside-the-scene rule.
- **Seed, song fragment, still pool offer kinds:** NOT RECOVERABLE FROM PLAN
- **Per-bird offer cooldown of a few minutes:** NOT RECOVERABLE FROM PLAN
- **Settle top-bar gesture:** Why: settle quiets mood and ends presence cleanly while contributing nothing to drift.
- **5-second any-click undo for settle:** NOT RECOVERABLE FROM PLAN
- **Field notebook auto-generated, read-only, sparse, and scrollable backward indefinitely:** Why: it records observations in naturalist voice without becoming a feed, editing surface, or user-behavior mirror.
- **Presence accounting under the strict three-condition conjunction:** Why: watching birds without moving is the product, but unattended focused tabs should stop crediting; server caps also prevent double drift.
- **Single non-panning, non-scrolling, non-zooming horizontal scene:** Why: the product is one screen, so the user looks at the birds rather than operating a camera.
- **Three perch zones system-assigned from mood and personality:** Why: perch is a signal the user reads, never a control.
- **Local-timezone day/night cycle:** Why: the aviary reflects local time without collecting location; timezone is client-reported, persisted, and "never inferred from IP."
- **Rare non-assertive ambient weather:** Why: weather adds short-lived mood effects and ambient variation without becoming a user-facing affordance.
- **Ambient micro-motion:** Why: continuous idle motion prevents birds from reading as paused and supports "feels alive, not robotic."
- **No UI chrome inside the scene:** Why: scene controls would make the user look at the product instead of the bird.
- **Thin top bar with exactly four affordances:** Why: the bar stays sparse and keeps the visual anchors limited.
- **Top bar fades on cursor stillness:** Why: controls recede while the aviary remains the focus, while keyboard focus keeps controls visible.
- **Per-invite, email-addressed, revocable visit invitations off by default:** Why: social is optional and bounded; invitations are specific, revocable, and not a discovery or profile surface.
- **30-day unused invitation expiry:** NOT RECOVERABLE FROM PLAN
- **Read-only ambient visitor view:** Why: visitors cannot interact, have no representation, and create no co-presence, preserving the host's aviary as not-social-network.
- **Visitor activity contributes nothing to host drift:** Why: visitor tokens lack `events:write`, structurally enforcing that visits do not change the host's birds.
- **Host-side visit log in account settings, pull-only, no badge:** Why: the host can inspect visits without any badge, unread state, or announcement loop.
- **Optional visit-notification toggle off by default and not in onboarding:** Why: the plan keeps visit notification outside the default product and away from onboarding.
- **Running naturalist screen-reader narration:** Why: narration is generated from the same scene state as visuals so accessibility users get the same living aviary.
- **Reduced-motion as designed cross-fade rendering:** Why: reduced motion is not a stripped fallback; it has its own "slow, quiet aesthetic."
- **Runtime-generated call captions:** Why: captions match what actually played because they come from the same call AST as synthesis.
- **WCAG AA contrast and full keyboard navigation:** Why: accessibility ships with the product, with visible focus against every aviary state.
- **Performance budgets:** Why: first-bird timing, JS size, frame rate, and memory growth are treated as affective product requirements, not only technical metrics.
- **Procedural WebAudio with graceful-silence-plus-captions fallback:** Why: recorded audio is never a contingency, and captions carry meaning when audio is unavailable.
- **Last two major versions of Chrome, Safari, Firefox, Edge:** NOT RECOVERABLE FROM PLAN
- **Matter-of-fact unsupported-browser surface:** Why: system surfaces use matter-of-fact voice rather than naturalist phrasing or in-aviary announcement.
- **Native apps out of scope with no protocol accommodation:** Why: avoiding accommodation keeps v1's data model and protocol small.
- **No gamification features:** Why: achievements, streaks, scores, badges, and similar features would move attention from bird observation to product progress.
- **No Tamagotchi mechanics:** Why: death, hunger, distress, decaying happiness, and negative drift on neglect contradict monotonic expressive drift.
- **No social-network surfaces:** Why: profiles, feeds, follows, comments, avatars, leaderboards, and show-off rendering would turn ambient visits into a network.
- **No push notifications or marketing email about the aviary:** Why: notification and marketing surfaces would announce rather than invite noticing.
- **No customizable scenes, panning, scrolling, zooming, or user-controlled perches:** Why: the aviary remains one non-camera scene where perch placement is bird behavior, not user control.
- **The "one screen" scope test:** Why: it is the tiebreaker for whether a feature makes the user "look at a bird, or at the product."

### Architecture

- **Five services, one client, one shared TypeScript package:** Why: the product is small, and microservice-per-noun decomposition would add sync surfaces the one canonical record model exists to eliminate.
- **TypeScript end to end:** Why: sharing the simulation pure-function core as code keeps client-predicted motion and server-canonical motion from diverging visibly.
- **Node 22 LTS, strict mode, no `any` in the sim engine:** Why: the simulation core is correctness-sensitive and shared across server and client.
- **PostgreSQL 16 as the single canonical store:** Why: the product is "small-row, high-fanout-read, low-write" and does not justify a second database.
- **Redis as cache and queue only:** Why: if Redis is lost, the product degrades to slower snapshot reads and nothing else.
- **Server owns bird reality; client owns current appearance:** Why: anything longer than the snapshot interval must agree across phone and laptop, while sub-second motion can stay local.
- **Scene model boundary between DTO and renderers:** Why: visitor view, reduced-motion view, narration, captions, audio, and canvas become consumers of one state instead of parallel implementations.
- **Pure consumers of `(sceneModel, t)`:** Why: preventing consumers from mutating the scene model keeps accessibility surfaces from drifting out of sync with visuals over time.
- **No WebSocket at v1:** Why: a 60-second tick and kilobyte snapshots do not need sub-second freshness, and sockets would add a stateful tier, reconnection logic, and another delivery path.
- **HTTP polling with ETag and jitter:** Why: it matches the tick cadence and makes steady-state polls a near-zero-body `304`.
- **Revisit trigger for sockets only with co-presence or tick below ~10s:** Why: those are the cases that would create actual need for sub-second freshness.

### Domain model and data model

- **UUIDv7 identifiers and no natural keys:** Why: time-sortable UUIDs are index-friendly, and synthetic identifiers avoid natural-key coupling.
- **Encrypted account email plus HMAC lookup hash:** Why: sign-in lookup does not require plaintext email, and the plaintext boundary stays narrow.
- **Pending email change row:** NOT RECOVERABLE FROM PLAN
- **Session device label:** Why: settings can show coarse device context such as "Safari on iPhone" for revocation.
- **Aviary `created_at` as sole input to new-bird pacing:** Why: newcomer availability remains age-only.
- **Aviary timezone stored server-side:** Why: day phase is local-time based without collecting location or inferring IP.
- **Aviary `simulated_through` and `tick_seq`:** Why: they define the canonical simulation clock and deterministic PRNG stream index.
- **Aviary `rng_seed`:** Why: deterministic replay and catch-up depend on per-aviary seeded randomness.
- **Bird `voice_seed` fixed at adoption:** Why: timbre identity "never drifts," preserving recognizability.
- **Bird `last_greeted_at`:** Why: return-greeting weighting rotates greeters gently rather than repeating the same bird every session.
- **Interaction events as append-only immutable idempotency records:** Why: clients may write only events; ordered log consumption allows deterministic folding into server state.
- **Client timestamps advisory only:** Why: server timestamps are authoritative for ordering and crediting.
- **Presence windows derived server-side:** Why: drift input comes from hardened presence accounting rather than trusting raw client pings.
- **Notebook entries immutable and kept forever:** Why: the notebook must "scroll back indefinitely."
- **Notebook `template_id` hidden from users:** Why: it supports voice QA and dedupe without surfacing machinery.
- **Invitation and visit rows:** Why: invitations are revocable and visits are records of redeemed links, while visits contribute nothing to simulation.
- **Account settings for reduced motion, captions, audio, visit notifications, narration verbosity:** Why: these are account-level accessibility and social defaults, with visit notifications off by default.
- **No visit, session, streak, days-active, or score columns:** Why: their absence is "load-bearing"; without the column, future gamification is not a two-hour ticket.
- **Interaction-event 90-day retention:** Why: after the tick folds events into canonical state, longer retention creates behavioral history with no product use and privacy cost.
- **Presence-window 180-day retention:** Why: it is long enough to re-derive drift if a calibration bug needs replay, then dropped.
- **Visit retention at 12 months:** NOT RECOVERABLE FROM PLAN
- **Trait-exposure boundary through `renderBird()`:** Why: bird rows are never selected directly into DTOs, keeping trait numbers physically absent from responses.
- **Repository lint forbidding `Bird` row import into API DTO module:** Why: the boundary is enforced mechanically, not by convention.
- **Contract test with extreme vs mid-range traits:** Why: snapshot JSON must not reveal trait values except through permitted render projections.

### Simulation engine

- **Hot/cold tick scheduling with deterministic catch-up:** Why: live ticking all accounts is wasteful, but observable semantics require the returning user to see an aviary that has been running.
- **Tick determinism invariant:** Why: live minute-by-minute stepping and catch-up replay must produce byte-identical state for multi-device coherence and continuity.
- **Counter-based PRNG instead of wall-clock or `Math.random()`:** Why: randomness must be replay-safe and deterministic.
- **Pure step function:** Why: determinism requires `(state, inputsForThatMinute) -> state`.
- **Catch-up coarsening beyond 48 hours:** Why: it bounds replay cost while absent-user intervals have zero drift input and multi-day mood dynamics are dominated by closed-form day/night signal.
- **Simultaneous bird-to-bird influence application:** Why: computing from step-start state avoids order-dependence between birds.
- **Tick latency alarm at p99 5s:** Why: over seven birds the step should be pure arithmetic; high latency indicates an I/O regression.
- **Client-side three-condition presence check:** Why: presence requires visible, focused, recently active viewing, while allowing still watching for a few minutes.
- **Four-minute activity window:** Why: it is long enough for genuinely still watching, short enough to stop crediting an unattended open laptop.
- **Server-side ping hardening:** Why: client pings are not trusted; valid session, arrival window, caps, and truncation protect drift.
- **Credit cap of 60 presence-seconds per wall-clock minute:** Why: two tabs or devices do not double drift.
- **Visitor tokens lacking `events:write`:** Why: visitors cannot create presence windows structurally.
- **Settle and session end terminating presence identically:** Why: no penalty, no difference in credit, and no recovery surface.
- **Drift as monotonic saturating low-pass filter:** Why: birds become more expressive without ever losing trait progress.
- **Single `applyDrift()` mutation site:** Why: the non-negative clamp and property test make monotonic drift enforceable.
- **Drift calibration from named targets:** Why: constants are solved to produce instrument-detectable change at week one and user-visible attended-bird change around week three.
- **Forward-only calibration changes:** Why: changing constants retroactively would rewrite nobody's birds; history remains intact.
- **Mood weighted-softmax with hysteresis:** Why: mood reacts to time, interaction, weather, and personality while persisting for tens of minutes instead of flickering.
- **Mood dwell windows:** Why: no transition before dwell expiry prevents minute-to-minute flicker, except for startle.
- **Personality gating for wary mood:** Why: high-boldness birds should be materially less likely to become wary on identical input.
- **Daily-ish reset after local 04:00 with jitter:** Why: moods refresh from personality-weighted priors without all birds resetting in unison.
- **No mood change on session open:** Why: cross-session persistence means returning to whatever the simulation produced, not a manufactured starting mood.
- **Day phase from local clock, not solar position:** Why: solar correctness would require collecting location, which the product deliberately does not collect.
- **Weather state machine clear/rain/wind:** Why: rare weather creates short-lived mood and call effects without adding thunderstorms, snow, or a weather affordance.
- **Perch selection from boldness, mood, and social pull:** Why: perch is readable behavior expressing personality and mood.
- **Rate-limited perch zone changes:** Why: birds should not ping-pong, and moves render as flight instead of teleport.
- **Renderer compresses spacing on narrow viewports instead of simulation reassignment:** Why: phone and laptop show the same bird in the same place.
- **Server call scheduling with client synthesis:** Why: the server decides calls and seeds for coherence, while audio generation stays client-side and procedural.
- **Weighted call grammar runtime:** Why: signatures, motifs, phrases, and realisations create variation without recorded loops.
- **Recognizability invariant:** Why: mood, drift, weather, and time may modulate tempo and ornamentation, but timbre and motif subset stay fixed so a user can know a bird by ear.
- **Emergent chorus:** Why: overlapping call windows and social warmth produce call-response behavior instead of scheduled stacked loops.
- **At most three concurrent voices:** Why: a fourth call is deferred for mix clarity and CPU control.
- **Wary contagion capped at one hop per tick:** Why: an aviary cannot cascade into all-wary.
- **Return-greeting absence bands:** Why: greeting form changes with absence length while remaining procedurally varied.
- **Greeting composition space instead of variant list:** Why: phrase, ornamentation, timing, angle, and step offsets should make variation real rather than enumerable.
- **No textual welcome anywhere:** Why: greeting should be noticed in bird behavior, and CI prevents welcome strings or toast-like primitives.

### API surface and sync model

- **Base `/v1`, JSON, cookie/bearer auth:** NOT RECOVERABLE FROM PLAN
- **Idempotency keys on mutating requests:** Why: replayed client writes return original results and support append-only event ingestion safely.
- **Matter-of-fact API errors:** Why: naturalist voice never appears in error bodies; system failures should not pretend to be the aviary.
- **Snapshot endpoint with catch-up and ETag:** Why: reads serve canonical current state, avoid unnecessary bodies, and keep cold accounts semantically current.
- **Notebook keyset pagination:** Why: infinite backward scroll is read-only and efficient.
- **Optional `/v1/narration`:** Why: server-generated narration text is available, but current-scene narration is otherwise client-generated to avoid desync.
- **Batched event ingest flushed every 15s or page-hide:** Why: events remain append-only and idempotent while matching presence cadence and sendBeacon behavior.
- **No endpoint writes personality, mood, or perch:** Why: the absence is enforcement of server-owned bird reality.
- **Onboarding adoption body carries only names and no species catalogue:** Why: the user is not offered species choice.
- **Rename endpoint changes name only:** Why: name is decoupled from species and every other bird identity field.
- **Newcomer accept capped at 7:** Why: the bird cap protects recognizability and mix limits.
- **Visitor redeem issues snapshot-read-only token:** Why: a visitor can see one aviary briefly without notebook, settings, events, or drift effects.
- **Visitor snapshot omits greeting:** Why: return-greeting belongs to the host session, not ambient visitor viewing.
- **Revoked/expired visit returns matter-of-fact unavailable surface:** Why: visit failure is a system surface, not an aviary announcement.
- **Server-only writes to personality enforced by Postgres roles:** Why: the rule becomes a database permission rather than a code review habit.
- **Additive deltas, never absolute trait values:** Why: clients report attention events; the server decides what they mean.
- **Atomic cursor advance and state write:** Why: a crash mid-tick reprocesses the same events and deterministic replay produces the same state.
- **Single-writer advisory lock per aviary:** Why: scheduled ticks and read-triggered catch-up serialize.
- **Optimistic concurrency on `tick_seq`:** Why: it is belt-and-braces behind the advisory lock.
- **Multi-device event folding:** Why: two devices should see identical canonical bird state while both can contribute events without double-crediting presence.
- **Listen-in per-device local mix:** Why: listen-in is attention by a person at a device, not a synced property of the aviary.
- **Error surfaces only for rare sustained cases:** Why: transient fetch failures keep rendering the last scene model because the aviary is still there.

### Frontend rendering pipeline

- **Canvas 2D with pre-rasterized sprite/atlas pipeline:** Why: it meets the 2 MB and 5-year-old-laptop targets for a <=7-bird scene without WebGL overhead.
- **Separate non-visual DOM accessibility tree:** Why: Canvas stays cheap while birds remain focusable and screen-reader navigable.
- **Preact for chrome:** Why: it is small, sufficient, and does not touch the render loop.
- **No load state:** Why: the first frame should show a real scene model; there is no spinner, skeleton, or entry animation code path.
- **Bootstrap snapshot inlined into HTML:** Why: it removes the initial round trip so first bird draws as soon as the critical bundle parses.
- **Quiet field when snapshot is absent:** Why: it is a real scene, not a placeholder, and birds arrive rather than crossfading from a loader.
- **Reduced-motion preference read before first paint:** Why: reduced-motion users never see a flash of full motion.
- **Layered frame loop with cached bitmaps:** Why: it keeps the composite within the frame budget and leaves headroom.
- **Dirty-region drawing when scene is still:** Why: rendering work stays low during quiet motion.
- **Perch changes rendered as flight:** Why: snapshots arrive 60 seconds apart but birds must never teleport.
- **Mood changes cross-fade posture:** Why: birds should not snap from fluffed to alert.
- **Scheduled calls carried in snapshot:** Why: audio and beak/throat motion can be frame-accurate without a round trip.
- **Contradictory snapshot completes current motion then re-targets:** Why: never snap-back.
- **Idle micro-motion client PRNG seeded from bird and tick:** Why: devices show similar behavior without needing frame sync.
- **No hold pose with zero motion:** Why: the lowest-energy bird still breathes and blinks; "reads as paused" is a bug.
- **Bird timers phase-offset from each other:** Why: micro-motion never fires in unison across birds.
- **Leaves and feathers as pooled client ornaments:** Why: they add ambient life without server state or per-leaf allocation.
- **No pointer parallax:** Why: pointer response would make the scene behave like an app, not a window.
- **Virtual coordinate scene layout:** Why: responsive fitting guarantees every bird remains in safe area.
- **No clipping/overlap layout invariant tests:** Why: birds must not crop or overlap across viewports.
- **Empty-aviary quiet field and staggered first-bird arrivals:** Why: the first birds enter a real scene, after which the aviary is never empty again.
- **Settle companion control adjacent to offer:** Why: it keeps the top bar at four visual anchors while satisfying top-bar access.
- **Top bar never fades during keyboard focus or open panels:** Why: keyboard-only users are never chasing an invisible control.
- **Reduced-motion second render profile:** Why: the same scene state drives a deliberately authored quiet surface, not a broken animation.
- **Reduced-motion removes particles and parallax but preserves calls, drift, mood, notebook:** Why: accessibility changes presentation, not the underlying aviary.

### Field notebook

- **Template-and-slot notebook generation instead of an LLM:** Why: the voice must be exactly right, reproducible for QA, private, and not generic.
- **Notebook detectors:** Why: entries come from observable bird, aviary, weather, chorus, perch, drift-band, and night activity moments rather than user-behavior logging.
- **Novelty scoring against the aviary's own baseline:** Why: population baselines would require cross-account aggregation, which the telemetry boundary forbids.
- **Sparsity governor targeting about one entry per three days:** Why: active users should not get a feed, and entries should be sparse unless something genuinely unusual happens.
- **Template and subject recency exclusion:** Why: prevents recent repetition and templated-feeling prose.
- **Lowercase naturalist prose:** Why: it is part of the notebook voice rules.
- **Present tense with limited day-reference past tense:** Why: entries are observations, not product event logs.
- **No second person in notebook phrase bank:** Why: it mechanically forecloses "you visited every day this week," "welcome back," and user-behavior observations.
- **Subject must be a bird, aviary, or weather:** Why: the user can never become the notebook subject.
- **No trait numbers, exclamation marks, or gamification vocabulary:** Why: notebook voice stays observational and avoids progress language.
- **Notebook overlay over a still-running aviary:** Why: closing the notebook should not feel like returning to an app.
- **No edit, delete, annotate, share, or export-as-image:** Why: the notebook remains read-only observation, not a social or productivity surface.
- **Naturalist date labels instead of timestamps:** Why: entries stay in naturalist form rather than event-log form.

### Audio pipeline

- **One `AudioContext` created on first user gesture:** Why: browser policy requires gesture start, and recreating contexts would undermine stability.
- **Fixed voice pool:** Why: no per-call node allocation makes the zero-memory-growth requirement achievable.
- **Subtle stereo pan from scene position:** Why: the audio scene matches the visual one.
- **Pitch curves, oscillator frequency ramps, and filters rather than samples:** Why: calls are procedural and have no audio files in the bundle.
- **Per-playback unique phrase seed:** Why: every playback is unique and no cached rendered motif buffer is needed.
- **Listen-in gain ramps with `setTargetAtTime`:** Why: engage/disengage is smooth rebalancing, never a click or channel switch.
- **Ambient floor never zero:** Why: other birds quiet but never disappear from the aviary.
- **Disengage triggers from click, focus, empty space, and Escape:** Why: all PRD interaction paths release listen-in cleanly.
- **Memory discipline across audio, ornaments, notebook, snapshots, and event queue:** Why: the 30-minute no-growth target requires fixed pools, in-place mutation, virtualization, and bounded queues.
- **Graceful silence with captions forced on when audio unavailable:** Why: no recorded fallback exists, and meaning is carried without in-aviary error announcement.
- **Audio silent on hidden tab or audio-off setting:** Why: the context suspends when not visible, and user preference silences without forcing captions unless needed.
- **Asset allowlist excluding audio file formats:** Why: "no recorded audio, unconditional" becomes a build error.

### Accessibility surfaces

- **Client-side screen-reader narration sharing notebook phrase engine:** Why: the client holds the current scene, avoids extra requests, and keeps voices identical rather than merely similar.
- **Polite live narration cadence of 30-60s idle:** Why: narration observes without interrupting or announcing.
- **Priority bumps for user-initiated events, still polite:** Why: important changes are surfaced sooner without using `assertive`, which interrupts.
- **Narration brief/standard only, no verbose:** Why: "more narration is not more access."
- **Accessibility DOM mirror in scene spatial order:** Why: keyboard focus, screen-reader navigation, and hit-testing share one source of truth with the canvas.
- **Roving tabindex keyboard model:** Why: the bird group is a single tab stop while arrow keys move spatially between birds.
- **Enter listen-in and Escape exit/undo paths:** Why: keyboard interaction mirrors pointer/touch affordances and gives settle undo without a pointer.
- **Focus indicator with luminance-selected variants:** Why: visible focus must read at midday and night across sampled backgrounds.
- **Captions generated from the call AST:** Why: caption text matches what actually played.
- **Captions near calling bird with scrim if needed:** Why: captions are the one permitted text inside the scene and must meet contrast against the local background.
- **Caption setting `auto` default:** Why: captions appear when audio is unavailable or muted without forcing them otherwise.
- **Three-state reduced-motion setting:** Why: account preference can honor system, force on, or force off from first paint.
- **Matter-of-fact voice on accessibility and account settings:** Why: system surfaces remain direct and normal-capitalized, separate from naturalist voice.
- **Automated and moderated accessibility verification:** Why: launch acceptance is whether the aviary feels alive to accessibility users, not merely operable.

### Privacy and telemetry boundary

- **Separate telemetry store with no simulation DB access:** Why: request counts and latencies cannot become per-account behavioral analytics.
- **Closed metric dimension allowlist:** Why: account, aviary, bird, email, and free-form dimensions are rejected at ingest.
- **Typed metric dimensions:** Why: adding account-scoped telemetry should be a type error.
- **Named privacy reviewer for metric schema changes:** Why: the boundary must be reviewed when its allowlist changes.
- **Audited time-boxed admin reads for account debugging:** Why: specific account investigation happens through authenticated simulation DB reads, not telemetry.
- **Structured logging email redaction in production:** Why: PII enforcement happens in production, not only tests.
- **Invitee email encryption and lookup hashing:** Why: invitees get the same PII handling as account emails.
- **Export contains only requesting account data:** Why: account export does not cross account boundaries.
- **Hard-delete cascade after 30 days:** Why: account-linked simulation, notebook, invitations, and visits are removed together, while backups expire within one generation.
- **Deletion telemetry as per-run count only:** Why: deletion operations are monitored without per-account metrics.

### Testing, tooling, rollout, and risks

- **Property tests for monotonicity, determinism, dwell, timbre, and presence credit:** Why: the core product promises are mathematical invariants.
- **Contract tests for no trait numbers, no trait write routes, visitor scope, and matter-of-fact errors:** Why: product boundaries are enforced at the API surface.
- **Integration tests for seeded tick, multi-device interleaving, and idempotent replay:** Why: canonical state must survive realistic ordering and replay cases.
- **E2E tests for onboarding, greeting, listen-in, offers, settle, notebook, visits, and keyboard traversal:** Why: user-visible workflows need end-to-end proof.
- **Visual regression across viewports, day phases, and motion profiles:** Why: the scene must remain coherent across responsive layouts and accessibility modes.
- **Calibration harness:** Why: drift must land in instrument-detectable, user-visible, and no-decrease target bands.
- **Aliveness harness:** Why: motion, call, greeting, and perch variation must not collapse into canned or still behavior.
- **Memory soak:** Why: zero memory growth is a product requirement sustained over 30 minutes.
- **Perf lab:** Why: first-bird-visible and bundle size are enforced per commit rather than at the end.
- **Voice lint:** Why: notebook, narration, and captions stay within the banned lexicon and composition rules.
- **Banned-component lint:** Why: toast, snackbar, banner, notification, badge, confetti, achievement, streak, progress bar, leaderboard, and spinner primitives cannot enter the design system.
- **Banned-string lint:** Why: welcome-back, streak, level-up, congratulations, and similar copy cannot create announcement or gamification surfaces.
- **Schema guard for gamification columns:** Why: future counters cannot quietly become easy schema additions.
- **PR template scope questions:** Why: contributors must confront whether a change announces, surfaces user numbers, or adds counters.
- **Milestone overlap of accessibility with audio:** Why: captions and narration depend on the call AST and scene model, so accessibility cannot wait until after launch.
- **Internal dogfood for two weeks:** Why: "feels canned" is only reliably caught by living with the product.
- **Private beta around drift calibration:** Why: real presence patterns may differ from assumed session behavior.
- **Waitlist ramp gates:** Why: production scale advances only while tick, snapshot, first-bird, and calibration signals stay healthy.
- **Quiet in-flow newcomer offer:** Why: a new bird offer is not a notification, badge, or email, and it is never re-pitched.
- **Listener study before higher bird counts:** Why: audio-mix evidence must support recognizability before enabling more birds.
- **Day-one operational instrumentation:** Why: health, performance, and aggregate counts are watched within the telemetry boundary.
- **Explicitly not instrumenting engagement metrics:** Why: building a north-star engagement metric would create the incentive the non-goals forbid.
- **Drift calibration risk controls:** Why: constants are versioned, forward-only, and measured against aggregate presence distributions.
- **Determinism risk controls:** Why: `Math.random()` or wall-clock in the step function would silently corrupt catch-up and multi-device coherence.
- **Personality state backup and consistency checks:** Why: personality state loss is "catastrophic and near-invisible," while monotonicity gives a strong daily corruption check.
- **Audio prototype and listening gate:** Why: bad procedural audio reads worse than no audio, and recorded audio is not a contingency.
- **Recognizability regression test:** Why: feature polish must not tint timbre and erode per-bird identity.
- **Quarterly moderated screen-reader sessions:** Why: accessibility can regress after launch unless treated as a standing practice.
- **Cold-account pre-warm on magic-link request:** Why: returning after months should not pay a long replay latency when the account can catch up before opening.
- **Notebook prose dogfood review:** Why: the notebook is the most concentrated voice surface and must not read as filled-in.
- **Magic-link deliverability mitigations:** Why: magic link is "the only front door," so ESP setup and failure guidance are launch-critical.
- **Definition of done requiring three-week daily use and a perceived bird change:** Why: v1 is not done until a team member can say a specific bird changed without prompting.
