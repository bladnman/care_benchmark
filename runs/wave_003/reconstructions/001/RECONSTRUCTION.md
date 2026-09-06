## System-level intent

- Preserve a small private place with owner-bound scope. This appears in "private aviary," "one owner and one canonical aviary per activated account," the exclusion of public/social surfaces, and the final line that the product must remain "a small private place whose birds notice the owner."

- Make bird identity durable and identity-preserving across time. This appears in "durable identities," "Bird UUID, adopted_at, and voice identity survive renaming, code deployment, and schema migration," and the release requirement for "slow, nonnegative, identity-preserving evolution."

- Put canonical life on the server, not in the client. This appears in "continue changing on the server while nobody watches," "Normal idle time is real server simulation, not catch-up on GET," "clients never decide outcomes," and "There is no client simulation."

- Treat qualified, idle attention as the main interaction. This appears in "Its main interaction is qualified, idle attention," the qualified presence predicate, and the phrase "honest presence" in the definition of complete v1.

- Keep absence non-punitive. This appears in the exclusions of "deaths, hunger, distress caused by absence," "negative personality drift," and the direct rule that "Absence does not enter a transition toward wary, sadness, illness, or lower traits."

- Express state convincingly but restrainedly. This appears in "convincing, restrained expression," "varied noticing gesture," "one subtle greeting," "no synchronized welcome chorus," and the instruction to use a "restrained inline offer-panel state" rather than failure or reward surfaces.

- Avoid engagement-game framing. This appears in the exclusions of "progress dashboard," "streak," "badges," "achievements," "rewards for activity," and "engagement-driven unlocks," plus the rollout rule to not use interaction analytics to decide which features are "engaging."

- Keep hidden numeric state hidden from product surfaces. This appears in "never expose raw vectors," "There is no in-app export preview or trait visualization," "no numeric trait display," and the DTO rule to never serialize traits or drift accumulators into the client snapshot except the explicit export.

- Make accessibility, captions, reduced motion, and narration first-release obligations. This appears in "from the first release," "not final-stage retrofits," "same relationship/identity review," and the release stop language around "serious accessibility regression."

- Separate relationship data from operational data. This appears in "Simulation interaction data is used only to advance that account's aviary and produce its own notebook," "telemetry schema allowlist," "no DB-to-warehouse path," and "no model-training connection."

- Keep system obligations plain and outside the scene. This appears in "system obligations handled plainly outside that scene," "matter-of-fact human copy," "direct system language," and the instruction that errors, account, privacy, and accessibility settings use normal sentence casing.

- Prove aliveness and correctness with evidence, not proxies. This appears in "No final acceptance substitutes aggregate engagement, a feature count, or a successful build," the release evidence requirements, and the use of synthetic fixtures plus explicit design/accessibility review.

- Make performance part of the product meaning. This appears in "A visit begins with birds already doing something," "First real bird" under 500 ms, the risk "First paint feels like a loading app," and the rule to "never add spinner/entry choreography."

## Per-feature whys

### Product boundary and binding decisions

- Modern browser application with one owner, one canonical aviary, and revocable browser sessions: The plan ties this to a "private aviary," canonical durable state, owner multi-device access, and security through independently revocable sessions.

- Email magic links as the chosen sign-in mechanism: NOT RECOVERABLE FROM PLAN

- Two system-assigned starter birds as the exact starting count: NOT RECOVERABLE FROM PLAN

- Approximately six authored species as the exact species count: NOT RECOVERABLE FROM PLAN

- Age-based adoption up to seven birds: The plan's rationale is gradual expansion without activity rewards: eligibility uses "aviary age alone," real accounts gain opportunities only when "actual age qualifies," and rollout must not reward "active testers."

- Persisted five-dimensional personality, daily-scale mood, behavior, calls, lighting, weather, and motion: The plan uses these to create slow aliveness, "daily-scale mood," "continuous expressive motion," and evolution that remains the "same bird."

- Return-greeting: The plan says a visit begins with birds already doing something, "followed by one bird noticing the returning owner"; the greeting proves arrival recognition without making presence credit a prerequisite.

- Listen-in: The plan frames listen-in as a local attention target and mix change that lets the owner attend to a bird without commanding calls, showing labels, or making a reward mechanic.

- Three offers: The plan gives offers as meaningful but bounded interactions whose server-authored reactions can produce mood response and notebook facts without food, inventory, hunger, or task loops.

