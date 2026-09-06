# Pocket Aviary — v1 implementation plan

Run 001, wave_003. Deliverable: an executable implementation plan; no product implementation or evaluation is part of this run.

## 1. Product boundary and binding decisions

The implementation must preserve a private aviary whose birds have durable identities and continue changing on the server while nobody watches. Its main interaction is qualified, idle attention. A visit begins with birds already doing something, followed by one bird noticing the returning owner. Engineering success requires both correct persistent state and convincing, restrained expression of that state.

Planning inputs are prd/1-START_HERE.md and all nine files it lists: product_brief.md, concepts.md, bird_engine.md, interactions.md, aviary_layout.md, accounts_sync.md, social_optional.md, accessibility_perf.md, and non_goals.md. Concepts supplies the authoritative vocabulary. No separate design-system or rendering document was supplied; the concrete defaults below fill those gaps. Numerical defaults are implementation starting points, with explicit calibration and release gates, rather than additional product promises.

### 1.1 Included in the complete v1

- Modern browser application, one owner and one canonical aviary per activated account; email magic links and independently revocable browser sessions.
- Two system-assigned starter birds, user naming and renaming, approximately six authored species, and age-based opportunities to adopt further birds up to an enforced maximum of seven.
- Persisted five-dimensional personality, daily-scale mood, bird-to-bird behavior, procedural calls, local-time lighting, rare weather, and continuous expressive motion.
- Return-greeting, listen-in, three offers, settle with a five-second undo, and precise presence accounting.
- Sparse, automatically written, read-only field notebook with indefinitely accessible history.
- Owner multi-device access, verified email changes, account export, recoverable deletion followed by hard deletion, and a plain-language privacy link.
- Individually issued, one-time email visit invitations; read-only visits; revocation; host-only visit log; the explicitly optional visit-notification setting described below.
- Screen-reader prose, call captions, complete keyboard operation, reduced-motion rendering, WCAG AA user-copy contrast, and audio-unavailable behavior from the first release.
- Synthetic performance coverage, aggregate operational telemetry, bounded runtime resources, and operational recovery procedures.

### 1.2 Excluded

No native clients, payments, password sign-in, SSO, shared ownership, multiple aviaries per account, customizable scenes, scene navigation, bird placement controls, species catalog, rarity, or paid adoption. No deaths, hunger, distress caused by absence, feeding schedules, decaying happiness, or negative personality drift. No numeric trait display, progress dashboard, adoption counter, streak, visit-frequency calendar, badges, achievements, levels, ranks, quests, rewards for activity, or engagement-driven unlocks.

No profiles, follows, public discovery, public feed, rankings, comments, chat, avatars, co-presence, mutual-visit mechanics, automatic invitations, or special presentation for visitors. No aviary reminders, browser push, marketing mail, welcome banners, celebration UI, or notebook-update badges. No recorded-call path, including fallbacks. No production interaction-data warehouse, population drift dashboard, behavioral recommendations, or model training on user-bird relationships.

### 1.3 Decisions where the PRDs need interpretation

| Issue | Implementation decision |
| --- | --- |
| Numeric personality prohibition versus the export explicitly containing current vectors | Normal HTML, client snapshots, ARIA, settings, support views, and telemetry never expose raw vectors. Honor the specifically required JSON export by including the owner's current vectors only in that requested, protected download. This is a narrow portability exception to the otherwise absolute display prohibition; the two statements cannot both be satisfied literally. There is no in-app export preview or trait visualization. |
| Four top-bar icons versus a top-bar settle action | Keep exactly account/settings, accessibility, notebook, and offer icons. The offer popover contains the three offers and a separately grouped settle action. Settle therefore remains reachable from the top bar without adding a fifth permanent icon. |
| Keyboard focus starts listen-in versus Enter starts listen-in | Keyboard focus on a bird begins listen-in. Enter explicitly engages or resumes it and is idempotent while already active. Escape disengages while retaining keyboard focus. Leaving the bird or moving to another bird changes the mix as specified. |
| No notifications versus an optional visit-notification toggle | Default is entirely silent. The optional toggle enables live, matter-of-fact visit notices only inside the account/settings visit-log surface while it is open. It does not enable email, OS/browser push, top-bar badges, or scene overlays. The PRD leaves the channel unspecified; this bounded in-product implementation preserves its explicit opt-in without adding a re-engagement channel. |
| Local day/night versus a single state across devices | Persist one IANA aviary timezone, initially detected from the owner's browser. All devices and visitors use that timezone. Travel does not silently change it; account settings can update it. An update affects future lighting and mood calculations together on the server. |
| Greeting in one to two seconds versus a roughly minute-long tick | Server command handlers authorize short-lived reaction programs immediately from current canonical state. Only the tick changes persistent personality and mood. Acknowledged programs are shared in snapshots and consumed once by the tick; clients never decide outcomes. |
| Scene already in motion versus starter fly-in | Ordinary navigation and returns always show current motion. Only the real, first adoption transition may show the empty quiet field followed by a soft first fly-in. Repeated navigation cannot replay onboarding. |
| Nearly transparent top bar versus contrast | Fade its background and decorative layer almost away; keep any still-visible actionable glyph above its tested contrast floor. Never fade focused controls or user-copy text below AA contrast. |
| Settle across devices | The initiating view gets a temporary evening-light/mix override and stops its presence immediately. The server processes a small shared mood-quieting influence. Other owner views retain their own presentation and presence; canonical bird mood remains shared. Listen-in, mute, and accessibility presentation are similarly local choices over shared state. |
| Muting mentioned as behavior, but no specified mute-to-drift function | Muting changes presentation and can enable captions. It never decreases a trait, counts as negative attention, or changes the bird's recognizable voice. Bird-focused attention can still count while calls are captioned. Do not invent a reward for leaving audio enabled. |
| Unspecified age and interaction windows | Start with a five-minute activity window, 15-second presence reports, three-minute per-bird offer cooldown, 60-second ticks, and adoption ages of 90, 180, 270, 365, and 540 elapsed days for birds three through seven. Validate these constants with synthetic scenarios and review; do not infer them from population interaction logs. |

These decisions are the implementation contract for v1. Changes require updating the affected behavior, API, accessibility, and test contracts together, rather than letting individual clients choose different interpretations.

## 2. Architecture and ownership

### 2.1 Service shape

Use a TypeScript web stack with a small DOM/SVG client, an HTTP application service, a dedicated simulation worker pool, and PostgreSQL as canonical storage. A minimal Preact shell is sufficient for account dialogs and top-bar controls; the animation loop does not run through component reconciliation. Avoid a 3D engine, general game framework, or recorded-media pipeline.

Deploy static assets through a CDN. An authenticated edge HTML handler obtains an authorized, current presentation snapshot and embeds critical SVG, CSS, the snapshot, and a tiny motion bootstrap. Personalized HTML and responses are private and never enter shared public caches. The edge can cache immutable revision-addressed blobs, but must resolve the current authorized head before serving them; an asynchronously stale edge replica is not a source of canonical truth.

Start with one primary database region and worker deployment, close to the first serving region. Scale worker partitions and read delivery before introducing another write region. There is never more than one logical writer for an aviary at a time. A durable queue can wake workers, but PostgreSQL due times, locks, and committed versions determine whether work actually occurs.

Separate processes and permissions:

- **Web/API:** authentication, authorization, event validation, short-lived reaction authorization, state projections, settings, invitations, exports, deletion requests. It cannot update personality columns.
- **Simulation workers:** due ticks, event consumption, persisted personality deltas, moods, weather, behavior schedules, notebook observations, and adoption eligibility. They hold the only personality-update database role.
- **Transactional mail/job workers:** explicitly requested magic links, invitations, email verification, and export delivery. They receive account/job UUIDs, obtain an authorized destination only at send time, and receive no bird interaction history.
- **Lifecycle workers:** expiry, export cleanup, hard deletion, and backup/key erasure coordination.
- **Operational metrics collector:** accepts a fixed allowlist of timings, counts, and errors. It has no database credentials or network route to simulation records.

### 2.2 Module boundaries for implementation

| Module | Owns | Must not own |
| --- | --- | --- |
| Domain and persistence | Bird identity, schemas, immutable event definitions, revisions, transactional invariants | Browser session behavior |
| Simulation core | Pure tick transition, nonnegative drift, mood hazards, social coupling, deterministic behavior planning | HTTP, DOM, WebAudio, analytics |
| Interaction service | Authorization, idempotency, ordered appends, cooldown reservations, immediate reaction envelopes | Direct personality mutation |
| Presentation projection | Export-safe versus client-safe DTO separation, action timelines, geometry/style parameters, observation facts | Writing bird identity or rebuilding traits |
| Shared presentation runtime | Sampling an authorized action at a timestamp; interpreting call descriptors and prose facts | Mood decisions, drift, adoption, authoritative random outcomes |
| Scene renderer | SVG composition, bounded micro-motion, ornaments, captions, focus outline, interpolation | Canonical time advancement |
| Audio renderer | Procedural synthesis and local mixing | Picking which bird calls or deciding an offer outcome |
| Accessible surface | Narrative composition, semantic controls, event queue, reduced-motion preference application | Reading hidden traits |
| Account/social surfaces | System voice, device management, consent, invitations, portability and deletion | Engagement UI or scene customization |

The server's presentation projection can derive numeric drawing or synthesis parameters, such as a color or pitch curve. Those are not raw personality coordinates. Do not ship a hidden client-side personality object merely because the visible UI does not display it.

### 2.3 Authoritative versus local state

Authoritative state includes bird UUIDs, names, species/version, voice identity, personality, filter accumulators, mood and timers, perch/action schedules, ambient weather, timezone, interaction decisions, cooldowns, notebook, and adoption history.

Local state includes focused bird, listen-in mix, device audio availability, mute, open panel, viewport geometry, render ornaments, interpolation clock, and an initiating view's settle overlay. Local changes cannot overwrite or fork a bird. Account preferences provide defaults across devices; an incapable browser can apply stricter local fallback without rewriting everyone else's preference.

The client receives versioned snapshots plus explicit event acknowledgments. There is no client simulation, client-to-client protocol, absolute personality update, or state merge. A persistent browser cache is unnecessary for v1; keep live snapshots in memory and clear them on logout, account change, or visit termination.

## 3. Data model and transactional invariants

Use UUIDs for accounts, aviaries, birds, sessions, invitations, jobs, and domain events. Store UTC timestamps; convert to the aviary's IANA timezone only for behavior and prose. Names are display values, never identifiers.

### 3.1 Records

