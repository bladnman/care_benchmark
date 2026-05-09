# Implementation Plan - Pocket Aviary v1

## 1. Scope
Pocket Aviary v1 focuses on establishing a low-key, observational relationship between the user and a small set of procedurally animated birds. The scope is strictly limited to a browser-based experience with high-fidelity affective depth and minimal UI chrome.

### In-Scope
- **Platform:** Modern web browsers (Chrome, Safari, Firefox, Edge).
- **Core Engine:** Procedural bird animation, call synthesis (WebAudio), and server-side simulation tick.
- **Population:** 2 starter birds, capping at 7 based on aviary age.
- **Interactions:** Return-greeting, Listen-in, Offer (seed, song, pool), Settle gesture, Field Notebook.
- **Identity & Sync:** Magic-link auth, single canonical aviary per account, multi-device sync.
- **Social:** One-to-one email-based visit invitations (read-only, ambient).
- **Accessibility:** Designed surfaces for screen-reader narration, reduced-motion mode, and call captioning.

### Explicit Non-Goals
- **No Native Apps:** Web-only for v1.
- **No Gamification:** No streaks, scores, levels, or achievements.
- **No Custodial Mechanics:** Birds do not die or require maintenance; no "hunger" or "happiness" meters.
- **No Social Network:** No profiles, discovery feeds, or public aviaries.

---

## 2. Architecture
The system follows a thin-client, thick-server simulation model to ensure continuity and multi-device coherence.

- **Client:** React (TypeScript) for UI chrome and layout; WebAudio for procedural call synthesis; Canvas/WebGL for bird rendering and scene composition. The client is a view-only renderer of snapshots.
- **Simulation Service (Server):** A Node.js service running the "tick" (1-minute cadence). It consumes an append-only event log (presence, interaction events) and updates the canonical state.
- **State Storage:** PostgreSQL for persistent records (Account, Bird, Personality Vector, Notebook). Redis for recent event-log buffering and active session snapshots.
- **Render Pipeline:** The client pulls a JSON snapshot. It uses a state-interpolation layer to smooth transitions between snapshot positions. Micro-motions (preening, head-tilting) are handled by client-side local loops informed by the bird's current mood.

---

## 3. Data Model

### Account
- `id`: UUID (Synthetic)
- `email_hash`: Encrypted email for magic-link auth.
- `created_at`: Timestamp (used for bird-adoption pacing).
- `settings`: (Accessibility preferences, notification toggles).

### Bird
- `id`: UUID (Stable identity).
- `account_id`: Owner reference.
- `species_id`: Pool reference.
- `name`: User-assigned string.
- `personality_vector`: 
    - `boldness`: [0.0 - 1.0]
    - `social_warmth`: [0.0 - 1.0]
    - `vocal_frequency`: [0.0 - 1.0]
    - `plumage_saturation`: [0.0 - 1.0]
    - `curiosity`: [0.0 - 1.0]
- `current_mood`: Enum (wary, content, curious, drowsy, alert).
- `drift_accumulator`: Buffer for pending drift calculations from the tick.

### Field Notebook
- `id`: UUID
- `account_id`: Reference.
- `timestamp`: Creation time.
- `content`: Naturalist prose string.

---

## 4. API Surface

### Auth
- `POST /auth/request-link`: Triggers magic-link email.
- `POST /auth/verify`: Consumes link, returns session JWT.

### Aviary State
- `GET /aviary/snapshot`: Returns current state of all birds, weather, time-of-day, and active events.
- `GET /aviary/notebook`: Returns paginated notebook entries.

### Interactions
- `POST /aviary/events`: Appends events to the log (e.g., `presence_ping`, `listen_in_start`, `offer_seed`).

### Social
- `POST /social/invite`: Generates visit link for email.
- `DELETE /social/invite/:id`: Revokes invitation.
- `GET /social/visit/:token`: Returns ambient snapshot for visitors.

---

## 5. Simulation Engine Design

