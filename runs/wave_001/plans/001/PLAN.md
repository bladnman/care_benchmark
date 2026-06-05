# Pocket Aviary implementation plan

## 1. Product stance and v1 scope

Pocket Aviary v1 is a browser-only, single-account, single-aviary product centered on two starter birds that can slowly grow to a maximum of seven birds over the lifetime of the aviary. The core experience is a single horizontal scene that feels like it was already alive before the user arrived: birds are mid-motion on first render, one bird notices the returning user without any textual welcome, calls vary procedurally, and small user interactions accumulate into slow personality drift over weeks.

V1 includes:

- Email magic-link accounts with per-device sessions, session revocation, verified email changes, export, and deletion.
- One canonical aviary per account, server-authored simulation ticks, and multi-device sync through server snapshots.
- Two starter birds selected by the system from a small coherent species pool, user-assigned names, stable bird identities, rename support, and age-based future bird arrivals up to seven.
- Hidden personality vectors, persistent mood, server-side drift, bird-to-bird interactions, procedural call grammars, return greetings, listen-in, offers, settle, field notebook, presence accounting, day/night cycle, ambient weather, ambient micro-motion, and top-bar controls.
- Optional read-only visit invitations with per-invite links, revocation, expiration, visit log, and notifications off by default.
- First-class accessibility: screen-reader narration, call captions, keyboard navigation, visible focus, reduced-motion rendering, WebAudio graceful-silence fallback with captions, WCAG AA user-copy contrast.
- Aggregate operational telemetry only: load timings, first-bird timings, frame timings, audio errors, request counts, tick latency, and anonymized session-duration histograms.

V1 explicitly excludes:

- Native iOS or Android apps.
- Any gamification: achievements, badges, streaks, scores, levels, XP, counters, visit calendars, leaderboards, ranks, public comparisons, or "birds adopted" progress surfaces.
- Tamagotchi mechanics: no death, hunger, distress, decaying happiness, punishment for absence, feeding schedules, or obligation loops.
- Social network features: profiles, follows, public discovery, comments, chat, shared co-presence, visitor avatars, mutual visits, public feeds, rankings, or show-off rendering.
- User control over scene layout: no panning, zooming, scrolling, placing birds, assigning perches, or arranging the aviary.
- Exposure of personality vector values in user-facing UI, exports excepted only as raw account-data export, debug UI, telemetry, accessibility labels, captions, or notebook prose.
- Recorded audio loops or recorded-audio fallback.

Every implementation decision should be checked against four invariants:

1. The server is the only writer of canonical aviary state and personality vectors.
2. Presence is measured as attention, not mere open-tab time.
3. The user is noticed by birds, never announced to by UI chrome.
4. Interaction data exists to run that user's aviary, not to become aggregate product data.

## 2. High-level architecture

### 2.1 Service shape

Use a small service-oriented architecture with hard data-boundary enforcement:

- `web-client`: browser app for rendering, audio, interaction capture, account surfaces, notebook, visit viewing, and accessibility UI.
- `api-service`: authenticated HTTP/SSE or WebSocket facade for snapshots, interaction ingestion, account operations, visits, export, and settings.
- `simulation-service`: server-owned tick runner that consumes account-scoped event logs, updates moods and personality vectors, writes canonical snapshots, and emits notebook candidates.
- `auth-service` or auth module: magic-link issuance, validation, session-token management, email verification, rate limits, and device revocation.
- `notification-mailer`: transactional email only for magic links, email verification, export download links, and optional visit notifications when explicitly enabled.
- `export-worker`: on-demand account export generation and short-lived download link issuance.
- `deletion-worker`: 30-day soft-delete recovery window and hard deletion across all account-scoped stores.
- `observability-pipeline`: aggregate-only operational metrics; it must not read simulation records or interaction payload fields beyond coarse operational dimensions.

For v1, these may deploy as a modular monolith plus worker processes if that reduces operational complexity, but the codebase should keep module boundaries as if services were split. In particular, analytics code must not import simulation storage accessors, and simulation code must not depend on client rendering types.

### 2.2 Data stores

Use separate logical stores even if backed by the same database cluster:

- Relational primary database for accounts, encrypted email, sessions, aviary records, birds, personality vectors, moods, settings, visits, notebook entries, and deletion state.
- Append-only account-scoped interaction event log for presence pings, listen-in events, offers, settle, naming, and visit administrative events.
- Snapshot cache for current aviary render state, keyed by account UUID and snapshot version.
- Short-lived token store for magic links, email-change links, visit links, and export links.
- Aggregate telemetry backend with no account UUID, bird ID, interaction payload, personality vector, notebook content, or email fields.

The account UUID is the only identifier used outside the encrypted account record. Email must not appear in logs, partition keys, metrics labels, trace attributes, worker queue names, or simulation records.

### 2.3 Client/server split

The server owns:

- Canonical bird identity, species, names, personality vectors, current mood, mood timers, long-lived drift history, notebook entries, visit authorization, account state, and snapshot versions.
- Simulation ticks, event-log ordering, additive personality deltas, mood transitions, weather scheduling, age-based bird availability, and return-greeting input signals.
- Naturalist prose generation for durable notebook entries and optionally screen-reader narration templates if server-side generation gives better consistency.

The client owns:

- Rendering a received snapshot smoothly without mutating canonical state.
- Local interpolation between snapshots, idle visual ornaments that do not affect simulation, WebAudio synthesis from call grammar descriptors, call captions, top-bar fade behavior, keyboard/focus interactions, and presence-signal detection.
- Submitting interaction events and presence pings to the server with idempotency keys.
- Graceful local handling of temporary network gaps using the last valid snapshot while making clear only through matter-of-fact error surfaces when canonical state cannot be refreshed.

