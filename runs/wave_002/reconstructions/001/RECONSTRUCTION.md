## System-level intent

- **Affective requirements must become structural or automated constraints.** The plan's organizing principle says "every affective rule in the PRD is translated into a structural or automated constraint," because affective requirements "rot silently." This shows up in missing design-system primitives, revoked database grants, serializer allowlists, CI-failing property tests, and the named "charm-invariant suite."

- **Notice, never announce.** The plan repeatedly rejects UI that announces the user's arrival or system state: "No 'Welcome back!' toast," "no textual welcome," "session start adds no text," no autoplay prompt, no spinner, no modal on session start, and a return-greeting that is one bird noticing rather than a surface telling the user they arrived.

- **The relationship is protected from becoming stat management.** Personality vectors are "never exposed numerically," trait values cross the wire only as "the decision the trait already made," mood has no label or badge, perch is "a signal the user reads," and the account export exception is framed as "data portability, not a dashboard." The plan names the protected thing directly: "the relationship."

- **Specificity is the charm engine.** The plan uses "charm comes from specificity" as a motivating requirement and turns it into stable bird identity, per-bird call signatures, bird names or descriptors in naturalist copy, generated greetings with "zero exact duplicates," and notebook entries that stay sparse so observations do not become noise.

- **The aviary continues whether or not the client is present.** Server-side simulation, dormant catch-up, snapshot pull, and "anything that must survive a device switch or an absence is server state" all serve the same intent: "the aviary has been continuing" should be true, not a rendering trick.

- **Change is monotonic, non-punitive, and relationship-shaped.** Drift is "monotonic toward expressive," absence contributes "no change," neglect produces "ambient quietness only," and a correction can add drift but "can never take drift away." This appears in the drift formula, the database trigger, property tests, and the launch posture of erring slow.

- **Presence means qualified attention, not an open tab or engagement score.** The plan implements the "three-signal conjunction," credits server-derived intervals, treats multi-device presence as "union, not sum," and explicitly says watching birds without moving "is the actual product." It also rejects DAU/WAU dashboards, visit streaks, and notebook observations of user behavior.

- **Privacy is enforced through architectural absence.** The synthetic-ID rule, encrypted email plus blind index, no account dimensions in telemetry, no ETL path from the simulation database, no cross-account aggregates, and no visitor event table are all presented as ways to make the wrong thing hard or impossible to add later.

- **There are two voices, and the line is mechanical.** Naturalist surfaces use lowercase, present-tense observation with specific nouns; system surfaces use "matter-of-fact voice." The line is drawn by directory structure, copy linters, and the rule that "any surface where the user is engaging with the system as a system" leaves the naturalist register.

- **Accessibility is the actual product, not a stripped-down variant.** The plan says accessibility "ships with v1, not after" and that the user gets "the actual product." Reduced motion uses the same scene graph and state, narration shares the naturalist grammar package, and M3 is a launch gate rather than a later fix.

- **Procedural assets are both an aesthetic and performance choice.** Bird artwork is procedural geometry, calls are synthesized, motifs are parametric primitives, and recorded audio is banned. These choices make six species, seven birds, five plumage steps, recognizable calls, and the 2MB and 500ms budgets fit together.

- **Determinism keeps continuity honest.** The tick has deterministic randomness, no wall-clock reads outside the day/weather stage, byte-for-byte catch-up equivalence, and replayable behavior for tests and incidents. The plan treats determinism as the reason catch-up can preserve the conceit rather than approximate it.

- **Social is optional, read-only, and deliberately non-networked.** Visits are off by default, invite-scoped, revocable, read-only, and separated by router and database role. The plan explicitly frames visits as a "seed crystal for a social network" and removes the tables, routes, and metrics that would let that happen quietly.

- **The architecture is deliberately boring where novelty is not the product.** Five deployable units, REST/JSON, polling, Postgres as the event log, Canvas 2D, and no Kafka/WebSockets/WebGL are justified as keeping attention on the novel risks: drift, continuity, recognizability, accessibility, and the first-bird experience.

