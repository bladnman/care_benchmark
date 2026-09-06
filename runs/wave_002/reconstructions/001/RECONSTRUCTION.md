## System-level intent

- Observational relationship, not custody or game: The plan frames Pocket Aviary as "an observational relationship rather than a custodial or gamified one." This shows up again in "No Gamification Elements," "No Tamagotchi / Custodial Mechanics," "uncurated, read-only, non-gamified" notebook behavior, and the rule that "absence causes quietness, never suffering."
- Feels alive, not robotic: The plan anchors aliveness in "continuous server-side simulation," an "authoritative server tick (~60s)," birds rendered "mid-motion," no "wake-up" transitions, and calls that are "procedurally synthesized" rather than recorded loops.
- Notice, never announce: The user return experience is intentionally indirect. The plan forbids "Welcome back!" toasts, banners, streaks, milestone badges, and arrival dialogs; the "return-greeting is performed exclusively by the birds" through "absence-modulated procedural calls and posture shifts."
- Charm comes from specificity: The plan enforces a "dual-voice architectural separation." Naturalist Voice is "lowercase, present-tense, bird-centric prose" for product surfaces; Matter-of-Fact Voice is "standard capitalized, concise technical clarity" for system surfaces.
- Restraint over richness: The plan repeatedly narrows scope to "exactly one horizontal scene," "fixed 1-screen viewport," "no scrolling, panning, or zooming," two starter birds, and a "hard ceiling of seven birds" to preserve "psychoacoustic call discriminability."
- Presence as real interaction: "Idle attention is the primary input to personality drift." The plan treats a user "sitting and watching without clicking" as the "dominant signal" and makes drift monotonic, with "zero negative decay on absence."
- Server authority and additive events: The plan's client/server boundary keeps the server as "single source of truth" and rejects "Last-Write-Wins" client state. Clients emit events and heartbeats; the server processes them sequentially and accumulates presence additively.
- Quiet, non-social social visits: The plan allows "optional, quiet, email-invited read-only ambient viewing" while excluding "social network mechanics," visitor interactions, co-presence, visitor drift impact, and default notifications.
- Privacy by architectural partition: The plan keeps email only in an encrypted account column, uses synthetic UUIDs in foreign keys, logs, metrics, and queues, and prohibits telemetry containing bird names, species selections, personality vectors, mood states, presence durations, notebook texts, offer choices, or visitor emails.
- Accessibility without changing the voice: Accessibility surfaces still use the same naturalist language and calm cadence: `aria-live="polite"` narration, procedural call captions, keyboard navigation, reduced-motion cross-fades, and WCAG AA contrast.

## Per-feature whys

**Executive Summary & Design Principles Alignment**

- Ambient, browser-based virtual aviary: The plan's why is to host birds in a "single, responsive horizontal scene" and model "an observational relationship rather than a custodial or gamified one."
- Two to seven procedurally animated birds: The plan ties the lower bound to adoption and the upper bound to restraint, "relationship depth," and "psychoacoustic call discriminability."
- Continuous server-side simulation: The plan's why is that birds "live" even when tabs are closed and avoid robotic "wake-up" transitions.
- Procedural vocalizations: The plan's why is "Feels Alive, Not Robotic," with client-side WebAudio and no "recorded audio loops."
- Initial paint in mid-motion: The plan's why is to avoid "wake-up" transitions, entry animations, loading spinners, a black frame, fade-in, or loader animation.
- Return-greeting: The plan's why is "Notice, Never Announce"; arrival is noticed through birds, not system messages.
- Slow, monotonic personality drift: The plan's why is "presence" and "weeks of idle attention," with neglect never penalized.
- Naturalist Voice: The plan's why is "Charm Comes from Specificity" and bird-centric prose on product surfaces.
- Matter-of-Fact Voice: The plan's why is "concise technical clarity" on system surfaces.
- One horizontal scene: The plan's why is "Restraint Over Richness" and a viewport that fits without scrolling, panning, or zooming.
- Presence as dominant signal: The plan's why is that "a user sitting and watching without clicking provides the dominant signal."

**Scope & Boundary Definition**

