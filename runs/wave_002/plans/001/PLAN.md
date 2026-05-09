# Plan - Pocket Aviary (v1)

## Scope
- v1 Scope: 2-7 birds, browser-based, single-user accounts, multi-device sync, naturalist-voice field notebook, opt-in guest visits, accessibility (narration, captioning, reduced-motion).
- Out of scope: native mobile apps, gamification (streaks, badges), Tamagotchi mechanics (punitive hunger/death), social network features (discovery, profiles, public feeds).

## Architecture
- Client: React (TypeScript) + Vanilla CSS. WebAudio API for procedural calls. Render pipeline driven by snapshot interpolation.
- Server: Node.js (Express/FastAPI), Postgres.
- Canonical state: Server-side simulation tick (one-minute cadence) processes interaction event log to update persistent state. Clients read snapshots.
- Sync: Event log -> Server Tick -> Snapshot -> Clients. No client-to-client sync; server is the only writer.

## Data Model
- Account: UUID-based synthetic ID. Emails encrypted.
- Birds: Personality vector (boldness, social warmth, vocal frequency, plumage, curiosity), stable UUID, name, species.
- Simulation: Mood (state enum), presence events, interaction logs, notebook entries.

## Interaction & Simulation Engine
- Simulation Tick (1m): Consumes interaction log, computes drift deltas, updates mood, writes canonical state.
- Drift: Additive deltas from presence, listen-in, offers. Monotonic toward expressive.
- Mood: Daily-ish transitions, influenced by time, ambient events, recent interactions.
- Procedural Engine: Call-grammar (motifs + variation) synthesized via WebAudio.
- Presence: Conjunction of (visibilityState: visible AND focus AND pointer/key activity).

## Sync Model
- Canonical state on server. Client-side is state-less snapshot rendering.
- Event log is append-only. Tick order is canonical. No last-write-wins for personality.

## Frontend Rendering
- Snapshot interpolation for movement between perch zones.
- Idle micro-motion continuous (preening, head-tilting).
- Viewport responsiveness: Horizontal compression/expansion, no cropping.
- Loading: Empty state/quiet field while loading snapshot (no spinner).

## Accessibility
- First-class Narration: Naturalist prose generated at low cadence.
- Reduced-motion: Designed mode (cross-fades instead of animations).
- Captioning: Real-time procedural prose descriptions of calls.
- WCAG AA contrast for all UI/chrome.

## Performance
- Bundle: <2MB gzipped.
- TTFB (First Bird): <500ms on 4G mid-tier device.
- Runtime: 60fps idle on 5yr-old mid-range laptop. No memory growth over 30m.

## Rollout
- Web-only. Sync availability of new species based on aviary age.
- Telemetry: Aggregate-only (load, latency, errors, frames) with strict PII protection.

## Risks
- Drift calibration: Tuning too fast/slow.
- Audio: Uncanny valley in call synthesis.
- Sync: Clock drifts if client-side logic leaks.
- Accessibility: Narration queue overflow.
