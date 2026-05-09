# Implementation Plan: Pocket Aviary v1

This document outlines the engineering plan for the implementation of Pocket Aviary v1, a browser-based virtual aviary focused on long-term relationships with procedural animated birds.

---

## 1. Scope and v1 Boundaries

### In Scope
- **Core Engine:** Procedural bird behavior (personality drift, mood transitions, idle motion).
- **Audio:** Client-side WebAudio procedural call synthesis and chorus mixing.
- **Interactions:** Presence accounting, listen-in, offers (seed, song, pool), and settle gesture.
- **Content:** Field notebook (automated naturalist logs), starter species pool (6 species).
- **Accounts & Sync:** Email magic-link auth, server-side simulation tick, multi-device sync.
- **Social:** One-time read-only visit invitations (opt-in).
- **Accessibility:** Screen-reader narration, reduced-motion mode (cross-fade), call captions.
- **Rendering:** Responsive horizontal scene with parallax, day/night cycle, ambient weather.

### Out of Scope (Non-Goals)
- **No Native Apps:** Web-only at v1.
- **No Gamification:** No streaks, scores, levels, or achievements.
- **No Custodial Mechanics:** No hunger, health, or death. Neglect results in ambient quiet, not distress.
- **No Social Network:** No profiles, discovery feeds, chat, or co-presence.

---

## 2. Architecture

### System Shape
- **Client (React/TypeScript):** Thick client for rendering and audio synthesis. Stateless in terms of canonical bird data; pulls state snapshots.
- **Server (Node.js/FastAPI):** Owns the "Source of Truth." Runs the slow simulation tick.
- **Persistence:** Relational database (e.g., PostgreSQL) for account and bird records; append-only event log for user interactions.
- **CDN:** Edge caching for static assets and potentially initial state snapshots to hit the <500ms first-bird target.

### Rendering Pipeline Boundary
- **Simulation (Server):** Computes positions, moods, and trait drift.
- **Interpolation (Client):** Receives snapshots and smoothly animates birds between discrete states to avoid "teleporting."
- **Ornaments (Client):** Non-canonical motion (leaves, feathers, micro-movements) is generated locally to preserve "felt-aliveness" without server overhead.

---

## 3. Data Model

### Account & Aviary
- `AccountID` (UUID, synthetic to protect PII).
- `Email` (Encrypted, stored once).
- `AviaryID` (One per account).
- `Settings` (Accessibility prefs, visit notifications).

### Bird Record
- `BirdID` (Stable UUID).
- `SpeciesID` (Reference to species pool).
- `Name` (User-assigned).
- **Personality Vector:**
  - `Boldness`, `SocialWarmth`, `VocalFrequency`, `PlumageSaturation`, `Curiosity` (Scalars 0.0-1.0).
- **Current State:**
  - `Mood` (Enum: wary, content, curious, drowsy, alert).
  - `PerchPosition` (Enum: front, middle, back).
  - `LastInteractionTimestamp`.

### Interaction Log
- Append-only log of `PresenceEvents`, `OfferEvents`, `ListenInEvents`.

---

## 4. Simulation Engine Design

### The Slow Tick
- Runs server-side every ~60 seconds.
- **Process:**
  1. Fetch active aviaries (or those with recent interaction logs).
  2. Consume event log for presence-time and interaction signals.
  3. **Drift Function:** Apply low-pass filter to traits. Drift is monotonic toward expressive; no negative drift on neglect.
  4. **Mood Transitions:** Update mood based on Time of Day (local user time), recent interactions, and ambient events (weather).
  5. **Auto-Notebook:** Periodically evaluate state to generate naturalist prose entries.
  6. Commit new canonical state.

### Drift Calibration
- Measurable in instruments after 1 week.
- Visible to users after 3 weeks.

---

## 5. Sync & Multi-Device Model

### Canonical State
- Server is the **only writer** of personality vectors.
- Clients submit **Interaction Events**, never absolute trait values.

### Conflict Prevention
- Additive server-authored deltas processed in event-log order.
- "Last-write-wins" is explicitly prohibited for trait data.

---

## 6. API Surface

### Client -> Server
- `POST /auth/request-link`: Request magic link.
- `GET /state`: Pull current aviary snapshot.
- `POST /events`: Submit interaction event (presence ping, offer, listen-in).
- `POST /invite`: Issue visit invitation.

### Server -> Client (Snapshots)
- Returns birds, moods, positions, and current server time (for cycle sync).

---

## 7. Frontend Rendering & Audio

### Scene Composition
- Horizontal scene, 3 depth layers for parallax.
- Day/Night: Smooth CSS/SVG filter transitions anchored to user local time.
- Responsive: Preserves aspect ratio; never crops birds.

### Audio Pipeline
- **Procedural Calls:** WebAudio `AudioWorklet` or `Oscillator` chains using species-specific motif grammars.
- **Chorus Mixing:** Dynamic gain management. Listen-in uses a slow gain ramp (3-5s) to focus one bird and lower others to ambient (-12dB to -18dB).
- **No Looped Audio:** Every call has random pitch/timing variations within its motif.

---

## 8. Accessibility Surfaces

### Narration & Captions
- **Narration:** Naturalist prose (lowercase, present-tense) delivered to ARIA live regions at a slow cadence (~45s).
- **Captions:** Procedurally generated descriptions (e.g., "soft three-note rise") appearing near birds.
- **Reduced Motion:** Replace 60fps animations with slow (1-2s) cross-fades between key poses. Remove ambient leaf/feather particles.

### Navigation
- Logical tab order: Top Bar -> Aviary (Bird focus) -> Field Notebook.
- Keyboard shortcuts for `Listen-In` (Enter) and `Offer` (Key-bound).

---

## 9. Performance & Observability

### Budgets
- **JS Bundle:** <2MB (Gzipped).
- **First-Bird-Visible:** <500ms on 4G/mid-tier mobile.
- **Runtime:** Consistent 60fps for idle micro-motion on 5-year-old hardware.
- **Memory:** Zero growth over 30-minute sessions (resource pooling for audio/sprites).

### Observability
- **Synthetic:** Heartbeat checks for simulation-tick latency (Alarm at p99 > 5s).
- **RUM:** Anonymous timing for page load and audio context initialization.
- **Privacy Boundary:** Telemetry never includes bird names, trait values, or interaction history.

---

## 10. Rollout & Risk

### Calibration Risks
- **Drift "Vibe" Check:** If drift is too fast, birds feel like Tamagotchis. If too slow, like static wallpapers. Requires early testing with simulated "weeks" of interaction.
- **Audio Uncanniness:** Procedural calls must avoid "beepy" synth qualities. Lean into soft noise-shaping and organic attack/decay curves.

### Implementation Risks
- **Presence Honesty:** Ensuring the conjunction of (Visibility + Focus + Activity) correctly identifies attention without false negatives for "still watching."
- **Sync Race Conditions:** Ensuring the event log is consumed atomically per account.

### Rollout Phase
1. **Alpha:** 2 birds per aviary; basic species pool.
2. **Beta:** Intro of species 3-6; visit invitation feature.
3. **v1 Launch:** Full accessibility suite and notebook generation.
