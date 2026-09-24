## System-level intent

- **Ongoing aviary, not an engagement game.** The plan repeatedly frames the product as "watchful presence" and an "ongoing aviary," with "no textual return greeting," "no spinner or entry sequence," and explicit exclusions for "scores, streaks," "achievements," "levels," "leaderboards," "care meters," "hunger, distress or death," and "push engagement." This intent shows up in Product contract and scope, Scene and audio pipelines, Performance, privacy, and operations, and the rollout guidance to monitor "technical health and opt-in qualitative reports, not engagement leaderboards or population interaction patterns."

- **Server-authored, durable, canonical life.** The plan wants the aviary to continue "with no client connected." It says the "server is the sole writer" of personality, mood, perch/behavior schedule, weather, notebook, and adoption availability; clients submit only "bounded intentions and attention evidence." The same intent appears in the split between "Canonical simulation state," "Ephemeral session state," and "Render ornaments," plus the durable tick, checkpoint, catch-up, and "no client personality writes or last-write-wins" language.

- **Stable bird identity and additive personality.** Identity is treated as something that must survive projection, export, rename, visit views, and migration. The plan says to "preserve bird IDs and vectors," "never replace an existing ID," reject migrations that regenerate or reset a vector, and keep traits from declining: "after absence the signal fades, but the trait never declines." The risk table repeats this as "immutable bird IDs" and "Never reduce traits on neglect."

- **Honest presence without punishment.** Presence must be qualified by all three signals: visible, focused, and recent pointer or key activity. The plan says a fresh event "cannot retroactively credit unobserved time," "mere visible tab time does not" count, and simultaneous devices are unioned to "prevent double drift." At the same time, tab-close and settle receive "the same absence treatment" and "No engine penalty exists for either."

- **Privacy by separation and omission.** The plan separates encrypted email from synthetic account UUIDs, restricts analytics from bird, event, notebook, and per-account state, omits raw vectors from snapshot projections, forbids logging emails or invitation secrets, and uses aggregate-only operational metrics. It also says no visitor presence rows, no visitor event route, and "No host interaction signal is sourced from a visitor."

- **Naturalist voice for the aviary, matter-of-fact copy for systems.** Product voice is "lowercase, specific naturalist observation," while auth, account, errors, revocation, and settings use "direct matter-of-fact copy." Notebook entries are "present-tense naturalist prose," screen-reader narration is "slow naturalist prose," and errors use "matter-of-fact copy in the UI."

- **Accessibility is a designed mode, not a dump of internals.** The plan calls for a reduced-motion "designed second renderer" with the same canonical state and actions, keyboard-complete controls, captions from the actual grammar event parameters, and screen-reader narration from the same snapshot and grammar events. It warns that accessibility must not become "a state dump" and says not to narrate raw status codes, perch numbers, or trait values.

- **A continuous first scene across devices.** The scene must fit "without panning, zooming, or cropping a bird," show a bird within 500 ms, initialize "mid-preen, scan, tilt, shuffle, or call," and avoid a spinner, fade, or entrance sequence. Edge snapshots, compact critical assets, responsive layout, and soft reconciliation all serve the first usable frame being the ongoing aviary.

- **Procedural, recognizable, non-canned calls.** Audio intent is a versioned call grammar with per-species motifs and a stable per-bird signature seed. The plan rejects "recorded call loops," "recorded fallback," and "layered recordings," while requiring variation in microtiming, pitch, timbre, and gaps so calls are recognizable without sounding canned.

- **Social access stays narrow and read-only.** Visits are named invitations with scoped viewer sessions, expiry, revocation, no event route, no visitor presence collector, no visitor drift, and no public profiles, discovery, comments, co-presence, or shared aviaries. The host sees visit information only "on demand in settings," with "no badge or default notification."

- **Versioned calibration with explicit gates.** The plan keeps engine rules, snapshots, call grammar, notebook grammar, scene layout, and tunables versioned. It uses deterministic fixtures, qualitative review, acceptance gates, server flags, cohort rollout, and invariant tests so calibration can change "without resetting a bird or altering old notebook entries."

## Per-feature whys

### Product contract and scope

