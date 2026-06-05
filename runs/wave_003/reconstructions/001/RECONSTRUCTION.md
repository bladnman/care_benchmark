## System-level intent

- **Server-canonical life, rendered locally.** The plan repeats that the server holds "Canonical State," the client "never owns state," and the server is "the only writer of personality vectors." This shows up in the render pipeline boundary, the service shape, the sync model, and the load-bearing decision "Server canonical state."

- **Additive, ordered change instead of overwrites.** The plan's sync philosophy is "additive deltas only," "append-only," and "No last-write-wins for personality state." The conflict example explains the intent: both the laptop and phone session contributions should be applied "in event-log order."

- **Slow, monotonic personality change over weeks.** The plan frames drift as "measurable" after about one week and "visible" after about three weeks, with traits that "only move up, never down on neglect." It also calls "Server-side simulation tick," "Additive personality drift," and "Monotonic drift" load-bearing.

- **A relationship shape, not a game shape.** The non-goals and load-bearing decisions reject "achievements," "streaks," "levels," "scores," "badges," and "XP." The plan names "No gamification" as non-negotiable for "relationship shape" and "No Tamagotchi" as tied to the monotonic rule.

- **Naturalist product voice over exposed mechanics.** The plan uses "naturalist prose," "lowercase," "present-tense," and "specific" for screen-reader narration, notebook entries, and call captions. It also says screen-reader users should hear "naturalist prose, not state lists" and that field notebook observations should focus on "mood states, not personality numbers."

- **Accessibility as a designed surface.** Accessibility is in V1 scope, success criteria, and risk mitigation. Reduced-motion mode is explicitly "Not 'animations off'" but "a designed surface in its own right," and accessibility work "ships with v1 (not post-launch)."

- **Performance as part of the spell.** The plan ties bundle size, "time-to-first-bird," "60fps idle motion," no memory growth, and simulation latency to the user-facing goal that birds "feel alive" and not robotic. Performance degradation is treated as a launch risk.

- **Restrained social and telemetry boundaries.** Social is "read-only," "opt-in per host," with "no co-presence," and social-network surfaces are out of scope. Telemetry allows "counts, latencies, error rates" but says "Per-account interaction state" and "Per-bird state" are never part of telemetry.

## Per-feature whys

**V1 In Scope**

- **Bird count:** NOT RECOVERABLE FROM PLAN

- **Single-user accounts with email magic-link sign-in:** NOT RECOVERABLE FROM PLAN

- **Aviary: one horizontal scene, three perch zones, day/night cycle.** The plan ties this to scene composition: birds are positioned by "perch_zone, mood, personality," perch zones support spatial panning, and the day/night cycle updates "palette, call volume, bird positions." Time of day also feeds mood, such as "drowsy near dusk" and "alert morning."

- **Personality vectors.** The plan uses personality vectors to make birds have "distinct personalities," shape motif timing and pitch, influence moods, and drift visibly over weeks. It also says personality values are "never exposed to user," keeping the visible experience behavioral rather than numerical.

- **Mood states.** The plan treats mood as fast and temporary, driven by time of day, recent interactions, ambient events, and personality weighting. The rationale appears in the "Mood-Drift Confusion Risk": mood transitions should be visible in motion, while notebook observations focus on "mood states, not personality numbers."

- **Procedural call synthesis.** The plan says this is non-negotiable for "call variety and chorus." It also rejects recorded loops because they would "sound canned" or violate the bundle budget.

- **Return greeting.** The product success criterion says users should report birds "remember them," and narration events include a prompt on "return-greeting."

- **Listen-in.** The plan makes listen-in both an interaction and an audio mix behavior: it raises the focused bird's mix level and lowers others. It also affects drift through "social warmth" and "vocal frequency," supporting the success criterion that birds "react to interactions."

- **Offer.** The plan connects offer reactions to curiosity drift and mood: accepted offers move curiosity up and can maintain "content"; ignored offers can move a bird to "wary." This supports the success criterion that birds "react to interactions."

- **Settle.** The plan calls settle a "soft session end" and says it should "End presence window cleanly" with "no drift change." It also ends a session explicitly in the presence-time rules.

