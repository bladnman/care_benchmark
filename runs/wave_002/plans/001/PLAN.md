# Pocket Aviary - Implementation Plan

## Scope
- **In Scope (v1):** A web-only virtual aviary starting with 2 birds (scaling to a maximum of 7 based on aviary age). Features include single-user accounts via magic-link sign-in, server-side simulation ticking, multi-device sync via a single canonical server state, presence accounting, a read-only field notebook, procedural audio synthesis, screen-reader narration, reduced-motion mode, call captioning, and a read-only opt-in visit invitation system.
- **Out of Scope (v1 & Beyond):** Native mobile apps, gamification (no streaks, levels, or achievements), Tamagotchi mechanics (no hunger, death, or negative drift), and social network surfaces (no public directories, leaderboards, or co-presence).

## Architecture
- **Client/Server Split:** A thick client handles rendering, WebAudio procedural synthesis, and capturing user interactions. The server is the authoritative source of truth for the aviary simulation, executing a regular background tick to process events and update canonical state.
- **Render Pipeline Boundary:** The client pulls lightweight state snapshots from the server and handles all interpolation, micro-motion, and procedural variation (audio motifs and ambient particle drift). The server does not dictate frame-by-frame layout, only semantic state (e.g., current mood, perch zone).

## Data Model
- **Account:** Indexed by a synthetic UUID. Email is stored encrypted strictly for authentication and account exports. Per-device session tokens are managed here.
- **Bird:** Stable internal ID, user-assigned name, and species.
- **Personality Vector (Slow Timescale):** Normalized scalar values for Boldness, Social Warmth, Vocal Frequency, Plumage Saturation, and Curiosity.
- **Mood (Fast Timescale):** Enumerated states (e.g., wary, content, curious, drowsy, alert).
- **Interaction Event Log:** Append-only log containing event types (presence, listen-in, offer, settle), timestamps, and targeted bird IDs.
- **Notebook Entry:** Naturalist prose string and timestamp.
- **Visit Invitation:** Token, visitor email, host UUID, and expiration timestamp.

## API Surface
- **Auth:** `POST /api/auth/magic-link` and `POST /api/auth/verify`.
- **State:** `GET /api/aviary/snapshot` (returns bird positions, moods, and recent events).
- **Interactions:** `POST /api/events` (appends interaction events like presence ping, listen-in, offers to the server log).
- **Social:** `POST /api/invites` (create), `DELETE /api/invites/:id` (revoke), `GET /api/invites` (visit log).
- **Settings:** Account export generation and soft-deletion endpoints.

## Simulation Engine Design
- **The Tick:** A server-side cron job running at a slow cadence (~once per minute). It processes the append-only event log.
- **Drift Function:** A low-pass filter calculates personality drift monotonically upwards based heavily on presence-time and interactions.
- **Presence Definition:** Strict conjunction of `visibilityState == visible`, window focus, and recent pointer/keypress activity.
- **Mood & Notebook:** The tick updates mood based on local time, weather, and recent interactions, and occasionally generates naturalist field notebook entries using procedural prose templates.

## Sync Model
- **Canonical State:** The server owns all personality and mood state. Clients never write absolute state values, preventing last-write-wins conflicts.
- **Multi-Device:** Devices passively sync by pulling the same canonical snapshot. Event logs are processed sequentially by the server tick, ensuring deterministic state progression regardless of which device submitted the event.

## Frontend Rendering Pipeline
- **Scene Composition:** A single responsive horizontal scene with three depth planes (front, middle, back perches). Foreground and background layers feature subtle parallax.
- **Idle Micro-Motion:** Continuous, mood-shaped frame-by-frame animations (preening, scanning) that are active immediately upon load (no entry/loading animations).
- **Reduced-Motion Mode:** A specifically designed aesthetic replacing frame-by-frame motion and ambient particle drift with slow, calming cross-fades between poses.

## Audio Pipeline
- **Procedural Call Synthesis:** Handled entirely client-side via WebAudio API using a library of species-specific motifs. Timing and pitch are modulated by the bird's personality (vocal frequency) and mood.
- **Chorus & Listen-In:** Real-time mixing prevents phase-cancellation. The "listen-in" interaction triggers a slow, gradual volume ramp to focus on a specific bird while others fade to ambient levels.
- **Fallback:** Graceful silence with call captions if WebAudio is unavailable.

## Accessibility Surfaces
- **Screen-Reader Narration:** Sourced from semantic aviary state, delivering slow-cadence, naturalist prose updates (e.g., "a warbler perches on the high branch..."). User-initiated events receive priority queuing.
- **Captions:** Procedurally generated text descriptions of audio calls that fade in near the calling bird.
- **Navigation & Visuals:** Full keyboard navigability (Tab, Arrow keys, Enter, Escape) with high-contrast focus indicators. All user-copy text meets or exceeds WCAG AA contrast standards.

## Performance Budgets and Observability
- **Budgets:** 
  - Initial JS bundle < 2MB (gzipped).
  - Time-to-first-bird visible < 500ms on mid-tier mobile (4G).
  - 60fps idle motion on a 5-year-old laptop.
  - Zero memory growth over a 30-minute session.
- **Observability:** Aggregate telemetry only (request counts, latency, session-duration histograms, audio errors). 
- **Privacy Boundary:** Absolutely no per-bird state or interaction logs are included in aggregate telemetry. p99 simulation-tick latency must alarm if it exceeds 5 seconds.

## Rollout
- **V1 Launch:** Users start with exactly 2 birds from a pool of ~6 species.
- **Ramping:** Additional birds are offered based purely on aviary age, scaling up to a maximum of 7 over months to preserve per-bird audio recognizability.
- **Instrumentation:** RUM and synthetic performance checks deployed from day one to monitor the 500ms TTFB and 60fps runtime budgets.

## Risks
- **Drift Calibration:** If the low-pass filter for personality drift is too aggressive, the product feels like a Tamagotchi; if too slow, it feels unresponsive. Requires extensive internal testing.
- **Presence Signal Integrity:** Browser-level heuristics for window focus and visibility can be inconsistent across platforms. If presence is over-counted, population-wide drift will artificially accelerate.
- **Audio Uncanniness:** Procedural synthesis may result in robotic-sounding motifs or jarring chorus overlaps if the WebAudio mixing logic isn't perfectly tuned.
- **Accessibility Spam:** Screen-reader narration queues could become overwhelming if the pacing isn't strictly throttled, forcing users to mute it.