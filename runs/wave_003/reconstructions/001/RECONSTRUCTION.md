# Pocket Aviary - Blind Reconstruction

## System-level intent

- **Thin-client, thick-server continuity.** The plan repeatedly carries a "thin-client, thick-server architecture" where the browser "handles no authoritative simulation state" and the server is "the authoritative state machine." This shows up in Architecture, Simulation Engine Design, and Sync Model, and is tied to "data continuity and cross-device sync."

- **Snapshot-pull and event-append, not CRUD.** The API is explicitly "designed for snapshot-pull and event-append, not CRUD." The Sync Model repeats the same philosophy by saying clients submit "interaction events (not state mutations)" and the server calculates "additive deltas."

- **Slow, expressive change without game pressure.** The plan excludes "Gamification" and "Tamagotchi mechanics," and makes personality drift "monotonic toward expressive" over "~1 week" for measurable change and "~3 weeks" for user-visible change. The Drift Calibration risk says drift that is too fast would make Pocket Aviary "feel like a game," while too slow "feels broken."

- **Exact presence as a consequential signal.** Presence is not casual activity. The scope requires "Exact presence accounting," and Presence Validation requires `visibilityState === 'visible'`, window focus, and "pointer/key activity within a recent calibration window." The drift risk shows why this matters: presence that is "too easily triggered" or a filter that is "too loose" changes the product feel.

- **Naturalist affect and product voice.** The plan carries a naturalist voice through "Auto-generated Field Notebook (naturalist prose)," "Screen-Reader Narration" that generates "slow-cadence, naturalist prose," and narration that "matches notebook voice." Audio is described as "the core affective spine."

- **Accessibility as the same experience, not an afterthought.** Accessibility is named as a "designed, first-class experience, not an afterthought." The plan includes screen-reader narration, call captions, reduced motion, keyboard navigation, focus rings, WCAG AA contrast, and a risk that regressions could break "the affective experience for screen-reader users."

- **Procedural, lightweight sensory surfaces.** The plan favors WebAudio, procedural calls, SVGs, lazy-loading, and "procedural assets" to meet bundle and audio constraints. Procedural synthesis is used because audio should fit "bundle budgets" and avoid "phase-canceling artifacts."

- **Privacy-bounded single-user state.** The Data Model says all data is keyed by "a synthetic UUID," with emails "encrypted and stored only once," and the Performance and Observability section adds that telemetry "explicitly excludes per-bird state or per-account interaction history." Social visiting is "optional, opt-in, read-only."

## Per-feature whys

### Scope

- **Browser-based virtual aviary (one horizontal scene).** NOT RECOVERABLE FROM PLAN

- **Start with 2 birds, capping at 7 (availability based on aviary age).** The plan ties the 2-bird start to the "core 2-bird experience" in Launch Strategy and ties the cap/unlock behavior to "Age-Gated Scaling." Additional birds "unlock slowly based on the chronological age of the aviary, not usage metrics," which aligns with the no-gamification boundary.

- **Single-user accounts.** The plan's architecture says the account-backed server state is what gives Pocket Aviary "data continuity and cross-device sync." The out-of-scope boundary also excludes "Social network surfaces," leaving the account model single-user except for controlled visits.

- **Email/magic-link authentication.** NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick for canonical state, bird personality vectors, and mood.** The tick exists so the server remains canonical: it processes the append-only event log, updates "personality vectors and moods," logs notebook entries, and manages day/night/weather states tied to timezone while remaining "uncoupled from client rendering."

- **Multi-device sync.** Sync is achieved by "eliminating client-side state ownership." Both laptop and phone clients read "the exact same server snapshot," so there is "no client-to-client sync" and no "last-write-wins" resolution.

- **Return-greeting.** NOT RECOVERABLE FROM PLAN

- **Listen-in.** The Audio Pipeline explains the rationale as attentional focus: focusing a bird "smoothly ramps its gain" while attenuating "but not muting" other birds and ambient noise.

- **Offer.** NOT RECOVERABLE FROM PLAN

- **Settle.** NOT RECOVERABLE FROM PLAN

- **Auto-generated Field Notebook.** The server tick logs field notebook entries, and the Data Model stores "timestamp, prose content." Its product role is tied to the naturalist voice because screen-reader narration "matches notebook voice."

