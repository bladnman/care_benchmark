# Pocket Aviary — v1 implementation plan

## 1. Basis, scope, and acceptance invariants

This is an implementation plan for a separate engineering team, not a product implementation. It is based on all ten supplied PRD files: 1-START_HERE.md, product_brief.md, concepts.md, bird_engine.md, interactions.md, aviary_layout.md, accounts_sync.md, social_optional.md, accessibility_perf.md, and non_goals.md. No external design-system document was supplied; producing the visual tokens and checking them against the requirements below is part of the work.

Ship a browser-based aviary with one account, one canonical scene, two starter birds, and an eventual maximum of seven. The experience comprises continuous bird behavior; procedural calls; a varied return-greeting; honest presence accounting; listen-in, offer, and settle; a sparse, permanent field notebook; email magic-link accounts; multi-device access; private, revocable visits; and complete accessibility surfaces. Personality changes should become instrumentally measurable after approximately one week of regular attention and perceptible after approximately three weeks.

V1 includes six species, editable bird names, age-based adoption beyond the initial pair, local-time lighting, occasional weather, per-device session revocation, verified email changes, account export, and recoverable deletion followed by hard deletion. Accessibility, privacy controls, and performance requirements are release requirements, not follow-up enhancements.

Exclude native apps, payments, passwords, SSO, multiple aviaries, shared ownership, configurable scenes, arranging birds, panning, scene scrolling, zoom controls, catalogs of starter birds, rarity, hunger, illness, death, distress caused by absence, custodial tasks, achievements, counters, scores, streaks, levels, badges, visit calendars, public profiles, discovery, rankings, follows, comments, chat, avatars, co-presence, and unsolicited reminders. Do not build underlying engagement rankings or cross-account bird statistics that could later expose these features.

Engineering acceptance invariants:

| Invariant | Consequence for implementation |
| --- | --- |
| The bird remains the same bird | Stable bird UUID, persistent vector and call identity, migration continuity, tested restores; no regeneration from interaction history. |
| Absence is acceptable | No negative personality delta, no absence-induced distress, no goodbye requirement, and no copy about missed visits. |
| Presence is the principal input | Credit only visible AND window-focused AND recently active owner intervals; union simultaneous device intervals before computing drift. |
| The server owns the simulation | Clients submit events and render projections; only the simulation service creates or changes personality vectors. |
| The place was already moving | Initial and resumed frames sample existing action phases; no ordinary entry sequence, spinner, wake-up, or fade from a static scene. |
| Noticing is the welcome | A bird's varied response is the entire arrival surface. No welcome text, toast, badge, banner, or absence counter. |
| The relationship is private | Simulation data has no analytics, training, recommendation, or third-party export pipeline. |
| Accessible modes carry the same experience | Shared state and call descriptors drive visuals, prose, reduced-motion poses, and captions from the first public release. |
| The scene remains comprehensible | All birds visible in one scene, three perch zones, seven-bird hard cap, restrained calls and visual density. |

## 2. Explicit implementation decisions and PRD tensions

These decisions make otherwise ambiguous requirements executable. They are recorded as decisions, rather than left as questions for the implementers.

1. **Numerical personality and export.** The absolute numerical non-exposure rule in concepts.md/bird_engine.md conflicts with accounts_sync.md explicitly requiring current vectors in a downloadable JSON export. This plan honors the specific export requirement as one narrow exception: an explicitly requested private account archive contains the raw current vectors. Ordinary snapshots, settings, narration, captions, notebook entries, error payloads, and user-accessible debug surfaces never expose them. This does relax the literal absolute rule for the archive; do not claim both incompatible statements have been fully satisfied. No vector dashboard or export-based visit history is added.
2. **Optional visit notifications.** The broad refusal of aviary notifications conflicts with social_optional.md's explicit opt-in toggle. Implement only the specific exception: an off-by-default setting can send one quiet transactional email when an invite is first used. No push infrastructure, onboarding prompt, toast, badge, re-invitation, reminder, or return invitation. Auth, invite, verification, and requested-export emails are transactional system actions. The toggle does not authorize any other aviary emails.
3. **Four top-bar icons and settle.** Keep exactly the four named icons: account/settings, accessibility, notebook, and offer. Put a separately named settle command in the offer popover, separated from the three gifts. This keeps settle reachable from the top bar without introducing a fifth permanent icon or a scene button.
4. **Scene chrome versus access.** Opted-in call captions and visible keyboard focus outlines are the explicit accessibility exceptions to the otherwise chrome-free scene. No persistent bird labels, mood icons, hover tooltips, or numerical overlays.
5. **One local clock across devices.** Store one account-level IANA timezone, initialized from the first browser. Every device and visitor renders that zone. A device detecting a different zone can offer a matter-of-fact change in settings; it never silently changes the aviary clock. The user can deliberately change the account zone while traveling.
6. **Slow tick versus immediate interaction.** Run a scheduled simulation tick every 60 seconds whether anyone is connected or not. Owner arrival, offer, settle, and adoption can wake the same serialized simulation worker for an earlier pass. This is still a server tick, not a second client simulator. Elapsed-time integration makes additional passes unable to accelerate drift.
7. **Audio on first encounter.** Browsers can refuse sound before a gesture. Attempt playback only under browser permission rules; when blocked, show the already-moving aviary with captions and allow the first suitable user gesture or sound setting to resume audio. Do not pretend audible calls before permission can be guaranteed. No welcome modal is introduced to obtain it.
8. **Keyboard listen-in.** User-directed bird focus engages listen-in as interactions.md requires. Enter explicitly engages it and is idempotent if already engaged; Escape releases it. Arrow movement transfers attention to the new bird. Pointer/touch activation on the already listened-in bird toggles it off. Thus Enter does not accidentally undo the listen-in that keyboard focus just engaged.
9. **Settle scope.** Settling changes a canonical scene override, so owner devices and visitors see the same evening state. It ends current owner presence windows. A new deliberate owner action resumes the scene; visitor actions cannot do so. The override also ends when its originating view closes or its view lease expires. Mute and listen-in gains remain device-local.
10. **Sound and accessibility do not penalize birds.** Sound output settings are not negative drift inputs. Attention to a bird through captions or narration receives the same listen-in credit as audible attention. The detailed drift specification governs the brief's broader suggestion that mute behavior can shape the relationship.
11. **Visit lifetime and history.** Unused links expire after 30 days as specified. Choose an eight-hour, non-renewable maximum for a consumed visit session; another visit requires a new deliberate invitation. Approximate duration measures the visitor's visible render session only. Revocation removes the invitation from active/outstanding rows; completed visits remain in the historical log. Interpret the PRD's “absence” confirmation as removal from the active list, without an added success notification.

All other numerical values below are initial engineering parameters, versioned in configuration. They are not user-facing goals or counters. Changes must preserve the PRD's invariants.

## 3. Service shape and module boundaries

Use a TypeScript application with server-rendered HTML, a small browser runtime, PostgreSQL, and a separately deployed worker process that shares a pure simulation package with the HTTP service. Keep v1 as a small number of deployables rather than a collection of independent domain microservices.

| Component | Owns | Must not own |
| --- | --- | --- |
| Browser shell and SVG scene renderer | Focus, view lifecycle, input capture, interpolation, local audio mix, captions, accessibility preferences | Canonical mood transitions, drift, adoption eligibility, or a writable vector |
| HTTP/auth service | Authentication, authorization, schema validation, event append, state reads, account commands, visit grants | Client-proposed personality writes or unserialized simulation updates |
| Simulation worker | Due ticks, ordered event consumption, vectors, moods, canonical action schedules, notebook observations | Analytics emission containing bird state |
| PostgreSQL | Canonical account and aviary records, event ordering, durable timers, outbox, identity and sharing permissions | Publicly readable simulation tables |
| Delivery worker | Requested transactional mail, export generation, deletion lifecycle | Bird-event analytics, mail bodies derived from presence history |
| Trusted edge delivery layer | Static assets and authenticated initial response delivery | Shared caching of private HTML, snapshots, invites, or exports |
| Operational collector | Allowlisted aggregate performance and failure metrics | A read credential to the simulation database |

Suggested source modules for future implementation: domain types and projection schemas; simulation/drift; simulation/mood; simulation/actions; simulation/call-grammar; simulation/notebook; persistence/transactions; auth; visits; lifecycle; web/scene; web/audio; web/accessibility; and synthetic QA fixtures. Call-grammar descriptor expansion is shared with the browser, but drift and hidden vector models are server-only and excluded from browser dependency graphs.

Use PostgreSQL row locks and durable due-work records for coordination. A database notification can wake a waiting HTTP response or worker, but it is only a hint: due work and committed versions remain queryable if the notification is lost. Do not require Redis, a streaming platform, or a second state store at initial scale.

Serve fingerprinted visual modules and motif definitions from a CDN. Serve private initial HTML through an authenticated edge request that reads the primary canonical projection. Set private HTML/state responses to no-store and never use a public cache key for them. A small, versioned server-side projection cache may be used inside the trusted application only; authorization is checked independently on every request, especially every visitor pull.

The normal data path is:

1. Authenticate the browser; read its one canonical aviary and current projection.
2. Return initial HTML containing the tiny scene representation, current action phases, and an escaped bootstrap snapshot.
3. The browser begins already-in-progress rendering, submits an owner arrival event, and opens a bounded snapshot long-poll.
4. An interaction is validated and appended durably. A worker wake-up accelerates response where appropriate.
5. The worker consumes the committed event prefix, updates state, persists a new projection and version, and publishes a version-change hint after commit.
6. Visible clients pull the new version and interpolate. Hidden clients stop rendering and pulling; server work continues.

## 4. Persistent model and integrity rules

