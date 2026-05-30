# Pocket Aviary v1 Implementation Plan

## 1. Scope

### In scope for v1

- Browser-only product for modern Chrome, Safari, Firefox, and Edge.
- Single-user accounts with email magic-link authentication.
- One canonical aviary per account.
- Two starter birds at onboarding, with server-controlled age-based expansion up to seven.
- One horizontal responsive aviary scene with no pan, zoom, or scroll.
- Server-side simulation tick governing canonical bird state.
- Presence-driven personality drift, mood transitions, procedural calls, idle motion, ambient weather, and day/night continuity.
- Listen-in, offer, settle, field notebook, account settings, accessibility settings, and quiet visit invitations.
- Multi-device sync by shared server-authored canonical state.
- Accessibility-first narration, reduced-motion rendering mode, keyboard support, and call captions.
- Aggregate-only operational telemetry and privacy boundaries that exclude per-bird interaction history from analytics pipelines.

### Explicitly out of scope

- Native mobile apps.
- Gamification of any kind, including streaks, achievements, scores, visit counters, badges, or leaderboards.
- Tamagotchi mechanics such as hunger, decay, punishment, visible distress, or negative drift on absence.
- Shared aviaries, co-presence, chat, comments, public discovery, profiles, follows, or social feeds.
- Customizable scenes, multi-aviary accounts, public stats, or exposing personality vectors numerically.

### Planning assumptions

- The product is implemented as a web app plus a small backend API/simulation service and relational storage.
- Exact species art direction and color tokens are delivered by design separately, but the engineering plan includes the rendering hooks and constraints they must satisfy.
- “A few minutes” for recent user activity will be calibrated during implementation and launched behind a server-configurable threshold.

## 2. Product Architecture

### Topology

- `Web client`: renders the aviary, synthesizes audio, measures eligible presence, submits interaction events, and renders accessible surfaces.
- `API service`: authenticates sessions, serves canonical snapshots, accepts interaction events, issues invite links, serves notebook data, manages account/session state.
- `Simulation worker`: runs the server-side tick, consumes ordered interaction events, updates canonical bird state, generates notebook candidates, manages weather/day-night derived state.
- `Primary database`: stores accounts, birds, canonical aviary state, interaction events, notebook entries, invites, sessions, and audit-safe operational metadata.
- `Queue/scheduler`: triggers per-account simulation ticks on cadence and queues heavier derived work such as notebook generation.
- `Email service`: sends magic links, invite links, export links, and account lifecycle emails.
- `Edge delivery`: serves HTML shell plus a small bootstrap snapshot path so first-bird rendering stays under budget.

### Service boundaries

- The client is a renderer and event producer, never a canonical state authority.
- The API service is the only ingress for account, session, snapshot, and interaction traffic.
- The simulation worker is the only writer for personality vectors, moods, call timing seeds, weather state, and derived canonical bird state.
- Analytics and RUM ingest only aggregate operational metrics; they do not read simulation tables or interaction logs with per-account semantic payloads.

### Suggested implementation shape

- Frontend: React/TypeScript SPA with SSR or edge-rendered shell for rapid first paint.
- Backend: typed application service with REST or JSON-over-HTTP endpoints plus scheduled workers.
- Storage: Postgres for canonical state and append-only event log, Redis or equivalent for short-lived caches/rate limits if needed.
- Job execution: durable queue for simulation ticks, notebook generation, exports, and email send workflows.

## 3. Client/Server Split

### Client responsibilities

- Authenticate via magic-link session bootstrap.
- Fetch the current snapshot on load, visibility regain, long suspension recovery, and periodic keepalive.
- Render scene composition, per-bird interpolation, reduced-motion variants, and top-bar surfaces.
- Track presence eligibility inputs: visibility, focus, recent pointer/key activity.
- Submit interaction events: presence heartbeat, listen-in start/end, offer, settle, notebook fetches, visit session fetches.
- Synthesize procedural calls and captions from server-provided motif/config seeds.
- Maintain local ephemeral UI state only: focus state, currently listened-in bird, local fade timers, accessibility toggles mirrored from server preferences.

### Server responsibilities

- Hold canonical aviary state and bird identity continuity.
- Order and persist interaction events.
- Calculate and apply drift deltas and mood transitions.
- Decide greeting candidates, active weather windows, day/night state, and available adoption milestones.
- Generate notebook entries from canonical events and cadence rules.
- Enforce invite, privacy, session, and export policies.

