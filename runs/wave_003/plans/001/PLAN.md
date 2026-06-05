# Pocket Aviary v1 Implementation Plan

## 1. Product Interpretation and V1 Scope

Pocket Aviary v1 is a single-user, web-only ambient aviary with two starter birds, room for up to seven birds over time, magic-link accounts, multi-device sync, optional read-only visits, accessibility surfaces, procedural calls, and a server-authored simulation. The implementation must make the aviary feel continuous, private, and alive without turning it into a game, pet-care loop, social network, or notification product.

The v1 product includes:

- Browser-only client for modern Chrome, Safari, Firefox, and Edge, last two major versions.
- Single-user account model with one canonical aviary per account.
- Email magic-link sign-in, revocable sessions, email change verification, account export, and 30-day soft deletion.
- Two starter birds per new aviary, selected by the system from a coherent small species pool.
- User-assigned and renameable bird names.
- Persistent server-side bird identity, personality vectors, moods, call signatures, perch/motion state, and drift history.
- Server-side simulation tick at roughly one-minute cadence.
- Client rendering of one horizontal scene with three perch zones, local day/night cycle, ambient weather, ambient motion, return greetings, listen-in, offers, settle, and field notebook.
- Procedural client-side WebAudio call synthesis with captions and graceful silence fallback.
- First-class screen-reader narration, reduced-motion rendering, keyboard access, and WCAG AA text contrast.
- Optional read-only visit invitations by email, off by default and revocable.
- Aggregate operational telemetry only.

The v1 product explicitly excludes:

- Native iOS or Android apps.
- Password auth, SSO, payments, billing, tiers, or subscriptions.
- Multiple aviaries per account, shared aviaries, household accounts, or team profiles.
- User-controlled scene customization, bird catalog selection, perch placement, or bird stat displays.
- Any score, streak, badge, level, XP, achievement, leaderboard, visit counter, or "birds adopted" counter.
- Tamagotchi mechanics: hunger, death, sickness, distress, negative drift, or obligation surfaces.
- Public discovery, profiles, follows, comments, chat, co-presence, shared cursors, or social feeds.
- Push notifications, default emails about aviary activity, "welcome back" text, or return announcements.
- Recorded audio fallback or looped bird audio.
- Per-account or per-bird interaction aggregation for analytics, training, recommendations, or population dashboards.

Ambiguities resolved for implementation:

- Use a TypeScript web stack end-to-end so simulation math, schema types, call grammar descriptors, and client state contracts can be shared safely without duplicating definitions.
- Render birds and scene with Canvas/WebGL through a thin custom scene layer, not DOM-per-bird animation, to control frame timing, reduced-motion variants, and mobile performance.
- Use a relational primary database for canonical account, aviary, bird, event, invite, and notebook records, with a queue-backed simulation worker for ticks.
- Treat the field notebook as server-generated from canonical state and event summaries so it syncs consistently across devices and remains sparse.

## 2. System Architecture

### 2.1 High-Level Shape

The system has four runtime surfaces:

1. Web client
   - Browser application responsible for sign-in screens, account settings, accessibility settings, top bar, scene rendering, input capture, WebAudio synthesis, captions, and screen-reader narration.
   - Reads canonical aviary snapshots from the server.
   - Writes only interaction events and presence pings.
   - Never writes personality vectors, mood state, notebook entries, bird positions, drift values, or simulation-owned fields.

2. API service
   - Authenticates sessions.
   - Serves initial HTML plus an edge-cacheable bootstrap envelope where possible.
   - Provides state snapshot, event ingestion, account, notebook, export, accessibility settings, and visit APIs.
   - Performs idempotency, validation, authorization, and rate limiting.
   - Does not compute personality drift inline except for lightweight read-model shaping.

3. Simulation service
   - Owns the canonical simulation tick.
   - Runs roughly once per minute per active or recently active aviary, and at a lower catch-up cadence for long-idle aviaries.
   - Consumes append-only interaction events in order.
   - Applies server-authored additive deltas to personality vectors.
   - Computes mood transitions, perch targets, call scheduling descriptors, ambient weather state, notebook candidates, and snapshot versions.
   - Is the only writer for simulation-owned canonical state.

4. Background jobs
   - Magic-link email delivery.
   - Account export generation and expiring download links.
   - Account hard deletion after 30 days.
   - Invite expiration and revocation cleanup.
   - Synthetic performance probes.
   - Low-priority notebook compaction/indexing if needed for long-lived accounts.

### 2.2 Deployment Topology

Use independently deployable services but keep the codebase organized as one product repository:

- `apps/web`: TypeScript browser app.
- `apps/api`: API service.
- `apps/sim`: simulation worker.
- `packages/contracts`: shared JSON schemas, TypeScript types, validation helpers, event names, mood/personality constants, and API contracts.
- `packages/simulation-core`: pure deterministic simulation functions.
- `packages/call-grammar`: procedural call motif descriptors and caption generation.
- `packages/render-scene`: scene graph primitives, bird pose descriptors, reduced-motion renderer adapters.
- `packages/voice`: naturalist and matter-of-fact copy helpers with lintable boundaries.
- `packages/observability`: aggregate metrics definitions that cannot accept per-bird or per-account simulation payloads.

Keep service entrypoints thin. The simulation math and call grammar should be pure and heavily tested; API handlers should validate, authorize, enqueue or persist, and return.

### 2.3 Client/Server Ownership Boundary

Server owns:

- Account identity and sessions.
- Aviary identity and lifecycle.
- Bird stable IDs, species, names, personality vectors, drift history, mood, current perch zone, current action, current call schedule seed, and notebook entries.
- Offer cooldown state.
- Presence accounting rollups.
- Visit invites, visit log, and visitor authorization.
- Snapshot versioning.

Client owns:

- Rendering interpolation between server snapshots.
- Local input state and focus state.
- Audio context lifecycle and actual call synthesis from server-provided call descriptors.
- Caption display for calls actually synthesized.
- Reduced-motion presentation.
- Top-bar fade behavior.
- Temporary optimistic UI for user actions that do not claim simulation results.

Client may cache:

- Last successful snapshot for quick warm paint.
- Static species visual assets and motif descriptors.
- User settings needed before authenticated API return, such as reduced-motion preference.

