## System-level intent

- Continuity over application shell. This appears in the opening definition of done: the product must preserve "continuity, restraint, and the sense that the birds--not an application shell--noticed the user." It recurs in the invariant that "the first resolved scene frame is already in progress," in startup rules that avoid "spinner," "welcome text," and "app loaded" animation, and in the instruction to fetch a fresh snapshot after long hidden intervals rather than make the client simulate the gap.
- Server-owned canonical life. The plan repeatedly insists that "the server owns one canonical aviary," that clients submit "facts about interactions, never desired trait values," and that "no personality write path exists outside the tick worker." This shows up in the architecture boundary between web client, API service, and simulation workers, in deterministic tick transactions, and in multi-device rules where clients never merge "vectors, moods, or perches."
- Relationship privacy and non-exposure. The plan carries a privacy promise through hidden personality values, "render-safe" snapshots, encrypted email boundaries, no vector serialization through the general API, and "aggregate-only operational and performance telemetry." It also says telemetry must never join to "simulation events or bird state" and must not reconstruct an aviary.
- Presence means qualifying attention, not engagement time. The invariant says presence counts only while "visibility, focus, and recent pointer-or-key activity are all true," and "open-tab time is never treated as presence." The same principle appears in validated presence segments, tab hide behavior, overlap unioning, and the risk mitigation against "idle/background tabs."
- Non-punitive, monotonic expression. The plan says absence cannot "reduce a trait, create distress, or remove plumage saturation," and drift must be "monotonic toward expressive." It repeats this in the excluded scope for hunger, death, illness, decaying happiness, and punishment, and in the risk that "neglect accidentally becomes punishment."
- Restraint against gamification and social mechanics. The plan explicitly bars "score, streak, achievement, level, care meter, notification loop, public profile, discovery surface, shared aviary," and "leaderboard-ready interaction aggregates." It carries this through adoption without progress bars, notebook entries that are not a feed, visits without co-presence or visitor interactions, and dashboards that contain only allowlisted operational measures.
- Naturalist product voice with direct system language. The plan sets a voice split: product prose is "lowercase, specific naturalist prose," while identity, settings, errors, sync, and accessibility settings use "direct, matter-of-fact system language." The same intent appears in notebook/narration grammar, editorial review, and the risk that naturalist voice could become generic or "invades errors."
- Procedural individuality rather than canned presentation. The plan says calls are "synthesized procedurally," no recorded loops ship, each bird has a "stable_call_signature_seed," and human review must find signatures distinguishable through seven birds. Motion also avoids "short obvious loops" through varied action lengths, phase offsets, and rests.
- Accessibility as affective parity. Accessibility is not framed as a fallback: reduced motion is a "separate renderer," captions derive from actual calls, narration is composed from render-safe facts, and launch requires disabled-user review. The final checklist says full motion, reduced motion, screen reader, captions, keyboard-only, silence fallback, and zoomed mobile journeys must retain the aviary's "affective core."
- Visits as explicit, revocable, read-only access. The invariant says visits are "explicit, revocable, read-only" and do not contribute presence or interaction input. This recurs in separate visit endpoints, a single snapshot capability, no mutation routes, revocation revision keys, active invalidation, and the rejection of public-social mechanics.
- Deterministic, reproducible state across devices and retries. The plan uses idempotency keys, server ordering, monotonic snapshot revisions, row locks, deterministic stochastic seeds, engine/config versions, and property tests for retries and row iteration order. The stated intent is that retries produce "the same result" and rollout can be reproduced without retaining raw private behavior forever.
- Bounded architecture with fewer failure modes. The plan prefers PostgreSQL transactions, append-only interaction rows, an outbox, row-level tick claims, bounded caches, and no client database. It explicitly rejects Kafka and whole-application event sourcing for v1 because the chosen shape provides "the required correctness with fewer failure modes."

## Per-feature whys

### Outcome and product invariants

