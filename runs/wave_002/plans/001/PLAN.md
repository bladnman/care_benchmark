# Pocket Aviary V1 Implementation Plan

## 1. Product Reading and Implementation Posture

Pocket Aviary V1 is a web-only, single-user, quietly social virtual aviary. The implementation must protect the product's central affective contract: the aviary feels like a small living place that continues without the viewer, not like an app that starts performing when opened. The highest-risk engineering decisions are therefore not only technical. Loading, simulation ownership, audio variation, presence accounting, privacy boundaries, and accessibility narration all directly shape whether the product feels alive.

The plan below treats the server as the canonical owner of aviary continuity and the browser as a renderer, interaction collector, and procedural audio/runtime animation surface. Clients never own personality state. Clients never tick the simulation. Clients never expose hidden numeric traits to users. User-visible product surfaces use naturalist, lowercase, present-tense prose; system/account/error/accessibility settings surfaces use matter-of-fact prose.

## 2. V1 Scope

### In Scope

V1 includes:

- Modern browser web app only, responsive from phone to desktop.
- Email magic-link authentication with one account per user and one aviary per account.
- Two starter birds at account creation, selected by the system from a coherent species pool of about six species.
- User naming and renaming of birds without changing bird identity.
- A hard V1 cap of seven birds per aviary.
- Age-based availability of additional birds after the initial two, not tied to visit count, streaks, score, or payments.
- Server-side canonical aviary state advanced by a slow simulation tick, approximately once per minute.
- Hidden per-bird personality vectors with monotonic drift toward expressiveness.
- Mood state persisted across sessions and shaped by time of day, recent interactions, ambient events, and personality.
- Procedural client-side calls via WebAudio, with per-bird recognizable call signatures.
- Listen-in, offer, settle, field notebook, presence accounting, and return-greeting interactions.
- Single horizontal aviary scene with three perch zones, day/night cycle, rare ambient weather, top bar controls, and no UI chrome inside the scene.
- Multi-device sync through a single canonical server state.
- Read-only visit invitations by email, off by default, revocable, expiring after 30 days.
- Screen-reader narration, call captions, reduced-motion rendering, keyboard navigation, focus treatment, and WCAG AA user-copy contrast.
- Account export, account deletion with 30-day soft-delete, session management, and email-change verification.
- Aggregate-only operational telemetry and performance observability that never includes per-bird/per-account interaction state.

### Explicitly Out of Scope

V1 excludes:

- Native iOS or Android apps.
- Password auth, SSO, payments, tiers, billing, or subscriptions.
- Shared aviaries, households, multiple profiles, or multi-aviary accounts.
- Customizable scenes, user-controlled bird placement, catalog-based starter bird selection, or avatar-like customization.
- Any gamification surface: achievements, streaks, scores, badges, levels, XP, green-dot calendars, visit-frequency displays, or milestone celebrations.
- Tamagotchi mechanics: hunger, death, visible distress, decaying happiness, caretaker obligations, or punishment for absence.
- Public discovery, profiles, follows, feeds, rankings, comments, chat, co-presence, visitor avatars, or leaderboards.
- Push notifications, email nudges about the aviary, automatic re-engagement loops, or default visit notifications.
- Recorded-audio fallback paths.
- Client-side personality simulation or last-write-wins personality updates.
- Exposing personality vector numbers anywhere in user-facing product, settings, export UI summaries, accessibility labels, captions, or debug-like user surfaces.

The JSON account export may contain current personality vectors because the PRD explicitly requires it. That export must be treated as private account data delivered by verified email link, not as an in-product statistics surface.

## 3. Architecture Overview

### Service Shape

Build V1 as a web application backed by a small set of server modules that can start as a modular monolith and scale into separately deployed services only when load requires it:

- Web client: TypeScript app responsible for rendering, input collection, WebAudio synthesis, reduced-motion rendering, captions, keyboard navigation, and local interpolation between server snapshots.
- API server: authentication, account settings, state snapshot reads, interaction event writes, visit invite flows, account export/deletion, and system/error surfaces.
- Simulation worker: the only writer of canonical aviary state, personality vector deltas, mood transitions, weather state, notebook entries, and age-based bird availability.
- Email worker: magic links, account export download links, email verification, and visit invitations.
- Snapshot cache/edge delivery layer: serves the first state snapshot quickly enough to support first bird visible under 500ms.
- Observability pipeline: aggregate operational metrics only; no per-bird state or per-account interaction history.

### Storage

Use a relational primary store with transactional guarantees for accounts, birds, aviary state, interaction events, notebook entries, invitations, and sessions. The key invariant is event-log ordered simulation, not arbitrary client updates. PostgreSQL is a good fit for V1 because it supports transactional event ingestion, row-level locking for per-aviary ticks, JSON fields for compact state fragments where appropriate, and durable relational boundaries for privacy-sensitive account data.

Use a short-lived cache for snapshots and session reads, but do not make cache state authoritative. Redis or an equivalent queue/cache can support simulation scheduling, idempotency locks, magic-link rate limiting, and snapshot fanout, but any cache loss must degrade to fresh database reads rather than state loss.

### Boundary Between Canonical and Ephemeral State

Canonical server state includes:

