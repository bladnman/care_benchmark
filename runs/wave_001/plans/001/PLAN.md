# Pocket Aviary V1 Implementation Plan

## 1. Intent and planning stance

This plan translates the PRD into an implementation-ready v1 for a small engineering team. The product goal is not feature breadth; it is to make a browser-based aviary feel continuously alive, specific, and private across weeks of use. Every technical decision below is in service of four constraints:

- the aviary must feel in motion before the user thinks about the app,
- presence must matter without becoming obligation,
- the server must own canonical bird identity and drift,
- accessibility and privacy must ship as first-class parts of the product rather than follow-on fixes.

Where the PRD is intentionally high-level, this plan makes explicit calls and names them so execution does not stall.

## 2. Scope

### In scope for v1

- Modern-browser web app only, with responsive desktop and mobile layouts.
- Email magic-link authentication and one aviary per account.
- Two starter birds at account creation, with age-based expansion up to a cap of seven birds.
- Canonical server-side simulation tick that advances mood, presence-derived expressivity, weather, notebook generation, and personality drift.
- Core interactions: passive watching, listen-in, offer, settle, notebook viewing.
- Persistent field notebook with sparse auto-generated entries.
- Multi-device sync through authoritative server snapshots and append-only interaction events.
- Quiet, opt-in visit invitations with read-only visitor sessions.
- Accessibility surfaces: screen-reader narration, reduced-motion mode, keyboard navigation, call captions, contrast-compliant chrome.
- Operational telemetry, performance instrumentation, account export, account deletion, session revocation.

### Explicitly out of scope

- Native iOS or Android clients.
- Gamification of any kind: streaks, achievements, levels, counters, badges, green-dot calendars.
- Tamagotchi mechanics: hunger, distress, decaying happiness, death, punishment for absence.
- Social-network surfaces: profiles, feeds, discovery, comments, chat, co-presence, leaderboards.
- User-facing numerical personality stats or developer-facing debug panels exposed in production.
- Recorded audio fallback paths or heavy live-sync infrastructure such as co-authoring or shared simulation.

## 3. Decision calls and assumptions

The PRD leaves several implementation choices open. This plan makes the following calls:

- **Server-owned “expressive energy” layer:** personality traits only drift upward, but birds must still become quieter after long absence. To reconcile that, v1 stores a separate short-horizon `expressive_energy` signal per bird that decays with inactivity and shapes greeting frequency, call density, and approach behavior without lowering permanent personality traits.
- **Rule-based notebook and narration generation:** v1 uses a deterministic prose generator with curated templates, grammar rules, and event prioritization rather than a freeform LLM path. This keeps the voice specific, sparse, safe, and consistent.
- **Polling over WebSockets:** canonical state reaches clients through bootstrap plus low-frequency snapshot polling keyed by version/ETag. This is simpler, cheaper, and consistent with the PRD’s “clients pull snapshots and interpolate” rule.
- **Single deployable backend plus workers:** v1 uses one TypeScript service boundary with separate API and worker processes rather than many microservices. This keeps the architecture understandable while preserving clean module boundaries.
- **Visitor surface is render-only:** visitors get the same ambient scene and audio state the host would see, plus local accessibility controls. They do not get host settings, notebook access, listen-in, offers, or settle controls.
- **Remote-config calibration:** presence window length, drift gains, notebook sparsity thresholds, and age-based bird unlock timing are all remotely configurable without code deploys.

## 4. System architecture

### High-level topology

V1 should be built as four runtime components sharing one canonical relational database:

1. **Web application**
   - Edge-served HTML shell with embedded bootstrap snapshot.
   - React/TypeScript client for scene rendering, input handling, accessibility surfaces, and WebAudio synthesis.
   - Responsive scene layer, top-bar UI, account/settings pages, and visitor pages.

2. **Application API**
   - Magic-link auth issuance and consumption.
   - Snapshot reads, event ingestion, notebook reads, account settings, export/delete flows, visit invite CRUD, visit log reads.
   - Local validation of offers, cooldowns, invite tokens, session revocation, and access control.

