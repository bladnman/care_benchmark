# Pocket Aviary v1 Implementation Plan

## 1. Scope and product contract

Pocket Aviary v1 is a web-only, single-account, single-aviary product centered on two starter birds that persist and drift over time inside one horizontally framed browser scene. The implementation must preserve the core affective contract from the PRD: the aviary feels continuously alive, notices the user without announcement UI, rewards attention without punishing absence, and never collapses into game loops, pet-care obligations, or social-network dynamics.

### In scope for v1

- Browser-only product for modern Chrome, Safari, Firefox, and Edge.
- Email magic-link authentication with one aviary per account.
- Two starter birds at account creation, with age-based expansion up to a hard cap of seven birds.
- Canonical server-side simulation tick that advances mood, drift, time-of-day behavior, weather windows, greetings, and notebook-worthy events whether or not a client is open.
- Read-only field notebook generated in naturalist prose.
- User interactions: passive presence, listen-in, offer, settle, notebook browsing, account/settings, accessibility settings.
- Multi-device sync via canonical server state plus append-only interaction events.
- Quiet optional social feature: invite-by-email read-only visits, off by default, revocable, expiring.
- Accessibility surfaces as first-class product features: screen-reader narration, reduced-motion mode, call captions, keyboard support, contrast compliance.
- Aggregate operational telemetry and performance instrumentation only, with strict exclusion of per-bird interaction history from analytics.

### Explicitly out of scope

- Native mobile apps.
- Gamification of any form: streaks, scores, badges, XP, counters, leaderboards, visit frequency surfaces.
- Tamagotchi-style hunger, decay, death, or punitive neglect mechanics.
- Shared aviaries, co-presence, visitor interactions, chat, comments, profiles, discovery, public feeds.
- Scene customization, multi-aviary accounts, paid tiers, public ranking, recorded-audio fallback.

### Planning assumptions to lock early

- v1 will be delivered by one product team, so architecture should minimize service count while preserving hard boundaries around simulation authorship and privacy.
- The product should degrade to silence plus captions if audio synthesis is unavailable, rather than introducing a lower-fidelity recorded-audio path.
- The system should treat all bird personality state as canonical server data with durable migrations and recovery safeguards, because personality loss is the worst failure mode.

## 2. System architecture

### High-level shape

Build the product as a thin-client web app backed by a single authoritative application platform with three major runtime responsibilities:

1. `web-client`
   Browser app responsible for rendering the aviary, synthesizing audio with WebAudio, capturing qualified interaction events, interpolating between server snapshots, and presenting notebook/settings/accessibility surfaces.

2. `api-app`
   Stateless HTTP API tier responsible for auth flows, snapshot reads, interaction-event ingestion, visit-link resolution, account/settings mutations, export/delete workflows, and server-generated prose endpoints when needed.

3. `simulation-worker`
   Server-side scheduled worker responsible for per-account simulation ticks, drift updates, mood transitions, weather windows, notebook-entry generation, stale-session cleanup, and derived snapshot materialization.

Support infrastructure:

- Primary relational database for accounts, birds, snapshots, notebook entries, invites, visit logs, and durable event log.
- Job queue / scheduler for simulation ticks, email sends, exports, and deletion windows.
- Object storage for exports and any compact visual/audio motif assets that should not live in bundle code.
- CDN/edge delivery for HTML, JS, CSS, SVG/bitmap assets, and snapshot bootstrap payloads.
- Email provider for magic links, invites, exports, and deletion confirmation.

### Architectural principles

- Server is the only writer of bird personality vectors and canonical mood state.
- Clients append events; they never submit authoritative bird state.
- Snapshot reads must be cheap enough to support visibility refreshes and keepalives.
- Privacy boundary is enforced architecturally: simulation data and aggregate telemetry use separate pipelines and schemas.
- Initial load path is optimized for "first bird visible" rather than full app hydration.

### Recommended implementation stack

- Frontend: TypeScript + React, rendered as a compact SPA with SSR or edge-rendered shell for fast first paint.
- Backend: TypeScript service runtime with shared domain models between API and worker packages.
- Database: PostgreSQL for durable relational state and ordered event ingestion.
- Queue/scheduler: managed durable queue with per-account scheduling and retry semantics.
- Email: transactional provider with template support.

This keeps the platform small enough for one team while still separating interactive API concerns from simulation execution.

## 3. Domain and data model

### Core entities

#### Account

- `account_id` UUID synthetic identifier.
- Encrypted verified email.
- auth status, deletion status, created_at, deleted_at.
- locale and preferred timezone.
- settings: audio enabled, captions enabled, reduced-motion override, visit notification opt-in.
- current session registry for revocable device sessions.

