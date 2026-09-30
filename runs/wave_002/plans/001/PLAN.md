# Pocket Aviary — v1 implementation plan

## 1. Delivery contract and scope

Build a browser-only aviary whose continuity belongs to the birds, not to an open tab. Each account owns exactly one aviary. The server is the only authority for its birds, personality, mood, weather, behavioral schedules, and notebook. A client displays that state, interpolates its presentation, synthesizes scheduled calls, and submits observations of user interactions. No client computes persistent personality updates.

V1 includes email magic-link accounts, two server-selected starter birds, a coherent six-species pool, renameable birds with permanent identities, age-based optional adoption up to seven birds, return-greetings, listen-in, three offers, settle, precise presence accounting, a sparse read-only field notebook, multi-device snapshots, optional individual visit invitations, account export/deletion/session management, narration, call captions, and a separately designed reduced-motion renderer. Accessibility and performance are launch requirements, not follow-up work.

Exclude native apps, payments, passwords/SSO, shared or multiple aviaries, configurable scenes, user-directed perch placement, species catalogs/rarity, hunger, feeding requirements, death, distress from absence, numeric trait displays, achievements, streaks, scores, badges, visit-frequency surfaces, public discovery, profiles, follows, comments, chat, co-presence, leaderboards, and general engagement notifications. Do not build underlying cross-account bird statistics that could later become those surfaces.

Product acceptance centers on five observable properties:

1. The first normal-session frame is an aviary already mid-action; a bird notices an owner within one to two seconds.
2. Watching without clicking has a real effect over weeks, while unattended/background tabs do not.
3. Returning after absence preserves bird identity and personality, with ordinary ambient life and no guilt cues.
4. Every device reads one canonical record; no action can overwrite another device's personality drift.
5. Audio-off, screen-reader, and reduced-motion users receive the same specific, living aviary through their respective surfaces.

### Explicit implementation decisions where the PRDs leave gaps or conflict

- Use a 60-second baseline tick, a five-minute recent-activity window, 15-second visible snapshot pulls, and 15-second presence heartbeats. All are versioned configuration, with changes gated by synthetic calibration rather than private interaction analytics.
- Persist an account IANA timezone initialized from the first browser. All devices and visitors use it. Travel does not silently let two devices alternately redefine morning; a matter-of-fact account setting changes it explicitly.
- Use five mood states: wary, content, curious, drowsy, alert. Sleeping/settled are poses and lighting states, not extra numerical needs or health states.
- Age eligibility begins at 90, 180, 270, 365, and 540 elapsed days for birds three through seven. Eligibility persists without expiry. Adoption is optional and not earned by attention.
- The layout's exhaustive four-icon list omits settle, while the interaction and accessibility files require a top-bar settle control. Keep four primary icons; put a clearly labeled settle action in the top-bar offer/actions popover, also directly reachable by a keyboard shortcut. It remains a top-bar action without an extra scene control.
- Focus on a bird engages listen-in, as the interaction file requires. Enter ensures engagement without toggling it off; Escape disengages while retaining keyboard focus. Moving focus to another bird starts its listen-in. This resolves the accessibility file's Enter wording without making keyboard navigation inconsistent.
- The export specification expressly includes current personality vectors, while the engine says numbers are never exposed. Adopt the narrow account-data-export exception: the downloaded machine-readable JSON includes the required vectors; no product surface, normal API snapshot, ARIA label, debug toggle, or rendered export preview displays them. These two literal requirements cannot both hold; this plan prioritizes the explicit export schema solely for portability, and records the exception for product review.
- The social specification expressly allows opt-in visit notifications despite the general no-notification rule. Implement only an account-settings opt-in email for visits, initially off, without onboarding prompts, in-product badges, or push. This is a narrow specific exception; magic-link, invitation, and requested export emails are transactional.
- Browsers can prohibit audio before a user gesture. Attempt playback only when allowed; otherwise start the living visual scene and matching captions immediately, then unlock procedural audio after an explicit user gesture. Never circumvent autoplay restrictions or substitute recorded sound. A fresh restricted browser cannot literally have audible calls on its first frame.

## 2. Architecture and responsibility boundaries

Use a TypeScript web application and server, PostgreSQL for transactional canonical state and event storage, a minute scheduler plus bounded worker pool, an authenticated edge delivery layer, and a transactional-email adapter. Start with one application service and worker deployment from a shared domain package rather than numerous independently deployed microservices. Provision regions close to the target launch audience and leave account-UUID partition routing behind a small storage abstraction.

Separate these modules with typed interfaces:

| Module | Owns | Must not do |
| --- | --- | --- |
| Identity/account service | Email verification, sessions, timezone/settings, export/deletion | Mutate vectors or broadcast behavioral data |
| Interaction gateway | Authorization, validation, ordered event append, receipts, cooldown reservations | Accept absolute personality or client-generated canonical moods |
| Simulation worker | Vector deltas, mood, perches, action/call schedules, greeting/offer decisions, sparse observations | Depend on an attached client or rebuild vectors from history |
| Snapshot projector | Read committed state, produce bounded render descriptions and read-only visitor projection | Expose raw vectors or private event logs |
| Notebook module | Structured observation selection and naturalist prose | Record visit streaks or produce generic session logs |
| Browser renderer | Scene composition, interpolation, focus hit targets, local ornament motion | Choose persistent perches or evolve birds |
| Browser audio/accessibility | Synthesize call descriptions, captions, narration, mix envelopes | Substitute sound recordings or expose traits |
| Operational telemetry | Allowlisted timings, counts, error classes | Query simulation tables or collect bird/account interaction payloads |

Use common versioned contracts for snapshots, events, grammar descriptors, and semantic observations. Share geometry and grammar interpretation between initial server-rendered SVG and browser rendering. Package persistent simulation logic separately from renderer code so it cannot accidentally be bundled into the client.

The browser has a thin DOM application shell and a Canvas2D scene renderer. Inline SVG supplies the first visible birds before the full renderer is ready. Small SVG/path assets and procedural pose transforms cover the six species. Canvas avoids an expensive changing DOM tree; DOM buttons aligned with bird hit regions supply accessible interaction. Implement scene and audio adapters so missing audio never blocks rendering. No external 3D engine, recorded-audio catalog, runtime language-model dependency, or general-purpose analytics SDK is needed.

Each canonical change commits state and a new monotonically increasing aviary revision. Schedule cache-publication work in the same transaction using an outbox. The database remains the authority if an edge cache or outbox is delayed. A user event may request an expedited server tick; this is the same simulation transaction and sole vector writer, not a second mutation path. Coalesce requests per aviary and limit them to at most one expedited pass per second. Ordinary unattended aviaries still advance every minute.

## 3. Persistent data model

