## System-level intent

1. **Persistent, slow, observational relationship instead of task completion**

   The plan opens by defining Pocket Aviary as "a persistent, browser-based ambient aviary" where users form "slow, observational relationships" with birds. This intent shows up again in "passive idle attention," "rare, auto-generated naturalist observations," the "automated naturalist diary," and the risks around birds changing too quickly and feeling "like a clicker game" or too slowly and feeling like a "screensaver."

2. **Non-punitive ambient care**

   The plan repeatedly rejects punishment and obligation. "No Tamagotchi / Custodial Dynamics" says birds never die, starve, fall sick, or express distress, and that absence results in "ambient quietness, never punitive decay or negative drift." The simulation encodes the same philosophy as the "Monotonicity Law": "Neglect never reduces a trait."

3. **Anti-gamification and anti-engagement-ping product shape**

   The plan explicitly forbids "streak counters, visit tallies, green-dot activity calendars, badges, achievements, levels, experience points, or progress bars." Growth is paced "entirely by calendar age" with "no interaction requirements, attendance streaks, or adoption fees." The no-push boundary also rejects "marketing emails enticing users back to the aviary."

4. **Server-authoritative canonical simulation over client-owned state**

   The architecture treats the client as presentation and input capture only. In "Server-Authoritative State Invariant," clients are "strictly render and input capture nodes" and never calculate or submit "personality vectors, mood states, or aviary clock variables." This is paired with "append-only event streaming" and "Additive Server Deltas" to make "Last-Write-Wins" anomalies architecturally impossible.

5. **Privacy isolation and minimization**

   The data model separates account identity from PII with "synthetic UUIDv4 primary keys" and encrypted email. Personality vectors are "stored exclusively on the server" and "strictly excluded from client serialization payloads." Observability has a "HARD BOUNDARY" and "Zero per-bird, per-vector, or per-user interaction events" in analytics.

6. **Procedural, lightweight, immediate aliveness**

   The plan insists on "100% procedural WebAudio synthesis," "zero recorded audio samples," "Zero External Media Requests," compact vector/procedural visuals, and an initial bundle under 2 MB. The "First Frame Mid-Action" invariant forbids "entry transitions, spinners, or static fades" so the aviary appears alive immediately.

7. **Accessibility as affective experience, not only compliance**

   The accessibility section states that accessibility is "an emotional, first-class design surface rather than a compliance checklist." This appears in the Naturalist voice for narration and captions, the rejection of "robotic ARIA attribute updates," reduced-motion cross-fades, call captions, keyboard navigation, and dual-tone focus rings.

8. **Quiet social without social-network mechanics or visitor influence**

   The optional social feature is "read-only ambient visit invitations" with "zero visitor co-presence and zero visitor-induced drift." The non-goals rule out "public aviary discovery feeds, user profiles, following lists, comments, chat overlays, visitor avatars, or leaderboards," keeping social presence quiet, revocable, and host-initiated.

## Per-feature whys

**1. Executive Summary & Scope Boundary**

- **Modern web browsers with no native application dependencies**: NOT RECOVERABLE FROM PLAN

- **Exactly two starter birds, age-paced growth, and hard ceiling of seven birds**: Starter count is specified as the adoption shape. Later growth is paced by aviary age so there are "no interaction requirements, attendance streaks, or adoption fees." The seven-bird cap has an explicit empirical justification: "Procedural audio recognizability breaks down above seven concurrent callers," and the cap "preserves intimate, individual bird recognition by ear."

- **Fixed initial pool of six biological species silhouettes**: NOT RECOVERABLE FROM PLAN

- **Single horizontal viewport scene with no panning, zooming, or scrolling**: NOT RECOVERABLE FROM PLAN

- **Three perch zones: front, middle, back**: The plan ties perch zones to behavioral signaling by saying users cannot control placement because "Perch positions are strictly behavioral signals." The zones also support depth presentation and audio distance filtering.

- **Dynamic day/night palette tied to user local time**: The plan uses local solar phase for diurnal mood changes: night increases `drowsy` / `settled`, while morning raises `alert` and `content`. The visual palette supports that diurnal cycle.

- **Rare, subtle ambient weather**: Weather is used as a mood and call-grammar input, with rain lowering call rate, and as a Field Notebook trigger such as `WEATHER_RESPONSE`. The plan does not articulate a broader rationale beyond those simulation hooks.