3. **Simulation worker**
   - Owns the per-aviary tick.
   - Reads newly ingested events, updates mood and expressive energy, applies monotonic drift deltas, schedules weather and calls, emits notebook entries, and writes canonical snapshots.

4. **Background jobs**
   - Email sending for magic links, invite links, export delivery.
   - Soft-delete sweeper and hard-delete erasure after 30 days.
   - Performance synthetic-check reporter and operational alerting jobs.

### Recommended stack

- **Frontend:** React + TypeScript with SSR-capable framework support for fast initial HTML delivery.
- **API/worker runtime:** Node.js + TypeScript.
- **Primary database:** PostgreSQL.
- **Queueing:** Postgres-backed durable job queue for v1; no separate Kafka dependency required.
- **Caching:** short-lived in-memory or Redis cache for magic-link rate limiting, ETag snapshot cache, and session lookups if needed.
- **Email:** transactional provider with tokenized magic links and invite links.
- **Assets:** small SVG/compact bitmap species art plus procedural audio motif definitions.

This is intentionally conservative. The product risk is in simulation feel, not infrastructure novelty.

### Client/server split

The server owns:

- account identity and sessions,
- stable bird identity and all persistent bird state,
- personality vectors,
- mood state,
- expressive energy,
- weather state,
- notebook entry generation,
- visit entitlement,
- canonical snapshot versions.

The client owns:

- rendering interpolation between canonical snapshots,
- local ornaments such as ambient leaves/feathers,
- WebAudio synthesis from server-supplied motif/call seeds,
- top-bar fade and other presentation-only state,
- local accessibility preferences not yet synced,
- event capture and idempotent submission.

### Render boundary

The client must never invent canonical bird behavior. The server snapshot should provide:

- current birds in aviary order,
- perch zone, pose family, pose phase seed, and motion state,
- mood and expressive-energy-derived behavior flags,
- active weather state and time-of-day phase,
- upcoming call schedule seeds for a short horizon,
- notebook summary metadata,
- snapshot version and server time.

The client then interpolates, animates, and synthesizes from those instructions. This preserves coherence across devices and in visitor mode.

## 5. Data model

### Core entities

#### `accounts`

- `account_id` UUID, synthetic and globally unique.
- `email_ciphertext` encrypted canonical email.
- `email_lookup_hash` deterministic hash for login lookup and rate limiting.
- `timezone`, `locale`.
- `created_at`, `deletion_requested_at`, `deleted_at`.
- `settings_json` for visit notifications, captions, reduced-motion override, muted-audio preference.

No service outside the auth/account boundary should use raw email as an identifier.

#### `device_sessions`

- `session_id` UUID.
- `account_id`.
- `created_at`, `last_seen_at`, `revoked_at`.
- `device_label` inferred from user agent for settings display.
- `last_ip_region` coarse only, optional.

#### `aviaries`

- `aviary_id` UUID.
- `account_id` unique.
- `state_version` bigint.
- `created_at`.
- `last_tick_at`.
- `last_presence_end_at`.
- `current_day_phase`.
- `current_weather_state`.
- `settled_until` nullable.
- `next_bird_unlock_at`.

#### `birds`

- `bird_id` UUID, stable forever.
- `aviary_id`.
- `species_id`.
- `display_name`.
- `adopted_at`.
- `sort_index`.
- `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity` as floats in `[0,1]`.
- `mood_state`.
- `mood_entered_at`.
- `expressive_energy` float in `[0,1]`.
- `current_perch_zone` enum `front|middle|back`.
- `pose_family` enum such as `preen|scan|tilt|rest|call`.
- `pose_seed`.
- `call_signature_seed`.
- `offer_cooldown_until`.
- `last_greeted_at`.

#### `interaction_events`

- `event_id` UUID.
- `idempotency_key` string unique per session.
- `account_id`, `aviary_id`, `session_id`.
- `bird_id` nullable.
- `event_type` enum:
  - `presence_ping`
  - `listen_in_started`
  - `listen_in_ended`
  - `offer_seed`
  - `offer_song`
  - `offer_water`
  - `settle_started`
  - `settle_undone`
  - `visibility_resumed`
