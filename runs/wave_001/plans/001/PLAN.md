# Pocket Aviary: Phase 1 Implementation Plan (v1)

## 1. Scope & Non-Goals
This plan covers the development of the V1 Pocket Aviary product as defined in the PRD.

### In-Scope (V1)
- **Core Experience:** A browser-based, single-scene horizontal aviary with 2-7 birds.
- **Bird Engine:** Personality vectors (Boldness, Social Warmth, Vocal Frequency, Plumage Saturation, Curiosity), fast-timescale Mood system, and procedural call synthesis.
- **Simulation:** Server-side tick (~1 min cadence) advancing canonical state (personality drift, mood transitions, presence accounting).
- **Interactions:** Listen-in (focusing a bird), Offer (seed, song fragment, still pool), Settle (soft session-end), and Return-greeting (procedural).
- **Accounts & Sync:** Single-user email magic-link authentication, multi-device sync via canonical server state, and synthetic UUID-based identity.
- **Social:** Opt-in, read-only, ambient "Visits" via email invitation.
- **Field Notebook:** Auto-generated naturalist observation log.
- **Accessibility:** Screen-reader narration (naturalist prose), reduced-motion mode (cross-fade animation), call captioning, and WCAG AA contrast.
- **Performance:** <2MB JS bundle, <500ms time-to-first-bird, 60fps idle motion.

### Non-Goals (Explicitly Excluded)
- **Gamification:** No scores, streaks, levels, badges, or achievements.
- **Tamagotchi Mechanics:** No mortality, hunger, or distress; no negative drift on neglect.
- **Native Apps:** Web-only for V1.
- **Social Network Surfaces:** No profiles, follows, public discovery, or chat.
- **Co-presence:** Visitors are read-only and do not interact or influence the host's birds.

## 2. Architecture & Data Model

### Service Shape & Client/Server Split
- **Client (Frontend):** A reactive web application responsible for rendering the aviary scene, synthesizing audio via WebAudio, handling user interactions, and managing local presence signals. It acts as a stateless viewer of the server-side canonical state.
- **Server (Backend):** The authoritative source of truth. It hosts the simulation engine, manages the append-only interaction event log, persists the canonical aviary state, and runs the periodic simulation tick.

### Data Model
- **Account:** `UUID (Primary Key)`, `Encrypted Email`, `Settings (Accessibility, Social Opt-in, Notification Opt-in)`.
- **Aviary:** `UUID (FK to Account)`, `Last Tick Timestamp`.
- **Bird:** `UUID (Stable ID)`, `Species ID`, `User-assigned Name`, `Personality Vector (5 floats)`, `Current Mood (Enum)`, `Current Perch (Front/Middle/Back)`, `Last Interaction Timestamp`.
- **Interaction Event Log:** `Event ID`, `Bird ID`, `Type (Offer, Listen-in Start/End, Settle, Presence Ping)`, `Payload (Offer type, etc.)`, `Timestamp`.
- **Field Notebook Entry:** `Entry ID`, `Timestamp`, `Prose Content (Naturalist)`.
- **Visit:** `Visit ID`, `Host Account UUID`, `Visitor Email`, `Status (Outstanding/Active/Revoked)`, `Last Access Timestamp`.

## 3. Simulation Engine Design

### The Server-Side Tick
A recurring task (approx. 60s) that:
1. **Consumes the Event Log:** Processes all pending interaction events for the account in chronological order.
2. **Calculates Personality Drift:** Applies low-pass filters to update personality vectors based on accumulated presence-time and interaction-weighting. Drift is monotonic toward expressive.
3. **Updates Mood:** Transitions bird moods based on:
    - Recent interactions.
    - Time of day (local to user).
    - Ambient events (weather, other bird calls).
    - Personality constraints (e.g., high-boldness reduces wary probability).
4. **Generates Notebook Entries:** Periodically (based on significant events or time) synthesizes naturalist prose based on recent state changes.
5. **Persists State:** Writes the updated canonical aviary state to the database.

