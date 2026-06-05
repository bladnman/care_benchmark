# Pocket Aviary V1 Implementation Plan

## 1. Product Interpretation And Scope

Pocket Aviary v1 is a calm, browser-only, single-user virtual aviary where the core value is felt aliveness over time. The product should behave less like a game and more like a small place that continues without the viewer. The implementation must protect three invariants above ordinary feature convenience:

- The birds feel alive through server-side continuity, procedural variation, mood-shaped motion, and audio that never repeats as a loop.
- The user is noticed by the aviary, never announced by the system. The bird greeting is the welcome surface.
- Presence is real interaction, but absence is never punished.

V1 includes:

- Modern browser web app only.
- Email magic-link accounts.
- One account, one canonical aviary.
- Two starter birds per new aviary, growing over age toward a hard cap of seven.
- Server-side simulation tick.
- Hidden per-bird personality vectors with slow monotonic drift.
- Daily-ish mood state and mood-shaped idle behavior.
- Procedural client-side calls, listen-in mixing, and call captions.
- Offer interactions for seed, song fragment, and still pool.
- Settle gesture as an optional session-end affordance.
- Field notebook with sparse naturalist observations.
- Multi-device sync through one canonical server state.
- Read-only visit invitations, off by default and revocable.
- Screen-reader narration, reduced-motion mode, keyboard access, captions, and WCAG AA copy contrast.
- Aggregate operational telemetry that excludes per-bird state and per-account interaction history.
- Account export and soft-then-hard deletion.

V1 explicitly excludes:

- Native iOS or Android apps.
- Any game goals, scores, achievements, streaks, badges, XP, levels, rankings, or counters.
- Any Tamagotchi-style decay, death, hunger, distress, or obligation mechanic.
- Public discovery, profiles, follows, comments, chat, shared aviaries, co-presence, leaderboards, or comparison surfaces.
- User-controlled bird placement, scene customization, multiple aviaries, payments, or configurable species catalogs.
- Push notifications or product emails about aviary activity.
- Recorded call loops or recorded-audio fallback.
- Client ownership of personality state.

Any future request that conflicts with these exclusions should be treated as a product change requiring explicit re-approval, not as backlog.

## 2. System Architecture

Use a TypeScript-first web architecture with clear ownership boundaries:

- Browser client: renders the aviary, runs interpolation, synthesizes audio, captures presence signals, submits interaction events, and displays account/settings/notebook/visit surfaces.
- API service: owns authentication, account settings, state snapshots, event ingestion, notebook reads, visit invites, exports, deletion, and accessibility settings.
- Simulation service: the only writer of canonical aviary state, bird personality vectors, mood, notebook generation decisions, and derived bird state.
- Scheduler/worker system: enqueues and runs per-aviary simulation ticks at roughly one-minute cadence.
- Persistence layer: durable relational database for accounts, aviaries, birds, personality vectors, events, notebook entries, visits, sessions, export jobs, and deletion state.
- Realtime-lite delivery: clients pull small snapshots on lifecycle events and low-frequency keepalive; optional server-sent events can be added only for snapshot invalidation, not as a second state source.
- Aggregate observability pipeline: operational metrics only; it must not read per-bird simulation records for product analytics.

Recommended implementation baseline:

- React with TypeScript for the client.
- Canvas or WebGL-backed scene renderer with an SVG/DOM fallback only for non-scene UI. Use a single renderer abstraction so normal and reduced-motion modes share state interpretation.
- WebAudio for procedural calls and mixing.
- Node/TypeScript or another strongly typed backend runtime for API and simulation code. Keep the simulation package pure enough to run deterministically in tests.
- PostgreSQL for durable state, using transactions and row-level locks or advisory locks for per-aviary tick serialization.
- A lightweight queue or scheduled worker for tick execution and account export jobs.
- Object storage for short-lived export files, if download links are not generated directly from the API.

The client never computes canonical personality, mood, drift, bird adoption timing, notebook entries, or visit authorization. It renders snapshots and submits events. This is the primary architectural boundary.

## 3. Service And Module Boundaries

Organize the product around domain modules rather than generic layers:

- `auth`: magic links, sessions, token revocation, email changes.
- `accounts`: account settings, export, deletion, privacy surfaces.
- `aviaries`: canonical aviary records and snapshot assembly.
- `birds`: species pool, bird identity, names, personality vectors, moods, adoption scheduling.
- `events`: append-only interaction events, idempotency, validation, consumption state.
- `presence`: client presence sampling and server presence-window aggregation.
- `simulation`: tick runner, drift, mood transitions, ambient events, return-greeting candidates, notebook generation.
- `notebook`: sparse entry persistence, prose templates, read API.
- `visits`: invite lifecycle, read-only visit sessions, revocation, visit log.
- `renderer`: scene composition, animation state, reduced-motion renderer, focus geometry.
- `audio`: procedural call grammar, synthesizer, mixer, listen-in ramps, captions.
- `accessibility`: narration, captions, keyboard behavior, settings, reduced-motion preference.
- `observability`: aggregate metrics and synthetic performance checks.

Keep the simulation module independent from HTTP controllers. It should accept a canonical state plus ordered events and time inputs, then return state deltas, emitted observations, and operational timing data. This separation is critical for deterministic calibration tests.

## 4. Core Data Model

All IDs should be opaque UUIDs or ULIDs. The account UUID is the only identifier allowed in internal messages, logs, partitions, and foreign keys. Email is encrypted and stored only on the account record.

