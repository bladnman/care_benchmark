## System-level intent

1. Keep one continuing, canonical aviary. This appears first in "one continuing aviary per signed-in account," "one canonical aviary," and the definition of completion where the owner "returns to the same continuing aviary on another device." It is reinforced by "unique account constraint forbids a second aviary," "one transactional writer domain," and "a stale client must never become the emergency source of truth."

2. Make the product feel alive immediately, not loaded or restarted. The plan repeatedly requires "recognizable birds already in motion," "mid-action first frame," "not a neutral loading pose," "no spinner, generic skeleton," and "no hydration reset." The one-time "gentle first fly-in" is explicitly an exception, and returning navigation "never reruns it."

3. Favor calm presence over engagement pressure. The product contract says "Leaving causes no penalty" and excludes "visit streaks," "adoption counters," "ranks," "badges," "engagement campaigns," "welcome banners," "return toasts," and "absence-duration copy." Later sections repeat "no punishment," "no guilt message," "no celebration when connection returns," and "no toast/badge framework on product events."

4. Keep personality hidden, private, and expressive rather than measurable. The plan says "numbers are never exposed to the user," "no trait meters," "no raw personality," "no vector values in DOM/ARIA/snapshot," and "without a stat or penalty." Personality should appear through "subtle perceptual change across weeks," "species-specific behavior curves," and "mood and expressive drift," not public controls.

5. Preserve stable bird identity across time, devices, and failures. This appears in "stable UUID," "private immutable identity seed/call signature," "survive every rename and version migration," "never seed a new identity to repair a missing vector," and "canonical vectors and identities survive retries, migrations, restore, and device changes."

6. Make the server authoritative for simulation while allowing prompt presentation. The plan uses "server-authored reaction plan," "only the tick role can update persistent personality and mood fields," "server-only additive changes," and "immediate animations are authoritative server instructions, not optimistic local personality changes." Client queues may hold commands, "never a vector delta."

7. Treat accessibility as the same product, not a fallback. The first section says launch must work with "a screen reader, reduced motion, silent audio, and keyboard input." Later, "accessibility failures block launch just like visual/audio failures," reduced motion must keep "scene state, bird identity, mood, calls, captions, and notebook identical," and the product must remain "specific and alive in narration, captioned silence, keyboard use, and reduced-motion cross-fades."

8. Use naturalist product voice for the aviary and matter-of-fact system voice for controls/errors. The plan calls for "Naturalist narration," "lowercase present tense," "specific verbs," and "observational" greeting/offer narration, while system errors use "stable system codes plus matter-of-fact copy, not naturalist messages."

9. Keep sharing quiet, explicit, read-only, and non-influential. This appears in "Explicit email invitations for read-only ambient visits," "visitor never sends a return command," "cannot submit simulation events," "no drift/greeting/listen caused by visitor," and the completion condition that a visitor sees "that same scene without influencing it."

10. Prefer procedural specificity over generic or prerecorded substitutes. The plan rejects a "generic bird shape differentiated only by color," "downloaded bird recordings," "recorded-loop fallback assets," and "temporary generic graphics/recorded calls." Six species need distinct silhouettes and calls need "recognizable anchor" identity.

11. Make operational observability service-health only. The plan restricts telemetry to "performance, availability, and anonymous aggregate health," says dashboards are "service health only," and forbids "clicks-per-session," "presence-per-account trends," "listen-in popularity," "offer conversion," and "average boldness."

12. Calibrate with synthetics and consented evaluation, not customer behavior mining. The plan says "Synthetic schedules drive calibration," "do not train on customer histories," "do not use customer telemetry to tune drift," and "customer monitoring never reports private bird behavior."

13. Make age, not engagement or payment, the progression mechanism. Adoption "depends only on aviary age," declining or ignoring an adoption "has no effect," and "Do not accelerate adoption because a user clicks more or pays; there is no payment tier."

14. Treat performance, durability, privacy, and accessibility as release gates. The plan says budgets are "release gates," privacy regressions "block rollout," "No accessibility surface deferred to a later release," and launch stops on "budget or identity/sync/privacy failures."

## Per-feature whys

### Product contract and scope

- One canonical aviary per signed-in account: The plan's rationale is continuity: a browser window opens onto "one continuing aviary," and completion requires returning to "the same continuing aviary on another device."

- First ordinary frame contains recognizable birds already in motion: The plan frames this as the normal continuing-scene experience, rejecting a "neutral loading pose," "spinner," "generic skeleton," or reset-style arrival.

- One bird notices an owner returning within one to two seconds: The plan uses this to give a "varied notice without a textual welcome"; the greeting is a bird action, not a "welcome text" or "return toast."

- Attention changes personality across weeks while mood varies within a day: The plan's why is "slow expressive drift across weeks without a stat or penalty" plus within-day aliveness through mood, time, and behavior.

