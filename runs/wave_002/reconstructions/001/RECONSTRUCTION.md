## System-level intent

- **Web-only bounded product surface**: The plan repeatedly keeps the experience inside a browser client: "Web-Only Client", "Zero native apps", "No Native Applications", and "No iOS/Android native wrappers or platform-specific code." This shows up in Scope & System Boundaries, Explicit Non-Goals, and the Browser Client topology.

- **Anti-gamification and non-punitive persistence**: The plan prohibits "streak counters", "level/XP systems", "badges", "achievements", "scores", and "public rankings." Bird growth is tied to "aviary age" while "avoiding any visit-count or interaction-frequency gamification." Birds "never die, starve, fall ill, or show distress"; drift is "strictly monotonic toward expressive" with "Zero negative drift."

- **Server-authoritative canonical state**: The plan treats the server tick as the "sole writer" of personality vectors and canonical snapshots. Clients are "stateless renderers" and "event submitters"; "No Client-Authoritative State" is an explicit non-goal, and "Last-Write-Wins (LWW) conflict resolution is entirely avoided."

- **Privacy isolation around identity and telemetry**: The plan uses an "immutable UUID v4" `account_id` across internal relations and telemetry, while "Encrypted email storage" is "restricted strictly to the auth service." The Observability section defines a "Strict Privacy Isolation Boundary" excluding per-bird names, personality vectors, mood states, user presence duration, notebook observations, and email addresses from external analytics/telemetry pipelines.

- **Slow-timescale ambient progression**: The simulation tick runs at a slow cadence, notebook observations are sparse, and drift is tuned for "~1 week instrument visibility, ~3 week user visibility." New bird pacing is by Day 30, Day 90, Day 180, Day 270, and Day 365 rather than interaction frequency.

- **Naturalist product voice**: The Field Notebook is a log of "naturalist observations"; screen-reader narration uses "naturalist prose descriptions"; user-initiated actions push updates "in naturalist voice"; and the risk section guards against "mechanical UI state logs" replacing "naturalist screen-reader narration strings."

- **Procedural, recognizable audio without recorded assets**: The plan states "Zero Recorded Audio" and "100% WebAudio procedural synthesis." It also ties the 7-bird cap to "audio recognizability limits" and gives each bird a `vocal_motif_seed` so one bird's call is "uniquely recognizable" from another's.

- **Accessibility as a complete implementation surface**: Accessibility is not a late add-on in the plan. It includes "Screen-Reader Narration", "Reduced-Motion Surface", "Procedural Call Captions", "Keyboard Nav & Contrast", and "Complete accessibility implementation" in Phase 1 deliverables.

- **Performance and observability with privacy boundaries**: The plan names quantitative budgets such as "Initial JS Bundle < 2MB", "Time to first bird visible < 500ms", "60fps idle rendering", "0 memory growth over 30 minutes", and "Simulation Tick p99 Latency < 5.0 seconds." Observability is limited to operational telemetry while private aviary data is excluded.

## Per-feature whys

### Scope & System Boundaries

- **Web-Only Client**: NOT RECOVERABLE FROM PLAN

- **Single-User Accounts & Magic-Link Auth**: NOT RECOVERABLE FROM PLAN

- **Synthetic Account Identifier (`account_id`)**: The plan articulates this as an internal identity and privacy boundary. The immutable UUID is used across "internal database relations, API calls, server-side simulation logs, cache keys, and telemetry," while email storage is separated into the auth service.

- **Encrypted email storage (`AES-GCM-256`)**: The plan's reason is isolation: encrypted email storage is "restricted strictly to the auth service," and the schema adds `email_hash` for "lookup without decryption."

- **Starter aviary initialized with 2 starter birds**: NOT RECOVERABLE FROM PLAN

- **User can rename birds at any time**: NOT RECOVERABLE FROM PLAN

- **Strict cap of 7 birds maximum per aviary**: The plan states the rationale directly: the cap is "enforced by audio recognizability limits."

- **New bird unlock pacing tied strictly to aviary age**: The plan gives the rationale as avoiding "visit-count or interaction-frequency gamification." The rollout schedule repeats the same age pacing through Day 365.

- **Server-Side Simulation Tick**: The plan's rationale is canonical, client-independent state progression. The tick runs "independently of client connections," computes slow personality drift, evaluates fast mood transitions, and writes canonical state snapshots.

