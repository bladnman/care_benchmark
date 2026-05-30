## System-level intent

- The aviary should feel alive before, during, and after a visit. This shows up in the scope as "feels alive, continues without the viewer," in the success criterion that first render feels like an "already-running aviary," in the server tick that advances state "whether or not a client is open," and in the first-load instruction to show a "quiet field rather than a spinner."
- The relationship should not become a game, duty, or performance. The plan says it must "never turns the relationship into a game, duty, or social performance," explicitly excludes "streaks, counters, badges, levels, achievements," excludes "hunger, illness, decay, punishment for absence," makes bird additions depend on "aviary age rather than engagement," and says "absence cannot decrease" traits.
- The server owns truth, while clients render and emit events. The architecture states "clients render and emit events; the server owns truth," says the client "never computes durable bird state," and the sync model repeats that "only server tick mutates durable bird state."
- Continuity matters across devices and across weeks. This appears in "one canonical aviary per account," success criteria that birds "maintain continuity across devices and across weeks," persisted mood/personality state, stable call and animation seeds, and multi-device behavior where all devices read "the same canonical snapshots."
- Change should be slow, monotonic, and expressive, not punishing. The drift section says "monotonic toward expressive," "presence-time can increase traits; absence cannot decrease them," "no visible trait jumps within a single session," and "human-noticeable change after ~3 weeks."
- The product should notice rather than announce. The plan names "noticed, never announced" for greetings, keeps the top bar sparse and fading, avoids "new entry!" celebration in the notebook, and uses sparse narration rather than queue spam.
- Accessible modes should be the real product. The success criteria say "Accessible modes feel like the real product, not stripped-down fallbacks," and later sections call reduced motion "a designed mode" and accessibility reviews "first-class product surfaces" and "launch-blocking scope."
- Product voice should be quiet, naturalist, and sparse. Notebook entries use "templated naturalist grammar," narration is "slow, sparse, and prose-first," raw telemetry language is forbidden, and auth/sync failures are deliberately "matter-of-fact" rather than naturalist.
- Privacy should be technically enforceable, not policy-only. The success criteria require privacy promises to be "technically enforceable rather than policy-only"; telemetry is "aggregate health metrics"; analytics must not collect bird ids, notebook text, personality values, or per-account histories.
- Complexity should be prevented structurally where possible. The plan favors a "modular monolith" because risk is "simulation correctness and feel, not service sprawl," and the sync section says to "Prevent conflicts structurally rather than resolving them cosmetically."

## Per-feature whys

### 1. Scope and product boundaries

- Browser-based client for modern Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN
- Email magic-link authentication with revocable per-device sessions: The plan ties this to authenticated account and visitor access, session revocation, "15-minute magic-link expiry and one-time consumption," and export delivery through a "verified-email link."
- One canonical aviary per account, synced across devices: The why is continuity and conflict avoidance: one account has one truth, users see continuity "across devices and across weeks," and the sync model "eliminates client reconciliation of personality and mood."
- Server-side simulation tick: The why is that the aviary "continues without the viewer" and clients must not own durable state. The tick advances moods, drift, calls, weather, greetings, and notebook candidates "whether or not a client is open."
- Two starter birds and age-gated additions up to seven: The why is gradual expansion without engagement pressure. The plan says expansion is based on "aviary age rather than engagement," and later ramps bird count only after "audio recognizability review" and performance checks.
- Hidden personality vectors and visible mood expression: The why is to let birds develop durable identity while keeping internal calibration hidden. The plan separates personality from the bird row so "durable identity and tunable traits can evolve independently," while snapshots expose mood but not "numeric personality vector values."
- Presence accounting based on visibility + focus + recent input conjunction: The why is honest attention. The server constructs windows only where all three signals are true, mitigates "Background tabs or dual-device sessions inflate drift," and does not trust client-computed totals.
- Return greeting: The why is the "noticed, never announced" moment. It is tied to "true absence duration," computed from canonical state, and kept coherent across devices.
- Listen-in: The why is attentive focus without isolating the bird from the aviary. The transition should feel like "attentive listening rather than channel switching"; the focused bird gains presence while others are softened, "never mute."
- Offers: The why is modest expressive influence and observation. Offers can nudge curiosity and boldness "upward modestly," and an "offer reaction that reveals mood/personality" can become a notebook candidate.
- Offer seed, offer song fragment, and offer still pool as specific offer kinds: NOT RECOVERABLE FROM PLAN
- Settle: NOT RECOVERABLE FROM PLAN
- Field notebook: The why is sparse naturalist memory of notable state, not activity logging. Entries come from "notable state patterns, not every event," avoid "raw telemetry language," and use deterministic "naturalist grammar."
- Quiet visit invitations with read-only rendering for visitors: The why is sharing without social performance or simulation contamination. Visitors receive read-only snapshots, visitor activity is "excluded from drift inputs," and public social surfaces are out of scope.
- Accessibility surfaces: The why is that accessible modes must be product-equivalent. Narration, reduced motion, captions, and keyboard support are included so the product remains alive without relying only on motion or audio.
- Operational telemetry limited to aggregate health and performance metrics: The why is privacy enforcement. The plan allows timing, frame, memory, audio failure, tick, request, and error metrics but excludes per-account bird state and simulation records from analytics.

