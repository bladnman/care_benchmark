# Pocket Aviary - Implementation Plan

## 1. Scope
**In Scope for v1:**
- Web-only browser-based client (latest 2 versions of major browsers).
- Single horizontal aviary scene with 3 perch zones, day/night cycle, and ambient weather.
- 2 starter birds per aviary, expanding up to 7 over time based on account age.
- Procedural bird calls synthesized via WebAudio.
- Single-user accounts with magic-link email auth.
- Cross-device sync via server-side simulation tick.
- Interactions: Notice/Return-greeting, Listen-in, Offer (seed, song, pool), Settle, Field notebook.
- Social: Read-only visits via one-time email invite.
- Accessibility: Screen-reader narration, reduced-motion mode (cross-fades), procedural call captions, full keyboard navigation, WCAG AA contrast.

**Out of Scope:**
- Native apps (iOS/Android).
- Gamification (streaks, levels, points, leaderboards).
- Tamagotchi mechanics (hunger, death, negative drift from neglect).
- Social networks (public discovery, chat, comments, co-presence, avatars).
- Panning, zooming, or customizing the scene layout.

## 2. Architecture
The system follows a thin-client, thick-server architecture to enforce the "aviary continues without the viewer" rule and prevent sync conflicts.
- **Client:** A web frontend (React with Canvas/WebGL or pure DOM/CSS depending on rendering tests, plus WebAudio). Responsible for rendering state snapshots, interpolating motion, procedural audio synthesis, and sending interaction events.
- **Server:** A set of backend services (Node.js/Go) providing the Auth API, Snapshot API, and Event Intake API.
- **Simulation Engine:** A background worker process that runs a continuous "tick" (approx. 1/min) across all active aviaries, computing drift and mood transitions regardless of client connectivity.
- **Database:** PostgreSQL/CockroachDB for relational data (accounts, birds, personality vectors), backed by an event store (Kafka/Redis streams) for processing client interaction events.

## 3. Data Model
- **Account:**
  - `account_id`: Synthetic UUID (primary key across all systems).
  - `email`: Encrypted, used strictly for auth and export/visit invites.
  - `created_at`: Timestamp.
- **Bird:**
  - `bird_id`: Stable UUID.
  - `account_id`: Foreign key.
  - `name`: User-assigned string.
  - `species`: Enum defining visual silhouette and motif library.
  - `personality_vector`: JSON/struct storing scalar values for Boldness, Social Warmth, Vocal Frequency, Plumage Saturation, Curiosity. (Persisted server-side only).
  - `current_mood`: Enum (wary, content, curious, drowsy, alert).
- **Event Log:**
  - Append-only store of user interactions (presence ping, listen-in, offer, settle).
- **Field Notebook:**
  - `entry_id`, `account_id`, `timestamp`, `prose_content` (naturalist voice).

## 4. API Surface
- **Auth:** `/auth/request-link`, `/auth/verify` (issues expiring session tokens).
- **State Pull:** `/api/aviary/snapshot` -> Returns current layout, bird positions, moods, active animations. Polled on visibility change, render gaps, and slow keepalive.
- **Event Push:** `/api/aviary/events` -> Accepts batch of interaction events.
- **Offers:** `/api/aviary/offer` -> Submits an offer (seed, song, pool), checking cooldowns.
- **Social:** `/api/visits/invite` (generates email), `/api/visits/revoke`.
- **Notebook:** `/api/notebook/entries` (paginated read-only list).

## 5. Simulation Engine Design
- **The Tick:** A server-side chron job running every minute. It drains the event log for an account, computes additive deltas to the personality vector, and determines mood.
- **Presence Accounting:** A presence event requires `visibilityState === 'visible'` AND `document.hasFocus()` AND pointer/key activity within the last N minutes. The client sends a presence ping periodically when all three hold.
- **Drift Function:** Low-pass filter. Personality traits drift monotonically towards expressiveness. Drift is calculated based on presence-time, listen-ins, and offers. Neglect results in zero drift, not negative drift.
- **Mood Transitions:** Evaluated during the tick based on recent events in the log, local time of day, ambient weather, and current personality vector.
- **Notebook Generator:** An LLM-assisted or template-driven service triggered occasionally by the tick when notable state changes occur, generating specific, lowercase, present-tense observations.

