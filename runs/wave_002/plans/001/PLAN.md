# Pocket Aviary — v1 implementation plan

Assigned run: `wave_002/plans/001`. Deliverable: an implementation plan, with no product implementation or evaluation performed.

This plan is based on all ten required PRD files: `1-START_HERE.md`, `product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, and `non_goals.md`. Decisions below fill implementation gaps; calibration constants are starting values to validate, not measurements of a built product. No external design document is assumed to have been supplied.

## 1. Product boundary and delivery outcome

Ship a browser aviary that is already in motion when opened. Two individually recognizable birds begin a persistent relationship with the owner. The relationship changes slowly through measured attention; moods and ambient behavior continue on the server while nobody watches. All devices observe one canonical aviary. Seven birds is a hard maximum, including during migrations and concurrent adoption requests.

V1 includes six coherent species designs, procedural calls, the three perch zones, local-time lighting, rare gentle weather, return-greetings, listen-in, three offer types, settle with undo, sparse read-only notebook observations, naming and renaming, age-based adoption, email magic-link accounts, device revocation, verified email change, export, deletion and recovery, and deliberately invited read-only visits. Screen-reader narration, call captions, reduced motion, keyboard access, and performance instrumentation ship with the first usable slice.

Exclude native apps, payments, subscriptions, multiple aviaries per account, shared ownership, scene customization, dragging birds, public discovery, profiles, follows, comments, chat, visitor avatars, shared cursors, rankings, rewards, achievements, streaks, counters of adoption or owner attendance, hunger, distress, death, decaying happiness, and any client-owned personality simulation. Do not build latent aggregate interaction datasets or social-ranking statistics. No prerecorded bird calls, including as an error fallback. No welcome toast, absence counter, notification badge, loading spinner, or normal-return entrance sequence.

The implementation is successful only if the same specific bird remains recognizable after renaming, absence, another device, and a software update; a regular observer senses a change after weeks without seeing numbers; and accessible presentations carry the same specificity and calmness.

## 2. Explicit interpretation decisions

These decisions allow implementation without questions to the author. They must remain visible in engineering acceptance criteria rather than become undocumented exceptions.

| Ambiguity or tension | V1 decision |
| --- | --- |
| Four top-bar icons are listed, while settle must also be reachable from the top bar. | Keep the four icons: account/settings, accessibility, notebook, and offer. The offer popover contains the three offers and a separated `settle` action. It is an action reached from the top bar, not a fifth persistent icon or a control inside the scene. |
| Keyboard focus initiates listen-in in the interaction description; the keyboard specification also requires Enter. | Focusing a bird begins listen-in. Enter explicitly toggles it, including re-engaging after Escape while focus stays put. Arrow focus transfer moves listen-in to the next bird. Escape ends listen-in without trapping focus or immediately restarting it. |
| A minute-scale tick must coexist with a greeting in 1–2 seconds and prompt offer reactions. | Schedule every aviary at 60-second intervals regardless of connections. User events can wake the same serialized tick reducer early. Its elapsed-time accounting prevents extra drift or daily mood resets when invoked more often. There is no second simulation writer. |
| Devices may be in different timezones. | Persist one IANA timezone on the account, initially suggested by the first browser and subsequently changed explicitly in settings. Every client and visitor uses it. A device's different timezone does not silently overwrite the account clock. |
| Settle persists locally until leaving or re-engagement, while all devices share mood. | Settle ends that viewing session's presence and applies its evening presentation overlay until that session closes or re-engages. A small server-authored mood-quieting impulse is canonical. The overlay is a viewing-session gesture, not a replacement for the aviary's shared day/night clock. Other active owner sessions continue to count valid presence. |
| The brief describes muting as meaningful but does not assign a drift direction. | Track audio preference for that account's experience only. Audible and muted qualified presence count equally; caption users get equal drift. Do not treat mute as rejection, quiet the stored vocal trait, or create a hidden listening requirement. |
| Export explicitly includes current vectors, while concepts prohibit ever showing their numerical values. | The stronger non-exposure rule governs the user-facing export. Include a versioned, lossless, encrypted personality-state block per bird, sealed by the account's archival key, alongside readable names, moods, settings, and notebook. Do not include plaintext trait values or a client decryption key. Label this machine-preserved state plainly in the export documentation. This preserves a copy but deliberately limits independent interpretation; it is not a claim of a fully interoperable numeric export. A future decision to expose cleartext would require changing the PRD, not an engineering toggle. |
| General notification prohibitions coexist with an explicit optional visit-notification setting. | Treat `social_optional.md` as a narrowly named exception: implement an optional plain email notice of a completed visit, off by default and available only in settings. No push infrastructure, in-aviary announcements, badges, onboarding prompt, or aviary-reminder emails. Authorization is the explicit toggle; disabling it also cancels undelivered notices. Default behavior is silent logging only. |
| Revocation says the visitor disappears from the visit log, yet the log supplies historical sharing transparency. | Remove the outstanding/active invitation row immediately; retain a dated historical visit row marked ended/revoked until its retention period expires. Do not leave an active-looking visitor, show a success toast, or erase historical disclosure evidence as a side effect of revocation. |
| Used invitation lifetime is unspecified; unused invitations expire at 30 days. | Exchange a one-use link for one render-only browser session, with a two-hour absolute lifetime and a 15-minute inactivity lease. Reloads within that session work. Reopening the consumed link does not mint another session. The host must deliberately invite again for a later visit. |
| Exact adoption intervals and presence activity window are unspecified. | Start with a four-minute pointer/key activity window. Offer bird numbers 3–7 at aviary ages 90, 180, 270, 365, and 540 days. These are versioned calibration choices, never visit-count thresholds. |

The encrypted export choice and the notification exception are intentional product tradeoffs, not claims that the source documents are perfectly consistent. Build and test the stated behavior so a later policy change has a single, explicit place to start.

## 3. Architecture and ownership

Use a typed web stack with a small SVG scene runtime, an HTTP application service, a simulation worker, PostgreSQL as the authoritative transactional store, a due-work scheduler, and encrypted object storage for exports and backups. Use TypeScript for the browser, API contracts, narration/call descriptor types, and the pure simulation reducer to avoid mismatched event semantics. The implementation team pins supported dependencies at build kickoff; the plan does not depend on an unverified framework version.

The application service and worker can deploy separately from a single repository. Begin with a modular service and database-backed job/outbox tables, not a broker fleet or one process per bird. Partition work by synthetic aviary/account UUID when scaling. A CDN serves public hashed assets; an authenticated edge handler delivers private HTML and a small inline presentation snapshot. Private responses never enter a shared public cache.

Boundaries:

1. **Auth/account module:** magic-link verification, encrypted identity data, device sessions, timezone/preferences, deletion, export authorization, and invitation access. It never writes an existing bird's personality.
2. **Command/event ingress:** validates the actor, schema, idempotency key, session lease, and resource ownership; allocates an aviary-local sequence and appends an event transactionally. No client absolute trait writes are accepted.
3. **Simulation worker:** the sole owner of canonical bird mood, existing personality vectors, behavior plans, call events, cooldown outcomes, and notebook facts. It consumes ordered events and advances scheduled time in a transaction.
4. **Snapshot projector:** reads a committed simulation revision and produces safe presentation data. It removes raw trait vectors and private evidence, preserving mood-shaped action descriptors, positions, timing, and grammar parameters needed to draw and synthesize.
5. **Browser scene/audio runtime:** evaluates the presentation timeline against estimated server time. It draws poses, interpolates authoritative transitions, synthesizes authorized calls, captions them, and changes local mix/presentation preferences. It never makes persistent behavioral or drift decisions.
6. **Operational metrics collector:** accepts a restrictive metrics schema. It has no access to the simulation database or event log. Rendering performance and infrastructure health leave the product boundary; bird behavior and interaction history do not.

Use a shared semantic observation layer as input to notebook prose, narration, and call captions. Share facts and vocabulary, not one generic string emitted at three different cadences. Species assets, the regular-motion renderer, the reduced-motion renderer, the audio synthesizer, and narration all consume the same versioned behavior/call descriptors.

### Invariants enforced at boundaries

- An active account has exactly one aviary; an initialized aviary has 2–7 birds, immutable bird IDs, and persisted personality vectors.
- Trait updates are nonnegative server-authored deltas. Rename, login, migration, event retries, and lack of attendance cannot reset or overwrite them.
- Every tick commits state revision, consumed event cursor, generated observations, and next due time atomically. Failed commits do not acknowledge consumed evidence.
- Owner presence is the union of valid intervals across sessions, not the sum of device durations.
- Visitor authorization cannot reach any owner interaction/event endpoint, including greeting or presence.
- Snapshot, DOM, ARIA, errors, product debug surfaces, telemetry, and plaintext export fields contain no personality trait numbers.
- A rendition can be paused because its tab is hidden; the server simulation cannot be paused because no clients are connected.
- Private state is never used for population-level drift calibration, recommendation, training, or engagement analysis.

## 4. Persistent data model

Use UUID primary and foreign keys. Store instants in UTC and retain the canonical IANA timezone for diurnal interpretation. Store exact tick/cursor numbers as integers. Use database constraints and transactions for correctness rather than assuming an API call executes once.

| Record | Essential fields and rules |
| --- | --- |
| `accounts` | `id`, encrypted verified email, auth-only blind lookup index, status, created time, verified time, timezone, preferences revision, deletion deadline, account data-key reference. Email is stored once here and is never an identifier, route parameter, queue partition, metric dimension, or log message. |
| Pending invite identity | Represent an unregistered recipient by an account row with `invited_pending` status, UUID, and encrypted email; no aviary, owner privileges, onboarding, or marketing follows from invitation. Reuse it on explicit account creation. References in invitations point to this row rather than duplicating emails. Remove orphaned pending identities after associated invitations/logs expire. |
| `aviaries` | `id`, unique `account_id`, adoption-initialized time, state revision, event high-water mark, last simulated time, next due time, scheduler fencing/version field, engine version, RNG state, environment state, archive-key reference. Age starts at aviary creation and is independent of attendance. |
| `birds` | `id`, `aviary_id`, species/version, name, adopted time, ordinal, immutable signature seed/profile, trait schema and five canonical values, mood, mood-since time, transient influence timers, current perch/action phase, persisted low-pass state, drift remainder, next offer eligibility, last greeting facts. Numeric traits remain in the protected simulation storage type. |
| `behavior_segments` | Aviary revision, bird ID, semantic action, start/end server times, source/target perch and pose parameters, descriptor seed. Compact rolling schedule, pruned after consumption; no indefinite animation-frame history. |
| `call_events` | Stable event ID, bird ID, start/duration, grammar version, ordered motifs, bounded pitch/timing/timbre descriptors, response relationship. Preserve enough canonical schedule for snapshots; do not store synthesized waveform files. |
| `device_sessions` | ID, account UUID, token hash, created/last-seen time, absolute expiry, revocation time, short user-recognizable browser/device label. No browser fingerprinting. |
| `view_sessions` | Session UUID, device-session UUID, owner/visitor mode, opened time, last confirmed heartbeat, presence sequence, activity deadline, terminal/settled state, listen target and lease. Listen mix position itself is local. |
| `interaction_events` | UUID, aviary-local ordered sequence, unique actor session/client event key, account UUID, event type, optional bird UUID, validated payload, client sequence, server-received/effective time, schema version. Append-only until controlled retention/deletion. No coordinates or literal typed keys. |
| Presence evidence | Bounded closed intervals and checkpointed daily credited totals, with view-session reference and validation flags. The worker unions intervals across devices and persists the consumed portion. Never expose totals as a user metric. |
| Drift state | Per-bird, per-trait positive evidence filter, daily capped presence/listen/offer contributions, last integration time. Store with canonical bird state. Raw logs are not needed to reconstruct a lost vector and must not be treated as its backup. |
| `notebook_entries` | ID, aviary UUID, stable sort timestamp and ID, prose, template version, fact references, optional bird IDs, historical name text. Immutable and indefinitely readable until account deletion. No edit/delete/annotation API. |
| `observation_state` | Private recent greeting order, weather/pose noteworthy facts, notebook sparsity budget, repeat-suppression keys. Compact bounded memory of aviary facts, not a diary of user attendance. |
| `adoption_offers` | UUID, aviary, ordinal, eligibility timestamp, assigned species/signature seed, accepted/declined/deferred state. Persist the proposed bird so reload cannot reroll it. |
| `invitations` | UUID, host account UUID, recipient account UUID, capability hash, created/unused-expiry/redeemed/revoked time, revision, notification consent at delivery time. One-time redemption is transactional. |
| `visit_sessions` / `visit_log` | Invitation UUID, token hash, started/last-read/ended time, capability version, expiry, approximate duration. Access-controlled host log contains recipient identity through an authorized join. Never enters the simulation event stream. |
| `auth_challenges` | UUID, account UUID, purpose, hashed random token, issued/15-minute expiry/consumed times; new-address ciphertext in a separate short-lived verification challenge until verified. Delete it on consumption/expiry. |
| `outbox_jobs` / `export_jobs` | UUID, account UUID, job type/state, idempotency key, retry time, encrypted object reference/expiry. Emails resolve the destination at send time instead of copying it into queues. |

Names: accept 1–32 Unicode grapheme clusters after trim, reject control characters and markup, and escape every rendering and mail context. Provide natural default suggestions. Name collisions within an aviary are allowed; accessible controls disambiguate by species/perch. Renaming modifies only name and display revision, not ID, traits, signature, or previous notebook prose.

Initial trait values are seeded in a moderate band, approximately 0.20–0.50, to leave expressive headroom and preserve bird differences. A newly created bird receives its vector once within the adoption transaction. Check all values are finite and inside the normalized range. A migration adding a trait must preserve every existing value and identity, initialize only the new field deterministically, and undergo before/after continuity tests.

Indexes include unique aviary/account, events by aviary/sequence, idempotency key by view session, notebook by aviary/time/ID, due ticks by next-due time, live sessions by account, and invitation token hash. Enforce the seven-bird cap while holding the aviary row lock; a count checked outside the transaction is insufficient.

## 5. Server tick and canonical ordering

### Scheduling and transaction

Start with a 60-second scheduled cadence and distribute initial due offsets using a stable hash of aviary UUID. A dispatcher claims due work in bounded batches using database row locks or equivalent leases. Multiple workers may poll; only the worker holding the aviary lock with the current fencing token can commit. Deduplicate scheduled and user-event wakeups by aviary. Interactive events can trigger an early pass, coalesced over approximately 100 milliseconds; they do not postpone the next regular due tick.

For each pass:

1. Read the committed state, last simulated instant, persisted filter values, and event cursor under the aviary lock.
2. Select events after the cursor up to a fixed high-water mark. Order by server sequence; never reorder by a client's wall clock.
3. Advance from the previous instant to each event's validated effective time, applying elapsed-time drift, environment/mood evolution, and any due behavioral facts. Apply the event once. Then advance to the pass target time.
4. Compute nonnegative trait deltas; resolve mood and offer reactions; choose perch/action segments and call motifs for the upcoming presentation horizon. Greeting and accepted user gestures can replace the unexecuted suffix of that horizon.
5. Generate notebook candidates from actually realized facts, subject to sparsity and truth checks. Do not write an observation about a future planned call that might be superseded.
6. Commit birds, environment, canonical presentation schedule, event cursor, notebook entries, snapshot revision, next due time, and any outbox messages in one transaction. Acknowledgment is sent only after commit.

The API service can wait briefly for this pass and return its snapshot for an interaction. It cannot apply its own mood or drift shortcut. Set an initial 400-millisecond processing target for greeting/offer command completion under healthy conditions; after an 800-millisecond wait limit return `202` with a receipt and pollable result. The renderer may acknowledge the control locally, but must not invent an offer acceptance or permanent mood change.

### Time continuity and restarts

Simulation time is server UTC, monotonically advanced per aviary. Segment across timezone/day boundaries when computing daily inputs and diurnal mood. A timezone change is effective from its committed timestamp; it does not rewrite previous days' credited presence or reset the aviary's age. Use UTC elapsed time for leases, cooldowns, adoption, and invitation expiry, so DST changes cannot duplicate credit.

Normal ticks advance regardless of recent access. After a worker outage, catch up from persisted state using elapsed intervals and the stored RNG schedule; do not restart a bird from its seed or infer a new vector from event history. No-input drift decay has a closed-form integral. Environment changes and mood transitions use scheduled change boundaries, allowing long empty periods to skip inert minutes while still accounting for the elapsed diurnal cycles. Process any actual intervening input exactly in order. Cap work per transaction and persist progress between catch-up chunks. A client read never becomes the sole trigger for this catch-up system.

Store seeds, engine versions, and logical random-event counters so a failed transaction retried on another worker produces the same result. Duplicate ticks and event deliveries must be observationally identical to one successful tick. Publish a new revision only after its associated state and schedule are coherent.

### Presentation horizon

Each committed revision supplies roughly 120 seconds of timestamped action/call descriptors. The server determines behavior, interactions between birds, and call opportunities; the client only renders or synthesizes them. Revise future descriptors on changed mood or user actions without replaying already emitted events. A call already sounding finishes with its existing signature and envelope; canceled future calls are removed by event ID.

After an unrefreshed horizon expires, the client can continue harmless bounded pose breathing or reduced-motion cross-fades and the quiet field, but cannot invent new canonical calls, offer outcomes, weather transitions, or notebook facts. Suppress new simulation-dependent sound and display a matter-of-fact connection status in the chrome. On recovery, fetch the current revision and resume at its time, never fast-forward missed audio through the speakers.

## 6. Exact presence and slow drift

### Presence collection

The presence state machine is active only for an authenticated owner viewing their own aviary. Its predicate is:

`document.visibilityState === 'visible' AND document.hasFocus() AND trusted pointermove-or-key activity within 4 minutes AND view session not ended/settled`.

The first three terms are mandatory. Read the document properties when starting and ending each sample, and subscribe to focus/blur, visibility, pagehide, and qualifying activity. Use trusted `pointermove` and physical keyboard activity corresponding to the PRD's keypress definition; support modern `keydown` where the deprecated keypress event omits navigation keys. Do not count a bare click, scrolling, audio playing, foreground display alone, or network polling as attention. Never manufacture activity from programmatic focus or synthetic events. Store only activity timestamps/booleans, never pointer paths or typed content. Validate touchscreen pointer movement and keyboard/screen-reader navigation in real browsers; extend event normalization only if it represents the same specified action, not a looser “tab open” signal.

Maintain monotonic local interval times. Send closed, retrospective presence samples approximately every 15 seconds while eligible, with per-view sequence and claimed elapsed seconds. Flush the final partial interval on blur, hide, settle, and pagehide using a best-effort beacon. At an activity-window expiry, close the interval even if no input event arrives to trigger it. If a browser crashes, only previously confirmed intervals count; the server must not credit a forward lease just because a tab was once open.

The server bounds each claimed interval by time since its previous acknowledgment, maximum sample duration, active session lifetime, and timestamp skew. It clips or rejects stale/overlapping sequence data and unions valid intervals across devices. No sample may retroactively claim hours of offline attention. The browser signals are an honest attention approximation, not a claim of cryptographic proof of human gaze; rate limits and caps bound fabricated traffic without invasive monitoring.

Four minutes intentionally permits still watching. It does not turn a 30-minute motionless tab into 30 minutes of presence. This consequence is documented in calibration tests; no “stay active” nags or attendance indicators compensate for it.

### Drift function

Keep all five traits on `[0,1]`. The persistent state is the current vector plus a slow positive-evidence filter for each trait; updates never derive the vector anew from events.

Initial calibration model:

- Count at most 24 minutes of unioned owner presence per account-local day, with 12 minutes as the synthetic regular-visit reference. Caps are invisible and affect drift only, not the ability to keep watching.
- For each bird/trait, presence contributes weight 0.8. Qualified listen-in contributes up to 0.1 for social warmth and vocal frequency. Resolved offers contribute up to 0.1 for relevant curiosity/boldness traits. Each secondary component is bounded by that day's already qualified presence allowance; button traffic cannot become the dominant source.
- Offers near a bird produce the smaller boldness evidence; server-confirmed acceptance adds curiosity evidence. Listen duration is measured as the overlap of the listen lease with qualified presence, not a client-submitted number accepted on faith. Settle has zero directed personality evidence.
- Convert capped evidence to a nonnegative rate `u_j(t)` normalized so a reference day contributes at most one evidence-day. Integrate the persisted low-pass state with time constant `tau = 3 days`: `dE_j/dt = (u_j - E_j) / tau`. This lets evidence acquired before departure continue to have a small effect while absent.
- For elapsed time, integrate `A_j = integral(E_j(t) dt)` and compute `delta_j = (1 - p_j) * (1 - exp(-k_j * A_j))`, initially `k_j = 0.012 per day`, with trait-specific tuning bounded near 0.008–0.014. Apply `p_j := p_j + max(0, delta_j)` under the tick transaction. Use closed-form integration for constant-input intervals and split at evidence boundaries.

Persist fractional precision so small deltas are not rounded away. The exponential formulation limits values smoothly near one, composes across arbitrary tick subdivisions, and ensures an accelerated interactive tick cannot grant an extra fixed increment. Filters can decay in absence; the personality vector cannot. Zero evidence after a long absence approaches zero further drift, not negative drift. Starting seeds retain long-lived differences; later rates are tested for convergence that would make every bird feel alike.

For synthetic birds near 0.3 with 12 qualified minutes daily, aim for approximately 0.02–0.04 measurable trait change after seven days and 0.08–0.13 after 21 days, subject to trait and interaction mix. These are engineering calibration bands, not UI statistics or measured production behavior. A reference single 20-minute session should change no trait by as much as 0.001. Validate the integral with a long-run simulator before choosing final constants; if the proposed constants miss the bands, adjust them through versioned configuration, not by rewriting existing vectors.

### What absence may change

Absence never lowers boldness, social warmth, vocal-frequency trait, plumage saturation, or curiosity. It does not bias mood toward wary, hunger, sadness, sickness, or anger. Recent-interaction mood impulses and extra response opportunities can naturally expire, leaving ordinary ambient behavior. Stored vocal propensity remains intact even when night, rain, or the end of a listen-in makes current calls quieter. On a later return, absence length shapes a new greeting; it is not a penalty factor or a visible absence counter.

## 7. Mood, gestures, species, and notebook behavior

### Mood and bird-to-bird behavior

Use five initial moods: wary, content, curious, drowsy, and alert. “Settled” is a presentation/pose condition, not a sixth negative or rewarded personality state. Implement a versioned semi-Markov transition model with minimum dwell times and bounded transition hazards. Suggested initial dwell range is 5–30 minutes, with an interaction able to prompt a short response sooner. Time of day, weather, recent interactions, neighboring calls, and personality influence transition weights.

Daily-ish reset means transient mood influences lose their force over roughly 18–30 hours and diurnal weighting changes smoothly; it does not mean midnight resets every bird to content. Login has no neutralization branch. Content favors preening; curiosity favors head tilts and investigation; wariness favors the back perch and scanning; drowsiness favors low, fluffed poses; alertness favors upright scanning and more readiness to respond. Never attach visible mood labels or meters.

Simulate birds jointly within an aviary tick. A call can schedule a response in another bird after a species-appropriate randomized offset; a wary cue can temporarily increase nearby wariness with bounded probability. Response depth is capped at two and a short refractory window prevents an alarm or chorus feedback loop. Weather and bird-to-bird signals affect transient expression, not negative trait updates. Night dims the scene gradually; most species settle, while the nightjar-like species retains a low but real chance of calling.

### Greeting

On navigation or a visible-return transition, submit one idempotent `arrival` event for that view transition. This event requests a greeting but is not presence by itself. The worker determines absence from confirmed viewing-session transitions, combining the returning view's gap with the aviary's last owner-observed time. A fresh view has no reason to invent an absence duration. A long absence is kept private.

Select one initial greeter using boldness and social warmth tempered by mood. A wary or drowsy bird may lose the initial selection without disappearing from the aviary. Generate the form from continuous parameter ranges: gaze direction and dwell, head angle, approach distance, optional motif length, onset, and response delay. Short returns tend toward glances; longer gaps can produce a reorientation or approach. Keep a bounded recent greeting descriptor fingerprint and resample to avoid identical cues, while preserving each bird's characteristic tendencies.

Schedule the first bird noticing in 1–2 seconds from visible activation under the supported performance profile. A secondary bird may respond later after a randomized offset; do not greet in unison. Coalesce simultaneous owner-device arrival events within a brief account window to avoid multiple duplicated greetings in the canonical scene. The accessible narration can promptly observe the bird's response, such as `pip lifts her head from the front rail`; it must not say welcome, enumerate absence, or announce a session.

### Offers and settle

The top-bar offer popover offers a seed, a song fragment selected from a small curated motif library, and a still pool. The server chooses the recipient(s) from available birds and their position/mood; the user does not directly click a bird to feed it. Use a three-minute per-bird cooldown shared across all offer types and all owner devices. A gesture can receive natural responses from several eligible birds, but grant each bird's evidence at most once and bound the event's total secondary credit.

Seed responses include approach, watch then approach, and no approach. Song fragments are soft synthesized musical gestures to which birds may join, pause, or call against. Pool responses include drinking, bathing, and observing. A drowsy nonresponse is a valid reaction, not an error. The server sets a short lifetime on an offered object and avoids accumulating persistent objects. Repeated requests during cooldown do not affect traits or restart the object; give a quiet inline popover explanation when needed, not a countdown, success toast, or punitive message.

Settle begins a 4-second evening-light and mix transition on the initiating view and immediately closes that view's presence. Append a uniquely identified settle event with a small transient mood-quieting impulse. Any click anywhere in the scene during the next five seconds sends an undo tied to that settle ID and reverses the presentation smoothly. Keyboard Enter/Space while the scene has focus provides the same undo. A timely undo removes the still-active settle impulse by ID; it does not rewind unrelated canonical state. The server allows bounded transport slack for a client-observed click within the five-second window. After five seconds, explicit re-engagement ends the settled overlay and begins a new eligible presence interval. Pointer drift alone does not undo settle. Tab-close and settle both end presence without drift penalty or a recovery ritual.

### Species and adoption

Author six species with distinct silhouettes, plumage palettes, posture tendencies, and motif families; one has the specified nightjar-like nocturnal signature. The system chooses two different starter species. Show the two birds that arrived with default names and optional name fields; do not offer a catalog, rarity, stat comparison, or avatar configuration.

Create starter IDs/vectors atomically and retain them through incomplete naming. The first-ever initialized account may use the specified quiet field followed by a soft fly-in; reduced motion uses a gentle pose cross-fade. Record initialization so a reload never creates another empty/adoption entrance. Regular returns always show ongoing motion.

At each age threshold in section 2, make the next adoption available on demand in the account's bird/adoption surface. No badge, counter, timer, modal interruption, email, or attention-earned reward. The proposed species and identity seed are stable. Accepting does not depend on visit count; deferring has no penalty. A long-absent owner can accept already age-eligible additions sequentially without a missed-visit condition. The seventh bird may repeat a species, but signature selection ensures the two individuals sound distinct. The API, worker, and database transaction all enforce the cap.

### Field notebook

Generate observations from realized server facts with a curated grammar, not a runtime language-model service or an event-to-text dump. This keeps private data local to the user's simulation and makes factual and tonal tests deterministic. Use bird names, species, perch, observed weather, call contour, and established greeting-order facts. Examples of acceptable constructions: `pip greets before wren this morning, a first this week` and `wren watches the back perch, feathers fluffed against the rain`.

An entry claiming “a first this week” requires retained evidence of greeting order for the relevant local week; if the comparison is unknown, use a noncomparative observation. Never infer “watched for an hour” or “visited every day” from presence and put it into prose. Use historical display names in stored entries; a rename does not rewrite an observation's history.

Begin with an ordinary observation budget averaging one entry per 72 hours of a regularly observed aviary, capacity one, plus a noteworthy-event allowance limited to one additional entry in 24 hours. Suppress repeated topics over a rolling week. Require a meaningful aviary fact, so a timer does not force filler or backfill hundreds of entries after absence. Very active synthetic users must still produce a sparse notebook. A shared observation module can select quiet facts, but notebook persistence is always the worker's responsibility.

Read entries in descending stable cursor order, 30 at a time, with an accessible “load earlier observations” control and retained date context. Keep all historical entries queryable; pagination and view virtualization must not archive them. There is no editing, per-entry deletion, annotation, streak surface, or attendance export. Account deletion removes the whole record under its separate system flow.

## 8. API contracts and client synchronization

Use HTTPS, versioned JSON endpoints, secure HttpOnly same-site session cookies, explicit CSRF protection for state changes, and strict request schemas that reject unknown writable state fields. Token-bearing routes use `Referrer-Policy: no-referrer`, no third-party resources, and scrubbed access logging. GET requests do not consume authentication or invitation links; mail scanners must not spend a one-time capability.

### Owner state and event APIs

| Method and route | Behavior |
| --- | --- |
| `GET /aviary` | Authenticated HTML with current private presentation snapshot and first-frame SVG. If not signed in, show the matter-of-fact magic-link surface. No greeting event is inferred merely from an HTTP prefetch. |
| `GET /api/v1/aviary/snapshot?afterRevision=...` | Return the latest committed canonical presentation projection with `serverTime`, revision, timeline coverage, and a private ETag. Return `304` only after current authorization and revision checks. Never return numeric personality vectors. |
| `POST /api/v1/aviary/views` | Open or resume an owner view, returning its UUID and sequence/heartbeat limits. Include an idempotent visible-transition ID to request the arrival event once. |
| `POST /api/v1/aviary/events` | Submit a bounded batch of arrival, presence interval, listen start/end, offer, settle, undo-settle, re-engage, or view-end events. Return per-event receipts with server sequence, accepted/rejected status, and committed revision when available. |
| `GET /api/v1/aviary/event-results?receipt=...` | Retrieve an authorized event's resolution if its initial response was `202`; include reaction descriptors and current snapshot revision. |
| `GET /api/v1/notebook?before=...&limit=30` | Immutable entries and an opaque stable cursor. No mutation variants. |
| `PATCH /api/v1/birds/{birdId}/name` | Submit only the new name and expected display revision; validate ownership and commit without touching identity/traits. Return a conflict if a concurrent rename changed the version. |
| `GET /api/v1/adoption` | Return the one currently available age-based adoption proposal, if any. No attendance counts, rarity, future countdown, or progress gauge. |
| `POST /api/v1/adoption/{offerId}/accept` | Idempotently name and adopt the persisted proposal; transaction checks age eligibility and cap, initializes the new bird once, and schedules its introduction. |
| `POST /api/v1/adoption/{offerId}/defer` | Hide the proposal from the current adoption interaction without changing age eligibility or penalizing the aviary. |

An interaction envelope contains `schemaVersion`, `clientEventId`, `viewSessionId`, `clientSequence`, `type`, an optional `birdId`, and the minimal type-specific payload. The server supplies actor account ID, authoritative sequence, receipt time, and effective time. A client timestamp is only a bounded diagnostic/time-normalization hint. Never accept `personality`, arbitrary mood/perch state, free-form call grammar code, or purported hours of attention.

Deduplication is unique on `(viewSessionId, clientEventId)` and records a request-body fingerprint. An identical retry receives the original result; reusing an ID with a different body is an error. Grant the ordered event sequence in the same transaction that appends the event. Do not rely on auto-increment values assigned before independently committing transactions; ingress and the tick use the aviary serialization boundary so an event cannot commit behind an already consumed high-water mark.

Initial payload bounds: at most 16 events and 16 KB per batch, a 15-second presence heartbeat, and per-view/per-account request limits high enough for ordinary navigation but low enough to prevent unbounded queues. A cooldown rejection is a semantic result, not a new offer event with another dose of evidence. API errors use stable codes and concise matter-of-fact copy: invalid request, expired session, unavailable visit, stale version, temporary loading failure. Do not reveal whether a guessed invitation was once valid or who owns an unknown bird UUID.

### Snapshot contract

Return one coherent object with `schemaVersion`, `enginePresentationVersion`, `aviaryRevision`, `serverTime`, `timelineStart/End`, account timezone/daylight descriptor, current weather and transition, and an ordered array of birds. Each bird carries its stable ID, display name, species asset version, silhouette/palette rendering data, current mood-derived pose/perch, timed transitions, immutable public signature identifier, and active/upcoming call descriptors. Owner snapshots may include offer availability and pending owner event results; visitors get no private control data, notebook, presence totals, event cursors, or account settings.

Derived numeric rendering values such as coordinates, duration, oscillator frequency, and color are permitted implementation data; they are not labeled trait scores. Do not include the underlying five-dimensional vector under an alternate name. Author a separate serialization type so spreading a database bird record into JSON is impossible without a type/lint failure.

At seven birds, target a snapshot below 16 KB gzip and below 40 KB uncompressed by bounding the descriptor horizon. Immutable motif tables and species assets are cached public assets, not repeated in every response. Action IDs let the browser merge schedules without duplicating a call or greeting.

### Pull cadence, ordering, and reconnect

While visible, pull every 15 seconds with jitter. Fetch immediately on becoming visible, on a render-frame gap greater than two seconds, after an acknowledged interaction that lacks a full snapshot, or after a connectivity restoration. Debounce overlapping triggers into one request, with a pending “fetch again” flag if necessary. Snapshot polling is independent of presence: a visible but unfocused tab may keep rendering and pulling while accruing no presence.

Maintain the highest applied revision. An older response from a slower request must never replace a newer one. Estimate server clock offset from response timing and smooth small corrections; use timestamped trajectories to evaluate the present phase. Interpolate position/pose transitions, not personality values. If two snapshots straddle a completed flight, join smoothly from the last displayed pose to the current authoritative pose without replaying the entire past flight. Reduced motion uses the equivalent cross-fade.

On hiding, stop requestAnimationFrame, stop new audio scheduling, fade/suspend audio, close presence, and cease routine polling. On returning, revalidate the session and refresh state before scheduling new calls. Do not play a backlog. A long device sleep and out-of-order fetches cannot revert mood, names, or a revoked visit.

During a network outage, leave the last safe visual rendition within its validity rules, show system status in the top bar, and disable new canonical gestures when their outcomes cannot be known. Keep only a small in-memory set of unacknowledged idempotent gesture requests. Retry recent acknowledged-unknown submissions with their original IDs; discard unsubmitted stale offers after 30 seconds and do not replay a day's interaction queue. Do not store vectors or a competing simulation in local storage or a service worker. Presence confirmed before a disconnection remains valid; offline time earns no retrospective credit.

Two devices may offer simultaneously; the first canonical sequence gets the per-bird cooldown and the second receives the resulting current availability. Two devices listening to the same bird contribute unioned qualified time, not twice the time. Listening to different birds uses the per-account bounded secondary budget. Preferences/renames use explicit expected revisions and a matter-of-fact reload-and-retry surface if needed; no client chooses between competing personality histories because there is only one history.

## 9. Account lifecycle, security, export, and deletion

### Account endpoints

| Method and route | Behavior |
| --- | --- |
| `POST /api/v1/auth/magic-links` | Request a sign-in email; generic success response prevents account enumeration. Issue a random single-use token with a 15-minute expiry. |
| `POST /api/v1/auth/magic-links/consume` | Atomically consume the token and create a per-device session. The link landing page requires an intentional submit so prefetching does not sign in a browser. |
| `GET /api/v1/account` / `PATCH /api/v1/account/preferences` | Account/system settings, timezone, audio and accessibility preferences, optional visit notices; changes use a settings revision. |
| `GET /api/v1/account/sessions` / `DELETE /api/v1/account/sessions/{id}` | List and revoke device sessions. Revoking a session also invalidates its view leases immediately. |
| `POST /api/v1/account/email-change` / `POST /api/v1/account/email-change/verify` | Start verification of the new address, then commit the switch atomically. Old address remains authoritative until verification. |
| `POST /api/v1/account/exports` | Queue a consistent, encrypted on-demand export; deliver the download link only to the verified address. |
| `GET /api/v1/account/exports/{id}/download` | Require valid account authorization and a short-lived export capability. Stream with private/no-store and download headers. |
| `POST /api/v1/account/deletion` | Mark deletion pending with a server deadline 30 days away after explicit confirmation on the account page. |
| `POST /api/v1/account/recover` | Restore a pending account within the deadline when its owner chooses “I changed my mind.” |

Use cryptographically random tokens with at least 256 bits of entropy and store only their hashes. Start with five requests per email lookup bucket per 15 minutes plus broader network abuse limits. Resolve an address only inside the identity/auth module; its keyed blind index supports equality and rate limiting but is never used as an internal account identifier or emitted in logs. Magic-link tokens are purpose-bound so an email-change link cannot authenticate or redeem a visit.

Device sessions have a 30-day absolute lifetime initially, rotate tokens on authentication/security-sensitive changes, and revoke immediately in the authoritative session store. Cookies are secure, HttpOnly, and same-site; origin/CSRF checks protect every mutation. No authentication state in local storage. Require a recently verified magic-link session for deletion, export, or email change if the current session is older than 15 minutes. The user can receive another magic link without losing their existing aviary state.

Email change verifies the new address before replacing the encrypted account email and its lookup index. If the address belongs to another active account, do not merge accounts or aviaries; give a direct system error. Verification invalidates the challenge and rotates the current credential, with revocation of other device sessions as a deliberate security consequence described on the settings screen. The old email remains valid until that transaction commits. Clear transient new-address data on expiry or success.

### Export implementation

Generate exports from a single committed database snapshot at revision R. Include schema/format version, generation time, account settings, stable bird IDs and names/species, current moods, sealed current personality-state blocks, and every notebook entry. Exclude raw presence/listen/offer history and other users' contact information. Export is not an owner-attendance log. Handle a large notebook by streaming from a consistent snapshot or immutable revision boundary, not by keeping an entire account in API memory.

The sealed block contains the exact canonical vector, trait schema, and necessary persisted drift state encrypted for archival preservation; the JSON does not disclose its numerical values. Keep the archival key within the protected account key system, separate from browser data and observability. No import or trait-inspection UI is added in v1. State the limitation in the export description: the bird's personality state is preserved as an encrypted machine record. This deliberately follows the non-exposure decision in section 2 and requires a product-spec change if fully independent numerical portability later becomes necessary.

Store the completed export encrypted, expire it after 24 hours, and use a download capability lasting at most that long. Recheck session and deletion status at download, not only job creation. Queue payloads and mail-provider metadata contain job IDs and delivery data only, never bird state. A user-requested export email, magic-link email, address-verification email, and explicit invitation email are transactional system surfaces.

### Deletion and recovery

When deletion is requested, mark the account immediately, stop simulation scheduling for that pending-deletion account, terminate view/visitor leases, revoke outstanding invitations, and cancel pending export/notification jobs. Show the recovery action on every signed-in pending-deletion page, including on a newly consumed magic link. Do not keep the aviary silently active behind a deleted-account surface. Soft deletion is a distinct lifecycle state, not ordinary user absence.

Recovery before the 30-day deadline preserves exact bird IDs/vectors, mood records, names, notebook, and age. Resume time advancement from the stored state using the same no-negative-drift rules, with no fabricated presence during deletion. Previously revoked visitors remain revoked; recovery does not silently share the aviary again. An atomic status/deadline check resolves races between recovery and the hard-delete worker. At or after the deadline, recovery cannot recreate a supposedly retained vector.

Hard deletion removes all account-linked live records, including birds, vectors, observations, events, sessions, invitations in either role, visit logs containing that identity, exports, pending email jobs, and account-specific support/error logs. Anonymous aggregate metrics have no account association to remove. Destroy the account's unique data/archival keys so retained encrypted backups cannot restore its private contents; choose a key/storage service whose irreversible erasure semantics actually support the 30-day deadline. Do not call a recoverable key-disable operation deletion. Restore procedures must apply erasure records before serving any restored dataset, and restore drills must prove that a deleted account cannot be recovered from old backups.

Retain processed raw interaction events for at most seven days after successful checkpointing for retry diagnostics and that user's simulation integrity, then remove them. Keep only bounded compact simulation evidence still needed for filters and notebook truth. Idempotency receipts can retain minimal event ID/result metadata for the short retry window without retaining detailed interaction payloads. Notebook entries persist for the account lifetime. Keep visit history for 90 days, disclosed in settings, to support host sharing transparency without an indefinite contact trail; the PRD's indefinite history requirement applies to the notebook. Hard deletion overrides every retention policy.

## 10. Read-only visits

### Invitation API and lifecycle

| Method and route | Behavior |
| --- | --- |
| `POST /api/v1/visits/invitations` | Owner explicitly supplies a recipient email; create a named UUID-backed invitation and send one transactional one-time link. There is no global discoverability flag. |
| `GET /api/v1/visits/invitations` | Owner sees their own outstanding/active invitations and expiration state in account settings. |
| `DELETE /api/v1/visits/invitations/{id}` | Revoke unused or active invitation and its capability version transactionally. Return the updated settings list without a success announcement. |
| `POST /api/v1/visits/redeem` | Exchange an unexpired unused capability for a render-only cookie; atomically mark the invitation consumed. |
| `GET /api/v1/visits/current/snapshot` | Validate active visit, host lifecycle, session lease, and invitation revision on every request, then project the host's current canonical scene. |
| `POST /api/v1/visits/current/end` | Best-effort closure of the visit log; server lease expiry also closes it. No host simulation event is emitted. |
| `GET /api/v1/visits/log?before=...` | Host-only reverse-chronological history with recipient email, date, approximate duration, and current outstanding invitations. No badge or public route. |

Invitation states are `issued -> redeemed -> ended`, with transitions to `revoked` from issued/redeemed and `expired` from unused issued after 30 days. A consumed token is never restored to issued. Token redemption uses an atomic conditional update to prevent simultaneous browsers redeeming one invitation twice. Render-only session credentials are distinct from owner credentials even if the visitor already has their own account. Receiving an invitation is not a grant to mutate the host's aviary.

The visitor accesses the same renderer, canonical moods, positions, calls, day/night clock, and weather as the owner. Strip owner actions and bird listen-in interactivity; do not merely hide buttons while leaving the event API authorized. Do not trigger arrival/greeting or presence on visit open, do not emit attention evidence, and do not permit offers, settle, host notebook access, naming, or adoption. Local audio enable/mute, captions, narration, and reduced-motion controls remain available because they change only the visitor's presentation. They use visitor-local preferences and cannot change host settings.

A visitor's top bar has only relevant system/accessibility/leave controls; no unusable owner offer/notebook controls. No visitor-specific flattering scene, cursors, avatars, comments, chat, host-online indicator, mutual-visit invitation, automatic return invitation, or live co-presence message is added. Owner-generated gestures may naturally appear as changes in the canonical host scene, but no identity or “someone is here” cue accompanies them.

Poll visitor state every 15 seconds while visible and validate before every response, including any ETag/304 path. On revocation, the next pull returns the same matter-of-fact unavailable surface as an expired link, clears the host scene, stops audio, closes the visit lease, and drops cached private data. Normal maximum visible revocation delay is the poll interval plus request latency. Hidden visitors must validate before showing the scene on return. If disconnected, allow no continued access past a 30-second authorization freshness deadline; fail closed to an unavailable/connection surface instead of retaining the host view indefinitely. This realizes “next snapshot pull” revocation without promising instantaneous network delivery.

Visit duration uses server read timestamps and bounded inactivity intervals, not the owner's presence algorithm. A hidden visitor does not keep extending duration through timers. Store the record silently for the host. When the optional notice toggle is on, send at most one plain system email after a completed visit, using only recipient/date information and no bird details, return prompt, streak, or marketing text. Check consent again at send time. Off means no visit email, no in-product notification, and no badge; the log remains accessible on demand.

## 11. Visual rendering and responsive composition

### First frame and scene runtime

Use an SVG scene with a bounded imperative animation layer and small DOM controls above it. SVG supplies compact species silhouettes, direct transforms, and a shared server/client first-frame representation without a large game engine. Keep framework rerenders out of requestAnimationFrame; a single scene clock writes transforms/opacity to persistent nodes. Use procedural feather/palette variation where inexpensive and compact static SVG shape assets otherwise. Do not use external font or audio downloads to gate first paint.

The authenticated HTML contains the latest committed presentation snapshot and an SVG already evaluated at its time. Critical styles and a minimal scene bootstrap continue the current phase before account/notebook/settings modules load. Server/client share the descriptor-to-pose evaluator and asset versions, preventing hydration from resetting transforms or replacing all bird nodes. Negative phase offsets or equivalent timestamp evaluation begin breathing/preening midway through the existing action. No delayed visibility, entrance fade, scale-from-zero, or spinner surrounds the birds on return.

If authenticated state truly cannot arrive, paint the quiet sky/field immediately and a faint non-bird ambient cue where motion preferences allow it. Do not display invented starter birds or another account's cached scene. On an extended failure, a matter-of-fact error and retry action appear in system chrome. First-time adoption alone uses the specified initial fly-in; its reduced-motion equivalent is a slow pose cross-fade.

### Geometry

Use a normalized landscape stage containing fixed front, middle, and back perch bands. Assign safe bird slots within each band on the server with a layout projection on the client for viewport geometry. Perch selection is behavioral; collision resolution can shift an x coordinate within that chosen band but cannot move a wary bird to the front merely for layout convenience.

Fit the landscape stage inside available viewport height below the top bar, preserving bird shape aspect ratios. On narrow phones, compress the space between slots and reduce ornament density while retaining every bird's full body and a minimum interaction target. At seven birds distribute across the three bands with staggered positions; reserve edge and caption margins. On very wide screens add breathing space between perches rather than scaling birds without limit. Use a maximum content width centered in the quiet field if needed.

Test at 320-pixel-wide portrait, common mobile landscape, tablet, laptop, ultrawide, and zoomed viewports. The scene itself never pans, scrolls, zooms, crops birds, or lets a flight end offscreen. Flight arcs stay inside safe margins. Browser zoom remains supported for accessibility; system dialogs and the notebook may scroll their content, and enlarged text must reflow instead of being clipped in an artificially locked page. No drag-to-place handlers, mood labels, inline names, hover tooltips, control icons, or badges sit in the scene. Captions and visible keyboard focus are the explicit accessibility exceptions to the no-copy/no-chrome rule.

### Motion and environment

Render server-selected preening, scanning, head tilts, weight shifts, and flights using continuous eased motion parameters, with slow bounded variations rather than repeating a fixed animation clip. Calls can drive small throat/body/head motion from the same timed descriptor. At every snapshot boundary retain continuity of position and velocity where practical; do not restart idle phase once a minute.

The sky/foliage palette follows account-local sunrise-like morning, midday, evening, and night phases, using a synthetic daily schedule rather than geolocation. No actual weather service or location permission is needed. Start ambient rain at roughly 2–3 brief episodes a week, each around 2–5 minutes, with occasional soft wind; the server owns weather timing and its transient mood effects. Rain is gentle, with no thunder, storm urgency, distress, or “weather alert.”

Leaves and feathers are client-only rendering ornaments, as required. Seed them from the current visual time so a returning frame can already contain a drifting leaf. Use a bounded pool, initially no more than 12 particles at once, and subtle low-amplitude parallax. They never produce events or personality evidence. Avoid full-scene blur/filter animation, excessive layers, or high-frequency shimmering.

### Chrome and motion preferences

The top bar fades after four seconds of pointer stillness only if it contains no focus and no popover/dialog is open. Restore it immediately on pointer movement or keyboard activity. Keep it fully visible while focus is within it, on coarse-pointer/touch-only devices without a cursor, and in an explicit “keep controls visible” accessibility preference. When faded, remove no controls from the accessibility tree or tab order. Do not leave readable text at insufficient contrast: fade the whole inactive icon layer and reveal text labels only in the fully opaque state.

Read `prefers-reduced-motion` before the first scene frame. An explicit settings override takes precedence and can return to following the system preference. In reduced motion, replace each animated action with a sequence of designed still poses cross-fading over roughly 2–4 seconds; cross-fade flights between perch poses; remove leaf/feather drift and parallax; slow palette transitions. Do not briefly play the normal flight before a delayed settings module notices the preference. Preserve all calls/captions, greetings, moods, offers, drift, and notebook observations.

## 12. Procedural audio, listen-in, and call captions

### Grammar and synthesis

Each species supplies motif definitions such as a rise, trill, two-part phrase, low pulse, or airy sustained tone. Each bird gets a stable signature profile at adoption: characteristic frequency band, interval contour, rhythm tendencies, timbre envelope, and motif preferences. These identity-bearing ranges are immutable across naming and ordinary drift. Personality changes calling likelihood, chorus participation, and bounded expressiveness; mood changes phrasing and intensity within recognizable bounds. The seventh-bird design must distinguish two birds of the same species.

The server call runtime expands species/signature grammar into a semantic call descriptor: motif sequence, note count, contour, pauses, duration, intensity class, spatial position, and waveform/noise-envelope parameters. A seed adds bounded variation without changing the identity-bearing contour. The client synthesizer executes those descriptors through WebAudio oscillators, filters, gain envelopes, and procedurally generated reusable noise tables. No audio files contain bird calls, and no stored waveform loop is the call source. Song-fragment offers use a small authored symbolic motif library synthesized through the same infrastructure.

Build one AudioContext per tab, one persistent bus per bird, an optional offer bus, and a master limiter/gain stage. Reuse bounded scratch buffers and persistent routing. Where standard oscillator nodes are one-shot, release/disconnect them on completion and clear application references rather than attempting illegal node reuse. An AudioWorklet is optional only if profiling proves it necessary; any worklet and message buffers are bounded and lazily initialized after the minimal scene. The procedural noise table is generated from a seed, not downloaded as a recording.

Use a scheduler checking roughly every 25 milliseconds with about 100 milliseconds of WebAudio lookahead, driven by AudioContext time reconciled with the server timeline. Cap simultaneous notes initially at 24 and simultaneous independent bird phrases at four; queue or omit only still-future low-priority phrases within the canonical schedule policy when needed. Do not cut a sounding bird mid-call to satisfy the cap. Keep stereo placement subtle and verify the entire chorus in mono. Headroom and gentle limiting prevent clipping when calls overlap; do not rely on phase alignment or stacked recordings for chorus.

### Listen-in mix

The client can begin a local listen-in ramp immediately while its corresponding start event is acknowledged for qualified duration accounting. Ramp over approximately 1.5 seconds: raise the focused bird by about 3 dB relative to its ambient level and lower others by about 9 dB, with the master maintaining headroom. Other birds remain audible. On disengagement, return all buses to ambient over the same duration. A focus transfer retargets ramps from their current gain values so rapidly moving between birds produces no clicks or volume jumps.

End on repeated click of the focused bird, empty-scene click, focus leaving the birds, Escape, settle, hide, or session end. Focusing another bird transfers the mix and ends the prior qualified listen lease. A missing end event times out with its view/presence lease and never grants indefinite drift. Local mix changes are not synchronized between devices or visitors; the canonical calls, mood, and drift are.

### Autoplay and failure

Attempt only browser-permitted startup/resume. Previously authorized audio may play with the first frame, but fresh browser autoplay restrictions make universal immediate sound impossible; do not promise or fake it. Keep the scene alive with captions until a normal user gesture can resume AudioContext. Provide `Enable sound` within the existing accessibility/audio surface in matter-of-fact voice, not a modal blocking the birds. A keyboard gesture can enable audio just as a pointer gesture can.

If WebAudio is absent, blocked after an explicit attempt, interrupted by hardware, or otherwise fails, transition smoothly to silence and enable captions by default. Preserve a user's explicit caption choice while explaining that sound is unavailable in settings. Never fetch recorded audio as recovery. On a hidden tab suspend audio and skip elapsed calls; on a visible return resume only current/upcoming descriptors after authorization/state refresh. Unfocused but visible tabs may keep audio if permitted, while presence remains false.

### Captions from the actual call

Generate prose from the expanded semantic descriptor that the audio scheduler actually uses, after note-count/contour/variation choices: `a soft three-note rise`, `a low trill, a pause, then another`, or `a single sharp call from the back perch`. Do not use one fixed caption per bird, motif ID, or mood. When sound is unavailable or muted, the same scheduled descriptor drives a caption of the call being represented. A canceled call produces no misleading caption; a delayed future call's caption follows its final scheduled time.

Place captions near the calling bird with collision-aware offsets and a quiet contrasting backing. Retain readability for the phrase length, normally at least 2.5 seconds, while avoiding a queue of captions detached from current calls. Merge simultaneous phrases into a short spatially accurate group description only if individual captions would collide; retain specific attribution in the accessible description. Standard mode uses gentle opacity transitions; reduced motion uses the slower designed fades. Caption nodes are not all separate live regions, so screen readers are not forced to hear every call on top of the running narration.

## 13. Accessibility as a complete presentation

Use semantic DOM controls for all top-bar actions and system surfaces. Treat the visual SVG as presentation and maintain a corresponding accessible scene region with an intelligible description and owner bird controls. Do not ask assistive technology to interpret thousands of SVG feather paths. Bird control names can say `listen in on pip, the warbler on the front perch`, without numeric trait or mood-state strings. Visitors receive descriptive bird content without actionable listen-in controls.

Keyboard order starts with the four top-bar items and then one entry into the owner bird group. Use roving tabindex so arrow keys move among birds in stable spatial order and DOM focus is preserved through pose changes. Focus begins listen-in; Enter toggles; Escape ends it. Returning to the top bar ends listen-in. Add a documented configurable shortcut, initially Alt+O, to open the offer popover; the top-bar button remains fully reachable if a browser or assistive technology reserves the shortcut. Offer choices and settle work with normal button keys. Dialogs have labels, sensible initial focus, Escape dismissal, and focus restoration to the invoking control. No hover-only interactions or focus traps.

Narration is an authored prose composition from the same semantic scene state, delivered into one polite, atomic live region at approximately 45-second idle intervals. Describe a coherent moment with bird-specific activity, relative perches, and light/weather where meaningful. Avoid `mood: content`, event codes, raw state lists, and system session language. Keep a bounded latest-observation queue, suppress near-duplicates, and never replay narration accumulated in a hidden tab.

User-initiated greeting/offer/settle observations receive a small priority bump and can appear promptly; coalesce bursts and do not interrupt a system dialog's text. Use polite announcements rather than assertive interruptions for the aviary. Provide pause narration, resume, and read-current-scene controls in accessibility settings plus an optionally visible transcript region. Pause affects speech delivery, not simulation, access to the current description, or notebook generation. Screen-reader users should be able to hear a quiet observation, navigate an offer, hear its specific response, and continue listening without fighting a queue.

Define actual contrast tokens during the first slice because the referenced designer document is not provided. Require at least 4.5:1 for ordinary text, 3:1 for large text, and 3:1 for focus/control indicators against adjacent colors. Use higher-contrast backing where day/night or plumage would vary the caption background. The night scene cannot make labels or focus disappear. Allow high-contrast/forced-color presentation and 200% text scaling; target 44-by-44 CSS-pixel touch areas where feasible without displaying extra scene chrome. Never disable browser zoom.

Accessibility QA includes VoiceOver with Safari on macOS/iOS, NVDA with Firefox/Chrome on Windows, keyboard-only navigation, touchscreen with captions, reduced motion enabled before load, mid-session preference changes, muted audio, unsupported WebAudio, forced colors, and narrow/zoomed viewports. Combine automated semantic/contrast checks with human evaluation of narration and reduced-motion charm; automated ARIA checks cannot prove the intended experience.

## 14. Performance budgets and operational instrumentation

### Budgets and measurement definitions

| Requirement | Implementation budget and release measurement |
| --- | --- |
| Initial JavaScript less than 2 MB gzip | Enforce a hard build failure at 2 MB for all JS requested before the first bird is painted, including third-party transitive code. Aim below 200 KB, with a minimal first-frame bootstrap below 25 KB. Lazy-load account, full accessibility editor, notebook, invitations, and optional advanced audio code. Do not lazy-load the initial reduced-motion preference or semantic scene. |
| First bird visible in less than 500 ms | Measure navigation start to the first actually painted bird, not React mount, skeleton paint, request finish, or a hidden SVG insertion. Budget about 180 ms to first private HTML byte, 80 ms critical transfer, 100 ms parse/style/first bird paint, and 100 ms margin. Keep critical HTML/inline state/visual assets near 45 KB gzip. Initial JS has a separate hard ceiling; being under 2 MB alone cannot achieve this time target. |
| Responsive greeting | Under healthy conditions, first noticing begins within 1–2 seconds after visible activation. Measure it independently from first-bird paint so a fast static-looking scene cannot pass. |
| Idle at 60 fps | Target a 16.7 ms frame on a five-year-old mid-range laptop, including a 30-minute seven-bird chorus/weather session. Keep scene JS below about 4 ms and rendering/layout below about 8 ms in ordinary frames, leaving browser overhead. Record frame percentiles and missed-frame proportion; persistent dropped frames fail, even if average fps appears acceptable. |
| No memory growth over 30 minutes | After a 5-minute warmup, run 30 minutes with representative calls, listen-in, offers, notebook paging, focus changes, and hides/resumes. Require no retained-heap growth trend after equivalent GC sampling, no monotonically growing DOM/listener/node counts, and bounded buffers/contexts. Repeatable growth is a release blocker; allow only measured GC/instrumentation noise, not a “small leak” budget. |
| Small state and bounded compute | Seven-bird snapshots below 16 KB gzip; event batches at most 16 KB; finite schedule horizon, caption/narration queues, particle pools, and notebook DOM windows. Test worst-case overlapping responses instead of only two silent birds. |
| Tick operational alarm | Alarm when p99 simulation tick latency exceeds 5 seconds. Measure compute-plus-commit separately from scheduled-due-to-start lag so a fast worker with a stuck queue is visible. Healthy compute target is below 100 ms for seven birds, to be established by load tests. |

Create a reproducible mobile performance profile: a named mid-tier Android handset, supported Chrome, a fresh authenticated navigation without warmed asset cache, and controlled 4G-like bandwidth/latency. Start the laboratory profile at 5 Mbps downstream and 80 ms round-trip network latency, and report the actual device, throttling, geographies, cache state, and percentiles with every result. Use physical devices in addition to CPU-throttled desktop browsers. Run at least 20 navigations for each release candidate; target p95 first-bird paint below 500 ms on that profile and preserve the raw synthetic distribution. Separately measure warm returns, initial adoption, slow networks, and unsupported-browser flows rather than blending away a cold-start regression.

Authenticated first-frame delivery is a real architectural risk: a far-away database, cold runtime, or extra auth round-trip can spend the entire budget. Keep first-read auth and snapshot projection in one request, avoid a waterfall for species assets, precompute compact presentation snapshots at commit time, keep rendering nodes small, and deploy each account's authoritative region near its supported audience. Serve private dynamic HTML through the edge without publicly caching it. Do not claim that a globally replicated eventually stale vector cache solves this problem. If the initial slice misses the budget, correct the critical path or narrow the launch geographies before adding features; record unavoidable network outliers honestly.

For first-bird instrumentation, bracket the committed SVG visibility with animation-frame paint confirmation and verify the mark against browser trace/video in synthetic runs. For an HTML-streamed bird, do not wait for the full application bundle to mark a paint that happened earlier. Cross-check that the bird is in the viewport and nontransparent. This metric must not be satisfied by a 1-pixel placeholder.

### Operational health without behavioral analytics

Collect only route-class request counts, status/error counts, page-load timings, first-bird timing histograms, frame-duration histograms, audio-context failure categories, anonymized session-duration histograms, snapshot payload sizes/latencies, worker tick/queue latencies, job backlogs, and database health. Coarse browser major, platform class, application version, and synthetic region are sufficient dimensions. Production metrics must not use account UUID, bird count, bird UUID, species, mood, trait, greeting selection, offer acceptance, presence duration, invite recipient, call descriptor, or notebook content as labels.

Aggregate session duration means a coarse duration of the application session for operational health, without an account dimension; it is not a proxy export of qualified presence. Discard fine-grained samples after aggregation. Strip IPs, credentials, route IDs, referrers containing tokens, and request bodies from metrics ingestion. No third-party session replay, heatmaps, advertising SDK, raw event analytics, voice recording, or behavioral warehouse connector exists. Metrics collector credentials cannot read the simulation store.

Bounded support/error logs may associate a synthetic account UUID with an authentication/loading failure, as permitted by the PRD, but never serialize bird state or interaction content. Restrict access, retain at most seven days, and include them in deletion. General traces use route templates and opaque request IDs. Scrub exception objects before serialization; a failed request object must not leak a token, email, vector, or offer payload through a stack trace or error SDK.

Instrument these boundaries from the first vertical slice. Add schema tests that deliberately attempt forbidden telemetry fields and verify rejection before storage/export. Tick and audio errors should include a stable error category and build version, enabling diagnosis without the relationship data. Synthetic fixtures may expose vector charts within engineering tests because they are generated test birds; do not repurpose those tools as a real-user debug mode.

### Capacity and recovery

Scheduled load is proportional to all nondeleted aviaries, not currently connected browsers. For N aviaries, provision for roughly N/60 regular ticks per second plus bounded interactive wakeups. At 100,000 aviaries that is about 1,667 regular ticks/second; this is a sizing example, not a launch forecast. Benchmark the seven-bird reducer and database writes before setting a cohort limit. Shard the due-work queue by UUID, spread due times, batch claims, and bound worker concurrency to protect the database.

Use point-in-time backups, encrypted durable checkpoints, and rehearsed restoration of canonical vectors and their event cursor together. A snapshot restored without its matching cursor risks replaying evidence; a cursor restored ahead of its vector risks losing evidence. Test both failure modes. Recovery must preserve IDs/signatures and avoid newly seeding any existing bird. If state integrity cannot be established, show a system outage surface and restore correctly rather than silently replacing a bird.

Separate alarms for stale canonical revisions, tick due lag, p99 tick time over five seconds, auth mail failure, export/deletion jobs past deadline, growing database queues, audio-context error spikes, first-bird regressions, and memory regressions in synthetic runs. No alarm is based on whether users clicked enough, a bird's personality, or aggregate drift.

## 15. Validation strategy and acceptance matrix

All tests below are work for the implementation team. No test execution or product behavior is claimed by this planning deliverable.

### Engine, identity, and time

- Property-test nonnegative bounded trait updates under empty input, positive input, random event sequences, month-long absence, and variable tick spacing. Splitting one elapsed interval into many early ticks must produce equivalent filter integration and final traits within numerical tolerance.
- Advance synthetic regular observers for 1, 7, 21, 90, and 365 days. Check calibration bands, long-run headroom, individual signature/behavior differences, and no single-session visible jump. Compare quiet qualified presence with repeated button spam: presence remains dominant and cooldowns/secondary caps hold.
- Close every client for two weeks while running the server scheduler. Verify mood/day/night/weather continue, no presence appears, vectors never decrease, stored IDs/signatures persist, and opening does not reset mood. Inspect absence-tail drift as evidence from before departure, not fabricated attendance.
- Exercise daily-ish mood evolution across DST changes, timezone changes, midnight, device clock jumps, delayed ticks, and long outages. Wary cues may arise from allowed mood/ambient causes, never merely from absent-owner time.
- Rename, restore backups, upgrade engine/grammar versions, and adopt concurrently. Existing identity and traits must be bit-preserved unless a legitimate tick delta is applied; only the new bird initializes. Concurrent seventh/eighth adoption permits one valid seventh bird and no eighth.
- Test bird-to-bird response delays, capped alarm propagation, nocturnal call behavior, nonidentical greeting parameters, one first greeter, and later staggered responses. Synthetic sessions spanning different absence lengths must produce meaningfully different greeting distributions rather than three canned clips.

### Presence and multi-device correctness

- Exhaustively test the eight combinations of visible, focused, and recently active. Only the all-true combination accrues presence; settle/ended/visitor states additionally disable it.
- Test four-minute boundary expiration, trusted versus synthetic input, visible-but-unfocused windows, focused-looking minimized windows, hidden tabs receiving key events elsewhere, sleeping laptops, touchscreen movement, keyboard navigation, and no pointer coordinates or typed keys in persisted data.
- Open laptop and phone concurrently. Union overlapping presence intervals; do not double-count time. Union same-bird listen intervals, cap different-bird secondary evidence, and recover correctly if only one view ends or settles.
- Replay duplicate events, lose the POST response after server commit, reorder fetch completions, expire a session mid-write, crash a worker between each transaction step, and restart multiple schedulers. Each accepted event is applied once, no higher cursor skips an uncommitted lower event, and no stale client snapshot overwrites a newer revision.
- Reject absolute personality writes, unknown event types, wrong-account bird IDs, excessive claimed durations, expired sequences, and all visitor submissions to owner endpoints.

### Experience, audio, and notebook

- Review initial normal-return video frame by frame: birds start midway through an ongoing action, motion does not reset during hydration, and greeting starts in 1–2 seconds. No welcome text, static-to-scene fade, loading spinner, or absence count exists. Verify the unique first-adoption exception separately.
- Test all offer types against all moods, full cooldown coverage across devices/types, no-reaction cases, offer object cleanup, and no immediate trait-stat disclosure. Settle/undo at 0, 4.9, 5.1 seconds and after a delayed reply must preserve the defined gesture semantics without rewinding unrelated events.
- Run deterministic grammar tests for signature bounds and variation, then offline spectral/envelope checks for clipping, discontinuities, repeated waveform loops, and stereo/mono chorus behavior. Test seven birds including duplicate species. Subjective listening with synthetic birds over repeated sessions evaluates recognizability and uncanniness; waveform tests alone cannot establish charm.
- Check caption note counts, pauses, contours, and attribution against the final expanded/scheduled descriptors. Compare muted, unavailable-audio, and audible presentations. Listen-in ramps are gradual, nonfocused birds remain audible, and rapid focus changes produce no clicks.
- Generate notebook histories for sparse, regular, very active, and absent synthetic observers. Check cadence limits, truth of comparisons, named specificity, present-tense/lowercase grammar, indefinite pagination, preserved historical names, and absence of attendance or achievement language. No API permits annotations or per-entry deletion.

### Access, privacy, and lifecycle

- Run keyboard/assistive-technology scenarios described in section 13 with all main interactions, overlays, naming, invitation, export, error, deletion, and recovery surfaces. Test focus outline contrast in every lighting phase and narration queue length during rapid actions.
- Verify reduced motion is active on the first frame, replaces flights/micro-motion with cross-fades, removes ornaments, and retains calls/captions/semantic observations. Do not accept a static screenshot as evidence of a complete reduced-motion experience.
- Test a forwarded/consumed link, two simultaneous invitation redemptions, unused expiry at 30 days, revoked active/hidden/offline visits, read-only rendering, and recipient identity privacy. A visitor sitting for an hour cannot alter any host presence/drift state. Default visits create no emails except the host-requested invitation and no UI notice.
- Replay magic links, test the 15-minute boundary, email-change races, revoked devices, CSRF attempts, enumeration resistance, token leakage through referrers/logs, and cross-account object access. Mail scanner GETs do not consume capabilities.
- Export revision-consistent names/moods/notebook and sealed vectors, check that plaintext outputs/DOM/ARIA contain no trait numbers, expire the download, and delete during export generation. Verify the archival limitation is accurately described.
- Test deletion immediately, recovery just before deadline, recovery racing hard deletion, and after-deadline behavior. Inventory and erase every account-linked store, including logs, contact references, objects, jobs, and backups through key erasure. A restore drill must not reanimate a hard-deleted account.
- Scan telemetry schemas and captured synthetic network traffic for forbidden emails, bird fields, raw events, presence samples, notebook prose, and tokens. Verify analytics credentials cannot query protected storage. Do not run this check by copying real-user histories into a testing warehouse.

### Performance and release evidence

Run the explicit bundle, mobile first-bird, five-year-laptop 60fps, and 30-minute retained-memory checks with two and seven birds, all supported rendering/audio modes, and late notebook pages. Test the last two major Chrome, Safari, Firefox, and Edge versions at release; unsupported browsers receive a concise matter-of-fact explanation before loading an incompatible scene runtime.

The release evidence package contains synthetic workload definitions, device/browser/network profiles, distributions and traces for performance, property/concurrency/security test results, accessible interaction recordings/reviews, listening-review notes, migration/restore checks, and privacy-schema assertions. It must not contain real account interaction histories or a dashboard of production traits. A check marked “planned” is not evidence of passing it.

## 16. Implementation sequence and rollout gates

Organize implementation around completed slices with explicit dependencies. The engineering team may divide ownership by module, but no additional research repository or hidden product specification is a prerequisite.

| Stage | Work and ownership | Exit gate |
| --- | --- | --- |
| 0. Contracts and fixture foundation | Engine/API owners define immutable IDs, events, revisions, presence intervals, vector storage, descriptor schema, canonical timezone, and role separation. Design/accessibility/audio owners author six-species direction and first two species' pose/motif/narration examples. | Written contracts and synthetic fixtures cover the interpretation decisions, including export/notifications. No vector serialization path or visitor event permission exists. |
| 1. Two-bird living slice | Deliver authenticated private HTML, server minute tick while disconnected, two persisted birds, one continuous scene, ongoing call grammar, return-greeting, visible-return fetch, and exact presence. Include captions, narration, and reduced motion now. | First-frame and greeting budgets, absence continuity, one complete keyboard/screen-reader flow, two-device snapshot coherence, and basic memory bounds pass. Resolve critical-path latency before expanding scope. |
| 2. Relationship and gestures | Add elapsed-time drift/filter integration, all moods, bird-to-bird responses, seed/song-fragment/pool offers, listen-in ramps, settle/undo, rename, environment, and sparse notebook. | Synthetic one/three-week calibration and concurrency properties pass; a multi-session listening/visual review finds recognizable individuals. No user-visible numbers, attendance cues, or obligation mechanics. |
| 3. Full account/privacy lifecycle | Complete device revocation, address verification, export/sealing, deletion/recovery, retention jobs, key erasure, backup restore, and matter-of-fact error surfaces. | End-to-end deletion/restore and account-access tests pass; observability schema is proven separate from simulation data. |
| 4. Quiet visits and full species pool | Add named one-use invitations, read-only renderer projection, logs, revocation/freshness leases, optional off-by-default notices, and remaining species. | Visitor cannot influence host state; revocation works visible/hidden/offline; silence-by-default and no co-presence are verified. |
| 5. Age-based growth and sustained quality | Enable persisted adoption offers, test two/three/five/seven birds with synthetic aged aviaries, finish mobile collision/caption layout and seven-bird chorus. | Seven-bird cap, signature differentiation, 60fps, no 30-minute memory growth, and full accessibility/browser matrix pass. |
| 6. Production ramp | Launch complete v1 to a limited account cohort, then increase operational capacity and bird-availability flags as described below. | Health and qualitative review gates remain satisfied; rollback procedures preserve state. No required accessibility/lifecycle feature is deferred to a later launch patch. |

Use internal synthetic aviaries first, including virtual 18-month-old accounts, to validate all adoption stages without accelerating real owners' ages. A small dogfood group can evaluate actual listening/visual continuity, but automatic drift calibration must use synthetic workloads; do not aggregate their private events into population tuning datasets. Voluntary qualitative feedback is sufficient to refine authored motion/call parameters without collecting account histories.

Production rollout begins with two starters and all v1 interactions, accessibility, accounts, notebook, and opt-in visit functionality. Apply an operational growth ceiling of two initially, then three, then seven after the corresponding full-quality tests pass. Raising the ceiling only permits already age-eligible adoption; it never grants an attention reward, forces adoption, or creates a progress display. Complete seven-bird readiness before real owners encounter an eligible offer that cannot be fulfilled. If a later incident lowers admission limits, preserve all already adopted birds and their IDs; stop new adoption availability instead of hiding or removing birds.

Ramp account admission conservatively, for example internal -> invited pilot -> limited public cohort -> broad release, using tick capacity, first-bird latency, audio error rates, and synthetic performance/accessibility regression results. Do not gate expansion on visit frequency, offer usage, average traits, “engagement,” or number of social invites. Gather explicit qualitative review of whether the place feels continuous and whether bird signatures remain recognizable; keep feedback separate from private simulation records.

Roll back client code and authored presentation versions using backward-compatible snapshot contracts. Version engine constants and migrations; never restore an older complete vector snapshot over later legitimate drift as a software rollback. A severe drift-calibration issue can set future drift rates to zero temporarily while preserving vectors and continuing safe mood/environment ticks. A defective call motif can be disabled in favor of other procedural motifs from the same signature family; it cannot fall back to a recorded loop. Keep narration/captions available during audio mitigation. Any integrity incident fails safely with system clarity rather than reseeding birds.

## 17. Risk register and concrete responses

| Risk | Early signal / test | Response and owner |
| --- | --- | --- |
| Drift is too fast, too slow, or converges all birds | Synthetic 7/21/365-day experiments and longitudinal review with fixed bird identities | Engine owner adjusts versioned future coefficients/filter caps, preserving accumulated vectors. Presence must remain the dominant input; never tune from aggregate production relationships. |
| Presence quietly counts unattended tabs | Full predicate matrix, overnight background tests, unioned multi-device interval tests | Client/API owners make transitions close intervals precisely and server clamps retrospective evidence. Do not substitute polling or click counts. |
| Duplicate processing loses or doubles drift | Crash injection, out-of-order commits, replay/retry, scheduler fencing tests | Storage/engine owner keeps cursor and vector commit atomic, serializes sequence assignment, and restores coherent checkpoints. |
| Canonical data is lost or reset in migration | Bird identity checks, PITR drills, canary migration assertions | Data owner blocks migration/release and restores the existing record; no regeneration fallback. |
| “Alive” becomes canned | Repeated greeting/idle/call comparison and listening review over days | Motion/audio owners expand continuous parameterization and contextual responses while preserving per-bird signature constraints. |
| Audio is uncanny, harsh, or indistinguishable at seven birds | Mono/stereo seven-bird reviews, envelope/clipping checks, repeat-signature identification | Audio owner tunes motifs, headroom, density, and immutable identity bands; hold adoption ramp if recognizability fails. |
| Fresh browser blocks sound | Real Safari/mobile/permission tests, aggregate audio-context error categories | Client owner retains motion and captions, resumes on allowed gesture, and provides clear settings controls. No recorded fallback or forced modal. |
| Fast first paint is only a cached desktop success | Cold authenticated physical-device runs across launch geographies | Web/platform owners reduce critical bytes/round-trips, inline actual current scene, and keep private cache isolation. Do not claim compliance based on warm-only results. |
| Memory grows from call nodes, timeline segments, or notebook DOM | Automated 30-minute retained-memory/DOM/resource tests | Client/audio owners bound lifetimes and explicitly release finished nodes/handlers; block rollout on a repeatable trend. |
| Accessible rendition loses calmness or functionality | Human narration/reduced-motion sessions plus keyboard/contrast checks | Accessibility/design owner revises prose cadence, pose transitions, focus behavior, and copy backing as first-class design work. |
| Invitations leak or continue after revocation | Capability replay and hidden/offline revocation tests | Auth owner revalidates every pull, separates roles, expires authorization freshness, and scrubs tokens from URLs/logs. |
| Analytics becomes a relationship dataset | Schema injection tests, credentials audit, captured synthetic payload inspection | Platform/privacy owner prohibits bird/event fields at ingestion and denies warehouse access to simulation storage. |
| Ambiguous export expectation | Review of readable JSON plus sealed personality blocks | Product/API owner documents the limitation explicitly. Do not quietly expose numerical traits or falsely promise independently readable state. |
| Optional notifications expand into an engagement loop | Default-account and consent-toggle end-to-end tests; copy review | Product/account owner confines the exception to explicitly requested visit email notices and keeps all other social/chrome reminders absent. |
| Background simulation cost overwhelms capacity | Synthetic all-account load, due-lag and transaction latency alarms | Platform owner reduces redundant scheduling overhead, scales UUID partitions, and limits new account admission without making ticking contingent on viewers. |

## 18. Definition of v1 completion

V1 is ready when a newly signed-in owner receives two stable named birds; the first ordinary return paints an already-moving bird within the stated supported-device budget and one bird notices within 1–2 seconds; valid presence shapes a persisted, monotonic vector on the intended timescale; and closing the browser for weeks leaves a continuing, nonpunitive aviary.

The same canonical birds and moods must appear on another device, with event retries and simultaneous interactions unable to lose or duplicate drift. The owner can use all three offers, listen-in, settle/undo, naming, sparse indefinite notebook history, settings, export, deletion/recovery, and quiet invitation/revocation. The seven-bird configuration must be fully tested even while age keeps most real accounts at two. Visitors can observe the real current scene without contributing presence or causing any host greeting or interaction.

Screen-reader narration, captions, reduced-motion cross-fades, keyboard focus, contrast, and silent WebAudio fallback are complete at launch. Bundle, first-bird, frame-time, retained-memory, tick-latency, privacy, and restoration evidence meet the gates above. Product copy remains specific, lowercase, and observational; sign-in, errors, account, sync, and accessibility settings remain matter-of-fact. There are no numbers to optimize, chores to complete, or reminders to appease.

Planning boundary: only this plan and the assigned runtime metadata are deliverables for this run. Product implementation, evaluation, and wave timing are outside this planner's execution scope.
