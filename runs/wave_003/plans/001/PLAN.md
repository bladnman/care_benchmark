# Pocket Aviary v1 Implementation Plan

## Planning stance

This plan makes a few explicit calls where the PRD leaves room for interpretation:

- Canonical aviary state is server-owned end to end. Clients never write personality, mood, or bird state directly.
- The simulation has two server-side layers: immediate transient scene reactions for responsiveness, and a slower once-per-minute durable tick for drift and mood evolution.
- Snapshot delivery is pull-based, not client-authoritative sync. Host and visitor clients poll small authoritative snapshots and interpolate locally.
- Notebook prose, call captions, and screen-reader narration are generated from deterministic product logic, not an external LLM. That keeps voice consistent, latency low, and private bird history out of third-party systems.
- V1 should favor a compact, custom render/audio stack over heavyweight engines so the bundle and first-bird budgets remain achievable.

## Scope

### In scope for v1

- Web-only product for modern Chrome, Safari, Firefox, and Edge.
- Single-user account model with email magic-link auth.
- One canonical aviary per account.
- Two starter birds, age-based expansion up to a configurable max of seven.
- Continuous server-side simulation with durable bird identity, personality drift, mood persistence, ambient weather, and time-of-day behavior.
- Host interactions: watch, listen-in, offer, settle, notebook reading, account/settings, accessibility settings.
- Multi-device sync from a single authoritative aviary state.
- Quiet social feature: invite-by-email read-only visits, revocable and off by default.
- Accessibility surfaces that are first-class product surfaces: narration, reduced-motion mode, captions, keyboard operation, WCAG AA copy contrast.
- Aggregate-only operational telemetry, synthetic performance monitoring, privacy-preserving exports and deletion.

### Explicitly out of scope

- Native apps, desktop wrappers, or separate mobile clients.
- Shared ownership of an aviary, co-presence, chat, public discovery, profiles, comments, follows, leaderboards, or feeds.
- Scores, streaks, achievements, badges, counters, or any user-facing engagement metric.
- Tamagotchi mechanics: hunger, decay, bird death, distress, punishment for absence.
- User-facing personality numbers, bird stats dashboards, or any trait optimization surface.
- Scene customization, multiple aviaries per account, catalog-based bird picking, or payments.

## Architecture

### Service shape

Ship four runtime services plus static delivery:

- `web-shell`: serves the SSR HTML shell, inline initial snapshot payload, top-bar/settings UI bundle, and static scene assets through a CDN.
- `aviary-api`: handles auth, session management, snapshot reads, event ingestion, notebook queries, settings, export/delete flows, invite issuance, visit reads, and revocation.
- `simulation-worker`: owns canonical state mutation. It applies immediate transient reactions on accepted interaction events and runs the durable per-aviary minute tick.
- `mail-worker`: sends magic links, visit invites, export links, deletion confirmations, and email-change verification.
- `ops-metrics`: aggregate-only metrics and synthetic monitoring sinks, physically separated from per-account simulation tables.

For v1, keep the persistent backend simple:

- PostgreSQL as the primary store for accounts, birds, state snapshots, invites, notebook entries, and append-only interaction events.
- A durable job queue for per-aviary due ticks, email jobs, deletion deadlines, and export generation.
- Object storage only if needed for exported JSON bundles; no media CDN for bird audio because calls are synthesized client-side.

### Client/server split

Server responsibilities:

- Own all canonical aviary state: bird identity, personality vectors, mood, perch targets, time-of-day state, weather windows, notebook entries, invite state, visit logs, and deletion/export state.
- Accept host interaction events and visitor read requests.
- Compute transient reaction state immediately after host events.
- Advance durable simulation state on the minute tick.
- Enforce privacy, invite revocation, rate limits, and deletion semantics.

Client responsibilities:

- Render the current scene from authoritative snapshots.
- Interpolate motion between authoritative targets.
- Synthesize procedural audio from server-provided call descriptors.
- Manage local UI affordances: top-bar fade, keyboard focus movement, local notebook scrolling, captions, and settings presentation.
- Detect presence signals and send them as events; never infer drift locally.

### Render pipeline boundary

The boundary should be crisp:

