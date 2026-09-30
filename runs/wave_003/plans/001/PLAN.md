# Pocket Aviary — v1 implementation plan

This plan is based on all ten required files in `prd/`, including the phase-1 planning guide. It defines an executable v1 without implementing it. Numerical constants below are initial engineering choices to validate in synthetic calibration and design reviews, not claims that tuning has already occurred.

## 1. Product boundaries and decisions

Ship a browser-only, private, single-user aviary per account, two system-selected starter birds, a coherent six-species pool, and a hard seven-bird ceiling. Include naming and renaming, continuous bird-to-bird behavior, owner return-greetings, listen-in, three offers, settle, sparse read-only notebook, email magic links, device sessions, multi-device canonical state, account export/deletion, individually issued read-only visit invitations, and all accessibility modes at launch.

No scores, progression counters, streaks, rewards for attention, species rarity, catalog selection, hunger, death, distress from absence, trait dashboard, editable notebook, bird placement controls, scene customization, scrolling geography, payments, shared aviaries, multiple aviaries, native clients, chat, profiles, follows, feeds, public discovery, rankings, or push notifications. Adoption depends only on elapsed aviary age. Never compute social rankings or population interaction statistics in anticipation of later features.

Resolve underspecified or conflicting details explicitly:

- **Export and hidden traits:** `accounts_sync.md` asks for current personality vectors in export, while `bird_engine.md` forbids the user ever seeing numerical vectors under any surface. Preserve the stricter affective invariant: export names, identities, species, moods, qualitative observable behavior, notebook and settings, but omit numerical personality vectors and hidden derived trait coefficients. Document this exclusion in the export schema and account help. Preserve exact vectors in internal backups only. Do not disguise them as encoded user-visible numbers.
- **Visit notifications:** the social spec allows an off-by-default account toggle, despite the broader prohibition on announcements and aviary email. Implement an explicit opt-in for a quiet textual notice inside the visit-log settings page only, with no badge, toast, modal, email or push. Off by default, absent from onboarding. Ordinary visit log access remains available regardless of the toggle. This is the narrowest useful interpretation compatible with the headline principles.
- **Local time across devices:** store one account IANA timezone, initialized from the first browser and changeable explicitly in settings. All devices and visitors use this canonical timezone; do not automatically let a traveling device change everyone else's aviary.
- **Top-bar inventory:** retain four primary affordances: account/settings, accessibility, notebook, offer. Put settle as a clearly named action in the offer/action popover with a keyboard shortcut; this satisfies top-bar reachability without adding another persistent icon. Bird renaming and age-based adoption live in settings. An eligible adoption can be mentioned quietly within the offer popover, never as a popup or counter.
- **Timing:** begin with 60-second simulation ticks, a five-minute activity window, 15-second presence reporting, 30-second owner snapshot pulls, three-minute per-bird offer cooldown, and 45-second idle narration. Treat these as versioned configuration with acceptance ranges below.
- **Adoption pacing:** age eligibility at 90, 180, 270, 365 and 540 days for birds three through seven. Availability is independent of visits; user accepts and names the offered bird. Permit one pending arrival at a time, with no urgency, countdown, paid acceleration or missed opportunity. Cap enforcement is transactional.

Product prose is lowercase, present-tense, bird-specific naturalist observation. Identity, errors, account, sync and accessibility settings use normal capitalized, matter-of-fact language. Maintain a reviewed copy catalog with examples and forbidden language; avoid general-purpose toast infrastructure in the aviary.

## 2. Architecture and ownership

Use a TypeScript web client, a TypeScript HTTP application, PostgreSQL for canonical state and private simulation data, and independently deployed simulation workers. Serve immutable compressed assets through a CDN. The authenticated HTML/bootstrap endpoint runs near the database through an edge entry point; inline the small authorized snapshot into HTML. Cache public assets globally, never shared-cache personalized HTML or snapshots. Email delivery is an isolated outbox worker for authentication, explicit invitations, verification and requested export only.

A modular backend is sufficient for v1; do not introduce microservices for birds, notebook and weather separately. Modules: identity, owner APIs, simulation, snapshot projection, notebook, visits, account lifecycle, and operational metrics. Workers share the versioned simulation package with the API but only workers possess database permission to update personality columns. Event intake cannot update them. Deployment configuration and database grants enforce that boundary.

The canonical aviary contains persisted bird identities, vectors, moods, perches, slow behavior plans, weather and processed-event cursor. Clients receive only an observable projection: silhouettes, palette tokens, pose/transition descriptions, call descriptors and timing. No raw vectors, trait labels or one-to-one numerical trait aliases in JSON, ARIA, debug screens, exports or error payloads. Observable animation and audio necessarily reflect personality; encode them as bounded presentation instructions rather than a trait model to optimize.

