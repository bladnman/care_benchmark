## System-level intent

1. Slow-burn, observational relationship. The plan opens by defining Pocket Aviary v1 as "focused on a slow-burn, observational relationship" and carries that through "procedural bird personality drift," "Notice, never announce" return greetings, micro-motion such as preening/scanning, and drift that takes "weeks for visible effect."

2. Expressive change without pressure or punishment. The plan pairs "monotonic toward expressive" with explicit non-goals: "No Gamification," "No Custodial Pressure," no birds dying, starving, or showing distress, and no "maintenance" chores. The drift constraint says neglect gives "zero delta, never negative."

3. Quiet naturalist product voice. The Field Notebook is a "naturalist observation log," screen-reader narration uses "naturalist prose," and call captions describe sound in observational language such as "a low trill, repeated." The product voice is descriptive rather than celebratory or achievement-driven.

4. Server-authoritative continuity. The server is the "authoritative simulation engine," the "sole writer of personality state," and the source of snapshots. This supports "No-Last-Write-Wins," additive events from multiple devices, and the "aviary continued without you" effect.

5. Procedural, living-feeling birds instead of static loops. The plan emphasizes procedural personality drift, rules-based call grammar, WebAudio synthesis, micro-motion, parallax, interpolation, timing variation, and pitch variation. The risks section specifically guards against "robotic" audio and "Screensaver" pacing.

6. Bounded privacy and bounded social presence. The plan uses a synthetic AccountID "to protect PII," encrypted email, opt-in "one-to-one read-only visit invitations," "No Social Network," and analytics that avoid tracking "specific per-bird interactions in aggregate."

7. First-class accessibility and performance as product constraints. Accessibility is included in v1 with "first-class screen-reader narration," reduced motion, call captions, and keyboard control. Performance has explicit budgets for bundle size, "Time to First Bird," memory leaks, and stable 60fps.

## Per-feature whys

### 1. Scope and v1 Definition

- Core Engine: The plan's rationale is to support the "slow-burn, observational relationship" through "procedural bird personality drift (monotonic toward expressive)," moods, and procedural calls that make birds change slowly and become more expressive.

- "Notice, never announce" return greetings: The rationale is embedded in the phrase itself and the overall observational intent: the return greeting should be noticed quietly rather than announced as a reward or event.

- Listen-in: The plan explains this through "mix re-balancing"; later audio details say the focused bird gains +6dB while others drop to -12dB as an "ambient wash" over 2-second ramps.

- Offer gesture: The plan makes offers part of the interaction events consumed by the simulation tick. In the drift formula, interaction frequency contributes through `InteractionWeight * Frequency`.

- Settle gesture: NOT RECOVERABLE FROM PLAN

- Single horizontal aviary scene: The plan later says Canvas/WebGL is used "to handle 7 birds with micro-motion and parallax at 60fps," giving the scene a performance and rendering rationale.

- Field Notebook: The plan calls it a "naturalist observation log," connecting it to the quiet naturalist product voice and later storing it in the relational data store.

- Top-bar chrome: NOT RECOVERABLE FROM PLAN

- Account/Settings: NOT RECOVERABLE FROM PLAN

- Email magic-link auth: NOT RECOVERABLE FROM PLAN

- Server-side simulation tick: The rationale is to process recent events, update personality vectors and moods, and write new snapshots from the authoritative server.

- Multi-device sync with additive server-authored personality deltas: The plan's rationale is that clients send events, the server computes new state, and multiple devices can contribute without last-write-wins conflicts.

- Screen-reader narration: The plan frames this as "first-class" accessibility and later specifies naturalist prose delivered as live-region updates.

- Reduced-motion mode: The rationale is accessibility; the plan replaces continuous motion with "2-3 second cross-fades between static poses."

- Call captions: The rationale is accessibility for vocalizations, using text descriptions such as "a low trill, repeated" near the bird when vocalizing.

- One-to-one read-only visit invitations: The rationale is bounded social presence: visits are "opt-in only" and fit the explicit non-goal of "No Social Network."

### 2. Architecture

- Client (React/TypeScript): The client is kept as a "thin rendering and audio synthesis layer" so it handles interpolation and procedural grammars while the server remains authoritative.

- Server (Node.js/TypeScript): The server is the "authoritative simulation engine" and "sole writer of personality state," which supports consistent drift and sync.

- Relational (PostgreSQL): NOT RECOVERABLE FROM PLAN

- Event Log (Append-only): The rationale is that interaction events such as presence, offers, and listen-ins are consumed by the simulation tick.

- Cache (Redis): The plan says Redis holds "canonical aviary state snapshots for fast client delivery."

- Client Snapshot Pull: Pulling snapshots on visibility change and a low-frequency heartbeat keeps clients aligned with server state without making the client authoritative.

- Interpolation: The rationale is explicit: interpolate from "start" to "target" state "to maintain 60fps without teleports."

