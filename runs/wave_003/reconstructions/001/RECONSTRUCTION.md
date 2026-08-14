## System-level intent

- **Ambient, observational relationship rather than a game.** The plan names Pocket Aviary as an "ambient, browser-based virtual aviary" built around "low-key, observational relationships." This principle shows up again in the product mandate of "no gamification, no win conditions, no counters, and no custodial burden," and in the non-goals banning "streaks," "XP," "badges," "levels," and "achievement toasts."
- **"Notice, never announce."** The plan states this as the foundational design tenet and repeats it through sparse notebook cadence, quiet social visits, no push or reminder emails, no toasts, top-bar chrome that "auto-fades," and "zero-entry-pop" loading with no spinner, splash, or fade-from-black.
- **"Feels alive, not robotic."** This phrase is the core technical mandate. It appears in the bird engine's personality and mood-shaped micro-motion, the idle motion engine that avoids "rigid loop cycles," procedural return-greetings varied by absence length, and audio mitigations for "harsh, robotic, or repetitive" procedural calls.
- **Genuine attention, not passive open tabs.** The plan defines attention through a strict "presence" condition and discrete interactions. It reappears in the "3-factor presence condition," server-side validation, "discard unverified presence intervals," and the risk mitigation for "presence inflation."
- **Slow-timescale change with short-timescale mood.** The plan separates "personality drift over weeks" from a "Fast-timescale mood state machine." Calibration targets call for "measurable" change after one week and "noticeable" change after three weeks, while moods respond to solar time, weather, offers, and social contagion.
- **Non-punitive care.** The plan says traits have "zero decay or negative drift on neglect," "Traits never decay," and "Absence simply yields" no change. The non-goals say birds never die, starve, fall ill, or show distress, and that neglect creates "ambient quietness, never punitive degradation or mistrust."
- **Server-side authority and deterministic sync.** The simulation tick is called the "canonical heart of the product." The plan reinforces this with "Clients NEVER write state," an "Append-Only Event Stream," "Deterministic Sequential Consumption," and "zero client-side personality authoring."
- **Privacy by isolation and omission.** The plan uses "Synthetic Account UUID," encrypted PII, aggregate-only metrics, and a "Privacy & Telemetry Firewall." It strictly forbids per-bird data, per-account interaction histories, field notebook contents, and behavioral vectors in analytics.
- **Two-register product voice.** The plan draws a boundary between "Naturalist Register" for aviary narration, captions, notebook entries, and offer responses, and "Matter-of-Fact Register" for sign-in, timeouts, failures, export/deletion, and accessibility configuration.
- **Accessibility as part of the aviary surface.** Accessibility features keep the same product logic: naturalist screen-reader prose, procedural call captions, reduced-motion still poses, keyboard navigation, and visible focus indicators. The plan also guards against "Screen-Reader Queue Saturation" with polite cadence.
- **Lightweight immediacy.** Performance intent supports the ambient illusion: "<2MB bundle," "sub-500ms time-to-first-bird," "Time to First Bird Visible <500 ms," edge snapshot inlining, and "zero blocking network calls before initial paint."

## Per-feature whys

### Executive Summary & Scope

