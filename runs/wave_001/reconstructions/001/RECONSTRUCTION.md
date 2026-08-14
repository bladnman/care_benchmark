## System-level intent

- **Ambient virtual aviary, not a game surface.** This appears in the title and scope as a "browser-based ambient virtual aviary," in the "subtle ambient weather" and "micro-motion" scene design, and in the explicit prohibition on "Gamification Elements" such as streak counters, badges, achievements, levels, XP, or score counters.

- **Quiet, naturalist product voice.** The plan repeatedly favors naturalist language and quiet observation: the "Field Notebook" is made of "auto-generated naturalist prose observations," screen-reader narration delivers "naturalist prose updates," and notebook generation uses "lower-case present-tense rules." The social mode is named "Quiet Social Visits."

- **Presence should matter only when it is real attention.** The "tri-condition presence detector" and "tri-condition attention validator" recur in scope, client responsibilities, simulation design, and risk mitigation. The mitigation phrase is explicit: avoid "Unattended open tabs inflating presence-time" by requiring `document.visibilityState === 'visible'`, `document.hasFocus()`, and recent user input.

- **No neglect punishment or custodial pressure.** The prohibitions say "No Tamagotchi / Custodial Mechanics" and "No bird illness, hunger, death, distress animations, or negative drift on neglect." The drift section carries the same principle technically: "Traits never decrement upon neglect."

- **Server-canonical continuity over client authority.** The architecture uses a "centralized authoritative simulation daemon," and sync is governed by the "Single Writer Principle." Clients submit "discrete events," "never absolute state," while devices read "the same snapshot cache."

- **Smooth continuity across sessions and devices.** The plan emphasizes "single canonical server-simulated aviary state," "snapshot distribution," "smooth 1.5s interpolation," and backend simulation continuing while hidden tabs pause rendering. Refocused tabs render the aviary "as having continued uninterrupted."

- **Hidden inner systems should surface behaviorally, not numerically.** The plan prohibits "Numerical Trait Exposure," stores "Hidden Personality Vectors" as "Server Canonical Only," and targets "visible behavioral/visual drift" rather than exposed vector scalars.

- **Privacy is a boundary, not just a logging concern.** The architecture calls out "PII isolation," encrypted email, synthetic UUIDs, telemetry decoupled from simulation tables, and analytics pipelines that explicitly block bird names, personality vectors, offer choices, and notebook logs.

- **Procedural variation should avoid repetition while staying light.** The plan uses procedural return-greeting, Markov mood transitions, template-based notebook generation, and WebAudio species grammars with micro-randomized jitter. The audio rationale says synthesized calls fit the bundle constraint and "eliminate repetition."

- **Accessibility is a parallel sensory surface.** The plan includes screen-reader narration, procedural call captions, reduced-motion cross-fade mode, full keyboard navigation, focus rings, and WCAG AA contrast. Audio fallback "automatically activating the call captions surface" shows captions are part of the core experience.

- **Performance budgets protect the feeling of aliveness.** The rendering section specifies "Instant Aliveness," "First frame renders immediately," and "No loading spinners or fade-ins." The performance section fixes time-to-first-bird, frame rate, memory, and bundle budgets.

## Per-feature whys

**1. Scope & System Boundaries**

- **Platform: Modern web browsers.** NOT RECOVERABLE FROM PLAN

- **Aviary Scene: single horizontal viewport-constrained scene.** The frontend rendering section says the canvas maintains aspect ratio constraints "to ensure all birds remain in frame across mobile and wide displays."

- **Three perch zones: Front, Middle, Back.** The rendering and audio sections use these zones to create depth planes and matching spatial audio: Front is "full stereo & dry," Back is "centered & slightly low-pass filtered."

- **Local-time day/night cycle.** The simulation advances "circadian lighting according to local timezone," and mood transitions are conditioned on time of day, with "dusk -> drowsy" and "dawn -> alert."

- **Subtle ambient weather: rain, wind, falling leaves/feathers.** Weather participates in mood and observation: "rain increases dampening factor," and "weather passage" is an anchor event for notebook generation.

- **Micro-motion.** The plan ties this to aliveness and visual continuity: procedural body bob, tail wag, preening, eye blink, and head-tilt cycles make the aviary feel alive, while Bézier perch hops prevent teleports.

- **Starting pair from a 6-species pool.** NOT RECOVERABLE FROM PLAN

- **Capacity cap strictly at 7 birds.** NOT RECOVERABLE FROM PLAN

- **Additional bird adoption gated solely on aviary age.** NOT RECOVERABLE FROM PLAN

- **Passive presence tracking.** The rationale is to count attention only when valid and to prevent "Unattended open tabs inflating presence-time." Presence is validated against rate limits, with a maximum of 60 seconds accrued per 60 seconds.

