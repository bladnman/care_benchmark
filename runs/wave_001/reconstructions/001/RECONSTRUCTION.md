## System-level intent

- Continuing place, not progression: The opening says Pocket Aviary is "a relationship with a continuing place, not a progression system." This shows up again in the ban on "welcome toasts, streaks, badges, points, or engagement notifications," the out-of-scope list for "leaderboards" and "achievement/streak/visit-frequency surfaces," and the v1 exit decision that the session must feel like "a continuing aviary rather than a loading screen, dashboard, or obligation loop."

- Server-canonical, client-humble simulation: The plan repeatedly keeps authority on the server: "The server is the only writer" of personality, mood, and canonical scene state; clients "submit facts/events and render snapshots"; the read API cannot "accept state replacement"; the interaction API cannot "mutate birds"; and personality/mood/scene have "no PATCH endpoint at all."

- Presence means real host attention: Qualifying presence requires visible document, focused window, and recent pointer/key activity. The plan stresses that "a visible open tab alone never creates presence time," that "visitor activity never creates host presence," and that heartbeats are "facts, not an invitation to count raw tab-open time."

- Slow expressive growth without guilt: Personality changes "only slowly and only toward expressive values." Absence cannot "decrease personality traits or create sickness/distress," the out-of-scope list rejects "hunger/death/decaying happiness," and validation includes tests against "negative drift or guilt UI."

- Durable bird identity: A bird has a "durable opaque ID independent of name/species"; renaming, another-device login, and content changes "must not replace that identity or its vector history." Rollback likewise cannot replace bird IDs or regenerate birds from logs.

- Naturalist observational product voice: The product view is "naturalist, lowercase, specific, and observational." Account/auth/sync-error/accessibility settings are "direct matter-of-fact language," while notebook and narration prose are observable, lower-case, present-tense, and never expose raw numbers, mood labels, streaks, or user behavior.

- Privacy by domain boundary and minimization: "Per-account interaction data remains inside the simulation domain." Operational telemetry is aggregate only and cannot contain names, account IDs, emails, events, state, prose, or reconstructable relationship history. Email and invite recipient email are encrypted, and deletion erases account-owned primary records, event/history rows, and operational identifiers.

- Procedural sound with accessible equivalents: Calls are "generated procedurally in the browser"; recorded fallbacks and loop libraries are prohibited. The same grammar instance supplies captions, and accessibility surfaces are synchronized to the same normalized snapshot as the renderer rather than becoming a static fallback.

- Calm immediate scene, sparse chrome: The frontend must draw the quiet field or first bird without waiting for noncritical bundles, and "never show a spinner, static app shell, fade-in, or wake-up sequence." The scene has sparse top-bar controls, no badges, no inline labels, no app chrome, calm natural colors, and birds already "mid-action."

- Resilient convergence and lifecycle control: Multi-device state is built from append-only facts, server sequence, idempotency keys, state versions, ETags, stale-client recovery, transactionally processed ticks, and rollback-compatible schemas. Export, session revocation, soft deletion, restore, hard deletion, and a deletion ledger are part of the same lifecycle intent.

## Per-feature whys

### Delivery boundary and product invariants

- One canonical, server-simulated aviary per signed-in account: The plan's rationale is convergence and canonical relationship state: the server is the only writer, clients render snapshots, and multi-device interactions append facts rather than replacing state.

- Two starter birds: NOT RECOVERABLE FROM PLAN

- Age-gated growth to a hard maximum of seven: The plan ties the cap to voice allocation, gain/chorus budgets, renderer allocation, seven-bird stress mixes, mobile frame time, audio failure gates, and the refusal to use an "engagement-based unlock."

- Email magic-link accounts: The plan grounds this in verified email, non-enumerating responses, single-use 15-minute links, hashed sessions, account lifecycle, and encrypted email handling without email-derived IDs.

- Multi-device state convergence: The rationale is to let a phone and laptop submit concurrently while durable server sequence, idempotency, state versions, and stale-snapshot discard ensure each event is consumed once and interpolation moves only forward.

