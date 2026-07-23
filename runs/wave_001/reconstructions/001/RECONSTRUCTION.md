## System-level intent

- **Slow, ambient relationship-building.** The plan defines the core value as "slow, ambient relationship-building" where birds notice presence "over days and weeks." This shows up again in "subtle presence signals," "low-frequency interactions," the 1-week and 3-week calibration thresholds, and the notebook's "maximum 1 entry per 48-72 hours."
- **Server-authored canonical state.** The plan repeatedly makes the server the sole author: "Server-Authored Canonical State," "clients are stateless renderers," "Single Source of Truth," and "clients never submit vector state replacements." Event ingestion is a write-ahead log for the tick rather than a place for client state mutation.
- **Presence means active attention, not background existence.** The "Strict Conjunction Presence Model" requires visible document state, active focus, and recent pointer/keyboard activity. The tick explicitly evaluates "Presence Triple-Condition" before counting valid presence seconds.
- **Positive-only, non-punitive care.** The plan's care philosophy is "Monotonic Positive-Only Personality Drift": traits move toward expressiveness and "never" down. It says absence means "ambient stillness, never mistrust, distress, or decay," and the Tamagotchi non-goal forbids hunger, health, death, starvation, and negative trait decay.
- **Quiet anti-gamification.** Product surfaces have "No Announcement UI," and the plan refuses "streaks, scores, levels, badges, visit counters, green-dot calendars, or XP." The progression schedule says adoption reflects "relationship deepening rather than gamified rewards."
- **Naturalist voice integrity.** The product voice is "lowercase, present-tense naturalist field-notebook prose." It applies to product surfaces, notebook entries, captions, and screen-reader narration, while "system/error/auth surfaces drop to a matter-of-fact tone."
- **Procedural, lightweight sensory design.** Audio is "Procedural WebAudio Synthesis" because pre-recorded loops are prohibited to prevent "phase cancellation, acoustic repetition, and bundle bloat." Visuals also lean on "compact SVG path definitions" and a "procedural palette."
- **Privacy wall around bird and presence data.** The plan isolates per-bird interaction telemetry "strictly to the user's simulation" and forbids exporting personality vectors, mood histories, user presence duration, notebook text, emails, or hashes into telemetry.
- **Accessibility as a parallel naturalist surface.** The plan includes ARIA narration, reduced-motion cross-fade rendering, procedural call captions, full keyboard navigation, and WCAG AA contrast, while preserving the same naturalist prose instead of technical status text.
- **Immediate living scene under strict budgets.** The aviary should appear "mid-motion on frame 1," avoid "loading spinners, skeleton frames, or black screens," and meet strict budgets: initial JS bundle under 2MB, time-to-first-bird under 500ms, 60fps idle motion, and zero memory growth over 30 minutes.

## Per-feature whys

### Executive Summary & Core Architectural Guarantees

- **Pocket Aviary web-only virtual aviary.** The plan frames the product as a "modern, web-only virtual aviary application," and enforces that boundary later through standard browser APIs: HTML5 Canvas/WebGL and WebAudio.
- **Small group of birds, from 2 at launch to 7 by aviary age.** The why is relationship pacing: the plan says users adopt a "small group" and later says growth by calendar age exists so adoption reflects "relationship deepening rather than gamified rewards."
- **Server-Authored Canonical State.** The rationale is to make clients "stateless renderers" and prevent local mutation of personality vectors or state drift.
- **Strict Conjunction Presence Model.** The rationale is to count only active presence: visible document, focused window, and recent pointer/keyboard activity inside the calibrated activity window.
- **Monotonic Positive-Only Personality Drift.** The plan's rationale is explicitly non-punitive: absence causes "ambient stillness" and no mistrust, distress, decay, or downward trait movement.
- **Procedural WebAudio Synthesis.** The plan says pre-recorded audio loops are prohibited to prevent "phase cancellation, acoustic repetition, and bundle bloat."
- **No Announcement UI & Naturalist Voice Integrity.** The rationale is a quiet product surface with no toasts, banners, streaks, levels, or achievement popups, using "lowercase, present-tense naturalist field-notebook prose."
- **Performance & Privacy Boundaries.** The plan makes the budgets and telemetry wall part of the product guarantee: fast first render, stable long sessions, and no per-bird interaction telemetry in analytics warehouses.

### 1. Scope & Boundaries (V1 Inclusion & Non-Goals Enforcement)

