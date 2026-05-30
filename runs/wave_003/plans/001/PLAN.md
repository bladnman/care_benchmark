# Pocket Aviary v1 Implementation Plan

## 1. Executive summary

Pocket Aviary v1 is a web-only, single-account, single-aviary product whose core promise is that a small set of birds feels alive over days and weeks without becoming a game, chore, or social network. The implementation must preserve four non-negotiables:

- The aviary has one canonical server-authored state that continues on a slow tick whether clients are connected or not.
- Birds feel alive through procedural calls, mood-shaped idle behavior, and return greetings that notice the user without announcing them.
- Presence is the dominant drift input and must be measured honestly from visibility, focus, and recent user activity together.
- Privacy and tone boundaries are hard: per-bird interaction history stays inside the simulation system, and system/error surfaces use matter-of-fact voice while product surfaces remain naturalist.

This plan assumes a small frontier team delivering a browser product with a lightweight service backend, progressive rollout, and strong observability around performance, tick correctness, and accessibility quality. The product is not implemented in v1 as a real-time multiplayer or native application; the architecture deliberately narrows scope to protect affective quality.

## 2. Scope

### In scope for v1

- Browser-only client for modern Chrome, Safari, Firefox, and Edge.
- Magic-link authentication with one aviary per account.
- Two starter birds at account creation, expanding over aviary age up to a cap of seven.
- Server-side simulation tick at roughly once per minute for canonical personality drift, mood transitions, and continuity.
- Snapshot-based client rendering with first-bird visible within 500ms on target hardware/network.
- Interactions: return greeting, listen-in, offer, settle, field notebook reading.
- Multi-device sync via shared canonical server state.
- Quiet invite-only social feature: read-only visitor sessions via per-invite email links.
- Accessibility surfaces: screen-reader narration, reduced-motion mode, keyboard support, call captions, AA contrast.
- Operational telemetry that excludes per-account bird-state analytics.

### Explicitly out of scope

- Native iOS or Android apps.
- Gamification of any kind: scores, streaks, badges, levels, counters, visit calendars.
- Tamagotchi mechanics: hunger, decay, penalties for absence, distress states, bird death.
- Social-network surfaces: public discovery, comments, profiles, follows, co-presence, chat, leaderboards.
- Shared aviaries, multiple aviaries per account, scene customization, user-controlled perch placement.
- Recorded-audio fallback path.

### Product calls to make up front

- Treat settle as optional ceremony only, never as a required state transition.
- Keep the visual scene on a single screen with no panning/zooming, including on narrow mobile viewports.
- Keep notebook sparse by policy, even if event volume would support denser logging.
- Preserve voice split in the design system and content pipeline as a platform rule, not copy guidance.

## 3. System architecture

### High-level shape

Use a three-tier architecture:

1. Web client
2. Application API layer
3. Simulation and persistence layer

The web client renders the aviary, captures presence and interaction events, synthesizes procedural audio, and consumes canonical state snapshots. The application API authenticates users, serves snapshots, receives append-only interaction events, manages invitations, and exposes account/settings/notebook endpoints. The simulation layer owns all personality and mood mutation, running a scheduled tick that consumes recent events and writes canonical aviary state.

### Recommended service decomposition

- `web-client`
  Browser app, rendering, WebAudio synthesis, reduced-motion rendering, captions, screen-reader narration, top-bar/settings surfaces.
- `api-gateway`
  Authenticated HTTP API for account, aviary snapshot, event ingestion, notebook reads, settings, export, invite management, visitor access.
- `simulation-worker`
  Scheduled tick processor and background jobs for drift, mood transitions, greeting seeds, notebook generation candidates, bird-age unlock evaluation.
- `notification-worker`
  Magic-link emails, invite emails, export emails, optional visit notifications.
- `persistence`
  Relational primary store plus object/blob store for exports if needed.
- `observability stack`
  Aggregate metrics, tracing, logs scrubbed of PII and per-bird state.

### Deployment and runtime

- Serve static web assets from CDN.
- Run stateless API services behind load balancer.
- Run simulation workers from a durable queue or scheduler with partitioning by account UUID.
- Store canonical state in a primary relational database with row-level locking/versioning around simulation writes.
- Use a regional deployment model first; add read edge caching for initial HTML/bootstrap payload only.

