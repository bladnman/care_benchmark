## System-level intent

1. Pocket Aviary is meant to be a "tranquil" observational relationship, not a gamified application. This shows up in the Executive Summary phrase "observational relationship rather than a gamified application," in the non-goals forbidding streaks, XP, achievements, counters, and retention mechanics, and in the Conclusion's goal of an experience that feels "alive, quiet, and lasting."

2. The plan protects "emotional integrity" through strict non-goals. The Out-of-Scope section says prohibited features are excluded "to preserve the emotional integrity and core thesis of Pocket Aviary." That intent appears again in the risk table under "Drift Runaway / Tamagotchi Effect" and "Accidental Streak Gamification."

3. The relationship should deepen slowly through "presence and gentle interaction." The Executive Summary says hidden personality traits drift "over weeks of presence and gentle interaction," while the drift calibration targets make daily changes "completely imperceptible" and only perceptible after "3 weeks of regular visits."

4. Absence must never become punishment. The non-goals say birds cannot starve, sicken, or die, and "neglect never causes distress, sadness, or negative personality regression." The drift section repeats that traits "never decrease due to absence or neglect" and that an unvisited bird "does not revert or regress."

5. The aviary should feel as if it has been continuing without the user. This appears in the server-side simulation tick "advancing canonical state independently of client connections" and in the Zero-Spinner Instant Start Architecture, whose stated reason is "to uphold the core principle that the aviary has been continuing without the user."

6. The system is server-canonical and client-projected. The architecture states "Client is a Projection," "Server is Canonical," and "Append-Only Interactions." The multi-device section "completely rejects client-side simulation, distributed client clocks, and client-side last-write-wins."

7. Privacy is designed as an airgap, not an afterthought. The plan requires synthetic `account_id` use, stores email only encrypted in `accounts`, prohibits most services from inspecting email addresses, and bars operational telemetry from accepting interaction events, personalities, or notebook entries.

8. Product voice is split into two mutually exclusive registers. The Naturalist Voice is "strictly lowercase, present-tense, bird-named, observational, evocative," while the Matter-of-Fact Voice is "concise, clear, helpful, neutral" for authentication, account, sync, error, and settings surfaces.

9. Accessibility is a primary aesthetic surface. The in-scope list calls accessibility "first-class," the accessibility section says screen-reader users get a "fully realized naturalist accompaniment rather than mechanical state change announcements," and reduced motion is described as an "intentional, high-craft alternate aesthetic."

10. Focus should not break the ecosystem. Listen-in raises one bird and lowers ambient birds "without full mute"; the audio section says background birds remain audible "to preserve the feeling of an unbroken ecosystem"; chorus scheduling avoids unnatural synchronized calls.

11. The experience is web-native, immediate, and performance-disciplined. The plan specifies modern evergreen browsers, a lean bundle under 2MB gzipped, first bird visible under 500ms, sustained 60fps, no loading spinners, procedural vector/audio assets, and no native applications.

## Per-feature whys

### Executive Summary & Scope Boundary

- Single-User Accounts & Auth: NOT RECOVERABLE FROM PLAN

- Single aviary per account: NOT RECOVERABLE FROM PLAN

- Bird Population Mechanics: The plan says population growth is capped and "paced strictly by aviary chronological age" so birds represent "relationships deepened over time rather than unlocked achievements." It also says no clicks, purchases, or engagement volume can accelerate the timeline.

- Naming flow at onboarding and via settings: NOT RECOVERABLE FROM PLAN

- Presence Accounting: The plan treats presence as a factual positive-attention signal, using the strict conjunction of visible document, focused window, and recent pointer or key activity. It later frames presence-time as the dominant drift weight and guards against drift inflation.

- Return-Greeting: The rationale is that "the birds notice the user." A greeting by one bird within 1-2 seconds of session return makes return visible and is modulated by absence duration, boldness, and mood rather than uniform playback.

- Listen-in: The feature lets the user focus a single bird while keeping ambient birds audible. The plan explicitly rejects "full mute" and later says this preserves "the feeling of an unbroken ecosystem."

- Offers: The plan frames seed, song fragment, and still pool as "gift affordances" whose reactions are shaped by curiosity and mood. The per-bird cooldown also mitigates "Drift Runaway / Tamagotchi Effect" by decoupling repeated clicks from leveling behavior.

