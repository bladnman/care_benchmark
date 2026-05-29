## System-level intent

- **Affective constraints are acceptance criteria, not aspirations.** The opening paragraph says the plan treats "aliveness," "notice never announce," "specificity," "restraint," the "naturalist/matter-of-fact voice split," "monotonic-toward-expressive drift," "server-authoritative state," and "explicit non-goals" as non-negotiable. This shows up again where out-of-scope items are "enforced as build rules" and in the anti-leak guards that make violations build failures.

- **Restraint: the aviary should notice, never announce.** This appears in the prohibition on "Welcome back" surfaces, in return-greetings that vary by absence without text, in top-bar chrome that fades, in mood expressed "without any label," in screen-reader narration written as observations rather than state lists, and in the ban on spinner/fade-from-static load sequences.

- **Product voice is split between naturalist prose and matter-of-fact system copy.** The plan centralizes strings into `naturalist.*` for "aviary, notebook, narration, captions, offer prompts" and `system.*` for "auth, account, sync errors, accessibility settings, unsupported browser, visit-unavailable." It repeatedly names system/account/error copy as "matter-of-fact" and aviary surfaces as naturalist.

- **Aliveness comes from specific, stable bird identity rather than gamification.** The plan pairs stable per-bird IDs, names, seeds, call signatures, mood-shaped motion, and "cap-of-7" recognizability with hard bans on achievements, streaks, levels, scores, badges, counts, Tamagotchi mechanics, and social-network surfaces.

- **Drift is monotonic-toward-expressive, never punitive.** The plan's engine rule is "delta_t is clamped to >=0," with "no neglect term, no decay term, no negative weight." Absence should be "flat, not declining"; a bird may be quieter through mood or time-of-day expression, not because a trait dropped.

- **Server-authoritative state is a product boundary.** The plan says the server owns "personality vectors, mood, drift, mood timers, canonical positions/intentions" and the client submits events only. It repeats that "there is no write endpoint for personality," the tick is "the only writer," and clients never compute or submit personality.

- **The engine is the value, not the topology.** The architecture section says to keep services few because "the engine is the value, not the topology." The rollout de-risks deterministic engine behavior and calibration before UI polish.

- **Calibration is a design method where final values are open.** The plan names starting values for presence window, tick cadence, drift targets, offer cooldown, notebook sparsity, and species pool, then ties them to calibration methods: golden trajectories, a beta manual watch study, and one reviewable `drift_calibration` config.

- **Privacy is an architectural boundary, not a policy footnote.** The plan says email appears only on `account.email_encrypted`, identifiers are synthetic UUIDs, the analytics warehouse never reads the simulation DB, telemetry has no per-account or per-bird dimension, and ML never receives per-bird fields.

- **Accessibility is first-class and designed, not stripped fallback.** The plan says accessibility "ships with v1," reduced motion is "a distinct, calmer aesthetic," and a reduced-motion mode arriving as a "v1.1 fix" is a launch defect. Acceptance includes screen-reader walkthroughs and reduced-motion visual review, not just automated checks.

- **Performance is part of the feel.** Bundle size, time-to-first-bird, 60fps idle, no memory growth, synthetic monitoring, and tick-latency alarms are acceptance criteria. The first frame should be "already in motion," and the load path must avoid spinner/fade-from-static sequences.

- **Guardrails exist because the plan expects well-meaning erosion.** The anti-leak section says these are "temptations a well-meaning contributor will reach for," so naturalist/system voice violations, gamification tokens, negative deltas, telemetry leaks, personality exposure, non-tick vector writes, and aviary spinners become CI failures.

## Per-feature whys

### Scope

- **Single horizontal scene** - The plan ties this to keeping all birds in frame at every viewport, with "no pan/scroll/zoom" and no in-scene chrome. The scene is meant to be watchable rather than navigated.

- **Two starter birds**: NOT RECOVERABLE FROM PLAN

- **Cap of 7 birds** - The plan connects the cap to recognizability: each bird keeps a stable call signature, and the "cap-of-7 affordance" helps a listener tell birds apart by ear.

- **Three perch zones** - The plan uses front/middle/back zones to map depth through scale, y-position, and subtle blur, and to express mood through perch tendency.

- **Day/night anchored to user local time** - The plan says "morning in the user's timezone is morning in the aviary," and uses stored timezone plus snapshot-local phase so the visual and narration agree.

