# Pocket Aviary - V1 Implementation Plan

## Scope

**In Scope for V1:**
- Single-user accounts with magic-link email sign-in.
- A single canonical aviary per account, syncing implicitly across multiple devices.
- Up to seven birds per aviary (starts with two) from a fixed pool of roughly six species.
- The procedural Bird Engine (personality vectors, monotonic drift based on presence, fast-timescale mood states).
- Procedural audio synthesis client-side using WebAudio.
- Single horizontal scene with day/night cycle matching the user's local timezone.
- Core interactions: sit and watch (presence), return-greeting, listen-in, offer, settle.
- Auto-generated field notebook in a naturalist voice.
- Read-only visit invitations (opt-in, revocable).
- First-class accessibility: screen-reader naturalist prose narration, reduced-motion cross-fade mode, call captioning, and WCAG AA contrast.
- Performance budgets: <2MB initial JS bundle, <500ms time-to-first-bird, 60fps idle on older machines, no memory leaks.

**Out of Scope for V1 (Non-Goals):**
- Native mobile applications (iOS/Android).
- Any gamification (scores, streaks, achievements, counters).
- Tamagotchi mechanics (no death, no hunger, no negative drift on neglect).
- Social network surfaces (no discovery feed, no leaderboards, no profiles, no co-presence, no chat).

## Architecture

**Service Shape:**
- **Frontend Client:** Web-only SPA (React/TypeScript or vanilla JS depending on rendering weight) served via CDN for fast `<500ms` TTFB. Uses WebAudio for procedural calls, Canvas/WebGL or optimized DOM for rendering.
- **Backend Service:** A Node.js or Go service handling auth, API requests, and the simulation tick. Stateless horizontally scalable workers.
- **Simulation Worker:** A cron-like distributed worker system that processes the simulation tick (approximately once per minute) for all active aviaries.
- **Database:** PostgreSQL or similar relational DB for persistence (Accounts, Aviaries, Birds, Notebook Entries, Event Logs).
- **Cache:** Redis for rate-limiting, session token management, and fast recent-event log buffering before the simulation tick.

**Client/Server Split & Render Pipeline Boundary:**
- **Server:** Sole owner of canonical state. Generates the personality vector, processes the event log to calculate drift and mood transitions, and writes the field notebook entries.
- **Client:** View layer only. Pulls canonical state snapshots, interpolates motion between states, and submits user interactions as append-only events. Synthesizes audio procedurally based on current state variables provided by the server.

## Data Model

All account references internally use a synthetic UUID. Email is stored encrypted and isolated.

- **Account:** `id` (UUID), `encrypted_email`, `created_at`.
- **Session:** `token`, `account_id`, `device_info`, `expires_at`.
- **Aviary:** `id`, `account_id`, `created_at`, `current_weather`.
- **Bird:** `id`, `aviary_id`, `species_id`, `user_assigned_name`, `personality_vector` (JSON/array: boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity), `current_mood` (enum), `perch_zone` (enum), `created_at`.
- **Interaction Event Log:** `id`, `aviary_id`, `event_type` (presence_ping, listen_in_start, offer_seed, etc.), `target_bird_id` (optional), `timestamp`.
- **Notebook Entry:** `id`, `aviary_id`, `prose_text`, `timestamp`.
- **Visit Invite:** `id`, `host_account_id`, `visitor_email_hash` (or encrypted), `token`, `status`, `expires_at`.

## API Surface

Clients communicate via a REST or GraphQL API over HTTPS.

- `POST /auth/magic-link`: Request a sign-in link.
- `POST /auth/verify`: Exchange magic link token for a session token.
- `GET /aviary/state`: Pull current canonical state snapshot (birds, moods, positions, time-of-day offsets).
- `POST /aviary/events`: Submit batch of interaction events (append-only). E.g., presence pings, offers.
- `GET /aviary/notebook`: Fetch paginated field notebook entries.
- `POST /aviary/invites`: Create a visit invite.
- `GET /visit/:token`: Fetch read-only ambient state snapshot for a visitor.

## Simulation Engine Design

- **The Tick:** A scheduled server-side process that runs ~once per minute per aviary. It consumes the unprocessed `Interaction Event Log` for that aviary since the last tick.
- **Drift Function:** A slow low-pass filter. Computes additive deltas based on presence time (dominant), listen-ins, and offers. Deltas are always positive (monotonic toward expressive) and are applied to the bird's `personality_vector`.
- **Mood Transitions:** Evaluates recent events + local time of day + ambient weather + bird personality to update the `current_mood` of each bird.
- **Event Log Consumption:** Once the tick processes the events and updates the Aviary/Bird tables, the events can be archived or discarded.

