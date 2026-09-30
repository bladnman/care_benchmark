# Pocket Aviary — v1 implementation plan

## 1. Delivery contract and governing invariants

Build a browser aviary whose birds have durable identities and whose behavior continues while no browser is open. Ship two starter birds, six species, a hard seven-bird ceiling, magic-link accounts, one canonical aviary per owner, multi-device viewing, all four interaction modes, the read-only field notebook, optional invitations, and the complete accessible experience together. The implementation work described here is future work; this deliverable is a plan, not product code.

This plan is based on `prd/1-START_HERE.md` and all nine PRD documents it lists. Where terminology conflicts, `concepts.md` governs. The separate visual-design document mentioned in the PRD was not supplied in the permitted inputs; producing its concrete palette, poses, and contrast tokens is a delivery task below rather than an assumed dependency already satisfied.

Treat the following as architectural constraints and release gates:

1. A bird's UUID, saved personality vector, and call identity survive renames, migrations, device changes, outages, and long absences. Only the simulation worker updates personality, through nonnegative additive deltas against its existing saved value.
2. Presence requires visible document AND focused window AND recent trusted pointer movement or keypress. Open tabs, visitors, polling, and audio playback alone never qualify. Overlapping owner devices count once.
3. Simulation advances on the server approximately every minute even with zero clients. Clients render time-indexed plans and interpolate; they do not run mood transitions, compute drift, or reconcile competing bird states.
4. Calls are generated procedurally in the browser. There are no recorded-call assets or recorded fallback. Call identity remains recognizable as mood and expression change.
5. Normal return begins with birds already mid-action and one bird noticing within one to two seconds. No entry sequence, greeting text, spinner, absence counter, or announcement accompanies it. The first adoption's soft fly-in is a distinct, one-time exception.
6. Absence has no negative personality delta, suffering, hunger, chore, or loss. Transient activity may quiet naturally; saved expressiveness does not decrease.
7. Bird behavior and interaction history remain in the owner's simulation domain. Operational analytics cannot query that database or receive its fields. No actual-account drift averages, engagement rankings, or training datasets exist.
8. Reduced motion, narration, captioning, keyboard operation, and AA text contrast ship with the normal scene. They preserve its affective quality.

Exclude native apps, payments, passwords/SSO, multiple aviaries, shared accounts, scene customization, bird placement controls, catalog selection, scores, progress meters, streaks, badges, leaderboards, public discovery, profiles, comments, chat, avatars, co-presence, and bird-care mechanics. No recurring re-engagement campaigns or aviary activity notifications exist except the explicit opt-in visit exception addressed below. Do not build underlying tables or metrics for these excluded features.

## 2. Decisions for ambiguities in the supplied specification

These decisions make the implementation executable without asking the planner's human for clarification. Record them in the engineering design review and retain the related contract tests.

| Question | v1 decision |
|---|---|
| Hidden numerical traits versus an export explicitly containing current vectors | Hide trait numbers from every product view, narration, caption, ordinary API, and client debugging surface. The user-requested, machine-readable account export includes exact vectors as `accounts_sync.md` requires. Treat this as a narrow data-portability exception, disclose its scope in the export UI, and do not turn it into a statistics viewer. The PRD's absolute wording and export requirement conflict; this is an explicit interpretation, not a claim that the conflict disappears. |
| No notifications versus the optional visit-notification setting | Ship the social document's small setting, off by default and absent from onboarding. If deliberately enabled, send one quiet transactional email per redeemed visit, with a delivery throttle and no badges, toasts, push, reminders, or engagement copy. This is the sole optional activity-mail exception; authentication, verification, invite, and export emails are system transactions. |
| Four top-bar icons versus a top-bar settle affordance | Keep exactly account/settings, accessibility, notebook, and offer icons. The offer affordance opens a small actions panel containing the three offer choices and a separately labeled `settle` action. This makes settle reachable from the top bar without a fifth icon. |
| Keyboard focus starts listen-in versus Enter triggers it | Bird focus reached through keyboard navigation engages listen-in. Enter explicitly engages the same interaction idempotently. Escape disengages while retaining focus, with re-engagement on Enter or a subsequent bird-focus change. Pointer click toggles independently of its incidental DOM focus event. |
| Local time on devices in different timezones versus a single shared day/night state | Save one IANA timezone on the account, initialized from the first owner's browser, and use it on the server and all clients, including visitors. It is editable in settings; other devices do not silently overwrite it. Traveling owners can change it there. This interprets local time as the owner's chosen local timezone and avoids competing canonical mornings. |
| Prompt reactions versus minute-scale event consumption | Persist an accepted interaction and a server-authored short presentation plan together. Return that plan immediately for the visible gesture; the next normal tick consumes the event and applies canonical mood/drift effects. Clients never speculate a new mood or vector. All snapshot readers see the same accepted presentation plans while they are active. |
| Settled state across devices | Settling ends the initiating view's presence and creates an aviary-wide evening presentation override. It lasts until that view closes/expires or an owner actively re-engages. Another device's polling alone cannot undo it; an actual owner interaction can. Other devices' legitimate presence is neither erased nor double-counted. |
| Visitor email storage versus email stored only on an account record | Reuse the identity/account record as the sole encrypted email store, including a provisional recipient identity for someone without an owner account. Invitations and logs reference recipient UUIDs. A provisional visitor gets no aviary and no owner permissions; becoming an owner later creates their one aviary atomically. |
| One-time invitation and unspecified active-session lifetime | Unused links expire at 30 days. Redemption consumes the link once and grants one read-only browser visit, lasting at most 24 hours, ending earlier on revocation or 30 minutes without a visit heartbeat. Reopening that consumed link cannot start another visit. |
| Adoption pacing | Make slots available at aviary ages 90, 180, 270, 365, and 540 days, for a ceiling of seven including starters. Availability depends only on creation time, never attention. A quiet offer waits for the owner to accept; missing it loses nothing. |
| Silence preference and evolution | Muting never lowers a trait or excludes a user from growth. Qualified presence and focused attention count equally with audio, captions, or silence. Volume is an output preference, not a neglect signal. |

Initial numerical constants throughout the plan are versioned build parameters. Synthetic calibration can change them before launch within the stated invariants; it cannot introduce negative drift, increase seven, or substitute real-user behavioral analytics.

## 3. Service architecture and ownership

Use a TypeScript web client, native Canvas2D drawing with compact SVG bird rigs, native WebAudio synthesis, a TypeScript HTTP application, a separate simulation-worker process, and PostgreSQL as the transactional authority. Use an object store for short-lived exports and a transactional email adapter. Pin supported dependencies during implementation; this plan does not depend on a particular framework release. Keep the scene runtime independent of the DOM component layer used for settings and panels.

Deploy HTML assembly at a regional edge near users with a private connection to the canonical application. CDN-cache public, content-addressed rigs, motifs, and JavaScript; never place personalized HTML or snapshots in a public/shared CDN cache. The edge assembles a small private initial snapshot with the HTML. For the first release use one writable database region with tested failover; scale worker shards before adding regional writes. No client-to-client protocol, CRDT, websocket dependency, or eventual personality merge is necessary.

### Process boundaries

| Component | Responsibility | Forbidden responsibility |
|---|---|---|
| Identity module | Magic links, encrypted email, sessions, verified email changes, recipient identity resolution | Simulation inputs in email/logging tools; email as a partition key |
| Owner HTTP module | Authenticated snapshots, idempotent events, names/settings, adoption, notebook reads | Accepting vectors or client mood/position writes |
| Visit HTTP module | Atomic link consumption, read-only snapshot authorization, visit heartbeat/log | Owner event submission, greeting generation, presence accounting |
| Simulation worker | Ordered event consumption, persisted filters, additive drift, mood, weather, behavioral and call plans, notebook observations | Analytics emission of bird state; rebuilding vectors from history |
| Presentation projection | Combine saved world state with accepted short gesture plans, materialize rendering descriptors | Inventing personality or bypassing invitation revocation |
| Browser scene runtime | Draw timed descriptors; synthesize call descriptors; maintain local audio mix, accessibility output, input qualification | Choosing semantic outcomes or writing canonical state |
| Operational telemetry | Timings, counts, errors, resource health using a closed metric schema | Simulation database access, behavioral payload logging |

Owner API credentials can append events and mutate approved metadata, but cannot update personality columns. The worker database role alone has that permission. A snapshot serializer with explicit allowlisted fields strips vectors, signal filters, private cooldown internals, invitation details, and account identifiers not needed by the browser. Make permission and serializer checks integration tests rather than conventions.

Keep authentication, invitation authorization, and event admission authoritative at the HTTP service. A worker cannot resurrect a soft-deleted account, an expired visit, or a revoked device session. Use a transactional outbox for required emails and export jobs; include synthetic IDs only, resolving the encrypted email in the identity/mail boundary just before delivery.

### Frontend/domain separation

