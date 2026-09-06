## System-level intent

1. **A quiet, non-gamified observational relationship.** This shows up in the hard boundary against "Gamification / Engagement Mechanics," the ban on "streak counters," "level-up confetti," "achievements," and "progress bars," and the bird-count ramp rationale that "Rejects gamified task-completion unlocks; honors the slow deepening of an observational relationship."

2. **Ambient aliveness instead of static app state.** The plan names "First-frame aliveness," "rendering birds mid-action," "ambient motion and sound already active," "procedural micro-motion," "rare subtle ambient weather," and "continuous day/night lighting cycle." The frontend section reinforces this with "No static start poses."

3. **Server-authored continuity and consistency.** The plan separates "a deterministic, server-side simulation and persistence engine" from the browser client. It repeatedly says the "server" is the "Sole Author," there is "zero client-side simulation," "No Last-Write-Wins," and one "canonical aviary per account."

4. **Presence must mean actual attention.** Presence tracking is defined as a "Strict conjunction" of visibility, focus, and recent input. The risk table explains the why: avoid "Ghost Presence Leaks" where a "background tab counts presence" and "birds drift while user is away."

5. **No punishment, distress, or custodial pressure.** The non-goals state that birds "never die, starve, fall ill, or show distress" and that absence produces "ambient quietness, never negative trait drift." The drift formula enforces this through "Monotonic constraint" and "traits never decrease."

6. **Privacy and isolation as architectural boundaries.** The schema section says PostgreSQL "ensures strict entity isolation using synthetic UUIDs"; non-goals ban "PII Leakage / Data Monetization"; observability keeps a "Strict boundary between operational metrics and bird simulation data" and prohibits "User IDs tied to bird names," "per-bird personality vectors," and "visitor graphs."

7. **Naturalist voice over app-interface voice.** The API surface says errors are "matter-of-fact" while domain payloads are structural; accessibility narration uses "observational prose," "matching naturalist voice," and "Strictly avoids announcement style." The notebook uses "naturalist grammar templates" with "present-tense, lowercase observations."

8. **Accessibility is part of the aesthetic experience.** The plan says Pocket Aviary treats accessibility as "a primary aesthetic surface rather than an compliance checklist." This appears in naturalist screen-reader narration, procedural call captions, reduced-motion alternatives, keyboard navigation, focus rings, and release-blocking screen-reader testing.

9. **Lightweight procedural craft within hard performance budgets.** The plan emphasizes a "<2MB" bundle, "time-to-first-bird <500ms," "60fps," "zero heavy 3D engines," "zero audio asset bloat," and "100% procedural WebAudio synthesis" to preserve responsiveness and avoid "canned audio artifacts."

10. **Social is quiet, read-only, and host-controlled.** The social section names "Quiet Visits," "Pure read-only ambient viewing," "no visitor interactions," "zero co-presence," "no chat/comments/avatars," "instant host revocation," and "visit notifications off by default."

## Per-feature whys

### 1. Scope & Boundary Enforcement