### Render pipeline boundary

- Server snapshot includes semantic/physical targets, not frame-by-frame animation data.
- Client derives smooth motion from snapshot positions, posture targets, call schedules, and ambience seeds.
- Server owns “what is true now”; client owns “how it is smoothly shown now.”
- Hidden/background tabs stop rendering and audio playback locally, but canonical time continues on the server.

## 4. Data Model

### Core entities

#### Account

- `account_id` UUID synthetic identifier.
- `email_encrypted`.
- `email_verified_at`.
- `timezone`.
- `created_at`, `deleted_at`, `hard_delete_after`.
- `settings_json`:
  - audio enabled
  - captions enabled
  - reduced-motion override
  - screen-reader narration preference
  - visit notification opt-in
- `aviary_created_at`.

#### Session

- `session_id` UUID.
- `account_id`.
- `device_label`, `user_agent_summary`.
- `created_at`, `last_seen_at`, `revoked_at`, `expires_at`.

#### AviaryState

- `account_id`.
- `version` monotonically increasing integer.
- `local_time_context` derived from account timezone and current timestamp.
- `day_phase` enum: dawn, morning, midday, evening, night.
- `settled_until` nullable timestamp.
- `active_weather` nullable struct: type, intensity, started_at, ends_at.
- `bird_count_cap_current`.
- `last_simulated_at`.

#### Bird

- `bird_id` UUID stable forever.
- `account_id`.
- `species_id`.
- `display_name`.
- `adopted_at`.
- `sort_index`.
- `personality_vector`:
  - boldness
  - social_warmth
  - vocal_frequency
  - plumage_saturation
  - curiosity
- `mood_state`.
- `mood_updated_at`.
- `perch_zone_target`.
- `pose_state`.
- `call_signature_seed`.
- `last_greeted_at`.
- `offer_cooldowns_json`.
- `visual_seed`.

#### InteractionEvent

- `event_id` ordered UUID.
- `account_id`.
- `bird_id` nullable for account-level events.
- `client_session_id`.
- `event_type` enum:
  - presence_heartbeat
  - listen_in_started
  - listen_in_ended
  - offer_seed
  - offer_song_fragment
  - offer_still_pool
  - settle_started
  - settle_cancelled
  - visit_opened
- `occurred_at_client`.
- `received_at_server`.
- `dedupe_key`.
- `payload_json` constrained by type.

#### PresenceWindow

- Materialized or derived table keyed by account/session/day to support drift accounting.
- Stores only validated windows, not raw pointer paths or keystroke contents.
- Fields: `window_started_at`, `window_ended_at`, `qualified_seconds`, `source_session_id`.

#### NotebookEntry

- `entry_id` UUID.
- `account_id`.
- `bird_ids` array.
- `entry_timestamp`.
- `entry_text`.
- `entry_reason_code` internal only.
- `salience_score`.

#### Invite

- `invite_id` UUID.
- `host_account_id`.
- `visitor_email_encrypted`.
- `token_hash`.
- `status` enum: pending, active, revoked, expired.
- `created_at`, `expires_at`, `revoked_at`, `last_used_at`.
- `notifications_enabled_at_send`.

#### VisitLog

- `visit_log_id`.
- `invite_id`.
- `host_account_id`.
- `visitor_email_redacted_for_host_view`.
- `visited_at`.
- `approx_duration_seconds`.

### Data handling rules

- Personality vectors are stored as canonical columns or typed JSON and updated only by the simulation worker.
- Email appears only on account and invite records in encrypted form; logs and downstream systems use synthetic IDs.
- No table exposes personality vector fields directly to client-facing settings or analytics.
- Notebook text is immutable once written, except for operational redaction tooling if required for abuse/security incidents.

## 5. API Surface

### Authentication and account

- `POST /v1/auth/magic-link/request`
  - Input: email.
  - Output: accepted response with rate-limit-safe generic copy.
- `POST /v1/auth/magic-link/consume`
  - Input: token.
  - Output: session token, account bootstrap, redirect target.
- `GET /v1/account`
  - Returns account settings, active sessions, visit settings summary.
- `POST /v1/account/settings`
  - Updates accessibility/audio/notification preferences.
