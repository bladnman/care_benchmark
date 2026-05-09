# Pocket Aviary V1 Implementation Plan

## Planning stance

This plan treats Pocket Aviary as an affective systems product, not a game, dashboard, pet simulator, or social app. The implementation must make the aviary feel like a place that continues without the user: birds are already mid-action on first paint, the server advances canonical state whether clients are connected, the audio is procedural, and the product notices the user through bird behavior rather than announcing through UI.

Where the PRD leaves implementation choices open, this plan makes conservative calls:

- Use a TypeScript web stack end to end, with a thin React-style UI shell for account/settings/notebook chrome and an imperative scene/audio runtime for the aviary itself.
- Use PostgreSQL as the primary canonical store, with append-only interaction events, row-level account partitioning by synthetic UUID, and a queue-backed simulation worker for once-per-minute ticks.
- Use a layered 2D render pipeline: canvas or WebGL for the aviary scene and birds, DOM only for top-bar chrome, dialogs, settings, notebook, captions, and accessibility live regions.
- Use deterministic procedural generation from curated grammars for calls, field-notebook entries, captions, narration, weather, and small variations. Do not introduce model-training, recommendation systems, or aggregate behavioral analysis of per-bird history.

## V1 Scope

V1 includes:

- Browser-only Pocket Aviary with one canonical aviary per single-user account.
- Email magic-link sign-in, revocable per-device sessions, email change verification, JSON account export, and 30-day soft deletion followed by hard deletion.
- Two starter birds per new account, user-assigned names, stable bird identities, a coherent species pool, and an age-gated path toward a maximum of seven birds.
- Server-authored canonical simulation state: personality vectors, mood, perch choice, weather, notebook entries, visit visibility, and state snapshots.
- Client-rendered aviary scene with continuous idle motion, responsive single-screen layout, day/night color, ambient visual motion, and no chrome inside the scene.
- Procedural WebAudio bird calls, chorus mixing, listen-in mix ramps, generated call captions, and graceful silence with captions when WebAudio is unavailable or blocked.
- Presence accounting as the conjunction of visible document, focused window, and recent pointer/key activity.
- Interactions: return-greeting, listen-in, offer, settle with five-second undo, field notebook, and quiet read-only visit invitations.
- Accessibility surfaces shipping with V1: screen-reader narration, reduced-motion rendering, captions, keyboard navigation, focus indicators, WCAG AA text contrast, and matter-of-fact system/error copy.
- Aggregate-only operational telemetry and synthetic performance checks.

V1 explicitly does not include:

- Native iOS or Android clients.
- Scores, levels, streaks, achievements, badges, visit calendars, counters, XP, ranks, tiers, or any equivalent gamification surface.
- Hunger, death, distress, negative trait decay, caretaker obligations, happiness meters, or feeding schedules.
- Shared aviaries, public profiles, follows, comments, chat, discovery feeds, leaderboards, public directories, co-presence, avatars, or friend-of-friend sharing.
- User-controlled bird placement, scene customization, multiple aviaries per account, payments, billing, push notifications, or marketing-style engagement emails.
- Exposing personality-vector numbers to users in any surface, including debug-like account views.

## System Architecture

### Service shape

Build the product as a web app backed by a small set of services with clear ownership:

- Web client: serves the application shell, renders the aviary, runs WebAudio synthesis, records presence signals, submits interaction events, and presents settings/notebook/visit/account surfaces.
- Auth/account service: handles magic-link request and consumption, sessions, email changes, export requests, deletion lifecycle, and visitor invitation management.
- Aviary state API: returns bootstrap and snapshot payloads for owner and visitor views, scoped by account UUID or visit token.
- Interaction event API: accepts idempotent append-only events from owner clients. Visitor clients never get this write path.
- Simulation worker: advances canonical aviary state on a slow server-side tick, consumes interaction events in order, updates mood/personality/notebook/weather state, and emits fresh snapshot versions.
- Export/deletion worker: generates account exports and completes hard deletion after the 30-day soft-delete window.
- Metrics pipeline: receives only aggregate operational metrics and synthetic performance results, never per-bird or per-account relationship history.

The simulation worker is the only component allowed to write personality vectors. Clients never submit absolute bird state. Most client interactions are event writes plus local ephemeral rendering response; canonical consequences arrive in later snapshots.

### Client/server split

The server owns:

- Account identity, sessions, invite authorization, and deletion/export state.
- Stable bird identity, species assignment, names, personality vectors, current mood, mood timers, canonical perch/position intent, weather state, age-gated adoption eligibility, notebook entries, and current snapshot version.
- Drift and mood calculations, cooldown enforcement, event ordering, conflict prevention, and privacy boundaries.
- Rare prose generation decisions that need canonical historical context, especially field notebook entries.

