## System-level intent

- A quiet, bounded, single-place product. This shows up first in the "browser-only, single-screen aviary" boundary, "one signed-in owner and one canonical aviary," the hard seven-bird limit, and the exclusions of "native clients, payments, scene customization, multiple or shared aviaries, profiles, chat, comments, public discovery, feeds, rankings, achievements, scores, levels, streaks." The plan repeatedly avoids growth, social, and gamified surfaces.

- Ambient observation over engagement pressure. The plan says "Offers are gestures, not feeding," adoption "can be declined or deferred without pressure," there is "no generic notification service for aviary activity," and rollout success must not use "user engagement/streak metrics." Presence is calibrated with "quiet watching, not tuned for click volume."

- Canonical, server-authored life rather than client simulation. This appears in "The worker is the only writer of personality, mood, perch decisions, weather, and notebook observations," "The browser owns only ephemeral rendering state," "A stale client may submit an intent, never a replacement snapshot," and "no local simulation advancement or speculative durable vector writes."

- Continuity without punishment. Personality uses "nonnegative_delta" and "No absence or neglect term subtracts from any trait." Two weeks away "never harms a bird"; absence makes greeting probability "more ambient," not punitive. The plan reconciles "quieter returns with non-punitive monotonic personality."

- Privacy by projection, separation, and omission. Ordinary snapshots "Never raw vectors"; exports are the explicit exception. Email "never appears in account IDs, partition keys, general logs, or telemetry." Relationship data does not enter "RUM, tracing attributes, training, recommendations, or population dashboards," and dashboards have no "bird, account, email, event payload, or invitation dimensions."

- Product voice is observational in the scene and plain in system surfaces. The stated copy rule is "lowercase, present-tense naturalist observation" for product prose, while "identity, error, sync, and accessibility-settings copy is direct and matter-of-fact." Notebook templates avoid "generic event-log language" and "behavior judgments about the user."

- Accessibility is a release path, not a supplement. The plan titles a section "Accessibility as a release path," requires that reduced-motion keeps "the same moods, calls, offers, notebook, and continuity," and says the release condition is that "the birds still feel like a continuing place, not merely that controls can be reached."

- First paint and long-session performance are part of the product feel. The plan requires "first bird visible <500 ms," "no spinner," a "mid-pose, already-running first frame," "60 fps idle," and 30-minute memory/frame endurance. It says a scene with missing "long-session performance is not v1."

- Visits are sharing without agency transfer. Visits use "the same host state" but a "distinct authorization and presentation boundary"; visitor projections have "neither controls nor notebook/account data," no visitor event endpoint, and "Visitor pages never initialize the presence reporter."

- Versioned, deterministic evolution protects bird identity. Schema and rules are "versioned so migrations do not regenerate birds"; ticks use a "seeded deterministic PRNG"; rollout can "roll back rules by version without replacing bird IDs or lowering vectors."

## Per-feature whys

### Product boundary and decisions

- Browser-only, single-screen aviary: NOT RECOVERABLE FROM PLAN

- One signed-in owner and one canonical aviary: NOT RECOVERABLE FROM PLAN

- Starting each account with two system-selected birds: NOT RECOVERABLE FROM PLAN

- A coherent pool of about six species: the plan names "coherent pool" as the reason-like constraint, and later says to expand only after species call signatures "remain recognizable."

- Age-based invitations to adopt additional birds: the plan frames age gates as "independent of attention," "not promises shown as progress," and something that can be "declined or deferred without pressure."

- Hard limit of seven birds: the scene must keep "all seven birds" visible on phone and desktop, with collision-safe composition, CPU, call recognition, and accessibility remaining acceptable during rollout.

- Owners name and rename birds: NOT RECOVERABLE FROM PLAN

- Return-greetings: greetings make owner entry or "genuine focus return" responsive to accepted presence history. The plan uses absence length to vary glance, approach, and call, while avoiding "a synchronized all-bird greeting" and "repeated arrivals" from visibility flurries.

- Ambient watching: the plan treats quiet presence as the main signal. Presence is dominant in drift, but the "activity window must be tested with quiet watching, not tuned for click volume."

- Listen-in: listen-in makes one focused call prominent while "every other bird remains audible at ambient level," and contributes bounded per-bird signals to warmth and vocal frequency.

