# Pocket Aviary v1 implementation plan

## 1. Outcome, scope, and product invariants

Build a browser aviary whose birds have persistent identities and slowly become more expressive through attention. The first meaningful frame shows an ongoing place, not an app starting. A return is noticed by a bird; the system never welcomes the user with a banner. The visual, audio, narration, caption, and reduced-motion experiences ship together.

V1 includes email magic-link accounts; one canonical aviary per account; two system-selected starter birds from six species; user naming and renaming; age-based optional adoption up to seven; three perch zones; local-time lighting; quiet occasional weather; mood-shaped motion; procedural calls and bird-to-bird responses; listen-in, offers, settle; a sparse read-only notebook with indefinite browsing; account export, deletion and recovery; device revocation; multi-device state consumption; and explicitly invited read-only visits. Support the last two major versions of Chrome, Safari, Firefox and Edge.

Exclude native clients, passwords and SSO, payments, scene customization, multiple aviaries, user placement of birds, shared ownership, recorded calls, public discovery, profiles, chat, comments, rankings, social cursors, hunger, death, distress, meters, achievements, streaks, visit-frequency surfaces, badges and notification engagement loops. No underlying cross-account interaction analytics should exist that could later become a leaderboard or recommendation feature.

Enforce these invariants in service contracts and automated checks:

- Bird UUIDs, persisted personality and call identity survive renames, deployments and migrations. No regeneration on login or cache misses.
- Only the simulation role writes personality; all deltas are nonnegative and applied exactly once to persisted vectors. No client absolute values or last-write-wins personality updates.
- Hidden personality numbers never appear in user-visible API payloads, DOM, ARIA, logs, debug controls or exported artifacts.
- Presence requires visibility AND window focus AND recent pointermove/keypress, and ends on settle, close, inactivity, blur or hidden state. Visitors contribute none.
- Calls are always procedural; lack of WebAudio yields silence with captions, never recordings.
- Absence cannot lower a trait, harm a bird or produce a guilt surface. Mood and recent-attention effects may change while baseline identity persists.
- Operational telemetry cannot read simulation events or include birds, names, traits or account interaction histories.

## 2. Decisions where the PRD needs interpretation

These decisions allow execution without new product questions. Keep them in architecture decision records during implementation.

**Export versus hidden vectors.** `accounts_sync.md` asks for personality vectors in exports, while `bird_engine.md` prohibits exposing numerical personality at any version or surface. Honor the stronger unconditional prohibition: export bird identities, names, species, current mood, notebook and settings, with naturalist qualitative descriptions in place of numeric vectors. Do not hide numerical vectors in an opaque downloadable file either. State the export contents plainly in settings. Internal backup preserves exact vectors and is unrelated to user export. This is a deliberate departure from that one export field, not an accidental omission.

**Visit notifications versus no notification surface.** The optional setting in `social_optional.md` conflicts with the product-wide refusal of aviary push/email notifications. Implement an off-by-default setting for an on-demand visit summary within account settings, visible only when settings are opened, without badge, toast, push or email. No unsolicited notice reaches the host. The always-available visit log remains accessible even when the summary setting is off. Authentication, user-requested exports and explicit invitations are transactional emails and remain in scope.

**Top-bar contents versus settle.** Keep the four required icon groups: account/settings, accessibility, notebook and offer. Put settle as a plainly named action in the offer menu, with a keyboard shortcut visible there. This satisfies top-bar reachability without a fifth icon. Invitations, names, adoption and audio preference live in settings or subordinate menus; no controls are embedded in the scene. Caption text and a visible keyboard-focus outline are explicit accessibility exceptions to the scene's no-chrome rule.

**Focus and listen-in.** Keyboard entry into the scene gives a bird navigation focus; Enter engages listen-in as required by the explicit keyboard specification. Pointer click/tap engages directly. Moving keyboard focus away disengages; moving to another bird disengages the previous bird, and Enter engages the new one. Focus alone is never counted as a meaningful listen-in duration. No additional mouse-hover behavior.

**Local time across devices.** Store one account IANA timezone, initially from the first browser and subsequently editable in settings. Both owner devices and visitors use that timezone. Do not let each browser independently override it, or a visitor and phone would see different canonical day/night states. Offer a matter-of-fact timezone setting for travel; no automatic timezone races.

**Starter and later adoption pacing.** Start with two distinct system-selected species. Offer an additional system-selected bird at aviary ages 90, 180, 270, 365 and 540 days, subject to the seven-bird cap. Eligibility is time-only and persists until accepted; declining or ignoring it costs nothing. Show availability quietly in bird settings on demand, never a countdown, reward or scene badge. This reaches five or six birds around a year without teaching attention-for-rewards. Selection is not rarity-based and the user never sees a catalog. The schedule is versioned and can be adjusted without changing already adopted birds.

**Fast response and slow tick.** A minute tick advances long-lived simulation, but delaying a greeting or offer by a minute would fail the product. Use the same serialized server engine for immediate event response planning, which creates canonical short action timelines. These event transactions do not write personality; only scheduled ticks consume events for drift. Mood changes induced by an interaction are authored server-side with timers, while scheduled ticks reconcile them. No client simulation of moods, reactions or personality.

## 3. Architecture and ownership

Use a TypeScript web client and Node.js service, PostgreSQL as the source of truth, a durable minute scheduler/worker and a transactional mail delivery adapter. Begin as one deployable API with separate worker processes, not independent microservices. Share versioned schema validators and presentation timeline types; keep the server engine and protected personality types in a server-only package. Use SQL migrations and explicit DB roles to separate event ingestion, simulation mutation, account lifecycle and snapshot reads.

