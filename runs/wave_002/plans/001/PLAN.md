# Implementation Plan: Pocket Aviary (v1)

## Scope
Pocket Aviary v1 includes core simulation, multi-device sync, naturalist-voice interface, and read-only social visits. Non-goals strictly respected: no native app, no gamification (streaks, achievements), no Tamagotchi-style distress mechanics, and no public social-network surfaces.

## Architecture
- **Client**: React (TypeScript) with procedural rendering (canvas + WebAudio).
- **Server**: Node.js/FastAPI handling simulation ticks (event-log processor) and state snapshot delivery.
- **Boundary**: Client sends interaction events to event log; server tick consumes events and updates canonical state snapshot for clients.

## Data Model
- **Birds**: Canonical personality vector (stored on server).
- **Event Log**: Append-only list of interactions (offer, listen-in, etc.).
- **Snapshots**: Periodic state (per-bird position, mood, animation frame).

## API Surface
- GET /snapshot: Fetch current aviary state.
- POST /events: Submit interaction event (offer, settle, etc.).
- Auth via magic-link.

## Simulation Engine
- Server-side tick processes event logs to calculate drift and mood updates.
- Mood transitions are stochastic functions applied to personality vectors.

## Sync Model
- Canonical state lives on the server.
- Clients pull snapshots; server simulation tick is the only writer.
- Additive server-authored deltas ensure no last-write-wins conflicts.

## Rendering & Audio Pipeline
- Procedural rendering at 60fps, interpolation between snapshots.
- WebAudio-based procedural synthesis for bird calls, with caption fallback.

## Accessibility
- First-class narration (naturalist prose).
- Reduced-motion mode (cross-fades for micro-motion).
- Captioning for procedural calls.

## Performance
- Bundle < 2MB; TTFB < 500ms; No memory growth over 30min session.

## Rollout
- CDN-backed state snapshots. Telemetry restricted to aggregate operational metrics (latency, error rates).

## Risks
- Drift calibration complexity.
- Audio synthesis uncanny valley.
- Accessibility design vs. standard ARIA automation.
