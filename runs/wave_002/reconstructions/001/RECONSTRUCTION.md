## System-level intent

- Living continuity before interaction: The plan repeatedly wants a place that is already alive, not a page that starts only when manipulated. This shows up in "already in motion when opened," "moods and ambient behavior continue on the server while nobody watches," normal ticks that "advance regardless of recent access," and the rule that the server simulation "cannot be paused because no clients are connected."

- Stable, recognizable individuals: The product succeeds only if "the same specific bird remains recognizable after renaming, absence, another device, and a software update." This intent appears in immutable bird IDs, persisted personality vectors, stable signature profiles, migration/restore continuity tests, and the warning that recovery must not "silently replacing a bird."

- One canonical aviary and one history: "All devices observe one canonical aviary," and "there is only one history." The plan carries this through server-owned ordering, the simulation worker as "the sole owner," no "second simulation writer," shared cooldowns, unioned presence, coherent snapshots, and client rules that prevent stale responses from replacing newer revisions.

- Relationship change is slow, measured, and nonpunitive: The relationship "changes slowly through measured attention," with positive evidence filters, daily caps, no single-session visible jump, and nonnegative deltas. The same principle appears in "absence never lowers" traits, "settle has zero directed personality evidence," and the definition of a "continuing, nonpunitive aviary."

- No numbers to optimize: The plan wants change to be felt without exposed scores: a regular observer should sense change "without seeing numbers." It forbids visible mood labels, meters, attendance counts, trait numbers in snapshots/DOM/ARIA/errors/telemetry/plaintext export fields, and ends with "There are no numbers to optimize."

- Calm, observational product voice: The plan's copy and surfaces avoid urgency, praise, shame, and ceremony. It says no welcome toast, absence counter, notification badge, loading spinner, success toast, punitive message, reminders, rewards, achievements, or streaks; account and error copy stay "matter-of-fact," and final product copy remains "specific, lowercase, and observational."

- Privacy and data minimization are product boundaries: The plan excludes latent aggregate interaction datasets, social-ranking statistics, third-party session replay, heatmaps, advertising SDKs, raw event analytics, and behavioral warehouse connectors. Operational metrics must not include bird, mood, trait, presence, invite-recipient, call, or notebook data, and "private state is never used for population-level drift calibration, recommendation, training, or engagement analysis."

- Accessibility is a complete presentation, not a patch: Screen-reader narration, call captions, reduced motion, and keyboard access "ship with the first usable slice." Accessible presentations must carry "the same specificity and calmness," and section 13 calls accessibility "a complete presentation" with semantic controls, authored narration, contrast tokens, keyboard flows, and human evaluation.

- Social access is deliberate, read-only, and quiet: Visits are "deliberately invited read-only visits," with no public discovery, profiles, follows, comments, chat, visitor avatars, shared cursors, host-online cue, or co-presence message. Visitor authorization cannot reach owner event endpoints, and default behavior is silent logging only.

- Authored procedural specificity over canned assets: The plan forbids prerecorded bird calls and stored waveform loops. It favors procedural species grammar, stable signature profiles, continuous parameterization, curated notebook grammar, and validation against the risk that "Alive becomes canned."

- Bounded resources are part of the experience: The plan pairs the calm scene with strict budgets: small snapshots, finite horizons, bounded queues and particle pools, first-bird timing, 60 fps, no memory growth, and tick latency alarms. It treats resource drift and slow first paint as release blockers, not polish items.

- Evidence gates over promises: The plan states that tests are work for the implementation team and "no test execution or product behavior is claimed" by the plan. Release requires property/concurrency/security tests, accessible interaction reviews, listening-review notes, migration/restore checks, and privacy-schema assertions; "planned" is not evidence.

## Per-feature whys

**Product boundary and delivery outcome**

- Browser aviary already in motion: The plan's rationale is to make the first ordinary return feel like a continuing place; birds start "midway through an ongoing action" and there is no normal-return entrance sequence, spinner, or static-to-scene fade.

- Two starter birds: The plan ties two initial birds to the beginning of "a persistent relationship with the owner" and to the success condition that "the same specific bird remains recognizable" over time.

- Seven-bird hard maximum: NOT RECOVERABLE FROM PLAN

- Six coherent species designs: NOT RECOVERABLE FROM PLAN

- Native apps, payments, subscriptions, multiple aviaries, shared ownership, scene customization, dragging birds, and public social surfaces are excluded: The why is to keep v1 inside one browser aviary with one owner relationship, no shared ownership, no public discovery, and no extra social or commerce systems.