### Account

Fields:

- `id`: synthetic UUID.
- `encrypted_email`: encrypted canonical email.
- `email_verified_at`.
- `pending_email_change_encrypted`, nullable.
- `created_at`, `updated_at`.
- `deleted_at`, nullable.
- `hard_delete_after`, nullable.
- `privacy_policy_version_seen`.
- `visit_notifications_enabled`: default false.
- `timezone`: IANA timezone detected at onboarding and user-editable.

Invariants:

- No service uses email as identifier.
- Soft-deleted accounts are recoverable for 30 days.
- Hard deletion removes account, birds, vectors, notebook, visits, events, telemetry join keys, and export artifacts.

### Session Token

Fields:

- `id`.
- `account_id`.
- `device_label`.
- `token_hash`.
- `created_at`, `last_seen_at`, `revoked_at`, `expires_at`.

Tokens are per-device and revocable from account settings.

### Magic Link

Fields:

- `id`.
- `email_hash` or encrypted email lookup key.
- `token_hash`.
- `created_at`, `expires_at`.
- `consumed_at`.
- `request_ip_hash`, `request_user_agent_hash` for rate-limiting and abuse controls.

Links expire after 15 minutes and are invalid after first use.

### Aviary

Fields:

- `id`.
- `account_id`.
- `created_at`.
- `local_timezone`.
- `bird_cap`: constant 7 for v1.
- `settled_until_reengaged`: boolean or state flag scoped to active session presentation, not long-term penalty.
- `last_tick_at`.
- `next_tick_after`.
- `snapshot_version`: monotonic integer.
- `active_weather_event_id`, nullable.

One account has exactly one aviary in v1.

### Bird Species

Versioned static data:

- `species_key`.
- `display_family_name` for naturalist prose, not necessarily shown as UI stat.
- `silhouette_profile`.
- `default_palette`.
- `motif_library_key`.
- `night_activity_profile`.
- `pose_set_key`.

Species rarity is not a user-facing concept. New birds draw from the pool by calm variety constraints, not collectible rarity.

### Bird

Fields:

- `id`: stable bird UUID, never regenerated.
- `aviary_id`.
- `species_key`.
- `name`.
- `adopted_at`.
- `name_updated_at`.
- `current_mood`.
- `mood_updated_at`.
- `current_perch_zone`: front, middle, or back.
- `current_pose_key`.
- `last_call_at`.
- `last_greeted_at`.
- `created_from_starter_pair`: boolean.

Identity invariants:

- Renaming does not change species, personality, mood, or call identity.
- Migrations must preserve `id` and personality vectors.
- Bird count is capped at seven.

### Personality Vector

Persist one row per bird:

- `bird_id`.
- `boldness`.
- `social_warmth`.
- `vocal_frequency`.
- `plumage_saturation`.
- `curiosity`.
- `updated_at`.
- `version`.

Use normalized numeric ranges, for example 0.0 to 1.0, with private calibration constants. Values are never exposed to users, narration, captions, notebook UI, exports are allowed only because account export explicitly includes vectors. They must not appear in product UI or telemetry.

### Bird Mood

Mood is stored on the bird or in a `bird_mood_state` table:

- `bird_id`.
- `mood`: wary, content, curious, drowsy, alert, settled.
- `mood_started_at`.
- `decay_after`.
- `last_transition_reason`: internal enum for debugging, not user-facing.

Mood persists across sessions and is advanced by the server tick.

### Interaction Event

Append-only records:

- `id`.
- `aviary_id`.
- `account_id`.
- `session_id`.
- `client_event_id`: idempotency key.
- `type`: presence_ping, listen_in_start, listen_in_end, offer_seed, offer_song_fragment, offer_still_pool, settle_start, settle_undo, settle_complete, rename_bird, accessibility_setting_changed.
- `bird_id`, nullable.
- `occurred_at_client`.
- `received_at_server`.
- `payload`: constrained JSON per event type.
- `consumed_by_tick_id`, nullable.

Invariants:

- Clients submit events, never state deltas.
- Event payloads cannot contain personality values.
- Duplicate `client_event_id` for the same session is ignored.
- Events are consumed by the simulation tick in deterministic server order with client time used only as secondary context.

### Presence Window

Store aggregated windows derived from presence pings:

- `id`.
- `aviary_id`.
- `account_id`.
- `session_id`.
- `started_at`.
- `ended_at`.
- `duration_seconds`.
- `source_event_ids` or compact range.
- `closed_reason`: hidden, blurred, inactivity_timeout, settle, tab_close, session_expired.

Presence requires all three client signals at the same time:

- Document visibility is visible.
- Window has focus.
- Pointermove or keypress occurred within calibrated recent window.

The server should validate cadence and cap maximum contiguous presence to prevent runaway drift from buggy clients.

### Offer Cooldown

Fields:

- `bird_id`.
- `offer_type`.
- `available_after`.
- `last_offer_event_id`.

Cooldown is per-bird and per-offer class. It prevents within-session saturation without framing the user as punished.

### Field Notebook Entry

Fields:

- `id`.
- `aviary_id`.
- `created_at`.
- `entry_date_local`.
- `prose`.
- `trigger_type`: greeting_pattern, quiet_stretch, weather_moment, mood_shift, adoption, age_milestone, rare_chorus.
- `related_bird_ids`: array.
- `internal_significance_score`: not user-facing.

