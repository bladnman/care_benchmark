# Pocket Aviary v1 — implementation plan

## 1. Delivery intent and product guardrails

Build Pocket Aviary as a browser-only, single-account-per-aviary experience whose primary interaction is attentive observation. The v1 technical design must make the aviary feel continuous even when no browser is open: the server owns canonical simulation state; browsers render a current snapshot and submit bounded interaction evidence. Every implementation choice should protect the product's three essential qualities:

- **Continuity rather than performance.** The first useful frame is an already-living aviary, birds have stable identities, calls and greetings are varied, and absence never creates punishment.
- **Noticing rather than announcing.** The product has no welcome toast, progress UI, streak, badge, notification loop, or generic event log. Product prose is specific, lowercase naturalist observation; account, error, sync, and accessibility-setting surfaces are direct and matter-of-fact.
- **A private relationship rather than an engagement system.** Personality data is hidden from product UI, interaction history only drives the owner's simulation, visitors are read-only and cannot affect it, and telemetry is aggregate operational telemetry only.

### V1 boundary

Ship two starter birds, gradual age-gated additions up to seven, a single unpannable horizontal aviary, email magic-link accounts, server-side simulation, multi-device snapshot sync, presence accounting, listen-in, three offers, settle, the read-only notebook, optional private email invitations, narration, call captions, and reduced motion.

Explicitly exclude native apps, payments, public discovery, profiles, follows, comments, chat, shared/co-present aviaries, multi-aviary accounts, scene customization, user-controlled bird placement, recorded-audio fallback, push/email engagement prompts, gamification, and caretaker mechanics. Do not add hidden counters, visit-frequency reporting, achievement analytics, hunger/health state, or a trait/status panel as an internal shortcut that can leak into the UI.

## 2. Target architecture and service boundaries

Use a modular web application with a versioned HTTP API. A single deployable backend can initially contain the modules below behind clear interfaces; separating them into independently deployed services is deferred until scale requires it. The database is the authority for account, aviary, bird, event, invitation, and notebook records. The server simulation worker is the only actor allowed to mutate canonical bird personality or mood.

| Boundary | Ownership and responsibilities | Explicit exclusions |
| --- | --- | --- |
| Web client | Authenticate, fetch render snapshots, interpolate and draw the scene, synthesize calls, collect qualified presence, submit intent events, render accessible surfaces. | Never derive or persist canonical personality/mood; never tick simulation; never write a trait value. |
| API/auth module | Magic-link issuance/consumption, per-device session tokens, account settings, export/deletion lifecycle, authorization, rate limits. | No email as public/internal entity key; no product-tone error copy on system surfaces. |
| Aviary query module | Build compact owner/visitor render snapshots from canonical records; enforce invitation scopes; return stable version/ETag. | No client-specific mutation or visitor presence recording. |
| Interaction ingestion module | Validate, de-duplicate, append owner events; accept bounded presence intervals; make event receipt durable before acknowledgment. | No direct personality update or client-calculated drift. |
| Simulation worker | Process event log in order, run one-minute canonical ticks, update mood/personality/state projections, create sparse notebook candidates. | No client render loop; no analytics export of per-account data. |
| Invitation module | Issue, redeem, expire, revoke, and audit specific email invitations; write silent visit-log records. | No public listing, mutual access, chat, or notification by default. |
| Observability pipeline | Aggregate performance/error metrics and synthetic probes. | No per-bird state, raw event log, email, or account-level interaction history. |

Run API processes statelessly. Use a relational primary store for invariants and ordered transactions; use a durable job/queue mechanism for tick scheduling, mail, export generation, and notebook generation. Store generated exports in short-lived encrypted object storage. Put cacheable shell and initial snapshot delivery at the CDN edge, but protect all authenticated snapshot responses with account-appropriate cache controls and authorization.

### Authoritative state versus render projection

Maintain two forms of state deliberately:

1. **Canonical domain state** is durable server data: bird identity, personality vector, mood, simulation clock/cursors, invitations, and account settings.
2. **Snapshot projection** is a small, versioned render document assembled from canonical state: perches, current/target poses, call schedule/seed, weather, light phase, settle state, narration/caption inputs, and notebook delta cursor. It may be cached or recomputed, but is never the source of truth.