#### Aviary

- `aviary_id` UUID, one-to-one with account.
- created_at.
- current bird count and next age-based adoption eligibility timestamp.
- settled state marker for active session presentation.
- current ambient weather window state.
- current local day-part derivation inputs.

#### Bird

- `bird_id` stable UUID.
- `aviary_id`.
- species key.
- current display name.
- adoption index and adopted_at.
- active perch zone and render pose metadata.
- personality vector fields: boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity.
- current mood enum plus mood_started_at and transient mood modifiers.
- call signature seed / motif library references.
- cooldown timestamps for offers.
- drift_history summary fields for operational inspection only, never user-visible.

#### Interaction event

- `event_id`, `account_id`, `aviary_id`, optionally `bird_id`.
- event type: presence_ping, listen_in_started, listen_in_ended, offer_seed, offer_song, offer_pool, settle_started, settle_canceled, session_visible, session_hidden.
- occurred_at from server receive time plus client-side local timestamp for diagnostics.
- device/session id.
- event payload constrained to non-PII operational data.

#### Presence session accumulator

- per-device active presence window derived from visibility + focus + recent input.
- last qualified activity timestamp.
- accumulated presence seconds since last tick.
- closed/reopened markers.

#### Notebook entry

- `entry_id`, `aviary_id`.
- created_at.
- prose body.
- source tags for internal generation logic such as greeting-order change, prolonged quiet, weather moment.
- significance score used only to preserve sparsity and avoid spam.

#### Invite and visit

- invite id, host account id, visitor email, token hash, created_at, expires_at, revoked_at.
- visit session id, invite id, started_at, ended_at, approx_duration_seconds.

#### Materialized aviary snapshot

- `aviary_id`, `version`, generated_at.
- per-bird render state: perch, pose, motion phase, mood, call timing hints.
- ambient scene state: day-part, weather, settled state.
- audio hints: current chorus windows, active listen-in target if any for that client response.
- narration summary seed and caption seed inputs.

### Data retention rules

- Canonical simulation state persists until deletion.
- Event log retention should be long enough to support replay and audits, but bounded; after compaction, only canonical state and minimal recovery artifacts remain.
- Analytics store receives only aggregate metrics with no account or bird dimensions.

## 4. API surface

Design the API around canonical snapshot reads and event appends.

### Auth and account

- `POST /auth/request-magic-link`
- `POST /auth/consume-magic-link`
- `POST /auth/logout`
- `GET /account`
- `POST /account/email-change/request`
- `POST /account/email-change/confirm`
- `GET /account/sessions`
- `POST /account/sessions/{sessionId}/revoke`
- `POST /account/export`
- `POST /account/delete`
- `POST /account/delete/cancel`

### Aviary state

- `GET /aviary/snapshot`
  Returns the current materialized snapshot plus minimal bootstrap data for immediate render.
- `GET /aviary/notebook`
  Paginated read-only notebook entries, newest first.
- `GET /aviary/birds`
  Bird names/species/settings surfaces only; no raw personality numbers returned to product clients.

### Interaction ingest

- `POST /aviary/events`
  Batched append-only ingest for interaction events. Server validates event type, session, timestamps, and bird ownership.
- `POST /aviary/presence`
  Optional specialized endpoint if presence pings need tighter validation/rate limiting than general events.

### Visit flow

- `POST /visits/invites`
- `GET /visits/invites`
- `POST /visits/invites/{inviteId}/revoke`
- `GET /visit/{token}`
  Resolves one-time or active visit session and returns read-only host snapshot authorization.
- `POST /visit/{token}/heartbeat`
  Records visit duration only; does not affect host drift.

### Accessibility and settings

- `GET /settings/accessibility`
- `POST /settings/accessibility`

### API behavior rules

- Snapshot responses are versioned and cache-aware, but authenticated responses remain short-lived.
- Event append endpoints acknowledge quickly and leave drift interpretation to the tick.
- Visit endpoints never expose host account PII beyond explicitly invited email context needed for logs.
- No endpoint returns raw personality vectors to the product UI, except export generation and potentially internal admin tooling outside v1 scope.

## 5. Simulation engine design

### Tick cadence and ownership

- Run per-account simulation at roughly 60-second cadence, jittered slightly to avoid thundering herds.
- Worker acquires a per-aviary lock, reads unprocessed interaction events plus recent canonical state, computes next state, persists atomically, and advances the materialized snapshot.
- Ticks continue regardless of client connectivity.

### Drift model