- Leaving causes no penalty: The plan directly rejects "guilt," "distrust/neglect punishment," "distress state," and "notebook observations about the user's attendance."

- Naturalist narration, captions, keyboard access, contrast, and reduced motion at launch: The plan says the same product must remain "specific and alive" in these modes and that accessibility failures block launch.

- Operational telemetry restricted to performance, availability, and anonymous aggregate health: The rationale is a privacy boundary; the plan says private interactions remain private and absence of behavior analytics is "an architectural requirement."

- Do-not-build list covering payments, streaks, badges, chat, rankings, shared cursors, and engagement campaigns: The plan's rationale is to avoid gamification, guilt, social pressure, announcements, and engagement mechanics around the aviary.

### Decisions where the PRD needs interpretation

- Export omits numeric personality vectors: The plan says "numbers are never exposed to the user" and this "favors the stronger product prohibition."

- Visit-start email only when the host enables the setting: The plan treats it as a "specifically named social exception" and says it must not establish "a general notification service."

- Four top-bar icons with settle inside the offer panel: The rationale is to keep "settle in the top bar without a fifth permanent icon."

- Captions and keyboard focus outlines inside the scene: The plan calls them "intentional accessibility exceptions" to the no-text/no-chrome scene rule.

- Bird focus starts gentle listen-in, with Enter/Escape behavior: The plan reconciles focus-based listen-in and explicit keyboard control while preventing an explicit toggle-off from immediately restarting.

- One account-level IANA timezone: The rationale is that "Every device and visitor sees that timezone's day" and browser timezone changes must not "silently fork or retime the aviary."

- Age-based adoption at 90/180/270/365/540 days: The plan makes adoption a "quiet item" with "no badges, countdowns, or achievement wording," and declining or ignoring it "has no effect."

- Five-minute activity window, 15-second pings, and 180-second per-bird offer cooldown: The plan calls these "versioned calibration constants, not public meters."

- Mute preference excluded from personality drift: The rationale is to avoid decreasing traits, rewarding unmuting, or penalizing "hearing differences."

- Immediate immutable reaction plan plus later tick materialization: The plan's rationale is prompt response without any "client-authored mood or personality write" and without a "second interpretation at tick time."

### Architecture and responsibilities

- TypeScript web application: NOT RECOVERABLE FROM PLAN

- Small server-rendered entry shell and inline first-frame scene: The plan ties this to first-frame performance and birds appearing before panel hydration.

- Dedicated imperative SVG scene controller: The plan connects the critical scene to hydration independence, frame budget, and avoiding per-frame component tree updates.

- Preact DOM panels: NOT RECOVERABLE FROM PLAN

- Node.js HTTP service and Vite asset build: NOT RECOVERABLE FROM PLAN

- PostgreSQL for identity, canonical aviaries, ordered events, notebook, and visits: The rationale is transactional ordering, row locks, unique constraints, export/deletion traversal, and one writer per aviary.

- API/web process plus horizontally scalable tick/job worker: The plan needs independent minute ticks, jobs, and scaling by worker/database partitioning while keeping one writer per aviary.

- Edge gateway with public immutable assets and authenticated bootstrap forwarding: The rationale is fast first-frame delivery without putting authenticated state into shared CDN cache.

- Encrypted object storage for exports and backups: The rationale is temporary private exports, private backups, and deletion-aware restore/crypto-erasure.

- No distributed event bus, analytics warehouse, or client-side simulation framework in v1: The plan keeps v1 bounded, server-authoritative, and free of behavior analytics.

- Versioned public/private contract boundary: The plan uses it to keep public API types from containing "private trait fields" and to support engine/version migrations.

- Simulation boundary separate from browser, audio, and telemetry: The rationale is that trait deltas, mood tables, action planning, and seeded randomness are domain logic, not client timers or analytics.

- Identity boundary separate from bird behavior and analytics: The plan uses this to keep encrypted emails, tokens, sessions, permissions, and deletion away from simulation and metrics.

- Tick worker as sole persistent personality/mood writer: The rationale is to prevent direct personality mutation, double application, and client/server divergence.

- Client scene renders projections only: The plan says browsers get "a projection sufficient for rendering, not a copy of the personality model."

- Client audio synthesizes authorized call descriptors only: The rationale is that the client must not generate behavioral calls from "a local personality model."

- Server visits separate from owner presence and shared interaction state: The plan uses this to keep visits read-only and non-influential.

- Operations limited to aggregate metrics, synthetic tests, and release controls: The rationale is to avoid reads of "interaction payloads or simulation values for analytics."

- Durable materialized state plus bounded accepted-action timeline: The plan uses this to make snapshots consistent and allow immediate accepted plans to fold onto committed state.

- Ten-second polling instead of WebSocket co-presence: The plan says this is "deliberately polling rather than WebSocket co-presence" and later avoids any claim of subsecond shared presence.