Client must not cache in a way that can fork canonical state. On every visibility return, long frame gap, or auth/session change, it refreshes from the server snapshot.

### 2.4 Rendering Pipeline Boundary

The server returns semantic scene state, not pixels or animation frames. A snapshot contains:

- `snapshot_version` and server timestamp.
- Local-time-derived phase descriptors: dawn, morning, midday, evening, night.
- Weather descriptor and remaining duration.
- Bird render descriptors: bird ID, species, display name, current mood, perch zone, pose family, action phase, movement target if any, call descriptor seed/window, visual drift descriptors, and accessibility prose fragments.
- Offer and settle state.
- Top-level notebook unread state only if needed for accessibility; no badges or engagement counts.

The client transforms descriptors into:

- Canvas/WebGL draw commands.
- WebAudio synthesis events.
- Call captions.
- Screen-reader narration queue entries.
- Keyboard focus targets.

This separation preserves the server as canonical while keeping high-frequency animation and audio local enough to hit the performance budget.

## 3. Data Model

### 3.1 Core Entities

Use synthetic UUIDs for all internal identifiers. Email appears only on the encrypted account record and email-delivery records that require it.

`Account`

- `id`: synthetic UUID.
- `encrypted_email`: encrypted verified email.
- `email_verified_at`.
- `created_at`, `updated_at`.
- `pending_deletion_at`, `hard_delete_after`.
- `privacy_policy_version_acknowledged`.
- `settings_id`.

`Session`

- `id`: UUID.
- `account_id`.
- `device_label`.
- `created_at`, `last_seen_at`, `revoked_at`, `expires_at`.
- `token_hash`.
- `user_agent_family` and coarse platform, avoiding precise fingerprinting.

`MagicLink`

- `id`: UUID.
- `email_hash_for_lookup`.
- `encrypted_email`.
- `token_hash`.
- `created_at`, `expires_at`, `consumed_at`.
- `request_ip_rate_bucket`.

`Aviary`

- `id`: UUID.
- `account_id`.
- `created_at`.
- `state_version`: monotonic integer.
- `last_tick_at`.
- `local_timezone`: IANA timezone selected from client signal and user setting.
- `settled_until_reengaged`: boolean or nullable state marker.
- `bird_count_cap`: default 7.

`Bird`

- `id`: UUID stable for life.
- `aviary_id`.
- `species_id`.
- `display_name`.
- `adopted_at`.
- `renamed_at`.
- `call_signature_seed`.
- `visual_seed`.
- `created_as_starter`: boolean.
- `retired_at`: reserved for future migrations only, not v1 user-visible removal.

`BirdPersonality`

- `bird_id`.
- `boldness`.
- `social_warmth`.
- `vocal_frequency`.
- `plumage_saturation`.
- `curiosity`.
- `updated_at`.
- `version`.

Traits are normalized decimals in a bounded range such as 0.0 to 1.0. Do not expose these values through user-facing APIs, settings, exports intended for casual reading, ARIA labels, or notebook prose. Account export includes vectors because the PRD requires it; mark them as raw export data and keep them out of product UI.

`BirdMoodState`

- `bird_id`.
- `mood`: enum `wary | content | curious | drowsy | alert | settled`.
- `mood_intensity`.
- `entered_at`.
- `expires_after`.
- `last_transition_reason`: internal enum, not user-facing.

`BirdSceneState`

- `bird_id`.
- `perch_zone`: `front | middle | back`.
- `perch_slot`: stable slot within zone for collision avoidance.
- `pose_family`.
- `action_state`: `preen | scan | call | shuffle | rest | investigate_offer | drink | bathe | greet | transition`.
- `action_started_at`.
- `action_seed`.
- `target_perch_zone`.
- `movement_started_at`, `movement_ends_at`.

`PresenceWindow`

- `id`.
- `aviary_id`.
- `session_id`.
- `opened_at`, `closed_at`.
- `last_qualified_presence_at`.
- `qualified_seconds`.
- `closed_by`: `hidden | blur | inactivity | settle | tab_close | session_expiry`.

`InteractionEvent`

- `id`.
- `aviary_id`.
- `account_id`.
- `session_id`.
- `bird_id`: nullable for aviary-level events.
- `event_type`: `presence_ping | listen_in_start | listen_in_end | offer_seed | offer_song_fragment | offer_still_pool | settle | reengage_after_settle | notebook_opened | accessibility_setting_changed`.
- `occurred_at_client`.
- `received_at_server`.
- `idempotency_key`.
- `payload`: typed JSON with strict schema per event.
- `processed_at_tick`.

Events are append-only. Corrections are new events, never updates that rewrite history, except for operational flags like `processed_at_tick`.

`OfferCooldown`

- `aviary_id`.
- `bird_id`.
- `offer_type`.
- `available_after`.

`NotebookEntry`

- `id`.
- `aviary_id`.
- `created_at`.
- `observed_at`.
- `entry_type`: internal enum.
- `naturalist_text`.
- `source_snapshot_version`.
- `source_event_ids`: optional internal references.
- `visible`: boolean for moderation or migration only; users cannot delete.

`AviarySnapshot`

- `aviary_id`.
- `version`.
- `generated_at`.
- `payload`: compact JSON descriptor.
- `etag`.

Store either the latest snapshot plus enough canonical state to regenerate, or a rolling short history for interpolation/debugging. Do not store indefinitely large per-minute snapshots.

### 3.2 Species and Motif Definitions

Species definitions are versioned static content:

- `species_id`.
- Display category, not rarity.
- Silhouette and pose asset references.
- Default palette.
- Plumage saturation mapping rules.
- Motif library reference.
- Perch preference priors.
- Reduced-motion pose set.
- Caption vocabulary hints.

Keep this content out of user-customizable settings. New species can be added later, but existing birds keep their `species_id`, seeds, and identity.

### 3.3 Visit Entities

`VisitInvite`

- `id`.
- `host_account_id`.
- `host_aviary_id`.
- `visitor_email_encrypted`.
- `visitor_email_hash_for_lookup`.
- `token_hash`.
- `created_at`.
- `expires_at`: 30 days.
- `accepted_at`.
- `revoked_at`.
- `last_used_at`.

