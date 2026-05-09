## System-level intent

- Calm, non-gamified ambience over progression mechanics. This shows up in the Scope non-goals: "no gamification," "no death," "no hunger," "no negative drift on neglect," and no "scores, streaks, achievements, counters." It returns in Rollout where bird additions are based "strictly on aviary account age" to prevent "gamified grinding," and in Risks where drift that is too fast would feel "like a Tamagotchi."

- One canonical, server-owned aviary rather than local device state. This appears in Scope as "a single canonical aviary per account," in Client/Server Split where the "Server" is the "sole owner of canonical state," and in Sync Model where there is "No Client-Side State Ownership" and "no 'last-write-wins' race conditions."

- Presence should change the birds slowly, positively, and expressively. The Simulation Engine says drift is a "slow low-pass filter," that "presence time" is "dominant," and that deltas are "always positive (monotonic toward expressive)." The Risk section frames the calibration target: not so fast it becomes "a Tamagotchi" and not so slow it feels "unresponsive."

- Naturalist prose is a primary product voice. The plan names an "Auto-generated field notebook in a naturalist voice" and "screen-reader naturalist prose narration." The Accessibility section gives example prose such as "a small grey bird is perched on the front rail..." and the Notebook model stores `prose_text`.

- Procedural variation is preferred over static media. The Audio Pipeline says "No static audio loops," calls are "dynamically constructed at runtime," and "procedural variance" prevents "phase-canceling artifacts." The Audio Uncanniness risk repeats the "no loops" rule.

- Accessibility is first-class and tested as core behavior. Scope names "First-class accessibility" and lists narration, reduced-motion, captions, and WCAG AA contrast. The Risk section says accessibility surfaces should be in the "core automated testing suite" so PRs fail if narration or keyboard focus states break.

- Performance is a product constraint, not an afterthought. Scope and Performance Budgets require "<2MB initial JS bundle," "<500ms time-to-first-bird," "60fps idle on older machines," and "no memory leaks." Rollout requires "Instrumentation from Day One" to monitor the 500ms TTFB and 5s tick latency budgets before general availability.

- Privacy-bounded sharing and measurement. The Data Model says account references use a "synthetic UUID" and email is "encrypted and isolated." Visit invitations are "read-only," "opt-in," and "revocable." Observability has a "Privacy Boundary" where telemetry never includes per-account interaction history, personality vectors, or PII.

## Per-feature whys

### Scope

- **Single-user accounts with magic-link email sign-in:** NOT RECOVERABLE FROM PLAN

- **A single canonical aviary per account, syncing implicitly across multiple devices:** The why is implicit sync from one "server-side canonical database"; all clients read the same canonical state, and the simulation tick advances the aviary "regardless of which client is connected, or if no client is connected."

- **Up to seven birds per aviary (starts with two) from a fixed pool of roughly six species:** NOT RECOVERABLE FROM PLAN

- **The procedural Bird Engine (personality vectors, monotonic drift based on presence, fast-timescale mood states):** The plan uses the Bird Engine to let interaction history create slow expressive change: drift computes additive deltas from "presence time (dominant), listen-ins, and offers," and mood transitions use recent events, local time, weather, and personality.

- **Procedural audio synthesis client-side using WebAudio:** The plan rejects "static audio loops" and uses WebAudio so calls can be dynamically constructed with pitch and timing variations from `vocal_frequency` and `current_mood`, while falling back gracefully to silence and captions if WebAudio is unavailable or blocked.

- **Single horizontal scene with day/night cycle matching the user's local timezone:** The day/night and timezone part is tied to the simulation because mood transitions evaluate "local time of day." The plan gives scene composition details for the horizontal scene, but no separate why for the horizontal form.

- **sit and watch (presence):** Presence is the dominant input to the drift function, and the event log includes `presence_ping` so presence time can move personality vectors "monotonic toward expressive."

- **return-greeting:** NOT RECOVERABLE FROM PLAN

- **listen-in:** Listen-ins contribute to drift deltas, and the audio pipeline gives listen-in an attention mechanic: focusing a bird ramps its gain node while other ambient sound drops but does not fully mute.

- **offer:** Offers contribute to additive drift deltas, and the API/event log examples include `offer_seed`, making offers part of the append-only interaction history consumed by the server tick.

- **settle:** NOT RECOVERABLE FROM PLAN