- `client_occurred_at`.
- `server_received_at`.
- `payload_json` for focus duration, audio enabled, visibility metadata, and UI context.
- `processed_at` nullable.

#### `presence_windows`

- Derived from valid `presence_ping` events.
- `window_id` UUID.
- `account_id`, `aviary_id`, `session_id`.
- `started_at`, `ended_at`.
- `qualifying_seconds`.

This table gives the simulation worker clean, auditable presence inputs without depending on raw DOM events.

#### `aviary_snapshots`

- `aviary_id`, `state_version`.
- `server_time`.
- `scene_state_json` containing time phase, weather, settled state, bird render state, call schedule seeds, and animation horizon.
- `etag`.

This is the optimized read model served to host and visitor clients.

#### `notebook_entries`

- `entry_id` UUID.
- `aviary_id`.
- `observed_at`.
- `entry_type` enum such as `greeting_shift`, `weather_quiet`, `long_quiet`, `offer_reaction`, `first_time_pattern`.
- `body_text`.
- `dedupe_key`.

#### `visit_invites`

- `invite_id` UUID.
- `host_account_id`, `host_aviary_id`.
- `visitor_email_ciphertext`, `visitor_email_hash`.
- `token_hash`.
- `status` enum `pending|accepted|revoked|expired`.
- `created_at`, `expires_at`, `revoked_at`, `accepted_at`.

#### `visit_sessions`

- `visit_session_id` UUID.
- `invite_id`.
- `started_at`, `ended_at`.
- `approx_duration_seconds`.

These records exist for host transparency only and must not flow into cross-account analytics.

#### `account_export_jobs`

- `job_id` UUID.
- `account_id`.
- `status`.
- `requested_at`, `completed_at`.
- `download_token_hash`, `download_expires_at`.

## 6. API surface

### Authentication and account

- `POST /api/auth/magic-link/request`
  - Input: email.
  - Output: generic success response.
  - Behavior: issue one-time token, enforce per-email rate limit, send email.

- `GET /auth/magic-link/consume?token=...`
  - Validates one-time token, creates device session, invalidates token, redirects into the aviary app.

- `GET /api/account/sessions`
  - Lists current device sessions.

- `POST /api/account/sessions/:sessionId/revoke`
  - Revokes one device session.

- `POST /api/account/export`
  - Creates export job and emails download link.

- `POST /api/account/delete`
  - Starts soft-delete window.

- `POST /api/account/delete/cancel`
  - Restores account inside the 30-day window.

### Aviary state

- `GET /api/aviary/bootstrap`
  - Returns the initial snapshot plus account-scoped settings needed for first render.
  - Used for initial page load and embedded into SSR HTML when authenticated.

- `GET /api/aviary/snapshot?sinceVersion=<n>`
  - Returns `304` if unchanged via ETag/version.
  - Otherwise returns authoritative snapshot, server time, state version, and any system-level messages.

- `POST /api/aviary/events/batch`
  - Accepts a small ordered array of interaction events with idempotency keys.
  - Validates access, cooldowns, and bird IDs.
  - Returns acknowledgement by event, plus current authoritative cooldown/timing hints.

- `GET /api/aviary/notebook?cursor=<opaque>`
  - Returns notebook entries in reverse chronological order, paged.

### Social visits

- `POST /api/visits/invites`
  - Input: visitor email.
  - Creates invite, sends one-time visit link, defaults to 30-day expiry.

- `GET /visit/<token>/bootstrap`
  - Validates token and returns read-only snapshot plus host display metadata.

- `GET /api/visits/log`
  - Host-only read of invitation states and past visits.

- `POST /api/visits/invites/:inviteId/revoke`
  - Revokes invite immediately.

### Response design principles

- Every mutating API uses opaque IDs and never trusts client-computed state.
- Event ingestion is idempotent and tolerant of retries.
- Snapshot APIs are cacheable per version/ETag but never shared cross-user.
- System errors use matter-of-fact copy and map to explicit UI states.