- **Browser-only, single-screen aviary:** The plan ties this to an aviary that "fits any supported viewport" without panning, zooming, or cropping a bird, and to the first usable frame showing the "ongoing aviary."

- **One canonical aviary per signed-in person:** NOT RECOVERABLE FROM PLAN

- **Two starter birds assigned by the system, with naming as the only initial choice:** NOT RECOVERABLE FROM PLAN

- **Hard maximum of seven birds:** NOT RECOVERABLE FROM PLAN

- **Age-paced arrivals from a roughly six-species pool:** The plan says offers derive only from aviary creation time, "never session count or interaction volume," and should be tuned "without promising more birds as a reward."

- **Returning host noticed by one bird within one to two seconds:** The plan's rationale is behavioral noticing rather than copy: the greeting is conditioned on "elapsed absence, boldness, and mood," quick returns favor "a glance," days away favor "a slightly longer orientation," secondary responses are staggered, and greetings must not cycle "a tiny fixed list."

- **No textual return greeting:** The plan places this under the same product voice as naturalist observation and keeps the greeting in bird behavior rather than system copy.

- **Sparse read-only field notebook:** The notebook is for "salient canonical observations," not every session or raw event. Its rarity budget, distinctiveness threshold, dedupe window, and ban on "user attendance," "vector values," and "behavioral score" keep it observational rather than a stats surface.

- **Local-time lighting and rare ambient weather:** Local time drives lighting and night behavior with DST-aware conversion; rare rain and wind provide seeded environmental variation that can quiet calls or nudge mood probabilities without changing bird identity or history.

- **Magic-link accounts:** The plan supports this through 15-minute single-use links, per-email/IP abuse limits, secret redaction, and revocable per-device sessions, which makes sign-in direct while preserving abuse and session controls.

- **Multiple signed-in devices:** The plan uses per-device revocable sessions and merges overlapping host intervals across devices "before drift" to support multiple devices without double-crediting presence.

- **Account export:** The plan frames export as "data portability," explicitly including raw vectors behind account settings while not rendering them as an in-product stats surface. Export artifacts are short-lived and scoped to the verified account.

- **Account deletion:** Deletion immediately marks the account, suspends sessions and invitations, offers 30-day recovery, and then hard-deletes dependent records and artifacts, including account-linked telemetry where any exists.

- **Named read-only visit invitations:** The plan allows invited viewing while preventing social discovery or visitor influence: visitors get a narrow snapshot route, no event route, no presence rows, no interaction signal, and revocation on the next pull.

### Delivery shape and ownership

- **Small web client, authenticated API, durable simulation worker, and transactional store:** The plan assigns rendering and synthesized calls to the browser, host authorization and mutation control to the API, durable ticking to the worker, and canonical records to the store so the aviary can keep running without client ownership.

- **Versioned deterministic engine package:** The engine is shared by worker and test fixtures so ticks, catch-up, migration behavior, and calibration can be reproduced and gated with invariant tests.

- **Relational database for canonical records and object store only for short-lived export artifacts:** The plan uses the relational database for accounts, aviaries, birds, event log, invitations, visits, notebook, and tick checkpoints because those are transactional canonical records; the object store is limited to temporary account-export artifacts.

- **Server as sole writer of simulation state:** This prevents vector, mood, perch, weather, notebook, or adoption state from being written by clients and avoids "client last-write-wins for personality."

- **Clients submit bounded intentions and attention evidence:** This lets the browser express listen-in, offers, settle/undo, and qualified presence while keeping authority for drift, mood, and identity on the server.

- **Edge-served signed fresh snapshot:** The plan uses this so "the first bird need not await the full JS bundle," while freshness validation and canonical reconciliation prevent stale or unauthorized state from becoming authoritative.

- **No private snapshot in a shared public cache:** The plan states this as an authorization and privacy boundary for returning sessions.

- **Canonical simulation state:** Persisted, versioned server ticks keep the simulation continuing with no client connected.

- **Ephemeral session state:** Local focus, audio permission, top-bar opacity, panels, and settle undo are session-only because settle is a "session gesture" and "does not alter bird identity or permanent traits."

- **Render ornaments:** Leaf and feather drift and subsecond pose interpolation are generated locally from a snapshot seed because they "never enter the event log or cause drift."