Create internal modules for `simulation`, `presentation-contract`, `scene-runtime`, `call-runtime`, `accessibility-output`, and `system-panels`. Presentation contracts carry timestamps, seeds, perceptual styles, poses, and call structures, not personality vectors. A shared pure pose resolver evaluates the supplied descriptors at a time; using it in server first-frame rendering and client rendering does not give clients a simulation engine. Render-only leaves and feathers have no canonical records.

## 4. Persistent and transient data model

Use random UUIDs for accounts, aviaries, birds, sessions, invitations, and notebook entries. Use UTC database timestamps and explicit IANA timezone names. Never reference email in foreign keys, log keys, task IDs, URLs, shard keys, or telemetry. Encrypt email with an envelope key outside the database. A secret-keyed email lookup digest is allowed only as a restricted uniqueness/lookup index on the account record; it is not an external identifier and never leaves identity storage.

| Record | Required fields and constraints |
|---|---|
| `account` | `id`, sole encrypted current email, restricted lookup digest, encrypted pending-email value/expiry on the same record, verification state, identity/owner activation state, IANA timezone, settings revision, creation time, nullable deletion-request/purge times. One owner account has exactly one aviary. |
| `device_session` | UUID, account UUID, hash of opaque session secret, creation/last-seen/expiry/revocation times, user-readable coarse browser/device description. Never raw user-agent strings in aggregate telemetry. |
| `magic_link` | UUID, account UUID, keyed token digest, request purpose, expiry, consumed time. Fifteen-minute expiry; atomic single consumption. |
| `aviary` | UUID, unique owner UUID, creation time, world revision, presentation revision, last/next tick times, last processed event sequence, deterministic world seed, engine/content versions, current weather, local-day anchor, active settle override, sparse-notebook gating state. |
| `bird` | Stable UUID, aviary UUID, species/rig version, user name, adoption time, five persisted traits, five persisted filtered signals, mood and intensity/timer, stable call-signature parameters/seed, current behavioral plan, rendered plumage style, interaction cooldown deadlines. Renames update only name/revision. |
| `interaction_event` | Aviary UUID, per-aviary increasing sequence, event UUID, owner/session/view UUIDs, type, optional bird UUID, server receipt time, admitted bounded interval/payload, event-schema version, persisted server reaction/gesture plan where applicable. Append-only until retention expiry or account purge. Unique `(aviary_id,event_id)` and `(aviary_id,sequence)`. |
| `presence_coverage` | Owner/aviary UUID, UTC minute bucket, 60-bit qualifying-second mask; optional per-bird listen-in masks. These are private simulation inputs, unioned across sessions and retained only for the rolling input window. |
| `view_lease` | Session/view UUID, open/closed state, last heartbeat, most recent qualified interval, current listen-in bird, settle state, and greeting epoch. Short-lived and server-expiring; no owner visit calendar. |
| `notebook_entry` | UUID, aviary UUID, observed time/local date, observation category, immutable naturalist text, facts needed for export/audit, content version. Cursor index `(aviary_id,observed_at,id)`. Retained for account lifetime; no user edit/delete endpoint. |
| `bird_observation_summary` | Per-aviary/bird facts such as recent greeting order, perch tendencies, noteworthy call combinations. Bounded windows for notebook selection; never owner attendance streaks or analytics input. |
| `adoption_offer` | UUID, aviary UUID, age milestone, system-chosen species, availability time, accepted/deferred status. At most one visible pending offer; no rarity or score. |
| `invitation` | UUID, host UUID, recipient UUID, hashed link token, creation/unused-expiry/redemption/revocation times. Tokens are never stored in plaintext. |
| `visit_session` | UUID, invitation UUID, hash of read-only secret, started/last-seen/ended/absolute-expiry times. Refer to recipient identity for the host's private log. |
| `export_job` | UUID, account UUID, status, consistent-read revision, private object reference, download-token digest, expiry. Artifacts expire within 24 hours and are excluded from analytics. |

For traits use normalized `[0,1]` storage, constrained finite numeric values, and initial values primarily in `[0.15,0.45]` with species-specific narrow bands. Retain substantial headroom. Call signature, silhouette, and bird UUID remain fixed; trait drift changes expression, not identity. A seventh bird can share a species with another because the species pool has six; distinct per-bird timbre/rhythm seeds preserve recognition without inventing rarity.

Persist the personality vector itself and filter accumulators. Do not derive them from raw history during login, deployment, or restore. Keep a small private per-tick delta/checkpoint record for recovery verification, not as the source from which vectors are normally rebuilt. An additive delta and the event high-water mark commit in the same transaction.

Retain raw events for 30 days after successful consumption, deleting unconsumed events only after resolution or account purge. Keep rolling coverage for 48 hours so a 24-hour input window and late arrivals fit; remove expired buckets. Keep private observation summaries only as long as their observation window requires. Persistent notebook records are sparse and unlimited in time, not preloaded into the browser. Account deletion overrides every normal retention policy.

## 5. HTTP contracts and authorization

Use same-origin JSON APIs, TLS, opaque HttpOnly/Secure/SameSite session cookies, CSRF protection for mutations, strict payload schemas, and bounded request sizes. Links exchange a high-entropy secret once for a cookie, then redirect to a clean URL. Never log link parameters, request bodies, snapshot JSON, names, or decrypted email. Use uniform request-link responses to avoid account enumeration. Error payloads are `{code, message, retryable, request_id}` with matter-of-fact copy and no raw exception messages.

### Owner and system endpoints

| Method / route | Contract |
|---|---|
| `POST /api/auth/links` | Request a magic link for an email. Generic response; initial throttle three requests per email per 15 minutes plus coarse network abuse limits. Store expiry at 15 minutes. |
| `POST /api/auth/consume` | Atomically consume token and issue a revocable device session; redirect handling is outside JSON response. Replay/expiry uses the same clear failure surface. |
| `GET /api/account` | Account/system settings and device list without simulation internals. Pending-deletion accounts receive a restoration surface. |
| `PATCH /api/account/settings` | Audio, narration, captions, reduced-motion override, timezone, opt-in visit mail. Require `If-Match` settings revision. Conflicting metadata edits receive 409 and a reload prompt, not a vector merge. |
| `POST /api/account/email-change` and `POST /api/account/email-change/verify` | Stage encrypted new address, verify before atomic switch; old email remains valid meanwhile. Verification cannot create a second aviary. |
| `DELETE /api/account/sessions/:id` | Revoke a device immediately; current-session revocation clears its cookie and view leases. |
| `POST /api/account/export` | Idempotently request an on-demand snapshot job; email a short-lived authenticated download link to the verified address. |
| `POST /api/account/deletion` and `POST /api/account/restore` | Mark deleted immediately with purge at 30 days; restore on deliberate `I changed my mind` action during the window. |
| `GET /api/aviary/snapshot` | Private snapshot; conditional ETag over world/presentation/settings revisions. Authorize every request, including would-be 304 responses. |
| `POST /api/aviary/views` | Open/return owner view; idempotency key and return epoch; request greeting plan against saved current state. This records an observation opportunity, not presence credit. |
| `POST /api/aviary/events` | Bounded ordered batch of interaction events; return admitted event receipts and presentation plans. Unknown types, traits, positions, absolute mood, or foreign bird IDs are rejected. |
| `GET /api/aviary/notebook?cursor=...&limit=...` | Owner-only immutable entries, newest first, maximum 40 per page. No write, annotation, or per-user visit-history endpoints. |
| `PATCH /api/aviary/birds/:id/name` | Validated normalized Unicode name, 1–32 graphemes; revision conflict protection. Preserve bird UUID and all behavioral state. |
| `GET /api/aviary/adoption-offer` | Return an eligible system-selected bird offer, or none; never a numeric progress/countdown surface. |
| `POST /api/aviary/adoptions` | Accept a pending offer atomically; lock aviary, enforce seven including concurrent requests, create one stable bird, consume offer. Starter adoption is a separate account-bootstrap transaction creating exactly two. |
| `POST /api/aviary/adoption-offer/defer` | Leave availability intact and quietly suppress the card for 30 days; no expiring reward. |

### Visit endpoints

| Method / route | Contract |
|---|---|
| `POST /api/invitations` | Host explicitly supplies recipient email; identity boundary resolves recipient UUID; create invitation and transactional email. Off until this action; no global sharing flag. |
| `GET /api/invitations` | Host settings list outstanding/active invitations and historical visits, with recipient email resolved only for this authorized private surface. |
| `DELETE /api/invitations/:id` | Host-only revocation transaction. Mark invitation and active visit revoked, suppress any pending opt-in notification, invalidate visit authorization. No success toast. |
| `POST /api/visits/consume` | Atomic unused-link consumption after checking 30-day expiry and host/account status. Set only a read-only visit cookie. |
| `GET /api/visits/snapshot` | Read the host's canonical presentation projection using the same scene contract and current host timezone. Check invitation/session status before returning either 200 or 304. |
| `POST /api/visits/heartbeat` | Maintain approximate visit duration, not simulation presence. Separate route/schema/storage with no bird ID or activity flags. |
| `POST /api/visits/end` | Best-effort close; server timeout bounds duration if the browser disappears. |

