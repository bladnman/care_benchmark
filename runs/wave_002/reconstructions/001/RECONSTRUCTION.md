## System-level intent

- One canonical, account-backed aviary: The plan repeatedly frames V1 as "one canonical aviary" and "one account, one aviary, one user." This shows up in Product Boundary, the one-aviary data model, canonical state, no device-owned state, and the Definition of Done's "same canonical aviary across devices."
- Already alive, not woken by the app: The product should be "already alive when opened," with birds "mid-motion," a server simulation that "has continued whether or not a client was connected," and first-frame behavior with "No spinner-to-aviary transition" and "No 'wake up' entry."
- Server-authored simulation, client-rendered experience: The plan's central split is that "The server is the only authority" while clients "render snapshots, synthesize calls, interpolate motion, and submit events." This appears in the invariants, Client/Server Split, Tick Ownership, No Last-Write-Wins, and launch gates.
- Presence is meaningful but strictly qualified: The plan says "Presence is real interaction, but only when all presence conditions hold." The same intent appears in presence pings, presence windows, device overlap merging, and the presence-inflation risk.
- Slow, non-punitive change: The plan says "Personality drift is slow, monotonic toward expressive, and never punitive." Drift uses small saturating deltas, absence has no negative deltas, and "No single session produces visible personality movement."
- Non-gamified opacity: The plan repeatedly blocks "personality numbers, visit-frequency counters, streaks, scores, achievements, or explicit optimization feedback." Hidden vectors, notebook filters, adoption rules, telemetry exclusions, and copy audits all carry this principle.
- Stable identity with indirect expressivity: Birds have "stable bird IDs," stable call and visual seeds, hidden personality vectors, and safe derived "display parameters." Renaming changes only the name; the identity, species, drift history, and call signature remain stable.
- Voice boundaries are product architecture: The plan distinguishes "naturalist prose" for product surfaces from "matter-of-fact prose" for system, account, sync, accessibility, and error surfaces. Notebook, narration, errors, export, deletion, and visit revocation all enforce this boundary.
- Quiet, read-only visits rather than social networking: Visitors "can observe a host aviary read-only," "never affects host drift," and cannot create presence or interaction events. The plan also excludes public discovery, comments, feeds, friend graphs, and co-presence.
- Accessibility carries the same charm: The plan states "Accessibility must ship in V1, not as a later retrofit" and warns against a "flattened state list." Narration, reduced motion, captions, keyboard navigation, and QA are meant to preserve affect, not merely access.
- Privacy boundaries are part of the product promise: PII isolation, separate simulation and telemetry stores, aggregate-only metrics, no interaction data in analytics, export, and deletion all reinforce that per-bird/account data stays out of product-unsafe pipelines.
- Performance protects aliveness: The 2MB bundle budget, 500ms first-bird target, 60fps idle, no spinner, hidden-tab rendering stop, and synthetic timing checks are all tied to preserving the "already alive" premise.
- Constrained V1 scope resists product drift: The explicit exclusions, no catalog/shop framing, age-only bird additions, no notification badge framework, and risks around product voice and social expansion keep V1 "small," "specific," and "non-gamified."

## Per-feature whys

### 1. Product Boundary and V1 Scope

