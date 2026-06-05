## System-level intent

- **A living scene, already in progress.** The plan repeatedly centers the feeling that the aviary "was already alive before the user arrived." This shows up in birds being "mid-motion on first render," the first frame showing the aviary "mid-action," no spinner, procedural calls that avoid exact repetition, and a return greeting that should feel "like the bird noticed, not like the system started a scene."

- **Birds notice the user; UI chrome does not announce the user.** One invariant says, "The user is noticed by birds, never announced to by UI chrome." The same intent appears in "without any textual welcome," "Never pair greeting with textual welcome, banner, toast," and the guardrail that "Any UI copy that greets the user textually on return is a bug."

- **No gamification, obligation loops, or pet-care punishment.** The plan excludes achievements, badges, streaks, scores, levels, XP, counters, leaderboards, "Tamagotchi mechanics," death, hunger, distress, decaying happiness, feeding schedules, and guilt surfaces. The drift function is "not a reward system," absence creates "no negative personality drift," and notebook entries must not become "session logs or engagement reports."

- **Canonical aviary state belongs to the server.** The first invariant says, "The server is the only writer of canonical aviary state and personality vectors." This intent is echoed by server-authored simulation ticks, ordered event logs, snapshot versions, "Clients never tick the simulation," and the guardrail that "Any client-side simulation tick is a bug."

- **Presence is attention, not open-tab time.** The second invariant says, "Presence is measured as attention, not mere open-tab time." The plan grounds this in the three-condition presence rule: visible document, focused window, and recent pointer/key activity, with server clamping and no raw pointer/key timelines.

- **Interaction data exists to run the user's aviary, not aggregate behavior products.** The fourth invariant says this directly. The architecture reinforces it with aggregate-only telemetry, no account UUID or bird ID in metrics labels, observability that "must not read simulation records," and monitoring "operational health, not bird behavior."

- **Hidden traits should become expressive, not numerical.** Personality vectors are internal, bounded values; the user never sees vector values in UI, narration, captions, notebook, analytics, or settings. The plan allows traits to appear as plumage richness, calling tendency, greeting, mood, and slow drift, while forbidden-surface tests reject "boldness number" and other stat surfaces.

- **Naturalist voice instead of state labels.** Notebook prose, captions, and narration use "naturalist prose," "lowercase, present-tense, specific, observational" language. The plan forbids "state-list output," "mood: content" labels, technical call labels, event-log wording, and gamification copy.

- **Accessibility is part of the core aviary, not a fallback shell.** V1 includes "First-class accessibility." Reduced motion is a "parallel rendering mode" over the same snapshot model, screen-reader users should understand "the aviary as a living scene," captions must match procedural calls, and accessible surfaces must not expose hidden trait numbers.

- **V1 scope is calm, bounded, and deliberately non-social.** The plan keeps v1 "browser-only," "single-account," "single-aviary," with two starter birds and a lifetime cap of seven. Visits are optional and read-only; social network features, public discovery, visitor avatars, comments, chat, follows, rankings, and show-off rendering are excluded.

## Per-feature whys

### Product stance and v1 scope

