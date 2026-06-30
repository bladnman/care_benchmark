## System-level intent

1. Keep ownership boundaries small, hard, and mapped to "load-bearing rules." The Architecture section says the backend is "kept deliberately small" and is "not a microservice-per-noun system"; the services have "hard ownership boundaries that map directly to the PRD's load-bearing rules." This intent appears again where visitor writes are impossible "by construction," where telemetry is "architecturally isolated," and where future gamification would require "adding a new aggregate pipeline" rather than becoming a quiet UI addition.

2. Treat the server as the canonical state owner and the client as a "thin renderer and an event emitter." The Client/server split says the client "never computes personality, mood-transition logic, or drift," and the Sync model says "only the tick writes" personality and mood. This is the design philosophy behind "no last-write-wins," "no client-side merge logic," and "the client never owns state."

3. Enforce product refusals structurally, below the UI layer. The Scope section says non-goals are "architectural constraints, not just UI omissions." This shows up in no client-writable personality state, absent streak or visit-frequency data fields, separate visitor routes with "no write path," and telemetry that has "no per-account dimension." The plan repeatedly prefers "structural defense" and "by construction" over convention.

4. Use the snapshot as the render and sync boundary. The Render pipeline boundary says "everything below the line" is server-owned and "everything above the line" is client-owned rendering. A "fresh page load reconstructs the full visual state from one snapshot fetch," which supports multi-device sync, avoids client drift, and keeps client caches from becoming source of truth.

5. Preserve a quiet, naturalist product voice instead of exposing meters or state lists. The plan uses "naturalist prose," "sparse," "read-only," "matter-of-fact," "quiet field," "no spinner," and "no UI chrome inside the aviary." Screen-reader narration must not become "Bird 2: mood content"; notebook prose should avoid literal repeated templates. The same voice is shared across narration and notebook.

6. Make accessibility a v1 product surface, not a late repair. Scope includes screen-reader narration, reduced motion, call captioning, WCAG AA contrast, and keyboard navigation. Rollout says "accessibility and core engine are not phased apart," while Risks says future additions carry "two render paths and a narration/caption obligation, not one."

7. Make change feel alive over weeks, without becoming a reward loop. The Drift calibration section encodes "measurable in instruments after ~1 week, visible to users after ~3 weeks" as a testable contract. Risks frames the failure modes as "Tamagotchi-by-accident" versus "screensaver." Birds-per-aviary ramp is based on "age, not interaction count or tier."

8. Prefer bounded, testable systems over unbounded runtime complexity. The plan names bundle size, first-bird timing, 60fps idle motion, no memory growth, tick latency, bounded audio chains, pooled buffers, virtualized notebook lists, and release-blocking CI gates. The technology choices are described as "defensible defaults" and "sufficient" for v1 rather than expansive infrastructure.

9. Keep privacy and telemetry aggregate-only. The Privacy and telemetry boundary says per-bird interaction events and personality/mood state "exist only to drive that account's own simulation." The analytics pipeline is never allowed to read the simulation database and emits only operational metrics, which also makes no-streak and no-visit-frequency rules durable.

10. Keep social ambient, revocable, and isolated. The Visit service has no write path, visitor sessions are read-only, invitations expire or can be revoked, and visitor presence "never drives host drift." This keeps social beyond "the single visit affordance" out of v1 while still allowing an ambient visit experience.

## Per-feature whys

### 1. Scope

- Single canonical aviary per account: the plan's rationale is multi-device simplicity and no client-side merging. It later says two clients reading the same canonical record need "no client-to-client protocol at all."

- Magic-link sign-in: NOT RECOVERABLE FROM PLAN

- Two starter birds at signup: NOT RECOVERABLE FROM PLAN

- Server-controlled bird growth to a cap of seven: the server controls growth so bird availability cannot become activity-count or attention-count gamification. Later, bird ramp is "age-gated, never attention/visit-count-gated," and the seven-bird cap also bounds rendering and audio resources.

- A roughly six-species pool: NOT RECOVERABLE FROM PLAN

- Personality vector with five traits, server-owned, monotonic-toward-expressive, never exposed numerically: the rationale is to make bird behavior and appearance drift over time while preventing users or clients from seeing or writing raw trait values. The plan enforces this at data and API schema layers.