- Browser web app for modern Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN
- Email magic-link sign-in: NOT RECOVERABLE FROM PLAN
- One account, one aviary, one user: The plan uses this to preserve one "canonical aviary" and avoid device-owned or merged state; the same user sees "the same canonical aviary across devices."
- Two starter birds and seven-bird cap: Two birds support beta validation of "recognizability of two procedural calls" and drift calibration; the seven-bird cap is tested for "chorus recognizability, caption overlap, layout compression, and 60fps idle."
- Bird renaming: Renaming exists so a bird's product name can change while "Species, identity, drift history, and call signature seed remain stable."
- Server-side simulation tick: The tick exists because the simulation "has continued whether or not a client was connected" and is the only writer of personality, mood, perch changes, weather, notebook observations, and call schedule continuity.
- Hidden personality vectors, moods, stable identities, call seeds, perch state, and drift history: The plan hides raw values because the user should never see "personality numbers" or optimization feedback, while stable IDs and seeds keep birds recognizable and specific.
- Presence accounting from visibility, focus, and recent pointer/key activity: The rationale is to make presence "real interaction" while preventing "background tabs, overlapping devices, or stale activity pings" from overcounting attention.
- Return-greeting: It lets "one bird notices the user's return" without "any textual welcome." The server chooses greeting candidates so the client does not invent canonical behavior, and visitors do not trigger greetings.
- Listen-in: Listen-in gives the focused bird stronger per-bird influence on social warmth and vocal frequency, while the UI/audio remains calm: focused audio ramps up and other birds stay audible at an "ambient floor."
- Offer interactions and offer types: Offers give small deltas to curiosity and boldness, with the receiving bird emerging from "mood/personality simulation" rather than a direct selection; cooldowns prevent click-repeat saturation.
- Settle with short undo window: Settle "ends presence and may quiet mood" without driving long-term traits or penalizing absence. The five-second undo window makes the quieting reversible when the user clicks again.
- Field notebook browsing: The notebook is for "sparse, read-only naturalist observations," not raw logs, stats, achievements, or editable feed behavior.
- Single horizontal responsive aviary scene with three perch zones: The plan wants one calm stage where every bird remains visible, the scene stays readable, and there is no panning, scrolling, zooming, drag placement, badges, or hover labels inside the aviary.
- Day/night cycle, rare weather, and ambient micro-motion: These support the scene being "already alive," tied to local time, mood, call frequency, weather, and subtle continuous motion.
- Fading top bar: The top bar fades so controls do not dominate the aviary; keyboard focus restores visibility so discoverability is not lost.
- Quiet loading field: The quiet field avoids a spinner and supports the first-frame intent that the aviary does not feel empty or woken up after initial account creation.
- Accessibility settings, narration, call captions, keyboard navigation, and reduced-motion rendering: These ship in V1 because accessibility is "not as a later retrofit" and should preserve the same naturalist charm and complete interactions.
- WebAudio procedural call synthesis with captions on graceful silence: Procedural calls avoid "recorded call loops" and keep signatures recognizable across mood and drift; captions keep the product functional when WebAudio is unavailable.
- Account export and deletion: Export includes hidden vectors because "the user's account data is theirs"; deletion cascades through account-scoped records so privacy boundaries are enforced.
- Per-device sessions and session revocation: NOT RECOVERABLE FROM PLAN
- Opt-in read-only visit invitations, revocation, expiration, and visit log: Visits are quiet observation, "off by default" in spirit, revocable, expiring, and kept out of the main chrome so they do not become public discovery, feeds, co-presence, or drift input.
- Aggregate-only operational telemetry and synthetic performance monitoring: Telemetry is allowed only for operational counts, latencies, timing, and errors; the plan disallows per-bird/account interaction analytics to protect privacy and product direction.

### 2. System Architecture

- Backend module boundaries: The four modules separate auth/account, aviary state, simulation, and notification/email so V1 can be a modular monolith or separate services while keeping strict boundaries around authority and responsibilities.
- Separate simulation and telemetry stores: The rationale is explicit: analytics must not ingest bird IDs, bird state, per-account event streams, notebook text, or personality vectors.
- Client/server split: The server owns canonical identity, mood, drift, notebook entries, visits, export, deletion, and event validation; the client owns rendering, interpolation, WebAudio, captions, presence detection, and input responsiveness.
- Snapshot render pipeline: Snapshots contain enough state to render "the aviary as already in progress," then clients interpolate until the next snapshot and refresh after hidden/suspended states.

### 3. Data Model

- Relational canonical database with PostgreSQL as a V1 fit: The plan chooses this because the shape is "account-scoped, transactional, and benefits from strong ordering."
- Account PII model with synthetic UUID, encrypted email, and keyed email hash: The purpose is PII isolation; email is not used outside the account table, logs, telemetry, partitions, or background jobs.
- Sessions data model with device label, token hash, and revocation timestamps: NOT RECOVERABLE FROM PLAN
- Magic links expiring after 15 minutes and invalidated on consumption: NOT RECOVERABLE FROM PLAN
- Aviaries table with exactly one aviary per account: This carries the V1 boundary that there is "exactly one aviary per account" and one canonical scene.
- Birds table with stable IDs, call signature seed, and visual seed: Stable fields preserve bird identity, procedural recognizability, and silhouette/plumage variation across renames and drift.
- Server-only bird personality vectors: Raw trait values are kept out of product APIs because no API should return raw values and user-facing surfaces must not show personality/debug numbers.
- Bird drift ledger: The ledger is for "simulation correctness and troubleshooting," not product UI or aggregate analytics.
- Append-only interaction events with server receipt order and idempotency: This lets clients submit events while the server validates, stores, deduplicates, and consumes them without double-counting retries.
- Presence windows materialized from pings: Materialization makes "drift calibration and idempotency easier" while preserving the all-three-conditions rule for active presence.
- Notebook entries: Entries are "product prose, not raw logs" and should not mention visit frequency, streaks, personality numbers, or internal event names.
- Visit invite and visit session data: Visitors get read-only snapshot tokens; visitor-local mute or caption controls do not touch the host aviary.
- Export and deletion records: Export includes birds, vectors, moods, notebook, settings, and visits because the data is the user's; deletion cascades through account-scoped records while aggregate metrics without account dimension can remain.

