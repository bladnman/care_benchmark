## System-level intent

1. **A calm, already-running place, not a task loop.** The plan repeatedly frames Pocket Aviary as "one calm, horizontal scene" where "a first frame must look like a place already in motion." This shows up in the product contract, the scene renderer's "motion already in progress," and the performance risk around the "already-running conceit."

2. **Personality should be felt, not optimized.** The plan wants a user to "feel personality drift over weeks without ever being shown a stat to optimize." That intent appears in hidden personality vectors, no visible progress system, no "trait numbers," no "visit counts," no "achievement language," and the final acceptance language of "no stat-management UI."

3. **The server owns canonical life.** The client "never owns canonical mood, position, personality, presence totals, notebook truth, or bird identity," and the simulation service is the only personality-state writer. This principle appears in the architecture boundary, API rejection of engine fields, snapshot acknowledgement as the canonical boundary, and sync-corruption mitigations.

4. **Slow, forgiving drift with no guilt for absence.** Drift is "slow," "additive," "dominant" from presence-time, and must "never" move negatively because of neglect. The plan ties this to one-week instrument-visible and three-week user-visible targets, and closes with "no guilt for absence."

5. **Honest presence, not tab inflation.** Presence is only counted when visibility, focus, and recent pointer/key activity all hold, with bounded heartbeats and server deduplication. The plan explicitly rejects "unbounded 'tab open' flags," "background tabs," and client-claimed duration.

6. **Quiet social access stays narrow and non-influential.** Visits are "quiet," "opt-in," "per-invite," "read-only," revocable, expiring, and excluded from owner presence/events. The risks section names the principle directly: visits must not "expand into a social network or influence simulation."

7. **Privacy boundaries are part of the product shape.** The plan uses a "synthetic UUID boundary," encrypted email, aggregate-only telemetry, and a simulation store isolated from analytics. It says no per-bird state should cross into telemetry and warns against "privacy leakage through convenience identifiers."

8. **Naturalist voice belongs to the aviary; system language stays direct.** Notebook and narration use "lowercase, present-tense naturalist prose," while API/system/account/error surfaces use "direct matter-of-fact language" and "never naturalist euphemisms."

9. **Accessibility ships as the same experience, not a fallback.** The accessible surface must ship "in the same milestone as the visual scene." Narration, captions, reduced motion, keyboard access, contrast, and screen-reader tests are treated as "launch blockers."

10. **Determinism and replay protect the simulation.** The tick must be deterministic for fixed inputs, store engine/config versions, advance an event cursor, and support replay tooling. This appears in the tick algorithm, delivery foundations, test strategy, and scheduler/catch-up mitigations.

11. **Compact delivery supports affect.** The plan's 2 MB bundle, 500 ms first-bird gate, edge-friendly snapshot, lazy secondary surfaces, and synthetic monitoring all serve the first-frame and "already-running" product contract.

12. **Operational hardening must preserve tone.** Rollout flags, release gates, test matrices, and risk reviews are tied to protecting "no stat-management UI," "no notification-driven social loop," "no visitor influence," and "no client-owned canonical state."

## Per-feature whys

### 1. Product contract and v1 scope

