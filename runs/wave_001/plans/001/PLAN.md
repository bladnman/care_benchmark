# Pocket Aviary v1 Implementation Plan

## Planning posture

This plan interprets the PRD into an executable v1 build program for a frontier engineering team. Where the PRD leaves implementation details open, this plan makes conservative choices that preserve the core product promises: the aviary feels continuously alive, presence is the dominant input, the server owns canonical state, and no surface drifts into game, notification, or social-network patterns.

## 1. Scope

### In scope for v1

- Web-only product for modern browsers on desktop and mobile web.
- Email magic-link authentication with one aviary per account.
- Two starter birds at account creation; automatic age-based expansion up to a hard cap of seven birds.
- Single horizontal aviary scene with three perch zones, day/night cycle, rare ambient weather, and continuous ambient motion.
- Server-side simulation tick that advances canonical mood, drift, and activity whether or not a client is open.
- Hidden per-bird personality vectors and visible per-bird moods.
- User interactions: passive presence, return-greeting, listen-in, offer, settle, notebook viewing.
- Field notebook with sparse, auto-generated naturalist observations.
- Multi-device sync by reading the same server-authored canonical state.
- Accessibility surfaces: screen-reader narration, reduced-motion mode, call captions, keyboard navigation, WCAG AA text contrast.
- Optional quiet social layer: per-invite read-only visits, invite revocation, visit log, optional host notifications off by default.
- Aggregate operational telemetry only, with hard exclusion of per-account simulation history from analytics pipelines.

### Explicitly out of scope

- Native iOS/Android apps.
- Scores, streaks, achievements, counters, levels, rewards, or any other gamification.
- Hunger, health, distress, death, or any punitive neglect mechanic.
- Shared aviaries, co-presence, public discovery, comments, profiles, follows, or ranking.
- Scene customization, bird-stat dashboards, multiple aviaries per user, push notifications, recorded-audio fallback, or public-facing “show-off” rendering.

### Release framing

V1 should ship as a tightly scoped, high-quality experience rather than a platform. Protecting tone and architectural correctness is more important than expanding surface area. A feature that weakens aliveness, sync correctness, or privacy should be cut even if technically feasible.

## 2. Product architecture

### System shape

Implement Pocket Aviary as three production services plus a web client:

1. `web-client`
   Browser app responsible for rendering the aviary, synthesizing procedural audio, collecting presence/interactions, and presenting account/settings/accessibility/social surfaces.
2. `aviary-api`
   Stateless application API responsible for auth/session handling, snapshot reads, event ingestion, notebook reads, invitation management, export/delete requests, and settings.
3. `simulation-service`
   Server-owned tick processor responsible for consuming ordered event logs, updating canonical aviary state, applying mood/drift transitions, generating notebook candidates, and publishing fresh snapshots.
4. `notification-worker`
   Background worker for magic-link email, invitation email, export-link email, and optional visit notifications.

Use a relational primary datastore for canonical account/aviary/bird state plus an append-only interaction-event table/stream. Keep the architecture intentionally simple: one canonical source of truth and one simulation writer.

### Client/server split

- Server owns all persistent simulation state, invitation state, notebook state, and account state.
- Client owns ephemeral UI state only: hover/focus, current audio mix ramp, open panels, temporary reduced-motion transitions, optimistic affordance state for event submission.
- Client never computes or stores authoritative personality updates.
- Client may interpolate between server snapshots for smooth visual motion, but interpolation must be purely presentational.

### Rendering boundary

- Server snapshot includes semantic state: bird identities, current perch targets, motion state keys, mood, call timing seeds/windows, ambient state, settle state, and notebook entries.
- Client rendering engine maps semantic state into live motion, audio, captions, narration, and responsive layout.
- Ambient ornaments that do not affect semantics, such as leaves/feathers drift, remain client-only and non-persistent.

## 3. Data model

### Core entities

#### Account

- `account_id` UUID synthetic primary key.
- `email_encrypted`
- `email_verified_at`
- `status` (`active`, `pending_deletion`, `deleted`)
- `created_at`, `updated_at`, `scheduled_hard_delete_at`
- `timezone`
- `feature_flags`

#### Session

