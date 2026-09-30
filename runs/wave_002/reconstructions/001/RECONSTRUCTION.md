## System-level intent

- Durable bird identity is a release gate, not a UI detail. The plan repeats that a bird's "UUID, saved personality vector, and call identity survive" renames, migrations, device changes, outages, absences, backup restore, deletion recovery, seven-bird addition, and rollback. It also says trait drift changes "expression, not identity", and that rollback must never "seeds new birds" or "recomputes vectors from history".
- Canonical server authority protects continuity. The plan says simulation advances "on the server approximately every minute even with zero clients", clients "render time-indexed plans and interpolate", and clients never "run mood transitions, compute drift, or reconcile competing bird states". The process-boundary tables, snapshot contracts, event admission, and tick transaction all preserve this same split.
- Presence is intentionally narrow, honest, and non-invasive. The plan's formula is `visible && document.hasFocus() && trustedPointerOrKeyAge < 240s && !settled && ownerViewActive`, and it excludes open tabs, visitors, polling, audio playback, clicks alone, timers, synthetic events, and screen-reader announcements. It calls this "an honest client protocol with bounds, not invasive attention detection".
- Absence is non-punitive. Section 1 says absence has "no negative personality delta, suffering, hunger, chore, or loss"; section 7 says quietness after return comes from "decayed short-term arousal, not distrust"; deletion recovery advances ordinary mood/time "without presence for the deleted interval". The plan repeatedly forbids guilt, recovery chores, and absence counters.
- Aliveness should feel continuous, not announced. Normal return begins with birds "already mid-action" and one bird noticing within "one to two seconds"; the plan forbids an entry sequence, greeting text, spinner, absence counter, announcement, and fixed three-animation rotation. First-frame rendering, phase-correct handoff, procedural calls, naturalist prose, and reduced motion all serve this same continuity.
- Privacy boundaries are structural. The plan says bird behavior and interaction history remain in the "owner's simulation domain"; analytics cannot query that database or receive its fields. Email is encrypted and never used in foreign keys, log keys, task IDs, URLs, shard keys, or telemetry. Snapshot serializers, telemetry allowlists, export rules, and deletion/purge all enforce the same boundary.
- Product voice is quiet, naturalist, and non-gamified. The plan excludes scores, progress meters, streaks, badges, leaderboards, public discovery, and re-engagement campaigns. It asks for "matter-of-fact copy", "restrained naturalist copy", lowercase present-tense observation prose, no "welcome back", and no "achievement surfaces". It ends by saying feature count, visit frequency, and visible progress are not success criteria.
- Accessibility ships with the normal scene and must preserve affective quality. Section 1 says reduced motion, narration, captioning, keyboard operation, and AA text contrast "ship with the normal scene" and "preserve its affective quality". Later sections require the reduced-motion scene to feel "already alive", accessibility findings to block release, and no inaccessible beta as v1.
- Procedural media preserves recognizable identity without canned assets. The plan forbids recorded-call assets and recorded fallback, uses native WebAudio and species grammars, and requires call identity to remain recognizable as mood and expression change. Captions are generated from the resolved procedural graph, not fixed strings.
- Bounded resource use is part of the product experience. The seven-bird ceiling, 12 KB seven-bird snapshot envelope, 500 ms first visible bird budget, 60 fps idle target, bounded audio voice count, fixed ornament pools, and memory soak all appear as gates so aliveness does not become slow, leaky, or visually crowded.
- Social features are private, named, optional, and read-only. Invitations are explicit, one-time, email-addressed, and absent from onboarding. Visitors see the host's current state through the same renderer but cannot listen in, offer, settle, adopt, rename, read notebook, generate presence, or mutate anything.
- Calibration and release decisions avoid real-owner behavioral analytics. Drift coefficients, performance checks, RUM, rollout gates, and fixtures are synthetic-only or low-cardinality operational data. The plan explicitly rejects actual-user mean drift, engagement funnels, daily active-user streaks, trait rankings, per-bird attention dashboards, and owner drift telemetry.

## Per-feature whys

### Delivery contract and ambiguity decisions

