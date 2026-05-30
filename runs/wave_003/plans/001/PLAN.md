# Pocket Aviary v1 Implementation Plan

## 1. Scope
Pocket Aviary v1 is a browser-based, single-user virtual aviary focused on observational relationship-building. 

### In-Scope
- **Core Loop**: Procedural bird behavior (mood, idle motion, calls) responding to presence and specific interactions (listen-in, offer, settle).
- **Bird Engine**: Personality drift (monotonic toward expressive), mood cycles (fast-timescale), and procedural call grammar.
- **User Experience**: Naturalist field notebook, return-greeting, single horizontal scene with three perch zones.
- **Account & Sync**: Magic-link authentication, single canonical aviary per account, multi-device synchronization via server-side simulation.
- **Social (Opt-in)**: Read-only ambient visits via email invitation.
- **Accessibility**: Naturalist prose narration for screen readers, reduced-motion mode (cross-fade posing), and call captioning.
- **Performance**: <2MB initial bundle, <500ms time-to-first-bird, 60fps idle motion.

### Out-of-Scope (Non-Goals)
- **Gamification**: No streaks, levels, achievements, or scores.
- **Tamagotchi Mechanics**: No death, hunger, or distress; neglect only leads to reduced expressiveness.
- **Social Network Surfaces**: No profiles, follows, public discovery, or chat.
- **Native Apps**: Web-only for v1.
- **Co-presence**: Visitors cannot interact or be seen by the host.

## 2. Architecture

### System Overview
A client-server model where the server is the sole authority for state and simulation.
- **Client**: A lightweight web application responsible for rendering the scene, synthesizing audio, and capturing user presence/interaction signals.
- **Simulation Service (Server)**: An authoritative engine that executes the "tick" at a ~1-minute cadence. It consumes an append-only event log and produces updated state snapshots.
- **API/Communication**: Clients pull lightweight state snapshots and push interaction events to an event log.

### Service Shape
- **Auth Service**: Handles email magic-link generation and verification.
- **Simulation Service**: Manages the aviary state, personality vectors, mood transitions, and the drift function.
- **Event Log Service**: An append-only store for user interactions (presence pings, offers, etc.).
- **Snapshot Service**: Serves the current canonical aviary state to clients.
- **Social/Invite Service**: Manages one-time visit links and revocation.

## 3. Data Model

### Account Entity
- `account_uuid` (Primary Key, Synthetic UUID)
- `email` (Encrypted)
- `settings` (Accessibility, Social opt-in, etc.)

### Aviary Entity (Owned by Account)
- `aviary_id` (UUID)
- `account_uuid` (FK)
- `bird_list` (List of Bird entities)
- `notebook_entries` (List of generated observations)
- `last_tick_timestamp`

### Bird Entity
- `bird_id` (Stable UUID)
- `species_id` (Reference to species pool)
- `name` (User-assigned string)
- `personality_vector`:
  - `boldness` (Scalar)
  - `social_warmth` (Scalar)
  - `vocal_frequency` (Scalar)
  - `plumage_saturation` (Scalar)
  - `curiosity` (Scalar)
- `current_mood` (Enum: wary, content, curious, drowsy, alert)
- `last_mood_update` (Timestamp)

### Event Log Entry
- `event_id` (UUID)
- `account_uuid` (FK)
- `bird_id` (Optional FK)
- `event_type` (presence, listen-in_start, listen-in_end, offer, settle)
- `timestamp`

## 4. API Surface

### Client-to-Server (Events)
- `POST /events`: Appends an interaction event to the log.
  - Payload: `{ event_type, bird_id?, metadata? }`

### Server-to-Client (State)
- `GET /snapshot`: Returns the current aviary state.
  - Payload: `{ birds: [...], mood_ambient, time_of_day, weather, notebook_recent }`

### Social/Visit Flow
- `POST /invites`: Host initiates an invite by email.
- `GET /visit/{token}`: Visitor pulls a read-only, non-interactive snapshot.
- `DELETE /invites/{email}`: Host revokes an invitation.

## 5. Simulation Engine Design

