## System-level intent

- Make the aviary "continuous, private, and alive" while explicitly keeping it from becoming "a game, pet-care loop, social network, or notification product." This appears in the opening product interpretation and is reinforced by exclusions for scores, streaks, hunger, public discovery, chat, push notifications, and return announcements.

- Keep the server as the canonical author of the aviary. The plan says the client "reads canonical aviary snapshots," "writes only interaction events and presence pings," and "never writes personality vectors, mood state, notebook entries, bird positions, drift values, or simulation-owned fields." It repeats this as "the server database is the only canonical state."

- Prefer quiet, ambient observation over command, reward, or obligation. The plan uses phrases like "ambient aviary," "quiet field," "no textual welcome," "no badges," "no UI trait values," and "not direct bird command." Offers, listen-in, settle, return greetings, and notebook entries are framed as observation and gentle interaction, not goals.

- Treat bird change as slow, server-authored, and expressive only. The plan calls for "server-authored additive deltas," "no negative drift on neglect," "no single session produces visible personality movement," and calibration where instruments detect regular-use drift after "about one week" while users feel it after "about three weeks."

- Preserve bird identity and continuity. The plan emphasizes "bird stable IDs," "species_id," "call_signature_seed," "visual_seed," persisted seeds, stable call signatures, and the rule to "never replace bird identity." Rendering or species changes must not reset perceived continuity.

- Separate naturalist voice from matter-of-fact system copy. The plan creates "two explicit copy modes": naturalist voice for aviary narration, notebook, captions, offers, and observations; matter-of-fact voice for sign-in, settings, errors, revocation, export, and deletion. Copy governance blocks "welcome back," "streaks," "achievements," and stats.

- Make accessibility part of the product, not a fallback shell. Accessibility is described as "first-class," with screen-reader narration, reduced-motion rendering, keyboard access, captions, and WCAG AA contrast. The risk section warns against "accessibility flattening" where accessible modes expose state lists instead of "the product's charm."

- Keep privacy and observability boundaries strict. The plan allows "aggregate operational telemetry only," forbids "per-bird state in telemetry" and "per-account interaction history in analytics," keeps emails encrypted, and requires metrics APIs that accept only approved aggregate fields.

- Use procedural, local audio to make calls recognizable without recorded loops. The plan calls for client-side WebAudio synthesis, "stable call signature seed," variation that preserves recognizability, captions from the same grammar tokens, and "no recorded call loops, no downloaded call files, and no recorded fallback."

- Make performance part of the feeling of life. The plan sets "first bird visible <500ms," "60fps idle motion," "no spinner in aviary loading path," "quiet field fallback," and the risk that missing the first-bird budget makes loading feel "like an app waking up."

- Prevent multi-device divergence through deterministic replay and append-only events. The plan calls for seeded randomness so tests can replay behavior and "prevents multi-device divergence," plus idempotency keys, row locks, ordered tick processing, and no client state merging.

- Keep visits read-only, optional, and bounded. Visit invitations are "off by default and revocable," visitor sessions "never create presence windows or interaction events," and the risk section says visits must not become "co-presence or discovery."

- Keep the notebook sparse and observational, not a feed. The plan says notebook entries are "sparse naturalist observations, not event logs," roughly one entry every few days, no per-session guarantee, no counters, no reactions, no sharing, and mitigates "Notebook overproduction" so it does not become "a feed."

## Per-feature whys

### Product interpretation and v1 scope

- Browser-only client for modern Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN

- Single-user account model with one canonical aviary per account: The plan ties this to "private" and "one canonical aviary," and excludes shared aviaries, household accounts, team profiles, social feeds, and co-presence.

- Email magic-link sign-in and revocable sessions: The plan uses magic links with 15-minute expiry, token invalidation, neutral responses to avoid email enumeration, token hashes only, and revocable per-device sessions as part of account and session security.

- Email change verification: NOT RECOVERABLE FROM PLAN

- Account export, 30-day soft deletion, and restore: The plan says export is generated on demand and emailed as a time-limited link to the verified email; soft delete "immediately blocks normal use while allowing restore," and hard delete removes account, aviary, birds, vectors, moods, events, notebook, invites, sessions, exports, and telemetry join keys after 30 days.

- Two starter birds per new aviary: NOT RECOVERABLE FROM PLAN

- System-selected birds from a coherent small species pool: The plan's rationale is coherence; it says starter birds are "selected by the system from a coherent small species pool," and species definitions are versioned static content rather than user-customizable settings.

- User-assigned and renameable bird names: Rename updates "display name only" and has "no effect on identity, personality, mood, or call signature," so names personalize the aviary without changing simulation identity.

