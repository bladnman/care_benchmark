# Pocket Aviary v1 implementation plan

## 1. Delivery intent and scope

Build a web-only, single-user Pocket Aviary in which one canonical aviary continues to simulate on the server while a browser renders it as a calm, already-in-progress place. A new account begins with two stable-identity birds and can eventually grow to no more than seven through aviary-age-based availability. The v1 release includes magic-link accounts, a single canonical aviary, server-side simulation, multi-device snapshots, the field notebook, presence accounting, listen-in, offers, settle, local-time day/night, rare ambient weather, read-only named visits, screen-reader narration, reduced-motion rendering, call captions, and the account/privacy controls required by the PRD.

The release deliberately does not include a native client, shared or multi-aviary accounts, payments, customizable scenes, public discovery, profiles, follows, feeds, comments, chat, co-presence, leaderboards, achievements, scores, levels, badges, streaks, visit-frequency dashboards, hunger/health/death/distress mechanics, push notifications, or email notifications by default. No implementation should create an extension point whose default behavior accidentally becomes one of those surfaces.

The core invariants are:

1. The server is the only writer of canonical personality, mood, bird identity, and simulation state.
2. Clients submit facts as append-only interaction events; they never submit absolute personality or mood values.
3. Presence requires visible document + window focus + recent pointer or key activity.
4. Drift is slow and monotonic toward expressive: neglect never decreases personality traits.
5. A visitor is read-only and contributes neither presence nor interaction events to the host.
6. Product surfaces use lowercase, present-tense, specific naturalist prose; account, settings, accessibility, sync, error, and unsupported-browser surfaces use matter-of-fact system language.
7. No user-facing surface exposes personality-vector numbers or turns presence into a score.
8. Calls are procedurally synthesized in the browser. There is no recorded-audio fallback.

## 2. Proposed architecture

### 2.1 Service shape

Start with a modular server application and a separately scheduled simulation worker rather than splitting every domain into independently deployed microservices. The modules have explicit interfaces so the worker, API, export job, and visit surface can be separated later without changing the domain model:

- **Web client and edge bootstrap**: serves the browser shell, critical renderer, small scene assets, and a short-lived current-state bootstrap payload at the edge. It does not calculate simulation outcomes.
- **Identity/session module**: requests and consumes magic links, issues per-device session tokens, tracks revocation, handles verified email changes, and exposes account deletion/export controls.
- **Aviary state module**: owns accounts, aviaries, stable bird identities, canonical birds, moods, weather, simulation revisions, and snapshot reads.
- **Interaction ingestion module**: validates and appends offers, listen-in boundaries, settle events, and presence pings with account/device authentication and idempotency keys.
- **Simulation worker**: claims one aviary at a time, consumes ordered events, advances the ~one-minute tick, updates mood and monotonic personality deltas, generates notebook observations, and writes one new canonical revision transactionally.
- **Invitation/visit module**: creates per-email invitation records, sends the one-time link, redeems it into a read-only visit session, serves host snapshots, records host-visible visit-log facts, and enforces revocation and 30-day unused expiration.
- **Notification/email module**: sends magic links, verified email-change links, exports, and invitations. Visit notifications are disabled by default and are only sent when the host explicitly enables the optional setting.
- **Telemetry gateway**: accepts aggregate operational metrics only. It must not have read access to the simulation database or interaction-event payloads.

Use a relational primary store with transactions and row-level locking for the canonical state and append-only event log. The exact hosting provider is an implementation choice, but the design must provide durable ordering, unique constraints, encrypted-at-rest fields, and a worker lease. A cache/CDN may accelerate snapshots; it is never authoritative. A queue is appropriate for tick scheduling, email, and exports, but a queue message is not the source of truth.

The primary account record stores the email once, encrypted. Every internal reference, log field, event partition key, telemetry dimension, and foreign key uses a generated account UUID. Redact email and invite tokens from application logs. Use short-lived, hashed or otherwise non-reversible token representations for magic-link and invitation lookup where feasible.

### 2.2 Server/client boundary