- **Presence Accounting**: The plan makes presence the "primary input to monotonic personality drift toward expressive." The strict conjunction of visible, focused, and recent activity defines what counts as presence.

- **Return-Greeting**: NOT RECOVERABLE FROM PLAN

- **Listen-In**: The plan's rationale is focused listening without destroying ambient presence. One bird's call is raised in the mix while others drop "to an ambient floor" and are "never muted."

- **Offers**: The plan gives a rationale for cooldowns: a 3-5 minute per-bird cooldown "prevents mashing and drift saturation." The plan does not articulate a separate rationale for seed, song fragment, or still pool as specific offer types.

- **Settle**: The plan frames this as an "evening lighting shift and audio quiet." It also says "Tab closure and settle are structurally equivalent at the engine level," making settle part of the same quieting model as leaving.

- **Field Notebook**: The plan's rationale is to produce "naturalist observations" through a "system-generated, read-only log" with sparse generation of about one entry every few days.

- **Optional Read-Only Visits**: The plan's rationale is ambient sharing without social or simulation effects. Visitors get a "read-only ambient view" with "no visitor co-presence," "no visitor cursors," "no visitor interaction capability," and "no visitor presence accounting in host drift."

- **Screen-Reader Narration**: The plan articulates this as slow, polite, naturalist prose describing aviary state, with immediate queued updates for user-initiated actions.

- **Reduced-Motion Surface**: The plan's rationale is replacing continuous motion with "slow cross-fades" between static poses, disabling particles, and slowing day/night transitions for reduced-motion users.

- **Procedural Call Captions**: The plan ties captions to procedural audio: captions are generated near calling birds from "procedural audio motif parameters" and "WebAudio motif parameter envelopes."

- **Keyboard Nav & Contrast**: The plan's rationale is full focus navigation and WCAG AA contrast compliance, with a high-contrast dual ring passing over all day/night backgrounds.

- **Performance & Observability**: The plan's rationale is a responsive, stable aviary: first bird visible under 500ms, 60fps idle rendering, no memory growth over 30 minutes, and operational telemetry that excludes private aviary data.

### Explicit Non-Goals

- **No Native Applications**: NOT RECOVERABLE FROM PLAN

- **No Gamification**: The plan's rationale is a strict product policy against streaks, calendars, XP, badges, achievements, scores, rankings, and visit-count or interaction-frequency pacing.

- **No Tamagotchi Mechanics**: The plan's rationale is non-punitive care: birds "never die, starve, fall ill, or show distress," and there is "No decay of personality vectors on neglect."

- **No Social Network Features**: The plan's rationale is to avoid public discovery, feeds, follower graphs, user profiles, comments, chat, and leaderboards while retaining only optional read-only visits.

- **No Client-Authoritative State**: The plan's rationale is to keep clients from calculating drift or directly mutating personality vectors, preserving server-authoritative canonical state.

- **No Recorded Audio Assets**: The plan's rationale is full procedural synthesis: "100% WebAudio procedural synthesis" and "Zero network requests" for fallback MP3 files.

### Architecture & Service Topology

- **Browser Client**: The plan's rationale is a thin renderer model: state snapshots arrive as HTTP JSON and are "interpolated locally for 60fps rendering," while WebAudio and presence tracking stay in the client.

- **API & Event Gateway**: The plan's rationale is central handling of magic-link verification, token issuance, TLS termination, rate limiting, input schema validation, request signatures, and endpoint routing.

- **API Service**: The plan positions this service as the boundary for snapshots, interaction event appends, notebook reads, and visit invite creation/revoke.

- **Simulation Tick Service**: The plan's rationale is an autonomous background loop that consumes interaction events, calculates low-pass drift, updates moods, generates sparse notebook entries, and writes canonical snapshots.

- **PostgreSQL**: The plan's rationale is persistence: PostgreSQL is the "persistent system of record" for accounts, birds, vectors, notebook entries, and visit invitations.

- **Redis / Key-Value**: The plan's rationale is fast client reads and sessions: Redis is a "high-speed snapshot cache (<50ms reads)" and session store.

- **Append-Only Event Store**: The plan's rationale is ordering. It "guarantees event ordering for the simulation tick."

### Data Model & Schema Definitions

- **Accounts table**: The plan's rationale is an internal UUID key with encrypted email and a hashed lookup path, plus settings for visit notifications, reduced motion, and captions.

