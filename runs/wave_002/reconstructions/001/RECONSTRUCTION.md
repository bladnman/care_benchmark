## System-level intent

1. Calm, non-gamified, non-punitive ambient care. This shows up in the explicit non-goals: "no achievements, streaks, levels, scores," "no dying, hunger, negative drift on neglect," and in the drift rule that traits are "monotonic toward expressive" with "plumage never desaturates on neglect." The rollout reinforces this by making the third bird "entirely detached from engagement metrics."

2. Server-authoritative, append-only truth. The plan says "the server is the authoritative source of truth," the client "interpolates snapshots and emits interaction events," and the API accepts "append-only interaction events." The simulation consumes the "append-only log," and "last-write-wins is explicitly avoided" to prevent "device conflict data loss."

3. Slow time, with visible but subtle change. The plan distinguishes "slow personality drift and fast mood changes," uses a server-side tick at "~1/min," and names the calibration target as personality changes that are "visible over weeks but imperceptible over days."

4. Naturalist prose as product voice. The field notebook is "written in naturalist prose," screen-reader narration describes the aviary in "naturalist prose," and call captions use "generative prose." The accessibility risk also centers on prose that stays synced with the "slow aviary pace."

5. Privacy and bounded sociality. The account model includes "encrypted email"; social visits are "opt-in" and "read-only"; social non-goals exclude "public discovery," "leaderboards," and "shared co-presence." Observability is "aggregate telemetry only" with "No per-bird state or interaction logs" sent to analytics.

6. Accessibility as a first-class surface of the aviary. The scope includes "screen-reader narration, reduced-motion cross-fades, call captions, WCAG AA contrast, keyboard navigability." The detailed accessibility surfaces include a slow ARIA-live feed, captions near birds, keyboard traversal, and WCAG AA focus outlines.

7. Client-side aliveness with graceful degradation. The frontend uses interpolation, "idle micro-motions," and client-generated "Leaves, feathers, and gentle parallax" to maintain "aliveness without server roundtrips." Audio degrades to "graceful silence with call captions enabled."

8. Performance protects the core affective experience. The budgets aim for "Time-to-first-bird visible < 500ms" while "avoiding load spinners," "60fps idle motion," and "Zero memory growth over a 30-minute session." Rollout instrumentation watches TTI and WebAudio failure rates "to ensure the core affective experience is landing."

## Per-feature whys

### Scope

- Browser-based, single-horizontal-scene virtual aviary: The frontend scene is a "single responsive viewport" that preserves aspect ratio "to keep all birds in-frame without panning." The rollout keeps V1 "web-only."

- Single-user accounts via magic-link email authentication: NOT RECOVERABLE FROM PLAN

- Starts with 2 birds: NOT RECOVERABLE FROM PLAN

- Cap at 7 birds over the aviary's lifespan: NOT RECOVERABLE FROM PLAN

- Return-greeting interaction: NOT RECOVERABLE FROM PLAN

- Listen-in interaction: Focusing a bird "triggers a slow volume ramp-up for its channel and a simultaneous ramp-down for the rest," making the interaction an audio focus surface.

- Offer interaction with seed, song fragment, pool: The plan grounds offers as interaction events that are appended to the log and consumed by the simulation for moods and personality drift. The specific seed, song fragment, and pool choices are not further explained.

- Settle interaction: NOT RECOVERABLE FROM PLAN

- Server-side simulation ticking: The tick advances state "independent of connected clients" and tracks "slow personality drift and fast mood changes" from the authoritative server state.

- Auto-generated, read-only field notebook: The plan grounds the notebook in "naturalist prose" and stores entries as timestamped prose strings. It is fetched as notebook entries rather than authored by the user.

- Procedural bird calls with chorus mixing: WebAudio uses "species-specific motif libraries" shaped by mood and vocal frequency, and real-time chorus mixing avoids "phase-canceling artifacts of looped tracks."

- Accessibility bundle: The rationale is that non-visual, reduced-motion, captioned, keyboard, and contrast surfaces all carry the aviary experience: screen-reader narration, "calm CSS/canvas cross-fades," call captions, WCAG AA contrast, and keyboard navigability.

- Opt-in, read-only social visits by email invitation: The rationale is bounded social access: invitation generates a "read-only visit token," visitors fetch an "ambient snapshot," and social network mechanics such as public discovery and shared co-presence are excluded.

### Out of Scope and Non-goals

- Native mobile applications: NOT RECOVERABLE FROM PLAN

- No gamification: The plan excludes "achievements, streaks, levels, scores" and makes bird progression "entirely detached from engagement metrics."

- No Tamagotchi mechanics: The plan excludes "dying, hunger, negative drift on neglect" and says traits move "monotonic toward expressive," with plumage never desaturating on neglect.

- No social network mechanics: The plan limits social to opt-in, read-only visits and excludes "public discovery," "leaderboards," and "shared co-presence."

### Architecture

- Client/server boundary: The client is a "stateless rendering and audio engine" while the server is "the authoritative source of truth." This supports canonical snapshots and avoids client-side truth conflicts.

- API Server: It handles auth, accepts "append-only interaction events," and serves state snapshots so clients remain event-appenders and snapshot-readers.

- Simulation Engine: It is a background worker that ticks at roughly one-minute intervals and processes the event log to "update moods and drift personality vectors."