The server owns identity, time interpretation, simulation, bird positions as semantic perch zones, mood, weather state, call intent, drift, notebook generation, invite authorization, and snapshot revisions. The client owns interpolation between snapshots, frame-level idle poses, ambient leaf/feather ornaments, scene compositing, focus behavior, audio synthesis, caption presentation, and the accessibility narration queue. Client rendering can be paused when hidden; the server tick never pauses.

The snapshot contains enough information to place the first frame in progress: bird stable ID and display name, species visual key, perch zone, pose/motion seed and phase, mood-derived semantic cues, call intent/timing/seed/descriptor, local-time phase, weather state, active offers, settle state, and the canonical revision/server timestamp. It does not contain the raw personality vector. The server may provide bounded presentation parameters derived from personality, but no endpoint or debug payload exposed to the browser should make the vector reconstructable as a stats surface.

The client must treat snapshots as immutable inputs. It interpolates from revision N to N+1, detects revision gaps, and refetches after visibility changes, suspension/long frame gaps, or failed keepalive. If a client is stale, it may finish a visual interpolation but must not author a replacement state.

### 2.3 Consistency and failure policy

Every aviary has a monotonically increasing canonical revision and a simulation cursor for the last consumed event sequence. The worker obtains a renewable lease per aviary, reads events after the cursor in sequence order, computes a deterministic transition from the prior state plus server time and a versioned calibration configuration, and commits state, cursor, notebook entries, and tick metadata in one transaction. A crashed worker can retry safely because event application is guarded by sequence/cursor checks and event IDs are unique.

The API accepts client-generated idempotency keys for every event. Duplicate submissions return the original accepted sequence or a safe duplicate result. Server timestamps, not device clocks, determine event ordering within bounded ingestion rules. An event arriving too late is retained for auditability and applied only if its defined simulation window is still open; it must never cause a client to overwrite a newer state.

Do not use last-write-wins for personality, mood, or the bird record. All personality changes are additive server-authored deltas processed in event order. Snapshot caches are invalidated or versioned by canonical revision. If a client requests a stale revision for a mutation, the server rejects the mutation with a matter-of-fact retry response; it does not merge client state.

## 3. Data model

The following logical records are required. Names are illustrative; schemas should preserve the ownership and invariants.

### 3.1 Account and access records

- `accounts`: `account_id` UUID, encrypted verified email, email-verification status, created/updated timestamps, deletion state/deletion deadline, timezone identifier, and settings version. The email is not an identifier outside this row.
- `device_sessions`: `session_id` UUID, account UUID, hashed token, device label/metadata safe for display, created/last-used timestamps, expiry/revocation timestamps. Account settings can revoke one session without affecting the others.
- `magic_link_requests`: hashed single-use token, account or pending-email reference, issued/expiry/consumed timestamps, request throttling fields. Links expire after 15 minutes and are invalidated immediately on use.
- `email_change_requests`: account UUID, encrypted pending email, verification token hash, expiry/consumed state. The old address remains active until the new address verifies.
- `account_settings`: accessibility preferences, captions preference, timezone, optional visit-notification preference, and other matter-of-fact system settings. No gamification or visit-frequency fields.
- `account_deletion_requests`: account UUID, requested timestamp, hard-delete deadline, recovered timestamp. Restore is allowed during the 30-day soft-delete period; after hard deletion, birds, vectors, notebook, telemetry records, and account-linked data are removed.

### 3.2 Aviary and bird records