- Hidden documents stop drawing, audio scheduling, and routine polls while server ticking continues: The rationale is resource control without freezing continuity.

- Authenticated bootstrap embeds private snapshot plus first-frame SVG: The plan uses this to hit the first-bird target while keeping shared caches away from personalized HTML/snapshots.

- Audio autoplay fallback to silent scene with captions: The plan's rationale is to respect autoplay rules, avoid an audio welcome modal, and keep the complete silent aviary.

- Quiet sky/foliage field when bootstrap misses target: The plan treats this as a "measured performance failure, not the normal experience," avoiding fake placeholders.

- One-time post-adoption empty field and first fly-in: The rationale is that it is the "specified exception" to ongoing-motion entry and must not rerun on returns or other devices.

### Data model and invariants

- UUIDs for stable identity and monotonic integers for aviary order: The plan uses UUIDs for durable identities and monotonic integers so committed append order equals simulation consumption order.

- UTC timestamps plus IANA timezone: The rationale is consistent account-local day, DST safety, and no device-local canonical day model.

- Account-owned rows carry account ownership or traversable foreign keys: The plan uses this for export and deletion.

- Row access policies and separate database roles: The rationale is service authorization and separation among ingestion, simulation writes, auth email decryption, and operations.

- Encrypted account email and keyed lookup digest: The plan's rationale is that plaintext decrypts only for identity/mail operations, and email/digest must not become an operational ID.

- MagicLink token hashing and temporary encrypted destination for new-account challenges: The rationale is single-use bearer safety and purging temporary address material after account creation.

- DeviceSession with hashed opaque cookie secret and no localStorage bearer token: The plan's rationale is secure device sessions that can be revoked and fail closed.

- AccountSettings without personality controls or attendance history: The rationale is hidden personality and no product attendance log.

- Aviary unique account constraint: The rationale is to forbid a second aviary and maintain one canonical aviary.

- Bird stable identity seed, call signature, stored vector, and filter memory: The plan uses these to preserve identity, calls, and personality through rename, migration, retry, and restore.

- Species templates with six silhouettes/palettes and no rarity tiers or rewards: The rationale is recognizable species differentiation without gamified rarity.

- OwnerAttentionSession as bounded private simulation input, not analytics: The plan keeps presence useful to the engine without becoming an analytics session record.

- InteractionEvent with idempotency, assigned sequence, and immutable reaction plan: The rationale is retry safety, ordered consumption, and no rerolling accepted decisions.

- TickCommit retained for consistency and never emitted as behavior analytics: The rationale is operational correctness without behavior metrics.

- ScenePlan short rolling horizon: The plan wants continuity for rendering while avoiding "an unlimited event feed."

- NotebookEntry immutable with no edit/delete API and no archival cutoff: The rationale is a sparse read-only field notebook with indefinite history.

- VisitInvitation, VisitSession, and VisitLog separated from owner sessions: The plan uses separate capability/session types so visits cannot become device sessions, owner attention, or simulation inputs.

- ExportJob expiring encrypted object and one-use download capability: The rationale is private, temporary export delivery.

- OutboxJob with minimal encrypted references and no serialized simulation state: The rationale is that mail queues must not replicate private simulation state.

- Encrypted identity-address store for named visitors: The plan says this is the "necessary narrow extension for named email invites, not permission to replicate plaintext."

- Exactly two starter birds and maximum seven birds: The plan grounds this in product scope, never creating a persistent empty owner aviary and preventing adoption races past seven.

- Nonnegative bounded additive trait deltas: The rationale is no vector decreases, no punishment, and monotonic private personality continuity.

- Atomic tick revision, cursor, trait, mood, notebook, and plan commit: The plan uses this so retry cannot apply drift twice and visible state cannot partially advance.

- No raw interaction payload, notebook prose, bird identifier, trait value, email, or bearer token in metrics/logs: The rationale is privacy-schema containment.

- Name validation, escaping, and rename through account bird settings: The plan supports editable names while preventing markup/control characters and avoiding an in-scene context menu.

### API and authorization contracts

- HTTPS JSON `/api/v1` with idempotency, CSRF, ETags, private no-store, and stable matter-of-fact errors: The rationale is secure owner mutation, cache correctness, versioning, and system copy rather than naturalist error text.

- Magic-link request always returns 202: The plan says this avoids account enumeration details.

- Magic-link consume is atomic and scanner-safe: The rationale is one winner for concurrent replays and no redemption by email security scanners.

- Account settings use `If-Match`: The plan says this prevents overwriting timezone or accessibility settings on another device.

- Device session listing and revocation: The rationale is explicit owner control; revoked sessions clear cookies and fail closed.

- Verified email change before atomic switch: The plan keeps the old address authoritative until success and preserves account UUID and aviary continuity.

- Adoption endpoint with availability token and locked recheck: The rationale is server-owned species/age eligibility and protection against concurrent adoption races.