- Account, sessions, email verification state, deletion state.
- Aviary ID, creation time, local timezone, current day/night anchor, active weather summary, settled status.
- Bird identities, names, species, personality vectors, current moods, mood timers, perch-zone intentions, current call scheduling seeds, and adoption availability state.
- Interaction event log records submitted by the account owner.
- Field notebook entries.
- Visit invitations and visit log records.

Client ephemeral state includes:

- Interpolated bird animation progress between snapshots.
- Local procedural micro-motion phase.
- Ambient leaves/feathers and local-only decorative drift.
- Current listen-in mix ramp state before event acknowledgement, reconciled to server accepted events.
- Local focus state, top bar fade state, captions display timing, audio context state, reduced-motion rendering mode.

The client may optimistically render reversible UI transitions for responsiveness, but the next snapshot is always authoritative for canonical bird/mood/aviary state.

## 4. Core Data Model

### Account

Fields:

- id: synthetic UUID used everywhere outside the account record.
- encrypted_email: encrypted verified email, never used as a partition key or telemetry identifier.
- email_verified_at.
- created_at, updated_at.
- deletion_requested_at, hard_delete_after.
- timezone: IANA timezone used for day/night and mood signals; updated conservatively from client with account-level confirmation when it changes materially.
- settings: accessibility preferences, audio/caption preferences, visit-notification opt-in, reduced-motion preference override, privacy policy acknowledgement if needed.

Rules:

- Email appears only on the account record and email delivery jobs.
- Logs, metrics, event messages, and sharding keys use account UUID or non-reversible internal identifiers.
- Soft-deleted accounts cannot write interactions and are excluded from simulation except for deletion recovery checks.

### Session

Fields:

- id: UUID.
- account_id.
- device_label: user-visible, generated from browser/device metadata in matter-of-fact language.
- token_hash.
- created_at, last_seen_at, revoked_at.
- user_agent_family and approximate device class for session list display.

Rules:

- Session tokens are per-device and revocable.
- Revoked or expired sessions cannot request snapshots or write events.

### Aviary

Fields:

- id: UUID.
- account_id unique.
- created_at.
- current_state_version: monotonic integer incremented by simulation tick.
- settled_state: active/inactive plus last_settled_at.
- current_weather: none/rain/wind with start/end and intensity.
- local_day_phase: derived on tick from account timezone, not hand-edited by clients.
- next_bird_available_at and bird_count_cap.
- last_tick_at.

Rules:

- One aviary per account for V1.
- The state version is returned in snapshots and referenced by interaction writes for diagnostics, but clients do not submit authoritative state.

### Bird

Fields:

- id: stable UUID.
- aviary_id.
- species_id.
- display_name.
- adopted_at.
- name_updated_at.
- personality_vector: hidden normalized values for boldness, social warmth, vocal frequency, plumage saturation, curiosity.
- personality_version: increments only from simulation ticks.
- mood: enum such as wary, content, curious, drowsy, alert, settled.
- mood_started_at, mood_expires_at.
- perch_zone: front/middle/back plus optional perch slot.
- motion_intent: current canonical behavior family such as preen, scan, call, rest, approach_offer.
- call_signature_seed: stable per-bird seed for recognizable procedural motif selection.
- recent_offer_cooldowns: per offer type or per bird timestamp.

Rules:

- Bird ID is permanent and survives renaming and future migrations.
- Personality values are stored server-side and updated only by the simulation worker.
- Personality values are not sent to ordinary client surfaces. Snapshots send derived render/audio parameters, not raw trait numbers.
- Export jobs can include raw vectors because the account export is explicitly private account data.

### Species

Fields:

- id.
- common_name/internal label.
- silhouette family.
- default palette and plumage parameter ranges.
- call motif library definition.
- motion pose set availability.
- active-at-night flag for the nightjar-like species.

Rules:

- Species rarity is not a V1 feature.
- Species selection for starter birds should feel like arrival, not user configuration.

### Interaction Event

Append-only records:

- id: UUID idempotency key supplied by client or generated server-side.
- account_id, aviary_id.
- actor_type: owner or visitor; visitor events are render/session telemetry only and never simulation inputs.
- bird_id nullable.
- event_type: presence_ping, listen_in_start, listen_in_end, offer_seed, offer_song_fragment, offer_still_pool, settle, settle_undo, rename_bird, adoption_name_set.
- client_observed_state_version.
- occurred_at_client, received_at_server.
- payload: constrained typed JSON for event details.
- consumed_by_tick_at nullable.

Rules:

- The event log is the only client-to-server path for simulation inputs.
- Presence events are recorded only when visibility, focus, and recent pointer/key activity are all true.
- Visitors cannot create simulation-input events.
- Events are processed in server receive order with monotonic safeguards; client timestamps are advisory for session shape, not authority for reorderable state mutation.

### Presence Window

Derived from presence_ping events:

- account_id, aviary_id.
- started_at, ended_at.
- total_present_seconds.
- source_event_ids.
- device/session id.
- quality flags such as clock_skew_detected, long_gap_split.

Rules:

- Presence ends on settle or tab close/unload if a terminal event arrives; otherwise it times out after calibrated inactivity.
- A visible but idle tab without recent pointer/key activity does not accrue presence.
- A focused browser window without visible document does not accrue presence.