- **Modern web browsers**: NOT RECOVERABLE FROM PLAN
- **Single horizontal aviary scene fitting viewport without scrolling, panning, or zooming**: The plan ties this to a contained core experience where the whole aviary remains visible; the rendering section later requires "100% bird visibility."
- **Three depth perch zones: Front, Middle, Back**: NOT RECOVERABLE FROM PLAN
- **Two starter birds at onboarding chosen automatically from a 6-species pool**: NOT RECOVERABLE FROM PLAN
- **User naming during adoption and subsequent renaming**: NOT RECOVERABLE FROM PLAN
- **Age-paced unlocking of additional bird adoption slots up to seven birds**: The rollout section says this is "To preserve recognizability and emotional connection" and "honors the slow deepening of an observational relationship."
- **Continuous day/night lighting cycle tied to user local timezone**: NOT RECOVERABLE FROM PLAN
- **Rare subtle ambient weather and micro-motion**: The plan uses these to create "ambient" presence and "first-frame aliveness" without heavier mechanics.
- **Procedural micro-motion for birds reflecting mood and personality**: The plan connects motion to mood/personality so birds feel alive and individually expressive rather than static.
- **First-frame aliveness**: The plan's rationale is explicit: birds start "mid-action" with motion and sound active, and the animation section says "No static start poses."
- **Quiet sky field for initial network fetch**: NOT RECOVERABLE FROM PLAN
- **Presence Tracking**: It prevents "background tab" presence and calibration damage; presence must be visible, focused, and recently active.
- **Return-Greeting**: The plan grounds this in absence duration, boldness, and mood while preserving quietness through "zero welcome toasts or banners."
- **Listen-In**: The rationale is audio focus: raise the selected bird while "ducking other birds to ambient floor without complete muting."
- **Offers: seed, song-fragment, still water pool**: The plan says reactions depend on "mood/curiosity" and use per-bird cooldowns.
- **Settle Gesture**: The plan frames it as an "opt-in gentle evening fade and call dampening" with undo, and "equivalent to tab close at the engine level."
- **Field Notebook**: The plan calls for "sparsely generated naturalist prose observations" in a read-only modal/panel; the sparsity supports the observational, non-spammy voice.
- **Email-based single-use invitation links**: The single-use and expiration rules support host-controlled, bounded "Quiet Visits."
- **Read-only ambient viewing of host's canonical aviary**: The why is to keep visits quiet: "no visitor interactions," "no visitor presence counted," "zero co-presence," and no social surfaces.
- **Host visit log in settings**: The plan gives host visibility into visits, while keeping "visit notifications off by default."
- **Magic link authentication**: NOT RECOVERABLE FROM PLAN
- **Single canonical aviary per account**: The plan ties this to "server-side authoritative simulation tick" and multi-device consistency.
- **Server-authored additive deltas**: The rationale is to avoid "client-side simulation" and "last-write-wins collisions on personality state."
- **Account data JSON export**: NOT RECOVERABLE FROM PLAN
- **30-day soft deletion lifecycle**: NOT RECOVERABLE FROM PLAN
- **Screen-reader naturalist running prose narration**: The plan treats accessibility as aesthetic, using naturalist prose instead of announcement style.
- **Procedural call captions**: Captions match the "procedural motifs" and provide a fallback when audio is unavailable.
- **Reduced-motion mode**: It replaces continuous animations and removes particles for users who request reduced motion while preserving slow lighting changes.
- **Full keyboard accessibility and WCAG AA contrast compliance**: The plan grounds this in keyboard operation, high-contrast focus, focus-trapping, and release quality.
- **Initial JS bundle <2MB**: The plan connects this to lightweight rendering, no heavy engines, no audio sample bloat, and dynamic imports.
- **Time-to-first-bird <500ms**: The plan connects this to first-frame experience and fast initial canvas render.
- **60fps steady rendering**: The plan grounds this in long idle sessions on "5-year-old baseline laptop hardware."
- **Zero memory growth over 30 minutes**: The plan ties this to avoiding tab crashes during long open sessions.
- **No native mobile apps**: NOT RECOVERABLE FROM PLAN
- **No gamification / engagement mechanics**: The plan rejects mechanics that would turn the experience into streaks, levels, badges, or progress tracking.
- **No custodial / Tamagotchi mechanics**: The why is to prevent birds from showing distress and keep absence from creating negative trait drift.
- **No social network surfaces**: The plan keeps the product away from public directories, profiles, following, feeds, leaderboards, and showcases.
- **No PII leakage / data monetization**: The why is isolation and privacy: synthetic UUIDs, no data lakes, no third-party sharing, and no model training.

### 2. Architecture & Service Topology

- **Deterministic server-side simulation and persistence engine**: The rationale is authoritative continuity and a single source of truth for simulation.
- **Low-latency WebGL/WebAudio client**: The plan connects this to responsive rendering and audio in the browser.
- **API Gateway & Auth Service**: The plan says it handles magic link dispatch, token verification, route access control, and visitor token validation.
- **Event Ingest Service**: Its why is to receive client events and write append-only records for the simulation.
- **Simulation Tick Worker**: Its why is to process active and dormant aviaries, advance mood, compute deltas, handle adoption eligibility, generate notebook entries, and save canonical snapshots.
- **Snapshot & State Distribution Service**: Its why is to deliver lightweight JSON state on load, visibility restoration, or stream updates.
- **Scene Renderer**: Its why is responsive horizontal scenery, perch zones, lighting, and ambient particles.
- **Audio Synthesizer**: Its why is procedural calls and sound beds with ducking and fallback handling.
- **Accessibility & Narration Controller**: Its why is ARIA live descriptions and synchronized call captions.