- Rename endpoint only accepts name and expected version: The rationale is no effect on identity, behavior, vector, or call, with stale edits rejected.

- Snapshot endpoint excludes raw personality, filter state, owner presence totals, and attention history: The rationale is render projection without private model or attendance data.

- Event endpoint bounded batch with explicit accepted/duplicate/rejected results: The plan says not to silently claim all events were accepted and preserves ordered/idempotent command handling.

- Notebook cursor pagination with no write endpoints: The rationale is long-term queryability of a read-only notebook without arbitrary offset scans.

- Export request/status/download with 24-hour one-use capability: The plan uses this for private delivery while omitting raw traits.

- Deletion and recovery endpoints serialized and idempotent: The rationale is recoverable deletion before the 30-day deadline and irreversible hard purge after it.

- Visit invitation explicit recipient email: The rationale is "Never auto-invite contacts" and keep sharing deliberate.

- Invitation and visit log pages without badge or unread count: The plan keeps transparency on demand without engagement chrome.

- Invitation revocation atomically revokes active visits and uses no success toast: The rationale is access removal without notifications or celebratory UI.

- Visit redemption creates visit-only cookie and does not require owner account: The rationale is read-only ambient access without making a visitor into an owner session.

- Visit snapshot checks capability and host lifecycle on every pull: The rationale is next-pull revocation and no cached private stream after access ends.

- Visit end cannot submit simulation events: The rationale is lifecycle duration refinement without visitor influence.

- Owner event kinds limited to return, presence, listen, offer, settle, undo, and attention end: The rationale is bounded simulation input; settle/undo use epochs rather than arbitrary state fields.

- Scene descriptors may contain rendering values but not trait names or hidden vectors: The rationale is that pitch, color, and duration are rendering outputs, not personality statistics.

- Capability separation enforced at routing and database authorization: The plan says hiding buttons is insufficient; visit credentials must receive 403 for owner routes.

### Canonical simulation and immediate responses

- Once-per-60-second ticks spread by aviary UUID hash: The rationale is bounded load regardless of viewers and no closed-tab dependence.

- One writer per aviary through transaction/advisory lock: The plan uses this to prevent double application and maintain serial correctness.

- Deletion-pending accounts continue absence-only ticking until recovery or purge: The rationale is continuity on recovery without invented presence or browser replay.

- Tick reads ordered events through a committed high-watermark and never rebuilds personality from historical log: The rationale is bounded work and stable personality independent of retained raw events.

- Stored reaction plans consumed by tick: The rationale is no rerolling or second interpretation of accepted offers/greetings.

- At most one notebook observation per tick with unique source key: The rationale is sparse entries and retry-safe no-duplication.

- Lag catchup in bounded slices using actual UTC time: The rationale is order preservation without multiplying current presence by an outage duration.

- Command service appends immutable cue plans without changing trait or persistent mood columns: The rationale is responsive presentation without client/server state ownership confusion.

- Idempotency key returns original sequence and plan: The plan uses this for lost-response retry correctness.

- Interruptions append cancellation/transition plans: The rationale is append-only history and explicit cancellation rather than editing old events.

- Provisional local listen-in gain and settle reversal: The rationale is responsiveness, while drift inputs still require server receipt.

- Initial traits around 0.20-0.45: The plan says this avoids uniformly identical starters and initialization near saturation.

- Call signature initialized separately from vocal-frequency trait: The rationale is that increasing vocal frequency cannot replace the recognizable motif.

- Separate deterministic pseudorandom streams: The plan specifies separation by behavior, weather, calls, and observations to keep seeded randomness controlled.

- Private rolling 24-hour durations and capped interaction summaries: The plan calls them engine state, "not user counters."

- Presence-dominant drift drives with capped offer/listen contributions: The rationale is that "interaction-only drive cannot dominate regular presence" and clicking cannot accelerate drift arbitrarily.

- Low-pass memory and additive nonnegative deltas: The plan uses this for slow week-scale change, no visible single-session jump, residual positive tail, and never-decreasing vectors.

- Synthetic calibration fixtures and consented design evaluation: The rationale is to validate perceptual drift without customer histories or customer telemetry.

- Absence quietness handled by recent-attention expression, mood, and time of day: The rationale is no decreased trait, dulled feathers, learned distrust, distress state, or guilt observation.

- Trait-to-behavior curves instead of public meters: The plan says this keeps identity consistent while showing subtle perceptual change.

- Five mood states with sleeping/settled as expressions: The rationale is to avoid harmful need states while still varying posture/environment.

- Account-clock day/night curves and synthetic weather: The rationale is no location service, host timezone parity, and DST-safe device consistency.

- Daily relaxation window and dwell limits: The plan uses these to avoid reopening resets and noisy state flipping.

- Rain and wind as rare deterministic environmental events: The rationale is ambient expression that modifies calls/mood but not traits, without severe weather or user selectors.