### Notebook Entry

Fields:

- id.
- aviary_id.
- created_at.
- entry_date_local.
- prose: naturalist lowercase present-tense observation.
- source_kind: greeting_pattern, weather_moment, quiet_stretch, mood_shift, bird_order, adoption, rare_chorus, etc.
- source_refs: internal references to events/state versions, never shown to user.

Rules:

- Entries are rare: roughly every few days for regularly visited aviaries, more often only for genuinely notable moments.
- Entries are read-only and never user-editable.
- Entries describe the aviary, not the user's engagement pattern.

### Visit Invitation and Visit Log

Invitation fields:

- id.
- host_account_id, aviary_id.
- visitor_email_encrypted.
- token_hash.
- created_at, expires_at, used_at, revoked_at.
- active_visit_session_id nullable.

Visit log fields:

- id.
- invitation_id.
- host_account_id.
- visitor_email_encrypted for host settings display only.
- started_at, ended_at, approximate_duration_seconds.

Rules:

- Visit links are one-time or session-bound as specified by product decision during implementation; outstanding invites expire after 30 days.
- Host can revoke outstanding or active invites immediately.
- Visitor sees a read-only ambient view and creates no simulation inputs.
- Host gets no default push/email/in-product notification. Optional visit notifications are off by default and not promoted during onboarding.

## 5. API Surface

All API responses that represent system/account/error flows use matter-of-fact copy. Product prose is carried as state or generated content for aviary/notebook/narration surfaces.

### Authentication and Account

- POST /auth/magic-link
  - Input: email.
  - Behavior: create one-time link expiring in 15 minutes, rate limited by email and IP, always respond generically.

- GET /auth/consume?token=...
  - Behavior: consume unused token, create session, redirect to aviary or onboarding. Used tokens are invalidated immediately.

- GET /account
  - Returns account settings, session list, accessibility/audio preferences, deletion state.

- PATCH /account/settings
  - Updates matter-of-fact settings only: captions, audio default, reduced-motion override, visit notifications, timezone confirmation.

- POST /account/email-change
  - Starts new-email verification.

- POST /account/email-change/confirm
  - Commits verified email switch.

- POST /account/export
  - Queues JSON export and emails a short-lived verified download link.

- POST /account/delete
  - Starts 30-day soft deletion.

- POST /account/restore
  - Restores account within soft-deletion window.

- DELETE /sessions/{sessionId}
  - Revokes a device session.

### Aviary State and Interaction

- GET /aviary/snapshot
  - Returns compact canonical snapshot for owner.
  - Includes state_version, server_time, aviary day phase/weather/settled state, birds with derived render parameters, mood descriptors, perch zones, current motion intents, call grammar runtime seeds, offer cooldown availability, top-level notebook unread marker if appropriate.
  - Excludes raw personality vector values.
  - Supports ETag/state_version conditional fetch.

- GET /aviary/bootstrap
  - First-load endpoint optimized for time-to-first-bird, potentially edge-cached per session with strict privacy controls.
  - Returns the minimum renderable state: scene parameters, birds, positions, current moods, call scheduling seeds, and enough account UI state to draw first frame.

- POST /aviary/events
  - Input: batch of typed interaction events with idempotency keys.
  - Behavior: validate ownership/session, validate presence conjunctive evidence where applicable, append events, return accepted/rejected event ids and current state_version.
  - Does not mutate personality or mood directly.

- POST /aviary/birds/{birdId}/name
  - Renames bird through a typed event plus direct metadata update, preserving bird identity.

- POST /aviary/adoptions
  - When age-based availability exists, accepts the new bird arrival naming flow.
  - Does not expose a catalog or rarity mechanics.

- GET /aviary/notebook
  - Paginates read-only entries newest-first or chronological by UI choice.
  - Does not include generic event logs or user-behavior summaries.

### Visit Flow

- POST /visits/invitations
  - Host inputs visitor email. Creates revocable invitation, sends one-time link, expires in 30 days.

- GET /visits/invitations
  - Host settings surface listing outstanding/active invites and visit log.

- DELETE /visits/invitations/{inviteId}
  - Revokes outstanding or active invite.

- GET /visit/consume?token=...
  - Visitor consumes valid invite link and receives a visitor session for read-only viewing.

- GET /visit/snapshot
  - Returns host aviary render snapshot to visitor, with no notebook mutation and no host-specific account controls.
  - If revoked/expired, returns matter-of-fact unavailable surface.

Visitor snapshot reads can be counted in aggregate request metrics and host visit logs, but visitor presence never enters the host simulation event log.

## 6. Simulation Engine Design

### Tick Ownership

The simulation worker runs a slow per-aviary tick, initially every 60 seconds with room for calibration. It is the only writer for:

- Personality vector deltas.
- Mood transitions.
- Canonical perch zone and motion intent.
- Weather state transitions.
- Settled state expiration/re-engagement effects.
- Notebook-entry generation.
- New-bird availability scheduling.
- Snapshot state_version increments.

Use a per-aviary lock or lease so only one worker ticks an aviary at a time. The tick must be idempotent across retries: the worker records the last consumed event id/time and avoids applying the same event twice.

### Tick Inputs

Each tick reads:

- Current canonical aviary and bird state.
- Unconsumed owner interaction events since last tick.
- Derived presence windows and recent presence-time.
- User local time/day phase.
- Active or scheduled ambient weather.
- Offer cooldown state.
- Bird-to-bird call/mood influence state.
- Aviary age for adoption availability.

It does not read aggregate analytics and does not consult cross-account population data.

### Drift Function

Implement drift as a low-pass filter over recent and accumulated positive signals. Calibrate around the PRD target:

- Instrument-measurable drift after roughly one week of regular visits.
- User-visible drift after roughly three weeks.
- No single session creates visible personality change.

Trait effects:

- Presence-time: dominant input, nudges birds toward expressiveness broadly while respecting per-trait caps.
- Listen-in: stronger per-bird signal for social warmth and vocal frequency.
- Offers: accepted offers nudge curiosity; offers near a bird nudge boldness modestly.
- Settle: mood-quieting and presence-ending signal, not a direct personality drift boost.

Monotonic expressive rule:

- Positive signals can move traits upward within calibrated bounds.
- Neglect does not reduce boldness, warmth, vocal frequency, plumage saturation, or curiosity.
- Absence can make current mood quieter or less greeting-prone through fast-timescale mood and recent-signal decay, but it cannot make personality regress.

Implementation detail:

- Store raw normalized trait values server-side.
- Derive render/audio parameters through bounded mapping functions so small numeric changes do not produce sudden visual/audio jumps.
- Keep calibration constants versioned. Include migration hooks so future tuning can apply carefully without resetting birds.

### Mood System

Mood is a per-bird enum with timers and transition weights. Initial V1 moods should include wary, content, curious, drowsy, alert, and settled or an equivalent internal settled state.

Transition inputs:

- Time of day: morning favors alert/content; dusk favors drowsy/settled; full night settles most birds except night-active species.
- Recent interactions: accepted offers favor content/curious; listen-in can favor alert/social response; settle favors drowsy/settled.
- Ambient weather: rain dampens vocal frequency and can soften or quiet; wind can increase alert/wary weights.
- Personality: bold birds resist wary transitions; curious birds approach offers; warm birds greet and respond more.
- Bird-to-bird influence: alarm-like calls can spread wary; high vocal frequency can join chorus events.

Persistence:

- Mood persists across sessions.
- Opening the tab must never reset mood to neutral.
- Snapshot load should show the mood the server has advanced to, with client interpolation only for visual continuity.

### Return-Greeting Selection

On a new visible owner session or return from absence, the server snapshot should include a greeting opportunity descriptor, or the client should request one through an event that the tick/snapshot layer validates. The rule should select one primary bird, not all birds.

Inputs:

- Absence length since last owner presence.
- Bird boldness, warmth, current mood, perch zone, and recent greeting history.
- Small procedural randomness seeded by aviary/bird/session so it is varied but stable for the moment.

Behavior:

- Short absence: glance, head tilt, quiet call.
- Longer absence: approach, longer call, response from another bird.
- Multiple eligible birds stagger by randomized offsets; no simultaneous on-cue chorus.
- No text welcome, no absence banner, no toast.

### Offer Handling

Offer events are gestures, not inventory or feeding mechanics. The engine validates per-bird cooldowns of a few minutes to prevent trait saturation.

Offer types:

- Seed: curious/content birds approach; wary birds wait; drowsy birds may ignore.
- Song fragment: plays a melodic motif; birds may join, quiet, or call against it based on vocal frequency and mood.
- Still pool: creates a temporary scene element; birds may drink, bathe, or watch.

Offer outcomes should update mood and future drift inputs, not display success/failure scoring. Avoid any UI language implying reward, feeding, or obligation.

### Field Notebook Generation

Notebook generation runs in the simulation worker and chooses sparse observations from notable state patterns. Candidate triggers:

- A bird greets before another for the first time in a while.
- Rare quiet stretch.
- Weather plus mood/perch combination.
- Bird-specific drift crossing a hidden expressive threshold that manifests as a visible behavior, without naming the number.
- A new bird arrival.
- A rare chorus.

Generation rules:

- Lowercase, present-tense, naturalist voice.
- Specific to named birds and moments.
- Not an event log.
- Not every session.
- Never summarizes user visit frequency or praises engagement.
- Never exposes trait values.

Use templated prose with controlled variation for V1 rather than unconstrained model generation. This keeps privacy, tone, latency, and testability under control. A prose rule engine can combine approved fragments with state facts and should have golden tests for voice violations.

## 7. Sync Model and Conflict Prevention

### Canonical State Propagation

Clients pull snapshots from the server and interpolate locally. Multi-device sync works because each device reads the same canonical state. There is no client-to-client sync and no merge of client-owned personality state.

Snapshot strategy:

- Pull on initial load.
- Pull when visibility changes to visible.
- Pull after long render-frame gaps or wake-from-sleep detection.
- Pull on low-frequency keepalive while visible.
- Pull immediately after event submission if server indicates accepted events may affect near-term render state.

### Event Ingestion

Interaction events are appended with idempotency keys. Clients can retry safely. The server validates:

- Session ownership.
- Account not deleted/revoked.
- Event type allowed for owner vs visitor.
- Bird belongs to aviary.
- Offer cooldown constraints.
- Presence ping contains the three required client-side facts.

The server stores rejected event reasons for debugging but returns only matter-of-fact, user-appropriate errors where visible.

### Conflict Rules

- Personality vectors: no client writes, no last-write-wins, no merge UI.
- Mood: simulation worker writes only; clients can submit interaction events that influence future mood.
- Bird rename: last accepted owner rename can win because names are direct metadata, not drift. Record name history only if needed for audit/export, not as a user-facing feature.
- Settle: settle and settle undo are short-window events; if multiple devices conflict, the latest accepted event within the undo window determines settled visual state, but presence accounting still closes or reopens based on accepted events.
- Visit revocation: revocation wins immediately; visitor snapshot returns unavailable on next pull.

### Offline and Poor Connectivity

V1 should support graceful short disruptions, not an offline product:

- Existing rendered scene can continue visually from the last snapshot for a short period, with clear internal stale-state handling.
- Interaction submissions queue briefly and retry with idempotency keys while the session is valid.
- If the snapshot is too stale, show a matter-of-fact system surface: Something went wrong loading your aviary. Try reloading.
- Do not let offline clients simulate personality or advance canonical mood.

## 8. Frontend Rendering Pipeline

### Rendering Technology Boundary

Use a deterministic scene renderer with a clear separation between state interpretation and drawing. A practical V1 stack is:

- React or equivalent for app shell, top bar, settings, notebook, auth, visit flow, and accessibility panels.
- Canvas/WebGL or highly optimized SVG/DOM hybrid for the aviary scene, depending on asset direction and performance tests.
- A scene runtime module that consumes server snapshots and emits render commands, animation phases, captions, and audio scheduling cues.

Keep the bird engine domain model out of UI components. UI components should consume derived view models, not raw server records.

### First Frame and Loading

The first frame must be the aviary or a quiet field, never a machine-like spinner.

Implementation path:

- Server-render or edge-inline minimal bootstrap data when possible.
- Draw quiet field immediately from CSS/design tokens.
- As soon as the minimal snapshot arrives, draw birds in current mid-action pose, not at an entry default.
- Defer notebook/settings/visit code and non-critical assets.
- Avoid fade-from-static; the scene should appear already in motion.

For slow connections, the quiet field may show subtle non-bird motion cues, but no spinner, progress bar, welcome copy, or app-like skeleton.

### Scene Composition

Render a single horizontal scene that always fits in the viewport:

- Background sky/foliage plane.
- Middle plane with perches and birds.
- Optional foreground branch/leaf plane.
- Three logical perch zones: front, middle, back.
- Top bar above the scene, not inside it.

Responsive rules:

- Preserve all birds in frame at all supported sizes.
- Compress spacing on narrow viewports without cropping birds.
- Widen on desktop without turning the scene into an explorable geography.
- No panning, scrolling, or zooming.

### Bird Rendering

Bird render state derives from species, mood, perch zone, motion intent, and derived personality render parameters:

- Boldness influences front/back presence and approach behaviors.
- Plumage saturation influences visual richness within bounded natural palettes.
- Mood controls pose family and idle micro-motion.
- Species controls silhouette and motif-specific motion/call identity.

Idle micro-motion:

- Preening, scanning, head tilts, body shuffles, quiet rests.
- Never perfectly still in a paused way while visible.
- Motion loops must use procedural variation, phase offsets, and mood/personality selection to avoid canned cycles.

Reduced-motion mode:

- Replace micro-animation with slow cross-fades between still poses.
- Replace flight paths with perch-to-perch cross-fades.
- Remove leaf drift.
- Keep day/evening color changes slowed.
- Keep calls, captions, notebook, mood, and drift intact.

### Top Bar and Controls

Top bar contains only:

- Account/settings.
- Accessibility settings.
- Field notebook.
- Offer affordance.
- Settle affordance, if represented as separate or within the sparse control set.

Rules:

- Fade nearly transparent after a few seconds of cursor stillness.
- Return on pointer movement or keyboard activity.
- No badges for visit logs, notebook updates, achievements, or visits.
- No hover tooltips inside the aviary scene.
- Keyboard focus must bring controls visibly back.

### Interaction Flows

Return:

- Client renders snapshot immediately.
- Greeting descriptor triggers one bird's varied notice behavior within one to two seconds.
- No textual welcome or absence copy.

Listen-in:

- Click/tap/focus bird to engage.
- Gradual audio mix ramp up for focused bird; other birds lower to ambient, never silent.
- Disengage on repeated bird activation, another bird focus, empty-scene click, or focus exit.
- Submit listen_in_start/end events.

Offer:

- Open from top bar, not by clicking a bird directly.
- Choose seed, song fragment, or still pool from a small, calm affordance.
- Render reaction shaped by snapshot/engine response.
- Enforce cooldown quietly; avoid punitive language.

Settle:

- Trigger from top bar.
- Shift lighting to evening over a few seconds; quiet calls.
- Any click in aviary within five seconds undoes.
- Submit settle or settle_undo events.
- Closing without settle is ordinary and produces no recovery surface.

Notebook:

- Read-only naturalist entries.
- Infinite/long scroll with memory-safe virtualization.
- No edit/delete/annotate.
- No visit-frequency summaries.

