# Pocket Aviary — executable v1 implementation plan

## 1. Product contract and scope

Build a browser window into one continuing aviary per owner. The first encounter contains two server-selected, individually named birds; age alone makes additional adoptions available, up to seven. Six species provide distinct silhouettes and procedural call signatures, including a nightjar-like species that can remain active at night. Implement magic-link accounts, canonical server simulation, cross-device snapshots, presence accounting, greeting, listen-in, the three offers, settle, rename/adoption settings, a sparse read-only field notebook, explicit email visit invitations, privacy/account lifecycle, narration, captions, and reduced motion in the initial release.

The release criterion is a believable relationship over weeks, not a collection of working controls. Identity and personality survive renames, device changes, migrations, and absence. No action can buy or rapidly grind expressiveness. Birds continue moving through moods without a connected client. User absence never reduces a personality trait or creates hunger, sickness, distress, resentment, or obligation.

Do not build native clients, payments, SSO/passwords, shared aviaries, multiple aviaries, scene customization, bird placement controls, public discovery, profiles, follows, feeds, chat, comments, co-presence, rankings, scores, levels, achievements, streaks, counters, visit-frequency summaries, push notifications, or recorded call audio. Do not construct analytics that could later power these surfaces. Ordinary tab closure is a complete and valid goodbye.

Product prose uses lowercase, present tense, named birds, specific observations, and restrained bird vocabulary. Identity, settings, accessibility configuration, failures, unsupported-browser messages, and account operations use direct ordinary English. There is no return toast, banner, arrival modal, absence-duration label, or textual welcome. Narrating a bird's greeting for a screen reader is the accessible bird greeting, not a welcome announcement.

### 1.1 Decisions where the PRDs leave details open or conflict

These are implementation decisions, not additional features:

| Topic | Binding v1 decision |
| --- | --- |
| Cadences | Canonical simulation steps are 60 seconds; visible owner snapshots every 15 seconds, visitor snapshots every 10 seconds; presence pings every 15 seconds; recent pointer/key activity window initially 5 minutes. Calibrate using synthetic histories and explicit usability studies. |
| Account timezone | One stored IANA timezone drives the canonical aviary. Initialize from the creating browser; an owner can change it in settings when traveling. Devices never silently overwrite it. A visitor uses the host's timezone. This resolves the impossibility of one shared mood being in two local evenings simultaneously. |
| Mood states | `wary`, `content`, `curious`, `drowsy`, `alert`. Sleeping is a drowsy pose, not a health state. No mood label is displayed on birds. |
| Offer cooldown | Three minutes per receiving bird across all offer kinds and devices, enforced by server admission. A gesture may affect more than one eligible bird; rejected/ignored reactions carry no punishment. |
| New adoption dates | Initial candidate schedule: third at day 90, fourth at 180, fifth at 270, sixth at 365, seventh at 540 since aviary creation. Availability persists, can be ignored, and has no countdown, unlock announcement, scarcity, rarity, or attention requirement. Review schedule before release; age, not engagement, remains the only criterion. |
| Settle control | Layout's sparse icon list omits settle, while interactions require it in the top bar. Add a small fifth top-bar settle affordance; no scene chrome. |
| Captions and focus | The layout's no-inline-copy rule has explicit accessibility exceptions: call captions near the bird and a high-contrast focus outline. They are not status labels, badges, or tooltips. |
| Keyboard listen-in | Focus entering a bird engages listen-in as interactions specifies; Enter explicitly toggles it. Arrow focus transfers it; Escape disengages while retaining focus. Tab away ends it. Suppress automatic re-engagement on the same focus after Escape until a new explicit action/focus movement. |
| Exported vectors | `accounts_sync.md` requests current vectors in a JSON export, but the engine prohibits exposing numbers in any user surface. Export a server-sealed, versioned personality capsule per bird instead of readable numerical fields. It contains the current persisted vector, encrypted with a server-only key; no plaintext values appear in JSON, DOM, ARIA, logs, or APIs. Export includes readable names/moods/notebook/settings. This prioritizes the absolute no-numbers rule; the copy is not a user-readable vector backup. Import/restore is outside v1. |
| Visit notifications | The general no-notifications principle conflicts with the explicit social settings toggle. Treat the narrower social requirement as an exception: an account-settings toggle, off by default and absent from onboarding, permits a plain visit email only after explicit opt-in. Never push, toast, badge the settings icon, or email about birds, absence, weather, adoption, or notebook entries. Invitation, sign-in, verification, and requested export mail are transactional. |
| Revocation and log history | Revocation removes an invitation from outstanding/current access and terminates its session. Keep completed visit-history rows for transparency; do not erase who previously saw the aviary to manufacture the specified absence of an active visitor. No success notification. |
| Sound on navigation | Browser autoplay restrictions prevent promising audible calls on every first visit. Immediately render the actual call captions if audio cannot run; resume synthesis after an allowed gesture. An already permitted AudioContext starts at the current call phase, without replaying earlier calls. Never block birds behind a sound-enable modal. |

The PRD references a separate visual design system but does not supply it. The team must produce approved palette, contrast, silhouette, pose, and focus tokens as an implementation dependency; do not invent a dependency on an unread repository document.

## 2. Architecture and ownership boundaries

Use a TypeScript web client with a small semantic HTML shell and SVG bird renderer; a modular TypeScript HTTP service; PostgreSQL as canonical storage; and independently deployable simulation, email, export, and deletion workers. Keep the simulation kernel as a pure versioned library shared by server workers and synthetic test harnesses, never imported as a client state writer. No event broker or analytics warehouse is needed for v1. A transactional outbox and durable database jobs avoid infrastructure that adds ordering ambiguity.

Deploy stateless web/API instances near the primary database. Use CDN edges for immutable code/assets and an authenticated HTML delivery path that includes a private snapshot. Public CDN caches must never contain account snapshots. At greater scale, shard by synthetic aviary UUID and keep one primary writer for each aviary; do not use geographic multi-master personality writes.

### 2.1 Server responsibilities

- Authenticate owners and visitors; authorize every route and every snapshot, including cache revalidation.
- Store immutable bird identities, vectors, moods, recent simulation memory, event order, adoption eligibility, timezone, notebook, invitations, account lifecycle, and device sessions.
- Admit semantic interaction events, stamp/order them, reserve offer cooldowns, deduplicate retries, validate presence intervals, and build authoritative immediate response descriptors.
- Run the minute tick for every active aviary whether connected or not. Only the tick role writes personality columns. API/event roles have no permission to update vectors.
- Serve versioned render projections of current canonical state; provide fresh greeting/reaction descriptors promptly without requiring clients to simulate mood or drift.
- Generate sparse grounded notebook prose, lifecycle exports, deletion, and the isolated operational metrics described below.

### 2.2 Client responsibilities

- Draw a render projection at server time, interpolate pose/perch transitions, synthesize already-described calls, and generate local ornamental leaves/feathers. Never choose persistent mood, personality, weather, or bird identity.
- Collect the exact presence conjunction, submit ordered interaction events, pull snapshots, and apply strictly increasing versions.
- Maintain ephemeral presentation state: keyboard focus, local audio gain, transient settle preview, open settings, caption preference overrides when audio fails. These are not an alternative aviary.
- Translate shared semantic observations and call descriptors into narration/captions. Client procedural expansion is a deterministic rendering of server-authorized programs, not a simulation tick.

### 2.3 State flow

An authenticated request retrieves account status and a render projection in the HTML response. The initial SVG has real birds in current poses before the full client loads. The lightweight renderer adopts that SVG and advances its phase. A dedicated arrival request is admitted as an event, returns a procedurally selected greeting descriptor within the first second or two, and reserves it so concurrent owner tabs cannot provoke a synchronized greeting. Visible clients pull later versions; interactions append events and receive authoritative short-term reaction descriptors. The tick consumes events and commits mood, personality deltas, scene programs, notebook decisions, event cursor, and state version in one transaction.

Calls, bird poses, greeting outcomes, and weather are based on the same scene descriptor for owners and visitors. Listen-in gain is owner-local attention; it does not change the visitor's ambient mix. Settle is a canonical scene lighting override after consumption, so another device and visitors see the same settled aviary; an origin client previews it immediately. No visitor request causes a greeting or modifies the host scene.

## 3. Persistent data model

All IDs are random synthetic UUIDs, all durable times UTC, and all local-calendar calculations use the account timezone. Email is encrypted once in the identity/account record and is never a foreign key, partition key, queue identity, URL identifier, metric label, or log field. Lookup for magic links can use a keyed blind index in that record; it is sensitive, inaccessible to telemetry, and is not an internal identity. Invitations and sessions reference UUIDs.

