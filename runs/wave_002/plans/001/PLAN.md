# Pocket Aviary v1 Implementation Plan

## 1. Product Boundary and v1 Scope

Pocket Aviary v1 is a browser-only, single-account, single-aviary product. The core experience is one calm horizontal aviary scene with two starter birds, growing slowly over account age to a maximum of seven birds. The user can watch, listen in on one bird, offer a seed/song fragment/still pool, settle the aviary, browse a read-only field notebook, adjust account/accessibility settings, and optionally invite a named friend to view the aviary read-only.

The product must feel like a place that continues without the viewer. That requirement drives the system architecture: the server owns canonical aviary state, runs the slow simulation tick, persists bird identity/personality/mood, and exposes snapshots for clients to render. Clients render and submit interaction events; they never author personality or mood.

V1 includes:

- Email magic-link sign-in, per-device sessions, session revocation, verified email change, account export, 30-day soft deletion followed by hard deletion.
- One synthetic UUID account identifier used everywhere except the encrypted account email field.
- One canonical aviary per account.
- Two starter birds selected by the system from a small species pool of about six species.
- Bird naming at adoption and rename from bird settings.
- Hidden server-persisted personality vectors: boldness, social warmth, vocal frequency, plumage saturation, curiosity.
- Fast-timescale mood: a small server-owned enum with mood timers and transition causes.
- Server-side tick at about one minute cadence, calibrated during build.
- Append-only interaction events for presence, listen-in start/end, offer, settle, adoption, rename, and visit lifecycle.
- Procedural client-side calls with recognizable per-bird call signatures.
- Listen-in audio mix rebalancing without muting other birds.
- Offer interactions with per-bird cooldowns.
- Settle gesture with a five-second undo window.
- Field notebook with sparse, auto-generated, read-only naturalist observations.
- Optional visit invitations by email, read-only visitor sessions, revocation, expiration, visit log, and off-by-default visit notifications.
- First-class accessibility: screen-reader narration, reduced-motion rendering, call captions, keyboard navigation, visible focus indicators, WCAG AA user-copy contrast.
- Performance budgets: initial JS under 2MB gzip, first bird visible under 500ms on mid-tier mobile over 4G, 60fps idle motion on a five-year-old laptop, no client memory growth over 30 minutes, simulation tick p99 latency alarm at 5 seconds.
- Aggregate-only operational telemetry that never includes per-bird state or per-account interaction history.

V1 explicitly excludes:

- Native apps.
- Password auth, SSO, payments, subscriptions, or billing.
- Multiple aviaries, shared aviaries, household/team accounts, profiles, follows, public discovery, comments, chat, co-presence, avatars, leaderboards, rankings, public feeds.
- Gamification: achievements, streaks, scores, badges, levels, XP, "birds adopted" counters, visit calendars, or any user-facing visit-frequency surface.
- Tamagotchi mechanics: death, hunger, distress, decaying happiness, punitive neglect, caretaking chores.
- User control over bird placement or perch selection.
- Exposing personality vector numbers, even in advanced settings.
- Recorded audio fallback.

Implementation should treat these exclusions as product invariants, not postponed backlog.

## 2. Architecture Overview

Use a TypeScript-first web stack to keep schemas and domain types shared across client, API, and simulation worker while keeping the runtime boundaries strict.

Core deployable components:

1. **Web client**
   - React or equivalent component layer for top bar, settings, notebook, adoption, auth, invite, and accessibility surfaces.
   - A dedicated aviary renderer module for the scene, separate from ordinary UI components.
   - WebAudio call engine and caption generator.
   - Presence detector and event submitter.
   - Snapshot interpolator that treats server snapshots as canonical and client animation as presentation only.

2. **Edge/web app service**
   - Serves the HTML shell and critical assets.
   - Embeds or streams the smallest possible initial state bootstrap for authenticated users when available.
   - Handles unauthenticated routing, magic-link request/consume pages, unsupported-browser surface, and cache headers.

3. **API service**
   - Account/session management.
   - Snapshot reads.
   - Interaction event ingestion.
   - Notebook, settings, export, deletion, invite, visit, and revocation endpoints.
   - Matter-of-fact error surfaces and machine-readable error codes.

4. **Simulation worker**
   - Owns the server-side tick.
   - Pulls due aviaries, consumes ordered event logs, applies mood transitions and personality deltas, advances weather/day-night state, writes new canonical state versions, and emits notebook candidates.
   - Runs independently of connected clients.

5. **Email worker**
   - Sends magic links, invite links, account export links, and optional visit notifications.
   - Stores email send metadata by synthetic account ID plus email delivery ID. Do not put raw email into telemetry keys.

6. **Database**
   - PostgreSQL as the system of record for accounts, sessions, aviaries, birds, event logs, snapshots/state, notebook entries, invites, visit logs, exports, deletion markers, and settings.
   - Encryption for account email and any fields that must store invitee email.
   - Row-level constraints and transactions to enforce one aviary per account, bird cap, single writer of personality state, and event ordering.

7. **Queue/lock layer**
   - A job queue or PostgreSQL advisory-lock based scheduler for tick work. The first version can use database-backed scheduling if operational load is modest; make the abstraction swappable before scale pressure.
   - A per-aviary tick lock prevents concurrent simulation updates.