## 9. Audio Pipeline

### Procedural Call Runtime

Implement a WebAudio call engine that synthesizes calls from per-species motif libraries and per-bird stable seeds.

Core concepts:

- Species motif library: allowed pitch contours, note counts, intervals, timbre envelopes.
- Bird call signature seed: stable variation identity so Pip remains recognizable.
- Mood modifiers: tempo, brightness, pause length, call sharpness.
- Personality modifiers: vocal frequency affects call likelihood and chorus joining; warmth affects response probability.
- Runtime variation: each call differs in timing, pitch microvariation, envelope, and spacing.

Calls are scheduled client-side from snapshot-provided state and seeds, but the long-term call likelihood and bird state are server-derived. The client should not invent personality drift; it can vary individual calls within allowed grammar.

### Mixing

Default mix:

- Multiple birds audible as a soft ambient chorus.
- Per-bird spatialization can be subtle, tied to perch zone, but should not feel like a technical demo.
- Ambient weather and scene tones remain quiet.

Listen-in mix:

- Focused bird ramps up slowly.
- Other birds ramp down to ambient, not zero.
- Disengagement ramps back to default.
- Avoid hard cuts and track-solo metaphors.

Chorus:

- When two or more birds call in overlapping windows, synthesize actual overlapping procedural calls.
- Avoid stacking identical loops or fixed samples.
- Use per-bird phase and grammar variation to prevent combing/loop artifacts.

### Captions and Fallback

Captions derive from the same procedural call grammar that generated the audible call:

- Example caption shape: a soft three-note rise.
- Short, naturalist, near the calling bird.
- Fade with the call.
- Useful for audio-off and hearing differences.

If WebAudio is unavailable:

- Do not load recorded audio.
- Play in graceful silence.
- Turn captions on by default for that session unless the user has explicitly disabled them and can still access the setting.
- Surface any system limitation in matter-of-fact copy only where necessary.

### Audio Performance

- Reuse oscillators/buffers/envelopes where possible.
- Bound concurrent voices based on bird cap and weather/ambient channels.
- Avoid per-call allocations that leak over 30 minutes.
- Track aggregate audio-context errors, initialization failures, and underrun-like symptoms without per-bird/account payloads.

## 10. Accessibility Surfaces

### Screen-Reader Narration

Build a narration layer that consumes the same derived state as visual rendering and emits naturalist prose on a slow cadence.

Requirements:

- Idle updates roughly every 30 to 60 seconds.
- Prompt updates for return-greeting, offer reactions, settle, and other user-initiated events.
- Lowercase, present-tense, specific prose.
- No state-list phrasing such as mood: content or perch 2.
- No hidden trait numbers.
- No high-frequency queue spam.

Implementation:

- Maintain a narration queue with priority levels.
- Coalesce low-priority ambient changes.
- Use approved templates and state facts.
- Announce matter-of-fact system errors through normal accessible alert patterns.

### Keyboard Navigation

- Tab reaches top bar controls.
- Tab can enter aviary bird focus group.
- Arrow keys move between birds.
- Enter toggles listen-in for focused bird.
- Escape exits listen-in.
- Offer menu is fully keyboard navigable.
- Settle is keyboard reachable.
- Focus indicators remain visible against morning, evening, and night palettes.

### Reduced Motion

Honor both OS prefers-reduced-motion and explicit in-app accessibility settings. The in-app setting can override default behavior in a matter-of-fact settings surface.

Reduced motion is a designed rendering mode, not a static fallback. It must be tested alongside default rendering in CI and visual QA.

### Captions and Contrast

- Call captions are available through accessibility settings and automatically enabled for WebAudio failure.
- Caption placement must not obscure controls or imply UI labels inside the aviary.
- All user-copy text passes WCAG AA.
- Test contrast across day/night/weather and focus states.

## 11. Performance Budgets and Observability

### Budgets

Initial JS:

- Less than 2MB gzipped at first paint.
- Split account settings, accessibility settings, notebook history, visit flows, and export/delete flows.
- Keep motif libraries compact and procedural.

Time to first bird:

- Less than 500ms on a mid-tier mobile device over 4G.
- Use minimal bootstrap snapshot and immediate quiet-field draw.
- Do not wait for notebook/settings bundles or full asset preloads.

Runtime:

- 60fps idle motion on a five-year-old mid-range laptop.
- No memory growth over a 30-minute session.
- Pause rendering when hidden; resume from fresh snapshot when visible.
- Continue server simulation regardless of client visibility.

Simulation:

- Tick p99 latency alarm at 5 seconds.
- Tick should usually complete far below that.
- Backpressure and retries must not double-apply events.

### Observability

Allowed aggregate metrics:

- Request counts and latencies.
- Snapshot payload size and cache hit rates.
- First-bird-render timings.
- Client render-frame timings.
- Audio-context initialization/error counts.
- Simulation tick latency and failure counts.
- Magic-link send/consume success rates.
- Export/delete job success rates.
- Anonymized session-duration histograms without account dimensions.

Forbidden telemetry:

- Per-bird state in analytics.
- Raw personality vectors in dashboards.
- Per-account interaction histories in aggregate tools.
- Cross-account average drift dashboards.
- Visit-frequency surfaces for users.
- Any data path that makes future leaderboards or engagement ranking easy.

