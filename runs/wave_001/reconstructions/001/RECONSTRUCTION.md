## System-level intent

1. A low-key, observational relationship instead of a game loop. This is stated in the Scope as "focused on a low-key, observational relationship" and reinforced by the non-goals: "No Gamification" and "No Tamagotchi Mechanics." The same intent appears again in Risks, where drift that is too fast is described as "Tamagotchi-fication."

2. A small, bounded aviary rather than an expanding social product. The plan keeps returning to a "small set of birds," a "Single horizontal scene with 2-7 birds," "Starter: 2 birds," and "Cap: 7 birds." The Social non-goal also says "No Social Networking," which keeps the product away from profiles, follows, public discovery, or chat.

3. Server-owned canonical state with a client that renders. The Architecture says "the server owns the canonical state and simulation" while "the client acts as a high-performance renderer." The Render Pipeline Boundary repeats this as server-provided "what" and client-determined "how."

4. Expressiveness should emerge through personality, mood, drift, and procedural calls. The Bird Engine combines "Personality vectors, mood system, procedural call synthesis, and monotonic drift." The Simulation Engine updates the `personalityVector` as a "monotonic increase toward expressive," and the Call-Grammar Runtime applies "Variation" and "Mood Modulation."

5. Calm continuity matters more than abrupt state changes. The plan uses a server tick every "~60 seconds," a "low-pass filter," "slow ramp" audio changes, "slow cadence" screen-reader updates, and "Smooth interpolation" between snapshots. Reduced-motion mode uses "slow cross-fades" and slows color transitions.

6. Accessibility is part of the product voice, not a checklist. The Accessibility Surfaces use "naturalist prose," "Call Captions," and keyboard navigation. The Risks section explicitly warns against "'checklist' accessibility" and mitigates it by designing narration as "a first-class naturalist surface."

7. Privacy and bounded sharing are explicit boundaries. The Data Model uses a "Synthetic ID, non-PII" and encrypted email. Social is "Read-only, opt-in visit invitations." Observability adds a "Privacy Boundary" where "Per-bird interaction data is strictly excluded from telemetry pipelines."

## Per-feature whys

### 1. Scope

- **Aviary Core**: The single horizontal scene with 2-7 birds supports the plan's stated focus on "a low-key, observational relationship" with "a small set of birds." The v1 rollout reinforces the same why with "Starter: 2 birds" and "Cap: 7 birds."

- **Bird Engine**: Personality vectors, mood, procedural calls, and monotonic drift are the machinery for bird expressiveness. The Simulation Engine says drift updates the `personalityVector` as a "monotonic increase toward expressive," and the Call-Grammar Runtime varies pitch, timing, and mood modulation.

- **Return-greeting**: NOT RECOVERABLE FROM PLAN

- **Listen-in**: The Listen-in Mix makes one bird the focus while preserving the aviary context: "Focus Bird" volume ramps up, "Ambient Birds" volume ramps down, but ambient birds are "never mute."

- **Offer**: NOT RECOVERABLE FROM PLAN

- **Settle**: NOT RECOVERABLE FROM PLAN

- **Field Notebook**: The notebook stores "naturalist observations" with text and timestamp, matching the product's observational and naturalist surfaces.

- **Magic-link auth**: NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick**: The tick lets the server consume events, calculate drift, transition moods, and write canonical state. This supports the architecture where the server owns "the canonical state and simulation."

- **Multi-device state synchronization**: Multi-device sync is supported by the server as the canonical source, with clients pulling snapshots on visibility change or keepalive. The Beta explicitly tests "multi-device sync."

- **Read-only, opt-in visit invitations**: Visits allow sharing an aviary while staying inside the Social boundary: "Read-only, opt-in" and no profiles, follows, public discovery, or chat.

- **Naturalist screen-reader narration**: Narration exists to present the aviary as naturalist prose. Its slow cadence of "30-60s updates" avoids "queue flooding," and the Risks section says narration should be "a first-class naturalist surface."

- **Reduced-motion mode**: Reduced motion preserves the scene while reducing animation load: frame-by-frame animation is replaced with "slow cross-fades," ambient drift is removed, and color transitions slow down.

- **Call captioning**: Captions make procedural calls readable, including the audio fallback where the plan specifies "Graceful silence + automatic call captioning."

- **Strict bundle size**: The <2MB gzipped budget is tied to "procedural assets and code-splitting."

- **Time-to-first-bird**: The <500ms target is tied to "CDN edge snapshots and lean render path" so the initial bird view appears quickly.

### 2. Architecture

- **Client-server model**: The architecture separates canonical simulation from high-performance rendering: "the server owns the canonical state and simulation" and "the client acts as a high-performance renderer."

- **Frontend SPA with Canvas/WebGL**: The frontend uses a "high-performance rendering loop" to render the single aviary scene.

- **WebAudio API**: WebAudio supports client-side procedural call synthesis, avoiding recorded or looped samples.

- **Backend Simulation Service**: The backend manages account data and the simulation tick because the server owns canonical simulation state.

- **Auth Service**: The plan says the service handles "Magic-link generation and session token management." It does not give a deeper rationale beyond that service responsibility.

- **Database**: Persistent storage is needed for accounts, bird personality vectors, and the "append-only interaction event log" that the server tick consumes.

- **Render Pipeline Boundary**: The boundary lets the server provide the "what" and the client determine the "how," which preserves canonical state while keeping animation frames and audio synthesis client-side.

### 3. Data Model

- **`accountId`**: The account ID is a "Synthetic ID, non-PII," matching the plan's privacy boundary.

- **`email`**: The email is encrypted, matching the account privacy posture.

