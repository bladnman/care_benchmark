# Pocket Aviary V1 Implementation Plan

## 1. Product Framing and Execution Posture

Pocket Aviary V1 is a modern web-only, single-user virtual aviary with two starter birds, a maximum of seven birds over time, email magic-link accounts, a single canonical server-side aviary per account, multi-device sync, a quiet optional read-only visit flow, first-class accessibility surfaces, and a procedural visual/audio scene that feels alive without gamification.

The engineering plan should treat the PRD's affective constraints as hard technical requirements, not polish. The core implementation thesis is:

- The server owns the aviary's canonical simulation state.
- The browser renders, interpolates, synthesizes calls, submits interaction events, and never mutates personality state.
- The first visible frame must be an aviary already in motion, not an app loading into existence.
- User-visible product surfaces use naturalist voice; system surfaces use matter-of-fact voice.
- Presence is a precisely measured input to slow drift, not a generic engagement metric.
- Privacy is enforced by data boundaries, not just policy text.
- Accessibility surfaces deliver the actual product experience, not a simplified status feed.

The plan below is written as an execution blueprint for a frontier engineering team. It intentionally avoids implementing product code and focuses on architecture, contracts, data, algorithms, rollout, and risk controls.

## 2. V1 Scope

### In Scope

V1 includes:

- Browser-only Pocket Aviary, optimized for current Chrome, Safari, Firefox, and Edge.
- Single-user accounts with email magic-link sign-in.
- One canonical aviary per account.
- Two starter birds at adoption; additional birds unlocked by aviary age until a hard cap of seven.
- User-assigned and renameable bird names.
- A small coherent species pool of about six species, each with silhouette, palette, pose set, and procedural call motifs.
- Hidden server-side personality vectors per bird: boldness, social warmth, vocal frequency, plumage saturation, curiosity.
- Fast-timescale mood per bird: implementation-finalized set including wary, content, curious, drowsy, alert, and settled/night variants where needed.
- Server-side simulation tick at a slow cadence, approximately once per minute.
- Append-only interaction event ingestion for presence, listen-in, offers, settle, account/session events, and visit render sessions where appropriate.
- Precise presence accounting based on visibility, focus, and recent pointer/key activity.
- Return-greeting by one bird within the first second or two of an aviary becoming visible, varied by absence length, mood, and personality.
- Listen-in interaction with gradual audio mix rebalancing.
- Offer interaction for seed, song fragment, and still pool, with per-bird cooldowns.
- Settle gesture with slow evening lighting shift and five-second undo window.
- Field notebook with sparse naturalist observations.
- Thin top bar containing account/settings, accessibility settings, field notebook, and offer affordance.
- Top bar fade on cursor stillness and reappearance on pointer/key activity.
- Responsive single-scene aviary with no panning, scrolling, zooming, or birds cropped out of frame.
- Day/night cycle anchored to the user's local timezone.
- Rare quiet ambient weather and ambient visual micro-motion.
- Procedural client-side calls using WebAudio.
- Call captions generated from the same procedural grammar that produces the audio.
- WebAudio graceful-silence fallback with captions enabled by default.
- Screen-reader narration in naturalist prose.
- Reduced-motion rendering as a designed alternate visual register.
- Keyboard navigation across top bar, birds, listen-in, offer, settle, and settings.
- Account export as a JSON snapshot generated on demand and emailed to the verified address.
- Account deletion with 30-day soft-deletion and hard deletion afterward.
- Optional read-only visit invitations by email, off by default, revocable, expiring after 30 days.
- Host visit log in account settings, reachable on demand and never pushed.
- Aggregate-only operational telemetry for reliability and performance.

### Out of Scope

V1 explicitly excludes:

- Native iOS or Android apps.
- Password login, SSO, passkeys, household accounts, teams, or multi-profile accounts.
- Multiple aviaries per account.
- Shared aviaries or multi-user simulation.
- Client-authored personality state.
- Game mechanics of any kind: achievements, levels, scores, XP, badges, streaks, visit calendars, leaderboards, ranks, tiers, or "birds adopted" counters.
- Tamagotchi mechanics: hunger, health, distress, death, decaying happiness, or any punishment for absence.
- Push notifications or emails about the aviary's state.
- "Welcome back" banners, toasts, or text surfaces.
- Public discovery, profiles, follows, feeds, comments, chat, co-presence, avatars, visitor cursors, or mutual visits.
- Recorded audio call loops or recorded-audio fallback.
- User control over bird placement, perch assignment, scene customization, or catalog-style starter selection.
- Exposing personality numbers in product UI, debug UI, stats panels, or any optimization surface.

The account export is the only planned surface that can contain raw personality vector values because the PRD explicitly requires export of current aviary state, including vectors. Treat this export as a private machine-readable account data export, not a product display. It should sit behind account settings, require a fresh verification step or active session confirmation, and never render those values in the app.

## 3. Architecture Overview

### Service Shape

Use a small service architecture with clear ownership boundaries:

1. Web client:
   - TypeScript application.
   - React or an equivalent component layer for top bar, settings, notebook, auth, visit management, and accessibility controls.
   - Dedicated 2D scene renderer for the aviary, preferably Canvas/WebGL-backed with a retained scene graph, not DOM-heavy animation.
   - WebAudio procedural call engine.
   - Accessibility narration/caption layer.

2. Public API service:
   - Auth endpoints.
   - Snapshot endpoints.
   - Event ingestion endpoints.
   - Account/settings/export/delete endpoints.
   - Visit invitation and visitor snapshot endpoints.
   - Matter-of-fact error responses.

3. Simulation worker service:
   - Runs per-aviary ticks.
   - Consumes append-only interaction events in order.
   - Computes mood transitions, personality deltas, call schedules, notebook candidates, weather changes, adoption availability, and canonical snapshot state.
   - Is the only writer of personality vectors.

4. Email service integration:
   - Magic links.
   - Account export links.
   - Visit invitations.
   - No aviary-state engagement email.

5. Persistence:
   - Primary relational database, such as Postgres, for accounts, birds, canonical state, event log, sessions, invites, notebook, export jobs, deletion state, and audit metadata.
   - A queue or scheduled job system for simulation ticks, email sends, export generation, deletion finalization, and synthetic performance checks.
   - A low-latency cache for public-key configuration, session lookup, static species metadata, and edge-delivered initial snapshots where appropriate. The cache must not become the canonical simulation store.

6. Static asset/CDN edge:
   - Serves HTML, JS chunks, compact visual assets, and generated initial snapshot bootstrap when available.
   - Must support fast first-bird render without waiting for settings/account chunks.

7. Aggregate observability pipeline:
   - Operational metrics only.
   - No per-bird state, personality values, notebook content, account-specific interaction history, or account email.

### Client/Server Split

The client owns:

- Drawing the current snapshot.
- Interpolating between server snapshots.
- Running local idle ornaments that do not affect canonical state, such as leaves and feathers.
- Synthesizing procedural calls from server-provided call descriptors and local motif libraries.
- Gradually rebalancing audio mix during listen-in.
- Rendering captions from call grammar descriptors.
- Detecting visibility, focus, pointer/key activity, and sending presence pings.
- Submitting user interaction events with idempotency keys.
- Managing UI focus, reduced-motion rendering, screen-reader narration, and top-bar affordances.

The server owns:

- Accounts and sessions.
- Bird identity, species assignment, names, personality vectors, mood, per-bird cooldowns, and current canonical state.
- Adoption pacing by aviary age.
- Mood state progression while no clients are connected.
- Drift computation and persistence.
- Event ordering, validation, and deduplication.
- Notebook entry generation and persistence.
- Visit authorization and revocation.
- Export and deletion.

The client never:

- Writes personality vectors.
- Computes canonical drift.
- Decides long-term mood transitions.
- Advances the simulation tick.
- Treats local state as mergeable truth.
- Sends absolute personality, mood, or bird-position writes.

### Render Pipeline Boundary

The simulation state should contain semantic and parametric descriptors, not pixels. A snapshot should tell the client:

- Which birds exist.
- Stable bird IDs, names, species, and public visual variant IDs.
- Current mood label or mood rendering key.
- Perch zone and normalized perch coordinate.
- Current pose/motion descriptor with phase offset.
- Transition descriptors, if a bird is moving or greeting.
- Current call schedule windows and call grammar tokens.
- Current ambient state: local day phase, weather, settled state, scene palette key.
- Notebook unread-count equivalent should not be shown as a badge; the notebook endpoint can provide entries when opened.

The client turns these descriptors into visuals, audio, captions, and accessible narration. This preserves the server's canonical continuity while keeping the rendering system fast and expressive.

## 4. Data Model

### Account

Fields:

- `account_id`: synthetic UUID, primary identifier used everywhere except the encrypted email field.
- `email_ciphertext`: encrypted verified email address.
- `email_fingerprint`: keyed non-reversible lookup token for magic-link lookup and uniqueness checks.
- `created_at`, `updated_at`.
- `email_verified_at`.
- `pending_email_ciphertext`, `pending_email_fingerprint`, `pending_email_requested_at`.
- `deleted_at`, `hard_delete_after`, `restored_at`.
- `privacy_policy_version_accepted`.

Rules:

- Never use email as a database key, log key, queue key, telemetry dimension, partition key, or URL parameter.
- Logs may include `account_id` only where operationally necessary; avoid account-level dimensions in aggregate metrics unless needed for debugging and segregated from analytics.
- Soft-deleted accounts cannot render aviaries except for the recovery page.

### Device Session

Fields:

- `session_id`: UUID.
- `account_id`.
- `session_token_hash`.
- `created_at`, `last_seen_at`, `expires_at`, `revoked_at`.
- `device_label`: derived user-facing label such as browser/platform, sanitized.
- `last_ip_prefix` or coarse geography only if needed for security display; do not put precise IP in app-facing data.

Rules:

- Per-device sessions are revocable.
- Session timeout errors use matter-of-fact voice.
- Session tokens authorize the account and client type, not simulation writes beyond event submission.

### Aviary

Fields:

- `aviary_id`: UUID.
- `account_id`: one-to-one for V1.
- `created_at`.
- `local_timezone`: user's chosen or browser-confirmed timezone.
- `settled_until_reengaged`: boolean or state marker for current settled session state.
- `last_tick_at`.
- `simulation_version`.
- `snapshot_version`: monotonically increasing integer.
- `active_weather_id`, `weather_state`, `weather_started_at`, `weather_ends_at`.
- `next_species_offer_at` or computed from `created_at` and bird count.
- `settings_id`.

Rules:

- One aviary per account in V1.
- The aviary is never public by default.
- Local timezone drives day/night and time-of-day mood inputs.

### Bird

Fields:

- `bird_id`: stable UUID, never replaced.
- `aviary_id`.
- `species_id`.
- `name`.
- `created_at`, `renamed_at`.
- `display_variant_seed`: deterministic seed for visual details.
- `call_signature_seed`: deterministic seed for motif variation.
- `personality_boldness`.
- `personality_social_warmth`.
- `personality_vocal_frequency`.
- `personality_plumage_saturation`.
- `personality_curiosity`.
- `current_mood`.
- `mood_entered_at`.
- `current_perch_zone`: front, middle, back.
- `current_perch_slot`: normalized coordinate or named perch within zone.
- `current_pose_key`.
- `current_motion_phase`.
- `last_offer_at_by_offer_type` or normalized cooldown table.
- `last_listen_in_at`.
- `last_greeted_at`.
- `greeting_tendency_state`: server-derived state for selecting return-greeter without exposing counters.

Rules:

- Personality fields are not returned to ordinary product UI endpoints.
- Renaming changes only the name and rename timestamp.
- Species migration, asset updates, and call grammar updates must preserve `bird_id`, seeds, personality, mood, and notebook references.

### Species

Species metadata can be versioned configuration rather than per-account database rows:

- `species_id`.
- `display_name_internal`.
- `silhouette_key`.
- `pose_set_key`.
- `palette_key`.
- `default_personality_seed_ranges`.
- `call_motif_library_key`.
- `night_activity_profile`.
- `accessibility_descriptor_templates`.

Rules:

- Species rarity is not a V1 feature.
- Starter birds are selected by the system from the pool using balanced variety rules, not a user catalog.

### Interaction Event Log

Fields:

- `event_id`: UUID.
- `aviary_id`.
- `account_id`.
- `client_event_id`: idempotency key generated by the client.
- `event_sequence`: server-assigned monotonic sequence per aviary.
- `event_type`: `presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `settle_undo`, `bird_rename`, `audio_setting_change`, etc.
- `bird_id`: nullable; set for listen-in and per-bird offers/reactions where applicable.
- `payload_json`: bounded schema per event type.
- `client_observed_at`.
- `server_received_at`.
- `processed_at`.
- `processing_tick_id`.
- `source`: host, visitor, system, migration.

Rules:

- Host clients can submit interaction events.
- Visitor clients cannot submit presence or interaction events that affect simulation.
- Event payloads must be schema-validated and size-limited.
- Deduplicate by `account_id`, `aviary_id`, and `client_event_id`.
- The event log is for simulation only. It is not copied into analytics.
- After events are consumed and any required recovery window passes, raw payloads should be compacted or deleted where possible, retaining only per-account simulation summaries needed for continuity.

### Presence Window

Presence can be derived from validated pings, but materializing windows simplifies drift:

- `presence_window_id`.
- `aviary_id`.
- `account_id`.
- `started_at`, `ended_at`.
- `source_session_id`.
- `quality`: valid, interrupted, discarded.
- `seconds_counted`.
- `last_activity_at`.
- `ended_by`: hidden, blur, inactivity_timeout, settle, tab_close_heartbeat_timeout, session_end.

Rules:

- Count presence only when visibility is visible, window focus is true, and recent pointermove or keypress is within the calibrated activity window.
- Lean activity-window calibration toward allowing quiet watching, not twitchy engagement.
- Presence is not exposed as visit counts, streaks, calendars, or user-facing progress.

### Mood and Expression State

Mood is stored per bird. In addition, use a short-lived expression envelope or recent-presence influence per aviary/bird to reconcile two requirements:

- Personality drift never moves downward on neglect.
- A user returning after a long absence should see birds that are quieter and more ambient than during a period of recent attention, without visible distress.

Fields can include:

- `recent_presence_energy`: decays toward neutral baseline after absence, never below baseline.
- `last_host_presence_at`.
- `last_absence_duration_bucket`.
- `current_mood`.
- `mood_intensity`.

Rules:

- This envelope may modulate greeting probability, call frequency, and immediate expressiveness.
- It must not be presented as happiness, hunger, affection, or any user-obligation state.
- It must not reduce persisted personality values.

### Notebook Entry

Fields:

- `entry_id`.
- `aviary_id`.
- `created_at`.
- `local_day_label`.
- `text`.
- `entry_kind`: greeting_order, quiet_morning, weather, offer_reaction, mood_observation, long_absence_return, chorus, perch_pattern.
- `referenced_bird_ids`.
- `source_tick_id`.
- `visible_after`.

Rules:

- Naturalist voice: lowercase, present-tense, specific, no "you", no achievement framing.
- Sparse cadence: roughly one entry every few days for regular visitors, more only for genuinely noteworthy events.
- Read-only forever.
- No entries about user visit frequency, streaks, or engagement.
- No numeric trait values or system event-log language.

### User Settings

Fields:

- `account_id`.
- `audio_enabled`.
- `call_captions_enabled`.
- `reduced_motion_enabled`.
- `screen_reader_narration_enabled`.
- `visit_notifications_enabled`: default false.
- `timezone`.
- `color_contrast_preference` if design supports alternate contrast schemes.

Rules:

- Accessibility settings use matter-of-fact voice in settings UI.
- The aviary surface itself remains naturalist where copy appears.

### Visit Invite and Visit Session

Invite fields:

- `invite_id`.
- `host_account_id`.
- `aviary_id`.
- `visitor_email_ciphertext`.
- `visitor_email_fingerprint`.
- `token_hash`.
- `created_at`, `expires_at`, `revoked_at`, `accepted_at`.
- `last_used_at`.

Visit session fields:

- `visit_session_id`.
- `invite_id`.
- `host_account_id`.
- `aviary_id`.
- `visitor_email_fingerprint`.
- `started_at`, `ended_at`.
- `approx_duration_seconds`.
- `revoked_during_session`.

Rules:

- Invites default nonexistent; visits are off unless a host sends one.
- Visitor links are one-time or bounded-session tokens. If the product needs repeat visits during the 30-day invite window, issue a visitor session after token consumption; do not make the emailed token itself a reusable bearer token.
- Visitors can fetch read-only snapshots and audio/caption descriptors.
- Visitors cannot send presence pings, offers, listen-in, settle, notebook-affecting events, or any event consumed by the host simulation.
- Host visit log is reachable in settings and has no badge.

### Export Job

Fields:

- `export_job_id`.
- `account_id`.
- `requested_at`, `completed_at`, `expires_at`.
- `status`.
- `download_token_hash`.
- `object_storage_key`.

Rules:

- Export contains account settings, birds, current personality vectors, current moods, notebook entries, invites/logs if legally appropriate, and deletion state where relevant.
- Export generation is on demand and emailed as a link to the verified email.
- Export links expire.
- Export must not be advertised as a product feature or surfaced with naturalist language.

## 5. API Surface

All API responses should use structured error codes and matter-of-fact messages for system surfaces. All mutation endpoints must use idempotency keys. All endpoints must enforce synthetic account IDs internally and avoid email in URLs.

### Auth

`POST /api/auth/magic-link`

Request:

- `email`.
- Optional `redirect_path`, restricted to same-origin allowed routes.

Behavior:

- Normalize email for lookup.
- Rate-limit by email fingerprint, IP, and global send volume.
- Create a single-use token expiring in 15 minutes.
- Send matter-of-fact email with sign-in link.
- Response is generic whether or not the email exists.

`POST /api/auth/magic-link/consume`

Request:

- `token`.

Behavior:

- Validate token hash, expiration, and unused status.
- Create account if needed, or sign in existing account.
- Invalidate token immediately.
- Issue per-device session token.
- If new account, create empty aviary shell and direct to adoption flow.

`GET /api/sessions`

- Lists active device sessions for account settings.

`DELETE /api/sessions/{session_id}`

- Revokes a device session.

### Adoption and Bird Naming

`GET /api/aviary/adoption`

- Returns whether the account needs starter adoption or has age-unlocked bird availability.
- For first adoption, returns two server-selected starter bird placeholders with species descriptors sufficient for naming, not a catalog of choices.

`POST /api/aviary/adoption`

Request:

- Starter or age-unlocked adoption token.
- Names for new birds.

Behavior:

- Creates stable bird records.
- Seeds personality, visual variant, and call signatures.
- Writes no gamified adoption counters.

`PATCH /api/birds/{bird_id}/name`

- Renames a bird.
- Validates name length and content.
- Does not affect personality, mood, call signature, or notebook history.

### Snapshot Consumption

`GET /api/aviary/snapshot`

Parameters:

- Optional `since_version`.
- Optional `capabilities`: reduced-motion, captions, audio support.

Response:

- `snapshot_version`.
- `server_time`.
- `local_time_context`: day phase, timezone.
- `aviary_state`: settled, weather, palette key.
- `birds`: public descriptors without personality numbers.
- `call_schedule`: upcoming/current call grammar descriptors.
- `transition_descriptors`: current movement/greeting/offer reaction descriptors.
- `presence_config`: calibrated activity window and ping cadence.
- `settings`: accessibility/audio flags needed for rendering.

Behavior:

- Small payload, kilobytes.
- Cache headers must not leak private state across accounts.
- Use ETag or `snapshot_version` to avoid unnecessary payloads.
- Return matter-of-fact errors for auth/session/sync problems.

For first paint, embed a bootstrap snapshot in the HTML where safe and available. If the snapshot cannot be embedded quickly, render the quiet field immediately and hydrate the first bird as soon as the snapshot arrives. Do not display a spinner.

### Event Ingestion

`POST /api/aviary/events`

Request:

- `client_event_id`.
- `events`: bounded batch of event objects.

Event types:

- `presence_ping`: includes visible/focused/recent-activity booleans and timestamps.
- `listen_in_start`: includes `bird_id`.
- `listen_in_end`: includes `bird_id` and reason.
- `offer`: includes offer type and optional target context, not direct trait deltas.
- `settle`.
- `settle_undo`.
- `audio_capability`.
- `accessibility_setting_change`.

Behavior:

- Server validates whether events are allowed for host session.
- Server stamps sequence and receipt time.
- Server deduplicates idempotently.
- Server stores events for tick consumption.
- Response acknowledges acceptance, not simulation result, unless a low-latency interaction descriptor is safe to return.

Presence validation:

- The client reports the three required signals.
- The server discards pings that are stale, impossible, too frequent, hidden, blurred, or missing recent activity.
- The server should tolerate ordinary clock skew and laptop sleep gaps.

### Offers

Offer events can go through the generic event endpoint, but the client may benefit from a dedicated endpoint:

`POST /api/aviary/offers`

Request:

- `client_event_id`.
- `offer_type`: seed, song_fragment, still_pool.
- Optional `song_fragment_id` from a small server-approved library.

Behavior:

- Checks per-bird and per-offer cooldowns.
- Chooses receiving bird or birds based on mood, curiosity, proximity, and current state.
- Writes offer event.
- Returns immediate render descriptor if available: approach, wait, ignore, call-against, drink, bathe, watch.
- The simulation tick remains responsible for durable mood/drift effects.

### Notebook

`GET /api/notebook`

Parameters:

- Pagination cursor.

Response:

- Entries in reverse chronological or chronological display order, depending on design.
- No edit/delete endpoints.

Rules:

- Notebook opening must not mark engagement achievements.
- Avoid unread badges. If the product needs a subtle indication of new entries, use a restrained affordance that does not read as a notification count.

### Account Settings, Export, Deletion

`GET /api/account`

- Matter-of-fact account/settings data.

`PATCH /api/account/settings`

- Audio, captions, reduced motion, narration, visit notifications, timezone.

`POST /api/account/email-change`

- Starts verification of new email.

`POST /api/account/export`

- Creates export job and emails download link.

`POST /api/account/delete`

- Starts 30-day soft delete.

`POST /api/account/restore`

- Restores within soft-delete window.

### Visits

`POST /api/visits/invites`

Request:

- Visitor email.

Behavior:

- Creates a revocable invite expiring after 30 days.
- Sends visitor email.
- Does not create public profile or discoverability.

`GET /api/visits/invites`

- Lists outstanding/active/revoked/expired invites in account settings.

`DELETE /api/visits/invites/{invite_id}`

- Revokes invite immediately.

`POST /api/visits/consume`

- Consumes visitor token and establishes visitor session.

`GET /api/visits/{visit_session_id}/snapshot`

- Returns read-only host aviary snapshot.
- Does not emit host presence or interaction events.
- If revoked, returns matter-of-fact "visit no longer available" surface.

`GET /api/visits/log`

- Host view of visit log, reachable on demand only.

## 6. Simulation Engine Design

### Tick Lifecycle

Run a simulation tick for each active aviary approximately once per minute. An aviary should continue ticking even when no client is connected, but the implementation can use adaptive scheduling as long as canonical state remains correct when the user returns. For example:

- Hot accounts with recent presence tick every minute.
- Recently inactive accounts tick on a slower schedule plus catch-up computation at next access.
- Long-inactive accounts can use deterministic catch-up over elapsed intervals rather than enqueueing every missed minute.

The important invariant is that the aviary's state at return reflects elapsed time and prior inputs, not a frozen client resume.

Tick steps:

1. Acquire per-aviary lock with timeout.
2. Load aviary, birds, current canonical state, last processed event sequence, and relevant settings.
3. Fetch unprocessed interaction events in sequence.
4. Validate and compact presence pings into presence windows.
5. Aggregate recent host-only signals by bird and by aviary.
6. Update short-lived expression envelopes.
7. Compute personality drift deltas.
8. Apply mood transitions.
9. Choose perch and pose transitions.
10. Update call scheduling descriptors.
11. Possibly schedule or update ambient weather.
12. Generate sparse notebook candidate, if thresholds are met.
13. Check age-based adoption availability.
14. Persist changes transactionally.
15. Mark events processed.
16. Increment snapshot version.
17. Release lock and emit aggregate operational metrics.

No tick should write to aggregate analytics with per-bird details. Tick latency p99 over 5 seconds should page or alarm.

### Personality Ranges and Seeds

Use normalized trait ranges, for example 0.0 to 1.0, but keep the specific numeric representation fully internal. Starter birds should seed in moderate ranges so they feel distinct but not extreme:

- Boldness: enough spread that one bird may prefer the front perch more often.
- Social warmth: enough spread to vary greeting-first tendencies.
- Vocal frequency: enough spread for recognizable call cadence.
- Plumage saturation: visible species palette plus subtle long-term richness.
- Curiosity: enough spread for offer reactions.

Species can influence seed ranges, but individual bird identity should matter more than species stereotype. The same species should not always behave the same.

### Drift Function

Implement drift as a slow low-pass filter over host presence and host interactions. The function should be calibrated to meet:

- Instrument-measurable drift after about one week of regular visits.
- User-visible felt drift after about three weeks.
- No single session producing visible trait movement.
- No downward personality movement from neglect.

Proposed structure:

- Aggregate valid presence seconds into daily buckets per aviary.
- Attribute general presence lightly to all birds currently in the aviary.
- Attribute listen-in more strongly to the focused bird.
- Attribute offer interactions to birds that noticed or reacted, with accepted offers modestly increasing curiosity and approach-related signals modestly increasing boldness.
- Apply a saturating function so repeated high activity in one day cannot accelerate drift beyond the intended band.
- Smooth with an exponential moving average over days.
- Convert smoothed signal into tiny trait deltas at tick or daily rollup time.
- Clamp upward movement by trait-specific maximums and avoid hard visible jumps.

Trait mapping:

- Presence-time: broad expressive lift across social warmth, vocal frequency, plumage saturation, and small boldness/curiosity influence.
- Listen-in: targeted social warmth and vocal frequency for the listened-in bird.
- Offer near bird: small boldness signal.
- Offer investigated or accepted: small curiosity signal and short-term content mood nudge.
- Song fragment response: vocal frequency/social warmth signal for birds that join in.
- Settle: mood quieting and presence-window closure; no direct long-term trait push.

Neglect handling:

- Do not decrease personality values.
- Let recent-presence expression envelope decay to neutral baseline after absence, reducing immediate greeting intensity and call density without making birds distressed or less evolved.
- Absence length can shape return-greeting form: a long absence can produce slower re-orientation or a quieter first call, not guilt or visible suffering.

Calibration tests:

- Simulate one regular user over 7, 21, and 60 days and verify movement magnitude.
- Simulate an always-open background tab and verify zero or near-zero presence drift.
- Simulate heavy clicking/offers and verify cooldowns/saturation prevent visible same-session trait jumps.
- Simulate two-week absence and verify no personality regression, no distress mood, and only ambient quietness.

### Mood Transitions

Mood is fast-timescale and visible through motion, perch, calls, and offer reactions. Use a finite-state probabilistic transition model with:

- Current mood.
- Time of day in the user's timezone.
- Weather.
- Recent host events.
- Bird personality vector.
- Bird-to-bird influence.
- Recent-presence expression envelope.

Mood examples:

- Wary: back perch, scanning, lower approach probability.
- Content: preening, stable perch, soft calls.
- Curious: head tilts, investigates offers, moves between perches.
- Drowsy: low posture, fluffed, reduced calls, evening/night affinity.
- Alert: early morning, higher scan/call readiness.
- Settled: quiet evening mode after settle gesture or night transition.

Transition rules:

- Mood persists across sessions.
- Opening the tab does not reset mood.
- Time-of-day slowly nudges mood rather than snapping it.
- Weather produces short-lived mood nudges.
- A bird's alarm or wary state can influence nearby birds.
- High boldness dampens transition into wary on the same input.
- High social warmth increases response to other birds' calls.
- High vocal frequency increases chorus participation.

The output should be a mood plus rendering descriptors, not a label shown to the user.

### Return-Greeting Algorithm

The return-greeting should be a server-informed, client-rendered transition:

Inputs:

- Absence duration bucket since last valid host presence or visibility.
- Current bird moods.
- Boldness and social warmth.
- Recent greeter history.
- Time of day.
- Current perch and pose.
- Whether the aviary is settled or at night.

Selection:

- Choose one primary greeter.
- Weight toward bold/social birds, but allow variation so the same bird does not always greet.
- Avoid simultaneous greetings.
- If secondary reactions happen, stagger them by small randomized offsets.

Greeting forms:

- Short glance from preening for brief absences.
- Head tilt and soft call for ordinary return.
- Step toward front perch for bold/content birds after longer absence.
- Quiet call from back perch for wary birds.
- Longer call plus another bird's delayed response for long absence, if mood supports it.

No text accompanies this greeting. The client should be able to render it within the first second or two after snapshot availability.

### Perch and Motion State

Perch position is a signal, not user-controlled layout. The simulation chooses front/middle/back zone from:

- Mood.
- Boldness.
- Recent host presence.
- Offer context.
- Weather and time of day.
- Bird-to-bird spacing.

The server should produce stable perch choices that avoid jitter. Use transition hysteresis:

- Do not move a bird between perch zones too often.
- When a move is selected, emit a transition descriptor with start/end zone, duration, and motion style.
- Keep all birds visible and avoid overlapping silhouettes beyond designed groupings.

Client motion:

- Normal mode uses idle loops and procedural micro-motion.
- Reduced-motion mode uses slow pose cross-fades.
- Hidden tabs stop rendering, but state is refreshed on visibility return.

### Offer Handling

Offer flow:

1. User opens offer affordance in top bar.
2. User chooses seed, song fragment, or still pool.
3. Client submits offer event.
4. Server validates cooldown and current state.
5. Server chooses reaction candidates.
6. Client renders immediate reaction descriptor.
7. Tick applies mood and drift consequences.

Cooldowns:

- Per bird and offer type, a few minutes.
- Cooldowns are functional, never messaged as punishment.
- If no bird is ready to react, the offer can sit quietly or receive a naturalist observation, not an error-like rejection.

Offer reaction mapping:

- Seed: curious/content bird approaches; wary bird waits then may approach; drowsy bird may not move.
- Song fragment: birds may join, quiet, or call against depending on vocal frequency and mood.
- Still pool: birds may drink, bathe, watch, or ignore.

The user should not be able to spam offers to force drift. Saturation and cooldowns protect the engine.

### Field Notebook Generation

Notebook entries are sparse and specific. Implement generation as a rules-first system with templated naturalist prose and strict filters, not as a raw event log.

Candidate triggers:

- A bird greets first after not greeting first recently.
- Long quiet stretch with distinctive mood/perch.
- Rare weather plus bird reaction.
- Offer reaction that differs from recent pattern.
- A chorus event involving two or more birds.
- A long absence return where the aviary is notably quiet or re-orienting.
- Age-based arrival of a new bird, if framed as aviary observation rather than achievement.

Filters:

- Minimum days or meaningful-event interval between entries.
- No more than one entry per day except exceptional moments.
- Avoid repeated template shapes.
- Avoid user-behavior claims: no "you visited", no streaks, no frequency observations about the user.
- No numeric values or system terms.

Quality guardrails:

- Maintain a small style test suite that rejects exclamation points, achievement words, direct "you" framing, uppercase producty headings, trait names with numbers, and generic event-log phrasing.
- Review notebook templates with design/content before launch.
- Store final entry text, not only source facts, so the notebook remains stable over time.

### Age-Based Bird Addition

New birds become available by aviary age only. Implement a schedule such as:

- Starter: two birds at account creation.
- Third bird: after a few months.
- Additional birds: increasingly spaced intervals, with maximum seven.

The exact schedule can be configuration, but it must not depend on visit count, offers, listen-in totals, payments, or engagement. The UI should present the arrival as part of the aviary's life, not a reward.

## 7. Sync Model and Correctness

### Canonical State

The authoritative state is in the server database. Clients fetch snapshots and submit events. Multi-device sync is achieved by all devices reading the same canonical state, not by syncing clients to each other.

Invariants:

- One canonical personality vector per bird.
- One server tick writer per aviary at a time.
- Event log order is server-assigned.
- Ticks are idempotent or protected by transaction boundaries.
- Client retries cannot duplicate events.
- Stale clients cannot overwrite state.

### Event Ordering

Use `event_sequence` per aviary. On ingestion:

- Deduplicate by client idempotency key.
- Assign sequence in a database transaction.
- Store raw event with receipt time.
- Avoid trusting client time for ordering, but keep it for duration interpretation within safe bounds.

On tick:

- Process events greater than `last_processed_event_sequence`.
- Persist `last_processed_event_sequence` with state changes.
- Mark events processed with tick ID.

### Concurrency

Use row-level locks or advisory locks per `aviary_id` during tick. If two workers attempt the same aviary:

- One wins lock.
- Other exits or retries later.
- No partial tick writes.

For interaction ingestion while a tick is running:

- New events can append concurrently.
- The tick processes up to the sequence it selected at start.
- Later events wait for next tick.

### Client Snapshot Refresh

Clients should pull snapshots:

- On navigation/open.
- On visibility change to visible.
- After long render-frame gaps or detected sleep/wake.
- At a low-frequency keepalive while visible.
- After mutation acknowledgement when the local descriptor may need server confirmation.

The client can interpolate but must reconcile to server snapshots. If local rendering diverges, prefer smooth correction over visible snapping.

### Conflict Surfaces

Most conflicts should be architecturally impossible. User-visible surfaces are limited to:

- Expired or used magic links.
- Session timeout/revocation.
- Snapshot load failure.
- Revoked/expired visit.
- Unsupported browser.
- Temporary server outage.

All use matter-of-fact voice. No naturalist jokes or charm in error recovery.

## 8. Frontend Rendering Pipeline

### Technology Choice

Use TypeScript end to end. For the aviary scene, choose a rendering path that can meet 60fps idle motion and reduced-motion cross-fades:

- Canvas/WebGL scene renderer with a small retained scene graph.
- React or equivalent only around chrome and panels, not per-frame bird animation.
- Web Workers for expensive non-DOM calculations if profiling shows main-thread pressure.
- CSS transitions only for simple chrome opacity, panels, and focus rings.

The goal is to keep the browser main thread predictable, avoid DOM-heavy animation for birds, and preserve accessibility overlays outside the rendering hot path.

### Scene Composition

The scene consists of:

- Background sky and foliage.
- Three depth-aware perch zones: front, middle, back.
- Bird sprites/shapes or compact vector/bitmap assets.
- Foreground occasional branch/leaf elements.
- Ambient weather layer.
- Caption overlay layer.
- Focus indicator layer.

No controls, hover tooltips, badges, labels, or inline icons live inside the scene. Interactive chrome sits in the top bar above the scene.

### First Frame and Loading

Target:

- First bird visible under 500ms on mid-tier mobile over 4G.
- No spinner.
- No fade-from-static.
- No entry sequence once the aviary exists.

Implementation:

- Server-render minimal HTML with critical CSS and an embedded or quickly fetched bootstrap snapshot.
- Inline only the code required to draw quiet field plus first birds.
- Defer account settings, notebook panel, visit management, export, deletion, and other non-critical chunks.
- If snapshot is delayed, draw quiet field immediately with subtle motion cues and no machine-like loading indicator.
- Once snapshot arrives, draw birds mid-action using current pose phase.

For brand-new accounts after naming starter birds, the empty-aviary state uses the quiet field, then first birds can enter softly. After initial adoption, the user should never see an empty aviary again.

### Responsive Layout

Build a layout solver that maps canonical perch coordinates into viewport coordinates:

- Preserve all birds in frame on narrow phone widths.
- Expand spacing on wide desktop without making birds feel lost.
- Maintain an aspect ratio range that supports top bar plus scene.
- Never crop birds.
- Avoid panning, scrolling, or zooming.
- Keep top bar outside scene proper.

Use automated viewport tests for common phone, tablet, laptop, and wide desktop sizes.

### Motion System

Normal motion:

- Birds continuously idle: preen, scan, head tilt, body shuffle, call posture.
- Motion is keyed by mood and personality.
- Ambient leaves/feathers drift client-side at slow random intervals.
- Day/night palette transitions are gradual.
- Weather effects are quiet.

Reduced motion:

- Use still pose cross-fades.
- Replace flight paths with cross-fades between perches.
- Remove leaf drift.
- Slow ambient color shifts.
- Keep calls, captions, mood changes, notebook, and drift intact.

Hidden/background tabs:

- Stop rendering and pause local WebAudio as required by browser policy.
- Stop counting presence.
- Refresh snapshot on visibility return.

### Top Bar

Top bar contents:

- Account/settings.
- Accessibility settings.
- Field notebook.
- Offer affordance.

Rules:

- Fade nearly transparent after a few seconds of cursor stillness.
- Return on pointer movement, keyboard activity, focus, or touch interaction.
- No badges for streaks, visits, or engagement.
- Any new top-bar item requires product review against "notice, never announce."

### Interaction UX

Listen-in:

- Click/tap or keyboard focus on bird.
- Focused bird's calls ramp up gradually.
- Other birds ramp down but never silent.
- Disengage on same bird, another bird, empty scene, focus move, or Escape.
- Visual focus should be subtle and accessible, not a UI label over the bird.

Offer:

- Reached from top bar.
- Small panel with seed, song fragment, still pool.
- No direct click-on-bird feeding model.
- Keyboard navigable.
- Cooldown and non-reaction handled softly, without punitive copy.

Settle:

- Reached from top bar.
- Slow evening lighting shift.
- Calls quiet.
- Five-second undo via any click in aviary.
- Closing tab without settle is equally valid.

Field notebook:

- Top-bar notebook icon opens a panel or route.
- Entries are read-only and scrollable indefinitely.
- Typography should feel like field notes but remain readable and accessible.
- No edit/delete/comment affordance.

## 9. Audio Pipeline

### Procedural Call Engine

Build a WebAudio-based procedural synthesis engine:

- Species motif libraries define contour tokens, intervals, envelope shapes, timbre filters, and rhythm cells.
- Bird call signatures are seeded from species plus per-bird `call_signature_seed`.
- Personality and mood modulate timing, frequency, envelope, and chorus participation.
- Each call instance is generated at runtime with small variations.
- The same grammar descriptor can produce matching caption text.

Use AudioWorklet if needed for stable scheduling. Keep memory bounded by reusing buffers/nodes where possible and avoiding per-call leaks.

### Call Scheduling

The server snapshot provides call windows and grammar descriptors:

- Bird ID.
- Motif family.
- Mood modulation.
- Start time relative to snapshot.
- Expected duration.
- Caption tokens.
- Chorus relationship, if any.

The client schedules audio locally against `AudioContext.currentTime`, adjusting for drift when a new snapshot arrives. Calls can be omitted or softened if the browser blocks audio or the user has disabled it.

### Listen-In Mix

Use mix buses:

- Master ambient bus.
- Per-bird bus.
- Focused bird target gain.
- Non-focused ambient target gain.

Ramps:

- Engage over a slow, perceptible-but-gentle interval.
- Disengage over the same interval.
- Never hard-cut.
- Never mute non-focused birds fully.

### Browser Audio Policy

The PRD wants calls audible as the aviary is already in motion, but browsers may block audio before user activation. Plan for this explicitly:

- During onboarding, the naming/adoption interaction can unlock audio for many browsers.
- Persist user audio preference and resume audio context on subsequent gestures where required.
- If autoplay is blocked, render the aviary normally in graceful silence and make captions available or enabled according to settings/fallback rules.
- Do not show a "Welcome back" or loud audio-permission banner over the aviary.
- A matter-of-fact audio control in settings can explain when the browser requires interaction to play sound.

This is a launch risk because browser policy may prevent perfect first-frame audible calls. The mitigation is to make silence graceful, captions high-quality, and audio start unobtrusively on allowed interaction.

### Captions

Caption text is generated from call grammar tokens, not hard-coded per bird:

- "a soft three-note rise"
- "a low trill, paused, low trill again"
- "a single sharp call from the back perch"

Caption behavior:

- Optional in accessibility settings.
- Default on when WebAudio unavailable.
- Appears near calling bird, fading in/out with call.
- Must pass contrast requirements.
- Must not overlap incoherently with birds/top bar.
- Should respect reduced-motion settings for fade timing.

## 10. Accessibility Plan

Accessibility ships in V1, not after launch.

### Screen-Reader Narration

Implement a narration generator over the same snapshot state:

- Naturalist prose.
- Lowercase, present-tense, specific.
- Slow cadence: roughly every 30 to 60 seconds at idle.
- Priority bumps for return-greeting, offer reaction, settle, and meaningful user-initiated changes.
- `aria-live` region should be polite by default, with careful queue control to avoid flooding.

Do not expose:

- Perch numbers.
- Mood labels as raw states.
- Personality values.
- Event-log phrasing.

Example shape:

- "pip is on the front rail, calling softly. wren sits further back, fluffed against the morning light."

### Keyboard Navigation

Required keyboard flow:

- Tab through top bar controls.
- Tab into aviary scene.
- Focus first bird.
- Arrow keys move between birds.
- Enter toggles listen-in on focused bird.
- Escape exits listen-in.
- Offer panel opens from top bar and is fully keyboard navigable.
- Settle is reachable from top bar.
- Notebook and settings panels trap focus while open and restore focus on close.

Focus indicators:

- Visible against bright, dim, rainy, and night palettes.
- Soft enough to fit the surface but high-contrast enough to be usable.
- Tested with reduced-motion and captions enabled.

### Reduced Motion

Implement as a first-class renderer mode:

- Auto-enable from `prefers-reduced-motion`.
- User can override in accessibility settings.
- Cross-fade between poses.
- Remove ambient leaf/feather drift.
- Keep day/night color changes, slowed.
- Keep audio/captions/narration.
- Ensure reduced-motion users still see current mood and bird identity.

### Contrast and Text

All user-copy text must pass WCAG AA:

- Top bar icons/labels/tooltips where present.
- Settings.
- Account/auth.
- Errors.
- Captions.
- Notebook text.
- Narration if displayed visually.

The scene itself should not contain user-copy labels except captions/focus surfaces.

### Accessibility Testing

Test matrix:

- VoiceOver Safari macOS/iOS.
- NVDA or JAWS on Windows Chrome/Firefox.
- Keyboard-only desktop.
- Touch-only phone.
- Reduced-motion OS preference.
- Audio disabled.
- WebAudio blocked.
- High contrast or increased contrast mode where available.

Acceptance:

- A screen-reader user can sign in, adopt/name birds, experience the aviary, listen in, offer, settle, open notebook, manage settings, export/delete account, and manage visits.
- No interaction requires pointer-only access.
- Reduced-motion rendering is visually coherent and not a broken static fallback.

## 11. Privacy and Security

### PII Boundary

Email:

- Stored encrypted only on account/invite records.
- Indexed through keyed fingerprints.
- Never used as primary key, telemetry dimension, queue partition key, log identifier, or URL path parameter.

Account ID:

- Synthetic UUID.
- Used in internal references.
- Still treated as sensitive operational data.

Logs:

- Scrub email, magic-link tokens, invite tokens, event payloads, notebook text, bird names where possible.
- Use structured error codes.

### Interaction Data Boundary

Per-bird and per-account interaction state:

- Stored only for that user's simulation.
- Not copied to analytics.
- Not used for model training.
- Not used for recommendations.
- Not used for population dashboards.
- Not shared with third parties.

Aggregate operational telemetry allowed:

- Request counts.
- Latencies.
- Error rates.
- Anonymous session-duration histograms without account dimension.
- First-bird render timing.
- Render frame timing.
- Audio-context error counts.
- Simulation tick latency.

Disallowed telemetry:

- Average drift by trait.
- Most popular bird species by account interaction.
- Offer acceptance by bird personality.
- Per-account bird behavior analytics.
- Anything that requires reading the simulation database into analytics.

### Auth Security

Magic links:

- 15-minute expiration.
- Single use.
- Token hash stored server-side.
- Rate-limited by email fingerprint and IP.
- Generic request response.

Sessions:

- Secure, HttpOnly cookies or equivalent secure token storage.
- CSRF protection for cookie-based sessions.
- Session revocation.
- Sensible expiration and refresh policy.

Visit links:

- Token hash stored server-side.
- Expire after 30 days if unused.
- Revocable immediately.
- Visitor sessions are read-only and scoped to one aviary.

### Deletion

Soft deletion:

- Account immediately hidden/disabled.
- Recovery available for 30 days.

Hard deletion:

- Delete birds, vectors, moods, notebook, event logs, invites, sessions, export jobs, and telemetry rows tied to account where applicable.
- Ensure queued ticks/email/export jobs no-op after hard deletion.
- Record only minimal deletion audit if legally required, without bird data or interaction history.

## 12. Performance and Observability

### Budgets

Hard V1 budgets:

- Initial JS bundle under 2MB gzipped.
- First bird visible under 500ms on mid-tier mobile over 4G.
- 60fps idle motion on a five-year-old mid-range laptop.
- No client memory growth over 30 minutes.
- Simulation tick p99 alarm at 5 seconds.

### Client Performance Strategy

- Split initial scene renderer from account/settings/notebook/visit chunks.
- Keep critical CSS small.
- Use compact visual assets: SVG, compressed bitmaps, or procedural shapes.
- Avoid recorded audio assets.
- Pool animation objects and audio nodes.
- Avoid per-frame React state updates for scene motion.
- Batch canvas/WebGL draws.
- Use requestAnimationFrame only when visible.
- Pause rendering in hidden tabs.
- Use performance marks for navigation, quiet-field draw, first bird draw, first snapshot, first call scheduled, first stable frame.

### Server Performance Strategy

- Keep snapshots small.
- Use database indexes by `aviary_id`, `event_sequence`, `last_tick_at`, `account_id`, and invite token hash/fingerprint.
- Tick in bounded batches.
- Use lock timeouts and retry queues.
- Keep notebook generation deterministic/rules-based enough for predictable latency.
- Use catch-up simulation for long-inactive accounts rather than minute-by-minute replay where safe.

### Observability

Aggregate metrics:

- API request rate, latency, error code.
- Magic-link send/consume success rates.
- Snapshot size and latency.
- Event ingestion accepted/rejected counts by event type, without account/bird dimensions.
- Tick duration, backlog, lock contention, failures.
- First-bird render timings.
- Client frame timing distribution.
- Audio-context error counts.
- WebAudio unsupported/blocked counts.
- Memory soak test results.
- Accessibility smoke test pass/fail in CI.

Synthetic monitoring:

- Automated browsers from common geographies.
- Test signed-in seeded accounts with non-sensitive synthetic data.
- Measure first-bird, snapshot, frame timing, tick health.
- Run normal and reduced-motion modes.

Do not instrument engagement metrics that invite product drift: streaks, visit frequency by account, leaderboards, most active aviaries, or bird popularity.

## 13. Rollout Plan

### Phase A: Foundations

Deliverables:

- Repository structure and CI.
- TypeScript client shell.
- API service skeleton.
- Database schema and migrations.
- Synthetic account ID and encrypted email storage.
- Magic-link auth.
- Session revocation.
- Basic aviary/bird records.
- Event log with idempotency.
- Simulation worker lock/tick scaffold.
- Privacy-safe logging baseline.

Exit criteria:

- New account can sign in and create canonical aviary records.
- No email appears in logs outside explicit mail provider boundary.
- Tick can process a no-op aviary safely.

### Phase B: Simulation Core

Deliverables:

- Species config.
- Starter bird selection and naming.
- Personality vector persistence.
- Mood state machine.
- Presence validation and window materialization.
- Drift low-pass implementation.
- Return-greeting selection descriptors.
- Perch/motion descriptors.
- Offer cooldowns and reaction descriptors.
- Notebook candidate generator.
- Age-based bird availability config.

Exit criteria:

- Simulation tests prove no client personality writes.
- Calibration harness shows one-week measurable and three-week visible target bands.
- Background tab/open laptop simulation does not inflate presence.
- Two-week absence produces ambient quietness without negative personality drift.

### Phase C: Aviary Rendering

Deliverables:

- Single horizontal responsive scene.
- Three perch zones.
- Bird pose/motion renderer.
- Day/night palette.
- Ambient weather.
- Ambient leaves/feathers.
- Top bar and fade.
- First-frame quiet field and bootstrap snapshot.
- Listen-in visual focus.
- Offer and settle visual flows.
- Notebook panel.

Exit criteria:

- First bird visible under budget in lab tests.
- Birds never crop across supported viewports.
- No spinner or entry sequence appears after established aviary load.
- Scene remains 60fps in 30-minute idle test on target laptop.

### Phase D: Audio and Captions

Deliverables:

- WebAudio call engine.
- Species motif libraries.
- Per-bird signature seeding.
- Call scheduling from snapshot.
- Chorus mixing.
- Listen-in gain ramps.
- Caption generation from grammar.
- WebAudio blocked/unavailable fallback.

Exit criteria:

- Calls are recognizably per bird across mood variations.
- Repeated calls do not sound looped.
- Listen-in never hard-cuts or silences other birds.
- Captions match generated calls.
- Memory does not grow during 30-minute audio soak.

### Phase E: Accessibility and System Surfaces

Deliverables:

- Screen-reader narration.
- Keyboard navigation.
- Reduced-motion renderer.
- WCAG AA contrast validation.
- Accessibility settings.
- Account export.
- Account deletion/restore/hard-delete job.
- Matter-of-fact error surfaces.
- Unsupported-browser surface.

Exit criteria:

- Screen-reader and keyboard users can complete core flows.
- Reduced-motion mode feels intentionally designed.
- Settings/errors consistently use matter-of-fact voice.
- Export and deletion work without leaking PII or leaving simulation data behind.

### Phase F: Visits

Deliverables:

- Host invite creation.
- Visitor email and token consumption.
- Read-only visitor snapshot.
- Visit revocation.
- Visit log.
- Optional visit notification toggle, default off.

Exit criteria:

- Visitor cannot trigger presence, drift, offers, listen-in, settle, notebook entries, or host notifications by default.
- Revocation terminates visitor access on next snapshot pull.
- No profile/discovery/follow/comment surfaces exist.

### Phase G: Beta Hardening

Deliverables:

- Synthetic monitoring.
- Aggregate RUM.
- Load tests for tick backlog and snapshot API.
- Privacy review.
- Accessibility audit.
- Content/voice QA for notebook/narration/captions.
- Browser support matrix.
- Incident runbooks.

Exit criteria:

- Performance budgets met.
- Tick p99 below alarm threshold with headroom.
- No forbidden telemetry detected.
- Product review signs off on no gamification, no announcement surfaces, and no social creep.

### Launch and Ramp

Launch in cohorts:

1. Internal dogfood with synthetic and employee test accounts.
2. Small private beta with two birds only.
3. Broader beta with age-based third-bird availability enabled only for accounts old enough to exercise it.
4. General V1 launch with bird cap at seven but age schedule naturally limiting near-term count.

Ramp controls:

- Feature flags for visits, notebook generation, weather, age-based new bird offers, and audio captions.
- Server config for tick cadence and drift coefficients.
- Kill switches for WebAudio scheduling, weather, and visit invitations.
- No feature flag for gamification or engagement surfaces.

Day-one instrumentation:

- Performance and reliability metrics.
- Tick health.
- Auth and email delivery health.
- Aggregate browser support/fallback counts.
- Accessibility setting usage as aggregate counts only.

Do not add engagement dashboards that rank accounts or birds.

## 14. Testing Strategy

### Unit Tests

- Presence validation truth table.
- Event schema validation and idempotency.
- Magic-link expiration and single-use behavior.
- Session revocation.
- Drift monotonicity.
- Drift saturation under heavy interactions.
- Mood transition probabilities under controlled seeds.
- Offer cooldowns.
- Notebook template guardrails.
- Visit token expiration/revocation.
- Export inclusion/exclusion.
- Deletion cascade.

### Integration Tests

- Multi-device same-account snapshot consistency.
- Concurrent event ingestion while tick runs.
- Client retry does not duplicate event.
- Stale client cannot overwrite personality.
- Long laptop sleep then visibility return pulls fresh snapshot.
- Visitor snapshot cannot submit host events.
- Soft-deleted account cannot render aviary.
- Hard deletion removes simulation data.

### Simulation Calibration Tests

Create deterministic harnesses for:

- Regular five-minute daily visits for 7, 21, and 60 days.
- Short frequent visits.
- Long quiet watching with minimal pointer movement inside valid activity window.
- Background tab for two days.
- Heavy offer spam.
- Long absence then return.
- Two devices sending overlapping presence/listen-in events.

Assertions:

- Measurable after one week.
- Visible descriptor changes only after longer time.
- No negative personality drift.
- No presence inflation from hidden/background tabs.
- No single-session trait jumps.

### Frontend Tests

- Viewport coverage with screenshot/pixel assertions.
- No bird cropped or unreadable captions.
- Top bar fade and return.
- Keyboard navigation order.
- Listen-in engage/disengage.
- Offer panel and settle undo.
- Reduced-motion visual mode.
- First-frame no spinner.

### Audio Tests

- Grammar produces varied call instances.
- Same bird remains recognizable across variations.
- Mix ramps meet duration and gain targets.
- Non-focused birds remain audible.
- Captions align with generated grammar.
- WebAudio failure enables captions/default silence.
- 30-minute memory soak.

### Accessibility Tests

- Automated axe-style checks for chrome/panels.
- Manual screen-reader passes.
- Keyboard-only pass.
- Reduced-motion pass.
- Contrast checks across day/night/weather palettes.
- Captions with multiple birds calling.

### Privacy Tests

- Log scrubbing test: email, tokens, bird names, notebook text, and event payloads do not appear in logs.
- Telemetry schema test: no per-bird/per-account simulation fields in analytics events.
- Data export test: contains required account snapshot and no unrelated operational data.
- Deletion test: verifies all account simulation records removed after hard-delete.

## 15. Key Product Decisions to Lock Early

1. Rendering technology: choose Canvas/WebGL stack and asset format before building bird assets.
2. Trait ranges and drift coefficient config shape.
3. Presence activity-window calibration.
4. Mood enum and transition model.
5. Species pool and motif library structure.
6. Snapshot schema versioning.
7. Notebook/narration/caption template style guide and automated lint rules.
8. Browser audio policy UX, especially first-ever audible calls.
9. Initial bird age-unlock schedule.
10. Data retention period for raw consumed interaction events.

These decisions should be documented in engineering design docs, but the product boundaries above should remain unchanged.

## 16. Risks and Mitigations

### Drift Calibration Feels Wrong

Risk:

- Too fast: the product becomes stat-management by feel.
- Too slow: user attention feels irrelevant.
- Neglect accidentally feels punitive.

Mitigation:

- Build deterministic calibration harness before UI polish.
- Keep drift coefficients server-configurable.
- Separate personality from recent expression envelope.
- Test long absence explicitly.
- Review visible descriptor changes after simulated weeks with design/content.

### Sync Correctness Corrupts Bird Identity

Risk:

- Duplicate ticks, stale clients, or client-authored state silently lose drift.

Mitigation:

- Server-only personality writes.
- Per-aviary locks.
- Append-only event sequence.
- Idempotency keys.
- Integration tests for overlapping devices.
- Migration tests preserving `bird_id` and personality.

### Audio Sounds Canned or Browser Blocks It

Risk:

- Procedural synthesis is not rich enough, or browser autoplay prevents the intended audible first frame.

Mitigation:

- Prototype call grammar early.
- Include audio designer/engineer in species motif work.
- Test recognizability up to seven birds.
- Treat browser audio activation as a known constraint.
- Make captions and graceful silence high-quality.

### Performance Breaks the "Already Running" Conceit

Risk:

- The user sees a spinner, late first bird, or frame drops.

Mitigation:

- Initial scene chunk budget.
- Bootstrap snapshot.
- Quiet field fallback.
- Canvas/WebGL renderer.
- Automated first-bird and 30-minute soak tests in CI.
- Defer non-critical UI chunks.

### Accessibility Becomes a Flat Fallback

Risk:

- Screen-reader and reduced-motion experiences expose raw state instead of the aviary's charm.

Mitigation:

- Build narration and reduced-motion in parallel with visual scene.
- Content QA for narration/captions.
- Manual assistive tech testing.
- Treat reduced-motion renderer as a supported mode with its own acceptance criteria.

### Privacy Boundary Erodes Through "Helpful" Analytics

Risk:

- Per-bird events leak into dashboards, logs, or training sets.

Mitigation:

- Separate simulation DB from analytics.
- Telemetry schema allowlist.
- Log scrubbing tests.
- Privacy review for every new metric.
- No aggregate dashboards over bird personality, offer behavior, or account-specific interaction history.

### Social Feature Expands Into a Network

Risk:

- Visits invite profiles, badges, friend lists, comments, or notifications.

Mitigation:

- Keep visit code path read-only.
- No public user pages.
- Host notifications off by default and buried in settings.
- Product review for any social-surface change.
- Do not compute leaderboard-like stats.

### Voice Surfaces Become Generic

Risk:

- Notebook, narration, captions, and offers read like logs or achievements.

Mitigation:

- Shared content templates.
- Automated forbidden-phrase lint.
- Review generated entries from simulation harness.
- Separate naturalist and matter-of-fact copy systems.

### Scope Creep Around Customization

Risk:

- Scene customization, bird catalogs, and placement controls appear because they are common virtual-pet affordances.

Mitigation:

- Enforce starter system-selection.
- No bird placement endpoints.
- No scene customization model in schema.
- Treat perch as simulation output only.

## 17. Engineering Guardrails

Make these guardrails part of code review and launch readiness:

- No endpoint that writes personality except simulation worker internals.
- No client payload field named like a trait delta or absolute trait value.
- No UI component containing "streak", "score", "level", "achievement", "badge", "XP", "leaderboard", or "welcome back".
- No analytics event containing bird ID plus user/account dimension outside strictly operational debugging with controls.
- No email address in logs.
- No top-bar notification badges except explicit account/security states, and those must use matter-of-fact settings surfaces.
- No visitor event path into host simulation.
- No recorded call assets.
- No spinner in the aviary scene load path.
- No reduced-motion static fallback that removes mood/call/narration.
- No visible personality numbers outside explicit account export file.

## 18. Suggested Team Workstreams

1. Simulation and data integrity:
   - Schema, event log, tick, drift, mood, notebook facts, calibration.

2. Rendering and interaction:
   - Scene renderer, responsive layout, motion descriptors, top bar, listen-in, offer, settle, notebook UI.

3. Audio:
   - WebAudio engine, motifs, mixing, captions, memory/performance.

4. Accounts/privacy:
   - Magic links, sessions, synthetic ID, export, deletion, logging, telemetry boundaries.

5. Accessibility:
   - Narration, reduced-motion, keyboard, contrast, assistive tech QA.

6. Visits:
   - Invite tokens, visitor sessions, read-only snapshots, revocation, visit log.

7. Performance/observability:
   - Bundle budgets, synthetic monitoring, RUM allowlist, soak tests, runbooks.

Workstreams should integrate early through the snapshot schema and event log. The snapshot contract is the main seam between simulation, rendering, audio, and accessibility; stabilize it before deep polish.

## 19. Definition of Done for V1

V1 is ready only when:

- A new user can sign in, name two system-selected starter birds, and see the aviary with birds already in motion.
- Returning after short and long absences produces varied bird greetings without text announcements.
- Valid presence changes drift slowly; hidden/background tabs do not.
- A bird's personality persists across devices and sessions.
- Listen-in, offer, settle, notebook, account settings, accessibility settings, export, deletion, and visits all work.
- Visitors cannot affect host birds.
- Screen-reader, reduced-motion, captions, and keyboard paths are complete.
- Initial bundle, first-bird, frame-rate, memory, and tick-latency budgets are met.
- Operational telemetry is aggregate-only and privacy-reviewed.
- No gamification, Tamagotchi, notification, public social, or native-app surfaces have slipped in.

The finished product should feel like a small living window that continues without the user, notices them when they arrive, and never turns their attention into a score.