- `session_id`
- `account_id`
- `device_label`
- `created_at`, `last_seen_at`, `revoked_at`, `expires_at`

#### Aviary

- `aviary_id`
- `account_id`
- `created_at`
- `current_scene_state` summary fields for latest snapshot
- `settled_until` or `settled_state` marker
- `bird_capacity_current`

#### Bird

- `bird_id` stable UUID
- `aviary_id`
- `species_id`
- `display_name`
- `created_at`
- `active_status`
- `personality_vector`
  - `boldness`
  - `social_warmth`
  - `vocal_frequency`
  - `plumage_saturation`
  - `curiosity`
- `current_mood`
- `mood_updated_at`
- `current_perch_zone`
- `call_signature_seed`
- `visual_seed`
- `offer_cooldowns`

#### Presence session / ledger

- `presence_window_id`
- `account_id`
- `session_id`
- `started_at`
- `ended_at`
- `qualified_seconds`
- `source_conditions_summary`

This is derived from presence pings but stored explicitly for auditability and drift processing.

#### Interaction event

- `event_id`
- `account_id`
- `aviary_id`
- `bird_id` nullable when not bird-specific
- `session_id`
- `event_type`
  - `presence_ping`
  - `listen_in_start`
  - `listen_in_end`
  - `offer_seed`
  - `offer_song_fragment`
  - `offer_still_pool`
  - `settle_start`
  - `settle_undo`
  - `client_visible`
  - `client_hidden`
- `occurred_at`
- `client_context`
  - visibility/focus/activity booleans
  - viewport class
  - reduced-motion/audio-caption flags when relevant
- `idempotency_key`

#### Notebook entry

- `entry_id`
- `aviary_id`
- `created_at`
- `entry_text`
- `trigger_type`
- `bird_ids`
- `salience_score`

#### Invitation

- `invitation_id`
- `host_account_id`
- `visitor_email_encrypted`
- `token_hash`
- `created_at`
- `expires_at`
- `revoked_at`
- `first_used_at`
- `last_used_at`
- `notification_pref_snapshot`

#### Visit session

- `visit_session_id`
- `invitation_id`
- `host_account_id`
- `visitor_email_encrypted`
- `started_at`
- `ended_at`
- `approx_duration_seconds`

#### Accessibility and product settings

- `account_id`
- `reduced_motion_override`
- `call_captions_enabled`
- `audio_enabled`
- `visit_notifications_enabled`

### Data governance rules

- Never use email as a foreign key, analytics dimension, log identifier, or partition key.
- Personality vectors are stored canonical state, not derived views.
- Telemetry schema must exclude notebook text, bird names, personality values, and per-account interaction history.

## 4. API surface

### Auth and account

- `POST /auth/magic-link/request`
  Accept email, rate-limit, send link.
- `POST /auth/magic-link/consume`
  Validate single-use token, issue session.
- `GET /account`
  Return matter-of-fact account/settings surface data.
- `POST /account/email-change/request`
- `POST /account/email-change/confirm`
- `POST /account/export`
  Queue export and email secure link.
- `POST /account/delete`
  Soft-delete request.
- `POST /account/delete/cancel`

### Aviary state

- `GET /aviary/snapshot`
  Returns current canonical snapshot with version token and minimal data needed for first render.
- `GET /aviary/notebook?cursor=...`
  Paginated read-only notebook.
- `GET /aviary/settings`
  Accessibility/social/account-adjacent aviary settings if separated from account object.

### Event ingestion

- `POST /aviary/events/batch`
  Accept ordered interaction events with idempotency keys. Server validates event schema, rejects impossible sequences, persists append-only.

Batch events to reduce network cost, but flush immediately for state-shaping actions like listen-in start, offer, and settle.

### Social

- `POST /visits/invitations`
  Create per-email invite and send email.
- `GET /visits/invitations`
  List outstanding/revoked/expired invites.
- `POST /visits/invitations/{id}/revoke`
- `GET /visits/log`
  Host-only read of visit history.
- `GET /visits/{token}/snapshot`
  Read-only ambient snapshot for visitor.

Visitor snapshot endpoint must never accept event writes and must never create presence events against the host account.

### Operational contracts