- The server snapshot describes what is true: bird roster, current mood, perch zone, pose target, current action, weather modifier, settle state, call descriptor seeds, notebook cursors, and snapshot version.
- The client decides only how to draw that truth between snapshots: easing curves, feather/leaf ornaments, opacity ramps, gain ramps, and reduced-motion cross-fades.
- Decorative ambient leaf/feather drift is client-only and disposable. Bird positions, perch choices, greetings, offer reactions, and settle state are authoritative server outputs.

This preserves the "aviary continues without the viewer" illusion while preventing state divergence across devices.

## Data model

### Core entities

`accounts`

- `account_id` UUID primary key.
- `email_ciphertext` encrypted at rest.
- `email_verified_at`, `pending_email_ciphertext`.
- `timezone`, `created_at`, `soft_delete_at`, `hard_delete_at`.
- `visit_notifications_enabled`.

`session_tokens`

- `session_id`, `account_id`, `device_label`, `issued_at`, `last_seen_at`, `revoked_at`, `expires_at`.

`aviaries`

- `aviary_id`, `account_id`, `created_at`, `current_bird_cap`, `last_snapshot_version`, `last_tick_at`, `next_tick_due_at`.
- `scene_state` summary: local daypart, current weather event, settled flag, settle_started_at.

`birds`

- `bird_id`, `aviary_id`, `species_id`, `display_name`, `adoption_index`, `created_at`.
- Stable identity is invariant across renames, sync, and future migrations.

`bird_personality`

- One row per bird with hidden scalar traits: `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`.
- `last_drift_applied_at`.
- Optional per-trait accumulator fields so the tick can apply diminishing returns without recomputing from all history.

`bird_runtime_state`

- `bird_id`, `current_mood`, `perch_zone`, `pose_family`, `pose_phase`, `current_action`, `greeting_cooldown_until`.
- `offer_cooldown_until`, `last_greeted_at`, `last_listen_in_at`.
- `call_seed`, `call_phase_anchor`, `is_settled`.

`interaction_events`

- Append-only host event log with `event_id`, `account_id`, `device_id`, `bird_id` nullable, `event_type`, `occurred_at`, `ingested_at`, `payload_json`, `idempotency_key`.
- Event types: `presence_ping`, `visibility_resume`, `visibility_hidden`, `listen_in_start`, `listen_in_end`, `offer_seed`, `offer_song`, `offer_water`, `settle_start`, `settle_cancel`, `notebook_open`, `session_resume`.
- Retain raw events only for the minimum window needed for tick correctness, replay, and support; compact older history into durable state rather than keeping indefinite raw logs.

`presence_windows`

- Derived, server-owned representation of host attention intervals by device.
- Used to union overlapping device attention and prevent double-counting when the same host has multiple visible sessions.

`notebook_entries`

- `entry_id`, `aviary_id`, `created_at`, `entry_text`, `salience_type`, `source_tick_at`.
- Read-only after write.

`visit_invites`

- `invite_id`, `host_account_id`, `visitor_email_ciphertext`, `token_hash`, `status`, `created_at`, `expires_at`, `revoked_at`, `last_used_at`.

`visit_sessions`

- `visit_session_id`, `invite_id`, `started_at`, `last_seen_at`, `ended_at`, `approx_duration_seconds`.

`user_settings`

- Account-level defaults for captions, narration, audio muted, explicit reduced-motion opt-in, visit notifications, export preferences.
- Device-local overrides remain client-side where appropriate, especially `prefers-reduced-motion`.

### Snapshot shape

Every host snapshot should include:

- `snapshot_version`, `server_time`, `timezone`, `aviary_age_days`.
- `scene`: daypart, palette variant, weather modifier, settled state, top-level narration summary token.
- `birds[]`: `bird_id`, name, species, mood, perch zone, current action, pose target, motion timing window, visual saturation scalar, call descriptor seed, caption-ready call token, greeting eligibility, offer cooldown state.
- `transients[]`: active greeting, offer reaction, settle transition, current listen-in focus, currently active call cues.
- `notebook_summary`: latest entry header and unread marker semantics if needed.
- `limits`: current bird cap, next age-based bird offer eligibility if applicable.

Visitor snapshots use the same scene payload but exclude host settings, write affordances, and any host-only account metadata.

## API surface

### Auth and account

- `POST /auth/magic-link/request`
- `POST /auth/magic-link/consume`
- `GET /account/sessions`
- `DELETE /account/sessions/:sessionId`
- `POST /account/email-change/request`
- `POST /account/email-change/confirm`
- `POST /account/export`
- `POST /account/delete/start`
- `POST /account/delete/cancel`
- `GET /account/settings`
- `PUT /account/settings`