- Persistent bird identity, personality vectors, moods, call signatures, perch/motion state, and drift history: The plan uses these to make birds stable, continuous, recognizable, and server-canonical across sessions and devices.

- Server-side simulation tick at roughly one-minute cadence: The tick is the mechanism for server-authored mood, drift, perch/action, call scheduling, weather, notebook candidates, versioning, and snapshots. It keeps simulation-owned canonical state off the client.

- One horizontal scene: The scene rules say "one horizontal scene, no panning, no scrolling, no zoom," with responsive compression to keep all birds visible and recognizable.

- Three perch zones: NOT RECOVERABLE FROM PLAN

- Local day/night cycle: The plan uses local-time-derived phase descriptors and the account timezone to shape mood and preserve day/night continuity, especially in long absence catch-up.

- Ambient weather: Weather affects mood, vocal frequency, perch tendencies, and notebook noteworthiness; rain dampens vocal frequency, wind nudges alert or wary, and weather can intersect with notable mood.

- Ambient motion: The plan says to "never leave birds frozen" while visible; idle micro-motion, subtle breathing, preening, scanning, and response cues make the aviary feel alive.

- Return greetings: The plan uses absence length, mood, boldness, social warmth, and recent greeting history to select a greeting opportunity without textual welcome. Screen-reader narration may describe the observation in naturalist voice.

- Listen-in: Listen-in is a bird-specific signal for focused attention and an audio feature that should feel "like leaning attention, not switching tracks." It can affect social warmth and vocal frequency within caps.

- Offers: Offers are framed as small, bounded signals, "not direct bird command." Seed, song fragment, and still pool can influence curiosity, boldness, social warmth, content-related input, or vocal frequency, while cooldowns prevent saturation.

- Settle: Settle "ends presence cleanly and quiets mood" and has "no meaningful personality drift direction." It is a way to close the current presence window without obligation.

- Field notebook: The notebook is "server-generated from canonical state and event summaries" so it syncs consistently, remains sparse, and stores final naturalist text across devices and exports.

- Procedural client-side WebAudio call synthesis with captions and silence fallback: The plan avoids recorded audio loops and downloaded files, keeps audio local for performance, generates captions from the same grammar tokens, and turns captions on during graceful silence when WebAudio is unavailable.

- Screen-reader narration, reduced-motion rendering, keyboard access, and WCAG AA contrast: These surfaces are "first-class" and must ship with v1 so the product remains accessible without reducing it to raw state lists.

- Optional read-only visit invitations by email: Visits are "off by default and revocable," host-only to create, expiring, read-only, and must not create visitor events or presence windows that affect host drift.

- Aggregate operational telemetry only: The plan permits request counts, latency, frame timings, audio errors, and other aggregate operations metrics, while forbidding per-bird state, per-account history, population drift dashboards, and ML training datasets using per-bird interaction data.

- Excluding scores, streaks, badges, levels, XP, achievements, leaderboards, counters, and "birds adopted" counters: The plan excludes these so the aviary does not become a game or engagement product.

- Excluding hunger, death, sickness, distress, negative drift, and obligation surfaces: The plan excludes Tamagotchi mechanics and says absence itself does not create negative deltas.

- Excluding public discovery, profiles, follows, comments, chat, co-presence, shared cursors, and social feeds: The plan excludes these to preserve a single-user, private aviary and prevent visits from becoming social expansion.

- Excluding push notifications, default emails about aviary activity, "welcome back" text, and return announcements: The plan excludes notification loops and textual return framing, keeping return greetings as rendered observation.

- Excluding recorded audio fallback and looped bird audio: The plan requires procedural calls, no recorded fallback, no downloaded call files, and graceful silence when WebAudio is unavailable.

- Excluding per-account or per-bird interaction aggregation: The plan forbids analytics, training, recommendations, and population dashboards built from those aggregates to maintain the privacy boundary.

- TypeScript web stack end-to-end: The plan says this allows simulation math, schema types, call grammar descriptors, and client state contracts to be shared safely without duplicating definitions.

- Canvas/WebGL through a thin custom scene layer: The plan chooses this over DOM-per-bird animation to control frame timing, reduced-motion variants, and mobile performance.

- Relational primary database with queue-backed simulation worker: The plan uses it for canonical account, aviary, bird, event, invite, and notebook records, and for simulation ticks that run outside inline API requests.

- Server-generated field notebook from canonical state and event summaries: The plan says this keeps notebook sync consistent across devices and keeps entries sparse.

### System architecture

- Web client surface: The client owns sign-in screens, settings, scene rendering, input capture, WebAudio, captions, and narration because it handles presentation and browser lifecycles, while reading canonical snapshots and writing only events to avoid forking state.