The projection gives a newly opened client enough information to paint birds in media res without waiting for noncritical UI. The client uses a deterministic local animation seed derived from opaque server-provided seeds and snapshot version, so a reconnect does not present a visibly reset scene. It may interpolate visual state between pulls, but it must discard interpolation and request a fresh snapshot after visibility restoration or a long frame gap.

## 3. Domain model and storage design

Use synthetic UUIDs for all primary business references. Store email only on the encrypted account record and never in logs, telemetry dimensions, event keys, queue partition keys, or URLs. Make IDs opaque to clients.

### Core records

| Record | Key fields | Invariants |
| --- | --- | --- |
| `account` | `account_id`, encrypted `email`, verified time, lifecycle (`active`, `pending_delete`, `deleted`), deletion deadline, settings version | One active aviary per account; only verified email may receive exports/invites/magic links. |
| `device_session` | `session_id`, `account_id`, issued/last-used/revoked timestamps, device display metadata | A revoked session is rejected on every authenticated API. |
| `magic_link` | opaque digest, `account_id`/pending-email reference, issued/expiry/consumed time | 15-minute TTL; one successful consumption; request rate limiting keyed safely without exposing email. |
| `aviary` | `aviary_id`, `account_id`, creation time, simulation cursor, state version, local-time-zone policy, current weather/light/settle state | Exactly one per account; server tick progresses it even with no active sessions. |
| `bird` | stable `bird_id`, `aviary_id`, species id, user name, adopted time, current mood/mood expiry, perch/pose state, hidden vector, call signature seed | Identity survives rename, sync, migrations, and species-pool changes; count is 2 at creation and never exceeds 7. |
| `personality_vector` | `bird_id`, `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`, update version | Server-only values; bounded normalized range; non-decreasing drift deltas only. |
| `interaction_event` | sortable server sequence, event UUID/idempotency key, `aviary_id`, optional `bird_id`, type, server receipt time, client observed time, payload, source session | Immutable append-only log; only authorized owner sessions generate simulation-affecting events. |
| `presence_interval` | event linkage, qualifying start/end/server receipt bounds, evidence state, capped duration | Counts only qualified owner presence; deduplicated/merged by server rules; visitors never create one. |
| `simulation_run` | aviary/tick id, input sequence range, simulation instant, code/config version, resulting state version | One committed result per aviary/tick input range; retry-safe. |
| `notebook_entry` | entry id, `aviary_id`, occurred time, observation category, rendered prose, source state/event references | Auto-generated, read-only, sparse, indefinitely retained until account hard deletion. |
| `invite` | invite id, host account/aviary, encrypted invitee email, opaque one-time token digest, issued/expiry/revoked/redeemed fields | Per-invite opt-in; expires unused after 30 days; revocation makes future pulls fail. |
| `visit_log` | host aviary, invite id, visitor email reference, start/end/approximate duration | Host-visible only on demand; no badge or default notification. |

Account settings include time-zone preference/last confirmed local zone, caption preference, reduced-motion override (`system`, `reduce`, `full`), audio preference, visit-notification opt-in defaulting false, and accessible UI choices. Keep the user's actual display timezone as client-provided context subject to validation; persist enough information to make the day/night mapping coherent across devices without making raw location a requirement.

### Event schema and idempotency

Give every client mutation a UUID `event_id` and a per-session monotonic sequence. The ingestion endpoint atomically enforces uniqueness on `(aviary_id, event_id)` and returns the original receipt for retried requests. Event types are deliberately narrow:

- `presence_window`: qualified start/end plus evidence transitions, capped to a short reporting period;
- `listen_in_started` and `listen_in_ended`: target bird and duration boundary;
- `offer_presented` / `offer_resolved`: target bird, offer kind (`seed`, `song_fragment`, `still_pool`), cooldown authorization and simulation-selected result;
- `settle_started` / `settle_reversed` / `presence_ended`;
- server-authored `return_greeting`, `ambient_weather`, `mood_transition`, `notebook_observation`, and `adoption_offer` records for explainability and replay.

