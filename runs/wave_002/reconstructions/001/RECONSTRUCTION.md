## System-level intent

- Ambient life over game mechanics. The plan repeatedly rejects "gamification," "Tamagotchi mechanics," "scores," "green-dot calendars," "leaderboards," "death, hunger, distress, happiness meters," and social-network surfaces. The positive product posture is an aviary that "feels alive, not robotic" while favoring "Restraint over richness."

- Time-based, gentle growth instead of extraction or achievement. Bird growth is paced by "aviary age (not visit count or interaction score)," with "No paid tiers, no unlock mechanics beyond time." Drift is calibrated toward "visible drift at 3 weeks," not instant reward loops.

- The bird behavior is the welcome surface. The plan says, "No textual 'Welcome back!' toasts, banners, or modals -- ever. The bird greeting is the entire welcome surface." This intent also shows up in the return-greeting narration priority and the "aliveness regression suite" checking greeting variation and stagger.

- Server-side canonical state is the center of trust. The plan states that the simulation engine is the "sole writer" and that "there is exactly one canonical state per account." The client is a "state consumer + interaction producer, never a state author." This is intended to make sync "a property of the architecture rather than a feature."

- Additive, auditable event processing instead of direct client mutation. Interaction data is stored in an "append-only event log," events are consumed "in time order," and the client sends actions like "user listened in to Pip for 3 minutes" rather than personality or mood values. The plan uses this to avoid "last-write-wins," clock-skew ordering issues, and merge conflicts.

- Presence should mean actual attention, not an open tab. Presence requires `visibilityState === "visible"`, `document.hasFocus()`, and recent pointer/key activity. The drift function uses a "time-decayed accumulator" and decay that "prevents overnight-tab-open from accumulating drift."

- No punishment for absence. The plan says "Neglect produces no downward movement," traits move "monotonic toward expressive," tab close has "No penalty," and there is "No settle-required." This separates the product from Tamagotchi-style failure states.

- Naturalist prose is the product voice. The field notebook is "prose in naturalist voice"; screen-reader narration uses the "same naturalist prose voice"; captions are "lowercase, present-tense, specific." System failures are explicitly excluded from that voice: session timeout gets "No naturalist prose."

- Accessibility is part of the main experience, not an afterthought. Screen-reader narration, reduced-motion rendering, call captioning, WCAG AA contrast, keyboard navigation, semantic notebook markup, and launch-gate audits are all present in the plan. The accessibility risk mitigation says the audit is "a launch gate, not a post-launch task."

- Fast first life is central. The loading section says "No spinner. No fade-from-static," and the first frame with birds should show them "already in motion." The performance section treats "Time to first bird visible" as a core budget, and the bundle-size risk says losing that budget "kills the central conceit of the product."

- Privacy protects the user's relationship with the aviary. Email is encrypted and "never used as a key outside the account record"; raw personality vector values are not exposed; RUM is "Aggregate-only"; telemetry "never includes per-bird state, per-account interaction history, or any field that could reconstruct a user's relationship with their aviary."

- Quiet social boundaries. Visit invitations are read-only and ambient, with "No co-presence, no chat, no avatars, no comments. No public discovery." Friend-visited notification is off by default. This keeps visits from becoming a social network surface.

- Small signals carry aliveness. The plan warns that removing "a micro-motion," "a call variation," or "a greeting stagger" can cause "the death of the product." The "aliveness regression suite" turns those principles into acceptance criteria.

## Per-feature whys

### Scope

- Two starter birds per new account: The plan frames them as "the birds that arrived" and disallows "catalog" or "user species selection at adoption," so the rationale is to make the first experience feel like arrival rather than shopping or configuration.

- Up to seven birds per aviary: The plan gives no specific rationale for the cap itself beyond the v1 shape and the bird-per-aviary ramp. NOT RECOVERABLE FROM PLAN

- New bird-offer events based on aviary age: The rationale is to pace growth by time, "not visit count or interaction score," with "No paid tiers" and no unlocks "beyond time."