Use synthetic UUIDs for accounts, birds, sessions, invitations, and internal references. Server timestamps are UTC; store the account timezone separately. Version schemas and engine configuration explicitly. Use foreign keys, account scoping, unique constraints, and row-level authorization as complementary protections.

| Record | Principal fields and constraints |
| --- | --- |
| Account | account_id UUID; one encrypted email envelope containing verified and, temporarily, pending address; verification state; timezone; locale; settings revision; deletion timestamps; notification opt-in default false |
| DeviceSession | session_id UUID, account_id, random-token digest, issued/last-used/expiry timestamps, revoked_at, coarse user-readable device label |
| AuthChallenge | challenge_id UUID, account_id or temporary sign-in challenge reference, random-token digest, purpose, expires_at, consumed_at; no email in URL identifiers or logs |
| Aviary | aviary_id UUID, account_id UNIQUE, created_at, canonical_version, engine_version, last_simulated_at, next_tick_due_at, committed event cursor, daily mood anchor, weather state, settlement epoch and originating view |
| Bird | bird_id UUID, aviary_id, immutable species_id, user-editable name, adopted_at, immutable identity/signature seed, grammar compatibility version, persisted five-trait vector, five filtered input values, mood and mood timing, perch/action state, offer eligibility time |
| InteractionEvent | aviary_id, ordered sequence, event_id UUID, authenticated device/view references, client sequence, type, validated payload, received_at, effective interval when applicable; append-only until retention/deletion purge |
| PresenceCoverage | private per-aviary recent interval union and rolling time buckets; bounded duplicate tracking; no pointer coordinates, key content, or population export |
| ViewState | view_id, account/session, visibility episode, view lease, last input evidence time, last client sequence, current listen-in target and terminal markers |
| CanonicalAction | action_id, bird_id, type, start/end times, from/to perch and pose descriptors, seed, call descriptor or reaction facts; bounded rolling horizon, not an unbounded animation log |
| NotebookEntry | entry_id UUID, aviary_id, occurred_at, stable pagination sequence, referenced bird UUIDs, names-at-observation, immutable prose, template version, factual provenance key; no user editing endpoint |
| AdoptionOffer | aviary_id plus age slot UNIQUE, stable proposed species/identity seed, availability time, optional dismissal state, accepted bird UUID |
| Invitation | invite_id UUID, host account UUID, encrypted recipient address, random token digest, issued/expiry/consumed/revoked timestamps, access epoch |
| VisitSession / VisitRecord | opaque session token digest, invite UUID, started/last-visible/ended times, coarse duration; entirely separate from owner PresenceCoverage and InteractionEvent |
| ExportJob | job UUID, account UUID, requested/completed state, snapshot version, encrypted object reference, download-token digest and expiry |
| DeliveryOutbox | job UUID, account/invite/challenge reference, template/purpose, dedupe key, retry state; references addresses rather than copying them into queue payloads |

The account's encrypted email is an authentication/contact attribute, never an account identifier, partition key, log dimension, or inter-service routing value. If exact-address lookup needs a blind index, confine a keyed lookup tag to the authentication table and rate-limit store; it is not a reference used by other services. Encrypt the pending address within the same account envelope until verification finishes. Invitation recipient addresses are the separate contact data required by the sharing feature, stored encrypted once on each invitation and exposed only to its host's settings/log.

Persist personality as finite, normalized server-side values, initially in [0.20, 0.60], with valid range [0, 1]. Only the simulation database role can insert initial vectors or change them. HTTP handlers cannot update these columns. Names, mood, and call identity are not reconstructed from the vector. A rename changes only the name and revision.

Maintain the vector, filter memory, consumed event cursor, RNG/action counters, and canonical projection in the same transaction. Backups must preserve all of them together. Never rebuild a vector by replaying retained events: event retention is intentionally shorter than the bird's life.

Retention defaults:

- Keep raw owner interaction events for seven days after successful processing; never purge unconsumed events. Keep recent interval-union data for 48 hours and current filter/input summaries as needed by the live engine.
- Keep bounded, per-account notebook evidence long enough for its predicates, normally eight days. Preserve generated notebook entries indefinitely until hard account deletion; no archive boundary in the UI.
- Keep invite/visit records for host transparency while the account exists, with encrypted recipient fields. Expired/revoked links never become active again.
- Retain account-scoped operational errors only briefly, initially seven days, and make targeted erasure possible. Aggregate metrics contain no account identifier.
- Remove expired auth secrets and export objects promptly; retain only minimal idempotency/status data while it serves a defined function.

## 5. HTTP contracts, commands, and error handling

Publish schema-validated, versioned JSON contracts. Apply authorization before conditional responses, before opening a long-poll, and again immediately before returning its result. Never trust an account_id or actor kind supplied in a request body; derive ownership from the authenticated session.

| Method and route | Behavior |
| --- | --- |
| POST /api/auth/link | Request a 15-minute, single-use sign-in link; return the same matter-of-fact accepted response whether an address already exists or not |
| POST /api/auth/consume | Atomically consume the link, initialize a new account if needed, issue a device session |
| GET /api/account; PATCH /api/account/settings | Read/update timezone, preferences, and explicit visit-notification setting using a settings revision |
| GET /api/account/sessions; DELETE /api/account/sessions/:id | List device sessions and revoke one |
| POST /api/account/email-change; POST /api/account/email-verify | Stage and verify a new address before switching the canonical address |
| POST /api/account/deletion; DELETE /api/account/deletion | Start soft deletion or recover during the 30-day window |
| POST /api/account/exports; GET /api/account/exports/:id/download | Queue a version-consistent JSON archive and redeem a short-lived, authorized download link |
| GET /api/aviary/snapshot | Fetch the current presentation projection; optional known_version and bounded wait_seconds support long-polling |
| POST /api/aviary/events | Submit a small batch of owner interaction events; return per-event durable receipts and processing status |
| GET /api/aviary/events/:event_id | Retrieve a prior receipt/result after an uncertain request outcome without resubmitting a new gesture |
| PATCH /api/aviary/birds/:id/name | Rename the same UUID using expected metadata revision |
| GET /api/aviary/adoption; POST /api/aviary/adoption/:offer_id/accept | Read the one current age-based offer and accept it idempotently |
| GET /api/aviary/notebook?before=cursor | Read immutable entries, newest first, using stable keyset pagination |
| POST /api/account/invitations | Deliberately create one invitation to the entered email address |
| GET /api/account/invitations; GET /api/account/visits | Read outstanding/active grants and historical visit log |
| DELETE /api/account/invitations/:id | Revoke an outstanding or consumed invitation and its access epoch |
| POST /api/visit/consume | Exchange a valid unused invite token for a render-only session |
| GET /api/visit/snapshot | Read the same canonical scene projection through the visit grant |
| POST /api/visit/liveness; DELETE /api/visit/session | Update coarse visit duration or end the render session; never call the owner event API |

Snapshot envelope: schema_version, canonical_version, server_now, simulated_through, timezone, scene lighting/weather/settle descriptors, per-bird stable identity and name, effective appearance, current mood-derived action descriptors, bounded future call/action timeline, adoption availability for owners only, and capabilities. Ordinary snapshots exclude vectors, filter values, presence totals, visit frequency, raw events, hidden trait debug names, account email, and notebook history.

Appearance contains actual render instructions such as palette colors and pose geometry rather than normalized trait values. Mood enums may be internal presentation inputs but are never shown as status labels. Visitors receive exactly the same bird/environment projection, with owner-only capabilities and private account fields omitted. They do not receive the notebook.

An owner event carries event_id, view_id, monotonically increasing client_sequence, observed canonical/settlement epoch, event type, and an allowlisted payload. Supported types are arrival, presence_interval, listen_start, listen_end, offer, settle, undo_settle, resume, and view_end. The server supplies event order and authoritative timing. Offer payloads identify seed, song-fragment library item, or still pool; the server chooses the receiving eligible bird. No event type accepts a trait, absolute mood, perch assignment, or arbitrary call parameters.

The receipt distinguishes accepted, duplicate, processing, applied, and rejected. For an accelerated action, wait briefly for the tick and include its new snapshot/result if ready; otherwise return 202 and let the existing snapshot pull deliver it. Never manufacture a successful bird reaction merely because a request was sent.

Use a unique constraint on aviary_id plus event_id. Identical retries return the original result; the same ID with a different payload is rejected. Reject unknown fields, invalid bird ownership, excessive batch size, stale action expiry, and visitor credentials on owner routes. A consumed offer result is not played again when its receipt is fetched.

Bound snapshot long-polls to 30 seconds for owners and 15 seconds for visitors, with at most one in flight per visible view. A committed canonical version wakes them early. A timeout returns an unchanged-state response with server time. Do not hold a database transaction or connection for the wait; re-query on notification/deadline. Account or visit revocation wakes the waiter and returns an authorization failure instead of stale data.

System failures use direct language in the system layer above the scene or the relevant form: expired link, timed-out session, unavailable visit, or aviary loading failure. A conflicting rename/settings edit returns 409 with current metadata so the user can review and retry; it does not overwrite silently. Unknown outcomes are retried with the same idempotency key. There is no personality conflict picker.

## 6. Canonical simulation and multi-device correctness

### 6.1 Tick ownership and transaction

Schedule every live aviary independently of browser connections, initially every 60 seconds with a stable UUID-based phase offset to spread load. A due-work index supports selecting batches without scanning the whole account table. Multiple workers claim different aviaries using locked work rows; only one worker may advance a given aviary.

For each advance:

1. Lock the aviary aggregate and verify its lifecycle state and engine version.
2. Read the stored vector, filter memory, timers, action counters, current projection, and committed event cursor.
3. Read the next committed event prefix in order. Assign sequence numbers during event append under the same aviary ordering lock, so an uncommitted lower sequence cannot appear after a higher sequence has been consumed.
4. Apply each new event once, crediting only newly covered owner presence time and valid capped interaction inputs. Immediate commands can create action/mood impulses at their effective server time.
5. Advance completed logical 60-second drift intervals, daily mood anchors, weather, autonomous social behavior, and timers through the current server time. Early interaction passes do not add extra logical drift intervals.
6. Produce canonical nonnegative personality deltas, update the stored vector, and author a bounded future action/call timeline. Derive notebook candidates from facts, then apply its sparse publication policy.
7. Atomically commit updated state, consumed cursor, command results, notebook entries, next due time, canonical version, and presentation projection.
8. After commit, notify waiting clients/workers of the version. Losing this notification changes latency only; the next poll still reads the committed state.

No last-write-wins path exists for personality. A vector update is an additive server-authored delta applied to the vector locked in this transaction. Concurrent laptop/phone events cannot supply competing absolute values. A crash before commit retries the same cursor and state; a crash after commit sees the advanced cursor and does not apply the deltas twice.

Use deterministic random streams keyed by immutable bird/aviary identity, engine compatibility version, and persisted action counters. A retry produces the same result. Do not derive all behavior from wall-clock seconds alone, which could accidentally synchronize birds or repeat greetings. Preserve existing identity seeds and grammar compatibility on deployment.

### 6.2 Time, backlog, and continuity

Use UTC monotonic simulation boundaries, not browser clock time, for drift and event ordering. Resolve local time only for day/night and daily mood anchors. Timezone changes neither create extra drift time nor change adoption age. On daylight-saving changes, identify each daily anchor by local calendar date and guard against double execution.

If a worker misses ticks, process the missing logical intervals, including positive filter carry-over, weather timers, and mood transitions. Work in bounded chunks, initially at most 60 minute-steps per transaction, and immediately requeue the remaining backlog. Do not cap elapsed time by silently dropping the missing days; do not defer all inactive-account simulation until someone opens a tab.

At larger scale, deterministic batch computation may combine uneventful intervals analytically, but only after equivalence tests against minute-by-minute execution. Optimizing unattended aviaries must preserve actual server advancement and daily/weather boundaries.

Track simulated-through age separately from computation duration. Prioritize recovery when lag grows, including accounts with no connected clients. A stale server snapshot is marked with its timestamp; the client does not silently invent a replacement simulation. During a severe outage, retain the last valid scene with a matter-of-fact system status, then refresh when service returns. Preserve birds rather than resetting an account to recover availability.

### 6.3 Client synchronization

Each client maintains its highest applied canonical_version, clock-offset estimate, current authoritative action window, and a set of consumed presentation action IDs. Discard out-of-order responses. Use the server's version, not a client timestamp, to decide freshness.

Fetch immediately on initial navigation, hidden-to-visible transition, and render gaps over two seconds. Maintain the visible long-poll and reconnect with bounded jittered backoff. Abort background polling and all render/audio scheduling when hidden. On resume, discard expired call/action playback and interpolate into the current state; never play a day's backlog of calls or notebook announcements.

Clock mapping uses server_now plus measured request round-trip timing and a local monotonic clock. Slew small corrections; after a suspension or large correction, fetch a fresh state and reset scheduling against it. Wall-clock changes on a phone cannot advance the engine or create presence.

Render interpolation can choose a smooth path between authoritative poses, but cannot choose a new mood, perch destination, call, or personality. Ambient leaves and tiny decorative movement remain client ornaments. A bounded local idle pose may continue during a short network failure; new behavioral decisions and interactions wait for the server.

While offline, keep only a short in-memory retry queue. Presence intervals older than 90 seconds are discarded; offers and arrival actions expire after 30 seconds. Do not replay an offline session as fresh attention on reconnection. Resolve unknown outcomes by event ID before making a new action. Settings changes display their uncommitted state in the relevant form; they never masquerade as canonical success.

Concurrent commands follow committed event order. Rename conflicts use metadata revisions. Two adoption accepts lock the same aggregate and check both offer consumption and count. Two offers competing for the same bird see the same cooldown. Settle and resume are ordered against their settlement epoch. Device revocation is checked inside the event-append transaction so a revoked session cannot pass an earlier authorization check and commit later.

Propagation has network latency, but there are no divergent persistent device simulations to merge. All observed differences resolve by reading the newer canonical version, never by uploading client bird state.

## 7. Presence and attention accounting

### 7.1 Browser state machine

Start with a five-minute recent-activity window, deliberately generous to still watching. The predicate is:

    document.visibilityState == visible
    AND document.hasFocus() == true
    AND time_since_last_qualifying_input <= 300 seconds
    AND this is an active owner view, not settled or visiting

The first three terms are mandatory; the final exclusions prevent non-owner or terminal sessions from contributing. Record qualifying activity from trusted pointermove and keypress-equivalent keyboard events in this document. Use keydown to represent a physical keypress, including navigation keys, without collecting the key value. Synthetic/programmatic events do not refresh recency. A tap alone is not silently substituted for pointermove; touch pointer movement qualifies when the browser emits it. Test this explicitly on phones and with assistive keyboard navigation.

Listen to visibilitychange, window focus/blur, pointermove, keyboard activity, pagehide, and settlement/resume. Re-evaluate all predicate terms at every boundary and during a 15-second heartbeat. Accumulate only the observed qualifying interval since the last boundary. End the interval exactly on blur, hidden state, activity expiry, settle, or view close.

Do not require continual mouse movement: the entire remaining five-minute window after a qualifying input can count while visibility and focus remain true. Conversely, no initial arrival, open tab, sound playback, animation frame, notebook read receipt, or visitor session automatically creates presence.

Flush a partial interval on a terminal event when possible. If a browser disappears without a terminal request, unreported time is lost rather than guessed. A lease timeout bounds view state; it does not award presence up to its expiry. This favors a small undercount over manufacturing unattended hours.

### 7.2 Server validation and union

A presence payload contains a session-bound monotonic time interval, duration, and the minimum predicate evidence needed to validate the report; it contains no pointer path or typed content. Bind the monotonic clock to server time when the view is created. Each interval is normally at most 15 seconds, with a strict 20-second acceptance ceiling for timer jitter. Split long continuous attention into these bounded reports.

Reject intervals from visitors, revoked devices, deleted accounts, unknown view IDs, impossible sequence order, future time, expired input recency, or intervals crossing a recorded terminal boundary without clipping. Delayed delivery is accepted only within the 90-second retry grace and only for explicitly reported coverage, never an extrapolated heartbeat gap. Browser signals are a cooperative attention approximation, not proof of a person's gaze; do not add invasive tracking to pretend otherwise.

Credit presence as the union of intervals across all owner devices and tabs. For example, laptop 10:00–10:10 plus phone 10:05–10:15 produces 15 minutes, not 20. A duplicate event adds zero. Maintain a recent interval union and calculate the newly covered duration transactionally before updating rolling buckets.

Late, valid intervals enter the next unprocessed input calculation; do not recompute or subtract previously persisted personality. Because only new coverage is credited, retries cannot multiply prior input. Count no more than one second of total owner presence per second of wall time.

Listen-in duration is also server-bounded by a live view, qualifying presence, a valid start/target, an end or timeout, and deduplication. When devices attend different birds simultaneously, divide the shared attention time across those targets; the sum of bird-specific attention credit cannot exceed owner presence time. Do not double the dominant input by adding listen-in seconds to presence seconds.

Presence measurements remain private simulation state. They are never displayed as visit frequency, exported as a visit log, or forwarded to telemetry.

## 8. Bird engine implementation

### 8.1 Slow drift with explicit calibration

Maintain each stored trait p_j and its stored low-pass state s_j. Build normalized rolling 24-hour inputs from the validated coverage, not from click totals:

- P = min(unique presence seconds / 1200, 1).
- L_b = min(credited listen-in seconds for bird b / 600, 1).
- E_b = min(distinct eligible offers made near b / 3, 1).
- A_b = min(eligible offers accepted by b / 3, 1).

Twenty minutes of attention per day is a synthetic calibration fixture, not a recommendation to the user. Saturating inputs limit extreme sessions without ever subtracting from a bird.

Initial per-trait inputs:

| Trait | Input u_j |
| --- | --- |
| Boldness | 0.85 P + 0.05 E_b |
| Social warmth | 0.85 P + 0.10 L_b |
| Vocal frequency | 0.85 P + 0.10 L_b |
| Plumage saturation | 0.85 P |
| Curiosity | 0.85 P + 0.05 A_b |

Presence remains the dominant contributor even when all supplementary signals saturate. Offer exposure can influence boldness without acceptance; acceptance specifically influences curiosity. Settle contributes no slow-trait input. Mute, captions, reduced motion, screen-reader use, and absence contribute no negative input.

Use a seven-day low-pass time constant and an initial growth constant k_j = 0.008 per day. For constant input during a completed integration interval of duration dt in days:

    decay = exp(-dt / 7)
    s_new = u + (s_old - u) * decay
    integrated_signal = u * dt + (s_old - u) * 7 * (1 - decay)
    delta = (1 - p_old) * (1 - exp(-k_j * max(0, integrated_signal)))
    p_new = clamp(p_old + max(0, delta), p_old, 1)

Compute accurately for small dt, persist filter memory, and apply deltas only once. Use the same logical minute grid whether the worker ran once or was woken repeatedly by interactions. The continuous-time form permits later equivalent batching; it must not turn “one API call” into “one full drift step.”

