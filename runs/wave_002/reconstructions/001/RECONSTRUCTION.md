## System-level intent

1. Keep the aviary small, personal, and web-only for v1. The scope names "2-7 birds," "browser-based," "single-user accounts," and "multi-device sync," while rollout repeats "Web-only." This suggests a contained single-user product rather than a broad platform.

2. Preserve a calm field-notebook voice rather than game pressure or social competition. The scope includes a "naturalist-voice field notebook," while the out-of-scope list excludes "gamification (streaks, badges)," "Tamagotchi mechanics (punitive hunger/death)," and "social network features (discovery, profiles, public feeds)."

3. Make the server the canonical simulation authority. Architecture says "Canonical state: Server-side simulation tick" and sync says "Event log -> Server Tick -> Snapshot -> Clients." The sync model reinforces "Canonical state on server," "Client-side is state-less snapshot rendering," "No client-to-client sync," and "server is the only writer."

4. Treat bird personality as persistent expressive state, not a client-resolved conflict field. The data model gives each bird a "Personality vector" and "stable UUID," the simulation has drift "monotonic toward expressive," and the sync model says "No last-write-wins for personality."

5. Prefer quiet, continuous, non-jarring presentation. Frontend rendering uses "Snapshot interpolation," "Idle micro-motion continuous," "Horizontal compression/expansion, no cropping," and an "Empty state/quiet field while loading snapshot (no spinner)." Accessibility also specifies a "Designed mode" for reduced motion with "cross-fades instead of animations."

6. Make accessibility part of the core experience, not an afterthought. Scope includes "accessibility (narration, captioning, reduced-motion)," and the Accessibility section calls out "First-class Narration," "Reduced-motion: Designed mode," "Captioning," and "WCAG AA contrast."

7. Protect user data and measure only operational health. The data model says "UUID-based synthetic ID" and "Emails encrypted." Rollout telemetry is "Aggregate-only (load, latency, errors, frames) with strict PII protection."

8. Keep the experience lightweight and stable. Performance sets limits for "Bundle," "TTFB (First Bird)," "60fps idle," and "No memory growth over 30m," while risks identify drift, audio, sync, and accessibility failure modes.

## Per-feature whys

### Scope

- v1 Scope: 2-7 birds: NOT RECOVERABLE FROM PLAN
- browser-based: NOT RECOVERABLE FROM PLAN
- single-user accounts: NOT RECOVERABLE FROM PLAN
- multi-device sync: Supports the named sync model where clients read server snapshots; specific user-facing rationale is NOT RECOVERABLE FROM PLAN.
- naturalist-voice field notebook: Carries the calm field-notebook product voice and connects to "First-class Narration: Naturalist prose generated at low cadence."
- opt-in guest visits: NOT RECOVERABLE FROM PLAN.
- accessibility (narration, captioning, reduced-motion): Accessibility is first-class in scope and later expanded into "First-class Narration," "Reduced-motion: Designed mode," "Captioning," and "WCAG AA contrast."
- native mobile apps out of scope: NOT RECOVERABLE FROM PLAN
- gamification out of scope: The plan excludes "streaks, badges," supporting a non-gamified product shape.
- Tamagotchi mechanics out of scope: The plan excludes "punitive hunger/death," supporting a non-punitive aviary experience.
- social network features out of scope: The plan excludes "discovery, profiles, public feeds," supporting a single-user product rather than a public social network.

### Architecture

- React (TypeScript) + Vanilla CSS client: NOT RECOVERABLE FROM PLAN.
- WebAudio API for procedural calls: Supports the "Procedural Engine" that synthesizes "Call-grammar (motifs + variation)" via WebAudio.
- Render pipeline driven by snapshot interpolation: Supports smooth "movement between perch zones" from snapshots.
- Node.js (Express/FastAPI), Postgres server: NOT RECOVERABLE FROM PLAN.
- Server-side simulation tick on a one-minute cadence: Provides canonical processing of the "interaction event log" to update persistent state.
- Clients read snapshots: Keeps the client "state-less" and positions the server as "the only writer."
- Event log -> Server Tick -> Snapshot -> Clients: Establishes canonical ordering and avoids client-to-client sync.
- No client-to-client sync: Because the "server is the only writer."

