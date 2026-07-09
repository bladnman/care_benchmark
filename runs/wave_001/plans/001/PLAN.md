# Pocket Aviary v1 implementation plan

## 1. Outcome and product invariants

Build a modern-browser, single-account aviary that starts with two individually recognizable birds, remains alive without an open client, and can grow to at most seven birds on an age-based schedule. The engineering definition of done is not merely that each interaction exists: the product must preserve continuity, restraint, and the sense that the birds—not an application shell—noticed the user.

The following are release-blocking invariants:

- The server owns one canonical aviary and is the only writer of personality vectors. Clients submit facts about interactions, never desired trait values.
- Presence counts only while visibility, focus, and recent pointer-or-key activity are all true. Open-tab time is never treated as presence.
- Persistent personality changes are monotonic toward expressive. Absence cannot reduce a trait, create distress, or remove plumage saturation.
- Mood, recent-attention presentation, and weather may make a bird temporarily quieter or warier without mutating personality downward.
- The first resolved scene frame is already in progress. There is no spinner, wake-up sequence, textual welcome, or synchronized arrival animation.
- Calls are synthesized procedurally in the client. No recorded call loops or recorded-audio fallback ship.
- Personality values are absent from every interactive client API and product UI. The account-export exception is handled explicitly in Section 5.
- Product prose is lowercase, specific naturalist prose. Identity, settings, errors, sync, and accessibility-settings prose is direct, matter-of-fact system language.
- Visits are explicit, revocable, read-only, and do not contribute presence or interaction input to the host's simulation.
- No score, streak, achievement, level, care meter, notification loop, public profile, discovery surface, shared aviary, or native client is built or instrumented.

## 2. V1 scope

### Included

- Passwordless email magic-link accounts, per-device revocable sessions, verified email changes, export, 30-day recoverable deletion, and final hard deletion.
- One canonical aviary per account, two starter birds, naming and renaming, a stable identity for each bird, and age-gated opportunities to add birds up to seven.
- Five hidden personality traits, persistent mood, three perch zones, bird-to-bird responses, rare ambient weather, local-time day/night state, and server-side progression at roughly one-minute cadence.
- Return greeting, passive presence, listen-in, seed/song-fragment/still-pool offers, settle with five-second undo, and a sparse read-only field notebook.
- A single responsive horizontal scene, full-motion and designed reduced-motion renderers, subtle ambient ornaments, a fading top bar, and quiet-field loading and one-time empty states.
- Procedural call generation, recognizable per-bird signatures, chorus mixing, gradual listen-in rebalance, mute handling, runtime-derived captions, and graceful silence when WebAudio is unavailable.
- Screen-reader narration, keyboard navigation, visible focus, WCAG AA text contrast, captions, and user-controlled accessibility preferences.
- Read-only email invitations, visit grants, revocation, visit log, and opt-in visit notifications that default off.
- Aggregate-only operational and performance telemetry that never joins to simulation events or bird state.

### Excluded and intentionally not pre-built

- Native apps or native-specific protocol accommodations, payments, subscriptions, multiple aviaries, shared/household accounts, customizable scenes, species rarity, bird catalog selection, user-arranged perches, or more than seven birds.
- Hunger, death, illness, decaying happiness, negative personality drift, mandatory settle, or any consequence framed as punishment for absence.
- Streaks and disguised streaks, visit-frequency summaries, achievements, ranks, stats panels, personality dashboards, visit incentives, push notifications about the aviary, or engagement email.
- Profiles, follows, comments, chat, co-presence, avatars, shared cursors, mutual-visit mechanics, public URLs, search, feeds, leaderboards, and visitor interactions.
- Editable notebook entries, user journals, generic event logs, and notebook observations about the user's attendance.
- Recorded call files, a recorded fallback library, heavy parallax, panning, zooming, scene scrolling, or birds cropped out of a supported viewport.

Keep these exclusions out of schemas and metrics where practical. In particular, do not collect leaderboard-ready interaction aggregates or create public-discovery fields “for later.”

## 3. Architecture and ownership boundaries

Use a TypeScript monorepo with independently deployable packages and one shared domain-contract package:

1. Web client: a small server-rendered shell plus a browser scene runtime. It owns interpolation, animation, WebAudio synthesis, input collection, accessibility presentation, and session-local listen-in/settle presentation. It never calculates persistent drift.
2. API service: authenticates sessions, serves snapshots, validates and deduplicates interaction commands, resolves immediate offer reactions, manages accounts/invites, and exposes notebook pagination. It writes append-only domain events and non-personality records.
3. Simulation workers: claim due aviaries, consume new host events in server order, run deterministic mood/drift/weather/social-bird rules, produce notebook candidates and future call intents, persist canonical state, and publish a new materialized snapshot.
4. Email worker: consumes an outbox for magic links, export links, invitations, and explicitly enabled visit notifications. No simulation event payload enters the email system.
5. PostgreSQL primary: system of record for accounts, sessions, aviaries, birds, events, notebook entries, invitations, visit logs, tick leases, and revisions. Encrypt email and export objects with managed keys.
6. Private snapshot cache: stores only render-safe materialized snapshots by synthetic aviary ID and revision. It may be regionally replicated with a maximum one-tick staleness. It contains no email or personality vector.
7. Object storage: short-lived, authenticated account-export archives only. Download URLs expire and objects are deleted automatically.

Do not introduce Kafka, event sourcing of the entire application, or a client database for v1. The append-only interaction table plus an outbox and row-level tick claims provide the required correctness with fewer failure modes. Partition interaction events by time once volume requires it, but keep the canonical state transaction in PostgreSQL.

### Request and render flow

