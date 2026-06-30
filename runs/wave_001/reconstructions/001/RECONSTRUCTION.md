## System-level intent

1. Server-authoritative life, not client-authored state. The plan repeatedly makes the server the place where the aviary is real: "server-controlled growth," "server-side simulation tick," "server-canonical state," "the client never simulates," and "owns all canonical state." This shows up in the scope, client/server split, sync model, and "No last-write-wins for personality."

2. Structural prevention over policy or UI omission. The plan prefers making forbidden paths impossible: gamification primitives "must not exist even as disabled/internal-only code paths," the personality serializer has "no code path" that includes the vector, no recorded-audio asset pipeline exists, the quiet-field path has no reachable spinner component, notebook pattern inputs exclude user visit aggregates, and visitor tokens are scoped so visitors cannot write events.

3. Ambient naturalist product voice over system-surface intrusion. The product should not feel like a dashboard, game, or machine. This appears in "naturalist prose," "quiet field" instead of spinner, no location-permission prompt because that would be a "system-surface intrusion," no generic phrasing like "your bird is happier," and rollout review for "voice drift."

4. Calibration is visible and tunable rather than buried. The plan marks uncertain values with `[CALIBRATE]`, exposes bird-unlock pacing and drift weights through a config table, and requires drift calibration as a "required CI fixture" because drift is named a primary risk.

5. Growth and interaction must avoid gamification. Bird growth is "paced by aviary age (not engagement)," unlocks are "never engagement-driven," and the plan forbids streaks, scores, badges, counters, visit calendars, and even dormant scaffolds that could make those cheap to add.

6. The aviary continues without the viewer. The tick "must run for accounts with no connected client and no new events," the render loop can stop when hidden "without ever stopping the simulation," and settle closes a presence window without becoming a different engine state.

7. Privacy boundaries are load-bearing. Email appears in "exactly one column," all IDs are synthetic UUIDs, telemetry is "aggregate-only," the analytics warehouse never reads the simulation database, and CI/lint/type boundaries guard against regressions because the plan frames email-keying as "impossible to retrofit."

8. Accessibility is a designed v1 surface, not a fallback. Reduced-motion is "a parallel render mode," captions and narration share the naturalist voice system, keyboard navigation works inside the scene graph, and launch sequencing says these ship "with v1, not as a fast-follow."

9. Performance budgets are product requirements. The 500ms "time-to-first-bird" budget drives edge inlining, bundle budget, and critical render path; 60fps and no memory growth require CI perf tests, canvas/WebGL scene graph, audio pooling, and lifecycle bounds.

10. Procedural variation matters because repetition breaks the premise. Calls are "procedural, not looped," use motif parameters plus jitter so the same call is never "byte-identical twice," and chorus must be "true chorus" rather than stacked loops with phase artifacts.

11. Social is intentionally narrow and read-only. The plan allows a single visit affordance but rules out profiles, follows, feeds, comments, co-presence, shared cursor, and chat. Visitor attention "never drifts the host's birds," and visitor sessions are architecturally incapable of writing events.

12. Product surfaces should share one voice. Notebook prose, screen-reader narration, and call captions all use the same naturalist-voice template system so they do not read as "differently-voiced products glued together."

## Per-feature whys

### 0. Reading this plan

- `[CALIBRATE]` markers: The plan uses these so tunable choices are "visible" rather than "buried."
- `[DEFAULT]` decisions: The plan makes calls where engineering needs specificity so the team can execute "without returning to the PRD for clarification."

### 1. Scope