Clients never tick the simulation and never submit absolute values for mood, personality, perch choice, weather, or notebook content.

### 2.4 Render pipeline boundary

Snapshots should be render-ready but not frame-by-frame animation scripts. A snapshot contains enough state for the client to draw a first frame immediately and interpolate for a short interval:

- Snapshot metadata: `snapshot_id`, `aviary_id`, `server_time`, `snapshot_version`, `valid_until`, `local_time_context`.
- Scene state: day/night phase, palette key, settled flag, active weather, ambient soundscape level.
- Per-bird render state: stable bird ID, display name, species, visual palette parameters, current mood, perch zone, x/y anchor, depth plane, pose family, pose phase, current or upcoming transition, call grammar seed, call schedule window, greeting eligibility flags.
- Interaction affordance state: offer cooldowns by bird and offer type, active listen-in target if any, keyboard focus order hints.
- Notebook summary state: unread count is not used as a badge; instead expose notebook availability and entries for the notebook panel only.
- Accessibility state: narration/caption descriptors derived from the same state without exposing internal numerical traits.

The client may generate ambient leaves, feathers, subtle parallax offsets, and reduced-motion cross-fade timing locally. These are ornaments, not canonical state, and must never be used as simulation inputs.

## 3. Domain model

### 3.1 Accounts

Account fields:

- `account_id`: synthetic UUID, primary internal identifier.
- `email_encrypted`: encrypted verified email.
- `email_hash_for_lookup`: keyed hash for login lookup and rate limiting; not logged.
- `email_verified_at`.
- `created_at`, `updated_at`.
- `deletion_requested_at`, `hard_delete_after`, `deleted_at`.
- `settings_id`.

Account settings:

- Accessibility preferences: reduced motion override, captions on/off, screen-reader narration cadence preference where applicable, audio enabled, contrast mode if design system supports one.
- Privacy/account preferences: visit notifications opt-in, session management visibility.
- Locale/timezone: last-known IANA timezone from client, updated on sign-in and snapshot requests. The day/night cycle uses the user's local timezone.

Session fields:

- `session_id`, `account_id`, hashed session token, device label, created timestamp, last seen timestamp, revoked timestamp, user agent summary, approximate region if needed for account transparency. Do not include precise location.

Magic-link fields:

- `link_id`, `account_id` or pending email lookup, hashed token, purpose, issued timestamp, expires timestamp, consumed timestamp, request metadata for rate limiting.

### 3.2 Aviary

Aviary fields:

- `aviary_id`, `account_id`, `created_at`.
- `bird_count_cap` fixed at seven for v1.
- `settled_state`: current settled flag, set by the settle interaction until tab close or active re-engagement.
- `last_snapshot_id`, `last_tick_at`, `tick_cursor_event_id`.
- `weather_state`: current rare ambient weather event, start/end times, mood modifiers.
- `age_unlock_state`: schedule of future bird availability based only on aviary age.

There is one aviary per account in v1. The data model should enforce this with a uniqueness constraint, not just application logic.

### 3.3 Birds

Bird fields:

- `bird_id`: stable UUID, never replaced by rename or species updates.
- `aviary_id`.
- `species_id`: from v1 pool of approximately six species.
- `display_name`: user-assigned, renameable.
- `adopted_at`.
- `starter`: boolean for the first two birds.
- `active`: true unless a future hard-deletion flow removes the account; birds are not retired or swapped in v1.
- `visual_seed`, `call_seed`: stable random seeds for procedural visual variation and call grammar identity.

Species fields:

- `species_id`, display species description, silhouette family, default palette, visual motif parameters, call motif library, night-active flag for the nightjar-like species, accessibility description templates.

New-account adoption:

- The system selects two starter species from the pool using deterministic server-side randomness scoped to the aviary.
- The user names the birds during onboarding, with optional suggestions.
- The flow presents the birds as arrivals, not catalog selections.

Future bird arrivals:

- The server computes availability from aviary age, not visits, interactions, subscription, or scores.
- The user may name an arriving bird, but should not browse a catalog or optimize species choice.
- The cap of seven is enforced in service logic and database constraints.

### 3.4 Personality vector

Persist one vector record per bird:

- `boldness`.
- `social_warmth`.
- `vocal_frequency`.
- `plumage_saturation`.
- `curiosity`.
- `updated_at`.
- `version`.

Represent traits as normalized decimal values in a bounded range chosen by simulation calibration, for example `0.0` to `1.0`, but keep the exact scale internal. Store enough precision for slow drift without user-visible jumps. The user never sees these values in UI, narration, captions, notebook, account settings, or product analytics. The on-demand account export may include them as part of the user's raw data snapshot, clearly framed as exported data rather than a stats surface.

Only the simulation service can update vector fields. Enforce with:

- No public API route for vector writes.
- Storage methods for vector mutation only in the simulation module.
- Database permissions or application-level write guards separating API and simulation roles.
- Tests that client event handlers cannot mutate vectors.

### 3.5 Mood

Persist mood separately from personality:

- `bird_id`.
- `mood`: enum such as `wary`, `content`, `curious`, `drowsy`, `alert`, with final set defined during simulation implementation.
- `mood_started_at`.
- `mood_expires_after` or transition timer.
- `last_transition_reason`: internal enum for debugging; not user-facing.
- `intensity`: optional bounded internal scalar if needed for rendering; not exposed.

Mood persists across sessions and is advanced by server ticks. It must not reset to neutral on tab open.

Mood inputs:

- Recent interactions in event-log order.
- Local time phase.
- Weather modifiers.
- Bird-to-bird call and mood propagation.
- Personality vector values.
- Settled state.