- Mood system with a small enumerated state set: the plan uses mood as the compact server-owned state that drives transitions, calls, perch zones, narration, and rendering. Specific reasons for the exact state set are NOT RECOVERABLE FROM PLAN.

- Daily-ish mood reset: NOT RECOVERABLE FROM PLAN

- Procedural call synthesis, bird-to-bird call interaction, and chorus: the rationale is to avoid repeated recorded loops and phase-cancellation artifacts, while keeping calls varied, recognizable, and compact enough for the bundle budget.

- Return-greeting: NOT RECOVERABLE FROM PLAN

- Listen-in: the rationale is an audio-mix interaction that focuses one bird while preserving ambience. The plan says other birds reduce to a "reduced-but-nonzero ambient floor" and "never go silent."

- Offers: the plan gives rationale for server validation and cooldowns: offers have small drift influence and must not create drift saturation. Specific rationale for the seed, song fragment, and still pool offer types is NOT RECOVERABLE FROM PLAN.

- Settle with 5s undo: the rationale for the undo window is server authority. The server, "not client timer," controls the 5s window so client clock skew cannot extend it.

- Field notebook, sparse, naturalist, read-only: the rationale is to record only "notebook-worthy" state changes in product voice, with most ticks producing no entry "by design." It is generated from the same event/state stream as the tick and is read-only to clients.

- Presence accounting requiring visibility, focus, and recent pointer/key activity: the rationale is that drift should come from real presence rather than passive page load or replayed client claims. Presence is reconciled from server-timestamped ping density.

- Server-side simulation tick independent of client connection: the rationale is continuity. The tick "runs whether or not any client is connected" so birds continue changing across absence, reconnects, and multi-device use.

- Multi-device sync via canonical server state: the rationale is that sync is "an emergent property of the architecture," not a separate subsystem. Clients send events, the tick orders them, and snapshots replace local cache wholesale.

- Visit-invitation social feature: the rationale is to allow one limited, ambient social affordance while avoiding broader social-network surfaces. It is read-only, revocable, expiring, and isolated from drift.

- Screen-reader narration: the rationale is to give screen-reader users the same "naturalist prose" product voice, not automated state labels.

- Reduced-motion mode: the rationale is that reduced motion must be "its own designed render path," not "animations off," and must ship in v1.

- Call captioning: the rationale is to provide an equivalent for actual synthesized audio, especially when WebAudio is unavailable or denied.

- WCAG AA contrast and full keyboard navigation: the rationale is accessible operation of DOM chrome, captions, menus, focus, and account/error surfaces.

- Performance budgets: the rationale is first-frame immediacy, sustained idle life, bounded memory, and tick reliability on stated device/network targets.

- Account export as JSON snapshot, emailed: NOT RECOVERABLE FROM PLAN

- Account deletion with 30-day soft delete then hard delete: NOT RECOVERABLE FROM PLAN

- Aggregate-only operational telemetry: the rationale is privacy and anti-gamification. Telemetry should observe operations without per-account interaction patterns or per-bird state.

### 2. Architecture

- Four backend services plus a CDN-served client: the rationale is a "small number of services" with hard ownership boundaries, not "a microservice-per-noun system."

- Auth service owning no bird/personality data: the rationale is separation of account/session lifecycle from simulation state.

- Aviary service as the only writer of personality and mood: the rationale is canonical simulation ownership and prevention of client or visitor writes.

- Notebook service or module: the rationale is that notebook entries come from "the same event/state stream the tick consumes," avoiding duplicated product logic.

- Visit service with no write path to the aviary: the rationale is visitor isolation "by construction"; visitor tokens are never accepted by the event-write API.

- Telemetry/observability pipeline isolated from the aviary database: the rationale is that operational events are emitted separately and analytics never read simulation data.

- Client as thin renderer and event emitter: the rationale is implementing "the client never owns state" and giving multi-device sync "for free."

- Client-side audio synthesis from snapshot data: the rationale is that synthesis is a rendering concern, while simulation remains server-owned.

- Render pipeline boundary at the state snapshot: the rationale is to prevent client-side cache drift and allow a fresh page load to reconstruct visual state from one fetch.