- Rankings, rewards, achievements, streaks, counters, hunger, distress, death, decaying happiness, and reminders are excluded: The why is the plan's nonpunitive, no-obligation direction: "There are no numbers to optimize, chores to complete, or reminders to appease."

- No prerecorded bird calls, including error fallback: The why is that call identity comes from procedural species/signature grammar; recorded fallback would violate the no-audio-file and no-stored-waveform call source.

**Explicit interpretation decisions**

- Four top-bar icons with settle inside the offer popover: The plan resolves the icon/settle tension by keeping account/settings, accessibility, notebook, and offer as the persistent icons while making settle "an action reached from the top bar," not scene chrome or a fifth icon.

- Focus and Enter both operate listen-in: The plan's rationale is to satisfy both keyboard descriptions while avoiding focus traps; Escape ends listen-in "without trapping focus or immediately restarting it."

- One 60-second server tick with early user-event wakeups: The why is to let minute-scale simulation coexist with 1-2 second greetings and prompt offer reactions, while elapsed-time accounting prevents extra drift or daily mood resets.

- Account-level IANA timezone: The why is that all clients and visitors share one account clock; a device in another timezone "does not silently overwrite the account clock."

- Settle as local overlay plus canonical mood-quieting impulse: The why is to reconcile session-local settle with shared mood. The overlay is a "viewing-session gesture," while other active owner sessions still count valid presence.

- Mute/audio preference has equal drift: The plan says audible and muted qualified presence count equally so caption users get equal drift and mute does not become "rejection" or a hidden listening requirement.

- Encrypted personality-state export: The why is to preserve a lossless machine record while honoring the stronger "non-exposure" rule; plaintext trait values and client decryption keys are excluded.

- Optional visit-notification email: The why is a narrow exception for an explicitly requested completed-visit notice, off by default, without push, in-aviary announcements, badges, reminders, or onboarding prompts.

- Retaining historical visit rows after revocation: The why is host sharing transparency without leaving an "active-looking visitor" and without erasing disclosure evidence as a side effect.

- Used invitation session lifetime of two hours and 15-minute inactivity lease: NOT RECOVERABLE FROM PLAN

- Exact adoption ages of 90, 180, 270, 365, and 540 days: NOT RECOVERABLE FROM PLAN

- Four-minute pointer/key activity window: The plan says this "intentionally permits still watching" but does not turn a 30-minute motionless tab into credited presence.

**Architecture and ownership**

- Typed web stack and TypeScript contracts: The rationale is to "avoid mismatched event semantics" across browser, API contracts, narration/call descriptors, and the pure simulation reducer.

- Small SVG scene runtime: The plan uses SVG for compact silhouettes, direct transforms, and shared server/client first-frame representation "without a large game engine."

- HTTP application service plus simulation worker: The why is ownership separation; the API cannot apply mood or drift shortcuts, while the worker owns canonical bird mood, vectors, behavior plans, calls, cooldowns, and notebook facts.

- PostgreSQL as authoritative transactional store: The plan wants database constraints and transactions for correctness rather than assuming an API call executes once.

- Database-backed scheduler/job/outbox tables instead of a broker fleet: The rationale is a modular service that can deploy separately and scale by UUID partitions without one process per bird.

- Authenticated edge handler and private HTML snapshot: The why is fast first-frame delivery of private state while ensuring private responses never enter a shared public cache.

- Auth/account module boundary: The why is to keep identity, preferences, deletion, export, and invitation access separate from bird personality; it "never writes an existing bird's personality."

- Command/event ingress boundary: The why is actor validation, idempotency, ownership checks, aviary-local sequencing, and refusal of client absolute trait writes.

- Simulation worker boundary: The why is sole canonical ownership and ordered transaction processing, with no second writer for mood, vectors, behavior, calls, cooldowns, or notebook facts.

- Snapshot projector boundary: The why is safe presentation data: it removes raw trait vectors and private evidence while preserving descriptors needed to draw and synthesize.

- Browser scene/audio runtime boundary: The why is local rendering, synthesis, captions, and presentation preferences without any persistent behavioral or drift decisions.

- Operational metrics collector boundary: The why is to keep rendering/infrastructure health outside the relationship data; it has no simulation database or event-log access.

- Shared semantic observation layer: The plan wants notebook prose, narration, and captions to share facts and vocabulary, "not one generic string emitted at three different cadences."

**Persistent data model**

- UUID keys, UTC instants, canonical IANA timezone, and integer tick/cursor numbers: The why is correctness across transactions, due work, leases, cooldowns, expiry, and diurnal interpretation.