| Record | Required fields and constraints |
| --- | --- |
| Identity/account | `account_id`, encrypted verified/pending email, lookup blind index, verification status, `created_at`, `owner_activated_at`, account status, `delete_requested_at`, `hard_delete_at`, timezone, schema version. Lightweight visitor-only identities hold the invite address once without creating an aviary; activation as an owner creates exactly one aviary. |
| Account settings | Account UUID, captions, narration pacing/enablement, reduced-motion preference (`system` or `on`), audio enabled/volume, visit-notification opt-in default false. Settings revision for optimistic concurrency. A device's stricter system reduced-motion setting remains respected. |
| Device session | Session UUID, account UUID, token digest, safe user-assigned/device-family label, created/last-used/expiry times, revocation time. HttpOnly session cookie never enters JS storage. |
| Auth challenge | UUID, account UUID, purpose, one-use random token digest, expiry, consumed time, pending verification reference. Separate purpose binding for sign-in, email change, and invite redemption. |
| Aviary | Unique owner account UUID, aviary UUID, creation time, current canonical version, tick index, `last_tick_at`, next due time, engine/config/schema versions, deterministic seed/PRNG cursor, canonical light phase/weather schedule, settle generation/override, last owner-qualified presence time, event consumption cursor. |
| Bird | Stable bird UUID, aviary UUID, immutable species/motif-signature seed, name plus revision, adoption time, five persisted scalar traits in `[0,1]`, drift filter accumulators, mood/timers, perch zone/slot, current pose/transition program, call-program descriptor, recent-interaction decay memory, age/adoption ordinal. No normal delete/replacement endpoint. |
| Species definition | Versioned immutable definition key, six silhouettes/palettes, default trait seed distribution, stable identity call grammar, motif catalog, pose rig/pose library, nocturnal behavior profile. Existing birds remain pinned to compatible definitions across releases. |
| Interaction event | Account/aviary/device/client-session UUIDs, event UUID, monotonically increasing aviary sequence, client-local sequence, kind, constrained payload, server receipt time, accepted effective interval, schema version, admission result. Unique `(aviary_id,event_id)`; retrying the ID must use the identical payload. Append-only while retained. |
| Presence accumulator | Aviary UUID, validated recent intervals, server dedupe watermark, merged account-level presence seconds by tick/day; qualifying bird listen intervals separately, globally bounded by the same account-time union. Not a user-facing history or telemetry stream. |
| Admission state | Aviary/bird IDs, per-bird offer-next-allowed timestamp, owner greeting lease, bounded listen lease by client session, active reaction descriptors, version. Separate from traits and updated transactionally by event admission. |
| Notebook entry | UUID, aviary UUID, occurrence/creation timestamps, timezone date, grounded observation facts, template revision, resolved naturalist text, optional stable bird references. Immutable; name as observed is retained in old prose. Index by aviary and timestamp/ID for cursor pagination. |
| Invitation | UUID, host UUID/aviary UUID, visitor identity UUID, token digest, issued time, unused expiry at +30 days, consumed/revoked times, active visit-session reference. No permanent friends relation. |
| Visit session/history | UUID, invitation UUID, host/visitor UUIDs, session token digest, started time, last authorized pull, expiry/closed reason, approximate duration. Not a simulation event; no pointer coordinates, focus stream, or per-bird activity. |
| Durable jobs/outbox | UUID, account/aviary UUID where required, kind, due time, bounded payload containing only references, dedupe key, attempts, lease/status. Email workers retrieve a recipient only while sending. |
| Export artifact | UUID, account UUID, source canonical version, expiry, encrypted object key, download-token digest, lifecycle state. Private object storage; no state payload in job logs. |

Enforce at most seven birds inside an aviary transaction; concurrent adoptions lock the aviary and consume distinct age slots. A starter creation transaction creates exactly two distinct species, birds, vectors, names/default suggestions, and an initial scene. Failure rolls back all of it; never offer a reset to repair partial state. Names accept bounded Unicode text (1–40 grapheme clusters), normalize safely, reject control characters, render as text, and do not change seeds or simulation state.

Personality storage uses sufficient precision for minute-scale increments: a versioned, account-key-encrypted canonical payload preserves IEEE-754 double values, with finite-range and monotonic checks enforced by the simulation kernel before every commit. The database restricts writes to that payload to the tick role. Do not also retain an unencrypted numerical shadow column that would defeat account-key erasure. Store vectors themselves, not an event-replay recipe. Persist calibration/filter memory and consumption cursors so service restarts do not repeat deltas or erase the slow current. Maintain schema-compatible migrations, encrypted backups, and point-in-time recovery; restoration preserves IDs, vectors, versions, and processed-event cursors together.

## 4. API and command contracts

Serve same-origin `/api/v1` JSON endpoints with validated bodies, CSRF protection for cookie-authenticated mutations, origin checks, bounded input sizes, and endpoint-specific rate limits. Use typed owner/visitor authorization contexts; visitor routes cannot reach the command writer. Return request IDs and matter-of-fact errors without internal state, tokens, or email leakage.

### 4.1 Endpoint surface

| Endpoint | Input/output and behavior |
| --- | --- |
| `POST /auth/magic-links` | Email; generic accepted response to prevent account enumeration. Issue a 15-minute one-use challenge, at most 5 requests/email/15 minutes initially plus IP abuse limits. Do not lock the account because of abuse. |
| `GET /auth/link` and `POST /auth/consume` | GET renders a minimal confirmation page; POST atomically consumes purpose-bound token and creates a per-device session. This keeps email-link scanners from consuming links. A replay/expiry offers a clear new-link action. |
| `POST /auth/logout` | Revoke current device session and clear cookie. |
| `GET /account` / `PATCH /account/settings` | Settings/account status; mutations use expected revision. A timezone change is committed once, takes effect in the next canonical tick, and transitions lighting smoothly. |
| `GET /account/sessions` / `DELETE /account/sessions/{id}` | List/revoke own devices, including current device. Revocation rejects subsequent commands and snapshot pulls. |
| `POST /account/email-change` / `POST /account/email-change/verify` | Stage address and verify before replacing encrypted verified email. Existing address remains valid until atomic commit; a uniqueness collision produces direct recovery guidance. |
| `POST /aviary/starter-adoption` | Idempotency key and optional two names; create the assigned starter pair once. Pre-adoption preview is persisted, not rerolled on reload. |
| `GET /aviary/snapshot` | Latest projection, `ETag` canonical/projection versions and server time. `304` only after current authorization. Recheck on resume and keepalive. |
| `POST /aviary/events` | Bounded batch of event IDs, client session/sequences, and semantic payloads. Return individual acceptance status, canonical version, corrected server time, event cursor, and any authoritative greeting/reaction descriptors. Never accept personality or mood fields. |
| `PATCH /aviary/birds/{id}/name` | Name and expected name revision; a stale edit returns 409 and the latest name for an explicit retry rather than quietly replacing another edit. |
| `GET /aviary/adoption` / `POST /aviary/adoption` | Persistent eligible age-slot presentation; accept a server-proposed bird and optional name with idempotency key. No catalog, rarity, progress bar, or unlock toast. |
| `GET /aviary/notebook?before=...&limit=...` | Read-only descending cursor pages, maximum 30 entries/page. No edit/delete/annotation operations. |
| `POST /account/invitations` | Owner-specified email; create invitation/visitor principal and transactional mail job. No global sharing switch. |
| `GET /account/visits` / `DELETE /account/invitations/{id}` | Outstanding invitations plus historical visits; revoke under lock, invalidate active session immediately. Settings list updates in place. |
| `GET /visit/link` / `POST /visit/redeem` | Confirmation GET, atomic one-use redemption POST. Verify possession of the emailed token, issue narrowly scoped HttpOnly visit cookie, never create an owner session. Unused invite expires after 30 days. |
| `GET /visit/snapshot` / `POST /visit/end` | Authorized render projection; ending only updates access-log duration. No mutation/presence event endpoint. Expired/revoked access returns 410 with the same unavailable surface. |
| `POST /account/export` | Owner request; worker creates a consistent JSON snapshot and emails a one-use private download link. No scheduled export emails. |
| `POST /account/deletion` / `POST /account/recovery` | Mark deletion for 30 days; recovery is an explicit “I changed my mind” action before the deadline. Hard-deleted accounts cannot be recovered. |

Additional bird naming/adoption controls live in account settings, not inside the scene. All list routes are scoped by the authenticated synthetic account UUID, never by supplied email.