| Record | Essential fields and rules |
| --- | --- |
| Account | id; encrypted email; restricted keyed email-lookup digest; status; created_at; verified_at; pending encrypted new email and verification reference; deletion_requested_at; hard_delete_at. Email appears as stored plaintext nowhere. |
| Account settings | account_id; timezone; default audio setting; call captions; reduced motion as system/on; narration preferences; visit_notifications default false; settings_revision. |
| Auth challenge | id; account_id; token_hash; purpose; issued_at; expires_at; consumed_at; intended session action. Magic-link lifetime is 15 minutes. |
| Device session | id; account_id; token_hash; issued_at; last_used_at; idle/absolute expiry; revoked_at; user-readable browser/device label. Labels are bounded and sanitized. |
| Aviary | id; unique account_id; created_at; state_revision; last_tick_at; next_tick_at; engine_version; event_head; consumed_event_seq; weather plan; behavior seed/counter; notebook gating state. |
| Bird identity | id; aviary_id; adopted_at; name; species_id; immutable voice_seed/signature version; visual identity seed; archived migration provenance if needed. There is no reset or replace operation. |
| Bird state | bird_id; five persisted normalized traits; five persisted low-pass accumulators; mood; mood_since; next baseline reconsideration; transient influences; perch zone/anchor; current action program; cooldown_until; version. |
| Behavior program | id; aviary_id; revision; effective interval; per-bird pose/perch segments, social responses, call descriptors, weather influences, continuation cursor. Programs are server-authored, bounded, and timestamped. |
| Interaction event | id; aviary_id; per-aviary seq; actor owner account/session/view; event type; accepted_at; bounded client timing evidence; validated payload; authorized reaction outcome/program if relevant; schema version. Immutable after acceptance. |
| Presence coverage | aviary_id; retained, unioned server-time intervals; finalization watermark; bounded per-view receipt sequence/anchor. Contains no pointer coordinates or key contents. |
| View session | id; device_session_id; current foreground epoch; monotonic clock anchor; last heartbeat; terminal markers; current attention target; settle instance. Operational/session state, not a user-facing activity history. |
| Notebook entry | id; aviary_id; observed_at; created_at; locale; authored prose; bounded supporting fact references; bird IDs; names as observed; rule/content version. Owner-readable, append-only. |
| Adoption opportunity | id; aviary_id; ordinal; eligible_at; offered species/seed; accepted_at or deferred state. Eligibility uses aviary age alone; deferral does not reroll species. |
| Visit invitation | id; host_account_id; recipient_account_id; token_hash; created_at; expires_at; consumed_at; revoked_at; grant reference. It never grants owner scope. |
| Visit grant/session | id; invitation_id; token_hash; created_at; expires_at; revoked_at; last authorized pull. V1 active grants last at most 24 hours and require a new invite afterward. |
| Visit log | host_account_id; invitation_id; recipient_account_id; started_at; last_authorized_pull_at; ended_at. Approximate duration is derived from successful pulls, never bird presence. |
| Export job | id; owner_account_id; requested_at; snapshot_revision; object reference; token_hash; expiry; completion status. No link or secrets in logs. |
| Transactional outbox | UUID job type/reference, attempt metadata, completion marker. Carries references, not decrypted email or bird payloads. |

Store ordinary account email only on its encrypted account record. The restricted keyed digest exists there solely for lookup/uniqueness; it is never a service identifier, metric dimension, shard key, or log field. Invitees without accounts get an email-only identity record with a synthetic account UUID; it creates no aviary or owner permission. Activating an owner account atomically creates its one aviary. This lets invitations and host visit logs reference a UUID without duplicating email in invitation tables. Remove orphan invitation-only identities after their last required invitation/log retention ends; retain independent owner accounts.

Current and pending new email are different addresses temporarily stored on the account record for verification. A successful verification atomically checks uniqueness, replaces the address, removes the pending value, and invalidates the verification challenge. Until that commit, the old address remains the valid destination and sign-in address.

### 3.2 Enforced invariants

1. One activated owner account has exactly one aviary. Creation retries cannot create a second aviary or more starter birds.
2. Each aviary has two starter birds after onboarding and never more than seven. Adoption locks the aviary and checks count and eligibility in the same transaction.
3. Bird UUID, adopted_at, and voice identity survive renaming, code deployment, and schema migration. No migration draws a new personality or seed.
4. Each trait remains in [0,1]. A tick writes an additive, server-computed delta greater than or equal to zero; no API accepts an absolute client vector.
5. The committed vector is the source of truth. Logs and summaries cannot reconstruct, reseed, or replace it during navigation or normal recovery.
6. A tick commits its consumed-event cursor, vector changes, moods, program, notebook additions, and new revision atomically. An event cannot be credited twice or disappear between revisions.
7. Event sequence assignment is serialized with commits per aviary. Do not use a global sequence high-water mark that can skip a lower sequence from an uncommitted concurrent transaction.
8. Presence intervals overlap by union across owner devices, not by addition. Visitor view time never enters this ledger.
9. Invitation consumption, expiry, revocation, and grant creation are atomic. Check current authorization before every visit response, including a conditional 304 response.
10. Notebook entries have no user update/delete endpoints. Account deletion is the explicit whole-account exception to indefinite notebook retention.

### 3.3 Retention and durability

Retain raw simulation interaction events for 14 days after successful consumption, provided presence union/deduplication watermarks and required bounded observation facts have been folded durably. Retain unconsumed events until processed or the account is deleted. API idempotency receipts can expire after 48 hours; requests older than their admissible timing window cannot be resurrected as new interactions. Retain a bounded seven-day factual window for notebook comparisons such as which bird greeted first, not an owner visitation calendar.

Keep presence interval coverage for 24 hours, comfortably longer than the admitted report delay. Advance a rejection watermark before pruning so an old replay cannot gain credit once its interval has been removed. Store current vectors and filter state indefinitely while the account exists. Keep notebook entries indefinitely, accessible by indexed cursor pagination.

Use durable commits, encrypted backups and point-in-time recovery, and tested failover that fences the old writer. For acknowledged domain-event and vector commits, target zero data loss within the normal database failover domain. Do not acknowledge on a disposable replica. Disaster recovery must restore persisted state; it must never regenerate birds from historical events. Record and surface a genuine recovery incident as a system error if intact state cannot be recovered.

## 4. API contract

All owner routes require an active, authorized device session and same-origin mutation protection. JSON uses explicit schemas; reject unknown mutation fields rather than accepting a client-supplied mood, trait, perch, or account owner. Return stable machine error codes with matter-of-fact human copy on system surfaces.

### 4.1 Authentication and account routes

| Method and route | Contract |
| --- | --- |
| POST /api/auth/magic-links | Email input; generic response; issue a 15-minute single-use challenge. Initial limit: five requests per address per 15 minutes, with separate abuse protection. Resolve the address to its UUID before internal rate-limit accounting. |
| GET /auth/continue | Minimal token landing page; no state mutation on email-scanner GET requests. Strip secrets from browser-visible URL after capture; no third-party assets. |
| POST /api/auth/consume | Consume token atomically and issue this browser's session cookie. A small Continue action prevents scanners consuming a sign-in. Expired, replayed, or invalid tokens receive the same clear recovery route. |
| GET /api/account | System-facing settings and account status; never bird traits or behavioral counters. |
| GET /api/account/sessions | List current device sessions and approximate last use for security. |
| DELETE /api/account/sessions/:id | Revoke that owner's session; authorization is checked on every subsequent request. |
| POST /api/account/email-change | Request verification of the new address; do not switch the old address yet. |
| POST /api/account/email-change/verify | Consume verification challenge and commit the unique new address. |
| PATCH /api/account/settings | Allowlisted settings with expected settings_revision; a stale revision returns 409 and the current settings, not a silent overwrite. |
| POST /api/account/export | Idempotently enqueue a consistent JSON export and mail its download link to the verified address. |
| GET /api/account/export/:id/download | Owner-authorized, expiring, non-cacheable download; handle missing, expired, or deleted-account jobs plainly. |
| POST /api/account/deletion | Mark deletion immediately and return the 30-day recovery deadline. |
| POST /api/account/deletion/recover | Within the grace period, restore the same account/aviary/birds after the explicit “I changed my mind” action. |

Session-cookie defaults are Secure, HttpOnly, SameSite=Lax, host-bound; rotate sessions periodically, with 30-day idle and 90-day absolute limits. Keep revocation authoritative rather than relying on a long-lived JWT alone. Pending-deletion accounts may sign in to the recovery/system surface but may not submit bird interactions or use visits.

### 4.2 Owner state and interaction routes

| Method and route | Contract |
| --- | --- |
| GET /api/aviary/snapshot | Authorized, mutually consistent presentation snapshot. Accept ETag; response includes server_now and revision information even when a full payload is unnecessary. |
| POST /api/aviary/events | A bounded batch, maximum 16 events and 16 KB; per-event idempotency key, view ID and sequence. Returns ordered receipts, current revision, accepted server times, and immediate reaction envelopes where applicable. |
| GET /api/aviary/events/:event_id/receipt | Resolve an uncertain write result after connection loss using the same event ID. Never resend an uncertain offer with a fresh ID automatically. |
| GET /api/aviary/notebook?before=cursor&limit=30 | Owner-only, newest-first cursor page, with unbounded backwards navigation and no pagination count that resembles a score. |
| GET /api/aviary/birds | Owner bird-settings data: identity, species description and name, plus any age-eligible adoption opportunity; no traits or rarity. |
| PATCH /api/aviary/birds/:id/name | Name and expected name revision. Renaming changes neither identity nor behavior. |
| POST /api/aviary/adoptions | Accept a specific offered opportunity and name. Validate age, count and idempotency atomically; create one new stable bird. |

Event types are view_return, view_leave, presence_interval, listen_start, listen_end, offer, settle, settle_undo, and reengage. view_return/view_leave support greetings and view lifetime; they do not by themselves credit presence. Offers identify seed, song_fragment with a library motif ID, or still_pool. A top-bar popover can choose a recipient, defaulting to the currently listened-to bird or a server-chosen available bird. Clicking a bird in the scene itself never opens offers.

A presentation snapshot has this conceptual shape:

    schema_version, state_revision, event_head, server_now, effective_at, valid_until
    aviary: timezone, lighting anchors, weather interval, scene seed
    birds[]: id, name, species silhouette/version, derived visual style,
             current mood expression, perch zone/anchor, active pose segments,
             voice signature descriptor, scheduled call instances
    reactions[]: immutable event_id/program_id, target, start/end, outcome, segments
    observation_facts: prose-ready descriptions of the same state
    owner_capabilities: permitted actions, offer availability, adoption/settings revisions

The internal mood enum can be mapped to an expression token for renderers; the UI must not print “mood: content” or a state list. Never serialize the five traits, drift accumulators, private activity intervals, or all past interactions into this DTO. Notebook pagination and account panels load separately.

Receipts distinguish accepted, duplicate, unavailable, unauthorized, and rejected_timing. A cooldown result produces a restrained inline offer-panel state such as “the seed can wait a little”; it is not a failure toast, countdown, or loss. Authentication and connectivity failures use direct system language.

### 4.3 Visit routes and permissions

