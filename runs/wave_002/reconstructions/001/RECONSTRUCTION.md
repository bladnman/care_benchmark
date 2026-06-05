## System-level intent

1. A quiet place, not a conventional app surface. The plan repeatedly frames Pocket Aviary as "one calm horizontal aviary scene" and later as "web-only and quiet by design." This shows up in Product Boundary and v1 Scope, Client Rendering Pipeline, Product Invariant Tests, and Key Implementation Invariants, especially the exclusions of gamification, public feeds, UI chrome in the scene, and "textual welcome surfaces."

2. The aviary should feel like it continues without the viewer. The plan states that "the product must feel like a place that continues without the viewer" and says this "drives the system architecture." This intent appears in server-owned canonical state, the simulation worker that "runs independently of connected clients," the server-side tick, mood persistence, return greeting from server state plus absence length, and the rule that opening the tab should never reset a bird.

3. Clients render and submit facts; the server authors life. The most explicit boundary is: "clients can submit facts about user interaction, but only the simulation worker converts those facts into mood/personality changes." The same principle appears in Architecture Overview, Render Boundary, Sync and Conflict Model, Data Model, and Key Implementation Invariants: clients never write personality vectors, mood rows, or canonical state versions.

4. Identity and behavior should be stable, slow, and recognizable. The plan wants "stable identity-specific variation," persisted personality and mood, stable call signatures, "a user's long-term recognition of Pip's call," and drift that is measurable after about one week but user-visible after about three weeks. This appears in Product Boundary, Simulation Engine Design, Audio Pipeline, and Rollout Plan.

5. Interaction must be non-punitive and not gameable. The plan excludes "Tamagotchi mechanics," "achievements, streaks, scores, badges, levels, XP," and "punitive neglect." It also says absence must not create negative personality drift, offers should be unavailable quietly, and cooldowns are "functional drift protection" that should work "without scolding copy."

6. Privacy commitments are architectural rules. The plan uses that exact phrase in Privacy, Security, and Data Governance. It appears earlier in synthetic UUID discipline, encrypted email fields, keyed hashes, aggregate-only telemetry, no raw email in telemetry keys, no per-bird state in analytics, and no ML/model training from this data.

7. Product voice is split by surface. The plan distinguishes naturalist product surfaces from system surfaces: notebook entries use "lowercase, present tense, bird-named, specific" voice, while auth, settings, sync errors, export, delete, and unsupported-browser surfaces use "matter-of-fact voice." This split appears in Notebook Generation, Accessibility Surfaces, API Surface, Risks, and Product Invariant Tests.

8. Accessibility is part of product quality, not a fallback. The plan says "Accessibility ships in v1 and is tested as part of product quality," and later says accessible surfaces must not become "semantic fallbacks rather than the actual product." This principle appears in screen-reader narration, captions, reduced motion, keyboard navigation, visible focus indicators, and accessibility audits.

9. Performance budgets are product boundaries. The plan says to "track budgets from day one" and lists concrete limits for bundle size, first-bird render, idle motion, memory growth, snapshot size, and simulation tick p99. This intent appears in the architecture, renderer dependency choice, initial load path, audio runtime, rollout, tests, and risk mitigations.

10. V1 exclusions are invariants, not backlog. Product Boundary states that excluded items should be treated as "product invariants, not postponed backlog." This intent recurs in Product Invariant Tests and Key Implementation Invariants, where violations are called "a product change" that should be rejected for v1.

## Per-feature whys

### 1. Product Boundary and v1 Scope

- Browser-only, single-account, single-aviary product: Why: it protects the "core experience" of one calm horizontal scene and matches the invariant that "V1 is web-only and quiet by design."

- One calm horizontal aviary scene: Why: the product should feel like "a place that continues without the viewer" rather than a dashboard or social surface.

- Server-owned canonical aviary state: Why: the plan says this requirement "drives the system architecture"; the server owns state so clients do not author personality or mood.

- Two starter birds with slow growth to a maximum of seven birds: Why: growth is slow and age-based, not engagement-based; the rollout also says initial launch with two birds protects "audio recognizability and simulation load."

- Two starter birds selected by the system from a small species pool of about six species: NOT RECOVERABLE FROM PLAN

- Bird naming at adoption: NOT RECOVERABLE FROM PLAN

- Bird rename from bird settings: Why: the bird can be renamed "without changing identity/personality," preserving stable bird identity.

