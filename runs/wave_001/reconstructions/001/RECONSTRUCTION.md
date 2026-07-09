## System-level intent

- Build a calm, already-in-progress place rather than an app that begins from a loading or static state. This appears in the delivery intent as a browser that renders "a calm, already-in-progress place," in the frontend requirement that the first snapshot place "birds with non-zero pose phases and ambient motion already underway," in the risk about the "first-frame aliveness promise," and in the definition of done where two named birds are "already moving."

- Keep canonical life on the server and keep clients as renderers and event submitters. This appears in the core invariants: "The server is the only writer of canonical personality, mood, bird identity, and simulation state" and "Clients submit facts as append-only interaction events." It also appears in the server/client boundary: "Client rendering can be paused when hidden; the server tick never pauses," and in consistency rules rejecting "last-write-wins."

- Preserve stable identity and continuity. This appears in "stable-identity birds," "stable bird identities," "one canonical aviary," monotonically increasing "canonical revision," persisted mood, persisted render state, and the definition of done where the user can "return later to a server-advanced canonical aviary."

- Make drift slow, additive, monotonic, and expressive rather than punitive. This appears in the invariant "Drift is slow and monotonic toward expressive: neglect never decreases personality traits," the drift section's "additive, non-negative deltas," the requirement that "Neglect/absence contributes no negative personality delta," and tests that a "two-week absence cannot lower any personality trait."

- Avoid game, pet-care, native, and social-network mechanics. This appears in the explicit release exclusions: "payments," "public discovery," "profiles," "follows," "feeds," "comments," "chat," "leaderboards," "achievements," "scores," "levels," "badges," "streaks," "hunger/health/death/distress mechanics," and in the later risk that "feature creep turns a quiet aviary into a game or social network."

- Keep product expression naturalist and specific while keeping system surfaces direct. This appears in the invariant that product surfaces use "lowercase, present-tense, specific naturalist prose," while account, settings, accessibility, sync, error, and unsupported-browser surfaces use "matter-of-fact system language." The same split appears in API responses, narration, captions, and accessibility surfaces.

- Protect privacy through synthetic identifiers, minimum telemetry, and separation from analytics. This appears in account storage where "email is not an identifier outside this row," telemetry that accepts "aggregate operational metrics only," and the data-retention instruction not to copy simulation history to "an analytics warehouse, model-training dataset, recommendation store, leaderboard table, or cross-account aggregate."

- Keep visitors read-only and separate from host presence and interaction. This appears in the invariant "A visitor is read-only and contributes neither presence nor interaction events to the host," the visit module, visit APIs, and tests for "host-state mutation and presence leakage."

- Make accessibility first-class rather than a later fallback. This appears in the v1 scope including "screen-reader narration, reduced-motion rendering, call captions," the accessibility section's keyboard, narration, contrast, no-audio, and visitor-mode tests, and the risk mitigation making these "release-blocking deliverables from Milestone B onward."

- Keep audio procedural and browser-synthesized. This appears in the invariant "Calls are procedurally synthesized in the browser. There is no recorded-audio fallback," the call grammar runtime contract, the WebAudio graph, and the mitigation preferring "silence plus captions to a recorded fallback."

- Bound behavior for reliability and continuity. This appears in worker leases, sequence/cursor checks, idempotency keys, bounded catch-up, quarantined malformed events, snapshot revision semantics, and the principle that a malformed input must "preserve the last canonical state and stable bird identities rather than reset or regenerate birds."

## Per-feature whys

### Delivery intent and scope

- Web-only Pocket Aviary: the plan says the release "deliberately does not include a native client" and later excludes "native" mechanics; no further product rationale is articulated beyond the scoped v1 release. NOT RECOVERABLE FROM PLAN

- Single-user Pocket Aviary: the plan says v1 has "single-user" scope and excludes "shared or multi-aviary accounts"; this supports one canonical aviary and avoids social/account expansion surfaces.

- One canonical aviary: the rationale is continuity and authoritative simulation: "one canonical aviary continues to simulate on the server" and the user can "return later to a server-advanced canonical aviary."