### Client/server split

Server responsibilities:

- Canonical personality vectors and mood state
- Tick cadence and event-log consumption
- Bird unlock schedule based on aviary age
- Notebook entry generation and persistence
- Account/auth/session/invite/export/deletion flows
- Visitor authorization and revocation

Client responsibilities:

- Rendering current scene from snapshot
- Snapshot interpolation
- Audio synthesis from call grammar parameters
- Presence detection and interaction-event submission
- Listen-in mix changes
- Reduced-motion alternate rendering
- Screen-reader narration presentation and caption placement

### Render pipeline boundary

The server never streams raw animations. It sends semantic scene state plus timing anchors:

- bird ids, species, names
- current perch zone and target perch if transitioning
- current mood
- idle behavior seed/state
- call grammar parameters and near-future timing window
- ambient environment state: time-of-day phase, weather state, settled state
- snapshot timestamp and simulation version

The client converts semantic state into motion and audio locally. This keeps payloads small, supports sub-500ms first bird, and allows reduced-motion/audio settings to vary per device without changing canonical state.

## 4. Data model

Use a relational schema with append-only event tables plus canonical state tables.

### Core entities

#### Account

- `account_id` UUID primary key
- `email_encrypted`
- `email_hash_lookup` for sign-in lookup if needed
- `created_at`
- `deleted_at`
- `deletion_scheduled_for`
- `settings_json`
- `visit_notifications_enabled`

#### Session

- `session_id` UUID
- `account_id`
- `device_label`
- `created_at`
- `last_seen_at`
- `revoked_at`
- `expires_at`

#### Aviary

- `aviary_id` UUID
- `account_id` unique
- `created_at`
- `current_state_version`
- `settled_until` nullable
- `last_snapshot_generated_at`
- `local_timezone`

#### Bird

- `bird_id` UUID stable forever
- `aviary_id`
- `species_id`
- `display_name`
- `created_at`
- `adopted_phase` integer order within aviary
- `active` boolean

#### BirdPersonality

- `bird_id`
- `boldness`
- `social_warmth`
- `vocal_frequency`
- `plumage_saturation`
- `curiosity`
- `last_drift_applied_at`

Store as scalar normalized floats with guardrails and explicit min/max bounds.

#### BirdMoodState

- `bird_id`
- `mood`
- `mood_intensity`
- `mood_started_at`
- `last_transition_at`
- `carryover_context_json`

#### AviarySceneState

- `aviary_id`
- `state_version`
- `simulated_at`
- `time_of_day_phase`
- `weather_state`
- `settled_state`
- `scene_seed`

#### BirdSceneState

- `state_version`
- `bird_id`
- `perch_zone`
- `pose_state`
- `pose_progress`
- `target_perch_zone` nullable
- `call_state_json`
- `attention_state_json`

#### InteractionEvent

- `event_id` ULID or UUID
- `account_id`
- `aviary_id`
- `bird_id` nullable
- `event_type`
- `event_started_at`
- `event_ended_at` nullable
- `client_timestamp`
- `server_received_at`
- `session_id`
- `payload_json`
- `processed_at` nullable
- `tick_batch_id` nullable

Event types:

- `presence_ping`
- `listen_in_started`
- `listen_in_ended`
- `offer_seed`
- `offer_song_fragment`
- `offer_still_pool`
- `settle_started`
- `settle_undone`
- `visitor_session_started` for audit only, never drift

#### PresenceWindow

Optional derived table for tick efficiency:

- `presence_window_id`
- `account_id`
- `window_started_at`
- `window_ended_at`
- `qualified_seconds`
- `source_session_id`

This table is derived from presence pings and never directly client-authored.

#### NotebookEntry

- `entry_id` UUID
- `aviary_id`
- `observed_at`
- `created_at`
- `entry_text`
- `entry_type`
- `bird_ids_json`
- `source_state_version`

#### Invite

- `invite_id` UUID
- `host_account_id`
- `visitor_email_encrypted`
- `visitor_email_hash_lookup`
- `token_hash`
- `status` enum: pending, active, revoked, expired, consumed
- `created_at`
- `expires_at`
- `revoked_at`
- `visit_notifications_enabled_override` nullable