### 4.2 Snapshot projection

Include `schema_version`, `engine_version`, `canonical_version`, `projection_version`, `server_time`, `tick_time`, account timezone, scene light/weather/settle descriptors, and at most seven birds. Each bird contains UUID/name/species, current mood-derived poses, perch path with times, compact visual expression parameters, a stable call-signature definition key, and a bounded procedural call/action program with absolute times and seeds. Include admitted short-lived reaction overlays and narration facts when necessary. Owner-only metadata can include the latest accepted event cursor and adoption availability; the visitor projection omits those, account settings, email, notebook, and presence history.

Do not send the five personality numbers or a raw vector. The server maps them into a noninvertible presentation projection: bounded appearance palette bands, action program choices, and varied call parameters. Rendered effects necessarily imply behavior; avoid a one-to-one trait field disguised as a coefficient or an enumerable numeric stat panel. Projection schemas, bundled code, ARIA strings, error bodies, and source maps contain no state values. Internal synthetic instrumentation may inspect vectors inside the test harness, never in a deployed user-accessible debug view.

Target seven-bird projection under 12 KB uncompressed, with programs represented as compact recipes rather than hundreds of frame records. Recipes are valid at any time within their horizon; a snapshot derived from the latest committed state extends presentation deterministically without advancing canonical mood or personality. Use this to show continuous idle action between minute ticks. Clients cannot extend a recipe into a new authoritative mood, weather event, or accepted offer.

### 4.3 Event protocol

Allowed kinds are `arrival`, `presence`, `listen_start`, `listen_end`, `offer`, `settle`, `settle_undo`, `reengage`, and `session_end`. Presence carries only the bounded qualifying interval, visibility/focus booleans, and age of last qualifying activity, not keystrokes, pointer coordinates, or text. Listen events reference an existing bird and session; offer specifies one of three kinds and a library fragment ID if needed; settle-undo references its settle event. Mute/unmute is a device preference plus, if needed for the brief's behavior influence, a bounded owner audio-preference signal that can only modulate immediate ambient expression. It never reduces traits or penalizes an accessibility choice.

The server authenticates, validates, deduplicates, and assigns an aviary sequence while holding the same aviary serialization lock used by ticks. It reserves cooldowns and greeting leases, writes the event and immutable procedural reaction outcome, and returns the outcome. Outcome probabilities use the latest persisted mood/personality; the tick later applies that exact outcome and its delta input, rather than rolling a second reaction. Immediate visual/audio reaction is therefore server-authored but not delayed by a minute. Pending descriptors are included in pulls for other devices and visitors. Tick commit absorbs/removes pending overlays atomically to avoid double play.

If the network is unavailable, continue drawing the last valid scene quietly, show a small matter-of-fact connectivity status in the top bar or open settings when relevant, and do not pretend an offer was accepted. Do not create offline drift or replay hours of buffered presence. Retry uncertain acknowledged commands with the same ID; cap the short in-memory retry queue at 32 events, expire presence intervals older than 30 seconds, and discard stale arrival/listen/offer commands rather than replaying them after a long absence.

## 5. Canonical simulation engine

### 5.1 Scheduling, transaction, and recovery

Schedule every non-deleted active aviary each minute, with deterministic hash-based phase offsets to avoid a fleet-wide burst. PostgreSQL due-time indexes plus `FOR UPDATE SKIP LOCKED` leases distribute jobs; inside a transaction lock the aviary row, verify tick/version, and advance exactly one due step. A duplicate worker either waits or sees an already advanced index and exits.

For each step: read accepted events after the cursor through a fixed sequence cutoff; union presence intervals; update short-term attention/filter memories; compute nonnegative additive trait deltas; transition mood and perch/action programs; advance weather/light/settle state; select grounded notebook candidates; persist vectors, moods, PRNG cursor, cursor/version/next due time, notebook entries, and outbox work together. No partial consumption. An event admitted after the cutoff belongs to the next step.

Replay missed scheduled minute steps in bounded batches after downtime. Use persisted seeds, ordered events, UTC tick boundaries, and engine version to make catch-up deterministic. Do not defer all evolution until a viewer arrives. If backlog exceeds the worker batch limit, enqueue the remaining due work immediately and serve the latest committed projection without resetting moods. For long outages, test a no-event analytical filter-decay path, but only enable it after proving it produces the same integrated drift as minute steps and respects all day/weather boundaries; correctness comes before compression.

A daily-ish mood rebalancing is an internal transition kernel, not a forced midnight neutral reset. Daily baselines vary with timezone/time and individual history. Keep frame-scale rendering out of the tick: it chooses temporal programs; clients sample poses and calls from them.

### 5.2 Presence accounting and multiple devices

Client eligibility is exactly:

`document.visibilityState === 'visible' AND document.hasFocus() AND elapsedSincePointermoveOrKeypress <= 5 minutes AND notSettled`.

Use the explicit `pointermove` and keyboard event families only. Do not count an open tab, scrolling in another app, a periodic timer, audio playing, blur, or an offer click alone as presence. Keyboard/screen-reader interaction must exercise the documented key event path; test actual supported assistive technology. Touch pointer movement counts; a tap alone does not silently broaden the PRD. Study the mobile still-watching edge case before changing the calibrated window.

Track eligibility using a monotonic browser clock, split intervals immediately on visibility/focus/activity expiry, and ping every 15 seconds with at most the previous 15 seconds of qualified time. Send final partial intervals on blur, hidden, settle, and pagehide using a best-effort beacon. Freeze rendering when hidden, suspend local sound, and close listen leases; server ticking continues.

Server bounds each interval against receipt time, last accepted session interval, and a maximum 30-second delivery grace. Clamp clock skew, reject fabricated long spans, and require all flags/activity recency. The browser cannot prove human attention cryptographically; the goal is honest accounting and bounded amplification, not intrusive tracking. Never infer unreported presence from an unexpired lease or missing close message. A crashed/closed tab contributes only intervals actually accepted.

Merge overlapping intervals across tabs/devices at account level, so one human minute is at most one minute of presence. Attribute listen credit to eligible overlapping time for that bird, cap combined bird-specific listen credit to account presence, and split simultaneous different-bird credit fairly. Listen while visible but lacking the full presence conjunction can change the local mix, but contributes no drift. Ordinary closure and settle both stop presence, have no negative deltas, and retain equivalent earned attention.

### 5.3 Drift function and calibration

Persist each trait `x_j` in `[0,1]`, initialized from species-specific ranges roughly `[0.2,0.55]` with identity-seeded variation. Reserve headroom so starter birds differ without beginning near saturation. Trait seeds never regenerate.

For each day's running signal and minute integration:

- Presence dose `P = min(qualified_seconds_today / 900, 1)` initially treats 15 minutes/day as a full regular-visit dose, with a smooth saturation curve replacing the hard edge in the final calibration.
- Targeted qualified listen and accepted-offer doses contribute to relevant traits, cooldown-limited and individually saturated. Across each day their integrated contribution must not exceed 25% of that bird's actually earned presence contribution. No targeted interaction contribution exists without some qualified presence in that interval; clicking alone cannot substitute for watching.
- Persist a nonnegative low-pass exposure `q_j`: `q_next = q * exp(-dt/tau) + drive_j * (1-exp(-dt/tau))`, initially `tau = 72 hours`. Choose minute drive normalization so a regular synthetic schedule has the intended daily integrated dose; the daily dose above is an accounting cap, not permission to count the whole day repeatedly each minute.
- Apply `delta_j = r_j * q_integral * (1-x_j)`, with `q_integral` expressed in days and positive rates initially `r_j` in `0.004–0.008/day`. Add this delta to the persisted vector. Calibrate rates, signal normalization, and renderer mappings together; these are starting constants, not asserted empirical facts.

For an executable integration, charge only the newly earned dose on each step: `dP = min(today_seconds_after/900, 1) - min(today_seconds_before/900, 1)`. Smooth saturation may replace this function only with a versioned, equivalently bounded policy. Allocate nonnegative targeted dose `dI_j` from qualified listen/accepted-offer intervals, using per-bird cumulative budgets that ensure `sum(dI_j)` is at most `0.25 * cumulative_dP` across the day. Trait-specific weights allocate that small budget; presence remains each trait's dominant signal. A midnight change resets dose caps, never vectors or filter memory. For step duration `d` in days, use constant drive `u_j = (dP + dI_j)/d` over that step and `tau = 3 days`. The exact filter solution is `q_next = q*exp(-d/tau) + u_j*(1-exp(-d/tau))`; its step integral is `A_j = q*tau*(1-exp(-d/tau)) + u_j*(d-tau*(1-exp(-d/tau)))`. Compute `delta_j = (1-x_j)*(1-exp(-r_j*A_j))`, a stable nonnegative version of the small-step delta above. Use numerically stable exponential functions for tiny steps. Persist the charged dose, filter, vector, and cursor together. This makes one full earned daily presence dose integrate to approximately one exposure-day over time without treating a cumulative daily counter as a fresh signal each minute. No-input filter decay integrates only its finite previously earned exposure tail.