- Bounded bird-to-bird influence with one response hop and refractory window: The rationale is to prevent alarm/chorus feedback cascades while preserving differentiation.

- Server-created action spans and rolling 120-second horizon: The rationale is mid-action joins, continuity, bounded planning, and no client choice of behavior.

- Procedural species call grammars and deterministic client synthesis: The plan uses this for recognizable identity, variation without recordings, and same owner/visitor audibility.

### Presence, session behavior, and interactions

- Owner-only presence controller outside render loop: The rationale is precise attention accounting independent of scene animation.

- Presence requires visible document, focus, and recent trusted pointer/key activity: The plan says open tab, focus, audio playback, timer, or navigation alone must not earn attention.

- Fifteen-second intervals with server bounds, validation, and union across devices: The rationale is to prevent replay/inflation and avoid double-counting overlapping devices.

- No pointer coordinates, key contents, keystroke logging, or raw activity stream leaves the browser: The rationale is privacy while still bounding forged attention.

- Tab leader with BroadcastChannel where possible: The plan says server interval union remains the correctness mechanism; the tab lock is only a lightweight reduction.

- Accessible keyboard controls for qualifying activity: The rationale is not to exclude keyboard users or ask users to move a pointer for attention credit.

- Return command on fresh visible owner navigation/restoration/wake: The rationale is a noticing bird within the ongoing scene, not an entry animation or textual welcome.

- Greeting selection uses absence, warmth/boldness, and mood: The plan wants repeated greetings to vary within identity and long absence to be quieter, not punitive.

- Listen-in as local bird mix with idempotent start/end events: The rationale is immediate audio/semantic attention while stale selections cannot accumulate forever.

- Offer controls in top-bar panel, with server-selected receiver: The rationale is no drag gift, no in-scene target selection, and bounded non-rewarding observer reactions.

- Offer cooldown and caps: The rationale is that concurrent devices cannot bypass cooldown and offers cannot overwhelm presence or accelerate drift arbitrarily.

- Drowsy/wary birds may ignore offers without negative consequences: The plan keeps offers non-punitive and personality input tied only to actual accepted investigation.

- Settle warms/dims and quiets calls while ending initiating attention: The rationale is a deliberate mood gesture with "zero directional personality input" and no difference from plain close for qualified presence.

- Five-second settle undo with epoch validation: The rationale is accessible reversal without reopening unrelated older settlements or allowing delayed acknowledgments to resurrect state.

- Settlement presentation lease expires without pulls: The rationale is to detect a closed/suspended tab without counting a settled tab as attention.

### Snapshot synchronization and failure handling

- Server record plus committed plans as source of truth for every device: The rationale is no device vector writes, no local ticking mood model, and no late-device overwrite.

- Revision validation and ignoring older responses: The rationale is preventing stale HTTP responses from rolling back scene state.

- Smoothed server/client time offset: The plan says client wall time must not determine absence or drift.

- Hidden resume joins the resulting pose instead of reenacting missed history: The rationale is current continuity rather than replay.

- Ten-second visible pulls and faster pulls after actions/restoration/gaps: The rationale is cross-device visibility within the pull interval without claiming subsecond co-presence.

- Monotonic per-device command sequences and idempotency keys: The rationale is ordered retry, duplicate safety, and no untrusted timestamp reordering.

- Expire unaccepted presence/listen/offers after offline gaps: The rationale is avoiding replayed gestures into a different mood.

- Pending queue in memory only, cleared on sign-out/account change: The rationale is bounded transient command handling without persistent offline history.

- Offline mode samples only published horizon, then stationary breathing/cross-fades: The rationale is no local generation of new calls, moods, weather, or drift.

- Private revision-keyed caches with auth/lifecycle checks before serving: The rationale is that deletion and invite revocation cannot be bypassed by cache or 304.

- Strong reads/tick transactions on the home-region primary: The rationale is one transactional writer domain and no unbounded-lag replica state.

### Frontend scene and rendering pipeline

- Single SVG scene with procedural/SVG bird shapes and semantic overlays: The rationale is compact rendering plus accessible semantics, focus, and captions.

- Six species with distinct silhouettes, features, palettes, posture sets, and stable details: The plan rejects generic birds differentiated only by color.

- Mood expressions as posture/pose choices: The rationale is to avoid labels or status icons in the scene.

- Server-chosen logical zones/slots mapped responsively by the client: The rationale is stable chosen proximity across resize while remaining pixel-safe.

- Seven birds visible at 320x568 and landscape phones, no scroll/pan/zoom/drag: The rationale is a single horizontal aviary view, not vertical browsing or user-positioned birds.

- First-frame SVG server-rendered from the same snapshot used by the controller: The rationale is no hydration reset and no asset delay before showing a bird.

- One requestAnimationFrame loop with seeded micro-motion: The rationale is living motion without repeating fixed cycles, high-frequency shaking, or discontinuous jumps.