On authenticated navigation, the edge/server shell starts three operations in parallel: verify the session cookie, fetch the small render-safe snapshot, and stream the critical scene shell/assets. If a valid private snapshot is available, inline it in the HTML with a revision and server timestamp; otherwise render the quiet field and fetch it immediately. Never cache authenticated HTML publicly.

The browser builds a scene graph from the snapshot and starts birds at the supplied phase offsets and active poses, then interpolates from canonical server time. Noncritical settings, notebook, invite, and account code is split and loaded on demand. Visible clients refresh on a low-frequency interval, on visibility regain, after a long frame gap or device resume, and after accepted commands. Hidden clients suspend rendering and audio scheduling.

All mutations go to the primary region. A command acknowledgement includes the authoritative event ID and current/next expected snapshot revision. The next snapshot reconciles all local ephemeral presentation without a last-write-wins merge.

## 4. Data model

Use UUIDv7 or UUIDv4 synthetic identifiers generated server-side. Email appears only in the encrypted account field and invitation/visit-log fields where the product explicitly requires it; every internal reference uses UUIDs.

### Core records

| Record | Required fields and rules |
|---|---|
| Account | id, encrypted_email, email_lookup_hmac, email_verified_at, timezone_id, locale, status, deletion_requested_at, hard_delete_at, created_at, updated_at. Exactly one current aviary. Timezone is an IANA value inferred at onboarding, editable in settings, and canonical across devices. |
| DeviceSession | id, account_id, token_hash, device_label, created_at, last_seen_at, expires_at, revoked_at. The raw bearer token is never stored. Revocation is enforced on every authenticated mutation and bounded-cache read. |
| MagicLink | id, email_lookup_hmac, token_hash, purpose, expires_at, consumed_at, request_ip_hash, created_at. Consumption is atomic and single-use; expiry is 15 minutes. |
| Aviary | id, account_id, created_at, engine_version, state_revision, event_cursor, tick_number, next_tick_at, last_tick_at, last_host_presence_at, weather_state, weather_until, notebook_last_entry_at. |
| Bird | id, aviary_id, species_key, stable_call_signature_seed, name, adopted_at, personality_vector, personality_config_version, mood, mood_since, perch_zone, pose_key, action_phase, last_offer_at_by_type, created_at. Stable ID survives renames and migrations. |
| InteractionEvent | id, aviary_id, bird_id nullable, device_session_id, idempotency_key, type, server_received_at, bounded_occurred_at, payload_version, privacy_payload, processed_tick. Unique on device_session_id plus idempotency_key. Host events only feed simulation. |
| PresenceSegment | Represented as a validated interaction event with start, end, qualifying sample count, and capped duration; never as an unbounded client total. Server rejects overlap per session and excessive clock skew. |
| ImmediateReaction | id, source_event_id, bird_id, reaction_key, seed, starts_at, expires_at. Lets accepted offers render immediately and converge on other devices without letting clients choose a reaction. |
| NotebookEntry | id, aviary_id, observed_at, prose, observation_kind, source_tick, template_version, created_at. No raw trait values or user-attendance facts. Immutable and cursor-paginated indefinitely. |
| AdoptionAvailability | aviary_id, ordinal, available_at, offered_species_key, accepted_at, bird_id. Dates derive only from aviary age and a versioned schedule; visits and interactions are not inputs. |
| MaterializedSnapshot | aviary_id, revision, generated_at, valid_after, render_payload, narration_facts, checksum. Render payload excludes hidden vectors and private events. |
| Invite | id, host_account_id, host_aviary_id, encrypted_visitor_email, visitor_email_hmac, link_token_hash, status, created_at, unused_expires_at, consumed_at, revoked_at. Unused links expire after 30 days. |
| VisitGrant | id, invite_id, browser_token_hash, created_at, last_seen_at, revoked_at. Consuming the one-time email link creates the browser grant; replaying the link fails. A grant remains usable on that browser until revoked. |
| VisitSession | id, invite_id, grant_id, started_at, last_heartbeat_at, ended_at, approximate_duration_seconds. Never emits presence or bird interactions. |
| AccountSettings | account_id, reduced_motion_override, call_captions_enabled, audio_muted, visit_notifications_enabled, updated_at. Visit notifications default false. |
| OutboxMessage | id, type, account_id/invite_id, destination reference, created_at, claimed_at, sent_at, failure_class. Email addresses are resolved only inside the email boundary. |

Store personality as a typed five-field structure, not an unvalidated free-form blob. Use normalized [0,1] values, database bounds, and an engine/config version. Keep the vector encrypted at rest or protected by database-level encryption and never serialize it through the general API contract.

### State and retention rules

- Canonical bird and aviary state is backed up with point-in-time recovery. Add restore drills that verify vectors, stable bird IDs, event cursor, and notebook continuity together.
- Keep processed interaction events only for the bounded reconciliation/debug window approved by privacy review; propose 30 days for v1, then hard-delete. The canonical vector and mood, not event replay, are the persistent source of truth.
- Preserve notebook and visit log for the account lifetime unless hard deletion occurs. Do not copy either into analytics.
- Account hard deletion cascades through every online table, cache, outbox, visit grant, telemetry deletion index, backup tombstone workflow, and export object. Document the bounded backup expiry.
- Soft-deleted accounts cannot tick, send invites, or create new events. Signing in during the 30-day window offers a direct recovery action; recovery re-enables ticking without inventing missed presence.

## 5. API surface and contracts

Version all endpoints under /v1 and all event payloads independently. Responses carry request IDs, server time, and a stable error code. System errors use matter-of-fact copy in the client; product prose is not returned as an error message.

### Authentication and account

