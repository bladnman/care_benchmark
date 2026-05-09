# Pocket Aviary V1 Implementation Plan

## 1. Product Boundary and V1 Scope

Pocket Aviary V1 is a browser-only, single-user, account-backed virtual aviary. The user has one canonical aviary containing two starter birds and, over time, up to seven birds total. The core product is a single horizontal scene that is already alive when opened: birds are mid-motion, calls are procedural, the day/night state reflects the user's local time, and the server simulation has continued whether or not a client was connected.

The implementation goal is not to maximize features; it is to make a small set of interactions feel alive, specific, and non-gamified. Every engineering decision should preserve these invariants:

- The server is the only authority for aviary state, bird identity, personality vectors, mood, notebook entries, visit permissions, and account state.
- Clients render snapshots, synthesize calls, interpolate motion, and submit events. They never compute canonical personality or mood state.
- Presence is real interaction, but only when all presence conditions hold.
- Personality drift is slow, monotonic toward expressive, and never punitive.
- Product surfaces use naturalist prose; system, account, sync, accessibility, and error surfaces use matter-of-fact prose.
- The user never sees personality numbers, visit-frequency counters, streaks, scores, achievements, or explicit optimization feedback.
- Visitors can observe a host aviary read-only. Visitor presence never affects host drift.

V1 includes:

- Browser web app for modern Chrome, Safari, Firefox, and Edge, last two major versions.
- Email magic-link sign-in.
- One account, one aviary, one user.
- Two starter birds at account creation.
- Bird renaming.
- Server-side simulation tick at roughly one-minute cadence.
- Hidden bird personality vectors, moods, stable identities, procedural call grammar seeds, perch state, and drift history.
- Presence accounting from visibility, focus, and recent pointer/key activity.
- Interactions: return-greeting, listen-in, offer, settle with short undo window, notebook browsing.
- Offer types: seed, song fragment from a small library, still pool.
- Field notebook with sparse, read-only naturalist observations.
- Single horizontal responsive aviary scene with three perch zones, day/night cycle, rare weather, ambient micro-motion, fading top bar, and quiet loading field.
- Accessibility settings, screen-reader narration, call captions, keyboard navigation, reduced-motion rendering.
- WebAudio procedural call synthesis and graceful silence with captions when WebAudio is unavailable.
- Account export and deletion.
- Per-device sessions and session revocation.
- Opt-in read-only visit invitations, visit revocation, expiration, and visit log.
- Aggregate-only operational telemetry and synthetic performance monitoring.

V1 explicitly excludes:

- Native iOS or Android apps.
- Password login, SSO, payments, subscriptions, billing, or tiers.
- Multiple aviaries per account, shared aviaries, household accounts, or team profiles.
- Public discovery, profiles, follows, comments, chat, leaderboards, ratings, featured aviaries, social feeds, friend graphs, or co-presence.
- Push notifications, emails about aviary activity, or default visit notifications.
- Gamification: no achievements, streaks, scores, levels, badges, XP, visit calendars, "birds adopted" counters, or behavior summaries framed around the user's activity.
- Tamagotchi mechanics: no hunger, sickness, death, distress, decaying happiness, care schedules, visible meters, or penalties for absence.
- User-controlled layout of birds, panning, scrolling, zooming, scene customization, or bird catalog selection.
- Recorded call loops or recorded-audio fallback.
- Any user-facing numeric personality/debug surface.

## 2. System Architecture

Build Pocket Aviary as a web client plus a small set of backend services around one canonical simulation database.

### 2.1 Service Shape

Use four backend modules, deployable either as separate services or a modular monolith for V1 depending on team preference:

1. Auth and account service
   - Magic-link issuance and consumption.
   - Session token issuance, refresh, and revocation.
   - Account email storage and email-change verification.
   - Soft and hard deletion lifecycle.
   - Export request generation.

2. Aviary state service
   - Current account aviary snapshot reads.
   - Bird, mood, perch, animation phase, call-schedule, notebook, and settings reads.
   - Interaction event ingestion.
   - Visit snapshot reads with read-only permissions.

3. Simulation worker
   - Scheduled tick loop, approximately once per minute per active account partition.
   - Event-log consumption in deterministic order.
   - Personality drift delta computation.
   - Mood transitions.
   - Weather and ambient event scheduling.
   - Notebook observation generation.
   - Adoption-age eligibility for future birds.

4. Notification/email service
   - Magic-link email.
   - Account export download email.
   - Visit invitation email.
   - Optional host visit notifications only if explicitly enabled.

Keep simulation and telemetry stores separated. The simulation database contains per-account, per-bird state and interaction events. The analytics/observability pipeline receives only aggregate operational metrics and anonymized timing histograms; it must not ingest bird IDs, bird state, per-account event streams, notebook text, or personality vectors.

### 2.2 Client/Server Split

Server responsibilities:

- Canonical account and aviary identity.
- Synthetic account IDs and PII isolation.
- Bird creation, stable bird IDs, species assignment, and bird count cap.
- Persistent personality vector and mood state.
- Drift and mood updates.
- Notebook entry creation.
- Visit invite authorization and revocation.
- Snapshot construction.
- Privacy-preserving export and deletion.
- Validation of all interaction-event writes.

Client responsibilities:

- Initial render from server snapshot.
- Smooth interpolation between snapshots.
- Scene rendering, responsive layout, reduced-motion render mode, focus rings, top-bar fade.
- WebAudio call synthesis from server-provided call grammar and current call schedule.
- Listen-in mix ramping.
- Call captions generated from the same procedural call event that drives audio.
- Presence detection and presence ping submission.
- Keyboard and pointer interaction handling.
- Matter-of-fact account/settings/error surfaces.

The client may predict short visual interpolation, audio scheduling, and ambient ornaments such as leaves and feathers. It must not advance canonical mood, apply drift, issue notebook entries, or infer personality changes.

