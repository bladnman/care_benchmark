# Implementation Plan: Pocket Aviary v1

## 1. Scope
Pocket Aviary v1 is a browser-based, single-user virtual aviary focused on long-term observational relationships.

### In-Scope
- **Core Loop**: Presence-based interaction (idle watching), Listen-in (focusing), Offer (gestures), and Settle (session end).
- **Bird Engine**: Personality vectors (slow drift), Mood (fast state), Procedural call grammar, and Mood-shaped idle motion.
- **Environment**: Single horizontal scene with three perch zones, day/night cycle (local time), and ambient weather.
- **Accounts & Sync**: Magic-link email auth, single canonical aviary per account, and multi-device sync via server-side truth.
- **Social**: Opt-in, read-only, ambient "Visit" feature (no co-presence).
- **Field Notebook**: Auto-generated, naturalist-voice observation log.
- **Accessibility**: Screen-reader narration (prose-based), reduced-motion mode (cross-fade animation), and call captioning.
- **Performance**: <2MB JS bundle, <500ms time-to-first-bird, 60fps idle motion.

### Out-of-Scope (Non-Goals)
- **Gamification**: No streaks, scores, levels, achievements, or badges.
- **Tamagotchi Mechanics**: No mortality, hunger, or distress; drift is monotonic toward expressive (no punishment for neglect).
- **Social Network**: No profiles, follows, discovery feeds, or public profiles.
- **Native Apps**: Web-only for v1.

## 2. Architecture

### Service Shape
- **Client**: Modern web application (React/TypeScript) responsible for rendering (WebAudio, Canvas/WebGL), interaction capture, and interpolation of state snapshots.
- **Server**: Node.js/TypeScript service managing the simulation engine, persistence, and authentication.

### Client/Server Split
- **Server is the Authority**: The server owns the canonical state, the simulation tick, and the personality/mood computation.
- **Client is a Projector**: The client pulls snapshots and "plays" the state. It never writes personality or mood directly.
- **Interaction Flow**: Clients append interaction events (Presence, Offer, Listen-in, Settle) to an append-only log. The server simulation tick consumes this log to update the state.

### Render Pipeline Boundary
- The client handles all visual and audio synthesis. The server sends purely numerical and categorical state data (positions, mood enums, call motifs).

## 3. Data Model

### Account
- `account_id`: Synthetic UUID (primary key).
- `email`: Encrypted string (stored only once).
- `settings`: JSON object (accessibility, social opt-ins, etc.).

### Bird
- `bird_id`: Stable UUID.
- `species_id`: Reference to species pool.
- `name`: User-assigned string.
- `personality_vector`: `{ boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity }` (floats).
- `current_mood`: Enum `{ wary, content, curious, drowsy, alert }`.
- `last_mood_timestamp`: Timestamp for daily reset/modulation.

### Aviary
- `aviary_id`: Linked to `account_id`.
- `bird_list`: Array of `bird_id`s.
- `presence_history`: Aggregated presence-time for drift calculations.
- `notebook_entries`: Array of `{ timestamp, prose_text }`.

### Interaction Events (Append-only Log)
- `event_type`: `{ presence_ping, offer_given, listen_in_start, listen_in_end, settle }`.
- `payload`: Contextual data (e.g., which bird was listened to, which offer was made).
- `timestamp`: Server-side arrival time.

## 4. API Surface

### Client $\to$ Server
- `POST /auth/magic-link`: Request login.
- `POST /interactions`: Append event to the log.
- `POST /account/settings`: Update user preferences.
- `POST /account/export`: Request JSON snapshot via email.
- `POST /social/invite`: Send email invitation to a visitor.
- `DELETE /social/invite/{email}`: Revoke an invitation.

### Server $\to$ Client (State Snapshots)
- `GET /aviary/snapshot`: Returns current birds, positions, moods, call motifs, weather, and time-of-day.

## 5. Simulation Engine Design

### The Server-Side Tick
- **Cadence**: ~1 minute.
- **Process**:
  1. Fetch latest canonical state.
  2. Read and consume all new interaction events from the log since last tick.
  3. **Drift Calculation**: Apply low-pass filter to personality vectors based on presence-time and interaction weights.
  4. **Mood Transition**: Update moods based on current time, recent interactions, ambient weather, and personality.
  5. **Notebook Generation**: Randomly trigger an observation entry based on significant state shifts.
  6. Persist new state and clear/archive processed events.

