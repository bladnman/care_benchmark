# Pocket Aviary - Implementation Plan

## Scope
**In Scope for v1:**
- Browser-based virtual aviary (single horizontal scene, responsive viewport).
- Single-user accounts via email magic link.
- Two starter birds (max seven), procedural calls, personality drift (boldness, social warmth, vocal frequency, plumage saturation, curiosity) and mood transitions.
- Client-side procedural audio (WebAudio) with chorus mixing.
- Interactions: Return-greeting, listen-in, offer (seed, song fragment, pool), settle.
- Field notebook (auto-generated naturalist observations).
- Multi-device sync (server-authoritative simulation tick).
- Social: Read-only ambient visits via explicit, revocable email invitations.
- Accessibility: Screen-reader narration, reduced-motion mode (cross-fades instead of animations), call captions, WCAG AA contrast, keyboard navigation.
- Performance: <2MB initial JS bundle, <500ms time to first bird, 60fps idle on a 5-year-old laptop, no memory growth.

**Out of Scope (Non-Goals):**
- Native mobile apps (web-only for v1).
- Gamification (no achievements, streaks, levels, scores, badges).
- Tamagotchi mechanics (no death, hunger, distress; no negative drift on neglect).
- Social network surfaces (no profiles, follows, public feeds, leaderboards, mutual visits, or chat).

## Architecture
- **Client/Server Split:** Thick client for rendering and audio synthesis; authoritative server for persistent state, event log processing, and simulation ticking.
- **Service Shape:** 
  - **Auth/Account Service:** Handles magic links, session tokens, and identity management (using synthetic UUIDs exclusively internally).
  - **Simulation Service:** Runs the ~1 min tick, processing client event logs, applying personality drift, and updating mood.
  - **API/Sync Service:** Serves state snapshots to clients and ingests interaction events into an append-only log.
- **Render Pipeline Boundary:** The server dictates "where the birds are and what they are doing" (snapshots). The client handles tweening/interpolation, procedural micro-motion, rendering weather, and procedural call synthesis.

## Data Model
- **Account:** synthetic UUID (primary key), encrypted email, session tokens, visit log.
- **Aviary State:** Account ID, list of birds, current weather, local timezone anchor.
- **Bird:** 
  - **Identity:** Stable internal ID, species ID, user-assigned name.
  - **Personality Vector:** Boldness, social warmth, vocal frequency, plumage saturation, curiosity (all scalar values).
  - **Mood State:** Current mood (wary, content, curious, drowsy, alert), mood timer.
  - **Positional/Render State:** Current perch zone (front, middle, back), current animation state.
- **Event Log:** Append-only log of user interactions (presence ping, listen-in start/end, offer, settle).
- **Notebook Entries:** Auto-generated text in naturalist voice, timestamp, account ID.
- **Visits:** Invitation tokens, visitor email, expiration, revocation status.

## API Surface
- `POST /auth/magic-link`: Request a sign-in link.
- `POST /auth/verify`: Verify token and establish session.
- `GET /api/state`: Pull current aviary snapshot (birds, moods, positions, weather). Returns lightweight JSON.
- `POST /api/events`: Submit interaction events (append to log). Handled asynchronously.
- `GET /api/notebook`: Fetch read-only field notebook entries (paginated).
- `POST /api/social/invite`: Generate and send an invitation email.
- `DELETE /api/social/invite/{id}`: Revoke an invite.

## Simulation Engine Design
- **Tick Runtime:** A periodic job processes the event log for each active aviary every ~1 minute.
- **Drift Function:** Low-pass filter updating personality vectors. Increases monotonic traits (e.g., plumage, boldness) based on accumulated presence-time and interactions. Does not degrade on absence. Presence is strictly defined by visibility, window focus, and recent pointer/key activity.
- **Mood Transitions:** Evaluates recent events, local time of day, weather, and personality vectors to transition moods. Writes new moods to the canonical state.
- **Call-Grammar Runtime:** Server provides the overarching mood and vocal frequency trait; the client generates specific motif combinations and timings based on these parameters.

## Sync Model
- **Canonical State:** The server is the absolute source of truth.
- **Conflict Prevention:** Clients never write state directly, enforcing an additive, server-authored deltas model. Clients only append to the event log. The server's simulation tick consumes the log and computes the resulting state.
- **Multi-Device:** Devices poll `/api/state` on visibility change or low-frequency keepalive. Since all state changes are server-authored, devices natively stay in sync without client-side merging.

## Frontend Rendering Pipeline
- **Scene Composition:** Single horizontal HTML5 Canvas/WebGL scene. Background (sky/foliage), middle (birds/perches), foreground (ambient passing leaves).
- **Idle Micro-Motion:** Client interpolates between server snapshots and applies procedural idle animations (preening, head-tilts) matching the current mood.
- **Transitions:** Smooth tweening between perches. No teleporting. 
- **Reduced-Motion Mode:** Swaps frame-by-frame animation for slow cross-fades between static poses.

## Audio Pipeline
- **Procedural Call Synthesis:** WebAudio API synthesizes calls from a motif library. Pitch and timing varied client-side based on the bird's personality and mood.
- **Chorus Mixing:** Procedural synthesis avoids phase-cancellation artifacts common when looping overlapping recorded audio.
- **Listen-in Mix Decay:** Focusing a bird smoothly ramps up its gain node while gently ducking the ambient and other birds' gain nodes.
- **Fallback:** If WebAudio is unavailable or denied, the app falls back gracefully to silence with captions enabled. No recorded audio fallback exists.

## Accessibility Surfaces
- **Screen-Reader Narration:** ARIA live region receives slow, naturalist prose generated based on current aviary state (e.g., "a warbler perches on the high branch...").
- **Captions:** Dynamic text overlays generated from the procedural call grammar, sharing the same naturalist voice.
- **Focus & Keyboard Navigation:** Tab navigation through UI and birds, high-contrast focus indicators.
- **Contrast:** Strict WCAG AA compliance for all text over the aviary and UI chrome.

## Performance Budgets and Observability
- **Budgets:** 
  - Initial JS bundle < 2MB (gzipped).
  - Time-to-first-bird < 500ms (via edge CDN delivery of initial state/HTML and aggressive code splitting).
  - 60fps rendering, strict memory management (reuse WebAudio buffers, clear detached DOM nodes).
- **Observability:** 
  - Synthetic checks and RUM for bundle size, render frame times, and simulation tick latency.
  - Strict privacy boundary: Telemetry never includes PII, per-bird state, or per-account interaction logs.

## Rollout
- **V1 Release:** Web-only launch with the core 6-species pool.
- **Ramping:** Every new account gets 2 starter birds. Third and subsequent birds unlock purely based on aviary age (e.g., month 3, year 1).
- **Instrumentation:** Monitor simulation-tick latency (p99 alarm at 5s) and audio synthesis performance from day one to ensure the "alive" feel holds up.

## Risks
- **Drift Calibration:** If drift is too fast, the app feels like a Tamagotchi; if too slow, a static screensaver. *Mitigation:* Fine-tune the low-pass filter weights so changes are perceptible only after roughly three weeks.
- **Sync Correctness:** Dropped events or delayed ticks could break the illusion of continuous life. *Mitigation:* Robust append-only event queue and exact timestamping for presence windows.
- **Audio Uncanniness:** Procedural audio might sound robotic if the motif grammar is too limited. *Mitigation:* Invest heavily in the WebAudio nodes and generative grammar to ensure organic variation.
- **Accessibility Regressions:** Treating accessibility as a designed surface means visual changes could break narrative flow. *Mitigation:* Screen-reader prose generation must be actively maintained alongside visual state transitions.
