## System-level intent

1. **Strictly web-only, browser-native implementation.** This shows up in "Single-user virtual aviary running strictly in modern web browsers," the non-goal "No native applications," and the repeated use of browser primitives: "Canvas / WebAudio / DOM ARIA Live," "HTML5 Canvas," and "Web Audio API." The plan carries a boundary of "zero iOS/Android native codebases, zero app-store packaging."

2. **A quiet aviary, not a game or chore system.** The plan states "No gamification" and "No Tamagotchi mechanics," excluding "achievements, streaks, levels, XP, scores" and saying birds "never die, starve, fall ill, or show distress." This intent also appears in the drift rule that "Neglect never degrades traits" and "absence produces quiet ambient behavior, never punishment."

3. **Presence and return should feel ambient, not coercive.** The core gesture list names idle attention as "presence," while the non-goals reject "push notifications," promotional emails, "friend visited" toasts, and "Welcome back!" banners. The plan says "The bird greeting is the sole return surface."

4. **Canonical state belongs to the server, not the client.** This appears in "Strict canonical server-side simulation tick," "No client-authoritative state," "The server tick runner is the sole writer," and "Clients submit events to the append-only event log." The risk matrix ties this to avoiding "Race conditions or overwrites between laptop and mobile sessions."

5. **State changes should be additive, slow, and calibrated toward expressiveness.** The monotonic drift section says drift is "strictly additive and positive toward expressiveness," with an "Asymmetry guarantee" that deltas are nonnegative. The calibration target is tuned so measurable change appears around "7 days" and discernible behavioral change around "21 days," avoiding both "turning into a Tamagotchi" and "feeling like a static screensaver."

6. **Privacy isolation is architectural, not cosmetic.** The plan isolates email in `accounts`, translates sessions into a "synthetic UUID," uses encrypted email and HMAC blind indexes, and says telemetry must use "aggregate operational numbers only." It forbids "aggregate machine learning or population-level behavior clustering" on interaction records.

7. **The product voice is naturalist prose.** The plan repeatedly uses "naturalist" language: "field notebook," "naturalist observation gen," "running naturalist prose," and "Naturalist call captions." The risk matrix explicitly avoids accessible surfaces becoming "clinical state dumps rather than a living aviary."

8. **Accessibility must preserve the aviary mood.** The plan includes ARIA live narration, keyboard focus, high-contrast focus rings, call captions, WCAG AA contrast, and a reduced-motion mode. Reduced motion is not treated as a lesser version: it is an "intentional, contemplative cross-fade aesthetic."

9. **The first experience should be immediate and calm.** The architecture bootstraps "sub-500ms first-bird visibility," the rendering section requires "No loading spinners, entry splash animations, or fade-from-black sequences," and birds appear in idle poses on "frame 1."

10. **Audio should be procedural, recognizable, and gracefully optional.** The audio system uses procedural synthesis and "avoiding pre-recorded audio loops entirely." It aims for individual recognizability, emergent chorus behavior, and "graceful silence" with captions if Web Audio fails or autoplay blocks initialization.

## Per-feature whys

### 1. Scope & Architectural Principles

- **Single-user virtual aviary in modern web browsers:** The plan's stated rationale is the "strictly web-only" boundary, reinforced by the non-goal of "zero iOS/Android native codebases" and "zero app-store packaging."

- **Aviary population starting with 2 adopted birds, scaling by chronological age to a hard cap of 7 birds from a 6-species pool:** NOT RECOVERABLE FROM PLAN

- **Single horizontal scene without scrolling, panning, or zooming:** The plan ties this to a scene "fitting within the browser viewport" and later to keeping "all three perch zones and all active birds" strictly on-screen on mobile.

- **Three perch zones, subtle parallax, ambient micro-motion, local-time day/night cycles, and rare weather:** The plan uses these as part of the "single horizontal scene" and later feeds them into mood, perch, and ambient behavior so the aviary remains a living environment rather than a static screen.

- **Procedural client-side audio synthesis:** The plan says this avoids "pre-recorded audio loops entirely" and enables "emergent chorus behavior," dynamic listen-in rebalancing, and "naturalistic call captioning."

- **Presence gesture:** The plan makes presence the dominant driver of drift while filtering it through visibility, focus, recent activity, server verification, and clamping "to prevent background tab drift inflation."

- **Return-greeting gesture:** The plan makes the bird greeting "the sole return surface" while rejecting notifications, announcements, and "Welcome back!" banners.

- **Listen-in gesture:** The plan gives it a sonic rationale: selected Bird A gains focus at "+3 dB" while other birds become "soft ambient, never muted."

- **Offer gesture with seed, song-fragment motif, or still pool:** The plan ties offers to drift and mood: offering seeds or pools drifts "curiosity and boldness," and accepted offers nudge mood to `content` or `curious`.

