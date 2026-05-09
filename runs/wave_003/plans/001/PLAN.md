# Pocket Aviary — v1 Implementation Plan

## Scope
- v1 supports single-user, multi-device aviary sync.
- Starts with 2 starter birds, max 7 total.
- Includes procedural audio/animation engine, drift/mood systems, field notebook, visit invitations, and accessibility surfaces.
- Explicitly excludes native mobile apps, payments, shared aviaries, social network surfaces, and all forms of gamification (streaks, badges, leaderboards).

## Architecture
- **Client/Server Split**: Client (browser) handles rendering, WebAudio synthesis, and local input capture. Server (simulation engine) manages canonical state, simulation tick, drift calculation, and persistence.
- **Render Pipeline**: Snapshots are pulled from the server; client interpolates poses for smooth micro-motion and transitions.
- **Service Shape**: Simulation service (canonical state), event log (interaction ingestion), and public API for client state polling.

## Data Model
- **Birds**: Stable internal ID, assigned Name, Species (static), Personality Vector (scalar traits: Boldness, Social warmth, Vocal frequency, Plumage saturation, Curiosity), Current Mood (Enum).
- **Aviary**: Canonical record containing bird list, lighting state (day/night), weather, and presence metrics.
- **Events**: Append-only log of user interactions (listen-in, offer, presence pings) for server-side processing.

## API Surface
- `GET /aviary/snapshot`: Returns current canonical aviary state for client rendering.
- `POST /aviary/events`: Accepts interaction events (presence, offers, settle).
- `POST /auth/magic-link`: Sign-in/registration.

## Simulation Engine
- Server-side tick (~1 min): Processes event log, advances personality drift, updates mood transitions, recalculates bird positions/activities.
- **Drift Function**: Monotonic low-pass filter toward expressive based on presence-time, listens, and offers.

## Sync Model
- Canonical state owned by the server. Clients read snapshots. No conflict resolution needed for state (server is source of truth). Event log order determines state advancement.

## Rendering & Audio Pipeline
- **Rendering**: Procedural idle motion (preening, head-tilting) + parallax-aware scene composition. No entry animations.
- **Audio**: WebAudio synthesis via procedural call grammar (motif-based). Listen-in mix dynamically re-balanced.

## Accessibility
- **Narration**: Naturalist prose stream for screen readers, generated from aviary state.
- **Reduced Motion**: Designed alternate rendering path using cross-fades for motion instead of animation.
- **Captions**: Real-time prose descriptions of procedural calls.

## Performance
- **JS Bundle**: < 2MB (gzipped).
- **Time-to-First-Bird**: < 500ms on mid-tier mobile.
- **Runtime**: 60fps idle motion; zero memory growth over 30min.

## Rollout & Risks
- **Rollout**: Ship web-only v1, instrument performance metrics (TTFB, frame rates).
- **Risks**: Audio uncanniness (unavoidable phase-canceling or repetition), drift calibration (too fast/slow), sync correctness, accessibility register alignment.