Client timestamps are advisory and clamped relative to server receipt to prevent impossible intervals. Server sequence and receipt time order all simulation processing. Preserve raw owner event data only for simulation/privacy-retention requirements; route no event payload into analytics.

## 4. Public API and authorization surface

Document all endpoints with an OpenAPI contract, strict JSON schemas, error codes, and versioned snapshot/event formats. The client should be able to recover with a full snapshot after any schema/version mismatch. All system errors use matter-of-fact text and accessible remediation.

| API | Caller | Contract |
| --- | --- | --- |
| `POST /v1/auth/magic-links` | unauthenticated | Accept email, apply rate limits, send 15-minute link. Always use non-enumerating response semantics. |
| `POST /v1/auth/magic-links/consume` | link holder | Atomically consume one-time link and issue a revocable device session. |
| `GET /v1/aviary/snapshot` | owner session | Return compact current canonical render projection, `stateVersion`, server time, ETag, and delta/notebook cursor. Supports conditional GET. |
| `GET /v1/aviary/snapshot?sinceVersion=` | owner session | Return a compact delta only when compatible; otherwise return full projection. |
| `POST /v1/aviary/events` | owner session | Batch small idempotent intent events; validate schema, ownership, cooldowns, and timestamps; return accepted sequence numbers/current state hint. |
| `GET /v1/notebook` | owner session | Cursor-paginated, immutable entries, newest first/scrollback indefinitely. |
| `PATCH /v1/birds/{birdId}` | owner session | Rename only; cannot expose/update personality, mood, perch, or species. |
| `POST /v1/aviary/adoptions/accept` | owner session | Accept only a server-issued age-gated offer while capacity remains; supports naming; no catalog/rarity/score path. |
| `GET/PATCH /v1/account/settings` | owner session | Read/update explicit settings, including captions/reduced motion/audio and optional visit notification preference. |
| `GET /v1/account/sessions`, `DELETE /v1/account/sessions/{id}` | owner session | Show/revoke device sessions. |
| `POST /v1/account/export` | owner session | Queue a current JSON snapshot export and email a short-lived verified-address download link. |
| `POST /v1/account/deletion`, `POST /v1/account/deletion/cancel` | owner session | Start 30-day soft delete / restore before deadline. |
| `POST /v1/invites`, `GET /v1/invites`, `DELETE /v1/invites/{id}` | host session | Create named email invite, list outstanding/revoked/active invite state, or revoke immediately. |
| `POST /v1/visits/consume` | invite holder | Redeem one-time invite token into a constrained visitor session. |
| `GET /v1/visits/snapshot` | visitor session | Read-only host render projection. On revoked/expired invite, return terminal matter-of-fact availability response. |
| `GET /v1/visits/log` | host session | On-demand visit log and outstanding invitations only; no unread count. |

Visitors receive a separate, least-privilege token with a host aviary read scope. The API denies all owner event and account endpoints for that token. `GET /visits/snapshot` has its own short pull interval and checks invite revocation/expiry on every request, so an active visitor is terminated no later than the next snapshot poll. Visit duration is calculated server-side from visitor snapshot activity; it never enters host presence or simulation events.

## 5. Simulation engine

### Tick execution and correctness

Run an account-partitioned simulation tick roughly every minute. Use a scheduler that can retry missed work and a transactional per-aviary lease/advisory lock so only one worker commits a tick for an aviary at a time. For each run:

1. Read the aviary's committed simulation cursor and lock the aviary/version row.
2. Load unconsumed events in server sequence order up to a defined cutoff, plus current bird state and relevant time/weather state.
3. Evaluate day/night, rare deterministic weather, qualified presence, offer/listen-in/settle effects, and bird-to-bird signals.
4. Compute mood transitions and additive, clamped personality deltas.
5. Derive current perch/pose/call timing/render seed and evaluate sparse notebook candidates.
6. Atomically update canonical records, append server-authored derived events, advance cursor/state version, and release the lease.

Model each tick as a deterministic pure calculation over `(prior canonical state, ordered event range, tick instant, configuration version, seeded ambient randomness)`. Store the chosen seed/config version and input sequence bounds in `simulation_run`, making retries safe and enabling fixture replay. Never replay a tick by recomputing from an unbounded event history; durable vector/mood state is the baseline, and the cursor makes the calculation incremental.

