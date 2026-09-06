## System-level intent

- Ambient, unhurried presence as the central relationship model. This shows up in the opening description of an "ambient, browser-based virtual aviary" whose birds "evolve over days and weeks in response to unhurried human presence"; in the interaction model's "Unhurried presence"; and in presence validation that requires visible document, focus, and recent input so drift reflects actual attention.

- Slow, bounded, non-punitive change. The plan repeatedly frames change as gradual: "monotonic personality drift," low-pass-filter trait deltas, age milestones, and visible changes over 21 days. The non-goals also insist that "Absence produces ambient quietness, never penalty or negative trait drift."

- The server is the single source of canonical reality. The architecture section says "the server owns all state, simulation, drift calculations, and persistence" while "the client is a stateless rendering and input-capture node." The simulation worker is the "sole authority" for mutations, and multi-device synchronization uses a "Canonical Single-Writer Model."

- Privacy is structural, not cosmetic. The plan uses "synthetic account UUIDs," encrypted emails, blind indexes, row-level encryption, and an observability wall where telemetry must never include account ids, bird ids, trait values, interaction details, or notebook content. Visits are also "strictly isolated from host presence/drift."

- Anti-gamification and anti-custodial intent. The non-goals reject "streaks, scores, levels, badges, achievements, XP," "hunger meters," "adoption counters," and green-dot calendars. Birds "do not die, starve, fall ill, or show distress," preserving ambient quietness rather than maintenance pressure.

- Notice, never announce. This phrase appears in the voice-inconsistency risk, and the plan carries it through no "Welcome back" banners, no toasts, no level-up modals, staggered return greetings, naturalist notebook prose, and matter-of-fact system error text.

- Naturalist voice is bird-focused and quiet. The field notebook generator is "all lowercase, present-tense, bird-focused, zero exclamation points, zero user-centric metrics." Screen-reader narration must match the same "naturalist field notebook register."

- The aviary should feel alive immediately. The render plan requires an "already in motion" first frame so the aviary never "wakes up" or shows a loading spinner; bird phases are initialized from timestamp and seed so frame 1 can show mid-pose life.

- Procedural variation preserves product integrity. The plan prohibits pre-recorded audio loops, requires WebAudio FM/additive synthesis, uses deterministic animation offsets and grammar templates, and adds micro-jitter so "no vocalization is ever identical."

- Accessibility is part of the aesthetic surface. The accessibility section states that accessibility is "a primary aesthetic surface rather than an afterthought," then specifies live prose narration, call captions, reduced motion, keyboard navigation, focus contrast, and WCAG AA.

- Quiet social access without social-network dynamics. The plan allows one-time email-based read-only visits, but forbids co-presence, avatars, chat, comments, public directories, feeds, profiles, followers, reactions, and leaderboards.

- Operational observability must not become user surveillance. Allowed telemetry is operational aggregate health, while a "Strict Architectural Wall" keeps user state, interaction details, and notebook content out of the analytics sink.

## Per-feature whys

### 1. Scope & System Boundaries

- Single-User Accounts & Aviaries: NOT RECOVERABLE FROM PLAN

- Exactly one canonical aviary per account: NOT RECOVERABLE FROM PLAN

- Aviary begins with exactly two starter birds assigned from a 6-species pool: NOT RECOVERABLE FROM PLAN

- Aviary grows strictly based on aviary age milestones: NOT RECOVERABLE FROM PLAN

- Hard cap of seven birds: The plan later gives an audio rationale: the chorus "accommodates up to seven distinct call signatures before spatial and frequency masking limits recognizability."

- Passwordless email magic-link sign-in: The rationale present in the plan is security and privacy: 15-minute expiry, single-use tokens, cryptographic revocation, synthetic account UUIDs, encrypted emails, and emails never exposed across service domains.

- Internal identity uses synthetic account UUIDs: The rationale is PII isolation; the data model says all relational entities use synthetic UUIDv4 primary keys and PII is isolated and encrypted.

- Emails encrypted at rest and never exposed across service domains: The rationale is privacy management and PII isolation.