### Presence & Drift Calibration
- **Presence Signal:** The client sends "presence pings" only when `visibilityState === 'visible'`, `document.hasFocus() === true`, and recent `pointermove`/`keypress` activity is detected.
- **Drift Function:** $\Delta Personality = f(\text{PresenceTime}, \text{Interactions})$. The function is tuned so that measurable drift appears in telemetry after 1 week and visible behavioral change appears to the user after ~3 weeks.

## 4. API Surface & Sync Model

### Client-to-Server (Interaction Events)
Clients do not write state; they append events:
- `POST /events/presence`: Presence heartbeats.
- `POST /events/interaction`: `type: "offer" | "listen-in-start" | "listen-in-end" | "settle"`.

### Server-to-Client (State Consumption)
- `GET /snapshot`: Returns the current canonical state (bird positions, moods, call timing, notebook, etc.).
- Clients pull this on: visibility change, long render-frame gaps, or low-frequency keepalive.
- **Interpolation:** Clients use the snapshot to interpolate between state $N$ and $N+1$ for smooth motion/audio transitions.

### Visit Flow
1. **Host:** `POST /invites` with visitor email.
2. **System:** Sends email with magic-link containing `HostID` and `Token`.
3. **Visitor:** Follows link $\rightarrow$ authenticated read-only session $\rightarrow$ `GET /snapshot` (server filters for read-only mode).

## 5. Frontend Rendering & Audio Pipelines

### Rendering Pipeline
- **Scene Composition:** A single horizontal viewport using layered 2D/pseudo-3D elements (foreground/middle/background) with subtle parallax.
- **Animation:**
    - **Idle Motion:** Procedural, mood-shaped micro-motions (preening, scanning, head-tilting).
    - **Reduced-Motion Mode:** Replaces frame-by-frame animation with slow cross-fades between still poses.
- **Loading Sequence:** The "Quiet Field" approach. Avoid spinners. Render a soft-colored, lightly-animated scene immediately while the first snapshot is fetched.

### Audio Pipeline (WebAudio)
- **Procedural Call Synthesis:** No audio files. Synthesis of motifs (pitch, timing, timbre) based on the bird's `Vocal Frequency` and `Mood`.
- **The Chorus Mixer:** A dynamic mix where:
    - `Ambient`: All birds at a low, baseline volume.
    - `Listen-in`: The focused bird's volume ramps up; others stay in the ambient mix but at a reduced level.
- **Fallback:** If WebAudio fails, the system enters "Graceful Silence" with call captions enabled by default.

## 6. Accessibility & Performance

### Accessibility Surfaces
- **Naturalist Narration:** An ARIA-live or similar mechanism that reads out periodic, naturalist prose describing the aviary state (e.g., "a small grey bird is perched on the front rail...").
- **Call Captioning:** Short, mood-aware prose captions (e.g., "a soft three-note rise") rendered near the bird.
- **Keyboard/Focus:** Full navigation through the top bar and bird-to-bird focus using standard keyboard patterns.

### Performance Budgets
- **Bundle Size:** Initial JS < 2MB (gzipped).
- **Time-to-First-Bird:** < 500ms on mid-tier mobile/4G.
- **Runtime:** Consistent 60fps idle motion on 5-year-old laptops.
- **Memory:** Zero growth over a 30-minute session (tested via CI).

## 7. Rollout & Risks

### Rollout Strategy
1. **Alpha:** Internal testing of simulation accuracy and drift calibration.
2. **Beta:** Limited access to verify multi-device sync and "felt aliveness."
3. **V1 Launch:** Ramp up birds-per-aviary (starting at 2) and monitor performance/stability.

### Key Risks & Mitigations
- **Drift Calibration:** Risk of "Tamagotchi" (too fast) or "Screensaver" (too slow) feel. *Mitigation: Strict telemetry monitoring of personality deltas.*
- **Sync Correctness:** Risk of divergent simulations. *Mitigation: Server-authoritative model; clients are strictly read-only for personality state.*
- **Audio Uncanniness:** Risk of procedural calls sounding robotic. *Mitigation: High-quality motif libraries and emphasis on timing/pitch variation.*
- **Accessibility Regressions:** Risk of treating accessibility as a fallback. *Mitigation: Design accessibility as a primary, naturalist-voiced surface from day one.*