- Single-user accounts and one canonical aviary per account: The plan uses one canonical aviary so multi-device sync can read "the single canonical snapshot" and avoid client-to-client reconciliation.
- Magic-link auth: The account API always returns `202` for magic-link issuance to avoid an "account-enumeration leak."
- Two starter birds, server-controlled growth, 7-bird cap, and aviary-age pacing: The rationale is to keep growth paced by "aviary age (not engagement)" and prevent "interact more to unlock birds" from creeping in.
- Server-side simulation tick: The tick makes "the aviary continues without the viewer" true at the data layer and keeps personality/mood out of client authority.
- Client rendering with full visual scene, idle motion, day/night, weather, return-greeting, listen-in, offer, and settle: The plan ties these to rendering server snapshots as a living scene while keeping the client to rendering/interpolation rather than simulation.
- Procedural WebAudio call synthesis and chorus mixing: The rationale is "procedural, not looped" audio with calls that vary every time and a real-time chorus rather than stacked-loop artifacts.
- Silence plus captions fallback: The plan keeps "full visual fidelity" when WebAudio is unavailable and forces captions on by default instead of shipping a recorded-audio fallback.
- Field notebook, auto-generated, read-only, sparse: The rationale is naturalist observation of "noteworthy" aviary moments, not generic status or user-performance feedback.
- Visit invitation, read-only ambient view, opt-in, revocable: The rationale is a narrow social affordance where visitor attention "never drifts the host's birds" and revocation is immediate.
- Screen-reader narration, reduced-motion, call captions, keyboard navigation, WCAG AA: These are launch-critical accessible surfaces; a later reduced-motion mode is itself a "launch failure."
- Account export and soft-then-hard deletion: The rationale is account control with a 30-day "I changed my mind" window and hard deletion cascading across account-linked data.
- Aggregate-only operational telemetry: The rationale is that per-bird/per-account interaction data "never leaves the simulation database boundary."
- No native iOS/Android apps: The plan avoids native-client work while keeping HTTP/JSON snapshots from assuming a particular client runtime.
- No gamification primitives: They must not exist because even "build it disabled" is a risk and the forbidden features are the cheapest, most tempting additions after launch.
- No Tamagotchi mechanics: The rationale is to avoid death, hunger, decaying happiness, distress, and any drift that moves away from "monotonic toward expressive."
- No social-network surfaces beyond visits: The rationale is to avoid profiles, follows, feeds, discovery, friend-of-friend chains, and comments that would exceed the single visit affordance.
- Multi-aviary accounts and shared/household aviaries: NOT RECOVERABLE FROM PLAN
- Customizable scenes: NOT RECOVERABLE FROM PLAN
- Payments: NOT RECOVERABLE FROM PLAN
- Push notifications: NOT RECOVERABLE FROM PLAN
- Bird species pool count of 6 and species registry: The plan uses a static registry of silhouette, palette, and call-motif library so species assets and call grammar are versioned.
- Aviary-age unlock schedule in config: The rationale is to let product/design recalibrate exact pacing "without a deploy" while keeping unlocks server-side and not engagement-based.

### 2. Architecture

- Edge/CDN with initial snapshot inlined: The rationale is the "500ms time-to-first-bird budget" and avoiding a second round-trip before first paint.
- Stateless API service: The plan separates low-latency reads, event writes, auth, account management, visits, and notebook reads from tick correctness concerns.
- Decoupled simulation service: The rationale is that ordered event consumption and exclusive personality writes differ from horizontally scaled API requirements.
- Client never simulates: The rationale is to prevent clients from computing or sending absolute mood/personality and to keep all canonical state server-owned.
- Server computes call-grammar parameters while client synthesizes audio: This keeps calls "server-determined in character" while keeping bytes over the wire small.
- Render-pipeline boundary between simulation state and render state: This lets the client stop rendering when hidden without stopping simulation, and lets idle micro-motion feel continuous between minute-level snapshots.
- Source-of-truth transactional store: The rationale is canonical storage for accounts, birds, event log, notebook, invitations, and sessions keyed by synthetic UUIDs.
- Read-snapshot store: The rationale is cheap, low-latency snapshot pulls and keeping the tick the sole writer of derived state.
- Append-only event log: The rationale is that client interactions affect personality/mood only through ordered, idempotent event consumption.

### 3. Data model

- Synthetic UUIDs and encrypted email column: The rationale is that no table, log, queue, partition key, or telemetry event should be keyed by email.
- Personality vector omitted from API responses: The rationale is to ensure the user perceives traits through mood, perch, and call parameters rather than numeric personality values.
- Species registry with versioned call motif library: The rationale is to keep species visual assets and shared grammar definitions static-ish and versioned.
- Event `sequence` assigned by the API: The tick uses `sequence` rather than `occurred_at` because client-observed time is "untrusted for ordering."
- Aviary world timezone offset hint instead of geolocation: The rationale is that "their morning is the aviary's morning" without any location-permission prompt.
- Notebook entries with internal `trigger_kind`: The rationale is to generate prose from tick-detected patterns while never exposing the internal trigger form to the client.
- Notebook read-only with no edit/delete API: The rationale is structural read-only behavior, "not just a hidden one."
- Visit invitation visitors without Account records: Visitors are "not users of the product in the account sense."
- DeviceSession `user_agent_hint`: The rationale is the revocation UI label, such as "Safari on iPhone, last used 2 days ago."
- Bird name user-editable with no effect on engine: NOT RECOVERABLE FROM PLAN
- `is_nightjar_class`: NOT RECOVERABLE FROM PLAN

