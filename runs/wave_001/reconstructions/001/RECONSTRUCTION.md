## System-level intent

- **Felt-aliveness over traditional gaming mechanics**: The Executive Summary says Pocket Aviary prioritizes "felt-aliveness" over "traditional gaming mechanics, quests, streaks, or Tamagotchi-style custodial obligations." This shows up again in the strict exclusion of "achievements, streaks, levels, scores, badges," and in the drift risk warning that overshooting speed "turns birds into Tamagotchis."

- **Presence-based relationship building without punishment**: The vision names "presence-based relationship building," while the simulation rules make presence measurable through "strict triple-conjunction rules." The same intent appears in "Monotonic Constraint" and the non-goal that there is no "negative personality drift on neglect."

- **Naturalist field-notebook voice**: The plan repeatedly asks for "naturalist voice," "sparse naturalist prose entries," "naturalist prose screen-reader narration," and a "Naturalist field-notebook register" for product surfaces. The prose style is "lowercase, present-tense, descriptive."

- **Server-authoritative state, append-only history**: The plan emphasizes "single server-authoritative simulation state" and "append-only event logging." It repeats that the server "exclusively owns" canonical simulation state and that clients "never submit updated trait or vector states directly."

- **Quiet, opt-in social behavior**: The social surface is explicitly "Opt-In & Quiet," with "read-only visit invitation," "revocable tokens," and "silent visit logging without push/toast notifications." This aligns with the exclusion of feeds, directories, leaderboards, avatars, cursors, and comments.

- **Graceful access across motion, audio, keyboard, and screen-reader modes**: Accessibility intent appears in "naturalist prose screen-reader narration," "custom cross-fade reduced-motion mode," "procedural call captions," "WCAG AA contrast compliance," and "full keyboard navigation." The audio fallback also turns on captions automatically when WebAudio is unavailable or muted.

- **Smooth, immediate, procedural immersion**: The plan specifies "60fps web canvas," "ambient micro-motion," "Time-to-First-Bird Visible" under 500 ms, "no loading spinners," and procedural WebAudio instead of "recorded audio files or stock audio loops." The audio risk section says repetitive loops "break immersion."

## Per-feature whys

### Scope (v1 Boundaries & Non-Goals)

- **Aviary Core**: The plan frames the product as a "small group of animated birds" that "live in a single horizontal scene," so one canonical horizontal scene with 2 starter birds and a hard cap of 7 serves that small-group aviary premise.

- **Authentication & Accounts**: The plan connects account credentials to server ownership: the server "exclusively owns account credentials," and security hardening keeps email addresses encrypted as restricted PII.

- **Synthetic UUID account mapping**: The rationale is PII isolation. The plan says "Synthetic UUIDs" are used in internal service routing, logs, and telemetry while email addresses stay encrypted.

- **Account export & 30-day soft deletion**: NOT RECOVERABLE FROM PLAN

- **Bird Simulation & Engine**: The simulation advances slow-timescale personality vectors and fast-timescale mood states to support "felt-aliveness" through evolving traits, daily/session moods, perch behavior, calls, plumage, and curiosity.

- **Strict multi-signal presence accounting**: The why is presence-based relationship building with valid presence seconds. The tick loop only counts presence events matching the "strict triple-conjunction rules."

- **Return greetings**: The plan ties greetings to presence and social warmth: greetings are part of "Presence & Interactions," and high `social_warmth` increases greeting and call-response probability.

- **Progressive listen-in mix rebalancing**: The rationale is focused attention within the aviary soundscape. Selecting a bird raises the focused bird and soft-ducks others, making listen-in a presence interaction rather than a score or task.

- **Offers (seed, song fragment, still pool)**: The plan grounds offers in interaction and mood: offer events are logged, recent offers can affect mood transitions, and `curiosity` controls "Offer investigation likelihood."

- **Per-bird cooldowns for offers**: NOT RECOVERABLE FROM PLAN

- **Settle ritual**: The plan describes settle as an "evening lighting/quieting ritual," and the mood engine says dusk moves birds toward `drowsy`/`settled`.

- **5-second undo for settle**: NOT RECOVERABLE FROM PLAN

- **Field Notebook**: The rationale is sparse naturalist observation instead of stats or game progress. Notebook records are generated from sparse triggers and stored as "Naturalist observation string" prose.

- **Sync Architecture**: The rationale is avoiding multi-device conflict and preserving personality state. The plan says sync is driven by "single server-authoritative simulation state and append-only event logging" with "no LWW for personality state."

- **Rendering & Audio Pipelines**: The rationale is felt-aliveness through a smooth scene and procedural sound: "60fps web canvas visual scene," "ambient micro-motion," "local day/night light transitions," weather, and WebAudio call synthesis.