Presence raises all five traits. Listen predominantly raises social warmth and vocal frequency; approaching an offer slightly raises boldness, and accepting/investigating slightly raises curiosity. Plumage depends on sustained presence, not any gift type. Settle changes immediate mood/quietness only; neither settle nor absence applies a negative personality delta. No trait decay term exists. The filter may decay when absent, causing a finite residual positive drift from earlier earned attention, as the server-tick PRD requires; it cannot invent new attention or continue linear growth indefinitely.

Recent-attention modulation is separate from durable traits. With little recent owner attention birds remain healthy, keep their intrinsic baseline calls, and use calmer ambient expression; greeting intensity/frequency after the first small required noticing gesture can be lower. Absence may not drive wary mood, reduce plumage, or lower vocal-frequency traits. The return still contains one gentle noticing gesture even after weeks away. Treat the quietness requirement as a short-term expression choice, not a disguised negative personality update.

Before approving coefficients, simulate 1, 7, 21, 90, and 540 days using synthetic accounts: regular 15-minute visits, one long visit, 2-day background tabs, two devices at once, offer spam, listen only, mute/captions, no visits, and two-week absence after regular presence. Require nonnegative bounded traits, presence-dominant contributions, no visible single-session jump, measurable day-7 deltas (initial instrument gate: at least 0.005 on relevant synthetic traits), and recognizable week-3 expression changes. Validate visibility through consented qualitative studies with rendered synthetic trajectories, including blind audio signature matching and reduced-motion/narration versions. Never compute average real-user drift or mine private event logs to tune the model.

### 5.4 Mood, perch, weather, and bird-to-bird behavior

Use a seeded semi-Markov process with mood-specific minimum dwell times (initially 3–10 minutes), gradual transition probabilities, and a roughly daily relaxation toward a local-time baseline. Inputs: persisted personality, accepted recent interactions with decaying influence, current weather, and peer calls/moods. Dawn favors alert/curious, day content/curious, dusk/night drowsy; the nocturnal species retains an active low-rate profile. An offer can nudge content/curious immediately through its admitted descriptor, then be integrated on tick. A wary bird can approach later, and a drowsy bird may ignore a seed without failure copy.

Rain occurs about two or three times per synthetic week for short 3–8-minute windows; gentle wind about as rarely, with low amplitude. Seed/schedule server-side; do not query real weather or require location permission. Rain briefly reduces call density in presentation without changing the vocal-frequency vector; wind shifts alert/wary probabilities slightly. Relax weather influence after the event, not as a persistent penalty.

Perch selection uses three zones and collision-free slots. Boldness favors front, wary mood back, warmth proximity to another bird. Each move has start/end poses and a timed path; maintain clearance and never crop or stack indistinguishably on narrow layouts. Peer responses occur after a small randomized delay with capped probability and refractory periods. A wary/alarm influence can spread to a neighbor but cannot chain into a perpetual alarm storm. A chorus begins when two or more birds' runtime call windows overlap naturally; cap overlapping voices and refrain from synchronizing every bird.

### 5.5 Greeting and interaction resolution

On owner navigation/visible return, compute absence from canonical owner attention/session activity, not from a client-supplied timestamp. A quick return produces a glance, longer absence a more orienting head tilt/approach or varied call. Select the first bird with weighted boldness/warmth, mood, and recent greeting history; a wary/drowsy bird is less likely but not excluded. A fresh bounded greeting lease prevents parallel tabs from eliciting simultaneous greetings while each arrival can show a modest natural glance. Return a timed descriptor fast enough for a noticing action in 1–2 seconds. Use continuous parameter variation in timing, head angle, path, call contours, and pause lengths rather than a fixed set of clips. Store recent descriptor fingerprints to avoid exact consecutive repetition.

At most one bird initiates; any peer response is later and irregular. There is no staged whole-aviary arrival performance. Greetings do not reset existing poses/moods, refill trait doses, create a notebook entry on every visit, or expose absence lengths.

Offer admission places a seed or still pool at a safe front-scene location, or synthesizes one library song-fragment motif. Pick eligible receiving birds based on nearby position/mood/curiosity; reserve cooldowns before returning the descriptor. Responses include approach, hesitant delayed approach, watching, drinking/bathing, joining/calling against a fragment, or quiet ignoring. The song fragments themselves are synthesized from a small motif library, not recorded tracks. Expire offered scene props quietly. Repeated gestures during cooldown get calm contextual disabled-state copy in the offer panel, no countdown challenge or negative mood.

Settle immediately previews a 4-second lighting shift and quiet mix, ends the presence interval, and admits a small mood-quieting signal. The five-second undo is validated against the server-admitted settle timestamp/event ID. Pointer click anywhere in the scene reverses it in that window; Enter/Escape provides an accessible equivalent. After five seconds, explicit scene interaction is re-engagement rather than undo. Settled clients do not count passive pointer motion as renewed attention. Re-engagement clears the scene override via an event; natural local-time lighting resumes smoothly. Settling one device does not retroactively delete other devices' accepted presence; each settled session stops its own future intervals until it actively re-engages.

## 6. Snapshot sync and consistency

Canonical vector/mood writes are serialized per aviary. All clients read one committed record, and command retries are idempotent. No API accepts absolute personality values, there is no client merge, and no last-write-wins personality path. User-edited name/settings conflicts use revisions; those small explicit conflicts do not threaten drift.

Maintain a snapshot store in memory on the client with current canonical/projection version and server-clock offset estimated from round-trip bounds. Sample pose/call programs on that timeline. Lower versions from reordered HTTP responses are discarded; monotonic projection versions distinguish reaction admissions between ticks. At a new snapshot, interpolate compatible transitions over 300–800 ms instead of teleporting. If a long gap crosses many actions, jump directly to the current program's phase with a restrained pose blend, not an accelerated replay of unseen history.

Pull on visible resume, focus return where relevant, a render-frame gap above 2 seconds, network recovery, and each visible keepalive; debounce duplicate triggers. Hidden tabs do not keep polling or rendering. A resume fetch happens before scheduling new calls, cancels expired audio/animation jobs, and never replays old greeting/offer events. Canonical state can be at most one tick behind real time in healthy operation; the client projection bridges idle motion, not authoritative mood updates. Surface prolonged stale state as a matter-of-fact system issue; do not hide a prolonged outage behind endlessly invented activity.

A visitor uses the identical render projection and ambient synthesis, with owner controls and all interaction emitters absent. The visitor's local captions/reduced-motion/audio controls are allowed accessibility settings, not host interactions. No listen-in, greeting, offer, settle, host notebook, or presence endpoint is wired in the visitor module. Authorization also rejects forged writes independently of UI removal. Revocation is effective server-side at commit and appears at the next authorized pull, ordinarily within 10 seconds for a visible visit. Cancel the scene/audio and clear sensitive cached projection then. Recheck after hidden/suspended time before revealing birds. A revoked bearer cannot receive a stale authorized 304.

## 7. Frontend scene and rendering pipeline

### 7.1 Scene composition and first paint

Use layered SVG for the initial six-species/two-to-seven-bird scene, with quiet sky/foliage in the back, three semantic perch zones on the middle plane, and bounded foreground ornaments. A small renderer owns transforms and pose blending outside application component reconciliation. React or an equivalent lightweight component shell may own top bar/dialogs, but animation frames must not trigger a component-tree render. Do not introduce a large game engine or WebGL dependency to draw seven birds.

An authenticated HTML response embeds a safely escaped compact snapshot and initial SVG at its current server phase. Static assets are edge cached with content hashes. Owner/visit HTML is private and authorization-checked; it can be delivered through the edge without cross-account shared caching. Critical first-bird SVG paths, initial pose, and essential styles are inline, so the first bird does not wait for a JavaScript download, font, settings chunk, or audio initialization. The client adopts the same pose/phase to prevent a hydration reset. Use a system font for initial chrome.