- Bounded decorative leaves/feathers/weather ornaments: The rationale is calm decoration with fixed-size pools and no retained references after removal.

- Hidden/route teardown cancels frames, audio, timers, requests, observers, and refs: The rationale is no resource leaks and correct resume.

- Reduced motion as authored still-pose cross-fades: The rationale is respecting system safety while keeping the same canonical scene, identity, mood, calls, captions, and notebook.

- Four-icon top bar with no badge counts and low-chrome fade only when safe: The rationale is calm chrome without making focused controls faint or inaccessible.

- Lazy-loaded focus-managed account/accessibility/notebook/offer panels: The rationale is performance and keyboard/screen-reader usability while the scene continues.

### Audio pipeline and call captions

- One WebAudio graph per aviary view with procedural synthesis: The rationale is allocation-bounded generated calls with no downloaded recordings or recorded-loop fallbacks.

- Resolved call descriptors drive deterministic note events: The plan keeps clients from rolling new calls or choosing chorus participants locally.

- Identity anchored through motif family, pitch band, timbre ratio, and pauses: The rationale is recognizability across mood, drift, and chorus.

- Listen-in gain keeps nonfocused birds audible: The plan says the nonfocused floor is always greater than zero, preserving the aviary rather than soloing one bird completely.

- Autoplay/context failure leaves a complete silent aviary with captions: The rationale is normal capability handling without intrusive modal or generic placeholder.

- Captions generated from the same resolved descriptor as audio: The rationale is factual captions in both audible and silent modes, not fixed motif text.

- Caption collision avoidance, backing, and chorus phrasing: The rationale is readable contrast and not silently dropping all but the selected bird.

- Caption content excluded from narration live queue by default: The rationale is avoiding duplicate screen-reader chatter.

- Release/disconnect audio nodes and fixed-limit pools: The rationale is no clicks, clipping, backlog burst, or memory growth.

### Accessibility and product voice

- Decorative SVG plus one semantic bird control per stable bird ID: The rationale is assistive identification through name/species/listen-in without announcing mood labels, indices, personality numbers, or every frame.

- Narration composer uses server facts/projection: The rationale is factual naturalist prose that cannot contradict the visual cue.

- Idle live-region narration every 30-60 seconds with bounded queue/coalescing: The rationale is calm speech without forced duplicate announcements or backlog.

- Return/offer narration is observational, not system welcome: The rationale is product voice without transaction announcements.

- Keyboard traversal through top bar and birds, with Enter/Escape behavior: The rationale is keyboard parity for listen-in, offers, panels, and settle reversal.

- Bird hit regions, focus outlines, and no persistent hover label: The rationale is touch/keyboard accessibility while preserving no scene chrome except focus.

- Contrast, zoom, and color checks across time/weather/settle states: The rationale is readable controls/captions regardless of scene palette or color perception.

- Manual screen reader, keyboard, touch, and reduced-motion review: The rationale is to confirm the aviary communicates "particular birds and quiet change, not a technical state list."

### Field notebook generation

- Notebook entries generated in the tick from bounded private observation accumulator: The rationale is actual aviary facts, not every API request or session start.

- Candidate observations limited to behavior/environment facts: The plan excludes user attendance statistics, personality deltas, achievements, visitor feed items, and event-feed behavior.

- Private summaries support "first time this week" without replaying indefinite logs: The rationale is factual statements without exposing a visits calendar.

- Sparse cadence with minimum spacing and capped notable events: The rationale is a field notebook, not a per-session feed or volume-driven reward.

- Editorial templates with no external language-model service: The rationale is fact-constrained natural prose and no account data sent out.

- Historical prose immutable on rename while current UI uses current name: The rationale is preserving old observations as written while keeping current references current.

- Indefinite storage with cursor pagination and virtualized scrolling: The rationale is no archival horizon while keeping DOM/page memory bounded.

- Visitor scene excludes notebook contents and browsing control: The rationale is read-only ambient scene parity without exposing private notebook history.

### Account lifecycle, visits, and privacy operations

- Initial magic-link consumption creates one account, one aviary, two server-selected starter birds, and arrival cues together: The rationale is stable starters, retry consistency, and no persistent empty owner aviary.

- No catalog, rarity choice, avatar editor, inventory, or congratulatory modal at account creation: The rationale is avoiding rarity/reward/gamified onboarding.

- Encrypted email, lookup digests, hashed cookies, CSRF/origin protection, and explicit revocation: The rationale is identity privacy and device security.

- Magic links expire after exactly 15 minutes with inert GET landing and safe redirects: The rationale is scanner safety, replay protection, and redirect safety.

- Email change verifies new address while old remains active: The rationale is preserving account UUID and aviary continuity until a safe switch.

- Host-created named invitations absent from onboarding: The rationale is deliberate sharing, not auto-invite or engagement surface.

- Thirty-day unused invite and 24-hour visitor-only session: The plan calls the active-session limit conservative and says it avoids a permanent friend list or automatic future access.