- **Charm and product integrity are launch gates.** ABX listening tests, real-laptop performance gates, grant-matrix verification, copy review, the charm-invariant suite, and a six-week beta exist because the plan treats "feels alive" and "visible after about three weeks" as claims that must be tested before launch.

## Per-feature whys

### Scope boundaries and non-goals

- **Native apps.** The plan excludes native apps so the data model and protocols are not shaped around "hypothetical native-client constraints"; the API is designed for the web client being built.

- **Gamification prohibition.** Achievements, badges, scores, streaks, counters, ranks, and visit-frequency surfacing are banned because the notebook may observe the aviary but "may never observe the user's behavior," and because gamification would move the product toward counting and ranking.

- **Tamagotchi mechanics prohibition.** Death, hunger, distress, decay meters, and negative drift are excluded so neglect produces "ambient quietness only" rather than harm or punishment.

- **Social-network surfaces prohibition.** Profiles, follows, feeds, discovery, chat, comments, co-presence, leaderboards, and show-off rendering are excluded, and the plan also avoids computing the metrics those features would need, so they cannot be "just exposed" later.

- **Notifications prohibition.** Push, aviary email, and re-engagement mail are excluded because the plan treats notification-style re-engagement as outside the product's register; the only exception is a user-requested, default-off visit notification.

- **Payments, shared aviaries, multi-aviary accounts, customizable scenes, species catalogs, rarity.** NOT RECOVERABLE FROM PLAN

- **Recorded audio prohibition.** Recorded audio is banned "at any quality, under any fallback path" because the product needs procedural calls, no bundle audio files, no looping artifacts, and call captions tied to the live grammar rather than recordings.

- **Visual design system as a separate spec.** The plan keeps exact palette values, contrast ratios, focus-ring treatment, and silhouette artwork with the visual designer, while specifying the slots and automated checks, so design choices can vary without weakening validation.

- **Naturalist content package ownership.** Notebook templates, narration templates, caption phrases, and descriptors are authored by the voice owner as a versioned package because naturalist copy is "the product's charm engine" and cannot be an implementation side effect.

- **Procedural bird artwork.** Bird artwork is procedural geometry with designer-authored parameters, not sprite sheets, because the 2MB budget forces it and because six species plus plumage and mood-shaped poses need to fit in the scene budget.

### Accounts and identity

- **Single-user accounts, one aviary per account.** NOT RECOVERABLE FROM PLAN

- **Email plus magic-link sign-in.** The auth flow uses identical request-link responses and timing profiles to prevent account enumeration, and consumes links in a single transaction so replay is "impossible, not merely unlikely."

- **Per-device sessions listed and revocable.** Device-session metadata is intentionally coarse, such as "Safari on iPhone" and country-level region, for the user's session list "not fingerprinting"; revocation gives the account holder direct control over devices.

- **Email change with new-address verification before cutover.** The plan requires verification to the new address before changing account email, so the account does not move until the destination is proven reachable.

- **JSON account export.** Export is the narrow place where trait values may leave the server because it is a user-initiated file for "data portability, not a dashboard"; the product itself still displays no trait values.

- **Soft account deletion with a 30-day recovery window.** The feature preserves a recoverable period before hard deletion, matching the plan's explicit split between `soft_deleted_at` and `hard_delete_after`.

- **Synthetic account identifiers.** UUID account IDs and account_id references keep email out of natural keys, logs, partitions, shards, and telemetry, preventing the PII leakage failure the plan calls "the single most important boring detail."

- **Email blind index.** The blind index exists so sign-in can find an account without decrypting every row and so email-based rate limiting can work, but it is not a general-purpose key and is restricted to auth.

### The aviary

