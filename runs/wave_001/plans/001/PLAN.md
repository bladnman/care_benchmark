# Pocket Aviary v1 Implementation Plan

This plan outlines the technical implementation of Pocket Aviary v1, a browser-based virtual aviary focused on slow-timescale bird evolution and low-key user presence.

## 1. Scope and Constraints

### Included in v1
- **Birds:** 2 starter birds, capping at 7 total per aviary based on aviary age.
- **Core Mechanics:** Server-side simulation (ticks), personality drift (monotonic), mood transitions, presence accounting.
- **Interactions:** Return-greeting, Listen-in, Offer (seed, song fragment, still pool), Settle (soft session-end).
- **Surface:** Single horizontal scene, three perch zones, day/night cycle, ambient weather, field notebook.
- **Infrastructure:** Email magic-link auth, multi-device sync, server-side state snapshots.
- **Social:** Visit invitations (one-time link, read-only, no co-presence).
- **Accessibility:** Screen-reader narration (naturalist prose), reduced-motion mode (cross-fades), call captioning.

### Explicit Non-Goals
- **No Native Apps:** Web-only.
- **No Gamification:** No streaks, achievements, levels, or scores.
- **No Custodial Mechanics:** Birds do not die, starve, or show distress. No happiness meters.
- **No Social Network:** No profiles, discovery feeds, chat, or public ranking.

## 2. Architecture

### Client/Server Split
- **Server:** Canonical state owner. Runs the simulation engine (tick), processes event logs, manages auth, and generates naturalist prose for the notebook and accessibility narration.
- **Client (React/TypeScript):** State consumer and renderer. Responsible for high-fidelity animation (WebGL or Canvas), procedural audio synthesis (WebAudio), and interaction event emission.

### Render Pipeline Boundary
- The client receives a state snapshot (JSON) containing bird positions, moods, and active transitions.
- The client interpolates between snapshots at 60fps.
- Procedural elements (micro-motion, call variation) are computed client-side based on the snapshot parameters.

## 3. Data Model

### Account & Aviary
- `Account`: `id` (UUID), `email` (encrypted), `created_at`.
- `Aviary`: `account_id`, `created_at`, `weather_state`, `time_offset`.

### Bird Model
- `Bird`: `id`, `aviary_id`, `species_id`, `name`, `personality_vector` (JSON), `current_mood`, `current_perch`, `drift_history`.
- `PersonalityVector`: { `boldness`: float, `social_warmth`: float, `vocal_frequency`: float, `plumage_saturation`: float, `curiosity`: float }.

### Persistence
- `PresenceLog`: `account_id`, `start_time`, `end_time`, `duration`.
- `InteractionEvent`: `id`, `account_id`, `bird_id`, `type` (OFFER, LISTEN_IN, etc.), `timestamp`.
- `NotebookEntry`: `id`, `aviary_id`, `prose_content`, `timestamp`.

## 4. Simulation Engine Design

### The Server-Side Tick
- **Frequency:** ~1 minute.
- **Input:** Last state + interaction events since last tick + wall clock time.
- **Process:**
    1. Update Moods: Apply time-of-day bias, weather effects, and recent interaction "nudges."
    2. Apply Drift: Update personality vectors based on accumulated `PresenceLog` duration and specific interaction weights.
    3. Generate Observations: Occasionally trigger a naturalist prose generator for the Field Notebook.
    4. Save Snapshot: Commit the new canonical state to the database.

### Drift Function
- **Monotonic Expressivity:** Personality traits only increment (or stay flat) upon presence/interaction. Neglect results in zero delta, causing the bird to feel "ambient" but not "wary."
- **Calibration:** Target measurable numerical drift at 1 week, visible behavioral change at 3 weeks.

## 5. API Surface

### Client-to-Server
- `POST /auth/magic-link`: Request sign-in link.
- `GET /aviary/state`: Pull current snapshot (long-polling or low-frequency fetch).
- `POST /aviary/event`: Append interaction (Offer, Listen-in, Settle, Presence-ping).
- `GET /notebook`: Fetch paginated observation history.

### Social/Visit Flow
- `POST /invite`: Generate one-time visit URL for a specific email.
- `GET /visit/:token`: Public read-only route for visitors (no auth required, no interaction events accepted).
- `DELETE /invite/:id`: Revoke access.

## 6. Frontend Rendering & Audio

### Scene Composition
- **Layers:** Parallax background foliage, middle-plane perches/birds, foreground framing elements.
- **Animations:** Mood-keyed idle poses (preening, scanning). 
- **Transitions:** Soft fly-ins for new birds; cross-fades for reduced-motion mode.
- **Loading:** Deliver initial snapshot with HTML (SSR/Data injection) to achieve <500ms time-to-first-bird.

### Audio Pipeline
- **WebAudio Synthesis:** Procedural call generation using oscillator nodes and gain envelopes, shaped by species-specific motifs and current bird mood.
- **Chorus Mixing:** Staggered start times for calls; listen-in focuses one gain node while attenuating others (never 0%).
- **Accessibility:** Captions generated from the same motif metadata used by the audio engine.

## 7. Accessibility Surfaces

- **Narration Engine:** Periodically generates a naturalist description of the scene (e.g., "Pip is on the front perch, preening softly in the morning light").
- **Reduced Motion:** Replace all frame-based animations with slow cross-fades between key poses.
- **Keyboard Navigation:** Full Tab/Arrow/Enter/Escape support with visible focus rings.

## 8. Performance Budgets

- **Initial Payload:** <2MB gzipped.
- **TTFB (Time-To-First-Bird):** <500ms on 4G.
- **Runtime:** Consistent 60fps; zero memory leak over 30min sessions.
- **Observability:** Track render frame drops and snapshot latency (p99 < 5s).

## 9. Rollout & Risks

### Rollout Strategy
- Day 1: 2 birds per account.
- Age-based bird unlocking (e.g., Bird 3 at 1 month, Bird 4 at 3 months).
- Instrumentation: Monitor drift velocity and presence accounting accuracy.

### Risks
- **Drift Calibration:** Ensuring drift feels "earned" but not "fast."
- **Sync Correctness:** Preventing "stale state" glitches when switching devices.
- **Audio Uncanniness:** Procedural calls must avoid phase-canceling or repetitive artifacts.
- **Accessibility Regressions:** Ensuring the prose narration remains naturalist and doesn't devolve into state-logging.