- **Ambient browser-based virtual aviary:** The plan frames the product around "low-key, observational relationships" and an environment that "feels alive, not robotic."
- **Two starter birds scaling up to seven over months based on aviary age:** The plan ties growth to "over months" and "aviary calendar age," keeping progression calendar-based rather than tied to counters, XP, or visit activity.
- **Single horizontal viewport rendered in the browser:** The plan wants the aviary to "live" in one horizontal scene that fits "any viewport without panning/scrolling," with web-only delivery.
- **No gamification, win conditions, counters, or custodial burden:** The plan explicitly uses this to preserve the ambient, observational relationship and avoid streaks, levels, badges, adoption counts, or achievement toasts.
- **Passwordless authentication via 15-minute email magic links:** NOT RECOVERABLE FROM PLAN
- **Revocable per-device session tokens:** NOT RECOVERABLE FROM PLAN
- **Account export as JSON:** NOT RECOVERABLE FROM PLAN
- **Soft deletion with 30-day grace period followed by hard purge:** The plan calls this a "30-day recovery window" and says the user can sign in before then to cancel deletion.
- **Three perch zones:** The plan uses front, middle, and back zones as behavioral space: "bold birds, offer interactions" in front, "equilibrium birds" in middle, and "high-wary / distant birds" in back.
- **Day/night cycle matching user's local solar/clock time:** Solar time drives mood and scene light: dawn nudges birds toward `alert` or `content`, dusk moves toward `drowsy`, and night moves diurnal species to `settled`.
- **Subtle ambient weather:** The weather generator creates soft rain and wind as ambient influence; rain reduces vocal frequency and moves birds toward sheltered perch slots.
- **Continuous ambient micro-motion:** Micro-motion supports the mandate to feel alive, with leaves, feathers, preening, scanning, head-tilting, fluffed feathers, and weight shuffle.
- **Six initial species with distinct silhouettes, palettes, and call grammars:** The plan gives each species distinct visual and procedural audio identity through silhouettes, palettes, and motif grammars.
- **Hidden five-dimensional scalar personality vector:** The hidden traits power slow behavior and rendering changes while client snapshots expose only direct visual/audio scalars, keeping numerical trait vectors out of the user-facing surface.
- **Monotonic upward personality drift:** The plan articulates this as a way to avoid negative drift on neglect: absence yields no decay, and neglect never produces punitive degradation or mistrust.
- **Fast-timescale mood state machine:** The plan uses moods for short-term dynamics driven by solar time, weather, recent stimuli, and social contagion, separate from slow personality drift.
- **Personality and mood-shaped idle micro-motion:** The plan connects traits and moods to preening, scanning, head-tilting, and fluffed feathers so behavior is visible without counters.
- **Procedural return-greeting on tab open or return:** The greeting is varied by absence length, boldness, and mood so the return is noticed without becoming an announcement or toast.
- **Listen-in focus mode:** The plan uses dynamic audio rebalance to favor a focused bird while keeping other birds at an ambient floor, "preserving the sense of a shared physical room."
- **Offer interaction with seed, song fragment, and still pool:** Offers act as discrete stimuli that can nudge birds toward `content` or `curious` and contribute to drift accumulators.
- **Per-bird offer cooldowns:** NOT RECOVERABLE FROM PLAN
- **Settle session-end gesture:** The plan frames this as an end-of-session softening: evening light shift, audio quieting, and all-bird `settled` behavior.
- **Five-second undo grace for Settle:** NOT RECOVERABLE FROM PLAN
- **Field Notebook:** The notebook exists as a sparse, read-only, naturalist observation log for noteworthy events, with lowercase prose and anti-gamification filters rejecting numbers, stats, streaks, and exclamation marks.
- **Server-side simulation independent of client connection:** The plan calls the tick the "canonical heart" that progresses time-based dynamics independently of client connection.
- **Append-only interaction event stream:** Events preserve discrete interaction records for deterministic server consumption instead of direct client state writes.
- **Snapshot polling and visibility wakeups:** The plan uses polling and wakeups to keep clients aligned with canonical state and smooth interpolation after focus changes.
- **Zero client-side personality authoring:** The plan says this eliminates corruption, last-write-wins races, and conflicting personality updates across devices.
- **One-time email-based visit invitations:** The plan keeps social "optional & quiet," with email invites instead of public discovery, profiles, followers, chat, or leaderboards.
- **Read-only, non-co-present visits with zero visitor drift contribution:** The rationale is to preserve an ambient snapshot visit without interaction rights, shared cursor, visitor indicators, or influence on host bird drift.
- **Host-revocable visits:** The visit flow is host-controlled; unavailable visits may have expired or been revoked by the host.
- **Naturalist prose screen-reader live narration:** The plan makes screen-reader output part of the same aviary voice, using naturalist prose at a paced 30-60s idle cadence.
- **Procedural call captions near vocalizing birds:** Captions are an audio accessibility surface and automatic fallback when WebAudio is unavailable, positioned adjacent to the calling bird.
- **Reduced-motion mode:** The plan bypasses continuous skeletal animation, disables drifting leaves and feathers, and uses still-pose cross-fades to preserve the scene with less motion.
- **Keyboard navigation and visible focus indicators:** The plan makes all main actions reachable by keyboard and keeps a high-contrast focus ring visible across dawn, noon, dusk, and night scenes.
- **Client-side procedural WebAudio:** Procedural synthesis avoids static audio files, supports species grammars and chorus coordination, and protects the bundle budget.
- **Graceful silence with automated caption fallback:** When WebAudio is unavailable, the plan avoids exceptions and canned fallback audio while activating captions.

