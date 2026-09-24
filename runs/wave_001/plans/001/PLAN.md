# Pocket Aviary — v1 implementation plan

## 1. Product contract and delivery boundary

Build one private, browser-based aviary per account. A new account meets two server-selected birds, names them, and can eventually adopt up to seven as the aviary ages. The single horizontal scene shows birds already occupied with their own lives. The host can watch, listen in, offer a seed/song fragment/still pool, settle the scene, and read a sparse, system-written field notebook. Email magic-link accounts, cross-device continuity, optional read-only visits, export/deletion, call captions, naturalist screen-reader narration, and a designed reduced-motion scene are all v1 work. Support the last two major Chrome, Safari, Firefox, and Edge versions.

The governing experience tests are: the first rendered bird is already mid-action; a returning host is noticed by one bird within 1–2 seconds without textual welcome; a week of regular visits causes instrument-measurable but not session-visible drift; roughly three weeks causes perceptible change; absence never harms a bird; and two devices show one continuous aviary. Product copy is lowercase, present-tense, bird-specific naturalist prose. Sign-in, errors, sync, account, and accessibility settings use direct, matter-of-fact language.

Do not build native clients, payments, extra aviaries, scene customization, public discovery, profiles/follows/chat/comments/co-presence, leaderboards, achievements, scores, levels, streaks, visit-frequency displays, hunger/distress/death mechanics, or push reminders. No personality numbers appear in the aviary, notebook, bird settings, captions, narration, or visitor view. No text toast or banner announces a return. An offer is a gesture, never food required for survival.

### Decisions that settle PRD ambiguities

1. **Settle and the four-icon top bar.** Keep the named account, accessibility, notebook, and offer icons. The offer icon opens a small gesture menu containing the three offers and a distinct **settle** action. Thus settle is reached from the top bar without adding a fifth permanent icon or controls inside the scene. Validate this placement in usability testing before visual lock.
2. **Local time and cross-device consistency.** Store an IANA `aviary_timezone` on the canonical aviary, initially supplied by the first host device. A foreground host device that reports a different zone can update it through a versioned account mutation; the newest authenticated foreground update wins, and all clients receive the same new zone and lighting phase. Settings also allow correction. Never independently calculate two different canonical mornings on two devices. Smooth the visual transition on travel while the server uses the new zone for later ticks.
3. **Hidden traits versus account export.** The explicit export requirement includes current personality vectors. Treat the downloaded, verified-email JSON as a narrow data-portability exception to the no-numeric-traits product rule. Do not expose the values in interactive UI, visitor access, or telemetry; do not make exports an optimization dashboard.
4. **Optional visit notification versus no notifications.** The sole exception is the host's explicit, off-by-default visit-notification setting. If enabled, send one quiet email per visit; do not create push, badges, or generic aviary reminders. Invites themselves are transactional email to the named visitor.
5. **Browser audio policy.** Try to start procedural calls on scene entry when permitted. If the browser suspends audio pending user action, show the live visual aviary, enable call captions, and expose a direct sound control in accessibility settings; resume WebAudio on the next deliberate gesture. Never substitute recorded loops or a splash screen.

## 2. Architecture and ownership

Use a small web client, an authenticated API, a relational primary database, and a scheduled simulation worker. The API serves privately bootstrapped snapshots and appends validated interaction events. The worker alone changes canonical bird personality, mood, perch intent, weather, call schedule, and notebook facts. The browser renders/interpolates snapshots, synthesizes calls, and owns only local presentation state such as focus, listen-in gain, caption preference, and the temporary settle lighting transition. A visitor uses a separate read-only capability path and never opens an interaction or presence channel.

```text
browser shell / SVG scene / WebAudio / accessible prose
       | GET private bootstrap + snapshot; POST host events
       v
auth + aviary API ---- append-only, per-aviary ordered events
       |                         |
       v                         v
relational canonical state <-- minute simulation worker
       |
       +--> private host and authorized visitor snapshots

operational metrics <-- dedicated aggregate-only counters/timings
```

Keep the first paint path distinct from deferred settings, notebook, invite, and export code. Return a small, personalized bootstrap state with the HTML through an edge-served route; mark it private and `no-store`, never place it in a shared CDN cache. Server-render the initial SVG bird/perch geometry from that state so a bird appears before hydration or audio initialization. Embed a snapshot revision, server timestamp, active motion phase, and deterministic scene seed so hydration continues that pose rather than replaying an entry animation. A slow network gets the quiet sky/field shell, with no spinner, until the first state arrives. Only the one-time adoption transition may begin from an empty field and softly fly in a bird.