- **Schema versions for snapshots and engine rules:** Versioning allows migrations and calibration while preserving bird IDs and vectors and rejecting silent regeneration or vector reset.

- **Synthetic account UUID and encrypted email:** The plan encrypts email only on the account record and uses synthetic UUIDs for joins, logs, service messages, and metric dimensions to avoid email-derived keys.

- **Analytics access restriction:** Analytics jobs have no access to bird, event, notebook, or per-account state tables to keep private interaction and observation data out of analytics.

### Persistent data model

- **Account record with encrypted verified email, timezone, settings, deletion times, one aviary, and revocable sessions:** The fields support verified access, local-time behavior, accessibility and notification preferences, deletion/recovery, the one-aviary contract, and per-device session revocation.

- **Aviary record with monotone state version, tick checkpoint, engine seed, lighting/weather, next adoption, and layout version:** These fields support canonical projection, deterministic simulation, catch-up, versioned layout, and age-based adoption availability.

- **Bird record with stable UUID, species, name, traits, mood, perch/behavior, call seed, and offer timestamps:** The record supports stable identity, rename without replacement, bounded personality, mood behavior, call recognizability, and cooldown-controlled offers.

- **Constraining two to seven active birds:** NOT RECOVERABLE FROM PLAN

- **Never replacing bird IDs on rename or rule migration:** The stable-ID requirement is justified by migration, rename, export, and visit consistency.

- **Interaction event log with idempotency key and per-account sequence:** The event log enables deduplication, retry safety, bounded payload validation, and processing against a known tick/version.

- **Retention bounded to simulation and privacy duties:** The plan gives the why directly: retain only what simulation and privacy duties require.

- **Presence intervals as event-derived intervals:** The plan stores qualified evidence as intervals so overlapping host devices can be merged before drift and abandoned sessions can expire.

- **No visitor presence rows:** This enforces the product rule that visitor activity never sources host interaction signals or drift.

- **Field notebook entry stored with rendered naturalist text:** Rendering and storing the text prevents later grammar or rule changes from rewriting history.

- **Append-only notebook:** The append-only form preserves history and matches the plan's requirement that later rule changes do not rewrite stored observations.

- **Oldest-to-newest notebook pagination without archival cutoff:** NOT RECOVERABLE FROM PLAN

- **No notebook entry reports user attendance:** This keeps notebook prose observational rather than turning it into a visit-frequency surface.

- **Invitation and visit records with restricted email and secret handling:** Encryption, hashed redemption secret, scoped session, expiry, revocation, and telemetry separation support named visits without leaking invitee data or creating public/social discovery.

- **Private audit of vector deltas and engine versions:** The plan says this is for "correctness and recovery" and must be isolated from analytics.

- **Snapshot projections omit raw vectors and internal presence details:** The rationale is to expose renderable state while keeping bird trait numbers and presence internals out of product UI.

- **Account export includes vectors, moods, birds, notebook, and settings through a short-lived emailed link:** This preserves data portability while keeping raw vectors behind account settings and away from ordinary stats surfaces.

- **Deletion with immediate marking, session/invitation suspension, 30-day recovery, then hard delete:** The plan balances account recovery with eventual removal of dependent records, artifacts, and account-linked telemetry.

### API and authorization contract

- **Authenticated mutating routes with CSRF, rate limits, validation, and idempotency:** The plan uses these controls to make host mutations safe, bounded, retryable, and protected under cookie auth.

- **Visitor token limited to a narrow snapshot route:** This lets visitors view the same scene projection without any route that can affect events, drift, or canonical state.

- **Explicit schema versions and error codes with matter-of-fact UI copy:** Versioning and explicit errors support compatible clients and direct system copy for failures.

- **Magic-link issue and consume routes:** The 15-minute, single-use link, abuse limits, atomic consume, and secret redaction support passwordless auth without reusable or logged secrets.

- **Snapshot route with sinceVersion, ETag, server time, next recommended pull, and current timeline:** The route reduces unnecessary fetches, avoids stale authorization caches, and lets the client enter "mid-action."