`VisitSession`

- `id`.
- `invite_id`.
- `host_aviary_id`.
- `started_at`, `ended_at`.
- `last_snapshot_at`.
- `approx_duration_seconds`.
- `termination_reason`: `expired | revoked | closed | session_timeout`.

Visitor sessions never create presence windows or interaction events on the host aviary.

### 3.4 Settings

`AccountSettings`

- `account_id`.
- `reduced_motion`: `system | on | off`.
- `call_captions`: `on | off | auto_when_audio_unavailable`.
- `audio_enabled`.
- `screen_reader_narration`: `on | off | auto`.
- `visit_notifications`: off by default.
- `timezone`.
- `high_contrast_text`: optional if design requires beyond AA.

Accessibility settings use matter-of-fact copy. Changing an accessibility setting can be logged for operational support, but it must not feed bird drift.

## 4. API Surface

All APIs are JSON over HTTPS. Use schema validation at the edge and typed contracts shared with the client. Include request IDs in responses for support. Use idempotency keys for event writes and invite creation.

### 4.1 Auth and Account APIs

`POST /api/auth/magic-link`

- Body: `{ "email": string }`.
- Sends a 15-minute link if rate limits allow.
- Always returns a neutral success response to avoid email enumeration.

`POST /api/auth/magic-link/consume`

- Body: `{ "token": string }`.
- Invalidates token on successful consumption.
- Creates a session and returns a session cookie.
- Expired, consumed, or invalid tokens return matter-of-fact errors.

`GET /api/account`

- Returns account settings, session list, export/deletion state, and current verified email masked for display.

`PATCH /api/account/settings`

- Updates system settings and accessibility settings.
- Does not affect simulation drift except through future rendering behavior.

`POST /api/account/email-change`

- Begins new-email verification.

`POST /api/account/sessions/{session_id}/revoke`

- Revokes a device session.

`POST /api/account/export`

- Generates export asynchronously and emails verified address a time-limited download link.

`POST /api/account/delete`

- Marks account for deletion and sets `hard_delete_after`.

`POST /api/account/restore`

- Restores within the 30-day soft-delete window.

### 4.2 Aviary State APIs

`GET /api/aviary/bootstrap`

- Used at app start after auth.
- Returns account display settings, latest snapshot, snapshot ETag, notebook summary, active offer cooldowns, and feature flags needed to render.
- The first render path should be small enough to fit the <500ms first-bird budget.

`GET /api/aviary/snapshot?since_version=N`

- Returns `304` if unchanged or a compact snapshot if newer.
- Called on visibility return, long frame gaps, low-frequency visible keepalive, and after interaction event acknowledgement.

`GET /api/aviary/notebook?cursor=...`

- Returns paginated read-only notebook entries.
- No edit/delete endpoints.

`PATCH /api/aviary/birds/{bird_id}/name`

- Allows rename.
- Updates display name only; no effect on identity, personality, mood, or call signature.
- Uses matter-of-fact validation errors.

`GET /api/aviary/export-status/{export_id}`

- Optional status endpoint if email generation is asynchronous and the UI needs account-setting status.

### 4.3 Interaction Event APIs

`POST /api/aviary/events`

- Body: `{ "events": InteractionEventInput[] }`.
- Accepts batched presence pings and user interaction events.
- Requires per-event idempotency keys.
- Returns accepted/rejected event IDs and current snapshot version.
- Does not return drift deltas, trait values, or gamified feedback.

Event-specific payloads:

- `presence_ping`: visibility true, focus true, last activity timestamp, qualified duration slice, client monotonic clock metadata.
- `listen_in_start`: bird ID, input modality.
- `listen_in_end`: bird ID, duration, reason.
- `offer_seed`: optional target context, not direct bird command.
- `offer_song_fragment`: motif ID from small allowed library.
- `offer_still_pool`: placement descriptor generated by client within allowed scene region.
- `settle`: timestamp and current focused bird if any.
- `reengage_after_settle`: timestamp.

Presence pings must be server-validated for plausibility:

- Ignore pings from hidden or unfocused documents.
- Cap credited qualified seconds per wall-clock interval.
- Drop duplicate idempotency keys.
- Close windows after inactivity timeout.
- Treat settle and tab-close/visibility loss as terminal for the current presence window.

### 4.4 Visit APIs

`POST /api/visits/invites`

- Host-only.
- Body: `{ "email": string }`.
- Creates a 30-day one-time invite and sends email.
- Off by default; no onboarding prompt.

`GET /api/visits/invites`

- Host settings surface only.
- Returns outstanding, accepted, expired, and revoked invites plus visit log.

`POST /api/visits/invites/{invite_id}/revoke`

- Host-only.
- Revokes immediately.

`POST /api/visits/consume`

- Visitor consumes invite token.
- Creates a visit session if active and not expired/revoked.

`GET /api/visits/{visit_session_id}/snapshot?since_version=N`

- Returns host aviary snapshots with render-only permissions.
- If revoked or expired, returns a matter-of-fact "visit no longer available" response.

Visitor clients must not receive endpoints for event submission, notebook mutation, host settings, or bird renaming. If a visitor opens notebook-like information, it must be read-only and must not affect host state; v1 should avoid exposing the host notebook unless product design explicitly includes it later.

## 5. Simulation Engine Design

### 5.1 Tick Lifecycle

The simulation worker repeatedly:

1. Selects aviaries due for tick based on `last_tick_at`, recent activity, and catch-up policy.
2. Locks one aviary row or partition to prevent concurrent ticks for the same aviary.
3. Loads birds, personality vectors, mood state, scene state, offer cooldowns, recent unprocessed interaction events, local timezone, and current ambient event state.
4. Computes elapsed simulation time since last tick, bounded to avoid huge single-step jumps after outages.
5. Aggregates interaction events into calibrated inputs.
6. Applies personality drift as additive server-authored deltas.
7. Computes mood transitions.
8. Computes perch/action/call scheduling descriptors.
9. Generates sparse notebook candidates if thresholds are met.
10. Writes canonical state updates, marks events processed, increments aviary state version, and stores latest snapshot atomically.
11. Emits aggregate operational metrics only: tick duration, event count, success/failure, queue lag.