With an initial trait near 0.4 and sustained presence input near 0.85, the illustrative curve gives roughly +0.01 after a week and +0.056 after three weeks. Exact simulated visit windows will vary slightly because rolling input ramps at the start. Establish numeric acceptance bands against the actual fixture before tuning visual mappings.

Proposed calibration acceptance:

- Seven days of regular synthetic attention produces at least a 0.005 detectable change in the presence-driven traits.
- Twenty-one days produces a detectable behavioral/appearance difference in blinded comparison clips across several bird seeds, without a stats surface.
- One isolated 20-minute session never changes a trait by 0.001 or more and does not visibly alter appearance in that session.
- Seven days of repeated offers cannot produce the same broad drift as seven days of quiet, qualifying presence.
- Two weeks without any fresh events never decreases any trait. A stored positive filter can continue to produce a diminishing positive tail during absence; that is the remembered effect of prior attention, not fabricated new attention.

Treat these as calibration gates, not a claim that numerical change alone proves charm. Tune k values, trait-to-pose/color mapping, and signature bounds on synthetic accounts and designed research scenes. Never compute population-average drift from production accounts.

### 8.2 Mood and behavior

Use five initial moods: wary, content, curious, drowsy, and alert. Store current mood, its start time, minimum dwell deadline, daily baseline seed, and bounded recent impulses. A typical ordinary mood lasts tens of minutes; start with a five-minute minimum dwell and a base transition hazard around one change per 30 minutes, modulated by context. Explicit interactions can produce a brief reaction immediately without constantly replacing the underlying mood.

At local 04:00, refresh the daily baseline once for that local date. This is a change in transition bias, not a forced jump to neutral. Mood continues across midnight, sessions, devices, and restarts. Recent interaction biases decay over approximately 10–45 minutes.

| Input | Behavioral effect |
| --- | --- |
| Morning | More alert/curious candidates, gradually brighter light |
| Evening/night | Greater drowsy probability, lower ordinary call rate, low/fluffed poses |
| Accepted seed/pool | Brief content bias; candidate approach/drink/bathe/watch action |
| Novel offered fragment | Investigating tilt, response phrase, or thoughtful quiet according to current state |
| Passing rain | Temporary call-rate multiplier below one, fluffed poses, mild mood effects |
| Wind | Small alert/wary bias depending on bird traits |
| Nearby alarm-shaped call | Bounded wary impulse to neighbors; decay without a permanent trait loss |
| Higher boldness/warmth | Front-perch and companion-response preferences; reduced wary transition weight |

Wary is a transient environmental response, never a punishment for not visiting. Recent interaction activation may decay so unattended birds become more ambient, but it must not lower the stored vocal-frequency trait, erase color, create distress, or suppress all baseline calls. Every owner return still gets one noticing response; long absence can make it quieter and more orienting.

At each tick, plan a rolling 120-second action window from the current state and timers. Author preen, scan, weight-shuffle, head-tilt, perch move, rest, call, and neighbor-response actions with start times, durations, seeds, and transition controls. Commit completed facts in time order. Started actions are immutable; re-plan future actions with a short safety margin so clients do not abruptly reverse an in-progress motion.

Perch choice is a weighted decision over front/middle/back influenced by mood, boldness, social warmth, and proximity to other birds. Enforce separation and visibility in normalized scene coordinates. The client may solve final pixel placement but cannot move a bird to a different semantic perch zone.

Bird-to-bird response probabilities depend on warmth and vocal tendency, not only on a viewer action. A call can prompt one neighboring reply; wary can spread mildly and then settle. Cap reaction chains and give a bird a refractory interval so alarm cascades cannot trap the aviary in agitation. Allow genuine overlapping calls, initially at most three at once, to preserve legibility and avoid an undifferentiated chorus.

### 8.3 Return-greeting

An authenticated owner arrival is tied to a new navigation or visibility episode. The server stores the last owner-view departure/expiry separately from presence; a visitor does not update it. If another owner device is already visible, absence is short rather than falsely measured from an old device session.

On arrival, choose one leading bird using stable identity preferences, boldness, social warmth, current mood, and absence length. Compose the greeting from parameterized gaze orientation, preen interruption, head inclination, optional step, and optional grammar phrase. Short absence leans toward a glance; long absence toward a gentle re-orientation or approach. Do not announce the measured absence in text.

Target leader onset 0.3–1.5 seconds after the first scene frame, and always within two seconds under the supported performance envelope. If another bird responds, stagger it by a seeded 0.4–1.8 seconds; do not make the whole aviary greet on cue. Consecutive greetings vary continuous timings, pose parameters, and phrase details, with a guard against repeating the previous complete descriptor. A small rotating library of greeting clips is insufficient.

Deduplicate retries by arrival episode. Coalesce genuinely simultaneous openings by the same owner into the existing brief greeting window, preserving one leader rather than stacking several welcomes. An ordinary focus regain on the same visible page can refresh state without manufacturing repeated arrivals. Hidden-to-visible return remains a real arrival.

### 8.4 Offers and settle

The top-bar offer popover presents seed, a small library of five composed song fragments, and still pool. Fragments are note/motif instructions synthesized by the same procedural audio system, not recordings or user-uploaded audio.

The server chooses one eligible primary receiving bird, weighted by proximity and curiosity; current listen-in can provide a small proximity hint but cannot force acceptance. A seed is a brief front-perch gesture; a pool lasts roughly a minute; a fragment plays softly. Other birds may observe, but one primary bird receives the offer's drift credit.

Enforce a 180-second cooldown per receiving bird across all offer kinds, whether the bird accepts, waits, or ignores it. This prevents repeated ignoring from bypassing the throttle. Cooldowns and daily input caps are independent protections. If all birds are within cooldown, the popover quietly indicates that the aviary is resting between offers; do not show timers, progress rings, scarcity, or punitive copy.

A curious/content bird may approach within seconds; a wary bird can wait longer; a drowsy bird can remain where it is. The server determines and schedules the response. A successful API request does not imply an accepted gift. The naturalist narration/caption follows the actual response.

Settle starts a four-second evening-light and call-level transition and records a settlement epoch. End all currently open owner presence/listen credit at that boundary. Canonical mood receives only a small temporary quieting bias. Older in-flight presence reports are clipped to the boundary; stale device reports cannot reopen presence.

Any click in the scene within five seconds sends undo_settle and reverses the transition from its current value. Provide Escape/Enter access to the same undo behavior. An owner action after that window sends resume before its requested interaction and restores ordinary local-time lighting. Do not roll back a whole aviary snapshot to undo settlement.

A pagehide from the originating view ends its override; a bounded 90-second view lease handles crashes. Settled visible views keep only liveness, never presence, until active re-engagement. The next genuine owner arrival clears an orphaned override. Closing without settling merely ends presence; it does not alter personality rewards, incur a penalty, create a notebook reproach, or require a recovery interaction.

### 8.5 Species, naming, and adoption

Design six coherent silhouettes and motif families, for example a warbler, finch, sparrow, wren, robin, and nightjar-like bird. The last remains occasionally active at full night even when most birds have closed eyes and low resting poses. Species differences are recognizable without a catalog or rarity label.

Initialize a verified new account's aviary and two different starter species atomically through the engine initializer. Show them as the birds that arrived; offer suggested editable names. Naming may be skipped using defaults. A repeat sign-in or network retry must not create another aviary or another starter pair.

Create additional adoption eligibility from aviary age only, with proposed thresholds of 90, 180, 270, 365, and 540 elapsed days: third through seventh birds. This gives a few-month-old aviary a third bird and a year-old aviary approximately six if accepted. It is unrelated to visits, gifts, payment, or drift.

Expose at most one pending adoption invitation at a time in the existing offer/settings flow, without a badge, countdown, completion meter, or “earned” language. Store its proposed identity/species so refreshing or dismissing cannot reroll it. Declining or ignoring it has no consequence; it remains available quietly. If an old aviary has several eligible slots, reveal the next only after the current one is accepted or revisited in the flow, without spawning multiple birds automatically.

Prefer unused species until the pool is represented; a seventh bird can share a species but has a distinct stable individual call signature. Acceptance runs under the aviary lock and enforces a hard count of seven. Names accept bounded Unicode text, are escaped everywhere, and can be changed in bird settings without modifying personality, mood, adoption age, grammar identity, or earlier notebook prose.

## 9. Frontend rendering and session surface

### 9.1 Scene construction

Use a compact SVG scene and DOM controls, with a small imperative animation controller. Server rendering can emit the initial bird shapes directly; the renderer then updates transforms, pose parameters, and a few palette properties without rebuilding the application tree each frame. This avoids needing a large game engine or GPU texture pipeline for seven birds.

Compose sky/background foliage, back perches, middle perches, front perches/birds, and sparse foreground ornaments. Keep three semantic perch zones even where artwork uses a high or low branch inside a zone. Bird silhouettes use a shared rig with species-specific head/body/wing/tail shapes and compact feather details. Preserve individual plumage and silhouette across moods.

Map normalized perch positions into a responsive safe rectangle that includes the entire silhouette, flight control points, caption margins, and focus outline. Use a constrained layout with staggered slots across the three zones. At narrow widths, compress perch spacing and scale the contained scene while preserving bird proportions; never crop a bird, scroll to find it, or let it fly outside the viewport. At wide widths, increase spacing without making birds unrecognizably small.

Test at 320 CSS-pixel width, narrow phone landscape, tablet, ordinary laptop, and wide desktop dimensions. Reserve at least 44-by-44 CSS-pixel effective touch targets where feasible, separate overlapping hit areas, and keep all seven birds individually keyboard reachable. Browser zoom remains available; do not add an application zoom affordance or disable user scaling. Notebook/settings panels may scroll internally; the ordinary aviary scene does not.