- One canonical single-account aviary: the plan frames this as the base unit for continuity and server ownership, with "one canonical aviary per account" and no multiple aviaries or shared accounts in v1.
- Two starter birds: the plan says the aviary "starts with two individually recognizable birds," making recognizable individuality a first-session requirement rather than a later expansion.
- Maximum seven birds: the plan ties the cap to recognizability, snapshot size, tick cost, frame pace, memory, and chorus gates before ramping "through 3, 5, then 7."
- Age-based bird additions: the rationale is to keep adoption independent of visits, event counts, or attention; "dates derive only from aviary age" and adoption avoids catalog, rarity, progress, and interaction incentives.
- Stable bird identity with naming and renaming: stable IDs "survive renames and migrations," so names can change without breaking continuity, restore, or migration checks.
- Five-trait count: NOT RECOVERABLE FROM PLAN.
- Hidden personality values: the rationale is privacy and anti-stats restraint; values are absent from interactive APIs and UI, never in snapshots, logs, metrics, or dashboards, except for the narrow account-export tension.
- Persistent mood: mood is "canonical state, not a client label," so mood can shape weather, calls, greetings, and social response while remaining consistent across devices.
- Three perch-zone count: NOT RECOVERABLE FROM PLAN.
- Bird-to-bird responses: the plan uses neighbor mood/call intent and warmth so birds feel alive to one another, not only reactive to the user.
- Rare ambient weather: weather provides temporary variation to mood and expression while devices agree on persisted start/end; the plan keeps it subtle, with no thunder, snow, alerts, or user controls.
- Local-time day/night state: local time supports day bands, day-boundary transition bias, and long palette changes, preserving continuity without snapping to neutral.
- Roughly one-minute server-side progression cadence: NOT RECOVERABLE FROM PLAN.
- Return greeting: the plan's why is that birds, not the shell, noticed the user; greetings derive from absence, mood, boldness, warmth, and variation, and must not trigger all birds together or return greeting copy.
- Passive presence: presence is the dominant validated signal for drift, but only when visibility, focus, and recent activity make it qualifying attention.
- Listen-in: listen-in creates bird-specific attention, rebalance of the audio mix, and bounded server facts without broadcasting one device's mix to another.
- Offers: offers let the server resolve "one mood/personality-shaped reaction" from canonical state while cooldowns and caps keep them small and non-spammable.
- Seed, song-fragment, and still-pool offer types: NOT RECOVERABLE FROM PLAN.
- Settle with five-second undo: settle is for immediate quieting and presence termination, with a reversible presentation directive; the plan says it is not mandatory and does not create negative personality direction.
- Sparse read-only field notebook: the notebook preserves rare naturalist observations without becoming an editable journal, generic event log, attendance summary, or feed.
- Single responsive horizontal scene: the plan keeps birds in one scene with no camera/pan state, no scrolling, and no cropped birds across supported viewports.
- Full-motion renderer: the rationale is expressive bird behavior through bounded micro-actions, mood/personality weighting, phase offsets, rests, and no obvious loops.
- Reduced-motion renderer: the rationale is accessibility parity; it removes flight paths, drifting ornaments, and pointer parallax while preserving birds, mood, calls, drift, greetings, offers, and notebook observations.
- Subtle ambient ornaments: ornaments add atmosphere while staying "local-only, sparse, pooled, and not canonical," avoiding heavy parallax or canonical state complexity.
- Fading top bar: controls visually recede so the aviary remains primary, but pointer, keyboard, focus, and touch reveal them so controls are not unreachable.
- Quiet-field loading: the plan uses a quiet sky and low-motion cues to avoid spinner/progress copy and preserve the fiction that the scene is already alive.
- One-time empty first arrival: only post-onboarding may use an empty field and soft fly-in, and server-side completion prevents a cache clear from replaying "the first-arrival fiction."
- Procedural call generation: the plan rejects recorded loops so birds do not sound canned and so signatures can be generated from grammar, stable seeds, mood, and traits.
- Chorus mixing: the rationale is to allow multiple recognizable birds without clipping, flattened signatures, or unbounded voices.
- Gradual listen-in rebalance: ramps avoid hard cuts and preserve a nonzero ambient floor for other birds, keeping focus from becoming total mute.
- Mute handling: explicit mute is preserved while visual timing and captions continue; the plan says not to infer hearing status.
- Runtime-derived captions: captions come from the same expanded motif before synthesis so they describe "the sound actually scheduled."
- Graceful silence when WebAudio is unavailable: the plan prefers honest silence, session captions, and a matter-of-fact status over recorded fallback or fake playback.
- Screen-reader narration: narration gives cohesive observations from render-safe facts without announcing every tick, call, or pose.
- Keyboard navigation and visible focus: the plan makes bird targets reachable and operable by Tab, arrow keys, Enter, and Escape, with focus rings as an accessibility affordance rather than ambient chrome.
- WCAG AA contrast, text zoom, and forced-colors support: the rationale is usable DOM surfaces, visible focus, and support for narrow viewports without obscuring birds or controls.
- User-controlled accessibility preferences: preferences let users choose captions, reduced motion, and narration behavior without stripping the aviary's functional or affective core.
- Read-only email invitations: the plan keeps sharing explicit and narrow: one supplied email, no address-book import, and no onboarding prompt.
- Visit grants, revocation, and visit log: grants scope one browser to one aviary snapshot, revocation wins over cache hit rate, and the log preserves approximate duration without entering simulation.
- Visit notifications default off: the rationale is restraint against notification loops; notifications are explicitly enabled only and kept hidden from onboarding during rollout.
- Aggregate-only telemetry: metrics exist for operations and performance, not product mining; they cannot be joined to simulation events, bird state, visit relationships, or interaction histories.
- Excluded native apps, payments, subscriptions, multiple aviaries, customizable scenes, public profiles, feeds, and social mechanics as individual product exclusions: NOT RECOVERABLE FROM PLAN.

