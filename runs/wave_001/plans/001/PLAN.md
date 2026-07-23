# Implementation Plan: Pocket Aviary (v1)

## Executive Summary & Vision Alignment
Pocket Aviary is a modern, browser-based virtual aviary where a small group of animated birds (2 at adoption, up to 7 maximum) live in a single horizontal scene. The product prioritizes **felt-aliveness**, **naturalist voice**, and **presence-based relationship building** over traditional gaming mechanics, quests, streaks, or Tamagotchi-style custodial obligations.

This document details the engineering execution plan for v1 across system architecture, data models, simulation engines, sync pipelines, rendering/audio layers, accessibility, performance budgets, and operational rollouts.

---

## 1. Scope (v1 Boundaries & Non-Goals)

### 1.1 In-Scope for V1
* **Aviary Core**: 1 canonical horizontal scene per user account; supports 2 starter birds at adoption up to a hard cap of 7.
* **Authentication & Accounts**: Single-user email magic-link auth (15-min link expiration); session token management with device revocation; synthetic UUID account mapping; account export & 30-day soft deletion.
* **Bird Simulation & Engine**: Server-side 1-minute simulation tick advancing slow-timescale scalar personality vectors (boldness, social warmth, vocal frequency, plumage saturation, curiosity) and fast-timescale daily/session mood states (`wary`, `content`, `curious`, `drowsy`, `alert`).
* **Presence & Interactions**: Strict multi-signal presence accounting (document visibility + window focus + user activity threshold); return greetings (varied procedurally); progressive listen-in mix rebalancing; offers (seed, song fragment, still pool) with per-bird cooldowns; settle (evening lighting/quieting ritual with 5-second undo); read-only Field Notebook (sparse naturalist prose entries).
* **Sync Architecture**: Multi-device sync driven by a single server-authoritative simulation state and append-only event logging (no LWW for personality state).
* **Rendering & Audio Pipelines**: 60fps web canvas visual scene with ambient micro-motion, 3 perch depth zones, local day/night light transitions, weather events, and top-bar auto-fade; client-side WebAudio procedural call synthesis with motif libraries, dynamic choruses, and graceful silence fallback.
* **Social (Opt-In & Quiet)**: Single-user read-only visit invitation via email link; per-visitor revocable tokens; silent visit logging without push/toast notifications.
* **Accessibility & UX Voice**: Naturalist prose screen-reader narration (30–60s cadence); custom cross-fade reduced-motion mode; procedural call captions; WCAG AA contrast compliance; full keyboard navigation; dual-voice model enforcement (naturalist for product, matter-of-fact for system/errors).

### 1.2 Explicit Non-Goals & Strict Exclusions
* **Native Mobile Apps**: Web-only (Chrome, Safari, Firefox, Edge last 2 major versions).
* **Gamification & Engagement Mechanics**: Absolutely zero achievements, streaks, levels, scores, badges, adoption counters, green-dot calendars, XP, or visit-frequency counters.
* **Tamagotchi Custodial Mechanics**: No bird death, hunger meters, sickness, happiness decay, or negative personality drift on neglect (drift is monotonic toward expressive).
* **Social Network Surfaces**: No public feeds, discovery directories, global leaderboards, co-presence/avatars, shared cursors, or comments.
* **Recorded Audio / Stock Asset Fallbacks**: No recorded audio files or stock audio loops.

---

## 2. Architecture & System Topology

