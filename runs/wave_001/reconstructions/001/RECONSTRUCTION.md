## System-level intent

- Continuity of individually recognizable birds over weeks is the primary product philosophy. It is stated at the start as birds that "continue living between visits" and an "engineering success criterion" of "continuity of individually recognizable birds over weeks." It recurs in stable bird IDs and vectors surviving "renames and all upgrades," in restore rules that say "never regenerate a missing vector," and in rollout rules that "monotonic continuity remains the rule."

- The aviary is private, account-local, and bounded by data minimization. This appears in "browser-based, private aviary," "one account and one canonical aviary," "private state must never enter a shared CDN cache," "observability receives whitelisted numerical operational measurements," and "no analytics over users' bird interactions."

- Simulation authority belongs to the server, not the client. The plan repeats "server-authored" for visual/audio plans, behavior plans, deltas, and snapshots; the client may "render time-indexed plans" and synthesize audio, but may not write "personality, mood, or placement" or schedule "mood-affecting calls."

- Bird continuity is monotonic and non-punitive. Personality deltas are "nonnegative," input absence does not decrease values, "no term depends negatively on days absent," and the plan excludes "hunger, death, neglect penalties or distress." Longer absence changes "orientation and variation, not trust."

- Quiet attention matters more than engagement mechanics. The plan says "quiet watching is valid," but "repeated clicking" cannot substitute for quiet attention. Presence uses visible/focused/recent trusted activity, unions overlapping devices, clips sleep gaps, and rejects "invasive surveillance or anti-bot engagement mechanics."

- The product voice is calm, naturalist, and non-gamified. The plan excludes "scores, streaks, levels, counters, badges or achievements," rejects "textual welcome" and "guilt," requires "naturalist product language" and "matter-of-fact system language," and uses ordinary system messages such as `Your session timed out. Sign in again to keep watching.`

- Accessibility is a first-class version of the experience, not a degraded fallback. The plan includes "first-class narration, keyboard access, reduced motion, runtime call captions," says "accessibility is not a post-launch follow-up," and warns that an "accessible version loses charm" unless narration grammar, reduced-motion art direction, and real reviews pass.

- Procedural individuality should create recognition without catalogs, rarity, recordings, or runtime language models. Species definitions, signature seeds, bounded grammars, mood/personality variation, and "blind identification" gates all support recognizable individuals while the plan excludes "species catalogs or rarity," "recorded call assets," and "runtime external language model" prose.

- The first screen should feel alive, not like loading software. The plan requires an authenticated snapshot, inline critical SVG/CSS, birds already "mid-preen" with "ambient motion" in the first painted frame, "no spinner," "no fake placeholder birds," and performance gates where "a quiet loading field is not a passing bird."

- Sharing is deliberate, read-only, revocable, and non-participatory. Visitors use "a separate capability and route group," cannot submit owner events or access notebook data, see "the canonical host day/night," and must not become "a second participant." Revocation is checked on every pull and after resume.

- Durability, migration, and rollback must preserve bird identity. The plan calls for point-in-time backups, transactional migrations, checksums, restore drills, compatible grammar/asset versions, and never rolling back "canonical bird data" merely to roll back code.

- Conflicting source statements are resolved by a narrow engineering contract. Section 1 names "explicit engineering choices" for vector export, visit emails, four top-bar icons, captions/focus exceptions, expedited ticks, autoplay, timezone, invitation identity, revoked visit history, and keyboard listen-in so each client does not "independently interpret the conflict."

## Per-feature whys

### 1. Delivery contract and scope

- Browser-based private aviary: The plan's rationale is that birds "continue living between visits" and the product success criterion is recognizable continuity over weeks through "motion, calls, and occasional specific observations."

- One account and one canonical aviary: The plan ties this to a single coherent aviary and cross-device continuity: all devices read "the same canonical revision and mood," and another device cannot silently override the aviary timezone.

- Email magic-link authentication, device sessions, and verified email change: The plan uses these to keep the account private without passwords or SSO, avoid signup enumeration, support device revocation, and verify a new address before an atomic switch.

