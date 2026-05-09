# Implementation Plan: Pocket Aviary (v1)

## Scope

**In Scope for v1:**
- Web-only application (modern browsers).
- Single-user accounts with magic-link email sign-in.
- Single horizontal aviary scene, bounded to one screen, with a maximum of 7 birds (starting with 2).
- Procedurally generated audio calls with WebAudio synthesis.
- Server-side tick-based simulation for bird mood and slow personality drift.
- Client-side rendering of idle micro-motion and smooth state interpolation.
- The "Field Notebook": auto-generated, read-only naturalist observations.
- Core Interactions: Return-greeting, listen-in focus, offering (seed, song fragment, pool), and "settle" (soft session end).
- Opt-in, read-only social visit feature via email invitations.
- Strict privacy: No aggregation of per-bird or per-account interaction history.
- Comprehensive accessibility: Naturalist screen-reader narration, reduced-motion mode (cross-fade rendering), and procedural call captions.
- Responsive scaling preserving aspect ratio.

**Out of Scope (Non-Goals):**
- Native mobile applications (iOS/Android).
- Gamification (no achievements, no streaks, no scores, no badges).
- Tamagotchi mechanics (no death, hunger, or negative drift/punishment for neglect).
- Social networks (no profiles, public discovery, leaderboards, avatars, or co-presence).
- Customizations (no paid tiers, no custom scenes, no species selection catalog).
- Audio fallback to recorded loops (graceful silence with captions instead).

## Architecture

**Client/Server Split:**
- **Client (Frontend):** Thin, presentation-focused. Pulls state snapshots from the server and interpolates between them. Renders the visual scene, synthesizes procedural audio locally via WebAudio, and dispatches interaction events. Does not compute state or run the core simulation tick.
- **Server (Backend):** Authoritative source of truth. Handles authentication, runs the simulation tick (computing mood and personality drift), writes field notebook entries, and manages the append-only event log.

**Service Shape:**
- **Auth Service:** Issues and validates magic links, manages per-device session tokens, and handles account revocation.
- **Simulation Service:** A tick-driven worker that runs on a slow cadence (~1/minute). It reads the event log, advances the aviary state, and writes the new state to the database.
- **API Gateway/BFF:** Serves state snapshots to clients and ingests interaction events.

## Data Model

- **Account:** Contains an encrypted email, a synthetic UUID (primary key for all internal references), and settings (e.g., notification opt-in for visits).
- **Aviary:** Tied to a single account UUID. Holds the canonical state, timestamp of last simulation tick, local timezone offset, and current weather/day-night modifiers.
- **Bird:** Stable internal ID, assigned name, and species ID.
- **Personality Vector (Slow timescale):** Float values for Boldness, Social Warmth, Vocal Frequency, Plumage Saturation, and Curiosity. Updated solely by the server.
- **Mood (Fast timescale):** Enumerated state (e.g., wary, content, curious, drowsy, alert). Persisted across sessions.
- **Event Log:** Append-only store of interaction events (presence, listen-in, offers, settle) with timestamps.
- **Field Notebook:** Append-only log of generated naturalist observations, stored with timestamps.

## API Surface

- `GET /api/aviary/snapshot`: Client pulls the current state. Includes bird positions, moods, active animations, environmental state, and recent notebook entries.
- `POST /api/events`: Client submits interaction events (presence ping, listen-in start/stop, offer initiated, settle triggered). Events are appended to the log; the server responds with acknowledgment, not immediate state change.
- `POST /api/social/invite`: Host generates an invitation for a specified email.
- `DELETE /api/social/invite/:id`: Host revokes an invitation.
- `GET /api/social/visit/:token`: Visitor fetches the read-only ambient state of the host's aviary.
- `POST /api/auth/magic-link`: Request magic link.

## Simulation Engine Design

- **Server-Side Tick:** Runs asynchronously (~once per minute) per aviary. It aggregates the recent event log.
- **Presence Computation:** Confirms presence only if visibility, window focus, and recent input activity overlap. Settle or tab close ends the window.
- **Drift Function:** A low-pass filter over presence and interactions. Additive and monotonic (traits never decrease on neglect). Generates visible changes after ~3 weeks, measurable after ~1 week.
- **Mood Transitions:** Evaluated during the tick. Modulated by the latest interaction, time of day (derived from user timezone), weather events, and bounded by the bird's personality vector.
- **Call-Grammar Runtime:** The client runs the grammar rules based on the server-provided vocal frequency and mood parameters, selecting motifs and timing.