### 4. API Surface

- JSON over HTTPS with pull snapshots for V1: The plan says WebSocket or SSE can be added later, but V1 can meet the spec with "small snapshots and low-frequency polling."
- Auth and account APIs: These keep account state, settings, sessions, export, deletion, and email change in matter-of-fact system surfaces rather than product-naturalist surfaces.
- Aviary snapshot API: The snapshot is the canonical render source, kept to a small payload, pulled on initial navigation, visibility return, sleep resume, keepalive, and after events that should reflect quickly.
- Interaction event batch API: The endpoint enforces server receipt order, idempotency, cooldowns, deleted/revoked-session checks, unknown-bird checks, and blocks visitor tokens.
- Notebook API: It is paginated, read-only, and has no edit/delete endpoint because the notebook is naturalist observation history, not a user-authored annotation feed.
- Bird rename API: The plan treats renaming as account-adjacent but preserves product identity by validating the name and avoiding changes to species, identity, drift history, or call seed.
- Age-gated adoption endpoint: Eligibility is based on "aviary age, not visit count or interaction score," and the presentation is "a bird has arrived" rather than shop/catalog optimization.
- Visit APIs: Invites are per-email, expiring, revocable, and read-only, with no social prompts, no main-bar badge, and no host drift or event creation from visitor snapshots.
- Error surfaces: API errors use machine codes and matter-of-fact display copy because auth, account, sync, visit revocation, unsupported browser, and accessibility errors should not receive naturalist copy.

### 5. Simulation Engine Design

- Tick ownership and cadence: The tick is the only writer for personality, mood, perch, weather, notebook, and call schedule state so canonical simulation does not move in the client.
- Bounded catch-up after missed ticks: The plan avoids "thousands of minute-by-minute loops" after an outage and preserves "mood/day-night continuity and drift slowness."
- Drift function: Low-pass, saturating, additive deltas make drift measurable after regular use but keep "No single session" from producing visible movement, and absence produces no negative deltas.
- Internal simulation tests and calibration harnesses: Synthetic profiles verify the "slow-timescale promise" without turning outputs into product-facing metrics.
- Mood transitions: Probabilistic transitions make mood respond to time of day, weather, other birds, events, and personality while avoiding a reset to neutral when a session starts.
- Return-greeting selection: Absence length, boldness, social warmth, mood, and greeting history avoid the same bird greeting first every time unless traits justify it.
- Call grammar runtime: Motif libraries, stable call signature seeds, and mood/personality modulation give continuity across devices without recorded loops and preserve the invariant that a bird's call remains identifiable.
- Chorus behavior: Overlap, non-phase-locked repetition, species/bird frequency bands, stereo placement, and ambient non-focused birds prevent chorus from becoming repetitive or like a mixer solo.
- Notebook generation: A rule-and-template system is chosen initially because entries need to be sparse, specific, lower-case present-tense naturalist prose and not an unconstrained model in the critical path.
- Notebook safety filters: Deterministic filters for banned concepts prevent streaks, achievements, scores, hunger, death, neglect, numeric trait copy, and other excluded product concepts from entering prose.

### 6. Sync and Conflict Model

- One canonical aviary record per account: Devices "do not own or merge aviary state"; host clients pull from one snapshot endpoint and write append-only events.
- Event idempotency: Deduplication on device session and client event ID prevents duplicate pings or retry storms from double-counting presence or offers.
- No last-write-wins for simulation fields: Personality, mood, drift, notebook, and canonical perch are never absolute client writes; the tick computes additive deltas from logs.
- Rename as the only last-write-wins exception: The exception is limited to bird name because it is an account action, and the plan says it must not generalize to simulation fields.
- Presence across devices: Overlapping host device windows are merged and capped so two visible devices do not double personality drift for the same wall-clock minute.
- Snapshot freshness and reconciliation: Clients render immediately, pull on visibility return and sleep resume, avoid teleporting when possible, and use matter-of-fact retry surfaces if loading fails.

### 7. Frontend Rendering Pipeline