### Architecture and ownership boundaries

- TypeScript monorepo language choice: NOT RECOVERABLE FROM PLAN.
- Independently deployable packages with one shared domain-contract package: the rationale is clear ownership boundaries while keeping client, API, simulation, email, and contracts aligned.
- Web client owns interpolation, animation, WebAudio, input, and accessibility presentation: this lets the browser present the scene while never calculating persistent drift.
- API service validates and deduplicates interaction commands: the API owns authentication, command facts, immediate offer reactions, account/invite operations, and append-only domain events so clients cannot choose state.
- Simulation workers own deterministic mood, drift, weather, and social-bird rules: the plan puts canonical progression in server transactions and publishes materialized snapshots after state changes.
- Email worker uses an outbox boundary: no simulation event payload enters email, and addresses are resolved only inside the email boundary.
- PostgreSQL primary: the plan uses it as the system of record for accounts, sessions, aviaries, birds, events, notebook, invitations, visits, leases, and revisions, and to keep canonical state transactions together.
- Private snapshot cache: the rationale is fast render-safe snapshots by synthetic aviary ID and revision while excluding email and personality vectors.
- Object storage for export archives: object storage is limited to short-lived authenticated account exports with expiring download URLs and automatic deletion.
- No Kafka, whole-application event sourcing, or client database in v1: the plan says append-only interaction events, an outbox, and row-level tick claims provide correctness with fewer failure modes.
- Parallel authenticated navigation operations: verifying the session, fetching the snapshot, and streaming critical scene assets in parallel supports fast first scene without publicly caching authenticated HTML.
- Low-frequency refresh, visibility regain refresh, and hidden-client suspension: the rationale is convergence and freshness without wasting rendering or audio scheduling when hidden.
- Command acknowledgement with authoritative event ID and expected revision: this lets local ephemeral presentation reconcile to the next snapshot without last-write-wins merging.

### Data model