- **Bird Engine & Population.** The rationale for starter birds and age-gated growth is relationship pacing: two birds at launch, later caps by "aviary calendar age," and adoption as "relationship deepening rather than gamified rewards."
- **Magic link authentication via email.** The API rationale includes matter-of-fact responses and standard HTTP 200 regardless of account existence "to prevent email enumeration," plus single-use 15-minute tokens and rate limits.
- **Single-user account mapped 1:1 to a single canonical aviary.** The rationale is canonical ownership: one account maps to one authoritative aviary state.
- **Session token issuance with device revocation.** The rationale is account control across devices: sessions are tied to devices and can be revoked.
- **30-day soft deletion windows: NOT RECOVERABLE FROM PLAN**
- **JSON account data export: NOT RECOVERABLE FROM PLAN**
- **Procedurally staggered return-greetings.** The rationale comes from the core product value: birds "notice the user's presence over days and weeks" through subtle presence signals.
- **Listen-in focus mixing.** The plan's rationale is focused listening without erasing the ambient flock: target bird gain rises, other birds duck smoothly, and other birds are "never muted to 0.0."
- **Three offer gestures with per-bird cooldowns.** The rationale is positive, low-frequency interaction: accepted offers contribute to the effective interaction signal, while cooldowns keep interactions sparse.
- **Opt-in settle lighting shifts with 5s undo affordance.** The plan articulates the rationale through control language: settle is "opt-in" and includes an undo affordance rather than an irreversible mode switch.
- **Read-only field notebook.** The rationale is quiet observation instead of milestones or popups: entries are low-frequency, naturalist, and triggered by specific state transitions.
- **Single horizontal canvas scene: NOT RECOVERABLE FROM PLAN**
- **3 perch depth zones.** The rationale is expressive state made visible: mood and personality changes shift perch preference from back to middle/front, with the 3-week visual threshold triggering perch zone preference shifts.
- **Local-time day/night color and lighting shifts.** The rationale is that mood transitions are based partly on local time, including dusk/night, dawn/morning, and drowsy/settled states.
- **Ambient rain/wind weather effects and leaf/feather drift.** The rationale is ambient environment; the plan groups these with the "Visual & Audio Environment" and later disables particles in reduced-motion mode.
- **Top-bar auto-fade: NOT RECOVERABLE FROM PLAN**
- **Read-only visit invitations via single-use email links.** The rationale is constrained sharing: social visit tokens are read-only, single-use, and revocable.
- **Visitor rendering is non-interactive and isolated from host presence/drift mechanics.** The rationale is to protect the host simulation: visitor sessions cannot submit presence events and cannot affect drift.
- **Screen-reader naturalist prose narration stream.** The rationale is accessibility without breaking voice: narration uses ARIA live regions, updates only on meaningful changes, and avoids technical status language.
- **Reduced-motion cross-fade rendering mode.** The rationale is to honor `prefers-reduced-motion` by replacing 60fps path movement with slow opacity blends and disabling drifting particles.
- **Procedural call captions.** The rationale is accessible audio fallback: when WebAudio is silent or unavailable, captions describe calls near the bird.
- **Full keyboard navigation.** The rationale is complete non-pointer access to top bar, canvas interaction mode, bird focus, Listen-In, Offer, and Settle actions.
- **WCAG AA contrast compliance.** The rationale is accessibility compliance for product surfaces.
- **Native Apps refusal.** The rationale is "Web-browser context only," enforced by using HTML5 Canvas/WebGL and WebAudio standard browser APIs.
- **Gamification refusal.** The rationale is anti-engagement mechanics: no streaks, scores, levels, badges, counters, calendars, or XP, and no data model fields for engagement metrics.
- **Tamagotchi mechanics refusal.** The rationale is non-punitive care: no hunger, health meters, distress, bird death, starvation, or decay subroutines.
- **Social Network Features refusal.** The rationale is bounded social scope: no public discovery, leaderboards, feeds, comments, chat, avatars, co-presence, public API endpoints, or multi-user WebSocket channels.

### 2. Architecture & Component Boundary Design

- **Frontend Client (Web SPA).** The rationale is to render, animate, synthesize audio, capture the presence triple-conjunction, and send append-only interaction events while interpolating server snapshots.
- **API Gateway & Auth Service.** The rationale is centralized auth and rate control: magic link issuing, session validation, access tokens, and limits on magic links and event submission.
- **Event Ingestion Store.** The rationale is preventing client-driven state corruption by storing append-only interaction streams as a write-ahead log for the simulation tick.
- **Server Simulation Tick Service.** The rationale is canonical simulation authorship: the worker reads uncomputed events, updates personality vectors, evaluates moods, generates notebook entries, and refreshes snapshots.
- **PostgreSQL persistence.** The rationale is permanent storage for accounts, birds, synthetic UUID mappings, field notebook entries, persistent vectors, and logs.
- **Redis snapshot cache.** The rationale is ultra-fast snapshot delivery, explicitly under 50ms for REST/SSE snapshot reads.

