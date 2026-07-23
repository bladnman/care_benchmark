## System-level intent

1. Calm, quiet presence over time is the central product posture. This appears immediately in the Executive Summary: "a calm, browser-based virtual aviary" responding to a user's "quiet presence over days and weeks." It is reinforced by the non-goals: "No Gamification," "No Tamagotchi Mechanics," and "Zero outbound re-engagement messaging."

2. Absence is non-punitive. The plan repeats this in the non-goals and drift physics: "Absence produces quietness, never negative drift or punishment," and the monotonicity constraint says traits "never decrease due to neglect or absence."

3. Growth should be slow, sparse, and age-based rather than score-based. The population expands through "aviary age thresholds," and the drift calibration is set so "visible changes require ~21 days of visits." Notebook entries are "sparse," and the population ramp unlocks birds at Day 30, Day 90, and Day 180+.

4. The server is the source of truth; the client is a view and synthesis layer. The architecture section says the server retains "sole authority over simulation state, personality vectors, and session history," while the client is a "render-and-synthesis view layer." Sync repeats this as the "Single Writer Principle."

5. State changes should be append-only, derived, and conflict-free. The Event Ingestion Service receives "append-only event batches"; sync says clients send gesture notifications "never absolute trait overrides." The stated intent is to eliminate "last-write-wins race conditions and state divergence across devices."

6. Privacy is a design constraint, not just a storage detail. The plan uses "synthetic UUIDv4 identifiers," stores emails encrypted, says emails are "never referenced in logs or foreign keys," and limits observability to "privacy-preserving" aggregate telemetry while prohibiting "per-account behavioral profiles."

7. Social presence must remain optional, quiet, and powerless. The social affordance is "Optional & Quiet," "off by default," "instantly revocable," and gives visitors "zero co-presence or interaction power." The broader scope boundary rejects "public feeds," "user profiles," "comments," "leaderboards," and "public discovery directories."

8. Accessibility is part of the core surface. The plan includes a "Naturalist screen-reader narration stream," procedural call captions, full keyboard navigation, WCAG AA contrast, and a "dedicated reduced-motion cross-fade rendering engine." The audio fallback also enters `SILENT_CAPTIONED` mode.

9. Voice is intentionally split between naturalist prose and matter-of-fact system language. The plan uses "naturalist, present-tense, lower-case text" for narration, notebooks, and captions, but requires "matter-of-fact register" for auth, errors, and settings, enforced by a string linter.

## Per-feature whys

### 1. Executive Summary & Scope Boundary

- Aviary Population: The plan ties population growth to "aviary age thresholds" and later specifies a slow ramp from 2 starter birds to a 7 bird maximum cap. The why carried by the plan is slow aviary growth over time, aligned with a calm experience "over days and weeks."

- Authentication & Accounts: The plan's rationale is account continuity for a browser-based aviary while avoiding email as an internal identity key. It specifies "Synthetic UUID internal keying with encrypted email storage" and later says emails are "never referenced in logs or foreign keys."

- Multi-Device Sync: The rationale is to avoid "peer-to-peer or last-write-wins state conflicts." The server-side authoritative tick and immutable snapshots ensure each signed-in device reads the same canonical state.

- Presence Accounting Engine: The why is to measure user attention, not merely an open page. The plan says it "Measures user attention via strict conjunction" of visible document state, focus, and recent pointer/keyboard activity.

- Bird Engine: The rationale is slow, hidden expressiveness. Personality vectors are "server-persisted hidden" values that drift "monotonically toward expressiveness over weeks," while moods give fast-timescale states such as wary, content, curious, drowsy, and alert.

- Natural return-greeting on session load: NOT RECOVERABLE FROM PLAN

- Listen-in spatial mix focusing: The rationale is focused attention without erasing the rest of the aviary. The listen-in dynamics boost the chosen bird while reducing others to a "-16dB ambient floor"; the plan says other birds remain "subtly audible, preserving aviary co-presence."

- Gesture offers: The plan frames seed, song fragment, and still pool offers as "gentle interactions." In the drift formula, gentle interactions such as "listen-in" and "accepted offers" contribute to expressiveness without punitive mechanics.

- Per-bird cooldowns: NOT RECOVERABLE FROM PLAN

- Opt-in settle evening transition: NOT RECOVERABLE FROM PLAN

- Field Notebook: The rationale is sparse, read-only naturalist observation. It is an "auto-generated, read-only, sparse log of naturalist prose observations," generated by the simulation tick when notebook trigger criteria are met.

- Social Affordance: The why is an ambient visit that avoids social-network power. It is "Optional & Quiet," "Read-only," "Opt-in," "instantly revocable," "off by default," and gives visitors no co-presence or interaction power.

- Accessibility Surfaces: The plan's rationale is equivalent access through multiple surfaces: screen-reader narration, call captions, keyboard navigation, WCAG AA top-bar contrast, and reduced-motion rendering.