- Server-side narration prose: the rationale is to avoid duplicating "what changed and is it notebook-worthy" logic in two places with two voices that could drift apart.

- Notebook and narration generator as the same internal module: the rationale is shared "naturalist prose from the same state," differing only in cadence and triggers.

- Typed backend language and Postgres: the rationale is that strong consistency matters for the "no-last-write-wins" rule.

- Lightweight append-only log or narrow queue instead of a streaming platform: the rationale is low v1 volume, about one event per interaction.

- Simple tick scheduler over accounts-due index: the rationale is that v1 scale and ~1/min cadence do not require per-account always-on processes.

- Canvas or WebGL scene renderer: the rationale is predictable 60fps handling for up to seven independently animating birds and ambient ornaments.

- CDN/edge inlined initial snapshot: the rationale is meeting the "<500ms time-to-first-bird" target by avoiding a second round trip.

### 3. Data model

- Synthetic UUIDs for all IDs: the rationale is that "email is never a key anywhere outside the account record itself."

- Account settings for visit notifications, reduced motion, captions, and audio: the plan names the settings, but the rationale for storing them in this exact shape is NOT RECOVERABLE FROM PLAN.

- Session device_label: the rationale is a "user-visible" device label for the revocation list.

- MagicLink token_hash: the rationale is "never store the raw token." The rationale for the exact 15-minute expiry is NOT RECOVERABLE FROM PLAN.

- Bird species_id references a static species pool: the rationale is that species is not "a row the simulation mutates."

- Bird personality stored as a column rather than derived at read time: the rationale is the explicit rule "never derived from session history at runtime."

- Bird mood value "settled": the rationale is supporting night and settle-gesture state.

- Bird perch_zone as server-computed, not user-set: the rationale is that it remains a mood/personality signal rather than a customization control.

- Personality and mood absent from client-writable API payloads: the rationale is schema-layer enforcement, "not just by convention."

- InteractionEvent append-only table: the rationale is that it is the "only way client behavior reaches personality/mood."

- InteractionEvent sequence_no assigned at write time: the rationale is strict event order regardless of device or clock, plus tick ordering and idempotency.

- NotebookEntry prose stored as final naturalist text: the rationale is server-side generation and read-only client display.

- NotebookEntry referenced_bird_ids never exposed as structured data: the rationale given is future internal querying without structured client exposure.

- Invitation with encrypted visitor email, token hash, expiry, revoke fields: the rationale is revocable, expiring invitation lifecycle. The rationale for exactly 30 days is NOT RECOVERABLE FROM PLAN.

- VisitSession duration_accum_seconds and VisitLogEntry approximate duration: the rationale is the host-visible visit log's "approximate duration."

- Absence of streak counters, visit-frequency aggregates, days-active fields, and read-time personality computation: the rationale is enforcing non-goals, privacy, anti-gamification, and stored tick-owned personality.

### 4. API surface

- GET /v1/aviary/snapshot: the rationale is returning current renderable state while withholding raw personality scalars. This enforces "personality is never exposed numerically" because values "simply aren't transmitted."

- Snapshot visual and behavioral projections such as mood, perch_zone, call timing, and plumage tier: the rationale is to send only what the client needs to render, with personality effects "pre-baked" into render parameters.

- GET /v1/aviary/snapshot?since=: the rationale is a cheap diff/etag path for low-frequency keepalive and visibility-change refetch.

- GET /v1/notebook?cursor=: the rationale is paginated, reverse-chronological notebook entries with infinite scroll-back.

- GET /v1/narration/stream: the rationale is screen-reader narration at 30-60s cadence, with prioritized events pushed immediately.

- POST /v1/events/presence-ping: the rationale is logging the visibility/focus/recent-input conjunction while letting the server timestamp and reconcile presence rather than trusting a client presence boolean.

- POST /v1/events/listen-in-start and listen-in-end: the rationale for the end endpoint is that the server computes duration from the matching start, so the client does not self-report duration.

- POST /v1/events/offer: the rationale is server-side cooldown enforcement for "drift-saturation prevention."

- POST /v1/events/settle and settle-undo: the rationale is server authority over the 5s undo window so client clock skew cannot extend it.

- Auth magic-link request and verify endpoints: NOT RECOVERABLE FROM PLAN