- **Browser-only, single-user virtual aviary** - Why: The v1 contract is one account, one calm place, and explicitly excludes native apps, shared accounts, and multi-aviary accounts to preserve the single canonical aviary shape.
- **Two starter birds sharing one calm, horizontal scene** - Why: The initial scene must be complete with "two starters" and make the first frame feel like "a place already in motion."
- **One canonical aviary per account**: NOT RECOVERABLE FROM PLAN
- **Seven-bird v1 ceiling**: NOT RECOVERABLE FROM PLAN
- **Additional birds by aviary age** - Why: The plan says new birds become available by "aviary age, never by visit count, interaction score, payment, or a visible progress system," keeping users away from optimization loops.
- **No visible progress system for bird additions** - Why: The plan rejects counters and progress UI so bird availability does not become a visit-frequency or stat-management mechanic.
- **Email magic-link authentication** - Why: The plan excludes passwords/SSO and specifies one-time, expiring links with rate limits and transactional consumption for a narrow account-access surface.
- **Exact 15-minute magic-link expiry**: NOT RECOVERABLE FROM PLAN
- **Per-device sessions** - Why: Per-device session tokens make revocation immediate for API authorization and allow safe display of device metadata.
- **Account settings** - Why: Settings carry accessibility, audio, timezone, and notification preferences while rejecting engine fields, so user controls do not become simulation controls.
- **Account export** - Why: Export gives an authenticated JSON snapshot but is treated as sensitive, generated asynchronously, mailed only to the verified address, and kept out of analytics.
- **Account deletion and recovery window** - Why: Deletion is marked immediately, allows recovery, and later hard-deletes account-tied records, serving account control and privacy.
- **Exact 30-day deletion recovery window**: NOT RECOVERABLE FROM PLAN
- **Stable bird identities** - Why: Bird UUIDs are stable, and name changes "never replace the UUID or alter engine state," preserving identity and simulation continuity.
- **Renameable user-assigned bird names** - Why: Users can personalize names without exposing vector fields or changing engine truth.
- **Hidden personality vectors** - Why: The plan wants personality drift to be felt without stats to optimize, so vectors are not returned as ordinary UI state or shown in settings.
- **Moods, calls, and bird-to-bird reactions** - Why: These are part of making the aviary feel alive, with mood/call effects advanced by ticks and influenced by time, weather, interactions, and personality.
- **Server-side simulation tick without a connected client** - Why: The aviary evolves through slow server-side simulation and "runs whether or not a client is connected."
- **Single-screen scene** - Why: The aviary is a single calm place; the renderer must preserve every bird in frame and never pan, scroll, zoom, crop, or allow user placement.
- **Day/night cycle** - Why: Local day/night supports scene tone and mood time-of-day signals, while night is not treated as a dead state.
- **Rare ambient weather**: NOT RECOVERABLE FROM PLAN
- **Idle micro-motion** - Why: Preen, scan, head-tilt, and weight-shift derive from mood/personality presentation inputs so the birds convey life without visible stats.
- **Return-greeting** - Why: "The birds noticing the user is the welcome surface," replacing a generic welcome banner or absence announcement.
- **Listen-in** - Why: It gives attention to one bird while others fall only to ambient, preserving the chorus and avoiding silence.
- **Offers** - Why: Offers create server-approved, cooldown-bound bird reactions that reflect mood, without direct personality mutation.
- **Settle** - Why: Settle ends presence, ramps lighting/calls down, and gives a quiet boundary without requiring tab close.
- **Presence accounting** - Why: Presence is the dominant drift input but must be honest, bounded, and invisible as a metric.
- **Read-only field notebook** - Why: Notebook entries are sparse naturalist observations, not raw event logs, visit counts, achievement language, or a user-behavior report.
- **Multi-device snapshot sync** - Why: The server remains the only personality-state writer, preventing last-write-wins personality overwrites when devices interleave events.
- **Screen-reader naturalist narration** - Why: Narration gives an accessible version of the same snapshot/observation model without exposing hidden numeric traits.
- **Reduced-motion rendering** - Why: Motion-sensitive users retain mood, drift, calls, captions, and notebook behavior while micro-motion and flight paths become calmer cross-fades.
- **Procedural-call captions** - Why: Captions are generated from the actual scheduled motif, making the sensory experience available when audio is off or unavailable.
- **Keyboard access** - Why: Birds, listen-in, offer, settle, dialogs, and focus restoration must be operable without a pointer.
- **WCAG AA user-copy contrast** - Why: All visible copy, captions, narration shown visually, settings, account, error, and unsupported-browser surfaces must remain readable.
- **Quiet opt-in read-only visit flow** - Why: It allows invited viewing while keeping visitor capabilities from becoming owner access or influencing simulation.
- **Visit revocation and expiration** - Why: The host controls access, every pull rechecks revocation/expiration, and revoked visits get a direct "visit no longer available" surface.
- **Exact 30-day unused invite expiration**: NOT RECOVERABLE FROM PLAN
- **Host visit log** - Why: The host can see recipient, date, approximate duration, and outstanding invites, but the plan forbids badges or push so it does not become a notification loop.
- **Exclusion of Tamagotchi mechanics** - Why: The plan rejects hunger, death, distress, and decaying happiness to avoid guilt and absence punishment.
- **No generic welcome banner, toast, spinner, or away announcement** - Why: The birds themselves are the welcome surface, and the first frame should already be moving.

