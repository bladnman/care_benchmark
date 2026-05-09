# Implementation Plan: Pocket Aviary v1

## 1. Scope and v1 Definition

Pocket Aviary v1 is a browser-based virtual aviary focused on a slow-burn, observational relationship between the user and a small group of birds.

### Included in v1:
- **Core Engine:** Procedural bird personality drift (monotonic toward expressive), mood system, and client-side procedural call synthesis.
- **Interactions:** "Notice, never announce" return greetings, listen-in (mix re-balancing), offer gesture (seeds, song, pool), and settle gesture (soft exit).
- **Surfaces:** Single horizontal aviary scene, Field Notebook (naturalist observation log), Top-bar chrome, and Account/Settings.
- **Accounts & Sync:** Email magic-link auth, server-side simulation tick, multi-device sync with additive server-authored personality deltas.
- **Accessibility:** First-class screen-reader narration (naturalist prose), reduced-motion mode (cross-fades), and call captions.
- **Social:** One-to-one read-only visit invitations (opt-in only).

### Explicit Non-Goals:
- **No Gamification:** No streaks, scores, levels, or achievements.
- **No Custodial Pressure:** Birds do not die, starve, or show distress; no "maintenance" chores.
- **No Social Network:** No public feeds, discovery, profiles, follows, or chat.
- **No Native Apps:** Web-only for v1.

---

## 2. Architecture

### System Shape
- **Client (React/TypeScript):** Thin rendering and audio synthesis layer. Handles interpolation of snapshots and execution of procedural grammars.
- **Server (Node.js/TypeScript):** Authoritative simulation engine. Manages the "Tick" and is the sole writer of personality state.
- **Data Store:** 
  - **Relational (PostgreSQL):** Account metadata, bird records (species, IDs, names), and the Field Notebook.
  - **Event Log (Append-only):** Interaction events (presence, offers, listen-ins) to be consumed by the simulation tick.
  - **Cache (Redis):** Canonical aviary state snapshots for fast client delivery.

### Client/Server Split
- **Server-Side Tick:** Every ~60 seconds, the server processes recent events, updates personality vectors and moods, and writes a new snapshot.
- **Client Snapshot Pull:** Clients pull snapshots on visibility change and a low-frequency heartbeat (~10-15s).
- **Interpolation:** The client receives a "start" and "target" state for birds (position, mood) and interpolates motion to maintain 60fps without teleports.

---

## 3. Data Model

### Account & Aviary
- `AccountID` (UUID, synthetic to protect PII)
- `Email` (Encrypted, stored once)
- `AviaryState`: Birds list, weather, time-of-day, active visit tokens.

### Bird Object
- `BirdID` (Stable UUID)
- `SpeciesID`: Refers to species pool (visuals, base grammar).
- `Name`: User-assigned.
- `PersonalityVector`: {Boldness, SocialWarmth, VocalFrequency, PlumageSaturation, Curiosity} (Hidden scalars).
- `CurrentMood`: {Wary, Content, Curious, Drowsy, Alert}.
- `DriftHistory`: Record of accumulated presence-time and interaction weights.

### Interactions & Log
- `PresenceEvent`: {AccountID, Duration, Timestamp}.
- `InteractionEvent`: {AccountID, BirdID, Type (Offer/ListenIn), Subtype, Timestamp}.

---

## 4. Simulation Engine & Drift Function

### The Simulation Tick
- Runs server-side.
- **Mood Transition:** Computes next mood based on TimeOfDay (user local), ambient weather, and recent interaction events.
- **Drift Calculation:** 
  - `NewTraitValue = OldTraitValue + (PresenceWeight * Duration) + (InteractionWeight * Frequency)`
  - Filtered through a low-pass function to ensure slow change (weeks for visible effect).
  - **Constraint:** Monotonic increase only. Neglect results in zero delta, never negative.

### Procedural Call Grammar
- Rules-based motifs stored per species.
- Pitch and interval modulated by `VocalFrequency` (personality) and `Content/Alert` (mood).
- Audio synthesis occurs in WebAudio on the client.

---

## 5. Frontend Rendering & Audio Pipeline

### Rendering Pipeline
- **Canvas/WebGL:** For the horizontal aviary scene to handle 7 birds with micro-motion and parallax at 60fps.
- **Micro-motion Engine:** Procedural preening/scanning logic running continuously.
- **Reduced Motion:** Toggle to switch rendering to 2-3 second cross-fades between static poses.

### Audio Pipeline (WebAudio API)
- **Procedural Synthesis:** Oscillators/Buffers generating motifs rather than playing loops.
- **Mix Management:** 
  - **Ambient:** Balanced chorus of all birds.
  - **Listen-In Focus:** Focused bird gains +6dB, others drop to -12dB (ambient wash) over 2-second ramps.
- **Chorus Logic:** Real-time timing variation between birds to avoid phase-canceling or "robotic" synchronization.

---

## 6. Sync & Conflict Model

- **Additive Deltas:** Clients never send state; they send events. Server computes new state.
- **No-Last-Write-Wins:** Since only the server writes personality, multiple devices can only ever contribute to the event log. 
- **Snapshot Continuity:** If a client comes back from a long sleep, it pulls the current server-tick state, ensuring the "aviary continued without you" effect.

---

## 7. Accessibility Design

- **Narration Engine:** Server-side templating of naturalist prose (e.g., "A small grey bird is preening on the front rail"). Delivered as a live-region update to screen readers every 45s.
- **Call Captions:** Text-based descriptions (e.g., *a low trill, repeated*) displayed near the bird when vocalizing.
- **Keyboard Control:** Tab-accessible birds, Enter for Listen-In, Space for Offer.

---

## 8. Performance & Observability

### Budgets
- **JS Bundle:** <2MB gzipped (aggressive code splitting for settings/auth).
- **Time to First Bird:** <500ms on 4G (critical path: HTML -> Initial Snapshot -> Render Shell).
- **Runtime:** No memory leaks over 30min sessions; stable 60fps on 2021-era hardware.

### Observability
- **Operational:** Latency on sync-tick, bundle size, error rates for magic-link delivery.
- **Privacy Constraint:** No tracking of specific per-bird interactions in aggregate analytics. Only anonymized session duration and general health metrics.

---

## 9. Rollout Strategy

1. **Internal Alpha:** Verify drift calibration and audio "chorus" feel.
2. **Limited Beta:** Test magic-link and sync stability across device types.
3. **Public v1:** Launch with 2-bird starter cap, 7-bird aviary cap.
4. **Instrumentation:** Focus on "Time to First Bird" and "Simulation Tick Health."

---

## 10. Risks & Mitigations

- **Drift Calibration:** Too fast = Tamagotchi; Too slow = Screensaver. *Mitigation:* Explicit "instruments-visible" vs "user-visible" testing targets (1 week vs 3 weeks).
- **Audio Uncanniness:** Stacked procedural calls sounding robotic. *Mitigation:* Random jitter in motif start-times and pitch variation tied to bird ID.
- **Sync Drift:** Server/Client clock skew. *Mitigation:* All logic server-authoritative; client only interpolates between server-provided timestamps.
- **Accessibility Regressions:** Visual changes breaking narration prose. *Mitigation:* Narration prose generated from the same data model as the rendering engine.