| Method and route | Contract |
| --- | --- |
| POST /api/account/visit-invitations | Owner explicitly enters a recipient address; create one invite and send its one-time link. No automatic follow-up invitation. |
| GET /api/account/visits | Host-only outstanding invitations, active visits, and most-recent-first historical visits with email, date and approximate duration. No public listing. |
| DELETE /api/account/visit-invitations/:id | Revoke an outstanding or active invitation and its grant atomically; remove it from current outstanding/active lists. No success toast. |
| GET /visit/continue | Token landing page with the same scanner-resistant, matter-of-fact continuation pattern. |
| POST /api/visits/consume | Exchange an unused, unrevoked, unexpired invitation for a visit-only browser grant. Concurrent consumes yield at most one grant. |
| GET /api/visits/current/snapshot | Read-only projection of the host's canonical presentation, authorized on every pull. No owner capabilities, notebook, interaction endpoint, or owner session upgrade. |
| POST /api/visits/current/end | Optional best-effort closure for duration accounting; no simulation event. |

Unused invites expire 30 days after issuance. Used links never work again. The separate, 24-hour maximum visit grant allows normal reloads within that browser without making the original link reusable. Do not place a permanent visitor on an allowlist. A new visit after grant expiry requires a new deliberate invitation.

The visitor's allowed local controls are mute/audio permission, captions, reduced motion, and narration. These affect only that browser. Birds are not listen-in buttons in this view; offers, settle, greetings caused by arrival, and presence-report endpoints are unavailable both in UI and authorization middleware. Visitor time uses a separate social audit store and cannot enter the simulation event stream.

## 5. Simulation and temporal continuity

### 5.1 Tick schedule and commit algorithm

Schedule every existing, non-hard-deleted aviary approximately every 60 seconds, including those with no clients. Distribute due times across the minute by a stable UUID-derived offset; never key scheduling by email or “currently active.” Start with batches of 100 due work references and short per-aviary transactions rather than one transaction locking a whole batch.

For each due aviary:

1. Acquire its aggregate row/lease with a fencing token, then establish one transaction and current revision.
2. Read persisted bird state, filter accumulators, prior simulation time, RNG continuation, and the next committed range of interaction events in per-aviary sequence order.
3. Advance any missed logical minutes in order. Partition accepted events into their server-time intervals; never apply a newly received event to a simulated minute before its acceptance.
4. Reconcile qualified presence intervals, listen spans, accepted offer outcomes, pending settle influences, and idempotent reaction programs.
5. Integrate nonnegative personality deltas. Transition mood and ambient processes. Advance or extend bird programs and social responses while preserving ongoing segments.
6. Generate a notebook entry only if a supported noteworthy fact and sparsity gate allow it. Recompute age-based adoption eligibility independently of interactions.
7. Commit new state, cursor, schedule, notebook/outbox records, last_tick_at, next_tick_at, and incremented state_revision together.
8. Publish the committed revision pointer for snapshot delivery. A failed publish is retried from the outbox; clients can always obtain the committed revision from the primary path.

Duplicate queue delivery observes the advanced last_tick_at/fencing token and does nothing. A process crash before commit changes nothing; after commit a retry sees the consumed cursor. Event ingestion and ticks use the same aviary lock to establish a total committed order, but keep command transactions short so a burst of views cannot block a minute's simulation.

Normal idle time is real server simulation, not catch-up on GET. After a service outage, durable workers catch up in fixed logical minute steps from stored state; process at most 120 overdue steps per transaction and requeue remaining debt. Do not discard outstanding interaction inputs to reduce backlog. Suppress playback of past calls and bulk notebook creation during recovery; advance the actual state and choose only supported sparse observations. Reads never trigger a separate competing tick.

Compute throughput needs from total aviaries, not concurrent viewers: N/60 ticks per second before retries. For example, 100,000 accounts imply roughly 1,667 ticks per second and 144 million aviary ticks per day. This is a sizing example, not a claim that the first deployment supports it. Load-test the target cohort, write amplification, backup load, and twice-expected peak before opening that cohort. Stop capacity ramping before tick latency or durable write budgets fail.

### 5.2 Personality drift

Persist five normalized traits: boldness, social warmth, vocal frequency, plumage saturation, curiosity. Seed new birds from bounded species-informed distributions, initially approximately 0.2–0.6, with enough variation to distinguish the two starters. Persist that draw exactly once. A fixed voice identity is not resampled when a trait changes.

Treat attention as exposure, low-pass it, then apply positive deltas to stored values. For bird b and trait j in logical tick t:

    P_t = newly qualified owner-presence exposure, normalized by 900 seconds
    L_b,t = qualified listen-in exposure for that bird, normalized by 900 seconds
    O_b,j,t = bounded offer exposure appropriate to that trait
    u_b,j,t = 0.85 * P_t + 0.10 * a_j * L_b,t + 0.05 * c_j * O_b,j,t
    h = logical elapsed time in days; tau = 3 days initially
    decay = exp(-h / tau)
    r_next = decay * r_stored + (1 - decay) * (u_b,j,t / h)
    delta = max(0, min(1 - x_stored, k_j * r_next * h * (1 - x_stored)))
    x_next = x_stored + delta

Here r is the persisted filter state, not a user-visible statistic. Start k_j in the 0.006–0.010 per-day range and calibrate the full pipeline. Use double precision and stable small-step math. Fixed minute steps avoid a large post-outage step artificially accelerating drift. A later exact integral implementation must preserve the golden scenarios and positive-delta invariant.

P is the same deduplicated owner exposure for every bird; cap credited exposure at two 15-minute reference units per rolling 24 hours, with a smooth taper after the first unit. L is zero outside qualifying presence and capped at one reference unit per bird per 24 hours. For offers, one accepted eligible offer contributes a small fraction of a reference unit; all offer contributions together are capped at one unit per bird per 24 hours. The 85/10/5 weights keep ordinary attention dominant even when someone repeats actions. These are hidden saturation controls, never tasks or allowances to show the user.

Set a_j positive only for social warmth and vocal frequency. Set c_j positive for curiosity on acceptance and boldness on an eligible nearby offer, including a bird that watches rather than accepts. A rejected cooldown attempt contributes nothing. Presence can gently increase all five traits. Settle has no personality term. Repeated clicking, muting, closing a tab, long absence, and declining adoption never subtract from any trait.

During absence, u becomes zero; the filter's prior positive signal decays while its remaining influence may still produce a small positive delta. This implements continued drift from actual earlier attention. With no prior input, there is no drift; once the tail dissipates, the existing personality stays intact. Avoid a “decay toward baseline” function anywhere in migrations, daily resets, or inactive-account jobs.

Initial calibration fixtures use 15 minutes of qualified presence on five days per week, plus occasional listen-in and offers. Aim for typical numerical changes of approximately 0.01–0.025 after a week and 0.04–0.08 after three weeks, subject to initial headroom; those ranges are internal experimental anchors, not product UI. Acceptance ultimately requires a recognizable but unobtrusive three-week change and no visible change within a normal single session. Check cumulative perception across perch choice, sociability, vocal pacing, and plumage, not just a scalar threshold.

### 5.3 Mood and absence

Use wary, content, curious, drowsy, and alert as the v1 enum. Sleeping/settled are poses and lighting/mix conditions, not extra happiness meters. Store mood, onset, scheduled reconsideration, and time-limited influences. Reconsider the baseline over approximately 12–24 hours with jitter, not at navigation or a synchronized midnight reset.

Use a weighted transition/hazard table with minimum ordinary dwell times around ten minutes; user/ambient events can add short influences without making the state oscillate each minute. Time-of-day raises morning alertness and evening drowsiness. Recent accepted offers favor curious/content; rain gently reduces expressed call rate; wind may raise alertness or wariness; a neighbor's sharp call can briefly bias wary. Boldness reduces wariness probability; social warmth increases reply probability. Bound social propagation to one response generation per originating call/event to prevent feedback storms.

Absence does not enter a transition toward wary, sadness, illness, or lower traits. A bounded, fast-decaying attention context may reduce spontaneous owner-directed notices and extra social call responses after a session ends; baseline bird life continues. Every genuine owner return still gets one subtle greeting. Longer absence changes the form of re-orientation, not a numerical penalty or guilt surface.

Mood on reopen comes from the most recent canonical tick plus authorized active reactions. The client cannot assign neutral mood on mount. A baseline reconsideration may keep the existing mood, and changes blend through the current pose rather than snapping to a stock state.

### 5.4 Behavior, day/night, and weather

Each tick publishes approximately 120 seconds of bounded, timestamped behavior. Preserve segments already started; extend beyond the committed horizon with a deterministic RNG cursor so retries do not generate a different morning. Sample perch preference from mood and personality, resolve free anchors in front/middle/back, and reserve paths/endpoints to avoid collisions. No client command sets a perch.

Programs combine pose dwell, preen, scan, tilt, weight-shift, and occasional perch movement. Timing, amplitudes and overlaps vary continuously within species-safe limits. Another bird's scheduled call can provoke a head turn, answer, or nearby perch choice. Chorus arises from overlapping calls and responses, not a prerecorded “chorus” button.

Derive lighting from UTC plus the persisted timezone, using smooth morning/day/evening/night curves. Use a clock-time schedule rather than GPS or a location/weather provider. Offset changes from daylight saving time and explicit timezone edits blend lighting over several minutes; they do not reset personality or replay a day. Include a nightjar-like species whose normal call schedule remains active at night. Other species mostly rest, but the scene remains alive.

Generate weather on the server: initially about two to three soft rains per week, each five to twelve minutes, with occasional mild wind and a refractory interval. Weather has no real-world weather API, severe events, obligations, or notifications. Weather influence expires explicitly and cannot change the long-term vocal-frequency trait downward.

Leaves, falling feathers, and subtle decorative parallax are client render ornaments. They have no per-leaf server state, do not affect moods or notebook facts, and are removed in reduced motion. The server may describe a calm or windy period, but must not claim a particular client-only leaf was observed.

### 5.5 Immediate reactions without client authority

On an authorized view_return or offer, the command handler locks the aviary, reads the current canonical expression, and authors a short reaction envelope using a server seed and unique event ID. It appends the event and envelope in the same commit, then returns the envelope. This is a transient presentation decision, not a second personality writer or a client tick.

The envelope describes start time, preconditions, bird selection, continuous variation parameters, accepted/waiting/ignored outcome, call instance, and pose/perch segments. Every owner/visitor snapshot can include it while active. The next tick consumes the stored outcome exactly once and carries its endpoint and influence into the ongoing program; it does not replay the offer visually. A response delay causes the client to sample the envelope at the current phase rather than restart it.

Greeting selection weights boldness, warmth and mood, with a preference against mechanically repeating the last initiator. Require one initial noticing gesture inside the first one to two seconds under the supported load envelope. Vary glance direction, interruption of preening, head tilt, step distance, pause, motif and call envelope based on absence length and identity. Secondary responders, if any, begin later with randomized offsets; there is no synchronized welcome chorus.