8. **Observability**
   - Aggregate request/timing/error metrics.
   - Synthetic browser checks.
   - Client RUM for load, first-bird render, frame timing, audio-context errors, and memory growth checks.
   - No per-bird or per-account interaction telemetry.

The most important boundary is this: clients can submit facts about user interaction, but only the simulation worker converts those facts into mood/personality changes. Every API, table, and test should reinforce that boundary.

## 3. Repository and Module Organization

Organize implementation around domain boundaries, not framework folders.

Recommended top-level modules:

- `app/web`: browser app, routing, chrome, settings, auth views.
- `app/api`: API handlers/controllers.
- `app/worker-sim`: simulation tick worker.
- `app/worker-email`: email jobs.
- `packages/domain`: shared TypeScript domain types, value objects, constants, schema validators, voice taxonomy, event names.
- `packages/simulation`: pure simulation functions: drift, mood transitions, greeting selection, weather/day-night derivation, notebook candidate selection. No database access.
- `packages/renderer`: scene graph, interpolator, bird pose selection, reduced-motion renderer, caption placement.
- `packages/audio`: WebAudio synthesis, call grammar runtime, listen-in mixer, silence/caption fallback.
- `packages/accessibility`: narration generation, keyboard navigation map, focus contract, live region scheduler.
- `packages/db`: migrations, repository interfaces, transaction helpers.
- `packages/privacy`: redaction helpers, telemetry guardrails, PII scanners for logs/events.
- `packages/testing`: fixtures, deterministic seeds, simulated clock, fake audio context, snapshot builders.

Keep complex work units small. Simulation formulas, rendering interpolation, audio synthesis, narration, and notebook generation should each have pure functions with deterministic inputs. The API layer should orchestrate those modules, not contain domain logic.

## 4. Data Model

### 4.1 Accounts and Sessions

`accounts`

- `id`: synthetic UUID primary key.
- `encrypted_email`: encrypted verified email.
- `email_hash`: keyed hash for uniqueness and lookup; never use raw email as identifier.
- `status`: `active | soft_deleted | hard_delete_queued`.
- `created_at`, `updated_at`.
- `email_change_pending_encrypted_email`, `email_change_token_hash`, `email_change_expires_at`.
- `soft_deleted_at`, `hard_delete_after`.

`magic_links`

- `id`: UUID.
- `email_hash`: keyed hash.
- `token_hash`: one-time token hash.
- `expires_at`: 15 minutes after creation.
- `consumed_at`.
- `requested_ip_hash`, `requested_user_agent_hash` if needed for abuse controls, not product telemetry.

`sessions`

- `id`: UUID.
- `account_id`: synthetic UUID.
- `device_label`: best-effort user-facing label.
- `token_hash`.
- `created_at`, `last_seen_at`, `revoked_at`, `expires_at`.

Session tokens are per device, revocable, and never encode personality or aviary state.

### 4.2 Aviary and Birds

`aviaries`

- `id`: UUID.
- `account_id`: unique UUID.
- `created_at`.
- `local_timezone`: latest user-confirmed or browser-provided IANA timezone for day/night interpretation.
- `bird_count_cap`: default 7; store for forward compatibility but enforce v1 cap.
- `current_state_version`: monotonically increasing integer.
- `last_tick_at`, `next_tick_due_at`.
- `settled_until_reengagement`: boolean or nullable timestamp.

`birds`

- `id`: stable UUID.
- `aviary_id`.
- `species_id`.
- `name`.
- `adopted_at`.
- `ordinal`: adoption order, not surfaced as a score/counter.
- `current_perch_zone`: `front | middle | back`.
- `current_pose_key`.
- `current_motion_phase`.
- `current_call_seed`.
- `created_at`, `updated_at`.

`bird_personality`

- `bird_id`: primary key.
- `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`: normalized decimals.
- `updated_at`.
- `version`.

These values are server-only. They are not returned numerically to clients. Snapshot APIs may expose derived render parameters such as pose choice, call motif, palette variant, and perch, but never trait numbers.

`bird_mood`

- `bird_id`: primary key.
- `mood`: `wary | content | curious | drowsy | alert | settled`.
- `intensity`: optional normalized internal scalar, not exposed directly.
- `entered_at`.
- `expires_after`.
- `last_transition_reason`: internal enum for debugging, not user copy.

### 4.3 Canonical State and Snapshots

`aviary_state_versions`

- `aviary_id`.
- `version`.
- `computed_at`.
- `day_phase`: `dawn | morning | midday | evening | night`.
- `weather`: `clear | soft_rain | soft_wind`.
- `weather_started_at`, `weather_ends_at`.
- `settled_state`: boolean.
- `snapshot_payload`: compact JSON containing render-ready state without hidden numeric personality values.

The canonical snapshot payload includes:

- Aviary time context: local day phase, settle state, weather token, ambient seed.
- Per bird: stable ID, display name, species ID, derived visual palette key, perch zone, pose/motion descriptor, mood adjective if needed for narration generation, next call schedule hints, call grammar signature ID, render seed, and transition timestamps.
- UI affordance availability: offer cooldown summaries, listen-in eligibility, settle undo deadline if active.
- Notebook unread count is not needed and should not be invented. Notebook access is user-initiated.

