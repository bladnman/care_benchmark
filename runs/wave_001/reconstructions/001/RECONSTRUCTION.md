## System-level intent

- Durable private continuity. The plan repeatedly centers "persistent identities," "one canonical aviary per account," and bird UUIDs, personality, and call identity that "survive renames, deployments and migrations." It shows up in product invariants, the Bird record's "immutable identity seed," recovery that resumes "same birds and vectors," and rollback rules with "No rollback that silently replaces birds."

- An ongoing place, not an app starting. The plan says the "first meaningful frame shows an ongoing place," a return is noticed by a bird, and the system "never welcomes the user with a banner." The same intent appears in mid-action bootstrap rendering, "no spinner, fade-from-static or welcome," in-progress poses on return, and performance guidance that rejects "invented default birds."

- Slow, nonpunitive attention. Birds "slowly become more expressive through attention," but absence "cannot lower a trait, harm a bird or produce a guilt surface." This appears in nonnegative deltas, capped/saturated presence, no click-count reward, no muted-audio penalty, no missing-goodbye notice, and "Do not accelerate adoption for frequent attendance."

- Server-authored canonical life. The browser renders, interpolates, presents accessibility, and synthesizes calls, but it "cannot decide canonical perch moves, mood changes, weather, greeting selection, acceptance of offers or drift." The server engine owns ticks, immediate event planning, mood timers, drift, and canonical timelines so a "phone/laptop comparison should receive identical canonical moods and weather."

- Hidden inner state stays hidden. The plan prohibits exposing hidden personality numbers in API payloads, DOM, ARIA, logs, debug controls, exports, or operational telemetry. It repeats this in qualitative export descriptions, snapshot projection allowlists, "never ARIA personality numbers," and "No numerical traits in output contract snapshots or exports."

- Quiet product voice and no engagement machinery. The plan excludes rankings, badges, streaks, visit-frequency surfaces, notification engagement loops, and cross-account analytics. It asks for "matter-of-fact voice," "quiet disabled menu action," "no unsolicited notice," "no textual welcome," and "naturalist voice" for observation surfaces.

- Accessibility is a first-class deliverable, not a later adaptation. The plan says visual, audio, narration, caption, and reduced-motion experiences "ship together" and Stage A must not "postpone accessible architecture until art completion." It requires semantic scene descriptions, captions, narration, reduced motion, manual assistive testing, and asks whether it "feels like observing birds."

- Procedural recognizability instead of recordings or canned loops. Calls are "always procedural"; no recorded files, loop playback, or recordings as workaround. Seeded motifs, immutable signature anchors, varied envelopes, and "not cycling three canned animations" carry the intent that birds remain individually recognizable without becoming static assets.

- Explicit, bounded social access. Visits are "explicitly invited read-only visits," with no public discovery, profiles, chat, comments, rankings, shared ownership, or social cursors. Visitor routes cannot create presence or events, and visitor surfaces exclude notebook and private settings.

- Privacy and lifecycle trust over convenience. The plan treats interaction logs as "simulation inputs, not a warehouse," isolates telemetry from simulation tables, scopes invitee PII, destroys account keys on hard deletion, and says not to claim privacy compliance "merely because SQL rows were removed."

- Performance as release gate, not polish. The plan says PRD targets are "release gates, not aspirations," measures "first bird at actual renderer draw," and prefers missing a performance target temporarily to leaking aviaries. It connects first-frame product intent to compact snapshots, private edge bootstrap, bundle budgets, frame budgets, and memory soaks.

- Rollout preserves relationship state. The release and rollback language protects actual aviaries: feature flags cannot remove adopted birds, rollback cannot change IDs or reset vectors, calibration may pause new positive deltas but "never decrement prior drift," and existing owners keep their actual aviary during rollback.

## Per-feature whys

### Outcome, scope, and product invariants

