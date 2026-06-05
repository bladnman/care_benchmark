# Pocket Aviary V1 Implementation Plan

## 1. Executive Direction

Pocket Aviary V1 is a browser-only, single-user virtual aviary that feels like a small living place rather than an app or game. The central implementation promise is continuity: the aviary keeps advancing on the server whether the user is present or absent, clients render the current canonical state without owning it, and birds slowly become more expressive over weeks of honest presence.

This plan treats the bird engine, sync model, rendering pipeline, audio system, and accessibility surfaces as one product system. The highest-risk failure modes are not only technical outages; they are violations of tone and mechanics that would turn the product into a game, a pet-care chore, a notification loop, or a canned animation surface.

The V1 team should build a narrow but deep product:

- Two starter birds per account, with growth up to seven based on aviary age.
- One canonical aviary per account.
- Web-only modern browser support.
- Magic-link auth, per-device sessions, export, deletion, and revocation.
- Server-side simulation tick as the only writer of canonical personality and mood state.
- Append-only interaction events from clients.
- Procedural client-side calls with captions generated from the same call grammar.
- Field notebook entries generated sparsely from state and event significance.
- Quiet, opt-in read-only visits.
- Accessibility shipped with V1, including screen-reader narration, reduced-motion rendering, captions, keyboard navigation, and contrast compliance.

Anything that turns attention into a score, makes absence feel like failure, exposes hidden numeric traits to users, aggregates per-bird relationship data for analytics, or announces the user's return must be rejected at design review and code review.

## 2. Product Scope

### 2.1 In Scope for V1

V1 includes:

- Account creation and sign-in by email magic link.
- One account, one canonical aviary, one user.
- Two starter birds selected by the system from a small species pool of roughly six species.
- User naming and renaming of birds.
- Hidden per-bird personality vectors for boldness, social warmth, vocal frequency, plumage saturation, and curiosity.
- Per-bird mood state persisted across sessions and advanced by server ticks.
- Slow personality drift over weeks, dominated by valid presence-time.
- Return-greeting behavior within one to two seconds of opening or returning to the aviary.
- Listen-in interaction for one focused bird at a time.
- Offer interactions for seed, song fragment, and still pool.
- Settle gesture with five-second undo by re-engagement.
- Field notebook with rare, read-only, naturalist observations.
- Thin top bar with account/settings, accessibility settings, field notebook, and offer.
- Day/night cycle tied to the user's local timezone.
- Rare ambient weather and subtle client-side ambient ornaments.
- Browser rendering on desktop and mobile viewports with no panning, scrolling, zooming, or offscreen birds.
- Screen-reader narration, reduced-motion mode, call captions, and keyboard navigation.
- Read-only visit invitations by email, off by default and revocable.
- Host visit log in account settings.
- Account export and account deletion.
- Aggregate operational telemetry that excludes per-bird and per-account interaction content.

### 2.2 Explicitly Out of Scope

V1 must not include:

- Native iOS or Android apps.
- Multiple aviaries per account.
- Shared or household accounts.
- Public profiles, discovery, follows, comments, feeds, mutual visits, or co-presence.
- Push notifications, engagement emails, or default visit notifications.
- Achievements, badges, levels, streaks, scores, XP, leaderboards, rankings, calendar dots, visit counters, "birds adopted" counters, or disguised equivalents.
- Hunger, death, sickness, visible distress, happiness meters, care schedules, or negative trait drift from neglect.
- User control of bird placement, perches, scene layout, weather, or time of day.
- Exposed personality vector values in any user-facing surface.
- Recorded call loops or recorded-audio fallback.
- Per-bird interaction aggregation for analytics, training, recommendations, or population dashboards.

### 2.3 Product Invariants

These invariants should be encoded in architecture docs, automated tests where possible, and release checklists:

- Clients never write personality vectors, mood snapshots, drift totals, notebook entries, or canonical bird positions directly.
- The server simulation tick is the only writer of canonical bird state.
- Interaction events are append-only and processed in deterministic order.
- Drift moves traits monotonically toward expressive on positive presence and never downward on absence.
- Presence requires visible document, focused window, and recent pointer or keyboard activity.
- Bird identity is stable across renames, sync, migrations, and species asset updates.
- The user never sees hidden numerical personality or drift values.
- Naturalist voice is used only for product surfaces; account, sync, settings, error, and accessibility configuration surfaces use matter-of-fact system voice.
- No welcome toast, banner, modal, or "you have been gone" text exists.
- Visits are render-only and do not contribute presence, interactions, or drift.
- Accessibility modes are alternate presentations of the same aviary, not stripped fallbacks.

## 3. System Architecture

### 3.1 High-Level Shape

Use a web client plus server-backed simulation architecture:

- Web client: React or comparable component framework, Canvas/WebGL or SVG-plus-Canvas rendering layer, WebAudio call synthesis, keyboard and accessibility surfaces, and local ephemeral interpolation state.
- API edge: HTTP JSON endpoints for auth, state snapshots, interactions, settings, notebook reads, visits, exports, and account management.
- Simulation service: scheduled per-aviary tick worker that advances canonical state from recent event logs and wall-clock inputs.
- Data store: relational primary database for accounts, sessions, aviaries, birds, interaction events, snapshots, notebook entries, invitations, and settings.
- Job system: queues for magic-link delivery, account export generation, deletion finalization, notebook candidate generation, visit-email delivery, and periodic simulation work.
- CDN/edge cache: serves the application shell and optionally delivers a signed initial state envelope for fast first render.
- Observability pipeline: aggregate-only metrics, traces, and logs with explicit redaction and schema guardrails.

The client should be optimized for fast first bird and believable interpolation. The server should be optimized for correctness, determinism, privacy isolation, and low-latency snapshots.

### 3.2 Recommended Service Boundaries

Start with a modular monolith plus workers unless scale proves otherwise. The key is code ownership boundaries, not premature microservices.

Suggested modules:

- `auth`: magic links, sessions, email changes, session revocation.
- `accounts`: account profile, encrypted email storage, deletion, export.
- `aviaries`: aviary records, bird adoption, settings, timezone, bird naming.
- `simulation`: tick scheduling, state transition functions, drift, mood, weather, notebook triggers.
- `events`: append-only event ingestion, validation, idempotency, sequencing.
- `snapshots`: state read models for hosts and visitors.
- `notebook`: sparse naturalist observation generation and read APIs.
- `visits`: invitations, visit sessions, revocation, visit logs.
- `telemetry`: aggregate operational metrics wrappers and redaction helpers.

A modular monolith keeps the simulation transactionally close to the database while allowing worker processes to run separately. If later split is needed, `simulation` and `auth/email` are the first candidates.

### 3.3 Client/Server Split

Server responsibilities:

- Own account identity, sessions, and authorization.
- Own canonical aviary state.
- Own bird identity, species, names, personality vectors, moods, mood timers, drift totals, and current coarse positions.
- Advance state on scheduled ticks.
- Validate and append interaction events.
- Generate or persist notebook entries.
- Generate export snapshots.
- Enforce visit permissions and revocation.
- Provide small state snapshots to clients.

Client responsibilities:

- Render the latest state snapshot as a continuous scene.
- Interpolate bird movement and pose transitions between snapshots.
- Generate purely ornamental ambient leaves and feathers.
- Synthesize procedural calls with WebAudio from grammar parameters supplied in snapshots or static species metadata.
- Caption calls from the same grammar execution.
- Detect presence conditions and submit bounded presence pings.
- Submit user interaction events with idempotency keys.
- Present account, settings, accessibility, notebook, and visit surfaces.
- Apply reduced-motion and accessibility modes.

The client may maintain ephemeral animation phase, audio envelopes, focus state, and pending interaction UI, but none of that becomes canonical unless sent as an event and accepted by the server.

### 3.4 Render Pipeline Boundary

The state snapshot should contain semantic render instructions, not final animation frames:

- Bird IDs, species IDs, names.
- Current mood and pose family.
- Perch zone and normalized perch coordinate.
- Current transition intent, if any.
- Greeting candidate state if a return-greeting is active.
- Call scheduling windows and call grammar seed values.
- Day/night phase, weather state, settled state.
- Per-bird caption-eligible call metadata.