- Two stable-identity starter birds: the plan requires "two stable-identity birds" and later says "Launch all new accounts with exactly two starter birds." A distinct why for exactly two is not articulated. NOT RECOVERABLE FROM PLAN

- Maximum of seven birds: the plan ties this to validating "seven-bird CPU/audio/recognizability capacity" and later testing "chorus density from two through seven birds" for recognizability and CPU stability.

- Aviary-age-based availability: the rationale is that availability is "not tied to visits, interaction counts, scores, or payment" and the UI should describe an available bird as "a quiet offer, never as a reward, milestone, counter, or achievement."

- Magic-link accounts: the plan includes magic links in scope and gives constraints for 15-minute single-use links and no account-enumeration disclosure, but no product-level why for choosing magic links is articulated. NOT RECOVERABLE FROM PLAN

- Server-side simulation: the rationale is that the server is the "only writer" of canonical state, "the server tick never pauses," and clients must not author replacement state.

- Multi-device snapshots: the rationale is that different signed-in devices see "the same state," with revisioned snapshots and duplicate/stale-event handling preventing lost or double-applied events.

- Field notebook: the rationale is to provide "sparse specific notebook observations" in "naturalist prose" without exposing "raw event identifiers or numeric trait changes."

- Presence accounting: the rationale is to drive slow drift while preventing inflated or false presence, using "visible document + window focus + recent pointer or key activity" and server validation.

- Listen-in: the rationale is that listen-in is a "strong bird-specific signal" affecting social warmth and vocal frequency, and the audio pipeline lets the focused bird rise while others remain ambient.

- Offers: the rationale is to provide small, bounded interaction signals: "An accepted offer gives a small curiosity signal" and making an offer near a bird gives "a small boldness signal," while avoiding arbitrary content.

- Settle: the rationale is to quiet mood and close presence without affecting personality: "Settle quiets mood and closes presence but does not increase or decrease personality."

- Local-time day/night: the rationale is that local time shapes mood and activity, including night settling and nightjar-like call eligibility, and lets the aviary change while the user is away.

- Rare ambient weather: the rationale is mood-relevant ambience: weather is "rare, brief, quiet, and mood-relevant rather than a feature dashboard"; rain and wind affect vocal frequency, alertness, and wariness.

- Read-only named visits: the rationale is to let "a specifically invited visitor" see the "real host aviary" while avoiding public discovery, mutation authority, presence credit, and host-state leakage.

- Screen-reader narration: the rationale is to provide the same experience through a throttled narration region, generated from snapshots/interactions, without exposing "a state table, perch index, mood label, or personality vector."

- Reduced-motion rendering: the rationale is accessibility while preserving identity and simulation: replace micro-motion and flight paths with cross-fades, remove leaf drift, and keep "mood, calls, captions, notebook, server simulation, and bird identity" unchanged.

- Call captions: the rationale is that captions come from the "same call intent/grammar selection as the audio," support no-audio/error fallback, and avoid fixed captions disconnected from sound.

- Account/privacy controls: the rationale is to support session revocation, verified email change, export, soft/hard deletion, privacy controls, and deletion coverage so account-linked data can be removed and identifiers do not leak.

### Proposed architecture

- Modular server application with a separately scheduled simulation worker: the rationale is to avoid splitting every domain into independently deployed microservices while keeping "explicit interfaces" so worker, API, export job, and visit surface can later separate without changing the domain model.

- Web client and edge bootstrap: the rationale is to serve the shell, critical renderer, small assets, and short-lived state quickly, while ensuring it "does not calculate simulation outcomes."

- Identity/session module: the plan lists responsibilities for magic links, tokens, revocation, email changes, deletion/export controls. A distinct why for this module boundary is not articulated beyond explicit interfaces. NOT RECOVERABLE FROM PLAN

- Aviary state module: the plan lists ownership of accounts, aviaries, identities, birds, moods, weather, revisions, and snapshots. A distinct why for this module boundary is not articulated beyond explicit interfaces. NOT RECOVERABLE FROM PLAN