- Unhurried presence: The rationale is to make birds evolve from actual human attention rather than passive open tabs; later sections require visibility, focus, and recent input, and the risk table says false presence would corrupt the "emotional bond."

- Procedural return-greeting: The rationale is bird-led recognition without announcements. It varies by absence duration and personality, is never a simultaneous chorus, and is never accompanied by a banner or toast.

- Listen-in mix rebalancing: The rationale is "gradual acoustic soloing without muting other birds"; the audio section says other channels ramp down while "preserving presence without total muting."

- Naturalist offerings: The rationale is recoverable as gentle influence on bird state: offer proximity contributes to boldness, accepted offers contribute to curiosity, and recent accepted offers move a bird to content for 15-30 minutes.

- Per-bird cooldowns on offerings: NOT RECOVERABLE FROM PLAN

- Settle gesture: The rationale explicitly present is that it is "functionally identical to closing the tab" and expressed as a "soft twilight transition" rather than a system announcement.

- 5-second undo window for Settle: NOT RECOVERABLE FROM PLAN

- Field notebook: The rationale is sparse, read-only naturalist observation. It records notable bird-focused events every few days without user-centric metrics, exclamation points, or gamified numbers.

- Server-side continuous tick: The rationale is canonical continuity independent of client connections; it drives personality drift, Markovian mood shifts, localized diurnal/solar cycle, and weather patterns.

- Responsive single-screen Canvas2D/WebGL rendering with zero panning or zoom: The rationale is that no bird or perch is ever cropped on any screen ratio.

- Immediate "already in motion" first-frame load: The rationale is that the aviary should never "wake up" or show a loading spinner, and should meet the under-500ms Time-to-First-Bird budget.

- Pure client-side procedural audio synthesis: The rationale is no network-loaded audio loops, no bundle bloat from samples, randomized non-identical vocalizations, and preservation of product integrity.

- Screen-reader live prose narration: The rationale is an accessibility surface that speaks in the same naturalist register, with 30-60 second idle cadence rather than raw state announcements.

- Reduced-motion mode: The rationale is accessibility for reduced-motion preference while retaining static key-pose continuity and removing continuous skeletal, vertex, leaf, and feather motion.

- Real-time call captions: The rationale is accessibility and graceful silence: captions render adjacent to vocalizing birds, and are automatically enabled if AudioContext fails.

- WCAG AA compliance and complete keyboard navigation: The rationale is accessible operation of UI chrome, modal dialogs, canvas bird focus, and focus rings under all diurnal sky conditions.

- Quiet Social Visits: The rationale is read-only ambient sharing while preserving strict isolation from host presence/drift and avoiding co-presence, avatars, chat, or comments.

- One-time email-based invitations: NOT RECOVERABLE FROM PLAN

- Revocable visits: NOT RECOVERABLE FROM PLAN

- On-demand JSON state export: NOT RECOVERABLE FROM PLAN

- 30-day soft-delete grace period followed by automated hard database purge: The only rationale recoverable from the plan is "Lifecycle & Privacy Management"; the specific 30-day duration is NOT RECOVERABLE FROM PLAN.

- Native Mobile Applications out of scope: NOT RECOVERABLE FROM PLAN

- Strictly web-only with responsive mobile web: NOT RECOVERABLE FROM PLAN

- Gamification mechanics out of scope: The rationale is to prevent the aviary from becoming a metrics surface: no streaks, scores, levels, badges, achievements, XP, adoption counters, visit calendars, or exposed gamified numbers.

- Tamagotchi / Custodial Dynamics out of scope: The rationale is non-punitive ambient care. Birds do not die, starve, fall ill, show distress, or require maintenance schedules; absence creates quietness rather than penalty.

- Social Network Trappings out of scope: The rationale is quiet social access without feeds, profiles, followers, comments, reactions, leaderboards, or public directories.

- Announcements & Notifications out of scope: The rationale is the "Notice, never announce" voice; the risk table says celebratory toasts or gamified modals would break the naturalist tone.

### 2. High-Level Architecture & Service Topology