Deduplicate visibility/focus churn with one view-return ID per actual foreground epoch. Concurrent returns from two owner devices within two seconds share the primary greeting decision, rather than creating competing “first greeters.” Do not use the presence-activity gate as a prerequisite to greeting: noticing arrival and crediting sustained attention are separate concepts.

## 6. Presence and interaction state machines

### 6.1 Qualified presence

Maintain a local eligibility predicate evaluated both on relevant events and at reporting time:

    qualified(t) =
      document.visibilityState == visible
      AND document.hasFocus()
      AND t - lastTrustedPointerMoveOrKeypress <= 5 minutes
      AND this view is not settled or ended

A pointermove or an actual keyboard press updates the activity timestamp. Implement keyboard presses with the appropriate modern keyboard event rather than depending on the deprecated DOM event name; ignore synthetic dispatches and modifier-only activity. Do not count a timer, snapshot receipt, audio playback, network activity, click alone, focus alone, or merely opening a tab as proof of the required activity signal. Never collect pointer positions, key values, or text. Touch movement can supply a real pointermove; do not quietly widen the definition to every tap.

Use monotonic browser time for durations. At authenticated view creation, establish a server-time/monotonic-time anchor and a unique stream ID. Re-anchor after navigation or a long suspension; do not join intervals across a clock discontinuity. A five-minute window lets someone watch quietly without constant movement while bounding credit for a laptop left open. Show no countdown, “still there?” prompt, or demand to move.

Presence state transitions:

- **Ineligible to eligible:** open a qualified interval at the transition; no backdating to navigation.
- **Eligible reporting:** every 15 seconds send only the elapsed, qualified portion since the last report.
- **Blur, hidden document, activity expiry, settle, or pagehide:** close the interval at that boundary; send a terminal fragment if possible.
- **Tab close or lost connection:** credit only already reported elapsed intervals. No assumed future lease creates presence. Missing final seconds are preferable to inventing attention.
- **Foreground return:** refresh state and request a greeting; reopen presence only if the full conjunction holds.
- **Settled view:** remain ineligible even if harmless pointer movement occurs. Explicit re-engagement ends the settled state, after which the normal predicate applies.

On the server, validate stream/session ownership, sequence, maximum segment length, mapped timing, and terminal markers. Admit reports at most 30 seconds late and at most 30 seconds long; clamp minor end-time skew to receipt time and reject larger future/backdated claims. A repeated event ID returns its original receipt. Arbitrarily large or historical client durations cannot become exposure.

Normalize accepted intervals to a server-time basis and add only the interval difference against stored coverage. Overlapping intervals from laptop and phone therefore count once. Late reports admitted after a tick contribute only their newly uncovered duration to a subsequent tick; they do not rewrite an old vector. Expired reports and reports preceding the coverage finalization watermark are rejected rather than creating new credit after pruning.

Client visibility/focus evidence is an honest approximation of attention, not proof that a human looked at a pixel. Do not add invasive monitoring or pretend the server can independently verify it. The server enforces timing and duplication bounds; tests enforce the three-signal conjunction in the actual client.

### 6.2 Listen-in

Store one attention target per local view. Pointer activation toggles the same bird; selecting a different bird ends the previous span and begins another. Keyboard focus begins a span; Enter is idempotent engagement; Escape ends it; blur or focus leaving the bird ends it. Clicking empty scene space ends it. Distinguish pointer-induced DOM focus from keyboard focus so a click that disengages does not immediately re-engage through a focus handler.

Send listen_start and listen_end with the same span ID. Server duration is bounded by the span's accepted start/end, associated qualified presence intervals, and view termination. A missing end cannot accumulate beyond the last eligible reported interval. Retries cannot create a second span for the same ID.

For simultaneous devices, each bird's credited listen time is a union, and aggregate listen credit across birds is bounded by the account's qualified attention time. If two devices focus different birds over the same second, divide that second's secondary listen exposure between their active targets. Do not double the total effect by opening more devices.

The audio mix follows the local target immediately and independently of the next personality tick. Changing the mix never forces a bird to call on command. Listen-in does not imply soloing, a badge, a numerical state, or visible inline labels.

### 6.3 Offers

The offer popover provides seed, a song fragment from a small authored motif library, and a still pool. The server chooses or validates the receiving bird set from current position, mood and curiosity. An optional recipient choice is in this top-bar popover, not a bird-click menu. A whole-scene offer may have several eligible responders, but each is reserved and cooled down separately.

Reserve a three-minute cooldown for every bird assigned a meaningful reaction, including an eligible bird that waits or watches. Resolve reservation and event acceptance atomically across devices. A concurrent second offer cannot bypass the first by reading stale availability. A bird still on cooldown may remain visually present but gains no new offer exposure. Do not return a countdown or failure animation.

Seed reactions range from approach and investigate to delayed interest or no approach. A pool permits drinking, bathing or watching; a song fragment is softly synthesized from an authored note/motif grammar, with a bird joining, pausing or answering according to its expression. There is no food quantity, inventory, hunger, feeding requirement, or recorded audio clip.

Author the reaction on the server and return it promptly. The local popover may close immediately, but acceptance, mood response, and notebook facts require the acknowledged outcome. If the write result is uncertain, query the receipt by its original event ID. Do not animate a successful offer and later silently pretend it was ignored.

### 6.4 Settle, undo, and ordinary departure

Triggering settle creates a settle instance and immediately ends this view's qualified interval. Locally ramp lighting toward evening and calls toward a quieter ambient mix over approximately four seconds; keep the current birds and ongoing behavior. Maintain this overlay until closing the tab or explicitly re-engaging, rather than automatically popping back to daytime when the next snapshot arrives.

Any click in the aviary during the following five seconds reverses that settle. Keyboard Enter/Space while in the scene offers equivalent reversal. Consume this gesture as undo rather than also offering or toggling a bird. Return lighting and mix with a gentle inverse ramp, using the current natural-time lighting target. After five seconds, a deliberate scene action still re-engages normally; there is no persistent “undo” button or toast in the scene.

Append settle and settle_undo/reengage with the same settle instance ID. Hold the shared mood influence pending for five seconds to absorb the ordinary undo case. If cancellation races a tick, remove only that settle's temporary influence at the next tick; never roll back personality or unrelated events. Local reversal remains immediate. Late-network cases may leave a brief shared quiet mood until that next tick, which is a bounded transient effect rather than a state reset.

Closing without settling ends presence identically and adds no penalty. Settle supplies only its explicitly requested mood quieting and local lighting ritual; it has no bonus drift, streak credit, mandatory confirmation, or recovery message. If another owner device stays active, its independently qualified presence continues.

## 7. Snapshot consumption, outages, and concurrency

### 7.1 Pull cadence and revision rules

Bootstrap from the snapshot embedded in authenticated HTML. While visible, pull every 15 seconds with small jitter; use ETags to avoid unnecessary full bodies. Pull immediately on visibility regain, window focus regain when stale, an online transition, and a render-frame gap greater than two seconds. These reads refresh canonical state, not presence. Hidden tabs stop periodic visual pulls and rendering; server simulation continues.

Maintain a monotonic observed state_revision and event_head. Ignore older out-of-order responses. Keep locally acknowledged reaction envelopes until a snapshot covers their event sequence; do not erase a just-accepted offer with a response that was started before the offer. When a new tick incorporates that event, reconcile by program/event ID so its visual action is neither replayed nor applied twice.

Every full snapshot is internally consistent from one committed database view. Devices may briefly display different revisions between pulls, but they never have different independently authored personalities. Persistent settings and names use expected revisions and clear system conflict responses. There is no last-write-wins merge for personality and no automatic overwrite of a stale form.

### 7.2 Interpolation and clock handling

Estimate server time with request/response timestamps and a bounded round-trip estimate; use monotonic elapsed time thereafter. Smooth small clock corrections. Sample the same server-authored motion segments at estimated current time, including a segment that began before navigation. A newly received perch change has an explicit start, path, duration, and endpoint, not two isolated coordinates to teleport between.

If a snapshot interrupts a running program, preserve position/pose continuity with a short blend to the new authorized segment. The shared sampling runtime makes no mood or drift choices. It may advance the drawing phase, not the domain state. Decorative breathing is bounded presentation, not proof of a new server interaction.

On a long sleep, fetch current state before scheduling new calls. Sample whatever is in progress now and discard missed call start times. Never replay a night's queued calls, rapidly run hours of animations, or burst old narration on resume.

### 7.3 Failure policy

Use a bounded, in-memory retry queue for interaction receipts and short-lived presence fragments. Do not persist a replayable offline activity log or an offline bird state. Retry confirmed-idempotent operations with exponential backoff and jitter, initially one second, capped at 30 seconds. Drop presence fragments outside the admitted timing window.

While a valid authorized program remains available, continue rendering it. Once its horizon expires during an outage, preserve the last real birds and use only restrained decorative breathing or reduced-motion pose holds. Stop inventing calls, reactions, mood transitions, and personality changes. Show a matter-of-fact connectivity state in the system region when recovery is needed; use a quiet field if no authorized snapshot has ever arrived. There is no spinner or fake adoption fallback.

On 401 or revoked owner session, stop event submission, clear private state and sound, and show “Your session timed out. Sign in again to keep watching.” On unknown write outcome, retain the event ID and resolve its receipt. On 409 settings/name conflict, show the current value and a clear option to reapply the user's edit; do not present a bird-state merge chooser.

Visitor revocation checks happen at every authorized pull, including ETag checks. An active visible visit learns of revocation on its next pull, normally within 15 seconds. Hidden/suspended visits authorize again before resuming. Give visitor snapshots a short display lease of at most 30 seconds; if offline beyond the lease, stop the visit rather than allowing an unrevocable cached viewing session. On denial, tear down audio/rendering and display the same “This visit is no longer available” surface for expired, revoked, or invalid access.

## 8. Frontend rendering and scene interaction

### 8.1 First paint

The critical response contains the actual authorized birds, sampled partway through their current actions, as inline SVG. A tiny inline bootstrap binds the existing nodes, samples current phase before its first animation frame, and starts the render loop. Hydration adopts those nodes; it must not replace them with a neutral pose, fade them in, or replay entry motion.

Keep snapshot acquisition and critical SVG generation on the fast HTML path. Critical path assets are system fonts, compact CSS, current species paths, presentation state and the small sampler. Settings, notebook history, invitation forms, detailed accessibility settings, and non-current species assets are lazy imports. Do not wait for WebAudio, account-panel code, an image decode, or a full client application boot to draw the first bird.

On genuinely slow state delivery, show a quiet sky-colored field with minimal ambient cues. This is an exception/failure presentation, not a substitute for meeting the first-bird budget. It contains neither an inaccurate placeholder bird nor a loading spinner. Account creation is the sole empty-aviary transition: atomically establish the two starter identities, accept/default their names, then show a soft fly-in once. Store an onboarding-completed marker so reloads do not repeat it.