If the scheduler is late, advance using bounded catch-up windows rather than writing a burst of visibly separate changes. The client only sees the final coherent state plus a naturally interpolated projection. Alert when tick p99 exceeds five seconds; put failed jobs in a retry/dead-letter workflow with protected operator access, then replay idempotently from the recorded cursor.

### Personality drift model

Treat the vector as five bounded values in `[0, 1]` with server-owned initial seeds chosen by species/individual variation. The exact coefficients are configuration, not code constants, and ship behind a calibration version. For each trait, calculate a small positive delta from exponentially decayed recent evidence:

```text
signal = w_presence * qualified_presence_minutes
       + w_listen[trait] * listen_in_duration
       + w_offer[trait, offer_kind, outcome] * accepted_offer_signal
       + small social/ambient modifiers

delta = trait_rate * saturating(signal) * elapsed_tick_factor
next_trait = min(1, prior_trait + max(0, delta))
```

Presence is dominant. A qualified interval exists only while the document is visible, browser window is focused, and a pointer move or keypress occurred inside a calibrated several-minute activity window; any false condition closes/pauses the interval. The client reports state changes and periodic heartbeats rather than a single end-of-session total. The server caps/report-clamps intervals, rejects overlapping impossible sequences, and joins contiguous qualified slices. Watching without movement remains valid for the configured window; a background tab, unfocused window, or idle laptop does not generate drift.

Map interaction signals narrowly: listen-in contributes chiefly to the selected bird's social warmth and vocal frequency; offering near a bird makes a small boldness contribution; an accepted/relevant offer makes a small curiosity contribution; sustained presence can increase all expressive traits with species variation; plumage saturation follows sustained positive presence. Settle affects immediate mood/scene state and terminates the presence window, not trait score. Neglect adds no negative delta. It can yield ambient quieter behavior through absence of positive current signals and regular mood/ambient evolution, but it may never lower personality values, create distress, or trigger recovery debt.

Start with deliberately conservative coefficients and acceptance criteria: typical regular use produces an instrument-detectable but not user-visible movement after roughly one week, and a perceptible cumulative change after roughly three weeks. Build deterministic simulation fixtures for no activity, qualified idle presence, background-tab false presence, repeated offers at cooldown boundary, concurrent devices, two-week absence, and month-long regular use. Review the distribution using synthetic test aviaries only; do not aggregate real users' interaction histories.

### Mood, social behavior, and ambient events

Implement mood as an enumerated state machine: `wary`, `content`, `curious`, `drowsy`, and `alert`, with future additions only through a versioned transition table. On every tick, score candidate transitions from current mood, local time phase, temporary weather, recent event effects, the bird's own vector, and nearby birds' events. Use weighted probabilities plus hysteresis/minimum dwell time so a bird does not visibly flip mood on successive snapshots. Persist current mood and next evaluation/dwell boundaries; do not reset it when a browser session begins.

Examples of controlled rules:

- early morning biases alert; late evening/night biases drowsy/settled, while the night-active species retains an allowed late call path;
- rain reduces call propensity briefly; wind introduces small alert/wary bias;
- high boldness resists a wary transition, high curiosity increases response probability to novel offers, high social warmth increases call response/greeting candidacy;
- an alarm-like call may gently propagate wary bias to nearby birds; compatible high-vocal birds can generate a staggered chorus;
- offer outcomes are selected by canonical mood/vector and a recorded random seed, not by the client.

Weather is an aviary-level seeded schedule with rare short rain and occasional soft wind; it is not a user-facing weather system or per-leaf state. Perch selection is server-authored intention based on mood/boldness and occupancy constraints. The projection can target front/middle/back zones with a pose/action, and the client supplies the continuous path or reduced-motion transition.

### Calls, greetings, notebook, and growth rules

Give each species a compact procedural motif library and each bird a stable call-signature seed. The simulation projection supplies a next-call window, motif family, pitch/timing variation bounds, mood contour, and caption grammar inputs; the browser synthesizes the actual call. This preserves recognizability across variation without storing/download-loading recordings.