- Seed, song-fragment, and still-pool offers: offers are explicitly "gestures, not feeding." Their accepted events produce reaction poses and modest curiosity/boldness drift, with cooldowns and daily caps to prevent click bursts from moving traits visibly.

- Settle: settle is a quiet goodbye-like action. It "shifts light and mix over seconds," "ends presence," can quiet transient mood, and has a five-second undo; it "has no drift term" and should not become permanent night for another device.

- Sparse read-only field notebook: notebook entries capture "meaningful state transitions and contrasts over time" rather than every visit. Sparse rate limits, reviewed templates, and stored rendered prose keep it from becoming an event log or rewriting history.

- Magic-link accounts: magic links are one-use, 15-minute, atomically consumed, and rate-limited; "Uniform request response prevents email enumeration," and sessions are revocable per device.

- Cross-device state: cross-device behavior is grounded in the canonical snapshot, `state_version`, server-stamped event order, overlap union for presence, and "No last-write-wins client merge."

- Optional read-only visits: visits let a host share the "current ambient projection" while protecting host state. There is no discoverability endpoint, no visitor event endpoint, no notebook access, no vector access, and revocation is checked on every pull.

- Account export: export is an authenticated, on-demand, consistent JSON snapshot, and is the explicit exception where raw current vectors are available outside ordinary UI/API projection.

- Account deletion and restore: deletion "immediately disables normal use," allows "explicit restoration within 30 days," then purges account-linked stores using a deletion manifest and audit. Aggregate telemetry has "no account key to recover."

- Naturalist narration: narration gives assistive access to the same scene facts, using "specific lowercase present-tense prose with names, relative perch, weather, and call character."

- Call captions: captions keep the aviary complete when audio is blocked, unavailable, denied, or silent. They are generated from "the exact compiled motif and variation parameters used for that call."

- Deliberately designed reduced-motion scene: reduced motion preserves "the same moods, calls, offers, notebook, and continuity" while replacing pose animation and flights with slow still-pose cross-fades and removing drifting ornaments.

- Three depth zones and no pan, zoom, or scrolling: the plan says the aviary "remains fully visible on phone and desktop" and later requires perch spacing to keep all seven birds visible at narrow phone widths.

- Scene with no controls or labels: NOT RECOVERABLE FROM PLAN

- Thin top bar for account/settings, accessibility, notebook, offer, and settle: the top bar keeps controls outside the scene; the settle control is placed there because the interaction specification "explicitly requires top-bar access."

- No welcome toast, entry sequence, visit badge, or numeric personality display: the plan grounds this mainly in adjacent voice/privacy choices: "No textual return message," "Never raw vectors" in ordinary snapshots, and no stats UI. A feature-specific rationale is otherwise NOT RECOVERABLE FROM PLAN.

- Visit-notification setting off by default and only for visits: the plan says this setting is "never an onboarding prompt" and there is "no generic notification service for aviary activity," preserving the no push re-engagement boundary.

- Four-minute recent-activity window for presence: NOT RECOVERABLE FROM PLAN

- One-minute server tick: NOT RECOVERABLE FROM PLAN

- Age-based adoption eligibility at about months 3, 5, 8, 10, and 12: the plan says these are "versioned configuration, not promises shown as progress" and "independent of attention."

- Account IANA timezone for day/night cycle, settings override, and device-timezone suggestion: the plan uses local time for daylight, mood baselines, dawn/dusk/night behavior, and host-time visits; the override/suggestion rationale is NOT RECOVERABLE FROM PLAN.

### System shape and ownership

- Small web client, authenticated API, transactional primary database, one-minute simulation worker, email adapter, and aggregate-only operations telemetry path: this shape separates canonical state, identity delivery, and operations metrics. The API serves snapshots and accepts intents; the worker authors simulation; telemetry is aggregate-only.

- Private compact canonical snapshot and intent events: the snapshot projects state without raw vectors; intents let a stale client submit actions without replacing canonical state.

- Worker-only writes for personality, mood, perch decisions, weather, and notebook observations: this prevents client-submitted personality or mood and makes one transactional tick the source of truth.

- Versioned schema and simulation rules: versioning prevents migrations from regenerating birds and supports rule rollback without replacing bird IDs or lowering vectors.