### 2. Architecture

- Thin-client, server-canonical architecture: The why is canonical state integrity. Clients render and emit events, while the server owns personality, mood, notebook, visit authorization, and all durable mutations.
- Modular monolith for v1: The why is to preserve code boundaries without distributed runtime complexity. The plan says "The engineering risk is in simulation correctness and feel, not service sprawl."
- Server/client responsibility split: The why is to keep durable behavior authoritative while letting the client remain expressive. Servers persist, simulate, authorize, and materialize snapshots; clients render, synthesize local audio, interpolate, and submit events.
- Render boundary at scene directives: The why is to let clients "feel alive" without exposing or moving simulation logic client-side. The boundary is neither raw internals nor finished animation clips.
- Snapshot exclusions for numeric traits, drift deltas, and hidden coefficients: The why is to preserve hidden state and calibration privacy. The plan says snapshots should not expose personality values, drift deltas, or calibration coefficients.

### 3. Data model

- Separate personality vector table: The why is independent evolution of identity and tuning. The plan says this keeps "durable identity and tunable traits" separate.
- Persisted mood state with `mood_entered_at`: The why is session continuity and stable transitions. Mood "survives session boundaries," and entered time lets transitions respect dwell time and avoid "rapid oscillation."
- Presence windows/session presence aggregates: The why is recalibration. The plan stores "raw-ish qualified windows" so calibration can be adjusted without reinterpreting ambiguous logs.
- Append-only interaction event log: The why is sync safety and replayable tick processing. Events are idempotent by event id, consumed transactionally, and marked with `processed_at` and `tick_id`.
- Visitor activity kept out of simulation inputs: The why is to protect host state from visits. Visitor view events belong in visit logs only and are "excluded from simulation inputs."
- Email encryption, email hash lookup helpers, and synthetic UUIDs: The why is privacy. Email is encrypted, used for lookup/rate limiting only through hashes where needed, and "never used as an internal identifier."
- Visit invite and visit log records: The why is revocability and transparency. Invites are per-email, expire or revoke, and visitor activity is logged for host visibility without affecting drift.

### 4. API surface

- HTTP+JSON with snapshot polling: The why is that "simulation cadence is slow enough" that polling and explicit refresh triggers are sufficient for v1.
- Account export and delete APIs: The why is account lifecycle control. The plan includes export by verified email link, soft delete for 30 days, cancel delete, and hard delete cascade.
- Aviary snapshot API: The why is canonical rendering. It returns scene snapshot plus version, generation time, and refresh hints so clients render from server materialized state.
- Batched interaction events API: The why is to append semantic client activity without allowing client-authored state mutation. The event log is append-only and later consumed by ticks.
- Specialized presence API: The why is operational noise control. The plan keeps this endpoint if frequent presence pings make general event batching "too noisy."
- Visit APIs: The why is quiet, revocable, read-only access. They create, list, revoke, log, and render visitor snapshots while preserving host-owned simulation.
- Optional narration endpoint: The why is timing and queue control only if the main snapshot cannot serve screen-reader needs. The plan says use a separate endpoint "only if" independent fetching is required.
- Semantic event contract: The why is server-side validation. Clients send "semantic events, never calculated deltas," and the server validates qualifying presence rather than trusting totals.
- Snapshot refresh and versioning: The why is freshness, not mutation arbitration. Refresh happens on navigation, visibility return, large gaps, meaningful interactions, and periodic keepalive; versions track freshness because events remain append-only.