### 2.3 Render Pipeline Boundary

Snapshots should contain enough state for the client to render the aviary as already in progress:

- Snapshot server time and client-local day/night input.
- Bird IDs, names, species, current mood, perch zone, pose/animation state, animation phase, facing direction, and target transition if any.
- Personality-derived display parameters that are safe to expose indirectly, such as visual saturation multiplier bucket and behavior weights, but never raw vector values.
- Active weather state and remaining duration.
- Current settle state.
- Call grammar references, call schedule windows, and call-expression parameters.
- Notebook unread count only if it does not become a notification badge; prefer no badge and just notebook availability.

The client uses a deterministic renderer seeded by safe snapshot fields, then interpolates until the next snapshot. If the client is hidden or suspended, it stops rendering and pulls a fresh snapshot when visible again.

## 3. Data Model

Use a relational database for canonical state and event ordering. PostgreSQL is a good V1 fit because the data shape is account-scoped, transactional, and benefits from strong ordering. Use row-level partitioning or account-shard partition keys if scale requires it later.

### 3.1 Accounts

`accounts`

- `id`: synthetic UUID primary key.
- `email_encrypted`: encrypted verified email.
- `email_hash`: keyed hash for lookup and rate limiting, not exposed externally.
- `email_verified_at`.
- `created_at`.
- `deleted_at`: nullable soft-delete timestamp.
- `hard_delete_after`: nullable timestamp, set to deleted_at + 30 days.
- `settings_json`: account-level settings including accessibility, audio defaults, optional visit notifications.
- `timezone_last_seen`: last client-reported IANA timezone for local day/night and mood cues.

Do not use email as an identifier outside this table. Logs, telemetry, event streams, partitions, and background jobs reference only `account_id`.

`sessions`

- `id`: UUID.
- `account_id`.
- `device_label`: derived browser/device label for settings display.
- `token_hash`.
- `created_at`, `last_seen_at`, `revoked_at`, `expires_at`.

`magic_links`

- `id`: UUID.
- `email_hash`.
- `token_hash`.
- `created_at`, `expires_at`.
- `consumed_at`.
- `request_ip_hash` or rate-limit key.

Magic links expire after 15 minutes and are invalidated immediately on consumption.

### 3.2 Aviaries

`aviaries`

- `id`: UUID.
- `account_id`: unique.
- `created_at`.
- `bird_count_cap`: default 7.
- `settled_until_interaction`: boolean.
- `last_tick_at`.
- `weather_state`: current ambient weather descriptor.
- `weather_ends_at`.
- `local_time_anchor`: derived from latest trusted client timezone.

There is exactly one aviary per account in V1.

### 3.3 Birds

`birds`

- `id`: stable UUID.
- `aviary_id`.
- `species_key`.
- `name`.
- `created_at`.
- `ordinal`: adoption order.
- `active`: boolean.
- `current_mood`: enum or small string.
- `mood_started_at`.
- `mood_expires_after`: nullable target.
- `perch_zone`: enum `front`, `middle`, `back`.
- `pose_state`: compact pose/animation key.
- `call_signature_seed`: stable seed for procedural grammar.
- `visual_seed`: stable seed for silhouette/plumage variation.

Renaming changes only `name`. Species, identity, drift history, and call signature seed remain stable.

`bird_personality_vectors`

- `bird_id`: primary key.
- `boldness`: normalized scalar.
- `social_warmth`: normalized scalar.
- `vocal_frequency`: normalized scalar.
- `plumage_saturation`: normalized scalar.
- `curiosity`: normalized scalar.
- `updated_at`.
- `version`: monotonic integer for audit and optimistic tick updates.

This table is server-only. No API returns raw values. Admin/debug access must be controlled separately from product API and excluded from user-facing surfaces.

`bird_drift_ledger`

- `id`: UUID.
- `bird_id`.
- `tick_id`.
- `source_window_start`, `source_window_end`.
- `delta_json`: server-only trait deltas.
- `reason_codes`: server-only compact categories such as presence, listen_in, offer.
- `created_at`.

This ledger is for simulation correctness and troubleshooting, not product UI or aggregate analytics.

### 3.4 Interaction Events

`interaction_events`

- `id`: UUID or ULID sortable by server receipt.
- `account_id`.
- `aviary_id`.
- `bird_id`: nullable for aviary-level events.
- `client_event_id`: idempotency key.
- `device_session_id`.
- `type`: `presence_ping`, `listen_in_start`, `listen_in_end`, `offer_seed`, `offer_song_fragment`, `offer_still_pool`, `settle`, `settle_undo`, `bird_rename`.
- `occurred_at_client`: nullable client timestamp.
- `received_at_server`.
- `payload_json`: validated type-specific data.
- `consumed_by_tick_id`: nullable.

Events are append-only. Clients submit events; the server validates and stores them. The simulation tick consumes unprocessed events in server receipt order with idempotency by `(device_session_id, client_event_id)`.

Presence pings should include only the presence proof needed:

- visibility visible.
- window focused.
- recent pointer/key activity timestamp bucket.
- local timezone.
- current reduced-motion/audio preference if relevant to rendering, not drift.

Do not store raw pointer movement, key values, or high-cardinality behavioral telemetry.

### 3.5 Presence Windows

The simulation worker can derive presence windows from `presence_ping` events, but materializing them makes drift calibration and idempotency easier.

`presence_windows`

- `id`.
- `account_id`.
- `aviary_id`.
- `device_session_id`.
- `started_at`.
- `ended_at`.
- `duration_seconds`.
- `source_event_start_id`, `source_event_end_id`.
- `consumed_by_tick_id`.

A presence window is active only while all three conditions hold: document visible, window focused, recent pointer/key activity within the calibrated window. Settle, tab close heartbeat loss, visibility loss, focus loss, or activity timeout end the window. Settle and tab close are equivalent for drift purposes.