Use UUIDs for account, aviary, bird, invitation, session, and observation identities. Internal queues, storage keys, and logs reference UUIDs, never email. Every owned row has an account ownership path and foreign-key constraints. UTC timestamps encode instants; an IANA timezone supplies civil-time context.

| Record | Required fields and invariants |
| --- | --- |
| Account | UUID; encrypted verified email stored here only; keyed lookup digest for login uniqueness; encrypted pending new email until verification; created_at; timezone; status active/deleting; deletion_due_at; settings_revision; settings including audio, captions, motion, narration, visit-email opt-in |
| IdentityContact | UUID and encrypted recipient email for a named invitee who has no account; restricted to identity/email module; invitations reference its UUID; merge into Account identity if that address later signs up |
| MagicLink | Token digest, account/contact reference, purpose, created_at, expires_at = +15 minutes, consumed_at; single-use atomic consumption |
| DeviceSession | Session UUID, account UUID, token digest, creation/expiry/last-seen timestamps, user-readable device description, revoked_at; no bird data |
| Aviary | Unique account UUID, UUID, created_at, canonical revision, last_tick_at, event cursor, simulation version/RNG state, current weather and expiry, owner arrival state, presence aggregation cursor, behavioral horizon |
| Bird | Stable UUID, aviary UUID, species/version, mutable display name, adopted_at, immutable signature seed, persisted five-component vector, low-pass filter state, mood and mood_since, transition deadline, perch zone/slot, pose/motion phase, active transition, call sequence, offer cooldown_until |
| InteractionEvent | Aviary UUID, server sequence, client event UUID, session/presence-lease UUID, type, optional bird UUID, received_at, validated bounded timing fields, payload version; immutable after append; unique idempotency key |
| PresenceInterval | Account/aviary UUID, device lease, bounded start/end, verification flags, settlement boundary; internal simulation input, never a product history surface |
| Call/ActionSchedule | Aviary revision, stable action/call IDs, bird IDs, absolute time range, grammar/version/seed and expanded call descriptor, semantic facts, expiry; bounded rolling horizon |
| NotebookEntry | UUID, aviary UUID, occurred_at, created_at, bird identity references plus name-at-observation, observation kind, prose/template version, immutable prose; no edits/deletes except full account deletion |
| ObservationMemory | Last candidate time, recent themes, greeting order summaries and quiet/mood-duration summaries needed for that account's prose; compact and private |
| AdoptionEligibility | Aviary UUID, age milestone, available_at, accepted_at, offered species seed; not activity-derived |
| Invitation | UUID, host account UUID, recipient identity UUID, token digest, issued_at, unused_expires_at = +30 days, consumed_at, revoked_at, active visitor-session reference |
| VisitSession/VisitLog | Invitation/host/visitor identity UUIDs, token digest, started_at, last pull, ended_at, approximate duration, revoked/expired state; private account settings only |
| OutboxJob | UUID, owner UUID, type, payload reference, dedupe key, attempts, due_at; contains identifiers rather than plain email or raw vectors |
| ExportJob | UUID, owner UUID, status, encrypted object reference, expiry, token digest; deletion cascades and invalidates downloads |

Represent traits as double-precision scalars in [0,1], with seeded starter values in roughly [0.25,0.65]. Store initial seed values for controlled migrations and diagnostics, but never use those to replace persisted current values. Add database range checks; lock the aviary when inserting birds to enforce the seven-bird cap. Stable identity and signature seeds survive renames, migrations, software rollback, timezone changes, and offer responses. Do not provide bird reset/removal/regeneration routes in v1.

An invite must retain a recipient address to deliver mail and show the host who was invited. Keep that address in the restricted identity/contact store, not on invitation/event/log rows. This applies the account-email single-storage rule to invitees without creating unsolicited aviary accounts. The email adapter resolves UUIDs only at delivery and disables body/address logging.

Keep events long enough for consumption and a bounded retry/debug horizon, initially seven days after consumption. Compact into private rolling input buckets and observation memory; retain only what continuing simulation needs. Old notebook entries remain available indefinitely. The persisted vector is never reconstructed from event retention or notebook history. Backups include current vectors and tick cursors together.

## 4. HTTP contracts and security

Use same-origin HTTPS JSON endpoints. Owner authentication is a secure, HttpOnly, SameSite cookie tied to a revocable device session. Validate Origin and CSRF tokens for mutations. Visitor cookies have a distinct scope and cannot authorize owner endpoints. Bound names to 40 grapheme clusters, permit Unicode, trim unusable whitespace, and render as text rather than HTML. Bound JSON bodies, reject unknown event types and all attempted vector fields, and authorize bird IDs through aviary ownership.

| Endpoint | Contract |
| --- | --- |
| POST /auth/links | Email input; generic 202 to avoid address enumeration; 15-minute single-use link; initial per-email 5 requests/hour and per-IP abuse ceiling, with Retry-After |
| POST /auth/consume | Exchange token after explicit landing-page confirmation; atomic consume and issue device cookie; safe against mail scanners consuming a GET |
| POST /auth/logout | Revoke current session and clear cookie |
| GET /account; PATCH /account/settings | Matter-of-fact settings/session data; PATCH uses expected settings_revision and field updates |
| POST /account/email-change; POST /account/email-change/verify | Verify new address with a 15-minute single-use token; retain old login until verification, prevent duplicate verified addresses |
| GET /account/sessions; DELETE /account/sessions/{id} | List owned devices and revoke immediately; revoke current device also clears cookie |
| POST /adoption/start | Idempotently create/reserve the two server-selected starter birds and suggested names; no species catalog |
| POST /adoption/complete | Names for reserved birds; commit first encounter; retry must not create additional starters |
| PATCH /birds/{id}/name | Expected name revision and name only; stale concurrent rename returns 409 with latest value |
| GET /aviary/snapshot | Authorized committed snapshot, server time, revision and ETag; conditional pulls still check session validity |
| POST /aviary/events | Up to 20 validated events with unique IDs and client sequence; return per-event accepted/duplicate/rejected status, server sequence, receipt time, and any immediate server-produced render cues |
| POST /aviary/arrivals | Idempotent owner arrival/return event, server absence interval, greeting cue and authoritative revision; never used by visitors |
| GET /aviary/notebook?before={cursor}&limit=30 | Read-only keyset pagination newest first; oldest entries remain addressable; no write API |
| GET /adoption/available; POST /adoption/accept | Available age-based offer and names; locked eligibility/cap check; no progress meter or countdown |
| POST /account/invitations | Explicit email input, host-owned single-use invitation; send transactional message; social otherwise remains off |
| GET /account/visits; DELETE /account/invitations/{id} | Outstanding/active invites and historical visits; atomic revocation; no toast/badge |
| POST /visits/consume | Exchange unexpired/unrevoked token once into a bounded visitor cookie; safe GET landing page alone does not consume |
| GET /visits/snapshot | Same canonical scene/call projection, host timezone, no private settings/notebook/event data; validate revocation before every response including 304 |
| POST /account/exports; GET /account/exports/{token} | Authenticated request, asynchronous current-state JSON, expiring download link emailed only to verified address |
| POST /account/deletion; POST /account/recovery | Mark deletion and recover within 30 days; deletion-state pages offer "I changed my mind" |