Service modules: identity/session, aviary queries, owner commands/events, simulation, notebook generation, invitations/visits, account lifecycle/export, and privacy-filtered operational metrics. The worker owns scheduled ticks, export jobs, mail delivery and deletion jobs. Job payloads use synthetic IDs and token references, never email. Mail dispatch resolves encrypted recipient fields only at delivery; redact adapter errors and request bodies.

The browser owns rendering, interpolation, accessibility presentation and synthesis of server-described calls. It owns transient UI state such as focus, dialogs and listen-in gain. It cannot decide canonical perch moves, mood changes, weather, greeting selection, acceptance of offers or drift. Client-only leaves, feathers and subtle parallax are ornaments without persistent state or simulation input.

Render with a small Canvas 2D scene renderer, compact species shape/pose assets and HTML semantic controls over a separate accessibility layer. Keep renderer independent of React or similar UI reconciliation; a small component framework can manage menus and settings. Avoid a large game engine. Offscreen prerender static foliage and perch layers, composite birds with transforms and palette parameters, and keep simulation updates out of the animation-frame hot path. Use shared presentation descriptors for Canvas, reduced-motion poses, captions and narration.

CDN serves versioned static assets and generic HTML shell. An edge bootstrap handler validates the owner's session and fetches a compact presentation snapshot from the regional canonical service, embedding it in private, uncacheable HTML. Never publicly cache account-specific HTML. A CDN must not become a public account-state cache. Visits use the same renderer and projection through separately authorized read endpoints. Deploy the database and tick workers in one region initially; scale workers by synthetic aviary ID partitions before considering distributed state writers.

## 4. Persistent data model

All records carry schema/version fields and timestamps in UTC. UUIDs are synthetic, never email-derived. Use foreign keys and account-cascade relationships so deletion coverage is discoverable.

| Record | Important fields and constraints |
| --- | --- |
| Account | UUID, encrypted verified email, keyed email lookup digest, IANA timezone, locale, settings, created_at, deletion_requested_at. Email ciphertext lives here once; digest is restricted to authentication lookup and never a service identifier or log key. |
| Auth challenge | UUID, account or provisional-account UUID, token digest, purpose, expires_at, consumed_at, attempt bookkeeping. A provisional account holds its email in the account record rather than duplicating it in challenges. |
| Device session | UUID, account UUID, refresh/session token digest, created_at, last_seen_at, expires_at, revoked_at, coarse user-provided/device display name. No precise fingerprint. |
| Aviary | UUID, unique account UUID, created_at, state_revision, last_tick_at, event_cursor, RNG state, canonical timezone, weather timeline, recent-attention envelope, simulation/config version. |
| Bird | Stable UUID, aviary UUID, species/version, mutable display name, adopted_at, immutable identity seed and call-signature parameters, five persisted traits, low-pass filter state, current mood, mood_since/next_baseline_at, perch, active action timelines, per-bird offer cooldown. Never recreate from event history. |
| Interaction event | UUID, aviary UUID, owner session/client stream UUID, sequence, server receipt time, bounded client monotonic duration data, type, bird UUID if applicable, validated payload, processed tick revision. Append-only until its retention/deletion boundary. |
| Presence lease/interval | Owner stream, validated qualifying interval, expiry, closed reason, tick consumption cursor. Union intervals across devices per account; never sum overlapping attention twice. |
| Notebook observation | UUID, aviary UUID, observed_at, bird references, names as observed, prose, generator version, deduplication key. Entries immutable; no edit/delete/annotation endpoint. |
| Adoption entitlement | Aviary UUID, age threshold, stable offer seed/species, accepted bird UUID if any. Acceptance unique per threshold and transactional with bird count enforcement. |
| Invitation | UUID, host account UUID, encrypted invitee email, token digest, issued_at, expires_at, consumed_at, revoked_at, linked visit-session UUID. Non-account invitee PII is stored only in this scoped relation, never as an internal identifier. |
| Visit session/log | UUID, invitation UUID, capability digest, issued_at, expires_at, last_pull_at, ended_at, approximate duration, status. Join invitee email only when the host opens the log. No visitor events or presence records in simulation tables. |
| Lifecycle job | UUID, account UUID, type, status, attempt count, due_at, artifact reference/expiry where relevant. |

Normalize traits to [0,1], seed each species with bounded variation and avoid starting close to saturation. Persist filter accumulator and delta application cursor atomically with traits. Keep an internal invariant ledger of applied tick revision/checksum sufficient to detect double application, without copying raw traits into telemetry. Private database integrity checks can validate values; their operational output is success/failure only.

Snapshot projection explicitly allowlists fields: revision, server time, timezone/day phase, weather, bird UUID/name/species, qualitative behavior style, pose/perch/action timelines, stable public motif family, call descriptors and next-refresh hint. Do not send five scalar traits or a one-to-one reencoding of them. Presentation styles are coarse many-to-one descriptions; continuous visible plume color can be a rendered palette choice rather than a exposed trait scalar. A viewer can observe a bird's appearance, but receives no stat interface.

## 5. HTTP API and command contracts

All JSON contracts are schema-validated and versioned under `/v1`. Owner endpoints require secure HttpOnly SameSite cookies, CSRF protection for mutations, origin checks and account ownership checks. TLS everywhere. Use request IDs without sensitive payloads. Tokens have at least 256 bits of cryptographic randomness and are stored as digests. Link pages set no-referrer policy and no third-party assets; consume tokens by POST and redirect to clean URLs to limit history/log leakage.