- Browser-only delivery: NOT RECOVERABLE FROM PLAN
- Single-account, single-aviary product: The plan centers v1 on "one canonical aviary per account" and excludes social network features, public discovery, shared co-presence, and public comparisons.
- Two starter birds: The experience starts with a small coherent aviary and presents birds as "arrivals, not catalog selections."
- Lifetime growth to seven birds: The plan wants the aviary to "slowly grow" while keeping age-based arrivals conservative until "audio recognizability and scene density hold up."
- System-selected starter species: Server-side selection from a "small coherent species pool" avoids catalog browsing and optimization.
- User-assigned bird names: Names let notebook and narration prefer "bird names" and concrete observations rather than generic state labels.
- Rename support: Bird identity remains stable because `bird_id` is "never replaced by rename or species updates."
- Stable bird identities: Stable IDs, visual seeds, and call seeds preserve individual continuity across names, drift, visuals, and procedural call identity.
- Hidden personality vectors: Internal vectors allow "small user interactions" to accumulate into slow personality drift without creating a stats surface.
- Persistent mood: Mood is "fast-timescale and visible," persists across sessions, and "must not reset to neutral on tab open."
- Server-side drift: Drift is a "low-pass filter over slow signals, not a reward system," with no single-session jumps and regular use becoming visible only over weeks.
- Bird-to-bird interactions: Response calls, wary spread, and chorus events make the scene feel like a living aviary rather than isolated animated birds.
- Procedural call grammars: Calls vary procedurally, avoid recorded loops and exact repetition, and keep species and individual birds recognizable.
- Return greetings: Greeting selection should make the bird seem to have noticed the user, without "textual welcome, banner, toast, or 'you were gone' copy."
- Listen-in: The mix should feel "like listening, not switching channels," raising the focused bird while other birds lower but "never silence."
- Offers: Offers are scene gestures, "not by clicking directly on birds" and "without turning into a game button"; cooldowns prevent drift saturation.
- Settle: Settle quiets mood and closes the presence window; closing the tab without settling has "no recovery or guilt surface."
- Field notebook: The notebook captures sparse "naturalist prose" and "genuinely noteworthy events," not session logs, engagement reports, reactions, badges, or editable user notes.
- Presence accounting: Presence pings are coarse and purpose-limited so drift reflects attention, not mere tab-open time or raw movement traces.
- Day/night cycle: The day/night cycle uses the user's last-known local timezone so palette, mood, and time context feel locally grounded.
- Ambient weather: Weather is a rare ambient event with scheduling and mood modifiers, supporting naturalist variation without becoming user-controlled state.
- Ambient micro-motion: Leaves, feathers, parallax, and subtle motion are local ornaments for aliveness, not canonical state or simulation inputs.
- Top-bar controls: Controls stay above the scene and sparse so account, accessibility, notebook, and offer affordances do not become scene labels or game buttons.
- Read-only visit invitations: Visits allow ambient viewing while preserving the host aviary because visitors cannot mutate state, generate greetings, produce presence-time, or affect drift.
- Visit revocation, expiration, log, and optional notifications: Host control and transparency are supported, while notifications are off by default to avoid badges, toasts, push, or email unless explicitly enabled.
- First-class accessibility: Screen-reader narration, captions, keyboard navigation, focus, reduced motion, graceful silence, and contrast let the aviary remain complete across access modes.
- Aggregate operational telemetry: Telemetry measures load, frame, audio, request, tick, and session-duration health without turning interaction data into aggregate product data.

### High-level architecture

- Service-oriented architecture with hard data-boundary enforcement: The plan wants analytics code unable to import simulation storage and simulation code independent of client rendering types.
- Modular monolith option: Deploying as a modular monolith is acceptable "if that reduces operational complexity," while retaining service-like module boundaries.
- `web-client`: The client renders, synthesizes audio, captures interactions, and handles accessibility UI without mutating canonical simulation state.
- `api-service`: The API acts as the authenticated facade for snapshots, event ingestion, account operations, visits, export, and settings.
- `simulation-service`: The simulation service owns ticks, consumes account-scoped event logs, updates moods and vectors, writes canonical snapshots, and emits notebook candidates.
- Auth service or auth module: Magic links, validation, session tokens, email verification, rate limits, and device revocation keep account access secure and revocable.
- `notification-mailer`: Mail is limited to transactional purposes so email does not become an engagement or social notification channel by default.
- `export-worker`: Exports are on-demand and distributed through short-lived download links.
- `deletion-worker`: Deletion has a 30-day soft-delete recovery window followed by hard deletion across account-scoped stores.
- `observability-pipeline`: It is aggregate-only and "must not read simulation records or interaction payload fields" beyond coarse operational dimensions.
- Separate logical stores: Store boundaries keep account, event, snapshot, token, and telemetry data separated even if backed by one database cluster.
- Relational primary database: Durable account, bird, vector, mood, visit, notebook, and deletion records need relational ownership and constraints.
- Append-only account-scoped interaction event log: Ordered, idempotent events let server ticks apply drift and mood changes without client-side state merges.
- Snapshot cache: Current render state is keyed by account UUID and snapshot version for a low-latency snapshot path.
- Short-lived token store: Magic-link, email-change, visit, and export links need hashed or short-lived token handling.
- Aggregate telemetry backend: It excludes account UUID, bird ID, payload, vector, notebook, and email fields to preserve the privacy boundary.
- Encrypted email plus keyed lookup hash: Email is needed for login and rate limiting but must not appear in logs, partition keys, metrics, traces, queues, or simulation records.