### System Architecture & Boundaries

- **Client Browser Application:** The browser renders at 60fps, runs WebAudio, monitors presence, streams events, and interpolates snapshots so the visible aviary remains smooth while canonical state stays server-side.
- **Edge Gateway and API Layer:** The edge injects the latest snapshot into HTML to make the first bird visible in under 500ms and handles auth, snapshots, visits, and account management.
- **Simulation Tick Worker Fleet:** Workers apply drift, moods, weather, and notebook rules on a 60-second cadence, making the server the product's canonical timekeeper.
- **Data Isolation and Privacy Firewall:** Account state is keyed by synthetic UUID, and observability receives only coarse aggregate metrics, keeping per-bird, per-account, and interaction data out of analytics.

### Data Model & Database Schemas

- **Encrypted email plus email hash:** The schema says encrypted PII is stored with an HMAC-SHA256 hash "for lookup without decryption."
- **Sessions with revoked_at:** NOT RECOVERABLE FROM PLAN
- **Aviary version field:** The monotonic version supports canonical snapshots and versioned state returned to clients.
- **Bird perch zone and perch slot:** These fields support the front/middle/back spatial model and responsive scene placement.
- **Bird mood fields:** Mood state stores fast-timescale behavior such as `wary`, `content`, `curious`, `drowsy`, `alert`, and `settled`.
- **Bird trait fields:** Trait fields store the slow-timescale personality vector that shapes boldness, social warmth, vocal frequency, plumage saturation, and curiosity.
- **Bird drift accumulators:** Presence seconds, listen-in seconds, and offer interactions accumulate raw stimuli since the last drift step.
- **Interaction events table:** The append-only log records presence, listen-in, offer, and settle events for server tick processing.
- **Field notebook entries table:** Notebook entries persist sparse naturalist observations with an event type such as `first_greeter`, `long_quiet`, or `fluffed_cool_air`.
- **Visit invitations table:** Invite tokens, visitor email, expiry, and revocation support one-time, read-only visits controlled by the host.
- **Visit logs duration_seconds:** NOT RECOVERABLE FROM PLAN

### API Surface & Protocols

- **Matter-of-Fact API errors and confirmations:** The plan reserves direct, clear, standard sentence casing for system mechanisms like sign-in, failures, export, and deletion.
- **Magic-link initiation endpoint:** NOT RECOVERABLE FROM PLAN
- **Magic-link verification endpoint:** The endpoint consumes a single-use token and redirects to the root on success; failed links use Matter-of-Fact copy about expiration and retry.
- **Account export endpoint:** NOT RECOVERABLE FROM PLAN
- **Account deletion endpoint:** The endpoint schedules deletion for 30 days and explains the user can sign in before then to cancel deletion.
- **Aviary snapshot endpoint:** The snapshot provides current canonical state on boot, visibility changes, and polling while excluding numerical trait vectors from client state.
- **Interaction events batch endpoint:** The batch endpoint queues client interaction records for append-only server-side processing.
- **Notebook endpoint:** The endpoint returns historical naturalist observations with cursor pagination.
- **Visit invitation endpoint:** The endpoint sends an invitation link to a specified visitor email, matching the quiet, email-based visit model.
- **Visit view endpoint:** The endpoint returns a read-only aviary snapshot and uses a Matter-of-Fact unavailable message when an invitation is expired or revoked.