Long absence handling:

- Do not run one tick per missed minute for months of inactivity.
- Use catch-up windows that preserve day/night and mood continuity without fabricating presence.
- Personality drift during absence is based only on prior qualified presence and interactions, decaying through the low-pass filter; absence itself does not create negative deltas.

### 5.2 Determinism and Randomness

Use deterministic seeded randomness per aviary tick:

- Seed from aviary ID, snapshot version, bird ID, action family, and server timestamp bucket.
- Persist seeds needed for visible continuity.
- Do not use client randomness for canonical decisions.
- Client may use local randomness only for non-canonical ornaments such as leaf drift, bounded by reduced-motion settings.

This allows tests to replay behavior and prevents multi-device divergence.

### 5.3 Personality Drift Function

Represent each trait as a bounded scalar. Use a slow low-pass filter with weekly and three-week calibration targets.

Inputs by trait:

- Presence-time:
  - Dominant positive input across expressive traits.
  - Drives small increases in social warmth, vocal frequency, curiosity, and plumage saturation.
  - Can modestly support boldness when presence is consistent.
- Listen-in:
  - Bird-specific signal.
  - Increases social warmth and vocal frequency for the focused bird.
  - Duration-weighted, capped per day.
- Offers:
  - Seed: slight boldness and curiosity input when the bird approaches or investigates.
  - Song fragment: vocal frequency/social warmth input depending on response.
  - Still pool: curiosity/content-related input when investigated.
  - Any offer near a bird is a small boldness input, but cooldowns prevent saturation.
- Settle:
  - Ends presence cleanly and quiets mood.
  - No meaningful personality drift direction.

Rules:

- No negative drift on neglect.
- No single session produces visible personality movement.
- Clamp daily and weekly deltas.
- Calibrate so instruments can detect regular-use drift after about one week, and users can feel it after about three weeks.
- Store both current vector and internal drift accumulator values if needed, but expose neither to the product UI.

Candidate formula:

- Maintain per-trait exponentially weighted moving input `ewma_trait_signal`.
- On each tick, update EWMA from qualified event aggregates.
- Compute `delta = trait_gain * ewma_trait_signal * elapsed_days_fraction`.
- Apply diminishing returns as values approach upper bound.
- Clamp by `max_trait_delta_per_day`.
- Persist only after server tick.

Testing targets:

- A regular-presence synthetic user should cross measurable drift thresholds near day 7.
- A high-interaction single session should not cross visible thresholds.
- A two-week absence should produce no negative vector movement.
- Duplicate or replayed events should not double drift.

### 5.4 Mood System

Mood is a per-bird enum with intensity and timers. Initial v1 moods:

- `wary`
- `content`
- `curious`
- `drowsy`
- `alert`
- `settled`

Inputs:

- Time of day in account timezone.
- Recent accepted or ignored offers.
- Listen-in start/end and duration.
- Ambient weather.
- Other birds' calls and moods.
- Personality vector.
- Absence length for return-greeting selection, not as a punishment.

Transition design:

- Use weighted transition scores, not hard if/else chains, so personality can modulate behavior.
- Mood persists across sessions and changes through ticks, not tab open.
- Dusk increases drowsy/settled likelihood.
- Early morning increases alert/content likelihood.
- Rain dampens vocal frequency and nudges some birds inward or quieter.
- Wind nudges alert or wary depending on personality.
- A high-boldness bird resists wary transitions.
- A high-curiosity bird investigates offers more readily.

Avoid mood labels in UI. Mood is expressed through perch, motion, call timing, caption/narration prose, and notebook observations.

### 5.5 Return Greeting

On session return, the server snapshot includes a greeting opportunity descriptor if appropriate:

- Candidate bird selected by boldness, social warmth, current mood, recent greeting history, and absence length.
- Only one bird greets first.
- Other possible greetings are staggered with random offsets if they happen.
- Greeting form selected from pose/call/action grammar:
  - Quick glance for short absence.
  - Head tilt and low call for moderate absence.
  - Step toward front perch or longer call for long absence.
  - Warier birds may watch from back rather than approach.

The client renders the greeting without any textual welcome. Screen-reader narration may describe the observation promptly in naturalist voice.

### 5.6 Call Grammar Runtime

Each bird has:

- Species motif library.
- Stable call signature seed.
- Personality-shaped timing and pitch tendencies.
- Mood-shaped call envelope.

Server tick schedules call descriptors:

- Bird ID.
- Motif family.
- Start window.
- Pitch contour seed.
- Duration and phrase structure.
- Mood/style tags.
- Caption grammar tokens.

Client WebAudio synthesizer:

- Converts descriptors into oscillator/envelope/filter events.
- Varies every call while preserving recognizability.
- Mixes chorus events in real time.
- Supports listen-in gain ramps.
- Reuses buffers/nodes where possible to avoid memory growth.

No recorded call loops, no downloaded call files, and no recorded fallback.

### 5.7 Notebook Generation

Notebook entries are sparse naturalist observations, not event logs. Generate candidates from meaningful state changes:

- First greeter changes relative to recent pattern.
- A bird spends unusual time at a different perch zone.
- A rare chorus occurs.
- Weather intersects with a notable mood.
- A bird reacts to an offer in a way shaped by personality.
- Long quiet stretches in a regularly visited aviary.

Rules:

- Roughly one entry every few days for regular use.
- More frequent only for genuinely noteworthy moments.
- No entries about user visit frequency, streaks, or engagement.
- No numerical trait values.
- Lowercase, present-tense, specific, naturalist voice.
- Generated server-side from templates plus state-specific phrase variation, reviewed for tone.
- Store final text for consistency across devices and exports.

Use a notebook eligibility scorer:

- `noteworthiness_score`
- `minimum_gap_since_last_entry`
- `novelty_against_recent_entries`
- `privacy_check`
- `non_goal_check`

## 6. Sync Model and Conflict Prevention

### 6.1 Canonical State

The server database is the only canonical state. Clients never merge state with each other. Multi-device sync emerges from both devices reading the same snapshot and submitting append-only events.

Snapshot versioning:

- Every simulation tick that changes canonical state increments `Aviary.state_version`.
- Every event write returns the latest known version.
- Clients request `since_version` and receive either unchanged or a new snapshot.
- Clients discard local interpolation targets when server version advances incompatibly.

### 6.2 Event Ordering

Each event has:

- Server receive time.
- Client occurred time.
- Session ID.
- Idempotency key.
- Sequence number per client session if available.

Processing order:

1. Server receive order within an aviary is authoritative for mutation safety.
2. Client occurred time can refine duration calculations within plausible bounds.
3. Out-of-order or delayed events are accepted only if they fall within a bounded recent window and do not reopen closed presence windows incorrectly.

### 6.3 Conflict Cases

Two devices listening in to different birds:

- Both events can be recorded as attention events from the same account if both meet presence criteria.
- Per-session listen-in state is client-local for mix behavior.
- Drift input is capped by account-level daily/session limits to avoid multi-device amplification.

Two devices sending offers:

- Offers are events processed in order.
- Per-bird cooldown is checked by the server at event acceptance and again during tick.
- If an offer arrives during cooldown, the client receives a matter-of-fact "not available yet" response or a quiet no-op depending on UI design.

Rename from two devices:

- Bird rename is not simulation state.
- Use optimistic concurrency with bird record version.
- If conflicting, return matter-of-fact conflict and prompt reload.
- Never replace bird identity.

Session expiry mid-write:

- Event endpoint validates session before accepting.
- Accepted events remain valid even if session expires immediately after.
- Rejected events do not mutate state.

Magic-link replay:

- Consumed links cannot be consumed again.
- Existing sessions remain scoped to account and can be revoked.

Server outage:

- Client renders last known snapshot for a bounded period with matter-of-fact loading/error outside the aviary if needed.
- Do not invent drift locally.
- On recovery, refresh snapshot and reconcile by replacing canonical descriptors.

### 6.4 Presence Integrity

Presence qualifies only when all are true:

- `document.visibilityState === "visible"`.
- Window has focus.
- Pointer move or keypress occurred within calibrated recent window.

Client sends presence pings with evidence fields. Server credits bounded intervals, not raw client claims. Calibration should start with a 3-5 minute activity window, leaning longer to respect quiet watching, then adjust through qualitative testing and synthetic checks.

## 7. Frontend Rendering Pipeline

### 7.1 App Shell

Initial route flow:

1. Serve minimal HTML/CSS/JS shell.
2. Resolve auth session.
3. Fetch or receive embedded bootstrap snapshot.
4. Paint quiet field or immediate aviary snapshot.
5. Make first bird visible within 500ms on target device/network.
6. Load non-critical code-split surfaces after first bird: account settings, notebook history, visit management, export/delete flows.

Avoid spinners in aviary loading. Use quiet field with soft sky color and minimal motion cue if data is not ready.

### 7.2 Scene Composition

Layers:

- Background sky and distant foliage.
- Back perch zone.
- Middle perch zone.
- Front perch zone.
- Birds and offer objects.
- Subtle foreground branch/leaf elements.
- Caption layer.
- Top bar outside scene proper.

Scene rules:

- One horizontal scene, no panning, no scrolling, no zoom.
- Responsive compression keeps all birds visible.
- Wide screens increase spacing; narrow screens preserve bird visibility and recognizability.
- No buttons, labels, badges, hover tooltips, or overlays inside the aviary scene.
- Bird focus indicator is visible but soft and high-contrast.

### 7.3 Bird Rendering

Birds are rendered from pose families and small species assets:

- Pose families: perch idle, preen, scan, call, shuffle, drowsy, investigate, drink/bathe, greet, flight/transition.
- Personality affects:
  - Boldness: front/back perch likelihood and approach distance.
  - Social warmth: orientation toward user/other birds.
  - Vocal frequency: call pose frequency.
  - Plumage saturation: color richness and feather detail.
  - Curiosity: head tilts and offer investigation.
- Mood affects pose and timing.

Use interpolation for server-directed transitions:

- Perch changes have easing curves.
- Action phases continue smoothly across snapshot refreshes.
- If a snapshot arrives with a large discontinuity after suspension, use a short cross-fade or reposition hidden by natural motion, not teleporting.

### 7.4 Idle Micro-Motion

Default mode:

- Small weight shifts.
- Head tilts.
- Eye/blink details.
- Preening loops with procedural timing variation.
- Scanning.
- Subtle breathing.
- Occasional bird-to-bird response cues.

Never leave birds frozen in a "paused" pose while visible. Pause expensive rendering when tab hidden, but on return fetch a fresh snapshot and resume as if the aviary continued.

### 7.5 Reduced-Motion Renderer

Reduced-motion mode is a parallel presentation layer:

- Replace frame-by-frame motion with slow cross-fades between still poses.
- Replace flight paths with cross-fades between perch poses.
- Remove leaf/feather drift.
- Preserve day/evening color shifts, slowed.
- Preserve calls, captions, mood, notebook, drift, and interactions.
- Keep focus, keyboard, and listen-in available.

Architecture:

- Common scene descriptor input.
- Two renderer adapters: `MotionRenderer` and `ReducedMotionRenderer`.
- Shared layout and accessibility semantics.
- Separate visual QA baselines for both.

### 7.6 Top Bar and Interaction UI

Top bar contains only:

- Account/settings.
- Accessibility settings.
- Field notebook.
- Offer affordance.
- Settle affordance, if final visual design keeps it separate from offer/settings.

The PRD names settle as top-bar-triggered while listing the top bar icons as account/settings, accessibility, notebook, and offer. Resolve by making settle a quiet top-bar control in the sparse control group, visually subordinate and not a large CTA.

Behavior:

- Fades nearly transparent after a few seconds of cursor stillness.
- Restores on cursor movement or keyboard activity.
- Fully keyboard reachable.
- No badges for visits, streaks, notebook count, or activity.

### 7.7 Keyboard Model

- Tab cycles through top-bar controls.
- Tab enters aviary scene and focuses first bird.
- Arrow keys move focus among birds using visual/perch order.
- Enter toggles listen-in on focused bird.
- Escape exits listen-in or closes current popover.
- Offer menu is keyboard navigable.
- Settle is keyboard reachable.
- Focus indicators pass contrast across day/night states.