- **Procedural return-greeting.** NOT RECOVERABLE FROM PLAN

- **Listen-in mix focusing on an individual bird.** The mix raises the focused bird while keeping unfocused birds "gently audible in ambient background; never muted," then smoothly returns to "equal ambient balance."

- **Top-bar gesture offers: Seed, Song fragment, Still pool of water.** Offers become accepted interaction events that feed drift calculations and nudge mood transitions "to content/curious."

- **Per-bird offer cooldowns.** NOT RECOVERABLE FROM PLAN

- **Settle gesture.** NOT RECOVERABLE FROM PLAN

- **5s cancel/undo affordance for settle.** NOT RECOVERABLE FROM PLAN

- **Field Notebook.** The notebook exists to preserve "auto-generated naturalist prose observations" around anchor events such as a first greeting swap, weather passage, or long quiet session. The plan also says it "Preserves sparsity" by avoiding entries within 48 hours unless a rare observation threshold is crossed.

- **Single-user accounts authenticated via email magic links.** NOT RECOVERABLE FROM PLAN

- **15-minute validity and single-use magic links.** NOT RECOVERABLE FROM PLAN

- **Multi-device state via single canonical server-simulated aviary state.** The rationale is convergence and conflict prevention: laptop and mobile read the same snapshot cache, no client-to-client sync is attempted, and the server tick is the sole author of mood, placement, and personality state.

- **Device session management and revocation.** NOT RECOVERABLE FROM PLAN

- **Quiet Social Visits.** The rationale is read-only sharing without social-network mechanics: one-time, revocable visitor access has "no co-presence, no chat, no visitor drift impact."

- **Naturalist screen-reader running narration.** It provides naturalist prose updates every 30-60 seconds and immediate narration for gestures, using `aria-live="polite"`.

- **Procedural call captions.** Captions surface calls as text near birds and are synchronized with WebAudio envelope triggers; they also become the automatic fallback when AudioContext is blocked or unavailable.

- **Reduced-motion cross-fade mode.** The mode replaces skeletal continuous keyframes with "gentle 1.2-second alpha cross-fades," deactivates parallax and particles, and extends lighting transitions.

- **Full keyboard navigation and WCAG AA contrast.** Keyboard navigation lets Tab, arrows, Enter, and Escape operate top-bar actions, bird nodes, Listen-In, and focus clearing; focus rings meet WCAG AA across daytime and night palettes.

- **JSON aviary snapshot export.** The plan places this under "Data Rights & Export," but does not articulate a more specific rationale. NOT RECOVERABLE FROM PLAN

- **30-day soft deletion before permanent purge.** The rollout names "GDPR/30-day soft purge," grounding the feature in data-rights/privacy work.

**1.2 Explicit Non-Goals & Architectural Prohibitions**

- **No Native Applications.** NOT RECOVERABLE FROM PLAN

- **No Gamification Elements.** The plan frames this as part of the ambient product boundary: zero streak counters, visit tallies, activity grids, badges, achievements, levels, XP, or score counters.

- **No Tamagotchi / Custodial Mechanics.** The rationale is explicit in the prohibition and drift rule: no illness, hunger, death, distress animations, or negative drift on neglect; traits "never decrement upon neglect."

- **No Social Network Features.** The rationale appears in the social boundary: quiet visits allow read-only access, while the plan prohibits a public directory/feed, follower model, leaderboards, mutual visit co-presence, comments, and avatars.

- **No Announcement UI in Aviary.** NOT RECOVERABLE FROM PLAN

- **No Numerical Trait Exposure.** The rationale is to keep personality scalars hidden from UI, debug overlays, and telemetry; vectors are "Server Canonical Only" and blocked from analytics pipelines.

**2. Architecture & Service Topology**

- **Stateless edge/API layer.** It handles magic-link auth, session validation, PII-isolating synthetic UUID translation, snapshot cache, and rate limiting, while cached snapshots "reduce database read load."

- **Centralized authoritative simulation daemon.** It runs the 60s server tick, consumes append-only events, performs monotonic drift filtering, advances mood transitions, and auto-generates notebook logs as the authoritative state writer.

- **Lightweight client-side procedural synthesis/render engine.** The client renders interpolated snapshots and generates procedural audio locally while keeping the static bundle under the stated size budget.

- **Client static bundle below 2MB gzipped.** The client responsibility says the static bundle must stay under 2MB; the audio section makes the why more specific by avoiding looped samples to fit the bundle constraint.

- **Client State & Presence Coordinator.** It joins rendering, audio, and interaction events so the browser can submit presence pings, offers, listen-in, and settle events while interpolating server snapshots smoothly.

- **API Gateway synthetic UUID translation.** The rationale is named directly as "PII isolation": authenticated session tokens are converted to internal Account UUIDs.