- Settle Gesture: The plan describes it as a "soft session-end gesture" that transitions lighting to evening and quiets calls, with an undo grace window. Its stated role is to let the session end by settling the scene rather than using a retention or task-completion mechanic.

- Field Notebook: The plan's rationale is to record "unique moments" and "noteworthy naturalist moment" observations at sparse intervals, using lowercase present-tense naturalist prose instead of engagement feedback.

- Social Affordance (Visits): Visits give an opt-in ambient share while preserving the no-social-network thesis: read-only, revocable, no co-presence, no visitor interaction, and no presence or drift recorded from visits.

- Rendering Pipeline: The responsive single horizontal scene, perch zones, local-time day/night cycle, rare weather, and ambient particles support a browser aviary that stays visible and alive without requiring scrolling, panning, or loading affordances.

- Audio Pipeline: Procedural WebAudio call synthesis exists because the plan forbids recorded loops and later says static MP3/WAV samples would violate the bundle budget and destroy chorus dynamics through phase cancellation.

- Accessibility & Inclusion: Accessibility surfaces are "first-class," with screen-reader narration, reduced motion, dynamic captions, keyboard navigation, and WCAG AA contrast so the aviary remains a realized product surface across access modes.

- Sync & Simulation: Server-side simulation advances canonical state independently of connections, uses additive server-authored personality deltas, and provides multi-device read consistency so clients do not author state.

### Out-of-Scope

- No Native Applications: The plan places this under features prohibited "to preserve the emotional integrity and core thesis" and keeps Pocket Aviary web-only in desktop and mobile browser viewports.

- No Gamification Elements: The rationale is explicit: the system must never measure or reflect back user engagement habits, so it forbids streaks, visit counters, calendars, XP, achievements, badges, levels, and adoption counters.

- No Tamagotchi / Custodial Mechanics: The rationale is to avoid distress, sadness, death, hunger meters, chores, and negative personality regression. Neglect produces only ambient quietness.

- No Social Network Mechanics: The plan excludes public discovery feeds, profiles, comments, leaderboards, follower graphs, and real-time co-presence to keep visits from becoming a social network.

- No Push/Engagement Notifications: The rationale is that "the aviary exists only where and when the user opens the window," with no browser push, marketing emails, or retention reminders.

### Voice and Tone Separation Architecture

- Naturalist Voice (Product Surface): This voice is used where the aviary is experienced: visual scene, field notebook, screen-reader narration, and call captions. The plan's why is an observational, evocative, bird-named register with no exclamation marks, gamified terminology, or second-person directives.

- Matter-of-Fact Voice (System Surface): This voice is used for authentication, account management, sync conflicts, error boundaries, and accessibility settings so operational copy stays clear, helpful, neutral, and free of "false warmth or simulated naturalist charm."

### System Architecture & Component Topology

- Client Application (SPA): The client exists as a lean browser bundle responsible for real-time scene rendering, procedural audio, strict presence tracking, inputs, and accessible DOM projections; this supports the web-native, immediate aviary surface.

- Edge API Gateway: The gateway's rationale is operational safety around TLS, security headers, rate limiting, static asset caching, synthetic account UUID extraction, and PII scrubbing from operational logs.

- Auth Service: NOT RECOVERABLE FROM PLAN

- Aviary State & Event Service: The service ingests append-only client events and serves hydrated snapshots, matching the invariant that clients report factual interactions while the server interprets them.

- Simulation Engine Worker: The worker is the persistent heart of the aviary: a deterministic tick that advances mood, drift, age unlocks, and notebook entries as canonical server-authored state.

- Transactional Notification Service: Its rationale is separation: magic links, visit invites, and exports are processed as transactional email, "strictly fire-and-forget" and "completely isolated from analytics."

- Client is a Projection: The client never runs simulation or mutates personality or mood. Its role is to receive canonical snapshots, interpolate, and drive local micro-motions.

- Server is Canonical: All personality drifts, mood shifts, aviary unlocks, and field notebook entries are authored exclusively by the simulation worker to avoid client-side authority.

- Append-Only Interactions: Clients only report factual observations such as `presence_ping`, `listen_in_start`, `offer_made`, and `settle_triggered`; the server turns events into numerical deltas.