- Browser aviary with persistent bird identities: The plan's rationale is relationship continuity: birds "slowly become more expressive through attention," while UUIDs, persisted personality, and call identity survive renames, deployments, migrations, login, and cache misses.

- First meaningful frame as an ongoing place with return noticed by a bird: The plan says this makes the aviary "an ongoing place, not an app starting" and avoids a system banner; later sections reinforce this with in-progress poses, mid-action bootstrap, and no welcome/loading theatrics.

- Visual, audio, narration, caption, and reduced-motion experiences shipping together: The plan treats these as equally considered modes; accessibility cannot be postponed, and muted/reduced-motion/narrated experiences must "remain alive."

- Email magic-link accounts: NOT RECOVERABLE FROM PLAN

- One canonical aviary per account: The rationale is canonical continuity across devices and visitors; the account timezone, server revisions, and source-of-truth state prevent a visitor and phone from seeing different day/night or mood states.

- Two system-selected starter birds from six species: NOT RECOVERABLE FROM PLAN

- User naming and renaming: NOT RECOVERABLE FROM PLAN

- Age-based optional adoption up to seven birds: The plan says time-only eligibility "persists until accepted," costs nothing to decline or ignore, and reaches more birds "without teaching attention-for-rewards." The cap and threshold versioning preserve existing birds.

- Three perch zones: NOT RECOVERABLE FROM PLAN

- Local-time lighting: The rationale is a canonical day/night state using one account IANA timezone; this avoids "automatic timezone races" and prevents each browser or visitor from independently overriding the aviary's state.

- Quiet occasional weather: The plan frames weather as ambient and non-demanding: short rains and gentle wind, "No severe weather or attention demands," and no changes to vocal personality.

- Mood-shaped motion: Mood changes "generate gentle pose/perch trajectories, not numerical labels." The rationale is expressive state that appears in pose and timing while hidden mood codes and personality numbers stay out of user surfaces.

- Procedural calls and bird-to-bird responses: The rationale is recognizable living variation without recordings: immutable signature anchors remain, mood changes tempo/intensity, and responses/chorus are scheduled with bounds to avoid constant calling or alarm cascades.

- Listen-in: The plan gives a focused bird gradual gain while other birds remain audible ambient, tracks duration only during eligible presence, and keeps audio-off/caption-only users receiving meaningful focus and bird attention behavior.

- Offers: The plan treats offer as a quiet aviary gesture. Server-selected reactions, cooldowns, and tick-delayed trait effects keep it from becoming a click-count reward while still allowing approach, watch, bathe, drink, respond, or ignore behavior.

- Settle: The plan's rationale is to close the requesting stream's attention/listen-in windows, quiet that client mix, and set a session-scoped lighting override without ending another active device's presence or creating a missing-goodbye notice.

- Sparse read-only notebook with indefinite browsing: The rationale is naturalist continuity from "noteworthy canonical facts," not session timestamps or visit streaks. Old observations remain available, immutable, and coherent after rename because names are stored as observed.

- Account export: The rationale is user access to bird identities, names, species, current mood, notebook, and settings while honoring the stronger prohibition on numerical personality vectors and secrets.

- Account deletion and recovery: The plan explains this as lifecycle trust: soft deletion stops simulation and revokes access, recovery restores the "same IDs/vectors," and hard deletion purges relational state and destroys the per-account key.

- Device revocation: The rationale is active account security: revocation immediately blocks authenticated requests, sessions are centrally checked, and current-device sign-out is supported.

- Multi-device state consumption: The rationale is one canonical state with unioned attention, so two owner devices cannot double the daily dose and phone/laptop views receive identical canonical moods and weather.

- Explicitly invited read-only visits: The rationale is quiet sharing without public discovery or cross-account interaction loops. Visitors use authorized read endpoints, cannot mutate, cannot create presence or events, and are excluded from private owner surfaces.