### Client/server split and render pipeline

- Server-owned canonical state: Bird identity, species, names, vectors, mood, drift history, notebook entries, visits, account state, and snapshot versions remain authoritative on the server.
- Server-owned simulation decisions: Ticks, event ordering, personality deltas, mood transitions, weather, bird availability, and return-greeting signals are server decisions so clients cannot fork the aviary.
- Client rendering from snapshots: The client draws and interpolates a received snapshot "without mutating canonical state."
- Client local ornaments: Idle ornaments, audio synthesis, captions, top-bar fade, keyboard/focus behavior, and presence detection are client responsibilities because they do not alter canonical state.
- Interaction events with idempotency keys: Client events are submitted with idempotency keys so duplicate or concurrent devices produce ordered events, not divergent branches.
- Temporary network gap handling: The client may render the last valid snapshot briefly, but uses matter-of-fact error surfaces when canonical state cannot refresh.
- Render-ready snapshots: Snapshots are "not frame-by-frame animation scripts"; they provide enough state for an immediate first frame and short interpolation.
- Snapshot metadata and validity: `snapshot_id`, versions, server time, and validity let clients reconcile against authoritative server state.
- Per-bird render state: Stable bird ID, name, species, mood, perch, pose, call seed, schedule, and greeting flags let the client render aliveness without calculating simulation.
- Interaction affordance state: Offer cooldowns, listen-in target, and keyboard focus hints keep affordances synchronized with server rules.
- Notebook summary without unread badge: Notebook availability is exposed only for the notebook panel so the feature does not become a feed or badge mechanic.
- Accessibility descriptors from the same state: Narration and captions align with the visual/audio snapshot without exposing internal numerical traits.
- Local leaves, feathers, parallax, and cross-fades: These are "ornaments, not canonical state" and must never become simulation inputs.

### Domain model