### 8.2 Scene graph and responsive layout

Use one root SVG with a stable logical coordinate space and a contain transform:

1. Sky and time-of-day gradient.
2. Soft background foliage.
3. Back, middle, and front perch anchors and branches.
4. Bird bodies, heads, wings, tails and small feather detail.
5. Occasional foreground ornament layer.
6. Local caption and keyboard-focus layers, only when needed.

Use approximately 300 scene nodes or fewer at seven birds as the initial art budget. Prefer species silhouettes made from reusable SVG shapes and gradient/style parameters, not raster frame atlases or large bitmap stacks. Keep bird identity visible in silhouette, pattern and voice, not only in saturation.

Build normalized anchor sets for narrow, medium and wide viewports. Reflow anchor spacing without changing the authoritative front/middle/back choice. The viewport mapper owns collision-free screen coordinates; the server owns behavioral proximity. Interpolate between layout mappings on resize, or cross-fade in reduced motion, without creating a behavioral flight event.

Fit all seven birds inside scene bounds, including wings, flight-path extrema, outlines and captions. Use 44-pixel target regions where practical, distribute birds over all three depth zones, and prevent hit-region overlap by layout. Phone portrait and landscape must not crop birds or require scene scrolling, panning or zooming. Browser page zoom remains allowed; do not disable it in viewport metadata. Very short/zoomed viewports may letterbox the contained scene while account/notebook panels scroll normally outside it.

Establish supported layout fixtures at 320×568, 390×844, 768×1024, 1366×768, and 1920×1080, plus mobile landscape, dynamic browser toolbar changes, 200% text resize and 400% page zoom. The whole-scene no-scroll constraint applies to the aviary, not to a long notebook or system form.

### 8.3 Motion and controls

Sample motion using one requestAnimationFrame loop and reusable numeric state. Update only transform, opacity and limited color properties for changed parts. Do not rerender a UI tree at 60 Hz. Preen, scan, head tilt and weight-shift layers use seeded offsets and mood-shaped authored ranges so birds do not share identical phases. Background parallax is tiny and not tied to aggressive pointer tracking.

Use a bounded ornament pool, for example eight leaves/feathers maximum, with low generation frequency and immediate reuse. No per-leaf network records. Suspend requestAnimationFrame and ornament work while hidden; remove obsolete listeners and cancel scheduled callbacks on unmount.

The scene has no permanent labels, buttons, badges, tooltips, status icons, or draggable birds. Accessible bird hit targets are visually transparent until keyboard focus; clicking a bird means listen-in only. Captions and focus outlines are explicit accessibility exceptions, not a foothold for general scene chrome.

The top bar has four icons as specified. After four seconds of pointer stillness and no keyboard focus/open panel, fade the bar background and decorative parts over one second. Restore on pointer movement, keyboard activity, or touch. Pin full visibility for a focused control, open menu, active system error, or accessibility interaction. On touch devices keep the actionable controls readily visible; do not require a hidden-hover discovery gesture.

The offer popover also contains settle in a separate group. Account/settings contains bird naming/adoption, sessions, visits, timezone, export/deletion and privacy. Accessibility contains audio/mute, captions, reduced motion and narration controls. The notebook icon opens the read-only notebook. No new-bird dot or unread-note badge is attached to any icon.

### 8.4 Reduced motion as a full renderer

Choose reduced motion before the first visible motion using the OS media query and saved system/on preference. OS reduced motion is honored; an account setting can force it on. Listen for preference changes without requiring reload.

Replace continuous preen/scan movement with distinct still poses, changing roughly every eight to fifteen seconds via slow 1.5–3-second cross-fades. Replace perch flights with a cross-fade between correctly placed endpoint poses. Remove moving leaves, feathers and parallax. Keep day/night/weather color transitions, slowed to at least ten seconds for explicit changes. Do not flash normal flight before the preference is applied.

Greeting becomes a carefully timed pose change or cross-fade, with the same actual bird noticing and corresponding call/narration. Offers, moods, drift, notebook facts and full-quality audio are unchanged. Settle uses a slow lighting cross-fade. This mode must pass the same relationship/identity review as the default motion renderer, rather than being accepted because nothing moves.

## 9. Procedural audio and caption derivation

### 9.1 Stable voice, varied call

Author approximately six coherent species with distinct silhouettes and call motif grammars, including the nightjar-like nighttime signature. Each adopted bird receives a stable voice fingerprint within its species: register, timbral envelope, rhythm tendencies, interval profile and subtle breath/noise color. With seven birds and six species, at least one species may repeat; within-species fingerprints must remain distinguishable. There is no rarity hierarchy.

A call instance contains a unique call ID, bird/voice identity, grammar version, motif path or seed, authorized start time, tempo and expression bounds, spatial position, and variation seed. Expand it into a bounded intermediate representation of note segments, pitch curves, gaps, envelopes and texture events. Expand once per call and reuse that representation for both synthesis and captions.

Allow mood to vary pace, spacing and amplitude, and personality to vary calling probability and chorus participation. Limit pitch/timbre changes to the bird's identity envelope; start by limiting expressive register shifts to roughly two semitones and review by ear. Do not morph an established bird into a new voice because its vocal-frequency trait rises.

Use oscillators, shaped envelopes, filters and optional short synthesized noise grains through WebAudio. The small song-fragment offer library is authored note/motif data rendered by the same procedural machinery, not imported recordings. No call, chorus, ambience substitute, or compatibility path downloads recorded bird audio.

### 9.2 Scheduling and mixing

The server programs when birds call and how social responses arise. The client synthesizes those calls locally. Schedule with the audio clock using a short look-ahead, initially 150 milliseconds with a 50-millisecond scheduler interval, inside the received program horizon. Use one AudioContext per view, one master chain, reusable per-bird gain/pan/filter nodes, and a bounded transient voice pool.

Start with at most four oscillator/noise components per bird call and a maximum of 28 live synthesis voices. Normally schedule no more than three strongly overlapping call starts, leaving audible space for signatures; all seven birds remain represented across time. Normalize energy, preserve headroom, and use a soft limiter as protection, not as a substitute for a restrained score. A chorus means two or more real, independently varied call instances, sometimes including replies, not synchronized loops.

Listen-in gradually increases the target bird's relative gain and reduces the rest over approximately two seconds. Keep each other scheduled bird's gain strictly positive; initial relative weights can be 1.6 for the target and 0.35 for others before bounded normalization. Returning to ambient takes the same two-second ramp. Smoothly retarget an ongoing ramp on quick focus changes rather than stacking gain automations or making hard cuts.

Settle adds a gradual local quieting factor but does not zero the chorus. Explicit mute is the legitimate zero-master-gain case. Never amplify the focused bird enough to clip the master or make the user's quieter volume choice ineffective. Small stereo positions follow perch geometry without extreme pans.

If a call starts slightly late, align to its current phase where practical; if it has passed, skip it. Never play it from the beginning on every fresh snapshot. On hidden tabs, stop audio scheduling and suspend the context to preserve battery; resume from current authorized state after visibility returns. Keep the server's call/mood timeline running while the client is silent.

### 9.3 Browser permission and failure

Attempt to use an existing permitted context when the saved audio preference is enabled. On first-use autoplay blocking, render the actual aviary silently with captions enabled locally. A genuine user action can resume audio if the user has not muted it; a quiet “Enable calls” control is also available in accessibility/audio settings. Do not show a welcome/permission modal or claim sound started when the browser blocked it.

If WebAudio is absent, fails construction, stays unusable after permission, or fails during playback, keep the renderer and simulation running, turn local captions on by default, and expose direct system status in audio settings. There is no recorded fallback. A browser-specific failure must not overwrite the account's audio preference on working devices.

### 9.4 Captions from the actual call program

Derive captions after procedural motif expansion, from the same note count, rise/fall contours, trill spacing, intensity and pause structure given to the synth. Examples include “a soft three-note rise” and “a low trill, paused, low trill again”; these are grammar outputs, not fixed labels attached to a species.

Tie caption lifecycle to the actual scheduled call ID and adjusted start/end. If a stale call is skipped in normal audio playback, skip its stale caption as well. In silent fallback, caption the current authorized call that would be synthesized. Caption timing remains synchronized when a listen-in ramp changes loudness.

Place short captions near the calling bird in collision-resolved lanes with opaque-enough text backing to meet contrast at day and night. Reserve enough space for up to three concurrent calling birds; use up to two short lines per caption and stable placement, rather than overlapping prose or hiding one bird's call. Hold briefly for legibility if a call is very short, then fade without abrupt flashes. In reduced motion use the same slow-opacity visual language.

## 10. Accessible interaction and narrative

### 10.1 Semantic model and keyboard

Provide a semantic aviary region, labelled top-bar buttons, and a stable, roving focus set keyed by bird UUID. Tab visits the top bar controls and enters the scene at the first bird. Arrow keys move between birds in a predictable, documented order; Enter engages/resumes listen-in; Escape ends listen-in and closes an open popover as appropriate. Tab away ends listen-in. Preserve the focused bird identity across snapshots, perching changes and renames.

Native buttons and dialogs handle focus semantics. Do not apply a blanket application role to the whole page. Bird accessible names identify the bird and its available action, not trait values, mood codes or numbered perches. Visitors get a narratable scene without interactive bird/listen-in buttons.

Offer is fully keyboard-navigable through its top-bar button and an optional Alt+Shift+O shortcut, which can be disabled in accessibility settings. Do not make a global shortcut the only route or intercept ordinary typing in fields. Seed, song-fragment choice, pool, recipient selection and settle all work with normal menu/form keys. Escape closes the popover and restores focus to its opener.

Use a dual light/dark focus outline that remains visible against morning sky, foliage, a bird silhouette and full night. Never lose it under a caption or faded top bar. Popovers constrain focus while open and restore it predictably; notebook scrolling does not trap the keyboard in an infinite list.

### 10.2 Running naturalist prose

Build an observation model from the same presentation snapshot and active reaction descriptors that the visual renderer reads. It includes bird identity, relational perch description, observable action, audible call structure and ambient light/weather. A grammar of edited phrases composes a short connected paragraph. It never reads raw traits or converts enums into status announcements.

Use a polite, atomic live region with one idle update about every 45 seconds, bounded to the PRD's 30–60-second cadence. Do not enqueue identical paragraphs. A newer idle observation replaces an unsaid old one. User-initiated greeting, acknowledged offer and settle observations receive priority; keep at most two pending event observations plus one idle paragraph so a slow screen reader cannot accumulate minutes of stale speech.

Narrate the observation promptly without assertive interruption: “pip lifts her head from the front rail; a short call meets the quiet.” Do not say “Welcome back,” “offer succeeded,” “mood changed,” or “Pip: perch 2.” Ordinary procedural calls do not each produce a competing live-region announcement; call captions remain independently browseable, and important call changes can be folded into the next prose paragraph.