- Third bird at roughly 3 months, five or six across the first year, seventh at 450+ days: The plan says the ramp is "the product mechanic working as designed" and is "practically invisible" during early beta; it keeps growth slow and age-based.

- Single-user accounts: The plan explicitly excludes "shared aviaries" and "multi-aviary accounts," but it does not articulate a separate rationale for single-user accounts. NOT RECOVERABLE FROM PLAN

- Magic-link email sign-in: The plan says no passwords and no SSO in v1, but it does not articulate why magic link is the chosen sign-in method. NOT RECOVERABLE FROM PLAN

- Synthetic UUID account identifiers and encrypted email: The rationale is to avoid using email "as a key outside the account record" and keep account identity separate from the email address.

- Multi-device sync: The rationale is that both clients pull from the "same canonical record," making sync "a property of the architecture rather than a feature" with "no sync protocol, no merge conflicts, no CRDTs."

- Field notebook: The rationale is to sustain a "field-notebook illusion" through sparse, read-only, naturalist observations. Sparsity also "reduces repetition visibility."

- Field notebook cadence of roughly one entry every few days, sparser for active users and never per-session: The rationale is to avoid repetitive, obviously machine-written entries and preserve the notebook's naturalist feel.

- Presence accounting with the strict 3-condition check: The rationale is to make presence reflect actual attention and protect drift from false positives such as background tabs or overnight-open sessions.

- Visit invitations by email: The plan gives mechanics for host invites and visitor access, but not a distinct why for email as the invitation channel. NOT RECOVERABLE FROM PLAN

- Read-only visitor ambient view: The rationale is to allow visiting without turning the product into co-presence or social interaction; the plan explicitly excludes chat, avatars, comments, public discovery, and social-network surfaces.

- Visit invites expire after 30 days unused: The plan states the expiration behavior, but not a specific rationale. NOT RECOVERABLE FROM PLAN

- Visit invites revocable at any time: The rationale is account control over access to the aviary.

- Friend-visited notification opt-in and off by default: The rationale is to avoid default social notification behavior; the plan places it next to "No public discovery" and other quiet social boundaries.

- Screen-reader narration: The rationale is to provide the aviary experience as "naturalist prose" through a slow ARIA live-region cadence rather than only visual animation.

- Reduced-motion mode as cross-fade-based rendering: The rationale is to honor `prefers-reduced-motion` and a manual toggle while preserving a living visual state rather than simply turning the experience off.

- Call captioning: The rationale is to make procedural calls available as prose, and to support the designed WebAudio fallback of "graceful silence with captions on by default."

- WCAG AA contrast on user-copy text: The rationale is accessibility for all copy surfaces, with explicit launch-gate contrast audit.

- Keyboard navigation across all interactive surfaces: The rationale is that "every interactive surface" must be reachable and "every action executable."

- Account export as a JSON snapshot emailed: The plan states this feature but does not articulate its rationale. NOT RECOVERABLE FROM PLAN

- Account deletion with 30-day soft window then hard delete: The rationale is recoverability before final deletion; the API includes restore within the soft window and launch checks cover "soft delete, restore within 30 days, hard delete after 30 days."

### Architecture

- Three independently deployable services: The rationale is separation of responsibility: API Gateway/Auth for sign-in and account CRUD, Simulation Engine for server-side tick and state generation, Client SPA for rendering, WebAudio, interaction capture, and accessibility surfaces.

- Rust or Go for the Simulation Engine: The plan's rationale is that it is "CPU-bound" and "needs predictable latency."

- PostgreSQL for canonical state: The plan names the storage choice and entities, but does not give a separate why for PostgreSQL itself. NOT RECOVERABLE FROM PLAN

- Simulation Engine as the only writer of bird, personality, and mood state: The rationale is to protect canonical state and keep clients from writing personality or mood directly.

- API Gateway writes account records, session tokens, and interaction events: The rationale is authority separation: the Gateway is "the authority on auth and account state," while the Simulation Engine is "the authority on state snapshots."