### Data Model & Storage Specifications

- Synthetic Account ID & PII Airgap: The rationale is strict conformance to account privacy: schemas, messaging, cache keys, partition tokens, and telemetry use synthetic UUIDs, while email is encrypted and isolated to `accounts`.

- Email hash lookup: The `email_hash` exists for lookup "without decryption," keeping email inspection out of ordinary database flows.

- Magic link storage: NOT RECOVERABLE FROM PLAN

- User session storage: NOT RECOVERABLE FROM PLAN

- Aviary timezone, settled, and weather fields: These fields support local-time lighting, settle state, and rare ambient weather as canonical aviary state.

- Bird visual and call seeds: NOT RECOVERABLE FROM PLAN

- Hidden Personality Vectors: The table is explicitly "Server Canonical Only, Never Exposed to Client," matching hidden traits that drift toward expressiveness without becoming user-facing numbers.

- Fast-Timescale Mood States: Mood is separated from personality because it operates on a faster timescale, from about 10 minutes to 24 hours, with current mood and intensity stored canonically.

- Append-Only Interaction Event Log: The event log stores factual events, client timestamps, and processing status so the server tick can interpret interactions without accepting client-authored trait values.

- Field Notebook Observations: The table stores server-generated observation prose, observed birds, and context so the notebook remains a historical naturalist log.

- Optional Social Visits: Invitation and visit-log tables support opt-in, expiring, revocable ambient visits while keeping the interaction surface read-only.

- Telemetry Complete Airgap: Operational metrics are prohibited from accepting interaction events, bird personalities, or notebook entries to keep lived aviary details out of telemetry.

- Metrics Collected: The plan limits metrics to request rate, latency, simulation tick duration, render FPS, WebAudio error counts, and heap distributions so observability stays operational.

- Excluded Telemetry: Account identifiers, email domains, bird count, traits, presence duration, and offer frequencies are excluded to enforce privacy guardrails.

### API Surface & Contract Specifications

- Authentication and account-management error copy: Errors use matter-of-fact text, aligning system endpoints with the Matter-of-Fact Voice.

- `POST /api/v1/auth/magic-link`: NOT RECOVERABLE FROM PLAN

- Magic-link rate limit: NOT RECOVERABLE FROM PLAN

- `POST /api/v1/auth/verify`: NOT RECOVERABLE FROM PLAN

- `POST /api/v1/auth/session/revoke`: NOT RECOVERABLE FROM PLAN

- `POST /api/v1/account/export`: NOT RECOVERABLE FROM PLAN

- `DELETE /api/v1/account`: NOT RECOVERABLE FROM PLAN

- `GET /api/v1/aviary/snapshot`: The snapshot exposes canonical render state while stripping personality vectors. Only visible attributes such as perch, mood enum, plumage saturation scalar, visual seed, and call seed are returned.

- `POST /api/v1/aviary/events`: The endpoint batches interaction events logged while the tab was active, matching the append-only model where clients submit observations and the server interprets them.

- `GET /api/v1/aviary/notebook`: NOT RECOVERABLE FROM PLAN

- `POST /api/v1/visits/invite`: The invite endpoint supports opt-in ambient visits with an expiration timestamp rather than public discovery.

- `DELETE /api/v1/visits/invite/:invite_id`: Revocation supports the plan's "revocable email magic link" visit model.

- `GET /api/v1/visits/view/:token`: The rationale is read-only ambient viewing: invalid or revoked links get matter-of-fact 404 text, actions are disabled, events are blocked, and visit duration is logged without presence or drift.

### Simulation Engine Design & Drift Dynamics

- Simulation engine tick: The tick is the persistent heart of Pocket Aviary, fetching events, calculating inputs, applying drift, evaluating mood, checking age milestones, generating notebook prose, and committing canonical state.

- Monotonic low-pass drift filter: Personality drift is "very slow" and based on positive attention signals. It moves monotonically toward expressive traits and never decreases due to absence or neglect.

- Asymptotic saturation term: The `(1.0 - P)` term creates "natural saturation" and prevents runaway explosion.

- Session-scale drift target: A one-day, 15-minute session produces an imperceptible delta, explicitly to eliminate a "Pavlovian feedback loop."