### 2. Architectural shape

- **Browser client boundary** - Why: The client can render, interpolate, handle audio/focus/captions, and hold transient ornaments, but it must not own canonical mood, position, personality, presence totals, notebook truth, or bird identity.
- **Application/API service boundary** - Why: It centralizes auth, sessions, settings, snapshot reads, event validation, notebook/export/deletion, and permissions while never accepting absolute personality values from clients.
- **Simulation service boundary** - Why: A durable worker with a per-aviary lease consumes ordered events and advances canonical state even when no client is connected.
- **Persistence and delivery boundary** - Why: Account/aviary state, append-only events, and cursors need a transactional consistency domain, while initial HTML and compact snapshots can be edge-friendly.
- **Dedicated email provider** - Why: Magic links and invitations are email-delivered account/visit mechanisms; the plan does not route them through the simulation or telemetry system.
- **Aggregate-only operational telemetry pipeline** - Why: Telemetry must not read per-bird simulation records or user relationships.
- **Pull-based synchronization for v1** - Why: Low-frequency pulls on load, resume, visibility restoration, and keepalive are enough for kilobyte snapshots while preserving the canonical protocol for future push transport.
- **Monotonic snapshot version** - Why: Versions let clients reason about freshness without merging state.
- **Server time for ordering, ticks, expiration, and cursors** - Why: Server time is authoritative for event order, elapsed tick time, token expiration, and cursor advancement.
- **Stored or validated browser timezone** - Why: Local day/night and mood time-of-day signals should reflect the user's local context.
- **Visitor snapshot projection** - Why: Visitors receive the same host view but no owner capabilities and no presence or interaction writes.

### 3. Data model and invariants

- **Versioned records with generated UUIDs** - Why: Immutable creation identifiers, timestamps, and optimistic/version fields support stable identity, concurrency, replay, and audit.
- **Account synthetic UUID primary key** - Why: The plan says never to derive keys, partitions, telemetry dimensions, or log identifiers from email.
- **Encrypted email record** - Why: Email is account-sensitive data and must remain separate from telemetry/logging identifiers.
- **Verified-email status** - Why: Export links and account email flows are tied to the verified account address.
- **Visit-notification preference off by default** - Why: Visit notifications must not become a notification-driven social loop.
- **Deletion status/deletion deadline** - Why: Deletion has an immediate marked state and later hard-delete window.
- **Session random opaque token hash** - Why: API authorization can be revoked immediately without exposing raw session tokens.
- **MagicLink hashed one-time token** - Why: Transactional consumption invalidates the token immediately and makes replay/expiry a clear system error.
- **Aviary age epoch** - Why: Server-derived aviary age controls bird eligibility without visit counters or progress UI.
- **Aviary tick cursor and last tick time** - Why: The simulation can process ordered events once, catch up after delays, and resume safely.
- **Bird stable UUID and immutable identity metadata** - Why: Identity persists through rename and does not alter engine state.
- **BirdPersonality one canonical server-side row** - Why: Personality values are bounded, versioned, and changed only by the simulation service.
- **BirdMood persisted current state** - Why: Mood "persists across sessions" and is changed by ticks, not reset on client open.
- **BirdPresentationState separated from personality** - Why: Rendering can interpolate perch/action/call transitions without mutating engine truth.
- **SpeciesDefinition v1 pool of approximately six species**: NOT RECOVERABLE FROM PLAN
- **Species call signatures, motif grammar, caption vocabulary, and calibration version** - Why: Species definitions support recognizable procedural calls, captions, and versioned calibration.
- **Rarity not as a product mechanic** - Why: The plan states rarity should not become a mechanic, consistent with avoiding optimization and visible progress systems.
- **InteractionEvent idempotency key** - Why: Event appends and retries can be deduplicated and processed exactly once.
- **Presence as intervals or bounded heartbeats** - Why: Presence should not be an unbounded "tab open" flag.
- **Client event time for diagnostics only** - Why: Server receive time controls simulation order.
- **NotebookEntry read-only client surface** - Why: Notebook truth is generated server-side and cannot become client-authored history.
- **Notebook sparse generation budget** - Why: Entries should remain observations, not a session/event feed.
- **Invite per-recipient encrypted recipient email** - Why: An invite never creates a global discoverability flag.
- **VisitSession opaque visitor capability** - Why: Visitors can read authorized projections but cannot append owner interaction or presence events.
- **Exactly one aviary per account invariant**: NOT RECOVERABLE FROM PLAN
- **No more than seven birds invariant**: NOT RECOVERABLE FROM PLAN
- **Personality writes only from simulation service invariant** - Why: Hidden vector truth must not be client-owned or overwritten by sync.
- **Visitor capabilities cannot upgrade to owner access invariant** - Why: Visit access is read-only and narrow.
- **No per-bird state crosses into aggregate telemetry invariant** - Why: Analytics must not reconstruct relationships, mood, personality, or per-bird records.