- **Rare ambient weather** - Weather is a mood and expression input: rain dampens vocal expression, wind makes some birds alert and some wary.

- **Ambient leaf/feather drift** - The plan says these ornaments "keep the scene alive between bird actions" while remaining pure client rendering state.

- **Top-bar chrome with fade** - The plan keeps account/settings, accessibility, notebook, and offer controls out of the scene. Fade-to-near-transparent preserves the aviary surface until cursor or keyboard activity returns.

- **Per-bird persistent personality vector** - The plan uses the vector as the slow, server-authored substrate for stable identity and expressive drift from presence and interactions.

- **Mood enum** - Mood is the fast-timescale rendering envelope for perch tendency, idle-motion style, and call density. The user should read it from motion "without any label."

- **Procedural call grammar parameters** - The plan uses grammar params and seeds to keep each bird's call recognizable while generating fresh utterances and matching captions.

- **Slow-tick simulation** - A roughly 60s canonical tick lets the server update drift and mood while the client renders at 60fps from intentions and envelopes, rather than waiting on the server for frames.

- **Monotonic-toward-expressive drift** - The plan makes this the no-Tamagotchi rule: interaction can add tiny deltas, but absence or neglect never subtracts.

- **Bird-to-bird interaction** - Coupling lets chorus, wariness, and social perching emerge from nearby birds and traits; the plan says chorus events should be "emergent, not scripted."

- **Six-species pool**: NOT RECOVERABLE FROM PLAN

- **Nightjar-like night-active signature species**: NOT RECOVERABLE FROM PLAN

- **Stable per-bird identity** - The plan keeps bird IDs stable for account lifetime and call signatures stable across mood and drift so the bird remains recognizable.

- **Age-gated bird offers** - The plan says bird-count ramp is tied to aviary age, "never" to visit count, score, or payment, preserving the no-gamification boundary.

- **Return-greeting** - Greeting is meant to acknowledge absence through bird behavior rather than a "Welcome back" toast. Form scales with absence and bird personality, with staggered greeters and no textual welcome.

- **Listen-in** - The plan says listen-in should feel like "leaning in to listen, not switching a track," so gain ramps slowly and other birds remain ambient rather than muted.

- **Offer types: seed / song-fragment / still-pool**: NOT RECOVERABLE FROM PLAN

- **Offer cooldown at 3 minutes per bird per offer-type**: NOT RECOVERABLE FROM PLAN

- **Settle** - The plan ties settle to a slow evening lighting shift and says settle contributes no drift, "only ends presence cleanly."

- **Settle 5s undo**: NOT RECOVERABLE FROM PLAN

- **Field notebook** - The notebook is a sparse, read-only naturalist observation surface, with a hard rate cap and dedupe key, not a user-authored log or progress display.

- **Presence accounting by three-signal conjunction** - Presence is counted only when visible, focused, and recently active. The plan explicitly biases the activity window long because "watching without moving is the product" and refuses to infer presence from tab-open or connection liveness.

- **Magic-link auth**: NOT RECOVERABLE FROM PLAN

- **Single-user single-aviary**: NOT RECOVERABLE FROM PLAN

- **Synthetic UUID account id** - The plan uses synthetic UUIDs so email is not the identifier moving through services, logs, snapshots, telemetry, or simulation data.

- **Per-device revocable sessions**: NOT RECOVERABLE FROM PLAN

- **Email change with verification**: NOT RECOVERABLE FROM PLAN

- **Account export by emailed link**: NOT RECOVERABLE FROM PLAN

- **Soft-delete for 30 days, then hard-delete**: NOT RECOVERABLE FROM PLAN

- **Append-only client event log** - The log lets clients submit events rather than absolute state. The tick consumes ordered events and applies server-authored additive deltas.

- **Snapshot pull plus client interpolation** - Snapshots are bounded and cheap; interpolation makes perch moves smooth and avoids teleports without requiring frame-by-frame server updates.

- **Additive server-authored personality deltas** - This makes stale-client overwrites and "morning drift silently deleted by a stale lunch write" unreachable.

- **Read-only ambient visit** - The plan allows visitors to see an "identical" host snapshot but exposes no event endpoint, so visitor attention cannot drift host birds and there is "no show-off rendering."

