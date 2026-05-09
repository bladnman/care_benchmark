## System-level intent

- **Long-term observational relationships over short-term gamified engagement.** This is the core identity: "focused on long-term observational relationships rather than short-term gamified engagement." It also appears in the explicit "No Gamification" non-goal and in the drift risk: "If too fast, it's a game; if too slow, it's a screensaver."

- **A gentle sanctuary with no punishment for absence.** The plan rejects "hunger, death, or punishment for absence," includes "Settle (soft-exit gesture)," and uses a "Quiet Field" placeholder "instead of a spinner." These choices carry a soft, non-punitive product voice.

- **Server-canonical, event-log-based simulation.** The system shape says the "Server acts as the canonical simulation engine" and the "Client acts as a stateless rendering and interaction-capture surface." The sync model repeats this as "One Source of Truth" and "Clients never send absolute state values."

- **Procedural expressiveness shaped by personality.** The Bird Engine includes "procedural call grammar," "personality drift (monotonic toward expressive)," and mood systems. The audio pipeline uses "motif libraries" with "personality-shaped timing," while drift is "strictly additive/monotonic toward 1.0."

- **Naturalist prose as a cross-cutting interface.** The Field Notebook is a "Naturalist, auto-generated observation log." Notebook generation uses a "template-based naturalist prose generator," and screen-reader narration is updated with "naturalist prose snapshots."

- **Narrow, opt-in social presence rather than a social network.** Social is limited to "one-to-one read-only visit invitations (opt-in only)." The non-goals reject "public profiles, discovery feeds, or global discovery."

- **Accessible, calm, immediate experience.** Accessibility includes "screen-reader narration," "reduced-motion mode," "call captions," and keyboard navigation. Performance budgets emphasize "Time-to-First-Bird," 60fps, and a bundle size that is "critical for 'already in motion' feel."

## Per-feature whys

### Scope and Core Identity

- **Pocket Aviary as a browser-based, naturalist virtual bird sanctuary**: The rationale is to support "long-term observational relationships" instead of "short-term gamified engagement."

### Included in V1

- **Bird Engine**: Its procedural call grammar, monotonic personality drift, and mood systems support birds that become more "expressive" over time and can be observed across six species.

- **Return-greeting**: NOT RECOVERABLE FROM PLAN

- **Listen-in**: The plan later explains the mix rationale: gain nodes "elevate the focused bird while ducking others" with a gradual ramp.

- **Offer**: NOT RECOVERABLE FROM PLAN

- **Settle**: The rationale is directly stated as a "soft-exit gesture."

- **Aviary Scene**: Day/night cycles and ambient weather feed Mood Transition, while the single horizontal viewport and three planes create an observational sanctuary surface.

- **Account & Sync**: The rationale is multi-device continuity through "server-side simulation tick," "multi-device state synchronization," and "canonical snapshots."

- **Social**: The rationale is constrained sharing: "one-to-one," "read-only," and "opt-in only," while avoiding a public social network.

- **Field Notebook**: The rationale is to create a "naturalist, auto-generated observation log" from "noteworthy changes."

- **Accessibility**: The rationale is to provide alternate surfaces for the same experience through "screen-reader narration," "reduced-motion mode," and "call captions."

### Explicit Non-Goals

- **No Gamification**: The rationale is to avoid "short-term gamified engagement" and to prevent drift from feeling like "a game."

- **No Custodial Mechanics**: The rationale is explicitly "No hunger, death, or punishment for absence."

- **No Native Apps**: The plan frames Pocket Aviary as "browser-based" and V1 as "Web-only."

- **No Social Network**: The rationale is to avoid "public profiles, discovery feeds, or global discovery" in favor of narrow invitations.

### Architecture

- **Client-server architecture**: The rationale is that the "Server acts as the canonical simulation engine" while the "Client acts as a stateless rendering and interaction-capture surface."

- **React/TypeScript with Vanilla CSS for UI chrome**: NOT RECOVERABLE FROM PLAN

- **WebAudio for procedural calls**: The rationale is the "procedural call grammar" and call-grammar runtime.

- **Canvas/WebGL or SVG/DOM rendering pipeline**: NOT RECOVERABLE FROM PLAN

- **Node.js/TypeScript backend**: The rationale is to host the "Simulation Tick and Event Log."

- **Event Log**: The rationale is append-only interaction history so the server can "process these in order" and calculate presence and interaction weights.

- **State Store**: The rationale is account-level canonical aviary state for "Personality Vectors, current Mood, Notebook."

- **Magic-link session store**: The rationale given is "magic-link session management."

### Data Model

- **Account & Aviary model**: The rationale is to hold the account's canonical aviary state: birds, notebook entries, and visit invitations.

