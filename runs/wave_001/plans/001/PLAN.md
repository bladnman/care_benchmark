# Comprehensive Implementation Plan: Pocket Aviary

## Scope
Pocket Aviary is a browser-based virtual aviary featuring animated birds that respond to user attention. v1 includes two to seven birds, multi-device sync, a field notebook, presence accounting, opt-in read-only visits, screen-reader narration, reduced-motion, and captioning. Out-of-scope: native apps, gamification (streaks, badges, levels), social network features (profiles, feeds), and Tamagotchi-style negative reinforcement (birds dying, decaying happiness).

## Architecture
- **Client**: React/TypeScript, SPA architecture.
- **Backend**: Node.js/FastAPI, simulation engine running server-side tick (~1/min).
- **Communication**: REST for snapshot delivery; append-only event log for interaction submissions.
- **Rendering**: Browser-based rendering (Canvas/SVG); client interpolates between server snapshots.

## Data Model
- **Account**: Synthetic UUID, magic-link auth.
- **Bird**: Canonical identity (UUID), species, personality vector (drift substrate), mood (current expression).
- **Notebook**: Append-only log of system-generated observations.
- **Events**: Append-only log of interaction gestures (offer, listen-in, presence).

## Simulation Engine
- **Server Tick**: Canonical simulation state advances based on interaction event log.
- **Drift**: Personality vector monotonically drifts toward expressive based on presence-time and interaction signals.
- **Mood**: Fast-timescale state influenced by personality, time-of-day, ambient events, and interactions.
- **Presence**: Conjunction of `visibilityState: visible`, window focus, and recent user activity (pointer/key).

## Sync Model
- **Server-Authoritative**: Server is the only writer of state.
- **No client-client sync**: All clients pull canonical snapshots from server.
- **Additive Deltas**: Personality updates are computed server-side from event log history to prevent race conditions.

## Frontend Rendering Pipeline
- **Scene**: Single horizontal scene, no panning/zoom.
- **Animation**: Idle micro-motion driven by mood.
- **Transitions**: Interpolation between canonical snapshots for smooth motion.
- **Reduced-Motion**: Cross-fading still poses instead of frame-animated paths.

## Audio Pipeline
- **Synthesized**: Procedural call synthesis using WebAudio.
- **Chorus**: Multi-bird chorusing via runtime-varied motif combination.
- **Listen-in**: Mix re-balance (focused bird raised, others quieted).

## Accessibility
- **Narration**: Naturalist prose narration generated for screen readers.
- **Captioning**: Prose descriptions of procedural calls.
- **Navigation**: Keyboard focusable perches/birds; top-bar shortcut reachability.

## Performance
- **Bundle**: <2MB (gzipped) initial JS load.
- **TTFBird**: <500ms on 4G mid-tier mobile.
- **Idle Motion**: 60fps on 5-year-old laptop.

## Risks
- **Drift Calibration**: Balancing drift speed (visible to user in ~3 weeks) vs. testing speed.
- **Audio Uncanniness**: Ensuring procedural synthesis feels alive.
- **Sync Correctness**: Maintaining event log order to avoid personality drifts from conflicting.