## 6. Sync Model
- **Single Canonical State:** The server is the absolute source of truth.
- **Additive Deltas:** Clients never send absolute state (e.g., "set boldness to 0.5"). Clients only send actions. The server computes the resulting state changes. This prevents last-write-wins conflicts between devices.
- **Interpolation:** The client requests a state snapshot. If state changes between snapshots (e.g., bird moved), the client interpolates the movement rather than snapping.

## 7. Frontend Rendering Pipeline
- **Initial Load:** No spinners. The first frame renders birds mid-motion based on the snapshot. If the snapshot is delayed, a quiet empty field is shown.
- **Scene Composition:** 3 depth planes (front, middle, back perches). Parallax is minimal.
- **Lighting & Weather:** Client derives palette shifts from the local time of day. Ambient leaves/feathers are rendered via a client-side particle system independent of the server tick.
- **Reduced Motion:** If `prefers-reduced-motion` is true, the engine swaps animation loops (preening, flying) with slow cross-fades between static poses.
- **Top Bar:** Fades to near-transparent on cursor idle. Reappears on movement/focus.

## 8. Audio Pipeline
- **Synthesis:** Handled via WebAudio API. No recorded loops.
- **Motif Library:** Each species has an abstract grammar of call motifs. Runtime synthesis varies pitch, timing, and sequence based on the bird's current mood and vocal frequency trait.
- **Chorus Mechanic:** Independent bird audio nodes run simultaneously without phase-cancellation since generation is procedural.
- **Listen-in Mix:** When a bird is focused, an envelope generator slowly ramps up its audio node's gain and applies a slow decay to other birds' gains, returning to baseline on blur.
- **Fallback:** If WebAudio fails or is disabled, the system fails gracefully to silence and enables textual call captions.

## 9. Accessibility Surfaces
- **Screen Reader Narration:** A dedicated live-region aria element that periodically receives generated naturalist prose describing the scene (e.g., "a warbler perches on the high branch, calling softly."). Rate-limited to prevent queue flooding.
- **Captions:** Procedural call descriptions ("a soft three-note rise") rendered near the bird.
- **Keyboard Nav:** Focus ring with high contrast. Tab navigation through top bar -> birds. Enter to listen-in. Esc to cancel.
- **Contrast:** WCAG AA compliance enforced for all UI chrome and text overlays.

## 10. Performance Budgets & Observability
- **Budgets:**
  - Initial JS Bundle: < 2MB (gzipped). Code-splitting required for settings, visit logs, etc.
  - Time-to-first-bird: < 500ms on mid-tier mobile 4G.
  - Framerate: 60fps idle motion on 5yo hardware.
  - Memory: Flat footprint over 30 mins (reuse audio buffers, cleanup unseen DOM nodes).
- **Observability:**
  - Strict privacy boundary: No per-bird or per-account interaction telemetry.
  - Metrics collected: Bundle sizes, render framerates, TTFB, simulation tick latency (p99 < 5s alarm), WebAudio error rates, API HTTP codes.

## 11. Rollout
- **V1 Launch:** Supports 2 birds out of the gate.
- **Ramping:** A cron job evaluates account age. As accounts mature (e.g., weeks/months), new species offers are placed in the user's flow until the hard cap of 7 birds is reached.
- **Instrumentation:** Synthetic browser tests and aggregate RUM from day one to monitor the 500ms TTFB and 60fps budgets.

## 12. Risks
- **Drift Calibration:** If drift is too fast, the product feels like a Tamagotchi. If too slow, it feels broken. Mitigation: Extensive internal dogfooding with time-accelerated simulation environments before launch to tune the low-pass filter coefficients.
- **Audio Uncanniness:** Procedural generation might sound robotic if the grammar is too simple. Mitigation: Invest heavy frontend time in the WebAudio motif synthesis and randomized timing offsets.
- **Sync Correctness:** Dropped events from offline clients. Mitigation: Robust local queueing for interaction events with eventual delivery to the append-only log, ensuring the server eventually factors them into the tick.
- **Accessibility:** Screen reader narration sounding robotic. Mitigation: Treating narration generation as a core writing task, sharing the generation logic with the Field Notebook.