A snapshot contains schema/simulation/grammar versions, aviary revision, server_now, generated_at, timezone/light description, active weather, bird IDs/names/species, normalized perch geometry, current poses and transition timestamps, bounded expression parameters, timed action/call descriptors, and owner-only pending cues. It omits vectors, filter states, presence totals, raw interactions, and numeric mood intensities. Mood enum may be internal to renderer contracts but never appears as a visual label or raw narrated state.

Event types include presence segment, listen-in start/end, offer request, settle, settle undo, and explicit re-engagement. Clients submit intent and measured intervals, not outcomes or absolute personality values. Each request returns a bounded receipt. Cooldown, eligible receiving bird, acceptance, and reactions are server decisions. Client optimistic feedback can open a popover or show the selected offer artifact, but cannot claim a bird accepted or persist a mood before its server cue.

Use a 24-hour maximum visitor session as the initial interpretation of a one-time visit link. After consumption the link cannot start another browser session; unused links expire at 30 days. Visitors can resume within their session cookie window. Revoke authorization immediately in storage; an open visible client learns on its next pull, bounded by 15 seconds. Hide/show and network reconnection always reauthorize before resuming the view. Do not let a service worker serve an authorized visit offline after its authorization lease expires.

Errors return stable codes and plain messages: invalid/expired link, session expired, unavailable visit, stale settings, cooldown, unavailable state. Show errors in the relevant system panel, not naturalist prose or floating celebration/announcement UI. A failed optional interaction leaves the birds present and offers a clear retry surface.

## 5. Canonical simulation and deterministic continuity

### Tick execution and ordering

Schedule each active aviary for a minute tick regardless of connections, using a next_due_at index and distributed worker claims. Soft-deleted aviaries are suspended with their records intact for recovery. Use row locks and a transaction to enforce a single writer; event appenders take the same short aviary lock to allocate monotonically increasing server sequence numbers. Do not assume a database-generated sequence alone guarantees commit order.

A pass:

1. Lock aviary and birds; read last_tick_at, processed event cursor, persisted vectors/filter states, simulation version and RNG state.
2. Establish a committed event high-water mark. Read unconsumed events in server sequence order. Validate intervals and deduplicate overlap before producing inputs.
3. Advance elapsed minute boundaries in chronological order, splitting at event times, day-cycle boundaries, and weather/action deadlines. Late events use bounded server-validated intervals; never roll an entire aviary backwards to client time.
4. Compute nonnegative personality deltas from accepted inputs. Apply them additively to the locked current vector; never replace a vector from a client snapshot or replayed total.
5. Update mood, ambient activity without needs or distress, perch targets, interaction reactions, and bounded future call/action schedules. Derive observation candidates.
6. Persist vectors, moods, schedules, weather, observation state, tick time, cursor, revision, and outbox together. Commit once, then publish snapshots.

A worker crash before commit makes the pass unobservable; a crash after commit cannot reapply its consumed events. Outbox retries publish the same committed revision. The state/cursor/receipt transaction is the exactly-once effect boundary even when transport and jobs deliver at least once. Expose queue-age and compute-time histograms separately.

Expedited passes process accepted greetings and offers promptly without inventing elapsed drift time. If only 300ms elapsed, integrate that duration; never award another minute of drift because a user clicked. Settle and offer cues can therefore react within hundreds of milliseconds without waiting a minute. Only this worker code and database role can update personality columns.

During scheduler outages, perform chronological bounded catch-up from persisted current state, filter state, input buckets, and event cursor. Never reseed or rebuild vectors. Process at most one day per job chunk and retain overdue priority until current. Normal continuous ticking is still required; catch-up is resilience, not a lazy-on-open substitute. If authoritative state is temporarily behind, retain its timestamp and allow presentation interpolation only within the authorized schedule horizon; report an extended outage matter-of-factly.

### Monotonic drift function

For bird b and trait j, let x be the persisted trait. Maintain a rolling 24-hour presence total P formed from the union of eligible owner-device intervals. Set p = min(P / 1200 seconds, 1). Maintain per-bird eligible listen-in duration L with l = min(L / 600 seconds, 1), and eligible offer signals capped at three credited offers per rolling day. An offer near a bird supplies boldness input; actual acceptance supplies curiosity input. Disregarded offers do not manufacture acceptance.

Set a nonnegative input u_j = 0.85 p + 0.10 l_j + 0.05 o_j. Apply l only to warmth/vocal frequency and o only to boldness/curiosity; other terms are zero. Thus watching remains sufficient and dominant, and repeated offers cannot dominate. Initialize filter z_j to zero. For elapsed days dt, update z_new = z_old exp(-dt/3) + u_j(1-exp(-dt/3)). Integrate delta_j = k_j (1-x_j) integral(z_j dt), using the exact exponential integral for a constant input over the substep. Start k_j at 0.005/day, clamp x+delta to 1, and persist both x and z. Split steps wherever input changes rather than depending on worker invocation frequency.

No negative term exists. Filter signals decay after absence, but accumulated traits never do. A short positive tail after departure comes from previously earned attention, as the server-continuity requirement describes. Muting audio neither subtracts vocal frequency nor makes silent users' presence worth less. Settle ends input and nudges mood quietness; it earns no extra drift. Do not treat audio errors as rejection by the user.

Use seeded synthetic accounts to calibrate these starting values. For initial x near 0.4, regular 20-minute daily visits should produce approximately 0.01 numerical change after a week and 0.04–0.06 after three weeks, depending on secondary input. The exact visible mapping needs review: smoothly increase approach frequency, call participation, and feather detail without threshold jumps. A single normal session must not create an observable trait change. Saturation slows further drift instead of introducing levels, resets, or artificial new progression tracks.

Recent owner familiarity may ease back to a neutral ambient baseline after absence, affecting greeting frequency and session expressiveness only. It cannot lower a trait, cause distress, reduce the bird's established plumage, or make the bird progressively mistrust the user. Keep a minimum ambient call/activity rate appropriate to species and time of day. Wary mood comes from short-lived ambient/social stimuli, not lack of visits.

### Mood, perches, weather, and bird-to-bird behavior