- GET/POST account sessions list and revoke: the rationale is device session revocation using the user-visible session list.

- Email-change request and verify endpoints: NOT RECOVERABLE FROM PLAN

- Account export endpoint: NOT RECOVERABLE FROM PLAN

- Account delete and undo endpoints: NOT RECOVERABLE FROM PLAN

- Bird rename endpoint: NOT RECOVERABLE FROM PLAN

- Account settings patch endpoint: the rationale is allowing user control of visit notifications, reduced motion, captions, and audio preferences named in scope.

- Visit invite endpoint: the rationale is host-authenticated, email-based invitation for the single allowed social affordance.

- Visit revoke endpoint: the rationale is revocability of invitations.

- Visit log endpoint: the rationale is a quiet host-facing visit record plus outstanding invites.

- Token-authenticated visitor session and separate visitor snapshot endpoint: the rationale is structural read-only isolation; it is "a different route entirely from the event-write API."

- 410 Gone for revoked or expired visits: the rationale is rendering the "matter-of-fact" visit-no-longer-available surface.

- Visitor snapshot endpoint not feeding InteractionEvent: the rationale is making "visitor presence never drives host drift" true by construction.

### 5. Simulation engine design

- Scheduler sweeping every account every ~60s regardless of clients: the rationale is honoring "the tick runs whether or not any client is connected."

- Tick reads only events after the last processed watermark: the rationale is ordered, incremental event consumption.

- Tick computes elapsed real-world time since last tick: the rationale is correct mood/time-of-day progression after zero ticks for days or infrastructure downtime.

- Mood updates from time-of-day, weather, bird-to-bird propagation, and personality: the rationale is that mood should be modulated by ambient context and bird traits.

- Personality deltas aggregated from presence, listen-in, offers, and settle: the rationale is drift from behavior, with presence dominant, listen-in strong, offers small, and settle non-directional.

- Low-pass filter with a one-week/three-week calibration target: the rationale is slow, expressive drift that becomes instrument-measurable before it becomes user-visible.

- Clamp preventing negative deltas: the rationale is enforcing "monotonic-toward-expressive" even if weighting math later bugs.

- Perch_zone as a function of mood and boldness: the rationale is translating internal state into spatial behavior.

- Smoothed perch changes: the rationale is that a bird should not "teleport perch-to-perch every tick."

- Call-timing parameters from vocal_frequency and mood: the rationale is feeding the client call scheduler with behavioral propensities without making call moments server state.

- Notebook-worthiness evaluation in the tick: the rationale is writing entries from the tick's computed state change.

- Persisting bird rows and watermark in one transaction: the rationale is crash safety and idempotent re-run from the watermark.

- Presence ping anti-gaming: the rationale is that the server must be the timekeeper; a client cannot claim "I had 3 hours of presence" in one event.

- Drift calibration as a CI-level test: the rationale is making "calibrated correctly" an operational definition and a release gate.

- Notebook cooldown and noteworthiness score: the rationale is enforcing roughly one entry every few days for a regular aviary, with most ticks producing no entry.

- Structured-but-varied notebook prose generation: the rationale is avoiding literal repeated template strings while keeping vocabulary constrained to bird/event/time.

- Client-side call grammar runtime: the rationale is WebAudio synthesis from compact motifs, with no two calls byte-identical and chorus emerging when independent timers overlap.

### 6. Sync model

- Single writer for personality and mood: the rationale is eliminating direct write conflicts on canonical bird state.

- Clients writing only append-only InteractionEvent rows: the rationale is keeping concurrent device activity as ordered contributions rather than competing final values.

- No last-write-wins: the rationale is that clients never submit absolute personality values, so there is no personality write-write conflict.

- Server-assigned sequence_no for concurrent sessions: the rationale is safe ordering even when laptop and phone sessions overlap in wall-clock time.

- Snapshot pull on visibilitychange, large animation-frame gap, and keepalive: the rationale is avoiding stale state after foregrounding, resume from suspend, or normal visible use.

- Snapshot payloads kept small: the rationale is performance; they carry current renderable state, not per-bird raw history.

- Local snapshot cache replaced wholesale: the rationale is instant re-render on reconnect without ever merging or treating cache as source of truth.