- Settle with five-second undo: The plan's reason is a quiet departure ritual: it "immediately ends this view's qualified interval," applies local evening/quieting, and allows quick reversal without bonus drift, streak credit, or penalty.

- Precise presence accounting: The plan ties this to "honest presence," multi-device union instead of addition, and preventing invented attention from open tabs, missing terminal events, or stale reports.

- Sparse read-only field notebook: The plan's rationale is a private "field notebook" of bird observations, not a feed, owner behavior log, unread system, or editable history.

- Owner multi-device access: The plan uses this to let a second device read the "same durable birds and moods" while keeping local choices such as mute, listen-in mix, and settle overlay from forking shared state.

- Verified email changes, export, recoverable deletion, hard deletion, and privacy link: The plan treats these as account lifecycle, portability, and erasure obligations handled in system surfaces without disturbing the scene.

- One-time read-only visit invitations: The plan's rationale is quiet private sharing that "never grants owner scope," creates no visitor interaction events, and can be revoked.

- Optional visit-notification setting: The plan preserves explicit opt-in while avoiding re-engagement channels: notices are live, matter-of-fact, and only inside the open account/settings visit-log surface.

- Screen-reader prose, call captions, keyboard operation, reduced motion, contrast, and audio-unavailable behavior: The plan makes these first-release requirements so the experience still feels like "birds in a place" across sensory and input paths.

- Synthetic performance coverage, aggregate operational telemetry, bounded resources, and recovery procedures: The plan uses these to prove build health, correctness, performance, and recovery without using relationship data or engagement analytics.

- Excluding deaths, hunger, public feeds, rewards, reminders, recorded calls, warehouses, and model training: The plan uses exclusions to protect the non-punitive, private, non-game, non-surveillance character of the product.

### Decisions where interpretation is needed

- Raw-vector prohibition with export exception: The plan's why is to keep normal HTML, snapshots, ARIA, settings, support views, and telemetry free of raw vectors while honoring the specifically required protected JSON portability export.

- Four top-bar icons with settle inside the offer popover: The plan's reason is to keep "exactly account/settings, accessibility, notebook, and offer icons" while still making settle reachable from the top bar.

- Keyboard focus and Enter behavior for listen-in: The plan resolves ambiguity by making focus begin listen-in for keyboard operation, while Enter is an idempotent explicit engagement and Escape disengages without losing focus.

- One persisted IANA aviary timezone: The plan's why is a single state across devices and visitors, avoiding travel silently changing mood/lighting while still allowing an explicit settings update.

- Immediate greeting/reaction programs separate from minute ticks: The plan uses command-handler envelopes so reactions appear in "one to two seconds" while persistent personality and mood still change only in the tick.

- Starter fly-in only on real first adoption: The plan keeps ordinary navigation as "current motion" and prevents repeated onboarding or entry replay.

- Nearly transparent top bar with tested contrast: The plan wants decorative UI to fade away while focused controls, actionable glyphs, and user-copy text keep contrast.

- Settle across devices: The plan keeps settle as a local presentation and presence choice for the initiating view, with only a small shared mood-quieting influence, so other owner views continue independently.

- Muting without drift effects: The plan says mute changes presentation and can enable captions, but never decreases traits, counts as negative attention, changes voice identity, or rewards audio-on behavior.

- The exact five-minute, 15-second, three-minute, 60-second, and adoption-age constants: NOT RECOVERABLE FROM PLAN

### Architecture and ownership

- TypeScript specifically as the web stack: NOT RECOVERABLE FROM PLAN

- Small DOM/SVG client, HTTP service, simulation workers, PostgreSQL canonical storage, and minimal Preact shell: The plan's rationale is a small application where the animation loop avoids component reconciliation, supports first-bird performance, and avoids a 3D engine, game framework, or recorded-media pipeline.

- Authenticated edge HTML with embedded snapshot, critical SVG/CSS, and tiny motion bootstrap: The plan uses this so the first response contains actual authorized birds already in current motion, while personalized HTML stays out of shared public caches.

- One primary database region and one logical writer per aviary: The plan's rationale is canonical truth and no conflicting personality writes; scale read delivery and worker partitions before introducing another write region.

- Durable queue with PostgreSQL due times, locks, and committed versions as authority: The plan lets queues wake workers but prevents queue delivery from becoming canonical truth.

