# Pocket Aviary Implementation Plan - Phase 1

## 1. Scope and v1 Definition
Pocket Aviary v1 is a browser-based, naturalist virtual environment focused on observational relationships with 2-7 birds.

### Included in v1:
- **Core Engine:** Procedural bird simulation (personality drift, mood, grammar-based calls).
- **Environment:** Single-screen horizontal scene with 3 perch zones, day/night cycle, and ambient weather.
- **Interactions:** Return-greeting, Listen-in, Offer, Settle gesture.
- **Content:** Field Notebook (auto-generated naturalist observations).
- **Account/Sync:** Magic-link auth, server-side simulation tick, multi-device state sync.
- **Social:** One-time read-only visit invitations (opt-in).
- **Accessibility:** First-class screen-reader narration, reduced-motion mode, call captioning.

### Out of Scope (Explicit Non-Goals):
- Native apps (Web-only).
- Gamification (No streaks, levels, counters, or achievements).
- Custodial mechanics (No hunger, death, or distress meters).
- Social network surfaces (No profiles, discovery feeds, or public directories).

---

## 2. Architecture
The system follows a **Thin Client / Thick Server** model to ensure simulation continuity and sync correctness.

- **Client (Frontend):** React (TypeScript) SPA. Handles rendering, audio synthesis (WebAudio), and interaction logging. 
- **Server (Backend):** Node.js (TypeScript) API and Simulation Service.
- **Simulation Engine:** A server-side "Tick" service (advancing ~1/min) that processes interaction logs and updates canonical bird states.
- **Data Store:** PostgreSQL for account/bird state; Redis for high-frequency interaction event logging before tick processing.
- **Authentication:** Passwordless Magic-Link via SMTP/Third-party provider (e.g., Postmark/SendGrid).

---

## 3. Data Model

### Account & Aviary
- `Account`: `id` (UUID), `email` (encrypted), `created_at`, `settings` (JSON).
- `Aviary`: `id`, `account_id`, `created_at`, `last_tick_at`.

### Bird Model
- `Bird`: `id`, `aviary_id`, `name`, `species_type`, `created_at`.
- `PersonalityVector`: `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity` (all 0.0-1.0).
- `MoodState`: `current_mood` (enum: `wary`, `content`, `curious`, `drowsy`, `alert`), `last_transition_at`.

### Persistence
- `InteractionEvent`: `id`, `account_id`, `type` (`presence`, `offer`, `listen_in`), `payload`, `timestamp`.
- `NotebookEntry`: `id`, `aviary_id`, `prose`, `timestamp`.

---

## 4. Simulation Engine Design

### The Server-Side Tick
- **Interval:** 60 seconds.
- **Inputs:** Recent `InteractionEvents`, current `PersonalityVector`, Current Time (local to user), Weather Service.
- **Process:**
    1. **Calculate Presence:** Conjunction of `visibilityState`, focus, and activity events.
    2. **Personality Drift:** Apply low-pass filters to traits based on weighted inputs. Drift is monotonic toward "expressive."
    3. **Mood Transition:** Stochastic state machine influenced by personality, time of day, and recent interactions.
    4. **Generate Observations:** Periodic naturalist prose generation for the Notebook.
- **Output:** Updated canonical state snapshot.

### Drift Calibration
- **Measurable (DB):** ~1 week of regular presence.
- **Visible (UI):** ~3 weeks of regular presence.

---

## 5. API Surface

### Client-to-Server
- `POST /auth/request-link`: Send magic link.
- `GET /state/snapshot`: Returns full aviary state (birds, positions, moods, weather).
- `POST /events/log`: Submit batch of interaction events (presence pings, offers).
- `POST /social/invite`: Create visit invitation.

### Server-to-Client
- **Snapshots:** Delivered via polling or WebSocket for active sessions.
- **Narrative Updates:** Part of state snapshot for screen-reader consumption.

---

## 6. Sync Model
- **Canonical Source:** The Server.
- **Conflict Resolution:** No "Last-Write-Wins" on client state. The client only sends interaction intent; the server processes these linearly.
- **Interpolation:** Client interpolates bird positions and mood transitions between snapshots to avoid "snapping."

---

## 7. Frontend Rendering & Audio Pipeline

### Rendering (Canvas/WebGL)
- **Layering:** Parallax background, middle plane (birds/perches), top bar chrome (fading).
- **Animation:** Procedural idle micro-motion (preening, head-tilting) keyed to current mood.
- **Reduced Motion:** Cross-fades (3s duration) between key poses instead of frame-by-frame animation.

### Audio (WebAudio API)
- **Procedural Calls:** Motif-based synthesis using oscillators and filters. No samples.
- **Mixer:** 
    - **Ambient:** All birds at distance-weighted levels.
    - **Listen-in:** Linear ramp (2s) to focus one bird (+6dB) while lowering others (-12dB).
- **Chorus Logic:** Staggered start times for multiple callers to avoid mechanical synchrony.

---

## 8. Accessibility Surfaces
- **Narration:** `aria-live` region updated with naturalist prose every 30-60s.
- **Captions:** On-screen text for vocalizations ("a soft three-note rise") generated from the call grammar.
- **Keyboard:** Standard Tab/Arrow/Enter/Esc mapping for bird focus and interaction.
- **Contrast:** AA compliance for all UI text and icons.

---

## 9. Performance Budgets & Rollout

### Budgets
- **JS Bundle:** <2MB gzipped (Initial Load).
- **First Bird Visible:** <500ms on 4G.
- **Frame Rate:** 60fps steady (Idle).
- **Memory:** Zero growth over 30min session.

### Rollout Strategy
- **Alpha:** 2 birds per aviary, 100 invited users.
- **Beta:** Introduce Visit Invitations, 1000 users.
- **v1.0:** Full launch, monitor drift calibration and audio "uncanniness."

---

## 10. Risks & Mitigations
- **Drift Calibration:** Risk of birds changing too fast/slow. *Mitigation:* Daily DB audits of trait distributions; tuning constants in the Tick engine.
- **Audio Uncanniness:** Risk of procedural calls sounding "beep-y" or repetitive. *Mitigation:* Add jitter to timing/pitch; expand motif library; rigorous user testing of the chorus mix.
- **Sync Correctness:** Risk of interaction loss during flaky connections. *Mitigation:* Client-side event buffering and retry logic; server-side idempotency keys.
- **Accessibility Regressions:** Visual updates breaking narration flow. *Mitigation:* Automated testing of `aria-live` outputs against state changes.