Entries are sparse. Store prose as rendered at creation time so the notebook is a historical record, not a live reinterpretation.

### Visit Invite And Visit Session

Invite fields:

- `id`.
- `host_account_id`.
- `aviary_id`.
- `visitor_email_encrypted`.
- `visitor_email_lookup_hash`.
- `token_hash`.
- `created_at`.
- `expires_at`: 30 days.
- `revoked_at`, nullable.
- `first_used_at`, nullable.
- `last_used_at`, nullable.

Visit session fields:

- `id`.
- `invite_id`.
- `started_at`.
- `ended_at`.
- `approx_duration_seconds`.
- `last_snapshot_at`.

Visit sessions do not create presence windows and do not emit simulation interaction events.

### Accessibility Settings

Fields:

- `account_id`.
- `reduced_motion`: system, on, off.
- `call_captions_enabled`.
- `audio_enabled`.
- `screen_reader_narration_enabled`.
- `high_contrast_focus_enabled`, if needed after design validation.

Settings surfaces use matter-of-fact system voice.

## 5. API Surface

Use versioned JSON endpoints under `/api/v1`. All mutation endpoints accept an idempotency key. All user-facing error text on system surfaces uses matter-of-fact voice.

### Auth

- `POST /auth/magic-link/request`
  - Input: email.
  - Behavior: create a 15-minute magic link subject to rate limits.
  - Response: generic success regardless of account existence.

- `POST /auth/magic-link/consume`
  - Input: token.
  - Behavior: atomically consume token, create account if needed, create per-device session, initialize starter aviary if new.
  - Response: session token and first navigation target.

- `GET /sessions`
  - Returns active device sessions.

- `DELETE /sessions/:sessionId`
  - Revokes a session.

- `POST /account/email-change/request`
  - Starts new email verification.

- `POST /account/email-change/confirm`
  - Verifies and commits new email.

### Aviary State

- `GET /aviary/snapshot`
  - Auth: account session or valid visit session.
  - Query: `since_version` optional.
  - Response: compact canonical snapshot containing aviary time, snapshot version, birds, current moods, perch zones, pose seeds, call scheduling hints, weather, settled visual state, notebook unread-neutral metadata if needed, and capability flags.
  - Does not include personality vector numbers.

- `GET /aviary/bootstrap`
  - Returns HTML-embedded or edge-near minimal first snapshot for time-to-first-bird. This should include enough state to draw the first bird without waiting on settings, notebook, or account chrome.

Snapshot shape should separate canonical state from render hints:

- Canonical: bird IDs, names, species keys, moods, perch zones, local time phase, weather type, snapshot version.
- Render hints: deterministic seeds for current pose variation, call motifs due in the near window, interpolation target timing.
- Client-only: actual animation frames, leaf drift particles, per-frame audio synthesis.

### Interaction Events

- `POST /aviary/events`
  - Accepts a batch of constrained events.
  - Events: presence pings, listen-in start/end, offers, settle start/undo/complete.
  - Response: accepted event IDs and current server time.

- `GET /aviary/offer-state`
  - Returns per-bird offer availability without exposing cooldown as a game timer. The UI should use this to disable or soften offer controls quietly, not show countdowns.

Interaction endpoints must validate:

- The account owns the aviary.
- Visitors cannot submit events.
- Bird IDs belong to the account.
- Offer cooldowns are respected.
- Client event IDs are idempotent.

### Notebook

- `GET /notebook`
  - Pagination by cursor.
  - Returns historical prose entries.

No create, update, or delete endpoints exist for users.

### Visits

- `POST /visits/invites`
  - Host enters visitor email; system sends one-time link.
  - Visits are off unless an invite exists.

- `GET /visits/invites`
  - Account settings surface showing outstanding, active, expired, and revoked invites plus visit log.

- `DELETE /visits/invites/:inviteId`
  - Revokes invite immediately.

- `POST /visits/consume`
  - Visitor consumes invite token and receives a visit session.

- `GET /visits/session/:id/snapshot`
  - Read-only snapshot endpoint, same aviary view as host but no interaction affordances.

Visitor error surfaces:

- Visit no longer available.
- Visit expired.
- Something went wrong loading this aviary. Try again later.

### Account Data

- `POST /account/export`
  - Queues export job and emails a short-lived download link to verified email.

- `GET /account/export/:jobId`
  - Returns status.

- `POST /account/delete`
  - Marks account for deletion and sets hard delete date.

- `POST /account/recover`
  - Recovers within 30-day soft-deletion window.

- `GET /account/privacy`
  - Returns privacy-policy text and aggregate telemetry categories.

## 6. Sync And Consistency Model

The sync model is intentionally simple: the server is the only source of canonical aviary state, and the simulation tick is the only writer of personality vectors and moods.

Client flow:

1. On navigation, request bootstrap snapshot.
2. Render immediately from snapshot with motion already in progress.
3. Start low-frequency snapshot polling while visible.
4. Pull a fresh snapshot on visibility gain, focus regain after long gap, network reconnect, and laptop wake detection.
5. Submit interaction events in small batches with idempotency keys.
6. Interpolate between snapshots, never mutate canonical state locally.

Conflict prevention:

- Personality vectors are never last-write-wins.
- Event ingestion is append-only.
- The tick consumes ordered events and applies additive server-authored deltas.
- Per-aviary ticks run under a lock so two ticks cannot update the same aviary concurrently.
- Snapshot versions are monotonic; clients discard older snapshots.
- Client clock is never trusted for ordering across devices; server receipt order and tick boundaries define consumption order.