- API service: The API authenticates, serves bootstrap/state/account/notebook/export/accessibility/visit surfaces, and performs idempotency, validation, authorization, and rate limiting so user writes are safe before the simulation service processes them.

- API service avoiding inline personality drift: The plan reserves drift computation for the simulation service, except lightweight read-model shaping, to keep simulation ownership clean.

- Simulation service: It is the "only writer for simulation-owned canonical state," consumes append-only interaction events in order, and computes drift, moods, perch targets, call descriptors, weather, notebook candidates, and snapshot versions.

- Background jobs: Email delivery, export generation, hard deletion, invite cleanup, probes, and notebook compaction are separated from the request path because they are asynchronous lifecycle or operational tasks.

- Independently deployable services in one product repository: The plan keeps service entrypoints thin while using shared packages for contracts, simulation core, call grammar, rendering, voice, and observability.

- Shared contracts package: The plan uses shared JSON schemas, TypeScript types, validation helpers, event names, constants, and API contracts to keep client, API, and simulation aligned.

- Pure deterministic simulation core and call grammar: The plan says simulation math and call grammar should be "pure and heavily tested" so behavior can be replayed and validated.

- Observability package that cannot accept per-bird or per-account simulation payloads: This enforces the aggregate-only telemetry boundary at the API/schema level.

- Client/server ownership boundary: The plan assigns account, aviary, bird identity, personality, mood, scene state, offers, presence rollups, visits, and snapshot versioning to the server, while the client owns interpolation, focus, audio lifecycle, captions, reduced motion, top-bar fade, and temporary optimistic UI. The rationale is canonical state plus high-frequency local presentation.

- Client caching of last snapshot, static assets, motif descriptors, and pre-auth settings: The plan allows caching for quick warm paint and settings needed before authenticated API return, but warns it must not fork canonical state.

- Refresh on visibility return, long frame gap, auth change, or session change: The plan requires refresh to prevent cached or interpolated local state from diverging from server snapshots.

- Server returning semantic scene state rather than pixels or animation frames: The plan says this keeps the server canonical while letting the client transform descriptors into Canvas/WebGL, WebAudio, captions, screen-reader narration, and keyboard focus targets within performance budgets.

### Data model

- Synthetic UUIDs for all internal identifiers and encrypted email storage: The plan uses this to keep email only on encrypted account and delivery records, with no email in logs, metrics, queue names, shard keys, or trace attributes.

- Session device label, revocation, token hash, expiry, and coarse user-agent/platform: The plan supports revocable per-device sessions while avoiding precise fingerprinting.

- MagicLink token hash, email hash for lookup, encrypted email, expiry, consumed time, and rate bucket: The plan supports neutral auth, short-lived links, consumed-link invalidation, and rate limiting without logging raw email.

- Aviary state version, last tick, timezone, settled marker, and bird count cap: These fields support canonical snapshot versioning, tick scheduling, local day/night behavior, settle state, and enforcing the seven-bird cap without showing counters.

- Bird stable IDs, species IDs, display names, seeds, starter flag, and reserved retired_at: The plan uses stable identity, persisted seeds, and non-user-visible future migration space to avoid resetting perceived bird continuity.

- BirdPersonality vectors: The plan stores bounded normalized traits for simulation, but says not to expose them through UI, ARIA, casual exports, or notebook prose; account export includes them only as raw export data.

- BirdMoodState: The plan stores mood, intensity, timers, and internal transition reason so mood persists across sessions and changes through ticks, while raw mood labels stay out of UI.

- BirdSceneState: The plan stores perch, slot, pose, action, seeds, and movement timing to let snapshots produce smooth, collision-aware rendering without client-owned canonical positions.

- PresenceWindow: The plan records qualified presence and closure reasons so server-side drift can use bounded, validated presence rather than raw client claims.

- InteractionEvent: Events are append-only with strict typed payloads, idempotency keys, and processed-at tick markers so corrections are new events rather than history rewrites.

- OfferCooldown: The plan uses per-bird offer cooldowns to prevent saturation and enforce availability on the server.

- NotebookEntry final text and source references: The plan stores naturalist text with source snapshot/event references so entries are consistent across devices and exports, while users cannot edit or delete them.

- AviarySnapshot latest or rolling short history: The plan allows enough canonical state to regenerate or debug, but says not to store indefinitely large per-minute snapshots.

- Versioned species and motif definitions: The plan keeps species content out of user-customizable settings, supports later species additions, and keeps existing birds' species IDs, seeds, and identity.

- VisitInvite and VisitSession: Invites are named, expiring, revocable, and hashed/encrypted; visitor sessions track approximate duration but never create presence windows or interaction events on the host aviary.

