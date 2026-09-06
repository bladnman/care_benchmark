# Pocket Aviary — v1 implementation plan

Planning slot: wave_003 / 001. This is an implementation specification, not a product implementation. It is based only on `prd/1-START_HERE.md` and the nine PRDs it names. Proposed numerical constants below are initial engineering decisions with explicit validation gates, not claims of measured performance or completed calibration.

## 1. Product contract and release scope

Ship a private, browser-based aviary that appears to have been continuing before the viewer arrived. A new account meets two birds; their identities and slowly evolving personalities survive sessions, devices, deployments, and migrations. Watching counts as an interaction when the specified presence conditions hold. Absence never subtracts personality, damages birds, or creates an obligation to return.

V1 includes:

- Email magic-link accounts, one canonical aviary per account, device-session management, verified email changes, export, and recoverable deletion.
- Approximately six coherent species, including a nightjar-like species, two system-chosen starters, naming and renaming, and optional age-based adoption up to seven birds.
- Server simulation throughout the day and during absence; persistent personality, mood, perch choice, weather, and bird-to-bird behavior.
- Procedural greetings, listen-in, three kinds of offer, settle and its undo, precise presence accounting, and a sparse, indefinitely browsable field notebook.
- A single responsive horizontal scene, three depth zones, local-time lighting, restrained weather, ambient ornaments, and a fading top bar.
- Procedural WebAudio calls, recognizable individual signatures, chorus mixing, captions, naturalist screen-reader narration, and a designed reduced-motion surface.
- Individually issued, emailed, expiring, revocable invitations to a render-only visit, plus an on-demand visit log.
- Operational observability that cannot read private simulation inputs, synthetic calibration, and launch performance/accessibility gates.

Do not build native applications, password/SSO login, payments, subscriptions, multiple aviaries, shared ownership, configurable scenes, draggable birds, scene navigation, public discovery, feeds, profiles, follows, comments, chat, visitor avatars, or co-presence. No hunger, death, distress, declining happiness, scores, achievements, streaks, counters, tiers, competitive rankings, or return reminders. Do not compute hidden engagement rankings in anticipation of a future feature. No user-facing personality dashboard, raw mood badge, loading spinner, welcome message, or milestone announcement.

The two registers are part of acceptance: product observations and prompts use specific, lowercase naturalist prose; identity, errors, settings, privacy, and account operations use clear ordinary system language. Accessible observations are part of the product voice, while accessibility controls are system voice.

## 2. Explicit decisions where the PRDs leave gaps or conflict

These decisions allow an engineering team to proceed without inventing incompatible behavior in separate components.

| Topic | Implementation decision and reason |
| --- | --- |
| Numerical vectors versus account export | `accounts_sync.md` explicitly requires current personality vectors in a JSON export, while `bird_engine.md` prohibits numerical exposure anywhere. These cannot both be satisfied literally. Treat an explicitly requested private machine-readable account export as the sole portability exception: include the exact vectors in that file, but never in normal snapshots, UI, ARIA, captions, notebook, browser diagnostics, or analytics. Explain the exception in the export contract and record it as a deliberate departure from the absolute wording, not an accidental debug path. |
| Visit notifications versus no notifications | The social PRD explicitly permits a settings toggle. Implement a narrow, off-by-default, opt-in visit email; never push notifications, toasts, badges, onboarding prompts, reminders, or auto-invites. This is an explicit exception to the brief's broader prohibition on aviary emails. Auth links, invitation delivery, and requested export delivery are transactional system mail. |
| Four top-bar icons versus settle from the top bar | Retain exactly account/settings, accessibility, notebook, and offer icons. The offer icon opens a small action menu with the three offers and, separated, `settle`. Settle is thus reachable from the top bar without a fifth icon or scene overlay. |
| Keyboard focus versus Enter for listen-in | Owner bird focus starts listen-in; Enter ensures it is engaged and never accidentally toggles it off. Escape disengages while retaining focus; Enter can then re-engage. Pointer activation on the already listened-to bird toggles off. Moving to another bird transfers the mix; leaving the scene restores ambient. |
| One timezone versus traveling devices | Store one canonical IANA timezone on the account, initially suggested by the first browser. All devices and visitors see this timezone. Travel changes require an explicit settings update; later devices do not silently rewrite it. This preserves one shared day/night state. |
| Settle and multiple devices | Settle is an account-wide presentation override, owned by the initiating view's lifetime, with a small canonical mood influence at the next tick. It closes existing owner presence/listen windows. Any explicit owner re-engagement cancels it; visitor activity never does. Detailed ordering and expiry are defined below. |
| Sub-second reactions versus a minute tick | The command service immediately appends server-authored presentation decisions. Clients render those accepted decisions promptly. Only the minute tick writes canonical bird personality/mood. Both devices read the same ordered decisions; no client predicts personality or independently chooses the offer outcome. |
| Invitation recipient email stored once | An invitation targets a synthetic account-identity UUID. An unregistered recipient has a pending identity account row containing the encrypted email but no aviary or owner session. Invitation delivery never silently signs that person up for an aviary. |
| Additional-bird timing | Initial eligibility ages are 90, 180, 270, 365, and 540 days for birds three through seven. Eligibility does not expire, depend on activity, or require accepting previous offers on time. These are pacing constants to validate with product review, not reward milestones. |

The vector/export conflict is a material product risk: unlike the other gaps, the literal rules are incompatible. Implement the exception in an isolated serializer so a later decision can change export representation without exposing vectors elsewhere. Do not claim this resolves the philosophical contradiction.

## 3. Architecture and ownership boundaries

Use a TypeScript web application with server-rendered HTML/SVG, a small imperative scene runtime, and code-split component islands for forms, settings, notebook, and invitation administration. Use a relational PostgreSQL primary as the sole transactional source of truth. Begin with a modular application service and a separately scalable simulation worker process, not a mesh of microservices. A durable job mechanism and transactional outbox support ticks, mail, exports, and deletion. Static versioned assets live behind a CDN; private export objects live in encrypted object storage.

Logical modules and future implementation ownership:

| Module | Owns | Must not own |
| --- | --- | --- |
| Identity/account service | Magic links, sessions, account UUIDs, email encryption, settings, lifecycle | Bird decisions, personality changes |
| Aviary command service | Authentication, event ordering, idempotency, cooldown reservations, presentation receipts | Applying personality deltas or autonomous mood progression |
| Simulation worker | Tick scheduling, personality update, mood/weather/perches, timelines, notebook evidence | Client frame loops, sending engagement data to telemetry |
| Snapshot compiler | Owner/visitor projections, short behavior timelines, initial SVG/narration data | Exposing vectors or inventing local canonical state |
| Scene runtime | Sampling server timelines, interpolation, SVG updates, hit geometry, ornaments | Tick execution, choice of canonical mood/perch, presence inference from rendering |
| Audio/narration runtime | Procedural synthesis from call descriptors, gain ramps, captions, prose queue | Downloading recorded calls or changing simulated vocal frequency |
| Visit service | One-time grants, render-only authorization, revocation, visit log | Host presence, greetings, interactions, notebook mutations |
| Operations collector | Allowlisted timings, counts, errors, coarse hardware/browser buckets | Query access to the simulation database or identifiers in aggregate metrics |

Data flow:

1. An authenticated navigation obtains a compact private snapshot and initial SVG through an edge front door. The snapshot is embedded in the HTML so no client waterfall blocks the first bird.
2. The browser samples server-authored timelines against an estimated server clock, draws the birds in their current action phase, and starts independent ambient ornaments. It requests later snapshots by pull.
3. Owner commands append ordered events and, where needed, immediate presentation receipts. A command receipt can schedule a greeting or reaction now without mutating a personality vector.
4. A leased worker advances each aviary about every 60 seconds, including when no clients are connected. It atomically consumes events, adds personality deltas, transitions moods, and publishes a new canonical revision and timeline.
5. Owner and visitor snapshots come from the same canonical row plus committed presentation receipts. Local audio preferences and listen-in gains affect only the viewing device.

Keep canonical reads on the writer or an explicitly revision-fenced read path; an unconstrained read replica can return a state older than a just-accepted command. Edge acceleration must not weaken authorization: no shared caching of personalized HTML, snapshots, or visited aviaries. Static assets are public and immutable; private responses use `Cache-Control: private, no-store`. If an edge snapshot cache is introduced, it is service-side, keyed by synthetic aviary UUID and revision, with authorization on every request and no public CDN cache entry.

## 4. Durable data model

Use UUID primary keys, UTC instants, versioned schemas, explicit foreign keys, and application-level authorization on every aggregate root. Timezone conversion is presentation/circadian logic, not a change to persisted event time. Only the simulation database role may update personality columns after initial bird creation. Use integer millionths plus fractional remainder for vector arithmetic so repeated minute-sized increments are not lost to rounding.