- UUIDv7 or UUIDv4 synthetic identifiers: NOT RECOVERABLE FROM PLAN.
- Server-side synthetic identifiers and email HMAC/encryption: the rationale is that email appears only where product explicitly requires it while internal references use UUIDs.
- Account timezone as canonical and editable: the simulation needs a canonical IANA timezone for local day/night, local-day boundaries, and cross-device consistency.
- Device sessions store token hashes, not raw bearer tokens: the rationale is session security and revocation enforced on every authenticated mutation and bounded-cache read.
- Magic links are atomic and single-use: the rationale is replay prevention, neutral account-existence responses, and secure account creation/recovery.
- Exact 15-minute magic-link expiry duration: NOT RECOVERABLE FROM PLAN.
- Interaction events have idempotency keys and bounded occurred-at times: this lets clients submit facts safely while the server validates clock bounds, ownership, and deduplicates retries.
- Presence segments are validated events, not unbounded totals: the rationale is to reject overlap, excessive clock skew, and background/idle inflation.
- Immediate reactions: they let accepted offers render immediately and converge on other devices "without letting clients choose a reaction."
- Notebook entries are immutable and cursor-paginated: the rationale is durable naturalist observation without raw trait values, user-attendance facts, or analytics copies.
- Adoption availability records: dates derive only from aviary age and a versioned schedule, making adoption independent of visits and interactions.
- Materialized snapshots: the render payload excludes hidden vectors and private events so clients receive only render-safe state.
- Invite one-time link records: they allow a single 30-day one-time email link with status, consumption, revocation, and no public discovery.
- Exact 30-day unused invite expiry duration: NOT RECOVERABLE FROM PLAN.
- Visit grants: consuming the one-time link creates a browser grant, replaying the link fails, and the grant remains usable until revoked.
- Visit sessions: they maintain approximate duration only and never emit presence or bird interactions.
- Account settings: settings hold reduced motion, captions, mute, and notification preference, keeping accessibility and audio choices explicit.
- Typed personality vector with bounds, version, and encryption: the rationale is normalized finite state, migration/config reproducibility, and no free-form blob through the API.
- Processed interaction retention window: the plan proposes a bounded reconciliation/debug window so canonical vector and mood, not event replay, remain the persistent source of truth.
- Exact proposed 30-day processed-event retention window: NOT RECOVERABLE FROM PLAN.
- Notebook and visit-log retention for account lifetime: the plan preserves them for the account lifetime unless hard deletion occurs, while keeping them out of analytics.
- Recoverable deletion with hard-delete date: the rationale is a direct recovery action during the soft-delete window while ticks, invites, and events stop.
- Exact 30-day recoverable deletion window: NOT RECOVERABLE FROM PLAN.
- Hard deletion cascade: the rationale is complete account erasure across online tables, cache, outbox, grants, telemetry deletion index, backup tombstones, and export objects.
- Recovery without missed presence: recovery re-enables ticking "without inventing missed presence," preserving the non-punitive presence model.

### API surface and contracts