- **Exact presence accounting.** Presence is strict because it feeds the drift system. The plan requires visibility, focus, and recent pointer/key activity to "count as presence," and the risk section warns that loose presence would make personality drift too fast and "feel like a game."

- **Optional, opt-in, read-only social visit feature.** The visit feature is bounded to avoid becoming a social network surface: the plan excludes "discovery, profiles, mutual visits, avatars, comments, leaderboards," while the API supports "revocable" links for "a specific email."

- **Full accessibility surfaces.** The rationale is explicit: accessibility is "a designed, first-class experience." The surfaces preserve the aviary's affective experience through naturalist narration, call captions, cross-fade reduced motion, keyboard navigation, WCAG AA contrast, and strict review gates.

- **Client-side procedural audio synthesis.** Audio runs client-side "to fit bundle budgets and avoid phase-canceling artifacts." WebAudio synthesizes calls from a motif library, and no recorded call loops are used.

### Architecture

- **Client (Browser).** The client is lightweight because the server owns authority. It pulls "small state snapshots," interpolates positions, synthesizes audio locally, appends events, and "handles no authoritative simulation state."

- **Server.** The server is the "authoritative state machine" so the simulation can continuously process client events, update personality and mood, log notebook entries, and manage time/weather state independent of browser sessions.

- **Database.** The database persists the account, bird identity, personality, mood, and event log needed for continuity. It uses "synthetic UUIDs" and avoids email as the primary key, supporting the privacy boundary.

### Data Model

- **Account.** The account holds synthetic UUID, encrypted email, verified status, timezone, outstanding visit invites, and visit log. The rationale recoverable from the plan is continuity, timezone-tied simulation, and controlled visit management.

- **Bird: stable internal identifier, species ID, user-assigned name.** NOT RECOVERABLE FROM PLAN

- **Personality Vector (Slow-timescale).** The vector is persisted server-side so traits can drift slowly and monotonically. The plan names the direction as "only increases" and "toward expressive," with user-visible change taking weeks rather than immediate feedback.

- **Mood (Fast-timescale).** Mood is persisted and modulated by the server tick so birds can have current states such as "wary, content, curious, drowsy" based on recent events, time of day, weather, and personality.

- **Presence & Interaction Log.** The log is append-only because the system is event-append rather than state-mutation based. The server tick processes presence windows, listen-in, offers, and settle triggers from this log.

- **Field Notebook entries.** The notebook stores timestamped prose generated by the server tick. Its rationale is tied to the naturalist prose product voice and to making scene narration and notebook voice match.

### API Surface

- **Snapshot-pull and event-append API.** The API is deliberately "not CRUD" so clients read canonical snapshots and submit additive events rather than owning or mutating authoritative state.

- **`GET /aviary/snapshot`.** The endpoint returns the "current canonical state" of the aviary, including bird positions, moods, calls, and day/night state, which lets clients share the same server truth across devices.

- **`POST /aviary/events`.** The endpoint accepts arrays of presence blocks and interactions because the server tick processes events in order and calculates additive deltas.

- **`POST /aviary/invites` and `DELETE /aviary/invites/:id`.** These support the bounded social visit feature: links are created for "a specific email" and can be revoked when active or pending.

- **`GET /notebook`.** NOT RECOVERABLE FROM PLAN

### Simulation Engine Design

- **The Tick.** The tick is a cron-like server process on an "approx. 1-minute cadence" so simulation can proceed independently of rendering and process the `/aviary/events` log canonically.

- **Drift Function.** The low-pass filter over presence-time and interactions is designed to make change slow: "~1 week" for instrument-measurable change and "~3 weeks" for user-visible change. The risk section explains the balance: too fast feels like a game, too slow feels broken.

- **Mood Transitions.** Mood transitions are evaluated in the tick so bird states reflect recent events, local time of day, ambient weather, and the bird's personality vector.

- **Call Grammar Runtime.** The server defines behavioral call "intent" from vocal frequency and mood, while the client handles exact procedural synthesis. This keeps behavior canonical while audio generation stays client-side.

- **Presence Validation.** Strict gating exists so presence only counts when visible, focused, and recently active. This supports "Exact presence accounting" and protects drift calibration.

