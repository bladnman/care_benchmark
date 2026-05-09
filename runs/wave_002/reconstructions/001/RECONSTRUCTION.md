## System-level intent

- Affective durability over obvious shortcuts: The opening says the product's value is "largely affective" and names "felt-aliveness, naturalist voice, restraint" as its center. It warns that shortcuts like "recorded audio fallbacks, last-write-wins sync, ARIA-label automation, an entry spinner" would "destroy the product's center," so the implementation makes those shortcuts "unreachable, not just discouraged."

- Non-goals as architectural absences: The plan does not merely say no gamification, streaks, social feed, or Tamagotchi mechanics; it says "The non-goals are honored in code, not just in policy." This shows up in schema absences like no `visits_count`, no `daily_login_streak`, no account-level visit frequency, no analytics endpoint, and no reset-bird path.

- Server-canonical relationship state: Across the architecture, data model, sync model, and risks, the plan insists that the "server is the only writer" of personality, mood, perch, schedule, and notebook state. The client emits events and renders snapshots; it "never sends absolute state." This protects coherent drift across devices and time.

- Hidden interiority, visible only through expression: The plan repeatedly protects personality from numeric exposure. Snapshots "do not contain numeric personality vector values," render params are "squashed and noise-padded," and raw personality numbers are not in client-write paths. The bird's interior state is meant to be felt through mood, calls, motion, plumage, and notebook prose, not inspected.

- Slow, monotonic aliveness: Personality drift is "slow," "driven primarily by presence and listen-in," and "monotonic toward expressive." The plan rejects decay, death, hunger, and neglect harm; birds become more expressive rather than punishing absence. This appears in the drift function, calibration targets, tests, and Tamagotchi non-goal.

- Naturalist voice split from system voice: The plan keeps "naturalist voice" for aviary, notebook, narration, and captions, while "matter-of-fact voice" belongs to auth, settings, errors, unsupported browser, deletion, and export email. The code organization into `voice/naturalist/` and `voice/matter_of_fact/` and forbidden-phrase lints make voice continuity structural.

- Procedural life rather than canned theater: The audio section calls the audio pipeline the "affective spine" and says "if it sounds canned, the product reads as theater." This is why recorded-audio fallback is a hard no, calls are procedural, seeds are deterministic, and motion is parametric rather than looped key frames.

- Accessibility as part of the affective core: The plan says "Accessibility is a designed surface, not a checklist" and repeats that reduced motion is "not a stripped fallback." Narration, captions, keyboard access, focus indicators, and contrast are framed as ways to keep the same aviary experience available, not as separate compliance layers.

- Health observability without engagement operation: The plan measures errors, performance, audio-context failures, and tick latency, but explicitly says it does not measure DAU, MAU, retention, churn, or visit frequency. "We monitor health (errors, perf), not engagement. The product's success is felt; it does not have a dashboard."

- Privacy by topology and lint, not trust: Per-bird simulation data "never reaches analytics or training pipelines"; the analytics warehouse is on a separate network; analytics roles cannot read simulation tables; static metric schemas and forbidden-field lints block leakage. This is repeated in scope, architecture, telemetry, observability, and risks.

- Calibration as product stewardship: The plan treats drift weights, mood probabilities, weather rates, notebook sparsity, and call motif libraries as calibrated product material. Alpha and beta are for "validate aliveness," "calibration data," sound design review, accessibility review, and privacy-domain audits; after public v1, drift weights are held constant unless reviewed.

- Relationship durability and identity continuity: The plan treats bird identity and personality as precious state. `bird_id` is "never reused, never reset, never regenerated, never swapped"; migrations must preserve `bird_id` and the personality vector; personality-vector loss is "catastrophic, silent" because it destroys the user's relationship with their birds.

## Per-feature whys

- Scope / single-user accounts and magic-link sign-in: The plan grounds this in a narrow v1 account model and secure, simple auth. Magic links expire in 15 minutes, are single-use, use uniform responses to "avoid enumeration," and are rate-limited per email and IP.

- Scope / per-device session tokens with revocation surface: Device sessions are listed and revocable so a user can see and end sessions. The security section reinforces this with HttpOnly, SameSite, Secure cookies and abnormal session creation alarms.