- **Two starter birds, system-selected and user-named.** The plan says the first encounter should be "meeting an animal, not configuring an avatar," so there is no catalog or picker; default name suggestions keep the user from facing a blank.

- **Seven-bird cap and age-gated growth.** The cap is tied to recognizability and chorus legibility, while two birds are "companionship without overload"; age-gated offers make growth "relationship pacing," never a reward curve.

- **Single horizontal scene, one screen, no pan, scroll, or zoom.** NOT RECOVERABLE FROM PLAN

- **Three perch zones, bird-chosen and never user-arranged.** Perch is "a signal the user reads"; user controls are absent so front, middle, and back remain expressive outputs of mood and boldness rather than layout customization.

- **Local-time day/night cycle.** Day/night follows `account.tz_iana` so the aviary feels locally continuous, and timezone changes are hysteretic and clamped so "evening" does not visibly un-happen.

- **Rare ambient weather.** NOT RECOVERABLE FROM PLAN

- **Ambient leaf and feather drift with subtle parallax.** Leaves and feathers are client-side ornament because making them server state would cost per-leaf records for "zero user-visible benefit"; parallax is kept just past notice because the product is "not parallax-heavy."

- **Thin top bar with exactly four affordances.** The top bar is where chrome lives; the scene carries no buttons, badges, labels, or hover states, preserving the aviary as a watched place rather than an interface panel.

- **Quiet field loading and empty states.** The quiet field uses sky and foliage instead of a spinner because "a spinner says machine"; it reads as the aviary before birds have appeared or before data has arrived.

### The bird engine

- **Hidden five-trait personality vector.** The vector is server-persisted and never serialized because visible trait values would turn the relationship into stat management; traits reach the client only as resolved rendering decisions.

- **Monotonic drift toward expressive.** The plan makes drift non-negative by formula, database trigger, and property test so no code path can quietly take expressiveness away from a bird.

- **Five-state mood system.** Mood is a fast-timescale state that shapes hazards, calls, motion, and weather response; persistence and dwell floors keep mood from snapping or flickering when a session opens.

- **Procedural call grammar with stable signatures.** The signature/expressive split lets a bird remain recognizable across mood and personality drift: timbre, intervals, motif subset, and opening phrase do not change, while expression does.

- **Mood-shaped idle micro-motion.** The user reads mood from motion, so there are no mood labels, icons, or badges; pose sampling makes mood visible without announcing it.

- **Bird-to-bird call-and-response, mood contagion, and emergent chorus.** Chorus is designed to feel like something that "happened" rather than "fired," so overlapping independent schedules and mutual excitation replace scripted events.

- **Six-species pool with one nocturnal species.** The plan ties the pool to "about six species" and one night-shifted signature; the nocturnal species keeps night from becoming "a dead state."

- **Stable per-bird identity.** `bird.id`, call seed, signature, and migration rules make a bird invariant across rename, sync, and species-pool changes, preserving identity for the life of the account.

- **Age-gated third-bird-and-beyond offers.** Offers are driven by aviary age, not visit count or interaction score, so they pace a relationship rather than reward engagement.

### Interactions

- **Return-greeting.** The greeting is server-computed in the session-start snapshot so it is consistent, narratable, and testable; one bird notices the user with absence-shaped variation, and no text announces the arrival.

- **Listen-in.** Listen-in is a mix rebalance, not solo or mute; non-focused birds never go silent because silencing them would teach the user the aviary is "a set of soloable tracks."

- **Offer: seed, song fragment, still pool.** Offer lives in the top bar because chrome belongs there, uses cooldowns to avoid overuse, and appears as a bird arriving at the scene edge rather than as a notification, badge, or modal.

- **Settle with a five-second undo.** NOT RECOVERABLE FROM PLAN

- **Field notebook.** The notebook is sparse, read-only, auto-generated naturalist prose because it is an observer's record; entries observe the aviary, never the user's behavior, and scarcity keeps them meaningful.