| Record | Required fields and invariants |
| --- | --- |
| `accounts` | `account_id`, encrypted verified email, restricted blind lookup index, status (`pending_identity`, `active`, `deletion_pending`), created/verified timestamps, canonical timezone, deletion deadline, consent/settings version. Email is stored only here, never as a relational key. |
| `account_settings` | Account UUID; call-caption, narration and reduced-motion preferences; visit-email opt-in default false; version for conditional writes. Distinguish account defaults from local device audio capability. |
| `auth_tokens` | Token digest, account UUID, purpose, creation/expiry/consumption timestamps, pending new-email encrypted payload when relevant. Single use, 15-minute expiry for magic links. Email-change temporary payload is the explicit short-lived verification exception to normal email storage. |
| `device_sessions` | Session UUID, account UUID, token digest, created/last-used/expiry/revoked timestamps, coarse device label. Never raw session tokens or detailed fingerprinting. |
| `aviaries` | Aviary UUID, unique active account FK, created time, `state_revision`, `last_tick_at`, `next_tick_at`, `last_consumed_event_seq`, simulation version, weather/circadian state, deterministic RNG state, presentation revision. |
| `birds` | Stable bird UUID, aviary FK, species/version, current name, adopted timestamp, seed vector, current five-trait vector, fractional remainders, five filter states, current mood, mood-entered/due times, perch zone/slot, behavior timeline seed, immutable call-identity seed and signature parameters. Seed vector is historical identity data, never a reset source. |
| `interaction_events` | Aviary UUID + monotonically assigned sequence, event UUID, device/view UUID, server receipt/effective times, kind, validated minimal payload, schema version. Append-only during retention; unique event UUID per aviary. No raw pointer coordinates, key contents, or email. |
| `presence_intervals` | Account/aviary, source event sequence, view UUID, bounded effective start/end and evidence flags. Private simulation input; union intervals across devices before scoring time. Expire after consumption/retention. |
| `command_receipts` | Event UUID, result/status, sequence, associated state/presentation revisions, chosen server outcome, cue IDs, effective timestamps. Durable idempotency response for uncertain retries. |
| `presentation_actions` | Ordered greeting/offer/settle/resume cues; decision seed, target birds, animation/call descriptors, starts/ends, supersession/undo links. Short lived, authoritative presentation layer shared by snapshots. |
| `interaction_guards` | Per-bird offer reservation expiry; account arrival coalescing state; current settle epoch and initiating display lease. Updated under the same aviary lock as event acceptance. |
| `bird_observation_summaries` | Private bounded facts needed for notebook prose, e.g. greeting order for seven local days, extended preening, recent call motifs. This is simulation-local evidence, never population analysis. |
| `notebook_entries` | Stable ID, aviary UUID, created time/local date, ordered cursor key, generated prose, template version, minimal evidence references and name-at-observation. Immutable, retained until account deletion. |
| `adoption_offers` | Aviary UUID, eligibility age, predetermined species and seed, available/accepted/deferred state, accepted bird UUID. Persist the offer so refresh never rerolls it. |
| `visit_invitations` | Invitation UUID, host account/aviary UUID, recipient identity UUID, token digest, issued/unused expiry, consumed/revoked timestamps. No email duplication. |
| `visit_sessions` | Grant UUID, invitation UUID, token digest, created/last-pull/expiry/revoked timestamps. Dedicated render-only scope, never an owner session. |
| `visit_log` | Host UUID, invitation/recipient UUIDs, visit start and approximate end/duration, current grant status. Email resolved from identity at display time. No bird-interaction fields. |
| `jobs/outbox` | Job UUID, owner UUID where needed, type, scheduling/retry fields, references to private records. No email or full simulation snapshots in queue payloads. |
| `export_jobs` | Account UUID, consistent snapshot revision, status, encrypted object reference, download-token digest and expiry. Private transient data with lifecycle deletion. |

Supporting indexes: due ticks by shard/time; event `(aviary_id, seq)`; notebook `(aviary_id, created_at, entry_id)`; token digests; active session by account; invitations by host/status; deletion deadline. Enforce one aviary per activated account with a unique constraint. Serialize adoption through the aviary row lock and reject an eighth bird at the database/service boundary. Restrict deletion of live bird rows to whole-account lifecycle processing; v1 has no release/remove-bird command.

Names accept 1–40 Unicode grapheme clusters, are normalized for storage, escaped at every HTML boundary, and cannot introduce markup. Names are not keys. Renaming preserves species, identity, call seed, personality, mood, and notebook history. Old notebook prose retains the name that was observed; future prose uses the new name. Do not add a name uniqueness constraint that would force identity through naming.

Version migrations must preserve bird UUIDs and exact current vectors. Migrate stored values forward transactionally; never recreate birds from event history or a new seed algorithm. Maintain a fixture containing renamed birds, nonzero drift remainders, active mood, and old grammar versions through every migration.

## 5. API contracts and authorization

Use same-origin HTTPS JSON APIs and Secure, HttpOnly cookies. Owner commands require CSRF protection and origin validation. Visitor credentials are distinct and cannot authenticate owner routes. Validate schemas strictly; reject absolute personality values, raw mood setters, caller-selected perches, and unsupported fields. Return a matter-of-fact error code and safe message without reflecting tokens or identities.

### Account and data lifecycle endpoints

| Method/path | Contract |
| --- | --- |
| `POST /api/auth/magic-links` | Accept email; always return a generic accepted response. Limit initially to five requests/email/15 minutes plus coarse abuse controls. New requests do not extend already issued token expiries. |
| `POST /api/auth/magic-links/consume` | Atomically validate digest, expiry, purpose, and unused status, consume once, issue a device session. Link landing GET does not consume, protecting against email scanners. |
| `GET /api/account` | Safe account/settings/session summary for its owner only; never raw vectors. |
| `GET /api/account/sessions` / `DELETE /api/account/sessions/{id}` | List coarse devices and revoke immediately, including the current one. |
| `PATCH /api/account/settings` | Conditional write using settings revision; validate timezone and preferences. A stale settings edit gets 409 and a reload action, never a personality merge UI. |
| `POST /api/account/email-change` / `POST /api/account/email-change/verify` | Verify the new address before atomically replacing the old encrypted email/index. Old sign-in remains valid until verification; a uniqueness race fails clearly. |
| `POST /api/account/exports` | Explicit request creates idempotent export job; deliver download link to the current verified address. |
| `GET /api/account/exports/{id}/download` | Short-lived, owner-authorized/token-bound private JSON download; no analytics or token-bearing referrers. |
| `DELETE /api/account` / `POST /api/account/recover` | Mark deletion with exact 30-day deadline; restore before it under lock. Recovery requires fresh identity verification. |

### Aviary endpoints

| Method/path | Contract |
| --- | --- |
| `GET /api/aviary/snapshot?after_revision=…&after_presentation=…` | Return canonical snapshot plus active/recent action receipts. Revisions are monotonic; 304 is allowed when neither revision changed, but authorization still runs. |
| `POST /api/aviary/events` | Accept a bounded batch with unique `event_id`, view ID, event kind, client monotonic timing, and minimal event payload. Server adds account/aviary/sequence/time. Kinds include arrival, presence segment/end, listen start/end, offer, settle, resume, and explicit audio preference change. |
| `GET /api/aviary/events/{event_id}` | Retrieve accepted/rejected receipt after an uncertain network response. Same event ID and payload returns the same result; same ID with different payload returns 409. |
| `GET /api/aviary/notebook?cursor=…&limit=30` | Stable keyset pagination from newest to oldest; no edit/delete/annotate routes. |
| `GET /api/aviary/adoption-offer` / `POST /api/aviary/adoptions` | Show only an available age-based offer; accept it idempotently under the count lock. No catalog, count badge, countdown, or visit threshold. |
| `PATCH /api/aviary/birds/{id}/name` | Owner rename with name revision; immutable bird identity and simulated fields cannot be patched. |

An event response includes `event_id`, `seq`, `accepted_at`, `state_revision`, `presentation_revision`, `status`, optional `cue`, and optional retry guidance. A returned acceptance means durable commit, not that the next tick has already applied drift. Clients do not optimistically display a bird accepting an offer before receiving the server's decision. Normal offer acceptance targets under 300 ms at the regional service; visual offer placement can begin immediately without inventing the bird's response.

The snapshot contains `schema_version`, `simulation_version`, `state_revision`, `presentation_revision`, `server_now`, `simulated_through`, timeline validity, canonical timezone/light/weather, and per-bird identity/name/species/mood, perch coordinates, active action phase, compiled motion cues, call identity and current/upcoming call descriptors. It contains no personality vector or named numeric trait fields. The client receives concrete drawing/audio parameters that express personality, not a five-number personality object under another name. Exclude event history, private presence totals, emails, notebook archives, and visitor identities. Budget the seven-bird compressed snapshot to 20 KB or less, with small future schedules rather than sampled animation frames.

### Visit endpoints

| Method/path | Contract |
| --- | --- |
| `POST /api/account/invitations` | Host deliberately supplies recipient email; resolve/create pending UUID identity, persist one-time grant, email link. No automatic sharing. |
| `GET /api/account/invitations` / `DELETE /api/account/invitations/{id}` | Owner lists and revokes outstanding/active grants. Revocation atomically invalidates dependent visitor sessions. |
| `POST /api/visits/consume` | Consume unused unexpired token once and create a visit-scoped session, without creating an owner aviary or greeting. |
| `GET /api/visit/snapshot` | Recheck invitation/session/host status on every pull, return projection of the same scene state; never notebook or account data. |
| `GET /api/account/visit-log?cursor=…` | On-demand reverse chronological email/date/approximate duration plus outstanding invitations. No badges or unsolicited display. |

Unused invitations expire exactly 30 days after issue. Initial active visit-session lifetime is eight hours, with host revocation taking priority; the host issues a new invite for another visit. Reload within the existing valid session works without reusing the link. Consumed, expired, revoked, or unavailable links show the same general system message. A visitor's local mute/captions/motion controls remain available, but there are no greetings, listen-in, offers, settle, adoption, notebook, or host event endpoints.

## 6. Simulation engine and continuity

### 6.1 Tick execution and exactly-once effects

Start at a 60-second cadence with per-aviary jitter. Schedule every active aviary, not only currently connected ones. Partition due work by synthetic aviary UUID; a worker claims a durable lease, then locks the aviary row. Process ticks in a short database transaction with bounded event batches. At 100,000 aviaries, the service needs approximately 1,667 aviary advances per second; capacity-test this workload before admitting that cohort. Batch scheduling and database round trips, not independent canonical writers.

For each logical minute:

