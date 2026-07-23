## System-level intent

- **Quiet, ambient, non-gamified care**: The plan frames Pocket Aviary as a "single-screen, non-panning horizontal aviary scene" with "continuous time-of-day/ambient weather rendering," while explicitly excluding "streaks," "badges," "XP," "levels," "scores," and "visit-count displays." This intent also appears in "Optional & Quiet" social design and the "No Tamagotchi Mechanics" rule.

- **Non-punitive, slow personality change**: The hidden personality system is calibrated around "Low-Pass Drift" and a "slow monotonic drift target: 3 weeks." The clearest philosophy is the "Asymmetrical Monotonic Rule": "Neglect results in zero change" and "Traits never decrease." The plan repeats this as "without mistrust or decay."

- **Server-side canonical truth**: The plan repeatedly makes the server the authority: "single canonical server-side simulation tick," "No Client State Ownership," "server is the sole writer," and "No Last-Write-Wins." This shows up in architecture, sync resolution, and the risk mitigation for "Multi-Device State Conflict."

- **Event-log input, snapshot output**: The architecture separates "Event Streams" from "State Snapshots & SSE," uses an "Immutable append-only log for raw interaction events," and serves clients through a "read-optimized snapshot service." The intent is carried through the Edge Gateway, Simulation Engine, Snapshot Read Service, Redis cache, and canonical PostgreSQL writes.

- **Naturalist prose as the product voice**: The Field Notebook is "naturalist, prose-based observation log generated server-side." The same voice appears in "Lowercase naturalist prose," "Screen-Reader Narration," procedural call captions such as "a soft three-note rise," and the risk mitigation to "Share single naturalist prose generation engine across notebook, narration live region, and call captions."

- **Privacy and PII isolation by design**: The account model uses "Synthetic UUID account keying (PII isolation)," encrypted email, a blind hash index, and a "Strict Privacy Rule" that telemetry pipelines are "physically isolated from simulation databases." The risk table reinforces this with "Strict synthetic UUID primary keying" and email stored in a "single encrypted table column."

- **Graceful accessibility and fallback behavior**: Accessibility is not treated as a separate log view. The plan specifies "ARIA Live," "Reduced-Motion Mode," "Call Captioning," "silent-caption fallback," and high-contrast focus rings "tailored for AA compliance on dynamic scene backgrounds."

- **Immediate, performance-budgeted presence**: The plan sets "time-to-first-bird < 500ms," "First Frame Continuity," "No loading spinners or fade-ins," a bundle budget under 2 MB, and "60fps render on 5-year-old hardware." The product intent is that the aviary is visible and alive quickly, even on constrained devices.

- **Procedural audio/visual life without shipped loops**: The plan favors "procedural call motif libraries," "WebAudio procedural synthesis," native oscillators, randomized pitch/timing micro-offsets, and "No pre-recorded MP3/WAV files." The risk table ties this to avoiding "repetitive or phase-canceled" audio.

- **Opt-in, bounded social presence**: Social is "Optional & Quiet," "Opt-in," "host-initiated," "read-only," one-time, expiring, and revocable. The non-goals reject "public feeds," "discovery/explore tabs," "leaderboards," "user profiles," "comments," and "co-presence/shared cursors."

## Per-feature whys

### Scope & System Boundaries

- **Modern desktop and mobile web browsers**: NOT RECOVERABLE FROM PLAN

- **Single-screen, non-panning horizontal aviary scene**: The plan positions this as the "Core Experience" and pairs it with "continuous time-of-day/ambient weather rendering," giving the whole product a contained ambient surface rather than a navigated world.

- **Continuous time-of-day and ambient weather rendering**: The plan uses local time and weather as simulation inputs: "Dawn -> Alert," "Dusk -> Drowsy," "Rain -> Dampened Vocal / Wary," and `Mood_t = f(Personality, Weather, LocalTime, RecentInteractions)`.

- **Starter population of 2 birds and strict capacity of 7 birds**: NOT RECOVERABLE FROM PLAN

- **Progressive adoption unlocked via aviary age milestones**: NOT RECOVERABLE FROM PLAN

- **Six initial bird species with unique visual silhouettes, color palettes, and procedural call motif libraries**: NOT RECOVERABLE FROM PLAN

- **User-assignable and renameable bird names with persistent internal UUIDs**: The plan separates user-facing names from internal identity by combining "given_name" with UUID bird IDs, so renaming does not change the persistent bird record.