- Hidden server-persisted personality vectors: Why: personality is server-only and "not returned numerically to clients"; the plan makes "personality vectors are hidden forever" a key invariant.

- Fast-timescale mood enum with timers and transition causes: Why: mood is "fast-timescale, persisted, and visible through behavior," and opening the tab should never reset a bird to neutral.

- Server-side tick at about one minute cadence: Why: the tick lets aviaries advance while disconnected and lets the worker consume events, apply mood transitions, drift, weather, day-night state, and notebook candidates.

- Append-only interaction events: Why: they let clients submit facts while preserving ordered server-side conversion into mood/personality changes; the tick consumes events in sequence and can be idempotent under retry.

- Procedural client-side calls with recognizable per-bird call signatures: Why: a user's "long-term recognition of Pip's call" depends on stable per-bird identity, and recorded fallback is prohibited.

- Listen-in audio mix rebalancing without muting other birds: Why: the focused bird rises while other birds lower to ambient, "never silent," preserving the aviary as a living place rather than isolating one sound.

- Offer interactions with per-bird cooldowns: Why: cooldowns are "functional drift protection" that prevent repeated trait-inflating events and keep a single session from saturating curiosity or boldness.

- Settle gesture: Why: settle quiets mood and "cleanly ending presence" without creating directional personality drift.

- Five-second settle undo window: NOT RECOVERABLE FROM PLAN

- Field notebook with sparse, auto-generated, read-only naturalist observations: Why: entries should be stable historical prose, not raw event logs, and should avoid "achievement" language, numeric traits, and user visit-frequency phrasing.

- Optional visit invitations by email: Why: a named friend can view read-only while visitors cannot affect the host's aviary; visit logs are for transparency and operational accounting, not drift.

- Off-by-default visit notifications: Why: the plan bars badges, prompts, default notifications, and user-facing visit-frequency surfaces, keeping visits from becoming engagement mechanics.

- Screen-reader narration, reduced motion, call captions, keyboard navigation, focus indicators, and WCAG AA contrast: Why: accessibility is "first-class," ships in v1, and must preserve charm as product quality.

- Performance budgets: Why: first bird visibility, idle frame rate, memory stability, payload size, and tick latency are needed to keep the quiet place responsive and stable.

- Aggregate-only operational telemetry: Why: telemetry must never include per-bird state or per-account interaction history; operational curiosity must not cross the privacy boundary.

### 2. Architecture Overview

- TypeScript-first web stack: Why: it keeps schemas and domain types shared across client, API, and simulation worker while keeping runtime boundaries strict.

- Web client component layer plus dedicated aviary renderer module: Why: ordinary UI surfaces and the scene renderer remain separate, reinforcing the boundary between chrome and aviary.

- Presence detector and event submitter: Why: clients submit interaction facts while server-side simulation remains the only converter into drift and mood.

- Snapshot interpolator: Why: it treats server snapshots as canonical and keeps client animation as presentation only.

- Edge/web app service with smallest possible initial bootstrap: Why: it supports first paint and early authenticated rendering while protecting the first-bird performance budget.

- API service with matter-of-fact error surfaces and machine-readable error codes: Why: system surfaces use matter-of-fact voice and stable codes for sync/auth/account failures.

- Simulation worker: Why: it owns the server-side tick, advances due aviaries independently of connected clients, and writes canonical state versions.

- Email worker: Why: it separates magic links, invites, export links, and optional notifications while storing send metadata by synthetic account ID plus delivery ID so raw email is not used in telemetry keys.

- PostgreSQL system of record: Why: row-level constraints and transactions enforce one aviary per account, bird cap, single writer of personality state, and event ordering.

- Queue/lock layer with per-aviary tick lock: Why: it prevents concurrent simulation updates; the first version can be database-backed but should be swappable before scale pressure.

- Observability with aggregate request/timing/error metrics and RUM: Why: operations need timing, frame, audio, memory, and tick signals without per-bird or per-account interaction telemetry.

### 3. Repository and Module Organization

- Organize around domain boundaries, not framework folders: Why: simulation, rendering, audio, narration, and notebook generation need pure functions with deterministic inputs, while the API layer should orchestrate instead of containing domain logic.

- `packages/domain`: Why: shared domain types, constants, schema validators, voice taxonomy, and event names keep client, API, and worker schemas aligned.

