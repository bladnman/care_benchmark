## System-level intent

1. **Long-term observational relationships, not game progression.** The plan names the product as "a browser-based, single-user virtual aviary focused on long-term observational relationships." This intent shows up again in the Core Loop: "Presence-based interaction (idle watching), Listen-in (focusing), Offer (gestures), and Settle (session end)." It is reinforced by the non-goals: "No streaks, scores, levels, achievements, or badges."

2. **Non-punitive expressive drift.** The plan rejects "mortality, hunger, or distress" and says "drift is monotonic toward expressive (no punishment for neglect)." The Drift Function repeats the same principle: negative signals produce "zero change," preventing "'un-learning' of traits."

3. **Server-side canonical truth with a read-only client projector.** The architecture states "Server is the Authority" and "Client is a Projector." The Sync Model restates this as "Single Canonical Truth," "no 'merging' of states," and "No Last-Write-Wins."

4. **Interactions are append-only signals, not direct state edits.** The plan says clients append "Presence, Offer, Listen-in, Settle" events to an "append-only log." The server tick consumes that log, and clients "never write personality or mood directly." This protects personality and mood as computed simulation state.

5. **Ambient social, not a social network.** The Social scope is "opt-in, read-only, ambient 'Visit'" with "no co-presence." The non-goals exclude "profiles, follows, discovery feeds, or public profiles." The privacy risk mitigation adds "synthetic UUIDs" and telemetry decoupled from the simulation DB.

6. **Naturalist prose as part of the product voice.** The Field Notebook is an "auto-generated, naturalist-voice observation log." Accessibility narration is also "prose-based" and delivered as "naturalist prose updates," making prose observation a cross-cutting surface rather than only a log format.

7. **Accessibility surfaces are core features.** The plan includes "Screen-reader narration," "reduced-motion mode," and "call captioning" in scope. The risks section says to treat "accessibility surfaces (narration/captions) as core features in the CI/CD pipeline, not add-ons."

8. **Lightweight, immediate, smooth web experience.** The plan sets hard budgets: "<2MB JS bundle," "<500ms time-to-first-bird," and "60fps idle motion." Performance bloat is mitigated with "aggressive code-splitting and asset optimization" and "strict bundle size monitoring."

9. **Procedural, mood-shaped audiovisual expression.** The Bird Engine combines "Personality vectors," "Mood," "Procedural call grammar," and "Mood-shaped idle motion." The Audio Pipeline uses "WebAudio-based procedural synthesis," and the audio risk mitigation says to "prioritize procedural synthesis over loops" for variety and to address "Audio Uncanniness."

## Per-feature whys

### 1. Scope

#### In-Scope

- **Core Loop: Presence-based interaction (idle watching).** Why: The product is focused on "long-term observational relationships." Presence is also the signal used in "presence_history" and "presence-time" for drift calculations.

- **Core Loop: Listen-in (focusing).** Why: The plan defines Listen-in as focusing, and the Audio Pipeline explains that the focused bird's gain increases while the others decrease toward an "ambient" floor.

- **Core Loop: Offer (gestures).** NOT RECOVERABLE FROM PLAN

- **Core Loop: Settle (session end).** NOT RECOVERABLE FROM PLAN

- **Bird Engine: Personality vectors (slow drift).** Why: Personality vectors are the slow-changing expression layer. The server applies a low-pass filter based on "presence-time and interaction weights," with monotonic drift toward expressive traits and no punishment for neglect.

- **Bird Engine: Mood (fast state).** Why: Mood is the fast state that changes based on "current time, recent interactions, ambient weather, and personality." It shapes idle motion and call synthesis.

- **Bird Engine: Procedural call grammar.** Why: The server defines a call "motif" and the client synthesizes audio with WebAudio. The risks section says procedural synthesis over loops helps with "Audio Uncanniness" and "variety."

- **Bird Engine: Mood-shaped idle motion.** Why: Idle animations are "mood-driven" and include "preening, scanning, fluffing," so the birds' visible behavior expresses current mood.

- **Environment: Single horizontal scene.** NOT RECOVERABLE FROM PLAN

- **Environment: Three perch zones.** Why: The Frontend Rendering Pipeline says the three depth layers are for "spatial grouping."

- **Environment: Day/night cycle (local time).** Why: Mood transitions use "current time," and the frontend interpolates "lighting shifts."

- **Environment: Ambient weather.** Why: Mood transitions use "ambient weather," and snapshots include "weather."

- **Accounts & Sync: Magic-link email auth.** NOT RECOVERABLE FROM PLAN

- **Accounts & Sync: Single canonical aviary per account.** Why: It supports "Single Canonical Truth": all clients pull from the same account record, with no merging of states.

- **Accounts & Sync: Multi-device sync via server-side truth.** Why: Since the server is the sole writer of personality and mood, a client "simply sees the result of the last server tick."

- **Social: Opt-in, read-only, ambient "Visit" feature (no co-presence).** Why: It allows ambient visiting while staying outside "Social Network" behavior: no profiles, follows, discovery feeds, public profiles, or co-presence.