- TypeScript web application with server-rendered HTML/SVG first scene and hydrated HTML controls: the stated goal is to "paint a bird immediately" and hydrate "without replacing that first frame."

- PostgreSQL canonical rows, ordered event log, and transactional worker cursor: this supports ordered event consumption, one-writer ticks, retry safety, and state/cursor/version commits together.

- Durable scheduled-job queue backed by the same transactional store: the plan uses it for partitioned ticks, retry, catch-up, and backlog alerts without adding a streaming broker for v1.

- No streaming broker or client-state database for v1: NOT RECOVERABLE FROM PLAN

- SVG birds as individually focusable semantic elements: SVG supports "seven birds individually focusable semantic elements" and compact vector silhouettes while keeping DOM panels outside the scene.

- Transform/opacity updates on a bounded animation loop: this supports animation while preserving frame budget; profiling should optimize assets and invalidation before changing the accessible scene architecture.

- Loading the WebAudio worklet after first paint: audio must not block first bird paint.

- One aviary row per account with `state_version`, tick data, deterministic seed/counter, timezone, weather/settle state, and timestamps: these fields support canonical versioning, deterministic replay, local time, weather/settle continuity, and bounded catch-up.

- Partition scheduled ticks by aviary UUID and lock one aviary at a time: this preserves "one worker at a time" and prevents multi-device or retry races from losing identity/drift.

- Catch-up for unvisited accounts in bounded batches: this "preserves continuous canonical evolution without inventing presence during downtime."

- Browser-owned ephemeral rendering state and pending-intent outbox: the browser may interpolate, focus, open panels, and retry intents, but "never ticks mood or personality."

- Snapshot projections omitting raw vector values and private interaction history: this keeps ordinary UI/API from becoming a stats surface or exposing private history.

- Visitor narrower projection: visitors receive "neither controls nor notebook/account data," preserving read-only sharing.

- Private personalized responses and immutable CDN code/artwork: private `no-store` semantics avoid shared cache leakage, while immutable public assets can be cached safely.

- Quiet sky field when a snapshot is delayed: the plan wants "subtle cues, no spinner," preserving the ambient feel instead of an app-loading surface.

- First-bird fly-in only for one-time post-adoption empty state: later loads should begin "mid-pose, already-running," so the aviary feels continuous rather than newly launched.

- Autoplay-blocked call handling: if audible calls are blocked, the plan preserves the active call score and shows captions, then resumes audio on the first ordinary interaction "without a modal."

### Persistent model and privacy boundaries

- Synthetic UUID account primary key: email must not appear in IDs, partition keys, logs, or telemetry.

- Encrypted verified email and narrowly scoped identity lookup: email is needed for magic links and named invitees, but access is constrained by purpose.

- Account settings for audio, captions, reduced motion, visit notifications, timezone, sessions, export, deletion: NOT RECOVERABLE FROM PLAN

- Stable bird UUID and immutable motif/silhouette seed: bird identity survives renames, rule changes, and migrations.

- Five persisted normalized bird traits: traits drive behavior/plumage but remain internal; ordinary UI does not expose raw numbers.

- Append-only `InteractionEvent`: append-only ordering supports transactional tick consumption, idempotency, support/export/deletion policy, and avoids analytics use.

- `PresenceInterval` union across host devices: overlapping intervals are normalized so simultaneous tabs "cannot double count."

- `NotebookEntry` preserving rendered text: old entries do not rewrite after template changes.

- Per-device revocable sessions and one-use magic links: revocation and one-use tokens constrain account access; tokens are never logged.

- Invitation and visit token records: hashed one-use redemption, expiry, revoked/redeemed timestamps, and scoped visitor sessions support email invites without placing recipient email in generic event keys.

- No second clear-text invitation email field: the invitation service is the only place where recipient email is resolved, reducing clear-text exposure.

- Separate simulation storage from aggregate operations warehouse: credential and network separation prevents relationship data from flowing into operations telemetry.

- Privacy policy listing allowed aggregate categories and excluded relationship data: this makes the aggregate/privacy boundary explicit.

- Deletion manifest and audit of purged stores: this verifies account-linked stores are actually removed after the restoration window.