- Snapshot responses include `snapshot_version`, `generated_at`, and a short horizon of motion/audio seeds so the client can render instantly.
- Event endpoint is idempotent per event key and safe against replay.
- Conflict/error responses use matter-of-fact copy payloads, not product-surface prose.

## 5. Simulation engine design

### Tick cadence and execution model

- Run canonical tick approximately once per minute per active aviary shard.
- Process all new ordered events since last tick.
- Apply time-of-day and ambient-state advancement even with no recent events.
- Persist a new canonical snapshot only when state changes, but keep a heartbeat timestamp for continuity.

Use shardable scheduled jobs rather than per-account cron rows. The system should support millions of quiet aviaries by updating only what is due and what has changed meaningfully.

### Drift function

Implement drift as additive server-authored deltas with saturation guards:

- Presence-time provides the dominant positive signal.
- Listen-in contributes targeted uplift to `social_warmth` and `vocal_frequency`, with smaller curiosity lift if sustained.
- Offers contribute small targeted lift based on offered item and actual reaction.
- Neglect never causes negative trait deltas.

Recommended initial model:

- Maintain per-bird rolling exposure counters over 1-day, 7-day, and 21-day windows.
- Convert counters into small normalized drift deltas using sigmoid or bounded linear ramps.
- Apply deltas only upward toward per-trait soft caps to preserve slow visible change without runaway saturation.

Calibration target:

- Instrument-detectable change after roughly one week of regular presence.
- User-perceivable behavioral change after roughly three weeks.
- No single session causes visibly discontinuous personality movement.

### Mood model

Represent mood as enumerated state plus momentum:

- States: `wary`, `content`, `curious`, `drowsy`, `alert`.
- Maintain transition weights based on:
  - recent accepted/ignored offers
  - current local time bucket
  - ambient weather
  - neighboring bird state spillover
  - personality trait modifiers

Use a weighted transition matrix with hysteresis so moods do not snap on every tick. Mood should feel persistent and legible through motion, not jittery.

### Bird-to-bird interaction

At each tick, evaluate lightweight social coupling:

- high `vocal_frequency` birds increase chorus probability windows
- wary birds raise nearby wary likelihood
- high `social_warmth` birds increase return-call probability

Keep coupling local and bounded. The product needs a small social system, not chaotic flock simulation.

### Greeting selection

On visible session re-entry:

- Compute absence duration category.
- Rank candidate greeter birds by boldness, mood, recent greeting frequency, and whether another bird greeted recently.
- Choose one primary greeter, then optionally a staggered secondary responder.

Generate greeting plans semantically server-side so the client renders a real, state-informed return rather than a canned local animation.

### Call grammar runtime

- Each species owns a motif library and synthesis parameter envelope.
- Each bird owns a stable call seed preserving recognizability.
- Mood and personality modify timing, density, pitch spread, envelope sharpness, and chorus responsiveness, but not identity-defining motifs.

Server snapshot should provide call schedule hints or random seeds for the next short window; client synthesizes actual audio and captions from that structure.

### Notebook generation

Notebook is not an event log. Implement a salience pipeline:

1. Detect noteworthy state patterns and longitudinal contrasts.
2. Enforce sparsity thresholds.
3. Render naturalist prose from templates plus state-specific details.
4. Deduplicate against recent entries.

Candidate triggers:

- unusual greeting order
- long quiet stretch
- weather-driven hush
- repeated perch preference emerging
- first response pattern in a while

Use rule-based generation for v1, not LLM generation. Determinism, privacy, and prose control matter more than expressive breadth initially.

## 6. Sync and consistency model

### Canonical-state rule

- Simulation service is the only writer of bird personality, mood, and persistent aviary state.
- API service persists client events append-only.
- Clients only read snapshots and send events.

### Snapshot propagation

- On initial load, server returns latest snapshot embedded or fetchable with minimal latency.
- While visible, client polls at low frequency, such as every 30-60 seconds, with immediate refresh on visibility regain or long suspend.
- Optionally add server-sent events or websocket fanout later, but polling is sufficient for v1 and simpler to harden.

### Conflict prevention