- React or equivalent plus Canvas/WebGL or SVG/canvas hybrid: The renderer is chosen by prototype around time-to-first-bird, seven-bird 60fps idle, captions, and reduced-motion cross-fades, with the 2MB budget constraining engine weight.
- Scene composition: A single horizontal stage with sky/foliage, perch zones, bird layer, subtle foreground ornaments, and a thin top bar supports calm visibility without panning, scrolling, zooming, labels, badges, or inline tooltips.
- Loading and first frame: The implementation sequence targets "first bird visible within 500ms," draws a quiet field if delayed, draws birds into current poses, and defers non-critical panels so the aviary feels already alive.
- Bird motion: Composable pose states and local easing keep idle motion from looking paused while allowing calm, mood/personality-shaped movement from canonical descriptors.
- Reduced-motion rendering: Reduced motion is "not a static fallback"; cross-fades, stable focus, slowed shifts, and removed leaf drift preserve complete audio, captions, drift, mood, notebook, and interactions.
- Top bar and controls: The bar contains only account/settings, accessibility, notebook, offer, and settle; it fades away, returns on activity, and avoids badges so it does not turn notebook or visits into feed mechanics.
- Listen-in UI: Click/tap or keyboard focus plus Enter toggles a calm focus treatment, slow audio ramps, no "selected" label, and disengagement on exit conditions.
- Offer UI: Offers launch from the top bar and target the aviary rather than direct bird clicks so the receiving bird can emerge from server mood/personality simulation.
- Settle UI: Settle shifts lighting toward evening, quiets calls, and supports a five-second undo through an aviary click before settling becomes the calm state.
- Notebook UI: The notebook opens a read-only scroll surface with naturalist prose and no edit, delete, comment, or annotation affordances.

### 8. Audio Pipeline

- WebAudio synthesis engine: Oscillators/noise, motif envelopes, contours, timbre seeds, subtle room mix, and a limiter are used to create procedural calls without recorded loops or harsh chorus peaks.
- Audio scheduling: The client schedules calls ahead to avoid jitter but reconciles to server state, keeping call existence canonical while allowing bounded ambient continuity.
- Listen-in mix: Gain-node ramps make listen-in responsive locally, while listen-in start/end events go to the server because they affect per-bird drift.
- Captions: Captions are generated from the actual procedural call so audio and text describe the same motif, with descriptors shaped by mood and placement near the calling bird.
- WebAudio failure: When WebAudio is unavailable, the plan uses default-on captions and graceful silence rather than recorded fallback, while keeping visual simulation fully functional.

### 9. Accessibility Plan

- Screen-reader narration: Narration is generated from snapshot state in naturalist prose, with a 30- to 60-second idle cadence, priority updates, careful ARIA politeness, and no raw labels such as "perch 2" or "mood content."
- Keyboard navigation: The keyboard model covers top bar, aviary entry, bird-to-bird focus, listen-in, offer, settle, notebook, settings, and Escape behavior so the core product is reachable without pointer input.
- Reduced motion accessibility setting: The setting honors first-load preferences and keeps the full product complete through designed still-pose cross-fades and slowed transitions.
- Contrast and text: WCAG AA text across captions, settings, notebook, narration, errors, and all day/night/weather states keeps overlay copy legible even when scene art remains subtle.
- Accessibility QA: VoiceOver, Windows reader, keyboard-only scripts, reduced-motion regression, caption/audio-off, and contrast checks are launch requirements because accessibility ships in V1.

### 10. Privacy, Security, and Compliance Boundaries

- PII handling: Synthetic account IDs, encrypted email, keyed email hash, and no email in logs/telemetry/queues/partitions protect identity from leaking outside the account record.
- Interaction privacy: Per-bird events exist only to drive that account's simulation and must not flow to analytics, ML training, recommendations, dashboards, or third-party processors except required infrastructure under privacy controls.
- Account export: Export is on demand, uses a short-lived link, includes hidden vectors because the data belongs to the user, and uses matter-of-fact copy without marketing.
- Deletion: Soft deletion blocks normal access but allows recovery for 30 days; hard deletion cascades through account-scoped simulation, notebook, visit, and export data and remains auditable without retaining per-bird content.

### 11. Performance Budgets and Observability

- Initial JS bundle and code splitting: The less-than-2MB gzipped budget and deferred settings/visits/export/notebook history protect first paint and first-bird timing.
- Time-to-first-bird, runtime, and server budgets: First bird under 500ms, 60fps idle, bounded memory, hidden-tab render stop, small private snapshots, and tick p99 alarms all preserve aliveness and reliability.
- Allowed and disallowed metrics: Allowed metrics are operational; disallowed metrics such as drift averages, personality distributions, notebook analytics, and rankings prevent privacy erosion and gamified product direction.
- Synthetic checks: Scheduled automated browsers verify sign-in, snapshot, first bird, render loop, WebAudio, reduced motion, and code-split panels; synthetic accounts are marked to avoid export/deletion confusion.

