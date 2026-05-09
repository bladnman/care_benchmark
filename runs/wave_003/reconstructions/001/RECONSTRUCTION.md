## System-level intent

1. Server-owned canonical state, with clients reading snapshots rather than owning simulation truth.
   This shows up in "Client/Server Split," where the server "manages canonical state, simulation tick, drift calculation, and persistence"; in "Service Shape," where the simulation service is "canonical state"; and in "Sync Model," where "Canonical state [is] owned by the server," "server is source of truth," and "Clients read snapshots."

2. Interaction is ingested as ordered events and then processed by the simulation engine.
   The plan names an "Append-only log of user interactions" and later says the server-side tick "Processes event log." The sync rationale is also event-order based: "Event log order determines state advancement."

3. Expressive birds are procedural, state-driven, and tied to presence, listens, and offers.
   The scope includes a "procedural audio/animation engine" plus "drift/mood systems." The simulation engine "advances personality drift" and "updates mood transitions," while the drift function moves "toward expressive based on presence-time, listens, and offers." Rendering and audio are also procedural: "Procedural idle motion" and "WebAudio synthesis via procedural call grammar."

4. The aviary is deliberately bounded away from native mobile apps, payments, shared aviaries, social network surfaces, and gamification.
   Scope explicitly excludes "native mobile apps, payments, shared aviaries, social network surfaces, and all forms of gamification (streaks, badges, leaderboards)." That boundary sits beside "single-user, multi-device aviary sync" and "web-only v1."

5. Accessibility is treated as a product surface rather than a late add-on.
   Scope includes "accessibility surfaces." The Accessibility section gives dedicated surfaces for "Narration," "Reduced Motion," and "Captions," and the risks include "accessibility register alignment."

6. Performance is a first-class v1 constraint.
   The plan sets explicit targets for "JS Bundle," "Time-to-First-Bird," and "Runtime," and the rollout says to "instrument performance metrics (TTFB, frame rates)."

## Per-feature whys

### Scope

- Single-user, multi-device aviary sync: NOT RECOVERABLE FROM PLAN
- Starts with 2 starter birds, max 7 total: NOT RECOVERABLE FROM PLAN
- Procedural audio/animation engine: The plan later grounds this in "Procedural idle motion" for rendering and "WebAudio synthesis via procedural call grammar (motif-based)" for audio.
- Drift/mood systems: The simulation engine "advances personality drift" and "updates mood transitions"; the drift function moves "toward expressive based on presence-time, listens, and offers."
- Field notebook: NOT RECOVERABLE FROM PLAN
- Visit invitations: NOT RECOVERABLE FROM PLAN
- Accessibility surfaces: The plan names screen-reader "Narration," "Reduced Motion," and "Captions" as dedicated surfaces.
- Native mobile apps exclusion: NOT RECOVERABLE FROM PLAN
- Payments exclusion: NOT RECOVERABLE FROM PLAN
- Shared aviaries exclusion: NOT RECOVERABLE FROM PLAN
- Social network surfaces exclusion: NOT RECOVERABLE FROM PLAN
- Gamification exclusion: NOT RECOVERABLE FROM PLAN

### Architecture

- Client/Server Split: The browser handles "rendering, WebAudio synthesis, and local input capture," while the server manages "canonical state, simulation tick, drift calculation, and persistence."
- Render Pipeline: Snapshots are pulled from the server so the client can interpolate poses for "smooth micro-motion and transitions."
- Service Shape: The plan separates simulation service for "canonical state," event log for "interaction ingestion," and public API for "client state polling."

### Data Model

- Birds stable internal ID: NOT RECOVERABLE FROM PLAN
- Birds assigned Name: NOT RECOVERABLE FROM PLAN
- Birds Species (static): NOT RECOVERABLE FROM PLAN
- Birds Personality Vector: The plan later uses personality in the simulation engine, which "advances personality drift," and defines the drift function toward expressive based on "presence-time, listens, and offers."
- Birds Current Mood: The plan later uses mood in the simulation engine, which "updates mood transitions."
- Aviary canonical record: The plan connects this to canonical server state and `GET /aviary/snapshot`, which returns "current canonical aviary state for client rendering."
- Aviary bird list: NOT RECOVERABLE FROM PLAN
- Aviary lighting state (day/night): NOT RECOVERABLE FROM PLAN
- Aviary weather: NOT RECOVERABLE FROM PLAN
- Aviary presence metrics: The drift function is based partly on "presence-time," and events include "presence pings."
- Events append-only log: The plan says events are for "server-side processing," the server-side tick "Processes event log," and "Event log order determines state advancement."

### API Surface

- `GET /aviary/snapshot`: Returns "current canonical aviary state for client rendering."
- `POST /aviary/events`: Accepts interaction events, including "presence, offers, settle," for the event log and server-side processing.
- `POST /auth/magic-link`: Supports "Sign-in/registration."

### Simulation Engine

- Server-side tick (~1 min): Processes the event log, advances personality drift, updates mood transitions, and recalculates bird positions/activities.
- Drift Function: Uses a "Monotonic low-pass filter" to move "toward expressive based on presence-time, listens, and offers."

### Sync Model

- Canonical state owned by the server: This makes the server the "source of truth."
- Clients read snapshots: The plan ties this to no client-side state ownership and to `GET /aviary/snapshot` for rendering.
- No conflict resolution needed for state: The plan gives the reason directly: "server is source of truth."
- Event log order determines state advancement: This explains how ordered interaction events advance state.

### Rendering & Audio Pipeline

- Procedural idle motion: The plan names "preening" and "head-tilting" as idle motion, and the render pipeline says interpolation supports "smooth micro-motion and transitions."
- Parallax-aware scene composition: NOT RECOVERABLE FROM PLAN
- No entry animations: NOT RECOVERABLE FROM PLAN
- WebAudio synthesis via procedural call grammar: The audio feature is motif-based: "procedural call grammar (motif-based)."
- Listen-in mix dynamically re-balanced: NOT RECOVERABLE FROM PLAN

### Accessibility

- Narration: Provides a "Naturalist prose stream for screen readers," generated from aviary state.
- Reduced Motion: Provides an "alternate rendering path using cross-fades for motion instead of animation."
- Captions: Provides "Real-time prose descriptions of procedural calls."

### Performance

- JS Bundle < 2MB (gzipped): NOT RECOVERABLE FROM PLAN
- Time-to-First-Bird < 500ms on mid-tier mobile: NOT RECOVERABLE FROM PLAN
- Runtime 60fps idle motion: NOT RECOVERABLE FROM PLAN
- Runtime zero memory growth over 30min: NOT RECOVERABLE FROM PLAN

### Rollout & Risks

- Web-only v1 rollout: NOT RECOVERABLE FROM PLAN
- Instrument performance metrics: The rollout names "TTFB" and "frame rates" as performance metrics to instrument.
- Audio uncanniness risk: The plan identifies "unavoidable phase-canceling or repetition" as the risk.
- Drift calibration risk: The plan identifies the risk as drift being "too fast/slow."
- Sync correctness risk: NOT RECOVERABLE FROM PLAN
- Accessibility register alignment risk: NOT RECOVERABLE FROM PLAN
