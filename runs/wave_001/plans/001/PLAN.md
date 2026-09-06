# Pocket Aviary v1 implementation plan

## 1. Delivery contract and scope

Build a browser-based, private aviary whose birds continue living between visits. The engineering success criterion is continuity of individually recognizable birds over weeks, expressed through motion, calls, and occasional specific observations. The deliverable described here is a product implementation; this phase produces only this plan and the candidate metadata.

Source basis: `prd/1-START_HERE.md`, `product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, and `non_goals.md`. No implementation, existing architecture, external design-system document, or benchmark evaluation material is assumed.

V1 includes:

- One account and one canonical aviary, email magic-link authentication, device sessions, verified email change, export, deletion and recovery.
- Two system-selected starter birds; approximately six coherent species; age-based opportunities to adopt additional birds, with an invariant maximum of seven. Stable bird identities, renameable names, persistent personality, daily-ish mood, procedural calls, and bird-to-bird responses.
- One responsive horizontal scene with front/middle/back perch zones, local-time lighting, occasional weather, ongoing idle motion, and sparse top-bar controls.
- Return-greeting, attentive presence, listen-in, three kinds of offers, optional settle, and a sparse, read-only, indefinitely browsable field notebook.
- Private, individually issued, revocable, read-only visit invitations and an on-demand visit log.
- First-class narration, keyboard access, reduced motion, runtime call captions, and graceful silence when audio cannot run.
- Aggregate operational observability, durable server simulation, migration and restoration procedures, and release gates for accessibility, performance, and continuity.

Explicit exclusions: native apps; payments; passwords or SSO; shared or multiple aviaries; scene customization, placement, panning or zoom; species catalogs or rarity; hunger, death, neglect penalties or distress; personality displays; scores, streaks, levels, counters, badges or achievements; engagement reminders; public profiles, discovery, ranking, feeds, chat, comments, co-presence or mutual-visit systems. No real-weather integration, runtime language-model generation, recorded call assets, or analytics over users' bird interactions. No client simulation or personality merge feature. Notebook browsing and system panels may scroll; the scene may not.

### Product ambiguities resolved for implementation

These are explicit engineering choices, not additional requirements attributed to the PRD.

| Issue | V1 decision |
| --- | --- |
| Numerical personality is categorically hidden, but account export explicitly lists current personality vectors | Treat the specifically requested account portability export as the sole exception: raw vectors are present only in the authenticated downloadable JSON, never in a product panel, narration, ARIA, browser debug view, ordinary snapshot, or telemetry. These two source statements cannot both be satisfied literally; document this exception in the product contract and release review. Do not add a vector viewer or import/editor. |
| Brief refuses notifications; social spec explicitly allows an off-by-default visit-notification setting | Implement that narrowly scoped setting as an optional, quiet email for a newly redeemed invitation. No push, toasts, badges, onboarding prompt, reminders, repeated-visit campaigns, or other aviary emails. Authentication, invitation, requested export, and verified email-change messages are transactional exceptions. |
| Top bar lists four icons, but settle must be reachable from the top bar | Keep four icons: account, accessibility, notebook, and offer. The offer button opens a small action panel containing the three offers and a separated `settle` action. No fifth permanent icon or scene button. |
| Scene forbids labels and chrome, but captions and keyboard focus must be visible | Captions and focus outlines are explicit accessibility exceptions. No permanent nameplates, mood labels, tooltips, or inline action icons. |
| Tick is roughly once per minute, yet greetings and offers need prompt response | Use a once-per-minute scheduled tick plus expedited invocations of the same serialized server simulation function for accepted user actions. Only this function writes personality/mood/action state. Extra invocations cannot accelerate drift because integration uses elapsed time and consumed event offsets. |
| First scene must have sound, but browsers may deny autoplay | Draw the real moving scene immediately; attempt audio only where permitted. Otherwise use silence with captions enabled by default and an accessible audio control in accessibility settings. An eligible user gesture can resume the context. Never promise autoplay the browser disallows. |
| Different devices may report different local timezones | Store one canonical IANA aviary timezone, initially suggested by the onboarding browser. Changes are explicit in settings; another device cannot silently override it. Visitors use the host timezone. This favors one coherent aviary when devices travel. |
| No retained email outside account record, but named invitations need recipient addresses | Maintain recipient identity records in the accounts table, including pending identities without an aviary. Store encrypted email once there; invitation and visit records contain only UUID references. Redeeming a visit does not activate an owner account or create an aviary. |
| Revocation prose mentions disappearance from the visit log, while the log promises who saw what | Remove revoked invitations from outstanding/active rows immediately. Preserve historical visits with a revoked status in the host's private history until account deletion. This keeps sharing transparency without a success notification. |
| Keyboard focus itself engages listen-in, while Enter is also specified | Focus entering a bird engages listen-in; Enter also engages or re-engages it. Escape ends listen-in while retaining focus. Moving focus to another bird starts its mix; moving out ends it. Pointer activation of the same bird toggles it off. |

## 2. Architecture and ownership boundaries

Use a TypeScript web application with server-rendered HTML, compact SVG bird assets, and a small imperative scene runtime. Use a lightweight component layer for top bar, forms, notebook, settings, and dialogs. Render motion outside component reconciliation. A typed domain package contains server simulation rules, serializable visual/audio plans, and shared call-grammar interpretation; client builds exclude persistence, personality integration, and private event-processing code.

Deploy a stateless HTTPS application service, a background simulation worker pool, PostgreSQL as the canonical store and transactional event log, an identity/email delivery adapter, and an isolated export worker with private object storage. Start with one database write region and stateless edge delivery, not distributed multi-writer state. The CDN serves public versioned assets and forwards authenticated document requests to an edge application that fetches the account's current presentation snapshot. Private state must never enter a shared CDN cache.

Boundary:

1. Owner clients authenticate, submit interaction facts, pull presentation snapshots, and render time-indexed plans.
2. The API authorizes every operation, validates events, assigns sequence numbers, and atomically appends to the account-local log.
3. Simulation workers read canonical vectors plus new events, advance time, compute nonnegative deltas, update mood and action schedules, produce notebook candidates, and commit a new state revision in one transaction.
4. Snapshot projection strips private simulation inputs, exposing current positions, mood-shaped pose descriptions, effective appearance, timed transitions, and call recipes. Snapshot construction does not advance simulation.
5. Visitors use a separate capability and route group with presentation-only access. They cannot submit owner events or open private account/notebook data.
6. Observability receives whitelisted numerical operational measurements. It has no database access to simulation, notebook, invitation content, or event tables.

Suggested implementation modules and owning responsibilities:

| Module | Responsibility |
| --- | --- |
| `domain/simulation` | Tick integration, mood transitions, behavior selection, event folding, species definitions, deterministic seeds |
| `domain/presentation` | Snapshot DTOs, action timeline and grammar schema; no raw personality values |
| `services/identity` | Magic links, cookies, email change, device revocation, recipient identity |
| `services/aviary` | Authorization, event append, snapshot projection, naming and adoption transactions |
| `workers/simulation` | Due scheduler, per-aviary serialization, retries, sparse notebook generation |
| `workers/privacy` | Export, recovery, deletion, expiry, object cleanup |
| `web/scene` | SVG composition, animation interpolation, responsive placement, reduced-motion renderer |
| `web/audio` | Grammar interpreter, synthesis, mix ramps, audio clock and captions |
| `web/accessibility` | Semantic controls, prose narration, focus navigation and preferences |
| `web/system` | Sign-in, account/settings, notebook paging, invitations and matter-of-fact errors |
| `quality` | Synthetic fixtures, longitudinal calibration, browser tests, privacy and performance gates |

These are repository responsibilities for the implementing team, not instructions to create those files in this planning phase.

## 3. Persistent model and invariants

Use UUIDs for all accounts, birds, sessions, invitations and entries. Use UTC instants in storage; IANA timezone only for interpreting the aviary's day. Version schemas, simulation parameters, species assets, and grammar libraries independently. Use database constraints, explicit row locks and narrowly granted service roles rather than application convention alone.

| Record | Key fields and rules |
| --- | --- |
| Account | UUID; envelope-encrypted verified email; encrypted pending email; identity lifecycle; settings revision; creation time; deletion timestamps; account encryption-key reference. Pending invitees can exist without an aviary. Email lookup uses an isolated keyed blind index in this table, never as an identifier, partition key, log field or metric. |
| DeviceSession | UUID, account UUID, hashed token, created/last-seen/expiry/revoked times, user-visible coarse device label. Secure HttpOnly SameSite cookies; no bearer token in local storage. |
| MagicLink | UUID, account UUID, purpose, hashed random token, expiry at issuance plus 15 minutes, consumed time. One atomic successful consumption. |
| Aviary | UUID, unique account UUID, creation time, IANA timezone, state revision, tick cursor, processed event sequence, simulation version, PRNG state, weather schedule, mood-day anchor, last owner arrival/departure, settled state/action reference. |
| Bird | UUID, aviary UUID, immutable species ID, name, adoption time, immutable call-signature seed, current personality vector, nonnegative drift-filter state, mood, mood-entered time, mood reset time, perch/slot, active action, offer cooldown expiry. Constraint: all five traits within [0,1]. IDs and vectors survive renames and all upgrades. |
| SpeciesDefinition | Versioned six-species pool; silhouette, palette ranges, pose set, bounded call motifs, night activity weight. One nightjar-like species. Existing birds keep compatible species and signature versions across releases. |
| InteractionEvent | Aviary UUID, ordered sequence, unique event UUID, device/view ID, server received time, bounded client timing, type and validated payload, consumption metadata. Append-only until retention deletion. No client personality, mood, or placement writes. |
| OwnerView | Short-lived view UUID, session UUID, interval sequence, observed presence boundary, lease deadline, listen-in target, settled flag, last acknowledged ping. Device tabs are distinct views. |
| BehaviorPlan | Aviary revision, action IDs, bird IDs, start/end times, source/target perch, pose parameters, greeting seed, call descriptors and weather phases for a rolling horizon. Bounded data, server-authored. |
| NotebookEntry | UUID, aviary UUID, observed time/local date, involved bird references, stored prose, provenance facts, template version. Immutable and private; unique candidate identity prevents duplicate entries. |
| ObservationSummary | Bounded account-local facts needed to support specific notebook observations, e.g. recent first-greeter order. Never visit streaks or cross-account statistics. |
| Invitation | UUID, host UUID, recipient UUID, hashed random capability, created time, unused expiry +30 days, redeemed time, revoked time. No discoverability field. |
| VisitorSession / VisitLog | Invitation UUID, hashed scoped session token, start/last snapshot/end time, approximate duration. No visitor-to-host simulation events. History references recipient identity for host-authorized email display. |
| ExportJob | UUID, account UUID, status, snapshot revision, object key, hashed download credential, expiry, failure code. Private encrypted object and short-lived download. |
| TransactionalOutbox | UUID, delivery purpose, account/invite/job UUID references, delivery status and retry count. Resolve addresses just before sending; do not copy plaintext email into queue payloads. |

Database roles: only the simulation role updates existing bird vectors and mood/action state; adoption may initialize a new vector through a dedicated procedure but cannot replace existing ones. Snapshot/API roles cannot write those columns. All owner queries are scoped by the authenticated account UUID. Export worker has explicit read access unavailable to normal client APIs. Analytics credentials cannot read private tables.

Protect continuity with point-in-time database backups, encrypted storage, transactional migrations, checksums and restore drills. Never regenerate a missing vector from events or a seed: stop that aviary's simulation, report a system error, and restore its record. Initial seeds only create new birds. An internal synthetic fixture may expose vectors for calibration; production user-facing diagnostics may not.

## 4. API and account flows

Use same-origin JSON HTTPS endpoints under `/api/v1`, typed request/response schemas, CSRF protection, strict origin checks and body size/rate limits. UUID-scoped objects always require relationship authorization. Return stable error codes plus matter-of-fact messages in system panels. No global toast component. Mutating requests use idempotency keys. Reject unknown fields so a disguised absolute-vector write is not silently accepted.

| Endpoint | Contract |
| --- | --- |
| `POST /auth/magic-links` | Email and purpose; always generic accepted response. Default limit five requests per email per 15 minutes plus IP abuse control. New requests are possible after the window; no signup enumeration. |
| `POST /auth/magic-links/consume` | Token; atomic consume before cookie issuance; reused/expired token gives the same clear recovery path. Link landing GET does not consume, avoiding mail-scanner redemption. |
| `GET /account`, `PATCH /account/settings` | Settings, timezone and current device information; revision precondition prevents lost settings edits. Audio/accessibility preferences have device overrides. |
| `GET /account/sessions`, `DELETE /account/sessions/{id}` | List/revoke own devices. Every subsequent API call checks revocation; client also clears local runtime on 401. |
| `POST /account/email-change`, `POST /account/email-change/verify` | Verify new address before atomic switch; old address works until then. Use fresh purpose-bound tokens and reject address collisions. |
| `POST /aviary/adoption` | Initial naming of two server-selected birds; idempotent activation. Starter species and default names already shown during onboarding. No species catalog. |
| `GET /aviary/snapshot` | Current revision, server time, timezone, visual scene, bird presentation, call/action timeline, weather, settlement state. ETag/conditional response; owner only. |
| `POST /aviary/events` | Ordered bounded batch; returns per-event accepted/duplicate/rejected status, assigned sequences, latest revision and optional expedited snapshot. |
| `GET /aviary/events/receipts?ids=...` | Reconcile an uncertain acknowledged action without resubmitting a new logical action. Bounded to current owner view. |
| `PATCH /birds/{id}/name` | Validated plain-text name, 1–32 grapheme clusters; revision check. Escape everywhere, support international names, no identity replacement. Bird settings are reached through account settings. |
| `GET /aviary/adoption-opportunity`, `POST /aviary/adoptions` | At most one pending age-eligible opportunity; accept with name and opportunity ID, or quietly defer. Server checks age, prior acceptance and seven-bird cap under lock. |
| `GET /notebook?before=cursor&limit=30` | Stable chronological keyset paging through all entries. No entry mutation API. |
| `POST /invitations`, `GET /invitations`, `DELETE /invitations/{id}` | Explicit named invitation; host-only list and revocation. Sending is the opt-in. Address resolved through recipient identity, never exposed in URLs. |
| `POST /visits/redeem` | Single-use invite token to narrowly scoped visitor cookie; atomic redeem; no owner session. |
| `GET /visits/current/snapshot` | Checks host status, invitation revocation and visitor session every time, including conditional requests. Render-only projection of the canonical scene. |
| `GET /account/visits` | Host-only, recent-first history with recipient email, date, approximate duration and outstanding invites. No badge/unread count. |
| `POST /account/exports` | Enqueue consistent JSON export; email verified address when ready. No vectors returned by this endpoint. |
| `GET /account/exports/{id}/download` | Short-lived capability plus account authentication; checks account status; no-store attachment response. |
| `POST /account/deletion`, `POST /account/recovery` | Confirmed deletion marks account immediately; recovery within 30 days; hard-deleted accounts cannot recover. |

An interaction batch carries `event_id`, `view_id`, monotonic `view_sequence`, `kind`, `observed_at`, and type-specific payload. Allowed kinds are arrival, presence interval/end, listen-in start/end, offer, settle and re-engage. Server derives account and bird ownership. Client timestamps describe intervals but never decide log ordering or simulation time. Offers name a library item, not uploaded media. A receipt's accepted status means queued/committed, while its outcome/action ID confirms an actual simulation response.

Issue 30-day rolling device sessions with a 90-day absolute limit as a v1 default. Show the device list in account settings. A revoked or timed-out session cannot continue to submit queued events. Delete active local snapshots on logout/account switch; never show one account's cached birds to another.

New owner activation atomically creates one aviary and exactly two bird records with distinct initial species where possible. Naming can use defaults so it is not a blocking configuration exercise. The brief first-ever quiet-field-to-soft-fly-in sequence is explicitly marked by a persisted adoption transition. After that, return navigation always renders mid-action; refreshing cannot replay adoption.

## 5. Presence and interaction accounting

### Presence state machine

Set the initial recent-activity window to five minutes, configurable in a versioned simulation configuration. The client counts attention only while all of these are true: `document.visibilityState === 'visible'`, `document.hasFocus()`, and a trusted pointermove or keypress-equivalent keyboard event occurred within the last five minutes. Use trusted `keydown` for modern keyboard compatibility; do not count programmatic events, timers, audio playback, network polling, focus alone, or an open tab. Touch pointer movement can qualify; a stationary tap alone does not invent a pointermove. Verify this behavior on touch and assistive input hardware and adjust the explicit presence contract only if necessary, not silently.

Track state transitions with a monotonic clock. Start an interval only when the conjunction becomes true. Split it immediately on blur, hidden, activity timeout, settle, logout, or pagehide. Pointer movement need not continue throughout the interval: quiet watching is valid until the five-minute window expires. Opening a tab produces an arrival/greeting but no automatic five minutes of presence.

Every 15 seconds, submit the preceding eligible interval, with view sequence, duration, and the activity/visibility/focus eligibility assertions. Send a best-effort terminal beacon on exit; expiry of a server lease handles lost beacons. Store last accepted end and limit an interval to the actually elapsed, server-observed view lifetime; cap heartbeat gap credit at 30 seconds. Never count an entire sleep or disconnected night. A heartbeat contains no key values, pointer coordinates, browser history or keystroke count. Keep raw input events in neither memory history nor storage.

The browser cannot cryptographically prove a person was watching. The server's purpose is to prevent accidental inflation, replay and impossible durations, using session binding, sequences, interval bounds and rate limits. Do not add invasive surveillance or anti-bot engagement mechanics.

Merge overlapping accepted presence intervals across all owner views before tick integration: account-level time is the union, not the sum. Two devices showing the aviary for a minute contribute at most one minute. For listen-in attention, union overlapping intervals for the same bird and cap total account listen-in contribution at the account's accepted presence duration; if devices attend different birds simultaneously, divide the per-time listening allowance among targets. Input activity is never collected from visitors.

Settle terminates the initiating view's presence immediately and propagates the canonical settled scene. Existing other views close their presence intervals when they receive the settled revision; queued intervals are clipped at the authoritative settle timestamp. An explicit new engagement can begin a new interval, subject to the three presence conditions. Tab close and settle both end attention without negative drift; settle adds only a small temporary mood-quieting signal.

When hidden, stop animation frames, normal snapshot polling, caption timers and local audio scheduling. Release or suspend audio resources. The server keeps ticking. A visible but unfocused scene may keep rendering while contributing zero presence. Long frame gaps (initial threshold two seconds) trigger snapshot refresh; a sleep gap never accrues presence retroactively.

### Interaction semantics

- **Listen-in:** The mix responds locally immediately. Record target start/end, but credit only accepted presence overlap. Treat repeated starts for the same target as idempotent; switch closes the old interval first. Loss of focus, escape, empty-scene click, same-bird click, settle and page lifecycle changes end it. Do not let a forgotten focus run all night.
- **Offer:** Top-bar panel exposes seed, a small fixed song-fragment library, and still pool. A gesture offers to the aviary; the server chooses nearby eligible receiving birds using perch, mood and personality. Each bird has a three-minute cooldown across all devices. Approaching, waiting, ignoring, bathing or calling against a motif are valid responses; there is no success score. Cooldown rejection uses calm inline availability wording, not a countdown or a punishment. Accepted offers and nearby eligible offers contribute tiny bounded drift signals; retries and cooldown-blocked spam contribute none.
- **Settle:** Authoritative action stores start time and transition duration (initially six seconds), lowers light/calls, and supplies a small mood bias. Any scene click within five seconds sends re-engage and reverses from the current lighting value. After five seconds, deliberate bird/offer activation also re-engages. Pure mouse movement, a heartbeat or a snapshot pull does not cancel settlement. Closing the initiating view expires its settlement overlay; background mood remains what the server has computed. If a close message is lost, a short lease expires the overlay. Local immediate feedback is a presentation overlay only; it cannot change drift.
- **Arrival:** Every owner visible-return/navigation gets a new arrival ID with a view-local absence estimate and a server-known account absence baseline. Never confuse arrival with presence credit. Coalesce duplicate lifecycle events and near-simultaneous arrivals from multiple views so they do not restart an existing greeting. A later genuine quick return can produce a small glance, without another exaggerated entrance. Visitors never generate arrival events.

## 6. Server simulation and behavior design

### Scheduling and exactly-once effects

Each live aviary is due roughly every 60 seconds, staggered by a stable UUID-derived scheduling offset to avoid minute-boundary spikes. Workers claim due rows with leases and fencing tokens. Lock one aviary row, read its ordered unconsumed event range, and advance from `last_integrated_at` to the server's current instant. Snapshot publication, updated vectors/filter state, moods, action timeline, notebook entries, event cursor, next due time and revision commit together. On crash before commit, replay the same bounded work; after commit, the cursor and unique action IDs prevent reapplication.

Worker delivery may be at least once; state effects are exactly once. Per-aviary event sequence is assigned under the same account-local ordering lock used for admissions. Integrate continuous state up to each event's effective server time, apply the event, then advance to the target time. Expedited action invocations use the same transaction/function and elapsed-time integration; ten user events at one instant cannot become ten minutes of drift or ten mood resets. Queue the ordinary next due time independently of client traffic.

Absent accounts still get scheduled ticks. Do not optimize into client-triggered catch-up only. At larger scale shard by account UUID and batch due work. Capacity planning starts at `active_aviaries / 60` scheduled ticks per second plus action load, with twofold headroom. Soft-deleted accounts are deliberately suspended for the recovery window, since the user has requested deletion; recovery preserves vectors and resumes normal mood/time without invented attention.

Recover missed work in bounded chunks with fairness between accounts. Integrate low-pass state analytically over no-input spans, and step mood/weather at their scheduled boundaries using persisted deterministic PRNG state. Preserve the result of ordered historical inputs. Never replay missed calls or greetings to a returning client. If recovery exceeds the action latency budget, serve the last committed scene with a matter-of-fact inline loading error if needed and disable unavailable mutations; do not fabricate a current vector. Alarm on scheduler lag as well as transaction duration.

### Personality drift

Represent boldness, social warmth, vocal frequency, plumage saturation and curiosity as five stored normalized scalars in [0,1]. Initialize species-informed but individual seed values within roughly [0.2,0.55], with sufficient distance between the two starters. The call signature seed is immutable and separate. Initialization randomness never runs for an existing bird.

For each bird and trait maintain a persisted nonnegative low-pass drive `z`. Over elapsed time `dt` in days, with piecewise-constant bounded positive input `u`, use:

`z_next = exp(-dt / tau) * z + (1 - exp(-dt / tau)) * u`

Integrate the area under `z` over that interval, not merely one sample multiplied by number of ticks. Compute an additive server-authored delta `delta = min(1 - p, k_trait * integral(z))`, then store `p_next = p + max(0, delta)`. Initial filter time constant is three days. Apply explicit per-day and per-session budgets to bound short-session impact. Near the ceiling, taper gains smoothly without ever decreasing the existing value. Persist integration state and values; event history is not the definition of personality.

Build `u` from saturated daily attention with these initial proportions: at least 80% of the allowed total drive comes from account presence distributed to the birds, up to 15% from per-bird listen-in, and at most 5% from eligible offers. Trait-specific weights send listen-in mainly to warmth/vocal frequency, acceptance to curiosity, eligible nearby offers to boldness, and steady presence to all five including saturation. Settle has no personality weight. Cap the presence scale at an initial reference of 20 qualified minutes/day; credit beyond that has sharply diminishing returns. This is hidden calibration, never a visible daily goal. It makes repeated clicking unable to substitute for quiet attention.

When input ceases, `z` decays to zero, but `p` does not. Residual positive drive can continue producing tiny increases during absence, as the PRD explicitly expects. Mood/activity can relax from recent-interaction animation toward ambient behavior without decreasing warmth, vocal frequency, boldness or color. No term depends negatively on days absent. Avoid adding guilt to the greeting selector: a longer absence changes orientation and variation, not trust.

Initial synthetic calibration target, to be tuned through the acceptance process: two ordinary sessions totaling about 15 qualified minutes/day should produce roughly 0.008–0.025 trait change after seven days and 0.035–0.075 after 21 days for midrange starter traits. Set an initial daily trait delta cap of 0.004 and a maximum ten-minute session-associated delta of 0.002. These are test-harness values, not claims about production usage. Map those changes to perceptible accumulated perch proximity, reaction probabilities, plumage and call frequency at three weeks, while a single session remains below a perceptual threshold. No target or numeric value is shown to a user.

Calibrate with deterministic synthetic histories: idle-only eligible visits, active offers, listening, absent weeks, background-only tabs, bursty clicks, overlapping devices and ceiling values. Use a separate consented qualitative study with fixed synthetic aviaries for day-1/day-7/day-21 comparisons. Do not query production vectors, compute population drift averages, or use users' event histories to tune the model. Instrumentation can measure synthetic drift numerically while product telemetry cannot.

### Mood, weather and bird-to-bird behavior

Ship five states: wary, content, curious, drowsy and alert. Use a semi-Markov transition model with minimum dwell periods (initially three to ten minutes) and probabilities informed by local day phase, recent accepted interactions, personality and ambient events. Draw a new day-context bias on a daily-ish cadence (20–28 hours around the local cycle), softening toward it over hours. Never reset mood to neutral on open or at every tick. Late evening shifts most birds toward drowsy; nightjar-like species retain meaningful nighttime call probability. Wary is ordinary watchfulness, never visibly suffering.

Server behavior selection emits time-indexed preen, scan, tilt, shuffle, rest and perch-transfer actions. Personality influences front/back preferences, bird spacing, greeting likelihood and call response; mood chooses posture and tempo. Enforce a perch occupancy map so no two birds inhabit the same slot. Bird-to-bird effects are bounded: another call may receive a staggered answer; a mild alarm increases nearby wariness briefly; content birds may perch near one another. Refractory windows and capped transition probabilities prevent an alarm feedback loop or constant chorus.

Schedule weather with seeded arrival times: initially two to four short rains per week, lasting three to eight minutes, and occasional soft wind. Server stores weather phase and small mood modifiers; rain temporarily attenuates effective call rate, never the personality trait. No thunder, storms, distress or real location request. Decorative individual leaves/feathers are local ornaments and not database entities.

### Greetings and response timing

On owner arrival, the expedited server tick selects one primary bird by weighted warmth, boldness, mood and absence length. Compose its notice from continuous parameters: glance direction, head angle, delay, step distance, pose blending, call motif/pitch contour and timing. Use a persisted per-arrival seed and avoid the previous greeting fingerprint. This is compositional variation, not a three-animation rotation. An optional second bird can respond later with a randomized offset; never cue all birds in unison.

The primary notice should begin 0.4–1.8 seconds after first scene presentation under supported network conditions. It interrupts or blends from the current action at its actual phase; it is not the bird entering the scene. Integrate authenticated initial navigation with an idempotent arrival event so initial HTML can include the server-authored notice when latency allows. On visibility return, request a snapshot and submit the arrival together. Network failure cannot guarantee a two-second authoritative response: keep real ambient motion and handle the system failure plainly, rather than invent client simulation. No textual welcome, absence duration, toast, banner or calendar decoration.

### Call grammar runtime

Each species defines a bounded grammar of motifs consisting of note counts, relative pitch contours, trill/noise components, envelopes, gaps and allowed phrase combinations. Each bird's permanent signature seed sets a stable register, motif preferences and timbre within its species. Mood and personality alter bounded tempo, call frequency, intensity and expressive variation without changing the identity-defining register/contour family. Two same-species birds must still have separable signatures.

The server plans call IDs, timestamps, motif references, response relationships and variation seeds over a rolling 90–120 second horizon. The client expands a recipe into a shared note-event intermediate representation. Audio synthesis and caption generation both consume that exact representation. The client may synthesize continuous oscillator values and interpolate an action; it may not schedule new mood-affecting calls or bird responses beyond the server plan. Expired schedules are dropped after suspension; never play a backlog. If the horizon expires during an outage, complete the current action and use a safe idle pose/ambient ornament loop without pretending to progress the simulation.

Recognizability gates include blind identification of all starter signatures and all seven-bird combinations across moods and day phases. Reduce overlap, motif density or timbral similarity before adding birds if listeners cannot track individuals. Night calls and offered song fragments use the same procedural pipeline; none use recorded audio.

### Notebook generation

Maintain sparse candidate observations within the private simulation. Candidates must be supported by facts: an unusual greeter order compared with this aviary's recent bird observations, a sustained pose in cool weather, a rare exchange of calls, or a quiet interval. Store just the bounded facts required for that account; never a user's attendance calendar. Avoid fabricating events the client could not have seen if describing a user interaction.

Use a curated, compositional prose grammar with reviewed vocabulary, specific bird names, present tense and lowercase rendering. No runtime external language model or sending private state to a prose service. Rank novelty, require a minimum noteworthy threshold, deduplicate similar observations and maintain a long cooldown. Start with one ordinary entry per 48–96 hours for regularly visited aviaries, with at most one extra event-driven entry in 24 hours. Active users must not receive one entry per session. Long absence does not generate a backlog of filler observations.

Keep immutable rendered prose with name-at-observation, so renaming cannot rewrite the past. Retain structured bird references for identity continuity without exposing numbers. Entries never mention trait deltas, visits per week, streaks, reward language, or failed settling. Store every published entry until account deletion; use keyset pagination and accessible bounded rendering, not retention archiving. Notebook entries are neither editable, individually deletable nor annotatable.

## 7. Snapshot synchronization and failure recovery

A presentation snapshot contains `schema_version`, `state_revision`, `server_now`, `simulated_through`, `valid_until`, timezone/day phase, weather, settled transition, bird presentation records and timed action/call plans. Bird presentation records include stable ID/name/species, effective palette, silhouette, position and pose parameters, and current mood as needed for private narration construction; they do not include raw personality, drive/filter history or recent interaction history. Numeric render coordinates and sound frequencies are rendering parameters, not trait values. Do not rename trait fields to disguise numerical exposure.

Visible owner views pull every 15 seconds with jitter and ETags, immediately on returning visibility or a long frame gap, and after a queued action completes. Visible visitor views pull every ten seconds, including an authorization check before any 304 response. Background views stop polling; all returning visitor views reauthorize before revealing cached private content. State transport does not require WebSockets or a client-to-client channel.

Accept only a strictly newer revision for canonical state; equal revisions may update the server-clock estimate. Never let a late HTTP response overwrite a newer scene. Compute a smoothed wall-clock offset from server time and round-trip observations, using a monotonic local render clock; browser clock changes do not jump the birds. Find each action's current phase from its absolute time. Blend minor correction over 300–800ms; a long suspended flight resumes at the current perch/action rather than playing the missed flight. Reduced-motion corrections use the same slow cross-fade surface.

All devices read the same canonical revision and mood; polling may give them different freshness for up to a poll interval, not divergent authoritative states. Listen-in mix and accessibility preferences may differ locally without changing the bird identity or canonical timeline. The server serializes owner events from all devices, so simultaneous offers contend on the same per-bird cooldown and simultaneous adoptions contend on the same cap. Settings/name revision conflicts return the latest value and a matter-of-fact inline retry choice; personality never has a merge dialog.

Retain only bounded unacknowledged action IDs in memory. Retry accepted-intent writes with the same IDs; do not persist days of offline offers or listen-in intervals. Presence intervals older than the short admissible lease window are discarded, not later credited in bulk. On reconnect fetch the current revision, reconcile uncertain receipts, discard expired transient plans and resume valid future actions. If an offer's visible response is already in the past, do not replay it merely because its receipt arrived late.

Failures to sign in, load, authorize or write appear in the relevant system panel in ordinary capitalization: e.g. `Your session timed out. Sign in again to keep watching.` A temporary inability to refresh may keep the last known scene visible briefly with a quiet matter-of-fact status outside the scene. Auth failure, visit revocation, logout or account change clears private rendering immediately. Neither offline mode nor error recovery regenerates a bird or restores stale local personality.

## 8. Frontend scene and startup pipeline

### First frame

Serve authenticated HTML containing the compact current presentation snapshot and inline critical bird SVG/scene CSS from the edge application. Keep the initial render dependency independent of settings, notebook, invitation code, fonts and audio context initialization. Use system fonts. Seed CSS/imperative initial poses from server action timestamps and current clock estimate, including negative animation offsets where appropriate, so a bird is mid-preen and ambient motion is already ongoing in the first painted frame. Hydration adopts the existing nodes and phase without replacing the scene or restarting motion.

A very small bootstrap starts the time-indexed renderer immediately. Rendering the initial static silhouette and animating only after a large framework hydrates is not acceptable. At a slow/cold state response, render only the quiet sky/field with faint ambient cues; no spinner, progress indicator, fake placeholder birds or static-image-to-live fade. First-ever adoption is the one explicit quiet-field then soft-fly-in exception. Treat its persisted transition as completed after its first scheduled period so it cannot become a recurrent entry animation.

Do not fetch privately cached state from a public CDN URL. Authenticated HTML/snapshot responses use private/no-store browser policy; an optional short-lived server projection cache is keyed by synthetic account UUID and revision and read only after auth. Static species assets and code are public immutable CDN assets. Account/session revocation and deletion invalidate all private server caches. A cache miss is a measured cold path, not a reason to insert a staged welcome.

### Composition and motion

Use layered SVG with sky/foliage, back/middle/front perch groups, compact per-bird silhouette and feather detail, and limited foreground ornaments. Draw at most seven birds and a fixed small pool of leaves/feathers. Apply transforms and opacity changes; avoid per-frame DOM tree construction, layout reads or component state updates. Let one requestAnimationFrame coordinator update scene transforms, frame timing and captions. Use a bounded seeded noise function plus scheduled pose keyframes to vary breathing, weight shifts and head angles without obvious repeating cycles. Motion is mood-shaped and personality-derived through server presentation plans.

Interpolate between server-authorized start/end positions with calm arcs; do not invent perch decisions. Introduce slight independent phase and tempo differences so all birds never preen in sync. Foreground/background parallax is ambient and very small, with no pointer-tracking spectacle. Leaf and feather ornaments begin at random phases and intervals, do not affect mood, and have bounded lifetime/pool sizes. Stop rAF while hidden and resume from a fresh snapshot.

Represent perches in normalized scene coordinates with reserved slots and safe bird bounding boxes. On narrow screens compress horizontal spacing while allocating different slots across the three zones; scale the full composition uniformly within the available scene rectangle, using letterboxing where needed rather than cropping. All birds and their transition extents remain inside the safe area. Target 320 CSS px width through wide desktop; account for mobile safe areas, landscape height and browser dynamic toolbar. Keep the top bar outside the scene. At 200% text zoom, allow system panels to reflow and scroll without making the aviary a panning surface. Render targets for bird interaction stay at least 44 CSS px where possible; use spatial separation and accessible arrow navigation when small visual silhouettes would overlap.

Timezone-derived day/night palette interpolates continuously. Morning warms, midday brightens, evening warms/dims, full night retains enough silhouette separation and permits the nightjar-like activity. Settle overlays a reversible six-second evening blend; it does not change timezone or permanently advance the astronomical day. Weather adds restrained rain/wind layers and temporary effective activity changes; no dramatic effects.

### Chrome and input

The four-item top bar uses accessible names and restrained icons. After four seconds without cursor/keyboard activity it fades near-transparent only when no item is focused, hovered, expanded, or touched. Cursor motion, keyboard activity and touch restore opacity immediately. Keep keyboard-focused controls fully visible and all focused/open-panel text at full compliant contrast. On touch-only devices keep a reliably discoverable minimum opacity; the first tap reveals controls without accidentally issuing an offer. No interactive controls are hidden from the accessibility tree because of visual fade.

Tab traverses top-bar controls and one roving-tabindex entry into the bird group. Arrow keys move among birds in stable spatial order; preserve focused bird identity during motion, rename or snapshot updates. Enter invokes listen-in; Escape ends it and closes an open panel before returning focus appropriately. Leaving the bird group by Tab restores ambient mixing. The offer panel supports all actions without pointer precision; expose a documented modified shortcut, initially Alt+Shift+O where browser/OS permits, plus the ordinary Tab/Enter route. Do not install global single-letter shortcuts by default.

## 9. Audio, captions and narration

### WebAudio pipeline

Use one AudioContext per tab, one bounded per-bird gain/panner chain, a master gain and a gentle limiter. Expand each current call recipe into a compact note schedule: oscillator frequency/envelope, filtered noise components where required, articulation, gaps and timbre. Reuse procedural noise/wavetable buffers. Oscillators are one-shot nodes with explicit `stop`/`disconnect` on completion; any AudioWorklet optimization must preserve the same synthesis contract and have a native-node fallback. No recorded source or recorded fallback ships.

Schedule 100–200ms ahead using the audio clock; refresh the scheduler around every 25–50ms only while audio runs. Cap simultaneous synthesis voices initially at 24 for seven birds; decorrelate seeds and timings, cap chorus density, reserve headroom and avoid hard clipping. A secondary bird response has its own envelope and timing, not a duplicate loop. Stereo position reflects perch location subtly, with a clear mono mix. Loudness remains comfortable across quiet/chorus/offer states; listening should not require repeated volume adjustments.

Listen-in uses a 1.2–2 second smooth gain ramp: target bird moves roughly +3dB relative to ambient, other birds roughly -8dB but remain audible. Restore all birds to ambient on disengage over the same duration. Retarget ramps from current gain values to avoid pops when focus moves rapidly. Keep master headroom; no hard solo, abrupt channel switch, or gain reset. Settle gradually quiets calls. A mute preference affects only playback: no punishment, no decrease in traits, and no assumption that audio-off users were absent. The brief mentions mute/play as behavior but specifies no directional drift; v1 deliberately uses no mute-based personality weight.

On navigation, inspect AudioContext availability and attempt a permitted resume. A suspended, denied, interrupted or failed context changes to silent mode with captions on by default. Do not retry in a hot loop or block scene paint. The audio setting allows a user gesture to retry; on success schedule the next current call rather than replaying elapsed calls. Stop/suspend audio while hidden or on privacy revocation. Device interruption and context state changes are recoverable. No call-download endpoint exists.

### Captions from actual sound structure

The shared note-event representation supplies pitch movement, note count, trill/repetition pattern, intensity and bird/perch reference. Generate short naturalist text from those features, such as `a low trill, paused, low trill again`; never a fixed species caption. The silent fallback runs the same representation/timing, so captions describe the call that would have sounded. A caption lifetime tracks call onset/offset with a short readable tail, normally two to four seconds. Place it near its bird within safe bounds; use collision layout and a restrained opaque backing to preserve text contrast against all skies. Reduced-motion captions do not bounce or track flying paths; anchor and cross-fade them.

Avoid having simultaneous chorus captions overlap or leave the viewport: reserve caption slots by perch and coalesce closely timed phrases while retaining named attribution for ambiguous positions. Do not simply drop the quieter bird's caption. Caption preference is independent of mute. Failure enables captions by default but does not erase an explicit later preference to turn them off. Audio errors belong in accessibility settings, without technical overlays on birds.

### Narration as an authored experience

Generate prose from the same presentation snapshot and action IDs as the renderer, with a separate compositional naturalist grammar. Use a semantic aviary region, named bird controls and one polite, atomic live narration region. Bird accessible names identify the bird and available listen-in action without trait numbers or flat mood state dumps. Provide an on-demand scene-description control in accessibility settings for users who want a current reading.

At idle, issue at most one fresh prose description about every 45 seconds, within the specified 30–60 second range, only if there is something meaningfully different to describe. Examples should mention perch proximity, posture, call character and light, rather than enumerating database fields. User-initiated greeting, offer reaction and settle can produce prompt concise observations through a priority queue. Coalesce rapid interactions, replace stale queued idle prose, cap queue length at two, and pause unsolicited narration while a modal/notebook is being read. Do not use assertive alerts for bird activity. Never announce every animation frame, snapshot or call.

Captions are not each live regions: screen-reader narration already interprets the scene, and double-speaking would swamp the experience. A visually displayed narration panel, when enabled, uses the same readable prose. Keep persisted notebook voice and runtime narration lexicon aligned, but do not turn every narration into a notebook entry.

### Reduced motion and contrast

Resolve motion preference from OS `prefers-reduced-motion` and an explicit accessibility setting; use reduce if either requests it. Changing preferences takes effect without restarting birds. Render still preen/scan/fluff/tilt poses with slow two-to-four-second cross-fades, cross-fade between perch positions instead of flights, remove leaf/feather motion and parallax, and slow lighting transitions to eight-to-twelve seconds. Bird behavior, calls, captions, mood and drift continue. This is a separately art-directed path with its own visual acceptance review, not an animation-disable stylesheet.

Specify design tokens with measured contrast: normal text at least 4.5:1, large text at least 3:1, interactive/focus indicators at least 3:1 against adjoining colors. Keep labels at higher ratios where feasible. The focus outline must work over both night and midday via a two-tone treatment if needed. Verify captions on rain and all palette phases. Test reduced motion, forced colors, keyboard-only, magnification, VoiceOver/Safari and NVDA/Firefox or Chrome. Native semantic forms, headings and dialogs remain the basis for system surfaces.

## 10. Private visits, export and lifecycle

### Invitations and visitor sessions

Only the host's deliberate send action creates an invitation. Generate at least 256 bits of randomness for tokens, store only token hashes, use HTTPS, no-referrer landing pages and no third-party scripts on redemption/export routes. Tokens must not enter request logs. Landing GET displays a simple continuation surface; explicit POST consumes the link so automated email previews do not spend it. Redeem once into a visitor session with an initial two-hour lifetime; an interrupted visitor can resume with that same cookie within its lifetime, but the consumed invitation cannot mint another session. A host sends a new invite for another visit. This duration is a v1 implementation choice.

Visitor snapshots authorize each request against the invitation and host lifecycle. Revocation invalidates the visitor lease immediately server-side and returns `VISIT_UNAVAILABLE` on the next pull; clear visuals/audio on that response. With ten-second pulls, typical visible enforcement is within ten seconds plus network time. Give cached visitor render permission a maximum 30-second lease and stop displaying private scene data when offline beyond that lease. On hidden-to-visible resume, gate display on authorization. Bytes already seen cannot be revoked retroactively; do not claim they can.

The visitor sees the canonical host day/night, weather, bird posture, palette and calls, with the same renderer. No greeting, offer, listen-in, settle, host notebook, settings mutation, bird focus action or presence hook is instantiated. Visitor-local accessibility/audio preferences are allowed because they alter access, not the host simulation. Visitor snapshot pulls update only a coarse visit-session lifetime record. Compute approximate duration from first and last authorized pulls, close after timeout, and do not collect coordinates, gaze, input streams or co-presence. The host receives no visitor marker, live overlay or other scene change.

Log visits silently in account settings, newest first. Show authorized recipient address by resolving its account identity, approximate duration, date and outstanding invitations. Turning on optional visit email notifications requires an explicit account setting and sends at most one email per redeemed invite; check the setting at delivery time, cancel pending notification if revoked/deleted, and never notify on every poll. Off remains the default and there is no badge.

### Export

Generate a transactionally consistent snapshot containing birds, stable IDs, names, current personality vectors under the explicit export exception, current moods, all notebook entries and account settings. Include schema/version/timezone and export time; omit magic links, session secrets, recipient identities, raw presence/listen-in history and authentication hashes. Export requests do not create an interaction-history or visit-streak export surface. Large notebooks stream from a stable database snapshot into encrypted private storage rather than exhausting memory.

Email a short-lived download link to the verified address, valid initially for 24 hours and authorized again at download. The JSON is an attachment, not an HTML stats page. Expire and remove the object automatically, cancel jobs on deletion, and do not emit payload fields into logs. Clearly keep vectors out of all other presentation surfaces; there is no export import, restore-from-client or trait edit feature.

### Deletion and recovery

On deletion mark `deletion_requested_at` and `hard_delete_at = +30 days`, revoke visits and all ordinary sessions, cancel outbound visit notifications/export jobs, and stop normal simulation for that account. A new magic-link sign-in during the window yields a restricted recovery session and the matter-of-fact `I changed my mind` action on signed-in system pages. Recovery removes the deletion marker, issues a fresh session and preserves every bird UUID/vector/notebook entry; previously revoked invites and sessions remain revoked. Resume time/day-phase safely without awarding presence for the frozen recovery window.

At the deadline, a resumable deletion worker removes all account-owned birds, vectors, private events, notebook entries, sessions, links, outbox/job references, visit records, blobs and private cache keys. Remove references where the deleted account was a visitor as well as a host, so no recipient-email tombstone remains in another account's log. Delete orphan pending recipient identities once unused expired invitations and required private records are gone. Unused invitation tokens expire after 30 days and cannot be revived.

Retain operational data only in de-identified aggregate form. Any account-correlated error records use UUID rather than email and have a short deletion-aware retention path. Encrypt account-owned content with per-account keys; destruction at hard deletion makes backup remnants unreadable. Define database backup retention no longer than 30 days and a restoration procedure that re-applies deletion manifests before serving traffic. Retain only the minimal temporary UUID deletion manifest needed to prevent resurrecting backups, with no bird or interaction content, and purge it once backup expiry makes it unnecessary. Explain this operational exception to literal immediate removal of every physical backup byte; do not silently promise that backups bypass retention.

## 11. Privacy and observability boundaries

Store interaction events only to drive that user's simulation and bounded account-local observations. Retain processed raw events for seven days for private retry/failure recovery, then remove them; retain unprocessed events until safely consumed, with alarms on age. Persistent vectors and low-pass filter state are the source of truth, so deleting processed events does not reset the bird. Keep notebook entries indefinitely until account deletion. Limit visit history and identity access to its host/recipient authorization paths, never operational analytics.

The telemetry SDK accepts only enumerated measurement names and numeric buckets: request count/status, endpoint-class latency, aggregate anonymous session-duration histograms, first-bird timing, frame-duration buckets, audio-context failure category, tick execution time, due-to-commit lag and queue depth. Exclude event kind/offer item/bird name/species/trait/mood/presence durations/raw URLs/emails/invite tokens/payloads. Client session-duration samples are emitted into broad anonymous bins without persistent client/account IDs or full IP retention. Do not use RUM to measure which bird was listened to.

Account-level service-error diagnosis may use a UUID in a separate restricted, expiring operational log; it contains error code and request ID only, never interaction content. Do not send simulation exceptions with serialized state. Configure reverse proxies, mail adapter, error tracking and tracing to scrub email-bearing request bodies, cookies, token paths and URL query strings. The mail adapter necessarily resolves delivery addresses but must not retain them in application logs or copy per-bird data to delivery providers.

Enforce separation through telemetry schema validation, database role restrictions, network egress controls and negative tests with seeded PII. No warehouse connector may read the simulation database. No per-account state export goes to analytics, recommendation pipelines, third parties or model training. Synthetic performance and calibration accounts are isolated and explicitly labeled; only their deliberately artificial state can support detailed engine diagnostics. The privacy link in account settings names collected aggregate categories and excluded interaction data plainly.

## 12. Performance budgets and operational gates

The PRD's limits are release requirements. The smaller internal budgets below reserve margin rather than treating a 2MB bundle as a sensible target.

| Surface | Budget and validation |
| --- | --- |
| Initial JS | Strictly below 2MB gzip for all JS required by first paint; internal target below 180KB. CI sums the entire critical graph, not just the entry chunk. Lazy-load account, notebook, invitation, export and noncritical accessibility settings code while keeping essential semantics/motion preference detection in the critical path. |
| First bird | Under 500ms from navigation on a documented mid-tier mobile/4G fixture, including cold browser cache and authenticated snapshot delivery. Internal split: edge/network/auth/state about 200ms, critical transfer about 100ms, parse/render about 100ms, leaving 100ms margin. A quiet loading field is not a passing bird. |
| Snapshot | Target under 12KB compressed at two birds, under 24KB at seven including the rolling action/call horizon; public assets are referenced, not embedded repeatedly. Use projection compaction and horizon length tuning if budgets are exceeded. |
| Greeting | One primary bird begins noticing in one to two seconds under the supported performance profile; action endpoint target p95 below 250ms and p99 below 750ms. Measure actual visual notice, not merely HTTP acceptance. |
| Idle frame rate | Sustained 60fps over 30 minutes on a specified five-year-old midrange laptop. Main-thread scene work target p95 below 6ms; total frame p95 within 16.7ms at 60Hz, with fewer than 1% missed frames under the reference test. |
| Memory | No retained-memory growth across a 30-minute session after warmup. Compare post-GC steady-state samples and object counts; require zero monotonic trend beyond measured harness noise and an internal end-baseline envelope of 2MB or 3%, whichever is larger. The envelope is test noise tolerance, not permission for cumulative leaks. |
| Audio | No audible clicks/dropouts in 30-minute seven-bird chorus/focus/settle tests. Bounded contexts, node counts, buffers and scheduled-call queue. Profile audio thread separately from JS heap. |
| Tick | p99 execution latency alarm above five seconds; additionally alert on p99 due-to-commit lag above five seconds and sustained backlog. A fast transaction that waited two minutes is a failure. |
| UI/accessibility | No render-blocking setting chunks for keyboard/narration basics; minimum contrast/focus requirements and reduced-motion rendering ship in the initial release. |

Define the physical benchmark device, CPU throttling profile, bandwidth/RTT and cold/warm cases in the implementing team's test configuration before optimizing. A representative starting network fixture is 10Mbps down, 1Mbps up and 80ms RTT; test worse profiles separately and report them rather than rewriting the target after measurement. Measure at common geographies using scheduled browser probes. The strict sub-500ms goal may require account-region affinity and edge snapshot delivery; if the cold path fails, fix payload/edge latency before launch. Do not hide failure behind a warm-cache-only result.

Mark first-bird only after a real bird element from the authenticated snapshot is in the painted scene; use frame callbacks plus screenshot assertions in synthetic tests to avoid measuring just DOM insertion. Include font/caption display, late snapshot, reduced motion, WebAudio disabled and seven-bird conditions. Keep RUM timing distributions aggregate-only. Monitor failure rates for snapshot, action, auth and export flows without recording their content.

Memory CI exercises repeated listen-in switches, hundreds of procedural calls, offers, panel open/close, notebook scrolling, caption changes and hide/resume cycles. After each lifecycle, verify bounded listener count, timer count, audio contexts, nodes and scene references. Release render/audio nodes on logout/unmount and return pooled ornaments. Accessible notebook paging must let old entries be fetched indefinitely without retaining all fetched pages: use bounded pages with previous/next access and focus restoration rather than inaccessible infinite virtualization.

Browser support is the last two major releases of Chrome, Safari, Firefox and Edge. Maintain a versioned support test matrix at each release rather than hardcoding today's version numbers into the plan. Required rendering features must be available or produce a clear unsupported-browser page. WebAudio absence alone uses the specified silent captioned path, not a blanket unsupported-browser rejection.

## 13. Verification matrix

Tests protect the product's causal behavior and qualitative character, not only method outputs. Use deterministic clocks/seeds and a fake email adapter for integration tests; real-browser and human sensory checks remain separate release evidence.

| Area | Required evidence before public release |
| --- | --- |
| Personality preservation | Property tests over random event streams prove all trait deltas are nonnegative, vectors stay bounded, absence never reduces values and retries never double-apply. Rename, species-definition upgrade, schema migration, suspend/reconnect and restore preserve UUIDs and vectors. A missing vector fails closed instead of reseeding. |
| Slow calibration | Synthetic day-1/day-7/day-21 histories meet numerical targets; long absence has no negative drift; one long session and click spam cannot produce visible leaps. Qualitative blinded review confirms a week can be subtle and three weeks perceptible. Same tests at low/middle/near-ceiling initial traits. |
| Presence | Exercise all eight truth-table combinations of visible/focused/recent activity. Verify intervals start/end at exact boundaries, expire after the window, stop on settle/close and do not span laptop sleep. Two overlapping devices count union time. Mobile and assistive-input sessions are tested explicitly. |
| Tick correctness | Crash before/after cursor commit, duplicate lease delivery, out-of-order arrival, simultaneous expedited/scheduled runs, DST change and missed scheduler periods. At most one commit per fenced writer; replay yields identical state and notebook/action IDs. Scheduled ticks still advance mood/weather without any client. |
| Multi-device | Two browsers concurrently offer, rename, adopt, listen, settle and reconnect. Vectors are never client-written, old snapshots are ignored, cooldown/cap invariants hold, new sessions receive the same current mood. No last-write-wins personality endpoint exists. |
| Greetings | Capture quick-return and long-absence variants across bird personalities/moods. One initial notice, possible staggered responses, no synchronized cue, no repeated identical fingerprint, no textual greeting. First frame is mid-action after onboarding. |
| Audio/captions | Compare caption descriptors with generated note events; hear signatures across moods and seven-bird chorus in stereo/mono. Test smooth gains on every disengage path, no hard mute of other birds, browser suspension and silence fallback. Verify no recorded media assets/network requests. |
| Accessible charm | Screen-reader participants can follow which bird is doing what without a state dump; idle narration stays in the 30–60s band and queues do not pile up. Reduced-motion users get recognizable changing poses. Keyboard-only completes all owner/system flows. Captions readable in night/rain and at zoom. |
| Scene | Viewport matrix including small phone, landscape, zoom and seven birds; no clipped birds, scene scroll/pan, overlapping hit targets, chrome within the scene or unreadable focus. Thirty-minute visual review catches robotic loops and synchronized micro-motion. |
| Notebook | A month of synthetic sessions produces sparse, fact-supported prose; active sessions cannot create a feed. No duplicate entries after tick replay, no behavioral streak copy, no mutation endpoints, and all old pages remain accessible. |
| Auth/social security | Magic links expire at 15 minutes, consume once and resist concurrent replay; invitation links expire unused at 30 days. Visitor tokens get 403 on every owner mutation and no host notebook. Revocation is checked before cached/304 snapshots and after resume; visitor traffic changes no drift, mood or greeting state. |
| Privacy/lifecycle | Payload canaries never reach telemetry. Export contains the promised snapshot, no secrets or interaction history. Soft deletion recovers within 30 days without replacing birds; deadline purges data/keys/objects, revokes links, cancels mail and survives restore without resurrection. |
| Adoption | Initial activation exactly two birds; age-only eligibility independent of presence/clicks; no catalog/rarity. Concurrent accepts cannot exceed seven. Deferral has no expiry penalty or prompt escalation. |
| Product voice/non-goals | Review rendered UI, emails, captions, prose and error paths. Naturalist product language; matter-of-fact system language. No stats, streaks, welcome, badges, guilt, co-presence or hidden ranking metrics. |
| Performance | Critical bundle, cold/warm first-bird, 30-minute frame/memory/audio and tick-load reports pass the budgets above in the supported browser matrix. |

No synthetic automated test can establish that audio is pleasant or a bird feels alive. Include an explicit sound-design and accessibility review with real listening/reading/motion experiences; do not equate successful snapshots with experiential approval. Use fixed synthetic aviaries and consented feedback rather than mining real interaction history.

## 14. Implementation sequence and rollout

### Milestone A — Contracts and two-bird vertical slice

Agree on the explicit ambiguity decisions in section 1 as the working v1 contract. Produce the initial six-species silhouette/call/pose definitions and contrast tokens within the product work, because the referenced designer document is not supplied. Build the database ownership constraints, typed snapshot/event schemas, deterministic fake clock and a two-bird scheduled tick. Deliver a real authenticated snapshot rendered mid-action with one procedural call, matching caption, keyboard bird access and reduced-motion equivalent.

Exit: a closed browser's aviary advances on the server; reopening never resets mood or personality; first-frame and audio-start feasibility are measured on the target hardware. If the <500ms architecture is not feasible, address it now before expanding the feature set.

### Milestone B — Identity, durability and interaction correctness

Implement magic links/device sessions, initial adoption, names, accurate presence intervals, event deduplication, account union accounting and expedited ticks. Add greeting, listen-in, offers/cooldown, settle/undo and clock/snapshot recovery. Establish backup/restore and privacy-safe operational metrics from the first persistent account.

Exit: two concurrent clients pass continuity and crash-retry tests; zero client personality mutation; truth-table presence tests pass. Soft deletion/recovery and secure export scaffolding work before storing long-lived participant accounts.

### Milestone C — Character and accessible experience

Complete six species, individual signatures, mood-shaped motion, bounded chorus/bird responses, daily lighting, rare weather and full narration. Build sparse notebook generation and indefinite paging. Calibrate slow drift with synthetic histories and day-21 fixtures; conduct the qualitative sound/narration/reduced-motion reviews. Complete account/email/export/deletion flows and settings voice.

Exit: day-1/day-21 contrast is perceptible without numerical UI; absent birds remain themselves; accessible surfaces preserve specificity and calm. Seven-bird synthetic scenes already pass layout, audio and memory checks even while actual new accounts begin with two.

### Milestone D — Visits and full release qualification

Implement explicit invitations, recipient identity, read-only visitor capability, revocation, expiry, visit history and narrow notification setting. Finish security/PII tests, real browsers, international names, DST/timezone handling, error surfaces, CDN/privacy configuration, 30-minute soaks and load/restore exercises. Audit all non-goals in visible copy and backend schemas.

Exit: full v1 feature set, privacy boundaries and performance gates pass together. Accessibility is not a post-launch follow-up. No public rollout until deletion/export/revocation and backup restoration work.

### Milestone E — Staged release and bird growth

Release to internal synthetic accounts first, then a small invited owner cohort, then progressively larger account-UUID cohorts (initially 1%, 10%, 50%, 100%) with at least 48 hours of operational observation at each step. Use only operational error/performance aggregates for ramp decisions. Monitor tick p99, backlog, first-bird distribution, render/audio errors, auth failures and export/deletion completion. Do not instrument retention funnels, visit streaks, offer frequency, popular species or average production drift.

All new aviaries start with two birds. Adopt this initial calendar-age schedule: eligibility for birds three through seven at 90, 180, 270, 365 and 540 days after aviary creation. A year-old aviary can therefore have six birds, matching the intended pacing. An opportunity is quiet text within the owner-opened account/bird settings flow, never a toast, badge, email or countdown. On acceptance, the server picks the species from the same pool, shows a default name and allows renaming. Prefer species diversity for recognizability without rarity. Show only one opportunity at a time; do not issue a return-from-absence stack. Unaccepted opportunities remain available. Additional adoptions require distinct owner actions, and existing birds are never replaced.

Roll out rendering/mixer support at two, then three/four, then five/seven birds in internal and consented synthetic-age fixtures before those calendar cohorts become eligible. Production eligibility remains age-based and never depends on engagement or a paid tier. Feature flags may pause new adoption availability for a faulty renderer but must never delete, hide or downgrade birds already adopted. Keep the hard seven-bird constraint in the database procedure, API and engine.

For deployment rollback, retain backwards-compatible snapshot readers and compatible grammar/asset versions; use expand/contract schema changes. Never roll back canonical bird data to an older backup merely to roll back code. A bad drift release should first disable new positive drift integration while preserving current vectors, maintain mood/ambient ticks if safe, and be repaired forward. Incorrect negative drift is a critical continuity incident requiring restoration of known canonical values. Do not silently reduce an over-expressive bird to meet new tuning: monotonic continuity remains the rule. Record version provenance privately so migration mistakes can be diagnosed without exporting user state to telemetry.

## 15. Principal risks and response

| Risk | Detection and mitigation |
| --- | --- |
| Drift too fast, too slow or dominated by clicking | Synthetic longitudinal matrix, daily/session bounds, monotonic property tests and blinded day-21 review. Version weights; tune before broad release. Do not calibrate from population bird data. |
| Loss or double-application of personality | Per-aviary fenced transactions, atomic event cursor, restricted database writers, idempotent deltas, backup checksums and restoration drills. Missing vectors cause repair, never reseeding. |
| Presence inflation or exclusion | Conjunction truth table, interval union, sleep-gap clipping, trusted events and touch/assistive input review. Five-minute inactivity window preserves stationary watching while preventing overnight credit. |
| Calls sound synthetic, repetitive or indistinguishable | Species and signature design before library expansion, shared grammar structure with controlled continuous variation, bounded chorus overlap, seven-bird blind listening and mono tests. Preserve signature seeds through updates. |
| Stale snapshots, action lag or clock jumps | Revision gating, expedited server ticks, measured clock offsets, bounded horizon, immediate visibility refresh and no stale action replay. Distinguish freshness from client-side authority. |
| Initial scene reads as loading software | Inline authenticated snapshot/critical SVG, phase-correct motion before hydration, aggressive critical byte targets and cold-path timing. Quiet field is only a failure fallback, not a budget exemption. |
| Accessible version loses charm | Authored narration grammar, reduced-motion art direction, real screen-reader/motion-sensitive reviews and runtime caption equivalence. Block release on affected surfaces. |
| Email/tokens/private state leak through infrastructure | Account UUID identity, encryption, body/URL redaction, outbox references, private cache policy and negative telemetry tests. No analytics database reader. |
| Visitor subtly becomes a second participant | Separate routes/capability and client composition; no owner event or greeting hook; tests comparing host state before/after visitor-only sessions. Revocation gates every pull and return from hidden. |
| All-account ticking grows costly | Stagger/shard due work, batch reads, bounded action horizons, analytical no-input filter integration and load tests with headroom. Preserve scheduled advancement for absent aviaries. |
| Existing birds change identity after art/audio upgrade | Immutable IDs/signature seeds, compatible species versions, snapshot migration fixtures and before/after listening review. No regenerate/reset migration. |
| Product grows gamification or announcing UI through routine additions | No general toast/badge/streak infrastructure; product voice review and explicit negative acceptance cases. Use operational health goals, not engagement targets. |
| Literal PRD conflicts cause inconsistent implementations | Keep section 1 decisions in the engineering contract, particularly vector export and visit emails. Build one bounded behavior and test it; do not let each client independently interpret the conflict. |

Completion means the implementation team can demonstrate the full two-bird everyday experience and seven-bird capacity on the supported web surfaces, preserve the same birds across devices and absence, satisfy the privacy/lifecycle contracts, and pass both operational and experiential gates. This plan stops before implementation or benchmark evaluation.
