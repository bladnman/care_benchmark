# Pocket Aviary implementation plan

## 1. Product intent and v1 scope

Pocket Aviary v1 is a modern-browser web product built around one private, persistent aviary per account. The first usable version must make a small set of birds feel like they have been living continuously, noticing the user quietly, and changing slowly over weeks. The primary engineering goal is therefore not feature breadth; it is continuity, procedural variation, and restraint.

V1 includes:

- Email magic-link accounts with one canonical aviary per account.
- Two starter birds at account creation, with server-paced expansion up to seven birds based on aviary age.
- A single horizontal responsive aviary scene with three perch zones, local-time day/night, rare ambient weather, and continuous ambient motion.
- Server-side simulation tick as the only writer of canonical bird personality, mood, position, active weather, and notebook-trigger state.
- Hidden per-bird personality vectors, visible only through motion, calls, perch choice, color richness, greetings, and notebook observations.
- Mood-shaped idle motion, return-greetings, listen-in, offer, settle, and field notebook.
- Procedural client-side WebAudio calls and call captions generated from the same grammar.
- Multi-device sync by server snapshot plus append-only client interaction events.
- Read-only visit invitations, off by default, revocable, expiring after 30 days.
- First-class accessibility surfaces: screen-reader narration, call captions, reduced-motion rendering, keyboard navigation, and WCAG AA text contrast.
- Account export, soft deletion with 30-day recovery, device session management, and privacy-respecting aggregate operational telemetry.

V1 explicitly excludes native apps, passwords, SSO, payments, multiple aviaries, shared aviaries, public discovery, profiles, follows, chat, comments, leaderboards, achievements, scores, streaks, visit calendars, hunger/death/distress mechanics, direct bird placement, scene customization, push notifications, and any user-visible numeric trait/stat surface. These exclusions should be enforced in product review and in code review because many are cheap to add and expensive to undo.

Key planning calls where the PRD leaves implementation room:

- Use a TypeScript web stack end to end so simulation formulas, snapshot schemas, and client rendering contracts can share types without sharing authority.
- Use a relational primary store for canonical state and a durable append-only interaction event table rather than client-sourced document sync.
- Use a small server-rendered boot payload or edge-adjacent snapshot endpoint for time-to-first-bird, with full account surfaces code-split away from the aviary route.
- Use deterministic seeded procedural choices for greetings, idle beats, weather, calls, and notebook candidates so variation is real but debuggable.

## 2. Architecture

The system has four runtime surfaces:

1. Browser client
   - Renders the aviary, synthesizes calls, records presence signals, and submits interaction events.
   - Pulls canonical snapshots and interpolates between them.
   - Never computes or persists personality drift.
   - Never sends absolute personality, mood, or position values.

2. Web/API service
   - Handles auth, sessions, account settings, snapshots, notebook reads, visit invites, visit snapshots, event ingestion, export requests, and deletion requests.
   - Applies authorization and privacy boundaries.
   - Performs lightweight validation, idempotency, and rate limiting.

3. Simulation service
   - Runs the slow server-side tick, roughly once per minute per non-deleted aviary.
   - Consumes unprocessed interaction events in order.
   - Updates canonical bird state, mood timers, perch targets, weather windows, call scheduling seeds, notebook candidate state, and personality-vector deltas.
   - Is the only writer of personality vectors.

4. Background job workers
   - Send magic links and visit invites.
   - Produce account exports and emailed download links.
   - Finalize hard deletion after the 30-day soft-delete window.
   - Run synthetic performance checks outside the user request path.

Recommended deployment shape:

- A TypeScript React client with a minimal route shell and code-split surfaces.
- A TypeScript API service backed by PostgreSQL.
- A simulation worker scheduled by a durable job queue or database lease table. The tick must tolerate duplicate scheduling, so each tick execution uses per-aviary locking and an event cursor.
- Object storage for generated export files with short-lived signed download URLs.
- Email provider integration for magic links, export links, and visit invitations.
- CDN/edge caching for static assets and the unauthenticated shell only. Authenticated snapshots are not publicly cached.

The architectural boundary is strict: client code may contain rendering helpers, call synthesis, presence detection, and interpolation, but not authoritative drift or merge logic. Server code may expose prose narration and caption text, but must not expose numeric personality values to users. Internal admin and debug tooling can inspect numeric values in protected environments, but no product surface or account export UI may render them as stats; the export may include raw state because the PRD explicitly names personality vectors as exportable.

## 3. Data model

Use synthetic UUIDs for every account reference outside the encrypted email column. No log line, event partition, metric tag, or foreign key should use email as identity.

Core tables:

### accounts

- id: UUID primary key.
- encrypted_email: encrypted string, unique through a normalized-email hash.
- email_lookup_hash: keyed hash used only for login lookup and uniqueness.
- email_verified_at.
- created_at, updated_at.
- soft_deleted_at, hard_delete_after.
- timezone: latest confirmed user timezone for simulation display defaults.
- visit_notifications_enabled: boolean default false.
- accessibility_settings: JSON object for reduced motion override, captions default, narration preferences.
- privacy_policy_version_acknowledged.