Use synthetic monitoring with seeded test accounts that contain artificial data expressly created for monitoring. Keep those accounts isolated from production user analytics and clearly marked.

## 12. Privacy and Security Plan

- Use synthetic account UUIDs everywhere except the encrypted email field on account records and email delivery workflows.
- Encrypt emails and visitor invite emails at rest.
- Hash magic-link, visit-link, and session tokens.
- Expire magic links after 15 minutes and invalidate on use.
- Rate-limit magic-link requests per email and IP with generic responses.
- Require verification before email changes commit.
- Make account export links short-lived and deliver only to verified email.
- Soft-delete immediately hides/disables account writes; hard-delete after 30 days removes birds, vectors, notebook, telemetry links, invitations, sessions, and account records as required.
- Ensure telemetry deletion/anonymization respects the hard-delete promise where metrics could still be account-linked.
- Keep simulation data out of analytics warehouses.
- Add automated tests that fail builds if event payload schemas containing bird/personality fields are imported into analytics emitters.

## 13. Rollout Plan

### Phase A: Foundations

- Account model, magic-link auth, sessions, synthetic UUID invariant.
- Database schema for account, aviary, bird, event log, notebook, invitations.
- Minimal API server and simulation worker skeleton.
- Snapshot endpoint with fixture birds.
- Frontend quiet-field and first bird render path.
- CI checks for bundle size, schema ownership, and linted voice fixtures.

Exit criteria:

- New account can sign in, create one aviary, receive two starter birds, and render them from a server snapshot.
- No client code can write personality fields.

### Phase B: Simulation and Presence

- Presence conjunction detector in client.
- Event ingestion with idempotency.
- Server tick consumes events and updates mood/personality deltas.
- Mood persistence across sessions.
- Return-greeting descriptor and stagger logic.
- Offer cooldowns and settle/undo.

Exit criteria:

- Multi-device clients see same canonical state.
- Drift calibration test shows measurable changes at one-week simulated regular presence and no visible single-session jump.
- Absence causes no negative personality drift.

### Phase C: Rendering and Audio

- Species pool assets and silhouette/pose system.
- Three perch zones and responsive scene.
- Procedural call grammar runtime.
- Listen-in mix ramps.
- Day/night cycle, weather, ambient micro-motion.
- Reduced-motion renderer.

Exit criteria:

- First bird visible within budget in test harness.
- 60fps and no memory growth over 30-minute automated session.
- No repeated identical call output in procedural-call tests.

### Phase D: Product Surfaces

- Field notebook generation and UI.
- Accessibility narration queue.
- Captions from call grammar.
- Keyboard/focus flows.
- Account export/deletion/settings.
- Visit invitations, visitor snapshots, revocation, visit log.

Exit criteria:

- Screen-reader audit confirms naturalist narration rather than state-list labels.
- Visit sessions cannot submit simulation events.
- Revocation takes effect on next visitor snapshot.
- Account export includes required private data and is delivered securely.

### Phase E: Private Beta and Calibration

Start with internal seeded accounts and then a small private beta. Ramp constraints:

- Begin with two birds only for all accounts.
- Keep additional bird availability disabled until drift/audio/readability calibration passes.
- Enable third-bird availability for older beta aviaries first; do not exceed five birds in early beta.
- Move toward seven only after chorus recognizability and performance budgets hold.

Instrument from day one:

- Performance, latency, errors, tick health, first-bird timing, audio failures.
- Synthetic drift calibration on test accounts.
- Accessibility regression checks.

Do not instrument per-account behavior for product analytics. Product calibration should use synthetic accounts, explicit internal test accounts, QA sessions, and qualitative beta feedback, not population-level mining of private bird interactions.

## 14. Testing Strategy

### Unit Tests

- Drift monotonicity and boundedness.
- Presence conjunction logic.
- Mood transition weights and persistence.
- Offer cooldown enforcement.
- Event idempotency and tick consumption.
- Magic-link expiration/use invalidation.
- Visit revocation and expiration.
- Voice template linting for banned gamification/system phrases.
- Caption generation from call grammar.

### Integration Tests

- Two devices submit events; simulation consumes in order; both receive same snapshot.
- Client retry does not duplicate interaction effects.
- Session revocation blocks snapshots and events.
- Visitor read-only session cannot greet, offer, listen-in, settle, or create presence drift.
- Soft-deleted account cannot write and can restore within window.
- Hard-delete job removes account-linked simulation records.

### End-to-End Tests

- New user signs in, names two birds, sees aviary without spinner/welcome toast.
- Return after simulated absence triggers one varied bird greeting.
- Listen-in ramps audio and disengages correctly.
- Offer creates mood-shaped reaction and cooldown.
- Settle shifts lighting and undo works within five seconds.
- Notebook shows sparse naturalist entries, not event logs.
- Keyboard-only user can operate all interactions.
- Reduced-motion user receives cross-fade rendering.
- WebAudio unavailable path enables captions and ships no recorded fallback.

### Performance Tests

