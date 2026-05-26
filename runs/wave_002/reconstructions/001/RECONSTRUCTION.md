## System-level intent

- **Low-fidelity, high-affect relationship through idle attention and minimal interaction.** The plan states the core goal as fostering "a relationship between a user and a small set of virtual birds" through "idle attention and minimal interaction." This shows up in the small 2 to 7 bird scope, the restrained interaction set, and the emphasis on "presence-time" as the dominant drift input.
- **The aviary continues without the viewer.** The Architecture section names the "Canonical Server / Interpolating Client" model and says it exists to ensure the aviary "continues without the viewer." This also appears in "Server: Owns all state," the server-side tick, canonical snapshots, and multi-device sync.
- **No punishment, no counters, no gamification.** The plan rejects "streaks, achievements, levels, or counters," rejects "hunger, death, or distress," and says "Absence is not punished." The simulation carries the same principle through "Monotonic toward expressive (no negative drift on neglect)."
- **Server-owned truth with append-only user influence.** The plan gives the server the only writer role for personality vectors, while the client submits append-only events. This philosophy shows up in "No Conflict Resolution," "no 'write conflicts' on personality," and additive deltas that prevent "last-write-wins" data loss.
- **Naturalist voice instead of exposed state labels.** The accessibility and notebook surfaces use "Naturalist prose." Screen-reader narration is explicitly "descriptive prose" instead of "state labels," and the risk section names a "Naturalist Voice" rubric to keep narration from becoming "list-like."
- **Procedural, ambient liveliness over recorded assets or fixed loops.** The plan calls for WebAudio motifs, "No audio loops," mood-shaped idle micro-motion, ambient leaf and feather drift, and chorus mixing that creates a "natural chorus." The audio risk also favors "procedural variation over loops."
- **Fast, private, accessible first contact.** The performance budgets stress a bundle under 2MB, "Time-to-First-Bird" under 500ms, 60fps idle motion, and zero memory growth. The observability section adds a "Privacy Boundary" where no per-bird interaction data enters analytics, while accessibility includes screen-reader prose, reduced motion, and call captions.

## Per-feature whys

### 1. Scope

- **Core Experience: 2 to 7 birds in a single horizontal scene.** The plan ties this to "a small set of virtual birds" so the relationship remains focused. The rollout also treats the cap as something to increase only as "audio-mix recognizability is verified."
- **Server-side simulation tick.** The tick supports the architecture goal that the aviary "continues without the viewer" and makes the simulation service the "heart" that processes events, mood, and personality transitions.
- **Personality vectors.** Personality vectors carry the gradual relationship change: the simulation computes "personality/mood transitions," uses personality vector modifiers in mood updates, and calibrates drift to be "measurable at 1 week, visible at 3 weeks."
- **Mood system.** Mood is the plan's bridge between state and presentation: it is updated from local time, recent events, personality, and weather, then modulates animation, pitch, timing, and idle micro-motion.
- **Monotonic drift based on presence.** The rationale is the no-punishment principle: drift is "monotonic toward expressive," with "no negative drift on neglect," and "presence-time" is the dominant input because the experience is based on idle attention.
- **Return-greetings.** The rollout calls out validating the "feel" of the return-greeting in Internal Alpha, tying it to the relationship moment when the user returns.
- **Listen-in.** Listen-in gives focused attention to one bird: the audio mix uses a "linear ramp for focused bird volume" while other ambient volume is attenuated but not muted, and listen-in duration feeds drift.
- **Offers.** Offers are one of the minimal interactions that can shape drift; "offer success" is an input to the low-pass drift calculation.
- **Specific offer types: seeds, song fragments, still pools.** NOT RECOVERABLE FROM PLAN
- **Settle gesture.** NOT RECOVERABLE FROM PLAN
- **Single-user accounts.** Accounts support canonical server state and multi-device sync for one user's aviary.
- **Email magic links.** NOT RECOVERABLE FROM PLAN
- **Multi-device sync.** The plan's reason is canonical state across devices: the server is the "single source of truth," clients are read-only for state, and additive deltas prevent "last-write-wins" data loss.
- **Field Notebook.** NOT RECOVERABLE FROM PLAN
- **Opt-in, read-only visit invitations via email.** The plan allows a narrow social surface while preserving the non-goal of "No Social Networking": invitations are opt-in, visitor access is read-only, and there is no public discovery, profiles, follows, or co-presence.
- **Naturalist screen-reader narration.** The rationale is accessibility without turning the aviary into raw state labels: narration uses "Naturalist Prose" and descriptive prose "instead of state labels."
- **Reduced-motion rendering mode.** The plan provides this for accessibility via `prefers-reduced-motion` or settings, replacing frame-by-frame animation with slow cross-fades, removing leaf drift, and slowing color shifts.
- **Procedural call captioning.** Captions make procedural calls accessible by generating text such as "a soft three-note rise" and positioning it near the calling bird; the WebAudio fallback enables captions by default.
- **Optimized initial bundle and rapid first-paint.** The bundle budget exists "to ensure fast load and allow procedural audio over recorded," and the 500ms first-bird target is specified for 4G.