### account_sessions

- id: UUID.
- account_id.
- device_label, user_agent_family, created_at, last_seen_at, revoked_at.
- session_token_hash.
- last_ip_country or coarse region if needed for security review; do not store high-resolution location by default.

### magic_links

- id: UUID.
- email_lookup_hash.
- token_hash.
- expires_at, consumed_at, requested_ip_rate_bucket.
- created_at.

### aviaries

- id: UUID.
- account_id unique.
- created_at.
- current_version: monotonically increasing integer.
- local_timezone.
- bird_cap: integer default 7.
- settled_until_reengage: boolean or current settled state.
- current_day_phase: enum derived by tick from timezone.
- active_weather_id nullable.
- simulation_cursor_event_id: last processed interaction event.
- last_tick_at, next_tick_after.

### birds

- id: UUID stable identity.
- aviary_id.
- species_key.
- display_name.
- name_updated_at.
- adopted_at.
- ordinal_in_aviary.
- call_signature_seed.
- visual_seed.
- current_perch_zone: front, middle, back.
- target_perch_zone nullable.
- perch_transition_started_at, perch_transition_ends_at.
- current_pose_key.
- created_at, updated_at.

### bird_personality_vectors

- bird_id primary key.
- boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity.
- vector_version.
- updated_at.

Values are normalized internal floats. Clamp all traits to calibrated min/max ranges. Only the simulation service may update this table.

### bird_mood_states

- bird_id primary key.
- mood: wary, content, curious, drowsy, alert, settled, plus any final implementation states.
- mood_intensity: optional internal scalar.
- mood_started_at.
- mood_expires_after nullable.
- last_transition_reason_key internal.
- updated_at.

Mood is persisted and not reset on page open.

### interaction_events

- id: UUID.
- aviary_id.
- account_id.
- bird_id nullable.
- source_session_id.
- event_type: presence_ping, presence_end, listen_in_start, listen_in_end, offer_seed, offer_song_fragment, offer_still_pool, settle_start, settle_undo, settle_complete, rename_bird, accessibility_changed.
- client_event_id: idempotency key generated by client.
- occurred_at_client.
- received_at_server.
- payload: JSON constrained by event type.
- processed_at nullable.
- rejected_reason nullable.

Presence and interaction events are private simulation inputs. They are not exported to analytics.

### presence_windows

Derived by the simulation service from validated presence pings.

- id.
- aviary_id, account_id, source_session_id.
- started_at, ended_at.
- duration_seconds.
- confidence_flags: which browser signals were present.
- consumed_for_drift_at nullable.

The client sends pings only when visibility, focus, and recent pointer/key activity are all true. The server still consolidates and bounds windows to prevent runaway presence from stale tabs.

### offer_cooldowns

- bird_id.
- offer_type.
- cooldown_until.
- last_offer_event_id.

Cooldown is per bird and offer type or per bird across all offer types, depending on calibration. Start with per-bird global cooldown to keep offers gesture-like.

### notebook_entries

- id.
- aviary_id.
- entry_date_local.
- prose_text.
- observation_key.
- referenced_bird_ids array.
- created_at.
- source_tick_id.
- importance_score internal.

Entries are read-only and sparse. They are naturalist prose, not event logs.

### visit_invitations

- id.
- host_account_id.
- host_aviary_id.
- visitor_email_encrypted.
- visitor_email_lookup_hash.
- token_hash.
- created_at, expires_at, accepted_at, revoked_at.
- last_visit_started_at nullable.

### visit_sessions

- id.
- invitation_id.
- host_aviary_id.
- started_at, ended_at.
- approximate_duration_seconds.
- last_snapshot_at.
- terminated_reason nullable.

Visit sessions do not generate host presence events and cannot write interaction events.

### account_exports

- id.
- account_id.
- requested_at, completed_at, expires_at.
- object_storage_key.
- download_token_hash.
- status.

### telemetry_events

Do not create a general per-account behavioral table. Operational telemetry should flow to the metrics system directly with allowed fields only: route, status code, latency bucket, browser family, coarse geography if needed, bundle version, first-bird timing, frame timing bucket, audio error category, tick latency, and tick failure category.

## 4. API surface

All APIs return matter-of-fact error messages. Product-facing prose is returned only for aviary observations, notebook entries, captions, and narration.

### Auth and account

- POST /api/auth/magic-link
  - Body: email.
  - Creates a 15-minute single-use magic link with rate limiting by email hash and IP bucket.
  - Response is intentionally non-enumerating: "If that email can sign in, we sent a link."

- POST /api/auth/consume
  - Body: token.
  - Invalidates token on successful use and creates a device session.
  - Creates account and starter aviary if email is new.

- GET /api/account
  - Returns account settings, session list, export/deletion status, accessibility settings, and visit notification preference.

- PATCH /api/account
  - Updates matter-of-fact account settings only: accessibility settings, visit notifications, timezone, device labels.

- POST /api/account/email-change
  - Starts new-email verification. Commit only after new address verifies.