- **Social (Opt-In & Quiet)**: The rationale is a quiet visit model, not a social network. The plan specifies read-only invitations, revocable visitor tokens, and "silent visit logging without push/toast notifications."

- **Accessibility & UX Voice**: The rationale is access while preserving product voice: narration uses naturalist prose, reduced motion uses cross-fades, captions support procedural calls, and the dual-voice model separates product and system surfaces.

- **Native Mobile Apps exclusion**: NOT RECOVERABLE FROM PLAN

- **Gamification & Engagement Mechanics exclusion**: The plan explicitly prefers "felt-aliveness," "naturalist voice," and "presence-based relationship building" over quests, streaks, and game mechanics.

- **Tamagotchi Custodial Mechanics exclusion**: The rationale is avoiding custodial obligation and punishment. The plan excludes bird death, hunger, sickness, happiness decay, and negative drift, and says neglect does not reduce values.

- **Social Network Surfaces exclusion**: The rationale is the same "Opt-In & Quiet" social direction: no public feeds, discovery directories, leaderboards, co-presence, shared cursors, or comments.

- **Recorded Audio / Stock Asset Fallbacks exclusion**: The plan favors procedural WebAudio for bundle size and immersion, and the audio risk section says repetitive loops can "break immersion."

### Architecture & System Topology

- **Browser Client**: The browser client houses the features that make the scene live locally: Canvas/WebGL rendering, WebAudio synthesis, and presence/event monitoring.

- **API Gateway**: Edge snapshot caching supports the under-500 ms "Time-to-First-Bird Visible" target, while magic-link auth verification belongs to the account security boundary.

- **Aviary State & Query Service**: Its rationale is server-owned account and state access: auth/account management and snapshot generation sit beside server-exclusive ownership of bird identity records and canonical state.

- **Event Log Ingest Service**: The plan states its why directly: append-only event ingestion "preserves per-account stream."

- **Transactional Data Store**: The rationale is durable canonical storage for accounts, bird state and vectors, interaction event log, and Field Notebook records.

- **Server Simulation Tick Engine**: The tick engine is where canonical slow simulation happens: drift filtering, mood transitions, and notebook synthesis are server-side.

- **Server Authorization & Ownership**: The rationale is state integrity. The server exclusively owns credentials, bird identities, vector parameters, canonical ticks, and Field Notebook generation.

- **Client State Consumption**: The rationale is smooth client experience from authoritative state. Clients pull snapshots and receive deltas, then interpolate spatial perches and motifs between snapshots.

- **Event Submissions**: The rationale is preserving simulation truth. Client actions are immutable events, and clients never submit trait or vector states directly.

### Data Model & Schema Specification

- **Account Record**: The account schema supports PII restriction, soft deletion state, settings, notification toggles, and reduced-motion preference storage.

- **Bird Record**: The plan gives explicit whys for stable identity and species: `bird_id` remains stable across renames/syncs, and `species_id` determines visual silhouette and call motif grammar.

- **Personality Vector Record**: Each scalar has a behavioral why: boldness biases perch proximity, social warmth affects call-response and greetings, vocal frequency affects calls and choruses, plumage saturation changes visual richness, and curiosity affects offers and head-tilts.

- **Mood & Runtime State**: The record supports current mood, current perch zone, last interaction, and transitions that respond to time-of-day, offers, rain, alarm states, and boldness.

- **Interaction Event Log**: The rationale is immutable interaction history for server processing. It stores presence, listen-in, offer, settle, and undo events with payload and timestamp.

- **Field Notebook Entry**: The rationale is durable naturalist observation prose: entries store `body_prose` as a "Naturalist observation string."

### Simulation Engine Design

- **Server-Side Simulation Tick Loop**: The scheduled tick is the mechanism for server-authoritative, slow-timescale aviary change about once per minute.

- **Event Ingestion**: The rationale is processing the append-only interaction stream since the last processed sequence.

- **Presence Aggregation**: The rationale is calculating "valid presence seconds" from strict presence events before drift is applied.

- **Personality Drift Filter**: The rationale is slow, testable, user-visible change. The plan calibrates alpha so drift is detected after 7 days in tests and visible after 21 days to users.

- **Monotonic Constraint**: The rationale is that "neglect does not reduce values" and drift is "monotonic toward expressive."

- **Mood State Machine & Transitions**: The rationale is to let local context shape momentary life: rain reduces vocal frequency, dusk moves toward drowsy/settled, and high boldness reduces `wary` probability.

- **Notebook Generation Evaluation**: The rationale is sparse observation: only trigger conditions such as "first greeting order shift of the week" write naturalist prose entries.

- **Procedural Call Grammar Engine**: The rationale is species-specific living sound without stock audio. Species grammars define frequency sweeps, harmonics, trills, and pauses.