- **Revocable social invites**: NOT RECOVERABLE FROM PLAN

- **30-day invite expiry**: NOT RECOVERABLE FROM PLAN

- **Silent visit log** - The plan keeps visit awareness quiet: logs are recorded, but notifications only occur if the host opted in; otherwise the visit is silent.

- **Visit notification toggle off by default**: NOT RECOVERABLE FROM PLAN

### Architecture

- **Auth/Account service** - It owns email, sessions, email changes, exports, deletion lifecycle, invites, and visit-log reads because those are account and PII concerns rather than simulation concerns.

- **Aviary state service** - It serves snapshots and accepts interaction events but "does not mutate personality," preserving the tick as the only personality writer.

- **Simulation tick worker** - It is the only writer of personality vectors, consumes the log in order, writes canonical state, and emits notebook entries so drift meaning stays server-side.

- **Postgres as system of record** - The plan chooses Postgres-first for v1 simplicity while storing accounts, sessions, invites, birds, personality, mood, notebook, and the append-only log.

- **Snapshot KV** - A small Redis/edge KV holds the latest serialized snapshot for sub-100ms reads and CDN-edge seeding of first paint.

- **Server/client split** - The plan keeps meaning on the server and presentation on the client: the server owns personality, mood, drift, timers, intentions, narration inputs, weather, and local phase; the client owns rendering, interpolation, micro-motion, audio synthesis, presence detection, and captions.

- **Render intentions and envelopes** - The plan separates "intentions and envelopes" from actual motion so a 60s tick can produce a 60fps scene without server frame dependency.

- **Timezone handling** - The plan stores and updates account timezone so day/night and time-of-day mood effects remain local to the user even while away.

### Data model

- **Email only on `account.email_encrypted`** - The plan says all identifiers are synthetic UUIDs and email appears only there, keeping PII out of simulation and telemetry surfaces.

- **Keyed `email_hash`** - The plan says it is for login lookup only and "never logged," supporting auth lookup without exposing raw email.

- **Single-use magic links** - The plan requires atomic consumption and a 15-minute login expiry; the endpoint returns 204 always to avoid account enumeration.

- **Session token hashing** - The plan stores only the hash of a random opaque session secret.

- **`personality_vector` written only by tick** - This enforces the hard rule that clients and non-tick services cannot PUT or overwrite personality.

- **`mood_state`** - It stores current mood, perch, and call timing so mood persists across sessions and can be rendered as an envelope.

- **`bird_offer`** - The tick creates offers when aviary age crosses intervals and bird count is under 7, tying new birds to time rather than gamified counters.

- **Append-only `interaction_event`** - The table captures bounded client events for ordered tick consumption; this is the state-affecting path clients use.

- **`notebook_entry`** - Entries are read-only prose with dedupe keys, supporting sparse naturalist observations without repeated entries.

- **`tick_cursor`** - The cursor guarantees in-order, exactly-once-ish consumption and lets crashed ticks re-run without double-applying drift.

- **`visit_log_entry` approximate duration**: NOT RECOVERABLE FROM PLAN

- **Derived snapshot rather than table** - The plan keeps snapshots bounded and regenerable, with only derived expressive parameters and never raw personality numbers.

### API surface

- **`POST /auth/request-link` returns 204 always** - The plan explicitly ties this to avoiding account enumeration.

- **`GET /aviary/snapshot?tz=...`** - The route is cheap and updates timezone/last-seen, allowing load, visibility-change, frame-gap recovery, and keepalive refreshes.

- **`POST /aviary/events` as the only state-affecting write** - The plan prevents personality writes by making clients append validated, bounded events and letting the server decide meaning.

- **`presence_ping` covered intervals** - Covered intervals let the server accrue presence-time without per-second spam and clamp overlaps so they do not double-count.

- **Over-cooldown offer events accepted but no-op for drift** - The plan keeps the event path permissive while preventing repeated offer events from over-driving drift.

- **`GET /aviary/narration` with ETag** - Cheap polling supports slow-cadence screen-reader narration without overwhelming the queue or the service.

- **Read-only visit snapshot route** - Visit tokens can read but cannot reach an event endpoint, preserving no visitor drift and no co-presence.

- **Centralized copy namespaces and lint** - `naturalist.*` and `system.*` prevent aviary voice from leaking into account/error surfaces and vice versa.

### Simulation engine design