- Use append-only ordered events plus server tick offsets.
- Every event batch includes session id, sequence number, and idempotency key.
- Server rejects stale impossible transitions only for invariants, but generally accepts duplicates safely and deduplicates by key.

There is no last-write-wins state merge because clients never submit state snapshots.

### Cross-device behavior

- Two devices can both observe and emit events.
- If both are active, both contribute events to the same log.
- Tick processes by event time plus ingestion order tie-break.
- Presence qualification remains per session, then contributes to the same account-level drift model.

This preserves shared canonical continuity without requiring client-to-client coordination.

## 7. Frontend rendering pipeline

### Rendering architecture

- Use a declarative scene graph with semantic bird actors and lightweight animation state machines.
- Separate state ingestion from rendering: snapshot reducer -> render model -> animation/audio layers.
- Keep first render path minimal: draw scene, place birds, start ambient motion, then hydrate less critical chrome.

### Scene composition

- One horizontal responsive scene with front/middle/back perch zones.
- Responsive layout engine repositions perch anchors by viewport class while preserving all birds onscreen.
- Top bar is a separate chrome layer that fades with inactivity.

### Motion system

- Idle micro-motion comes from bird-specific pose loops selected by current mood and species.
- Movement between perches uses smooth interpolated trajectories.
- Ambient ornaments run on client timer loops independent from simulation tick.

Motion must start from a mid-action pose on first visible frame. Never gate the first meaningful frame on “intro” animation completion.

### Reduced-motion rendering

- Replace continuous motion loops with pose-state cross-fades.
- Replace flight paths with perch-to-perch dissolves.
- Remove leaf/feather drift.
- Slow color transitions while retaining day/night readability.

Build reduced-motion as a first-class render mode, not a CSS toggle layered onto the normal pipeline.

### Loading and empty states

- Loading state is the quiet field, not a spinner.
- Empty-aviary state exists only between signup/adoption completion and first bird arrival.
- Cold-load path should degrade gracefully without breaking the conceit that the aviary was already ongoing.

## 8. Audio pipeline

### Procedural synthesis

- Use WebAudio oscillators/noise/envelopes/filters plus species motif definitions.
- Compile call motifs into runtime envelopes with bird-specific seeds.
- Mix ambient aviary bus plus per-bird buses with slow gain ramps.

### Listen-in mix

- Focused bird gets gradual gain increase and slightly enhanced spatial clarity or EQ presence.
- Other birds attenuate but remain audible.
- Enter/exit ramps must be slow enough to read as attention, not track switching.

### Chorus mixing

- Support overlapping calls without clipping or phase artifacts by generating each call independently.
- Add conservative voice-count limits and dynamic range compression to prevent harsh stacking when many birds call.

### Caption and fallback integration

- Caption generator reads the same procedural call description that synthesis uses.
- If WebAudio is unavailable, default to silence plus captions-on and preserve the rest of the aviary unchanged.

### Performance constraints

- Preallocate/reuse audio nodes where feasible.
- Bound concurrent active synthesis voices.
- Verify no memory growth over 30-minute sessions.

## 9. Accessibility surfaces

### Screen-reader narration

- Provide a dedicated narration region fed by cadence-controlled naturalist prose updates.
- Idle updates every 30-60 seconds; priority updates on return-greeting, offer reaction, settle, and other user-triggered moments.
- Narration should summarize the current aviary scene, not expose hidden stats or raw coordinates.

Implement narration generation from the same semantic snapshot used by visual rendering. Do not maintain a separate accessibility-only truth model.

### Keyboard model

- Tab sequence: top bar items -> aviary entry -> birds -> modal/panel content.
- Arrow keys move between birds based on scene order.
- Enter toggles listen-in.
- Escape exits listen-in or closes active panel.
- Offer UI, notebook, settings, and settle must all be fully keyboard reachable.

### Visual accessibility

- WCAG AA contrast for all text and focus indicators.
- Focus outlines must remain visible across day/night and weather conditions.
- Captions positioned near calling birds without obscuring core scene readability.

### User preference persistence

- Respect `prefers-reduced-motion` on first load.
- Persist explicit user overrides server-side by account and locally pre-auth when needed.
- Persist captions/audio preferences consistently across devices.