### 4.4 Events

`interaction_events`

- `id`: UUID.
- `aviary_id`.
- `account_id`.
- `actor_type`: `host | visitor | system`.
- `actor_session_id`: nullable UUID.
- `event_type`: enum.
- `occurred_at_client`: nullable timestamp.
- `received_at_server`.
- `sequence`: server-assigned per-aviary sequence.
- `idempotency_key`: nullable client-generated key.
- `payload`: JSON with event-specific fields.
- `consumed_by_tick_version`: nullable integer.

Host event types:

- `presence_ping`: visible + focused + recent pointer/key activity window.
- `presence_end`: settle, tab close beacon, visibility loss, focus loss timeout, session timeout.
- `listen_in_start`: target bird ID.
- `listen_in_end`: target bird ID, reason.
- `offer_made`: offer type, optional target bird if chosen by server/client affordance, local timestamp.
- `settle_start`: includes undo deadline.
- `settle_undo`.
- `bird_rename`.
- `bird_adoption_completed`.
- `settings_changed`.

Visitor event types are deliberately narrow:

- `visit_snapshot_opened`.
- `visit_snapshot_closed`.

Visitor events are used only for visit log/transparency and operational accounting. They never become drift inputs.

### 4.5 Field Notebook

`notebook_entries`

- `id`: UUID.
- `aviary_id`.
- `created_at`.
- `entry_date_local`.
- `prose`: lowercase naturalist observation.
- `source_state_version`.
- `noteworthiness_key`: e.g. `first_greeter_shift`, `quiet_morning`, `weather_reaction`, `new_bird_settling`.
- `dedupe_window_key`.

Entries are generated sparsely. Store the final prose, not just templates, so the notebook is stable when a user reads historical entries. Do not expose raw event logs as notebook entries.

### 4.6 Offers and Cooldowns

`bird_offer_cooldowns`

- `bird_id`.
- `offer_type`: `seed | song_fragment | still_pool`.
- `available_after`.
- `last_offered_at`.

Cooldowns are functional drift protection. They should block repeated trait-inflating events without scolding copy.

### 4.7 Visits

`visit_invites`

- `id`: UUID.
- `host_account_id`.
- `aviary_id`.
- `encrypted_visitor_email`.
- `visitor_email_hash`.
- `token_hash`.
- `status`: `pending | active | revoked | expired`.
- `created_at`, `expires_at`, `revoked_at`, `first_used_at`, `last_used_at`.

`visit_sessions`

- `id`: UUID.
- `invite_id`.
- `token_hash`.
- `created_at`, `last_seen_at`, `ended_at`, `revoked_at`.

`visit_log_entries`

- `id`.
- `host_account_id`.
- `invite_id`.
- `visitor_email_display_encrypted`.
- `visited_at`.
- `approx_duration_seconds`.

The visit log is reachable in account settings only. No badges, no prompts, no default notifications.

### 4.8 Settings

`account_settings`

- `account_id`.
- `call_captions_enabled`.
- `reduced_motion_override`: `system | enabled | disabled`.
- `audio_enabled`.
- `visit_notifications_enabled`: default false.
- `accessibility_narration_enabled`.
- `timezone_override`: nullable.

## 5. API Surface

All APIs use authenticated sessions except magic-link and invite-link entry points. All mutating requests accept an idempotency key. Errors on system surfaces use matter-of-fact copy and stable error codes.

### 5.1 Auth and Account

- `POST /auth/magic-link`
  - Input: email.
  - Behavior: rate-limit per email hash and request origin, create 15-minute one-time token, email link.
  - Response: generic success regardless of account existence.

- `GET /auth/magic-link/consume?token=...`
  - Behavior: validate unexpired unused token, create account if needed, create session, invalidate token.
  - Response: redirect to adoption if new account needs starter naming, otherwise aviary.

- `GET /account`
  - Returns account settings, session list, export/delete status, verified email display.

- `PATCH /account/email`
  - Starts verified email change.

- `POST /account/email/verify`
  - Commits email change after new address verification.

- `POST /account/sessions/{sessionId}/revoke`
  - Revokes a device session.

- `POST /account/export`
  - Creates export job and emails verified address a download link.

- `POST /account/delete`
  - Starts 30-day soft deletion.

- `POST /account/delete/cancel`
  - Restores during the soft-delete window.

### 5.2 Aviary Snapshot and State

- `GET /aviary/snapshot`
  - Returns current compact snapshot plus server time and state version.
  - Used on navigation, visibility return, keepalive, and long frame-gap recovery.
  - Must not return numeric personality vectors or raw event history.

- `GET /aviary/bootstrap`
  - Optional edge-optimized endpoint for first paint. Returns the smallest render-critical snapshot: scene context and enough per-bird data to draw the first bird quickly.

- `GET /aviary/notebook?cursor=...`
  - Returns paginated notebook entries.

- `GET /aviary/birds`
  - Returns bird IDs, names, species, and derived user-facing descriptors only.

- `PATCH /aviary/birds/{birdId}/name`
  - Renames a bird without changing identity/personality.

### 5.3 Interaction Events

