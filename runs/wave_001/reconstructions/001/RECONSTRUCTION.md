## System-level intent

- Ambient, observational relationship: The plan's mission says Pocket Aviary is "ambient" and built around "low-key, observational relationships" with procedural birds. This shows up in the core premise that "idle attention is real interaction," in Presence as a core interaction, and in the sparse Field Notebook and naturalist narration surfaces.

- Long-term continuity without punishment: The plan says birds' "long-term personalities drift across weeks of presence" while short-term moods reflect "daily rhythms and subtle session events." The drift section reinforces this with "monotonic asymmetry," "traits never decay," and a two-week absence where "Birds return with their exact previously accumulated personality traits intact."

- Quiet anti-gamification: The plan repeatedly rejects game pressure through "No Gamification," "No Tamagotchi Mechanics," "No Push Notifications," and "No Announcement UI." It specifically bans "streaks, scores, levels, badges, XP," and says neglect creates "ambient quietness rather than negative drift or punishment."

- Server-authored canonical reality: The plan carries a strict "No Client-Authoritative Drift / No Last-Write-Wins" principle. It appears again in the append-only event log, the 60-second server-side tick, and "Server is the Sole Canonical Writer," with clients acting as "pure projection engines" that submit events.

- Product voice split between naturalist and system surfaces: The plan establishes a "Naturalist Register" for Aviary, Notebook, Narration, Captions, and Offer Prompts: "lowercase by default, present-tense, bird-named, specific, observational." It separates this from a "Matter-of-Fact Register" for auth, settings, errors, accessibility toggles, and sync conflicts: "clear, concise, direct, helpful."

- Privacy-preserving isolation: Privacy is not treated as a reporting afterthought. It appears in "Synthetic AccountID," encrypted email, blind indexes, aggregate-only operational telemetry, and the observability "STRICT DATA ISOLATION BOUNDARY" forbidding account IDs, bird names, personality vectors, presence durations, notebook contents, and interaction history in telemetry.

- Lightweight procedural browser experience: The plan stays web-only, uses a "single horizontal responsive 2D scene," an "ultra-lightweight 2D Canvas rendering context," real-time WebAudio synthesis, and a strict "<2MB" bundle budget. It explicitly avoids a heavy game engine and recorded audio loops.

- Accessibility as a parallel experience, not a toggle-off: Accessibility appears as screen-reader naturalist narration, real-time call captions, keyboard navigation, WCAG AA contrast, and a dedicated "reduced-motion cross-fade mode (not just disabled animations)." The plan treats accessible output as another way to observe the aviary.

## Per-feature whys

### Executive Summary & Product Scope

- Bird capacity and progressive unlock: The plan gives the why as maintaining "auditory recognizability and deep emotional attachment." Capacity starts with 2 birds and unlocks to a hard cap of 7 "strictly based on aviary age (not visit count or payment)."

- Web-only platform: NOT RECOVERABLE FROM PLAN

- Passwordless email magic links: The plan articulates the operational shape as "15-minute expiration" and "immediate invalidation on use," with one-time token verification and secure session cookie issuance. It does not add a broader product rationale beyond that security-oriented behavior.

- Single horizontal responsive 2D scene: NOT RECOVERABLE FROM PLAN

- Presence: The rationale is the core premise that "idle attention is real interaction" and the calibration goal "to prevent corrupted drift signals from background tabs or unattended machines." Presence requires visible tab, focused window, and recent pointer/key activity.

- Return-Greeting: The greeting exists so birds "notice the user's presence"; it is varied by boldness, mood, and absence duration, making the first second of return reflect personality and recent absence.

- Listen-in: The plan frames Listen-in as "audio focusing on a single bird" while later preserving "ambient presence" by never fully muting non-focused birds.

- Offer: The plan makes Offer a "gestural gift" that affects short-term mood and curiosity/boldness. It is also a trigger for the Curious mood, where the bird "perches forward" and "tilts head toward sound origin."