### Drift Function
- **Formula**: $P_{new} = P_{old} + \text{clamp}(\text{signal} \times \text{weight}, 0, \text{max\_delta})$
- **Monotonicity**: The $\text{clamp}$ ensures that negative signals (neglect) result in zero change, preventing "un-learning" of traits.

### Call-Grammar Runtime
- The server defines a "motif" per bird. The client synthesizes the actual audio using WebAudio, varying pitch and timing based on the bird's `vocal_frequency` and `mood`.

## 6. Sync Model

### Single Canonical Truth
- All clients (phone, laptop) pull from the same `account_id` record. 
- Because the server is the sole writer of personality/mood, there is no "merging" of states. A client simply sees the result of the last server tick.

### Conflict Prevention
- **No Last-Write-Wins**: Personality is updated via additive deltas calculated on the server. Clients cannot send absolute personality values.
- **Event Ordering**: The interaction log is processed sequentially by the server tick, ensuring deterministic state advancement.

## 7. Frontend Rendering Pipeline

### Visual Composition
- **Scene**: 2D/2.5D horizontal scene using a canvas-based engine.
- **Perch Zones**: Three distinct depth layers (front, middle, back) for spatial grouping.
- **Micro-Motion**: Continuous, mood-driven idle animations (preening, scanning, fluffing).
- **Transitions**: Smooth interpolation between state snapshots for bird movement and lighting shifts.
- **Reduced-Motion Mode**: Replaces frame-by-frame animation with slow, alpha-blended cross-fades between key poses.

### Audio Pipeline
- **Synthesis**: WebAudio-based procedural synthesis of call motifs.
- **Mixing**: 
  - **Ambient Mix**: Default state; all birds contribute to a background chorus.
  - **Listen-in Mix**: User focuses a bird; that bird's gain increases while others' gains decrease toward a baseline "ambient" floor.
- **Fallback**: If WebAudio is denied, the system enters "Graceful Silence" with mandatory call captions.

### Accessibility Surfaces
- **Narration**: A separate audio/textual stream of naturalist prose updates (one per 30-60s).
- **Captions**: Small, localized text overlays near birds describing call characteristics (e.g., "a soft three-note rise").
- **Keyboard**: Full focus management (Tab $\to$ Birds $\to$ Listen-in).

## 8. Performance Budgets and Observability

### Budgets
- **Bundle Size**: Initial JS < 2MB (gzipped).
- **LCP (First Bird Visible)**: < 500ms on 4G mid-tier mobile.
- **Frame Rate**: Consistent 60fps for idle motion.
- **Memory**: Zero growth over a 30-minute session (active pool management).

### Observability
- **Synthetic Monitoring**: Automated browser checks for load times and render-frame stability.
- **RUM (Real User Monitoring)**: Aggregate-only metrics (latency, error rates, session duration). **No PII or per-bird data.**
- **Alerting**: P99 simulation-tick latency > 5s.

## 9. Rollout

1. **Alpha (Internal)**: Test drift calibration and audio recognizability with a small set of "seed" birds.
2. **Beta (Limited Invite)**: Test sync stability and magic-link flow with a wider group.
3. **v1 Launch**: Public web release with two birds per account and the full suite of features.
4. **Instrumentation**: Day-one monitoring of simulation-tick latencies and client-side render-frame timing.

## 10. Risks

| Risk | Mitigation |
| :--- | :--- |
| **Drift Calibration** | Rigorous automated testing of the drift function against simulated interaction logs to ensure the 1-week/3-week targets are met. |
| **Sync Divergence** | Strictly enforce that the client is a "read-only" projector of personality/mood state. |
| **Audio Uncanniness** | Prioritize procedural synthesis over loops; use small motif libraries to ensure variety. |
| **Accessibility Regressions** | Treat accessibility surfaces (narration/captions) as core features in the CI/CD pipeline, not add-ons. |
| **Performance Bloat** | Aggressive code-splitting and asset optimization; strict bundle size monitoring in build steps.
| **Privacy Leaks** | Enforce synthetic UUIDs for all internal references; ensure telemetry pipelines are physically decoupled from the simulation DB. |
