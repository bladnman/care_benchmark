## System-level intent

1. Small, bounded, personal ambient product. The plan repeatedly narrows the product to a "Web-only application," "Single-user accounts," a "Single horizontal aviary scene," "bounded to one screen," and a "maximum of 7 birds." The same intent appears in the non-goals: no native mobile apps, no social networks, no public discovery, no leaderboards, and no custom scene/species catalog.

2. Anti-gamification and non-punitive care. The plan explicitly rejects "achievements," "streaks," "scores," "badges," "death," "hunger," and "negative drift/punishment for neglect." The same principle appears in the drift function: "Additive and monotonic" and "traits never decrease on neglect." The drift risk names the boundary: too fast feels "like a Tamagotchi"; too slow "feels dead."

3. Server authority over client assertion. The architecture makes the frontend "Thin, presentation-focused" while the backend is the "Authoritative source of truth." The sync model repeats this as "The server is the absolute source of truth," with "no client-side state merging." Clients emit events, not asserted state such as "boldness = 0.8."

4. Event-sourced, append-only integrity. The plan uses an "append-only event log," event ingestion with acknowledgment rather than immediate state change, and a server tick that processes events linearly. This shows up again in conflict prevention, where event sourcing prevents "last-write-wins overwrites from multiple devices."

5. Slow naturalist cadence. The simulation worker runs on a slow cadence, mood is evaluated during ticks, and personality drift becomes "visible after ~3 weeks" and "measurable after ~1 week." The product voice is naturalist rather than game-like: "Field Notebook," "naturalist observations," "naturalist prose," and captions in the "naturalist voice."

6. Strict privacy with telemetry separation. The plan states "Strict privacy: No aggregation of per-bird or per-account interaction history," stores an encrypted email and synthetic UUID, and separates "operational telemetry from per-bird/account interaction history." Rollout instrumentation uses aggregated performance and error rates with "No user behavior analytics or engagement tracking."

7. Calm, continuous presentation. The frontend interpolates between snapshots, keeps idle micro-motion running, avoids loading spinners, and starts "in media res" or with a "quiet sky color." The audio mix uses "soft, gradual volume automation" and says hard cuts are "strictly avoided." UI chrome is minimal and fades on idle.

8. Accessibility as designed experience, not checklist. The plan includes "Naturalist screen-reader narration," "reduced-motion mode," "procedural call captions," keyboard navigation, contrast, and a risk mitigation against "checklist ARIA attributes." Narration and reduced-motion views are part of the "core definition of done."

## Per-feature whys

**Scope**

- **Web-only application:** NOT RECOVERABLE FROM PLAN

- **Single-user accounts with magic-link email sign-in:** NOT RECOVERABLE FROM PLAN

- **Single horizontal aviary scene, bounded to one screen, maximum of 7 birds, starting with 2:** NOT RECOVERABLE FROM PLAN

- **Procedurally generated audio calls with WebAudio synthesis:** The plan ties this to WebAudio motif generation, personality and mood modulation, "simultaneous calls without phase-canceling artifacts," and the strict "no-loops rule." It also rejects "Audio fallback to recorded loops."

- **Server-side tick-based simulation for bird mood and slow personality drift:** The plan's rationale is canonical server state, slow cadence, and drift that is additive, monotonic, and processed from the event log rather than computed by the client.

- **Client-side rendering of idle micro-motion and smooth state interpolation:** The plan uses the client for presentation: snapshot positions are interpolated, idle micro-motion runs continuously, and the runtime target is "60fps idle motion."

- **The Field Notebook:** The plan's stated purpose is "auto-generated, read-only naturalist observations," stored as an append-only log with timestamps.

- **Return-greeting:** NOT RECOVERABLE FROM PLAN

- **Listen-in focus:** The plan gives a specific audio rationale: ramp up the focused bird and ramp others down to ambient, using soft automation and avoiding hard cuts.