- Synthetic account UUID: The account UUID is the internal identifier used outside the encrypted account record to avoid email leaking into operational systems.
- Encrypted verified email: Email is stored encrypted and used for verified account operations without becoming a public or logging identifier.
- Keyed email hash for lookup and rate limiting: The hash supports login lookup and rate limiting while remaining "not logged."
- Account deletion fields: Deletion timestamps support soft-delete, hard-delete deadlines, restoration, and final removal.
- Accessibility settings: Reduced motion, captions, narration cadence, audio, and contrast settings let users choose access modes explicitly.
- Visit notification opt-in and session visibility: Privacy/account preferences keep notifications off by default and make session management visible.
- Locale/timezone setting: Last-known IANA timezone drives the local day/night cycle.
- Session device label, last seen, and approximate region: These fields support account transparency and device revocation while avoiding precise location.
- Hashed magic-link tokens: Magic links are stored hashed, expire, and are single-use for secure account access.
- One aviary per account uniqueness constraint: The data model enforces one canonical aviary rather than relying only on application logic.
- `bird_count_cap` fixed at seven: The cap enforces v1's bounded lifetime growth and avoids uncontrolled scene density.
- Settled state: Settle persists until tab close or active re-engagement so the engine treats quieting and presence closure consistently.
- Weather state: Weather carries start/end times and mood modifiers to vary ambience and mood.
- Age unlock state: Future bird availability depends only on aviary age, not visits, interactions, subscription, or scores.
- Stable bird UUID: Birds are "not retired or swapped in v1," and identity survives rename or species updates.
- Starter flag: The first two birds remain distinguishable as starter birds in the model.
- Visual and call seeds: Stable random seeds create procedural variation and call grammar identity for each bird.
- Species metadata: Silhouette, palette, motif, night-active flag, and accessibility description templates support rendering, audio recognizability, night behavior, and narration.
- New-account adoption flow: The birds are presented as arrivals, not catalog selections, to avoid species optimization.
- Future bird arrival naming: The user may name arrivals without browsing a catalog or optimizing species choice.
- Personality vector persistence: Traits are stored with precision for slow drift while the scale remains internal.
- Vector export exception: Account export may include raw vectors as exported data, "clearly framed" away from a stats surface.
- Simulation-only vector mutation: No public route, guarded storage methods, separated roles, and tests prevent clients and API handlers from mutating vectors.
- Mood stored separately from personality: Mood can change visibly on a faster timescale without overwriting long-lived personality.
- Internal mood transition reason: Reasons support debugging and tests while staying non-user-facing.
- Append-only interaction events: Ordered account-scoped events let the tick apply presence, listen-in, offers, settle, naming, and admin events consistently.
- Coarse presence payloads: Presence stores booleans and durations, not raw movement, keystrokes, exact focus timelines, or per-frame attention traces.
- Notebook entries: Entries are read-only and sparse so they remain observational records, not an editable journal or engagement feed.
- Notebook source references hidden from UI: Internal refs support generation traceability without exposing event-log machinery.
- Visit invite records: Encrypted visitor email, token state, timestamps, revocation, and use times support controlled read-only access.
- Visit log: Host-visible email, date, approximate duration, invite status, and revocation action provide transparency without badges or social mechanics.
- Visitor constraints: Visitors cannot listen in, offer, settle, rename, open settings, alter notebook state, generate greetings, produce presence-time, or affect drift.

### API surface

- JSON over HTTPS for v1: The plan chooses small-payload simplicity first and reserves SSE/WebSocket for later if polling misses performance or battery goals.
- Idempotency keys on mutation routes: Mutations must be safe against duplicate requests and retry behavior.
- Magic-link request route: Neutral success responses avoid account enumeration; keyed hash and IP rate limits reduce abuse.
- Magic-link consume route: Single-use token consumption issues a per-device session and bootstraps the aviary for new accounts.
- Email-change request and confirm routes: New email verification must happen before commit.
- Account route: It exposes settings, session summaries, export/deletion status, and account controls in one account surface.
- Account settings patch: Settings copy stays "matter-of-fact" and avoids productized engagement language.
- Session revoke route: Per-device revocation gives users direct control over active sessions.
- Account export route: Export is queued and sent to the verified address through a short-lived link.
- Account deletion and cancel routes: Deletion is recoverable during the 30-day window and then hard-deleted.
- Host snapshot route: The snapshot returns canonical render-ready state, version, validity, cooldowns, notebook summary, and accessibility descriptors.
- Optional delta snapshot route: The delta form exists only if profiling shows payload costs, and must preserve "small-payload simplicity."
- Notebook API: Pagination provides notebook access without feed mechanics, badges, reactions, or editing.
- Bird management API: Names and settings are manageable, but there are "no stats."
- Rename route: Rename uses idempotency so display-name changes do not duplicate or replace bird identity.
- Snapshot pull triggers: Navigation, visibility, long render gaps, laptop wake, keepalive, and visit revocation checks keep the local scene fresh with canonical snapshots.
- Presence event API: Pings are sent only when all three presence conditions hold and include evidence as booleans/durations, not raw events.
- Listen-in, offer, and settle event APIs: Event routes append intent and return accepted event IDs while durable changes wait for the simulation tick.
- Immediate affordance state on event response: The client can animate responsively but must reconcile against the next authoritative snapshot.
- Presence validation: The server clamps durations, rejects impossible overlaps, deduplicates client events, ignores stale sessions, and ends pings on settle, hidden tab, blur, expiration, or inactivity.
- Host visit APIs: Invite creation, list/log, revocation, and notification settings provide controlled host management.
- Visitor visit APIs: Visit consumption and read-only snapshot access let visitors see the ambient aviary without host account data or mutation affordances.
- Revoked, expired, or deleted-host responses: Matter-of-fact copy preserves the product voice and avoids social drama.

