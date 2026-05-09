# Plan: Pocket Aviary Implementation (Wave 002, Run 001)

This plan details the implementation of the "Pocket Aviary" virtual bird system, adhering to the naturalist design philosophy ("feels alive," "notice never announce," "specificity," "restraint").

## 1. Scope
- **Included**: Browser-based aviary, bird simulation (personality drift, mood), procedural audio (WebAudio), magic-link auth, multi-device sync, read-only visitor feature, field notebook, accessibility (narration, captions), reduced-motion mode.
- **Excluded (per Non-Goals)**: Native apps, gamification (streaks, badges, scores), Tamagotchi mechanics (no decay/punishment), social network surfaces (discovery, profiles, public feeds).

## 2. Architecture
- **Client**: React + TypeScript, Canvas API for rendering (aviary), procedural audio via WebAudio API.
- **Server**: Node.js (Express), simulation engine (server-side tick).
- **Data Model**: Postgres (or similar SQL DB) for accounts, birds, and interaction history. Synthetic UUIDs for all account/bird references.
- **Render Boundary**: Client pulls snapshots of canonical aviary state; client performs interpolation.

## 3. Data Model & Sync
- **Canonical State**: Per-account, persisted server-side.
- **Sync**: Server-side simulation ensures clients pull the same snapshot (canonical state). No client-side merge or conflict resolution.
- **Event Log**: Append-only event log for client interactions (offer, listen-in, etc.). Server consumes log via simulation tick.

## 4. Simulation Engine
- **Tick**: ~1 minute cadence. Computes drift (personality vectors) and mood transitions.
- **Drift**: Presence-time based; monotonic toward expressive; non-symmetric (neglect does not cause negative drift).
- **Mood**: Fast-timescale, daily-ish reset; influenced by time-of-day, recent interactions, ambient events.

## 5. Frontend Pipeline
- **Rendering**: Horizontal scene, single-screen, no scrolling. Three perch zones.
- **Performance**: <2MB JS bundle, <500ms TTFB (time-to-first-bird), 60fps idle motion, no memory growth (leak-proof).
- **Accessibility**: First-class accessibility (naturalist narration, designed reduced-motion mode, call captioning).

## 6. Audio Pipeline
- **Procedural Calls**: Synthesized client-side (no recorded audio). Recognizable per-bird motifs.
- **Listen-in**: Gradual mix re-balancing (not solo/mute).

## 7. Rollout & Observability
- **Shipping**: Single-user, web-only.
- **Instrumentation**: Aggregate operational telemetry only (latency, error rates, frame times). No per-bird state in telemetry.
- **Alerts**: p99 simulation-tick latency > 5s alerts engineering.

## 8. Risks
- Drift calibration ("felt alive" vs "Tamagotchi").
- Audio synthesis variability across browser versions.
- Sync correctness (must preserve drift history).
- Ensuring performance targets (500ms TTFB) on mobile.