- Bundle budget gate under 2MB gzipped initial JS.
- First-bird visible under 500ms on emulated mid-tier mobile over 4G.
- 60fps idle on representative five-year-old laptop profile.
- 30-minute memory plateau test.
- Tick p99 latency synthetic load test.

### Privacy and Telemetry Tests

- Static checks block importing simulation schemas into analytics emitters.
- Event payloads are not logged with per-bird details.
- Account UUID, not email, appears in logs where identifiers are unavoidable.
- Export and delete jobs cover all account-linked tables.

## 15. Key Risks and Mitigations

### Drift Calibration Too Fast or Too Slow

Risk: birds change visibly between sessions or never feel changed after weeks.

Mitigation:

- Version calibration constants.
- Build synthetic time-travel tests for one day, one week, three weeks, and absence windows.
- Use hidden internal QA visual/audio comparisons, not user-facing stats.
- Keep drift low-pass and bounded; tune before enabling additional birds broadly.

### Presence Signal Inflation

Risk: background tabs or idle machines count as attention, corrupting drift.

Mitigation:

- Enforce visibility, focus, and recent pointer/key activity in client and server validation.
- Split presence windows on long gaps.
- Do not accept presence from visitors.
- Add tests for hidden tab, unfocused window, no recent activity, and suspended laptop.

### Sync Correctness Regression

Risk: client code or future API accidentally writes personality state or overwrites drift.

Mitigation:

- Keep personality writes in simulation worker module only.
- Restrict database permissions by service role if infrastructure allows.
- Add schema/code ownership tests and API contract tests.
- Review any endpoint that touches bird records for personality write paths.

### Audio Sounds Canned or Uncanny

Risk: procedural calls repeat too obviously, chorus blurs, or listen-in feels like track soloing.

Mitigation:

- Use stable bird identity plus runtime variation.
- Golden audio snapshots for repeated-call difference checks.
- Human QA for recognizability at two, five, and seven birds.
- Avoid hard mix cuts; test ramp durations.

### Accessibility Surface Feels Like a Fallback

Risk: screen-reader, captions, or reduced-motion users receive flattened state labels instead of the product.

Mitigation:

- Treat narration/captions/reduced-motion as primary product surfaces with design review.
- Add voice lint tests and screen-reader QA.
- Keep reduced-motion in visual regression suite.
- Ship accessibility with V1, not as follow-up.

### Privacy Boundary Erosion

Risk: useful-looking analytics pull in per-bird or per-account interaction state.

Mitigation:

- Separate simulation database access from analytics emitters.
- Define allowed metrics schemas early.
- Add static and runtime checks for forbidden fields.
- Document that aggregate drift analytics are disallowed even if anonymized.

### Product Surface Drifts Toward Gamification

Risk: contributors add small counters, badges, visit prompts, or welcome text because they are conventional.

Mitigation:

- Maintain banned surface checklist in product acceptance criteria.
- Add UI copy linting for welcome back, streak, achievement, level, score, days visited, and similar phrases.
- Require design review for any new top-bar item, notification, notebook entry type, or settings surface.

### Performance Breaks Felt-Aliveness

Risk: slow initial render forces spinner/loading patterns and undermines the continuing-place conceit.

Mitigation:

- Gate bundle and first-bird budgets in CI.
- Keep first snapshot minimal.
- Code split non-aviary surfaces.
- Prefer procedural compact assets over heavy media.

## 16. Engineering Organization

Recommended workstreams:

- Platform/auth/privacy: accounts, sessions, deletion/export, synthetic IDs, email flows.
- Simulation: data model, event log, tick worker, drift, mood, notebook generation.
- Rendering: scene runtime, bird assets, responsive layout, day/night/weather, reduced motion.
- Audio: procedural grammar, WebAudio synthesis, mixing, captions, fallback.
- Accessibility: narration, keyboard, focus, contrast, reduced-motion QA.
- Social/visit: invitation, visitor snapshot, revocation, visit log.
- Observability/performance: budgets, synthetic monitoring, aggregate-only RUM, CI gates.

Each workstream should expose narrow contracts. In particular, rendering/audio should consume derived bird view models and call grammar descriptors, not raw personality vectors; observability should consume aggregate timings and errors, not simulation records; and visits should consume read-only snapshots, not owner interaction APIs.

## 17. Acceptance Criteria Summary

V1 is acceptable when:

- A new user can sign in by magic link, name two starter birds, and see the aviary without spinner, welcome toast, or app-like launch ceremony.
- The aviary continues through server-side ticks while no client is connected.
- Two devices show one canonical aviary with no personality merge conflicts.
- Presence is counted only under the required three-signal conjunction.
- Drift is measurable after about one week of regular presence and visible only after about three weeks, with no negative drift from neglect.
- Calls are procedural, varied, recognizable per bird, and captioned from the same grammar.
- Listen-in, offer, settle, notebook, visits, export, deletion, and accessibility settings work within the tone and scope constraints.
- Screen-reader and reduced-motion users receive a designed aviary experience, not a stripped fallback.
- Initial bundle, first-bird, 60fps, memory, and tick latency budgets pass automated checks.
- Telemetry remains aggregate-only and cannot reconstruct per-bird/per-account relationships.
- No gamification, Tamagotchi, notification, public social, native app, or user-controlled scene/customization surfaces have slipped into the product.