### 12. Rollout Plan

- Build phases: The phase order builds engine/state foundations before scene, calibration before full interactions, accessibility before launch hardening, and privacy audits before release.
- Bird count ramp: The plan starts beta with two birds, then enables three, five, and seven birds behind a server flag only after validating recognizability, drift, timing, memory, captions, layout, and 60fps idle.
- Launch gates: Gates enforce the core invariants: magic-link/session basics, server-only personality writes, strict presence, drift calibration, no spinner, no recorded loops, accessibility pass, budgets met, privacy audit, and copy audit.

### 13. Testing Strategy

- Unit tests: Unit coverage focuses on drift, monotonicity, cooldowns, presence windows, mood probabilities, greeting selection, notebook filters, auth, visits, and host/visitor snapshot permissions because these enforce hidden simulation and boundary rules.
- Integration tests: Integration coverage verifies one account/aviary creation, event idempotency, single event consumption, overlapping-device drift caps, coherent multi-device state, stable renames, deletion, and visitor read-only boundaries.
- End-to-end tests: E2E coverage verifies the user-visible core loop: magic link, no-spinner aviary, greeting, keyboard listen-in, offer reaction/cooldown, settle/undo, notebook, reduced motion, WebAudio-blocked captions, export, and read-only visit.
- Long-run tests: Long-run coverage verifies memory, seven-bird idle performance, chorus recognizability, drift over one and three weeks, non-punitive absence, and missed-tick catch-up.

### 14. Risks and Mitigations

- Drift calibration mitigations: Calibration harnesses, regular-visit profiles, daily caps, hidden traits, and copy review prevent drift from becoming stat management or feeling meaningless.
- Presence inflation mitigations: Visibility, focus, recent activity, merged host-device windows, timeout endings, daily caps, and visitor exclusion prevent attention overcounting.
- Sync corruption mitigations: No public personality/mood write API, isolated simulation-worker database permissions, append-only event logs, transactional additive deltas, and tests protect canonical fields.
- Audio mitigations: Early motif grammar, stable signatures, mood/personality modulation, slow ramps, chorus tests, captions, and graceful silence prevent canned loops, uncanny repetition, and mixer-solo listen-in.
- Accessibility mitigations: Narration and reduced-motion are first-class renderer outputs, use naturalist prose rather than state labels, and are reviewed through QA and assistive-technology feedback.
- Product voice mitigations: Copy review, banned-language tests, matter-of-fact system components, and no notification badge framework prevent welcome banners, streak wording, badges, gamified counters, and charming error copy.
- Privacy mitigations: Separate stores, metric schema review, aggregate-only RUM, synthetic/local calibration data, and export audit logs prevent useful per-bird interaction data from leaking into analytics.
- Performance mitigations: CI bundle budgets, edge initial snapshots, code splitting, first-bird-before-noncritical-assets, synthetic timing checks, and avoiding heavy frameworks protect the "already alive" premise.
- Visit expansion mitigations: Account-settings management, per-email expiring invites, no public surfaces, no friend graph tables, read-only visitor tokens, and off-by-default unmarketed notifications prevent visits from becoming a social network.

### 15. Engineering Decisions to Make Explicit

- PostgreSQL plus Redis only for short-lived assistance: PostgreSQL remains the canonical store; Redis is not used for canonical simulation so state authority stays durable and ordered.
- Modular monolith first: The plan recommends this unless organizational constraints require microservices, while keeping module boundaries strict around auth, simulation, snapshots, and email.
- Pull snapshots for V1: Pull is preferred because it can meet V1 needs; SSE/WebSockets are deferred unless polling fails smoothness or battery needs.
- Rule/template notebook generator at launch: This keeps generation constrained, tested, and out of the critical tick path.
- Deterministic seeded procedural rendering: Seeded rendering reduces asset load while preserving bird variation and stable identity.
- Longer-side presence activity timeout: The timeout should count "quiet watching," not only active clicking or typing.
- Internal secure simulation debugging dashboards: Debugging surfaces may exist only as secure internal tooling and cannot be exposed through account or product UI.

### 16. Definition of Done for V1

- V1 done state: The done definition ties the plan together: same canonical aviary across devices, two birds already in motion, non-textual return notice, procedural recognizable calls, slow server-authored simulation, occasional naturalist notebook observations, charming accessibility, quiet read-only visits, met budgets, enforced privacy, and no excluded game, Tamagotchi, social network, notification, or native-app surface.