- Account email encrypted once with an auth-only blind lookup index: The rationale is that email is never an identifier, route parameter, queue partition, metric dimension, or log message.

- Pending invite identity as an `invited_pending` account row: The why is to represent unregistered recipients without aviary, owner privileges, onboarding, marketing, or duplicated emails.

- Aviary age begins at creation and is independent of attendance: The why is that adoption is age-based and not an attention threshold.

- Bird IDs, signatures, traits, mood, and drift state stored canonically: The rationale is continuity; numeric traits stay in a protected simulation storage type and identity survives rename, migration, and restore.

- Compact behavior segments: The why is to preserve enough schedule for snapshots without indefinite animation-frame history.

- Call events as semantic descriptors, not waveform files: The why is enough canonical schedule for snapshots and procedural synthesis while avoiding stored recordings.

- Device sessions with recognizable labels and no browser fingerprinting: The why is revocable account access without invasive identification.

- View sessions separate presence, listen lease, settled state, and local mix: The rationale is that local listen mix is presentation-only, while qualified duration accounting is canonical.

- Interaction events append-only with no coordinates or literal typed keys: The why is ordered exactly-once processing with minimized persisted interaction content.

- Presence evidence as bounded intervals and daily credited totals: The rationale is unioned multi-device presence and invisible drift accounting, never a user-facing metric.

- Drift state stored with canonical bird state: The plan says raw logs are not needed to reconstruct a lost vector and "must not be treated as its backup."

- Notebook entries immutable with historical name text: The why is stable history; renaming does not rewrite observation history, and there is no edit/delete/annotation API.

- Observation state as compact bounded memory: The plan wants greeting/weather/pose truth and repeat suppression, "not a diary of user attendance."

- Adoption offers persist proposed species and identity seed: The why is that reload cannot reroll the bird.

- Invitations use capability hashes and transactional one-time redemption: The why is controlled sharing where a consumed token cannot mint another visit.

- Visit sessions and visit log never enter the simulation event stream: The why is that visits are read-only and cannot affect host presence, greetings, drift, or interactions.

- Auth challenges are purpose-bound, hashed, expiring, and consumed once: The why is that an email-change link cannot authenticate or redeem a visit, and transient new-address data disappears.

- Outbox/export jobs resolve email at send time: The rationale is avoiding copied destination addresses in queues while keeping email transactional.

- Name validation and escaping: The why is safe rendering and mail contexts while allowing natural names; name collisions are allowed because accessible controls disambiguate by species/perch.

- Renaming changes only name and display revision: The why is that ID, traits, signature, and previous notebook prose preserve identity.

- Moderate initial trait seed band: The plan says this leaves expressive headroom and preserves bird differences.

- Migration adding a trait preserves existing values and identity: The why is continuity; only the new field initializes deterministically and before/after tests guard recognition.

- Seven-bird cap under aviary row lock: The rationale is correctness under concurrent adoption requests; an outside-transaction count is insufficient.

**Server tick and canonical ordering**

- Due-work scheduler with locks/fencing: The why is that multiple workers may poll, but only the current lock/fencing holder can commit.

- User events waking the same reducer early: The plan wants responsive greetings/offers without a second simulation writer or extra drift.

- Ordered event consumption by server sequence: The rationale is canonical history; events are never reordered by a client's wall clock.

- Notebook candidates only from realized facts: The why is truth; the worker must not write an observation about a future planned call that may be superseded.

- One transaction for birds, environment, schedule, cursor, notebook, revision, due time, and outbox: The why is atomicity; failed commits do not acknowledge consumed evidence.

- API `202` with pollable result after wait limit: The plan balances responsive control acknowledgement with the rule that the renderer must not invent offer acceptance or permanent mood change.

- Server UTC monotonic simulation time: The why is stable leases, cooldowns, adoption, invitation expiry, and no duplicated credit across DST.

- Catch-up after worker outage from persisted state: The rationale is no restart from seed, no inference of a new vector from event history, and no client read as the sole trigger.

- Stored seeds, engine versions, and random-event counters: The why is deterministic retry; duplicate ticks and deliveries are observationally identical to one successful tick.

- Presentation horizon as timestamped descriptors: The plan's rationale is that the server determines behavior and call opportunities, while the client only renders or synthesizes.

- Roughly 120-second presentation horizon: NOT RECOVERABLE FROM PLAN

- Horizon-expired client behavior: The why is to avoid invented canonical calls, offer outcomes, weather transitions, or notebook facts while still showing harmless bounded breathing/quiet field.

**Exact presence and slow drift**