- **Presence tracking with strict multi-signal conjunction**: The plan makes valid presence depend on all three client signals: visible document, focused document, and recent pointer or keyboard input. If any condition fails, "presence pings cease immediately."

- **Return-greeting, procedurally staggered and personality/absence-weighted**: NOT RECOVERABLE FROM PLAN

- **Listen-in with gradual WebAudio re-balance and solo focus**: The plan states that listen-in raises the focused bird slowly while non-focused birds ramp down but are "never zeroed out," preserving the chorus while creating focus.

- **Offers: seed, song fragment, still pool, with per-bird cooldowns**: NOT RECOVERABLE FROM PLAN

- **Settle as a soft evening shift session end gesture with 5s undo window**: The plan articulates this feature as a "soft evening shift session end gesture" and includes an undo window, but gives no further rationale beyond that gesture language.

- **Field Notebook generated server-side**: The plan says noteworthy triggers and cooldowns produce "naturalist prose" inserted into notebook entries, and later ties this same prose engine to accessible narration and call captions.

- **Email magic-link auth with 15-minute, single-use links**: NOT RECOVERABLE FROM PLAN

- **Synthetic UUID account keying**: The plan gives the reason directly as "PII isolation" and later ties the same choice to preventing "User emails leak into logs/spans."

- **Multi-device sync driven by a single canonical server-side simulation tick**: The plan says phone and desktop read "identical JSON snapshots generated by the server tick," avoiding "state overwrite bugs across concurrent devices."

- **Soft deletion with 30-day window and full JSON account export**: NOT RECOVERABLE FROM PLAN

- **Opt-in, host-initiated read-only ambient visits**: The plan's rationale is boundary-setting: visits are "Optional & Quiet," "read-only," one-time email invite links, expiring after 30 days, revocable immediately, with a private visit log.

- **No gamification**: The plan excludes "streaks, green-dot calendars, badges, XP, levels, scores, or visit-count displays," aligning the product with quiet ambient care rather than quantified progression.

- **No Tamagotchi mechanics**: The plan excludes "bird death, hunger, sickness, distress meters, or negative personality drift on neglect," which matches the monotonic drift rule that absence creates "zero change" and no "mistrust or decay."

- **No social network features**: The plan excludes "public feeds," "discovery/explore tabs," "leaderboards," "user profiles," "comments," and "co-presence/shared cursors," reinforcing the "Optional & Quiet" social boundary.

- **Web-only, no native apps**: NOT RECOVERABLE FROM PLAN

- **No client-side simulation authority**: The plan's rationale is explicit: clients "never mutate personality vectors or compute canonical drift," and this eliminates "state overwrite bugs across concurrent devices."

### Architecture & Service Topology

- **Decoupled client-server architecture with edge delivery, event ingestion, stateful tick engine, and read-optimized snapshot service**: The plan separates client event submission from server simulation and snapshot reads, so raw interactions flow into the event log while clients receive canonical cached state.

- **Edge Gateway**: The gateway exists to handle "magic-link authentication, session token issuance/validation, and CORS," validate incoming events by "schema, rate limits, token scopes," and append only valid events to the message broker.

- **Simulation Engine**: The engine is the canonical worker that consumes events and presence pings, computes mood, weather, calls, personality drift, Field Notebook observations, and writes canonical state to PostgreSQL plus snapshots to Redis/CDN.

- **Snapshot Read Service**: The plan makes this service read-optimized so clients can receive "lightweight JSON state snapshots" on request or through "Server-Sent Events (SSE) / WebSocket keepalives."

- **PostgreSQL canonical store**: The plan assigns PostgreSQL the canonical records for accounts, birds, personality vectors, species, notebook logs, and visit invitations.

- **Redis active-state cache**: The plan uses Redis as a "High-speed cache" for current active aviary snapshots and ephemeral presence session flags.

- **Kafka / Apache Pulsar append-only event log**: The plan uses this for an "Immutable append-only log for raw interaction events," supporting event consumption by the Simulation Engine.

### Data Model & Schema Definitions

- **Accounts table with encrypted email and blind index**: The plan ties this to PII isolation and the mitigation for PII leakage: email is encrypted and looked up through a blind hash index.

- **Aviaries table with one aviary per account**: NOT RECOVERABLE FROM PLAN

- **Species table with default palette and call grammar config**: NOT RECOVERABLE FROM PLAN