## 7. Presence model

Presence is the most contamination-sensitive logic in the product and should be isolated as a dedicated module with exhaustive tests.

### Client-side qualification

The client only emits `presence_ping` when all three are true:

- document is visible,
- window has focus,
- at least one pointer or key activity occurred within the configured trailing activity window.

V1 should start with a **180-second** activity window, held in remote config, and tune after observing calibration data.

### Ping cadence

- Emit a `presence_ping` every 60 seconds while the qualification rule holds.
- Emit an immediate `visibility_resumed` event when the tab becomes visible again after hidden state or a long frame gap.
- Stop emitting when any qualifying condition fails.

### Server-side coalescing

The API writes raw presence events; a presence coalescer normalizes them into contiguous windows with:

- max allowed gap of 75 seconds between pings inside one window,
- truncation at visibility/focus loss,
- rejection of future-skewed client timestamps beyond a small tolerance.

### Why this design

- It satisfies the PRD’s exact conjunction rule.
- It avoids storing noisy DOM-level movement data.
- It gives the tick a compact input for drift calibration.
- It makes “user is quietly watching” count without requiring frequent motion.

## 8. Simulation engine design

### Tick cadence and ownership

- Global scheduler enqueues due aviaries every minute.
- Worker acquires a per-aviary advisory lock before processing.
- Only one worker may mutate an aviary at a time.
- Tick writes are transactional: either the new state version commits in full or nothing does.

### Processing loop per aviary

1. Load current aviary row, birds, last processed event cursor, and recent presence windows.
2. Load unprocessed interaction events in server-received order.
3. Compute elapsed time since last tick and current local-time phase.
4. Update expressive energy from recent presence recency and interaction recency.
5. Apply mood transitions.
6. Apply monotonic trait drift deltas.
7. Select per-bird perch zone, pose family, and call behavior horizon.
8. Apply weather scheduling and any settle-state effects.
9. Generate notebook entries if noteworthy conditions fire and sparsity rules allow.
10. Persist new bird state, aviary state, snapshot projection, processed cursor, and incremented `state_version`.

### Expressive energy

This is the key implementation bridge between “no punishment” and “quiet after absence.”

- `expressive_energy` is a short-horizon score derived from recent qualified presence, recent listen-in, and recent successful offers.
- It decays toward a low ambient baseline over days of inactivity.
- It does **not** directly change personality values.
- It controls:
  - greeting probability,
  - call density,
  - likelihood of front-perch approach,
  - likelihood that a bird reacts promptly to an offer.

Suggested model:

- trailing 14-day exponentially weighted average of presence minutes,
- small boosts for recent listen-in and accepted offers,
- floor value above zero so birds never become inert.

### Personality drift

Trait drift must be measurable after about one week of regular visits and perceptible after about three weeks, without becoming gameable inside a single session.

Recommended approach:

- Represent each personality trait in `[0,1]`.
- Compute daily capped additive deltas from a low-pass filtered signal.
- Apply only positive deltas.
- Cap total trait movement per week to keep visible change gradual.

Trait influence plan:

- **Boldness**
  - Driven primarily by presence near the front of the scene and by offers made while the bird is already near the viewer.
- **Social warmth**
  - Driven by repeated return greetings and listen-in engagement.
- **Vocal frequency**
  - Driven by sustained presence and listen-in; increases how often the bird schedules calls and joins chorus moments.
- **Plumage saturation**
  - Driven almost entirely by sustained presence over longer windows.
- **Curiosity**
  - Driven by offers approached or accepted, not merely offered.

Recommended weight split for the aggregate drift signal:

- presence windows: 70%
- listen-in duration: 20%
- offer response outcomes: 10%

Settle should influence mood only, not long-term drift.

### Mood system

Use a small explicit state machine with these v1 states:

- `wary`
- `content`
- `curious`
- `drowsy`
- `alert`

Transition inputs:

- local time of day,
- current weather,
- recent successful/ignored offers,
- nearby bird signals from the same tick,
- bird personality traits,
- settle state.