- Interactive aviary: The plan makes the aviary the primary experience: birds are in frame, mid-action, gently moving, moodful, responsive to listen-in/offer/settle, and not replaced by dashboard, labels, counters, or app chrome.

- Field notebook: The notebook gives sparse, durable, meaningful naturalist observations from "unusual greeting ordering," "sustained quiet," weather/mood combinations, or distinctive reactions, without trait numbers, timestamps, streaks, or user-behavior reports.

- Quiet read-only visits: The rationale is deliberate sharing without social mechanics: email-bound capabilities are revocable, read-only, no-store, private to the host log, and never create co-presence, notifications by default, host presence, or simulation events.

- Accessibility surfaces: The plan treats accessibility as a release surface: keyboard, screen-reader, reduced-motion, and call-caption experiences preserve all interaction meaning and use the same normalized snapshot and actual call grammar.

- Account/privacy controls: Export, soft/hard deletion, restore, revocable sessions, encrypted email, short-lived export links, and deletion ledgers give owner-authorized lifecycle control while preventing account data from entering telemetry.

- Browser support for the last two major Chrome/Safari/Firefox/Edge releases: NOT RECOVERABLE FROM PLAN

### Service architecture and responsibilities

- TypeScript web client: NOT RECOVERABLE FROM PLAN

- Small modular service deployment: The plan uses this so the public API can stay stateless while all durable simulation work sits behind transactional storage and a scheduled worker.

- Edge/document delivery: It serves a minimal app shell and authenticated snapshot envelope from a nearby edge to improve first render while avoiding persistence or caches that expose another account's state.

- Identity/account service: Its rationale is to isolate magic links, encrypted verified email, lifecycle, device sessions, settings, export, and deletion while prohibiting email as a foreign key, log identifier, telemetry dimension, or queue partition.

- Aviary read API: It authorizes host or visitor capability and returns compact canonical snapshots, ETags, versions, and scene context without accepting replacement state or calculating personality in the request path.

- Interaction ingest API: It validates, deduplicates, sequences, and appends host facts so clients cannot directly mutate birds, generate drift, or submit visitor interaction facts.

- Simulation worker: It claims due aviaries and atomically advances tick, mood, state, and notebook eligibility so persistent simulation changes happen transactionally and not in the request path.

- Visit service: It issues, revokes, and validates encrypted email-bound capabilities and records a private host visit log while blocking co-presence, default host notifications, and visitor-created simulation events.

- Client renderer/audio runtime: It renders/interpolates snapshots and creates procedural calls, but cannot become a second source of truth or retain mutation queues across logins.

- Relational primary store with append-only events and scheduled-work queue: The rationale is row-level transactional guarantees, ordered event history, due-aviary processing, and queue keys limited to synthetic account and aviary IDs.

- Envelope encryption for email and invite recipient email: The plan uses this to keep personally identifying email material protected at rest and outside account references, logs, telemetry, and queue partitions.

- Short-lived aviary tick leases with one transaction for state and `last_processed_event_seq`: The rationale is to prevent overlapping workers from processing the same event range and to avoid skipped or reordered events.

- Durable outbox limited to account mail and export/deletion jobs: It supports permitted mail and lifecycle jobs without copying simulation data into analytics.

- Client/server boundary for ephemeral presentation state: The client owns interpolation, audio nodes, top-bar visibility, local focus, settle undo, and idempotent submission queue because these are presentation concerns; account, invitations, canonical aviary, bird, mood, history, notebook, and visit logs stay server-owned.

- WebGL retained scene graph with Canvas 2D fallback: The rationale is performant rendering across feature-detected browser capabilities while keeping the authoritative data model unchanged.

- Normalized snapshot for renderer input: The renderer receives normalized snapshot data rather than database-shaped records so scene presentation is decoupled from persistent storage shape.

### Canonical data model