For resilience, use a primary-state transaction for every tick and event append, a read replica or private edge read-through cache only if it can enforce monotonic revisions, and an explicit cache bypass after a host mutation. A client that receives an older revision keeps its newer state and retries. If a snapshot is unavailable, keep the last observed scene with a matter-of-fact retry surface in chrome; do not fabricate canonical drift locally.

## 3. Persistent model and privacy boundaries

Use UUIDs for account, aviary, bird, device session, notebook entry, invite, and visit. Never use email as a database foreign key, shard/partition key, telemetry dimension, or log identifier. Encrypt the account email on the account row; keep an HMAC lookup value there for sign-in. Pending magic-link and invite delivery records may hold a separately encrypted recipient address only for their short transactional lifetime. Redact emails and tokens from request logs. Store token hashes, not bearer tokens.

| Record | Required fields and invariants |
| --- | --- |
| `account` | UUID, encrypted verified email, email lookup HMAC, creation/deletion timestamps, accessibility/audio/visit-notification settings. Exactly one aviary. Email change is pending until new-address verification; old address remains active until then. |
| `device_session` | UUID, account UUID, hashed rotating token, created/last-used/revoked timestamps, limited device description. Every request checks revocation. |
| `aviary` | UUID, account UUID unique, created timestamp, IANA timezone and version, weather seed/state, last host-presence end, transient expressiveness factor, state revision, event sequence cursor, last logical tick. |
| `bird` | Immutable UUID, aviary UUID, species ID, mutable name, adoption timestamp/order, immutable call-signature seed, five normalized personality scalars (`boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`), current mood and dwell timer, perch/pose intent, grammar version. Only the tick's database role may update personality. A rename never changes identity or seed. |
| `interaction_event` | Aviary UUID, per-aviary sequence, idempotency UUID, host device session, server receipt time, validated type/payload, consumed tick. Types include bounded presence interval/end, listen-in start/end, offer accepted/rejected, settle, and adoption/rename audit facts as needed. Append-only writes; retention/pruning only after safe checkpoints. |
| `notebook_entry` | UUID, aviary UUID, time, referenced bird IDs and source-fact IDs, naturalist prose, template version. Immutable and pageable oldest-to-newest; retain until account deletion. |
| `invitation` / `visit` | Host UUID, encrypted invited email, hashed one-time link, 30-day unused expiry, redemption/revocation timestamps, short-lived visitor capability hash; visit start/end and approximate duration. No visitor presence or bird events. |

Snapshot payloads carry IDs/names/species, qualitative mood and pose, perched coordinates, timed transitions, call grammar parameters/seeds and schedule horizon, weather/lighting, notebook cursor, `revision`, and `server_time`. They do **not** contain numerical personality values, interaction history, account email, or hidden visitor data. A client needs only derived rendering parameters, never the raw vector. Keep schema/grammar versions with records and migrate without regenerating bird IDs or seeds. Back up and restore vectors as canonical data; a restore test must prove an existing bird retains identity and values.

Separate operational telemetry infrastructure from the simulation database. It receives only request/error counts, latency distributions, anonymous session-duration histograms, client frame timings, first-bird timings, audio errors, and tick timings. It has no job, credential, connector, or warehouse feed that can read per-bird events/vectors/notebook. Do not compute population drift, per-bird engagement, or cross-account interaction aggregates. Retain consumed raw events only for the recovery window needed by the tick (initially 30 days), then prune; notebook and canonical vectors remain. Export or deletion reads the private service, not analytics.

## 4. API and command contracts

Version the API (`/api/v1`) and use secure, HTTP-only, same-site host session cookies with CSRF protection on mutations. Validate origin, payload schema, bird ownership, maximum body sizes, and idempotency on every write. The host and visitor authorization paths are separate by design.