### Simulation engine design

- Server-side tick loop: The tick runs about once per minute per active aviary and "must continue whether or not clients are connected."
- Tick step ordering: Loading state, local time, weather, presence, interactions, drift, mood, bird-to-bird interactions, poses, calls, notebook, snapshot, and metrics in order makes the output deterministic and auditable.
- Idempotent tick failure handling: Transaction boundaries or per-tick versions prevent vector deltas from applying twice or events being marked processed without the produced state.
- Drift low-pass filter: Drift uses slow signals, explicit constants, and fixtures so it remains calibration, not reward.
- Presence-time as dominant positive signal: Presence should make quiet attention matter across all birds without becoming open-tab counting.
- Listen-in duration: Listen-in is a strong bird-specific signal for social warmth and vocal frequency because it reflects directed attention.
- Offers as small signals: Offers nudge curiosity or boldness without dominating drift.
- Settle as mood quieting: Settle closes presence and quiets mood, but is explicitly "not a personality reward."
- Absence behavior: Absence may reduce greeting likelihood indirectly but "never lowers stored trait values."
- Additive, bounded, monotonic vector deltas: Server-authored deltas avoid conflict and keep expressive movement within calibrated bounds.
- Per-bird accumulators and daily delta gates: Accumulators and caps prevent active sessions from saturating traits.
- Calibration simulations: Synthetic profiles prove absence, occasional visits, quiet watching, listen-in, and heavy offers behave within expected drift timing.
- Mood transition model: Deterministic rules plus weighted randomness make mood visible and varied without resetting to default on open.
- Time-of-day mood effects: Morning, dusk, and night biases make mood align with local day/night context.
- Weather mood effects: Rain and wind alter vocal and wary/drowsy tendencies as ambient modifiers.
- Interaction mood effects: Seeds, still pool, song fragments, and settle create immediate mood variation.
- Personality mood effects: Boldness, curiosity, social warmth, and vocal frequency shape greeting, investigation, response calls, and wary resistance.
- Bird-to-bird mood effects: Alarm-like calls and chorus allow mood propagation across birds.
- Return greeting selection: One primary greeting bird, absence-length variation, staggered secondary greetings, and no synchronized all-bird greeting keep greetings natural.
- Offer type reactions: `seed`, `song_fragment`, and `still_pool` produce different bird reactions tied to mood, species grammar, curiosity, and vocal frequency.
- Per-bird offer cooldowns: Cooldowns of a few minutes prevent drift saturation while quiet language avoids punishment.
- Notebook generator from state transitions and event summaries: It ranks specificity and rarity while avoiding raw user behavior, session logs, and engagement reports.
- Notebook templates with controlled variation: Template-driven prose is safer and sufficient; an unconstrained generative model requires safety, auditability, and privacy review.
- Forbidden notebook language tests: Tests reject achievement, streak, score, level, user-visited language, happiness meters, and trait numbers.

### Sync and consistency model

- Shared snapshot plus ordered event log: Multi-device sync comes from all clients reading the same snapshot and submitting to the same log.
- No client-to-client sync: The plan avoids client merges because personality state has one server-owned branch.
- Optimistic concurrency for snapshots and ticks: Versions and cursors prevent stale writes from overwriting canonical state.
- Duplicate event deduplication: Account, session, and client event IDs make retries safe.
- Client cannot submit absolute mood or personality values: Concurrent clients create more events, not divergent vector branches.
- Stale-device event handling: A phone with an older snapshot can submit an event and the tick applies it to the latest vector.
- Offer cooldown and settle authority: The server validates cooldowns and the next snapshot is authoritative.
- Offline behavior: Offline clients stop presence, do not accumulate drift, do not replay long offline presence, and reconcile on a fresh snapshot.
- Online refresh delay behavior: Last snapshots can render briefly with bounded local ornaments and audio schedules.
- Sync privacy boundary: CDN keys, metrics labels, client error reports, visitor snapshots, and analytics must not leak account-specific simulation details.