### 4. API and authorization surface

- **Explicit, versioned API contracts** - Why: Server timestamps and snapshot versions let clients reason about freshness without merging state.
- **POST /auth/magic-link** - Why: Email validation, rate limiting, one-time link creation, and a matter-of-fact confirmation form the passwordless account entry point.
- **POST /auth/magic-link/consume** - Why: Atomic consumption and per-device session issuance prevent replay while giving clear expiry errors.
- **GET /me and settings/session APIs** - Why: They expose only needed account/settings/session data and keep engine fields out of account controls.
- **PATCH /me/settings** - Why: Users can adjust accessibility, audio, timezone, and notification preferences, but engine state is rejected.
- **Session revocation APIs** - Why: Per-device sessions can be revoked immediately for authorization.
- **Export/delete/cancel APIs** - Why: Account data can be exported, deletion can be marked with recovery, and hard deletion follows the window.
- **GET /aviary/snapshot** - Why: It returns the current canonical snapshot and must not trigger a simulation tick or synthesize drift on demand.
- **POST /aviary/events** - Why: Typed idempotent batches capture allowed owner inputs while rejecting personality and mood writes.
- **GET /aviary/notebook** - Why: Notebook entries are chronological, paginated, read-only, and contain no visit-frequency statistics.
- **POST /aviary/birds/:id/name** - Why: Names can change without exposing vector fields or replacing bird identity.
- **Visit invite create/list/revoke APIs** - Why: The host controls per-recipient invites with expiration and revocation.
- **Visit bootstrap/snapshot APIs** - Why: Visitor capabilities are validated at bootstrap and every pull so revoked or expired access ends promptly.
- **Revoked active visit surface** - Why: The plan requires the matter-of-fact "visit no longer available" surface.
- **GET /visits/log** - Why: The host can review invites and approximate visit duration on demand, without badges or push.
- **CSRF protection for cookie-backed mutations** - Why: Mutating endpoints need strict origin/token handling.
- **Rate limits on link and invite issuance** - Why: Magic-link and invite flows are sensitive issuance surfaces.
- **Short-lived visitor tokens** - Why: Visitor capabilities should remain narrow and revocable.
- **Audit records with synthetic UUIDs** - Why: Auditing should avoid email/token leakage and stay on the synthetic identifier boundary.
- **Direct system API error strings** - Why: Account, sync, accessibility, browser, and revoked-visit failures should not be disguised as naturalist prose.

### 5. Presence protocol and simulation engine