- Excluded public discovery, profiles, chat, rankings, badges, streaks, notification loops, hunger, death, distress, and meters: The plan's rationale is to avoid leaderboards, recommendation features, guilt surfaces, engagement loops, and product language that contradicts observing birds.

### Decisions where the plan needs interpretation

- Export with qualitative descriptions rather than hidden vectors: The plan explicitly chooses the "stronger unconditional prohibition" on numerical personality exposure, including opaque downloadable files, while allowing naturalist qualitative descriptions.

- Settings-only visit summary: The rationale is resolving visit notifications against "no notification surface." It is off by default, on demand, visible only in settings, and sends no badge, toast, push, email, or unsolicited host notice.

- Four top-bar icon groups with settle inside the offer menu: The plan says this satisfies "top-bar reachability without a fifth icon" and keeps controls out of the scene, with captions and focus outlines as accessibility exceptions.

- Keyboard focus and listen-in behavior: The rationale is meeting the explicit keyboard specification while preventing focus alone from counting as meaningful listen-in duration; Enter engages, focus changes disengage, and no hover behavior is added.

- Fast response through the same serialized server engine: The plan says a minute tick advances long-lived simulation, but delaying greeting or offer by a minute "would fail the product," so immediate server-authored timelines are created without client simulation or personality writes.

### Architecture and ownership

- TypeScript web client and Node.js service: NOT RECOVERABLE FROM PLAN

- PostgreSQL as source of truth with a durable minute scheduler/worker: The rationale is durable canonical state, ordered ticks, persisted RNG/vector/cursor state, and recovery without regenerating or rebuilding birds from event history.

- One deployable API with separate worker processes: NOT RECOVERABLE FROM PLAN

- Versioned schema validators and presentation timeline types: The rationale is shared contracts across server, browser, Canvas, reduced-motion poses, captions, and narration so projections are coherent and schema-validated.

- Server-only package for engine and protected personality types: The rationale is that only the simulation role writes personality and hidden vectors do not leak into clients, exports, DOM, ARIA, logs, or telemetry.

- SQL migrations and explicit DB roles: The plan uses them to separate event ingestion, simulation mutation, account lifecycle, and snapshot reads, reducing accidental writes or reads across protected boundaries.

- Browser rendering/interpolation/accessibility/call synthesis ownership: The rationale is to keep the browser responsible for presentation and transient UI while preventing it from deciding canonical state.

- Client-only leaves, feathers, and subtle parallax: The plan says these are "ornaments without persistent state or simulation input," so visual richness cannot affect drift or canonical behavior.

- Small Canvas 2D renderer with semantic HTML accessibility layer: The rationale is compact rendering, accessible controls parallel to Canvas, and keeping simulation updates out of the animation-frame hot path.

- Avoiding a large game engine: The rationale is performance and bundle discipline around compact assets, prerendered static layers, and a first-bird target that depends on small critical code.

- Private edge bootstrap and CDN static assets: The rationale is fast first paint with a compact presentation snapshot while never publicly caching account-specific HTML or letting a CDN become a public account-state cache.

### Persistent data model

- Schema/version fields, UTC timestamps, synthetic UUIDs, and account-cascade relationships: The rationale is versioned migration, non-email-derived identifiers, and deletion coverage that is discoverable.

- Account email ciphertext plus keyed lookup digest: The plan confines email ciphertext to the Account record and restricts the digest to authentication lookup so email is not a service identifier or log key.

- Auth challenge token digests and provisional-account UUIDs: The rationale is secure challenge consumption without duplicating email fields in challenges.

- Device session records with coarse display names and no precise fingerprint: The rationale is session management and revocation without invasive fingerprinting.

- Aviary record with state revision, event cursor, RNG state, timezone, weather timeline, recent-attention envelope, and simulation/config version: The rationale is a canonical, recoverable state machine that supports deterministic ticks, weather, timezone, and revisioned snapshots.

