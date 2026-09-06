## System-level intent

- **Observational, unhurried relationship**: The headline concept defines Pocket Aviary as an "ambient virtual aviary" that establishes an "observational, unhurried relationship" with a small flock. This shows up across the single horizontal scene, quiet session interactions, sparse notebook, and the out-of-scope rejection of gamification, custodial systems, and push re-engagement.

- **Feels alive, not robotic**: The plan names this as a fundamental design principle. It appears in the server-side simulation tick, the mandate that "the first frame rendered has motion already underway," the "Already Alive" first frame rules, and procedurally varied calls and greetings so "no two moments sound identical."

- **Notice, never announce**: The plan names this principle and applies it to Return-Greeting: user arrival is registered through "subtle bird behavioral reactions" such as "glances, head-tilts, soft calls," with "no arrival toasts, welcome banners, level-up confetti, or streak popups."

- **Charm comes from specificity**: The plan says product prose uses a "naturalist field-notebook register" with "lowercase, present-tense, specific, bird-centric verbs." This governs notebook entries, captions, screen-reader narration, and the prohibition on raw system state such as `"bird position: 2"`.

- **Restraint over richness**: The plan names "Exactly one horizontal scene, no scrolling, no panning, no zooming, no clutter." It also ties the strict flock ceiling of seven to "preserving individual auditory recognizability."

- **Naturalist voice for the product, matter-of-fact for the system**: Product surfaces use the naturalist register, while "auth, settings, error, and sync conflict surfaces" use "clear, standard matter-of-fact English without faux-warmth." The API section repeats that error responses on system endpoints must be matter-of-fact.

- **Growth without gamification or guilt**: New birds are gated by "calendar tenure, never by click count, visit count, or currency." Personality drift is "monotonic" with "zero decay or punishment on neglect." Non-goals explicitly reject streaks, counters, XP, levels, hunger gauges, death, distress, guilt, and re-engagement notifications.

- **Server-authoritative calm**: The plan repeatedly uses the "Single Writer Principle": only the server simulation tick updates canonical personality vectors and moods, clients submit events only, guests are render-only, and race conditions are avoided through append-only events, a canonical snapshot, and distributed locks.

- **Privacy and accessibility are architectural constraints, not add-ons**: Synthetic UUIDs, encrypted email, blind indexes, isolated telemetry, screen-reader running prose, reduced-motion mode, call captions, keyboard navigation, and WCAG AA focus rings all appear in the core scope, schema, architecture, and risk mitigations.

## Per-feature whys

### Executive Summary & Product Scope

- **Lightweight browser-based aviary**: The plan frames the product as "lightweight" and rendered "directly in a web tab" for modern desktop and mobile browsers.

- **Single horizontal scene**: The rationale is "Restraint over richness": one scene with "no scrolling, no panning, no zooming, no clutter."

- **Small flock starting at 2 and capping at 7**: The plan says the strict ceiling preserves "individual auditory recognizability."

- **Modern browser platform support**: The plan connects the platform to "pure web standards only" and the browser-based product scope.

- **Email magic-link accounts**: NOT RECOVERABLE FROM PLAN

- **15-minute expiration, single-use, rate-limited magic links**: NOT RECOVERABLE FROM PLAN

- **Synthetic UUID internal keys and encrypted email at rest**: The rationale is PII isolation: "Telemetry and foreign keys never touch PII," emails are stored in exactly one encrypted column, and the risk matrix mitigates PII leaks into logs, shard keys, or analytics pipelines.

- **Revocable per-device sessions**: NOT RECOVERABLE FROM PLAN

- **30-day soft deletion with self-recovery, followed by hard purge**: The plan's own account deletion response says the user "can sign in within 30 days to cancel this request."

- **JSON snapshot export**: NOT RECOVERABLE FROM PLAN

- **Single canonical aviary per account**: NOT RECOVERABLE FROM PLAN

- **Aviary age progression instead of click count, visit count, or currency**: The plan says this preserves "the quiet deepening of the user's relationship without introducing gamification" and ensures weekly and daily visitors receive birds on the "exact same schedule."

- **Six initial bird species with distinct silhouettes, palettes, and call-grammar motifs**: The plan ties the species pool to distinct visual and acoustic identity: "distinct visual silhouettes, feather palettes, and call-grammar motif libraries."

- **Stable bird UUID identity**: The rationale stated is persistence "through renaming, sync, and migrations."

- **Hidden five-dimensional personality vector**: NOT RECOVERABLE FROM PLAN

- **Fast-timescale mood system**: The plan uses moods to respond to "local diurnal cycles, weather, recent interactions, and flock dynamics."

- **Monotonic, asymmetric personality drift**: The rationale is that values "drift toward expressive" with "zero decay or punishment on neglect"; the mathematical rule "guarantees the monotonic invariant."