### 3.6 Interaction events

Use an append-only event log with strict idempotency:

- Common fields: `event_id`, `account_id`, `aviary_id`, optional `bird_id`, `event_type`, `client_event_id`, `session_id`, `occurred_at_client`, `received_at_server`, `payload`, `processed_at_tick`, `schema_version`.
- `presence_ping`: visible, focused, recent activity true; duration window; client monotonic clock hints; no raw pointer/key data.
- `listen_in_started`, `listen_in_ended`: target bird, start/end, reason for disengage.
- `offer_made`: offer type `seed`, `song_fragment`, or `still_pool`; target context if any; cooldown basis.
- `settle_started`, `settle_undone`, `settle_committed`.
- `bird_renamed`.
- `notebook_opened` only if needed for UX state; do not use notebook reading as drift input.
- Visit administrative events: invite created/revoked/expired/used; visitor viewing sessions are stored in visit-log tables and must not enter simulation drift.

Do not store raw user movement, keystrokes, exact focus timelines, or per-frame attention traces. Presence pings should be coarse, purpose-limited signals.

### 3.7 Field notebook

Notebook entry fields:

- `entry_id`, `aviary_id`, `created_at`, `observed_at`.
- `text`: naturalist prose, lowercase, present-tense, specific.
- `source_event_refs`: internal references to state or event IDs used to generate the entry, not shown in UI.
- `entry_kind`: greeting pattern, quiet stretch, weather, bird-to-bird interaction, offer reaction, drift milestone, etc.
- `rarity_score` or suppression metadata to keep entries sparse.

Notebook entries are generated by the simulation service or a dedicated notebook generator attached to tick output. They are read-only, never editable, never deletable individually in v1, and scroll indefinitely.

Generation rules:

- Target roughly one entry every few days for a regularly visited aviary.
- Allow additional entries for genuinely noteworthy events, but enforce per-aviary minimum spacing and repetition suppression.
- Never write entries about the user's visit frequency, streaks, engagement, or account behavior.
- Never include numerical vector changes.
- Prefer bird names, species descriptions, local time context, relative ordering, mood-visible observations, and concrete details.

### 3.8 Visits

Visit invite fields:

- `invite_id`, `host_account_id`, `aviary_id`, `visitor_email_encrypted`, `visitor_email_hash_for_lookup`, hashed token, created timestamp, expires timestamp, revoked timestamp, first used timestamp, last used timestamp.

Visit session fields:

- `visit_session_id`, `invite_id`, approximate started/ended timestamps, approximate duration, active/revoked state.

Visit log:

- Host-visible account settings list with visitor email, date, approximate duration, outstanding invite status, and revocation action.
- No badge, toast, push, or email unless the host explicitly enables optional visit notifications.

Visitor constraints:

- Visitor receives read-only snapshots through a visit-scoped endpoint.
- Visitor cannot listen in, offer, settle, rename, open host account settings, alter notebook state, generate greetings, produce presence-time, or affect drift.
- If revoked or expired, next snapshot pull returns a matter-of-fact unavailable surface.

## 4. API surface

Use JSON over HTTPS for v1 with optional SSE/WebSocket later only if polling cannot hit performance or battery goals. All mutation routes require idempotency keys.

### 4.1 Auth and account APIs

- `POST /auth/magic-links`: request a magic link by email. Always return a neutral success response to avoid account enumeration. Rate-limit by keyed email hash and IP.
- `POST /auth/magic-links/consume`: consume token, invalidate on first use, issue per-device session token, create account if needed, and bootstrap aviary if new.
- `POST /auth/email-change`: request new email verification.
- `POST /auth/email-change/confirm`: verify and commit new email.
- `GET /account`: return account settings, session list summary, export/deletion status.
- `PATCH /account/settings`: update matter-of-fact settings only.
- `POST /account/sessions/{session_id}/revoke`: revoke one device session.
- `POST /account/export`: enqueue JSON export and email short-lived link to verified address.
- `POST /account/deletion`: mark for deletion and set hard-delete deadline.
- `POST /account/deletion/cancel`: restore during 30-day window.

System/account copy uses normal capitalization and matter-of-fact language.

### 4.2 Aviary snapshot APIs

- `GET /aviary/snapshot`: authenticated host snapshot. Returns current canonical render-ready state, snapshot version, short-lived validity, offer cooldowns, notebook summary, and accessibility descriptors.
- `GET /aviary/snapshot?since={snapshot_id}`: optional delta form if profiling shows payload costs; must preserve small-payload simplicity.
- `GET /aviary/notebook`: paginated notebook entries, newest-first or chronological with stable cursor. No feed mechanics, badges, reactions, or editing.
- `GET /aviary/birds`: management view for names and settings; no stats.
- `PATCH /aviary/birds/{bird_id}/name`: rename bird with idempotency key.

Snapshot pull triggers:

- Initial navigation.
- Visibility changes to visible.
- Long render-frame gap or wake from laptop suspension.
- Low-frequency visible-tab keepalive.
- Visitor revocation checking on visit sessions.

### 4.3 Interaction APIs

- `POST /aviary/events/presence`: coarse presence ping only when all three presence conditions hold. Include recent-activity window evidence as booleans/durations, not raw events.
- `POST /aviary/events/listen-in/start`.
- `POST /aviary/events/listen-in/end`.
- `POST /aviary/events/offer`: offer type and client context. Server validates cooldown.
- `POST /aviary/events/settle/start`.
- `POST /aviary/events/settle/undo`.
- `POST /aviary/events/settle/commit`.