Use owner-session and visit-session middleware with separate capabilities. A visit cookie cannot be promoted by changing route parameters or submitting owner event types. Visitors get local accessibility/audio-output settings but no bird listen-in, offer, settle, greeting event, notebook, adoption, or account mutations for the host. Focusing a visitor bird for narration remains descriptive and has no audio rebalance or owner event.

### Snapshot shape

Expose `schema_version`, `world_revision`, `presentation_revision`, `engine_version`, `server_now`, `valid_until`, `timezone`, lighting/weather descriptors, bird identity/name/species, current mood for internal rendering, perch/pose timelines, animation phase timestamps, palette styles, call descriptors, and active accepted gestures. Include notebook availability without unread counts and adoption availability only in the owner's relevant panel response. A visitor snapshot omits owner settings/account identity, cooldowns, notebook, and adoption information.

A call descriptor contains stable call ID, bird ID, start/duration, motif identity, per-call seed, timbre envelope, pitch contour and syllable timing needed to synthesize and describe it. It does not contain a normalized vocal-frequency trait. A motion descriptor contains segment start/end times, curve/pose identifiers, and current phase, not a client instruction to choose a mood or perch.

Snapshots should normally be a few kilobytes compressed; set a seven-bird envelope of 12 KB gzipped and prevent growth with event history. Include a 120-second behavior/call horizon. A tick at 60 seconds extends it while preserving already scheduled call IDs and ongoing segments; a reconnect does not replay expired calls.

### Interaction envelope

Each event has `event_id`, `view_id`, monotonically increasing device sequence, type, optional bird ID, and a small type-specific payload. Types are `presence_interval`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `settle_undo`, `reengage`, and `view_end`; view-open/return uses its dedicated route. The server stamps time and allocates the authoritative per-aviary sequence. Client timestamps are only bounded interval hints, never authority for simulation order.

An offer carries `seed|song_fragment|still_pool` and, for a song fragment, a library motif ID. The server chooses receiving birds, reaction, and optional plan from current saved state and cooldowns. Do not make the client submit `accepted=true`. A listen-in duration is determined from start/end and qualified view leases, not from an arbitrary client duration claim.

On retry with the same event ID, return the exact original receipt/plan without creating another log row, cooldown, mood effect, or drift input. Admit new events under a per-aviary transaction that allocates sequence, validates ownership, and reserves cooldowns/presentation plans; then release the lock. Return receipts within 200 ms p95 in the launch region. A lost response can therefore be safely retried without a second seed or pool appearing.

## 6. Presence, multi-device admission, and session lifecycle

### Browser qualification

Start with a 240-second recent-activity window, intentionally long enough for sitting quietly. Implement a tiny input-state machine independent of rendering and audio:

`qualified = visible && document.hasFocus() && trustedPointerOrKeyAge < 240s && !settled && ownerViewActive`.

Use trusted PointerEvent `pointermove` and actual keypress input, tracked with modern `keydown` handling for a pressed non-modifier key. Do not count focus events, clicks alone, media playback, polling, timers, synthetic events, scroll observers, or screen-reader announcements as activity. Do not record pointer coordinates or key values; retain only the monotonic last-activity timestamp. Mobile taps may trigger offers without establishing presence if they produce no qualifying movement; do not fabricate pointer movement to improve the metric.

Sample elapsed qualified intervals against a monotonic clock, truncate exactly on blur, hide, idle expiry, pagehide, settle, or session revocation, and submit every 15 seconds. A normal interval is at most 15 seconds; payload includes bounded elapsed/last-activity age and the qualifying flags. Use a best-effort final beacon for known completed time, with a CSRF-safe short-lived view credential. If it fails, never infer future attention from the last heartbeat.

On hidden tabs stop animation frames, caption timers, call scheduling, presence timers, and owner polling. On visible-but-unfocused tabs rendering can continue; presence must stop. At night or while muted, qualifying attention still counts. After laptop suspension or a render gap above two seconds, suspend presence accounting, discard unsubmitted uncertain elapsed time, pull a fresh snapshot, and restart from a newly validated interval. Do not credit the entire sleep gap.

If transport is offline, preserve only a small in-memory retry queue for event IDs already attempted. Do not persist an overnight backlog of presence. Intervals older than 30 seconds at admission are discarded; if the network cannot acknowledge, no lease is extended into the future. Explain a prolonged connection failure in the system surface without guilt about lost attention.

### Server qualification and deduplication

The browser's visibility/focus report cannot be cryptographically proven; this is an honest client protocol with bounds, not invasive attention detection. Server admission checks valid owner session/view, trusted-client schema, positive bounded duration, recent activity, visible/focused flags, monotonic sequence, and a receipt-relative timestamp envelope. Use server receipt time to anchor intervals and reject future or stale intervals; clip at activity expiry and view termination.

The tick unions accepted intervals into per-second coverage masks. Two windows, two devices, retrying pings, or overlapping intervals credit only new seconds. Listen-in masks are similarly unioned per bird, and intersected with qualified owner presence. Two devices focusing the same bird do not double its attention. Different birds may receive attention from the same owner window of time, but a shared per-aviary interaction budget prevents two devices multiplying total reinforcement.

Do not retrospectively rewrite past vectors when an interval arrives slightly late. Union its newly accepted coverage into the rolling input window at the next tick and apply any effect prospectively. Normal admission allows only the latest 30 seconds; no client can replay last week's attention. Saved vectors and tick watermarks make repeated delivery harmless.

### Return and greeting epochs

Opening a view or returning from a hidden state requests a greeting independently of qualified presence. The user may be quietly looking before moving the pointer; the bird still notices. Debounce repeated focus/visibility churn within five seconds using the same return epoch. Do not greet merely because a visible snapshot refreshes or an accessibility panel closes.

Use the last owner-visible interval/view departure to calculate absence length privately. Never show that number. Select a first greeter by a weighted draw using boldness, warmth, mood, recent greeting repetition, and species day/night activity. Guarantee one visual noticing plan within 0.6–1.6 seconds of a valid return on the target device; an optional call depends on mood and audio capability. Other birds may respond with randomized offsets, never an arrival chorus in unison.

The plan is assembled from head angles, gaze direction, step distance, response latency, call motif transformations, and ongoing pose phase. Short absences favor a glance; long absences favor re-orientation or approach. Save the selected plan/seed so retries are stable, but sample a new seed and continuous parameters for each actual return. No fixed three-animation rotation. Returning after two weeks preserves the same bird and vector; quietness comes from decayed short-term arousal, not distrust.

## 7. Server simulation and slow drift

### Scheduling and atomicity

Run each active owner aviary every 60 seconds, staggering initial due times by a stable UUID hash to avoid minute-boundary spikes. This includes aviaries with no connections. Soft-deleted accounts are suspended while preserving their state; that is an explicit account lifecycle state, not normal absence.

Dispatch due aviaries through a database due-time index or durable shard queue. Workers acquire the aviary row lock, verify a scheduling fence, then execute one transaction:

1. Load saved aviary/birds, prior tick time, persisted filters, and the ordered events after the committed watermark through a fixed captured high-water mark.
2. Admit valid intervals into coverage unions; resolve event outcomes using their stored server plans; expire leases, cooldowns, and finished presentation segments. Unknown event versions fail safely for that aviary rather than silently dropping attention.
3. Advance rolling signal windows and filter accumulators for elapsed server time; compute and apply nonnegative personality deltas to existing vectors.
4. Advance local-time mood processes, bounded neighbor influences, and persistent weather using deterministic seeded randomness.
5. Extend bird pose/perch/action and call plans, keeping existing IDs/segments stable. Evaluate sparse notebook candidates and age-based adoption availability.
6. Persist every changed bird, coverage/filter state, world plan, notebook entry, high-water mark, engine version, revision, next due time, and a private tick-checkpoint/delta record in the same commit.

A crash before commit has no effect; retry uses the same logical tick seed. A crash after commit sees the watermark and cannot reapply a delta. Event appenders and tick writers serialize on the same aviary boundary. Events admitted after the captured watermark are consumed next time. Database locks, not a best-effort distributed mutex, protect the single writer.

Keep scheduling delay and compute latency as separate metrics. Alarm if tick-computation latency p99 exceeds five seconds, as specified, and also if due-time lag p99 exceeds five seconds. Target ordinary seven-bird tick computation below 20 ms p95 before database contention. A one-minute cadence is not permission for five seconds of CPU work per aviary.

### Drift function

Use persisted nonnegative filtered input, not a target personality that decays toward baseline. For each trait `j`:

```
P = min(1, unique qualified presence seconds in last 24h / 900)
L_j = capped qualified listen-in reinforcement relevant to trait j
O_j = capped accepted/nearby-offer reinforcement relevant to trait j
u_j = min(1, 0.85 * P + 0.10 * L_j + 0.05 * O_j)
alpha = 1 - exp(-elapsed_hours / 48)
f_j_next = f_j + alpha * (u_j - f_j)
delta_j = k_j * elapsed_days * mean(f_j, f_j_next) * (1 - x_j)
x_j_next = min(1, x_j + max(0, delta_j))
```