Use a bounded probabilistic state machine with mood dwell times, personality-shaped transition weights, and a persisted next decision deadline. Re-evaluate approximately every 5–20 minutes and on salient accepted events. Daily-ish reset means the influence of old session inputs dissipates over roughly 18–30 hours and a fresh baseline is sampled during the local dawn period. It never means reset on page load or force all birds to neutral at midnight.

Choose perch zone by weighted scores: boldness favors front, wary favors back, social warmth favors proximity to other birds, drowsy favors sheltered slots. Reserve normalized perch slots atomically so two birds do not clip. Persist transition source/target/times and interpolate paths in the client. Preserve already-issued future action/call IDs and timing when extending the schedule horizon; do not regenerate overlapping schedules on each tick. Explicit interaction reactions may supersede a conflicting future action with a revisioned cancellation that all clients honor. Keep all flight control points within viewport-safe bounds. The user cannot send a bird to a perch.

Generate rain initially about three times weekly for 2–6 minutes and occasional soft wind using a seeded event process. Persist weather start/end and short-lived behavioral modifiers; rain temporarily attenuates call activity, not the vocal-frequency trait. Wind produces small species/personality-dependent alert/wary shifts. No thunder, urgent weather alerts, or real-weather dependency. Leaves and feathers are client ornaments, not database entities.

Scheduled calls can elicit another bird's reply after a variable delay, with social warmth/vocal frequency weighting. Alarm-type motifs may briefly influence nearby mood; cap propagation to one reaction hop and bound wary duration so an aviary cannot get trapped in escalating alarms. Chorus windows emerge from overlapping calls and responses, not a precomposed chorus track. At night most birds become drowsy/resting while the nightjar-like species retains appropriate quiet activity.

### Return-greeting

Record owner arrival on navigation, becoming visible, or recovering from a long frame gap. It is distinct from eligibility for presence: greeting does not require recent pointer activity. Coalesce duplicate arrival IDs and simultaneous device returns through an account-level short debounce so the same arrival cannot produce a fanfare.

Compute absence from server-recognized owner arrival/last active state, never from a displayed "days away" number. Weight the primary bird by boldness, warmth, mood, recent greeting history, and species. Select exactly one primary response within one to two seconds; choose head orientation, glance duration, approach distance, note count, contour, and pause continuously from a fresh event seed. A short absence usually yields a small glance; a longer one can yield reorientation or a fuller call. Do not replay fixed greeting clips or rotate a tiny canned list.

Other birds may respond later at randomized staggered offsets, but not in a synchronized arrival chorus. Avoid repeating the recent parameter tuple, while accepting that species identity must remain familiar. Return the cue with its server start time so every owner device can interpret the same canonical occurrence. Accessibility narration describes the bird noticing promptly; there is no welcome text, return banner, modal, calendar, or absence-length copy.

### Offers, listen-in, and settle

Offers come from the top bar and concern the aviary rather than clicking a bird to feed it. Server selection weights proximity, mood, curiosity, and availability. A seed may prompt approach, delayed inspection, or quiet disinterest. A still pool may prompt drinking, bathing, or watching. A library song fragment is a synthesized melodic phrase; responses may join, become quiet, or call against it. Specify three to five small symbolic song motifs, never uploaded or recorded audio.

Start a per-bird three-minute cooldown on a valid offer opportunity, even when the bird declines, so attempts cannot be mashed for boldness input. Request-time locked reservations prevent concurrent-device offers bypassing it. A tick converts a reserved intent into a deterministic outcome. A calm disabled option can say the bird is taking its time; no countdown, failure punishment, or scarcity rewards. Cap accepted signal credits separately from presentation cooldowns.

Listen-in mix is local to the device; its start/end intents supply targeted attention only within eligible presence intervals. Different devices can listen to different birds. Union duplicate same-bird listening and cap total targeted attention to the owner-presence union so two simultaneous devices do not double drift. Switching focus closes the previous lease and opens a new one.

Settle ends that device's presence immediately, emits a small server-authored quieting cue, and ramps its evening-light/audio overlay over about four seconds. Another actively used owner device need not be forced out of its session; the shared birds still receive the same small mood cue. A visitor sees shared canonical mood, not a personalized session overlay. Within five seconds, a click anywhere in the scene reverses the overlay; provide an equivalent keyboard undo. After that, deliberate scene activation or an interaction re-engages. Ignore passive pointer motion as re-engagement so settle remains settled. Undo resumes eligibility prospectively; it does not credit the settled interval. Tab-close ends presence just as validly, with no lost ritual bonus or recovery prompt.

## 6. Presence and synchronization correctness

The client eligibility predicate is exactly: document visible AND document has focus AND a qualifying pointermove or keypress within the preceding five minutes AND not settled. Use pointermove and keyboard press listeners; use keydown to normalize an actual keyboard press on browsers where the legacy keypress event omits navigation keys, without counting focus, polling, or synthetic application events as activity. Do not count clicks, document load, audio playback, or an open tab alone. Touch pointer movement qualifies as pointermove; no mobile shortcut weakens the predicate.

Track interval boundaries with a monotonic clock and translate using server clock offset. At each heartbeat submit only the eligible seconds since the previous heartbeat, the recent qualifying-activity age, and state boundaries, never raw keys, coordinates, or pointer traces. A false predicate closes the interval immediately. Send a best-effort final segment on blur, hide, settle, and page exit, but correctness cannot rely on an unload request reaching the server.

Each segment covers at most 15 seconds. Validate it against server-issued lease bounds and receipt time, with an initial five-second transport allowance. Require adjacent live heartbeats to close normal intervals; expire leases after 30 seconds. Do not award the entire gap if a suspended device reconnects. Drop delayed offline presence and clearly old offer/settle requests rather than backfilling hours of unverifiable attention. This may conservatively lose a few seconds during a disconnect; it must never credit unattended hours. Browser attestations cannot prove that a human is watching, but they enforce the specified honest-client contract without invasive tracking.

The server merges eligible interval unions across devices. Presence counts once per account-time interval, then supplies all birds' presence input. Listen-in time is a subset of that union. Offer spam, duplicate network receipts, background audio, visits, and multiple signed-in tabs cannot increase counted presence.

Clients pull snapshots on open, hidden-to-visible change, online reconnection, a render gap longer than two seconds, and every 15 seconds while visible. Hidden clients stop requestAnimationFrame, ornaments, audio scheduling, and presence. The server keeps ticking. A visible but unfocused client may continue rendering while crediting no presence. On resume, discard expired local schedules and old optimistic intents and request current state; do not render an hours-long backlog of calls or flights.

