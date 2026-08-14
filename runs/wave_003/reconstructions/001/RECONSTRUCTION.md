## System-level intent

1. Slow observation is the core interaction model. The plan opens by defining Pocket Aviary as "ambient" and says it is designed to cultivate a "slow, observational relationship." The central product philosophy is explicit: "idle attention is real interaction." This shows up again in the rejection of "hunger meters, scores, streaks, and push notifications" and in calibration targets where visible change is "subtle to the user."

2. The aviary should feel alive before, during, and after visits. The server split "ensures the aviary continues living even when the user is away," and the frontend "Immediate First Frame" is meant to satisfy the principle that the aviary "has been continuing without the viewer." The first rendered frame should show birds "mid-preen, mid-hop, or mid-call" without an "entry fade or spinner."

3. Authority belongs to the server, while the client renders and reports. The plan repeatedly uses "server-side canonical simulation tick," "authoritative server-side simulation engine," "sole writer," and "clients are strictly read-only renderers." This principle appears in the architecture, simulation worker, sync model, and risk mitigation for "Sync Race Conditions."

4. Personality change is gentle, monotonic, and never punitive. The executive summary says birds undergo "monotonic personality drift toward expressiveness." The drift section states the invariant "Neglect does not decay T"; absence produces "Delta T = 0" rather than regression. This also supports the non-goals: "no bird death" and "no negative drift on absence."

5. The product boundary rejects gamification and custodial pressure. The plan explicitly excludes "streaks," "visit counters," "badges," "levels," "scores," "XP," "milestone toasts," "hunger," "feeding schedules," "health/happiness bars," illness, and death. This appears as both product philosophy and scope control.

6. Privacy is structural, not only policy text. The plan uses "synthetic UUID keys throughout all internal layers," encrypted email storage, HMAC email lookup "without decryption," "Zero Per-Bird Analytics Pipeline," "Telemetry Isolation," and a "Strict Privacy Firewall." It also hides personality vectors from UI, debug panels, tooltips, client payloads, analytics, warehouses, and machine-learning datasets.

7. Product language is split by surface. The "Naturalist Voice" is for the aviary, notebook, screen-reader narration, captions, and offers: "lowercase by default, present-tense, bird-named, observational, quiet." The "Matter-of-Fact Voice" is limited to auth, settings, conflicts, revocation, and accessibility menus: "direct, functional, zero simulated warmth."

8. Social sharing must remain quiet and non-invasive. The social feature is "Optional & Quiet Social Visits," using one-time read-only links with no co-presence and "no visitor drift impact." The non-goals reject public discovery, directories, leaderboards, feeds, avatars, chat, comments, visit badges, and counters.

9. Accessibility is part of the aviary voice, not a diagnostic layer. The plan calls accessibility "First-Class" and requires live-region narration in "naturalist voice," procedural call captions, reduced-motion rendering, WCAG AA contrast, and keyboard navigation. The risk matrix warns against screen readers receiving "raw state dumps" because that would make the accessible experience feel "like a debugging tool."

10. The sensory system should be procedural, lightweight, and non-canned. The plan chooses procedural WebAudio, procedural SVGs, canvas rendering, and no recorded audio fallback. This is tied to preserving the bundle budget, avoiding "canned, repetitive audio loops," and keeping "Time to First Bird Visible" under 500ms.

## Per-feature whys

### Scope & Boundary Definitions

- **Single-User Accounts**: The plan grounds this feature in secure and private access: magic links are 15-minute, single-use, rate-limited, and sessions are "revocable per-device." Internal layers use "synthetic UUID keys."

- **Aviary Lifecycle & Scaling**: Starting with 2 birds and capping at 7 supports the "small flock" premise. Unlocking birds "strictly by aviary age" keeps progression aligned with a slow relationship rather than scores, levels, or other gamified triggers.

- **Persistent 5-dimensional personality vector**: The vector enables "slow, multi-week character evolution" through drift that changes perch choice, greeting frequency, plumage saturation, and call complexity without exposing raw numbers.

- **Fast-timescale mood state machine**: Mood gives birds near-term state changes separate from long-term personality drift. The plan describes it as "fast-timescale" and modulated by time of day, weather, and personality traits.

- **Return-greeting engine**: NOT RECOVERABLE FROM PLAN

- **Presence Accounting**: The tripartite test exists to keep inactive background tabs from inflating attention. The risk matrix says inflated presence makes birds "drift too fast" and makes the relationship feel "artificial."

- **Listen-in**: The feature lets the user focus a bird by elevating its call while peer birds are "gently" ducked to ambient levels "without muting them." The mix section says reducing target reverb brings it "acoustically closer to the listener."

- **Offer Gestures**: Offers are articulated as drift inputs. The weighting matrix makes accepted seed, song, and water pool offers contribute different personality deltas, with cooldowns limiting repeated impact.

- **Settle Gesture**: The plan states its reason directly: it is a user-initiated evening shift that ends "presence cleanly."