- Single Horizontal Scene: The plan's why is a "fixed 1-screen viewport" across desktop, tablet, and mobile with three depth zones.
- Bird Population & Capacity: The plan's why is relationship depth, age-based arrival, and a "hard ceiling" that preserves call discriminability.
- Customizable bird names: The plan connects this to individual bird identity and specificity.
- Additional birds unlocked exclusively by aviary age milestones: The plan's why is that no interaction volume, streaks, or purchases should accelerate relationship progression.
- Procedural Call Synthesis: The plan's why is real-time generated calls with "No audio loops or recorded audio files."
- Chorus & Spatial Mixing: The plan's why is a multi-bird sound field mapped to perch positions with ambient reverb.
- Return-Greeting interaction: The plan's why is a procedural notice on session start, modulated by absence, boldness, and mood.
- Listen-In interaction: The plan's why is to focus a single bird while retaining a "non-zero ambient floor" for the rest of the aviary.
- Offer interaction: The plan's why is to let offers be shaped by "curiosity and mood" while gating them with cooldowns.
- Settle interaction: The plan's why is an evening shift that is "functionally equivalent to closing the tab," with a 5-second undo window.
- Field Notebook: The plan's why is a sparse "naturalist observer log" that stays "uncurated, read-only, non-gamified."
- Presence Sentinel: The plan's why is strict true-presence tracking, not passive background accumulation.
- Single-user accounts and sync: The plan's why is one "canonical aviary per account" across devices.
- 15-minute email magic links: NOT RECOVERABLE FROM PLAN
- Synthetic UUID account isolation: The plan's why is preventing PII leakage into logs, caches, metrics, keys, queues, and telemetry.
- Social Visits: The plan's why is "optional, quiet" read-only ambient viewing with no visitor drift impact or visitor interactions.
- Accessibility: The plan's why is screen-reader narration, captions, keyboard navigation, reduced motion, and WCAG AA compliance.
- JSON aviary state export: The plan's why is data and privacy control.
- 30-day soft deletion transitioning to permanent purge: The plan's why is data and privacy control with a restore window.
- Strict telemetry isolation: The plan's why is "zero per-bird data in analytics."
- No Native Applications: NOT RECOVERABLE FROM PLAN
- No Gamification Elements: The plan's why is the non-gamified observational relationship.
- No Tamagotchi / Custodial Mechanics: The plan's why is that absence causes "quietness, never suffering."
- No Social Network Mechanics: The plan's why is quiet social visiting rather than profiles, feeds, chat, avatars, discovery, or co-presence.
- No Recorded Audio Fallback: The plan's why is graceful silence and captions instead of shipping sample audio files.
- No Client Authoritative State: The plan's why is that clients never compute or mutate personality vectors or moods directly.

**High-Level Architecture & System Boundaries**

- Canvas / WebGL presentation layer: The plan's why is 60fps rendering of a single scene, perch zones, parallax, micro-motion, and reduced-motion cross-fade.
- WebAudio procedural synthesizer and mixer: The plan's why is procedural calls, real-time chorus, stereo mixing, listen-in crossfade, silent fallback, and caption triggers.
- State and Presence Engine: The plan's why is interpolation between authoritative snapshots plus debounced presence reporting.
- Accessibility Surfaces: The plan's why is polite prose narration, call captions, and high-contrast keyboard navigation.
- Edge CDN and inlined initial snapshot: The plan's why is critical asset delivery, a small bundle, and first-frame paint without a secondary network fetch.
- Reverse proxy and rate limiting: NOT RECOVERABLE FROM PLAN
- API Gateway Service: The plan's why is authentication, snapshot, event ingest, visit, export, and deletion surfaces.
- Simulation Worker Daemon: The plan's why is the authoritative tick, append-only event consumption, monotonic drift, mood, diurnal state, and notebook generation.
- PostgreSQL persistence: The plan's why is relational storage for accounts, aviaries, birds, vectors, moods, and events.
- Redis storage: The plan's why is tick locks, active session caches, and SSE pub/sub.
- Server responsibilities: The plan's why is one source of truth for aviary state, time, weather, bird identity, personality vectors, moods, and notebook observations.
- Client responsibilities: The plan's why is smooth local rendering, non-authoritative ambient motion, real-time procedural audio, presence signals, narration, and captions.

**Comprehensive Data Model & Database Schemas**

