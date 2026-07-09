# Pocket Aviary v1 implementation plan

## 1. Delivery contract and scope guardrails

Build Pocket Aviary as a responsive browser application with one canonical, server-owned aviary for each signed-in account. The first shipped experience is a two-bird, one-screen place that is already alive on first render. It supports a calm session of watching, listening in on one bird, making a bounded offer, settling the scene, reading a generated Field Notebook, and—in a deliberately separate account surface—issuing and revoking read-only visit invitations. The client renders and submits facts about user actions; it never owns or computes durable bird personality.

The v1 team should ship these capabilities together rather than treating them as progressive enhancements:

1. Email magic-link identity, per-device revocable sessions, one aviary per account, starter adoption/naming, rename, export, and 30-day recoverable deletion.
2. A server-side, minute-scale simulation for persistent birds, personality drift, mood, weather, availability of later adoption, sparse notebook observations, and a snapshot that can seed a render without a visual startup sequence.
3. A visual and audio aviary with day/night and weather, three perch zones, continuous personality- and mood-shaped motion, real procedural calls, listen-in mixing, offers, settle, and client-side interpolation.
4. Precise host-only presence accounting, multi-device consistency, and read-only visitor sessions that cannot affect the host state.
5. Designed accessibility: prose narration, runtime call captions, keyboard operation, contrast, and a cross-fade-based reduced-motion scene.
6. Aggregate-only operations instrumentation and performance controls that protect the first-bird and long-session budgets.

The following are hard scope exclusions and must be encoded in product acceptance checks as well as left out of backlog decomposition: native clients; password and SSO login; multiple/shared aviaries; scene customization, panning, zooming, or user-positioned birds; user-visible trait values, moods as a dashboard, or hunger/happiness/health state; client-side personality writers; recorded-call fallback; feeds, profiles, discovery, follows, chat, comments, co-presence, avatars, and public sharing; push/email engagement notifications; social/gamification metrics, streaks, badges, levels, scores, ranks, or adoption counters. The only notification-like social behavior is the explicit host setting for visit notifications, off by default. It must not become an onboarding or badge surface.

Use two explicit copy domains throughout implementation and QA:

- **Aviary domain:** lower-case, present-tense, specific naturalist observations. This applies to greetings, notebooks, narration, offer responses, and captions.
- **System domain:** conventional, direct, matter-of-fact copy for authentication, sessions, account settings, accessibility settings, unsupported browsers, deletion, export, and errors.

No feature should manufacture urgency around absence. In particular, no state transition, metric, or copy path may turn lack of presence into negative personality movement, visible distress, or a recovery task.

## 2. System shape and bounded responsibilities

Deploy a modular web application with an edge-delivered browser client, a stateless API layer, a transactional simulation service/worker, a relational source of truth, an email delivery component, and an aggregate-only telemetry pipeline. Keep the simulation data store isolated from telemetry and analytics credentials; there should be no warehouse replication or query path from per-account simulation records.

### 2.1 Client responsibilities

The browser client is responsible for:

- Establishing an authenticated account or constrained visitor session.
- Fetching state snapshots, applying only ephemeral render interpolation, and refreshing on visibility return, suspension-sized frame gaps, and a low-frequency visible-tab poll.
- Collecting the raw browser signals necessary for presence and emitting bounded, idempotent presence-window events only while all required signals are true.
- Sending immutable interaction intents—listen-in start/end, offer selection, settle, adoption/naming actions, and account-setting changes—rather than calculations or absolute bird state.
- Producing deterministic scene composition from an authoritative snapshot plus client-local visual ornament seeds; client-only leaf/feather ornaments are not simulation state.
- Synthesizing procedural audio and captions from server-supplied call grammar/signature data; managing AudioContext lifecycle, mix ramps, and the silent-captioned fallback.
- Exposing semantic controls, controlled live narration, focus behavior, and the independently designed reduced-motion renderer.

The client must not persist canonical moods, traits, notebook content, an interaction queue capable of replaying stale state, or a client clock that affects simulation. It may keep a short, encrypted/session-scoped cache of the most recent snapshot solely to render a quiet field and accelerate re-open, but it must immediately reconcile against the server version and never render that cache as an authoritative durable change.

### 2.2 Server responsibilities

The API and simulation layer owns identity, authorization, all durable aviary records, immutable event acceptance, exact event sequencing, simulation tick execution, invitation revocation checks, notebook creation, and snapshot assembly. The simulation worker is the only process role with permission to update `bird_personality` and durable mood/scene projection records. API request handlers only append validated events or mutate explicitly administrative/account records.

Use a relational database with transactions and row-level locking or serializable transactions for account/aviary simulation work. A queue can schedule due aviaries, but the simulation transaction must independently detect elapsed ticks and serialize work per aviary, so duplicate deliveries or worker retries cannot apply a delta twice. Object storage is appropriate only for export artifacts and must use short-lived, single-account download links.

Keep a versioned simulation module separate from transport code. It receives canonical inputs and a simulation timestamp, returns deltas plus domain events, and has deterministic tests against a seeded pseudo-random generator. That boundary lets calibration evolve without making HTTP handlers or rendering code the hidden engine.