## 10. Performance budgets and observability

### Hard budgets

- Initial JS bundle under 2 MB gzipped.
- First bird visible under 500 ms on target mid-tier mobile over 4G.
- Idle motion at 60 fps on a five-year-old mid-range laptop.
- No client memory growth over 30 minutes.
- Simulation tick p99 under 5 seconds.

### Engineering tactics

- Code-split settings, account, invitation, and notebook history surfaces.
- Keep snapshot payload compact and seed-based.
- Prefer compact SVG/procedural visual assets over large bitmap atlases.
- Avoid heavyweight animation frameworks that increase bundle size or runtime overhead.
- Preconnect/CDN-cache initial HTML plus snapshot bootstrap.

### Observability

- Synthetic monitoring for first-bird timing, unsupported-browser rate, and snapshot latency from multiple geographies.
- Aggregate-only RUM for load time, render frame timing, audio failures, and visibility-resume delays.
- Service metrics for event-ingest latency, tick duration, email delivery failures, and invitation revoke propagation.

### What not to measure

- Do not aggregate per-bird trait histories across users.
- Do not pipe notebook text, bird names, or interaction sequences into analytics warehouses.
- Do not create engagement dashboards around visit streaks, bird counts per user, or similar gamification-adjacent metrics.

## 11. Security, privacy, and compliance posture

- Encrypt email and invitation-recipient addresses at rest.
- Use synthetic UUIDs for all internal joins/logs.
- Enforce short-lived signed download links for exports.
- Rate-limit magic-link and invite issuance.
- Log access to export/delete flows and administrative actions without exposing simulation content.
- Segregate simulation data plane from aggregate telemetry plane.

For deletion:

- Soft-delete account immediately.
- Block ordinary sign-in surfaces except recovery path during 30-day grace period.
- Hard-delete canonical state, event logs, notebook, invitations, visit logs, and telemetry keyed to account after grace window.

## 12. Delivery plan

### Phase A: foundations

- Stand up account/session model, magic-link flow, canonical aviary schema, event log, and snapshot endpoint.
- Implement starter-bird creation and static scene rendering with quiet field loading state.
- Build minimal simulation tick skeleton with time-of-day advancement only.

Exit criteria:

- A signed-in user can load an aviary with two named birds from canonical server state on two devices.

### Phase B: core aliveness loop

- Add personality vectors, mood engine, perch decisions, greeting selection, and presence qualification.
- Implement client motion renderer and semantic interpolation.
- Add basic procedural call synthesis and ambient mixing.

Exit criteria:

- Aviary appears mid-motion, birds greet on return, and server-side state advances correctly during client absence.

### Phase C: interactions and notebook

- Implement listen-in, offers, settle, and cooldown logic.
- Add notebook trigger detection and prose generation.
- Add top bar fade and notebook surface.

Exit criteria:

- User can complete the core session loop and receive sparse, high-quality notebook entries.

### Phase D: sync hardening and accessibility

- Harden multi-device event ingestion/idempotency.
- Add screen-reader narration, reduced-motion mode, captions, keyboard navigation, and focus treatment.
- Run performance and memory profiling.

Exit criteria:

- Accessibility surfaces feel product-complete, and all published budgets are on track.

### Phase E: quiet social and account polish

- Add invitations, visitor ambient view, revocation, visit log, export, deletion, and session management.
- Finalize matter-of-fact account/error surfaces.

Exit criteria:

- Optional social feature works read-only and does not affect host simulation.

### Phase F: launch hardening

- Synthetic monitoring, alarms, cross-browser certification, chaos testing for tick outages, privacy review, and copy QA.

Exit criteria:

- Privacy boundary verified, telemetry audited, and performance stable at target.

## 13. Rollout strategy

### Launch sequence

1. Internal dogfood with synthetic accounts and privacy-safe fixtures.
2. Small invite-only alpha focused on aliveness, audio uncanniness, and reduced-motion/screen-reader validation.
3. Controlled beta with operational telemetry and manual review of notebook prose quality.
4. Public v1 once performance, sync correctness, and accessibility pass criteria are stable.

### Feature flags