| Method and path | Behavior |
|---|---|
| POST /v1/auth/magic-links | Accept email, rate-limit by email HMAC and IP, enqueue a 15-minute single-use link, and return the same neutral response whether an account exists. |
| POST /v1/auth/magic-links/consume | Atomically consume token, create/recover as appropriate, issue rotating HttpOnly Secure SameSite=Lax session cookie, and return onboarding/account state. |
| GET /v1/account | Return matter-of-fact account/settings data and canonical timezone; no simulation internals. |
| PATCH /v1/account | Update verified settings/timezone with field validation and an If-Match account revision. |
| POST /v1/account/email-change | Send a verification link to the new address; commit only after consumption. Old address remains valid until then. |
| GET /v1/account/sessions | List sanitized device sessions. |
| DELETE /v1/account/sessions/{sessionId} | Revoke immediately, including cache invalidation. |
| POST /v1/account/export | Create an asynchronous export, email a short-lived download link to the verified address, and avoid exposing a polling endpoint with data. |
| POST /v1/account/deletion | Mark deletion and hard-delete date; revoke visit grants and stop ticks. |
| POST /v1/account/deletion/recover | Restore during the 30-day window after fresh authentication. |

The PRD simultaneously requires an export containing personality vectors and prohibits numerical personality exposure anywhere. Treat the explicit data-portability export as the narrow exception: the downloadable JSON contains the five current values and a clear machine-readable schema, but no product UI, API response, debug panel, or explanatory “bird stats” presentation displays or interprets them. Gate the export behind a fresh magic-link confirmation, log only export status, and flag this interpretation for product/privacy sign-off before implementation. If the hard prohibition is judged to override export wording, preserve the same export contract but replace vector values with an opaque encrypted continuity blob; do not make an ad hoc choice during development.

### Aviary and commands

| Method and path | Behavior |
|---|---|
| GET /v1/aviary/snapshot | Return render-safe state, revision, server time, active weather/transitions/reactions, call-intent horizon, greeting directive when requested, and settings needed before render. Support ETag/If-None-Match. Never return vectors. |
| POST /v1/aviary/sessions/open | Create a lightweight viewing session and return snapshot plus a procedurally seeded return-greeting plan derived from absence, mood, and boldness. Visits use a separate endpoint and never receive greetings. |
| POST /v1/aviary/events | Accept a small batch of versioned host events with idempotency keys. Validate source, clock bounds, bird ownership, and allowed transitions; return accepted/rejected status per event. |
| POST /v1/aviary/presence | Accept only compressed qualifying segments/heartbeats created by the presence state machine. Cap duration and reject hidden/unfocused declarations that conflict with session lifecycle. |
| POST /v1/aviary/offers | Validate server-side cooldown and resolve one mood/personality-shaped reaction from the current canonical state. Persist an event and immediate reaction, then return the render directive. |
| POST /v1/aviary/settle | Record a session-scoped settle event, end that session's presence, and return a five-second reversible presentation directive. |
| POST /v1/aviary/settle/undo | Valid only for the same session and within five seconds; records re-engagement without rolling back already elapsed presence. |
| PATCH /v1/aviary/birds/{birdId} | Rename with Unicode length/profanity policy appropriate to private names; stable identity and all state remain unchanged. |
| GET /v1/aviary/adoption | Return an available age-based arrival, if any, without progress bars or countdowns. |
| POST /v1/aviary/adoption | Accept and name an available bird atomically; enforce maximum seven and the server-derived available_at. |
| GET /v1/aviary/notebook?cursor= | Return immutable entries newest-first in bounded pages. No interaction log or attendance summary. |

Listen-in is a session-local mix action. Send start, heartbeat if long, and end events so the server can derive a bounded duration; losing visibility, changing bird, Escape, empty-space click, or session termination generates/end-synthesizes an end. Never broadcast one device's listen-in mix to another device.

### Invitation and visit

| Method and path | Behavior |
|---|---|
| POST /v1/invites | Host supplies one email; create a single 30-day one-time link. No address-book import or onboarding prompt. |
| GET /v1/invites | List outstanding/active/revoked invitations and visit-log rows on demand; no unread counts or settings badge. |
| DELETE /v1/invites/{inviteId} | Revoke link and all browser grants immediately and invalidate visit snapshots. |
| POST /v1/visits/consume | Atomically consume an unused link and issue a scoped HttpOnly visit-grant cookie. Do not create an account or host session. |
| GET /v1/visits/{inviteId}/snapshot | Revalidate grant/revocation on every poll and return the same render state, minus host controls, notebook, greeting, private fields, and all mutation capability. |
| POST /v1/visits/{inviteId}/heartbeat | Maintain approximate visit duration only. It cannot enter the simulation event table. |

Return HTTP 410 with a plain “This visit is no longer available” surface for expired or revoked links/grants. Revocation correctness takes priority over edge-cache hit rate: visitor snapshot cache keys include invite status revision, with active invalidation and short TTL as defense in depth.

## 6. Authentication, authorization, and privacy controls

- Hash magic and invite bearer tokens with a server-side pepper; show raw tokens only in email URLs. Consume using a single conditional database transaction.
- Use CSRF protection on cookie-authenticated mutations, strict origin checks, CSP with no third-party script origins, secure headers, output encoding, and dependency/SBOM scanning.
- Authorize every bird, notebook, invite, and snapshot through account-to-aviary ownership. A visit grant has one capability: read one aviary snapshot.
- Keep email decryption in account and email modules. Application logs use request IDs and, only when necessary for restricted diagnostics, synthetic UUIDs—never email, bird names, event payloads, vector values, or invite tokens.
- Separate the simulation database/role from the aggregate telemetry exporter. There is no ETL, CDC stream, or warehouse access to interaction, bird, notebook, or visit-log tables.
- Operational metrics may count endpoint requests, failures, duration buckets, frame timings, audio-context failures, and tick latency. They may not be dimensioned by account, aviary, bird, name, species, interaction type, mood, trait, invitation email, or visit relationship.
- Store client diagnostics in bounded in-memory buffers and upload only allowlisted numeric performance fields. Enforce metric schemas in code review and CI.
- Rate-limit magic links, invite creation, token consumption, and commands. Detect malformed/replayed events without building behavioral user profiles.

