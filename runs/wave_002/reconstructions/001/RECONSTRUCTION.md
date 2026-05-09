## System-level intent

- **Core but bounded v1 scope.** The plan defines Pocket Aviary v1 as "core simulation, multi-device sync, naturalist-voice interface, and read-only social visits" while saying non-goals are "strictly respected." The product intent is a focused first version with named exclusions: "no native app," "no gamification," "no Tamagotchi-style distress mechanics," and "no public social-network surfaces."

- **Canonical server state with the server tick as authority.** This shows up in Architecture and Sync Model: the "Client sends interaction events to event log," the "server tick consumes events and updates canonical state snapshot," "Canonical state lives on the server," and the "server simulation tick is the only writer."

- **Append-only interaction history becomes snapshots and deltas.** The plan repeatedly connects the "Event Log" to state production: an "append-only list of interactions" feeds a server-side tick that calculates "drift and mood updates," then produces "state snapshot delivery" and "Additive server-authored deltas."

- **Naturalist prose instead of game or distress pressure.** The Scope names a "naturalist-voice interface" and excludes "gamification (streaks, achievements)" and "Tamagotchi-style distress mechanics." Accessibility reinforces the voice with "First-class narration (naturalist prose)."

- **Procedural, web-delivered sensory experience.** The Client is React with "procedural rendering (canvas + WebAudio)," Rendering & Audio targets "60fps" and "interpolation between snapshots," and Performance sets "Bundle < 2MB," "TTFB < 500ms," and "No memory growth over 30min session."

- **Accessibility is part of the experience surface, not only compliance.** The plan gives Accessibility its own section with "First-class narration," "Reduced-motion mode," and "Captioning for procedural calls." Rendering & Audio also includes "caption fallback," and Risks call out "Accessibility design vs. standard ARIA automation."

- **Constrained social and telemetry surfaces.** The plan includes "read-only social visits" but excludes "public social-network surfaces." Rollout similarly restricts telemetry to "aggregate operational metrics (latency, error rates)."

## Per-feature whys

### Scope

- **Core simulation:** The plan ties this to the Simulation Engine: server-side ticks process event logs to calculate "drift and mood updates," with mood transitions applied to personality vectors.

- **Multi-device sync:** The rationale is server authority: "Canonical state lives on the server," clients "pull snapshots," and "Additive server-authored deltas ensure no last-write-wins conflicts."

- **Naturalist-voice interface:** The plan carries this through Accessibility as "First-class narration (naturalist prose)" and pairs it with exclusions of "gamification" and "Tamagotchi-style distress mechanics."

- **Read-only social visits:** The plan includes visits while also excluding "public social-network surfaces," so the recoverable rationale is social presence without a public social-network surface.

### Architecture

- **React (TypeScript) client:** NOT RECOVERABLE FROM PLAN

- **Procedural rendering (canvas + WebAudio):** The Rendering & Audio Pipeline explains this as procedural visual and audio output: "Procedural rendering at 60fps" plus "WebAudio-based procedural synthesis for bird calls."

- **Node.js/FastAPI server:** NOT RECOVERABLE FROM PLAN

- **Server handling simulation ticks:** The server is responsible for "simulation ticks (event-log processor)" so the canonical state changes happen through the tick rather than directly on clients.

- **State snapshot delivery:** The server delivers snapshots so clients can fetch or pull current aviary state via "/snapshot" and interpolate between snapshots.

- **Client/server boundary:** The plan's reason is authority separation: "Client sends interaction events to event log; server tick consumes events and updates canonical state snapshot for clients."

### Data Model

- **Birds as canonical personality vector:** The rationale appears in Simulation Engine, where "Mood transitions are stochastic functions applied to personality vectors."

- **Event Log as append-only list of interactions:** The event log is what the server-side tick processes to calculate "drift and mood updates"; examples include "offer" and "listen-in."

- **Snapshots as periodic state:** Snapshots carry "per-bird position, mood, animation frame," are fetched by clients, and support interpolation between snapshots.

### API Surface

- **GET /snapshot:** The plan states this fetches "current aviary state" and the Sync Model says clients "pull snapshots."

- **POST /events:** The plan states this submits interaction events such as "offer" and "settle," which feed the Event Log consumed by the server tick.

- **Auth via magic-link:** NOT RECOVERABLE FROM PLAN

### Simulation Engine

- **Server-side tick:** The rationale is to process event logs into "drift and mood updates" under server authority.

- **Drift updates:** The plan ties drift to server-side event-log processing, but also names "Drift calibration complexity" as a risk.

- **Mood updates and stochastic mood transitions:** The plan connects mood to personality by saying transitions are "stochastic functions applied to personality vectors."

### Sync Model

- **Canonical state lives on the server:** The reason is cross-client consistency, expressed by "server simulation tick is the only writer" and avoidance of "last-write-wins conflicts."

- **Clients pull snapshots:** The plan's rationale is snapshot-based sync: clients receive canonical state through snapshots rather than writing state directly.

- **Server simulation tick is the only writer:** This preserves the canonical state model and keeps updates server-authored.

- **Additive server-authored deltas:** The explicit rationale is to "ensure no last-write-wins conflicts."

### Rendering & Audio Pipeline

- **Procedural rendering at 60fps:** The plan states the target rendering behavior but does not give a separate product rationale beyond the procedural client experience.

- **Interpolation between snapshots:** This bridges periodic server snapshots with the 60fps rendering loop.

- **WebAudio-based procedural synthesis for bird calls:** The plan's rationale is procedural bird-call generation, while Risks identify "Audio synthesis uncanny valley."

- **Caption fallback:** The plan ties this to procedural calls and Accessibility's "Captioning for procedural calls."

### Accessibility

- **First-class narration (naturalist prose):** This carries the naturalist-voice interface into accessibility as "First-class narration."

- **Reduced-motion mode:** The plan states this uses "cross-fades for micro-motion," so the rationale is motion reduction while preserving micro-motion transitions.

- **Captioning for procedural calls:** This gives a caption surface for the procedural bird calls generated in the audio pipeline.

### Performance

- **Bundle < 2MB:** NOT RECOVERABLE FROM PLAN

- **TTFB < 500ms:** NOT RECOVERABLE FROM PLAN

- **No memory growth over 30min session:** NOT RECOVERABLE FROM PLAN

### Rollout

- **CDN-backed state snapshots:** NOT RECOVERABLE FROM PLAN

- **Telemetry restricted to aggregate operational metrics:** The plan says telemetry is limited to "aggregate operational metrics" and gives "latency" and "error rates" as the recoverable purpose.