### Aviary read path

- `GET /aviary/snapshot`
  - Supports `If-None-Match` or `since_version`.
  - Returns a compact authoritative snapshot or `304`.
  - Used on initial load, visibility resume, long frame gap recovery, after write acknowledgements, and low-frequency visible polling.
- `GET /aviary/notebook?cursor=...`
  - Paged read-only notebook history.

### Aviary write path

- `POST /aviary/events`
  - Accepts a small batched event list with idempotency keys and the client's last seen snapshot version.
  - Appends events, validates cooldowns and preconditions, applies any immediate transient state changes, and returns:
    - accepted event IDs
    - updated `snapshot_version`
    - optional transient directives for the local renderer to use before the next poll completes
    - recommended next refresh delay

This endpoint is the only host write surface for aviary behavior. Clients never send absolute bird values.

### Visit flow

- `POST /visits/invites`
- `GET /visits/invites`
- `DELETE /visits/invites/:inviteId`
- `GET /visits/log`
- `GET /visit/:token/snapshot`
- `GET /visit/:token/notebook`

Visitor endpoints are read-only. They do not accept presence or interaction events, and visitor attention is never written into host simulation inputs.

### Polling model

- Host visible tab: poll snapshot every 15 seconds with conditional requests, plus immediate refresh after accepted writes and on visibility resume.
- Visitor visible tab: poll every 30 seconds to keep revocation latency low while staying lightweight.
- Hidden tabs stop polling except for session expiry handling.

This stays faithful to the PRD's pull-based model while keeping revocation and cross-device freshness acceptable.

## Simulation engine design

### Overall shape

Implement the engine as one authoritative system with two execution cadences:

- Immediate transient reactor: runs synchronously inside `aviary-api` or a tightly coupled state service when a host event is accepted. It updates greeting queues, active listen-in focus, offer reactions, settle transitions, and other short-lived scene changes so the response feels immediate.
- Durable minute tick: runs in `simulation-worker` once per minute per aviary. It applies personality drift, mood evolution, perch/action target updates, weather progress, notebook generation, and age-based bird offer eligibility.

Both layers write through the server-owned canonical state. The client never simulates ahead.

### Presence accounting

Presence is the dominant drift input and must remain honest:

- The client emits a `presence_ping` only when visible, focused, and recently active.
- The server does not trust the client blindly. It stores device-specific windows, clamps impossible durations, and unions overlapping windows across devices before turning them into account-level attention time.
- Settle and tab-hide close open presence windows.
- Visitor sessions never create presence windows.

### Drift function

Use trait-specific additive deltas with saturation guards:

- Presence-time is the largest input for all expressive traits, especially plumage saturation and boldness.
- Listen-in duration contributes strongly to `social_warmth` and `vocal_frequency` for the focused bird.
- Offers contribute lightly to `curiosity`; accepted or closely investigated offers can add a smaller nudge to `boldness`.
- Settle ends presence cleanly but has no direct negative or positive drift term.
- Neglect never subtracts traits. The absence path is zero delta, not reversal.

Implementation approach:

- Keep a rolling daily accumulator per bird for weighted presence and interaction signals.
- On each tick, apply a small capped positive delta per trait using a low-pass filter so one long session cannot produce visible same-day jumps.
- Add diminishing returns within a 24-hour window to prevent grinding behavior from overpowering the intended three-week visible-drift cadence.
- Build a synthetic calibration harness before launch and tune against the PRD target:
  - measurable in instruments after roughly one week of regular visits
  - visible to users after roughly three weeks

Do not use production user-history aggregation to tune the drift model. Use synthetic accounts and controlled internal dogfood scenarios.

### Mood model

Model mood as a small finite-state machine with hysteresis:

- Candidate states: `wary`, `content`, `curious`, `drowsy`, `alert`.
- Each minute tick computes transition scores from:
  - local daypart
  - active weather event
  - recent host interactions with exponential decay
  - neighboring bird impulses such as chorus or alarm-like calls
  - personality modifiers, especially boldness and curiosity
- Hysteresis prevents one-minute oscillation. A bird should not bounce between `curious` and `content` on adjacent ticks without a meaningful new impulse.