- Bird record with stable UUID, immutable identity seed, call-signature parameters, traits, filter state, mood, perch, timelines, and cooldown: The rationale is identity continuity; the plan says "Never recreate from event history."

- Interaction event append-only records: The rationale is bounded retry, integrity, ordering, and tick consumption without client clocks arbitrating state.

- Presence lease/interval records: The rationale is strict qualifying attention, expiry, closed reasons, and unioning across devices so overlapping attention is never counted twice.

- Notebook observation records with observed names and immutable entries: The rationale is coherent old prose after rename, deduplication, generator versioning, and no edit/delete/annotation surface.

- Adoption entitlement records: The rationale is stable age-threshold offers, unique acceptance, transactional bird creation, and cap enforcement.

- Invitation records with scoped invitee PII: The rationale is explicit invitation without turning invitee email into an internal identifier or analytics field.

- Visit session/log records: The rationale is host transparency through approximate duration and status while keeping visitor events and presence out of simulation tables.

- Lifecycle job records: The rationale is trackable export, mail, and deletion jobs with artifact references and retries.

- Trait normalization, filter accumulators, and invariant ledger: The rationale is bounded monotonic drift and double-application detection without copying raw traits into telemetry.

- Snapshot projection allowlist: The rationale is to let a viewer observe bird appearance and behavior without receiving five scalar traits, one-to-one reencodings, or a stat interface.

### HTTP API and command contracts

- Versioned `/v1` JSON contracts with secure cookies, CSRF, origin checks, ownership checks, TLS, digest tokens, no-referrer link pages, POST token consumption, and clean redirects: The rationale is validated contracts and reduced token, history, log, and ownership leakage.

- Magic-link request endpoint with generic acknowledgment, keyed-digest/IP throttles, abuse backoff, and 15-minute expiry: The rationale is to avoid account enumeration and limit abuse without permanent lockout.

- Token consume endpoint with atomic one-time consumption: The rationale is clear concurrent replay behavior: one success and matter-of-fact expired/used responses for the rest.

- Device session listing and revocation endpoint: The rationale is owner visibility and immediate blocking of revoked authenticated requests.

- Email change with pending encrypted address and verification before switching: The rationale is preventing loss of the current verified address, then purging the pending address after the transaction.

- Aviary snapshot endpoint with monotonic revision, server time, ETag, and private/no-store handling: The rationale is fresh, cache-safe state for navigation, resume, and keepalive.

- Owner stream registration endpoint: The rationale is to register an owner stream, produce a return event, and return an authorized greeting timeline while excluding visitor code.

- Aviary events endpoint with bounded batches, event UUIDs, stream sequence, idempotency, revision, cooldown availability, and canonical reaction timelines: The rationale is retry-safe command ingestion where server receipt order supplies state order and no trait, mood, or arbitrary patch can be accepted.

- Bird rename endpoint with normalized plain text and expected name revision: The rationale is concurrent rename safety; stale requests return the current name for explicit retry rather than silent overwrite.

- Adoption availability and accept endpoints: The rationale is private age-based availability, one stable bird per accepted threshold, row locking, and seven-bird cap enforcement.

- Notebook keyset endpoint: The rationale is immutable newest-first browsing, stable pagination, and no edit/delete endpoint.

- Account settings endpoint with expected revision and independent patching: The rationale is preventing invisible lost edits while covering timezone, audio, narration/captions, reduced motion, and settings-only visit summary.

- Invitation endpoints for one email, listing/log, and revocation: The rationale is explicit intentional send, no public index, and no invitation fan-out.

- Visit consume and snapshot endpoints: The rationale is one-time invitation consumption, read-only capability, host timezone projection, revocation checks every request, and no automatic renewal.

- Export endpoint with consistent DB revision, signed link, short-lived artifact, and one-time token: The rationale is a coherent user export without long-lived artifacts or public CDN caching.

- Deletion and recovery endpoints: The rationale is immediate deletion marking with 30-day recovery that resumes same birds and vectors before hard purge.