- Scope / single canonical aviary per account: The sync model says this makes sync almost a misnomer: "there is nothing to sync because there is one record." It prevents client-vs-client conflicts and lets every device see the same mood and drift history.

- Scope / two starter birds at adoption, system-chosen species, user-named: NOT RECOVERABLE FROM PLAN

- Scope / bird cap at seven: The rollout ramp ties the cap to "observed call recognizability in user testing," moving from 2 to 3 to 5 to 7 as the chorus and species recognition can support more birds.

- Scope / new species offered at age-tied milestones: NOT RECOVERABLE FROM PLAN

- Scope / six-species pool with silhouette, palette, and motif library: The plan ties species to visual recognizability and procedural call recognizability. Each species needs fixed signature audio parameters and a tuned motif library; expansion is "calibration work, not v1."

- Scope / hidden personality vector: The hidden vector supports felt behavior without exposing numbers. The client gets mood, perch, pose, call seeds, and render shaping, while raw personality values stay server-side so the relationship is not reduced to visible stats.

- Scope / slow personality drift driven by presence and listen-in: The drift design makes regular presence and attention gradually visible. The calibration target is measurable change around 7 days and visible change around 21 days for a "typical regular user."

- Scope / monotonic drift toward expressive: The plan uses one-sided clamping and tests to ensure no event log can drive any trait downward. This supports the non-goal that birds do not decay, suffer, or punish neglect.

- Scope / mood layer persisted across sessions: Persisted mood, `mood_entered_at`, and decaying modifiers let mood survive sessions and multi-device sync "without recomputation."

- Scope / server-side simulation tick: The tick is the "architectural heart" and single writer of canonical aliveness. It consumes ordered events, writes mood/personality/perch/notebook state, and keeps decisions server-canonical.

- Scope / slow tick cadence near 60s: The cadence matches the product's slow, ambient character while keeping simulation cheap. Jitter spreads load and avoids synchronized waves.

- Scope / horizontal scene in the browser tab: The plan uses a single scene as the aviary surface, with client-side rendering for motion and ornaments so the simulation remains decisions, not pixels.

- Scope / exactly three perch zones: NOT RECOVERABLE FROM PLAN

- Scope / day/night anchored to local time: Local time supports aliveness; the risks section says wrong timezone would cause "morning calls at midnight," which is an aliveness regression.

- Scope / rare ambient weather: Weather shapes mood and call scheduling without becoming a game system. The target of roughly three events per week keeps it ambient and rare.

- Scope / ambient leaf and feather drift: These are "pure client ornaments" that add motion without reaching the server or affecting canonical state.

- Scope / sparse top-bar chrome: The top bar holds settings, accessibility, notebook, offer, and settle while keeping the aviary scene free of "buttons, badges, hover-tooltips, overlay icons, or inline labels."

- Scope / top-bar fade on cursor stillness: The fade supports restraint by letting chrome recede from the field. The plan specifies easing rather than snap so the surface stays gentle.

- Scope / listen-in interaction: Listen-in records attention for drift and changes only the local mix. It raises the focused bird and lowers others to ambient, but "other birds never go silent," preserving the living scene.

- Scope / offer interaction: Offers are user-initiated events that contribute to curiosity and a small amount to boldness. The plan treats them as interaction inputs to simulation, not rewards or achievements.

- Scope / specific offer kinds: seed, song fragment, still pool: NOT RECOVERABLE FROM PLAN

- Scope / per-bird offer cooldown: NOT RECOVERABLE FROM PLAN

- Scope / settle gesture: The sync section frames settle as "presence ends here," applied per session so another active device does not receive a settled lighting state.

- Scope / 5-second click-anywhere settle undo: NOT RECOVERABLE FROM PLAN

- Scope / field notebook: The notebook carries the product's naturalist voice in sparse, grounded prose. It is server-generated from verified facts, read-only, stored indefinitely, and audited through `generation_source`.

- Scope / read-only notebook: Read-only status protects the notebook as observed naturalist prose rather than a user-authored log or game journal.

