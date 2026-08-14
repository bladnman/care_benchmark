## System-level intent

- Felt aliveness is the core engineering imperative. It appears in Product Vision as "**felt aliveness**" and is carried through the "canonical server-side tick," birds "already in motion upon first frame," "non-repeating chorus acoustics," and personality vectors that "drift monotonically toward expressive behavior."
- The product wants a quiet, observational relationship rather than a game loop. The plan names a "quiet, observational relationship" across "days and weeks," a "Field Notebook" with sparse entries, "No Gamification," and absence that produces "ambient quietness, never distress."
- Presence must be honest, idle, and non-coercive. The plan uses "honest idle presence" in the core tenet, requires "tri-condition presence detection," and later ties this to preventing "background tabs or unattended open laptops inflating presence time."
- Server authority protects canonical continuity. The architecture makes the server responsible for "canonical bird personality vectors," "mood state," "aviary clock," and "monotonic drift integration," and the sync section states "The Server is the Sole Writer."
- Growth is expressive but hidden. The plan repeatedly treats personality vectors as "hidden," says values "never decrease," and prohibits "Numerical Personality Exposure" through UI or debug APIs.
- Product voice separates naturalist observation from system mechanics. Notebook entries and screen-reader narration use "naturalist, lowercase, sparse" or "literary, lowercase naturalist prose," while errors use "matter-of-fact tone" and strictly avoid naturalist prose.
- The media system should feel procedural, non-repeating, and uncanned. The plan calls for "procedural WebAudio synthesis," "No Recorded Loops," "Zero Audio Files," Markovian call motif grammar, and procedural skeletal micro-motion rather than frame-by-frame sprites.
- The web surface should appear immediately and quietly. This shows up in "Web-only," bundle budgets, "Time to First Bird Visible," "Immediate First Paint," "Fast State Hydration," and "No Spinners."
- Accessibility is first-class, not a later patch. The plan explicitly scopes "First-class naturalist screen-reader prose narration," "live descriptive call captions," keyboard navigation, WCAG AA chrome, and a "dedicated reduced-motion mode."
- Privacy is a product constraint, not just storage detail. The plan says "Zero aggregation or ML usage" of per-bird/per-account interaction data, uses "synthetic account UUIDs isolating PII," and excludes trait values, names, histories, and visit logs from telemetry.
- Social sharing is quiet and bounded. The visit flow is "read-only," "non-co-present," immediately revocable, expiring after 30 days, with "No Social Network Features" such as feeds, discovery, comments, avatars, or leaderboards.

## Per-feature whys

### Executive Summary & Scope

- Core Simulation Engine: The plan's rationale is to make the aviary feel alive through a "server-side simulation loop (60s tick)" that computes drift, mood transitions, and autonomous social dynamics continuously.
- Bird Population Model: The plan ties population to the long relationship arc: users cultivate birds "across days and weeks," starting with 2 and scaling by aviary calendar age up to "a hard cap of 7 birds."
- User-assigned stable naming with immutable internal UUIDs: NOT RECOVERABLE FROM PLAN
- Honest presence detection: The why is to reward "honest idle presence" and avoid false presence; the risk matrix says background tabs or unattended laptops must not inflate presence time.
- Procedural return-greetings: NOT RECOVERABLE FROM PLAN
- Smooth listen-in mix rebalancing: The plan articulates focus without isolation; the selected bird ramps up, other birds ramp down, and other birds "NEVER mute completely (preserving flock presence)."
- Three offer types with per-bird cooldowns: NOT RECOVERABLE FROM PLAN
- Opt-in settle evening transition with 5s cancel window: NOT RECOVERABLE FROM PLAN
- Field Notebook Service: The rationale is to preserve a "naturalist, lowercase, sparse" observational record, generated only from "noteworthy simulation moments" at about "1 entry every few days."
- Responsive single horizontal scene with three perch zones: The scene supports quiet observation and depth; perch zones are later tied to boldness, mood, scaled planes, desaturation, detail, and "Perch Zone Compositing."
- Procedural day/night solar curve, weather, and particles: The plan uses local solar time and weather to affect mood, call rates, shading, and ambient movement so the aviary changes without explicit game mechanics.
- Procedural Audio Engine: The rationale is "non-repeating chorus acoustics" through WebAudio synthesis, species motif grammars, spatial panning, chorus mixing, and graceful silence instead of canned loops.
- Accounts, Auth & Multi-Device Sync: The plan's why is secure, private continuity across devices: passwordless magic links, secure cookie sessions, synthetic account UUIDs, append-only events, and canonical snapshots prevent last-write-wins conflicts.
- Quiet Social visits: The rationale is sharing an ambient view without social-network pressure or mutation: visitor views are read-only, non-co-present, not recorded as interactions, and do not change drift.
- Accessibility: The plan makes a11y part of the product experience through naturalist narration, captions, keyboard controls, high-contrast focus, WCAG AA, and reduced-motion cross-fades.
- Telemetry & Privacy: The rationale is operational health without behavioral surveillance: only latencies, WebAudio error rates, bundle size, and frame times are collected, with zero aggregation or ML over bird/account interaction data.
- No Gamification: The plan prohibits streaks, XP, levels, counters, scores, badges, achievements, and milestones to protect the quiet observational premise.
- No Tamagotchi / Custodial Penalties: The plan explicitly says absence should create "ambient quietness, never distress," with no hunger, sickness, death, decay meters, or negative drift on neglect.
- No Social Network Features: The rationale is to keep visits quiet and bounded; the plan disallows feeds, discovery, friend graphs, comments, chat, avatars, and leaderboards.
- No Native Apps: NOT RECOVERABLE FROM PLAN
- No Intrusive Announcements: The plan rejects "Welcome Back" banners, modals, toasts, and notification loops so returning remains ambient rather than attention-demanding.
- No Numerical Personality Exposure: The rationale is that vector weights are "strictly internal server state," preserving hidden personality drift rather than turning traits into UI numbers.

