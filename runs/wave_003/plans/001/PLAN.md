# Pocket Aviary v1 Implementation Plan

## 1. Scope

Pocket Aviary v1 is a browser-only product with one canonical aviary per single-user account, magic-link auth, two starter birds, gradual expansion up to seven birds, a server-authored simulation tick, multi-device sync, procedural audio calls, field notebook entries, optional quiet visit invitations, and first-class accessibility surfaces. The core product promise is that the aviary feels like it continues without the viewer and notices the user without announcing them.

In scope:

- Web client for modern browsers on desktop and mobile web
- Account creation, magic-link sign-in, session management, account export, soft-delete and recovery
- Canonical server-side aviary simulation with per-bird personality vectors, mood, drift, and call timing state
- Presence accounting, listen-in, offer, settle, field notebook, and read-only visits
- Accessibility settings, reduced-motion rendering, call captions, keyboard navigation, and screen-reader narration
- Aggregate operational observability that excludes per-account bird relationship data

Out of scope and explicitly blocked in architecture and product surfaces:

- Native apps
- Gamification, streaks, counters, achievements, or user-behavior dashboards
- Tamagotchi-style negative neglect mechanics, hunger, distress, or bird death
- Shared aviaries, co-presence, discovery feeds, follows, comments, profiles, or rankings
- Scene customization, multiple aviaries per account, or public-facing showcase variants

## 2. Product Architecture

### 2.1 System shape

Use a three-surface architecture:

1. Web client: renders the aviary, captures presence and interaction events, synthesizes audio, and presents settings/notebook/social flows.
2. Application API: serves signed-in product APIs, auth/session workflows, read models, invite flows, and accessibility/account surfaces.
3. Simulation service: owns canonical aviary state and performs the minute-scale tick that advances mood, drift, notebook eligibility, weather windows, and call schedules.

Back the services with:

- Relational primary store for accounts, aviaries, birds, invitations, notebook entries, and session records
- Append-only event log table or stream for presence and interaction events
- Short-lived cache for snapshot delivery and invite/session validation
- Email delivery integration for magic links, exports, and visit invitations

### 2.2 Authority boundaries

- The server is the only writer of personality vectors, mood state, perch state, weather state, notebook generation records, and canonical call schedule seeds.
- Clients never mutate bird state directly. Clients only emit interaction events and request snapshots.
- Clients may render ornamental motion that is explicitly non-canonical, such as leaf drift and subtle parallax.
- Visit sessions are read-only render sessions backed by the same snapshot read path with a restricted capability token.

### 2.3 Render boundary

- Canonical state includes enough information to place birds mid-action on first paint: perch zone, pose family, interpolation target, call intent, mood, scene time-of-day, weather, and settle state.
- The client owns interpolation, animation blending, procedural audio synthesis, captions, and reduced-motion presentation.
- Rendering must tolerate late or dropped snapshots without visual snapping by treating the latest snapshot as truth and smoothing toward it.

## 3. Data Model

### 3.1 Core entities

- `account`: synthetic UUID primary key, encrypted email, lifecycle status, verified timestamps, settings, deletion window, notification preferences
- `session`: per-device revocable session token metadata, last seen, device label, auth state
- `aviary`: one-to-one with account, created_at, timezone, bird cap stage, settle state, current daypart, current weather window, visit settings
- `bird`: stable UUID, aviary_id, species, current display name, adopted_at, active flag, current perch zone, current pose state, mood, personality vector, drift totals, call signature seed
- `bird_personality_vector`: embedded or versioned struct with boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity, plus updated_at and provenance version
- `bird_mood_state`: current mood enum, entered_at, decay timer, modifiers, carryover reason
- `presence_window`: derived rolling aggregates rather than user-facing rows; persisted only as needed for simulation correctness
- `interaction_event`: append-only records for presence pings, listen-in start/end, offer submitted, offer reaction, settle start/cancel, visibility resume
- `notebook_entry`: aviary-scoped observation text, observation timestamp, cause tags, rarity score, display order
- `visit_invitation`: host account, invitee email, token hash, expires_at, revoked_at, notifications_enabled_at_send
- `visit_session`: invitation linkage, first_opened_at, last_seen_at, approximate_duration_seconds

### 3.2 Modeling rules