- **Return-Greeting**: The rationale is naturalist arrival recognition "within 1-2 seconds" while honoring "Notice, never announce"; it scales by absence length and bird boldness/mood without textual welcome banners or toasts.

- **Listen-In**: The plan says it "refocuses the acoustic field without unnatural cuts" by raising one bird, dimming others to an "ambient floor," and keeping the chorus "gently audible."

- **Top-bar gestural offers**: NOT RECOVERABLE FROM PLAN

- **Per-bird offer cooldowns**: The plan says cooldowns "prevent mechanic spamming" and "forecloses any possibility of click-spamming while allowing legitimate experimentation."

- **Settle Gesture**: The plan describes it as an "opt-in gentle session-end gesture" with dusk transition and dampened calls. Making tab close identical at the engine level keeps settling optional rather than required.

- **Five-second settle cancel grace window**: The rationale is undoability: "Any click within 5 seconds reverses the lighting shift instantly."

- **Field Notebook**: The rationale is the naturalist surface voice: a "read-only naturalist observation log" that is "sparse, poetic, and persistent."

- **Three-signal presence accounting**: The plan later justifies the 180-second input window as respecting "birdwatching is an idle activity where the user may sit quietly without touching the mouse for a couple of minutes."

- **Host-initiated social invitation**: The rationale is "Social (Quiet & Optional)" and avoiding discovery-feed or network behavior.

- **Read-only ambient guest view**: The plan says guests have "zero co-presence, no shared cursors, no interaction permissions, no visitor drift impact" and later calls them "Render-Only Guests."

- **Host visit log in settings**: NOT RECOVERABLE FROM PLAN

- **Friend-visited notifications default OFF**: The plan's non-goal says Pocket Aviary never reaches out "to pull the user back."

- **Screen-reader naturalist running prose narration**: The rationale is accessibility in the same naturalist voice, with prose narration rather than raw system state.

- **Reduced-motion mode**: The rationale is an accessible designed mode: cross-faded still poses, static camera, and disabled particles/parallax.

- **Real-time procedural call captions**: The plan supports users when audio is silent or unavailable; captions are enabled automatically and anchored near vocalizing birds.

- **Keyboard navigation and visible focus rings**: The rationale is accessibility with focus rings "meeting WCAG AA" and contrast > 7:1 across lighting phases.

- **Initial JS bundle <= 2.0 MB**: The rationale is performance: the edge cache and renderer choices serve the "lightweight" product and avoid missing the bundle budget.

- **Time to first bird visible < 500 ms**: The rationale is the "Already Alive" experience and avoiding a client-side round-trip waterfall.

- **60 fps idle motion, zero memory growth, and WebAudio graceful silence**: The plan ties these to long-running ambient use, older hardware, and avoiding failure when WebAudio is unavailable.

### System Architecture & Topology

- **Edge HTML SSR / snapshot injection**: The plan says inlining the current aviary snapshot "eliminates client-side round-trip waterfall" and satisfies "Time to First Bird (<500ms)."

- **Gateway & Session Service**: The plan says it manages magic-link authentication, secure cookies, synthetic account UUIDs, rate limiting, and session admin while isolating PII.

- **Aviary Ingestion Service**: The rationale is event-only ingestion: it appends interactions to a log and "emits no direct personality mutations."

- **Simulation Worker Service**: The rationale is canonical aliveness: it is the "server tick engine" that advances state, processes event batches, updates weather/mood/drift, and generates observations.

- **PostgreSQL canonical ACID store**: The plan says it is the "Single source of truth" and uses strict isolation "to prevent race conditions."

- **Redis low-latency cache and locks**: The rationale is operational: cached snapshots, distributed lease locks for simulation workers, work queues, and rate limiting.

- **Single Writer Principle**: The rationale is consistency: only the server tick updates canonical vectors and moods.

- **Event-Only Ingestion**: The rationale is to ensure clients have "zero write access to trait values."

- **Render-Only Guests**: The rationale is that visitors pull read-only snapshots and have interaction endpoints "completely disabled."

### Data Model & Database Schema

- **Accounts table with encrypted email and blind index**: The rationale is lookup without exposing raw email, while enforcing that emails live in exactly one encrypted column.

- **Sessions table with revoked_at**: NOT RECOVERABLE FROM PLAN

- **Magic links table with token_hash, expires_at, and used_at**: NOT RECOVERABLE FROM PLAN

- **Aviaries table with lighting, weather, settled state, last tick, and presence counters**: The rationale is to store canonical scene state for rendering and simulation.

- **Birds table with species, name, perch zone, mood, and greeting/call timestamps**: The rationale is persistent bird identity and renderable state for each bird.

- **Personality vectors table as server-only writes**: The rationale is to keep trait updates under the Single Writer Principle and prevent client-side trait mutation.