- `packages/simulation`: Why: drift, mood transitions, greeting selection, weather/day-night derivation, and notebook candidate selection must be pure and have no database access.

- `packages/renderer`: Why: scene graph, interpolation, reduced-motion rendering, pose selection, and caption placement belong in a renderer boundary separate from UI and canonical simulation.

- `packages/audio`: Why: WebAudio synthesis, call grammar, listen-in mixing, and caption fallback are a distinct bounded runtime.

- `packages/accessibility`: Why: narration, keyboard navigation, focus contract, and live region scheduling are product surfaces that need their own shared contract.

- `packages/privacy`: Why: redaction helpers, telemetry guardrails, and PII scanners protect the privacy commitments that are architectural rules.

- `packages/testing`: Why: deterministic seeds, simulated clock, fake audio context, fixtures, and snapshot builders support calibration and invariant testing.

### 4. Data Model

- Encrypted account email and keyed email hash: Why: email is verified and usable for lookup while raw email is not used as an internal identifier.

- Synthetic UUID account identifier: Why: it is the only account identifier used everywhere except encrypted account email, including services, logs, queues, and telemetry.

- Account status with soft delete and hard delete queue: Why: deletion hard-deletes account-tied records after 30 days while allowing recovery during the soft-delete window.

- Magic links with 15-minute one-time token hash: Why: links expire quickly, invalidate on use, and are hashed at rest.

- Generic magic-link request success: Why: the response is generic regardless of account existence, preventing the endpoint from revealing account existence.

- Requested IP/user-agent hashes for magic links: Why: they may be needed for abuse controls, not product telemetry.

- Per-device sessions: Why: sessions are revocable and never encode personality or aviary state.

- `device_label`: NOT RECOVERABLE FROM PLAN

- One `aviaries` row per account with unique account ID: Why: it enforces one canonical aviary per account.

- `local_timezone` and timezone override: Why: local timezone is used for day/night interpretation.

- `bird_count_cap`: Why: it stores forward compatibility while enforcing the v1 cap.

- `settled_until_reengagement`: NOT RECOVERABLE FROM PLAN

- Bird stable UUID, ordinal, and adopted timestamp: Why: bird identity and adoption order persist, but ordinal is "not surfaced as a score/counter."

- Bird perch zone, pose key, motion phase, and call seed: Why: snapshots expose derived render parameters without exposing hidden personality numbers.

- Bird personality version: Why: personality is server-only, hidden from clients, and updated as canonical state.

- Bird mood transition reason: Why: it is internal debugging information and not user copy.

- `aviary_state_versions`: Why: canonical snapshots are versioned, compact, render-ready, and exclude hidden numeric personality values.

- Notebook unread count omitted from snapshot payload: Why: "Notebook access is user-initiated" and unread count "should not be invented."

- Interaction event server-assigned per-aviary sequence: Why: event ordering lets the tick consume events in sequence and avoid last-write-wins personality or mood paths.

- Host presence event payload evidence flags: Why: the server validates visibility, focus, activity, cadence, timezone, and clock deltas but "does not try to infer gaze."

- Visitor event types limited to snapshot opened and closed: Why: visitor events are only for visit log/transparency and operational accounting, and "never become drift inputs."

- Notebook entries store final prose: Why: the notebook stays stable when a user reads historical entries.

- Notebook dedupe window key: Why: generation is sparse and noteworthiness-key dedupe prevents repeated entries.

- `bird_offer_cooldowns`: Why: cooldowns block repeated trait-inflating events without scolding copy.

- Visit invite encrypted visitor email and visitor email hash: Why: visitor email must be stored for invites and display while preserving encryption and lookup discipline.

- Visit log reachable in account settings only: Why: visits should produce no badges, prompts, or default notifications.

- Account settings for call captions, reduced motion, audio, visit notifications, narration, and timezone: Why: these settings expose accessibility, audio, visits, and day/night control without exposing personality vectors.

### 5. API Surface

- Mutating requests accept idempotency keys: Why: clients can retry event submits after outages without double-applying state changes.

- Stable error codes: Why: error surfaces are matter-of-fact and machine-readable.

- `POST /auth/magic-link`: Why: it rate-limits by email hash and request origin, creates a 15-minute one-time token, and returns generic success regardless of account existence.

- `GET /auth/magic-link/consume`: Why: it validates unexpired unused tokens, creates accounts if needed, creates sessions, and invalidates tokens.