Split behavioral and presentation clocks. Server ticks choose persistent mood, perch plans, weather, greeting intent, reaction plans and seeded call schedules. The client evaluates authorized plans against server time, interpolates motion and synthesizes calls. It can create non-semantic leaf ornaments and pose variation inside the supplied plan; it cannot choose canonical mood, consume events, advance drift, or simulate offline evolution. A call plan can include a contingent response motif so two birds react as a system without a server request per note.

For immediate interactions, the event transaction returns a server-issued presentation acknowledgment (offer position, pending reaction/greeting plan) valid until tick reconciliation. It does not update the vector. The next tick commits consequences using the same event seed and decision, so the animation does not contradict the canonical result. Owner devices get acknowledged changes on their next pull; do not promise simultaneous frame-level playback. Settle is session-local lighting and mixing, with a queued canonical mood-quieting input; one device cannot force another device into a session-ending UI.

## 3. Persistent model and retention

All internal primary and foreign keys are random UUIDs; email is never a partition key or identifier. Use UTC timestamps and an IANA timezone for local-day boundaries. Tables and constraints:

| Record | Principal fields and invariants |
| --- | --- |
| Account | UUID, encrypted verified email, encrypted pending email, timezone, settings, created_at, deletion_requested_at; exactly one aviary |
| Device session | UUID, account_id, hashed secret, issued_at, expires_at, revoked_at, human-readable device label; cookies never contain state |
| Magic-link challenge | UUID, account_id or signup challenge, hashed token, expires_at, consumed_at, purpose; 15-minute lifetime, atomic one-use consumption |
| Aviary | UUID, unique account_id, created_at, revision, tick_index, simulated_through, next_tick_at, event_cursor, engine_version, random seed, weather plan, bounded recent observation summaries |
| Bird | immutable UUID, aviary_id, species_id, editable name, adopted_at, five persisted traits in [0,1], low-pass signal accumulators, mood, mood_since, next_mood_review, perch zone/slot, behavior seed and plan, stable call signature, last_offer_at |
| Interaction event | UUID, aviary_id, monotonically allocated aviary sequence, device_session_id, type, optional bird_id, validated bounded payload, received_at, effective_at; unique event UUID and session sequence |
| Presence interval | event reference, session_id, bounded accepted start/end, listen-in target if any; merged coverage for owner attention, never visitors |
| Notebook entry | UUID, aviary_id, observed_at, text, template_version, semantic fact key; immutable to user, chronological cursor pagination |
| Invitation | UUID, host_account_id, recipient contact reference, hashed redemption token, created_at, unused_expires_at, consumed_at, revoked_at |
| Visitor contact | UUID and encrypted recipient email; used only to send invite and display the authorized host log, no public identity |
| Visit session/log | UUID, invite_id, hashed session token, started_at, last_pull_at, ended_at, approximate duration, expires_at; no simulation presence fields |
| Export job | UUID, account_id, status, created_at, expiry, private object reference, hashed download token |
| Outbox | UUID, account/contact reference, purpose and job reference, retry metadata; no email copied into job payloads |

Email for a signed-in account exists on its encrypted account record only; lookup uses a restricted keyed blind index, not an email-derived service identifier. Pending verification is identity data with prompt cleanup. Invitations to people without accounts require encrypted contact storage: isolate this necessary exception to the host-authorized address book/contact boundary, never copy addresses into simulation tables or logs. Email provider sees the minimum transactional recipient/content, never bird histories or states.

Store recent event data only as long as necessary for consumption, bounded recovery and cooldown/presence validation: initial target 30 days after consumption. Do not remove unconsumed events. Keep state, accumulators and compact bird-observation summaries persistently; vectors are never rebuilt from events. Notebook remains readable indefinitely until account deletion. Retain visit records for host transparency until deletion, with clear settings disclosure. Export object lifetime is 24 hours. Tokens and contact data have explicit expiry cleanup.

## 4. API contracts

All routes use TLS, authorization per resource, strict schema validation, body size limits, secure HttpOnly SameSite cookies, CSRF defense for mutations, request IDs and no sensitive response caching. UUID uniqueness is not authorization. Version payloads as `schema_version: 1` and engine plans separately.