- **Presence accounting on the three-signal conjunction.** Presence requires visible, focused, recent input because the engine should credit actual watching, not background tabs; the five-minute activity window is long because still watching is "the actual product."

### Sync and social

- **Server-side simulation tick independent of client connections.** The independent tick makes "the aviary has been continuing" true and prevents client connection state from becoming the source of continuity.

- **Snapshot-pull clients with interpolation and append-only event log.** Polling fits a 60s tick and sparse triggers, ETags make unchanged ticks cheap, and easing prevents snapshot disagreement from popping, remounting, or showing a load state.

- **Multi-device coherence as an architectural property.** The plan avoids mergeable client state; overlapping presence is unioned because one user on two devices is still "one user watching."

- **Visit invitations.** Visits are per-invite, email-addressed, read-only, revocable, expiring, and off by default so sharing exists without visitor drift, co-presence, or social-network infrastructure.

- **Visit log in account settings.** NOT RECOVERABLE FROM PLAN

- **Optional visit-notification toggle, default off.** This is the single allowed notification because it is user-requested and per-account, not re-engagement mail about the aviary.

- **Separate visitor router and SELECT-only role.** The visitor path has no event-ingest routes and no write grants, so visitor attention cannot be recorded as drift input; using the same snapshot builder prevents show-off rendering divergence.

### API and data mechanics

- **Five deployable units and no Kafka.** The plan calls the units "deliberately boring"; Postgres partitioned events are enough for the write rate, and avoiding streaming infrastructure avoids PII leaking through partition keys.

- **Grant-level separation between `aviary-api` and `sim-tick`.** This is the "one architecturally load-bearing decision": API code cannot update personality columns even if a developer writes the statement.

- **Client/server split.** The rule is that anything surviving a device switch or absence is server state, while 60fps render ornament is client state; this prevents ornamental details from bloating canonical simulation.

- **Render boundary at call plan and pose target.** The server sends expressive plans and motion phase offsets rather than pixels or animation frames, making the first drawn frame mid-action by construction.

- **Append-only interaction events with idempotency.** Client events are accepted append-only and deduped by `(account_id, client_event_id)` so retries do not double-count and clients never overwrite interpreted state.

- **Ninety-day raw-event retention.** The plan calls this a "privacy-positive default" because folded deltas and canonical vectors carry what the product needs, while long retention of per-offer records serves nothing requested.

- **Notebook entries stored rendered.** Existing entries must not silently rewrite when grammar changes because "the notebook is an observer's record, and a record that changes retroactively is not a record."

- **Snapshot serializer field allowlist and quantized drivers.** The allowlist prevents trait leakage, and five plumage steps avoid a continuous trait scalar "wearing a different name" while preserving visible drift.

- **Machine error codes mapped to client copy.** The server never emits user-facing prose because it would eventually emit a string in the wrong voice; the client maps codes to matter-of-fact system copy.

- **Polling instead of WebSockets.** With 60s ticks and sparse event-driven pulls, persistent sockets would add connection state for "no latency the user can perceive."

### Simulation engine

- **Exactly-once tick execution.** Compare-and-set on `last_tick_index` plus a unique `(bird_id, tick_index)` delta guard keeps workers from double-applying drift.

- **Deterministic randomness.** A splittable PRNG seeded by aviary, tick, stage, and bird makes tick behavior reproducible in tests and incident replay and enables catch-up equivalence.

- **Drift formula and calibration.** The formula approaches 1 asymptotically, early drift is faster than late drift, and `alpha` is derived from the plan's measurable one-week and visible three-week targets rather than hand-tuned "toward a feeling."

- **Daily presence saturation.** The soft cap makes "a single session never moves a personality value visibly" and prevents a mouse-jiggling nine-hour tab from out-drifting honest watching.

- **Hazard-based mood.** Mood uses propensities rather than hard triggers so personality, weather, events, day phase, and contagion can shape what the user sees without visible state-machine snaps.