Multiple devices:

- Two devices can submit presence and interaction events for the same account.
- The server aggregates events into one canonical stream.
- Presence windows should be capped and deduplicated by session so overlapping devices do not create impossible drift. If two authenticated devices are active at once, record both sessions operationally but cap drift contribution to a calibrated per-account maximum per tick.
- Listen-in and offer events affect their target bird when consumed; if cooldown prevents an offer, the event is accepted as attempted but ignored for drift, and the next snapshot reflects no special reaction.

Visitors:

- Visitors pull snapshots through visit authorization.
- Visitors never create presence, listen-in, offer, settle, notebook, or drift events.
- Revocation is enforced on the next snapshot pull and should also invalidate visit session tokens server-side.

## 7. Simulation Engine Design

The simulation tick is the heart of the product. It should be deterministic for a given input state, event sequence, and time seed.

### Tick Cadence And Execution

- Target cadence: about once per minute per active aviary, calibrated during build.
- Inactive aviaries still tick, but the scheduler can coalesce catch-up ticks after long inactivity into bounded time steps to avoid waste.
- A tick reads current canonical state, recent unconsumed events, local timezone context, ambient event schedule, and species/motif data.
- A tick writes personality deltas, mood transitions, perch/pose/call scheduling state, notebook entries if warranted, snapshot version, and event consumption markers.
- Tick p99 latency alarm threshold: 5 seconds.

Use a per-aviary transaction:

1. Acquire per-aviary lock.
2. Read aviary, birds, personality vectors, mood state, unconsumed events.
3. Convert raw events into normalized signals.
4. Update presence windows.
5. Compute drift deltas.
6. Compute mood transitions and perch preferences.
7. Schedule near-future call hints and ambient events.
8. Possibly generate a sparse notebook entry.
9. Persist updates and increment snapshot version.
10. Release lock.

### Drift Function

Drift is a low-pass filter over positive presence and interaction signals. It must be slow enough that no single session is visibly meaningful and fast enough that instrumentation detects regular use after about a week.

Inputs:

- Presence-time, dominant.
- Listen-in duration per bird.
- Offer attempts and accepted offers, with small weight.
- Time since adoption and aviary age for adoption pacing.
- Settle only closes a presence window and may quiet mood; it does not directly increase personality.

Trait effects:

- Presence-time nudges boldness, social warmth, vocal frequency, plumage saturation, and curiosity upward at different weights.
- Listen-in strongly nudges the focused bird's social warmth and vocal frequency.
- Offers near a bird nudge boldness and curiosity; accepted offers nudge curiosity slightly more.
- Plumage saturation moves only upward and slowly.
- Neglect never decreases traits.

Implementation approach:

- Maintain per-trait target deltas for each tick based on normalized signal intensity.
- Apply a very small bounded delta with a saturation curve as values approach trait caps.
- Use per-bird and per-species coefficients so birds remain distinct.
- Keep coefficients in versioned config.
- Log aggregate tick timing and delta magnitudes only in internal operational logs without account-identifiable analytics export.

Calibration tests:

- A simulated regular visitor profile shows measurable numeric drift after about seven days.
- The same profile shows visible render/audio differences after about three weeks.
- A user absent for two weeks does not produce negative trait movement.
- A tab left open without focus or recent pointer/key activity produces no presence drift.
- Repeated offers inside cooldown do not saturate curiosity.

### Mood System

Mood is fast-timescale, persistent, and shaped by current context.

Mood states:

- wary.
- content.
- curious.
- drowsy.
- alert.
- settled.

Transition inputs:

- Recent accepted or ignored offers.
- Listen-in engagement.
- Time of day in account timezone.
- Ambient rain or wind.
- Bird-to-bird calls, alarm-like motifs, chorus events.
- Personality vector bias.
- Last known mood and time spent in it.

Implementation approach:

- Use a weighted state machine rather than free-form continuous mood.
- Each tick computes transition probabilities from context and personality.
- Add hysteresis so mood does not flicker across adjacent ticks.
- Persist mood and started-at time.
- Expose mood only through behavior, pose, call cadence, narration, and notebook prose, not labels or badges.

Expected mappings:

- Wary birds prefer back perch, scan more, call less.
- Content birds preen and hold middle/front perches.
- Curious birds tilt toward sounds and investigate offers.
- Drowsy birds sit lower, fluff feathers, and call quietly.
- Alert birds scan and respond to ambient events.
- Settled birds quiet after the settle gesture or night phase.

### Return Greeting

The return greeting is generated from:

- Absence length since last presence window.
- Bird boldness.
- Social warmth.
- Current mood.
- Local time.
- Recent greeting history to avoid the same bird always greeting.

Algorithm:

1. On first visible authenticated snapshot after absence, request a greeting candidate in the snapshot.
2. The server selects one primary greeting bird with weighted randomness.
3. If other birds would respond, schedule staggered secondary reactions with small random offsets.
4. Client renders the greeting as ongoing behavior, not an overlay.
5. No text toast, no banner, no absence summary.

The greeting variants should include glance, two-note call, head tilt, step toward front perch, longer call, or call-and-response. The variation seed comes from the snapshot so repeated render attempts do not duplicate or desync.

### Ambient Events

Weather and ambient motion split across server and client:

- Server owns rare weather events that affect mood and call cadence.
- Client owns decorative leaf and feather drift with local random seeds.