- Matter-of-fact inline system messages and retry actions: The rationale is avoiding toasts over the aviary and keeping system errors direct.

- Quiet cooldown disabled menu action: The rationale is that cooldown is not punishment or countdown game UI.

### Simulation engine and calibration

- Scheduled 60-second ticks for every existing, non-deleted aviary, including no-client aviaries: The rationale is an ongoing server-side place whose long-lived simulation advances without browser presence.

- Stable UUID hash spreading and per-aviary locks: The rationale is load distribution, ordered ticks, and prevention of double drift or destructive offer/tick/rename interleaving.

- Deterministic seeded random streams separated by behavior, weather, and greeting: The rationale is varied movement and calls where a new ornament does not change unrelated behavior and birds are not just cycling canned animations.

- Worker downtime catch-up in bounded batches: The rationale is recovery without fabricating presence, rebuilding vectors from event history, or resetting mood.

- Drift function with filtered input, additive nonnegative deltas, caps, saturation, and synthetic calibration: The rationale is slow visible expressiveness through attention, not client replacements, negative drift, click-count reward, muted-audio penalty, or real-population analytics.

- Qualifying presence as the majority of daily signal, with listen-in and offers capped separately: The rationale is honest attention calibration where watching beyond the cap may affect mood but cannot accelerate personality indefinitely.

- Recent-attention envelope separate from personality: The rationale is allowing immediate greeting eagerness or chorus participation to fade during absence without lowering baseline vocal trait, color, trust, or curiosity.

- Five moods: NOT RECOVERABLE FROM PLAN

- Mood transitions from time-of-day, weather, recent reactions, and personality: The rationale is gentle pose/perch trajectories rather than numerical labels or login resets.

- Bird-to-bird responses, alarm nudges, and choruses: The rationale is social behavior with bounded propagation, refractory intervals, low response probability, and no alarm cascades or constant calling.

- Occupancy-aware perch slots for seven birds: The rationale is keeping every bird in frame without user-controlled placement.

- Persisted weather seed and schedule: The rationale is quiet weather continuity with rare short rains and gentle wind, not attention demands.

- Owner greeting selection and return action: The rationale is a bird noticing return within 1-2 seconds, with weighted variation to avoid always choosing the same bird and deduplication to avoid focus-flapping welcomes.

- Offer command planning and cooldowns: The rationale is immediate acknowledgment with canonical reaction timelines while personality effects wait for tick processing and cooldowns prevent repeated deltas.

- Song fragments as procedural motif library: The rationale is soft in-product expression without user uploads.

- Settle and reengage rules: The rationale is immediate quieting of the requesting stream, a short undo window, stale undo rejection, and identical drift treatment to close/lease expiry without missing-goodbye copy.

### Presence, sync and disconnect behavior

- Five-minute activity window and three-condition presence gate: The rationale is strict attention calibration with visibility, focus, and recent activity, rather than tab-open duration or broadened eligibility.

- Presence interval submission every 15 seconds with server duration bounds and 30-second lease expiry: The rationale is preventing crashed or sleeping clients from accruing hours and rejecting future timestamps or gross clock discrepancies.

- Best-effort close/hidden beacon: The rationale is to close promptly when possible while never relying on it or backfilling offline/device-sleep spans.

- Concurrent owner-device interval union: The rationale is that a person cannot create twice the daily dose by opening two tabs.

- Listen-in leases ending with presence, focus exit, another bird, or session expiry: The rationale is focused duration only while eligible attention holds.

- Bounded offline retries for discrete commands but no offline presence replay: The rationale is retry safety without performing stale offers/settles hours later or replaying an entire offline session as attention.

- Owner snapshot fetch cadence and hidden-tab behavior: The rationale is fresh state on navigation, resume, recovery, and render gaps while stopping hidden animation frames, ornaments, and call scheduling.