The client turns this into:

- Exact positions for the current viewport.
- Pose interpolation or reduced-motion cross-fades.
- Audio synthesis.
- Captions.
- Screen-reader narration.

This split keeps bandwidth small and avoids pretending that server ticks operate at animation-frame rate.

## 4. Data Model

### 4.1 Core Tables

Use UUID primary keys for all entities. Email is stored only on the account record, encrypted. Internal references use synthetic account IDs.

#### `accounts`

- `id`: UUID.
- `encrypted_email`: encrypted string.
- `email_hash`: keyed hash for lookup and rate limiting; never logged.
- `email_verified_at`: timestamp.
- `timezone`: IANA timezone, captured from client and updateable.
- `created_at`, `updated_at`.
- `soft_deleted_at`, `hard_delete_after`.
- `privacy_policy_version_acknowledged`.

#### `sessions`

- `id`: UUID.
- `account_id`: UUID.
- `device_label`: optional user-editable label or derived description.
- `token_hash`: hashed session token.
- `created_at`, `last_seen_at`, `revoked_at`, `expires_at`.
- `user_agent_family`: coarse, non-identifying.

#### `magic_links`

- `id`: UUID.
- `email_hash`: keyed hash.
- `account_id`: nullable until account creation/link.
- `token_hash`.
- `created_at`, `expires_at`, `consumed_at`.
- `request_ip_bucket`: privacy-preserving rate-limit bucket.

#### `aviaries`

- `id`: UUID.
- `account_id`: UUID unique.
- `created_at`.
- `timezone`: denormalized current timezone for simulation.
- `current_state_version`: monotonic integer.
- `last_tick_at`, `next_tick_after`.
- `settled_until_reengage`: boolean or state enum.
- `weather_state`: enum plus start/end timestamps.
- `day_phase_cache`: optional derived cache.
- `settings_json`: top-level aviary preferences that are not bird-specific.

#### `birds`

- `id`: UUID stable identity.
- `aviary_id`: UUID.
- `species_id`: enum/string from V1 species pool.
- `display_name`: user-controlled.
- `adopted_at`.
- `adoption_order`.
- `active`: boolean.
- `current_mood`: enum.
- `mood_entered_at`.
- `mood_intensity`: small normalized value if needed for transition smoothing.
- `perch_zone`: `front`, `middle`, `back`.
- `perch_slot`: normalized coordinate within zone.
- `pose_state`: enum or structured pose descriptor.
- `call_signature_seed`: stable seed.
- `visual_seed`: stable seed.
- `created_at`, `updated_at`.

#### `bird_personality_vectors`

- `bird_id`: UUID primary key.
- `boldness`: normalized decimal.
- `social_warmth`: normalized decimal.
- `vocal_frequency`: normalized decimal.
- `plumage_saturation`: normalized decimal.
- `curiosity`: normalized decimal.
- `updated_at`.
- `schema_version`.

Store vectors separately to make write ownership and audit checks strict. Only simulation code should have write access.

#### `bird_drift_ledgers`

- `id`: UUID.
- `bird_id`: UUID.
- `tick_id`: UUID.
- `trait`: enum.
- `delta`: decimal, non-negative for expressive traits.
- `input_window_start`, `input_window_end`.
- `reason_code`: enum such as `presence`, `listen_in`, `offer`, `age_calibration`.
- `created_at`.

This table is internal only. It supports debugging, calibration, and export. It must not feed aggregate analytics.

#### `interaction_events`

- `id`: UUID.
- `aviary_id`: UUID.
- `account_id`: UUID.
- `bird_id`: nullable UUID.
- `client_event_id`: idempotency key generated by client.
- `event_type`: enum.
- `occurred_at_client`: timestamp.
- `received_at_server`: timestamp.
- `sequence_number`: monotonic per aviary assigned by server.
- `payload_json`: validated bounded payload.
- `consumed_by_tick_id`: nullable UUID.
- `rejected_at`, `rejection_reason`: nullable.

Event types:

- `presence_ping`
- `presence_window_end`
- `listen_in_start`
- `listen_in_end`
- `offer_seed`
- `offer_song_fragment`
- `offer_still_pool`
- `settle_start`
- `settle_undo`
- `settle_confirmed`
- `bird_rename`
- `accessibility_mode_changed`

`bird_rename` is an account action, not simulation input for drift. Keep it in either a separate audit table or typed event with explicit exclusion from drift.

#### `presence_windows`

Derived or materialized from pings:

- `id`: UUID.
- `aviary_id`, `account_id`, `session_id`.
- `started_at`, `ended_at`.
- `valid_presence_seconds`.
- `source_event_first_sequence`, `source_event_last_sequence`.
- `closed_reason`: `hidden`, `blurred`, `activity_timeout`, `settle`, `tab_close_inferred`, `session_expired`.

Materializing presence windows makes simulation calibration auditable and avoids repeatedly reconstructing from raw pings.

#### `aviary_ticks`

- `id`: UUID.
- `aviary_id`.
- `started_at`, `completed_at`.
- `input_sequence_start`, `input_sequence_end`.
- `previous_state_version`, `new_state_version`.
- `status`: success, skipped, failed_retryable, failed_terminal.
- `duration_ms`.
- `error_code`: nullable.

#### `state_snapshots`

Persist current canonical state in normalized tables plus optionally a compact read-model:

- `aviary_id`.
- `state_version`.
- `snapshot_json`.
- `generated_at`.
- `expires_at`.

The JSON read model is safe because it is generated from canonical tables. It should not be the sole source for personality vectors.

#### `notebook_entries`

- `id`: UUID.
- `aviary_id`.
- `created_at`.
- `observed_at`.
- `entry_text`: naturalist prose, lowercase.
- `entry_type`: `greeting_order`, `quiet_stretch`, `weather`, `offer_reaction`, `mood_pattern`, `drift_visible`, etc.
- `source_tick_id`: nullable.
- `source_bird_ids`: array of UUIDs.
- `rarity_score`: internal.

Notebook entries are read-only to users and not editable.

#### `visit_invitations`

- `id`: UUID.
- `host_account_id`.
- `aviary_id`.
- `visitor_email_encrypted`.
- `visitor_email_hash`.
- `token_hash`.
- `created_at`, `expires_at`, `accepted_at`, `revoked_at`.
- `revoked_by_session_id`.

#### `visit_sessions`

- `id`: UUID.
- `invitation_id`.
- `aviary_id`.
- `visitor_email_hash`.
- `created_at`, `last_snapshot_at`, `ended_at`.
- `end_reason`: `expired`, `revoked`, `visitor_closed`, `inactive`.

Visitor identity is not represented inside the aviary. Visit sessions only authorize read snapshots and support the host's on-demand visit log.

#### `account_exports`

- `id`: UUID.
- `account_id`.
- `requested_at`, `completed_at`, `expires_at`.
- `download_token_hash`.
- `status`.
- `object_storage_key`.

### 4.2 Static Content and Config

Keep versioned static definitions in code or configuration:

- Species pool: species ID, silhouette metadata, palette ranges, motif library IDs, night activity flag.
- Mood definitions: allowed states, transition weights, pose families, call modifiers.
- Offer definitions: seed, song fragment, still pool; cooldowns; mood/personality response maps.
- Call grammar primitives: motifs, pitch ranges, rhythm patterns, timbre parameters, caption templates.
- Notebook templates and generation rules.
- Accessibility narration templates.
- Performance and simulation calibration constants.

Version these definitions so migrations can preserve bird identity and call recognizability if assets change.

### 4.3 User Export Shape

Account export JSON should include:

- Account metadata: created date, timezone, settings, current session list excluding tokens.
- Aviary: created date, current state summary, weather, settled state.
- Birds: IDs, names, species, adopted dates, current mood, personality vector values, drift ledger summaries.
- Notebook entries.
- Visit invitations and visit log.
- Privacy-readable interaction summary if necessary, but avoid dumping raw noisy presence pings unless explicitly included in export requirements.

Export may include personality vector numbers because the PRD explicitly says export includes current personality vectors. This must remain outside normal product UI and be delivered as a data-rights/account export surface, not as an in-product stats panel.

