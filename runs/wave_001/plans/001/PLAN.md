# Pocket Aviary — v1 implementation plan

This plan is based on `prd/1-START_HERE.md` and all nine PRDs it lists. It specifies implementation work; it does not implement the product. Numerical tuning values below are initial engineering decisions, with explicit acceptance gates where calibration is necessary.

## 1. Product boundary and decisions

Ship a browser aviary that appears to have been continuing before it was opened. Two individually recognizable birds arrive at account creation; their persistent identities and slowly changing personalities underpin the relationship. The visible experience is one quiet scene, procedural calls, occasional noticing, and a sparse field notebook. Attention matters without becoming a duty or a score.

V1 includes:

- Email magic-link accounts, one canonical aviary per owner, device session management, verified email changes, export, and recoverable deletion.
- Six coherent species, two server-selected starter birds, naming and renaming, age-based opportunities to adopt additional birds, and a hard seven-bird limit.
- Server-owned personality drift, persistent mood, bird-to-bird responses, time-of-day behavior, and rare ambient weather whether or not a browser is connected.
- Return-greetings, listen-in, the three offers, settle with five-second undo, qualified presence accounting, and an indefinitely readable, read-only notebook.
- Optional, individually addressed, one-time visit invitations; a visitor sees the actual ambient aviary and has no simulation input.
- Full-quality accessible narration, call captions, keyboard control, designed reduced-motion presentation, and graceful silence when WebAudio cannot run.
- Performance instrumentation and aggregate operational telemetry with a structural separation from private simulation data.

Exclude native applications, payments, SSO, passwords, shared ownership, multiple aviaries per account, scene customization, bird placement controls, species catalogs or rarity, public discovery, profiles, follows, comments, chat, co-presence, recorded call audio, and every form of gamification. There are no hunger or happiness meters, distress or death from absence, visit streaks, adoption counters, reminders to return, or rewards for frequency of use. Do not build latent leaderboard or engagement datasets. Additional birds are not earned through attention.

### 1.1 Explicit interpretations of conflicting or incomplete requirements

These decisions are part of the implementation contract, rather than unresolved questions for the next team.

| Issue | V1 decision |
| --- | --- |
| Personality values are never shown, but account export explicitly includes current vectors. | Ordinary product surfaces, DOM/ARIA, application snapshots, diagnostics available to users, and visitor responses never expose trait numbers. Honor `accounts_sync.md`'s explicit export contents as a narrow data-portability exception: a deliberately requested, privately delivered JSON file includes current vectors. This means a user can inspect numbers in that file; the literal absolute prohibition and the explicit export requirement cannot both hold. Do not add an in-product export preview or a statistics surface. |
| The brief rejects notifications, while the social PRD explicitly offers an off-by-default visit-notification toggle. | Ship that named exception in account settings only: optional transactional email when an invited visit begins. Default off, no onboarding prompt, no push implementation, badges, banners, or aviary toasts. Magic links, invitations, export delivery, and email verification are also purpose-specific transactional email, never marketing or return reminders. |
| The layout lists exactly four top-bar icons, while interactions require top-bar settle. | Keep four groups: account/settings, accessibility, notebook, and offer. The offer affordance opens an accessible small gesture menu containing the three offers and a separated `settle` item. Its accessible name is “offer or settle.” Thus settle is reachable from the top bar without adding a fifth icon. |
| The scene bans labels/chrome, while captions and focus indicators are required. | Call captions and keyboard focus indicators are explicit accessibility exceptions. No permanent bird labels, mood icons, hover tooltips, or offer buttons appear inside the scene. |
| Minute ticks versus a greeting within one or two seconds and a prompt offer response. | Use a server-only reducer normally scheduled every 60 seconds, also eligible for an expedited pass after an owner interaction. Both paths use the same transaction, event cursor, and elapsed-time mathematics. An expedited pass cannot accelerate drift. The next pass can occur promptly rather than making a bird wait a minute to react. |
| The first frame should already have audible calls, but browser autoplay can be blocked. | Render current action and call timing immediately. Start procedural sound immediately only when browser permission and saved audio preferences permit. Otherwise run the same call timeline silently with captions and a matter-of-fact audio control in settings; retry audio on an authorized user gesture. Never fake autoplay success or play a backlog of missed calls. |
| User-local time versus a single canonical aviary on devices in different timezones. | Store one IANA aviary timezone on the account, initialized from the first signed-in browser. All devices and visitors use that timezone. Offer an explicit timezone change in settings; do not let two traveling devices continually change the canonical day. Transition lighting gently after a change. |
| Listen-in starts on keyboard focus, and the keyboard spec also assigns Enter to listen-in. | Entering/focusing a bird begins listen-in; Enter ensures it is engaged. Escape disengages while leaving focus on that bird, and it stays disengaged until Enter or a subsequent focus change. Arrow keys transfer both focus and listen-in. No double firing from focus followed by activation. |
| A visit is one-time, but active-session lifetime is not specified. | An unused invite expires after 30 days. Atomic redemption exchanges its token for a revocable, render-only browser session lasting at most two hours, also ending after five minutes without a valid snapshot pull. Refreshing that browser may continue the same session; the original invitation cannot start another. A later visit requires a new invitation. |
| The revocation prose mentions a visitor disappearing from the log, while the log is a transparency record. | Remove revoked access from outstanding/active invitations and end the active visit. Preserve the historical visit row, marked ended/revoked, so the host can still see what was shared. No success toast or host notification. |
| Presence uses pointermove or keypress, including on touch and accessible devices. | Start with a five-minute recent-activity window. Use trusted Pointer Events movement and trusted physical keyboard input; implement keyboard detection using keydown where needed to include navigation keys. Do not count mere visibility, focus, a timer, synthetic events, or a click without the specified activity. A tap-only touch session can greet and offer without automatically becoming qualified idle presence. Test this limitation explicitly; do not silently broaden the definition. |
| A separate visual design-system document is referenced but not supplied. | Define and approve the concrete design tokens, six silhouettes, contrast combinations, and motion studies as a first delivery task using the supplied palette and accessibility rules. This plan does not assume unseen design specifications. |

## 2. Architecture and authority boundaries

Use a compact TypeScript web application with server-rendered HTML and a lightweight hydrated control shell, a dedicated Canvas 2D scene renderer, and native WebAudio synthesis. Use a TypeScript HTTP service and simulation worker sharing a versioned pure reducer package. PostgreSQL holds authoritative state, append-only events, auth/session records, and a transactional work outbox. A scheduler dispatches due aviaries to workers. A CDN serves public immutable assets; an authenticated edge gateway delivers owner-specific bootstrap HTML and its compact state snapshot. This is a modular service with two process roles initially, not a collection of independent microservices.

Suggested module responsibilities:

| Module | Responsibility and boundary |
| --- | --- |
| Domain contracts | Validated request schemas, public render projections, private canonical types, protocol versions, error codes. Never serialize a private bird record wholesale. |
| Auth/account service | Identity, magic links, device sessions, verified address changes, export/delete workflows, settings and consent. |
| Interaction ingress | Authenticate owner, validate event shape, assign sequence, enforce idempotency, append commands, request an expedited reducer pass. No personality-write privilege. |
| Simulation worker | Sole writer of personality, mood, drift-filter state, canonical bird actions and call schedules; owns greeting/offer decisions, weather, and notebook candidates. |
| Snapshot projector | Convert canonical state into render commands without vectors, history, email, or private aggregates; produce owner and narrower visitor projections. |
| Visit service | Invitation delivery, token redemption, revocation, visit sessions and host-only visit log. No route to owner interaction ingress. |
| Web scene runtime | Sample server action curves and synthesize scheduled calls; interpolate movement, manage captions and narration, gather qualifying input intervals. Never advance personality or choose a new mood. |
| Operational collector | Receive only allowlisted performance counters/histograms. Cannot query the simulation database or ingest event payloads. |

The ownership path is:

1. A browser sends a semantic owner event, such as offering a seed, with a unique event ID.
2. Ingress appends it once, gives it an aviary-local sequence, and wakes the worker.
3. The worker locks the aviary, integrates elapsed time, consumes events in sequence, makes behavioral decisions, and commits a new revision with a public render projection.
4. The initiating request may wait up to 250 ms for that committed revision; otherwise it returns accepted/pending and the client briefly polls for its result.
5. Browsers pull snapshots, accept increasing revisions, and render the same canonical state. Only local presentation differs: listen-in gain, captions, reduced motion, viewport, and accessibility preferences.

A click can change local audio gain immediately, since that is presentation. An offer cannot optimistically change personality, mood, or acceptance; it waits for a server-authored reaction. The offer menu can close promptly and show the placed item only when its accepted command arrives. No simulated success is shown for a failed write. The entire first greeting is a bird action, never a welcome toast.

### 2.1 Initial render and edge delivery

For a signed-in navigation, the gateway validates the session and account status before obtaining the current snapshot from a private revision cache or the origin. Embed the minimal JSON render projection and essential SVG bird poses in the HTML. Use a tiny inline bootstrap to draw the scene before loading the control framework. Include server time, action start/end times, and action phase so hydration continues a mid-action pose without restarting it.

Owner HTML and snapshots are private, never placed in a public CDN cache. A private edge store may cache a projection by opaque aviary UUID and revision, behind an authorization check on every request, with a maximum 60-second age and explicit revision invalidation. Auth, deletion status, and visit revocation checks must be current even on cache hits. Static assets alone have public immutable cache headers. Do not put snapshot content, invite tokens, or emails in cache keys visible to third-party observability.