## 7. Simulation engine

### Deterministic tick transaction

Run a scheduler continuously. Workers select due aviaries by next_tick_at using row locks with skip-locked semantics and a lease timeout. A tick does the following in one transaction:

1. Lock the aviary and its bird state; verify the expected engine version and tick lease.
2. Read unprocessed host interaction events after event_cursor through a fixed cutoff. Order by server_received_at then event ID. Visitor records are stored elsewhere and can never appear here.
3. Normalize overlapping listen-in and presence segments, cap signal per real-time interval, and deduplicate by the database constraint.
4. Compute time-of-day, local-day boundary, deterministic ambient weather, and bird-to-bird influence inputs using the account's canonical timezone.
5. Calculate all birds' next state from the same prior-state snapshot. Apply results only after every bird is evaluated so row order cannot change bird-to-bird effects.
6. Apply nonnegative drift deltas, mood transitions, perch/action state, weather state, and future call intents. Check finite/range/monotonic invariants before writing.
7. Score notebook candidates; insert at most one only if rarity, novelty, and cooldown rules pass.
8. Advance event_cursor, tick_number, state_revision, last_tick_at, and next_tick_at; materialize the render-safe snapshot; mark consumed events with the tick.
9. Commit state and publish cache invalidation through a transactional outbox.

Seed every stochastic choice from a cryptographic hash of aviary ID, bird ID where applicable, engine version, tick number, and decision namespace. Retries then produce the same result. Record engine/config version with state changes so a rollout can be reproduced without retaining private raw behavior forever.

If a worker misses ticks, run bounded catch-up steps from the prior canonical time rather than one giant delta. Limit work per transaction and immediately requeue until caught up. Never make the client simulate the gap. Alert on lag, lease churn, invalid state, or repeated catch-up, but do not attach bird state to alerts.

### Drift function

Represent every trait on [0,1]. Convert validated signals to weighted daily exposure units, with presence dominant, listen-in strongly bird-specific, and accepted/proximate offers small and bird-specific. Use a saturating low-pass update such as:

delta_trait = (1 - current_trait) × (1 - exp(-rate_trait × weighted_exposure))

The actual rates and per-signal weights live in a versioned configuration, not client code. Clamp the computed delta to zero or above, cap daily exposure so a long session cannot accelerate weeks of drift into hours, and apply diminishing returns near one. Settle affects immediate quieting and presence termination, not personality direction. Audio mute may influence call-expression/mood inputs only if product calibration confirms the brief's intent; it must not penalize traits or become an engagement signal.

Calibrate with accelerated and wall-clock cohorts representing quiet watching, short daily visits, occasional listen-in, frequent offers, long absence, and abusive command repetition. Acceptance targets:

- Numerical movement is reliably detectable by instrumentation tests after seven days of ordinary qualifying presence.
- A blinded human review detects coherent behavioral/visual change around three weeks, not after one session or a few days.
- Four weeks of no qualifying presence leaves all persistent traits exactly unchanged.
- Repeated actions within cooldown/daily caps cannot exceed the calibrated three-week envelope.
- Plumage saturation and every other expressive trait never decrease across ticks, migrations, retries, and concurrent devices.

Do not measure real users' population-average drift to tune this. Tune with synthetic simulations, internal consented test accounts, and prelaunch studies; the production privacy rule forbids using private interaction histories for population analysis.

### Mood and ambient expression

Use the enumerated v1 set wary, content, curious, drowsy, and alert. Mood is persistent canonical state, not a client label. For each tick, compute transition weights from current mood, local time band, recent accepted interactions, current weather, neighboring birds' prior mood/call intent, and personality. Sample deterministically, with minimum dwell periods and hysteresis to prevent minute-by-minute thrashing.

At a local-day boundary, bias the next several transitions toward a personality/time/weather equilibrium; do not snap to neutral. This implements the daily-ish reset while retaining session continuity. A high-boldness bird should have a lower wary transition probability under identical ambient inputs; a high-warmth bird should respond to neighbors more often.

Model post-absence quietness as a bounded recent-host-presence expression factor used for greeting/call probability, not as negative personality drift. It relaxes toward a neutral ambient baseline and rises with new qualifying presence. Birds remain alive and may call while the host is away; they never look sick, sad, or accusatory.

### Perches, weather, greetings, and adoption

- Map boldness and mood to weighted front/middle/back perch choices, then add deterministic variation and collision capacity. Users never submit perch commands.
- Generate rare weather from an aviary-seeded calendar with an average frequency of a few subtle events per week. Persist start/end so devices agree. Rain and wind influence mood temporarily; no thunder, snow, alerts, or user controls.
- At host-session open or visible return, derive absence from last qualifying host presence, choose one eligible bird by boldness/warmth/mood plus variation, and return a seeded greeting phrase of motion/call primitives. Optional second responses are staggered. Never trigger all birds together or return greeting copy.
- Precompute age-based adoption availability from configurable aviary-age offsets. The server chooses species from the coherent six-species pool while avoiding a catalog/rarity presentation. Adoption never depends on event counts or attention.

### Call grammar and notebook generation

Each species ships a versioned motif grammar. Each bird's permanent signature seed fixes recognizable interval contours, timbre family, register, and rhythmic fingerprint. Mood and traits vary tempo, density, articulation, and willingness to answer without replacing the fingerprint. The server schedules call intents and seeds over a short horizon; the browser expands those intents into sound and exact captions. This makes concurrent devices visually coherent without streaming audio.