| Surface | Contract |
| --- | --- |
| `POST /auth/magic-links`, `POST /auth/consume` | Request link with per-email and IP limits; opaque random token expires in 15 minutes and is consumed atomically once. Successful consumption creates a revocable per-device session. Return direct errors for expired/replayed links. |
| `GET /aviary`, `GET /aviary/snapshots?after=revision` | Host snapshot, server time, interpolation horizon, and optional one-session greeting cue. Return `ETag`/revision. Pull immediately on visibility return or long frame gap and at a low visible-tab interval (initial target 20–30 seconds). No client tick. |
| `POST /aviary/events` | Batch events with `event_id` idempotency UUID, device sequence, and bounded interval metadata. Return accepted event sequence and any rejected event with a precise reason. Never accept client-written vectors, mood, or absolute perch. |
| `POST /aviary/offers` | `{event_id, bird_id, kind, fragment_id?}`. Under aviary lock, enforce an initial three-minute per-bird cooldown, validate the song-fragment library, choose a server-seeded reaction from current mood/personality, append the event, and return a transient reaction cue. Rejections return remaining cooldown without a punitive message. |
| `POST /aviary/adoptions`, `PATCH /birds/{id}` | Enforce age eligibility, seven-bird cap, server-selected species, idempotent creation, and valid names. Rename changes only `name`. Starter birds are created in one transaction at first account setup. |
| `GET /notebook?cursor=` | Read-only, stable pagination back through the full notebook. No edit/delete endpoint. |
| `GET/PATCH /settings`, `GET/DELETE /sessions/{id}`, `POST /account/email-change` | Update preferences/timezone, list/revoke sessions, verify new email before switching. Settings and errors use system voice. |
| `POST /account/export`, `POST /account/delete`, `POST /account/restore` | Generate a consistent JSON snapshot on demand, email a short-lived single-use download link to verified email; enter 30-day soft deletion; restore during window after sign-in. |
| `POST /invites`, `GET /invites`, `DELETE /invites/{id}`, `GET /visits` | Deliberate per-email invitation, current outstanding/active grants, immediate revocation, and on-demand visit log. No default invite or onboarding prompt. |
| `GET /visit/{token}`, `GET /visit/{grant}/snapshot` | Atomically redeem a one-time invite into a short-lived read-only browser grant. Check expiry/revocation on **every** snapshot request. Hide all host mutation controls and notebook. A revoked/expired grant returns the same matter-of-fact unavailable surface. |

The visitor view receives the same canonical scene, calls, time, and weather as the host. It cannot call host event routes, cannot trigger a greeting or listen-in, and cannot change a bird even if it crafts requests. Its local accessibility preferences remain available. A visible visitor pulls often enough that revocation terminates the view at the next pull (target at most 30 seconds); a hidden visitor must validate before rendering again. Record visit duration approximately from grant activity, without treating it as host presence. Show recipient email and visits only in the host's settings log, without badges or host alerts by default.

## 5. Server simulation and calibration

Schedule one logical tick per aviary per minute, including when no device is connected. A worker claims a due aviary, takes a per-aviary transactional lock, reads committed events after its cursor in sequence, computes the next state, writes bird/aviary/notebook changes and the new cursor together, and commits. Event insertion obtains that same lock briefly to assign a gap-free per-aviary sequence; thus a late commit cannot be skipped by a tick watermark. A unique `(aviary_id, logical_minute)` tick key makes retries idempotent. During worker outages, replay every missing logical minute in order from persisted state and event times; never jump directly to a freshly recomputed bird. Alarm when tick latency p99 exceeds five seconds, and alert separately on growing due-tick lag.

All stochastic behavior uses stable bird/aviary seeds plus logical minute and grammar version. Re-running a tick after a crash gives the same result. The worker persists vectors directly; it never rebuilds them from a session log on ordinary reads. Make migrations additive, backup vectors and seeds, and compare restored records against checksums in recovery drills.

### Presence accounting