- **Events route for presence, listen-in, offer, settle, and undo:** The route accepts bounded client evidence and gestures, returns accepted IDs and rejected reasons, and refuses vector, mood, or perch writes so the server remains authoritative.

- **Notebook route with stable cursor pagination:** The plan exposes notebook entries as read-only history with stable pagination.

- **Bird rename route:** Rename is allowed with ownership and safety checks but "does not touch identity or engine state," preserving stable bird identity.

- **Adoption route:** The route accepts only a server-offered age-eligible bird, transactionally enforces cap and no duplicate acceptance, and persists server-assigned species/seed/identity before response.

- **Account settings, session revocation, email change, export, deletion, and recovery routes:** Fresh-auth checks, new-address verification, short-lived export links, and recovery/deletion flows protect account-sensitive actions.

- **Visit invitation, revocation, listing, log, redemption, and snapshot routes:** These routes create a host-controlled invitation surface, bind a narrow viewer session, check expiry and revocation on every pull, and keep visitors read-only.

- **One-time visit link interpreted as one-time redemption into a scoped visitor session:** This allows a visit to last beyond a single snapshot while still being bound, expiring, and revocable.

- **Unredeemed invite expiry after 30 days and revocation invalidating active sessions:** The plan's why is prompt authorization control; short visible-page pull intervals and resume checks make "visit no longer available" appear quickly.

- **Host visit log only on demand, with no badge or default notification:** The plan keeps visits from becoming default engagement or social pressure; opt-in visit notification appears only after deliberate enablement and with no onboarding prompt.

### Presence, event processing, and simulation

- **Presence qualified by visibility, focus, and recent pointer or key activity:** The rationale is "Honest presence": only time with all three signals is credited.

- **Three-minute activity window measured against people who sit quietly:** The plan explicitly avoids tightening presence to seconds because quiet observers may sit without constant input.

- **Immediate interval end on condition failure, settle, navigation, or page hide:** The plan prevents credit for time that is no longer qualified.

- **Heartbeats only while qualified, capped by lease plus grace:** Heartbeats bound credited time, expire abandoned sessions, and prevent unobserved or offline presence from creating drift.

- **No retroactive credit from a fresh pointer or key event:** The plan states that a new event cannot credit unobserved time.

- **No raw coordinates or keystrokes emitted:** Pointer movement is only a coarse interaction signal; the privacy rationale is to avoid sending raw input data.

- **Touch may count on mobile, but visible tab time does not:** The plan distinguishes active evidence from passive visibility.

- **Visitors never run the presence collector:** This enforces the rule that no visitor interaction signal affects the host aviary.

- **Server receipt time, clamping, deduplication, and union across host devices:** These keep the server authoritative, handle delayed client times, deduplicate retries, and prevent double drift from simultaneous devices.

- **Listen-in duration from validated start/end pairs with timeout caps:** This bounds the signal used for warmth and vocal-frequency changes.

- **New focused bird closes the previous listen-in interval:** The plan prevents overlapping focused-bird listen-in credit.

- **Settle terminates credited presence and supports five-second undo:** Settle is treated like ending presence, while the undo/re-engage event can reverse the client lighting transition within five seconds.

- **No engine penalty for settle or tab-close:** The plan explicitly gives the why as equal absence treatment with no penalty.

- **Approximately one-minute durable tick for every aviary, including inactive ones:** This keeps the simulation continuing independently of connected clients.

- **Partition leases, serialized database transactions, checkpoints, and idempotent crash retry:** These protect deterministic tick application and recovery.

- **Deterministic catch-up after worker downtime:** Catch-up handles elapsed ticks "without creating presence or forcing sudden client-visible jumps" and alarms on lag.

- **Normalized traits with species-specific distributions, nonnegative targets, low-pass step, and caps:** The plan uses this to create slow additive drift, keep traits bounded, and prevent visible jumps from individual sessions.

- **Qualified presence dominates trait input; listen-in and offers add smaller focused signals:** The plan calibrates personality change around regular attention while keeping offers and listen-in as smaller, bounded influences.

- **Offer cooldown and daily caps:** These prevent repeated offers or two devices from accelerating drift past intended caps.

- **Plumage saturation never drops:** The plan treats this as an additive trait and repeats the no-decline rule.