| Endpoint | Contract and result |
| --- | --- |
| POST /auth/magic-links | Email input; generic acknowledgment to avoid account enumeration; per-address keyed-digest and IP throttles. Start with 5 requests/hour/address plus abuse backoff, not permanent lockout. Link expires in 15 minutes. |
| POST /auth/consume | Atomically consumes one unused, unexpired token and creates one per-device session. Concurrent replay: one success, others clear matter-of-fact expired/used response. |
| GET/DELETE /account/sessions[/:id] | List/revoke own devices; revocation immediately blocks authenticated requests. Support sign-out of current device. |
| POST /account/email-change; POST /account/email-change/verify | Verify new encrypted pending address before a transaction switches the account email and lookup digest. Old address remains active until verified. Purge pending address afterward. |
| GET /aviary/snapshot | Presentation snapshot, monotonic revision, server time. ETag supports unchanged responses; private/no-store where needed. Fetch on navigation, resume and keepalive. |
| POST /aviary/streams | Registers an owner client stream and return event; returns an authorized greeting timeline and current snapshot. Not called by visitor code. |
| POST /aviary/events | Bounded batch of presence, listen-in start/end, offer, settle, reengage, audio preference changes and stream close. Each has event UUID and stream sequence. Returns accepted/rejected IDs, canonical reaction timeline when relevant, revision and cooldown availability. |
| PATCH /birds/:id/name | Name only, length 1–40 Unicode characters after normalization, sanitized as plain text. Use expected name revision for concurrent renames; stale request returns current name for explicit retry, not silent overwrite. |
| GET /aviary/adoption; POST /aviary/adoption/accept | Private age-based availability; acceptance creates exactly one stable bird with a user name/default, locks aviary row and enforces cap seven. |
| GET /notebook?before=cursor&limit=25 | Immutable observations newest-first, stable keyset pagination. No edit/delete endpoint; account deletion removes all. |
| GET/PATCH /account/settings | Timezone, audio, narration/captions, reduced motion, optional settings-only visit summary. Expected settings revision prevents invisible lost edits; unrelated fields patch independently. |
| POST/GET /account/invitations; DELETE /account/invitations/:id | Explicitly invite one email, list own invitations/log, revoke. No public index or invitation fan-out. Mail only follows an intentional send. |
| POST /visits/consume | Consume unexpired invitation exactly once; issue read-only capability in a secure cookie. Default visit session 24 hours, with no automatic renewal/re-invitation. |
| GET /visits/snapshot | Same visual/audio projection with host timezone; checks revocation every request; excludes notebook and owner/private settings. Visitor heartbeat updates only approximate visit log duration. |
| POST /account/exports | Generate snapshot at one consistent DB revision, send signed download link to verified address. Short-lived artifact (24 hours) and one-time download token. |
| POST /account/deletion; POST /account/recover | Mark deletion immediately with 30-day deadline; show recovery action on signed-in pages. Recovery before deadline resumes same birds and vectors. |

Owner mutation response codes: 401 for expired/revoked sessions, 403 for wrong capability, 409 for stale user-edit revision, 422 for invalid event, 429 for rate limits, 503 for unavailable canonical state. Display persistent inline system messages and retry actions in matter-of-fact voice, not toasts over the aviary. Cooldown is a quiet disabled menu action with accessible explanation, not punishment or countdown game UI.

The event endpoint rejects visitor capabilities regardless of requested URL. Restrict acceptable event names/payloads so no trait, mood or arbitrary state patch can be accepted. Idempotency keys and unique stream sequences make retries return the original result. Server receipt order supplies a per-aviary event sequence; client clocks never arbitrate state. Sequence gaps and delayed listen-end are bounded by lease expiry. Event batching is capped at 50 events/64KB and command rate-limited without limiting ordinary watching.

## 6. Simulation engine and calibration

### Tick transaction and continuous state

Schedule each existing, non-deleted aviary every 60 seconds, including those with no clients. Spread due times across the minute using a stable UUID hash. A worker takes a database advisory/row lock for the aviary, reads its stored state and ordered events after its cursor, advances timers and ambient state, computes nonnegative personality deltas, writes timelines/moods/notebook observations, advances cursor and revision, then commits. Retries cannot apply drift twice. Commands use the same lock and update compatible timelines so offer, tick and rename cannot interleave destructively.

Use deterministic seeded random streams separated by bird behavior, weather and greeting so a new ornament does not change unrelated behavior. Persist RNG state/version and action start timestamps. Randomness varies durations, path control points, head-angle envelopes and call motifs; it is not cycling three canned animations. Stable species silhouettes and identity call anchors remain recognizable.

On worker downtime, process elapsed minute boundaries in bounded batches using persisted state and validated historical inputs. Never fabricate presence to fill gaps. Keep ticks ordered per aviary; alert on backlog. Snapshot requests can prioritize an overdue aviary through the same engine, but cannot create a second writer. Large catch-up periods may coalesce absence-only timer evolution using a tested mathematically equivalent transition algorithm; they may not rebuild vectors from event history or reset mood. First ship straightforward bounded stepping and benchmark long-absence recovery before optimizing.

### Drift function

For trait j on a bird, store filtered input F_j and personality P_j. Each tick of length dt updates `F_j = exp(-dt/tau) * F_j + (1-exp(-dt/tau)) * I_j`, with initial tau 3 days. Compute `delta_j = min(per_day_remaining_cap, k_j * F_j * (1-P_j) * dt_days)` and `P_j := clamp(P_j + max(0, delta_j), 0, 1)`. This is an additive server-authored delta, never a replacement derived from client data. Positive retained filter input can continue gradual drift briefly after a visit; zero input eventually leaves traits unchanged, never decreases them.