### 7. Frontend rendering pipeline

- Canvas/WebGL single horizontal scene with three perch zones: the rationale is responsive layout that preserves "never crop a bird, never let one drift offscreen."

- Perch anchor points defined as viewport proportions: the rationale is responsive scaling rather than fixed-pixel drift.

- DOM top-bar UI layer: the rationale is accessibility and focus handling.

- Idle micro-motion state machine per bird: the rationale is mood-appropriate life: drowsy birds move slowly, alert birds move faster with more head movement.

- Smooth flight/hop interpolation for perch changes: the rationale is implementing "interpolates between snapshots for smooth motion" and avoiding teleporting.

- No spinner, no fade-from-static, no skeleton UI: the rationale is first paint as a continuation, not a loading artifact.

- First snapshot inlined in HTML: the rationale is avoiding a client fetch before first paint.

- Randomized idle phase on first frame: the rationale is making first paint look already alive rather than frame 0 of a cycle.

- Quiet soft-sky-color field for slow snapshot fetch: the rationale is providing an interim surface that is explicitly not a spinner, progress bar, or skeleton.

- Empty-aviary adoption transition with first and second starter birds flying in: the plan describes the transition, but the specific rationale for two entrances is NOT RECOVERABLE FROM PLAN.

- Empty state never shown again after birds exist: the rationale is account-state consistency enforced by checking bird_count before rendering the empty branch.

- Client-side day/night palette from local time: the rationale is that the client owns local-time rendering, while the server only needs timezone for mood transitions.

- Continuous day/night interpolation: the rationale is avoiding stepped hour-boundary changes.

- Server-flagged weather rendered client-side: the rationale is that all devices show the same weather moment and mood effects are tick-computed, while particles/shaders remain lightweight rendering.

- Reduced-motion render path: the rationale is that reduced motion is a designed surface, not disabling default animations.

- Reduced-motion pose cross-fades and no ambient ornaments: the rationale is lower motion while preserving state expression.

- Listen-in visual cue: the rationale is making the audio interaction coherent visually without adding "UI chrome inside the aviary."

### 8. Audio pipeline

- Per-species motif library in the client bundle: the rationale is compact procedural audio rather than recorded samples.

- Waveform/envelope/filter generators with runtime jitter: the rationale is recognizable species calls that never repeat identically.

- Per-bird seeded parameter offsets: the rationale is making each bird's calls recognizable as that specific bird.

- Live WebAudio chorus mixing: the rationale is avoiding phase-cancellation artifacts from stacked recorded loops.

- One oscillator/gain chain per bird into a shared mix bus: the rationale is generating each call fresh into the graph rather than summing fixed waveforms.

- Listen-in gain automation ramp: the rationale is smooth engage/disengage over about 1-2 seconds.

- Nonzero ambient floor during listen-in: the rationale is the "never go silent" rule.

- WebAudio fallback with no audio graph: the rationale is identical visual behavior when AudioContext is unavailable or denied.

- Captions default on in WebAudio fallback: the rationale is that silence with no captions would be "strictly worse" than the user's stored preference anticipated.

- No recorded-audio fallback: the rationale is enforcement by not building one.

- Audio buffer and oscillator pooling: the rationale is meeting the no-memory-growth requirement.

- Bounded concurrent voice chains sized to seven birds plus headroom: the rationale is controlling resource use for the bird cap and chorus overlap.

### 9. Accessibility surfaces

- Server-generated screen-reader narration in an aria-live polite region: the rationale is live-region-friendly naturalist prose on a 30-60s cadence, immediate for important events.

- Narration and notebook sharing the same generation system: the rationale is a shared product voice rather than a "parallel accessibility-only feature."

- No visual DOM/canvas ARIA-label automation: the rationale is avoiding labels like "Bird 2: mood content" and keeping narration authored prose.

- Captions generated from actual call-grammar parameters: the rationale is descriptions matching what was synthesized, not fixed strings keyed to call type.

- Captions as small fading text near the calling bird: the rationale is localizing the description to the source call while allowing contrast and font handling.

- Keyboard navigation through top bar, canvas birds, listen-in, offers, and settle: the rationale is full keyboard reachability.