- Presence predicate requiring visible, focused, recent trusted activity, and not ended/settled: The why is an "honest attention approximation," not a tab-open signal, polling signal, or cryptographic gaze proof.

- Trusted pointer movement and physical keyboard activity only: The rationale is to avoid manufactured activity from programmatic focus or synthetic events and to avoid storing pointer paths or typed content.

- Retrospective presence samples and final flushes: The why is that crashes only credit previously confirmed intervals, not a forward lease from an open tab.

- Server clipping, rejecting, and unioning claimed intervals: The plan prevents offline hours, stale overlaps, and double-counted multi-device attention.

- Daily presence/listen/offer caps: The rationale is that presence remains dominant and button traffic cannot become the dominant source.

- Listen duration measured as lease overlap with qualified presence: The why is to avoid accepting client-submitted numbers on faith.

- Settle has zero directed personality evidence: The rationale is that settle is not a trait-shaping reward or penalty.

- Positive evidence filter and exponential update: The plan says it composes across arbitrary tick subdivisions, prevents accelerated interactive ticks from fixed increments, and limits values smoothly near one.

- Fractional precision: The why is that small deltas are not rounded away on the intended slow timescale.

- Versioned configuration for drift constants: The rationale is to adjust future coefficients without rewriting existing vectors.

- Absence behavior: The plan says absence may let transient impulses expire and shape a later greeting, but it is "not a penalty factor or a visible absence counter."

**Mood, gestures, species, and notebook behavior**

- Five initial moods: The why is expressive ambient behavior through semi-Markov dwell times and transition hazards, not visible labels, meters, or a midnight reset to content.

- "Settled" as presentation/pose condition: The rationale is that it is not a sixth negative or rewarded personality state.

- Joint bird-to-bird simulation: The why is to let calls, wary cues, weather, and neighboring birds affect transient expression while response depth and refractory windows prevent alarm/chorus loops.

- Nightjar-like nocturnal behavior: The plan uses it to keep one species with a "low but real chance of calling" while most species settle at night.

- Arrival events for greeting: The rationale is that greeting is requested idempotently and is not presence by itself.

- Absence-derived greeting without visible absence: The why is that longer gaps can change form, but "a long absence is kept private."

- One initial greeter with staggered secondary response: The plan avoids unison greetings and duplicate owner-device arrivals while preserving characteristic tendencies.

- Continuous greeting parameters and recent fingerprint suppression: The rationale is to avoid identical cues and "three canned clips" while preserving per-bird tendencies.

- Offer popover with seed, song fragment, and still pool: NOT RECOVERABLE FROM PLAN

- Server-chosen offer recipients: The why is that the user does not directly click a bird to feed it; position/mood choose recipient(s), and several eligible birds can respond naturally.

- Three-minute per-bird offer cooldown across types and devices: The rationale is to bound secondary credit and prevent repeated requests from restarting objects or affecting traits.

- Drowsy nonresponse to offers: The plan treats no reaction as a valid natural response, "not an error."

- Quiet inline cooldown explanation: The why is calm product voice: no countdown, success toast, or punitive message.

- Settle/undo interaction: The rationale is a reversible session gesture that closes presence immediately, applies a small transient mood-quieting impulse, and does not rewind unrelated canonical state.

- First-ever initialization fly-in: The why is a unique first-adoption exception; regular returns must show ongoing motion and never recreate an empty/adoption entrance.

- Distinct silhouettes, plumage palettes, posture tendencies, motif families, and signature profiles: The rationale is recognizable individuals, including duplicate species at the seventh bird.

- No catalog, rarity, stat comparison, or avatar configuration during adoption: The why is to avoid optimizing, shopping, or comparing birds by status/stat surfaces.

- Age-based adoption on demand: The rationale is growth without visit counts, progress gauges, emails, modal interruptions, attention-earned rewards, or missed-visit penalties.

- Stable adoption proposal: The plan says proposed species and identity seed remain stable so defer/reload cannot reroll identity.

- Curated notebook grammar from realized server facts: The why is private, deterministic, factual, tonal prose without a runtime language-model service or event-to-text dump.

- Notebook truth checks for comparisons: The rationale is that "a first this week" requires retained evidence; unknown comparisons become noncomparative observations.

- Sparse notebook budget and repeat suppression: The why is to avoid filler, absence backfill, and very active users producing dense attendance diaries.

- Descending stable notebook pagination with "load earlier observations": The plan keeps historical entries queryable and accessible without archive, edit, per-entry delete, annotation, streak, or attendance export surfaces.

**API contracts and client synchronization**