- No hard-delete of a bird or vector reset during ordinary upgrades: this preserves bird continuity.

### API contract and event ingestion

- Versioned `/v1` JSON contracts with schema validation, authorization, `state_version`, server time, and simulation-rule version: this gives explicit compatibility, sync, and rule-version boundaries.

- CSRF or same-site safeguards, bounded bodies, rate limits, and idempotency keys: these protect mutation endpoints and make retries safe.

- `POST /v1/auth/links` and `POST /v1/auth/consume`: one-use magic links issue revocable per-device sessions; uniform responses prevent email enumeration.

- `GET /v1/aviary/snapshot`: owner-only compact projection gives rendering and semantic data needed to present birds while never returning raw vectors and keeping private cache semantics.

- `POST /v1/aviary/events`: accepted presence/listen/offer/settle intents are validated, deduped, server ordered, and reconciled on the next canonical snapshot or immediate server-authored cue.

- `GET /v1/notebook?cursor=...`: notebook is owner-only, newest-first, stable, paginated, and read-only.

- `PATCH /v1/birds/{id}`: owner-only rename preserves bird identity; a more specific rationale for renaming is NOT RECOVERABLE FROM PLAN.

- `POST /v1/adoptions`: age-gated adoption uses a transactional seven-bird cap and immutable bird ID/seed, preserving the visible/canonical limit and identity.

- `GET/PATCH /v1/settings` and `GET/DELETE /v1/sessions/{id}`: settings and device revocation support accessibility choices, timezone, visit notifications, and session control.

- Invitation create/list/revoke endpoints: the host explicitly creates named invites, can inspect outstanding/active invites with visit log, and can revoke; there is "No discoverability endpoint."

- Visit redeem/snapshot endpoints: one-use links become short-lived read-only visitor sessions; later polls check revocation before returning the host's current ambient projection.

- Unavailable surface for expired, replayed, or revoked links: using the same "matter-of-fact unavailable surface" avoids revealing which failure occurred.

- Export, deletion, and restore endpoints: sensitive account lifecycle actions require a fresh authenticated session and consistent state handling.

- Presence reports at most every 30 seconds under visibility, focus, and recent activity: the plan uses these conditions so background, blurred, idle, sleeping, or duplicate tabs do not fabricate time.

- Server-side presence plausibility checks and overlap union: caps, session-lifetime checks, idempotency, and cross-device union prevent unlimited credit from clock skew, retries, and lost beacons.

- Visitor pages without presence reporter: visitors never contribute presence to the host aviary.

- Snapshot polling on load, visibility return, suspend gaps, and low-frequency visible keepalive: this keeps the displayed state reconciled without local simulation advancement.

- Discarding out-of-order responses and interpolating from last displayed pose: this prevents sync regressions and hard snaps.

- Visitor revocation target under 15 seconds while visible: revocation should be "perceptibly immediate."

- Offline owner last-state viewing: an offline owner can view the last state with a small connectivity status, but cannot locally advance simulation or durably write vectors.

### Simulation, drift, mood, and calls

- Minute-boundary tick sequence: the sequence makes state changes ordered, deterministic, and committed together, including events, daylight/weather, attention, mood/perch, calls, notebook, cursor, and version.

- Seeded deterministic PRNG and replay fixtures: replay must reproduce state exactly.

- Distinct species-consistent per-bird vectors: birds begin with individual but species-consistent personality; the numbers stay internal.

- Seven-day exponential attention trace proposal: NOT RECOVERABLE FROM PLAN

- Monotonic personality deltas: nonnegative additive deltas ensure "No absence or neglect term subtracts from any trait."

- Presence as dominant common contribution: the plan makes quiet owner presence the main relationship signal.

- Listen-in drift bias: focused listening modestly biases warmth and vocal frequency for that bird.

- Offer drift bias: accepted offers modestly bias curiosity, and nearby offers modestly bias boldness.

- Offer cooldown and daily signal caps: cooldowns and caps prevent a burst of clicks from moving a trait visibly.

- Settle ending presence with no drift term: settle quiets mood and marks departure without changing personality.

- Offline coefficient calibration with scripted histories: this ensures drift is measurable over days and perceptible after about 21 days, with "no single-session visible jump."