- **Birds table with hidden bounded personality vector**: The plan uses this vector to drive slow personality drift, mood transitions, perch behavior, plumage saturation, and call frequency while keeping the traits hidden.

- **Current mood state on each bird**: The plan needs this for the "fast-timescale mood" computed from personality, weather, local time, and recent interactions.

- **Spatial perch state on each bird**: The plan maps perch zones to behavior: "Wary / Low-boldness birds" in Back, "Content / Standard perching" in Middle, and "Bold / Attentive birds" in Front.

- **Interaction events table**: The plan uses interaction events as raw append-only inputs for the simulation tick, including presence, listen-in, offers, and settle.

- **Notebook entries table**: The plan stores "Lowercase naturalist prose" with the trigger that produced it, matching the server-generated Field Notebook feature.

- **Visit invitations table**: The plan uses this to support one-time email invite links, expiry, revocation, visitor email isolation, and read-only ambient visits.

- **Visit logs table**: The plan uses this for the "Private visit log in settings."

### API Surface & Protocols

- **Request sign-in magic link endpoint**: NOT RECOVERABLE FROM PLAN

- **Verify magic-link token endpoint**: NOT RECOVERABLE FROM PLAN

- **Retrieve current canonical aviary snapshot endpoint**: The plan uses this endpoint to give clients the current server-generated snapshot, including weather, time of day, settled state, and birds.

- **Send interaction event payload endpoint**: The plan uses this endpoint so clients submit raw events such as presence ping, offer, listen-in, and settle rather than mutating canonical state.

- **Fetch paginated field notebook entries endpoint**: NOT RECOVERABLE FROM PLAN

- **Issue visit invite link endpoint**: The plan uses this to create the host-initiated, opt-in ambient visit path.

- **Revoke active visit invite endpoint**: The plan uses this to provide "immediate revocation" for quiet social visits.

- **Fetch visitor read-only snapshot endpoint**: The plan uses this to enforce read-only ambient visiting rather than social co-presence or mutation.

- **State Snapshot Schema**: The schema carries only the client-renderable canonical state: weather, time of day, settled flag, bird mood, perch zone, plumage saturation, call motif seed, and idle animation state.

### Simulation Engine Design

- **Server-side tick loop every 60 seconds for each active aviary**: The plan uses the tick to read event deltas, validate presence, execute low-pass drift math, compute mood, evaluate notebook triggers, and write canonical state/cache.

- **Low-pass drift calibration**: The plan calibrates drift to be "Measurable numerically at ~1 week" and "visually felt by user at ~3 weeks," with CI assertions to prevent birds feeling too fast or "frozen."

- **Presence-time as primary driver for boldness and plumage saturation**: The plan names presence-time as the "Primary driver" for those traits.

- **Listen-in duration as targeted driver for social warmth and vocal frequency**: The plan names listen-in duration as a "Targeted driver" for those traits.

- **Offer interaction as targeted driver for curiosity and boldness**: The plan names offer interaction as a "Targeted driver" for those traits.

- **Asymmetrical monotonic rule**: The plan's rationale is non-punitive: "Neglect results in zero change," "Traits never decrease," and absent birds shift without "mistrust or decay."

- **Mood state machine**: The plan uses local time, weather, interactions, and personality vector dampening so mood responds to immediate context without changing the slower personality vector.

- **Personality vector dampening of wary transitions**: The plan states that high boldness reduces Wary transition probability by 60%, letting long-term trait drift affect short-term mood.

### Presence & Multi-Device Sync Protocol

- **Presence pings every 30 seconds only when visible, focused, and recently active**: The plan's rationale is valid presence measurement: all three conditions must hold, and pings stop immediately when any fail.

- **Server as sole writer of bird personality vectors**: The plan's rationale is canonical coherence across devices and removal of client-side vector mutation.

- **Clients submit raw interaction events to append-only event log**: The plan keeps client input as raw events, leaving personality drift and canonical state to server tick processing.

- **No Last-Write-Wins**: The plan rejects LWW because client-side vector mutation is impossible and this eliminates concurrent-device overwrite bugs.

### Frontend & Audio Rendering Pipeline

- **HTML5 canvas rendering via 2D Context or lightweight WebGL fallback**: NOT RECOVERABLE FROM PLAN