- Export: The rationale is account portability through a "consistent JSON export," while keeping it an authenticated, downloadable exception rather than a product panel, stats page, import, restore-from-client, or trait editor.

- Deletion and recovery: The plan's reason is lifecycle control without accidental bird loss. Recovery within 30 days preserves "every bird UUID/vector/notebook entry"; hard deletion purges account-owned private data and prevents backup resurrection.

- Two system-selected starter birds and approximately six coherent species: The plan uses two starters for the first everyday experience and six species/signature definitions for recognizability without a catalog or rarity system.

- Age-based opportunities to adopt additional birds, maximum seven: The plan says this matches the "intended pacing," keeps production eligibility independent of engagement or paid tiers, and lets a year-old aviary have six birds while protecting recognizability and layout/audio capacity.

- Stable bird identities, renameable names, persistent personality, daily-ish mood, procedural calls, and bird-to-bird responses: The rationale is continuity of individually recognizable birds; names can change, but UUIDs, vectors, species compatibility, and signature seeds survive.

- Responsive horizontal scene with perch zones, local-time lighting, weather, ongoing idle motion, and sparse top-bar controls: The plan uses this to make a living scene that stays inside a safe area, remains legible from 320 CSS px to wide desktop, and avoids panning, zooming, labels, and chrome inside the scene.

- Return greeting: The plan ties this to prompt, individual recognition on arrival. A primary bird notices by warmth, boldness, mood, and absence length, with compositional variation rather than a repeated entrance or textual welcome.

- Attentive presence: The rationale is to credit "quiet watching" while preventing accidental inflation, sleep credit, replay, or impossible durations.

- Listen-in: The plan makes listen-in a way to attend to one bird while keeping the aviary social; the target rises in the mix, other birds remain audible, and credit only comes from accepted presence overlap.

- Three kinds of offers: The plan's rationale is a small owner gesture that the server interprets through perch, mood, and personality. It has no success score; accepted or nearby eligible offers produce only tiny bounded drift.

- Specific offer library items of seed, song-fragment library, and still pool: NOT RECOVERABLE FROM PLAN

- Optional settle: The plan uses settle as a quieting action that lowers light and calls, terminates the initiating view's presence, adds only a small temporary mood signal, and creates no negative drift or punishment.

- Sparse, read-only, indefinitely browsable field notebook: The plan's rationale is occasional specific observations backed by facts, not a feed, streak, diary of attendance, or editable user annotation surface.

- Private read-only visit invitations and visit log: The plan supports deliberate sharing while preserving privacy and transparency: invitations are individually issued and revocable, visitors are render-only, and the host can see who saw what without live markers or badges.

- Narration, keyboard access, reduced motion, captions, and graceful silence: The plan frames these as first-class access paths so users can follow the same aviary experience when audio, motion, pointer precision, or visual access differ.

- Excluding scores, streaks, counters, badges, achievements, reminders, public profiles, feeds, chat, comments, co-presence, and ranking: The rationale is the calm, non-gamified product voice and the privacy boundary around bird interactions.

- Excluding native apps, payments, passwords or SSO, shared or multiple aviaries, scene customization, placement, panning, and zoom: NOT RECOVERABLE FROM PLAN

- No real-weather integration, runtime language-model generation, recorded call assets, client simulation, personality merge, or analytics over users' bird interactions: The plan ties these exclusions to privacy, procedural consistency, and server-authored continuity.

### Product ambiguities resolved for implementation

- Raw personality vectors only in authenticated downloadable JSON: The plan says the export is the "sole exception" to hidden numerical personality because the source statements cannot both be satisfied literally; all ordinary product, telemetry, debug, narration, and snapshot surfaces keep vectors out.

- Optional visit email setting: The plan implements it narrowly as an off-by-default, quiet email for a newly redeemed invitation so notifications do not become pushes, toasts, badges, reminders, or repeated-visit campaigns.

- Four top-bar icons with settle inside the offer panel: The plan keeps sparse chrome and avoids a fifth permanent icon or scene button while still making settle reachable from the top bar.