### 2.3 First-render boundary

Serve a small app shell and a compact, cacheable scene renderer at the edge. Once authenticated, bootstrap the current snapshot through a CDN-near API/read path; do not block initial bird paint on settings panels, notebook history, invitation management, account export controls, or noncritical audio initialization. The scene renderer must render birds at snapshot-defined pose phase/position immediately, then attach high-detail animation and audio. On a slow snapshot request, show a quiet field with restrained ambient cues—never a spinner, skeleton, toast, wake-up animation, or static placeholder presented as the aviary.

## 3. Persistent model, invariants, and retention

Use opaque UUIDs for all account-facing and internal joins. Email is encrypted on the single account record and is never an identifier, event attribute, trace attribute, queue key, partition key, or telemetry label. Distinguish immutable facts/events from current simulation projections so repair and audit do not rely on reconstructing personality from history.

### 3.1 Core records

| Record | Key fields | Ownership and rules |
|---|---|---|
| `account` | `id` UUID, encrypted `email`, verification state, status (`active`, `pending_deletion`, `deleted`), deletion timestamps, timezone preference/default, created time | One aviary per active account. Email changes retain the old verified address until new verification succeeds. |
| `auth_magic_link` | opaque hashed token, `account_id`/pending-email reference, purpose, issued/expires/consumed timestamps, requester-rate bucket | 15-minute expiry; atomically consume once. Raw token is never stored. |
| `device_session` | opaque hashed refresh/session token, `account_id`, created/last-seen/revoked/expires timestamps, minimal device display metadata | Revocable individually. Authentication returns account-scoped claims only. |
| `aviary` | `id`, `account_id` unique, creation time, canonical version, last simulated time, local-time zone, settled-until/re-engagement metadata, current weather state/seed, next adoption eligibility | Exactly one per account. Version increments for externally observable durable changes. |
| `bird` | stable UUID, `aviary_id`, species id, user name, adopted timestamp, ordinal, active flag | Identity never changes on rename, migration, or species catalog changes. Start exactly two; enforce max seven. |
| `bird_personality` | `bird_id`, boldness, social warmth, vocal frequency, plumage saturation, curiosity, calibration version, updated tick/version | Hidden server-only normalized values. Updated only by simulation transaction as additive, clamped deltas; no decrease from neglect. |
| `bird_mood` | `bird_id`, enum (`wary`, `content`, `curious`, `drowsy`, `alert`), entered time, transition seed/context, current perch zone and pose/action phase | Persistent fast-timescale projection, not a client derivation. It carries through sessions and evolves between them. |
| `interaction_event` | immutable event UUID/idempotency key, `aviary_id`, actor type, device/session reference, event type, bird/offer target, client-observed time, server receipt time, ordered sequence, payload version | Host events only can contribute to drift. Server sequence is authoritative; validate all payloads and cooldowns on insertion. |
| `presence_window` | `id`, `aviary_id`, host session, start/last-qualified/end times, reason, bounded accumulated seconds, monotonic counters | Stores server-validated intervals from qualified pings, not raw mouse coordinates or keystrokes. Visitor sessions cannot create rows. |
| `simulation_cursor` | `aviary_id`, last processed event sequence, last simulated instant, lock/version | Makes tick application resumable and exactly-once relative to event range. |
| `scene_snapshot_projection` | `aviary_id`, version, generated time, birds' current perch/action/call schedule, time-of-day, weather, greeting eligibility | Materialized compact read model for API/edge delivery; regenerated transactionally from canonical state. |
| `notebook_entry` | immutable UUID, `aviary_id`, occurred time, generated time, source domain event/category, safe structured facts, final naturalist prose | Sparse, read-only, indefinitely scrollable. Generation is rate-limited and never becomes per-session event logging. |
| `offer_cooldown` | `bird_id`, offer category if needed, next allowed time | Enforced server-side for a few minutes; refusal is neutral and not punitive. |
| `visit_invitation` | UUID, host `aviary_id`, encrypted/hashed invitee email, opaque token hash, issued/expires/revoked/first-used timestamps, notification preference snapshot | Explicitly created, 30-day unused expiry, instantly revocable. Do not create a global social graph. |
| `visit_session` / `visit_log` | invitation, issued/revoked/end times, approximate duration, snapshot reads | Read-only authorization and transparency audit. The host log exposes email/date/approximate duration and outstanding invites only in settings. |
| `account_setting` | accessibility overrides, captions, visit-notification opt-in, privacy acknowledgement/version | System-surface settings; no engagement flags or public visibility field. |
| `export_job` | account id, requested/completed/expiry times, encrypted artifact reference | On-demand JSON, delivered only to the currently verified email using a short-lived link. |

### 3.2 Event taxonomy and data minimization

Version interaction schemas and allow only: `presence_qualified`, `presence_end`, `listen_in_started`, `listen_in_ended`, `offer_requested`, `offer_resolved`, `settle_requested`, `settle_undone`, `reengaged`, `adoption_completed`, `bird_renamed`, and server-created ambient/simulation events. An event includes an opaque account/aviary/bird ID and just enough structured information to reproduce its effect (for example, offer type), never raw input data, pointer coordinates, keyboard content, user-agent fingerprinting, or email.