- HTTPS, versioned JSON, secure same-site cookies, CSRF, strict schemas, no-referrer token routes: The why is authenticated state changes and token protection without leaking writable state fields.

- GET routes do not consume magic or invitation links: The plan explicitly protects against mail scanners spending one-time capabilities.

- Authenticated `/aviary` with private snapshot and first-frame SVG: The rationale is immediate current rendering; HTTP prefetch alone must not infer a greeting event.

- Snapshot endpoint excluding numeric personality vectors: The why is coherent canonical presentation without trait exposure.

- Owner view endpoint with idempotent visible-transition ID: The rationale is one arrival request for a visible transition.

- Event endpoint batch receipts: The why is bounded, ordered, acknowledged owner interactions tied to committed revisions.

- Notebook endpoint without mutation variants: The rationale is immutable read-only observations.

- Bird rename endpoint with expected display revision: The why is conflict detection without touching identity or traits.

- Adoption endpoints with one current proposal, accept, and defer: The plan prevents attendance counts, rarity, future countdowns, progress gauges, and penalties.

- Interaction envelope and server-supplied actor/effective time: The rationale is that client timestamps are diagnostic hints, not sources of mood, traits, hours, or grammar.

- Deduplication by view session and client event ID with body fingerprint: The why is identical retry returning the original result and different-body reuse becoming an error.

- Payload and rate bounds: The rationale is ordinary navigation support without unbounded queues; cooldown rejection is semantic result, not another evidence dose.

- Stable matter-of-fact API errors: The why is concise system copy and no disclosure of guessed invitations or unknown bird ownership.

- Coherent snapshot serialization type: The plan wants to make spreading a database bird record into JSON impossible and to avoid trait vectors under alternate names.

- Snapshot size and descriptor horizon bounds: The why is compact seven-bird state and cached public species/motif assets instead of repeated payloads.

- Visible polling every 15 seconds with jitter and immediate refresh triggers: The rationale is current rendering after visibility changes, gaps, interactions, and reconnection.

- Highest-applied revision and smooth server-clock correction: The why is that older slower responses cannot replace newer state and transitions stay continuous.

- Hide/return behavior: The plan stops animation/audio/polling, closes presence, refreshes on return, and avoids playing a backlog.

- Offline behavior: The rationale is last safe rendition plus disabled canonical gestures, no local competing simulation, no stored vectors, and no retrospective offline credit.

- Simultaneous devices: The why is that one canonical sequence resolves cooldowns, listen intervals are unioned, and "there is only one history."

**Account lifecycle, security, export, and deletion**

- Email magic-link account access: The plan's rationale is generic success to prevent account enumeration, random single-use tokens, scanner-safe landing, and no password/local-storage auth.

- Magic-link token expiry of 15 minutes: NOT RECOVERABLE FROM PLAN

- Account preferences for timezone, audio, accessibility, and optional notices: The why is explicit account-level settings with revisioned changes.

- Device session list/revocation: The rationale is user-recognizable sessions and immediate invalidation of view leases.

- Device session absolute lifetime of 30 days: NOT RECOVERABLE FROM PLAN

- Recent magic-link requirement for deletion, export, or email change: The why is extra protection for sensitive account actions.

- Verified email change: The plan verifies the new address before replacing encrypted account email, avoids merges, rotates credentials, and revokes other sessions as a deliberate security consequence.

- Consistent export at revision R: The rationale is names, moods, sealed vectors, and notebook from one committed snapshot without making export an attendance log.

- Sealed archival personality block: The why is exact preservation of vector/schema/drift state without disclosing numerical values or adding import/trait-inspection UI.

- Export expiry and download authorization checks: The plan rechecks session and deletion status at download and expires encrypted objects after 24 hours.

- Pending deletion state: The why is explicit lifecycle control; deletion stops scheduling, terminates leases, revokes invitations, cancels jobs, and is "not ordinary user absence."

- Recovery before 30-day deadline: The rationale is preserving exact bird IDs/vectors, mood records, names, notebook, and age without fabricated presence.

- Hard deletion and key destruction: The plan requires removing all account-linked records and destroying unique keys so backups cannot restore private contents.

- Processed raw interaction event retention for at most seven days: The why is retry diagnostics and user simulation integrity without treating raw logs as permanent backup.

- Visit history retention for 90 days: The plan says this supports host sharing transparency without an indefinite contact trail.

**Read-only visits**

- Owner-created named one-time invitations: The rationale is deliberate sharing with no global discoverability flag.

- Invitation state transitions and atomic redemption: The why is to prevent simultaneous browsers redeeming one invitation twice and to ensure a consumed token is never restored to issued.