- AccountSettings accessibility fields: Reduced motion, captions, audio, narration, timezone, and contrast settings control presentation; changing accessibility settings can be logged for support but "must not feed bird drift."

- Visit notifications off by default: The plan keeps visits optional and non-notification-oriented.

### API surface

- JSON over HTTPS with schema validation, shared typed contracts, request IDs, and idempotency keys: The plan uses these for validation, support, and safe retries of event writes and invite creation.

- Magic-link request endpoint: It always returns a neutral success response to avoid email enumeration, while sending a 15-minute link if rate limits allow.

- Magic-link consume endpoint: It invalidates the token on successful consumption and returns matter-of-fact errors for expired, consumed, or invalid tokens.

- Account endpoint returning settings, sessions, export/deletion state, and masked email: NOT RECOVERABLE FROM PLAN

- Account settings update: The plan says settings and accessibility changes do not affect simulation drift except through future rendering behavior.

- Session revoke endpoint: The plan supports revocable device sessions as part of account security.

- Export endpoint and export status: Export is asynchronous and delivered by a time-limited email link to the verified address; status exists only if the UI needs account-setting status.

- Delete and restore endpoints: The plan uses delete to mark the account and set hard_delete_after, and restore to recover within the 30-day soft-delete window.

- Aviary bootstrap endpoint: The bootstrap returns settings, latest snapshot, notebook summary, cooldowns, and flags, and the plan says it must be small enough for the "<500ms first-bird budget."

- Aviary snapshot endpoint with since_version: The plan uses 304 or compact newer snapshots on visibility return, long frame gaps, keepalive, and after interaction acknowledgement to keep clients synced.

- Notebook endpoint: It is paginated and read-only, with no edit or delete endpoints, matching the sparse naturalist notebook design.

- Bird rename endpoint: It changes display name only and does not affect identity, personality, mood, or call signature.

- Interaction event endpoint: It accepts batched presence and interaction events with idempotency keys and returns accepted/rejected IDs plus current snapshot version, but no drift deltas, trait values, or gamified feedback.

- Event payload schemas for presence, listen-in, offers, settle, and reengage: The plan uses strict schemas so events are calibrated inputs rather than direct client mutations.

- Presence validation: The server ignores hidden or unfocused documents, caps credited seconds, drops duplicate idempotency keys, closes inactive windows, and treats settle/tab close/visibility loss as terminal to prevent presence inflation.

- Visit invite creation: It is host-only, one-time, expires in 30 days, sends email, is off by default, and has no onboarding prompt.

- Visit invite list and revocation: The settings surface shows outstanding, accepted, expired, and revoked invites plus visit log, and revocation is immediate.

- Visit consume and visitor snapshot endpoint: The visitor gets render-only snapshots while active; expired or revoked access returns a matter-of-fact "visit no longer available" response.

- Visitor clients lacking event submission, notebook mutation, host settings, and bird renaming endpoints: The plan prevents visitors from affecting host state and says v1 should avoid exposing the host notebook unless product design explicitly includes it later.

### Simulation engine design

- Tick lifecycle: The worker selects due aviaries, locks one aviary row or partition, loads canonical state and events, bounds elapsed time, aggregates events, applies drift, computes mood/action/calls/weather/notebook, writes atomically, increments version, and emits aggregate metrics. The why is canonical, ordered, bounded simulation.

- Long absence handling: The plan avoids one tick per missed minute for months, uses catch-up windows to preserve day/night and mood continuity without fabricating presence, and says absence itself does not create negative deltas.

- Deterministic seeded randomness: Seeds from aviary ID, version, bird ID, action family, and timestamp bucket allow tests to replay behavior and prevent multi-device divergence.

- Client randomness only for non-canonical ornaments: The plan allows it for leaf drift bounded by reduced-motion settings because it does not affect canonical decisions.

- Personality drift low-pass filter: The plan uses slow EWMA, diminishing returns, daily clamps, and weekly/three-week calibration so drift is measurable before it is obvious and not gameable session by session.

- Presence-time input: The plan makes it the dominant positive input across expressive traits, supporting social warmth, vocal frequency, curiosity, plumage saturation, and modest boldness when consistent.

- Listen-in input: The plan makes it bird-specific, duration-weighted, and capped per day, increasing social warmth and vocal frequency for the focused bird.

- Offer inputs: Seed, song fragment, and still pool produce small trait inputs when birds approach, investigate, or respond, but cooldowns prevent saturation.

- Settle input: The plan says settle ends presence cleanly, quiets mood, and has no meaningful drift direction.

