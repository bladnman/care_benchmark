# Pocket Aviary V1 Implementation Plan

## 1. Product Boundary and V1 Scope

V1 is a web-only, single-user virtual aviary with one canonical aviary per account. A new account receives two starter birds, with room to grow to seven over the lifetime of the aviary based on aviary age, not engagement or payment. The core user experience is a single horizontal browser scene that is already alive when opened: birds are mid-motion, ambient sound is present when allowed, one bird notices the returning user without any textual welcome, and the user may quietly watch, listen in on one bird, offer a seed/song fragment/still pool, settle the aviary, or open the field notebook.

V1 includes:

- Email magic-link sign-in, per-device sessions, session revocation, verified email change, account export, and 30-day soft deletion followed by hard deletion.
- A server-side simulation tick that owns canonical aviary state, personality drift, mood transitions, notebook generation, weather, bird positions, and event consumption.
- A browser client that renders snapshots, interpolates motion, synthesizes procedural calls, collects presence and interaction events, and never writes personality state.
- Two starter birds chosen by the system from a small coherent species pool, user-assigned names, stable bird identities, and future birds unlocked only by aviary age.
- Hidden per-bird personality vectors, fast-timescale moods, procedural call grammar, per-bird cooldowns for offers, bird-to-bird interaction, and monotonic drift toward expressive traits.
- Field notebook entries that are sparse, read-only, naturalist, specific, and generated from noteworthy aviary moments rather than user-engagement facts.
- Optional read-only visit invitations, off by default, revocable, expiring after 30 days, and explicitly not co-presence.
- First-class accessibility: screen-reader narration, reduced-motion rendering, call captions, keyboard navigation, visible focus, WCAG AA text contrast, and unsupported-browser surfaces.
- Aggregate-only operational telemetry and synthetic checks, with a hard boundary that per-account/per-bird interaction state is not used for analytics, ML, recommendations, or population dashboards.

V1 explicitly excludes native apps, passwords, SSO, payments, shared aviaries, multi-aviary accounts, customizable scenes, public discovery, profiles, follows, comments, chat, leaderboards, achievements, scores, levels, streaks, visit-frequency displays, hunger/death/distress mechanics, push notifications, and recorded-audio fallbacks. The plan must preserve the emotional contract: the user is noticed by birds, not announced to by software; absence is never punished; system surfaces are clear and matter-of-fact; product surfaces are naturalist and restrained.

## 2. System Architecture

Use a three-part architecture:

1. Browser client: a modern web application for the aviary scene, account/settings surfaces, accessibility settings, notebook, offers, and visits.
2. API service: authenticated HTTP/WebSocket or SSE endpoints for snapshots, event ingestion, account/session operations, invite management, exports, and settings.
3. Simulation worker: a server-side scheduled worker that advances canonical aviary state roughly once per minute, consumes interaction events in order, applies drift deltas, transitions moods, emits notebook observations, and writes new snapshots.

Persist canonical state in a relational database with row-level transactional guarantees for accounts, aviaries, birds, personality vectors, moods, notebook entries, invites, sessions, and event logs. Store generated compact snapshot documents either in the relational database or a low-latency document/cache layer, but treat the database state as authoritative. The simulation worker is the only writer of personality vectors and mood progression. The client writes interaction events only.

The client/server split is strict:

- Client owns rendering, local interpolation, procedural audio synthesis, UI interactions, presence sampling, local accessibility preferences until synced, and graceful fallback presentation.
- Server owns identity, canonical aviary state, drift, mood, adoption timing, notebook persistence, event ordering, visit permissions, privacy deletion/export, and snapshot authorization.
- Simulation worker owns all derived state that must be canonical across devices.

The render pipeline boundary is snapshot-based. The server sends small state snapshots containing bird identity, species, names, current mood, perch/position targets, current/next motion state, call scheduling hints, active weather/day-night state, offer reactions, and notebook deltas. The client interpolates between snapshots and creates non-canonical ambient ornaments such as occasional leaves and feathers. Client-generated ornaments must never affect state or telemetry.

## 3. Data Model

### Accounts and Identity