- **Settle gesture with 5-second cancel affordance:** NOT RECOVERABLE FROM PLAN

- **Auto-generated naturalist field notebook:** The plan connects notebook generation to "Naturalist Observation Gen" and the broader naturalist prose system that also powers accessible narration and avoids "clinical state dumps."

- **Single-user accounts with 15-minute magic links:** NOT RECOVERABLE FROM PLAN

- **Synthetic UUID tenant partitioning:** The plan's rationale is privacy isolation: external email stays in `accounts`, while inter-service communication, state tables, audit logs, and telemetry use only `account_id`.

- **Revocable per-device session tokens:** NOT RECOVERABLE FROM PLAN

- **Self-serve JSON export:** NOT RECOVERABLE FROM PLAN

- **30-day soft-deletion grace period and restore:** NOT RECOVERABLE FROM PLAN

- **Canonical server-side simulation tick:** The plan says this advances "drift and moods independently of client connections" and prevents clients from calculating or mutating canonical personality or positions.

- **Optional private, read-only visit invitations:** The rationale is bounded sharing without social network surfaces: visits are private, email-link based, read-only, and avoid profiles, discovery feeds, comments, avatars, cursors, or co-presence.

- **Immediate host revocation and silent visit logging:** The plan ties revocation to host control and silent logging to the no-notification stance, including no "friend visited" toasts.

- **Screen-reader narration, reduced-motion mode, and WCAG AA contrast:** The plan frames these as accessibility surfaces, with reduced motion replacing frame-by-frame animation with "slow cross-fading static poses" and later an "intentional, contemplative cross-fade aesthetic."

### 2. System Architecture & Boundaries

- **Edge/Gateway & Auth Service:** Its rationale is to translate verified sessions into `account_id`, serve the static bundle, and inline the initial snapshot for "sub-500ms first-bird visibility."

- **15-minute magic link issuance and verification:** NOT RECOVERABLE FROM PLAN

- **HttpOnly, Secure, SameSite session cookies:** NOT RECOVERABLE FROM PLAN

- **Event Ingestion API as an append-only write endpoint:** The plan uses append-only events so clients submit interactions while the server tick remains the sole canonical writer.

- **Unauthorized or malformed event rejection, rate limits, and event validation:** The stated rationale is to accept only authenticated, well-formed interaction events before they enter the simulation pipeline.

- **Simulation Tick Worker Daemon:** The plan uses it to process accumulated events, advance drift, evaluate moods and weather, assign perches, and generate field notebook observations on the server.

- **State Snapshot Read API:** The rationale is to return "lightweight, compact JSON snapshots" containing only the data needed by clients: birds, moods, coordinates, diurnal phase, weather, and call schedules.

- **Isolated encrypted email storage:** The plan names this under "Privacy & Isolation Boundary" and the risk matrix as mitigation for "User emails leaking into operational logs, partitioned shards, or telemetry."

- **HMAC blind index for email lookup:** The plan includes this as part of the same PII boundary, enabling query lookup while keeping email encrypted at rest.

- **No aggregate machine learning or population-level behavior clustering on interaction records:** The rationale is strict partitioning by `account_id` and physical isolation from analytics warehouses.

### 3. Data Model & Storage Specifications

- **`accounts` table with encrypted email, blind index, settings, and soft delete fields:** The plan ties encrypted email and blind indexing to privacy, `visit_notifications: false` to the no-notification stance, `reduced_motion` to accessibility, and soft deletion to the 30-day account flow.

- **`sessions` table with device fingerprint, user agent, last seen, and revocation:** NOT RECOVERABLE FROM PLAN

- **`magic_links` table with token hash, expiry, and consumed timestamp:** NOT RECOVERABLE FROM PLAN

- **`aviaries` table with weather, settled state, tick sequence, and timestamps:** The plan uses these fields to support canonical snapshots, ambient weather, settle state, and tick sequencing.

- **`birds` table with species, name, perch zone, mood, and hidden personality vector:** The plan says the vector is "strictly server-authoritative" and uses it for boldness, warmth, vocal frequency, plumage saturation, and curiosity.

- **0.0 to 1.0 trait constraints:** The plan states the hidden personality vector consists of "scalars 0.0 to 1.0"; the constraints enforce that range.

- **Append-only `interaction_events` table:** The rationale is to collect client interaction events for server-side tick consumption while preserving client/server timestamps and processed status.

- **`notebook_entries` table:** The plan uses it for "Naturalist Field Notebook" prose generated sparsely by the simulation engine.

- **`visit_invitations` table:** The rationale is optional private visits with encrypted visitor email, token-based access, expiry, and host revocation.