Input I is bounded and weighted: qualifying presence supplies at least 80% of the maximum daily signal; listen-in and offers share the remaining at most 20%. Use 20 minutes of honest daily presence as the initial reference dose, with a smooth saturation curve and an upper cap at 60 minutes. Watching beyond the cap still counts as attention for mood but cannot accelerate drift indefinitely. Listen-in contributes only during qualified owner presence, capped per bird/day; offer near a bird nudges boldness, accepted investigation nudges curiosity, and accepted song responses may affect vocal expression. Rejected/cooldown offers supply no repeated delta. Settle adds no personality input. No click-count reward and no muted-audio penalty; audio preference is an expression/modulation choice, not a reason to remove personality progress.

Start coefficients so a 20-minute/day synthetic owner produces approximately 0.01–0.025 absolute trait change after seven days and 0.04–0.08 after 21 days from midrange seeds, with less than 0.003 in any one session. These are initial engineering targets, not surfaced product statistics. Validate exact values numerically using virtual time and compare motion/audio at weeks zero and three in blinded qualitative reviews. Tune trait-to-expression mapping as well as coefficient: numerical drift alone is insufficient if a user cannot feel it. Run all calibration on synthetic personas, never aggregate real interaction histories.

Separate a short-lived recent-attention envelope from personality. It can decay toward neutral ambience during absence, lowering greeting eagerness and immediate chorus participation without lowering baseline vocal trait, color, trust or curiosity. Never encode prolonged absence as a move toward distress or habitual wary mood. A long-return greeting can be longer in form while quieter in intensity.

### Mood and bird social behavior

Finalize five moods: wary, content, curious, drowsy and alert. Maintain baseline resampling timer near 24 hours with per-bird jitter; use smooth time-of-day weighting, recent reactions, short weather effects and personality to choose transitions. Login is not a reset trigger. Mood changes generate gentle pose/perch trajectories, not numerical labels. Local dusk biases drowsy, morning alert, accepted seed content; rain temporarily reduces call propensity without changing vocal personality. A nightjar-like species retains night activity.

Model bird-to-bird responses as scheduled events: a call may elicit one or two staggered replies, a brief alarm can nudge nearby birds toward wary, and overlapping compatible call windows can form a chorus. Bound propagation to avoid alarm cascades or constant calling; use refractory intervals and low response probability. Each bird has its own trajectory and next action. Keep all seven in frame with occupancy-aware perch slots, not user-controlled placement.

Weather uses a persisted aviary seed and schedule: initially 2–3 short rains per synthetic week, 2–5 minutes each, with intermittent gentle wind. No severe weather or attention demands. Motion ornaments are client-only and do not need server records.

### Greeting and immediate commands

On an owner return, derive absence from server-known owner visibility/presence/session transitions, not visit sessions. Choose one initial greeter by weighted boldness, warmth and current mood; avoid always choosing the same bird. Build a unique action with slight timing, gaze, step and call variation. Target start within 1–2 seconds of return; shorter absences favor glance/reorientation, longer ones favor a measured approach or call. Any other bird's response follows with random offsets; never cue all birds simultaneously. Deduplicate visibility/focus events within a short transition window (initially 10 seconds) so focus flapping does not retrigger welcomes or create artificial attention input.

Offer is an aviary gesture selected in the top bar, optionally directed through a receiving-bird choice in that menu. Server selects an eligible receiver if unspecified and writes the reaction: approach, wait, watch, bathe, drink, respond or ignore. Cooldown is initially 3 minutes per receiving bird across devices and offer types. Song fragments are a small procedural motif library played softly, not user uploads. Immediate command planning acknowledges input quickly while trait effects wait for the tick.

Settle closes the requesting stream's attention/listen-in windows immediately, quiets that client mix and sets a session-scoped lighting override. It supplies a small canonical mood-quieting event, but does not forcibly end a separate active device's presence. For five seconds, any scene click cancels settle and sends reengage; after five seconds explicit bird/menu interaction reengages. Use equivalent keyboard input to undo/reengage. Retain timestamps to reject stale undo from another stream. Closing without settle closes/lets the same lease expire, with identical drift treatment and no missing-goodbye notice.

## 7. Presence, sync and disconnect behavior

Choose a five-minute activity window initially, calibrated with synthetic watching scenarios and qualitative accessibility review. Track last pointermove or keypress with a monotonic client clock. At each presence interval evaluate all three conditions; break the interval immediately on blur, hidden, settle or expiration. Activity updates the recent timestamp but counts only while the conjunction holds. Ordinary clicks/taps generally create pointer activity; do not silently broaden eligibility to a tab-open check. Validate touch and assistive-keyboard browser event behavior, document any access gap, and keep the same strict gate for all supported paths.

Send qualifying intervals every 15 seconds with state flags, sequence and durations. Server bounds reported duration by receipt elapsed time, prior lease and maximum activity-window expiry; reject future timestamps or gross clock discrepancies. Lease expires after 30 seconds without refresh, preventing a crashed or sleeping client from accruing hours. Close/hidden uses a best-effort beacon, never relies on it. Do not backfill long offline spans or device sleep. Client signals approximate attention and are not proof of a human, so cap and saturate drift instead of adding invasive surveillance.

Union intervals from concurrent owner devices before integrating presence; a person cannot create twice the daily dose by opening two tabs. Listen-in duration has matching leases and ends when presence ends, focus exits, another bird takes attention or session expires. Bounded offline retries can replay discrete commands with original IDs, but stale offers/settles (older than two minutes) are rejected rather than performed hours later. Never replay an entire offline session as presence. Persist only a small in-memory retry queue; refresh authoritative state on recovery.