### 5. Simulation engine design

- Once-per-minute tick cadence: The why is ambient continuity at low cost. The cadence is "slow enough to feel ambient and cheap enough to run for all accounts regardless of active clients."
- Atomic tick stages: The why is correctness. The tick loads due aviaries, consumes events, updates presence, drift, mood, weather, position, calls, notebook, unlocks, then persists state and marks events processed "atomically."
- Monotonic drift function: The why is expressive growth without punishment. Presence, listen-in, and offers can increase traits, absence cannot decrease them, and headroom creates asymptotic saturation.
- Rolling time windows and slow behavior targets: The why is to prevent a long session from creating visible overnight change. The plan targets instrumentation-level movement after about one week and human-noticeable change after about three weeks.
- Presence accumulated per account session and applied at tick time: The why is future flexibility. The plan says this allows "bird-specific weighting later without redefining presence itself."
- Mood transition state machine: The why is fast-timescale expression that still persists. Inputs include time, weather, interactions, spillover, and personality modifiers; dwell time avoids rapid oscillation.
- Return greeting runtime: The why is coherence across devices and true absence duration. The server provides greeter, responders, style family, offsets, and seeds at session-start refresh.
- Call grammar runtime: The why is recognizable bird identity with variation. Stable `call_signature_seed` and motif libraries keep each bird recognizable while mood and trait modifiers alter expression.
- Bird-to-bird interaction: The why is to create "a small social system, not a crowded rules engine." Coupling changes chorus, wary propagation, and call response likelihood sparsely.
- Notebook generation: The why is notable observation rather than logging. Ranking looks for greeting changes, quiet stretches, weather/posture combinations, perch shifts, and revealing offer reactions, then applies sparse cadence.

### 6. Sync model

- Conflict prevention rules: The why is to avoid cosmetic reconciliation. The plan forbids client-submitted absolute personality values, last-write-wins bird state, and client-authored durable state.
- Multi-device behavior with simultaneous presence cap: The why is fairness and honest drift. Multiple sessions are legitimate, but aggregate presence is capped so a user does not "earn 2x drift" from laptop and phone.
- Offline and degraded handling: The why is graceful continuity without local durable simulation. The client freezes ambient render, queues fresh events, discards stale ones, and refreshes canonical state on reconnect.
- Session timeout and magic-link edge case voice: The why is copy discipline. Auth and sync failures should be "matter-of-fact," with account/system voice centralized separately from product-surface voice.

### 7. Frontend rendering pipeline

- Browser-native rendering stack: The why is motion richness within bundle and compatibility limits. The plan avoids heavyweight 3D/WebGL-first choices because scene complexity does not justify the "bundle and compatibility cost."
- Scene composition layers: The why is predictable depth and visibility. Layers order sky, perch zones, foreground, chrome, and overlays while ensuring birds "never crop" offscreen.
- First-load quiet field and already-in-pose bird: The why is the affective threshold that the aviary is already running. The plan prefers boot snapshot in HTML/bootstrap, a quiet field over a spinner, and no boot-up animation.
- Normal motion system: The why is aliveness through subtle seeded variance. Idle micro-motion, slow arcs or hops, parallax, weather overlays, and drift make the scene active without abrupt state changes.
- Reduced-motion mode: The why is product-equivalent accessibility. It uses pose cross-fades, removes drifting ornaments, preserves palette transitions, and keeps the same canonical state and audio/caption behavior.
- Listen-in transitions: The why is to feel like attention rather than switching channels. Audio mix and visual focus ease in and out over several hundred milliseconds.
- Top bar behavior: The why is calm interface presence. A sparse icon set fades nearly transparent after inactivity, restores on input, and remains keyboard reachable with visible focus.
- Field notebook UI: The why is quiet reading rather than reward mechanics. It is read-only, reverse chronological, paginated or infinite, with no badges, unread counts, or celebration.

### 8. Audio pipeline