- **`visit_logs` table:** The plan connects this to "silent visit logging" and the host's `GET /api/v1/visits/log` review surface.

### 4. API Surface & Protocols

- **Magic-link request returns generic `200 OK` regardless of whether email exists:** NOT RECOVERABLE FROM PLAN

- **Magic-link verify creates a session cookie and redirects to `/`:** NOT RECOVERABLE FROM PLAN

- **Session revoke endpoint:** NOT RECOVERABLE FROM PLAN

- **Account export endpoint:** NOT RECOVERABLE FROM PLAN

- **Account delete and restore endpoints:** NOT RECOVERABLE FROM PLAN

- **Aviary snapshot endpoint:** The plan's rationale is returning the canonical aviary snapshot, including `tick_sequence`, `server_time`, weather, settle state, birds, moods, plumage saturation, and target coordinates.

- **Batched aviary events endpoint:** The rationale is to batch append `presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, and related events into the event store.

- **Social visit invite, view, revoke, and log endpoints:** The plan ties these to private read-only visiting, guest-scoped sessions, immediate host revocation, and host review of visits.

### 5. Simulation Engine Design

- **Asynchronous 60-second tick loop per aviary:** The plan uses this cadence to ingest events, calculate verified presence, compute drift, evaluate diurnal/weather transitions, update moods, place birds, generate notebook entries, and write an atomic snapshot.

- **Presence pings only when visible, focused, and recently active:** The plan explicitly says this is "To prevent background tab drift inflation."

- **Server verification and clamping of presence:** The rationale is also anti-inflation: reject invalid timestamps and cap presence to "at most 60 seconds per tick."

- **Monotonic drift function:** The plan's rationale is the "Asymmetry guarantee": drift is positive only, inactivity produces zero delta, and "birds never become wary or dull from absence."

- **Presence weight as dominant drift driver:** The plan makes active presence the main source of expressiveness while still keeping deltas small.

- **Listen-in drift weight:** The plan says listen-in drifts "social warmth and vocal frequency."

- **Offer drift weight:** The plan says seed and pool offers drift "curiosity and boldness."

- **7-day and 21-day calibration targets:** The risk matrix gives the rationale: avoid drift that moves "too quickly" into Tamagotchi territory or "too slowly" into a static screensaver.

- **Mood set `{ wary, content, curious, drowsy, alert }`:** NOT RECOVERABLE FROM PLAN

- **Diurnal curve:** The plan uses local time to bias behavior: morning toward `alert` and `curious`, midday toward `content`, dusk and evening toward `drowsy`, and full night toward sleeping postures.

- **Nightjar remaining active at night:** NOT RECOVERABLE FROM PLAN

- **Ambient weather effects:** The plan makes weather affect behavior: rain dampens `vocal_frequency` and moves birds under cover, while high wind increases vigilance.

- **Species expansion schedule by aviary chronological age:** The plan states it is "based strictly on aviary chronological age," but gives no rationale for the exact day thresholds. NOT RECOVERABLE FROM PLAN

### 6. Sync Model & Conflict Prevention

- **Single canonical writer:** The plan's rationale is conflict prevention: no peer-to-peer state, no predictive local drift, no race conditions, no overwrites, and no "last-write-wins collisions."

- **Clients append events only:** The plan uses append-only events so clients can report interactions without mutating personality vectors, moods, or aviary state.

- **30-second keepalive snapshot polling while active and on visibility return:** NOT RECOVERABLE FROM PLAN

- **Procedural bezier hop or flutter transition between perch snapshots:** The rationale is avoiding teleportation: clients should not "snap the sprite" when server state changes.

- **Matter-of-fact session invalidation message in standard system typography:** NOT RECOVERABLE FROM PLAN

### 7. Frontend Rendering Pipeline

- **HTML5 Canvas with SVG/HTML accessibility DOM overlay:** The plan uses this split so the scene can render visually while interactive and accessible elements remain in DOM order.

- **Bounded aspect ratio with mobile cropping:** The rationale is keeping all three perch zones and "all active birds" on-screen across mobile viewports.

- **Five parallax layers with specific offsets:** NOT RECOVERABLE FROM PLAN

- **Immediate first frame from inlined snapshot data:** The plan's rationale is "Sub-500ms Render" and immediate first-bird visibility, with static sky, perch geometry, and idle bird poses rendered on frame 1.

- **No loading spinners, entry splash animations, or fade-from-black:** The plan states these are never displayed so the first experience remains immediate and calm.

- **Idle micro-motion:** The plan uses breathing, head-tilts, and micro-hops as "ambient micro-motion," with intervals influenced by curiosity and alertness.

- **Dynamic plumage saturation:** The rationale is to express the `plumage_saturation` personality trait visually by scaling color vibrancy from HSV definitions.

- **Reduced-motion mode:** The plan responds to `prefers-reduced-motion` or a toggle by disabling 60fps animation, using soft cross-fades, removing particles, and smoothing lighting transitions; the risk matrix frames this as an "intentional, contemplative cross-fade aesthetic."

### 8. Audio Pipeline

- **Procedural call synthesizer using Web Audio nodes:** The plan says calls are synthesized at runtime, "avoiding pre-recorded audio loops entirely."

- **Species motif grammars:** The rationale is distinct species identity through motifs such as rising whistles, trills, and clicks.

- **Per-bird pitch, duration, and cadence variance:** The plan says recognizability matters: a user should distinguish "Pip's specific motif envelope and timbre from Wren's."

- **Ambient chorus with Poisson timing and counter-calls:** The plan uses this for "emergent chorus behavior," with Bird B's counter-call probability determined by social warmth.

- **Listen-in gain ramp:** The rationale is selective attention: Bird A becomes louder, other birds become soft ambient, and none are fully muted.

- **Graceful silence fallback:** The plan uses this when Web Audio initialization fails or autoplay blocks audio, with "No recorded audio files" substituted and call captions automatically activated.

### 9. Accessibility Surfaces

- **ARIA live naturalist narration every 30-60 seconds:** The rationale is screen-reader status in "running naturalist prose" rather than raw state dumps.

- **Immediate priority updates only on explicit user actions:** The plan limits priority updates to actions such as return greeting or accepting an offer.

- **Call captions adjacent to vocalizing birds:** The plan matches captions to the synthesized motif so users can read "a soft three-note rise" or similar naturalist call descriptions.

- **Caption fade timings:** NOT RECOVERABLE FROM PLAN

- **Keyboard navigation through top bar, aviary, birds, listen-in, and modals:** The rationale is an accessible DOM order and keyboard access to bird focus, listen-in, and escape behavior.

- **High-contrast dual focus rings:** The plan says this ensures visibility across "daytime and nighttime backgrounds."

### 10. Performance Budgets & Observability

- **Initial JS bundle budget and dynamic imports:** The plan keeps the core engine under budget and code-splits "settings, visits, and notebook" to protect first-load performance.

- **Time to First Bird Visible under 500ms on mid-tier mobile over 4G LTE:** The rationale is immediate first-bird visibility on constrained devices.

- **Continuous 60fps target on a 5-year-old mid-range laptop:** The rationale is stable animation performance on older integrated graphics.

- **Flat memory graph over 30 minutes:** The plan ties this to zero leaking Web Audio nodes, reusable particle pools, and DOM recycling.

- **RUM metrics for First Bird Render, tick latency, audio failures, and frame drops:** The plan uses p50, p90, and p99 metrics as operational monitoring, including a p99 tick alarm over 5 seconds.

- **Telemetry boundaries:** The rationale is privacy: collect aggregate operational numbers only and log zero bird names, traits, notebook observations, or presence timestamps to analytics databases.

### 11. Rollout & Ramp Plan

- **Phase 1: Simulation & Audio Foundation:** The plan sequences event log, simulation daemon, monotonic drift math, and procedural calls first as the foundation.

- **Phase 2: Canvas Rendering & Naturalist UX:** The plan follows with scene construction, day/night cycles, micro-motion, ARIA narration, reduced-motion cross-fades, and top bar chrome.

- **Phase 3: Auth, Sync & Field Notebook:** The plan groups magic link auth, UUID partitioning, soft deletion, field notebook generation, and read-only visit invitations together.

- **Phase 4: Calibration, Hardening & Staged Launch:** The plan uses load testing, 30-day drift validation, and staged production launch to harden the system before release.

- **50,000 simulated account load test:** The plan gives this as hardening for accounts executing 1-minute ticks.

- **Production launch with 2 birds per new aviary:** The plan repeats the starting population, but gives no further rationale. NOT RECOVERABLE FROM PLAN

### 12. Risk Matrix & Mitigations

- **Multi-stage presence filter and automated drift tests:** The plan's rationale is drift calibration: prevent movement "too quickly" into Tamagotchi feel or "too slowly" into static screensaver feel.

- **Strict additive deltas:** The risk mitigation says these prevent "last-write-wins collisions" in multi-device scenarios.

- **Harmonic overtones, formant filtering, and randomized micro-delays:** The plan uses these to mitigate "Audio Uncanniness," including harsh synthesis and phase-canceling chorus events.

- **Naturalist prose engine powering ARIA narration and call captions:** The risk mitigation says this keeps accessibility from becoming "clinical state dumps."

- **Immediate translation of email to synthetic UUID, HMAC blind indexing, and firewall between simulation DB and analytics:** The plan uses these to mitigate "Privacy / PII Leak."