- **Three depth perches: Back, Middle, Front**: The plan uses depth zones to express bird state: wary or low-boldness birds in back, content birds in middle, and bold or attentive birds in front.

- **First Frame Continuity**: The plan gives the rationale directly: birds are drawn from snapshot-zero coordinates with "No loading spinners or fade-ins."

- **Top Bar UI fading after cursor stillness**: NOT RECOVERABLE FROM PLAN

- **Sky and weather background with time-of-day gradient, rain, and wind**: The plan uses weather and time as both visual ambience and mood inputs.

- **Foreground foliage and subtle parallax drift**: NOT RECOVERABLE FROM PLAN

- **Procedural motif synthesizer**: The plan uses native oscillators, filters, envelopes, and procedural motif variation instead of shipped audio files.

- **Chorus re-balancing during listen-in**: The plan makes focus gradual: focused bird gain ramps up over 1.2s, non-focused birds ramp down over 1.2s, and non-focused birds are "never zeroed out."

- **Graceful audio fallback**: The plan's rationale is resilience: if `AudioContext` is blocked or unavailable, the system enters quiet mode with automatic call captioning.

- **No pre-recorded MP3/WAV files**: The plan ties this to procedural synthesis and the risk mitigation of "Zero stacked audio loops."

### Accessibility Surfaces

- **Screen-reader narration through hidden ARIA live region**: The plan uses polite, atomic updates and server-generated naturalist prose every 30-60 seconds so the aviary is available as narration rather than a generic state log.

- **Reduced-motion mode**: The plan replaces skeletal frame-by-frame animation with "gentle 800ms cross-fades between static poses" and disables leaf and feather drift particles.

- **Call captioning**: The plan provides opt-in visual overlays near calling birds, generated from motif parameters as naturalist prose.

- **Keyboard and focus support**: The plan gives keyboard access to the scene and birds, with high-contrast focus rings for "AA compliance on dynamic scene backgrounds."

### Performance Budgets & Observability

- **Initial JS bundle under 2 MB**: The plan uses this as a technical budget verified by CI bundle analysis.

- **Time-to-first-bird under 500 ms**: The plan uses this as a user-visible performance budget verified by Lighthouse/WebPageTest over simulated 4G.

- **Idle render at 60 fps**: The plan uses this as a rendering budget verified by Chrome Performance Tracing on a 5-year-old laptop profile.

- **Zero memory leakage over 30 minutes**: The plan uses heap dump assertions before and after a 30-minute idle run.

- **Simulation tick p99 latency under 5000 ms**: The plan uses monitoring alert rules on the worker queue to keep server ticks within budget.

- **Aggregated operational telemetry only**: The plan limits telemetry to request rates, HTTP 5xx errors, CDN cache hit ratio, render frame drops, and WebAudio error codes.

- **Telemetry isolated from simulation databases**: The plan's rationale is privacy: no per-account bird interactions or notebook logs enter analytics stores.

### Rollout & Staging Strategy

- **Synthetic Simulation Validation**: The plan uses 10,000 synthetic aviaries over 30 simulated days to validate low-pass drift convergence and notebook prose generation rates.

- **Closed Internal Alpha**: The plan uses internal staging and team-domain auth to verify multi-device sync across desktop and mobile browsers.

- **Beta Rollout**: The plan uses a 5% canary to monitor CDN snapshot latency, WebAudio initialization success rates, and simulation tick queue depths.

- **General Availability**: The plan moves to 100% rollout with automated tick queue auto-scaling.

### Risk Matrix & Mitigations

- **Drift Calibration Imbalance mitigation**: The plan says automated low-pass mathematical assertions and 3-week simulated visits prevent birds from drifting too fast into a "Tamagotchi feel" or too slow so they feel "frozen."

- **Multi-Device State Conflict mitigation**: The plan uses "Server-only canonical simulation writer pattern," additive event log consumption, and "zero client-side state authority" to prevent devices overwriting personality drift histories.

- **Audio Uncanniness / Phase Cancellation mitigation**: The plan uses procedural motif variations and randomized pitch/timing micro-offsets, with "Zero stacked audio loops."

- **Accessibility Degradation mitigation**: The plan uses one naturalist prose generation engine across notebook, narration live region, and call captions so accessible surfaces do not feel like "generic state logs."

- **PII Leakage in Observability mitigation**: The plan uses synthetic UUID primary keys, encrypted email storage, and blind hash indexing so user emails do not leak into logs or spans.