`accounts`

- `id`: synthetic UUID, primary identifier everywhere outside the encrypted email field.
- `encrypted_email`: stored only on the account record.
- `email_verified_at`
- `created_at`, `updated_at`
- `deleted_at`, `hard_delete_after`
- `timezone`: IANA timezone inferred/confirmed from client, used for local day/night and mood signals.
- `settings`: JSON or normalized table for accessibility, audio, visit notification opt-in, privacy-policy acknowledgement if needed.

Never use email as a partition key, log key, analytics dimension, event stream identifier, or URL identifier. Logs and traces use account UUIDs only, and only where operationally necessary.

`sessions`

- `id`, `account_id`, `device_label`, `created_at`, `last_seen_at`, `revoked_at`
- `token_hash`, `expires_at`
- Optional client metadata limited to operational device/browser info, not bird state.

`magic_links`

- `id`, `account_id` or pending email reference, `token_hash`, `expires_at`, `consumed_at`, `requested_at`
- 15-minute expiration, one-time consumption, rate-limited by encrypted/hashed email lookup strategy.

### Aviary and Birds

`aviaries`

- `id`, `account_id`, `created_at`
- `settled_until_reengaged`: boolean or state marker for active settle state.
- `last_tick_at`
- `local_day_phase`: derived/cache only; source is account timezone plus current time.
- `weather_state`: active weather event type, started/ends timestamps, intensity.
- `version`: monotonically increasing snapshot/version for clients.

`birds`

- `id`: stable UUID, never replaced by renames or species changes.
- `aviary_id`
- `species_id`
- `name`
- `adopted_at`
- `current_mood`: enum such as `wary`, `content`, `curious`, `drowsy`, `alert`, `settled`
- `mood_started_at`, `mood_expires_at` or transition timers
- `perch_zone`: `front`, `middle`, `back`
- `pose_state`: compact canonical motion descriptor, not a full animation frame.
- `last_offer_at_by_type` or normalized cooldown table.
- `call_signature_seed`: stable seed for procedural call identity.
- `created_at`, `updated_at`

`personality_vectors`

- `bird_id`
- `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`
- Store as normalized decimals in an internal calibrated range, with constraints and migrations protecting against null/reset.
- `updated_at`, `drift_version`

Personality values are never returned numerically to user-facing clients. Internal admin/debug tooling may exist only in secured engineering environments and must not be exposed in product UI, exports meant for general display, ARIA labels, notebook text, or visit views. Account export may include current personality vectors because the PRD names them as exportable; label the export as raw JSON and keep it in account settings, not an in-product stats surface.

### Events

`interaction_events`

- `id`, `aviary_id`, `account_id`, `actor_type`: `host` or `visitor`
- `bird_id` nullable for aviary-wide events
- `type`: `presence_ping`, `listen_in_start`, `listen_in_end`, `offer_seed`, `offer_song_fragment`, `offer_still_pool`, `settle`, `settle_undo`, `rename_bird`, `snapshot_visible`, etc.
- `payload`: bounded schema per event type
- `occurred_at_client`, `received_at_server`
- `client_event_id`: idempotency key
- `consumed_at_tick`

Visitor events are limited to operational view events needed to serve the session and must not feed presence/drift. Host interaction events feed the simulation only after server validation.

`presence_windows`

- Optional derived table created by the simulation worker from pings.
- `account_id`, `aviary_id`, `started_at`, `ended_at`, `duration_seconds`, `source_event_range`
- Used for drift computation, not exposed to users.

Presence pings are accepted only when the client reports all three conditions: document visible, window focused, and recent pointer/key activity inside the calibrated window. The server should also reject impossible ping rates and clamp duration by server receive time to avoid inflated presence from clock skew or replay.

### Notebook

`notebook_entries`

- `id`, `aviary_id`
- `entry_date`
- `text`: naturalist lowercase prose
- `source_moment_type`: internal enum such as `greeting_order`, `quiet_morning`, `weather`, `offer_reaction`, `mood_pattern`
- `source_refs`: private references to state/events used, not exposed by default.
- `created_at`

