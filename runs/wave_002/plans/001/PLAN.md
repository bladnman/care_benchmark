# Pocket Aviary — v1 implementation plan

This plan is based on `prd/1-START_HERE.md` and all nine PRDs it lists. It defines implementation decisions, delivery dependencies, and acceptance evidence for a separate engineering team. It does not require product implementation during the planning phase. Numerical tuning values below are starting configurations to validate, not measured results.

## 1. Release scope and governing invariants

Build a browser-based, single-user aviary that feels as though it has continued between visits. Each account owns exactly one aviary, starts with two birds assigned by the system, and can adopt up to seven over the aviary's lifetime. Ship approximately six species with distinct silhouettes, palettes, and procedural call signatures, including a nightjar-like species that remains active at night.

V1 includes email magic-link accounts, device-session management, email change, export and deletion; continuous server simulation; greetings, idle attention, listen-in, the three offers and settle; the sparse, indefinitely browsable field notebook; age-based adoption and renaming; responsive day/night and occasional weather; multi-device snapshots; individually issued read-only visits with revocation and a private visit log; narration, keyboard navigation, reduced motion and call captions. Accessibility ships with the primary experience.

The release has these invariants:

1. Only the server simulation tick updates persisted personality. It applies nonnegative deltas to existing vectors. An absent user cannot lose a trait, a bird, a name, or accumulated drift.
2. Every bird has an immutable UUID and call identity. Renaming, migrations, deployments and device changes preserve that identity.
3. A client's responsibilities are observation reporting and presentation. It cannot advance mood, decide drift, run a second canonical simulation, or submit trait values.
4. Presence requires visible document AND focused window AND recent pointer/key activity. Overlapping owner devices cannot multiply time. Visitor time contributes nothing.
5. The first normal aviary frame is an already-running scene. Return greetings are bird behavior, never a toast, banner, modal, counter, or textual welcome.
6. Calls are procedurally synthesized. There are no recorded calls, audio loops, downloaded call files, or recorded fallback tracks.
7. The aviary is one horizontal view with all birds visible. Controls stay in the top bar and its opened panels. Captions and keyboard focus are deliberate accessibility exceptions to the otherwise unlabelled scene.
8. A visitor receives the same canonical birds, moods, positions, weather and day/night state as the owner. Visitor credentials cannot reach any simulation-writing path.
9. Operational telemetry cannot contain bird state or owner interaction history, and cannot read the simulation database for population analysis.

Explicit exclusions: native apps; payments or tiers; passwords and SSO; multiple/shared/household aviaries; scene customization or bird placement controls; public profiles, discovery, follows, comments, chat, avatars, co-presence and mutual-visit mechanics; hunger, illness, death, suffering or maintenance schedules; achievements, scores, streaks, counters, rewards for attendance, calendars of visits or numeric personality panels; reminder, marketing or aviary-engagement notifications. No underlying ranking or engagement dataset is built for later exposure. Browser zoom for accessibility remains supported; the prohibition on zoom concerns an in-scene camera control.

## 2. Decisions where the supplied specifications leave room or conflict

These are decisions for implementation, rather than unresolved questions for a future team.

| Topic | Decision and rationale |
| --- | --- |
| Canonical local time | Initialize an account IANA timezone from the first owner browser. Persist it, allow explicit change in account settings, and use it on every device and every visit. Do not silently change it when a travelling device opens the aviary. This reconciles local time with a single canonical day/night state. |
| Four top-bar icons versus top-bar settle | Keep exactly account/settings, accessibility, notebook and offer icons. The offer icon opens a small action panel containing the three offers and a separately grouped `settle` action. Settle has its own semantics; it is not a fourth offer. It remains directly reachable from the top bar without adding a fifth permanent icon. |
| Keyboard focus versus Enter to listen | Moving keyboard focus to a bird starts listen-in, as required by the interaction spec. Enter explicitly engages it and is idempotent when already engaged; Escape ends it without moving focus. Subsequent Enter re-engages. Pointer activation on the same bird toggles it off. This avoids a focus event immediately undoing the Enter action. |
| Numeric-vector prohibition versus account export | These requirements conflict literally: `bird_engine.md` prohibits numeric disclosure anywhere while `accounts_sync.md` explicitly requires current vectors in a downloadable JSON export. Choose the narrow portability exception: include current numeric vectors only in an explicitly requested, authenticated export file. All normal APIs, UI, DOM/ARIA, notebook, narration, public debug surfaces and telemetry exclude the vector. Do not describe this exception as satisfying an absolute prohibition on all numeric disclosure. |
| No notifications versus optional visit notifications | Honor the specific social-settings exception with one optional transactional email per actual visit start. Its account setting defaults off, is absent from onboarding, and is rechecked at delivery. There are no push notifications, in-scene notifications, reminder emails or notification badges. Sign-in, invite, email-verification and requested-export mail are also necessary transactional system mail. |
| Revocation and the visit log | Remove a revoked invite from outstanding/active entries without a success notification. Preserve the historical record that the visitor previously viewed the aviary, subject to retention/deletion. This preserves transparency about past sharing while implementing the specified disappearance from the active list. |
| Browser autoplay | Attempt permitted audio playback, but never delay rendering or claim sound can precede browser permission. Where playback is blocked, use silence and automatic captions until an eligible user gesture enables WebAudio. No autoplay permission modal interrupts the first bird. |
| Top-bar fading and accessibility | Fade the bar's decorative framing nearly away after inactivity; keep actionable icons at their required non-text contrast floor. All displayed text retains AA contrast. Keyboard focus, an open panel and touch use keep controls fully visible. Accessibility takes precedence over making functional controls literally invisible. |
| Additional adoption | Use initial eligibility ages of 90, 180, 270, 365 and 540 elapsed days for birds three through seven. Age is wall-clock time since aviary creation, including absence. A year-old aviary can have six birds. These dates are internal configuration, with no progress display, countdown or attendance requirement. |
| Muting and personality | Muted presence is worth the same as audible presence. Sound preference affects rendering, and explicit listen-in supplies its normal attention signal even when calls are captioned. The brief's mention of muting does not justify a negative trait effect or disadvantaged accessible experience. |
| Missing visual-design document | Do not depend on an unprovided design-system file. Establish the palette, pose sheets, focus treatment and measured contrast tokens in the first delivery stage using the supplied aesthetic constraints. |

The delivery lead records these decisions with the implementation. They are sufficient to proceed without more source documents or product questions.

## 3. Architecture and boundaries

Use a TypeScript web application, a small server application and a separately deployable simulation worker from the same codebase. Use PostgreSQL as the canonical transactional store. Start with a modular monolith: identity, command intake, snapshot projection, simulation, notebook, visits and account lifecycle have explicit module interfaces, but do not need separate network services. A database-backed job/outbox mechanism handles tick dispatch, transactional mail, exports and deletion. Add independently scalable worker processes before introducing an additional broker.

The client shell can use React for forms, navigation and panels. The scene renderer is a small imperative SVG controller operating outside React's per-frame state updates. Audio has a separate bounded WebAudio scheduler. Both consume a common immutable presentation model with timestamped tracks and call instructions. The renderer does not import the simulation package. The package containing personality storage, drift and behavioral decisions is server-only and is excluded from client builds.

Request flow:

1. The authenticated document route obtains a current owner snapshot through the server boundary and returns minimal HTML, critical styles, an inline first scene, and a safely serialized presentation snapshot. An edge runtime can deliver this response close to the user; it must authenticate each request and must not place personal HTML or snapshots in a shared CDN cache. Immutable code, species artwork and grammar definitions use ordinary CDN caching.
2. A small critical scene controller takes over the existing SVG at its current timeline position. The larger shell, account panels, notebook and audio worklet load independently.
3. The browser reports owner resume and interaction events, and pulls fresh snapshots. These requests reach the primary data boundary; neither a device nor a stale read replica becomes an authority.
4. Every approximately 60 seconds, a server worker advances each due aviary, including accounts without connected clients. It consumes admitted events in order, updates vectors and moods, extends scene/call timelines, and occasionally creates a notebook observation.
5. Snapshot projection reads the persisted state plus any durable, newly admitted short reaction cues. These cues allow an immediate offer/greeting/settle response before the next slow tick without letting the client mutate mood or personality.

The command path may author a short presentation cue using current canonical mood/personality under the aviary lock. The cue and its chosen outcome are persisted with the event; later snapshots and the tick reuse them. This is not a second model: command intake neither advances simulation time nor writes a vector or mood. The tick is the only transition function, and it consumes the recorded outcome once. Read endpoints do not invent interaction events.

Use server-authored, absolute-time bird action tracks, call intents and weather intervals with about a 120-second lookahead. These are visual/audio instructions, not a second durable bird history. Clients interpolate and synthesize those instructions; they do not choose canonical perches, roll mood changes or originate bird-to-bird responses. Leaves, feathers and gentle decorative parallax are client-only ornaments with no behavioral consequences.

Persist `engine_version`, `grammar_version`, state revisions and random-stream counters. A release may change future behavior tuning but must not change a bird's identity or rebuild its vector from logs. Maintain backward-readable snapshot contracts during rolling deployments. Worker rollback is a code/config operation, never restoration of an older personality snapshot over current data.

## 4. Data model and retention

Use UTC instants for storage and ordering; local timezone is a presentation/simulation input. Use UUIDs for every identity and relationship, and integer revisions/sequences for ordering. Avoid email in URLs, keys, partitions, traces and job payloads.