- `aviaries`: one row per account with `aviary_id`, account UUID, creation time, local timezone, canonical revision, last tick time, next tick due time, event cursor, current day/night phase, weather state, settle state, and worker lease fields. Enforce one aviary per account.
- `birds`: stable `bird_id`, aviary UUID, species key, user name, adoption timestamp, age-availability metadata, and lifecycle state. Names can change without changing the ID or any history.
- `bird_personality`: one server-only row per bird with normalized scalar values for boldness, social warmth, vocal frequency, plumage saturation, and curiosity, plus calibration/version metadata. No public serializer includes this row. Persist it directly; never rebuild it from events.
- `bird_mood`: current enumerated mood (`wary`, `content`, `curious`, `drowsy`, `alert`, with the final implementation set versioned), mood-entered time, transition reason class, and timer/decay state. Mood persists through sessions and is advanced by ticks.
- `bird_render_state`: current semantic perch zone (`front`, `middle`, `back`), pose/motion seed and phase, call schedule/descriptor, and current interaction reaction. This is presentation state derived and persisted by the tick so a new client can begin mid-action.
- `aviary_events`: append-only event sequence, event ID/idempotency key, account and aviary UUIDs, event type, bird UUID where relevant, server-ingested timestamp, client-observed timestamp for diagnostics only, validated payload, source kind (`host` or `visitor`), and consumed cursor metadata. Visitor interaction events are not accepted; visitor view sessions are separate.
- `presence_windows` or normalized presence events: host account/aviary UUID, start/end or bounded duration, source device session, server timestamps, and validation status. Store enough to drive drift and audit presence logic, but do not export it to aggregate telemetry.

### 3.3 Notebook, weather, and social records

- `notebook_entries`: aviary UUID, entry ID, observation timestamp, naturalist prose, source observation class, and simulation revision. Read-only in all clients; never expose raw event identifiers or numeric trait changes.
- `ambient_events`: aviary UUID, event ID, type (`rain`, `wind`, or future bounded event), start/end, intensity bucket, seed, and state transition version. Weather is rare, brief, quiet, and mood-relevant rather than a feature dashboard.
- `invitations`: invitation UUID, host account/aviary UUID, specifically named visitor email encrypted for host display, token hash, created/expiry/redeemed/revoked timestamps, and status. New invites are opt-in; unused invites expire after 30 days and cannot be revived.
- `visit_sessions`: visit session UUID, invitation UUID, host aviary UUID, redeemed timestamp, last snapshot pull, approximate duration accumulator, and terminated/revoked status. Visitors receive no host mutation authority and no host presence credit.
- `visit_log_entries`: host-visible visitor email/date/approximate duration and invitation association, ordered by recent visit. No settings badge is generated. A host may inspect the log on demand.
- `export_jobs`: account UUID, requested/completed/expired timestamps, object key, and delivery status. Export links are authenticated/short-lived and include birds, names, current personality vectors, moods, notebook entries, and account settings as required by the PRD.

Retain per-account simulation history only for the user's simulation and account controls. Do not copy it to an analytics warehouse, model-training dataset, recommendation store, leaderboard table, or cross-account aggregate. Hard deletion must cover canonical rows, event history, notebook, vectors, invite/visit records, export artifacts, and any account-linked telemetry that can identify the account.

## 4. API surface

All JSON APIs are versioned under `/v1`, authenticated with server-issued device session tokens except public magic-link request and invitation redemption. Responses include matter-of-fact error codes and actionable text for system surfaces; product observations remain naturalist prose.

### 4.1 Identity and account

- `POST /v1/auth/magic-link`: accept an email, apply per-email rate limits, create a 15-minute single-use request, and send the link. Do not reveal whether an account already exists.
- `POST /v1/auth/magic-link/consume`: consume once, create or resume the single-user account, issue a per-device session, set the account timezone from an explicit client/browser value subject to user settings, and create two starter birds transactionally for a new account.
- `POST /v1/auth/logout` and `GET /v1/account/sessions`: end or list this device's sessions; `DELETE /v1/account/sessions/{id}` revokes a selected device session.
- `POST /v1/account/email-change` and `POST /v1/account/email-change/verify`: verify the new email before switching.
- `POST /v1/account/export`: enqueue an export and email a download link to the verified address.
- `POST /v1/account/deletion` and `POST /v1/account/deletion/recover`: start or recover the 30-day soft deletion. A scheduled hard-delete job enforces the deadline.
- `GET/PATCH /v1/account/settings`: read and update accessibility, captions, timezone, and the optional visit-notification setting. The API must reject any attempt to create unsupported gamification settings.

### 4.2 Canonical aviary and notebook