- **5-second click-undo affordance**: NOT RECOVERABLE FROM PLAN

- **Naturalist Field Notebook**: The notebook preserves the authenticity of "a naturalist's field observations." Its sparse cadence, read-only log, and prose compiler enforce lowercase, present-tense, bird-named observations with "zero gamification nouns/verbs."

- **Optional & Quiet Social Visits**: Read-only, revocable visit links allow sharing without social-network mechanics. "No co-presence" and "no visitor drift impact" keep visitors from changing the host aviary.

- **First-Class Accessibility**: Accessibility is justified as an equal product surface: screen-reader narration uses the naturalist voice, captions mirror procedural calls, reduced-motion uses cross-fades, and keyboard navigation covers the aviary.

- **Procedural WebAudio Synthesis**: Runtime synthesis keeps calls dynamic and avoids shipped recordings. The fallback section says no recorded fallback is shipped to preserve bundle budget and avoid "canned, repetitive audio loops."

- **No Numerical Personality Exposure**: The plan hides personality numbers from UI, debug panels, tooltips, and payloads so users see derived behavior and rendering rather than raw vector metrics.

- **Naturalist Voice**: This voice keeps the product surface "observational" and "quiet," with lowercase present-tense prose and no exclamation marks or gamification jargon.

- **Matter-of-Fact Voice**: This voice is reserved for system surfaces so auth errors, settings, conflicts, and revocation remain "direct" and "functional" with "zero simulated warmth."

### System Architecture & Component Topography

- **Auth & Account Service**: It exists to issue, rate-limit, and verify magic links; create immutable synthetic account IDs; store encrypted email; manage revocable sessions; and handle JSON export plus a 30-day soft-deletion lifecycle.

- **Aviary API & Ingestion Service**: It serves current snapshots and ingests client events into an asynchronous queue, separating rendering reads from append-only interaction writes.

- **Authoritative Simulation Worker Engine**: It centralizes long-term mutation by ticking accounts, dequeuing events, calculating personality deltas, updating mood, evaluating notebook triggers, and publishing canonical snapshot deltas.

- **Data Isolation & Storage Layer**: PostgreSQL provides canonical storage, Redis holds un-ticked interaction logs, and telemetry routing structurally excludes per-bird vectors and interaction logs from analytics.

### Canonical Data Model & Schema Specifications

- **Encrypted email plus email hash**: The account model uses encrypted email and an HMAC-SHA256 `email_hash` explicitly "for lookup without decryption."

- **Revocable per-device sessions**: The sessions table tracks token hashes, user agent, activity time, and `revoked_at`, matching the plan's revocable per-device session boundary.

- **Canonical aviary record**: The aviary table stores the one account-linked aviary, settlement, weather, and `last_tick_at`, supporting the canonical simulation state.

- **Bird records**: Bird rows carry species, name, perch zone, current mood, offer cooldown, and last call so rendering and simulation can show individual birds rather than anonymous state.

- **Server-authoritative personality vectors**: The vector table is separate and bounded from 0.0 to 1.0 because these values are "Strictly Hidden from Client UI" and mutated by the authoritative drift system.

- **Append-only interaction events**: Events are stored with client timestamps, payloads, and processing state because the simulation tick consumes them after ingestion.

- **Field notebook entries**: Entries store prose text and observation context, matching the read-only observation log and sparse notebook generator.

- **Visit invitations and visit logs**: Invitations carry secure token hashes, expiry, revocation, and encrypted recipient email; logs keep masked visitor identity and duration for the settings visit log.

### API Surface & Inter-Service Protocol Contracts

- **POST /api/v1/auth/magic-link**: Its purpose is to request a sign-in magic link, with rate limits protecting IP and email request volume.

- **POST /api/v1/auth/verify**: Its purpose is to exchange a magic token for a secure bearer session and settings payload. Its error copy follows the matter-of-fact tone.

- **GET /api/v1/aviary/snapshot**: Its purpose is to fetch the authoritative rendering snapshot. The response includes derived rendering factors but excludes numerical personality vectors.

- **POST /api/v1/aviary/events**: Its purpose is to append batched presence and discrete interaction events and return queued acceptance rather than immediate mutation.

- **GET /api/v1/notebook**: Its purpose is to expose notebook entries as generated observation prose with pagination.

- **POST /api/v1/visits/invite and GET /api/v1/visits/stream/:token**: The invite endpoint creates secure visit URLs; the stream endpoint validates the visitor token and serves read-only snapshots while rejecting interaction submissions with `403 Forbidden`.

### Simulation Engine & Mathematical Drift Architecture

- **60s server simulation tick loop**: The loop exists to read unprocessed events, calculate presence and interaction weights, update personality, step mood, update idle behavior, evaluate notebook triggers, save canonical snapshots, and push cache updates.

- **Low-pass discrete accumulator with monotonic clamping**: This formulation makes personality drift slow, additive, capped per tick, and tied to weighted interaction signals.