- Use additive deltas over bounded scalar traits.
- Presence-time is dominant input; listen-in is second-order and bird-specific; offers apply small capped adjustments; settle ends presence but does not impose negative trait change.
- Drift is monotonic toward expressive: traits only increase from positive interaction, never decrease due to neglect.
- "Ambient quietness" on absence is implemented via lower greeting likelihood, reduced call participation, and mood defaults, not negative vector drift.

### Calibration targets

- Instrument-detectable drift after about one week of regular presence.
- User-perceivable drift after about three weeks.
- No single session should move any visible behavior so much that the user can infer a meter.

### Mood engine

- Mood is an enum with weighted transitions among wary, content, curious, drowsy, alert, and optionally settled/sleeping night variants.
- Inputs: recent accepted offers, recent listen-in attention, time of day in user timezone, ambient weather, nearby bird alarm/call interactions, and personality modifiers.
- Mood persists between sessions and across devices because the server snapshot is canonical.

### Greeting selection

- On visible return after absence, snapshot marks one primary greeter based on boldness, social warmth, current mood, and absence duration.
- Secondary greeting opportunities are staggered probabilistically, never synchronized.
- Greeting plan is precomputed in snapshot hints so the client can render within first seconds without extra round trips.

### Call grammar runtime

- Each species owns a motif library and synthesis parameter ranges.
- Each bird adds stable seed variation so the same species still sounds individually recognizable.
- Runtime chooses motif sequence, duration, pitch contour, pauses, and chorus timing based on mood and vocal-frequency trait.
- Caption generation derives from the same call plan structure so caption text matches actual played calls.

### Notebook generation

- Notebook entries are generated by the simulation worker from noteworthy moments, not from every session.
- Use template-plus-variation prose generation constrained to naturalist lowercase present-tense style.
- Entry generation must enforce sparsity with significance thresholds and per-aviary rate limits.
- Internal source tags support testing and tuning, but only prose is user-visible.

### Weather and day-part

- Day-part derives from account timezone and local clock.
- Weather windows are rare, short-lived scheduled ambient overlays with low-intensity mood effects.
- Night behavior keeps at least one species occasionally active so nighttime is quiet but not dead.

## 6. Sync and multi-device model

### Canonical-state strategy

- Clients always read from one server-authored snapshot lineage.
- Event log ordering is authoritative; personality vectors are updated only by the tick.
- No last-write-wins state merges are allowed for personality or mood.

### Client sync behavior

- Pull snapshot on initial load.
- Pull on visibility regain, long frame-gap recovery, periodic foreground keepalive, and after significant user actions that should reflect quickly.
- Use snapshot version numbers to skip redundant state application.
- Interpolate render positions and motion phases locally between snapshots for smoothness.

### Conflict prevention

- Session tokens scoped per device.
- Event ingest is idempotent with client-generated request ids.
- Worker stores last processed event cursor per aviary.
- Per-aviary tick lock prevents concurrent drift writes.
- Invite revocation and session expiry propagate through snapshot authorization checks on next pull.

### Failure handling

- If snapshot pull fails, show matter-of-fact system error surface, not naturalist prose.
- If event append fails transiently, queue locally and retry with bounded retention; do not synthesize client-side drift.
- If a session expires, client stops presence capture and prompts re-auth.

## 7. Frontend rendering pipeline

### Render architecture

- One scene renderer responsible for birds, perches, ambient layers, top bar chrome, and accessibility overlays.
- Snapshot bootstrap includes enough pose/motion state to draw birds immediately without waiting for non-critical surfaces.
- Animation system separates canonical state from render ornaments:
  - canonical: bird perch, pose family, active greeting, mood, weather.
  - ornamental: leaf drift, feather drift, subtle parallax.

### First-frame behavior

- Initial HTML/CSS renders quiet field quickly.
- First snapshot is inlined or fetched from edge fast enough to place at least one bird within 500ms.
- No spinner, no wake-up animation, no fade-from-static sequence.
- Birds enter already mid-action using snapshot motion phase.

### Bird motion system

- Motion clips or procedural pose states keyed by mood and action family: preen, scan, tilt, shuffle, call posture, settle posture.
- Render interpolation bridges snapshots without teleporting.
- Front/middle/back perch zones affect scale, opacity treatment, and subtle depth cues rather than requiring 3D scene complexity.

### Top bar and controls

- Sparse top bar above scene with account/settings, accessibility, notebook, offer, and settle affordances.
- Opacity fades near-transparent after inactivity and returns on cursor/keyboard activity.
- No inline UI chrome in scene.

### Responsive behavior