- `GET /v1/aviary/snapshot`: return the current canonical revision and render-safe semantic state. Support `If-None-Match`/revision conditional reads and a small edge/bootstrap variant for first paint. The response includes the account's host view only.
- `GET /v1/aviary/notebook`: return paginated, newest-first or explicitly scrollable read-only entries, with stable cursor pagination for indefinite history. It never returns raw event logs or numeric drift.
- `POST /v1/aviary/events`: accept only a closed event union: `listen_in_started`, `listen_in_ended`, `offer_submitted`, `settle_started`, `settle_undone`, and validated `presence_ping`/window closure. Require an idempotency key and enforce bird ownership, cooldowns, settle timing, and payload limits. `presence_ping` must include client visibility/focus/activity evidence; the server treats it as a claim to validate, not a blind duration write.
- `POST /v1/aviary/presence/close`: close a host presence window on settle, page lifecycle, or explicit keepalive expiry. It is safe to repeat. Browser lifecycle is best-effort; absence of a close is resolved by server timeout, never penalized.

The offer endpoint must not accept arbitrary content. It accepts one of `seed`, `song_fragment` from a server-defined small motif library, or `still_pool`, and the server chooses the receiving bird/reaction context. A per-bird few-minute cooldown is enforced server-side. The client may optimistically animate a pending gesture but must reconcile to the next snapshot.

### 4.3 Invitations and visits

- `POST /v1/visits/invitations`: host supplies a visitor email; server creates a per-invite token, sends a one-time link, and returns an invitation status without exposing a global discoverability model.
- `GET /v1/visits/invitations`: list outstanding invitations for the host, including expiration and revocation state.
- `DELETE /v1/visits/invitations/{id}`: revoke an outstanding or active invitation immediately. The next visitor snapshot pull terminates the session.
- `GET /v1/visits/log`: host-only, on-demand log of visitor email, date, approximate duration, and outstanding invites. No badge or automatic in-product notification.
- `POST /v1/visits/redeem`: consume an unexpired invitation link into a constrained read-only visit session. Expired or revoked links return the same matter-of-fact unavailable surface.
- `GET /v1/visits/{session}/snapshot`: return the host's current render-safe snapshot, including calls, day/night, and weather, with no mutation capability. Do not ingest presence or interaction events for this session.

The host snapshot path and visitor snapshot path should share a snapshot serializer and revision semantics, with authorization determining which fields are safe, so visitors see the real aviary rather than a special show-off rendering. A revocation check occurs on every snapshot pull; a websocket is not required for correctness.

## 5. Simulation engine

### 5.1 Tick lifecycle

Run a scheduled tick at approximately one-minute cadence per aviary, with jitter and catch-up limits. A worker claims due aviaries in batches, but each aviary's state transition is serialized. A tick should:

1. Acquire the aviary lease and read the canonical state, calibration version, local timezone, current weather, and unconsumed events.
2. Normalize accepted host events into bounded signals: presence duration, bird-specific listen-in duration, offer type/target, settle, and close/timeout markers.
3. Advance local-time phase, ambient event timers, mood timers, call schedules, perch selection, bird-to-bird response opportunities, and active reactions for elapsed time.
4. Apply personality deltas using only positive eligible inputs, clamped to configured ranges and daily/session caps.
5. Generate sparse notebook observations only when a noteworthy observation rule fires, subject to per-aviary sparsity limits.
6. Produce the next canonical snapshot/render state, increment revision, advance the event cursor, and commit all changes atomically.
7. Emit aggregate duration/latency/error metrics without account or bird dimensions, release the lease, and schedule the next tick.

If a tick is delayed, advance elapsed-time transitions using bounded time steps and record operational latency. Do not replay arbitrary client time. If the worker cannot safely determine order or encounters a malformed event, quarantine that event, preserve canonical state, alert operationally, and continue with safe events; never reset a bird.

### 5.2 Personality and drift

Represent each personality trait as a normalized scalar with a versioned seed/configuration and server-only storage. Implement the low-pass behavior as additive, non-negative deltas. One workable form is:

`delta_trait = clamp(alpha_trait * positive_signal * elapsed_presence / calibration_unit, 0, per_tick_cap)`