- **Weighting matrix**: The matrix gives different interaction signals different trait effects, allowing presence, listen-in, seed, song, and water pool offers to shape boldness, social warmth, vocal frequency, plumage, and curiosity differently.

- **Monotonicity invariant**: The invariant exists so neglect never decays personality. Absence leaves personality at its "current expressive level" while mood relaxes into an "ambient, quiet state."

- **Calibration targets**: The targets define pace: one week of daily attention should be detectable by harnesses but "subtle to the user"; three weeks should become "visibly apparent."

- **Mood Transition Engine**: The Markov mood system gives fast-timescale behavior changes based on day phase, weather, and traits. Each mood maps to visible or audible behavior: wary birds prefer back perches, content birds preen and join chorus, curious birds move forward, drowsy birds lower vocalization, and alert birds become responsive.

- **Naturalist Field Notebook Generator**: Sparsity prevents overproduction, the context engine looks for relative event patterns, and the voice enforcer keeps entries lowercase, present-tense, bird-named, and free of gamification language.

### Multi-Device Synchronization & Conflict Prevention Model

- **Single-Writer Authority**: The server tick is the "sole writer" so multi-device actions do not corrupt or overwrite drift.

- **Append-Only Ingestion & Idempotency**: Client UUIDs and Redis `HSETNX` duplicate rejection make network retries safe. Commutative event deltas let listen-in and offers from different devices process sequentially in the next tick.

- **Client Interpolation & State Blending**: Interpolation prevents birds from snapping or teleporting when a new snapshot arrives; positions, plumage saturation, and mood poses blend over 1.2 seconds.

### Frontend Rendering Pipeline & Visual Scene Architecture

- **Layered Canvas / SVG Scene Composition**: The plan uses a visual canvas plus DOM overlay so sky, weather, foliage, birds, perches, narration, captions, chrome, and keyboard hit-boxes can coexist with semantic accessibility.

- **Top Bar Chrome auto-fade**: NOT RECOVERABLE FROM PLAN

- **Immediate First Frame ("Already in Motion")**: The feature exists to show that the aviary has been continuing without the viewer. Snapshot preload and phase offsets avoid spinners and start birds mid-action.

- **Idle Micro-Motion Systems**: Breathing, head tilts, and preening create the "fluid visual scenes" and quiet ongoing bird behavior implied by the ambient aviary.

- **Reduced-Motion Rendering Mode**: Reduced motion replaces continuous skeletal transforms, wing flutters, flight trajectories, drifting leaves, and parallax with slow cross-fades or removal, matching user or system accessibility preferences.

### WebAudio Procedural Synthesis & Soundscape Engine

- **Procedural Call Grammar**: The grammar synthesizes calls dynamically from motifs, FM modulation, envelopes, filters, gain, panning, and per-bird seeds instead of loops.

- **Variation Engine**: Pitch jitter, micro-timing, and harmonic emphasis make calls vary by `vocal_frequency` and mood.

- **Chorus Assembly**: Staggering calls by 180ms to 450ms creates "realistic counterpoint rather than phase-canceling overlap."

- **Listen-In Mix Architecture**: Target gain, peer ducking, and reduced target reverb make the focused bird closer while keeping the ambient mix intact.

- **Graceful Silence Mode**: If WebAudio fails, the product moves smoothly into silence and auto-enables captions. No recorded fallback is shipped to preserve bundle budget and prevent repetitive audio.

### Accessibility Architecture & Inclusive Design Surfaces

- **Screen-Reader Narration Engine**: The live region updates every 30-60 seconds in slow observational prose, and user-triggered events are framed as observations rather than system status updates.

- **Procedural Call Captions**: Captions match the exact procedural motif played and appear near the calling bird, making the generated soundscape available visually.

- **Keyboard Interaction & Focus System**: Roving tabindex, arrow navigation, and key bindings provide full keyboard access to listen-in, offers, settle, notebook, and overlays.

- **High-Contrast Focus Indicator**: The dual-tone 3px outline is justified by WCAG AA visibility against both daylight and night scene backgrounds.

### Performance Budgets, Resource Constraints & Observability

- **Hard Performance Budgets**: Bundle size, first bird visibility, 60fps idle rendering, memory stability, and server tick latency are specified to protect the first visible bird, long idle sessions, and scalable ticks.

- **Permitted Operational Telemetry**: Aggregated request rates, tick latency histograms, WebAudio errors, bundle load times, and anonymized session duration buckets are allowed for operations.

- **Strict Privacy Firewall**: Bird names, personality vectors, individual presence logs, and per-account interaction histories are excluded from external telemetry, analytics warehouses, and machine-learning datasets.

### Rollout Strategy, Calibration Framework & Progression Pacing

- **Phased Implementation Milestones**: NOT RECOVERABLE FROM PLAN

- **Flock Progression Pacing**: The month-based schedule supports the earlier lifecycle rule that birds are unlocked strictly by aviary age until the 7-bird flock capacity is reached.