- Render-only visitor session credentials: The rationale is that even a visitor with their own account receives no owner mutation authority.

- Visitor sees same canonical scene: The why is to observe the real current renderer, moods, positions, calls, day/night clock, and weather, not a flattering visitor-specific scene.

- Visitor restrictions on greeting, presence, offers, settle, notebook, naming, and adoption: The rationale is that visits never affect host state or attention evidence.

- Visitor-local audio, captions, narration, and reduced motion: The why is that these change only visitor presentation and cannot change host settings.

- Visitor top bar with only relevant controls: The plan avoids unusable owner controls and avoids co-presence or social cues.

- Revocation and freshness checks on every visitor response: The rationale is fail-closed access; revoked, expired, hidden, or stale sessions clear host scene and cached private data.

- Visit duration from server read timestamps: The why is approximate host transparency without using the owner's presence algorithm or hidden timers.

- Optional completed-visit notice email: The plan confines this to one plain system email with recipient/date only, no bird details, return prompt, streak, marketing text, UI notice, or badge.

**Visual rendering and responsive composition**

- Authenticated first-frame SVG already evaluated at current time: The rationale is continuity through first paint and hydration; transforms do not reset and birds do not scale from zero or wait behind a spinner.

- Quiet sky/field on missing authenticated state: The why is to avoid invented starter birds or another account's cached scene while still painting a calm non-bird ambient cue and matter-of-fact retry chrome.

- Three perch zones: NOT RECOVERABLE FROM PLAN

- Server behavioral perch selection plus client layout projection: The plan preserves behavioral meaning; collision resolution cannot move a wary bird to the front for layout convenience.

- Responsive geometry retaining every full body and interaction target: The why is that phones, wide screens, zoom, captions, and seven birds must not crop, pan, scroll, or end flights offscreen.

- No scene labels, tooltips, icons, names, badges, or drag handlers: The rationale is a quiet visual field, with captions and visible keyboard focus as explicit accessibility exceptions.

- Continuous eased motion instead of fixed clips: The why is to avoid restarting idle phase once a minute and to keep motion alive rather than canned.

- Account-local synthetic lighting: The rationale is local morning/midday/evening/night without geolocation or a weather service.

- Rare gentle rain frequency and duration: NOT RECOVERABLE FROM PLAN

- Gentle weather with no thunder, storm urgency, distress, or alert: The why is calm ambient expression, not danger or obligation.

- Leaves and feathers as bounded client-only ornaments: The rationale is visual liveliness without events, personality evidence, blur-heavy animation, or unbounded particles.

- Top-bar fade after pointer stillness: The why is calm chrome while preserving focus visibility, tab order, touch usability, and contrast.

- Reduced motion before first frame: The plan's rationale is that motion-sensitive users must not briefly see normal motion before settings load; all semantics, calls, captions, moods, offers, drift, and notebook observations remain.

**Procedural audio, listen-in, and call captions**

- Species motif definitions and per-bird stable signature profiles: The rationale is recognizable call identity across naming and drift, with personality/mood altering bounded expressiveness rather than identity.

- Server semantic call descriptors and client WebAudio synthesis: The why is procedural authorized calls with bounded variation and no downloaded or stored recorded bird calls.

- One AudioContext, per-bird buses, scratch buffers, optional worklet only after profiling: The plan's rationale is bounded resources, legal oscillator-node lifecycle, and minimal first scene.

- Audio scheduler tied to server timeline: The why is synchronized call descriptors, subtle stereo, mono verification, headroom, and no cutting off a sounding bird to satisfy caps.

- Listen-in gain ramp: The rationale is to focus one bird while keeping others audible, avoiding clicks during focus transfer, and separating local mix from canonical calls/mood/drift.

- Listen-in end and timeout rules: The why is no indefinite drift from a missing end event and no cross-device synchronization of local mix.

- Browser-autoplay handling: The plan says universal immediate sound is impossible, so it keeps motion and captions alive until a normal gesture resumes audio.

- WebAudio failure fallback to silence and captions: The rationale is truthful degradation without recorded audio fallback, forced modal, or loss of the scene.

- Captions generated from the actual expanded descriptor: The why is accuracy; no fixed caption per bird/motif/mood, no caption for canceled calls, and muted/unavailable sound still represents the scheduled call.

- Caption placement and live-region behavior: The plan balances readability and attribution without queues detached from current calls or forcing screen readers to hear every call over narration.

**Accessibility as a complete presentation**

- Semantic DOM controls and accessible scene region: The rationale is that assistive technology should not interpret thousands of SVG feather paths; it receives intelligible bird controls and scene description.