At a normal return, the first frame has a preen/scan/idle pose sampled partway through its program; background and bird motion continue as the renderer attaches. No entry sequence, fade-from-static, greeting overlay, or spinner. If a current snapshot cannot be obtained yet, show a quiet soft sky field with very faint ambient cues, then place current birds without a staged arrival. After a persistent failure, provide direct error/retry copy in a system surface, rather than leaving an unexplained field forever.

The one-time initial adoption has its own allowed empty-field/fly-in: after the two birds/names commit, the first birds fly softly into their starting perches. This transition is identified by a one-use adoption-scene flag with server admission. Ordinary navigation, a stale response, rename, or restore never invokes it. Reduced motion cross-fades into initial poses instead.

### 7.2 Responsive geometry

Normalize scene geometry to three horizontal zones, then choose layout coordinates for the actual usable width/height below the top bar. Use `100dvh` with safe-area handling, no page scrolling for the scene, and no panning/zoom controls. Test 320-pixel-wide phones through wide desktop screens, portrait/landscape, and browser resizing. On narrow viewports compact spacing, reduce decorative density, and distribute seven visible birds among separated slots in three depth zones; preserve silhouette aspect ratios and target sizes. Do not simply crop a wide desktop canvas or shrink every bird below readability.

All bird bounding boxes and movement control points stay inside the visible safe rectangle, accounting for feathers, focus ring, captions, and nearby birds. Captions use collision-aware bounded placement; if neighboring captions would overlap, combine them into a quiet short caption group anchored near the callers, not a scrolling ticker. Never drag or let the user rearrange positions. Notebook/settings may scroll within their own accessible panels; the single-scene no-scroll constraint does not prevent indefinite notebook access or usable zoomed text.

### 7.3 Motion programs

Each bird rig supports finite sets of meaningful poses plus continuously parameterized micro-motion: small head orientations, breathing/body-weight shift, preen strokes, scan pauses, fluffed drowsy posture, and attentive tilts. Seed action intervals and vary interpolation envelopes so programs do not expose a looping cycle. Motion amplitude/frequency follows the server's mood-shaped program and bounded appearance projection. Calls can trigger head/throat poses from the same timed descriptor used by synthesis.

One `requestAnimationFrame` loop reads the projection and writes batched transforms, with precomputed paths and no per-frame allocations in the hot path. Maintain bounded ornament pools (initially at most 8 visible leaves/feathers); their random spawning is client-local and cannot generate weather/mood or notebook facts. Gentle parallax is time-based low-amplitude movement, not mouse-tracking spectacle. Ambient lighting interpolates the server's local-time phases and infrequent weather. No thunderstorm, flashes, dramatic rain, or attention-seeking event.

When hidden, cancel rAF and ornaments, suspend local sound, and release transient audio jobs; when visible but unfocused, rendering continues without counting presence. On resume, sample the fresh current state rather than resuming a stale animation clock. A 30-minute scene must remain as smooth as its first minute.

### 7.4 Chrome and reduced motion

Top bar: account/settings, accessibility, notebook, offer, and the explicit settle affordance. Controls have accessible names; scene birds have no visible nameplate/tooltip by default. After about 4 seconds of pointer/key stillness, lower top-bar opacity only when it contains no focus, hover, open menu, or error action. On pointer movement, keyboard activity, tap, or focus-in return full opacity before the user must act. A narrow screen has comfortably sized icon hit areas; no hover-only interaction.

Maintain text and icon contrast even in the faded state using stable surfaces/opacity floors. Keyboard focus prevents fade altogether. On touch-only devices keep sufficient visible controls rather than requiring an invisible first target. This is an accessibility-bound interpretation of “nearly transparent,” not permission to make controls unusable.

Reduced motion is a separate renderer mode selected before the first meaningful frame from `prefers-reduced-motion` or stored opt-in. Use slow 3–6-second cross-fades among carefully composed still poses, cross-fades between perches instead of flights, no moving leaves/feathers or parallax, and slower light changes. Avoid ghosting entire birds for long periods with well-designed pose blends. Mood, identity, response timing, call quality, notebook, and presence remain identical. Listen-in/offer/settle remain clear through prose/captions, pose choices, and restrained lighting. A runtime preference change switches without a burst of queued motion.

## 8. Procedural audio and caption runtime

### 8.1 Call grammar

A species grammar defines motif classes (rise, paired tones, trill, low repeated phrase, alarm-like short contour), allowed note intervals, register, spectral envelope, syllable/gap ranges, and controlled modulation. A bird's stable seed picks a recognizable sub-signature: characteristic interval pattern, timbre/formant profile, register range, and phrase shape. Store the definition version and seed permanently. Mood changes call duration, pauses, contour tension, and intensity within that signature's bounds; personality-derived presentation chiefly affects call timing and response propensity. Variation cannot make a known bird sound like a different species or identity.

Server descriptors specify grammar/version, seed, absolute start window, motif choices, and constrained expression overrides. A deterministic client expander yields an intermediate representation of syllables: frequency curves, duration, gaps, envelope, texture, and location. The audio scheduler and caption generator both consume this same expanded representation, so a caption describes the phrase actually synthesized. Do not store one canned caption per species or derive captions from a guessed mood label.

Temporal recipes include natural peer-response offsets and chorus membership. Owners and visitors at the same time expand the same authorized recipe; neither independently decides that a chorus has happened. Local audio suppression does not change its semantic call descriptor or the host simulation.

### 8.2 Synthesis and mixing

Use one lazily resumed AudioContext per page, reusable periodic waves and procedural noise buffers, and a graph consisting of bounded voices, one gain bus per bird, soft spatial placement by perch, a master bus, and a transparent peak limiter. No downloaded calls or recorded fallback. Synthesize with oscillator frequency ramps, filtered procedural breath/noise, and short amplitude envelopes to avoid clicks. Start with native WebAudio nodes; introduce a small AudioWorklet only if the target-browser profiling proves necessary. Worklets cannot be a prerequisite for the first bird.

Schedule on the AudioContext clock with a short lookahead (initially 100 ms) and a timer that fills about 250 ms ahead. Convert server times through the measured clock offset. Never schedule seconds of stale calls across a suspension, and never fire a burst on regain of focus. Mid-call join, where supported by the descriptor, starts at the current envelope phase; otherwise wait for the next call and keep its visual/caption observation consistent.

Limit active voices to an initial 12, with at most 3 loud simultaneous bird phrases and other birds' lower-level ambient contributions. Space tonal registers, stagger envelopes, and use independent procedural detail to avoid doubled fixed loops and phase artifacts. Preserve recognizable signatures rather than applying broad pitch randomization. Test night mode for audible but gentle nocturnal activity. Master limiting and volume settings prevent unexpected harsh calls or mobile loudness jumps.

Listen-in ramps the attended bird toward its comfortable foreground level over about 1.5 seconds and the other bird buses toward a low but nonzero ambient floor (initially roughly -12 dB relative to their ambient level). Disengage uses the same ramp back. Switching to another bird first releases the old target and ramps the new one; repeated actions cancel/reschedule gain automation from its current value, not from a hard zero. Double-click/tap same bird, empty-space click, Tab away, or Escape ends it. Mute is a master device setting, never an instruction to silence other birds as a listen mechanic.

Settle ramps master/ambient density gently and warms lighting; undo reverses the existing automation smoothly. No sound reveals a status counter, error, achievement, or visit. Audio muting cannot reduce personality or block accessible drift participation.

### 8.3 Lifecycle, silence, and quality gates

Autoplay-denied/suspended contexts, unavailable WebAudio, hardware errors, and rejected resume produce graceful silence with captions enabled by default for that session. Keep a clear sound setting in accessibility/account controls; a permitted user gesture may resume the context without an announcement. Do not repeatedly retry failing audio every frame. Show a direct status only within relevant settings if sound remains unavailable. Full procedural quality returns if the problem clears; never switch to audio files.

Disconnect and release finished oscillator/filter nodes; reuse envelopes, waves, arrays, noise buffers, and descriptor pools where feasible. Bound caches by grammar version and voice count. Pagehide/disposal cancels scheduler timers, pending gains, listeners, and worklet jobs. Track context count and node high-water mark in synthetic tests, not per-bird production analytics.

Audio designers produce blind listening tests for all six signatures, mood transitions, two-to-seven-bird choruses, headphones/speakers, mute/caption mode, and repeated 30-minute sessions. Ship only if calls feel bird-like without exact repeats, Pip remains recognizable across moods/drift, and the seven-bird chorus retains individual identities. If recognition fails at seven, lower the release rollout ceiling while improving mixing; the product's absolute engine cap stays seven, not an excuse to add more birds.

## 9. Accessibility and notebook as designed product surfaces

