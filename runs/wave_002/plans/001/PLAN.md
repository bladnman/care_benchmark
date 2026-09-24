# Pocket Aviary v1 — implementation plan

## 1. Product boundary and invariants

Build a browser-only aviary for one user and one canonical aviary per account. A new account receives two system-selected starter birds, with user-assigned and renameable names. Birds may be added as the aviary ages, to a hard maximum of seven. V1 includes the animated horizontal scene, procedural calls, return-greeting, presence accounting, listen-in, offers, settle, a sparse read-only field notebook, magic-link accounts, account export/deletion, multi-device sync, optional per-invite visits, narration, reduced-motion rendering, captions, and operational performance telemetry.

Treat these as design and data invariants: the server is the only writer of personality and mood; clients render snapshots and submit events; bird IDs survive rename and migrations; personality numbers never reach the client or user-facing surfaces; drift is one-way toward expressiveness and neglect never lowers traits or produces distress; visitors are read-only and never contribute presence or drift; calls are generated procedurally, never downloaded loops; the aviary has no scrolling/panning or embedded scene controls. Keep out native clients, payments, shared or multiple aviaries, scene customization, discovery, social profiles, chat/comments, public ranking, gamification, visit-frequency displays, push notifications, and any caretaker, hunger, death, or distress mechanics.

Product copy in the aviary, notebook, narration, captions, and offer prompts uses concise, specific, lowercase naturalist prose. Sign-in, account, settings, accessibility settings, and errors use direct matter-of-fact system copy. No welcome toast, badge, streak, arrival banner, or visit badge is permitted.

## 2. Architecture and service boundaries

Use a web client, an authenticated API, a relational canonical-state store, and a background simulation worker. Keep these as logical boundaries even if the first deployment packages the API and worker together. The database is authoritative. A durable queue or database-backed job table schedules ticks and retries; an account-scoped lease/lock and revision check prevent two workers from advancing one aviary concurrently. Commit each state revision and its consumed-event cursor atomically.

The browser obtains a compact, account-private initial snapshot and renders immediately. Static shell/assets may use a CDN; personalized snapshots must be private and keyed by account/session, never shared-cacheable across accounts. The client keeps only transient rendering/audio state and the latest snapshot revision. It never merges two aviary states. On the server, separate modules own identity/auth, aviary state and snapshots, event intake, simulation, notebook generation, and visit authorization. The analytics/RUM pipeline accepts aggregate operational measurements only and has no read path to simulation tables.

Start with a small architecture decision record for the specific database, queue, session-token format, and deployment platform. Select boring managed components with transactions, encrypted storage, backups, regional edge delivery, and worker scheduling; avoid adding a streaming platform unless measured scale requires it. Establish schema migration and rollback procedures before account data is admitted.

## 3. Persistent data model

Use synthetic UUIDs for account identifiers in every table, message, partition, log, and metric. Encrypt email on the account record; use it only for sign-in, verified account changes, and required mail delivery. Never use an email-derived key. Suggested records:

- `Account`: UUID, encrypted verified email, email verification state, IANA timezone, created/deletion timestamps, settings (captioning, reduced motion override, visit-notification opt-in).
- `Session`: opaque token digest, account UUID, device label, created/last-used/expiry/revocation timestamps. Tokens are per-device and revocable.
- `MagicLink`: token digest, encrypted destination or account reference, creation/expiry/consumption timestamps, and rate-limit metadata. Expire in 15 minutes and consume once.
- `Aviary`: UUID, account UUID (unique), revision, server tick cursor/time, last presence end, current day-phase/weather/settled state, event sequence cursor, created timestamp. Initialize two birds and the initial scene transactionally.
- `Bird`: stable UUID, aviary UUID, species, display name, server-only five-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), mood enum and mood timer, perch/action/call state, adoption timestamp and age-based next-bird eligibility. Store bounded normalized traits and version the seed/range configuration.
- `InteractionEvent`: account/aviary UUID, monotonic account/aviary sequence, idempotency key, event type, bird UUID when applicable, server receipt time, minimal payload, processing cursor. Types include eligible presence intervals, listen-in start/end, offer, settle/undo, and re-engagement. Append-only until safe retention/compaction policy is implemented; it exists only to drive that account's simulation. Do not store pointer paths or raw key data.
- `NotebookEntry`: UUID, aviary UUID, observed time, structured observation kind and bird references, generated prose, generation/version metadata. Read-only to the user; preserve history indefinitely while account exists.
- `Invite`: UUID, host account UUID, encrypted invitee email, one-time token digest, issued/expiry (30 days unused)/revoked/first-used times. It conveys read-only snapshot access only.
- `VisitSession` / `VisitLog`: invite UUID, start/end times, coarse duration, visitor email reference for the host's transparency log. Do not connect visits to host presence or analytics identity.
- `DeletionJob` (or equivalent): account UUID, scheduled hard-delete time and progress, so the 30-day recovery window and final purge are reliable and auditable.