- **Presence interaction**: Presence exists to compute "passive idle attention" only when `visible`, focused, and recently active. Its rationale is to prevent "Tab left open in background window inflating presence metrics" and "unintended rapid personality drift without actual user attention."

- **Return-Greeting interaction**: The greeting is procedurally varied and shaped by "absence length, boldness, and mood" so returns reflect individual bird state rather than a uniform response. No further rationale is articulated.

- **Listen-In interaction**: Listen-In focuses on a single bird while others remain ambient. The audio mix rationale is to "enhance intimacy and presence" while ensuring background birds are "never silenced," maintaining "background aliveness."

- **Offer interaction: seed, song fragment, still pool**: NOT RECOVERABLE FROM PLAN

- **Functional per-bird offer cooldowns**: NOT RECOVERABLE FROM PLAN

- **Settle interaction**: Settle is a "soft user-initiated session-ending gesture" that shifts the scene to evening lighting. The 5-second undo window gives a brief reversal path, and the plan requires Settle to be "engine-equivalent to tab-close" so presence accounting terminates equivalently.

- **Field Notebook**: The notebook is an "automated naturalist diary" for rare, noteworthy observations. Its lowercase, present-tense prose reinforces the Naturalist voice, and sparsity keeps entries rare: "Max 1 entry per 3-5 days" / "maximum 1 entry per 72 hours under regular visitation."

- **100% procedural WebAudio for avian vocalizations**: The plan's rationale is to avoid shipped samples and "canned audio," reduce external media requests, keep the bundle lightweight, and generate species-accurate "chirps, trills, and complex overtones."

- **Spatial positioning for calls and choruses**: Spatial panning and depth filtering map sound to screen position and perch depth, supporting recognizability and a natural "Spatial & Chorus Mixer."

- **Graceful silence mode with automatic call captions**: This preserves access when `AudioContext` is unsupported, blocked, or fails, while strictly avoiding MP3/WAV fallback loops and keeping call information available through captions.

- **Email magic-link accounts**: The 15-minute token lifetime is specified, and the endpoint uses constant-time response to prevent account enumeration. No broader rationale for choosing magic links is articulated.

- **Revocable per-device session tokens**: NOT RECOVERABLE FROM PLAN

- **Synthetic UUIDs separating identity from encrypted PII**: The rationale is privacy and breach containment: "Synthetic UUIDv4 primary keys used everywhere" and email encrypted "in a single table" mitigate "PII Contamination."

- **Server-authoritative simulation tick**: The tick runs independently of clients to advance canonical state, consume ordered events, and prevent stale clients from overwriting newer state.

- **Append-only client event streaming**: The rationale is to avoid "Last-Write-Wins" anomalies. Because clients submit events rather than absolute state, all mutations become "server-calculated derivations of the ordered, append-only event stream."

- **JSON account state export**: NOT RECOVERABLE FROM PLAN

- **30-day soft deletion and restore**: The restore endpoint cancels deletion within the 30-day window. The plan does not articulate why the window is 30 days.

- **Host-initiated read-only ambient visit invitations**: The rationale is quiet social access without social-network mechanics: visitors receive read-only snapshots, cannot submit events, and cause "zero visitor-induced drift."

- **One-time email visit links with 30-day expiration**: NOT RECOVERABLE FROM PLAN

- **Immediate visit invitation revocation**: The rationale is host control over visits; revoked links immediately stop returning the aviary and show Matter-of-Fact copy that the invitation is "no longer active."

- **Isolated visit log**: The host can inspect "masked emails and durations." The plan does not articulate a specific rationale beyond host visibility.

- **Dual-register voice architecture**: The rationale is to keep affective aviary surfaces and infrastructure surfaces distinct with "zero cross-contamination": Naturalist voice is lowercase, present-tense, specific, and observational; Matter-of-Fact voice is direct, functional, and uses "no false warmth or performative charm."

- **Screen-reader running prose narration**: The plan rejects "robotic ARIA attribute updates" because they exclude screen-reader users from the "affective aliveness of the aviary." Running Naturalist prose preserves the experience of moment, bird, light, and action.

- **Reduced-motion mode**: The feature removes continuous oscillations, particles, and flight paths, replacing them with "slow alpha cross-fades" and "elegant cross-dissolve" so the aviary remains legible without motion.