Record both client-observed and server-received timestamps for observability/debugging, but sequence all state impacts by a server-issued monotonic per-aviary sequence. Reject malformed, expired-session, unauthorized, duplicate-idempotency-key, visitor-originated, or out-of-policy events. Persist acceptance/rejection reason codes only as aggregate-safe operational counters; do not send simulation details to telemetry.

### 3.3 Retention and lifecycle

Soft-delete marks the account inaccessible immediately except through the recovery action; do not run new simulation or send invitations for pending-deletion accounts. A recovery inside 30 days reinstates the same records and bird identities. A scheduled hard-delete transaction/purge removes all account-linked database rows, export objects, tokens, invitation/session/log entries, and operational correlation mappings after day 30. Ensure backups follow a documented bounded purge schedule and never restore deleted accounts into active service without reapplying deletion tombstones.

Account export serializes the current birds (including current vectors), moods, notebook entries, and account settings as specified, with explicit schema version and generated-at timestamp. It is a private account artifact, not an internal analytics feed.

## 4. API, authorization, and delivery contracts

Use JSON over HTTPS with explicit versioned routes or media types. Every mutating host request supplies an idempotency key; snapshots carry `aviaryVersion`, `generatedAt`, and an opaque ETag/version. Return standard matter-of-fact system errors for auth/sync failures, not naturalist copy. API objects must omit hidden personality values from normal aviary and visitor responses; export is the explicit exception authorized from account settings.

### 4.1 Identity and account routes

- `POST /v1/auth/magic-links`: accepts email and purpose, applies rate limiting without account-enumerating responses, emails a 15-minute one-time link.
- `POST /v1/auth/magic-links/consume`: atomically consumes token and creates/rotates a per-device session. Return authenticated bootstrap metadata and an aviary-bootstrap URL.
- `POST /v1/auth/refresh`, `POST /v1/auth/logout`, `GET /v1/account/sessions`, `DELETE /v1/account/sessions/{id}`: issue/rotate/revoke only the caller's sessions.
- `PATCH /v1/account/email`: starts new-email verification; `POST /v1/account/email/confirm` commits only verified replacement. The old address remains valid until then.
- `GET/PATCH /v1/account/settings`: read/write accessibility and permitted social setting values. Keep setting controls separate from aviary scene bundle.
- `POST /v1/account/export`: enqueue export after reauthentication if required; do not return state in an API response that could be cached or exposed.
- `POST /v1/account/deletion`, `POST /v1/account/deletion/recover`: start/cancel the 30-day lifecycle. All copy here is system-domain.

### 4.2 Host aviary routes

- `GET /v1/aviary/bootstrap`: a compact authenticated snapshot for first paint: current time/day phase, weather, settled state, each bird's public name/species/render signature/mood expression/perch/pose and call schedule, greeting directive, accessibility narration seed, and `aviaryVersion`. It never includes numeric traits.
- `GET /v1/aviary/snapshot?sinceVersion=`: returns full compact snapshot or `304`/minimal delta when possible. Require a full snapshot after long suspension or version gap. The client calls on visibility regain, frame-gap recovery, and a low-frequency visible keepalive.
- `POST /v1/aviary/events`: host-only append endpoint. Accept one validated event or a compact batch of qualified presence interval pings. The server stamps sequence/time and returns accepted event IDs, not recomputed personalities.
- `GET /v1/aviary/notebook?cursor=`: paginated immutable entries ordered newest first, preserving indefinite history through cursor paging. It is lazy-loaded only when opened.
- `POST /v1/aviary/adoption`, `PATCH /v1/birds/{birdId}`: gated server-side by aviary age/eligibility and cap. Starter naming occurs in onboarding; rename preserves bird ID. Never expose a catalog/rarity/purchase endpoint.

The events endpoint must not be a general patch API. Schema validation should make requests such as `personality`, `mood`, `perch`, or arbitrary notebook text impossible to submit.

### 4.3 Visit routes and visitor claims

- `POST /v1/visits/invitations`: host creates one invitation for a named email. Rate-limit and do not reveal whether that email has an account. Store encrypted/hashed email only for invitation/log needs and send a one-time email link.
- `GET /v1/visits/invitations`, `DELETE /v1/visits/invitations/{id}`: settings-only host management and immediate revocation.
- `POST /v1/visits/consume`: validates an unexpired, unrevoked one-time invite link and issues a narrowly scoped visitor token/session. It may allow subsequent snapshot reads during an active authorized visit, but cannot receive host API scopes.
- `GET /v1/visit/snapshot`: returns the same rendered state surface as host snapshot, without greetings, presence/personal interaction affordances, trait values, notebook mutation, account data, or controls. Check invitation revocation on every request, not just token issuance. If unavailable, present the specified matter-of-fact visit-unavailable view.