followed by `trait = min(trait_max, trait + delta_trait)`. The exact alphas, trait maxima, starter seeds, and calibration units belong in a versioned simulation configuration, not hard-coded in client bundles. A change to the formula or coefficients is a migration/calibration decision and must not silently reinterpret historical events.

Weight signals in this order:

- Host presence-time is dominant and can influence expressive behavior across the aviary.
- Listen-in is a strong bird-specific signal affecting that bird's social warmth and vocal frequency.
- An accepted offer gives a small curiosity signal; making an offer near a bird gives a small boldness signal.
- Settle quiets mood and closes presence but does not increase or decrease personality.
- Neglect/absence contributes no negative personality delta. It can result in a quieter current mood or fewer observed greetings through normal mood/time behavior, but it never makes traits less expressive.

The calibration harness must simulate regular visits, irregular visits, long absence, repeated offers, and multiple devices. It must show measurable instrument drift after roughly one week of regular visits and user-perceivable but not session-by-session drift after roughly three weeks. Add explicit regression assertions that a two-week absence cannot lower any personality trait and that one session cannot produce a visibly large change. Never expose these numbers in the product UI.

### 5.3 Mood, time, weather, and bird relationships

Use a small, versioned mood state machine for `wary`, `content`, `curious`, `drowsy`, and `alert` (final enumerated set can be adjusted during calibration). Transitions combine recent host interactions, local time of day, active weather, neighboring calls, current mood duration, and personality-derived resistance. Persist the mood at the end of every tick; do not reset it on tab open. Daily-ish cadence means the state can soften or change while the user is away as the server crosses time phases.

Make the local timezone an account setting that can be changed through settings. Apply the current timezone to day/night and mood signals on the next tick with a clear, deterministic transition; do not infer a new timezone on every request. At night most species settle with low activity, while the nightjar-like species remains eligible for occasional calls.

Keep weather as a server-scheduled rare ambient event with a small seed and intensity. Rain temporarily damps vocal frequency; wind makes some birds alert and others wary. Weather ends automatically and does not become a weather history or dashboard. Bird-to-bird call response and wary spread are bounded per tick so a single alarm cannot cascade into permanent distress.

Select greeting bird, perch zone, call timing, and reaction from stable bird identity, mood, boldness, absence length, local time, and a server seed. Stagger multiple greetings by a small random offset. Absence changes the form of the greeting but does not create a guilt or penalty state.

### 5.4 Call grammar runtime contract

Define a versioned motif grammar per species: recognizable signature, bounded motif choices, pitch/timing ranges, envelope, pause patterns, and a prose caption descriptor. Personality and mood influence timing, pitch, repetition, and chorus participation without erasing the species/bird signature. The server schedules call intents and supplies deterministic seeds/grammar version; the client synthesizes the actual waveform. This allows two browsers to render the same semantic state while retaining safe client-side variation.

The caption descriptor must be generated from the same call intent/grammar selection as the audio, for example “a soft three-note rise” or “a low trill, paused, low trill again.” Never store a fixed caption list disconnected from the sound actually played. Call schedules may be interpolated visually/audibly between snapshots, but the server remains authoritative about semantic event timing.

## 6. Frontend rendering pipeline

### 6.1 Composition and initial frame

Use a single horizontal scene with a scene renderer (Canvas 2D or equivalent GPU-backed layer) plus semantic DOM controls/focus targets above it. Keep the renderer boundary explicit: it consumes a typed `AviarySnapshot` and local render clock, and returns only pixels/scene effects; it does not call APIs or mutate domain state. The DOM layer owns the thin top bar, focusable bird targets, captions, narration region, dialogs, and settings.

Compose background sky/foliage, subtle background/middle/foreground planes, three perch zones, bird layers, and occasional client-only leaf/feather ornaments. Keep parallax gentle. Birds remain visible at every responsive width; compress the scene and preserve aspect relationships rather than cropping, scrolling, or zooming. A bird's perch is a server-selected signal, never a drag target.