Owner snapshots: embed at navigation, pull every 30 seconds while visible, pull immediately after relevant command responses, visibility return, online recovery and render-frame gaps over 2 seconds. Hidden tabs stop animation frames, ornamental work and call scheduling; keep simulation on server. Resume fetches fresh state before continuing stale timelines. Visitors pull every 10 seconds while visible, and before resuming, to bound revocation exposure to the next pull. Do not continue a visitor animation indefinitely on an expired network response: stop displaying after a 15-second freshness limit during outages.

Each response has revision and server time. Ignore lower revisions and cancel obsolete timelines; interpolate positions/poses from absolute timeline timestamps using a smoothed server clock offset. On brief delay finish valid timelines without predicting mood/drift; on longer outage retain the last birds with quiet ongoing decorative pose changes and a matter-of-fact reconnect surface outside the scene. No invented calls beyond the schedule horizon. Never make stale state look like successful sync. Name/settings conflicts use explicit revisions; event conflicts disappear through server ordering and idempotency. A phone/laptop comparison should receive identical canonical moods and weather at the same revision, with device-local listen-in gains allowed to differ.

## 8. Scene and interaction rendering

Scene composition layers: time-colored sky and distant foliage; back/middle/front perches with depth scale; birds and shadows; sparse foreground ornaments. Layout maps normalized coordinates into available viewport bounds beneath the thin bar. Use responsive perch spacing and a safe inset for bird extents, captions and focus rings. On narrow portrait screens reduce gaps/scale assets and retain a horizontal scene in a fitted region; do not pan, scroll or crop the aviary. Menus and notebook may scroll within their own surfaces. At seven birds use collision-aware slot arrangements and compact caption lanes, not offscreen overflow.

Assets include six silhouettes, pose families for scan, preen, tilt, shuffle, rest and flight, muted palettes and signature motif data. Personality influences server-selected behavior and visual style, mood changes the current pose/timing. Renderer evaluates timeline progress at first paint using server time, so a returning bird is already mid-preen, not starting at frame zero. Bootstrap must include at least one active timeline. Loading without a usable snapshot uses soft sky and faint ambient color/motion, no spinner, fade-from-static or welcome. A failed load gets a clear error outside the scene. Only initial adoption permits a true empty scene and gentle fly-in of the new starters; subsequent navigation never repeats adoption entry effects.

Normal idle uses low-amplitude breathing, head movement, preening and weight shifts with varied phase/duration. Front/back travel follows soft paths and depth changes. Gentle parallax is autonomous, not rapid cursor-following motion. Stop ornament creation when hidden. Reduced-motion mode removes drift/parallax and flight paths, uses slow cross-fades between thoughtfully designed still poses/perches (initially 2–4 seconds), and slows lighting transitions (8–12 seconds). A change of preference switches at a pose boundary without flash. Audio, captions, moods, notebook and drift remain fully functional.

Top bar fades nearly transparent after four seconds of pointer inactivity, but stays fully visible on keyboard focus, open menu, focus-visible navigation or accessibility interaction. Pointer movement/key activity restores it. Never hide active focus or shrink hit targets; minimum targets 44px with text labels available to assistive tech. Pointer click a bird toggles listen-in; empty-scene click disengages. No hover labels or scene icons. Naming/settings are reached from account settings, not a tooltip over a bird.

Notebook uses keyset fetches and bounded DOM virtualization with accessible pagination/load-more alternatives. Old observations remain available indefinitely; unmount recycled rows and release event handlers so browsing does not retain the entire history. Generate specific entries from noteworthy canonical facts (a changed first greeter, an unusual perch choice, a quiet-weather moment), not session timestamps or visit streaks. Store observed names so old prose remains coherent after rename. Initial sparsity gate: at least 48 hours between routine entries, average about one per three days for a regular synthetic aviary, with a limited exception for distinctive observations. Deduplicate by observation key and cap novelty bursts; templates have varied clauses derived from real state, not generic happiness messages. No generative model or outside data processor sees private interactions.

## 9. Procedural audio and captions

Server plans compact call descriptors with bird ID, signature family/version, seeded motif sequence, absolute onset, durations, pitch contours, rhythm, envelope, timbre and response links. The client performs synthesis only. Preserve immutable signature anchors such as characteristic interval pattern and timbre range; mood changes tempo/intensity and trait expression changes propensity/variation inside recognizable bounds. No recorded files or loop playback for calls, songs or fallback.

Use one AudioContext per tab with bounded oscillator/noise voices, envelopes, filters, per-bird gain and a gentle master limiter. Begin with WebAudio native nodes; introduce AudioWorklet only if measured quality/CPU requires it. Schedule in short lookahead windows against the mapped server clock; skip calls whose onset is substantially past rather than replay a burst after resume. Reuse procedural noise buffers; disconnect finished nodes and bound outstanding scheduled events. A chorus contains independent descriptors with slight onset/pitch variation and gain headroom, not two identical waveforms stacked. Cap simultaneous voices and moderate spatial stereo so seven signatures remain distinguishable without fatigue.

Listen-in changes focused gain gradually (initially 1.2 seconds in, 1.8 seconds out); other birds reduce to audible ambient, initially -9dB, never zero. Repeated focus changes cancel/retarget gain automation smoothly. Track focused duration only while eligible presence holds. On disengage, settle, blur/hidden or context suspension, close listen-in appropriately. Audio off and caption-only users still receive meaningful focus and bird attention behavior; mute never reduces personality.