### Frontend rendering pipeline

- Initial load under 500ms to first bird: The plan uses this budget so the first experience is immediate aliveness, not waiting UI.
- Server-rendered or edge-injected initial snapshot: This helps first bird visibility and first-frame rendering.
- Critical CSS and first-render bird primitives early: Core aviary rendering is prioritized over secondary surfaces.
- Code-split notebook history, account settings, visits, export, and non-critical assets: Non-core surfaces cannot block first bird.
- No spinner: Slow initial state shows "quiet field with soft sky color and faint motion cues" so even loading preserves product voice.
- First frame mid-action: Pose phase, ambient state, and scheduled calls make the scene feel continuing rather than starting.
- Single responsive horizontal scene: No scrolling, panning, or zooming keeps scene layout controlled by the product rather than user arrangement.
- Three depth planes and perch zones: Background, middle, foreground, and server-selected zones provide depth while preserving canonical perch choice.
- Responsive coordinates: All birds stay in frame on narrow phones and wide desktops.
- Local day/night/weather palette: Calm naturalist colors reinforce time and weather context.
- Top bar above the scene: Controls remain available without becoming part of the aviary scene itself.
- Top-bar fade: It fades during stillness to preserve ambience and restores on input or focus for discoverability and keyboard accessibility.
- Lightweight bird rendering primitives: SVG/canvas/WebGL choice is driven by the 2MB and 60fps budgets.
- Species silhouettes and stable visual seeds: Birds remain individually recognizable while supporting variation.
- Plumage saturation from hidden vector: Personality can be expressed visually without exposing the trait value.
- Pose families and mood-shaped idles: Preen, scan, head-tilt, shuffle, rest, watch, approach, and call make mood visible.
- Reduced-motion cross-fades: Reduced motion preserves aviary state while replacing movement paths and continuous micro-motion.
- Seeded procedural timing and pose blending: Repeated behaviors should vary rather than replay identically.
- Listen-in rendering: Click, tap, or focus raises one bird in the mix without adding tooltips or scene labels.
- Offer rendering: Offers appear as gestures and local anticipation stays subtle and reconcilable to snapshots.
- Settle rendering: Evening lighting and call quieting make settle feel like a scene change rather than an account state.
- Five-second settle undo window: NOT RECOVERABLE FROM PLAN
- Notebook UI: It opens from the top bar and remains read-only, sparse, naturalist, and free of reactions, badges, edit/delete, or annotations.
- Reduced-motion renderer adapters: Building adapters over the same snapshot model keeps reduced motion first-class rather than a late CSS override.

### Audio pipeline

- Procedural call grammar: Motifs, bird seeds, personality mapping, mood changes, and species recognizability replace recorded loops with identifiable variation.
- WebAudio synthesis: Client-side oscillators, noise, envelopes, filters, gain, and subtle depth cues generate calls without downloaded recorded audio.
- Shared audio node pools: Node reuse prevents memory growth.
- Snapshot-provided call windows plus local seeded variation: The server schedules canonical windows while the client avoids exact repetition.
- Ambient chorus: Multiple birds can call as a "real chorus" while loudness remains constrained and calm.
- Depth-aware mixing: Distance and depth affect level and filtering for spatial coherence.
- Weather and time-of-day mixing: The mix dampens or warms with ambient state.
- Listen-in mix: Slow ramps make focus feel like listening, and other birds lower but never silence.
- Settle/night mix: Calls quiet gradually while night-active species may continue calling.
- Captions from the same call grammar event: Captions match the generated audio rather than describing separate inferred state.
- Caption placement and voice: Captions sit near the calling bird, fade with calls, avoid clutter, and use naturalist voice.
- WebAudio fallback: If audio is unavailable or denied, the product uses graceful silence, captions on by default, and matter-of-fact copy, with no recorded fallback.

### Accessibility plan