Keep the resident snapshot small enough to be delivered with the HTML. The gateway may also append a `return` event and include the resulting greeting descriptor when it finishes promptly; the bootstrap must not wait for that descriptor to draw existing motion. A fast post-bootstrap event response can supply the greeting within the first two seconds.

On a slow origin/cold path, render a quiet field of sky and restrained ambient color, without a spinner, progress percentage, or fade-in reveal of the whole aviary. Once data arrives, start at its current phase. The one exception is the true first adoption: a quiet empty field followed by a soft fly-in, or a pose cross-fade in reduced motion. Never reuse that entry sequence on return.

## 3. Persistent data model

Use UUID primary identifiers, UTC timestamps for storage, an IANA zone for the aviary clock, and monotonically increasing 64-bit aviary revisions and event sequences. Durable private payloads use account-scoped encryption. Apply ownership checks in the service and database row-level policies. Use separate database roles: ingress can append events, the worker can update simulation state, the projector can read only its required views, and telemetry has no simulation role.

| Record | Essential fields and constraints |
| --- | --- |
| `accounts` | `account_id`, encrypted current email, auth-only keyed lookup digest, verification time, IANA timezone, preferences/version, created time, deletion status/deadline. Email is never the account ID or an inter-service key. |
| `aviaries` | `aviary_id`, unique `owner_account_id`, `created_at`, `revision`, `last_integrated_at`, `last_event_seq`, `next_due_at`, simulation version, PRNG state, canonical day/weather state, active greeting episode, public projection version. One per active owner. |
| `birds` | Stable `bird_id`, `aviary_id`, immutable adoption time and species revision, editable name/version, persistent identity seed and voice signature, hidden trait vector, per-trait limits, persisted drift-filter values, mood/timer, perch/action state, offer eligibility time. No path renames or replaces the ID when the name changes. |
| `species_revisions` | Six launch species' silhouettes, palettes, motif/grammar versions, identity-safe pitch ranges, mood-response coefficients, nocturnal flag, pose definitions. Referenced revisions remain available while a living bird uses them. |
| `interaction_events` | `(aviary_id, seq)`, globally unique event UUID, authenticated device/session UUID, accepted time, bounded client monotonic timing metadata, type and strictly validated payload, result reference. Append-only; no client state blobs or trait values. Unique `(account_id, event_id)` handles retries across devices. |
| `presence_windows` | Window UUID, owner/device reference, monotonic anchor, last accepted segment, terminal reason, eligibility state. Contains no keys pressed, cursor positions, screenshots, or idle behavior traces. |
| `presence_coverage` | Short-lived per-aviary time coverage and credited frontier, with interval union information and daily normalization totals. Needed to count overlapping owner devices once. Not a user-facing visit calendar. |
| `attention_windows` | Bird UUID, start/end, associated presence window, heartbeat lease, credited duration. Multiple devices cannot multiply an interval's total listen-in credit. |
| `action_schedule` | Stable action/call IDs, bird references, UTC windows, motif seed, gesture curves, position endpoints, cancellation/replacement IDs. Persist only a bounded present/future horizon and compact observation evidence. |
| `notebook_entries` | Entry UUID, aviary UUID, timestamp/local date, finalized naturalist text, private evidence type/reference, template version. Immutable product entries; cursor pagination keeps all history accessible. No edit/delete/annotate endpoint. |
| `observation_memory` | Bounded private facts needed for truthful sparse observations: recent first-greeter records, current quiet stretches, rare behavior candidates, duplicate suppression. This is not population analytics or a rebuilt personality history. |
| `device_sessions` | Synthetic session UUID, account UUID, hashed opaque token, coarse user-editable device label, issued/last-used/expiry/revoked times. No raw token storage. |
| `auth_challenges` | Opaque challenge UUID, token hash, purpose, account/delivery reference, created/15-minute expiry/consumed times. Consume once transactionally. Pending email ciphertext is temporary verification material, removed after success/expiry. |
| `invitations` | Invite UUID, host/aviary UUID, recipient reference, one-time token hash, created/30-day unused expiry/redeemed/revoked times, consent recorded by explicit host action. |
| `visit_sessions` | Visit UUID, invitation UUID, hashed browser token, start/last-pull/end/expiry, termination reason. These are read-only credentials, never owner sessions. |
| `visit_log` | Host UUID, invite/recipient reference, visit start/end and approximate duration. Only the host can read it; the record cannot enter drift or notebook generation. |
| `export_jobs` | Job UUID, account UUID, consistent state revision, creation/expiry, encrypted object reference and hashed download token. Worker queues contain UUIDs, not addresses or exported state. |
| `outbox_jobs` | Job UUID, task type, opaque reference, retry state, availability time. Used for mail, private snapshot invalidation, exports, cleanup, and reducer wakeups without dual-write gaps. |

Traits are normalized internally to `[0,1]`: boldness, social warmth, vocal frequency, plumage saturation, curiosity. Mood is one of `wary`, `content`, `curious`, `drowsy`, or `alert`; a sleep-like pose is an expression of drowsiness rather than another persistent personality dimension. Weather and daily behavior modulate expression, never subtract from the vector.

Bird name validation permits Unicode, 1–40 grapheme clusters, trims whitespace, escapes HTML, and excludes control characters. Duplicate names are permitted; the underlying UUID resolves ambiguity, and accessible descriptions distinguish species/perch when needed. Name history does not affect identity; old notebook prose remains the observation as originally written rather than being rewritten during a rename.

### 3.1 Email and private identity handling

An account's current address has one encrypted authoritative copy on its account record. The keyed lookup digest is restricted to identity lookup/rate limits, never used as a sharding key, metric dimension, public identifier, or message key. Account UUIDs are random, not derived from email. Normalize conservatively without provider-specific dot or plus-address rewriting.

Invitation recipients who have no account need an encrypted address to deliver the invitation and display the required host-only log. Keep it once in a restricted invite-recipient record under a random UUID; this does not create an owner account or aviary. Existing-account recipients reference their account UUID. If a recipient later becomes an account holder, resolve and consolidate the address reference within the identity service. The explicit invitation-email requirement is not permission to copy owner emails into invitations, jobs, logs, analytics, or simulation rows.

Only the mail dispatcher resolves an opaque delivery reference to plaintext immediately before sending. Transactional mail necessarily gives the mail transport its recipient address, but never per-bird interactions, vectors, or simulation history. Host visit-notification email can say an invited visit began and link to settings; it carries no bird observations. Disable payload logging and tracking pixels.

### 3.2 Durability and migration

Persist the vector directly; never reconstruct it from event history, visits, species defaults, or browser storage. Store simulation version, filter state, event cursor, schedule revision, and vector updates in one commit. Persisted nonnegative deltas and a short account-private revision audit permit correctness checks without making event replay the persistence model.

Use database constraints for one aviary per owner, stable bird identities, trait bounds, nonnegative applied deltas, valid state transitions, and unique event IDs. Serialize adoption under the aviary lock so two devices cannot exceed seven. Require migrations to preserve every existing bird ID, vector, voice signature, and notebook entry. A missing/corrupt vector causes a quarantined account-level loading error and restoration from durable storage, never silent reseeding.

Back up the database continuously with point-in-time recovery and test restore of identity/vector continuity before launch. Target zero loss of acknowledged commits within a region through synchronous durability and a documented recovery-time target of one hour. Cross-region disaster recovery is separately measured; never claim zero loss if the actual backup topology cannot provide it. No release is allowed to use “regenerate birds” as a rollback strategy.

## 4. Server simulation

### 4.1 Tick scheduling and transaction

The baseline cadence is 60 seconds per aviary, distributed across the minute by a hash of the aviary UUID. Index `next_due_at`; workers claim batches with bounded leases and `SKIP LOCKED`, then lock each aviary for its transaction. A lost queue message cannot stop simulation because the due-state scan remains authoritative. Advance unobserved aviaries on the same schedule as observed ones; lack of a browser connection is not permission to defer all simulation until return.

Every pass performs the following work:

1. Lock the aviary and read persisted state, the consumed sequence, and the next eligible event range. Ingress assigns event sequences under the same ordering boundary, preventing an event from appearing behind an already-consumed cursor.
2. Advance behavioral time from the last integrated time to each event's server acceptance time, process the event, then advance to the pass time. Use UTC and a monotonic server-time guard; client wall clocks never determine simulation order.
3. Separately advance the persisted drift-integration clock only to the finalized presence watermark, initially thirty seconds behind server time. Credit accepted, deduplicated presence/attention intervals once; decay persisted filters and apply nonnegative personality deltas for the actual elapsed interval. Store `last_drift_integrated_at` independently of behavioral time, so late-but-valid segments never require backdating an already-applied delta.
4. Process due mood/weather boundaries, accepted offers, greetings, settle/re-engagement, and bird-to-bird effects. Random choices use a versioned deterministic generator keyed by bird/aviary seed, scheduled boundary, and event ID, not worker invocation count.
5. Preserve ongoing actions and extend the server-authored action/call horizon to 120 seconds. Explicitly cancel or revise future actions that an intervening event invalidates; never restart every bird at the minute boundary.
6. Evaluate sparse notebook candidates from committed behavioral facts. Persist only eligible observations.
7. Atomically commit vectors, filters, mood/action state, processed event cursor, event results, notebook additions, next due time, and new revision. Write private-cache invalidation to the outbox in that commit.

An owner event requests an expedited pass through this same reducer. Start with a 100 ms coalescing window per aviary and one active reducer at a time. Multiple fast requests cannot create parallel personality writers or more than one time advance for the same interval. Continuous-time filter integration and scheduled mood boundaries make ten expedited passes over a minute equivalent to one ordinary pass, except for the explicitly inserted interactions. Interaction handling cannot reset the minute timer indefinitely; background work remains due on its original cadence.