### 4. Simulation engine design

- 60-second tick cadence: The plan ties this to the PRD's "~once per minute" and marks exact cadence as calibratable.
- Tick runs for accounts with no connected client and no new events: The rationale is that time-of-day and weather mood transitions happen regardless of interaction.
- Single transaction per due account with cursor advancement: The rationale is idempotency; a crashed or retried tick pass must never double-apply a delta.
- Account UUID hash sharding: The rationale is that accounts have no cross-account simulation, since visits are read-only.
- Weighted mood scoring function: The rationale is to combine time-of-day, recent interactions, ambient events, and personality bias into the current mood.
- Mood hysteresis: The rationale is to prevent flapping between adjacent moods on borderline ticks.
- Mood persistence across sessions: The rationale is to satisfy "mood does not reset on tab open."
- Jittered daily-ish mood reset: The rationale is to avoid a synchronized midnight reset that would read as "systemic rather than alive."
- Non-negative personality deltas: The rationale is that monotonic-toward-expressive is load-bearing, and a sign error would be the "single most damaging regression" the service could silently ship.
- Presence-time as dominant drift input: The rationale is sustained attention leading to visual richness and willingness to call.
- Listen-in drift: The rationale is that focused listening specifically affects `social_warmth` and `vocal_frequency`, capped so one long listen-in does not dominate.
- Offer drift: The rationale is that accepting an offer affects `curiosity`, and offering near a bird affects `boldness`.
- Settle with no directional drift delta: The rationale is that settle only closes the presence window cleanly.
- Drift calibration harness in CI: The rationale is that drift calibration is a primary risk and must be a tracked fixture, not a one-off manual check.
- Presence pings only while visible, focused, and recently active: The rationale is to count qualifying attention using browser signals where those signals actually live.
- Server accumulation of ping count times interval: The rationale is to keep the authoritative definition server-enforced and prevent a compromised client from inflating elapsed time.
- No client-side grace period for presence: The rationale is to avoid loosening the definition and to prevent intermittent pings near the boundary from being rounded up.
- Personality vector never exposed through browser-reachable APIs: The rationale is to keep "is personality reachable from the client" a single, auditable boundary.
- Server-side call parameter selection: The rationale is that what a bird sounds like remains driven by server-authoritative state.
- Client-side call synthesis with micro-variation: The rationale is that the same motif never sounds byte-identical twice without the server enumerating every variation.
- Real-time chorus mixing: The rationale is genuine phase/timing interaction instead of stacked-loop phase cancellation.
- Wary mood propagation: The rationale is propagation, not instant synchronization, across nearby birds.
- Chorus eligibility bonus: The rationale is emergent joint-calling "without being scripted."

### 5. API surface

- `GET /aviary/state` from fast snapshot store: The rationale is low-latency canonical state delivery with "No personality fields, ever."
- `?reason=` hint on state reads: The rationale is server-side metrics about snapshot-pull patterns with no effect on response content.
- Return-greeting encoded in the snapshot: The rationale is to reuse call-grammar descriptors and let the client render the greeting without a separate endpoint or server-shipped audio.
- `POST /aviary/events` as only mood/personality mutation path: The rationale is append-only ordering with server-assigned `sequence`.
- Single events endpoint for presence, listen-in, offers, settle, and undo-settle: The rationale is shared auth, validation, and ordering concerns with a small stable type enum.
- Event rate limiting: The rationale is to prevent a misbehaving client from flooding the log while staying above legitimate interaction rates.
- Notebook `GET` only: The rationale is that no write/edit/delete endpoints exist at all.
- Magic-link issuance always returning `202`: The rationale is to avoid an account-enumeration leak.
- Magic-link verification invalidates the link: The rationale is single-use auth behavior.
- Account export via async emailed download link: The rationale is consistency with "emailed to the verified address," not synchronous API return.
- Account deletion with undelete window and scheduled hard delete: The rationale is a reversible 30-day pending state followed by cascading deletion.
- Visit invitation one-time token redemption: The rationale is a token-scoped visitor session for one host account, with revoked/expired links returning a matter-of-fact unavailable surface.
- Visitor sessions rejecting event writes by token scope: The rationale is that visitor attention must never drift the host's birds.
- Accessibility settings endpoint: The rationale is to persist explicit reduced-motion, captions, and narration preferences across devices while still honoring `prefers-reduced-motion` client-side.