### 3.6 Notebook Entries

`notebook_entries`

- `id`.
- `aviary_id`.
- `created_at`.
- `observed_at`.
- `entry_text`.
- `source_type`: `greeting`, `mood`, `weather`, `quiet`, `bird_interaction`, `offer_reaction`, etc.
- `source_refs_json`: internal references to bird IDs/events, not exposed.
- `visible`: boolean.

Notebook entries are sparse, read-only, and never editable. They are product prose, not raw logs. They should not mention visit frequency, user streaks, personality numbers, or internal event names.

### 3.7 Visits

`visit_invites`

- `id`.
- `host_account_id`.
- `host_aviary_id`.
- `visitor_email_encrypted`.
- `visitor_email_hash`.
- `token_hash`.
- `created_at`.
- `expires_at`: 30 days after creation.
- `accepted_at`.
- `revoked_at`.
- `last_used_at`.

`visit_sessions`

- `id`.
- `invite_id`.
- `host_aviary_id`.
- `started_at`.
- `ended_at`.
- `approx_duration_seconds`.
- `revoked_at`.

Visitors receive read-only snapshot tokens. They do not create presence, interaction, listen-in, offer, settle, or notebook events. If a visitor interacts with local controls such as mute or captions, those settings remain visitor-local and do not touch the host aviary.

### 3.8 Exports and Deletion

`account_exports`

- `id`.
- `account_id`.
- `requested_at`.
- `completed_at`.
- `download_token_hash`.
- `download_expires_at`.
- `status`.

Exports include birds, names, current personality vectors, moods, notebook entries, account settings, and visit invite/log records as applicable. Although raw personality vectors are hidden in product UI, export includes them because the user's account data is theirs.

Deletion jobs must cascade through accounts, sessions, aviaries, birds, personality vectors, drift ledgers, interaction events, presence windows, notebook entries, visit invites, visit sessions, exports, and operational records tied to the account. Aggregate metrics with no account dimension can remain.

## 4. API Surface

Use JSON over HTTPS. WebSocket or Server-Sent Events can be added later if the pull model proves insufficient, but V1 can meet the spec with small snapshots and low-frequency polling.

### 4.1 Auth and Account APIs

`POST /api/auth/magic-link`

- Input: email.
- Behavior: rate-limit by email hash and IP bucket; create 15-minute token; email link.
- Response: matter-of-fact success regardless of account existence.

`POST /api/auth/magic-link/consume`

- Input: token.
- Behavior: validate unexpired and unconsumed; create account if needed; create session; create starter aviary and two starter birds for new account.
- Response: session token and initial redirect target.

`GET /api/account`

- Returns account settings, session list, verified email display, deletion state.
- Matter-of-fact voice only.

`PATCH /api/account/settings`

- Updates accessibility, audio, caption, visit notification, and timezone-related settings.

`POST /api/account/email-change`

- Starts new email verification.

`POST /api/account/sessions/{id}/revoke`

- Revokes a device session.

`POST /api/account/export`

- Creates export job and emails verified address when ready.

`POST /api/account/delete`

- Starts 30-day soft deletion.

`POST /api/account/delete/undo`

- Restores within the soft-delete window.

### 4.2 Aviary State APIs

`GET /api/aviary/snapshot`

Returns the canonical render snapshot for the signed-in user:

- aviary ID and snapshot version.
- server time.
- local-time interpretation inputs.
- bird render states.
- mood and pose descriptors, not raw personality values.
- active weather.
- settle state.
- call grammar descriptors and upcoming call events.
- notebook metadata without gamified badges.
- feature flags and client budgets.

Snapshot payload target: single-digit KB for normal two-bird aviaries and still comfortably small at seven birds.

The client calls this endpoint:

- on initial navigation.
- when document becomes visible.
- after long frame gaps or resume from sleep.
- on a low-frequency visible keepalive.
- after events that should be reflected quickly, such as offer or settle.

`POST /api/aviary/events`

Accepts a batch of interaction events:

- idempotent client event IDs.
- type-specific payloads.
- client timestamps as advisory.
- server assigns canonical receipt order.

Validation rules:

- reject events for deleted/revoked sessions.
- reject listen-in/offer events for unknown birds.
- enforce per-bird offer cooldowns server-side.
- treat duplicate client event IDs as no-ops.
- visitor tokens cannot call this endpoint.

`GET /api/aviary/notebook?cursor=...`

Returns paginated notebook entries in naturalist voice. Read-only. No edit/delete endpoint.

`PATCH /api/aviary/birds/{bird_id}/name`

Renames a bird. This is a system/account-adjacent action but the resulting bird name is product surface. Validate length, profanity if product requires it, and avoid changing any identity fields.

`POST /api/aviary/adoptions/{offer_id}/accept`

Future V1 endpoint for age-gated third-and-beyond birds. Eligibility is based on aviary age, not visit count or interaction score. No catalog endpoint should expose rarity or optimize choice. If included in initial V1, it presents "a bird has arrived" rather than a shop/catalog.

### 4.3 Visit APIs

`POST /api/visits/invites`

- Host enters visitor email.
- Creates one-time invite, default expiry 30 days.
- Emails visitor.
- No social prompts during onboarding.

`GET /api/visits/invites`

- Account settings surface listing outstanding, expired, accepted, and revoked invites plus visit log.
- No badge or notification count on the main aviary top bar.

`POST /api/visits/invites/{id}/revoke`

- Immediate revocation. Active visitor receives no-longer-available on next snapshot pull.

`GET /api/visit/{token}/snapshot`

- Visitor read-only snapshot.
- Same rendering data as host snapshot except host-only account fields and notebook controls are omitted or read-only as specified.
- Does not trigger greetings, drift, presence, offer, settle, or notebook events.