- Visitor receives same scene/audio code with read-only capability flag: The rationale is ambient parity without a beautified visitor mode or host influence.

- Revocation invalidates active visit cookie and replaces scene on next pull: The rationale is no continued private snapshot stream while honoring next-pull semantics.

- Visit log records approximate who/when/duration on demand: The rationale is host transparency isolated from simulation and analytics, without badge/unread pressure.

- Optional visit-start email exactly once: The rationale is a narrow opted-in social exception, not aviary engagement mail.

- Export consistent snapshot via streaming JSON: The rationale is private access to account data without loading an old notebook fully into RAM.

- Export excludes interaction/presence logs, visitor addresses, tokens, seeds, traits, and rendering coefficients: The rationale is privacy and the explicit numeric-vector prohibition.

- Deletion pending for 30 days with recovery restoring same birds/vectors/notebook/settings: The rationale is recoverable lifecycle without silently re-enabling canceled visits.

- Hard purge deletes live data, jobs, caches, diagnostics, and encryption keys: The rationale is irreversible account deletion and no backup resurrection of vectors/relationship.

- Deletion-aware backups and restore drills: The rationale is to restore exact vectors/IDs when allowed and prove hard-deleted accounts cannot recover.

- Owner interaction events serve only the owner's simulation: The rationale is no training, recommendations, popularity, rankings, cohort engagement, or cross-account drift tuning.

- Seven-day raw simulation-event retention after consumption: The rationale is stable personality through compact private state, not historical raw event reconstruction.

- Operational metric schemas disallow account/bird/email/token/name/interaction/vector/notebook/invite fields: The rationale is service health without private behavior dimensions.

- Limited support/security diagnostics with short retention and allowlists: The rationale is concrete error lookup without automatic body serialization or per-bird state leakage.

- Privacy-policy link names allowed aggregate metrics and exclusions: The rationale is direct system transparency rather than naturalist prose.

### Performance budgets and observability

- Device/network/browser support matrix: The rationale is release-gate verification on realistic mobile, laptop, and browser conditions rather than later optimization.

- Initial JS less than 2MB gzipped with tighter internal critical targets: The rationale is first-bird performance and preventing required work from being moved out of accounting.

- First bird visible under 500ms: The rationale is that sky paint or placeholder must not be relabeled as the product experience.

- 60fps idle on old laptop: The rationale is seven-bird/rain/caption/audio smoothness without per-frame component updates or broad filter stacks.

- No client memory growth over 30 minutes: The rationale is bounded caches and no orphan audio nodes, workers, listeners, DOM rows, or retained growth.

- Snapshot size target 5-15KB and p95 under 20KB: The rationale is continuity and accessibility facts within efficient projection size.

- Tick health alarms for compute latency and scheduler lag: The rationale is missed-minute prevention without per-account behavior dimensions.

- Interaction responsiveness targets: The rationale is immediate local listen-in, prompt server-authored cues, ten-second cross-device visibility, and next-minute personality commitment.

- Streamed inline authenticated bird instead of relying on 2MB ceiling: The plan says the 2MB ceiling alone cannot meet the 500ms 4G first-frame target.

- `first-bird-frame` mark plus pixel/screenshot assertions: The rationale is proving an actual bird rendered, not merely DOM insertion, sky, or placeholder.

- Scheduled synthetics from several geographies using isolated synthetic accounts: The rationale is correctness/performance validation without mixing synthetic metrics with real-account relationship analytics.

- Metric cardinality and field checks in collection and CI: The rationale is rejecting sensitive fields before they enter observability.

### Verification and acceptance suite

- Deterministic time and seeded simulation tests: The rationale is reproducibility within a version, safety constraints, identity continuity, and calibration fixtures without freezing every future random sequence.

- Presence tests over visibility/focus/activity combinations and edge cases: The rationale is that "Only the full conjunction earns time" and no pointer/key contents transmit.

- Drift property tests and fixtures: The rationale is bounded nonnegative deltas, no single-session movement, residual positive tail, and no client vector fields.

- Tick/durability fault-injection tests: The rationale is serial equivalence with no duplicated/lost delta and no vector reset.

- Mood/continuity, greeting, interaction, rendering, audio, captions, notebook, auth, visits, privacy, and performance/a11y evidence: The rationale is each named behavioral/correctness risk must be proven before release.

- Engine calibration harness uses synthetic accelerated days/weeks: The rationale is inspecting private vectors in test infrastructure only without changing production clock semantics.

- Audio recognizability evaluation at a launch criterion: The rationale is proving same-bird identity matching across mood, drift, and chorus without exposing trait numbers.

- End-to-end tests with two browser contexts, crashes, request loss, and long-lived accessibility sessions: The rationale is real continuity, durability, and accessibility beyond automated checks.

### Delivery sequence and rollout