- UUIDs and opaque random IDs: The plan uses them so account and bird identity are durable and not email-derived or name/species-derived.

- `accounts`: Encrypted verified email, timezone, lifecycle, created/deletion dates, and no email-derived IDs support privacy, local-time simulation, and account lifecycle control.

- `device_sessions`: Hashed tokens, issued/last-used/revoked timestamps, and coarse device labels support revocable device access without storing raw tokens.

- `aviaries`: State version, tick times, local-time configuration, day/night/weather/settled state, and deterministic seed/version support canonical snapshots, scheduled simulation, local scene behavior, and reproducible scene choices.

- `birds`: Stable bird ID, current name, species, age-gate state, perch intent, personality vector, mood, and visual/call seed support durable identity and expressive behavior while keeping trait scalars out of normal product UI.

- `bird_trait_history`: Server-only sampled calibration/audit history exists for calibration and audit inside the account domain, not as an analytics source.

- `interaction_events`: Append-only sequences with idempotency keys and processed ticks support validation, deduplication, durable ordering, and once-only simulation consumption.

- `presence_windows`: Server-accepted intervals derived from heartbeats store only what simulation needs and avoid exposing visit-frequency history to host UI.

- `scene_state`: The rationale is to keep compact canonical positions, current actions, call scheduling, weather, and greeting context derived and replaceable, unlike persistent personality.

- `notebook_entries`: Immutable append-only entries persist final prose and source event IDs once so historical observations do not change with later template changes.

- `settings`: Accessibility, reduced-motion, captions, timezone, and visit-notification opt-in support non-product preferences while explicitly excluding a gamification preference.

- `visit_invites`: Email-specific, encrypted, single-use capabilities with expiry/revocation make each visit a deliberate invite and ensure unused invites expire after 30 days.

- `visit_access_sessions` and `visit_log`: These provide read-only capability sessions and host-visible recipient/time/duration facts without feeding presence or simulation.

- `export_jobs` and `deletion_jobs`: These record account-owned exports and soft/hard deletion execution so export links are short-lived, owner-directed, and deletion is scheduled and auditable.

- `state_version` in snapshots: Monotonic versions, server time, transition/tick hints, and schema version let clients reconcile, recover, discard stale snapshots, and avoid hidden numeric vectors in normal product responses.

### Public API and event contracts

- Versioned `/v1` API and JSON schemas: The plan wants versioning from day one to support compatibility, safe deltas/full snapshots, feature-flagged simulation versions, and rollback.

- UUID idempotency keys on all mutations: The rationale is that retries return the original accepted event and do not duplicate presence, offers, or visit invitations.

- Payload, schema, membership, cooldown, and session revocation validation before append: These checks prevent invalid, unauthorized, duplicate, or stale client facts from entering the append-only event stream.

- `POST /auth/magic-links`: Non-enumerating response language, per-email/IP abuse limits, and 15-minute expiry protect account existence and magic-link abuse.

- `GET/POST /auth/magic-links/consume`: Atomic one-use consumption, session issue, and link invalidation prevent link replay.

- `GET /aviary/snapshot`: ETags, state version, server time, render/call schedule, active transition, and narration source data provide compact canonical rendering and accessibility recovery without mutation.

- `POST /aviary/events`: Accepted sequence and cooldown/rejection facts let the client reconcile interaction outcomes while never exposing trait values.

- `GET /aviary/notebook?cursor=`: Pagination in reverse chronological order supports an indefinitely scrollable notebook while no edit/delete endpoints preserve immutable observations.

- `GET/PATCH /account/settings`, `GET /account/sessions`, `DELETE /account/sessions/:id`: These endpoints provide matter-of-fact accessibility/account settings and session revocation.

- `POST /account/export`, `POST /account/delete`, `POST /account/delete/restore`: These endpoints support owner-authorized export, 30-day soft deletion, and restore before irreversible work.

- `POST /visits/invites`, `GET /visits/invites`, `DELETE /visits/invites/:id`: The rationale is deliberate, revocable invitation management that is "never shown during onboarding."