| Endpoint | Contract |
| --- | --- |
| POST /auth/magic-link | Email input; generic accepted response to avoid enumeration; per-email secret-keyed limit initially 5 requests/hour plus coarse IP abuse limit; queue mail |
| POST /auth/redeem | Token exchange, single-use 15-minute transaction; create session, redirect to clean URL; replay returns actionable system error |
| GET /api/aviary/snapshot | Owner projection, revision, server_time, simulated_through, timezone, birds/plans, weather, capabilities, greeting acknowledgment; conditional request by revision plus plan horizon |
| POST /api/aviary/events | Up to 32 events per batch, stable UUID, session sequence, type, bounded timestamps; return accepted IDs, duplicates, rejected reasons, server sequence and presentation acknowledgments |
| GET /api/aviary/events/status | Bounded list of the device's event IDs; processed revision or rejection; useful after an ambiguous network timeout |
| GET /api/notebook?before=cursor | Owner only; 30 immutable observations/page; opaque stable cursor, no stats or editable routes |
| PATCH /api/birds/:id/name | Validated Unicode display name, 1–40 graphemes, plain text; separate metadata revision, If-Match for conflicting rename; never changes identity or traits |
| GET /api/adoption | Return current offered species/name suggestions only when age eligible; no scores or progression counts |
| POST /api/adoption/accept | Idempotent acceptance of signed offer ID, transactional cap check, stable new bird ID and name |
| GET/DELETE /api/account/sessions/:id | List device sessions / revoke; next request rejects revoked credentials |
| PATCH /api/account/settings | Timezone, accessibility/audio/caption settings, optional visit-log notice; optimistic metadata revision |
| POST /api/account/email-change; POST /auth/verify-email-change | Verify new address before atomic switch; current address stays valid until success |
| POST /api/account/export | Owner request; consistent database snapshot, bounded async job, email private download link to current verified address |
| POST /api/account/delete; POST /api/account/recover | Start 30-day soft deletion / restore within window; deleted state blocks normal simulation and visits |
| POST /api/invitations | Host explicitly provides recipient email; one-use token, 30-day unused expiry, mail outbox |
| GET /api/invitations; DELETE /api/invitations/:id | Owner list plus visit log; revoke outstanding/active authorization without success toast |
| POST /visits/redeem | Atomic single-use token exchange for scoped read-only visitor cookie; clean URL redirect |
| GET /api/visit/snapshot | Same authorized scene projection as host, no notebook/account/private metadata or greeting; recheck revocation every pull |
| GET /export/:token | Single-purpose expiring authenticated download capability; no shared cache, no numeric vectors |

Events are presence intervals, presence-end, owner-return, listen-in-start/end, offer with seed/song-fragment/pool type and approved motif ID, settle and undo-settle. Reject unknown types, arbitrary URLs/audio, trait mutations and client mood proposals. Event endpoints authorize the owner session and reject visitor credentials even if the client constructs requests manually. API errors use structured codes plus matter-of-fact text, not animation-only feedback.

Only explicitly named new-account onboarding creates the first aviary and its two birds, atomically. Choose two different starter species from the six-species pool, offer default names, then accept names without catalog UI. Names are text-escaped everywhere, including narration and captions.

## 5. Simulation, drift and continuity

### Scheduling and atomic ticks

A scheduler selects due aviaries from a `next_tick_at` index; workers lease rows using database row locking/skip-locked work selection. Each tick transaction locks its aviary and birds, reads events after the committed cursor in server sequence order, evaluates one minute of behavior, applies additive deltas, appends notebook observations if eligible, updates plans and revision, then commits cursor and next due time atomically. Duplicate queue delivery or crash cannot apply drift twice. Do not allow a stale worker lease to commit over a newer revision.

Allocate event sequence under the same aviary lock so committed order has no invisible sequence holes. Batch event intake is short; the simulation transaction has a bounded event budget and continues remaining work next pass. Never advance a cursor past deferred events. Only a successful committed tick advances `simulated_through`. Backups include exact vector/accumulator/cursor/version state; restoration never regenerates birds.

Ticks run with no clients present. Presence signal naturally decays after accepted intervals end; pre-departure input may continue causing small positive increments for a while. Mood, local daylight and weather advance regardless of owner activity. When the worker fleet falls behind, catch up bounded logical intervals on the server, preserving ordered interactions and day boundaries. For very long outages, use exact analytical decay of low-pass accumulators and piecewise day/weather transitions; do not replay every rendered pose. Validate this fast-forward against minute-step synthetic runs. No lazy client-side catch-up on opening a tab.

### Honest presence and multi-device accounting

Client tracks visibility, `document.hasFocus()`, and timestamp of the latest pointermove or keypress/keydown. Start counting only when all three conditions hold and the session is not settled. Do not substitute scroll, timer callbacks, audio playback or a network heartbeat for activity. Choose a five-minute input recency window initially to support still watching. Accessibility keyboard input counts normally. Newly opened tabs without qualifying activity can greet but cannot accrue drift presence yet.

Use a monotonic clock for durations and map to server time using response offsets. Accumulate qualifying 15-second intervals; close at blur, hide, inactivity deadline, settle or navigation. Send a best-effort terminal event with keepalive/beacon; the server expires absent heartbeats rather than assuming a clean close. Start no interval across a hidden gap or machine suspension. If timers wake late, report only known qualifying coverage, never wall-clock gap duration. Bound server acceptance to 30 seconds per interval and freshness of 60 seconds; very late/offline presence is dropped rather than credited retroactively. Handle forward/backward system-clock changes through monotonic timing.

Server merges overlapping owner coverage from all authenticated devices into a union per aviary: two screens cannot double drift. Listen-in coverage per bird is also interval-unioned and restricted to accepted owner presence. Cap the total listen-in attention credit per time slice to owner attention; simultaneous device targets split the secondary credit. Client reports remain a trust boundary, so reject impossible durations and rate-limit abuse without building invasive surveillance. Retain no pointer positions or key contents.