Weather:

- Short rain a few times per week.
- Occasional soft wind.
- No thunderstorm, snow, or assertive event.
- Mood effects are small and short-lived.

Day/night:

- Based on account timezone.
- Morning, midday, evening, night phases.
- Evening warms and quiets.
- Night settles most birds while allowing nightjar-like species to remain active.

### Notebook Generation

Notebook entries are sparse, naturalist, historical observations. The simulation tick should create candidates, then pass them through a sparsity gate.

Candidate triggers:

- First-greeter variation over a week.
- Long quiet stretch.
- Rare chorus.
- Weather plus mood moment.
- Adoption of a new bird.
- Notable shift that is visible in behavior, not raw vector values.

Rules:

- Roughly one entry every few days for regular use.
- More frequent only for genuinely notable events.
- Never write visit-frequency observations.
- Never expose personality numbers.
- Never read like event logs.
- Store final prose at creation.

Use template families with parameterized bird names, species descriptors, local date words, and visible behavior. Keep all product-surface prose lowercase, present-tense, and specific.

## 8. Frontend Rendering Pipeline

The aviary is one horizontal scene that fits on screen without panning, scrolling, or zooming.

### Scene Composition

Layers:

- Soft sky and background foliage.
- Back perch zone.
- Middle perch zone with birds.
- Front perch zone.
- Occasional foreground branch/leaf.
- Separate top bar above the scene.

Scene state comes from snapshots. Client render state includes:

- Current viewport mapping.
- Per-bird interpolation from current to target perch/pose.
- Idle micro-motion phase.
- Ambient ornament particles.
- Focus outline geometry.
- Audio/caption synchronization state.

The renderer must guarantee:

- All birds stay visible on supported viewports.
- Narrow viewports compress horizontally without cropping birds.
- Wide viewports add breathing room without turning the scene into an explorable map.
- No UI chrome appears inside the scene.

### First Frame And Loading

The first visible frame should be the aviary already in motion:

- Inline or edge-near bootstrap snapshot with enough state to draw the first bird.
- Critical renderer code in initial bundle.
- Non-critical account/settings/notebook modules lazy-loaded.
- No spinner.
- If slow connection prevents immediate snapshot, show quiet field with soft color and faint motion cues.
- Once snapshot arrives, draw birds directly in current poses, not through a wake-up animation.

### Idle Motion

Normal mode:

- Preening.
- Scanning.
- Head tilts.
- Weight shifts.
- Small call posture changes.
- Mood-shaped and personality-biased cadence.

Reduced-motion mode:

- Slow cross-fades between still poses.
- Cross-fade perch transitions instead of flight paths.
- No leaf drift.
- Day/evening color shifts remain but slow.
- Audio, captions, drift, mood, and notebook remain full product behavior.

### Interaction UI

Top bar:

- Account/settings.
- Accessibility settings.
- Field notebook.
- Offer affordance.
- Settle affordance if not integrated into offer/action menu.

Top bar behavior:

- Fade nearly transparent after cursor stillness.
- Return on cursor movement or keyboard activity.
- Always reachable by keyboard.
- Do not show badges, counters, or notification dots.

Bird focus:

- Pointer/tap on bird starts listen-in.
- Keyboard Tab enters scene and focuses first bird.
- Arrow keys move focus between birds.
- Enter toggles listen-in.
- Escape exits listen-in.
- Focus indicator is visible across day/night palettes.

Offer:

- Started from top bar, not by clicking a bird.
- Seed, song fragment, still pool.
- UI should feel like offering a gesture, not selecting an action in a game.
- Cooldown should be represented through gentle unavailable state, not visible timers.

Settle:

- Trigger from top bar.
- Lighting shifts to evening over several seconds.
- Calls quiet.
- Any click in the aviary within five seconds undoes settle.
- Tab close without settle is equally valid and receives no recovery surface.

## 9. Audio Pipeline

Calls are procedural and synthesized client-side with WebAudio.

### Call Grammar

Each species has a motif library. Each bird instance receives stable per-bird call identity parameters derived from species plus bird ID:

- Base pitch range.
- Motif preference weights.
- Timing jitter profile.
- Envelope shape.
- Timbre/noise mix.
- Mood modulation coefficients.
- Personality modulation coefficients.

Runtime call generation:

- Server snapshot provides upcoming call intent hints or seeds.
- Client synthesizes actual oscillator/noise/filter/envelope graph.
- Vocal frequency affects probability and timing of calls.
- Mood affects pitch, spacing, and envelope.
- Bird identity keeps recognizability across drift.

No audio loops are shipped. No recorded fallback exists.

### Mixing

Ambient mix:

- All active birds have low-level presence.
- Chorus events emerge from overlapping procedural calls.
- Weather and time of day shape overall gain and density.

Listen-in:

- Focused bird ramps up gradually.
- Other birds ramp down gradually but never to silence.
- Disengage returns to ambient with the same slow ramp.
- Use equal-power or perceptually smooth curves to avoid hard cuts.

### Captions

Caption text is generated from the same procedural call grammar that produces sound:

- "a soft three-note rise"
- "a low trill, paused, low trill again"
- "a single sharp call from the back perch"

Captions:

- Are opt-in from accessibility settings.
- Turn on by default if WebAudio is unavailable or audio permission fails.
- Appear near the calling bird.
- Fade in/out with calls.
- Use naturalist voice.
- Must match the actual generated call, not a generic bird label.