Maintain server-clock offset using request midpoint measurements and smooth it to avoid animation jumps. Ignore snapshots older than the highest accepted revision, including out-of-order responses from different edges. Cancel redundant pulls. Interpolate between timed poses and perch transitions; interpolation is presentation, not a client simulation tick. Do not extrapolate beyond the issued behavioral horizon, initially 90 seconds. If a fresh schedule is unavailable, bounded calm pose variation can continue, but do not invent canonical moves, greetings, weather, offers, or personality changes.

Retry transient event failures using the same event UUID. Keep a small in-memory pending queue, maximum 50 intents with a two-minute TTL. Do not persist a long offline action queue. Accepted duplicates return their original receipt. Settings/name conflicts return latest revisions for explicit retry; behavioral events append and never replace state. No personality endpoint accepts PUT/PATCH, and only the simulation database role can modify its vector columns.

Initial HTML carries an authenticated snapshot fetched from the primary/cache at the edge. Public CDN caching serves immutable shell/assets only. Private snapshot cache entries are keyed by aviary UUID, projection role, schema version, and canonical revision, with an authenticated lookup and a short maximum age. Never cache a personalized HTML response under a shared URL key. Normal snapshot polling uses a strong primary revision check rather than trusting cache age. Cache failures fall back to database; publication rejects revision regressions. Session/visit revocation is checked before accessing any cached projection.

## 7. Field notebook and prose generation

Use a deterministic naturalist language module over structured, factual observations, with authored compositional templates and constrained variation. Do not send private bird events to an external language model. Observation inputs include bird name/species/pose, sustained mood-shaped behavior, call shape, time-of-day, greeting order, weather, and bounded historical comparisons. A statement such as "first time this week" is valid only if that account's compact observation memory supports it.

Choose approximately one ordinary entry per three active days, with an independent rarity filter and a 48-hour ordinary-entry gap. Noteworthy moments can add an entry sooner, with a six-hour exceptional gap and a rolling cap of three entries per seven days as an initial tuning guard. Do not create one entry per session, one per offer, or a constant background feed. Inactive aviaries can accumulate occasional meaningful observations, but no repeated neglect summaries or user-frequency judgments.

Avoid generic messages and numeric trait narration. Compose lowercase, present-tense, bird-specific prose, for example a description of a named bird watching rain from the back perch. Distinguish the aviary's quietness from a judgment about absence. The observer describes birds, not user effort, attendance, or compliance.

Persist finalized prose with observation-time names, allowing older notes to remain authentic after a rename. Keep stable bird references internally for consistency. Notebook browsing is a lazy-loaded, read-only panel with keyset pagination and virtualized entries. Remove offscreen DOM and listeners; bound the page cache to five pages while allowing older pages to be re-fetched indefinitely. No edit, delete, annotate, user visit-log export, or streak decoration is added.

## 8. Rendering pipeline and visual interaction

### Critical navigation path

Authenticate at the edge, obtain a small committed render snapshot, and include it with HTML and critical inline scene styles/SVG. Draw birds using their scheduled phase at server_now rather than the start of a motion clip. Inline a tiny critical motion bootstrap that advances existing SVG poses while Canvas initializes. Do not wait for application hydration, account settings chunks, audio context, fonts, or notebook queries to show birds.

When Canvas is ready, render the identical geometry/pose and replace SVG in one frame without an entry fade, jump, or static-to-moving transition. Preserve phase and random seeds across the handoff. The first normal-session bird is mid-preen, mid-scan, or another ongoing action. A genuinely cold/slow snapshot uses a quiet sky/field with subtle optional ambient cues, never a spinner or a placeholder bird that would be a different identity.

The first-adoption empty scene is a distinct exception: after naming the two starters, the quiet field admits them with a gentle first fly-in. Reduced-motion substitutes a slow pose cross-fade. Record adoption presentation completion server-side so subsequent ordinary visits never restart an empty-scene onboarding sequence.

### Scene composition

Render soft sky/background foliage, three perch-depth bands, species-specific birds, optional offer/weather artifacts, and sparse foreground ornaments. Keep a restrained palette of soft blues/greens/browns/ochres. All birds remain visible and recognizable at all supported scene sizes. Do not add scene buttons, labels, badges, tooltips, mood icons, or drag handles. Captions and focus outlines are the narrowly required accessibility exceptions.

Use normalized scene geometry and reserved slots across front/middle/back zones, with responsive slot spacing and bounded z-order. On portrait phones compress the scene horizontally and letterbox when necessary rather than cropping birds or adding scene scrolling. Support safe-area insets, viewport-height changes, browser zoom, and coarse pointers. Surrounding settings/notebook content may scroll; the aviary scene itself never pans, scrolls, or zooms. Validate the seven-bird case at 320px viewport width. Keyboard/assistive controls remain accessible under high zoom even if nonessential decorations are removed.

Personality affects slow expressive parameters computed by the server; mood shapes idle pose distributions: wary scans at the back, content preens, curious tilts toward sounds, drowsy fluffs low on a perch. Drive micro-motion from bounded phase-based curves with varied durations, easing, and pauses. Subtle foreground/background parallax is slow and independent of aggressive pointer chasing. Limit ornaments initially to six concurrent leaves/feathers, pooled and recycled.

Top-bar controls are above the scene: account/settings (including bird rename, adoption, visits, export, deletion), accessibility, notebook, offer/actions. Fade after four seconds of cursor stillness; restore immediately on pointer movement or keyboard activity. Keep it fully visible while it or a popover has focus, while a modal is open, and on touch devices without hover. Never leave an interactive, unreadable control at low opacity during keyboard use. No badges signal notebook entries, eligibility, or visits. Age-based adoption appears calmly when the user opens its relevant panel, without achievement framing.

### Reduced-motion design

Use the OS preference plus an account setting of system/reduced; reduced wins when explicitly chosen. Switching modes takes effect without replacing simulation state. Replace body/head animation with slow cross-fades between curated still preen/scan/fluffed poses, initially six to ten seconds per pose. Replace flights by three-second fades between source and target perches; remove leaves, feather drift, and moving parallax. Retain day/night and settle color shifts over longer eight-to-twelve-second intervals. No sharp flashes or quick opacity pulses. Calls, captions, moods, drift, notebook, and interaction outcomes remain full quality.

## 9. Procedural audio and captions

Each species has a versioned symbolic motif grammar: note/trill tokens, frequency contours, duration envelopes, harmonic/noise balance, and pause distributions. Each bird's immutable signature seed selects a constrained register, timbre, and motif preference within the species. Mood changes note intensity, pacing, and contour; personality changes the probability/timing of calling and chorus participation. Keep register and timbral anchors stable enough to recognize the same bird across mood and weeks of drift, including two birds of the same species.