- Interaction ingestion module: the rationale is to validate and append bounded events with authentication and idempotency keys, keeping clients as fact submitters rather than state writers.

- Simulation worker: the rationale is to serialize tick work, consume ordered events, update canonical state transactionally, and maintain revisions.

- Invitation/visit module: the rationale is opt-in named visits with one-time links, read-only sessions, host-visible visit-log facts, revocation, and 30-day unused expiration.

- Notification/email module: the rationale is to send necessary identity, export, and invitation email while keeping visit notifications "disabled by default" unless the host explicitly enables them.

- Telemetry gateway: the rationale is aggregate operational metrics only, with no read access to simulation database or interaction-event payloads.

- Relational primary store with transactions and row-level locking: the rationale is durable ordering, unique constraints, encrypted-at-rest fields, and worker leases for canonical state and append-only event logs.

- Cache/CDN for snapshots: the rationale is acceleration only; it is "never authoritative."

- Queue for tick scheduling, email, and exports: the rationale is operational scheduling while keeping the queue message from becoming "the source of truth."

- Encrypted account email with generated account UUID references: the rationale is that email is stored once, redacted from logs, and never used as identifier outside the account row.

- Server/client boundary: the rationale is to keep semantic authority on the server and animation, compositing, focus, audio synthesis, captions, and narration on the client.

- Render-safe snapshot: the rationale is first-frame placement "in progress" without exposing the raw personality vector or making it reconstructable as a stats surface.

- Immutable client snapshots and revision-gap recovery: the rationale is that stale clients may finish visual interpolation but "must not author a replacement state."

- Monotonically increasing canonical revision and simulation cursor: the rationale is retry-safe worker behavior and ordered event application.

- Client-generated idempotency keys: the rationale is duplicate submissions returning the original accepted sequence or safe duplicate result.

- Server timestamps for ordering: the rationale is to avoid device-clock authority and keep event ordering bounded by ingestion rules.

- Stale-revision mutation rejection: the rationale is to avoid merging client state into canonical personality, mood, or bird records.

### Data model

- `accounts`: the rationale is to store verified email, timezone, deletion state, and settings while keeping email out of identifiers. The plan does not articulate a separate why for every listed field. NOT RECOVERABLE FROM PLAN

- `device_sessions`: the rationale is per-device session control so account settings can "revoke one session without affecting the others."

- `magic_link_requests`: the rationale is single-use, 15-minute link consumption with throttling and immediate invalidation on use.

- `email_change_requests`: the rationale is that "The old address remains active until the new address verifies."

- `account_settings`: the rationale is to hold accessibility, captions, timezone, and optional visit notification, with "No gamification or visit-frequency fields."

- `account_deletion_requests`: the rationale is 30-day restore and later hard deletion of birds, vectors, notebook, telemetry records, and account-linked data.

- `aviaries`: the rationale is one row per account with revision, tick, cursor, day/night, weather, settle, and lease fields; "Enforce one aviary per account."

- `birds`: the rationale is stable bird identity: names can change "without changing the ID or any history."

- `bird_personality`: the rationale is server-only storage and no public serialization; persist it directly and "never rebuild it from events."

- `bird_mood`: the rationale is that mood "persists through sessions and is advanced by ticks."

- `bird_render_state`: the rationale is that presentation state is persisted by the tick "so a new client can begin mid-action."

- `aviary_events`: the rationale is append-only event sequencing with idempotency, auditability, source kind, and no visitor interaction acceptance.

- `presence_windows` or normalized presence events: the rationale is to drive drift and audit presence logic without exporting it to aggregate telemetry.

- `notebook_entries`: the rationale is read-only naturalist prose tied to observation class and revision, without raw event identifiers or numeric trait changes.

- `ambient_events`: the rationale is rare, brief, quiet, mood-relevant weather rather than a dashboard.

- `invitations`: the rationale is opt-in named invites with 30-day unused expiration and no revival.