- **Encrypted Email**: The rationale is PII protection, reinforced by the "PII Leak" risk.

- **VisitInvites list**: The rationale is the one-to-one read-only visit invitation feature.

- **Bird Entity**: The rationale is to track species, name, personality vector, mood, and current perch for simulation and rendering.

- **Synthetic UUID account and bird identifiers**: The rationale is the "strict Synthetic UUID rule" used to mitigate "PII Leak."

- **Personality Vector**: The rationale is to provide scalar values for personality drift and "personality-shaped timing."

- **Mood State**: The rationale is Mood Transition based on "Time-of-Day, Weather, and recent Event Log triggers."

- **CurrentPerch**: The rationale is placement in the scene's front, middle, and back perches.

- **Name**: NOT RECOVERABLE FROM PLAN

### API Surface

- **POST /events**: The rationale is conflict avoidance: clients send events to the append-only log, not "absolute state values."

- **GET /state**: The rationale is to "retrieve current canonical snapshot."

- **POST /auth/magic-link**: The rationale is magic-link auth and sign-in link request.

- **POST /social/invite**: The rationale is to create the visit invitation used by the one-to-one read-only social feature.

- **Server-to-client snapshots**: The rationale is "local sync" through `AviaryState` plus `ServerTime`.

### Simulation Engine Design

- **Server-Side Tick cadence of about one minute**: NOT RECOVERABLE FROM PLAN

- **Process Event Log**: The rationale is to calculate `PresenceTime` and interaction weights before updating the simulation.

- **Personality Drift**: The rationale is monotonic movement "toward 1.0" and "toward expressive," with calibration needed so it is neither "a game" nor "a screensaver."

- **Mood Transition**: The rationale is to update mood from "Time-of-Day, Weather, and recent Event Log triggers."

- **Notebook Generation**: The rationale is to generate naturalist prose "if noteworthy changes occurred."

### Audio Pipeline: Call-Grammar Runtime

- **WebAudio API engine**: The rationale is runtime procedural call synthesis.

- **Motif libraries**: The rationale is to synthesize calls from "oscillator/noise configurations" sequenced with "personality-shaped timing."

- **Listen-in gain ramp**: The rationale is to "elevate the focused bird while ducking others" over 3-5 seconds.

### Frontend Rendering Pipeline

- **Scene Composition layers**: The rationale is three-plane visual depth: "Background (Atmospheric)," "Middle (Birds/Perches)," and "Foreground (Micro-parallax branches)."

- **Idle Motion**: NOT RECOVERABLE FROM PLAN

- **Reduced-Motion Mode**: The rationale is to override interpolation with an opacity cross-fade between static poses.

- **Quiet Field placeholder**: The rationale is to show a "soft sky gradient" instead of a spinner while waiting for the first snapshot.

### Sync Model

- **One Source of Truth**: The rationale is that the server is the only writer of `Personality Vector`.

- **Conflict Avoidance**: The rationale is that clients never send absolute values; all mutations are events processed in order.

- **Snapshot pulls on visibilityChange and heartbeat**: The rationale is to "re-align with the server tick."

### Accessibility Surfaces

- **Screen-Reader Narration**: The rationale is naturalist prose snapshots in a dedicated `aria-live` region every 30-60 seconds.

- **Call Captions**: The rationale is captions generated from the same motif library that drives audio.

- **Keyboard Navigation**: The rationale is keyboard access to cycle birds and listen-in with standard focus visibility.

### Performance Budgets

- **Bundle Size under 2MB gzipped**: The rationale is explicitly "critical for 'already in motion' feel."

- **Time-to-First-Bird under 500ms**: NOT RECOVERABLE FROM PLAN

- **Runtime at 60fps on 5-year-old hardware**: NOT RECOVERABLE FROM PLAN

- **Synthetic monitoring**: The rationale is observability for "Time to First Bird" and "Simulation Tick Latency."

### Rollout and Risks

- **Internal dogfooding with 2 birds max**: NOT RECOVERABLE FROM PLAN

- **Invitation-only beta**: The rationale is "to calibrate Drift."

- **V1 launch with Species Pool of 6 species**: NOT RECOVERABLE FROM PLAN

- **Drift Calibration mitigation**: The rationale is to avoid drift that is "too fast" and feels like "a game," or "too slow" and feels like "a screensaver," using beta instrumentation.

- **Audio Uncanniness mitigation**: The rationale is that procedural synthesis may fail to sound "natural," so professional audio motif design is needed.

- **Sync Lag mitigation**: The rationale is to prevent multi-device users from seeing "teleporting" birds through server-side tick dominance and client-side interpolation.

- **PII Leak mitigation**: The rationale is that email is used as ID, mitigated by the "strict Synthetic UUID rule."