Generate return greetings server-side when an authenticated owner returns after a meaningful visibility/session gap. Select one primary bird with weighted boldness, social warmth, current mood, and absence-duration context; choose a deterministic variation from the per-bird seed. Longer absence changes the form, not the tone. If secondary responses qualify, schedule them at randomized small offsets. Do not write generic arrival text and do not treat a visitor pull as a greeting/presence event.

Generate notebook entries from explicit observation candidates (unusual greeting order, sustained quiet, weather/mood combination, meaningful rare interaction) and a per-aviary sparsity policy. Persist rendered naturalist prose at generation time so old records do not change with a future text-template release. Cap ordinary entries to approximately one every few days for regularly visited aviaries, permit rare noteworthy exceptions, and prevent duplicate semantic entries. The notebook is read-only and never writes user-behavior commentary such as visit streaks.

Evaluate third-through-seventh-bird eligibility strictly from aviary age and capacity using an idempotent server-issued offer record. Use a quiet in-flow offer when eligible, not a countdown, a score, or an achievement. Pick species from the coherent pool without rarity/collection mechanics. Adoption creates a stable bird id and initial vector/call seed atomically; rename affects only user-facing name.

## 6. Sync, client session, and conflict model

The canonical server record and append-only owner event log eliminate client-to-client state merging. Every visible client is a reader of versioned snapshots; every mutation is an idempotent intent record. Do not implement last-write-wins fields for bird vector/mood/perch state.

### Client synchronization lifecycle

1. Authenticate, request shell plus a compact initial snapshot, and render available bird primitives immediately; defer settings, notebook history, invitation management, and nonessential icons.
2. Initialize local render state from the snapshot's server clock, animation seed, and call schedule. Start a low-frequency visible-tab conditional pull with ETag/version.
3. On `visibilitychange` to visible, focus restoration, reconnect, a long animation-frame gap/suspend, or an accepted local event, fetch a fresh full/delta snapshot. Rebase visual interpolation, never locally merge canonical state.
4. Queue user intent events in a small durable client outbox with idempotency keys. Retry with backoff while the session remains valid; show a matter-of-fact non-blocking sync error only when the user's requested action could not be accepted. Local audio/pose feedback may be speculative only when clearly reversible; canonical result arrives from the server.
5. On token expiration/revocation or unrecoverable snapshot mismatch, stop mutation, discard client projection, and direct the user through the applicable sign-in/reload error path.

For simultaneous phone/laptop usage, events arrive in server order and are consumed once by the next tick. A listen-in is device-local audio focus but may be logged as attention evidence for the selected bird; it must not globally mute another device's mix. A settle is local scene presentation plus an event ending that device's qualified presence, not an aviary-wide lockout. Name/settings writes use version preconditions and field-specific conflict errors; on mismatch, refetch and present a plain choice only where a user-owned editable field truly conflicts. Canonical simulation fields never expose a conflict UI because clients cannot write them.

## 7. Frontend scene and rendering pipeline

### Rendering architecture

Implement a lightweight scene renderer separated from DOM product controls:

- semantic HTML handles top bar, dialogs, settings, notebook, account/auth, keyboard focus, live narration, captions, and all error surfaces;
- a canvas/WebGL-capable scene layer (with a tested low-complexity canvas fallback within supported browsers) renders birds, perches, foliage, subtle parallax, lighting, weather, and ornamental leaves/feathers;
- a pure render-state adapter maps validated snapshot state to scene models; it must not calculate personality, server time progression, or eligibility;
- requestAnimationFrame runs only while visible; cancel/pause it on hidden state, release transient work, and refresh snapshot on return. Server simulation remains independent.

Keep visual assets procedural, SVG, or compact bitmap primitives. The initial scene bundle must stay below 2 MB gzipped. Split account/accessibility settings, notebook history, export/deletion, and invitation management into lazy chunks. Preload the minimal bird silhouettes/motif tables needed for the two starter birds and defer everything that cannot affect first bird paint.

### First-frame and scene behavior

Render a quiet-field placeholder only if the snapshot cannot arrive in time: calm sky/light color with one or two nonsemantic ambient cues, never spinner, skeleton dashboard, or fade-from-static. As soon as a valid snapshot arrives, place each bird in an in-progress pose/call/motion phase, initialize ambient variation at a nonzero time offset, and start rendering without a wake-up entrance. The only allowed empty scene is the brief post-adoption/pre-first-bird handoff, followed by the new bird's soft fly-in; an established aviary never becomes empty during loading.