Target fewer than 600 live SVG/DOM scene nodes at seven birds, with fixed ornament pools and bounded caption elements. Read layout during resize/initial composition, not once per bird per frame. Use one requestAnimationFrame driver and compute pose from absolute action time; never accumulate animation time by counting rendered frames.

### 9.2 First frame and ordinary return

The authenticated HTML contains the current bird geometry and an inline minimal phase controller. Current actions have nonzero elapsed phases on arrival. Where possible, preinitialize browser animation timing before first paint so the first visible frame is already mid-preen, mid-scan, or mid-call, and motion continues immediately.

Do not wait for settings code, notebook data, fonts, audio permission, or the entire six-species library to draw the first bird. Ship system fonts and the current birds' essential SVG data with the HTML. Subsequent code attaches to the existing scene without clearing it or re-running an entry animation.

If delivery genuinely takes longer, use the quiet sky/field and at most faint ambient cues. No spinner, fake skeleton birds, progress percentage, or cached bird from another account. Once the authoritative scene arrives, show the current phase directly, without a fade from static. The quiet field is not counted as first-bird performance success.

The sole starter exception is first adoption: after the two names are accepted/defaulted, the real empty field can receive a soft fly-in to the initial perches. Persist a first-arrival marker so reloading never repeats it. Later deliberate adoption may introduce the actual new bird gently while the existing scene continues; it never restarts the other birds.

When returning from a hidden tab, position the scene at its new current phase before painting it. Do not animate the bird from yesterday's last rendered perch as though the intervening day just happened in a second.

### 9.3 Movement, lighting, weather, and chrome

Normal idle behavior blends small body shifts, scanning, preening, and head orientation from server-authored actions. Parameterized curves and per-bird offsets prevent synchronized idle loops. Wary birds scan from farther back; content birds preen; curious birds orient toward sounds; drowsy birds sit lower with fluffed feathers. Users infer mood without a tooltip or label.

Use gradual day/night color curves in the stored account timezone: dawn warming, midday brightness, evening warmth, and dim night. Do not connect to real-world weather services. Schedule synthetic rain roughly three times per week, lasting approximately 4–10 minutes, and occasional gentle wind. Its mood and call-rate effects are canonical and brief; no dramatic storms, thunder, snow, or weather tasks.

Client-only leaves/feathers use a bounded pool and slow randomized intervals. Foreground/background parallax is tiny and independent of pointer tracking; this is not a visual demo of depth. Hidden tabs cancel the render loop and clear scheduled ornaments; showing the tab samples a fresh phase without replaying missed particles.

Keep the palette in soft blues, greens, browns, and muted ochres. The designer supplies validated light/dark tokens for text and focus contrast. Do not encode mood or identity solely in color.

The four-icon top bar becomes nearly transparent after four seconds of pointer stillness. Movement or keyboard activity restores it immediately. Keep it fully visible whenever it contains keyboard focus, a popover is open, or an accessibility setting requests persistent controls. On coarse-pointer devices, keep a discoverable quiet bar rather than hiding controls behind hover. Fading does not remove controls from the focus order. Focused controls and visible user copy retain required contrast.

Bird activation is for listen-in only. Offers, notebook, settings, and settle are reached through the top bar. Bird naming and adoption management live in settings, with no floating edit buttons inside the scene.

### 9.4 Reduced-motion rendering

Respect the operating system preference before first paint. Store an explicit accessibility preference of system or reduced; reduced stays active if either system preference or explicit setting requests it. Preference changes take effect live without rebuilding the aviary.

Replace continuous pose micro-motion with slow cross-fades among still preen/scan/rest poses, initially 2–4 seconds per transition. Replace flights with a cross-fade between origin/destination perches. Remove leaf/feather drift and parallax completely. Retain day/night and settle lighting, slowed to approximately 6–10 seconds. The five-second undo opportunity still begins at the settle command, independent of the longer visual transition.

Calls, procedural variation, caption detail, drift, mood, notebook observations, and greeting identity remain intact. A greeting becomes a gentle change of gaze or pose with its call/narration, not an empty static fallback. Test this as a separately composed visual surface, not as “disable every animation.”

## 10. Audio and procedural caption pipeline

### 10.1 Shared call descriptors

Define a small grammar for each species with approximately 3–5 motif families: note contours, bounded pitch ranges, trill structure, envelopes, gaps, and noisy/harmonic components. Assign each bird an immutable individual signature seed that adjusts characteristic interval ratios, timbre, and pitch center within its species range.

For each scheduled call the server sends call_id, bird_id, grammar version, start time, duration bounds, motif choices/seed, and bounded expressive modifiers. The client expands that descriptor into a concrete note/envelope sequence at runtime, then sends the same expanded sequence to audio synthesis and caption generation.

Personality shapes cadence and participation; mood shapes timing, emphasis, and modest pitch/envelope changes. Constrain variation so a bird's interval/timbre identity survives weeks of drift and all five moods. Do not randomize the signature seed on load, rename, deployment, or mood change.

Every call varies its continuous timing and articulation. A synthetic variation test checks descriptor repetition across thousands of calls, but human listening is required to establish that variation sounds alive rather than merely random. A shared species must still permit recognizing two individual birds by ear.

### 10.2 Scheduling and mixing

Create one AudioContext per visible application instance when browser permissions permit, never one per bird or call. Prefer an AudioWorklet voice engine with a fixed pool, initially 16 voices, reusable envelopes and scratch storage, and no allocation in the audio render callback. Where AudioWorklet is unavailable but WebAudio works, synthesize through oscillator/filter/gain nodes; stop and disconnect short-lived nodes and reuse reusable buffers. This remains procedural sound.

Map canonical call time to AudioContext time. Schedule approximately 150 milliseconds ahead with a small timer and never depend on animation-frame cadence for sound timing. Maintain a bounded set of scheduled/played call IDs; refreshes cannot play a call twice. Cancel future unsounded actions when the canonical plan changes. Do not cut a currently sounding phrase for a minor snapshot update; finish its envelope unless muting, hiding, or revocation requires shutdown.

Mix each bird through its own gain and mild position pan, then a master bus with headroom and a limiter. Begin with ordinary per-bird levels around -24 dBFS, with a conservative ceiling and a master limiter near -3 dBFS. Treat these as sound-design starting points, not promised loudness on every device. Avoid harsh high-frequency energy, identical envelopes, hard stereo jumps, and abrupt gating.

Chorus behavior comes from overlapping independently synthesized phrases and server-authored neighbor responses, not layered recorded loops. Allow up to three simultaneous bird calls initially, with small staggered responses for further birds. A soft song-fragment offer uses reserved voices and a quiet mix position so it does not erase individual calls.

Listen-in smoothly raises the target, initially toward -18 dBFS, and lowers other birds toward -30 dBFS over about two seconds. Other birds remain audible. Disengagement returns all gains to ambient over roughly 2.5 seconds. Retargeting ramps from the gains actually reached, not from a reset value. Pointer toggle, empty-scene click, Escape, focus departure, hidden state, and settle all release the mix without a hard cut. A focused bird is never implemented as a solo/mute switch.

Settle adds a gentle master-level reduction and lets current phrases finish softly. Day/night and weather apply modest canonical call-rate changes; they never rewrite the vocal-frequency personality value. At night the nightjar-like signature keeps the place audibly alive at low density.

### 10.3 Failure and permissions

Handle missing WebAudio, denied/suspended contexts, resume failures, output interruption, and exhausted audio resources. The fallback is silence with captions on by default. No MP3/OGG/sample-loop fallback and no downloaded recordings of calls.

Expose sound availability and mute controls in accessibility settings using matter-of-fact language. Do not place an audio-error toast in the aviary. On an allowed gesture, retry resuming a suspended context once, using the current timeline rather than replaying calls missed during the block.

When hidden, ramp sound down briefly, stop scheduling, and suspend the context. On return, rebuild the small scheduling window from the fresh canonical snapshot. Repeated visibility changes must not create extra contexts, duplicate nodes, orphan timers, or click artifacts.

### 10.4 Call captions

Generate caption prose from the expanded runtime sequence: number of notes, contour, trill/held shape, pauses, intensity, and current perch context. Examples of the style are “a soft three-note rise” or “a low trill, then a pause.” Do not attach one stock sentence to each species or mood.

Caption onset follows the actual scheduled audible onset within approximately 100 milliseconds where audio is running. If audio is muted or unavailable, the same canonical call descriptor drives a corresponding silent caption; do not claim the user heard it. Captions are enabled by the user's preference and automatically by sound-unavailable fallback.

Place short text near the calling bird with a small contrast-safe backing and restrained fades. Reserve collision-free caption positions during layout; the initial three-call overlap bound makes this manageable. For narrow layouts, choose shorter equivalent phrases and place the captions in distinct nearby lanes without losing which bird is calling. Do not silently omit a caller to meet a layout budget.

Keep caption DOM and timing queues bounded. Provide a readable current-caption region for assistive technology, but avoid automatically duplicating every caption into the narration live region; the user can choose detailed call narration. Reduced-motion users keep the same text and call timing with gentler visual transitions.

## 11. Notebook, narration, keyboard, and visual access

### 11.1 Field notebook generation

Use a deterministic, curated natural-language template system in v1, not an external language model. It reads the account's own canonical facts and bounded observation summaries. This gives controllable voice, factual specificity, no inference about unseen user behavior, predictable cost, and no third-party transfer of bird interactions.

Candidate predicates include a particular bird greeting before another for the first time in a verified recent interval, a sustained preening episode, a quiet call pattern during passing rain, or a rare companion response. Require factual evidence for relational wording. “First time this week” needs complete relevant coverage of that week; if the evidence window is incomplete, choose simpler truthful wording.