## 5. API Surface

### 5.1 API Principles

- All write endpoints require authenticated host sessions except visit acceptance and magic-link flows.
- All mutation endpoints accept idempotency keys.
- All snapshot responses include `state_version`, `server_time`, and cache-control semantics.
- Clients submit events, not canonical state.
- Visitors have separate read-only authorization scoped to one aviary and one invitation.
- Matter-of-fact error messages for auth, sync, visit revocation, unsupported browser, and system failures.

### 5.2 Auth and Account APIs

`POST /api/auth/magic-link`

Request:

```json
{ "email": "user@example.com" }
```

Behavior:

- Normalize email for lookup.
- Rate-limit by keyed email hash and coarse IP bucket.
- Create a 15-minute magic link.
- Email a one-time link.
- Return generic success regardless of account existence.

`POST /api/auth/magic-link/consume`

Request:

```json
{ "token": "opaque" }
```

Behavior:

- Validate unused and unexpired token.
- Create account if needed.
- Create aviary and starter birds if this is first sign-in.
- Invalidate token immediately.
- Issue per-device session token.
- Return initial account and aviary bootstrap pointer.

`GET /api/account`

Returns matter-of-fact account settings, sessions, timezone, accessibility preferences, visit settings.

`PATCH /api/account`

Updates settings such as timezone, accessibility defaults, visit notification opt-in.

`POST /api/account/email-change`

Starts verified email-change flow.

`POST /api/account/sessions/{sessionId}/revoke`

Revokes a session.

`POST /api/account/export`

Creates export job and emails download link to verified address.

`POST /api/account/delete`

Soft-deletes account for 30 days.

`POST /api/account/delete/cancel`

Restores within soft-delete window.

### 5.3 Aviary Snapshot APIs

`GET /api/aviary/bootstrap`

Returns the current snapshot plus static version references needed for immediate rendering:

```json
{
  "state_version": 1234,
  "server_time": "2026-06-05T14:41:00Z",
  "aviary": {
    "id": "uuid",
    "created_at": "...",
    "timezone": "America/Los_Angeles",
    "day_phase": "morning",
    "weather": { "type": "none" },
    "settled": false
  },
  "birds": [
    {
      "id": "uuid",
      "name": "pip",
      "species_id": "warbler_v1",
      "mood": "content",
      "perch_zone": "front",
      "perch_slot": 0.42,
      "pose": "preen_soft",
      "visual": { "plumage_saturation_band": "low_plus", "seed": "..." },
      "call": { "signature_seed": "...", "next_window": "...", "grammar_version": "v1" }
    }
  ],
  "static_versions": {
    "species": "2026.06.v1",
    "call_grammar": "2026.06.v1",
    "render_config": "2026.06.v1"
  }
}
```

Do not include raw personality vector values.

`GET /api/aviary/snapshot?after_version=1234`

Returns a fresh snapshot or 204/no-change if unchanged and within polling window. Triggered on visibility return, long frame gap, and low-frequency visible keepalive.

`GET /api/aviary/notebook?cursor=...`

Returns paginated notebook entries newest-first or oldest-first depending on UI decision. Entries are read-only.

`PATCH /api/aviary/birds/{birdId}`

For rename only:

```json
{ "display_name": "pippa", "client_event_id": "uuid" }
```

This writes a name/account event, not a personality event.

### 5.4 Interaction Event API

`POST /api/aviary/events`

Accepts a batch of validated events:

```json
{
  "client_batch_id": "uuid",
  "events": [
    {
      "client_event_id": "uuid",
      "type": "presence_ping",
      "occurred_at": "2026-06-05T14:41:10.000Z",
      "payload": {
        "visibility": "visible",
        "has_focus": true,
        "recent_activity": true,
        "activity_age_ms": 42000
      }
    }
  ]
}
```

Validation:

- Event type must be known.
- `occurred_at` cannot be too far in the future or implausibly old.
- Event payload must match schema and size limits.
- Bird IDs must belong to the user's aviary.
- Offer cooldowns are checked server-side.
- Settle undo is accepted only within the configured window.

Response:

```json
{
  "accepted": ["uuid"],
  "rejected": [{ "client_event_id": "uuid", "reason": "offer_cooldown" }],
  "server_time": "...",
  "next_snapshot_recommended_after_ms": 5000
}
```

The client should optimistically animate low-risk reactions only if it can reconcile with the next snapshot. For offer outcomes that affect mood or drift, prefer server-confirmed reaction windows to avoid divergent behavior.

### 5.5 Offer Flow API

Offers can use the general event API. Define event payloads:

- `offer_seed`: `bird_id` optional if offered to aviary rather than direct bird; target resolution can be server-side.
- `offer_song_fragment`: `fragment_id` from a small static library.
- `offer_still_pool`: placement hint optional, but client should not control final bird response.

Server response or subsequent snapshot should include a reaction descriptor:

```json
{
  "reaction": {
    "id": "uuid",
    "offer_event_id": "uuid",
    "bird_id": "uuid",
    "reaction_type": "approach_wait_then_peck",
    "starts_at": "...",
    "duration_ms": 12000,
    "caption_hint": "pip waits, then steps toward the seed."
  }
}
```

Reaction descriptors are not announcements; they drive rendering and narration.

### 5.6 Listen-In Flow

Use events:

- `listen_in_start` with `bird_id`.
- `listen_in_end` with `bird_id` and reason.

Client may begin audio ramp immediately for responsiveness, then reconcile if server rejects due to auth/session failure. Listen-in affects drift through consumed event duration, so the server computes final duration from event sequence and timestamps.

### 5.7 Settle Flow

Use events:

- `settle_start`.
- `settle_undo` if any click or keyboard re-engagement within five seconds.
- `settle_confirmed` generated by client after timer or inferred by server if no undo arrives.

The server treats settle as ending the presence window and mood-quieting. It must not create a required goodbye obligation.

### 5.8 Visit APIs

`POST /api/visits/invitations`

Host sends invite:

```json
{ "visitor_email": "friend@example.com" }
```

Creates 30-day one-time visit link. Visits are off until this deliberate action.

`GET /api/visits/invitations`

Lists outstanding and historical invitations in account settings.

`POST /api/visits/invitations/{id}/revoke`

Revokes immediately.

`POST /api/visits/accept`

Consumes or opens a visit token and creates a visit session.

`GET /api/visits/{visitSessionId}/snapshot`

Returns read-only snapshot using the same rendering state shape, excluding host-only settings, notebook mutation affordances, top-bar actions that imply interaction, and any host account details not needed for transparency.

Visitor snapshot requests:

- Do not create presence events.
- Do not trigger greetings.
- Do not affect call scheduling beyond observing current canonical state.
- Return matter-of-fact `visit_no_longer_available` when revoked or expired.

`GET /api/visits/log`

Host account settings only. Shows visitor email, dates, approximate duration, outstanding invitations. No badges or pushed alerts.

### 5.9 Error Surface Contract

Standard error object:

```json
{
  "error": {
    "code": "session_expired",
    "message": "Your session timed out. Sign in again to keep watching.",
    "retryable": false
  }
}
```

Messages in auth, sync, settings, visits, unsupported-browser, and accessibility settings are matter-of-fact. Product surfaces should not use these error strings as naturalist prose.

## 6. Simulation Engine Design

### 6.1 Tick Scheduling

The simulation tick should run approximately once per minute per active aviary, with adaptive scheduling:

- Recently active aviaries: target one-minute cadence.
- Inactive aviaries: still tick, but can batch and coalesce low-impact ticks while preserving day/night, mood, and drift continuity.
- Accounts marked for deletion: stop nonessential simulation and schedule deletion finalization.

Implementation:

- Maintain `next_tick_after` on each aviary.
- Worker claims due aviaries with row-level locking or advisory locks.
- Each tick runs in a transaction or a carefully bounded sequence:
  1. Claim tick.
  2. Load aviary, birds, vectors, moods, unconsumed events, recent presence windows, timezone.
  3. Compute deterministic transitions.
  4. Apply vector deltas and mood updates.
  5. Update bird positions and pose state.
  6. Possibly create notebook entry.
  7. Mark consumed event range.
  8. Increment state version and write snapshot read model.
  9. Release and schedule next tick.