- Edge / CDN Layer: The rationale is fast asset delivery, TLS termination, proxying, and injecting a pre-rendered bootstrap snapshot to satisfy the under-500ms Time-to-First-Bird budget.

- API Gateway Service: The rationale is a stateless place for authentication, session validation, snapshot reads, event ingestion, invitation routing, rate limiting, and the boundary between host mutations and visitor read-only access.

- Simulation Worker Fleet: The rationale is a distributed background authority for 60-second ticks, event processing, drift and mood updates, notebook generation, and canonical snapshot writes.

- PostgreSQL 16 Primary Cluster / Data Tier: The rationale is transactional persistence with strict foreign keys, read replicas for snapshot queries, and encryption for sensitive columns.

- Append-Only Event Ingestion: The rationale is clients submit interaction facts while canonical mutation remains in the simulation tick; this supports sequential event resolution and avoids last-write-wins.

- Denormalized Snapshot Store: The rationale is fast snapshot delivery through API reads, SSE, read replicas, and bootstrap cache.

- Observability Pipeline (Isolated): The rationale is collecting tick latency, throughput, locks, and WebAudio error counters while remaining structurally isolated from user data.

- Bundle JS under 2MB: The rationale is fresh cold-load performance, enforced by bundle analyzer in CI.

- Bootstrap State Snapshot Cache: The rationale is Time-to-First-Bird performance and first-paint continuity.

### 3. Data Model & Storage Schema

- Synthetic UUIDv4 primary keys for relational entities: The rationale is privacy-preserving identity that does not expose email or other PII.

- AES-256-GCM encrypted PII with external KMS: The rationale is PII isolation and encrypted sensitive columns.

- Blind index for email lookups: The rationale is email lookup without storing or exposing plaintext email.

- Account status with active, pending_deletion, purged: The rationale is lifecycle and privacy management around deletion and purge.

- Sessions with token hash and revoked_at: The rationale is revocable session authentication.

- Magic links with token hash, expiry, and consumed_at: The rationale is single-use, expiring passwordless authentication.

- Aviaries with timezone, weather, transition time, and last tick: The rationale is localized diurnal/solar cycle, subtle weather patterns, and worker tick continuity.

- Birds with personality vectors: The rationale is normalized continuous bird traits for long-term drift, visible changes, vocal behavior, curiosity, and perch behavior.

- Current mood and perch zone on birds: The rationale is fast-timescale behavior and scene placement driven by mood, boldness, weather, interaction events, and solar phase.

- Interaction events queue: The rationale is append-only input capture from clients for the simulation worker to process into canonical state.

- Unprocessed interaction event index: The rationale is efficient worker dequeue of events whose processed_at is null.

- Aviary snapshots read model: The rationale is denormalized current state delivery to clients, stream consumers, and bootstrap payloads.

- Notebook entries: The rationale is persisted naturalist prose observations with pagination by aviary and time.

- Visit invitations with hashed token and encrypted visitor email: The rationale is one-time email-based read-only visits with privacy-preserving visitor identity and revocation.

- Visit logs: The rationale explicitly present is "host review" of recent visits.

### 4. API Surface & Contract Specifications

- Standard HTTP status codes: NOT RECOVERABLE FROM PLAN

- Matter-of-fact client error messages: The rationale is product voice separation: naturalist prose belongs to product surfaces, while system errors stay matter-of-fact.

- `POST /api/v1/auth/magic-link`: The rationale is passwordless sign-in without revealing whether an address is registered.

- `POST /api/v1/auth/verify`: The rationale is exchanging a magic token for an HttpOnly, Secure, SameSite=Lax session cookie.

- `POST /api/v1/auth/logout`: The rationale is invalidating the session record.

- `POST /api/v1/account/export`: NOT RECOVERABLE FROM PLAN

- `POST /api/v1/account/delete`: The rationale is scheduling account deletion as part of lifecycle and privacy management.

- `POST /api/v1/account/restore`: The rationale is canceling a deletion request during the soft-delete grace period.

- `GET /api/v1/aviary`: The rationale is returning the current canonical state snapshot.

- `GET /api/v1/aviary/stream`: The rationale is emitting canonical snapshots every 60-second tick or on immediate environmental transitions.