- **Presence requires visibility, focus, and recent pointer/key activity** - Why: Presence should represent actual owner attention, not a hidden or unfocused tab.
- **Generous inactivity grace for quiet watching** - Why: The plan wants quiet watching to count as presence even without constant input.
- **No presence while hidden, unfocused, settled, or inactive** - Why: This blocks background-tab and settled-state inflation.
- **Settle and tab close end presence but are not required** - Why: A valid session should not depend on a user explicitly pressing settle.
- **Server-deduplicated bounded presence intervals** - Why: Overlaps and impossible durations are capped, and a client cannot claim presence by duration.
- **Visitor sessions never counted as presence** - Why: Visitors must not influence the host's simulation.
- **Approximately one-minute tick cadence** - Why: The plan wants slow server-side simulation with durable catch-up; the exact cadence itself is not further justified.
- **Durable per-aviary lease** - Why: It prevents concurrent workers from corrupting the same aviary state.
- **Catch-up path for delayed workers** - Why: Scheduler delays should not leave the aviary stale.
- **Ordered event consumption exactly once** - Why: The cursor/idempotency boundary supports retry, audit, and replay.
- **Mood timer, time-of-day, weather, ambient, call, and presentation advancement** - Why: Mood is a small fast state influenced by interactions, local time, weather, alarms, and personality.
- **Additive drift deltas from presence, listen-in, offers, and settle** - Why: Regular attention can slowly move expressive traits, while neglect alone never causes negative movement.
- **Low-pass drift filter and clamping** - Why: Drift should be detectable after about a week, felt after about three weeks, and avoid visible session-by-session jumps.
- **Transactional update of personality, mood, presentation, cursor, and snapshot version** - Why: Retries are safe and failures do not leave partial drift.
- **Sparse notebook observation generation** - Why: The notebook should record noteworthy naturalist observations, not raw logs, trait numbers, visit counts, or achievements.
- **Deterministic tick for fixed state, events, time, seed, and version** - Why: Determinism enables test fixtures and production incident diagnosis.
- **Engine/config versions stored with transitions** - Why: Calibration changes should not silently rewrite history.
- **Replay tooling** - Why: The team can diagnose production incidents without exposing per-account records to analytics.
- **Server-derived age-based bird availability** - Why: Bird offers appear without counters or progress UI and never reset existing vectors or moods.
- **Seven-bird terminal cap for offers**: NOT RECOVERABLE FROM PLAN

### 6. Frontend rendering and interaction pipeline

- **Critical scene path plus lazy secondary surfaces** - Why: First bird rendering should not wait on settings, notebook history, invitations, export, deletion UI, or audio permission.
- **Compact initial snapshot data and species render definitions** - Why: The scene can draw immediately from canonical state with the data needed for birds and procedural audio grammar.
- **Single horizontal responsive viewport** - Why: The whole aviary remains one screen with every bird visible on phones and wide desktops.
- **No pan, scroll, zoom, crop, or user placement** - Why: The scene remains calm and canonical rather than a customizable layout.
- **Initial pose/action phase and seed from snapshot** - Why: Birds appear in motion immediately instead of entering through an animation or spinner.
- **Quiet field while initial state unavailable** - Why: It avoids a generic loading spinner while state is not yet available.
- **Genuine empty state only during adoption** - Why: Empty state is reserved for a real product condition, not normal loading.
- **Interpolation between snapshot positions/actions** - Why: Birds do not teleport when the one-minute tick changes a target.
- **Hidden-tab pause and resume resync** - Why: The client saves work while hidden and corrects state after visibility return or long frame gaps.
- **Idle micro-motion from mood/personality presentation inputs** - Why: The visual behavior expresses hidden traits without exposing them.
- **Client-only leaf/feather ornaments** - Why: Ornaments can enrich the scene without becoming server-simulated per-leaf records.
- **Local-time day/night palette and weather state** - Why: The scene reflects local time and ambient state while supporting mood and active night species behavior.
- **Thin top bar for chrome** - Why: Account, settings, accessibility, notebook, offer, and settle stay out of the scene itself.
- **Top-bar fade after cursor stillness** - Why: Chrome recedes so the birds remain the welcome and focus surface.
- **No buttons, badges, hover tooltips, inline labels, or overlays in the scene** - Why: The plan protects the field from visible UI clutter and metric-like surfaces.
- **Settle in the top bar** - Why: Settle is an explicit quiet command with lighting/call ramps and a five-second undo, not an intrusive modal.
- **Five-second click-anywhere settle undo** - Why: Settle can be reversed briefly without turning it into a persistent prompt.
- **Interactions implemented as explicit state machines** - Why: Return-greeting, listen-in, offer, settle, and notebook flows have bounded transitions, event emission, and canonical acknowledgement points.
- **Return-greeting bird selection from boldness, mood, absence length, and seed** - Why: A short return should feel different from a long absence without textual welcome or away announcements.
- **Listen-in gain ramps** - Why: Focused audio rises while other birds remain audible at ambient level; the aviary never hard-cuts to silence.
- **Listen-in start/end events with bounded duration** - Why: Listen attention can influence drift without giving the client unbounded control.
- **Offer menu from top bar** - Why: Offers stay a quiet affordance rather than scene clutter.
- **Server-approved offer target and cooldown rules** - Why: Reactions come from canonical validation, not client-side personality mutation.
- **Mood-shaped offer reactions** - Why: Curious/content, wary, and drowsy birds react differently, making personality legible without stats.
- **Procedural song-fragment offer** - Why: The motif is soft/procedural and not a downloaded recording.
- **Settle event with lighting and call ramps** - Why: Settle quiets the scene over seconds and ends presence cleanly.
- **Notebook UI as sparse read-only entries** - Why: It must not become a session/event feed or user-behavior report.
- **Snapshot acknowledgement boundary** - Why: Provisional animations are allowed, but the next snapshot is authoritative for personality, mood, identity, and notebook history.