Use deterministic seeded randomness based on `aviary_id`, `tick_id`, `bird_id`, and time bucket so retries do not produce different canonical results.

### 6.2 Presence Processing

Presence must be honest. Client pings are accepted only as evidence for the three required conditions:

- Document visibility is `visible`.
- Window has focus.
- Pointer or key activity occurred within the configured activity window.

Recommended client algorithm:

- Track `lastPointerOrKeyAt`.
- Track current `visibilityState`.
- Track focus/blur.
- Every 30 seconds while visible/focused, submit a presence ping with current booleans and activity age.
- Submit terminal event on blur, hidden, pagehide, settle, or activity timeout when possible.
- Use `sendBeacon` for pagehide if available.

Recommended server algorithm:

- Do not trust a single ping as an entire presence interval.
- Build presence windows from consecutive valid pings, capped by max gap.
- Clamp credited presence to server receipt windows to prevent fabricated long sessions.
- End presence on settle, hidden, blur, session expiry, or missing valid ping beyond grace period.
- Use a calibrated activity window, likely three to five minutes, because quiet watching is valid.

Presence windows should record credited seconds, not just event count. The simulation consumes credited seconds in each tick's input range.

### 6.3 Drift Function

Drift is a slow low-pass filter over presence and interactions. It must be measurable after about one week of regular visits and visible after about three weeks.

Use normalized trait values in `[0, 1]` internally, with initial values seeded by species and individual bird seed. Keep product-visible bands derived from values but never expose numbers.

For each bird and trait:

```text
daily_signal = weighted_sum(
  presence_time_for_aviary,
  listen_in_time_for_bird,
  offer_events_near_or_accepted_by_bird,
  age_and_species_baseline
)

target_expressive_value = trait_max_for_bird
delta = responsiveness * low_pass(daily_signal) * remaining_headroom
new_value = min(old_value + max(delta, 0), trait_cap)
```

Properties:

- `delta` is never negative due to absence.
- Presence affects all birds, with slight weighting for birds that were active or greeting.
- Listen-in strongly affects the focused bird's social warmth and vocal frequency.
- Offers modestly affect curiosity and boldness, subject to cooldown.
- Plumage saturation moves only from sustained presence, not rapid offer interactions.
- Social warmth and vocal frequency may become more expressive with repeated listen-in, but not so fast that one session is visible.
- Trait headroom prevents runaway saturation.

Calibration targets:

- Regular user: 10-20 minutes of valid presence most days.
- Instrument-visible movement after roughly seven days: small but statistically detectable deltas in internal tests.
- User-visible movement after roughly 21 days: changed greeting likelihood, perch preference, call frequency, and subtle plumage richness.
- No single session creates a visible trait jump.

Testing:

- Simulate cohorts with zero, low, regular, and high presence for 30 days.
- Assert no trait decreases from absence.
- Assert no trait crosses visible thresholds before minimum expected windows under normal use.
- Assert offer spamming cannot saturate curiosity due to cooldown and low-pass constraints.

### 6.4 Mood Transitions

Mood is a fast-timescale state per bird:

- Candidate V1 moods: `wary`, `content`, `curious`, `drowsy`, `alert`, `settled`.
- Mood persists across sessions.
- Mood shifts from recent interactions, local time, weather, bird-to-bird interactions, and personality.

Use a weighted transition model, not hard-coded if/else chains:

```text
score(next_mood) =
  base_by_current_mood
  + time_of_day_weight
  + weather_weight
  + recent_event_weight
  + bird_personality_weight
  + neighbor_bird_weight
  + seeded_noise_small
```

Rules:

- Dusk increases `drowsy` and `settled`.
- Morning increases `alert` and greeting readiness.
- Rain dampens vocal frequency and nudges some birds toward `drowsy` or `wary`.
- Wind nudges alertness or wary depending on personality.
- Accepted seed nudges toward `content` or `curious`.
- Song fragment can prompt joining, quieting, or counter-call depending on vocal frequency and mood.
- Still pool can prompt drinking, bathing, or watching depending on curiosity and drowsiness.
- High boldness resists `wary`.
- High social warmth increases response to other birds.

Mood transition outputs drive:

- Perch zone preference.
- Pose family.
- Greeting likelihood.
- Call rate and motif modifiers.
- Narration and notebook candidate facts.

### 6.5 Return Greeting

Return-greeting is a session entry behavior driven by canonical state and absence length. It must not be a client-only canned animation.

Inputs:

- Last valid host presence end time.
- Absence duration bucket: short, same day, overnight, multi-day.
- Bird boldness, social warmth, mood, recent greeting history.
- Local time of day.
- Current perch and pose.

Algorithm:

1. On bootstrap or visibility return after absence, server includes a `greeting_window` if eligible.
2. Rank birds by greeting score:
   - Boldness.
   - Social warmth.
   - Mood readiness.
   - Whether the bird greeted recently, to avoid one bird always greeting.
   - Absence bucket.
3. Select one primary greeter.
4. Optionally select secondary response birds with staggered offsets, never simultaneous all-bird greeting.
5. Include a variation seed and behavior descriptor:
   - glance from preening.
   - quiet two-note call.
   - head tilt and small step forward.
   - longer call and another bird response.

The client renders the greeting naturally in the current scene. There is no toast, banner, modal, or text announcing return.

### 6.6 Perch and Pose Selection

Perch zones communicate mood and personality:

- Bold/content birds favor front or middle.
- Wary birds favor back.
- Drowsy birds favor stable, lower poses.
- Curious birds shift toward offered objects or sounds.

Use server state for coarse perch zone and transition intent. Client maps that to exact responsive coordinates. The user cannot control perches.

### 6.7 Weather and Day/Night

Day/night:

- Derived from account timezone and server time.
- Snapshot includes phase and gradient parameters.
- Client renders continuous transitions.
- Evening quiets calls and nudges drowsiness.
- Night is dim but not dead; nightjar-like species may remain active.

Weather:

- Rare, mild events only: short rain and soft wind.
- Server schedules weather events per aviary with seeded randomness and frequency caps.
- Weather affects mood and call rate modestly.
- Client renders rain/wind visually, with reduced-motion alternatives.

Weather should never become a feature users manage or a notification-worthy event.

### 6.8 Offer Reactions and Cooldowns

Offer cooldown:

- Enforce per bird and per offer family server-side, on the order of a few minutes.
- Cooldown rejection should be quiet and matter-of-fact only if surfaced in controls; avoid punitive language.

Targeting:

- User offers to the aviary through top bar, not by clicking a bird.
- Server decides which bird notices first based on mood, curiosity, boldness, proximity, and recent offer history.

Reactions:

- Seed: approach, wait then approach, ignore, watch, peck.
- Song fragment: join in, quiet, call against, tilt head.
- Still pool: drink, bathe, watch reflection, ignore while drowsy.

Drift:

- Accepted or investigated offers give small curiosity deltas.
- Offers near a bird give small boldness deltas.
- Cooldown and low-pass filtering prevent single-session farming.

### 6.9 Field Notebook Generation

Notebook entries are rare, specific, naturalist observations. They are not event logs.

Generation pipeline:

1. Simulation tick emits notebook candidates when noteworthy conditions happen:
   - First greeter changed.
   - Bird spent unusual time on a perch.
   - Quiet stretch after rain.
   - Offer reaction with distinctive mood.
   - Visible drift threshold crossed.
   - Bird-to-bird call pattern emerged.
2. Candidate scorer applies sparsity rules:
   - Roughly one entry every few days for regularly visited aviaries.
   - More often only for genuinely noteworthy events.
   - No entries that describe user visit frequency or streaks.
   - No entries exposing numerical traits.
3. Template renderer produces lowercase present-tense prose with bird names and concrete details.
4. Entry is persisted read-only.

Generation should be deterministic enough for tests and reviewable enough for tone QA. Avoid free-form LLM generation in V1 unless heavily constrained and privacy-reviewed; templated naturalist prose with varied slots is safer, cheaper, and easier to keep on voice.

### 6.10 Call Grammar Runtime