| Record | Required fields and constraints |
| --- | --- |
| Account | `account_id`, encrypted verified email, keyed email lookup digest, lifecycle state, creation time, timezone, settings/version, deletion request and hard-delete deadline. Email and any pending replacement address live only in this protected identity record. A pending recipient/login identity can exist before an account is activated; it has no aviary. |
| Auth challenge | UUID, account UUID, purpose, token hash, created/expiry/consumed times, expected account/email-change version. Magic links expire at 15 minutes. Store no raw bearer token. |
| Owner device session | UUID, account UUID, hashed opaque token, issue/last-use/expiry/revocation times and a coarse user-visible device label. Initial policy: 30-day inactivity expiry, 90-day absolute expiry. Neither user-agent text nor device identity becomes an analytics dimension. |
| Aviary | UUID, unique owner account UUID, creation time, revision, last/next tick, processed event sequence, random counters, engine/config version, adoption ordinal, weather/daylight state and last observation metadata. Exactly one activated aviary per account. |
| Bird | Immutable UUID, aviary UUID, species/version, adoption date, user name/name revision, immutable signature seed, five persisted normalized traits, low-pass filter state, mood/dwell state, perch/action state, call counter and cooldown end. A rename updates only name fields. |
| Interaction event | UUID/idempotency key, aviary UUID, server-assigned sequence, owner session/window UUID, kind, optional bird UUID, server receipt time, validated bounded interval data, and minimal typed payload. Kinds include resume, presence interval, listen start/end, offer, settle, settle undo, owner re-engagement and window close. No pointer coordinates, key values, screenshots or free text. |
| Command result / cue | Event reference, stable cue ID, server times, receiving bird/outcome where applicable, presentation instruction and idempotent response. Acknowledged cues survive worker crashes and appear in snapshots on other devices. |
| Presence window | Opaque owner-window UUID, expected sequence, last acknowledged interval, validity/terminal state, and bounded recent interval segments. Account-level union accounting prevents duplicated device time. This is operational simulation input, not an engagement log. |
| Transient simulation inputs | Per-bird/per-aviary low-pass accumulators, current-day saturated signal counters, accepted-offer cooldown reservations and recently processed event cursor. Persist filter state with the vector; do not reconstruct it from old events at startup. |
| Scene timeline | Revisioned, short-lived action/call tracks, absolute start/end times, origin cue IDs, seeds, grammar versions and a coverage end time. Old tracks are pruned once no longer needed for rendering/tick processing. |
| Notebook entry | UUID, aviary UUID, creation time/local date, immutable naturalist prose, bird UUID references, names-at-observation, template/version and a deduplication fact key. Persist text so later template/name changes do not silently rewrite old observations. |
| Adoption opportunity | Aviary UUID, ordinal, eligibility time, system-selected species/seed and accepted bird UUID if any. Unique `(aviary, ordinal)` prevents duplicate adoption across devices. Dismissing the presentation neither expires the opportunity nor rerolls its species. |
| Invitation | UUID, host account UUID, recipient identity UUID, token hash, issue/unused-expiry/consumption/revocation times. No recipient email copy. Unused expiry is 30 days. |
| Visit session | UUID, invitation UUID, hashed capability token, start/last snapshot/expiry/end times. Initially allow up to 24 hours absolute and 30 minutes idle; renewal requires a new invitation. Multiple refreshes in this browser reuse this session, not the one-time email token. |
| Visit log | Host/recipient UUIDs, invitation/session UUIDs, start/end and approximate duration. It contains sharing metadata only. The account-settings endpoint resolves the visitor email from the identity record; it does not copy it into the log. |
| Async job / export | UUIDs, account references, purpose, status, retry metadata; export has snapshot revision, encrypted-object reference, download-token hash and expiry. Mail outbox payloads refer to identities and template IDs, not email addresses or interaction bodies. |

Account settings include audio preference, captions preference, narration controls, reduced-motion preference (`system` or `reduce`), timezone and optional visit-email preference. Per-browser permissions and AudioContext readiness are local capabilities, not synchronized promises that another browser can play sound.

Normalize email for lookup in the identity module without inventing provider-specific alias rules. A keyed digest supports exact lookup and per-email throttling; it remains confined to identity storage and is not an account identifier anywhere else. Pending invitation recipients receive synthetic identity UUIDs so an unregistered visitor never forces an email-based foreign key. The activated owner account retains that UUID if they later sign up. During email change, the old verified and new pending ciphertext necessarily coexist on the same protected account record; only verification atomically replaces the primary address.

Retention decisions:

- Keep vectors, filter state and identity continuously for the account's life. Keep all notebook entries browsable; do not archive or hide old entries.
- Keep consumed raw simulation events for seven days for private transactional recovery, then prune below the persisted cursor. Never prune unconsumed events on a wall-clock timer. Keep minimal idempotency records for 30 days; do not turn them into a second behavioral history.
- Bound presence interval working data to the processing/correction window, then retain only the accumulator/checkpoint needed by that account's simulation. Raw interaction data has no warehouse sink.
- Keep visit history for the account's life unless account deletion removes a referenced identity; expired unused invites can be removed after a short 30-day housekeeping window. Remove orphan pending identities when they have no remaining authorized purpose. Visit history is not the owner's forbidden attendance history.
- Export objects expire and are deleted after 24 hours. Auth tokens and closed capabilities are purged after their security/replay-retention window. Logs use short, documented retention, initially seven days for operational diagnostics.
- Soft deletion lasts 30 days; hard deletion removes every account-linked record, object, session, queued job and diagnostic record. Aggregate metrics are deliberately irreversible aggregates with no account dimension and therefore have no account-linked row to retain.

Enforce bounds and foreign keys in storage, not only UI. The account-to-aviary uniqueness constraint and adoption transaction enforce one aviary and at most seven birds. Give API and worker database roles separate write privileges: API commands can append events and edit permitted identity/name/settings fields; personality columns are writable only through the worker's transition transaction. Do not expose a generic bird PATCH endpoint.

## 5. API contracts

All endpoints are same-origin HTTPS. Owner sessions use Secure, HttpOnly, SameSite cookies with CSRF/origin protection for mutations. Visit sessions use a separate narrowly scoped cookie/capability and cannot be upgraded to owner authority by changing a body field. Strip tokens from the address bar after redemption and send no referrer from token pages. Explicitly redact request bodies, query secrets, names, email and snapshots from infrastructure logging.

| Endpoint | Contract |
| --- | --- |
| `POST /api/v1/auth/magic-links` | Accept email, return the same generic response for existing/new identities, and send a single-use 15-minute link. Start at five requests/email/hour plus a short burst limit and coarse source abuse protection; never lock out indefinitely. |
| `POST /api/v1/auth/consume` | Atomically consume an unexpired token and issue a per-device session. Link landing GET is non-consuming so email scanners do not sign in or exhaust tokens. The matter-of-fact landing form performs the consume POST. |
| `GET /api/v1/account` | Return editable settings, lifecycle state and account-management information, excluding personality. |
| `PATCH /api/v1/account/settings` | Version-checked timezone, accessibility/audio and visit-mail preference changes. A stale version returns 409 plus current permitted fields for reapplication. |
| `GET /api/v1/account/sessions`; `DELETE /api/v1/account/sessions/{id}` | List/revoke the owner's devices. Revoking the current device clears its state and returns to sign-in. |
| `POST /api/v1/account/email-change`; `POST /api/v1/account/email-change/verify` | Verify the new address before an atomic switch. Concurrent changes invalidate earlier pending verification; the old address remains valid until completion. |
| `GET /api/v1/aviary/snapshot` | Return one transactionally consistent presentation snapshot, ETag/revision, server clock and timeline coverage. Supports conditional pulls. The HTML route embeds the same projection. |
| `POST /api/v1/aviary/resume` | Owner-only, idempotent visibility/window epoch; returns fresh snapshot and a server-authored return-greeting cue. Does not count presence without its three conditions. |
| `POST /api/v1/aviary/events` | Accept a small typed batch of presence/listen/settle/undo/close/re-engage events, each with UUID and window sequence. Validate intervals and return per-event accepted/duplicate/rejected results and server cue references. |
| `POST /api/v1/aviary/offers` | Accept `seed`, `song_fragment` plus allowlisted motif ID, or `still_pool`. The server chooses eligible receivers and persists the reaction/cooldown reservation. User-supplied mood, traits, outcome and arbitrary target/position are rejected. |
| `GET /api/v1/aviary/birds`; `PATCH /api/v1/aviary/birds/{id}/name` | Settings-only identity list and version-checked rename. Names: 1–32 Unicode graphemes, trimmed, escaped as text; no uniqueness requirement that would force unnatural naming. No state-reset side effect. |
| `GET /api/v1/aviary/adoption`; `POST /api/v1/aviary/adoption/{ordinal}/accept` | Return/accept at most the next age-eligible, system-selected opportunity. Acceptance includes optional name and an idempotency key. Initial adoption assigns both starters transactionally. |
| `GET /api/v1/aviary/notebook?before=...&limit=30` | Owner-only keyset pagination by `(created_at, id)`, returning persisted prose. There is no edit/delete/comment endpoint and no upper historical cutoff. |
| `POST /api/v1/account/invitations` | Explicit host action accepting a recipient email; resolve/create the protected identity and send the one-time visit link. No bulk, discovery or automatic invitation API. |
| `GET /api/v1/account/visits`; `DELETE /api/v1/account/invitations/{id}` | Private outstanding/active/history list and idempotent revocation. Historical sharing and outstanding invitations are separate sections. |
| `POST /api/v1/visits/redeem` | Atomically exchange an unused, non-revoked, unexpired invite token for a render-only browser capability. Does not greet birds, record presence or create an owner aviary. |
| `GET /api/v1/visits/snapshot` | Check live invitation/session/account status on every pull, then return the same canonical scene projection with owner-only management fields removed. |
| `POST /api/v1/account/export` | Recently authenticated explicit request; enqueue a consistent snapshot export and email a short-lived download link to the verified owner address. |
| `GET /api/v1/account/exports/{id}/download` | Require the owning account session and emailed single-use capability, then stream the encrypted-at-rest export over HTTPS with no-store. |
| `POST /api/v1/account/deletion`; `POST /api/v1/account/recovery` | Mark deletion immediately, or restore before the exact 30-day deadline. Recovery is an explicit `I changed my mind` action after sign-in; signing in alone does not cancel deletion. |