- Captions and keyboard focus in a label-free scene: The rationale is an accessibility exception; captions and focus outlines are allowed, but permanent nameplates, mood labels, tooltips, and inline action icons are not.

- Once-per-minute scheduled tick plus expedited invocations of the same simulation function: The plan uses this to get prompt greetings/offers without allowing extra invocations to accelerate drift or create a second writer.

- Audio autoplay fallback: The plan acknowledges browser autoplay limits, draws the real moving scene immediately, and uses silence with captions by default rather than promising impossible autoplay.

- Canonical IANA aviary timezone: The plan favors "one coherent aviary when devices travel"; another device cannot silently override the host day, and visitors use the host timezone.

- Recipient identity records for named invitations: The plan needs addresses for named invitations while keeping email encrypted once in the accounts table and using UUID references in invitations and visit records.

- Revoked invitation history: The rationale is "sharing transparency without a success notification"; active invitations disappear, but historical visits remain with revoked status until account deletion.

- Keyboard focus and Enter both engaging listen-in: The plan reconciles focus-based accessibility with Enter activation by letting focus engage, Enter re-engage, Escape end, and pointer activation of the same bird toggle off.

### 2. Architecture and ownership boundaries

- TypeScript web application with server-rendered HTML, compact SVG bird assets, and a small imperative scene runtime: The plan uses this split to render motion outside component reconciliation and keep the first scene responsive.

- Typed domain package excluding persistence, personality integration, and private event-processing code from client builds: The rationale is to share presentation and call grammar interpretation while keeping simulation authority and private state on the server.

- Stateless HTTPS app service, background simulation workers, PostgreSQL canonical store, identity/email adapter, and isolated export worker: The plan uses these to make PostgreSQL the canonical store and transactional event log while separating export and delivery responsibilities.

- One database write region, stateless edge delivery, and no distributed multi-writer state: The rationale is simpler canonical ordering for account-local logs and bird continuity.

- Public CDN assets with authenticated private snapshots: The plan separates immutable species/code assets from private state so "private state must never enter a shared CDN cache."

- Owner/API/simulation/snapshot/visitor/observability boundary: The plan's why is least authority. Owners submit facts and render plans; APIs authorize and sequence; workers advance state; snapshots strip private inputs; visitors are presentation-only; observability gets no private table access.

### 3. Persistent model and invariants

- UUIDs, UTC instants, independent versioning, constraints, row locks, and narrowly granted service roles: The plan uses these to protect continuity through migrations and to avoid relying on application convention alone.

- Encrypted account email and isolated blind index: The rationale is that email is not an identifier, partition key, log field, or metric, but lookup still works.

- Secure device sessions in HttpOnly SameSite cookies: The plan uses hashed tokens and visible device labels so sessions can be revoked and no bearer token sits in local storage.

- Magic links with hashed random tokens and one atomic consumption: The rationale is safe email sign-in, no scanner redemption on GET, and concurrent replay resistance.

- Bird records with immutable species ID, call-signature seed, personality vector, drift-filter state, mood, perch, and cooldown: The plan uses these fields so identities and vectors survive renames and upgrades, and only adoption can initialize a new vector.

- Simulation role as the only writer for existing bird vectors and mood/action state: The reason is to prevent snapshot/API/client paths from mutating personality or canonical behavior.

- Versioned species definitions, including compatible signatures and one nightjar-like species: The plan ties this to recognizability across releases and meaningful nighttime call probability.

- Interaction events as ordered, append-only, validated facts with no client personality, mood, or placement writes: The rationale is durable event folding without letting the client decide simulation state.

- OwnerView and BehaviorPlan records: The plan uses OwnerView to bound presence/listen-in leases per tab, and BehaviorPlan to carry bounded, server-authored rolling action/call horizons.

- Immutable private NotebookEntry and bounded ObservationSummary: The rationale is fact-supported specific observations with identity continuity, no duplicate entries, and no visit streaks or cross-account statistics.

- Invitation, VisitorSession/VisitLog, ExportJob, and TransactionalOutbox records using hashes, UUID references, private objects, and late address resolution: The plan uses these to avoid plaintext token/email leakage and to keep visitors from generating simulation events.