- Separate Web/API, simulation, mail, lifecycle, and metrics permissions: The plan uses role separation so Web/API cannot update personality, mail workers do not receive bird history, lifecycle handles erasure, and metrics cannot reach simulation records.

- Module boundaries for domain, simulation, interaction, projection, runtime, rendering, audio, accessibility, and account/social surfaces: The plan uses these boundaries to prevent HTTP/DOM/audio/analytics from owning drift, adoption, or authoritative random outcomes.

- Export-safe versus client-safe DTO separation: The plan's rationale is that derived drawing/synthesis parameters can exist without shipping a hidden client-side personality object.

- Authoritative versus local state split: The plan uses this to keep UUIDs, traits, mood, schedules, cooldowns, notebook, and adoption canonical while focus, mix, mute, panels, geometry, and settle overlay remain per-view.

- No client simulation, client-to-client protocol, absolute personality update, or state merge: The plan uses this to prevent local changes from overwriting or forking a bird.

- In-memory live snapshots cleared on logout, account change, or visit termination: The plan says persistent browser cache is unnecessary for v1 and clearing private state protects account/session boundaries.

### Data model and transactional invariants

- UUIDs for accounts, aviaries, birds, sessions, invitations, jobs, and domain events as the specific identifier form: NOT RECOVERABLE FROM PLAN

- UTC timestamps with IANA timezone conversion only for behavior and prose: The plan uses this to keep storage consistent while making local-time lighting, mood, and prose follow the canonical aviary timezone.

- Names as display values, never identifiers: The plan's why is that renaming must not change identity, behavior, stable references, or old notebook entries.

- Encrypted email plus restricted keyed email-lookup digest: The plan uses this for lookup and uniqueness while preventing email from becoming a plaintext field, service identifier, metric dimension, shard key, or log field.

- Email-only invitee identity with synthetic account UUID: The plan's rationale is allowing invitations and host visit logs to reference a UUID without duplicating email or creating an aviary or owner permission.

- Pending new email stored separately until verification: The plan uses this so the old address remains the valid destination and sign-in address until uniqueness and verification commit atomically.

- One activated owner account has exactly one aviary and creation retries cannot duplicate starter birds: The plan uses this to enforce canonical ownership and prevent retries from creating a second aviary or extra starters.

- Bird UUID, adopted_at, voice identity, visual seed, and personality draw survive renaming, deployment, and migration: The plan uses these to ensure the bird remains the same bird rather than being reseeded or replaced.

- Traits bounded in [0,1] with additive server-computed nonnegative deltas: The plan's rationale is no negative personality drift and no API accepting an absolute client vector.

- Atomic tick commit of cursor, vector changes, moods, program, notebook additions, and revision: The plan uses this so events cannot be credited twice or disappear between revisions.

- Per-aviary sequence assignment serialized with commits: The plan's reason is to avoid a global high-water mark skipping a lower sequence from an uncommitted concurrent transaction.

- Presence intervals unioned across owner devices and visitor time excluded: The plan uses this to prevent multi-device attention inflation and keep read-only visits out of the simulation ledger.

- Atomic invitation consumption, expiry, revocation, and grant creation: The plan uses this so visit authorization is checked on every response and cannot accidentally become owner scope.

- Append-only notebook with no update/delete endpoints: The plan treats notebook history as indefinitely accessible, with whole-account deletion as the explicit exception.

- Raw simulation-event retention after consumption, idempotency receipt expiry, presence coverage, and factual windows: The plan's rationale is to fold watermarks and observation facts durably, prevent replayed credit after pruning, and avoid an owner visitation calendar.

- The exact 14-day, 48-hour, 24-hour, seven-day, and related retention durations: NOT RECOVERABLE FROM PLAN

- Durable commits, encrypted backups, point-in-time recovery, and fenced failover: The plan uses these to target zero acknowledged data loss and restore persisted state without regenerating birds from historical events.

### API contract

- Scanner-resistant magic-link landing and consume flow: The plan's reason is that email-scanner GET requests must not mutate state or consume sign-in links; a small Continue action protects the token.

- The exact initial limit of five magic-link requests per address per 15 minutes: NOT RECOVERABLE FROM PLAN

- Authoritative session revocation with secure host-bound cookies: The plan uses this so revocation takes effect on later requests rather than trusting a long-lived token alone.

- The exact 30-day idle and 90-day absolute session limits: NOT RECOVERABLE FROM PLAN