Guidelines:

- mood persists across sessions,
- early morning biases toward `alert`,
- dusk/night biases toward `drowsy`,
- rain temporarily dampens vocal behavior and nudges toward `content` or `wary` depending on personality,
- nearby alarm-like calls can spread `wary`,
- high boldness dampens entry into `wary`.

### Return-greeting generation

The greeting path needs its own logic rather than being a generic animation:

- Compute absence bucket from `last_presence_end_at`:
  - short: `<2h`
  - medium: `2h–48h`
  - long: `>48h`
- Score each bird for “greeter likelihood” using boldness, social warmth, expressive energy, mood, and a same-bird cooldown.
- Select one primary greeter.
- Generate an action tuple such as:
  - glance + quiet two-note call
  - head tilt + step forward
  - longer call + delayed response from second bird
- Allow optional secondary reactions staggered by 300–1200ms.

This path should be deterministic from snapshot seed + absence bucket so that host and visitor render the same underlying state if they load simultaneously.

### Call runtime

The simulation worker does not synthesize audio; it schedules call intent.

Per bird per snapshot horizon it should emit:

- call motif family,
- timing seed,
- pitch range seed,
- intensity/mood modifiers,
- chorus-join eligibility.

The client audio runtime then turns those into actual sound and captions.

### Weather scheduling

Per aviary, use a seeded RNG stream so weather feels specific but is fully server-authored.

- rare rain target: 2 to 4 short events per week,
- soft wind target: several light events per week,
- no severe weather in v1.

### Notebook generation

Use a rule engine with curated templates and per-entry dedupe keys.

Candidate triggers:

- unusual greeter ordering,
- first greeting after long absence,
- unusually quiet morning,
- long preen/rest period,
- weather softening the aviary,
- repeated offer interest from a specific bird.

Sparsity rules:

- hard cap of 3 routine entries per rolling 7 days,
- major notable events may bypass that cap with stricter dedupe,
- never emit one entry per session.

Generation requirements:

- lowercase,
- present tense,
- bird-specific,
- no system jargon,
- no mention of hidden numeric state,
- no observation of user visit streaks or attendance patterns.

## 9. Sync, consistency, and conflict handling

### Canonical model

- The server is the only writer of personality, mood, expressive energy, and notebook state.
- Clients write interaction events only.
- Snapshot versions are monotonically increasing integers.

### Event ordering and idempotency

- Every event carries an idempotency key unique within a session.
- The API stores events as received and deduplicates by `(session_id, idempotency_key)`.
- The worker processes by `server_received_at` and stable insertion order, not by raw client time alone.

### Multi-device behavior

- A newly visible client immediately fetches a fresh snapshot.
- Visible clients poll every 30 seconds when active.
- Clients poll after long frame gaps or device resume.
- Stale clients never push corrected canonical state; they only send events.

### Preventing last-write-wins failures

- No client endpoint accepts absolute personality values.
- The tick takes an advisory lock per aviary.
- `aviaries.state_version` is checked on snapshot write.
- Event ingestion and snapshot reads are separate concerns; events remain valid even if sent from an older snapshot, subject to current server validation.

### Offer cooldown correctness

Cooldowns are validated server-side using `offer_cooldown_until`.

- If an offer arrives too early, the API accepts the request envelope but marks the event rejected with reason `cooldown_active`.
- The response returns current cooldown expiry from canonical state so the client can remain truthful.

### Failure surfaces

System failures must be explicit and matter-of-fact:

- expired magic link,
- session timed out,
- snapshot load failed,
- visit link expired or revoked,
- unsupported browser.

No naturalist copy on these surfaces.

## 10. Frontend rendering pipeline

### Shell and first paint

To hit the “already alive” feel:

- Serve an authenticated HTML shell with the latest bootstrap snapshot embedded inline whenever possible.
- Render a quiet field fallback only when no snapshot is available in time.
- Never show a spinner.
- Hydrate the scene progressively so the first bird can render before non-critical chrome loads.