- Visitor snapshot cadence and 15-second freshness limit during outages: The rationale is bounding revocation exposure and preventing expired network responses from animating indefinitely.

- Revision and server-time handling: The rationale is ignoring lower revisions, canceling obsolete timelines, avoiding client prediction of mood/drift, and making stale state visibly reconnect rather than look synced.

### Scene and interaction rendering

- Layered scene with sky, foliage, perches, birds, shadows, and sparse ornaments: The rationale is readable depth and an aviary composition under the thin bar.

- Responsive perch spacing, safe insets, fitted narrow portrait scene, and seven-bird collision-aware slots: The rationale is keeping bird extents, captions, focus rings, and all seven birds visible without panning, scrolling, cropping, or offscreen overflow.

- Six silhouettes, pose families, muted palettes, and signature motif data: The rationale is recognizable species and identity expression through server-selected behavior, visual style, mood, pose, and timing.

- Timeline evaluation at first paint using server time: The rationale is a returning bird already being mid-preen instead of starting at frame zero.

- Soft loading state and clear failed-load error outside the scene: The rationale is avoiding spinner, fade-from-static, welcome, or invented state while still communicating failure.

- Initial adoption empty scene and gentle fly-in: The plan allows this only for true first adoption; subsequent navigation avoids repeating entry effects so continuity is preserved.

- Normal idle movement and front/back travel: The rationale is low-amplitude, varied, autonomous life and depth rather than rapid cursor-following motion.

- Reduced-motion mode: The rationale is removing drift/parallax and flight paths while keeping audio, captions, moods, notebook, and drift fully functional, with cross-fades at pose boundaries and no flash.

- Top bar fade behavior: The rationale is a nearly no-chrome scene that still restores visibility for pointer/key activity and never hides active focus or shrinks hit targets.

- No hover labels or scene icons, with naming/settings in account settings: The rationale is keeping controls out of the scene while preserving accessibility text labels.

- Notebook virtualization and accessible load-more alternatives: The rationale is indefinite browsing without retaining the entire history or leaking handlers/memory.

- Notebook specificity, sparsity, deduplication, and no outside generator: The rationale is entries from real canonical state in a naturalist voice, not generic happiness messages, user-frequency prose, or private interaction processing by outside data systems.

### Procedural audio and captions

- Server-planned call descriptors and client synthesis: The rationale is procedural calls with bird identity, signature anchors, mood and trait expression, and no recorded files or loop playback.

- One AudioContext per tab with bounded voices, limiter, lookahead scheduling, noise reuse, and disconnected finished nodes: The rationale is quality and CPU/memory control without bursts after resume.

- Chorus descriptors with slight variation and gain headroom: The rationale is avoiding identical stacked waveforms and keeping seven signatures distinguishable without fatigue.

- Listen-in gain ramps and ambient reduction: The rationale is smooth focus retargeting where other birds remain audible, never zero, and focus closes on disengage, settle, blur/hidden, or suspension.

- Audio off, captions-only, and mute behavior: The rationale is that users still receive meaningful focus and bird attention behavior, and mute never reduces personality.

- Browser autoplay handling and explicit audio-enable control: The rationale is respecting browser policy while rendering birds and captions immediately, avoiding autoplay modal/toast, delayed audio blasts, or recorded fallback.

- Caption prose from finalized call descriptors: The rationale is captions that describe actual count, contour, texture, pauses, and intended procedural calls rather than fixed species strings.

- Caption placement, collision, density, and separation from narration: The rationale is readable high-contrast captions near sources without overlap and without overloading the narration live region.

### Accessibility and voice implementation

- Semantic scene description and roving-tabindex bird list parallel to Canvas: The rationale is accessible bird labels and interaction instructions without exposing personality numbers or raw mood codes.

- Keyboard order through top-bar groups into birds, arrow movement, Enter listen-in, Escape disengage, and focus trapping/restoration: The rationale is predictable keyboard operation for offers, settle, menus, and bird navigation.