- WebAudio-first synthesis: The why is low bundle weight and per-call variation. The client synthesizes from motif instructions rather than shipping recorded audio assets.
- Recognizable per-bird identity: The why is continuity over time. Stable motif family and signature seed combine with limited variation so mood changes expression "without erasing identity."
- Chorus mixing: The why is intelligibility at higher bird counts. The mixer preserves recognizability, avoids clipping and crowding, and thins overlap windows near the bird-count ceiling.
- Listen-in mix: The why is focus without muting the social scene. The focused bird gets presence and clarity while others drop to ambient and gradually return.
- WebAudio fallback behavior: The why is graceful degradation without low-quality imitation. If unavailable or blocked, the app switches to silent mode, turns captions on by default, and keeps rendering and interaction intact.
- Audio testing: The why is determinism, compatibility, leak prevention, recognizability, and uncanniness control. The plan calls out motif determinism, AudioContext startup/resume, long-session leak tests, and perceptual QA.

### 9. Accessibility surfaces

- Screen-reader narration: The why is sparse prose access to canonical aliveness. It is fed by scene state and user-triggered events, deduplicated to avoid queue spam, and written as naturalist prose rather than raw state dumps.
- Keyboard model: The why is full interaction without pointer. Keyboard can reach the top bar, aviary entry, bird focus, listen-in, escape, offer, and settle.
- Call captions: The why is accurate audio-off and hearing-difference support. Caption text comes from the "actual procedural output" and is anchored to avoid obscuring the scene while respecting contrast.
- Reduced motion and audio-off users: The why is designed equivalence. Reduced motion is not disabled animation, and narration/notebook/captions continue to communicate aliveness without audio.
- Contrast and focus: The why is reliable access in calm palettes. The plan requires WCAG AA for copy surfaces and visible focus rings, with dusk/night states tested because calm palettes can hide cues.
- Accessibility QA gates: The why is launch quality. Screen-reader, keyboard-only, reduced-motion, and caption checks are specified as first-class gates rather than afterthoughts.

### 10. Performance, observability, security, and rollout

- Performance budgets: The why is preserving the affective first-bird threshold and long-session stability. Budgets include first bird under 500ms, 60fps idle motion, no 30-minute memory growth, bundle size, and tick p99.
- Engineering implications for budgets: The why is to keep the first experience small and smooth. The plan calls for code splitting non-core flows, small snapshots, server-side scene directives, audio reuse, and capped ornament/call complexity.
- Aggregate observability: The why is operational health without privacy erosion. Metrics cover load, render, frame, memory, audio failures, tick latency, requests, and errors, while excluding per-account simulation details.
- Internal admin/debug tooling: The why is incidents remain diagnosable without analytics mirroring. Debug access is for a specific account under authenticated operational workflows.
- Privacy boundary implementation: The why is enforceable privacy. Simulation database access is separate from analytics, telemetry schemas ban sensitive fields, ids are synthetic UUIDs, and emails are encrypted.
- Visit privacy: The why is host control and no simulation contamination. Invites are per-email and revocable, visitor activity is logged for transparency but excluded from drift, expired links fail closed, and notifications are opt-in.
- Account lifecycle: The why is controlled account safety and deletion. Magic links expire and are one-time, delete is soft for 30 days before hard cascade, and export goes through verified email.
- Rollout phases: The why is staged validation of feel, calibration, accessibility, visits, bird count, and privacy. Internal prototype validates canonical simulation and first-bird timing; alpha adds notebook/offers/settle/accessibility; beta adds visits and cautious bird-count ramp; GA ships alarms and audits.
- Day-one instrumentation: The why is launch health in aggregate. It tracks auth, snapshot latency, first-bird timing, tick latency/backlog, presence qualification rate, audio failures, and accessibility mode adoption counts.
- Risk mitigations: The why is preserving the core contract under calibration, presence, sync, audio, accessibility, performance, and privacy risks. Mitigations include calibration harnesses, server-side conjunction, idempotent events, perceptual QA, launch-blocking accessibility reviews, bundle CI, and telemetry schema linting.
- Recommended implementation order: The why is to build canonical truth before expressive layers and expansion. The sequence starts with schema/auth/snapshot, then simulation, presence/drift, rendering, audio, interactions, accessibility, visits, privacy hardening, and QA before bird-count expansion.
- Open implementation decisions: The why is to allow calibration and prototyping without changing the product. Thresholds, mood coefficients, unlock schedule, render split, and template library are "acceptable ambiguities" as long as non-negotiable constraints hold.