- Layout system preserves all birds in frame across mobile and desktop widths.
- Wide screens increase perch spacing; narrow screens compress spacing without cropping birds.
- Focus management remains stable when viewport changes.

### Reduced-motion pipeline

- Dedicated renderer variant switches animated paths into pose cross-fades, removes drifting leaf ornaments, and slows palette transitions.
- Simulation inputs remain identical; only visual presentation changes.

## 8. Audio pipeline

### WebAudio synthesis

- Client creates one bounded audio engine with reusable nodes and motif generators.
- Each bird has a stable synthesis profile seeded from species + bird id.
- Chorus mixing supports overlapping calls with per-bird pan/volume/spatial hints subtle enough to preserve calm.

### Listen-in mix

- Focusing a bird ramps its mix up and ramps others down to ambient, never to zero.
- Mix transitions are eased over time to feel like attentive listening, not channel switching.
- Keyboard and pointer interactions share the same audio-state machine.

### Audio fallback behavior

- If WebAudio init fails or permission is unavailable, app defaults to silence plus captions enabled.
- Silence fallback still renders call timing visually and textually.
- No pre-recorded audio fallback path is implemented.

### Audio performance safeguards

- Reuse oscillators/buffers where possible.
- Bound concurrent call count to maintain recognizability and protect CPU.
- Monitor audio-context errors and voice-node growth.

## 9. Accessibility surfaces

### Screen-reader narration

- Generate slow-cadence naturalist narration from snapshot state and prioritized interaction events.
- Idle narration approximately every 30-60 seconds; prompt narration for greeting, successful offer reactions, and settle events.
- Use a dedicated live-region strategy that avoids flooding screen-reader queues.

### Captions

- Per-call caption text derived from actual procedural call plan.
- Positioned near calling bird with AA-compliant contrast and calm transitions.
- User-toggle in accessibility settings, with automatic enablement when audio fallback engages.

### Keyboard and focus

- Predictable top-bar tab order.
- Bird focus traversal via arrow keys once focus enters scene.
- Enter toggles listen-in; Escape exits listen-in; offer and settle are keyboard reachable.
- Focus rings designed for bright and dim aviary states.

### Reduced motion and contrast

- Respect `prefers-reduced-motion` on first load and allow explicit override in settings.
- All product/system text meets WCAG AA.
- Reduced-motion mode is tested as a full experience, not a passive CSS flag.

### Accessibility QA

- Include manual screen-reader testing on VoiceOver and NVDA-equivalent path if feasible.
- Add snapshot-driven narration tests to ensure prose remains naturalist and non-metric.

## 10. Privacy, security, and compliance shape

- Use synthetic UUIDs everywhere outside the encrypted account email field.
- Keep simulation database and operational telemetry stores logically separated.
- Exclude per-account and per-bird event streams from analytics warehouse ingestion.
- Magic links expire after 15 minutes and are single-use.
- Session revocation is immediate for future requests.
- Account deletion is soft for 30 days, then hard-delete via scheduled job with export/storage cleanup.
- Export generation produces JSON snapshot download via emailed verified link.

## 11. Performance budgets and observability

### Budgets

- Initial JS bundle under 2MB gzipped.
- First bird visible under 500ms on mid-tier mobile over 4G.
- Idle motion at 60fps on five-year-old mid-range laptop.
- No client memory growth over 30-minute sessions.
- Simulation tick latency p99 under 5 seconds.

### Observability

- Aggregate-only RUM: navigation timing, first-bird timing, frame pacing, audio errors, snapshot latency.
- Worker metrics: tick queue depth, tick duration, lock contention, notebook generation rate, invite email failures.
- Synthetic browser probes from key geographies against signed-out shell and synthetic signed-in test accounts.

### What not to measure

- No dashboards keyed by account, bird, or interaction history for product analysis.
- No population-level ranking or comparative drift features.
- No telemetry that reconstructs a specific user's relationship with their birds.

## 12. Delivery plan and workstreams

### Phase A: Foundations

- Stand up auth, account model, session management, aviary/bird schema, and event log.
- Implement starter-bird creation and stable species/name model.
- Build snapshot materialization and per-aviary tick scheduling skeleton.

### Phase B: Core simulation

- Implement presence qualification, drift deltas, mood transitions, day-part, weather, greeting selection, and notebook generation.
- Add deterministic simulation tests and calibration harnesses for one-week and three-week drift targets.

### Phase C: Core client

- Build first-frame load shell, scene renderer, top bar, bird focus model, snapshot interpolation, and matter-of-fact system surfaces.
- Add listen-in, offer, settle, notebook browsing, and accessibility settings.

