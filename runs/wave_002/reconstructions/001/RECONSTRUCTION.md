## System-level intent

- Private, account-bound, revocable access is a core product principle. This shows up in "one browser-based, private aviary per email magic-link account," "quiet, revocable, read-only visit invitations," the edge response that is "never a publicly cacheable account response," and visitor credentials that can never reach owner event endpoints.

- The aviary is meant to be a continuing canonical world, not a client-side illusion. The plan repeats this through "database is the sole source," "simulation worker is the only writer," per-aviary `state_version`, immutable `tick_id`, stable bird IDs, deterministic seeds, and an acceptance demonstration where the "first visible frame is the continuing aviary."

- The product voice is quiet, naturalist, and non-gamified. Section 1 requires "lowercase, present-tense, specific naturalist observations," while excluding "status badge," "metric panel," "ranks," "achievements," "streaks," and "scores." The notebook and narration are also bounded to bird/place observations rather than stats or user-frequency statements.

- Relationship growth must never punish absence. The plan says "absence cannot harm or reset birds," vector deltas are "nonnegative," "settle adds no drift," and absence may change greeting or ambient expression "without lowering traits or showing distress."

- Attention is qualified without becoming surveillance. Presence is based on visible, focused, recent pointer/key activity, and the server union of leases is called "an honesty mechanism, not antifraud surveillance." The plan explicitly stores no pointer coordinates or keystrokes and rejects open-tab, scroll, WebAudio, visitor, or offline backfill claims.

- Accessibility is part of the same experience rather than a separate fallback. The plan ships "naturalist narration, reduced motion, and call captions," keeps "the same state, calls, notebook, moods, and drift" under reduced motion, and treats the no-sound first visit as "a complete product experience."

- The first frame should feel ongoing, calm, and resilient. This appears in the private edge snapshot, "critical first-bird path," animation phase anchors, "no spinner," "no bird reset," and the fallback to a "soft quiet field" or "plain retry control outside the scene" when state genuinely lags or fails.

- Operational data collection must remain privacy-minimal. The plan allows only "aggregate operational counters/histograms," excludes account/bird/event dimensions and stable analytics IDs, keeps the simulation database and private events out of telemetry/model-training pipelines, and forbids instrumentation of "per-account relationship data."

- Rollout and verification are governed by invariants, not feature count. The plan says to "ship in gates," "gate each stage on invariants," use synthetic scenarios and opt-in research, and release-block on deterministic fixtures, property tests, browser tests, audio checks, copy reviews, performance tests, and real assistive-technology sessions.

## Per-feature whys

### 1. Product contract and decisions

- Browser-based, private aviary per email magic-link account: NOT RECOVERABLE FROM PLAN

- Two system-selected starting birds from a coherent pool of roughly six species: NOT RECOVERABLE FROM PLAN

- Owner naming and later renaming: renames are meant to preserve identity and continuity; the plan says renames "never replace ID or reset state" and cause "No behavior changes on rename."

- Growth to seven birds through age-based adoption opportunities: the plan ties growth to aviary age alone, never "session, visit, or offer totals," and keeps the seven-bird cap as a hard product boundary.

- Single responsive horizontal scene with three perch zones: the scene should stay "within one viewport with no pan/zoom/scene scrolling" while keeping two to seven birds visible across narrow phones and wide screens.

- Procedural calls: calls use grammar descriptors, motif IDs, seeds, and per-bird signatures so birds stay recognizable without "recorded call loops" or a "repeated exact loop feel."

- Mood-shaped motion: mood is kept as a separate finite state with dwell times and hysteresis so expression changes without resetting traits or flapping every minute.

- Day/night light using the owner's authoritative IANA time zone: the plan wants a canonical day/night cycle owned by the account, initialized from the browser, protected by 30-minute hysteresis, and never set by visitors.

- Rare weather: rain and wind are intentionally "sparse deterministic schedules" with only short-lived mood and call effects, matching the plan's quiet scene rather than constant spectacle.

- Listen-in: listen-in focuses one bird by ramping it up and others down while keeping an ambient floor, and its bounded intervals can later influence drift without making local mix focus canonical state.

- Three offers as gestures, not feeding: accepted offers add small server-authored contributions such as curiosity or boldness while avoiding feeding, punishment, or a visible timer economy.

- Settle: settle provides an evening lighting and call-quieting transition, ends the current presence interval like closing the tab, and has a five-second undo path.