1. Read the persisted canonical state, simulation version, last tick time, event cursor, filter remainders, and RNG state under the lock.
2. Select committed event sequences above the cursor up to a transaction-consistent watermark. Validate effective times against previously accepted boundaries, deduplicate intervals, and resolve ordered presentation decisions already made by the command service.
3. Form account presence intervals and per-bird attention inputs; update persistent low-pass filters and add nonnegative personality deltas.
4. Advance local-time/circadian inputs, rare weather, mood timers, and bird-to-bird reactions. Generate the next canonical perch/action/call timeline.
5. Derive supported notebook candidates and update compact private observation summaries. Insert qualifying prose only under the notebook sparsity rules.
6. Persist vectors, moods, timelines, filter state, RNG state, consumed cursor, next tick time, and incremented state revision atomically. Record an operational timing measurement through an allowlisted collector that cannot receive the state.

The logical tick key is `(aviary_id, logical_tick_time)`. A retry must either observe the completed cursor/revision and do nothing or repeat the uncommitted deterministic transition. Persist notebook uniqueness keys and action cue IDs in the same transaction. A worker crash before commit rolls back all effects; a crash after commit cannot reapply drift. Two workers claiming the same account serialize at the database lock even if a queue lease expires.

Use a fixed event lock order and server-assigned sequence numbers. Command acceptance also uses this aviary lock briefly, so no cooldown decision or event can race into the middle of a tick's committed state. The worker does not hold a lock while rendering audio, making network requests, sending mail, or producing an export file.

### 6.2 Missed ticks, recovery, and deployment changes

Tick outages are caught up on the server from persisted state, unconsumed events, and elapsed time. Step chronologically using the pinned simulation version and logical timestamps, in batches capped initially at 60 minutes per transaction. Release between batches and prioritize visible aviaries without starving unattended ones. Mark a recovering snapshot with `simulated_through`; clients may continue its bounded cosmetic motion but cannot pretend it is current. Large gaps are a degradation, not the normal absence mechanism.

Do not reconstruct personality from the lifetime event log. Recovery starts from the stored vector/filter/cursor checkpoint. Do not reset mood on reconnection. Use virtual-clock tests to prove a delayed series of logical ticks produces the same vector and event consumption as uninterrupted ticks. Daily daylight-saving transitions and timezone edits use elapsed UTC for drift; local time affects only circadian and prose decisions.

A compatible deployment preserves stored grammar/signature and simulation-version interpretation. Canary a new version on synthetic aviaries first; migration adds fields with safe defaults and converts old records without replacing bird IDs. Rollback selects the prior compatible algorithm for subsequent ticks; it must never restore an old state snapshot over newer drift. Make a pre-migration backup and verify a forward repair path, because blindly rolling back a database can erase the relationship.

### 6.3 Personality initialization and drift equation

Initialize each trait in `[0,1]`, typically within `[0.25,0.65]`, with a species baseline and bounded per-bird offsets. Ensure the starters have legibly different silhouettes, call signatures, and initial tendencies without labeling either better. The five traits are boldness, social warmth, vocal frequency, plumage saturation, and curiosity. Values persist and may only increase through accepted positive input, asymptotically approaching the upper bound; no max-level surface exists.

Use nonnegative, saturating daily input doses, not click counts. Let `P` be the union of accepted owner presence seconds for the current canonical local day. Let `L_b` be qualified listen-in seconds for bird b. For a starting calibration, define `D_P = 1 - exp(-P/900)` and `D_L,b = 1 - exp(-L_b/600)`. Each tick receives only the increment in these doses since the previous tick. Daily buckets reset, but accumulated filter state and vectors do not.

Offer dose uses separately capped valid approaches and accepted offers: `D_O,b = 1 - exp(-n_b/3)`, where `n_b` counts eligible gestures for the relevant trait, not rejected attempts. A near-bird offer supplies a small boldness input; acceptance supplies a curiosity input. Seed, fragment, and pool all share the same per-bird cooldown. Repeated taps inside cooldown contribute nothing.

Trait routing uses:

- Presence coefficient 0.80 for every trait and bird.
- Listen-in coefficient up to 0.15 for social warmth and vocal frequency only.
- Offer coefficient up to 0.05 for boldness/curiosity as appropriate.
- Settle has zero personality coefficient. Explicit mute or audio unavailability never subtracts any trait.

Thus presence remains the dominant signal, and even extreme command activity cannot outrun the daily input bounds. Each bird receives the same account presence dose rather than one-seventh of it in a full aviary. Listen-in remains bird-specific. An eligible offer can cause an immediate mood-shaped visual reaction even when recent owner input does not qualify for drift; secondary personality credit requires overlap with honest owner presence so unattended automation cannot replace attention.

For each trait, let `q` be the weighted nonnegative dose accumulated during a tick, `h` its duration in days, `tau = 3 days`, `a = exp(-h/tau)`, and `x = q/h`. Persist the filtered drive `F` and update:

`F_next = a * F + (1-a) * x`

`I = F * tau * (1-a) + x * (h - tau * (1-a))`

`delta = (1-p) * (1-exp(-k_trait * I))`

`p_next = p + max(0, delta)`

The equation integrates a low-pass drive over the tick instead of discarding the minutes after the user leaves. Use numerically stable exponential-difference functions and fractional carry. Start `k_trait` in the range 0.007–0.012 per normalized input-day, then choose one versioned coefficient per trait from synthetic calibration. These are not exposed account knobs. With regular 10–20 minute daily presence, the acceptance target is roughly 0.008–0.02 absolute change by day seven and 0.03–0.07 by day 21 for typical initial traits, with no visually apparent single-session jump. The perceptual targets, not these draft numeric bands, determine final coefficients.

When no new input arrives, `q=0`: the filter decays, its remaining positive influence can still accumulate, and the vector cannot fall. The finite integral of the filter tail makes absence-based growth bounded. Quietness after absence comes from short-lived social salience and time-of-day/mood, never from reducing warmth, boldness, saturation, or vocal-frequency traits. Maintain a separate nonnegative recent-attention envelope that relaxes toward neutral ambient behavior; it does not cause wary, sad, hungry, or distressed states because the user left. Baseline calls and motion remain alive.

Calibration fixtures must include regular watching with no offers, a single visit followed by three weeks away, continuous foreground activity, two simultaneously active devices, repeated offer spam, caption-only listen-in, a timezone change, and a year-old aviary. Compare trajectories internally in synthetic fixtures; do not build an average-real-user drift dashboard.

### 6.4 Mood and social behavior

Finalize v1 mood to five states: wary, content, curious, drowsy, and alert. Mood has an entered time, bounded intensity, next-review time, and temporary influence timers. At each tick, evaluate transition hazards from circadian baseline, current weather, persisted personality, recent accepted reactions, and other birds' cues. Use a minimum dwell period, initially five minutes for ordinary mood changes, to avoid flickering. A startling bird call may create a shorter transient attentive pose without repeatedly flipping the enum.

| Mood | Motion/perch expression | Call/exchange expression |
| --- | --- | --- |
| Wary | Further-back preference, scanning, cautious approach | Shorter, spaced calls; bounded neighbor alert influence |
| Content | Preening, weight shuffles, comfortable middle/front perch | Soft phrases and occasional responsive calls |
| Curious | Head-tilts toward offers/sounds, investigation | Varied responses or joining an offered motif |
| Drowsy | Low/fluffed posture, still-pose sequences | Sparse soft calls; may ignore an offer |
| Alert | Upright orientation, active scanning without alarm loops | Clearer short calls and attentive replies |

A randomized baseline refresh every 20–28 hours supplies the daily-ish mood reset. It changes transition probabilities and gradually releases yesterday's session influences; it does not replace the current state with a default at midnight or on tab open. Morning favors alert/content; dusk favors drowsy. At night most birds settle, while the nightjar-like species may remain active. Temporary wary is an ordinary response to ambient events, never a penalty for absence.

Choose perch zones probabilistically from mood, boldness, and available space. A scene has bounded slots in front/middle/back; the server assigns slots and schedules transitions with collision avoidance. Client aspect ratio changes map those same logical slots to safe positions. The client never changes a bird's emotional proximity to solve layout.

Server timelines include call-and-response cues. Use short randomized response delays and per-bird refractory periods; permit overlapping motifs for a chorus but cap simultaneous starts and response-chain depth. A neighbor alarm can influence mood briefly, but the influence expires and cannot form a self-sustaining all-wary loop. A visitor's audio playback, owner mute state, or browser throttling cannot drive bird-to-bird transitions; those come from canonical planned calls.

### 6.5 Weather, light, and behavioral timelines

Use procedural ambient weather, not a location/weather API. Start with two or three short rains per week on average and occasional gentle wind, independently of owner activity. Store the next scheduled event and RNG state. Rain lasts initially 3–8 minutes, lowers short-term calling propensity, and clears its influence afterward. Wind makes some birds watchful and others more alert. No severe weather, actionable alert, or weather notification exists.

Calculate lighting continuously from the stored timezone with versioned morning/day/dusk/night curves. V1 uses clock-time curves rather than geographic sunrise; no location permission is needed. Smooth an explicit timezone change over several minutes to avoid an abrupt scene jump. Settle is a reversible overlay on this shared base, not a timezone rewrite.

Each tick compiles a rolling 120-second timeline containing action IDs, start/end times, pose phases, perch transfers, upcoming calls, and transition seeds. Preserve already-issued overlapping segments; only extend or explicitly supersede them. Server compilation is deterministic from persisted state and RNG counters. Expanding a motif or analytically sampling a pose in the client is rendering, not a client simulation tick. Per-leaf and feather particles are deliberately outside canonical state.

## 7. Presence, interactions, and session state machines

### 7.1 Exact owner presence

Represent each owner view as `hidden`, `visible-unfocused`, `visible-focused-inactive`, `present`, or `settled`. `present` requires all of: `document.visibilityState === 'visible'`, `document.hasFocus() === true`, and a trusted pointermove or keyboard press within the previous five minutes. Five minutes is the starting activity window because quiet watching is expected; tune only through explicit synthetic and usability work. `keydown` is the implementation of a keyboard press, including navigation keys, since printable-only events would exclude keyboard navigation. Do not record key values.