- `POST /aviary/events`
  - Accepts a batch of interaction events. The server assigns sequence numbers, validates actor permissions, stores events append-only, and returns accepted event IDs.
  - For presence pings, the client includes evidence flags: visible, focused, recent activity timestamp, user local timezone, and monotonic clock deltas. Server validation rejects impossible cadence but does not try to infer gaze.

- `POST /aviary/listen-in/start`
  - Convenience endpoint wrapping event creation and returning updated mix instructions if needed.

- `POST /aviary/listen-in/end`
  - Records end reason.

- `POST /aviary/offers`
  - Input: offer type.
  - Server checks cooldowns and current state, records offer, returns immediate reaction seed and cooldown state. The tick later converts the accepted event into mood/drift updates.

- `POST /aviary/settle`
  - Records settle start, returns undo deadline and lighting transition instructions.

- `POST /aviary/settle/undo`
  - Valid only within five seconds.

### 5.4 Visits

- `POST /visits/invites`
  - Host enters visitor email. Creates a 30-day invite and emails one-time link.

- `GET /visits/invites`
  - Host account settings view: outstanding invites, active/revoked/expired status, visit log summaries.

- `POST /visits/invites/{inviteId}/revoke`
  - Revokes immediately.

- `GET /visit/consume?token=...`
  - Visitor entry point. Creates visitor session if token is valid, not expired, not revoked.

- `GET /visit/snapshot`
  - Visitor read-only snapshot. Returns host aviary snapshot without interaction affordances, notebook mutation, host-only settings, or personality values.

- `POST /visit/close`
  - Optional best-effort visit duration endpoint.

If a visit is revoked, expired, or unavailable, return matter-of-fact copy such as: "This visit is no longer available."

## 6. Simulation Engine Design

### 6.1 Tick Scheduling

The simulation worker runs a due-aviary loop:

1. Select aviaries where `next_tick_due_at <= now` and account is active.
2. Acquire a per-aviary lock.
3. Load current birds, personality vectors, moods, cooldowns, latest state version, unconsumed events ordered by sequence, recent notebook dedupe keys, and account settings needed for simulation.
4. Compute local time/day phase from the aviary timezone.
5. Advance weather and ambient conditions using deterministic seeded randomness.
6. Convert events into presence windows and interaction aggregates.
7. Apply mood transitions.
8. Apply personality drift deltas.
9. Choose perch zones, motion descriptors, call schedule seeds, and derived render descriptors.
10. Generate sparse notebook candidates if noteworthiness and dedupe rules pass.
11. Write the new state version, updated birds/moods/personality/cooldowns, notebook entries, and consumed event markers in one transaction.
12. Set `next_tick_due_at` to the next calibrated cadence.

The worker must be idempotent under retry. If a tick transaction fails, no event should be marked consumed. If the worker crashes after commit but before ack, retry should detect the state version and avoid double-applying events.

### 6.2 Determinism and Randomness

Use seeded PRNG streams at the simulation boundary:

- Account/aviary seed for ambient variation.
- Bird seed for stable identity-specific variation.
- State-version seed for tick-local stochastic decisions.

This allows reproducible tests and avoids repeated canned patterns. Do not use unseeded randomness deep inside simulation functions.

### 6.3 Presence Aggregation

Presence counts only when all conditions hold:

- `document.visibilityState === "visible"`.
- Window has focus.
- Pointer movement or keypress occurred within the calibrated recent-activity window.

Client sends presence pings only while all three are true. The server aggregates consecutive accepted pings into bounded presence windows with maximum gap tolerance. The activity window should start conservatively around three minutes and be calibrated with user testing toward the longer side, because watching quietly is real use.

Presence aggregation outputs:

- Presence duration since last tick.
- Longest continuous presence window.
- Whether presence ended via settle, tab close, visibility loss, focus loss, or timeout.

Only host presence feeds drift. Visitor presence never feeds drift.

### 6.4 Personality Drift

Implement drift as a slow low-pass filter over recent host presence and interactions. The calibration target:

- Instrument-measurable changes after about one week of regular visits.
- User-visible changes after about three weeks.
- No single session causes visible trait movement.

Trait deltas:

- Presence-time is dominant and nudges expressive traits upward gradually.
- Listen-in duration for a bird nudges social warmth and vocal frequency upward.
- Offers accepted or investigated nudge curiosity upward.
- Offers made near a bird nudge boldness upward slightly.
- Sustained attention can nudge plumage saturation upward.
- Settle quiets mood but does not create a directional personality delta beyond cleanly ending presence.

Drift is monotonic toward expressive. Do not implement negative drift for absence, punishment, suspicion, lower saturation, or decaying warmth. Absence can affect current mood and greeting likelihood through ambient quietness, but it must not reduce stored personality values.

Use per-trait maximums and saturating curves so high-activity users cannot max traits quickly. Include a daily/weekly cap by trait. Do not surface caps to users.

### 6.5 Mood Transitions

Mood is fast-timescale, persisted, and visible through behavior. A simple state machine is sufficient for v1 if weighted by personality and context:

- `wary`: more back-perch, scanning, lower offer approach probability.
- `content`: preening, middle/front perch, soft calls.
- `curious`: head tilts, investigates offers, responds to song fragments.
- `drowsy`: low posture, fewer calls, evening/night bias.
- `alert`: more scanning/calls, wind or morning bias.
- `settled`: quiet evening posture after settle or night conditions.