Export is a point-in-time JSON snapshot containing the specified account settings, birds including vectors/moods, and notebook. Generate on demand and email a short-lived download link to the verified address. Soft deletion marks the account unavailable and recoverable for 30 days; hard deletion removes account, bird, event, notebook, invite, session, export artifacts, and associated per-account telemetry identifiers. Define backup expiry/deletion behavior as part of the storage design so hard deletion has an honest boundary.

## 4. API and event flow

Keep routes versioned, authenticated, idempotent where they mutate, and scoped from the session's account UUID rather than accepting a client-selected account ID.

- `POST /v1/auth/magic-links` requests a link with per-email rate limits; `POST /v1/auth/sessions` consumes its one-time token and creates a device session. Provide matter-of-fact errors for expired/replayed links. Add session list/revoke, new-email verification, export request, and deletion/recovery endpoints under `/v1/account`.
- `GET /v1/aviary/snapshot` returns revision, server timestamp/tick age, local-time phase inputs, scene/weather state, birds' renderable state, active transitions and event cursor. Do not include personality values or private interaction history. Support revision/ETag responses. Pull on first open, visibility return, long frame gap, and low-frequency visible-tab keepalive.
- `POST /v1/aviary/events` accepts a batch of narrowly typed events with client idempotency keys and observed client time only as advisory context. Server receipt time and sequence order govern processing. Validate bird membership, cooldowns, offer types and session ownership. Return a durable receipt and canonical revision; for an offer or settle, schedule/apply promptly so its reaction is visible without waiting a full minute. The scheduled tick remains the only canonical simulation writer. A low-latency event processor can invoke the same serialized tick routine through the accepted event sequence.
- Presence heartbeats report only visibility, window-focus, recent pointer-or-key activity boolean/time, and a bounded interval; never raw movement or key values. The client sends them only while all three eligibility conditions hold. Server caps intervals, de-duplicates, and closes them on hidden/unfocused/idle/settle/disconnect timeout. Calibrate the “few minutes” activity window with privacy-preserving internal fixtures; never equate an open tab with presence.
- Listen-in submits start/end events for the focused bird, but mix levels remain a client audio concern. Offers are server-validated events; a cooldown of a few minutes is per bird. For settle undo, accept a reversal only inside the five-second window; any later click may re-engage normally.
- Visit management routes issue an invitation by named email, list pending/active/recent visits in account settings, and revoke it. Mail a one-time visitor link; enforce its 30-day unused expiry and revocation on every snapshot request. `GET /v1/visits/{token}/snapshot` returns the same canonical renderable scene as the host, with no account interaction endpoints or notebook controls. Enforce read-only on the server, log approximate visit duration, and exclude visitor activity from simulation. The visit log is on-demand and has no badge; any optional visit notification remains explicitly off by default.

Interaction acknowledgements should distinguish accepted, duplicate, expired, cooldown, and unavailable cases in system voice. Never let client absolute vector or mood fields enter an event payload.

## 5. Simulation and behavioral engine