- **Offering (seed, song fragment, pool):** NOT RECOVERABLE FROM PLAN

- **Settle:** The plan calls this a "soft session end" and uses it to end the presence window along with tab close.

- **Opt-in, read-only social visit feature via email invitations:** The plan frames this as a narrow social surface: invite by email, revoke invitations, visitor fetches "read-only ambient state," while non-goals reject social networks, profiles, public discovery, avatars, leaderboards, and co-presence.

- **Strict privacy:** The plan articulates no aggregation of per-bird or per-account interaction history, no behavior analytics, no engagement tracking, and strict separation from operational telemetry.

- **Comprehensive accessibility:** The plan ties accessibility to naturalist screen-reader narration, reduced-motion cross-fade rendering, procedural call captions, keyboard support, high-contrast focus indicators, WCAG AA text, and definition-of-done coverage.

- **Responsive scaling preserving aspect ratio:** NOT RECOVERABLE FROM PLAN

**Architecture**

- **Client (Frontend):** The rationale is a "Thin, presentation-focused" client that pulls snapshots, interpolates, renders, synthesizes audio locally, and dispatches events without computing state or running the simulation tick.

- **Server (Backend):** The rationale is to be the "Authoritative source of truth" for authentication, simulation tick, mood, personality drift, field notebook entries, and append-only event log.

- **Auth Service:** NOT RECOVERABLE FROM PLAN

- **Simulation Service:** The plan makes it a tick-driven worker that reads the event log, advances aviary state, and writes new state to the database on a slow cadence.

- **API Gateway/BFF:** The plan's purpose is serving state snapshots to clients and ingesting interaction events.

**Data Model**

- **Account encrypted email:** The plan's rationale is strict privacy; email is stored encrypted while internal references use a synthetic UUID.

- **Account synthetic UUID:** The stated purpose is "primary key for all internal references."

- **Account settings:** NOT RECOVERABLE FROM PLAN

- **Aviary:** The plan uses it as the single-account canonical state holder, including last simulation tick, timezone offset, and weather/day-night modifiers used by mood and environment.

- **Bird:** NOT RECOVERABLE FROM PLAN

- **Personality Vector:** The plan makes this a "Slow timescale" vector updated solely by the server, with drift visible over weeks and monotonic traits that do not decrease on neglect.

- **Mood:** The plan makes mood the "Fast timescale" state, persisted across sessions and evaluated during ticks from interaction, time of day, weather events, and personality bounds.

- **Event Log:** The rationale is append-only storage of interaction events that the simulation tick can read linearly, supporting conflict prevention and idempotent processing.

- **Field Notebook data:** The plan uses this as an append-only timestamped store for generated naturalist observations.

**API Surface**

- **GET /api/aviary/snapshot:** The purpose is client pull of current state, including bird positions, moods, active animations, environmental state, and recent notebook entries; the sync model also uses snapshot pulls when a client opens or regains visibility.

- **POST /api/events:** The rationale is event ingestion rather than state assertion. Events are appended to the log, and the server responds with acknowledgment, not immediate state change.

- **POST /api/social/invite:** The plan's purpose is for a host to generate an invitation for a specified email.

- **DELETE /api/social/invite/:id:** The plan's purpose is host revocation of an invitation.

- **GET /api/social/visit/:token:** The plan's purpose is visitor access to the host aviary's "read-only ambient state."

- **POST /api/auth/magic-link:** NOT RECOVERABLE FROM PLAN

**Simulation Engine Design**

- **Server-Side Tick:** The plan makes this asynchronous, about once per minute per aviary, and grounded in aggregation of the recent event log.

- **Presence Computation:** The rationale is strict confirmation of presence only when visibility, window focus, and recent input activity overlap; settle or tab close ends the window.

- **Drift Function:** The plan uses a low-pass filter over presence and interactions, with additive and monotonic traits, so change is measurable after about one week and visible after about three weeks without punishment for neglect.

