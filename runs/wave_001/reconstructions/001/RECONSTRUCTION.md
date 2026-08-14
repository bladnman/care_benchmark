## System-level intent

- **A low-key, companionable relationship, not a game or chore loop.** This is stated in the executive summary as "a low-key, companionable relationship rather than a gamified or custodial experience." It reappears in the strict guardrails: "No Gamification," "No Tamagotchi Mechanics," "No Push/Notification Engagement Loops," and "No In-Aviary UI Chrome or Text Banners."
- **Quiet ambient aliveness over days and weeks.** The plan repeatedly uses "quiet," "ambient," "subtle interactions," "open window on a living habitat," and "expressive aliveness." It shows up in the hidden personality drift, mood transitions, idle micro-motion, procedural calls, sparse notebook, and quiet settle behavior.
- **Server-owned continuity and canonical state.** The architecture says "the server owns the single source of truth" and "advances the aviary state via an asynchronous simulation loop." The product scope also requires "exactly one aviary per account" rendered identically across browsers, and the simulation tick guarantees "continuity regardless of client connectivity."
- **Additive events instead of state overwrites.** The multi-device model rejects "Last-Write-Wins" by saying clients "never author or submit state values" and submit only "append-only semantic interaction events." The simulation engine is the "sole writer" of bird state.
- **Privacy and PII separation as a core product boundary.** The plan names "strict privacy requirements," "PII boundary enforcement," synthetic UUIDs, an isolated encrypted `accounts_auth` table, scrubbed logs, and telemetry stores that exclude email, IP addresses, location, and per-account behavior.
- **Two voices with strict boundaries.** The "Naturalist Voice" is lowercase, present-tense, specific, observational, and free of gamification or second-person commands. The "Matter-of-Fact Voice" is direct, neutral, and reserved for authentication, errors, unsupported browser warnings, invite revocation, and settings.
- **Accessibility as full parity of charm.** The accessibility section says it is "an intentionally designed surface providing full parity of charm." Reduced motion is not just static fallback; the risk matrix says it must remain a "first-class pose cross-fading surface."
- **Performance preserves immediacy and the living illusion.** Hard budgets such as "<500 ms" Time-to-First-Bird, "60 fps," "zero memory growth," and bundle limits are tied to the birds appearing immediately present, mid-breath or mid-preen.
- **Absence must be non-punitive.** The plan says absence produces "ambient quietness without negative trait degradation," drift deltas become zero, traits "never decrease," and ignored birds "do not mistrust or regress."
- **Sharing remains private, ambient, and non-social-networked.** Social visits are "private, opt-in, email-invited, read-only ambient visits" with "zero co-presence" and "zero visitor drift influence," while the guardrails reject profiles, follows, feeds, comments, shared cursors, and multiplayer overlays.

## Per-feature whys

### Executive Summary & Product Scope