## Sync Model

- **Canonical State:** The server is the absolute source of truth. There is no client-side state merging.
- **Conflict Prevention:** Uses an event-sourcing model. Clients emit events rather than asserting state (e.g., "listen-in happened", not "boldness = 0.8"). The server's tick processes these linearly, preventing last-write-wins overwrites from multiple devices.
- **Propagation:** When a client opens or regains visibility, it pulls the latest snapshot. Devices are implicitly in sync because they read the same server-computed state.

## Frontend Rendering Pipeline

- **Scene Composition:** Three depth planes (background, middle with three perch zones, foreground). Bounded horizontally, responsive aspect ratio.
- **Animation:** Birds are placed at snapshot positions and interpolated. Idle micro-motion (preening, head tilts) runs continuously. No loading spinners; the scene begins in media res or with a quiet sky color on cold cache.
- **Reduced-Motion Mode:** A distinct designed aesthetic. Disables frame-by-frame animation and ambient drift, replacing them with slow, elegant cross-fades between still poses.
- **Top Bar:** Minimal UI chrome, fades to near-transparency on idle.

## Audio Pipeline

- **Procedural Call Synthesis:** WebAudio API used to stitch together small motif libraries at runtime. Timing and pitch are modulated by personality (vocal frequency) and mood.
- **Chorus Mixing:** Individual WebAudio nodes per bird. Mixing allows simultaneous calls without phase-canceling artifacts found in looped audio.
- **Listen-in Mix:** Soft, gradual volume automation (ramp up for focused bird, ramp down to ambient for others). Hard cuts are strictly avoided.
- **WebAudio Fallback:** If WebAudio is unavailable, the application defaults to graceful silence and enables procedural captions automatically. No recorded loops are served.

## Accessibility Surfaces

- **Screen-Reader Narration:** Server- or client-generated naturalist prose (e.g., "a small grey bird is perched on the front rail..."). Updates slowly (30-60s) to avoid queue flooding, with priority interrupts for user actions.
- **Captions:** Procedurally generated text mirroring the audio calls (e.g., "a soft three-note rise"), using the naturalist voice.
- **Keyboard Navigation:** Full keyboard support (Tab, Arrow keys, Enter, Escape) with high-contrast, designed focus indicators.
- **Contrast:** All user-facing UI text meets or exceeds WCAG AA standards.

## Performance Budgets and Observability

- **Time-to-First-Bird:** < 500ms on a mid-tier 4G mobile connection. Achieved via aggressive code-splitting and small snapshot payloads.
- **Bundle Size:** < 2MB gzipped initial JavaScript bundle.
- **Runtime Budget:** 60fps idle motion on a 5-year-old laptop. No memory growth over a 30-minute session (reused audio buffers, bounded queues).
- **Observability:** Synthetic checks, aggregate RUM (page load, render frame timing, audio context errors).
  - *Constraint:* Strict separation of operational telemetry from per-bird/account interaction history.
  - *Alarm:* Server tick latency > 5s at p99.

## Rollout

- **V1 Launch:** Web-only, single-user auth, 2 starter birds, full simulation and audio pipelines.
- **Bird Ramp-up:** New bird species are offered chronologically based on aviary age, capped at 7 birds.
- **Instrumentation Day 1:** Aggregated performance and error rates. No user behavior analytics or engagement tracking.

## Risks

- **Drift Calibration:** Too fast, and it feels like a Tamagotchi; too slow, and it feels dead. *Mitigation:* Extensive internal beta testing focusing purely on the 1-to-3-week timescale.
- **Sync Correctness:** Dropped events or out-of-order processing corrupting the personality vector. *Mitigation:* Robust append-only logging and idempotent simulation ticks.
- **Audio Uncanniness:** Procedural generation sounding robotic or repetitive. *Mitigation:* High variation in motif libraries and strict adherence to the no-loops rule.
- **Accessibility Regressions:** Falling back to checklist ARIA attributes instead of designed prose. *Mitigation:* Integration of narration and reduced-motion views in the core definition of done for all UI changes.