### System Architecture & Service Topology

- Frontend Client as pure TypeScript Canvas/WebAudio SPA: The plan ties this to a lightweight web surface with "No heavyweight runtime frameworks" and an initial bundle under 2MB gzipped.
- Simulation Tick Worker Engine: The rationale is horizontal, canonical processing of active and dormant aviaries every 60 seconds, partitioned by account hash.
- Primary Relational Store: PostgreSQL is used for "transactional persistence" of accounts, birds, event logs, notebook entries, and visits.
- Fast State Cache: Redis exists for "fast snapshot reads," magic-link validation, rate limiting, and ephemeral presence buffering.
- Server Authority: The plan makes the server authoritative for personality vectors, moods, timers, weather, age-based unlocks, drift integration, and notebook persistence so clients cannot submit canonical state.
- Client Authority: The client owns high-frequency rendering, WebAudio synthesis, local interpolation, presence detection, and particles because those are local, responsive, presentation-layer behaviors based on server seeds and snapshots.

### Data Model & Storage Schema

- Encrypted account email and blind index: The rationale is privacy isolation with deterministic lookup, shown by `email_encrypted` plus "HMAC blind index for deterministic lookup."
- Append-only interaction events: The plan uses append-only events so interactions can be drained by the tick worker and processed sequentially rather than overwriting canonical state.
- Notebook entries persistence: The plan persists entries so sparse naturalist observations remain available through the notebook service.
- Visit invitations with token hash, encrypted recipient email, expiry, and revocation: The rationale is bounded, private, revocable sharing with token lookup and a host-visible invitation lifecycle.

### API Surface & Communication Protocols

- Minimal REST/JSON over HTTPS with HttpOnly cookie session: The rationale is simple snapshot/event/auth communication with secure session identification.
- Magic link request and verify endpoints: The plan uses 15-minute cryptographically random tokens, Redis hashing, immediate invalidation, and session cookies for passwordless sign-in.
- Magic-link rate limit of 3 requests per 15 minutes per IP/email: NOT RECOVERABLE FROM PLAN
- Aviary snapshot endpoint: The endpoint serves the canonical state used by clients, including weather, settled state, birds, moods, plumage saturation, and motif seeds.
- Aviary events endpoint: The endpoint accepts local interactions as queued events, matching the append-only event stream and server-side tick processing model.
- Field Notebook endpoint: The endpoint exposes the persisted sparse notebook entries in reverse time order, supporting the naturalist observation surface.
- Visit invitation endpoints: The plan uses invite, active, delete, and view endpoints so hosts can create, inspect, revoke, and share read-only visits with 404/410 responses when revoked or expired.

### Simulation Engine Design & Mechanics

- 60-second server-side tick loop: The rationale is a canonical recurring process that drains events, aggregates presence, applies drift, updates mood/perches, synthesizes observations, and serializes snapshots.
- Monotonic Low-Pass Drift Function: The plan's why is gradual expressiveness calibrated to be "instrumentally detectable" after about 7 days and "visually/audibly distinct" after about 21 days.
- Asymmetric Non-Decay Invariant: The rationale is anti-penalty continuity; if the user is absent, deltas are zero, values never decrease, and the bird "merely" becomes ambient in moment-to-moment behavior.
- Mood Transition Engine: The plan uses faster mood timescales, local solar time, weather, and interaction nudges so behavior changes over hours without changing long-term personality.
- Call Grammar & Chorus Engine: The rationale is natural chorus dynamics: deterministic motif seeds, species grammars, warmth-based replies, staggered delays, and prevention of "synthetic unison calling."
- Species Pool: The plan gives each species distinct pitch, call, activity, perch, plumage, boldness, or nocturnal behavior, supporting varied audiovisual expression.
- Adoption Unlock Schedule: The rationale is calendar-age-based gradual arrival from 2 starter birds to the 7-bird hard cap, matching the long-term aviary relationship rather than immediate collection.