- **Single-User Accounts via email magic links:** NOT RECOVERABLE FROM PLAN
- **Canonical Multi-Device Aviary:** The why is canonical continuity: one aviary per account is "simulated continuously server-side and rendered identically across desktop and mobile browsers."
- **Two starter birds:** NOT RECOVERABLE FROM PLAN
- **Age-based population expansion up to 7 birds:** The plan ties additions "exclusively to aviary calendar age," not "visit streaks, click counts, or interaction points," preserving the anti-gamification boundary. The cap of 7 birds exists "to guarantee procedural audio recognizability."
- **Custom bird names and default suggestions:** NOT RECOVERABLE FROM PLAN
- **Persistent internal UUIDs for birds:** NOT RECOVERABLE FROM PLAN
- **Hidden 5-dimensional personality vector with monotonic drift:** The why is slow "expressive aliveness" without punishment. The plan says traits "never decrease," absence gives zero delta, and birds do not "mistrust or regress."
- **Fast-timescale 5-state mood transitions:** The why is dynamic visible behavior: moods respond to time of day, weather, social contagion, and personality bias, giving faster changes alongside slow drift.
- **Procedural idle micro-motion:** The plan wants the visual presentation to feel "like an open window on a living habitat"; breathing, head saccades, preening, and weight shifts create immediate aliveness.
- **Procedural call generation via WebAudio:** The why is aliveness without repetition or canned files. The risk matrix says audio fatigue and mechanical repetition would make the "core aliveness" fail.
- **Return-greeting:** The plan grounds this in the "companionable relationship" and treats return-greeting as a priority event for immediate naturalist narration.
- **Listen-In:** The why is focused attention without breaking ambient presence: the focused bird rises in the mix while other birds remain audible as "soft ambient accompaniment" and are "never completely muted."
- **Offers: seed, song fragment, still pool:** The plan gives these interaction deltas into boldness, curiosity, vocal frequency, and social warmth, so offers are a subtle way to shape monotonic drift.
- **Per-bird offer cooldowns:** NOT RECOVERABLE FROM PLAN
- **Settle gesture:** The why is evening quiet: it shifts lighting, quiets audio, and the mood machine says a settled aviary forces birds toward `drowsy` within 2 ticks.
- **Settle gesture 5-second cancel window:** NOT RECOVERABLE FROM PLAN
- **Presence accounting with strict 3-factor conjunction:** The why is preventing "Ghost Presence Inflation"; background tabs or forgotten windows must not cause birds to drift "without human presence."
- **Field Notebook:** The why is sparse, read-only naturalist observation. The generator preserves "sparsity" and produces lowercase, present-tense observations rather than gamified milestone text.
- **Social Visits:** The why is private ambient sharing without social mechanics: visits are opt-in, read-only, zero co-presence, and have zero drift influence.
- **30-day visit token lifetime:** NOT RECOVERABLE FROM PLAN
- **Silent visit logging:** The plan pairs logging with no default "friend visited" notifications, supporting the no-notification and no-social-network boundary.
- **Accessibility Surfaces:** The why is "full parity of charm" through narration, captions, keyboard access, high contrast, and reduced-motion pose cross-fades.
- **Performance and Platform:** The why is a lightweight, immediate web experience: a PWA-ready bundle under 2MB, "<500ms" Time-to-First-Bird, 60fps, and zero memory growth.

### Architecture & Service Topology

- **Snapshot-and-event-stream architecture:** The why is server-owned truth with thin clients: the server advances state while clients render from snapshots.
- **Edge / CDN layer with injected bootstrap payload:** The plan says this exists to "guarantee `<500ms` Time-to-First-Bird."
- **API Gateway & Auth Service:** The why is magic-link handling, rate limiting, route guarding, and "PII boundary enforcement."
- **Event Ingestion Pipeline:** The why is an "immutable, ordered event log" for interaction events that the simulation can process transactionally.
- **Simulation Service / Tick Engine:** The why is continuous simulation across active and idle aviaries, low-pass personality drift, mood state, environment, and notebook generation.
- **Real-Time Gateway:** The why is live snapshot broadcast, presence disconnection handling, and immediate termination of revoked visits.
- **Client Application:** The why is to keep the browser a procedural renderer and interaction surface: it renders the 2D scene, synthesizes audio, manages focus, and updates live regions from server state.
- **Architectural Privacy & PII Boundary:** The why is to "honor the strict privacy requirements" by isolating email, using synthetic UUIDs, and scrubbing logs, queues, caches, traces, and telemetry.

### Data Model & Database Schemas

- **`accounts_auth` with encrypted email and email hash:** The plan says the email address is stored exclusively in an isolated encrypted table, and the hash is for lookup "without decryption."
- **`auth_magic_links`:** The table supports single-use expiring token validation for magic-link auth; a deeper product why is NOT RECOVERABLE FROM PLAN.
- **`device_sessions` with revocation:** The table supports active sessions and explicit session revocation in account management.
- **`aviaries` with timezone, settled state, last tick, and weather:** These fields support local solar time, settle behavior, continuous ticking, and weather-dependent mood/rendering.
- **`birds` with species, custom name, perch zone, and slot:** Species drives motifs and visuals; perch zone and slot support the layered scene and avoid overlapping occupied slots.
- **`personality_vectors`:** The bounded floats implement the hidden monotonic drift model for boldness, social warmth, vocal frequency, plumage saturation, and curiosity.
- **`bird_moods`:** The fields support the 5-state mood machine, mood expiry, call timing, and last interaction.
- **`interaction_events`:** The why is append-only interaction ingest; unprocessed events are indexed for the next simulation tick.
- **`notebook_entries`:** The why is storing sparse naturalist observations with involved birds and observation type.
- **`visit_invitations`:** The why is private, email-invited visits with encrypted recipient email, token expiry, and instant revocation.
- **`visit_logs`:** The why is silent visit logging while exposing only a masked visitor identifier.