The client owns:

- Smooth rendering between canonical snapshots.
- Local interpolation, pose blending, micro-motion, ambient leaves/feathers, top-bar fade, responsive scene fitting, and reduced-motion pose cross-fades.
- WebAudio synthesis and mixing from server-provided bird call signatures plus runtime mood/personality parameters.
- Generated captions for the actual procedural call played.
- Presence signal detection and event batching.
- Ephemeral listen-in mix ramps and visual focus treatment.

The client may predict or animate toward the current state, but it does not advance simulation truth. If a tab is hidden, suspended, or offline, rendering can pause; simulation continuity comes from the next server snapshot.

### Render pipeline boundary

Use a single scene runtime that consumes a typed `AviarySnapshot` and produces:

- Visual frame state for birds, perches, background, weather, day/night palette, and ambient ornaments.
- Audio scheduling inputs for each bird's call grammar.
- Accessibility state for captions, keyboard focus, and narration generation.

The DOM should not be the per-frame animation engine. DOM is appropriate for top-bar controls, account flows, settings, notebook, invite screens, captions, and live regions. Birds and scene motion belong in canvas/WebGL/SVG-runtime layers to protect the 60fps and memory budgets.

## Data Model

### Core records

`accounts`

- `id`: synthetic UUID, primary account identifier used everywhere except the encrypted email field.
- `email_ciphertext`: encrypted verified email.
- `email_lookup_hmac`: keyed lookup value for magic-link routing and rate limiting; never logged.
- `created_at`, `updated_at`, `timezone_last_seen`, `deleted_at`, `hard_delete_after`.
- `settings`: audio preference, captions preference, reduced-motion override, visit-notification opt-in, accessibility settings, privacy-policy acknowledgment version.

`sessions`

- `id`, `account_id`, `device_label`, `created_at`, `last_seen_at`, `revoked_at`.
- Per-device session token hash; raw tokens are never stored.

`aviaries`

- `id`, `account_id`, `created_at`, `current_snapshot_version`, `last_tick_at`.
- One row per account in V1. Enforce this at the database layer.
- `settled_until_session_end` is client-ephemeral, not a long-term aviary property, but the tick can store recent settle events for mood quieting.

`species`

- Static or migrated reference data: species id, silhouette profile, default palette, call motif library, night-activity flag, pose set, size class, and safe range modifiers.
- No rarity field in V1.

`birds`

- `id`: stable UUID that survives rename, sync, migration, and species-library changes.
- `aviary_id`, `species_id`, `name`, `adoption_index`, `adopted_at`.
- `call_seed`, `motion_seed`, and optional `notebook_voice_seed` for deterministic variation without losing identity.
- `personality_vector`: hidden scalar values for boldness, social warmth, vocal frequency, plumage saturation, curiosity.
- `mood_state`: enum such as wary/content/curious/drowsy/alert/settled, plus `mood_entered_at` and confidence/decay fields if needed.
- `current_perch_zone`: front/middle/back/high/low as a canonical intent, with client-specific exact positions derived from responsive layout.
- `last_offer_at_by_offer_type` or equivalent server-side cooldown state.

`interaction_events`

- Append-only log with `id`, `aviary_id`, `account_id`, optional `bird_id`, `session_id`, `device_id`.
- `event_type`: `presence_ping`, `presence_end`, `session_open`, `visibility_return`, `listen_in_start`, `listen_in_end`, `offer_seed`, `offer_song_fragment`, `offer_still_pool`, `settle`, `settle_undo`, `rename_bird`, `adoption_name_confirmed`.
- `client_event_at`, `server_received_at`, `idempotency_key`, `client_sequence`, `payload`, `consumed_by_tick_id`.
- Events are ordered by server receipt plus idempotency/sequence safeguards. The simulation worker can use client timestamps for duration estimates only within bounded skew tolerance.

`presence_windows`

- Derived records created by the simulation worker or event processor from valid presence pings.
- Store account-local windows with start, end, duration, device/session, and validation notes.
- These are simulation inputs, not analytics facts, and remain in the simulation database.

`aviary_snapshots`

- Current canonical render snapshot per aviary, plus a short history for debugging/replay.
- Includes snapshot version, tick id, server time, account timezone, bird render intents, mood, call scheduling hints, weather, day/night phase, notebook unread count if needed, and return-greeting eligibility data.
- Do not store generated per-frame motion; store compact state from which the client can interpolate.

`simulation_ticks`

- `id`, `aviary_id`, `started_at`, `completed_at`, `status`, `input_event_range`, `duration_ms`.
- Contains operational timing and replay references, not user-facing prose.

`field_notebook_entries`