### Drift function

Persist five traits and five low-pass accumulators. For trait j, compute a normalized daily signal `u_j` from accepted presence minutes plus small eligible listen-in/offer credits. Start with presence at least 80% of total signal weight under typical use; cap all interaction-only credit so repeated clicks cannot dominate. Listen-in primarily supports warmth/vocal frequency, eligible offers support boldness and accepted offers curiosity. Settle has zero trait credit. Audio mute never subtracts or rewards traits: explicit listening is the attention signal, while sound preference remains an accessibility choice.

For each logical step dt, update `s_j = exp(-dt/tau)*s_j + (1-exp(-dt/tau))*u_j`, initially tau = 3 days. Apply `delta_j = k_j * s_j * (1-p_j) * dt_days`, with bounds and a per-day positive limit. Persist `p_j = min(1, p_j + max(0, delta_j))`. Initial traits sample distinct species-centered ranges around 0.2–0.6; choose k values initially around 0.015–0.025/day, then calibrate. Persist deltas atomically against existing values, never replace vectors with a client or old snapshot. Absence makes signal decay, not traits decline. Saturation slows change smoothly; established identities remain distinct through stable signature and species baseline.

Synthetic fixtures: 15–20 minutes qualifying presence daily for 7 and 21 days; irregular visits; 14-day absence; always-background tab; overlapping devices; burst offers; listen-in with sound muted. Target measurable trait change at one week (initial numeric acceptance band 0.005–0.03) and blind human perceptibility around three weeks, while one session remains below the visual threshold. Calibrate using fictional histories only, never production population bird metrics. Review high-starting traits separately to avoid ceilings flattening all birds.

Reduced recent greeting frequency after absence is transient familiarity/expression, not personality damage. Mood and ambient call rates may quiet within ordinary night/weather ranges, but absence cannot induce distress, loss of color, mistrust, hunger or punishment. On return, no days-away text appears.

### Mood, perches, weather and bird relationships

Finalize five moods: wary, content, curious, drowsy, alert. Store mood and timers. Use a weighted transition matrix with minimum dwell times (initially 5–20 minutes), biased by local day phase, recent validated interactions, weather and personality. A daily review gently reweights baselines; it does not reset to neutral at midnight or session open. Rain temporarily reduces expression; wind mildly favors alert/wary. No alarm graphic or sustained distress is associated with wary.

Choose front/middle/back zones through personality/mood weights, reserve collision-free slots, and choose timed travel plans. Keep stable identity through perch changes. Bird calls can elicit another bird's contingent answer after a variable delay; propagate wary mildly with decay to avoid feedback lock-in. Chorus emerges from overlapping calls, not an arrival cue involving every bird. Bound simultaneous voices and transitions so seven birds remain legible.

Sample short rain initially 2–3 times/week and occasional light wind with an independent seeded schedule. Persist weather start/end and intensity curve; weather is not live location weather. Daylight is a continuous timezone-based palette/audio envelope, including DST handling. At night most birds use eyes-closed low poses; one nightjar-like species retains occasional active calls. No dead night scene.

### Greetings and offers

On navigation or hidden-to-visible return, submit a unique owner-return event. Server derives absence from last owner visibility/session intervals, not a client duration claim. One primary bird is chosen by boldness, warmth, mood and recent greeting history. Generate continuous variations in glance angle, duration, approach distance, motif contour and timing from a fresh event seed; do not rotate a small canned list. Quick returns bias toward a glance, longer ones toward gentle reorientation. Subsequent birds may respond after stochastic delays, never in synchronized welcome chorus. Acknowledge within the initial 1–2 seconds; inline bootstrap can include a greeting intent for initial navigation. Duplicate navigation requests reuse event identity and do not create repeated greetings. Visitors never generate these events.

Offer from the top bar creates a server-selected recipient plan weighted by proximity, curiosity and mood. Enforce three-minute cooldown per eligible bird across all devices, independent of offer type. A multi-bird reaction can include only cooldown-eligible birds; a repeat on an ineligible bird gives no drift credit. Curious/content birds investigate, wary birds wait, drowsy birds may ignore. Song fragments are a reviewed small procedural motif library; response may join, pause or answer. Pools produce drinking/bathing/watching poses and bounded reflective visuals. No offer inventory, feeding schedule or penalty for rejection. Give subtle scene feedback and accessible prose, with neutral inline system feedback if every bird is cooling down.

## 6. Synchronization and failure handling

Pull owner snapshots every 30 seconds while visible, immediately on return, network restoration, interaction acknowledgment requiring refresh, and render gaps over two seconds. Use `server_time`, revision and plan sequence to discard older out-of-order responses and align clocks. Plans cover at least 120 seconds, so normal pulls do not interrupt calls or motion. When unchanged state still needs fresh plan timing, return a new projection/horizon even if the persisted revision is unchanged; ETags include projection epoch.