- `POST /v1/account/export`
  - Queues export email.
- `POST /v1/account/delete`
  - Soft-deletes account.
- `POST /v1/account/delete/cancel`
  - Restores during grace period.
- `POST /v1/account/email-change/request`
  - Starts verified email change flow.

### Aviary snapshot and notebook

- `GET /v1/aviary/snapshot`
  - Returns canonical aviary snapshot with:
    - snapshot version
    - server timestamp
    - day phase
    - settled state
    - active weather
    - birds array
    - current greeting candidate hints
    - call motif configs/seeds
    - accessibility narration seed text or render descriptors
- `GET /v1/aviary/notebook?cursor=...`
  - Paginated read-only notebook entries.
- `GET /v1/aviary/bootstrap`
  - Returns minimal HTML/bootstrap payload optimized for first-bird render.

### Interaction events

- `POST /v1/aviary/events`
  - Batch endpoint for ordered append-only events.
  - Accepts dedupe keys and server acknowledges accepted IDs.
- `POST /v1/aviary/presence`
  - Optional dedicated heartbeat endpoint if presence traffic needs isolated controls; otherwise folded into events batch.

### Visit invitations

- `POST /v1/visits/invites`
  - Host creates invite for one email.
- `GET /v1/visits/invites`
  - Host lists outstanding and historical invites.
- `POST /v1/visits/invites/{inviteId}/revoke`
  - Revokes immediately.
- `GET /v1/visits/{token}/snapshot`
  - Visitor read-only snapshot, stripped of host-only controls and event endpoints.
- `POST /v1/visits/{token}/open`
  - Starts visit log timing.
- `POST /v1/visits/{token}/close`
  - Ends visit log timing best-effort.

### Error and conflict behavior

- Expired links, revoked invites, unsupported browsers, session timeout, and snapshot failures return matter-of-fact system copy.
- Client retries idempotent snapshot fetches with capped backoff.
- Event ingestion is idempotent by `dedupe_key`.

## 6. Simulation Engine Design

### Tick cadence and scheduling

- Target cadence: one simulation tick per account per minute, with jittered scheduling to avoid fleet spikes.
- Tick jobs are keyed by account and skipped if a newer completed tick already covers the time window.
- If the system falls behind, the next tick coalesces elapsed intervals rather than replaying every missed minute individually.

### Tick inputs

- Last canonical aviary state.
- Ordered interaction events since the prior processed watermark.
- Current wall clock in account timezone.
- Active settle state.
- Weather scheduler state.
- Bird age milestones and invitation state when relevant.

### Tick phases

1. Validate and fold interaction events into derived short-lived signals.
2. Update presence windows from accepted presence heartbeats.
3. Compute per-bird drift deltas with bounded monotonic increases only.
4. Advance mood state machine using recent interactions, day phase, weather, and personality weights.
5. Select new perch targets, pose tendencies, and greeting readiness.
6. Advance call scheduling seeds and chorus opportunities.
7. Evaluate notebook-worthy moments and queue/write sparse entries.
8. Persist new canonical state atomically with processed event watermark.

### Drift function

- Use a low-pass accumulation model per trait.
- Dominant signal: qualified presence seconds.
- Secondary signals:
  - sustained listen-in increases social warmth and vocal frequency for that bird
  - accepted or near-target offers slightly increase curiosity and boldness
  - settle affects mood quieting only, not long-term drift direction
- Drift never decreases traits on neglect.
- Lack of recent presence influences greeting likelihood and momentary ambience through mood/behavior layers, not personality rollback.
- Daily and weekly caps prevent a single unusually long day from compressing multi-week change into one session.

### Calibration targets

- Instrument-detectable trait movement after roughly one week of regular visits.
- Human-noticeable change after roughly three weeks.
- No visible trait swing within a single ordinary session.
- Birds remain differentiated; drift moves expression without collapsing species/personality identity.

### Mood state machine

- Base state set: wary, content, curious, drowsy, alert.
- Transition weights depend on:
  - current mood
  - personality vector
  - recent accepted offers
  - listen-in attention
  - time of day
  - current weather
  - spread effects from nearby bird calls/alarm states
- Mood persists across sessions and is stored canonically.
- Settled state biases all birds toward drowsy/quiet without overwriting deeper personality.