Bootstrap the first snapshot with HTML/edge delivery and a minimal critical renderer. Draw the quiet field only when state is not yet available; no spinner, generic loading animation, or fade-from-static. As soon as the snapshot arrives, place birds with non-zero pose phases and ambient motion already underway. The post-adoption empty state is a quiet field only until the first starter birds enter; after that, a normal aviary never renders empty. Non-critical account, invitation, notebook, and accessibility settings code is split from the initial bundle.

### 6.2 Frame loop and transitions

Maintain a bounded local render clock and interpolate server semantic positions, transitions, and call timing. Use deterministic per-bird idle micro-motion selected from mood: wary scanning/back-perch weight, content preening, curious head tilt, drowsy fluffed low posture, alert attention. Idle animation continues while visible even when no new snapshot arrives. Client-only ornaments have bounded lifetimes and pooled objects.

When hidden, stop `requestAnimationFrame` and audio scheduling work that cannot be heard, retain the latest snapshot, and request a fresh snapshot on visibility/focus return. On a long frame gap or system suspend, discard stale interpolation and resume from a current server snapshot. A state transition such as a settle uses a reversible client presentation state keyed to the server event; any click in the five-second settle undo window submits/records the undo and reverses the lighting shift.

The top bar contains only account/settings, accessibility, notebook, and offer controls. Fade it nearly transparent after cursor stillness, restore it on pointer/keyboard activity, and keep all controls keyboard reachable. No scene badges, hover tooltips, inline labels, status counters, or overlay icons are permitted.

Reduced motion is a separate renderer mode selected from `prefers-reduced-motion` or an explicit setting. Replace frame-by-frame micro-motion and flight paths with slow cross-fades between still poses; remove leaf drift; retain but slow color/day-night transitions. Mood, calls, captions, notebook, server simulation, and bird identity remain unchanged.

## 7. Audio pipeline

Create one bounded WebAudio graph per active browser session: per-bird call gain nodes into a chorus/ambient bus, a focused-bird/listen-in gain stage, and a master output. Use pooled oscillator/noise/envelope nodes or scheduled reusable buffers and disconnect/release them after each call. There must be no per-call unbounded allocation and no recorded audio asset path.

Synthesize calls from the species motif grammar, bird signature seed, mood, personality-derived bounded parameters, and call seed. Two or more simultaneous calls should mix as independently generated calls with recognizable signatures. The listen-in interaction applies ramped gain changes over a perceptual short fade: raise the focused bird, lower other birds to ambient, never hard-mute them. Clicking the focused bird, focusing another bird, clicking empty scene space, or moving keyboard focus away ends listen-in and ramps back to the ambient mix.

Unlock/resume the audio context from the first permitted user gesture and show no intrusive permission announcement. If WebAudio is unavailable, denied, or errors, switch to graceful silence and enable captions by default; do not download or play a recorded fallback. Offers use the same procedural path for the song-fragment motif. Test chorus density from two through seven birds and assert recognizability and CPU stability at the cap.

Captions are created from the actual call descriptor, positioned near the calling bird, and faded in/out with that call. They remain accessible to keyboard and screen-reader users without making every ambient call an interruptive announcement. Audio settings use matter-of-fact language; captions and call descriptions retain the naturalist voice.

## 8. Accessibility and content surfaces

- Build a semantic DOM representation of bird focus targets in scene order. Tab enters the first bird, arrow keys move among birds, Enter starts listen-in, Escape exits it, and focus movement away ends it. The offer and settle controls have visible names and keyboard paths.
- Provide a throttled `aria-live` narration region generated from the same snapshot/interactions as the visual renderer. At idle, produce one naturalist prose observation approximately every 30–60 seconds; prioritize a return-greeting, successful offer reaction, and settle event, while coalescing updates so the screen-reader queue cannot flood. Do not expose a state table, perch index, mood label, or personality vector as the primary experience.
- Show the same narration visually when the accessibility surface calls for it, with lowercase, present-tense, specific prose. Keep all system errors, authentication, settings, sync, unsupported-browser, and revocation surfaces capitalized/direct and actionable.
- Generate call captions at runtime from the call grammar and make them available when audio is off or fails. Ensure captions and narration do not collide with focus indicators or obscure birds.
- Meet WCAG AA contrast for all user-copy text, captions, top-bar labels, settings, account pages, errors, and focus indicators in both bright and dim/settled scenes. Focus treatment must remain visible against all day/night/weather palettes.
- Test keyboard-only, screen-reader, reduced-motion, no-audio, zoomed/narrow viewport, high-contrast/forced-colors where supported, and touch paths. Verify that visitor mode remains read-only and has an accurate matter-of-fact unavailable state after revocation.