Visitor assets and route guards must not expose mutable controls through hidden DOM, keyboard shortcuts, or API authorization gaps. Start/finish visit logging from authorized snapshot access only; calculate approximate duration from bounded session/read intervals rather than invasive tracking. Host notification sends only when the host has explicitly enabled the setting; default behavior is a silent settings log with no badge.

## 5. Simulation engine and canonical tick

### 5.1 Scheduling, locking, and time

Run a scheduler at least once per minute that identifies active aviaries due for simulation, plus an on-demand catch-up at authenticated snapshot read. Canonical tick time is server UTC converted to the aviary/account local time zone for day phase; never trust client wall clock for state. For each aviary, lock its cursor/aviary row, calculate elapsed tick boundaries since `last_simulated_at`, consume all unprocessed accepted events in server sequence order, produce one or more bounded tick transitions, update projections and cursor atomically, then release the lock.

The design must be safe when workers run twice, jobs arrive late, requests overlap, or a device replays an idempotency key:

1. The cursor records the last committed event sequence and tick instant in the same transaction as bird changes.
2. Event insertion deduplicates `(aviary_id, idempotency_key)` and rejects old/revoked sessions.
3. Tick math iterates deterministic bounded intervals and records a simulation version/calibration version, so retry from the cursor reaches the same result.
4. A snapshot request can enqueue/perform bounded catch-up but cannot bypass the serialization or write direct personality.
5. Simulation is rate-limited/catch-up capped with a documented degradation path for extreme downtime: simulate time in coarser deterministic intervals after a threshold while preserving mood/day boundaries and never inventing interaction/presence input.

### 5.2 Presence pipeline

In the client, maintain a small state machine for `visible`, `focused`, `lastMeaningfulActivityAt`, `qualified`, and `settled`. Pointer movement or keypress updates activity locally; do not transmit the raw event. A periodic checker qualifies presence only if `document.visibilityState === "visible"`, `window/document` has focus, and activity is within a build-calibrated several-minute window. When all are true, begin/continue a qualified window and emit coarse heartbeat intervals. On any condition failing, on `visibilitychange` hidden, `blur`, explicit settle, unload/pagehide, or expiry of activity, close/stop the window.

The server validates heartbeat intervals against session age, receipt time, maximum interval length, monotonic client sequence, and a modest clock-skew allowance. It caps credited seconds per interval, closes stale windows, and ensures a visitor token cannot call the endpoint. This yields honest attention accounting without storing behavior traces. Lost network or a frozen tab must fail closed after the last accepted bounded interval; it must not infer presence from a tab simply remaining open.

Settling creates a `settle_requested` event, ends qualified presence exactly as loss of attention does, and applies the visual/audio settle projection. Any scene click inside five seconds creates `settle_undone`/`reengaged` and returns to normal eligible presence. Closing without settle is semantically equivalent for simulation and earns neither penalty nor missing-step state.

### 5.3 Personality drift

Store trait values in a normalized bounded range with server-selected starter seeds and per-species baseline distributions. For every tick, derive nonnegative evidence terms from new, accepted host events since the cursor:

- qualified presence duration is the largest input and contributes a very small, saturating amount toward expressiveness across relevant traits;
- listen-in duration applies a controlled bonus to the focused bird's social warmth and vocal-frequency trajectories;
- a valid offered gesture applies a small boldness signal for being offered near the bird, while accepted/investigated offers add small curiosity signal;
- settle changes only near-term mood/scene and terminates presence; it does not grant a personality bonus.

Apply a low-pass, bounded additive update, for example `delta = remaining_headroom * rate * evidence_weight * smoothing`, then clamp to the valid range. Rates and weights belong in a versioned server calibration table/feature configuration and are tested as a whole; they must not be tuned from production user interaction data. Ensure the update can never be negative because of lack of presence, inactivity, elapsed time, rejected offers, or a mood transition. “Ambient” after absence is represented by fewer recent greeting opportunities / normal low-frequency behavior, not reduced trait storage.

Build calibration tests around product outcomes rather than only formulas: a normal regular-visit synthetic profile produces measurable nonzero trait changes after about seven days, produces changes perceptible through behavior after roughly three weeks, and cannot visibly shift a trait after one session. Run edge cases for long absence, two overlapping devices, excessive offer attempts, clock changes, and a visitor watching for hours. Persist the calibration version with each update to support safe migrations/analysis without retroactively recomputing identity-defining vectors.

### 5.4 Mood, weather, social bird behavior, and scene projection

Mood is a durable finite-state model; it is not reset on page open. At each tick, calculate candidate transitions from current mood duration, local day phase, current weather, recent accepted interactions, nearby bird events, and trait-weighted probabilities. Use a deterministic PRNG seed derived from aviary/bird/tick/calibration version so repeated processing produces identical outcomes. Examples:

- dawn shifts likelihood toward alert, dusk/night toward drowsy/settled;
- rain temporarily suppresses vocal tendency and can make susceptible birds quieter;
- wind can increase alertness or wary likelihood depending on traits;
- an accepted offer can increase content/curious likelihood; a high-boldness bird resists entering wary under the same ambient trigger;
- an alarm-like call domain event can temporarily make nearby birds wary;
- a high-social/high-vocal pair may create a chorus opportunity.