Each route appends events and returns an accepted event ID plus any immediate affordance state. The simulation tick applies durable state changes. The client may run local transitional animation for responsiveness, but must reconcile against the next snapshot.

Presence event validation:

- Client sends pings only when `document.visibilityState === "visible"`, window focus is true, and pointer/key activity occurred inside the calibrated recent window.
- Server clamps event durations, rejects impossible overlaps, deduplicates by `client_event_id`, and ignores stale sessions.
- Presence pings end on settle, tab hidden, blur, session expiration, or absence of activity beyond the window.

### 4.4 Visit APIs

Host:

- `POST /visits/invites`: create one invite for a visitor email, send one-time link, default expiration 30 days.
- `GET /visits`: list outstanding invites and visit log.
- `POST /visits/invites/{invite_id}/revoke`: revoke immediately.
- `PATCH /visits/settings`: opt into or out of visit notifications.

Visitor:

- `POST /visits/consume`: consume or validate visit link, create visit-scoped session.
- `GET /visits/{visit_session_id}/snapshot`: read-only host aviary snapshot.

Visitor snapshot responses must omit host account settings and any fields not needed to render the ambient aviary. Revoked, expired, or deleted-host states return matter-of-fact copy.

## 5. Simulation engine design

### 5.1 Tick loop

Run one server-side tick per active aviary at roughly one-minute cadence, calibrated during build. The tick must continue whether or not clients are connected.

Tick steps:

1. Load aviary, birds, current personality vectors, current moods, settings relevant to simulation, weather state, and unprocessed interaction events in order.
2. Compute local time phase from the account's last-known timezone.
3. Update weather schedule and short-lived weather modifiers.
4. Aggregate validated presence-time since the last tick.
5. Apply immediate interaction effects to mood and offer cooldown state.
6. Compute low-pass personality drift deltas from presence and interactions.
7. Apply monotonic expressive deltas to personality vectors.
8. Run mood transitions for each bird.
9. Run bird-to-bird interaction opportunities: response calls, wary spread, chorus events.
10. Select perch zones and pose families from mood/personality.
11. Schedule call windows and call grammar seeds for the next snapshot interval.
12. Generate notebook entry candidates and persist only if sparsity and specificity rules pass.
13. Write a new canonical snapshot, advance tick cursor, mark events processed, and emit aggregate tick latency metrics.

The tick must be idempotent around failure. Use transaction boundaries or per-tick version records so a crash cannot apply vector deltas twice or mark events processed without writing the state they produced.

### 5.2 Drift function

The drift function is a low-pass filter over slow signals, not a reward system. It should be implemented as a calibration module with explicit constants and test fixtures.

Inputs:

- Presence-time: dominant positive signal distributed across all birds, weighted by current visibility/attention window.
- Listen-in duration: strong bird-specific signal for social warmth and vocal frequency.
- Offers: small bird-specific or aviary-level signal; accepted or investigated offers nudge curiosity, and offer proximity can nudge boldness.
- Settle: mood quieting and presence-window closure, not a personality reward.
- Absence: no negative personality drift. It may reduce greeting likelihood indirectly through mood and lack of recent presence signals, but it never lowers stored trait values.

Output:

- Additive server-authored deltas by trait, applied in event-log order.
- Bounded monotonic movement toward expressive values. Clamp to max range.
- No per-session visible jumps. Typical regular use should be instrument-measurable after one week and user-visible after about three weeks.

Implementation approach:

- Maintain per-bird exponentially weighted accumulators for recent presence and interaction signals.
- Convert accumulators to small per-tick deltas using trait-specific coefficients.
- Gate maximum daily delta per trait to prevent active sessions from saturating values.
- Run calibration simulations using synthetic behavior profiles: absent, occasional visitor, regular quiet watcher, frequent listen-in user, heavy offer user.
- Assert that no synthetic profile produces visible drift in a single session and regular visits produce measurable movement after about a week.

### 5.3 Mood transitions

Mood is fast-timescale and visible. Use a transition model that mixes deterministic rules and weighted randomness:

- Time of day: morning biases alert/content; dusk biases drowsy/settled; night biases settled except night-active species.
- Weather: rain dampens vocal frequency and can nudge toward drowsy or wary; wind can nudge alert/wary depending on personality.
- Interactions: accepted seed or still pool can nudge content/curious; song fragment can prompt calling or quieting based on vocal frequency; settle nudges drowsy/settled.
- Bird personality: high boldness resists wary; high curiosity increases investigation; high social warmth increases greeting and response calls.
- Bird-to-bird: alarm-like calls can spread wary; chorus can lift alert/content.

Mood must persist across sessions and never snap to default on open. Store transition reasons internally for debugging and tests, but do not surface them to users.

### 5.4 Greeting selection

Return greeting is not a generic arrival animation. On snapshot requests that represent fresh navigation, return from hidden state, or long absence, the server should provide greeting candidates:

- Select one primary greeting bird based on boldness, social warmth, current mood, recent greeting history, and absence length.
- Vary greeting form by absence length: quick glance for short absence, longer call or step forward after longer absence.
- Stagger secondary greetings, if any, by small randomized offsets.
- Never generate synchronized all-bird greeting.
- Never pair greeting with textual welcome, banner, toast, or "you were gone" copy.

The client executes the greeting as part of normal bird motion and call synthesis. It should feel like the bird noticed, not like the system started a scene.

### 5.5 Offers

Offer types:

- `seed`: visual gesture; curious/content birds may approach quickly, wary birds later, drowsy birds may ignore.
- `song_fragment`: soft motif from a small library; response depends on vocal frequency, mood, and species grammar.
- `still_pool`: reflective surface in front of scene; birds may drink, bathe, or watch.