Do not count navigation, a focus event by itself, media playback, a timer, synthetic input, or an open tab as recent activity. A pointer click/tap without a pointermove or keypress does not bypass the conjunction. Test actual mobile pointer events and assistive input: where a modality does not emit either specified activity event, document and solve the input requirement explicitly rather than silently treating foreground time as attention. Focus detection limitations are a calibration risk, not a reason to invent presence.

Track transitions with the client monotonic clock. Accumulate only intersecting eligible spans, splitting immediately on blur, hidden, activity expiry, settle, or close. Send compressed interval pings every 15 seconds while present and flush a bounded terminal segment on exit with keepalive/beacon where available. If exit delivery fails, an accepted interval still only credits its measured past span; never grant an open-ended future interval. Rendering a still-visible unfocused window may continue, but its presence is zero.

The server validates start/end ordering, reasonable clock offset, activity age, session/view ownership, and maximum segment length. Initially accept at most 30 seconds per segment and a 45-second live lease for listen association; older unacknowledged presence is discarded rather than backfilled after a long offline session. Record small retries idempotently, but do not persist a replayable offline presence queue. Browser evidence cannot prove human attention against a malicious client; duration caps, daily saturation, and rate limits bound the failure without invasive tracking.

Deduplicate by event UUID and union accepted time intervals across all owner views before computing account presence. Two devices visible at once must never provide two seconds of presence for one second of wall time. For overlapping listen-in on different birds, share at most one second of targeted attention proportionally across active targets; identical targets are deduplicated. Associate listen time with qualified presence, not the duration a focus flag remained stored in a disconnected tab. All visitor endpoints lack the code path and permission to emit presence.

### 7.2 Arrival and procedural greeting

On fresh owner navigation, transition back to visible, or return after a long frame gap, issue one arrival event per visibility epoch after the initial scene is visible. Do not require a presence ping before a greeting; a bird can notice an arrival that does not yet qualify as sustained attention. Use a server-maintained last-owner-attention/visibility boundary to estimate absence, never expose the elapsed duration to the user.

The server draws a weighted primary greeter using warmth, boldness, current mood, and recent greeter history. Drowsy/wary birds are less likely, not excluded absolutely. Produce a continuous procedural gesture from absence bucket, gaze target, head angle, approach amount, timing, and optional call motif; compare its fingerprint to the previous greeting and resample an exact repeat. Short absence favors a glance; long absence favors gentle reorientation or a longer call. Preserve recognizable individual tendencies without rotating three canned animations.

Start the primary cue within 1–2 seconds of return under the supported latency target. Coalesce simultaneous arrivals from devices within a two-second window into the same primary cue, so one return cannot make every bird greet. Any follower is a delayed response with randomized spacing, never a synchronized arrival chorus. Accepted cue IDs propagate to other devices; they are not replayed when a snapshot repeats. A greeting is the only welcome surface. Narration can describe it promptly as an observation; no toast, banner, text addressed to the user, or absence counter is added.

### 7.3 Listen-in

Listen-in is a local audio attention state keyed by bird UUID. Pointer/touch activation starts it; repeated activation on that same bird ends it. Focus transfer ends the former bird's mix and begins the next, with smooth ramps. Empty scene activation, leaving bird keyboard focus, Escape, hidden state, or settle ends it. On any termination, send an idempotent end event and close the server attention lease; loss of that event is bounded by the lease.

The local mix does not force extra canonical calls and is not synchronized to a different owner's device. Accepted focus duration contributes only through the next tick's bounded per-bird attention dose. Caption users and audio-muted users can pay the same focused attention; do not tie drift credit to successfully producing speaker output. A user deliberately muting calls can supply a small expiring quiet-mood influence, but audio permission failure, hardware silence, and reduced-motion preferences cannot alter personality or penalize the user.

### 7.4 Offers

The top-bar menu offers seed, one of a small curated set of procedural melodic fragments, and still pool. It never requires clicking a bird. The server chooses a nearby eligible receiving bird from current logical perches and curiosity/mood, preferring the current listen-in target only when that bird is eligible and naturally near the offering location. Reserve a three-minute cooldown per receiving bird across all offer types and all devices under the aviary lock. If all birds are cooling down, the menu quietly disables the action with a short contextual explanation; there is no countdown or penalty message.

Choose and persist the response at acceptance: approach/accept, watch/wait, ignore, or a species-appropriate drink/bathe/call response. Curious/content birds are more likely to approach; wary birds may wait; drowsy birds may remain still. These are observable behavior choices, not success/failure labels. Store the decision in the event so delayed ticks do not reroll it using a different mood. Limit one transient offering object at a time; replacement or expiry fades it out rather than accumulating scene clutter.

The server returns an immediate timeline with the offer object and reaction; next tick applies the small mood influence and eligible drift input. Caption the actually emitted fragment/response from its procedural descriptor. No generic `gift accepted` announcement exists. Narration describes the observed reaction, including a bird merely watching, without treating non-acceptance as a user failure.

### 7.5 Settle, undo, close, and re-engagement

Settle appends a presentation epoch that ramps shared lighting toward evening and calls quieter over four seconds. It closes current owner presence/listen intervals at acceptance. The initiating browser continues a display-only heartbeat while visible so the override can remain without generating presence. Other devices read the same override; explicit owner activity in any device sends resume and supersedes it. A visitor cannot renew or cancel this owner display lease.

Any click/tap in the aviary within five seconds sends undo and is consumed as undo, rather than also offering or toggling a bird. For keyboard parity, Escape during those five seconds performs the same action. Undo uses the settle event ID; repeated undo is harmless, and a stale undo cannot cancel a newer settle. It reverses lighting/audio from their current values, restores the shared time-of-day view, and opens a fresh eligible presence window only if all conditions hold. If the tick already consumed settle, resume removes its remaining quiet influence on the next tick; vectors are unaffected either way.

After five seconds, explicit scene/keyboard action re-engages through resume; passive rendering or pointer drift alone does not accidentally cancel settle. Closing the initiating view clears the override on the server's close event; a missing close event expires its display lease after 45 seconds. Closing without settling ends presence through the same interval rules and carries exactly zero special penalty or different drift weighting. Network loss cannot hold an aviary permanently in a settled state.

## 8. Pull synchronization and degraded operation

Use visible-owner pulls every 10 seconds with jitter, plus immediate pulls on return to visibility, focus after a long gap, successful command, and render gaps longer than two seconds. This is client snapshot polling, not simulation ticking. A command response supplies its presentation revision immediately; the regular snapshot catches up to the next canonical tick. Visitors pull every two seconds while visible to make revocation prompt; conditional responses keep unchanged pulls small.

Track the tuple `(state_revision, presentation_revision)` with cue IDs and server timestamps. Do not install an older canonical revision merely because its HTTP response arrived later. An old state response cannot erase newer action receipts. Request responses include enough monotonic information to merge the two version axes deterministically, and expired actions leave short-lived tombstones so an out-of-order response cannot revive them. No vector is merged client-side.

Estimate server clock offset from request/response midpoint, bounded by round-trip uncertainty. Audio schedules against AudioContext time mapped from this clock; rendering uses the same cue times. Adjust clock estimates gradually for small corrections. If the browser sleeps, suspend rendering/audio and renew snapshots on wake before processing any future cue. Drop calls and greetings whose end time has passed; never play a burst of everything missed while the laptop was closed.

Reconcile compatible motion updates over 300–800 ms using their current pose/velocity, not a fade of the whole aviary. Server cues already in progress are sampled at their elapsed phase. A major stale-position difference resolves as an authored perch transition; reduced motion cross-fades between the two valid poses. Do not fabricate a new mood to disguise stale data.

On network failure, show the last valid scene with bounded ongoing cosmetic micro-motion, stop crediting unsent attention, disable commands that require acceptance, and retry with jittered exponential backoff from one to 30 seconds. Use a quiet system status in the top-bar/panel region when action is blocked; no repetitive error toasts. After timeline expiry, do not invent new social events or notebook observations. Reconnection installs the current server state rather than replaying a local simulation. A cold load with no snapshot shows the quiet sky field and a direct retry error only when needed, never another account's cached birds.

Keep pending commands only in bounded memory with their original event IDs. After an uncertain offer/settle response, query its receipt before retrying; never generate a new ID for a retry. Expired sessions stop sends and show the system sign-in message. An owner resuming in another device sees all committed state on the next pull. Access checks run even for 304 responses, so a revoked session cannot continue by receiving cached authorization success.

An owner settings/name conflict uses conditional versions and offers reload/reapply of the user's specific edit. It is not a sync conflict between birds and must never offer to choose one device's personality history over another.

## 9. Frontend scene and loading pipeline

### 9.1 One rendering tree from first paint onward

Use retained SVG with compact species silhouettes and grouped transforms for body, head, wings, tail, and feather details. Seven birds do not justify a large general-purpose 3D or game-engine dependency. The server renders the first valid bird poses into the HTML with the state timestamp and a tiny inline timeline bootstrap. The bootstrap sets elapsed animation phase before first scene paint; the full runtime takes over the same SVG nodes without replacing or fading the scene. Initial pose selection already places birds mid-preen, mid-scan, or at an appropriate calling pose. Never show a static preview and then animate an entry sequence.

Rendering pipeline:

1. Read one immutable compiled snapshot and its action timeline.
2. Map logical perch slots into the current viewport's safe coordinate system.
3. Sample each server-authored pose segment at `server_now`, blend any explicit reconciliation transition, and update only changed SVG transforms/attributes in one animation-frame pass.
4. Sample slow lighting/weather curves and bounded independent leaf/feather ornaments.
5. Place semantic focus geometry and enabled call captions using those same projected bird bounds.

The runtime does not call the component framework 60 times a second. Forms/notebook state are separate islands; their re-render cannot restart a bird action or audio context. Keep species assets reusable through SVG symbols; avoid heavy blur filters, expensive path mutation, layout reads inside per-bird updates, and per-frame object allocation. Precompute pose bases and use fixed-size buffers for interpolation.