Mood persists across sessions because it lives in canonical state, not client memory.

### Perch choice and visible action selection

Each durable tick should update a bird's next target action and preferred perch zone:

- High boldness biases front perches; wary mood biases back perches.
- Content mood biases preening and relaxed scanning.
- Curious mood biases head-tilt, listening, and approaching recent offers.
- Drowsy mood lowers posture and call intensity near dusk/night.

The server emits target pose/action windows, and the client interpolates toward them.

### Greeting logic

On `session_resume` or visibility return after a meaningful absence:

- The server chooses one primary greeting bird using weighted selection across boldness, current mood, and recent greeting history.
- Longer absences widen the greeting repertoire toward closer perches, longer calls, or stronger visual notice.
- Secondary responders are optional and staggered by short random offsets.
- Greeting history and cooldowns prevent one bird from monopolizing every return unless its personality clearly warrants it.

### Offer reactions

Offers are a server-authored reaction surface:

- Validate per-bird cooldowns.
- Pick a receiving bird or small candidate set based on recent proximity, curiosity, and mood.
- Emit a transient reaction plan immediately: approach, ignore, watch, join the song fragment, drink, bathe, or remain still.
- Fold any lasting mood or drift implications into the next durable tick.

### Weather

Weather is generated server-side from a seeded lightweight scheduler:

- Soft wind and short rain only.
- Frequency target: a few small events per week.
- Weather alters mood/call propensities briefly but never dominates the scene.

### Call grammar runtime

Each species gets a compact motif library:

- Interval families
- cadence families
- trill shapes
- envelope/timbre profiles

Each bird derives a stable call identity from:

- species motif subset
- a per-bird seed controlling tempo bias, interval micro-variation, attack softness, and repetition limits
- current mood
- current vocal-frequency trait

The server snapshot does not send audio buffers. It sends descriptor tokens and scheduling windows; the client synthesizes the actual sound.

### Notebook generation

Notebook entries should be written by a deterministic observation generator:

- Inputs: salience events from ticks and accepted interactions, weather windows, greeting anomalies, unusual quiet, perch preference changes, notable pair behaviors.
- Guardrails: minimum spacing, dedupe by salience type, and explicit refusal to mention counters, metrics, or hidden numeric state.
- Voice: naturalist, lowercase, present tense, bird-specific, observational.

Target cadence:

- sparse by default, roughly every few days for a normally visited aviary
- more frequent only when a truly noteworthy event happens

## Sync model

### Single canonical writer model

- Only the server simulation system mutates canonical bird state.
- Host clients write append-only events.
- Visitor clients never write to the host's aviary.
- Snapshot versions are monotonic and every accepted write advances or references one.

### Preventing conflicts

- Require idempotency keys on every client event batch.
- Serialize per-aviary writes with row locks or a per-aviary work queue so two devices cannot apply transient state simultaneously without ordering.
- Collapse overlapping device presence intervals at the account level before drift inputs are computed.
- Return a refresh-required response if a client writes against an obsolete precondition, then refetch and retry from fresh state.

### Resume and suspension handling

- On visibility regain, app resume, or long render-frame gap, refetch snapshot immediately.
- The client stops local rendering and audio when hidden; the server tick keeps running.
- A resumed client discards stale transient local animation state in favor of the fresh authoritative snapshot.

### Revocation and expiry

- Visit revocation is enforced on every visitor snapshot read.
- Magic-link reuse, expired session tokens, and deletion-window state are handled as matter-of-fact system errors, not naturalist copy.

## Frontend rendering pipeline

### Rendering architecture

Use a layered frontend:

- SSR HTML shell for the quiet field, top-bar frame, and inline serialized initial snapshot.
- React/TypeScript for account/settings/notebook/accessibility surfaces and focus management.
- A compact custom scene renderer, preferably Canvas 2D with a tiny scene graph abstraction, for birds, perches, background layers, and ornaments.

Avoid a heavyweight game engine for v1. The scene is small, the bundle budget is tight, and seven birds with modest layered motion do not justify a large rendering runtime.

### First-frame strategy

- Server-render the quiet field immediately.
- Inline the first snapshot in the initial HTML so the renderer can draw the first bird without waiting for a second round trip.
- Defer non-critical settings/account bundles until after the first scene paint.

This is how the product hits the "already in motion" illusion without a spinner.

