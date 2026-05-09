## System-level intent

1. "Notice, never announce" is a scope boundary and a product voice rule. It shows up in the opening scope statement, in return-greetings that are "purely procedural visual/audio" with no textual "Welcome back," and in loading that shows a "quiet field" rather than a spinner.

2. "Feels alive, not robotic" is the other explicit scope boundary. It shows up in the Bird Engine, the periodic "Tick" as "The Heartbeat," ambient motion such as "preening" and "head-tilting," the sync risk that maintaining "Feels Alive" requires a reliable server tick, and the accessibility risk that narration prose must remain "naturalist and not robotic."

3. Birds should evolve from "user presence and small interactions," slowly enough to feel real. This shows up in presence-based drift, interaction event logs, the drift calibration target of "measurable in instruments at 1 week, visible to user at 3 weeks," and the risk that drift that is "too fast feels like a toy; too slow feels like a screensaver."

4. Absence is not punished. This shows up in the non-goal that birds "do not die, hunger, or show distress," the drift rule that values "never decrease due to neglect," and rollout gating where birds 3-7 are unlocked by account age "not engagement."

5. The aviary should have one reliable truth across devices. This shows up in the "Server-Side Canonical State" model, "thin clients rendering state snapshots," clients that "never write state," and the server as "the single source of truth for personality and mood."

6. Privacy and social boundaries stay quiet and opt-in. This shows up in "PII-Partitioned" account data, `email_encrypted`, "No bird-state or interaction history in aggregate telemetry," "One-time read-only visit invitations (opt-in only)," and "No Social Network Features."

7. Accessibility surfaces should preserve the same naturalist voice rather than becoming a separate mechanical layer. This shows up in "Naturalist screen-reader narration," prose such as "pip is fluffed against the cool air," "Procedural Captions," "Reduced-Motion" pose cross-fades, keyboard navigation, and the regression risk around narration staying naturalist.

8. Performance is part of the alive feeling. This shows up in the 2MB bundle, "Time-to-First-Bird" under 500ms, locked 60fps, "Zero memory growth over 30 mins," rendering from the first available snapshot frame, and the quiet-field cold-cache path.

## Per-feature whys

### Scope

- Pocket Aviary v1 / browser-based virtual aviary: The plan says the product is a virtual aviary "where birds evolve based on user presence and small interactions," bounded by "Notice, never announce" and "Feels alive, not robotic."

- Bird Engine: The plan groups personality vectors, mood, and procedural call grammar as the engine that makes birds evolve and feel alive rather than robotic.

- Personality vectors: The plan uses Boldness, Social Warmth, Vocal Frequency, Plumage Saturation, and Curiosity as drift targets shaped by presence signals; they also affect return-greeting selection, mood probability, and call cadence.

- Mood system: The plan uses mood to connect time of day, recent interactions, and personality to visible state and audio state; mood also shapes synthesizer timbre and pitch shift.

- Procedural Call Grammar: The plan uses motif libraries and WebAudio synthesis so calls can be species-specific and shaped by `vocal_frequency` and `mood`, while avoiding "repetitive artifacts."

- Presence-based drift: The plan makes presence part of bird evolution, but calibrates it so change is "measurable in instruments at 1 week" and "visible to user at 3 weeks."

- Return-greetings: The plan makes returning noticeable through one bird picked by `boldness` and `mood`, with other responses staggered, while keeping the voice "purely procedural visual/audio" and avoiding textual "Welcome back."

- Listen-in: The plan uses Listen-in to emphasize a focused bird with "Gain-node rebalancing" while retaining ambient "Aviary" noise.

- Offer: The plan treats Offer as a small interaction appended to the event log and says an "Offer success" can nudge mood toward `content`.

- Seed: NOT RECOVERABLE FROM PLAN

- Song: NOT RECOVERABLE FROM PLAN

- Pool: NOT RECOVERABLE FROM PLAN

- Settle gesture: NOT RECOVERABLE FROM PLAN

- Field Notebook: The plan gives it real-world timestamps, "Naturalist prose ID" content templates, bird names, and events, so its rationale is to render observed aviary events as naturalist prose records.

- Single horizontal responsive scene: NOT RECOVERABLE FROM PLAN

- Three perch zones: The plan attaches the zones to "z-depth," so the recoverable rationale is depth in the horizontal scene.

- Local-time-anchored day/night cycle: The plan uses solar position in the user's timezone as an input to mood transitions, grounding the aviary in local time.

- Ambient weather: NOT RECOVERABLE FROM PLAN

- Magic-link email auth: NOT RECOVERABLE FROM PLAN

- Multi-device sync via server-side canonical state: The plan says this keeps a canonical state on the server, avoids conflicts, and lets thin clients render snapshots.

- Server-side simulation tick: The plan calls the tick "The Heartbeat"; it updates active aviaries, consumes interaction event logs, writes canonical positions and moods, and must be reliable for "Feels Alive."

- Naturalist screen-reader narration: The plan uses a hidden live region with prose describing the aviary so screen-reader output remains in the naturalist voice.

- Reduced-motion mode: The plan replaces frame-by-frame animation with "2-second cross-fades between static poses," preserving the experience with less motion.