- **Lazy active-aviary ticks** - Dormant aviaries tick lazily on next access to avoid scanning the whole table every minute.

- **Presence-time from pings** - The plan uses pings as the only server evidence of presence, making tab-open and connection liveness insufficient.

- **Drift function as low-pass filter** - Personality moves through tiny calibrated deltas from presence and interactions, so change is slow and reviewable.

- **Presence-dominant weighting** - The plan says presence dominates because the core behavior is watching; listen-in and offers shape focused traits more specifically.

- **`drift_calibration` config** - Constants live in one reviewable object so calibration changes are explicit and gated by golden tests.

- **No negative personality delta** - The engine clamps deltas to nonnegative values, with tests proving absence is flat rather than declining.

- **Mood transition inputs** - Recent interactions, time of day, weather, and personality bias mood so the rendered envelope reflects state without labels.

- **Mood persistence across sessions** - Session-end mood is session-start mood, modulated only by ticks while away, so birds do not reset unnaturally on return.

- **Bird-to-bird coupling** - Calling, wary mood, and social warmth influence nearby birds within bounded coupling, creating chorus and social placement without scripts.

- **Call grammar runtime** - Species motifs plus seed/personality transforms make each bird stable by ear while letting every utterance vary.

### Sync model

- **Single canonical state** - Multi-device sync is a property of laptop and phone reading the same snapshot, not a separate client-to-client feature.

- **No last-write-wins** - Clients submit events, not absolute personality values, so old reads cannot overwrite newer drift.

- **Per-account event ordering** - Server timestamps plus per-account partitioning provide per-aviary order without global ordering cost.

- **Postgres partitioned event log for v1** - The plan calls it the "simplest correct option" with ordered reads and a transactional cursor.

- **Stream migration behind same interface** - If volume warrants, Kafka/Kinesis can replace the log backend without changing client and tick contracts.

- **Snapshot last-tick-wins only** - The plan allows this only because snapshots are derived and regenerable; canonical personality is not last-write-wins.

- **Freshness pulls after visibility change and long frame gaps** - The client recovers from tab return or laptop resume while keeping polling cheap.

- **No websocket required for v1** - The plan says snapshots are kilobytes and polling is cheap; websocket is deferred unless perceived latency demands it.

- **Sync/error matter-of-fact voice** - Auth replay, session timeout, and outages are named exceptions to naturalist voice and live in `system.*`.

### Frontend rendering pipeline

- **Responsive scene composition** - Birds are never cropped; spacing compresses or widens so the whole aviary remains visible.

- **Layered depth** - Background, mid-plane perches and birds, and occasional foreground elements create depth while remaining a single horizontal scene.

- **Canvas2D or lightweight WebGL renderer** - The plan ties renderer choice to the 60fps old-laptop target and 2MB budget.

- **Small SVG/procedural birds with rigged parts** - The plan uses these for micro-motion while staying within the bundle budget.

- **No entry animation, spinner, or fade-from-static** - The plan rejects these because the first frame should already feel alive rather than staged.

- **Edge-seeded first frame already in motion** - A CDN-edge snapshot lets birds appear at current positions and motions immediately, supporting time-to-first-bird and aliveness.

- **Quiet field loading state** - If a snapshot must be fetched, the plan allows a soft field with faint motion cues, "never a spinner."

- **Empty-aviary first-bird soft fly-in** - The plan uses the quiet field only before first bird arrival; after that, the user "never sees an empty aviary again."

- **Mood-shaped idle micro-motion** - Preen, scan, head-tilt, weight-shuffle, and drowsy fluffed poses let mood be read from motion without labels.

- **Procedural variation in motion** - Phase and seed offsets keep idle movement from reading as a loop.

- **Perch transitions and greetings interpolated from snapshots** - Interpolation makes state changes smooth rather than teleporting between server snapshots.

- **Reduced-motion cross-fade surface** - The plan treats reduced motion as "a distinct, calmer aesthetic," preserving calls, drift, mood, notebook, and slowed day/evening color shifts.

- **Return-greeting absence-length signal and greeter selection** - The server chooses greetings from last presence, boldness, and mood so greeting behavior is specific and not a canned welcome.

- **Multiple greeters staggered** - Staggering avoids "a unison chorus on cue" and keeps greetings from feeling scripted.