### Greeting runtime

- On snapshot generation for a newly visible client, server marks 1-2 plausible greeting candidates using absence length, mood, and boldness.
- Client resolves exact onset within the first second or two using those hints and live render timing.
- Multiple greeting-capable birds stagger by randomized small offsets; never unison.

### Call grammar runtime

- Each species owns a motif library and parameter ranges.
- Each bird has a stable signature seed ensuring recognizability across sessions.
- Mood and vocal frequency alter interval, density, pitch contour range, and chorus-join likelihood.
- Server shares compact descriptors; client synthesizes sound and captions from the same descriptors.

### Bird addition milestones

- New birds unlock by aviary age milestones maintained server-side.
- The unlocking surface should remain quiet and nongamified; the plan assumes a soft offer surfaced in product voice rather than any celebratory ceremony.
- Adoption preserves the “birds arrived” framing by system-selected species with user naming only.

## 7. Sync Model

### Canonical state strategy

- Server-owned canonical state with monotonically increasing snapshot version.
- Clients fetch and render snapshots, never merge personality state.
- Interaction events append to a single ordered log and are folded by the server.

### Conflict prevention

- No client writes absolute trait values.
- All interaction submission endpoints are idempotent via dedupe keys.
- Presence heartbeats are coarse-grained and collapsible to avoid overcount.
- Snapshot responses include version and server time so clients can discard stale responses.

### Multi-device behavior

- Laptop and phone can both observe the aviary concurrently.
- Either client can submit listen-in, offer, or settle events; both later snapshots converge because the server serializes events.
- If two devices issue conflicting immediate intents, the product favors last accepted interaction event for ephemeral UI state while long-term personality remains additive and ordered.
- Visitors never contribute drift or host-visible presence.

### Recovery paths

- On reconnect after suspension or offline gap, client fetches fresh snapshot before resuming motion/audio.
- If event submission fails temporarily, client stores a short bounded retry queue for idempotent resend; expired events beyond a short threshold are dropped rather than replayed indefinitely.
- Session expiry or auth failure exits to matter-of-fact re-auth flows.

## 8. Frontend Rendering Pipeline

### Scene composition

- Single scene canvas/DOM layer structure:
  - background sky/foliage gradient layer
  - middle-plane perches and birds
  - foreground occasional branch/leaf layer
  - top bar chrome outside the scene
- Three perch zones represented as semantic targets, not freeform drag positions.

### First-bird rendering

- Server or edge returns a minimal bootstrap snapshot in the initial document response or immediately adjacent fetch.
- Client renders quiet field instantly if snapshot is delayed, with no spinner.
- First bird appears already mid-action using initial pose/call descriptors, not entry animation, except the explicit post-onboarding empty-to-first-bird case.

### Motion system

- Default mode:
  - requestAnimationFrame-driven interpolation
  - low-frequency pose changes layered over small micro-motion loops
  - subtle parallax and ambient ornament generation
- Reduced-motion mode:
  - pose cross-fades instead of continuous motion
  - perch transitions cross-fade rather than animated paths
  - leaf/feather drift disabled
  - day/night color transitions slowed but retained

### Interaction rendering

- Listen-in ramps audio mix and may visually reinforce focus with subtle posture/orientation, not overt UI badges.
- Offers originate from top-bar affordance and create in-scene props only as long as needed.
- Settle warms/dims lighting over several seconds with five-second undo window.
- Top bar fades to near transparency on pointer/keyboard stillness and returns on activity.

### Responsive behavior

- Preserve all birds in frame across phone and desktop widths.
- Use layout rules that widen spacing on large screens and compress perch distances on narrow screens without cropping.
- Keep hit targets accessible on touch and keyboard without adding in-scene chrome.

## 9. Audio Pipeline

### Core design

- WebAudio-only procedural synthesis from species motif libraries.
- One audio engine instance per tab with bounded voices and reusable buffers/nodes.
- Bird-specific signature seeds preserve recognizable timbre/rhythm families.

### Signal flow

- Snapshot descriptors feed a scheduler that plans likely call windows for the next short horizon.
- Synth graph generates envelopes, pitch contours, and subtle timbral variation at playback time.
- Listen-in adjusts gain nodes gradually:
  - focused bird rises
  - others attenuate but remain audible
- Weather and settle affect global mix gently.

