# Implementation Plan: Pocket Aviary (v1)

## 1. Scope
Pocket Aviary is a low-fidelity, high-affect browser-based experience. The core goal is to foster a relationship between a user and a small set of virtual birds through idle attention and minimal interaction.

### In-Scope (v1)
- **Core Experience:** 2 to 7 birds in a single horizontal scene.
- **Engine:** Server-side simulation tick, personality vectors, mood system, and monotonic drift based on presence.
- **Interactions:** Return-greetings, Listen-in, Offers (seeds, song fragments, still pools), and Settle gesture.
- **Systems:** Single-user accounts via email magic links, multi-device sync (canonical server state), and the Field Notebook.
- **Social:** Opt-in, read-only visit invitations via email.
- **Accessibility:** Naturalist screen-reader narration, reduced-motion rendering mode, and procedural call captioning.
- **Performance:** Optimized initial bundle (<2MB) and rapid first-paint (<500ms).

### Out-of-Scope (Non-Goals)
- **No Gamification:** No streaks, achievements, levels, or counters of any kind.
- **No Tamagotchi Mechanics:** No hunger, death, or distress. Absence is not punished.
- **No Social Networking:** No public discovery, profiles, follows, or co-presence.
- **No Native Apps:** Web-only for v1.

## 2. Architecture
The system follows a **Canonical Server / Interpolating Client** model to ensure the aviary "continues without the viewer."

### Service Shape
- **Auth Service:** Handles magic link generation, verification, and session token issuance.
- **Simulation Service:** The "heart" of the product. Runs the server-side tick, manages the event log, and computes personality/mood transitions.
- **State API:** Provides small, read-only snapshots of the current aviary state to clients.
- **Event API:** Append-only endpoint for clients to submit interaction events (presence pings, offers, etc.).
- **Account Service:** Manages user profiles and synthetic UUID mapping to avoid PII leaks.

### Client/Server Split
- **Server:** Owns all state. The only writer of personality vectors. Computes the simulation tick.
- **Client:** A rendering shell. Pulls snapshots and interpolates motion/audio. Synthesizes audio procedurally via WebAudio.

## 3. Data Model

### Account
- `account_id` (Synthetic UUID)
- `email` (Encrypted)
- `created_at` (Timestamp)
- `settings` (Accessibility preferences, visit notifications)

### Bird
- `bird_id` (Stable internal UUID)
- `species_id` (Reference to species pool)
- `name` (User-assigned string)
- **Personality Vector:** (Scalar values, server-side only)
    - `boldness`
    - `social_warmth`
    - `vocal_frequency`
    - `plumage_saturation`
    - `curiosity`
- **Mood:** (Enumerated state: `wary`, `content`, `curious`, `drowsy`, `alert`)
- `current_perch` (Front, Middle, Back)

### Event Log
- `event_id` (UUID)
- `account_id` (FK)
- `event_type` (e.g., `presence_ping`, `offer_accepted`, `listen_in_start`)
- `timestamp` (UTC)
- `metadata` (JSON: bird_id, offer_type, duration)

### Field Notebook
- `entry_id` (UUID)
- `account_id` (FK)
- `content` (Naturalist prose)
- `timestamp` (UTC)

## 4. API Surface

### State Retrieval
- `GET /api/state`: Returns the current snapshot.
    - Payload: `{ birds: [{ id, species, mood, perch, pos, animation_state }], environment: { time_of_day, weather, is_settled } }`

### Event Submission
- `POST /api/events`: Submits one or more events.
    - Payload: `[{ type: 'presence_ping', duration: 60 }, { type: 'offer', bird_id: '...', item: 'seed' }]`

### Visit Flow
- `POST /api/visits/invite`: Host sends email $\rightarrow$ Server generates one-time link.
- `GET /api/visits/snapshot?token=...`: Visitor retrieves read-only snapshot.

## 5. Simulation Engine Design

### Server-Side Tick
Runs every $\sim 60$ seconds:
1. **Process Event Log:** Consume all events since the last tick.
2. **Calculate Drift:** Update personality vectors using a low-pass filter.
    - Input: `presence-time` (dominant), `listen-in` duration, `offer` success.
    - Constraint: Monotonic toward expressive (no negative drift on neglect).
3. **Update Moods:** Transition mood based on:
    - Local time of day.
    - Recent interaction events.
    - Personality vector modifiers.
    - Ambient weather events.