- **Top bar exact contents** - Account/settings, accessibility settings, field notebook, and offer affordance are the only top-bar items; this avoids badges, overlays, labels, and other in-scene chrome.

- **Top bar fade** - Fade keeps controls available while letting the scene return to near-uninterrupted watching after stillness.

### Audio pipeline

- **Client-side procedural synthesis** - The plan says this is "non-negotiable," supports the <2MB budget, avoids recorded audio, and enables true chorus because layered recordings would phase-cancel.

- **Fresh utterance generation** - Oscillator/wavetable, envelopes, and motif transforms make the same call never heard twice identically.

- **Shared chorus audio graph** - Simultaneous, slightly detuned procedural calls render chorus as actually simultaneous rather than stitched playback.

- **Gentle limiting on master bus** - The plan uses limiting to avoid clipping when several birds call at once.

- **Listen-in gain ramps** - Slow gain changes make listen-in feel like leaning in; other birds remain ambient and there are no hard cuts.

- **Single bounded `AudioContext` and pooled nodes** - This supports the no-memory-growth CI test by avoiding leaked per-call allocations.

- **WebAudio graceful silence with captions** - If WebAudio is unavailable or denied, the aviary still runs and captions default on; no recorded-audio path ships.

### Accessibility surfaces

- **Screen-reader narration** - Naturalist prose comes from the same canonical state as the visual, so nonvisual users receive observations rather than state lists.

- **Slow narration cadence** - Idle updates are about every 30-60s so the screen-reader queue is not overwhelmed.

- **Priority bumps for user-initiated events** - Return-greeting, offer reaction, and settle can prompt narration while staying observational.

- **Call captions** - Runtime captions use the same grammar params as synthesis, so caption text matches what played.

- **Captions on by default in silent mode** - This keeps call information available when WebAudio is unavailable.

- **Keyboard navigation** - The plan gives keyboard access to top-bar items, birds, listen-in, offers, and settle, making the aviary operable without pointer input.

- **Visible focus indicator** - A soft high-contrast outline must work against bright and dim aviary states.

- **WCAG AA user copy** - The plan applies contrast to top-bar labels, settings, account/error surfaces, captions, and displayed narration.

- **Scene carries no copy except top bar** - This reinforces the no-label, watchable scene stance.

### Performance budgets and observability

- **Initial JS bundle <=2MB gzipped** - The plan ties this to code-splitting settings, accessibility settings, and visit/invite flows out of first paint, plus procedural/small-SVG visuals and CI size limits.

- **Time-to-first-bird under 500ms** - Edge-seeded snapshots, drawing birds before non-critical assets, and deferred synth warmup are the stated reasons.

- **60fps idle motion on a 5-year-old laptop** - The plan treats this as a sustained runtime budget, not just launch performance.

- **No memory growth over 30 minutes** - Reused audio buffers, bounded contexts/workers, notebook DOM cleanup, and CI enforcement protect long watching sessions.

- **No per-bird or per-account analytics** - The plan says the simulation DB is never read by analytics and ML never receives per-bird fields, preserving the privacy boundary.

- **Synthetic browser fleet** - Synthetic checks measure load timing, first-bird render, frame timing, and audio-context errors across common geographies.

- **Aggregate-only RUM** - RUM keeps page-load, first-bird-render, frame timing, audio errors, tick latency, and anonymized session-duration histograms without per-account dimensions.

- **Tick-latency p99 >5s alarm** - The alarm catches simulation delay that would make the canonical state feel stale.

- **Last two major browser versions only** - The plan avoids legacy shims because they would cost the bundle budget.

### Engineering structure and guardrails

- **`client/` package** - It owns rendering, audio, presence detection, accessibility surfaces, and copy namespaces because those are client responsibilities in the split.

- **`engine/` package** - It is pure, deterministic, framework-free, and seedable so the tick worker and tests run the same simulation logic.

- **`services/auth`, `services/state`, `services/tick`** - These are thin app tiers around engine and datastores, matching the few-services architecture.

- **`shared/` package** - Shared schemas, copy namespaces, UUID helpers, and PII helpers keep contracts consistent across services and client.

- **Deterministic engine** - Given state, events, seed, and clock, determinism makes drift calibration testable and tick replay/idempotency possible.

- **30-minute memory CI test** - A headless session with real audio graph and render loop catches heap and audio-node growth.