- Scope / procedural client-side WebAudio synthesis: Procedural-only audio avoids the "canned-software signal." The plan says the audio pipeline is the "affective spine" and rejects recorded fallback as product-damaging.

- Scope / chorus mixing: The chorus is the "audible payoff of procedural synthesis"; mixing preserves intelligibility while making several birds feel like they are calling together.

- Scope / reduced-motion mode: Reduced motion is a "designed surface" that keeps birds drifting, moods changing, and notebook entries accruing while removing continuous ornament motion.

- Scope / screen-reader narration: Narration gives the same naturalist, lowercase, present-tense aviary to screen-reader users, with slow cadence and priority bumps for user-initiated events.

- Scope / call captioning from the same call grammar: Captions share seeds and descriptors with synthesis, so text and sound describe the same generated call and remain available when audio is muted or unavailable.

- Scope / full keyboard navigation: The plan makes every interactive surface reachable and tests keyboard-only access, so the aviary remains usable without pointer input.

- Scope / visible focus indicators on light and dark scenes: Focus needs to read across bright and dim aviary states, so the plan specifies a soft white halo with a dark inner stroke tuned by design.

- Scope / WCAG AA chrome and overlays: Contrast is enforced because text and overlays appear over day/night palettes. CI blocks insufficient contrast.

- Scope / visits: Visits are email-bound, opt-in, read-only, revocable, logged, expiring, and off by default so sharing exists without becoming a social network or write surface.

- Scope / visit-notify opt-in off by default: The default-off choice aligns with privacy and restraint; the product does not pull users back or surface social pressure by default.

- Scope / account export: Export gives account control through on-demand JSON and an emailed, one-time, short-TTL signed download link.

- Scope / soft-delete with 30-day recovery and hard delete after 30 days: Soft delete allows recovery while hard deletion gives a final account lifecycle. During the recovery window, sign-in works to recover.

- Scope / aggregate-only telemetry: Telemetry exists for operational health, not relationship tracking. Per-bird simulation data never reaches analytics or training pipelines.

- Scope / browser support and unsupported-browser surface: The support matrix includes browsers with the needed rendering and AudioWorklet capabilities; older browsers get a "matter-of-fact unsupported-browser surface."

- Explicitly out of scope / native apps: The plan rejects native iOS and Android for v1 because architecture and protocol design "assume browser-only" and should not pre-shape the data model for a native client.

- Explicitly out of scope / gamification surfaces: Achievements, streaks, scores, badges, levels, and disguised calendars are excluded because they would shift the product toward engagement mechanics and away from restraint.

- Explicitly out of scope / Tamagotchi-style mechanics: Birds do not die, hunger, decay, or visibly suffer from neglect because drift is monotonic toward expressive and absence is not punished.

- Explicitly out of scope / social-network surfaces: Profiles, follows, feeds, comments, leaderboards, friend chains, mutual visits, and chat are excluded so visits do not become a social network.

- Explicitly out of scope / push or email notifications about the aviary: The rationale is explicit: "The product does not pull the user back; the user comes when they come."

- Explicitly out of scope / payments, multi-aviary accounts, shared aviaries, customizable scenes, scenery extensions: NOT RECOVERABLE FROM PLAN

- Explicitly out of scope / recorded-audio fallback path: This is a hard rule because recorded fallback would make calls feel canned and damage the affective spine.

- Explicitly out of scope / streak counters of any kind: Streaks are excluded even in disguise, and their metrics are absent from schema so reintroducing them is costly.

- Architecture / six logical services with preserved seams: The plan says deployments may collapse, but "the seams below are real" at the data-flow level, preserving separation among web, auth, aviary API, simulation, mailer, and telemetry.

- Architecture / edge HTML renderer with initial state hint: The edge injects initial state for signed-in users to support the "<500ms first-bird" target.

- Architecture / auth service: Auth owns magic-link issuance, verification, session lifecycle, and email changes so identity flows stay separate from aviary simulation.

- Architecture / Aviary API: The API handles snapshot reads, event-log writes, settings, visits, exports, and deletion, keeping client interaction with canonical state mediated through typed endpoints.

- Architecture / simulation worker pool: Workers are the sole writer of personality, mood, perch, schedule, and notebook entries, enforcing canonical aliveness.