- Backups, checksums, restore drills, and fail-closed missing vectors: The rationale is that a missing vector must be restored, not regenerated from events or seed, because "initial seeds only create new birds."

### 4. API and account flows

- Same-origin JSON endpoints, CSRF, strict origin checks, rate/body limits, relationship authorization, stable error codes, no global toast, idempotency keys, and unknown-field rejection: The plan uses these to prevent disguised absolute-vector writes, lost edits, and confusing system behavior.

- Magic-link request and consume flows: The plan's rationale is no signup enumeration, fresh requests after the abuse window, atomic consume before cookies, and clear recovery for reused or expired tokens.

- Account settings, sessions, and email change flows: The plan uses revision preconditions to prevent lost settings edits, device revocation to stop subsequent calls, and verified new addresses to avoid unsafe switches.

- Initial adoption endpoint and first-ever quiet-field-to-soft-fly-in sequence: The plan makes activation idempotent, default names non-blocking, and the adoption transition persisted so refresh cannot replay it.

- Snapshot, events, and receipts endpoints: The rationale is to expose the current server-authored presentation, accept/duplicate/reject events deterministically, and reconcile uncertain acknowledged actions without submitting a new logical action.

- Bird rename endpoint: The plan supports international names and escaping while preserving identity; a rename cannot replace a bird.

- Adoption opportunity endpoints: The plan uses age eligibility, opportunity IDs, locks, and the seven-bird cap so adoption is quiet, owner-directed, and not driven by presence or clicks.

- Notebook endpoint with keyset paging and no mutation API: The rationale is indefinite browsing of old entries without turning the notebook into an editable feed.

- Invitation, visit, export, deletion, and recovery endpoints: The plan uses these for explicit sharing, render-only visitor access, host history, authenticated export delivery, and a 30-day deletion recovery path.

- Numeric endpoint defaults such as five magic-link requests per 15 minutes, 30-day rolling sessions with a 90-day absolute limit, and 1-32 grapheme bird names: NOT RECOVERABLE FROM PLAN

### 5. Presence and interaction accounting

- Presence requiring visible document, focused document, and trusted recent pointermove or keyboard-equivalent event: The plan uses this conjunction to count actual attention while excluding timers, polling, focus alone, audio playback, open tabs, or programmatic events.

- Monotonic interval tracking, 15-second submissions, best-effort terminal beacons, server leases, and heartbeat gap caps: The rationale is to avoid all-night credit, sleep credit, replay, impossible durations, and storage of raw key or pointer data.

- Explicit statement that the browser cannot cryptographically prove watching: The plan's why is to prevent accidental inflation without adding "invasive surveillance or anti-bot engagement mechanics."

- Union of overlapping owner presence across devices and capped listen-in credit: The plan ensures two devices do not double-count time and simultaneous listen-in is divided rather than inflated.

- Settle ending presence without negative drift: The rationale is that settle and tab close end attention but should not punish the birds; settle adds only a small temporary mood-quieting signal.

- Hidden and unfocused scene handling: The plan stops animation, polling, captions, and audio scheduling when hidden while letting the server keep ticking; an unfocused visible scene can render with zero presence.

- Listen-in interaction semantics: The plan gives immediate local mix response but records bounded start/end intervals, clips forgotten focus, and credits only accepted presence overlap.

- Offer interaction semantics: The plan lets the server choose nearby eligible receiving birds through perch, mood, and personality, allows ignoring or responding, rejects spam without punishment, and gives no success score.

- Exact three-minute per-bird offer cooldown: NOT RECOVERABLE FROM PLAN

- Settle interaction semantics and re-engage: The plan makes settlement an authoritative quieting transition that can reverse from current lighting, is not canceled by mouse movement or polling, and cannot change drift through local feedback alone.

- Exact five-second re-engage window and six-second settle duration: NOT RECOVERABLE FROM PLAN

- Arrival events: The plan separates arrival/greeting from presence credit, coalesces duplicate or near-simultaneous arrivals, allows quick-return glances, and prevents visitor arrival events.

### 6. Server simulation and behavior design