Worker retries are harmless after either a failed transaction or an ambiguous acknowledgment: the committed event cursor and revision distinguish work already applied. A worker crash before commit produces no partial delta. A crash after commit may repeat a delivery but not a mutation. Use a fencing token if a queue lease can expire while the worker is still running.

When infrastructure misses ticks, catch up from the durable checkpoint using the same server reducer in bounded chronological steps. Fold deterministic continuous filter decay analytically over event-free spans; advance actual mood/weather boundaries with their scheduled times. Do not invent presence or a retrospective notebook entry for every missed minute. Catch-up is outage recovery, not the ordinary simulation model. A new client reads that recovery state rather than running its own catch-up engine.

### 4.2 Precise presence and attention accounting

Start with `activity_window = 300 seconds` and heartbeat/closed-segment delivery every 15 seconds. At every relevant browser event and heartbeat, eligibility is exactly:

`document.visibilityState === 'visible' && document.hasFocus() && trusted_pointer_or_key_age <= 300 seconds && owner_session_active && !locally_settled`.

The final two conditions restrict whose attention can count and end a settled session; they never relax the PRD's required conjunction. Fresh navigation or focus alone does not manufacture a recent pointer/key event. Do not send coordinates, key contents, pointer trajectories, or samples of what the user was doing.

Maintain a monotonic-time interval collector. Visibility loss, blur, activity expiry, settle, logout, and pagehide close the current eligible segment immediately. An eligible heartbeat sends only the closed interval since the last collection, at most 15 seconds, with its unique segment ID and current session anchor. If the activity deadline falls between timer callbacks, clip the interval at that deadline. Detect long frame/timer gaps and stop accounting across suspension; do not convert a laptop sleep into a long presence segment. Use a best-effort terminal beacon, but never rely on unload delivery for correctness.

The server establishes a time anchor when the presence window opens and checks sequence, session ownership, elapsed duration, receipt age, and maximum segment size. Accept a segment only while its bounded freshness window remains open; initially allow at most 30 seconds of delivery delay. Reject impossible/future spans, duplicate segment IDs, and stale replay. Clamp only trivial clock-estimation error; reject materially inconsistent timestamps. This can validate protocol consistency, not prove that a human was looking at the screen, and it should not expand into fingerprinting or surveillance.

Union time coverage across all owner devices before crediting it. Keep a short canonical coverage ledger with a finalization watermark so an interval crossing two ticks, two tabs, or two event batches is still credited exactly once. Finalize an interval only after the 30-second receipt window closes; later segments are rejected rather than rewriting an already-integrated vector. This introduces a small accounting delay, immaterial to week-scale drift. No missing heartbeat extends presence into the future. Connection loss discards unsent presence after the freshness limit; offline hours never arrive as a batch of claimed attention.

Listen-in starts and ends are semantic events tied to a bird and an eligible owner window. A heartbeat renews the bounded attention lease. Apply credit only to the portion that intersects qualified presence. Two devices listening to the same bird use interval union; devices focusing different birds divide a capped total attention interval across those birds rather than multiplying it. Local audio focus can still work without eligible presence, but it earns no idle-attention credit. The server never infers that audible audio proves attention.

Settle and tab-close terminate the originating presence window identically. Their terminal reason is not a positive/negative drift input. Settle adds only the specified small transient mood-quieting effect. If another owner device is still actively qualifying, its presence remains valid: closing one tab must not terminate another device's attention.

### 4.3 Monotonic slow drift

Use a stored leaky input filter per trait, not a moving average of the trait itself. A decaying input filter may continue delivering a small tail of previously earned influence during absence; the personality value itself cannot decay. This directly implements the PRD's distinction between positive history continuing to matter and invented attention on return.

Initial calibration inputs, measured privately within that account:

- Presence supplies at least 80% of the reference positive dose. Ten qualified minutes per local day defines one reference visit; cap credited influence at twenty minutes per local day so an all-day browser cannot compress months of character change into days.
- Listen-in supplies at most 15%, concentrated on social warmth and vocal frequency for the attended bird. Five qualified minutes reaches that day's listen-in normalization. It is strong relative to individual offers, but presence remains dominant.
- Offers supply at most 5%. A nearby eligible offer contributes a small boldness dose; actual acceptance contributes curiosity. At most two qualifying offers per bird per day contribute additional drift. Presentation may still respond later; these are private calibration ceilings, never counters or rewards.
- Settle, muted audio, declined offers, failed requests, absence, visitor watching, and ordinary device closure supply no negative dose. Muting changes presentation and is never treated as neglect. Song responses can affect transient mood without turning audio preference into a trait penalty.

For each trait, let `p` be the persisted value, `c` its persisted expressive ceiling, `E` its nonnegative filter state, and `tau = 3 days` initially. Positive evidence produces a bounded dose `d` over elapsed interval `dt` measured in days; `u = d / dt` is the input rate. Integrate the filter exactly for a constant input interval:

`E_next = E * exp(-dt/tau) + u * (1 - exp(-dt/tau))`

`I = u * dt + (E - u) * tau * (1 - exp(-dt/tau))`

`delta = (c - p) * (1 - exp(-k * max(0, I)))`

`p_next = min(c, p + max(0, delta))`.

Choose initial `k = 0.006 per normalized filter-day`, adjusting trait-specific output curves during calibration. Use stable small-interval numerical functions and double precision. Zero elapsed time means zero time-driven delta. Distribute finalized evidence over its accepted interval, with a consistent handling of the short receipt watermark, so batching cannot change the credited dose. Once a time interval has been integrated, no late request backdates or subtracts its result.

Start trait seeds roughly in `[0.15, 0.50]` and persistent bird-specific ceilings roughly in `[0.65, 0.95]`. Seed ranges are species-appropriate and varied, not rarity tiers. Preserve differences through ceilings, response curves, stable voices, and silhouettes instead of allowing every bird to converge to an identical maximum. Low-pass coefficients, trait curves, and seed distributions are versioned configuration; existing values are never reset when they change.

The numbers are starting hypotheses, not evidence that calibration is complete. The executable calibration gate uses synthetic histories and consented structured prototypes:

| Scenario | Required outcome |
| --- | --- |
| Ten qualified minutes daily, two starter birds | Statistically measurable positive trait movement by day 7; use approximately 0.005–0.02 absolute movement as an initial diagnostic band, not a displayed target. |
| Same regular history through day 21 | Modest observable changes in approach, response, calling, and plumage compared with day 0; blinded reviewers can recognize the individual and notice its change without a number. |
| One typical session | No visible personality jump; target each trait delta below 0.001 during that session. Mood may react promptly. |
| Twenty-minute versus eight-hour eligible day | Influence remains bounded by the private cap; no reward for leaving the aviary open. |
| Rapid offers/listen switching or concurrent devices | Cannot bypass caps, cooldowns, interval union, or idempotency. |
| Fourteen days absent after regular use | Every trait is at least its departure value. Filter tails can add a little; mood continues daily, greeting frequency is ambient, and no distress, desaturation, or mistrust appears. |
| No qualified presence, including a visible unfocused window | Zero presence dose. Birds retain their initial identity and continue mood/action simulation. |

Do not tune against real population interaction histories or average production drift. Use fabricated histories, deterministic seeds, and explicit research sessions with test aviaries. Production telemetry never collects these trait measurements.

### 4.4 Mood, weather, and small social behavior

Implement a time-dependent stochastic mood transition model over the five named states. Store the current state, when it began, the next eligible transition time, and recent transient influences. Score candidate transitions from local time, current mood inertia, personality, accepted interactions, and current weather. Sample on scheduled boundaries, not per rendered frame or per network request. Set a baseline minimum dwell of two minutes, allowing specific meaningful interactions to shorten it without flicker.

Daily-ish renewal means the weight of yesterday's transient influences decays over approximately 12–24 hours; a local dawn window changes the baseline gradually. It does not mean assigning every bird `content` at midnight or session-open. An accepted seed may favor content, a novel motif may favor curious/alert, rain temporarily reduces call rate, wind can favor alert or wary, and a rare alarm-like call may influence nearby birds. Wary can arise from ambient behavior, never as punishment for user absence. Recovery from wary is a normal mood transition, not something the user must fix.

Compute day phase from the account zone with smooth authored curves: dawn around 06:00, daytime from 08:00, dusk around 18:00, night from 21:00, interpolated across boundaries. These are a local-clock approximation, not geolocation or a weather API. Include the active nightjar-like species among the six: its night activity differs from the mostly drowsy flock. Test DST jumps and timezone changes without replaying a day, applying duplicate drift dose, or snapping the palette.

Schedule ambient rain at an initial mean of two or three short episodes per week, lasting about three to eight minutes, with minimum spacing and low intensity. Wind is occasional and similarly restrained. Persist weather start/end and its bounded transient effect. No thunder, distress events, prompts, or real-world weather dependency.

A server-authored bird action is a timed descriptor: action ID, bird ID, pose family, perch/position endpoints, start/end, curve parameters, and any associated call ID. Mood and traits shape the distribution: content favors preening, curious favors head turns, wary favors back perches and scanning, drowsy favors fluffed low poses, boldness increases front-perch use, warmth increases nearby perching and responses. Perch allocation avoids collisions. None of these are user-controllable placement commands.