- `visit_sessions`: the rationale is read-only visit snapshots with no host mutation authority and no host presence credit.

- `visit_log_entries`: the rationale is host on-demand inspection without a settings badge.

- `export_jobs`: the rationale is authenticated short-lived export delivery including birds, vectors, moods, notebook, and settings "as required by the PRD."

- Hard deletion coverage: the rationale is to remove canonical rows, event history, notebook, vectors, invite/visit records, export artifacts, and identifiable telemetry.

### API surface

- Versioned `/v1` JSON APIs: the plan requires versioning and authentication, but no distinct why for `/v1` is articulated. NOT RECOVERABLE FROM PLAN

- Matter-of-fact API errors and actionable system text: the rationale is the product/system voice split: system surfaces are direct while product observations remain naturalist prose.

- `POST /v1/auth/magic-link`: the rationale is rate-limited sign-in without revealing whether an account exists.

- `POST /v1/auth/magic-link/consume`: the rationale is single-use consumption, account creation/resume, per-device session issue, timezone setup, and transactional starter-bird creation.

- `POST /v1/auth/logout`, `GET /v1/account/sessions`, and session deletion: the rationale is ending/listing sessions and revoking a selected device session.

- Email change endpoints: the rationale is verifying the new email before switching.

- Account export endpoint: the rationale is enqueueing export and emailing a download link to the verified address.

- Account deletion and recovery endpoints: the rationale is starting or recovering the 30-day soft deletion before scheduled hard delete.

- Account settings endpoint: the rationale is to update accessibility, captions, timezone, and optional visit notification while rejecting unsupported gamification settings.

- Aviary snapshot endpoint: the rationale is returning current canonical revision and render-safe state, supporting conditional reads and first paint.

- Notebook endpoint: the rationale is read-only paginated history without raw event logs or numeric drift.

- Aviary events endpoint: the rationale is accepting only a closed event union with idempotency, ownership checks, cooldowns, settle timing, and validated presence claims.

- Presence close endpoint: the rationale is safe repeatable presence closure, with server timeout resolving missing browser lifecycle closes "never penalized."

- Closed offer payloads: the rationale is to avoid arbitrary content, enforce cooldowns, and make the server choose receiving bird/reaction context.

- Invitation creation endpoint: the rationale is one-time named invitation without exposing a global discoverability model.

- Invitation listing endpoint: the rationale is host visibility into expiration and revocation state.

- Invitation revocation endpoint: the rationale is immediate revocation, with next visitor snapshot pull terminating the session.

- Visit log endpoint: the rationale is host-only, on-demand visibility without badge or automatic in-product notification.

- Visit redemption endpoint: the rationale is constrained read-only session creation and same unavailable surface for expired or revoked links.

- Visitor snapshot endpoint: the rationale is real host aviary rendering with no mutation capability, presence ingestion, or interaction events.

- Shared host/visitor snapshot serializer: the rationale is that visitors see "the real aviary rather than a special show-off rendering."

### Simulation engine

- Scheduled tick at approximately one-minute cadence: the plan specifies cadence, jitter, and catch-up limits; a distinct why for approximately one minute is not articulated. NOT RECOVERABLE FROM PLAN

- Per-aviary serialized state transition: the rationale is ordered, deterministic canonical updates and retry safety.

- Tick acquisition of lease/state/config/timezone/weather/events: the rationale is to compute deterministic transitions from prior state, server time, and versioned calibration.

- Normalized accepted host events: the rationale is to turn client facts into bounded signals before drift/mood application.

- Elapsed-time advancement for local-time phase, weather, mood, calls, perch, responses, reactions: the rationale is continuity while user is away and while server tick never pauses.

- Positive personality deltas with caps: the rationale is slow expressive drift with per-tick, daily, and session limits, never negative absence effects.

- Sparse notebook generation: the rationale is noteworthy observations subject to per-aviary sparsity limits, not raw event logs.

- Atomic tick commit: the rationale is to keep state, cursor, notebook entries, and metadata consistent.