### 2.1 High-Level Component Topology
```
                  +-----------------------------------+
                  |      Browser Client (React/TS)    |
                  |  - Canvas / WebGL Renderer        |
                  |  - WebAudio Synthesis Engine      |
                  |  - Presence & Event Monitor       |
                  +-----------------+-----------------+
                                    |
                           HTTPS / WebSocket
                                    |
                  +-----------------v-----------------+
                  |          API Gateway              |
                  |  - Edge Snapshot Caching (CDN)    |
                  |  - Magic-Link Auth Verification   |
                  +-----------------+-----------------+
                                    |
          +-------------------------+-------------------------+
          |                                                   |
+---------v-----------------------+                 +---------v-----------------------+
|  Aviary State & Query Service   |                 |    Event Log Ingest Service     |
|  - Auth & Account Management    |                 |  - Append-only event ingestion  |
|  - State snapshot generation    |                 |  - Preserves per-account stream |
+---------+-----------------------+                 +---------+-----------------------+
          |                                                   |
          |         +-------------------------------+         |
          +-------->| Transactional Data Store (DB) |<--------+
                    |  - Accounts (Encrypted PII)   |
                    |  - Bird State & Vectors       |
                    |  - Interaction Event Log      |
                    |  - Field Notebook Records     |
                    +---------------+---------------+
                                    ^
                                    | Read/Write State
                    +---------------+---------------+
                    |  Server Simulation Tick Engine |
                    |  - Cron worker (~1 min tick)  |
                    |  - Drift Low-Pass Filter      |
                    |  - Mood Transition Evaluator  |
                    |  - Field Notebook Synthesizer |
                    +-------------------------------+
```

### 2.2 Client/Server Boundary Specification
1. **Server Authorization & Ownership**: The server exclusively owns account credentials, bird identity records, vector parameters, canonical simulation ticks, and Field Notebook generation.
2. **Client State Consumption**: Clients pull JSON state snapshots via HTTPS on load, visibility return, or keepalive ping, and receive real-time deltas over WebSocket during active sessions. Clients interpolate spatial perches and visual/audio motifs smoothly between snapshots.
3. **Event Submissions**: Client actions (`listen-in`, `offer`, `settle`, `presence-ping`) are posted as immutable interaction events to the event log. Clients never submit updated trait or vector states directly.

---

## 3. Data Model & Schema Specification

### 3.1 Relational / Document Schemas

#### Account Record (`accounts`)
* `account_id` (UUIDv4, PK): Synthetic account identifier.
* `email_encrypted` (TEXT): AES-256 encrypted email address (PII restricted).
* `created_at` (TIMESTAMPTZ): Account instantiation timestamp.
* `deleted_at` (TIMESTAMPTZ, NULLable): Soft-deletion timestamp (hard delete after 30 days).
* `settings` (JSONB): Accessibility preferences, notification toggles (`visit_notify_enabled`), reduced motion override.

#### Bird Record (`birds`)
* `bird_id` (UUIDv4, PK): Stable bird identifier across all renames/syncs.
* `account_id` (UUIDv4, FK -> `accounts.account_id`): Owner account reference.
* `name` (VARCHAR(64)): User-assigned bird name.
* `species_id` (VARCHAR(32)): Species identifier from the 6-species v1 pool (determines visual silhouette and call motif grammar).
* `adopted_at` (TIMESTAMPTZ): Adoption timestamp.
* `slot_index` (INT): Perch allocation index (0 to 6).

#### Personality Vector Record (`bird_personality_vectors`)
* `bird_id` (UUIDv4, PK, FK -> `birds.bird_id`).
* `boldness` (FLOAT, [0.0, 1.0]): Perch proximity bias.
* `social_warmth` (FLOAT, [0.0, 1.0]): Call-response and greeting probability.
* `vocal_frequency` (FLOAT, [0.0, 1.0]): Unobserved call interval & chorus propensity.
* `plumage_saturation` (FLOAT, [0.0, 1.0]): Visual richness multiplier (monotonic drift up).
* `curiosity` (FLOAT, [0.0, 1.0]): Offer investigation likelihood and head-tilt rate.
* `updated_at` (TIMESTAMPTZ): Last simulation update.

#### Mood & Runtime State (`bird_mood_states`)
* `bird_id` (UUIDv4, PK, FK -> `birds.bird_id`).
* `current_mood` (ENUM: `wary`, `content`, `curious`, `drowsy`, `alert`).
* `current_perch_zone` (ENUM: `front`, `middle`, `back`).
* `last_interaction_at` (TIMESTAMPTZ).
* `updated_at` (TIMESTAMPTZ).

#### Interaction Event Log (`interaction_events`)
* `event_id` (UUIDv4, PK).
* `account_id` (UUIDv4, FK -> `accounts.account_id`).
* `bird_id` (UUIDv4, NULLable, FK -> `birds.bird_id`).
* `event_type` (ENUM: `presence_tick`, `listen_in_start`, `listen_in_end`, `offer_given`, `settle_triggered`, `settle_undone`).
* `payload` (JSONB): Offer type (`seed`, `song_fragment`, `still_pool`), duration, etc.
* `created_at` (TIMESTAMPTZ): Event emission timestamp.