- Architecture / mailer: The mailer centralizes transactional email for magic links, visit invites, exports, and email changes, matching the encrypted-email boundary.

- Architecture / telemetry pipeline: Telemetry is isolated as aggregate-only operational metrics on a network with no read path to the simulation DB.

- Data stores / Postgres primary state DB: The plan chooses Postgres for ACID semantics, advisory locks, mature tooling, and enough capacity for v1, while rejecting eventual-consistency debugging for "a domain where additive correctness matters."

- Data stores / append-only Postgres event log: The plan chooses this over Kafka for v1 because operations are simpler, volumes are low, and advisory locks naturally enforce single-writer-per-account.

- Data stores / object storage for export downloads: Signed URLs with short TTLs support export delivery without permanently exposing account data.

- Data stores / Redis cache: The cache supports snapshot hot-path reads, magic-link nonce storage, and rate-limit counters, all named as short-lived operational needs.

- Data stores / separate analytics warehouse: Isolation ensures aggregate telemetry cannot read simulation state or per-bird data.

- Client/server split / client emits events, never absolute state: This prevents client-vs-server conflicts and keeps personality, mood, perch, and notebook state canonical.

- Client/server split / client renders from snapshots: Snapshots provide skeletal canonical state; the client fills in interpolation, audio synthesis, listen-in mix, and ornaments to render smoothly without round-trips.

- Render pipeline boundary / snapshot as boundary: The split "keeps the simulation cheap" and lets the client render at 60fps.

- Render pipeline boundary / server-emitted call schedule: Scheduling calls server-side with seeds keeps multi-device and visit coherence, while synthesis stays client-side.

- Snapshot / no numeric personality vector values: This protects the rule that personality is never exposed numerically and prevents users from reading the vector off the wire.

- Data model / encrypted email on account: Email is decrypted only by auth and mailer services and is "never used as an identifier outside this table."

- Data model / timezone on account: `tz_iana` lets day/night and time-of-day behavior render against the user's local timezone.

- Data model / personality filter state: The filter state supports the low-pass drift function with running averages and decay accumulators.

- Data model / no cross-account bird index: The plan says no cross-account index ever exists, supporting tenant isolation.

- Data model / mood modifiers: Modifiers record decaying sources such as offers, rain, or peer alarm calls so the next tick can shape mood transitions.

- Data model / append-only interaction events: Event order is server-assigned by `ingest_seq`; workers consume in order and mark `consumed_at`, supporting idempotent ordered simulation.

- Data model / notebook `generation_source`: Each entry stores the facts and template that drove the prose so future audits can verify grounding.

- Data model / hashed visit tokens and visitor email lookup: Raw invite tokens are never stored, and the visitor email hash supports lookup at link consumption without exposing the raw token.

- Data model / device labels: Device labels are derived from user agent for the revocation UI.

- Snapshot / derived render params: Render params are mood/personality/time-of-day functions, "deliberately squashed and noise-padded," so they can shape motion without revealing the personality vector.

- Data model open call / personality scalar range and calibrated seed distributions: Scalars in [0,1] and per-species calibrated distributions give the simulation a tunable base for drift.

- Data model open call / locked mood enum: The plan locks wary, content, curious, drowsy, and alert for v1 because adding moods later requires calibration.

- API surface / REST typed errors and voice mapping: The API returns structured codes while client-side voice mapping ensures system errors never bleed into naturalist UI.

- Auth API / uniform magic-link request response: Always returning 200 avoids email enumeration.

- Auth API / verify consumes magic link: Consuming the link enforces single-use auth.

- Aviary API / snapshot with private no-store: Snapshots carry current private aviary state and should not be stored by shared caches.

- Aviary API / snapshot delta support: The wire format supports deltas when `since` is recent, but defaults to full snapshots for v1 simplicity.

- Aviary API / event write endpoint: Server-assigned `ingest_seq` and 202 Accepted make event submission append-only and asynchronous to simulation.

- Aviary API / bird detail omits personality numbers: The endpoint returns name, species, mood, perch, and pose, preserving the hidden vector.