Entries are sparse: target roughly one every few days for regular use, with additional entries only for genuinely noteworthy moments. No entry may describe user visit frequency, streaks, engagement, scores, or hidden numeric traits.

### Visits

`visit_invites`

- `id`, `host_account_id`, `aviary_id`, `visitor_email_encrypted` plus lookup hash as needed
- `token_hash`, `created_at`, `expires_at`, `revoked_at`, `used_at`
- `notification_preference_snapshot` if per-invite behavior is needed.

`visit_sessions`

- `id`, `invite_id`, `host_account_id`, `started_at`, `last_snapshot_at`, `ended_at`, `revoked_at`
- Records approximate duration for host transparency log.

`visit_log_entries`

- `host_account_id`, `visitor_email_encrypted/display`, `visited_at`, `approx_duration_seconds`, `invite_id`

Visit logs are reachable from account settings on demand. They produce no badge, toast, push, email, or default notification. A host may opt into visit notifications in settings, off by default.

## 4. API Surface

Use versioned JSON APIs with typed request/response schemas. Keep snapshots compact and cacheable where possible, but always authorize by account/session/invite.

### Auth and Account APIs

- `POST /api/auth/magic-link/request`: accepts email, rate-limits, creates or finds account, sends 15-minute link. Response is matter-of-fact and does not reveal whether an email exists.
- `POST /api/auth/magic-link/consume`: consumes one-time token, issues session token, initializes starter aviary if needed.
- `GET /api/account`: returns account settings, verified email, sessions, deletion status.
- `PATCH /api/account/settings`: updates audio/accessibility/visit-notification settings.
- `POST /api/account/email-change/request` and `POST /api/account/email-change/verify`.
- `GET /api/account/sessions`, `DELETE /api/account/sessions/{session_id}`.
- `POST /api/account/export`: queues export and emails a verified-address download link.
- `POST /api/account/delete`, `POST /api/account/delete/cancel`.

All account and error responses use matter-of-fact voice.

### Snapshot APIs

- `GET /api/aviary/snapshot`: authenticated host snapshot. Returns canonical version, server time, account timezone/day phase, birds, mood descriptors, perch targets, active weather, settled state, call grammar seeds/motifs needed for client synthesis, notebook unread count if used without badges, and allowed interactions.
- `GET /api/aviary/snapshot?since_version=N`: returns delta or full snapshot if the version is stale.
- `GET /api/visits/{token}/snapshot`: read-only visitor snapshot. Does not trigger greeting, presence, offer eligibility, settle controls, or notebook interactions beyond what the host has allowed for view-only display.

Clients pull snapshots on initial load, visibility becoming visible, long render-frame gaps, resume from sleep, and low-frequency keepalive while visible. Use SSE/WebSocket only if it simplifies low-latency visit revocation and snapshot invalidation; polling is acceptable because cadence is slow and payloads are small.

### Interaction APIs

- `POST /api/aviary/events`: batch endpoint for host events with idempotency keys.
- Event types include presence pings, listen-in start/end, offer, settle, settle undo, notebook open, rename bird, and accessibility/audio setting changes if those need event logging.
- The server validates cooldowns, session ownership, bird identity, and event shape. Invalid events return matter-of-fact errors and do not mutate simulation state.

Offers are reached through the top bar, not direct bird clicking. Cooldowns are per bird and per offer class, a few minutes long, enforced server-side to protect drift calibration. The client may show disabled offer choices quietly, but avoid gamified cooldown timers; use restrained affordance states.

### Visit APIs

- `POST /api/visits/invites`: host enters visitor email; creates one-time invite, sends email. Visits are off until this action.
- `GET /api/visits/invites`: account-settings list of outstanding, expired, revoked, and used invites.
- `DELETE /api/visits/invites/{invite_id}`: revokes immediately.
- `GET /api/visits/log`: on-demand visit log with email, date, approximate duration, outstanding invites.
- Visitor snapshot responses after revocation/expiration return a matter-of-fact "visit no longer available" surface.

No API supports chat, comments, visitor interactions, co-presence, profiles, discovery, follows, public listings, or visitor-originated drift.