Every simulation-writing request is owner-authorized against the target account, not just authenticated. Mutations use idempotency keys. Version matching protects names, settings and lifecycle changes; personality has no client version to merge because it is not writable. Payload validation uses a strict discriminated schema, with unknown state fields rejected. Cap a batch at 20 events and 16 KB, and use bounded per-session command admission. Presence heartbeats are not throttled behind discretionary offers.

Snapshot envelope: `schema_version`, `revision`, `server_now`, `as_of`, `timeline_through`, `engine/grammar_versions`, canonical local light/weather, active transient scene directives, and birds with UUID/name/species, current mood, perch/action anchors, derived appearance and call instructions. The owner snapshot may also include cue acknowledgements. Personality vectors, filter values, internal trait labels and drift deltas are not in the presentation DTO. A physical parameter such as call pitch or an SVG color is a rendering instruction, not a renamed raw trait field. Do not serialize the persistence object and then subtract a few fields; construct an allowlisted projection.

Use 401 for expired owner sessions, 403 for wrong capabilities, 409 for stale versions, 422 for invalid events and a generic 410 `visit no longer available` for consumed/expired/revoked links where appropriate. Functional offer cooldown can return `not_available` with no countdown or punitive language. Inline system errors explain the next action plainly. Do not add success toasts for routine interactions, renames or revocation.

## 6. Server simulation and exact update semantics

### 6.1 Scheduling, order and atomicity

Assign each aviary a stable phase within a 60-second interval. Workers query indexed `next_tick_at`, lease due rows with bounded batches and `SKIP LOCKED`, and process accounts regardless of connected-device count. Do not implement the scheduler as a scan of online sessions. Multiple worker processes share the due index; an aviary is serialized independently so one slow account cannot block unrelated accounts.

Command admission and tick processing both acquire the same aviary row lock. Event sequence is allocated under that lock in the transaction that appends the event. This avoids the subtle gap where a global sequence is allocated by one transaction, a later sequence commits first, and the tick permanently skips the earlier event. Acknowledgement happens only after commit.

For each due logical minute, in one transaction:

1. Read the current persisted state and event cursor; establish the ordered admitted event range and effective server-time cutoff.
2. Consume events in sequence. Reject impossible references, union eligible time intervals, close terminal windows, and use persisted offer/greeting outcomes rather than rerolling them.
3. Advance low-pass filters and apply nonnegative trait deltas to the stored vectors. Apply mood transitions, daily/timezone boundaries, weather and bird-to-bird effects.
4. Preserve unexpired action/call cues and extend the canonical timeline. Generate eligible noteworthy observation candidates, then apply notebook sparsity gates.
5. Commit vectors, filter state, mood, random counters, timeline, notebook insertion, cursor, revision and next due time together. A failed transaction leaves none of these partially advanced.

Tick retries see the already-advanced cursor/revision and do not apply events again. Unique event IDs, notebook fact keys and adoption ordinals reinforce transaction boundaries. Lock timeouts retry with jitter and an upper bound; repeated failures raise an operational error without reinitializing birds. The worker records timing after releasing the transaction and sends no row content to monitoring.

After an outage, catch up on the server in ordered logical minute steps from persisted state, in bounded transactions of at most 60 steps. Continue the scheduled worker even with no clients. Do not use return navigation as the trigger to restart simulation. Do not reconstruct a vector from history or replace it with an age-derived value. Future acceleration of quiet intervals must match a minute-by-minute reference for filter integration, mood boundaries, PRNG streams and pending events before being enabled.

Initial capacity sizing uses roughly `active_accounts / 60` ticks per second, including absent owners. For example, 100,000 accounts imply about 1,667 ticks/second; benchmark actual row/CPU cost before that capacity is offered. Limit launch account count to measured headroom, provision workers against scheduled load, and measure queue age as well as execution time. Scaling is an engineering capacity problem, not a reason to stop absent aviaries.

### 6.2 Drift function and calibration

Store traits in `[0, 1]`. Seed individual birds in a moderate, varied range, initially approximately `0.18–0.42`, with species tendencies and individual variation. Avoid identical starters and avoid seed values already near saturation. Use floating-point double precision or an equivalently precise bounded representation, and preserve original persisted values during migrations.

Define daily-normalized positive signal increments, retaining only the accumulator state needed for this account:

- Presence `dP`: newly credited union seconds divided by 600. Start with a 10-minute regular daily visit as the calibration reference. Saturate the daily useful presence signal smoothly by approximately 30 minutes to prevent marathon sessions from overwhelming weeks-scale tuning; show no quota or meter.
- Listen `dL_b`: a bird's eligible listen seconds divided by 600, bounded by owner presence and split across simultaneously listened-to birds on different devices. Its weight affects social warmth and vocal frequency.
- Offer `dO_b`: small, capped increments for server-admitted offers near that bird and separately for acceptance. Begin with at most three effective offer contributions per bird/day, in addition to the per-bird cooldown. Nearness affects boldness; acceptance affects curiosity. Cooldown failures contribute zero.
- Settle has no personality signal. Audio permission, mute, absence, deletion recovery and tab-close also produce no negative personality signal.

For each trait, construct `dS = 0.85*dP + 0.10*a_trait*dL_b + 0.05*b_trait*dO_b`, where the mapping coefficients are zero for unrelated traits and at most one otherwise. The weights make presence dominant under the reference pattern; even a user making no offers and never using audio develops expressive birds. Offer attempt/acceptance share their small budget rather than each obtaining the full budget.

Implement a causal low-pass filter with initial time constant `tau = 3 days`. For a tick of length `dt` days, set `u = dS/dt`, then use the exact constant-input update:

`h_next = u + (h_previous - u) * exp(-dt/tau)`

The integrated filtered input over that tick is:

`I = u*dt + (h_previous - u)*tau*(1 - exp(-dt/tau))`

Apply the additive server-authored increment:

`delta_trait = max(0, (1 - current_trait) * rate_trait * max(0, I))`

`next_trait = min(1, current_trait + delta_trait)`

Begin `rate_trait` near `0.008–0.012` per normalized day, then calibrate the appearance/behavior mapping as well as the rate. Persist `h_next` and `next_trait` atomically. With no new input, the filter decays toward zero while its residual positive contribution can continue to raise a trait briefly. The trait itself never decays. Changing a target, weather or a filter coefficient must not create a negative delta. The clamp is a final invariant guard, not a substitute for correctly signed inputs.

Synthetic reference checks: one ordinary session produces less than `0.005` absolute change in every trait and no perceptibly abrupt change; approximately seven days of regular visits produce readily instrumentable movement, initially target about `0.015–0.05`; approximately 21 days produce enough accumulated behavioral/visual change for blinded observers to notice, initially about `0.08–0.16` where mappings make that useful. These are calibration bands, not product statistics. Test the formula under the actual daily event pattern before accepting those bands; do not claim the initial constants guarantee human perception.

Calibrate with scripted synthetic aviaries, deterministic time acceleration, local rendering/audio comparisons and human perception feedback. Do not mine real accounts' event histories, compute an average production drift dashboard, or send per-bird data to analytics. Repeated click bursts, multiple devices, no-audio use, long absence and very long sessions are mandatory trajectories. Human testing must judge continuity and recognizable individual birds, not just detect color changes.

### 6.3 Mood, ambient life and absence

Use five initial moods: wary, content, curious, drowsy and alert. Keep mood distinct from the trait vector. Persist mood and a dwell timer, with typical dwell of 5–30 minutes and short reaction cues allowed within it. Sample transitions from bounded weights over current mood, personality, recent accepted reactions, local day phase and weather. Apply hysteresis so a bird does not alternate abruptly between curious and wary every tick.

A daily-ish boundary refreshes mood tendencies around the account's local dawn; it does not reset traits or snap every bird to content. Stagger transition opportunities across birds. Dusk favors drowsy; dawn favors alert; night leaves most birds settled with the nightjar-like species able to call. Use IANA timezone rules with UTC tick order so repeated or missing DST clock hours do not double-consume events or reset mood twice.

Rain appears initially about two or three times/week and lasts roughly 4–10 minutes; gentle wind is similarly occasional. Weather is generated from the account's seeded server schedule, not real-world geolocation or a weather service. It influences short-lived mood/expression and call density, never the persisted vocal-frequency trait. Wary behavior is brief natural alertness, not suffering or a penalty for absence.

Use recent-presence context only for temporary orientation/greeting intensity. After two weeks away, birds retain all their traits and recognizable signatures, continue ordinary ambient activity and can respond more quietly without becoming distrustful or losing plumage. A return does not reset mood. The server's elapsed-day evolution explains what changed in the interim.

Server action selection maps mood and personality to bounded perch preferences and idle behaviors: bold birds tend forward; wary birds scan from farther back; content birds preen; curious birds inspect sounds/leaves; drowsy birds fluff and lower their bodies. Reserve perch occupancy before scheduling movement. No owner placement command is available.

### 6.4 Calls, social behavior and greetings