- Bird control names with species/perch but no trait or mood strings: The why is specific access without exposing hidden numbers or mood labels.

- Keyboard order with roving tabindex: The rationale is stable spatial bird navigation through pose changes, with offer/settle/dialogs reachable by normal keys and no hover-only interactions or focus traps.

- Alt+O offer shortcut: NOT RECOVERABLE FROM PLAN

- Authored narration in one polite live region: The why is a coherent quiet moment from semantic state, avoiding raw state lists, event codes, session language, near-duplicates, or hidden-tab backlogs.

- Priority bumps for user-initiated observations: The rationale is that screen-reader users can hear a greeting/offer/settle response promptly without interrupting dialogs or fighting a queue.

- Pause/resume/read-current-scene controls and optional transcript: The plan preserves access to current description while pause affects speech delivery only.

- Contrast tokens, forced colors, 200% text scaling, touch targets, and browser zoom: The why is that night, lighting, plumage, captions, labels, and focus indicators remain usable.

- Accessibility QA across assistive technologies and conditions: The plan says automated checks cannot prove the intended experience, so human evaluation of narration and reduced-motion charm is required.

**Performance budgets and operational instrumentation**

- Initial JavaScript budget and lazy loading: The rationale is first-bird paint; initial reduced-motion preference and semantic scene cannot be lazy-loaded because they affect the first presentation.

- First bird visible in less than 500 ms: The plan measures the actually painted, viewport, nontransparent bird, not React mount, skeleton, request finish, or a 1-pixel placeholder.

- Responsive greeting metric: The why is that a fast static-looking scene cannot pass; first noticing must be measured separately from first-bird paint.

- 60 fps idle budget: The rationale is sustained seven-bird chorus/weather performance on a five-year-old mid-range laptop, using frame percentiles and missed-frame proportion rather than average fps.

- No memory growth over 30 minutes: The plan treats repeatable retained heap, DOM, listener, node, buffer, or context growth as a release blocker.

- Small state and bounded compute: The why is worst-case seven-bird/caption/response stability through bounded snapshots, event batches, horizons, queues, pools, and DOM windows.

- Tick operational alarms: The rationale is seeing both slow compute/commit and stuck queues; healthy compute is separate from due-to-start lag.

- Reproducible cold mobile performance profile: The plan wants honest physical-device, network, geography, cache, and percentile evidence rather than blended warm-cache results.

- First-frame delivery critical-path controls: The why is that extra auth/database/runtime round trips can spend the budget; private dynamic HTML must be fast without publicly cached stale vectors.

- Operational health metrics only: The rationale is infrastructure diagnosis without turning route/session/performance data into behavioral analytics.

- Bounded support/error logs with deletion: The why is diagnosing auth/loading failures without serializing bird state, interaction content, tokens, emails, vectors, or offer payloads.

- Schema tests for forbidden telemetry fields: The plan instruments privacy boundaries from the first vertical slice rather than relying on policy after collection.

- Synthetic vector charts only for generated test birds: The rationale is engineering calibration without creating a real-user debug mode.

- Capacity planning for all nondeleted aviaries: The why is that scheduled load is not contingent on current viewers; every aviary continues ticking.

- Point-in-time backups and checkpoints of vectors with event cursor: The rationale is avoiding replayed or lost evidence and preserving IDs/signatures during recovery.

- State-integrity outage surface: The plan says if integrity cannot be established, show a system outage and restore correctly rather than silently replacing a bird.

- Operational alarms excluding engagement/personality: The why is health monitoring without alarms based on clicks, personality, or aggregate drift.

**Validation strategy and acceptance matrix**

- Property, calibration, absence, mood, migration, and bird-to-bird engine tests: The why is to prove nonnegative drift, timescale, continuity, no fabricated attendance, no reset, and non-canned greetings.

- Presence and multi-device tests: The rationale is to verify the exact attention predicate, four-minute boundary, unioned intervals, deduplication, ordering, and rejection of forbidden client/visitor writes.

- Experience, audio, and notebook tests: The why is to check ongoing return motion, offers/settle semantics, procedural recognizability, caption accuracy, sparse factual notebook prose, and no attendance/achievement language.

- Access, privacy, and lifecycle tests: The rationale is to prove accessibility flows, reduced motion, invitation revocation, account security, sealed export, deletion/recovery, and telemetry privacy.

- Performance and release evidence package: The plan requires synthetic workload definitions, device/browser/network profiles, distributions, traces, reviews, migrations, restores, and assertions because "planned" is not evidence.

**Implementation sequence and rollout gates**