## 9. Performance, observability, and privacy controls

Enforce these release gates:

- Initial JavaScript bundle is under 2 MB gzipped.
- First bird is visible in under 500 ms on the defined mid-tier mobile/4G test profile, using the bootstrap snapshot and critical renderer.
- Idle motion sustains 60 fps on a five-year-old mid-range laptop for a 30-minute session.
- A 30-minute session has no meaningful client memory growth; audio nodes/buffers, workers, and scrolled notebook references are bounded and released.
- Simulation-tick latency p99 stays below five seconds; alert when it crosses that threshold.
- Supported browsers are the last two major versions of Chrome, Safari, Firefox, and Edge. Older browsers receive a clear unsupported-browser page rather than a compatibility bundle.

Measure synthetic browser checks from representative geographies and aggregate-only RUM for navigation timing, first-bird timing, frame timing buckets, audio-context errors, snapshot failures, and tick latency. Remove account UUID, bird ID, email, event type, personality, mood, notebook prose, and interaction history from aggregate telemetry. Do not use per-bird events for model training, recommendations, rankings, product analytics, or cross-account population analysis. Keep telemetry ingestion physically/permission-wise separate from the simulation store.

Add CI and staging checks for: first-paint timing, bundle budget, frame budget, memory stability, WebAudio error fallback, snapshot revision gaps, duplicate events, tick retries, presence conjunction correctness, monotonic drift, hard-delete coverage, invitation revocation, visitor non-mutation, and accidental numeric personality serialization. Include automated scans that reject gamification terms/components and reject email/account identifiers in logs or telemetry payloads.

## 10. Rollout and operational plan

### Milestone A: domain contracts and simulation fixtures

Define versioned snapshot/event schemas, stable IDs, account and bird invariants, calibration configuration, event idempotency, timezone semantics, and worker lease behavior. Build deterministic fixture accounts for two birds, multi-device overlap, long absence, weather, time transitions, duplicate events, and deletion. Establish invariant tests before UI work.

### Milestone B: first-paint scene and bird engine

Ship the critical renderer, quiet-field bootstrap, three-zone scene, stable bird identity, server tick, mood transitions, perch/pose state, procedural call grammar contract, and client audio graph. Validate two birds at first, then test the same renderer at seven. Calibrate greeting variation, absence-length behavior, idle motion, and three-week drift with human review plus instrumented fixture timelines.

### Milestone C: host interactions and notebook

Add precise presence accounting, listen-in ramps, offer types/cooldowns/reactions, settle/undo, notebook generation and pagination, day/night, rare weather, and screen-reader/reduced-motion/caption surfaces. Verify absence and tab-close are neutral at the engine level and that no session-level action visibly rewrites personality.

### Milestone D: identity, sync, privacy, and visit path

Add magic links, session revocation, verified email change, export, soft/hard deletion, multi-device snapshot behavior, invite redemption/revocation/expiration, read-only visitor sessions, silent host visit log, and the explicitly off-by-default optional visit-notification setting. Test every visitor path against host-state mutation and presence leakage.

### Milestone E: performance and release hardening

Run last-two-major-version browser tests, mobile 4G first-paint tests, long-running memory/frame tests, WebAudio permission/error tests, accessibility audits, deletion/privacy audits, worker load tests, and chaos tests for snapshot gaps and retries. Gate release on the numerical budgets and the qualitative rule that the first frame is already in motion.

### Bird-count ramp

Launch all new accounts with exactly two starter birds. Keep the availability schedule age-based and server-controlled, not tied to visits, interaction counts, scores, or payment. Implement the third-bird and later offers behind a configuration flag so the team can validate seven-bird CPU/audio/recognizability capacity before broadening availability. A staged ramp may begin with the third bird only after the configured aviary-age interval, then extend the same age-based schedule toward five, six, and seven. The UI should describe an available bird as a quiet offer, never as a reward, milestone, counter, or achievement. Do not expose the ramp as progress.