Generate notebook entries server-side from structured notable facts, using a curated, versioned naturalist grammar rather than a remote generative model. Candidate facts include unusual greeting order, extended quiet/preening, weather-shaped behavior, a first stable perch tendency, or a rare chorus. Exclude trait numbers, session counts, streak-like phrasing, and judgments of the user's behavior. Apply a global several-day cooldown, novelty suppression, and significance threshold so active accounts do not create a feed. Linguistic snapshot tests and editorial review are release gates.

## 8. Multi-device consistency and offline behavior

- The snapshot revision is monotonically increasing. Clients replace canonical scene inputs only with a newer revision and use server time plus measured offset for interpolation.
- Commands are facts with random idempotency keys. Safe retries return the original result. The server orders accepted events; clients never merge vectors, moods, or perches.
- Two devices may submit presence concurrently. Union overlapping intervals per account before drift weighting so the same human minute is not double-counted. Keep device source only in the private reconciliation window.
- Listen-in and settle presentation is session-local. A settle ends presence and warms/quiets that device's scene, while the tick consumes its small mood signal. It must not force a concurrently watched device into a goodbye state. A later canonical mood snapshot still converges normally.
- Offer cooldown is server-authoritative per bird. Simultaneous devices race in one transaction: one receives an accepted reaction; the other receives a quiet cooldown response and cannot create a second drift input.
- Renames and account/settings changes use optimistic revisions. On conflict, refetch and show a matter-of-fact settings error; never last-write-win silently.
- The client may queue a small bounded set of interaction facts during a transient network failure, stamped with monotonic local timing and expiry. It may not queue adoption, rename, account, invite, or offer acceptance. On reconnect, the server validates timing and may reject stale facts. There is no offline simulation or separately evolving aviary.
- On tab hide, stop requestAnimationFrame and future audio scheduling, finalize eligible presence, and persist only session-local UI preferences. On resume/suspend gap, discard prediction and fetch a fresh snapshot before resuming transitions.

## 9. Frontend rendering pipeline

### Scene composition

Use one GPU-accelerated 2D canvas/WebGL scene for sky/weather, background foliage, three depth planes, perches, birds, and ornaments. Keep controls, dialogs, semantic bird targets, captions, and screen-reader narration in a DOM layer. The semantic targets track canvas bird bounds but do not add visible chrome until keyboard focus or captions require it.

Build a small retained scene graph with deterministic layers:

1. local-time sky and quiet weather field;
2. subtle background foliage;
3. back, middle, and front perch groups with bird actors;
4. sparse foreground branch/leaf treatment;
5. DOM accessibility and interaction layer;
6. thin top bar outside the aviary scene.

Use normalized scene coordinates and responsive layout constraints. On narrow screens compress gaps and bird scale within minimum readability bounds; on wide screens expand negative space. All bird bounding boxes must remain within the safe viewport. No camera/pan state exists.

The interaction documents require settle in the top bar while the layout's exhaustive icon list omits it. Resolve v1 in favor of the explicit interaction and keyboard requirements: include a quiet settle action as a fifth top-bar control, visually subordinate to offer, and record this as a design sign-off item. Do not hide settle inside Offer or make the whole scene an unlabeled gesture.

### Startup and snapshot transitions

- Inline critical CSS, the quiet-field background, two starter silhouette assets, and the minimum scene bootstrap. Defer notebook/account/invite code and nonvisible species assets.
- If snapshot is available, paint a bird before initializing audio, top-bar fading, or optional ornaments. Start at its server-provided pose/action phase—never frame zero of a canned loop.
- If not available, paint a quiet sky with one or two low-motion light cues and no spinner/progress copy. Replace it directly with the current scene when ready, with no “app loaded” animation.
- Only the post-onboarding first arrival may show an empty quiet field and soft bird fly-in. Persist completion server-side so a cache clear cannot replay the first-arrival fiction.
- Reconcile snapshot changes through state-specific transitions: perch movement path in full motion, cross-fade in reduced motion, pose continuation when unchanged, and no teleport unless returning from a long hidden interval where a fresh already-in-progress pose is more honest.

### Motion systems

Full-motion bird actors use layered skeletal/vector poses with bounded micro-actions: breathe/weight shift, preen, scan, head tilt, look toward a call, fluff, and perch transition. The scheduler weights actions by mood and personality, respects minimum dwell times, and seeds variation from snapshot directives. Avoid short obvious loops by mixing action lengths, phase offsets, and rests.

Ambient leaf/feather ornaments are local-only, sparse, pooled, and not canonical. Parallax amplitudes stay small and pointer motion never whips the scene. Day/night palette changes interpolate over long periods. Settle applies a several-second session-local lighting/audio envelope and is reversible for five seconds.

Reduced-motion is a separate renderer selected before first bird paint from prefers-reduced-motion or saved override. It uses slow cross-fades among authored still poses, cross-fades perches without flight paths, removes drifting leaves/feathers and pointer parallax, and slows color changes. It preserves birds, mood, calls/captions, drift, greetings, offers, and notebook observations. Changing the preference tears down one scheduler and starts the other without reloading or duplicating audio.

### Input and chrome

Top-bar opacity falls near-transparent after a calibrated stillness delay, but never while it or a dialog has keyboard focus. Pointer movement, keyboard activity, focus entry, and touch reveal it. Do not use hidden controls that become unreachable on touch.

Click/tap or Enter on a bird starts listen-in; repeat, another bird, empty scene, Escape, focus departure, tab hide, or session close ends it. Arrow keys traverse birds in spatial order. Keep visible soft high-contrast focus around the semantic bird target; this is an accessibility affordance, not ambient chrome.