- **Field notebook.** The plan frames notebook entries as "naturalist prose, lowercase" and "naturalist observation." It also says notebook observations should focus on "mood states, not personality numbers," preserving the product voice and hiding raw personality vectors.

- **Server-side tick.** The plan names this as non-negotiable for "feels alive over weeks." The tick consumes events, updates drift and moods, advances ambient events, writes canonical state, and sometimes generates notebook entries.

- **Multi-device sync via server canonical state and additive deltas.** The explicit rationale is sync correctness: last-write-wins could overwrite a morning personality update with a stale lunch update, but additive deltas let both contributions apply "in event-log order."

- **Read-only visit invitations.** The plan keeps visits "optional," "read-only," "opt-in per host," and with "no co-presence." In context with the non-goals, the why is to allow visiting without creating profiles, follows, feeds, comments, or other social-network surfaces.

- **Accessibility surfaces.** The plan says accessibility surfaces must be functional for screen reader, keyboard, reduced motion, captions, and contrast. The success criteria explain the whys: naturalist prose instead of state lists, a designed reduced-motion surface, interactions without mouse, and bird-call understanding without audio.

- **Performance budgets.** The plan uses bundle, first-bird, FPS, memory, and tick-latency budgets to protect the user-facing experience from feeling slow or broken. Performance degradation is a listed risk because the bundle can grow and "time-to-first-bird" can exceed 500ms.

**V1 Out of Scope**

- **Native mobile applications:** NOT RECOVERABLE FROM PLAN

- **Gamification.** The plan states "No gamification" is non-negotiable for "relationship shape." It specifically excludes achievements, streaks, levels, scores, badges, and XP.

- **Tamagotchi mechanics.** The plan excludes bird death, hunger, distress, and happiness decay, and connects monotonic drift to the "'no Tamagotchi' rule." Traits "only move up, never down on neglect."

- **Social network surfaces.** The plan excludes profiles, follows, public feed, and comments on visits. This aligns with optional visits being read-only, opt-in, and without co-presence.

- **Push notifications:** NOT RECOVERABLE FROM PLAN

- **Payments or monetization:** NOT RECOVERABLE FROM PLAN

- **Shared households or multi-user accounts:** NOT RECOVERABLE FROM PLAN

**Architecture**

- **Render pipeline boundary.** The plan's key principle is that the client renders snapshots and submits events, while the server owns canonical state and personality writes. This protects state ownership while still allowing client-side interpolation, calls, ambient motion, day/night rendering, narration, captions, and reduced motion.

- **Snapshot caching and animation interpolation.** The plan says clients cache "last N states for interpolation" and interpolate between snapshots for "smooth motion." This lets the server tick stay coarse while the browser keeps the aviary visually continuous.

- **Append-only event log.** The event log is timestamped and ordered per account, serves as input to the simulation tick, and is central to additive drift. Its why is correctness under multi-device overlap and avoiding last-write-wins personality overwrites.

- **State and event APIs.** `GET /api/state` exists to get the current "canonical state snapshot"; `POST /api/events` exists to append interaction events rather than write state directly. This follows the rule that clients are read-only for state.

- **Account export, soft deletion, restore, and hard delete:** NOT RECOVERABLE FROM PLAN

**Simulation Engine Design**

- **Tick cadence:** NOT RECOVERABLE FROM PLAN

- **Interaction event processing.** The plan maps interactions to small drift or reset effects: offers can update curiosity, listen-in updates social warmth and vocal frequency, listen-in end resets audio mix, settle cleanly ends presence, and presence pings accumulate presence-time. The rationale is that user activity becomes ordered simulation input, not direct state mutation.

- **Drift calibration.** The plan targets measurable drift after about one week and visible drift after about three weeks of regular visits. The calibration method simulates 30 days and verifies personality vectors shift by 0.1-0.2 per trait so birds do not feel "robotic or unresponsive."

- **Mood transitions.** Mood is computed from time of day, recent interactions, ambient events, and personality weighting. The rationale is that birds react in visible, short-term ways while personality remains slow and permanent.