Each species has motif libraries. Each bird has a stable call signature seed. Personality and mood shape runtime variation.

Call scheduling:

- Server snapshot includes call windows or rate parameters.
- Client schedules actual WebAudio calls within those windows using deterministic seeds.
- High vocal frequency increases chance of unobserved calls and chorus participation.
- Listen-in changes mix, not canonical call existence.

Call synthesis parameters:

- Motif sequence.
- Pitch contour.
- Timing gaps.
- Timbre/noise envelope.
- Volume envelope.
- Mood modifiers.

Recognizability:

- Stable bird seed keeps Pip recognizable across weeks.
- Drift changes rate and expressive variation, not the fundamental identity.
- Species defines broad motif family; individual seed defines bird identity.

Captioning:

- Captions are generated from the exact motif execution:
  - "a soft three-note rise"
  - "a low trill, paused, low trill again"
  - "a single sharp call from the back perch"
- Caption text should not be a static label per species.

## 7. Sync Model and Conflict Prevention

### 7.1 Canonical State

There is one canonical aviary record per account. All host devices and visitor sessions read from it. Only the server tick writes personality, mood, drift, and canonical bird state.

This removes client-to-client sync as a problem. Devices do not merge state; they submit events and read snapshots.

### 7.2 Event Ordering

Server assigns monotonic sequence numbers per aviary when events are accepted. The simulation tick consumes events by sequence number.

Requirements:

- Unique constraint on `(aviary_id, client_event_id)` for idempotency.
- Sequence assignment in a transaction.
- Tick records consumed sequence range.
- Late events can be accepted if within allowed window and consumed in a later tick, but cannot rewrite previous tick state.
- Very old events are rejected or treated as non-drift operational events, with matter-of-fact client handling.

### 7.3 Conflict Prevention

Avoid conflict surfaces by design:

- Personality: no client writes, no LWW.
- Mood: no client writes.
- Bird names: simple account mutation with optimistic concurrency. If two devices rename the same bird, latest accepted rename can win because name is user-owned text, not drift. Return updated state version.
- Settings: use per-setting last update with ordinary conflict handling.
- Sessions: revocation is immediate by token invalidation.
- Visit revocation: visitor next snapshot returns revoked.

Actual sync conflict UI should be rare and limited to auth/session/outage cases.

### 7.4 Multi-Device Behavior

When a second device opens:

- It signs in independently with magic link or existing session.
- It requests the current snapshot.
- It sees the same canonical aviary, mood, drift, names, notebook.
- Its presence pings count only when its own document is visible, focused, and recently active.

Concurrent host sessions:

- Multiple host devices can submit presence. The server should avoid double-counting the same user's simultaneous presence.
- Recommended rule: union valid presence windows per account across sessions for drift input, capped by wall-clock time. Two devices open for the same 10 minutes should credit at most 10 minutes, not 20.
- Listen-in and offers are event-specific and can be accepted from either device subject to cooldown and ordering.

### 7.5 Offline and Suspended Clients

V1 should not promise offline operation.

- If network drops, client continues rendering current snapshot for a short grace period but marks controls as unavailable matter-of-factly if writes fail.
- Do not advance canonical simulation locally.
- Do not accumulate long offline presence for later submission.
- On reconnect or visibility return, fetch snapshot and interpolate to current state.
- If the laptop suspends, detect long frame gap and fetch snapshot before resuming full animation.

## 8. Frontend Rendering Pipeline

### 8.1 Application Shell

Primary routes:

- `/`: authenticated aviary or sign-in.
- `/auth/consume`: magic-link consumption.
- `/settings`: account/settings modal or route.
- `/notebook`: notebook panel.
- `/visit/:token`: visit acceptance and read-only aviary.

Use code splitting:

- Initial aviary shell and renderer in critical bundle.
- Account settings, export, deletion, visit management, and notebook pagination lazy-loaded.
- Static species/render metadata optimized and cacheable.

### 8.2 First Render

Goal: first bird visible within 500ms on mid-tier mobile over 4G.

Implementation approach:

- Server-render or edge-embed a minimal bootstrap envelope when authenticated if feasible.
- Ship a small critical renderer capable of drawing quiet field and first birds before loading settings/notebook code.
- Preload species assets needed for current birds only.
- Use compact SVG or procedural vector assets for birds.
- Avoid blocking first bird on WebAudio initialization, notebook data, settings panel code, or full asset atlas.

Loading state:

- Quiet field with soft sky color and faint motion cues if snapshot is delayed.
- No spinner.
- No fade from static into aviary.
- Once snapshot arrives, place birds mid-action.

### 8.3 Scene Composition

Render one horizontal scene:

- No panning, scrolling, zooming.
- Three perch zones: front, middle, back.
- Responsive mapping from normalized perch coordinates to viewport pixels.
- Preserve aspect ratio enough to keep every bird visible.
- Wide desktop: more space between perches.
- Narrow phone: compressed spacing without cropping birds.

Layering:

- Background sky and foliage.
- Back perch zone.
- Middle birds and perches.
- Front ornaments and occasional foreground branch.
- Captions and focus indicators as accessibility overlays, not general UI chrome.
- Top bar above scene proper.

Do not put buttons, badges, labels, hover tooltips, or status icons inside the aviary scene.

### 8.4 Animation Model

Use a local animation state machine per bird:

- Input: server pose family, perch zone, mood, transition descriptor, call events.
- Output: current pose frame, transform, subtle body motion, head direction, beak movement.

Idle micro-motion:

- Preening.
- Scanning.
- Head-tilting.
- Weight shifts.
- Feather settling.
- Low drowsy posture.

Mood-shaped motion:

- Wary: back perch, more scanning, smaller movements.
- Content: preen, relaxed posture.
- Curious: head tilts, looks toward sounds/offers.
- Drowsy: fluffed, lower posture.
- Alert: sharper head movement and call readiness.

Avoid visible looping:

- Use stochastic timing and pose variation.
- Desynchronize birds.
- Keep motion slow.
- Never show all birds changing on the same beat.

### 8.5 Reduced-Motion Rendering

Reduced-motion mode is not static.

Implementation:

- Replace frame-by-frame micro-motion with slow cross-fades between still poses.
- Replace flight or perch moves with cross-fades or gentle opacity/position blends.
- Remove leaf and feather drift.
- Keep slow day/night color transitions, possibly at reduced rate.
- Keep mood, calls, captions, notebook, and drift unchanged.

The renderer should expose a `motionProfile` abstraction:

- `full`: normal micro-motion and transitions.
- `reduced`: pose cross-fades and no ambient drifting ornaments.

This prevents bolting reduced motion onto every animation manually.

### 8.6 Top Bar

Top bar contains:

- Account/settings.
- Accessibility settings.
- Field notebook.
- Offer affordance.

Rules:

- Sparse icons only.
- Full opacity on pointer movement, keyboard activity, or focus within top bar.
- Fade nearly transparent after a few seconds of stillness.
- Remain discoverable for keyboard and screen-reader users; do not hide from accessibility tree.
- In reduced-motion mode, fade should be instant or slow opacity change according to motion preference.

### 8.7 Interaction Details

Listen-in:

- Click/tap bird, keyboard focus bird plus Enter.
- Gradual audio ramp up for focused bird.
- Other birds ramp down but never silence.
- Disengage by second click, focus another bird, empty-scene click, Escape, or focus leaving aviary.
- Visible focus should be subtle but accessible.

Offer:

- Top-bar affordance opens small calm menu.
- Options: seed, song fragment, still pool.
- Keyboard navigable.
- Submit event and wait for reaction descriptor or next snapshot.
- Do not make offer affordance feel like an inventory or action bar.

Settle:

- Trigger from top bar.
- Lighting shifts to evening over a few seconds.
- Calls quiet.
- Any click or key re-engagement in five seconds undoes.
- Closing tab without settling is equally valid; no recovery surface.

Notebook:

- Top-bar icon opens read-only panel.
- Naturalist entries, scrollable indefinitely.
- No edit, delete, annotate, react, share, or export-from-notebook affordance.

### 8.8 Screen Reader and Semantics

Provide:

- A named aviary region.
- Roving keyboard focus for birds.
- Naturalist narration live region with polite updates.
- Controls with matter-of-fact labels.
- Captions available visually and programmatically.

Avoid:

- ARIA labels that expose raw mood enums as mechanical status.
- High-frequency live-region updates.
- Repeating every animation state change.

Narration manager:

- Idle updates every 30-60 seconds.
- Priority updates for return-greeting, offer reaction, settle.
- Queue coalescing to avoid overwhelming the screen reader.
- Same naturalist voice as notebook.

### 8.9 Browser and Device Support

Support latest two major versions of Chrome, Safari, Firefox, and Edge.

Detect:

- WebAudio availability.
- Canvas/WebGL or chosen renderer capability.
- Reduced motion preference.
- Visibility/focus APIs.

Unsupported browsers receive matter-of-fact page:

"Pocket Aviary needs a newer browser to run. Use the latest version of Chrome, Safari, Firefox, or Edge."

Do not ship old-browser compatibility paths that bloat the critical bundle.

## 9. Audio Pipeline

### 9.1 Goals

Audio must make birds recognizable without using loops. The system needs:

- Per-species motif families.
- Per-bird stable call identity.
- Mood and personality variation.
- Real-time chorus mixing.
- Listen-in mix balancing.
- Captions generated from actual calls.
- Graceful silence with captions when WebAudio is unavailable.

### 9.2 WebAudio Architecture

Client audio modules:

- `AudioContextManager`: handles permission, resume/suspend, lifecycle.
- `CallScheduler`: schedules call events from snapshot parameters.
- `CallSynthesizer`: creates oscillator/noise/envelope nodes for motif execution.
- `BirdVoiceRegistry`: maps bird IDs to species motifs and stable signature seeds.
- `Mixer`: ambient bed, per-bird gain nodes, listen-in ramps, master output.
- `CaptionEmitter`: emits caption descriptors from motif execution.
- `AudioTelemetry`: aggregate-only errors and initialization timing.

Use reusable node pools where feasible. Avoid per-call allocations that leak. Clean up scheduled nodes.

### 9.3 Call Grammar

Represent a motif as structured data:

```json
{
  "motif_id": "warbler_three_note_rise",
  "segments": [
    { "shape": "sine", "duration_ms": 90, "pitch_delta": 0 },
    { "shape": "sine", "duration_ms": 110, "pitch_delta": 3 },
    { "shape": "sine", "duration_ms": 130, "pitch_delta": 7 }
  ],
  "caption_template": "a soft three-note rise"
}
```

Runtime modifies:

- Base pitch from species and bird seed.
- Timing jitter from seed and mood.
- Volume envelope from mood and distance/perch.
- Call frequency from vocal-frequency trait band.
- Joining probability from social warmth and chorus context.

### 9.4 Listen-In Mix

When listen-in starts:

- Focused bird gain ramps up over 800-1500ms.
- Non-focused bird gains ramp down to ambient floor, not zero.
- Ambient scene remains present.
- Captions may prioritize focused bird if captions are on.

When listen-in ends:

- Gains return to ambient balance over the same slow ramp.

This must feel like leaning attention toward one bird, not soloing a track.

### 9.5 WebAudio Fallback

If WebAudio cannot run:

- Do not download recorded fallback audio.
- Enable call captions by default for that session.
- Show a matter-of-fact note in accessibility/audio settings, not as an aviary overlay.
- Keep all visual, narration, drift, and notebook behavior.

### 9.6 Audio Testing

Automated:

- Unit tests for grammar-to-caption parity.
- Determinism tests for seeded call identity.
- Memory tests across 30-minute synthetic sessions.
- Mixer tests ensuring non-focused birds never reach zero gain.

Human QA:

- Recognizability of two, five, and seven birds.
- No obvious loops after 10 minutes.
- Listen-in ramp feels gradual.
- Captions match perceived call shape.

## 10. Accessibility Implementation

### 10.1 Accessibility Is V1-Critical

Accessibility features ship with V1, not as follow-up work. Treat them as product surfaces with the same design QA as visuals and audio.

### 10.2 Screen-Reader Narration

Build a narration generator that consumes the same snapshot model as renderer.

Inputs:

- Bird species, name, perch zone, mood, pose.
- Day phase and weather.
- Greeting/reaction descriptors.
- Call events if relevant.

Output:

- Lowercase, present-tense naturalist prose.
- No raw state labels or numbers.
- No user behavior summaries.

Cadence:

- Idle narration every 30-60 seconds.
- Immediate but polite narration for user-initiated events.
- Coalesce overlapping events.

Examples:

- "pip is near the front rail, turning her head toward a soft call from the back perch."
- "rain passes lightly through the aviary. wren is quiet under the high branch."

### 10.3 Captions

Caption settings:

- Off by default when audio works, user-toggleable.
- On by default when WebAudio is unavailable or audio permission fails.

Caption behavior:

- Appears near calling bird.
- Fades in and out with call.
- Uses naturalist prose fragments.
- Generated from actual call grammar execution.
- Keyboard and screen-reader accessible.

### 10.4 Keyboard Navigation

Keyboard map:

- Tab: top bar items, then aviary region.
- Arrow keys: move focus between birds inside aviary.
- Enter/Space on focused bird: listen-in toggle.
- Escape: exit listen-in or close current panel.
- Top-bar shortcuts: open offer, notebook, settings if product/design approves.

Focus:

- Visible outline against bright and dim scenes.
- Focus order stable and understandable.
- No hidden controls inside scene.

### 10.5 Reduced Motion

Respect `prefers-reduced-motion` on first load and allow override in accessibility settings.

Test reduced-motion mode independently:

- No drifting leaves/feathers.
- No flight paths.
- Pose changes via slow cross-fades.
- Day/night remains but softened.
- No parallax-heavy motion.
- Audio, captions, narration, notebook, and drift remain complete.

### 10.6 Contrast and Settings

All user-copy text meets WCAG AA:

- Top bar labels/tooltips if any.
- Settings pages.
- Account and auth forms.
- Error surfaces.
- Captions.
- Visual narration if displayed.
- Visit revocation and unsupported-browser surfaces.

Accessibility settings use matter-of-fact language because they are system surfaces.

## 11. Privacy, Security, and Data Governance

### 11.1 PII Boundary

Email:

- Stored encrypted only on account record and invitation record where necessary.
- Never used as database primary key, shard key, log identifier, telemetry dimension, or queue partition key.
- Use synthetic UUIDs internally.
- Use keyed hashes only for lookup/rate-limiting, never exposed.

Logging:

- Structured logs must reject fields named `email`, raw tokens, raw magic links, raw session tokens, and per-bird event payloads.
- Provide safe IDs: account UUID, aviary UUID, request ID.
- Avoid logging notebook text if it can be tied to account.

### 11.2 Simulation Data Boundary

Per-bird interaction events and personality state are used only to drive that user's simulation and export.

Forbidden:

- Analytics warehouse reads from simulation database tables containing per-account bird events.
- Aggregate dashboards of average drift, average offers per bird, popular bird names, or similar relationship-derived metrics.
- ML training on per-bird interaction history.
- Recommendation systems based on per-bird interactions.

Allowed:

- Aggregate operational metrics: request counts, latencies, error rates, anonymized session-duration histograms, render-frame timing, audio-context errors, simulation-tick latency.
- Metrics must not include account ID, bird ID, species if it risks relationship inference, or event payloads.

### 11.3 Auth Security

- Magic links expire in 15 minutes.
- Used links invalidated immediately.
- Session tokens stored hashed server-side.
- Secure, HTTP-only cookies for web sessions where possible.
- CSRF protection for session-authenticated writes.
- Rate-limit magic-link requests.
- Device/session revocation from settings.
- Email change verifies new address before commit.

### 11.4 Account Deletion

Flow:

1. User requests deletion in settings.
2. Account marked soft-deleted immediately.
3. User can sign in during 30-day window and cancel deletion.
4. After 30 days, hard-delete account, aviary, birds, vectors, events, notebook, visits, exports, sessions, telemetry linkages where applicable.

The hard-delete job should be idempotent and auditable.

## 12. Performance Budgets and Observability

### 12.1 Budgets