- Settle: The plan's why is a "user-initiated soft session-end" that shifts the scene to evening and quiets audio, with a "5-second undo affordance."

- Field Notebook: The notebook is "auto-generated, sparse, read-only" and uses "naturalist observation" on "notable simulation milestones," avoiding a user-authored journal or achievement feed.

- Quiet social invitations: The plan allows "one-to-one email invitations" for "read-only, non-co-present ambient observation" while recording "No presence or drift" from visitors. Revocability and read-only access preserve the quiet social boundary.

- Accessibility package: The plan ties accessibility to observation: screen-reader "naturalist running narration," real-time call captions, reduced-motion cross-fade mode, keyboard navigation, and WCAG AA contrast.

- No gamification: The why is an "absolute prohibition" on extrinsic progress surfaces such as "streaks, scores, levels, badges, XP," counters, calendars, and graphs.

- No Tamagotchi mechanics: The plan's rationale is non-punitive care: birds "never die, starve, fall ill, or show distress," and neglect means "ambient quietness rather than negative drift or punishment."

- No social network surfaces: The plan prevents public or competitive social dynamics by banning "public directories, discovery feeds, profiles, follows, shared aviaries, co-presence cursors/avatars, comments, or leaderboards."

- No push notifications: The rationale is to avoid "external pings, reminders, web push, or email digests urging visits."

- No announcement UI: The rationale is to avoid loud arrival framing such as "'Welcome back!' toasts, achievement modals, or arrival banners."

- No client-authoritative drift / no last-write-wins: The plan's why is state integrity: clients "never write personality state directly" or submit absolute traits, and conflicting sessions append to one event stream instead of overwriting state.

- Naturalist Register: The why is an observational product voice: lowercase, present-tense, bird-named, specific, no exclamation marks, no gamified terminology, and no second-person demands.

- Matter-of-Fact Register: The why is clear system communication for auth, session timeout, sync conflicts, account settings, accessibility toggles, and errors, "devoid of faux warmth or naturalist disguise."

### System Architecture & Component Boundaries

- Edge API Gateway: The plan gives it responsibility for TLS, session verification, rate limiting, request routing, synthetic account UUID validation, and serving an SSR snapshot for a "<500ms" first frame.

- Simulation Engine Worker Service: The service exists to execute the 60-second server-side tick, process interaction events, compute low-pass personality drift, update mood and weather, write canonical snapshots, and generate field notebook entries.

- Primary Relational Store: Its role is normalized persistence for accounts, birds, personality vectors, current moods, field notebook entries, and visit invitations.

- Append-Only Event Log: The plan names it the "sole input stream for simulation tick delta calculations," storing raw interaction and presence events.

- Operational Telemetry Service: The why is aggregate-only operations insight, "strictly isolated from any account ID or bird state."

### Data Model & Storage Schema

- Accounts table: The plan grounds this in synthetic account IDs, PII encrypted at rest, blind-index lookup, timezone, soft deletion, accessibility settings, and visit notification settings defaulted off.

- Magic links table: The rationale is one-time link handling through token hashes, expiry, and consumed timestamps tied to an account.

- Active sessions table: The rationale is device-level session visibility and revocation, with device names, user agents, and last active timestamps.

- Aviaries table: The plan uses it for lifecycle and canonical snapshot control: created time, settled state, weather state, last tick time, and monotonic version.

- Birds table: The rationale is stable bird identity plus hidden personality vectors normalized to [0.0, 1.0], short-term mood, offer timing, and perch zone placement.

- Interaction events table: The why is append-only ingestion for presence, listen-in, offer, settle, and unsettle events, with unconsumed indexing for simulation ticks.

- Field notebook entries table: The rationale is storing sparse generated observations tied to aviaries and, when applicable, birds.

- Visit invites and visit logs: The plan supports quiet social viewing through expiring, revocable invite tokens and a host-visible visit log with masked visitor email.

