## System-level intent

- Treat Pocket Aviary as "an affective systems product, not a game, dashboard, pet simulator, or social app." This governs the Planning stance, the V1 exclusions, the interaction model, the copy rules, and the Definition of Done.
- Make the aviary feel "like a place that continues without the user." This appears in the Planning stance through birds "already mid-action on first paint," server-side canonical state that advances whether clients are connected, persistent mood, and a return experience based on state that continued while the user was away.
- Let the product notice the user through birds, not UI announcements. The Planning stance says the product "notices the user through bird behavior rather than announcing through UI"; Return-greeting repeats that it is "a generated bird behavior, not a UI message"; Launch gates ban a "welcome toast/banner"; the Definition of Done says UI cannot announce what birds are meant to notice.
- Keep simulation truth on the server and make "dangerous conflicts impossible." The architecture says the simulation worker is the only component allowed to write personality vectors, clients write events but "never personality, mood, or notebook rows," and Sync Model says the important rule is not to resolve conflicts well but to prevent them.
- Preserve relationship without obligation or punishment. The plan repeatedly blocks "scores, levels, streaks," "caretaker obligations," hunger/death/distress, and negative trait decay. Drift is "monotonic toward expressive," absence does not move stored personality downward, and age-gated birds are "not as a reward."
- Keep privacy boundaries structural, not cosmetic. The plan uses synthetic UUIDs, encrypted email fields, keyed lookup HMACs, aggregate-only operational metrics, synthetic seeded accounts for calibration, and explicitly disallows per-bird values, per-account interaction histories, and analytics dimensions using email or visitor email.
- Split product voice by surface: "naturalist voice" for product-facing bird/notebook/narration/caption surfaces and "matter-of-fact voice" for auth, sync, account, settings, deletion/export, accessibility, unsupported-browser, and revoked/expired visit surfaces. The plan says this split must be encoded into component ownership and copy review.
- Make accessibility part of the same product, not a fallback. Reduced motion is a "designed alternate scene runtime," narration comes "from the same state as the visual scene," captions are generated from actual call parameters, and launch is blocked if narration, captions, reduced motion, or keyboard navigation are incomplete.
- Treat performance as part of the "already alive" conceit. First-bird timing, no spinner, small critical runtime, no DOM work in animation frames, WebAudio reuse, memory stability, and snapshot size all exist to avoid a loading state that would break the sense of a living place.
- Keep visits quiet, explicit, and read-only so social features do not become a network. Visit invitations are per-email, visitor sessions have no events endpoint, visitors cannot trigger greetings or drift, visit logs live only in settings, and the risk section rejects profiles, discovery, comments, co-presence, and public data structures.

## Per-feature whys

### Planning stance and V1 scope

- TypeScript web stack with a thin React-style UI shell and imperative scene/audio runtime: chosen to keep account/settings/notebook chrome separate from "the aviary itself," where scene and audio need runtime control rather than app chrome.
- PostgreSQL canonical store with append-only interaction events and a queue-backed simulation worker: supports a primary canonical store, ordered event facts, and once-per-minute server ticks so the server can advance state even when clients are disconnected.
- Layered 2D render pipeline: separates canvas/WebGL aviary rendering from DOM chrome so birds and scene motion can protect the 60fps and memory budgets.
- Deterministic procedural generation from curated grammars: provides calls, field-notebook entries, captions, narration, weather, and small variations without model-training, recommendation systems, or aggregate behavioral analysis of per-bird history.
- Browser-only Pocket Aviary: NOT RECOVERABLE FROM PLAN
- One canonical aviary per single-user account: supports "one row per account in V1," every device reading the same canonical snapshots, and no client-to-client sync or personality merge.
- Email magic-link sign-in: NOT RECOVERABLE FROM PLAN
- Revocable per-device sessions: lets account settings revoke device sessions and lets the server reject events from revoked sessions.
- Email change verification: the new email "must verify before commit" while the old email remains active, protecting verified account identity during change.
- JSON account export: gives a queued account export with encrypted download reference and email-only delivery as part of account control and privacy-safe data access.
- Thirty-day soft deletion followed by hard deletion: gives a deletion lifecycle with cancel support before the hard-delete window completes.
- Two starter birds per new account: NOT RECOVERABLE FROM PLAN
- User-assigned names: NOT RECOVERABLE FROM PLAN
- Stable bird identities: bird ids survive rename, sync, migration, and species-library changes so identity is not lost when names or implementation details change.
- Coherent species pool: NOT RECOVERABLE FROM PLAN
- Age-gated path toward seven birds: the plan says availability is by account age, "not engagement," should not show progress bars or "earn more birds" framing, and must wait until engine/audio budgets hold.
- Excluding scores, levels, streaks, achievements, badges, counters, XP, ranks, tiers, and equivalents: prevents gamification surfaces from replacing the affective, non-game product.
- Excluding hunger, death, distress, negative trait decay, caretaker obligations, happiness meters, and feeding schedules: prevents obligation, punishment, or absence-driven harm from entering the relationship.
- Excluding shared aviaries, public profiles, follows, comments, chat, discovery feeds, leaderboards, public directories, co-presence, avatars, and friend-of-friend sharing: keeps V1 from becoming public social pressure or a network.
- Excluding user-controlled bird placement, scene customization, and multiple aviaries per account: NOT RECOVERABLE FROM PLAN
- Excluding payments and billing: NOT RECOVERABLE FROM PLAN
- Excluding push notifications and marketing-style engagement emails: protects the quiet product posture and avoids notification-style announcements or engagement pressure.
- Excluding personality-vector numbers from every user surface: keeps hidden scalar values from becoming numeric trait exposure or debug-like relationship stats.