Use three logical perch zones and responsive coordinate transforms. Narrow layouts compress spacing but always keep every bird in frame; wide layouts expand spacing without changing the one-screen/no-pan/no-zoom rule. Draw bird scale/depth, silhouette opacity, and subtle focus treatment by zone. The user cannot drag birds or issue movement commands.

Drive normal-mode micro-motion through bounded pose state machines: preen, scan, head tilt, weight shuffle, call, approach, and rest. Sample variations from per-bird render seeds and mood/vector inputs, but do not make a static looping cycle. Overlay gentle ambient drift, modest parallax, local-time palette, and sparse ornament-only leaves/feathers. Weather modifies visual/audio ambience softly and never becomes an intrusive event.

The thin top bar contains only account/settings, accessibility, field notebook, offer, and settle affordances as appropriate. Fade it almost transparent after cursor stillness and restore it on pointer movement/keyboard activity. Keep it discoverable and keyboard reachable; do not place labels, badges, or hover tooltips in the aviary scene. Use naturalist prose in product affordances and direct copy in system/settings surfaces.

### Interactions and motion states

Click/tap/focus selects a bird for listen-in; keyboard Tab enters the scene at the first bird, arrows move bird focus, Enter toggles listen-in, and Escape exits it. The focused bird receives a high-contrast visible focus indicator that works against day/night palettes; focus must be semantic even if the canvas owns visible pixels.

Listen-in ramps the selected bird's gain up and all others down to ambient rather than cutting them. Clicking the same bird, choosing another, clicking empty scene, or moving focus away exits/replaces listen-in with the same smooth ramp. Offer originates only from the top-bar affordance; it presents the three allowed gifts and returns a canonical response animation from the next snapshot/event result. Enforce per-bird few-minute cooldown server-side and convey unavailable state without shaming language. Settle shifts toward evening and quiets calls over seconds; any aviary click in the next five seconds reverses it. Closing a tab remains equivalent at the simulation/presence level.

## 8. Client audio and caption pipeline

Create one managed WebAudio graph per visible client session: master gain, ambient bus, focused/listen-in bus, per-active-call voice nodes, limiter/compressor, and lifecycle cleanup. Browser gesture requirements may defer audible playback until permission, but the scene and captions must still behave as an alive aviary. Do not use pre-recorded call assets in any path.

For every server-scheduled call, choose an allowed motif plus deterministic variation from the bird's call seed and snapshot seed. Synthesize oscillator/noise/envelope/filter components client-side, using mood to change phrase length, pitch contour, spacing, and timbre while preserving the bird's recognizable signature. Schedule with lookahead against the AudioContext clock, use short bounded voice pools, reuse buffers/nodes where practical, and dispose completed voices. Implement chorus by independently scheduling/mixing bird calls, with slight stagger and spatial/depth-aware gain, not by stacking loops.

When listen-in starts, automate gain ramps over a calm fixed range; leave nonfocused birds audible at ambient gain. When the render tab hides, pause/suspend safely as browser policy allows and reconstruct scheduling from a fresh snapshot on resume. Monitor AudioContext state and error classes without collecting content, bird ids, or account identifiers.

Generate caption text from the same instantiated call grammar (motif/contour/repetition/mood) used for synthesis, then attach it to the calling bird's accessible DOM overlay. Captions use concise naturalist descriptions, are synchronized to actual call timing, and fade gently. If WebAudio is unavailable, denied, or fails, switch to graceful silence and enable captions by default; never silently download a canned-audio substitute.

## 9. Accessibility and content-system plan

Treat accessible output as a designed representation of the same canonical state, not a hidden list of engine values.