Rules:

- Offers are opened from the top bar, not by clicking directly on birds.
- Per-bird cooldown of a few minutes prevents drift saturation.
- Cooldown language, if any, should be quiet and not punitive.
- Offer reactions are simulation events, with client-side transitional rendering reconciled to snapshots.

### 5.6 Notebook generation

Build a notebook generator that reads state transitions and event summaries, not raw user behavior. It should:

- Rank candidate observations by specificity and rarity.
- Prefer relative, naturalist observations: "pip greeted before wren today" rather than "greeting event recorded."
- Enforce sparse cadence with per-kind cooldowns and global per-aviary spacing.
- Avoid entries that turn into session logs or engagement reports.
- Use lowercase, present-tense prose for product surfaces.
- Include tests for forbidden language: achievement, streak, score, level, user visited, every day, happiness meter, boldness number, etc.

Notebook generation can start from curated templates with controlled variation, as long as outputs remain specific to bird, mood, time, and observed event. Do not use an unconstrained generative model in v1 unless there is a strict safety layer, deterministic auditability, and privacy review; template-driven prose is safer and sufficient.

## 6. Sync and consistency model

### 6.1 Canonical state

The canonical state lives on the server. Multi-device sync is achieved by all clients reading the same snapshot and submitting interaction events to the same ordered log. There is no client-to-client sync and no merge of personality state.

Use optimistic concurrency internally:

- Snapshots carry a version.
- Ticks write `version + 1` only if the cursor and prior version match.
- Interaction events are accepted independently but applied only by ticks.
- Duplicate client events are deduplicated by account/session/client event ID.

### 6.2 Conflict prevention

Personality conflicts are prevented by design:

- Clients cannot submit absolute personality or mood values.
- The simulation tick consumes ordered events and applies additive deltas.
- Concurrent clients produce more events, not divergent state branches.
- A phone that opened from an older snapshot can still submit "listened in to Pip for 2 minutes"; the tick applies that event to the latest vector, not to the phone's stale vector.

For offer cooldowns and settle:

- Server validates cooldown at event ingest or tick application.
- Client should optimistically animate only reversible transitions.
- Next snapshot is authoritative.

### 6.3 Offline and network gaps

V1 should be conservative:

- If online but snapshot refresh is delayed, continue rendering last snapshot for a short grace period with local idle ornaments and audio schedule bounded by snapshot validity.
- If the client is offline, stop submitting presence and do not accumulate offline drift. The user can still see the last scene briefly, but it should not pretend canonical simulation is updating locally.
- When connection returns, pull a fresh snapshot and smoothly reconcile; do not replay long offline presence.
- Account or sync failures use matter-of-fact copy.

### 6.4 Privacy boundary in sync

Do not leak account-specific simulation details into:

- CDN cache keys that include email.
- Metrics labels.
- Client error reports.
- Visitor snapshots beyond render-needed fields.
- Analytics events.

Telemetry should know that a snapshot request succeeded in a region and how long it took, not which bird called or which account offered a seed.

## 7. Frontend rendering pipeline

### 7.1 App shell and loading

The initial path must reach first bird visible in under 500ms on a mid-tier mobile device over 4G. Build the shell around:

- Server-rendered or edge-injected minimal HTML with initial small snapshot when possible.
- Critical CSS inline or early loaded.
- First-render bird primitives available in the initial bundle.
- Non-critical surfaces code-split: account settings, accessibility settings, export/deletion, visit management, notebook deep history.
- No spinner. Slow initial state shows quiet field with soft sky color and faint motion cues.

The first frame after snapshot must show the aviary mid-action: bird pose phase already advanced, leaf or ambient state possibly in progress, calls scheduled as if continuing.

### 7.2 Scene composition

Use a single responsive horizontal scene with no scrolling/panning/zooming:

- Three depth planes: background foliage/sky, middle plane birds/perches, optional foreground branch/leaf ornaments.
- Three perch zones: front, middle, back. Server picks zones; client maps zones to responsive coordinates.
- Responsive coordinate system that preserves all birds in frame on narrow phones and wide desktops.
- Palette system keyed by local day/night phase and weather, using calm naturalist colors.
- Top bar above the scene proper, not inside it. It contains only account/settings, accessibility settings, field notebook, and offer affordance.

Top bar fade:

- Fade nearly transparent after a few seconds of cursor stillness.
- Restore on pointer movement, keyboard activity, or focus.
- Keep controls discoverable and keyboard accessible even when faded.

### 7.3 Bird rendering

Implement birds as lightweight vector/SVG/canvas/WebGL primitives after prototyping the best route against the 2MB and 60fps budgets. The rendering engine must support:

- Species silhouette families.
- Stable visual seeds for individual variation.
- Plumage saturation mapped from hidden vector to visual richness without exposing the value.
- Pose families for preen, scan, head-tilt, perch-shuffle, drowsy rest, wary watch, approach, and call.
- Mood-shaped idle selection.
- Depth-plane scaling and subtle parallax.
- Smooth perch transitions in standard mode.
- Cross-fade pose transitions in reduced-motion mode.

Avoid animation loops that replay identically. Use seeded procedural timing, pose blending, and mood/personality parameters so repeated behaviors vary.

### 7.4 Interaction rendering

Listen-in:

- Click/tap/focus bird to listen in.
- Gradually raise focused bird in mix and lower others to ambient.
- Show visual focus only as needed for accessibility; no tooltip or label in the aviary scene.
- Disengage on repeated activation, different bird focus, empty-scene click, Escape, or focus leaving.

Offer:

- Top-bar affordance opens a sparse offer chooser.
- Offer appears as a gesture in the scene without turning into a game button.
- Reaction animation follows snapshot/server event result when available; local anticipation must be subtle and reconcilable.