- Versioned endpoints and event payloads: the rationale is stable contracts with request IDs, server time, stable error codes, and matter-of-fact client errors.
- Magic-link request endpoint: rate limits by email HMAC and IP and returns the same neutral response whether an account exists, preventing enumeration.
- Magic-link consume endpoint: atomically consumes a token, creates or recovers an account, and issues a rotating secure session cookie.
- Account read/update endpoints: they expose matter-of-fact account/settings data and canonical timezone, but no simulation internals.
- Verified email change: the old address remains valid until the new verification link is consumed, preventing unverified account takeover.
- Session list/revoke endpoints: sanitized device sessions and immediate revocation support per-device control and cache invalidation.
- Account export endpoint: the plan treats data portability as the narrow exception to vector non-exposure, gated by fresh magic-link confirmation and product/privacy sign-off.
- Export encrypted continuity blob alternative: if the hard prohibition overrides vector export wording, the plan preserves export continuity without displaying values by using an opaque encrypted blob.
- Account deletion/recover endpoints: deletion stops ticks and revokes visit grants, while recovery is possible after fresh authentication during the soft-delete window.
- Aviary snapshot endpoint: it returns render-safe state, revision, server time, transitions, call horizon, and settings, with ETag support and no vectors.
- Host session open endpoint: it creates a lightweight viewing session and returns a greeting plan derived from absence, mood, and boldness; visits are separate and never receive greetings.
- Aviary events endpoint: small batches with idempotency keys let the server validate source, time, ownership, and transitions and return per-event acceptance.
- Presence endpoint: it accepts only compressed qualifying segments and rejects hidden or unfocused declarations that conflict with lifecycle.
- Offers endpoint: server-side cooldown and canonical state resolve one reaction and prevent clients from choosing reactions or creating duplicate drift inputs.
- Settle endpoint: it records a session-scoped settle event, ends that session's presence, and returns a reversible quieting directive.
- Settle undo endpoint: same-session, five-second undo records re-engagement without rolling back already elapsed presence.
- Rename endpoint: private Unicode names can change while stable identity and state remain unchanged.
- Adoption endpoints: they return and accept only server-derived age-based arrivals, with maximum seven and no progress bars or countdowns.
- Notebook endpoint: it returns immutable bounded pages newest-first, explicitly not an interaction log or attendance summary.
- Listen-in events: start, heartbeat, and end facts bound duration, while visibility loss, bird change, Escape, empty-space click, and session termination end the session-local mix.
- Invite creation endpoint: one host-supplied email creates one 30-day one-time link, with no address-book import or onboarding prompt.
- Invite listing endpoint: invitations and visit-log rows are on demand, with no unread counts or settings badge.
- Invite revocation endpoint: revocation invalidates link, browser grants, and visit snapshots immediately.
- Visit consume endpoint: it issues a scoped HttpOnly visit-grant cookie without creating an account or host session.
- Visitor snapshot endpoint: every poll revalidates grant/revocation and returns host state without host controls, notebook, greeting, private fields, or mutation capability.
- Visit heartbeat endpoint: it maintains approximate visit duration only and cannot enter the simulation event table.
- Expired/revoked visit HTTP 410 surface: the plan uses plain "This visit is no longer available" language and prioritizes revocation correctness over edge-cache hit rate.

### Authentication, authorization, and privacy controls

- Token hashing with server-side pepper: raw magic and invite bearer tokens appear only in email URLs, reducing stored-token exposure.
- CSRF, strict origin checks, CSP, secure headers, output encoding, and dependency/SBOM scanning: these protect cookie-authenticated mutation routes and browser surfaces.
- Account-to-aviary ownership checks: every bird, notebook, invite, and snapshot is authorized through ownership; a visit grant has one capability, reading one snapshot.
- Email decryption isolation: account and email modules hold decryption, while logs use request IDs and synthetic UUIDs rather than email or private payloads.
- Log restrictions: logs must not contain email, bird names, event payloads, vector values, or invite tokens, preserving the relationship privacy promise during diagnostics.
- Separate simulation database/role and telemetry exporter: this blocks ETL, CDC streams, and warehouse access to interaction, bird, notebook, or visit-log tables.
- Operational metric allowlist: metrics may count requests, failures, durations, frame timings, audio failures, and tick latency, but not dimensions that reconstruct account, bird, trait, mood, species, invite, or relationship state.
- Bounded client diagnostics: only allowlisted numeric performance fields upload, with schemas enforced in code review and CI.
- Rate limits and replay detection: the plan protects magic links, invites, token consumption, and commands without building behavioral user profiles.

### Simulation engine