- **Week-one, week-three, no-session-jump, and two-week-absence fixtures:** The plan uses these to calibrate change: detectable around week one, noticeable around week three, no single-session jump, and healthy familiar birds after absence.

- **Mood enum with timers, transition probability, hysteresis, and minimum dwell:** The plan uses these to prevent jitter and make mood depend on prior mood, local time, offers, weather, other calls, and traits.

- **Daily-ish reset as gradual time-of-day pull:** The plan rejects neutral resets at midnight or page open, preserving continuity.

- **Seeded rain and wind schedules:** Rare weather adds ambient variation that briefly affects calls and mood probabilities.

- **Stored IANA timezone with DST-aware conversion:** This makes local lighting and ticks correct while a timezone change affects only future lighting/ticks, not identity or history.

- **Night behavior with nightjar-like late calls:** The plan uses species behavior to let night settle most birds while leaving one kind eligible for occasional late calls.

- **Perch zones selected from mood and boldness with spacing constraints and stable IDs:** This ties visible position to canonical mood and traits while preserving identity and layout constraints.

- **Call events and bird-to-bird responses:** Calls can nudge nearby moods and create "organic chorus windows" without a synchronized return cue.

- **Return greeting scheduling:** Absence, boldness, warmth, mood, seeded variation, staggered secondary responses, and visitor suppression make greetings specific, non-synchronized, and non-canned.

- **Notebook generation from salient canonical observations:** Rarity budget, distinctiveness threshold, dedupe window, and stored rendered prose keep notebook entries sparse, observational, and historically stable.

- **Age-based adoption availability from aviary creation time only:** This prevents adoption from being a reward for session count or interaction volume and preserves the seven-bird cap under concurrent devices.

### Scene and audio pipelines

- **Normalized scene coordinate system with front, middle, and back perch zones:** This supports responsive layout and consistent bird/perch placement across viewport changes.

- **Viewport layout solving bird bounds and perch spacing:** The plan uses phone compression, uniform scaling, and wide-viewport air to keep silhouettes visible and hit areas accessible.

- **Small silhouettes over a calm sky/foliage scene:** The plan gives the visual direction as restrained foreground depth with compact assets, supporting performance and a calm aviary.

- **Top bar as the only place for account, accessibility, notebook, and offer controls:** The plan keeps buttons, labels, tooltips, and counters outside the scene.

- **Top-bar opacity falls after stillness and returns on pointer or keyboard activity:** This preserves access to controls while reducing visual chrome during quiet watching.

- **Bird pose initialized from snapshot behavior phase and elapsed server time:** The first frame can be mid-action instead of a new entrance.

- **Small server-provided timeline with interpolation and gradual clock correction:** This bridges snapshots and avoids snapping except on large-gap resync with a soft pose transition.

- **Local ornaments and parallax:** These add subtle motion but are local-only ornaments that do not enter canonical state or drift.

- **Pause rendering and ornaments when hidden, then fetch fresh canonical state on return:** This avoids hidden-tab work and reconnects to canonical simulation after visibility or browser resume.

- **Quiet cold uncached field instead of spinner or fade:** The plan wants the first scene to remain visually coherent and not imply a new aviary.

- **Only real post-adoption empty state gets a soft first-bird fly-in:** Entrance motion is reserved for the genuine adoption state, not ordinary loading.

- **Settle lighting fade and lowered calls:** Settle creates a local closing transition while normal day/night lighting continues independently of tab openness.

- **Reduced-motion renderer:** The renderer keeps the same canonical state and actions while replacing micro-motion and flight paths with still-pose and perch-to-perch cross-fades, removing drifting leaves/feathers, and honoring changes live.

- **Versioned call grammar per species with stable per-bird signature seed:** This keeps each bird recognizable across moods and drift while allowing generated variation.

- **Vocal-frequency trait changes call probability and chorus participation, not identity motif:** The plan protects call identity from being rewritten by drift.

- **WebAudio synthesis or AudioWorklet with pooled voice objects and bounded polyphony:** This implements generated calls, schedules against the audio clock, and controls performance and mixing.

- **Restrained ambient bus, limiter, and volume control:** The plan uses these to keep multi-bird audio bounded and listenable.

