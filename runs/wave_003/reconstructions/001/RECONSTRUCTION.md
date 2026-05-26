# Reconstruction: Pocket Aviary (v1)

## System-level intent

- Observational relationship rather than game. The plan states this directly in Scope: "a browser-based, low-fidelity virtual aviary designed as an observational relationship rather than a game." It reappears in Non-Goals as "No Gamification," and in Risks where drift that is "too fast" would "feel like a game."
- Presence should create slow, expressive change. The plan pairs "monotonic drift based on presence" with a Drift Function that is "monotonic toward expressive" and calibrated so "1 week" gives measurable drift while "3 weeks" gives visible drift. The same intent shows up in the tick process, where `presence_time` is calculated from "visibility, focus, and activity."
- The aviary has a canonical life on the server. Architecture says the backend is "the source of truth," while the split says "Server owns" personality vectors, mood transitions, drift calculations, and canonical simulation time. Sync repeats this with "Only the server writes personality/mood."
- The client should be a thin, sensory surface. The frontend is described as "a thin rendering layer" that pulls snapshots, synthesizes audio/visuals, and writes interaction events. The Render Pipeline Boundary keeps the client mapping state snapshots to "visual assets and audio motifs."
- The experience should avoid custodial pressure and public social pressure. Non-Goals reject "hunger, death, or distress" and say it is "not a Tamagotchi." Social is limited to "opt-in, read-only visitor invitations via email," while "No Social Network" rejects profiles, follows, public discovery, and chat.
- Accessibility is part of the aesthetic, not a stripped version. The plan specifies "Naturalist prose" screen-reader narration, reduced-motion "cross-fade rendering," and call captioning. The accessibility risk mitigation says to "Design narration and reduced-motion as first-class aesthetic experiences."
- Lightweight immediacy matters. Scope sets "<2MB initial bundle," "<500ms time-to-first-bird," and "60fps idle motion." Rendering reinforces this with first frame birds "mid-action" and no "wake-up" animations; Observability instruments "Page load," "TTFB," render timing, and tick latency.

## Per-feature whys

### Scope

- Browser-based, low-fidelity virtual aviary: The plan frames it as an "observational relationship rather than a game."
- Single horizontal scene: NOT RECOVERABLE FROM PLAN
- 2-7 birds: NOT RECOVERABLE FROM PLAN
- Personality vectors: They are the substrate for "monotonic drift based on presence" and let traits move "toward expressive."
- Mood system: Mood drives transitions based on "time-of-day, ambient events, and recent interactions" and shapes "micro-motions" and call variation.
- Procedural calls: The plan wants a "unique chorus" from motifs varied in real time, while avoiding procedural calls that sound "robotic" or like "looped samples."
- Monotonic drift based on presence: Drift gives long-term visible change from presence and interaction without turning the aviary into a game or a screensaver.
- Return-greetings: They are treated as high-priority accessibility events with "Immediate updates for return-greetings."
- Listen-in focus: The audio pipeline gives it a "gradual mix re-balance" where the focused bird rises and others drop to ambient.
- Offer gestures: NOT RECOVERABLE FROM PLAN
- Settle session-end: NOT RECOVERABLE FROM PLAN
- Read-only Field Notebook: It preserves entries as timestamped "prose" in a "Naturalist voice."
- Single-user accounts with magic link: NOT RECOVERABLE FROM PLAN
- Server-side simulation tick: It lets the backend remain "the source of truth" for mood, drift, event processing, and canonical simulation time.
- Multi-device sync: Both laptop and phone pull the same `GET /state` snapshot.
- Opt-in, read-only visitor invitations via email: This gives a small social surface while honoring "No Social Network" and keeping visits read-only.
- Naturalist prose screen-reader narration: The plan wants narration in a "Naturalist prose" voice with slow cadence and priority updates, and says accessibility should be a "first-class aesthetic" experience.
- Reduced-motion cross-fade rendering: It replaces frame-by-frame animation and removes ambient drift so reduced motion is still an aesthetic version of the aviary.
- Call captioning: Captions translate procedural calls into descriptive prose such as "a low trill," and become the default when WebAudio is unavailable.
- Performance budgets: They protect the lightweight feel: "<2MB," "<500ms time-to-first-bird," "60fps," and "zero memory growth over 30 mins."
- No gamification: The rationale is explicit: the aviary is "rather than a game," so there are no streaks, achievements, levels, XP, or badges.
- No custodial mechanics: The rationale is explicit: it is "not a Tamagotchi," with no hunger, death, or distress.
- No social network: The plan limits social interaction to invite-only, read-only visits and rejects profiles, follows, public discovery, and chat.
- Web-only for v1: NOT RECOVERABLE FROM PLAN