Calls can produce probabilistic responses in another bird after a natural delay; allow at most two response links per initial call and enforce refractory intervals to prevent endless echo chains. Chorus occurs when compatible calls overlap naturally, not when all birds receive an arrival cue. Weather effects and alarm spread remain bounded and recover without interaction.

## 5. Session and interaction behavior

### 5.1 Return-greeting

Treat a return as a server-observed owner event distinct from qualified presence. Fresh navigation, becoming visible after hiding, and resuming after suspension request one greeting episode. Include the previous owner view-window end/last seen time and a server-calculated absence duration. A stale client cannot claim an arbitrary absence. This information is private simulation input and is never presented as “away for X days.”

Choose the first bird with weighted sampling using boldness, warmth, mood, and how recently it greeted. One bird must notice within one or two seconds under the supported network envelope; a drowsy or wary bird can notice with a subtle glance instead of a conspicuous call. After a very short absence, favor a glance or head angle change. After hours/days, favor longer orientation, an approach, or a varied call. A second bird may respond after a small randomized delay, never in synchrony with the first.

Construct each greeting from continuous parameters: head angle, glance direction, weight shift, approach distance, hesitation, motif order, pitch contour, pause lengths, and optional response delay. Use a fresh episode seed and reject an exact repeat of the last descriptor. A stable voice and personality-specific gesture distribution make the bird recognizable; no canned sequence or rotating set of three clips satisfies the gate. Sample the greeting into the existing action rather than freezing the scene before it plays.

Deduplicate browser retries by event ID. Coalesce near-simultaneous returns from the same owner into one brief canonical greeting episode, so two tabs do not make the entire flock greet twice. Both devices render that same episode; each subsequent real return may get its own subtle notice. Visitors never create return events. Screen-reader narration observes the greeting promptly without a textual welcome banner in the visual scene.

### 5.2 Listen-in and offers

Listen-in is local audio presentation plus a server attention event. A pointer click/tap engages it; clicking the same bird again, clicking empty scene space, leaving bird keyboard focus, or Escape disengages. Choosing a different bird transfers attention. Use one local state machine so pointer, keyboard focus, and event handlers cannot accidentally begin two attention windows. Transfer/end events are idempotent, and forgotten ends expire with the heartbeat lease.

The offer menu contains seed, a small library of softly synthesized song fragments, and still pool. Use four authored song motifs initially; these are offered musical phrases, not recorded bird calls. A user chooses an offer kind/motif, not a personality target or a placement command. The server chooses a plausible receiving bird from those near the offering area and not on cooldown; the command references the chosen bird internally so the cooldown is truly per bird.

Start with a three-minute per-bird offer cooldown enforced by the reducer for all three offer kinds together. Serialize simultaneous submissions. Only one offer prop is active in the scene at a time, with a bounded natural lifetime; further requests while the prop remains active are a quiet no-op or an understated inline menu response. A cooldown rejection does not trigger another reaction or drift credit. Do not expose countdowns, escalating prices, or a “come back later” prompt.

The reducer chooses reactions using current mood and curiosity: approach and inspect; pause and watch; delayed approach by a wary bird; no approach when drowsy. Seed is a gesture, not food inventory. Pool can result in drinking, bathing, or watching without thirst mechanics. A song fragment may prompt joining, a contrasting call, or a quiet pause based on vocal frequency and mood. Declining/ignoring an offer is normal, with no failure label, guilt, or negative trait delta. On a network error, the system panel uses direct error language rather than pretending the bird refused.

### 5.3 Settle, undo, and continued watching

On accepted settle, end the originating presence/attention window, apply a small transient quieting influence, and schedule a roughly four-second shift to an evening palette with a soft audio reduction. It is an optional goodbye, not a completion state. Keep a local settled latch so subsequent timer pings or incidental pointer movement do not resume attention. Persist enough session-scoped state to distinguish settled watching from fresh navigation.

Any scene click within five seconds reverses the shift, consuming that click as undo before it can trigger listen-in or another action. Provide equivalent keyboard activation of the scene and an accessible undo action in the top-bar gesture menu. The undo request references the settle event and its anchored elapsed time; accept only a genuine in-window activation with bounded network transit tolerance. Undo removes the still-active transient settle envelope, not past personality drift. After five seconds, an intentional click/key activation in the aviary re-engages normally and smoothly returns lighting to the current local-time palette.

Closing the settled tab ends the session overlay; the next navigation is a fresh view of the current canonical aviary. If the browser disappears without a close message, expire its overlay with the session lease rather than leaving the aviary permanently in evening. Another actively re-engaging owner device can clear the shared transient visual envelope. Existing qualifying presence on another device is never cancelled merely because this device settled. Visitors cannot settle or undo.

### 5.4 Notebook generation and adoption

Generate notebook prose from committed behavioral evidence with authored, composable templates and a shared naturalist vocabulary. No external language-model service is required, and private interactions never leave the simulation boundary. Templates use names, species, perch descriptions, time-of-day context, observed call shapes, and short factual comparisons. They do not serialize event types, expose trait numbers, count user visits, praise diligence, or imply that a bird suffered during absence.

Rank candidates such as a bird greeting before another for the first time in a recent seven-day window, an unusual quiet morning, a sustained distinctive perch/pose, or a small weather reaction. Evaluate novelty and evidence, not whether a session ended. Start with a normal minimum spacing of 48 hours and an approximately three-day baseline opportunity when a real observation exists. Exceptional moments can shorten spacing to twelve hours, with no more than two entries in a rolling seven days from the same observation family. These are tuning rules for rarity, not promises of an entry on a timer.

Finalize text only when its specific facts are true. A “first time this week” bird comparison needs actual per-bird greeting evidence; if that evidence has expired or is incomplete, choose other wording. Persist the text and template version; later template changes do not regenerate the history. Read older entries through stable cursor pagination indefinitely. Notebook interaction never edits, annotates, or deletes entries. Account deletion is the privacy exception that removes the entire account's data.

Create two starter bird IDs and their persistent seeds transactionally when onboarding begins. The server chooses two different species from the six-species pool to establish contrasting silhouettes/calls; naming defaults are suggestions, not a catalog. Confirming names completes adoption and uses the one-time empty-field/fly-in surface. Retrying onboarding does not create two additional birds. Renaming lives under bird settings inside account/settings and has no simulation side effects.

For later adoption, use elapsed aviary age, initially offering opportunities at 90, 180, 270, 365, and 540 days for birds three through seven. This produces a few birds over months, five or six around one year, and at most seven. The dates are internal policy, never a progress calendar or a countdown. Show at most one pending opportunity, quietly in account/settings when opened, without a badge, notification, species shopping catalog, or forced modal. Declining/postponing costs nothing. For an old two-bird account, present further age-eligible opportunities no sooner than fourteen days after each accepted adoption so five birds do not arrive in a burst; that spacing depends on adoption time, never interaction frequency.

Additional species are server-selected without rarity. Prefer a species not already represented until the pool is exhausted, then permit a visually and vocally distinct individual from an existing species. At seven the offer simply ceases to exist; there is no visible capacity meter. The seven-bird limit is enforced in the database transaction as well as the interface.

## 6. HTTP API and contracts

Version the API under `/v1`. All mutation routes require validated content types, same-origin/CSRF checks for cookie-authenticated requests, authorization, bounded payloads, and idempotency keys where retries can create work. Owner tokens and visitor tokens are separate credential classes. Errors have a stable code, a matter-of-fact message, and an appropriate retry/re-authentication action. Never return a bird's private record in an error.

| Method and route | Contract |
| --- | --- |
| `POST /auth/magic-links` | Email input; generic response regardless of account existence. Create a 15-minute, one-use challenge and dispatch transactional mail. |
| `POST /auth/magic-links/consume` | Atomically consume token, create/recover identity as appropriate, issue per-device secure session cookie; replay/expiry gives the same useful sign-in error. |
| `POST /auth/logout` | Revoke this session and terminate its presence windows. |
| `GET /account`, `PATCH /account/settings` | Read/update timezone, audio/accessibility preferences, and notification consent with a settings-version precondition. No vectors or interaction statistics. |
| `GET /account/sessions`, `DELETE /account/sessions/{id}` | List the owner's device sessions and revoke the chosen one, effective on its next request. |
| `POST /account/email-change`, `POST /account/email-change/verify` | Send verification to the pending new address; atomically switch only on valid consumption. The old address remains authoritative until then. |
| `POST /account/export` | Queue a consistent private export and email its expiring download link to the verified address. Repeated idempotency key yields the same job. |
| `POST /account/deletion`, `POST /account/recovery` | Mark a 30-day recoverable deletion, or restore within that window after authenticated explicit confirmation. |
| `POST /aviary/adoption` | Finish the two starter names once, or accept the currently eligible age-based opportunity. Server chooses species and verifies capacity. |
| `PATCH /aviary/birds/{id}/name` | Owner-only rename with bird-name version; stale edits return a conflict for refresh rather than silently overwriting. |
| `GET /aviary/snapshot` | Owner render projection, server time, revision, action/call horizon, account capability flags. Supports ETag/conditional requests after auth. |
| `POST /aviary/events` | Small ordered batch of owner semantic events; max 16 events/32 KB initially. Return per-event accepted/rejected/applied status, server sequence, and optionally committed result/revision. |
| `GET /aviary/event-results?ids=...` | Bounded lookup of pending event results for this owner; useful after a timeout without resubmission under a new ID. |
| `GET /aviary/notebook?before=...&limit=...` | Immutable entries, max 30 per page, stable time/ID cursor. Only owner access. |
| `POST /visits/invitations` | Explicit owner action with recipient email; create a one-use invite and dispatch its link. Off by default means no invites exist until this action. |
| `GET /visits/invitations`, `DELETE /visits/invitations/{id}` | Owner sees outstanding/active permissions and can revoke any of them. |
| `POST /visits/redeem` | Exchange valid unused token for one render-only visitor cookie; atomic with visit-log start. |
| `GET /visit/snapshot` | Visitor-only ambient projection for its specific aviary. Check invite/session/host status before every response, including `304`. |
| `POST /visit/end` | End that visitor session best-effort; no owner simulation event. |
| `GET /account/visit-log?before=...` | Host-only visitor email, date, approximate duration, and status; no settings badge or push-style feed. |

