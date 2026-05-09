# PLAN.md — Pocket Aviary V1 Implementation Plan

This plan outlines the implementation of Pocket Aviary V1, a browser-based virtual aviary focused on ambient, evolving relationships between users and birds.

## 1. Scope
- **In Scope:** Two starter birds (cap at 7), magic-link authentication, multi-device sync, procedural bird engine (mood/personality drift), field notebook, presence-based interaction tracking, read-only visitor feature.
- **Out of Scope (Non-goals):** Games/gamification (streaks, scores), Tamagotchi mechanics (punishing neglect), social networks (profiles, discovery), native mobile apps.

## 2. Architecture
- **Client/Server Split:**
    - **Server:** Node.js simulation engine. Canonical state manager. Handles authentication, event log processing, and simulation ticks.
    - **Client:** React (TypeScript) + WebAudio + Canvas/SVG-based rendering. Stateless; pulls snapshots from the server.
- **Render Pipeline Boundary:** Client renders snapshots; interpolates states smoothly between snapshots.

## 3. Data Model
- **Account:** Synthetic UUID, encrypted email.
- **Bird:** Stable internal ID, personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), current mood (enum: wary, content, curious, drowsy, alert).
- **Presence:** A conjunction of `visibilityState`, `window focus`, and recent `pointermove/keypress`.
- **Event Log:** Append-only log of user interactions (offer, listen-in, settle).

## 4. API Surface
- `GET /snapshot`: Pull current canonical state.
- `POST /events`: Submit interaction event (presence ping, listen-in, offer).
- `POST /auth/magic-link`: Auth flow.

## 5. Simulation Engine Design
- **Tick:** Server-side simulation running once per minute.
- **Drift:** Low-pass filter over presence-time and interaction signals. Monotonic toward expressive.
- **Mood:** Fast-timescale state reset on a daily-ish cadence, modulated by time of day, personality, and events.

## 6. Sync Model
- Canonical state lives on the server.
- Clients consume snapshots.
- No client-to-client sync; no last-write-wins for personality (server-only authorship).

## 7. Frontend Rendering Pipeline
- 60fps idle micro-motion.
- Parallax background/foreground.
- Procedural animation (cross-fading still poses for reduced-motion).

## 8. Audio Pipeline
- Procedural call grammar using WebAudio.
- Chorus mixing through procedural combination.
- Slow ramp for listen-in mix balance.

## 9. Accessibility Surfaces
- **Narration:** Naturalist prose updated every 30–60s via screen reader.
- **Reduced-Motion:** Designed cross-fade sequence for motion.
- **Captioning:** Procedural prose captions for calls.
- **Keyboard:** Full reachability.

## 10. Performance Budgets
- **Bundle:** < 2MB (gzipped).
- **Load Time:** First bird < 500ms.
- **Runtime:** 60fps on 5-year-old hardware, no memory growth over 30 mins.

## 11. Rollout
- Web-only v1.
- Telemetry: Operational health only. No per-bird state in analytics.

## 12. Risks
- Audio uncanniness: Procedural grammar must avoid repetition.
- Drift calibration: Must not feel too slow or too fast.
- Sync correctness: Hard dependencies on server tick integrity.