- Browser aviary whose behavior continues while no browser is open: the plan ties this to server simulation that advances every minute with zero clients, so birds remain alive as saved state rather than only animating in an open tab.
- Two starter birds: NOT RECOVERABLE FROM PLAN
- Six species: NOT RECOVERABLE FROM PLAN
- Hard seven-bird ceiling: the plan treats seven as a recognizability and resource boundary, with seven-bird listening tests, caption collision checks, frame/memory budgets, an atomic database cap, and a rollout that delays higher counts until gates pass.
- Magic-link accounts: NOT RECOVERABLE FROM PLAN
- One canonical aviary per owner: the plan uses this to avoid multiple aviaries, shared accounts, personality merges, competing canonical mornings, and verification creating a second aviary.
- Multi-device viewing: the plan wants laptop and phone to see the same mood and weather without vector merges, while overlapping owner devices "count once" and device-local listen-in does not seize another device's mix.
- Read-only field notebook: the rationale is to preserve sparse naturalist observations without turning private history into a raw event feed, comments surface, unread counter, milestone, score, or attendance record.
- Optional invitations: the rationale is quiet private sharing through a named, one-time, read-only visit while preventing co-presence, public discovery, host mutations, presence credit, or visitor rankings.
- Complete accessible experience: the plan makes accessibility a v1 release gate so reduced motion, captions, narration, keyboard operation, and contrast are not later add-ons and retain the normal scene's affective quality.
- Hidden numerical traits with export exception: the plan hides trait numbers from product views, narration, captions, ordinary APIs, and client debugging so the experience does not become a statistics viewer; exact vectors appear only in the narrow machine-readable portability export.
- Optional visit-notification setting: it is off by default and absent from onboarding so visit mail stays a "sole optional activity-mail exception" and never becomes badges, toasts, push, reminders, or engagement copy.
- Four top-bar icons with settle inside the offer actions panel: the plan keeps exactly account/settings, accessibility, notebook, and offer icons while still making settle reachable without a fifth icon.
- Keyboard focus and Enter for listen-in: keyboard bird focus engages listen-in for accessibility, Enter is idempotent, Escape disengages while retaining focus, and pointer click toggles independently so incidental DOM focus does not create conflicting behavior.
- Single saved IANA timezone: the plan saves the owner's chosen timezone to keep server and clients, including visitors, on one shared day/night state and to avoid "competing canonical mornings".
- Prompt reactions through accepted server plans: the plan returns a short presentation plan immediately for visible responsiveness, then lets the next normal tick apply canonical mood/drift effects, avoiding client speculation about mood or vectors.
- Settled state across devices: settle ends the initiating view's presence and creates an aviary-wide evening override, while polling alone cannot undo it and other legitimate owner presence is neither erased nor double-counted.
- Provisional recipient identity for visitor email: the plan reuses the identity/account record as the sole encrypted email store, lets invitations reference recipient UUIDs, and gives provisional visitors no aviary or owner permissions.
- One-time invitation lifetime: unused links expire at 30 days and redemption grants one read-only browser visit so reopening a consumed link cannot start another visit.
- Age-based adoption pacing: slots depend only on creation time, never attention, so missing an offer loses nothing and future birds do not become a reward for visiting.
- Silence preference and evolution: muting is an output preference, not neglect; qualified presence and focused attention count equally with audio, captions, or silence.

### Service architecture and data ownership