A submitted event envelope contains `event_id`, `client_window_id`, `type`, bounded monotonic timing fields, and a type-specific payload. Supported types are `return`, `presence_segment`, `presence_end`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `undo_settle`, and `reengage`. The server derives account/session identity from credentials; a body cannot claim another owner. Offer payloads carry a kind and optional library motif ID, never a desired reaction. Unknown fields and any attempted vector/mood assignment are rejected.

Each snapshot includes `schema_version`, `aviary_revision`, `server_time`, `valid_until`, timezone/day-phase/weather envelopes, and a small bird array. Each bird provides its stable ID, name, species/visual revision, current mood for the rendering layer, perch/position, current action and phase, future action descriptors, and voice/grammar references. Call events include event ID, bird ID, scheduled onset/duration, motif grammar version, variation seed, and bounded synthesis parameters. A server-derived appearance/behavior projection supplies only what rendering needs; it does not expose a normalized trait vector or hidden daily totals. Quantize visual material presets and nonessential response controls to avoid accidentally publishing a one-to-one trait-to-number API.

Public projection generation uses an explicit allowlist and negative serialization tests. Private filters, seeds that permit reconstruction of the entire simulation, cooldown counters, coverage ledgers, account emails, event logs, and notebook evidence never enter the scene snapshot. A stable voice seed for local procedural variation is a separate seed with no access to private simulation randomness. The requested export is a separate authenticated job with its documented vector exception, not a reusable public snapshot mode.

### 6.1 Pull cadence, reconciliation, and outages

Pull immediately on navigation, visible return, focus recovery after a long gap, and a render gap above two seconds. While visible, pull every fifteen seconds with small jitter; after an owner action, use its committed result or brief result polling to fetch the new revision promptly. Hidden tabs stop scene rendering, routine pulls, narration delivery, and local audio scheduling. Server ticks continue. Snapshot polls by themselves are not presence.

Track last applied revision and action/call IDs. Discard out-of-order responses. Apply state replacements at one frame boundary, preserving ongoing actions by ID and interpolating changed position/lighting envelopes. Estimate server-clock offset from request timing and smooth minor adjustments. On resume, discard expired calls and sample current action phases; never play the calls missed during suspension. Local client clocks cannot advance mood or personality.

One canonical record removes personality merge conflicts; it does not make two network screens physically simultaneous. Document the normal fifteen-second maximum polling staleness, with prompt refresh for the acting device. Both converge to the same committed revision, and neither can persist a conflicting state. Where a stale request attempts a versioned settings/name edit, respond `409` with the new version and matter-of-fact refresh guidance; semantic interaction events remain ordered rather than merged.

Keep pending owner commands in a bounded in-memory queue with stable event IDs. Retry network-ambiguous commands within thirty seconds and query their results; do not durably replay offers, settle, or old presence after reopening the browser. If a command was committed despite a lost response, the same ID returns its original outcome. When authentication expires, close attention windows, stop writes, clear private browser state on logout, and show the direct sign-in surface.

During a transient outage, render only the server-authored valid action/call horizon. Once exhausted, retain a quiet known pose with non-semantic breathing/ambient presentation and gracefully quiet the audio; do not simulate an invented sequence of mood or call events. Show a matter-of-fact connection explanation in the system surface, without a full-screen spinner. Resume from the fresh canonical phase after recovery. Never synthesize a replacement aviary, reset a vector, or label missing data as a bird mood.

## 7. Frontend scene and accessible surfaces

### 7.1 Composition and animation boundary

Use Canvas 2D for the moving scene, with a server-rendered lightweight SVG/pose bootstrap that is replaced seamlessly at the same sampled action phase. Keep the four top-bar groups, dialogs, notebook, narration, captions, and keyboard bird controls in semantic DOM. The scene engine has no framework re-render on each animation frame; a bounded scene graph samples immutable snapshot descriptors through `requestAnimationFrame`.

Compose soft sky and foliage in the background, three depth-separated perch zones, birds at their canonical perch coordinates, and sparse foreground branches/leaves. Cache static drawing into offscreen canvases; use compact vector shapes or small atlases for feather/pose details. Scale plumage richness gradually through server-derived material choices. Do not add an unnecessarily large 3D library, physics engine, video asset, or per-frame DOM layout pass.

Sample movement, head angles, breathing, preening and weight shifts from server-authored curves and action times. Continuous micro-motion runs while visible even if the window is not focused; attention eligibility is separate from visual liveness. Interpolate a changed perch along a slow short arc and preserve an ongoing trajectory when a later snapshot references the same action. On a large correction after suspension, sample the current authoritative pose rather than playing the entire old path rapidly. Never visibly reset all actions on snapshot arrival.

Leaf and feather drift are the deliberate exception to server-authored actions: generate these ornaments locally at slow random intervals, with a fixed small particle pool and no database/event representation. Gentle parallax is an ambient layer offset, not a user-controlled camera, scroll mechanic, or mouse-follow spectacle. Ornaments cannot affect mood, bird decisions, calls, or notebook facts.

### 7.2 One-screen layout

Reserve top-bar height plus safe-area insets; fit the entire scene inside the remaining dynamic viewport height. Body scrolling, scene panning, scene zooming, and drag-to-place are absent. Browser text zoom remains available for accessibility; system dialogs and notebook may scroll independently without turning the aviary into an explorable canvas.

Use normalized perch zones with responsive anchor layouts. At narrow widths, bring anchors closer while retaining depth cues and uniform bird scale. Reflow perch anchors, rather than cropping birds or distorting their silhouettes. Define layouts for 320 px phone width through ultrawide desktops and test landscape phones as well as portrait. Seven birds must remain inside a safe inset at every animation endpoint and along travel paths. Bound paths and caption placements during every viewport resize, including virtual-keyboard changes.

The interactive DOM layer provides generously sized invisible hit areas and a visible focus treatment for the same birds, without permanently labeling them. Use at least 44 CSS px touch targets where space permits, with collision-aware hit testing and keyboard alternatives for tight layouts. Maintain aspect ratio through uniform scene scaling and anchor spacing; do not crop one bird to preserve another. Capping device pixel ratio at 2 is allowed for runtime budget after visual QA.

Use calm blues, greens, browns and ochres, with day/night tokens designed as pairs for actual text/background combinations. No saturated status badges or score-like numbers. User-copy text is limited to top-bar/system surfaces and explicit caption/narration exceptions. Bird mood is expressed by posture and call, not a label or icon in the scene.

After four seconds of pointer stillness, fade the top-bar background and decorative treatment nearly away. Restore on pointer movement, touch interaction, or keyboard activity. Never fade while a menu is open or focus is within the bar. Do not lower text/icon contrast by blindly applying opacity to the entire control subtree; active/focused content remains legible against all palettes. The resting bar should recede while the discoverable control strokes retain their tested contrast. A pointer interaction restores the bar before activating its target.

### 7.3 Keyboard, narration, captions, and contrast

Implement the bird layer as one Tab entry with a roving tab stop; arrow keys traverse birds in stable spatial order, preserving bird IDs during rearrangement. Tab through the four top-bar groups, then enter the aviary at the first bird; Tab onward exits normally. Enter engages listen-in, Escape disengages, and moving focus to another bird transfers it. Settings dialogs use native dialog semantics, contain focus appropriately while open, close with Escape, and return focus to their trigger. Offer choices and settle are fully operable without pointing. Provide an explicit offer keyboard shortcut displayed in the menu, using a modifier combination that does not seize common assistive-technology commands, and allow it to be disabled.

Keep a dual-tone soft focus outline with enough separation from feathers and branches to read in day and night. All user text must meet WCAG AA: 4.5:1 for normal text and 3:1 for qualifying large text; controls/focus indicators also receive at least 3:1 against adjacent colors. Caption backgrounds use a restrained solid/translucent backing whose worst-case composited color is tested, not an assumed average sky color. Test forced colors, text enlargement, 200% zoom, and 400% zoom/reflow for system surfaces. Do not disable pinch/text zoom in the viewport metadata.

Narration is composed from the same canonical scene and active call descriptors as the renderer. Use a shared naturalist lexicon/template package with notebook generation, allowing a compact client-side narration module to summarize current state without another network roundtrip. Produce about one idle observation every 45 seconds, within the required 30–60 second range, skipping redundant prose. Example: “pip rests on the front rail, turning toward a low call from the back branch.” Never read a vector, an event code, or a mechanical list of positions/moods.

Use one polite, atomic live region with a bounded queue. User events such as a greeting, offer reaction, or settle get the next suitable narration slot promptly; coalesce a burst rather than interrupting every few seconds. Discard obsolete idle narration when a user event arrives and clear the queue when hidden. Do not speak every animation frame, call onset, poll, or focus movement. Focus labels can name the bird/species and explain listen-in without announcing hidden state. Provide narration pause and a visually readable prose area in accessibility settings; using those controls does not alter simulation.