## 5. Simulation Engine Design

### Tick Cadence and Ownership

Run a simulation tick approximately once per minute per active/non-deleted aviary. The worker may batch accounts, but each aviary update must be transactional:

1. Lock or version-check the aviary row.
2. Read unconsumed host interaction events since the previous tick.
3. Derive presence windows and interaction summaries.
4. Apply drift deltas to personality vectors.
5. Transition moods based on current mood, personality, recent interactions, local time, weather, and bird-to-bird signals.
6. Choose perch/pose/call scheduling state for the next snapshot.
7. Potentially create sparse notebook entries for noteworthy moments.
8. Write updated canonical state, increment snapshot version, and mark events consumed.

If a tick fails, retry idempotently. Never let a client perform catch-up drift. If tick latency exceeds budget, the user may see a stale but coherent aviary, not divergent client-owned state.

### Drift Function

Represent each trait update as a low-pass filtered additive delta:

- Inputs: validated presence duration, listen-in duration per bird, accepted/attempted offers, offer types, settle terminal signal, and long-term caps.
- Dominant weight: presence time.
- Secondary weights: listen-in for the focused bird's social warmth/vocal frequency, offers for curiosity and boldness, song fragments for vocal response, still pool for curiosity/content reactions.
- Neglect behavior: no negative trait movement. Absence may reduce current expressiveness through mood and lower greeting frequency, but persisted traits do not drift downward.
- Calibration: measurable internal drift after about one week of regular visits; visible behavioral difference after about three weeks.

Implementation details:

- Use bounded trait values with asymptotic approach toward upper expressive ranges to prevent saturation.
- Weight recent presence through a decaying accumulator so a single long session cannot produce visible change.
- Apply per-day or per-week maximum deltas for each trait.
- Keep plumage saturation especially slow and monotonic.
- Store drift audit records internally for debugging/calibration, but do not expose to users or aggregate per-account state into analytics.

Testing should include simulated cohorts with known presence patterns: regular short visits, long idle but invalid background tabs, heavy offer attempts, two-week absence, multi-device overlap, and visitor sessions. Expected result: only valid host presence and host interactions produce slow positive drift; visitors and background tabs produce none.

### Mood System

Use a small enumerated mood set: `wary`, `content`, `curious`, `drowsy`, `alert`, and `settled` as the initial implementation set unless design calibration removes one. Mood is fast-timescale and persists across sessions. It resets only through daily-ish server progression, not on page open.

Mood transition inputs:

- Time of day in user's timezone: morning biases alert/curious, evening biases drowsy/settled, night settles most birds while a nightjar-like species may remain active.
- Recent interactions: accepted offers nudge content/curious, listen-in may nudge social warmth expression, settle quiets mood.
- Weather: rain dampens vocal expression, wind raises alert/wary likelihood.
- Bird personality: boldness resists wary transitions, curiosity increases offer investigation, vocal frequency increases chorus participation.
- Bird-to-bird signals: alarm/wary states can spread mildly; high vocal birds can trigger response calls or chorus.

Mood drives perch zone, idle motion, call rate, greeting probability, offer reaction, and narration/notebook prose. The UI must not display mood labels as status chips.

### Greeting Selection

On session start or return from absence, the server snapshot includes a greeting opportunity derived from absence length, mood, boldness, social warmth, and recent greeting history. The client renders it as one bird noticing the user within one to two seconds. If multiple birds qualify, stagger secondary responses by small randomized offsets. Never show a textual welcome, absence count, or "you've been gone" message.

Greeting forms include glance, head tilt, small step toward front perch, soft two-note call, longer call after long absence, or another bird's response. These are procedural combinations, not fixed canned animations.

### Call Grammar Runtime

Each species has a motif library and synthesis parameters. Each bird has a stable call seed and personality-shaped variation. The server returns enough identity and mood/call scheduling information for the client to synthesize:

- Motif selection and ordering.
- Pitch/rhythm ranges.
- Timbre/envelope parameters.
- Mood modifiers.
- Vocal-frequency-driven call probability.
- Chorus response windows.

