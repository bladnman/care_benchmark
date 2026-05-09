## System-level intent

- **Observational relationships over game or care loops.** The plan defines Pocket Aviary v1 as a "browser-based, naturalist virtual environment focused on observational relationships with 2-7 birds." This shows up again in the Field Notebook's "auto-generated naturalist observations," "naturalist prose" generation, and the explicit non-goals: "No streaks, levels, counters, or achievements" and "No hunger, death, or distress meters."

- **Simulation continuity and sync correctness come from a canonical server.** The architecture explicitly chooses a "Thin Client / Thick Server" model "to ensure simulation continuity and sync correctness." The same intent appears in the server-side "Tick" service, "Updated canonical state snapshot," "Canonical Source: The Server," and "No 'Last-Write-Wins' on client state."

- **Regular presence should create slow, expressive change.** The simulation uses "Personality Drift" with "low-pass filters" and drift "monotonic toward 'expressive.'" Calibration is intentionally slow: "Measurable (DB): ~1 week of regular presence" and "Visible (UI): ~3 weeks of regular presence." The risks section keeps this intent visible by monitoring whether birds change "too fast/slow."

- **The sensory experience should avoid mechanical or uncanny behavior.** Animation uses "procedural idle micro-motion" keyed to mood. Calls are "motif-based" and procedural. Chorus logic uses "staggered start times" to "avoid mechanical synchrony," while audio risk mitigation targets calls sounding "'beep-y' or repetitive" with "jitter to timing/pitch," expanded motifs, and testing of the "chorus mix."

- **Accessibility is first-class, not an add-on.** Accessibility is included in v1 as "First-class screen-reader narration, reduced-motion mode, call captioning." It is carried into state snapshots through "Narrative Updates" for "screen-reader consumption," into rendering through reduced-motion cross-fades, and into risk mitigation through automated testing of "`aria-live` outputs against state changes."

- **Social exposure should be small, read-only, and opt-in.** Social is limited to "One-time read-only visit invitations (opt-in)." The non-goals reinforce the boundary with "No profiles, discovery feeds, or public directories."

- **v1 should stay web-based, compact, and stable in-session.** The plan names "Web-only" as a non-goal boundary for native apps, keeps the environment to a "Single-screen horizontal scene," and sets budgets for "Initial Load," "First Bird Visible" on 4G, "60fps steady," and "Zero growth over 30min session."

## Per-feature whys

### 1. Scope and v1 Definition

- **Browser-based, naturalist virtual environment:** The plan's stated rationale is focus on "observational relationships with 2-7 birds."

- **Core Engine:** NOT RECOVERABLE FROM PLAN

- **Environment: single-screen horizontal scene with 3 perch zones:** NOT RECOVERABLE FROM PLAN

- **Day/night cycle and ambient weather:** These feed the simulation: "Current Time (local to user)" and "Weather Service" are Tick inputs, and mood transition is influenced by "time of day."

- **Return-greeting:** NOT RECOVERABLE FROM PLAN

- **Listen-in:** The audio mixer gives Listen-in a clear purpose: "focus one bird (+6dB) while lowering others (-12dB)" through a "Linear ramp (2s)."

- **Offer:** The plan treats offers as interaction intent, logging "offers" through `POST /events/log`; recent interactions then influence drift and mood transition.

- **Settle gesture:** NOT RECOVERABLE FROM PLAN

- **Field Notebook:** The Notebook exists to hold "auto-generated naturalist observations"; the Tick process includes "Generate Observations" as "Periodic naturalist prose generation for the Notebook."

- **Magic-link auth:** NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick:** The tick processes interaction logs and updates "canonical bird states," supporting the Thin Client / Thick Server goal of "simulation continuity and sync correctness."

- **Multi-device state sync:** The plan grounds sync in the server as "Canonical Source" and avoids "Last-Write-Wins" by having the client send only "interaction intent."

- **One-time read-only visit invitations:** The plan makes this social feature "opt-in" and "read-only," consistent with the explicit non-goal of no "profiles, discovery feeds, or public directories."

- **Screen-reader narration:** The rationale is direct access to "naturalist prose" through an "`aria-live` region" and "Narrative Updates" in the state snapshot for "screen-reader consumption."

- **Reduced-motion mode:** The rendering path replaces frame-by-frame animation with "Cross-fades (3s duration) between key poses."

- **Call captioning:** Captions provide "On-screen text for vocalizations" generated from the call grammar, such as "a soft three-note rise."

- **Web-only:** NOT RECOVERABLE FROM PLAN

### 2. Architecture

- **Thin Client / Thick Server:** The plan explicitly says this model is chosen "to ensure simulation continuity and sync correctness."

- **React (TypeScript) SPA:** NOT RECOVERABLE FROM PLAN

- **Node.js (TypeScript) API and Simulation Service:** The server owns the API and Simulation Service in the Thick Server model that supports continuity and correctness.

- **Server-side Tick service:** The tick advances about once per minute, processes interaction logs, and updates "canonical bird states."

- **PostgreSQL:** The plan assigns PostgreSQL to "account/bird state."

- **Redis:** Redis is for "high-frequency interaction event logging before tick processing."