### API Surface & Wire Protocols

- Authentication and account lifecycle endpoints: NOT RECOVERABLE FROM PLAN

- Aviary state endpoint: The why is to fetch the "current canonical snapshot," including version, server time, local offset, settled state, weather, and bird state.

- Aviary stream endpoint: The rationale is pushing "state delta updates and notebook additions every minute or upon notable state mutation."

- Client interaction ingestion endpoint: The rationale is appending an interaction batch to the "server-side event log" rather than letting the client write state.

- Visit endpoints: The plan's why is host-managed quiet social access: create invite, list/revoke invites, stream a read-only snapshot, and reject all visitor interaction event submissions.

### Simulation Engine & Mathematical Models

- 60-second server-side simulation tick: The plan runs this "irrespective of client connection status" so events, drift, moods, day/night, weather, eligibility, notebook heuristics, and broadcasts are server-driven.

- Presence verification and calibration: The plan explicitly says the purpose is "to prevent corrupted drift signals from background tabs or unattended machines," with client batching and server clamping.

- Personality vector drift: The why is gradual expressiveness over weeks of presence, using a low-pass filter toward maximum expressiveness while enforcing "traits never decay" and zero change during absence.

- Drift calibration targets: The rationale is to make week-one change "measurable in backend instrumentation" but subtle, three-week change "visibly and audibly noticeable," and two-week absence exactly flat.

- Fast-timescale mood state machine: The plan uses it so mood reflects "time of day, weather, and recent interaction events" across Wary, Content, Curious, Drowsy, and Alert states.

- Procedural call-grammar runtime: The rationale is species-specific motif grammar shaped by `vocal_frequency` and `curiosity`, including frequency envelopes, glissando curves, timbre harmonics, and syllable timing.

- Field Notebook generator heuristics: The plan's why is sparse naturalist observation, targeting "~1 entry per 2-4 days" on notable milestones such as greeting precedence, perch transition, weather interaction, and drowsy synchrony.

### Sync Model & Conflict Avoidance

- Strict single-writer architecture: The rationale is conflict avoidance across devices: clients submit intent and sensor data, while the server processes a unified account stream in timestamp order.

- Deduplication: The why is handling "replayed requests from flakey mobile networks" through idempotency UUIDs at ingestion.

- Visibility change handling: The rationale is clean resume from hidden to visible by fetching a fresh snapshot, resynchronizing scene state, and opening the SSE stream.

- Connection loss and resume: The plan's why is to detect drops, back off, and resume without visual disruption when the version is unchanged.

- Tab sleep / laptop lid close handling: The rationale is to avoid client-side catch-up after a time jump; the client stops the render clock and pulls a fresh server snapshot.

### Frontend Rendering Pipeline

- HTML5 Canvas scene layers: The rationale is a layered ambient scene: sky, foliage, perch zones, particles, and weather composite, with top-bar chrome fading after cursor inactivity.

- Canvas / WebGL engine with zero heavy game engine dependencies: The plan connects this to the lightweight browser goal and bundle budget.

- Three perch zones: The rationale is spatial depth and focus through z-index, scale, opacity, desaturation, natural lighting, and foreground feather detail.

- Smooth interpolation for perch changes: NOT RECOVERABLE FROM PLAN

- Idle micro-motion generators: The rationale is continuous aliveness and preventing "visual stagnation" through breathing, head scanning, preening, tail wag, and foot reset.

- Day/night and atmospheric shaders: The plan uses local solar time and settled state to create dawn, midday, dusk, and night palettes, with night behavior differing by species.

- Reduced-motion cross-fade mode: The why is a dedicated mode "not just disabled animations," replacing flight paths and limb motion with alpha cross-fades and static pose changes.

- Loading sequence and empty-state handling: The plan rejects "Spinners or Loading Bars," aims for an immediate tranquil sky and wind audio, avoids "pop" entry animations, and lets starter birds arrive organically.