When the initial snapshot is delayed, the fallback is a quiet sky/foliage field with very faint ambient movement. It is a genuine loading contingency, not permission to count the field as the first bird in performance tests. Authentication errors replace it with a direct system surface. A previously initialized account must never render an empty aviary as though it were new because a request failed.

New-account adoption is the sole intentional empty-to-bird arrival sequence: choose two starters server-side, offer default names that the user can keep/change, then animate the first arrivals softly into the quiet field after adoption is committed. Later ordinary navigation always begins mid-action. Additional age-based adoption may use a restrained single-bird arrival, with no celebration or counter.

### 9.2 Composition and responsive constraints

Render sky/soft foliage, back perch, middle perch, front perch, birds ordered by depth, and occasional foreground ornament. Front/middle/back proximity remains legible through scale, contrast, and placement. No scene panning, scrolling, zooming, placement tools, hover labels, or inline buttons exist. Captions and accessibility focus outlines are the explicit exceptions to the prohibition on scene chrome.

Use a fixed logical horizontal composition with responsive slot spacing and aspect-preserving fit. Narrow screens reduce horizontal gaps and use the three existing depth lanes; wide screens add breathing room. Recompute safe positions on resize without changing canonical zone/slot choice. Bound all bird bodies, motion envelopes, and flight paths inside the scene, accounting for wings, caption extents, and focus rings. Reject a path during timeline/layout validation if its transformed bounds leave the safe rectangle. Letterbox modestly when necessary rather than cropping a bird or stretching its silhouette.

Acceptance viewports include 320×568, 390×844, 568×320, 768×1024, 1366×768, and 2560×1440 with two and seven birds. UI text zoom/reflow may make settings and notebook panels scroll; the aviary itself remains a single fitted scene. At extreme zoom, preserve keyboard access and legibility rather than shrinking controls below usable size. Hit regions remain at least 44×44 CSS pixels on touch, with nonoverlapping routing and a semantic keyboard alternative for tightly spaced birds.

Use muted blue, green, brown, and ochre tokens and a warm light palette. Create actual day/night/weather contrast samples during the design slice because the separate visual design document is not among the supplied PRDs. Do not block engineering on that missing document: v1 acceptance requires at least 4.5:1 normal text, 3:1 large text, and 3:1 focus/control boundaries, with a 7:1 target for small caption text on its plate. Verify every interpolated lighting extreme, not just noon screenshots.

### 9.3 Idle motion, top bar, and panels

Pose generators express preening, scanning, weight shifting, and head-tilting through smooth trajectories with small seeded timing variation. Avoid obvious repeated loops and twitching changes on tick boundaries. Drowsy is a quiet living posture, not a paused animation. Idle ornament pools have strict caps, initially four leaves and two feathers, and generate at slow irregular intervals. Parallax is a very small automatic layer drift, not a mouse-following effect or a scroll interaction.

The top bar starts visible and fades after four seconds of pointer stillness. Restore it immediately on pointer movement, keyboard activity, pointer proximity, focus within it, or a touch intended for controls. Never fade an open menu, focused control, tooltip/help copy, or settings surface. On touch and screen-reader/keyboard use, favor discoverable controls. Fade the bar's fill and decorative weight while preserving accessible contrast for any still-visible functional content; do not leave low-contrast text halfway visible merely to satisfy the fading effect. Only four icons exist, all with accessible names.

Opening account, accessibility, notebook, or offer panels does not reset the scene. Use a bounded panel above the layout or a small dialog outside the scene proper. Trap focus only in a modal dialog; restore focus to the invoking control on close. Notebook scroll does not pan the aviary. Bird settings/names and quiet adoption availability live in account settings, reached deliberately, with no `new` badge or age threshold announcement. The offer menu remains the fast path for gestures and settle.

## 10. Procedural audio and captions

### 10.1 Shared call descriptors

Author about six species motif libraries as compact parameter grammars: note count/rhythm families, pitch contours, harmonic/noise components, envelope shapes, pause rules, and allowable ornamentation. A bird receives an immutable call-identity seed and bounded signature parameters at adoption. Species supplies the family; the identity seed supplies a recognizable contour/register/timbre within it. Mood changes articulation and spacing, and long-term vocal frequency changes how often it calls, without replacing the identifying contour.

The server schedules call IDs, start times, grammar version, motif seed, and constrained expressive parameters in its timeline. The client expands each seed deterministically into a realized call descriptor containing note starts/durations, frequencies/curves, envelope components, and descriptive features. Use this same realized descriptor for synthesis, beak timing, and caption text. Client sampling does not select whether a canonical bird calls or whether another bird replies; those decisions already exist in the server plan.

Each generated call varies in micro-timing, articulation, pitch detail, or ornamentation within the signature bounds. Do not ship recorded bird calls, loops, sampled phrase fallbacks, or files masquerading as procedural variation. The user-offered melodic fragment library is symbolic note data synthesized through the same runtime, not downloaded audio recordings.

### 10.2 WebAudio implementation and mix

Use one AudioContext per document. Prefer an AudioWorklet with a fixed voice pool and no allocations in its real-time processing callback; use 16 bounded synthesis voices initially, enough for seven birds with two overlapping components plus the offered fragment. Reuse wavetable/noise resources generated locally. If AudioWorklet is unavailable but WebAudio functions, use bounded OscillatorNode/filter/envelope graphs with deterministic teardown and reuse of reusable resources. Oscillators that must be recreated are disconnected, dereferenced, and verified collectible when complete.

Keep per-bird gain/filter buses feeding a master limiter and gain stage. Use mild position-based stereo placement that still sounds coherent in mono. A chorus allows genuine overlapping variable motifs; avoid perfectly synchronized starts, identical phases, and dense competing registers. Schedule 100–200 ms ahead against AudioContext time, refresh the scheduling window without duplicating call IDs, and discard elapsed calls after a suspension. Never create a new context on each listen-in, return, or settings change.

Starting mix parameters, subject to listening review:

- Ambient bus at calibrated comfortable level; focused bird rises approximately 3 dB over 0.8 seconds.
- Other birds fall approximately 6 dB over 1.2 seconds and retain an audible ambient floor. They do not become silent.
- Disengagement decays all gains back to ambient over 1.2 seconds. Rapid focus changes ramp from the current gain instead of restarting from a hard endpoint.
- Settle ramps toward a quieter shared mix over four seconds; normal nocturnal activity remains possible.
- Normalize for bird count and simultaneous voices. Reserve headroom and limit peaks below −1 dBFS; test headphones, laptop speakers, mono, and mobile output. Avoid shrill transients and pumping that makes a chorus sound compressed or mechanical.

Listen-in affects playback gain only; it cannot alter someone else's mix or rewrite a bird's call identity. Continue canonical call scheduling while muted so captions and mood read the same aviary. Hidden tabs stop rendering, fade local audio down briefly, and suspend the context; the server continues without them. A visible unfocused window may continue motion/audio but earns no presence. On return, render and schedule from fresh state rather than replaying calls from the gap.

### 10.3 Permission and failure behavior

Browser autoplay rules make unconditional first-frame audibility impossible to guarantee. If audio is already permitted, begin at the current cue phase; otherwise show the living scene with captions automatically enabled and permit sound through an ordinary explicit gesture/settings control. No blocking `tap to enter` splash, autoplay error toast, or fake recorded sound is used. The first purposeful owner gesture may resume a suspended context when the user's audio preference allows it; an explicit mute is never overridden.

Catch context creation/resume failures, worklet failure, device changes, interruptions, and missing WebAudio. Fall back to graceful silence with call captions on by default; expose a direct matter-of-fact sound control in accessibility settings. A failed advanced synthesis path may try basic procedural WebAudio if available, but there is no recorded-audio fallback. All drift, offers, moods, narration, and notebook behavior remain available in silence. Persist the user's caption choice independently from the automatic fallback state, so restoring audio does not unexpectedly discard captions they enabled.

### 10.4 Caption generation and placement

Derive prose from the realized call: note count, rising/falling contour, pauses, trill texture, intensity, and where the bird is perched. Examples include `a soft three-note rise` and `a low trill, paused, low trill again`; these are compositional patterns, not a single fixed caption per bird or sound ID. If a voice limit changes the emitted descriptor, update its caption accordingly; do not describe notes the runtime dropped.

Show each caption close to its calling bird with a subtle contrast plate and gentle entry/exit matched to the call. Keep it in bounds and route neighboring captions into deterministic nearby slots. For seven birds, avoid overlapping text by spacing canonical call starts when needed and allowing a nearby stacked caption lane with clear bird-name association. Do not suppress important captions simply to preserve a clean screenshot. Reduced-motion caption transitions are slower opacity changes without translation. Calls are described even in silence because the procedural descriptor still runs; screen-reader users receive the paced narration instead of every caption being forced into a second live region.

## 11. Accessibility as the same product

Implement accessibility during the first two-bird scene, not as a post-launch audit. Use real DOM controls for top-bar items and a semantic bird navigation layer aligned with SVG positions. The visual SVG can be hidden from the accessibility tree when a parallel descriptive region already represents it; do not double-announce every SVG part. Never put trait values into labels, debug attributes, narration, or `aria-valuenow`.

Screen-reader surface:

- A concise initial naturalist scene paragraph from current state, followed promptly by a description of the primary greeting if one occurs. It describes birds and place, not the user's arrival as a system announcement.
- A polite, atomic narration region updated about every 45 seconds while visible. Generate a coherent short paragraph from the same current pose, call, light, and weather facts used by rendering.
- A bounded queue, initially three observations. Coalesce stale ambient updates and prioritize accepted user gestures without interrupting control labels. Never enqueue a minute of missed narration after hiding or sleeping.
- User actions may produce prompt short prose, such as a specific bird approaching a pool. Avoid enum labels, raw event logs, stock `state changed` sentences, and unsupported claims about feelings.
- Controls to pause narration and explicitly read the current scene. Pausing narration does not halt the simulation. Opening a dialog suppresses routine scene chatter until the user returns.