The client keeps a small in-memory queue of at most 32 short-lived pending events with stable IDs and retry backoff. Retry ambiguous writes with the same ID. Never queue days of offline presence in local storage. An offer can show an in-progress placement while awaiting acknowledgment, but cannot invent acceptance. If disconnected, continue rendering the last authorized ambient plan for a bounded horizon, then use quiet resting poses and captions without extrapolating new canonical moods or calls indefinitely. Show a quiet inline system connection state in the top bar, no failure toast. On reconnect, fetch canonical state and interpolate; discard expired interaction attempts with direct inline wording.

Metadata changes use If-Match; simultaneous renames return a conflict with the latest name and a clear retry path. Personality conflicts are structurally impossible because no metadata endpoint touches vector fields. Different-device settle cannot end the other device's qualifying presence; aviary-level union ends when no owner session qualifies. Five-second undo-settle marks the paired event canceled if unconsumed; if already consumed, apply a compensating mood intent on the next tick, never reverse trait deltas. Any click within five seconds resumes the session lighting without accidentally activating an underlying offer/bird action; later deliberate re-engagement resumes normally.

## 7. Scene rendering and interaction surfaces

Use a small inline SVG scene and critical CSS for the first visible bird, then a lightweight SVG renderer driven by requestAnimationFrame for normal motion. Avoid a large 3D engine. Precompute compact species silhouettes, feather overlays and pose anchors; use transforms and opacity rather than layout changes. Server-provided plan progress at current time seeds the first pose mid-action; bootstrap never starts every animation at zero. Normal returns have no entry fly-in or fade-from-static. The true first adoption is the deliberate exception: quiet empty field, then a soft fly-in to initial perch (cross-fade in reduced motion).

Compose sky/background foliage, back/middle/front perches, birds, optional still pool, and limited foreground ornaments. Mood drives preening, scanning, head tilt, fluff and weight shuffle; feather color detail changes only slowly. One bounded ornament pool handles leaves/feathers at slow intervals, without server leaf state. Subtle parallax is slow ambient displacement, never scroll/pointer-follow spectacle. Stop drawing when hidden; restore from a fresh snapshot without replaying missed motion.

Use normalized layout slots and a fit-to-viewport stage. On phones redistribute horizontal spacing and scale down while retaining all seven bird silhouettes; preserve a horizontal scene with letterboxing where needed. No panning, zoom or page scrolling for geography. Notebook/settings panels may scroll independently. Support narrow 320px viewport, short landscape windows and 200% text zoom; controls reflow above the scene. Larger invisible semantic hit targets can overlap bird silhouettes only with deterministic nearest-center selection, never make a bird unreachable.

The four top-bar icons have accessible names and minimum 44px touch targets. Fade after four seconds of pointer stillness only if no focused control or open panel exists; retain enough opacity for copy contrast or hide labels fully while inactive. Pointer/key/touch activity restores it. Keyboard focus always keeps controls visible. No scene labels, tooltips, buttons or mood icons. Call captions and visible keyboard outlines are explicitly accessibility exceptions.

Bird interaction uses a semantic DOM layer aligned with the SVG positions. Clicking/tapping a bird engages listen-in, clicking again ends it, changing bird moves attention, empty-space click ends it. Tab enters a roving bird group, arrows move among birds in stable logical order, focus enables listen-in, Enter toggles it, Escape exits. Moving keyboard focus away ends it. Avoid conflict between focus-on-click and click toggle by tracking a single action source/state machine. No drag-to-place behavior.

## 8. Procedural audio and captions

Define six motif grammars with species-specific pitch regions, envelopes, contour shapes, note gaps, timbre/filter settings and response rules. Each bird gets a stable signature seed whose motif proportions/timbre survive renaming, mood, engine changes and years of drift. Mood changes timing, softness and pauses; personality changes call frequency and bounded contour variation without replacing identity.

A server-issued call descriptor contains bird ID, start offset, motif tree, seed and bounded acoustic parameters. Client expands the grammar into timed note descriptors. The same expansion produces captions from actual note count, contour, trill and pauses, including calls in silence mode; captions are not fixed labels per species. A caption such as “a low trill, paused, low trill again” describes the scheduled call, not hidden mood values.

Use one AudioContext per page, procedural oscillator/filter/envelope synthesis and reusable noise buffers. Schedule a bounded lookahead, cancel stale plans after resume, and never replay missed calls in a burst. Limit active voices, disconnect finished nodes, retain no unbounded descriptor history. Support two-or-more-bird chorus with staggered contours, conservative gain staging, spatial pan from perch, soft limiter and measured peak headroom. Test mono output and phone speakers as well as headphones.

Listen-in ramps over roughly 1.5–2 seconds toward a gently elevated focused gain and lower ambient gains for other birds; ambient gains remain above zero. Disengagement uses the same ramp. Mixer rebalances continuously when focus changes or a bird moves. Listen-in events count qualifying attention duration, not button presses. Settle quiets the local mix over a few seconds. Day/night and rain modulate calls without permanently changing vocal traits.