- Settings API / explicit timezone update: The client reports timezone changes so day/night and time-of-day behavior stay aligned with the user's current locale.

- Export API / emailed download link: The mailer sends a one-time, short-TTL signed link, avoiding direct long-lived exposure of export data.

- Visits API / read-only visit snapshot: Visitors receive the same snapshot schema but cannot submit events; any POST with a visit token is rejected to preserve host state.

- API rules / no cross-account account_id parameter: Endpoints are scoped to the requester so cross-account reads are not possible through request parameters.

- API rules / no analytics client endpoint: Aggregate telemetry comes from server logs and synthetic perf, not identifiable browser pings.

- API rules / named event types only: Unknown event types are rejected so new interactions require deliberate change and calibration review.

- API open call / fetch-on-event rather than SSE: v1 uses visibility changes, frame gaps, keepalive, and user actions because the read pattern is bursty and long-lived connections are costly on mobile.

- Simulation / per-account tick jitter: Jitter spreads load and avoids synchronized waves.

- Simulation / advisory lock per account: The advisory lock guarantees single-writer-per-account.

- Simulation / event cursor advanced atomically with state write: Atomic cursor advancement keeps consumed events and canonical state consistent.

- Simulation / drift function low-pass filter: Low-pass smoothing makes change gradual and calibratable rather than jumpy.

- Simulation / presence-weighted drift: Presence pings dominate drift because presence is the primary expression of attention in this product.

- Simulation / listen-in drift contribution: Listen-in strongly contributes to social warmth and vocal frequency for the focused bird because it represents directed attention.

- Simulation / offer drift contribution: Offers contribute to curiosity and a small amount to boldness, making them gentle interaction inputs.

- Simulation / 7-day measurable and 21-day visible drift targets: These targets define the pace where instruments detect change before users visibly notice it.

- Simulation / plumage saturation slowest drift: Plumage is the most visible long-term signal, so it drifts more slowly and targets visibility around 21 days.

- Simulation / mood transitions shaped by personality, recent events, time, weather, and peers: Mood is intended to feel contextual, not random, and to preserve bird-to-bird influence.

- Simulation / persisted mood state-machine moves: Persisting mood transitions lets mood survive sessions and sync without recomputation.

- Simulation / call scheduling rather than synthesis: The server schedules calls and seeds; the client realizes audio. This keeps canonical timing coherent and audio rendering local.

- Simulation / deterministic call seeds: Seeds make realized audio identical across clients and visits for the same time slice.

- Simulation / response links between calls: `responds_to` links support call-and-response chains and bird sociality.

- Simulation / alarm propagation: Wary or alert transitions influence peers, making bird states relational.

- Simulation / chorus formation: Widened call windows encourage overlap and let the rendering layer produce a comprehensible chorus.

- Simulation / day/night phases with smooth transitions: Phase and progress allow client palette interpolation instead of snap changes.

- Simulation / weather at low Poisson rate: Low-rate events make weather rare ambient texture rather than a constant mechanic.

- Simulation / notebook token bucket: The one-entry-per-day average preserves sparsity and prevents notebook noise.

- Simulation / noteworthy-pattern eligibility: Entries appear only for meaningful state diffs or event windows, so prose remains grounded in observable change.

- Simulation / no LLM at runtime in v1: Runtime LLM prose is rejected because it risks numeric leakage, gamification language, and hallucinated facts.

- Simulation / forbidden prose patterns: Blocklists prevent streak, achievement, unlock, count, user-address, and visit-social leakage.

- Simulation / tick correctness invariants: CI tests single writer, advisory locking, idempotency, monotonic drift, deterministic call seeds, account scoping, and latency because tick correctness is not "best effort."

- Sync / pull-based read path: Snapshot pulls on load, visibility, long frame gaps, keepalive, and selected actions fit the slow-tick cadence and recover after suspend or throttling.

- Sync / event batching write path: Presence pings batch at a small cadence while user-initiated events send immediately, balancing low overhead with responsiveness.

- Sync / overlapping presence deduplication: Multiple devices showing the aviary do not produce double presence time.

- Sync / listen-in deduplication across sessions: Effective listen-in seconds are summed once per second of attention per account and bird.