Make weather rare, soft, bounded, and server-authored from an aviary-level deterministic schedule—short rain a few times weekly and occasional wind, no severe weather feature. Keep scene state minimal: local time phase, palette interpolation target, weather start/end/intensity seed, settled state, and per-bird perch/action/call opportunities. A sampling/projection layer chooses front/middle/back perches from mood and boldness while preventing collisions and keeping every bird in frame. It also produces current action phase offsets so first paint begins mid-preen/scan/call rather than at frame zero.

Schedule calls as opportunities, not prerecorded outputs. The server snapshot can include deterministic short-horizon timing/motif selection seeds, allowing clients to render the same current mood/signature without treating audio playback as durable state. Stagger greetings/call responses with small deterministic-random offsets; never fire every bird at once as a return cue.

### 5.5 Greeting, offers, notebook, and later adoption

On a qualifying host scene return/open after snapshot reconciliation, generate an ephemeral greeting directive from absence duration, mood, boldness/social warmth, and the current scene. Select one primary bird with weighted fairness/avoid-repeat history, then optionally stagger a related response. Short absences favor a glance; longer absences permit a stronger reorientation/call. It is visual/audio/narration behavior, never a text welcome or an event that changes personality. Do not run greeting for visitors.

An offer request targets the aviary through the top-bar affordance, then resolves toward a receiving bird using current state. The server checks per-bird cooldown, creates a neutral response/rejection if unavailable, and writes a structured outcome domain event. The rendering layer expresses seed/song-fragment/still-pool reaction through mood and curiosity; no offer is feeding, maintenance, score progress, or a way to force a trait.

Generate notebook entries from a small template/grammar library driven by structured noteworthy domain facts (first greeter pattern, unusual quiet, weather/mood pairing, long preening, a meaningful new social interaction). The text generator must be deterministic/reviewable and use safe named facts, not arbitrary model output. Enforce spacing: target one entry every few days for a regular account, allow exceptions for genuinely notable events, and use a per-aviary rate limiter plus deduplication of similar facts. A notebook entry can observe birds and scene but must never report attendance, streaks, raw traits, event timestamps as a feed, or visit-frequency behavior.

Schedule new-bird eligibility entirely from aviary age and a configured quiet cadence—never clicks, presence totals, subscription, social activity, or performance. When eligible, offer a species arrival within the product flow without catalog selection; enforce two starters and maximum seven server-side. Names are suggested/user-assigned and independently renameable; species availability has no rarity economy.

## 6. Client rendering, interaction, and visual composition

### 6.1 Scene architecture

Build the aviary as an isolated scene component with a declarative state adapter and a rendering engine (Canvas/WebGL or DOM/SVG hybrid chosen by prototype performance). The chosen engine must support crisp semantic focus targets and captions over a performant animated scene. Keep account/settings/notebook/invitation pages outside the scene bundle via route-level code splitting.

Use a scene graph with these layers, from back to front:

1. day/night sky and soft distant foliage;
2. middle plane and three semantic perch-zone anchors;
3. bird silhouette/plumage/action nodes, each with stable bird ID and accessible focus proxy;
4. restrained local weather effects;
5. occasional client-only foreground leaf/branch/feather ornament;
6. optional naturalist call captions and keyboard focus outline;
7. a thin top bar above—not within—the scene.

All layout derives from normalized scene coordinates and safe viewport constraints. On narrow screens compress horizontal spacing and adapt perch anchors without cropping, hiding, or allowing a bird to leave frame; on wide screens increase breathing room. There is no scene pan/zoom/scroll. The Field Notebook itself may scroll as a separate panel/surface; its virtualized rows must not retain unbounded rendered nodes.

### 6.2 Snapshot interpolation and first paint

Treat a snapshot as an authoritative target state with sequence/version and server time. The client estimates render time only for interpolation, computes pose phase from the snapshot seed, and eases toward the next snapshot without writing back positions. On a changed perch, animate a short mood-appropriate transition; on reconnect after long gap, choose a gentle cross-fade/reposition rather than an implausible long client-simulated flight. A suspend/visibility return forces fresh snapshot reconciliation before extrapolation resumes.

The initial renderer should paint an existing still/mid-action pose from bootstrap fields before optional detail/physics/audio modules initialize. Do not gate it on the notebook, weather effect assets, settings, full species library, or audio permission. If bootstrap is unavailable before the first-paint target, paint only a quiet field and retry with bounded backoff; do not display a spinner, modal, generic app loader, or entry transition.

### 6.3 Motion and day states

Map mood and public render traits to a small composable library of micro-actions: preen, scan, head tilt, weight shuffle, call posture, rest, approach, and mild response. Combine deterministic phase offsets with bounded random variation so birds do not synchronize. A wary bird tends to back perch/scans; content preens; curious orients toward novelty; drowsy sits low/fluffed. Ambient ornaments are generated at idle cadence from local seeded RNG and are discarded after their animation; they are never ticked, stored, or networked.