- `GET /account`: NOT RECOVERABLE FROM PLAN

- Verified email change endpoints: NOT RECOVERABLE FROM PLAN

- Session revocation endpoint: Why: device sessions are per-device and revocable.

- Account export endpoint: Why: export jobs email the verified address a download link and are part of the account lifecycle/privacy surface.

- Account deletion and cancel endpoints: Why: deletion starts a 30-day soft deletion, with restoration during the soft-delete window.

- `GET /aviary/snapshot`: Why: it is used on navigation, visibility return, keepalive, and long frame-gap recovery; it must not expose numeric personality vectors or raw event history.

- `GET /aviary/bootstrap`: Why: it returns the smallest render-critical snapshot to draw the first bird quickly.

- `GET /aviary/notebook`: Why: notebook access is paginated and user-initiated.

- `GET /aviary/birds`: Why: it returns only bird IDs, names, species, and derived user-facing descriptors.

- Bird rename endpoint: Why: it renames without changing identity/personality.

- Batched interaction event ingestion: Why: the server assigns sequence numbers, validates actor permissions, stores append-only events, and returns accepted event IDs.

- Listen-in start/end convenience endpoints: Why: they wrap event creation and mix/reason recording without letting the client commit drift.

- Offers endpoint: Why: the server checks cooldowns and current state, returns immediate reaction seed, and leaves mood/drift effects for the tick.

- Settle and settle undo endpoints: Why: settle records the start and returns undo/lighting instructions; undo is valid only within five seconds.

- Visit invite endpoints: Why: hosts create 30-day invites, inspect status and logs, and revoke immediately.

- Visit consume endpoint: Why: visitors get sessions only if tokens are valid, not expired, and not revoked.

- Visitor snapshot endpoint: Why: visitors receive a read-only snapshot without interaction affordances, notebook mutation, host-only settings, or personality values.

- Visit close endpoint: Why: it records best-effort visit duration.

- Revoked, expired, or unavailable visit copy: Why: the plan wants matter-of-fact copy such as "This visit is no longer available."

### 6. Simulation Engine Design

- Due-aviary tick loop: Why: it loads canonical state and unconsumed events, computes time/weather, applies mood and drift, writes snapshots, notebook entries, cooldowns, and consumed markers in one transaction.

- Tick idempotence under retry: Why: failed transactions should not mark events consumed, and retry after commit should avoid double-applying events.

- Seeded PRNG streams: Why: they allow reproducible tests and avoid repeated canned patterns.

- Presence requires visibility, focus, and recent pointer/key activity: Why: presence inflation would overcount background/open tabs; watching quietly is real use, so the activity window starts conservatively and calibrates toward the longer side.

- Host-only presence drift: Why: visitor presence never feeds drift, preserving read-only visits.

- Personality drift as a slow low-pass filter: Why: changes should be instrument-measurable after about one week, user-visible after about three weeks, and no single session should cause visible trait movement.

- Presence-time dominant drift: Why: regular presence is the main signal for gradual expressive trait movement.

- Listen-in drift contribution: Why: listen-in duration nudges social warmth and vocal frequency upward for the listened bird.

- Offer drift contribution: Why: accepted/investigated offers nudge curiosity, and offers near a bird nudge boldness slightly.

- Settle drift behavior: Why: settle quiets mood and cleanly ends presence but does not create directional personality delta.

- No negative drift for absence: Why: absence must not become punishment, suspicion, lower saturation, decaying warmth, or stored trait loss.

- Per-trait maximums and daily/weekly caps: Why: high-activity users should not max traits quickly and caps should not be surfaced to users.

- Mood state machine: Why: mood is fast, persisted, visible through behavior, and weighted by personality, time, weather, recent interactions, and bird-to-bird interactions.

- Return greeting selection: Why: the bird greeting is the welcome surface, so the algorithm chooses one bird and sometimes a staggered second response, never all birds simultaneously.

- No textual welcome copy: Why: "The bird greeting is the welcome surface."

- Offers launched from top bar, not by clicking birds directly: Why: the aviary itself has no embedded UI buttons or hover controls, and users do not control bird placement or perches.

- Seed offer type: Why: it gives birds a curiosity/mood-shaped reaction such as investigates, eats, waits, or watches.