- DELETE /api/sessions/{sessionId}
  - Revokes a device session.

- POST /api/account/export
  - Queues export and emails signed link when ready.

- POST /api/account/delete
  - Marks account soft-deleted, sets hard_delete_after.

- POST /api/account/recover
  - Restores within 30 days.

### Aviary snapshots

- GET /api/aviary/snapshot
  - Authenticated host endpoint.
  - Returns snapshot_version, server_time, local_day_phase, weather, settled state, birds with species, name, mood expression keys, perch/pose state, transition timing, call schedule seeds, notebook unread count if needed without badges, and accessibility prose snippets where relevant.
  - Does not return numeric personality values.
  - Supports If-None-Match or since_version for low-frequency polling.

- GET /api/aviary/bootstrap
  - Used during initial page load to deliver the smallest state needed to draw first bird within 500ms.
  - May be embedded into server-rendered HTML for authenticated requests when possible.
  - Contains no account settings beyond what the first render needs.

- GET /api/aviary/notebook
  - Paginated read-only notebook entries.
  - No event-log language, no user-behavior summaries.

- GET /api/aviary/narration
  - Optional endpoint if narration is server-generated. Returns current slow-cadence naturalist narration derived from snapshot state.
  - Can be folded into snapshot if simpler.

### Interaction ingestion

- POST /api/aviary/events
  - Body: array of client events with client_event_id, event_type, occurred_at_client, optional bird_id, and payload.
  - Validates session ownership, event type, event timestamp bounds, cooldown eligibility, and bird membership.
  - Returns accepted/rejected statuses, current snapshot_version, and any immediate user-facing observation needed for offer or settle feedback.
  - Does not update personality directly.

Recommended event semantics:

- listen_in_start and listen_in_end include bird_id, audio focus start/end, and client monotonic time for duration sanity checks.
- offer events include offer_type and selected song_fragment_key when relevant.
- presence_ping is accepted only when the client asserts visible, focused, and recent_activity true. The server requires regular pings and closes windows after missing heartbeat threshold.
- settle_start writes an event immediately; settle_undo within five seconds writes a separate event. If no undo arrives, the client can send settle_complete or the server can infer completion during tick.

### Bird naming and adoption

- PATCH /api/birds/{birdId}
  - Rename only. No species, personality, perch, or mood updates.

- GET /api/aviary/adoption-offer
  - Returns whether an age-paced new bird is available.
  - No count-progress copy, no "earn" language.

- POST /api/aviary/adopt
  - Accepts the currently available age-paced bird offer and name.
  - Server selects species; user does not browse a catalog.

This endpoint can ship after the starter experience if the age gate means no user can reach it in early v1, but the data model should support it from day one.

### Visit invitations

- GET /api/visits/invitations
  - Host account settings surface: outstanding invites, recent visit log, revoked/expired state.

- POST /api/visits/invitations
  - Body: visitor email.
  - Creates one-time visit link expiring in 30 days.

- DELETE /api/visits/invitations/{inviteId}
  - Revokes immediately.

- GET /api/visit/{token}/snapshot
  - Visitor read-only endpoint.
  - Returns same rendering snapshot as host, with visitor permissions and no notebook mutation, no listen-in, no offer, no settle.
  - If revoked/expired, returns matter-of-fact "visit no longer available" surface.

Visit clients must not call /api/aviary/events.

## 5. Simulation engine design

### Tick scheduling and correctness

Run a tick approximately once per minute for every active aviary. The tick cadence can jitter slightly to avoid load spikes, but the simulation uses actual elapsed time so jitter does not alter drift. Each tick:

1. Acquires a per-aviary lease or row lock.
2. Reads the aviary, birds, personality vectors, mood states, active weather, and unprocessed interaction events after simulation_cursor_event_id.
3. Consolidates presence pings into presence windows.
4. Applies offer cooldowns and interaction-derived mood nudges.
5. Computes slow personality deltas from eligible presence/listen/offer inputs.
6. Computes mood transitions from recent interactions, local time, weather, bird-to-bird effects, and personality.
7. Chooses perch targets and pose families.
8. Advances or ends weather events.
9. Produces call scheduling seeds and chorus opportunities for client synthesis.
10. Emits rare notebook entries when observation criteria pass sparsity gates.
11. Updates canonical state and current_version atomically.
12. Marks consumed events with processed_at and advances the cursor.

If a tick fails mid-transaction, no partial state is committed. If a tick is retried, idempotency comes from processing only unprocessed events and from deterministic random seeds keyed by aviary_id, tick window, and state version.

### Presence consolidation

Presence is valid only when all three browser-side facts are true: document visible, window focused, and pointer/key activity in the recent activity window. Start calibration with a recent activity window of 180 seconds, then tune with user research and synthetic testing. To avoid punishing still watching, err toward a longer window rather than a twitchy one.

Server rules:

- Presence pings are accepted at a low cadence, for example every 15-30 seconds while valid.
- A presence window starts at the first valid ping after absence.
- A window ends on explicit presence_end, settle, tab-close beacon if delivered, session expiration, or heartbeat timeout.
- Heartbeat timeout should be generous enough to handle mobile sleep but bounded enough to prevent all-night drift. Start with 90 seconds after the last valid ping for active browser sessions and cap any single inferred window at a calibrated maximum.
- The server ignores pings from visitor sessions.

Presence-time is the dominant drift input. It is not displayed as a counter or exported as a user-facing engagement metric.

### Personality drift

Implement personality drift as a slow, additive, bounded low-pass update. The exact formula should be versioned and calibration-tested, but the shape should be:

- Compute daily or tick-window signal aggregates per bird:
  - presence_time_shared: valid host presence while bird exists.
  - listen_in_duration_for_bird.
  - offer_count_for_bird, offer_acceptance_or_interest outcome.
  - settle events only as mood quieting, not as a strong trait delta.
- Convert signals to tiny positive deltas with diminishing returns per day.
- Apply trait-specific weights:
  - Boldness: presence + offers near bird.
  - Social warmth: presence + return-greeting participation + listen-in.
  - Vocal frequency: listen-in + chorus participation + presence.
  - Plumage saturation: sustained presence over multi-day windows.
  - Curiosity: offers and ambient novelty responses.
- Clamp to max expressive bounds.
- Never apply negative deltas for neglect.

Calibration target:

- Instrument-measurable drift after about one week of regular visits.
- User-visible drift after about three weeks.
- No single session produces a visible personality jump.

Add drift formula versioning:

- Store formula_version in tick audit logs or vector update records.
- Maintain migration tooling for future formula changes without resetting birds.
- Never recompute personality from raw event history as the runtime source of truth. Event history can support offline calibration in non-production fixtures, but production personality state is persisted and updated forward.

### Mood transitions

Mood is a small persisted enum with optional intensity and expiry. The transition system should be probabilistic but bounded, with deterministic seeded randomness for reproducibility.

Inputs:

- Current mood and duration in that mood.
- Local time phase: morning, midday, evening, night.
- Weather: rain dampens vocal tendency; wind nudges alert/wary depending on bird.
- Recent events: offer accepted, offer ignored, listen-in, settle, return after absence.
- Personality: high boldness resists wary; high curiosity explores offers; high vocal frequency joins chorus.
- Bird-to-bird influence: alarm-like or wary states can spread softly; content calls can invite chorus.

Outputs:

- Mood enum/intensity.
- Pose family.
- Perch preference.
- Call probability modifier.
- Offer reaction tendencies.

Mood persists across sessions. Opening a tab does not reset it. The return-greeting is generated from current state plus absence length; it is not a reset animation.

### Return-greeting

On each host session start or visibility return after a meaningful absence, generate a greeting plan:

- Candidate birds are weighted by boldness, social warmth, current mood, recent greeting history, and absence length.
- Choose one primary greeter. If others respond, stagger them by small offsets.
- Select greeting action: glance, head tilt, short call, step forward, longer call, paired response.
- Return a short-lived greeting event in the snapshot or immediate session response so the client renders it promptly.

No text welcome is generated. No "you have been away" copy is generated. If narration is active, the greeting can be narrated as an observation, not an announcement.

### Offers

Offers are gestures from the top bar, not direct manipulation of birds. The server validates cooldowns and records the offer event. The simulation chooses the reaction based on mood and personality:

- Seed: curious/content birds approach more readily; wary birds delay; drowsy birds may ignore.
- Song fragment: call grammar may respond by joining, pausing, or calling against the motif.
- Still pool: birds may drink, bathe, watch, or ignore.

Offer outcome affects fast mood immediately or at next tick, and contributes only small slow drift. The cooldown prevents single-session trait saturation.

### Settle

Settle is a state transition plus interaction event:

- Client starts slow evening-lighting transition and quiets calls.
- Server records settle_start, ends current presence window, and nudges mood quieting.
- Five-second undo writes settle_undo and reverses client lighting.
- After completion, aviary remains settled until close or active re-engagement.

Closing without settle ends presence without penalty.

### Weather and day/night

The tick owns weather windows and day/night phase. Local timezone comes from account settings/client timezone and should be kept current without being noisy.

- Rain occurs a few times per week per aviary, not globally for all users at once.
- Wind/leaf events can be client-rendered ornaments, but weather that affects mood must be in canonical state.
- Night is quiet, not dead; support one nightjar-like species remaining active.

### Notebook generation

Notebook entries are rare observations. Generate candidates during tick when meaningful state changes or unusual combinations occur, then apply sparsity rules.

Candidate triggers:

- First greeter changed compared with recent history.
- Notable quiet stretch.
- Weather plus mood effect.
- Offer reaction with specific bird behavior.
- Perch pattern that reflects slow drift.
- Chorus involving identifiable birds.

Rules:

- Naturalist voice, lowercase, present tense, specific.
- No user-behavior summaries, streaks, visit counts, or raw state values.
- Roughly one entry every few days for regular use, with maximum frequency caps.
- Store final prose, not just templates, so the record remains stable.