Expose optional visible narration and pause/resume/repeat controls in accessibility settings. Pausing narration affects only speech updates, never the simulation. Captions do not duplicate every call into the screen-reader queue. Prompt system errors have their own matter-of-fact accessible status and are not disguised as nature observations.

### 10.3 Contrast, reflow and sensory independence

Specify final color tokens with measured ratios. Require at least 4.5:1 for normal user-copy text, 3:1 for qualifying large text, and at least 3:1 for focus/control boundaries as applicable. Cover account/error forms, labels, captions, visible narration and controls over both weather and lighting extremes. Use a solid or controlled translucent caption background whose worst-case composited contrast is tested.

Names, actions and notebook prose cannot rely on color or sound alone. The two starter birds remain identifiable by silhouette/markings as well as names in accessible surfaces. Calls, greetings, offers and settling have meaningful visual and prose equivalents. Preserve readable line lengths, text scaling, browser zoom, and non-hover access on phones.

Include NVDA with Firefox/Chrome, VoiceOver on macOS/iOS Safari, and a Windows screen-reader/browser combination such as JAWS/Edge in the release verification matrix. Automated semantics and contrast tests are necessary but not sufficient: people using screen readers and reduced motion must assess whether the surface feels like birds in a place.

## 11. Notebook, naming, and gradual adoption

### 11.1 Observation generation

The simulation emits internal candidate facts, not ready-made generic event-log messages. Initial rules include a changed ordering of bird greetings, an unusually long quiet preen, a distinctive perch choice over time, a response to a rare rain, or a new social call pairing. Facts must be supported by server-authored state/programs and bounded historical summaries.

Score candidates for specificity and novelty within that aviary only. Start with an ordinary minimum interval of 48–72 hours, a hard cap of four entries in a rolling week, and at most one exceptional bypass per week with at least twelve hours since the last entry. The objective is roughly one entry every few days for regular use, not writing up every session. Tune down if the notebook feels like a feed.

Compose lowercase, present-tense, bird-specific prose using an authored, versioned grammar. Preserve names as observed in old entries; renaming does not rewrite the past or break stable bird references. Do not claim “first time this week” without a sufficient retained factual window. Do not infer a user's habits, list sessions, report elapsed presence, or write that the owner visited every day.

Never derive notebook facts from client-only leaves, inferred physical listening, visitor presence, or an unacknowledged offer. Never send interaction data to a remote language model for phrasing. The content system needs substantial editorial variety, but can remain deterministic, testable and private.

Store final entries, not just templates requiring historical reconstruction. Use an indexed (aviary_id, observed_at, id) cursor. Load 30 at a time and release offscreen presentation nodes without making old entries inaccessible to keyboard/screen-reader navigation. Offer an accessible “older observations” pager as an alternative to virtualization. There is no edit, annotate, delete-entry, share-entry, unread count, or archive-hidden-history surface.

### 11.2 Starter identity and later adoption

The first owner activation creates two distinct system-selected species, stable IDs/seeds, and default name suggestions. The user meets those birds and can choose names; there is no species catalog or “best starter” choice. Names accept bounded Unicode text, initially up to 40 grapheme clusters, with control characters removed and safe text rendering. Renames require no behavior migration.

Create age opportunities at 90, 180, 270, 365 and 540 days from aviary creation. A year-old aviary can therefore have five or six birds depending on what the owner accepted. Offer at most one pending introduction at a time. A deferred opportunity remains available quietly in bird settings; absence, inactivity or declining it never expires a bird, resets the age clock or rerolls a species.

The offered species and individual voice seed are fixed for that opportunity. Later opportunities can include an already present species because the species pool is about six and the aviary can contain seven birds. Calibrate within-species voices before enabling the seventh bird. Adoption is explicit and optional, with no notification, badge, level, count display, cost, required visit history, or quantity-based celebration.

The server enforces both age eligibility and the seven-bird cap in one transaction. Age uses actual elapsed UTC time from creation, not tick count or number of sessions, so downtime and travel cannot alter eligibility. A rollout gate may delay offering a larger capacity to an entire cohort for reliability, but it must not tie availability to behavior or remove birds already adopted.

## 12. Privacy, account lifecycle, and access control

### 12.1 Separate relationship data from operational data

Simulation interaction data is used only to advance that account's aviary and produce its own notebook. Enforce this with database roles, network boundaries and a telemetry schema allowlist. There is no analytics/warehouse reader, CDC export, general-purpose query service, or model-training connection to bird tables or event storage.

Operational metrics may include request/error counts, latency distributions, tick delay, render timing, audio-context failure counts and anonymized session-duration histograms. Emit coarse histograms without an account, bird, session, invitation, name, species, mood, trait, call, offer, presence-duration history, or notebook-content dimension. A session-duration histogram is produced as an anonymous bucket, not a joinable timeline of a person's visits.

Short-retention operational logs may use synthetic account UUIDs to diagnose authorization or system failures; they contain no bird payloads or interaction details. Default retention is seven days and they support deletion by account UUID. Metrics pipelines never get those account IDs. Normalize URL paths before logging so bird IDs, addresses and tokens do not leak through path/query strings.

Allow transactional mail delivery to receive only the intended recipient and the specific requested system message. Do not attach bird facts, interaction payloads, or instrumentation identifiers. Disable click tracking and provider analytics on sensitive links. No third-party analytics, session replay, advertising, remote fonts, or production behavior-analysis SDK belongs in the browser.

### 12.2 Authentication and request safety

Generate cryptographically random 256-bit link/session secrets; retain hashes rather than usable tokens. Use atomic consume conditions against expiry and consumed_at. An email-scanner GET cannot consume either auth or invitation state. Ensure browser continuation and API responses use no-store and no-referrer handling; scrub tokens from application, CDN, proxy and error logging.

Apply same-origin/CSRF protection on cookie-authenticated mutations, a restrictive CSP, output escaping, and bounded schemas. Store user names as text and render them through text-safe APIs, including SVG accessible names and notebook prose. Normalize email conservatively without provider-specific dot/plus rewriting. Verify a changed address before committing it and check uniqueness at commit.

Every object lookup combines its ID with an authorized owner or visit scope. A valid bird ID from another aviary grants nothing. A visit cookie is a separate, path-scoped capability and cannot satisfy owner middleware, even if the visitor also owns an aviary in the same browser.

Return generic email-request outcomes to avoid account enumeration. Keep rate limits reasonable and recoverable in direct system copy. Do not show the visitor whether a link failed because it was revoked, expired, guessed, or already used. These controls protect identity and correctness without adding product ceremony to ordinary watching.

### 12.3 Visit transparency and notification semantics

Show the host's visit history only on demand in account settings: recipient email, date and approximate duration, plus outstanding/active invitations. Resolve the email through the authorized encrypted identity record at display time; it is never the primary key. Estimate duration from successful snapshot pulls and stop it after a missing-pull timeout, accepting approximately one pull interval of uncertainty. Never interpret that duration as host-bird presence.

Keep visit history for 180 days as the initial retention default, documented in the privacy copy; the PRD specifies indefinite notebook history, not indefinite social audit retention. Expired/revoked invitations are removed from the current invitation/active-visitor list. Existing historical visits can remain as historical access records until their retention limit or account deletion; there is no revocation-success alert.

With visit_notifications false, record visits silently. With it explicitly true, publish a small account-panel status only while the host has the visit-log surface open; announce it in matter-of-fact prose for assistive technology. Coalesce repeated pulls into one visit, not repeated notices. Do not surface the setting in onboarding, turn it on automatically, or send anything while the host is away.

The owner sees no special effect in the aviary when a visitor arrives. A visitor sees the same canonical birds, time and weather, including host-authored events that naturally occur, and cannot cause any new event. Match host/visitor projection fixtures at equal revision/time, apart from permissions and local accessibility/audio preferences.

### 12.4 Export

An explicit export request creates a background job using a consistent read of the owner's current state and all notebook entries. Capture export schema version, generation time, aviary/bird UUIDs, names/species, adoption times, current vectors, current moods, notebook, and account settings. Do not include session secrets, invitation tokens, internal encryption keys, raw owner activity history, or another account's private state.

Stream large notebooks into JSON under a consistent database snapshot instead of building an unbounded in-memory object. Encrypt the temporary object, make it available only to the requesting owner, and expire it after 24 hours. Mail a download link to the currently verified address; require authorized retrieval. Do not mail the data itself or expose an in-product preview of numeric traits.

This is the explicit vector-portability exception in section 1.3. The UI can say “Download account data” in system voice; it must not explain the export as a way to optimize bird statistics. Exports have no import/reset endpoint in v1.

### 12.5 Soft and hard deletion

Soft deletion immediately marks the account, sets a UTC deadline exactly 30 days later, disables owner interaction writes, revokes outstanding/active visits, and restricts existing browser sessions to recovery and system functions. Keep state intact during the grace period and continue its slow simulation so recovery finds the same living aviary, not reseeded birds. Signing in within the window exposes “I changed my mind”; only that explicit action cancels deletion.

Recovery clears the pending deletion and preserves UUIDs, vector/filter values, voices, notebook and elapsed aviary age. Do not reinstate old visitor grants implicitly; sharing must be deliberate again. At or after the deadline, recovery and decryption access are denied even if a cleanup worker is temporarily delayed.

Hard deletion removes account/aviary/bird state, vectors, filter accumulators, events, coverage, notebook, invites/grants/logs tied to that account, sessions, challenges, exports, jobs, account-scoped operational logs, indexes and caches. Remove references where this account was a visitor without deleting an unrelated host's aviary. Delete orphan invite-only identity rows when no other authorized relationship requires them.

Design backup erasure before launch. Encrypt private account payloads with account-specific keys held in a separately erasable key service; do not make deleted keys recoverable from ordinary database backups. At the hard deadline, revoke/destroy all relevant key versions, flush or fence worker key caches, and delete live rows/objects. Retained backup ciphertext must become irrecoverable at that deadline and expire on the bounded backup schedule, initially 30 days. State this physical-ciphertext versus recoverable-data distinction plainly in the privacy implementation notes.

Restore procedures consult the surviving key registry before making any account readable, purge rows whose keys are gone, and never restore deleted access grants from a backup. Test a recovery from a backup taken before deletion and prove that the deleted account still cannot be read. Irreversible, anonymous aggregate metric buckets have no account linkage to delete; any telemetry record still tied to the UUID must be removed.

## 13. Performance budgets and operational measurement

### 13.1 Budget table