Initial JS bundle:

- Less than 2MB gzipped at first paint.
- Track per-route and critical bundle sizes in CI.
- Fail CI on budget regression unless explicitly approved.

Time to first bird:

- Less than 500ms on mid-tier mobile over 4G.
- Synthetic tests should measure navigation start to first bird paint.
- The first bird should render without waiting for audio, notebook, full settings, or non-current species assets.

Idle runtime:

- 60fps idle motion on a five-year-old mid-range laptop.
- No memory growth over 30 minutes.
- WebAudio buffers and nodes cleaned up.
- Worker and timer lifetimes bounded.

Simulation:

- Tick p99 latency alarm over 5 seconds.
- Tick failures retry safely without double-applying drift.
- Snapshot size remains in kilobytes.

### 12.2 Observability

Client aggregate metrics:

- Page load timing.
- First bird render timing.
- Render frame timing buckets.
- Long task counts.
- Audio context init success/failure.
- WebAudio fallback count.
- Memory growth synthetic checks.
- Unsupported browser counts.

Server aggregate metrics:

- Auth request counts and error rates.
- Snapshot latency and size.
- Event ingestion latency and rejection counts by reason.
- Tick duration and failure counts.
- Queue lag.
- Export job duration.
- Visit snapshot latency and revocation responses.

Privacy constraints:

- No per-account bird state in metrics.
- No raw interaction payloads.
- No account-level dimensions in aggregate dashboards.
- No average drift dashboards.

### 12.3 Synthetic Monitoring

Run scheduled synthetic browsers from common geographies:

- Authenticated warm path with seeded test account.
- First bird timing.
- Snapshot refresh.
- WebAudio initialization.
- Reduced-motion render.
- Screen-reader narration smoke check if tooling supports DOM live-region validation.

Use dedicated synthetic accounts excluded from product analytics and clearly tagged as operational.

## 13. Rollout Plan

### 13.1 Development Milestones

Milestone 1: Foundations

- Auth, accounts, sessions.
- Aviary and bird data model.
- Starter bird adoption.
- Snapshot read API.
- Static species definitions.
- Basic renderer with quiet field and two birds.

Exit criteria:

- New account sees two named starter birds.
- Snapshot renders in browser.
- No client writes canonical bird state.

Milestone 2: Simulation Core

- Tick worker.
- Event ingestion.
- Presence windows.
- Mood transitions.
- Drift ledger.
- Perch and pose updates.
- Deterministic seeded randomness.

Exit criteria:

- Simulated 30-day cohorts meet drift calibration assertions.
- Multi-device reads same canonical state.
- Absence never creates negative drift.

Milestone 3: Interaction Surface

- Return-greeting.
- Listen-in event and mix behavior.
- Offers and cooldowns.
- Settle and undo.
- Top bar fade.
- Notebook candidate generation and sparse entries.

Exit criteria:

- No textual welcome surfaces.
- Offer spamming cannot saturate drift.
- Notebook entries are specific and sparse.

Milestone 4: Audio and Accessibility

- WebAudio call grammar.
- Chorus mixing.
- Call captions.
- Screen-reader narration.
- Reduced-motion renderer.
- Keyboard navigation.
- WCAG AA pass for text surfaces.

Exit criteria:

- Accessible surfaces pass design and QA review.
- WebAudio fallback enables captions without recorded audio.
- Keyboard-only user can operate core flows.

Milestone 5: Accounts, Sync, Privacy, Visits

- Account export.
- Deletion.
- Session revocation.
- Email change.
- Visit invitations, revocation, visit log.
- Privacy telemetry guardrails.

Exit criteria:

- Visitor cannot interact or create presence.
- Revocation ends visit on next snapshot.
- Export includes required state.
- Hard-delete job removes account data.

Milestone 6: Performance, Calibration, Beta

- Bundle budget enforcement.
- First-bird optimization.
- 30-minute memory tests.
- Synthetic monitoring.
- Drift calibration tuning.
- Tone QA pass.

Exit criteria:

- Meets performance budgets.
- Human QA confirms calls avoid obvious loops.
- Seven-bird chorus remains recognizable enough for cap validation.
- Launch checklist has no invariant violations.

### 13.2 Bird Count Ramp

V1 starts all accounts with two birds. Additional birds become available by aviary age only.

Suggested pacing for V1:

- Day 0: two starter birds.
- Around 2-3 months: third bird invitation appears.
- Later intervals increase slowly, with maximum seven.

Do not use visit count, engagement, offers, streaks, payment, or achievements for bird availability. The availability surface should be quiet and naturalist, not a reward announcement.

### 13.3 Beta Strategy

Private alpha:

- Team and invited testers.
- Focus on engine correctness, rendering stability, and tone violations.
- Use synthetic and test accounts for calibration; do not inspect real per-bird histories casually.

Closed beta:

- Broader browser/device matrix.
- Validate first-bird performance on real networks.
- Validate accessibility with users who rely on screen readers, captions, keyboard navigation, or reduced motion.
- Validate long-term drift with accelerated internal test fixtures and real-time beta observation.

Public V1:

- Keep invitations/social off by default.
- No push notifications.
- Monitor aggregate operational health.
- Ramp account creation if tick load or first-bird latency degrades.

### 13.4 Instrument From Day One

Instrument:

- Time to first bird.
- Bundle size.
- Snapshot latency and size.
- Tick p50/p95/p99.
- Tick failures.
- Event rejection counts.
- WebAudio failures.
- Frame timing buckets.
- Memory synthetic results.
- Export/deletion job success.

Do not instrument:

- Per-bird drift averages.
- Offer popularity by account/bird.
- Bird name popularity.
- Streaks or visit counts as product metrics.
- Visitor behavior beyond host-visible visit log and aggregate operational request counts.

## 14. Testing Strategy

### 14.1 Unit Tests

Simulation:

- Drift monotonicity.
- Drift calibration windows.
- Offer cooldown enforcement.
- Mood transition weighting.
- Presence window construction.
- Event ordering and idempotency.
- Deterministic tick retry.

Auth/account:

- Magic-link expiry and consumption.
- Session revocation.
- Email change verification.
- Soft/hard deletion.
- Export generation.

Audio:

- Grammar determinism.
- Caption parity.
- Mixer gain floors.
- WebAudio fallback branching.

Rendering:

- Responsive perch mapping keeps all birds visible.
- Reduced-motion profile suppresses disallowed motion.
- Top bar fade triggers correctly.

### 14.2 Integration Tests

- New account to first aviary render.
- Multi-device snapshot consistency.
- Two devices submitting events without personality conflict.
- Visibility return after long absence fetches fresh state.
- Settle ends presence and can undo within five seconds.
- Visit invite, accept, render, revoke.
- Visitor cannot submit interaction events.
- Account deletion removes dependent records after window.

### 14.3 End-to-End Tests

Browser matrix:

- Chrome, Safari, Firefox, Edge latest two majors.
- Mobile Safari and Chrome Android.
- Reduced-motion OS preference.
- WebAudio blocked/unavailable.
- Keyboard-only navigation.

Flows:

- Magic-link sign-in.
- Starter adoption and naming.
- Return-greeting after simulated absence.
- Listen-in engage/disengage.
- Offer each type.
- Open notebook.
- Change accessibility settings.
- Invite visitor and revoke.
- Export account.

### 14.4 Simulation Calibration Tests

Build accelerated simulation harnesses:

- Zero presence for 30 days: no negative drift, ambient continuity.
- Regular presence for 7 days: instrument-visible deltas.
- Regular presence for 21 days: visible threshold changes.
- Heavy offer spam: cooldown prevents saturation.
- Concurrent device sessions: presence union cap prevents double-counting.
- Long absence then return: greeting varies by absence, birds quieter but not distressed.

### 14.5 Tone and Product Invariant QA

Create a release checklist for forbidden surfaces:

- Search UI strings for welcome-back language.
- Search for streak, score, level, achievement, badge, XP, rank, leaderboard, hunger, happiness meter.
- Confirm no raw personality values in normal UI or accessibility labels.
- Confirm no calendar visit visualization.
- Confirm no notification defaults.
- Confirm no social discovery surfaces.
- Confirm no recorded audio assets in critical bundle.
- Confirm notebook entries never describe user visit frequency.