### System Architecture

- Web client: serves the shell, renders the aviary, runs WebAudio, records presence, submits interaction events, and presents settings/notebook/visit/account surfaces because those are client-owned smooth rendering and surface responsibilities.
- Auth/account service: owns magic links, sessions, email changes, exports, deletion lifecycle, and visitor invitations because these are account identity and authorization boundaries.
- Aviary state API: returns bootstrap and snapshot payloads scoped by account UUID or visit token so owner and visitor reads share canonical snapshots with correct authorization.
- Interaction event API: accepts idempotent append-only owner events because clients submit facts, not absolute bird state; visitor clients never receive this write path.
- Simulation worker: advances canonical state, consumes events, updates mood/personality/notebook/weather, and emits snapshot versions because it is the server-side authority for simulation truth.
- Export/deletion worker: separates export generation and hard deletion after the 30-day soft-delete window from interactive app requests.
- Metrics pipeline: receives only aggregate operational metrics and synthetic performance results so telemetry never becomes per-bird or per-account relationship history.
- Simulation worker as the only writer of personality vectors: prevents client or race-driven corruption of personality continuity.
- Client/server split: the server owns identity, sessions, birds, mood, personality, snapshots, cooldowns, privacy, and canonical prose decisions; the client owns interpolation, pose blending, WebAudio, captions, presence detection, and ephemeral listen-in treatment so smooth experience does not become simulation truth.
- Single scene runtime consuming `AviarySnapshot`: gives one typed boundary that can produce visual frame state, audio scheduling inputs, and accessibility state from the same canonical source.

### Data Model

- Synthetic account UUIDs: the plan says the synthetic UUID is the primary identifier used everywhere except encrypted email and is the only identifier in logs, telemetry dimensions, partitions, queues, and relationships.
- Encrypted email plus lookup HMAC: lets magic-link routing and rate limiting work while email appears only in encrypted fields and controlled email-delivery jobs.
- Session token hashes: raw tokens are never stored, supporting revocable per-device sessions without storing secrets.
- One `aviaries` row per account: enforces the V1 single-aviary shape at the database layer.
- `species` reference data with no rarity field: provides silhouettes, palettes, call motifs, pose sets, and safe range modifiers without introducing rarity.
- `birds` stable UUIDs, seeds, hidden vectors, mood, perch zone, and cooldown state: preserves identity, deterministic variation, server-hidden personality, fast mood state, responsive layout derivation, and offer cooldown enforcement.
- `interaction_events` append-only log: gives ordered, idempotent event facts with session/device context that the simulation worker can consume once.
- `presence_windows` as derived records: keeps presence as simulation input in the simulation database, "not analytics facts."
- `aviary_snapshots` as compact canonical render snapshots: stores snapshot version, bird intents, mood, call hints, weather, day/night phase, and return-greeting data without storing generated per-frame motion.
- `simulation_ticks`: records operational timing and replay references, "not user-facing prose."
- `field_notebook_entries`: keep entries read-only and sparse, generated from curated observation patterns rather than generic event logs.
- `visit_invitations`: explicit per-email invites with one-time token flow prevent permanent public URLs.
- `visit_sessions`: provide read-only authorization context with no presence events, interaction events, or drift inputs.
- `visit_log_entries`: remain reachable only from account settings and have "No badge or push surface."
- Monotonic expressive personality drift: absence never decays stored personality downward, preserving the no-punishment relationship model.
- Bird count cap of seven enforced in validation, adoption eligibility, UI, and simulation assumptions: keeps data, interface, and engine assumptions aligned.