Call captions come from the runtime call score described in section 8. Place them near their bird, with collision-aware offsets inside scene bounds and a gentle fade matching the call. A short call's text persists long enough to read, initially at least two seconds. For overlapping calls, preserve bird association and stack/reflow nearby captions; never silently discard a bird's caption because another is calling. Avoid adding every caption to the live narration queue. A visitor may use local caption, narration, volume and reduced-motion controls without acquiring bird interactions.

### 7.4 Reduced-motion is a separate rendering design

Honor `prefers-reduced-motion` at first paint, with an account preference to explicitly opt in; effective mode is reduced when either requests it. Do not briefly play full motion before settings hydrate. Replace body/head micro-motion with slow cross-fades between authored still poses, initially over 4–8 seconds. Replace flight with a cross-fade between perch poses, not a shortened flight. Remove leaf/feather drift and parallax. Keep ambient day/weather color changes but slow them further. Settle and its undo remain legible through gradual palette changes.

The semantic action, call score, mood, greeting selection, notebook, and drift are identical between rendering modes. A greeting becomes a varied glance/approach pose sequence rather than disappearing. Audio quality and captions remain complete. Review the reduced-motion experience as its own calm visual composition, with users sensitive to motion, rather than accepting “no animation errors” as proof of quality.

## 8. Procedural audio pipeline

### 8.1 Grammar and identity

Give each species a small authored grammar of call motifs: whistle arcs, paired notes, pulses, short trills, breathy edges, rests, and phrase alternatives. Each bird gets an immutable voice signature at adoption: bounded register, timbre/harmonic shape, characteristic interval contour, rhythm tendencies, and variation seed. Two birds of the same species still differ. Mood and drift modulate timing, density and expression within that signature; they do not replace its grammar or shift it beyond recognizability.

The server chooses when a bird calls, which motif family fits the action, and whether another bird responds. The client expands a call descriptor deterministically into a score containing timed notes, frequency curves, amplitude/filter envelopes, rests and timbre parameters. It does not roll new behavioral decisions or schedule a new call because a local oscillator ended. Grammar versions and seeds make the same canonical call reproducible on another device while consecutive calls remain varied.

Guarantee variation through bounded continuous note lengths, inter-note gaps, envelope slopes, vibrato, timbre changes, phrase length and motif alternatives. Preserve recognizable anchor intervals. Reject exact repeated score fingerprints in the server's recent call window where necessary. Procedural does not mean unbounded randomness: squeals, sudden register jumps, digital beeps and busy warbling are failure cases to be removed in listening review.

Compile captions from the final score, after variation is resolved, so “a soft three-note rise” actually corresponds to a soft rising three-note phrase. Generate descriptions of low trills, held notes, pauses, rises and sharp calls from the same notes/envelopes that drive synthesis. Do not attach one fixed caption string to a species or nominal motif. Muted/fallback execution compiles that same score and caption without requiring an audible device.

### 8.2 Synthesis and mixing

Create at most one AudioContext per document after the appropriate permission/gesture. Use a small AudioWorklet with a fixed pool of approximately sixteen synthesis voices and reused buffers where supported. A native OscillatorNode/GainNode/BiquadFilterNode implementation within WebAudio is the compatible alternative when AudioWorklet is unavailable; explicitly disconnect and release each ended node. Neither path uses recorded calls or downloaded audio loops.

Use per-bird gain buses into a restrained chorus bus, with a soft limiter for accidental overlaps and conservative headroom. Use simple pan tied to scene position with a mono-safe mix; spatial audio must not be required to identify a bird. Keep ambience, if present, extremely low and synthesized from generated noise, never a recorded loop. Start with a cap of three or four naturally overlapping calls, allowing all seven identities time in the mix rather than letting one frequent bird dominate.

Maintain a short scheduling lookahead of about 100 ms, serviced approximately every 25 ms while visible. Schedule note envelopes against the audio clock using the server-clock mapping. The canonical call horizon is much longer than this queue; do not enqueue two minutes of uncancellable oscillator work. On a cancelled future call, hidden tab, permission loss or logout, ramp out and remove scheduled work. If a call is already partly elapsed at navigation, synthesize its remaining phase when feasible; otherwise skip that expired onset rather than replaying it late.

Listen-in ramps over about 1.5 seconds on engage/disengage, with a target bird lift near +3 dB and other bird buses near -9 dB relative to their ambient reference. Tune by ear, with a nonzero floor for every other bird bus; listen-in never hard-mutes the surroundings. Transfer attention by simultaneous smooth gain automation. Settle reduces the entire chorus gradually and restores the current ambient mix on re-engagement. Explicit user mute can silence the master bus and never subtract personality or presence.

### 8.3 Capability and fallback states

Model audio as `not_started`, `running`, `user_muted`, `autoplay_suspended`, `unavailable`, or `interrupted`. Only an actual running context is considered audible. Browser permission denial, missing WebAudio, hardware/context errors or failed resume enter graceful silence with captions on by default. A suspended autoplay context can be retried on a genuine gesture; do not bombard the user with permission prompts or pause the aviary behind an audio modal. Keep a clear audio status/control inside accessibility settings. Preserve deliberate mute preferences.

When audio returns, synchronize to current score time, not the beginning of the session. Every failed/successful transition must clean up context listeners, schedules and nodes. No recorded-audio fallback exists. The same procedural architecture serves the four offered song fragments, with quiet authored melody parameters and mood-shaped bird responses.

Audio release gates include randomized 30-minute sessions, mono headphones/speakers, low-volume listening, repeated returns, device interruptions, and seven-bird mixes. Have listeners identify each of a user's birds from call samples before and after simulated three-week drift. Test both naturalistic appeal and recognizability; a technically correct oscillator implementation can still sound mechanical or uncanny and cannot ship on automated tests alone.

## 9. Identity, visits, export, and deletion

### 9.1 Authentication and session management

Generate at least 256 bits of randomness for magic-link and session secrets; store only hashes. Magic-link consumption is a single transaction testing purpose, expiry and unused status. Use a token landing page with no third-party resources and no-referrer policy; exchange via an intentional same-origin POST so mail scanners/prefetch cannot consume a link merely by issuing GET. Remove the token from browser history immediately after exchange. Avoid distinguishing unknown accounts in responses.

Start mail-request limits at five requests per fifteen minutes per address lookup key and twenty per hour per IP, with short expiration of abuse counters. Tune operationally without using emails as general identifiers; permit another request after the limit window rather than locking the account indefinitely. These limits do not create aviary reminders. Keep magic-link validity at exactly fifteen minutes; requesting a new link never revives a used one.

Issue Secure, HttpOnly, SameSite cookies with an opaque per-device session token, initially thirty-day absolute lifetime and seven-day inactivity renewal rules. Rotate tokens safely without replacing the stable device-session ID. Session revocation is checked on every protected request, including cached snapshot access. The session list shows understandable device labels and last-use times; revoking a device ends its pending attention lease. A stale in-flight interaction still has exactly-once semantics if it was accepted before revocation; newly unauthorized requests cannot append events.

Verify a new email with a fresh purpose-bound fifteen-minute link before atomically moving the authoritative address. The old address remains valid until commit. Clear expired pending ciphertext and enforce uniqueness in the identity service. Require a recently authenticated session for email change, export and deletion; a fresh magic link is the available re-authentication mechanism, without introducing passwords. Use clear system copy for replay, expiry, outages and conflicts.

### 9.2 Visit capability isolation

Creating an invitation requires a deliberate host action naming the email recipient; it does not enable public visibility. An unused token expires after thirty days and cannot be restored. The one-time emailed token proves possession of that invitation, with no public profile or full account creation required for the visitor. Redemption atomically marks it consumed and creates a scoped visit session, so simultaneous token opens cannot create two visits.

The visitor bundle reuses the scene/audio/narration renderer with a read-only projection and no owner event dispatcher. The backend independently rejects all visitor calls to owner endpoints; hiding controls is not authorization. Render the same bird appearances, moods, perches, scheduled calls, weather and canonical timezone as the host. Do not prettify the aviary or create greetings, listen-in changes, offers, settle, notebook access, host-facing cursors, or social overlays. Local volume/caption/reduced-motion preferences are allowed because they do not affect the host or simulation.

Visit snapshots never append presence or interaction events. Maintain approximate visit duration in the visit service using successful snapshot requests, not the drift event log. A close beacon can finish the record; otherwise cap duration at the last successful pull plus one poll interval and expire the idle session. The host-only log includes visitor email, date and approximate duration, plus outstanding invitations. It is ordinary account transparency, never an aviary observation or an engagement counter.

Revoke an invitation in one transaction, invalidating every session issued from it. On the visitor's next snapshot pull, including a conditional request, return a generic no-longer-available error instead of a cached body or `304`. Cancel its audio, clear private render state, and replace the scene with the matter-of-fact access surface. Hidden visitors validate before drawing again. Give visitor projections a short freshness lease, initially twenty seconds; if the browser cannot revalidate, stop the visit rather than rendering cached host activity indefinitely. Revocation cannot erase pixels already seen, but it prevents subsequent authorized access.

The notification toggle is off by default and only in settings. If enabled, the transactional outbox sends at most one visit-start email per actual redeemed session; retries use the visit ID for deduplication and verify current consent before dispatch. No host email is sent merely because an invitation was opened by a scanner, revoked, or expired. No in-product badges, notifications, shared cursors or live-host presence information are added. Invitation creation/revocation updates the settings list in place.

### 9.3 Export