The filter may fall during absence; the personality may not. Presence accounts for at least 85% of full-scale reinforcement. Tune `k_j` initially around 0.0045–0.0070 normalized units per full-input day, with species-specific narrow multipliers; set the exact launch values through the calibration gates below. The headroom factor slows saturation. This is an additive server-authored update to `x_j`, never an assignment of a client-provided absolute value or a reconstruction from events.

Interpret listen-in primarily as warmth/vocal reinforcement for the focused bird, with a maximum of 10 qualified minutes per bird per day. Offers contribute small curiosity reinforcement if accepted and small boldness reinforcement if near that bird, with at most three drift-relevant offers per bird per day. Further properly spaced offers may still create a visual response, but cannot accelerate slow drift. Presence affects all five traits for all birds, including birds the owner never clicks. Settle changes no slow trait.

Keep visual mapping subtle: increased boldness changes front-perch tendency, warmth changes greeting/call-response tendency, vocal frequency changes scheduling density, saturation changes plumage gently, and curiosity changes investigation likelihood. Apply a daily perceptual change budget so even a very long visit cannot produce an obvious same-session change. Never display its budget or a cap meter.

The 15-minute/day fixture is an initial definition of regular visits. Across the seeded species/trait distribution, aim for detectable numerical shifts after seven days (roughly 0.01–0.025 in affected traits) and perceptually distinguishable behavior/plumage after 21 days (roughly 0.04–0.075 where suitable). These ranges are calibration hypotheses, not promises to users. Test full daily input, irregular short visits, presence-only, interactions-only, muted/caption use, two overlapping devices, and two-week absence. A single ordinary session must remain below the experimentally identified visible threshold.

Calibrate only using synthetic accounts and generated trajectories. Do not compute actual-user mean drift or use account event logs to tune population behavior. Conduct blind comparisons of generated day-zero/day-seven/day-21 scenes and calls; use reproducible synthetic fixtures to inspect trait changes internally. Preserve no numeric trait dashboard in the shipped browser.

### Mood and ambient continuity

Finalize five moods for v1: wary, content, curious, drowsy, alert. Store mood, transition time, minimum dwell, and short-lived modifiers. Use a time-varying stochastic transition matrix influenced by canonical local hour, interaction outcomes, weather, saved personality, and nearby birds. Evaluate at tick boundaries with minimum dwell of several minutes; do not roll an independent new mood each time a snapshot is requested.

Use a daily-ish relaxation process (approximately 18–30 hours) that softens yesterday's transient mood toward the time-of-day distribution. It runs continuously; opening the tab or crossing midnight never resets every bird to neutral. Evening favors drowsy; morning favors alert/content. Wary can arise from ambient alarm/wind, not from owner absence. A nightjar-like species retains a nonzero late-night call probability while most birds rest.

Offers may nudge curiosity/content according to the stored reaction, not force happiness. Settle contributes a small expiring quieting modifier after its five-second undo window. Rain briefly reduces call density without decrementing the vocal trait. Wind can raise alertness or wariness according to temperament. Cap neighbor propagation so one alarm does not keep all birds wary indefinitely.

Maintain a nonzero species-appropriate ambient call/idle baseline even without presence. Recent interaction arousal decays over hours and can make a returning aviary quieter than its last active session; it does not erase slow personality. Test absence at identical time-of-day/weather to distinguish harmless arousal relaxation from an accidental negative-trait implementation.

### Behavior and weather plans

Choose perch zone probabilistically from mood and boldness, then select an available geometric slot with collision constraints. Preserve the meaning of front/middle/back; do not let responsive packing move a wary bird to the front merely to fit. Reserve flight endpoints on the server and publish timed curves. Idle plans mix preening, scanning, tilting, weight-shuffling, and resting using variable duration and phase, not a repeated fixed animation cycle.

Schedule rain two to four times per synthetic week, lasting roughly two to six minutes, and occasional soft wind. Persist event start/end/seed so every device sees the same rain. No real-world weather or location service is needed. Render-only leaves/feathers can vary per device and do not enter the event log.

### Catch-up, versioning, and recovery

If workers miss a few minutes, replay missed logical minute steps using saved state and unconsumed events. For a long outage, process bounded chunks with a scheduling fence: integrate persisted filter decay analytically between event/timezone/weather boundaries, then advance mood through bounded chronological substeps. This recovery advances saved state; it never regenerates the bird from historical events. Do not replace the always-running scheduler with lazy simulation on return.

Retain original effective times, never award invented absence presence, and never generate a flood of retrospective notebook entries. A recovering aviary can serve its last safe snapshot with a matter-of-fact stale-state explanation while catch-up completes. Protect the p99 tick metric by separately labeling recovery-work duration; recovery backlog is still alarmed, not hidden.

Store engine/content versions and deterministic seeds. Migrations preserve UUIDs and vectors exactly, validate monotonic bounds, and pin existing call signatures to their compatibility version. Back up saved vectors, filters, and watermarks; run restoration drills that compare exact vectors and identity. Do not use a new engine version as a reason to reseed birds.

## 8. Canonical sync and interaction presentation

Pull a snapshot on navigation, visible return, foreground resume after a long frame gap, and every 15 seconds while visible. Use ETags to make unchanged pulls cheap. Keep the authoritative read on the primary or a replica with an explicit revision fence; after an admitted event, a reader must not fall behind its receipt revision. Include `server_now` to estimate a smoothed clock offset; bounded local interpolation compensates for normal network delay without changing simulation time.

The view-open/event response contains short server-authored plans immediately, so the greeting or seed reaction need not wait up to a minute. Persist these plans with the accepted event and expose them in the snapshot projection. The next tick consumes the identical accepted outcome. For example, if the server decided Pip waits and Wren approaches the pool, neither the client nor the next tick rerolls the recipients. Mood may remain the previous saved mood until its tick; the gesture itself is a short response, not a client mood change.

These plans affect presentation revision and ETag, while tick commits advance world revision. Their authority is still server-side. The frontend reconciles by plan ID and effective time: an acknowledgment replaces the pending affordance state, never restarts an already playing animation; a later snapshot contains or expires the same plan. A visitor may observe an owner-generated reaction but cannot cause one.

Keep only local intent as local state: which bird this device listens in on, open panel, keyboard focus, volume hardware state, and transient request progress. Owner listen-in events affect future drift when consumed, but the audio mix itself is device-local. A phone focusing Wren must not seize the laptop's focused mix. Global saved audio/accessibility preferences sync through settings revisions; hardware audio availability and visitor output preferences remain local.

For simultaneous offers, per-bird server cooldown reservations decide admission in log order. An aviary-level transient-offer limit prevents two devices covering the scene with pools/seeds. For simultaneous renames/settings edits, return a 409 and current revision, then let the user retry the specific metadata edit. No personality conflict UI exists because clients cannot write it.

Never rewind to a lower revision. On resume, discard elapsed call IDs and resolve poses at the current server time. Interpolate a short unanticipated positional correction only when it is small; after a long suspension use the current pose directly rather than making the bird fly through an hour of history. Preserve identity and ongoing motion phase across DOM hydration, snapshot refresh, and reduced-motion switches.

If snapshots fail briefly, finish existing valid plans and continue bounded local breathing/preen pose evaluation. Do not invent new semantic calls or transitions beyond the supplied horizon. At horizon expiry reduce to last-known poses/quiet ambience and show a clear reconnect/reload surface in the system layer. Do not reset to starter birds, restore an old local vector, or silently report successful offers. A server timeout after an event write is resolved by its idempotent retry/receipt.

## 9. Interaction details and adoption

### Listen-in

Click/tap a bird to engage; click it again or empty scene to disengage; choosing another bird ends the prior interval and starts the new one. Keyboard focus follows the decision in section 2, moving out ends it, and Escape always releases it. Send start/end receipts, with a lease timeout if an end event is lost. Credit only intersected qualified attention, not the lifetime of a stale focused element.

Ramp the focused bus gradually over 1.5–2 seconds; lower others approximately 6–9 dB with a nonzero floor. On disengagement return over about two seconds. Switching birds ramps both buses and never cuts or mutes the rest of the aviary. The same behavior works with captions and narration even if audio is unavailable; it still represents focused attention.

### Offers

Open the top-bar offer panel using pointer or a scoped `o` shortcut when scene focus is active and no text input is being edited. Expose seed, song fragment, and still pool as keyboard-operable choices; the small song library has three to five short procedural melodic motifs, not recorded audio.

Reserve a 180-second per-bird offer cooldown server-side, initially. On seed/water offers choose an unobtrusive scene location and determine candidates by distance/perch, mood, and curiosity. Store each bird's approach/watch/bathe/drink/ignore outcome with its response latency. A drowsy bird may simply watch or remain at rest; copy never calls that a failure. Song motifs play softly, with joining, waiting, quieting, or a contrasting call selected from vocal disposition/mood.

Limit active offer ornaments to one pool and a small bounded seed gesture; expire them without housekeeping. When all relevant birds are cooling down, the panel uses restrained naturalist copy such as `the birds are still with the last offering`, without timers, red errors, or a progress bar. A network/authentication failure uses matter-of-fact copy instead. The server reserves accepted offer outcomes in receipt order and releases no personality control to the client.