### Scene composition

Recommended layer order:

1. sky and background foliage
2. middle-plane perches
3. birds
4. optional weather overlays
5. foreground branch/leaf drift
6. top bar

Birds should be positioned in three logical perch zones with scale/parallax differences keyed to front/middle/back depth.

### Bird rendering approach

Use lightweight species art assets plus pose-state transitions:

- each species has a silhouette, palette, and compact pose set,
- idle micro-motion is expressed through transform/pose interpolation, not frame-heavy sprite sheets,
- pose selection is driven by snapshot state,
- motion remains continuous while the tab is visible.

### Reduced-motion rendering

When reduced motion is enabled:

- replace micro-animation with slow cross-fades between still poses,
- replace flight paths with perch-to-perch cross-fades,
- remove leaf/feather drift,
- retain slow day/evening palette shifts,
- keep all mood and drift logic intact.

### Top bar behavior

Top bar contains only:

- account/settings,
- accessibility settings,
- notebook,
- offer,
- settle.

Behavior:

- fades toward transparency after a few seconds of inactivity,
- returns on pointer or keyboard activity,
- remains keyboard reachable and visually focusable in all light states.

### Interaction UI

- Clicking/tapping/focusing a bird toggles listen-in.
- Offer is opened from top bar and targets a chosen bird inside the panel flow rather than direct bird click.
- Settle is top-bar triggered, visually reversible for 5 seconds, and then treated as mood-quieting plus presence end.

### Visitor page

The visitor render path reuses the same scene renderer but mounts a read-only shell:

- no offer UI,
- no settle UI,
- no listen-in affordance,
- local-only accessibility controls allowed,
- snapshot source is invite-scoped and never writes events that affect the host.

### Responsive layout

- Desktop widens inter-perch spacing.
- Mobile compresses spacing while preserving all birds in frame.
- Never crop birds or let them leave the viewport.
- Keep tap targets and focus order viable on small screens.

## 11. Audio pipeline

### Architecture

Build the audio system around WebAudio with one master graph per page:

- one bird bus per bird,
- one ambient/chorus bus,
- optional ducking/send control for listen-in,
- one master bus with mute and gain controls.

### Procedural call generation

Each species owns a motif library defined as:

- envelope shapes,
- oscillator/noise blend parameters,
- rhythm patterns,
- pitch movement templates,
- ornament options.

Each bird’s call identity is then derived from:

- species motif family,
- stable `call_signature_seed`,
- current mood modifiers,
- vocal-frequency trait,
- current expressive energy.

### Scheduling

- Schedule 3 to 5 seconds ahead in the audio clock.
- Refresh future call schedule whenever a new authoritative snapshot arrives.
- Blend schedule updates rather than hard-resetting active calls.

### Listen-in mix

Listen-in should rebalance, not solo:

- raise focused bird gain gradually,
- lower other bird buses gradually,
- never mute other birds completely,
- restore ambient balance on exit with matching fade durations.

### Captions

Caption strings must be generated from the same call grammar tokens that produced the audio event. This ensures captions describe what actually played.

### Fallback

If WebAudio is unavailable or fails to initialize:

- run the aviary silently,
- automatically enable captions,
- preserve all visual and notebook behavior,
- surface audio errors only through aggregate operational telemetry.

## 12. Accessibility surfaces

### Screen-reader narration

Provide a dedicated narration channel sourced from canonical snapshot state, not ad hoc ARIA labels.

Implementation approach:

- hidden live region with queued prose updates,
- idle narration cadence every 30 to 60 seconds,
- prioritized narration on greeting, offer reaction, and settle,
- dedupe to avoid repeating near-identical observations.

Narration content must:

- stay in naturalist voice,
- reference concrete birds and positions,
- avoid numeric state labels,
- avoid chatty announcement phrasing.

### Keyboard navigation

- Tab cycles top-bar items.
- Entering the scene focuses the first bird.
- Arrow keys move between birds.
- Enter toggles listen-in.
- Escape exits listen-in or dismisses overlays.
- Offer panel and settings must be fully keyboard navigable.