- Sync / settle per session: The plan says settle means "presence ends here"; global settle would require client-to-client signaling.

- Sync / settings last-writer-wins: For rename, prefs, and accessibility settings, the failure mode is acceptable and reversible.

- Sync / stable bird identity: `bird_id` is never reused or reset because identity continuity protects the user's relationship with each bird.

- Sync / no reset bird operation: The absence of reset prevents accidental destruction of accumulated personality and relationship state.

- Frontend / TypeScript throughout: NOT RECOVERABLE FROM PLAN

- Frontend / Canvas2D renderer: Canvas2D is chosen because the 2MB initial bundle cap is tight and it can hit 60fps on the 5-year-old laptop target with less bundle and complexity than WebGL.

- Frontend / Preact chrome: Preact or equivalent keeps top bar, settings, notebook, and modals light enough for the bundle budget.

- Frontend / single small reactive store: A small store fits the snapshot-and-prefs state model; Redux-like weight is unnecessary.

- Frontend / code splitting: Only the aviary route is in the initial route; settings, notebook, visits, export, and deletion load on demand to protect first paint.

- Frontend / inline initial snapshot: A <=5KB snapshot in edge HTML is sufficient to draw the first birds quickly and support the first-bird budget.

- Frontend / tiny critical chunk: Keeping the critical chunk <=50KB gzipped lets the renderer bootstrap and first bird paint before the rest of the app loads.

- Frontend / lazy audio context: Audio is created only after first user gesture because of browser autoplay policies.

- Frontend / captions auto-enabled before audio: Captions cover the silent pre-audio state when the user has audio enabled but the context is not yet created.

- Frontend / scene z-planes and parallax: Background, mid plane, and foreground composition produce depth while staying subtle.

- Frontend / layered bird rendering: Species silhouettes, plumage, accents, and render params let visual identity and mood expression vary without exposing raw personality.

- Frontend / parametric idle micro-motion: Eased noise and random phase offsets ensure "the same motion is never replayed identically."

- Frontend / mood expression in motion: Wary, content, curious, drowsy, and alert each map to different posture and movement, making mood visible without labels.

- Frontend / perch-to-perch transitions: Server scheduling keeps timing canonical, while client easing and midpoint pickup make flight smooth and coherent after reloads.

- Frontend / mood transitions as cross-fades: Cross-fading avoids hard cuts and lets mood-shaping parameters change gently.

- Frontend / reduced-motion rendering: Continuous micro-motion becomes cross-faded pose changes, flights become cross-fades, ornaments disappear, and day/night shifts slow, so the mode is complete rather than stripped.

- Frontend / chrome overlays trap focus and dim aviary: Modals remain accessible while preserving the aviary context behind them.

- Frontend / no in-scene buttons or labels: This hard rule preserves the field quality of the scene and prevents UI chrome from colonizing the aviary.

- Frontend / quiet loading and empty state: The same quiet field covers empty, loading, and cold-cache states; no spinner or loading label appears because spinner-like machinery was named as a shortcut that would damage the product center.

- Frontend / performance CI gates: Bundle size, first-bird-render, 60fps, and memory growth tests make the non-negotiable budgets enforceable.

- Audio / single lazy AudioContext: One context per page avoids unnecessary audio resources and respects autoplay policy.

- Audio / graceful silence with captions: If audio cannot start, the aviary still works quietly with captions instead of falling back to canned recordings.

- Audio / tap-to-enable-calls affordance: A small top-bar icon invites audio activation without pulling focus from the aviary.

- Audio / AudioWorklet preferred: AudioWorklet is available in the support matrix and avoids the v1 use of ScriptProcessor.

- Audio / species motif libraries: Motifs give each species recognizable signature parameters while still allowing procedural variation.

- Audio / per-bird stable deviation seed: Birds within a species can sound individually different through small pitch and timing offsets.

- Audio / mood-shaped call distributions: Mood and personality shape motif selection so calls express current state.

- Audio / soft-knee chorus bus: Compression preserves intelligibility while keeping the impression of simultaneous birds.

- Audio / listen-in mix ramp: Gradual gain ramps focus attention on one bird without a snap and without silencing others.