### 9.1 Semantic access and focus

Pair the visual SVG with real positioned semantic bird controls, not a canvas-only application role. There is one Tab entry point to the bird group and roving `tabindex` among birds. Arrow keys move in stable spatial/adoption order; Enter engages/toggles listen-in; Escape exits; Tab leaves for the next surface. Controls use bird names and species plus an action description, not raw personality values or constantly changing mood/state labels. Screen-reader narration conveys the scene separately. Expose pressed state for the listen interaction without using disallowed product vocabulary such as “solo” in accessible labels.

The top bar precedes the scene in normal keyboard order. Offer opens via a documented `Alt+O` shortcut when no text field is being edited, with `aria-keyshortcuts` and an equally usable button. Do not rely on an unmodified letter that interferes with assistive-technology reading commands. The offer chooser has keyboard navigation, Escape dismissal, and focus return. Notebook/settings dialogs preserve sensible reading order, trap focus only while modal, restore the invoking control on exit, and never conceal focused controls. Support coarse pointers with at least 44 CSS-pixel targets.

Use a dual-tone focus ring that remains distinguishable across dawn, night, rain, plumage, and caption backgrounds, with at least 3:1 focus/non-text contrast. Body text/captions/settings achieve at least 4.5:1, larger text at least 3:1, with automated checks on actual composited colors. Test 200–400% zoom and enlarged text; when controls/panels need more vertical room, allow their internal document flow rather than clipping account actions to preserve a rigid scene height.

### 9.2 Narration cadence and prose

Generate short naturalist paragraphs from the exact render projection and admitted user-event facts using a deterministic template grammar shared with notebook voice rules. At idle choose a 45-second cadence (within the specified 30–60 seconds), skipping unchanged or redundant prose. Describe named birds or distinguishing species, spatial setting, current meaningful pose/call, light/weather, and a small particular. Do not announce every frame, numeric traits, internal mood enum, event ID, or machine state list.

Use one polite atomic live region plus a stable read-on-demand scene paragraph. Maintain a queue capped at two observations; prioritize a greeting/offer/settle observation promptly over a stale idle paragraph while preserving ongoing speech. Coalesce peer responses into one observation. Avoid an assertive interrupt for ordinary bird action. Errors that block an explicit operation use separate clear system messaging. Opening the notebook or a settings form pauses unsolicited idle narration until the user returns, so it does not overwhelm reading; retain the current paragraph for on-demand access.

Narration is enabled and operable without attempting to detect screen-reader software. Provide pacing/quieting controls in accessibility settings. Validate full workflows with VoiceOver/Safari and NVDA/Firefox or Chrome; automated ARIA checks cannot establish whether the accessible aviary feels alive. No screen-reader view is a stats interface.

### 9.3 Captions

From the actual expanded call phrase, describe contour, syllable count when natural, trill/repetition, loudness, pauses, and spatial source in concise lowercase prose. Show near the calling bird with gentle opacity transitions, protected high-contrast background, and no flashing. Keep simultaneous captions legible; expiry follows call duration plus a brief reading grace. With reduced motion use restrained opacity changes; with graceful-silence fallback the same semantic phrase continues through captions, so sound loss does not erase calls.

Do not send every caption to the screen reader's live queue as well as narration. Visual captions and the slow narration are coordinated alternatives; caption prose can be exposed on demand. Call captions are optional except automatic session defaults when audio cannot work, and users can still adjust them explicitly.

### 9.4 Sparse, grounded field notebook