### Bird rendering

- Represent each bird as a small species-specific rig/pose library plus palette parameters, not as large sprite sheets.
- Snapshot state selects the current pose family, target perch, action timing, and saturation scalar.
- The renderer interpolates movement, head tilt, and preen/scan cadence locally inside server-provided bounds.

### Scene layering

Recommended draw order:

- sky/background foliage
- back perch zone
- rear birds
- middle perch zone
- front perch zone
- foreground leaves/branches
- captions/focus outlines
- top bar overlay

### Reduced-motion rendering

- Replace frame-by-frame micro-motion with slow cross-fades between still poses.
- Replace flight paths with cross-fade repositioning.
- Remove leaf/feather drift.
- Preserve palette shifts, call playback, captions, notebook, and mood/state changes.

### Top bar behavior

- Keep controls in a thin top bar only.
- Fade to near-transparent after inactivity.
- Restore on pointer movement or keyboard activity.
- Maintain keyboard visibility and clear focus styling even when visually faded.

## Audio pipeline

### Runtime architecture

- Use WebAudio with `AudioWorklet` for deterministic scheduling and low-GC synthesis.
- Maintain one synthesis voice per active bird and a shared chorus bus.
- Schedule calls from server-provided descriptor windows, not random local guesses.

### Bird call synthesis

Each call is generated from:

- oscillator/noise combinations
- envelopes
- filters
- motif timing descriptors
- per-bird timbre seeds

Mood and vocal frequency influence:

- cadence density
- attack softness
- pitch drift range
- chorus join likelihood

### Mix behavior

- Ambient mode keeps all birds audible at low, varying levels.
- Listen-in ramps the focused bird up and others down gradually; others never mute fully.
- Use only subtle depth cues. Do not introduce dramatic stereo gimmicks that make the aviary feel like a track mixer.

### Caption coupling

- Generate captions from the same descriptor object used by audio scheduling.
- Render them near the calling bird and expose the same text to assistive surfaces where useful.

### Fallback

- If WebAudio or AudioWorklet is unavailable, run the aviary silently.
- Automatically enable captions for that session.
- Do not ship recorded audio fallbacks.

## Accessibility surfaces

### Screen-reader narration

- Maintain an aria-live narration region fed by a narration scheduler.
- Idle cadence: one prose update every 30-60 seconds.
- Priority cadence: prompt narration for return greetings, accepted offers, settle, and major scene changes.
- Narration text comes from the same state as the visual scene and stays in naturalist voice.

Add a manual "describe aviary now" control in accessibility settings so screen-reader users can request an immediate summary without waiting for cadence.

### Keyboard navigation

- Tab moves through top-bar items.
- Entering the aviary with Tab focuses the first bird.
- Arrow keys move between birds in scene order.
- Enter toggles listen-in on the focused bird.
- Escape exits listen-in and closes transient overlays.
- Offer and settle remain top-bar reachable and fully keyboard navigable.

### Captions

- Optional per-account setting, quickly toggleable.
- Short, prose-like, near-bird overlays with AA contrast.
- Must not obscure birds or become a marquee; keep them transient and sparse.

### Reduced motion

- Honor `prefers-reduced-motion` by default.
- Allow manual override in settings.
- Ensure the reduced-motion scene still feels intentional, not disabled.

### Copy discipline

- Naturalist voice: aviary scene, notebook, captions, narration.
- Matter-of-fact voice: auth, errors, sync failures, settings, unsupported browser, revoked/expired invite surfaces.

### Accessibility QA

Treat this as a release gate, not a post-launch patch:

- VoiceOver on Safari
- NVDA on Windows with Chrome or Firefox
- keyboard-only flows on desktop
- reduced-motion regression suite
- caption overlap checks on narrow mobile screens

## Performance budgets and observability

### Budgets

- Initial JS bundle under 2 MB gzipped.
- First bird visible within 500 ms on a mid-tier mobile device over 4G.
- 60 fps idle rendering on a five-year-old mid-range laptop.
- No meaningful client memory growth over a 30-minute session.
- Simulation tick p99 under 5 seconds.

### Tactics to hit budgets

- Inline initial snapshot in HTML.
- Code-split settings, account, visit-management, and export/delete flows.
- Keep the scene renderer custom and small.
- Use vector rigs or compact reusable assets over large sprite sheets.
- Reuse audio nodes and buffers aggressively; no per-call unbounded allocations.
- Pause render loop when hidden and resume from fresh snapshot.