- Append-only `interaction_events` table: The rationale is ordered event consumption and auditability; the tick consumes events "in time order" and drift history is also append-only.

- CDN-served SPA and edge reach: The rationale is TLS termination and "geographic reach" while the SPA is served from edge.

- Live, short-TTL snapshot pulls with no cached snapshots: The rationale is that each pull must be live because the snapshot is current aviary state.

- Client/server boundary where the server sends snapshots and the client interpolates: The rationale is to let the server own state while the client owns rendering and interaction capture.

- Client-side procedural call synthesis: The rationale is that no audio is streamed from the server; the client can synthesize from motif IDs and parameters.

- Scene compositor, bird renderer, and audio engine as separate render layers on a shared state bus: The rationale is decoupling; "the scene compositor doesn't call the audio engine; both read the same snapshot."

### Data model and snapshots

- `stable_id` on Bird: The rationale is explicitly that it is "immutable" and "survives rename, sync, migration."

- User-assigned, renameable bird names: The plan states the behavior but not a specific rationale. NOT RECOVERABLE FROM PLAN

- Hidden personality vector values: The rationale is that the user never sees raw traits; the snapshot sends derived pose, mood, saturation, and call data, "not raw traits."

- `drift_history` as an append-only log: The rationale is "audit."

- Notebook entries stored as already-rendered prose: The rationale is to preserve generated naturalist entries as written, with entries "written once" and "immutable."

- `InteractionEvent` with nullable `bird_id`: The plan explains it is nullable for settle and presence events, but gives no broader rationale. NOT RECOVERABLE FROM PLAN

- `consumed_by_tick` on interaction events: The rationale is to let the Simulation Engine consume new events once and mark them after writing canonical state.

- Session records with `token_hash`, `device_label`, and revocation: The plan states the data and revocation endpoints but does not articulate a separate rationale beyond session management. NOT RECOVERABLE FROM PLAN

- Visit logs with start, end, and duration: The plan provides a visit log endpoint but no specific rationale for log contents. NOT RECOVERABLE FROM PLAN

- Snapshot format with `notebook_recent` last five entries: The rationale is "quick display"; full notebook history loads lazily as a deferred chunk.

- Snapshot size target as "kilobytes, not megabytes": The rationale is performance and fast live delivery.

- Snapshot excludes personality vector values: The rationale is to avoid exposing values "to the user -- not in stats panels, debug views, or any surface."

### API surface

- Magic-link request and verify endpoints: The plan states the endpoints but does not articulate a distinct rationale beyond implementing magic-link sign-in. NOT RECOVERABLE FROM PLAN

- Session listing and specific session revocation endpoints: The rationale is revocation UI and account session control.

- Snapshot pull on tab-open, visibility-change after hidden, and keepalive while visible: The rationale is that clients see current canonical state and resume from fresh state after inactivity.

- Notebook pagination with `after`: The plan states pagination but does not articulate a separate rationale. NOT RECOVERABLE FROM PLAN

- Account export request endpoint emails the export: The plan states the behavior but not the reason for email delivery. NOT RECOVERABLE FROM PLAN

- Batched interaction events flushed every about five seconds or on unload via `sendBeacon`: The rationale is to reduce write chatter while avoiding lost events on page unload.

- Idempotency key per interaction event: The rationale is implied duplicate protection for event appends, but the plan does not explain it directly. NOT RECOVERABLE FROM PLAN

- Presence heartbeat separate from the event batch: The rationale is explicit: "to avoid presence data being lost in a batch flush."

- Visit token landing without auth: The plan says no auth is required, "just the token," but does not articulate why. NOT RECOVERABLE FROM PLAN

- Bird rename endpoint: The plan states that bird names are renameable but does not articulate a separate rationale. NOT RECOVERABLE FROM PLAN

- Bird offer endpoint as syntactic sugar over events: The rationale is convenience; it "also goes through the events endpoint" while offering a direct surface.

- Account restore endpoint: The rationale is to undo deletion within the soft window.