- One-week instrument measurability target: A week of daily visits must be detectable by automated test assertions and telemetry instruments while remaining subtle.

- Three-week user perception target: After regular visits, changes become visible in perch closeness, richer color, calls, and bolder offer response without a sudden jump.

- Neglect behavior: With no presence input, the delta is zero; birds stay at earned trait values and do not regress.

- Mood State Machine & Dynamic Contagion: Mood operates on a faster timescale than personality and lets birds respond to local time, weather, recent interactions, and neighboring birds.

- Mood state list: NOT RECOVERABLE FROM PLAN

- Local Time of Day transitions: Sun calculation based on user timezone shifts night toward `drowsy` and dawn toward `alert` and `content`, tying mood to the aviary's local-time cycle.

- Ambient Weather transitions: Rain increases `wary` or quiet `drowsy` probability and reduces vocalization rate, making rare weather change behavior and sound.

- Recent Interactions transitions: Accepted seed offers nudge mood to `content` or `curious`, while rapid unhandled UI movement nudges wary birds toward `wary`.

- Social Contagion: Neighboring birds can become `wary` after an alarm call, making mood contagious across adjacent perches.

- Bird Species Pool: The pool consists of six "distinct ornithological archetypes," each carrying a different call style, perch preference, boldness, curiosity, or time-of-day identity.

- Age-Based Adoption Pacing: Birds arrive by account age so new birds represent "relationships deepened over time rather than unlocked achievements"; clicks, purchases, and engagement cannot accelerate the timeline.

- Naturalist Field Notebook Generation Engine: The engine generates entries only when a "noteworthy naturalist moment" occurs, keeping the log observational rather than exhaustive.

- Field Notebook rate limit: The maximum of one entry per 48-72 hours, even under continuous high presence, keeps entries sparse.

- Field Notebook trigger heuristics: First greeter shifts, weather introspection, extended quietness, and perch shifts define "noteworthy naturalist moment" criteria from lived aviary behavior.

- Field Notebook style invariant: Lowercase, present-tense, bird-named prose with no exclamation marks keeps notebook entries in the Naturalist Voice.

### Multi-Device Sync & Conflict Prevention

- Single Canonical Server: The plan rejects client-side simulation, distributed client clocks, and client-side LWW conflict resolution for personality data so the simulation worker remains the sole writer.

- Clients Pull Snapshots: Snapshot pulls on page load, visibility return, refocus, and keepalive let each device refresh from canonical state rather than resolve conflicts locally.

- Clients Stream Append-Only Events: A device sends factual interaction events; another device later polls the newly calculated canonical state, preserving read consistency.

- No Absolute State from Client: Clients cannot send trait values, and attempts receive 400 Bad Request, preventing client-authored personality overwrites.

- Bounded Presence Deduplication: Concurrent open tabs are merged with an interval union algorithm so presence can never double-count or inflate drift speed.

- Clock Skew Mitigation: Client timestamps more than plus or minus 120 seconds from server UTC are normalized to `server_received_at`, protecting canonical timing from distributed client clocks.

### Frontend Rendering Pipeline

- Single `<canvas>` element: NOT RECOVERABLE FROM PLAN

- No Panning / No Scrolling: Narrow viewports use letterboxing or branch compression so all perches and active birds remain 100% visible at all times.

- Three Perch Zones: The front perch conveys "intimacy and boldness," the middle perch is the standard resting area, and the back perch is used by wary or drowsy birds.

- Zero-Spinner Instant Start Architecture: The explicit rationale is to uphold the principle that "the aviary has been continuing without the user."

- Inlined Initial Snapshot: Inlining the latest canonical snapshot lets the first frame render before network requests.

- First-Frame Mid-Action Paint: Birds begin mid-breath, mid-preen, or mid-scan on frame 1 so the aviary feels already in motion.

- Quiet Field Cold Fallback: On cache miss or cold connection, the engine renders a quiet sky gradient and soft breeze particle, preserving the no-spinner rule.

- Procedural Idle Micro-Motion: Eye and head movement, respiratory sway, preening, and weight shuffle make birds feel alive through client-side local micro-motion without client-side simulation authority.

- Reduced-Motion Mode: Reduced motion is treated as an "intentional, high-craft alternate aesthetic" rather than a lesser mode.