Attempt context start within browser policy. Fresh navigation cannot promise audible playback before a user gesture. Render the bird and procedural call captions immediately, provide a quiet labeled audio-enable action in settings, and resume on permitted user activation. No autoplay modal or toast. If unavailable/denied/broken, force captions on for that session and stay silent; retry only through an explicit system control. Suppress delayed audio blasts on resume. Persist audio/caption preference and make fallback status clear in accessibility settings.

Generate caption prose from the actual finalized call descriptor, including count, contour, texture and pauses, rather than a fixed string on the species. Place captions near the source with stable, high-contrast backplates and fade duration matching the call. Coordinate simultaneous captions to avoid overlap; limit density with coherent chorus prose when necessary while identifying sources accessibly. Captions continue when muted or fallback-silent because they describe intended procedural calls. In reduced-motion they appear/disappear with restrained opacity changes. Do not feed every caption into the narration live region, which would overload it.

## 10. Accessibility and voice implementation

Maintain a semantic scene description and roving-tabindex bird list parallel to Canvas. Each bird has a name/species accessible label and interaction instructions; never ARIA personality numbers or raw mood codes. Tab reaches the four top-bar groups then enters the first bird; arrows move among stable bird IDs, Enter engages listen-in, Escape disengages/returns as appropriate. Menus have predictable focus trapping and restoration, including offers and settle. Visitors receive semantic bird descriptions without active owner controls. Accessible focus outlines remain visible against day/night palettes and do not function as persistent scene chrome.

Generate narration from the same presentation snapshot and canonical events: lowercase present-tense prose naming birds, perch character, light and calls. Default idle cadence 45 seconds, configurable within 30–60 seconds; deduplicate unchanged descriptions. Use one polite live region with a latest-state queue, not append-only announcements. Greeting/offer/settle gets prompt priority after current speech, coalescing closely spaced events rather than interrupting system errors or dialog focus. Narration pause/repeat and optional visual transcript live in accessibility settings. Call captions are separate from narration and can remain on with narration off.

Honor OS reduced-motion by default; an explicit user setting can choose reduced mode across devices, with system preference evaluated safely at startup before scene motion. Audit all copy and focus/caption states for WCAG AA: 4.5:1 normal text, 3:1 large text and meaningful non-text focus/control boundaries. Do not rely on bird color or audio alone for interactive state; semantic listen-in status and distinct poses/narration carry it. Accessible settings use normal system prose; observation surfaces use the naturalist voice.

Voice work is a deliverable: template libraries, prohibited-word tests on authored copy, and editorial review of combinations. Never auto-generate "welcome back," missing-user duration, session-count entries or achievement language. System errors directly explain action and recovery. Manual test with VoiceOver/Safari and NVDA/Firefox or Chrome, keyboard-only users, captions-only users, zoom/reflow, high contrast and reduced-motion users. Evaluate whether it feels like observing birds, not merely whether controls have labels.

## 11. Privacy, security and lifecycle

Operational telemetry uses an explicit allowlist of metric names/fields and a separate collector that has no credentials/network access to simulation tables. Record request counts/latencies, response-class errors, anonymized session-duration buckets, first-bird time, frame timing, audio failures and tick duration. Never log command bodies, bird IDs/names, notebook prose, presence intervals, offer/listen-in events, trait deltas or account-dimensional session duration. Restricted service-error correlation may use account UUID for debugging auth/service failure, without bird fields or user behavior; use short retention and do not feed it into aggregate relationship analytics. Synthetic test accounts are visibly segregated from production, and all drift dashboards are synthetic-only.

Interaction logs are simulation inputs, not a warehouse. Retain processed raw events for seven days for bounded retry/integrity handling, then purge after their durable contributions/filter state have committed. Keep deduplication keys without payload for 30 days, bounded by stream validity. Notebook observations and bird state persist for account life. Visit logs retain only invitation identity, approximate duration and timestamps; no page interaction recording. Email ciphertext for named non-account invitees is necessary PII distinct from owner account email and stays confined to invitations, with no analytics use. Purge expired invitation PII after a 90-day host transparency window unless an active visit still needs it; expose that retention plainly in settings.

Issue owner sessions with bounded lifetime (initially 30 days, renewable on valid activity), revoke via a centrally checked store, and prevent deleted accounts accepting events. Invitations expire unused after 30 days; consumed visit capabilities cannot mutate and end after 24 hours or revocation. Check permissions against host deletion and revoke status on every pull; never rely on cached capability authorization. Email delivery may transit a provider solely for authorized transactional delivery; simulation and interaction contents never go to that provider. Avoid any third-party analytics SDK and scrub exported artifacts from access logs.

Soft deletion immediately stops simulation/commands, revokes visits, cancels export links/jobs and suspends ordinary account use. Allow magic-link sign-in to a restricted recovery screen during 30 days; "I changed my mind" restores the same IDs/vectors and restarts mood time advancement without manufacturing absence drift. At deadline, lock account and purge all relational state, events, visits/invitee PII, export objects, sessions, queued mail and account-associated diagnostic records. Race between recovery and purge is transactionally decided by the deadline/row lock. Track deletion job completion and failure, never silently drop failed jobs.

Encrypt per-account private data using envelope keys; hard deletion destroys the per-account key as well as deleting live records, so retained backup media cannot recover the account's relationship data. Backup retention is finite (initially 30 days), restores apply a tombstone/key-deletion ledger before opening service, and restore tests confirm deleted data cannot reappear. Do not claim privacy compliance merely because SQL rows were removed. Export links expire and artifacts are encrypted, access-controlled and excluded from public CDN caching. Export generation uses a consistent transaction and never exports secrets, raw presence logs, device tokens or other accounts' PII.