- Aggregate tick metrics without account or bird dimensions: the rationale is operational visibility without private simulation dimensions.

- Bounded delayed-tick catch-up: the rationale is to advance elapsed-time transitions without replaying arbitrary client time.

- Quarantine malformed events: the rationale is to preserve canonical state and continue with safe events, "never reset a bird."

- Normalized scalar personality traits: the rationale is versioned seed/configuration and server-only storage.

- Versioned calibration configuration: the rationale is that formula/coefficient changes are migration/calibration decisions and must not silently reinterpret history.

- Signal weighting order: the rationale is to make host presence dominant, listen-in strong and bird-specific, offers small, settle neutral, and absence non-negative.

- Calibration harness: the rationale is to verify one-week measurable drift, three-week user-perceivable but not session-by-session drift, two-week absence non-regression, and one-session caps.

- Mood state machine: the rationale is a small versioned set combining interactions, local time, weather, calls, duration, and personality-derived resistance.

- Account timezone setting: the rationale is deterministic day/night and mood behavior without inferring a new timezone on every request.

- Server-scheduled rare weather: the rationale is bounded mood-relevant ambience that ends automatically and does not become history or dashboard.

- Bounded bird-to-bird response and wary spread: the rationale is preventing one alarm from cascading into permanent distress.

- Greeting selection: the rationale is to vary greeting form by identity, mood, boldness, absence length, time, and seed without creating guilt or penalty state.

- Versioned motif grammar per species: the rationale is recognizable signatures with bounded variation from personality and mood.

- Server-scheduled call intents with client waveform synthesis: the rationale is same semantic state across browsers while retaining safe client-side variation.

- Captions generated from call intent/grammar: the rationale is captions connected to the actual sound played.

### Frontend rendering pipeline

- Single horizontal scene with renderer plus semantic DOM controls: the rationale is an explicit renderer boundary consuming snapshots and returning only pixels/effects while DOM owns controls, captions, narration, dialogs, and settings.

- Background planes, three perch zones, bird layers, and ornaments: the plan describes the composition, but a distinct why for these exact visual layers is not articulated. NOT RECOVERABLE FROM PLAN

- Gentle parallax: NOT RECOVERABLE FROM PLAN

- Birds visible at every responsive width: the rationale is preserving aspect relationships rather than cropping, scrolling, or zooming.

- Server-selected perch, not drag target: the rationale is that perch is a server-selected signal and clients do not mutate domain state.

- Bootstrap first snapshot with critical renderer: the rationale is first-bird visibility and already-in-progress first frame.

- Quiet field before state: the rationale is no spinner, generic loading animation, or fade-from-static, preserving calm/aliveness.

- Non-critical code splitting: the rationale is initial bundle and first-paint performance.

- Bounded local render clock and interpolation: the rationale is smooth visual transitions between server semantic snapshots without client state authority.

- Mood-selected idle micro-motion: the rationale is visible ongoing life while no new snapshot arrives, grounded in mood.

- Stop animation/audio work when hidden and refresh on return: the rationale is pausing client rendering while the server tick continues and avoiding stale interpolation after suspension.

- Reversible settle transition with five-second undo: the rationale is to make settle presentation reversible and tied to the server event/undo window.

- Thin top bar with account/settings, accessibility, notebook, and offer controls only: the rationale is to avoid scene badges, hover tooltips, inline labels, status counters, and overlay icons.

- Top bar fading after stillness: the plan states this behavior; a distinct why is not articulated beyond calm scene presentation. NOT RECOVERABLE FROM PLAN

- Reduced-motion renderer mode: the rationale is accessibility while keeping simulation, mood, calls, captions, notebook, and identity unchanged.

### Audio pipeline

- One bounded WebAudio graph per active session: the rationale is bounded resources, controlled mixing, and no unbounded per-call allocation.

- Procedural calls from motif grammar and seeds: the rationale is species/bird signatures with bounded variation and no recorded audio asset path.

- Simultaneous-call mixing: the rationale is independent recognizable signatures across chorus density.