Inputs:

- Time of day: morning increases alert/content, evening increases drowsy/settled, night settles most species except nightjar-like species.
- Weather: rain dampens vocal frequency and can soften toward drowsy; wind can increase alert or wary.
- Recent interactions: accepted offer can nudge content/curious; listen-in can nudge social warmth expression; settle moves toward settled.
- Bird personality: high boldness resists wary; high curiosity increases curious; high vocal frequency increases call probability.
- Bird-to-bird interaction: alarm/wary can spread lightly; chorus can emerge when multiple vocal birds overlap.

Mood persists across sessions. Opening the tab should never reset a bird to neutral.

### 6.6 Return Greeting

Greeting selection runs from the current server state plus absence length. It chooses one bird, occasionally a staggered second response, never all birds simultaneously.

Algorithm inputs:

- Host absence duration since last presence.
- Bird boldness/social warmth derived internally.
- Current mood.
- Local time/day phase.
- Recent greeting history to avoid repeats.
- Species call/motion capabilities.

Output:

- Greeting bird ID.
- Greeting mode: glance, head tilt, short call, step toward front, longer call, call-and-response.
- Stagger offsets if another bird responds.
- Narration/caption priority event.

Do not implement textual welcome copy. The bird greeting is the welcome surface.

### 6.7 Offers

The offer system has three offer types:

- Seed: investigates/eats/waits/watches based on curiosity and mood.
- Song fragment: client plays a motif softly; bird may join, counter-call, or go quiet based on vocal frequency and mood.
- Still pool: reflective surface; birds may drink, bathe, or watch.

Offers are launched from the top bar, not by clicking birds directly. The server validates cooldowns and writes the event; the client may render the immediate reaction seed, but only the tick commits mood/drift effects.

Cooldowns:

- Per bird and offer type.
- A few minutes, calibrated to prevent a single session from saturating curiosity/boldness.
- UI should make unavailable offers quiet, not punitive.

### 6.8 Notebook Generation

Notebook generation should be rule-driven for v1, with a template library written in naturalist voice and populated by specific state facts. Avoid ML generation at launch because privacy and consistency requirements outweigh novelty.

Candidate triggers:

- First greeter changes relative to recent history.
- A rare weather reaction.
- Long quiet morning/evening.
- New bird settling in.
- Notable perch pattern shift after weeks of drift.
- Chorus event with two recognizable birds.
- A bird's offer reaction if genuinely uncommon.

Sparsity:

- Target roughly one entry every few days for regularly visited aviaries.
- Enforce dedupe windows by noteworthiness key.
- Do not create entries for every session.
- Do not write entries about the user's visit frequency.

Voice:

- Lowercase, present tense, bird-named, specific, no exclamation, no "achievement" language, no numeric traits.

## 7. Client Rendering Pipeline

### 7.1 Render Boundary

The server snapshot is canonical. The client renderer is responsible for:

- Drawing the current scene from a snapshot.
- Interpolating between snapshots.
- Scheduling visual micro-motion from render descriptors and seeds.
- Rendering reduced-motion variants.
- Routing focus/listen-in/offer/settle interactions to event submission.

The client must not:

- Advance mood or personality.
- Simulate drift.
- Decide canonical perch changes.
- Persist local divergent state as truth.

### 7.2 Scene Technology

Use a canvas/WebGL renderer for the aviary scene and standard DOM for top bar/settings/notebook. Keep the renderer dependency small enough to protect the 2MB bundle budget. If using a rendering library, tree-shake aggressively and prove its cost with bundle analysis before committing.

Scene graph:

- Background sky/foliage layer.
- Back/middle/front perch zones.
- Bird sprites or vector rigs with species silhouettes.
- Foreground branch/leaf ornament layer.
- Caption/focus overlay layer.

The aviary itself has no embedded UI buttons, labels, badges, or hover tooltips. Top bar chrome is separate and fades nearly transparent after cursor stillness.

### 7.3 Initial Load

First paint path:

1. Serve HTML shell and critical CSS.
2. Include or fetch a compact bootstrap snapshot.
3. Render quiet field immediately if snapshot is not ready.
4. Draw first bird as soon as minimal bird descriptor arrives.
5. Defer notebook/settings/invite code chunks.

There is no spinner, wake-up animation, or fade-from-static. If network delay requires a loading state, use the quiet field with faint ambient cues.

The first real aviary frame should have motion already in progress: a bird mid-preen, another calling softly, a leaf/feather drift if motion settings allow.

### 7.4 Responsive Layout

Maintain one horizontal scene across viewport sizes:

- No panning, scrolling, or zooming.
- Preserve all birds in frame.
- On narrow viewports, compress perch spacing and reduce ornamental density.
- On wide viewports, increase negative space without adding new controls or geography.
- Perch zones remain semantically front/middle/back, not x-coordinate slots the user can manipulate.

### 7.5 Motion and Reduced Motion

Default motion:

- Continuous idle micro-motion: preen, scan, head tilt, body shuffle.
- Mood-shaped pose selection.
- Gentle ambient leaves/feathers.
- Subtle parallax only.
- Smooth listen-in focus treatment and top-bar fade.

Reduced motion:

- Replace micro-motion with slow cross-fades between still poses.
- Replace flight/path transitions with cross-fades between perches.
- Remove leaf drift.
- Keep slowed color/day-night transitions.
- Keep calls/captions/notebook/mood/drift intact.

Both render paths consume the same server state.

### 7.6 Keyboard and Focus

Keyboard contract:

- Tab through top bar items.
- Tab into scene focuses the first bird.
- Arrow keys move focus between birds.
- Enter toggles listen-in on focused bird.
- Escape exits listen-in.
- Offer menu is fully keyboard navigable.
- Settle is reachable from top bar.

Focus indicators must be visible across bright/dim aviary states and should read as gentle, not game-like.

## 8. Audio Pipeline

### 8.1 Procedural Call Grammar

Each species defines:

- Motif primitives: pitch contours, durations, timbre parameters, envelope shapes.
- Variation rules: small pitch/time/timbre perturbations.
- Mood modifiers: drowsy lowers frequency/volume, alert sharpens attack, content softens contour, wary shortens calls.
- Personality modifiers: vocal frequency affects call rate and chorus joining; bird seed preserves recognizable signature.

Each bird instance receives a stable call signature derived from species plus bird seed. A user's long-term recognition of Pip's call depends on that stable identity.

### 8.2 WebAudio Runtime

Build a bounded WebAudio engine:

- One audio context per page.
- Reusable oscillators/nodes where practical.
- Bounded voice pool.
- No per-call unbounded allocation.
- Master ambient bus, per-bird bus, listen-in gain automation.
- Captions generated from the same grammar event that produces sound.

Listen-in:

- Focused bird gain rises slowly.
- Other birds lower to ambient, never silent.
- Disengage returns to ambient with the same ramp.
- No hard cuts.

### 8.3 Chorus Mixing

Chorus events are emergent from scheduled calls, not a single canned sound. When multiple bird call windows overlap:

- Preserve per-bird signature.
- Avoid phase-cancellation by procedural variation.
- Limit simultaneous voices to protect recognizability and CPU.
- Keep seven birds as hard v1 cap.

### 8.4 Fallback

If WebAudio is unavailable or blocked:

- Do not load recorded audio.
- Continue visual aviary.
- Enable call captions by default for the session.
- Show matter-of-fact accessibility/settings copy if user needs to understand why audio is silent.

## 9. Accessibility Surfaces

Accessibility ships in v1 and is tested as part of product quality.

### 9.1 Screen-Reader Narration

Narration is slow naturalist prose generated from the same state as the visual surface:

- Idle cadence around 30-60 seconds.
- Faster priority for user-initiated events: return greeting, offer reaction, settle.
- No high-frequency state spam.
- No numeric personality values.
- No "Pip mood: content" labels.

Implement a narration queue with priority and dedupe. The queue feeds an ARIA live region or equivalent screen-reader surface. Narration generation should live in a shared module with notebook voice helpers but maintain separate cadence and purpose.

### 9.2 Captions

Call captions:

- Opt-in via accessibility settings, automatic when WebAudio fallback triggers.
- Generated from procedural call grammar.
- Short naturalist descriptions near the calling bird.
- Fade in/out with the call.
- Respect reduced-motion settings for fade behavior.
- Pass WCAG AA contrast.

### 9.3 Settings and System Voice

Accessibility settings, auth, sync errors, account settings, export, delete, and unsupported browser surfaces use matter-of-fact voice. Product surfaces use naturalist voice. Maintain this split in copy review and automated string linting where possible.

### 9.4 Testing

Accessibility test matrix:

- Screen reader smoke tests for current Safari/VoiceOver and Chromium/NVDA or equivalent.
- Keyboard-only path through sign-in, adoption, aviary, listen-in, offers, settle, notebook, settings, invite.
- Reduced-motion visual regression snapshots.
- Caption contrast across day/night/weather palettes.
- Narration cadence test to prevent queue flooding.

## 10. Sync and Conflict Model

The sync model is intentionally simple: one server-owned canonical aviary, many clients reading snapshots and writing events.

Rules:

- Clients never write personality vectors, mood rows, or canonical state versions.
- Interaction events are append-only and sequence-ordered per aviary.
- Tick consumes events in sequence.
- Personality updates are additive server-authored deltas.
- No last-write-wins path exists for personality or mood.
- Multi-device clients see the same state because both read the same snapshot.

Conflict/error cases:

- Magic-link replay: token already consumed returns matter-of-fact expired/used flow.
- Session expires mid-write: event rejected with `SESSION_EXPIRED`; client prompts sign-in.
- Server outage on event submit: client can retry idempotently for a short window; do not locally apply personality/mood.
- Long suspended laptop: on resume, detect frame gap and pull snapshot before continuing rendering.
- Visit revoked during active session: next visitor snapshot returns visit unavailable surface.

Client local state should be limited to:

- Current snapshot and interpolation state.
- Pending event queue with idempotency keys.
- User settings cache.
- Listen-in UI state before server ack only where harmless; reconcile quickly.

## 11. Privacy, Security, and Data Governance

Privacy commitments are architectural rules:

- Email is encrypted on the account/invite records and looked up via keyed hashes.
- Synthetic account UUID is the only internal identifier across services, logs, queues, and telemetry.
- Per-bird interaction events are stored only for that account's simulation.
- Analytics warehouse never ingests per-bird state, personality vectors, mood histories, notebook prose, or account interaction logs.
- ML/model training receives none of this data.
- Operational telemetry is aggregate-only: request counts, latencies, error rates, anonymized session-duration histograms, render timings, audio errors, simulation tick timings.

Security measures:

- Magic links expire after 15 minutes and invalidate on use.
- Invite links expire after 30 days and can be revoked immediately.
- Session tokens are hashed at rest and revocable.
- Account deletion hard-deletes all account-tied records after 30 days.
- Exports are generated on demand, sent to verified email, and expire.
- Abuse controls for magic-link/invite endpoints without turning them into notification surfaces.

Add automated guardrails:

- Log redaction helper required by API/worker logging.
- Static tests that fail if telemetry schemas include bird IDs, personality fields, notebook prose, or raw account IDs as dimensions.
- Database permission separation: analytics readers cannot read simulation tables.

## 12. Performance and Observability

### 12.1 Budgets

Track budgets from day one:

- Initial JS under 2MB gzip.
- First bird visible under 500ms on mid-tier mobile over 4G.
- 60fps idle motion on a five-year-old laptop for 30 minutes.
- No client memory growth over 30 minutes.
- Snapshot payload in kilobytes, not megabytes.
- Simulation tick p99 alarm at 5 seconds.

### 12.2 Engineering Tactics

- Code-split account settings, accessibility settings, notebook, invite flow, export/delete, and admin-only tools.
- Keep first paint renderer minimal.
- Use compact vector/SVG/procedural assets where possible.
- Use client-side procedural audio instead of recorded assets.
- Precompute render descriptors server-side where it saves client CPU without exposing hidden values.
- Reuse WebAudio buffers/nodes.
- Bound particle/ornament counts.
- Stop rendering when hidden, but immediately refresh snapshot on visibility return.
- Run memory tests with long sessions and repeated notebook open/close.

### 12.3 Observability

Allowed metrics:

- API latency and error counts by endpoint.
- Magic-link send/consume success rates without raw email dimensions.
- Snapshot payload size distributions.
- Simulation tick duration and backlog depth.
- First-bird render timing.
- Render frame timing histograms.
- Audio context failures.
- Unsupported browser counts.
- Aggregate session-duration histograms without account dimension.

Disallowed metrics:

- Average drift by trait.
- Per-bird interaction aggregation.
- "Most listened bird species."
- Account-level behavior dashboards.
- Visit frequency displayed or analyzed as product engagement beyond anonymized operational histograms.

## 13. Rollout Plan

### Phase A: Foundations

- Set up repository modules, shared domain schemas, database migrations, auth/session model, telemetry guardrails, and privacy tests.
- Build magic-link auth, sessions, one-aviary account creation, synthetic UUID discipline, and account settings shell.
- Create deterministic simulation fixture harness before building user-visible simulation.

Exit criteria:

- Account creation produces one aviary and two starter birds.
- Email is not used as identifier outside encrypted account/invite storage and keyed lookup hashes.
- Clients can authenticate and fetch an empty/minimal snapshot.

### Phase B: Canonical Simulation

- Implement bird/personality/mood state.
- Implement event log and server-side tick.
- Implement presence aggregation.
- Implement drift and mood state machine with calibration constants.
- Implement snapshot versioning and interpolation descriptors.
- Implement no-client-writes tests for personality/mood.

Exit criteria:

- Tick advances disconnected aviaries.
- Multi-device snapshot reads match.
- Drift is measurable in test harness over simulated week and visible in derived descriptors over simulated three weeks.
- No negative drift on absence.

### Phase C: Aviary Rendering and Audio

- Build first-paint path, quiet field, horizontal scene, perch zones, bird rendering, idle micro-motion, day/night, ambient weather, top bar fade.
- Build WebAudio call grammar, per-bird signatures, listen-in mixer, and captions.
- Build reduced-motion renderer.

Exit criteria:

- First bird under 500ms in synthetic test.
- 60fps idle on target laptop class.
- Calls are procedural and captions match generated calls.
- Reduced-motion mode is a distinct designed render path.

### Phase D: Interactions and Notebook

- Implement return greeting, listen-in, offers/cooldowns, settle/undo, notebook sparse generation, bird rename.
- Add keyboard navigation across all interactions.
- Add screen-reader narration queue.

Exit criteria:

- No textual welcome surfaces exist.
- Offers cannot saturate drift in one session.
- Notebook entries are sparse, specific, and never generic event logs.
- Screen-reader narration cadence passes queue tests.

### Phase E: Account Lifecycle and Visits

- Implement account export, soft/hard deletion, session revocation, email change.
- Implement visit invites, visitor snapshot, revocation, expiration, visit log, optional notifications off by default.

Exit criteria:

- Visitors cannot submit drift-affecting events.
- Revocation terminates visitor access on next snapshot.
- Visit log is reachable on demand with no badges/prompts.
- Deletion/export satisfy privacy requirements.

### Phase F: Calibration, Beta, Launch