### Chorus handling

- The engine supports overlapping calls without phasey loop artifacts because calls are generated per event.
- Mixer limits concurrency to preserve clarity at seven birds.
- Spatialization stays subtle; do not turn the aviary into an exaggerated stereo toy.

### Caption coupling

- Each generated call event also emits a structured descriptor translated to prose captions.
- Caption text matches actual runtime call shape, not a canned label bank.

### Fallback behavior

- If WebAudio is unavailable or blocked, run in silence with captions defaulted on.
- No recorded-audio fallback path is implemented.

## 10. Accessibility Surfaces

### Screen-reader experience

- Provide a dedicated narration region with paced naturalist prose updates.
- Prioritize user-triggered event narration over idle updates.
- Keep idle narration cadence around 30-60 seconds to avoid queue overload.
- Use the same semantic inputs as visual rendering so narration and visuals describe the same aviary moment.

### Keyboard and focus

- Tab order:
  - top bar controls
  - entry into aviary scene
  - per-bird navigation with arrow keys
- Enter toggles listen-in on the focused bird.
- Escape exits listen-in or closes transient panels.
- Focus outlines are high-contrast and visible in all day/night states.

### Captions and visual text

- Caption toggle in accessibility settings.
- Caption placement follows calling bird while avoiding overlap collisions where possible.
- All text surfaces meet WCAG AA contrast.

### Reduced-motion and audio-off users

- Respect `prefers-reduced-motion` on first visit and persist explicit overrides.
- Audio-off or hardware-muted users can still access captioning and notebook/noticing surfaces without degradation in product voice.

### Testing gates

- Screen-reader smoke coverage for VoiceOver and NVDA-equivalent workflows.
- Keyboard-only navigation tests in CI.
- Automated contrast checks on text-bearing surfaces.
- Manual review of narration cadence and prose quality before launch.

## 11. Performance Budgets and Observability

### Budgets

- Initial JS bundle under 2 MB gzipped.
- First bird visible under 500 ms on a mid-tier mobile device over 4G.
- Idle motion at 60 fps on a five-year-old mid-range laptop.
- No client memory growth over 30-minute sessions.
- Simulation tick latency p99 under 5 seconds, alarmed at or above threshold.

### Performance tactics

- Split settings, invite, and export flows out of the critical path.
- Ship compact motif/config data instead of heavy media.
- Reuse WebAudio nodes and rendering objects.
- Keep snapshot payloads small and versioned.
- Precompute or cache derived greeting/weather/day-phase descriptors server-side where it reduces tick cost.

### Observability

- Aggregate RUM:
  - first-bird render
  - snapshot fetch latency
  - frame timing summaries
  - audio engine init failures
  - unsupported browser rate
- Backend metrics:
  - tick duration
  - tick backlog depth
  - event ingestion latency
  - email send success/failure
  - invite revoke propagation delay
- Synthetic checks from multiple geographies to validate startup and snapshot delivery.

### Privacy boundary in telemetry

- Metrics never include bird names, personality vectors, notebook text, or per-account interaction histories.
- Aggregation keys use anonymized/sessionless buckets only.

## 12. Rollout Plan

### Phase A: internal vertical slice

- Deliver end-to-end flow for account creation, two starter birds, snapshot rendering, basic mood/call pipeline, and read-only notebook scaffolding.
- Validate first-bird startup, simulation tick correctness, and presence measurement honesty.

### Phase B: private alpha

- Limited internal/external friendly testers with small account counts.
- Enable listen-in, offers, settle, captions, reduced motion, and notebook sparsity tuning.
- Instrument drift calibration review weekly using internal tools that show trait movement to operators only.

### Phase C: closed beta

- Introduce visit invitations, session management, export, and deletion/recovery flows.
- Ramp bird count unlock milestones gradually:
  - launch all users with two birds
  - unlock third bird only after milestone confidence
  - raise to broader age-based expansion after audio recognizability and performance remain healthy

### Phase D: v1 launch

- Open self-serve onboarding with guardrails on invite volume and export load.
- Keep operational kill switches for:
  - invite creation
  - notebook generation
  - audio descriptor complexity
  - bird unlock milestones

### Day-one instrumentation

- Track startup speed, tick health, invite flow reliability, session failures, unsupported browsers, narration toggle usage, caption usage, and reduced-motion usage.
- Do not instrument bird-specific behavioral dashboards for product analytics.