- Listen-in gain ramps: the rationale is focused listening without hard-muting other birds.

- Listen-in ending on click/focus movement: the rationale is clear focus behavior and ramp back to ambient mix.

- Audio unlock on first permitted gesture: the plan states this behavior and no intrusive permission announcement; a distinct why is not articulated. NOT RECOVERABLE FROM PLAN

- Graceful silence and captions by default on WebAudio failure: the rationale is no recorded fallback and accessible no-audio behavior.

- Song-fragment offers using procedural path: the rationale is consistency with the call grammar and no recorded audio path.

- Chorus density testing: the rationale is recognizability and CPU stability at the cap.

- Captions positioned near calling bird and faded with call: the rationale is connection to the actual call and non-interruptive accessibility.

### Accessibility and content surfaces

- Semantic DOM bird focus targets: the rationale is keyboard and screen-reader access to the scene in scene order.

- Keyboard controls for birds, listen-in, and exit: the rationale is keyboard operation and focus movement behavior.

- Offer and settle controls with visible names and keyboard paths: the rationale is accessible operation.

- Throttled `aria-live` narration: the rationale is naturalist prose without flooding the screen-reader queue.

- Narration priorities for return-greeting, offer reaction, and settle: the plan states priorities but not a distinct why for this ordering. NOT RECOVERABLE FROM PLAN

- Visual narration when accessibility surface calls for it: the rationale is the same naturalist prose experience visually and audibly.

- Capitalized/direct system errors and revocation surfaces: the rationale is the product/system voice split and actionable language.

- Runtime call captions available when audio is off or fails: the rationale is no-audio accessibility with captions generated from the call grammar.

- Caption/narration layout avoiding focus indicators and birds: the rationale is not obscuring the primary scene or focus state.

- WCAG AA contrast and visible focus across palettes: the rationale is accessibility in bright, dim, settled, day/night, and weather states.

- Accessibility test matrix: the rationale is release confidence across keyboard-only, screen-reader, reduced-motion, no-audio, zoomed/narrow, high-contrast/forced-colors, touch, and visitor revocation paths.

### Performance, observability, and privacy controls

- Initial JavaScript under 2 MB gzipped: the rationale is tied to first-paint and critical-bundle release gates; no separate why for this exact number is articulated. NOT RECOVERABLE FROM PLAN

- First bird visible under 500 ms on mid-tier mobile/4G: the rationale is the first-frame aliveness promise and bootstrap snapshot/critical renderer.

- 60 fps idle motion for 30 minutes on five-year-old mid-range laptop: the rationale is sustained calm motion without performance degradation.

- No meaningful client memory growth in 30 minutes: the rationale is bounded and released audio nodes/buffers, workers, and notebook references.

- Simulation-tick latency p99 below five seconds: the rationale is operational continuity; the exact threshold's why is not separately articulated. NOT RECOVERABLE FROM PLAN

- Last two major versions of Chrome, Safari, Firefox, and Edge: the rationale is defined browser support with older browsers receiving a clear unsupported-browser page rather than a compatibility bundle.

- Synthetic browser checks and aggregate-only RUM: the rationale is operational measurement without account UUID, bird ID, email, event type, personality, mood, notebook prose, or interaction history.

- Telemetry separated from simulation store: the rationale is privacy and permission separation.

- CI and staging checks: the rationale is release-blocking verification of first paint, budgets, fallback behavior, revision gaps, idempotency, retries, presence correctness, monotonic drift, deletion, visits, and serialization.

- Automated scans rejecting gamification terms/components and email/account identifiers in logs or telemetry: the rationale is preventing accidental game/social surfaces and privacy leaks.

### Rollout and operational plan

- Milestone A, domain contracts and simulation fixtures: the rationale is to establish versioned schemas, invariants, calibration, idempotency, timezone, and worker leases, with invariant tests before UI work.

- Milestone B, first-paint scene and bird engine: the rationale is to validate the critical renderer, stable identity, tick, mood, perch/pose, audio grammar, seven-bird capacity, greetings, absence behavior, idle motion, and three-week drift.

