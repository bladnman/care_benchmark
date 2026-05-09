# Pocket Aviary v1 Implementation Plan

## 1. Product Frame And Non-Negotiables

Pocket Aviary v1 is a web-only, single-user virtual aviary built around a slow relationship with a small number of birds. The implementation must optimize for felt continuity: the aviary appears to have been running before the user arrived, birds react without announcement, personality changes accrue over weeks, and every accessible surface preserves the same quiet naturalist character.

The product should be built as an account-backed browser app with a canonical server-side simulation. The client renders and sonifies state; it does not own bird personality, mood continuity, notebook observations, invite state, or conflict resolution. The server advances each aviary on a slow tick whether or not a client is connected.

The v1 scope includes:

- Web app only, running in the last two major versions of Chrome, Safari, Firefox, and Edge.
- Email magic-link sign-in, per-device sessions, session revocation, verified email change, account export, and 30-day soft deletion.
- One account, one canonical aviary, two starter birds, and support for age-gated growth up to seven birds.
- Hidden per-bird personality vectors, persistent mood, procedural call grammar, bird-to-bird interaction, server-authored drift, and server-authored mood transitions.
- Return greetings, listen-in, offers, settle, field notebook, presence accounting, and quiet top-bar controls.
- Optional visit invitations: named email invites, read-only ambient visiting, revocation, expiration, visit log, and optional visit notifications off by default.
- First-class accessibility: screen-reader narration, call captions, keyboard navigation, reduced-motion mode, focus treatment, and WCAG AA text contrast.
- Aggregate-only operational telemetry and synthetic performance checks, with no per-bird or per-account relationship data in analytics.

The v1 scope explicitly excludes:

- Native iOS or Android apps.
- Password auth, SSO, payments, paid tiers, or billing.
- Multiple aviaries per account, shared aviaries, household profiles, or co-presence.
- Public discovery, profiles, follows, comments, chat, mutual visits, rankings, or leaderboards.
- Achievements, scores, streaks, levels, badges, XP, calendars of visits, "birds adopted" counters, or any disguised gamification.
- Tamagotchi mechanics: hunger, illness, death, distress, decaying happiness, required feeding, or punishment for absence.
- User-visible personality numbers, debug trait panels, optimization dashboards, or export surfaces that present traits as something to manage.
- User-controlled bird placement, scrolling, panning, zooming, customizable scenes, drag-to-place, or perch-selection commands.
- Recorded audio loops or recorded fallback audio.
- Push notifications, email engagement prompts, or "welcome back" announcement surfaces.

Two voice regimes must be enforced from the beginning:

- Product surfaces use naturalist field-notebook voice: lowercase, present-tense, specific, bird-centered, no exclamation, no gamified phrasing, and no direct behavioral judgment of the user.
- System surfaces use plain matter-of-fact voice: sign-in, account settings, sync errors, accessibility settings, deletion, export, and unsupported-browser states.

This split should be represented in copy ownership, code review checklists, component naming, automated string linting where practical, and design QA.

## 2. Architecture Overview

Build the system as five cooperating layers:

1. Web client
   - TypeScript application delivered as server-rendered HTML plus a small client bundle.
   - A scene renderer for birds, perches, day/night, weather, and micro-motion.
   - A WebAudio call engine and caption generator.
   - Accessible DOM surfaces for top-bar controls, settings, notebook, narration, captions, and keyboard focus.
   - A snapshot consumer that interpolates server state and submits user events.

2. Edge/bootstrap layer
   - Serves the app shell and, for signed-in users, embeds or early-fetches a compact initial aviary snapshot.
   - Keeps time-to-first-bird under 500ms on a mid-tier mobile device over 4G.
   - Uses CDN caching only for static assets and unauthenticated shell resources; account snapshots remain authenticated and non-cacheable by shared caches.

3. API service
   - Owns auth, sessions, account settings, event ingestion, snapshots, notebook reads, invite management, account export, deletion, and matter-of-fact error responses.
   - Accepts client interaction events into an append-only log.
   - Never accepts client writes to personality vector, mood, notebook observations, canonical positions, or current aviary state.

4. Simulation service
   - Runs account-sharded ticks at roughly one-minute cadence.
   - Consumes unprocessed interaction events in server order.
   - Rolls up presence accurately.
   - Applies drift deltas, mood transitions, weather effects, bird-to-bird reactions, notebook eligibility, and canonical snapshot updates.
   - Is the only writer of personality vectors and mood.

5. Data and operations layer
   - Primary relational store for accounts, birds, state, events, invites, notebook entries, deletion/export jobs, and session records.
   - Job queue for magic-link email, account export, deletion finalization, invite email, and simulation shards.
   - Aggregate operational telemetry pipeline with hard schema boundaries that exclude per-bird fields and per-account interaction histories.
   - Synthetic browser fleet for performance and accessibility regression checks.

Recommended baseline stack:

- TypeScript end to end for client, API, and simulation worker to keep domain model types shared and audited.
- Postgres as the source of truth, using transactions, row-level locking, SKIP LOCKED account tick leases, JSONB only for bounded snapshot payloads, and normal columns for queryable state.
- Redis or equivalent only for ephemeral rate limits, magic-link one-time token lookup, session revocation cache, and simulation shard leases; Postgres remains the durable authority.
- A queue such as Cloud Tasks, SQS, or Sidekiq-style durable jobs for email, export, deletion, and simulation scheduling.
- WebGL or Canvas 2D for the scene renderer, chosen after a spike against the 500ms and 60fps budgets. If WebGL is used, preserve a tested Canvas 2D reduced-capability path only if it does not bloat the initial bundle.
- WebAudio with AudioWorklet where available and a main-thread fallback for supported browsers that lack worklet support but have enough WebAudio capability.

The architecture should deliberately avoid real-time multiplayer primitives. Snapshot polling plus low-frequency visible-tab keepalive is sufficient for a single canonical state, and it preserves the PRD's "clients pull snapshots and interpolate" model. Server-sent events can be added later for efficiency, but v1 should not require a live socket for correctness.

## 3. Core Domain Model

Use synthetic UUIDs for all internal identifiers. Email is stored only on the account record, encrypted at rest, and never used as a partition key, foreign key, log key, metric label, cache key, or URL identifier.

### Account

Fields:

- `id`: synthetic UUID.
- `email_encrypted`: encrypted verified email.
- `email_verification_state`: verified, pending_change.
- `pending_email_encrypted`: nullable.
- `created_at`, `updated_at`.
- `timezone`: IANA timezone, captured from client and updateable, used for day/night and mood inputs.
- `deleted_at`: nullable soft-delete timestamp.
- `hard_delete_after`: nullable timestamp, normally `deleted_at + 30 days`.
- `privacy_policy_version_acknowledged`: version string.