### 7.8 Field Notebook UI

- Opens from top bar.
- Read-only scrollable entries.
- Lowercase naturalist entries.
- Infinite or paginated history.
- No editing, deletion, reactions, comments, sharing, counters, or badges.
- Loading states use quiet matter-of-fact system copy if needed.

## 8. Audio Pipeline

### 8.1 WebAudio Architecture

Components:

- `AudioContextManager`: handles browser permission, resume/suspend, device errors.
- `CallScheduler`: receives snapshot call descriptors and schedules synthesis.
- `BirdVoice`: per-bird synthesis graph using motif descriptors.
- `ChorusMixer`: combines calls and ambient levels.
- `ListenInMixer`: gradual gain ramps for focused bird and ambient reduction for others.
- `CaptionEmitter`: receives the same grammar tokens used for synthesis.

### 8.2 Procedural Synthesis

Use a compact synthesis approach:

- Oscillators/noise sources shaped by envelopes and filters.
- Motif grammar defines phrase lengths, pitch intervals, rests, trills, and timbre.
- Species sets base timbre/motif family.
- Bird seed makes individual call signature stable.
- Mood changes envelope, spacing, and intensity.
- Personality vocal frequency changes scheduling and chorus participation.

Recognizability target:

- A user should identify Pip's call across mood changes after repeated exposure.
- Up to seven birds remain separable in the mix.
- Automated audio tests can compare motif identity while verifying variation.

### 8.3 Listen-In Mix

When listen-in starts:

- Focused bird gain rises gradually.
- Other birds drop gradually to ambient but never silence.
- Ambient scene remains audible.
- The ramp should feel like leaning attention, not switching tracks.

Disengagement:

- Same slow ramp back to ambient.
- Triggered by same bird toggle, another bird focus, empty scene click, keyboard focus away, Escape, or route change.

### 8.4 Captions

Captions are generated from the procedural grammar at runtime:

- "a soft three-note rise"
- "a low trill, paused, low trill again"
- "a single sharp call from the back perch"

Rules:

- Captions appear near the calling bird.
- Fade in/out with the call.
- Pass contrast.
- Use naturalist voice.
- Match actual synthesized call, not generic species text.
- Auto-enable when WebAudio unavailable or audio disabled if user setting allows.

### 8.5 Fallback

If WebAudio is unavailable:

- Do not load recorded audio.
- Render the aviary in graceful silence.
- Turn captions on by default for that session.
- Show matter-of-fact accessibility/audio status in settings if needed, not an aviary overlay.

## 9. Accessibility Surfaces

### 9.1 Screen-Reader Narration

Implement a narration queue fed by the same scene snapshot:

- Idle cadence: roughly every 30-60 seconds.
- User events: prompt narration for return greeting, offer reaction, settle, and listen-in changes.
- Naturalist voice, lowercase, present-tense, specific.
- No numerical stats, raw mood labels, perch indexes, or trait values.
- Avoid high-frequency updates that flood screen-reader queues.

Example generation inputs:

- Bird species, name when appropriate, mood expression, perch zone translated into prose, time of day, weather, call activity.

Example output style:

- "pip watches from the front rail, quiet between calls. wren is further back, feathers fluffed in the morning light."

Implementation:

- A `NarrationComposer` maps descriptors to prose templates and variation.
- ARIA live region uses polite updates for idle narration and a restrained priority for user-initiated events.
- Users can configure narration in accessibility settings.

### 9.2 Semantic Access

- Scene container has a concise label.
- Birds are focusable controls for listen-in, with names but not stat descriptions.
- Top-bar controls have matter-of-fact labels.
- Offer menu has clear keyboard and screen-reader structure.
- Notebook is a readable document region.
- Settings and account errors use matter-of-fact voice.

### 9.3 Reduced Motion

Use system `prefers-reduced-motion` by default, with in-product override. Reduced motion must ship at v1 launch and be included in visual regression and performance testing.

### 9.4 Contrast and Captions

- All user-copy text passes WCAG AA.
- Caption placement avoids busy visual regions when possible.
- Day/night palette has tested text contrast tokens.
- Focus outlines remain visible in morning, midday, evening, and night palettes.

### 9.5 Accessibility QA

Run:

- Keyboard-only walkthroughs for onboarding, aviary, listen-in, offer, settle, notebook, settings, visit invite, account export/delete.
- Screen-reader walkthroughs on VoiceOver/Safari and at least one Windows reader/browser pair.
- Reduced-motion visual QA.
- Caption/audio-off QA.
- Automated axe checks for account/settings/notebook surfaces.

## 10. Performance and Observability

### 10.1 Budgets

Hard budgets:

- Initial JS bundle <2MB gzipped at first paint.
- First bird visible <500ms on mid-tier mobile over 4G.
- 60fps idle motion on a five-year-old mid-range laptop.
- No client memory growth over 30 minutes.
- Simulation tick p99 latency alarm at >5 seconds.

### 10.2 Performance Strategy

Bundle:

- Code-split account settings, accessibility settings, visit management, notebook history, export/delete, and non-critical admin surfaces.
- Keep render core, bootstrap client, and minimal auth path small.
- Use procedural audio instead of recorded assets.
- Use compact vector/SVG/atlas assets for bird visuals.
- Tree-shake motif libraries and species definitions.

First bird:

- Deliver bootstrap snapshot with initial document when authenticated where feasible.
- Keep snapshot payload small.
- Draw first bird before loading non-critical textures or full notebook code.
- Use cached static species assets with versioned URLs.
- Quiet field fallback if snapshot is delayed.

Runtime:

- Use requestAnimationFrame with adaptive quality under load.
- Avoid per-frame allocations.
- Pool audio nodes where practical.
- Reuse caption elements.
- Pause rendering when hidden, but continue server simulation.
- Detect long frame gaps and refresh snapshot.

Memory:

- CI 30-minute soak test.
- Track heap snapshots in synthetic browser.
- Verify no unbounded arrays of call descriptors, captions, notebook entries, or particles.
- Bound ambient ornament pools.

### 10.3 Observability Boundary

Allowed aggregate metrics:

- Request counts and latency.
- Auth success/error counts.
- Snapshot latency and payload size.
- Event ingestion counts by coarse type, without account/bird dimensions in analytics warehouse.
- Simulation tick duration, queue lag, and failure counts.
- First-bird-render timings.
- Render frame timings.
- Audio context errors.
- WebAudio unavailable counts.
- Session-duration histograms anonymized and not joinable to account.

Disallowed:

- Per-bird state in telemetry.
- Per-account interaction history in analytics.
- Population dashboards of average drift, bird traits, offer behavior, or individual bird outcomes.
- ML training datasets using per-bird interaction data.

Implementation guard:

- Define metrics APIs that accept only approved aggregate fields.
- Keep simulation database inaccessible from analytics jobs.
- Add static review checks or schema allowlists for telemetry events.
- Include privacy review as a release gate.

## 11. Rollout Plan

### 11.1 Build Phases

Phase A: Foundations

- Repository structure and shared contracts.
- Account/auth/session model.
- Synthetic UUID enforcement.
- Basic aviary, bird, personality, mood, and event schema.
- Simulation worker skeleton.
- Snapshot API.
- Minimal web shell with quiet field and static test bird.

Phase B: Canonical Simulation

- Server tick locking and versioning.
- Presence event ingestion and qualification.
- Personality drift EWMA and clamping.
- Mood transitions.
- Perch/action state.
- Snapshot generation.
- Deterministic replay tests.

Phase C: Rendering and Audio

- Responsive horizontal scene.
- Three perch zones.
- Bird pose/action renderer.
- Return greeting.
- Ambient day/night, weather, leaf/feather ornaments.
- WebAudio call grammar.
- Listen-in mixer.
- Captions.
- Reduced-motion renderer.

Phase D: Product Interactions

- Offer flow and cooldowns.
- Settle and undo.
- Field notebook generation and UI.
- Bird rename.
- Top bar fade.
- Accessibility settings.

Phase E: Accounts, Privacy, and Visits

- Account settings, session revocation, email change.
- Export and deletion flows.
- Visit invite, consume, read-only snapshot, revoke, expiration, visit log.
- Matter-of-fact error surfaces.
- Privacy policy link and telemetry boundary enforcement.

Phase F: Hardening

- Performance budgets and synthetic probes.
- Accessibility QA.
- Cross-browser testing.
- Long-running memory tests.
- Simulation calibration with synthetic personas.
- Security/privacy review.
- Copy/tone review for naturalist vs system surfaces.

### 11.2 Bird Count Ramp

Launch:

- Every new account starts with two birds.
- Cap seven in data model and service validation from day one.
- Do not show "2/7" counters.

Post-launch ramp:

- Third bird eligibility based on aviary age, not visit count.
- Keep new-bird offers quiet and naturalist.
- Do not frame as reward, unlock, achievement, or milestone.
- Start with internal/beta eligibility disabled behind a server flag.
- Enable third-bird age offer only after call recognizability and scene density are validated.

### 11.3 Release Strategy

- Internal dogfood with resettable test accounts, clearly separated from production.
- Private beta with limited invites, no public discovery.
- Gradual production ramp by account creation rate.
- Feature flags for:
  - Weather frequency.
  - Notebook generation.
  - Visit invites.
  - Third-bird eligibility.
  - Reduced-motion renderer fallback kill switch only to safer reduced presentation, not to inaccessible default.

Do not feature-flag in gamification counters or notification loops.

### 11.4 Day-One Instrumentation

Ship aggregate monitoring from day one:

- First-bird-render p50/p95/p99.
- Bundle size per route.
- Snapshot API latency and error rate.
- Simulation tick p50/p95/p99 and lag.
- Client frame timing aggregate.
- Audio context error counts.
- Memory soak CI.
- Auth magic-link success/error counts.
- Visit invite delivery/error counts.

No per-bird or per-account behavioral dashboards.

## 12. Testing Strategy

### 12.1 Simulation Tests

Unit tests:

- Drift clamps and monotonic expressive movement.
- No negative drift from absence.
- Single-session high activity not visibly moving traits.
- Weekly measurable drift target under regular presence.
- Three-week visible descriptor changes under regular presence.
- Mood transition weighting.
- Offer cooldown enforcement.
- Event idempotency.
- Deterministic seeded replay.

Integration tests:

- Multi-device event ingestion without personality overwrite.
- Server tick processes events in order.
- Snapshot version increments.
- Long absence catch-up.
- Magic-link session plus aviary bootstrap.
- Account deletion removes simulation data after hard-delete window.

### 12.2 Client Tests

- Snapshot rendering contract tests.
- Visibility return refresh.
- Long frame gap refresh.
- Listen-in gain ramp timing.
- Caption text matches call descriptor.
- Reduced-motion renderer receives same scene descriptors.
- Keyboard focus order.
- Top-bar fade and restore.
- No spinner in aviary loading path.

### 12.3 End-to-End Tests

Flows:

- New account magic-link sign-in, starter birds, first aviary render.
- Return greeting after simulated absence.
- Qualified presence accumulation.
- Listen-in start/end.
- Offer seed/song/pool with cooldown.
- Settle and undo.
- Notebook entry appears after simulated noteworthy event.
- Account export.
- Account soft delete and restore.
- Visit invite, visitor read-only render, revocation.
- WebAudio unavailable -> captions and silence.

### 12.4 Accessibility Tests

- Screen-reader narration cadence and content.
- Keyboard-only complete session.
- Reduced-motion complete session.
- Captions with audio off.
- WCAG AA checks across palettes.

### 12.5 Performance Tests

- Bundle budget CI gate.
- First-bird synthetic check on throttled mid-tier mobile profile.
- 30-minute memory soak.
- 60fps idle on benchmark laptop profile.
- Simulation tick load test with queued events.

## 13. Security and Privacy Plan

### 13.1 PII Handling

- Email encrypted on account record.
- Synthetic UUID everywhere else.
- Email hashes only for lookup/rate-limit where needed.
- No email in logs, metrics, queue names, shard keys, or trace attributes.
- Add automated log scrubber tests and code review checklist.

### 13.2 Session Security