- Provide a dedicated polite live narration region managed by a narration scheduler. At idle, emit one naturalist observation every 30–60 seconds at most; coalesce/suppress redundant updates. Give return greetings, accepted offer reactions, and settle events a measured priority bump without interrupting speech excessively.
- Build narration from the render projection and canonical event result using shared naturalist templates: lowercase, present tense, named/visible particulars. Never expose personality numbers, raw perch indexes, generic status codes, or a firehose of pose transitions.
- Honor `prefers-reduced-motion` on first render and expose a persistent override. In reduced motion, render slow cross-fades between designed still poses, replace flight paths with perch-to-perch cross-fades, remove leaf drift, and retain slower palette changes, calls/captions, mood, and notebook activity. This is a tested rendering mode, not an animation kill switch.
- Provide call-caption control in accessibility settings and ensure the fallback path defaults it on. Caption positioning must not obscure controls; caption text, top-bar labels, settings, errors, and visually displayed narration meet WCAG AA contrast across all light phases.
- Make every control operable via keyboard, include visible focus indication in all states, correct dialog focus trapping/return, touch targets, semantic names, and headings/landmarks for non-scene UI. Ensure bird focus and canvas interaction have an equivalent semantic control model for screen-reader and keyboard users.
- Test with at least VoiceOver/Safari, NVDA/Firefox, and Chrome screen-reader paths; test reduced motion, forced colors/high contrast where supported, keyboard-only flows, muted/no-audio, denied WebAudio, narrow mobile layout, and zoom/reflow behavior for settings/notebook surfaces.

Maintain a small content-template library with tone tests: product templates reject exclamation, gamification words, user-behavior scoring, and generic status copy; system templates require clear subject/action/remedy. Review any generated notebook/narration/caption copy through deterministic fixtures before release.

## 10. Performance, reliability, security, and observability

### Budgets and technical acceptance gates

| Measure | V1 gate | Verification |
| --- | --- | --- |
| Initial JavaScript | under 2 MB gzip | CI bundle analyzer and production artifact check per route. |
| First bird visible | under 500 ms on mid-tier mobile over representative 4G | scripted performance trace from navigation to first actual bird pixels, warm and cold cache. |
| Idle rendering | 60 fps on a five-year-old mid-range laptop | 30-minute automated soak with frame-time percentiles, normal and two-to-seven-bird scenarios. |
| Memory | no upward trend over 30 minutes | heap/renderer/audio soak; assert bounded audio voices, scene objects, notebook virtualization, and worker lifecycle. |
| Tick latency | p99 below 5 seconds | server histogram/alert, synthetic seeded aviary load. |
| Snapshot size | low kilobytes for ordinary aviary | contract-size CI and CDN/API timing tests. |

Instrument aggregate-only RUM for navigation/request timing, first-bird render, long frames, AudioContext errors, snapshot failures, and feature fallback rates. Use anonymous/rotating aggregation that cannot be joined to account or bird behavior. Run synthetic browser checks from representative geographies for availability, auth handoff test accounts, first render, snapshot freshness, audio fallback, and revocation behavior. Do not emit bird ids, event payloads, email, notebook text, trait values, or per-account timelines into logs/traces/metrics. Redact request bodies and token values at the logging boundary.

Apply standard web protections: TLS, secure/httpOnly/same-site session cookies or equivalent secure token handling, CSRF defenses for cookie-authenticated mutations, strict CSP, rate limiting, secret rotation, signed opaque download/invite tokens stored as digests, encryption for email and export data, authorization checks at every data access, and audit events for account/session/invitation/deletion administration. Make account hard deletion a verified asynchronous purge across primary DB, queues, objects, and operational data within the stated lifecycle; do not retain a shadow simulation archive.

## 11. Delivery sequence and rollout

### Engineering milestones