- Drift rules against neglect punishment and fast visible change: The plan says no negative drift, no single-session visible movement, daily and weekly clamps, and no product UI exposure of vectors or accumulators.

- Mood enum with intensity and timers: The plan uses mood as persistent simulation state influenced by time of day, offers, listen-in, weather, other birds, personality, and absence length for greeting selection, not punishment.

- Weighted mood transitions: The plan chooses weighted scores rather than hard if/else chains so personality can modulate behavior.

- Mood expression without UI labels: The plan avoids mood labels in UI and expresses mood through perch, motion, call timing, caption/narration prose, and notebook observations.

- Return greeting descriptor: The server selects a candidate bird by boldness, social warmth, current mood, recent greeting history, and absence length; only one bird greets first, others may stagger, and the client renders it without textual welcome.

- Call grammar runtime: Species motif library, stable signature seed, personality-shaped timing/pitch, and mood-shaped envelope make calls varied while preserving recognizability.

- Server-scheduled call descriptors with client synthesis: The server stays canonical over scheduling, while the client performs oscillator/envelope/filter synthesis and captions locally for performance and browser audio lifecycle.

- Notebook generation from meaningful state changes: Entries come from first-greeter changes, unusual perch time, rare chorus, weather plus mood, personality-shaped offer reactions, or long quiet stretches, so they are noteworthy rather than event logs.

- Notebook eligibility scorer: Noteworthiness, minimum gap, novelty, privacy check, and non_goal_check keep entries sparse, private, and non-feed-like.

### Sync model and conflict prevention

- Server database as only canonical state: Multi-device sync emerges from devices reading the same snapshot and submitting append-only events, not merging with each other.

- Snapshot versioning: The plan increments state_version on simulation changes, returns latest version on event writes, supports since_version polling, and has clients discard interpolation when server versions advance incompatibly.

- Event ordering with server receive time, client occurred time, session ID, idempotency key, and optional sequence: Server receive order is authoritative for mutation safety, while client time only refines duration within plausible bounds.

- Delayed or out-of-order event bounds: The plan accepts them only within a bounded recent window and prevents reopening closed presence windows incorrectly.

- Two devices listening to different birds: Both can be recorded if they meet presence criteria, but account-level caps prevent multi-device amplification.

- Two devices sending offers: Events are processed in order, cooldown is checked at acceptance and tick, and cooldown arrivals get matter-of-fact unavailable responses or quiet no-ops.

- Rename from two devices: Rename is not simulation state, uses optimistic concurrency, returns matter-of-fact conflict and prompt reload if conflicting, and never replaces bird identity.

- Session expiry mid-write: The plan validates session before accepting, keeps already accepted events valid, and prevents rejected events from mutating state.

- Magic-link replay: Consumed links cannot be consumed again, while existing sessions remain account-scoped and revocable.

- Server outage behavior: The client may render last known snapshot for a bounded period but must not invent drift locally; recovery replaces descriptors from canonical snapshot.

- Presence integrity rules: Presence qualifies only with visible document, focused window, and recent pointer/key activity; the server credits bounded intervals, leaning longer to respect quiet watching while preventing background tabs from corrupting drift.

### Frontend rendering pipeline

- Minimal app shell and bootstrap: The plan serves minimal HTML/CSS/JS, resolves auth, fetches or embeds bootstrap, paints quiet field or snapshot, and makes the first bird visible within 500ms before loading non-critical surfaces.

- Avoiding spinners in aviary loading: The plan uses a quiet field with soft sky color and minimal motion cue because loading should not feel like an app waking up.

- Code-split account settings, notebook history, visit management, export/delete, and other non-critical surfaces: The plan protects the first-bird budget and bundle budget.

- Scene composition layers: NOT RECOVERABLE FROM PLAN

- Responsive scene compression: The plan keeps all birds visible; wide screens increase spacing and narrow screens preserve bird visibility and recognizability.

- No buttons, labels, badges, hover tooltips, or overlays inside the aviary scene: The plan preserves the quiet ambient scene and avoids gamified or UI-heavy clutter.

- Bird focus indicator: The plan makes it visible, soft, and high-contrast so keyboard focus is accessible without disrupting the scene.

- Bird pose families: The plan uses pose families to express idle, preen, scan, call, shuffle, drowsy, investigate, drink/bathe, greet, and transition states through rendering.

- Personality affecting bird rendering: Boldness, social warmth, vocal frequency, plumage saturation, and curiosity are expressed through perch likelihood, orientation, call pose frequency, color richness, feather detail, head tilts, and offer investigation rather than stats.

- Mood affecting pose and timing: Mood is rendered behaviorally so UI can avoid raw mood labels.