- **Mood Transitions:** The plan's rationale is mood that reflects the latest interaction, time of day from user timezone, weather events, and bird personality bounds.

- **Call-Grammar Runtime:** The plan runs grammar rules on the client from server-provided vocal frequency and mood parameters, selecting motifs and timing.

**Sync Model**

- **Canonical State:** The rationale is that the server is the absolute source of truth and there is no client-side state merging.

- **Conflict Prevention:** The plan uses event sourcing because clients emit events rather than asserting state; server ticks process them linearly, preventing last-write-wins overwrites from multiple devices.

- **Propagation:** The plan syncs devices by pulling latest snapshots on open or regained visibility, because devices read the same server-computed state.

**Frontend Rendering Pipeline**

- **Scene Composition:** NOT RECOVERABLE FROM PLAN

- **Animation:** The rationale is smooth presentation from snapshot positions through interpolation, continuous idle micro-motion, and no loading spinners, starting in media res or quiet sky color.

- **Reduced-Motion Mode:** The plan gives it a "distinct designed aesthetic" that disables frame-by-frame animation and ambient drift, replacing them with slow cross-fades between still poses.

- **Top Bar:** NOT RECOVERABLE FROM PLAN

**Audio Pipeline**

- **Procedural Call Synthesis:** The plan modulates timing and pitch by personality and mood using WebAudio motif libraries at runtime.

- **Chorus Mixing:** The rationale is simultaneous calls without "phase-canceling artifacts found in looped audio."

- **Listen-in Mix:** The plan uses soft, gradual volume automation to foreground the focused bird and reduce others to ambient; hard cuts are strictly avoided.

- **WebAudio Fallback:** The rationale is graceful silence with procedural captions automatically enabled, with no recorded loops served.

**Accessibility Surfaces**

- **Screen-Reader Narration:** The plan uses naturalist prose, slow updates every 30-60 seconds to avoid queue flooding, and priority interrupts for user actions.

- **Captions:** The plan mirrors audio calls in procedural text such as "a soft three-note rise," using the naturalist voice.

- **Keyboard Navigation:** The plan provides full keyboard support and high-contrast, designed focus indicators.

- **Contrast:** The plan targets WCAG AA or better for all user-facing UI text.

**Performance Budgets and Observability**

- **Time-to-First-Bird:** The plan sets a sub-500ms budget on mid-tier 4G and says it is achieved through aggressive code-splitting and small snapshot payloads.

- **Bundle Size:** NOT RECOVERABLE FROM PLAN

- **Runtime Budget:** The plan targets 60fps idle motion and no memory growth over 30 minutes, using reused audio buffers and bounded queues.

- **Observability:** The plan uses synthetic checks and aggregate RUM for page load, render frame timing, and audio context errors, with strict separation from per-bird/account interaction history and an alarm on server tick latency.

**Rollout**

- **V1 Launch:** NOT RECOVERABLE FROM PLAN

- **Bird Ramp-up:** The plan offers new bird species chronologically based on aviary age and caps the aviary at 7 birds.

- **Instrumentation Day 1:** The rationale is aggregated performance and error rates only, with no user behavior analytics or engagement tracking.

**Risks**

- **Drift Calibration:** The plan's rationale is the feel boundary: too fast becomes Tamagotchi-like, too slow feels dead; mitigation focuses internal beta testing on the 1-to-3-week timescale.

- **Sync Correctness:** The plan names dropped events or out-of-order processing as risks to the personality vector; mitigation is robust append-only logging and idempotent simulation ticks.

- **Audio Uncanniness:** The plan names robotic or repetitive procedural generation as the risk; mitigation is high variation in motif libraries and strict adherence to the no-loops rule.

- **Accessibility Regressions:** The plan names checklist ARIA as the risk; mitigation is making narration and reduced-motion views part of the core definition of done.