- **Mood dwell floor and non-snapping baseline.** Dwell floors prevent visible flicker, session start cannot reset mood, and the daily baseline shifts the hazard field gradually rather than setting mood outright.

- **Perch sampling with stickiness.** Stickiness keeps birds from changing perch every tick, while boldness, warmth, and mood make position expressive and legible.

- **Call plan generation with buffered horizon.** Inhomogeneous Poisson scheduling, refractory limits, response scheduling, and a held 120s buffer keep calls social, non-scripted, and resilient to late snapshots.

- **Dormant catch-up.** Catch-up reduces cost while preserving the property exactly: for no-event intervals it must be byte-for-byte equivalent to folding every 60s tick, including notebook entries.

### Frontend rendering pipeline

- **Canvas 2D.** The plan chooses Canvas 2D because seven birds, four layers, and ornaments do not need WebGL, while WebGL would add context-loss handling, shader compilation, and core-chunk size against the 500ms budget.

- **Boot path with inline snapshot and first bird under 500ms.** Inlining the snapshot avoids a network round trip for first render, paints correct light early, and supports the success check that the user "does not perceive a load."

- **Bundle budget and chunk allocation.** Per-chunk budgets keep the first paint under 2MB gzipped, with deliberate headroom so the budget remains an enforced product constraint rather than a late optimization.

- **Four-layer scene composition.** Sky, background foliage, perch plane, and foreground ornament separate ambience from birds; birds do not parallax because subject parallax would make the aviary feel like "a diorama."

- **Responsive layout and no cropped birds.** Safe-area anchors, zone scaling, and viewport tests make every bird fully visible from small mobile to large desktop, preserving legibility at two and seven birds.

- **Noise-driven pose walk rather than clips.** The pose walk ensures birds are "never still in a way that reads as paused" and never visibly looping, with `motion_phase` ensuring no first-frame clip start.

- **Reduced-motion register.** Reduced motion swaps the sampler while keeping the same scene graph, state, calls, drift, mood, notebook, and narration, preventing the mode from rotting into a fallback.

- **Hidden-tab and resume behavior.** Rendering, audio, and presence stop when hidden to save battery and avoid false presence, while server simulation continues and the returning first frame is mid-action again.

- **Scene carries no chrome.** In-scene controls, labels, badges, and tooltips are absent so the aviary remains a scene; only captions and focus rings may overlay it as accessibility surfaces.

- **Onboarding and empty aviary.** Onboarding introduces birds as arrivals, not selected avatars; the empty aviary is a real quiet-field state so the gap before birds arrive is designed rather than a blank.

### Audio pipeline

- **Fully synthesized audio graph.** Synthesized calls and ambient bed avoid file assets, looping, and recorded-audio fallbacks while keeping audio tied to bird signatures and live plans.

- **Voice pooling.** Persistent oscillator pools are required for the 30-minute no-memory-growth rule and scheduling latency, because WebAudio oscillator nodes are single-use.

- **Lookahead scheduler.** A 25ms interval and 150ms horizon commit automation ahead of time so calls survive main-thread jank.

- **Chorus mixing.** Spatial spread, crowd ducking, and no pumping compressor preserve per-bird recognizability at the seven-bird cap.

- **Listen-in mix floor.** The focused bird comes forward, but other birds are clamped above silence so the aviary never becomes a collection of solo tracks.

- **Autoplay-policy resolution.** If audio is blocked, the aviary runs silently with captions rather than showing a "click to enable sound" prompt, because a prompt would be an announcement.

- **Captions from the same call plan as synthesis.** Using the same plan means captions cannot describe a call that did not play, and caption text stays in naturalist voice.

- **Ambient bed and offer sounds.** Procedural filtered noise and motifs prevent audible loops, and offer responses come back through the tick and call plan rather than an immediate mechanical sound effect.

### Accessibility surfaces