- Settings updates with expected settings_revision: The plan's rationale is avoiding hidden overwrites by returning 409 and current settings for stale edits.

- Account export job and authorized expiring download: The plan uses this for consistent protected portability, mailed only to the verified address, without links or secrets in logs.

- Soft deletion and recovery routes: The plan's reason is an immediate deletion mark with a 30-day recovery window where the same account, aviary, and birds can be restored by an explicit action.

- Authorized aviary snapshot with ETag and revision data: The plan uses this to deliver mutually consistent current presentation state while avoiding unnecessary full payloads.

- Bounded batch event route with idempotency and receipt lookup: The plan uses this to handle connection loss without replaying or minting a fresh offer ID for an uncertain write.

- Owner-only notebook pagination without a count: The plan's rationale is unbounded backwards history while avoiding a pagination count that resembles a score.

- Bird settings, renaming, and adoption routes without traits or rarity: The plan uses these to expose identity, species description, name, and opportunities without turning internal state into stats or catalog gameplay.

- Event types view_return, view_leave, presence_interval, listen_start, listen_end, offer, settle, settle_undo, and reengage: The plan uses them to separate greetings/view lifetime, sustained presence, listen spans, offers, and settle state so none implies another effect accidentally.

- Receipts and cooldown copy: The plan's why is calm product voice; unavailable/cooldown is "not a failure toast, countdown, or loss."

- Visit invitation, consume, current snapshot, and end routes: The plan uses these for deliberate one-time visits, read-only projection, revocation, and duration accounting without simulation events.

- The exact 30-day unused invitation expiry and 24-hour visit-grant maximum: NOT RECOVERABLE FROM PLAN

- Visitor local controls for mute, captions, reduced motion, and narration only: The plan's rationale is visitors can adapt presentation locally while birds are not listen-in buttons and no owner capability or simulation input exists.

### Simulation and temporal continuity

- Ticks every roughly 60 seconds for every existing non-hard-deleted aviary: The plan uses this to make server life continue for inactive accounts and to size throughput from total aviaries rather than concurrent viewers.

- Stable UUID-derived tick offsets, batches, and short per-aviary transactions: The plan's rationale is to spread work, avoid email-based scheduling, and prevent a burst of views from blocking a minute's simulation.

- Tick algorithm reading state/events and committing state, cursor, schedule, notebook, outbox, and revision together: The plan uses this to integrate presence, offers, settle, moods, weather, behavior, notebook, and adoption atomically.

- No GET catch-up and fixed-step outage recovery: The plan's reason is that reads must not trigger competing ticks, and recovery should advance actual state without old call playback or bulk notebook creation.

- Throughput sizing by total aviaries: The plan uses this because inactive accounts still tick, so capacity cannot be planned from active viewers alone.

- Personality drift formula with attention exposure, listen-in, and offers: The plan's rationale is slow positive evolution from qualified owner presence, with ordinary attention dominant over repeated actions.

- Hidden saturation controls and exposure caps: The plan states these are "never tasks or allowances to show the user," preventing progress mechanics while bounding repeated interactions.

- Absence tail without baseline decay: The plan uses prior-input filter tails to allow continued drift from earlier attention, but with no input the existing personality stays intact and never decays toward baseline.

- Calibration around one-week and three-week perception: The plan's reason is visible long-term change that is "recognizable but unobtrusive," with no visible change inside a normal single session.

- The exact 85/10/5 weights, coefficient ranges, calibration numeric ranges, and formula constants: NOT RECOVERABLE FROM PLAN

- Mood enum with wary, content, curious, drowsy, and alert: The plan uses these as internal expression states while sleeping/settled remain poses and lighting/mix conditions, not extra meters.

- Baseline mood reconsideration over 12-24 hours with jitter: The plan's reason is to avoid mood changes at navigation or synchronized midnight reset.

- Absence not causing wary, sadness, illness, or lower traits: The plan uses this to keep longer absence a change in re-orientation form, not a penalty or guilt surface.

- Server-authored behavior programs with perch, pose, calls, and social responses: The plan uses these to keep current motion deterministic, collision-aware, and authoritative while preventing client perch commands.

- Local-time lighting, persisted timezone, and night-active species: The plan's rationale is a shared day/night state across devices with active nighttime life, without travel silently resetting schedules.

- Server-generated rare weather without real-world API: The plan uses this for ambient variation without location/weather obligations, severe events, or notifications.