On the host client, maintain a monotonic local activity timestamp from **pointermove or keypress only**. A candidate interval is present only while `document.visibilityState === 'visible'`, the window has focus, and activity is within an initial four-minute window (calibrate within the PRD's “few minutes”). Recompute at each visibility/focus/activity transition and on a 15–20-second heartbeat. End immediately on blur, hide, settle, or pagehide; use `sendBeacon` as a best-effort end and a short server lease expiry if it is lost. A quiet watcher remains present for the configured activity window after the last movement; simply leaving a tab open does not count.

The server accepts only authenticated host intervals with monotonic device sequence, bounded interval length, plausible clock skew and receipt age, and no continuation after settle/revocation. Missing heartbeats cannot be filled with an unlimited client claim. Union overlapping valid intervals from the host's devices so two open devices do not double-count presence. The tick consumes *presence-time*, not heartbeat count. A test matrix must cover hidden-but-focused, visible-but-unfocused, inactive, suspend/resume, two-device overlap, late beacons, and mobile/touch behavior. The strict pointermove/keypress definition may undercount motionless touch users; test it explicitly with assistive and phone input, and adjust the calibrated activity window rather than silently broadening the PRD's event definition.

### Drift

Normalize traits to `[0,1]` and seed each species/bird in a varied but recognizable middle range. Treat the following as an initial calibration model, with constants held in a versioned engine configuration and fitted only against synthetic fixtures and consented qualitative observation (never population interaction analytics):

```text
P_day = min(unioned host-presence minutes today, 30) / 30
L_b   = min(valid listen-in minutes for bird b today, 10) / 10
O_b   = min(accepted, cooldown-respecting offers for b today, 2) / 2
X_b,i = wP_i * P_day + wL_i * L_b + wO_i * O_b
p_b,i(next) = min(1, p_b,i + (1 - p_b,i) * k_i * incremental(X_b,i))
```

Start with presence carrying at least 70% of each relevant trait's total possible daily input; listen-in favors social warmth/vocal frequency, an approached offer slightly favors boldness, and an accepted offer slightly favors curiosity. `k_i` starts near 0.01–0.015 per fully signaled day and is tuned with time-compressed, seeded week/month scenarios. Apply only the newly earned daily input at each tick; track daily caps so repeated heartbeats, repeated offers, and retries cannot multiply it. Plumage saturation responds primarily to sustained presence. All deltas are nonnegative and diminish near one; no neglect or settle event subtracts a trait. A normal single session stays below visual detectability, seven regular days yield measurable stored deltas, and about 21 days yield recognizable differences in perch approach, greeting/reply likelihood, call frequency, or plumage. Gate calibration with both numeric fixture assertions and blinded perceptual reviews.

Absence may lower a separate, bounded **transient expression gain** (greeting/call willingness) toward a nonzero ambient floor over days; it never changes the stored vector downward, never produces distress, and rebounds with ordinary presence. This resolves “quieter after two weeks” without punishing the bird or the host. Settle ends presence and briefly quiets mood/calls; it has no independent drift reward or penalty.

### Mood, place, greetings, notebook

Use `wary`, `content`, `curious`, `drowsy`, and `alert` as the initial mood enum. A seeded semi-Markov transition samples mood only after a minimum dwell time, with weights from current mood, local time, recent accepted offers, soft ambient weather, nearby birds' calls/wariness, and personality. At local dawn, relax toward a day baseline over multiple ticks rather than snapping to neutral. Mood persists across sessions; the client never resets it on open. Give wary birds more back-perch intent, bold birds more front-perch intent, content birds more preening, curious birds more tilts, and drowsy birds low/fluffed poses. Bird-to-bird response windows yield occasional real call-and-answer/chorus events, not independent loops.

Create rare, short rain or wind windows from the aviary seed (initial target a few per week), persist the current window, and let it temporarily adjust mood/call probability. The scene's leaves and feathers are local, seeded rendering ornaments and do not require database records. Local time controls gradual light and chorus attenuation; at night most birds settle while the nightjar-like species may remain active.

On a fresh host entry or visibility return, the server derives absence length from the last genuine host-presence end and chooses one greeter using boldness, warmth, mood, and a seeded variation. Return a cue that starts within 1–2 seconds **after** the already-active first frame: glance for a short absence; more reorientation/longer call/front approach for a long absence. Keep a stable recognizable manner for each bird but vary microtiming, pose, and motif. A possible second response is staggered, never synchronized on arrival. No visitor greeting. Capture a source fact if an observed moment merits a notebook entry.

Generate notebook entries from eligible factual state changes and events through a reviewed naturalist prose grammar with bird names, time, perch, weather, and comparisons to previous facts. Validate every referenced fact against canonical state, avoid references to user visit frequency, reject repetitive phrasing, and enforce a sparse per-aviary budget (initially about one entry per few days, with a narrow exception for unusual moments). The notebook is an observation log, not a session/event dump. Store immutable prose and template version so later wording changes do not rewrite the user's history.

### Adoption

Age, not activity, unlocks additional adoption opportunities. Seed the first two species on account creation; present them as arrivals with suggested editable names, not a species catalog. Start eligibility for bird three around day 90, then approximately days 180, 270, 365, and 540 for birds four through seven. Treat those intervals as release-calibrated configuration, never a progress bar, visit reward, score, notification, or countdown. A quiet “adopt a bird” action appears in account settings when eligible; the API enforces one adoption per age slot and the seven-bird maximum. New species comes from the coherent six-species pool, balancing distinct silhouettes/calls without rarity tiers.

## 6. Rendering and interaction pipeline

Use a lightweight SVG/DOM scene with three normalized perch zones and compact per-species vector shapes. Place every bird within a safe horizontal/vertical box at narrow and wide viewports; redistribute spacing rather than cropping, scrolling, panning, or zooming. Background sky/foliage and occasional foreground elements form shallow depth; parallax is subtle. Derive plumage richness and pose parameters from server-provided qualitative render tokens, not raw vector values. Avoid expensive per-frame SVG filters. Animate transforms and opacity on bounded layers to stay on the compositor where available.

The snapshot contains state at `server_time`, transition start/end times, pose phase, and a short deterministic horizon. Estimate server clock offset and interpolate movements/poses on `requestAnimationFrame`; a newer revision changes intent but does not teleport the bird. If a long frame gap or tab resume occurs, stop rendering while hidden, fetch a fresh snapshot first, then resume at its current phase. Weather leaves/feathers run only while visible. The first frame is the server-rendered ongoing pose. Keep all scene controls out of the scene; call captions and focus indication are accessibility overlays, not navigation chrome.

The top bar fades nearly transparent after a few seconds of pointer stillness, and returns on pointer movement, keyboard activity, or focus. Never fade controls while a keyboard user has focus in them. The offer menu is navigable and does not obscure birds unnecessarily. Listen-in starts on bird click/tap or Enter on a focused bird, disengages on repeat, another bird, empty scene, Escape, or focus leaving the bird region. Its local mix responds immediately; the server logs bounded start/end duration for later drift. The offer API returns the authoritative reaction cue before animating it; on failure show a direct, quiet system message and leave canonical state unchanged. Settling crossfades lighting/calls over seconds; any scene click in the first five seconds consumes the click and undoes settle. Give keyboard users an equivalent Escape/Enter undo. After five seconds, any deliberate new engagement restores the normal scene. Closing the tab and settling both terminate presence without a penalty.

Reduced-motion is a separate render mode chosen by `prefers-reduced-motion` or setting: slowly crossfade authored still poses, crossfade perch changes instead of flight paths, remove drifting leaves/feathers, and slow palette transitions. Keep canonical mood, calls/captions, notebook, and drift intact. Do not merely freeze the normal scene.

## 7. Procedural audio and captions

Author roughly six species motif libraries with distinct contours, rhythmic signatures, and timbres. Give each bird a stable signature seed that selects a recognizable subset; mood and personality vary tempo, pitch envelope, phrasing, call probability, and response timing within limits that preserve identity. The server supplies a seeded short-horizon call score/current active-call phase; the client runs the same grammar version to synthesize each note through WebAudio oscillators/noise, envelopes, and light filters. Schedule slightly ahead using the audio clock; synchronize score changes to snapshot revisions without replaying calls. Never download call loops. Keep nodes/buffers in bounded pools, disconnect completed voices, and cap simultaneous voices to the seven-bird chorus.

Mix individual birds into a limited master bus with headroom and soft limiting. Chorus scheduling permits overlapping independent calls and occasional replies while avoiding persistent masking. Listen-in ramps the chosen bird up and others down over about one second, but other birds retain an audible ambient floor; release over a similarly gentle ramp. Time-of-day and rain attenuate activity without muting the night scene. A song-fragment offer uses the same small procedural motif approach and allows a mood-shaped bird response.

Generate each call caption from the *actual realized motif parameters* and bird position/mood, for example a three-note rise or a paused low trill. Position the short text near the caller, keep it legible at AA contrast, and time its appearance to the call. Captions can be opted into at any time and default on if WebAudio is missing, blocked, or deliberately off; never fall back to recorded audio. Conduct blind recognition sessions across birds/moods and a seven-bird chorus test. Rework motifs/mixing if listeners cannot reliably recognize individual familiar birds or if calls sound looped or synthetic in the wrong way.

## 8. Accessibility as a full product surface

Generate slow naturalist narration from the same snapshot/facts as the scene, with concrete species, positions, light, sound, and notable behavior. At idle, update only about every 30–60 seconds and coalesce minor changes. Prioritize a return greeting, accepted offer reaction, or settle without flooding the screen-reader queue; write observations, not “mood changed” or coordinate readouts. Use one controlled live region plus an on-demand scene summary, and test actual speech queue behavior with major screen readers. Notebook prose remains the same voice. Controls and error text retain clear system labels.

Tab reaches the four top-bar controls and then the bird region. Arrow keys move between birds; Enter starts/stops listen-in; Escape exits listen-in or closes a menu; the gesture menu and settings are fully keyboard navigable. Use visible high-contrast focus treatment against both day and night, appropriate hit targets, and stable focus across snapshot updates/renames. Test captions, top-bar text, settings, notebook, errors, and focus cues against WCAG AA in each palette; do not rely on color or audio alone to express a call or focus. Audit reduced-motion crossfades for vestibular comfort rather than assuming any fade is safe. Test the full adoption, host, visit, export, and deletion paths with keyboard and screen reader before launch.

## 9. Performance, operations, and acceptance gates

Set CI gates for initial JavaScript under **2 MB gzipped**, measured at first paint including critical runtime chunks; lazy-load notebook/settings/invite/export code. Keep compact SVG species assets and procedural audio code; do not gate the first bird on audio, notebook, or noncritical assets. On an agreed mid-tier mobile/4G synthetic profile, require first bird visible within **500 ms** (track navigation-to-first-bird using a dedicated paint mark; use p95 in repeated release runs). Record aggregate-only real-user first-bird distributions to catch field regressions without account/bird identifiers. The privately embedded state should remain in the kilobyte range and be delivered with the shell.

On a five-year-old mid-range laptop, run a 30-minute, seven-bird, active-weather/chorus stress scene at **60 fps idle target** with no sustained frame budget overruns. Compare post-GC heap after warm-up and at 30 minutes; require no upward slope beyond a small measurement-noise tolerance, and inspect audio-node, timer, listener, and virtualized-notebook counts. A full half-hour soak is a CI/release gate, not a one-minute demo. Run browser-specific audio fallback and reduced-motion variants. Monitor tick computation p50/p95/p99, queue lag, snapshot size/latency, HTTP errors, first-bird paint, frame timing, and AudioContext failures from day one. Synthetic browsers run from common geographies; real-user metrics are anonymous aggregates only. Alert at simulation tick p99 over five seconds and on stale snapshots/due-tick backlog.

Release tests must also prove: vector nondecrease under neglect, cap-limited positive drift under event spam, one-week and three-week fixture targets, preserved IDs/vectors after migration and restore, no bird reset on open, two devices converging after concurrent events, no overlapping-presence double count, visitor events rejected, invitation revocation on next pull, 15-minute single-use magic links, 30-day unused invites, 30-day deletion/recovery, full export contents, no greeting toast, and no counters or visit-frequency prose. Use synthetic bird histories for engine calibration so real per-bird data never enters analytics.

## 10. Security, deletion, and recovery

Rate-limit magic-link and invitation email issuance, apply token entropy and hashing, expire and consume links atomically, rotate/revoke per-device sessions, and protect every host mutation against CSRF. Keep visitor grants scoped to one aviary and one read-only route; never grant notebook, account, or event permissions. A revocation changes grant state transactionally before the API responds. Use encrypted invitation recipient addresses only where needed to send and display the host's log. Never log link tokens, email, request bodies containing bird events, or vector values.

Account export takes a consistent transaction snapshot, includes birds/IDs/names/current vectors/moods, notebook, and account settings, and sends a short-lived single-use download link to the currently verified email. Deletion immediately disables active sessions, visits, invites, and new tick work while retaining recoverable state for 30 days. A returning user may explicitly restore during that period. At day 30, purge account, aviary, birds, events, notebook, invites/visits, session records, pending email jobs, and account-scoped operational records; cryptographically erase account-scoped backup encryption keys and verify that no recoverable copy remains. Keep deletion-job receipts free of per-bird or email data. Test restore before day 30 and failed restore after hard deletion.

## 11. Build sequence and rollout

1. **Foundation:** finalize SVG/audio/narration prototypes with designer and accessibility partners; lock API schemas, synthetic-ID/email boundary, vector ownership, canonical timezone, transactional event ordering, and fixture-based engine tests. Build private SSR bootstrap and first-bird timing mark early.
2. **Continuity core:** create account/adoption flow, persistent birds, minute worker, deterministic seeds, snapshot/interpolation path, and two-device convergence/recovery tests. Prove a bird survives reload, month-long catch-up, backup restore, and rename before adding ornamental polish.
3. **Life and gestures:** tune mood/perch/calls/weather/greetings, honest presence intervals, listen-in, offers/cooldowns, settle/undo, and sparse notebook. Review real-time and time-compressed behavior for canned cues, rapid drift, and quiet absence.
4. **Equal-access surface:** deliver reduced-motion scene, narration, captions/WebAudio silence path, keyboard controls, and AA palette/focus QA alongside the main scene. Do not defer these after launch.
5. **Account and social completion:** session revocation, email change, export/deletion, invite redemption/revocation, host visit log, off-by-default notification toggle, and privacy isolation audits.
6. **Release gates:** run multi-browser/device, audio recognition, accessibility, 30-minute soak, 4G first-bird, tick-lag, restore/deletion, and abuse tests. Remove any product-surface counters/toasts added during development.

Ship first to internal and invited beta accounts with two birds, then widen the cohort once paint, tick, sync, audio, and accessible-surface gates pass. Exercise 3-, 5-, and 7-bird synthetic aviaries before general release, then retain the age-based adoption schedule for real accounts; do not use interaction or visit frequency to accelerate access. Keep capability flags for the optional invite flow and higher bird counts so a regression can be contained without rewriting existing bird identities. Instrument aggregate health from the first cohort. Halt expansion on vector loss/reset, cross-device divergence, visitor writes, hidden-tab presence inflation, accessibility regressions, first-bird misses, sustained frame/memory failure, or recurrent tick lag. General availability includes all v1 paths even though a newly created aviary naturally begins with two birds.

## 12. Principal risks and responses

| Risk | Early evidence | Response |
| --- | --- | --- |
| Drift is too fast, too slow, or rewards clicking | Synthetic 7/21-day fixture misses targets; blinded users notice session jumps or no long-term change | Tune versioned caps/weights with time-compressed fixtures and qualitative review; preserve stored vectors and migrate coefficients without resetting birds. Presence stays dominant and deltas stay nonnegative. |
| Presence inflation or exclusion | Hidden/unfocused intervals accepted; duplicate device minutes; touch/screen-reader sessions undercount | Conjunction tests, bounded leases, union intervals, focused usability tests, and explicit calibration of the few-minute activity window. Never replace it with “tab open.” |
| Sync corrupts identity or loses drift | Nonmonotonic revision, skipped event sequence, vector mismatch after restore | Single server writer, per-aviary ordered event lock, idempotent tick, revision checks, migration/restore checksums, and fail-closed stale snapshot handling. |
| Procedural calls sound canned or chorus masks identity | Repeated audible motif or poor seven-bird recognition | Expand constrained grammar variation, preserve signature anchors, mix with headroom, human blind listening, and seed-based regression renders. Do not add recorded loops. |
| First bird feels like a loading transition | Paint mark misses 500 ms; hydration resets pose | Private SSR bird in initial HTML, tiny state, deferred code, seed/phase continuity, quiet-field-only slow path. |
| Accessible versions lose aliveness | Static reduced-motion scene, narration queue flood, captions disagree with audio | Separate crossfade composition, paced fact-based narration, captions from realized grammar, screen-reader/vestibular testing in release gates. |
| Privacy boundary erodes through analytics | Metric proposal requires bird/account fields; request logs contain emails/events | Separate telemetry credentials and schemas, allowlisted metric definitions, log redaction tests, privacy review of every new metric. |
| Invitation links outlive intent | Revoked visitor still sees snapshots; link replay | One-time redemption, grant checks on each pull, ≤30-second visible polling, pre-render validation on return, and revocation integration tests. |

The team should treat “feels alive” as a system property: continuity, restraint, recognizability, honest attention, and equal-access charm must survive the same end-to-end release gates as correctness and speed.