- **`settings`**: Settings hold "Accessibility preferences" and the "visit notification toggle," so account state can carry accessibility and opt-in visit choices.

- **`speciesId`**: NOT RECOVERABLE FROM PLAN

- **`name`**: NOT RECOVERABLE FROM PLAN

- **`personalityVector`**: The vector gives the simulation traits to drift and express. It feeds call variation, mood transition, and trait effects such as "high boldness resists `WARY`."

- **`currentMood`**: Mood drives transitions, "Mood-shaped micro-motions," and call "Mood Modulation."

- **`perchZone`**: NOT RECOVERABLE FROM PLAN

- **`presenceLog`**: Presence timestamps feed the Drift Calculation, which applies a low-pass filter to "presence-time and interaction signals."

- **`eventLog`**: The append-only event log lets the server consume interactions sequentially, avoiding "last-write-wins."

- **`notebookEntries`**: Notebook entries store "naturalist observations" with text and timestamp for the Field Notebook.

### 4. API Surface

- **`GET /state`**: This endpoint returns the "current canonical snapshot" that clients pull and interpolate.

- **`POST /events`**: This endpoint appends interaction events so the server tick can consume them.

- **`POST /auth/request-link`**: NOT RECOVERABLE FROM PLAN

- **`GET /auth/verify`**: NOT RECOVERABLE FROM PLAN

- **`POST /visits/invite`**: The host provides a visitor email so the server can generate a "one-time link," keeping visits opt-in and bounded.

- **`GET /visits/view/{token}`**: This endpoint gives visitors a "read-only state snapshot," matching the Social constraint.

### 5. Simulation Engine Design

- **Event Consumption**: Reading new events from the `eventLog` lets the tick process interactions in sequence.

- **Drift Calculation**: The low-pass filter keeps drift slow while updating personality through "presence-time and interaction signals" as a "monotonic increase toward expressive." Risks says this also mitigates "Tamagotchi-fication."

- **Mood Transition**: Mood changes reflect local time of day, recent interactions, ambient weather, and personality traits, so birds respond to context and traits.

- **State Update**: Writing the new state to the DB preserves the server as the canonical source.

- **Call-Grammar Runtime**: The call grammar creates species-based calls from motifs, personality variation, and mood modulation.

- **WebAudio oscillators and filters**: Synthesis through oscillators and filters avoids "looped samples."

### 6. Sync Model

- **Canonical Source**: The server as "sole writer of personality state" prevents competing personality writes.

- **Client Role**: The client is "read-only for state" and "write-only for events," keeping rendering and event submission separate from canonical simulation.

- **Conflict Resolution**: Avoiding "last-write-wins" and processing events sequentially keeps interaction handling ordered.

- **Propagation**: Pulling snapshots on visibility change or keepalive keeps clients updated from canonical state.

### 7. Frontend Rendering Pipeline

- **Layering**: NOT RECOVERABLE FROM PLAN

- **Idle micro-motions**: Idle motion is "Mood-shaped," which makes moods visible through preening and scanning.

- **Transitions**: Smooth interpolation between server-provided perch positions mitigates the Sync Correctness risk of state jitter.

- **Loading without spinners**: The first snapshot renders immediately, and delay falls back to a "quiet field," preserving the calm surface instead of a spinner.

### 8. Audio Pipeline

- **Client-side WebAudio synthesis**: Procedural WebAudio keeps the app from relying on recorded samples.

- **Chorus Mixing**: Mixing exists so multiple procedural calls can overlap "without phase-canceling."

- **Listen-in Mix**: The mix creates focus through slow volume ramps while keeping ambient birds present and "never mute."

- **Fallback**: The fallback keeps the experience graceful through silence plus automatic call captions.

### 9. Accessibility Surfaces

- **Screen-Reader Narration**: Naturalist prose describes the scene, and slow 30-60s updates avoid queue flooding.

- **Call Captions**: Procedural text such as "a low trill" appears near the calling bird, tying captioning to the visible aviary.

- **Keyboard Nav**: Keyboard navigation gives a route through the top-bar, birds, and listen-in interaction.

### 10. Performance Budgets & Observability

- **Bundle Size**: Procedural assets and code-splitting are the stated reason the bundle can stay below 2MB gzipped.

- **Time to First Bird**: CDN edge snapshots and a lean render path are the stated reason for the <500ms target.

- **Runtime**: The 60fps and zero memory growth targets support stable operation on "5-year-old hardware" over 30 minutes.

- **Synthetic Monitoring**: Automated browsers check load times and render frames to observe the performance budgets.

- **RUM**: RUM collects anonymized session durations and audio-context errors, observing real runtime and audio issues.

- **Privacy Boundary**: Excluding per-bird interaction data from telemetry protects the privacy boundary while allowing observability.

### 11. Rollout

- **Alpha**: Internal testing focuses on drift calibration because the plan expects it to be "1 week measurable / 3 weeks visible."

- **Beta**: The small cohort tests "multi-device sync and WebAudio compatibility."

- **v1 Starter and Cap**: Starting with 2 birds and capping at 7 keeps launch within the small bounded aviary described in Scope.

- **New bird offers based on aviary age**: NOT RECOVERABLE FROM PLAN

### 12. Risks & Mitigations

- **Drift Calibration mitigation**: A strict low-pass filter and test harness verification prevent drift from becoming "Tamagotchi-fication."

- **Audio Uncanniness mitigation**: Random variation and mood-based timing reduce the risk that procedural synthesis sounds "robotic."

- **Sync Correctness mitigation**: Client-side interpolation between snapshots reduces state jitter.

- **Accessibility Regressions mitigation**: Treating narration as a "first-class naturalist surface" reduces the risk of "'checklist' accessibility."