- **Append-only interaction event log**: The rationale is conflict-free processing and preservation of interaction gestures for the simulation tick.

- **Notebook entries table**: The rationale is persistent field notebook observations with focal bird and weather context.

- **Visit invites and visit logs**: The rationale is private, revocable guest access and host-visible visit duration records.

### API Surface & Contract Specifications

- **Matter-of-fact endpoint errors**: The rationale is the "Naturalist voice for the product, matter-of-fact for the system" principle.

- **`POST /api/v1/auth/magic-link`**: NOT RECOVERABLE FROM PLAN

- **`GET /api/v1/auth/verify`**: The rationale is to consume the "single-use token within its 15-minute validity window" and establish a secure session cookie.

- **`POST /api/v1/account/export`**: NOT RECOVERABLE FROM PLAN

- **`POST /api/v1/account/delete`**: The rationale is a "30-day soft deletion grace period" with cancellation by signing in.

- **`GET /api/v1/aviary/snapshot`**: The rationale is fetching "the current canonical aviary state for rendering and audio scheduling."

- **`POST /api/v1/aviary/events`**: The rationale is batch submission of client interactions and presence heartbeats into the event-only model.

- **`POST /api/v1/visits/invite`**: The rationale is a "private, revocable guest invitation."

- **`POST /api/v1/visits/revoke`**: The rationale is immediate revocation of an outstanding or active visit token.

- **`GET /api/v1/visits/guest-snapshot`**: The rationale is a "Read-only ambient snapshot" that validates tokens, disables presence recording, ignores interaction attempts, and logs visit duration.

### Simulation Engine Design

- **Deterministic 60-second server-side tick**: The rationale is that the canonical state "advances" independently of users and remains server-authoritative.

- **Distributed aviary lock during tick**: The rationale is avoiding concurrent writes while the worker pulls events, evaluates drift/moods, and commits canonical state.

- **Presence and drift filtering**: The rationale is to transform valid presence and interactions into slow, monotonic expressive change.

- **Diurnal cycle and weather transitions**: The rationale is mood and scene variation tied to local time and weather.

- **Observation and narration engine**: The rationale is to compose "naturalist prose" for notebook and screen-reader surfaces.

- **Personality drift low-pass filter**: The rationale is diminishing returns as traits approach maximum expression and a hard guarantee that traits "never decrease on user absence or neglect."

- **Drift calibration targets**: The rationale is to make 7-day changes detectable by regression tests but "imperceptible to casual visual inspection," with 21-day changes substantively shifting behavior.

- **Mood Markov chain**: The rationale is fast mood changes influenced by dusk, dawn, offers, long absence, boldness, and social contagion.

- **Procedural call-grammar runtime**: The rationale is varied acoustic identity through motif types, production rules, micro-timing variance, micro-pitch variance, and personality modulation.

### Multi-Device Sync & Concurrency Model

- **Clients as stateless renderers and event dispatchers**: The rationale is server-authoritative sync from a single canonical state snapshot.

- **No client-to-client peer synchronization**: The plan rejects WebRTC mesh and local CRDTs because multiple sessions should pull one canonical state.

- **Additive interaction events**: The rationale is conflict-free processing; concurrent heartbeats are logged with unique UUIDs.

- **Union of active presence intervals**: The plan says simultaneous presence on two devices counts as exactly one minute, "preventing artificial drift acceleration."

- **No last-write-wins state submission**: The rationale is that clients never transmit absolute state, so "race conditions and last-write-wins data loss are architecturally eliminated."

- **Smooth interpolation on snapshot updates**: The rationale is "avoiding visual snaps" when perch zones or coordinates change.

- **Visibility and sleep resync**: The rationale is recovering from tab visibility changes, OS laptop sleep, or background throttling by pulling a fresh snapshot.

### Frontend Rendering & Micro-Motion Pipeline

- **Standard 60 fps Canvas2D/WebGL render loop**: The rationale is idle motion in the single horizontal scene with layers for sky, canopy, perch zones, particles, micro-motion, and focus highlights.

- **Reduced-motion discrete cross-fade engine**: The rationale is to disable continuous frame loops, ambient particles, and parallax while still showing state transitions through smooth alpha cross-fades.

- **Responsive virtual coordinate space**: The rationale is preserving vertical scale while compressing horizontal perch spacing on narrow viewports.

- **Visible viewport invariant for birds**: The rationale is that "no bird is ever cropped or placed offscreen."

- **Already Alive first frame**: The rationale is to show the aviary "has been continuing without the viewer" by placing birds mid-action on the first frame.

- **No loading spinners or fade-from-blank transitions**: The rationale is the same already-alive mandate; a cold fetch shows calm sky and faint mist until birds mount.

### Audio Pipeline & Procedural Call Synthesis

- **Per-bird procedural synthesizer graph**: The rationale is species- and bird-specific calls generated from oscillators, modulation, filtered noise, envelopes, panning, and gains.