### The Server-Side Tick
- **Cadence:** 1 minute.
- **Process:**
    1. Fetch pending interaction events since the last tick.
    2. Calculate **Presence-Time** (validating pings against visibility/focus rules).
    3. Update **Personality Vectors**: Apply monotonic drift function (moving toward expressive) based on presence and interaction weights.
    4. Transition **Mood**: Evaluate local time, ambient weather, and recent interactions to update bird moods.
    5. Generate **Notebook Entries**: Probabilistic check based on noteworthy state changes (e.g., "Pip greeted before Wren").
    6. Write new **Canonical Snapshot** to Redis/Postgres.

### Drift Function
Implemented as a low-pass filter. $V_{new} = V_{old} + (Input \times DriftCoefficient)$.
- Coeff calibrated for visible change in ~3 weeks of regular presence.
- Asymmetric: Input is $\ge 0$. Neglect results in 0 change, never negative.

### Call-Grammar Runtime
- Client-side motif sequencer. Motif sets are species-specific.
- Timing ($T_{interval}$) is modulated by `vocal_frequency` and `mood`.
- Pitch and vibrato varied by `social_warmth`.

---

## 6. Sync Model
- **Canonical Source:** The Server is the only writer for personality and notebook state.
- **Conflict Prevention:** Clients never submit state; they submit *events*. This makes last-write-wins irrelevant for bird traits.
- **Propagation:** Clients pull snapshots. Long-polling or low-frequency fetch (30s) ensures multi-device consistency without high overhead.

---

## 7. Frontend Rendering Pipeline

### Scene Composition
- Three-plane parallax: Foreground (branches), Middle (birds/perches), Background (foliage/sky).
- **Idle Micro-motion:** Procedural CSS/Canvas transforms for "breathing" and "scanning."
- **Reduced Motion:** Opt-in surface using slow cross-fades (3s) between still poses instead of frame-by-frame animation.

### Loading Strategy
- **Critical Path:** State snapshot is embedded in initial HTML or fetched immediately.
- **First Frame:** Render birds in static "ready" poses from snapshot instantly. Begin procedural animation loops only after assets load. No spinner.

---

## 8. Audio Pipeline

### Synthesis
- WebAudio oscillators and gain nodes synthesize species-specific motifs.
- **Chorus Mixing:** Gain levels for all birds are managed by a central `AviaryMixer`.
- **Listen-in Mix:** Focus on Bird A triggers a 2s linear ramp: `Gain_A` rises (+6dB), `Gain_Others` drops (-12dB).

### Audio Fallback
- If WebAudio fails: Silence + Automatic Call Captions enabled.

---

## 9. Accessibility Surfaces

### Narration
- A hidden ARIA-live region (polite) updated by a naturalist prose generator.
- Cadence: 45s at idle.
- Voice: "a small bird is preening on the front rail."

### Captions
- Visual overlay triggered by WebAudio motifs.
- Naturalist phrasing: "a low trill, pausing, low trill again."

### Keyboard
- Full Tab/Arrow/Enter navigation. Focus indicators designed for naturalist high-contrast (soft glow).

---

## 10. Performance Budgets

- **JS Bundle:** <2MB gzipped. (Code-splitting for settings/notebook).
- **Time-to-First-Bird:** <500ms on 4G.
- **Runtime:** Consistent 60fps on 2021-era mid-range hardware.
- **Memory:** Zero growth over 30min session (strict buffer reuse).

---

## 11. Rollout & Risk

### Calibration
- **Drift Risk:** If birds drift too fast, it feels like a toy; too slow, it feels like a screensaver.
- **Mitigation:** Server-side drift coefficients are tweakable without client updates. Internal "Phase 0" with a week of simulated presence to verify feel.

### Technical Risks
- **Sync Jitter:** Snapshot interpolation might "pop" on high-latency links.
- **Mitigation:** Adaptive interpolation windows based on RTT.

### Rollout
- **Phase 1:** Invite-only alpha (100 users) to calibrate drift and server load.
- **Phase 2:** Public v1 launch with 2-bird start.
- **Instrumentation:** Aggregate RUM for frame-rate and bundle size; no PII or bird-state telemetry.