Use templated prose with controlled variation for v1 rather than open-ended generative text. It is safer for tone, privacy, localization later, and repeatability.

## 6. Sync model

The sync model is server canonical state plus append-only events. There is no client-to-client merge.

Client rules:

- Pull snapshot on initial load.
- Pull snapshot on visibility becoming visible.
- Pull snapshot after long frame gaps or resume from sleep.
- Poll at low frequency while visible, or use SSE/WebSocket for version notifications if it can be done within complexity and battery budgets.
- Submit events with idempotency keys.
- Render optimistically only for ephemeral UI/audio transitions that the server can later confirm or correct without changing personality.

Server rules:

- current_version increments on every canonical state update.
- Snapshots include version and server_time.
- Event ingestion accepts duplicates by client_event_id without double-applying.
- The simulation cursor processes events in received order with timestamp sanity checks.
- Personality vectors are updated only by simulation tick.
- Conflicts are prevented by authority boundaries rather than resolved by last-write-wins.

Multi-device scenarios:

- Laptop listen-in and phone open concurrently: both submit events; tick consumes in order; both devices see subsequent canonical version.
- Phone opened from stale snapshot: client displays boot state quickly, then refreshes snapshot on visibility and reconciles visual transitions.
- Session expires mid-write: event ingestion returns matter-of-fact auth error; no partial personality mutation has occurred.
- Revoked session: subsequent event writes fail; snapshot pulls redirect to sign-in.

## 7. Frontend rendering pipeline

### Application structure

Use route-level code splitting:

- Aviary route: minimal shell, renderer, audio engine, snapshot client, presence tracker, top bar icons.
- Account/settings route: loaded on demand.
- Accessibility settings: small enough to load with settings or as a separate chunk.
- Notebook panel: lazy-loaded after first bird render.
- Visit route: renderer plus read-only snapshot client; no event submission bundle.

Initial JS must stay under 2MB gzipped. Add CI bundle-size gates from the first milestone.

### Scene renderer

Use Canvas 2D or WebGL via a thin custom renderer, chosen after a prototype measures asset complexity and mobile performance. Avoid a heavy game engine. The renderer should support:

- One horizontal responsive scene with stable aspect constraints.
- Three depth/perch zones with no panning, scrolling, or zooming.
- Per-bird skeleton or pose-state rendering.
- Procedural or compact SVG/bitmap bird assets.
- Mood-shaped idle micro-motion.
- Subtle foreground/background parallax.
- Day/night palette shifts.
- Weather overlays.
- Reduced-motion alternate path.

The first frame should draw from the bootstrap snapshot with birds already mid-action. If snapshot fetch is delayed, render the quiet field loading state, not a spinner. Once a bird has appeared after adoption, never show an empty aviary as a normal return state.

### Motion system

Represent bird animation as a state machine:

- pose_family: perch_idle, preen, scan, call, head_tilt, step_forward, fly_between_perches, settled_sleep.
- mood modifiers: wary scans/back perch; content preens; curious tilts/investigates; drowsy low posture; alert upright.
- transition timing from canonical snapshot where needed.
- client-only micro-variation for breathing, feather shifts, and tiny weight changes.

Client-only ambient ornaments:

- Leaf/feather drift.
- Subtle foreground parallax.
- Tiny lighting noise.

Canonical/stateful visual features:

- Bird identity, species, mood, perch zone, transition target.
- Weather that affects mood.
- Settled state.
- Day/night phase.

### Top bar and interaction UI

Top bar contains only account/settings, accessibility settings, notebook, offer, and settle if settle is separate from offer/actions. It fades nearly transparent after cursor stillness and returns on cursor movement or keyboard activity.

No UI chrome appears inside the aviary scene except call captions when enabled and focus indicators for keyboard access. No tooltips over birds, no labels, no status icons, no badges.

Offer UI:

- Opens from top bar.
- Contains seed, song fragment, still pool.
- Uses naturalist voice.
- Does not imply feeding or caretaking.
- Handles cooldown with restrained disabled state; avoid punitive copy.

Notebook:

- Icon in top bar.
- Read-only list of entries.
- No badge for "new" entries if it feels like notification pressure; if a subtle unread indicator is needed for usability, prefer an accessible label inside the opened notebook rather than a persistent badge.

### Keyboard model

- Tab traverses top bar controls.
- Tab into scene focuses the first bird.
- Arrow keys move between birds.
- Enter toggles listen-in on focused bird.
- Escape exits listen-in or closes open panel.
- Offer panel and settings are fully keyboard-navigable.
- Focus outline is visible over bright, evening, and night palettes.

### Reduced-motion renderer

Reduced-motion mode is selected by prefers-reduced-motion or explicit setting. It uses the same snapshot state but different presentation:

- Replace micro-motion loops with slow cross-fades between still poses.
- Replace flights with cross-fades between perch positions.
- Remove leaf/feather drift.
- Keep day/night color changes, slowed.
- Keep calls, captions, notebook, mood, drift, and interactions.

This mode should be tested as a first-class renderer, not as animation-disabled CSS.