Keyboard contract: Tab traverses top-bar controls in visual order, then enters the scene at its first bird through a single roving tabindex. Arrow keys move between birds in stable spatial order. Focus starts listen-in; Enter ensures it; Escape exits it. Tab away restores ambient mix. All menu actions support ordinary arrows/Enter/Escape. Provide the offer menu through its top-bar button and a documented optional shortcut, disabled until enabled to avoid screen-reader/browser conflicts. Settle remains reachable without a custom shortcut. Visible focus outlines have a two-tone treatment legible against every lighting/weather state; moving birds cannot move keyboard focus to a different identity.

Reduced-motion preference uses the OS setting on first load, with an explicit application override (`system`, `reduced`, `standard`). In reduced mode, replace continuous pose motion with slow 2–4 second cross-fades among curated still poses. Perch changes cross-fade between valid old/new positions; no interpolated flight path is shown. Remove drifting leaves/feathers and parallax, slow lighting transitions, and retain the same moods, calls, captions, drift, and notebook. Do not freeze the entire scene or remove greeting/offer reactions: express them through different still poses and observation prose. Changes to the preference apply without reload or state reset.

Visitor mode keeps the same naturalist narration, captions, local sound controls, and motion preferences, but bird navigation describes the scene without engaging listen-in. Accessible controls must not accidentally send owner interaction events. Test this independently from merely hiding visual buttons.

Acceptance includes keyboard-only use, VoiceOver with Safari on macOS/iOS, NVDA with a supported Windows browser, browser zoom/text scaling, forced-colors/high-contrast settings, reduced motion at initial navigation, and a muted/unavailable audio context. Evaluate prose and pacing with screen-reader users; automated ARIA checks cannot establish whether the aviary feels alive.

## 12. Field notebook and adoption pacing

Use a local, deterministic observation composer with authored naturalist grammar. No external language-model call receives private events. Generate candidates from facts the server actually knows: a changed greeting order, an unusual but supported call response, extended preening, a brief weather-linked posture, or a sustained quiet interval. Require evidence for qualifiers such as `first time this week`; retain a compact seven-day greeting-order summary rather than deriving assertions from whatever raw logs happen to remain.

Separate candidate selection from prose realization. Score candidates only inside that aviary for specificity/novelty; do not rank accounts or collect population behavioral scores. Use a cooldown starting at 48 hours for ordinary entries, an average target of one every two to four days under regular use, and a rare exception for a clearly different noteworthy event. Initially cap exceptional publication at two entries per day and deduplicate repeated topics over seven days. This prevents a heavily active owner from generating a feed. A daily timer does not force an entry if there is nothing worth observing.

Store the final prose with its template version and observation evidence at publication. An old entry must not change when templates improve or a bird is renamed. Avoid logs such as `session started`, numerical vector deltas, interaction totals, visited-day counts, or observations of the user's habit. Observations of the aviary may refer to quietness without implying guilt or neglect.

Use cursor pagination and a bounded rendered window, initially at most 90 notebook items in the DOM. Release references to items outside the window while preserving focus and scroll anchors. Let users move back indefinitely; do not archive, hide, or truncate old entries because of their age. The notebook has no edit, delete, annotate, or visitor routes. Account deletion is the only operation that removes it.

Initial account activation creates exactly two stable bird records and one aviary atomically. The system chooses distinct starter species from the pool and persists their seeds before showing name suggestions. Refreshing or abandoning naming cannot reroll birds. Names have useful defaults and can be changed at any time from bird settings.

For later adoption, eligibility depends solely on `aviary.created_at` and the stored age schedule. A quiet entry in account/bird settings presents the next offered bird when the owner chooses to look. No onboarding invitation to collect birds, scarcity, countdown, celebratory unlock, achievement, or adopted-count display is added. The owner may defer indefinitely. Accept at most one outstanding offer per transaction; if several age thresholds elapsed, reveal the next on a later deliberate settings visit, not as a queue of reward popups. The seventh may share a species, but it must have a distinguishable individual signature. Never replace an existing bird to improve species variety.

## 13. Privacy, account lifecycle, and visit details

### 13.1 Identity and private storage

Generate account UUIDs independently of email. Encrypt email at rest with restricted key access; use a secret-keyed normalized blind index solely for identity lookup on the account record. Neither email nor its lookup index is a sharding key, log field, trace label, queue identifier, or analytics dimension. Temporary new-email verification state is encrypted and expires. Render visit-log addresses by authorized lookup; do not copy them onto invitations, session logs, or bird data.

Use encrypted transport/storage, scoped application roles, and explicit projections for owner versus visitor. Apply a restrictive CSP and encode embedded snapshot JSON so names cannot break out of a script element. Keep magic/invite/export tokens out of URLs after exchange, access logs, analytics, and referrers. Use high-entropy tokens stored only as digests, same-site cookies, and rate limits. The email provider receives only the transactional recipient and necessary link/message, never bird events, vectors, notebook content, or presence.

Create magic-link sessions with an initial 30-day inactivity timeout and 90-day absolute limit; renew securely and expose revocation in settings. Replayed links fail after the first atomic consume. Email change does not replace account UUIDs, aviaries, or birds. Avoid account enumeration through request responses or invite UI. An invitation grants a bearer capability intended for its emailed recipient; forwarding can transfer that capability, so keep it one-time, short-lived after use, and revocable instead of claiming strong recipient identity without authentication.

### 13.2 Retention and operational separation

Private simulation input is used only to advance its owner's aviary. Keep consumed raw interaction events and presence intervals for 14 days for bounded recovery/debugging, then purge them after verifying their cursor is covered by durable state. Do not delete unconsumed events just because the worker is delayed. Keep compact notebook evidence summaries up to 28 days and immutable notebook prose for the account lifetime. Keep command idempotency receipts for 30 days; clients cannot replay older commands under fresh IDs automatically. Retain visit logs for 90 days initially, with outstanding grants shown until consumed/revoked/expired. Publish these concrete retention periods in settings/privacy copy.

The metrics collector uses a separate credential/network boundary and has no query or replication access to simulation tables. No analytics warehouse consumes change-data-capture from birds or events. No training, recommendations, population bird analysis, session replay, heatmaps, or third-party behavioral SDK is permitted. Test payload allowlists with canary emails, bird names, IDs, and event fields to prove that these values cannot reach the collector.

Aggregate operations data has no account, bird, session, invitation, or stable device ID. A separate restricted short-lived operational error store may use account UUID to diagnose an account error, without event kind, bird fields, request body, or interaction chronology. Set its retention initially to seven days and include it in account deletion. Strip query strings, authorization headers, tokens, names, and emails from logs. Anonymous aggregate buckets cannot later be mapped back to an account.

### 13.3 Export and deletion

Export from a consistent database snapshot: birds and stable IDs, names, current vectors under the explicit exception in section 2, current moods, notebook entries, relevant settings, and schema/timestamp information. Do not export presence histograms, visit streaks, raw owner interaction history, other people's account data, or auth secrets. The export is not a UI stats panel. Large notebooks stream to an encrypted private object without holding all entries in memory. Deliver a download link valid for 24 hours to the verified address; expire the object and token afterward. No import feature is planned for v1.

Deletion marks the account immediately with an exact 30-day recovery deadline, revokes owner and visitor grants, stops queued mail/exports, and suspends simulation writes while retaining the last complete state. Display only the system recovery surface after fresh sign-in. `I changed my mind` clears deletion under a lock before the deadline, restores the same UUIDs/vectors/moods, and resumes server time advancement from the stored checkpoint. No new adoption or vector reseeding occurs.

At the deadline, an idempotent deletion job removes the account, aviary, birds, vectors/filters, notebook/evidence, inputs, sessions, grants, visit associations in either host/recipient role, outbox jobs, operational records, exports, private caches, and identity lookup index. Preserve unrelated hosts' birds and accounts. Keep only nonidentifying aggregate metrics that cannot be tied to anyone; there is no account-linked telemetry left to retain. Use transactional deletion markers to prevent a racing tick, export, mail worker, or recovery request from recreating data.

Backups must honor the 30-day hard-deletion promise. Encrypt account-private data under deletable account keys and destroy those keys at final deletion; separately remove live records and objects. Maintain a deletion journal outside restored database snapshots so restoring an older backup reapplies deletion tombstones before serving traffic and cannot restore erased keys. Validate this with a restore drill. Do not claim ordinary backup rotation alone fulfills the deadline.

Pending recipient identities that never activate an aviary are garbage-collected when no valid grants or retained visit records reference them. Host revocation removes the outstanding/active grant from the displayed list and terminates its visitor session; completed historical visits remain only within the log's retention policy. No toast announces revocation. A hidden visitor tab fetches authorization before resuming; a visible tab receives the unavailable surface on its next two-second pull. Data already viewed cannot be made unseen, but no further snapshot is disclosed.

Visit durations are approximate start-to-last-successful-pull intervals, rounded to minutes; do not collect pointer events or visibility-attention evidence from visitors. The visit log is transparency about access, not an engagement surface. Opt-in visit email sends at most one quiet message per consumed invitation, after checking that the toggle remains on; revocation/deletion cancels unsent messages. Keep the toggle entirely in account settings and never enable it implicitly.

## 14. Performance budgets and operational observability

Performance is a release property of the actual animated aviary, not a landing-page score. Fix a reference mid-tier mobile device, a five-year-old mid-range laptop, and reproducible network profiles during the first milestone. For the mobile gate start with 4 Mbps down, 1 Mbps up, and 80 ms round-trip latency, plus tests on physical devices and more adverse connections. Publish the profile with each result; a warm desktop measurement is not evidence for mobile first-bird performance.