### Simulation Engine Design

- **60-second simulation tick:** The tick is the "canonical heart of the product," processing events and progressing time-based dynamics.
- **Event ingestion and validation:** The tick validates the 3-way presence conjunction and discards unverified presence intervals to keep drift tied to genuine attention.
- **Low-pass personality drift:** The drift formula creates slow-timescale change from presence, listen-in, offers, and settle stimuli.
- **Calibration targets:** One week of regular use should produce a "measurable instrument shift," while three weeks should create a "noticeable behavioral shift visible to the user."
- **Monotonicity enforcement:** The max clamp ensures traits never decay, matching the non-punitive neglect principle.
- **Mood transitions:** Mood changes use solar time, weather, offers, alarm contagion, and soft decay toward trait-attractor equilibrium.
- **Field notebook sparse trigger evaluation:** The generator waits at least 48 hours, checks noteworthy predicates, and emits lowercase naturalist entries only when a stochastic trigger passes.
- **Solar time calculation:** Local dawn, midday, dusk, and night provide base mood and call probability biases.
- **Nocturnal Nightjar behavior:** Nightjar shifts to alert/active at night, giving species-specific behavior inside the day/night model.
- **Weather Markov process:** Low-probability rain creates occasional ambient changes for 15-30 minutes, reducing calls and prompting sheltered perch choices.
- **Bird-to-bird social dynamics:** Nearby birds can transition to `wary` after a neighbor does, making moods socially contagious.

### State Synchronization & Concurrency Model

- **Server-authoritative architecture:** The plan says this eliminates data corruption and last-write-wins races across multiple devices.
- **Clients never write state:** Clients do not send traits, moods, or coordinates, preserving canonical server state.
- **Deterministic sequential consumption:** The server processes a single chronological event log and applies additive deltas.
- **Client interpolation:** Clients interpolate over 1.5 seconds so server snapshots produce smooth visual transitions.
- **Three-factor presence detector:** Presence requires visible document, focused document, and user activity in the last 300 seconds.
- **Dual-device presence unioning:** Overlapping presence intervals are unioned and capped at 60 seconds per wall-clock minute to prevent drift duplication.

### Frontend Rendering Pipeline

- **Layered viewport and scene composition:** Layers separate sky, foliage, perch zones, particles, and UI chrome so birds and ambient elements can be composed spatially.
- **Fixed aspect ratio bounded box:** The plan keeps birds from being cropped or pushed offscreen across ultra-wide and 320px mobile screens.
- **Zero-entry-pop loading architecture:** The first frame draws birds mid-action with no spinner, fade-from-black, or splash sequence, supporting immediate ambience.
- **Soft sky-colored quiet field when data is delayed:** The fallback preserves calm ambient presentation without a loading announcement.
- **Procedural idle motion with sine, cosine, and Perlin noise:** The plan chooses procedural perturbation "rather than rigid loop cycles" to avoid robotic repetition.
- **Preening, scanning, head-tilts, and weight shuffle:** These motions make mood and attention legible through small bird-centric behaviors.
- **Reduced-motion cross-fades:** Still-pose and perch cross-fades preserve state changes while avoiding continuous motion and particles.
- **Top-bar chrome auto-fade:** Fading to near-invisible chrome keeps the aviary surface quiet; instant wake preserves usability.

### Audio Engine & Procedural Synthesis

- **100% procedural WebAudio synthesis:** The plan avoids static samples and looped sound files while supporting species-specific grammar and the <2MB bundle budget.
- **Species call grammars:** Each species has a distinct motif, from warbler sweeps to nightjar churring, giving birds recognizable voices.
- **Anti-phase staggering:** Staggering prevents birds from vocalizing on the same audio buffer frame, avoiding "unnatural phase reinforcement" and comb filtering.
- **Contagion window:** Calls from high `social_warmth` birds can prompt neighboring callback checks, connecting audio behavior to social traits.
- **Listen-In gain matrix:** The focused bird rises while ambient birds duck but are never muted, preserving a shared physical room.
- **WebAudio fallback:** The plan bypasses failed audio setup, activates captions, and avoids canned fallback files to protect bundle size.