#### Field Notebook Entry (`notebook_entries`)
* `entry_id` (UUIDv4, PK).
* `account_id` (UUIDv4, FK -> `accounts.account_id`).
* `body_prose` (TEXT): Naturalist observation string (lowercase, present-tense).
* `created_at` (TIMESTAMPTZ).

---

## 4. Simulation Engine Design

### 4.1 Server-Side Simulation Tick Loop
The simulation engine executes a scheduled tick per active aviary approximately once per minute:
1. **Event Ingestion**: Ingest unprocessed entries from `interaction_events` since the last processed sequence.
2. **Presence Aggregation**: Calculate valid presence seconds based on `presence_tick` events matching strict triple-conjunction rules.
3. **Personality Drift Filter**: Compute additive trait updates using a low-pass exponential moving average:
   $$\Delta \text{Trait} = \alpha \cdot f(\text{PresenceTime}, \text{Interactions})$$
   Where $\alpha$ is calibrated such that numerical drift is detected in test instruments after 7 days of typical use and visible to the user after 21 days.
   * **Monotonic Constraint**: $\Delta \text{Trait} \ge 0$ for all traits (neglect does not reduce values).
4. **Mood State Machine & Transitions**: Evaluate time-of-day (user timezone offset), recent offers, and nearby bird alarm states:
   * Ambient rain $\rightarrow$ vocal frequency reduction.
   * Dusk $\rightarrow$ transition toward `drowsy`/`settled`.
   * High boldness $\rightarrow$ reduced probability of transitioning into `wary`.
5. **Notebook Generation Evaluation**: Check sparse trigger conditions (e.g., first greeting order shift of the week). If triggered, write naturalist prose entry.

### 4.2 Procedural Call Grammar Engine
* **Motif Definition**: Each species defines a context-free grammar of frequency sweeps, harmonic structures, trill lengths, and pause intervals.
* **Personality Modulation**: High `vocal_frequency` shrinks inter-call intervals; high `social_warmth` increases call-matching probability when another bird calls.
* **Real-Time Synthesis (Client WebAudio)**:
  * Audio nodes: Custom `OscillatorNode` chains combined with `GainNode` envelopes and subtle bandpass filters.
  * Spatialization: Pan and gain modulated smoothly based on perch zone (`front`, `middle`, `back`) and active `listen-in` focus state.

---

## 5. Sync & State Propagation Model

### 5.1 Snapshot Delivery & Interpolation
1. **Initial Hydration**: Client fetches `/api/v1/aviary/snapshot`. Server returns JSON snapshot containing scene time, current weather, active birds, their target perches, current moods, plumage vectors, and audio seed parameters.
2. **Client Interpolation**: Client local render engine moves birds along bezier paths between perch updates over multiple frames, avoiding positional snapping.

### 5.2 Event Logging & Concurrency Handling
* Clients emit interaction payloads to `/api/v1/events/append`.
* **No Last-Write-Wins (LWW) for Vectors**: Clients never transmit absolute trait scalar values. In case of multi-device usage (e.g., laptop and phone open simultaneously), both devices append events to the same immutable log. The server tick processes events strictly in timestamp order.

---

## 6. Frontend Layout, Rendering, & Audio Pipelines

### 6.1 Aviary Scene Layout & Render Pipeline
* **Single Horizontal Scene**: Fixed 16:9 canvas with responsive scaling (no horizontal panning or vertical scrolling).
* **Perch Zones**: 3 layered planes (`back`, `middle`, `front`).
* **Visual Atmosphere**: Smooth gradient sky mapping tied to local time; soft ambient particle loops (drift leaves, falling feathers) rendered client-side.
* **Initial Render Handshake**: First frame renders birds mid-pose in current simulation state. Loading fallback is a quiet ambient field (no loading spinners).

### 6.2 Interactive Controls & Top Bar
* **UI Controls**: Top bar contains Account/Settings, Accessibility Settings, Field Notebook, and Offer Affordance.
* **Top Bar Auto-Fade**: Fades to 0% opacity after 3 seconds of mouse inactivity; returns to 100% on movement or keyboard focus.