- Settings endpoint for notification preferences: The plan states notification preferences but does not articulate a distinct rationale. NOT RECOVERABLE FROM PLAN

### Simulation engine design

- Tick interval of about 60 seconds: The plan says it is configurable and calibrated; sync risk discussion treats a roughly 60-second delay as below the perceptible threshold for bird behavior changes.

- Tick runs whether or not a client is connected: The rationale is that the simulation continues server-side while tabs are hidden, closed, or devices suspended.

- Minimal work for accounts with no recent events and no active sessions: The rationale is scaling; dormant accounts only advance cheap timers and time-of-day transitions.

- Per-account independent ticks: The rationale is account isolation and independent simulation work.

- `event_offset` and ordered event reads: The rationale is strict event ordering for state updates.

- Presence-time delta with recency weighting: The rationale is low-pass filtering so recent presence matters more and stale presence does not keep accumulating.

- Drift formula `alpha * presence_minutes * trait_weight * decay_factor`: The rationale is calibrated, time-decayed personality drift.

- Social warmth and curiosity more presence-sensitive, plumage saturation slowest: The plan states the sensitivity choices but does not articulate separate rationale. NOT RECOVERABLE FROM PLAN

- Traits only move upward and neglect produces no downward movement: The rationale is to make change "monotonic toward expressive" and avoid punishing quiet birds.

- New birds seed at 0.3-0.5 species-dependent: The plan states the starting range but not a specific rationale. NOT RECOVERABLE FROM PLAN

- Seed offer adds curiosity: The plan states the mapping but not the product reason for seed specifically affecting curiosity. NOT RECOVERABLE FROM PLAN

- Song fragment offer adds vocal frequency: The plan states the mapping but not the product reason for song specifically affecting vocal frequency. NOT RECOVERABLE FROM PLAN

- Still pool offer has no direct trait effect but moves mood toward content: The plan states the mapping but not the product reason for still pool specifically affecting contentment. NOT RECOVERABLE FROM PLAN

- Time-of-day mood bias: The rationale is to make mood respond to local day phase.

- Weather mood bias: The rationale is to make rain and wind matter to bird behavior; rain dampens vocal frequency and wind biases toward alert or wary.

- Recent interactions affecting mood: The rationale is for offers, listen-in, and settle gestures to have visible behavioral consequences.

- Personality filter on mood transitions: The rationale is that high-boldness birds resist wary transitions and high-curiosity birds tend toward curious more often, making personality shape behavior.

- Mood persists and does not reset to neutral: The rationale is continuity of the bird state.

- Boldness-weighted perch assignment: The rationale is to express personality spatially: high boldness means front perch is more likely; low boldness means back perch is more likely.

- Mood-modified perch assignment: The rationale is that wary birds pull back and curious birds may step forward, making mood visible in placement.

- Perch assignment derived rather than separately persisted: The plan states this but does not articulate a separate rationale. NOT RECOVERABLE FROM PLAN

- Vocal-frequency-based call probability: The rationale is to make the trait affect audible behavior.

- Mood-modified calls: The rationale is that drowsy, alert, and wary moods change how often and how long birds call.

- Motif ID and parameters stored in the snapshot: The rationale is so the client's WebAudio engine can "synthesize it identically."

- Chorus detection: The rationale is to affect client mix levels when overlapping calls happen.

- Bird-to-bird call response modulated by social warmth: The rationale is to make social warmth shape inter-bird call behavior.

- Notebook-generation function based on "noteworthy" conditions: The rationale is to write entries for first-time events, unusual perch choices, mood changes, and weather rather than every session.

- Notebook cap of at most one entry per tick and one per calendar day: The rationale is sparsity and avoiding repetitive prose.

- Immutable notebook entries: The rationale is to keep entries "written once" as part of the field notebook record.

- Weather events with low probability and soft transitions: The rationale is ambient variation that ramps rather than hard switching.

- Drift calibration CI targets at 7 and 21 days: The rationale is to detect measurable drift after simulated daily presence and perceptible render output after 21 days.