- Sparse read-only field notebook: notebook entries are rare, immutable, deduplicated, and written as naturalist observations, giving memory without a stats UI or archive cutoff.

- Quiet, revocable, read-only visit invitations: a visitor watches the same bird scene, mood, weather, and calls without the host notebook or interaction controls, so visiting cannot change the aviary.

- Account export and deletion: export is the only user-facing route that can disclose numeric vectors, while deletion removes account-linked state, events, notebook, sessions, invitations, visits, exports, and operational records after the recovery period.

- Multi-device access: devices pull the same canonical projection and the server merges overlapping qualified presence intervals so two devices do not double-credit attention.

- Naturalist narration, reduced motion, and call captions: these make the scene meaningful with screen reader active, motion reduced, or sound unavailable while preserving the same canonical state.

- Lowercase present-tense naturalist observations and plain system language for identity, errors, sync, and settings: bird prose belongs to bird experience, while errors and account surfaces stay direct and system-like.

- No return toast, absence counter, status badge, metric panel, or visit prompt in the aviary: the plan says "Bird greetings carry the welcome," and the surface avoids metric panels, guilt, and in-scene prompts.

- No personality numbers in ordinary UI, including accessibility labels: trait numbers are omitted from snapshots and ordinary UI so users receive expression and export rights without a stats interface.

- No native client, second aviary, customizable scene, public discovery, shared simulation, chat, profiles, payments, ranks, achievements, streaks, scores, aviary notifications, or recorded-call fallback: NOT RECOVERABLE FROM PLAN

- Optional visit email notification, off by default: this is the sole notification exception, only sent when explicitly enabled, rate-limited through the outbox, and never represented by an in-aviary badge.

- Four-minute activity window, 60-second tick, and few-minute per-bird offer cooldown: the plan says these operational choices should be tuned with synthetic and consented test accounts, not population analysis of private interaction histories.

- Settle reachable through a top-bar menu: the plan gives the reason directly: "so the four-icon top bar remains sparse."

- Visitor scene matching the host scene while excluding host notebook and controls: the rationale is read-only visitation with the same bird world but no host private surfaces or interaction affordances.

### 2. System boundaries

- Small authenticated web client, HTTP/API application, transactional database, scheduled simulation worker, transactional outbox/email worker, and private object store: the split supports canonical simulation truth, transactional email, and short-lived exports without making the client the authority.

- API and simulation deployed close to the canonical database: NOT RECOVERABLE FROM PLAN

- Private edge handler serving minimal current scene projection and critical CSS/SVG: this supports a fast, private first scene while remaining "never a publicly cacheable account response."

- Lazy-loaded notebook, account, invitation, and settings code after the first scene: secondary surfaces should not block the initial aviary scene.

- Public asset CDN limited to species silhouettes, compact decorative assets, and versioned client code: only non-account assets are public, while account scene responses stay private.

- Database as sole source and simulation worker as only writer of mood, position, call schedule, weather, notebook observations, and personality vectors: this prevents API handlers and clients from setting canonical traits or mood.

- Client composition, interpolation, audio synthesis, and presence evidence without canonical ticks: the client can render and report evidence but cannot author simulation history.

- Client-only leaves, feathers, parallax, and focus/mix state with no persistent effect: ambient ornament and local listening state are presentation, not durable simulation.

- Separately versioned protocol schemas for canonical state, public scene projection, event payloads, and narration: versioning and migrations preserve stable bird IDs and stored vectors.

- Per-aviary `state_version`, immutable `tick_id`, and small projections that omit vectors, raw presence, private event history, and visitor addresses: these support sync and replay while keeping private internals out of normal scene payloads.

- Separate authenticated export endpoint for numeric vectors without adding a stats UI: export satisfies account export while preserving the ordinary UI's no-metrics principle.

- Visitor projection using the same scene fields but excluding host notebook and settings: the visitor receives the shared read-only scene, not host-private controls or records.

### 3. Persistent model and retention

- `Account` with synthetic UUID, encrypted verified email, one-to-one aviary, and no email-derived keys, partitions, telemetry tags, or log identifiers: this keeps identity in a restricted vault and prevents email from becoming a system-wide identifier.

- `Session` with opaque hashed token, secure cookie attributes, and device revocation: this supports multi-device access with account-level revocation.

- `Aviary` carrying canonical zone, tick clock, state version, weather/light, settle transition, adoption opportunity, and last owner presence end: this centralizes the simulation clock and world descriptors.

- `Bird` with immutable UUID, persisted normalized vector, motif identity, call seed, and offer cooldown: this preserves stable identity and accumulated state across rename, mood, and trait drift.