### 7. Audio and caption pipeline

- **Client-side WebAudio graph** - Why: Procedural oscillator/noise/filter/envelope calls can be generated without recorded call loops.
- **Reusable nodes or pooled buffers** - Why: Audio resources stay bounded and can be disposed or reused.
- **Stable species/personality call signature** - Why: Calls remain recognizable across mood and drift.
- **Fresh call synthesis from motif grammar** - Why: Runtime variation prevents identical calls while preserving recognition.
- **Mood-shaped pitch/timing and vocal-frequency cadence** - Why: Audio reflects current mood and hidden personality presentation inputs.
- **Runtime call seeding** - Why: Calls vary without losing stable signature.
- **Master ambient bus and per-bird buses** - Why: The chorus can mix focused, ambient, weather, offers, and return-greetings without hard cuts.
- **Listen-in bus gain ramps never to zero** - Why: Other birds fall to ambient, preserving the aviary as a living chorus.
- **Small compact call grammar** - Why: The grammar must fit the initial bundle and avoid recorded loops.
- **Caption descriptor generated with scheduled call** - Why: Captions describe the actual motif, not a static per-bird string.
- **Caption near the calling bird with fade** - Why: The caption is visually tied to the call's lifetime and source.
- **Narration queue exposure for captions when appropriate** - Why: Procedural calls remain accessible beyond visual captioning.
- **Graceful silence when WebAudio fails** - Why: Visual simulation continues and captions default on rather than introducing recorded-audio fallback.
- **Clear audio setting and matter-of-fact audio errors** - Why: Audio permission/failure is handled directly while the rest of the aviary remains functional.

### 8. Accessibility and inclusive surfaces

- **Accessible surface in the same milestone as the visual scene** - Why: Accessibility is a launch blocker, not a later degraded fallback.
- **Polite bounded screen-reader narration queue** - Why: Narration comes from the same scene model without flooding the queue.
- **Idle narration every 30-60 seconds**: NOT RECOVERABLE FROM PLAN
- **Narration priority for return-greeting, successful offer, and settle** - Why: Important bird/user moments are surfaced first without exposing raw mood or trait state.
- **No primary exposure of mood labels, perch IDs, personality numbers, or raw transitions** - Why: Accessibility should preserve the naturalist experience and hidden-trait intent.
- **prefers-reduced-motion support** - Why: Motion reduction follows user/system preference while preserving core mood, drift, calls, captions, and notebook behavior.
- **Explicit accessibility motion setting** - Why: Users can choose reduced motion independent of system preference.
- **Micro-motion and flight replaced with cross-fades** - Why: The visual surface remains calmer while still showing state changes.
- **Leaf drift removed in reduced motion** - Why: Decorative motion is unnecessary for reduced-motion users.
- **Captions toggle** - Why: Users control captions, and captions default on if audio fails.
- **Caption contrast requirements** - Why: Captions must be readable as part of WCAG AA user-copy contrast.
- **Keyboard map for top bar, scene birds, listen-in, offer, settle, and dialogs** - Why: The full experience must be reachable and operable without a pointer.
- **Visible focus outlines over bright and dim scene states** - Why: Keyboard focus must remain perceivable across day/night palettes.
- **Semantic names and roles for birds/actions** - Why: Assistive technology can describe the experience without revealing hidden numeric traits.
- **No focus traps in notebook/settings/invite dialogs** - Why: Lazy surfaces must not strand keyboard or screen-reader users.
- **Return to prior scene focus after lazy surfaces** - Why: Dialogs should preserve the user's place in the scene.
- **WCAG AA validation for user-copy surfaces** - Why: Product copy, captions, narration shown visually, account/settings/errors, and unsupported-browser surfaces must meet contrast.
- **Naturalist voice only on product surface** - Why: System surfaces remain direct and matter-of-fact.