- Song fragment offer type: Why: it lets the client play a motif softly and lets the bird join, counter-call, or go quiet based on vocal frequency and mood.

- Still pool offer type: Why: it creates a reflective surface where birds may drink, bathe, or watch.

- Rule-driven notebook generation for v1: Why: privacy and consistency requirements outweigh ML novelty at launch.

- Notebook candidate triggers: Why: entries should be noteworthy, such as first greeter changes, rare weather reaction, long quiet morning/evening, new bird settling, perch pattern shift, chorus event, or uncommon offer reaction.

- Notebook sparsity: Why: entries should appear roughly every few days, not every session, and should not describe user visit frequency.

- Notebook voice: Why: lowercase, present tense, bird-named, specific, no exclamation, no achievement language, and no numeric traits keep the naturalist voice.

### 7. Client Rendering Pipeline

- Server snapshot as render boundary: Why: the client draws, interpolates, schedules micro-motion, and submits events while never advancing mood, simulating drift, or deciding canonical perch changes.

- Canvas/WebGL scene with DOM chrome: Why: the scene needs a dedicated renderer while top bar/settings/notebook remain ordinary UI, and renderer dependencies must protect the 2MB bundle budget.

- Scene graph with background, perch zones, bird rigs, foreground ornament, and caption/focus overlay: Why: it creates a horizontal aviary scene without embedded controls.

- No embedded UI buttons, labels, badges, or hover tooltips in the aviary: Why: top bar chrome is separate and fades nearly transparent after cursor stillness.

- First paint quiet field: Why: there is no spinner, wake-up animation, or fade-from-static; network delay should still feel like a quiet field with faint ambient cues.

- Draw first bird from minimal descriptor: Why: the first bird should be visible under the first-paint budget.

- Defer notebook/settings/invite chunks: Why: first paint renderer stays minimal.

- First real frame with motion already in progress: Why: the aviary should not feel awakened by the user.

- One horizontal responsive scene with no panning, scrolling, or zooming: Why: all birds remain in frame and perch zones stay semantic, not user-manipulable slots.

- Reduced-motion render path: Why: it replaces micro-motion and flight/path transitions with still-pose cross-fades while keeping calls, captions, notebook, mood, and drift intact.

- Keyboard navigation contract: Why: keyboard-only users can move from top bar to scene, focus birds, toggle listen-in, use offers, and settle.

- Gentle visible focus indicators: Why: focus must remain visible across bright/dim states and read as "gentle, not game-like."

### 8. Audio Pipeline

- Species procedural call grammar: Why: motif primitives, variation rules, mood modifiers, personality modifiers, and bird seed create recognizable per-bird signatures.

- Stable bird call signature: Why: long-term recognition of a named bird's call depends on stable identity.

- Bounded WebAudio engine: Why: one context, reusable nodes, bounded voice pool, and no unbounded per-call allocation protect memory and CPU budgets.

- Captions generated from the same grammar event as sound: Why: captions match generated calls.

- Listen-in gain automation: Why: focused bird gain rises slowly, other birds lower to ambient but never silence, and disengage has no hard cuts.

- Chorus events as emergent scheduled overlap: Why: chorus should preserve per-bird signature, avoid canned sounds, protect recognizability and CPU, and respect the seven-bird cap.

- WebAudio unavailable fallback: Why: the visual aviary continues, call captions turn on for the session, and matter-of-fact copy explains silence if needed.

- Recorded audio fallback excluded: Why: the plan explicitly prohibits recorded fallback and favors procedural audio for budget and product consistency.

### 9. Accessibility Surfaces

- Screen-reader narration from visual state: Why: narration is "slow naturalist prose generated from the same state as the visual surface."

- Narration queue with priority and dedupe: Why: priority handles user-initiated events while avoiding high-frequency state spam.

- Narration without numeric or mechanical labels: Why: no numeric personality values and no "Pip mood: content" labels preserve product voice.

- Call captions: Why: captions support accessibility and audio fallback, come from procedural call grammar, fade with calls, respect reduced motion, and pass WCAG AA contrast.

- Matter-of-fact system voice and naturalist product voice: Why: settings, auth, sync errors, export, delete, and unsupported-browser surfaces have a different copy contract from product surfaces.

- Accessibility test matrix: Why: screen reader, keyboard-only, reduced-motion, caption contrast, and narration cadence tests keep accessibility part of product quality.

### 10. Sync and Conflict Model