### Architecture

- Frontend as thin rendering layer: It pulls state snapshots, synthesizes audio/visuals, and writes interaction events while the backend owns simulation.
- Backend simulation service: It is the "source of truth" for event log, canonical bird states, and periodic simulation ticks.
- Database for account metadata, encrypted emails, bird vectors, moods, and Field Notebook log: It persists the canonical data the server simulation needs. The plan gives privacy vocabulary for account data through "encrypted emails" and "non-PII" UUIDs.
- Server owns personality vectors, mood transitions, drift calculations, and canonical simulation time: This keeps core simulation canonical and server-authored.
- Client owns procedural synthesis, frame-by-frame interpolation, and presence detection: These are sensory and local activities that map snapshots into real-time audio/visual behavior and presence events.
- Render pipeline boundary: The snapshot-to-assets boundary keeps state transport small while allowing the client to map mood, perch, and call timing to motifs and visuals.

### Data model

- Synthetic UUID account id: The stated rationale is "non-PII."
- Encrypted email: The privacy rationale is carried by "Encrypted string."
- Account settings for accessibility preferences and notification toggles: They support accessibility preferences and notifications.
- Stable internal bird UUID: NOT RECOVERABLE FROM PLAN
- Species identifier from the species pool: NOT RECOVERABLE FROM PLAN
- User-assigned bird name: NOT RECOVERABLE FROM PLAN
- `boldness`: NOT RECOVERABLE FROM PLAN
- `social_warmth`: NOT RECOVERABLE FROM PLAN
- `vocal_frequency`: It drives call variation: pitch and timing vary based on the bird's `vocal_frequency` trait and current mood.
- `plumage_saturation`: It is part of the personality vector that can drift "toward expressive"; no more specific rationale is stated.
- `curiosity`: NOT RECOVERABLE FROM PLAN
- `current_mood`: Mood is used for transitions, micro-motions, and call variation.
- `perch_zone`: It maps to the Z-Axis of "Front, Middle, Back."
- Append-only event log: The tick reads events since last tick, and the mitigation for event loss is an "Append-only event log with server-side processing."
- Field Notebook entries with timestamp and prose: They preserve the "Naturalist voice" log over time.

### API surface

- `GET /state`: It returns the canonical snapshot used by the aviary, multi-device clients, and read-only visitors.
- `POST /events`: It appends interaction events so the server tick can process presence, offers, and other interaction signals.
- `POST /auth/request-link`: NOT RECOVERABLE FROM PLAN
- `GET /auth/verify`: NOT RECOVERABLE FROM PLAN
- `POST /account/export`: NOT RECOVERABLE FROM PLAN
- `POST /visit/invite`: It enables opt-in visitor invitations by email.
- `DELETE /visit/revoke`: It lets a specific invitation be invalidated, supporting opt-in visitor control.
- `GET /visit/{token}/state`: It provides read-only state for visitors, matching the plan's read-only social model.

### Simulation engine design

- Server-side tick frequency of about one minute: NOT RECOVERABLE FROM PLAN
- Reading the event log since last tick: It lets the server process interaction history into canonical updates.
- Calculating `presence_time` from visibility, focus, and activity: This operationalizes "presence" for drift and avoids using a single passive signal.
- Updating personality vectors using the drift function: This is how presence and interactions become long-term expressive change.
- Transitioning mood based on time-of-day, ambient events, and recent interactions: Mood reacts to both ambient context and interaction context.
- Advancing mood timers: NOT RECOVERABLE FROM PLAN
- Low-pass filter over presence and interaction signals: This smooths drift across signals before it changes personality vectors.
- Monotonic direction toward expressive traits: The plan says "traits only move up," making change additive and never punitive.
- One-week and three-week calibration targets: They keep drift from feeling "too fast" like a game or "too slow" like a screensaver.
- Species-specific call motifs: They make calls species-specific rather than generic.
- Client-side pitch and timing variation: Variation creates a "unique chorus" and mitigates "Audio Uncanniness."