### WebAudio Failure

If WebAudio is unavailable:

- Do not load recorded audio.
- Continue visual aviary.
- Enable captions by default for the session.
- Show matter-of-fact settings explanation only if the user opens audio/accessibility settings.

## 10. Accessibility Plan

Accessibility ships with v1, not as a patch.

### Screen-Reader Narration

Provide a dedicated narration stream, not raw ARIA state dumps.

Behavior:

- Generate naturalist prose from canonical snapshot state.
- Idle cadence: roughly every 30 to 60 seconds.
- Faster only for user-initiated events such as return greeting, offer reaction, or settle.
- User can pause or configure narration from accessibility settings.
- Narration uses lowercase, present-tense, specific observations.

Implementation:

- Maintain a narration queue with priorities.
- Avoid queue spam by coalescing low-priority ambient updates.
- Use polite live regions for ordinary updates and assertive only for system errors.
- Do not announce personality values, mood labels, or perch numbers.

### Keyboard And Focus

All controls are keyboard reachable:

- Tab through top bar.
- Tab into scene.
- Arrow among birds.
- Enter for listen-in.
- Escape to leave listen-in.
- Offer menu fully keyboard navigable.
- Settle reachable from top bar.
- Settings and notebook use standard accessible modal or page patterns.

Focus indicators:

- Visible at WCAG AA contrast across palette states.
- Soft enough to fit the product but never ambiguous.

### Reduced Motion

Respect `prefers-reduced-motion` on first load. Persist explicit user choice in account settings.

Reduced-motion renderer must:

- Use the same snapshot state.
- Replace motion with slow pose cross-fades.
- Remove drifting leaves/feathers.
- Avoid flight paths.
- Preserve day/night color changes at slower cadence.
- Preserve all simulation, audio, captions, notebook, and visit behavior.

### Contrast And Copy

- All top bar labels, settings, account surfaces, errors, captions, and visual narration pass WCAG AA.
- Product-surface prose uses naturalist voice.
- Account, error, sync, privacy, and accessibility settings use matter-of-fact voice.

## 11. Privacy And Telemetry Boundaries

The privacy architecture is part of the product.

Allowed telemetry:

- Request counts.
- API latencies.
- Simulation tick latencies.
- Error rates.
- Aggregate session-duration histograms without account dimension.
- First-bird-render timing.
- Render-frame timing.
- Audio-context errors.
- Bundle size and memory tests.

Disallowed telemetry:

- Per-bird interaction history.
- Personality vectors.
- Mood history tied to an account.
- Offer/listen-in details in analytics warehouse.
- Account-level bird behavior dashboards.
- Population-level drift analysis.
- ML training or recommendation use of per-bird records.

Implementation rules:

- Simulation database is not a source for analytics warehouse jobs.
- Logs use synthetic account IDs only where needed for debugging and are sampled/redacted.
- Per-account debugging access requires explicit support tooling and audit logs.
- Metrics emitted from simulation code must be operational aggregates, not product behavior aggregates.

## 12. Performance Plan

Budgets:

- Initial JS bundle under 2MB gzipped.
- First bird visible within 500ms on mid-tier mobile over 4G.
- 60fps idle motion on a five-year-old mid-range laptop.
- No memory growth over a 30-minute session.
- Simulation tick p99 latency alarm at 5 seconds.

Implementation tactics:

- Keep critical bootstrap renderer small.
- Lazy-load notebook, account settings, accessibility settings, visit management, and export/delete flows.
- Use compact vector/SVG/procedural bird assets where possible.
- Avoid recorded audio assets.
- Reuse WebAudio nodes and buffers.
- Pool render objects for particles and captions.
- Pause rendering while document is hidden, but keep server simulation running.
- Use edge-cached HTML plus small personalized bootstrap state when feasible.
- Avoid hydration work for non-critical chrome before first bird.

CI and synthetic checks:

- Bundle-size gate.
- Lighthouse-like first-bird measurement on mobile profile.
- 30-minute memory soak.
- Render frame timing test with seven birds.
- WebAudio stress test with chorus and listen-in.
- Reduced-motion rendering regression test.
- Snapshot payload size test.

## 13. Rollout And Delivery Milestones

### Milestone 1: Foundations

- Project skeleton, typed domain model, DB migrations.
- Magic-link auth and sessions.
- Account UUID and encrypted email storage.
- Starter aviary creation with two stable birds.
- Basic snapshot API with placeholder scene state.
- Client bootstrap that draws a quiet field and birds from snapshot.

Exit criteria:

- New account can sign in and see two named starter birds.
- No personality values in UI.
- No client writes canonical state.

### Milestone 2: Simulation Core

- Append-only event log.
- Presence pings with three-signal validation.
- Server-side tick with per-aviary locking.
- Mood state machine.
- Initial drift function with calibration harness.
- Perch selection and return greeting selection.

Exit criteria:

- Multi-device clients read same canonical aviary.
- Regular presence produces measurable drift after simulated week.
- Absence does not create negative drift.
- Background tab produces no presence.

### Milestone 3: Rendering And Interaction

- Full single-screen responsive scene.
- Perch zones and mood-shaped idle poses.
- Top bar with fade behavior.
- Listen-in focus interaction.
- Offers and settle gesture.
- Quiet loading field and first-frame motion.

Exit criteria:

- No spinner or welcome toast.
- First bird visible under performance budget in synthetic profile.
- Keyboard can operate listen-in, offer, and settle.