- **Auto-generated field notebook in a naturalist voice:** The plan makes notebook entries server-written `prose_text`, carrying the "naturalist voice" product language into persisted observations rather than a counter or score surface.

- **Read-only visit invitations (opt-in, revocable):** The why is controlled ambient sharing: visitors get a "read-only ambient state snapshot," while invites are "opt-in" and "revocable" and the social network surfaces remain out of scope.

- **First-class accessibility: screen-reader naturalist prose narration, reduced-motion cross-fade mode, call captioning, and WCAG AA contrast:** These surfaces preserve the aviary experience across screen reader, motion, audio, keyboard, and contrast needs; the plan also guards them with automated tests against regressions.

- **Performance budgets: <2MB initial JS bundle, <500ms time-to-first-bird, 60fps idle on older machines, no memory leaks:** The budgets are meant to keep the web aviary lightweight, quick to first bird, stable through long sessions, and smooth on "older machines."

### Out of Scope for V1

- **Native mobile applications (iOS/Android):** The plan's rationale is that V1 launches "web-only" as a Web-only SPA.

- **Any gamification (scores, streaks, achievements, counters):** The rationale is to avoid "gamified grinding" and keep the product away from progression counters.

- **Tamagotchi mechanics (no death, no hunger, no negative drift on neglect):** The rationale is non-punitive drift; personality deltas are "always positive," and the risk section treats "feels like a Tamagotchi" as a failure mode.

- **Social network surfaces (no discovery feed, no leaderboards, no profiles, no co-presence, no chat):** The rationale is consistent with read-only, opt-in, revocable visits rather than discovery, profiles, co-presence, or chat.

### Architecture

- **Frontend Client:** The frontend is web-only and CDN-served for fast "<500ms" TTFB; it uses WebAudio for procedural calls and Canvas/WebGL or optimized DOM depending on rendering weight.

- **Backend Service:** The service handles auth, API requests, and simulation tick while staying "stateless" and "horizontally scalable."

- **Simulation Worker:** The worker exists to process the simulation tick "approximately once per minute" for active aviaries.

- **Database:** The database persists Accounts, Aviaries, Birds, Notebook Entries, and Event Logs.

- **Cache:** Redis is for "rate-limiting, session token management, and fast recent-event log buffering before the simulation tick."

- **Server:** The server owns canonical state so personality vectors, drift, mood transitions, and field notebook entries are authoritative.

- **Client:** The client is "View layer only" so it can interpolate motion, submit append-only events, and synthesize audio from server-provided state without owning canonical drift or mood.

### Data Model

- **Synthetic UUID account references and encrypted isolated email:** The rationale is the privacy boundary around identity and PII.

- **Session:** Sessions store token, account, device info, and expiry to support authenticated multi-device access.

- **Aviary with `current_weather`:** `current_weather` feeds mood transitions, which evaluate ambient weather.

- **Bird with species, user name, personality vector, current mood, and perch zone:** These fields carry the procedural bird state needed for drift, mood, rendering, audio variation, and user naming.

- **Interaction Event Log:** The event log is append-only input to the simulation tick, avoiding absolute client state and supporting robust retries for presence pings.

- **Notebook Entry:** Notebook entries persist the server-written naturalist `prose_text` over time.

- **Visit Invite:** Visit invites carry host, visitor identity hash or encryption, token, status, and expiry so read-only visits can be opt-in, revocable, and time-bounded.

### API Surface

- **REST or GraphQL API over HTTPS:** The plan uses this as the client/server communication surface for auth, state, events, notebook entries, and visits.

- **`POST /auth/magic-link`:** NOT RECOVERABLE FROM PLAN

- **`POST /auth/verify`:** NOT RECOVERABLE FROM PLAN

- **`GET /aviary/state`:** This endpoint lets clients pull the current canonical state snapshot of birds, moods, positions, and time-of-day offsets.

- **`POST /aviary/events`:** This endpoint keeps interactions append-only by submitting batches such as presence pings and offers rather than absolute state values.

- **`GET /aviary/notebook`:** NOT RECOVERABLE FROM PLAN

- **`POST /aviary/invites`:** NOT RECOVERABLE FROM PLAN

- **`GET /visit/:token`:** This endpoint supports the read-only visit model by fetching an ambient state snapshot for a visitor.

### Simulation Engine Design