- `id`, `aviary_id`, `created_at`, `entry_date_local`, `entry_text`, `source_pattern`, referenced bird ids, optional referenced event ids.
- Entries are read-only and sparse. They are generated from curated observation patterns, not from generic event logs.

`visit_invitations`

- `id`, `host_account_id`, `visitor_email_ciphertext`, `visitor_email_lookup_hmac`, `token_hash`, `created_at`, `expires_at`, `revoked_at`, `used_at`, `last_visit_at`.
- Invites are explicit and per-email. Tokens are one-time for claiming a visitor session or scoped visit session, not permanent public URLs.

`visit_sessions`

- `id`, `invitation_id`, `host_aviary_id`, `created_at`, `last_snapshot_at`, `ended_at`, `end_reason`.
- Read-only authorization context. No presence events, no interaction events, no drift inputs.

`visit_log_entries`

- `host_account_id`, `visitor_email_ciphertext`, approximate duration, visit date/time, invitation id.
- Reachable from account settings only. No badge or push surface.

`account_exports`

- Request status, requested_at, completed_at, expiry, encrypted download object reference, and delivery email status.

### State invariants

- One account has exactly one aviary in V1.
- The account synthetic UUID is the only identifier used in logs, telemetry dimensions, partitions, queues, and database relationships.
- Email appears only in encrypted account/invite fields and carefully controlled email-delivery jobs.
- Only the simulation worker writes personality vectors, mood transitions, notebook entries, and canonical snapshot versions.
- Clients can write events, but never personality, mood, or notebook rows.
- Visitors cannot write any event that affects the host aviary.
- Personality drift is monotonic toward expressive and never decays downward because of absence.
- The bird count cap of seven is enforced in data validation, adoption eligibility, UI, and simulation assumptions.

## API Surface

### Auth and account

- `POST /auth/magic-link/request`
  - Body: email.
  - Sends a 15-minute magic link. Rate-limit by keyed email lookup and IP reputation. Response is intentionally generic.

- `POST /auth/magic-link/consume`
  - Body: token.
  - Invalidates token on success, creates a session, returns app bootstrap authorization.
  - Expired, replayed, or invalid links use matter-of-fact error copy.

- `GET /account`
  - Returns email display value, sessions, settings, deletion status, export status, and invite-notification preference.

- `PATCH /account/settings`
  - Updates accessibility, audio, captions, reduced-motion, and visit-notification preferences.

- `POST /account/email-change/request` and `POST /account/email-change/confirm`
  - New email must verify before commit; old email remains active until then.

- `POST /account/export`
  - Queues a JSON export and emails a download link to the verified address.

- `POST /account/delete` and `POST /account/delete/cancel`
  - Starts or cancels the 30-day soft-delete window.

- `DELETE /sessions/{session_id}`
  - Revokes a device session.

### Aviary bootstrap and snapshots

- `GET /aviary/bootstrap`
  - Returns the initial app shell data and a compact current snapshot. This endpoint should be edge/cache assisted per account session where safe so first bird can render within 500ms.
  - If snapshot generation is briefly unavailable, the client renders the quiet field, not a spinner.

- `GET /aviary/snapshot?since_version=...`
  - Owner-only current canonical snapshot.
  - Used on visibility return, long render-frame gaps, low-frequency visible keepalive, post-resume, and after interaction batches.
  - Returns 304/no-change equivalent when possible.

- `GET /aviary/notebook?cursor=...`
  - Returns paginated read-only notebook entries. No edit/delete endpoints.

- `GET /aviary/adoption-eligibility`
  - Returns whether an age-gated new bird is available, not as a reward but as a quiet account/aviary state. This should not be rendered as gamified progress.

- `POST /aviary/adoptions`
  - Used only when an age-gated bird is actually offered. Server selects species; user supplies or confirms name. Enforce cap.

- `PATCH /birds/{bird_id}/name`
  - Rename only. No effect on identity, personality, mood, or call signature.

### Interaction events

Use one batched event endpoint rather than many direct state mutation endpoints:

- `POST /aviary/events`
  - Body: ordered batch of client events with idempotency keys.
  - Validates owner session, event schema, per-bird cooldowns for offers, and listen-in/session constraints.
  - Returns accepted/rejected event ids, current snapshot version, and any matter-of-fact validation errors.

Events:

- `session_open`: marks navigation/open and supports return-greeting computation.
- `visibility_return`: requested when a hidden tab becomes visible; used to compute absence-tier greeting.
- `presence_ping`: sent only while all presence conditions hold: visible document, focused window, recent pointer/key activity inside calibrated window.
- `presence_end`: sent when any presence condition fails or the user settles/closes where possible.
- `listen_in_start` and `listen_in_end`: bird-specific attention signals; client can start local mix ramp immediately, server records drift input.
- `offer_*`: includes offer type, intended local scene position if relevant, and candidate receiving bird if the client can infer one. Server decides canonical reaction.
- `settle` and `settle_undo`: settle quiets mood and ends presence; undo within five seconds cancels the local settled rendering and records the correction.