Give each bird an immutable individual signature consisting of a species grammar, seed, pitch neighborhood, timbral envelope family and characteristic motif relations. Signature generation never uses the mutable name. Mood changes articulation, pauses and density within an identity-preserving range; drift changes calling frequency and willingness to answer more than it changes the recognizable sound. Start pitch variation within a small band, about ±4%, and verify identity by listening.

The server schedules call intents and response relationships; the client expands each intent's versioned grammar and variation seed into a concrete note/envelope description, then synthesizes it. The same expansion supplies captions. A monotonically increasing bird call counter contributes to seeds, and immediate duplicate expanded parameter sequences are rejected. Real variation in timing, pitch contour, breath and pause placement replaces a short rotation of canned variants.

Other birds can answer a call with a delayed probability shaped by warmth and vocal frequency. Schedule those answers on the server, use refractory windows, and cap chain depth to prevent one call from recursively exciting the whole aviary forever. Nearby alert/wary calls can nudge mood at the next tick. Chorus intervals arise from overlapping compatible calls and responses; do not play a premixed chorus or trigger all birds as an arrival announcement.

On owner resume, select one primary greeter using weighted boldness, warmth, current mood, last greeting history and absence length. Use a fresh server seed to vary head angle, partial-preen interruption, glance duration, approach and call contour. Short absences usually produce a glance; longer ones allow reorientation or a longer call. Aim for the primary notice within 1–2 seconds of normal navigation/return. Secondary noticing uses independent small offsets, initially 0.4–1.8 seconds, and is optional. Persist the primary bird and cue to support observations such as a change in who greets first.

Resume requests are idempotent per visibility epoch. Brief focus/visibility flapping is coalesced; one arrival cannot become a burst of five greetings. A read-only snapshot pull, prefetch, notebook opening or visitor arrival never creates a greeting. When audio is blocked, the same greeting remains legible as motion, narration and a call caption.

## 7. Presence, interactions and multi-device sync

### 7.1 Honest presence intervals

Start with a five-minute recent-activity window, deliberately toward the long end so still watching counts. Track `visibilityState`, `document.hasFocus()` and a monotonic timestamp of the latest trusted pointer movement or keypress-equivalent event. Use modern keyboard events to observe actual key activity without recording keys; do not count script dispatch, window focus, a network response or an open tab as activity. Touch pointer movement and accessible keyboard operation use the same rule. No periodic synthetic pointer event or audio playback keeps a window present.

The client emits a heartbeat every 30 seconds only for intervals during which all three conditions held. Segment intervals on blur, hiding, activity expiry and settlement so a partly invalid interval is not counted as fully valid. Do not initially credit the preceding five minutes when an activity event first occurs. On blur/hide/close, attempt a final bounded beacon. A missed beacon loses at most a small unacknowledged interval; the server never extrapolates indefinitely from the last ping.

A presence payload contains the opaque window ID, sequence, elapsed qualifying milliseconds since acknowledgement, bounded timing/condition assertions and the latest activity age. It contains no raw input data. The server bounds credit by receipt spacing and the 30-second reporting window, rejects negative/overlapping/future segments, and closes invalid or stale windows. Client clocks cannot advance the aviary or manufacture hours of reported credit. The server cannot prove a human's gaze or that a malicious browser told the truth; the goal is correct ordinary browser behavior, not surveillance or anti-cheat machinery.

At tick time union valid owner intervals across all windows/devices before adding presence. One minute of overlap is one minute. For bird-specific listen time, union duplicates for the same bird; if multiple birds are listened to on different owner devices in the same time slice, divide that attention budget among those birds. Never exceed the global qualified presence duration with the sum of listen allocations. Presence remains private simulation input and is never displayed as a duration, streak or visits calendar.

Settle and close terminate the invoking presence window. A settled tab can continue fetching state for display without earning presence. Undo or a new genuine interaction creates a new window; it cannot reopen and backfill the terminal interval. Other genuinely active owner devices may continue contributing their own union time. Visitor code does not load or instantiate the owner presence reporter, and server capability checks independently enforce the same exclusion.

### 7.2 Interaction outcomes and cooldowns

Listen-in is a local auditory focus with owner attention events, not an instruction to change every device's mix. Clicking/tapping another bird, roving keyboard focus, empty-space activation and focus leaving the scene end the old listen interval exactly once. On suspension or error, close it or let its short lease expire. Do not count an abandoned listen-start indefinitely.

Offers originate from the top-bar panel. Place the seed/pool in a fixed safe front area, or play one of a small library of procedural melodic fragments. The server chooses an eligible lead receiver using current proximity, curiosity and mood; other birds may visibly notice without each collecting a reward-like input. Seed responses include approach, delayed approach and indifference; pool responses include drinking, bathing and watching; melodic responses include joining, quieting and a call against the motif. Drowsy non-response is a valid outcome.

Reserve a three-minute cooldown per receiving bird atomically at command admission, including ignored offers, so parallel devices cannot bypass it while awaiting the next tick. If all potential receivers are cooling down, the offer is unavailable; show a brief naturalist explanation inside the opened panel, no timer or punitive state. A rejected/retried offer does not create an extra cue or drift input. The admitted outcome remains fixed through retries and later mood changes.

Settle sends a durable scene directive and a small mood-quieting event. Fade the canonical lighting toward evening and gently reduce call activity; a session-end gesture is not full silence. The directive is associated with its initiating owner window and visible in canonical snapshots, including visits. It lasts while that settled window remains open, until an owner actively re-engages, or until the initiating window closes/expires. Owner display keepalives may keep the directive visible without being presence pings. A 45-second missed-display lease bounds a lost close beacon. Another device's passive snapshot pull does not undo it; genuine owner interaction does.

Any click in the aviary within five seconds of settle sends an undo referencing that directive ID and immediately reverses the locally acknowledged transition from its current interpolation value. Keyboard Enter/Space in the scene provides an equivalent undo. Idempotent server undo clears only that matching directive, never a newer settle from another device. If settle and undo precede a tick, their transient mood contribution cancels; if the earlier quieting already happened, the next tick clears its temporary input without reversing personality. After five seconds, deliberate re-engagement still resumes the aviary; it is simply no longer the accidental-click undo path. Closing without settle follows the same presence termination and no penalty.

### 7.3 Snapshot propagation and recovery

Pull on initial navigation, visibility becoming visible, window re-focus after a meaningful gap, a render-frame gap over two seconds, and every 15 seconds while visible. Pull immediately after an acknowledged mutation if its response did not already include the newer projection. Stop normal rendering and unnecessary polling when hidden; on return fetch before resuming expired schedules. The server continues ticking throughout.

Snapshots carry monotonically increasing revisions. A client discards a late response older than the displayed revision, reconciles cues by stable ID, and never replays the same offer or call after a refresh. Estimate server-to-monotonic-client clock offset using request midpoint timing; adjust gently so small clock corrections do not jerk birds or detune calls. Timezone changes are a server setting update, not a device clock correction.

Merge only presentation continuity: interpolate from the current drawn pose toward newly authored anchors, keeping semantic state equal to the latest snapshot. Do not merge mood or trait copies. A scheduled flight already in progress remains in progress; a long suspension skips expired motion/audio instead of playing a backlog. Under normal service, the 120-second timeline covers 15-second polling jitter and brief packet loss.

If connectivity fails, retain the last bounded presentation while its timeline is valid, disable server-dependent offers/adoption, and use an inline matter-of-fact connection message in the available system surface when an action needs it. Do not pretend an unacknowledged offer succeeded. Once the semantic timeline expires, avoid inventing mood/calls; show a quiet field/explicit loading error as appropriate and resume from an authoritative snapshot after recovery. This is a degraded service condition, not an alternate offline simulation. Presence may retain only a tiny in-memory retry window, at most two minutes with server-bounded segments, and never accumulate offline hours or a durable backlog across reloads.

Name/settings conflicts use 409 with the latest values and a clear retry path; they cannot overwrite a vector. Session expiry, replay and outages use ordinary system prose, with no naturalist euphemism. No client uploads its stored snapshot on sign-in, so an older phone session cannot erase the laptop's drift.

## 8. Frontend scene and interaction surfaces

### 8.1 First paint and render pipeline

Use compact SVG species artwork and articulated pose components, with procedurally varied transforms, feather details and palette application. Avoid a general-purpose 3D/game engine, large animation runtime or video background. Keep the render boundary independent of the account/UI framework so first-bird rendering does not wait for hydration, an AudioContext, notebook data or all six species' assets.

The server renders the first SVG using the snapshot's absolute-time action anchors evaluated at response time. The critical controller evaluates the same tracks at estimated current server time and takes ownership without replacing or fading the scene node. Every repeating visual micro-motion is phase-offset from its bird/action seed and absolute time, never initialized at phase zero on mounting. A newly seen bird can therefore be mid-preen, mid-weight-shift or already partway through a call. The scene's action is continuous before the return-greeting is layered onto it.

Render layers, from back to front: sky/daylight palette; quiet distant foliage; back, middle and front perch anchors; depth-sorted birds; the brief offer surface if any; restrained foreground foliage/ornaments; opt-in captions and focus treatment. Top-bar DOM sits outside the scene's clipping rectangle. Scene SVG is hidden from the accessibility tree to avoid exposing every decorative feather; a synchronized semantic surface supplies meaningful access.

Drive one `requestAnimationFrame` loop for visual evaluation. Update transforms/opacity only where possible, cache geometry, and avoid DOM measurement in the frame loop. Position captions in a separate lower-frequency layout pass. Server action tracks determine a bird's behavior and destination; small breathing, feather movement and transition easing are deterministic rendering elaborations within that action. Decorative leaves/feathers can use a small local seeded pool, at slow intervals, without writing events or influencing server mood.