- Encrypted email on accounts only: The plan's why is to prevent PII leakage and decrypt only for link generation, settings display, and export operations.
- Email hash: The plan's why is login lookups without using email as an internal identifier.
- Sessions table: NOT RECOVERABLE FROM PLAN
- Magic links table: NOT RECOVERABLE FROM PLAN
- Aviaries table: The plan's why is canonical per-account state for timezone, weather, settle state, and tick time.
- Birds table: The plan's why is bird identity, species, custom names, and unique perch positions.
- Personality vectors table: The plan's why is strictly server-side hidden traits that drive behavior, calls, plumage, and curiosity.
- Bird moods table: The plan's why is a fast-timescale emotional state separate from slow personality drift.
- Append-only interaction event log: The plan's why is sequential, additive processing and conflict prevention.
- Notebook entries table: The plan's why is storing system-authored naturalist observations and focal birds.
- Visit invitations table: The plan's why is email-invited, token-based visits that can expire or be revoked.
- Visit logs table: The plan's why is chronological visit history recorded silently.
- Snapshot data contract: The plan's why is to deliver current aviary state on session open, periodic synchronization, and SSE updates.
- `call_signature` in snapshots: The plan's why is that the client synthesizes calls in real time from motif grammar instructions.
- `notebook_snippet` in snapshots: The plan's why is surfacing the latest naturalist observation without opening the full notebook.

**API Surface & Communication Protocols**

- `/api/v1/auth/magic-link`: NOT RECOVERABLE FROM PLAN
- `/api/v1/auth/verify`: NOT RECOVERABLE FROM PLAN
- `/api/v1/auth/sessions`: NOT RECOVERABLE FROM PLAN
- `/api/v1/auth/sessions/:id`: NOT RECOVERABLE FROM PLAN
- `/api/v1/aviary/snapshot`: The plan's why is fetching current authoritative aviary state with conditional caching.
- `/api/v1/aviary/stream`: The plan's why is SSE delivery of tick snapshots, weather shifts, and live updates.
- `/api/v1/aviary/events`: The plan's why is batching client interaction events and presence heartbeats for server-side processing.
- `/api/v1/aviary/notebook`: The plan's why is paginated access to field notebook observations.
- `/api/v1/birds/:id/rename`: The plan's why is customizable bird names while only mutating `custom_name`.
- `/api/v1/visits/invitations`: The plan's why is issuing quiet email visit invites with daily rate limiting.
- `/api/v1/visits/invitations/:id`: The plan's why is immediate host revocation of a visit invitation.
- `/api/v1/visits/log`: The plan's why is chronological visit history for the host.
- `/api/v1/visits/view/:token`: The plan's why is public token-based read-only visitor access, returning 403 when revoked or expired.
- `/api/v1/visits/stream/:token`: The plan's why is visitor SSE for read-only aviary updates.
- `/api/v1/account/export`: The plan's why is account state packaging and a secure download link.
- `/api/v1/account`: The plan's why is 30-day soft deletion.
- `/api/v1/account/restore`: The plan's why is cancelling soft deletion within 30 days.
- Interaction event ingestion payload: The plan's why is debounced 30-second batches, with immediate submission on key interaction boundaries.
- Error handling and voice division: The plan's why is Matter-of-Fact system voice for authentication, session, visit, and loading failures.

**Simulation Engine Design & Drift Mathematics**

- 60s Simulation Worker Daemon: The plan's why is an authoritative server tick regardless of open tabs.
- Redis aviary lock: The plan's why is preventing duplicate tick execution across workers.
- Drain Event Log: The plan's why is processing unconsumed events ordered by `id ASC`.
- Calculate True Presence Seconds: The plan's why is accumulating only valid three-factor `presence_ping` duration.
- Compute Personality Drift Deltas: The plan's why is monotonic drift from presence, listen-in, and offers.
- Diurnal and Weather Dynamics: The plan's why is solar lighting, ambient weather, and mood/call modulation.
- Mood State Machine: The plan's why is a fast-timescale emotional state distinct from personality.
- Return-Greeting Queue: The plan's why is greeting schedules for newly returning sessions.
- Field Notebook Heuristics: The plan's why is generating naturalist observations when threshold conditions are met.
- Transactional canonical state commit and Redis SSE publish: The plan's why is updated state plus active-client propagation.
- Three-factor presence rule: The plan's why is that background tabs never generate drift.
- 180-second idle cutoff: The plan's why is halting presence reporting until the next input event.
- Personality trait model: The plan's why is to connect hidden traits to front-perch behavior, flock chorus, call frequency, plumage, and offer inspection.
- Asymmetric monotonic drift function: The plan's why is that traits move up on positive presence and interactions and "never move down due to absence or neglect."
- Drift calibration constants: The plan's why is measurable change after one week and felt aliveness after three weeks.
- Mood transition modulators: The plan's why is boldness resisting fear, wary social contagion, and mood persistence across sessions.
- Inhomogeneous Poisson call frequency: The plan's why is unobserved call rate shaped by species, vocal frequency, mood, diurnal phase, and weather.
- Six-species motif pool: The plan's why is species-specific acoustic variety.
- Absence-Modulated Return-Greeting Algorithm: The plan's why is greeting strength and sequence shaped by absence, boldness, warmth, and staggered response.