### 3. Data Model & Storage Schema

- **Synthetic UUIDs and monotonic numerical limits**: The plan says these ensure "strict entity isolation."
- **Accounts table with encrypted email and email hash**: The comment says email is encrypted and the hash allows lookup "without decrypting."
- **Aviaries table with one canonical aviary per account**: The comment gives the why: "One canonical aviary per account."
- **Birds table with stable identity, personality vector, mood, visual and spatial state**: The table comment says it stores "Stable bird identity, personality vector, and current mood."
- **Hidden personality vector never exposed to user**: The plan keeps personality hidden and later prohibits telemetry of "per-bird personality vectors," supporting strict entity isolation and privacy.
- **Slot index between 0 and 6 with unique aviary slot**: The why is enforcing the strict cap of seven birds.
- **Append-only interaction event log**: The plan says it is "consumed by simulation tick," supporting server-authored additive computation.
- **Field notebook entries as immutable historical records**: The generation section says entries are saved as "immutable historical records."
- **Visit invitations with token hash, expiration, and revocation**: The why is read-only guest access with 30-day expiration and instant host revocation.
- **Visit log entries**: The plan uses them for an "on-demand host visit log" while keeping visit notifications off by default.

### 4. API Surface & Protocols

- **Errors return matter-of-fact text and domain payloads carry structural data**: The rationale is adherence to the defined product voice.
- **Magic-link request endpoint with rate limit**: The endpoint issues tokens by email and rate-limits requests to 5 per hour per email.
- **Magic-link verify endpoint with secure session cookie and immediate invalidation**: The plan grounds this in 15-minute, single-use authentication.
- **Logout endpoint**: NOT RECOVERABLE FROM PLAN
- **Current canonical aviary snapshot endpoint**: The why is fetching the current server-authored aviary state.
- **SSE event stream**: The why is broadcasting simulation tick updates and real-time settle state.
- **Client interaction event endpoint**: The why is collecting presence, listen-in, offer, and settle events for the append-only log.
- **Settle and unsettle endpoints**: The why is a 5-second undo window for the settle gesture.
- **Adopt endpoint**: The why is adopting only when the age threshold is reached.
- **Notebook endpoint**: The why is returning read-only observations.
- **Social invitation endpoint**: The why is generating a visitor invite for a friend email.
- **Social logs endpoint**: The why is retrieving recent visit logs.
- **Invitation revocation endpoint**: The why is immediate host revocation.
- **Account export endpoint**: NOT RECOVERABLE FROM PLAN
- **Account delete endpoint**: NOT RECOVERABLE FROM PLAN
- **Visitor token endpoint rejecting interaction posts**: The why is read-only visiting, enforced with "This visit is read-only."

### 5. Simulation Engine Design

- **Authoritative server-side cron/worker tick every 60 seconds**: The rationale is server-authored simulation and consistent canonical snapshots.
- **Presence Calculation Pipeline**: The why is validated attention: visibility, focus, and recent input must all hold before presence pings count.
- **Heartbeat every 30 seconds while present**: The plan uses heartbeats to aggregate non-overlapping validated presence intervals.
- **Halting transmission immediately when presence fails**: The why is preventing ghost presence.
- **Drift Function Calibration**: The why is transforming cumulative presence and interactions into personality vector adjustments.
- **Low-pass filter and diminishing returns**: The plan says diminishing returns apply as traits approach ceiling, smoothing drift.
- **Monotonic drift**: The why is that "Neglect" causes zero change and traits "never decrease."
- **Presence-time as primary input**: The plan uses validated presence as the primary signal for slow personality drift.
- **Listen-in trait effects**: The plan says focused listening increases `social_warmth` and `vocal_frequency`.
- **Offer trait effects**: The plan says accepted offers increase `curiosity` and `boldness`.
- **7-day instrumental drift target**: The why is automated assertions can detect it while it remains "imperceptible to casual eye."
- **21-day visible drift target**: The why is longer-term visible change in perch preference, plumage saturation, and call cadence.
- **Mood Transition FSM**: The why is fast-timescale mood update on each tick.
- **Diurnal clock**: The plan ties mood to local time and settled/asleep behavior.
- **Recent session interactions influencing mood**: The why is accepted offers and listen-in nudge birds toward curious/content states.
- **Ambient events influencing mood**: The plan says rain damps calls and nudges mood toward content or drowsy.
- **Social contagion**: NOT RECOVERABLE FROM PLAN
- **Absence persistence**: The why is reopening reveals "current real-time mood without artificial resets."
- **Procedural call grammar runtime**: The why is to compose species calls from motifs with stochastic variation.
- **Mood-based call syntax**: The rationale is that alert, content, curious, and drowsy calls sound different according to mood.
- **Chorus generator**: The why is responsive answering calls based on `social_warmth`, `vocal_frequency`, mood, and humanized stagger.
- **Field Notebook Generation Engine**: The why is sparse naturalist observations based on aviary history.
- **Sparsity enforcement**: The plan says entries occur at most every 3-5 days unless an inflection event occurs.
- **Naturalist grammar templates**: The why is present-tense, lowercase observations in the product voice.