- Deterministic tick transaction: locking aviary and bird state, ordering host events, and applying all birds from the same prior snapshot prevents row order from changing bird-to-bird effects.
- Visitor records structurally excluded from host events: this preserves read-only visit semantics and prevents visitors from influencing simulation.
- Overlap normalization and signal caps: listen-in and presence are bounded so concurrent devices or repeated facts cannot inflate drift.
- Time-of-day, weather, and bird-to-bird inputs use account timezone: this keeps local-day behavior and social effects canonical.
- Nonnegative drift and range checks: finite/range/monotonic invariants enforce expressive-only personality changes.
- Notebook candidate scoring: rarity, novelty, and cooldown limit entries so active accounts do not create a feed.
- Transactional cursor, snapshot materialization, and cache invalidation: the rationale is atomic state advancement and recoverability around failures.
- Deterministic stochastic seeds: hashing aviary ID, bird ID, engine version, tick number, and namespace makes retries reproduce the same result.
- Engine/config version recording: rollout can be reproduced without retaining private raw behavior forever.
- Bounded catch-up: missed ticks run from prior canonical time in steps rather than one giant delta, and the client never simulates the gap.
- Drift low-pass function: saturating deltas make movement detectable over ordinary days and weeks while diminishing returns prevent rapid jumps.
- Presence, listen-in, and offer weighting: presence is dominant, listen-in is strongly bird-specific, and accepted/proximate offers are small and bird-specific.
- Daily exposure caps and cooldowns: a long session or repeated actions cannot accelerate "weeks of drift into hours."
- Settle and audio mute handling in drift: settle affects quieting and presence termination only, and audio mute must not penalize traits or become an engagement signal.
- Calibration cohorts: synthetic quiet watching, short daily visits, listen-in, offers, absence, and abusive repetition calibrate timescale without mining real users.
- No production drift analytics: the plan forbids using private interaction histories for population analysis, so tuning uses synthetic simulations, internal consented accounts, and prelaunch studies.
- Mood transition model: wary, content, curious, drowsy, and alert vary through current mood, local time, interactions, weather, neighbors, and personality, with dwell and hysteresis to avoid thrashing.
- Daily-ish mood reset: local-day boundary bias nudges toward equilibrium without snapping to neutral, preserving session continuity.
- Post-absence quietness as recency expression: quietness affects greeting and call probability but is not negative drift; birds remain alive and never look sick, sad, or accusatory.
- Perch choice weighting: boldness and mood map to front/middle/back choices with deterministic variation and collision capacity, while users never submit perch commands.
- Weather generation: an aviary-seeded calendar creates rare subtle events, persists start/end for device agreement, and avoids alert-like weather or controls.
- Greeting planner: eligible birds are chosen by absence, boldness, warmth, mood, and variation so greetings are personal, staggered, and non-synchronized.
- Species selection for adoption: the server chooses from a coherent six-species pool while avoiding catalog and rarity presentation.
- Call grammar: versioned species motifs plus stable signature seeds produce recognizable interval, timbre, register, and rhythm while mood/traits vary performance.
- Notebook generation: server-side curated naturalist grammar avoids remote generative models, trait numbers, session counts, streak-like phrasing, and judgments of user behavior.

### Multi-device consistency and offline behavior

- Monotonic snapshot revision: clients replace canonical inputs only with newer revisions and interpolate from server time.
- Random idempotency keys: safe retries return the original result while the server orders accepted events.
- Concurrent presence union: overlapping intervals per account are unioned so the same human minute is not double-counted.
- Session-local listen-in and settle presentation: a settle can warm/quiet one device without forcing another watched device into a goodbye state.
- Offer race transaction: simultaneous devices resolve once; one gets the accepted reaction and the other gets a cooldown response.
- Optimistic revisions for rename/settings: conflicts refetch and show matter-of-fact settings errors rather than silently last-write-winning.
- Bounded transient offline queue: the client may queue only small interaction facts with expiry, not adoption, rename, account, invite, or offer acceptance, and there is no offline simulation.
- Tab hide behavior: requestAnimationFrame and audio scheduling stop, eligible presence finalizes, and only session-local UI preferences persist.
- Resume after suspend gap: the client discards prediction and fetches a fresh snapshot before resuming transitions.

### Frontend rendering pipeline