### 2. Architecture

- **Canonical Server / Interpolating Client model.** The stated reason is to ensure the aviary "continues without the viewer."
- **Auth Service.** NOT RECOVERABLE FROM PLAN
- **Simulation Service.** It is the "heart" of the product because it runs the server-side tick, manages the event log, and computes personality and mood transitions.
- **State API.** The State API gives clients "small, read-only snapshots" so the client can remain a rendering shell rather than a state owner.
- **Event API.** The append-only endpoint lets clients submit presence pings, offers, and other events without creating direct state writes or personality conflicts.
- **Account Service synthetic UUID mapping.** The plan states the reason directly: synthetic UUID mapping exists "to avoid PII leaks."
- **Server-owned state and personality writes.** The server owns all state and is the only writer of personality vectors so there are no write conflicts on personality.
- **Client rendering shell.** The client only pulls snapshots, interpolates motion/audio, and synthesizes WebAudio; this preserves canonical server truth while keeping presentation local.

### 3. Data Model

- **Stable internal bird UUID.** NOT RECOVERABLE FROM PLAN
- **User-assigned bird name.** NOT RECOVERABLE FROM PLAN
- **Personality vector scalar values.** The scalar values are server-side only because the server computes drift and is the only writer of personality vectors.
- **Mood enum.** Mood gives the simulation an enumerated state that can drive transitions, animation state, pitch, timing, and idle micro-motion.
- **Current perch.** It supports the scene snapshot and animated perch changes between front, middle, and back positions.
- **Event Log.** The event log is consumed by the tick, preserves event order, and supports additive deltas across devices.
- **Field Notebook entry content as Naturalist prose.** The rationale present in the plan is voice consistency: notebook entries use the same "Naturalist prose" vocabulary that also governs accessible narration.

### 4. API Surface

- **GET `/api/state`.** It returns the current snapshot the rendering client needs: birds, environment, mood, perch, position, animation state, time of day, weather, and settled state.
- **POST `/api/events`.** It submits one or more user events into the append-only event log so the simulation tick can process presence, offers, listen-in, and related interactions.
- **POST `/api/visits/invite`.** The invite endpoint supports the opt-in email visit flow without public discovery.
- **GET `/api/visits/snapshot?token=...`.** The snapshot endpoint gives a visitor read-only access through a one-time link.

### 5. Simulation Engine Design

- **Tick cadence of about 60 seconds.** NOT RECOVERABLE FROM PLAN
- **Process Event Log.** The tick consumes all events since the last tick so user attention and interactions affect the next canonical state.
- **Low-pass filter drift.** The low-pass filter supports gradual calibration; the risk section warns against drift being "too fast (gamified)" or "too slow (static)."
- **Presence-time as dominant drift input.** This directly matches the core goal of relationship through "idle attention."
- **Listen-in duration and offer success as drift inputs.** These let the minimal interaction set affect personality transitions without direct client writes.
- **Monotonic drift constraint.** It implements "Absence is not punished" by allowing drift only "toward expressive" and with "no negative drift on neglect."
- **Mood updates from local time, recent interactions, personality, and weather.** These inputs make moods depend on time of day, interaction history, personality vector modifiers, and ambient weather events.
- **Notebook entries generated if thresholds are met.** NOT RECOVERABLE FROM PLAN
- **Motif Library.** Species-specific motifs provide the source material for procedural calls.
- **Client-side WebAudio synthesis.** The plan uses WebAudio so pitch and timing can be modulated by mood and vocal frequency, and so the product can avoid audio loops.
- **Chorus Logic.** Independent mixing avoids phase-canceling and creates a "natural chorus."

### 6. Sync Model