## 12. Performance budgets and observability

The PRD targets are release gates, not aspirations. Measure the first bird at actual renderer draw, not first HTML byte or framework mount. Define supported performance fixture: a physical mid-tier mobile device over repeatable 4G shaping, authenticated navigation without warm asset cache; document network RTT/throughput and report distributions alongside the threshold. Anonymous sign-in/adoption time is outside this timing, but first aviary navigation after adoption is measured. Capture reduced-motion first pose using the same mark.

| Budget | Engineering allocation / verification |
| --- | --- |
| Initial JavaScript <2MB gzip | Aim for <250KB gzip of critical renderer/UI/state bootstrap to make 500ms credible; lazy-load account, notebook, visit management and deeper settings. CI asserts compressed first-paint dependency graph, not just entry chunk. |
| First bird <500ms on mid-tier mobile/4G | Target edge/bootstrap TTFB <150ms, critical transfer/parse <200ms, first scene draw <50ms, leaving margin. Embed compact snapshot and minimum species shapes in initial response; no waiting for audio initialization or noncritical textures. Report p50/p95 and failure fraction in the agreed synthetic fixture; gate p95 <500ms there. |
| Small snapshots | Target two-bird snapshot <8KB compressed and seven-bird <20KB, with bounded timeline horizon and no full notebook/event history. |
| 60fps for 30 minutes on five-year-old laptop | 16.7ms frame period, renderer CPU p95 <8ms and no sustained missed-frame bursts. Test two and seven birds, captions and chorus under real viewport resizing. |
| No memory growth over 30 minutes | Warm up five minutes, sample retained heap/native resources after controlled collection at 5/15/30 minutes; no continuing upward trend or retained call/row objects. Treat >5% or >5MB unexplained increase as failure; inspect smaller trends rather than declaring them harmless. Bound buffers, nodes, workers and caches explicitly. |
| Tick p99 <=5 seconds | Aggregate tick compute latency alert above 5s; additionally track due-to-commit lag and failed job counts so a quick tick with huge queue delay is not hidden. Initial lag alert >120s. |
| Immediate command response | Target p95 <300ms regionally, with greeting start <=2s including fetch/render. No loading theatrics while commands are pending. |

Synthetic browsers run normal/reduced-motion/caption/silent cases from common geographies using synthetic accounts. RUM transmits sampled timing histograms without stable account/device/bird dimensions. Bundle, frame and memory suites run against production builds; test hidden/resume, laptop suspension and 30-minute chorus/listen-in cycling. Broad telemetry coverage cannot be used to infer which real bird was offered what. Alert on auth/mail errors, snapshot failures, job lag and audio initialization failures. Unsupported browsers get a direct explanation of supported versions; do not add old-browser polyfill weight to critical bundles.

Cold state delivery is the greatest performance risk: a 2MB limit alone cannot produce 500ms over 4G. Prototype and measure the private edge bootstrap before building complex assets. If latency misses, first reduce payload, parse work and geographic round trips; never show invented default birds or a public-cached personal snapshot to improve the metric. Offline/cold outage uses quiet field/error and is reported as a failed first-bird sample, not excluded opportunistically.

## 13. Implementation sequence and acceptance evidence

### Stage A — contracts, engine skeleton and first-frame proof

Define DB schema/roles, snapshot/event schemas, timezone decision and privacy field allowlists. Build synthetic fixture generator and virtual clock. Prototype two species, mid-action bootstrap rendering, silent fallback, reduced-motion poses and narration. Implement persisted identities, deterministic minute tick and immediate serialized event planning. Exit when a synthetic aviary advances without a browser, a return draws in-progress poses, and the mobile first-bird fixture passes. Do not postpone accessible architecture until art completion.

### Stage B — private owner vertical slice

Magic-link lifecycle, device sessions, starter adoption/naming, per-owner snapshot and event ingest, strict presence interval accounting, greeting, listen-in gain ramps, seed/song/pool reactions and settle/undo. Complete six species silhouette and signature libraries. Exit with single-session and concurrent-device correctness, successful browser audio restrictions/fallback, keyboard flows and all scene actions available without overlays. No story surface for missing goodbye.

### Stage C — continuity and account completeness

Calibrate drift with accelerated synthetic weeks, mood persistence/timezone/DST/weather, bird social behavior, sparse notebook, age eligibility and atomic adoption cap. Add account settings, export policy, email verification, recovery/hard deletion, session revocation and privacy text. Exit with stable identities/vectors through deploy/restore, verified deletion coverage and accessible indefinite notebook retrieval. No numerical traits in output contract snapshots or exports.

### Stage D — quiet visits and launch hardening

Explicit invitation issuance, one-time consumption, 30-day expiry, 24-hour read-only sessions, revocation, host on-demand log and optional settings-only summary. Run permission fuzzing to prove visitor routes cannot create presence or events. Finish caption collision handling for seven birds, focus contrast all phases, browser/manual assistive tests, performance/memory soak and operational fault drills.

### Stage E — release and ramp

Ship all accessibility/account/privacy/visit features in initial v1. Begin with an internal synthetic environment and small consenting owner pilot; then a staged 1%, 10%, 50%, 100% account rollout using operational health and qualitative feedback, never engagement rates or real drift aggregation. Hold each step through a 30-minute performance soak and a full daily mood boundary; allow several weeks of pilot observation to validate affective drift. Freeze signups/ramp when auth, state integrity, accessibility or timing gates fail. Ordinary owners start with two birds; test fixtures cover seven from day one.