- Screen-reader narration from the same snapshot: Narration describes the living scene consistently with visuals and audio.
- Idle narration cadence: A 30-60 second cadence and priority queue prevent flooding and respect interruption behavior.
- Event narration priority: User-initiated return greetings, offers, and settle can narrate faster because they are immediate interactions.
- Deterministic narration templates: Golden outputs and forbidden phrase checks keep narration testable.
- Possible server-side narration generation: Server generation is considered if it improves consistency with notebook language.
- Keyboard map: Keyboard users can reach top bar controls, birds, listen-in, panels, offers, settings, and settle.
- Stable spatial bird focus order: Arrow-key focus among birds follows the scene rather than arbitrary DOM order.
- Focus treatment: High-contrast outlines and no hover-only affordances make controls discoverable across palettes.
- Motion and captions settings: `prefers-reduced-motion`, explicit reduced motion, captions, audio-failure captions, and contrast checks keep access modes available from first render.
- Accessibility acceptance tests: V1 ships only when screen-reader, keyboard-only, reduced-motion, captions, and hidden-trait privacy requirements are satisfied.

### Performance and observability

- Initial JS under 2MB gzipped: Bundle size protects first-load performance.
- Snapshot payloads in kilobytes: Small snapshots keep refresh cheap and battery/performance friendly.
- Runtime 60fps and no memory growth: The plan wants rich rendering/audio to stay stable for 30-minute sessions.
- Hidden-tab rendering stop: Rendering stops when hidden while simulation continues server-side, preserving battery and canonical state.
- Tick p99 latency alarm at 5 seconds: Server simulation health needs operational alerting.
- Snapshot cache low-latency path: Current snapshots should serve quickly without recomputing simulation on demand.
- Synthetic browser checks: Common-geography automation measures first bird, JS size, snapshot latency, frames, audio init, reduced-motion first render, and accessibility smoke paths.
- Aggregate RUM: Load, first-bird, frame, audio, request, tick, and session-duration histograms monitor operations without account or bird dimensions.
- Telemetry exclusions: Per-bird state, personality values, notebook content, offer/listen details, visit graph, email, account UUID, and raw pointer/key data are not collected.
- CI quality gates: Tests for drift, idempotency, mood persistence, cooldowns, visits, auth, deletion, snapshots, accessibility, bundle size, performance, audio cleanup, privacy lint, and forbidden surfaces enforce the plan's boundaries.

### Security and privacy implementation

- Fifteen-minute magic-link expiry: Short-lived links reduce token risk.
- Single-use hashed magic links: Hashing at rest and consume-once behavior harden account access.
- Rate limits by keyed email hash and IP: Abuse controls avoid logging raw email.
- Secure HTTP-only session storage or equivalent: Session tokens should be hardened and hashed at rest.
- Per-device session revocation: Users can revoke individual devices.
- Email change verification: The new address must be verified before account email changes.
- Email encryption at rest: Email remains protected as PII.
- Account UUID everywhere else: Non-email identifiers keep operational systems from exposing email.
- Short-lived signed export URLs: Export access is time-bounded.
- Soft-delete and hard-delete: Account state is hidden/restorable first, then birds, vectors, notebook, visits, sessions, events, telemetry ties, and export artifacts are removed.
- Privacy policy in account settings: It names aggregate categories and excludes per-bird interaction state from analytics, training, and third-party sharing.
- Host endpoint ownership checks: Host routes require account session and ownership.
- Visit endpoint checks: Visitor routes require a valid visit session and active invite.
- Visitor serializer exclusions: Visit responses omit host account data and mutation affordances.
- Revocation on next snapshot: Visitor access ends on the next snapshot pull.
- Deleted host invalidates visits: Deleted or soft-deleted accounts cannot continue to host visit sessions.

### Implementation phases