- Store session token hashes only.
- HttpOnly, Secure, SameSite cookies.
- Revocable per-device sessions.
- Magic links expire after 15 minutes and invalidate on consume.
- Rate-limit magic-link requests by email hash and IP bucket.

### 13.3 Visit Security

- Invite tokens hashed at rest.
- One named visitor email per invite.
- Expire unused invites after 30 days.
- Revoke active invites immediately.
- Visitor snapshot endpoint checks invite/session validity on every request.
- Visitors cannot write events.

### 13.4 Deletion and Export

- Soft delete immediately blocks normal use while allowing restore.
- Hard delete job removes account, aviary, birds, vectors, moods, events, notebook, invites, sessions, exports, and telemetry join keys after 30 days.
- Export generated on demand and emailed as a time-limited link to verified email.
- Export access audited at aggregate operational level only.

## 14. Voice and Copy Governance

Create two explicit copy modes:

Naturalist voice:

- Aviary narration.
- Notebook.
- Captions.
- Offer prompts and bird observations.
- Lowercase by default, present-tense, specific, no exclamation, no "you" announcement framing.

Matter-of-fact voice:

- Sign-in.
- Account settings.
- Sync errors.
- Unsupported browser.
- Accessibility settings.
- Visit revocation/expiration.
- Export/delete flows.

Implementation:

- `packages/voice` exposes separate helpers/modules for naturalist and system copy.
- Copy review checklist blocks "welcome back", streaks, achievements, stats, and user-behavior observations.
- UI lint or snapshot tests search for banned phrases: "welcome back", "streak", "achievement", "level", "score", "XP", "badge", "days visited", "leaderboard".

## 15. Key Risks and Mitigations

Drift calibration too fast:

- Risk: birds feel gameable or visibly change session-by-session.
- Mitigation: synthetic persona tests, daily/weekly clamps, beta calibration, no UI trait values.

Drift calibration too slow:

- Risk: users feel nothing changes.
- Mitigation: instrument-only thresholds after one week, qualitative review after three-week simulations, notebook observations that notice real but sparse changes without exposing numbers.

Presence inflation:

- Risk: background tabs corrupt drift.
- Mitigation: strict visibility + focus + recent input conjunction, server plausibility caps, tests for hidden/minimized/background cases.

Sync correctness:

- Risk: multi-device overwrites or duplicate events corrupt personality.
- Mitigation: server-only personality writes, append-only events, idempotency keys, row locks, ordered tick processing, no last-write-wins for vectors.

Audio uncanniness:

- Risk: procedural calls feel synthetic or repetitive.
- Mitigation: motif grammar iteration, per-bird seeds, audio QA, variation tests, no loops, recognizability testing at seven-bird cap.

Performance misses first-bird budget:

- Risk: loading feels like an app waking up.
- Mitigation: <2MB gate, embedded bootstrap snapshot, quiet field fallback, aggressive code splitting, render first bird before non-critical surfaces.

Accessibility flattening:

- Risk: accessible modes expose state lists instead of the product's charm.
- Mitigation: designed narration, reduced-motion renderer, captions from grammar, accessibility QA from day one.

Tone drift:

- Risk: toasts, badges, welcome text, or system-y notebook entries creep in.
- Mitigation: copy governance, banned phrase tests, design review, no generic event-log notebook architecture.

Privacy boundary erosion:

- Risk: per-bird interaction data leaks into analytics.
- Mitigation: physical/service boundary between simulation DB and analytics, metrics schema allowlist, privacy review release gate.

Social feature expansion:

- Risk: visits become co-presence or discovery.
- Mitigation: visitor read-only authorization, no visitor events, no public surfaces, visit notifications off by default.

Notebook overproduction:

- Risk: notebook becomes a feed.
- Mitigation: minimum gaps, novelty scoring, sparse generation tests, no per-session entry guarantee.

Bird identity migration:

- Risk: species or rendering changes reset perceived bird continuity.
- Mitigation: stable bird IDs, persisted seeds, migration tests, no regeneration of personality or call signature.

## 16. Engineering Work Breakdown

Team workstreams:

1. Platform and data
   - Database schema, migrations, account/session/auth, event log, snapshot storage, deletion/export jobs.

2. Simulation
   - Pure simulation core, tick worker, drift calibration, mood transitions, notebook eligibility, deterministic replay.

3. Client scene
   - Render pipeline, responsive layout, bird poses, day/night/weather, interactions, reduced-motion.

4. Audio
   - Call grammar, WebAudio synthesis, chorus mixer, listen-in ramp, captions, fallback.

5. Accessibility and copy
   - Narration composer, keyboard model, captions, WCAG validation, naturalist/system voice governance.

6. Social and account settings
   - Visit invites, revocation, visit log, settings, export/delete, session revocation.

7. Observability and QA
   - Aggregate metrics, synthetic probes, performance CI, privacy checks, E2E suite.

Critical dependencies:

- Contracts and schema before client/simulation parallelize deeply.
- Simulation snapshot format before final rendering/audio integration.
- Voice rules before notebook/narration/caption content expands.
- Performance budget checks before adding visit/settings/notebook route code to main bundle.

## 17. Definition of Done for V1

V1 is ready when:

- A new user can sign in by magic link, receive two starter birds, name them, and see the first bird within the target budget.
- The aviary opens already in motion, with no spinner or welcome text.
- Qualified presence, listen-in, offers, and settle are recorded as events and processed by server tick.
- Personality vectors drift only through server-authored monotonic expressive deltas.
- Mood persists across sessions and changes through server tick.
- Multi-device sessions show one canonical aviary without merge conflicts.
- Procedural calls are synthesized client-side, varied, captioned, and mixed through listen-in ramps.
- Reduced-motion, screen-reader narration, captions, keyboard navigation, and AA text contrast ship with v1.
- Notebook entries are sparse, specific, read-only, and naturalist.
- Visit invites are off by default, read-only, revocable, expiring, and do not affect host drift.
- Account export, deletion, session revocation, and email change work.
- Aggregate-only observability is in place without per-bird/per-account interaction analytics.
- Performance, accessibility, privacy, and simulation calibration gates pass.
- No non-goal surfaces exist in product code, copy, API, telemetry, or settings.