Interpolate local-day color continuously, use warm evening/quieted behavior, and leave one configured night-active species eligible to move/call at night. The explicit settled view uses a slow few-second lighting/mix change. Clicking anywhere within five seconds reverses it; after that, normal re-engagement behavior is defined by fresh snapshot and interaction state.

The top bar contains only account/settings, accessibility, Field Notebook, offer, and settle controls as permitted. It fades nearly transparent after a few seconds of cursor stillness and returns on pointer movement or keyboard activity. Do not add labels/badges/tooltips into the scene; use accessible names and visually available labels in the chrome where needed.

### 6.4 Interaction and focus model

Pointer/tap and keyboard hit targets use the same action dispatcher. Clicking/tapping/focusing a bird enters listen-in; activating it again, focusing another bird, clicking empty scene, or leaving bird focus disengages. Keyboard order enters top-bar controls predictably, then first bird; arrow keys move birds in visual/order-aware sequence; Enter toggles listen-in; Escape exits. Use a high-contrast, calm focus outline visible across palettes and a logical reduced-motion equivalent.

Offers originate only from top bar, open a compact keyboard-operable chooser, and send a one-time intent. Render outcome from the authoritative response/event/snapshot; do not optimistically alter traits or invent acceptance. Settle is top-bar accessible and has the five-second scene undo. Account, invitation, and accessibility settings are matter-of-fact pages/panels with normal focus trapping and return-focus behavior.

## 7. Procedural audio, captions, and fallback

Define each species with a compact call grammar: motif primitives, pitch/rhythm/envelope ranges, timbral synthesis parameters, spacing, and textual descriptor grammar. Derive each bird’s stable signature from its stable ID/species seed and vary individual calls from mood, vocal-frequency trait bucket, ambient state, and server-supplied opportunity seed. This preserves recognition across weeks while preventing exact repeats.

Implement WebAudio synthesis with a small reusable voice pool, parameter automation, gain buses, and a scheduler looking only a short horizon ahead. Avoid allocating a new graph/buffer per chirp. A global ambient bus holds the chorus; each bird has a bus; a listen-in ramp gradually raises the focused bus while attenuating—but never silencing—others. Apply equal-power/parameter ramps on engage and disengage so the result feels like attention moving, not tracks switching. Handle simultaneous calls with voice limits and perceptual prioritization while preserving a seven-bird recognizability ceiling.

On browser autoplay restrictions, make a user gesture enable audio without blocking the visual aviary or presenting a coercive prompt. If WebAudio is unavailable, denied, suspended beyond recovery, or hardware fails, run the aviary in graceful silence with call captions enabled by default. Do not download recorded clips under any fallback condition. Surface audio failures only as aggregate error counts and a usable settings state, never as a disruptive in-scene alert.

When captions are enabled, derive text from the exact selected procedural motif/envelope at runtime (for example, a soft three-note rise), position it near the calling bird without covering controls, and fade it in/out with the rendered call. Captions must remain readable at AA contrast, obey reduced motion, avoid queueing/collision overlap, and have a semantic alternative for screen readers rather than duplicating every caption in a noisy live region.

Test audio with deterministic seeded motif renders, no-exact-repeat windows, signature distinguishability checks, listen-in ramp timing, seven-bird stress, lifecycle cleanup, background/suspend recovery, AudioContext denial, and fallback captions. Profile output/memory for a 30-minute session and verify voice nodes/buffers are bounded and disposed/reused.

## 8. Accessibility design and verification

Accessibility is a parallel primary experience, not a final ARIA pass. Produce a shared state-to-language formatter whose input is the same public snapshot/procedural event data used by rendering, with templates reviewed for naturalist, lowercase, present-tense specificity. It must never disclose numeric traits or turn mood into a raw state list.

### 8.1 Narration behavior

Provide an intentional screen-reader narration region/controller with:

- an initial concise naturalist scene observation after first meaningful snapshot/greeting;
- idle updates roughly every 30–60 seconds only when the scene meaningfully changes;
- a prioritized but brief naturalist observation for host-initiated greeting, valid offer result, and settle;
- deduplication, interruption policy, and queue caps to avoid flooding the reader;
- no narration for every animation frame, presence heartbeat, raw API update, or visitor state transition.

Represent birds as accessible, named interactive controls with role/name/instructions appropriate to the listen-in action, without exposing invisible personality values. Offer and settle have explicit semantic buttons and status feedback. Test with at least VoiceOver/Safari, NVDA/Firefox or Chrome, and a keyboard-only path. Validate that narration remains useful when visual captions/audio are off and that system errors use direct copy.

### 8.2 Reduced motion and visual access

Default to `prefers-reduced-motion` and allow an explicit persistent override. The reduced-motion rendering adapter keeps all birds, scene states, day color shifts, audio, captions, mood/drift, and notebook behavior, but replaces frame-by-frame micro-action with slow cross-fades among still poses and perch-to-perch cross-fades. Remove leaf/feather drift and slow color changes. Do not merely pause a live animation loop or show a static illustration.