### Settle and undo

Settle starts an evening-light transition over about five seconds and gently lowers calls. Send the event and stop initiating-view presence immediately; a submitted interval is cut at that time. Delay its small semantic quieting modifier until the five-second undo window has elapsed, so an undo does not require restoring an old mood over unrelated changes.

Any scene click within five seconds cancels the same settle ID, reverses the lighting smoothly from its current value, and reopens presence qualification. Provide the equivalent focused-scene keyboard activation for accessibility. After five seconds active owner re-engagement still restores normal lighting and begins a new presence window; it is no longer an undo of the previous mood signal. Closing a tab ends its view/presence, with a best-effort close event and lease timeout fallback. Neither closing nor settling changes slow traits or produces a recovery chore.

### Starter and later adoption

On first verified owner activation create the aviary and two system-selected distinct species with stable UUIDs in one transaction. Offer suggested names and allow user names, but no catalog or avatar configuration. The brief first empty field and soft fly-in belong to this initial adoption flow only. If the flow reloads after creation, show those same birds rather than create replacements.

Future offers use the age schedule in section 2. Pick the proposed species once per milestone, favoring species not present while permitting duplicates at the cap; no rarity label. Present the offer quietly inside the offer panel and bird settings, without a top-bar badge, popup, email, or `birds adopted` count. Availability accumulates harmlessly during absence. Show one pending bird at a time; after acceptance, reveal any next eligible offer no sooner than the next day to avoid an adoption rush. All eligible milestones remain available, with no countdown or demand to visit.

Enforce the ceiling in the database transaction holding the aviary lock, not only with disabled UI. Renaming is in bird settings, never a scene tooltip. Preserve user names in settings and use escaped normalized names in naturalist observations; ordinary product prose is lowercase. Renaming never changes the stable call seed, traits, mood, history, or adoption age.

## 10. Scene rendering and first-frame continuity

### Scene composition

Use a single logical horizontal scene with soft sky/foliage, back/middle/front perch zones, bird rigs, and a bounded foreground ornament layer. No panning, scene scroll, scene zoom, cropping of birds, drag placement, hover labels, mood icons, or badges. Browser zoom remains available; it changes responsive layout rather than being blocked.

Canvas2D owns continuous scene drawing. Keep top-bar controls, panel/dialog surfaces, focus targets, captions, and narration in semantic DOM. Bird rigs use compact SVG paths parsed/prepared once, with reusable joints/poses and no per-frame asset decoding. Keep all seven birds to a bounded drawing complexity; do not introduce a 3D engine or full game framework.

Define fixed logical slots per perch zone and solve their screen positions on resize using zone-preserving constraints, bird bounding boxes, and 44-pixel minimum hit targets where possible. A narrow 320-pixel phone still fits all birds across three vertical depth rows; widen separation on desktop rather than scale birds into invisibility. Clamp movement curves to safe inset bounds. Do not let collision resolution change semantic perch zone. Reserve the top bar and safe areas using the dynamic viewport height; only notebook/settings panels scroll internally.

Use a calm palette of blues, greens, browns, and muted ochres. Day/night color is a continuous function of server time in the saved timezone; no geolocation or actual sunrise API is required. Choose smooth local-hour bands, including gradual morning/evening transitions. At night retain readable focus/captions and occasional nightjar-like activity. Weather overlays stay subtle, with no loud event.

### First bird under 500 ms

Authenticated HTML contains a private, compact snapshot and server-rendered SVG bird poses computed at their current action phases using the shared pose resolver. Critical styles and the tiny bootstrap run immediately; no webfont, settings bundle, audio permission, or noncritical asset blocks the first bird. The bootstrap starts live phase advancement and hands over to Canvas2D with matching rig geometry and time origin in one frame, without fade, pop, or entry animation. Do not publish a static default pose and animate it awake.

Only public rigs/code are CDN-cached. Personalized HTML uses `private, no-store`; the edge performs authorized dynamic assembly, not shared snapshot caching. If auth/state takes a beat, render a quiet sky field with one faint ambient cue, no spinner. A real initial account/adoption transition can briefly be empty and fly in; ordinary return cannot.

For fresh navigation, combine snapshot delivery and view-open greeting admission in the initial authenticated request when possible. For existing-tab return, request both state and greeting in one round trip. The tiny first-frame path must run before code-split system surfaces. Keep a startup mark for first visible bird, separate from hydration, interactivity, and audible calls.

### Motion implementation

Evaluate server-timed curves/pose segments against a smoothed server clock. Blend transitions through pose joints and body weight shift; derive low-amplitude idle variation from stable per-segment seeds and absolute phase. Avoid independent perpetual loops with identical cadence. Limit parallax to slow background offset, not pointer-following motion that requires constant user movement.

Leaves/feathers are client-only ornaments with a fixed pool, low spawn frequency, and recycled objects. Their initial phase can already be mid-drift. The server owns bird and weather events; an ornament never changes a bird's mood or the notebook.

The four-icon top bar fades nearly transparent after four seconds of pointer stillness. Restore on pointer/key activity, keyboard focus, an open panel, or touch interaction. Keep the focused/open control fully readable and preserve focus indicators. On touch, the first tap on the bar reveals it without accidentally activating an obscured action; a deliberate second activation opens the selected surface. No fifth icon or in-scene action widget appears.

### Reduced motion as a renderer

Use the same snapshot and semantic outcomes. Cross-fade between designed still poses over slow intervals, with roughly 6–12 seconds between idle pose changes. Replace flight paths with a gentle 1.5–3-second cross-fade at departure/arrival perches; no spatial movement. Remove leaf/feather drift and parallax; retain slow lighting/weather color changes, with settle lighting slowed to roughly eight seconds. Calls, caption content, drift, mood, and notebook remain intact.

Respect `prefers-reduced-motion` on first render. Offer system/default, on, and off overrides in matter-of-fact accessibility settings. Switching at runtime resolves the current semantic pose without restarting a bird, creating a new greeting, or altering simulation. Never send a reduced-motion user a static, inert picture.

## 11. Procedural call grammar, audio, and captions

### Stable call identity with per-call variation

Each of six species supplies a compact grammar: motif rules, syllable shapes, harmonic/noise balance, characteristic intervals, contour families, and admissible pauses. At adoption assign a bird's stable within-species signature (pitch neighborhood, timbre proportions, rhythm accent) and keep it forever. Mood shifts envelope/spacing/tension within the identity envelope; personality changes scheduling density and response tendency. Do not expand drift until Pip sounds like another species.

The server schedules independent calls plus response edges: one call can invite another within a plausible latency window, and two or more overlapping calls form a chorus. Publish stable call IDs and seeds across devices. Extend the rolling horizon without regenerating previously published calls. Limit intense overlaps to roughly four simultaneous birds, with remaining birds able to answer just after; seven birds need recognizable space, not a wall of sound.

In the browser expand a call seed through the pinned grammar into a resolved syllable graph. Each call varies continuous pitch contours, duration, breath/noise, and inter-syllable spacing within the signature, with a previous-shape avoidance rule so repeated calls are not identical. The graph drives synthesis, beak pose timing, and captions. It is not a collection of fixed audio variants or looped files.

### Audio graph and runtime

Use one lazily initialized AudioContext per page, a bounded pool of procedural voices, a gain bus per bird, an offer bus, and an output gain/soft limiter. Compose calls from oscillator contours, filtered generated noise, and envelope shaping. Reuse prepared noise buffers and motif tables. Start with at most 32 simultaneous synthesis voices, freeing/disconnecting completed nodes and retaining no per-call closures indefinitely. An optional AudioWorklet can be added only if profiling justifies it; it must preserve the same graph/caption contract and fit the bundle.

Schedule about 100–150 ms ahead against AudioContext time and the smoothed server clock. Skip calls already ended; start a late but still audible call at the correct syllable/phase only if it avoids a transient. Never replay a backlog after wake, a focus change, or permission grant. Use gentle attack/release and gain ramps to avoid clicks. A normalized chorus mix limits clipping and avoids a focused bird forcing others silent.

The small offered song fragments use the same procedural path through the offer bus. There are no audio downloads for calls, no recorded fallback, and no repeated chorus file. Test call variation acoustically and by ear, including mono playback, phone speakers, headphones, and seven-bird scenes.

### Browser audio permissions and graceful silence

Audible first-frame calls are contingent on browser permissions; never promise to bypass autoplay restrictions. If an existing permission permits it, schedule the currently active call immediately. Otherwise draw the actual calling bird and show captions by default until the owner deliberately enables audio through a control/gesture. Keep the invitation/return free of an audio-permission welcome modal. The system audio setting may say `Enable audio` plainly.

If WebAudio is absent, fails to initialize, is denied, or loses hardware output, keep the complete scene in graceful silence and default captions on. Retry once on a user-requested enable action rather than looping errors. Suspend the context on hidden tabs and close/release it on navigation; resuming uses fresh descriptors. Persist an explicit mute preference without penalizing drift. Never attempt recorded audio as a compatibility path.