- `GET /visits/:capability/snapshot`: Validation on every pull returns read-only rendering or a matter-of-fact unavailable surface, so revocation reaches the next polling interval.

- `POST /visits/:capability/open` and `POST /visits/:capability/close`: These record only host visit-log facts and approximate duration, with no visitor interaction-event route.

- Pull-first snapshot delivery with optional server-sent invalidation hint: The plan chooses recovery from navigation, visibility return, and long render gaps without making persistent client state depend on a stream.

- `last_seen_state_version`, `304`, compact deltas, and full snapshot recovery: The rationale is efficient delivery where safe and a full canonical recovery path whenever needed.

### Simulation engine

- Due-aviary tick about once per minute with jitter: Jitter avoids a global thundering herd while regular ticks keep scene, mood, and notebook transitions current.

- Worker lock, ordered unprocessed events, bounded transitions, state writes, notebook candidates, consumed range, and state version in one commit: The rationale is atomic canonical advancement with no skipped or reordered events.

- Retry on serialization conflict: This protects correctness when concurrent work collides.

- Bounded elapsed-time catch-up after missed ticks: The plan uses segmented catch-up so time-of-day and mood transitions remain plausible instead of jumping.

- Reproducible scene randomness from `(aviary_id, tick_number, simulation_algorithm_version)`: The rationale is replay of a reported state inside the account domain without treating event history as a substitute for persisted personality.

- Secure randomness only for identifiers/capabilities and deterministic pseudorandom choices for simulation variation: This separates security entropy from reproducible simulation behavior.

- Focus/blur/visibility/pointer/key presence detection: The rationale is to count only qualifying host attention, not open tabs, background tabs, or visitors.

- Server-capped heartbeats with skew rejection and close conditions: These prevent inflated or impossible presence windows and close windows on lost qualification, settle, unload, session expiry, or missing heartbeat.

- Calibrated grace interval of several minutes: The plan says this keeps "a quiet watcher" from being discarded after a short still spell.

- Decayed per-bird signal from qualifying presence and eligible interactions: This is the basis for slow personality drift from actual relationship facts rather than raw tab-open time.

- Bounded low-pass trait delta with remaining headroom: The rationale is slow, capped movement toward expressive values, with mature birds naturally slowing and no single session producing a noticeable jump.

- Listen-in personality contribution: Listen-in primarily contributes warmth and vocal-frequency, giving attention a small expressive effect.

- Offer personality contribution: An accepted offer gives a small curiosity signal and offering nearby gives a small boldness signal.

- Settle affecting mood/scene only: Settle is kept as short-lived mood and scene state rather than a personality-growth mechanic.

- No negative trait delta from absence, neglect, cancellation, or ignored offer: The rationale is to avoid sickness, distress, decay, and guilt loops.

- Calibration fixtures for sparse, typical, and high-but-human presence: These tune normal regular use toward instrument-detectable change around one week and perceptible differences around three weeks while avoiding session jumps.

- Mood finite state machine with minimum dwell: The dwell interval prevents flicker, while probabilistic weights make mood respond to time, weather, interactions, adjacent calls, and personality.

- Mood stored across sessions and ticks: Opening a tab cannot reset mood, preserving continuity.

- Perch intent/action, weather opportunity, and call-window scheduling each tick: These turn mood and personality into observable behavior such as scanning, preening, sound/leaf attention, and low/fluffed poses.

- Rare soft weather: Short rain and occasional wind provide temporary small effects while preserving the "restrained ambient weather" scope.

- Client-only leaves and feathers: They are ornaments and not per-leaf server entities, keeping the canonical model compact.

- Eligible host-return greeting cue: The rationale is a quick, bird-led return moment weighted by boldness/social warmth/mood, executing within one or two seconds without banners or synchronized chorus.

- Settle event with five-second re-engage undo: The plan makes settle a server-visible lighting/audio state that can be cancelled by valid re-engage/click, so it remains optional and undoable.