- **Personality Modulation**: The rationale is making vectors audible: high `vocal_frequency` shrinks inter-call intervals, and high `social_warmth` increases call-matching probability.

- **Real-Time Synthesis and Spatialization**: The rationale is procedural calls that fit the scene: oscillator chains, gain envelopes, filters, pan, and gain respond to perch zone and listen-in focus.

### Sync & State Propagation Model

- **Initial Hydration**: The rationale is loading an authoritative aviary snapshot containing scene time, weather, birds, target perches, moods, plumage vectors, and audio seed parameters.

- **Client Interpolation**: The plan states the rationale directly: move birds along bezier paths over multiple frames, "avoiding positional snapping."

- **Event Logging & Concurrency Handling**: The rationale is multi-device correctness. Laptop and phone sessions both append immutable events to the same log.

- **No Last-Write-Wins (LWW) for Vectors**: The rationale is preventing personality vector overwrite and data loss; server ticks process timestamped events instead.

### Frontend Layout, Rendering, & Audio Pipelines

- **Single Horizontal Scene**: The rationale is the core premise that birds live in "a single horizontal scene"; the layout enforces no horizontal panning or vertical scrolling.

- **Fixed 16:9 canvas with responsive scaling**: NOT RECOVERABLE FROM PLAN

- **Perch Zones**: The rationale is layered depth and behavior. Zones are `back`, `middle`, and `front`, and perch zone also drives audio pan/gain and boldness-related proximity.

- **Visual Atmosphere**: The rationale is ambient felt-aliveness through sky mapping tied to local time and soft particle loops such as drift leaves and falling feathers.

- **Initial Render Handshake**: The rationale is immediate presence: the first frame renders birds mid-pose in current simulation state.

- **Loading fallback as quiet ambient field**: The rationale is calm product tone; the plan explicitly says "no loading spinners."

- **UI Controls**: NOT RECOVERABLE FROM PLAN

- **Top Bar Auto-Fade**: NOT RECOVERABLE FROM PLAN

- **Ambient Chorus**: The rationale is a procedural WebAudio soundscape with dynamic gain allocation rather than recorded loops.

- **Listen-in Mix Ramp**: The rationale is auditory focus: the selected bird rises to +3dB while other birds soft-duck by -12dB over 1.5 seconds.

- **Audio Fallback**: The rationale is graceful silence and accessibility. If WebAudio is unavailable or muted, visuals continue and procedural captions are enabled.

### Accessibility & UX Voice Architecture

- **Screen-Reader Narration Subsystem**: The rationale is naturalist access to the scene through an `aria-live="polite"` region with slow-cadence prose every 30-60 seconds.

- **Event Priority**: The rationale is responsive narration for user-triggered actions: greetings, offers, and settle produce immediate naturalist updates.

- **Prose Style**: The rationale is the naturalist field-notebook register: "lowercase, present-tense, descriptive."

- **Reduced-Motion Mode**: The rationale is accessibility for `prefers-reduced-motion` or manual settings while retaining the aviary's light transitions through lengthened cross-fades.

- **Dual-Voice Enforcer Pattern**: The rationale is voice separation: product surfaces use naturalist prose, while auth, sync errors, settings, and accessibility config use matter-of-fact standard English.

### Performance Budgets, Security, & Observability

- **Initial JS Bundle Size**: The rationale is performance, achieved through "WebAudio code generation instead of audio samples," compact SVG bird rigs, and dynamic code splitting.

- **Time-to-First-Bird Visible**: The rationale is fast first presence: under 500 ms from edge CDN cached HTML/snapshot on 4G.

- **Frame Rate Target**: The rationale is stable idle rendering at 60fps on 5-year-old mid-tier laptop hardware.

- **Memory Growth Budget**: The rationale is no unaccounted leak growth across 30 minutes of continuous runtime.

- **PII Isolation**: The rationale is privacy and security hardening: emails are encrypted in `accounts`, while synthetic UUIDs are used for routing, logs, and telemetry.

- **Telemetry Boundary**: The rationale is observability without personal interaction detail: collect API latency, render FPS, and tick duration without per-bird state or per-user interaction parameters.

### Rollout, Instrumentation, & Risk Mitigation

- **Delivery Phasing & Ramping**: NOT RECOVERABLE FROM PLAN

- **Drift Calibration safeguard**: The rationale is avoiding Tamagotchi-like pressure: overshooting drift speed "turns birds into Tamagotchis," so synthetic 7-day and 21-day streams test the presence accumulator.

- **Procedural motif variation generator**: The rationale is preventing "Audio Fatigue / Uncanniness"; non-repeating pitch offsets, harmonic jitter, and dynamic silence pauses protect immersion.

- **Sync conflict safeguard**: The rationale is preventing "Multiple open tabs overwriting personality vectors" through immutable event-log architecture, server-authoritative ticks, and zero client vector writes.
