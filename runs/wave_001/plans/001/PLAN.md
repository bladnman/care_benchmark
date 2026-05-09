# Implementation Plan: Pocket Aviary v1

## 1. Scope
Pocket Aviary v1 is a browser-based virtual aviary where birds evolve based on user presence and small interactions. The scope is strictly bounded by the design philosophy of "Notice, never announce" and "Feels alive, not robotic."

### In-Scope (v1)
*   **Bird Engine:** Personality vectors (Boldness, Social Warmth, Vocal Frequency, Plumage Saturation, Curiosity), Mood system, and Procedural Call Grammar.
*   **Interactions:** Presence-based drift, return-greetings, Listen-in, Offer (Seed, Song, Pool), Settle gesture, and the Field Notebook.
*   **Aviary Layout:** Single horizontal responsive scene, three perch zones, local-time-anchored day/night cycle, and ambient weather.
*   **Infrastructure:** Magic-link email auth, multi-device sync via server-side canonical state, and server-side simulation tick.
*   **Accessibility:** Naturalist screen-reader narration, reduced-motion mode (pose cross-fades), call captions, and keyboard navigation.
*   **Social:** One-time read-only visit invitations (opt-in only).

### Out-of-Scope (Non-Goals)
*   **No Native Apps:** Web-only (v1).
*   **No Gamification:** No streaks, scores, levels, badges, or achievements.
*   **No Tamagotchi Mechanics:** Birds do not die, hunger, or show distress. Absence is not punished.
*   **No Social Network Features:** No public discovery, profiles, follows, or co-presence.

---

## 2. Architecture
The system follows a **Server-Side Canonical State** model with thin clients rendering state snapshots.

### Service Shape
*   **Auth Service:** Handles magic-link generation, email verification, and session token issuance.
*   **Simulation Service (The Engine):** Runs the periodic "Tick" (~1min) to update all active aviaries. Consumes interaction event logs.
*   **State API:** Provides read-only snapshots of the current aviary state to clients.
*   **Event Log Service:** Append-only ingestion of client-side interaction events (Offer, Settle, Listen-in, Presence).
*   **Static Asset CDN:** Serves the 2MB (max) frontend bundle, bird SVGs/bitmaps, and motif libraries.

### Client/Server Split
*   **Server:** Owns personality vectors, mood transitions, drift calculations, and the canonical simulation tick.
*   **Client:** Handles rendering, WebAudio synthesis of procedural calls, local interpolation of snapshots, and presence measurement.

---

## 3. Data Model

### Account (PII-Partitioned)
*   `account_id`: UUID (Primary key used in all other services)
*   `email_encrypted`: Encrypted PII
*   `aviary_id`: Reference to the user's aviary

### Bird
*   `bird_id`: UUID (Stable identity)
*   `species_id`: From pool (6 species)
*   `name`: User-assigned string
*   `personality_vector`: { `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity` } (floats [0.0, 1.0])
*   `current_mood`: Enum { `wary`, `content`, `curious`, `drowsy`, `alert` }
*   `perch_id`: Current target perch
*   `state_timestamp`: Last updated by tick

### Field Notebook
*   `entry_id`: UUID
*   `account_id`: Owner
*   `timestamp`: Real-world time
*   `content_template`: Naturalist prose ID
*   `context_data`: { `bird_names`, `events` }

---

## 4. Simulation Engine Design

### Server-Side Tick (The Heartbeat)
1.  **Event Aggregation:** Collect all events for the account since the last tick (Presence time, Offers, etc.).
2.  **Drift Function:** Apply additive deltas to personality vectors.
    *   `drift_delta = filter(presence_signals, weight)`.
    *   Calibration: Measurable in instruments at 1 week, visible to user at 3 weeks.
    *   Monotonic: Values never decrease due to neglect.
3.  **Mood Engine:** Transition moods based on:
    *   Time of day (Solar position in user's timezone).
    *   Recent interactions (e.g., Offer success nudge toward `content`).
    *   Bird Personality (e.g., high `boldness` reduces `wary` probability).
4.  **State Snapshot Generation:** Write the new canonical positions and moods for client consumption.

### Call-Grammar Runtime
*   **Motif Library:** Species-specific MIDI-like fragments (pitch offsets and timing).
*   **Synthesizer:** WebAudio oscillators and filters shaped by `vocal_frequency` (cadence) and `mood` (timbre/pitch shift).

---

## 5. Sync & Interaction Model

### Sync Strategy
*   **Client Snapshots:** Client pulls every ~1min while visible or on visibility change.
*   **No Conflicts:** Clients never write state; they only append to the event log. The server is the single source of truth for personality and mood.
*   **Presence Precision:** Recorded only when `visible` + `focused` + `activity` (last 3 mins). Heartbeat pings sent to server for aggregation.

### Return-Greeting Logic
*   Triggered on fresh session or visibility change after >15 min.
*   Algorithm: Pick one bird based on `boldness` and `mood`. Stagger other responses by randomized offsets.
*   Voice: Purely procedural visual/audio; no textual "Welcome back."

---

## 6. Frontend Rendering Pipeline

### Scene Composition
*   **Horizontal Layout:** Responsive canvas with 3 perch zones (z-depth).
*   **Ambient Motion:** Continuous client-side micro-animations (preening, head-tilting) and leaf/feather drift.
*   **Loading:** Start rendering from the first available snapshot frame. If cache is cold, show a "quiet field" (sky/perch) rather than a spinner.

### Audio Pipeline
*   **WebAudio Synthesis:** Procedural generation of calls from motifs.
*   **Listen-In Mix:** Gain-node rebalancing with slow ramps (linear/exponential) to emphasize the focused bird while retaining ambient "Aviary" noise.
*   **Fallback:** If WebAudio fails, auto-enable Call Captions and play in silence.

---

## 7. Accessibility Surfaces

### Narration & Captions
*   **Naturalist Narration:** A hidden live region updated with prose describing the aviary (e.g., "pip is fluffed against the cool air").
*   **Procedural Captions:** Near-bird text tags describing call character (e.g., "a low trill").
*   **Reduced-Motion:** Replace frame-by-frame animation with 2-second cross-fades between static poses.

### Navigation
*   Full keyboard support: Tab through chrome; Arrow keys to switch bird focus; Enter to Listen-in.

---

## 8. Performance & Observability

### Budgets
*   **Bundle Size:** <2MB gzipped (Initial payload).
*   **Time-to-First-Bird:** <500ms on 4G/Mid-tier.
*   **Runtime:** Locked 60fps on 5-year-old hardware. Zero memory growth over 30 mins.

### Observability
*   **Operational Telemetry:** Latency (Tick, API), Bundle size, FPS, JS Errors.
*   **Privacy Boundary:** No bird-state or interaction history in aggregate telemetry.

---

## 9. Rollout & Risks

### Rollout Plan
*   **Day 0:** 2 birds per account.
*   **Age-Based Gating:** Birds 3-7 unlocked by account age (not engagement).
*   **Instrumentation:** Validation of drift monotonicity in shadow mode before GA.

### Key Risks
*   **Drift Calibration:** Too fast feels like a toy; too slow feels like a screensaver. Requires strict instrument-based tuning.
*   **Sync Continuity:** Maintaining "Feels Alive" requires the server tick to be highly reliable.
*   **Audio Uncanniness:** Procedural calls must avoid phase-canceling or repetitive artifacts.
*   **Accessibility Regressions:** Ensuring narration prose remains naturalist and not robotic during updates.