### Caption generation

Generate captions from the resolved procedural graph actually scheduled: syllable count, relative contour, trill/spacing, intensity, and perch context. Examples include `a soft three-note rise` and `a low trill, paused, low trill again`. No fixed string per motif; do not describe a three-note rise if the resolved call contains two descending notes.

Show the caption near the calling bird with a high-contrast soft backing, bounded two-line width, and slow fade. Use a collision-aware layout that keeps text in view and off bird focus targets; do not silently drop captions during choruses. The grammar still resolves when muted or WebAudio is unavailable, so captions describe the intended current call rather than an unrelated fallback. Call IDs prevent duplicate captions on refresh. Reduced-motion uses opacity changes without movement.

Do not place every caption into an assertive live region. Screen-reader narration summarizes activity on its slower cadence; a user can request the current focused bird's call description without hearing seven captions stacked over the narration queue.

## 12. Naturalist observations and field notebook

Use a small, authored observation grammar and factual selector, not an external language model or raw event dump. This preserves voice and keeps private history out of third-party text-generation systems. Facts come from the owner's saved bird observations, current plans, and weather: greeting order, persistent perch tendencies, a rare response pairing, a long quiet preen, or a short rain's effect. No owner attendance counts, absence length, drift numbers, score, or generic `session started` entries are candidates.

Candidates receive private significance scores based on novelty within that aviary, then pass sparsity rules: a routine entry no more often than every 72 hours, a noteworthy exception no more than once per 24 hours, and at most two exceptional entries in a rolling week. Tune generated-fixture output to roughly one entry every few days for regular visits. A very active fixture should not produce one entry per session; a long outage should not backfill a feed.

Validate facts before selecting wording. `pip greets before wren today, first time this week` requires actual private greeting-order observations, not speculation from warmth. The relevant counts describe birds, never the owner's frequency. Use lowercase, present tense, names/species/perches, and specific details. Preserve an observation's stored historical wording after a rename rather than rewriting the past; new entries use the new name.

Notebook is read-only and host-only behind its top-bar icon. Fetch cursor pages, virtualize/recycle rows with accessible reading order, and release rows/listeners after scroll-out. Keep all past entries available indefinitely; do not archive, truncate, add comments, or offer individual delete/edit. Account export/deletion remain system-level privacy operations. No unread counter, new-entry badge, celebratory milestone, or notebook notification exists.

Separate authoring rules for product and system copy. Naturalist grammar is shared with captions/narration but does not style auth or errors as bird metaphors. Add a copy review gate that searches all displayed strings for prohibited welcome/gamification/absence phrasing and manually checks context; a string search alone cannot certify the tone.

## 13. Accessibility and alternate output surfaces

### Screen-reader experience

Expose a scene landmark and one roving-focus semantic target per bird, positioned to match the rendered bird without adding visible labels/buttons. Accessible names identify the bird and describe its perch/action naturally; they never disclose normalized traits, enumerated mood labels as a status list, or perch numbers. Mark ornamental canvas/SVG internals as hidden from assistive technology.

Generate coherent running prose from the same snapshot/action graph as rendering, using the shared naturalist grammar. Update one polite, atomic narration region every 45 seconds at idle, within the requested 30–60-second range. Coalesce superseded descriptions so a screen reader never hears a minute of stale queued prose. Give a return-greeting, accepted offer reaction, or settle a prompt observation at the next practical opportunity, normally within two seconds, while retaining polite queue behavior.

An example is `pip is on the front rail, preening slowly. wren watches the back branch. the light is soft this morning.` A greeting observation can describe a head turn; it must not announce `welcome back`. Suppress unchanged prose, allow narration pause/resume and on-demand `describe aviary` in accessibility settings, and ensure panel reading is not continually interrupted. Essential auth/error text uses normal system language and appropriate status semantics separately.

### Keyboard and focus

Tab proceeds through the four top-bar controls and then the scene's first bird. Arrow keys move among birds in spatial order with stable UUID-based ties; only one scene bird is in the Tab sequence. Enter engages listen-in, Escape disengages; leaving the scene ends it. Scope `o` to scene focus to open offers. The offer panel, settle, notebook, account, adoption, and settings are fully operable with standard keyboard controls and restore focus to their invoker on close.

Keep a high-contrast, nonflashing focus outline against daylight, rain, and full-night colors, with a paired light/dark edge if necessary. Maintain semantic focus on UUID during snapshot updates or renames. If a dialog closes or the view loses authorization, restore focus to an appropriate system control; never strand it on a removed canvas overlay. No pointer-only hover behavior exists.

In visitor mode, expose descriptive bird navigation but not a `listen in` button or any host action. A visitor can change their own captions/reduced-motion/audio-output preference without submitting owner events. Test that assistive-technology focus alone cannot create host presence or a greeting.

### Contrast, sizing, and reduced motion

Require at least 4.5:1 for normal user-copy text, 3:1 for large text, and 3:1 for focus/component boundaries. Establish actual token ratios for every palette, not just nominal AA compliance on a white design board. Captions/narration use controlled backings so dynamic plumage/lighting cannot invalidate contrast. Keep bar text readable whenever focused; fading an unfocused icon does not fade an active label or keyboard ring.

Validate 200% text zoom, high zoom with reflowing panels, high-contrast/forced-color mode, touch target sizes, and no scene cropping of any bird on narrow layouts. The one-screen constraint applies to the aviary; notebook/account dialogs can scroll for access. Use system typography to avoid font downloads and preserve text rendering.

Reduced-motion mode is implemented by the pose renderer in section 10, including the very first frame. Captions, calls, and slow color changes remain. Test this as a designed product, not simply as a check that `animation: none` was applied.

Run automated semantic/contrast checks plus manual NVDA/Firefox and VoiceOver/Safari sessions across sign-in, starter names, return, listen-in, offer, settle/undo, notebook, visit revocation, settings, export, and deletion. Use real audio/caption and reduced-motion configurations in the same end-to-end scripts. Accessibility findings affecting core interactions block release.

## 14. Invitations and quiet visits

Host settings provide explicit email entry, current invitations, revoke, and the private visit log. Do not prompt sharing during onboarding, show a visitor list in the scene, add badges, or use a public link. Generate a high-entropy one-time link, save only its digest, and mail it only to the named recipient through the identity boundary. Treat the link as a bearer grant for that named invitation; do not claim it prevents the recipient voluntarily forwarding it. Stronger identity proof would be a later policy change, not an implicit profile feature.

Redemption atomically consumes the invitation and creates one read-only session. Show the host's current state through the same renderer, with host-local day/night, current weather, and existing owner-triggered plans. No special beautification, visiting greeting, shared cursor, or host presence marker appears. The visitor cannot listen in, offer, settle, adopt, rename, read notebook, or generate presence; check all of this at the API as well as in the UI.

Visit heartbeats update only start/end/approximate-duration information. Pull state every 15 seconds while visible and immediately on visible return. Authorize each request before serving cached/conditional data. Revocation therefore removes access on the next pull (within about 15 seconds for an active visible visitor), returns `visit_no_longer_available`, stops audio, clears host state from the DOM and memory, and displays the same matter-of-fact surface as expiry. Hidden visitors authorize before rendering again.

Unused invitations expire at 30 days, with no revival or automatic resend. Active sessions have the separate finite lifetime decided above. The host log lists historical visits newest first, recipient email, date, approximate duration, and outstanding invitations. Revocation removes the active/outstanding row; retain historical completed visits for transparency rather than pretending the visit did not happen. This interprets the PRD's reference to absence in the log as absence from its active list.

If the host has explicitly enabled optional visit mail, enqueue one notification for redemption, recheck the toggle/host status before delivery, and cap delivery to one email per hour with no deferred digest or retry reminders that create a stream. Default behavior is silent logging. No analytics compute most-visited aviaries or visitor rankings. Account/recipient deletion removes the related invitation and visit records rather than leaving recoverable email in log snapshots.

## 15. Account export, deletion, and privacy controls

### Export

On request, generate JSON from one consistent database revision after advancing any due tick. Include birds with stable IDs/names/species, exact current personality vectors under the narrow portability exception, current moods, immutable notebook entries, and account settings. Include schema/content version and export timestamp; exclude raw presence events, device secrets, magic-link tokens, invitation tokens, and owner attendance history. Do not build a visit-frequency export disguised as account portability.

Stream large notebooks from the consistent snapshot rather than buffering them all. Store the encrypted artifact privately for at most 24 hours and email a clean short-lived download link to the verified owner address. Require owner authorization or one-time download-token exchange; no public object URL, shared CDN caching, URL/body logging, or export attached directly to email. Delete artifacts and outstanding download grants on account deletion/purge. Export/download is a system surface with matter-of-fact copy and no trait-preview UI.

### Thirty-day deletion lifecycle

At deletion request, mark the account immediately, pause its ticks, revoke invitations/visitor sessions, invalidate export downloads, and stop accepting owner simulation events. Retain the saved aviary unchanged during the recovery window. Sign-in during the next 30 days shows a system restoration surface with `I changed my mind`, not a normal aviary or guilt message. Recovery clears deletion status, restores access to the same bird IDs/vectors, and advances ordinary mood/time from saved state without presence for the deleted interval; do not revive old invitations automatically.