- Interpolation and cross-fade on snapshot refreshes: The plan uses easing, action continuation, cross-fade, or hidden repositioning to avoid teleporting after suspension.

- Idle micro-motion: The plan says birds must never be frozen while visible, and uses procedural timing variation to keep the aviary alive.

- Reduced-motion renderer: The plan implements a parallel layer that replaces motion with slow cross-fades, removes leaf/feather drift, slows color shifts, and preserves calls, captions, mood, notebook, drift, interactions, focus, keyboard, and listen-in.

- Shared descriptor input for motion and reduced-motion renderers: The plan keeps accessibility semantics and layout shared while maintaining separate visual QA baselines.

- Top bar sparse controls: The top bar contains only account/settings, accessibility settings, field notebook, offer affordance, and maybe a quiet settle control, keeping the aviary scene free of overlays.

- Settle as quiet top-bar control: The plan resolves settle by making it visually subordinate and not a large CTA.

- Top bar fade and restore: The plan fades controls nearly transparent after cursor stillness and restores them on cursor movement or keyboard activity to keep the scene quiet while preserving reachability.

- No top-bar badges: The plan forbids badges for visits, streaks, notebook count, or activity to avoid counters and engagement framing.

- Keyboard model: The plan supports Tab, arrows, Enter, Escape, keyboard-navigable offer menu, settle reachability, and focus contrast so the complete aviary interaction model is keyboard accessible.

- Field notebook UI: It opens from the top bar as read-only scrollable entries with lowercase naturalist copy, no editing, deletion, reactions, comments, sharing, counters, or badges.

### Audio pipeline

- AudioContextManager, CallScheduler, BirdVoice, ChorusMixer, ListenInMixer, and CaptionEmitter: The plan separates browser permission/device handling, descriptor scheduling, per-bird synthesis, chorus mixing, listen-in gain ramps, and captions so audio and captions share the same call grammar.

- Compact procedural synthesis: Oscillators, noise sources, envelopes, filters, motif grammar, species timbre, bird seed, mood, and personality generate varied calls without recorded assets.

- Recognizable individual calls: The plan says a user should identify Pip's call across mood changes after repeated exposure and that up to seven birds should remain separable in the mix.

- Automated audio tests: The plan uses tests to compare motif identity while verifying variation.

- Listen-in gain ramps: The focused bird rises gradually, other birds drop but never silence, and ambient remains audible so the interaction feels like leaning attention.

- Listen-in disengagement triggers: The same slow ramp reverses on toggle, another bird focus, empty scene click, keyboard focus away, Escape, or route change, avoiding abrupt track-switching.

- Captions generated from procedural grammar: Captions match the actual synthesized call, appear near the calling bird, use naturalist voice, pass contrast, and auto-enable when audio is unavailable or disabled if settings allow.

- WebAudio unavailable fallback: The plan renders graceful silence, turns captions on by default for that session, and puts matter-of-fact status in settings rather than an aviary overlay.

### Accessibility surfaces

- Screen-reader narration queue from the same scene snapshot: The plan uses one descriptor source for rendering and narration so screen-reader output reflects canonical scene state.

- Narration cadence and priority: Idle narration every 30-60 seconds and restrained user-event priority avoid flooding screen-reader queues.

- Naturalist narration style: The plan uses lowercase, present-tense, specific prose and avoids numerical stats, raw mood labels, perch indexes, or trait values to preserve charm.

- Semantic access: The scene container has a concise label, birds are focusable listen-in controls with names but not stats, top-bar controls have matter-of-fact labels, the offer menu is structured, and the notebook is a readable document region.

- Reduced motion default and override: The plan uses system prefers-reduced-motion by default, adds an in-product override, ships it at v1 launch, and includes it in visual regression and performance testing.

- Contrast and captions: The plan requires WCAG AA text, caption placement that avoids busy regions, tested day/night text tokens, and focus outlines visible across palettes.

- Accessibility QA: Keyboard-only, screen-reader, reduced-motion, caption/audio-off, and axe checks are required to prove the surfaces work end to end.

### Performance and observability

- Hard budgets: The plan sets bundle, first-bird, 60fps idle, memory, and simulation tick p99 budgets to keep the aviary responsive and alive.

- Bundle strategy: Code-splitting, small render core, procedural audio, compact visual assets, and tree-shaken motifs/species keep first paint and first bird fast.

- First-bird strategy: Embedded bootstrap where feasible, small snapshots, drawing the first bird before non-critical assets, cached species assets, and quiet field fallback protect the first-bird budget.

- Runtime strategy: requestAnimationFrame, adaptive quality, avoiding per-frame allocations, pooling audio nodes, reusing captions, pausing rendering when hidden, and refreshing after long frame gaps prevent jank and divergence.