### 6. Multi-Device Synchronization & Conflict Prevention

- **Server as Sole Author**: The why is that the client never mutates state, only event logs.
- **No Last-Write-Wins**: The why is preventing conflicting clients from overwriting personality traits and mood.
- **Append-only event aggregation across simultaneous clients**: The why is chronological additive deltas when laptop and phone are both open.
- **Snapshot interpolation**: The why is perch transitions glide smoothly "rather than snapping."
- **Server-marked settle state**: The why is shared settle state across connected clients.
- **SSE or poll update for settle fade**: The why is connected clients initiate the warm sunset fade together.
- **Tab restoration state fetch**: The why is re-anchoring positions smoothly after backgrounding or wake.

### 7. Frontend Rendering Pipeline

- **Single lightweight canvas**: The plan connects this to fitting the whole horizontal scene and the performance budget.
- **Sky layer with local solar elevation**: The why is reflecting dawn, midday, sunset, and dusk lighting.
- **Background soft-focus tree canopies with parallax**: NOT RECOVERABLE FROM PLAN
- **Back, middle, and front perch zones**: The layers place resting, primary, and bold/offer-related birds at different depths.
- **Particle layer**: The why is ambient leaves/feathers drifting at randomized intervals.
- **Fixed 16:9 bounding anchor and CSS contain**: The why is dynamic auto-scaling and no horizontal scrollbars.
- **Mobile perch spacing contraction**: The why is "100% bird visibility."
- **Procedural SVG/2D-path birds with no video/GIF loops**: The rationale is avoiding pre-rendered loops and heavy assets.
- **Breathing motion**: The why is subtle natural idle movement.
- **Head cock motion**: The why is glance behavior toward viewer or other birds.
- **Tail bob motion**: NOT RECOVERABLE FROM PLAN
- **Preening motion**: NOT RECOVERABLE FROM PLAN
- **Procedural phase timers initialized from `Date.now()`**: The why is first-frame aliveness.
- **Reduced-motion disabling frame-by-frame movement**: The why is honoring reduced-motion preference.
- **Static composed naturalist poses in reduced motion**: The why is replacing continuous animation while preserving the bird scene.
- **Cross-fade perch transitions in reduced motion**: The why is replacing flight curves.
- **Culling ambient particles in reduced motion**: The why is removing drifting leaf and feather particles.
- **Slowed diurnal lighting cross-fades in reduced motion**: The why is retaining lighting transitions gently.

### 8. Audio Pipeline & Synthesis Architecture

- **Native WebAudio procedural synthesis without sample files**: The plan says this preserves the bundle budget and avoids asset bloat.
- **Species-tailored synthesis graph**: The why is producing species-specific warbles, trills, and organic resonance.
- **Dual oscillators and pitch envelopes**: NOT RECOVERABLE FROM PLAN
- **FM synthesis for warbles or trills**: The why is species with warbles or trills.
- **Low-pass filter**: The why is "softening harsh digital harmonics" and "organic resonance."
- **Amplitude envelope**: NOT RECOVERABLE FROM PLAN
- **Default spatial panning and calm ambient level**: The plan grounds this in natural perch coordinates and calm ambient mix.
- **Listen-in gain ramp for selected bird**: The why is audio focus on the chosen bird.
- **Ducking other bird channels without muting**: The why is keeping other birds as a quiet background bed.
- **High-frequency ambient rolloff during listen-in**: NOT RECOVERABLE FROM PLAN
- **Smooth disengagement interpolation**: The why is returning to equal ambient levels.
- **Graceful silent mode when AudioContext fails**: The why is fallback for unsupported browser, autoplay restriction, or hardware failure.
- **Captions enabled by default in silent mode**: The why is preserving call information when audio is unavailable.
- **No recorded audio fallbacks**: The rationale is preserving the "<2MB bundle budget" and preventing "canned audio artifacts."