The server selects call occurrence times and emits a deterministic expanded descriptor with a unique call ID, grammar version, seed, note count, envelope/contour parameters, pauses, and contextual semantics. Client grammar interpretation synthesizes it through WebAudio; it does not schedule new behavioral decisions. Descriptors rather than audio files travel in snapshots. A shared grammar runtime derives caption phrases from the exact realized note/trill/contour descriptor, not a species-level fixed caption string.

Use one AudioContext per visible owner/visit tab, reusable species noise/wavetable buffers, per-bird gain buses, a master gain and conservative limiter. Procedural noise components are generated locally. Short-lived oscillators are stopped/disconnected after their envelope; buffers and graph references are bounded. Begin with a maximum of 12 simultaneous voices and seven bird buses. Overload drops the quietest unscheduled ornament voice rather than distorting foreground calls. Enforce comfortable peak/RMS levels and attack/release envelopes that avoid clicks.

Schedule audio using AudioContext time with roughly 100ms lookahead and a small scheduler loop. Map canonical server time into the audio clock. On resume, skip elapsed calls rather than rushing them. If a snapshot arrives mid-call, render its remaining semantic caption and resume sound only when envelope continuity permits; never blast a call from its start to catch up.

Listen-in ramps the target bird from ambient gain toward a gentle +3 to +6dB relative prominence over 1.5–2.5 seconds and ramps others to about 35–50% ambient, never zero. Target switching smoothly crosses envelopes. Disengagement restores ambient on the same slow timescale. Limit total loudness so listening in does not become a volume spike. Night, weather, and settle quiet the shared context gradually. Muting is a deliberate local/account audio preference; it stops audible output without stopping captions, calls' semantic schedule, presence, or simulation.

Call captions sit near the bird with a contrast-safe quiet backing, avoid covering its head, and appear/disappear with its actual call descriptor. Permit two or three simultaneous captions, using short combined descriptions during dense chorus without losing bird attribution. Reduced-motion uses slow opacity changes. Captions are available independently of audio and automatically on when audio cannot run. Do not route every caption through an assertive live region; narration has its own pacing.

Catch unavailable WebAudio, denied/suspended contexts, failed initialization, and hardware errors. Fall back to graceful silence with captions enabled; use a matter-of-fact explanation in accessibility settings. Audio can be retried through a user gesture. There is no recorded-audio fallback, no downloaded call file, and no loop pretending to be procedural.

## 10. Accessibility and voice acceptance

Use one shared semantic observation stream for visual cues, runtime captions, and prose narration so surfaces agree. Idle screen-reader narration updates every 45 seconds, adjustable within the required 30–60-second window. A polite, atomic live region presents a brief naturalist paragraph of current birds/place. Do not announce every snapshot, perch index, mood label, or trait. Keep at most one idle update and two priority interaction observations queued, superseding stale prose rather than creating an accumulating speech backlog.

Return-greetings and accepted offer/settle outcomes receive a prompt observation with priority over idle text. Suppress duplicate descriptions of the same cue. Opening settings or reading notebook pauses ambient narration where necessary to preserve control over speech. Provide pause/resume narration and a current observation action in accessibility settings; visual narration text, when enabled, also meets contrast requirements.

Represent the aviary as a labeled region with a roving-tabindex group of bird controls. Tab reaches top-bar controls then the first bird; arrow keys move between birds by stable scene order; focus engages listen-in; Enter ensures engagement; Escape disengages; Tab away disengages and leaves the group. Bird accessible names use name/species and a brief observational phrase, never vector values. Keep the Canvas decorative to assistive technology to avoid duplicating its DOM representation.

Offer/actions opens via its button or an unmodified O shortcut only when the scene has focus, with no shortcut capture in text inputs or assistive contexts that intercept it. The popover supports arrows/Tab, Enter, Escape, and focus restoration. S activates settle in the same restricted context. The five-second undo is keyboard reachable. Modals trap focus appropriately and restore it; notebook pagination and scrolling remain keyboard and screen-reader usable. Visitor mode exposes no listen-in bird buttons or owner action shortcuts; its scene description remains accessible.

Provide at least 44px touch targets for top-bar controls and accessible bird hit areas. Use a two-tone soft focus outline that remains distinct against morning, rain, evening, and night. Design text tokens for at least 4.5:1 ordinary text and 3:1 large text, with at least 3:1 controls/focus treatment; test captions against every light state. Top-bar idle fade must not undermine text contrast when controls are being used. Include zoom, high contrast/forced colors, touch, and text scaling in acceptance.

Author product prose in lowercase present-tense naturalist voice with specific named bird observations. Account, auth, errors, sync, privacy, deletion, and accessibility settings use ordinary capitalization and direct matter-of-fact messages. Do not disguise an outage as weather or an expired sign-in as a bird behavior. Review every user-visible string for this boundary and for accidental welcome/gamification copy.

## 11. Account lifecycle, privacy, and optional social behavior

At first verified sign-in create account/aviary identity transactionally; adoption reservations are idempotent. Show the two arrived species with suggested names, permit naming, and enter the scene without catalog browsing. Renaming never affects the signature seed or engine state. Email lookup uses a keyed digest within identity only; all other references use synthetic UUIDs. Session settings permit individual revocation. Rotate session tokens and cap them initially at 30 days with reauthentication; magic-link reuse never resurrects a revoked session.

Generate exports by reading one consistent committed aviary revision and paginating immutable notebook entries within that export's cut-off. Include stable bird IDs/names/species, current vectors under the explicit export exception, moods, notebook entries and account settings; exclude raw event logs, presence history, visitor emails, session secrets and tokens. Encrypt the object at rest, expire it and its one-time download token after 24 hours, and email only the verified address. Link pages suppress referrers and token logging. Do not expose a trait preview in settings.

Deletion immediately marks the account, revokes active owner/visitor sessions and invitations, stops simulation, cancels outbound jobs, and removes public access. Allow a fresh magic-link sign-in to a recovery-only surface for 30 days. Recovery retains bird IDs/vectors and resumes elapsed ambient evolution without inventing absence presence; it does not reissue old visit permissions. At day 30, purge account identity/contact ownership, birds/vectors, event inputs, notebook, private visit records, sessions, export objects, outbox jobs, cache entries, and any account-correlated operational traces.

Use encrypted backups with per-account key material that can be destroyed on hard deletion and restore-time deletion tombstones; a backup restoration must not resurrect purged accounts. Choose backup retention no longer than the hard-deletion promise can support with cryptographic erasure. Anonymous aggregate counters have no account linkage and cannot reconstruct deleted data; identifiable error traces, if enabled for system issues, use UUIDs only and are covered by deletion. Test recovery at day 29 and irreversible purge after day 30.

Keep simulation data inaccessible to the telemetry/analytics service account. Private bird events are for that user's simulation and nothing else: no training, recommendation systems, population interaction analysis, average trait dashboards, or third-party behavioral tracking. No request-body logging on interaction, snapshot, export, or invite routes. Mail transport handles only explicitly required identity/transactional content, not bird interaction history.