### Visit invitations

- `POST /visits/invitations`
  - Host enters visitor email. Creates a 30-day invite and emails a one-time link. Visits default off because no invite exists until the host creates one.

- `GET /visits/invitations`
  - Account-settings list of outstanding, active, expired, and revoked invites plus visit log.

- `DELETE /visits/invitations/{id}`
  - Revokes immediately.

- `POST /visits/consume`
  - Visitor follows invite link. Creates a scoped read-only visit session if token is valid.

- `GET /visits/{visit_session_id}/snapshot?since_version=...`
  - Visitor snapshot read. No events endpoint is available. If revoked or expired, returns a matter-of-fact "visit no longer available" surface.

## Simulation Engine Design

### Tick cadence and scheduling

Run a server-side simulation tick around once per minute per active aviary, with batching and backpressure. "Active" here means the account is not hard-deleted and has a valid aviary; the tick continues even when no clients are connected, though the worker can use coarser scheduling for long-inactive accounts as long as local-time mood continuity remains correct when they return.

Each tick:

1. Loads the aviary, birds, current snapshot version, unconsumed interaction events, recent presence windows, weather schedule, and account timezone.
2. Converts valid presence pings into bounded presence windows.
3. Applies offer cooldown logic and records canonical offer reactions.
4. Computes drift deltas from presence and interactions.
5. Transitions mood based on previous mood, time of day, ambient events, recent interactions, and personality.
6. Chooses perch-zone intents and idle-motion modes.
7. Advances or schedules weather and chorus opportunities.
8. Generates rare notebook entries if a noteworthy pattern crossed threshold.
9. Writes personality deltas, mood, snapshot, notebook, consumed-event markers, and tick metadata in one transaction.

If a tick fails, retry idempotently from the same unconsumed event range. Never partially apply personality changes without consumed-event bookkeeping.

### Presence processing

The client is responsible for detecting the three required conditions, but the server is responsible for accepting only coherent presence windows:

- Document visible.
- Window focused.
- Pointermove or keypress observed within the calibrated recent-activity window.

Presence pings include client sequence, condition flags, recent-activity age, and local timestamp. The server rejects or truncates windows with impossible durations, large clock skew, duplicated ids, revoked sessions, or hidden/unfocused flags. The exact pointer/key recency window should be calibrated during build; start with a few minutes and validate against manual watching sessions so still observation is not prematurely cut off.

Presence is the dominant drift input, but the event must remain honest. Do not count tab-open, audio-playing, or page-loaded as presence.

### Drift function

Implement drift as a slow low-pass filter over daily and weekly accumulators:

- Presence-time is the primary input for all expressive traits, especially social warmth, boldness, vocal frequency, and plumage saturation.
- Listen-in adds bird-specific weight toward social warmth and vocal frequency for the focused bird.
- Offers add small curiosity and boldness signals, with acceptance/reaction shaping magnitude.
- Settle primarily quiets current mood and closes the presence window; it should not become a trait-growth hack.
- Absence does not produce negative drift. It can lead to quieter current mood or lower greeting likelihood through fast-timescale state, but stored personality values do not move downward because the user was away.

Calibration targets:

- A typical bird with regular visits shows measurable numerical drift after about one week in instrumentation.
- A typical user notices visible/aural drift after about three weeks when comparing memory, notebook, and current behavior.
- No single session can visibly move a trait.
- Repeated offers in one session cannot saturate curiosity; enforce per-bird cooldowns of a few minutes.

Use capped deltas per tick/day/week to prevent accidental runaway. Store drift changes as server-authored deltas with before/after values in an internal audit trail, but never expose those values to the user.

### Mood transitions

Mood is a persisted fast-timescale state, not a tab-open default. Model mood as a small state machine with weighted probabilistic transitions:

- Inputs: previous mood, local time, weather, recent offer/listen-in/settle events, presence window shape, nearby bird calls, and personality vector.
- Personality biases transitions: high boldness reduces wary probability, high curiosity increases investigation of offers, high vocal frequency increases chorus participation.
- Time of day biases: morning alert/content, dusk drowsy/settled, night mostly settled except night-active species.
- Weather biases: rain temporarily dampens vocal frequency, wind can nudge alert or wary depending on bird.
- Bird-to-bird interactions can propagate wary mood, call responses, and chorus openings.

Use deterministic seeded randomness per tick/bird so behavior varies without becoming unreplayable. Persist mood at session end and continue transitioning while the user is away.

### Return-greeting

The return-greeting is a generated bird behavior, not a UI message. On `session_open` or `visibility_return`, compute a greeting plan from:

- Absence length since last owner-visible presence or session event.
- Bird boldness, social warmth, current mood, perch, and recent greeting history.
- Stagger rules if multiple birds are eligible.

The plan should choose one primary greeting bird and optionally delayed secondary responses. Greeting forms include glance, head tilt, quiet call, step toward front perch, or longer call plus response. Never render a toast, banner, modal, or "you have been away" text.

To avoid repeated greetings from refresh loops, scope greeting plans to a session/opening id and cooldown them within a short window.

### Call grammar runtime

Each bird gets a stable call signature from species motif library plus bird seed:

- Species defines motif families, pitch ranges, timbral filters, and allowed rhythmic shapes.
- Bird seed creates recognizable individual variation.
- Personality and mood modulate timing, pitch contour, envelope, and call probability.
- Vocal frequency controls unobserved call rate and chorus readiness.
- Listen-in affects mix levels, not the bird's identity or the presence of other birds.

The server includes compact call parameters in snapshots; the client synthesizes actual calls via WebAudio. Captions are generated from the same scheduled call parameters, so the text matches the sound actually played.

### Field notebook generation

Generate notebook entries from specific observation patterns, not raw event logs:

- Greeting-order changes: "pip greeted before wren today, first time this week."
- Sustained quiet: "a long stretch of quiet this morning..."
- Mood/weather combinations: rain-dampened calls, cool morning fluffed posture.
- Offer reactions that are rare for that bird.
- Bird-to-bird chorus or response moments.

Keep entries sparse: roughly one every few days for regular users, with additional entries only for genuinely notable moments. Enforce per-aviary minimum spacing and pattern diversity so the notebook never becomes a feed. Use curated naturalist prose templates with bird names, species descriptors, local daypart, and observed behavior. Do not include user-behavior summaries like visit streaks or "you returned after X days."

## Sync Model

### Canonical state

The server is the sole source of truth for the aviary. Multi-device sync is achieved by every device reading the same canonical snapshots and writing only interaction events. There is no client-to-client sync and no personality merge algorithm.

The important rule is not "resolve conflicts well"; it is "make the dangerous conflicts impossible." Clients never write personality or mood. They submit event facts. The simulation worker processes those facts once in order and writes canonical deltas.

### Event idempotency and ordering

Every interaction event carries an idempotency key and client sequence. The server:

- Deduplicates retries.
- Rejects events from revoked sessions.
- Bounds client timestamp skew.
- Orders simulation consumption by server receipt and stable tie-breakers.
- Records the last consumed event id per tick.

For listen-in duration, use start/end pairs with server timestamps as the source of truth where possible. If an end event is missing because the tab closed, close the window at session end, visibility loss, heartbeat timeout, or next tick timeout.

### Multi-device behavior

If the same user opens laptop and phone:

- Both pull the same snapshot version.
- Both can submit events.
- Listen-in is local audio focus on that device plus a server attention event; it does not need to force every other device into the same mix.
- Offers and rename/adoption events are canonical and appear on both devices after snapshot refresh.
- Personality drift from both devices' valid owner presence accumulates through the event log without either device overwriting the other.

If a device is briefly offline, it may continue rendering the last snapshot and queue a very small bounded set of owner events with idempotency keys. On reconnect, the server accepts only events whose timing and session validity are still coherent. The client must never simulate catch-up drift locally. If coherence is uncertain, drop the stale event and show a matter-of-fact sync error only in the relevant system surface.

### Visitor sync

Visitors receive read-only snapshots of the host aviary. They cannot trigger greetings, listen-in, offers, settle, notebook changes, presence windows, or drift. A visitor snapshot request checks invite/session validity every time. Revocation takes effect on the next pull and returns a matter-of-fact unavailable surface.

Visitor audio and captions are local render choices. They do not affect the host.

## Frontend Rendering Pipeline

### Scene composition

Build the aviary as one responsive horizontal scene with:

- Background sky/foliage plane.
- Middle plane for perches and birds.
- Optional foreground branch/leaf pass-through.
- Three meaningful perch zones: front, middle, back, plus implementation-specific positions within those zones.
- Thin top bar above the scene, never inside it.

The scene always fits in the viewport without panning, scrolling, or zooming. On phone, compress spacing and simplify nonessential background detail without cropping birds. On desktop, widen spacing while preserving recognizability and proximity cues.

### First paint and loading

The first visible state should be either:

- The actual aviary with birds already mid-action, if bootstrap snapshot is ready within budget.
- A quiet field with soft sky color and faint ambient cues if a snapshot takes a beat.

Never show a spinner, progress bar, "loading aviary" banner, or entry animation. The rendering runtime should be able to draw a bird from compact local assets immediately once snapshot data arrives, without waiting for settings/notebook/invite code chunks.