- TypeScript web client: NOT RECOVERABLE FROM PLAN
- TypeScript HTTP application: NOT RECOVERABLE FROM PLAN
- Canvas2D drawing with compact SVG bird rigs: the plan uses it for continuous scene drawing, compact reusable rigs, bounded seven-bird complexity, phase-correct first-frame handoff, and semantic DOM controls outside the canvas.
- Native WebAudio synthesis: the rationale is procedural call identity with no recorded assets or recorded fallback, plus captions driven by the same resolved call graph.
- Separate simulation-worker process: the plan gives the worker sole responsibility for ordered event consumption, additive drift, mood, weather, behavior, call plans, and notebook observations, keeping clients and HTTP handlers from writing canonical personality.
- PostgreSQL as transactional authority: the plan relies on row locks, ordered sequences, atomic watermark/delta commits, fenced scheduler work, and restoration drills to prevent double effects and preserve exact vectors.
- Object store for short-lived exports: export artifacts are private, expire within 24 hours, and are excluded from analytics.
- Transactional email adapter and outbox: required auth, verification, invite, export, and optional visit emails are queued with synthetic IDs and resolve encrypted email only inside the identity/mail boundary.
- Regional edge private initial snapshot: the plan uses edge-authenticated HTML assembly with a compact private snapshot to meet first visible bird timing without public snapshot caching.
- CDN-cached public content-addressed rigs, motifs, and JavaScript: only public immutable assets are shared-cacheable; personalized HTML and snapshots are private so owner state does not enter a public CDN cache.
- One writable database region for first release: the plan chooses this before regional writes so no client-to-client protocol, CRDT, websocket dependency, or eventual personality merge is needed.
- Process boundaries and database roles: the plan assigns forbidden responsibilities so email/logging tools cannot receive simulation inputs, owner APIs cannot accept vectors, visit APIs cannot submit owner events, and telemetry cannot access behavioral payloads.
- Snapshot serializer allowlist: stripping vectors, signal filters, cooldown internals, invitation details, and unnecessary account identifiers turns privacy and permission rules into integration tests rather than conventions.
- Presentation contracts without personality vectors: timestamps, seeds, perceptual styles, poses, and call structures give the browser enough to render without giving it a simulation engine.
- Random UUIDs, encrypted email, and no email-derived keys outside identity storage: the rationale is to keep email out of foreign keys, logs, URLs, shard keys, telemetry, and external identifiers.
- Revocable device sessions: hashed opaque session secrets, expiry, revocation, and coarse browser/device descriptions support multi-device viewing without raw user-agent strings in aggregate telemetry.
- Append-only interaction events with idempotency: per-aviary sequences and event UUID uniqueness make retries safe and prevent another log row, cooldown, mood effect, or drift input.
- Presence coverage masks: per-second masks allow overlapping windows/devices/retries to union once and keep private simulation inputs only for the rolling input window.
- View leases: short-lived server-expiring leases bound open views, heartbeat, listen-in bird, settle state, and greeting epoch without creating an owner visit calendar.
- Bird observation summaries: bounded private windows support notebook selection while excluding owner attendance streaks and analytics input.
- Adoption offers as one visible pending offer with no rarity or score: this keeps adoption quiet, system-chosen, age-gated, and non-gamified.
- Trait storage with normalized headroom and stable call signature/silhouette/UUID: the plan keeps growth finite and nondecreasing while ensuring drift changes expression rather than identity.
- Event, coverage, and observation retention windows: 30-day raw event retention, 48-hour coverage, bounded summaries, sparse unlimited notebook records, and account deletion override balance recovery, rolling inputs, notebook history, and privacy.

### HTTP contracts and authorization

- Same-origin JSON APIs, secure cookies, CSRF, schemas, bounded payloads, and no request/snapshot/body logging: the plan uses these to protect account and aviary state while keeping errors matter-of-fact and non-leaky.
- Uniform magic-link request responses: the stated purpose is avoiding account enumeration.
- Settings revisions and 409 on conflicts: metadata conflicts reload and retry specific edits rather than creating vector merges.
- Snapshot authorization, including would-be 304 responses: the plan prevents revoked sessions or visitors from receiving cached/private host state.
- View-open route separate from presence credit: opening or returning can request a greeting, but it records only an observation opportunity, not attention.
- Event route rejecting unknown types, traits, positions, absolute mood, and foreign bird IDs: this preserves server authority over semantic outcomes and canonical state.
- Adoption-offer endpoint without numeric progress/countdown: the plan keeps future birds from becoming a progress surface or expiring reward.
- Separate owner-session and visit-session middleware: visitor cookies cannot be promoted into owner actions by route parameters or submitted event types.
- Visitor snapshots omitting owner settings, account identity, cooldowns, notebook, and adoption information: the rationale is read-only viewing without host private controls or system surfaces.
- Snapshot horizon and size envelope: a 120-second behavior/call horizon and seven-bird 12 KB gzipped cap keep snapshots small and avoid attaching event logs or notebook pages to every pull.
- Interaction envelope with server timestamps and authoritative sequence: client timestamps are bounded hints only, so simulation order comes from server receipt and per-aviary sequence.
- Idempotent event receipts and plans: retrying the same event ID returns the exact original result so lost responses do not duplicate outcomes or seeds.

### Presence, simulation, and recovery