- Visitor semantic descriptions without active owner controls: The rationale is read-only visit access that remains accessible without exposing owner commands.

- Narration from the same presentation snapshot and canonical events: The rationale is coherent lowercase present-tense prose about birds, perches, light, and calls, deduplicated from the same state as the scene.

- One polite live region with latest-state queue and event priority: The rationale is avoiding append-only announcement overload while giving greeting/offer/settle priority after current speech.

- Narration pause/repeat and optional visual transcript: The rationale is accessible control of narration in accessibility settings.

- OS reduced-motion default and cross-device reduced-mode setting: The rationale is safe startup before scene motion and consistent reduced mode across devices.

- WCAG AA copy, focus, caption, and non-text boundaries: The rationale is ensuring interactive state does not rely on bird color or audio alone.

- Template libraries, prohibited-word tests, and editorial review: The rationale is voice as a deliverable, avoiding "welcome back," missing-user duration, session-count entries, and achievement language.

- Manual VoiceOver, NVDA, keyboard-only, captions-only, zoom/reflow, high contrast, and reduced-motion testing: The rationale is judging whether the experience feels like observing birds, not merely whether controls have labels.

### Privacy, security and lifecycle

- Operational telemetry allowlist with a separate collector lacking simulation-table access: The rationale is measuring operations without logging birds, names, traits, command bodies, notebook prose, presence intervals, or account interaction histories.

- Synthetic-only drift dashboards: The rationale is calibration and observability without aggregating real relationship histories.

- Processed raw event retention for seven days and deduplication keys for 30 days: The rationale is bounded retry and integrity handling while treating interaction logs as inputs, not a warehouse.

- Visit logs with invitation identity, approximate duration, and timestamps only: The rationale is host transparency without visitor page interaction recording.

- Expired invitation PII purge after a 90-day host transparency window: The rationale is scoped non-account invitee PII retention, plainly exposed in settings.

- Bounded owner sessions, unused invitation expiry, 24-hour visit capabilities, and per-pull permission checks: The rationale is revocation and deletion enforcement without relying on cached authorization.

- Transactional email provider use only for authorized delivery: The rationale is allowing auth/export/invitation mail while keeping simulation and interaction contents out of the provider.

- Avoidance of third-party analytics SDKs: The rationale is preventing exported artifacts and interaction data from entering external analytics channels.

- Soft deletion: The rationale is to immediately stop simulation/commands, revoke visits, cancel export links/jobs, and suspend ordinary account use while keeping a 30-day recovery path.

- Hard deletion with row lock, purge jobs, diagnostic cleanup, and account-key destruction: The rationale is preventing private records from remaining or reappearing through backups.

- Backup tombstone/key-deletion ledger and restore tests: The rationale is ensuring deleted relationship data cannot reappear from retained backup media.

- Encrypted export artifacts and short-lived links: The rationale is access control, exclusion from public CDN caching, and no export of secrets, raw presence logs, device tokens, or other accounts' PII.

### Performance budgets and observability

- First-bird measurement at actual renderer draw on a mid-tier mobile/4G fixture: The rationale is treating the user-visible bird as the release gate rather than first byte or framework mount.

- Initial JavaScript budget with critical renderer/UI/state bootstrap target under 250KB gzip: The rationale is making the 500ms first-bird target credible, with deeper settings and management lazy-loaded.

- First bird under 500ms using private edge bootstrap, compact snapshot, and minimum species shapes: The rationale is fast actual aviary navigation without waiting for audio initialization or noncritical textures.

- Small snapshot targets: The rationale is bounded payloads with timeline horizon and no full notebook/event history.

- 60fps and no-memory-growth budgets: The rationale is keeping long-running two- and seven-bird scenes, captions, chorus, buffers, nodes, workers, and caches bounded.

- Tick p99 and lag alerts: The rationale is not hiding a huge queue delay behind quick compute time.