### 6. Sync model

- Multi-device sync without client-to-client logic: The rationale is that every device reads the same canonical snapshot and no device holds authoritative state.
- Snapshot refresh on visibility, resume gap, and keepalive: The rationale is to keep the client near the server's latest pass without client simulation.
- Immediate bounded refresh after visible interactions: The rationale is to show reactions sooner without blocking the UI on tick processing.
- "Preventing conflicts" instead of resolving them: The rationale is that clients never submit absolute state, so there is structurally no conflict to resolve.
- No last-write-wins for personality: The rationale is that only the tick writes personality under a single-account transaction and API request schemas have no personality-shaped field.

### 7. Privacy and identity

- Synthetic account ID everywhere: The rationale is that email-keyed systems are "impossible to retrofit" away from later.
- CI guard against new email fields: The rationale is to catch regressions where schema or log-schema fields named `email` appear outside the account service module.
- Separate simulation database and aggregate telemetry pipeline: The rationale is that analytics must never read bird/account interaction data or export it to ML/training.
- Telemetry SDK type rejection of per-bird/per-account fields: The rationale is to enforce aggregate-only telemetry in code, not merely by convention.
- Export JSON includes current personality values and moods: The rationale is account export completeness, delivered through a verified-address email link.
- Hard deletion cascades through account-linked telemetry references: The rationale is that even operational-alerting references still carry account UUIDs and should be cleaned up.

### 8. Frontend rendering pipeline

- Canvas or WebGL-lite 2D scene graph: The rationale is 60fps idle motion on a 5-year-old laptop without layout thrash.
- Birds as sprite/vector entities with pose-blend state: The rationale is continuous micro-motion that would be a poor fit for DOM nodes with CSS animations.
- First snapshot rendered mid-pose: The rationale is to avoid a neutral/idle default and get to first bird immediately.
- Quiet-field loading mode instead of spinner: The rationale is to honor the aviary surface; a shared spinner must not be reachable so the rule is enforceable by code review.
- Mood-shaped pose-blend rendering: The rationale is that mood drives perch and pose, while the client reads mood directly and does not infer it.
- Day/night from server-computed local time phase with client interpolation: The rationale is gradual visual transition despite once-per-tick snapshots.
- Weather as server snapshot field but ambient ornaments as client-only: The rationale is that per-leaf state would be pointless server load for something with no simulation meaning.
- Responsive proportional perch zones: The rationale is to avoid cropping birds on narrow viewports and avoid popping during resizing.
- Reduced-motion parallel render mode: The rationale is to avoid the "stripped-fallback failure mode" and make reduced motion authored alongside full motion.
- Settle gesture: The rationale is a rendering and engagement-window gesture that quiets the aviary and ends presence without becoming a different engine state or adding drift weight.
- Five-second settle undo: The rationale is to locally revert the lighting shift and write `undo_settle` within a small bounded window.

### 9. Audio pipeline

- WebAudio procedural synthesis with jitter: The rationale is that no two plays of the same call are byte-identical.
- Audio node and buffer pooling: The rationale is to satisfy the no-memory-growth budget.
- Shared `AudioContext` and per-bird gain chains for chorus: The rationale is true real-time chorus rather than pre-rendered layering.
- Listen-in gain ramps: The rationale is focus without ever reducing other birds to zero, because the plan says "never go silent."
- Single listen-in mix-state machine: The rationale is to avoid double-ramp glitches when switching focus directly between birds.
- WebAudio unavailable fallback with captions on by default: The rationale is full visual fidelity with silence plus captions when audio synthesis cannot run.
- No recorded-audio fallback path: The rationale is to remove the "quick fallback" under deadline pressure and keep the procedural-audio premise honest.

### 10. Accessibility surfaces