### Milestone 4: Audio And Captions

- Procedural motif libraries for species pool.
- WebAudio synthesizer and mixer.
- Listen-in mix ramps.
- Chorus behavior.
- Runtime-generated captions.
- WebAudio failure path with captions on.

Exit criteria:

- Same bird is recognizable across moods in internal listening tests.
- No recorded call files in bundle.
- Captions match generated motif structure.

### Milestone 5: Notebook And Accessibility

- Sparse notebook candidate generation.
- Naturalist prose templates.
- Screen-reader narration queue.
- Reduced-motion renderer.
- WCAG AA verification for copy surfaces.
- Accessibility settings.

Exit criteria:

- Narration is prose, not state labels.
- Reduced-motion mode preserves product behavior.
- Notebook entries remain sparse under active-use simulation.

### Milestone 6: Visits, Export, Deletion, Privacy

- Invite creation, consumption, expiration, revocation.
- Read-only visit snapshots.
- Visit log in account settings.
- Account export job and email link.
- Soft deletion and hard deletion worker.
- Privacy policy surface.

Exit criteria:

- Visitor cannot generate interaction events.
- Revocation blocks next snapshot.
- Export contains owned aviary snapshot.
- Hard deletion removes tied records.

### Milestone 7: Beta And Launch Hardening

- Synthetic performance fleet.
- Operational dashboards.
- Calibration review for drift and mood.
- Audio uncanny/repetition testing.
- Accessibility audit.
- Privacy review.
- Browser support pass.

Launch criteria:

- All performance budgets pass.
- No gamification or notification surfaces present.
- Privacy boundary reviewed at code and data-pipeline level.
- Reduced-motion, captions, keyboard, and narration are launch-ready.
- Seven-bird stress profile remains performant and comprehensible.

## 14. Bird Growth And Adoption Pacing

New accounts start with two birds selected by the system from the species pool. The user names them but does not choose species from a catalog.

Additional birds:

- Become available based on aviary age, not visit count, score, or interaction volume.
- Are presented as birds that arrived, not rewards earned.
- Must not use language like unlock, level, achievement, or milestone reward.
- Never push notifications or emails.
- Should appear quietly in the aviary flow when the user is present.

The adoption scheduler should be configuration-driven:

- Third bird after a few months.
- Later birds at longer intervals.
- Year-old aviary may have five or six.
- Hard maximum seven.

Adoption events can create notebook entries, but not celebratory achievement language.

## 15. Copy And Voice Rules

Product surfaces:

- Naturalist voice.
- Lowercase by default.
- Present tense.
- Specific bird and moment language.
- No exclamation-heavy celebration.
- No direct engagement framing where the bird can carry the moment.

System surfaces:

- Matter-of-fact voice.
- Normal capitalization.
- Direct error and action language.
- Used for auth, account, sync, privacy, accessibility settings, unsupported browser, visit revoked/expired, deletion, export.

Forbidden copy patterns:

- Welcome back.
- You have been gone X days.
- Streak, achievement, badge, level, score, XP, rank.
- Your bird is happier.
- Trait names or numbers.
- Friend visited notifications by default.

## 16. Testing Strategy

### Unit Tests

- Drift deltas are monotonic non-negative.
- Presence requires all three signals.
- Offer cooldown prevents same-session saturation.
- Mood transition hysteresis prevents flicker.
- Magic links expire and are single-use.
- Visit revocation invalidates sessions.
- Client event idempotency.
- Snapshot version ordering.
- Personality vectors are absent from UI snapshot responses.

### Integration Tests

- New account onboarding creates one aviary and two birds.
- Laptop and phone sessions submit events to one event log.
- Tick consumes events in order and updates canonical state.
- Visitor session can pull snapshots but cannot submit events.
- Account deletion removes all tied data after hard-delete window.
- Export includes required account-owned state.

### Simulation Calibration Tests

- Regular presence profile over one week yields measurable drift.
- Regular presence profile over three weeks yields behavior/render differences.
- Two-week absence produces quietness but no distress or negative drift.
- Multiple device overlap does not double-count beyond cap.
- Seven birds preserve call recognizability in scripted mix tests.

### Client Tests

- First-bird timing budget.
- Responsive scene keeps all birds visible.
- Top bar fades and returns on input.
- Keyboard navigation path.
- Reduced-motion cross-fades.
- Captions synchronize with call generation.
- WebAudio failure enables captions and avoids recorded fallback.
- 30-minute memory soak.

### Content And Product Guardrail Tests

Use static and snapshot tests for forbidden surfaces:

- No welcome toast/banner strings.
- No streak/achievement/score/badge/level strings.
- No visible personality trait labels.
- No visit notification badges by default.
- No buttons inside the aviary scene except focusable birds as scene subjects.

These tests are not a substitute for review, but they catch predictable regressions.

## 17. Observability And Operations

Dashboards:

- Auth request volume and failure rate.
- Snapshot latency and payload size.
- Simulation tick queue depth, duration, failure count, p99 latency.
- First-bird-render timing.
- Client frame timing.
- Audio context errors.
- Memory soak regressions from CI.
- Export and deletion job status.
- Visit snapshot authorization errors.

Alerts:

- Tick p99 over 5 seconds.
- Tick failure rate above threshold.
- Snapshot p95 latency above launch SLO.
- Magic-link send failure spike.
- Error rate on bootstrap snapshot.
- Hard-delete worker backlog beyond policy window.