- Audio / captions near calling bird: Placement connects caption text to the visible bird and fades with the call envelope.

- Audio / no WebAudio recorded fallback: The plan explicitly ships no recorded-audio fallback path; unsupported browsers get matter-of-fact handling.

- Audio / synth node pooling and zero steady-state allocation: Pooling controls memory and CPU during long ambient sessions.

- Audio / CPU budget during chorus: The 20% of one core target keeps chorus affordable on the 5-year-old laptop profile.

- Audio / mute and captions prefs: Captions are forced on when mute is on or audio is unavailable, keeping call information accessible.

- Audio / single linear volume control: NOT RECOVERABLE FROM PLAN

- Accessibility / aria-live narration: A polite live region gives slow naturalist updates without overwhelming assistive technology.

- Accessibility / narration priority bumps: User-initiated events receive priority so interactions get timely feedback.

- Accessibility / server-side primary narration generation: Server-side generation keeps voice consistency with notebook prose and avoids a heavier client template engine.

- Accessibility / client-side fallback narration: The fallback covers snapshot-only contexts when server narration is unavailable.

- Accessibility / keyboard tab order and arrow navigation: The plan makes top-bar controls and bird focus predictable, with Enter for listen-in and Escape to exit.

- Accessibility / modal focus traps: Focus trapping and return-to-trigger behavior keep overlays usable.

- Accessibility / contrast-aware caption and overlay tint: The aviary changes palette across day and night, so captions and overlay text need adaptive readability.

- Accessibility / voice continuity across surfaces: Naturalist and matter-of-fact copy are separated by modules and lints so copy cannot drift by accident.

- Accessibility / accessibility settings page: Captions, mute, volume, narration, narration rate, and reduced-motion override give users direct control over access surfaces.

- Accessibility / manual and automated screen-reader testing: Review and tests protect narration, captions, focus order, contrast, and ARIA attributes from regressions.

- Performance / first-bird, bundle, 60fps, memory, and tick budgets: These budgets are "non-negotiable" because performance is part of felt aliveness, not only engineering hygiene.

- Performance / synthetic performance fleet: Automated browsers repeatedly run real session loops so regressions in first-bird-render, frame rate, audio, and snapshot latency alert.

- Observability / aggregate-only RUM: Page and render timings are useful health signals, but the plan forbids per-account dimensions and per-event PII.

- Observability / static telemetry schema: Build-time metric definitions and privacy-domain review prevent forbidden fields from entering telemetry.

- Observability / hashed account_id in tick logs: Hashing supports operational cardinality while avoiding email, bird names, raw deltas, and relationship details in logs.

- Observability / daily tick replay audit: Replaying sampled event logs checks idempotency and alarms on drift.

- Observability / not measuring DAU, MAU, retention, churn, or visit frequency: The product is not operated on engagement metrics, and its success "does not have a dashboard."

- Versioning / snapshot and event payload versions: Versioned payloads and 30-day previous-schema support let clients survive rollout windows.

- Versioning / bird-table migration structural tests: Every migration touching birds must preserve `bird_id` and personality vector, guarding identity continuity.

- Versioning / backups and restore drills: Data migrations are backed up and restore drills run quarterly to make recovery real.

- Rollout / internal alpha: Alpha validates aliveness, first frame, noticing, calls, drift weights, mood probabilities, weather rates, notebook sparsity, renderer choice, sound review, and reduced-motion review.

- Rollout / closed beta: Beta stresses sync, multi-device behavior, accessibility surfaces, production synthetic performance, and telemetry boundary auditing.

- Rollout / public v1: Public v1 opens signup, raises the bird cap toward seven, wires the notebook, and ships all accessibility surfaces at launch rather than deferring them.

- Rollout / month-by-month bird cap ramp: The cap rises based on observed call recognizability in user testing.

- Rollout / drift weights held constant after public v1: Weight changes require calibration review because drift pace defines the relationship.

- Rollout / notebook templates expand from writer-of-record output: Template growth remains reviewed and authored, never auto-generated.

- Risks / calibration harness: The harness replays scripted event logs and reports trait-change-per-week distributions because the 1-week measurable / 3-week visible target spans many weights.