Rules:

- One active aviary per account.
- Deleted accounts cannot receive simulation ticks, invites, exports, or new sessions except restore flow during the soft-delete window.
- Account export and deletion jobs must address the account by synthetic UUID only.

### SessionToken

Fields:

- `id`: UUID.
- `account_id`.
- `device_label`: browser-supplied coarse label, e.g. "Safari on iPhone"; user-editable later if needed.
- `token_hash`: hash of bearer token, never raw token.
- `created_at`, `last_seen_at`, `expires_at`, `revoked_at`.
- `last_ip_prefix`, `last_user_agent_hash`: optional security metadata with retention bounds.

Rules:

- Revocation is immediate for API requests and takes effect for open tabs on their next snapshot or event call.
- Session list is shown in matter-of-fact account settings.

### Aviary

Fields:

- `id`: UUID.
- `account_id`: unique.
- `created_at`.
- `bird_count_cap`: v1 default 7.
- `starter_completed_at`: nullable until adoption names are accepted.
- `current_state_version`: monotonic integer.
- `last_tick_at`.
- `last_user_presence_at`: nullable.
- `settled_until_reengage`: boolean or timestamped state.
- `active_weather_event_id`: nullable.

Rules:

- The aviary exists after account creation, but visual birds appear after starter adoption naming.
- New bird availability is based on aviary age, not visit count, interaction count, payment, score, or streak.

### Bird

Fields:

- `id`: stable UUID.
- `aviary_id`.
- `species_id`: from the v1 species pool.
- `name`: user-assigned display name.
- `created_at`.
- `adoption_sequence`: 1..7.
- `personality_boldness`, `personality_social_warmth`, `personality_vocal_frequency`, `personality_plumage_saturation`, `personality_curiosity`: normalized hidden scalars.
- `personality_version`: monotonic integer for audit and migration.
- `current_mood`: enum, initially from finalized set such as wary, content, curious, drowsy, alert, settled.
- `mood_started_at`, `mood_intensity`, `mood_carryover_until`.
- `current_perch_zone`: front, middle, back.
- `current_pose`: compact enum or animation state descriptor.
- `call_signature_seed`: stable seed for procedural motif variation.
- `last_offer_at_by_type`: server-maintained cooldown state.

Rules:

- Bird identity is never regenerated by rename, migration, sync, species pool changes, or client cache loss.
- Personality values are not returned to the standard client UI. Export may include them because the PRD requires export of current personality vectors, but the export should label them as raw data and not display them as an in-product stats surface.
- Clients may receive derived render descriptors, not raw trait values, unless an explicit internal debug build is used outside user-facing production.

### Species

Represent the species pool as versioned static data shipped to both simulation and client:

- `species_id`.
- Silhouette and pose set references.
- Default plumage palette parameters.
- Motif library: call motif primitives, pitch range, envelope defaults, rhythmic tendencies.
- Night activity flag for the nightjar-like species.
- Accessibility vocabulary: species phrasing for narration and captions.

The species pool should have about six species in v1. Rarity is not modeled.

### InteractionEvent

Append-only table:

- `id`: UUID.
- `account_id`, `aviary_id`.
- `session_id`.
- `device_event_id`: client idempotency key.
- `server_received_at`.
- `client_observed_at`: client timestamp for sequencing hints, never authoritative.
- `event_type`: presence_ping, listen_in_start, listen_in_end, offer_submitted, offer_reaction_ack, settle_requested, settle_undone, notebook_opened, snapshot_visible, visit_snapshot_visible, etc.
- `bird_id`: nullable depending on event type.
- `payload`: bounded JSON with event-specific fields.
- `processed_at`: nullable.
- `processing_tick_id`: nullable.

Rules:

- Unique constraint on `(session_id, device_event_id)` prevents duplicate processing.
- Payload schemas are versioned and validated. Reject unknown payload fields that could be mistaken for state writes.
- Visitor events are stored only as visit operational events where needed for visit duration/logging; they never enter the host simulation event stream.

### PresenceWindow

Derived server-side representation:

- `id`.
- `account_id`, `aviary_id`.
- `session_id`.
- `started_at`, `ended_at`.
- `source_event_ids`: optional compact audit reference.
- `confidence`: valid, truncated, discarded.

Rules:

- Presence requires visible document, focused window, and recent pointermove or keypress within the calibrated window.
- The client reports condition samples; the server forms windows and unions overlapping windows across sessions for drift so two active devices do not double-count a single user's wall-clock attention.
- Presence stops on settle, tab close/unload where reported, hidden state, focus loss, session timeout, or inactivity beyond the calibrated activity window.
- Presence is not surfaced to the user as a streak, counter, log, or visit-frequency summary.

### AviaryStateSnapshot

Persist the latest canonical snapshot, either as a table plus compact JSON blob or normalized current-state rows:

- `aviary_id`.
- `version`.
- `generated_at`.
- `server_time`.
- `local_time_context`: derived from account timezone.
- `birds`: per-bird render descriptors: id, name, species, mood-derived pose, perch zone, animation phase, call scheduling hints, plumage render params, greeting eligibility.
- `weather`: none, light_rain, soft_wind, ending, plus start/end.
- `lighting`: morning, midday, evening, night, settled, with interpolation factor.
- `notebook_summary`: latest entry id/date preview if notebook exists, no badge count.
- `available_offers`: offer types and cooldown availability.
- `available_new_bird`: nullable age-gated prompt descriptor.
- `top_bar_controls`: permissions for account, accessibility, notebook, offer, settle.

Rules:

- Snapshot payloads stay in the kilobyte range.
- ETags or version numbers allow clients to skip redraw work when state is unchanged.
- Snapshot includes enough animation phase data for the first visible frame to show birds mid-action without a wake-up animation.

### NotebookEntry

Fields:

- `id`.
- `aviary_id`.
- `created_at`.
- `observation_date_local`.
- `text`: naturalist prose.
- `source_tick_id`.
- `source_observation_type`: greeting_order, quiet_morning, rain_response, perch_shift, chorus_event, etc.
- `referenced_bird_ids`: array.

Rules:

- Entries are rare: roughly every few days for a regularly visited aviary, with event-based exceptions for genuinely noteworthy moments.
- Entries are read-only, never editable, never deletable independently from account deletion.
- Entries must describe aviary observations, not user compliance or visit frequency.

### VisitInvitation And VisitSession

Invitation fields:

- `id`.
- `host_account_id`, `aviary_id`.
- `visitor_email_encrypted`.
- `token_hash`.
- `created_at`, `expires_at`, `used_at`, `revoked_at`.
- `created_by_session_id`.

Visit session fields:

- `id`.
- `invitation_id`, `host_account_id`, `aviary_id`.
- `visitor_email_encrypted`.
- `created_at`, `last_seen_at`, `ended_at`, `end_reason`.

Rules:

- Invite links are one-time or session-establishing links with bounded lifetime; unused invites expire after 30 days.
- Revocation immediately invalidates outstanding links and active visit sessions at the next snapshot pull.
- Visitor sessions are read-only and use a separate auth scope from host sessions.
- Visitor presence never enters host drift.
- Visit log is available in account settings, never badged or pushed.

## 4. API Surface

All endpoints use HTTPS, JSON, CSRF protection for cookie-based sessions or bearer tokens for token-based sessions, idempotency keys for writes, and matter-of-fact error copy. API logs must include request ids and synthetic account ids only.

### Auth And Account

- `POST /api/auth/magic-link/request`
  - Input: email.
  - Behavior: normalize email for storage lookup, rate-limit by email hash and IP prefix, create 15-minute token, send email.
  - Response: always generic success to avoid account enumeration.

- `GET /auth/magic-link/consume?token=...`
  - Behavior: validate unused unexpired token, invalidate it, create per-device session, redirect to aviary.
  - Error: matter-of-fact expired/invalid surface with option to request a new link.

- `GET /api/account`
  - Returns account settings, session list, privacy policy link, export/deletion status, accessibility defaults.

- `PATCH /api/account/email`
  - Starts verified email change; old email remains active until new email is verified.

- `POST /api/account/sessions/{sessionId}/revoke`
  - Revokes a device session.

- `POST /api/account/export`
  - Queues export and emails a verified-address download link when ready.

- `POST /api/account/delete`
  - Starts 30-day soft deletion.

- `POST /api/account/restore`
  - Restores during soft-delete window.

### Adoption And Bird Settings

- `GET /api/aviary/adoption`
  - Returns starter species chosen by server and default name suggestions if starter adoption is incomplete.

- `POST /api/aviary/adoption`
  - Input: two names.
  - Behavior: validates names, creates two bird records with stable ids and initial vectors, marks starter complete.

- `PATCH /api/birds/{birdId}`
  - Input: name only in v1.
  - Behavior: rename bird. No personality or species mutation.

- `GET /api/aviary/new-bird-offer`
  - Returns age-gated availability when eligible.

- `POST /api/aviary/new-bird-offer/accept`
  - Creates the next bird from the server-chosen species pool, up to cap seven. This is not tied to engagement score.

### State And Events

- `GET /api/aviary/bootstrap`
  - Returns initial snapshot plus static species asset manifest hash and account UI permissions.
  - Used immediately after app shell load. Target response is small enough for edge-adjacent delivery.

- `GET /api/aviary/snapshot?afterVersion=N`
  - Returns 200 with snapshot if newer, 204 if unchanged, or appropriate matter-of-fact error.
  - Called on visibility becoming visible, after long render gaps, and visible-tab keepalive.

- `POST /api/aviary/events`
  - Input: batch of interaction events with idempotency keys.
  - Accepts listen-in start/end, offer submitted, settle requested, settle undone, presence samples, and client lifecycle signals.
  - Response: accepted event ids and any immediate user-facing affordance constraints, such as offer cooldown.
  - Does not synchronously mutate personality.

- `POST /api/aviary/offers`
  - Convenience endpoint or event wrapper for seed, song fragment, and still pool.
  - Server validates per-bird cooldowns and creates event(s); the simulation tick determines long-term effects.
  - Immediate reaction animation can be returned as a short-lived render hint, but the canonical state remains server-owned.

- `POST /api/aviary/listen-in`
  - Convenience endpoint for focus changes if not using generic events.
  - Records focused bird and timestamps. Client handles mix ramp immediately; simulation later uses duration for drift.

- `POST /api/aviary/settle`
  - Records settle request, returns transition hint for the slow evening shift.

- `POST /api/aviary/settle/undo`
  - Accepted only within five seconds of settle request. Records undo and returns normal lighting hint.

### Notebook

- `GET /api/aviary/notebook?before=entryId&limit=30`
  - Returns read-only entries newest-first or chronological depending on UI decision.
  - No create/update/delete endpoint exists for users.

### Accessibility Settings

- `GET /api/accessibility`
  - Returns persisted preferences: reduced motion override, captions default, narration cadence preference if offered, audio mute/default.

- `PATCH /api/accessibility`
  - Updates preferences in matter-of-fact settings UI.

### Visits

- `POST /api/visits/invitations`
  - Host creates named email invite. Sends one-time link. Visits remain off unless this is called.

- `GET /api/visits/invitations`
  - Host account settings list outstanding, expired, revoked, and recently used invites.

- `POST /api/visits/invitations/{inviteId}/revoke`
  - Revokes an invite and any active visit sessions.

- `GET /visit/{token}`
  - Consumes or opens a visitor session if token is valid.

- `GET /api/visit/snapshot?afterVersion=N`
  - Visitor read-only snapshot of host aviary. Same render state, no host-only settings, no interaction permissions.

- `GET /api/visits/log`
  - Host on-demand visit log with visitor email, date, approximate duration, and outstanding invitations.

- `PATCH /api/visits/notification-setting`
  - Host opt-in for visit notifications, off by default and not presented during onboarding.

### API Copy Rules

Enforce these with review and tests:

- No success toast for return-greeting, visit, notebook update, offer outcome, or presence accrual.
- No "welcome back", "you have been gone", "streak", "achievement", "level", or "score" strings in product UI.
- Errors and settings use normal capitalization and direct instructions.
- Product observations use lowercase naturalist prose and avoid addressing the user as a metric subject.

## 5. Simulation Engine Design

The simulation service is the product's behavioral authority. It must be deterministic enough to test, varied enough to avoid canned repetition, and slow enough that the user perceives change across weeks rather than minutes.

### Tick Scheduling

- Run a tick for each active, non-deleted aviary about once per minute.
- Use a durable lease table or queue shard so exactly one simulation worker owns an aviary tick at a time.
- If a tick is missed because of outage or load, process catch-up in bounded increments. Do not simulate thousands of minute steps one by one after a long absence; instead compute elapsed-time effects with capped catch-up formulas that preserve mood continuity without excessive work.
- Store `tick_id`, `started_at`, `completed_at`, `duration_ms`, `events_processed_count`, and operational status. Do not log per-bird personality values to telemetry.