- **Snapshot cache.** The plan says it serves cached aviary snapshots directly "to reduce database read load."

- **Append-only interaction event log.** The rationale is delta ingestion: clients append events to a transaction log, and the worker consumes unprocessed events during ticks.

- **Data and telemetry boundary.** The rationale is privacy protection: email is encrypted and never logged, operational telemetry is decoupled from simulation tables, and per-bird vectors and interaction logs are blocked from analytics.

**3. Data Model**

- **Accounts with encrypted email and blind email hash.** The rationale is encrypted email at rest plus a blind index "for login lookup."

- **Auth sessions.** NOT RECOVERABLE FROM PLAN

- **Aviaries.** NOT RECOVERABLE FROM PLAN

- **Birds.** NOT RECOVERABLE FROM PLAN

- **Personality vectors.** The plan marks them as "Hidden Personality Vectors (Server Canonical Only)," supporting hidden trait scalars and server-authored behavior/visual drift.

- **Interaction events.** The rationale is append-only presence and interaction ingestion, with unprocessed events indexed for the simulation tick.

- **Notebook entries.** The rationale is storage of "Field Notebook Observations" as dated prose text, ordered by aviary and observed date.

- **Visit invitations and visit logs.** They support one-time, expiring, revocable quiet visits and visit history.

**4. API Surface & Protocols**

- **HTTPS REST endpoints returning structured JSON.** NOT RECOVERABLE FROM PLAN

- **Auth magic-link endpoints.** NOT RECOVERABLE FROM PLAN

- **Active login session list and revocation endpoints.** NOT RECOVERABLE FROM PLAN

- **Aviary snapshot endpoint.** The snapshot carries the "current canonical aviary state" needed for multi-device convergence, interpolation, resumed tabs, and read-only visitor access.

- **Batched client event submission.** The rationale is delta-based event ingestion: clients submit presence, offer, listen-in, and settle events instead of absolute state.

- **Read-only Field Notebook API.** The rationale is paginated access to observation logs while keeping notebook entries generated by the simulation.

- **Social visit invite, log, revoke, and visitor snapshot APIs.** These implement quiet social visits with read-only snapshots and rejection when links are revoked or expired.

**5. Simulation Engine Design**

- **60-second backend tick loop.** The tick loop provides asynchronous authoritative progression for active aviaries and continues even when browser rendering halts.

- **Event ingestion.** It pulls unprocessed interaction events since the last tick so drift and mood changes come from append-only event deltas.

- **Presence validation.** The rationale is anti-spoofing and background-tab control: presence pings are rate-limited to real-world time bounds.

- **Monotonic low-pass drift filter.** The plan states the why directly: traits never decrement upon neglect, drift should be measurable after 7 days, and visible behavioral/visual drift should appear after 21 days.

- **Drift calibration tests.** The risk section says tests are needed because traits drifting too quickly can feel "like a toy," while drifting too slowly can feel "static."

- **Mood State Machine & Circadian Modulation.** Mood transitions create state variation conditioned on time of day, accepted offers, boldness, and weather.

- **Naturalist Notebook Generation.** The rationale is sparse, state-matched observations: entries occur every 2-4 days or on anchor events, using bird names and lower-case present-tense grammar.

- **Notebook sparsity rule.** The plan says it preserves sparsity by not emitting entries within 48 hours unless a rare observation threshold is crossed.

**6. Sync Model & Conflict Prevention**

- **Single Writer Principle.** The rationale is conflict prevention: the server-side simulation tick is the sole author of personality vectors, bird placements, and mood states.

- **Delta-Based Event Ingestion.** The plan says clients submit discrete events and "never absolute state," preventing clients from overwriting canonical state.

- **Multi-Device Convergence.** Laptop and mobile read the same snapshot cache, and a resumed tab interpolates to canonical server state over 1.5 seconds.

- **No client-to-client peer sync.** The rationale is convergence around canonical snapshots rather than peer state exchange.

- **Hidden/minimized tab rendering halt.** The plan states the rationale as conserving battery and CPU.

- **Backend simulation continues for offline/background tabs.** The rationale is that the aviary appears to have "continued uninterrupted" when the tab refocuses.

**7. Frontend Rendering Pipeline**

- **Canvas / 2D Context viewport sizing.** The rationale is to keep all birds in frame across mobile and wide displays.

- **Back, Middle, and Front depth planes.** These provide scene depth: soft sky and distant trees in back, main branches and bird rigs in middle, and foreground foliage parallax in front.

- **Procedural skeleton/rig.** The rationale is micro-motion: body bob, tail wag, preening, blink, and head tilt.

- **Bézier perch hops.** The rationale is explicit: velocity curves are calculated "to prevent teleports."