Generate candidates during the tick, score local novelty, and suppress duplicates using an account-local semantic fingerprint. Start with a 72-hour ordinary-entry interval, a 24-hour noteworthy exception, and at most three entries in a rolling seven days. These limits apply even to heavily active accounts. A normal, regularly visited aviary should receive an entry every few days, not on every session.

Entries describe birds and place, never the user's frequency of visits, session start time, presence minutes, numerical drift, streaks, achievement, or absence. Use lowercase, present-tense naturalist prose, preserving specific bird and moment details. Avoid generic “your bird is happier” or machine-event wording.

Persist rendered prose and names-at-observation so renaming does not rewrite the observer's past. Store stable bird references separately for data integrity. The notebook is read-only: no edit, delete, annotation, or visitor-writing endpoint. Account deletion still removes it as part of deleting the entire account.

Use keyset pagination with a page size of 30 and a bounded client cache, initially three pages. Re-fetch older pages when necessary; do not hide or archive them server-side. Keep keyboard/screen-reader focus attached to the currently focused entry when pruning offscreen pages. Opening the notebook creates no celebration or “new entries” badge.

### 11.2 Running narration

Build narration from the exact projected state the scene uses. A shared naturalist lexicon supports the notebook, offer observations, caption wording, and narration without making them the same stream.

At idle, emit one compact prose observation around every 45 seconds, within the 30–60 second requirement. Describe a few salient birds, their visible actions, and light/weather rather than enumerating every mood/perch field. Do not narrate animation frames, decorative leaves individually, or raw internal state.

Return-greeting and user-initiated offer/settle reactions get a small priority bump and prompt observations. Use a polite, atomic live region with a queue of at most one pending idle observation and a short bounded action queue. Replace stale idle prose rather than building a backlog; do not interrupt a user navigating settings with repetitive narration. A greeting observation is bird behavior, never “welcome back.”

Provide narration controls in accessibility settings: enabled/paused, optional visible transcript, and optional detailed calls. A screen-reader user can hear the living scene without audio calls, and can use calls without competing redundant prose. The visible transcript receives the same contrast and text-size treatment as other user copy.

### 11.3 Focus and controls

Tab traverses the four top-bar controls and then enters one roving bird focus group. The first entry focuses the first bird; arrow keys move between birds by their stable scene order. User-directed focus engages listen-in; Enter engages idempotently; Escape releases; leaving the group restores the ambient mix. Empty scene clicks and repeated pointer activation of the active bird also release.

Expose each bird with an accessible name and a concise current naturalist description plus the listen-in action. Do not expose vector numbers in ARIA, DOM data attributes, tooltips, or a “debug accessibility” surface. Focus outlines are soft but high-contrast against morning and full-night backgrounds.

The offer popover is a keyboard-operable menu/dialog with explicit focus management. Provide an unmodified shortcut while scene focus is active, such as O for offer; do not intercept typing in text fields or browser/assistive-technology modifier combinations. Settle and its undo remain reachable without a pointer. Escape closes popovers and restores focus to their invoker.

Notebook and settings dialogs have correct headings, labels, focus containment, close controls, and focus restoration. Make sign-in, expired link, sync failure, revoked visit, export request, email verification, and deletion recovery usable by keyboard and screen reader. System error language is direct, without pretending that a bird is reporting the error.

Visitors can focus/read scene descriptions and adjust their own sound, captions, and motion preferences. They cannot engage listen-in or invoke any owner action; accessible controls do not create a hidden interaction path.

### 11.4 Contrast and access verification

All user-copy text must meet WCAG AA: at least 4.5:1 for ordinary text and 3:1 for large text; interactive boundaries and focus treatment must remain perceivable against the scene. Test captions over every day/night palette and weather state, not just a static white settings page. Verify text scaling and reflow, visible focus, touch targets, and no keyboard traps.

Automated checks cover semantic roles, missing labels, contrast tokens, prohibited numeric exposure, and focus transitions. Manual sessions on VoiceOver/Safari and NVDA/Firefox or Chrome cover actual prose pacing, keyboard listening, offered responses, and dialog navigation. Test reduced-motion mode as a full 30-minute experience and muted/caption-only mode as a complete product journey.

## 12. Accounts, visits, exports, and erasure

### 12.1 Authentication and identity lifecycle

Generate high-entropy random magic-link tokens, store only their digests, expire them after exactly 15 minutes, and consume them through a conditional transaction that succeeds once. Allocate a synthetic account/challenge identity before sending a link for a new address; keep any unverified address encrypted in the account/auth boundary and remove abandoned unverified records after a short retention window. Initialize the aviary only after successful verification.

Use a link landing page followed by an explicit confirmation POST so email scanners do not consume the token or create a visit. Put the secret in a URL fragment where practical, scrub it from the address bar before other navigation, and never load third-party assets on token pages. Use no-referrer policy and a strict content-security policy. Auth links, invite links, and export links have separate token purposes and cannot substitute for one another.

Issue Secure, HttpOnly, SameSite cookies for per-device sessions; never put session credentials in localStorage. Start with 30 days idle expiry and 90 days absolute expiry. Account settings show coarse device labels and last-used time, with revocation. Revocation invalidates queued/pending authorization, wakes relevant long-polls, and takes effect on the next protected request.

Rate-limit magic-link requests initially to five per address per 15 minutes, with a separate coarse abuse limit per network source. Use short-lived protected lookup keys, not email-bearing logs or long-term visitor fingerprints. Keep response wording non-enumerating. Rate-limit invalid consume attempts and invitation sending separately. Do not present these limits as behavioral rules of the aviary.

For an email change, store the pending address encrypted, issue a purpose-bound verification challenge, and leave the old address authoritative until verification commits. Check uniqueness and consume the challenge atomically. On success, preserve account/bird UUIDs and invalidate outstanding sign-in challenges issued for the old address. An expired verification leaves the old account intact.

Perform authorization on every object access, including notebook pages, bird renames, invitations, exports, and long-poll completion. Validate input lengths and HTML-escape names everywhere; never allow a bird name to become markup or a narration command. Apply CSRF protection to cookie-authenticated mutations.

### 12.2 Read-only visits

An owner deliberately enters an email and sends an invitation from account settings. There is no global discoverability setting, onboarding share prompt, default recipient, persistent friend relationship, or automatic repeat invitation.

Store a one-time token digest with a 30-day unused expiry. Consumption is atomic: the first valid confirmation creates a render-only session, marks the invitation used, and creates the host's silent visit-log entry. Subsequent uses of the invitation URL fail, even while the first visitor session remains valid.

The visitor session has only snapshot and coarse liveness capabilities. It cannot access owner event endpoints, notebook data, renaming, adoption, offers, listen-in, settle, or host presence. A visitor who also has a separate owner account cannot change the host's aviary by supplying its UUID.

Reuse the same scene/action projection and renderer. The visitor sees the current host timezone, light, rain, birds, moods, and scheduled calls. Do not add special plumage, celebratory behavior, arrival greeting, host-online status, visitor cursor, or co-presence indicators. Local sound/motion/caption preferences affect only the visitor's experience.

Track approximate visit duration through visible-session liveness, rounded to coarse minutes. Hidden time is not actively watched time; abandon a session after missing liveness without extrapolating an entire afternoon. This record belongs to the private host sharing log, not to simulation inputs or aggregate behavioral analytics. No visitor event changes the drift filter.

Revocation atomically marks the invite revoked, increments its access epoch, invalidates associated sessions, and wakes waiting reads. Reauthorize before every snapshot response, including unchanged responses. The visitor receives “This visit is no longer available,” stops rendering the private scene, stops audio, and clears its in-memory snapshot. At worst a disconnected client learns at its next pull; do not claim previously displayed information can be remotely erased.

Unused expired/revoked links and consumed links that cannot create another session use the same matter-of-fact unavailable surface. Revocation updates the host's active list inline without a success toast or mail. Old link secrets never regain validity if the owner sends another invitation.

If and only if the host has explicitly enabled visit notifications, enqueue one deduplicated email when an invite is consumed. Check the setting again before delivery. The default is silent logging; there are no visit badges or push messages. Disable mail-open/click tracking for these transactional messages.

### 12.3 Export

On explicit request, capture a repeatable-read JSON snapshot at one canonical version, including stable bird identities, names, species, current persisted vectors, current moods, notebook entries, and account settings. Include schema_version, generated_at, and canonical_version. The raw-vector inclusion is the narrow exception recorded in section 2; no normal application endpoint exposes it.

Exclude session secrets, magic-link tokens, raw interaction logs, presence totals, visit-frequency history, and hidden operational records. Do not expose another person's invitation contact data simply because an account export includes settings.

Generate the archive in a trusted worker, encrypt its temporary storage, and email only a download link to the currently verified account address. Require the matching signed-in account plus a short-lived random link, initially 24 hours, and serve it as a download without rendering a traits table. Remove the object on expiry, hard deletion, or job invalidation. Use no-store responses and prevent download tokens from entering access logs.

The mail provider receives only the delivery address and transactional link text, never the JSON archive or bird interaction data. Queue jobs reference synthetic IDs; resolve the verified destination just before sending.

### 12.4 Deletion and recovery

Starting deletion immediately sets deletion_requested_at and hard_delete_at = request time + 30 days. Put signed-in routes into a matter-of-fact recovery state with “I changed my mind” available throughout the window. Do not make birds plead, become ill, or describe the user's departure.

Stop new owner interaction acceptance, invalidate invitations and export links, and restrict existing sessions to recovery/account-lifecycle actions. Continue the stored aviary's unattended simulation during the recoverable period so recovery does not reset or reconstruct the birds. Recovery clears deletion scheduling and restores the same UUIDs, vectors, filters, notebook, and ordinary account access; it does not silently re-enable revoked sharing.