### Accessibility Surfaces

- **ARIA live region:** A polite, atomic live region makes narration available without interrupting active screen-reader speech.
- **Screen-reader pacing:** Idle narration every 30-60 seconds prevents frequent micro-motions from spamming assistive technology.
- **Priority event bumps:** Return greetings, offers, and settle actions can surface promptly because they are user-initiated or meaningful events.
- **Naturalist prose synthesis:** Snapshot coordinates and moods become bird-centric naturalist sentences rather than system text.
- **Call caption opt-in and auto-fallback:** Captions support users who enable them and users whose audio context is unavailable.
- **Spatial caption positioning:** Captions appear near the calling bird, preserving the scene relationship between sound and source.
- **Keyboard focus flow:** Tab, arrow keys, Enter, Space, Escape, and `O` provide non-pointer access to settings, notebook, offers, settle, bird focus, and Listen-In.
- **High-contrast double-ring focus indicator:** The focus ring is designed to remain visible across dawn, noon, dusk, and settled night scenes.

### Performance Budgets, Optimization & Observability

- **Initial JS bundle under 2.0 MB:** The plan links this to tree-shaken Vanilla JS/TypeScript, procedural audio, SVG vector sprites, and dynamic imports.
- **Time to first bird visible under 500 ms:** Edge snapshot inlining, CSS sky gradient, and zero blocking calls before paint serve this budget.
- **Runtime 60 fps:** Canvas/WebGL batching, no DOM thrashing, object pooling, and CSS transforms support the frame-rate target.
- **Memory leak ceiling of 0 MB over 30 minutes:** WebAudio node pooling, buffer reuse, cyclic reference clearing, and bounded notebook scrolling support the ceiling.
- **Simulation tick p99 under 5 seconds:** Sharded workers, indexed batch queries, and bulk PostgreSQL updates support tick latency.
- **Allowed operational telemetry:** Counts, latencies, p99 duration, WebAudio success/failure, FPS histograms, and JS error stacks are allowed because they are coarse operational metrics.
- **Forbidden telemetry:** Individual bird names, moods, traits, per-account counts, notebook text, relationship metrics, and aggregate trait drift averages are forbidden by the privacy boundary.

### Phased Rollout & Lifecycle Strategy

- **Core Engine and Single-Aviary Foundations:** The first phase groups the synthesizer, schema/auth, server simulation, presence validator, rendering pipeline, perch zones, and lighting as foundations.
- **Interactions, Notebook and Accessibility:** The second phase adds return-greeting, Listen-In, Offer, Settle, notebook generation, ARIA narration, captions, and reduced-motion after the core engine exists.
- **Multi-Device Sync, Social Visits and Hardening:** The third phase groups sync, read-only visits, performance harnesses, and drift calibration verification as hardening and broader lifecycle work.
- **Progressive bird unlocking schedule:** The plan uses fixed aviary age milestones from 30 to 360 days, keeping additional birds tied to calendar age and capped at seven.

### Risk Management & Engineering Mitigations

- **Drift calibration and presence inflation mitigation:** The plan identifies unattended tabs as a risk that could compress multi-week progression into days, so it enforces 3-factor presence, rejects excessive heartbeats, and saturates long single-day presence.
- **Multi-device sync mitigation:** The plan avoids conflicting personality updates by using zero client-side state authoring and a single unified event log.
- **Procedural audio fatigue and uncanny valley mitigation:** The plan counters harsh, robotic, repetitive audio with micro-Perlin pitch wobble, formant filtering, non-repeating motifs, convolver reverb, and mix ducking.
- **Screen-reader queue saturation mitigation:** The plan throttles idle updates and uses polite ARIA delivery so assistive tech remains usable.