### Multi-Device Sync & Conflict Prevention

- Strict Single-Writer Principle: The plan states the why directly: no last-write-wins corruption, no state divergence, and no lost interaction credit when laptop and phone sessions overlap.
- Snapshot Fetch Cadence: Fetching on load, visibility return, wake from sleep, and 60-second focused keepalive keeps devices aligned to the canonical snapshot without client-to-client conflict.
- Error Surfaces & Voice Separation: The rationale is to reserve naturalist prose for the aviary and use "matter-of-fact tone" for auth, session, and network failures.

### Client-Side Rendering Pipeline

- Scene Graph & Composition: The plan uses backdrop, back/middle/front perch zones, foreground particles, scaling, desaturation, feather detail, and solar palette shading to create readable depth in a single canvas.
- Zero-Loading-State Initialization: The rationale is immediate quiet presence: pre-computed sky, cached bird positions, smooth interpolation when the snapshot arrives, and "No Spinners."
- Micro-Motion Engine: Procedural breathing, head-tilts, scanning, preening, shuffling, and flight transitions exist to make idle birds feel alive without frame-by-frame sprites.
- Reduced-Motion Pipeline: The rationale is accessible motion control: disable skeletal motion and particles, replace flights with cross-fades, and slow day/night transitions.
- Top Bar UI Chrome: The plan keeps chrome minimal and fading so controls exist without dominating the observational surface.

### WebAudio Procedural Sound Architecture

- Procedural Synthesis Engine with no recorded loops: The rationale is pure WebAudio sound with "0 bytes of audio media," small code size, and no recorded or canned loop artifacts.
- Mix Matrix & Listen-In Ducking: The plan gives focus to the selected bird while preserving "flock presence" through dynamic scaling and non-muting ducking.
- WebAudio Fallback: The rationale is graceful silent mode with automatic call captions while protecting bundle budgets and preventing canned fallback audio.

### Accessibility Architecture

- Naturalist Screen-Reader Narration Surface: The plan gives screen readers a periodic naturalist equivalent of scene state while throttling updates to 30-60 seconds and interrupting only for direct gestures.
- Call Captioning Subsystem: Captions describe active motifs near the calling bird and in aria-live, synchronized with the synth envelope for non-audio access to calls.
- Keyboard Navigation & Focus Ring: The rationale is full keyboard operation and WCAG AA-visible focus across top bar, birds, overlays, listen-in, notebook, offers, and settle.

### Performance Budgets & Observability

- Hard Performance Budgets: The plan links budget targets to enforcement: bundle-size CI failures, edge snapshot and canvas pre-render for TTFBird, rAF frame budgets, reusable audio nodes, pre-allocated particle buffers, leak tests, and Prometheus/Grafana alerts.
- Observability & Privacy-Preserving Telemetry: The rationale is operational visibility for HTTP, frame drops, WebAudio failures, queue depth, and tick latency while excluding trait values, names, mood distributions, histories, offers, visits, tracking IDs, and advertising analytics.

### Rollout & Release Plan

- Phased Rollout Schedule: NOT RECOVERABLE FROM PLAN
- Day-1 Instrumentation: The rationale is launch health visibility through synthetic end-to-end browser runners from 5 global edge locations and Sentry logging scrubbed of email/PII attributes.

### Risk Matrix & Mitigation Strategies

- Drift Calibration mitigation: The plan's why is avoiding personality drift that is too fast ("Tamagotchi effect") or too slow ("screensaver effect") through headless multi-month simulations.
- Sync Race Conditions mitigation: The rationale is avoiding state divergence and lost interaction credit by enforcing append-only ingestion and server-only personality computation.
- Audio Uncanniness mitigation: The plan wants to avoid harsh, robotic, or fatiguing procedural audio through resonator filters, randomized pitch micro-deviations, and harmonic overtone saturation.
- Accessibility Degradation mitigation: The rationale is preventing ARIA live spam or desynchronization through throttled narration and priority interruption only for direct gestures.
- Presence False Positives mitigation: The rationale is preventing background tabs or unattended laptops from inflating presence through strict tri-condition enforcement.