Browser autoplay permissions can prevent audible calls at navigation. Attempt permitted context resume, but never claim guaranteed autoplay or use a recorded fallback. While context is blocked, unavailable or fails, show the same living scene in graceful silence with captions enabled by default. A deliberate sound control in accessibility settings or the first appropriate user gesture can resume WebAudio. User mute remains respected across devices through settings and never harms drift. Stop/suspend audio when hidden and rebuild schedule on return. Distinguish intentional mute from capability failure while keeping captions usable in either case.

## 9. Notebook and accessible experience

Notebook generation is deterministic, private observation selection from canonical facts and a bounded history: first greeter change relative to recent local days, an unusual perch preference, a distinctive quiet preening stretch, or a weather response. Use curated phrase grammars with bird names, timing and contextual detail rather than an LLM service or generic event logs. Maintain semantic deduplication keys. Initially allow one ordinary observation every 2–4 days and at most one additional noteworthy observation/day. Review with synthetic long-lived aviaries to ensure active users do not receive a feed. Never record user visit streaks, trait numbers or “session started.” Cursor-paginated notebook remains indefinitely available and read-only; virtualization releases offscreen resources without preventing screen-reader browsing.

Narration uses the same scene projection and reviewed naturalist grammar. One polite live region emits an evolving paragraph about birds, positions, calls and light approximately every 45 seconds at idle. Compare semantic meaning and suppress unchanged paragraphs. Owner greetings, acknowledged offers and settle get a prompt priority observation, coalesced into the queue without interrupting navigation or reading the notebook. Do not use assertive announcements for ordinary bird motion. The live region is not a list of raw changes or numeric traits. Provide a visible transcript and narration pause control in accessibility settings. Keep system errors in an independent appropriately announced matter-of-fact region.

Reduced motion combines OS preference with an explicit persisted choice; system preference applies immediately before rendering. Replace pose micro-motion with slow still-pose cross-fades (initially 3–6 seconds), flights with perch cross-fades, remove falling leaves and parallax, and slow palette transitions. Preserve procedural calls, captions, moods, drift and notebook. Do not ship a static frozen fallback. Captions appear beside each bird with contrast-backed text, remain long enough to read, avoid collisions via bounded layout, and fade slowly; provide transcript access when seven calls would overcrowd the scene. Captions are not all duplicated into the narration live region.

Use real buttons, labeled dialogs, focus containment in open popovers, Escape dismissal and focus restoration. Offer choices and approved song motifs are fully keyboard usable. Implement a documented shortcut (Alt+O, with configurable alternative if needed) and always preserve a visible button route. Focus outline passes contrast against morning, evening, rain and night backgrounds. All text meets WCAG AA: at least 4.5:1 normal text, 3:1 large text; interactive visual indicators at least 3:1. Avoid color-only meaning. Consult accessibility users during design and test VoiceOver/Safari and NVDA/Firefox plus keyboard-only and touch readers; automated audits alone do not establish an alive accessible aviary.

## 10. Visits, account security and lifecycle

Visitor links grant only one bounded read-only session, initially up to two hours. Unused links expire at 30 days; consumed tokens never work again, even if cookie is lost. Reinviting is explicit. Token-bearing pages contain no third-party scripts, use no-referrer, and redirect to clean URLs immediately after redemption. Rate-limit invite creation to prevent email abuse; do not pre-populate visitor lists or onboarding share prompts.

Visitors pull every 15 seconds while visible, immediately on return. Every pull checks invite/session/deletion status; no authorization cache can bypass revocation. Revoked visitor loses access on next pull; stop audio and discard scene/private state, then show “This visit is no longer available.” Offline visit snapshots expire after the polling grace period (initially 30 seconds) and cannot provide indefinite revoked access. Previously seen content cannot be remotely erased from a person's memory; minimize browser persistence. Never issue a visitor greeting, call attention event, presence ping, listen-in action, offer, settle or owner notebook access. Local audio/caption accessibility controls affect only the visitor device. Visitor day/night uses host timezone and identical projected birds/weather; no embellished presentation.

Track approximate visit duration from successful pulls, not simulation presence; close on timeout. Settings log shows recipient email, dates, duration and outstanding invitations without badges. Revocation removes active/outstanding authorization while retaining the historical transparency record labeled ended/revoked; this resolves the social spec's “absence” confirmation without silently destroying who saw the aviary.

Magic-link and visitor redemption are atomic and token hashes are stored. Session list/revocation, email verification, authorization and CSRF tests are release gates. Recheck revoked session state on every mutation/pull. Avoid leaking email in request URLs, logs or error reports. Strip sensitive query strings and reject logging of request bodies by default.