### Day-one operations

Alert on tick p99, worker lease starvation, snapshot error rate, auth-link delivery/consume errors, visitor revocation lag, first-bird budget regressions, audio-context error spikes, memory/frame regression, and deletion-job failures. Keep on-call dashboards aggregate and operational. Provide a runbook for replaying a quarantined event, rebuilding a snapshot cache from canonical state, and restoring a soft-deleted account, with no client-side state repair command.

## 11. Risks and mitigations

### Drift calibration is too fast, too slow, or accidentally negative

Mitigate with a versioned formula, synthetic timelines, one-session caps, one-week/three-week calibration gates, explicit long-absence non-regression tests, and server-only additive deltas. Do not tune from aggregate user behavior because per-bird interaction history is private and not an analytics dataset. Use human review of before/after snapshots without exposing trait numbers in product.

### Multi-device events are lost or applied twice

Mitigate with append-only server sequencing, unique event IDs/idempotency keys, cursor-based tick transactions, leases, retry-safe jobs, revisioned snapshots, and overlap tests where laptop and phone submit events against stale snapshots. Never use client absolute state or last-write-wins.

### Audio feels canned, uncanny, or overwhelms the chorus

Mitigate with stable per-bird signatures plus bounded procedural variation, deterministic grammar descriptors, ramped listen-in mixing, no hard mutes, node pooling, and listening tests at two through seven birds. Prefer silence plus captions to a recorded fallback when the audio context fails.

### The first-frame aliveness promise is broken by loading or hydration

Mitigate with an edge/bootstrap snapshot, a sub-2MB critical bundle, non-blocking noncritical code, immediate non-zero pose phases, and a quiet field instead of a spinner. Test cold cache on mobile 4G, not only warm desktop loads.

### Accessibility becomes a later fallback

Mitigate by making narration, captions, reduced-motion rendering, focus navigation, contrast, and no-audio behavior release-blocking deliverables from Milestone B onward. Review prose for naturalist specificity and system/error voice separately.

### Presence is inflated by background tabs, idle machines, or visitors

Mitigate with the server-validated conjunction of visibility, focus, and recent pointer/key activity, bounded pings, timeout closure, explicit host/visitor source types, and tests for hidden/background/minimized windows. Visitor sessions never create presence or interaction events.

### Privacy leaks through identifiers, exports, logs, or telemetry

Mitigate with a synthetic UUID boundary, encrypted email-at-rest, redacted tokens, separate telemetry permissions, schema-level telemetry allowlists, deletion coverage tests, and export authorization. Treat accidental email in a log or aggregate metric as a release-blocking defect.

### Feature creep turns a quiet aviary into a game or social network

Mitigate with a product-surface review checklist: no scores, streaks, badges, levels, profiles, public discovery, chat, comments, co-presence, or notifications by default; no visit-frequency language in notebook entries; no stats panel; no custom scene. Reject any metric or table whose only purpose is to support one of those future surfaces.

### Tick load or pathological weather/call cascades harms continuity

Mitigate with per-aviary leases, bounded catch-up, capped event batches, bounded mood propagation, rare scheduled weather, aggregate latency alarms, and safe quarantine behavior. A slow or malformed input must preserve the last canonical state and stable bird identities rather than reset or regenerate birds.

## 12. Definition of done

The release is complete only when a new account can sign in, meet two named birds already moving, return later to a server-advanced canonical aviary, watch presence affect slow expressive drift without any visible stats, use listen-in/offers/settle, read sparse specific notebook observations, and see the same state from another signed-in device. A specifically invited visitor can view the real host aviary read-only and can be revoked without affecting the host. Screen-reader narration, captions, keyboard operation, reduced-motion rendering, and WebAudio silence fallback are first-class experiences. All numerical performance, memory, accessibility, privacy, sync, and deletion gates pass. No product code or surface implements the excluded game, pet-care, native, or social-network mechanics.