| Surface | Required budget and initial engineering allocation | Verification |
| --- | --- | --- |
| Initial JavaScript | Hard cap below 2,000,000 gzipped bytes loaded for initial paint. Target below 180 KB for the initially used application code, with a motion/bootstrap module below 25 KB. | Build manifest counts all eager/transitive chunks; network trace verifies actual initial fetches. |
| Critical response | Target first-paint HTML/CSS/SVG/state/bootstrap transfer below 60 KB compressed for two birds; no blocking remote font/image/audio request. | Cold-navigation waterfall and server timing, including auth and snapshot fetch. |
| First real bird | Under 500 ms from navigation on the agreed physical mid-tier phone/4G baseline. Use p95 under 500 ms as the release gate, report raw outliers, and test both cold and warm navigation. | Browser trace plus screenshot/pixel-based proof that the visible node is an actual authorized bird, not a placeholder. |
| Greeting | One bird notices inside one to two seconds on the same supported connection; secondary greetings stagger. | Event receipt, visible pose/call start and narrative timing under controlled latency. |
| Snapshot | Initial target at most 8 KB compressed for two birds and 24 KB for seven, including bounded programs; history loads separately. | DTO size tests and sampled byte counts without payload logging. |
| Idle rendering | 60 fps on a five-year-old mid-range laptop, including minute 30. Aim for renderer CPU work below 6 ms p95 within a 16.7 ms frame. | Thirty-minute traces at two and seven birds, including weather/captions. |
| Main-thread stalls | No recurring tasks over 50 ms while idle; lazy panel initialization must not interrupt a call/flight. | Long-task metrics and scripted interaction traces. |
| Audio | Bounded 28-voice pool, one AudioContext, one scheduler, no repeated initialization, no audible scheduler underruns at the certified load. | Offline synth stress and live audio/CPU tests. |
| Memory | No sustained retained-memory growth over 30 minutes; stable DOM, listener, oscillator and buffer counts after warm-up. | Real CI browser session with collection-assisted samples and retained-object checks. |
| Simulation | Approximately one-minute cadence for every aviary; alarm if p99 tick latency exceeds five seconds. Measure both compute duration and completion lateness from scheduled due time. | Synthetic idle/active fleets, operational histograms and backlog-age alarms. |
| Authorization | Every visit snapshot/304 rechecks grant status; visitor display lease at most 30 seconds without renewal. | Active revoke, hidden-return and offline expiry tests. |

Use a fixed, documented mid-tier Android phone and a five-year-old mid-range laptop as physical reference devices. Record CPU/memory, OS, browser version, network RTT and bandwidth for reproducible comparisons. A starting 4G lab profile is 10 Mbps down, 3 Mbps up and 80 ms RTT; include slower profiles to inspect degradation, not to redefine away the 500 ms target. Last-two-major-version browser coverage is updated with each release.

Treat the 500 ms budget as an architecture test early in development. If auth, region placement, TLS, snapshot generation or too much critical code consumes it, fix that path before visual polish. A 2 MB-compliant bundle alone does not prove first-bird performance. Do not disguise missed performance with an invented local bird.

### 13.2 Bounded resource lifetime

Pool ornaments and audio buffers; cap scheduled-call queues at the snapshot horizon and remove finished call references. Stop and disconnect completed transient audio nodes and cancel abandoned gain schedules. Reuse stable per-bird gain/filter nodes. Tear down a view's context, workers, timers and listeners on account switch, visit termination or page teardown.

Notebook virtualization must release view objects and subscriptions when entries leave the retained window, while the accessible pagination path keeps history reachable. Keep only a small bounded number of fetched pages in memory, initially three. Do not retain every snapshot or receipt for the lifetime of the tab; retain only current state and bounded reconciliation receipts.

The 30-minute memory CI test exercises repeated listen-in changes, offers respecting cooldown, panel open/close, notebook pagination, visibility changes, reduced-motion toggles and audio suspension/resumption. Sample after a five-minute warm-up. Require stable retained object counts and no statistically sustained positive heap slope; permit only a documented measurement-noise band, initially 1 MiB between collected samples, not repeated growth disguised as tolerance.

### 13.3 Instrumentation from day one

Add performance marks for navigation, authorized state arrival, first real bird paint, critical sampler ready, first greeting expression and audio readiness. Collect render-frame distributions, long tasks, memory/resource counts in synthetic runs, audio-context errors, snapshot/command latency, tick compute time, tick due-time delay and job failure counts.

Production RUM receives only allowlisted anonymous timings and failure codes. Do not attach bird count, species, trait values, notebook text, interaction event IDs, offer/listen counts, owner activity timelines or precise presence durations. Use synthetic fixtures to compare two versus seven birds and to calibrate drift; do not add a per-account relationship dimension just because a chart would be convenient.

Run scheduled synthetic browser checks from common serving geographies, with both cold loads and returning views, and nightly 30-minute endurance runs. Alarm on p99 tick latency above five seconds and on growing oldest-due backlog; alarm on repeated first-bird violations, authorization errors and audio initialization regressions. Route alerts to engineering operations, never to product users.

Prohibit measuring DAU/retention/streak optimization, “average boldness,” popular offers, species rankings, most-visited aviaries, or social growth funnels from interaction data. No underlying leaderboard data is computed in anticipation of a future surface. Use build health, performance, correctness fixtures and designed-user research with synthetic aviaries to guide product quality.

## 14. Verification plan and release evidence

All tests in this section are implementation work for the future engineering team. The current phase-one deliverable does not run or implement them.

### 14.1 Deterministic simulation and persistence tests

Use a seeded RNG, injectable clock and persisted-state fixtures, with the same tick function used in production. Required scenarios:

- Twenty-one days of ordinary qualified visits; a week of observation without clicks; intense repeated offers; two weeks and ninety days of complete absence; long sessions versus many short sessions.
- Assert every trait delta is nonnegative and every value bounded. No-input birds remain stable; prior-input filter tails can advance during absence; session-level changes stay below the agreed perceptual threshold.
- Verify distinct initial birds, immutable IDs/voices, rename continuity, same stored vectors after deployment/migration, and no personality reconstruction from events.
- Verify morning, evening, nightjar activity, daily-ish mood changes, weather expiry, social response bounds and DST/timezone changes without mood resets on login.
- Verify age-only adoption, fixed deferred species, creation retries, simultaneous acceptance, and the seven-bird cap.
- Produce internal diagnostic plots and comparison render/audio fixtures from synthetic state. Never use production vectors to make population calibration plots.

The perceptual tests use first-day, one-week and three-week snapshots with identical species/identity and comparable time-of-day. Reviewers should see little or no within-session change, instruments should detect the week change, and the three-week presentation should feel different while remaining the same bird. Include low/high initial trait headroom and an inactivity control.

### 14.2 Event, multi-device, and failure tests

| Case | Required result |
| --- | --- |
| Two devices report the same eligible minute | One minute of presence, never two; overlapping listen targets share the secondary attention budget. |
| Visible but unfocused; focused but hidden; no recent pointer/key activity | Zero new qualified presence in each case. Test every false conjunction branch independently. |
| Tab remains open overnight | No invented presence beyond actual qualified reports; server still ticks. |
| Blur/hidden/settle/close in the middle of an interval | Credit stops at the boundary; missing terminal delivery never grants future attention. |
| Client clock jumps or wakes after suspension | New anchor and fresh snapshot; no huge duration, catch-up call burst, or default mood. |
| Out-of-order or duplicate events | One accepted effect; no skipped committed event due to sequence/commit races. |
| Crash before/after tick commit | Either all state/cursor changes commit or none do; retry cannot double drift or notebook entries. |
| Worker lease split/failover | Fenced writer cannot commit; persisted identities/vectors survive. |
| Two offers to the same bird from different devices | One cooldown reservation wins; no duplicate reaction or curiosity impulse. |
| Offer accepted but response lost | Receipt lookup returns the original outcome/program; no new offer ID is minted automatically. |
| Snapshot older than an event acknowledgment | Acknowledged reaction remains until a covering snapshot; no visual restart or loss. |
| Settle/undo races tick and other owner view | Local reversal works; only the settle influence is removed; other view's presence persists. |
| Stale settings or rename form | 409 with current value; no hidden overwrite or personality mutation. |
| Backend is unavailable beyond the behavior horizon | No invented domain changes or accumulated offline interactions; clear system recovery path. |

Use property-based interleavings around event commit, high-water advancement, interval union and cooldown reservation. Add database-role tests that attempts to update vectors through the web role fail, and DTO tests rejecting raw vector fields in every owner/visitor response except the explicit export.

### 14.3 Auth, privacy, deletion, and visit tests

- Auth links expire at 15 minutes and are single-use under concurrent consumes; invitation links expire after 30 unused days and grant exactly one visit session.
- Link scanners do not consume links; query strings and tokens are redacted at every logging layer.
- Session revocation takes effect on the next request. Email change cannot switch the address before verification or overwrite another account.
- A visitor cannot submit any event type, trigger return-greeting, accrue presence/listen time, mutate names, access the notebook or upgrade a visit cookie to an owner session.
- Identical host/visitor timestamps and revisions produce the same scene/calls/weather; opening a visitor view changes no simulation state.
- Revocation while open, revocation while hidden, grant expiry, network loss and 304 polling all terminate access according to the pull/lease policy.
- No default host notice, badge, email or push occurs on visiting. The optional setting affects only the open account-panel notice.
- Export is a consistent full JSON snapshot with all notebook pages and current vectors, is owner-protected, expires, and contains no tokens or foreign state.
- Recovery on day 29 restores the same identities/state; recovery at/after the deadline fails. Hard deletion covers live rows, cached state, logs, temporary exports, cross-account visit references and key versions.
- Restoring a pre-deletion backup cannot make erased account data decryptable or resurrect a visit capability.
- Automated egress assertions reject emails, IDs, state payloads, interaction history and unapproved dimensions in aggregate telemetry. Test collector/warehouse roles cannot read simulation storage.

### 14.4 Visual, audio, accessibility and voice tests

Record short browser traces for first adoption, ordinary first paint, short return, long return, all five moods, six species, night, rain, seven-bird layouts, and rapid foreground switching. Verify no spinner-to-bird transition, welcome copy, entry replay, synchronized greeting, clipping, offscreen bird, generic status label or hover-only action.

Render grammar instances in an offline WebAudio test context to check envelope bounds, finite values, discontinuities, expected timing and voice-pool cleanup. Listen to real-browser mixes for aliasing, harshness, repeating motifs, recognizability across mood/three-week drift, and chorus masking. Verify listen-in always keeps non-target scheduled calls above zero gain and ramps smoothly. Compare caption facts directly to the post-variation note representation.

Run keyboard-only end-to-end paths through every top-bar surface, every bird, offers, settle/undo, notebook history, authentication, device revocation, invitations, export and deletion recovery. Check focus persistence after snapshots and focus restoration after dialogs. Run automated contrast and semantic checks for day/night/weather, plus actual screen-reader sessions to check prose, cadence, event priority and queue growth.

Review normal motion and reduced motion separately with participants who use the relevant accessibility surfaces. A static replacement scene, technical state-list narration, redundant live call announcements, uncaptioned procedural variation or uncomfortable pose transition blocks release.

Use a product-copy checklist and targeted UI assertions for prohibited surfaces: no “Welcome back,” “days away,” achievements, counters, streaks, guilt, unread badges, or numerical traits. Product prose is lowercase, specific and observational; system errors, auth, account, privacy and accessibility settings use direct normal sentence casing. Check templates and dynamic interpolations, not just hard-coded strings.