- **Dynamic call captions**: Captions make procedural calls available visually, especially in silence/fallback contexts, and match the procedural audio parameters and envelope rather than using generic labels.

- **WCAG AA contrast, keyboard navigation, and dual-tone focus rings**: The rationale is complete navigation and reliable visibility across "bright morning and dim evening lighting."

**1.2 Explicit Non-Goals**

- **No Native Applications**: NOT RECOVERABLE FROM PLAN

- **No Gamification Surfaces**: The rationale is to keep the aviary from becoming a clicker or progress system. The plan rejects progress artifacts and later names the risk that overly aggressive drift "feels like a clicker game."

- **No Tamagotchi / Custodial Dynamics**: The rationale is non-punitive care: birds never suffer, and absence causes "ambient quietness" rather than "punitive decay or negative drift."

- **No Social Network Mechanics**: The rationale is to preserve quiet social boundaries: no discovery feeds, public profiles, chat, avatars, or leaderboards, while optional visiting stays host-initiated and read-only.

- **No Monetization / Commercial Friction**: NOT RECOVERABLE FROM PLAN

- **No Direct Avatar or Scene Placement Control**: The rationale is that "Perch positions are strictly behavioral signals," so user dragging or redecorating would confuse simulation expression with direct customization.

- **No Push / Out-of-App Engagement Pings**: The rationale is to avoid external prompts "enticing users back to the aviary."

**2. System Architecture & Service Topology**

- **Presentation & Interaction Client as vanilla TypeScript SPA with lightweight reactive primitives**: The plan's rationale is performance and small delivery surface: no heavy UI frameworks, a bundle budget under 2 MB, and 60 FPS rendering.

- **Canvas2D / WebGL rendering pipeline**: The rationale is to render the aviary at 60 FPS with procedural visuals, fixed scene layers, and frame-0 aliveness.

- **Presence Monitor**: The rationale is to capture real attention from visibility, focus, and input, and to send interaction events without letting background tabs inflate drift.

- **Screen-Reader Live Narrator & Call Captions in the browser layer**: The rationale is to keep accessibility synchronized with visual and audio state, using live Naturalist prose and procedural call descriptors.

- **Edge Proxy / CDN**: The rationale is to terminate TLS, cache static assets, rate-limit, and inject the initial snapshot to "guarantee the <500ms time-to-first-bird metric."

- **Aviary API & Ingestion Service**: The service validates and batches events, serves canonical snapshots, and appends to the event stream. The plan's why is canonical state access and ordered event ingestion for server-authoritative simulation.

- **Auth & Session Service**: NOT RECOVERABLE FROM PLAN

- **Visit & Social Service**: The rationale is to create, revoke, and serve read-only ambient visits without visitor state mutation.

- **PostgreSQL as relational store and append-only event store**: The rationale is to serve as both system of record and immutable interaction log, supporting canonical snapshots and ordered event processing.

- **Simulation Engine Cluster with stateless tick workers**: The rationale is to drive the aviary every 60 seconds, validate presence integrity, compute drift and moods, schedule calls, generate notebook observations, and atomically commit snapshots.

- **Isolated Operational Telemetry**: The rationale is to monitor latency, ticks, bundle delivery, audio errors, and FPS while enforcing the "HARD BOUNDARY" against user, bird, vector, and interaction analytics.

**3. Data Model & Database Schemas**

- **Synthetic UUID primary keys for accounts and internal references**: The rationale is identity/PII separation and privacy containment.

- **AES-256-GCM encrypted email and salted email hash**: The rationale is encrypted PII storage while allowing lookup by hash.

- **Sessions table with token hashes, user agent, truncated IP prefix, and revocation**: The IP prefix is "for audit without PII." The plan does not articulate a specific rationale for every session field.

- **Magic links table with single-use token hash and expiration**: The rationale is single-use verification with 15-minute expiry. No broader rationale is articulated.

- **Aviary table with `tick_version`, `last_tick_at`, settle fields, and weather fields**: The rationale is canonical snapshot versioning, tick scheduling, settle state, and active weather state.

- **Birds table with stable identities, species, name, perch, mood, and greeting time**: The rationale is persistent bird identity and current canonical presentation state. No deeper rationale is articulated.