- Risks / weights as single rollbackable config: Keeping weights in one config makes rollback immediate and avoids migration when calibration is wrong.

- Risks / DB grants for personality, mood, and perch writes: Grants enforce that only the simulation worker can update canonical bird state.

- Risks / structural code scans for forbidden writes: Scans catch accidental endpoints, tools, or code paths that mutate personality or related state outside the worker.

- Risks / audio spectral variety test: Synthesizing 1000 calls per species and checking feature variety guards against robotic repetition.

- Risks / sound-design ownership and user testing: Sound design review and alpha questions about repeated calls guard against "audio uncanniness."

- Risks / accessibility regression tests and manual review: Dedicated tests and reviewers protect reduced motion and narration when rendering changes.

- Risks / privacy-domain owner and metric audits: Required review, linting, quarterly audits, and annual external review counter gradual privacy boundary erosion.

- Risks / edge-cached HTML and per-region edge presence: These mitigate fragile first-bird latency from slow edge/API/cold cache paths.

- Risks / visit abuse controls: Email binding, single-use device sessions, immediate revocation, rate limits, and throttled visit snapshots reduce token sharing and scraping.

- Risks / personality write-ahead journal: Journaling personality before ticks allows restoration on failure and protects against silent relationship loss.

- Risks / personality-vector daily alarm: Sampling for unexplained large drops detects silent destructive changes to personality vectors.

- Risks / timezone refresh on every snapshot pull: Updating timezone prevents aliveness regressions like morning calls at midnight.

- Risks / magic-link replay controls: Single-use random tokens, 15-minute expiry, rate limits, verified email changes, session revocation, and alarms address auth attacks.

- Risks / PR checklist against streaks and achievements: The checklist asks whether a surface exposes visit frequency, attention, counts, user-address, or comparison, guarding against future engagement features.

- Risks / notebook prose lint and review: Forbidden-pattern lint, source facts, and quarterly human review keep prose grounded and in voice.

- Risks / cross-platform synthesis determinism test: Testing the same seed and scheduled time across browsers preserves multi-device and visit coherence.

- Cross-cutting / user copy only in voice modules: Centralizing copy plus linting prevents inline strings from breaking the voice split.

- Cross-cutting / analytics package import boundary: `analytics/` cannot import simulation, birds, notebook, or events, blocking accidental privacy leakage at code-organization level.

- Cross-cutting / mailer-only email decryption: The mailer package handles email, and no other package may decrypt the email field.

- Cross-cutting / logging scrubbers: Logging utilities scrub `bird_id`, `personality`, and `notebook` fields by default to protect relationship data.

- Cross-cutting / test strategy breadth: Unit, integration, property, E2E, accessibility, performance, and voice tests correspond to the plan's hard rules and risk areas.

- Security / HTTPS and secure cookies: HTTPS-only endpoints and HttpOnly, SameSite=Lax, Secure cookies protect sessions.

- Security / random hashed magic-link and visit tokens: 256-bit random tokens hashed at rest prevent raw-token exposure.

- Security / CSP: Restricting script origins to first-party and audio worklet origin limits script injection risk.

- Security / rate limits: Auth, invite, and export rate limits reduce abuse of public-facing endpoints.

- Cross-cutting open call / English-only v1: NOT RECOVERABLE FROM PLAN

- Dependencies / required roles: Engine owner, rendering owner, audio designer, writer-of-record, accessibility reviewer, privacy-domain owner, and backend infrastructure roles map directly to the specialized domains the plan treats as quality-critical.

- Dependencies / external services: Edge functions, transactional email, object storage, monitoring, and synthetic perf fleet are required because first paint, auth, exports, alarms, and regression detection depend on them.

- Deferred / SSE, localization, LLM-grounded prose, co-presence, customization, native apps: These are deferred to keep v1 aligned with browser-only, slow-sync, reviewed-prose, restrained-surface scope.

- Final framing / implementation rules: The final section restates the whys: server-only writes keep drift coherent; procedural-only audio keeps chorus alive; designed accessibility keeps the affective core available; aggregate telemetry keeps the relationship private; absence of streak/achievement metrics makes reintroduction structurally costly.