- **Screen-reader narration.** Narration shares the naturalist grammar package with the notebook so the two surfaces cannot drift into different products; slow, polite updates avoid forcing the user to silence it.

- **Keyboard navigation.** Roving focus makes birds one tab stop with directional movement by position and perch zone, giving keyboard users the same listen-in vocabulary without turning birds into generic controls.

- **Focus indication.** The double light/dark outline is chosen and sampled against all scene states so focus remains visible against bright sky, dim night, and weather.

- **Captions as DOM overlays.** Captions serve audio-off, hearing-difference, and noisy-environment users, stay hidden from screen readers to avoid live-region flooding, and use a local scrim for AA contrast.

- **Contrast validation.** The plan keeps user copy surface area small and testable, with token-pair checks for static chrome and palette-ramp sampling for dynamic overlays.

- **Voice split enforcement.** Separate directories and linters make naturalist and system strings mechanically distinct, catching misplaced copy rather than relying on review memory.

### Performance and observability

- **Performance budgets.** Bundle size, first-bird timing, frame rate, memory, and tick latency are enforced because the product depends on not feeling like a load, a machine, or a leaking long-running tab.

- **Memory traps pre-empted.** Audio pooling, virtualized notebook, bounded raster caches, fixed particle pools, and snapshot trimming directly protect the "zero memory growth over 30 minutes" requirement.

- **Aggregate RUM only.** The plan measures page load, frames, audio availability, route latency, tick lag, and aggregate auth counts without per-account dimensions to preserve the privacy architecture.

- **Deliberately unmeasured engagement and drift metrics.** No DAU, WAU, retention cohorts, per-bird production telemetry, or drift aggregates are collected because those metrics are product-hostile or violate the privacy commitment.

- **Synthetic cohort for drift calibration.** Calibration is tested with seeded staging aviaries because mining production bird data, even in aggregate, would violate the plan's privacy rule.

- **Product-integrity alarms.** Drift monotonicity violations, email-in-log hits, tick lag, grant-denied personality writes, and catch-up failures page because they indicate silent product failure, not ordinary bugs.

- **Browser support without old-engine shims.** The plan supports recent major browsers and rejects compatibility shims because polyfill bundles would bloat the critical path, and the bundle is the binding client constraint.

### Rollout, risks, and success posture

- **M0 audio and engine spike with ABX gate.** Procedural call recognizability is the most novel risk, so it is tested before major client work; if two birds cannot pass, seven will not.

- **M1 canonical spine before product surfaces.** Schema, grants, synthetic IDs, auth, tick, serializer allowlist, event ingest, and presence land first because retrofitting these protections is the failure mode the plan wants to avoid.

- **M3 accessibility gate before full feature completion.** Reduced motion, narration, captions, keyboard navigation, focus, contrast, and copy linters gate progress so accessibility cannot become a v1.1 afterthought.

- **Six-week private beta.** The beta has a hard floor because the plan's headline drift claim is "visible after about three weeks," and two-week testing cannot observe it at all.

- **Binary launch gates.** Launch depends on ABX, real-laptop performance, accessibility, grants, monotonicity, catch-up, copy, charm invariants, and privacy gates because each protects a load-bearing part of the product.

- **Instrumentation from day one and post-launch hardening.** The plan refuses to add synthetic cohorts or charm invariants later; the first three months focus on calibration and hardening because drift and seven-bird chorus require real time.

- **Launching `alpha` under the derived value.** The plan deliberately launches around 15% slow because under-drift can be corrected monotonically, while over-drift cannot be corrected for the launch cohort.

- **Tick cost managed by equivalence, not approximation.** Dormant catch-up is acceptable only because it is byte-for-byte equivalent; if the test fails, catch-up is disabled and the fleet falls back to full ticking.

- **Visit scope-creep guardrails.** Visits are kept from becoming a social network by absent visitor event tables, no visitor write routes, no cross-account aggregates, and invariants that visitors never drift the host.