On soft deletion, immediately disable regular aviary access, tick eligibility, visitors and outstanding links; permit only recovery and relevant account functions during 30 days. Recovery restores exact stored bird state and resumes server-time catch-up without inventing absence presence. At hard deletion, cascade birds/vectors/events/notebook/sessions/invites/contact references/exports/outbox and account-associated operational records. Remove encrypted account keys and private object files. Backups use account-scoped key erasure and deletion tombstones honored during any restore so erased data cannot reappear. Irreversibly anonymous aggregates have no account link to remove. Test deletion/recovery/hard-delete with a controllable clock and restore rehearsal.

## 11. Budgets, observability and verification

Treat thresholds as release gates, not aspirations:

| Budget | Implementation and evidence |
| --- | --- |
| Initial JS <2MB gzip | Target <250KB for aviary/bootstrap/audio core; dynamic-load account, notebook, accessibility and invite panels; CI totals all eagerly loaded chunks |
| First bird <500ms on mid-tier mobile/4G | Inline critical scene and compact snapshot with HTML, avoid serial state-fetch/font/asset gates; mark first actual bird paint, not background paint; test warm and cold navigation with explicit device/network profile |
| Snapshot kilobytes | Target 5–20KB raw for seven birds, excluding notebook/history; payload size gate and no hidden vectors |
| Greeting in 1–2 seconds | Measure actual bird notice start after navigation/return; synthetic tests include blocked audio and slow API |
| 60fps idle on five-year-old laptop | Target main-thread animation work <4ms/frame, frame p95 <16.7ms on reference hardware across a 30-minute seven-bird session |
| No memory growth over 30 minutes | After warmup and forced-GC laboratory samples, heap/DOM/audio-node/worker counts plateau; repeated call and panel cycles retain no increasing slope, investigate any sustained increase |
| Tick latency p99 <=5 seconds | Instrument queue delay and computation/commit latency separately; alarm if computation/commit p99 exceeds 5 seconds and also alert on missed minute deadlines |

The 2MB ceiling alone does not achieve 500ms; critical-path bytes and TTFB require separate budgets. Start with TTFB target <=200ms in common geographies and critical HTML/SVG/snapshot <=35KB compressed. If remote geography misses first-bird budget, improve regional routing/snapshot projection and critical payload rather than fake cached birds from another state. Quiet sky with faint motion is only the exceptional slow-load field, never a spinner. Define supported benchmark profiles in CI and run physical-device tests before launch.

Measure synthetic navigation, first bird, frame distribution, heap retention, audio startup/failure, snapshot size and tick deadlines. RUM contains only aggregate load/frame timings, errors, request counts and anonymous duration histograms. Bucket and aggregate client measurements before shipping without account/bird IDs, state, event type histories or names. Operational diagnostic logs may use account UUID for narrowly scoped errors with short retention, but cannot include interaction payloads or vectors. The telemetry service role has no simulation-table read access; no analytics warehouse or model-training export has access. Suppress sensitive URL/query/body data in error tooling. Do not instrument retention, streaks, offer popularity, per-bird growth, visit rankings or production average drift.

Verification suites:

1. Pure engine property tests: identity immutability, nonnegative deltas, exact state persistence, no absence penalty, deterministic seeded plans, cooldown enforcement, bounded moods/weather, cap seven, local midnight/DST and fast-forward equivalence.
2. Presence matrix tests for every combination of visible/focus/recent activity; inactivity deadline, quiet five-minute watching, blurred visible windows, hidden tabs, suspension, settle/close equivalence, lost terminal ping and duplicate/overlapping devices.
3. Transaction/integration tests: duplicate event delivery, worker crash before/after commit, competing workers, event intake racing cursor advancement, ambiguous retries, out-of-order pulls and migrations. Assert no lost or double-applied drift.
4. Audio/visual review: six-species distinguishability, same bird across moods and after drift, no exact call repetition, chorus headroom/mono behavior, greeting variety and staggering, no cold-start animation. Run blind synthetic 1/7/21-day perception reviews without asking production users' histories into telemetry.
5. Accessibility checks: keyboard paths, focus against all palette states, live-region cadence/priority/dedup, caption-to-motif correspondence, reduced-motion still-pose behavior and user evaluations. Snapshot/ARIA/export contract tests forbid traits and stat language.
6. Security/lifecycle tests: cross-account IDs, visitor mutation attempts, token replay/expiry, revocation timing including offline, email change, device revocation, export scoping and hard deletion/restoration.
7. Performance soak: two and seven birds, rapid focus changes, notebook scrolling, repeated visibility changes, multiple panels, fallback audio and 30-minute sessions on all supported engines. Fail on unbounded node/queue counts, not only average heap.

Support the last two major Chrome, Safari, Firefox and Edge versions. Provide a clear unsupported-browser system surface beyond that range; do not add legacy compatibility bundles. WebAudio failure within supported browsers still gets silence/captions.

## 12. Delivery sequence and rollout