For a delayed snapshot, render a quiet sky/field with restrained ambient color, never a spinner, progress badge or skeleton flock. On successful first load, draw the continuing authoritative pose directly, with no generic entrance sequence. If the service remains unavailable, provide the specified matter-of-fact loading-error surface and retry. The <500 ms budget is a release constraint under the defined healthy network/device profile, not a promise during an outage.

The one-time adoption state is an explicit exception: after the naming flow, the empty quiet field receives two softly entering starter birds. Store a consumed introduction cue so reload cannot replay their arrival. Later voluntary adoptions can have a similarly specific new-bird arrival, but returning existing birds never re-enter as though created anew. In reduced motion, these intentional arrivals are slow pose cross-fades.

### 8.2 Layout and visual language

Start with a logical scene approximately 1000 by 480 units and normalized front/middle/back safe anchors. Responsive layout can redistribute available slots within each depth zone while preserving the server-selected zone and bird identity. Use letterboxing or surrounding quiet field as needed to preserve bird proportions; do not stretch silhouettes, crop a bird, scroll a camera, or let a flight path leave the safe rectangle. All path control points and body extents are validated against that rectangle.

At narrow widths use staggered occupancy across the three zones, with collision-aware spacing and larger invisible hit targets. Test two through seven birds at 320 CSS pixels wide, common phone portrait/landscape sizes, short landscape windows, tablets and wide desktops. The scene remains horizontal on a portrait phone; unused vertical space may remain quiet rather than becoming a second screen. Settings and notebook panels may scroll normally. Browser text enlargement, pinch zoom and 200%/400% accessibility zoom are not disabled to enforce the no-camera rule.

Use soft blue, green, brown and ochre tokens. Treat plumage saturation as a subtle server-derived appearance change and cap visual mapping before it becomes neon. Daylight transitions continuously through warm morning, clearer midday, warm evening and dim night. No thunderstorm, alarm flash, assertive weather or UI accent competes with a bird. Parallax is a tiny bounded ornamental offset; it is not driven by large cursor excursions or a camera gesture.

Use four top-bar buttons with accessible names and large hit areas, preferably at least 44 by 44 CSS pixels. Their panels hold all management text and actions. The bar begins visible, quiets after about four seconds without pointer/key activity, and returns on movement, keyboard activity or touch interaction. Never fade a focused control, a panel's owning button, or a pending error's text. For touch devices keep the controls discoverable without requiring hover.

No inline bird-name labels, mood badges, tooltips, offer buttons, status meters or ownership counters appear in the scene. Bird-specific naming/settings are reached from the account panel. Clicking a bird means listen-in; it does not open a configuration menu.

### 8.3 Reduced motion

Read `prefers-reduced-motion` before starting motion and combine it with the saved `system/reduce` preference. Do not show a full-motion frame while waiting for the settings panel bundle. System media-query changes update the presentation in place and dispose of now-unused motion tracks.

Use designed still-pose sequences for each mood/action, with slow cross-fades instead of articulated micro-motion. Starting targets: a held pose lasting 8–16 seconds and a 2–3 second cross-fade where the bird's action allows it. Flight becomes a fade between source and destination perches with no translating body. Remove moving leaves, falling feathers and parallax. Retain slower ambient color transitions, initially about twice the standard duration. A settle still changes the light; a curious bird still looks inquisitive; a greeting still notices through a pose change.

Keep procedural calls, captions, mood evolution, drift, notebook and every interaction intact. Caption transitions are gentle opacity changes, with no bounce or travelling text. Reduced motion has its own visual review against the same aliveness standard, not an exemption from it.

## 9. Procedural audio and captions

### 9.1 Grammar and synthesis

Represent a call as a versioned grammar with a few motif relations, syllable types, pitch contours, durations, rests, amplitude/filter envelopes and optional breath/noise components. The bird signature constrains that grammar's recognizable family. A call intent contains its bird UUID, absolute time, grammar version, seed and bounded expressive modifiers; it never points to a recorded asset.

Expand the intent into a concrete call description once. The audio backend and caption formatter both read that exact description. Do not separately randomize caption text or describe a fixed stock call while the audio plays a variation. The small song-fragment offer library is also note/motif data synthesized through this pipeline, with soft timbre and no uploaded user recordings.

Use an AudioWorklet-backed reusable voice pool when available; keep a small native WebAudio oscillator/filter/envelope implementation for browsers that expose WebAudio but not the preferred worklet capability. This remains procedural synthesis. Start with a maximum of 16 synthesis voices and a bounded note queue. Preallocate worklet buffers, keep processing free of unbounded allocations, and reclaim/disconnect transient native nodes on completion. Do not create a new AudioContext per call or per bird.

The scheduler wakes approximately every 50 ms and fills a roughly 150 ms lookahead against `AudioContext.currentTime`, mapped from server time. Use audio-clock scheduling, not animation frames or `setTimeout` note-on accuracy. On a snapshot update, reconcile future notes by call ID; retain already started compatible calls, remove invalid future ones, and never replay completed calls. If the first visible snapshot falls mid-call, synthesize the remaining contour/envelope at its actual offset or wait for the next call when a partial onset would click. Captions reflect the same remaining event.

When hidden, end owner listening/presence as appropriate, ramp audio down briefly and suspend audio scheduling/context to protect battery. On return, reauthorize visits if relevant, fetch current state and resynchronize to future calls. A suspended laptop does not resume yesterday's half-filled audio queue. While the document is visible but loses window focus, rendering can continue; presence still stops. Audio behavior follows the user's sound choice and browser capability, independently of whether attention is being credited.

### 9.2 Chorus and listen-in mixing

Each bird feeds a dedicated gain bus and shallow positional pan corresponding to its visible position. Buses feed a common ambient/master stage with conservative headroom and a limiter for accidental peaks. Use modest distance attenuation so back-perch birds remain identifiable. Start around 6 dB of mix headroom and normalize cautiously as voices overlap, then tune by ear; normalization must not pump audibly or make all birds sound equally loud.

The server schedules restrained overlaps, usually two or three foreground calls, leaving real rests. Bird signatures, independent phases and varied phrasing make a chorus; repeating synchronized recordings do not. Test all seven birds even when the initial customer experience contains two. Avoid phase relationships that cancel important motifs or build a harsh high-frequency wash.

On listen-in, begin with approximately +3 dB for the focused bird and -8 dB for others relative to the current ambient mix, bounded by master headroom. Ramp both directions over about two seconds, including switching to a different bird. The un-focused buses remain nonzero: mix attenuation never silently becomes soloing. An unfocused bird can naturally be silent between calls, but the mixer must not mute it. Cancelling/replacing ramps starts at the current envelope value, preventing steps and clicks.

Settle gently quiets calls and mixing over approximately four seconds, following its canonical directive. Undo/re-engagement reverses from current values; it does not jump or restart a track. User master mute intentionally silences all sound; its existence is distinct from listen-in's nonzero ambient rule. Keep sound preference synchronized, but do not force another browser to play without its own permission.

### 9.3 Fallback and caption layout

If WebAudio is unavailable, blocked, cannot resume, or fails because of the device, the aviary continues in graceful silence with captions on by default. A previously explicit caption preference can remain stored, but an audio-failure session enables the fallback caption surface and lets the user change it knowingly. Show a plain sound-status control inside accessibility settings; do not interrupt the scene with an error toast or an autoplay modal. On an eligible user gesture, attempt resume and crossfade into currently valid calls. Never download a recording as fallback.

Caption generation describes the expanded contour and rhythm in naturalist language: for example, `a soft three-note rise` or `a low trill, paused, low trill again`. Incorporate actual note count, rise/fall, articulation, rest and perch context where useful. If audio is muted/unavailable, the caption describes the procedural call the scene is making; do not suppress the aviary's vocal life merely because the output device is silent.

Position captions near the calling bird with a small high-contrast backing treatment and a collision solver that accounts for bird bounds, focus rings, screen edges and other current captions. Use sensible minimum text size, wrap to a short two-line form, and stagger according to actual call scheduling. Do not place all captions in an unrelated fixed log. When density would make captions illegible, constrain future server call density for the supported maximum and choose concise truthful descriptions; do not independently drop selected birds' captions while their calls play. Keep a bounded current-caption set and dispose of it after calls end.

## 10. Accessible experience and naturalist writing

### 10.1 Semantic interaction model

Provide landmarks for the top bar, aviary, notebook and opened system panels. Use a roving-tabindex bird group: Tab visits the top-bar items then the first/current bird; arrows move among birds in visible spatial order; Enter engages listen-in; Escape disengages. A subsequent Tab leaves the scene without requiring seven stops. A bird's accessible name identifies it naturally, such as `pip, a warbler on the front perch`; it does not expose traits, a numbered perch, a mood meter or an unexplained numeric value.

Offer and settle remain fully keyboard reachable through the top bar and its panel. Offer options use ordinary buttons with focus order, dismiss behavior and focus return. Provide an optional documented modifier shortcut such as Alt+Shift+O for the offer panel, remappable/disableable from accessibility settings, with the normal focused-button Enter path always available. Never intercept typing in names/email or rely on a shortcut that conflicts with assistive technology as the only route.

Panels have an appropriate labelled dialog/popover pattern; modal panels contain focus only while open, Escape closes them, and focus returns to their opener. Do not trap focus inside the aviary. Keyboard focus uses a soft two-tone outline that remains visible on bright and night palettes. If a focused bird changes perch, its stable DOM/semantic identity retains focus. Pagination cannot recycle the currently focused notebook content out from under a reader.