- `POST /api/v1/aviary/events`: The rationale is batching client interaction events as input to server-owned simulation.

- `GET /api/v1/aviary/notebook`: The rationale is paginated access to naturalist observations.

- `POST /api/v1/invitations`: The rationale is creating email-based visit invitations.

- `DELETE /api/v1/invitations/{invite_id}`: The rationale is immediate revocation of a visit invitation.

- `GET /api/v1/invitations/log`: The rationale is recent-visit host review.

- `GET /api/v1/visit/{visit_token}`: The rationale is public read-only invited access that validates tokens, returns the snapshot stream, and disallows writes.

### 5. Simulation Engine Design

- Simulation worker runs independently of client connections: The rationale is that client presence is not required for canonical mood, drift, environmental state, or notebook generation.

- Tick Scheduler Execution Cycle every 60 seconds: The rationale is a continuous, discrete cadence for event pull, presence accounting, drift, mood, perch, notebook, adoption, and snapshot updates.

- Event Pull up to current tick time: The rationale is processing unhandled interaction events in canonical order.

- Presence Accounting: The rationale is validating genuine presence and crediting at most 60 seconds per 60-second tick even across multiple devices.

- Drift Function Execution using discrete Low-Pass Filter: The rationale is slow, measurable trait evolution rather than erratic jumps.

- Monotonic Invariant for personality vectors: The rationale is non-punitive change; traits never decrement, and no presence means no drift.

- Calibration constants: The rationale is to make 7-day changes measurable and 21-day changes visibly distinct while respecting saturation ceiling.

- Fast-Timescale Mood State Machine: The rationale is mood responsiveness to solar phase, weather, recent accepted offers, and adjacent bird alarm calls.

- Perch Selection Model: The rationale is making boldness, contentment, and wariness visibly affect front, middle, and back perch probabilities.

- Procedural Field Notebook Generator: The rationale is sparse observation of notable differential events, rendered in deterministic naturalist grammar and stripped of user-centric metrics.

- Strict sparsity gate for notebook entries: The rationale is quiet observation, with at least 48 hours between standard entries.

- Relative greeting order notes: The rationale is documenting notable differential events compared to historic logs.

- Extended quiet or weather reaction notes: The rationale is documenting notable environmental and behavioral changes.

- Distinct perch choices under specific weather notes: The rationale is documenting bird behavior in relation to weather.

- Adoption Threshold Evaluator: NOT RECOVERABLE FROM PLAN

- Day 0, 30, 90, 180, 270, and 360 bird arrivals: NOT RECOVERABLE FROM PLAN

### 6. Multi-Device Synchronization & Conflict Prevention

- Canonical Single-Writer Model: The rationale is avoiding split-brain personality states, corrupted vectors, desynchronization, and last-write-wins.

- Clients never run local simulation ticks: The rationale is that all devices receive identical canonical snapshots rather than pushing vector mutations.

- Presence Disambiguation with visibility, focus, and recent input: The rationale is preventing background tabs or forgotten browser windows from accumulating false presence time.

- Presence heartbeat every 60 seconds: The rationale is sending presence only when the three presence rules conform.

- Server union of presence intervals: The rationale is making double-counting attention impossible across multiple sessions.

- Settle State Synchronization: The rationale is that one device's settled state becomes canonical and other devices ease into the settled lighting state through SSE.

- Visual twilight ease on Settle: The rationale recoverable from the plan is soft transition into settled lighting rather than abrupt UI change.

- Click-anywhere undo during Settle window: NOT RECOVERABLE FROM PLAN

### 7. Frontend Rendering Pipeline

- Fixed 1920 x 1080 virtual viewport: The rationale is stable single-screen rendering that can be scaled into the browser.

- CSS object-fit contain: The rationale is ensuring no bird or perch is ever cropped on any screen ratio.

- Top Bar Chrome with Settings, Accessibility, Field Notebook, Offer Gift: NOT RECOVERABLE FROM PLAN

- Background Layer with diurnal sky, sun/moon elevation, and weather tint: The rationale is visualizing localized diurnal/solar cycle and subtle weather patterns.