- The exact weather rate and duration, such as two to three rains per week for five to twelve minutes: NOT RECOVERABLE FROM PLAN

- Client-only leaves, feathers, and parallax ornaments: The plan's rationale is decoration without server state, mood effects, notebook facts, or reduced-motion burden.

- Immediate server-authored reaction envelopes: The plan uses these to make greetings/offers prompt while preserving server authority and consuming outcomes exactly once in the next tick.

- Greeting selection and variation: The plan's why is one subtle, varied noticing gesture within one to two seconds, with no mechanical repeat and no synchronized welcome chorus.

### Presence and interaction state machines

- Qualified presence predicate using visibility, focus, recent trusted pointer/key activity, and not-settled state: The plan uses this as an "honest approximation of attention" with no invasive monitoring.

- Five-minute activity window: The plan says it lets someone "watch quietly without constant movement" while bounding credit for a laptop left open.

- Presence reporting, validation, clamping, rejection watermarks, and unioning: The plan's rationale is to prevent huge, future, duplicate, historical, or multi-device-inflated durations from becoming exposure.

- Not collecting pointer positions, key values, or text: The plan's reason is to avoid invasive monitoring while using only enough signal for qualified presence.

- Listen-in state machine for pointer, keyboard focus, Enter, Escape, blur, and empty-scene clicks: The plan uses this to keep one local attention target, complete keyboard operation, and avoid accidental re-engagement through focus handlers.

- Listen-in duration bounded by qualified presence and split across simultaneous devices: The plan's rationale is to prevent opening more devices from doubling total effect.

- Offer popover with seed, song fragment, still pool, optional recipient, cooldown, and server-authored reactions: The plan uses this for meaningful bird reactions without food quantity, inventory, hunger, recorded clips, countdowns, or stale availability races.

- Settle, undo, and ordinary departure: The plan's rationale is a local quieting ritual that ends presence immediately, supports a five-second reversal, and never rolls back personality, adds bonus drift, or penalizes closing.

### Snapshot consumption, outages, and concurrency

- Snapshot pull cadence, ETags, observed revisions, and event-head reconciliation: The plan uses these to keep canonical state current without erasing acknowledged reactions, replaying offers, or accepting old out-of-order responses.

- Interpolation and monotonic clock handling: The plan's rationale is pose/perch continuity and current-phase sampling instead of teleporting, replaying missed calls, or bursting old narration after sleep.

- Bounded in-memory retry queue and no offline bird state: The plan uses this to avoid replayable offline activity logs, invented calls, invented reactions, mood transitions, or accumulated offline interactions.

- Revoked session and unknown write handling: The plan's rationale is to stop private state/sound on authorization loss and resolve uncertain writes by original event ID rather than creating duplicate outcomes.

- Visitor revocation checks and short display lease: The plan uses this so a visit cannot become an unrevocable cached viewing session.

### Frontend rendering and scene interaction

- Inline authorized SVG first paint: The plan's rationale is that the first visible bird must be an "actual authorized bird" already sampled in current action, not a placeholder, spinner, neutral pose, fade-in, or replay.

- Quiet sky-colored field on genuinely slow state delivery: The plan treats this as truthful failure presentation rather than a substitute for the first-bird budget.

- Onboarding fly-in only once after account creation: The plan's why is that account creation is the sole empty-aviary transition, and reloads must not repeat it.

- Root SVG scene graph with bounded node budget and reusable shapes: The plan uses this for performance, identity visible in silhouette/pattern/voice, and avoiding raster stacks.

- Responsive anchor sets, collision-free layout, and no scene scroll/pan/zoom requirement: The plan's rationale is fitting up to seven birds, hit regions, outlines, wings, paths, and captions across phone and desktop fixtures.

- No permanent scene labels, badges, draggable birds, or hover-only action: The plan uses this to keep the scene visually quiet; captions and focus outlines are explicit accessibility exceptions.

- Top-bar fading and touch/focus visibility rules: The plan's why is to let the interface recede while keeping controls discoverable, focused, accessible, and visible for touch.

- Offer popover, account/settings, accessibility, and notebook surfaces: The plan uses these to keep system obligations and choices outside scene chrome while avoiding unread badges and extra icons.

- Reduced motion as a full renderer: The plan's rationale is preserving identity, greeting, offers, moods, audio, and relationship review without flashing normal motion or accepting a static replacement scene.