## 8. Audio pipeline

Calls are procedural via WebAudio. No recorded audio fallback exists.

### Call grammar model

Each species defines:

- Motif primitives: pitch contour, duration, envelope, harmonic/noise mix.
- Timing ranges.
- Mood modifiers.
- Recognizable signature constraints.
- Caption grammar fragments.

Each bird instance has:

- call_signature_seed.
- species_key.
- vocal_frequency trait internal to server.
- current mood.
- call schedule seed in snapshot.

The client synthesizes calls from these inputs. The server need not transmit raw personality values; it can transmit derived call parameters such as call_density_bucket, motif_variant_seed, and mood_key.

### Mixing

Default mix:

- Each bird has recognizable spatial placement and low-volume ambient presence.
- Chorus events are layered with varied timing and pitch to avoid loop artifacts.
- Evening/night phase lowers density and volume.
- Rain dampens vocal density.

Listen-in:

- Focused bird ramps up gradually.
- Other birds ramp down but never to silence.
- Disengage ramps back to ambient.
- No hard cuts.

Settle:

- Overall call density and gain quiet over a few seconds.
- If user undoes settle within five seconds, ramp back gently.

### Captions and fallback

Caption text is generated from the actual procedural call parameters:

- "a soft three-note rise"
- "a low trill, paused, low trill again"
- "a single sharp call from the back perch"

When WebAudio is unavailable, audio is silent and captions are enabled by default for that session. The user sees a matter-of-fact accessibility/settings indication if needed, not a product-surface apology.

### Performance

- Reuse AudioNodes and buffers where possible.
- Avoid per-call allocations that accumulate.
- Bound the number of simultaneous calls.
- Suspend audio context when hidden, but do not treat that as simulation pause.
- Add a 30-minute memory-growth test focused on audio allocations.

## 9. Accessibility surfaces

Accessibility must ship in v1, not as a later patch.

### Screen-reader narration

Provide a live region or equivalent narration channel that updates slowly:

- Idle cadence: every 30-60 seconds.
- Immediate priority for return-greeting, offer reaction, settle, and major state changes.
- Naturalist prose, lowercase, present tense, specific.
- No raw mood labels unless phrased observationally.
- No personality vector numbers, perch numbers, or event-log phrasing.

Use a queue with coalescing so narration does not flood the screen reader. A new high-priority event can replace stale idle narration.

### Semantic model

- Top bar controls have matter-of-fact accessible names where they are controls.
- Bird focus targets have naturalist labels or names without numeric stats.
- Account/settings/error surfaces use matter-of-fact English.
- Visit revoked/expired surfaces are clear and direct.

### Captions

- User can enable from accessibility settings.
- Auto-enable when WebAudio fails.
- Position near calling bird without obscuring the scene.
- Fade in/out with call; respect reduced motion by simplifying fade timing.
- Meet WCAG AA contrast.

### Visual accessibility

- Contrast tokens verified for top bar, captions, panels, settings, errors, and focus outlines across morning, evening, and night palettes.
- Do not rely on color alone for offer state, selected listen-in state, or errors.
- Hit targets support touch on phone viewports.

### Testing

- Automated axe/core checks for account/settings/notebook/offer panels.
- Keyboard traversal tests for top bar, scene bird focus, listen-in, offer, settle, and notebook.
- Manual screen-reader QA on VoiceOver Safari and NVDA/Firefox or equivalent.
- Reduced-motion visual regression tests comparing normal and reduced render paths.

## 10. Privacy, security, and data boundaries

Privacy rules:

- Email is encrypted and never used as an internal identifier outside lookup hash.
- Per-bird interaction events are used only for that user's simulation.
- No simulation database reads from analytics jobs.
- No per-account or per-bird dimensions in aggregate telemetry.
- No model training on per-bird interaction data.
- Visitors never affect host drift.

Security controls:

- Magic links expire after 15 minutes and are invalidated on use.
- Token hashes only; never store raw magic-link or session tokens.
- Rate limit magic-link requests by email hash and IP bucket.
- CSRF protection for cookie-authenticated writes or use same-site secure cookies with CSRF tokens.
- Session revocation from account settings.
- Visit tokens are hashed, revocable, one-time or session-bound, and expire after 30 days unused.
- Account export links are signed, short-lived, and emailed only to verified address.
- Soft-deleted accounts cannot produce snapshots or receive simulation ticks except deletion/recovery flows.

Logging:

- Log account UUIDs only where needed for operational debugging.
- Do not log email, bird names, notebook prose, interaction payloads, or raw snapshots by default.
- Redact request bodies for event ingestion, auth, export, deletion, and invite endpoints.

## 11. Performance budgets and observability

Budgets:

- Initial JS bundle under 2MB gzipped.
- First bird visible under 500ms on mid-tier mobile over 4G.
- 60fps idle motion on a five-year-old mid-range laptop.
- No client memory growth over 30 minutes.
- Simulation tick p99 alarm above 5 seconds.

Implementation tactics:

- Code-split settings, notebook, visit management, export, and deletion flows.
- Inline or edge-deliver bootstrap snapshot for authenticated aviary route.
- Draw first bird before non-critical assets, notebook code, settings code, and advanced weather effects.
- Use compact assets and procedural rendering where practical.
- Keep renderer state bounded; pool objects for animation and audio.
- Pause rendering when hidden; resume by pulling a fresh snapshot.
- Use service worker only if it does not add bundle or stale-state complexity; static asset caching is useful, snapshot caching is risky.

Observability:

- Synthetic checks from common geographies: first-bird timing, snapshot latency, render availability, unsupported browser surface.
- Aggregate RUM: page load timing, first-bird timing bucket, frame timing bucket, audio context error category, route-level error rates.
- Server metrics: snapshot p50/p95/p99, event ingestion latency, tick duration, tick failures, tick backlog, email send failures, export job latency, hard deletion job completion.
- Privacy-safe dashboards with no account, bird, event-payload, or notebook dimensions.

CI gates:

- Bundle-size budget.
- Performance smoke test for first render on throttled profile.
- 30-minute memory growth test at least nightly.
- Simulation deterministic tests and drift calibration fixtures.
- Accessibility automated tests.

## 12. Rollout plan

### Milestone 0: technical spikes

- Renderer prototype with two birds, three perch zones, day/night palette, and first-frame bootstrap.
- WebAudio grammar prototype with two species and listen-in mixing.
- Simulation tick prototype with event cursor, deterministic seeds, and mood transitions.
- Presence detector prototype across Chrome, Safari, Firefox, mobile Safari.
- Bundle and first-bird timing baseline.

Exit criteria: first bird can draw from snapshot under budget in a controlled prototype; procedural calls are recognizable for at least two birds; presence pings are not generated from hidden/unfocused tabs.

### Milestone 1: account and canonical aviary foundation

- Magic-link auth, sessions, starter account creation.
- Account UUID identity and encrypted email storage.
- Aviary, birds, personality, mood, event log, and snapshot schema.
- Simulation tick lease/cursor.
- Host snapshot endpoint and basic client render.

Exit criteria: user can sign in, receive two starter birds, open on two devices, and see the same canonical state.

### Milestone 2: bird engine v1

- Mood transition system.
- Slow personality drift with calibration fixtures.
- Return-greeting selection.
- Perch selection and pose families.
- Bird-to-bird call response/chorus seeds.
- Offer cooldown and reaction outcomes.
- Settle flow.

Exit criteria: no client writes personality; one week of fixture presence produces measurable drift; a single session does not visibly jump traits; closing without settle is neutral.

### Milestone 3: rendering and audio quality

- Production renderer with responsive scene.
- Idle micro-motion and reduced-motion renderer.
- Procedural call synthesis for initial species pool.
- Listen-in mix ramps.
- Day/night and weather visuals.
- Quiet field loading state.

Exit criteria: first frame appears already in motion; no spinner; 60fps budget passes; WebAudio unavailable path gives captions by default.

### Milestone 4: notebook and accessibility

- Sparse notebook generation.
- Screen-reader narration queue.
- Call captions from grammar.
- Keyboard navigation.
- WCAG contrast verification.
- Account/settings matter-of-fact surfaces.

Exit criteria: screen-reader and reduced-motion users get a designed aviary, not a static fallback; notebook entries are naturalist observations, not event logs.

### Milestone 5: visits, export, deletion, privacy hardening

- Visit invite/revoke/expire flow.
- Visitor read-only route.
- Visit log in account settings.
- Export generation and email link.
- Soft/hard deletion worker.
- Telemetry boundary review and logging redaction.

Exit criteria: visitor cannot write events or generate presence; revoked visit ends at next snapshot; analytics cannot read per-bird simulation data.

### Milestone 6: private beta and calibration

- Start with two-bird aviaries only.
- Invite small beta across desktop/mobile and assistive tech users.
- Instrument aggregate performance and error health.
- Review drift calibration weekly using synthetic fixtures and consenting internal test accounts only, not population per-bird analytics.
- Tune presence window, offer cooldown, mood transition probabilities, call density, and notebook sparsity.

Bird ramp:

- V1 starts all accounts with two birds.
- Enable third-bird age offer after the relevant account age interval only after two-bird quality and audio recognizability hold.
- Gradually test four through seven birds with synthetic audio-recognizability and performance checks before any account can reach those counts.

Launch gate:

- Bundle under 2MB.
- First bird under 500ms target on test profile.
- 30-minute no-memory-growth pass.
- p99 tick latency below alarm threshold.
- Accessibility sign-off.
- Privacy boundary review complete.
- No gamification/social-network leakage in UI copy or telemetry.

## 13. Test strategy

Simulation tests:

- Drift is monotonic toward expressive and never decreases from neglect.
- Presence from hidden/unfocused/no-activity tabs is rejected.
- Listen-in and offers produce small bounded deltas.
- One-week fixture shows measurable internal drift; one-session fixture does not exceed visible threshold.
- Mood persists across sessions and changes through tick/time/weather.
- Return-greeting chooses one primary bird and staggers secondary responses.
- No client event can set personality, mood, or perch directly.