- Call grammar motif libraries with procedural variation: The rationale is call variety without shipping recorded audio files.

- Mood-shaped attack and decay envelopes: The rationale is that alert and drowsy states should sound different.

- Species-deterministic timbre: The rationale is species-specific calls.

### Sync model

- Canonical state principle: The rationale is explicit: "no sync protocol, no merge conflicts, no CRDTs."

- Two devices pulling from the same canonical record: The rationale is that laptop and phone see "the same birds, same moods, same drift."

- Client sends interaction facts, not personality or mood values: The rationale is that the server decides what the interaction means for boldness or mood.

- Server-side timestamps on interaction receipt: The rationale is "preventing clock-skew ordering issues."

- Stale snapshot IDs ignored on event batches: The rationale is that events remain additive and appended regardless of what snapshot the client thinks it is on.

- No last-write-wins: The rationale is that "there are no writes to win over"; mutations are additive deltas by a single writer in a single time order.

- Session timeout message: The rationale is to use a matter-of-fact surface for auth failure, with "No naturalist prose."

- No silent reauthentication: The plan states this behavior but does not articulate a rationale. NOT RECOVERABLE FROM PLAN

- Hidden tab stops rendering, presence pings, snapshot pulls, and audio: The rationale is that hidden tabs should not count as presence, and audio stops by browser policy.

- Visible tab pulls a fresh snapshot and interpolates: The rationale is to show birds as having continued in the interim without a hard cut or "waking up" animation.

- Closed tab ends presence with no penalty: The rationale is to avoid punishment and settle requirements.

- Device suspend treated like hidden tab: The rationale is consistent lifecycle handling while the simulation continues.

### Frontend rendering pipeline

- React or Preact with Canvas/SVG hybrid renderer: The rationale is to keep "the 60fps idle-motion budget" away from DOM reconciliation cost while preserving DOM accessibility for UI chrome.

- Canvas for the aviary scene: The rationale is performance for idle motion.

- DOM for top bar and overlays: The rationale is preserving accessibility for account, settings, notebook, and visit flow surfaces.

- Background pass rendered once and cross-faded on day-phase transitions: The rationale is efficient rendering and soft transitions.

- Static perch pass: The plan states the geometry choice but not a separate rationale. NOT RECOVERABLE FROM PLAN

- Ambient particles generated client-side with no server state: The plan states the implementation but does not articulate a separate rationale. NOT RECOVERABLE FROM PLAN

- Bird body-part composition: The rationale is pose variation through head tilt, eye direction, beak state, wing, and tail positions.

- Plumage saturation modulates species base color: The rationale is to make the `plumage_saturation` trait visible as richer or more muted color.

- Pose state machine: The rationale is to map moods and interactions into visible bird behaviors.

- Pose selection driven by mood and personality: The rationale is that bold/content, wary, drowsy, and curious birds behave differently on screen.

- Loading sequence with quiet field: The rationale is to avoid spinner/static loading while giving a soft ambient surface before state arrives.

- First bird visible within 500ms: The rationale is fast first life and the "central conceit" protected by performance budgets.

- Placeholder silhouettes if snapshot arrives before main bundle: The rationale is seamless early rendering when data arrives before the full renderer.

- No rendering without state if main bundle arrives first: The rationale is that the client should not invent bird state.

- No entry animation or birds-appearing transition after initial adoption fly-in: The rationale is that the first frame with birds should show them "already in motion."

- Reduced-motion cross-fades between still pose frames: The rationale is to replace continuous motion while keeping state changes legible.

- Reduced-motion disabling ambient leaf/feather drift: The rationale is visual motion reduction.

- Reduced-motion leaving audio unaffected: The rationale is that reduced-motion is "visual only."

- Top bar fade to near-transparent after cursor stillness: The plan states the behavior but does not articulate a separate rationale. NOT RECOVERABLE FROM PLAN

- Keyboard-focusable top bar icons: The rationale is keyboard accessibility.

- Viewport-filling canvas with letterboxing rather than cropping: The rationale is that "All birds always visible."