The client uses WebAudio to synthesize calls at runtime. Never ship recorded calls. Maintain per-bird recognizability across drift by anchoring timbre/motif identity to species plus stable bird seed while allowing timing/frequency/mood variation.

## 6. Sync and Conflict Model

The server is canonical. There is no client-to-client sync and no last-write-wins path for personality or mood.

Conflict prevention rules:

- Clients submit events with idempotency keys; repeated submissions are deduplicated.
- Clients never submit absolute personality, mood, perch, or drift values.
- Simulation consumes events in server-received order with client timestamps used only as advisory within bounded skew.
- Overlapping devices both append events; the next tick applies additive deltas once.
- Snapshot versions let clients detect stale views and refresh.
- Offer cooldowns and settle state are enforced server-side.
- Rename conflicts can use last-write-wins because names are user-authored labels, not drift state; keep a simple audit history for support/export.
- Settings updates can be last-write-wins by field with `updated_at`, because they are explicit account preferences.

When a session times out, a magic link is expired/replayed, or snapshot load fails, show matter-of-fact errors. Do not wrap sync/account errors in naturalist voice.

## 7. Frontend Rendering Pipeline

### Application Shell

Use a compact web app with routes for:

- `/` or `/aviary`: authenticated aviary scene.
- `/signin` and magic-link consumption.
- `/settings/account`
- `/settings/accessibility`
- `/notebook`
- `/visit/{token}` for read-only visitor view.

The aviary scene is the first screen after auth. Avoid dashboard/landing framing. The thin top bar contains only account/settings, accessibility settings, field notebook, and offer affordance. The top bar fades nearly transparent after a few seconds of cursor stillness and returns on pointer movement or keyboard activity.

### Scene Composition

Render one horizontal scene that fits the viewport with no panning, scrolling, or zooming. Use a responsive coordinate system with three perch zones: front, middle, back. Keep every bird visible on narrow phone and wide desktop layouts. Preserve aspect ratio constraints so no bird can drift offscreen or be cropped.

Scene layers:

- Background sky/foliage with local day/night color phase.
- Middle-plane perches and birds.
- Subtle foreground branch/leaf elements.
- Client-side ambient leaf/feather drift at slow random intervals, disabled in reduced-motion mode.

The initial load path must display either the live aviary with birds mid-action or, if snapshot delivery is delayed, a quiet field with faint motion cues. No spinner, no fade-from-static, no wake-up animation, no "ready" transition.

### Bird Animation

Birds use mood/personality-shaped idle motion:

- Wary: back perch, scanning, smaller motion, delayed approach.
- Content: preening, balanced posture, soft call readiness.
- Curious: head tilts, attention to offers/sounds, front/middle perch investigation.
- Drowsy/settled: low posture, fluffed feathers, quieter calls.
- Alert: sharper glances, higher posture, wind/alarm responsiveness.

Use deterministic seeded variation per bird so behavior feels consistent without looping. Animation state may be a mix of skeletal/parametric animation and compact species assets. Avoid cycle repetition that becomes obvious during a short session. When snapshots change perch/pose state, interpolate smoothly; no teleporting unless the server state explicitly represents a scene cut after resume, and even then mask with natural motion.

### Interactions

Listen-in:

- Mouse/touch click or keyboard focus/Enter on a bird.
- Gradual mix ramp up for focused bird and down for others; never silence others.
- Disengage on same bird, another bird, empty scene, or keyboard focus exit.
- Visual focus should be subtle and accessible, not a selected-state badge.

Offer:

- Top-bar affordance opens seed, song fragment, still pool choices.
- Server validates cooldown and returns reaction opportunity.
- Reaction is rendered according to target bird mood/personality.
- Avoid direct bird-click offer mechanics.

Settle:

- Top-bar affordance triggers slow evening lighting shift and quieted calls.
- Any click in aviary within five seconds undoes.
- Closing tab without settle is normal and not surfaced as failure.

Notebook:

- Open from top-bar icon.
- Read-only, scrollable indefinitely.
- Sparse naturalist entries, no event-log formatting.

## 8. Audio Pipeline

Use WebAudio for all calls. Build an audio engine with:

- Species motif libraries encoded as compact procedural definitions.
- Per-bird stable seeds for recognizable call signatures.
- Mood/personality modifiers for pitch, rhythm, frequency, envelope, and response probability.
- A scheduler that aligns call events with server snapshot hints but can add small local variation within allowed bounds.
- A mixer with ambient mode, listen-in ramps, chorus handling, and master mute.
- Bounded reusable buffers/nodes to avoid memory growth.

Listen-in is a mix rebalance, not a solo/mute feature. Ramp times should be slow enough to feel like attention settling, likely 700-1500ms, calibrated by design. Other birds remain audible as ambient.

Song-fragment offers use a small procedural motif library, not recorded tracks. Bird responses can join, quiet, or call against depending on mood/vocal frequency.

If WebAudio is unavailable or denied, play in graceful silence and enable call captions by default. Do not include recorded audio fallback. Respect browser autoplay policies by starting audio only after the required user gesture where necessary, while keeping visual/caption experience alive before audio permission.

## 9. Accessibility Surfaces

Accessibility is part of the core product, not a later fallback.

Screen-reader narration:

- Generate prose from the same snapshot state as the visual renderer.
- Use naturalist lowercase present-tense voice.
- Idle cadence roughly every 30-60 seconds.
- Promptly narrate user-initiated meaningful moments such as return greeting, offer reaction, listen-in state, and settle.
- Avoid state lists, hidden numeric values, mood labels as data, or rapid event spam.

Reduced-motion:

- Respect `prefers-reduced-motion` and explicit accessibility setting.
- Replace micro-motion with slow cross-fades between still poses.
- Replace flights with cross-fades between perch states.
- Remove leaf/feather drift.
- Preserve day/night color shifts, calls/captions, mood, notebook, and drift.

Call captions:

- Optional in accessibility settings; default on when WebAudio fails.
- Generated from the actual procedural call grammar at runtime.
- Short naturalist phrases near the calling bird, fading in/out with the call.
- Must pass contrast requirements across day/night states.

Keyboard:

- Tab through top-bar controls.
- Tab into aviary focuses first bird.
- Arrow keys move between birds.
- Enter toggles listen-in.
- Escape exits listen-in and closes transient panels.
- Offer menu and settings are fully keyboard navigable.
- Focus indicators are visible on bright/dim backgrounds and restrained enough not to turn the scene into a UI grid.

Contrast and browser support:

- All copy surfaces meet WCAG AA.
- Unsupported browsers receive matter-of-fact guidance.
- Last two major versions of Chrome, Safari, Firefox, and Edge are supported.

## 10. Performance and Observability

Performance budgets:

- Initial JS bundle under 2MB gzipped.
- First bird visible within 500ms on a mid-tier mobile device over 4G.
- 60fps idle motion on a five-year-old mid-range laptop for a 30-minute session.
- No client memory growth over 30 minutes.
- Simulation tick p99 latency alarm at 5 seconds.

Implementation choices to hit budgets:

- Server-render or edge-embed the initial compact snapshot when possible.
- Code-split account settings, accessibility settings, notebook history, visit management, and export flows.
- Keep bird assets compact: procedural shapes/SVGs/small sprites where appropriate.
- Do not ship recorded audio.
- Reuse WebAudio nodes/buffers and animation objects.
- Suspend rendering when hidden, but refresh snapshot on visibility return.
- Use workers only where they improve frame stability and remain bounded.

Observability:

- Synthetic browser checks from common geographies measure first-bird render, snapshot latency, render frame timing, audio initialization, and reduced-motion path.
- Aggregate Real User Monitoring captures page load timing, first-bird timing, frame timing histograms, audio-context errors, API errors, and simulation tick latency.
- Server metrics include request counts, latencies, error rates, queue depth, tick duration, failed ticks, invite email delivery, magic-link consumption failures, export jobs, and deletion jobs.
- No telemetry includes per-bird state, personality values, per-account interaction history, notebook content, visitor identity as analytics dimension, or anything reconstructing a user's relationship with their aviary.

## 11. Privacy, Security, and Data Lifecycle