### Sync model

- Server writes personality and mood: This preserves canonical state.
- Additive deltas: They prevent "last-write-wins" conflicts.
- Laptop and phone pulling the same `GET /state` snapshot: This gives multi-device consistency.

### Frontend rendering pipeline

- Single horizontal layout with no panning or scrolling: NOT RECOVERABLE FROM PLAN
- Three perch zones: They provide the Z-Axis of Front, Middle, and Back.
- Local-time anchored day/night palette shifts: NOT RECOVERABLE FROM PLAN
- Rare, low-impact weather: Weather is ambient and intentionally "low-impact."
- Continuous mood-shaped idle micro-motions: They make birds feel alive while reflecting mood.
- First frame renders birds mid-action with no wake-up animations: NOT RECOVERABLE FROM PLAN
- Smooth interpolation between snapshot positions: It smooths movement between server snapshots.
- Reduced-motion slow cross-fades between still poses: They replace frame-by-frame animation while keeping a visual experience.
- Removing ambient leaf and feather drift in reduced motion: This reduces motion load.

### Audio pipeline

- WebAudio procedural synthesis: It combines motifs and varies them in real time.
- Unique chorus: The chorus comes from motif combination and real-time variation.
- Listen-in gradual mix rebalance: It makes the focused bird rise while others drop to ambient.
- Graceful silence when WebAudio is unavailable: It provides a fallback.
- Captions enabled by default when WebAudio is unavailable: Captions preserve call information during graceful silence.

### Accessibility surfaces

- Screen-reader Naturalist prose: It gives the screen-reader surface the same product voice as the Field Notebook.
- Lowercase, present-tense narration: NOT RECOVERABLE FROM PLAN
- Slow 30-60s narration cadence: It avoids "queue flooding."
- Immediate return-greeting and offer-reaction updates: These are high-priority moments.
- Runtime call-caption prose such as "a low trill": It maps procedural calls into descriptive prose.
- Fading text near the calling bird: It associates captions with the bird making the call.
- Keyboard navigation through top bar, birds, Listen-in, and Escape: It provides keyboard access to the core focus interaction.

### Performance budgets and observability

- `<2MB` gzipped bundle: NOT RECOVERABLE FROM PLAN
- `<500ms` time to first bird: It supports the plan's immediate "time-to-first-bird" goal.
- `60fps` on 5-year-old hardware: It supports idle motion and smooth rendering on older hardware.
- Zero memory growth over 30 minutes: It keeps long idle sessions stable.
- Page load, TTFB, render-frame timing, and simulation-tick latency metrics: They observe the core performance surfaces the plan budgets.
- Simulation tick p99 below 5 seconds: NOT RECOVERABLE FROM PLAN
- Excluding per-bird interaction data from aggregate telemetry: The rationale is "Privacy."

### Rollout plan

- Infrastructure Alpha: NOT RECOVERABLE FROM PLAN
- Engine Beta: NOT RECOVERABLE FROM PLAN
- Surface Gamma: NOT RECOVERABLE FROM PLAN
- V1 Launch starting with 2 birds per aviary: NOT RECOVERABLE FROM PLAN
- Age-based bird addition up to max 7: NOT RECOVERABLE FROM PLAN
- Instrumenting performance metrics from day one: It ensures the performance budgets are observed from launch.

### Risks and mitigations

- Drift calibration harness: It verifies the one-week and three-week targets so drift does not feel "too fast" like a game or "too slow" like a screensaver.
- Motif variation for audio: It mitigates "Audio Uncanniness" and avoids looped samples.
- Append-only event log with server-side processing: It mitigates "Sync Correctness" and potential event loss.
- First-class narration and reduced-motion design: It mitigates "Accessibility Regressions" where accessibility feels like a "stripped" version.