### The Tick (~1 minute cadence)
The tick is the only process allowed to mutate the aviary's canonical state.
1. **Fetch Log**: Retrieve all new events since the last tick.
2. **Presence Processing**: Calculate total presence-time from presence-pings.
3. **Drift Computation**: Apply the low-pass filter to personality vectors based on presence and interaction weights (Presence > Listen-in > Offers). Drift is monotonic toward expressive.
4. **Mood Transition**: Update bird moods based on:
   - Recent interactions.
   - Local time-of-day (from server clock).
   - Ambient weather/events.
   - Personality-driven probabilities.
5. **Notebook Generation**: Occasionally generate a new naturalist observation entry based on notable recent events.
6. **State Commit**: Save the new canonical state and clear processed events.

### Drift Calibration
- **Target**: Measurable numerical change in ~1 week; visible user-perceived change in ~3 weeks.
- **Constraint**: Drift must be monotonic. Neglect does not lower traits; it simply fails to raise them.

## 6. Sync Model

### Single Source of Truth
The server-side simulation is the only authority.
- **No Client Ownership**: Clients never send absolute values (e.g., `set_boldness(0.6)`). They only send signals (e.g., `presence_event`).
- **Additive Deltas**: The simulation engine computes deltas and applies them to the canonical vector, preventing "last-write-wins" conflicts between devices.
- **Snapshot Interpolation**: To ensure smoothness, clients fetch snapshots and use local interpolation (e.g., for bird position and pitch) to bridge the gap between ticks.

## 7. Frontend Rendering Pipeline

### Scene Composition
- **Visual Layer**: HTML5 Canvas or WebGL for the horizontal scene.
- **Perch Zones**: Three distinct Z-depth layers (front, middle, back).
- **Atmosphere**: Shader-based day/night transitions and ambient weather (rain/wind) effects.
- **Micro-motion**: Continuous idle loops (preening, scanning, head-tilting) that are mood-keyed.

### Reduced-Motion Mode
- **Mechanism**: Replaces frame-by-frame animations with slow, aesthetic cross-fades between distinct poses.
- **Visuals**: Removes ambient leaf/feather drift; maintains ambient lighting shifts.

### Audio Pipeline
- **Engine**: WebAudio API.
- **Procedural Synthesis**: Calls are generated via motif libraries and real-time pitch/timing modulation.
- **The Chorus**: Real-time mixing of multiple procedural streams.
- **Listen-in Mix**: A gradual gain-ramp for the focused bird's stream while attenuating others.
- **Fallback**: If WebAudio fails, the system enters "graceful silence" with enabled call captions.

## 8. Accessibility Surfaces

### Screen-Reader Narration
- **Implementation**: A hidden live region (`aria-live="polite"`) receiving naturalist prose updates.
- **Cadence**: ~30–60s idle cadence; immediate priority for user-initiated events (return-greeting, offer, settle).
- **Voice**: Lowercase, present-tense, naturalist observations (e.g., "a small grey bird is perched on the front rail...").

### Call Captioning
- **Implementation**: Small, fading text overlays near the calling bird.
- **Content**: Short, mood-driven prose descriptions (e.g., "a soft three-note rise").

### Keyboard Navigation
- **Focus Management**: Tabbed access to top bar; arrow keys to navigate bird focus in the aviary.
- **Interactions**: `Enter` for listen-in; `Escape` to exit.

## 9. Performance Budgets and Observability

### Budgets
- **JS Bundle**: <2MB (gzipped).
- **Time to First Bird**: <500ms on mid-tier mobile/4G.
- **Runtime**: 60fps on 5-year-old hardware.
- **Memory**: Zero growth over a 30-minute session.

### Observability
- **Synthetic Monitoring**: Automated browser fleets testing load and render timings globally.
- **RUM (Real User Monitoring)**: Aggregate-only metrics (latencies, error rates, frame drops).
- **Privacy Boundary**: No per-user or per-bird data is sent to analytics.

## 10. Rollout
- **Phase 1**: Internal alpha for stability of the simulation tick and drift calibration.
- **Phase 2**: Limited beta to calibrate the "feels alive" threshold (presence/drift balance).
- **Phase 3**: Public v1 launch.

## 11. Risks
- **Drift Calibration**: If drift is too fast, it feels like a game; if too slow, it feels like a screensaver.
- **Sync Correctness**: Incorrect delta application could lead to lost personality history.
- **Audio Uncanniness**: Non-procedural artifacts or poor WebAudio implementation breaking the "aliveness" spell.
- **Accessibility Regression**: Treating accessibility as a "fallback" rather than a designed surface.