### API Surface and Interactions

- Generic magic-link request response with keyed lookup and IP reputation rate limits: protects the auth surface while keeping system copy intentionally generic.
- Magic-link consumption invalidates tokens on success and uses matter-of-fact errors for expired, replayed, or invalid links: protects session creation and avoids charming error tone.
- `GET /account`: exposes email display, sessions, settings, deletion status, export status, and invite-notification preference from the account/settings surface. Rationale beyond account visibility: NOT RECOVERABLE FROM PLAN
- Account settings updates for audio, captions, reduced motion, visit notification, and accessibility: lets user preferences and accessibility settings control the surfaces that need them.
- Export endpoint queues JSON export and emails a download link to the verified address: keeps export asynchronous and tied to verified email delivery.
- Delete/cancel endpoints start or cancel the 30-day soft-delete window: allows account deletion without immediate irreversible hard deletion.
- Session deletion endpoint revokes a device session: supports per-device session control.
- Aviary bootstrap includes a compact current snapshot and can be edge/cache assisted: supports first bird rendering within 500ms; if unavailable, the client draws "the quiet field, not a spinner."
- Snapshot endpoint with `since_version`: supports visibility return, long frame gaps, low-frequency visible keepalive, post-resume, and after interaction batches while returning no-change when possible.
- Notebook endpoint is paginated and read-only: preserves the field notebook as observation, not an editable feed.
- Adoption eligibility endpoint: returns availability "not as a reward" and must not render as gamified progress.
- Adoption endpoint: server selects species, user supplies or confirms name, and cap is enforced so adoption stays age-gated and bounded.
- Bird rename endpoint: rename only, with no effect on identity, personality, mood, or call signature.
- Batched `/aviary/events`: uses idempotency keys, validates owner session, event schema, offer cooldowns, and listen-in/session constraints so client interactions remain facts for later simulation, not direct state mutations.
- `session_open`: supports return-greeting computation.
- `visibility_return`: computes absence-tier greeting when a hidden tab becomes visible.
- `presence_ping`: counts only visible document, focused window, and recent pointer/key activity so presence remains honest.
- `presence_end`: closes presence when any condition fails or settle/close occurs where possible.
- `listen_in_start` and `listen_in_end`: record bird-specific attention for drift while allowing the client to start local mix ramps immediately.
- `offer_*` events: carry offer type and local context while the server decides canonical reaction.
- `settle` and `settle_undo`: settle quiets mood and ends presence; undo within five seconds records correction and cancels local settled rendering.
- Visit invitation creation/list/revoke/consume endpoints: keep visits explicit, expiring, revocable, and per-email.
- Visitor snapshot endpoint with no events endpoint: keeps visitors read-only; revoked or expired visits return a matter-of-fact "visit no longer available" surface.

### Simulation Engine Design

- Once-per-minute server-side simulation tick: advances active aviaries even when no clients are connected, while allowing coarser scheduling for long-inactive accounts if local-time mood continuity remains correct.
- Tick transaction: loads state, consumes events, computes drift and mood, chooses perch intents, advances weather/chorus, maybe generates notebook entries, and writes deltas/snapshot/bookkeeping together so personality is not partially applied.
- Idempotent tick retry from the same event range: prevents partial personality changes without consumed-event bookkeeping.
- Presence processing: server accepts only coherent presence windows and rejects impossible durations, clock skew, duplicates, revoked sessions, hidden flags, or unfocused flags because "tab-open, audio-playing, or page-loaded" must not count as presence.
- Slow low-pass drift function: uses daily and weekly accumulators so regular visits become measurable after about one week and visible after about three weeks, while no single session visibly moves a trait.
- Per-bird offer cooldowns: prevent repeated offers in one session from saturating curiosity.
- Settle as mood quieting, not trait growth: closes presence and quiets current mood without becoming a "trait-growth hack."
- Persisted fast-timescale mood state machine: mood is "not a tab-open default" and continues transitioning while the user is away.
- Deterministic seeded randomness per tick/bird: allows variation without becoming unreplayable.
- Return-greeting plan with primary bird, optional secondary responses, session/opening id, and cooldown: gives bird-based welcome behavior while avoiding refresh-loop greetings.
- Call grammar runtime with species motifs plus bird seed: makes calls recognizable per species and individual bird while personality and mood modulate timing, pitch, envelope, and probability.
- Captions generated from the same scheduled call parameters: keeps text matched to the actual sound played.
- Field notebook generation from specific observation patterns: creates sparse naturalist observations and avoids raw event logs, visit streaks, or "you returned after X days."