- 240-second recent-activity window: the plan says it is "intentionally long enough for sitting quietly".
- Strict browser qualification formula: presence requires visible document, focused window, recent trusted pointer/key activity, not settled, and active owner view so open tabs, audio, polling, and visitors do not create attention.
- Excluding focus, clicks alone, media playback, polling, timers, synthetic events, scroll observers, and screen-reader announcements: the plan prevents incidental browser behavior from becoming presence.
- Not recording pointer coordinates or key values: only a monotonic last-activity timestamp is retained for privacy.
- Hidden tab, suspension, and offline handling: the plan stops animation/audio/presence/polling in hidden tabs, discards uncertain sleep gaps, avoids overnight presence backlogs, and never infers future attention from heartbeats.
- Server-side admission checks for presence: because the client report cannot be cryptographically proven, the server bounds the honest protocol with session/view validity, schema, positive duration, activity, flags, sequence, and timestamp envelopes.
- Coverage union and listen-in masks: the plan credits overlapping intervals only once, intersects listen-in with qualified owner presence, and uses a shared per-aviary interaction budget so two devices cannot multiply reinforcement.
- Return and greeting epochs: the bird can notice a quiet owner before pointer movement, repeated focus churn is debounced, absence length stays private, and greeting plans are saved so retries stay stable.
- Sixty-second server scheduling with stable UUID staggering: this keeps aviaries advancing without clients while avoiding minute-boundary spikes.
- Atomic tick transaction with high-water mark: loading state, consuming ordered events, applying nonnegative deltas, advancing mood/weather/plans, notebook candidates, and revisions in one commit prevents double application after crashes.
- Slow drift function: persisted nonnegative filtered input, 85 percent presence weighting, headroom, and daily perceptual budgets create gradual growth without same-session jumps, absence penalties, or client-provided absolute values.
- Mood continuity: stored mood, minimum dwell, daily-ish relaxation, time-of-day, weather, personality, and nearby birds avoid tab-open or midnight neutral resets and avoid using owner absence as a wary penalty.
- Behavior and weather plans: server-owned perch, flight, call, rain, and wind seeds make every device see the same bird/weather events while render-only ornaments remain local and nonsemantic.
- Catch-up, versioning, and recovery: missed work advances saved state through bounded chronological recovery, never lazy simulation on return, never invented absence presence, never a flood of retrospective notebook entries, and never reseeding birds on engine changes.

### Sync, interactions, adoption, and scene rendering

- Snapshot polling on navigation, visible return, resume, and every 15 seconds while visible: the plan keeps visible clients current while using ETags and revision fences so unchanged pulls are cheap and post-event reads do not fall behind receipts.
- Persisted short presentation plans for greetings and offer reactions: they give immediate gestures without waiting for the next minute tick, while the next tick consumes the same outcome instead of rerolling.
- Local-only device intent: listen-in focus, open panels, keyboard focus, volume hardware, and transient request progress remain local so one device does not seize another device's mix.
- Resume and failure horizons: expired calls are skipped, small corrections are interpolated, long suspensions jump to current pose, and failed snapshots degrade to known poses/quiet ambience rather than inventing semantic calls or resetting birds.
- Listen-in: the rationale is focused attention with a gradual audio mix, nonzero ambient floor, captions/narration parity, and credit only for qualified attention rather than stale focus lifetime.
- Offers: seed, song fragment, and still pool create unobtrusive scene reactions chosen by the server from saved state and cooldowns; drowsy watching is not failure, and restrained copy avoids timers, red errors, and progress bars.
- Settle and undo: settle creates an evening-light quieting transition, cuts initiating-view presence immediately, delays semantic quieting until the five-second undo window, and never changes slow traits or creates a recovery chore.
- Starter adoption flow: first activation creates one aviary and two system-selected distinct species in one transaction, with the first empty field and soft fly-in as the only entry exception; reload after creation shows the same birds rather than replacements.
- Future adoption offers: age milestones, one pending bird, quiet placement in offer panel/settings, no badge/popup/email/count, no adoption rush, and atomic cap enforcement keep adoption independent of attention and non-gamified.
- Rename: validated names update only name/revision, preserving UUID, call seed, traits, mood, history, and adoption age.
- Single logical scene without panning, zoom, placement controls, hover labels, mood icons, or badges: the plan keeps the aviary calm and not game-like while browser zoom remains available.
- Responsive perch zones and fixed logical slots: the plan preserves front/middle/back meaning, avoids cropping, keeps minimum hit targets where possible, and fits all birds on a narrow phone without moving a wary bird forward merely to fit.
- First-frame continuity: private snapshot, server-rendered current poses, tiny bootstrap, and matching Canvas handoff produce a bird already in motion under the 500 ms target without a static default pose, spinner, or loading animation.
- Four-icon top-bar fade: fading reduces visual interruption after stillness while pointer/key/touch/open panel/focus restore keeps controls readable and avoids accidental touch activation.
- Reduced-motion renderer: the same semantic snapshot cross-fades designed still poses and slow lighting/weather so reduced-motion users receive an alive scene, not a static inert picture or separate simulation.

### Calls, notebook, accessibility, visits, lifecycle, and rollout