- Run internal dogfood with seeded test aviaries over simulated and real time.
- Tune drift, mood, offer cooldowns, narration cadence, notebook sparsity, call recognizability, and performance.
- Run accessibility audits.
- Run privacy/data-flow audit.
- Launch closed beta with two-bird aviaries only.
- Ramp age-based availability for additional birds after confidence in audio recognizability and simulation load.

Initial launch should cap all new accounts at two birds while the age-based adoption system exists but has not yet matured enough to add third birds in production. This is consistent with v1 because age, not engagement, unlocks future bird availability.

## 14. Test Strategy

### 14.1 Unit and Property Tests

- Drift monotonicity: traits never decrease due to absence.
- Drift rate: simulated regular presence meets one-week instrument and three-week visible targets.
- Presence conjunction: visible-only, focus-only, and activity-only do not count.
- Mood transitions: time/weather/personality interactions produce bounded expected states.
- Event ordering: tick consumes events in sequence.
- Offer cooldowns: repeated offers are blocked from repeated drift contribution.
- Greeting selection: one primary greeter, staggered secondary only, no all-birds-unison.
- Notebook sparsity and dedupe.
- Narration cadence and priority.
- Caption generation from call grammar.
- Visit events do not feed drift.

### 14.2 Integration Tests

- Magic-link request/consume/replay/expiry.
- Multi-device: laptop sends events, phone sees canonical post-tick state; no client merge.
- Suspended laptop resume pulls snapshot and does not continue stale state.
- Account deletion and recovery window.
- Export contains user data snapshot but not telemetry internals.
- Invite create/consume/revoke/expire.
- WebAudio unavailable yields captions-on graceful silence.

### 14.3 Performance Tests

- Bundle budget CI gate.
- First-bird render synthetic test on throttled mid-tier mobile profile.
- 30-minute idle render memory test.
- 60fps idle render benchmark.
- Simulation tick p99/load test with due-aviary backlog.
- Snapshot payload size budget.

### 14.4 Accessibility Tests

- Keyboard-only user journeys.
- Screen-reader narration smoke tests.
- Reduced-motion visual regression.
- Caption contrast across palettes.
- Top bar fade discoverability and focus behavior.

### 14.5 Product Invariant Tests

Automated string and route audits should fail builds for:

- "welcome back" copy.
- Streak/achievement/score/badge/level terminology.
- User-facing personality numbers.
- Visit notification badges.
- Textual visit-frequency surfaces.
- UI controls inside the aviary scene.

These tests will not catch every violation, but they prevent the easiest regressions.

## 15. Risks and Mitigations

### Drift Calibration

Risk: drift moves too fast and feels gameable, or too slow and feels inert.

Mitigation: build deterministic simulation harness early, tune against one-week/three-week targets, add daily/weekly caps, and run long-horizon seeded aviary tests before launch.

### Presence Inflation

Risk: presence pings overcount background/open tabs and corrupt drift.

Mitigation: enforce the three-signal conjunction in client and server validation, reject impossible ping cadence, and test background/focus/activity edge cases. Do not relax this for "engagement" metrics.

### Sync Correctness

Risk: client-local state or last-write-wins paths creep in for convenience.

Mitigation: database permissions, repository interfaces, and tests should make personality/mood writable only from the simulation worker. API event ingestion is append-only.

### Audio Uncanniness

Risk: procedural calls sound synthetic, repetitive, or blur together at higher bird counts.

Mitigation: prototype audio before visual polish, maintain per-bird signature tests with human listening sessions, cap v1 at seven, initially launch with two, and treat recorded fallback as prohibited.

### Accessibility Regression

Risk: accessible surfaces become semantic fallbacks rather than the actual product.

Mitigation: ship narration/reduced-motion/captions with core milestones, include them in design review, and test them as product surfaces, not compliance add-ons.

### Privacy Boundary Drift

Risk: operational curiosity leads to per-bird aggregation or account-level analytics.

Mitigation: telemetry schema allowlist, data warehouse access controls, privacy tests, and documentation that per-bird state is simulation-only.

### Product Tone Violations

Risk: ordinary app UI patterns introduce toasts, badges, streaks, or cheerful system copy.

Mitigation: copy taxonomy, product invariant tests, design review checklist, and explicit matter-of-fact vs naturalist surface ownership.

### Performance Budget Pressure

Risk: renderer/audio/settings dependencies push bundle over 2MB or delay first bird.

Mitigation: bundle budget CI, code splitting, minimal bootstrap renderer, asset budgets, and early performance prototypes on throttled profiles.

### Notebook Quality

Risk: notebook entries are too generic, too frequent, or accidentally describe user behavior.

Mitigation: rule-based sparse generation, dedupe windows, copy review, and tests that block user-frequency phrasing.

## 16. Key Implementation Invariants

- The server is the only writer of personality and canonical mood.
- Personality vectors are hidden forever.
- Presence requires visibility, focus, and recent pointer/key activity.
- Absence never creates negative personality drift.
- Visitors cannot affect the host's aviary.
- The bird greeting is the only welcome surface.
- The aviary scene contains no UI chrome.
- Calls are procedural; no recorded fallback.
- Accessibility surfaces preserve charm.
- Telemetry is aggregate-only and excludes per-bird/per-account relationship data.
- V1 is web-only and quiet by design.

If a future implementation shortcut violates any invariant above, it is not a shortcut; it is a product change and should be rejected for v1.