### API Surface & Communication Protocols

- **Matter-of-fact API error format:** The why is the "Matter-of-Fact Voice" for system errors: clear, direct, neutral, and not faux-warm.
- **Auth magic-link request, verify, and revoke endpoints:** They implement the account authentication flow; a deeper why for choosing magic links is NOT RECOVERABLE FROM PLAN.
- **Account export archive via signed download link:** NOT RECOVERABLE FROM PLAN
- **Account delete and restore with 30-day soft deletion:** NOT RECOVERABLE FROM PLAN
- **Aviary bootstrap endpoint:** The why is cold-start speed; it is injected into initial HTML to satisfy the "<500ms" First-Bird requirement.
- **Aviary event submission endpoint:** The why is accepting interaction events into the append-only log and reporting accepted count.
- **Real-time WebSocket `SNAPSHOT_UPDATE`:** The why is to push current canonical state, lighting, weather, bird moods, narration prose, and notebook updates every tick or after significant interaction.
- **Visit invite and immediate revoke endpoints:** The why is private opt-in visiting plus instant revocation.
- **Read-only visit stream and termination card:** The why is that visitors receive only read-only scene parameters, and revoked or expired invites transition to a clean matter-of-fact termination surface.

### Simulation Engine Design & Drift Calibration

- **60s simulation tick loop:** The why is continuity: it ingests events, computes presence, advances drift and mood, generates observations, persists snapshots, and broadcasts updates regardless of client connectivity.
- **Presence time requiring visible, focused, recent input:** The why is that if any condition fails, presence delta is zero, preventing ghost presence.
- **Monotonic personality drift formula:** The why is non-negative, low-pass growth toward aliveness; traits never decrease and ignored birds become ambient and quiet.
- **Calibration targets at 1 week and 3 weeks:** The why is both "Instrument Measurability" for telemetry/test assertions and "Felt Human Perception" for noticeable behavior after 3 weeks.
- **Interaction deltas for listen-in and offers:** The why is to map subtle interactions to warmth, vocal frequency, boldness, and curiosity without visible scores.
- **Diurnal mood modulation:** The why is time-sensitive behavior: morning increases alertness, dusk and night weight drowsiness, and settled aviaries become drowsy.
- **Weather mood modulation:** The why is environmental dynamics: passing rain dampens calls and soft wind increases alertness.
- **Social contagion:** The plan names adjacent-bird wary-state influence as "Social Contagion," supporting group mood dynamics.
- **Personality bias in mood transitions:** The why is that high boldness changes behavior by reducing wary probability and boosting curious probability.
- **Notebook heuristic triggers:** The why is to detect meaningful observations such as greeter shifts, perch breakthroughs, weather harmony, and extended stillness while remaining sparse.
- **Template-free naturalist prose synthesizer:** The why is to follow strict grammar rules: lowercase, present-tense, bird-named, and no gamification phrasing.

### Multi-Device Sync & Conflict Model

- **Single-writer additive event architecture:** The why is multi-device consistency.
- **Elimination of Last-Write-Wins:** The why is to prevent drift overwrites and lost session history; clients submit events, not state values.
- **Simultaneous sessions contributing additive records:** The why is that phone and laptop both feed the shared queue and receive the same resulting snapshot.
- **Positional interpolation:** The why is natural movement when a perch changes, using a flight arc or hop instead of a snap.
- **Mood transition blending:** The why is smooth animation behavior rather than "snapping animation frames."
- **Absence recovery via bootstrap:** The why is immediate canonical recovery after hours away, with birds initialized mid-action at server positions.

### Frontend Rendering & Animation Pipeline