- Notebook candidate generation and sparsity gate: The plan targets roughly one entry every few days for regular use and more only for noteworthy moments, avoiding logs, streaks, and behavior reports.

- Persisted notebook prose and source event IDs: This keeps historical observations stable rather than live templates that can change history.

- Call grammar runtime contract: The simulation supplies motif family, timbre seed, modifiers, and time window so the browser can procedurally synthesize calls whose signature remains identifiable while captions derive from the same grammar.

- Offset bird-to-bird responses and chorus windows: Offset randomness prevents identical loop starts and chorus collapse.

- Active call voice limits and seven-bird allocation: The plan uses voice caps, gain staging, compression, and server validation to keep audio bounded and performant.

### Sync, integrity, and lifecycle

- Append-only device interactions: They allow concurrent phone/laptop events while server sequence determines order and the simulation consumes each once.

- Per-session sequence for diagnostics only: It can diagnose gaps but "never determines personality order."

- Version and server clock on the read API: These let clients discard stale snapshots and interpolate only forward.

- Rejected-event reconciliation: After cooldown, revoked session, or expired invite, the client removes optimistic presentation and shows system wording only when action needs explanation.

- No last-write-wins for bird fields: The rationale is to avoid losing drift or overwriting canonical bird state; names/settings instead use version/ETag preconditions.

- No PATCH endpoint for personality, mood, or scene: This enforces server-canonical simulation and prevents client-side state replacement.

- Old-client wake recovery: A suspended client drops stale interpolation, pulls a fresh snapshot, and resumes from canonical state.

- Encrypted/ephemeral client cache for offline quiet field: It can render a quiet field offline but cannot queue arbitrary state mutations across logout or treat cached bird state as current.

- Soft deletion and hard deletion workflow: Revoking sessions and visit capabilities, removing the aviary from reads, allowing 30-day restore, and erasing records at hard delete provide lifecycle control without retaining bird or account content.

- Deletion ledger: It proves completion while avoiding retained bird or account content.

### Frontend scene and interaction pipeline

- Minimal HTML, critical CSS, and authenticated snapshot data: The rationale is drawing the quiet field or first bird without waiting for settings, notebook, invite, account bundles, audio permission, or noncritical chunks.

- Calm quiet field when data is late: It avoids spinner, static shell, fade-in, or wake-up sequence and preserves the continuing-place feel.

- Scene-node normalization: Background, sky, perch zones, birds, weather, foreground ornament, and top bar become a retained scene with existing birds mid-action and call scheduling live.

- Time-based interpolation from server timestamps and action envelopes: This avoids frame-count dependence, teleporting, and stale interpolation after long gaps.

- Mood-specific idle micro-motion: Preen, scan, head tilt, and weight shift keep the visible scene alive.

- Stop animation while hidden and pull snapshot before return rendering: This saves hidden work while recognizing that authoritative simulation continues.

- Pointer, touch, and keyboard focus mapped to birds: The plan makes listen-in reachable by clicking, tapping, or focusing birds, with explicit end/rebalance behavior.

- Offers only through top-bar affordance: The rationale is that users cannot direct-manipulate perch placement.

- Settle as several-second lighting/call envelope with in-scene undo: It is an optional gesture rather than a blocking leave flow.

- Responsive layout keeping every bird in frame: Horizontal compression, desktop gaps, and no pan/zoom/scroll/crop preserve the whole aviary across devices.

- Sparse top bar with account/settings, accessibility, notebook, and offer affordances: The rationale is to keep chrome minimal while preserving necessary controls.

- Top bar fade and keyboard-focus protection: It fades after cursor stillness but returns on activity, and focused controls cannot become visually unavailable.

- No badges, inline labels, hover-only tooltips, or app chrome in scene: This keeps the product naturalist and non-dashboard-like.

- Code-split settings, notebook, export/deletion, and invitation management: The rationale is initial-route performance and first-bird speed.