### Bird rendering and motion

Represent each bird as a stable identity with pose states:

- Perched idle: scan, preen, body-shuffle, head tilt, drowsy fluff.
- Transition: hop/short flight between perches.
- Offer reaction: approach, wait, drink/bathe/watch, call response.
- Greeting: glance, step forward, call, tilt.
- Chorus/call posture.

Mood changes the pose distribution rather than layering a label. Wary birds sit farther back and scan; content birds preen; curious birds tilt toward sounds/offers; drowsy birds settle low. Plumage saturation affects rendered color richness subtly over weeks.

The client interpolates from snapshot intents to smooth motion. It can create non-canonical micro-motion continuously from bird seed and mood, but canonical perch/mood/call state comes from the server.

### Reduced-motion rendering

Reduced-motion mode is a designed alternate scene runtime:

- Replace frame-by-frame micro-motion with slow cross-fades between still poses.
- Replace flight paths with cross-fades between perch poses.
- Remove ambient leaf/feather drift.
- Keep day/night color shifts, slowed.
- Keep calls, captions, notebook, mood changes, and drift.

This mode should share the same snapshot inputs and state semantics as the default renderer. It is not a separate simplified product.

### Top bar and controls

Top bar contains only:

- Account/settings.
- Accessibility settings.
- Field notebook.
- Offer affordance.
- Settle control can live with the offer/action cluster if iconography remains sparse and clear.

After a few seconds of cursor stillness, fade the bar nearly transparent. Restore on pointer movement, keyboard activity, or focus. Top-bar labels and dialogs use matter-of-fact voice for settings/system surfaces; product-facing offer/notebook text uses naturalist voice.

Do not put hover labels, mood badges, numeric stats, counters, or buttons on birds. Bird focus can show an accessibility/focus outline, but not an app-like status chip.

## Audio Pipeline

### WebAudio graph

Use WebAudio for procedural calls:

- One shared audio context per tab.
- Per-bird voice scheduler that creates oscillator/noise/filter/envelope graphs from motif parameters.
- A master ambient bus and per-bird gain nodes.
- Listen-in gain ramps: focused bird rises gradually, other birds drop to ambient but never silence.
- Chorus scheduler that allows overlapping procedural calls without loop phase artifacts.

Minimize per-call allocations. Reuse buffers/nodes where possible, and include memory instrumentation to catch 30-minute growth.

### Autoplay and fallback

Modern browser audio policy may block sound before user gesture. Handle this without announcement-style UI:

- Initialize visual scene immediately.
- Prepare audio context in suspended state if needed.
- Resume audio on the first eligible user gesture such as listen-in, offer, settings interaction, or explicit audio preference action.
- If WebAudio is unavailable or remains denied, run in graceful silence and default call captions on.
- Do not ship recorded-audio fallback. Canned recorded loops are worse than silence with captions for this product.

### Captions

Call captions are generated from the procedural call parameters at runtime:

- Text near calling bird.
- Short naturalist descriptions: low trill, soft three-note rise, single sharp call from back perch.
- Fade in/out with call.
- Respect reduced motion and contrast requirements.
- Available to users with audio off, hearing differences, noisy environments, blocked WebAudio, or explicit caption preference.

Captions should not expose motif ids, pitch numbers, or audio-engine terminology.

## Accessibility Surfaces

### Screen-reader narration

Provide a slow, naturalist narration stream from the same state as the visual scene:

- Idle cadence around every 30 to 60 seconds.
- Priority bump for return-greeting, offer reactions, settle, and other user-initiated events.
- Polite live region by default, with controls in accessibility settings to pause or adjust verbosity.
- Prose describes the aviary as a place, not a list of states.

The narration generator should use curated templates and state descriptors. Do not expose mood enum labels, perch indexes, or personality numbers.

### Keyboard model

Keyboard support:

- Tab enters top bar controls and then the aviary scene.
- Arrow keys move focus between birds when the scene has focus.
- Enter toggles listen-in on focused bird.
- Escape exits listen-in or closes the active panel.
- Offer menu and settings are fully keyboard navigable.
- Settle is reachable from the top bar.

Focus indicators must pass contrast against bright day and dim night states. Focus treatment should read as a gentle outline or glow, not a badge.

### Contrast and copy

All user-copy text passes WCAG AA at minimum. Product surfaces use naturalist voice. System surfaces use matter-of-fact voice:

- Auth failures.
- Sync/loading errors.
- Unsupported browser.
- Account/session/deletion/export settings.
- Accessibility settings.
- Visit revoked/expired surfaces.

This split must be encoded into component ownership and copy review, not left to individual implementation judgment.

## Performance and Observability

### Budgets

Budgets:

- Initial JS bundle under 2MB gzipped.
- First bird visible under 500ms on mid-tier mobile over 4G.
- 60fps idle motion on a five-year-old mid-range laptop for a 30-minute session.
- No client memory growth over 30 minutes.
- Snapshot payloads in kilobytes, not megabytes.
- Simulation tick p99 alarm above 5 seconds.

Implementation tactics:

- Code-split account settings, accessibility settings, visit invitation flow, export/delete flows, and older notebook pagination.
- Keep critical render runtime small and preloaded.
- Use compact procedural bird assets and motif definitions instead of recorded audio.
- Include current snapshot in bootstrap response when signed in.
- Avoid DOM work in animation frames.
- Reuse audio objects and bound worker/thread lifetimes.

### Observability

Allowed aggregate metrics:

- Request counts, latency, status codes.
- First-bird-render timing.
- Bundle size and route chunk sizes.
- Frame timing distributions.
- Audio-context errors and unavailable WebAudio counts.
- Simulation tick duration and event backlog size.
- Snapshot payload size.
- Auth magic-link delivery/consume success rates.
- Export/deletion job success/failure.

Disallowed telemetry:

- Per-bird personality values.
- Per-account interaction histories.
- Population-level drift analysis from real user birds.
- Leaderboard-ready counts like most visits, most birds, longest streak.
- Any analytics dimension that uses email or visitor email.

Use synthetic seeded accounts for calibration dashboards when population-level engine behavior needs inspection. Keep those clearly separated from real user simulation data.

## Rollout Plan

### Build phases

1. Foundations
   - Schema, auth, account UUID identity, sessions, magic links, encrypted email handling, soft deletion, export job skeleton, and privacy-safe logging.
   - Basic aviary/bird creation with two starter birds and stable identities.

2. Canonical simulation
   - Append-only event log, simulation worker, tick transactions, mood persistence, presence windows, drift delta storage, snapshot creation, and replay tests.
   - Internal calibration tools using synthetic accounts only.

3. Scene runtime
   - One-screen responsive aviary, bird pose system, perch zones, day/night palette, weather, quiet field loading, top-bar fade, and first-frame mid-action rendering.

4. Audio and calls
   - Species motif libraries, per-bird call seeds, WebAudio synthesis, chorus scheduler, listen-in ramps, WebAudio fallback, and generated captions.

5. Interactions
   - Return-greeting plans, listen-in, offers with cooldowns/reactions, settle/undo, rename, and age-gated adoption surfaces.

6. Field notebook and prose surfaces
   - Notebook pattern detector, sparse entry generation, read-only notebook UI, narration generator, copy-system split between naturalist and matter-of-fact surfaces.

7. Accessibility
   - Reduced-motion renderer, keyboard navigation, screen-reader live region, focus treatment, captions settings, contrast audit, unsupported-browser surfaces.

8. Visits
   - Invitation creation, email delivery, token consumption, read-only visitor snapshot, revocation, visit log, notification opt-in off by default.

9. Hardening
   - Performance budgets, memory tests, synthetic browser checks, security review, privacy review, replay/idempotency tests, load tests, and copy/design QA.

### Bird-count ramp

Launch accounts start with two birds. The engine and data model support seven, but bird availability is age-gated:

- Internal/staging: test full seven-bird load and chorus recognizability with synthetic accounts.
- Early production: only two starter birds visible for new accounts while monitoring first-bird timing, frame timing, audio errors, and tick latency.
- Age-gated production: enable third-bird availability for aviaries past the product-defined age threshold once engine/audio budgets hold.
- Later age thresholds: gradually enable fourth through seventh by account age, not engagement. Do not display progress bars or "earn more birds" framing.

### Launch gates

Do not launch V1 until:

- The first bird renders under 500ms in synthetic checks on target conditions.
- 30-minute client sessions show no memory growth.
- Simulation tick p99 is comfortably under the 5-second alarm threshold.
- Drift calibration demonstrates measurable synthetic drift after one week equivalent and visible synthetic/QA differences after three weeks equivalent.
- Screen-reader narration, captions, reduced-motion mode, and keyboard navigation are complete, not follow-up work.
- Privacy review confirms no per-bird simulation data reaches analytics or training systems.
- Copy review confirms no welcome toast/banner, gamification language, numeric personality exposure, or accidental system-tone charm on error/settings surfaces.

## Test Strategy

### Engine tests

- Property tests that clients cannot write personality or mood state.
- Replay tests for event logs producing deterministic personality/mood/snapshot results.
- Drift calibration tests for one-week measurable and three-week visible targets using synthetic accounts.
- Negative drift tests proving absence does not reduce personality traits.
- Presence validation tests for visible/focused/recent-activity conjunction and clock-skew handling.
- Offer cooldown tests preventing single-session saturation.
- Mood persistence tests across session close, local-time day/night changes, and weather.
- Bird identity tests across rename, sync, adoption, and migration.