- **Instant Aliveness / Zero Entry Transition.** First frame rendering, mid-cycle bird placement, and no spinners or fade-ins make the aviary feel immediately alive.

- **Cold network empty sky gradient.** The rationale is to remain calm during cold requests before starter birds glide to perches.

- **Reduced-motion automatic activation.** The rationale is accessibility via `prefers-reduced-motion` or a manual toggle.

- **Reduced-motion cross-fades, no parallax/particles, extended lighting fades.** These reduce continuous movement while preserving naturalist bird poses and day-to-dusk state.

**8. Audio Pipeline & WebAudio Synthesis**

- **No looped audio samples.** The plan states two reasons: fit the bundle constraint and eliminate repetition.

- **Species motif grammars.** The rationale is varied procedural calls built from structural synthesizer graphs and 1-4 pitch/frequency sweeps.

- **Micro-randomized jitter in frequency and tempo.** This supports organic variation; the risk mitigation also names "organic randomized pitch offsets" as protection against artificial-sounding audio.

- **Spatial positioning by perch zone.** The rationale is audio correspondence to visual depth: front birds are dry/full stereo, back birds are centered and low-pass filtered.

- **Ambient chorus dynamic gain bus.** The plan says it prevents clipping and applies ducking when multiple birds call.

- **Listen-In focus gain ramp.** The rationale is focused listening without cutting off the aviary: focused birds rise to +3dB, unfocused birds remain gently audible, and disengaging returns to equal ambient balance.

- **WebAudio fallback to silence.** The rationale is graceful failure when AudioContext is blocked, unavailable, or denied, with call captions automatically activated.

**9. Accessibility Surfaces**

- **Screen-reader narration live region.** The rationale is naturalist prose access to the aviary state and immediate narration for user gestures, throttled through `aria-live="polite"`.

- **30-second idle narration throttle.** The risk section gives the why: prevent frequent state updates from "clobbering screen-reader queues."

- **Procedural call captioning near birds.** The rationale is text access to calls, synchronized precisely with WebAudio envelope triggers.

- **Keyboard navigation and focus traps.** The rationale is full keyboard operation through top-bar actions, aviary bird nodes, Listen-In, and clearing Listen-In.

- **High-contrast focus rings and WCAG AA contrast.** The rationale is readable focus visibility across daytime and night palettes.

**10. Performance Budgets & Observability**

- **Initial JS bundle budget.** The rationale is tied to the lightweight client and audio choice: procedural synthesis avoids samples so the bundle stays small.

- **Time to First Bird Visible.** The rationale is instant aliveness: a bird should be visible within the stated budget from the edge CDN.

- **Frame-rate budget.** The rationale is smooth rendering on a 2021 mid-tier laptop and an explicit 30fps low-power mobile mode.

- **Flat memory profile and zero per-frame garbage collector pressure.** The rationale is stability over a 30-minute ambient session; WebAudio buffer nodes and canvas frame buffers are recycled.

- **Simulation tick duration metrics.** The rationale is operational alerting when p99 tick latency exceeds 5 seconds.

- **API latency and error-rate metrics.** NOT RECOVERABLE FROM PLAN

- **Synthetic client lighthouse probes.** NOT RECOVERABLE FROM PLAN

- **Analytics prohibitions on bird names, vectors, offer choices, and notebook logs.** The rationale is privacy enforcement.

- **Bucketed anonymized session duration histograms.** The rationale is privacy-preserving operational measurement.

**11. Rollout & Milestone Plan**

- **Phase 1: Core Simulation & DB Engine.** NOT RECOVERABLE FROM PLAN

- **Phase 2: Procedural Audio & WebAudio Synthesis.** NOT RECOVERABLE FROM PLAN

- **Phase 3: Canvas Rendering & Micro-Motion.** NOT RECOVERABLE FROM PLAN

- **Phase 4: Interactions, Notebook & Gestures.** NOT RECOVERABLE FROM PLAN

- **Phase 5: Social Visits & Privacy Layer.** NOT RECOVERABLE FROM PLAN

- **Phase 6: Accessibility, Perf Audit & Launch.** NOT RECOVERABLE FROM PLAN

**12. Risks & Mitigations**

- **Drift calibration simulation unit tests.** The rationale is to avoid traits drifting too quickly and feeling "like a toy" or drifting too slowly and feeling static.

- **Formant filter modulation and chorus ducking bus.** The rationale is to mitigate "Audio Uncanniness / Phase Cancellation" from layered procedural calls.

- **Strict client-side presence conjunction check.** The rationale is to mitigate "Presence Signal Spoofing / Background Tab Drift."

- **Strict 30-second narration throttle and `aria-live="polite"`.** The rationale is to mitigate "Screen-Reader Queue Flooding."