- Minimum 320x480 viewport and proportional perch compression: The rationale is small-phone support without losing birds.

### Audio pipeline

- WebAudio `AudioWorklet` on a dedicated `AudioContext`: The rationale is client-side synthesis and precise scheduling without server audio streaming.

- Motif libraries encoded as parameter arrays instead of audio files: The rationale is small bundle size, about 30KB total and within the 2MB budget.

- Per-call synthesis from snapshot timestamps and motif parameters: The rationale is to reproduce server-selected calls at the right time.

- Auto-disconnect and release audio nodes after call tail: The rationale is memory hygiene.

- Node pooling: The rationale is "No per-call allocation that isn't freed" and the no-memory-growth budget.

- Ambient mix gains by bird and perch zone: The rationale is spatial-feeling quiet presence, with back perch slightly quieter.

- Listen-in mode gain ramp: The rationale is to focus one bird while keeping the rest as ambient presence.

- Non-focused birds never fully silent in listen-in: The rationale is that "they remain audible as a quiet ambient presence."

- Chorus reverb bump: The rationale is to make overlapping calls feel "like they're in the same space" while staying "gentle" and not dramatic.

- Two-second look-ahead scheduling: The plan states the scheduling window but does not articulate a separate rationale. NOT RECOVERABLE FROM PLAN

- WebAudio fallback to graceful silence with captions on by default: The rationale is a designed failure mode when audio is unavailable, with no recorded-audio fallback.

### Accessibility surfaces

- Client-side screen-reader narration from snapshots: The rationale is to represent the current aviary in naturalist prose using available state.

- Idle narration cadence of 30-60 seconds: The rationale is a slow ambient update rhythm.

- Faster narration for return-greeting, offer reaction, and settle gesture: The rationale is priority for user-initiated events.

- ARIA live region with `aria-live="polite"`: The rationale is screen-reader delivery without interruptive behavior.

- Narration priority queue: The rationale is that user-initiated events queue ahead of idle updates, while deferred idle updates are not dropped.

- Call caption toggle off by default, except on when WebAudio unavailable: The rationale is opt-in captioning unless captions are needed for graceful audio failure.

- Caption generation from motif ID and parameters: The rationale is to describe the actual procedural call being synthesized.

- Captions near the calling bird: The rationale is to locate the caption with the bird that made the call.

- Caption contrast backdrop: The rationale is WCAG AA contrast against the aviary background.

- Manual reduced-motion toggle persisted locally and synced as account-level accessibility preference: The rationale is to let the preference follow the user across devices.

- Keyboard tab order through top bar, scene, birds, listen-in, escape, and offer dropdown: The rationale is complete keyboard operation of the aviary and overlays.

- High-contrast focus indicator with inner shadow: The rationale is visibility against both bright and dim aviary states.

- WCAG AA text contrast: The rationale is readable user-copy surfaces; canvas text is limited to captions that meet contrast.

- Screen-reader-only notebook articles: The rationale is natural reading through semantic HTML and polite announcement of new entries.

### Performance budgets and observability

- Critical initial JS budget under 2MB: The rationale is to protect time-to-first-bird and the central conceit.

- HTML shell plus inline CSS under 15KB: The rationale is fast quiet-field delivery.

- Motif and bird sprite asset budgets: The rationale is to keep audio and visual assets within the performance envelope.

- Deferred code-splitting for settings, visit flow, and notebook full-view: The rationale is that the critical path stays focused on renderer, audio engine, snapshot client, and event writer.

- Runtime budget for time to first bird visible: The rationale is measuring whether the first bird appears within the intended fast-start experience.

- Runtime budget for 60fps idle motion: The rationale is sustained living motion.

- Runtime budget for memory growth under 5MB over 30 minutes: The rationale is preventing long-session degradation.

- Runtime budget for audio context latency: The rationale is keeping scheduled calls accurate.

- Runtime budgets for simulation tick, snapshot size, and snapshot delivery: The rationale is keeping server state fresh and lightweight.