### 14.5 Browser and endurance matrix

Support the last two major versions of Chrome, Safari, Firefox and Edge, including mobile Safari and Chrome. Unsupported browsers receive a small matter-of-fact explanation and required browser information; they do not download a heavy legacy compatibility path.

Run fast unit/contract checks on every change. Run representative scene/a11y browser paths on rendering or interaction changes. Run the actual 30-minute memory/frame/audio suite in CI on release candidates and nightly on the current main build, with two-bird and seven-bird fixtures. Repeat broad suites when a new change, failure or unresolved concern warrants it; do not substitute many short runs for the required sustained session.

Required release evidence consists of passing invariant/transaction tests, privacy/authorization/deletion checks, real-device first-bird traces, 30-minute stable-resource traces, cross-browser coverage, caption-to-call consistency, and recorded visual/audio/accessibility review findings resolved against the current build.

## 15. Delivery sequence, ownership, and rollout

### 15.1 Build sequence

| Increment | Responsible capability | Deliverable and dependencies | Exit gate |
| --- | --- | --- | --- |
| 0 — Contracts and aliveness prototype | Product design, audio, accessibility, platform | Lock DTO/event schemas and decisions; author two silhouettes/voice grammars; build the tiny SSR/SVG sampling experiment, naturalist narration and reduced-motion prototype using synthetic state. | First actual bird under 500 ms on baseline hardware/network; distinguishable calls; narration and reduced-motion experience accepted before a large app framework is committed. |
| 1 — Durable server core | Backend/simulation | Account/aviary/bird storage, persisted vectors/filter state, minute scheduler, ordered log, interval union, deterministic moods/programs and migration discipline. | Absence/monotonicity/identity tests and crash/retry/worker-fencing tests pass. |
| 2 — Complete two-bird interaction slice | Client, simulation, audio, accessibility | Authenticated bootstrap, current-motion rendering, greetings, listen-in, offers, settle, snapshot refresh, all accessible equivalents and silence/caption fallback. Depends on increments 0–1 contracts. | Real two-device flow; honest presence; no minute-long offer delay; no first-frame or keyboard regression. |
| 3 — Living history and full species set | Simulation, content, design/audio | Sparse notebook, weather/day-night refinement, six complete species, naming and age-based adoption opportunities; tune synthetic week/three-week trajectories. | Supported facts and sparse prose; stable voice identity; calm night behavior; no progression or obligation surfaces. |
| 4 — Account lifecycle and private visits | Identity/privacy, backend, client | Email changes, device revocation, export, soft/hard deletion, backup erasure, one-time invitations, visit log and bounded optional notices. Build alongside prior increments only behind disabled release gates. | Permission matrix, revoke-on-pull, privacy egress and deletion/restore tests all pass. |
| 5 — Seven-bird capacity and endurance | Client/audio/performance, infrastructure | Responsive layout at cap, repeated-species voice separation, bounded queues/nodes, full browser matrix and server capacity testing. | Seven birds meet memory/frame/mix/caption budgets for 30 minutes; all supported accessible paths pass. |
| 6 — Controlled release | Release/operations with product and accessibility review | Staged owner-account cohorts, operational dashboards, support/system copy, expiry/deletion schedules, rollback rehearsal. | Every complete-v1 gate is green; no accessibility/privacy capability is postponed to a later release. |

These are engineering responsibilities, not additional runtime agents or outputs for the current planning run. Keep accessible narration, reduced motion and captions in each vertical slice; they are not final-stage retrofits.

### 15.2 Rollout mechanics

Use deterministic cohorts based on a synthetic account UUID hash, never email, visit frequency or interaction score. Start with synthetic/internal fixtures, then an invitation-only owner pilot, then approximately 5%, 25% and full availability of the intended serving cohort. Hold each external step for at least 48 hours of healthy operational metrics and the relevant manual review; use synthetic time advancement to cover months of adoption/drift, since a 48-hour pilot cannot establish those behaviors.

Every new real account still starts with two birds. Separately gate maximum adoption eligibility at three, then five, then seven after corresponding render/audio/capacity evidence passes. Use aged synthetic aviaries to exercise those sizes immediately. Real accounts gain opportunities only when their actual age qualifies; the rollout must not manufacture early age, reward “active testers,” or force additional birds into an aviary.

The complete v1 must support the seven-bird cap even if early production accounts have not aged enough to reach it. Never lower a live account's existing bird count during a rollout pause. Stop new adoption opportunities temporarily if capacity is unhealthy, while preserving every already-adopted identity.

From the first pilot, run aggregate first-bird/render/audio/tick/authorization instrumentation, transactional mail error monitoring, and deletion/expiry schedules. Do not add interaction analytics to decide which features are “engaging.” Assess drift, audio identity and felt aliveness with synthetic longitudinal fixtures and explicit design review.

### 15.3 Deployment and rollback

Version database schemas, engine math, behavior programs, call grammars and projection DTOs separately. Use additive schema migrations and an overlap period where the server can serve the current and immediately previous client DTO. Keep art and grammar versions required by living birds available; no deployment reassigns their voices.

Rehearse rolling a worker/client version back while events continue arriving. Keep the persisted canonical vector and cursor; do not roll them back to an older backup just because code was reverted. Old workers must refuse an unsupported state version rather than reseeding it. Deploy a compatible repair or temporarily pause the affected worker if necessary, preserving pending events.

Operational switches may stop new invitations/adoptions, select the last working renderer/grammar interpreter, or force silence-with-captions for a broken browser audio path. They must not reset birds, delete drift, enable recorded calls, or disable accessibility to keep the normal path running. If simulation is paused for correctness, retain events and catch up through the supported server path after repair; do not claim normal continuity while the system is in an outage.

Treat a privacy authorization defect, any lost/negative personality delta, identity replacement, unrecoverable growth in memory, persistent first-bird budget failure, or serious accessibility regression as a release stop. Publish operational incident copy in the system register only when users need it to understand blocked access or recovery.

## 16. Risks and concrete response plans

| Risk | Early evidence | Mitigation / release response |
| --- | --- | --- |
| Drift feels instantaneous or imperceptible | Synthetic week/three-week fixtures miss the numerical/perceptual gap; button-heavy fixtures dominate. | Adjust the low-pass/weight constants and expressive mapping together; keep presence dominant and deltas nonnegative; repeat perceptual comparison before rolling the engine version. |
| Absence quietly becomes punishment | Inactivity tests lower a trait, increase wariness from absence, or show distress/guilt copy. | Remove negative pathways; distinguish transient attentive expression from stored personality; block the release. |
| Multi-device attention inflates drift | Overlap fixtures exceed wall-clock qualified time or duplicate listen credit. | Union normalized intervals, split concurrent target exposure, retain rejection watermarks, and property-test retries/out-of-order delivery. |
| Commit races lose or double drift | Cursor/vector disagree after faults, or late low-sequence events disappear. | Serialize aggregate commit order, atomically persist cursor and deltas, fence workers, and verify failure recovery on the real database. |
| Immediate actions drift away from canonical state | Offer restarts at the next snapshot or different devices choose different outcomes. | Persist immutable server-authored reaction envelopes; reconcile by event/program ID and carry endpoints into the tick. |
| Voice uncanniness or loss of identity | Users hear exact repeats, harsh tones, or cannot recognize a bird after a mood/drift change. | Refine motif variation and timbre limits; preserve signature versions; reduce simultaneous masking; do not replace with recordings. |
| Seven birds blur into ambience or become uncaptionable | Identity listening tests fail; mobile captions overlap; voice budget overruns. | Improve scheduling/mix and within-species fingerprints; hold larger-count rollout until tested, without removing existing birds. |
| First paint feels like a loading app | Traces exceed 500 ms or hydration starts from a static neutral pose. | Reduce critical bytes/round trips, move authorized delivery closer, and adopt server-sampled SVG; never add spinner/entry choreography. |
| Server tick cost scales with all accounts | Backlog/p99 delay grows as inactive accounts accumulate. | Size from total aviaries, stagger ticks, batch acquisition and partition workers, benchmark database writes, pause enrollment before overload. |
| Long-session resource leak | Retained heap, DOM nodes, audio nodes or callbacks climb across repeated interactions. | Enforce pools/lifetimes, dispose all view resources, and require a passing 30-minute CI rerun for the repair. |
| Accessibility becomes a state-reporting fallback | Screen-reader queue floods, captions mismatch calls, reduced motion looks paused. | Keep shared observation/call programs; slow the prose cadence; design distinct still-pose transitions; require user experience review, not only markup checks. |
| Canonical timezone surprises travelers | Devices disagree about morning or silently reset the schedule. | Store one timezone, offer an explicit system setting, blend changes, test DST; never derive separate device moods. |
| Presence undercounts still/touch/assistive users | Defined activity signal never occurs on a supported input path despite visible focus. | Test real input methods and calibrate the few-minute window conservatively; preserve the specified conjunction, document its approximation, and do not invent click/visibility-only credit. |
| Notebook becomes a feed or reveals owner behavior | Entries repeat every session or describe visit habits. | Enforce fact eligibility, sparsity and editorial tests; retain bird observations only and remove behavior-derived rules. |
| Invite accidentally becomes co-presence | Visitor generates events, owner sees an arrival effect, or an endpoint trusts a visit cookie as owner auth. | Separate grant scope and client controller, deny all mutation routes, compare host/visitor snapshots and simulation before/after visitor sessions. |
| Privacy leakage through “helpful” diagnostics | Request bodies, email, vectors or event IDs reach analytics/error tooling. | Fixed schemas, redaction at every boundary, no DB-to-warehouse path, and egress/role tests from day one. |
| Deletion resurrected by backup or cache | Restored state remains readable after the deadline. | Account-key erasure, cache fencing, deletion-aware restore, and an actual pre-deletion-backup restore test before launch. |
| Spec ambiguities become inconsistent implementations | Export values leak into UI, settle gains a fifth icon, or optional notices turn into push mail. | Treat section 1.3 as the common contract and include these exact boundaries in API, UI and release checks. |

## 17. Definition of complete v1

The team can ship when a new owner signs in, meets and names two distinct birds, sees current motion within the performance budget, and receives one varied noticing gesture; when a second device reads the same durable birds and moods; and when days of server operation produce slow, nonnegative, identity-preserving evolution from honest presence.

The same release must deliver smooth listen-in, meaningful offers and optional settle, sparse readable notebook history, working age-only adoption through seven birds, six coherent species including active nighttime life, full audio/caption/narration/reduced-motion experiences, quiet read-only invitations, revocation, verified account changes, complete export/deletion, and passing sustained performance/privacy/correctness gates.

No final acceptance substitutes aggregate engagement, a feature count, or a successful build for the required behavior. The product must remain a small private place whose birds notice the owner, with all system obligations handled plainly outside that scene.