- Personality values are hidden from product UI and accessible surfaces; only internal services and staff tooling can inspect them.
- Drift must be represented as additive history-safe deltas, not replacement values from clients.
- Presence raw events should expire after the simulation window needed for correctness; long-term retention should keep only bounded operational aggregates that do not reconstruct user behavior.
- Notebook entries are immutable after creation.
- Visitor activity never writes host drift inputs.

## 4. API Surface

### 4.1 Auth and account APIs

- `POST /auth/magic-link/request`
- `POST /auth/magic-link/consume`
- `POST /auth/session/revoke`
- `GET /account`
- `PATCH /account/settings`
- `POST /account/export`
- `POST /account/delete`
- `POST /account/delete/cancel`

### 4.2 Aviary read model APIs

- `GET /aviary/snapshot`: returns canonical scene snapshot with birds, scene state, notebook unread count, top-bar capability flags, and visit mode
- `GET /aviary/notebook?cursor=...`
- `GET /aviary/accessibility-state`
- `GET /aviary/visit-log`

Snapshot response should include:

- Snapshot version and server timestamp
- Aviary daypart, settle state, weather, reduced data needed for first-frame rendering
- Bird records with stable ids, names, species, mood, perch zone, pose state, interpolation hints, call schedule seed/timing, and caption metadata seeds

### 4.3 Interaction event APIs

- `POST /aviary/events/presence`
- `POST /aviary/events/listen-in`
- `POST /aviary/events/offer`
- `POST /aviary/events/settle`

Design requirements:

- Events are idempotent through client-generated event ids
- Presence writes are coarse-grained heartbeats, not per-motion spam
- Offer endpoint enforces per-bird cooldown server-side
- Settle endpoint supports cancel/re-engage inside the undo window

### 4.4 Visit APIs

- `POST /visits/invitations`
- `POST /visits/invitations/{id}/revoke`
- `GET /visits/{token}/snapshot`
- `GET /visits/{token}/status`

Visit tokens should be one-time for first entry, then exchange into a short-lived read-only visit session. Visitor surfaces must not expose notebook mutation, interaction posting, or host account metadata beyond what is needed for transparency.

## 5. Simulation Engine Design

### 5.1 Tick cadence and flow

Run the simulation on an approximately once-per-minute cadence per active aviary partition. Each tick:

1. Loads the aviary canonical state and recent unconsumed events.
2. Reconstructs validated presence-time increments from qualifying presence events.
3. Applies mood transition logic based on daypart, weather, recent offers, recent listen-in attention, and bird-to-bird coupling.
4. Computes additive personality drift deltas using calibrated low-pass filters.
5. Advances perch/pose/call intent state for the next snapshot window.
6. Evaluates notebook entry generation candidates with sparsity controls.
7. Persists the new canonical state and marks consumed events.

### 5.2 Drift function

Implement drift as monotonic upward adjustments toward expressive traits, with per-trait weights and strong diminishing returns:

- Presence-time is the dominant driver across boldness, social warmth, vocal frequency, and plumage saturation
- Listen-in adds a stronger targeted effect to social warmth and vocal frequency for the focused bird
- Offers bias curiosity and modestly boldness when the bird engages
- Neglect does not decrement personality values; instead, short-term greeting frequency and visible expressiveness emerge from mood and current state

Calibration targets:

- Instrument-detectable drift after about one week of regular presence
- User-perceptible change after about three weeks
- No visible single-session jumps

### 5.3 Mood system

- Keep a small enumerated mood set: wary, content, curious, drowsy, alert
- Mood transitions should be probabilistic within bounded rules, not deterministic lookup tables
- Daypart and weather provide baseline mood pressure
- Recent interactions provide local nudges
- Personality vector shapes transition probabilities, not hard outcomes
- Mood persists across sessions and is never reset on tab open

### 5.4 Bird-to-bird behavior

- Propagate alarm or wary signals locally across birds with decay
- Allow chorus emergence when multiple birds with compatible call windows and higher vocal frequency overlap
- Use social warmth to influence who greets first and who perches nearer others

### 5.5 Notebook generation

Generate notebook entries from noteworthy state changes and rare combinations, not from every session. Use a rule-plus-template pipeline:

- Candidate detection: first greeter changes, unusual quiet stretches, mood/weather interactions, repeated attention patterns, new bird milestones based on aviary age
- Copy generation: naturalist lowercase prose templates parameterized by observed specifics
- Sparsity gate: target one entry every few days for regular use, with exceptions for genuinely distinctive moments

## 6. Sync Model

### 6.1 Canonical state propagation

- Clients fetch a snapshot on open, on visibility regain, after long suspension gaps, and periodically while visible
- Snapshots are versioned; clients ignore stale responses that arrive out of order
- Client state is disposable and reconstructible from the latest snapshot plus local UI mode

### 6.2 Conflict prevention

- No client-side writes to canonical bird state
- Interaction events are append-only with server ordering
- Personality and mood updates happen only inside the simulation tick transaction
- Use optimistic version checks for snapshot-derived state reads, but never expose merge UI for bird state because divergence should be structurally prevented

### 6.3 Multi-device semantics

- Two devices can emit interactions concurrently; the event log serializes them
- If one device resumes from suspension, it must refresh snapshot before allowing new interaction surfaces that depend on cooldowns or settle state
- Presence qualification remains local per device, but only the account owner's authenticated sessions can contribute

## 7. Frontend Rendering Pipeline

### 7.1 Client application structure

- Initial shell delivers a quiet field immediately
- Snapshot hydration places the aviary into a live scene without spinner-first framing
- Top bar is a sparse overlay with fade behavior and accessibility affordances
- Aviary canvas or DOM/SVG scene graph renders birds, perches, background, and subtle ornaments

### 7.2 Scene composition

- Three perch zones with layered depth cues
- Responsive layout rules that preserve all birds in frame on narrow viewports
- Day/night palette transitions driven from snapshot time-of-day
- Weather overlays and ornamental drift generated client-side within strict limits

### 7.3 Motion strategy

- First frame renders birds mid-action using snapshot pose state
- Continuous idle micro-motion is mood-shaped
- Perch changes interpolate smoothly
- Listen-in ramps audio mix and visual focus gradually
- Settle transitions lighting and audio over several seconds with a five-second undo path

### 7.4 Reduced-motion path

- Replace frame-by-frame motion with cross-fades between authored still poses
- Remove drifting leaf/feather ornaments
- Retain daypart color transitions at slowed cadence
- Keep all interaction affordances and notebook behavior identical

## 8. Audio Pipeline

### 8.1 Procedural synthesis runtime

- Ship compact motif libraries per species
- Use WebAudio oscillators, envelopes, filters, and light noise shaping to synthesize calls at runtime
- Parameterize pitch contour, timing, and spacing by bird-specific seeds, mood, and vocal frequency

### 8.2 Mix model

- Ambient aviary mix is always multi-bird, never hard-muted except global audio-off
- Listen-in raises the focused bird and attenuates others to ambient, not silence
- Chorus events are mixed to preserve per-bird recognizability
- Daypart and settle state influence master ambience and call density

### 8.3 Captions and fallback

- Generate caption strings from the actual synthesized call events
- Anchor captions near the calling bird with accessible text presentation
- If WebAudio is unavailable or blocked, run silent mode with captions on by default and no recorded-audio fallback

## 9. Accessibility Surfaces

### 9.1 Screen-reader narration

- Maintain a dedicated live-region narration channel with slow cadence and event prioritization
- Generate prose from the same state graph used by rendering and notebook logic
- Keep narration observational and naturalist, never stat-list based
- Rate-limit updates to avoid queue flooding; user-triggered reactions can preempt idle narration

### 9.2 Input accessibility

- Full keyboard traversal for top bar, birds, offers, notebook, and settings
- High-contrast focus treatment that survives bright and dim scene states
- Semantic labeling for all controls in matter-of-fact system voice where appropriate

### 9.3 Visual and auditory accessibility

- WCAG AA minimum for all user-copy text and overlays
- Reduced-motion and call-caption settings persisted per account and respected across devices
- Audio-off users still receive a coherent aviary experience through captions and notebook/narration continuity

## 10. Performance Budgets and Observability

### 10.1 Budgets

- Initial JS bundle under 2MB gzipped
- First bird visible within 500ms on mid-tier mobile over 4G
- 60fps idle rendering on a five-year-old mid-range laptop
- No memory growth over 30-minute sessions
- Simulation tick p99 under 5 seconds with substantial headroom

### 10.2 Engineering tactics