- `InteractionEvent` as idempotent, append-only, briefly retained operational replay data that never feeds analytics: retries and recovery are supported without turning private interaction history into analytics.

- `PresenceLease` per owner session only: leases let the tick compute the union of qualified owner intervals, while visitor sessions have no presence record and cannot affect birds.

- `NotebookEntry` as immutable, cursor-paginated, and without archive cutoff: the notebook stays a durable, read-only naturalist record.

- `Invitation`, `VisitSession`, and `VisitLog` with encrypted recipient address, hashed one-use token, revocation, expiry, and approximate visit duration: visits are scoped, revocable, and visible to the host without widening simulation access.

- `Outbox` for magic links, invitations, optional visit emails, and export links with redacted tokens and addresses: email effects are transactional and general logs avoid sensitive data.

- Encrypted PII, hashed magic and invitation tokens, expiring export links, and 30-day recoverable deletion followed by purge: retention is limited to retry, recovery, and account deletion needs.

- Reauthenticated export and deletion: sensitive account actions require the owner to reauthenticate.

### 4. API and authorization

- JSON over HTTPS, typed schemas, CSRF protection, rate limits, and owner/visit scopes on every request: the API contract is explicit and authorization is checked for each operation.

- Magic-link request and consume routes with 15-minute expiry, one-time consumption, limits, generic request response, and direct replay/expiry errors: the flow avoids reusable links and limits information leakage while giving clear system errors after consumption failure.

- Owner snapshot route with `ETag=state_version`, server time, interpolation anchors, and no trait numbers: this supports efficient canonical sync and keeps numerical traits out of ordinary UI.

- Owner event route with bounded batches, idempotency UUIDs, ownership/order/payload/cooldown/skew validation, and no absolute personality or mood writes: events can be retried safely but cannot overwrite canonical state.

- Notebook route with stable newest-first pagination of immutable observations: the read-only notebook can be browsed without mutating entries.

- Settings, sessions, email change, export, deletion, and recovery routes with expected revisions and matter-of-fact 409 conflicts: account changes avoid client last-write-wins and surface current server values.

- Adoption and rename routes using server-issued age opportunity, seven-bird cap, and stable bird ID: adoption growth is enforced transactionally, and naming never changes behavior.

- Host-only invitation creation, listing, revocation, and quiet visit log with 30-day unused expiry: the host controls visit access and history without public discovery.

- Visit redeem and visit aviary routes with one-use emailed token, scoped visit session, grant checks, `410 visit no longer available`, and no owner event endpoints: mailbox access creates only a revocable read-only grant.

- Active visit grant with visible pulls about every 15 seconds: repeated pulls bound revocation latency to the next pull.

- Same matter-of-fact unavailable surface for revoked, expired, or consumed links: visitors do not learn which unavailable case occurred.

### 5. Presence, event ordering, and simulation

- Owner presence state machine requiring visible document, focus, and genuine pointer/key activity within four minutes: the plan qualifies actual attention and rejects open tabs, playback, scrolling, visitor activity, and offline backfill.

- Server presence leases with receive-time clamping, 45-second expiry, overlapping-device interval union, and no pointer coordinates or keystrokes: the plan calls this an "honesty mechanism, not antifraud surveillance."

- Offer cooldown validated server-side with UI prevention and no punitive timer display: repeated gestures are bounded by server acceptance while the interface stays non-punitive.

- Scheduler queuing each aviary's due minute even with no connected clients and committing exactly one logical tick at a time: the world continues server-side and tick retries are no-ops after commit.

- Bounded sequential catch-up, deterministic seeds, row locking or compare-and-swap, checksums, and backup/restore drills: outage recovery preserves elapsed day/night and weather and prevents duplicate or hidden-reset ticks.

- Drift as a slow low-pass signal with additive nonnegative capped deltas: the plan rejects click counters, negative absence drift, and single-session visible trait jumps.

- Presence, listen-in, accepted offers, and proximity as differently weighted drift inputs: owner attention and gestures have modest expressive effects, with listen-in focused on warmth/vocal frequency and offers/proximity on curiosity/boldness.

- Synthetic and consented scenario tuning with no production population drift dashboards: calibration happens without analyzing private production histories.

- Mood as persisted finite state with dwell times, stochastic transitions, probability tables, and hysteresis: mood changes with time, interactions, weather, neighboring calls, and vectors without flapping or resetting on page open.