- Audio Pipeline: The rationale is procedural, spatial, graceful sound. The plan calls for WebAudio procedural call synthesis, spatial chorus mixing, listen-in gain decay, and a "graceful silent fallback with captions."

- No Gamification: The rationale is to keep the aviary from becoming a score or competition system. The plan explicitly bans streak counters, levels, scores, badges, XP, rankings, and visit calendars.

- No Tamagotchi Mechanics: The rationale is non-punitive care. The plan bans death, hunger, distress meters, happiness decay, and says "Absence produces quietness, never negative drift or punishment."

- No Social Network Features: The rationale is to keep visits from becoming public social infrastructure. The plan bans public feeds, profiles, comments, leaderboards, public discovery directories, avatars, and co-presence.

- No Native Mobile Apps: NOT RECOVERABLE FROM PLAN

- No Push Notifications / Marketing Email: The rationale is no outbound re-engagement messaging. The plan states "Zero outbound re-engagement messaging."

### 2. Architecture & Service Boundaries

- HTML Shell & Initial Snapshot Injection: The rationale is first-frame speed. Embedding the snapshot in the initial HTML "guarantees initial frame render under 500ms without a secondary client fetch."

- Static Asset Delivery: The rationale is fast global delivery of compiled bundles, design tokens, and SVG environmental assets through "global CDN edge nodes."

- Auth & Account Service: NOT RECOVERABLE FROM PLAN

- Event Ingestion Service: The rationale is high-throughput, append-only capture of presence and interaction events. It receives batches, validates signatures, and pushes them to an append-only stream.

- Simulation Tick Engine: The rationale is autonomous canonical state production. It processes queued events, drift math, mood shifts, time/weather, notebook entries, and commits canonical snapshots.

- Social & Visit Subsystem: The rationale is read-only visiting with blocked interaction. It issues single-use 30-day visit tokens, validates visitor requests, serves read-only snapshots, and blocks event ingestion for visitors.

### 3. Data Model & Database Schemas

- Synthetic UUIDv4 identifiers and encrypted email storage: The rationale is privacy and decoupling account identity from PII. Emails are encrypted in an isolated column and never used in logs or foreign keys.

- Hidden Personality Vectors: The rationale is server-canonical personality state that is not directly exposed as UI machinery. The table is labeled "Server Canonical Only - Never Exposed to UI."

- Append-Only Interaction & Presence Event Log: The rationale is event-sourced simulation input. The log stores presence pings, listen-in, offers, and settle events for the tick engine to process.

### 4. API Surface & Contract Specifications

- Magic-link expired/error response: The rationale is the matter-of-fact system register. The plan labels the response text as "(Matter-of-fact register)."

- Account export: NOT RECOVERABLE FROM PLAN

- Account delete with 30-day soft-delete grace period: NOT RECOVERABLE FROM PLAN

- Aviary Snapshot API: The rationale is canonical read state. The snapshot exposes server time, weather, settled state, birds, mood, plumage saturation, and vocal frequency as the client-readable state.

- Aviary Event Batch API: The rationale is client notification rather than client authority. The client sends accepted event batches; the server later derives simulation changes.

- Visit snapshot and visitor `403 Forbidden` behavior: The rationale is read-only visiting. Visit sessions can fetch snapshots, but interaction submission routes return `403 Forbidden`.

- Invite revocation endpoint and unavailable visit surface: The rationale is instant revocability with matter-of-fact feedback. Revoked visitors receive `404/410` and see "This visit is no longer available."

### 5. Simulation Engine Design & Drift Physics

- Server-Side Simulation Tick Protocol: The rationale is a consistent pipeline from pending events to presence windows, drift, mood, notebook triggers, and committed canonical snapshots.

- Presence Accounting Algorithmic Logic: The rationale is valid presence only when the page is visible, focused, and recently active. The plan says a presence event is valid "if and only if" all three conditions are satisfied.

- Monotonic Personality Drift Physics: The rationale is gradual expressiveness without punishment. The calibration makes visible changes require about 21 days, and the monotonicity constraint makes absence yield no negative change.

- Mood State Transition Matrix: The rationale is fast-timescale behavior driven by local time, weather, and recent gestures, separate from slow personality drift.

### 6. Sync Architecture & Multi-Device Consistency

- Single Writer Principle: The rationale is to eliminate state divergence. Only the server-side simulation engine writes to personality vectors and bird moods.

- Append-Only Event Sourcing: The rationale is to prevent clients from overwriting canonical traits. Clients send gestures and listen-in notifications, "never absolute trait overrides."

- Snapshot Read Replicas: The rationale is consistency across devices. Laptop and phone sessions pull the same canonical snapshot from the server.

- Client-Side Interpolation: The rationale is visual smoothness. A 2.5-second lerp window prevents bird positions from "snapping or jumping."

### 7. Frontend Rendering Pipeline & Visual Scene Architecture