### Procedural audio and caption derivation

- Stable voice fingerprint for each adopted bird: The plan uses this so birds remain recognizable across mood, drift, and repeated species.

- Procedural calls, motif grammars, and no recorded audio: The plan's rationale is varied call instances and song-fragment offers rendered by the same machinery, while preserving the explicit no-recorded-call path.

- Server call scheduling with client WebAudio synthesis and bounded mixing: The plan uses this to create real overlapping calls and replies, preserve audible space for signatures, and avoid synchronized loops.

- Listen-in gain ramps with non-target birds still above zero: The plan's rationale is focus without soloing, clipping, or making the rest of the aviary disappear.

- Audio permission and WebAudio failure behavior: The plan keeps the actual aviary running silently with captions and direct system status, without a welcome/permission modal, fake sound claim, or recorded fallback.

- Captions derived from the actual expanded call program: The plan's reason is synchronized factual captions from note count, contours, trills, intensity, and pauses rather than fixed species labels.

- Caption placement near calling birds with collision-resolved lanes and contrast backing: The plan uses this to keep calls legible without overlapping prose or hiding birds, including reduced motion.

### Accessible interaction and narrative

- Semantic aviary region, top-bar buttons, roving bird focus, and keyboard routes: The plan's rationale is complete keyboard operation and stable focus by bird UUID without exposing trait values, mood codes, or numbered perches.

- Visitor semantic view without interactive bird/listen-in buttons: The plan uses this because visits are read-only and cannot create simulation events.

- Offer keyboard access through top-bar button and optional shortcut: The plan's reason is that shortcut access cannot be the only route or interfere with ordinary typing.

- Focus outlines and dialog focus restoration: The plan uses these so focus remains visible across lighting, weather, captions, faded top bar, snapshots, and dialogs.

- Running naturalist prose from observation facts and reaction descriptors: The plan's rationale is accessible expression of the same visible/audible state without raw traits, enum status, or technical state lists.

- Live-region cadence and queue bounds: The plan uses this to avoid identical paragraphs, stale speech buildup, and assertive interruption while still prioritizing user-initiated events.

- Avoiding phrases such as "Welcome back," "offer succeeded," and "mood changed": The plan's why is lowercase, specific, observational product prose rather than system-state narration.

- Visible narration and pause/resume/repeat controls: The plan treats these as accessibility presentation choices that do not affect simulation.

- Contrast, reflow, and sensory independence: The plan's rationale is that names, actions, notebook prose, calls, greetings, offers, and settling must not rely on color or sound alone.

- Real screen-reader and reduced-motion participant review: The plan says automated tests are not sufficient; users must assess whether the surface feels like birds in a place.

### Notebook, naming, and gradual adoption

- Notebook candidate facts from simulation state/programs: The plan uses this so entries describe supported noteworthy bird observations rather than generic event logs.

- Notebook sparsity gates and novelty scoring: The plan's rationale is "roughly one entry every few days," not writing up every session or becoming a feed.

- Lowercase present-tense bird-specific prose with deterministic private phrasing: The plan uses this to preserve product voice, editorial variety, testability, and privacy without a remote language model.

- Preserving names as observed in old notebook entries: The plan's reason is that renaming does not rewrite the past or break stable bird references.

- Storing final notebook entries with indexed cursor pagination: The plan uses this to keep indefinite history accessible while bounding memory and avoiding edit/share/unread/archive surfaces.

- Bounded Unicode names up to 40 grapheme clusters as the exact limit: NOT RECOVERABLE FROM PLAN

- First owner activation with two distinct system-selected species and no starter catalog: The plan's rationale is meeting and naming birds without a "best starter" choice or species catalog.

- Age opportunities at 90, 180, 270, 365, and 540 days with one pending introduction: The plan uses this for gradual, optional adoption based on actual elapsed age, with deferral that does not expire or reroll the bird.

- The exact adoption ages of 90, 180, 270, 365, and 540 days: NOT RECOVERABLE FROM PLAN

- Fixed offered species and voice seed for each opportunity: The plan's why is identity stability and no rerolling after deferral.

- Seven-bird cap enforced with age in one transaction: The plan uses this to prevent over-cap adoption, simultaneous acceptance races, and behavior-based availability.

### Privacy, account lifecycle, and access control

- Relationship data separated from operational data: The plan's rationale is that simulation interaction data advances only that account's aviary and notebook, with no analytics reader, CDC export, query service, or model training connection.