- **Birds table**: The plan's rationale is "stable bird identity and hidden personality vector," including mood, perch, species, name, and traits.

- **Interaction events table**: The plan's rationale is append-only event ingestion for presence, offers, listen-in, and settle, with `processed_by_tick` marking tick consumption.

- **Field Notebook entries table**: NOT RECOVERABLE FROM PLAN

- **Visit invitations table**: The plan's rationale is revocable, expiring read-only access through visitor email hash, token hash, `expires_at`, and `is_revoked`.

### API Surface & Web Protocols

- **`POST /api/v1/auth/request-link`**: The plan's rationale is passwordless entry through a magic link request with a 3-requests-per-email-per-15-minutes rate limit and a non-specific response message.

- **`GET /api/v1/auth/verify`**: The plan's rationale is single-use token validation and secure session issuance through an HTTP-Only, Secure, `SameSite=Lax` cookie.

- **`POST /api/v1/auth/logout`**: The plan's rationale is explicit invalidation of the active `session_id`.

- **`GET /api/v1/aviary/snapshot`**: The plan's rationale is canonical state retrieval: clients pull server timestamp, tick id, weather, bird moods, perches, plumage saturation, motif seeds, and target poses.

- **`POST /api/v1/aviary/events`**: The plan's rationale is event submission rather than state submission, so the server tick can process interactions and presence.

- **`POST /api/v1/social/invites`**: The plan's rationale is one-time invited read-only visiting, with invite id and expiry returned.

- **`DELETE /api/v1/social/invites/:invite_id`**: The plan's rationale is immediate revocation of an invitation.

- **`GET /api/v1/social/visit`**: The plan's rationale is read-only access for visitors and rejection of event post attempts.

- **`GET /api/v1/notebook`**: The plan's rationale is access to generated Field Notebook entries.

### Simulation Engine & Algorithm Specifications

- **60-second tick execution**: The plan's rationale is a slow server cadence that collects unprocessed events, calculates valid presence, updates slow vectors, updates fast moods, writes sparse notebook observations, serializes snapshots, and marks events processed.

- **Valid presence minutes**: The plan's rationale is a 15s window that only counts when `visible = true`, `focused = true`, and `active = true`.

- **Personality drift formula**: The plan's rationale is "positive presence and targeted interactions" driving monotonic asymptotic convergence. `(1.0 - P_t)` prevents overshoot, and neglect yields "Zero negative drift."

- **Drift scaling constant**: The plan's rationale is calibration for "~1 week instrument visibility, ~3 week user visibility."

- **Mood transition state machine**: The plan's rationale is fast-timescale mood movement based on local time, weather, and recent interaction events.

- **WARY to CONTENT transition**: The plan ties this to "High boldness" and nearby offers accelerating transition.

- **CONTENT to CURIOUS transition**: The plan ties this to an offer, head-tilt, and approach to the front perch.

- **CONTENT to DROWSY transition**: The plan ties this to sunset or a user-initiated `SETTLE`.

- **ALERT mood**: The plan ties this to sudden ambient weather or a neighbor alarm call.

### Multi-Device Sync & Conflict Prevention

- **Server-Authoritative Architecture**: The plan's rationale is one canonical writer: the simulation tick is the "sole writer" while client devices only render and submit events.

- **Concurrent Session Event Ingestion**: The plan's rationale is allowing multiple open devices to submit pings and interactions while preserving order through `BIGSERIAL` event ids.

- **Presence ping deduplication**: The plan's rationale is preventing "artificial presence inflation" when multiple active devices are in the same 15s window.

- **Avoiding Last-Write-Wins conflict resolution**: The plan's rationale is that clients submit events, not state, so LWW is "entirely avoided."

### Frontend Rendering Pipeline & Visual Design System

- **Render loop architecture**: The plan's rationale is constant 60fps rendering and smooth interpolation of position, rotation, and feather movement between snapshot states.

- **Sky & Lighting Gradient**: The plan ties this layer to local timezone time-of-day.

- **Back Perch Layer**: The plan ties this layer to distant foliage and subtle parallax at 0.2x cursor movement.

- **Middle Perch Layer**: The plan uses this as the main perch layer where birds sit, preen, and rest.

- **Front Perch & Offer Layer**: The plan ties this foreground layer to the water pool/seed tray offer zone and 1.0x parallax.