`POST /api/visit/{token}/session/end`

- Optional best-effort endpoint for duration logging. Snapshot timeout should also close sessions.

### 4.4 Error Surfaces

API errors should carry machine codes and matter-of-fact display copy for system surfaces. Example codes:

- `MAGIC_LINK_EXPIRED`
- `SESSION_TIMED_OUT`
- `AVIARY_LOAD_FAILED`
- `VISIT_REVOKED`
- `VISIT_EXPIRED`
- `UNSUPPORTED_BROWSER`

Do not generate naturalist error copy for auth, account, sync, visit revocation, unsupported browser, or accessibility settings errors.

## 5. Simulation Engine Design

### 5.1 Tick Ownership and Cadence

Run the simulation tick on the server at approximately one-minute cadence. The tick is the only writer of:

- personality vector changes.
- canonical mood transitions.
- perch-choice changes tied to mood/personality.
- weather scheduling.
- notebook observations.
- call schedule state used for snapshot continuity.

Implementation approach:

1. Scheduler enqueues accounts/aviaries due for tick.
2. Worker locks an aviary row with a short lease or transaction-level advisory lock.
3. Worker reads unconsumed interaction events since last tick.
4. Worker derives presence windows and offer/listen-in summaries.
5. Worker computes drift deltas.
6. Worker updates mood, perch, weather, and call schedule.
7. Worker maybe writes a sparse notebook entry.
8. Worker marks events/windows consumed and commits atomically.

If a tick is missed, the next tick should advance based on elapsed time with bounded catch-up. Do not run thousands of minute-by-minute loops after a long outage; process elapsed time in a calibrated aggregate pass that preserves mood/day-night continuity and drift slowness.

### 5.2 Drift Function

Represent each personality trait as a normalized scalar, for example `[0.0, 1.0]` with hidden seed distributions. Initial values should be varied enough for two starter birds to feel distinct, but not so extreme that one bird feels broken or absent.

Use a low-pass filter with saturating monotonic deltas:

- Presence-time is the dominant input across all birds in the aviary.
- Listen-in contributes stronger per-bird deltas to social warmth and vocal frequency for the focused bird.
- Offers contribute small deltas to curiosity and boldness, especially when a bird approaches or responds.
- Settle ends presence and may quiet mood; it does not drive long-term traits.
- Absence does not produce negative trait deltas.

Pseudo-shape:

- Convert consumed events into per-bird signal windows.
- Normalize daily signal amount against a target "regular visit" profile.
- Apply small deltas with trait-specific gains.
- Run through a saturating curve so frequent sessions cannot accelerate visible drift within a single day.
- Clamp values to allowed ranges.
- Persist additive deltas in a drift ledger.

Calibration targets:

- Instrument-measurable drift after about one week of regular visits.
- User-visible drift after about three weeks.
- No single session produces visible personality movement.
- No click-repeat interaction can saturate a trait because offer cooldowns and daily caps limit input.

Create internal simulation tests with synthetic users:

- regular short daily viewer.
- weekend-only viewer.
- intense single-session clicker.
- absent for two weeks.
- listen-in-heavy user.
- offer-heavy user.

The desired outputs are not product-facing metrics; they are calibration harnesses to verify the slow-timescale promise.

### 5.3 Mood Transitions

Mood is an enum such as `wary`, `content`, `curious`, `drowsy`, `alert`, with room for a `settled` or `sleeping` state if implementation needs it. Mood persists across sessions and changes through:

- recent events in current and recent sessions.
- local time of day.
- weather.
- other birds' calls and moods.
- personality vector biases.

Use transition probabilities rather than fixed rules. Example:

- Morning increases `alert` probability.
- Dusk and settle increase `drowsy`/`settled`.
- Rain lowers call frequency and nudges some birds toward `wary` or `quiet/content`.
- High boldness dampens `wary` transitions.
- High curiosity increases approach behavior after offers.
- Another bird's alarm-like call temporarily raises `wary` probability nearby.

Persist mood with `mood_started_at` and avoid snapping to neutral on open. Session start should read current mood after elapsed ticks, not reset it.

### 5.4 Return-Greeting

On host snapshot after an absence or visibility return, the server should include greeting candidates rather than requiring the client to invent them. Select one primary bird using:

- absence length.
- bird boldness.
- social warmth.
- current mood.
- recent greeting history to avoid the same bird greeting first every time unless traits justify it.

Then provide a greeting descriptor:

- glance, head tilt, step forward, soft call, longer call, call-and-response.
- randomized small timing offset if more than one bird may react.
- mood/personality parameters.

The client renders this as part of normal bird behavior, never as a modal, toast, banner, or textual welcome. No "you were away" copy.

Visitors do not trigger host greetings. They simply see the aviary in its current state.

### 5.5 Call Grammar Runtime

Each species owns a motif library:

- motif shapes: interval contours, duration patterns, trill/no-trill flags.
- synthesis voice parameters.
- caption templates derived from actual motif structure.

Each bird has a stable call signature seed. Personality and mood modulate:

- call frequency.
- pitch variation.
- phrase spacing.
- chorus responsiveness.
- intensity.

The server can schedule call windows and provide grammar parameters; the client synthesizes the audio using WebAudio. This gives continuity across devices without shipping recorded loops. The recognizability invariant is that Pip's call remains identifiable across mood and drift.

For chorus:

- Allow overlapping procedural calls.
- Avoid phase-locked repetition.
- Use species/bird-specific frequency bands and stereo placement subtly.
- Keep the non-focused birds audible during listen-in at ambient level.

### 5.6 Notebook Generation

Notebook entries are sparse observations. Implement notebook generation as a rule-and-template system initially, not an unconstrained language model in the critical path. The generator can combine:

- bird names.
- relative ordering facts, such as first greeter this week.
- mood/perch observations.
- weather.
- quiet stretches.
- notable offer reactions.
- bird-to-bird call responses.

Rules:

- Lowercase, present tense, naturalist voice.
- Specific to birds and moments.
- No direct "you" unless future copy rules explicitly allow it.
- No achievements, stats, visit streaks, counters, or internal event labels.
- No raw personality values or "mood: content" style labels.
- About one entry every few days for regular use, plus rare noteworthy events.

Use deterministic safety filters for banned product concepts: streak, achievement, score, level, badge, XP, rank, hunger, death, sick, neglected, "you visited X days", and any numeric trait copy.

## 6. Sync and Conflict Model

### 6.1 Canonical State

There is one canonical aviary record per account. Devices do not own or merge aviary state. Every host client pulls from the same snapshot endpoint and writes only append-only interaction events.

### 6.2 Event Idempotency

Every client event includes:

- `device_session_id`.
- `client_event_id`.
- event type.
- target bird if applicable.
- advisory client timestamp.

The server deduplicates on `(device_session_id, client_event_id)` and orders by receipt time. This prevents duplicate pings or retry storms from double-counting presence or offers.

### 6.3 No Last-Write-Wins

Never accept absolute client writes for personality, mood, drift, notebook, or current canonical perch. The tick computes additive deltas from event logs. If two devices are active, both submit events; the server consumes them in order and produces one resulting state.

Renames are a normal account action and can use last-write-wins for the bird name only, with `updated_at` and simple conflict messaging if two devices rename simultaneously. This exception must not generalize to simulation fields.

### 6.4 Presence Across Devices

Presence from multiple host devices can overlap. The product premise is one user, not multiple users, but the same user may leave the aviary open on laptop and phone. To avoid inflated drift:

- Track presence per session/device.
- Merge overlapping presence windows at the account/aviary level before drift.
- Cap daily presence contribution to calibrated maximums.
- Do not let two visible devices double personality drift for the same wall-clock minute.

Visitor presence is excluded entirely.

### 6.5 Snapshot Freshness

Snapshots include a version and server timestamp. Clients should:

- Render immediately from the first valid snapshot.
- Pull on visibility return and sleep resume.
- Reconcile in-progress local interpolation to server state without teleporting when possible.
- If snapshot load fails, show matter-of-fact retry surface over or instead of the quiet field depending on whether any prior snapshot exists.

The first visible scene should avoid a spinner. If authenticated state is unknown, show account system UI. If authenticated and snapshot is pending, show quiet field motion cues until the first bird can be drawn.

## 7. Frontend Rendering Pipeline

### 7.1 Technology Choice

Use a modern web stack with:

- React or equivalent component model for app shell, settings, notebook, and accessibility surfaces.
- Canvas/WebGL or high-performance SVG/canvas hybrid for the aviary scene.
- WebAudio for procedural calls.
- CSS/JS media queries for reduced motion and responsive layout.

The exact renderer should be selected through a prototype focused on time-to-first-bird, seven-bird idle 60fps, call caption placement, and reduced-motion cross-fades. Avoid a heavy game engine if it threatens the 2MB initial JS budget.

### 7.2 Scene Composition

The scene is one horizontal stage:

- Background sky/foliage plane.
- Back, middle, front perch zones.
- Bird layer with depth-aware scale/blur only if subtle.
- Foreground ambient branch/leaf ornaments.
- Thin top bar above the aviary scene, not inside it.

No panning, scrolling, zooming, drag-to-place, hover labels, badges, or inline tooltips inside the aviary.

Responsive behavior:

- Keep every bird visible on narrow phones.
- Preserve overall aspect ratio and scene readability.
- Increase spacing on wide desktop rather than adding extra geography.
- Do not crop birds out of frame.

### 7.3 Loading and First Frame

The app should target first bird visible within 500ms on mid-tier mobile over 4G. Implementation sequence:

1. Serve minimal HTML/CSS shell.
2. Inline or edge-embed a compact initial snapshot when possible.
3. Load core renderer and starter visual assets in the initial bundle.
4. Draw quiet field immediately if snapshot is delayed.
5. Draw birds directly into current poses/animation phases when snapshot arrives.
6. Defer settings, account management, export, visit management, and notebook history code.

No spinner-to-aviary transition. No "wake up" entry. For a brand-new account after adoption only, use the specified empty-aviary quiet field followed by first bird soft fly-in; after that the user should never see an empty aviary.

### 7.4 Bird Motion

Represent bird visual behavior as composable pose states:

- perch idle.
- preen.
- scan.
- head tilt.
- call.
- step along perch.
- front/back perch transition.
- drowsy settled pose.
- wary scan pose.

Personality and mood select probabilities and timing. The client receives canonical pose/transition descriptors and uses local easing to render smoothly. Idle motion should never look paused, but it can be subtle enough for calm.

Reduced-motion mode replaces continuous animation with:

- slow cross-fades between still poses.
- no leaf drift.
- no flight paths.
- slowed color/day-night shifts.
- stable focus outlines and captions.

### 7.5 Top Bar and Controls

Top bar includes only:

- account/settings.
- accessibility settings.
- field notebook.
- offer affordance.
- settle affordance if not grouped with offer/actions.

The top bar fades nearly transparent after a few seconds of cursor stillness and returns on pointer or keyboard activity. Keyboard users must never lose discoverability: focus should restore top-bar visibility.

Avoid notification badges unless needed for account/system errors. Notebook should not become a feed badge. Visit logs are in settings, not main chrome.

### 7.6 Interaction Details

Listen-in:

- click/tap bird or keyboard focus plus Enter.
- slow audio ramp up for focused bird.
- slow ramp down for others, never to silence.
- visual focus treatment should be understated; no "selected" label.
- disengage on same bird, different bird, empty scene, or focus exit.

Offer:

- launched from top bar.
- choose seed, song fragment, or still pool.
- target is the aviary, not a direct bird click.
- receiving bird emerges from mood/personality simulation.
- server enforces per-bird cooldown.
- client renders offer object/reaction from snapshot/event response.

Settle:

- launched from top bar.
- lighting shifts to evening over a few seconds.
- calls quiet.
- any click in aviary within five seconds sends undo and reverses locally after server acknowledgement or optimistic safe reversal.
- closing tab without settle is equivalent for presence and never penalized.

Notebook:

- top-bar notebook opens read-only scroll surface.
- entries are naturalist prose.
- infinite/paginated history.
- no edit/delete/comment/annotation.

## 8. Audio Pipeline

### 8.1 WebAudio Synthesis

Build a small procedural synthesis engine:

- oscillator/noise sources shaped by species motif.
- envelope generators for note attack/decay.
- pitch contours and intervals from motif grammar.
- per-bird timbre seed.
- subtle room/space mix.
- limiter to avoid harsh chorus peaks.

No recorded call loops. No recorded fallback.

### 8.2 Scheduling

The server snapshot provides call windows and grammar seeds. The client schedules calls slightly ahead of playback to avoid jitter. If a snapshot arrives late, the client can continue ambient scheduling within bounded rules until refreshed, but must reconcile to server state.

Call cadence depends on:

- vocal frequency trait, provided as hidden-derived behavior parameters rather than raw value.
- mood.
- time of day.
- weather.
- chorus response windows.
- listen-in state for mix only, not canonical call existence.

### 8.3 Listen-In Mix

Use gain nodes per bird:

- focused bird ramps up over a calm interval.
- non-focused birds ramp down to ambient floor.
- disengage ramps all back to ambient.
- no hard cuts.
- no complete mute of other birds.

Listen-in start/end events go to the server because they affect per-bird drift. Mix changes can happen immediately client-side for responsiveness.

### 8.4 Captions

Generate captions from the actual procedural call:

- motif contour maps to prose such as "a soft three-note rise".
- mood affects descriptors such as soft, low, sharp, paused.
- species/bird position can produce "from the back perch" where useful.

Captions appear near the calling bird, fade with the call, meet contrast requirements, and are available by user setting or default-on when WebAudio is unavailable.

### 8.5 WebAudio Failure

If WebAudio is unavailable or blocked:

- Do not load recorded sounds.
- Enable captions by default.
- Show matter-of-fact accessibility/audio setting if explanation is needed.
- Keep visual simulation fully functional.

## 9. Accessibility Plan

Accessibility must ship in V1, not as a later retrofit.

### 9.1 Screen-Reader Narration

Create a narration layer that produces naturalist prose from the same snapshot state:

- idle cadence around 30 to 60 seconds.
- priority updates for return-greeting, offer reaction, settle, and significant mood/weather changes.
- no rapid fire ARIA live updates.
- no raw state labels such as "perch 2" or "mood content".

Use an ARIA live region with careful politeness settings. Allow users to pause or adjust narration cadence in accessibility settings without disabling the visual product.

### 9.2 Keyboard Navigation

Required keyboard model:

- Tab moves through top bar items.
- Tab enters the aviary scene.
- Arrow keys move focus between birds.
- Enter toggles listen-in on focused bird.
- Escape exits listen-in or closes open panels.
- Offer menu is keyboard navigable.
- Settle is keyboard reachable.
- Notebook and settings are fully keyboard navigable.

Focus rings must be visible against bright, dim, day, night, rain, and settled scenes.

### 9.3 Reduced Motion

Honor `prefers-reduced-motion` on first load and expose an explicit setting. Reduced motion is not a static fallback; it uses designed still-pose cross-fades, slowed transitions, and removed leaf drift. Audio, captions, drift, mood, notebook, and interactions remain complete.

### 9.4 Contrast and Text

All text meets WCAG AA:

- top-bar labels/tooltips if present.
- settings/account/error surfaces.
- captions.
- notebook.
- narration if displayed visually.
- unsupported-browser surface.

Scene art itself can be subtle, but any user-copy overlay must remain legible in all day/night/weather states.

### 9.5 Accessibility QA

Ship with:

- screen-reader smoke tests on macOS VoiceOver and at least one Windows reader.
- keyboard-only test scripts.
- reduced-motion visual regression tests.
- caption/audio-off tests.
- contrast checks for all themes/states.

## 10. Privacy, Security, and Compliance Boundaries

### 10.1 PII Handling

- Generate synthetic UUID account IDs at creation.
- Store email encrypted on account record only.
- Use keyed email hash for lookup/rate limiting.
- Never put email in logs, telemetry dimensions, queue names, partition keys, snapshot IDs, or error messages.

### 10.2 Interaction Privacy

Per-bird interaction events exist only to drive that account's simulation. They must not flow to:

- analytics warehouse.
- ML training datasets.
- recommendation systems.
- population dashboards.
- third-party processors except required infrastructure under privacy controls.

Operational telemetry can include aggregate request counts, latencies, error rates, anonymized session-duration histograms, render timings, audio errors, and tick latency. It cannot include bird state, account-level interaction histories, notebook text, or personality vectors.

### 10.3 Account Export

Generate export on demand and email a short-lived download link to the verified email. Include:

- account settings.
- birds and names.
- current personality vectors.
- current moods.
- notebook entries.
- visit invites/logs.

Use matter-of-fact copy. Do not market this as a feature; keep it in account settings.

### 10.4 Deletion

Soft-delete immediately for 30 days, blocking normal access except recovery. Hard-delete after the window:

- account.
- sessions.
- aviary.
- birds.
- personality vectors.
- interaction events.
- drift ledger.
- notebook.
- visit data.
- export artifacts.

Hard-delete jobs should be idempotent and auditable without retaining per-bird content after deletion.