Maintain WCAG AA minimum contrast for every user-copy surface in all morning/evening/night palettes: top bar, visible narration, captions, focus outline, setting controls, forms, errors, and visit-unavailable state. Run automated contrast tests against palette tokens and manual tests over all dynamic backgrounds. Respect text scaling, zoom, narrow viewports, target sizes, and color-independent state cues.

## 9. Security, privacy, and operational boundaries

Use secure, HTTP-only, same-site session cookies or equivalent tightly scoped token storage; rotate refresh credentials; hash opaque auth/invite tokens; protect magic-link and invitation consumption against replay; rate-limit email endpoints; protect state-changing routes with CSRF mitigation where cookie based; and authorize every resource by account/aviary/invitation scope. Log opaque request IDs and safe error classes only. Scrub email, raw tokens, personal names when feasible, notebook prose, bird state, and interaction payloads from application logs, traces, error reports, and metric labels.

Enforce a data-access boundary in service accounts and network policy:

- simulation service can read/write its account-scoped canonical store and append safe operational counters;
- telemetry collector receives only allowlisted aggregate metric events (request/latency/error/render-frame/audio error/tick timing plus anonymized session-duration histogram bin);
- analytics/ML identities have no credentials, replication, table access, or export path to simulation/event/notebook data;
- email service receives verified delivery address only for the current transactional message and does not receive bird or interaction details;
- support/debug tools use a redacted account lookup workflow and cannot browse private aviary history by default.

Threat-model magic-link forwarding/replay, stolen session tokens, invitation forwarding, invitation revocation during an active visit, cross-account object access, duplicate event delivery, malicious presence spam, client clock manipulation, renderer/audio denial of service, and accidental PII in observability. Include authorization and privacy-boundary tests in CI, not documentation-only controls.

## 10. Performance budgets, telemetry, and quality gates

Establish explicit release gates:

- initial JavaScript at first paint is under 2 MB gzip, with CI bundle analysis and a failure threshold;
- on a representative mid-tier mobile device/4G profile, first real bird is visible in under 500 ms at p75/p95 targets agreed before launch; test cold authenticated and warm paths separately;
- idle scene remains 60 fps on a five-year-old mid-range laptop under two and seven bird scenarios, with separate normal and reduced-motion measurements;
- a 30-minute session shows no positive memory growth trend beyond a tight, documented noise tolerance, including audio, snapshots, notebook opening/closing, visibility cycles, and caption usage;
- simulation tick p99 remains below 5 seconds, with alarm well before user-visible degradation;
- snapshot payloads remain kilobytes, versioned, compressed, cacheable only within correct authorization boundaries, and contain no hidden traits or unnecessary history.

Instrument synthetic browsers from common geographies for availability, auth/bootstrap latency, first-bird render, snapshot latency, and renderer failures. Instrument aggregate real-user measurements only: navigation/first-bird timings, frame-time buckets, audio-context error category, generic API/tick latency/error counts, and anonymized session-duration histograms. Use coarse buckets and privacy-reviewed schemas. Do not include account IDs, emails, bird IDs, names, interaction events, notebook facts, personality values, exact presence, invitation identity, or per-account dimensions anywhere in RUM/metrics.

Add dashboards for aggregate system health and alerts for tick p99, elevated bootstrap failures, auth delivery/consume failures, visitor authorization errors, error-rate changes, first-bird regression, prolonged frame degradation, and audio fallback spikes. Dashboards must not permit segmentation by account or bird. Use traces only with redacted opaque request IDs and short retention.

CI should include unit tests for deterministic simulation, property tests for monotonic bounded drift and event order, migration tests, API contract/authorization tests, browser integration tests, accessibility scans plus assistive-technology manual test scripts, visual regression tests for day/weather/mood/reduced-motion layouts, audio deterministic/memory tests, load tests for tick/snapshot contention, and privacy schema linting that rejects prohibited telemetry fields.

## 11. Incremental rollout plan

### Phase A — foundations and calibration harness

Implement account IDs/auth/session revocation, core schema/migrations, event acceptance/idempotency, simulation cursor/locking, a deterministic simulation library, snapshot contract, and a non-production calibration harness. Build fixture aviaries for regular presence, absence, overlapping devices, visitor-only observation, offers, day/night/weather, and seven-bird progression. Establish trace-redaction/telemetry allowlists and the performance test environment before feature work spreads.

### Phase B — private internal vertical slice

Ship two starter birds, state bootstrap, initial renderer, server tick, host presence, mood/idle projection, basic calls/listen-in, settle, and accessibility narration/reduced motion to a very small internal cohort. Add notebook only once structured facts and rate limits exist. Use synthetic test accounts and synthetic monitoring for drift calibration—not staff/pilot interaction histories aggregated as product analysis. Verify first-frame behavior, canonical multi-device state, one-time links, and no client personality writers.

### Phase C — controlled external beta

Enable magic-link onboarding/adoption/naming, field notebook, offers/cooldowns, export/deletion, full keyboard/caption support, and account settings for a small opt-in cohort. Start at two birds and make later adoption eligibility intentionally unavailable until tick, audio, and long-session metrics are stable. Gate number of birds by aviary age only when enabled; ramp eligibility to three, then additional age cohorts, measuring aggregate rendering/audio/tick health while keeping the product free of user-facing progress framing.