- **Field Notebook: Auto-generated, naturalist-voice observation log.** Why: The server randomly triggers entries based on "significant state shifts," using naturalist prose as the observation voice.

- **Accessibility: Screen-reader narration (prose-based).** Why: Narration provides a separate audio/textual stream of naturalist prose updates, and accessibility surfaces are core features.

- **Accessibility: Reduced-motion mode (cross-fade animation).** Why: It replaces frame-by-frame animation with "slow, alpha-blended cross-fades between key poses."

- **Accessibility: Call captioning.** Why: Captions describe call characteristics near birds, and if WebAudio is denied the system enters "Graceful Silence" with mandatory call captions.

- **Performance: <2MB JS bundle.** Why: The plan treats bundle size as a budget and mitigates "Performance Bloat" with code-splitting, asset optimization, and bundle size monitoring.

- **Performance: <500ms time-to-first-bird.** Why: The LCP budget is "First Bird Visible" in under 500ms on "4G mid-tier mobile."

- **Performance: 60fps idle motion.** Why: The experience depends on "consistent 60fps for idle motion" and render-frame stability.

#### Out-of-Scope (Non-Goals)

- **Gamification: No streaks, scores, levels, achievements, or badges.** Why: This protects the intent of "long-term observational relationships" from game progression mechanics.

- **Tamagotchi Mechanics: No mortality, hunger, or distress.** Why: The plan wants "no punishment for neglect" and monotonic drift toward expressive traits.

- **Social Network: No profiles, follows, discovery feeds, or public profiles.** Why: This keeps Social to opt-in, read-only, ambient Visit rather than a public network.

- **Native Apps: Web-only for v1.** NOT RECOVERABLE FROM PLAN

### 2. Architecture

- **Client: Modern web application (React/TypeScript).** Why: The client is responsible for rendering with WebAudio and Canvas/WebGL, interaction capture, and interpolation of state snapshots.

- **Server: Node.js/TypeScript service.** Why: The server manages the simulation engine, persistence, and authentication.

- **Server is the Authority.** Why: It owns canonical state, the simulation tick, and personality/mood computation.

- **Client is a Projector.** Why: It pulls snapshots and "plays" state; it never writes personality or mood directly.

- **Interaction Flow: append-only log.** Why: Clients append interactions, and the server simulation tick consumes the log to update state.

- **Render Pipeline Boundary.** Why: The server sends only numerical and categorical state data, while the client handles all visual and audio synthesis.

### 3. Data Model

#### Account

- **account_id: Synthetic UUID.** Why: The privacy mitigation says to enforce synthetic UUIDs for all internal references.

- **email: Encrypted string (stored only once).** Why: The plan stores email in encrypted form and limits it to one stored copy, aligning with the privacy-leak mitigation.

- **settings: JSON object.** Why: Settings hold "accessibility, social opt-ins, etc.," which are user preferences exposed by the API.

#### Bird

- **bird_id: Stable UUID.** NOT RECOVERABLE FROM PLAN

- **species_id: Reference to species pool.** NOT RECOVERABLE FROM PLAN

- **name: User-assigned string.** NOT RECOVERABLE FROM PLAN

- **personality_vector.** Why: Personality is the slow-drift state used for boldness, social warmth, vocal frequency, plumage saturation, and curiosity.

- **current_mood.** Why: Mood is the fast state used by motion, call synthesis, and mood transitions.

- **last_mood_timestamp.** Why: It supports "daily reset/modulation."

#### Aviary

- **aviary_id linked to account_id.** Why: This implements the single canonical aviary per account.

- **bird_list.** NOT RECOVERABLE FROM PLAN

- **presence_history.** Why: Presence history is the aggregated presence-time used for drift calculations.

- **notebook_entries.** Why: Notebook entries store the generated observation log prose.

#### Interaction Events (Append-only Log)

- **event_type.** Why: Event type records presence pings, offers, Listen-in starts and ends, and Settle events for the server tick to consume.

- **payload.** Why: Payload supplies contextual data such as "which bird was listened to" or "which offer was made."

- **timestamp: Server-side arrival time.** Why: Server-side timestamps support event ordering and deterministic state advancement.

### 4. API Surface

#### Client to Server

- **POST /auth/magic-link.** NOT RECOVERABLE FROM PLAN

- **POST /interactions.** Why: It appends events to the interaction log.

- **POST /account/settings.** Why: It updates user preferences, including accessibility settings and social opt-ins.

- **POST /account/export.** NOT RECOVERABLE FROM PLAN

- **POST /social/invite.** Why: It supports opt-in Visit through email invitation.

- **DELETE /social/invite/{email}.** Why: It revokes an invitation, preserving opt-in control.

#### Server to Client (State Snapshots)

- **GET /aviary/snapshot.** Why: The client pulls the snapshot to project current birds, positions, moods, call motifs, weather, and time-of-day.

### 5. Simulation Engine Design

#### The Server-Side Tick

- **Cadence: ~1 minute.** NOT RECOVERABLE FROM PLAN

- **Fetch latest canonical state.** Why: The server owns canonical state.