Audio availability is never a prerequisite for using listen-in, offering a motif, settling, naming or navigating. Visitors get local audio/caption/reduced-motion controls and meaningful scene descriptions, but no owner interaction buttons or keyboard listen-in behavior. Reading a visiting scene must not accidentally dispatch a host event.

### 10.2 Running narration

Build a deterministic observation-to-prose formatter shared in vocabulary with the notebook. It consumes the same current scene/call description as visual rendering. It writes a short connected observation, not a bullet list of state mutations: `pip preens on the front rail. farther back, wren rests with feathers fluffed; the evening light is warm.`

Use a polite, atomic live region for a single current narration paragraph. At idle, consider an update every 45 seconds, within the specified 30–60 second cadence, only when there is something meaningfully new. Coalesce ordinary changes and retain at most the current paragraph plus one pending replacement. Do not let call captions and narration each announce the same event automatically to the screen reader.

Promptly narrate the primary return-greeting, an acknowledged offer reaction and settle as observations, while preserving the polite queue and avoiding overlapping speech. When the user moves focus to system forms, pause ordinary ambient announcements until they return; meaningful user-action results remain available. Supply settings to pause narration, change its slow cadence and read the current scene on demand. A displayed transcript uses the same prose and contrast as the live region. No server speech generation or third-party language model is needed.

The narration should convey particular birds, posture, calling, relative depth and time/weather without announcing `mood: wary` or revealing any numeric substrate. Favor lowercase, present-tense naturalist prose; honor names safely as text. Account, error, sync and accessibility-settings instructions use normal capitalized, matter-of-fact language. Audit that voice boundary explicitly, including failures and expired links.

### 10.3 Contrast and access acceptance

Specify and measure text tokens for at least 4.5:1 contrast for ordinary text and 3:1 where the large-text criterion applies. Target at least 3:1 for actionable non-text controls and focus indicators against adjacent colors. Validate captions on every day/night/weather background, not only a design-system swatch. Use a stable backing surface where dynamic scene color would otherwise break contrast. Color alone never identifies an action, error or focused bird.

Test keyboard-only operation, high zoom, browser text enlargement, reduced motion, muted audio, failed WebAudio, high-contrast/forced-color settings, and simultaneous combinations. Screen-reader review includes VoiceOver with Safari, NVDA with Firefox/Chrome, and a mobile reader/browser combination within the support matrix. Automated accessibility checks catch missing semantics/contrast problems; human sessions decide whether the narration and alternate pose surface retain the product's charm and whether queues are tiring.

## 11. Notebook, adoption and account experience

### 11.1 Sparse observations

Generate notebook candidates on the server from specific facts already needed by that aviary: a different first greeter, an unusual but sustained perch choice, a notable bird-to-bird exchange, a quiet weather/posture combination or a recognizable change in behavior across weeks. Maintain a small private fact summary where comparison requires it; never derive production population statistics. Do not write a row for each presence heartbeat, session start, offer or tick.

Start with a 48-hour soft spacing target and roughly one normal entry every two or three days for regularly visited aviaries. Allow genuinely notable events past the soft gate, with a hard burst guard of at most one entry in 24 hours and at most three in a rolling week during initial tuning. Those guards can be loosened only if sparse long-run fixture reviews support it. Deduplicate both exact facts and repetitive template shapes, so active users cannot create a busy feed by repeating offers.

Entries are specific observations of the birds, never observations of user compliance. `pip greets before wren today` is permitted; `you visited every day this week`, `session started at 7:43`, and `boldness increased by 0.03` are forbidden. Historical wording such as `first time this week` must be supported by that aviary's retained bird fact summary, not invented for variety. The default prose is present tense, including dated notes.

Persist entries once and keep them read-only. Preserve names-at-observation when a bird is renamed, retaining its immutable UUID relation. Use keyset pagination with a bounded three-to-five-page client cache and cursor-based refetching in either direction; old text stays available indefinitely without keeping its entire DOM/query history in memory. Do not place unread badges on the notebook icon, archive older entries or add a calendar view.

### 11.2 Adoption and names

On account activation, select two different species with distinguishable silhouettes/calls and create their stable identities in one transaction. Present default names and optional edits as meeting these particular birds, not configuring avatars. The user cannot browse a species catalog or reroll for rarity. Persist adoption completion so a interrupted naming session resumes with the same birds.

For later birds, reveal only the next eligible opportunity when the user opens the relevant account/birds surface. No onboarding share prompt, announcement, counter, progress bar, email or automatic arrival accompanies an age threshold. The user may defer indefinitely; elapsed age still determines eligibility, with one opportunity presented at a time. Choosing not to adopt does not impair the existing birds.

Select later species from the coherent pool, preferring variety until all six are represented, then use a distinct individual signature for a seventh bird from that same pool. Selection is not ranked or rare. Adoption acceptance reserves the next perch capacity, checks the seven-bird limit under the aviary lock and returns the existing result on retry. Two simultaneous devices cannot adopt an eighth bird or different versions of the same opportunity.

### 11.3 Account lifecycle

Account pages use the quiet system voice. Session management lists current/recent devices without a tracking dashboard. Email verification, session revocation, replay and error paths have explicit states and clear recovery actions. No system error is translated into a bird metaphor.

An export worker uses a repeatable-read snapshot or equivalent version-pinned read to gather birds, names, current vectors under the documented portability exception, moods, all notebook entries and account settings. Include schema version and capture time. Exclude authentication secrets, bearer tokens and private operational event logs. Encrypt the temporary file, email only its protected download link, require owner authentication at download, and delete it after 24 hours. Recheck account lifecycle and verified recipient immediately before delivery; an email change invalidates/reissues delivery authorization rather than sending to an obsolete address.

Deletion is an explicit account-settings operation with a clear confirmation of the 30-day recovery period. Immediately mark the account pending deletion, terminate active visits, invalidate invites/exports and revoke ordinary device access. A subsequent magic-link login can access only the recovery/deletion surface. Suspend ordinary simulation jobs for the pending-deletion account while preserving its entire state. Recovery before the deadline restores access to those same bird UUIDs/vectors and advances any missed time on the server without manufacturing presence or penalizing the deletion interval.

At the deadline, an idempotent erasure worker removes live records, replicas, objects, jobs, capabilities, account-linked diagnostics and any cross-account invitation/visit references to the deleted identity. Account deletion and recovery lock the lifecycle row so a recovery cannot race a completed purge. Retry partial external cleanup and alert until every target confirms completion; do not report hard deletion complete while an export object or mail job remains.

Backups need a designed erasure path, not an exception in small print: encrypt account-owned payloads with per-account data keys, keep those keys outside restorable application backups, and irreversibly destroy them at hard deletion along with live records and lookup mappings. Ensure old backups cannot restore destroyed keys or re-enable deleted accounts. Physical encrypted remnants age out on a bounded backup retention schedule. Conduct restore/erasure drills demonstrating unreadability and absence of account linkage; ordinary backup restoration must preserve, not regenerate, surviving bird vectors. Operationally document the distinction between immediate logical/cryptographic erasure and physical backup expiry.

## 12. Visits, revocation and privacy boundaries

Each invitation is a deliberate email address entry by the host. There is no global discoverability flag, default visitor list, automatic sharing prompt, reciprocal invite, permanent friend connection or public URL directory. The emailed bearer link proves access through possession of that inbox/link; it is not published or treated as an owner session. Use cryptographically random tokens with only their hashes stored.

An unused invitation expires after 30 days. Redemption transactionally consumes it once and creates a scoped visit session. Multiple tabs redeeming the same token cannot produce multiple sessions; a reload with the already-issued visit cookie is allowed within its lifetime. A fresh visit after session expiry requires a new deliberate invitation. Expired/revoked/otherwise unavailable links show the same matter-of-fact surface and cannot be revived.

The visitor uses the common renderer/audio grammar and the host's canonical timezone, birds, palette, weather, active cues and moods. Local sensory preferences may change captions, motion or sound output, as they do for owner devices; there is no special beautified state. Owner presence, host-online status, device identities, settings, notebook, vectors and interaction logs are not in the visit response. The visitor cannot cause a greeting, listen-in, offer, settle, notebook mutation, adoption, drift or owner notification merely by manipulating hidden controls. Authorization rejects direct HTTP attempts too.

A visitor snapshot pull validates current invitation status against authoritative storage every time, including conditional/304 responses. Never return a cached successful authorization after revocation. Use a 15-second visible polling/authorization lease; on the next pull after revocation, clear the scene and stop scheduled audio before displaying `This visit is no longer available.` If validation fails because the network is down, stop exposing the scene when the short lease expires instead of using the owner's 120-second offline presentation grace. Hide/suspension requires revalidation before redisplay. Previously seen content cannot be made unseen, but no further snapshots or calls continue after access loss is recognized.

Record start and approximate duration only for the host's private visit history. Snapshot pulls can update a coarse visit-last-seen field; they are never presence events or drift input. Round the displayed duration to minutes and label it approximate; do not infer that a visitor paid continuous attention. Revocation closes an active visit, removes it from active/outstanding lists and leaves the authorized historical transparency record until retention/deletion applies. There is no notification or badge confirming that revocation worked.

If the host explicitly enables visit emails, enqueue one message on visit-session creation, deduplicated by session UUID. Check the setting, host lifecycle and revocation status again before sending. Send a restrained system email linking to account settings, with no bird state, tracking pixel, reminder or recommendation. Default-off accounts produce no host mail, push, toast or in-product announcement. The invite email itself and this optional notice are delivery transactions, not a social engagement campaign.

Privacy enforcement is structural:

- Simulation tables and blobs are private to simulation, authorized projections and account export/deletion jobs. Analytics/monitoring roles have no read permission on them.
- All internal identity references are UUIDs; decrypt email only inside identity/delivery or an authorized settings response. Mail providers receive the address necessary for the requested delivery, never interaction payloads or exported bird data in message bodies.
- Logs accept an allowlist of operational fields. No body logging, session replay, input capture, raw URLs containing tokens, arbitrary JSON serialization or user-content error attachments.
- Aggregate metric emitters use a separate typed contract that cannot accept a bird, event or snapshot object. No training, recommendations, cohorting by behavior, average drift, most-visited rankings or cross-account interaction analysis pipeline exists.
- Limited account-specific operational errors may use the synthetic UUID to diagnose authentication/sync failure, with short retention and erasure support. They contain error code/timing only, not what the account's birds did.
- Put a plain privacy-policy link in account settings, explicitly naming allowed timing/error/count categories and excluding per-bird state and interaction history. Never add an exportable owner-attendance log as a privacy feature; the product explicitly prohibits that surface.

## 13. Performance budgets and observability

### 13.1 Budgets and measurement contracts

| Budget | Implementation target and verification |
| --- | --- |
| Initial JavaScript | Hard gate: initial JS at first paint below 2 MB gzipped. Working target: no more than 400 KB total initial route JS, with a critical scene controller about 30 KB or less. Report the actual eager dependency graph, including transitive imports and work started by preload tags. |
| Critical first-bird transfer | Initial working cap of about 70 KB compressed for the essential HTML/styles/snapshot/starter artwork/controller needed for the first bird. Secondary species, panels and audio preparation do not block that bird. |
| First bird | Below 500 ms from navigation to a visibly rendered actual bird under the fixed mid-tier-mobile 4G test profile. Begin with 4 Mbps downlink, 80 ms RTT and a documented physical mid-tier Android reference device; also run colder/slower networks as diagnostic cases. Healthy-path component budget: TTFB 200 ms, essential transfer 140 ms, parse/evaluate 50 ms, pose/paint 35 ms, leaving margin. Confirm with visual frame evidence, not merely an API-response mark. |
| Snapshot | Target under 12 KB compressed at seven birds with roughly 120 seconds of schedule. No notebook history, raw logs, full traits or unused species assets in the snapshot. Measure typical and worst-case response sizes. |
| Return notice | One primary bird notices within 1–2 seconds on normal navigation/return, including audio-blocked and reduced-motion paths. Observe behavior, not an invisible cue-enqueued timestamp. |
| Idle frame budget | Sustained 60 fps on the fixed five-year-old mid-range laptop fixture at its normal 60 Hz display, including seven birds, weather and captions. Aim for render JS under 3 ms/frame and total work below 16.7 ms; investigate dropped frames rather than averaging away stalls. |
| Memory | A 30-minute session shows no retained-heap upward trend after warmup, and no increase in live AudioContexts, voices, listeners, workers or detached DOM. Test actual heap/resource counts; RSS fluctuations alone do not prove a leak or its absence. |
| Tick | Alarm when p99 simulation-tick latency exceeds 5 seconds, as specified. Track compute/transaction time separately from due-to-commit lateness, queue age and failed retries so a fast worker behind a long queue cannot appear healthy. |

Choose and record the exact physical browser/device fixtures in the first stage and retain them across release comparisons. Browser support is the last two major versions of Chrome, Safari, Firefox and Edge at release; feature-detect WebAudio/worklet capabilities within that matrix. Requalify when the supported window advances. Older unsupported browsers receive a small, matter-of-fact explanation of the required browser version, not a heavy compatibility bundle.

Run cold-navigation first-bird checks, not only warm local development reloads. Use a synthetic browser fleet from common geographies, compressed transfer accounting and screenshot/filmstrip verification of the first actual bird. Measure initial adoption separately from returning-owner loads so the intentional naming/entry state does not distort the metric. Track healthy real-world timing percentiles and the fraction over 500 ms; a cold-cache regression remains a regression even if aggregate averages look fast.

The memory soak alternates listen targets, offers when eligible, opens/closes panels, scrolls old notebook pages, changes captions/reduced motion, hides/restores the tab and repeatedly resumes audio. Sample retained heap after a fixed warmup and consistent collection strategy at intervals through minute 30. Gate on a stable plateau: no sustained positive slope above measurement noise, no monotonic retained-object count growth, and no more than a small predeclared noise band, initially 2 MB, from stabilized baseline. Investigate a failure with allocation/retainer profiles; do not widen the band to conceal it.

Bound the caption/cue dedup cache by timeline age, the note queue by lookahead, the notebook cache by page count and ornaments by pool size. Dispose of subscriptions, observers, audio nodes, timers and worklet resources on account change/revocation/unmount. There is at most one scene loop and one AudioContext per live application instance. Test 30-minute runtime, not only the first minute's frame rate.

### 13.2 What is and is not measured

Allowed aggregate RUM: navigation/load timing buckets, first-bird timings, frame-duration histograms, audio-context failure counts, coarse technical browser/capability categories and anonymized session-duration histograms. Server metrics: request counts, latency/error buckets, worker queue health, tick execution/lag, export/deletion job success and rate-limit health. Aggregate dimensions are low-cardinality technical dimensions such as build, browser family, synthetic test region and coarse operation family.

Do not attach account UUID, session UUID, invitation UUID, bird UUID/name/species, vector/mood, offer type, listen target, presence duration history, full URL, email or interaction payload to RUM/aggregate metrics. Coarse operational `command` health does not become per-offer usage analysis. Anonymous duration histograms are emitted as buckets without stable device identifiers and are not joined to server account logs. Exclude cookies/referrers from the RUM collection endpoint where possible and drop network identifiers before aggregate retention.

There is no session-replay SDK, general product-analytics event stream, training sink, recommendation dataset or production bird-population dashboard. Synthetic fixtures may report exact simulated traits and seven-bird test density because they contain no personal relationship data. Instrumentation schema tests reject forbidden fields, and a staging traffic/log review verifies that infrastructure middleware follows the schema rather than silently recording bodies.

Alerts should identify a technical problem and runbook action: tick p99 >5 seconds, sustained due-time lag approaching a minute, rising snapshot/auth failures, broken export/deletion jobs, first-bird budget regression, or an audio/render error spike. Do not alert the user about their aviary. The operating team can pause risky adoptions/new account intake, revert code or increase workers without clearing vectors or freezing established healthy accounts as a normal cost optimization.

## 14. Verification strategy

Build tests around failure modes and invariants, with seeded fixtures and a controllable server clock. Avoid tests that only restate UI markup or assert a helper's own implementation detail. Evidence includes automated checks and named human perceptual/accessibility reviews; one cannot substitute for the other.

| Area | Required cases and acceptance evidence |
| --- | --- |
| Drift correctness | Property tests over randomized valid event histories prove all traits bounded and nondecreasing; zero negative change across weeks of absence; no reset on rename, reconnect, migration or deletion recovery. Check one-session/week/three-week calibration trajectories and preservation of filter state through worker restart. |
| Presence precision | Exercise all eight combinations of visibility/focus/recent activity, the exact expiry boundary, still watching inside the window, hidden background overnight, blur, touch movement, synthetic events, dropped beacons and suspend/resume. Only the all-true combination credits time. |
| Multi-device timing | Two overlapping windows yield union time, not a sum. Duplicate/reordered heartbeats do not add time; split listen targets cannot exceed presence. Terminal settle/close cannot be reopened by a late ping. Owner/visitor overlap leaves host drift identical to the owner-only case. |
| Ordered tick writes | Crash before/after append acknowledgement and at each transaction boundary; race event append with tick; duplicate worker leases; fail commit and retry. Compare against a serial reference: no skipped sequence, double drift, lost cue or duplicate notebook entry. |
| Canonical continuity | Close all browsers while advancing the server clock through several local days; inspect server state before any new navigation. Mood/weather/call state continued. Open two devices, race replies and suspend one: both converge to current revision without vector uploads, neutral mood reset or replayed audio. |
| Command responsiveness | Greeting/offer/settle cues appear promptly after admission and survive the next tick. Retry the command before and after the tick and from a reconnect: same outcome/cue, no new cooldown/drift. Simultaneous offers respect per-bird reservations. |
| Settle | Click undo at 4.9 seconds; boundary at five seconds; keyboard equivalent; close without settle; lost close beacon; another owner's active device; later re-engagement. No duplicate presence, compulsory goodbye or abrupt gain/lighting discontinuity. |
| Notebook | Script several weeks of quiet/active fixtures; confirm sparse, factual, varied prose and no owner-attendance observations/numeric traits. Open an old entry after rename and pagination eviction; it stays accessible with historically stable text. |
| Adoption | Starters are fixed through reload; age eligibility advances without visits; no catalog/rarity; concurrent acceptance is idempotent; no eighth bird; seven remain visible and audible as individuals. |
| Authentication | Expired/used magic links, scanner GET, parallel consume, CSRF, unknown identity enumeration, revoked devices, email verification races, stale settings versions and direct cross-account object references. No email or bearer token appears in captured logs. |
| Visit isolation | Visitor attempts every owner mutation, including forged presence and owner UUID substitution. No greeting, host presence, notebook access or host state change occurs. Compare owner and visitor canonical projections field-for-field where shared. |
| Visit lifecycle | Expire unused invite at 30 days, redeem twice, refresh a valid capability, revoke active/unused invite, pull with ETag after revocation, lose network, hide/resume and expire session. Access ends at next validation/lease boundary; the unavailable text is consistent. Default-off sends zero host notifications; opt-in sends exactly one allowed message. |
| Account data lifecycle | Export at a single revision during a tick and while notebook pagination would span pages; verify complete schema and protected download. Delete, recover just before deadline, race recovery/purge, retry failed object deletion, restore a backup after purge. Confirm surviving birds retain identity and deleted data cannot reappear. |
| Visual aliveness | Filmstrip ordinary cold/warm returns: first frame is mid-action, primary greeting varies with seed/mood/absence, no entry wake-up/toast, no bird crops or teleports. Review day/night/weather and all supported widths with two and seven birds. |
| Audio/captions | Deterministic expansion matches actual notes and caption facts; distinct intents vary within signature limits. Test overlapping calls, gain ramps, fast listen changes, mute, settle, suspended context, worklet failure and unavailable WebAudio. No recorded asset request occurs. |
| Human audio review | Listen to long sessions and blinded signature comparisons across moods/drift and two-to-seven-bird mixtures. Suggested initial recognition target: at least 80% correct bird attribution among familiar signatures at seven birds, with no persistent uncanny/harsh/fatiguing pattern. Tune before increasing capacity; a passing DSP test alone is insufficient. |
| Accessible aliveness | Complete the core session through screen reader, keyboard, reduced motion and silence. Verify 30–60 second idle prose cadence, prompt user-event observations, no double live-region narration, visible focus, readable captions and no lost interaction. Human reviewers judge naturalist voice and calmness. |
| Performance/privacy | Enforce bundle, first-bird, sustained-frame and 30-minute memory gates in CI/release automation. Exercise each technical alert with synthetic failures. Scan built DTOs, DOM/ARIA, telemetry payloads and operational logs for prohibited state/PII and verify monitoring roles cannot query simulation storage. |