#### VisitSession

- `visit_session_id`
- `invite_id`
- `host_account_id`
- `started_at`
- `ended_at`
- `duration_seconds`
- `last_snapshot_at`

#### ExportRequest

- `export_request_id`
- `account_id`
- `requested_at`
- `completed_at`
- `download_token_hash`
- `expires_at`

### Data invariants

- One `aviary` per `account`.
- Bird ids are never replaced, even if names or species rendering rules evolve.
- Personality vectors mutate only in server-side simulation transactions.
- Mood carries across sessions and is never reset by client open.
- Visitor sessions never write presence or interaction events into host drift inputs.
- Email is stored only on account and invite records in encrypted form and never used as a join key elsewhere.

## 5. API surface

Use HTTP JSON APIs for v1. Real-time streaming is unnecessary because the tick is slow and clients can poll lightly.

### Auth and account

- `POST /auth/magic-link/request`
  Input: email
  Output: accepted
- `GET /auth/magic-link/consume?token=...`
  Output: session established, redirect/bootstrap payload
- `POST /auth/session/revoke`
- `GET /account`
- `PATCH /account/email`
- `POST /account/export`
- `POST /account/delete`
- `POST /account/delete/cancel`

### Aviary bootstrap and snapshots

- `GET /aviary/bootstrap`
  Returns current aviary summary, account settings relevant to rendering, initial state snapshot, notebook preview, top-bar flags.
- `GET /aviary/snapshot?since_version=<n>`
  Returns latest canonical state. Support `304` or compact no-op response if unchanged.
- `GET /aviary/notebook?cursor=<...>`
  Read-only paginated entries.

### Interaction event ingestion

- `POST /aviary/events`
  Accepts one or batched events. Server validates schema, session ownership, cooldowns, and event-type semantics.

Batch shape:

```json
{
  "events": [
    {
      "event_id": "ulid",
      "type": "presence_ping",
      "bird_id": null,
      "started_at": "2026-05-30T12:00:00Z",
      "ended_at": "2026-05-30T12:02:00Z",
      "payload": {
        "visibility": "visible",
        "focused": true,
        "recent_activity": true
      }
    }
  ]
}
```

### Offer flow

- `GET /aviary/offers/catalog`
  Static or lightly dynamic list of seed/song fragment/still pool options and cooldown availability.
- `POST /aviary/offers`
  Convenience endpoint that writes an offer event after validating cooldown.

### Social visit flow

- `POST /invites`
  Host creates invite by visitor email.
- `GET /invites`
  Host reads outstanding/active/revoked invites and visit log summary.
- `POST /invites/{invite_id}/revoke`
- `GET /visit/consume?token=...`
  Establishes read-only visitor session.
- `GET /visit/{visit_session_id}/snapshot`
  Returns host aviary snapshot if invite still active.

### Accessibility and settings

- `GET /settings/accessibility`
- `PATCH /settings/accessibility`

Settings fields:

- reduced_motion
- captions_enabled
- audio_enabled
- screen_reader_narration_enabled override if needed

### Error-surface contract

All account, auth, sync, invite, and unsupported-browser errors return concise matter-of-fact copy keys plus structured remediation hints. Product-surface prose is not generated from these endpoints.

## 6. Simulation engine design

### Tick model

Run the simulation tick about once per minute per active account partition. Each tick:

1. Load unprocessed interaction events since last processed watermark.
2. Derive qualified presence-time windows.
3. Compute drift deltas per bird.
4. Compute mood transitions using recent events, current local time, ambient weather, and personality.
5. Advance scene state: perch selection, pose states, call timing windows, settle progression, weather triggers.
6. Evaluate notebook-worthy observations and create entries sparingly.
7. Persist new canonical state and mark consumed events in one transaction.

### Partitioning and correctness

- Partition work by `account_id`.
- Ensure no two simulation workers process the same account concurrently.
- Use row locks or lease records on `aviary`/`account` during tick.
- Make tick idempotent via a batch watermark and event processing markers.

### Presence qualification

Presence is the highest-risk calibration area and needs a strict algorithm:

1. Client emits `presence_ping` every 60-120 seconds while the conjunction holds:
   visible, focused, recent input within trailing activity window.
2. Ping payload includes the qualifying interval start/end and the three booleans.
3. Server discards pings that do not satisfy all conditions.
4. Tick aggregates accepted windows, de-duplicates overlaps, caps implausible continuity, and credits qualified seconds.

Guardrails:

- No credit when tab is hidden or window unfocused.
- No credit if recent input has expired.
- No retroactive reconstruction from raw client page-open durations.
- Cap a single continuous window to defend against buggy clients.

### Drift function

Use additive server-authored deltas with asymmetry toward expressive traits:

- Presence-time is dominant.
- Listen-in adds modest warmth and vocal-frequency weight to the focused bird.
- Offers nudge curiosity and, secondarily, boldness when approached.
- Settle affects immediate mood quieting only and should not materially alter long-term drift.
- Neglect does not apply negative trait deltas.

Implementation model:

- Each trait has a small per-tick delta computed from recent qualified signals and damped by current level.
- Use a low-pass filter so deltas shrink near upper bounds and never jump visibly session-to-session.
- Maintain measurable change after about one week of regular visits and user-visible change after about three weeks.

Suggested formula family:

- `trait_next = clamp(trait_current + alpha * input_signal * (1 - trait_current_normalized)^beta)`

where `alpha` and `beta` are tuned per trait. This preserves monotonic increase while slowing saturation.

### Mood system

Mood is a finite-state machine with probabilistic weighting:

- Candidate states: wary, content, curious, drowsy, alert.
- Inputs:
  recent interactions
  local time-of-day bucket
  ambient weather
  neighboring bird call events
  current personality vector
  settled state

Design rules:

- Mood persists between sessions.
- Mood changes should be legible in motion and calls, not only in labels.
- High boldness reduces wary transitions.
- Evening pushes drowsy/settled probabilities up.
- Rain reduces vocal activity and nudges content/drowsy or wary depending on species/personality.

### Bird-to-bird behavior

Model lightweight social coupling inside the tick and client runtime:

- One bird call can seed a response probability for nearby birds.
- Wary states can spread softly across birds.
- High vocal-frequency birds are more likely to join emergent chorus windows.
- Greeting choice can be informed by social warmth and who greeted recently.

### Greeting selection

On client session resume/open, the server or client derives a greeting script seed from:

- absence length
- current mood
- boldness
- recent greeting history

Only one bird initiates the greeting within the first 1-2 seconds; others may respond with small staggered offsets. Store enough recent greeting history to avoid repetitive patterns.

### Weather and day/night

- Drive time-of-day from account local timezone.
- Trigger rare weather events from deterministic randomness seeded per aviary/week so they feel coherent but not synchronized globally.
- Weather affects mood and call frequency temporarily, but does not create permanent drift.

### Notebook generation

Notebook entries should be generated from observation candidates, not raw events. Build a rules engine that recognizes moments such as:

- one bird greeting first after a pattern reversal
- unusually quiet morning
- prolonged preening/content scene
- weather plus mood combination worth noting

Rate-limit notebook production so active accounts still average roughly one entry every few days unless something specifically noteworthy occurred. Store final prose, not only templates, to preserve historical continuity if generators change later.

## 7. Sync model

### Canonical-state strategy

The server is the single writer for personality, mood, and scene state. Clients only:

- read snapshots
- submit interaction events
- render/interpolate locally

This prevents client divergence and makes phone/laptop views inherently consistent.

### Snapshot lifecycle

Clients request:

- bootstrap snapshot on load
- refresh on visibility regain
- refresh after long frame gap or resume from suspension
- low-frequency keepalive refresh while visible, such as every 30-60 seconds

Snapshots include `state_version` and `simulated_at`. If unchanged, server returns compact response to reduce bandwidth.

### Conflict prevention

- No last-write-wins on personality or mood state.
- Server processes append-only events in order.
- Client event ids are idempotent so retries do not duplicate effects.
- Offer cooldown validation happens server-side.
- Visitor sessions use separate auth context and cannot hit host event-ingestion endpoints.

### Offline and degraded connectivity

V1 should prefer honesty over faux continuity:

- If the client loses connectivity briefly, continue local rendering from last snapshot for ambient feel.
- Queue interaction events locally with short retry window.
- If queue age exceeds a threshold or session is stale, show matter-of-fact sync status in top-bar/settings surface, not in aviary scene.
- Do not allow extended offline simulation that later merges personality changes. The server remains the only authority.

## 8. Frontend rendering pipeline

### Rendering stack

Use a modern browser rendering architecture:

- Canvas or WebGL/WebGPU-backed scene for birds and ambient motion
- DOM for top bar, settings, notebook, account flows, accessibility overlays
- Separate timing loop for scene rendering and audio scheduling

Choose the scene technology based on team expertise, but require:

- smooth interpolation
- efficient layering
- subtle parallax
- pose blending or sprite/SVG state transitions
- deterministic reduced-motion alternative

### Initial load path

1. Download minimal HTML/CSS shell and critical JS.
2. Fetch bootstrap snapshot with initial scene state.
3. Render quiet field immediately if snapshot is still in flight.
4. As soon as snapshot lands, draw first bird already mid-action.
5. Hydrate remaining controls lazily.

No spinner. No explicit loading animation. The quiet field is the only fallback visual.

### Scene composition

Layers:

- background sky/foliage
- middle-plane perches and birds
- subtle foreground leaves/branches
- DOM top bar overlay
- optional caption overlays near birds

Birds select among three perch zones: front, middle, back. Camera never pans.

### Motion system

Render bird life through:

- idle pose libraries keyed by mood/species
- interpolation between perch targets
- ambient micro-motion ornaments like leaves/feathers
- greeting animations composed from the same primitives, not bespoke cinematic sequences

The goal is continual low-amplitude motion, never UI-like flourish.

### Reduced-motion rendering

Implement a parallel render mode:

- replace continuous pose motion with slow cross-fades between still poses
- replace flight paths with cross-fade relocations
- remove leaf/feather drift
- keep slow color/lighting transitions

This should be a first-class rendering path with its own QA coverage, not a flag that simply disables animation calls.

### Top bar and chrome

- sparse icon-only or icon-plus-label bar above scene
- fades nearly transparent after inactivity
- returns on pointer or keyboard activity
- contains settings/account, accessibility, notebook, offers, settle affordance

Keep all non-scene interactions out of the aviary plane itself.

## 9. Audio pipeline

### Core approach

Synthesize all bird calls client-side with WebAudio from motif libraries and runtime parameters delivered in snapshots or generated from species seeds.

Components:

- species motif library
- per-bird call signature parameters
- mood modifiers
- vocal-frequency timing scheduler
- ambient chorus mixer
- listen-in mix controller

### Call recognizability

Each bird needs a stable audible identity across mood drift. Achieve this by fixing:

- motif family
- pitch neighborhood
- rhythmic tendency
- timbral contour

Then vary:

- timing
- call length
- spacing
- ornamentation intensity
- chorus response likelihood

### Listen-in mix

- Focused bird gain rises gradually over 300-800ms.
- Other birds attenuate to ambient, never silence.
- Clicking away or refocusing ramps back smoothly.

### Chorus mixing

- Avoid phasey stacking by scheduling distinct procedural calls rather than looping samples.
- Put soft headroom limits on simultaneous calls to protect clarity with up to seven birds.
- Maintain subtle spatial separation only if it does not imply scrolling geography; otherwise keep mix mostly centered with light depth cues.

### Captions from runtime grammar

Generate caption prose from the same call grammar events actually played. This avoids mismatch between audio and caption and keeps the caption surface alive instead of canned.

### Fallback behavior

If WebAudio is unavailable or blocked:

- run silent mode
- default captions on
- preserve notebook, narration, and visual behavior
- avoid shipping recorded audio fallback

## 10. Accessibility surfaces

### Screen-reader narration

Build a narration engine from scene state, using naturalist prose and low-frequency updates:

- idle cadence around 30-60 seconds
- priority narration for greeting, successful offers, settle, and major mood-visible changes
- queue management to avoid flooding assistive tech

Implementation approach:

- Generate concise prose from snapshot deltas and current scene composition.
- Announce only meaningful state, not every micro-transition.
- Use ARIA live regions with priority separation for urgent vs ambient narration.