**Multi-Device Synchronization & Conflict Prevention**

- Single Canonical State and Anti-Conflict Model: The plan's why is to avoid split-brain and reconciliation conflicts.
- No Last-Write-Wins on Client State: The plan's why is that clients emit events, not state replacements.
- Additive Server Accumulation: The plan's why is supporting simultaneous phone and desktop presence while capping to 60 seconds per clock minute.
- Snapshot propagation via SSE: The plan's why is pushing differential snapshots to all active connections for an aviary.
- Snapshot polling fallback: The plan's why is continued synchronization if SSE fails or disconnects.
- No teleporting birds between perches: The plan's why is organic flight or hop transition instead of disjointed state changes.
- Plumage saturation lerp: The plan's why is smooth visual transition to the target value.
- Audio parameter smoothing: The plan's why is smooth local synthesis updates after snapshots.

**Frontend Rendering Pipeline**

- Full-viewport Canvas scene: The plan's why is a one-screen aviary scene with no scrolling, panning, or zooming.
- Responsive framing: The plan's why is that perches remain 100% visible on desktop, tablet, and mobile.
- Sky and Solar Backdrop: The plan's why is lighting mapped to solar elevation and time of day.
- Far Background parallax: NOT RECOVERABLE FROM PLAN
- Middle Ground perch depth: The plan's why is front, middle, and back perch zones with scale and desaturation to express depth.
- Ambient Particles: The plan's why is client-side atmosphere with "zero simulation state."
- UI Chrome layer: The plan's why is minimal top-bar navigation overlay and call captions.
- 60fps Idle Micro-Motion Engine: The plan's why is continuous motion that "avoids robotic looping."
- Breathing: The plan's why is procedural chest scale fluctuation.
- Scanning and Head-Tilts: The plan's why is randomized micro-rotations derived from curiosity.
- Preening: The plan's why is a content mood behavior.
- First Frame Rendering: The plan's why is immediate mid-motion rendering without loader, fade, or black frame.
- Battery Optimization: The plan's why is pausing the render loop when the document is hidden.
- Reduced-Motion Mode: The plan's why is replacing particles, parallax, skeletal micro-motion, and flight paths with cross-fading still poses and smoother lighting.
- Top-Bar Chrome Auto-Fade: The plan's why is minimal chrome that fades during cursor stillness and restores on interaction.

**Procedural Audio Pipeline**

- Algorithmic Sound Generation Architecture: The plan's why is operating "strictly without audio files or sample loops."
- WebAudio node topology: NOT RECOVERABLE FROM PLAN
- FM Carrier Synthesis: The plan's why is mathematical bird-call generation.
- Warbler synthesis: The plan's why is rapid ascending multi-note sweeps.
- Wren synthesis: The plan's why is energetic, syncopated trills and staccato clicks.
- Dove coo synthesis: The plan's why is soft, low-frequency resonant dual-tone coos.
- Chorus Superposition: The plan's why is natural blending without acoustic phase cancellation from layered samples.
- Spatial Positioning: The plan's why is mapping a bird's perch X coordinate into stereo position.
- Listen-In Dynamics: The plan's why is a focus gain boost while non-focused birds stay at a non-zero ambient floor.
- Graceful WebAudio Fallback: The plan's why is graceful silence and automatic captions when AudioContext is unavailable.

**Accessibility Surfaces**

- Running Screen-Reader Narration: The plan's why is a polite live region with naturalist, lowercase, present-tense, bird-centric observations.
- Idle narration cadence: The plan's why is one observation every 45-60 seconds to avoid flooding.
- Immediate narration on user actions: The plan's why is prioritized observation of interaction outcomes.
- Procedural Call Captions: The plan's why is captions when toggled on or when audio context is disabled.
- Motif-derived caption text: The plan's why is caption text generated directly from active synthesis parameters.
- Keyboard Navigation: The plan's why is full interactive control without a pointing device.
- Focus Indicator: The plan's why is WCAG AA visibility against bright midday sky and dark evening backdrops.
- Contrast Verification: The plan's why is WCAG 2.1 AA compliance for chrome text, captions, notebook entries, and icons.