Invitations are individually deliberate, revocable, and initially nonexistent. There is no discoverable flag or onboarding sharing prompt. Visitors get identical canonical birds, lighting, weather, call descriptors, and quality, using the host timezone. They get no arrival/greeting event, interaction endpoints, presence tracker, listen-in mix controls, offer/settle, private notebook, or host co-presence overlay. Authorization enforces this even if a visitor handcrafts an owner API request.

Visit timing uses server receipt of snapshot pulls solely for the host's private log, not personality or engagement analytics. Approximate duration excludes long pull gaps and ends at expiry/revocation. Account settings list recipient email resolved from the identity store, date, approximate duration, and outstanding invitations. Keep historical visits for transparency; revoked invitations disappear from outstanding/active sections and historical rows remain marked ended/revoked. Do not notify the host when revocation succeeds. With the explicit visit-email opt-in on, send at most one quiet email per consumed visit; no duplicate mail on retries or return polls. With it off, a visit produces no host notification anywhere.

## 12. Performance budgets and operational observability

Treat the 2MB gzipped initial-JS limit as a ceiling, not a delivery target. Aim for under 250KB gzipped critical/client scene code and under 400KB including initial audio grammar/runtime; lazy-load notebook, account/accessibility panels, invitations, and export surfaces. Inline critical SVG/CSS/pose bootstrap should be tens of kilobytes. Keep initial snapshots typically under 12KB and below 24KB at seven birds by limiting horizon descriptors and omitting private history. Compress responses and immutable assets.

| Requirement | Engineering budget and verification |
| --- | --- |
| First bird <500ms | Reference mid-tier phone over a specified 4G profile; target edge auth/snapshot/TTFB below 200ms, HTML transfer/parse below 150ms, initial draw below 100ms; measure navigation start to non-placeholder bird pixels |
| Greeting 1–2 seconds | Time from visible owner return to primary visual cue, including arrival API/expedited tick; synthetic offline-degraded cases separately labeled |
| Idle 60fps | Frame budget 16.7ms on a five-year-old mid-range laptop for 30 minutes at seven birds; scene work target below 6ms/frame, no ordinary long tasks above 50ms |
| No memory growth over 30 minutes | After warm-up, repeated samples/full-GC lab comparisons show no sustained retained-heap slope or growing audio-node/DOM/listener counts; investigate growth above measurement noise, not simply permit an unbounded slope |
| Tick p99 alarm above 5 seconds | Measure scheduled due-to-commit latency and compute duration independently; alarm if p99 exceeds five seconds over a five-minute window, and on growing scheduler backlog |
| Supported browsers | Last two major versions of Chrome, Safari, Firefox, Edge; unsupported surface matter-of-fact; supported browser without working audio uses captions |

The sub-500ms path relies on inline already-authenticated state and minimal critical bytes, not loading a near-2MB framework then drawing. Synthetic performance fixtures must specify actual network latency/bandwidth, device CPU, geography, cache state, and account bird count. Gate launch on the required reference profile; also record cold connection, cold regional cache, and constrained-network tails. Extremely slow links may necessarily show the quiet field, but they must not introduce a spinner or fictitious birds.

Bound devicePixelRatio initially at 2 for Canvas and use cached path assets/background layers. Allocate typed motion arrays and ornament pools once; avoid object creation per frame. Remove old snapshots and schedules as soon as superseded. One animation loop, audio context, and bounded scheduler per tab; no proliferating workers. Pause hidden work and dispose contexts/listeners at logout, visit termination, or route teardown.

From day one emit aggregate operational metrics: request counts/latency/error codes by endpoint class, snapshot byte sizes, navigation/first-bird durations, sampled frame duration histograms, audio failure categories, tick compute/queue lag, cache/outbox failures, and anonymous coarse session-duration histograms. Do not include bird IDs/names/species/moods/traits, presence seconds, offer/listen behavior, invitation recipients, emails, or account interaction histories. Strip high-cardinality account identifiers from metric dimensions; send only allowlisted anonymous summaries. Keep system-error UUID tracing separate from aggregate telemetry and retention-controlled.

Run scheduled synthetic browsers in common launch geographies using artificial fixtures only. Dashboards show health/performance, not engagement, retention, visit streaks, adoption rates, or mean personality drift. Schema validation and a canary that inserts sentinel private fields test that those fields never reach logs/metrics. Calibrate mechanics through authored synthetic cohorts and explicitly recruited usability sessions with consent; do not analyze production private event histories.

## 13. Engineering sequence, testing, and rollout

### Milestone A — contracts and aliveness prototype

Owners: technical lead, simulation engineer, visual/motion designer, audio engineer, accessibility specialist. Deliver versioned schemas, six silhouettes/signature grammars, three perch-zone geometry, naturalist/system copy guide, and artificial accounts at two/seven birds. Build a thin server snapshot-to-first-SVG-to-Canvas path with one procedural call grammar and reduced-motion still poses. This is a future implementation milestone, not product code included in this plan.

Exit: first frame already moving, no handoff jump, representative first-bird/frame budgets met, first grammar/caption agree, screen-reader prose/focus model validated. Test the affective experience early so infrastructure cannot lock in a robotic renderer.

### Milestone B — persistence, tick, and honest presence

Owners: backend/infra and simulation engineers. Deliver database migrations, restricted vector writer, deterministic worker, event ordering/idempotency, union presence/listen intervals, mood/timezone/weather, call/action horizons, and snapshot projection. Add backup/restore and migrations preserving bird identity before real accounts are accepted.

Exit: concurrent devices/events produce one committed vector; zero eligible activity produces zero invented input; server runs while all clients are absent; restart/outbox/crash cases preserve state. Accelerated synthetic weeks meet drift monotonicity and timing targets. No reset-from-log recovery exists.

### Milestone C — full owner experience and accounts

Owners: frontend/backend, content designer, security reviewer. Deliver magic links/sessions, adoption/naming, return greetings, offers/cooldowns, listen-in envelopes, settle/undo/re-engagement, sparse notebook, timezone settings, export, recovery/purge. Extend grammar/renderer to all six species and seven-bird fixtures. Accessibility features remain in each vertical slice.

Exit: the full quiet session works by mouse/touch/keyboard and without audio; no welcome or gamification copy; state is consistent after cross-device switch and laptop suspension. Fresh accounts always start with two server-chosen birds.

### Milestone D — visits and launch hardening

Owners: identity/backend/frontend plus accessibility/QA. Deliver individually issued invitations, visitor projection, single-use tokens, next-pull revocation, private settings log, off-by-default visit-email opt-in. Complete browser matrix, 30-minute memory tests, geo synthetic probes, deletion erasure tests, privacy telemetry canaries, and sound-recognizability review.