- Responsive fixed-aspect-ratio 2D WebGL canvas: NOT RECOVERABLE FROM PLAN

- Scene layering stack: The rationale is to map environment, perch depth, mood, and UI into a composited visual scene. The background is driven by local sunrise/sunset math, perch zones include wary/drowsy and high-boldness birds, and the UI layer includes call captions.

- Foreground Particle System: The rationale given is cosmetic ambient micro-motion. The plan calls falling leaves and feather drift "client-side purely cosmetic micro-motion."

- Top bar overlay fading after cursor stillness: NOT RECOVERABLE FROM PLAN

- Micro-Motion & Idle Procedural Physics: The rationale is procedural bird animation tied to state. Breathing cycles, head tilts, and preening sequences are triggered by curiosity, ambient calls, and content mood.

- Reduced-Motion Engine Path: The rationale is reduced motion support. Continuous skeletal spring updates and particles are disabled, and transitions become slow cross-fades between static pose snapshots.

### 8. Audio Pipeline & Procedural WebAudio Synthesis

- Choice to ship zero recorded audio files: NOT RECOVERABLE FROM PLAN

- Specific warble, trill, and nightjar motif recipes: NOT RECOVERABLE FROM PLAN

- Chorus Panning & Spatial Mixing: The rationale is spatial placement and clipping control. Each bird has dedicated gain and panner nodes mapped to perch position, and a compressor prevents clipping during chorus events.

- Listen-In Focus Dynamics: The rationale is focused listening that preserves aviary co-presence. The selected bird ramps up while other birds ramp down but remain subtly audible.

- WebAudio Fallback Architecture: The rationale is graceful silent operation. If WebAudio fails or is blocked, the engine enters `SILENT_CAPTIONED` mode without popups or alerts, and captions automatically activate.

### 9. Accessibility Surfaces & Naturalist Narration

- Naturalist Screen-Reader Narration Stream: The rationale is to feed screen readers through an invisible `aria-live="polite"` region in the same naturalist voice as the aviary.

- Narration cadence of 45 seconds during idle and immediate update on action: NOT RECOVERABLE FROM PLAN

- Procedural Call Captions: The rationale is a visible caption surface for calls, with soft typography near calling birds and automatic activation when audio falls back to silent captioned mode.

- Keyboard Navigation Contract: The rationale is full keyboard operation. Tab, arrows, Enter, and Escape cover top-bar controls, bird focus, listen-in, and modal/listen-in exit, with a high-contrast focus ring.

### 10. Performance Budgets & Observability

- Initial JS Bundle Size budget: The rationale is enforceable frontend weight control. The budget is capped at `<= 1.8 MB` gzipped and enforced via bundle analyzer in CI.

- Time-to-First-Bird Visible budget: The rationale is quick arrival into the aviary. The target is `<= 450 ms` on 4G/mid-tier mobile CPU, achieved through inline server snapshot injection.

- Runtime Frame Rate budget: The rationale is smooth idle motion on older hardware. The plan targets steady 60fps on 5-year-old mid-range hardware.

- Memory Footprint budget: The rationale is long-session stability. The plan requires zero heap accumulation over 30 continuous minutes and verifies it with automated Chrome Driver CI tests.

- Allowed Telemetry: The rationale is operational visibility without behavioral profiling. Allowed telemetry is aggregate infrastructure and runtime health data such as latency histograms, tick p99, bundle load time, and WebAudio error rates.

- Strictly Prohibited Telemetry: The rationale is privacy-preserving observability. The plan bans individual bird drift values, user interaction counts, listen-in durations per bird, and per-account behavioral profiles.

### 11. Rollout Strategy & Verification Plan

- Phase 1 launch contents: NOT RECOVERABLE FROM PLAN

- Aviary Population Ramp: The rationale is slow age-based expansion. Day 0 starts with 2 birds, Day 30 unlocks the 3rd bird invitation, Day 90 unlocks the 4th, and Day 180+ continues slow unlocks up to the 7 bird maximum cap.

- Verification Matrix: The rationale is production-grade evidence for the plan's stated budgets and risks. Tests cover bundle size, first paint, drift monotonicity, sync safety, accessibility, and memory leak behavior.

### 12. Risk Matrix & Mitigations

- Drift Calibration Mitigation: The rationale is to avoid drift that is too fast or too slow while preserving privacy. The plan tunes low-pass constants against "telemetry aggregate distributions without inspecting individual accounts."

- WebAudio Context Autoplay Mitigation: The rationale is to recover from autoplay rejection gracefully. A first canvas click resumes WebAudio, while captions display automatically if the context remains suspended.

- Browser Clock Skew Mitigation: The rationale is multi-device consistency despite local clock differences. Server timestamps anchor snapshots, and clients interpolate relative to server time.

- Voice / Tone Drift Mitigation: The rationale is register discipline. An automated string linter keeps naturalist register reserved for notebook, narration, and captions while enforcing matter-of-fact register for auth, errors, and settings.
