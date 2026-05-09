# Pocket Aviary: Phase 1 Implementation Plan

## 1. Scope and Core Identity

Pocket Aviary is a browser-based, naturalist virtual bird sanctuary focused on long-term observational relationships rather than short-term gamified engagement.

### Included in V1
- **Bird Engine**: Procedural call grammar, personality drift (monotonic toward expressive), and mood systems for 6 species.
- **Interactions**: Return-greeting (procedural), Listen-in (mix-based), Offer (seed/song/pool), and Settle (soft-exit gesture).
- **Aviary Scene**: Single horizontal viewport with 3-plane parallax, day/night cycles, and ambient weather.
- **Account & Sync**: Magic-link auth, server-side simulation tick, multi-device state synchronization via canonical snapshots.
- **Social**: One-to-one read-only visit invitations (opt-in only).
- **Field Notebook**: Naturalist, auto-generated observation log.
- **Accessibility**: Screen-reader narration (naturalist prose), reduced-motion mode (cross-fade pose rendering), and call captions.

### Explicit Non-Goals (Out of Scope)
- **No Gamification**: No streaks, counters, levels, or achievements.
- **No Custodial Mechanics**: No hunger, death, or punishment for absence.
- **No Native Apps**: Web-only (v1).
- **No Social Network**: No public profiles, discovery feeds, or global discovery.

---

## 2. Architecture

### System Shape
A client-server architecture where the **Server** acts as the canonical simulation engine and the **Client** acts as a stateless rendering and interaction-capture surface.

- **Frontend**: React/TypeScript with Vanilla CSS for the UI chrome; WebAudio for procedural calls; Canvas/WebGL or SVG/DOM (to be finalized) for the rendering pipeline.
- **Backend**: Node.js/TypeScript service hosting the Simulation Tick and Event Log.
- **Persistence**: 
    - **Event Log**: Append-only store for interaction events.
    - **State Store**: Key-value or Document store (e.g., PostgreSQL with JSONB) for account-level canonical aviary state (Personality Vectors, current Mood, Notebook).
    - **Auth**: Redis or similar for magic-link session management.

---

## 3. Data Model

### Account & Aviary
- `AccountID` (UUID)
- `Email` (Encrypted)
- `AviaryState`:
    - `Birds`: List of `BirdID`s.
    - `Notebook`: Array of `Entry` (Timestamp, Prose).
    - `VisitInvites`: List of `Invite` (Email, LinkHash, Status).

### Bird Entity
- `BirdID` (UUID)
- `SpeciesID` (Reference to species pool)
- `Name` (String)
- **Personality Vector** (Scalars 0.0-1.0):
    - `Boldness`, `SocialWarmth`, `VocalFrequency`, `PlumageSaturation`, `Curiosity`.
- **Mood State**: Enum (`Wary`, `Content`, `Curious`, `Drowsy`, `Alert`).
- `CurrentPerch`: (`Front`, `Middle`, `Back`).

---

## 4. API Surface

### Client-to-Server (Interaction Events)
- `POST /events`: Submit interaction event (Payload: `type: "offer" | "listen_in" | "presence_ping" | "settle"`, `bird_id?`, `duration?`).
- `GET /state`: Retrieve current canonical snapshot.
- `POST /auth/magic-link`: Request sign-in link.
- `POST /social/invite`: Create visit invitation.

### Server-to-Client (Snapshots)
- Returns `AviaryState` + `ServerTime` for local sync.

---

## 5. Simulation Engine Design

### The Server-Side Tick
- **Cadence**: ~1 minute.
- **Logic**:
    1. **Process Event Log**: Calculate `PresenceTime` and interaction weights.
    2. **Personality Drift**: Apply `Drift(current_vector, delta)` where delta is derived from presence/interactions. Drift is strictly additive/monotonic toward 1.0.
    3. **Mood Transition**: Update based on Time-of-Day, Weather, and recent Event Log triggers.
    4. **Notebook Generation**: Run template-based naturalist prose generator if noteworthy changes occurred.

### Audio Pipeline: Call-Grammar Runtime
- **Engine**: WebAudio API.
- **Synthesis**: Motif libraries (oscillator/noise configurations) sequenced with personality-shaped timing.
- **Listen-in**: Gain nodes manage a gradual ramp (3-5s) to elevate the focused bird while ducking others.

---

## 6. Frontend Rendering Pipeline

### Scene Composition
- **Layers**: Background (Atmospheric), Middle (Birds/Perches), Foreground (Micro-parallax branches).
- **Idle Motion**: Pose-based interpolation using `requestAnimationFrame`.
- **Reduced-Motion Mode**: Overrides interpolation with a `opacity` cross-fade between static poses (Duration: 1s).
- **Loading**: "Quiet Field" placeholder (soft sky gradient) instead of a spinner while waiting for first snapshot.

---

## 7. Sync Model

- **One Source of Truth**: The Server is the only writer of `Personality Vector`.
- **Conflict Avoidance**: Clients never send absolute state values. All mutations are sent as *events* to the append-only log. The server processes these in order.
- **Snapshots**: Clients pull snapshots on `visibilityChange` and at a low-frequency heartbeat (30s) to re-align with the server tick.

---

## 8. Accessibility Surfaces

- **Screen-Reader Narration**: A dedicated `aria-live` region updated with naturalist prose snapshots every 30-60s.
- **Call Captions**: Dynamic text overlays near birds, generated from the same motif library that drives the audio.
- **Keyboard Navigation**: Standard focus ring (high-contrast outline). `ArrowKeys` to cycle birds, `Enter` to listen-in.

---

## 9. Performance Budgets

- **Bundle Size**: < 2MB gzipped (critical for "already in motion" feel).
- **Time-to-First-Bird**: < 500ms on 4G/Mid-tier mobile.
- **Runtime**: 60fps on 5-year-old hardware.
- **Observability**: Synthetic monitoring for "Time to First Bird" and "Simulation Tick Latency" (p99 < 5s).

---

## 10. Rollout and Risks

### Rollout Strategy
- **Phase 1**: Internal dogfooding (2 birds max).
- **Phase 2**: Invitation-only beta to calibrate Drift.
- **Phase 3**: V1 launch with Species Pool (6 species).

### Primary Risks
- **Drift Calibration**: If too fast, it's a game; if too slow, it's a screensaver. Mitigated by beta instrumentation.
- **Audio Uncanniness**: Procedural synthesis failing to sound "natural." Mitigated by professional audio motif design.
- **Sync Lag**: Multi-device users seeing "teleporting" birds. Mitigated by server-side tick dominance and client-side interpolation.
- **PII Leak**: Email used as ID. Mitigated by strict Synthetic UUID rule.