- **Canonical State.** The server is the "single source of truth" so all devices see the same aviary state.
- **No Conflict Resolution.** The plan says conflict resolution is unnecessary because clients are read-only for state and append-only for events.
- **Additive Deltas.** Additive changes are applied in event log order to prevent "last-write-wins" data loss across devices.

### 7. Frontend Rendering Pipeline

- **Scene layering: background, middle, foreground.** NOT RECOVERABLE FROM PLAN
- **Snapshot interpolation.** Interpolation exists to "smoothly transition bird positions between state snapshots."
- **Quiet field loading state.** Rendering starts immediately with a "quiet field" if the snapshot is pending, supporting rapid first contact.
- **First frame with motion in progress.** The plan states that the first frame "must show motion in progress," aligning loading with the feeling of a living aviary.
- **Idle micro-motion.** Continuous preening and scanning are shaped by mood, giving each bird ongoing visible life while idle.
- **Ambient leaf and feather drift.** NOT RECOVERABLE FROM PLAN
- **Perch transition paths.** Perch changes are animated paths to make state changes visible and smooth.
- **Reduced-motion cross-fades for transitions.** Cross-fades preserve the same state change while honoring reduced-motion mode.
- **Top Bar fading to transparent.** The reason given is "Minimalist chrome"; the bar fades after inactivity so the scene remains primary.

### 8. Audio Pipeline

- **Procedural synthesis.** The plan specifies procedural WebAudio and "No audio loops"; the performance section also says the bundle budget allows procedural audio over recorded audio.
- **Listen-In Mix.** The focused bird gets a linear volume ramp while other birds are attenuated but "not muted," preserving both focus and ambient chorus.
- **WebAudio fallback.** If WebAudio is unavailable, the aviary is silent and captions are enabled by default, preserving accessibility.

### 9. Accessibility Surfaces

- **Screen-reader narration as Naturalist Prose.** It uses descriptive prose rather than state labels, and the risk section says audits should prevent narration from becoming "list-like."
- **Narration pacing.** Updates every 30 to 60 seconds, with priority for user-initiated events, so narration is paced and responsive.
- **Reduced-motion trigger.** The mode is triggered by `prefers-reduced-motion` or settings, respecting both platform preference and explicit user choice.
- **Reduced-motion rendering.** It replaces frame-by-frame animation with slow cross-fades between static poses.
- **Reduced-motion ambient changes.** It removes leaf drift and slows color shifts to reduce motion load.
- **Call caption text.** Captions are generated from procedural calls, such as "a soft three-note rise."
- **Call caption placement.** Captions are positioned near the calling bird so the caption stays tied to the source.

### 10. Performance and Observability

- **Bundle size under 2MB gzipped.** The plan states the reason: "to ensure fast load and allow procedural audio over recorded."
- **Time-to-First-Bird under 500ms on 4G.** This budget supports rapid first-paint and a fast first encounter with the aviary.
- **60fps idle motion on 5-year-old hardware.** This keeps continuous idle motion viable on older devices.
- **Zero memory growth over 30-minute sessions.** The plan says this is "strictly enforced in CI," making long idle sessions safe.
- **Aggregate telemetry.** Request counts, latencies, and render timings support observability without per-bird interaction analytics.
- **Privacy Boundary.** The rationale is explicit: "No per-bird interaction data ever enters the analytics pipeline."
- **p99 simulation-tick latency alarm over 5 seconds.** The alarm watches the latency of the server-side simulation tick, a core part of the product's "heart."

### 11. Rollout

- **Internal Alpha.** The reason is to validate the "feel" of the return-greeting and procedural calls.
- **Calibration Phase.** The reason is to tune drift until it meets the "measurable at 1 week, visible at 3 weeks" target.
- **v1 Launch instrumentation.** The plan instruments "time-to-first-bird" and audio-context errors at launch.
- **Scaling birds-per-aviary cap.** The cap increases gradually up to 7 as "audio-mix recognizability is verified."

### 12. Risks and Mitigations

- **Heavy instrumentation of personality vectors in test harness.** This mitigates drift that is too fast and feels "gamified" or too slow and feels "static."
- **Iterative motif design.** This mitigates procedural calls sounding "robotic."
- **Strict adherence to procedural variation over loops.** This also mitigates audio uncanniness by keeping calls varied rather than looped.
- **Strict append-only event log and server-side-only personality writes.** This mitigates rare race conditions in event log processing.
- **Regular audits using the Naturalist Voice rubric.** This mitigates accessibility regressions where narrations become "list-like."