- One GPU-accelerated 2D canvas/WebGL scene plus DOM layer: the plan puts visual scene work in one performant layer while controls, dialogs, semantic bird targets, captions, and narration remain accessible DOM.
- Deterministic scene layers: ordered sky, foliage, perches/birds, foreground treatment, accessibility layer, and top bar keep composition stable.
- Normalized scene coordinates and responsive constraints: narrow screens compress gaps and scale within readability bounds, wide screens expand negative space, and birds remain inside the safe viewport.
- No camera/pan state: the plan prevents panning, zooming, scene scrolling, and birds cropped out of supported viewports.
- Settle as fifth top-bar control: the plan resolves a discrepancy in favor of explicit interaction and keyboard requirements, with design sign-off.
- Inline critical CSS, quiet-field background, starter silhouettes, and minimum bootstrap: the rationale is first bird paint before noncritical settings, notebook, invite, account, audio, top-bar fading, or ornaments.
- Paint bird before audio/top-bar/ornaments when snapshot exists: visual life takes precedence over optional systems.
- Quiet sky when snapshot is unavailable: no spinner or progress copy, then direct replacement with the current scene.
- Snapshot transitions by state: movement paths, reduced-motion cross-fades, pose continuation, and honest fresh poses avoid teleporting except after long hidden intervals.
- Full-motion actors: layered skeletal/vector poses and bounded micro-actions create expressive life while dwell times and seeded variation avoid canned loops.
- Reduced-motion renderer teardown/swap: changing preference rebuilds the scheduler without reloading or duplicating audio, preserving parity.
- Top-bar reveal and focus behavior: opacity falls after stillness but never while focused/dialog active, and pointer, keyboard, focus, and touch reveal it.
- Bird listen-in input routes: click/tap or Enter starts, while repeat, another bird, empty scene, Escape, focus departure, tab hide, or session close ends it, making the state explicit and keyboard accessible.
- Spatial arrow traversal: arrow keys move among birds by spatial order, matching the scene rather than a hidden list.

### Audio pipeline

- AudioWorklet-based bounded polyphonic synthesizer: the plan generates motif tokens into tonal/noise components without fetching call audio.
- Per-bird gain/pan bus into ambient/weather/master buses: this preserves individual signatures while supporting spatial mix and chorus control.
- Gentle limiter and voice caps: the rationale is preventing chorus clipping without flattening individual signatures.
- Preallocated worklet state, ring buffer, and pooled typed arrays: the plan needs proof of no 30-minute growth.
- Listen-in ramps: focused bird gain rises and others fall to a nonzero floor over one to two seconds, avoiding hard cuts and full mute.
- Mood-shaped performance parameters: mood can change tempo, density, articulation, and willingness to answer without changing the stable motif fingerprint.
- Caption derivation from expanded motif: captions include contour, repetition, intensity, and source location for the exact sound scheduled.
- User mute: mute stops the master bus while keeping visual timing and captions, and the plan warns not to infer hearing status.
- Autoplay handling: the plan accepts honest browser constraints, avoids blocking audio modals and fake playback, and allows ordinary interaction to unlock audio.
- WebAudio/worklet failure mode: partial nodes are disposed, the session runs in silence, captions default on for that session, and no recorded fallback downloads.

### Accessibility implementation

- Landmarks for top bar, aviary, dialogs, and status: semantic structure makes the product navigable without exposing hidden mood labels or coordinates.
- Bird semantic button-like targets named by user-assigned name/species: accessible names use visible identity rather than hidden state.
- Tab order through account, accessibility, notebook, offer, settle, and scene: the plan puts controls and scene access in a predictable keyboard path.
- Modal focus trapping and restoration: focus is trapped only in true modals and restored on close, preventing keyboard loss.
- Naturalist narration composer: render-safe facts produce one lowercase observation every 30-60 seconds at idle without coupling to the animation loop.
- Exact 30-60 second idle narration cadence: NOT RECOVERABLE FROM PLAN.
- Narration deduplication and bounded queues: the plan avoids enqueueing every tick/call/pose and ensures rapid commands cannot grow the queue without bound.
- User controls for narration pause/current scene summary: users can pause narration or request summary without silencing ordinary control feedback.
- Call captions near birds: collision avoidance, contrast backing, and source location make audio visible while avoiding double-speaking with narration.
- DOM transcript for captions: screen readers can access call information without relying on visual proximity alone.
- Reduced-motion release parity: reduced motion ships initially and receives the same functional and editorial matrix as full motion.
- Manual accessibility review: automated checks are necessary but not sufficient, so keyboard-only, VoiceOver/Safari, NVDA, zoom, hearing/caption, and vestibular review with disabled testers block launch.