| Surface | Required budget / initial engineering allocation | Verification |
| --- | --- | --- |
| Initial JS | Hard cap below 2 MB gzip for everything required at first paint; working target at most 150 KB gzip, with a tiny scene bootstrap independent of panel code | CI asset manifest sums actual initial imports; fail the cap and require review of target regressions |
| First bird | Below 500 ms from authenticated scene navigation on the reference mid-tier mobile/4G profile; target p95 below 500 ms over repeated cold runs | Browser mark tied to a real visible persisted bird plus filmstrip/screenshot validation, not a placeholder element or the quiet field |
| First response path | Initial target: TTFB at most 180 ms, compressed first-scene HTML/state/assets at most 60 KB, parse/bootstrap at most 80 ms, final style/paint at most 60 ms | Trace timing from navigation through actual first draw; revise architecture if the full sum misses 500 ms |
| Snapshot | At most 20 KB compressed at seven birds; no unbounded history or sampled frames | Schema-size fixtures at largest supported state and after repeated interactions |
| Greeting | One primary notice starts within 1–2 seconds of owner return under the reference profile | Cue timestamp and visual/narration probe with varied absence/mood fixtures |
| Idle frame | 60 fps on reference old laptop with seven birds over 30 minutes; target 95% of intervals at or below 16.7 ms, scene CPU under 4 ms/frame | Long-running automated trace plus physical-device sampling; inspect spikes and dropped frames rather than trusting average fps |
| Audio | No clicks, underruns, or unbounded voice growth; steady audio callback CPU below half its available block time | Offline waveform checks, real-time device tests, bounded voice/node counters in synthetic builds |
| Memory | No sustained retained-memory growth in a 30-minute scene/notebook/audio session | CI soak after warm-up; post-GC heap near baseline within 2 MiB instrumentation tolerance, no increasing retained-object counts or positive sustained slope |
| Tick | Routine service target well under one second per aviary; required alarm when p99 tick latency exceeds five seconds | Duration histogram and separate schedule-lateness/oldest-unprocessed metrics |
| Revocation | Every snapshot checks live authorization; visible visitor loses access on next two-second pull under normal network conditions | Revoke during a visit, inspect denied conditional pull, then repeat from hidden state |

The first-bird budget is strict on the reference profile, not a promise to defeat arbitrary network delay. Measure both cold connections/caches and repeat navigation. Inline critical styles, initial visible species geometry, snapshot, and the minimal sampler; defer settings, invitation UI, notebook history, extra species assets, audio worklet initialization, and noncritical fonts. Do not preload large media or a general animation framework. If the true first-bird path misses the budget, shorten delivery and bootstrap work; never substitute a random bird or claim the loading field as success.

The memory soak prewarms the bounded caches and then exercises repeated call generation, listen transfers, offers, settle/undo, snapshots, notebook pagination, resize, and visibility transitions. Assert one AudioContext, a fixed number of workers/pools, bounded event/receipt/caption queues, stable detached-node counts, and released notebook pages. Run instrumented Chromium soak in CI and periodic Safari/Firefox physical-device checks, because a single engine's garbage collector can hide another's leak. Memory thresholds allow measurement noise, not accumulated per-call allocations.

Synthetic browser probes run from several common geographies against designated synthetic accounts at a scheduled cadence. Measure first bird, frame timing, snapshot latency, audio startup/failure, and recovery from suspension. The supported-browser matrix is the last two major versions of Chrome, Safari, Firefox, and Edge, including mobile Safari/Chrome where applicable. Keep the matrix moving with releases; don't pin an old major version as permanently supported. Older/unsupported browsers get a concise system explanation without loading compatibility bundles for them.

Day-one aggregate operational metrics:

- Request volume, status/error counts, latency buckets, and retry rates by coarse service route group.
- First-bird/page-load/frame-time distributions by coarse browser/device class and app version.
- AudioContext/worklet failure and interruption counts, without per-call descriptors or bird IDs.
- Tick duration, scheduling lag, queue depth, transactional retry/deadlock rate, worker failures, and oldest pending work age.
- Export/deletion/mail job completion/failure latency and authorization-denial counts, without recipient or account labels.
- Optional anonymous session-duration histograms calculated locally and transmitted as coarse buckets without a session identifier; never used as an engagement objective.

Allowlist these fields at collection, bound metric label cardinality, and strip IP/user-agent detail at the ingress before retention. Do not capture payloads, full URLs, raw stack local variables, bird states, names, vectors, offered items, presence totals, per-account interaction sequences, or stable visitor identity. Avoid cross-linkable trace IDs on client behavioral events. Monitoring receives no private database credentials.

Alert on p99 tick latency over five seconds, missed scheduling deadlines, persistent authentication/snapshot failure, growing transaction conflict rates, stalled deletion/export queues, and audio failures above the synthetic baseline. Separate tick execution duration from queue lateness so a fast worker executing old work cannot appear healthy. An on-call runbook identifies how to pause admission, drain queues, roll back compatible code, and restore authorization; it never recommends rebuilding personality from logs.

## 15. Verification plan and acceptance evidence

Implement the following suites alongside their owning modules. These are planned product tests, not tests run as part of this planning task.

### 15.1 Simulation and transaction properties

| Test | Required result |
| --- | --- |
| No input for 1, 7, 30, and 365 days | Vectors never decrease; any initial filter tail produces only bounded positive drift; no hunger/distress/absence penalty appears |
| Regular synthetic watching at days 0/7/21 | Measurable instrument change near week one and perceptible matched-mood change near week three; no single session visibly changes plumage/proximity |
| Offer spam and replay | Shared per-bird cooldown and dose bounds hold; rejected/repeated events produce no additional delta or repeated notebook publication |
| Single versus two overlapping devices | Same union presence yields the same presence-driven deltas; overlapping attention cannot double-count seconds |
| Every combination of three presence conditions | Only visible AND focused AND recent permitted activity earns presence; inactivity boundary is exact under virtual time |
| Settle versus close | Equivalent presence termination and zero negative personality effect; settle adds only the documented temporary mood/presentation influence |
| Out-of-order/duplicate events and crash injection | Each accepted sequence affects drift at most once; no partial commit separates vector, cursor, RNG, and notebook |
| Two tick workers plus concurrent commands | Database serialization gives a single ordered result without lost updates, duplicate offer reservations, or double adoption |
| Worker outage then catch-up | Same persisted-state result as uninterrupted logical ticks for the pinned algorithm, without a client replay or reset |
| Daily mood refresh / DST / timezone change | No personality reset, negative elapsed time, double-awarded input, or tab-open mood reset; daily dose bookkeeping cannot be restarted repeatedly by timezone edits |
| Rename and migration | UUID, vector, filter, mood, and call identity preserved byte-for-byte where unchanged; old notebook prose remains stable |
| Bird-to-bird calls and weather | Responses are staggered and bounded, rain/wind influences expire, wary feedback cannot sustain itself indefinitely |

Use property-based generation of nonnegative input sequences, event retries, random timing gaps, and concurrent writers. Keep golden fixtures for engine versions and independent algebra checks of the drift integration. A passing arithmetic test alone does not establish visible three-week change; that requires the perceptual review below.

### 15.2 Owner and visitor end-to-end scenarios

Run each principal flow with supported pointer, touch, and keyboard input:

1. Request/consume a magic link; replay it, expire it, and open its landing page with a simulated email scanner. Exactly one successful consumption issues a session.
2. Activate a new account, keep/change suggested names, refresh during adoption, and confirm exactly two original birds with stable identity.
3. Open the scene, return from another tab, suspend a laptop, and return after simulated days. Observe mid-action first frames, a single varied primary greeting, and no textual welcome.
4. Listen in, transfer focus rapidly, click empty space, leave via Tab, press Escape, and hide the tab. Gains ramp correctly; other birds remain ambient; end events/leases are bounded.
5. Offer each item under every mood and from two devices simultaneously. Verify correct chosen recipient, cooldown, non-accepting behavior, captions, and later canonical mood update.
6. Settle, undo before five seconds, attempt stale undo, close while settled, lose the close request, and resume from a second owner device. No permanent override or extra presence remains.
7. Scroll a large notebook backward indefinitely and return to the scene. Entries remain readable/immutable and memory does not grow with visited pages.
8. Issue an invitation, visit before/after expiry, replay its token, revoke during playback, and restore a hidden visitor tab. All paths are render-only; host vectors, presence, greetings, and notebook receive no visitor input.
9. Toggle visit email on/off around queued delivery. Default produces no message or badge; opt-in sends only the specified quiet message and respects cancellation.
10. Rename on one device while the other has a stale form. Resolve only the name edit; both devices retain the same bird and canonical simulation.
11. Export from a notebook-heavy account; verify a consistent snapshot and token expiry. Initiate deletion, recover just before the deadline, race recovery with final deletion, then restore a pre-deletion backup in a test environment. Identity/data recovery obeys the deadline and erased accounts cannot reappear.

Auth tests include cross-account object-ID substitution, a visitor attempting every owner route, revoked-session 304 responses, CSRF, token logs/referrers, hostile bird-name markup, invite recipient enumeration, and concurrent email change. Scope assertions must be on the server; hiding a button is not the test.

### 15.3 Perceptual and accessible review

Maintain synthetic, reproducible scene recordings at common and edge states, and conduct human reviews with sighted/audio, screen-reader, caption-only, and reduced-motion users. Review initial motion, greeting specificity, naturalist language, emotional neutrality after absence, and whether quietness feels continuous rather than broken.

For drift calibration, use the same synthetic bird at day zero, seven, and 21 under matched mood/light, then test varied real-session-like conditions where mood can mask personality. Seek week-one numerical evidence and week-three observable differences without showing numbers to participants. Use a small 21-day consented pilot to check the felt timescale; collect volunteered qualitative feedback, not exported private interaction histories or population drift statistics. Begin this pilot early so it overlaps engineering rather than being waived at launch.