- Absence producing quieter greeting or ambient expression without distress: recent-presence context can shape expression while traits never decline.

- Nocturnal species active at night: NOT RECOVERABLE FROM PLAN

- Perch selection weighted by boldness and mood with non-overlapping responsive slots: perch placement expresses state while keeping birds visible.

- Rain and wind as sparse deterministic schedules: weather stays rare, mild, and tied to short-lived mood/call effects.

- Server-scheduled call descriptors stored for current/upcoming calls: two devices consume one world timeline, and the client can synthesize audio and captions from the same descriptor.

- Bird-to-bird responses and bounded chorus density: varied response delays and voice limits preserve recognizability through seven birds.

- Song-fragment, seed, and still-pool offers represented as state transitions rather than canned clips: reactions follow mood and personality instead of playing fixed media.

- Greeting assembled from current canonical state and owner absence duration, with owner-only trigger and fresh state pull: arrival feels like a response to the continuing aviary rather than an entry animation that wakes it.

- Notebook generation from state-level observations with rarity threshold, deduplication, multi-day cooldown, reviewed grammar/templates, and voice lint: entries stay sparse, naturalist, bounded, and avoid visit-frequency or trait statements.

- Adoption opportunities based on aviary age alone, one available bird at a time, no badges or urgency, and no catalog or rarity draw: additions are calm age milestones rather than gamified or behavior-triggered rewards.

### 6. Sync and failure behavior

- Owner devices pulling canonical projection on initial load, visibility return, long suspend gaps, and about every 30 seconds while visible: devices regularly rejoin the same server timeline.

- Keeping the last rendered frame during transient fetch and reconciling over bounded transitions: network fetches should not break the visual continuity of the aviary.

- Hidden tab stopping rendering and audio, then fetching before restarting phases on resume: this saves work while ensuring resumed visuals and calls come from current descriptors.

- Pending network events with restrained local feedback, idempotent retry, and server reconciliation: the UI must not claim an offer changed a bird until the server accepts it.

- Auth failures, outages, settings/name conflicts, and state outages using direct system copy, sign-in or retry paths, current server values, and no bird reset: failures stay plain and do not imply the aviary reset.

- No personality UI conflict because clients cannot write personality: server-only vectors remove that conflict surface.

### 7. Client rendering and audio

- Initial authenticated snapshot embedded with HTML and first bird drawn from minimal SVG/critical CSS before hydration: the first frame should look ongoing without spinner, static fade, or general entrance animation.

- Soft quiet field on genuine cold snapshot lag and soft fly-in only for the unique post-adoption empty field: normal visits should never show an empty aviary.

- One-viewport scene with responsive normalized perch anchors, minimum separation, safe margins, and scale/spacing adjustments: two to seven birds remain visible on narrow phones and wide screens.

- Local parallax from leaves and feathers as non-tick records: gentle depth is allowed as rendering detail without becoming simulation state.

- Stable keyed bird nodes, one `requestAnimationFrame` loop, seeded phase offsets, species-specific silhouettes, and mood-weighted schedules: motion should be varied and stable rather than one obvious global cycle.

- Hidden-loop stop and timer/listener cleanup on unmount: this protects performance and retained memory.

- Continuous day/night blends, settle overlay with five-second any-scene-click undo, no control or badge in the scene, sparse top bar, and fade toward transparency after pointer stillness: the scene stays chrome-free while controls remain readable and focusable.

- Reduced motion from `prefers-reduced-motion` or explicit account setting, using still-pose cross-fades, no drifting leaves/parallax, slower color transitions, and the same canonical state: accessibility changes motion presentation, not simulation behavior.

- WebAudio synth voices driven by call grammar descriptors with stable spectral/timing signatures, bounded dynamics, and chorus headroom: bird voices remain recognizable without recorded loops or clipping.

- Listen-in engagement by click/tap, Enter, arrow navigation, repeat click, empty space, Escape, or focus movement, with local mix focus outside canonical state and bounded intervals reported as owner events: listening is accessible local focus while still available for later drift.

- Call captions generated from exact emitted grammar parameters and stacked to avoid overlap: captions match what the audio system actually emits.

- Autoplay handling that constructs the current schedule immediately, starts sound only after user activation, avoids fake loops or arrival modals, and falls back to silence with captions: audio constraints should not break continuity or the no-sound product experience.

### 8. Accessibility and copy acceptance

- Semantic scene region with static orientation and separate throttled live narration queue consuming the same descriptors as the renderer: screen reader output describes the same world rather than a parallel state dump.