### Tick Pipeline

Each tick should:

1. Load aviary, birds, current snapshot version, account timezone, active weather, and unprocessed events through a transaction or consistent read.
2. Validate and order unprocessed events by server receipt time with client timestamps only as hints.
3. Build presence windows from presence samples, ending windows on hidden/focus-lost/inactivity/settle/session-timeout.
4. Union overlapping host presence across devices for drift accounting.
5. Apply event effects to fast state: listen-in durations, offer reactions, settle state, undo, and cooldowns.
6. Compute mood transitions using current mood, personality, local time, weather, bird-to-bird influence, and recent interactions.
7. Compute slow drift deltas from presence, listen-in, offers, and elapsed time.
8. Clamp drift to monotonic expressive bounds and update personality vectors.
9. Select perch zones, pose states, call scheduling hints, and possible greeting candidates.
10. Possibly generate a rare notebook entry if observation eligibility rules are met.
11. Possibly start or end a rare weather event.
12. Write updated birds, snapshot, processed-event markers, and tick metadata atomically.

### Personality Drift

Normalize each trait to a hidden 0..1 range in storage, with species-specific initial distributions and render mappings. Never expose the raw range in user UI.

Use a low-pass filter:

- Aggregate each day's valid host presence into a bounded daily presence score.
- Aggregate per-bird listen-in duration with a cooldown/cap so leaving listen-in active cannot saturate drift.
- Aggregate offers by type and bird reaction, respecting per-bird cooldowns.
- Convert the aggregate signal into tiny positive deltas.
- Apply deltas through a smoothing constant calibrated so regular use produces measurable instrumentation movement after roughly one week and visible render/audio change after roughly three weeks.

Trait mapping:

- Presence-time: broad expressive lift across boldness, social warmth, vocal frequency, and plumage saturation, with species/personality-specific weighting.
- Listen-in on a bird: stronger positive input to that bird's social warmth and vocal frequency.
- Offer near a bird: small positive input to boldness.
- Accepted/investigated offer: small positive input to curiosity.
- Song fragment response: small input to vocal frequency and social warmth if the bird joins or responds.
- Settle: mood-quieting and presence termination; no direct positive or negative personality drift.
- Absence/neglect: no negative personality deltas. Birds may become ambient through mood and greeting likelihood, not trait decay.

Calibrate with simulation harnesses:

- Synthetic users with no presence should show no punitive trait decay.
- Regular short visits should show small numeric movement by day seven.
- Daily multi-hour active presence should not reach cap quickly.
- Repeated offers in one session should be bounded by cooldown and daily caps.
- Two simultaneous devices should not double-count presence.

### Mood System

Finalize a small enum before implementation: wary, content, curious, drowsy, alert, settled. Keep it small enough that rendering, audio, narration, and tests can cover every state.

Mood transition inputs:

- Current mood and dwell time.
- Local time band: morning, midday, evening, night.
- Recent host interactions: return, listen-in, offers, settle.
- Weather: rain dampens vocal frequency and quiets; wind can raise alert/wary probability.
- Other birds: wary spreads lightly; calls can invite response; chorus can lift social warmth expression.
- Personality: high boldness resists wary and front-perch avoidance; high curiosity increases investigate behavior; high vocal frequency increases call propensity.

Mood behavior:

- Mood persists across sessions and is advanced by server ticks.
- Opening the tab never resets a bird to neutral.
- Mood outputs should be readable through pose, perch, call timing, and narration, not labels.
- Night usually moves birds toward settled or drowsy, except the nightjar-like species can remain active.

### Return-Greeting

Greeting is a snapshot-time behavior derived from server state and client lifecycle:

- On fresh navigation or visible return after absence, the server marks greeting candidates based on absence length, mood, boldness, social warmth, recent greeting history, and local time.
- The client renders one greeting within one to two seconds. If multiple birds qualify, stagger them with small randomized offsets.
- Greeting forms include glance, quiet call, head tilt, step toward front perch, longer call, and another bird's response.
- Never show text for greeting. The bird greeting is the entire welcome surface.
- Do not trigger greeting for visitor sessions; visitors observe the host aviary and do not cause host-facing interactions or drift.

### Offers

Offer types:

- Seed: visual object near the front; curiosity/boldness/mood determine approach, wait, or ignore.
- Song fragment: a short motif from a small library; vocal frequency and mood determine join, quiet, or call-against reaction.
- Still pool: reflective surface; birds may drink, bathe, or watch.

Rules:

- Offers are launched from the top bar, not by clicking birds.
- Per-bird cooldown is a few minutes; enforce server-side.
- Reactions are mood/personality-shaped and procedurally varied.
- Offer events are gestures, not feeding or care mechanics; no hunger, inventory, or reward loop.

### Bird-To-Bird Interaction

Model bird-to-bird interaction in the tick and call scheduler:

- Calls from one bird can produce response probability in nearby or socially warm birds.
- A wary bird can lightly shift others toward alert/wary for a short window.
- Chorus emerges when call windows overlap from high vocal frequency and social warmth; do not trigger chorus as a reward animation.
- Perch spacing should avoid overlapping birds and should express social warmth through proximity without becoming user-controlled layout.

### Notebook Generation

Use a deterministic, rule-based naturalist prose generator for v1 rather than a general language model. This keeps privacy boundaries simple, avoids per-account training questions, and makes style QA enforceable.

Generation inputs:

- Greeting order changes.
- Rare weather responses.
- Extended quiet periods.
- Perch shifts that are meaningful relative to a bird's history.
- Chorus events.
- Offer reactions if rare or distinctive.
- Long-term drift crossing a render-notice threshold, described behaviorally rather than numerically.

Sparsity:

- A regular aviary should receive about one entry every few days.
- Active users should not get an entry every session.
- Apply per-observation cooldowns and a global notebook cadence cap.

Style constraints:

- Lowercase, present-tense, specific.
- Name birds when useful.
- Describe aviary observations, never user adherence.
- No "achievement", "streak", "you visited", "level", or numeric trait copy.

## 6. Sync And Conflict Model

The sync strategy is canonical server state plus append-only client events. There is no client-to-client sync and no merge of personality state.

### State Authority

- Server simulation is the only writer of personality vectors, mood, perch choice, cooldowns, weather, and notebook observations.
- Clients write interaction events only.
- Snapshot versions are monotonic. Every state-changing tick increments the version.
- Clients treat local state as a render cache. A new snapshot can override local interpolation targets but should transition smoothly where possible.

### Event Ordering And Idempotency