## 10. Audio pipeline

Implement a versioned call-grammar runtime with an AudioWorklet-based bounded polyphonic synthesizer:

- Decode server call intents into motif tokens, then expand them with the stable bird signature seed and per-call seed. Produce pitched tonal/noise components, envelopes, pauses, trills, and articulations; do not fetch call audio.
- Route each bird through its own gain/pan bus into ambient/weather and master buses. Cap concurrent voices and total gain; use a gentle limiter to prevent chorus clipping without flattening individual signatures.
- Use a preallocated worklet state/ring buffer and pooled typed arrays. Bound scheduled call horizon and discard canceled events. Track AudioContext/worklet lifecycle to prove no 30-minute growth.
- Listen-in ramps the focused bus upward and other buses to a nonzero ambient floor over roughly one to two seconds; disengagement uses a symmetric slow ramp. Never hard cut or fully mute other birds.
- Mood shapes performance parameters; vocal frequency shapes scheduling density and chorus response probability. Neither changes the stable motif fingerprint.
- Derive caption tokens from the same expanded motif before synthesis, including contour, repetition, intensity, and source location. Captions must describe the sound actually scheduled, not a fixed bird string.
- User mute stops the master bus cleanly but keeps visual timing/captions available. Record only a bounded settings/interaction fact if required by the engine; do not infer hearing status.

Browser autoplay policy can prevent sound before a user gesture, so “already audible” cannot be guaranteed on a first-ever navigation. Attempt to resume only when policy permits; never show a blocking audio modal or fake playback. While locked, the scene remains visibly alive and call captions are temporarily available; the first ordinary pointer/key interaction may unlock audio. Preserve explicit mute. Test returning sessions where browser permission allows immediate audio separately from first visits. Product and accessibility review must approve the non-announcing top-bar status for a permanently blocked context.

If WebAudio or the worklet cannot initialize, dispose partial nodes, run in silence, enable captions by default for that session, and expose a matter-of-fact status in accessibility settings. Do not download a recorded fallback.

## 11. Accessibility implementation

### Semantic structure and navigation

- Use landmarks for top bar, aviary, notebook/settings dialogs, and status. Give each bird a semantic button-like target named by user-assigned name/species without announcing hidden mood labels or coordinates.
- Tab order reaches account, accessibility, notebook, offer, settle, then the scene. Within the scene, arrow keys move among birds; Enter toggles listen-in and Escape exits.
- Trap focus only in true modal dialogs, restore it on close, and keep all focus rings visible across day/night palettes at AA contrast or better.
- Support 200% text zoom and narrow viewports without obscuring birds or controls. Respect platform high-contrast/forced-colors for DOM surfaces.

### Naturalist narration

Build a narration composer from render-safe facts, independent of the visual animation loop. It emits one cohesive lowercase observation every 30–60 seconds at idle, deduplicates facts, and pauses while a prior announcement is speaking. User-initiated greeting, offer reaction, settle, and focus changes enter a priority queue but remain observations rather than state-machine announcements.

Use a polite live region for ambient narration and a separate carefully managed region for immediate user results. Never enqueue every tick, call, or pose. Give users controls to pause narration or request the current scene summary without silencing ordinary control feedback. Voice and screen-reader testing must cover rapid commands and ensure the queue cannot grow without bound.

### Captions and visual accessibility

Caption each actual call near the bird, with collision avoidance, safe contrast backing, and a DOM transcript accessible to screen readers without double-speaking the audio narration. Allow captions regardless of audio state; default them on only for WebAudio failure/lock as approved and for saved user choice.

The reduced-motion renderer ships in the initial release and receives the same functional and editorial test matrix as full motion. Automated WCAG checks are necessary but not sufficient: perform manual keyboard-only, VoiceOver/Safari, NVDA/Firefox or Chrome, screen zoom, hearing/caption, and vestibular review with disabled testers before launch.

## 12. Performance budgets and enforcement

Translate PRD budgets into CI gates and field alarms:

| Budget | Engineering gate |
|---|---|
| Initial JavaScript under 2 MB gzipped | Enforce by route/entry manifest in CI, with a tighter internal target of 1.5 MB to retain regression room. Lazy chunks do not excuse moving first-scene code out of the measured entry. |
| First bird visible under 500 ms | Lighthouse/WebPageTest-style runs on a pinned mid-tier mobile profile and shaped 4G, measuring navigation start to the bird paint performance mark. Run warm and cold snapshot cases by geography. |
| 60 fps idle on five-year-old mid-range laptop | Thirty-minute browser trace on pinned representative hardware; require stable frame pacing and report p95/p99 long frames, not just average fps. |
| No memory growth over 30 minutes | Repeat 30-minute full-motion, reduced-motion, listen-in, notebook-scroll, hide/resume, and chorus scenarios. Compare post-GC retained heap and worklet memory to a bounded steady-state envelope. |
| Tick p99 below 5 seconds | Record duration only, by deployment/version—not account. Alert at 5 seconds and separately on due-queue lag. |

Keep snapshot payload in kilobytes and set a concrete CI ceiling after prototyping; start with 32 KB compressed for seven birds. Budget critical assets, main-thread startup work, draw calls, active DOM nodes, audio voices, and particle pools. Use content hashes and long-lived caching for public assets, but private snapshots are never publicly cacheable.

Instrument from day one:

- navigation-to-first-bird, snapshot latency/status, bundle/asset transfer, long-frame histogram, dropped-frame histogram, bounded heap trend, AudioContext/worklet failure count, tick duration/lag/retry, API latency/error rate, and email delivery failure class;
- synthetic browsers in common regions for anonymous starter fixtures, including reduced motion, silence fallback, and slow snapshot paths;
- release/version and coarse browser/device-class dimensions only.