- **Read and consume all new interaction events from the log since last tick.** Why: Interaction events are the input signals for updating state.

- **Drift Calculation.** Why: It applies a low-pass filter to personality vectors using presence-time and interaction weights.

- **Mood Transition.** Why: Moods update based on current time, recent interactions, ambient weather, and personality.

- **Notebook Generation.** Why: An observation entry can be triggered by significant state shifts.

- **Persist new state and clear/archive processed events.** Why: The tick advances canonical state and handles processed log entries.

#### Drift Function

- **Formula with clamp.** Why: The clamp makes negative signals produce zero change, preventing "'un-learning' of traits."

- **Monotonicity.** Why: It encodes "no punishment for neglect" by making drift monotonic toward expressive.

#### Call-Grammar Runtime

- **Server-defined motif per bird.** Why: The server sends motifs while the client synthesizes audio.

- **Client WebAudio synthesis with pitch and timing variation.** Why: Pitch and timing vary based on vocal_frequency and mood, supporting procedural variety.

### 6. Sync Model

- **Single Canonical Truth.** Why: Phone and laptop pull from the same account record.

- **Server sole writer of personality/mood.** Why: There is no merging of states.

- **Client sees the result of the last server tick.** Why: The client does not reconcile state; it projects server truth.

- **No Last-Write-Wins.** Why: Personality is updated through server-calculated additive deltas rather than absolute client values.

- **Clients cannot send absolute personality values.** Why: This prevents clients from overwriting personality state.

- **Event Ordering.** Why: Sequential processing by the server tick ensures deterministic state advancement.

### 7. Frontend Rendering Pipeline

#### Visual Composition

- **2D/2.5D horizontal scene using a canvas-based engine.** NOT RECOVERABLE FROM PLAN

- **Perch Zones: three distinct depth layers.** Why: They provide spatial grouping.

- **Micro-Motion.** Why: Continuous mood-driven idle animations express preening, scanning, and fluffing.

- **Transitions.** Why: Smooth interpolation covers bird movement and lighting shifts between state snapshots.

- **Reduced-Motion Mode.** Why: It replaces frame-by-frame animation with slow alpha-blended cross-fades between key poses.

#### Audio Pipeline

- **WebAudio-based procedural synthesis of call motifs.** Why: The plan prioritizes procedural synthesis over loops for audio recognizability, variety, and avoiding uncanniness.

- **Ambient Mix.** Why: It is the default state where all birds contribute to a background chorus.

- **Listen-in Mix.** Why: It lets the user focus a bird by raising that bird's gain while lowering others toward an ambient floor.

- **Graceful Silence.** Why: If WebAudio is denied, the system still exposes calls through mandatory captions.

#### Accessibility Surfaces

- **Narration.** Why: It provides a separate audio/textual stream of naturalist prose updates every 30-60s.

- **Captions.** Why: Localized overlays near birds describe call characteristics such as "a soft three-note rise."

- **Keyboard.** Why: Full focus management supports Tab to Birds to Listen-in.

### 8. Performance Budgets and Observability

#### Budgets

- **Bundle Size: Initial JS < 2MB (gzipped).** Why: This enforces the performance budget and guards against performance bloat.

- **LCP (First Bird Visible): < 500ms on 4G mid-tier mobile.** Why: The first bird should be visible quickly on mid-tier mobile.

- **Frame Rate: Consistent 60fps for idle motion.** Why: Idle motion depends on stable render-frame timing.

- **Memory: Zero growth over a 30-minute session.** Why: The plan calls for active pool management to prevent growth during a session.

#### Observability

- **Synthetic Monitoring.** Why: Automated browser checks watch load times and render-frame stability.

- **RUM (Real User Monitoring).** Why: Aggregate-only metrics capture latency, error rates, and session duration with "No PII or per-bird data."

- **Alerting: P99 simulation-tick latency > 5s.** Why: Simulation-tick latency is important enough to page when P99 exceeds 5 seconds.

### 9. Rollout

- **Alpha (Internal).** Why: It tests drift calibration and audio recognizability with a small set of "seed" birds.

- **Beta (Limited Invite).** Why: It tests sync stability and magic-link flow with a wider group.

- **v1 Launch.** Why: It is the public web release with two birds per account and the full suite of features.

- **Instrumentation.** Why: It monitors simulation-tick latencies and client-side render-frame timing on day one.

### 10. Risks

- **Drift Calibration.** Why: Automated testing of the drift function is needed to ensure the 1-week/3-week targets are met.

- **Sync Divergence.** Why: The mitigation is to strictly enforce the client as a read-only projector of personality/mood state.

- **Audio Uncanniness.** Why: Procedural synthesis over loops and small motif libraries provide variety.

- **Accessibility Regressions.** Why: Narration and captions must be core CI/CD features, not add-ons.

- **Performance Bloat.** Why: Aggressive code-splitting, asset optimization, and bundle size monitoring control bloat.

- **Privacy Leaks.** Why: Synthetic UUIDs and telemetry pipelines physically decoupled from the simulation DB reduce privacy risk.