- Scheduled roughly once-per-minute ticks for every live aviary, including absent accounts: The plan's rationale is that birds keep living without client-triggered catch-up only.

- Leases, fencing tokens, ordered event ranges, one transaction, and exactly-once effects: The plan uses these to make at-least-once worker delivery produce exactly-once state effects and prevent double-application of personality.

- Bounded missed-work recovery with analytical no-input integration: The rationale is fairness between accounts and continuity without replaying missed calls or greetings, fabricating vectors, or ignoring scheduler lag.

- Personality drift as five stored normalized scalars with persisted nonnegative low-pass drive: The plan uses this to make personality the source of truth, produce slow positive change, and keep absence from decreasing trust, warmth, color, or calls.

- Hidden calibration from presence, listen-in, and eligible offers: The plan's why is that steady presence drives most change, listen-in and offers are small bounded signals, and "repeated clicking" cannot substitute for quiet attention.

- Synthetic calibration targets, daily/session caps, and test-harness values: The plan uses these to make week-one change subtle, three-week change perceptible, and one session below a perceptual threshold while showing no targets to users.

- Calibration using deterministic synthetic histories and consented qualitative study, not production vectors or user event histories: The rationale is to tune behavior without mining private bird data.

- Mood model with wary, content, curious, drowsy, and alert states: The plan uses mood to shape posture and tempo, keep mood from resetting to neutral on open/tick, make wary ordinary watchfulness, and allow nightjar-like nighttime calls.

- Behavior selection for preen, scan, tilt, shuffle, rest, perch-transfer, and bounded bird-to-bird effects: The plan uses this to make personality influence spacing, greetings, and responses while occupancy maps and refractory windows prevent collisions, alarm loops, and constant chorus.

- Seeded rain and soft wind: The rationale is occasional weather and small mood/effective-call modifiers without real location request, thunder, storms, distress, or database ornaments.

- Exact initial weather frequency and duration of two to four rains per week lasting three to eight minutes: NOT RECOVERABLE FROM PLAN

- Greetings and response timing: The plan uses weighted selection, persisted per-arrival seeds, continuous pose/call parameters, and optional staggered second responses so a greeting is a specific notice, not all birds in unison or a repeated animation.

- Call grammar runtime: The plan uses species grammars and permanent signature seeds so calls are procedural, bounded, mood-shaped, and still individually separable across same-species birds, seven-bird chorus, night calls, and offered fragments.

- Server-planned call horizon and shared note-event representation: The rationale is that audio and captions describe the same actual structure while the client cannot invent mood-affecting calls or play a backlog after suspension.

- Notebook generation: The plan's why is sparse, fact-supported, private naturalist prose with novelty thresholds and long cooldowns, not one entry per session, absence backlogs, attendance calendars, trait deltas, streaks, or editable annotations.

### 7. Snapshot synchronization and failure recovery

- Presentation snapshots that include current presentation state but not raw personality, filter history, or recent interaction history: The plan uses snapshots to expose renderable life while keeping numerical traits out of ordinary surfaces and avoiding disguised trait fields.

- Owner and visitor polling with ETags, jitter, visibility refreshes, and visitor authorization before any 304: The rationale is freshness without WebSockets or client-to-client channels, and visitor revocation enforcement on every pull.

- Strictly newer revision acceptance, server-clock smoothing, monotonic render clock, and correction blends: The plan prevents late HTTP responses, browser clock changes, or suspended flights from jumping or overwriting the birds.

- Multi-device canonical state: The plan ensures devices may differ in freshness but not in authoritative mood, cooldowns, adoptions, or personality; there is no personality merge dialog.

- Offline/reconnect behavior: The plan keeps only bounded unacknowledged action IDs, retries accepted intents with same IDs, discards old presence and transient plans, and does not replay offers whose visible response is past.

- Failure presentation: The rationale is matter-of-fact system panels outside the scene, temporary last-known scene display only when appropriate, and immediate clearing of private rendering on auth failure, revocation, logout, or account change.

### 8. Frontend scene and startup pipeline

- Authenticated HTML with compact snapshot and inline critical bird SVG/scene CSS: The plan uses this so a bird is already mid-action in the first painted frame and hydration does not replace the scene or restart motion.