- Zero-presence and two-week-absence cases: these prove persisted vectors never decrease, attention decays, greetings become more ambient, and baseline calls remain.

- Five moods with daily-ish relaxation toward local-time baseline: mood continuity should not reset on session open.

- Mood transition probabilities based on context and history: persisted mood, mood age, local time, weather, gestures, personality, and calls make changes stateful rather than random flicker.

- Dwell-time floors and hysteresis: these prevent flicker.

- Night mood behavior and nightjar-like exception: night is mostly drowsy/settled while retaining occasional calls for the nightjar-like species.

- Sparse seeded weather: weather adds short rain and occasional wind that adjust mood/call rates briefly, with "no catastrophic events."

- Bird-authored perch selection: boldness and mood choose front/middle/back zones while preserving collision-safe composition for seven birds.

- Settled tab local evening overlay and quiet mix: settle affects the owner session so another device does not mistake a goodbye for permanent night.

- Click within five seconds undoes local settle transition: NOT RECOVERABLE FROM PLAN

- Return-greeting greeter selection: one eligible greeter is weighted by boldness, warmth, mood, and recent greeting history to create varied but bounded greeting behavior.

- Motif library and stable bird-specific signature: stable signature keeps a bird recognizable, while variation changes phrasing without erasing identity.

- Semantic call descriptors compiled client-side: descriptors carry bird ID, motif, timing, variation, mood envelope, and gain intent so two devices can render the same underlying events without downloaded call files.

- Vocal-frequency, mood, time, and weather call shaping: traits and conditions shape rate, chorus joining, articulation, and density.

- Bird-to-bird bounded responses and refractory periods: these prevent runaway choruses.

- Scheduling near-future calls between pulls: this lets the client render semantic calls between polling intervals.

- No looped recordings or downloaded call files: NOT RECOVERABLE FROM PLAN

- Notebook generation from state transitions and contrasts: entries record noteworthy observations such as first greeter, unusual quiet, rain after a stretch, or perch preference shift.

- Notebook rate limits near one entry every few days: this keeps the notebook sparse and avoids manufacturing an entry every visit.

- Reviewed deterministic notebook templates: templates bind concrete bird/time/scene facts, suppress duplicates, avoid visit counts and trait values, and avoid behavior judgments about the user.

### Client scene and interaction rendering

- One logical coordinate system with sky/foliage, three perch depths, birds, and foreground leaves: this supports composition across desktop and phone while keeping the scene coherent.

- Compact SVG or procedural shapes for approximately six species: the plan wants compact assets and isolated pose drawing; a more specific rationale is NOT RECOVERABLE FROM PLAN.

- Separating pose drawing from state choice: this keeps rendering independent from simulation choice.

- Responsive perch spacing within viewport safe areas: all seven birds remain visible at narrow phone widths without clipping or hiding focused birds.

- No inline labels, hover tooltips, drag placement, or camera movement: NOT RECOVERABLE FROM PLAN

- Host local-time daylight palettes and restrained wind/rain: local time ties scene conditions to the aviary's time cycle; restraint preserves ambient feel.

- Client-only leaf and feather drift ornaments: ornaments are independent of canonical bird state and "never sent as interaction data."

- `requestAnimationFrame` only while visible: frame work pauses in hidden tabs to avoid unnecessary work and resumes after fetching current state.

- First paint at server-provided mid-action phase: starting at a preen, scan, shuffle, or call avoids pose-zero and makes the scene feel already alive.

- Blending to anchors with bounded correction: this prevents hard snaps while correcting long gaps.

- Reduced-motion rendering swaps: slow still-pose cross-fades, perch fades, and no drifting ornaments preserve continuity without motion-heavy animation.

- Listen-in engagement and disengagement controls: pointer, tap, keyboard Enter, Escape, focus changes, and empty-scene taps make listen-in operable by mouse, touch, and keyboard.

- Gradual listen-in gain ramps: focused calls become prominent without hard-cutting the mix, while other birds remain audible.

- Separating visual focus from event ingestion: a failed network write "does not hard-cut the mix."

- Offer control surface with three choices plus song-fragment library: NOT RECOVERABLE FROM PLAN

- Server-accepted offer reactions: reaction poses come from accepted canonical events rather than local-only client state.