- Phase A foundations and contracts: Schema, snapshot/event contracts, auth, privacy-safe telemetry, starter species, bootstrap, and basic shell come first so sign-in, aviary creation, snapshots, no-email identifiers, and vector write protection are proven before deeper work.
- Phase B simulation core: Tick processing, drift, mood, presence, perch/greeting/weather/call descriptors, and calibration come before polish so drift timing, neglect behavior, mood persistence, and multi-client vector safety are validated.
- Phase C rendering and interactions: Scene, bird renderer, top bar, listen-in, offers, settle, interpolation, visibility refresh, and reduced motion prove first bird timing, no spinner/toast, keyboard navigation, and hidden-tab behavior.
- Phase D audio and accessibility: Procedural calls, grammar signatures, chorus/listen-in, captions, fallback, narration, contrast, and focus ensure two-to-seven bird recognizability and a complete aviary for reduced-motion and screen-reader users.
- Phase E notebook, account operations, and visits: Notebook, rename, export, revocation, email change, deletion/recovery, and visits complete sparse prose, account control, and read-only visitor behavior.
- Phase F hardening and launch ramp: Monitoring, RUM, reviews, gates, calibration, accessibility review, beta, and rollout ensure budgets pass, tick dashboards are live, telemetry cannot read simulation records, and forbidden surfaces remain excluded.

### Rollout plan

- Internal dogfood: Dogfood validates first-frame aliveness, greeting variation, call non-repetition, top-bar fade, no announcement surfaces, sync, revocation, presence window tuning, reduced motion, and captions.
- Invite-only beta: Beta calibrates drift pacing with real use while relying on per-account simulation, synthetic checks, and opt-in qualitative research instead of aggregate per-bird analytics.
- Audio and scene-density beta checks: Controlled tests validate recognizability from two to seven birds before ramping bird count.
- Notebook beta tuning: Beta tunes notebook sparsity and language.
- Visit beta tuning: Beta confirms invitation expectations and revocation behavior.
- Gradual public opening: Public rollout starts with two birds, conservative age-based additions, visit notifications off by default, and aggregate operational monitoring.
- Instrumentation from day one: First bird, bundle size, snapshot/tick latency, frame timing, audio-context errors, memory growth, auth errors, and visit authorization failures are monitored as operational counts.

### Risks, mitigations, guardrails, and open decisions

- Drift calibration mitigation: Calibration harnesses, synthetic profiles, golden ranges, daily max deltas, and qualitative review prevent drift from becoming a stat-management toy or feeling irrelevant.
- Presence inflation mitigation: The exact three-condition rule, server clamps, overlap rejection, and tests for hidden tabs, blur, no activity, sleep, and settle/tab-close prevent tab-open time from corrupting drift.
- Sync correctness mitigation: No client vector writes, ordered append-only events, idempotent ticks, snapshot versions, cursors, and concurrency tests prevent stale or concurrent devices from overwriting personality state.
- Audio uncanniness mitigation: Procedural grammar, no recorded loops, stable seeds, runtime variation, chorus tests, and audio-design review prevent repetition and blurred bird identity.
- Accessibility regression mitigation: Treating narration, captions, keyboard, and reduced motion as core phases plus golden tests avoids flattened labels and stripped fallback experiences.
- Product-boundary creep mitigation: Forbidden-surface tests, copy review, sparse top bar, and product review for counters, notifications, badges, public surfaces, or account metrics prevent gamification, announcements, social loops, and obligations.
- Privacy boundary mitigation: A separated telemetry module, PII-safe logging lint, no identifying metrics labels, and privacy review for new event types prevent simulation data from leaking into analytics, logs, training, or identifiers.
- Performance mitigation: Early bundle gates, code splitting, snapshot-first rendering, object/audio reuse, and 30-minute synthetic CI tests protect load, frame, memory, and audio budgets.
- Engineering guardrails: Vector mutation must live in simulation tick processing; textual return greetings, user-visible counters, visitor drift events, client simulation ticks, recorded call files, raw state narration, and identifying telemetry dimensions are bugs.
- Open implementation decisions: Exact trait ranges, coefficients, presence window, mood enum, tick cadence, polling versus SSE/WebSocket, bird rendering technology, species details, notebook template size, age schedule, and narration location remain build-time choices that "must preserve the PRD invariants rather than create new product surfaces."