Exit: visitors cannot produce any simulation input, including forged requests; default visits remain silent to the host; reduced-motion, captioned, and narrated surfaces are release-complete. Operational runbooks cover tick backlog, vector restoration, email problems, compromised sessions, export failures, and audio fallback.

### Required verification matrix

- Simulation properties: traits stay in range and nondecreasing over random absence/input schedules; a one-minute burst never creates visible drift; muted watching is equivalent presence; settle and close end presence equally; filter residual is bounded; tick partitioning and retries produce equivalent results.
- Presence truth table: test all eight visible/focused/recent-activity combinations, exact five-minute boundary, no initial activity, blur/hide race, settled pointer movement, 30-second lease expiry, suspend/resume, delayed events, overlapping devices, and visitor exclusion.
- Transaction integration: crashes before/after state commit, duplicate receipts, concurrent tick claims, sequence allocation commit order, event watermark races, seven-bird concurrent adoption, two-device offer cooldown, stale names/settings, publication out-of-order, revocation with conditional GET.
- Continuity: overnight/daylight-saving/timezone changes, two-week absence, reconnect after long sleep, code/grammar migration, rollback, primary/cache failover, backup restore. Assert exact bird UUID/signature/vector preservation, not just visual plausibility.
- Procedural audio: descriptor-to-caption fidelity, smooth mix switch/end, nonzero other-bird ambient gains, no clipping, bounded voices, unavailable/suspended contexts, mute/unmute, and absence of audio-file requests. Use synthetic sound sets for blind bird-identity recognition across moods at two/four/seven birds; initial gate is at least 80% identification after familiarization, tuned before launch.
- Visual first-frame: frame captures at first paint and renderer handoff, all viewport sizes/day states, seven birds always visible, onboarding-only fly-in, no spinner/entry fade, changed greeting parameters over repeated returns.
- Accessibility: automated contrast/semantics checks plus manual VoiceOver/Safari and NVDA/Firefox/Chrome sessions; keyboard-only offers/settle/undo/notebook/auth; reduced-motion pose/perch fades; caption collisions; forced colors and 200–400% zoom. Validate charm and speech cadence with users, not only ARIA presence.
- Performance: compressed bundle gate, reference 4G first-bird measurement, 30-minute seven-bird soak with repeated notebook/setting opens, audio and listeners leak accounting, tick load at projected launch population, hidden-tab battery/work checks.
- Privacy/security: unauthorized bird/account references, token replay/expiry, scanner-safe link consumption, CSRF/XSS, recipient identity isolation, no plaintext emails in logs, no private simulation fields in telemetry, deletion and backup erasure, export atomic snapshot and expired-link denial.

### Rollout policy

Use internal synthetic accounts, then a small consented private beta, then a staged infrastructure ramp (for example 1%, 10%, 50%, 100% of admitted accounts) controlled by health/quality gates rather than engagement conversion. Keep deterministic fixture accounts available in every environment. Feature flags are server-controlled and default-deny social until its acceptance suite passes. Full public v1 includes visits and all accessibility surfaces.

New accounts remain two birds at every stage. Capacity-test four and seven birds with artificially aged fixtures first; let beta accounts adopt a third only when age-eligible. Gradually raise the permitted age-eligible adoption ceiling from three to five to seven after audio distinction, scene layout, frame/memory, and concurrency gates pass. Do not auto-add birds, charge for them, expose a count reward, or tie rollout eligibility to visits. Do not require waiting a year to validate seven birds: accelerated private fixtures supply evidence before public eligibility matures.

Operational rollback disables new invites/adoptions or a faulty grammar version and preserves all existing birds, vectors, signatures, notebook entries, and ownership. Never reduce an existing aviary's bird count or reseed a bird to roll back code. Pin grammar/simulation versions and migrate them with continuity checks. If audio alone fails, the graceful-silence caption path keeps the aviary usable. Do not market a partial release that lacks a required accessibility mode as completed v1.

## 14. Risks and concrete mitigations

| Risk | Mitigation and decision gate |
| --- | --- |
| Drift feels immediate or imperceptible | Synthetic 1/7/21/90-day cohorts, continuous expression mapping, observational user review, versioned constants; never adjust via production private-history aggregation |
| Absence accidentally punishes birds | Separate transient familiarity/mood from immutable accumulated trait direction; absence regression tests and content review; minimum ambient life |
| A retry or competing worker loses/adds drift | Aviary row lock, ordered committed event cursor, atomic state/outbox transaction, restricted writer role, duplicate/stale/concurrent integration tests |
| Vector loss silently destroys identity | Transaction-consistent backups, restore rehearsals with equality checks, durable vectors independent of log retention, monotonic migration guard, no auto-reseed fallback |
| Presence inflates because of focus/mobile gaps | Three-condition truth table, small validated segments, union across devices, no offline backfill, no click-only shortcut; calibrate activity window with synthetic/manual observation |
| Server schedules feel sluggish | Expedited passes reuse the canonical writer, bounded request coalescing, server-provided interaction cues; instrument due-to-commit and cue latency |
| Calls become uncanny or indistinguishable | Stable signature anchors, bounded procedural variation, audio review, seven-bird recognition gate, gentle gain/voice limits; retain caption fallback |
| Narration/captions overwhelm attention | Shared semantic source, 45-second idle cadence, bounded queue, priority observation coalescing, caption placement/chorus tests and manual assistive review |
| Initial render misses the affective budget | Inline private snapshot/SVG, tiny motion bootstrap, lazy noncritical surfaces, regional synthetic gates; quiet-field slow path, never fabricated identity |
| Edge caching exposes another account or stale visit | Auth before cache, role-specific keys, strong revision handling, no shared personalized HTML cache, revocation check even on 304 |
| Privacy leaks through ordinary logs/analytics | Separate DB roles and stores, no request bodies, UUID-only system traces, allowlisted metrics, sentinel canaries, deletion across cache/object/backup paths |
| Minute ticking all accounts becomes costly | Benchmark per-aviary tick at seven birds, indexed due queues, sharded claims, batched DB I/O and bounded schedule generation; scale workers without skipping unattended accounts |
| Design ambiguities generate scope creep | Document decisions above, keep four primary icons and no scene chrome, copy lint/manual review, age-only optional adoption, narrow export/visit-email exceptions |

Completion means the team can demonstrate a quiet, continuous aviary across devices and accessible modes, with measured drift, procedural recognizable calls, precise presence, bounded resource use, and private data remaining private. A feature checklist alone does not establish that the birds feel alive; release review must include the actual first return, a few minutes of uneventful watching, and a comparison after simulated weeks.