- Cooldown affordance text only in the control surface: repeated offers are restrained with matter-of-fact text.

- Ordinary tab close requiring no gesture: NOT RECOVERABLE FROM PLAN

### Audio and captions

- One bounded WebAudio graph: per-bird voices, gains, buses, limiter, and master gain cap simultaneous voices and avoid unbounded nodes per call.

- Envelopes, pitch/timing deviations, filtering, and spatial cues: these render call descriptors into varied calls.

- Crossfaded chorus layers and summed-energy caps: this avoids clipping.

- Listen-in ramps over roughly 0.5-1.5 seconds and settle longer quieting ramp: these transitions keep audio gradual and support the listen/settle interactions.

- Time and mood affecting rates rather than a global playback loop: this keeps audio tied to semantic calls instead of a canned loop.

- Captions generated from compiled motif parameters: captions match the generated sound exactly, including note count, rise/fall, trill, pause, softness/sharpness, and perch.

- Caption placement near birds with accessible text equivalent: captions remain associated with the source bird without covering birds or controls.

- Captions enabled by choice and default when audio is unavailable, denied, or silent: audio-off remains "a complete aviary through captions and narration."

- No recorded-audio fallback: NOT RECOVERABLE FROM PLAN

### Accessibility as a release path

- Semantic top-bar buttons and accessible dialogs: controls must be reachable and correctly labeled.

- Keyboard order and commands: Tab reaches controls and scene, first bird receives focus, arrows move bird focus, Enter starts listen-in, and Escape exits so keyboard users can complete the same quiet session.

- Focus restoration when panels close: NOT RECOVERABLE FROM PLAN

- High-contrast focused-bird outline: focus must work in "day, dusk, night, rain, and all species colors."

- No hidden trait numbers or raw mood state as ARIA status: assistive surfaces should not expose internal stats or raw mood as a feed.

- Naturalist narration from the same snapshot and call/gesture facts: narration stays semantically aligned with the visual scene.

- Idle live-region cadence and coalescing: updates only every 30-60 seconds when meaningful so the "screen-reader queue does not grow."

- Prompt but noninterruptive priority for greetings, accepted offers, and settle: important state changes are conveyed without flooding.

- Functional labels direct, scene prose observational: this preserves the product voice split between system controls and naturalist scene prose.

- Caption access without flooding the live region: captions are available as text but do not overwhelm narration.

- Respecting `prefers-reduced-motion` and live switching: users can change motion mode without resetting the scene.

- Color tokens and WCAG AA contrast verification: text contrast must hold on every palette, including top-bar fade and captions.

- Top-bar opacity returning on keyboard focus: keyboard users must get a readable bar.

- Testing with keyboard, screen readers, zoom, reduced motion, audio blocked/off, captions plus narration, and long sessions: the release condition is that access modes still feel like "a continuing place."

### Performance and observability gates

- Initial gzipped JS under 2 MB: the plan sets this CI gate while targeting lower to preserve room for critical CSS and scene assets.

- Code-splitting account/settings, notebook, invite flow, and noncritical species detail: noncritical surfaces must not block first paint.

- First private snapshot in kilobytes and no wait for audio/lazy panels before paint: the first bird should appear quickly.

- First bird visible under 500 ms on a mid-tier 4G phone profile: this measures the real product moment, "the first actual bird pixel," not a placeholder.

- Warm and cold run distributions with regression reporting: NOT RECOVERABLE FROM PLAN

- 60 fps idle for 30 minutes on a five-year-old midrange laptop: long sessions should remain smooth.

- CI memory endurance and AudioNode growth checks: rejects sustained heap/AudioNode growth while allowing bounded GC sawtooth.

- Notebook DOM virtualization: this bounds DOM and releases scrolled-out references.

- Scheduled synthetic browsers from common geographies: these check first-bird timing, continuity, keyboard/reduced-motion/captions, failures, and revocation.

- Aggregate RUM with coarse metrics only: RUM captures timing, frame, audio, errors, and coarse session-duration histograms without per-account interaction analytics.

- Server metrics and tick p99 alarm over 5 seconds: tick queue lag, duration, catch-up, retries, ingest rejection, and snapshot latency protect canonical continuity.