- Very small bootstrap and time-indexed renderer before framework hydration: The rationale is that a static silhouette that waits for a large framework is not acceptable.

- Quiet sky/field cold fallback with no spinner, progress indicator, fake birds, or static-to-live fade: The plan uses this to avoid the initial scene reading as loading software while staying honest when state is slow.

- Private/no-store snapshot responses and public immutable static assets: The rationale is to keep private state out of public CDN paths while allowing public species/code assets.

- Layered SVG, bounded ornaments, transforms/opacity, one requestAnimationFrame coordinator, and seeded micro-motion: The plan uses this for performance and lifelike variation without synchronized robotic loops or per-frame DOM work.

- Server-authorized perch interpolation and responsive safe layout: The rationale is to keep birds inside the safe area from small phones to desktop, avoid invented perch decisions, and use letterboxing instead of cropping or scene panning.

- Timezone day/night palette, settle evening blend, and restrained weather layers: The plan uses these to make local time visible without changing timezone, advancing the astronomical day, or adding dramatic effects.

- Four-item top bar fade behavior: The rationale is sparse chrome that stays discoverable and fully accessible when focused, hovered, touched, expanded, or on touch-only devices.

- Keyboard input with roving tabindex, arrows, Enter, Escape, and focus preservation: The plan uses this so bird access works without pointer precision and focused bird identity survives motion, rename, and snapshot updates.

- Exact modified shortcut `Alt+Shift+O`: NOT RECOVERABLE FROM PLAN

### 9. Audio, captions and narration

- WebAudio procedural pipeline with one AudioContext per tab, per-bird gain/panner chains, one-shot oscillators, noise/wavetable reuse, and no recorded fallback: The rationale is procedural calls that stay bounded, recoverable, and consistent with the grammar contract.

- Audio scheduling, chorus density caps, stereo/mono behavior, and loudness comfort: The plan uses these to avoid clicks, clipping, duplicate loops, and repeated volume adjustments.

- Exact scheduler intervals, 100-200ms ahead window, and 24-voice cap: NOT RECOVERABLE FROM PLAN

- Listen-in gain ramps where the target rises and other birds remain audible: The plan's why is focus without hard solo, abrupt channel switch, pops, or reset.

- Mute preference affecting playback only: The plan explicitly says mute has no punishment, no trait decrease, and no assumption that audio-off users were absent.

- Silent mode and captions on audio failure: The rationale is graceful behavior under autoplay denial or device interruption: do not block paint, hot-loop retries, or replay elapsed calls.

- Captions from actual sound structure: The plan uses the shared note-event representation so captions describe the call that sounded or would have sounded, not fixed species text, and collision layout prevents overlap or unreadable text.

- Narration as an authored experience: The plan uses the same presentation snapshot and action IDs as the renderer, a naturalist grammar, semantic aviary region, named bird controls, and one polite atomic live region so access preserves specificity without trait numbers or flat mood dumps.

- Idle and priority narration queue: The rationale is meaningful, concise prose every 30-60 seconds at most, with rapid interactions coalesced and unsolicited narration paused while modal/notebook reading is happening.

- Captions not being live regions: The plan says screen-reader narration already interprets the scene and double-speaking would swamp the experience.

- Reduced motion: The plan treats reduced motion as a separately art-directed path with slow cross-fades, removed parallax/leaf motion, ongoing behavior/calls/mood/drift, and its own visual acceptance review.

- Contrast and assistive-technology testing: The plan uses measured contrast tokens, two-tone focus outlines, caption verification on rain/night, forced colors, keyboard-only, magnification, VoiceOver, and NVDA to make system surfaces accessible.

### 10. Private visits, export and lifecycle

- Deliberate host invitations with 256-bit random tokens, token hashes, HTTPS, no-referrer pages, no third-party scripts, and POST redemption: The rationale is private sharing that email previews cannot spend and request logs cannot leak.

- Initial two-hour visitor session lifetime: NOT RECOVERABLE FROM PLAN

