# Pocket Aviary - Phase 1 Implementation Plan

## Scope
**In Scope for v1:**
- **Core Experience:** Single-user, web-only virtual aviary. A single horizontal scene without panning or scrolling.
- **Bird Engine:** Two starter birds (cap at seven). Procedural calls (WebAudio client-side), mood-shaped idle motion, and long-term personality drift (boldness, social warmth, vocal frequency, plumage saturation, curiosity).
- **Interactions:** Idle attention (Presence), Return-greeting, Listen-in (audio mix adjustment), Offer (seed, song fragment, still pool), and Settle (soft session-end).
- **State & Sync:** Server-side simulation tick (~1/min) advancing canonical state. Single aviary per account. Multi-device sync by reading the same server state.
- **Accounts:** Single-user, magic-link email sign-in.
- **Features:** Read-only Field Notebook (auto-generated naturalist observations). Social Visit feature (opt-in, read-only ambient view via email link).
- **Accessibility:** Screen-reader narration (naturalist prose), reduced-motion mode (cross-fades instead of frame animations), call captioning, full keyboard navigation, WCAG AA contrast for UI.

**Out of Scope (Explicit Non-Goals):**
- Native mobile apps (iOS/Android).
- Gamification (streaks, achievements, scores, levels).
- Tamagotchi mechanics (hunger, death, negative drift on neglect).
- Social network surfaces (profiles, public discovery, chat, co-presence, leaderboards, public aviaries).
- Multi-aviary accounts, customizable scenes, payments.
- Real-time client-to-client sync or client-authored personality state.

## Architecture
**Service Shape:**
- **Client (Browser):** A lightweight rendering and audio synthesis engine. It pulls state snapshots, interpolates motion, synthesizes procedural audio via WebAudio, and submits interaction events to a server log. No local state authority.
- **Server:** The authoritative state machine.
  - **Auth Service:** Handles magic-link generation, verification, and session token management.
  - **Simulation Engine (Tick Worker):** A background process that periodically (e.g., ~1/min) processes the append-only event log for each active aviary to update mood, personality drift, and generate Field Notebook entries.
  - **API Server:** Serves state snapshots to clients and receives interaction events.
  - **Database:** Stores accounts, synthetic UUID mappings, aviary state, bird states (personality vectors, mood), and Field Notebook entries.

**Boundary:** The client is a dumb renderer of a smart server. The client never mutates personality or mood directly; it only appends to the event log.

## Data Model
- **Account:** `account_id` (synthetic UUID), encrypted email, session tokens.
- **Aviary:** `aviary_id`, `account_id`, `created_at`, `weather_state`, `time_of_day_offset`.
- **Bird:** `bird_id` (stable), `aviary_id`, `species_id`, `name`, `adopted_at`.
  - **Personality Vector (Hidden):** `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity` (scalars).
  - **Mood State:** `current_mood` (wary, content, curious, drowsy, alert), `mood_updated_at`.
- **Event Log (Append-Only):** `event_id`, `aviary_id`, `event_type` (presence_ping, offer_seed, listen_in_start, settle), `timestamp`, `bird_id` (optional).
- **Notebook Entry:** `entry_id`, `aviary_id`, `timestamp`, `text` (naturalist prose).
- **Social Invite:** `invite_id`, `host_account_id`, `visitor_email_hash`, `status` (active/revoked/expired), `created_at`.

## API Surface
- `POST /auth/magic-link`: Request sign-in link.
- `POST /auth/verify`: Verify link, return session token.
- `GET /api/aviary/state`: Returns the canonical snapshot (birds, positions, moods, weather, lighting). Used on load, visibility changes, and keepalive polling.
- `POST /api/aviary/events`: Submit a batch of recent interaction events (presence pings, offers, listen-in).
- `GET /api/aviary/notebook`: Fetch read-only Field Notebook entries (paginated).
- `POST /api/aviary/settle`: Trigger the settle gesture.
- `POST /api/social/invite`: Generate a visit link for an email.
- `DELETE /api/social/invite/:id`: Revoke an invite.
- `GET /api/visit/:invite_id`: Visitor endpoint to pull read-only state.