- Reduced-motion cross-fades and removed particles: Cross-fading still poses, dissolving perch changes, removing leaf and feather drift, and slower lighting transitions honor `prefers-reduced-motion: reduce`.

### Audio Pipeline & Procedural Syrinx Synthesis

- WebAudio procedural call synthesis: The plan rejects static MP3/WAV samples because they "violate the bundle budget" and destroy chorus dynamics through phase cancellation.

- Syrinx physical modeling: Dual FM oscillator pairs, formant filters, envelopes, motif grammars, stochastic pitch, duration, and micro-vibrato produce avian timbre and variation on each vocalization.

- Chorus Mixing & Natural Stagger: Stochastic staggering exists for "Avoidance of Synchronization," preventing unnatural unison calls unless a duet is intended.

- Spatial Positioning: Stereo panning maps each bird's audio to its horizontal perch coordinate, tying sound to visible position.

- Listen-In Mix Dynamics: The target bird rises while ambient birds attenuate, but the plan forbids hard cut or full mute to preserve the unbroken ecosystem.

- Autoplay Handling: Because browser policies block audio until a user gesture, the UI starts in graceful silence and resumes `AudioContext` silently on the first click or keypress.

- WebAudio Fallback: If WebAudio fails, the app enters silent mode with call captions automatically enabled, with no recorded audio fallback path.

### Accessibility Surfaces & Inclusive Design

- Screen-Reader Narration: The rationale is a "fully realized naturalist accompaniment rather than mechanical state change announcements."

- Screen-reader idle throttle: Idle narration every 30-45 seconds, using `aria-live="polite"`, prevents queue overload and avoids interrupting the user.

- Real-Time Call Captions: Captions serve hard-of-hearing users and users in quiet environments.

- Dynamic Caption Prose: Captions are derived from procedural synthesis parameters so they describe the actual call shape, such as a rising tone, trill, or alarm chip.

- Keyboard Navigation Model: The plan guarantees "Full functional parity" without mouse interaction.

- Contrast & Visual Hierarchy: WCAG AA contrast against morning and midnight backgrounds keeps top bar labels, notebook text, captions, and settings readable.

### Performance Budgets, Verification & Observability

- Initial JS Bundle budget: The 2MB gzipped ceiling supports the lean, browser-native aviary and is enforced through a bundle analyzer gate.

- Time to First Bird Visible budget: The under-500ms target ensures the first bird appears quickly on 4G mid-tier mobile, matching instant-start intent.

- Runtime Framerate budget: Sustained 60fps keeps the scene graph, birds, particles, and micro-motion smooth.

- Memory Heap Growth budget: Net-zero heap growth over 30 minutes guards against leaks during long ambient sessions.

- Simulation Tick Latency budget: A p99 below 5 seconds keeps server-authored state progression timely.

- Bundle Size Enforcement strategies: No heavyweight UI libraries, compact vector paths, and dynamic code-splitting for non-critical dialogs support the bundle ceiling.

- Time to First Bird strategies: Inline initial state and synchronous first-frame paint support the no-spinner, first-bird-visible budget.

- Zero Memory Leak strategies: Recycled WebAudio node pools and notebook DOM virtualization prevent allocation growth.

- Client RUM and Server Metrics: Observability covers page load timing, time to first bird, WebAudio failure, FPS degradation, tick duration, query latency, queue depth, and error rate so performance can be measured without aviary-content telemetry.

- Privacy Enforcement tests: CI asserts that telemetry does not contain `account_id`, `bird_id`, `name`, `personality_vectors`, or interaction history.

### Rollout, Testing & Calibration Strategy

- Sprint 1 through Sprint 5 phased milestones: NOT RECOVERABLE FROM PLAN

- One-week drift calibration harness: The harness asserts weekly trait increase and instrument-detectable change, matching the instrument measurability target.

- Three-week drift calibration harness: The harness asserts larger trait increases and forward perch migration, matching the user perception target.

- Six-month neglect calibration harness: The harness proves zero negative drift by asserting traits remain identical after long offline time.

- Saturation Guard: The guard asserts no trait exceeds 1.0000 under pathological click frequency, matching the saturation and runaway-prevention intent.