4. **Commit State:** Save new canonical bird states and generate notebook entries if thresholds are met.

### Call Grammar Runtime
- **Motif Library:** Each species has a set of procedural audio motifs.
- **Synthesis:** Client-side WebAudio. Pitch and timing are modulated by `mood` and `vocal_frequency`.
- **Chorus Logic:** Multiple birds calling simultaneously are mixed independently to avoid phase-canceling, creating a natural chorus.

## 6. Sync Model
- **Canonical State:** The server is the single source of truth.
- **No Conflict Resolution:** Since clients are read-only for state and append-only for events, there are no "write conflicts" on personality.
- **Additive Deltas:** The simulation tick applies additive changes to personality vectors based on the event log order, preventing "last-write-wins" data loss across devices.

## 7. Frontend Rendering Pipeline

### Scene Composition
- **Layering:** Background (sky/foliage) $\rightarrow$ Middle (perches/birds) $\rightarrow$ Foreground (occasional leaf/branch).
- **Interpolation:** Smoothly transition bird positions between state snapshots.
- **Loading:** Start rendering immediately with a "quiet field" if the snapshot is pending. First frame must show motion in progress.

### Motion
- **Idle Micro-motion:** Continuous per-bird animations (preening, scanning) shaped by `mood`.
- **Ambient Motion:** Client-side generated leaf and feather drift.
- **Transitions:** Perch changes are animated paths (or cross-fades in reduced-motion mode).

### Top Bar
- Minimalist chrome. Fades to transparent after a few seconds of inactivity.

## 8. Audio Pipeline
- **Procedural Synthesis:** Uses WebAudio to generate calls from motifs. No audio loops.
- **Listen-In Mix:** Linear ramp for focused bird volume; attenuated (but not muted) ambient volume for others.
- **Fallback:** If WebAudio is unavailable, the aviary is silent with captions enabled by default.

## 9. Accessibility Surfaces

### Screen-Reader Narration
- **Naturalist Prose:** Server/Client generates descriptive prose (e.g., "a small grey bird is perched on the front rail...") instead of state labels.
- **Pacing:** Updates every 30–60 seconds, with priority for user-initiated events.

### Reduced-Motion Mode
- Triggered by `prefers-reduced-motion` or settings.
- **Rendering:** Replaces frame-by-frame animation with slow cross-fades between static poses.
- **Ambient:** Removes leaf drift; slows color shifts.

### Call Captioning
- Runtime generation of captions based on the procedural call (e.g., "a soft three-note rise").
- Positioned near the calling bird.

## 10. Performance and Observability

### Budgets
- **Bundle Size:** $<2\text{MB}$ (gzipped) to ensure fast load and allow procedural audio over recorded.
- **Time-to-First-Bird:** $<500\text{ms}$ on 4G.
- **Runtime:** 60fps idle motion on 5-year-old hardware.
- **Memory:** Zero growth over 30-minute sessions (strictly enforced in CI).

### Observability
- **Aggregate Telemetry:** Request counts, latencies, and render timings.
- **Privacy Boundary:** No per-bird interaction data ever enters the analytics pipeline.
- **Alarms:** p99 simulation-tick latency $> 5\text{s}$.

## 11. Rollout
1. **Internal Alpha:** Validate the "feel" of the return-greeting and procedural calls.
2. **Calibration Phase:** Tune the drift function to meet the "measurable at 1 week, visible at 3 weeks" target.
3. **v1 Launch:** Open to small cohorts. Instrument "time-to-first-bird" and audio-context errors.
4. **Scaling:** Gradually increase birds-per-aviary cap (up to 7) as audio-mix recognizability is verified.

## 12. Risks and Mitigations
- **Drift Calibration:** Risk of drift being too fast (gamified) or too slow (static). *Mitigation: Heavy instrumentation of personality vectors in test harness.*
- **Audio Uncanniness:** Procedural calls sounding "robotic." *Mitigation: Iterative motif design and strict adherence to procedural variation over loops.*
- **Sync Correctness:** Rare race conditions in event log processing. *Mitigation: Strict append-only event log and server-side-only personality writes.*
- **Accessibility Regressions:** Narrations becoming "list-like." *Mitigation: Regular audits using the "Naturalist Voice" rubric.*