- Memory strategy: 30-minute soak tests, heap snapshots, no unbounded call/caption/notebook/particle arrays, and bounded ornament pools prevent memory growth.

- Allowed aggregate metrics: Request latency, auth counts, snapshot latency, event counts by coarse type, tick duration, queue lag, first-bird timings, frame timings, audio errors, WebAudio unavailable counts, and anonymized non-joinable session-duration histograms support operations without behavioral dossiers.

- Disallowed observability: Per-bird telemetry, per-account interaction history, average drift dashboards, offer behavior dashboards, individual outcomes, and ML datasets using per-bird interaction data are banned to protect privacy.

- Metrics implementation guard: Approved aggregate-field APIs, simulation DB isolation from analytics, schema allowlists, static checks, and privacy review release gates enforce the boundary.

### Rollout plan

- Phase A foundations: The plan starts with repository structure, contracts, auth/session, UUIDs, schema, worker skeleton, snapshot API, and minimal shell/test bird because contracts and schema are critical dependencies before deeper parallel work.

- Phase B canonical simulation: Tick locking, versioning, presence qualification, drift, mood, perch/action, snapshots, and replay tests come before final rendering/audio integration because the snapshot format must stabilize.

- Phase C rendering and audio: The plan adds scene, perch zones, bird renderer, return greeting, day/night, weather, ornaments, WebAudio, listen-in, captions, and reduced motion after canonical simulation is available.

- Phase D product interactions: Offers, settle, notebook, rename, top-bar fade, and accessibility settings follow the core simulation/rendering base.

- Phase E accounts, privacy, and visits: Settings, session revocation, email change, export/delete, visits, matter-of-fact errors, policy link, and telemetry boundary enforcement gather account and privacy surfaces together.

- Phase F hardening: Performance, accessibility, cross-browser, memory, calibration, security/privacy, and copy/tone reviews gate release readiness.

- Launch bird count: Every new account starts with two birds, the cap of seven is enforced from day one, and the plan says not to show "2/7" counters.

- Third-bird ramp: Eligibility is based on aviary age rather than visit count, offers are quiet and naturalist, and the plan forbids framing as reward, unlock, achievement, or milestone.

- Third-bird beta flag: The plan keeps eligibility disabled behind a server flag until call recognizability and scene density are validated.

- Release strategy: Internal dogfood uses resettable test accounts separated from production, private beta uses limited invites and no public discovery, and production ramps by account creation rate.

- Feature flags: Weather, notebook generation, visit invites, third-bird eligibility, and reduced-motion fallback are flaggable, but reduced-motion fallback can only go to a safer reduced presentation, not an inaccessible default.

- No feature flags for gamification counters or notification loops: The plan prevents non-goal surfaces from creeping in through rollout.

- Day-one instrumentation: Aggregate first-bird, bundle, snapshot, tick, frame, audio, memory, auth, and visit delivery metrics ship from day one with no per-bird or per-account behavioral dashboards.

### Testing strategy

- Simulation tests: Drift clamps, no negative absence drift, no single-session visible movement, weekly and three-week calibration, mood weighting, cooldowns, idempotency, replay, event ordering, snapshots, absence catch-up, auth bootstrap, and deletion tests protect the server-authored simulation contract.

- Client tests: Snapshot contracts, visibility and frame-gap refresh, listen-in ramps, caption-call matching, reduced-motion descriptor parity, keyboard focus, top-bar fade, and no-spinner loading protect presentation, accessibility, and sync behavior.

- End-to-end tests: New account, starter birds, render, return greeting, presence, listen-in, offers, settle, notebook, export, delete/restore, visits, and WebAudio-unavailable flows prove the v1 product surface works across services.

- Accessibility tests: Narration cadence/content, keyboard-only session, reduced-motion session, captions with audio off, and WCAG AA checks verify the "first-class" accessibility commitment.

- Performance tests: Bundle budget CI, throttled first-bird check, memory soak, 60fps idle profile, and simulation tick load tests verify the product's performance gates.

### Security and privacy plan

- PII handling: Encrypted email, synthetic UUIDs, lookup hashes only where needed, no email in logs/metrics/queues/shards/traces, log scrubber tests, and review checklists reduce PII exposure.

- Session security: Token hashes, HttpOnly/Secure/SameSite cookies, revocable sessions, short-lived magic links, consume invalidation, and rate limits protect account access.

- Visit security: Hashed invite tokens, one named visitor email, 30-day expiry, immediate revocation, per-request validity checks, and no visitor writes keep visits read-only and bounded.