- One server-owned canonical aviary with many snapshot readers and event writers: Why: multi-device clients see the same state because both read the same snapshot.

- No last-write-wins path for personality or mood: Why: personality and mood are server-authored through ordered event consumption and simulation worker writes only.

- Client pending event queue with idempotency keys: Why: local state remains limited and retries can happen without locally applying personality or mood.

- Suspended laptop frame-gap recovery: Why: after a long suspend, the client must pull a snapshot before continuing rendering.

- Visit revoked during active session behavior: Why: the next visitor snapshot returns visit unavailable, terminating access through the canonical read path.

### 11. Privacy, Security, and Data Governance

- Email encryption and keyed lookup hashes: Why: email can support account and invite workflows without becoming an identifier in services, logs, queues, or telemetry.

- Per-bird interaction events stored only for that account's simulation: Why: analytics warehouse and operational telemetry must not ingest per-bird state or account interaction logs.

- No ML/model training on this data: Why: privacy commitments exclude this data from training.

- Aggregate-only operational telemetry: Why: request counts, latencies, error rates, anonymous session histograms, render timings, audio errors, and tick timings are allowed without per-account or per-bird dimensions.

- Magic link expiry and invalidation: Why: short-lived one-time links support security.

- Invite expiry and immediate revocation: Why: visitor access is limited and controllable by the host.

- Hashed revocable session tokens: Why: session tokens are protected at rest and can be revoked.

- Account deletion hard delete after 30 days: Why: all account-tied records are removed after the soft-delete window.

- Expiring exports sent to verified email: Why: exports are generated on demand and delivered through the verified account channel.

- Abuse controls for magic-link/invite endpoints: Why: endpoints need abuse protection without becoming notification surfaces.

- Automated log redaction and telemetry schema tests: Why: builds should fail if telemetry includes bird IDs, personality fields, notebook prose, or raw account IDs as dimensions.

- Analytics reader permission separation: Why: analytics readers cannot read simulation tables.

### 12. Performance and Observability

- Initial JS under 2MB gzip: Why: bundle size protects first paint and browser-only performance.

- First bird visible under 500ms: Why: the first bird is the critical user-facing load moment.

- 60fps idle motion for 30 minutes and no memory growth: Why: the quiet continuous scene must remain stable over real watching sessions.

- Snapshot payload in kilobytes: Why: snapshots should stay compact render-ready state, not large state dumps.

- Simulation tick p99 alarm at 5 seconds: Why: the server-owned life needs timely canonical updates.

- Code splitting for settings, notebook, invite, export/delete, and admin-only tools: Why: non-first-paint surfaces should not delay the minimal renderer.

- Compact vector/SVG/procedural assets: Why: they support asset budgets and bundle control.

- Precomputed render descriptors server-side: Why: they can save client CPU without exposing hidden values.

- Stop rendering when hidden and refresh snapshot on return: Why: hidden rendering wastes resources, but visible return must resume from canonical state.

- Allowed observability metrics: Why: endpoint latency, magic-link rates, payload sizes, tick backlog, render timing, audio failures, unsupported browser counts, and anonymous session histograms support operations without privacy boundary drift.

- Disallowed observability metrics: Why: average drift by trait, per-bird aggregation, species popularity, account dashboards, and visit-frequency engagement analysis would violate the aggregate-only telemetry boundary.

### 13. Rollout Plan

- Foundations before visible simulation: Why: modules, schemas, migrations, auth, telemetry guardrails, privacy tests, and deterministic simulation fixtures are needed before user-visible simulation.

- Canonical Simulation before Rendering and Audio: Why: disconnected tick advancement, matching multi-device snapshots, measurable drift, and no negative drift must be true before presentation polish depends on them.

- Rendering and Audio before Interactions and Notebook: Why: first paint, 60fps idle, procedural calls, captions, and reduced motion establish the aviary surface before return greeting, offers, settle, and notebook.

- Account Lifecycle and Visits after core interactions: Why: export/delete/session/email and read-only visit controls must satisfy privacy requirements once the aviary interaction model exists.

- Calibration, Beta, Launch: Why: drift, mood, cooldowns, narration cadence, notebook sparsity, call recognizability, performance, accessibility, and privacy/data-flow need dogfood, audits, and beta tuning.

- Launch with two-bird aviaries only: Why: age-based adoption exists, but production should wait for confidence in audio recognizability and simulation load before additional birds mature into availability.

