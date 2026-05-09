# Implementation Plan - Pocket Aviary v1

This document outlines the engineering strategy for delivering Pocket Aviary v1. It translates the naturalist product requirements into a specific, executable technical architecture.

---

## 1. Scope and Boundaries

### In-Scope (v1)
*   **Birds**: Initial adoption of 2 birds, capping at 7 (scaling based on aviary age).
*   **Engine**: Personality drift (monotonic toward expressive), fast-timescale mood, and procedural call synthesis.
*   **Interactions**: Presence tracking (visibility + focus + activity), listen-in (audio re-balancing), offers (seed, song, pool), and settle gesture.
*   **Journaling**: Auto-generated naturalist "Field Notebook."
*   **Accounts**: Single-user magic-link auth, multi-device state sync via server-side canonical simulation.
*   **Social**: Read-only visit invitations (per-invite opt-in).
*   **Accessibility**: Naturalist prose narration for screen readers, reduced-motion cross-fade mode, and call captions.

### Out-of-Scope (v1)
*   Native apps (web-only).
*   Any form of gamification (streaks, levels, counters).
*   Custodial mechanics (birds do not die or get hungry).
*   Social network features (profiles, chat, public discovery).

---

## 2. Architecture

The system uses a **Server-Authoritative Simulation** model. Clients are thin rendering and interaction-capture shells.

### 2.1 Component Split
*   **Simulation Service (Server)**: A Node.js service running a "Universal Tick." It advances aviary state (drift, mood, position) for all accounts.
*   **API Gateway**: Handles magic-link auth, state snapshot delivery, and append-only event ingestion.
*   **Client (Web)**: React-based SPA. Handles WebGL rendering, WebAudio synthesis, and presence/interaction logic.

### 2.2 The Tick Pipeline
1.  **Ingest**: Collect interaction events (Offer, Listen-in, Presence Pings) from the Event Log.
2.  **Evaluate**: Update Personality Vectors using a low-pass filter (drift function).
3.  **Transition**: Update Moods based on Time-of-Day (local to user), ambient events, and recent interactions.
4.  **Snap**: Generate a static JSON snapshot (bird perches, mood states, call motifs).
5.  **Broadcast**: Store the snapshot in a distributed cache (Redis) for low-latency client retrieval.

---

## 3. Data Model

### 3.1 Persistent Storage (PostgreSQL)
*   **Accounts**: `id (UUID)`, `email_hash`, `created_at`, `settings`.
*   **Aviaries**: `account_id`, `age_days`, `last_tick_at`.
*   **Birds**: `id`, `aviary_id`, `species_id`, `name`, `personality_vector (jsonb)`, `current_mood`, `last_interaction_at`.
*   **Field Notebook**: `id`, `aviary_id`, `entry_text`, `timestamp`.
*   **Visit Invites**: `id`, `host_id`, `visitor_email`, `token`, `status`, `expires_at`.

### 3.2 Ephemeral / Fast Storage (Redis)
*   **Event Log**: List of recent interactions per account, drained by the Ticker.
*   **State Snapshots**: The latest canonical state for immediate delivery to clients.

---

## 4. API Surface

### Auth
*   `POST /auth/request-link`: Triggers magic link email.
*   `POST /auth/verify`: Consumes link, returns JWT and initial state snapshot.

### Aviary State
*   `GET /aviary/state`: Returns the latest snapshot. Used for initial load and re-sync.
*   `POST /aviary/events`: Appends interaction events (Type: `PRESENCE`, `LISTEN_IN`, `OFFER`, `SETTLE`).

### Social
*   `POST /social/invite`: Host issues an invitation.
*   `GET /social/visit/:token`: Visitor pulls read-only snapshot.

---

## 5. Simulation Engine Design

### 5.1 Personality Drift (The "Slow Current")
Drift is monotonic. `Trait_new = Trait_old + (Signal * Weight)`.
*   **Presence Signal**: 1 minute of validated presence = `+X` to Boldness/Social Warmth.
*   **Neglect**: No signal = 0 change. Birds never become more wary.
*   **Calibration**: Log-based dampening to ensure visible change takes ~3 weeks.

### 5.2 Mood System (The "Weather")
Mood is a state machine: `Alert -> Content -> Drowsy`.
*   **Transition Probability**: Influenced by `Local_Time` (e.g., Drowsy increases at sunset) and `Recent_Offer`.

### 5.3 Call Grammar
Birds don't play MP3s; they follow motifs.
*   **Motif**: A sequence of pitch/timing deltas.
*   **Variation**: Random jitter applied to pitch (±5%) and timing (±10%) at runtime.

---

## 6. Sync and Consistency

*   **No Client Authorship**: Clients never compute drift. They only report duration/type of interaction.
*   **Interpolation**: Clients receive "Perch A" and "Perch B". The frontend animator uses a spring-based interpolation to move the bird, ensuring 60fps movement even if the server tick is slow.
*   **Snap-to-State**: On return from background, the client requests a new snapshot and hard-syncs to the current server time to prevent "time travel."

---

## 7. Frontend Rendering Pipeline

*   **Scene Composition**: A single HTML5 Canvas (WebGL context).
*   **Idle Micro-motions**: Procedural "breathing" (scale pulsing) and "scanning" (head rotation) driven by per-mood shaders.
*   **Day/Night**: A full-screen gradient overlay with `mix-blend-mode: multiply` for night and `soft-light` for golden hour, tied to local `Date.now()`.
*   **Reduced Motion**: If `prefers-reduced-motion` is true, disable path animations. Swap `requestAnimationFrame` updates for 2-second cross-fades between keyframe poses.

---

## 8. Audio Pipeline (WebAudio)

*   **Synthesis**: Use `OscillatorNode` + `GainNode` envelopes to simulate bird calls.
*   **Spatialization**: `PannerNode` maps birds to left/center/right based on their horizontal scene position.
*   **Listen-In Mix**: When a bird is focused, we apply a 2-second linear ramp to its `GainNode` to 1.0, while ducking other birds to 0.1.

---

## 9. Accessibility Surfaces

*   **Narration Engine**: A client-side template system that converts the latest `StateSnapshot` into naturalist sentences (e.g., `{bird_name} is {mood_description}`). Injected into an `aria-live="polite"` region.
*   **Captions**: Absolute-positioned `<div>` elements over the canvas, tracking bird coordinates, displaying the call motifs as text (e.g., "a low trill").
*   **Keyboard**: Tab-index management for birds. `Space/Enter` triggers `LISTEN_IN`.

---

## 10. Performance Budgets

*   **Bundle**: <2MB Gzipped. Avoid heavy frameworks; use standard Web APIs.
*   **Load Time**: Initial snapshot is embedded in the first HTML response to hit the <500ms first-bird-visible target.
*   **Memory**: Strict object pooling for audio nodes and render particles to prevent GC pauses.

---

## 11. Rollout and Risks

### Rollout
*   **Internal Alpha**: 2 birds, static weather.
*   **Beta**: 5 birds, full drift function, multi-device sync.
*   **v1 Launch**: 7 birds max, social visits, accessibility narration.

### Risks
*   **Sync Race Conditions**: If a user interacts on two devices simultaneously. *Mitigation: Server processes the event log sequentially; last-processed event defines the current mood.*
*   **Audio "Uncanniness"**: Procedural calls sounding too "beep-boop." *Mitigation: High-pass filters and subtle noise injection in WebAudio envelopes.*
*   **Drift Calibration**: Too fast = Tamagotchi; too slow = Static. *Mitigation: Server-side toggle to adjust drift weights without client deployment.*