- Immediate command response budget: The rationale is greeting start within 2 seconds and no loading theatrics while commands are pending.

- Synthetic browser coverage and RUM without stable account/device/bird dimensions: The rationale is broad timing visibility without inferring which real bird was offered what.

- Unsupported-browser explanation instead of old-browser polyfill weight: The rationale is preserving critical bundle weight while directly explaining supported versions.

- Cold state delivery prototype before complex assets: The rationale is that "a 2MB limit alone cannot produce 500ms over 4G," and if latency misses the plan reduces payload, parse work, and round trips rather than leaking personal state or inventing default birds.

### Implementation sequence and acceptance evidence

- Stage A contracts, engine skeleton, and first-frame proof: The rationale is proving schema/roles, privacy allowlists, virtual time, persisted identities, deterministic tick, immediate event planning, silent fallback, reduced-motion poses, narration, and first-bird performance before art completion.

- Stage B private owner vertical slice: The rationale is proving auth, sessions, adoption/naming, per-owner snapshots/events, strict presence, greetings, listen-in, offers, settle/undo, audio fallback, keyboard flows, and no scene overlays or missing-goodbye story.

- Stage C continuity and account completeness: The rationale is proving drift calibration, mood/timezone/DST/weather, social behavior, sparse notebook, adoption cap, settings, export, email verification, recovery/deletion, revocation, stable identities/vectors, deletion coverage, and no numerical traits in outputs.

- Stage D quiet visits and launch hardening: The rationale is proving explicit invitations, one-time consumption, expiry, 24-hour read-only sessions, revocation, on-demand logs/summaries, visitor permission fuzzing, seven-bird captions, focus contrast, assistive tests, soak, and fault drills.

- Stage E release and ramp: The rationale is staged rollout by operational health and qualitative feedback, "never engagement rates or real drift aggregation," with gates for performance soak, mood boundary, and auth/state/accessibility/timing failures.

- Additional-bird support ramp: The rationale is validating recognizability and performance at each count while never removing adopted birds, changing IDs, resetting vectors, or accelerating adoption for frequent attendance.

- Rollback strategy: The rationale is immutable assets, compatible snapshot contracts, validated worker/schema versions, preserved RNG state and vectors, paused positive deltas if needed, restoration tests, clear failures, and no silent bird replacement.

### Required verification matrix and risks

- Engine property tests: The rationale is proving bounded monotonic traits, zero-presence behavior, residual filter bounds, mood continuity, stable IDs, cooldowns, seven-bird caps, rare weather, and bounded response propagation across virtual time and multiple seeds.

- Presence property tests: The rationale is proving the strict gate under focus/visibility/activity combinations, idle expiry, hide/blur, settle/undo, crashes, suspend/resume, clock skew, stale retry, multiple devices, and visitor exclusion.

- Transaction property tests: The rationale is proving duplicate/out-of-order submissions, concurrent workers, command/tick interleaving, crash/replay, revision reordering, and migrations preserve serial-equivalent vectors and identities.

- Account/social security tests: The rationale is proving token expiry/consumption, CSRF, unauthorized access, revocation, deleted hosts, invite expiry/reuse, visitor injection rejection, export expiry, and deletion including diagnostics/backups.

- Affective/accessibility review: The rationale is proving greetings and calls are varied, no textual welcome appears, silent/reduced/narrated modes remain alive, captions match generated calls, signatures are recognizable, notebook is sparse/specific, and no user-frequency prose appears.

- Performance/privacy gates: The rationale is proving bundle, first bird, memory, hidden resume, WebAudio failure, private cache isolation, telemetry field rejection, analytics role isolation, and day-one alerts without relationship metrics.

- Main risk responses: The plan uses prevention and fallback to preserve the same principles: pause deltas without subtracting personality, fail state reads rather than regenerate birds, prefer missing a performance target to leaking aviaries, never move canonical simulation to clients, and use deletion drills before public ramp.