## 11. Performance Budgets and Observability

### 11.1 Budgets

Initial JS bundle:

- Less than 2MB gzipped at first paint.
- Code-split account settings, visit management, export, notebook history, and non-critical panels.

Time to first bird:

- Less than 500ms on a mid-tier mobile device over 4G.
- Prefer edge-delivered initial snapshot.
- Draw first bird before non-critical assets.

Runtime:

- 60fps idle motion on a five-year-old mid-range laptop.
- No memory growth over 30 minutes.
- Reuse procedural audio buffers/nodes where possible.
- Bound worker threads and AudioContexts.
- Stop rendering when hidden; simulation continues server-side.

Server:

- Snapshot payload small and cache-aware, but private.
- Tick p99 latency alarm at 5 seconds.
- Tick workers idempotent and lock-bounded.

### 11.2 Observability

Allowed metrics:

- API request counts and latencies.
- auth success/failure counts without email dimensions.
- snapshot latency and payload size.
- simulation tick duration and failure counts.
- client first-bird timing.
- render-frame timing histograms.
- audio-context errors.
- WebAudio unavailable counts.
- memory growth test results.
- unsupported browser counts.

Disallowed metrics:

- average drift by population.
- per-bird interaction aggregates.
- per-account visit frequency dashboards.
- personality vector distributions.
- notebook content analytics.
- leaderboards or rankings, even internal prototypes that could leak product direction.

### 11.3 Synthetic Checks

Run scheduled automated browsers from common geographies:

- sign in to synthetic accounts.
- load snapshot.
- verify first bird visible timing.
- verify render loop active.
- verify WebAudio creation where supported.
- verify reduced-motion mode.
- verify notebook/settings code split loads.

Synthetic accounts must be clearly marked and excluded from any user-data export/deletion confusion.

## 12. Rollout Plan

### 12.1 Build Phases

Phase A: Engine and state foundation

- Account/auth with synthetic IDs.
- Aviary, birds, personality vector, mood schema.
- Event ingestion and idempotency.
- Server tick skeleton.
- Snapshot endpoint.
- Two starter birds.

Phase B: Core scene prototype

- One-screen responsive scene.
- Three perch zones.
- Bird pose renderer.
- Day/night palette.
- Quiet loading field.
- First-frame snapshot render.
- Basic keyboard focus.

Phase C: Simulation calibration

- Presence windows.
- Drift low-pass function.
- Mood transitions.
- Perch selection.
- Return-greeting selection.
- Internal calibration harnesses.
- Offer cooldown enforcement.

Phase D: Audio and captions

- Species motif library.
- Per-bird procedural call signatures.
- WebAudio scheduler.
- Chorus mixing.
- Listen-in gain ramps.
- Caption generation from actual calls.
- WebAudio unavailable fallback.

Phase E: Product interactions

- Listen-in event lifecycle.
- Offer seed/song/pool flows.
- Settle and undo.
- Notebook generation and browsing.
- Top-bar fade.
- Bird rename.

Phase F: Accessibility and settings

- Screen-reader narration.
- Reduced-motion renderer.
- Accessibility settings.
- Contrast QA.
- Keyboard completion.
- Unsupported-browser surface.

Phase G: Accounts, privacy, and social

- Session revocation.
- Email change.
- Export.
- Soft/hard deletion.
- Visit invitations.
- Read-only visitor snapshots.
- Visit log and revocation.
- Optional visit notification setting, off by default.

Phase H: Performance, reliability, and launch hardening

- Bundle audit.
- First-bird timing optimization.
- 30-minute memory tests.
- Tick p99 alarms.
- Synthetic monitors.
- Privacy pipeline audit.
- Copy review for voice boundaries.

### 12.2 Bird Count Ramp

Launch beta accounts with two birds only. Validate:

- recognizability of two procedural calls.
- drift calibration after one and three weeks.
- first-bird timing.
- no memory growth.
- accessibility surfaces.

Then enable age-gated additional birds behind a server flag:

- three-bird aviaries for older synthetic/internal accounts.
- five-bird stress tests.
- seven-bird cap tests for chorus recognizability, caption overlap, layout compression, and 60fps idle.

Do not tie added birds to visit count, streaks, payments, or interaction volume. Age only.

### 12.3 Launch Gates

Do not launch until:

- Magic-link auth and session revocation are complete.
- Server tick is the only writer of personality.
- Presence requires visibility, focus, and recent pointer/key activity.
- Drift calibration passes internal harness targets.
- No client endpoint can write personality or mood directly.
- Snapshot resumes correctly after sleep/visibility changes.
- No spinner appears in the normal aviary loading path.
- Procedural audio has no looped assets.
- Reduced-motion, captions, narration, and keyboard navigation pass QA.
- Initial bundle is under 2MB gzipped.
- First bird visible is under 500ms in target synthetic test.
- 30-minute memory growth test passes.
- Privacy telemetry audit confirms no per-bird/account interaction data leaves simulation storage.
- Copy audit finds no welcome toast, streak language, achievements, gamified labels, or naturalist error copy.

## 13. Testing Strategy

### 13.1 Unit Tests

- Drift delta computation.
- Drift monotonicity and clamping.
- Offer cooldowns.
- Presence-window derivation.
- Mood transition probabilities under controlled seeds.
- Greeting candidate selection.
- Notebook text rule filters.
- Magic-link expiry and one-time consumption.
- Visit invite expiry/revocation.
- Snapshot permission differences between host and visitor.

### 13.2 Integration Tests

- New account creates one aviary and two starter birds.
- Client event batches are idempotent.
- Tick consumes events once.
- Two device sessions submit overlapping presence; drift is not double-counted.
- Laptop listen-in and phone offer produce one coherent server state.
- Rename does not alter bird identity.
- Soft delete blocks access and undo restores.
- Hard delete removes account-scoped records.
- Visitor snapshot cannot submit host events.
- Revoked visitor sees matter-of-fact unavailable surface on next pull.