### Phase D: Audio and accessibility

- Integrate procedural WebAudio engine, captions, screen-reader narration, reduced-motion renderer, and keyboard polish.
- Run dedicated compatibility/performance passes across supported browsers.

### Phase E: Social and lifecycle

- Implement invite creation/revocation, read-only visit sessions, visit log, export, deletion recovery window, and account settings completion.

### Phase F: Hardening and launch prep

- Privacy boundary verification, migration rehearsal, load tests, synthetic monitoring, calibration tuning, and staged rollout.

## 13. Rollout plan

### Launch strategy

- Start with internal dogfood using synthetic and employee accounts.
- Launch closed beta with fixed small cohort and bird count capped at two while drift/audiovisual calibration stabilizes.
- Enable age-based third-bird offers only after greeting, call recognizability, and notebook sparsity metrics are healthy.
- Ramp maximum bird availability gradually from 2 to 3 to 5 to 7 across cohort growth, validating chorus recognizability and performance at each step.

### Day-one instrumentation

- Auth success/failure rates.
- Snapshot latency and first-bird render timing.
- Tick duration, backlog, and lock contention.
- Audio engine failure rate and caption fallback rate.
- Accessibility setting adoption and screen-reader error reports.
- Invite creation/revocation success rates.

### Operational playbooks

- If tick latency rises, pause new bird unlocks before changing the core experience.
- If audio engine errors spike in a browser family, force silent-caption mode for that browser while investigating.
- If notebook generation becomes too dense, adjust significance thresholds centrally without shipping client changes.

## 14. Testing strategy

### Simulation correctness

- Deterministic tick replay tests from recorded event sequences.
- Property tests for monotonic drift and no negative-trait movement on neglect.
- Conflict tests ensuring concurrent device sessions never overwrite personality history.

### Client correctness

- Visual snapshot tests for day/night, weather, perch states, reduced-motion mode, and top-bar fade behavior.
- Interaction tests for listen-in, settle undo window, offer cooldowns, and visibility regain refresh.
- Accessibility tests for keyboard traversal, contrast, caption presence, and narration cadence triggers.

### End-to-end flows

- Sign-up and starter-bird naming.
- Multi-device same-state validation.
- Visit invite, read-only session, revocation, and expiration.
- Export and deletion recovery.

### Performance tests

- CI bundle-size gates.
- Automated first-bird rendering budget checks on representative devices.
- 30-minute soak tests for memory stability and audio node reuse.

## 15. Key risks and mitigations

### Drift calibration risk

Risk: birds feel static or feel gameable.
Mitigation: build calibration harnesses early, replay synthetic visit patterns, and gate rollout on one-week measurable / three-week perceptible targets.

### Sync correctness risk

Risk: device races or replay issues silently erase personality evolution.
Mitigation: append-only events, server-only vector writes, per-aviary locks, idempotent event ingest, and replayable audit cursors.

### Audio uncanniness risk

Risk: calls sound repetitive, synthetic in the wrong way, or chorus becomes mush.
Mitigation: species motif libraries with stable per-bird seeding, recognizability tests, capped concurrent calls, and staged bird-count ramp.

### Accessibility regression risk

Risk: accessible surfaces feel like degraded fallbacks.
Mitigation: treat reduced-motion and narration as dedicated workstreams with acceptance criteria tied to product feel, not mere compliance.

### First-frame illusion risk

Risk: loading behavior reveals "app startup" instead of "aviary already alive."
Mitigation: prioritize snapshot bootstrap path, quiet-field loading shell, and no-spinner rule in design and implementation reviews.

### Privacy boundary risk

Risk: engineers accidentally route per-bird interaction data into telemetry or logs.
Mitigation: schema separation, code review checklists, telemetry allowlist, and automated log scanning for disallowed fields.

### Notebook quality risk

Risk: prose becomes repetitive, generic, or too frequent.
Mitigation: constrained generation templates, significance scoring, cadence throttles, and editorial review of sampled outputs before launch.

## 16. Exit criteria for v1 readiness

Pocket Aviary v1 is ready when the team can demonstrate that:

- two starter birds feel alive immediately on load across supported browsers;
- canonical simulation continues across devices and absences without state conflicts;
- drift is measurable in a week and perceptible in roughly three without exposing trait math;
- notebook entries remain sparse, specific, and naturalist;
- reduced-motion, captions, narration, and keyboard paths deliver the actual product rather than a degraded surrogate;
- no gamification, punitive neglect, or accidental social-network surfaces have leaked into the experience;
- privacy and telemetry boundaries have been verified in production-like environments.