- Avoid large framework UI libraries, raster/audio asset packs, and per-bird DOM animation trees: The plan ties this to route budgets, procedural audio, and performance.

- Calm natural colors and AA-compliant copy across day/night states: The rationale is a design-system input that preserves atmosphere and accessibility.

### Audio and accessibility surfaces

- WebAudio after first browser-permitted user gesture: This respects browser policy while allowing the visual aviary to begin immediately.

- Bounded voice graph per bird with oscillator/noise/sample-free synthesis: The rationale is procedural, sample-free sound with reusable resources and no recorded media.

- Short lookahead audio scheduling: The plan uses it to keep audio bounded rather than scheduling unbounded future work.

- Listen-in gain ramp: The focused bird becomes clearer while other birds remain audible and no track is muted.

- Audio teardown and voice caps: These prevent leaks in long sessions and enforce bounded audio at seven birds.

- Graceful silence plus generated captions when WebAudio fails: The rationale is accessibility and reliability without downloading fallback recordings.

- Audio preference changes without simulation changes: Preference changes alter playback but not call schedules or simulation state.

- Semantic interaction layer synchronized to canvas: The plan provides ordinary labeled controls, tab/arrow/enter/escape flows, keyboard-complete offer and settle, and AA focus indicators.

- Polite live-region narration service: It emits concise naturalist updates every 30-60 seconds, coalesces superseded updates, and gives controlled priority to greeting, offer reaction, and settle without announcement spam.

- Observable narration vocabulary: Narration uses lower-case present-tense prose and never raw perch numbers, mood labels, or personality values.

- Reduced-motion mode: It replaces continuous micro-motion and flight paths with slow pose/perch cross-fades, removes leaf drift, and retains state changes, calls/captions, notebook, and interaction meaning.

- Captions generated at call synthesis time: They reflect the actual motif, pitch, rhythm, and mood, anchor near the caller, and avoid fixed species strings.

- VoiceOver, NVDA, keyboard, contrast, and reduced-motion instrumentation tests: These verify accessibility behavior rather than relying only on snapshot tests.

### Performance, reliability, privacy, and observability

- Initial JS under 2 MB gzip with CI bundle analysis: The rationale is enforcing route budgets.

- First bird visible under 500 ms on mid-tier mobile/4G: The first render must not wait for noncritical chunks or audio permission.

- Idle 60 fps for 30 minutes on five-year-old mid-range laptop: The plan uses frame-time and long-task budgets to keep the aviary viable over long sessions.

- 30-minute soak test for heap, audio, RAF, and offscreen notebook nodes: The rationale is catching client leaks and retained work.

- Simulation tick p99 latency, backlog, lease-conflict, error-rate, and freshness alarms: These detect worker health and stale personality write risk.

- Availability degradation with clear retry path: The plan prefers a clear retry path rather than stale personality writes.

- Aggregate operational instrumentation only: Route/error counts, latency, first-bird timing, frame-time distribution, audio errors, synthetic checks, tick backlog, and anonymous duration bins support operations without relationship data.

- Metric schema rejection for identifiers and high-cardinality per-user tags: This prevents bird IDs, account IDs, emails, names, events, payloads, state versions tied to accounts, and notebook prose from reaching telemetry.

- Separate observability credentials and network access: The rationale is to prohibit warehouse reads from account/simulation tables.

- Logging redaction review: It protects request bodies, magic links, capabilities, and emails before export.

- Security reviews: The plan covers magic-link expiry/single use, capability entropy/hashing, recipient authorization, CSRF/session fixation, rate limits, encrypted PII access, export TTL, deletion races, and immediate revocation.

- Visitor private/no-store snapshot responses with capability validation every pull: This makes revocation effective by the next polling interval.

### Implementation sequence and rollout

- Schemas/migrations, synthetic IDs, encrypted email, magic-link/session lifecycle, deletion skeleton, app shell, and privacy-safe logging first: The rationale is establishing identity, lifecycle, and privacy foundations before simulation features.