- Back, Middle, and Front Perch Zones: The rationale is expressing low boldness, wary, resting, high boldness, and alert states through spatial placement.

- Foreground parallax twigs and rare drifting leaves / down feathers: NOT RECOVERABLE FROM PLAN

- "Already in Motion" First Frame Load: The rationale is meeting the requirement that the aviary never wakes up or shows a loading spinner.

- Bootstrapped state snapshot in initial HTML: The rationale is parsing state before first paint and satisfying Time-to-First-Bird.

- Deterministic mathematical animation offsets: The rationale is rendering birds mid-pose on frame 1 using now timestamp and bird seed.

- Fallback ambient sky and resting foliage during delayed snapshot: The rationale is avoiding a blank or spinner-like cold load while birds fade in within the 500ms window.

- Return-Greeting Choreography for less than 5 minutes, 5 minutes to 24 hours, and more than 24 hours: The rationale is scaling greeting behavior to absence duration, from ambient glance to extended procedural sequence.

- Strictly staggered greetings: The rationale is preventing simultaneous choruses.

- 60 FPS Idle Micro-Motion Engine: The rationale is continuous ambient life through respiration, weight-shift, and head scanning.

- Head scanning modulated by curiosity and wary mood: The rationale is making personality and mood visible in motion.

- Halt animation loop when tab is hidden: The rationale is conserving battery and CPU resources.

- Reduced-Motion automatic detection and toggle: The rationale is respecting reduced-motion preference through accessibility settings.

- Static key-pose cross-fades in reduced motion: The rationale is replacing continuous skeletal and vertex animations with slow visual transitions.

- Cross-faded perch transitions in reduced motion: The rationale is preserving transition comprehension without flight motion.

- Removing leaf and feather drift particles in reduced motion: The rationale is eliminating ambient particle motion.

### 8. Procedural Audio Pipeline

- WebAudio-only AudioContext engine: The rationale is zero network audio files or sample loops.

- Master Gain with Settle Ease: NOT RECOVERABLE FROM PLAN

- Ambient Soundscape Bus: The rationale is procedural wind and rain through noise and filters, with attenuation during listen-in.

- Bird Chorus Bus with per-bird sub-buses: The rationale is supporting up to seven bird voices with independent gain, panning, and signatures.

- Per-Bird Voice Generator: The rationale is distinct procedural voice generation per bird with carrier, FM modulator, syrinx bandpass filter, stereo panner, and gain.

- Syrinx Synthesis Model: The rationale is simulating the avian vocal organ through FM oscillators and formant-shaping bandpass filtering.

- Carrier pitch range, modulator range, and formant filter: The rationale is pitch, trill/chirp modulation, and biological resonance.

- Micro-Jitter: The rationale is that no vocalization is ever identical.

- Poisson vocalization schedule: The rationale is scheduling calls from each bird's `vocal_frequency`.

- Chorus Interaction & Avoidance: The rationale is preventing acoustic crowding.

- Reciprocal chorus response for high social_warmth birds: The rationale is answering another bird's call within a timed window according to social warmth.

- Listen-In Mix Dynamics: The rationale is focusing one bird while preserving the audible presence of other birds and ambient sound.

- Equal-power crossfade ramps: The rationale is smooth gain transition during listen-in and disengage.

- Graceful Silence Mode: The rationale is handling failed, blocked, or unsupported AudioContext without a synthetic error banner.

- Automatic captions when audio fails: The rationale is keeping vocal events legible adjacent to birds in silence.

- Strict prohibition on pre-recorded sample fallbacks: The rationale is preventing bundle bloat and preserving product integrity.

### 9. Accessibility Surfaces

- Accessibility implemented as a primary aesthetic surface: The rationale is explicitly stated: it is not an afterthought.

- Screen-reader live region: The rationale is polite, atomic naturalist prose updates for assistive technology.

- 30-60 second idle narration cadence: The rationale is avoiding flooding screen-reader queues while preserving semantic stillness.

- Immediate priority narration for user actions: The rationale is surfacing return-greeting, offer acceptance, and settle without raw state announcements.