### 9. Accessibility Surfaces

- **ARIA live region for observational prose updates**: The why is naturalist screen-reader narration as an aesthetic surface.
- **30-60 second idle narration cadence**: The plan says this keeps narration gentle.
- **Prioritized user-triggered narration updates**: The why is to describe important user events promptly.
- **Avoiding announcement style**: The why is preserving naturalist prose instead of robotic app-state announcements.
- **Floating call captions adjacent to vocalizing bird**: The why is unobtrusive, spatially connected captions.
- **Caption strings generated from call grammar runtime**: The why is matching captions to procedural motifs.
- **Caption fade timing**: NOT RECOVERABLE FROM PLAN
- **Keyboard traversal through top-bar icons and birds**: The why is full keyboard accessibility.
- **Arrow navigation among birds by spatial perch position**: The why is keyboard navigation that follows the scene layout.
- **Enter/Space listen-in and Escape disengage**: The why is keyboard control of listen-in.
- **Silhouette-conforming high-contrast focus indicator**: The why is visible focus across daylight and night backgrounds.
- **Modal focus trapping and focus return**: The why is accessible dialog behavior.

### 10. Performance Budgets & Observability

- **CI bundle analyzer enforcing <2MB**: The why is hard bundle-budget enforcement.
- **TTFBird measurement from navigation start to canvas render**: The why is enforcing time-to-first-bird.
- **30-minute framerate measurement on 5-year-old hardware**: The why is steady long-session rendering.
- **Heap snapshot diffing after 30-minute sessions**: The why is catching memory leaks.
- **Simulation tick p99 latency budget**: The why is worker batch processing deadline.
- **No three.js or Babylon.js**: The plan says this supports a custom lightweight 2D canvas renderer.
- **100% procedural WebAudio synthesis**: The why is saving "dozens of megabytes of audio samples."
- **Dynamic imports for settings, visits, and export dialogs**: The why is splitting non-critical chunks.
- **Permitted operational telemetry**: The why is measuring request counts, latency, FPS, audio errors, and tick duration without simulation data.
- **Prohibited telemetry**: The why is preserving the boundary around bird names, vectors, interaction sequences, and visitor graphs.
- **Payload sanitization before logging**: The why is privacy-preserving observability.

### 11. Rollout & Aviary Scaling Strategy

- **Internal dogfooding with two birds**: The why is to verify drift calibration over 3 weeks and calibrate presence detection.
- **Closed alpha**: NOT RECOVERABLE FROM PLAN
- **Beta with social visits**: The why is introducing opt-in visitor links and validating multi-device sync across cross-device sessions.
- **General availability with species unlocks**: NOT RECOVERABLE FROM PLAN
- **Bird count ramp-up by account age**: The why is to "preserve recognizability and emotional connection."
- **Permanent cap of seven birds**: The plan ties this to age-paced progression, recognizability, and emotional connection.

### 12. Risk Management & Failure Modes

- **Drift calibration too fast mitigation**: The why is avoiding traits that visibly shift session-to-session and feel like a "Tamagotchi widget."
- **Ghost presence leak mitigation**: The why is preventing background tabs from counting presence and ruining calibration.
- **Multi-device divergence mitigation**: The why is preserving personality history against conflicting phone/laptop writes.
- **Procedural audio uncanniness mitigation**: The why is preventing synthesized calls from sounding "harsh, robotic, or grating over long sessions."
- **Accessibility degradation mitigation**: The why is keeping narration from becoming "spammy or robotic" and ruining the affective experience.
- **Memory leak mitigation**: The why is avoiding tab crashes during long open sessions.