- Aviary/bird/event/state models, tick worker, deterministic fixtures, snapshot endpoint, and property tests next: The plan prioritizes canonical state, drift correctness, ordering, stable identity, and no client-vector mutation path before feature-heavy surfaces.

- First two-bird renderer, day/night/weather/three-zone composition, quiet-field loading, interpolation, return cue, visibility recovery, and performance harness before feature-heavy surfaces: The rationale is proving the core scene and performance before adding heavy UI.

- Presence, listen-in, offers, settle/undo, mood/action integration, notebook, and procedural audio/captions after core renderer: These add interaction and expression once canonical rendering exists.

- Accessibility surfaces in parallel with each interaction: The rationale is that accessibility is part of each interaction, not a late static fallback.

- Account settings/export/session management and visit flows after core interactions: The plan sequences lifecycle and deliberate visits after interaction foundations.

- End-to-end multi-device, privacy, accessibility, resilience, soak/performance validation, and content/voice review: The rationale is release readiness across convergence, privacy, accessibility, reliability, performance, and product voice.

- Internal synthetic cohort then small opt-in external cohort: Synthetic accounts and fixture aviaries let the team test before real users; small opt-in rollout keeps the first external release controlled.

- Tick and notebook rules behind versioned configuration: The rationale is controlled rollout and algorithm versioning.

- Aggregate health/performance/error metrics plus consented qualitative research: The plan explicitly avoids aggregating personal interaction behavior to tune drift, using simulator fixtures and direct interviews instead.

- Progressive bird cap rollout through age-based server eligibility: The rationale is to wait for chorus recognizability, mobile frame time, and audio failure gates rather than using engagement prompts, notifications, or growth metrics.

- Backward-compatible schemas, feature-flagged simulation versions, and protected recovery records: The rollback rationale is restoring prior server algorithm/read model without replacing bird IDs or erasing drift.

- Never roll back by regenerating birds from logs or accepting client cache as truth: The plan protects durable identity and server-canonical state.

### Validation matrix and principal risks

- Drift-rate validation: Deterministic multi-week fixtures, rate caps, headroom, human review, and hidden trait values exist so drift does not feel like a counter or change too fast.

- Neglect/guilt validation: Nonnegative trait-delta property tests and copy/UI review prevent absence from becoming decay, streaks, missed-session surfaces, or guilt UI.

- Concurrent-device validation: Transactional event sequencing, leases, idempotency fuzzing, suspend/resume tests, and no absolute personality endpoint mitigate lost drift.

- Presence-inflation validation: Browser lifecycle tests, server caps/timeouts, and overnight background scenarios enforce all qualifying signals.

- Outage/missed-tick validation: Segmented catch-up, deterministic replay, version reconciliation, stale-client recovery, and backlog alarms prevent state divergence.

- Call-quality validation: Listening tests, novelty checks, gain/voice budgets, seven-bird mixes, and no recorded-loop path mitigate repetitive or collapsed chorus behavior.

- Audio-leak validation: Captions fallback, AudioWorklet/node soak tests, heap/voice counters, and 30-minute CI profiles mitigate audio failure and leaks.

- Accessibility validation: Narration cadence, generated captions, reduced-motion cross-fades, keyboard flows, and assistive-technology user review prevent accessibility from becoming a static fallback.

- First-render validation: Synthetic 4G first-bird gates, inline snapshot/quiet field, and visual review against spinners/fades/entry animations prevent a loading-app feel.

- Invitation validation: Entropy, hashed capabilities, no-store responses, validation per pull, revocation tests, 30-day expiry, and recipient-bound flow mitigate leaks or revocation delay.

- Telemetry validation: Metric allowlists, redaction tests, warehouse isolation, log sampling, and deletion/export audits keep private relationship data out of telemetry.

- Scope-creep validation: Architecture/API review rejects profile, feed, chat, and counter primitives, and release checklist checks surfaces against non-goals.