Deliberately do not measure real-user trait values, mood distribution, calls per bird, offers, listen-in behavior, presence by account, notebook content, species popularity, visitor relationships, or any metric that reconstructs an aviary. Session-duration histograms are anonymized at emission, coarsely bucketed, and contain no join key.

## 13. Delivery sequence

### Milestone 0: contracts, threat model, and design prototypes

- Freeze product invariant tests, render-safe snapshot schema, event schema, privacy metric allowlist, voice guide, and ownership map.
- Resolve/sign off the settle top-bar discrepancy, export/vector tension, autoplay behavior, canonical timezone behavior during travel, and exact adoption-age schedule.
- Prototype two species end to end: signature grammar, full/reduced poses, captions, narration, and a seven-voice chorus stress fixture.
- Establish baseline hardware/browser lab and performance CI before feature code expands.

Exit: architecture/privacy/accessibility reviews approve contracts; first-bird prototype meets budget with two birds; no hidden vector appears in captured client traffic.

### Milestone 1: identity and canonical state foundation

- Implement account/magic-link/session lifecycle, encrypted email boundary, account timezone, one aviary/two-bird creation, stable IDs, backup/restore, deletion state machine, and materialized snapshot.
- Implement tick scheduler, deterministic transaction/retry, event idempotency, engine versioning, and empty synthetic engine rules.
- Add snapshot client with quiet field, already-in-progress first paint, visibility refresh, and no client persistence of canonical state.

Exit: two devices converge under racing events and suspend/resume; restore preserves IDs/vectors; magic-link replay and session revocation tests pass.

### Milestone 2: bird engine and core scene

- Implement trait/mood/weather/perch/bird-interaction rules, presence state machine, greeting planner, full and reduced scene actors, responsive constraints, day/night cycle, and stable six-species asset contracts.
- Build accelerated drift simulator and invariant/property tests. Calibrate with synthetic scenarios before enabling production drift.
- Implement rename and age-based adoption through seven with no catalog/progress mechanics.

Exit: seven-bird state stays in frame across supported viewports; drift/mood tests meet timescale and monotonic requirements; full and reduced renderers have parity.

### Milestone 3: interactions, audio, and notebook

- Implement listen-in, server-resolved offers/cooldowns/reactions, settle/undo, audio grammar/worklet/mixer, caption derivation, narration composer, and sparse notebook generation/pagination.
- Editorially validate product/system voice separation and all reaction/notebook/narration templates.
- Complete 30-minute frame/memory/audio soak suite and browser-autoplay/fallback matrix.

Exit: no repeated recorded asset exists, seven signatures remain distinguishable in blinded review, offer races resolve once, and accessibility manual test blockers are closed.

### Milestone 4: accounts completion and visits

- Add settings/session revocation/email change, export, soft/hard deletion cascade, invitation outbox, one-time grant consumption, read-only snapshot capability, revocation, visit log, and opt-in notification setting.
- Prove by authorization tests that visitor requests cannot hit host mutation routes or emit host presence/events.
- Run privacy data-flow audit and deletion/export drills.

Exit: active revocation terminates by the next snapshot pull; hard deletion removes all named stores; telemetry schema audit finds no simulation fields.

### Milestone 5: hardening and launch candidate

- Browser matrix for last two Chrome/Safari/Firefox/Edge versions; unsupported-browser page for older clients.
- Accessibility audit, localization/long-name stress, security review, backup restore, chaos tests for tick/email/cache outages, and all performance gates.
- Operational runbooks for tick lag, snapshot cache corruption, email delay, WebAudio failure spike, and deletion/export failure.

Exit: launch acceptance matrix passes at two birds and latent seven-bird fixtures; privacy, accessibility, security, and product design owners sign off.

## 14. Verification strategy and acceptance matrix

### Simulation and state

- Property tests: every trait stays finite/in range/nondecreasing; retries are deterministic; row iteration order does not affect output; one event is applied once; a visitor event is structurally impossible; caps/cooldowns hold.
- Golden time-travel tests across dawn/dusk, daylight-saving changes, timezone edits, long absence, weather start/end, engine-version migration, and account soft-delete/recovery.
- Accelerated 1/7/21/90-day scenarios with defined synthetic attention patterns and human visual/audio reviews at the promised thresholds.
- Concurrency tests with two devices overlapping presence, listen-in, offer, rename, settle, and resume. Assert unioned presence and no lost additive drift.
- Failure injection before/after event cursor, state write, snapshot materialization, and cache invalidation. Assert atomic recovery and stable bird identity.

### Experience and voice

- First-frame capture tests prove there is no static wake pose, spinner, welcome text, synchronized greeting, or repeated first-arrival sequence.
- Statistical greeting tests cover absence/mood/boldness weighting and staggered responses without revealing deterministic repetition.
- Notebook corpus review checks lowercase/present/specific phrasing, rarity, deduplication, no user attendance, no numerical state, and indefinite pagination.
- Interaction tests cover every disengagement route, settle undo at boundary times, rejected offer cooldown, muted/locked/silent audio, and top-bar fade with keyboard/touch.

### Audio and visuals

- Grammar unit tests map seeds to deterministic motif/caption pairs and ensure mood variation preserves identity features.
- Automated waveform checks catch clipping, DC offset, excessive loudness, hard listen-in cuts, and unbounded voices; human studies test recognizing an individual among up to seven.
- Viewport screenshots for narrow phones through wide desktop, all bird counts, all perches, dawn/noon/night/weather, long names, zoom, contrast, full/reduced motion, and caption collisions.
- Thirty-minute and repeated hide/resume tests verify no leaked canvases, DOM nodes, timers, AudioContexts, worklets, buffers, observers, or notebook rows.

