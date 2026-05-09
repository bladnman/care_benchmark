# Implementation Plan: Pocket Aviary v1

## 1. Scope and Strategy
This plan covers the implementation of Pocket Aviary v1, a browser-based virtual aviary focused on long-term relationships with procedural birds through observation and idle attention.

### 1.1. In-Scope (v1)
- **Core Engine:** Personality vector model, fast-timescale mood transitions, monotonic expressive drift.
- **Simulation:** Server-side authoritative tick (~1 min cadence) for persistent aliveness.
- **Frontend:** Single-screen responsive horizontal scene, three-perch depth model, day/night and ambient weather cycles.
- **Audio:** Client-side WebAudio procedural call synthesis using motif grammars.
- **Interactions:** Return-greeting, Listen-in (mix focus), Offers (seed/song/pool), Settle gesture.
- **Features:** Field Notebook (auto-generated naturalist prose), magic-link auth, multi-device sync.
- **Social:** One-time read-only visit invitations (email-based).
- **Accessibility:** Screen-reader narration (naturalist prose), reduced-motion mode (cross-fade rendering), call captioning.

### 1.2. Out-of-Scope (Non-Goals)
- **No Native Apps:** Web-only v1.
- **No Gamification:** Absolute prohibition on streaks, levels, achievements, or scores.
- **No Custodial Pressure:** No hunger, death, or distress meters; no punishment for absence.
- **No Social Network:** No public discovery, follows, profiles, or chat.
- **No Multi-Aviary:** One aviary per account.

---

## 2. Architecture
The system follows a thin-client, thick-server pattern to ensure aliveness independent of the user's presence.

### 2.1. System Overview
- **Client (React/TypeScript):** State-driven rendering engine. Interpolates between server snapshots. Handles WebAudio synthesis and local presence detection.
- **Simulation Service (Node.js/Go):** Periodically processes the "Tick". Computes personality drift and mood transitions. Writes canonical state.
- **API Gateway:** Handles authentication (Magic Link), state delivery (Snapshot API), and event ingestion (Event Log).
- **Persistence Layer:**
    - **Canonical State Store (Document DB):** Current aviary state (bird positions, moods, vectors).
    - **Event Log (Append-only):** User interaction history (offers, presence, listen-ins).
    - **Notebook Store:** Persistent generated observation logs.

### 2.2. Render Pipeline
- **Scene Composition:** Layered Canvas or SVG-based rendering with 3-plane parallax (Foreground/Middle/Background).
- **Interpolation:** Client maintains a small buffer of snapshots to smoothly animate bird transitions between perches.
- **Micro-motion:** Procedural animations (preening, head-tilting) run on a local loop modulated by the current mood state.

---

## 3. Data Model

### 3.1. Account
- `account_id`: Synthetic UUID (PII isolation).
- `email_encrypted`: Encrypted email for auth.
- `created_at`: Aviary age anchor for bird adoption pacing.

### 3.2. Bird Record
- `bird_id`: Immutable identifier.
- `species_id`: Link to species pool (visuals/motifs).
- `name`: User-defined string.
- `personality_vector`: `{boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity}` (0.0 - 1.0).
- `mood_state`: Current enum (wary, content, curious, drowsy, alert).
- `current_perch`: (front, middle, back).

### 3.3. Interaction & Presence
- `presence_ping`: Conjunction of `visibilityState: visible`, window focus, and recent input.
- `event_entry`: Type, timestamp, and metadata (e.g., `OFFER_ACCEPTED`).

---

## 4. Simulation Engine Design

### 4.1. The Tick
- **Interval:** 60 seconds.
- **Logic:**
    1. Fetch all events since the last tick for an account.
    2. Update `personality_vector` using a low-pass filter (Additive Drifts).
    3. Update `mood_state` based on time-of-day (Local TZ), ambient weather, and recent interaction density.
    4. Compute "Idle Actions" (perch changes, call triggers) for the next snapshot.

### 4.2. Drift Function
- **Monotonicity:** Trait values move up on positive input; neglect results in ambient stagnation (no negative drift).
- **Calibration:** Measurable in logs after 1 week; perceptible by users after ~3 weeks.

---

## 5. Audio Pipeline

### 5.1. Procedural Call Synthesis
- **WebAudio API:** Motif-based oscillators/samplers. 
- **Grammar:** Motifs combined based on bird species and modulated in pitch/timing by `vocal_frequency` and `mood`.
- **Mixing:** "Listen-in" focus applies a 2-second gain ramp to the target bird while lowering others to 20% volume (never 0%).

### 5.2. Fallback
- Silence + visual captions if WebAudio is unavailable. No recorded loop fallback.

---

## 6. Sync and Privacy

### 6.1. Multi-Device Sync
- Clients are read-only for state; write-only for events.
- No "Last-Write-Wins" conflicts on personality because the client never writes it.
- State Snapshots include a `server_time` to align local day/night cycles.

### 6.2. Privacy Mandate
- **PII Isolation:** Email is never used as an internal key.
- **Data Boundary:** Interaction logs are strictly for the user's simulation. No aggregate training, no third-party sharing, no "population averages."

---

## 7. Accessibility

### 7.1. Designed Surfaces
- **Narration:** `aria-live` region updated with naturalist prose descriptions of the scene every 45s.
- **Reduced Motion:** Replace path-based flight with 1s opacity cross-fades. Replace jittery micro-motion with slow pose-morphing.
- **Captions:** On-screen text for procedural calls (e.g., "*a low trill*") using the naturalist voice.

---

## 8. Performance and Observability

### 8.1. Budgets
- **Bundle Size:** <2MB gzipped initial payload.
- **TTFB (Time to First Bird):** <500ms on 4G.
- **Runtime:** 60fps on 5-year-old hardware. Zero memory growth over 30-min sessions.

### 8.2. Observability
- Instrumented p99 latency for the Simulation Tick.
- Aggregate RUM (Time-to-first-bird, render-frame drops).
- Explicit exclusion of per-bird state from telemetry pipelines.

---

## 9. Rollout and Risks

### 9.1. Rollout
- **Phase 1:** 2 starter birds, max 7.
- **Pacing:** New bird adoption slots unlocked by aviary age (e.g., Month 1, Month 3, Year 1).

### 9.2. Risks
- **Drift Calibration:** Function may feel static or too rapid; requires "dry-run" simulation testing in CI.
- **Sync Lag:** Snapshot interpolation might jitter on poor connections; requires robust client-side smoothing.
- **Audio Fatigue:** Procedural motifs might become "uncanny" if variation is too low; requires a diverse motif library per species.
- **Accessibility regressions:** Automation of naturalist prose for narration is complex; requires template-based generation with high variance.