- Procedural Call Captions: The rationale is synchronized text generated directly from active call motif grammar.

- Subtitle pills adjacent to calling birds: The rationale is associating caption text with the vocalizing bird.

- Caption fade timings: NOT RECOVERABLE FROM PLAN

- Logical Tab sequence: The rationale is complete keyboard navigation through chrome, canvas bird focus, listen-in, and modals.

- Arrow-key 2D scene proximity focus: The rationale is keyboard movement between birds according to scene layout.

- Enter / Space listen-in and Escape disengage: The rationale is keyboard operation of listen-in and modal closing.

- Dual-stroke focus outline: The rationale is guaranteed minimum 4.5:1 contrast across dawn, midday, dusk, and midnight sky conditions.

- WCAG AA contrast for chrome, labels, and modal dialogs: The rationale is accessible readability.

- Top bar chrome contrast adjustment during diurnal lighting shifts: The rationale is maintaining text and icon contrast as the background changes.

### 10. Performance Budgets & Observability

- Initial JS Bundle Size under 2MB gzipped: The rationale is fresh cold-load performance, enforced by CI.

- Time to First Bird under 500ms: The rationale is immediate first-bird visibility on mid-tier mobile over 4G through bootstrap snapshot and critical CSS.

- 60fps continuous animation: The rationale is smooth continuous rendering on a 5-year-old laptop through lightweight canvas draw calls and zero render-loop GC allocation.

- 0MB net memory leak over 30 minutes: The rationale is stable long sessions through bounded audio node pools, reusable canvas buffers, and leak tests.

- Simulation tick latency p99 under 5 seconds: The rationale is processing 10,000 aviaries within worker budget and alerting on slow ticks.

- Allowed Telemetry: The rationale is operational health through status rates, cache hit ratios, connection saturation, tick histograms, WebAudio failure rates, WebGL loss counts, and aggregate session buckets.

- Strict Architectural Wall for telemetry: The rationale is preventing account ids, bird ids, traits, interaction details, and notebook content from entering telemetry.

- Physically distinct network egress path for telemetry: The rationale is independent credentials and separation from user data pathways.

### 11. Rollout & Phased Deployment Strategy

- Phase 1 Audio & Engine Verification: The rationale is validating procedural syrinx synthesis across major browsers and verifying personality drift stability and monotonic convergence through accelerated 1-year simulations.

- Phase 2 Closed Internal Dogfooding: The rationale is calibrating presence accounting idle windows and verifying multi-device sync on desktop and mobile.

- Phase 3 Quiet Alpha Cohort: The rationale is validating visit invitations and verifying zero cross-contamination of host drift.

- Phase 4 General Availability v1: The rationale is launching with strict adherence to the 2-to-7 bird age progression schedule.

- Day-One Simulation worker tick duration dashboard: The rationale is tracking p50, p95, and p99 worker timing.

- Magic-link email delivery latency and bounce dashboard: The rationale is monitoring authentication email delivery.

- Client-side WebAudio initialization success vs. silent fallback dashboard: The rationale is monitoring audio availability and graceful silence.

- Database replication lag dashboard: The rationale is monitoring primary-to-snapshot-replica freshness.

### 12. Technical Risks & Mitigation Strategies

- Drift Calibration mitigation: The rationale is preserving the affective bond by avoiding birds that feel erratic, like a Tamagotchi, or like a static screensaver.

- Multi-Device Race Conditions mitigation: The rationale is preventing split-brain personality states, corrupted vectors, and desynchronization through single-writer sequential event resolution.

- Audio Uncanniness & Fatigue mitigation: The rationale is protecting the core emotional affordance by preventing harsh, synthetic, or repetitive calls that make users mute audio.

- Accessibility Regressions mitigation: The rationale is preventing live-region flooding that would force a user to silence the screen reader.

- Presence Invalidation mitigation: The rationale is preventing background tabs and forgotten windows from accumulating false presence time and corrupting the emotional bond.

- Voice Inconsistency mitigation: The rationale is protecting "Notice, never announce" and naturalist tone from celebratory toasts or gamified modals.