- Milestone C, host interactions and notebook: the rationale is to add presence, listen-in, offers, settle, notebook, day/night, weather, and accessibility surfaces while verifying absence/tab-close neutrality and no session-level personality rewrite.

- Milestone D, identity, sync, privacy, and visit path: the rationale is to add auth, revocation, email change, export, deletion, multi-device behavior, visits, and off-by-default notification settings, with visitor tests against mutation and presence leakage.

- Milestone E, performance and release hardening: the rationale is to gate release on browser, mobile, memory/frame, WebAudio, accessibility, deletion/privacy, worker load, chaos, numerical budgets, and first-frame motion.

- Bird-count ramp configuration flag: the rationale is to validate seven-bird CPU/audio/recognizability capacity before broadening availability.

- Third-bird and later age-based offers: the rationale is availability based on configured aviary age, not visits, interaction counts, scores, or payment.

- Quiet offer UI for available bird: the rationale is to avoid reward, milestone, counter, achievement, and progress framing.

- Day-one operations alerts: the rationale is operational awareness for tick p99, worker leases, snapshots, auth links, visitor revocation, first-bird budget, audio errors, memory/frame, and deletion failures.

- Runbooks for quarantined event replay, snapshot-cache rebuild, and soft-deleted account restore: the rationale is operational recovery from canonical state with "no client-side state repair command."

### Risks and mitigations

- Drift calibration risk: the rationale for mitigations is avoiding too-fast, too-slow, or negative drift while preserving private per-bird history and hiding trait numbers.

- Multi-device event risk: the rationale for mitigations is preventing lost or duplicated events, stale snapshot overwrites, client absolute state, and last-write-wins.

- Audio risk: the rationale for mitigations is avoiding canned, uncanny, or overwhelming chorus behavior while preserving procedural signatures and safe silence fallback.

- First-frame risk: the rationale for mitigations is keeping the first-frame aliveness promise with bootstrap snapshot, small critical bundle, non-zero pose phases, and quiet field.

- Accessibility risk: the rationale for mitigations is making narration, captions, reduced motion, focus, contrast, and no-audio behavior release-blocking rather than later fallback.

- Presence inflation risk: the rationale for mitigations is preventing background tabs, idle machines, and visitors from producing host presence.

- Privacy leak risk: the rationale for mitigations is preventing identifiers, exports, logs, or telemetry from exposing account-linked data.

- Feature creep risk: the rationale for mitigations is preserving a quiet aviary rather than scores, streaks, profiles, discovery, chat, co-presence, notifications, stats, or custom scenes.

- Tick load or cascade risk: the rationale for mitigations is preserving continuity and stable bird identities when load, weather, calls, or malformed input become pathological.

### Definition of done

- New account sign-in: the definition of done requires this capability, but a distinct why is not articulated. NOT RECOVERABLE FROM PLAN

- Meeting two named birds already moving: the rationale is the calm, already-in-progress first-frame promise.

- Returning later to a server-advanced canonical aviary: the rationale is server continuity while away.

- Presence affecting slow expressive drift without visible stats: the rationale is expressive drift without gamified or numeric personality surfaces.

- Listen-in/offers/settle use: the rationale is host interaction with bird-specific signals, bounded offers, and settle quieting/closing presence.

- Sparse specific notebook observations: the rationale is naturalist observation without raw events or numeric trait changes.

- Same state from another signed-in device: the rationale is canonical multi-device snapshots.

- Specifically invited visitor read-only view and revocation: the rationale is real-host-aviary viewing without affecting host state.

- Screen-reader narration, captions, keyboard operation, reduced motion, and WebAudio silence fallback as first-class experiences: the rationale is accessibility as release-blocking.

- Numerical performance, memory, accessibility, privacy, sync, and deletion gates: the rationale is release completeness through measurable gates.

- No excluded game, pet-care, native, or social-network mechanics: the rationale is preserving the scoped quiet aviary and preventing feature creep.