At the 30-day deadline, an idempotent purge workflow removes the account, birds, personality/filter state, all event/coverage records, notebook, invitations, visit records owned by that account, sessions, challenges, exports, caches, queued mail/jobs, and any UUID-linked operational records. Recheck deletion generation immediately before purge so a recovery that committed first cannot race with destructive work.

Encrypt account-owned sensitive state under account-scoped keys held in an erasure-capable key service. Backup and restore design must make a hard-deleted account unrecoverable: destroy its usable keys, remove active replicas/caches/objects, and expire retained encrypted backup material under a documented storage lifecycle. A restored backup must not resurrect data whose key no longer exists. Do not describe inaccessible residual backup ciphertext as an active retained account, and do not promise physical byte removal earlier than the storage lifecycle actually supports.

Choose storage/key retention that can meet the day-30 irrecoverability commitment before launch. Test hard deletion against every store, including restore from an older backup, rather than treating a deleted Accounts row as sufficient. If any vendor or key backup could restore the account after day 30, that storage choice blocks release.

## 13. Privacy boundary and operational observability

Separate simulation storage credentials from the metrics pipeline. The analytics/metrics collector has no query account against the simulation database, and the simulation worker has no general event-export connector. Disable default browser session replay, click tracking, heatmaps, third-party analytics SDKs, and payload-capturing error integrations.

Allowlist operational fields before emission. Strip full URLs, query strings, auth headers, request bodies, email, bird names/IDs, vector values, mood, call/action descriptors, invite recipients, offer types, and presence/listen timing. Collect route templates rather than literal identifier-bearing paths. The account UUID may appear in a narrowly scoped short-lived operational error record when necessary to diagnose an account failure; it is not a metrics label.

Collect from day one:

- Aggregate request count, latency, status class, service/build version, and broadly scoped region.
- Simulation computation duration, scheduler lag, retry/failure counts, oldest unprocessed work age, and queue depth, without per-bird inputs or account dimensions.
- First-bird-render time, navigation stages, long tasks, frame-time distributions, and coarse client memory health where available.
- Audio context/resume/synthesis error counts and underrun/resource failures, without bird or call details.
- Anonymous, coarse session-duration histograms, measured as page-view duration rather than presence-time and emitted without a stable session/account ID.
- Delivery/export/deletion job success/failure and queue age, with no content or recipient dimensions.

Aggregate timing samples into buckets and strip transport identifiers at collection. Avoid retaining raw per-navigation timelines that can be linked back to an account. Keep cardinality bounded by build, coarse browser family, and broad region; enforce minimum aggregation groups before displaying sparse combinations. Test the actual emitted payloads, not only a privacy policy document.

Never collect population-average drift, per-species interaction rates, most-offered birds, daily-active relationship charts, visit streaks, most-visited aviaries, recommendations, user rankings, or training examples from the production simulation. Calibration uses authored synthetic scenarios, not a hidden production data warehouse.

Synthetic monitoring runs fictional test aviaries from common geographies on a schedule, including authenticated first render, interaction response, supported browsers, and error surfaces. Keep these accounts separate from real-user data. Aggregate RUM verifies operational performance without reconstructing anyone's aviary.

Link a plain-text privacy policy from account settings naming the allowed aggregate categories and explicitly excluding per-bird interaction state. Emails and notification preferences follow the narrow explicit actions in this plan; no marketing or re-engagement pipeline is added.

## 14. Performance budgets and measurement

### 14.1 Load path

The hard initial JavaScript ceiling is less than 2,000,000 gzip-compressed bytes. Set a much smaller engineering target: at most 350 KB of initial JavaScript, with the pre-first-bird phase controller and essential scene code at most 100 KB. Lazy-load account settings, extended accessibility settings, notebook history, invitation management, and unused species detail. Never wait for them to draw the current birds.

Keep ordinary seven-bird snapshots around 32 KB JSON or less and approximately 10 KB compressed, using compact action/grammar descriptors rather than waveform data or full notebook history. Starter bootstrap state is smaller. Reject accidental serialization of event history or the server vector model into HTML.

Measure time to first bird from aviary navigation start to the first painted real bird, not to the quiet field, hydration completion, or a cache placeholder. The requirement is below 500 milliseconds on a mid-tier mobile device over representative 4G. Instrument both cold asset-cache and repeat-navigation cases; a warm-cache-only result does not satisfy it.

An initial budget allocation is approximately 180 ms for connection/edge delivery overhead, 100 ms for authenticated state/HTML delivery, 40 ms for the critical transfer, 50 ms for parsing/style/first bird paint, with the remainder as headroom. Validate rather than assume those components fit together; network round trips are the main risk.

Use physical mid-tier Android hardware, initially a Pixel 6a-class device, and a repeatable 4G profile around 9 Mbps down, 1.5 Mbps up, and 80 ms round-trip latency. Document the exact hardware/browser/profile in test artifacts. Include common geographic paths and unpooled navigation connections. Authentication email reading and naming are user actions outside this timing; the subsequent real aviary navigation is measured normally.

If the budget fails, prioritize smaller inline bird assets, reduced critical scripts, warm application capacity, efficient account/projection lookup, streaming authenticated HTML, and an appropriate primary-serving region. Do not “fix” the metric by rendering a fake bird, exposing private snapshots through shared CDN caching, or omitting the cold case. Networks outside the supported envelope use the quiet field honestly.

### 14.2 Runtime and memory

Target sustained 60 fps on a five-year-old mid-range laptop, including the seven-bird scene for 30 minutes. Start with a main-thread frame budget around 3 ms of scene scripting, 2 ms of style/layout, and 6 ms of paint/composition, leaving headroom within 16.7 ms. Track missed frames and long tasks as well as the average.

Use pooled ornaments, stable SVG rigs, precomputed curves, bounded action queues, and one animation driver. Audio pools, nodes, timers, event listeners, subscriptions, and contexts have explicit owners and disposal paths. Audio callbacks allocate no unbounded data; notebook pagination drops offscreen page references without breaking focus.

A CI soak runs two-, four-, and seven-bird scenes, audio on/off, repeated listen-in/offer/settle, notebook scrolling, resize, visibility changes, and reduced motion for 30 minutes. Compare retained heap and resource counts after warm-up and equivalent garbage-collection points. Acceptance is no sustained upward trend and no growing retained node/context/timer/page counts; a small measurement-noise allowance, initially 5 MB, is not permission for linear growth. Any consistent slope is investigated.

Hidden documents have zero scene animation frames and no active call scheduling. The server continues ticking. Verify CPU/battery relief and that reopening reconstructs current state rather than restarting a frozen animation.

### 14.3 Service health and browsers

Start with a typical seven-bird tick-computation target below 50 ms, excluding queue wait. The required alert is p99 tick computation latency exceeding five seconds. Separately alert on scheduler lag over two minutes, unconsumed event backlog, failed projections, or a worker that stops advancing unattended accounts. A fast tick executed an hour late is not healthy.

For accelerated interactions, target an event-to-authoritative-reaction-start p95 below 500 ms under ordinary service conditions; return-greeting remains within its one-to-two-second envelope. Measure receipt-to-tick-to-snapshot latency separately from visual interpolation.

Support the last two major versions of Chrome, Safari, Firefox, and Edge. Maintain a release-dated matrix and feature detection for rendering, cookies, and timing APIs. Missing WebAudio uses the supported silent-caption path; a genuinely unsupported browser gets a matter-of-fact browser-requirements surface. Do not ship legacy compatibility bundles that defeat the size budget.

## 15. Implementation sequence and release gates

The following are future engineering work packages, not actions performed by this planning run. Ownership names are team responsibilities; no benchmark evaluation or product implementation is part of this deliverable.

| Milestone | Ownership and concrete deliverables | Dependency and completion gate |
| --- | --- | --- |
| M0 — contracts and sensory prototype | Product/design defines tokens, six silhouette/grammar briefs, voice lexicon, reduced-motion compositions; engineering freezes projection/event schemas and decision records | Review first-frame behavior, captions/focus exception, numerical export exception, and notification exception before building divergent surfaces |
| M1 — durable two-bird vertical slice | Backend/engine delivers account initialization, scheduled unattended ticks, stored identity/vector/filter, projection reads, arrival events; frontend delivers SSR moving scene, one procedural signature, captions, keyboard and reduced motion | Two clients show one persisted aviary; restart/hidden return preserves it; first-bird measurement and accessibility smoke work from this milestone |
| M2 — full interaction and drift engine | Engine completes calibrated inputs, mood/weather, social calls, offers/cooldowns, listen accounting, settle/undo, age adoption, sparse notebook | Synthetic 7/21/absence-day fixtures pass; duplicate/crash/concurrent event checks preserve deltas and count caps |
| M3 — full audiovisual and accessible product | Frontend/audio/design completes six species, seven-bird layout, chorus, caption grammar, narrative queues, notebook pagination, all palettes and preferences | Human signature/variation review; VoiceOver/NVDA journeys; all viewport and reduced-motion scenarios acceptable |
| M4 — account and visit lifecycle | Backend completes real magic-link delivery, device revocation, email changes, invitations/revocation/log, optional notification toggle, export, 30-day recovery and hard purge | Cross-account authorization, one-use token races, read-only visits, consistent export, deletion/restore tests pass |
| M5 — operational hardening | Platform/QA completes aggregate telemetry, privacy payload inspection, worker failover, load profiles, soak CI, browser matrix and synthetic monitors | Performance budgets met, no memory growth, no telemetry relationship leakage, restore drill preserves identities and erasure |
| M6 — controlled release | Product/QA reviews lived pacing and copy; platform enables rollout in successive deployment cohorts | Complete v1 capabilities, support surfaces, verified rollback behavior and release gates below |