For calls, conduct blind identification within two-, three-, five-, and seven-bird synthetic aviaries across multiple moods and drift stages. The initial gate is at least 80% correct individual identification after familiarization, with no persistent confused pair, plus listener reports that variations remain recognizably the same bird. Refine distinct signatures/mix if seven fails; keep the seven-bird release path gated rather than reducing accessibility or adding a visible identification stat. Analyze these synthetic listening tasks only, not production listening behavior.

Reduced-motion review must see a living series of still poses, not static birds. Screen-reader review must hear paced connected observations, not an event list. Caption review compares every phrase with its realized call descriptor and checks legibility at narrow width with seven birds. Voice/interaction QA explicitly rejects welcome toasts, reward vocabulary, care obligations, engagement numbers, and numerical trait leaks.

### 15.4 Traceability and evidence artifacts for implementation

| PRD | Principal implementation evidence |
| --- | --- |
| `product_brief.md` / `concepts.md` | Scope/copy review, presence truth table, identity invariants, no-gamification UI/API scan |
| `bird_engine.md` | Versioned simulation fixtures, drift properties, mood/perch traces, seven-bird call-identification review |
| `interactions.md` | Arrival/offer/listen/settle end-to-end tests, presence lease tests, sparse notebook fixtures |
| `aviary_layout.md` | First-frame filmstrips, responsive screenshots, three-zone layout bounds, top-bar and reduced-motion captures |
| `accounts_sync.md` | Transaction crash/concurrency tests, multi-device revision tests, privacy contract checks, export/deletion restore drill |
| `social_optional.md` | Invitation/replay/revocation tests, visitor write-denial suite, off-by-default notification test |
| `accessibility_perf.md` | Assistive-technology review, caption/audio checks, bundle/first-bird/frame/memory reports, live operational alarms |
| `non_goals.md` | Release surface inventory showing no game/care/social-network/native expansion |

Each feature's completion requires its user-visible demonstration plus the relevant invariants; a backend unit test cannot stand in for audio taste, accessibility experience, or first-frame performance. Store implementation evidence without private production bird data. No product evaluation is performed in this phase-1 deliverable.

## 16. Delivery sequence and rollout

Plan for roughly 10–12 engineering weeks, with a minimum three-week longitudinal pilot overlapping the middle stages. Staffing assumption: two client engineers, one backend/identity engineer, one simulation engineer, one audio engineer, and shared visual design, accessibility/QA, and operations capacity. This is a sequencing estimate, not a commitment to skip gates if staffing differs.

| Milestone | Executable work and dependencies | Exit gate |
| --- | --- | --- |
| M0 — contracts and fixtures, week 1 | Freeze decisions in section 2; define snapshot/event schemas, privacy allowlists, species/signature assets, synthetic clock/data fixtures, reference devices, and copy rules | A signed-off implementation contract, sample snapshots within size target, and a testable two-bird storyboard including reduced motion/narration |
| M1 — persistent living slice, weeks 2–3 | Account activation, atomic two-bird creation, persistent minute tick, first SSR SVG scene, client interpolation, basic procedural calls, caption/narration baseline | Open/close/return without resetting identity or mood; preliminary mobile first-bird and old-laptop frame gates pass; no recorded audio |
| M2 — relationship loop, weeks 4–5 | Exact presence, drift/filter persistence, arrival decisions, listen/offer/settle state machines, bird-to-bird behavior, daily mood/weather, initial notebook composer | Concurrency/idempotency/monotonicity suites pass; every gesture works by keyboard; start 21-day longitudinal pilot |
| M3 — full private account, weeks 6–7 | Multi-device failure paths, complete settings/rename/adoption, exports/deletion/recovery, private telemetry boundary, visitor grants and log, opt-in visit mail | Two-device/visitor isolation, token lifecycle, privacy contract, and deletion/backup restoration tests pass |
| M4 — perceptual and scale hardening, weeks 8–9 | All six species, three/five/seven-bird fixtures, 30-minute memory/audio soaks, call identification, responsive/caption collision passes, full assistive-tech review | All v1 accessibility/performance budgets pass at seven; pilot confirms drift timescale; narration/notebook voice approved |
| M5 — staged release, weeks 10–12 | Operational drills, capacity/admission limits, synthetic geography probes, support copy, staged public cohorts | No unresolved data-loss, privacy, cross-account access, inaccessible primary flow, or failed headline performance gate |

API schemas and fixtures are the interface between these work areas. Frontend can render compiled fixtures while simulation is implemented; audio and captioning share the realized descriptor before either is finalized. Identity/privacy work cannot be deferred until after adding real accounts. Persisted drift and deletion correctness block any public release.

Bird-count ramp is quality validation, not an attention reward: internal synthetic/staff test aviaries progress two → three → five → seven, measuring density, frame time, narration load, caption placement, and recognition at each step. Public accounts always start with two and follow their true creation-age eligibility. Never accelerate a user's aviary age because they entered a beta cohort, visited frequently, or supplied useful feedback. Ship the complete seven-bird capability before normal age-based eligibility can reach it; feature flags may pause new adoptions during a fault but must never remove existing birds or reduce their identities.

Public traffic ramp: invitation-only small pilot, then approximately 1%, 10%, 50%, and 100% of eligible sign-ups, with at least 24–48 hours of healthy operations between steps and longer where a failure needs investigation. Admission is bounded by measured unattended tick capacity as well as visible request capacity. Use operational error/performance/security gates and qualitative product review, never retention, time spent, offer frequency, streak-like metrics, or average trait growth as ramp criteria.

At every stage, synthetic probes and privacy-safe operational telemetry are active. Keep kill switches for new invitations, visit emails, and new adoption acceptance. The base scene, accessibility surfaces, and existing bird records cannot be selectively turned off as a shortcut to passing a gate. A faulty simulation deploy is paused/rolled back to a compatible worker while canonical state is preserved; do not compensate by writing approximate vectors. Communicate actual system failures in the direct system register.

Before broad availability, rehearse tick-worker outage/recovery, database failover, delayed queue processing, failed mail delivery, export object expiry, mass visitor-token revocation, and a hard-deletion backup restore. Test supported-browser updates before moving the rolling last-two-version window. Pin old bird call grammar/signature compatibility until migrations demonstrate preserved recognizability.

## 17. Risks, mitigations, and final readiness criteria

| Risk | Likely failure | Mitigation and stopping condition |
| --- | --- | --- |
| Drift too fast/slow | A session feels like leveling up, or three weeks feels unchanged | Persistent low-pass filter, capped inputs, synthetic week-scale trajectories, matched-mood perceptual tests, real-time pilot; block release until the intended timescale is observed |
| Saturation erases individuality | All older birds converge to one expressive behavior | Stable species/identity signatures, distinct seeded baselines, asymptotic drift, mood/pose diversity independent of trait maximum; test year-scale fixtures without inventing negative drift |
| Presence inflation | Background tabs, duplicate devices, retries, or stale focus make birds change too fast | Exact conjunction, interval union, bounded leases/segments, dose saturation, replay tests; no foreground-only shortcut |
| Presence exclusion | Touch/assistive input lacks the stipulated activity events | Physical-device/assistive testing, explicit keyboard-press handling, documented evidence; don't secretly loosen the definition or introduce invasive tracking |
| Lost or duplicate personality | Client overwrite, tick retry, stale replica, migration reseed | Tick-only write role, transactional cursor/delta/RNG update, revision-fenced reads, property tests, backup/restore and migration fixtures |
| Stale or inconsistent reactions | Immediate cues diverge from next-tick decisions across devices | Persist server-authored outcomes at acceptance; ordered presentation revision; tick consumes the same outcome rather than rerolling; cue deduplication and supersession |
| Audio uncanniness | Repeated loops, shrill transients, density, indistinguishable birds | Procedural identity grammar, bounded variation, controlled chorus starts, blind identification, device listening review; no recorded fallback |
| Inaudible first frame | Browser autoplay/hardware blocks sound | Honest silence/captions plus explicit sound affordance; retain motion and narration; never fake successful audio or block entry |
| Accessibility regression | Narration floods queues, captions collide, reduced motion freezes the scene | Shared fact/call descriptors, bounded narration, dedicated cross-fade renderer, seven-bird/phone checks, real assistive-user review in every milestone |
| Performance under real load | First bird exceeds 500 ms, 30-minute session leaks, unattended ticks lag | Small SSR/bootstrap, fixed pools, measured reference hardware, long CI soaks, capacity/admission limits, separate tick-lag alarm |
| Privacy leakage | Emails in logs; event history reaches metrics/ML; export link leaks | Synthetic UUIDs, one encrypted identity store, token redaction, collector allowlists/no DB access, lifecycle tests, no behavioral SDKs |
| Export contradicts hidden traits | Requested JSON becomes a way to inspect numeric traits | Explicit isolated portability exception and no normal-UI exposure; acknowledge the conflict rather than claiming both literal rules are met |
| Notification exception spreads | Visit opt-in grows into reminders/engagement loops | Narrow consented email job only, no onboarding prompt or badges, surface inventory at release, no general campaign infrastructure |
| Deletion is incomplete | Backups or jobs resurrect an account or private vectors | Per-account key erasure, deletion journal replay, idempotent sweeper, job lifecycle fence, tested restore beyond deadline |
| Notebook feels generated | Generic/false observations or entries every session | Evidence-backed local composer, novelty/cooldown rules, immutable prose, editorial samples; no population-trained generation from private history |
| Feature drift | More birds, richer geography, counters, shared presence undermine the concept | Hard seven-bird cap, fixed three-zone scene, server-denied visitor writes, explicit non-goal review; decline scope expansion within v1 |

V1 is ready only when an owner can meet two birds, return across devices without identity/mood resets, observe truthful attention-driven change over weeks, interact without care obligations, hear or read recognizable procedural calls, and use every primary flow through the supported accessible surfaces. The same implementation must continue ticking when nobody visits, keep visitors from changing the host's birds, honor deletion/export commitments, stay within all performance budgets, and avoid collecting private relationship data for aggregate analysis. Remaining work is implementation and validation of this plan; phase 1 ends with this document and its runtime metadata.
