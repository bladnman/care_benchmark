# Implementation Plan: Pocket Aviary (v1)

## 1. Scope
Pocket Aviary is a browser-based virtual aviary focused on a low-key, observational relationship between a user and a small set of birds.

### In-Scope (v1)
- **Aviary Core**: Single horizontal scene with 2-7 birds.
- **Bird Engine**: Personality vectors, mood system, procedural call synthesis, and monotonic drift.
- **Interactions**: Return-greeting, Listen-in, Offer, Settle, and Field Notebook.
- **Accounts & Sync**: Magic-link auth, server-side simulation tick, and multi-device state synchronization.
- **Social**: Read-only, opt-in visit invitations.
- **Accessibility**: Naturalist screen-reader narration, reduced-motion mode, and call captioning.
- **Performance**: Strict bundle size (<2MB) and time-to-first-bird (<500ms) targets.

### Out-of-Scope (Non-Goals)
- **No Gamification**: No streaks, achievements, levels, or scores.
- **No Tamagotchi Mechanics**: No hunger, death, or punishment for neglect.
- **No Social Networking**: No profiles, follows, public discovery, or chat.
- **No Native Apps**: Web-only.

---

## 2. Architecture
A client-server model where the server owns the canonical state and simulation, and the client acts as a high-performance renderer.

### Service Shape
- **Frontend (Client)**: A Single Page Application (SPA) using a high-performance rendering loop (Canvas/WebGL) and WebAudio API.
- **Backend (Simulation Service)**: A stateful service managing account data and the simulation tick.
- **Auth Service**: Magic-link generation and session token management.
- **Database**: Persistent storage for accounts, bird personality vectors, and the append-only interaction event log.

### Render Pipeline Boundary
The client pulls state snapshots and interpolates motion. The server provides the "what" (positions, moods, traits), and the client determines the "how" (animation frames, audio synthesis).

---

## 3. Data Model

### Account
- `accountId`: UUID (Synthetic ID, non-PII).
- `email`: Encrypted string.
- `settings`: Accessibility preferences, visit notification toggle.

### Bird
- `birdId`: UUID.
- `speciesId`: Reference to species pool.
- `name`: User-assigned string.
- `personalityVector`:
    - `boldness`: Scalar (0.0 - 1.0)
    - `socialWarmth`: Scalar (0.0 - 1.0)
    - `vocalFrequency`: Scalar (0.0 - 1.0)
    - `plumageSaturation`: Scalar (0.0 - 1.0)
    - `curiosity`: Scalar (0.0 - 1.0)
- `currentMood`: Enum (`WARY`, `CONTENT`, `CURIOUS`, `DROWSY`, `ALERT`).
- `perchZone`: Enum (`FRONT`, `MIDDLE`, `BACK`).

### Presence & Interaction
- `presenceLog`: Timestamps of presence-events.
- `eventLog`: Append-only log of interactions (`OFFER_ACCEPTED`, `LISTEN_IN_START`, `SETTLE_TRIGGERED`).
- `notebookEntries`: List of naturalist observations (text, timestamp).

---

## 4. API Surface

### State Consumption
- `GET /state`: Returns the current canonical snapshot of the aviary (birds, moods, positions, weather).
- `POST /events`: Appends interaction events to the user's log.

### Auth Flow
- `POST /auth/request-link`: Takes email, sends magic link.
- `GET /auth/verify`: Validates link, issues session token.

### Visit Flow
- `POST /visits/invite`: Host provides visitor email $\rightarrow$ server generates one-time link.
- `GET /visits/view/{token}`: Visitor accesses read-only state snapshot.

---

## 5. Simulation Engine Design