- Bird-cap ramp flag for gradual expansion from 2 to 3+ birds by aviary age.
- Social invitations flag.
- Audio synthesis tuning flags by species family.
- Narration cadence and notebook sparsity tuning flags.

### Day-one instrumentation

- First bird render success rate
- Snapshot latency
- Tick duration
- Audio error rate
- Unsupported browser rate
- Invite email delivery
- Export/delete completion health

Do not instrument qualitative ranking of birds or user-behavior loops that could be repurposed into gamification.

## 14. Testing strategy

### Unit and property tests

- Drift monotonicity: positive presence never lowers traits; neglect never creates negative deltas.
- Mood hysteresis: no rapid oscillation under steady conditions.
- Event deduplication and ordering correctness.
- Invite token expiry/revocation behavior.
- Call caption generation matches procedural grammar branch actually played.

### Integration tests

- Multi-device concurrent sessions converge to the same canonical state.
- Visibility-hide/resume reload path restores current snapshot correctly after long suspend.
- Visitor sessions never create host presence or interaction events.
- Deletion hard-purge removes canonical and derived records after grace window.

### Performance tests

- First-bird render benchmark on representative mobile profile.
- 30-minute memory soak with audio on and notebook interaction.
- Tick throughput benchmark under high active-account load.

### Accessibility QA

- Screen-reader sessions on supported browser/AT combinations.
- Reduced-motion visual QA for continuity and comfort.
- Full keyboard-only traversal.
- Contrast and focus visibility testing across time-of-day states.

## 15. Major risks and mitigations

### Risk: drift calibration feels fake or too fast

If drift moves too quickly, the birds read as reactive UI. If too slowly, the relationship feels inert.

Mitigation:

- Gate release on explicit calibration studies with synthetic and human review.
- Expose tuning coefficients via config, not code edits.
- Instrument drift deltas internally without surfacing traits to users.

### Risk: sync correctness silently corrupts birds

The worst failure is losing or overwriting personality continuity.

Mitigation:

- Single writer for canonical state.
- Append-only event log with idempotency keys.
- Audit jobs checking invariant continuity for bird IDs and trait histories.
- Backup/restore drills for canonical state.

### Risk: audio feels canned or harsh

Procedural audio is central and easy to get almost-right in a way users still perceive as fake.

Mitigation:

- Prototype species motifs early.
- Run listening tests before feature-complete UI work.
- Prefer fewer, better voices over ambitious synthesis breadth.

### Risk: accessibility surfaces ship as degraded fallbacks

The product fails if reduced-motion or screen-reader users get a hollow variant.

Mitigation:

- Build narration/reduced-motion in the same milestone as core rendering.
- Add release blockers tied to accessibility delight, not only compliance boxes.

### Risk: quiet social becomes a privacy leak or engagement wedge

Invitations can easily drift toward discovery or implicit tracking.

Mitigation:

- Keep social endpoints and schemas sharply separate.
- Require explicit invite creation for every visitor.
- Default notifications off and avoid badges.
- Review every social copy and telemetry field against the quiet-social rule.

### Risk: performance misses break the “already alive” conceit

A slow first render or janky long session immediately weakens the product promise.

Mitigation:

- Treat first-bird timing as a launch-blocking metric.
- Budget bundle weight continuously in CI.
- Run soak tests and browser-matrix profiling before beta expansion.

## 16. Open implementation choices resolved by this plan

- Use polling plus visibility-based refresh for v1 sync rather than websockets, because canonical correctness matters more than live multiplayer behavior.
- Use rule-based notebook prose generation for v1 rather than LLM generation, to preserve control, privacy, and sparsity.
- Use weighted state machines for mood and bounded additive drift for personality, not opaque learned models.
- Treat reduced-motion as a distinct render mode owned by the rendering layer.
- Keep all bird identity, drift, and mood continuity server-authored and persistent from day one.

## 17. Definition of done for v1

Pocket Aviary v1 is done when a user can sign in on browser devices, meet two birds that feel distinct and alive, return days later to a coherent evolving aviary, quietly interact through presence/listen-in/offer/settle, read sparse notebook observations, optionally invite a friend to observe read-only, and do all of this with strong accessibility support, performant rendering, and no surface that feels like a game, a notification system, or a social network.