- Database storing accounts, bird profiles, interaction event logs, and notebook entries: NOT RECOVERABLE FROM PLAN

### Data model

- Account synthetic UUID primary key: NOT RECOVERABLE FROM PLAN

- Encrypted email: This supports the plan's "strict privacy boundary" and its privacy-minimal observability posture.

- Active sessions: NOT RECOVERABLE FROM PLAN

- Social invites: These support "opt-in, read-only social visits via email invitation" and read-only visit tokens.

- Bird stable internal ID: NOT RECOVERABLE FROM PLAN

- Bird species: Species matters because procedural synthesis uses "species-specific motif libraries."

- User-assigned bird name: NOT RECOVERABLE FROM PLAN

- Personality Vector: These traits are the target of "slow personality drift"; vocal frequency shapes calls, plumage saturation is named in the no-neglect rule, and curiosity/social warmth/boldness feed the expressive profile.

- Mood enum: Mood carries "fast mood changes," is evaluated every tick, and shapes procedural calls through "dynamic pitch/timing shaped by mood."

- Presence Event strict Boolean: The three-condition check is meant to avoid inflated drift rates; the risk section says incorrect presence accounting could "inflate drift rates across the entire user base."

- Notebook Entry with timestamp and naturalist prose string: This stores the auto-generated field notebook as timestamped "naturalist prose."

### API surface

- `POST /api/auth/magic-link`: NOT RECOVERABLE FROM PLAN

- `GET /api/aviary/snapshot`: This fetches "canonical state" so clients can render positions, moods, and active animations from the server source of truth.

- `POST /api/events`: This appends interaction and presence events to the log so the simulation can consume them and avoid last-write-wins conflict data loss.

- `GET /api/notebook`: This fetches notebook entries for the read-only field notebook.

- `POST /api/social/invite`: This generates a "read-only visit token" for opt-in social visits.

- `GET /api/social/visit/:token`: This fetches an "ambient snapshot for visitors," keeping social access read-only.

### Simulation engine design

- Tick: The roughly one-minute cadence advances state "independent of connected clients."

- Drift Function: A "low-pass additive filter over presence and interactions" makes traits drift slowly, with the risk section naming the goal as visible over weeks and imperceptible over days. The monotonic expressive rule prevents neglect punishment.

- Mood Transitions: They are evaluated per tick from "time of day," recent interactions, ambient weather events, and personality baseline, supporting the plan's fast mood layer.

- Event Processing: The engine consumes the append-only log, and "last-write-wins is explicitly avoided" to prevent device conflict data loss.

### Sync model

- Multi-device sync: Because clients are "snapshot-readers and event-appenders," simultaneous laptop and phone use resolves to "the same canonical state computed by the server."

### Frontend rendering pipeline

- Single responsive viewport scene: The scene preserves aspect ratio "to keep all birds in-frame without panning."

- Smooth interpolation between server snapshots: Interpolation keeps rendering smooth while the server remains authoritative.

- Idle micro-motions: Preening and head tilts "run continuously" to maintain the aviary's ambient aliveness.

- Reduced-Motion Mode: Active animations are replaced with "slow, calm CSS/canvas cross-fades between static poses."

- Ambient leaves, feathers, and gentle parallax: These are generated client-side "to maintain aliveness without server roundtrips."

### Audio pipeline

- Procedural Synthesis: WebAudio uses "species-specific motif libraries" with pitch and timing shaped by mood and vocal frequency.

- Real-time chorus mixing: The rationale is to avoid "phase-canceling artifacts of looped tracks."

- Listen-in: Focusing a bird slowly raises that bird's channel and lowers the rest, turning focus into a calm audio isolation interaction.

- Audio fallback: If WebAudio fails or is unavailable, the app degrades to "graceful silence with call captions enabled."

### Accessibility surfaces

- Screen-Reader Narration: A slow "30-60s" ARIA-live feed describes the aviary in naturalist prose, matching the slow aviary pace without overwhelming the queue.

- Call Captions: Captions provide "generative prose describing procedural calls" and are positioned near the bird, including when WebAudio is unavailable.

- Focus / Keyboard: Full keyboard traversal and WCAG AA focus outlines make the top bar and birds operable without pointer input.

### Performance budgets and observability

- Initial JS bundle under 2MB gzipped: The budget is set "to fit within WebAudio and logic constraints."

- Time-to-first-bird visible under 500ms: The rationale is "avoiding load spinners" and getting to the core visible bird experience quickly.

- 60fps idle motion on a 5-year-old laptop: This protects the continuous idle micro-motion experience on older hardware.

- Zero memory growth over a 30-minute session: This supports long ambient sessions without degradation.

- Aggregate telemetry only: Observability tracks request counts, latency, and bundle sizes without crossing the privacy boundary.

- P99 tick latency alarms at 5 seconds: These protect the server-side tick cadence from falling too far behind.

- No per-bird state or interaction logs in analytics: This enforces the "strict privacy boundary."

### Rollout

- V1 shipped web-only with email sign-in, core simulation, and social visits: NOT RECOVERABLE FROM PLAN

- Bird progression third-bird offer by account age: The third bird triggers "exclusively on account age" and is "entirely detached from engagement metrics."

- RUM for TTI and WebAudio failure rates: Instrumentation starts there "to ensure the core affective experience is landing."