Work on audio, accessibility, and engine behavior begins in the first vertical slice so their constraints can change architecture while it is still inexpensive. Do not leave them until an otherwise finished UI needs retrofitting.

Bird-count ramp:

1. Develop and privately test the initial two-bird experience first.
2. Enable three/four birds in synthetic aged aviaries and internal release cohorts once mixing and responsive layout pass.
3. Enable all seven in synthetic aged aviaries and a controlled cohort; pass the full 30-minute soak and signature/accessibility review.
4. Launch public v1 with the age gates and seven-bird cap implemented and qualified. Every new public account still starts with two. Do not defer seventh-bird support until real accounts become old enough to discover its bugs.

Feature flags gate new adoption availability and code rollout, never remove or hide birds already adopted. Rollback must continue rendering existing seven-bird accounts and preserve all UUIDs/vectors. Do not restore an old database snapshot merely to roll back a visual or engine deploy; roll back compatible code/configuration around the current persisted state.

Use versioned migrations with expand/contract compatibility, saved fixtures of old bird records, and a forward repair path. Any engine-parameter change changes future deltas only; no backfill silently rewrites existing personality. Preserve grammar compatibility or explicitly migrate it with recognizability checks, never reseed the bird.

Public rollout can advance through small deterministic account cohorts, for example 1%, 10%, 50%, then 100%, after aggregate operational and accessibility checks. Cohort selection uses synthetic IDs and operational configuration, not relationship behavior or engagement scores. Hold or roll back on identity corruption, drift loss, privacy leakage, inaccessible flows, audio failures, or missed performance gates.

Release-blocking evidence:

- All hard product invariants hold in automated fixtures and human surface review, including no welcome/gamification/absence-guilt copy.
- The stored-vector and event-order model survives retry, concurrent devices, worker crash, failover, migrations, and restore.
- Synthetic drift curves meet the week/three-week intent; human reviewers can distinguish longer-term expression without seeing numbers.
- Every v1 account/visit/lifecycle route works with keyboard and supported assistive technology.
- Cold and warm first-bird tests, bundle gates, supported-browser runs, and seven-bird 30-minute soaks pass.
- Actual telemetry payloads contain only allowed operational fields, and hard deletion cannot be undone by restoring an old backup.

## 16. Verification matrix for the implementation team

| Area | Required scenario and expected result |
| --- | --- |
| Presence predicate | Exercise all eight combinations of visible/focused/recent input; only all-three true contributes time |
| Still attention | One input followed by five minutes of visible/focused watching qualifies until the exact recency deadline |
| Unattended tab | Hidden, blurred, overnight idle, and suspended documents create no invented presence; closing loses at most an unreported small interval |
| Input compatibility | Trusted keyboard/navigation and touch pointermove behavior work; programmatic focus/mouse events do not manufacture activity |
| Multi-device coverage | Overlapping laptop/phone time is unioned; same-target and different-target listen-in cannot multiply the shared attention budget |
| Event delivery | Duplicate, reordered, delayed, expired, and same-ID/different-payload events have defined results and never duplicate a delta/reaction |
| Tick transaction | Crash before/after commit, duplicate worker claim, and missed notifications preserve one vector update and durable progress |
| Tick cadence | More arrival/offer wake-ups do not accelerate drift; unattended accounts continue to advance without being opened |
| Slow drift | Seven/21-day quiet-presence fixtures, high-offer fixtures, and two-week absences meet the monotonic/calibration rules |
| Mood continuity | Tab reopen, device switch, midnight, DST, timezone change, and worker restart never reset to a neutral default |
| Bird identity | Rename, migration, duplicate species, export, and backup restore preserve UUID, vector, signature seed, and prior notebook prose |
| Greeting | Quick return, long absence, varied moods, repeated same bird, and simultaneous owner openings produce one varied leader promptly; visitor arrival produces none |
| Offers | Accept, wait, ignore, all-birds-cooling, two-device race, and muted fragment preserve server-chosen reactions and per-bird cooldowns |
| Listen-in | Pointer toggle, Enter, arrows, Escape, empty click, focus departure, hidden state, and rapid retarget preserve smooth audible ambient others |
| Settle | Four-second fade, click undo before five seconds, keyboard undo, late re-engagement, origin close/crash, stale device ping, and plain tab close preserve terminal presence with no penalty |
| Adoption | Every age boundary, ignored offer, duplicate accepts, old account with multiple eligible slots, and cap seven behave without rewards/counters/rerolling |
| Notebook | Evidence-backed specific prose, absent evidence, heavy sessions, rename, and years of pagination preserve sparsity, truth, immutability, and full history |
| Rendering | Initial cold paint, refresh, visible snapshot update, suspension return, seven-bird phone layout, all perches, and first-adoption exception avoid clipping/teleporting/ordinary entry animation |
| Reduced motion | Before-first-paint preference, live changes, flight/pose cross-fades, settle undo, and captioned night scene remain alive without drift/parallax motion |
| Audio | Unsupported/blocked WebAudio, mid-call interruption, output failure, seven signatures, three-call overlap, and 30-minute node usage avoid recordings, clicks, duplication, and leaks |
| Captions/narration | Runtime motif variations match prose, silent fallback enables captions, slow narration does not backlog, and user actions receive prompt naturalist observations |
| Keyboard/contrast | All surfaces, full-night captions, fading bar focus, touch widths, text zoom, screen-reader narration, and restored dialog focus pass |
| Authentication | 15-minute expiry, simultaneous consumes, link scanner GET, revoked session mid-write, and old/new email race cannot duplicate or transfer identity |
| Visits | 30-day unused expiry, one-time use, active revocation, long-poll authorization, visitor-owned separate account, hidden duration, and default-off notifications preserve render-only access |
| Export/deletion | Consistent snapshot, expired download, changed email, recover-before-deadline race, purge retry, all-store erasure, and old-backup restoration honor ownership and privacy |
| Telemetry | Inject names/emails/tokens/vector-like fields into controlled fixtures; emitted metrics/logs redact or reject them and never expose event payloads |
| Performance | Cold 4G first-bird under 500 ms, initial JS under 2 MB gzip, 60 fps old-laptop idle, and flat 30-minute resources are enforced in build/release checks |

Use property-based tests for nonnegative drift, unioned interval coverage, event idempotency, count cap, and time-step equivalence. Use integration tests with real database transactions for concurrency and auth revocation. Use audio rendering tests for clipping/timing and caption agreement, plus human listening for recognition and uncanniness. Visual and accessibility review remains necessary where a numeric assertion cannot establish the intended experience.

## 17. Risks, mitigations, and stop conditions

| Risk | Concrete mitigation and signal | Response |
| --- | --- | --- |
| Drift is too fast, too slow, or becomes an optimization game | Fixed minute integration, dominant bounded presence, low-pass state, synthetic week/three-week fixtures, no visible traits | Tune future rates/mappings; never subtract established personality or ship trait counters |
| Presence silently inflates | Exact conjunction, bounded observed intervals, cross-device union, no crash extrapolation, eight-case predicate tests | Block release or disable faulty input credit until fixed; retain already stored vectors rather than resetting birds |
| Sync corrupts identity or loses prior drift | Single writer, ordered committed log, transactional cursor/state, idempotency, restore/migration drills | Stop rollout immediately; repair from verified canonical backups/checkpoints without client state merges |
| Calls feel repetitive, synthetic in an unpleasant way, or indistinguishable | Stable signatures, bounded procedural variation, overlap limits, listening sessions across six/seven birds | Hold bird-count ramp and revise motifs/mix without changing identity seeds |
| Audio policy prevents the advertised first sound | Permission-aware resume and immediate caption fallback, no fake autoplay claim | Preserve moving scene; expose matter-of-fact sound controls and measure aggregate failures |
| Screen-reader or reduced-motion mode loses the charm | Shared factual descriptors, designed prose/cross-fades, human assistive-technology review from M1 | Treat regressions as release blockers rather than postponing access work |
| Cold first-bird target is missed | SSR current-phase SVG, tiny critical path, warm serving tier, measured network budget | Optimize actual delivery; no spinner, fake bird, or unsafe shared state cache |
| Idle runtime grows memory or drains batteries | Pools, bounded queues, disposal ownership, hidden-tab shutdown, 30-minute soak | Block release on retained growth or background rendering |
| Tick fleet becomes expensive or falls behind | Indexed due work, staggered schedule, bounded batches, lag monitoring, equivalence-tested analytical batching | Add worker capacity or equivalent batching; do not stop ticking absent users |
| Notebook becomes repetitive or an activity report | Evidence predicates, immutable specific prose, local novelty limits, no user-behavior templates | Suppress low-quality candidates; a sparse notebook is preferable to generic filler |
| Sharing leaks host state or influences drift | Separate grant/session routes, same private projection, revocation on every pull, no visitor simulation events | Revoke faulty grants, stop sharing rollout, preserve owner state |
| Private relationship data leaks into tooling | Credential separation, emission allowlists, no payload capture, private exports, all-store erasure | Stop the faulty pipeline immediately and correct data handling before resuming |
| Product contradictions produce inconsistent implementations | Explicit decisions in section 2 and contract review at M0 | Keep one chosen behavior per surface; do not quietly add both competing interpretations |
| Feature growth erodes restraint | Scope checks in design/code review, no engagement data plumbing, seven-bird hard cap | Reject counters, public comparisons, custody mechanics, and announcement surfaces |

Completion means an engineering team can implement and verify the scoped v1 from these contracts while preserving the birds' continuity, the quiet interaction model, and the privacy boundary. This planning deliverable does not run evaluation, implement the product, or authorize work outside the assigned phase-1 files.