- **Chorus as overlapping independent generated calls:** This avoids layered recordings and supports organic chorus windows.

- **Listen-in ramps focused bird up and other birds to quiet nonzero ambient:** The plan focuses attention without muting the rest of the aviary completely.

- **Specified disengagement transitions for same bird, another bird, empty space, and keyboard focus departure:** These make focus changes predictable across pointer and keyboard use.

- **Song-fragment offer through the same synthesis path with server-decided response:** This keeps offer audio consistent with call grammar and makes response depend on mood and personality.

- **Captions generated from actual grammar event parameters:** Captions describe the call that would play, including bird/location and audible contour.

- **Autoplay and WebAudio failure handling:** The first scene renders without waiting for audio permission; captions and silence maintain access until sound is available, with no intrusive prompt, no recorded fallback, and no audio recording.

### Accessibility, interaction, and copy

- **Meaningful scene focus region with individual bird controls in the accessibility tree:** The plan keeps visual chrome outside the scene while making birds operable and perceivable to assistive technology.

- **Keyboard traversal and commands:** Tab, arrows, Enter, Escape, and focus-departure behavior make listen-in and bird navigation keyboard-complete.

- **Keyboard-complete offer menu, notebook, settle, settings, and dialogs:** The plan requires visible focus across day and night backgrounds and touch targets that survive phone compression.

- **Screen-reader narration from snapshot and grammar events:** The plan uses shared render/audio sources so narration is naturalist, current, and tied to actual scene events.

- **Slow idle narration and queued user-initiated observations:** The 30-60 second pacing, coalescing, and stale-update handling avoid flooding the live-region queue.

- **No narration of raw status codes, perch numbers, or trait values:** This keeps accessibility from becoming a state dump.

- **Call captions optional normally and default when WebAudio is unavailable:** Captions preserve access when sound cannot play.

- **Captions near the calling bird with readability and no cover over another bird:** The plan wants captions to be spatially useful without blocking birds.

- **Text alternative in the accessibility tree:** This gives captions an assistive-technology equivalent.

- **WCAG AA contrast across daylight, dusk, rain, and settled lighting:** The plan accounts for changing scene conditions, not just one background.

- **Authored naturalist copy and direct system copy:** Product surfaces use naturalist observation; sign-in, errors, revocation, and settings use direct system language.

- **Testing combined accessibility modes:** The plan explicitly tests screen-reader, keyboard-only, captions, muted audio, reduced motion, and combinations rather than isolated toggles.

### Performance, privacy, and operations

- **Initial JS under 2 MB gzip, first bird visible within 500 ms, 60 fps idle, and no sustained memory growth:** The plan treats these as hard budgets and says 500 ms is a "performance gate, not a best-effort aspiration."

- **CI bundle budgets and mid-tier mobile cold-load lab profile:** These make the performance budgets enforceable before launch.

- **First-scene code and tiny assets in the critical path:** This supports the first-bird timing target.

- **Lazy-loaded notebook, account, invitation, and settings views:** These keep non-first-scene surfaces out of the critical path.

- **Bounded ornaments, voice pool, AudioContexts, timers, and notebook virtualization:** The plan uses bounded resources to protect frame time and memory.

- **Scene work below 16.7 ms with browser headroom:** This supports 60 fps idle on older hardware.

- **Aggregate-only operational metrics:** The plan collects request counts, latencies, first-bird timing, frame-time histograms, memory trends, audio errors, snapshot freshness, event acceptance/retry counts, tick lag, and auth/invite errors without private per-bird or notebook content.

- **Logs with opaque request and synthetic account IDs only:** The plan forbids emails, invitation secrets, per-bird state, presence histories, and notebook prose in logs.

- **No population drift dashboard or analytics warehouse copy of event tables:** The plan keeps drift calibration out of private interaction-history aggregation.

- **Deterministic synthetic fixtures and consented qualitative research for calibration:** This is the allowed path for tuning drift instead of population interaction analytics.

- **Per-account operational path under strict access control:** The plan allows corruption diagnosis while keeping that path out of telemetry.

- **Synthetic browsers from common geographies and tick-lag alarms:** Scheduled synthetic monitoring and p99 tick alarms support operational health.