**Milestone 1 — contracts and experience foundation.** Finalize schema/projection, hidden-vector boundary, copy catalog, initial palette/contrast tokens and two species' silhouette/call prototypes. Produce normal/reduced-motion first-frame prototypes and narrated/captioned equivalents. Exit: reviewers experience specificity and aliveness without announcements; critical render path is measured on target hardware. Establish privacy-safe operational metrics from day one.

**Milestone 2 — canonical persistence and identity.** Implement auth/session lifecycle, atomic starter adoption, server ticks, durable vectors, ordered event log, correct presence union, account timezone, snapshots and crash recovery. Build synthetic calibration harness. Exit: two devices share one canonical state; seven/21-day synthetic trajectories meet targets and 14-day absence never decreases traits; restore preserves identities/vectors.

**Milestone 3 — complete two-bird experience.** Integrate greeting generation, perch/mood plans, bird replies, procedural audio/mixer, offers/cooldowns, settle/undo, notebook and all keyboard/narration/caption/reduced-motion settings. Exit: no loading/greeting toast, no canned calls, independent accessible user sessions pass, 30-minute budgets hold. This milestone includes accessibility rather than scheduling it later.

**Milestone 4 — account lifecycle and quiet visits.** Add email change, session revocation, restricted exports, soft/hard deletion and restore safeguards, host settings log and invitations. Exit: visitors cannot influence simulation; revocation and expiry tests pass; deletion purge is proven; private data cannot enter metrics.

**Milestone 5 — full species/count envelope.** Complete six coherent species and age-based adoption to seven. Test synthetic aged aviaries with 3, 5 and 7 birds; reserve perch slots and stabilize caption/mix density. Exit: recognizability remains satisfactory at seven and budgets hold. Do not expose seven until this gate passes; maintain server hard cap throughout.

**Launch ramp.** Use internal synthetic accounts first, then an explicitly opted-in closed beta, then small general cohorts (e.g. 1%, 10%, 50%, 100%) gated on operational health and designed-surface reviews. Start real aviaries with two birds; age naturally governs later adoption. During rollout enable higher-count adoption only after 3/5/7 reference tests, and never remove already adopted birds if a gate regresses. No launch progress indicators or notifications in user aviaries. No production interaction aggregation to select winners or tune drift.

Version engine coefficients and motif grammars; migrate persisted data with reversible schema steps and preserved bird signature. Keep previous render-plan decoder during rolling deployment. Rollback means restore compatible code/config, never restore older vector snapshots over newer drift. Tick pause during incidents retains events/cursors and resumes catch-up. Audio failure rolls back synthesis or switches to silence/captions; never substitutes recordings. Invite disablement prevents new grants and preserves explicit revocation access. Release verification includes backup recovery before public ramp.

## 13. Principal risks and responses

- **Drift too fast/slow or saturation convergence:** bounded positive deltas, presence-dominant weights, persisted signal decay, synthetic multiweek histories and blind review. Change coefficients prospectively; never renormalize/reset existing birds or mine production histories for population calibration.
- **Lost continuity or duplicate drift:** worker transactions, unique event IDs, cursor atomicity, restricted writer roles and restore drills. Block any release that can overwrite traits with old snapshots.
- **Presence inflation and battery waste:** conjunction tests, short interval bounds, union across devices, hidden render/audio suspension and suspension-gap tests. Do not convert “open tab” into attention for convenience.
- **Audio uncanny or indistinct:** six signature grammars, stable bird signature, bounded continuous variation, chorus listening trials and mono/phone checks. If seven is not recognizable, delay higher-count rollout while improving synthesis; keep seven as ceiling.
- **First frame feels started rather than ongoing:** inline authorized snapshot, time-offset pose evaluation, small critical payload; test cold navigation. Respect first-adoption fly-in as the sole empty-scene exception.
- **Accessibility regressions:** separate designed reduced-motion renderer, shared call-caption descriptors, narration pacing tests and hands-on users in every milestone. No launch waiver that postpones a mode.
- **Privacy creep:** database permission separation, strict telemetry schemas, no body/state logging, contact isolation and automatic contract tests. Product health comes from technical reliability and designed-surface review, not engagement dashboards.
- **Visit misuse or revocation delay:** token scoping, expiry/replay protection, every-pull authorization, short visitor polling and no offline persistence. Explicitly reject visitor events at the server.
- **Operational tick cost:** partition due-work by UUID, monitor queue age, bounded event batches and deterministic server catch-up. Scale workers/database capacity before minute deadlines slip; do not move simulation to clients to save cost.
- **Requirements ambiguity:** retain the documented export, notification, timezone and top-bar interpretations in engineering acceptance criteria. They preserve the strongest product constraints and can be changed deliberately through specification review without blocking implementation.

V1 is complete when all required surfaces work for owner and visitor roles, the canonical simulation survives absence and device changes without losing identity, accessible experiences retain the product's calm specificity, and performance/privacy gates pass. Implementation and evaluation are outside this phase-1 deliverable.