Run a server tick at roughly one-minute cadence, independent of connected clients. Ticks must be deterministic for a given state, ordered event range, time input, and stored random seed. Use an account-scoped transaction/lease: load state and unprocessed events, compute the next revision, persist all bird/aviary changes and consumed sequence cursor together, then acknowledge the job. Retries must be idempotent; gaps, duplicate jobs, clock skew, and worker restarts cannot double-apply presence or offers. Catch-up after downtime advances elapsed mood/day-phase state with bounded steps, not one unbounded loop per missed minute.

Represent personality as five bounded normalized scalars stored only in `Bird`. Maintain a slow low-pass accumulator for eligible presence time and accepted interaction signals. Presence time is dominant. Listen-in contributes more to that bird's social warmth and vocal frequency; accepted offers nudge curiosity; approaching/receiving an offer can nudge boldness. Settlement ends presence and quiets mood but adds no directional drift. Apply only nonnegative deltas, saturate at configured caps, and make no negative/neglect deltas. Hide exact values from snapshot contracts, narration and notebook generation.

Make drift coefficients, presence activity window, bounds, seed distributions and adoption-age thresholds server-side versioned configuration. Use internal synthetic fixtures to tune the PRD calibration: measurable instrument change after about a week of regular eligible presence, but no visibly attributable single-session change and perceptible difference only after about three weeks. Freeze the selected configuration version on each update so later tuning does not rewrite history. Do not use cross-account behavioral aggregation to calibrate it.

Mood is a small enum (wary, content, curious, drowsy, alert, with implementation finalization documented) and persists across sessions. The tick transitions mood based on recent same-session events, local-time cycle, transient ambient weather, bird-to-bird calls/alarm effects, and personality weights. Persist transition start/end and mood timers so a reconnect does not reset to neutral. Rain briefly dampens calling; wind can alter alertness/wariness. Absence does not become distress: quiet/ambient behavior is a presentation of reduced recent presence, not a worsening mood or declining trait.

Store stable species motif identity and per-bird call seed. Runtime call grammar combines motif, timing, pitch and envelope variation shaped by species, mood and vocal frequency without losing each bird's recognizable signature. Produce a structured call event (motif parameters plus concise caption template parameters) so audio, captions and narration derive from the same generated call. Birds may respond to another bird's call and form a chorus. Keep the six-species pool coherent; species rarity is not a mechanic. Adoption flow selects the initial two species; later offers are based only on aviary age and capped at seven, never visit count or score.

Generate notebook observations only for noteworthy aviary events and sparse quiet observations, targeting about one entry every few days for a regularly visited aviary. Use structured templates and approved naturalist phrasing tied to actual bird/time/scene facts. Do not write session logs, vector deltas, visit-frequency summaries, or generic “achievement” copy. Avoid repeated observations with per-aviary deduplication/cooldowns.

## 6. Client rendering pipeline

Build a one-screen, responsive horizontal scene with front, middle, and back perch zones. Scene composition uses a compact scene graph: quiet foliage/sky background, perch plane, birds and their pose/action layers, occasional foreground branch, and separate sparse top-bar DOM. A Canvas 2D renderer is a suitable initial choice for low-cost compositing and stable frame pacing; keep controls, text, focus targets, and narration in semantic HTML. Validate rendering choice against the 60fps and browser support budgets before locking it. At narrow widths compress spacing without cropping a bird; at wide widths expand space, with no pan, zoom, or scroll.

The snapshot supplies an already-active bird state. Start with a non-static pose, current transition progress, call timing and idle activity; never add a wake-up or loading spinner. If state is late, draw the quiet field and subtle local ambient cues. Use an explicit one-time fly-in only from post-adoption empty aviary to first bird. The client interpolates between authoritative snapshots and derives inexpensive idle micro-motion from bird state, mood and stable local seeds. Keep leaves/feathers as client-only ornaments with no server event or personality effect. Birds choose perch and idle action; users cannot arrange them.

Use requestAnimationFrame while visible, stop scene rendering when hidden, and request a fresh snapshot when visible or after suspended frame gaps. Continue no client simulation during hidden periods: the server tick owns continuity. Keep top bar icons limited to account/settings, accessibility, notebook and offer. Fade the bar nearly transparent after a few seconds without pointer/keyboard activity and restore on activity; preserve keyboard discoverability and focus visibility.