- Reviewed naturalist narration with bird name/species, perch/depth, call or weather, and temporal detail, deduplicated at idle cadence with polite preemption: narration stays sparse, timely, and bounded.

- Notebook as ordinary read-only content in the same voice, without trait values, raw mood enums, or "perch 2" labels: accessible text uses naturalist language without exposing internals.

- Tab order through top-bar controls and scene, arrow-key bird movement, Enter listen-in, Escape exit, complete keyboard paths, and focus stability by bird ID: keyboard users can operate the moving scene and secondary surfaces.

- Call captions optional for normal audio, default when audio is off or unavailable, and explicitly enableable: captions cover audio absence without forcing them for everyone.

- WCAG AA contrast for top bar, captions, settings, notebook, errors, and visible narration in all light/weather states: visible text remains readable across the scene's palettes.

- Plain direct language for system errors and accessibility settings: these surfaces should never use evasive bird prose.

- Screen reader, vestibular, keyboard-only, zoom/phone, and audio-off caption testing before release: acceptance depends on real assistive technology and mode-specific review.

### 9. Budgets, instrumentation, and rollout

- Initial JS under 2 MB gzipped, critical bird SVG/CSS and private snapshot before secondary work, first bird under 500 ms, sustained 60 fps, no retained-memory trend, pooled audio, capped voices/ornaments, and virtualized notebook: the product must be fast and sustainable on defined mid-tier hardware.

- Last two major Chrome, Safari, Firefox, and Edge versions supported with older browsers receiving an unsupported-browser page: NOT RECOVERABLE FROM PLAN

- Aggregate operational counters and histograms only, with no account/bird/event dimensions or stable analytics user ID: observability supports operations without tracking private relationships.

- Short-retention diagnostic logs, no analytics warehouse flow, and no telemetry/model-training access to simulation database or private interaction events: diagnostics stay controlled and separate from analytics and training pipelines.

- No instrumentation of return frequency, average drift, popular offers, bird traits, friend graphs, or per-account relationship data: the plan excludes relationship analytics even when they might be product-informative.

- Synthetic scenarios, opt-in research sessions, and qualitative accessibility reviews for calibration: tuning happens outside private production-event population analysis.

- Gated rollout through Foundation, Core aviary, and Full v1 surface, with internal dogfood, simulated weeks, opt-in beta, then public ramp: rollout earns risk down before public growth.

- Do not increase the seven cap: NOT RECOVERABLE FROM PLAN

- Server flag to pause new adoptions or call-scheduling variants, plus rollback through versioned grammar and additive migrations without regenerating birds: reversibility must protect existing identities and vectors.

### 10. Verification and risk controls

- Deterministic fixtures, property tests, browser integration tests, audio golden checks, copy reviews, 30-minute performance/memory tests, and real assistive-technology sessions: verification is the release gate for simulation, privacy, sync, audio, copy, performance, and accessibility invariants.

- Drift risk controls including synthetic week/three-week trajectories, presence conformance, per-day caps, shadow simulation, versioned config, and no history rewrite: drift should not be too fast, too slow, or corrupted by false presence.

- Multi-device and retry risk controls including server sequence, idempotency IDs, one tick writer, stored checkpoints, overlap-union presence, restore drills, and two-device fixtures: these prevent lost or duplicated drift.

- Audio risk controls including grammar variation, stable signatures, bounded chorus density, listener studies, headroom/limiting, and silence-plus-captions fallback: these counter canned or uncanny sound while accepting browser autoplay as a launch constraint.

- First-frame and performance risk controls including private edge snapshot, phase anchors, critical first-bird path, lazy secondary surfaces, visibility cleanup, and sustained-device benchmarks: these address sterile first frame, frame regressions, and memory regressions.

- Accessibility risk controls including narration, caption, and reduced-motion design reviews plus release-blocking screen-reader, keyboard, contrast, and vestibular tests: accessibility must not become a static fallback.

- Privacy and visit-scope risk controls including PII vault, encrypted invite addresses, owner/visitor projections, no interaction telemetry join, default-off visits/notifications, deletion audit, and revocation checks on every visitor pull: these reduce privacy and scope leakage.

- Final v1 acceptance demonstration with a returning owner on a second device, continuing first frame, stable identities and accumulated vectors, elapsed-time mood, no-sound/screen-reader/reduced-motion meaning, and read-only visitor non-influence: the demo ties together continuity, accessibility, multi-device sync, and read-only visiting.