- Deletion and export: Soft delete blocks normal use while preserving restore, hard delete removes simulation and account data after 30 days, export goes to verified email as a time-limited link, and export access is audited only at aggregate operational level.

### Voice and copy governance

- Naturalist voice: The plan assigns aviary narration, notebook, captions, offer prompts, and bird observations to lowercase, present-tense, specific copy with no exclamation and no "you" announcement framing.

- Matter-of-fact voice: The plan assigns sign-in, settings, sync errors, unsupported browser, accessibility settings, visit revocation/expiration, and export/delete flows to plain system copy.

- Separate voice modules and copy review: `packages/voice`, lintable boundaries, banned phrase checks, and review checklists prevent "welcome back," streaks, achievements, stats, and user-behavior observations from entering product copy.

### Key risks and mitigations

- Drift calibration too fast: The plan sees gameable or session-by-session change as the risk and mitigates with synthetic persona tests, daily/weekly clamps, beta calibration, and no UI trait values.

- Drift calibration too slow: The plan sees users feeling nothing changes as the risk and mitigates with instrument-only thresholds after one week, three-week qualitative review, and sparse notebook observations of real changes without numbers.

- Presence inflation: The plan sees background tabs corrupting drift as the risk and mitigates with visibility, focus, recent input, server plausibility caps, and hidden/minimized/background tests.

- Sync correctness: The plan sees multi-device overwrites or duplicate events corrupting personality as the risk and mitigates with server-only personality writes, append-only events, idempotency keys, row locks, ordered ticks, and no last-write-wins for vectors.

- Audio uncanniness: The plan sees synthetic or repetitive calls as the risk and mitigates with motif grammar iteration, per-bird seeds, audio QA, variation tests, no loops, and recognizability testing at seven birds.

- Performance misses first-bird budget: The plan sees loading feeling "like an app waking up" as the risk and mitigates with the bundle gate, embedded bootstrap, quiet field fallback, code splitting, and first-bird-before-non-critical rendering.

- Accessibility flattening: The plan sees accessible modes losing charm as the risk and mitigates with designed narration, reduced-motion renderer, grammar-derived captions, and accessibility QA from day one.

- Tone drift: The plan sees toasts, badges, welcome text, or system-y notebook entries creeping in as the risk and mitigates with copy governance, banned phrase tests, design review, and no generic event-log notebook architecture.

- Privacy boundary erosion: The plan sees per-bird interaction data leaking into analytics as the risk and mitigates with a physical/service boundary, metrics allowlist, and privacy review release gate.

- Social feature expansion: The plan sees visits becoming co-presence or discovery as the risk and mitigates with visitor read-only authorization, no visitor events, no public surfaces, and visit notifications off by default.

- Notebook overproduction: The plan sees the notebook becoming a feed as the risk and mitigates with minimum gaps, novelty scoring, sparse generation tests, and no per-session entry guarantee.

- Bird identity migration: The plan sees species or rendering changes resetting perceived continuity as the risk and mitigates with stable bird IDs, persisted seeds, migration tests, and no regeneration of personality or call signature.

### Engineering work breakdown and definition of done

- Platform and data workstream: Database schema, migrations, account/session/auth, event log, snapshot storage, and deletion/export jobs form the data and lifecycle foundation.

- Simulation workstream: Pure simulation core, tick worker, drift calibration, mood transitions, notebook eligibility, and deterministic replay own canonical behavior.

- Client scene workstream: Render pipeline, responsive layout, bird poses, day/night/weather, interactions, and reduced motion own the visible aviary.

- Audio workstream: Call grammar, WebAudio synthesis, chorus mixer, listen-in ramp, captions, and fallback own procedural sound and caption parity.

- Accessibility and copy workstream: Narration composer, keyboard model, captions, WCAG validation, and naturalist/system voice governance keep accessibility and tone aligned.

- Social and account settings workstream: Visit invites, revocation, visit log, settings, export/delete, and session revocation own bounded account and visit surfaces.

- Observability and QA workstream: Aggregate metrics, synthetic probes, performance CI, privacy checks, and E2E suite enforce release gates.

- Critical dependencies: Contracts and schema come before deep client/simulation parallelism, snapshot format before final rendering/audio integration, voice rules before notebook/narration/caption expansion, and performance checks before adding visit/settings/notebook code to the main bundle.

- V1 readiness: The plan defines done as magic-link sign-in, two starter birds, names, first-bird budget, motion without spinner or welcome text, server-processed events, monotonic server drift, persistent mood, canonical multi-device sync, procedural varied captioned calls, full accessibility, sparse read-only notebook, bounded read-only visits, account lifecycle, aggregate-only observability, release gates, and no non-goal surfaces in code, copy, API, telemetry, or settings.