### 6.3 Audio Pipeline & Mix Controls
* **Ambient Chorus**: Procedural WebAudio soundscape with dynamic gain allocation.
* **Listen-in Mix Ramp**: Selecting a bird executes an exponential gain ramp: focused bird increases to +3dB relative to mix, while other birds soft-duck by -12dB over 1.5 seconds.
* **Audio Fallback**: If WebAudio is unavailable or muted, the app silently renders visual scene with procedural captions automatically enabled.

---

## 7. Accessibility & UX Voice Architecture

### 7.1 Screen-Reader Narration Subsystem
* **Live Region (`aria-live="polite"`)**: Receives slow-cadence naturalist prose updates every 30–60 seconds.
* **Event Priority**: User-triggered actions (greetings, offers, settle) yield immediate naturalist narration updates.
* **Prose Style**: Lowercase, present-tense, descriptive (e.g., *"a grey warbler sits on the low perch, preening softly in the morning light"*).

### 7.2 Reduced-Motion Mode
* Activated automatically via `prefers-reduced-motion` or manual settings toggle.
* Replaces frame-by-frame animation paths with soft, 1.2-second cross-fades between static perching and preening poses.
* Disables ambient leaf/feather particle drift; retains day/night light transitions with lengthened cross-fade curves.

### 7.3 Dual-Voice Enforcer Pattern
* **Product Surfaces** (Aviary, Notebook, Offer prompts, Narration): Naturalist field-notebook register.
* **System Surfaces** (Magic link auth, sync errors, settings, accessibility config): Matter-of-fact, clear, standard English phrasing.

---

## 8. Performance Budgets, Security, & Observability

### 8.1 Quantitative Performance Targets
* **Initial JS Bundle Size**: $< 2\text{ MB}$ gzipped (achieved via WebAudio code generation instead of audio samples, compact SVG bird rigs, and dynamic code splitting).
* **Time-to-First-Bird Visible**: $< 500\text{ ms}$ on 4G connections from edge CDN cached HTML/snapshot.
* **Frame Rate Target**: Stable 60fps idle rendering on 5-year-old mid-tier laptop hardware.
* **Memory Growth Budget**: 0 MB unaccounted leak growth across 30 minutes of continuous runtime.

### 8.2 Privacy & Security Hardening
* **PII Isolation**: Email addresses stored strictly in encrypted `accounts` table. Synthetic UUIDs used exclusively in internal service routing, logs, and telemetry.
* **Telemetry Boundary**: Operational metrics (API latency, render FPS, tick duration) collected without any per-bird state or per-user interaction parameters.

---

## 9. Rollout, Instrumentation, & Risk Mitigation

### 9.1 Delivery Phasing & Ramping
1. **Phase A (Infrastructure & Core Engine)**: Server simulation tick, relational schema, auth magic-link flow, synthetic UUID mapping.
2. **Phase B (Audio & Render Integration)**: Canvas rendering, WebAudio procedural call grammar, initial 2-bird adoption flow, presence accounting.
3. **Phase C (Interaction & Accessibility)**: Listen-in mix ramps, offers, settle ritual, Field Notebook generator, screen-reader narration, reduced-motion cross-fader.
4. **Phase D (Social & Verification)**: Read-only visit invitations, revocable tokens, performance budget regression testing, accessibility WCAG AA audit.

### 9.2 Critical Technical Risks & Safeguards
* **Drift Calibration Risk**: Overshooting drift speed turns birds into Tamagotchis.
  * *Mitigation*: Comprehensive automated regression test suite testing presence accumulator over synthetic 7-day and 21-day timeline streams.
* **WebAudio Audio Fatigue / Uncanniness Risk**: Repetitive audio loops break immersion.
  * *Mitigation*: Procedural motif variation generator enforcing non-repeating pitch offsets, harmonic jitter, and dynamic silence pauses.
* **Sync Conflict Data Loss**: Multiple open tabs overwriting personality vectors.
  * *Mitigation*: Immutable event-log architecture with server-authoritative tick execution; client writes zero vector data.

---
*End of Implementation Plan.*