- **Personality vectors stored server-side only**: The rationale is privacy and simulation integrity: vectors are "NEVER sent to client."

- **Interaction event log with unprocessed events**: The rationale is immutable, ordered ingestion for tick workers and no LWW state overwrites.

- **Notebook entries table**: The rationale is persistence for naturalist observations and infinite historical scroll.

- **Visit invitations and visit logs tables**: The rationale is revocable, expiring read-only visits and host-viewable visit history with masked emails.

**4. API Surface & Protocols**

- **JSON over HTTPS and secure SameSite session cookie**: NOT RECOVERABLE FROM PLAN

- **`POST /api/v1/auth/magic-link` constant-time response and rate limit**: The rationale is to prevent account enumeration and throttle requests.

- **`POST /api/v1/auth/verify` single-use token consumption**: The rationale is to validate and immediately invalidate the login token before setting a session.

- **`POST /api/v1/auth/session/revoke`**: NOT RECOVERABLE FROM PLAN

- **`POST /api/v1/account/export` via signed download URL to verified email**: NOT RECOVERABLE FROM PLAN

- **`POST /api/v1/account/delete` and `POST /api/v1/account/restore`**: Restore exists to cancel pending deletion during the 30-day window. The plan does not articulate a deletion rationale.

- **`GET /api/v1/aviary/state` with ETag based on `tick_version`**: The rationale is retrieval of the canonical snapshot and version-aware state consumption.

- **Strict absence of personality vector numbers from state payload**: The rationale is privacy and simulation integrity: vectors are never exposed in client payloads, DOM, or network tabs.

- **Batched `POST /api/v1/aviary/events` every 15-30 seconds**: The rationale is append-only ingestion of presence, listen-in, offer, settle, and undo events for server processing.

- **`POST /api/v1/aviary/offers` with cooldown response**: NOT RECOVERABLE FROM PLAN

- **Settle and settle undo endpoints**: The rationale is to support the soft session-ending gesture and 5-second undo grace window.

- **Paginated notebook endpoint ordered by recorded time**: The rationale is read-only historical notebook access.

- **Visit invite, revoke, log, and visitor state endpoints**: The rationale is host-initiated read-only visiting, immediate revocation, host inspection, and rejection of visitor event submissions.

**5. Simulation Engine Design**

- **60-second server-side simulation tick**: The rationale is to advance canonical state independently of browser activity, consume events, compute mood and drift, schedule calls, generate notebook entries, and increment `tick_version`.

- **Ingestion and event verification with row lock and anti-tamper validation**: The rationale is presence-time integrity and bounded valid presence seconds.

- **Monotonic low-pass drift filter**: The rationale is slow personality evolution tied to presence and interactions while enforcing "Zero negative drift on neglect."

- **Fast-timescale mood transitions and social contagion**: The rationale is to reflect diurnal cycle, weather, offers, absence returns, and neighboring bird state without changing slow personality traits.

- **Call grammar and antiphonal stagger scheduling**: The rationale is to derive call frequency from traits and mood, stagger replies, prevent "synchronous cacophony," and create "natural avian dialogue."

- **Rare Field Notebook prose synthesizer**: The rationale is to detect noteworthy predicates, gate them sparsely, and create rare Naturalist diary entries.

- **Atomic snapshot commit with `tick_version` increment**: The rationale is canonical state consistency after each tick.

- **Presence accounting conjunction**: The rationale is to count only real visible, focused, recently active presence and clamp drift against clock drift or burst pings.

- **Drift calibration thresholds**: The rationale is that one week of regular use should be instrument-detectable and three weeks should be user-noticeable, avoiding both overnight change and months of apparent stasis.

- **Mood state machine**: The rationale is to model fast observable behavior from sun, interactions, absence, and social contagion while keeping birds expressive.

- **Generative motif grammar**: The rationale is species-specific vocalization without samples, with mood and weather shaping call rate.

- **Notebook trigger detection and sparsity filter**: The rationale is to emit observations only for noteworthy moments and keep them rare.

**6. Multi-Device Synchronization & Conflict Resolution**

- **Server-authoritative state invariant**: The rationale is to prevent stale mobile or laptop clients from overwriting each other and to eliminate "classic distributed system race condition" behavior.