**Performance Budgets & Observability Boundary**

- Initial JS Bundle Size budget: The plan's why is critical asset delivery under a 2MB gzipped budget.
- Time to First Bird budget: The plan's why is a first bird visible in under 500 ms on mid-tier mobile over simulated 4G LTE.
- Idle Render Frame Rate budget: The plan's why is continuous 60fps during a 30-minute session.
- Client Memory Growth budget: The plan's why is zero leak over 30 minutes.
- Simulation Tick Latency budget: The plan's why is p99 worker tick latency below 5 seconds.
- Critical Path HTML Inlining: The plan's why is that the render engine does not wait for a secondary fetch before first frame.
- Aggressive Code Splitting: The plan's why is keeping core rendering, audio, presence, and snapshot parsing small while lazy-loading settings and notebook code.
- Zero Recorded Audio Assets: The plan's why is saving 15-30 MB and fitting within the 2MB budget.
- Allowed telemetry boundary: The plan's why is observability for request rate, latency, tick duration, cache hit ratio, FPS, dropped frames, and WebAudio initialization errors.
- Prohibited telemetry boundary: The plan's why is strict privacy isolation for bird-specific data, presence, notebook text, offers, and visitor email addresses.
- Logging rule: The plan's why is keeping application logs to synthetic `account_id` and `aviary_id` UUIDs only.

**Social Visits Flow & Implementation**

- Invitation creation: The plan's why is host-initiated "Invite a Friend" from Settings.
- Cryptographically secure visit token and hash storage: The plan's why is token-based access with hashed storage.
- 30-day visit invitation expiry: NOT RECOVERABLE FROM PLAN
- Visitor read-only snapshot and SSE session: The plan's why is read-only ambient access to the host's aviary.
- Instant revocation: The plan's why is terminating visitor stream immediately and showing a matter-of-fact inactive-invitation message.
- Identical visitor visual and audio rendering: The plan's why is the same birds, perches, moods, weather, and daylight phase.
- Interaction stripping: The plan's why is a visitor interface with zero controls.
- Zero Presence Attribution: The plan's why is visitor attention generating zero drift for host birds.
- No Co-Presence Markers: The plan's why is no visual or audio indication that a visitor is watching.
- Notifications Disabled by Default: The plan's why is visits logged silently unless the host explicitly opts in.

**Implementation Phases, Rollout & Capacity Ramp**

- Milestone 1, Simulation Core & Data Engine: The plan's why is to establish schemas, synthetic UUID isolation, email encryption, simulation tick, drift, diurnal clock, authentication, and sessions first.
- Milestone 2, WebAudio Procedural Synthesis: The plan's why is to build species synthesis, chorus, stereo positioning, listen-in curves, graceful silence, and captions before rendering completion.
- Milestone 3, Canvas Rendering & Micro-Motion: The plan's why is to build responsive 60fps rendering, perch zones, sky, zero-delay motion, and reduced-motion mode.
- Milestone 4, Interactions, Field Notebook & Accessibility: The plan's why is to add return greeting, presence sentinel, offer, settle, narration, and keyboard navigation together.
- Milestone 5, Social Visits, Hardening & Verification: The plan's why is to finish read-only visits, revocation, memory-leak stress testing, bundle enforcement, integration, and drift calibration.
- Aviary Capacity Ramp: The plan's why is "to preserve the sense of relationship depth" and prevent acceleration by interaction volume, streaks, or purchases.

**Technical & Design Risks and Mitigations**

- Drift calibration test suites: The plan's why is preventing birds from reaching maximum boldness in 3 days or showing zero change after 2 months.
- Presence spoofing and background drain mitigation: The plan's why is preventing falsely accumulated weeks of presence time.
- Multi-device split-brain mitigation: The plan's why is avoiding disjointed bird moods or lost drift progress across phone and laptop.
- Audio uncanniness mitigation: The plan's why is preventing metallic, shrill, or fatiguing sound during extended 30-minute listening sessions.
- Screen-reader live region throttling: The plan's why is avoiding a flooded speech queue that becomes annoying and unusable.
- Memory leak mitigation: The plan's why is preventing heap growth from 60fps canvas animation and dynamic WebAudio node creation over long sessions.