- CI heap snapshot test over 30 minutes: The rationale is enforcing "No memory growth" and detecting retained disposed objects.

- Reusing audio nodes, canvas context, virtualized notebook DOM, replacing snapshot data in-place, and canceling animation frames: The rationale is to meet the memory-growth budget.

- Synthetic monitoring from three geographies: The rationale is to measure first bird, frame rate, and memory in recurring real browser sessions.

- Aggregate-only RUM: The rationale is to observe performance without including per-bird state, per-account history, or data that could reconstruct the relationship with the aviary.

- Telemetry pipelines never reading the simulation database: The rationale is the privacy boundary between analytics and user aviary state.

- Server-side metrics and alerting: The rationale is operational visibility into tick latency, event lag, snapshot generation, API health, and email delivery.

### Rollout

- Internal alpha: The rationale is to validate drift calibration, sync correctness, and audio pipeline on real devices before external users.

- Closed beta with 50-100 invited users: The rationale is real-world validation of drift calibration and presence-accounting accuracy across background tabs, laptop sleep, and phone browsers.

- No marketing during closed beta: The plan states this but does not articulate a separate rationale. NOT RECOVERABLE FROM PLAN

- Open beta with no feature gates and full monitoring: The rationale is broad launch while instrumentation is live and bird-count unlocking remains age-based.

- Bird-per-aviary ramp live from day one: The rationale is that it is "not a rollout toggle" but the product mechanic.

- Presence-accounting debug metric removed after calibration: The rationale is temporary internal validation only.

- Drift calibration monitoring with aggregated anonymized histograms: The rationale is detecting production calibration drift without per-account exposure.

- Launch checklist accessibility audits: The rationale is to make contrast, screen-reader narration, reduced motion, keyboard navigation, and captions launch gates.

- Launch checklist performance and bundle checks: The rationale is to verify memory, first bird, bundle size, and device performance before launch.

- Launch checklist account deletion, visit flow, and multi-device sync: The rationale is end-to-end validation of core account, visitor, and sync workflows.

### Risks and mitigations

- Server-side `alpha` feature flag for drift calibration: The rationale is changing drift speed with a config push and "no client deploy needed."

- Conservative initial drift `alpha`: The rationale is to avoid birds changing too fast and feeling "like a Tamagotchi."

- Snapshot generation applying all received events before return: The rationale is that snapshots reflect events received before the request, limiting sync inconsistency.

- Accepting the race where an event arrives between snapshot-read and snapshot-return: The rationale is that the next tick reflects it and about 60 seconds is "below the perceptible threshold for bird behavior changes."

- Filtered waveform motif design and chorus reverb: The rationale is to avoid procedural calls sounding "synthy" or harsh and to smooth frequency overlap.

- Pre-rendered audio buffers as an escape hatch: The rationale is quality if WebAudio synthesis cannot meet the bar, trading bundle budget for quality.

- Co-locating narration templates with bird behavior definitions: The rationale is avoiding accessibility regression when new bird behaviors are added.

- Reduced-motion CI tests beside default render path: The rationale is ensuring every visual change has a reduced-motion variant.

- Bundle size CI failure and import lint rule: The rationale is preventing bundle creep from killing time-to-first-bird.

- Adaptive tick scheduling and account-ID sharding: The rationale is scaling server-side simulation as user base grows.

- Stuck-key detection and possible human-presence heuristic: The rationale is reducing presence-accounting false positives while preserving still-watching users.

- Transactional email service, dedicated IP warm-up, delivery monitoring, resend link, and possible one-time passcode backup: The rationale is that email deliverability is a hard failure for a magic-link-gated product.

- Notebook template variety, 30-day no-repeat rule, sparse entries, and possible lightweight language-model escalation: The rationale is preserving prose quality and avoiding repetitive machine-written entries.

- "Aliveness regression suite": The rationale is protecting greeting variation, staggered greetings, procedural call variation, and non-repeating idle motion as concrete acceptance criteria for "Feels alive, not robotic" and "Restraint over richness."