Do not build dashboards showing:

- Most-listened birds.
- Average drift.
- Offer rates per bird.
- Account-level bird behavior.
- Visit popularity rankings.

Operational debugging should use short-lived, audited support views with synthetic account IDs and minimal necessary state.

## 18. Security And Abuse Controls

- Encrypt email at rest.
- Store token hashes, never raw tokens.
- Rate-limit magic-link requests by email lookup hash, IP hash, and broader abuse heuristics.
- Expire magic links after 15 minutes.
- Invalidate used links immediately.
- Use CSRF protections where cookie sessions are used.
- Validate event ownership on every event.
- Ensure visitor tokens cannot access host settings, notebook if not intended, or mutation endpoints.
- Revoke visit sessions immediately server-side on invite revocation.
- Keep export links short-lived and account-bound.
- Audit account deletion and recovery.

The security model should avoid turning abuse controls into user-visible friction unless necessary. When shown, errors use matter-of-fact voice.

## 19. Key Risks And Mitigations

### Drift Calibration Too Fast

Risk: birds visibly change session-to-session, making the product feel like stat management.

Mitigation:

- Simulation calibration suite before beta.
- Slow low-pass coefficients.
- Product review using week and month time-lapse fixtures.
- No UI exposing drift numbers.

### Drift Calibration Too Slow

Risk: users feel nothing changes and the aviary becomes a screensaver.

Mitigation:

- Instrument-only measurable drift after one simulated week.
- Three-week visible behavior target.
- Notebook entries can notice visible behavior changes without numbers.

### Presence Inflation

Risk: background tabs or idle devices count as attention and corrupt drift.

Mitigation:

- Enforce visibility, focus, and recent pointer/key activity.
- Server-side cadence validation.
- Max contribution caps.
- Tests for background and laptop-sleep cases.

### Audio Feels Canned Or Uncanny

Risk: repeated motifs, hard listen-in cuts, or chorus artifacts break aliveness.

Mitigation:

- Procedural grammar only.
- Stable per-bird identity parameters with runtime variation.
- Listening tests across bird counts.
- Smooth mix ramps.
- No recorded fallback.

### Sync Corrupts Personality

Risk: concurrent devices overwrite drift or split canonical state.

Mitigation:

- Server-only personality writes.
- Append-only events.
- Per-aviary tick lock.
- Monotonic snapshot versions.
- Integration tests for overlapping sessions.

### Accessibility Becomes A Checklist Fallback

Risk: screen-reader and reduced-motion users receive flattened state labels or static product.

Mitigation:

- Accessibility workstream from milestone 1.
- Naturalist narration templates.
- Designed reduced-motion renderer.
- Accessibility acceptance criteria in launch gate.

### Social Feature Expands Into Network Behavior

Risk: visits lead to notifications, public profiles, comparison, or co-presence.

Mitigation:

- Visits module exposes only invite, revoke, read-only snapshot, and visit log.
- Static guardrails for forbidden copy.
- No visitor events in event schema.
- Notifications off by default and settings-only.

### Privacy Boundary Erodes

Risk: per-bird interaction data leaks into analytics for harmless-looking insights.

Mitigation:

- Data-pipeline-level separation.
- Analytics schemas exclude simulation records.
- Privacy review for every metric.
- Account UUID only in internal references.

### Performance Budget Missed

Risk: scene, audio, accessibility, and account code push load past the first-bird threshold.

Mitigation:

- Initial bundle budget in CI.
- Lazy-load non-critical surfaces.
- Procedural assets.
- Bootstrap snapshot optimization.
- Synthetic first-bird checks from the beginning.

## 20. Engineering Review Checklist

Before launch, verify:

- The first visible state is not a spinner or app-like loading sequence.
- No return text welcomes the user.
- Bird greeting varies by absence, mood, and personality.
- Personality values are hidden from UI.
- Presence requires all three signals.
- Drift is monotonic toward expressive.
- Server tick is the only writer of personality and mood.
- Client snapshots do not include raw personality vectors.
- Multi-device sessions share one canonical aviary.
- Visitors are render-only and do not affect drift.
- Notebook entries are sparse and naturalist.
- No gamification strings or surfaces exist.
- Reduced-motion mode is designed, not static.
- Call captions derive from actual procedural calls.
- WebAudio failure does not load recorded fallback.
- Account, sync, privacy, accessibility, and error surfaces use matter-of-fact voice.
- Telemetry excludes per-bird and per-account interaction analytics.
- Seven-bird cap is enforced in data and UI.

## 21. Defensible Implementation Choices For Ambiguities

The PRD leaves some implementation details open. Use these defaults unless later design work explicitly changes them:

- Use TypeScript across client and server for shared domain types.
- Use PostgreSQL as source of truth because transactional per-aviary ticks and append-only events are central.
- Use row-level or advisory locks for per-aviary tick serialization.
- Use pull-based snapshots for v1; add SSE only for invalidation if polling proves insufficient.
- Use Canvas/WebGL for the aviary scene to meet performance budgets, with DOM for controls and accessibility surfaces.
- Store notebook prose as generated text, not re-rendered from events.
- Use generated captions from call grammar rather than authored caption banks.
- Cap overlapping multi-device presence contribution at the account level.
- Treat account export as an account-settings job delivered by email link, not an instant client download.

These choices keep the product conservative, privacy-preserving, and aligned with the PRD's constraints while leaving visual design specifics to the design system.