### 3. Data Schemas & Domain Data Models

- **Synthetic UUIDv4 account references.** The rationale is privacy and indirection: internal references use `account_id`, while raw emails are encrypted on the accounts record.
- **Encrypted email plus email_hash.** The rationale is lookup without decryption while keeping raw user emails encrypted.
- **Personality Vectors table.** The rationale is server-controlled, monotonic, bounded trait storage for boldness, social warmth, vocal frequency, plumage saturation, and curiosity.
- **Bird Mood State table.** The rationale is rendering mood and perch state from canonical data: mood plus last perch zone drive behavior and placement.
- **Interaction Event Ledger.** The rationale is append-only event processing with event type, target bird, duration, and timestamp available for the simulation tick.
- **Notebook Entries table.** The rationale is persistence for generated field notebook observations tied to the aviary.
- **Visit Invitations table.** The rationale is scoped sharing with host aviary, invitee hash, unique token, expiration, and revocation.

### 4. API Surface Specifications

- **`POST /api/v1/auth/magic-link`.** The rationale is safe login initiation: the response stays matter-of-fact and constant across account existence to prevent email enumeration.
- **`POST /api/v1/auth/verify`.** The rationale is session establishment: a valid magic token returns an access token, session id, and account id.
- **`DELETE /api/v1/auth/sessions/:session_id`.** The rationale is device/session revocation.
- **`GET /api/v1/aviary/snapshot` with `If-None-Match` / ETag: NOT RECOVERABLE FROM PLAN**
- **`POST /api/v1/aviary/events`.** The rationale is append-only interaction submission; the endpoint accepts events such as presence and listen-in and reports `accepted_count`.
- **`POST /api/v1/social/invites`.** The rationale is read-only social access through expiring invitations.
- **`GET /api/v1/social/visit/:invite_token/snapshot`.** The rationale is read-only visitor rendering: it returns the host canonical snapshot, while presence event submissions return 403 Forbidden for visit tokens.

### 5. Simulation Engine Design

- **Server-side tick service at 60-second intervals.** The rationale is periodic canonical processing for each aviary, active or inactive, instead of local client simulation.
- **Presence Triple-Condition evaluation.** The rationale is to derive valid presence seconds only from events that satisfy the strict presence model.
- **Monotonic Low-Pass Personality Drift Function.** The rationale is slow, bounded expressiveness: traits approach 1.0 according to the effective signal, and when the signal is zero, no subtraction occurs.
- **Effective signal from presence, listen-in, and accepted offers.** The rationale is to combine subtle presence and low-frequency interactions into one drift input, with listen-in and offers weighted differently.
- **Calibration Verification Bounds.** The rationale is to make change detectable over a week, visually meaningful after three weeks, and testable against over-calibration.
- **Mood Transition State Machine.** The rationale is that local time and session events should shape states such as wary, content, curious, drowsy, and alert.
- **Perch Selection Rules.** The rationale is spatial expression of mood: wary birds favor back perches, curious birds favor front perches, and drowsy birds avoid the front perch.
- **Naturalist Field Notebook Generator.** The rationale is sparse observational prose generated from specific state transitions rather than generic milestones.
- **Notebook sparsity constraint.** The rationale is low-frequency calm: at most one entry per 48-72 hours under regular usage.
- **Notebook prose template engine.** The rationale is voice integrity: entries stay lowercase, present-tense, and naturalist.

### 6. Multi-Device Sync & State Propagation Architecture

- **Single Source of Truth.** The rationale is to keep canonical aviary state in the server database and Redis cache, with no local persistence for personality vectors or mood states.
- **Preventing Last-Write-Wins corruption.** The rationale is to avoid lost presence/drift data by accepting append-only events and calculating deltas server-side.
- **SSE multi-tab and multi-device propagation.** The rationale is simultaneous consistency: phone and desktop receive the exact same snapshot and interpolate transitions.

### 7. Frontend Rendering Pipeline & Visual Scene Architecture