- **Procedural Bird Sprites**: The plan's rationale is trait-visible rendering: color saturation scales dynamically by the `plumage_saturation` trait.

- **Top-Bar Chrome UI**: The plan's rationale is "Minimal HTML overlay" for Settings, Accessibility, Field Notebook, and Offers, auto-fading after pointer inactivity.

- **Reduced-Motion rendering changes**: The plan's rationale is substituting slow cross-fades and static pose keyframes for continuous animation loops, flight arcs, and particles.

### Audio Pipeline & WebAudio Runtime

- **Procedural Call Synthesis Architecture**: The plan's rationale is "Zero Recorded Audio" with WebAudio nodes for FM synthesis, formant shaping, gain envelope, spatial panning, and master compression.

- **Motif Generator**: The plan's rationale is species-level motif grammar plus per-bird seed variation so individual calls are uniquely recognizable.

- **Chorus Anti-Phase Staggering**: The plan states the rationale directly: randomized 200ms to 750ms call starts prevent "unnatural phase cancellation artifacts."

- **Listen-In Mix Dynamics**: The plan's rationale is focus without muting: the selected bird ramps up, others ramp down to -12dB, preserving an ambient presence floor.

- **WebAudio Fallback Strategy**: The plan's rationale is resilience to autoplay restrictions or missing hardware. It enters Quiet Mode, auto-enables captions, and makes no fallback MP3 requests.

### Accessibility Implementation Strategy

- **Screen-Reader Narration Surface**: The plan's rationale is slow, polite, atomic naturalist prose that describes bird placement, calls, time of day, and light.

- **Immediate narration for user-initiated actions**: The plan's rationale is to push return-greeting and accepted-offer updates immediately in naturalist voice.

- **Procedural Call Captioning**: The plan's rationale is real-time caption generation from the same motif envelopes driving the WebAudio call.

- **Keyboard Navigation**: The plan's rationale is full reachability through `Tab`, with keys for Listen-In, Offers, Field Notebook, and Settle.

- **Contrast and focus indicators**: The plan's rationale is WCAG AA contrast over all day/night backgrounds through a high-contrast dual ring.

### Performance Budgets & Observability

- **Initial JS Bundle Size budget**: The plan's rationale is enforced size control through a Webpack/Vite bundle analyzer check in CI.

- **Time to First Bird budget**: The plan's rationale is first visible bird under 500ms, measured by a PerformanceObserver mark on a 4G network profile.

- **Frame Rate budget**: The plan's rationale is 60fps measured over a 30-minute continuous run on Intel HD 520 baseline hardware.

- **Memory Allocation Growth budget**: The plan's rationale is no memory growth over 30 minutes, verified by Puppeteer heap snapshot comparison in CI.

- **Simulation Tick p99 Latency budget**: The plan's rationale is server tick latency under 5 seconds with Datadog/Prometheus timer alarms.

- **Allowed Operational Telemetry**: The plan's rationale is operational visibility into latency, API errors, cache ratios, synthetic load timings, and WebAudio initialization errors.

- **Strict Privacy Isolation Boundary**: The plan's rationale is excluding per-bird names, personality vectors, mood states, user presence duration, notebook observations, and email addresses from external analytics/telemetry pipelines.

### Rollout & Operations Strategy

- **Phase 1 Deliverables**: The plan restates the intended delivered package: single-user web app, two starter birds, server-side simulation, WebAudio synthesis, accessibility implementation, and optional read-only visit invitations.

- **Aviary Age Pacing Schedule**: The plan's rationale is the same anti-gamification age pacing from Scope: bird unlocks occur on account age days rather than visit counts or interaction frequency.

### Engineering Risks & Mitigation Strategies

- **Drift Calibration Instability mitigation**: The plan's rationale is avoiding two failures: drift that feels "too rapid" and reads as a "gamified Tamagotchi," or drift that is "imperceptible" and reads as static.

- **Browser WebAudio Autoplay Restrictions mitigation**: The plan's rationale is that browsers may block `AudioContext` startup, so the system resumes audio on first user interaction and uses Quiet Mode with captions until audio resumes.

- **Multi-Tab Presence Duplication mitigation**: The plan's rationale is preventing users with 5 tabs from inflating presence time "5x."

- **Accessibility Framing Regression mitigation**: The plan's rationale is preventing developers from substituting "mechanical UI state logs" for naturalist narration strings.