- All client write requests include a `device_event_id`.
- Server receipt time is authoritative for ordering.
- Client-observed time is retained only to improve duration estimates when safe.
- Duplicate events are accepted idempotently and not processed twice.
- Events older than a conservative cutoff are accepted only if they are lifecycle/end events needed to close presence; otherwise they are discarded with operational logging.

### Multi-Device Behavior

- Multiple host devices can be signed in simultaneously.
- Each device can render the same snapshot and submit events.
- Presence across devices is unioned by wall-clock interval for account-level drift.
- Listen-in drift is per bird and capped, with simultaneous listen-in intervals unioned per bird.
- Offer cooldowns are server-enforced; if two devices offer at once, one wins by server order and the other gets a matter-of-fact unavailable response or no-op affordance.
- Settle affects the aviary view for that session immediately and canonical state through the event stream. If another active device re-engages, settle can be lifted by interaction without framing either device as conflicting.

### Snapshot Pulls

Clients pull snapshots:

- On initial boot.
- When visibility changes to visible.
- After long render-frame gaps or resume from sleep.
- On a low-frequency visible-tab interval.
- Immediately after accepted write events where the client needs updated permissions or cooldowns.

Use `afterVersion` and 204 responses to reduce payload. Keep visitor and host snapshot APIs separate by auth scope even if they share renderer payload types.

### Conflict Surfaces

Most conflicts should be prevented architecturally. User-facing conflict/error surfaces are reserved for:

- Expired or consumed magic links.
- Revoked or expired visitor links.
- Revoked sessions.
- Snapshot load failures.
- Offer cooldown races.
- Unsupported browsers.

All conflict/error copy is matter-of-fact. Do not naturalize failures.

## 7. Frontend Rendering Pipeline

The frontend must make the first visible state feel like an already-running aviary, not a loaded app.

### Boot Path

1. Browser receives minimal HTML, critical CSS, and static manifest.
2. If authenticated, client fetches or receives initial snapshot as early as possible.
3. Client paints quiet field immediately if snapshot is not ready.
4. As soon as snapshot and minimal species assets are available, first bird is visible in its current pose and animation phase.
5. Non-critical assets, notebook UI, account settings, visit management, export/delete UI, and accessibility settings are code-split.

No spinner, no progress bar, no welcome modal, no entry animation, no fade-from-static.

### Scene Composition

Render one horizontal scene that fits the viewport:

- Background sky and foliage.
- Back, middle, and front perch zones.
- Birds on stable middle plane with mood/personality-shaped pose.
- Occasional foreground branch/leaf elements.
- Ambient weather overlays.
- Top bar above the scene, not inside it.

Responsive rules:

- No panning, scrolling, or zooming.
- All birds remain visible on phone and desktop.
- Narrow viewports compress spacing without cropping birds.
- Wide viewports widen perch spacing but keep a single glanceable scene.
- Maintain stable hit targets for birds and top-bar controls.

### Bird Rendering

Use a state-driven animation system:

- Server snapshot gives mood, perch zone, pose family, animation phase, call scheduling hints, and derived visual parameters.
- Client selects interpolated frames or procedural poses from species assets.
- Mood changes alter posture, head movement, perch depth, call readiness, and micro-motion.
- Personality-derived render parameters influence boldness/proximity, call frequency hints, and plumage saturation without exposing numbers.

Idle micro-motion:

- Preening, scanning, head tilting, body shuffle, feather fluffing, low-perch drowsy poses.
- Runs continuously while visible.
- Uses deterministic seeds plus local randomness bounded by snapshot state so it does not look looped.
- Stops expensive rendering when hidden, but resumes from server snapshot rather than pretending no time passed.

Reduced-motion mode:

- Replaces micro-animation with slow cross-fades among still poses.
- Replaces flight paths with cross-fades between perch states.
- Removes ambient leaf drift.
- Preserves color shifts, calls/captions, mood, notebook, and drift.

### Top Bar

Top bar controls:

- Account/settings.
- Accessibility settings.
- Field notebook.
- Offer.
- Settle.

Rules:

- Top bar sits above the aviary scene.
- It fades nearly transparent after a few seconds of cursor stillness.
- It returns on pointer movement or keyboard activity.
- Icons have accessible names and visible focus states.
- No badges for visits, notebook count, streak, or activity.

### Interaction Handling

Listen-in:

- Click/tap/keyboard focus on a bird engages listen-in.
- Click same bird again, focus another bird, click empty space, or move focus away to disengage.
- Audio mix ramps gradually in and out.
- Other birds quiet to ambient but never fully silence.
- Client records start/end events and duration hints.

Offer:

- Opened from top bar.
- Shows seed, song fragment, and still pool as quiet options.
- Keyboard navigable.
- Server validates cooldown and returns reaction hints.
- Client renders offer object and mood-shaped reaction.

Settle:

- Top-bar action.
- Slow evening lighting shift and quieter calls.
- Five-second undo by any click in aviary.
- Closing tab without settle is equally valid.

Notebook:

- Top-bar icon opens read-only notebook.
- Entries are naturalist observations, not event logs.
- No edit/delete/comment affordances.

### Client Lifecycle And Presence

The client maintains a presence sampler:

- Tracks `document.visibilityState`.
- Tracks window focus.
- Tracks recent pointermove or keypress timestamp.
- Sends bounded presence samples while all three conditions hold.
- Sends lifecycle events on hide, blur, settle, unload/pagehide where possible.
- Treats mobile browser lifecycle as lossy; server closes stale windows after inactivity.

Do not count scroll inside notebook or settings as separate engagement metrics for drift unless the aviary remains visible/focused and the presence definition holds.

## 8. Audio Pipeline

Audio is procedural, per-bird, and personality-shaped.

### Call Grammar

Each species defines motif primitives:

- Pitch contour patterns.
- Rhythm cells.
- Envelope shapes.
- Timbre/noise components.
- Mood modifiers.
- Caption phrase fragments tied to actual generated contour.

Each bird has a stable call signature seed. The call engine combines species motifs, bird seed, current mood, vocal-frequency expression, and local call context to produce varied calls that remain recognizable as that bird.

### WebAudio Runtime

Implement:

- One shared AudioContext.
- Pooled oscillators/noise sources or AudioWorklet synthesis to avoid per-call allocation churn.
- Per-bird gain nodes.
- Master ambient gain and user mute control.
- Listen-in mix bus with slow ramping.
- Chorus mixer that allows overlapping calls without phase artifacts.
- Output limiter/compressor tuned conservatively to prevent harsh peaks.

Call scheduling:

- Server snapshot provides call windows/hints.
- Client schedules calls locally within those windows using deterministic variation.
- Birds call more often according to vocal-frequency expression and mood.
- Rain and evening reduce call density.
- Nightjar-like species can remain active at night.

### Listen-In Mix

On listen-in:

- Focused bird gain rises over a slow ramp.
- Other birds lower to ambient, never zero.
- Ambient/weather audio remains subtle.
- Disengage restores ambient mix with same ramp.

No hard cuts and no solo/mute UI vocabulary.

### Captions

Captions are generated from the same call grammar event that produced the sound:

- "a soft three-note rise"
- "a low trill, paused, low trill again"
- "a single sharp call from the back perch"

Rules:

- Captions are optional and can default on when audio is unavailable.
- Caption text appears near the calling bird and fades with the call.
- Captions must pass contrast and not overlap incoherently with scene controls.
- Captions use naturalist voice and do not expose waveform or motif ids.

### Fallbacks

- If WebAudio is unavailable or permission is denied, play in graceful silence.
- Turn captions on by default for that session and explain in matter-of-fact settings/error copy if needed.
- Do not ship recorded audio fallback.

## 9. Accessibility Plan

Accessibility is a product surface, not a compliance layer.

### Screen-Reader Narration

Build a narration generator from the same snapshot state as the visual renderer. It should output slow naturalist prose:

- Idle cadence: roughly every 30 to 60 seconds.
- Faster only for user-initiated events: return-greeting, offer reaction, settle, listen-in changes if important.
- Queue management prevents screen-reader flooding.
- User can pause or adjust narration in accessibility settings if needed.

Implementation:

- Use an ARIA live region with polite updates for idle narration.
- Use assertive sparingly only for system errors that block use.
- Keep narration strings distinct from visual labels so naturalist voice is preserved.
- Do not narrate raw state lists, perch numbers, mood labels, or personality values.

### Keyboard Navigation

Required behavior:

- Tab moves through top-bar controls.
- Tab can enter the aviary scene.
- Arrow keys move focus between birds.
- Enter toggles listen-in for the focused bird.
- Escape exits listen-in or closes transient UI.
- Offer menu is fully keyboard navigable.
- Settle is keyboard reachable.
- Notebook and settings are keyboard navigable with focus trap only when modal/dialog semantics require it.

Focus visuals:

- Soft high-contrast focus ring visible against morning, evening, night, and weather palettes.
- Focus ring must not look like an in-scene badge or label.

### Reduced Motion

Support both:

- OS preference via `prefers-reduced-motion`.
- User setting override.

Reduced-motion QA must verify that:

- Birds still feel alive through pose changes, cross-fades, calls, captions, and narration.
- No animated flight paths remain.
- Ambient leaf drift is removed.
- Day/evening color transitions remain but are slowed.
- Performance remains within budget.

### Captions And Audio-Off

- Captions are available independently of screen-reader narration.
- Captions can default on if audio is muted/unavailable.
- Captions must be generated from actual calls.
- Audio settings use matter-of-fact UI copy; caption prose uses naturalist voice.

### Contrast And Text

- WCAG AA minimum for top bar, settings, account, errors, captions, notebook, and any displayed narration.
- Test contrast in morning, midday, evening, night, rain, and settled palettes.
- No user-copy text appears inside the aviary scene except captions and focus/accessibility affordances.

## 10. Privacy And Security Plan

### PII Boundary

- Email encrypted on account record only.
- Synthetic UUIDs everywhere else.
- Logs, metrics, queues, exports, and inter-service messages use account UUID.
- Email sending jobs receive the minimum decrypted email data at send time and do not log it.
- Visitor emails follow the same encrypted-storage rule.

### Interaction Data Boundary

Per-bird interaction data drives only that user's simulation. It must not be:

- Sent to analytics warehouse.
- Used for model training.
- Aggregated into population dashboards.
- Shared with third parties.
- Used to recommend behavior to other users.

Allowed telemetry:

- Request counts.
- Latencies.
- Error rates.
- Anonymous session-duration histograms without account dimension.
- First-bird render timings.
- Render-frame timings.
- Audio-context errors.
- Simulation-tick latency and failure counts.

Implement telemetry schemas that make forbidden fields impossible to include by accident. Add automated tests or schema linting to reject event names or labels containing bird id, species id attached to account activity, personality fields, raw interaction payloads, email, invite email, or notebook text.

### Account Export

Export includes:

- Birds.
- Names.
- Current raw personality vectors.
- Current moods.
- Notebook entries.
- Account settings.
- Visit invite/log data if required for user transparency.

Export delivery:

- On-demand job.
- Download link emailed to verified address.
- Link expires.
- Access logged by account UUID.

The in-product export screen should describe it plainly as a data export, not as a stats dashboard.

### Account Deletion

Deletion flow:

- User confirms in account settings.
- Account is soft-deleted immediately.
- Sessions are revoked or blocked except restore.
- Simulation ticks stop.
- Invites are revoked.
- Export jobs are canceled unless already delivered.
- User can restore within 30 days.
- Hard-delete job removes account, birds, vectors, notebook, event log, telemetry join records, invites, and sessions after window.

Backups should honor hard deletion through documented retention windows.

### Visit Privacy

- Visits are off by default.
- Host invites by specific email.
- Invite expires after 30 days unused.
- Host can revoke immediately.
- Visitor cannot interact or affect drift.
- Host can inspect visit log on demand.
- No badge, push, email, or in-product notification unless host explicitly opts in.

## 11. Performance And Observability

### Budgets

- Initial JS bundle under 2MB gzipped.
- First bird visible under 500ms on mid-tier mobile over 4G.
- 60fps idle motion on a five-year-old mid-range laptop for a 30-minute session.
- No client memory growth over 30 minutes.
- Snapshot payloads in kilobytes.
- Simulation tick p99 latency alarm at 5 seconds.

### Client Performance Work

- Code split account settings, notebook, visit management, export/delete, and accessibility settings.
- Keep species assets compact and cacheable.
- Use procedural render parameters where possible.
- Preload only the two starter bird assets and critical scene assets at boot.
- Reuse audio buffers/nodes.
- Avoid per-frame object allocation in animation loops.
- Pause rendering when hidden and resume from fresh snapshot.
- Add memory-leak tests for 30-minute synthetic sessions.

### Server Performance Work

- Shard simulation ticks by account id.
- Batch ticks where safe while preserving per-aviary transactions.
- Index unprocessed events by aviary and server receipt time.
- Keep notebook generation rule-based and cheap.
- Keep snapshots precomputed during tick so reads are fast.
- Use backpressure on tick queues and alarm before ticks fall behind.