Reduced-motion is a designed alternate renderer: transition among still/preen poses with slow cross-fades, cross-fade between perches instead of flight, remove leaf/feather drift, and slow day-to-evening palette shifts. Do not pause bird drift, mood or calls. Honor `prefers-reduced-motion` by default and allow an explicit settings override.

## 7. Audio pipeline

Synthesize calls client-side from a compact motif library with WebAudio oscillators/noise/envelopes and seeded runtime variation. Each bird keeps a recognizable signature across mood and trait drift; species supplies a motif family, bird seed supplies stable identity, and mood/frequency alter expression and probability rather than erasing motif. Schedule at most seven voices and chorus interactions. Reuse nodes/buffers where possible, dispose completed voices, and cap contexts/workers. Audio context startup follows browser gesture/autoplay rules; do not delay first bird paint for it.

On listen-in, ramp the selected bird up and other birds down to ambient, never silence, over a smooth configurable envelope; disengage on reselect, selecting another bird, empty-space click, or keyboard focus leaving. Return to ambient with the same ramp. Settle gently lowers calls. Offer-song fragments are soft procedural motifs. If WebAudio is unavailable or denied, remain silent and turn captions on by default; never bundle recorded fallback audio.

Captions are generated from the actual motif/parameters (“a low trill, paused, low trill again”), appear near the calling bird with the call, and fade naturally. Audio and captions share one call event source to prevent mismatches. Keep all user audio controls and status text accessible, and ensure a failed audio context does not create repeated error announcements.

## 8. Accessibility and performance budgets

Ship accessibility with v1. Provide a slow naturalist prose narration derived from the same snapshot as the scene, idle update every 30–60 seconds, with prompt but restrained priority for greeting, accepted offer, and settle. Use a polite live region/queue that coalesces superseded narration and prevents overlapping or high-frequency announcements. Do not expose internal vector values or merely enumerate state labels.

All interactive elements work with keyboard: top bar via Tab, scene entry focuses the first bird, arrow keys move among birds, Enter starts listen-in, Escape ends it, offer menu is keyboard navigable, settle is reachable, and focus leaves listen-in consistently. Focus outlines must remain visible on day and night backgrounds. Check screen-reader prose and call captions against visual copy voice, and all user copy against WCAG AA contrast.

Budgets and release gates:

- Initial JavaScript at first paint: under 2 MB gzipped; code-split settings, accessibility settings, and invite management.
- First bird visible: under 500 ms on a mid-tier mobile device over 4G, including authenticated snapshot path and a warm/cold start profile.
- Idle motion: 60 fps on a five-year-old mid-range laptop, tested for a sustained 30-minute session.
- No client memory growth over 30 minutes; bound audio contexts, workers, and retained notebook render nodes.
- Support the latest two major releases of Chrome, Safari, Firefox, and Edge; show direct matter-of-fact unsupported-browser guidance outside that range.

Measure aggregate page/load and first-bird timing, frame timing, audio-context failures, request errors, and tick latency. Synthetic browsers should cover common geographies. Alert when tick latency p99 exceeds five seconds. RUM carries no account dimension, bird ID/state, event payload, email, or interaction history. Keep the simulation database unreachable from analytics/warehouse credentials. Avoid recording presence totals or engagement funnels as product metrics.

## 9. Delivery sequence and rollout