- Code-split settings, export, and visit-management surfaces
- Keep snapshots small and edge-cacheable where auth permits
- Reuse audio nodes and buffers aggressively
- Bound scene graph allocations and notebook DOM retention
- Defer non-critical ornaments until after first-bird render

### 10.3 Observability

Collect only aggregate operational metrics:

- Page load and first-bird timing
- Client frame timing and memory trend checks
- Audio-context initialization failures
- API latency and error rates
- Simulation tick latency, failure count, and event backlog depth

Do not collect:

- Per-bird state in telemetry streams
- Per-account interaction histories in analytics warehouses
- Population-level dashboards that summarize individual relationship dynamics

## 11. Rollout Plan

### 11.1 Launch phases

1. Internal dogfood with fixed two-bird aviaries, no social, instrumentation-heavy validation of presence, tick correctness, and audio reliability.
2. Private beta with full account lifecycle, notebook, accessibility surfaces, and invite flow behind explicit feature flags.
3. v1 public launch with visits default-off, two starter birds, and staged unlock scheduling for additional birds by aviary age.

### 11.2 Ramping strategy

- Keep new bird unlocks disabled in earliest internal phases while drift and recognizability are tuned
- Roll out additional bird availability gradually and monitor audio recognizability, render stability, and notebook quality as bird count increases
- Launch visits only after host privacy, revocation, and read-only enforcement are verified end to end

### 11.3 Day-one instrumentation

- Presence qualification rate versus raw foreground sessions
- Tick backlog and skipped-event alarms
- Snapshot freshness on resume from hidden/suspended states
- Audio failure rates by browser family
- Accessibility mode usage rates only in aggregate, without user-behavior profiling

## 12. Testing and Validation Strategy

- Property tests for monotonic drift and no-negative-neglect guarantees
- Simulation replay tests that verify deterministic outputs from ordered event logs
- Concurrency tests for multi-device interaction ordering and invite revocation
- Golden snapshot tests for first-frame render states across dayparts, moods, and bird counts
- Accessibility acceptance tests for screen-reader narration cadence, keyboard traversal, captions, and reduced-motion rendering
- Long-session soak tests for memory stability and audio node reuse
- Browser matrix tests for supported versions and graceful unsupported-browser handling

## 13. Risks and Mitigations

### 13.1 Drift calibration risk

Risk: birds may feel static or may change too quickly.

Mitigation: make drift coefficients remotely configurable, backstop with replayable simulation fixtures, and gate launch on week-scale internal observation rather than only unit metrics.

### 13.2 Sync correctness risk

Risk: stale clients, duplicated events, or concurrent sessions may corrupt canonical state or violate cooldowns.

Mitigation: append-only idempotent events, server-only state mutation, snapshot versioning, and aggressive resume refresh rules.

### 13.3 Audio uncanniness risk

Risk: procedural calls may sound synthetic in a bad way or lose per-bird recognizability in chorus.

Mitigation: species-specific motif tuning, perceptual listening tests, mix caps at seven birds, and silent-caption fallback over poor recorded substitutes.

### 13.4 Accessibility regression risk

Risk: the accessible surface may ship as a stripped fallback that loses the product’s charm.

Mitigation: treat narration, reduced-motion, and captions as launch-blocking first-class deliverables with dedicated acceptance criteria and product review.

### 13.5 Performance risk

Risk: missing the first-bird or bundle budgets breaks the “already alive” illusion.

Mitigation: budget ownership per subsystem, CI perf gates, and ruthless deferral of non-essential assets and code paths.

### 13.6 Privacy boundary risk

Risk: engineers may leak email or per-bird interaction data into logs and telemetry through convenience shortcuts.

Mitigation: synthetic UUID enforcement, schema linting for telemetry payloads, separate simulation and analytics boundaries, and explicit code review checks on PII and bird-state export paths.

## 14. Open Implementation Decisions to Resolve Early

- Exact presence heartbeat interval and inactivity timeout window
- Concrete trait ranges and initial seeding distribution by species
- Notebook generation rule thresholds and copy template library size
- Snapshot transport strategy, polling cadence, and whether server-sent updates are warranted versus pure polling
- Rendering technology choice for scene graph, provided it meets the motion and accessibility requirements

These should be resolved in early engineering design reviews without changing the PRD-level product contract above.