- Call captions: The plan uses captions to describe call character, such as "a low trill," and auto-enables them if WebAudio fails.

- Keyboard navigation: The plan provides full keyboard support so users can tab through chrome, use Arrow keys to switch bird focus, and press Enter to Listen-in.

- One-time read-only visit invitations: The plan constrains social use to "opt-in only" read-only visits while excluding public discovery, profiles, follows, and co-presence.

### Architecture

- Server-Side Canonical State model: The plan uses this model so thin clients render state snapshots while the server owns personality, mood, drift, and the canonical simulation tick.

- Simulation Service (The Engine): The plan has it run the periodic tick for all active aviaries and consume interaction event logs, making it the service form of "The Heartbeat."

- State API: The plan makes the API read-only and snapshot-based so clients can consume current aviary state without writing canonical state.

- Event Log Service: The plan makes interaction ingestion append-only so clients record Offers, Settle, Listen-in, and Presence without creating write conflicts.

- Static Asset CDN: The plan uses the CDN for the 2MB max frontend bundle, bird assets, and motif libraries, tying asset delivery to the performance budget.

- Server ownership of personality vectors, mood transitions, drift calculations, and canonical tick: The plan keeps these on the server so personality and mood have one source of truth.

- Client rendering, WebAudio synthesis, local interpolation, and presence measurement: The plan leaves the client with rendering, procedural call playback, interpolation of snapshots, and measuring presence, matching the "thin clients" split.

### Data Model

- Account (PII-Partitioned): The plan partitions account data around `account_id` as the primary key used elsewhere and stores `email_encrypted`, supporting the privacy boundary around PII.

- Bird stable identity: NOT RECOVERABLE FROM PLAN

- Species pool (6 species): NOT RECOVERABLE FROM PLAN

- User-assigned name: NOT RECOVERABLE FROM PLAN

### Simulation Engine Design

- Event Aggregation: The plan aggregates events since the last tick so presence time and offers can feed drift and mood updates.

- Drift Function: The plan applies additive deltas to personality vectors using `filter(presence_signals, weight)`, making evolution gradual and presence-based.

- Drift Calibration: The plan calls for tuning that is instrument-visible at one week and user-visible at three weeks, avoiding toy-like speed or screensaver slowness.

- Monotonic drift: The plan says values "never decrease due to neglect," carrying the no-punishment rule into the engine.

- Mood Engine: The plan transitions moods from time of day, recent interactions, and bird personality; examples include Offer success nudging `content` and high `boldness` reducing `wary` probability.

- State Snapshot Generation: The plan writes canonical positions and moods for client consumption after the tick.

- Motif Library: The plan uses species-specific "MIDI-like fragments" so calls can vary by species while remaining procedural.

- Synthesizer: The plan shapes WebAudio oscillators and filters by `vocal_frequency` and `mood`, connecting personality and mood to cadence, timbre, and pitch shift.

### Sync & Interaction Model

- Client Snapshots: The plan pulls every minute while visible or on visibility change so the client stays aligned with canonical state.

- No Conflicts: The plan avoids conflicts because clients "never write state" and only append to the event log.

- Presence Precision: The plan records presence only when `visible` + `focused` + `activity` in the last 3 minutes, making drift depend on active presence rather than mere open tabs.

- Heartbeat pings: The plan sends pings to the server for aggregation, feeding the tick with presence signals.

### Frontend Rendering Pipeline

- Ambient Motion: The plan uses "preening," "head-tilting," and leaf/feather drift to keep the scene continuously alive.

- Loading from the first available snapshot frame / quiet field: The plan starts rendering from the first snapshot and, on a cold cache, shows sky/perch rather than a spinner, preserving the quiet product voice.

- Audio Fallback: The plan auto-enables Call Captions and plays in silence if WebAudio fails, keeping call information available without broken audio.

### Performance & Observability

- Bundle Size: The plan sets a <2MB gzipped initial payload, matching the lightweight web scope and CDN asset plan.

- Time-to-First-Bird: The plan sets <500ms on 4G/mid-tier, making the first bird arrive quickly enough to support the alive feeling.

- Runtime: The plan requires locked 60fps on 5-year-old hardware and zero memory growth over 30 minutes, protecting long-running ambient use.

- Operational Telemetry: The plan measures tick latency, API latency, bundle size, FPS, and JS errors so reliability and performance can be observed.

- Privacy Boundary: The plan excludes bird-state and interaction history from aggregate telemetry, preserving the privacy boundary.

### Rollout & Risks

- Day 0 with 2 birds per account: NOT RECOVERABLE FROM PLAN

- Age-Based Gating: The plan unlocks birds 3-7 by account age "not engagement," preserving no gamification and no punishment for absence.

- Instrumentation in shadow mode: The plan validates drift monotonicity before GA, checking that the no-neglect-decrease rule holds before launch.

- Drift Calibration risk: The plan says drift that is too fast "feels like a toy" and too slow "feels like a screensaver," so instrument-based tuning is required.

- Sync Continuity risk: The plan says maintaining "Feels Alive" requires a highly reliable server tick.

- Audio Uncanniness risk: The plan says procedural calls must avoid phase-canceling and repetitive artifacts.

- Accessibility Regressions risk: The plan says narration prose must remain naturalist and not robotic during updates.