1. **Foundations and invariants.** Define schemas/migrations, opaque-ID policy, auth/session lifecycle, event idempotency, simulation lock/cursor, API contracts, configuration versioning, privacy-safe logging, and deterministic fixture harness. Gate on migration and authorization tests.
2. **Canonical aviary vertical slice.** Create two starter birds, server tick, snapshot projection, one browser renderer, qualified presence events, basic day/night/mood, first-frame/no-spinner behavior, and server-side replay fixtures. Gate on canonical multi-device and no-false-presence tests.
3. **Affective interaction layer.** Add procedural greeting/call pipeline, listen-in, offers/cooldowns, settle/undo, bird-to-bird responses, notebook sparsity/content system, and age-gated adoption. Gate on tone/content and long-timescale calibration fixtures.
4. **Accounts and private social.** Complete magic-link flows, session management, export/deletion, invitations/revocation/expiry, visitor snapshot scope, silent visit log, and matter-of-fact failure surfaces. Gate on permission and revocation tests.
5. **Accessibility/performance hardening.** Complete narration scheduler, captions, reduced-motion renderer, keyboard semantics, browser fallback, bundle splitting, performance/memory soaks, and support-matrix tests. These are release requirements, not post-launch work.
6. **Controlled release.** Launch to an internal/synthetic cohort first, then an invite-only external cohort with only two birds. Observe aggregate operational health and qualitative opt-in feedback without inspecting private interaction histories. Expand availability after correctness and performance gates remain stable; enable later age-gated bird offers only after their timing reaches the configured aviary-age thresholds, ramping maximum eligible bird count cautiously from 2 to 3, then toward 7.

Use feature/config flags only for operational rollout and calibration (tick cadence, drift coefficients, weather frequency, notebook sparsity, species availability, maximum eligible count). Never use them to introduce gamification, alerts, visitor influence, or a separate social mode. Configuration changes carry version/audit metadata and are evaluated against synthetic fixtures before release; do not retroactively recompute stored personality history.

### Release criteria

Before general availability, require successful end-to-end tests for magic-link expiry/replay, simultaneous device events, worker retry/idempotency, reconnect after suspension, visitor read-only enforcement/revocation, deletion recovery/hard purge, audio denied fallback, and every accessibility interaction. Run the complete performance suite against two and seven birds. Require product review of first-frame behavior, greeting/no-announcement behavior, notebook/narration/caption specificity, system-copy clarity, and a negative-scope audit proving no scores, streaks, badges, push loops, trait display, public discovery, or tamagotchi state reached the release surface.

## 12. Key risks and mitigations

| Risk | Mitigation and launch signal |
| --- | --- |
| Drift changes too fast, too slowly, or becomes behaviorally gameable | Versioned conservative coefficients, server-only computation, qualified-presence validation, saturation/cooldowns, replay fixtures for week/three-week targets, synthetic distribution reviews. |
| Background tabs inflate presence | Require visibility + focus + recent activity simultaneously; heartbeat/reporting caps; client/server sequence validation; test hidden/unfocused/idle edge cases. |
| Concurrent-device writes silently lose character history | Append-only idempotent intent events, server sequencing, per-aviary tick lease, additive server deltas, no client trait writes/last-write-wins. |
| Tick delay/retry causes discontinuity or duplicate drift | Durable cursor/input ranges/config seeds, transactional commit, idempotent run records, bounded catch-up, p99 alert and replay tooling. |
| Calls sound canned, collide, or consume memory | Per-bird motif/signature grammar, deterministic variation, staggered chorus, pooled bounded voices, 30-minute audio soak, no recorded fallback. |
| Initial loading breaks the alive illusion | Minimal snapshot/projection, CDN delivery, deferred noncritical code, quiet-field fallback only, first-bird pixel regression test. |
| Accessibility becomes a state dump or a disabled animation fallback | Dedicated narration/content scheduler, shared canonical source, designed reduced-motion cross-fades, manual assistive-tech review, release-blocking tests. |
| Naturalist voice leaks into account errors or generic/product-announcement UI leaks into scene | Separate content domains/templates, copy linting and visual review, test fixtures for no welcome/streak/badge/notification strings. |
| Invitation sharing becomes a social network or affects host birds | Per-email one-time scopes, read-only visitor API, no visitor presence events, no discovery/chat/comments, default-off invitations, revoke-on-next-pull tests. |
| Privacy boundary erodes through logs/analytics | Synthetic UUIDs, encrypted email isolation, redaction, aggregate-only metric schema review, deny simulation database access to analytics/ML pipelines, deletion purge audit. |
| Seven birds degrades recognizability/performance | Age-based gradual ramp, species/motif mix tests, 2–7 bird load/audio tests, retain hard capacity ceiling and reduce rollout ceiling if the experience fails. |

The plan is successful only if operational correctness and privacy are achieved without turning the product into a dashboard. The server must make continuity real; the client must make that continuity quiet, immediate, specific, and accessible.