### Phase D — visits and broader ramp

After host-only privacy guarantees and account controls pass review, enable invitation creation for a limited host cohort, then expand. Test revocation on every snapshot, invitation expiry, visitor render-only controls, log transparency, and silent-default notifications. Do not ship social discovery or co-presence as a “follow-up.” Expand account rollout progressively with rollback switches for authentication delivery, server tick changes, renderer versions, AudioContext issues, and visit handling.

### Phase E — general availability and ongoing safety

Open v1 only when budgets pass across required browser versions (last two Chrome, Safari, Firefox, Edge), accessibility scripts are signed off, aggregate operational alerts are staffed, privacy access controls are audited, and deletion/export drills succeed. Unsupported browsers receive only the direct system-surface explanation. Continue controlled calibration changes through versioned simulation configuration, canarying on synthetic fixture accounts before applying to live state; never recompute or replace historical personality vectors to “correct” calibration.

## 12. Risks, decisions, and mitigations

| Risk | Why it harms the product | Prevention and response |
|---|---|---|
| Drift is too fast, too slow, or negatively tied to absence | Turns care into optimization or guilt; the core weeks-long relationship fails. | Versioned low-pass calibration, synthetic outcome tests at 1-week/3-week horizons, property test no negative neglect drift, feature-gated changes, no retroactive vector rebuild. |
| Presence overcounts inactive tabs or can be spoofed | Population-wide silent acceleration corrupts personality and trust. | Require simultaneous visible + focused + recent local activity; coarse bounded server validation; fail closed; no visitor events; abuse limits. |
| Multi-device races lose personality updates | Users encounter subtly inconsistent birds. | Server-only writes, append-only ordered events, per-aviary transactional cursor/lock, idempotency, no LWW/API patches, conflict/concurrency load tests. |
| Tick outages/retries duplicate or skip time | Canonical continuity breaks even without clients. | Cursor/tick transaction, deterministic seeds, retry-safe intervals, catch-up strategy, tick p99/lag alarms, operational replay drills. |
| First paint reads as loading | Violates the “already continuing” conceit immediately. | Compact bootstrap/read path, mid-action pose fields, bundle gate, quiet-field fallback only, mobile synthetic first-bird tests. |
| Audio becomes looped, harsh, or leaks memory | Calls lose recognizability/aliveness or degrade long sessions. | Grammar/signature tests, short-horizon scheduler, gain ramps, reusable voice pool, 30-minute profiling, silent captioned fallback only. |
| Reduced motion is a stripped static fallback | Excludes users from the product's actual quality. | Separate cross-fade renderer and visual regression suite; test at build time with normal feature parity. |
| Screen-reader updates flood or become status dumps | Accessible experience becomes unusable or emotionally flat. | Shared prose formatter, 30–60s cadence, meaningful-change dedupe, prioritized bounded event queue, manual assistive-tech testing. |
| Notebook turns into generic logs or engagement record | Destroys naturalist voice and reintroduces behavioral tracking. | Structured observation templates, rate limiter/deduper, content review tests banning attendance/raw-stat phrasing. |
| Invitation routes become accidental social platform | Violates privacy and changes simulation meaning. | Per-invite emails, constrained visitor claims, render-only route, no graph/profile/discovery schema, immediate revocation checks, product scope tests. |
| PII or relationship data reaches telemetry | Breaks explicit privacy commitment and creates irreparable data exposure. | Synthetic UUIDs, encrypted email single-home rule, telemetry schema allowlist/lint, separate credentials/network boundary, redaction tests and access audit. |
| Browser variations/AudioContext policy cause degraded experience | Fallback can look broken or inaccessible. | Required-browser matrix, progressive audio enablement, caption-first silent fallback, unsupported-browser system surface, canary monitoring. |
| Temptation to add toast, stats, streaks, or extra chrome | Reframes the product as an app/game rather than a place. | Product acceptance checklist/code review guardrails, UI inventory tests, forbid the data surfaces as well as UI, owner sign-off for any new user-facing metric. |

## 13. Definition of done

V1 is ready only when a signed-in person can open two devices and see one continuous canonical aviary; a return produces a varied bird-led greeting rather than a welcome UI; host-qualified presence alone creates slow, monotonic, invisible numeric drift; birds retain stable identity/mood across sessions; snapshot-driven rendering presents a bird already in motion; calls are synthesized and listen-in gently rebalances rather than mutes; offers/settle/notebook follow their bounded quiet semantics; accessibility users receive a designed living aviary; a visitor can only observe a host’s actual scene and revocation stops that observation promptly; and all stated performance, privacy, deletion/export, and browser support gates pass.

Final release review should explicitly inspect the negative space: no raw trait UI, no gamification data/UI, no absence punishment, no recorded audio, no scene controls/chrome, no native-client commitments, no social graph/co-presence/public discovery, and no per-account interaction data in telemetry. If any item is present, it is a v1 scope failure even when the surrounding implementation functions.