Ramp live additional-bird support separately, initially enabling the age-gated third-bird entitlement for eligible accounts, then thresholds through seven after recognizability/performance validation at each count. Feature flags may defer an offer but cannot remove adopted birds, change IDs or reset vectors. Existing owners keep their actual aviary during rollback. Do not accelerate adoption for frequent attendance. Use virtual age only in test accounts so a year-long feature does not need a year before QA.

Rollback client releases via immutable asset versions and compatible snapshot contracts. Roll back workers only across validated schema/config versions; preserve RNG state and durable vectors. A problematic calibration can pause new positive deltas and retain state while repaired, never decrement prior drift. Back up before migrations, test restoration and publish clear system failures if canonical state cannot be served. No rollback that silently replaces birds.

## 14. Required verification matrix

Engine properties: traits bounded and monotonic; zero new presence cannot generate negative deltas; a residual filter produces only bounded positive drift; one session below perceptibility; weekly/three-week targets; mood not reset by login; stable IDs and motif anchors; per-bird cooldown across devices; seven cap under concurrent acceptance; rare weather; bounded chorus/alarm propagation. Deterministic tests use virtual time, randomized event sequences and multiple seeds, not just one happy path.

Presence properties: all eight combinations of the three gates; five-minute idle expiry; hide/blur transitions; settle/undo; lost close beacon; stream crash; suspend/resume; clock skew; stale retry; two simultaneous devices unioned; visitors entirely excluded. Check rendered-but-unfocused windows and focused-but-hidden pages explicitly. A background overnight tab produces zero sustained presence input.

Transaction properties: duplicated/out-of-order submissions, concurrent tick workers, command/tick interleaving, crash before/after commit, queue replay, snapshot revision reordering and network partition. Compare resulting persisted vector to a serial reference processor and verify cursor advancement never discards or repeats inputs. Restore backup and migrate species/config versions without replacing bird identity or losing accumulated traits.

Account/social security: magic-link concurrent consumption and 15-minute expiry, CSRF, unauthorized bird access, revoked session, unverified email change, deleted host, invite reuse/30-day expiry, revoked active visit at next pull, stale visitor network cutoff, visitor command injection, export expiry and hard deletion including diagnostics/backups. Confirm mail jobs carry IDs rather than raw recipient fields and no token appears in access logs.

Affective/accessibility review: repeated greetings with short/long absences and different moods are specific and procedurally varied; no textual welcome; muted/reduced-motion/narrated experiences remain alive; call captions match generated contours and count; signature recognition blind tests at two through seven birds across moods; naturalist notebook specificity and sparsity; no observed user-frequency prose. Keyboard/assistive tests include offer, settle undo, naming, adoption, notebook, invitations and recovery.

Performance/privacy gates: production bundle dependency graph, cold mobile first bird, 30-minute seven-bird render/audio memory soak, long-hidden resumption, forced WebAudio failure, private-cache isolation and telemetry schema validation. Seed forbidden sensitive fields into test events and verify collectors reject them; prove analytics roles cannot query simulation data. Synthetic operational dashboards provide day-one alerts without relationship metrics.

## 15. Main risks and response

| Risk | Prevention and fallback |
| --- | --- |
| Drift feels instant, invisible or saturates | Synthetic weekly trajectories, bounded input/caps, blinded three-week comparisons, tune mapping and coefficients together. Pause deltas without subtracting accumulated personality. |
| Presence inflation quietly breaks calibration | Strict conjunction, short leases, interval unions, sleep/hidden tests and daily saturation. Never use open-tab duration or real population behavior analytics. |
| Data loss removes identity continuity | Durable vectors/filter state, transaction cursor, restricted DB writers, backup/restore and migration invariants. Fail state reads clearly rather than regenerate missing birds. |
| Greetings or calls feel canned | Procedural envelopes and motifs, stable identity anchors with variation, review repeat sessions, avoid rotation-only assets. No audio recordings as workaround. |
| Seven-bird chorus becomes uncanny or blurred | Recognizability panel at each count, gain/voice bounds, staggered responses and slow rollout of age entitlements. Maintain actual existing birds if rollout pauses. |
| Tick backlog freezes the place | Distributed due times, row locks, lag metrics, bounded catch-up and regional capacity tests. Never move canonical simulation to clients. |
| Edge bootstrap leaks personal state | Private authenticated responses, no shared account caching, token redaction and cache-isolation tests. Prefer missing performance target temporarily to leaking aviaries. |
| Browser autoplay breaks first-call conceit | First frame and captions independent of context, explicit gentle enable control, silence fallback, no delayed burst or recorded alternative. |
| Accessibility becomes a mechanical state list | Shared presentation descriptors, authored prose, slow queue, pose cross-fades, manual user review and release gates for all modes. |
| Scene compresses poorly or focus disappears | Narrow/zoom/night seven-bird fixtures, collision-aware slots, top-bar fade exception for focus and high-contrast accessible caption backing. |
| Future analytics/engagement feature erodes scope | Enforced data permissions and metric allowlists, no warehouse access, authored-copy checks, explicit non-goals in review acceptance criteria. |
| Lifecycle leaves private records behind | Account-key destruction, scoped relations/tombstones, auditable purge jobs, restore tests and deletion drills before public ramp. |

The v1 completion condition is a complete, browser-only aviary with durable private continuity, quiet gestures, individually recognizable procedural birds and equally considered accessible modes. Engineering sign-off requires the acceptance evidence above; no implementation, scoring or later-phase evaluation is part of this planning deliverable.