Privacy:

- Per-bird interactions are stored only for that account's simulation.
- Analytics pipelines must not read the simulation database.
- ML/model-training datasets must not receive per-bird or per-account interaction fields.
- Privacy policy in account settings names aggregate operational categories and explicitly excludes per-bird interaction state.

Security:

- Encrypt email fields at rest.
- Store tokens as hashes.
- Use secure, httpOnly session cookies or equivalent browser-safe token storage.
- CSRF protection for cookie-authenticated mutations.
- Rate-limit magic-link requests and event ingestion.
- Validate all event payloads with schemas.
- Do not expose internal UUIDs beyond what the client needs; visit tokens are unguessable and hashed at rest.

Deletion/export:

- Export includes birds, names, vectors, moods, notebook entries, and account settings as raw JSON delivered by emailed download link to verified address.
- Soft deletion immediately disables normal use and schedules hard deletion after 30 days.
- Recovery during the window restores the account.
- Hard deletion removes birds, vectors, notebook entries, events, sessions, invites, telemetry linkages where applicable, and export artifacts.

## 12. Rollout Plan

### Phase A: Foundations

- Define schemas and migrations for accounts, aviaries, birds, vectors, moods, events, notebook, visits, sessions.
- Implement magic-link auth, synthetic account IDs, session revocation, and starter aviary creation.
- Build snapshot endpoint and event ingestion with idempotency.
- Create simulation worker skeleton with transactional tick ownership.
- Establish telemetry boundary and privacy tests that prevent simulation tables from entering analytics jobs.

### Phase B: Core Aviary

- Implement two-bird starter scene with responsive perch zones.
- Render first snapshot with birds mid-action and quiet-field fallback.
- Implement mood-shaped idle motion, local day/night palette, subtle weather, and top-bar fade.
- Build presence detection from visibility + focus + recent pointer/key activity.
- Implement listen-in event flow and audio mix ramps.

### Phase C: Bird Engine

- Implement personality vector persistence and monotonic drift function.
- Add mood transition model, bird-to-bird interaction, greeting selection, offer reactions, and per-bird cooldowns.
- Build procedural call grammar with species motifs, bird seeds, mood/personality modifiers, and chorus scheduling.
- Add calibration harnesses for one-week measurable and three-week visible drift targets.

### Phase D: Product Surfaces

- Implement offer menu, settle/undo, field notebook generation/display, bird rename settings, account settings, accessibility settings.
- Add screen-reader narration, reduced-motion renderer, captions, keyboard navigation, and contrast verification.
- Implement account export, deletion/recovery, email change, unsupported-browser surface.

### Phase E: Visits

- Implement invite creation/email, read-only visitor snapshots, revocation, expiration, visit sessions, visit log, and optional visit notifications off by default.
- Verify visitors cannot create host drift, greetings, offers, listen-in events, notebook changes, or co-presence state.

### Phase F: Hardening and Launch

- Performance pass for 2MB bundle, 500ms first bird, 60fps idle, 30-minute memory stability.
- Accessibility QA with screen reader, keyboard-only, reduced-motion, captions, audio-denied, high-contrast conditions.
- Multi-device sync tests for overlapping sessions, stale snapshots, event replay, and suspension/resume.
- Privacy/security review of logs, traces, exports, deletion, invite tokens, and analytics boundaries.
- Closed beta with capped two-bird aviaries only, then gradual enablement of age-based third-bird availability after calibration.

Ramp birds-per-aviary carefully: launch with two, enable third-bird age offer only after drift/audio/notebook telemetry shows stable performance and support reports do not indicate confusion. Increase cap availability by aviary age cohorts while monitoring audio recognizability research, client CPU, and simulation tick cost. The hard cap remains seven.

## 13. Testing Strategy

Unit tests:

- Drift function bounds, monotonicity, decay, per-day caps, no negative neglect drift.
- Mood transition probabilities by personality/time/weather.
- Presence qualification and invalid ping rejection.
- Offer cooldown enforcement.
- Call grammar caption generation matching synthesized call parameters.
- Notebook sparsity and forbidden language filters.
- Token expiration/consumption/revocation.