### Server-Side Tick
Runs every ~60 seconds. Process:
1. **Event Consumption**: Read new events from the `eventLog`.
2. **Drift Calculation**: Apply low-pass filter to presence-time and interaction signals to update `personalityVector` (monotonic increase toward expressive).
3. **Mood Transition**: Update `currentMood` based on:
    - Time of day (User's local TZ).
    - Recent interactions.
    - Ambient weather events.
    - Personality traits (e.g., high boldness resists `WARY`).
4. **State Update**: Write new canonical state to DB.

### Call-Grammar Runtime
Implemented client-side. Each species has a `MotifLibrary`.
- `Call = [Motif A] + [Variation(Pitch/Timing based on Personality)] + [Mood Modulation]`.
- Synthesis via WebAudio oscillators and filters to avoid looped samples.

---

## 6. Sync Model
- **Canonical Source**: Server is the sole writer of personality state.
- **Client Role**: Purely read-only for state; write-only for events.
- **Conflict Resolution**: No "last-write-wins." Events are processed sequentially by the server tick.
- **Propagation**: Clients pull snapshots on visibility change or keepalive.

---

## 7. Frontend Rendering Pipeline

### Scene Composition
- **Layering**: Background (sky/foliage) $\rightarrow$ Middle (birds/perches) $\rightarrow$ Foreground (branches/leaves).
- **Motion**:
    - **Idle**: Mood-shaped micro-motions (preening, scanning) running at 60fps.
    - **Transitions**: Smooth interpolation between server-provided perch positions.
- **Loading**: No spinners. Initial state snapshot renders immediately. If delayed, show a "quiet field" (soft sky).

### Reduced-Motion Mode
- Replace frame-by-frame animation with slow cross-fades between key poses.
- Remove ambient leaf/feather drift.
- Slow down color transitions.

---

## 8. Audio Pipeline
- **Synthesis**: Client-side WebAudio. No recorded samples.
- **Chorus Mixing**: Multiple procedural calls overlap without phase-canceling.
- **Listen-in Mix**:
    - Focus Bird $\rightarrow$ Volume $\uparrow$ (slow ramp).
    - Ambient Birds $\rightarrow$ Volume $\downarrow$ (slow ramp, never mute).
- **Fallback**: Graceful silence + automatic call captioning.

---

## 9. Accessibility Surfaces
- **Screen-Reader Narration**:
    - Server/Client generates naturalist prose (e.g., "a small grey bird is perched...").
    - Slow cadence (30-60s updates) to avoid queue flooding.
- **Call Captions**: Procedural text (e.g., "a low trill") appearing near the calling bird.
- **Keyboard Nav**: Tab through top-bar $\rightarrow$ Arrow keys between birds $\rightarrow$ Enter for listen-in.

---

## 10. Performance Budgets & Observability

### Budgets
- **Bundle Size**: < 2MB gzipped (achieved via procedural assets and code-splitting).
- **Time to First Bird**: < 500ms (achieved via CDN edge snapshots and lean render path).
- **Runtime**: 60fps on 5-year-old hardware; zero memory growth over 30 mins.

### Observability
- **Synthetic Monitoring**: Automated browsers checking load times and render frames.
- **RUM**: Anonymized session durations and audio-context errors.
- **Privacy Boundary**: Per-bird interaction data is strictly excluded from telemetry pipelines.

---

## 11. Rollout
1. **Alpha**: Internal testing of drift calibration (1 week measurable / 3 weeks visible).
2. **Beta**: Small cohort to test multi-device sync and WebAudio compatibility.
3. **v1 Launch**:
    - Starter: 2 birds.
    - Cap: 7 birds.
    - New bird offers based on aviary age.

---

## 12. Risks & Mitigations
- **Drift Calibration**: Risk of "Tamagotchi-fication" if drift is too fast. *Mitigation*: Strict low-pass filter and test harness verification.
- **Audio Uncanniness**: Procedural synthesis might sound "robotic." *Mitigation*: Heavy use of random variation and mood-based timing.
- **Sync Correctness**: Risk of state jitter. *Mitigation*: Client-side interpolation between snapshots.
- **Accessibility Regressions**: Risk of "checklist" accessibility. *Mitigation*: Design narration as a first-class naturalist surface.