## Simulation Engine Design
- **Server-Side Tick:** A cron-like worker that iterates over active aviaries (~1/min). It reads the event log since the last tick.
- **Drift Function:** A slow low-pass filter. It applies additive deltas to personality vectors based primarily on accumulated `presence` time, and secondarily on specific interactions (e.g., listen-in increases `vocal_frequency`). Drift is strictly monotonic toward expressive (never decreases).
- **Mood Transitions:** Evaluates recent events, time-of-day, ambient weather, and base personality to transition the short-term mood. Mood persists across sessions.
- **Notebook Generator:** During the tick, evaluates conditions (e.g., "first time bird X greeted before bird Y") and sporadically generates a prose entry.
- **Call-Grammar Runtime:** While audio is synthesized client-side, the server determines the high-level call frequency and timing based on mood and `vocal_frequency` to ensure the snapshot dictates when a bird *should* call.

## Sync Model
- **Canonical Server State:** The server is the single source of truth.
- **No Client-Side Resolution:** Clients do not sync with each other and there is no last-write-wins for state. Clients only push events.
- **Multi-Device:** A phone and laptop open simultaneously will both poll `GET /api/aviary/state`. If the phone submits an `offer` event, the server processes it on the next tick, updates the state, and both devices pull the updated mood on their next snapshot request.

## Frontend Rendering Pipeline
- **DOM / Canvas:** Minimal UI chrome in HTML/CSS (top bar). The scene is rendered using Canvas or WebGL (e.g., PixiJS) to handle smooth parallax and sprite animations within the 2MB bundle budget.
- **Interpolation:** The client receives absolute positions and states from the server and interpolates movement (e.g., hopping between perches) to ensure continuous motion.
- **Idle Micro-Motion:** Runs continuously client-side based on the current mood enum.
- **Reduced-Motion Mode:** Swaps frame-by-frame animations and movement paths for slow cross-fades between static poses.
- **Loading:** No spinner. Loads into a quiet field background with motion already in progress upon state fetch.

## Audio Pipeline
- **Procedural Synthesis:** Uses WebAudio API. No static audio loops.
- **Motif Library:** Small base audio samples/oscillators are modulated at runtime (pitch, timing) based on bird species and personality.
- **Listen-In Mix Decay:** Focusing a bird smoothly interpolates the WebAudio gain nodes. The focused bird stays at 1.0, while others slowly ramp down to an ambient level (e.g., 0.2), never 0.0.
- **WebAudio Fallback:** If audio context is denied or fails, fallback to silence with captions. No recorded tracks.

## Accessibility Surfaces
- **Screen-Reader Narration:** A visually-hidden ARIA live region receives slow, naturalist prose updates (e.g., every 30-60s) describing the scene, prioritized by user interactions.
- **Captions:** Opt-in text overlays near calling birds, dynamically matching the procedural call ("a soft three-note rise").
- **Focus & Keyboard:** Full keyboard tab navigation for the top bar and birds. High-contrast focus rings.
- **Contrast:** WCAG AA minimum for all UI copy.

## Performance Budgets and Observability
- **Budgets:**
  - Initial JS bundle < 2MB (gzipped).
  - Time-to-first-bird < 500ms on mid-tier mobile 4G.
  - 60fps idle motion on a 5-year-old laptop.
  - Zero memory leaks over a 30-minute session.
- **Observability:** Aggregate telemetry only. Track page load, render-frame timings, simulation tick latency (alarm at p99 > 5s), and audio-context errors. **Strict Privacy:** No per-bird state or per-user interaction history is logged to analytics.

## Rollout
- **Launch:** Web-only, magic-link auth. Two birds per new account.
- **Ramping Birds:** As an aviary ages (measured in weeks/months, not engagement), the engine will unlock the ability to adopt up to 5 additional birds (cap of 7).
- **Instrumentation from Day One:** Client performance metrics and server tick latency monitoring to ensure the engine scales without degrading the "alive" feeling.

## Risks
- **Drift Calibration:** If presence tracking is too loose or the drift filter too aggressive, birds will change too fast, breaking the illusion of slow growth.
- **Sync Correctness:** Dropped events or out-of-order processing could stall personality drift.
- **Audio Uncanniness:** Procedural synthesis failing to sound organic, or the chorus mechanic becoming a cacophony with 7 birds.
- **Performance / Bundle Size:** Exceeding the 2MB budget with high-quality procedural audio logic or detailed bird sprites, causing load states that break the entry illusion.