### Keyboard navigation

- Tab into top-bar controls
- Enter aviary focus mode
- Arrow keys move between birds
- Enter toggles listen-in
- Escape exits listen-in
- Offer and settle reachable without pointer

Bird focus should visibly outline the bird against bright and dark scenes.

### Captions

- User-toggleable
- Positioned near active caller
- Fade with call duration
- Maintain AA contrast
- Avoid overlapping important scene elements by using per-bird anchor zones

### Reduced-motion and settings

- Honor `prefers-reduced-motion` on first load
- Allow explicit override in accessibility settings
- Persist per account or per device depending privacy/product choice; v1 recommendation is per-account with local override fallback

### Visual contrast and text

- All top-bar, settings, account, caption, and error text meets AA contrast.
- Unsupported-browser, auth, and sync errors use matter-of-fact tone with accessible semantics.

### Accessibility QA

Ship dedicated QA coverage for:

- VoiceOver, NVDA, and TalkBack/VoiceOver mobile equivalents as feasible
- keyboard-only session completion
- reduced-motion session feel
- captions with audio disabled

## 11. Performance budgets and observability

### Hard budgets

- Initial JS bundle under 2MB gzipped
- First bird visible under 500ms on mid-tier mobile over 4G
- 60fps idle rendering on five-year-old mid-range laptop
- No memory growth over 30-minute session
- Simulation tick p99 under 5 seconds

### Engineering implications

- Code-split settings, notebook history pagination, invite/account flows
- Keep snapshot payloads compact and semantic
- Reuse audio buffers and object pools
- Bound in-memory history retained by scene runtime
- Avoid unbounded DOM node accumulation from captions/narration

### Observability

Collect aggregate-only telemetry:

- bootstrap latency
- first-bird-render time
- snapshot fetch latency and payload size
- render-frame timing
- audio-context creation failures
- tick duration and backlog
- API error rate

Explicitly exclude:

- raw personality vectors in telemetry
- per-account interaction histories in analytics
- bird names or notebook text in aggregate metrics

### Test harnesses

- Synthetic browser runs from common geographies against production-like env
- Load tests on event ingestion and tick scheduling
- Soak tests for 30-minute memory stability
- Deterministic simulation tests for drift and mood transitions

## 12. Rollout plan

### Phase 0: foundation

- Finalize domain model and voice/content rules.
- Implement auth, account UUID model, baseline aviary schema, event ingestion, and simulation skeleton.
- Stand up a static prototype of scene rendering and WebAudio spike to validate first-bird and bundle budgets early.

### Phase 1: single-account aviary core

- Deliver account creation, two-bird adoption/naming, canonical state snapshots, server tick, basic scene rendering, time-of-day lighting, idle motion, and listen-in.
- Validate first-bird and 60fps budgets before adding optional surfaces.

### Phase 2: interaction and continuity

- Add offers, settle, mood persistence, notebook generation, multi-device sync validation, account settings, and export/delete flows.
- Lock down privacy boundary and telemetry schema.

### Phase 3: accessibility and polish

- Ship reduced-motion mode, narration, captions, keyboard navigation, contrast tuning, unsupported-browser flow.
- Tune greeting variety, notebook sparsity, mood readability, and audio recognizability.

### Phase 4: quiet social

- Add invite, visitor session, revocation, visit log, optional visit notifications off by default.
- Prove visitor sessions cannot mutate host drift or presence.

### Controlled release

- Start with employee/internal dogfood.
- Then limited beta with fixed cap on accounts.
- Ramp birds-per-aviary unlock schedule conservatively; third-bird unlock can remain feature-flagged until audio clarity is validated in real usage.

## 13. Instrumentation from day one

Track from initial launch:

- sign-in success and magic-link expiry rates
- first-bird render timing
- snapshot freshness on resume
- tick latency and backlog
- offer cooldown rejection rate
- narration/caption enablement rates in aggregate only
- WebAudio unavailable rate
- invite creation, revocation, and expired-link rate

Do not track from day one:

- cross-account behavioral comparisons
- bird-trait distributions by named cohort
- anything reconstructing individual relationship histories outside account export and direct operational debugging