- Procedural call grammar with stable bird signatures: species grammars and within-species signatures make calls recognizable across mood and personality changes while per-call seeds avoid canned repetition.
- Chorus scheduling with stable call IDs: response edges and overlap limits create plausible chorus while preserving space for seven recognizable birds instead of a wall of sound.
- Bounded WebAudio graph: one AudioContext, voice pools, per-bird buses, gain ramps, soft limiter, and node cleanup prevent backlog, clipping, clicks, and memory growth.
- Graceful silence around browser audio permissions: captions and visuals continue when autoplay, WebAudio, or hardware fail; audio begins only on deliberate enable and muting never penalizes drift.
- Captions generated from the resolved procedural graph: caption text matches the actual syllable count, contour, timing, intensity, and context, and collision-aware layout keeps captions readable without overloading assertive live regions.
- Authored naturalist observation grammar instead of an external language model or raw event dump: this preserves voice and keeps private history out of third-party text-generation systems.
- Notebook fact validation and sparsity: novelty scoring and cadence limits make entries specific and occasional, not one entry per session, outage backfill, owner attendance count, or trait-number report.
- Read-only host-only notebook pagination: cursor pages, virtualization, no edits/deletes/comments, no unread counter, and indefinite history keep it a private field notebook rather than a social feed or task list.
- Screen-reader scene experience: semantic bird targets and shared naturalist grammar describe action and perch without visible labels, normalized traits, mood status lists, or canvas internals.
- Narration cadence and queue behavior: a polite atomic region every 45 seconds, coalescing, prompt observations, and no "welcome back" avoid stale queued prose and semantic state dumping.
- Keyboard and focus model: four top-bar controls, one roving scene bird, arrow movement, Enter listen-in, Escape release, focus restoration, and no pointer-only hover behavior make all core surfaces operable.
- Visitor accessibility controls without host actions: visitors can adjust local captions/reduced-motion/audio output while assistive-technology focus cannot create host presence or greetings.
- Contrast, text zoom, touch target, forced-color, and manual screen-reader checks: the plan treats dynamic lighting, night, captions, panels, and reduced motion as release-blocking accessibility surfaces.
- Explicit email invitations and one-time bearer links: host-initiated named invites avoid onboarding sharing prompts, public links, scene visitor lists, badges, and implicit profile features.
- Read-only visits through the same renderer: visitors see host-local day/night, weather, and existing owner-triggered plans without beautification, greeting, shared cursor, host marker, or owner capabilities.
- Visit heartbeats and revocation: duration logging stays separate from simulation presence, and revocation/expiry authorize before 304 or rendering so access is removed on the next pull and host state is cleared from DOM and memory.
- Host visit log and optional visit mail: the log provides private transparency, while optional mail is one throttled quiet redemption email and default behavior is silent logging.
- Account export: the plan includes exact current personality vectors only as a portability exception from a consistent revision, while excluding raw presence, device secrets, tokens, invitation tokens, and attendance history.
- Thirty-day deletion lifecycle: immediate mark/revocation/pause plus a restoration surface protects recovery, then day-30 purge and backup key destruction protect privacy without pretending immutable backups vanish instantly.
- Telemetry allowlist: request counts, latency/error distributions, first-bird timings, frame/memory samples, audio failures, tick duration/lag, and anonymous duration histograms support operations without behavioral fields, identifiers, names, mood, traits, call seeds, notebook content, or email.
- Performance budgets: byte, first-bird, payload, frame, audio, memory, server, and hidden-tab gates make the calm living scene feasible on physical mobile and five-year laptop profiles without redefining slow results as success.
- Verification strategy: deterministic fixtures, synthetic-only accounts, database transaction tests, browser end-to-end tests, fault injection, listening by ear, and manual prose review are required because numerical assertions alone cannot certify aliveness, calmness, recognizability, or voice.
- Six work packages: the plan sequences contracts/prototypes, durable simulation, client/audio, notebook/lifecycle, visits/hardening, and staged release so interfaces and gates exist before broad launch.
- Staged rollout and increasing bird maxima: internal/synthetic, private beta, cohorts, and maxima of three/five/seven advance only after operational, accessibility, first-frame, memory, recognizability, and resource gates pass, preserving accrued eligibility and never downgrading existing bird counts.
- Kill switches and rollback: separate switches for invitations, future adoption, offer admission, optional mail, and engine pinning allow severe incidents to freeze writes and serve last safe state without recomputing vectors, reseeding birds, discarding seven-bird scenes, or falling back to recorded calls.