### Visual accessibility

- WCAG AA contrast for all text and focus treatments.
- Focus ring must remain visible against bright morning and dim evening palettes.
- Captions should avoid covering the bird body and should remain readable over the scene.

### Audio accessibility

- Captions optional but easy to enable.
- Silence fallback defaults captions on.
- Muted audio remains a first-class mode, not an error condition.

### Motion accessibility

- Respect `prefers-reduced-motion` on first load.
- Allow an explicit override in accessibility settings.
- Reduced-motion mode should persist per account setting once chosen.

## 13. Privacy, security, and account integrity

### Privacy boundaries

- Per-bird interaction history is used only to drive that account’s simulation.
- No analytics pipeline may ingest bird trait values, per-bird notebook text, or user interaction event detail beyond operational necessity.
- Aggregate telemetry is limited to counts, latencies, error classes, duration histograms, and performance timings with no per-account simulation detail.

### Identifier policy

- All internal joins use synthetic `account_id`.
- Raw email stays confined to the account/auth tables.
- Invitee emails get the same encrypted-plus-hash treatment.

### Auth and session security

- Magic links expire after 15 minutes.
- Used links are invalidated immediately.
- Per-email request limits and IP-based abuse controls apply.
- Session revocation is immediate for future requests.

### Deletion and export

- Soft delete immediately hides the aviary from normal use.
- Hard delete erases all account-bound records after 30 days.
- Export packages include current bird state, notebook entries, settings, and visit metadata that belongs to the user.

## 14. Performance budgets and observability

### Hard budgets

- Initial JS bundle under 2MB gzipped.
- First bird visible under 500ms on mid-tier mobile over 4G.
- Idle motion at 60fps on a five-year-old mid-range laptop.
- No measurable client memory growth over a 30-minute session.
- Simulation tick latency p99 under 5 seconds.

### Performance tactics

- Inline only the minimum bootstrap state required for first render.
- Code-split settings, account pages, and visit-management flows.
- Keep species art compact and reuse pose assets.
- Reuse audio nodes/buffers to avoid per-call allocation churn.
- Suspend rendering when hidden; do not suspend server simulation.

### Observability

Collect aggregate-only telemetry for:

- request counts and error rates,
- snapshot fetch timings,
- first-bird render timings,
- long frame counts and frame-time distributions,
- audio initialization failures,
- worker tick durations,
- visit invite email send failures.

Do not collect:

- per-bird trait values in analytics,
- notebook text in analytics,
- account-comparable engagement metrics,
- population-level “most visited aviary” style metrics.

### Synthetic monitoring

Run scheduled browser probes from representative geographies to validate:

- first-bird SLA,
- audio graph initialization,
- snapshot freshness,
- degraded-state copy for failures.

## 15. Delivery workstreams

### Workstream A: platform and auth

- account schema, synthetic ID discipline, magic links, sessions, settings shell, delete/export primitives.

### Workstream B: canonical state and simulation

- bird schema, event ingest, presence windows, tick worker, mood/drift modules, snapshot projection, notebook generator.

### Workstream C: scene renderer and interaction UX

- responsive scene, top bar, bird focus/listen-in, offer UI, settle flow, notebook UI, quiet field loading state.

### Workstream D: audio and accessibility

- species motif library, WebAudio engine, captions, narration system, reduced-motion path, keyboard semantics.

### Workstream E: social visits and operational tooling

- invite lifecycle, visitor page, visit logs, rate limiting, synthetic monitoring, telemetry dashboards, privacy review.

Each workstream should ship behind feature flags and land into an integration environment early, because the core risks are cross-cutting.

## 16. Testing strategy

### Deterministic simulation tests

- fixed-seed fixtures for drift over 1 day, 1 week, and 3 weeks,
- absence scenarios proving no negative trait drift,
- expressive-energy decay scenarios proving birds become quieter without personality loss,
- mood transition snapshots for time-of-day and weather combinations,
- notebook sparsity tests.

### API and contract tests