- **Passwordless Magic-Link via SMTP/Third-party provider:** NOT RECOVERABLE FROM PLAN

### 3. Data Model

- **Account:** NOT RECOVERABLE FROM PLAN

- **Aviary:** `last_tick_at` supports the server-side tick's canonical simulation timing.

- **Bird:** NOT RECOVERABLE FROM PLAN

- **PersonalityVector:** It is an input to the Tick and the target of "Personality Drift," with traits updated by "low-pass filters" based on weighted inputs.

- **MoodState:** It supports the "Mood Transition" process, which is a "Stochastic state machine" influenced by personality, time of day, and recent interactions.

- **InteractionEvent:** Interaction events are the server's source for "presence pings, offers" and other recent interactions before tick processing.

- **NotebookEntry:** It persists the generated "prose" observations from the Field Notebook flow.

### 4. Simulation Engine Design

- **Calculate Presence:** Presence is calculated as a "Conjunction of `visibilityState`, focus, and activity events," giving the tick a concrete input for regular presence.

- **Personality Drift:** Drift applies "low-pass filters" to weighted inputs and moves "monotonic toward 'expressive,'" with visible change only after regular presence.

- **Mood Transition:** Mood changes are grounded in a "Stochastic state machine" influenced by "personality, time of day, and recent interactions."

- **Generate Observations:** This creates "Periodic naturalist prose generation for the Notebook."

- **Drift Calibration:** The reason is the risk of birds changing "too fast/slow"; mitigations are "Daily DB audits of trait distributions" and tuning Tick constants.

### 5. API Surface

- **`POST /auth/request-link`:** NOT RECOVERABLE FROM PLAN

- **`GET /state/snapshot`:** The snapshot returns full aviary state: "birds, positions, moods, weather," and carries "Narrative Updates" for screen-reader consumption.

- **`POST /events/log`:** This endpoint submits batches of interaction events, including "presence pings" and "offers," for server-side processing.

- **`POST /social/invite`:** This creates the plan's limited visit invitation feature.

- **Snapshots via polling or WebSocket:** These keep "active sessions" updated with server-to-client state.

- **Narrative Updates:** Their rationale is "screen-reader consumption" as part of the state snapshot.

### 6. Sync Model

- **Canonical Source: The Server:** This keeps state processing on the server rather than the client.

- **No Last-Write-Wins on client state:** The plan avoids client state conflict by having the client only send "interaction intent."

- **Linear server processing of interaction intent:** The server processes interaction intent "linearly," supporting sync correctness.

- **Client interpolation:** Interpolation exists "to avoid 'snapping'" between snapshots.

### 7. Frontend Rendering & Audio Pipeline

- **Canvas/WebGL rendering:** NOT RECOVERABLE FROM PLAN

- **Layering:** NOT RECOVERABLE FROM PLAN

- **Procedural idle micro-motion:** Animation is "keyed to current mood," so motion expresses mood state.

- **Reduced Motion rendering path:** Reduced motion uses "Cross-fades (3s duration) between key poses instead of frame-by-frame animation."

- **Procedural calls with no samples:** NOT RECOVERABLE FROM PLAN

- **Ambient mixer:** Ambient audio keeps "All birds at distance-weighted levels."

- **Listen-in mixer:** Listen-in focuses one bird at "+6dB" while lowering others to "-12dB."

- **Chorus Logic:** Staggered start times exist "to avoid mechanical synchrony."

### 8. Accessibility Surfaces

- **Narration:** An "`aria-live` region" updates with "naturalist prose every 30-60s" for screen-reader narration.

- **Captions:** Captions make vocalizations visible as on-screen text generated from call grammar.

- **Keyboard:** Standard "Tab/Arrow/Enter/Esc" mappings support bird focus and interaction.

- **Contrast:** AA compliance is the stated target for "all UI text and icons."

- **Automated `aria-live` testing:** This mitigates "Visual updates breaking narration flow."

### 9. Performance Budgets & Rollout

- **JS Bundle <2MB gzipped:** The plan ties this budget to "Initial Load."

- **First Bird Visible <500ms on 4G:** The budget centers the first visible bird on a 4G connection.

- **60fps steady idle frame rate:** The budget requires steady idle animation performance.

- **Zero memory growth over 30min session:** The budget targets stable long-session memory behavior.

- **Alpha: 2 birds per aviary, 100 invited users:** NOT RECOVERABLE FROM PLAN

- **Beta: introduce Visit Invitations, 1000 users:** NOT RECOVERABLE FROM PLAN

- **v1.0 monitoring:** Full launch monitors "drift calibration and audio 'uncanniness,'" matching the named risks.

### 10. Risks & Mitigations

- **Drift Calibration mitigation:** Daily DB audits and Tick tuning address the risk of birds changing "too fast/slow."

- **Audio Uncanniness mitigation:** Timing/pitch jitter, a larger motif library, and chorus testing address calls sounding "'beep-y' or repetitive."

- **Sync Correctness mitigation:** Client-side buffering, retry logic, and idempotency keys address "interaction loss during flaky connections."

- **Accessibility Regressions mitigation:** Automated `aria-live` tests address the risk of visual updates "breaking narration flow."