- Visitor authorization, revocation, and cached render lease: The plan enforces revocation on each request, stops displaying private data offline beyond a short lease, and says already-seen bytes cannot be revoked retroactively.

- Visitor read-only same renderer without greeting, offer, listen-in, settle, notebook, settings mutation, focus action, or presence hook: The rationale is that the visitor sees the canonical host aviary but does not become a second participant or change host simulation.

- Silent visit log and optional visit email notification: The plan provides account-setting transparency and at most one explicit notification per redeemed invite, while off remains default and there is no badge.

- Export snapshot: The rationale is a transactionally consistent JSON attachment with birds, IDs, names, vector exception, moods, notebook, settings, schema/version/timezone, and export time, omitting secrets and raw interaction history.

- Short-lived export download link, encrypted private storage, expiry, and object cleanup: The plan uses this to avoid payload fields in logs, memory exhaustion for large notebooks, and long-lived export objects.

- Exact 24-hour export download validity: NOT RECOVERABLE FROM PLAN

- Deletion and recovery: The plan uses immediate deletion marking, session/visit/job revocation, restricted recovery, preservation of birds during the recovery window, hard-deletion of account-owned records and keys, and backup deletion manifests to avoid resurrection.

### 11. Privacy and observability boundaries

- Interaction event retention only for that user's simulation and bounded observations: The plan retains processed raw events for seven days for retry/failure recovery, then removes them because vectors and filter state are the source of truth.

- Telemetry SDK with enumerated numerical buckets and explicit exclusions: The rationale is operational health without event kind, offer item, bird name, species, trait, mood, presence duration, URLs, emails, tokens, payloads, persistent IDs, or full IP retention.

- Restricted account-level service-error logs and infrastructure scrubbing: The plan permits short-lived UUID/error-code diagnostics but forbids serialized simulation state, email-bearing request bodies, cookies, token paths, and copied per-bird data to mail providers.

- No warehouse connector, analytics pipeline, recommendation pipeline, third party, or model training access to per-account state: The plan keeps detailed diagnostics limited to synthetic, explicitly labeled calibration/performance accounts.

### 12. Performance budgets and operational gates

- Critical JS and first-paint budgets: The plan's rationale is reserve margin below the PRD limit, lazy-loading noncritical surfaces while preserving essential semantics/motion preference detection.

- First bird under 500ms and measured as a real bird element from authenticated snapshot: The plan says a quiet loading field is not a passing bird and DOM insertion alone is not enough.

- Snapshot size targets: The rationale is compact projection across two to seven birds while referencing public assets instead of embedding them repeatedly.

- Greeting, idle frame rate, memory, audio, tick, and UI/accessibility budgets: The plan uses these gates so visual notice, 30-minute frame/memory/audio behavior, scheduler lag, and accessibility basics pass before release.

- Cold/warm benchmark fixtures and common geographies: The plan requires reporting worse profiles and fixing payload/edge latency rather than hiding failure behind warm-cache-only results.

- Memory CI lifecycle checks: The rationale is to prevent retained listener, timer, audio context, node, scene reference, ornament, or notebook page growth over long sessions.

- Browser support as the last two major releases of Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN

### 13. Verification matrix

- Verification protecting "causal behavior and qualitative character": The plan requires deterministic clocks/seeds, fake email adapter, real browsers, and separate human sensory checks because method outputs alone are not enough.

- Personality, calibration, presence, tick, multi-device, greeting, audio/caption, accessible charm, scene, notebook, auth/social, privacy/lifecycle, adoption, product voice, and performance evidence: The plan ties each area to the specific failure it guards against, including negative drift, click spam, sleep gaps, stale snapshots, hard mutes, inaccessible charm, feed-like notebooks, visitor mutation, resurrection, catalog/rarity, and gamification.

- Sound-design and accessibility review with real listening/reading/motion experiences: The plan states that no synthetic automated test can prove audio is pleasant or a bird feels alive.

### 14. Implementation sequence and rollout

- Milestone A contracts and two-bird vertical slice: The rationale is to settle ambiguity decisions, prove a closed browser's aviary advances on the server, and measure first-frame/audio-start feasibility before expanding features.