- **The Tick:** The tick consumes unprocessed interaction events since the last tick and turns them into canonical Aviary/Bird updates.

- **Drift Function:** The low-pass drift function makes changes slow, presence-dominant, and always positive toward expressive personality rather than punitive neglect.

- **Mood Transitions:** Mood transitions use recent events, local time of day, ambient weather, and bird personality to update `current_mood`.

- **Event Log Consumption:** After the tick updates canonical tables, processed events can be archived or discarded because their effects have been written to Aviary/Bird state.

### Sync Model

- **No Client-Side State Ownership:** The rationale is to avoid client-computed drift, authoritative mood changes, absolute state submissions, and "last-write-wins" race conditions.

- **Snapshot Pulling:** Clients pull on load, visibility change, and low-frequency keepalive to stay near current canonical state without owning it.

- **Implicit Sync:** Multiple devices stay in sync because they read the same server-side canonical database, and the simulation tick advances state even when no client is connected.

### Frontend Rendering Pipeline

- **Scene Composition:** NOT RECOVERABLE FROM PLAN

- **Idle Micro-Motion:** Idle motion keeps birds continuously alive on screen, and the animations are "mood-shaped" so visible behavior reflects current mood.

- **Transitions:** Interpolated positions make birds fly or hop to new perches "rather than teleporting."

- **Reduced-Motion Mode:** Reduced motion honors OS preference or user setting by replacing frame-by-frame motion and flight paths with "slow, graceful cross-fades" while retaining color shifts.

### Audio Pipeline

- **Procedural Call Synthesis:** Procedural synthesis avoids "static audio loops" and lets pitch/timing vary by species motif, `vocal_frequency`, and `current_mood`.

- **Chorus Mixing:** Procedural variance lets multiple simultaneous birds mix "naturally" without "phase-canceling artifacts."

- **Listen-In Mix Decay:** Listen-in focusing ramps one bird's gain while reducing, but not fully muting, ambient mix and other birds.

- **Fallback:** If WebAudio is unavailable or blocked, the aviary "degrades gracefully to silence" with call captions enabled by default.

### Accessibility Surfaces

- **Screen-Reader Narration:** A polite live region updates every 30-60 seconds, with priority bumps for explicit interactions, so screen-reader users receive naturalist prose descriptions of the aviary.

- **Captions:** Captions describe procedural audio near calling birds, such as "a soft three-note rise."

- **Keyboard Navigation:** Full tab indexing, arrow-key bird navigation, Enter for listen-in, and high-contrast focus rings make the core interaction surface keyboard-operable.

- **Visuals:** WCAG AA contrast is required for all user-copy text and UI chrome.

### Performance Budgets & Observability

- **Initial JS bundle must be strictly `< 2MB` gzipped:** The rationale is keeping the initial web load lightweight.

- **Time-to-First-Bird (TTFB) `< 500ms` on mid-tier mobile (4G):** The rationale is fast arrival at the first visible bird on a constrained device/network target.

- **60fps rendering during 30+ minute sessions with zero memory leaks:** The rationale is sustained smooth idle sessions, enforced in CI.

- **Aggregate Telemetry:** Aggregate telemetry monitors page load times, TTFB, tick latencies, WebAudio error rates, and anonymized session durations.

- **Privacy Boundary:** Telemetry never includes per-account interaction history, personality vectors, or PII.

- **Alarms:** A server simulation tick p99 latency above 5 seconds triggers a critical alert.

### Rollout

- **Launch: Ship V1 as web-only with two starter birds:** NOT RECOVERABLE FROM PLAN

- **Ramping Birds:** Additional birds are offered by account age to prevent "gamified grinding."

- **Instrumentation from Day One:** Synthetic performance checks and aggregate telemetry must be active before general availability to monitor the 500ms TTFB and 5s tick latency budgets.

### Risks

- **Drift Calibration:** Headless clients simulating weeks of interaction are used to tune curves so drift is neither "like a Tamagotchi" nor "unresponsive."

- **Sync Correctness:** Append-only event logs and robust retry logic for `presence_ping` submissions mitigate lost presence time from dropped connection events.

- **Audio Uncanniness:** Dedicated sound design passes and the "no loops" rule mitigate robotic procedural synthesis.

- **Accessibility Regressions:** Automated tests make PRs fail if narration text generation or keyboard focus states break.