### Data Model

- Account with UUID-based synthetic ID: Supports privacy by avoiding direct account identity in the model; paired with "Emails encrypted."
- Emails encrypted: Supports "strict PII protection."
- Birds with personality vector: Enables mood, drift, and expressive differences through "boldness, social warmth, vocal frequency, plumage, curiosity."
- Birds with stable UUID: NOT RECOVERABLE FROM PLAN
- Birds with name and species: NOT RECOVERABLE FROM PLAN.
- Simulation mood state enum: Supports mood updates in the simulation tick and "Daily-ish transitions."
- Presence events: Feed drift and interaction logic; presence is defined as visibility, focus, and pointer/key activity.
- Interaction logs: Consumed by the simulation tick to compute drift deltas, update mood, and write canonical state.
- Notebook entries: NOT RECOVERABLE FROM PLAN

### Interaction & Simulation Engine

- Simulation Tick (1m): Consumes the interaction log, computes drift deltas, updates mood, and writes canonical state.
- Drift: Additive deltas from presence, listen-in, offers: Supports personality/mood movement that is "monotonic toward expressive."
- Mood daily-ish transitions: Lets mood be influenced by "time, ambient events, recent interactions."
- Procedural Engine call-grammar: Creates calls from "motifs + variation" synthesized via WebAudio.
- Presence as visible + focus + pointer/key activity: Ensures presence means active visible engagement, not merely an open page.

### Sync Model

- Canonical state on server: Prevents client-side authority and keeps the server as the single source for simulation state.
- Client-side state-less snapshot rendering: Keeps clients reading snapshots rather than owning simulation state.
- Append-only event log: Supports canonical tick ordering.
- Tick order is canonical: Avoids conflicting personality resolution.
- No last-write-wins for personality: Protects personality from conflict resolution that would overwrite expressive state.

### Frontend Rendering

- Snapshot interpolation for movement between perch zones: Enables movement from snapshot states without client simulation authority.
- Idle micro-motion continuous: Gives the aviary ongoing life through "preening, head-tilting."
- Horizontal compression/expansion, no cropping: Preserves viewport responsiveness while keeping birds visible.
- Empty state/quiet field while loading snapshot: Keeps loading quiet and avoids a spinner.

### Accessibility

- First-class Narration: Provides "Naturalist prose generated at low cadence."
- Reduced-motion designed mode: Uses "cross-fades instead of animations."
- Captioning: Gives "Real-time procedural prose descriptions of calls."
- WCAG AA contrast: Ensures all "UI/chrome" meets the stated contrast target.

### Performance

- Bundle <2MB gzipped: NOT RECOVERABLE FROM PLAN
- TTFB (First Bird) <500ms on 4G mid-tier device: Supports fast first-bird availability on a constrained mobile network/device.
- 60fps idle on 5yr-old mid-range laptop: Keeps idle animation smooth on older hardware.
- No memory growth over 30m: Preserves runtime stability over a continuous session.

### Rollout

- Web-only rollout: Keeps v1 aligned with the browser-based scope.
- Sync availability of new species based on aviary age: NOT RECOVERABLE FROM PLAN.
- Aggregate-only telemetry: Measures "load, latency, errors, frames" while maintaining "strict PII protection."

### Risks

- Drift calibration: The risk is "Tuning too fast/slow," so calibration exists to control the pace of drift.
- Audio: The risk is "Uncanny valley in call synthesis," so audio synthesis needs to avoid unnatural calls.
- Sync: The risk is "Clock drifts if client-side logic leaks," reinforcing server canonical state and avoiding client-side simulation leakage.
- Accessibility: The risk is "Narration queue overflow," so narration must be managed to avoid overload.