- **Master dynamics compressor**: The rationale is preventing chorus peaking.

- **Listen-In gain ramps**: The rationale is smooth audio rebalance: focus bird rises over 1500ms, others drop to an ambient floor, and disengagement ramps everyone back over 2000ms.

- **Autoplay-resilient suspended AudioContext**: The rationale is browser autoplay handling; a subtle affordance or first interaction resumes audio.

- **No pre-recorded fallback audio**: The rationale is the explicit mandate that WebAudio failure should produce "graceful silence" with captions rather than fallback clips.

### Accessibility Surfaces

- **ARIA live narration region**: The rationale is polite, atomic screen-reader narration in the naturalist register.

- **Semantic focus anchors over canvas birds**: The rationale is making canvas birds keyboard-focusable and screen-reader addressable.

- **Canvas marked aria-hidden**: The rationale is separating the visual layer from the accessible DOM surfaces.

- **Visual call captions overlay**: The rationale is high-contrast callouts synchronized with procedural calls and useful when audio is silent or unavailable.

- **Narration priority bumps**: The rationale is immediate feedback for discrete user gestures while keeping the ordinary cadence slow.

- **Hard rule against raw system state**: The rationale is avoiding robotic attribute lists and preserving naturalist prose.

- **Keyboard bindings for listen-in, offers, and settle**: The rationale is full keyboard navigation across top-bar controls and bird targets.

- **Double-outline focus indicator**: The rationale is contrast > 7:1 across all lighting phases.

### Performance Budgets, Verification & Telemetry

- **Bundle analyzer CI enforcement**: The rationale is making the 2.0 MB gzipped bundle budget a PR build failure.

- **Synthetic Lighthouse TTFBird release block**: The rationale is preventing regressions to first bird visibility on 4G mobile.

- **Puppeteer frame trace benchmark**: The rationale is verifying 60 fps idle motion on older hardware.

- **Thirty-minute heap snapshot diffing**: The rationale is enforcing "0.00 MB net growth" for an ambient app that may stay open.

- **Simulation tick latency alerting**: The rationale is keeping p99 worker batches under 5 seconds.

- **WebAudio node reuse, canvas zero allocations, and DOM stability**: The rationale is avoiding allocation churn and continuous node creation/removal.

- **Privacy-first telemetry gateway**: The rationale is aggregating operational metrics while dropping account IDs, bird IDs, names, vectors, presence timestamps, and interaction logs.

### Rollout & Phased Deployment Strategy

- **Phase 0 synthetic audit**: The rationale is to verify schema, simulation, audio, renderer, performance budgets, leak tests, and drift mathematics before dogfooding.

- **Phase 1 closed alpha dogfooding**: The rationale is to calibrate audio aesthetics "to prevent auditory fatigue" and audit screen-reader narration and reduced-motion with the accessibility team.

- **Phase 2 limited canary beta**: The rationale is to verify multi-device sync across desktop/mobile and absence/return mechanics after 7-day and 14-day inactivity.

- **Phase 3 general availability**: The rationale is public signup only after account starts and age progression timers are ready.

- **Flock ramp schedule**: The rationale is "quiet deepening" without gamification, with bird arrivals based strictly on calendar tenure.

### Risk Matrix & Technical Mitigations

- **Configurable drift learning rate**: The rationale is mitigating drift that feels too fast like "Tamagotchi stat grinding" or too slow like a "static screensaver."

- **Event-only append log and single-threaded tick per aviary**: The rationale is avoiding dropped presence and jerky repositioning in concurrent multi-device sessions.

- **Subtle synthesis variation and compressor**: The rationale is avoiding robotic, grating, or chiptune-like procedural audio over extended listening.

- **Captions during autoplay blocks**: The rationale is preserving the experience while audio is muted or pending.

- **Centralized prose generation and rate limits**: The rationale is avoiding spammy narration or robotic attribute lists.

- **Synthetic UUIDs, encrypted email, blind index, and telemetry schema rejection**: The rationale is avoiding PII or compliance leaks into logs, shard keys, or analytics.

### Defensible Implementation Decisions

- **Bespoke lightweight 2D WebGL/Canvas2D renderer**: The rationale is keeping the initial JS bundle comfortably under the 2.0 MB budget rather than using heavier game engines.

- **180-second activity window**: The rationale is that "birdwatching is an idle activity where the user may sit quietly without touching the mouse for a couple of minutes."

- **Absence thresholds for greetings**: The rationale is scaling short absence to "a subtle glance or soft single note" and long absence to "noticeable re-orientation and front-perch approach."

- **240-second offer cooldown**: The rationale is to prevent click-spamming while preserving legitimate experimentation.

- **5-second settle undo grace**: The rationale is instant reversal of the lighting shift if the user clicks within the grace window.