- **Canvas container fixed 16:9 with `overflow: hidden`.** The rationale is to enforce the single, unpanned browser scene without horizontal or vertical scrollbars.
- **Layered scene layout.** The rationale is depth and interaction placement: background, back perch, middle perch, front perch, and foreground overlay each hold distinct visual and UI responsibilities.
- **Initial Frame Render "Mid-Action" Guarantee.** The rationale is immediate life: render birds at snapshot positions with offset idle animation phases on frame 1, or render quiet ambient background without spinners, skeletons, or black screens if the snapshot is delayed.
- **Idle Motion subroutines.** The rationale is mood-specific ambient life: content birds preen, wary/alert birds scan, and perched birds show weight shifts.
- **Reduced-motion rendering mode.** The rationale is to reduce motion while preserving state communication through keyframe cross-fades and slower lighting transitions.

### 8. WebAudio Procedural Synthesis & Audio Pipeline

- **Per-Bird WebAudio Voice Engine.** The rationale is procedural bird calls built from oscillators, noise, filters, gains, mixer, chorus, and listen-in gain ramp rather than audio files.
- **Procedural Call Generation Engine.** The rationale is species and state expressiveness: motifs use pitch sweeps, timbre control, and modulation by `plumage_saturation` and `mood`.
- **Chorus Anti-Collision Algorithm.** The rationale is to prevent phase cancellation and "unnatural acoustic stacking" when multiple birds call at once.
- **Listen-In Mix Ramping.** The rationale is focused attention without total silence elsewhere: one bird ramps up, other birds duck, and disengagement returns all voices to ambient baseline.
- **Silent WebAudio Fallback Mode.** The rationale is graceful accessibility when audio cannot initialize or permission is withheld: output disables silently and call captions turn on.

### 9. Accessibility Surfaces Specification

- **Screen-reader ARIA Live region.** The rationale is a hidden, polite, atomic narration stream for aviary state.
- **Cadence Engine.** The rationale is avoiding high-frequency assistive-technology noise: it evaluates every 45 seconds and updates only on meaningful state drift or position shifts.
- **Tone Compliance for narration.** The rationale is to preserve naturalist voice and avoid technical status messages such as "Screen reader update."
- **Keyboard Navigation Map.** The rationale is complete keyboard access across chrome, canvas, bird focus, Listen-In, Offer, and Settle.

### 10. Performance Budgets, Optimization & Observability

- **Quantitative Performance Budget Contract.** The rationale is explicit performance enforcement: bundle size, first render, frame rate, memory growth, and server tick latency each have strict ceilings or alarm thresholds.
- **No External Audio Assets.** The rationale is eliminating `.mp3` and `.wav` downloads.
- **Vector Graphics & Procedural Palette.** The rationale is compact rendering: SVG path bodies and procedural plumage shaders instead of heavy assets.
- **Code Splitting.** The rationale is keeping non-critical surfaces out of the initial path by dynamically importing auth settings, notebook viewer, and social invite bundles on click.
- **Telemetry & Privacy Wall.** The rationale is to allow operational metrics while forbidding export or logging of per-bird vectors, mood histories, individual presence duration, notebook contents, emails, and hashes.

### 11. Phased Rollout & Progression Plan

- **Aviary Age Growth Progression Schedule.** The rationale is explicit: "adoption reflects relationship deepening rather than gamified rewards."
- **Day 0 onboarding with 2 starter birds.** The rationale is initial small-group adoption: the system auto-selects two distinct species and the user assigns names.
- **Month 2 through Month 14 bird offers.** The rationale is quiet, calendar-age progression up to a hard maximum cap of seven birds.
- **Engineering Release Sequence.** The rationale is ordered execution from "Core Engine & Data Model Validation" through prototype, auth/sync/privacy, accessibility/naturalist polish, and canary performance verification.

### 12. Risk Matrix & Technical Mitigations

- **Drift Over-Calibration mitigation.** The rationale is preventing birds from changing "noticeably between consecutive days"; calibration tests assert bounded change after seven days.
- **Audio Phase Cancellation mitigation.** The rationale is preventing multiple procedural calls from firing at the same sample frame by injecting mandatory jitter.
- **Multi-Device Race Conditions mitigation.** The rationale is preventing client state overrides and lost drift data by making the server tick the sole author of vector deltas.
- **Accessibility Degradation mitigation.** The rationale is avoiding screen-reader flooding through ARIA live-region rate limiting.
- **Memory Growth mitigation.** The rationale is meeting the 30-minute memory guarantee by recycling WebAudio nodes and canvas offscreen buffers.