- Milestone B identity, durability, and interaction correctness: The plan sequences auth, sessions, adoption, presence, deduplication, concurrent clients, backup/restore, metrics, export scaffolding, and deletion/recovery before storing long-lived participant accounts.

- Milestone C character and accessible experience: The plan waits for species, signatures, mood-shaped motion, chorus/responses, lighting, weather, narration, notebook, drift calibration, and qualitative reviews so day-1/day-21 contrast is perceptible without numerical UI.

- Milestone D visits and full release qualification: The rationale is that read-only sharing, revocation, expiry, visit history, notification setting, PII/security tests, browsers, timezone, errors, CDN/privacy, soaks, and load/restore must pass together; accessibility is not a follow-up.

- Milestone E staged release and bird growth: The plan ramps from synthetic accounts to invited owners to account-UUID cohorts using only operational aggregates, not retention funnels, visit streaks, offer frequency, popular species, or average production drift.

- Exact staged rollout percentages and 48-hour observation windows: NOT RECOVERABLE FROM PLAN

- Calendar-age adoption schedule at 90, 180, 270, 365, and 540 days: The plan says this produces the intended pacing, lets a year-old aviary have six birds, keeps opportunities quiet, and never bases eligibility on engagement or paid tier.

- Rendering/mixer support staged at two, then three/four, then five/seven birds: The rationale is to validate capacity before calendar cohorts become eligible while feature flags may pause new adoption but never delete, hide, or downgrade adopted birds.

- Rollback and migration behavior: The plan preserves compatible readers/assets, uses expand/contract schema changes, never rolls back canonical bird data just to roll back code, disables new positive drift before repair if needed, and treats negative drift as a critical continuity incident.

### 15. Principal risks and response

- Drift too fast, too slow, or dominated by clicking: The plan responds with synthetic longitudinal matrix, bounds, monotonic property tests, and blinded review so tuning happens before broad release and not from population bird data.

- Loss or double-application of personality: The plan responds with fenced transactions, atomic cursors, restricted writers, idempotent deltas, backup checksums, restore drills, and repair instead of reseeding.

- Presence inflation or exclusion: The plan responds with the conjunction truth table, interval union, sleep-gap clipping, trusted events, and touch/assistive input review to preserve stationary watching without overnight credit.

- Calls sounding synthetic, repetitive, or indistinguishable: The plan responds with signature design before expansion, shared grammar, bounded chorus overlap, seven-bird blind listening, mono tests, and preserved signature seeds.

- Stale snapshots, action lag, or clock jumps: The plan responds with revision gating, expedited ticks, measured offsets, bounded horizon, visibility refresh, no stale action replay, and a distinction between freshness and client-side authority.

- Initial scene reading as loading software: The plan responds with inline authenticated snapshot, critical SVG, phase-correct motion before hydration, aggressive critical bytes, and cold-path timing.

- Accessible version losing charm: The plan responds with authored narration grammar, reduced-motion art direction, real screen-reader and motion-sensitive reviews, runtime caption equivalence, and release blocking.

- Email, tokens, or private state leaking through infrastructure: The plan responds with UUID identity, encryption, redaction, outbox references, private cache policy, negative telemetry tests, and no analytics database reader.

- Visitor becoming a second participant: The plan responds with separate routes/capability, separate client composition, no owner event/greeting hook, before/after host-state tests, and revocation on every pull and hidden return.

- All-account ticking cost: The plan responds with staggered/sharded due work, batch reads, bounded horizons, analytical no-input integration, load tests, and preserving scheduled advancement for absent aviaries.

- Existing birds changing identity after art/audio upgrade: The plan responds with immutable IDs/signature seeds, compatible species versions, migration fixtures, before/after listening review, and no regenerate/reset migration.

- Product growing gamification or announcing UI: The plan responds with no general toast/badge/streak infrastructure, product voice review, explicit negative acceptance cases, and operational health goals rather than engagement targets.

- Literal source conflicts causing inconsistent implementations: The plan responds by keeping section 1 decisions in the engineering contract and building one bounded behavior instead of letting each client interpret conflicts independently.