### Security, privacy, and accessibility

- Token expiry/replay, enumeration, CSRF, XSS through names, IDOR, revoked session, invite theft/replay/revocation, cache-key isolation, and rate-limit tests.
- Static contract tests reject personality vectors/private event fields in snapshots, logs, error reports, metrics, visit responses, and client source maps.
- Export/deletion integration tests inventory every datastore and prove short-lived download authorization and cascade completion.
- Automated semantics/contrast plus manual keyboard, VoiceOver, NVDA, reduced-motion, zoom, captions-only, and silent-fallback journeys. Verify narration queue pacing and absence of duplicate announcements.

## 15. Rollout and operations

Use versioned server flags for engine configuration, call grammar, notebook templates, and allowed maximum birds. Flags must not create behavioral analytics cohorts or expose controls to users.

1. Team dogfood with synthetic/resettable accounts. Validate operational correctness; do not use employee interaction patterns as production calibration data without consent.
2. Accessibility and audio design partner preview with two birds. Block release on affective parity, not only checklist defects.
3. Small invite-only production cohort at two birds, with simulation writes enabled and visits disabled initially. Watch only aggregate errors, latency, memory/frame health, deletion/export, and support reports.
4. Enable offers/notebook and then quiet visits after authorization/revocation soak. Keep notification toggle hidden from onboarding and off.
5. Ramp the server maximum through 3, 5, then 7 only after chorus recognition, snapshot size, tick cost, frame pace, and memory gates pass at each level. Age eligibility still controls what any real aviary can adopt.
6. General availability after a full soft-delete window has been exercised in production-like staging, restore drills pass, and tick p99/queue lag have stable headroom.

Use immediate kill switches for drift application, new notebook generation, visits, notification email, and noncritical weather—not for canonical snapshot reads. A drift kill switch pauses deltas while mood/continuity continue; it never rolls vectors back. Rollback engine/config by version with explicit migrations and deterministic fixtures. Never reset or regenerate a bird to recover from a deployment.

Day-one dashboards contain only the allowlisted aggregate operational measures. Qualitative product calibration comes from opt-in research and support, not mining interaction records. Review telemetry schemas, data retention, bundle size, browser support, and accessibility at every release.

## 16. Principal risks and mitigations

| Risk | Mitigation and release signal |
|---|---|
| Drift is too fast/slow or action-spammable | Versioned saturating rates, daily caps, cooldowns, synthetic long-horizon simulator, one-week instrument/three-week human targets, and a pause-only kill switch. |
| Neglect accidentally becomes punishment | Nonnegative database/engine invariants; separate decaying recency expression from persistent traits; long-absence golden tests and language review. |
| Tick retry or device races lose/duplicate drift | Transactional cursor, idempotency keys, deterministic seeds, row locks, unioned presence intervals, additive server-only deltas, and failure-injection tests. |
| A vector is reset during migration/restore | Stable bird IDs, vector/config versions, point-in-time backup, migration checksums, restore drills, and a no-regeneration incident policy. |
| Presence overcounts idle/background tabs | Explicit client state machine, recent activity window, hide/blur termination, server caps/overlap union, suspend tests, and no open-tab fallback. |
| Audio sounds canned or birds blur together | Stable signature grammar plus per-call seeds, no loops, bounded chorus mix, motif repetition tests, and blinded recognition testing through seven birds. |
| Autoplay blocks the promised audible first frame | Design around honest browser constraints, nonblocking unlock on ordinary interaction, temporary captions, no modal, and browser-specific acceptance criteria. |
| Rendering misses 500 ms or leaks over time | Minimal inline scene path, on-demand chunks, fixed budgets in CI, pooled actors/audio/ornaments, pinned hardware soaks, and max-bird ramp gates. |
| Accessible modes become stripped fallbacks | Ship designed reduced-motion/narration/captions in every milestone, parity matrices, disabled-user review, and release-blocking manual audits. |
| Naturalist voice becomes generic or invades errors | Curated grammar/templates, editorial golden corpus, surface-class contract, and system-copy review for all error/settings routes. |
| Snapshot/cache leaks private or stale visit state | Render-safe projection with schema denial tests, private cache isolation, revocation revision keys, active invalidation, and short visitor TTL. |
| Operational telemetry violates the relationship privacy promise | Separate roles/pipelines, allowlist-only SDK, no join keys, CI schema enforcement, privacy review, and no production drift analytics. |
| Visit scope expands into social mechanics | Capability-limited read API, no visitor events, no public identifiers/endpoints, no badges, and product invariant tests against mutation routes. |
| PRD ambiguities cause late redesign | Obtain milestone-0 sign-off on export/vector policy, settle placement, autoplay status, timezone/travel behavior, notification email wording, and adoption schedule. |

## 17. Final release checklist

- Every supported client sees the same canonical birds, moods, weather, and stable identities; no personality write path exists outside the tick worker.
- A normal session works without dialogs: current scene, varied greeting, passive presence, listen-in, offers, settle, notebook, and quiet return.
- Two-bird startup and seven-bird stress paths meet bundle, first-bird, frame, memory, snapshot, tick, and audio-recognition gates.
- Full motion, reduced motion, screen reader, captions, keyboard-only, silence fallback, and zoomed mobile journeys all retain the aviary's affective core.
- Account lifecycle, visit revocation, export, recovery, hard deletion, backup restore, and security tests pass on production-equivalent infrastructure.
- No prohibited gamification, care, notification, public-social, personality-stat, or behavioral-analytics surface exists in UI, schema, email, API, or dashboard.
- Product, design, accessibility, privacy, security, and operations owners sign the invariant list before general availability.