- **Unsupported browser explanatory surface:** The plan handles unsupported clients directly instead of partial or misleading behavior.

- **Network outage read-only local view:** The last coherent snapshot remains visible with a matter-of-fact status outside the scene, but unverified offline presence is not credited.

- **Short authenticated queue for bounded idempotent events:** This supports brief interruptions while requiring canonical state before queued outcomes play back.

- **Expired session asks for sign-in rather than inventing local drift:** The plan preserves server authority when authentication is gone.

### Execution sequence and acceptance gates

- **Foundation phase:** The plan starts with schema, synthetic IDs, encrypted email, magic-link consumption, sessions, deletion/export scaffolding, privacy boundaries, and API contracts because these are prerequisites for identity, privacy, and authenticated mutation.

- **Canonical engine phase:** Deterministic ticks, checkpoints, idempotency, presence union, trait drift, mood/weather, schedule, and adoption are gated with replay, crash/retry, absence, DST, monotonic trait, and migration tests to prove the server simulation is durable and stable.

- **First scene phase:** Snapshot delivery, responsive composition, mid-action first frame, greeting, idle behavior, and hidden-tab resume are gated by cold-load first-bird timing and no spinner or entry sequence.

- **Interactions and voice phase:** Listen-in, offers, settle/undo, notebook, and copy are gated by mood reaction coverage, no repeated canned greetings, sparse notebook output, and no user-attendance prose.

- **Audio and accessible renderers phase:** Grammar, mixing, captions, reduced motion, narration, keyboard, and contrast are gated by recognizability, screen-reader and reduced-motion qualitative tests, failure tests, and 30-minute frame/memory runs.

- **Visits and release phase:** Invitation redemption/revocation/log, read-only projection, settings, threat/privacy review, and monitoring are gated by revocation on next pull, visitor event rejection, no visitor drift, no default notification, and no hidden discovery.

- **Rollout behind server flags:** Internal aviaries, then a small opt-in cohort, then broader availability let drift and accessibility reviews happen before broad launch.

- **Increase supported count from two toward seven only after tests:** The plan links higher active counts to chorus recognizability and scene-density tests, while keeping the seven cap enforced from day one.

- **Launch gates require budgets and privacy boundary:** The plan says launch is not justified by "a pleasant demo" unless performance and privacy gates are met.

### Key risks and explicit decisions

- **Drift too fast, too slow, or falsely credited:** The plan mitigates this with strict three-signal presence, heartbeat lease, cross-device union, week-one/week-three fixtures, per-day caps, qualitative review, and no trait reduction on neglect.

- **Lost vectors or divergent devices:** Server-only additive deltas, serialized ticks, atomic checkpoints, immutable bird IDs, backup/restore drills, migration invariants, and no client last-write-wins are the stated mitigation.

- **Tick backlog after outage:** Durable partition leases, idempotent catch-up, lag alarms, bounded replay, controlled reconciliation, and ticking inactive aviaries prevent backlog from manufacturing attendance or causing sudden client jumps.

- **Calls sound canned or chorus blurs:** Stable grammar signatures, bounded variation, pooled synthesis, seven-bird listening tests, nonzero listen-in mixing, and no recorded loops are the stated mitigation.

- **Autoplay prevents audible first call:** The plan decides to draw instantly, start WebAudio only after a permitted gesture, and rely on captions and silence rather than promising prohibited audible autoplay.

- **Accessibility becomes a state dump:** Shared narration/caption grammars, slow live-region pacing, designed reduced-motion poses, and human testing across combined modes are the stated mitigation.

- **Privacy leaks through logging or exports:** Schema separation, synthetic IDs, secret redaction, restricted export jobs, short-lived links, no analytics access to the interaction store, and hard-deletion verification are the stated mitigation.

- **Browser/device variance breaks first frame:** Edge snapshots, compact critical bundle, asset fallback silhouettes, lab runs, and no spinner are the stated mitigation.

- **Remaining tunables as versioned server configuration:** Exact activity window, tick cadence, trait rates, offer cooldown, greeting timing, weather frequency, and adoption thresholds remain tunable so calibration can change "without resetting a bird or altering old notebook entries."