### 14. Test Strategy

- Drift monotonicity and drift rate tests: Why: absence must never decrease traits, and regular presence should meet one-week instrument and three-week visible targets.

- Presence conjunction tests: Why: visible-only, focus-only, and activity-only must not count.

- Mood transition tests: Why: time, weather, and personality interactions should remain bounded and expected.

- Event ordering tests: Why: tick must consume events in sequence.

- Offer cooldown tests: Why: repeated offers must be blocked from repeated drift contribution.

- Greeting selection tests: Why: greeting should produce one primary greeter, staggered secondary only, and no all-birds-unison.

- Notebook sparsity and dedupe tests: Why: entries must stay sparse, specific, and not generic event logs.

- Narration cadence and caption generation tests: Why: narration should not flood and captions should match call grammar.

- Visit events do not feed drift tests: Why: visitors cannot affect the host's aviary.

- Integration tests for auth, sessions, resume, deletion, export, invites, and audio fallback: Why: lifecycle, sync, privacy, and fallback paths need end-to-end coverage.

- Performance tests: Why: bundle budget, first-bird render, idle memory, idle frame rate, tick latency, and snapshot size are product budgets.

- Accessibility tests: Why: keyboard-only journeys, screen-reader smoke tests, reduced-motion regression, caption contrast, and top bar focus behavior keep accessibility from regressing.

- Product invariant string and route audits: Why: they prevent the easiest regressions into welcome copy, gamification terms, personality numbers, visit badges, visit-frequency surfaces, and UI controls inside the aviary scene.

### 15. Risks and Mitigations

- Drift Calibration mitigation: Why: deterministic simulation harness, one-week/three-week targets, daily/weekly caps, and long-horizon seeded tests reduce the risk that drift feels gameable or inert.

- Presence Inflation mitigation: Why: three-signal conjunction and cadence validation prevent background/open tabs from corrupting drift, and the plan says not to relax this for engagement metrics.

- Sync Correctness mitigation: Why: permissions, repository interfaces, and tests keep personality/mood writable only from the simulation worker.

- Audio Uncanniness mitigation: Why: audio prototypes, signature tests, human listening sessions, seven-bird cap, two-bird launch, and no recorded fallback reduce synthetic/repetitive/blurry calls.

- Accessibility Regression mitigation: Why: narration, reduced motion, and captions ship with core milestones and are tested as product surfaces.

- Privacy Boundary Drift mitigation: Why: telemetry allowlists, warehouse controls, privacy tests, and simulation-only per-bird state block account-level analytics and per-bird aggregation.

- Product Tone Violations mitigation: Why: copy taxonomy, product invariant tests, design review, and matter-of-fact vs naturalist ownership prevent toasts, badges, streaks, and cheerful system copy.

- Performance Budget Pressure mitigation: Why: bundle CI, code splitting, minimal bootstrap renderer, asset budgets, and throttled prototypes protect the 2MB and first-bird budgets.

- Notebook Quality mitigation: Why: rule-based sparse generation, dedupe windows, copy review, and tests block generic, frequent, or user-behavior entries.

### 16. Key Implementation Invariants

- The server is the only writer of personality and canonical mood: Why: this is the central product and architecture boundary.

- Personality vectors are hidden forever: Why: exposing trait numbers would violate the hidden-server-personality invariant.

- Presence requires visibility, focus, and recent pointer/key activity: Why: presence must represent real host presence without inferred gaze or background inflation.

- Absence never creates negative personality drift: Why: absence must not become punishment or decaying warmth.

- Visitors cannot affect the host's aviary: Why: visits are read-only and only for transparency/operational accounting.

- The bird greeting is the only welcome surface: Why: no textual welcome copy should replace the bird behavior surface.

- The aviary scene contains no UI chrome: Why: controls belong to separate top bar chrome, preserving the scene as a place.

- Calls are procedural; no recorded fallback: Why: procedural calls support identity, captions, performance, and the explicit fallback prohibition.

- Accessibility surfaces preserve charm: Why: accessibility is a product surface, not a compliance add-on.

- Telemetry is aggregate-only and excludes per-bird/per-account relationship data: Why: privacy commitments are architectural rules.

- V1 is web-only and quiet by design: Why: if a shortcut violates the invariants, the plan says it is "a product change" and should be rejected for v1.