### Audio Pipeline & Procedural Soundscape

- Procedural WebAudio synth architecture: The plan explicitly says the why is to adhere to the "<2MB" bundle budget and "eliminate canned audio loops."

- Dynamic chorus mixing and natural staggering: The rationale is avoiding "phase cancellation and synthetic unison artifacts," while limiting density to keep foreground calls clear.

- Listen-in focus dynamics: The why is focused attention without erasing the aviary: focused bird gain rises, non-focused birds soften but are "never fully muted," preserving "ambient presence."

- WebAudio fallback: The plan's rationale is graceful silent mode with automatic captions and "No recorded audio fallbacks," preventing bundle bloat and maintaining auditory fidelity.

### Accessibility Architecture

- Screen-reader narration engine: The rationale is continuous scene narration in the "naturalist field-notebook voice," with idle cadence and immediate descriptive updates for user actions.

- Real-time call captions: The why is to describe procedural calls from the audio grammar motifs as accessible, non-obtrusive text near the calling bird.

- Keyboard navigation and focus manager: The rationale is full keyboard operation across controls and birds, with a high-contrast outline ensuring WCAG AA contrast across dawn and midnight palettes.

### Performance Budgets, Optimization & Observability

- Initial JS bundle budget: The plan's rationale is implementation discipline: zero game engine runtime, vanilla WebAudio synth, and code-splitting settings/notebook.

- Time-to-First-Bird budget: The why is a "<500 ms" first bird on 4G through inline critical CSS/SVG, edge-rendered SSR snapshot, and zero blocking asset waterfalls.

- Runtime framerate budget: The rationale is sustained "60 fps" using 2D Canvas dirty-rect rendering, offscreen buffering, and RAF throttling on idle.

- Memory leakage budget: The plan targets "0 MB" delta over 30 minutes through reusable WebAudio buffer pools, static object pools, and no closure leaks.

- Tick latency budget: The rationale is p99 tick latency under 5 seconds through Redis snapshot caching and partitioned batch processing.

- Privacy-preserving observability boundary: The why is strict separation between allowed operational telemetry and forbidden account, bird, personality, presence, notebook, and interaction data.

### Rollout Strategy, Verification & Growth Ramp

- Progressive bird unlock algorithm: The rationale is "auditory recognizability and deep emotional attachment," with age-based arrivals from Day 0 through Month 16+ and no visit-count or payment trigger.

- Simulation Drift Calibration Harness: The why is validating trait growth against the target curve and enforcing "zero negative drift on neglect."

- Audio Uncanny Valley & Psychoacoustic Audit: The rationale is verifying 7 concurrent procedural bird calls have zero clipping, zero phase distortion, and enough signal separation during listen-in.

- Memory & Long-Running Soak Suite: The why is heap stability during long active sessions.

- Accessibility Compliance Test: The rationale is automated contrast and ARIA live-region coverage across "all 24 solar cycle lighting states."

### Risk Analysis & Mitigation Matrix

- Drift calibration mitigation: The plan's why is avoiding automated-click exploitation and too-fast saturation through 3-way presence verification, 60-second clamping, and low-pass asymptotic scaling.

- Multi-device race mitigation: The rationale is preventing simultaneous actions from overwriting state through append-only events, canonical simulation ticks, and additive chronological deltas.

- Audio uncanniness mitigation: The why is avoiding robotic, screechy, or repetitive calls with multi-segment frequency modulation, micro-pitch jitter, and randomized syllable motifs.

- Screen-reader flooding mitigation: The rationale is preventing annoyance by throttling idle narration and prioritizing user actions with polite queuing and concise naturalist prose.

- PII / privacy leakage mitigation: The why is avoiding email leakage into telemetry or logs through synthetic UUIDs, AES-GCM email encryption, blind indexes, and CI linting against email keys.