Take a transactionally consistent snapshot containing birds and stable IDs, names, current vectors under the explicit portability exception, moods, notebook entries, and account settings, with a schema version and snapshot time. Paginate internally while retaining a consistent database snapshot so a large notebook does not mix unrelated revisions. Exclude raw auth secrets, device tokens, internal event logs and private infrastructure details.

Write encrypted export output to private object storage; email a short-lived download link to the currently verified address. Initially expire the link after twenty-four hours and the object after forty-eight hours. Require owner authentication and a valid download token, stream with attachment disposition, and disable public caching. Recheck account status at download time. Jobs and mail carry only opaque identifiers; no vector or notebook contents go to the email provider. Deletion cancels pending exports and removes completed objects immediately. The settings surface reports delivery plainly, without exposing a vector preview.

### 9.4 Recoverable deletion and hard erasure

After the user's explicit delete confirmation, mark the account for deletion immediately with a UTC deadline exactly thirty days later. Revoke visitor access and ordinary device sessions, cancel mail/export jobs, purge downloads and edge projections, and stop accepting owner interactions. A newly authenticated session during the window opens a matter-of-fact recovery surface with “I changed my mind.” Keep birds, vectors and notebook durably intact during that window; the server may continue their ambient tick so recovery has continuity, but no synthetic presence is added.

Recovery clears the deletion marker transactionally before the deadline and resumes normal access. It does not silently reinstate revoked invitations or previously revoked device sessions. At the hard-deletion deadline, delete every live account-linked record: identity and pending challenges, aviary, birds/vectors/filter state, events/coverage, notebook/evidence, visit grants/logs, private diagnostics, jobs, exports, blobs and cache entries. Expired delivery recipients that have no remaining lawful account/invite reference are erased as well. Test idempotent cleanup when a batch fails midway.

Backups must not defeat the deadline. Encrypt private account data with account-scoped keys whose destruction cannot be undone by restoring a database backup; the independent key service must not restore deleted keys from the same recovery path. At the deadline, destroy the account key and sweep live rows/indices, leaving any backup ciphertext unrecoverable. Maintain only non-reversible cleanup completion evidence without a retained account identifier. Restoration drills must show that a deleted account cannot reappear. Short-lived diagnostics carrying a synthetic account UUID are explicitly purged; anonymous aggregate histograms have no account linkage or removable per-account record by design.

## 10. Privacy and operational observability

Store interaction events only to drive their owner's simulation, with no analytics export, training feed, recommendation input, session-replay product, or third-party behavioral SDK. Give simulation storage and operational metrics separate credentials, ingestion schemas and network access. A telemetry service account cannot read birds, vectors, notebook, visit history or event tables. A dashboard cannot later expose “average boldness” because that data never enters its pipeline.

Retain consumed raw simulation events initially for seven days for account-private correctness recovery, then remove them once the cursor and durable aggregate/filter state make them unnecessary. Retain unconsumed events until successfully processed or explicitly resolved; alert on backlog rather than deleting it under a TTL. Short-lived coverage and observation evidence have their own bounded windows. Keep notebook entries indefinitely for that account. Keep visit-log details initially ninety days for sharing transparency, while active/outstanding permissions remain visible until resolved. Publish the retention choices plainly in account settings.

Allow aggregate request counts, latency/error histograms, anonymized coarse session-duration histograms, first-bird timing, frame timing, audio-context errors, tick runtime and due-job delay. Do not attach account/bird/device IDs, email, invitation IDs, action/call IDs, full URLs, event names that reconstruct interactions, vector values, bird count, mood, offers, listen duration or notebook text to aggregate metrics. Synthetic performance fixtures may label their artificial bird-count scenarios; that is different from classifying production users by their birds.

Aggregate client samples before transport where practical and strip request-level identifiers on metric ingestion. Use coarse device/browser buckets, a bounded sample rate and no stable analytics cookie. Session duration, if collected, is an anonymous histogram without an account dimension, not a retention/frequency report. Redact request bodies, headers, query tokens and state objects from exception capture. No automatic screenshot or DOM recording is enabled. Internal security/support logs may carry the synthetic account UUID only where necessary to diagnose an account-level failure, with restricted access and short retention, never simulation state.

Provide a plain privacy-policy link in settings identifying the operational categories and explicitly excluding per-bird interaction analysis. Treat an observability schema change like an API change: require allowlist review and a test payload proving excluded fields are rejected. Calibrate bird behavior with synthetic fixtures and voluntary prototype research, not production interaction aggregation.

## 11. Performance budgets and verification

Performance is a launch requirement because a late, static or stuttering first bird changes the experience. A bundle under the maximum is necessary but insufficient; measure the actual first bird and sustained session.

| Surface | Budget and implementation target | Verification |
| --- | --- | --- |
| Initial JavaScript | Hard cap below 2 MB gzipped. Target at most 250 KB total initial-route JS and under 20 KB for the drawing bootstrap. Lazy-load notebook/settings/visits and nonessential audio modules. | Build emits gzip-size manifest; CI fails at the hard cap and flags target regressions. Include all automatically fetched initial chunks, not just one entry file. |
| First-bird critical delivery | Aim for HTML, compact projection and two starter poses in roughly 36 KB gzipped or less; no blocking font, external image, settings chunk or audio permission. | Trace cold and warm authenticated navigation and verify the first paint containing a bird. |
| Time to first visible bird | Below 500 ms on the agreed mid-tier mobile/4G reference. Budget roughly 180 ms for connection/edge, 130 ms for auth/state/HTML, 90 ms transfer/parse, and 60 ms draw/paint, leaving margin. | Twenty repeated runs per reference condition, cold browser cache included; require the target in the reference suite, track p75/p95 live and synthetic distributions. Never claim the target from an unpainted JS mark. |
| Greeting | First noticing action begins within 1–2 seconds of a normal owner navigation/return. Expedited reducer p95 target below 250 ms excluding client network. | Measure actual visible/accessible greeting onset with normal, slow, and interrupted connections. Slow-path errors are reported separately, not hidden in an average. |
| Snapshot | Kilobytes, initial target below 16 KB uncompressed at seven birds; bound descriptor horizon/count. | Contract tests with worst-case active weather, chorus and transitions. No notebook/history in the snapshot. |
| Idle rendering | 60 fps on a five-year-old mid-range laptop across a full thirty-minute session; target at least 95% of frames within 16.7 ms and typical scene work below 8 ms. | Test a documented 2021-class quad-core laptop with integrated GPU/8 GB memory at representative resolution, with two and seven synthetic birds. |
| Audio | Bounded voice/node count, one context, no repeated underruns, stable gain ramps and prompt interruption cleanup. | Audio harness plus real-browser listening and a thirty-minute randomized call/interaction run. |
| Client memory | No sustained memory growth over thirty minutes after initial caches warm. | Leak CI compares post-GC retained heap at minute 2 and minute 30 and samples slope; reject growing retained object counts, detached nodes, contexts, buffers or queues. Repeat to distinguish noise from a leak; a larger acceptable leak slope is not the fix. |
| Simulation | Baseline minute cadence; p99 end-to-end tick latency alarm above 5 seconds, including due-to-complete delay, plus separate queue and reducer measurements. | Load and fault tests across disconnected and active aviaries; synthetic monitoring checks that states advance while no client exists. |

The mobile reference should be a real mid-tier device plus a reproducible browser profile, initially using approximately 9 Mbps down, 1.5 Mbps up, 80 ms RTT and a documented CPU throttle. Cold session validation and a cold private snapshot cache are distinct tests; do not use only a warmed developer desktop to certify 500 ms. If the private edge/auth path misses the target, reduce roundtrips, critical bytes and origin distance; do not introduce a fake starter bird or a spinner to mask the miss.

Run scheduled synthetic browsers from common geographies with synthetic accounts and fixed histories. Production RUM remains aggregate-only. Alert on first-bird latency regressions, frame-time distributions, audio-context error spikes, tick p99 above five seconds, overdue-tick age, queue backlog, transaction retries and snapshot failure rate. Do not instrument retention goals, daily-use streaks, click funnels, population drift or return-notification conversion.

Support the last two major versions of Chrome, Safari, Firefox and Edge, including mobile Safari/Chrome for relevant platforms. Combine maintained browser-version support policy with feature detection; supported browsers with an unavailable audio device still receive the graceful-silence aviary. Older unsupported browsers receive a concise matter-of-fact upgrade surface. Do not ship a large legacy compatibility layer or recorded-audio workaround.

## 12. Build sequence, test gates, and rollout

Organize delivery around complete vertical experiences rather than shipping the accessible or durable version later. The responsibilities below are workstream ownership for the eventual engineering team, not instructions to create additional planners.