### 9. Performance, observability, and privacy boundaries

- **Initial JavaScript bundle under 2 MB gzipped** - Why: The critical path must stay small enough to support the already-running first frame.
- **First bird visible under 500 ms on mid-tier mobile/4G** - Why: The first bird must appear quickly enough to preserve the place-in-motion conceit.
- **Idle scene at 60 fps on older laptop** - Why: The calm scene should remain smooth during ordinary watching.
- **No client memory growth over 30 minutes** - Why: Long quiet sessions should not leak resources.
- **Kilobyte snapshot payloads** - Why: Pull sync stays lightweight and edge-friendly.
- **No long-frame resync loop** - Why: Resume/resync should correct state without causing performance instability.
- **Simulation tick p99 below five-second alarm threshold** - Why: Tick latency is monitored before stale simulation becomes a product problem.
- **Synthetic browser checks from common geographies** - Why: Navigation, first-bird, audio, and accessibility are monitored across real delivery conditions.
- **Aggregate-only RUM** - Why: Page load, frame timing, audio errors, snapshot/tick latency, and session-duration histograms can be measured without account, bird, mood, personality, or history identifiers.
- **Operational logs on synthetic UUID boundary with token/email scrubbing** - Why: Logs support operations without leaking account identifiers.
- **Restricted simulation diagnostics store** - Why: Engine health can track tick duration, lease contention, backlog, replay failures, and drift fixtures without analytics reading per-bird records.
- **Service-health dashboards only** - Why: Dashboards may aggregate health but not user relationships.
- **Alerts for tick, commit, staleness, auth, audio, and performance regressions** - Why: Operational failures are caught before they undermine the aviary contract.
- **Audio resource pooling, call caps, DOM release, context bounds, heap snapshots** - Why: Memory safety is required for long sessions and seven-bird chorus cases.
- **Rendering profiles for hidden tabs, low-power mobile, seven birds, reduced motion, captions, and visibility changes** - Why: These are the stress cases most likely to break performance or accessibility.

### 10. Delivery sequence and rollout

- **Vertical slices preserving the final contract** - Why: Each slice should advance implementation without temporarily violating the product's tone and canonical-state rules.
- **Foundations and contracts first** - Why: Schemas, invariants, threat model, API errors, engine config, replay fixtures, event cursor, and tick lease must exist before UI polish.
- **Engine slice** - Why: Starter birds, hidden vectors, mood, tick, drift, age eligibility, bird effects, notebook, and snapshot serialization are the simulation core that later surfaces rely on.
- **Catch-up, retry, concurrent-device, and visitor-exclusion exercise in engine slice** - Why: These cases prove server-owned, ordered, non-visitor drift before richer UI.
- **Critical scene slice** - Why: The initial snapshot, responsive one-screen composition, day/night/weather, micro-motion, chrome fade, return-greeting, resync, and reduced-motion architecture establish the felt aviary.
- **Immediate first-bird and bundle gates in scene slice** - Why: The affective first-frame contract depends on performance from the beginning.
- **Interaction/audio slice** - Why: Presence, listen-in, offers, settle, procedural audio, captions, and silence fallback are added together so interaction cannot mutate canonical personality directly.
- **Account and notebook slice** - Why: Magic links, settings, sessions, rename, notebook, export, deletion, system errors, keyboard, narration, focus, contrast, and reduced motion complete account/accessibility obligations.
- **Quiet visits slice** - Why: Per-invite links, host projection, no visitor events, visit log, expiration, revocation, and off-by-default notifications are implemented as a narrow read-only surface.
- **Hardening slice** - Why: Browser matrix, performance profiles, memory soak, screen-reader/keyboard passes, security/privacy review, export/deletion verification, and fault injection guard release quality.
- **Launch with two birds and server-controlled age eligibility** - Why: Everyone starts from the same calm aviary and later birds arrive without progress counters or engagement prompts.
- **Operational flags for tick engine, audio, offers, notebook, and visits** - Why: Faulty optional surfaces can be disabled without changing canonical state.
- **Ramp from fixtures to beta to broader release only after gates hold** - Why: Expansion waits on first-bird, tick p99, memory, accessibility, and privacy gates.
- **No engagement prompts, push campaigns, visible metrics, or public directory during ramp** - Why: Rollout must not add gamification, social discovery, or notification loops.