### Performance budgets and enforcement

- Initial JavaScript budget under 2 MB gzipped with 1.5 MB internal target: the rationale is CI enforcement and regression room for first-scene code.
- First bird visible under 500 ms: the budget measures navigation start to bird paint on pinned mid-tier mobile and shaped 4G so the first scene feels immediate.
- 60 fps idle gate: long representative traces measure frame pacing and p95/p99 long frames rather than average fps.
- No 30-minute memory growth: soak scenarios compare post-GC heap and worklet memory to a bounded steady-state envelope.
- Tick p99 below 5 seconds: duration is recorded by deployment/version and alerts on latency and due-queue lag without account dimensions.
- Snapshot payload ceiling starting at 32 KB compressed for seven birds: the rationale is keeping private snapshots in kilobytes and enforcing a concrete CI ceiling after prototyping.
- Public asset caching: content hashes and long-lived caching are allowed for public assets, while private snapshots are never publicly cacheable.
- Day-one instrumentation: navigation, snapshot, transfer, frames, heap, audio failure, tick, API, and email metrics support operations and performance.
- Synthetic browsers with anonymous starter fixtures: common-region checks cover reduced motion, silence fallback, and slow snapshot paths without real-user state.
- Deliberate non-measurement of bird behavior and relationship fields: the rationale is avoiding metrics that reconstruct an aviary or mine private relationships.

### Delivery sequence, rollout, and operations

- Milestone 0 contracts, threat model, and prototypes: the plan freezes invariant tests, schemas, allowlists, voice guide, ownership map, and ambiguity sign-offs before feature code expands.
- Milestone 1 identity and canonical state foundation: account lifecycle, encrypted email, one aviary/two birds, tick scheduler, snapshots, convergence, restore, replay, and revocation establish correctness first.
- Milestone 2 bird engine and core scene: trait, mood, weather, perch, greeting, renderers, and adoption are calibrated with synthetic scenarios before production drift.
- Milestone 3 interactions, audio, and notebook: listen-in, offers, settle, audio grammar, captions, narration, and notebook are paired with editorial validation and soak tests.
- Milestone 4 accounts completion and visits: settings, export, deletion cascade, invitations, grants, revocation, visit log, and notification settings are paired with authorization, privacy, and deletion/export drills.
- Milestone 5 hardening and launch candidate: browser matrix, unsupported-browser page, accessibility, localization, security, backup restore, chaos tests, performance gates, and runbooks complete launch readiness.
- Versioned server flags: engine configuration, call grammar, notebook templates, and maximum birds can change without behavioral analytics cohorts or user controls.
- Team dogfood with synthetic/resettable accounts: the rationale is operational correctness without using employee interaction patterns as production calibration data unless consented.
- Accessibility and audio design partner preview: the release blocks on affective parity, not only checklist defects.
- Invite-only production cohort at two birds with visits disabled initially: the plan watches aggregate errors, latency, memory/frame health, deletion/export, and support reports before expanding.
- Offers/notebook before quiet visits: authorization and revocation soak before visit expansion reduces social/privacy risk.
- Ramp maximum birds through 3, 5, then 7: the plan gates each level on chorus recognition, snapshot size, tick cost, frame pace, and memory while age eligibility controls real adoption.
- General availability after soft-delete and restore readiness: a full soft-delete window, restore drills, and tick p99/queue headroom are prerequisites.
- Kill switches for drift, notebook generation, visits, notification email, and noncritical weather: they pause risky systems without rolling vectors back or disabling canonical snapshot reads.
- Engine/config rollback by version: rollback uses explicit migrations and deterministic fixtures, never bird reset or regeneration.
- Day-one dashboards: only allowlisted aggregate operational measures appear, and qualitative calibration comes from opt-in research and support rather than mining interaction records.