- idempotent event ingestion,
- cooldown enforcement,
- invite revoke/expire behavior,
- session revocation,
- deletion/export flows.

### End-to-end tests

- first account creation and two-bird adoption,
- return after short/medium/long absence buckets,
- multi-device open with one device resuming from stale state,
- visitor opening revoked link,
- reduced-motion and keyboard-only flows.

### Accessibility QA

- screen-reader walkthroughs on Safari/VoiceOver and at least one Windows screen reader,
- caption readability across palettes,
- focus-order audits,
- contrast snapshots for morning/evening/night.

### Performance QA

- CI bundle gate,
- first-bird synthetic benchmark,
- 30-minute soak for memory growth,
- 60fps idle scene benchmark on target hardware.

## 17. Rollout plan

### Phase 0: internal calibration

- Enable auth, two birds, renderer, basic tick, and notebook in an internal environment.
- Tune presence window, expressive-energy decay, and first-bird render path using synthetic accounts.
- Do not expose social visits yet.

### Phase 1: closed alpha

- Small set of internal/external friendly users.
- Focus on drift calibration, greeting quality, notebook sparsity, audio recognizability, and accessibility breakpoints.
- Keep bird count fixed at two for all alpha accounts.

### Phase 2: private beta

- Enable age-based unlock scheduler behind remote config.
- Initial unlock proposal:
  - bird 3 at 8 weeks
  - bird 4 at 16 weeks
  - bird 5 at 28 weeks
  - bird 6 at 40 weeks
  - bird 7 at 52 weeks
- Enable quiet visit invites for a subset of accounts.

### Phase 3: public v1

- Launch with two starter birds for all users and the unlock schedule active by cohort.
- Ship visits off by default.
- Maintain strong rollback levers for:
  - invite issuance,
  - new bird unlocks,
  - notebook generation,
  - audio engine experiments,
  - presence-window calibration.

### Day-one instrumentation

- first-bird visible timing,
- snapshot error rate,
- tick p95/p99,
- audio init error rate,
- offer rejection rate due to cooldown,
- screen-reader narration queue errors,
- invite-email delivery failures.

These are sufficient to keep the product healthy without measuring the user relationship as a KPI artifact.

## 18. Major risks and mitigations

### Drift calibrates too fast or too slow

- Mitigation: fixed-seed simulations, remote-config gains, weekly calibration review during beta, explicit “visible at ~3 weeks” acceptance criteria.

### No-negative-drift rule produces birds that feel static after absence

- Mitigation: separate expressive-energy layer, explicit tests for “ambient but alive” behavior, design review on long-absence returns.

### Audio sounds synthetic in a bad way or birds lose recognizability

- Mitigation: tiny species pool, per-bird signature seeds, audio listening reviews on real devices, captions sourced from the same grammar, no recorded fallback compromise.

### Snapshot/polling model causes visible jumps

- Mitigation: server supplies short behavior horizon, client interpolates between perch/pose states, refresh on visibility resume, aggressive stale-state E2E coverage.

### Multi-device event ordering corrupts state

- Mitigation: append-only event store, advisory locks, server-received ordering, no client-authored absolute values, snapshot version invariants.

### Accessibility surfaces become second-class

- Mitigation: reduced-motion and narration ship inside the core renderer milestone, dedicated accessibility QA gates, failure to meet these blocks public launch.

### Performance budgets are missed by the initial web stack

- Mitigation: bundle budget gate from week one, code-splitting plan before feature growth, asset budget tracking per species, synthetic first-bird checks in CI.

### Privacy leaks through logging or analytics convenience

- Mitigation: schema-level identifier discipline, analytics allowlist review, account-ID-only operational logging, no notebook/body text in telemetry, privacy review before beta.

## 19. Execution summary

The fastest credible v1 is a server-authoritative web product with a small, carefully tested simulation core and a lightweight renderer/audio client that prioritizes first-bird presence over UI richness. The product will succeed or fail on whether the birds feel continuous, specific, and gentle to return to. The plan above is intentionally biased toward preserving that feeling even when it costs extra implementation discipline.