- Hidden but tab-reachable per-bird focus model synced to canvas position: the rationale is keyboard focus and a visual focus ring over canvas-rendered birds.

- Offer affordance as DOM popover/menu: the rationale is straightforward focus trapping and AA contrast.

- Automated contrast checking over rendered chrome: the rationale is verifying WCAG AA for DOM user copy in CI.

### 10. Performance budgets and observability

- Initial JS bundle at or below 2MB gzipped: the rationale is supported by procedural audio, route-level code splitting, and compact visuals.

- Time to first bird under 500ms: the rationale is edge-delivered inlined snapshot, no non-critical blocking assets, and the bundle budget.

- Sustained 60fps idle motion: the rationale is Canvas/WebGL, bounded idle-state-machine complexity, and capped ambient ornaments.

- No detectable memory growth over 30 minutes: the rationale is pooled audio resources, virtualized notebook list, and bounded worker/audio lifecycle.

- Simulation tick p99 latency under 5s: the rationale is small per-account transactions, bounded event-log reads, and indexed accounts-due queries.

- Synthetic browser checks: the rationale is monitoring page load and first-bird-render timing from multiple geographies.

- Aggregate RUM metrics: the rationale is operational visibility into frames, audio-context errors, tick latency, and anonymized session duration without per-account dimensions.

- Memory-growth regression test and drift-calibration test as release-blocking CI gates: the rationale is that these are "release-blocking, not advisory."

- Unsupported-browser page for older browsers: the plan specifies this, but its rationale beyond not maintaining compatibility shims is NOT RECOVERABLE FROM PLAN.

### 11. Rollout

- V1 launch with accessibility and core engine complete: the rationale is that reduced-motion, narration, and captions "cannot land as a post-launch fix."

- Internal dogfood: the rationale is validating drift calibration against real usage over 1-3 weeks because "feels alive over weeks" is a felt judgment, not only a numeric one.

- Soft launch with capped signups: the rationale is validating tick scheduler latency and multi-device sync behavior against real accounts and overlapping sessions.

- Open signup after tick latency, first-bird timing, and drift calibration hold: the rationale is production telemetry readiness.

- Birds-per-aviary ramp based on aviary age: the rationale is pacing without interaction count or tier pressure.

- Age thresholds as a tunable config table: the rationale is changing pacing post-launch without a code change.

- Fixed age-gated mechanism with no interaction-count aggregate: the rationale is privacy and anti-gamification; a future activity-based request would contradict the data model.

- Day-one aggregate RUM, synthetic checks, tick latency, audio errors, drift canary, and aggregate lifecycle funnel: the rationale is production observability without per-account interaction-pattern analytics.

### 12. Risks

- Drift calibration release gate and dogfood window: the rationale is avoiding both "Tamagotchi-by-accident" and "screensaver."

- Server-side config for drift time constant: the rationale is tuning without a client release if early production data misses the target.

- Sequence_no as the only ordering authority: the rationale is correctness under concurrent multi-device sessions and clock skew.

- Atomic per-account tick transaction: the rationale is preventing a tick crash from committing personality changes and watermark advancement separately.

- Overlapping-device integration tests with crash-and-retry: the rationale is ensuring no event is double-counted or dropped.

- Sound-design review cycles for call grammar: the rationale is that audio believability is not resolvable by engineering alone.

- Accessibility checklist gate for future bird-state-affecting features: the rationale is ensuring every new feature ships reduced-motion rendering and narration/caption text in the same change.

- Data model lacking streak, visit-count, or frequency aggregates: the rationale is making gamification creep require visible architectural change.

- Structurally separate visitor snapshot route and integration test: the rationale is ensuring visitor tokens never create InteractionEvent rows and visitor attention never drives host drift.

### 13. Privacy and telemetry boundary

- Per-bird interaction events and personality/mood state used only for that account's own simulation: the rationale is the privacy boundary.

- Analytics/telemetry warehouse never reading the aviary/simulation database: the rationale is preventing per-bird fields from becoming analytics inputs.

- Separate operational event emission path: the rationale is allowing request counts, latencies, error rates, and anonymized histograms without per-account dimensions.

- No aggregate pipeline that could be repurposed into gamification: the rationale is durability of no-streak-counter and no-visit-frequency-surface rules.