- **Anti-leak CI guard** - The plan makes non-goal violations build failures because they are tempting regressions.

### Drift calibration and test harness

- **Golden trajectories** - Canonical simulations test regular watching, weekend-only visits, absence and return, and heavy listen-in so calibration is grounded in expected behaviors.

- **Instrument-detectable by day 7 and user-perceptible by day 21** - These targets define the desired pace between "screensaver-slow" and "Tamagotchi-fast."

- **Flat absence trajectory** - The two-week-absence test proves absence does not punish birds.

- **Focused bird outpacing peers under listen-in** - The plan uses this to verify that listen-in specifically affects social warmth and vocal frequency.

- **One calibration config** - Keeping constants in one place makes tuning "a single reviewable diff."

- **Beta watch study for activity window** - The manual study calibrates the presence window while preserving the bias that still-watching counts.

### Rollout

- **Foundations first** - Auth, synthetic UUID, sessions, encrypted email, datastores, snapshot KV, and edge-seeded HTML come first, with anti-leak guards and voice namespaces "from commit one."

- **Engine core before UI polish** - The plan says "calibration is the riskiest thing, de-risk it first."

- **Tick worker after engine core** - Event-log consumption, additive deltas, snapshot generation, and idempotent cursor come before client aviary work so the canonical state path exists.

- **Client aviary after tick worker** - Rendering, idle motion, interpolation, first-frame-in-motion, top bar fade, and presence detection then attach to canonical state.

- **Audio before interactions** - Procedural synthesis, chorus, listen-in mix, fallback captions, and memory CI are wired before return-greeting, offer, settle, notebook, and narration.

- **Accessibility pass gated as v1 launch criteria** - Screen-reader cadence, reduced motion, captions, keyboard nav, and contrast must be complete before launch.

- **Social after accessibility**: NOT RECOVERABLE FROM PLAN

- **Account lifecycle after social**: NOT RECOVERABLE FROM PLAN

- **Performance hardening before GA** - Synthetic monitoring and tick-latency alarms gate GA after core product and lifecycle features exist.

- **Bird-count ramp** - Every aviary starts at 2, later offers fire by age intervals up to 7, and cadence is never tied to visit count, score, or payment.

- **Aggregate offer-cadence instrumentation** - The plan permits only aggregate, anonymized measurement to verify pacing, not per-account tracking.

### Risks, mitigations, and deferred specs

- **Drift miscalibration mitigation** - Golden trajectories, isolated constants, and monotonicity tests address the central risk of being "Tamagotchi-fast or screensaver-slow."

- **Sync correctness mitigation** - Single writer, additive deltas, ordered log consumption, and no last-write-wins path make lost drift structurally unreachable.

- **Audio uncanniness mitigation** - Procedural synthesis, per-utterance variation, detuned chorus offsets, slow ramps, and sound-design time address synthetic-sounding birds.

- **Accessibility regression mitigation** - Voice continuity tests, reduced-motion review, and launch criteria prevent narration from becoming state lists or reduced motion becoming "animations off."

- **Notice-never-announce erosion mitigation** - Anti-leak guards catch toast/banner/streak surfaces before they ship.

- **PII leakage mitigation** - Synthetic UUIDs, email-only account storage, HMAC lookup hashes, no logged email hash, and no per-account telemetry dimensions protect identity.

- **First-bird and bundle regression mitigation** - Edge-seeded snapshots, code-splitting, procedural assets, size limits, and synthetic perf gates protect first paint.

- **Tick scalability mitigation** - Lazy ticking, sharding by account UUID, p99 alarms, and possible stream migration address scale without changing contracts.

- **Memory growth mitigation** - Pooled audio buffers, bounded contexts/workers, notebook DOM cleanup, and the 30-minute CI test address long-session leaks.

- **Palette colors and contrast ratios deferred to design-system spec** - The plan states these belong to the visual designer's spec, while this plan defines interfaces and acceptance criteria.

- **Perch geometry and responsive breakpoints deferred to rendering spec** - The plan names the owner spec rather than inventing final layout values.

- **Motif libraries and sound-design parameters deferred to audio spec** - The plan names the audio spec as owner of final species motifs and synthesis parameters.

- **Final calibration constants deferred to tests and beta study** - The plan leaves final constants to golden trajectories and the manual watch study rather than hard-coding them here.