### 13.3 End-to-End Tests

- Sign in by magic link.
- Initial aviary appears without spinner.
- A bird greets within the intended window.
- Keyboard listen-in works and audio mix ramps.
- Offer reaction appears and cooldown is enforced.
- Settle shifts lighting and undo works for five seconds.
- Notebook opens and reads entries.
- Reduced-motion mode swaps continuous motion for cross-fades.
- WebAudio blocked leads to captions-on silence.
- Account export request sends link.
- Visit invite opens read-only ambient view.

### 13.4 Long-Run Tests

- 30-minute client memory test.
- 60fps idle with seven birds.
- Seven-bird chorus recognizability and clipping tests.
- One-week and three-week synthetic drift simulations.
- Long absence return with no punitive state.
- Missed tick catch-up after simulated outage.

## 14. Risks and Mitigations

### 14.1 Drift Calibration Too Fast or Too Slow

Risk: Birds visibly change after a few sessions, turning the product into stat management, or fail to change enough after weeks, making presence feel meaningless.

Mitigation:

- Build calibration harness before full UI launch.
- Define regular-visit synthetic profiles.
- Use daily caps and saturating curves.
- Keep trait values hidden from users.
- Review notebook copy to avoid over-announcing drift.

### 14.2 Presence Inflation

Risk: Background tabs, overlapping devices, or stale activity pings overcount attention and accelerate drift.

Mitigation:

- Require visibility, focus, and recent pointer/key activity.
- Merge overlapping host device windows.
- End windows on visibility/focus loss and activity timeout.
- Cap daily presence contribution.
- Exclude visitors.

### 14.3 Sync Corruption

Risk: A client write path or last-write-wins update overwrites personality or mood.

Mitigation:

- No public API accepts personality/mood writes.
- Database write permissions isolate simulation worker.
- Event log is append-only.
- Tick applies additive deltas transactionally.
- Tests assert clients cannot mutate canonical simulation fields.

### 14.4 Audio Feels Canned or Uncanny

Risk: Procedural calls repeat too obviously, chorus sounds synthetic, or listen-in feels like a mixer solo.

Mitigation:

- Invest early in motif grammar and variation.
- Use stable bird signatures with mood/personality modulation.
- Avoid hard cuts; ramp listen-in slowly.
- Test chorus at two, five, and seven birds.
- Use captions and graceful silence instead of recorded fallback.

### 14.5 Accessibility Becomes a Flattened State List

Risk: Screen-reader and reduced-motion users receive a technically accessible but affectively inferior product.

Mitigation:

- Treat narration and reduced-motion as first-class renderer outputs.
- Use naturalist prose generation, not state labels.
- Include accessibility QA in launch gates.
- Review with users of assistive technology before public launch.

### 14.6 Product Voice Drift

Risk: Well-meaning UI additions introduce welcome banners, badges, streak wording, gamified counters, or charming error copy.

Mitigation:

- Central copy review checklist.
- Banned-language tests for notebook and product copy.
- Matter-of-fact system copy components.
- No notification badge framework in the aviary top bar except true account/system errors.

### 14.7 Privacy Boundary Erosion

Risk: Per-bird interaction data leaks into telemetry or analytics because it is useful for calibration.

Mitigation:

- Separate simulation database from analytics pipeline.
- Metric schema review before instrumentation.
- Aggregate-only RUM.
- Internal calibration harnesses use synthetic or local data, not production per-account history.
- Audit logs for data exports without retaining content after deletion.

### 14.8 Performance Misses Break Aliveness

Risk: Heavy renderer/audio/settings code causes a visible load and undermines the "already alive" premise.

Mitigation:

- Enforce 2MB initial bundle budget in CI.
- Edge initial snapshot.
- Code split non-core panels.
- Draw first bird before non-critical assets.
- Synthetic first-bird timing checks.
- Avoid heavy general-purpose game frameworks unless proven within budget.

### 14.9 Visit Feature Expands Into Social Network

Risk: Invitations grow into profiles, discovery, comments, or notifications.

Mitigation:

- Keep visit management in account settings.
- Invites are per-email, revocable, expiring, and off by default.
- No public surfaces or friend graph tables.
- Visitor tokens are read-only and cannot create simulation events.
- Optional visit notifications are off by default and not marketed.

## 15. Engineering Decisions to Make Explicit

The PRD leaves a few implementation details open. Recommended V1 decisions:

- Use PostgreSQL as canonical store and Redis only for short-lived auth/session/rate-limit assistance, not canonical simulation.
- Use a modular monolith first unless organizational constraints require microservices; keep module boundaries strict around auth, simulation, snapshots, and email.
- Use pull snapshots for V1 rather than WebSockets; add SSE/WebSockets only if polling cannot meet smoothness or battery needs.
- Use a rule/template notebook generator at launch; consider richer generation only if it can be constrained, tested, and kept out of the critical tick path.
- Use deterministic seeded procedural rendering for bird variation to reduce asset load.
- Calibrate presence activity timeout toward the longer side so quiet watching still counts.
- Treat all simulation debugging dashboards as internal secure tooling and ensure they cannot be exposed through account/product UI.

## 16. Definition of Done for V1

Pocket Aviary V1 is done when a signed-in user can open a modern browser and see the same canonical aviary across devices; two starter birds are already in motion; one bird notices the user's return without any textual welcome; calls are procedural and recognizable; presence, listen-in, offers, and settle feed a slow server-authored simulation; the notebook occasionally records specific naturalist observations; accessibility surfaces carry the same charm; visits are quiet and read-only; performance budgets are met; privacy boundaries are enforced; and no excluded game, Tamagotchi, social network, notification, or native-app surface has slipped into the product.