### Observability

Collect only aggregate operational metrics:

- first-bird render timing
- snapshot latency and error rates
- frame-time distributions
- AudioWorklet/audio-context failure counts
- tick duration p50/p95/p99
- invite email send success/failure
- session-duration histograms without per-account drilldown

Never send per-bird personality, per-account interaction sequences, or notebook text into the metrics pipeline.

### Synthetic monitoring and calibration

- Maintain synthetic browser probes from common geographies for load/render/audio checks.
- Maintain synthetic aviary accounts with scripted presence/offer/listen-in patterns for drift and notebook calibration.
- Use those synthetic accounts, not production user histories, to validate the "one week measurable / three weeks visible" target.

## Rollout

### Delivery phases

1. Foundations
   - Auth, account model, synthetic IDs, session management, canonical snapshot schema, event ingestion, minute tick scaffold, quiet field shell.
2. Core aviary alpha
   - Two birds only, no visits yet.
   - Greeting, watch/presence, listen-in, offer, settle, mood, call synthesis, reduced-motion baseline, initial narration.
3. Private beta
   - Notebook generation, export/delete, keyboard completeness, captions, visit invites, revocation, visit log, synthetic perf dashboards.
4. V1 launch
   - Full accessibility bar, operational alerting, privacy policy link, current bird-cap ramp flags, unsupported browser handling.

### Birds-per-aviary ramp

Start conservatively even though the model supports seven:

- Launch cohorts begin at two birds.
- Enable third-bird age-based offers only after audio recognizability, tick stability, and snapshot size remain healthy in synthetic and dogfood environments.
- Raise the configurable cap by cohorts from 3 to 5 to 7 as operational confidence grows.
- Keep the cap server-configurable so the team can freeze expansion without migrations if chorus clarity or performance regresses.

### Day-one instrumentation

From day one, instrument:

- auth success/failure rates
- first-bird timing
- snapshot poll success rate
- event ingestion error rate
- tick latency
- audio init failure rate
- reduced-motion/caption mode usage counts in aggregate
- invite issuance/revocation success

Do not instrument or aggregate "average user drift," "most offered bird," or any similar per-account behavior analysis.

## Risks and mitigations

### Drift calibrates too fast or too slow

- Mitigation: synthetic calibration harness, explicit trait caps, daily diminishing returns, release gating against the PRD timing targets.

### Presence inflation corrupts the whole model

- Mitigation: require visibility plus focus plus recent activity, union overlapping device windows, clamp impossible durations, test hidden-tab and multi-device scenarios aggressively.

### Immediate reactions and minute ticks diverge

- Mitigation: keep both paths server-authored off the same canonical state and versioning model; transients expire explicitly and durable tick reaffirms the new baseline.

### Audio feels canned or repetitive

- Mitigation: stable per-bird call identity plus enough motif variation, repetition guards, synthetic long-session listening tests, and a hard refusal to fall back to recorded loops.

### Accessibility surfaces flatten the product voice

- Mitigation: deterministic prose generators shared across notebook, narration, and captions; accessibility review as a product review, not only a compliance review.

### Bundle or first-bird budget is missed

- Mitigation: custom renderer, inline initial snapshot, strict bundle budgets in CI, aggressive code splitting, and no heavyweight animation/game frameworks by default.

### Sync correctness fails under multi-device host usage

- Mitigation: per-aviary write serialization, idempotent event ingestion, account-level presence union, refresh-required responses on stale preconditions.

### Privacy boundaries erode through observability shortcuts

- Mitigation: separate metrics pipeline from simulation tables, synthetic UUID everywhere outside the account record, privacy review on every new metric, no per-account analytics warehouse export.

### Visit revocation is not immediate enough

- Mitigation: 30-second visitor polling, server-side authorization on every visitor snapshot request, explicit revoked/expired surface, and revocation tests in CI.

## Execution guidance for the engineering team

Build this as a product with a small surface and unusually strict behavioral rules. The central success criterion is not feature count; it is whether the aviary feels like it has continuity, specificity, and restraint without violating privacy or accessibility. Every implementation shortcut that turns the system into a dashboard, a game, or a social feed should be treated as a regression, even if it would be conventional elsewhere.