- Stage 0 contracts and fixture foundation: The why is to lock immutable IDs, events, revisions, presence, vector storage, descriptors, timezone, role separation, and interpretation decisions before product code creates forbidden paths.

- Stage 1 two-bird living slice: The rationale is to prove first-frame/greeting budgets, disconnected server ticking, two persisted birds, exact presence, keyboard/screen-reader flow, two-device coherence, and memory bounds before scope expands.

- Stage 2 relationship and gestures: The why is to add drift, moods, responses, offers, listen-in, settle/undo, rename, environment, and notebook only once calibration, concurrency, recognizability, and no-number/no-obligation checks pass.

- Stage 3 full account/privacy lifecycle: The rationale is to complete revocation, email change, export/sealing, deletion/recovery, retention, key erasure, restore, and error surfaces with observability separated from simulation data.

- Stage 4 quiet visits and full species pool: The why is to prove visitors cannot influence host state, revocation works across visibility/connectivity states, and silence-by-default/no co-presence hold.

- Stage 5 age-based growth and sustained quality: The rationale is to test two/three/five/seven birds, mobile collision/caption layout, seven-bird chorus, cap, signature differentiation, 60 fps, memory, accessibility, and browser matrix before growth.

- Stage 6 production ramp: The why is to launch complete v1 to a limited cohort while preserving health, qualitative review, rollback procedures, and no deferred required accessibility/lifecycle feature.

- Internal synthetic aviaries and virtual 18-month accounts: The rationale is validating adoption stages without accelerating real owner ages and without aggregating private events into population tuning datasets.

- Small dogfood group for listening/visual continuity: The plan limits this to qualitative feedback so authored motion/call parameters can improve without collecting account histories.

- Operational growth ceiling from two to three to seven: The why is quality gating; raising the ceiling permits already age-eligible adoption but never creates an attention reward, forced adoption, or progress display.

- Conservative account admission ramp: The rationale is tick capacity, first-bird latency, audio errors, synthetic performance/accessibility regressions, and qualitative continuity, not visit frequency, offer usage, average traits, engagement, or social invites.

- Backward-compatible client rollback and versioned engine constants: The why is preserving vectors and legitimate drift; severe calibration issues can pause future drift rates without overwriting state.

- Audio mitigation without recorded fallback: The rationale is to keep procedural motifs, narration, and captions available while never falling back to recorded loops.

- Integrity incident handling: The plan says failures must resolve with "system clarity" rather than reseeding birds.

**Risk register and v1 completion**

- Drift risk response: The rationale is to adjust future coefficients/filter caps while preserving accumulated vectors and avoiding aggregate production relationship tuning.

- Presence risk response: The why is to close intervals precisely and clamp retrospective evidence rather than substitute polling or click counts.

- Duplicate-processing risk response: The rationale is atomic cursor/vector commits, serialized sequencing, and coherent checkpoints.

- Migration/data-loss risk response: The why is to block release or restore existing records, with no regeneration fallback.

- Canned-aliveness risk response: The plan expands continuous parameterization and contextual responses while preserving per-bird signatures.

- Audio-recognizability risk response: The rationale is tuning motifs, headroom, density, and identity bands, and holding adoption ramp if recognizability fails.

- Autoplay-block risk response: The why is to retain motion/captions and resume only on allowed gesture, with clear settings controls.

- First-paint risk response: The plan reduces critical bytes/round-trips, inlines current scene, isolates private cache, and rejects warm-only evidence.

- Memory-growth risk response: The rationale is bounded lifetimes and explicit release of nodes/handlers, with rollout blocked on repeatable trends.

- Accessibility risk response: The plan treats prose cadence, pose transitions, focus behavior, and copy backing as first-class design work.

- Invitation leak/revocation risk response: The why is every-pull revalidation, role separation, freshness expiry, and token scrubbing.

- Analytics risk response: The plan prohibits bird/event fields at ingestion and denies warehouse access to simulation storage.

- Export expectation risk response: The rationale is documenting the sealed-block limitation without exposing numerical traits or promising independently readable state.

- Notification-loop risk response: The plan confines notices to explicitly requested visit emails and keeps other social/chrome reminders absent.

- Background-simulation cost risk response: The why is to reduce scheduling overhead, scale UUID partitions, and limit new accounts without making ticks contingent on viewers.

- Definition of v1 completion: The rationale is the combined proof that two stable named birds, already-moving return, 1-2 second noticing, valid monotonic presence-driven drift, nonpunitive absence, multi-device canonical state, complete owner features, read-only visitors, accessibility, performance, privacy, restoration, and observational copy all hold at launch.