### Sync Model

- **No client-to-client sync and no last-write-wins resolution.** The why is direct: sync is achieved by removing client state ownership, so devices read the same server snapshot instead of reconciling competing client states.

- **Clients submit interaction events, not state mutations.** Events are processed by the server-side tick "in order," and the server calculates additive deltas to the personality vector, preserving canonical state.

### Frontend Rendering Pipeline

- **Initial Load with a tiny inline snapshot.** The aviary "loads immediately in-motion" with "No spinners." If there is no bird state, an empty aviary uses "a quiet sky field."

- **Scene Composition.** Three plane perches and subtle parallax support placement based on mood/boldness, which are determined by the server.

- **Micro-motion.** Idle animations and ambient particle drift run independently of the server tick so the scene can remain alive between server snapshots.

- **Transitions.** Client interpolation between snapshot coordinates exists "to ensure smooth movement."

- **Reduced-Motion Mode.** Reduced motion is not only disabling animation; it is a "designed aesthetic" using slow cross-fades between static poses, with ambient drift disabled.

### Audio Pipeline

- **Procedural Synthesis.** WebAudio synthesizes calls from a motif library so there are "No recorded audio loops" for calls and the product can meet bundle and audio-artifact constraints.

- **Chorus Mixing.** Multiple birds calling are mixed in real time so the aviary can support simultaneous bird calls.

- **Listen-in audio behavior.** The gain ramp lets the user focus one bird while other birds and ambient noise are attenuated "but not muting," preserving the wider scene.

- **Fallback.** If WebAudio is blocked, the scene continues in "graceful silence" with procedural captions enabled.

### Accessibility Surfaces

- **Screen-Reader Narration.** Narration uses "slow-cadence, naturalist prose" to describe the scene and "matches notebook voice," preserving the same affective product voice.

- **Call Captions.** Captions translate procedural calls into text such as "a soft three-note rise" and place descriptions near the calling bird.

- **Keyboard Navigation.** Full tab indexing, arrow-key bird focus, Enter for listen-in, and high-contrast focus rings make the core interactions keyboard-accessible.

- **Contrast.** All user-copy text targets "strict WCAG AA compliance."

### Performance Budgets and Observability

- **Bundle Size.** The < 2MB gzipped initial JS budget requires "heavy use of procedural assets, SVGs, and lazy-loading for non-critical UI."

- **Time-to-First-Bird.** The < 500ms target on mid-tier 4G supports the immediate aviary experience named in the Frontend Rendering Pipeline.

- **Runtime.** The 60fps idle-motion target and "No memory leaks over a 30-minute session" support a smooth, durable ambient scene on a 5-year-old laptop.

- **Observability.** Synthetic checks and aggregate RUM watch TTFB, frame timings, and simulation latency, with a p99 < 5s alarm for simulation latency.

- **Privacy Boundary.** Telemetry excludes "per-bird state or per-account interaction history," preserving the privacy boundary while still allowing aggregate performance monitoring.

### Rollout

- **Launch Strategy.** The launch focuses on the web app, the "core 2-bird experience," magic-link auth, and the foundational server tick, matching the plan's emphasis on canonical server state before expansion.

- **Age-Gated Scaling.** Bird expansion is based on chronological aviary age "not usage metrics," preserving slow growth without gamified usage pressure.

- **Instrumentation.** Day 1 metrics focus on TTFB, frame drops, WebAudio initialization failures, and server tick latency because those are the highest-risk parts of the immediate, sensory, server-tick experience.

### Risks

- **Drift calibration testing.** The plan calls for heavy staging tests with simulated daily interaction logs because too-loose presence or filtering makes personality drift too fast and "feel like a game," while too-slow drift "feels broken."

- **Sync/event race conditions.** Poor network batching could make the server tick process bursts of interactions inappropriately, so the event pipeline has to preserve appropriate processing behavior under poor network conditions.

- **Audio auditory review.** Procedural synthesis requires intensive auditory review because calls may fail to sound natural or simultaneous birds may create "dissonant chords."

- **Accessibility review gates.** Strict gates are required because adding UI without corresponding naturalist narration can break "the affective experience for screen-reader users."