Sync tests:

- Concurrent device events are processed once and in order.
- Duplicate client_event_id is idempotent.
- Stale snapshots reconcile without LWW personality overwrites.
- Session revocation blocks event writes.
- Visit snapshot cannot submit events.

Frontend tests:

- Initial route draws quiet field or first bird, never spinner.
- Snapshot resume after hidden tab pulls fresh state.
- Keyboard interactions match spec.
- Top bar fades and returns without hiding focus from keyboard users.
- Reduced-motion renderer uses cross-fades and removes leaf drift.
- Captions match call grammar.

Performance tests:

- Bundle size gate.
- Throttled first-bird timing.
- 60fps idle scene with seven birds on target hardware profile.
- 30-minute memory stability including calls and notebook panel use.
- WebAudio error fallback.

Privacy/security tests:

- Email never appears in logs, telemetry tags, or IDs.
- Per-bird events are absent from analytics pipeline.
- Magic links expire and single-use consumption is atomic.
- Export link goes only to verified email and expires.
- Hard deletion removes account-tied records after recovery window.

Copy/voice tests:

- Product surfaces use naturalist voice.
- System/error/settings surfaces use matter-of-fact voice.
- No welcome toast, streak, achievement, public discovery, profile, stats, or raw personality copy appears in the UI.

## 14. Risks and mitigations

### Drift calibration feels wrong

Risk: birds change too fast and feel gameable, or too slowly and feel static.

Mitigation: version the drift formula, build fixture-based calibration tests before beta, tune with internal long-running aviaries, and expose changes only through bird behavior and notebook observations. Do not add user-visible stats to make drift legible.

### Presence accounting overcounts idle tabs

Risk: background tabs produce false drift and corrupt the core mechanic.

Mitigation: require visible + focused + recent pointer/key activity client-side, consolidate server-side with heartbeat bounds, test browser edge cases, and reject visitor presence entirely.

### Sync correctness silently loses personality history

Risk: multi-device writes overwrite or duplicate drift.

Mitigation: append-only events, idempotency keys, simulation-only vector writer, event cursor, row locks, current_version snapshots, and tests for stale concurrent sessions.

### Audio feels canned or unpleasant

Risk: procedural calls become repetitive, harsh, or unrecognizable in chorus.

Mitigation: prototype early, keep bird cap at seven, define per-species motif constraints, test listen-in ramps, add audio QA for repetition and phase artifacts, and choose silence plus captions over recorded fallback.

### Accessibility surface becomes a flattened fallback

Risk: screen-reader/reduced-motion users get state labels instead of the product.

Mitigation: ship narration, captions, and reduced-motion renderer as core milestones; include assistive-tech QA in beta; treat naturalist prose generation as product work, not compliance cleanup.

### Performance budget conflicts with visual richness

Risk: assets and code push first-bird render past 500ms.

Mitigation: budget gates from milestone 0, code-split non-aviary surfaces, compact/procedural assets, bootstrap snapshot, and render first bird before non-critical effects.

### Privacy boundary erodes through telemetry

Risk: useful debugging dashboards start ingesting per-bird events.

Mitigation: separate simulation database from analytics, define allowed metric dimensions, redact logs by default, and review every new metric against the privacy commitment.

### Product restraint erodes through small additions

Risk: welcome toasts, badges, counters, visit notifications, or public sharing appear as harmless engagement improvements.

Mitigation: encode non-goals in acceptance criteria, add UI copy review, add automated text scans for forbidden terms where practical, and require product sign-off for any new surface that comments on user frequency or comparative status.

### Notebook prose becomes generic or too frequent

Risk: entries read like event logs or become a feed.

Mitigation: template from observation keys with sparse gates, store final prose, cap frequency, review examples in design QA, and ban user-behavior summaries.

### Visit feature expands into social network mechanics

Risk: read-only visits attract chat, profiles, discovery, or co-presence requests.

Mitigation: isolate visitor route as render-only, omit event APIs from visitor bundle, keep visit logs in settings, default notifications off, and avoid public identifiers beyond named email invites.

## 15. Implementation guardrails

- If a feature requires showing a number to motivate use, it probably violates the product.
- If a client wants to write canonical bird state, it is crossing the architecture boundary.
- If a surface tells the user they are back, it is replacing the bird greeting and should be removed.
- If a metric needs per-bird or per-account interaction history, it is outside allowed telemetry.
- If an accessible mode removes the feeling of the aviary, it is incomplete.
- If a visit can change host state, it is not a visit.
- If absence makes birds worse, sick, sad, or mistrustful, the engine has become a Tamagotchi.
- If adding a bird makes calls indistinguishable, the cap/ramp must hold until audio quality catches up.

This plan should be executed as a quality-first v1: two birds, one scene, one canonical server simulation, and a small set of interactions that remain quiet. The product succeeds when the user feels that the aviary was already there, that a bird noticed them without the system announcing it, and that weeks of gentle attention have changed the birds without ever becoming a thing to optimize.