## Sync Model

- **No Client-Side State Ownership:** The client never computes personality drift or authoritative mood changes. It never submits absolute state values (no "last-write-wins" race conditions).
- **Snapshot Pulling:** Clients fetch the latest snapshot on load, on visibility change, and on a low-frequency keepalive.
- **Implicit Sync:** Because all clients (e.g., laptop and phone) read from the same server-side canonical database, they naturally remain in sync. The simulation tick advances the aviary regardless of which client is connected, or if no client is connected.

## Frontend Rendering Pipeline

- **Scene Composition:** Single viewport, horizontally responsive, no scrolling. Three distinct z-depth planes (background foliage, middle perches/birds, foreground parallax elements).
- **Idle Micro-Motion:** Mood-shaped idle animations (preening, scanning) that run continuously.
- **Transitions:** Client interpolates bird positions between server snapshots so birds fly/hop to new perches rather than teleporting.
- **Reduced-Motion Mode:** Triggered via OS preference or user setting. Replaces frame-by-frame micro-motion and flight paths with slow, graceful cross-fades between static poses. Retains color shifts.

## Audio Pipeline

- **Procedural Call Synthesis:** Uses WebAudio API. No static audio loops. The client contains a library of small audio motifs per species. Calls are dynamically constructed at runtime with pitch and timing variations influenced by the bird's `vocal_frequency` and `current_mood`.
- **Chorus Mixing:** Multiple birds calling simultaneously mix naturally without phase-canceling artifacts due to procedural variance.
- **Listen-In Mix Decay:** Focusing a bird smoothly ramps its specific gain node while applying a slow volume drop (but not full mute) to the ambient mix and other birds.
- **Fallback:** If WebAudio is unavailable or blocked, the aviary degrades gracefully to silence with call captions enabled by default.

## Accessibility Surfaces

- **Screen-Reader Narration:** The client provides a live ARIA live region (polite) that receives server-generated or client-constructed naturalist prose (e.g., "a small grey bird is perched on the front rail..."). Updates occur every 30-60 seconds, with priority bumps for explicit user interactions.
- **Captions:** Opt-in text overlays near calling birds describing the procedural audio (e.g., "a soft three-note rise").
- **Keyboard Navigation:** Full tab-indexing of the top bar, arrow-key navigation between birds, Enter to trigger listen-in. High-contrast focus rings.
- **Visuals:** WCAG AA contrast ratio minimum for all user-copy text and UI chrome.

## Performance Budgets & Observability

- **Budgets:**
  - Initial JS bundle must be strictly `< 2MB` gzipped.
  - Time-to-First-Bird (TTFB) `< 500ms` on mid-tier mobile (4G).
  - 60fps rendering during 30+ minute sessions with zero memory leaks (enforced in CI).
- **Observability:**
  - **Aggregate Telemetry:** Page load times, TTFB, tick latencies, WebAudio error rates, anonymized session durations.
  - **Privacy Boundary:** Telemetry *never* includes per-account interaction history, personality vectors, or PII.
  - **Alarms:** Server simulation tick p99 latency > 5 seconds triggers a critical alert.

## Rollout

- **Launch:** Ship V1 as web-only with two starter birds.
- **Ramping Birds:** Additional birds are offered to users based strictly on aviary account age (e.g., offer 3rd bird at 1 month), preventing gamified grinding.
- **Instrumentation from Day One:** Ensure synthetic performance checks and aggregate telemetry are active to monitor the 500ms TTFB and 5s tick latency budgets in production before general availability.

## Risks

- **Drift Calibration:** If the low-pass filter is too fast, the product feels like a Tamagotchi; if too slow, it feels unresponsive. *Mitigation:* Extensive internal playtesting with headless clients simulating weeks of interaction to tune the exact mathematical curves prior to launch.
- **Sync Correctness:** Dropped connection events might result in lost presence time. *Mitigation:* The append-only event log and robust retry logic on the client for `presence_ping` submissions.
- **Audio Uncanniness:** Procedural synthesis might sound robotic if motifs lack sufficient variance. *Mitigation:* Dedicated sound design passes and strict adherence to the "no loops" rule.
- **Accessibility Regressions:** Custom accessibility surfaces (prose narration, reduced-motion) might degrade as new features are added. *Mitigation:* Build these into the core automated testing suite, ensuring PRs fail if narration text generation or keyboard focus states break.