### Sync Model

- Server as sole source of truth: multi-device sync works by every device reading canonical snapshots and writing events, with no client-to-client sync or personality merge algorithm.
- Event idempotency and ordering: dedupes retries, rejects revoked sessions, bounds clock skew, orders by server receipt and stable tie-breakers, and records last consumed event per tick.
- Listen-in duration handling: uses start/end pairs and server timestamps where possible; missing ends close at session end, visibility loss, heartbeat timeout, or next tick timeout.
- Multi-device behavior: both devices can submit events, but offers/rename/adoption become canonical only through snapshots, and listen-in stays local audio focus plus a server attention event.
- Brief offline behavior: the client may render the last snapshot and queue a very small bounded event set, but must never simulate catch-up drift locally; stale uncertain events are dropped.
- Visitor sync: visitors receive read-only host snapshots, cannot trigger greetings/listen-in/offers/settle/notebook/presence/drift, and revocation takes effect on the next pull.

### Frontend Rendering Pipeline

- One responsive horizontal scene: keeps the aviary in a single viewport without panning, scrolling, or zooming while preserving birds and proximity cues across phone and desktop.
- Quiet field loading state: if the actual aviary is not ready, a soft field replaces spinners, progress bars, banners, or entry animations to preserve the living-place conceit.
- Bird pose states: represent idle, transition, offer reaction, greeting, and chorus/call posture so behavior carries product meaning.
- Mood through pose distribution: avoids labels; wary birds sit back and scan, content birds preen, curious birds tilt, and drowsy birds settle low.
- Client interpolation and micro-motion: creates continuous smooth motion from snapshot intents without changing canonical perch, mood, or call state.
- Reduced-motion renderer: replaces micro-motion and flight paths with cross-fades, removes ambient drift, keeps calls/captions/notebook/mood/drift, and shares snapshot semantics so it is not a separate simplified product.
- Sparse top bar with fade: keeps account/settings/accessibility/notebook/offer/settle controls above the scene and nearly transparent after stillness so chrome does not live inside the aviary.
- No hover labels, mood badges, numeric stats, counters, or bird buttons: prevents birds from becoming app-like status chips.

### Audio Pipeline

- WebAudio procedural graph: creates oscillator/noise/filter/envelope graphs from motif parameters so calls are procedural rather than recorded loops.
- Listen-in gain ramps: focused bird rises gradually while other birds drop to ambient but never silence, preserving the aviary rather than isolating one bird completely.
- Chorus scheduler: allows overlapping procedural calls without loop phase artifacts.
- Memory-conscious audio object reuse: supports the 30-minute no-growth budget.
- Autoplay fallback: initializes visuals immediately, resumes sound on eligible gesture, and if WebAudio is blocked or unavailable runs in graceful silence with captions on.
- No recorded-audio fallback: the plan says canned recorded loops are worse than silence with captions for this product.
- Captions near calling bird: support audio-off users, hearing differences, noisy environments, blocked WebAudio, and caption preference while avoiding motif ids, pitch numbers, or engine terminology.

### Accessibility Surfaces

- Screen-reader narration stream: uses the same state as the visual scene, slow cadence, priority bumps, polite live region, and curated naturalist prose so it describes the aviary "as a place, not a list of states."
- Keyboard model: gives top-bar and scene access, bird-to-bird focus, listen-in with Enter, Escape behavior, keyboard navigable offer/settings, and reachable settle.
- Gentle focus treatment: must pass contrast in day and night while reading as an outline or glow, "not a badge."
- WCAG AA contrast and copy split: keeps all text accessible and keeps naturalist voice for product surfaces and matter-of-fact voice for system surfaces.
- Copy ownership and review: prevents the naturalist/matter-of-fact split from being left to individual implementation judgment.