Use a server-side rule engine over simulation facts, never an external language-model request carrying private events. Candidate facts include unusual first-greeter order, a particular bird resting on a new perch, sustained quiet, characteristic weather response, and naturally emerging chorus variation. Keep per-aviary bounded observation memory (such as last week's first-greeter distribution) solely to drive that aviary's own notebook. It is not exported to analytics or a behavior dashboard.

Initially allow one ordinary entry every 3 days and no more than one noteworthy extra in a 24-hour window, with a soft rolling cap of 3 per week. Record only candidates with sufficient specificity and novelty; do not manufacture a daily quota. Busy users can therefore have more relevant entries without receiving a feed. Quiet background simulations may make occasional aviary observations, but never entries about user absence, attention frequency, or “every day this week.”

Write prose from validated facts with bird names resolved at observation time, lowercase present tense, no praise, no gamification. Store provenance only in the private simulation database. Include date headings; no “session started” log strings. Entries cannot be edited, deleted individually, annotated, or hidden by the user, while account deletion removes them as part of privacy lifecycle. Cursor pagination and virtualization keep arbitrary history available indefinitely without retaining every page in memory. Reuse entry IDs to avoid duplicates after retrying a tick.

## 10. Identity, visiting, privacy, and account lifecycle

### 10.1 Authentication and mail

Generate high-entropy opaque tokens; store digests, never plaintext. Consume magic links atomically within 15 minutes; the same token cannot issue two sessions under parallel requests. Rate limits key to a sensitive private lookup rather than putting an email in metrics/logs. Use secure same-site HttpOnly cookies, session fixation protection, TLS, redacted structured logs, and scoped database service roles. Session expiry initially 30 days rolling with an absolute 90-day cap; device revocation takes effect on the next request. A session timeout cannot lose an already committed interaction, because event retries deduplicate after reauthentication.

Email verification is a distinct one-use challenge. Pending email changes remain private account fields until verification; commit atomically, invalidate the pending challenge, and preserve synthetic account/bird IDs. A visitor identified by that account UUID follows the verified current address in log display rather than embedding copied PII in every invitation row. Record original invitation destination in the identity's constrained verification history only if operationally necessary, with bounded retention; never use it as a reference key.

Mail providers receive only the address and required transactional text/link, never vectors, moods, names, notebook, or per-bird interactions. Export email contains an opaque access link, not the exported JSON. Use short-lived in-memory recipient access and content-redacted mail-job diagnostics.

### 10.2 Invite lifecycle and read-only enforcement

Host settings list outstanding invitations and completed visits, newest first. A deliberate email entry creates one invitation, unused for at most 30 days. Redemption requires the mailed bearer link's one-time secret and creates a scoped visitor session; possession is the v1 inbox-possession proof, not a claim of immutable human identity. Token forwarding remains a security risk; mitigate with short-lived active sessions, high-entropy secrets, host revocation, no URL referrer leakage, and confirmation before consumption. Do not add mandatory owner signup merely to visit.

An active visit session lasts at most 2 hours; after it ends, repeated use of the consumed invite fails, and another visit requires another deliberate invitation. This resolves otherwise unspecified active-link duration while preserving one-time, nonpermanent access. GET/pull authorization checks active invitation status and deletion status before reading/caching the projection. Visitor-access recording lives outside the simulation event table. Duration is approximated from first/last authorized pulls, capped by expiry; it is not owner presence or behavioral analytics.

Revocation commits immediately and prevents all future snapshots; the next visible pull terminates the scene and sound. The settings list changes in place without a toast, badge, or confirmation email. If optional visit notifications were explicitly enabled, emit at most one plain visit email per redeemed invitation via outbox, naming only the visit and settings access, with no bird state or attention-driving language. Default accounts emit none. Guests cannot subscribe to host changes, receive notebook copies, trigger greetings, or generate host drift. Show the host's current birds, light, weather, and call signatures without a promotional rendering.

### 10.3 Privacy storage and telemetry firewall

Per-bird events exist only in the simulation store for that account's engine. Do not route them through client analytics SDKs, external crash replay, third-party event collectors, training systems, recommendation pipelines, or population drift dashboards. Do not log request bodies from event/snapshot/export routes. Do not send screenshots or DOM session replay of the aviary to observability services.

After transactional consumption and a 7-day bounded recovery window, delete raw interactions; retain only the persisted vectors, private bounded filter/observation memory, recent dedupe keys (up to 30 days), and account-owned notebook. Presence intervals are compacted into recent private tick/day accumulators and pruned once no longer needed. Raw retention is a documented initial policy, not archival user behavior. Keep notebook history until account deletion. Visit history remains for sharing transparency until deletion; it cannot become a permanent friends list or leaderboard source.

Operational metrics use explicitly allowlisted fields, bucketed durations/counts, browser-family/version bucket, coarse device class, deployment version, and coarse geographic synthetic region. No email, account/aviary/bird/session UUID, name, vector, event kind/bird action, precise location, or URL token enters RUM. An isolated first-party receiver strips network identifiers before aggregation. Restricted account-level incident logs may contain synthetic UUID and error code for permitted troubleshooting, never interaction payload/state; they expire within 14 days and are included in deletion cleanup. Analytics roles have no read privileges on simulation tables or replica, and the outbox cannot publish event contents to metrics.

The privacy settings link names the exact aggregate categories and explains the exclusion of bird/interaction state. Measure and test this boundary as code/schema/network egress, not just policy prose.

### 10.4 Export and deletion

Export reads a consistent canonical database snapshot, including pending settings/name edits but not speculative client state. Produce a versioned JSON document with birds/IDs/names/species, mood, current sealed personality capsules, notebook entries, account settings, creation/export timestamps, and schema/engine versions. The capsule encrypts the actual current vector using server-only authenticated encryption; it is not a recomputation from logs. Explain the capsule plainly in account export help without revealing numbers. Do not include session/auth/invite secrets, recipient email lookup digests, device cookies, other users' private details, or raw interaction history. Email a private download link that expires after 24 hours and is invalidated after download; delete object artifacts after expiry. Revoke outstanding export links upon account deletion.

At deletion request, mark the account immediately, revoke owner/visitor access to the live scene and all invitations, suspend new commands/exports/mail, and show the direct 30-day recovery surface after sign-in. Pause its simulation jobs during the recoverable deletion window rather than continuing to collect any inferred presence; persist existing vectors unchanged. Recovery before the exact UTC deadline restores the account and one aviary, resumes ticks/mood progression without attention invention, and retains identities. Revoked invitations stay revoked and must be deliberately reissued.

At 30 days, run idempotent hard deletion of birds/vectors/events/notebook/settings/sessions/invites/visit records/account-linked operational logs/jobs/export objects and search indexes, including visitor-only identity rows if no other valid reference requires them. Delete account-specific data-encryption keys and mark cleanup complete in a nonidentifying operations counter. If deleting a visitor identity referenced by another host's visit log, redact the email/identity and retain only a nonidentifying historical “deleted visitor” fact where necessary for the host's transparency; remove the deleted-account linkage.

Use per-account encryption for sensitive simulation payloads and exports so backups retained for disaster recovery become unreadable after key erasure. Bound backup retention to at most 30 days and document that encrypted inaccessible remnants age out physically; require a deletion registry/key-erasure checkpoint before any restored service accepts traffic. A database restore must not resurrect a deleted account, session, or visit grant. Deletion cleanup metrics never label the deleted UUID. Test recovery on both sides of the deadline, retry after partial cleanup, and restore from a pre-deletion backup.

## 11. Performance budgets, operations, and evidence

### 11.1 Enforced budgets

| Requirement | Engineering allocation and verification |
| --- | --- |
| Initial gzipped JS below 2 MB | Set an internal target below 200 KB for first-scene JS and below 500 KB including immediately needed grammar; reserve remaining hard-cap space instead of filling it. Build analysis counts every initial-request script, inline executable code, and preload. Split account/accessibility/notebook/visit management chunks. No audio-file downloads. |
| First bird below 500 ms | Define from navigation start to first painted non-placeholder bird. Inline real pose/SVG/state in private HTML, aim for edge-delivered TTFB below 200 ms and minimal parse/paint. Gate representative mid-tier mobile 4G runs on p95 below 500 ms across agreed geographic coverage, report cold/warm separately. A hard requirement cannot be guaranteed on arbitrary network outage; persistent misses block rollout rather than changing the metric. |
| 60 fps for a 30-minute idle scene | On a five-year-old midrange laptop, target animation main-thread work below 4 ms/frame and total frame time within 16.7 ms, p95; minimize missed-frame rate and sustained runs of missed frames. Test seven birds, weather, call captions, notebook opening, and normal/reduced motion. |
| No memory growth over 30 minutes | After a 5-minute warm-up, compare repeated post-GC heap/native/audio measurements through minute 30 and inspect retained-object counts. Require no sustained positive slope beyond harness noise; initial tolerance band is 5% with a 1 MB floor, requiring investigation rather than accepting a leak. Audio nodes/contexts, caption queues, ornaments, timers, notebook pages, and listeners remain bounded. |
| Tick latency p99 alarm above 5 seconds | Track compute/transaction duration and separately scheduled-due-to-commit lag. Alarm either p99 over 5 s for 5 minutes, page on sustained backlog or missed ticks, and set an internal healthy target below 100 ms per aviary step. |
| Small snapshots | Seven-bird uncompressed payload below 12 KB, typical two-bird lower; bounded reaction horizon and no notebook payload in scene polling. |

The 500 ms budget is more demanding than the 2 MB maximum alone suggests; the inline first paint is essential. Rendering actual private state from edge HTML must never rely on public caching. Test cold authentication/session checks and DB latency rather than measuring only a cached shell.

### 11.2 Instrument from day one

Use aggregate-only RUM for navigation/TTFB/first-bird, frame-time buckets/missed frames, audio-context success/error classes, API latency/status, snapshot freshness buckets, and anonymized session-duration histograms. No dimension represents a specific account, bird, event, offer preference, or drift rate. Track bundle/asset sizes in build artifacts. Monitor tick execution/lag/backlog, dedupe failures, transaction retry rates, export/deletion job completion latency, and mail failure counts with payload-free operational measurements. State-bearing invariant failures produce a code/version/count, not a vector dump.

Run scheduled synthetic browsers in common geographies for supported browser families, phone and laptop profiles, first paint, greeting latency, audio availability, 30-minute memory stability, and keyboard/narration flows. Use synthetic aviaries with public fixture identities and generated histories only. Separate synthetic engine calibration reports from production metrics. No engagement/retention target, daily-active-user streak report, adoption rank, average account personality chart, or “most visited” metric is a release objective.

### 11.3 Availability and recovery

Apply health checks to API authorization, primary database, due-job scheduling, and outbox consumers. If the tick backlog rises, expand worker capacity by UUID partition and reduce noncritical export throughput; never drop events or suppress inactive-account ticks to improve a chart. Estimate base tick capacity as `active_aviaries / 60` steps/sec plus outage catch-up headroom; load-test at twice expected launch accounts and full seven-bird work per step. Polling capacity is visible clients/15 plus visitors/10 requests/sec; ETags save payload but not authorization.

Keep read replicas out of event admission/next-tick reads unless their consistency is proven; primary reads are the v1 safe default. Database migrations use expand/backfill/verify/contract, preserve vectors/IDs and engine cursors, and are reversible until contraction. Pin active bird grammars and engine configuration by version. A release rollback must not reapply consumed deltas or regenerate species seeds. Maintain encrypted backup/PITR drills, account deletion restore drills, and a worker outage replay exercise before external beta.

## 12. Build sequence and validation gates

Ship in dependency order with cross-disciplinary design/a11y ownership from the start. Each milestone has executable evidence and product review; a working checkbox alone is insufficient.

### Milestone A — contracts and synthetic aliveness spike

Engine/backend lead defines schema, projection/event types, invariants, exact presence aggregation, seed/version policy, and deterministic minute-step library. Visual/audio designers deliver six stable silhouettes/signatures and pose/prose grammar primitives; accessibility lead prototypes narration/reduced-motion at the same time. Build a synthetic two-bird slice showing mid-action first paint, a varied greeting, procedural calls, captions, and slow still-pose cross-fades. Choose SVG versus any alternative only after measuring this slice on target hardware.

Exit evidence: no repeated greeting fingerprint in a seeded 1,000-return exercise, blinded call identity agreement across moods meets an initial 80% recognition gate after familiarization, readable WCAG AA controls, first-bird/JS budgets demonstrated with authenticated fixture HTML, and no personality numbers in client projection. Calibrate identity recognition thresholds through the design study rather than silently treating them as universal facts.

### Milestone B — canonical persistence, auth, and tick correctness

Implement owner identity/session/email flow, transactional two-bird adoption, persisted vectors/moods, minute scheduler, ordered event admission, private HTML snapshot, periodic pulls, and exact presence collection. Build real cross-device/session test tooling with synthetic accounts. Establish typed logging/telemetry firewall before adding product interaction logging.

Exit evidence: simultaneous workers/duplicate jobs/retries advance once; a process crash between event read and commit loses/repeats no delta; concurrent tabs do not double presence; all seven combinations violating one of the three presence predicates count zero; sleep/hidden/background/clock-skew cases pass; long absent accounts continue ticking; session expiry never overwrites state; backups restore bird IDs/vectors/cursors. Auth scanner/replay/rate-limit/device-revocation tests pass.

### Milestone C — complete owner session and accessibility

Implement all offers/cooldowns, listen gain ramps, settle/5-second undo/re-engage, rename, timezone/day/night/weather, bird-peer responses, sparse notebook generation/pagination, and full keyboard/narration/caption/reduced-motion paths. Immediate descriptor admission bridges minute ticks. Lazy settings UI uses direct system prose; product review audits every visible string for announcement/gamification drift.

Exit evidence: mood persists through reopening rather than snapping; offers reserve cooldown atomically across devices; ignored offers produce no penalty; settle and closure yield equivalent retained attention; undo validates its own generation and deadline; voices other than the listened-to bird never reach zero solely from listen-in; seven birds stay visible on a 320 px viewport; screen-reader workflows are reviewed with users, not just automated scans. Notebook remains sparse after both quiet and hyperactive synthetic schedules.

### Milestone D — visits and account lifecycle

Implement opt-in invitations, one-use redemption/scoped visitor session, settings visit log/revocation, default-off notification policy, email change, requested export, soft deletion/recovery/hard purge, and backup deletion protection. Reuse the ambient renderer rather than creating a guest-only prettified scene.

Exit evidence: visitor stays render-only even when forging each owner event/API; an hour visit changes no host presence/filter/traits; revocation/expiry blocks next pull and stale ETag/cache access; no greeting on guest entry; no default notification/badge/mail on a visit; invitation expiry/redemption races pass; export has a coherent canonical version, no plaintext trait numbers or secrets; deletion purges/cryptographically erases account data and a restore cannot reintroduce it.

### Milestone E — longevity, load, and release candidate

Run accelerated synthetic 7/21/90/540-day trajectories, seven-bird chorus studies, cross-browser two-device chaos tests, sustained tick catch-up/load, and 30-minute memory/audio/render suites. Complete manual accessibility/voice review and browser-support fallback. Last two major Chrome/Safari/Firefox/Edge versions are the supported test matrix; unsupported browsers receive direct explanatory copy, not a heavy compatibility bundle.

Exit evidence: all performance budgets pass, p99 tick instrumentation/alarms are live, privacy egress scans reject seeded sensitive fields, drift targets satisfy the synthetic/qualitative gates, migration/rollback/restore runbooks are rehearsed, and no blocking keyboard/narration/reduced-motion defect remains. Do not defer accessibility to a post-launch patch.

### 12.1 Essential test design

- Property-based simulation tests cover monotonic/finite traits, capped doses, stable IDs/signatures, retained moods, deterministic seeds and tick ordering, and finite no-presence residual drift.
- Transaction integration tests inject duplicate admissions, reordered client events, concurrent offer/adoption/rename attempts, worker lease expiry, commit crashes, and database restarts. Assert stored vectors/cursors, not only response codes.
- Browser end-to-end tests cover fresh adoption versus normal return, first paint/hydration phase, focus/visibility/activity conjunction, settle/undo timing, tab suspend/resume, mute/autoplay failure, captions, keyboard focus transfer, and guest negative authorization.
- Accessibility verification combines semantic/contrast automation with real screen-reader/reduced-motion/zoom sessions, including seven-bird caption collisions and faded top-bar focus. Review whether prose preserves the actual product.
- Golden grammar/fact tests check captions against expanded calls and notebook/narration against source facts; prohibit numeric personality prose, generic event logs, welcome strings, absence blame, streak language, and needless announcement UI.
- Soak/load/privacy/lifecycle tests cover native audio resources as well as JS heap, signed-download expiry, deletion/recovery deadlines, network/request/log redaction, and unauthorized role access to simulation tables.

## 13. Rollout and calibration control

Start internal synthetic fixtures, then a small consented alpha, then staged external beta and general release (initially 1%, 10%, 50%, 100% of eligible new-owner traffic). Stage routing uses a deterministic synthetic UUID hash, not per-account behavioral analytics. Proceed only after each cohort's operational budgets hold and targeted usability review finds no regressions. Add no invitations during onboarding and no alpha gamification scaffolding that can leak into release.

All new accounts receive two birds. Independently feature-gate age-based additional adoption by release ceiling: two initially, then three/four, then five to seven after the seven-signature audio, mobile layout, accessibility, and worker-load gates pass. Test older aviaries through time-shifted synthetic fixtures instead of awarding live beta accounts extra birds for activity. Account age is actual creation age and is never reset by sign-in or deployment. Do not offer above the current tested ceiling; once adopted, a bird is never hidden, removed, or reset if a feature flag is rolled back. A kill switch can stop new invitations/adoptions or new offers while preserving the existing scene/state; it cannot replace birds.

Configuration versions include presence window, cooldowns, drift coefficients, mood transition kernel, weather rate, notebook sparsity, grammar, and renderer mapping. Changes are applied forward only, never by recalculating vectors from old history. Safety checks assert nonnegative deltas and no identity discontinuity. Calibrate through synthetic schedules and explicit studies of synthetic rendering, not production interaction aggregation. Do not A/B users' bird personalities and compare engagement. Maintain a configuration changelog and replay-compatible engine packages for already committed state.

Release operations inspect aggregate errors/performance, worker backlog, privacy-boundary checks, supported-browser first paint, and accessible workflows. User-reported quietness/uncanny calls/sync issues can guide reproduction with synthetic fixtures; retrieving a real account's private state is not an automatic debugging or analytics path. A severe incident freezes new writes safely or stops adoption invitations, with a direct system error if needed, while protecting the last canonical state and earned drift.

## 14. Principal risks and mitigations

| Risk | Detection and response |
| --- | --- |
| Drift is too fast, imperceptibly slow, or gift-driven | Simulate regular/single-session/spam schedules, inspect day-7 numerical and week-3 perceptual gates, enforce targeted-credit limits, and tune rates/visual mapping with synthetic studies. Apply future coefficients only; never reset traits. |
| Double counting or lost drift across devices | Serialize event admission/tick, unique IDs/cursors, account-time interval union, duplicate-worker/crash injection, and API role separation. Treat any vector decrease as a critical invariant violation. |
| Personality exposure through API/export/debugging | Separate projection schema, server-sealed export capsules, no deployed numeric debug panels, redacted logs, and seeded network/DOM/export scans. Accept the explicit limitation that export does not disclose readable vector values. |
| Audio sounds mechanical or signatures blur at seven | Keep identity motifs invariant, vary envelopes continuously, chorus staggering/voice headroom, blind recognition/30-minute listening review. Pause additional adoption rollout if it fails; preserve existing birds. |
| Autoplay or Safari audio lifecycle interrupts aliveness | Start with real visible birds/captions, one bounded AudioContext, permission-aware resume, cancel stale scheduling, and supported-browser fallback tests. No recorded substitutes. |
| Accessible surface becomes a state list or static scene | Narration prose review, slow coalesced queue, authored still-pose cross-fades, same-state captions, user testing before release, and focus/contrast/zoom automation in every change. |
| First-bird budget fails on cold mobile navigation | Inline actual SVG/phase and snapshot, small critical payload, near-primary auth/edge delivery, cold/warm geography tests. Diagnose TTFB separately; never measure quiet-field paint as first bird. |
| Leaks accumulate during long sessions | Bounded programs/queues/voice resources/ornaments/notebook pages, disposal hooks, post-GC/native-resource 30-minute CI, and scheduled synthetic soaks. |
| Clock/timezone/suspend causes teleporting or duplicate calls | Canonical IANA timezone, UTC tick order, monotonic client timing with offset correction, resume pulls, expire old descriptors, and compatible pose blending. No mood reset on open. |
| Tick cost grows with inactive accounts | Schedule by UUID, test full fleet cadence and twice launch capacity, bounded backlog batches, separate operational lag alarms. Do not turn ticking into client-on-demand evolution. |
| Quiet social feature expands into co-presence or default mail | Separate read-only auth context/module/storage, no visitor event writer, default-off mail tests, no discovery metrics, and product-scope review of all new surfaces. |
| Deletion/export/backups conflict with identity continuity/privacy | Consistent export snapshots, account-scoped encryption, bounded objects/backups/logs, restore-time deletion registry, recovery deadline tests, and idempotent purge jobs. Recovery retains IDs, hard deletion cannot resurrect them. |
| Spec ambiguities invite silent feature creep | Treat section 1.1 as recorded v1 decisions, review product/a11y/privacy implications before changing them, and keep explicit exclusions in acceptance criteria. No novelty feature substitutes for improving birds. |

Completion means a new owner meets two already-living birds quickly; watching has an honest slow effect; another device sees the same birds; a guest can only observe; absence is harmless; and the same quiet aliveness is available through sound, captions, narration, or reduced motion. The engineering team should implement only this defined release and validate the stated gates before adding adoption capacity or expanding delivery.