- Coarse operational metrics and short-retention logs: The plan uses these for diagnosing system failures while avoiding joinable activity timelines, bird payloads, interaction details, or relationship dimensions.

- Transactional mail receives only recipient and requested system message: The plan's why is to keep bird facts, interaction payloads, instrumentation identifiers, click tracking, and provider analytics out of sensitive links.

- Token hashes, atomic consumes, no-store/no-referrer, CSRF, CSP, escaping, and scoped lookups: The plan uses these to protect identity and correctness without adding product ceremony.

- Generic email and visit-link failure outcomes: The plan's rationale is avoiding account enumeration and not revealing whether a visit link was revoked, expired, guessed, or already used.

- Host visit history on demand in account settings: The plan uses this for transparency about recipient, date, approximate duration, and invitations without public listing or treating visit duration as bird presence.

- The exact 180-day initial visit-history retention default: NOT RECOVERABLE FROM PLAN

- Visit notification semantics: The plan's reason is silent default recording, explicit opt-in, open-panel-only notices, coalescing repeated pulls, and no away-channel notifications.

- Export contents and protected delivery: The plan uses export as the explicit vector-portability exception, including current vectors in a consistent owner-protected JSON snapshot without tokens, foreign state, import, reset, or optimization framing.

- Soft deletion with 30-day recovery: The plan's rationale is that recovery finds the same living aviary, preserving UUIDs, vectors, voices, notebook, and age, while disabling interactions and old visitor grants.

- Hard deletion and account-key erasure: The plan uses this so account data cannot be resurrected from live rows, caches, objects, logs, or pre-deletion backups, while unrelated host aviaries are preserved.

### Performance, measurement, verification, and rollout

- Initial JavaScript, critical response, first real bird, greeting, snapshot, frame, audio, memory, simulation, and authorization budgets: The plan's rationale is that aliveness and revocability must hold under measurable load, not be disguised by placeholders, spinners, or short tests.

- The exact byte, millisecond, voice-count, frame, and memory-budget values: NOT RECOVERABLE FROM PLAN

- Bounded resource lifetime for ornaments, audio nodes, queues, notebook pages, snapshots, and receipts: The plan uses this to prevent sustained memory/resource growth during long sessions while keeping history reachable.

- Instrumentation from day one with anonymous allowlisted timings and failures: The plan's rationale is operational health, drift calibration, and endurance monitoring without DAU, retention, streak, species ranking, offer popularity, or relationship analytics.

- Deterministic simulation and persistence tests: The plan uses these to verify nonnegative bounded drift, stable identity, migration continuity, age-only adoption, and perceptual change that remains the same bird.

- Event, multi-device, and failure tests: The plan's reason is preventing duplicate credit, skipped events, double drift, stale overwrites, bogus presence, and invented offline changes.

- Auth, privacy, deletion, and visit tests: The plan uses these to prove link safety, scoped visits, export protection, recovery/deletion boundaries, and telemetry egress limits.

- Visual, audio, accessibility, voice, browser, and endurance tests: The plan's rationale is actual experience evidence: no loading transition, clipping, generic labels, hover-only actions, harsh/repeating voices, caption mismatch, queue floods, or unsupported browser surprises.

- Build sequence increments: The plan uses early contracts and aliveness prototypes to prove first actual bird, calls, narration, and reduced motion before committing to a large app framework; later increments keep accessibility and privacy inside each vertical slice.

- Rollout cohorts and staged adoption-cap gates: The plan's rationale is reliability evidence from synthetic/internal fixtures and staged cohorts, not email, visit frequency, interaction score, manufactured age, or active-tester rewards.

- Deployment and rollback versioning for schemas, engine math, behavior programs, call grammars, and DTOs: The plan uses this to preserve voices, vectors, cursor, and pending events across rollback instead of reseeding or reverting canonical state.

- Operational switches and release stops: The plan's why is that invitations/adoptions/renderers/audio can be paused or forced to safe presentation, but defects in privacy authorization, personality deltas, identity, memory, first-bird performance, or accessibility block release.

- Definition of complete v1: The plan's acceptance rationale is that shipping requires actual owner sign-in, two distinct named birds, current motion, a varied noticing gesture, multi-device durable state, slow nonnegative evolution, and all privacy/performance/accessibility gates; feature count, engagement, or successful build are not substitutes.