- **Ambient events.** Leaf drift, weather transitions, and bird-to-bird interaction are advanced by the tick. The plan says leaf drift is client-side but the server tracks phase "for consistency," and weather affects mood and vocal frequency.

- **Notebook entry generation.** The tick generates a naturalist observation with low probability. The calibration point says notebook probability is tested for "sparsity," so the rationale recoverable from the plan is that notebook entries should be occasional observations rather than constant output.

- **Presence-time accumulation.** Presence only counts when the document is visible, focused, and has recent pointer or key activity. The risk section says this avoids corrupting drift with tab-open time; drift should correlate with "active usage, not tab-open time."

**Frontend Rendering Pipeline**

- **Top bar fade to transparent after cursor stillness:** NOT RECOVERABLE FROM PLAN

- **Initial load.** The plan has initial load fetch the server snapshot, place birds, start ambient motion, and begin day/night rendering. This supports the "time-to-first-bird" budget and the requirement that the first visible bird arrive quickly.

- **Idle motion.** Idle motion is client-side procedural animation, "mood-shaped," and independent of the server tick. This lets wary birds scan and content birds preen without needing the server to drive every frame.

- **Reduced-motion rendering.** Reduced-motion replaces frame-by-frame animations and perch paths with cross-fades, slows color shifts, removes leaf drift, and keeps calls and captions. The plan's why is that reduced motion is a designed surface, not broken animations or simply "animations off."

**Audio Pipeline**

- **Motif library and motif player.** Motifs vary by species, mood, personality, timing, and pitch. The rationale is call variety: high vocal frequency changes intervals, boldness changes loudness and simplicity, curiosity increases variation, and mood shifts pitch.

- **Chorus mixer.** The mixer supports listen-in, ambient levels, real-time mixing, and spatial panning by perch zone. The plan rejects pre-recorded stacks, tying this to real-time chorus and avoiding canned sound.

- **Call captioning.** Caption text is generated per call, appears near the calling bird, and uses the same naturalist voice as the field notebook. The success criterion says call-caption users should understand bird calls without audio.

- **WebAudio fallback.** If WebAudio is unavailable, the plan chooses "graceful silence," captions on by default, and no recorded fallback because recorded audio would violate bundle budget or "sound canned."

**Accessibility Surfaces**

- **Screen-reader narration.** The plan gives narration a cadence and queue management "to prevent overwhelming screen reader." Its voice is naturalist, lowercase, present-tense, and specific, so screen-reader users hear prose rather than state lists.

- **Keyboard navigation and focus indicators.** Keyboard users must be able to focus birds, move between birds, listen-in, exit, open offer, open notebook, and settle. Focus indicators must be visible against bright and dim aviary states so all interactive surfaces can be used without a mouse.

- **WCAG AA contrast.** The plan requires all user-copy text to pass AA contrast, including top bar labels, settings, error surfaces, captions, and narration visual display.

**Performance Budgets and Observability**

- **Synthetic monitoring and aggregate RUM.** The plan measures page load, first-bird render, frame timing, audio-context errors, tick latency, and errors so performance and reliability can be caught in CI and production.

- **Telemetry boundary.** Aggregate metrics are allowed, but per-account interaction state and per-bird state are never included. The why recoverable from the plan is observability without collecting individual aviary behavior.

- **Error budget alarms.** Simulation-tick p99 above 5 seconds, audio-context errors above 1%, and time-to-first-bird p95 above 750ms all alarm. These thresholds defend the same performance and audio risks named later.

**Rollout**

- **Single deploy to production with no feature flags for core features:** NOT RECOVERABLE FROM PLAN

- **Optional feature flag for visits:** NOT RECOVERABLE FROM PLAN

- **Bird-per-aviary ramp.** The plan says no ramp is needed because "2 is the minimum, 7 is the cap" and the cap is "built into engine."

- **Instrumentation from day one.** Synthetic performance checks, aggregate RUM, and error rates are immediate, while "No per-account telemetry" is by design. This matches the performance and telemetry boundaries above.