Settle:

- Top-bar affordance triggers slow evening lighting shift and call quieting.
- Five-second undo by any click in the aviary.
- Closing tab without settling is equivalent at engine level and has no recovery or guilt surface.

Notebook:

- Opens from top bar.
- Read-only, naturalist prose, sparse entries, indefinite scroll.
- No reactions, badges, edit/delete, or user-authored annotations.

### 7.5 Reduced-motion pipeline

Reduced-motion is a parallel rendering mode:

- Trigger from `prefers-reduced-motion` or accessibility settings.
- Replace continuous micro-motion with slow cross-fades between still poses.
- Replace flight/perch movement paths with cross-fades.
- Remove leaf/feather drift.
- Preserve day/night palette changes, slowed.
- Preserve mood, drift, calls, captions, notebook, and interactions.

Build reduced-motion as first-class renderer adapters over the same snapshot model, not as a late CSS override.

## 8. Audio pipeline

### 8.1 Procedural call grammar

Each species has motif primitives and each bird has stable call identity:

- Motifs: pitch contour, duration envelope, timbre/noise shape, rest pattern, optional trill.
- Bird seed: stable variation in pitch range, motif preference, timing offset, timbral color.
- Personality mapping: vocal frequency increases call probability and chorus participation; social warmth increases response calls; mood changes tempo, spacing, intensity, and contour.
- Species recognizability: grammar should keep each species and individual bird identifiable across mood and drift.

Use WebAudio synthesis client-side:

- Oscillator/noise nodes, envelopes, filters, gain, panning/depth cues if subtle.
- Shared node pools to prevent memory growth.
- Schedule calls from snapshot-provided windows and local seeded variation.
- Avoid exact repetition; no downloaded recorded calls.

### 8.2 Mixing

Ambient mix:

- Multiple birds can call at once as a real chorus.
- Overall loudness is constrained and calm.
- Distance/depth affects level and filtering.
- Weather and time of day dampen or warm the mix.

Listen-in mix:

- Focused bird gradually rises.
- Other birds lower but never silence.
- Engage/disengage ramps are slow enough to feel like listening, not switching channels.
- Keyboard focus can trigger the same mix behavior.

Settle/night:

- Calls quiet gradually.
- Night-active species may continue calling.

### 8.3 Captions and fallback

Caption text is generated from the same call grammar event that produces audio:

- A single sharp call from the back perch.
- A soft three-note rise.
- A low trill, paused, low trill again.

Captions:

- Optional in accessibility settings.
- On by default when WebAudio is unavailable or denied.
- Placed near the calling bird without becoming clutter.
- Fade in/out with calls.
- Use naturalist voice, not technical labels.

WebAudio fallback:

- Graceful silence.
- Captions on by default.
- Matter-of-fact settings/error copy if audio cannot start.
- No recorded fallback path.

## 9. Accessibility plan

### 9.1 Screen-reader narration

Create a narration layer fed from the same snapshot as the visual renderer:

- Default cadence around every 30-60 seconds at idle.
- Faster only for user-initiated events such as return greeting, successful offer, settle.
- Queue priority that prevents flooding and respects screen-reader interruption behavior.
- Naturalist prose: lowercase, present tense, specific, observational.
- No state-list output, no vector numbers, no "mood: content" labels.

Implementation options:

- Start with deterministic client-side narration templates using snapshot descriptors.
- Move durable or complex narration generation server-side if needed for consistency with notebook language.
- Keep narration testable with golden outputs and forbidden phrase checks.

### 9.2 Keyboard and focus

Keyboard map:

- Tab through top-bar controls.
- Tab into aviary focuses first bird.
- Arrow keys move focus among birds in stable spatial order.
- Enter toggles listen-in on focused bird.
- Escape exits listen-in or closes top-bar panels.
- Offer chooser and settings are fully keyboard navigable.
- Settle reachable from top bar.

Focus treatment:

- Soft but high-contrast outline visible against morning, evening, night, and weather palettes.
- No hover-only affordances.
- Focus state should not add labels inside the aviary scene unless needed by assistive tech.

### 9.3 Motion, captions, contrast

- Honor `prefers-reduced-motion` on first render before animations start.
- Provide explicit reduced-motion setting.
- Provide call captions setting and WebAudio-failure auto captions.
- Ensure all top-bar labels, settings, account surfaces, errors, captions, and visual narration pass WCAG AA.
- Test contrast across day/night/weather palettes.

### 9.4 Accessibility acceptance tests

Ship v1 only when:

- Screen reader can understand the aviary as a living scene without visual access.
- Keyboard-only user can sign in, name birds, listen in, offer, settle, open notebook, manage account, invite/revoke visits, and update accessibility settings.
- Reduced-motion mode shows the actual aviary state through cross-fades, not a static fallback.
- Captions match actual procedural calls.
- No accessibility surface exposes hidden trait numbers or flattens the product into state labels.

## 10. Performance and observability

### 10.1 Budgets

Initial load:

- Initial JS bundle under 2MB gzipped.
- First bird visible under 500ms on mid-tier mobile over 4G.
- Snapshot payloads in kilobytes, not megabytes.
- No blocking load on notebook history, account settings, visit management, export, or non-critical assets.

Runtime:

- 60fps idle motion on a five-year-old mid-range laptop for 30-minute sessions.
- No memory growth over 30 minutes.
- Reuse audio nodes/buffers.
- Bound workers, animation loops, timers, and event listeners.
- Stop rendering when tab hidden; simulation continues server-side.

Server:

- Simulation tick p99 latency alarm at 5 seconds.
- Tick cadence roughly once per minute with jitter/backpressure safeguards.
- Snapshot endpoint low-latency path from current snapshot cache.

### 10.2 Measurement

Synthetic checks:

- Automated browser fleet from common geographies.
- Measures navigation to first bird visible, initial JS size, snapshot latency, frame stability, audio init, reduced-motion first render, and accessibility smoke paths.

Aggregate RUM:

- Page load timings.
- First-bird render timings.
- Render-frame timings.
- Audio-context errors.
- Snapshot request latencies.
- Simulation tick latencies.
- Anonymized session-duration histograms without account or bird dimensions.

Do not collect:

- Per-bird state.
- Personality values.
- Notebook content.
- Offer/listen-in details as analytics.
- Visit relationship graph as analytics.
- Email or account UUID as telemetry dimensions.
- Raw pointer/key data.

### 10.3 CI and quality gates

CI should include:

- Unit tests for drift monotonicity, event idempotency, mood persistence, cooldowns, visit read-only restrictions, magic-link expiry, and deletion.
- Calibration simulations for drift timing.
- Snapshot schema contract tests.
- Accessibility tests for keyboard and ARIA/narration behavior.
- Bundle-size budget gate.
- Performance regression test for no memory growth and representative idle frame timing.
- Audio graph tests for node cleanup and caption matching.
- Privacy lint/tests that reject email/account/bird identifiers in telemetry labels.
- Forbidden-surface tests for gamification language and welcome toasts in product UI.

## 11. Security and privacy implementation

### 11.1 Auth security

- Magic links expire after 15 minutes.
- Links are single-use and stored hashed.
- Rate-limit requests by keyed email hash and IP.
- Sessions use secure, HTTP-only cookies or equivalent hardened token storage.
- Session tokens are hashed at rest and revocable per device.
- Email change requires new address verification before commit.

### 11.2 Data protection

- Encrypt email at rest.
- Use account UUID everywhere else.
- Restrict export links to short-lived signed URLs.
- Soft-delete immediately hides/restores account state; hard-delete after 30 days removes birds, vectors, notebook, visits, sessions, interaction events, telemetry ties if any, and export artifacts.
- Privacy policy in account settings names aggregate categories and explicitly excludes per-bird interaction state from analytics/training/third-party sharing.

### 11.3 Access control

- Host endpoints require account session and account ownership.
- Visit endpoints require valid visit session and active invite.
- Visitor response serializer excludes host account data and mutation affordances.
- Revocation takes effect on next visitor snapshot pull.
- Deleted or soft-deleted host account invalidates visits.

## 12. Implementation phases

### Phase A: Foundations and contracts

Deliver:

- Domain schema migrations for accounts, sessions, aviaries, birds, vectors, moods, event log, snapshots, notebook, visits.
- Snapshot schema and interaction event schema.
- Auth flow with magic links and sessions.
- Privacy-safe logging and telemetry primitives.
- Starter species data structure and two-bird account bootstrap.
- Basic app shell with quiet field loading state.

Exit criteria:

- New account can sign in, create one aviary with two named starter birds, and fetch a snapshot.
- No email used as internal identifier outside encrypted account record and keyed lookup.
- API role cannot mutate personality vectors.

### Phase B: Simulation core

Deliver:

- Tick runner with idempotent event consumption.
- Personality vector persistence and monotonic drift module.
- Mood transition module.
- Presence ping ingestion and validation.
- Perch selection, return-greeting candidate selection, weather scheduling, call schedule descriptors.
- Calibration harness for synthetic user profiles.

Exit criteria:

- Regular-visit synthetic profile produces measurable drift after about a week and no visible single-session jump.
- Neglect never lowers stored personality traits.
- Mood persists across sessions and changes with ticks/time/weather.
- Multiple clients submitting events cannot overwrite vector state.

### Phase C: Rendering and interactions

Deliver:

- Responsive single horizontal scene.
- Bird renderer with species silhouettes, pose families, mood-shaped idle, and first-frame mid-motion.
- Top bar with account/settings, accessibility, notebook, offer.
- Listen-in, offer, settle, and undo interactions.
- Snapshot interpolation and visibility-change refresh.
- Reduced-motion rendering adapter.

Exit criteria:

- First bird visible within target on test profile.
- No spinner or welcome toast exists.
- Keyboard user can navigate top bar and birds.
- Hidden tab stops rendering and visible return refreshes snapshot.

### Phase D: Audio and accessibility

Deliver:

- WebAudio procedural call engine.
- Per-bird call grammar signatures, chorus mixing, listen-in ramps.
- Caption generation from call grammar.
- WebAudio failure path with captions.
- Screen-reader narration.
- Contrast and focus treatment across palettes.

Exit criteria:

- Two to seven bird mixes remain individually recognizable in user testing targets.
- Captions match generated calls.
- Reduced-motion and screen-reader users receive a complete aviary experience.
- Audio graph shows no memory growth in 30-minute test.

### Phase E: Notebook, account operations, and visits

Deliver:

- Sparse notebook generator and notebook UI.
- Rename, export, session revocation, email change, deletion/recovery.
- Visit invite, consume, read-only snapshot, revocation, expiration, visit log, optional notifications.

Exit criteria:

- Notebook entries are specific, sparse, naturalist, and never event-log/gamification copy.
- Export contains account snapshot and is emailed to verified address.
- Visitor cannot mutate host state or affect drift.
- Revoked visitor sees matter-of-fact unavailable surface on next snapshot.

### Phase F: Hardening and launch ramp

Deliver:

- Synthetic monitoring and aggregate RUM.
- Privacy/security review.
- Bundle and runtime performance gates.
- Drift calibration review.
- Accessibility review with assistive-tech users.
- Internal beta, limited external beta, staged rollout.

Exit criteria:

- All budgets pass in CI and synthetic checks.
- Tick p99 alarm and dashboards are live.
- No telemetry path can read simulation records for aggregate behavior analysis.
- Launch checklist signs off on non-goals and forbidden surfaces.

## 13. Rollout plan

Start with a small internal dogfood cohort, then invite-only beta, then gradual public opening.

Dogfood goals:

- Validate first-frame aliveness, return-greeting variation, procedural call non-repetition, top-bar fade, and absence of announcement surfaces.
- Exercise multi-device sync and session revocation.
- Tune presence activity window so quiet watching still counts.
- Test reduced-motion and captions from day one.

Beta goals:

- Calibrate drift pacing with real regular-use patterns while respecting privacy. Use per-account simulation only for that user's experience; use synthetic and opt-in qualitative research for calibration review rather than aggregate per-bird analytics.
- Validate audio recognizability from two to seven birds in controlled tests.
- Tune notebook sparsity and language.
- Confirm visit invitation expectations and revocation behavior.

Public rollout:

- Start all accounts with two birds.
- Keep age-based additional birds conservative; ramp only after audio recognizability and scene density hold up.
- Keep visit notifications off by default.
- Monitor aggregate operational health, not bird behavior.

Instrumentation live from day one:

- First bird visible.
- Initial bundle size.
- Snapshot latency.
- Tick latency.
- Frame timing.
- Audio-context errors.
- Memory-growth synthetic test.
- Auth error rates.
- Visit revocation/snapshot authorization failures as counts only.

## 14. Risks and mitigations

### Drift calibration risk

Risk: Drift is too fast and the product becomes a stat-management toy, or too slow and presence feels irrelevant.

Mitigation:

- Build calibration harness before polishing interactions.
- Maintain synthetic profiles and golden expected ranges.
- Gate daily maximum deltas.
- Run beta qualitative reviews focused on whether users notice change only in retrospect.

### Presence inflation risk

Risk: Counting tab-open or focus alone corrupts drift across accounts.

Mitigation:

- Implement the exact three-condition presence rule.
- Server clamps durations and rejects stale/overlapping pings.
- Tests cover hidden tab, unfocused window, no recent activity, laptop sleep, and settle/tab-close.

### Sync correctness risk

Risk: Concurrent devices or stale snapshots overwrite personality state.

Mitigation:

- No client vector writes.
- Ordered append-only events.
- Idempotent tick transactions.
- Snapshot versions and event cursors.
- Concurrency tests with overlapping laptop/phone sessions.

### Audio uncanniness risk

Risk: Calls repeat, phase, sound synthetic in the wrong way, or blur above a few birds.

Mitigation:

- Procedural grammar from the start.
- No recorded loops.
- Per-bird stable seeds plus runtime variation.
- Chorus tests with two to seven birds.
- Audio-design review before increasing bird-count availability.

### Accessibility regression risk

Risk: Accessible surfaces become flattened labels or reduced-motion becomes a stripped product.

Mitigation:

- Treat narration, captions, keyboard, and reduced-motion as core phases, not post-launch work.
- Include assistive-tech acceptance tests.
- Golden tests for naturalist narration and captions.
- Reduced-motion renderer shares snapshot model with standard renderer.

### Product-boundary creep risk

Risk: Helpful-looking features introduce gamification, announcement UI, social loops, or pet-care obligations.

Mitigation:

- Add forbidden-surface tests and copy review.
- Keep top bar sparse by architecture.
- Require product review for any new user-facing counter, notification, badge, public surface, or account metric.
- Document non-goals in engineering onboarding.

### Privacy boundary risk

Risk: Per-bird interaction data leaks into analytics, logs, model training, or identifiers.

Mitigation:

- Separate telemetry module that cannot import simulation storage.
- PII-safe logging lint.
- No account UUID, bird ID, email, notebook text, or event payload in metrics labels.
- Privacy review for every new event type.

### Performance risk

Risk: Rich rendering/audio exceeds bundle, first-bird, memory, or frame budgets.

Mitigation:

- Bundle gate early.
- Code-split non-core surfaces.
- Snapshot-first render path.
- Reuse audio nodes and animation objects.
- Synthetic 30-minute runtime test in CI.

## 15. Engineering guardrails

Use these guardrails throughout implementation:

- Any code that wants to mutate personality vectors must live in the simulation module and run inside tick processing.
- Any UI copy that greets the user textually on return is a bug.
- Any user-visible counter related to visits, birds collected, progress, score, streak, or level is a bug.
- Any visitor event that enters drift calculation is a bug.
- Any client-side simulation tick is a bug.
- Any recorded bird-call audio file in the product bundle is a bug.
- Any accessible narration that exposes raw state labels instead of naturalist prose is a bug.
- Any telemetry dimension that can identify an account, bird, visitor, notebook entry, or interaction content is a bug.

## 16. Open implementation decisions

These are defensible calls the engineering team should settle during build without reopening product scope:

- Exact numeric trait ranges and drift coefficients.
- Exact presence recent-activity window, leaning long enough to count quiet watching.
- Final mood enum and transition weights.
- Tick cadence, starting near one minute.
- Snapshot polling cadence versus SSE/WebSocket, based on battery and latency tests.
- Rendering technology choice for birds after bundle/performance prototypes.
- Species pool details and motif libraries.
- Notebook template library size and suppression thresholds.
- Age schedule for third through seventh birds.
- Whether narration templates run client-side, server-side, or hybrid.

These decisions must preserve the PRD invariants rather than create new product surfaces.