At day 30 a fenced purge removes birds, vectors, events, coverage, notebook, observation summaries, settings, sessions, links, invites/visits, outbox/jobs, exports, account-linked diagnostic records, and the identity record. Traverse both host and recipient references. Retries are idempotent; a tombstone containing only the minimum purge job identifier/time prevents stale queues or restores recreating the account, and is itself bounded/deleted after restore-protection expires.

Address backups explicitly: segregate sensitive records into account-key-encrypted payloads where necessary so day-30 key destruction renders retained backup copies unreadable, then expire physical backup copies under a documented short retention. Maintain purge manifests used during restoration before any traffic is accepted. Immutable backups cannot be advertised as immediate physical erasure; the release privacy review must verify cryptographic erasure and restore-time suppression. Do not promise hard deletion if vectors or notebooks remain recoverable in backup storage.

### Telemetry boundary

Collect only allowlisted request counts, latency/error distributions, first-bird timings, frame-time distributions, memory/performance samples, audio-context failures, tick duration/lag, and anonymous session-duration histograms. Remove account/bird/session IDs, names, event types, mood, traits, notebook content, call seeds, email, and full URLs from analytics events. Bucket durations client-side and send a histogram update without an enduring browser identifier.

Operational request IDs are random and short-lived. Restricted application diagnostics may reference synthetic account UUIDs only when necessary to investigate an account error, never interaction contents; retain them briefly with a purge index so deletion removes all account-linked records. Aggregate metrics have no per-account dimension and cannot be reversed into an aviary history. Keep identity UUIDs out of RUM entirely.

Use an allowlist collector, no session replay, no DOM capture, no automatic request-body logging, no third-party analytics that receives state, and no ML/training export. Telemetry credentials have no simulation/identity database read access. Audit message schemas and intentional failure paths for accidental payload inclusion. A plain-language settings privacy link names collected operational categories and explicitly excludes behavioral use. Generated synthetic accounts may be instrumented fully for tests; real-owner vectors never enter an aggregate calibration pipeline.

## 16. Performance budgets and operational verification

| Area | Launch budget and verification |
|---|---|
| Initial JavaScript | Hard ceiling below 2 MB gzipped across all modules loaded before first paint; engineering target below 250 KB, with a much smaller first-frame bootstrap. Count dynamically imported startup chunks too. CI artifact-size gate. |
| Critical first-frame resources | Target HTML/private snapshot/rig payload below 40 KB gzipped and no blocking custom font or media request. Public assets immutable and edge-delivered. |
| First visible bird | Below 500 ms from aviary navigation on the designated mid-tier phone/4G profile; separately measure first paint, hydration, greeting, and first audible call. Gate repeated cold and warm runs, report p50/p95 and cold-navigation misses without redefining the clock. |
| State payload | Typical few KB; seven-bird snapshot below 12 KB gzipped, bounded 120-second timeline. No event log or notebook attached to every pull. |
| Idle rendering | Sustain 60 fps on a representative five-year-old mid-range laptop for 30 minutes at two and seven birds. Target scene CPU below 6 ms p95 and total main-thread frame work below 8 ms p95 within the 16.7 ms frame budget. |
| Audio | Bounded voice/context count, no backlog after wake, no audible clipping/clicks. Measure scheduling underruns and context failures without call identity telemetry. |
| Memory | No sustained client growth over a 30-minute soak after warmup. Reused pools, bounded timelines/queues, released audio nodes, and virtualized notebook pages; inspect retained objects and heap slope. |
| Server | Event admission below 200 ms p95 in launch region; normal tick below 20 ms p95 target, p99 compute alarm above 5 s, independent due-lag alarm. Event backlog and scheduler-fence failures alarm. |
| Hidden tabs | No requestAnimationFrame work, new audio scheduling, presence timers, or periodic snapshot pulls. Visit/owner lease expiration occurs on the server. |

Define the synthetic mobile profile before coding: a physical mid-tier phone plus controlled 4G-like downlink/round-trip latency, with a reproducible browser-throttled equivalent (initially approximately 10 Mbps down, 1 Mbps up, 80 ms RTT, mobile CPU throttling). Also run slower-network diagnostics; do not quietly replace the PRD's 500 ms requirement with a desktop measurement. Establish a representative 2021-class integrated-GPU, 8 GB laptop for the current five-year baseline. Supported browsers are the last two major releases of Chrome, Safari, Firefox, and Edge; derive that test matrix during implementation rather than pinning stale versions in this plan.

The first-frame performance spike precedes architectural lock-in. Validate edge-authenticated HTML, phase-correct bird rigs, and initial JavaScript on physical hardware. If it misses 500 ms, reduce critical bytes, origin round trips, and bootstrap work before adding systems panels; do not resolve the miss with a canned bird loading animation. The fallback remains a quiet field on genuinely slow delivery, but that does not turn a budget failure into a pass.

Memory CI runs seeded 30-minute scenes with repeated offers, listen switches, context suspend/resume, notebook scrolling, and panel open/close. After equivalent forced-GC samples in the automation harness, require no statistically significant upward retained-heap slope and only a small bounded noise envelope (initially 5% or 2 MB, whichever is larger). Verify object counts/pool bounds separately so a generous noise allowance cannot hide leaks. Use browser-native memory tooling where available and retained-node/listener counters elsewhere.

Run geographic synthetic checks using synthetic aviaries, not replayed actual-user state. RUM reports aggregate first-bird/frame/audio error distributions with low-cardinality browser/device/network buckets. Monitor database contention, per-shard due backlog, outbox failures, invitation authorization errors, and privacy-collector rejections. No engagement funnels, daily active-user streaks, trait rankings, or per-bird attention dashboards are release metrics.

## 17. Verification strategy and release acceptance

Build a deterministic simulation fixture clock and seed generator before calibrating the engine. Use synthetic-only test accounts with saved checkpoints. Test invariants and end-to-end outcomes; avoid tests that merely restate implementation lines.

| Test family | Required cases and expected result |
|---|---|
| Identity/persistence | Rename, migration, backup restore, deletion recovery, seven-bird addition, two-device refresh: stable UUIDs/vectors/signatures remain exactly continuous. |
| Presence truth table | All eight visible/focused/recent-activity combinations; only the all-true case credits time. Boundary at 240 seconds, blur midinterval, no initial input, hidden tab, mobile tap without move, and trusted versus synthetic input. |
| Multi-device attention | Overlapping intervals union once, replayed pings add zero, different device sequence orders remain safe, visitor watch time adds zero. Qualified listen-in masks have no stale-duration tail. |
| Drift properties | Every trait finite, bounded, and nondecreasing; no same-session visible jump; presence dominates interaction-only fixtures; day-seven instrument signal and day-21 perceptual change; long absence leaves vectors intact and filter tail bounded. |
| Event/tick correctness | Crash before/after commit, duplicated/out-of-order transport, lost acknowledgment, two workers, admission concurrent with tick, unknown event schema, catch-up across DST: exactly-once effects in admitted log order. |
| Mood continuity | No tab-open/midnight neutral reset; daily-ish relaxation, dusk/night species activity, short weather effects, bounded alarm contagion, absence not used as a wary penalty. |
| Presentation sync | Offer receipt and next snapshot share plan ID/outcome; late snapshots never rewind; same saved day/weather across two timezones; suspend/resume skips expired calls; no client personality fields accepted. |
| Calls/chorus | Calls differ by generated contours, preserve identity across moods, render chorus without clipping or loop artifacts, caption graph matches synthesis, seven signatures distinguishable in controlled listening tests. |
| Interaction UX | Focus/click/Enter/Escape semantics, smooth mix with audible ambient floor, cooldown race, pool/seed limits, settle five-second undo, tab-close equivalence, no welcome/absence/achievement surfaces. |
| Auth/lifecycle | Fifteen-minute single-use links; concurrent replay yields one session; device revoke blocks next write; email switch only after verification; export consistent and expires; day-30 purge covers storage/logs/backup recovery. |
| Visit capabilities | All owner mutations rejected for visitor credentials; no greeting/presence/listen events; 30-day unused expiry; concurrent redemption once; revocation checked before 304 and before resumed rendering. |
| Notebook | Sparse specific observations, facts justified, no attendance copy or trait numbers; unbounded history accessible by cursor; no annotation APIs or retained virtual rows. |
| Accessibility/performance | Full manual scripts and automated checks from sections 13/16 in supported browsers; reduced-motion first frame, readable night captions, no hidden-tab work, physical-device first-bird budget, 30-minute frame/memory soak. |
| Privacy | Serializer cannot leak traits into normal snapshots; telemetry rejects behavioral fields and all identifiers; account-linked logs purge; email remains solely in identity storage; no token/referrer leakage. |

Use fake timers/unit properties for engine arithmetic and qualification, real database transaction integration for tick/admission/idempotency, browser end-to-end tests for scene/audio/accessibility, and controlled worker/network fault injection for recovery. Listen by ear and review prose manually; numerical assertions cannot certify aliveness, calmness, recognizability, or voice.