## 13. Risks and Mitigations

### Drift calibration risk

- Risk: birds change too quickly and feel gamey, or too slowly and feel static.
- Mitigation: tune with internal operator dashboards, daily caps, per-signal weighting controls, and staged rollout of new-bird milestones.

### Sync correctness risk

- Risk: duplicated or missing interaction events lead to lost drift or inconsistent moods across devices.
- Mitigation: idempotent event API, ordered watermarks, server-only personality writes, snapshot versioning, and reconciliation alerts on version regressions.

### Audio uncanniness risk

- Risk: procedural calls sound synthetic, repetitive, or overly musical.
- Mitigation: species-specific motif libraries, seed-based variation, human ear review sessions, and synthesis parameter tuning before expanding species pool.

### Accessibility regression risk

- Risk: accessible surfaces become state dumps or lag behind visual behavior.
- Mitigation: treat narration/captions/reduced-motion as primary deliverables in definition of done, not follow-up polish.

### Performance risk

- Risk: first-bird target slips due to heavy bundle, snapshot bloat, or render thrash.
- Mitigation: budget checks in CI, bootstrap payload testing, and hard ownership of bundle budgets by feature teams.

### Privacy/compliance risk

- Risk: engineers accidentally pipe PII or per-bird history into analytics/logs.
- Mitigation: synthetic account IDs everywhere, schema linting for telemetry payloads, code review checklist, and restricted warehouse access with no simulation-table replication.

### Product tone risk

- Risk: harmless-looking toasts, counters, or celebratory UI elements dilute “notice, never announce.”
- Mitigation: central UX review against explicit non-goals and tone rules for any new surface, including system/product voice distinction.

## 14. Delivery Sequencing

### Workstream 1: platform and auth

- Account model, magic-link auth, sessions, deletion/export primitives, settings scaffold.

### Workstream 2: canonical simulation

- Bird/personality schema, event log, simulation tick, mood engine, age milestones, weather/day-phase state.

### Workstream 3: client rendering

- Bootstrap shell, scene renderer, bird interpolation, top bar, settle flow, notebook reader, responsive behavior.

### Workstream 4: audio and captions

- Motif libraries, synthesis engine, listen-in mixing, caption generation, silent fallback.

### Workstream 5: accessibility

- Narration engine, keyboard model, reduced-motion renderer, contrast validation, QA scripts.

### Workstream 6: social and account utilities

- Invite issuance/revocation, visitor snapshot path, visit logs, export email flow.

### Suggested critical path

- Auth and canonical snapshot path first.
- Then simulation worker and starter birds.
- Then rendering/audio startup experience.
- Then interaction events and notebook generation.
- Then accessibility hardening and visit invitations.
- Then bird-unlock expansion and final calibration.

## 15. Testing and Acceptance

### Functional acceptance

- A new user can sign in, name two starter birds, and see an aviary already in motion.
- Returning after absence produces one procedural greeting within the first seconds and no textual welcome.
- Presence is recorded only when visibility, focus, and recent activity all qualify.
- Two devices show the same canonical aviary state after snapshot refresh.
- Visitors can observe but cannot drift or mutate host state.

### Nonfunctional acceptance

- Performance budgets pass on representative devices/networks.
- Long session memory tests remain flat.
- Screen-reader and reduced-motion paths preserve product tone.
- Privacy review confirms no per-account bird history in telemetry or logs.

### Operational readiness

- Dashboard and alerting coverage for tick lag, snapshot errors, audio failures, and email flows.
- Runbooks for expired-link spikes, invite revoke issues, and backlog growth.

## 16. Open Decisions to Resolve During Build

- Exact recent-activity timeout for presence qualification.
- Exact minute cadence and jitter strategy for simulation scheduling.
- Whether notebook generation is inline during tick or queued from salience candidates.
- Whether bootstrap snapshot is embedded in SSR HTML or fetched from an ultra-low-latency endpoint.
- Final species pool and motif-library production workflow with design/audio collaborators.

This plan intentionally keeps those questions bounded to implementation choices while preserving the PRD’s non-negotiable product rules: server-owned continuity, monotonic expressive drift, quiet social affordances, naturalist accessibility, and no gamified or custodial mechanics.