## 14. Major risks and mitigations

### Drift calibration risk

Risk:
Birds drift too fast and feel game-like, or too slowly and feel static.

Mitigation:

- Build deterministic simulation tuning harnesses with synthetic presence patterns.
- Use internal evaluation scripts for one-week and three-week calibration targets before beta.
- Keep drift coefficients remote-configurable within bounded safe ranges.

### Presence honesty risk

Risk:
Buggy presence measurement over-credits background tabs and corrupts drift globally.

Mitigation:

- Require all three signals for qualification.
- Server-side sanity checks and caps.
- Separate metrics on percentage of credited time vs open-tab time in anonymized aggregate form.
- Add client integration tests for visibility/focus/activity transitions.

### Sync correctness risk

Risk:
Multiple devices overwrite or duplicate drift effects.

Mitigation:

- Append-only idempotent event ingestion.
- Server-only personality writes.
- Per-account simulation locking.
- Resume/visibility-change snapshot refresh path exercised in integration tests.

### Audio uncanniness risk

Risk:
Calls sound canned, repetitive, or muddy beyond a few birds.

Mitigation:

- Prototype procedural motif library first.
- Conduct recognizability testing at 2, 5, and 7 birds.
- Gate additional bird unlocks behind clarity metrics and qualitative review.

### Accessibility regression risk

Risk:
Accessible modes technically function but lose the product's charm.

Mitigation:

- Treat narration and reduced-motion as dedicated product surfaces with PM/design review.
- Include accessibility-specific experiential QA, not only checklist QA.
- Block launch if reduced-motion or narration feel like stripped fallbacks.

### Performance budget risk

Risk:
Bundle or rendering complexity pushes first-bird over target and breaks the "already alive" illusion.

Mitigation:

- Set perf budgets in CI from the first render prototype.
- Defer non-critical routes aggressively.
- Keep bird art and call generation lightweight and reusable.

### Privacy boundary risk

Risk:
Per-bird state leaks into logs, analytics, or service identifiers.

Mitigation:

- Synthetic account UUID everywhere outside encrypted account record.
- Log scrubbing at SDK and collector level.
- Separate simulation datastore access from analytics systems.
- Privacy review on every new telemetry field.

## 15. Delivery organization

Recommend four workstreams running in parallel after schema/API alignment:

- Simulation and data platform
- Web rendering and audio
- Account/auth/settings/invite surfaces
- Accessibility, QA, and observability

Shared weekly integration checkpoints should validate:

- snapshot contract stability
- tone boundary correctness
- perf budget trend
- drift calibration trend

## 16. Open implementation decisions to resolve early

- Exact tick cadence within the "~once per minute" target
- Whether snapshots are generated on read or precomputed after each tick
- Final scene technology: Canvas 2D vs WebGL-based renderer
- Exact activity window length for presence qualification
- Per-account vs per-device persistence behavior for accessibility settings
- Whether notebook prose is rule-template based or LLM-assisted offline generation; v1 recommendation is deterministic template/rules only for consistency, latency, and privacy

## 17. Acceptance criteria

V1 is ready when all of the following are true:

- A user can sign in via magic link on phone and laptop and see the same aviary mood/state.
- Two starter birds feel behaviorally distinct in greeting, calls, and perch choices.
- Presence-driven drift is measurable after one week of regular visits and still subtle within any single day.
- First bird appears within 500ms on target conditions without spinner UI.
- Screen-reader narration, reduced-motion mode, captions, and keyboard controls all preserve the same product feel rather than acting as degraded fallbacks.
- Visitor sessions are clearly read-only and do not affect host bird drift.
- No telemetry path contains per-bird interaction history outside the simulation domain and user-requested export.

## 18. Recommended build order

1. Schema, auth skeleton, snapshot/event contracts
2. Simulation tick with deterministic tests
3. Core scene renderer plus quiet-field bootstrap
4. Procedural call prototype and listen-in
5. Presence capture and drift tuning
6. Offers, settle, and notebook
7. Accessibility surfaces
8. Invites and visitor mode
9. Export/delete and final privacy hardening

This order protects the central illusion first: continuity, presence, and live-feeling birds before optional surfaces.