1. **Foundations:** settle data contracts, threat model, schema/migrations, synthetic UUID rule, account/session storage, event idempotency, privacy review and deletion semantics. Build account auth plus empty account creation and two stable starter birds.
2. **Canonical engine:** implement snapshot contract, event log, serialized tick worker, event replay/retry behavior, mood transitions, timezone/day-phase, presence conjunction and bounded interval accounting. Validate calibration with synthetic accounts/events and fault injection before real-user rollout.
3. **Aviary surface:** implement composition and animation, active-first render, responsive three-depth scene, top bar, greeting, reduced-motion renderer and keyboard/focus controls. Tune first-bird time and sustained frame/memory budgets.
4. **Interaction and audio:** add listen-in events/mix, offers/cooldowns/reactions, settle/undo, procedural grammar, chorus, matching captions and graceful silence. Verify bird signatures remain distinguishable at seven voices.
5. **Notebook and access:** add sparse structured observations, narration queue, account settings, export, session revoke, verified email change, deletion/recovery, and screen-reader/contrast review. These are release scope, not post-launch accessibility work.
6. **Visits:** add explicit per-invite email flow, read-only visitor tokens/snapshots, revocation, expiry and transparent host log. Test visitor access cannot write any event, and visitor viewing does not affect host drift. Keep visits disabled until access-control checks pass.
7. **Controlled launch:** internal synthetic and staff accounts first, then a small production cohort, then progressive rollout (for example 1%, 10%, 50%, all) with pause/rollback gates on auth errors, cross-account access, tick latency, rendering budget, and audio failure. Roll forward only when privacy, accessibility, and performance gates pass. Track no engagement or ranking metrics. Configure age-based new-bird offers conservatively and allow server-side tuning without user-facing visit counters.

## 10. Risks and mitigations

- **Drift feels inert or too fast:** low-pass coefficients, presence window and caps are uncertain. Use synthetic week/three-week fixtures, inspect hidden instrument deltas internally, and tune versioned configuration. Reject any change that causes absence to lower traits or makes one session legible as stat movement.
- **Presence is inflated or lost while watching quietly:** browsers suspend tabs, focus/visibility signals vary, and activity window is intentionally generous. Send only eligibility heartbeats, bound server intervals, explicitly test visibility/focus/idle transitions, and tune the few-minute window without raw input collection.
- **Concurrent devices duplicate/lose changes:** enforce append-only sequence, unique idempotency keys, one serialized server writer and atomic state/cursor commits. Test overlapping events, retry, worker crash and replay scenarios; never add last-write-wins vector writes.
- **Tick backlog or clock changes cause jumps:** use bounded catch-up, persisted last tick and local timezone, monitor p99, and test DST/timezone edits and server downtime. Client interpolation handles presentation only.
- **Birds or calls feel uncanny/canned:** seed motifs per species/bird, vary grammar within identity bounds, stagger return greetings, and listen-test two through seven simultaneous birds. Do not solve with loops, repetitive canned greetings or visitor-only beautification.
- **Audio blocked by browser policy:** never gate scene rendering on audio. Start WebAudio on permitted gesture and default to captions when unsupported/denied; do not add a recorded-audio fallback.
- **Narration overwhelms or flattens affect:** use prose templates grounded in canonical scene facts, throttle/coalesce updates and prioritize only user-initiated moments. Test with screen-reader users as a release requirement.
- **Accessibility regresses during motion/design changes:** include reduced-motion, keyboard and contrast checks in every scene acceptance pass; ensure the alternate renderer is complete at launch.
- **Invite link leaks or remains valid after revocation:** store only token digests, scope tokens to one invite/read-only route, verify authorization on every pull, enforce expiration and revocation, and test attempts to call host mutation routes.
- **PII or relationship data enters telemetry:** use synthetic IDs, scrub structured logs, isolate credentials/schema, and review event schemas before shipping. Operational RUM uses aggregate metrics only.
- **Notebook becomes a feed or records user behavior:** gate on noteworthy aviary events, per-aviary sparsity/deduplication and prose review; prohibit session counts, visit cadence and vector values.
- **Account deletion leaves hidden state:** track all account-linked data stores, exports, mail artifacts and backups; exercise recovery during 30 days and verify hard purge at expiry.

## 11. Definition of ready to launch

Launch only when a new account receives two stable birds; state is identical across two signed-in devices; server restart/retry cannot lose or double-apply an event; mood persists and drift calibration is within target; no client API can mutate vectors; hidden/background clients do not accrue presence; invite visitors are read-only and revocable; notebook output is sparse and specific; the scene meets load/frame/memory targets; calls are procedural and captions match them; narration, reduced motion, keyboard navigation, focus and AA contrast are complete; export, recovery and hard deletion work; operational dashboards exclude account/bird behavior; and no out-of-scope social or gamified surface has entered the build.