- Telemetry schema allowlist and secret/PII scans: this enforces that dashboards and telemetry do not leak relationship data.

- Support last two major Chrome, Safari, Firefox, and Edge versions: NOT RECOVERABLE FROM PLAN

- Feature-detect WebAudio and reduced motion with unsupported-browser surface: unsupported cases get direct system copy instead of broken behavior.

- Avoid compatibility bundles that threaten first paint: compatibility support must not compromise the first-bird paint gate.

### Build sequence, verification, and rollout

- Freezing contracts, schemas, ownership matrix, copy rules, composition grid, motifs, and accessibility examples before UI: the plan wants deterministic fixtures and privacy allowlists in place before building visible experience.

- Reviewing settle/top-bar discrepancy and timezone setting with design: these are called out as interpretation decisions to validate.

- Persistence/auth/tick spine before first living scene: canonical identity, encrypted email, sessions, aviary, birds/vectors, events, ticks, catch-up, and snapshots prove the server model before the UI depends on it.

- Proving no client path writes vectors: this enforces worker-only authorship.

- Concurrency, retries, worker crashes, timezone/DST, and long-absence tests: these protect one-writer state, stable IDs, monotonic traits, and no fabricated presence.

- Two-species internal first living scene: the plan starts behind internal flags and expands only after signatures remain recognizable.

- Recorded design references and human listening: actual on-screen/audio behavior must match intended scene behavior, and repeated greetings/calls are detected.

- Scripted 1-, 7-, 21-, and 60-day histories: these calibrate drift and mood over time.

- Absence and high-click adversarial cases: these prove no harm from absence and no visible trait movement from click bursts.

- Blind listening tests for each bird across mood and seven-bird chorus: call identity must be recognizable before ramp.

- Full access and account lifecycle in the same release train: narration, reduced motion, keyboard/focus, contrast, settings, export, deletion/restore are not after launch.

- Verifying export includes current vectors while ordinary UI/API never expose them numerically: this preserves the explicit export exception.

- Quiet visits after full access/lifecycle: invite flow, read-only projection, visit log, notification setting, expiry, and revocation are added with authorization tests.

- Internal accounts, small opt-in cohort, broader web availability: controlled rollout lets gates catch issues before wider availability.

- Keeping later age gates behind configuration: adoption expansion can be enabled gradually after call recognition, layout, CPU, and accessibility remain acceptable.

- Rollback by rule version without replacing bird IDs or lowering vectors: rollback preserves continuity and monotonic traits.

- Stop ramp on canonical-state mismatch, persistent audio failure, or accessibility gate failure: these are launch-blocking risks.

- Behavioral acceptance tests: endpoint tests are insufficient; the plan requires scenarios for long-unvisited account, zero background presence, cross-device dedupe, 21-day expression change without numeric UI, two-week absence, captions, visitor limits, revocation, and keyboard/screen-reader/reduced-motion sessions.

### Main risks and responses

- Drift too fast, too slow, or accidentally negative: versioned fixtures, rate caps, monotonic assertions, synthetic cohort review, coefficient rollback, and never recomputing stored vectors from history.

- Presence inflated by inactive or duplicate tabs: three-signal client state, immediate interval close, server bounds, cross-device union, blur/sleep tests, and quiet-watching calibration.

- Multi-device or retry loses identity/drift: single transactional writer, event cursor, idempotency, row lock, stable UUIDs, chaos tests, state-version alarms, and no last-write-wins merge.

- Calls canned, clipped, or blocked by autoplay: stable signature with variation, listening tests, bounded graph/limiter, and caption-first autoplay behavior.

- First paint or long session feels loading or leaking: inline private snapshot, critical scene path, no spinner/fade, defined-device performance CI, and 30-minute endurance.

- Accessibility becomes static or noisy translation: shared semantic state, authored narration cadence, reduced-motion pose system, caption/audio equivalence, and user review as launch gates.

- Visits leak access or change host birds: scoped read-only sessions, owner/visitor API separation, revocation every pull, no visitor presence reporter, auth tests, and token replay/expiry tests.

- Relationship data leaks to telemetry: synthetic UUIDs, encrypted email, schema allowlist, separate credentials, payload scans, no simulation DB-to-warehouse connection, and deletion audit.