### Observability

Dashboards:

- API latency/error rate by endpoint.
- Magic-link request/consume failures, without email dimensions.
- Snapshot freshness and payload size.
- Tick latency p50/p95/p99 and backlog.
- Event ingestion rate and idempotency duplicate rate.
- First-bird timing synthetic and aggregate RUM.
- Render frame timing aggregate RUM.
- Audio-context error counts.
- Export/delete job success/failure counts.
- Invite link expired/revoked/opened counts without visitor email labels.

Forbidden dashboards:

- Average personality drift by population.
- Per-species engagement or per-bird interaction analytics.
- User visit streaks.
- Leaderboards of aviaries, visits, bird counts, or session lengths.

## 12. Testing Strategy

### Unit Tests

- Presence condition logic: visible + focused + recent pointer/key only.
- Presence window close behavior on hidden, blur, inactivity, settle, session timeout.
- Multi-device union prevents double-counting.
- Drift monotonicity and cap behavior.
- Offer cooldown enforcement.
- Mood transition matrix.
- Greeting candidate selection and staggering.
- Notebook eligibility and style filters.
- Magic-link expiry and single-use consumption.
- Invite expiry/revocation.
- Session revocation.
- Export/deletion job selection.

### Property And Simulation Tests

Build an offline simulation harness with seeded synthetic accounts:

- No presence for weeks yields no negative personality drift.
- Regular short visits produce measurable drift after about a week.
- Regular visits produce visible derived expression after about three weeks.
- Heavy clicking/offers cannot saturate curiosity or boldness in one session.
- Concurrent devices preserve additive event ordering and do not lose drift.
- Missed ticks catch up without runaway CPU or mood snapping.
- Bird count growth follows aviary age only.

### Integration Tests

- Full signup and starter adoption.
- Return after short absence and long absence.
- Listen-in engage/disengage from mouse, touch, and keyboard.
- Offer seed/song/still pool with cooldown.
- Settle and five-second undo.
- Notebook open/read.
- Account export.
- Soft delete and restore.
- Hard-delete dry-run in staging.
- Host invite, visitor view, revocation, expired link.
- Visitor cannot submit interactions to host simulation.

### Accessibility Tests

- Automated axe checks for settings, notebook, auth, visit, and top-bar surfaces.
- Screen-reader manual QA for narration cadence and voice.
- Keyboard-only complete session.
- Reduced-motion visual QA.
- Captions with audio muted and WebAudio unavailable.
- Contrast checks across palettes and weather.

### Performance Tests

- Bundle size CI gate.
- Synthetic first-bird timing on throttled mobile profiles.
- 30-minute idle session memory test.
- 60fps animation budget test on representative laptop hardware.
- Audio call allocation test.
- Tick p99 load test with realistic event volume.

### Copy And Scope Tests

Add string linting for banned terms in user-facing product surfaces:

- welcome back
- streak
- achievement
- badge
- level
- score
- XP
- happiness meter
- hungry
- died
- leaderboard
- public profile

Use allowlisted exceptions only in developer docs and tests.

## 13. Delivery Plan

### Milestone 0: Foundations

- Finalize stack, repository structure, CI, deployment environments, secret management, and database migrations.
- Define shared domain types for account, aviary, bird, event, snapshot, invite, notebook, and settings.
- Implement auth/session foundation with synthetic account ids and encrypted email.
- Implement static species pool format and initial placeholder visual/audio assets sufficient for testing.
- Establish privacy-safe telemetry schemas and banned-field linting before feature work begins.

Exit criteria:

- Magic-link sign-in works in staging.
- Account id boundary verified in logs.
- Empty aviary/adoption state exists.
- Initial app shell loads under bundle budget with placeholder scene.

### Milestone 1: Canonical State And Simulation Skeleton

- Implement aviary, bird, event log, and snapshot tables.
- Implement starter adoption: two server-selected birds, names, stable ids.
- Implement simulation tick lease and no-op tick.
- Implement event ingestion with idempotency.
- Implement current snapshot read and versioning.
- Implement basic presence sampler and server presence windows.

Exit criteria:

- A signed-in user can adopt two birds and see a server-authored snapshot.
- Tick advances snapshot version.
- Clients can submit events without mutating personality directly.
- Presence windows are formed only from valid condition conjunctions.

### Milestone 2: Bird Engine

- Implement personality vector seeding and hidden persistence.
- Implement drift low-pass function with calibration harness.
- Implement mood enum and transition rules.
- Implement perch selection and pose descriptors.
- Implement return-greeting candidate generation.
- Implement offer cooldowns and reaction selection.
- Implement bird-to-bird call/response and wary propagation.
- Implement rare weather scheduling.

Exit criteria:

- Simulation harness demonstrates one-week measurable and three-week visible drift targets.
- No negative drift on absence.
- Mood persists across sessions and advances through time.
- Snapshot contains enough render descriptors for mid-action first frame.

### Milestone 3: Scene Renderer And Core Interactions

- Build responsive one-screen scene.
- Implement top bar with fade behavior.
- Implement bird rendering, idle motion, mood-shaped pose, perch zones, day/night, weather, and ambient ornaments.
- Implement listen-in UI state and client event recording.
- Implement offer menu and reaction rendering.
- Implement settle and five-second undo.
- Implement hidden-tab render pause and resume snapshot pull.

Exit criteria:

- First bird visible under 500ms in synthetic test with production-like assets.
- No spinner or welcome surface.
- Birds remain visible across phone and desktop widths.
- Listen-in/offer/settle work by mouse, touch, and keyboard.

### Milestone 4: Audio And Captions

- Implement procedural motif grammar and stable per-bird call signatures.
- Implement WebAudio synthesis, chorus mixing, and listen-in gain ramps.
- Implement generated captions from call grammar.
- Implement graceful silence with captions if WebAudio unavailable.
- Add audio performance tests for allocation and long session behavior.

Exit criteria:

- Calls vary without loops and remain recognizable by bird.
- Listen-in sounds like mix rebalancing, not solo switching.
- Captions match generated calls.
- No recorded audio assets are used.

### Milestone 5: Notebook And Accessibility

- Implement notebook observation rules and prose generator.
- Implement notebook UI as read-only, sparse, and naturalist.
- Implement screen-reader narration generator and live region.
- Implement reduced-motion renderer.
- Implement accessibility settings.
- Complete contrast and keyboard QA.

Exit criteria:

- Screen-reader user receives a living aviary narration, not a state list.
- Reduced-motion mode is a designed surface, not a static fallback.
- Notebook entries are rare and specific.
- All interactive surfaces pass keyboard and contrast acceptance.