### Sync and API tests

- Idempotent event batch retry.
- Multi-device overlapping sessions with no last-write-wins personality loss.
- Revoked session event rejection.
- Magic-link expiration and replay invalidation.
- Visitor read-only authorization and revocation on next snapshot.
- Soft deletion restore and hard deletion purge.
- Export content correctness and email-only delivery.

### Frontend tests

- First-frame no-spinner assertion.
- Responsive fit with two through seven birds on phone, tablet, desktop, and wide desktop.
- Top-bar fade and keyboard restore behavior.
- Keyboard listen-in flow and Escape behavior.
- Reduced-motion cross-fade renderer coverage.
- Audio graph memory and node reuse over 30 minutes.
- Captions generated from actual procedural call parameters.
- Matter-of-fact error copy snapshots and naturalist product copy snapshots.

### Accessibility tests

- Screen-reader narration cadence and priority events.
- WCAG AA contrast for top bar, captions, settings, notebook, and errors.
- Focus visibility in day, dusk, and night palettes.
- `prefers-reduced-motion` automatic behavior plus manual override.
- WebAudio unavailable path with captions on by default.

### Privacy and observability tests

- Log/metric scans for email, visitor email, bird names, personality vectors, and per-account event payload leakage.
- Analytics schema allowlist enforcing aggregate-only metrics.
- Synthetic-account-only calibration dashboard guardrails.
- Deletion job removes simulation, notebook, invite, export, and telemetry-linked account records within policy.

## Key Risks and Mitigations

### Drift calibration misses the emotional target

Risk: drift that is too fast makes the product feel gameable; drift that is too slow makes presence feel irrelevant.

Mitigation: build a synthetic calibration harness early, use capped low-pass deltas, review one-week and three-week synthetic histories with design/product, and keep all exact coefficients behind server-side configuration with audit trails.

### Sync bugs corrupt personality continuity

Risk: multi-device races silently overwrite drift or reset a bird, breaking the user's relationship with that bird.

Mitigation: enforce server-only personality writes at the schema/API layer, process append-only events transactionally, add replay tests, keep stable bird ids immutable, and alert on impossible vector resets.

### Audio becomes repetitive or uncanny

Risk: users hear repeated motifs as loops, or chorus mixing blurs individual bird identity before seven birds.

Mitigation: prototype call grammar before final visuals, test recognizability for two through seven birds, seed individual variation per bird, avoid recorded fallback, and generate captions from the exact call parameters so audio and text stay aligned.

### Accessibility surface becomes a flattened fallback

Risk: screen-reader, reduced-motion, or caption users get state labels rather than the actual product.

Mitigation: ship accessibility in the main build phases, use naturalist narration/caption templates, run design review on reduced-motion aesthetics, and block launch if accessibility surfaces are incomplete.

### Performance compromises the "already alive" conceit

Risk: slow bundle, snapshot, or render path forces a visible loading state.

Mitigation: keep the critical runtime small, include snapshot in bootstrap, draw quiet field instead of spinners, code-split noncritical flows, and instrument first-bird timing from day one.

### Privacy boundary erodes under analytics pressure

Risk: per-bird interaction data leaks into aggregate analysis, training, logs, or future engagement features.

Mitigation: separate simulation storage from analytics, implement metric schema allowlists, use synthetic accounts for engine dashboards, encrypt email fields, use UUIDs in logs, and require privacy review for every new metric.

### Social feature expands into a network

Risk: visit invitations grow into profiles, discovery, comments, co-presence, or notifications.

Mitigation: keep visits read-only by API design, provide no visitor event-write path, default notifications off, expose visit log only in settings, and reject public/discovery data structures in V1 schema.

### Product voice gets diluted

Risk: generic event logs, welcome banners, gamified copy, or cute error messages break the tone.

Mitigation: establish copy ownership by surface type, snapshot-test user-facing strings, encode banned terms and patterns in review checklists, and make the bird greeting the only welcome surface.

## Definition of Done for V1

V1 is complete when a new user can sign in by magic link, meet two named birds in a one-screen browser aviary, return later to a scene that has continued server-side, listen in, offer, settle, read sparse field-notebook observations, use the product with keyboard/screen reader/reduced motion/captions, invite a friend to a read-only visit, export or delete their account, and do all of this without encountering gamification, obligation, public social pressure, numeric trait exposure, or notification-style announcements.

The engineering definition of done is not just that the features exist. It is that the architecture protects the product's absences: clients cannot own personality, visitors cannot affect drift, absence cannot punish birds, telemetry cannot become behavioral analysis, and UI cannot announce what the birds are meant to notice.