### Performance, Observability, Rollout, Tests, Risks, and Done

- Initial bundle, first-bird, fps, memory, snapshot, and tick budgets: keep the product within the performance envelope needed for "already alive."
- Code-splitting noncritical flows and preloading the critical render runtime: protects first render while account/settings/visit/export/delete/older notebook code loads separately.
- Compact procedural bird assets and motif definitions: reduce bundle/payload size and avoid recorded audio.
- Avoiding DOM work in animation frames: protects idle motion performance.
- Allowed aggregate metrics: support operations, performance, auth delivery, export/delete jobs, and synthetic checks without relationship telemetry.
- Disallowed telemetry: blocks per-bird personality values, per-account histories, population-level drift analysis from real user birds, leaderboard-ready counts, and email dimensions.
- Synthetic seeded accounts for calibration dashboards: allow engine behavior inspection while keeping real user simulation data separate.
- Build phases from foundations through hardening: sequence identity/privacy, simulation, scene, audio, interactions, prose, accessibility, visits, and hardening so core invariants land before launch.
- Bird-count ramp: tests seven-bird load internally, starts production with two birds, then enables more by account age only after budgets hold.
- Launch gates: block V1 until performance, memory, tick latency, drift calibration, accessibility, privacy, and copy constraints are met.
- Engine tests: prove clients cannot write personality/mood, replay is deterministic, drift calibrates, absence does not reduce traits, presence is honest, offers do not saturate, mood persists, and identity survives rename/sync/adoption/migration.
- Sync and API tests: verify idempotent event retries, multi-device overlap without last-write-wins personality loss, revoked sessions, magic-link replay/expiration, visitor read-only revocation, deletion, and export correctness.
- Frontend and accessibility tests: assert no spinner, responsive fit, top-bar fade/keyboard restore, keyboard listen-in, reduced-motion cross-fade, audio memory reuse, captions from actual call parameters, copy snapshots, narration cadence, contrast, focus visibility, reduced-motion behavior, and WebAudio unavailable captions.
- Privacy and observability tests: scan for email, visitor email, bird names, personality vectors, per-account event payload leakage, analytics allowlist compliance, synthetic dashboard guardrails, and deletion coverage.
- Drift calibration risk mitigation: use early synthetic calibration, capped low-pass deltas, design/product review of one-week and three-week histories, and server-side coefficients with audit trails.
- Sync bug risk mitigation: enforce server-only personality writes, append-only transactional events, replay tests, immutable bird ids, and impossible vector reset alerts.
- Audio repetitiveness risk mitigation: prototype call grammar early, test recognizability from two through seven birds, seed individual variation, avoid recorded fallback, and align captions to exact parameters.
- Accessibility fallback risk mitigation: ship accessibility in main phases, use naturalist templates, review reduced-motion aesthetics, and block launch if incomplete.
- Performance risk mitigation: keep critical runtime small, include snapshot in bootstrap, draw quiet field, code-split noncritical flows, and instrument first-bird timing from day one.
- Privacy erosion risk mitigation: separate simulation storage from analytics, use metric schema allowlists, synthetic accounts for dashboards, encrypted email fields, UUID logs, and privacy review for every new metric.
- Social expansion risk mitigation: keep visits read-only by API design, no visitor event-write path, notifications off by default, settings-only visit log, and reject public/discovery structures in V1 schema.
- Product voice risk mitigation: establish copy ownership by surface type, snapshot-test strings, encode banned terms/patterns, and make the bird greeting the only welcome surface.
- V1 user Definition of Done: new user can sign in, meet two named birds, return to continued server-side scene, listen in, offer, settle, read notebook observations, use accessibility modes, invite read-only visitor, export/delete account, and avoid gamification, obligation, public social pressure, numeric traits, and notification announcements.
- Engineering Definition of Done: architecture protects the product's absences by ensuring clients cannot own personality, visitors cannot affect drift, absence cannot punish birds, telemetry cannot become behavioral analysis, and UI cannot announce what birds are meant to notice.