### Milestone 6: Account Completeness And Visits

- Implement account export and deletion.
- Implement session list and revocation.
- Implement email change verification.
- Implement invite creation, email, visitor sessions, read-only snapshot, visit log, revocation, expiration, and optional notifications off by default.
- Ensure visitor events cannot enter host simulation.

Exit criteria:

- Host can invite and revoke visitor.
- Visitor sees actual aviary read-only.
- Visit log is on-demand only.
- Account export and deletion pass privacy review.

### Milestone 7: Hardening, Calibration, And Launch

- Run long-session memory and frame tests.
- Run tick load tests.
- Run copy/scope linting.
- Run privacy telemetry audit.
- Run accessibility manual QA.
- Calibrate drift and mood with internal dogfood over several weeks.
- Prepare staged rollout.

Exit criteria:

- All v1 budgets met.
- No forbidden telemetry fields.
- No gamified or announcement-style surfaces.
- Drift calibration approved through instruments and qualitative review.

## 14. Rollout Plan

### Internal Dogfood

- Start with employees/testers only.
- Two birds per aviary.
- Enable aggregate performance, error, and tick telemetry from day one.
- Review daily for first-bird timing, tick backlog, audio errors, and accessibility regressions.
- Run drift calibration silently through instruments; do not expose trait numbers to testers in product.

### Private Beta

- Invite small cohort.
- Keep two-bird start and cap seven, but delay age-gated third-bird offers until calibration confirms timing.
- Enable visit invitations for a subset after host/visitor privacy QA.
- Monitor support issues, unsupported browser rate, magic-link failures, and snapshot freshness.

### Gradual V1 Launch

- Open signups gradually.
- Keep birds-per-aviary growth conservative: two at start, third only after the aviary age threshold, later birds slowly over months.
- Do not add engagement prompts to improve retention.
- Use aggregate health metrics, not per-bird behavior dashboards, to judge launch stability.

### Post-Launch Calibration

- Tune drift constants and mood probabilities only through server configuration/migrations with test harness validation.
- Avoid user-visible announcements about calibration changes.
- Preserve bird identity and personality continuity through migrations.

## 15. Key Risks And Mitigations

### Drift Calibration Too Fast Or Too Slow

Risk: birds feel manipulable session-to-session or inert over weeks.

Mitigations:

- Offline simulation harness with synthetic behavior profiles.
- Feature-flagged drift constants.
- Separate measurable and visible thresholds.
- Daily caps and low-pass smoothing.
- No negative drift on absence.

### Presence Signal Corruption

Risk: background tabs or multiple devices inflate drift.

Mitigations:

- Strict visible + focused + recent activity conjunction.
- Server-derived windows from samples.
- Union overlapping devices.
- Stale-window timeout.
- Tests for background, sleep, phone lock, and multi-device cases.

### Sync Correctness Failures

Risk: one device overwrites drift or mood from another.

Mitigations:

- Append-only events.
- Server-only personality writes.
- Transactional ticks.
- Idempotency keys.
- No client-submitted absolute personality state.
- Snapshot versions and conflict-free reads.

### Audio Feels Canned Or Harsh

Risk: repeated calls or poor mixing break aliveness.

Mitigations:

- Procedural motif grammar.
- Stable bird seeds plus runtime variation.
- Chorus mixer tests.
- Long-session listening QA.
- No recorded fallback.
- Conservative limiter and gain staging.

### Accessibility Becomes A Stripped Fallback

Risk: screen-reader and reduced-motion users get a flatter product.

Mitigations:

- Narration and reduced-motion built before launch, not after.
- Naturalist prose generator from same state.
- Reduced-motion visual design QA.
- Captions generated from actual calls.
- Manual assistive-tech testing.

### Notebook Becomes Generic Or Too Frequent

Risk: notebook reads like an event log or feed.

Mitigations:

- Rule-based observation eligibility.
- Global cadence caps.
- Style linting.
- Copy review with examples.
- Ban user-behavior summaries.

### Scope Creep Toward Game Or Social Network

Risk: engagement surfaces creep in through "small" features.

Mitigations:

- Explicit banned terms and components.
- Product review checklist for every new surface.
- No metrics infrastructure for leaderboards/streaks.
- Visit feature implemented as read-only scope with separate auth.

### Privacy Boundary Leakage

Risk: per-bird interaction history enters analytics or logs.

Mitigations:

- Telemetry schemas that lack fields for bird ids, personality, notebook text, and raw events.
- Log scrubbing and tests.
- Separate operational metrics from simulation database.
- Privacy review before export, deletion, and visit launch.

### First-Bird Performance Miss

Risk: loading feels like an app booting instead of a place already alive.

Mitigations:

- Embedded/early initial snapshot.
- Critical asset preload.
- Code splitting.
- Quiet field fallback, never spinner.
- Synthetic 4G/mobile tests in CI.

### Browser Audio And Lifecycle Differences

Risk: Safari/mobile audio permissions and lifecycle events break calls or presence.

Mitigations:

- Browser-specific WebAudio tests.
- Graceful silence with captions.
- Server-side stale presence timeouts.
- Page Lifecycle API handling.
- Conservative assumptions when client lifecycle events are missing.

## 16. Engineering Guardrails

Code review should reject:

- Any client write path that sets personality, mood, or canonical bird position.
- Any user-facing trait number.
- Any welcome toast, return banner, streak, achievement, badge, level, score, leaderboard, public profile, or "your friend visited" default notification.
- Any recorded call loop or recorded audio fallback.
- Any analytics event containing bird id, personality vector, notebook text, raw interaction payload, email, or visitor email.
- Any visit feature that allows visitor interaction or host drift.
- Any reduced-motion implementation that simply freezes the aviary.
- Any accessibility narration that is a mechanical state list.
- Any scene layout that pans, scrolls, zooms, crops birds, or puts chrome inside the aviary.

Definition of done for v1:

- A new user can sign in, name two starter birds, and see an already-moving aviary within the performance budget.
- The birds greet through behavior, not text.
- Presence, listen-in, offers, settle, mood, drift, calls, notebook, sync, visits, account export/deletion, and accessibility all work against server-owned canonical state.
- Multiple devices converge on the same aviary without personality conflicts.
- Visitors can only observe.
- The product has no gamification, no Tamagotchi punishment, no public social surface, and no native-app dependency.
- Accessibility users get the actual Pocket Aviary experience in an adapted form.
- Operational telemetry can run the service without turning the user's relationship with their birds into an analytics product.