- Contracts and feasibility before secondary panels: The rationale is proving schemas, data boundaries, streamed first frame, procedural calls, silent/reduced-motion/narration paths, 500ms feasibility, and identity recognizability first.

- Canonical core before complete owner experience: The rationale is durable identity, ordered events, reaction plans, locked ticks, interval union, additive drift, mood/time/weather, and crash tests before full interaction polish.

- Complete owner experience before lifecycle and sharing: The rationale is two-device continuity, keyboard/screen-reader review, and seven-bird layout while real users still begin at two.

- Lifecycle/quiet sharing before hardening and release: The rationale is authorization, replay, revocation, and privacy-boundary tests before launch.

- No temporary generic graphics or recorded calls as alpha: The rationale is avoiding an early product experience that "later establishes the wrong experience."

- Staged bird-count rollout with internal seven-bird synthetic accounts first: The rationale is verifying mix/layout/runtime budgets before real age-eligible birds.

- Prelaunch adoption flag may defer but never remove birds or present locks/tiers: The rationale is no loss of existing birds and no gamified progression surface.

- Engine version releases through schemas and deterministic fixture comparisons: The rationale is stable identifiers/vectors and additive, idempotent migrations.

- Drift rollback by future coefficients/presentation sensitivity, not trait reset: The rationale is preserving accumulated vectors and identity.

- Operational kill switches for offers/adoptions/invites, audio, or ornaments: The rationale is mitigating incidents without erasing state, disabling accessibility, or turning off canonical ticking.

- Severe tick correctness incident rejects new mutations and replays from last proven cursor: The rationale is preserving accepted events/vectors and keeping stale clients from becoming truth.

- Practical scaling by measured capacity and UUID partitions: The rationale is tick load grows with accounts, but v1 need not build a million-account platform before proving the product.

- No optimization may freeze unconnected accounts or change absence semantics: The rationale is continuity even when no one is viewing.

### Risks, mitigations, and launch blockers

- Drift too fast/slow/dominated by gestures: The rationale for mitigation is to keep presence dominant and block release if single sessions visibly move traits or weeks show no recognizable change.

- Silent personality loss or double application: The rationale is stable IDs, single writer, atomic tick/cursor/delta, and repair from private proven state rather than reset/reseed.

- Canonical base and fast projection divergence: The rationale is immutable reaction decisions, revision tuple, one reducer, and no rerolling accepted cues.

- Inflated presence: The rationale is monotonic activity windows, bounded intervals, server union/expiry, and no credit for open tabs/audio/visitor/missing heartbeats.

- Synthetic, repetitive, or unrecognizable bird sound: The rationale is stable signature anchors, procedural contours, spacing, headroom, and no recordings.

- Accessible experience losing specificity: The rationale is designed cross-fades, fact-constrained prose, grammar captions, calm queue, visible focus, and launch-blocking accessibility failures.

- 500ms target versus private-state latency: The rationale is compact authenticated first-frame streaming without shared-cache leaks or generic placeholder metrics.

- Missed minute ticks or outage drift: The rationale is spread ticks, bounded work, partitioned writer domains, analytic catchup, and no client tick fallback.

- Visit credential changing host behavior or leaking data: The rationale is separate capability/session type, no owner event route, minimal projection, and authorization before cached reads.

- Revocation appearing successful while visitor keeps pulling: The rationale is snapshot status checks, short visible pull interval, no public snapshot cache, and preserved transparency logs.

- Numeric/personality or PII leaks: The rationale is projection allowlists, export conflict decision, encrypted address references, token redaction, and isolated metrics credentials.

- Scene accumulation of audio nodes, notebook rows, or ornaments: The rationale is bounded pools/horizons/page windows and deterministic disposal.

- Timezone/DST and settle race snaps/divergence: The rationale is one explicit account timezone, UTC integration, gentle retiming, and epoch-specific undo/re-engagement.

- Copy slipping toward metrics, guilt, or announcements: The rationale is naturalist fact templates for product surfaces, direct system copy, no attendance templates, and no toast/badge framework.

### Definition of v1 completion

- Newly signed-in owner meets two identifiable birds and returns to the same continuing aviary on another device: The rationale is continuity and identity as the finished product state.

- Owner can watch, listen, offer, settle, and read sparse observations: The rationale is the core calm aviary interaction set without stats or penalty.

- Mood today plus slow drift across weeks without a stat or penalty: The rationale is expressive change, not meters, punishment, or client-owned simulation.

- Named invited visitor sees same scene without influencing it, with revocation at next pull: The rationale is quiet sharing with read-only parity and access control.

- Product remains specific and alive in narration, captioned silence, keyboard use, and reduced-motion cross-fades: The rationale is accessible completeness at v1, not later remediation.

- Account lifecycle/privacy operations and quiet-sharing controls ship with the product: The rationale is no launch depending on later privacy, sharing, accessibility, simulation repair, or gamification removal.