Model migration tests include a backup fixture with long-lived named birds, nontrivial filter state and an old grammar version. Upgrade and roll back code against it without changing IDs or decreasing a trait. If a migration needs a new field, supply a compatible default without replacing the vector. Changing sound implementation must preserve audible signature or keep the older grammar for existing birds.

## 15. Delivery sequence and rollout

Use sequential integration gates with parallelizable engineering responsibilities inside each stage; this plan does not require planner subagents. Keep the team's normal work tracking separate from the user-facing product, which has no progress/gamification surface.

| Stage | Responsibility and concrete output | Exit gate |
| --- | --- | --- |
| 1. Contracts and perceptual slice | Delivery lead plus backend, visual, audio and accessibility specialists agree the decision table, schemas, palette/pose sheets, six signature families and physical test fixtures. Build a throwaway/synthetic two-bird vertical slice with the actual render/audio/accessibility boundaries, not a recorded demo. | A continuing first frame, varied notice, listen mix and reduced-motion/narration alternatives meet the affective direction; critical payload fits the first-bird strategy. No unsupported dependency on external design documents remains. |
| 2. Canonical persistence and clocks | Backend implements identity/session base, aviary/bird storage, ordered event admission, tick scheduling, drift/mood/weather, migrations and consistent snapshot projection. | Serial-reference, crash/retry, multi-device presence and week-scale synthetic trajectories pass. State visibly advances with every client closed. Vector backup/restore works. |
| 3. Complete owner session | Frontend/audio integrate greetings, procedural tracks, top-bar controls, offers/cooldowns, settle/undo, names and sparse notebook. Accessibility specialist owns narration cadence, captions, keyboard and alternate poses throughout. | End-to-end owner session passes on the support matrix, including blocked audio, narrow phone, suspend/resume and account errors. No numeric/attendance surfaces or canned greetings appear. |
| 4. Lifecycle and quiet visits | Backend/frontend complete adoption ages/cap, email change, device revocation, export, deletion/recovery, per-invite visits, log and explicitly optional visit email. | Authorization matrix, token/revocation races and data-lifecycle drills pass. Visitor activity has zero effect on host vectors or presence. |
| 5. Hardening and long-run calibration | Performance engineer/test lead run fleet first-bird checks, 30-minute memory/frame soaks, seven-bird call recognition and long-timescale synthetic/live-clock test aviaries. Review final naturalist/system copy and privacy plumbing. | All launch invariants and hard budgets have evidence. Human audio/accessibility reviews sign off; instrumentation contains only allowed technical data. |
| 6. Limited release and expansion | Operations releases to a small bounded owner cohort, monitors technical health, then expands account capacity only with worker/database headroom and unchanged user-state guarantees. | Stable operational/error budgets and qualitative product feedback; no engagement or production drift dashboard is used to justify expansion. |

All functional v1 scope, including visits, export/deletion and accessibility, must be ready before calling the release v1. A private technical preview may test smaller slices, clearly labelled internally, but does not redefine missing requirements as later enhancements.

Instrument operational performance and privacy-schema checks from stage 1, not as post-launch add-ons. Put tick transaction time, due lag, snapshot load failures, actual first-bird rendering, client frame/memory probes in synthetic runs, audio-context errors, and export/deletion completion on the initial technical dashboard. Use only the aggregate/operational categories defined above.

Bird-count rollout has two separate clocks:

- Engineering first validates synthetic two-bird aviaries, then three/four, then all seven, including seven distinct individual signatures drawn from six species. Synthetic age advancement tests all adoption dates before production release.
- Actual owners always start with two. Additional birds appear solely through the documented age-based opportunities. Initial production cohorts therefore naturally have two birds; later increases do not depend on visit count or paying. Release capacity gates must be ready before the first relevant age threshold and cannot remove an adopted bird or reset an established identity. Never advertise a counter or faster adoption for active users.

Use server-side, versioned configuration for drift coefficients, mood weights, weather frequency, notebook sparsity and call-density ceilings. Changes apply prospectively, preserve stored filter/vector state and are validated on synthetic trajectories before rollout. Do not silently make personalized or engagement-optimized variants. Record config versions in canonical state for diagnosis without exporting state to analytics.

Rollback paths: revert faulty UI/audio code while retaining compatible snapshot/grammar versions; temporarily disable issuance of new invitations or new adoption acceptance if those paths are broken; slow new-account intake if scheduled tick capacity is threatened. Keep existing vectors and notebook intact. During a worker incident, recover from committed state and ordered catch-up, never restore a baseline bird or accept a client snapshot as truth. A release cannot solve load by ceasing simulation whenever owners are absent.

## 16. Risks and mitigations

| Risk | Detection and mitigation |
| --- | --- |
| Drift is perceptibly too fast or effectively invisible | Synthetic daily profiles, bounded single-session deltas and blinded one/three-week comparisons. Tune low-pass rate and expression mapping together; do not inspect production users' histories. Preserve monotonicity when tuning. |
| Positive drift saturates and birds converge | Moderate heterogeneous seeds, asymptotic `(1-current)` increments, immutable signature/individual style and saturated daily signals. Test months/years of synthetic time so all birds do not become identical front-perch callers. |
| Multi-device race silently erases personality | No client vector endpoint; per-aviary ordering lock, additive worker transaction, durable cursor/idempotency and crash tests. Backups and migration tests treat identity/filter persistence as core data safety. |
| Presence inflation changes the entire calibration | Explicit three-way conjunction, bounded retrospective pings, union intervals, terminal windows and no visitor reporter. Keep condition truth-table tests alongside the drift reference harness. |
| A one-minute tick makes interactions feel delayed | Persist immediate server-authored reaction cues without updating vectors/mood outside the tick. Reuse outcomes on the next tick and across devices; test duplicate cue suppression. |
| Calls feel artificial, harsh or indistinguishable | Bounded signature families, silence between phrases, real variation, conservative mixing and long human listening sessions. Test seven birds before age-based growth reaches them. Retain older grammar identities when changing synthesis code. |
| Accessible modes lose the relationship | Co-design prose and still-pose surfaces in the first slice, test multimodal combinations and require human review before release. Avoid flooding narration or substituting raw state labels. |
| Cold load violates the already-alive conceit | Inline private snapshot/pose, a tiny critical controller and species assets, no audio/framework gate, real mobile filmstrips and strict bundle/transfer budgets. Quiet-field error handling cannot be used to pass the first-bird metric. |
| Background simulation is too expensive or gets behind | Phase-distributed indexed due scheduling, bounded work, load-tested account caps, independent queue-lag monitoring and scalable workers. Optimize reference-equivalent computation, never redefine absent birds as paused. |
| Revoked visits keep receiving cached state | Per-pull authoritative authorization including 304s, short display lease, no shared snapshot caching, suspension revalidation and audio cancellation. Test dropped networks as well as successful revoke responses. |
| PII or private behavior leaks through ordinary tooling | UUID-only internal references, identity-only email storage, allowlisted DTO/metrics/log contracts, no body/replay capture, separated DB roles and erasure tests. Examine staging middleware outputs, not only application call sites. |
| Export/notification wording hides genuine PRD contradictions | Keep the explicit narrow decisions in section 2 and implement their boundaries exactly. Do not let an export exception become a stats panel or an optional visit email become reminder marketing. |
| Soft deletion or backups resurrect a different bird | Retain identity/vector/filter state during recovery, serialize recovery/purge, destroy account data keys at hard deletion and rehearse restores. Never regenerate missing personality as an automatic recovery shortcut. |
| Quiet chrome becomes undiscoverable or inaccessible | Four large named controls, focus/touch persistence, measured contrast floors and keyboard testing. Sparse appearance must not remove usable actions. |
| Notebook becomes repetitive, frequent or gamified | Fact-backed templates, sparse admission, local repetition guards and multi-week fixture review. No per-session logs, unread badges, visit calendars or owner-attendance prose. |

Completion means an engineering team can ship the scoped product with measured continuity, sync correctness, sensory quality, accessibility and operational performance. A successful launch preserves the birds the user has come to know; it does not substitute counters, notifications or a resettable simulation for that relationship.