- **Clients as render and input capture nodes only**: The rationale is that clients never submit absolute state, personality vectors, mood states, or clock variables, preserving canonical server simulation.

- **Additive server deltas from append-only event stream**: The rationale is ordered, coalesced mutation rather than Last-Write-Wins.

- **Concurrent device coalescing of overlapping presence**: The rationale is to prevent "double-counting of physical presence hours."

- **30-second visible polling cadence**: NOT RECOVERABLE FROM PLAN

- **Immediate re-sync on visibility return, online event, or OS sleep/wake frame gap**: The rationale is to recover from hidden tabs, network drops, and sleep/wake gaps with a fresh canonical snapshot.

- **Hermite coordinate and pose interpolation**: The rationale is that the client "does not snap" when state changes; it produces smooth hop or flight arcs over 1,200ms.

**7. Frontend Rendering Pipeline**

- **60 FPS on legacy hardware**: The plan states the visual pipeline is engineered for "60 FPS performance on legacy hardware" while preserving frame-0 mid-motion.

- **Fixed 1920 x 1080 virtual canvas with letterboxing/pillarboxing protection**: NOT RECOVERABLE FROM PLAN

- **Layered scene composition**: The layers organize top-bar chrome, particles, perch depths, parallax landscape, and sky. The plan does not articulate a single explicit rationale beyond rendering the fixed scene.

- **Top-bar chrome fades after cursor stillness**: NOT RECOVERABLE FROM PLAN

- **First Frame Mid-Action invariant**: The rationale is to avoid spinners/static splash screens and make birds appear "mid-breath" or "head tilted" on frame 0.

- **Quiet field fallback before snapshot**: The rationale is to keep ambient sky and leaf drift active during slow network snapshot arrival while never showing a loading spinner.

- **Micro-motion procedural engine**: The rationale is "idle aliveness" through procedural kinematics rather than repetitive sprite loops.

- **Reduced-motion cross-fade pose system**: The rationale is to remove continuous motion and particle drift while preserving gentle state changes through static poses and dissolves.

**8. Procedural Audio Pipeline**

- **WebAudio procedural syrinx graph**: The rationale is synthesized bird calls with no shipped samples and compact motif data.

- **Physical syrinx synthesis architecture**: The rationale is species-accurate chirps, trills, overtones, resonance, and breathiness without phase cancellation.

- **Spatial chorus and dynamic mix engine**: The rationale is to map calls to bird position and perch depth and blend multi-bird chorus cleanly.

- **Master compressor and limiter**: The rationale is to prevent clipping during multi-bird chorus.

- **Listen-In dynamic mix decay**: The rationale is intimacy and presence for the focused bird while maintaining background aliveness because other birds are "never silenced."

- **Graceful Silence Mode**: The rationale is to handle unsupported, blocked, or failed audio without violating the no recorded loops constraint.

**9. Accessibility Surfaces & UX Voice Architecture**

- **Accessibility as emotional, first-class design surface**: The rationale is to avoid treating accessibility as only a "compliance checklist."

- **Naturalist Voice**: The rationale is observational, lowercase, present-tense product experience specific to individual bird and moment, with no tech jargon, exclamations, or announcement framing.

- **Matter-of-Fact Voice**: The rationale is direct system communication for auth, errors, settings, sync, and revocation with functional instructions and "no false warmth."

- **Zero cross-contamination between registers**: The rationale is to keep aviary affect separate from infrastructure and system notices.

- **Screen-reader running prose live region**: The rationale is to preserve "affective aliveness" instead of mechanical state-label dumps.

- **Event-driven screen-reader narration interrupts**: The rationale is to narrate significant interactions such as Return-Greeting, Offer Reaction, and Settle Gesture immediately in Naturalist prose.

- **Procedural call captions**: The rationale is synchronized visual access to calls, derived from motif parameters and rendered near the calling bird.

- **Keyboard navigation and focus ring standards**: The rationale is complete keyboard operation and focus visibility across changing aviary lighting.

**10. Performance Budgets, Asset Optimization & Observability**

- **Initial JS bundle under 2 MB**: The rationale is small static delivery; CI fails pull requests exceeding the budget threshold.

- **Time to First Bird under 500ms**: The rationale is immediate visible aviary arrival on mid-tier mobile using edge snapshot inlining and zero blocking asset downloads.