### 11. Test strategy and acceptance gates

- **Property tests for drift monotonicity, clamping, low-pass behavior, no neglect change, and no single-session jump** - Why: Tests enforce slow, forgiving personality drift.
- **Replay fixed event streams across retries, delayed ticks, catch-up, and concurrent devices** - Why: Deterministic state, snapshots, and versions prove the simulation is resumable.
- **Assert only simulation service can write personality** - Why: API and database boundaries must reject client absolute-vector mutations.
- **Mood persistence and local-time/weather transition tests** - Why: Mood should persist across sessions and respond to ambient conditions.
- **Bird-to-bird effects, stable IDs after rename, age-based additions, seven-bird cap, and sparse notebook tests** - Why: Core aviary invariants and non-gamified notebook behavior are release gates.
- **Exact-once cursor, idempotent presence overlap, offer cooldown, settle/undo, and visitor-exclusion tests** - Why: Input processing must be ordered, bounded, and owner-only for drift.
- **Magic-link, session, email-change, export, deletion, CSRF, authorization, and token-leakage tests** - Why: Account and security surfaces are sensitive and must fail safely.
- **Snapshot, stale/resume, event batch, error surface, invitation, and visitor contract tests** - Why: Clients and visits need explicit behavior without hidden state merges.
- **Two-device additive-delta scenarios** - Why: Interleaved sessions should preserve both contributions without last-write-wins personality overwrites.
- **Browser tests for first frame, layout, top bar, greeting, listen-in, offers, settle, notebook, and ambient transitions** - Why: The sensory scene must preserve the no-spinner, one-screen, already-running contract.
- **WebAudio tests for success, denial, unavailable API, cleanup, caption correspondence, chorus recognition, and no recorded-audio request** - Why: The audio/caption experience must be procedural, accessible, and resource bounded.
- **Screen-reader, reduced-motion, caption, keyboard, focus, contrast, unsupported-browser, and announcement-surface tests** - Why: Accessibility and voice constraints are release-blocking.
- **Performance and heap soak gates with aggregate measurements only** - Why: Release proof should measure behavior without retaining per-account or per-bird data.
- **Final release acceptance tone** - Why: The product is complete only when it has "no stat-management UI, no guilt for absence, no notification-driven social loop, no visitor influence on the host's simulation, and no client-owned canonical state."

### 12. Risks and mitigations

- **Versioned coefficients and deterministic calibration fixtures** - Why: Drift can be adjusted if too fast or too slow without exposing numbers or adding engagement metrics.
- **Presence conjunction and bounded intervals** - Why: It mitigates background-tab inflation while allowing quiet watching.
- **Server tick as sole personality writer with append-only cursor processing** - Why: It mitigates sync corruption and lost drift.
- **Persisted last tick/cursor, p99 and staleness alerts, deterministic elapsed-time replay** - Why: Scheduler failure should not silently stale the aviary.
- **Recognizable audio signatures, runtime variation, voice budgets, listener testing, and gain ramps** - Why: Seven-bird audio should avoid repetition, uncanniness, and overwhelm.
- **Critical path kept small with inline/edge snapshot and lazy secondary surfaces** - Why: Initial load must not break the already-running conceit.
- **Narration, captions, reduced motion, keyboard, contrast, and screen-reader tests built alongside scene** - Why: Accessibility should not become a degraded afterthought.
- **Narrow per-recipient read-only invite capability** - Why: Visits should not become a social network or affect simulation.
- **Synthetic UUIDs, analytics isolation, log scrubbing, export/deletion/telemetry payload tests** - Why: Convenience identifiers and telemetry must not leak private or per-bird state.
- **Review every new surface against voice and non-goal rules** - Why: The product should reject welcome banners, streaks, visit counts, progress counters, achievements, and engagement nudges.