### 3. Data Model

- AccountID: The plan says this is a synthetic UUID "to protect PII."

- Email: The plan gives a privacy/storage rationale through "Encrypted, stored once."

- BirdID: The rationale is stable bird identity; the plan specifies a "Stable UUID."

- SpeciesID: The rationale is to refer to the species pool for "visuals" and "base grammar."

- Name: NOT RECOVERABLE FROM PLAN

- PersonalityVector: The rationale is to hold hidden scalars that drive drift and expressive traits: Boldness, SocialWarmth, VocalFrequency, PlumageSaturation, and Curiosity.

- CurrentMood: The rationale is that moods are computed by the tick and modulate calls, with states such as Wary, Content, Curious, Drowsy, and Alert.

- DriftHistory: The plan says it records "accumulated presence-time and interaction weights," which are inputs to drift calculation.

- PresenceEvent: The rationale is to log AccountID, Duration, and Timestamp so presence can be consumed by the simulation tick and contribute to PresenceWeight.

- InteractionEvent: The rationale is to log Offer and ListenIn events with Subtype and Timestamp so recent interactions can affect mood and drift.

### 4. Simulation Engine & Drift Function

- Mood Transition: The plan computes next mood from TimeOfDay, ambient weather, and recent interaction events, grounding mood in environment and recent behavior.

- Drift Calculation: The rationale is slow visible change: presence and interactions add positive deltas, a low-pass function ensures "weeks for visible effect," and the monotonic constraint prevents negative outcomes from neglect.

- Procedural Call Grammar: The rationale is species-specific and state-sensitive calls: rules-based motifs are stored per species, with pitch and interval modulated by VocalFrequency and mood.

### 5. Frontend Rendering & Audio Pipeline

- Canvas/WebGL: The plan uses it for the horizontal aviary scene "to handle 7 birds with micro-motion and parallax at 60fps."

- Micro-motion Engine: The rationale is continuous procedural preening/scanning logic, supporting an observational scene that keeps birds subtly alive.

- Reduced Motion rendering: The rationale is to provide a toggle from continuous rendering into "2-3 second cross-fades between static poses."

- Procedural Synthesis: The plan says WebAudio generates motifs "rather than playing loops," making bird calls procedural instead of looped assets.

- Ambient mix: The rationale is a "balanced chorus of all birds."

- Listen-In Focus: The rationale is focused listening: one bird gains +6dB while others drop into an "ambient wash" over 2-second ramps.

- Chorus Logic: The plan says real-time timing variation avoids phase-canceling or "robotic" synchronization.

### 6. Sync & Conflict Model

- Additive Deltas: The rationale is that "clients never send state; they send events," and the server computes new state.

- No-Last-Write-Wins: The plan explains that since only the server writes personality, multiple devices only contribute to the event log.

- Snapshot Continuity: The rationale is the "aviary continued without you" effect when a client returns from long sleep and pulls the current server-tick state.

### 7. Accessibility Design

- Narration Engine: The rationale is screen-reader access through server-side "naturalist prose" delivered as a live-region update every 45s.

- Call Captions: The rationale is text access to bird vocalizations, displayed near the bird when vocalizing.

- Keyboard Control: The rationale is keyboard accessibility through tab-accessible birds, Enter for Listen-In, and Space for Offer.

### 8. Performance & Observability

- JS Bundle budget: The rationale is keeping the gzipped bundle under 2MB, with "aggressive code splitting for settings/auth."

- Time to First Bird: The rationale is fast first experience on 4G, with a critical path of "HTML -> Initial Snapshot -> Render Shell."

- Runtime budget: The rationale is stable long sessions: no memory leaks over 30 minutes and stable 60fps on 2021-era hardware.

- Operational observability: The rationale is to track sync-tick latency, bundle size, error rates for magic-link delivery, and general operational health.

- Privacy Constraint: The rationale is privacy-preserving analytics: no aggregate tracking of specific per-bird interactions, only anonymized session duration and general health metrics.

### 9. Rollout Strategy

- Internal Alpha: The rationale is to verify drift calibration and audio "chorus" feel.

- Limited Beta: The rationale is to test magic-link and sync stability across device types.

- Public v1 caps: NOT RECOVERABLE FROM PLAN

- Instrumentation: The rationale is to focus on "Time to First Bird" and "Simulation Tick Health."

### 10. Risks & Mitigations

- Drift Calibration testing targets: The rationale is avoiding both "Too fast = Tamagotchi" and "Too slow = Screensaver," using "instruments-visible" and "user-visible" testing targets.

- Audio jitter and bird-ID pitch variation: The rationale is mitigating procedural calls that sound "robotic."

- Server-authoritative sync timestamps: The rationale is mitigating Server/Client clock skew by making all logic server-authoritative and having the client interpolate between server-provided timestamps.

- Narration prose generated from the same data model as rendering: The rationale is preventing visual changes from breaking narration prose.