- **Canvas / WebGL 2D renderer:** The why is to make the scene feel "like an open window on a living habitat."
- **Layered scene graph with parallax:** The why is depth and habitat atmosphere across sky, foliage, perch zones, leaves, branches, droplets, and call captions.
- **Fixed visual aspect bounds and responsive letterbox/pillarbox:** The why is stable visible bounds across desktop and mobile browsers.
- **Zero Bird Cropping:** The why is that all 7 birds remain "100% visible on screen without scrolling or panning."
- **Three depth perch zones:** The why is visual depth: front birds get crisp detail and full saturation, back birds get smaller scale and atmospheric haze.
- **Procedural skeletal micro-motion:** The why is living behavior through breathing, head tilts, saccades, preening, body shuffle, and tail twitch.
- **Mid-action startup:** The why is immediate aliveness; birds appear "mid-breath or mid-preen without an entry animation."
- **Diurnal shading:** The why is local-solar-time ambience across dawn, midday, dusk, and night.
- **Weather FX:** The why is to make passing rain and soft wind visibly affect the aviary.
- **Reduced-Motion mode:** The why is accessibility without losing aliveness: motion becomes slow pose cross-fades, perch movement becomes fade-out/fade-in, and drifting particles are suppressed.

### Procedural Audio Synthesis & Chorus Engine

- **Client-side WebAudio synthesis:** The why is that "no recorded audio files or loops are used."
- **Six-species call grammar library:** The why is species-specific procedural motifs defined by frequencies, modulation, filters, and transition graphs.
- **Pitch and micro-timing jitter:** The why is to "eliminate mechanical repetition."
- **Spatial panning:** The why is that each bird's audio position maps to its horizontal perch position.
- **Listen-In mix ramping:** The why is smooth focus: one bird becomes more present while the others become soft accompaniment and are never fully muted.
- **WebAudio graceful silence fallback:** The why is to avoid loading "low-quality recorded audio files or canned audio fallbacks"; captions continue automatically.

### Accessibility Surfaces & UX Details

- **Screen-reader narration live region:** The why is naturalist descriptions of the aviary scene through an `aria-live="polite"` surface.
- **Narration pacing and queue management:** The why is to avoid flooding screen-reader speech queues with 30-60 second background cadence.
- **Priority narration events:** The why is immediate courteous prose for offer accepted, return-greeting, and settle gesture.
- **Real-time call captions:** The why is motif-matched procedural text above the vocalizing bird.
- **Keyboard navigation and focus hierarchy:** The why is full keyboard control across top-bar actions, birds, Listen-In, and panels.
- **High-contrast dual-ring focus indicator:** The why is a visible 4.5:1 minimum contrast outline across all lighting states.
- **Contrast and legibility compliance:** The why is WCAG AA readability for chrome, dialogs, notebook text, captions, headings, and icons.

### Performance Budgets & Observability

- **Initial JS bundle budget:** The why is the PWA-ready web bundle under 2MB, enforced by analyzer and CI failure before the cap.
- **Time-to-First-Bird budget:** The why is immediate visibility through inlined snapshot bootstrap and pre-parsed silhouettes.
- **Idle 60fps budget:** The why is steady rendering on a 5-year-old reference laptop.
- **Zero memory growth budget:** The why is stable 30-minute sessions verified by automated Playwright leak detection.
- **Simulation tick p99 budget:** The why is operational health of the continuous tick engine.
- **Permitted operational telemetry:** The why is aggregate service health without exposing per-account behavior.
- **Prohibited telemetry:** The why is strict privacy: no per-account events, trait distributions, mood analytics, emails, IP addresses, or location tracking.

### Rollout Plan & Aviary Lifecycle

- **Starter adoption experience:** The why is a quiet first-run path: magic link, allocated aviary, two distinct species, quiet naming prompt, and soft initial flights into the "initial quiet field."
- **Calendar-age bird additions:** The why is anti-gamification; additions are not tied to streaks, clicks, or points.
- **Gentle species arrival motif on next morning session:** The why is that growth appears as a quiet arrival rather than a popup or milestone.
- **Strict cap at 7 birds:** The why is procedural audio recognizability.

### Risk Analysis & Mitigation Matrix

- **Drift saturation mitigation:** The why is avoiding the "Tamagotchi Trap"; if traits ramp too fast, birds saturate in days and the "illusion breaks."
- **Multi-device desynchronization mitigation:** The why is preventing overwritten drift and lost session history by keeping the server as sole state writer.
- **Audio fatigue mitigation:** The why is that mechanical repetition would make the user mute the tab and the "core aliveness" would be lost.
- **Ghost presence mitigation:** The why is preventing background tabs or forgotten windows from accumulating presence.
- **Voice tone contamination mitigation:** The why is preserving the core product aesthetic by blocking gamified toasts and forbidden phrases.
- **Accessibility regression mitigation:** The why is preventing reduced-motion users from being excluded from experiencing aliveness.