Tone review:

- Naturalist surfaces lowercase, present-tense, specific, no exclamation, no "you" framing.
- System surfaces capitalized, direct, matter-of-fact.
- Accessibility narration has the same charm as visual product.

## 15. Engineering Organization

### 15.1 Team Workstreams

Workstream A: Platform and data

- Auth, account, data model, migrations, deletion/export, privacy guardrails.

Workstream B: Simulation

- Tick worker, drift, mood, event processing, notebook candidates, calibration harness.

Workstream C: Web rendering

- Scene renderer, responsive layout, animation state machines, top bar, interaction UI.

Workstream D: Audio

- WebAudio synthesis, grammar, mixer, captions, memory/performance.

Workstream E: Accessibility and UX quality

- Screen-reader narration, keyboard model, reduced motion, contrast, tone QA.

Workstream F: Visits and sharing

- Invitations, visitor sessions, revocation, visit log, read-only rendering.

These workstreams need shared contracts:

- Snapshot schema.
- Event schema.
- Static species/call grammar schema.
- Tone string ownership.
- Privacy-safe telemetry wrapper.

### 15.2 Code Organization

Example monorepo layout:

```text
apps/web/
  src/aviary-renderer/
  src/audio/
  src/accessibility/
  src/routes/
  src/state/

apps/api/
  src/auth/
  src/accounts/
  src/aviaries/
  src/events/
  src/snapshots/
  src/visits/

apps/workers/
  src/simulation/
  src/email/
  src/export/
  src/deletion/

packages/contracts/
  snapshot.schema.ts
  events.schema.ts
  species.schema.ts
  call-grammar.schema.ts

packages/simulation-core/
  drift.ts
  mood.ts
  presence.ts
  weather.ts
  notebook-candidates.ts

packages/product-voice/
  naturalist-templates.ts
  system-messages.ts

packages/telemetry/
  metrics.ts
  redaction.ts
```

Keep complex work units in dedicated modules. Do not let simulation, rendering, audio, and API validation collapse into long route handlers or giant client components.

### 15.3 Review Gates

Require review from:

- Simulation owner for drift/mood/event changes.
- Privacy owner for telemetry, export, deletion, logging, visit changes.
- Accessibility owner for rendering, audio, captions, keyboard, narration.
- Product/design owner for text and tone surfaces.

Automated gates:

- Bundle budget.
- Type/schema checks.
- Unit and integration suite.
- Product-invariant string scan.
- Privacy redaction tests.
- Performance synthetic smoke.

## 16. Key Risks and Mitigations

### 16.1 Drift Calibration Risk

Risk: drift is too fast and feels gameable, or too slow and feels inert.

Mitigations:

- Build accelerated simulation harness early.
- Define measurable one-week and three-week thresholds.
- Keep offer effects small and cooldown-bound.
- Use low-pass smoothing and headroom.
- Tune with synthetic cohorts before public beta.

### 16.2 Presence Inflation Risk

Risk: tab-open or multi-device sessions inflate presence, making birds drift too quickly.

Mitigations:

- Enforce visible + focused + recent activity on server.
- Materialize presence windows with max gap.
- Union concurrent device presence by wall-clock time.
- Reject implausible backfilled presence.
- Test laptop-open-overnight and dual-device cases.

### 16.3 Sync Correctness Risk

Risk: client writes or LWW updates silently erase drift.

Mitigations:

- Database permissions or repository boundaries prevent client/API routes from writing vectors.
- Only simulation module can mutate personality.
- Sequence event log and tick consumed ranges.
- Integration tests for concurrent devices.
- Code review checklist for canonical-state ownership.

### 16.4 Audio Uncanniness Risk

Risk: procedural calls sound harsh, repetitive, or unrecognizable in chorus.

Mitigations:

- Prototype call grammar before full UI.
- Human listening sessions with 2, 5, and 7 birds.
- Stable per-bird seeds.
- Caption parity tests.
- No recorded fallback that would introduce canned loops.

### 16.5 Accessibility Regression Risk

Risk: accessible surfaces become mechanical labels or stripped fallbacks.

Mitigations:

- Accessibility implementation starts with renderer, not after it.
- Narration templates reviewed for voice.
- Reduced-motion mode has its own visual QA.
- Keyboard tests in CI.
- Screen-reader user beta before launch.

### 16.6 Performance Risk

Risk: renderer, audio, or settings code breaks 500ms first-bird and 2MB bundle budgets.

Mitigations:

- Critical bundle budget in CI from first milestone.
- Lazy-load noncritical surfaces.
- Compact assets and procedural generation.
- Synthetic first-bird monitoring.
- Memory tests for audio and renderer.

### 16.7 Privacy Boundary Risk

Risk: per-bird interaction data leaks into logs, analytics, or training.

Mitigations:

- Separate simulation storage from analytics pipeline.
- Redaction wrappers and log schema tests.
- Metrics API rejects account/bird dimensions.
- Privacy review for every telemetry addition.
- No aggregate drift dashboards.

### 16.8 Tone Creep Risk

Risk: contributors add harmless-seeming toasts, counters, badges, notifications, or generic event logs.

Mitigations:

- Product invariant checklist.
- String scans for forbidden language.
- Central product-voice package.
- Design review required for new surfaces.
- Document naturalist/system voice split in contributor guidelines.

### 16.9 Visit Scope Creep Risk

Risk: read-only visits become co-presence, comments, chat, profiles, or discovery.

Mitigations:

- Visitor auth token only permits snapshot reads.
- Visitor client has no interaction controls.
- Visit sessions cannot create events.
- No visitor identity in aviary scene.
- Visit log stays in settings with no badges.

## 17. Open Implementation Decisions

The PRD leaves several details to implementation. Recommended defaults:

- Activity window for presence: start with 4 minutes, tune between 3 and 5.
- Tick cadence: start with 60 seconds for active aviaries; batch inactive while preserving continuity.
- Mood enum: ship `wary`, `content`, `curious`, `drowsy`, `alert`, `settled`.
- Rendering technology: prefer Canvas/WebGL hybrid if bird animation complexity requires it; keep DOM for controls and accessibility. Use SVG assets if they meet 60fps and bundle budgets.
- Notebook generation: use constrained templates in V1, not unconstrained generative prose.
- Export personality vectors: include in account export as data rights, but never in ordinary UI.
- Concurrent host presence: credit union of valid windows, not sum.
- Initial snapshot delivery: use authenticated API first; add edge-embedded bootstrap if 500ms cannot be met.

Each decision should be revisited only with evidence from performance, accessibility, calibration, or privacy testing.

## 18. Launch Readiness Checklist

Before V1 launch:

- Two starter birds appear within 500ms target on synthetic mid-tier mobile path.
- Initial bundle is under 2MB gzipped.
- Server tick p99 below 5 seconds in load test.
- Drift harness meets one-week and three-week calibration targets.
- No negative drift from neglect.
- Multi-device sync test passes without LWW personality writes.
- Presence inflation tests pass.
- Seven-bird chorus remains individually recognizable enough for cap rationale.
- Reduced-motion mode is complete and reviewed.
- Screen-reader narration is naturalist, paced, and useful.
- Captions match actual call grammar.
- Keyboard navigation covers all interactive surfaces.
- WCAG AA text contrast passes.
- Visitor cannot interact, trigger greetings, or affect drift.
- Account export and deletion work end to end.
- Logs and metrics contain no raw emails, tokens, per-bird event payloads, or personality vectors.
- Product-invariant scan finds no gamification, streak, welcome-back, hunger, distress, public discovery, or notification-default surfaces.
- Human tone review approves notebook, narration, offer, settle, auth, sync, and settings copy.

## 19. Summary

The implementation should protect one idea: the user's quiet presence shapes a small living-feeling aviary over time. That requires server-owned continuity, honest presence accounting, slow monotonic drift, procedural calls, restrained UI, and accessibility surfaces that carry the same charm as the default visual/audio experience. The best V1 is not broad. It is precise, calm, privacy-preserving, and technically strict about the few rules that make the birds feel alive.