Release requires all must-have accessibility surfaces and invariants, not an inaccessible beta presented as v1. Maintain synthetic staff aviaries of multiple ages (including a year-old/six-bird and 18-month/seven-bird fixture) so age pacing and the maximum can be tested without changing actual owners' clocks or introducing a score-based shortcut.

## 18. Work packages, sequencing, and rollout

Use six work packages with explicit interfaces and exit criteria. Engineers can work concurrently inside the product team after contracts are agreed; this plan itself does not launch agents or implement those packages. Estimate approximately 12–15 weeks for a small team with backend/simulation, client/audio, and design/accessibility ownership; calibrate the schedule after the first hardware spike. The multiweek drift study runs alongside implementation using accelerated synthetic clocks and realistic nonaccelerated review clips.

### Package A — contracts and aliveness prototypes (weeks 1–2)

Deliver versioned snapshot/event contracts, state ownership table, first-frame pose resolver, two species rigs and procedural signatures, exact presence qualification, provisional design tokens, reduced-motion still poses, and sample narration/caption grammar. Benchmark the private edge first-frame path on mobile and 60 fps idle on baseline laptop. Exercise an audible chorus and silent caption fallback. Exit only when the architecture can plausibly meet first-bird timing and both normal/reduced-motion scenes feel already alive.

### Package B — durable identity, auth, and canonical simulation (weeks 2–5)

Deliver PostgreSQL schema/migrations/roles, encrypted identity boundary, 15-minute links and per-device sessions, two-bird bootstrap, append-only event admission, fenced minute scheduler, durable vectors/filters, presence unions, mood/timezone/weather, deterministic presentation plans, and crash-safe commits. Run the week/three-week drift fixture matrix and tune positive-only coefficients. Exit with concurrent-device/crash tests proving preserved vectors and zero visitor/hidden-tab credit.

### Package C — complete client and audio surface (weeks 3–7)

Deliver phase-correct Canvas handover, responsive zones, all six species, idle/flight/weather, top-bar fade, listen-in mixing, offers and song library, settle/undo, mute/permission handling, and bounded WebAudio allocation. Ship semantic bird targets, cross-fade reduced motion, captions, and narration in the same components. Exit with keyboard/manual screen-reader scripts and the seven-bird 30-minute frame/memory budget passing.

### Package D — notebook, age adoption, and system lifecycle (weeks 5–8)

Deliver sparse observation selection, read-only paginated notebook, renames, age-only future offers with atomic cap, settings revisions/timezone, session revocation, verified email change, JSON export, deletion/recovery/purge including backup strategy, and the privacy text. Exit with historical notebook, vector continuity, export correctness, and complete purge/restore drills; no attendance surfaces may appear in copy.

### Package E — optional visits and integrated hardening (weeks 7–10)

Deliver named one-time invitations, read-only capability, silent duration log, revocation/expiry, local visitor accessibility controls, and the explicit off-by-default visit-mail setting. Test authorization before conditional responses, no host mutations through any visit route, and no co-presence/presence credit. Run link replay, suspended-tab, hardware-audio, outage, and telemetry leakage tests. Exit with the full browser matrix and required accessibility flows intact.

### Package F — staged release and observation (weeks 10–15)

Deploy backward-compatible contracts and engine versions first, with synthetic alarms/performance checks active before real traffic. Begin with internal synthetic and explicitly invited owner testing, then a small private beta and progressively larger cohorts. Use operational failure/latency/performance gates and qualitative review of generated fixtures to advance, not visit frequency or owner drift telemetry. Cohort routing uses synthetic UUID hash without sending account IDs into analytics.

All launch accounts start with two birds. Implement the seven-bird storage/scene/audio ceiling before broad launch, verify 3/5/7-bird synthetic fixtures from day one, and release age-gated adoption in increasing enabled maxima of three, five, then seven only as each scale passes recognizability and resource gates. Complete that rollout before real accounts reach their respective age milestones. If a maximum gate is delayed, preserve accrued eligibility and explain availability plainly only in the adoption panel; never remove or reset an already adopted bird. A rollout flag cannot downgrade existing bird counts.

Ramp cohorts (for example internal, 1%, 5%, 25%, all) only after a stable operational interval with no vector-loss, presence inflation, authorization, accessibility, first-frame, or memory regressions. At every stage include silent/reduced-motion/screen-reader use and mobile devices. Do not postpone an accessible renderer to v1.1.

Maintain separate kill switches for new invitations, future adoption acceptance, offer admission, and optional mail, plus engine-version pinning. In a severe simulation incident freeze new writes on affected aviaries and serve the last safe state with clear system copy; restore from exact saved vectors/checkpoints. A rollback never seeds new birds, recomputes vectors from history, discards existing seven-bird scenes, or falls back to recorded calls. Use additive migrations and compatible presentation contracts so frontend rollback still reads newer saved identity/state.

## 19. Risks and operational responses

| Risk | Detection and prevention | Response |
|---|---|---|
| Drift too fast, too slow, or eventually saturated | Synthetic seeded 7/21/90/365-day trajectories, attention-only versus clicks-only, perceptual blind comparisons, conservative initial seeds/coefficients | Tune future nonnegative deltas and perceptual mapping under an engine version; do not decrement or reseed existing traits. |
| Presence silently inflated | Truth-table, hidden-tab, suspension, overlapping-device, stale-lease tests; coverage union invariant | Stop offending event admission, fix client qualification, preserve saved vectors; never compensate with negative drift. |
| Tick lost updates or double effects | DB single-writer role, row fences, atomic watermark/delta, retry/failover chaos, restoration drill | Isolate affected aviary, recover exact checkpoint, verify IDs/vectors before resuming; do not guess from client snapshots. |
| Minute tick feels unresponsive | Accepted server gesture plans returned immediately and shared in projections | Reduce admission latency and descriptor cost; do not move semantic simulation into the browser. |
| Audio feels canned or uncanny | Species-specific grammar review, per-call contour variation, stable-signature listening trials, bounded chorus overlap | Adjust envelope/grammar within signature compatibility; keep procedural synthesis or silence, never loops. |
| Seven birds erase recognizability or exceed budgets | Synthetic maximum-count listening, caption collision, physical-hardware 30-minute soak | Delay new high-count adoption availability before milestones; improve spacing/mix/pools while retaining existing birds and cap. |
| Autoplay fails the audible first-frame expectation | Permission/device/browser matrix, explicit separate audible metric | Start the same living visual/caption scene, enable audio on deliberate gesture, explain system status without a greeting modal. |
| Accessibility becomes a semantic state dump | Naturalist prose review, real screen-reader sessions, slow queue coalescing, reduced-motion aesthetic review | Treat output quality as a release-blocking defect and fix the shared presentation grammar/renderer. |
| First bird misses 500 ms | Critical-byte CI gate, edge TTFB and physical mobile synthetic runs, startup trace | Reduce auth/state round trips and critical runtime/assets; retain quiet-field fallback without declaring the requirement waived. |
| Long-running render/audio leaks | Retained-object and heap-slope soak, bounded pools/horizons, context/listener lifecycle checks | Fix lifetime ownership; disable the leaking ornament/optional effect while preserving calls and accessible output. |
| State/PII enters observability | Collector allowlist, roles with no DB read, redaction tests, token-query suppression, privacy audit | Block the emitter, delete contaminated operational records, fix schema before re-enabling; do not normalize broader collection. |
| Revoked visitor still sees host data | Authorization before 304/cache, next-pull and resume tests, no public state cache | Terminate visit output and invalidate private grants; stop invitation issuance if the boundary is broken. |
| Export or backups contradict privacy | Explicit export exception, single-revision export, per-account keys and purge manifests, deletion drills | Correct lifecycle/storage before public launch; do not conceal recoverable copies behind a soft-delete flag. |
| A convenience UI adds gamification or announcements | Scope review, copy tests, explicit no-counters/no-badges contracts, manual return review | Remove the surface and its underlying statistic rather than making it an optional toggle. |

## 20. Engineer handoff checklist

Before implementation, publish the event/snapshot schemas, database ownership/retention matrix, shared phase resolver, six-species art/audio grammar contract, authored naturalist observation rules, system-copy rules, design tokens with contrast ratios, and the synthetic calibration fixture catalog. Establish exact hardware/network test profiles and separate metric schemas for public operational telemetry and private test instrumentation.

Before release, demonstrate: an owner returning to the same birds after a synthetic two-week absence; laptop/phone seeing the same mood and weather without vector merges; first bird under 500 ms with motion in progress; a procedurally varied greeting within two seconds; presence-only positive drift on the expected weeks scale; offers/settle with immediate gestures and ordered server effects; recognizable seven-bird calls without memory growth; an equally alive reduced-motion/silent/screen-reader surface; sparse notebook prose; a visitor unable to change anything; revocation on the next pull; and recoverable-then-hard account deletion with no PII in aggregate telemetry.

These are the completion criteria for v1. Feature count, visit frequency, and visible progress are not success criteria for this product.