- **60 FPS sustained over 30 minutes**: The rationale is long-session smoothness verified on a five-year-old reference CPU allocation.

- **Zero memory leak over 30 minutes**: The rationale is long-session stability, verified by heap snapshot delta testing.

- **Simulation tick p99 under 5 seconds**: The rationale is timely server-side canonical state advancement across partitions.

- **Zero external media requests**: The rationale is faster first render and smaller bundle: no audio downloads or large raster sprite sheets.

- **Edge-injected snapshot**: The rationale is eliminating a secondary roundtrip for initial state.

- **Aggressive code-splitting**: The rationale is to defer account settings, accessibility modals, visit management, and export UI until needed.

- **Allowed aggregate telemetry**: The rationale is operational monitoring of edge rates, Core Web Vitals, TTFBird, FPS, audio initialization, and tick workers.

- **Strictly prohibited telemetry**: The rationale is privacy: no per-account, per-bird, interaction, name, offer, notebook, or presence data in analytics, and simulation data is air-gapped.

**11. Rollout, Aviary Growth & Species Pool**

- **Six species with distinct silhouettes and vocal traits**: The rationale is recognizability through body shape, perch tendencies, and vocal motifs. The plan does not articulate why exactly these six species were chosen.

- **Pip / Warbler**: NOT RECOVERABLE FROM PLAN

- **Wren / Cactus or House Wren**: NOT RECOVERABLE FROM PLAN

- **Chickadee**: NOT RECOVERABLE FROM PLAN

- **Nuthatch**: NOT RECOVERABLE FROM PLAN

- **Finch**: NOT RECOVERABLE FROM PLAN

- **Nightjar**: NOT RECOVERABLE FROM PLAN

- **Bird arrivals governed entirely by calendar age**: The rationale is to avoid "interaction requirements, attendance streaks, or adoption fees."

- **Day 0 starter pair, user names them**: NOT RECOVERABLE FROM PLAN

- **Bird 3 at Day 60, Bird 4 at Day 150, Bird 5 at Day 270, Bird 6 at Day 365, Bird 7 at Day 500**: NOT RECOVERABLE FROM PLAN

- **Seven-bird hard ceiling**: The rationale is that audio recognizability breaks down above seven, and the cap preserves "intimate, individual bird recognition by ear."

- **Milestone 1: Simulation & Audio Test Harness**: The rationale is to validate drift calibration over 12 virtual weeks before full product implementation.

- **Milestone 2: Client Canvas & Interaction Core**: The rationale is to build scene, viewport scaling, micro-motion, presence conjunction, and listen-in curves after core simulation/audio.

- **Milestone 3: Auth, Sync & Notebook**: The rationale is to ship identity, multi-device canonical state, and naturalist notebook once interaction core exists.

- **Milestone 4: Accessibility, Social & Security Audit**: The rationale is to complete live regions, reduced motion, visit links, and contrast verification before launch.

- **Milestone 5: Staged Production Launch**: The rationale is to validate real-world continuous presence, tick worker scaling, multi-device sync integrity, rate-limiting, and operational monitoring before public availability.

**12. Technical Risk Matrix & Mitigation Strategies**

- **Drift Calibration controls**: The rationale is to keep birds from changing overnight like a clicker game or appearing static like a screensaver.

- **Sync Race Condition controls**: The rationale is to avoid divergent aviary states and lost drift history across laptop/mobile transitions.

- **Audio Fatigue & Uncanniness controls**: The rationale is to prevent harsh, robotic, grating calls that would make users mute audio and break "the primary affective relationship."

- **Presence Spoofing & Background Leak controls**: The rationale is to avoid unintended rapid personality drift without actual attention.

- **Accessibility Regression controls**: The rationale is to keep screen-reader users included in the affective aliveness of the aviary.

- **PII Contamination controls**: The rationale is to prevent email leakage into telemetry or partition logs and uphold privacy commitments.

**13. Verification Checklist for Execution Teams**

- **Bundle, TTFBird, no audio files, vector exposure, monotonic drift, Settle/tab-close equivalence, Naturalist live region, reduced-motion cross-fades, visit expiry/revocation, and no gamification verification**: The checklist exists so execution teams do not declare v1 ready until the plan's performance, privacy, simulation, accessibility, social, and anti-gamification commitments are concretely verified.