| Increment | Owner responsibility and concrete output | Exit gate/dependency |
| --- | --- | --- |
| 1. Contracts and expressive prototypes | Domain/backend lead owns private/public schemas, invariant tests and a synthetic-time reducer harness. Visual/audio/accessibility leads define six coherent silhouettes and voices, top-bar tokens, normal/reduced-motion studies, narration and caption grammar. | Reviewed product voice, explicit conflict decisions from section 1, prototype recognizable calls, approved contrast combinations; no production interaction collection. |
| 2. Durable two-bird slice | Backend owns magic-link account creation, atomic two-starter onboarding, stable bird IDs/vectors, minute scheduling with no clients, event cursor/locking, and private snapshots. Web owns embedded initial pose and continued motion. | Restart/restore does not change identity; suspended laptop and newly signed-in phone read the same canonical record; first-bird critical path measured early. |
| 3. Attention and responsive behavior | Backend owns presence unions, expedited passes, monotonic filters, mood/time/weather, greetings, offers and settle. Web owns keyboard/pointer state machines and cancellation. | Truth-table, multi-device, race, replay, absence and one-/three-week calibration gates pass on synthetic histories; no single-session visible drift. |
| 4. Complete accessible aviary | Visual/audio/accessibility owners finish six species, three-to-seven-bird layouts, procedural chorus, mix ramps, narration, captions, reduced motion and silence fallback. | All interaction paths usable by keyboard and screen reader; real listening and reduced-motion reviews approve the affective experience; performance holds at seven synthetic birds. |
| 5. Quiet account depth | Backend/web own notebook rarity and indefinite pagination, naming/settings, age-based adoption, export/delete/recovery, invite/revoke/log and optional email consent. | Full account-lifecycle and visitor-isolation tests; no history or vector data leaks into ordinary snapshots/telemetry; hard erasure restoration drill succeeds. |
| 6. Release qualification | Operations and QA own browser/device matrix, geography probes, thirty-minute soak, migration/backup rehearsals, load testing and privacy schema audit. Product/editorial review inspects every normal/error surface. | Required budgets and invariants pass together. Screen-reader, caption and reduced-motion surfaces are launch blockers, not a post-launch phase. |

Maintain a single end-to-end reference scenario throughout: create account, meet/name two birds, see a mid-action return, qualify five minutes of quiet presence, listen to one bird, offer and watch a shaped response, settle/undo, close without settling, resume on a second device, inspect a rare notebook observation, invite/revoke a visitor, export, request deletion, and recover. A time-controlled synthetic continuation covers weeks of drift and months of adoption without accelerating a real user's birds.

### 12.1 Required automated and experiential tests

| Area | Tests that must exist before release |
| --- | --- |
| Authority/persistence | Reject client vector/mood writes; property-test all trait deltas nonnegative; demonstrate cursor and vector update atomicity; migration/restart/restore preserve ID, vector, filters and voice. Missing vectors fail explicitly. |
| Time and ordering | Same accepted event history under one-minute versus subdivided/expedited passes produces equivalent drift within numerical tolerance; duplicate events, worker retry, lock lease expiry and crash-after-commit do not double apply. Test daylight saving, clock skew and long outage catch-up. |
| Presence | Exercise all eight visibility/focus/recent-activity combinations, activity-window boundary, no-activity navigation, hidden/blur/pagehide, keyboard-only input, trusted versus synthetic events, pointer/touch movement, sleep gaps, offline replays, duplicate segments, overlaps and late delivery around the watermark. |
| Multiple devices | Overlapping presence contributes a union, listen-in total stays bounded, concurrent offers respect one cooldown, concurrent adoption remains at seven, stale name/settings edits conflict, returned snapshots never decrease revision. |
| Product behavior | One initial greeter and staggered responses; absence length changes the gesture distribution; no welcome/away-time text; birds retain mood through re-entry; no distress or declining traits after absence; settle undo within five seconds and ordinary close have no penalty. |
| Notebook/adoption | Sparse true observations, no session-log substitutes, no user-frequency wording, no trait numbers; stable pagination to oldest entries; name changes preserve history; age thresholds ignore visit count/offers; retries cannot duplicate starters or new birds. |
| Audio/captions | Deterministic score/caption agreement, variation and stable identity, nonzero other-bird gain during listen-in, gradual ramps, bounded voices/nodes, no downloaded recordings, missing/interrupted WebAudio and autoplay denial. |
| Accessibility/layout | Manual VoiceOver/Safari and NVDA/Firefox or Chrome passes; full keyboard/touch paths; no excessive live announcements; normal/reduced-motion equivalents; seven birds/captions remain visible at narrow widths; contrast in morning, rain, dusk and night; focus retained on snapshot update. |
| Visits/auth | Magic-link replay/expiry/scanner handling; unused-invite expiry, concurrent redemption, active revocation on the next pull including `304`, offline visitor lease, no visitor event writes/greeting/notebook access, default-off notifications and deduplicated opted-in mail. |
| Lifecycle/privacy | Consistent export includes specified fields only and does not leak via mail/caches; email change waits for verification; deleted accounts lose visitor/export access; recover before day 30; erase at day 30 and verify restore cannot revive data. Reject forbidden telemetry fields. |
| Performance/liveness | Real first-bird paint under 500 ms in the defined reference suite, initial JS under 2 MB gzip, thirty-minute frame/memory/audio soak, background render shutdown, server progression with every browser closed, tick-latency alarm exercised by injected delay. |

Use property-based tests for timing/interval arithmetic, deterministically seeded unit tests for the reducer/grammar, database integration tests for transactional boundaries, browser end-to-end tests for user interactions, and real-device/manual review for sound and accessible charm. Do not replace listening or motion-sensitive review with screenshots. Do not test “feels alive” only by asserting a CSS animation exists.

### 12.2 Shipping and bird-count ramp

Begin with synthetic internal aviaries at two, three, five and seven birds. Exercise every species and duplicate-species identities before any outside account is added. Run accelerated histories only on marked synthetic fixtures, never by advancing real accounts or seeding their vectors from a new default.

Next, run a small explicitly recruited browser pilot with two starter birds and the complete v1 account/accessibility/privacy features. Track operational quality only; solicit direct qualitative feedback about recognizing birds, sound, quietness and motion. Use test environments aged synthetically to validate adoption and seven-bird mixing without pretending a new real aviary is months old. A real pilot's growth remains based on its actual creation time.

Expand account admission in controlled cohorts after the first-bird, tick, persistence, leak, accessibility and listening gates pass. Every new owner starts with two. Validate three-bird production eligibility first as real aviaries reach the age threshold, then expand the operationally approved capacity toward five and seven. Any capacity ramp is a temporary quality-control ceiling on new adoption only: never remove, hide, reset or downgrade an already adopted bird. V1 engineering and release qualification still support all seven, and the final supported capacity is seven rather than an indefinitely deferred feature.

Instrument the privacy-safe operational metrics from the first internal build, not after launch. Put feature flags around new invitation issuance, new adoption opportunities and grammar/renderer revisions, with a separate conservative ability to pause drift writes on a demonstrated corruption risk while retaining queued input and stored vectors. A rollback can stop new changes and restore compatible code; it cannot reseed or subtract personality. Keep old grammar/asset versions until migrations are proven identity-preserving. Disabling new invitations must not bypass revocation or orphan active permissions.

Release artifacts should include a versioned tuning configuration, schema migration/rollback procedure, synthetic calibration results, browser/assistive-technology matrix, performance report, listening and reduced-motion review notes, restore/erasure evidence, and an operational runbook. These are outputs of the implementation project, not additional files produced by this planning run.

## 13. Risks and response

| Risk | Prevention, detection, and response |
| --- | --- |
| Drift too fast or too faint | Integrate elapsed time, cap doses, use nonnegative filtering and versioned curves; compare synthetic day 0/7/21 fixtures and blinded qualitative review. Tune future influence rates, never reset existing vectors or collect population bird metrics. |
| Absence accidentally punishes a bird | Property-test monotonic vectors and separate temporary mood/expression from traits. Inspect two-week absence sequences for desaturation, guilt wording and distress. Remove negative couplings, including any mute penalty. |
| Double-counted presence or dropped drift | Canonical interval union, finalized watermark, two integration clocks, per-event idempotency and single transactional cursor. Test concurrent devices and crash/retry boundaries; quarantine corrupt state instead of recreating birds. |
| Dead/canned arrival | Inline a real current pose, preserve action phase, server-author greeting variation and absence signals. Measure first bird/greeting and inspect repeated returns. No welcome text, generic entry sequence or spinner fallback. |
| Audio uncanniness or blurred identity | Constrain grammar/timbre variation, preserve bird voice anchors, bounded chorus overlap and nonzero ambient mix. Require recognition/listening tests through drift and at seven birds; keep a bad voice revision out of rollout. |
| Accessibility becomes a state-list fallback | Share semantic scene/score data while authoring prose and pose cross-fades intentionally. Manual screen-reader/reduced-motion review is a release gate. Test notification cadence and caption truth, not only ARIA presence. |
| First-bird budget fails on real networks | Private edge bootstrap, minimal critical bytes, no blocking framework/audio/fonts. Test cold reference conditions early and continuously; fix architecture before promising the experience. |
| Unobserved tick cost grows excessively | Distribute due times, use indexed batches and O(birds) bounded state; measure schedule delay as well as reducer cost. Scale workers by due backlog. Do not silently switch to client ticking or resume-only simulation. |
| Privacy leaks through operational convenience | Separate roles/schemas, reject unsafe telemetry, strip token URLs and bodies, send only opaque job references. Exercise an automated forbidden-field test and review real sample operational payloads from synthetic fixtures. |
| Invitation grants excessive or stale access | Separate visitor credentials and endpoints, authorize every pull, short visitor freshness lease, atomic one-time redemption/revocation. No shared owner cookies, public caches or optimistic stale visitor playback. |
| Loss of identity during migration or rollback | Version immutable species/voice references and preserve IDs/vectors/filter state. Require restore/migration comparisons before release and retained compatible assets. Never use resetting as repair. |
| Feature creep changes the product | Review every new surface against naturalist/system voice, no gamification, no user-frequency exposure, no distress and no unsolicited notifications. Specific export/visit-email exceptions do not authorize general statistics or messaging. |

V1 is ready when the same durable birds are recognizable across sessions and devices, the first scene already feels in progress, qualified quiet attention changes personality only over weeks, absence is safe, every supported sensory/interaction mode retains the product's character, and the required privacy and operational gates pass. Shipping more controls or more events does not compensate for failure of those conditions.