- Screen-reader narration generator: The rationale is live naturalist prose from snapshot state on an `aria-live="polite"` region.
- Narration priority staying polite: The rationale is to avoid interrupting screen reader output with an announcement-like behavior the voice rules are wary of.
- Narration sharing notebook voice logic: The rationale is that the two surfaces should not read as differently voiced products glued together.
- Caption text generated from selected motif and parameters: The rationale is that captions match what was actually played, not a fixed motif string.
- Captions near the calling bird: The rationale is to pair the text with the actual audio/visual source.
- Keyboard navigation through birds, listen-in, offers, and settle: The rationale is full keyboard reachability for the scene and top-bar affordances.
- Scene-graph focus outline: The rationale is that birds are canvas/WebGL entities and the indicator must be legible against both bright and dim palettes.
- WCAG AA contrast checks in CI: The rationale is enforceable contrast for all user-copy text through design-token and visual-regression assertions.

### 11. Performance budgets and observability

- Initial JS under 2MB gzipped: The rationale is the critical path budget, driving procedural audio, compact assets, and code splitting.
- Code-splitting account/settings/visit surfaces: The rationale is to keep infrequently reached surfaces out of the critical path.
- Time-to-first-bird under 500ms: The rationale is immediate first paint from inlined snapshot before audio, settings, or notebook code loads.
- Synthetic and field measurement gated in CI/release: The rationale is that a release regressing the synthetic check blocks deploy.
- 60fps idle motion for 30 minutes: The rationale is sustained runtime quality, validated against a representative low-end device tier rather than manual spot checks.
- No memory growth over 30 minutes: The rationale is a real CI test, backed by audio pooling, notebook virtualization, and bounded worker/AudioContext lifecycle.
- Synthetic checks from multiple geographies: The rationale is to exercise load to first-bird render and basic interaction across delivery conditions.
- Aggregate RUM and tick-latency alarm: The rationale is operational visibility without per-account interaction data, including a p99 tick latency alarm above 5s.
- Browser support last two major versions: The rationale is the plan's cost-benefit line against graceful feature-by-feature degradation below that support boundary.

### 12. Field notebook generation

- Tick-coupled notebook generation: The rationale is pattern-matching "noteworthy" simulation history near the source of canonical behavior.
- Candidate notebook patterns: The rationale is to write about aviary behavior such as greeting order, quiet stretch, mood-and-perch combinations, and weather coinciding with behavior.
- Naturalist prose templates: The rationale is specificity in bird names, timing, and behavior, never generic phrasing like "your bird is happier."
- Sparse entry cadence: The rationale is roughly one entry every few days and "never one entry per session."
- Cooldown/budget rather than pure pattern-frequency cap: The rationale is to preserve sparsity even for very active users.
- Observation/behavior boundary: The rationale is to keep notebook entries about the aviary, never the user.
- Pattern functions excluding visit-frequency aggregates: The rationale is to make a "you've been here every day this week" entry structurally unreachable.

### 13. Rollout

- Accessibility surfaces shipped with v1: The rationale is that delayed reduced-motion is a launch failure, so accessibility is a go/no-go gate.
- Bird-count ramp starting at two server-selected species: The rationale is to follow the adoption-flow rule and prevent user choice from becoming an early customization/growth vector.
- Third-bird and beyond through scheduled age checks: The rationale is to keep unlocks purely aviary-age-driven and never engagement-driven.
- Day-one aggregate RUM, tick alarms, drift trends, bundle gate, and WebAudio fallback counter: The rationale is to see performance, calibration, and silence-plus-captions experience from launch.
- Gradual exposure: The rationale is to gate rollout on synthetic performance, memory/perf CI, and manual review of notebook/narration strings.
- Manual voice-quality review before full exposure: The rationale is to catch "generic-sounding output" and voice drift before scale.

### 14. Risks

- Drift calibration mitigation: The rationale is that too-fast drift becomes "Tamagotchi-shaped," while too-slow drift becomes the "screensaver" failure.
- Sync load-testing for many accounts due simultaneously: The rationale is that tick delays under contention could look like "my bird didn't react."
- Sound-design iteration and listening-review gate: The rationale is that robotic or repetitive procedural synthesis would defeat the reason for avoiding recorded audio.
- Accessibility PR checklist trigger: The rationale is that bespoke accessible surfaces can drift out of sync when species, moods, or interactions change.
- No dormant achievements or gamification tables: The rationale is to make gamification creep structurally hard rather than merely forbidden by policy.
- Visitor token-scope enforcement against writes: The rationale is that co-presence would require a fundamentally different multi-user simulation model, so it must not fall out of a UI relaxation.