Integration tests:

- New account creates two starter birds and one aviary.
- Multi-device event submission produces one canonical state.
- Client cannot mutate personality.
- Visitor cannot produce drift or interactions.
- Revoked visits terminate on next snapshot.
- Soft deletion and recovery, hard deletion job.
- Export contains required state without becoming a stats UI.

End-to-end tests:

- Sign in, first aviary render, return greeting without welcome toast.
- Listen-in by mouse and keyboard with mix ramp.
- Offer seed/song/pool and observe mood-shaped reaction.
- Settle and undo within five seconds.
- Notebook read-only browsing.
- Reduced-motion path.
- WebAudio denied path with captions on.
- Screen-reader narration cadence and priority events.
- Visit invite, visitor read-only view, revocation surface.

Performance tests:

- Bundle-size CI gate under 2MB gzipped.
- Synthetic 4G first-bird visible under 500ms.
- 30-minute memory leak test.
- 60fps idle test on representative hardware profile.
- Tick p99 and queue-depth load tests.

Product-guardrail tests:

- Static/content tests ban welcome toasts, streak language, achievement/badge/score/level terms, hunger/death/distress states, numeric trait exposure, public discovery routes, comments/chat, and recorded audio assets.
- API contract tests assert personality fields are absent from normal snapshots.
- Analytics schema tests assert no per-bird/per-account interaction fields are exported.

## 14. Key Risks and Mitigations

Drift calibration risk: drift may feel too fast, too slow, or too controllable. Mitigate with simulation harnesses, capped deltas, cohort replay tests, and staged rollout before enabling more birds.

Sync correctness risk: overlapping devices could duplicate or lose interaction effects. Mitigate with server-only personality writes, append-only idempotent events, transactional tick consumption, and versioned snapshots.

Audio uncanniness risk: procedural calls may sound artificial or repetitive. Mitigate with early audio prototyping, species motif reviews, recognizability tests, chorus stress tests, and no recorded fallback compromise.

Accessibility regression risk: accessible surfaces could become state lists or static fallbacks. Mitigate with dedicated narration/reduced-motion design review, screen-reader QA, caption grammar tests, and launch-blocking accessibility criteria.

Performance risk: 500ms first bird and 2MB bundle may conflict with visual/audio richness. Mitigate by compact assets, code splitting, edge snapshots, procedural audio, and CI gates from the start.

Tone leakage risk: system UI may use naturalist voice where clarity is needed, or product UI may add announcement/gamification surfaces. Mitigate with copy linting, design review checklists, and route/component tests for banned patterns.

Privacy boundary risk: interaction data could leak into analytics through convenient joins. Mitigate with separate storage permissions, pipeline schema tests, access reviews, and metrics definitions that use aggregate operational data only.

Notebook quality risk: entries may become generic logs or too frequent. Mitigate with source-moment selection, sparsity rules, forbidden event-log phrasing, and editorial review of generated templates.

Visit feature scope risk: visits could expand into social-network patterns. Mitigate by API absence: no comments, chat, public listing, co-presence, profiles, or visitor interaction event types.

## 15. Open Implementation Decisions

- Exact personality numeric ranges and delta constants should be chosen during calibration, with the PRD targets as acceptance criteria.
- Exact presence recent-activity window should be calibrated toward the longer side to support quiet watching.
- Final mood enum may adjust after design/audio prototyping, but must remain small and hidden from user-facing status UI.
- Rendering technology should be selected by prototype evidence against bundle and frame budgets; Canvas/WebGL/SVG hybrids are acceptable if they preserve accessibility overlays and reduced-motion mode.
- Snapshot transport can start with HTTP polling/deltas and add SSE/WebSocket only if revocation or freshness requirements need it.
- Notebook generation can begin with deterministic templates over state moments; any future generative system must obey privacy, sparsity, and voice constraints and must not train on per-account interaction histories.

These decisions are intentionally implementation-calibration items, not product-scope openings. They cannot be used to add gamification, native clients, public social surfaces, punishment for absence, numeric personality displays, or recorded audio.
