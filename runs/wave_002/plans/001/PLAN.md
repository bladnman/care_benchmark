# Pocket Aviary v1 — implementation plan

## 1. Product contract and scope

Build a browser-only, single-user aviary that feels as though it continues between visits. The user adopts two starter birds, learns them through quiet observation, and can eventually have up to seven in one horizontal scene. Presence—not click volume or a reward loop—is the primary signal behind gradual, one-way personality drift. The server owns the one canonical aviary; clients render snapshots and submit events.

V1 includes email magic-link accounts, one aviary per account, two starter birds, age-paced additions up to seven, bird renaming, moods and the server-side tick, procedural calls, listen-in, offers, settle, the read-only field notebook, multi-device sync, account export/deletion/session management, optional per-invite read-only visits, narration, captions, keyboard support, and reduced-motion rendering.

Keep out of scope: native clients, multiple or shared aviaries, scene customization, payments, social discovery/profiles/follows/comments, leaderboards, achievements, streaks, scores, visible personality stats, hunger/distress/death, and push or engagement loops. Do not add a bird-count badge, visit calendar, activity dashboard, or copy that reports the user’s visit frequency. Perch placement remains bird-chosen. Visiters never affect host state.

### Decisions to settle during implementation

- Use the five example moods—wary, content, curious, drowsy, alert—as the initial enum; validate transitions during calibration. A daily boundary triggers reevaluation, not a reset to neutral.
- Store an IANA timezone on the aviary/account and use it for day/night and mood timing. Refresh it through account preferences when the user’s timezone changes.
- Treat visit notifications as a narrow exception to the broad “no notifications” principle: disabled by default, never prompted during onboarding, with the optional host setting described in `social_optional.md`. Confirm the delivery channel before enabling that setting. No visit badge or unsolicited message is sent by default.
- The offer affordance opens from the top bar. Offer kind and optional recipient are chosen in that panel, including a keyboard-accessible bird list; clicking a bird in the scene does not initiate an offer. This resolves the per-bird cooldown/recipient requirement without adding an in-scene control.
- Browser autoplay restrictions can prevent sound before a user gesture. Start/resume WebAudio silently on the first allowed gesture; until then preserve the moving scene and use captions when enabled or when audio is unavailable. Do not add an audio gate or welcome modal.
- Count overlapping valid presence leases from the same account as their time union, not as additive device-minutes. This prevents two open devices from accelerating drift.

## 2. Architecture and boundaries

Use a small web client plus authenticated application APIs, a canonical aviary/simulation service, durable account-scoped storage, and a scheduled tick worker. Supporting modules handle magic-link email, visit invitations, notebook reads, export/deletion jobs, and aggregate operational telemetry. Keep the exact framework and database choice open to the engineering team; preserve these boundaries regardless of stack.

The server is authoritative for bird identity, personality vectors, mood transitions, offer eligibility, event ordering, notebook creation, weather/call schedules, and snapshot versions. A client may animate between server states but cannot mutate canonical bird state. Each API write has an idempotency key. Each aviary snapshot has a monotonically increasing version and event-log watermark. Tick workers claim an aviary lease, process events in server sequence order, and commit state plus the consumed watermark transactionally (or with equivalent compare-and-swap). A retry must not apply an event or drift delta twice.

Keep operational telemetry physically and logically apart from simulation state. The telemetry collector accepts only aggregate request counts, latencies, errors, render timings, audio errors, and anonymous session-duration histograms; it has no read path to bird records or interaction logs. Logs and inter-service keys use the synthetic account UUID, never email.

## 3. Data model

All records are account-scoped by synthetic UUID. Email is encrypted on the account record and is never a partition key, trace attribute, or log identifier.

- **Account**: UUID, encrypted verified email, email-change challenge state, created/deletion timestamps, IANA timezone, account preferences, notification preference (default off), and schema version.
- **Device session**: opaque hashed/revocable token, account UUID, issue/expiry/revocation times, and minimum device label needed for the session list. Never log raw tokens.
- **Aviary**: UUID/account UUID, creation time (drives age pacing), canonical version, event watermark, last tick time, timezone, local day/night phase, settled-until/undo window, current ambient weather and its bounded expiry, and deterministic scene seed.
- **Bird**: stable UUID, aviary UUID, species from a pool of about six, user name, adoption time, hidden normalized personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), mood and mood timestamp/timer, perch zone, current pose/action and phase, stable call-signature seed, next-call schedule/counter, and last-offer time. Rename updates only the name. The client snapshot exposes appearance/pose outputs needed to render, never raw personality numbers. Personality vectors are available only in the account’s explicitly requested export, not in product UI or telemetry.
- **Interaction event**: server-assigned sequence, UUID/idempotency key, account/aviary UUID, event type, optional bird UUID, validated bounded payload, server receipt time, and client session ID. Types include presence lease start/heartbeat/end, listen-in start/end, offer, settle/settle-undo, and other allowed user action. Store only for the owner’s simulation; do not copy it to analytics. A visitor can never write this record.
- **Notebook entry**: stable ID, created time, referenced bird IDs, observation/fact inputs, approved prose/template version, and rendered naturalist text. Append-only to users, readable indefinitely; no edit/delete UI.
- **Visit invitation/log**: invitation ID, host UUID, invited email held only for delivery/visit transparency, one-time token hash, expiry, redemption/revocation state, and minimal visit start/end/approximate-duration records. Do not create public profiles or discovery indexes.

Starter adoption creates exactly two server-selected species and stable IDs; the user supplies names or accepts suggestions. New adoption opportunities depend only on aviary age and stop at seven. Do not expose a collection counter or make additions depend on attention, offers, or a paid tier.

## 4. API surface and user flows

Use authenticated, versioned APIs with bounded request bodies, idempotency keys on writes, and explicit matter-of-fact errors. Suggested resource shapes (names may adapt to house conventions):

- `POST /v1/auth/magic-links` requests a link with per-email rate limits. `POST /v1/auth/magic-links/consume` validates, consumes once, and issues a per-device session. Links expire after 15 minutes; a successful use invalidates the link. Email change verifies the new address before switching.
- `GET /v1/aviary/snapshot` returns the canonical public-to-render state, server time, snapshot version and event watermark; conditional requests may use ETag/version. Pull after boot, visibility return, long render gap, and a low-frequency visible-tab keepalive. The response includes per-bird renderable pose/perch/mood/call timing, scene/day-night/weather, transition phase, and any active interaction cue, but no numeric personality vector or event history.
- `POST /v1/aviary/events` accepts only known event types and bounded payloads. Presence heartbeats are accepted only for authenticated host sessions and credited for at most the interval since the previous fresh heartbeat; no arbitrary client backfill. Listen-in sends start/end with bird ID. Offer sends item kind and optional recipient; server checks eligibility/cooldown and returns an immediate deterministic reaction cue while recording the event for the next tick. Settle returns a five-second undo deadline; an in-scene click during that window sends settle-undo and starts a new presence lease. The client never sends trait values or a proposed full state.
- `GET /v1/aviary/notebook?cursor=…` returns paginated, immutable entries. Generate a sparse entry from notable aviary observations, usually no more than about one every few days for a regular aviary; a noteworthy event can qualify sooner subject to a sparsity cap. Never write a user-visit-frequency observation.
- Account settings endpoints list/revoke device sessions, update name/timezone/accessibility preferences, verify email changes, request a JSON export, and request/recover deletion. Export includes birds, names, current vectors and moods, notebook, and settings; generate on demand and email a short-lived download link to the verified address. Deletion is recoverable for 30 days, then hard-deletes birds, vectors, events, notebook, invites, sessions, and account-linked records.
- `POST /v1/visits/invitations` sends a named invite to one email; `GET` lists outstanding invites and the visit log; `DELETE /v1/visits/invitations/{id}` revokes. Unused invitations expire after 30 days. Redeem the emailed one-time token into a narrowly scoped, read-only visitor session. Visitor snapshot pulls re-check invite revocation/expiry on every pull and return a matter-of-fact unavailable surface after revocation. Visitor open/close/heartbeat data estimates visit duration without becoming presence or drift input.

Magic-link, session, invite, and export failures use plain system language. Product-facing narration, offer reactions, captions, and notebook text use specific lowercase naturalist prose. No welcome toast or banner appears on host return.

## 5. Simulation engine and calibration

### Tick and consistency

Run a server tick at approximately one-minute cadence for every aviary, including those without connected clients. Partition due aviaries across workers. Track `last_tick_at`, event watermark, and version so a delayed or restarted worker can calculate elapsed time and apply unconsumed events exactly once. Use an exclusive per-aviary lease or compare-and-swap transaction; never use last-write-wins state replacement. Alarm when tick latency p99 exceeds five seconds.

On each tick: consume ordered owner events; union overlapping valid presence intervals; update smoothed attention signals and positive personality deltas; evaluate mood transitions and timers; advance perch/pose/action and call schedule; update day/night and rare weather; generate eligible notebook observations; then commit one canonical snapshot/version. Server elapsed time, not a client clock, bounds each presence interval. A missed tick is recovered by elapsed-time calculations without inventing presence during the gap.

### Presence and drift

The browser reports a presence lease only while all three conditions hold: document visible, window focused, and pointer movement or keypress within a calibrated “few minutes” window (lean longer so quiet watching counts). Drop the lease immediately on hidden/unfocused state; tab close/pagehide is best-effort, with lease timeout as the reliable end. Send no raw pointer or key stream. Store only the active/inactive lease interval needed by that account’s simulation.

Implement drift as a low-pass filter over validated presence-time and secondary events. Presence is dominant; listen-in chiefly nudges the selected bird’s social warmth and vocal frequency; offers create a small curiosity nudge if accepted and a small boldness nudge for offering near a bird; settle quiets mood and closes the presence lease but adds no trait direction. Use bounded non-negative deltas toward each trait’s expressive end, with clamping at its upper range. No no-show or inactivity path subtracts from a trait. Keep constants, smoothing window, and per-event weights in versioned server configuration so calibration is reversible without resetting bird identity.

Build a deterministic simulation harness that replays synthetic presence and event traces against seeded aviaries. Calibrate so regular use produces instrument-measurable changes around one week and user-visible, gradual change around three weeks; a single session is not visually measurable. Include invariants for monotonic traits, absence never decreasing a trait, stable IDs, bounded caps, duplicate-event idempotency, and replay consistency. Keep numbers out of visual controls and product copy.

### Mood, social behavior, adoption, and notebook

Mood is fast state and personality is slow state. Persist mood across sessions; reevaluate at local day boundaries from recent owner interactions, local time, ambient events, and hidden personality. Use mild, deterministic-weighted transitions: rain briefly dampens calls, wind shifts alertness/wary behavior, an alarm call can gently spread wariness, and boldness moderates wary transitions. Absence can leave birds quieter/ambient, never hungry, distressed, angry, dead, or less colorful. Mood drives visible action and perch choice rather than a meter or label.

Return greeting chooses one likely bird using boldness, mood, and absence duration. Stagger other likely greetings with small seeded offsets; never trigger a unison welcome. A short absence may produce a glance, longer absence a reorientation/call, with procedural variation inside those constraints. For visitors, render the host’s current state without a host-arrival greeting, event submission, or presence credit.

Age-based adoption should be a quiet optional flow with server-owned thresholds: two at creation, a third after a few months, and progressively spaced offers such that a year-old aviary may have five or six; never more than seven. Final thresholds are product calibration values, independent of sessions or attention. The small species pool has no rarity or catalog-selection mechanic.

Generate notebook prose from approved naturalist templates/facts tied to actual bird/aviary events, never generic event rows or trait deltas. Keep the log sparse and useful indefinitely. Entries may note bird behavior (“pip greeted before wren today”) but cannot praise or count user attendance. Apply voice linting and review to templates; no free-form LLM generation is needed for v1.

## 6. Frontend scene and interaction pipeline

Split the shell into: (1) lightweight HTML top bar and accessible controls, (2) one renderer consuming normalized snapshots, (3) a narration/caption/accessibility layer derived from the same snapshot, and (4) interaction-event client. Keep domain state and simulation math out of the renderer. Prefer a compact 2D Canvas/SVG scene with generated/compact bird assets; avoid a large 3D runtime. The renderer covers a single horizontal, non-scrollable scene with front/middle/back perch zones, foliage/sky layers, and subtle parallax. No buttons, labels, hover icons, or controls are placed inside the scene.

The first meaningful frame uses the current server pose and animation phase, with ambient motion already in progress; do not fade in from static, show a spinner, or play a wake-up animation. The first signup has one real empty-aviary interval between naming and the two starter fly-ins; subsequent loads immediately show the continuing aviary. During an unavailable/slow state show the quiet sky field with faint motion, not a spinner. Deliver HTML, compact scene assets and a small boot snapshot fast; serve personalized boot data privately (never shared-cache one account’s state). CDN-cache only safe shared assets.

Interpolate between snapshots locally for smooth motion and reconstruct phases from server time. Stop scene rendering when hidden; on visibility return fetch the canonical snapshot before resuming. Match narrow phone layouts without cropping birds and widen desktop spacing while preserving all three zones. Use local scene time from the host’s timezone. Top bar contains only account/settings, accessibility, notebook, and offer controls; fade it nearly transparent after cursor stillness and restore on pointer or keyboard activity. Settle lives in the top bar, with a slow evening shift, quieter calls, and a five-second in-scene click undo.

Listen-in is a gradually ramped mix: focused bird louder, others reduced but always audible; clicking it again, choosing another bird, empty scene, or leaving focus disengages with a slow ramp. Offers are initiated in the top-bar panel, with seed/song-fragment/still-pool choices and a receiver chosen through the panel. Apply a few-minute per-bird cooldown and explain in quiet, matter-of-fact control state if that bird is cooling down; do not turn cooldown into a reward timer. Offer response comes from server-returned cue and current bird mood/personality; no direct bird-click offer interaction.

## 7. Procedural audio

Build a small WebAudio synthesizer around per-species motif grammars. Each stable bird has a seeded signature; a call sequence counter plus constrained timing/pitch variation gives fresh calls while preserving species/bird recognizability through mood and drift. Mood changes articulation/timing; vocal-frequency changes call probability and chorus participation, not the bird’s recognizable core signature. The server publishes call timing/grammar version/seeded cue in snapshots; the client synthesizes it. No recorded-call download or recorded fallback.

Use a bounded voice pool and reusable buffers/nodes; schedule calls with low allocation and disposal discipline. A chorus mixer balances calls spatially/temporally without turning simultaneous motifs into identical loops. Rain and evening lower call activity. Song-fragment offers use the compact motif library, not recorded music. Generate the caption phrase from the same grammar parameters used for the rendered call so caption and audio match. Listen-in gain envelopes are gradual. If WebAudio is missing, denied, or fails, remain silent and turn captions on by default. Respect the user’s mute preference and stop local audio when the page is hidden.

## 8. Accessibility and performance

Ship accessibility as part of v1. Provide a screen-reader running naturalist narration sourced from the same snapshot, roughly every 30–60 seconds at idle and promptly after user-initiated greeting/offer/settle actions. Queue updates so they do not overlap or chatter. Do not announce a list of coordinates, mood codes, or personality numbers. Captions are opt-in when audio works, short naturalist descriptions near the calling bird, and generated from each actual procedural call. In unavailable-audio fallback, captions default on.

Honor `prefers-reduced-motion` and an account setting. Render slow cross-fades between still poses, cross-fade perch changes, remove drifting leaves, and slow day/evening palette shifts. Calls and simulation continue. Every interactive surface works by keyboard: tab through top bar, enter scene at first bird, arrow between birds, Enter toggles listen-in, Escape exits, and offer/settle/settings/notebook controls are keyboard reachable. Focus rings remain visible over every scene palette. All user-copy text meets WCAG AA contrast; older than the last two major releases of Chrome/Safari/Firefox/Edge receive a clear unsupported-browser page.

Performance gates:

- Initial JavaScript bundle below 2 MB gzipped.
- First bird visible under 500 ms on a mid-tier mobile device over 4G; measure navigation to first visible bird from synthetic browsers and privacy-safe RUM.
- 60 fps idle motion on a five-year-old mid-range laptop.
- No client memory growth over a 30-minute session; bound audio contexts/workers, reuse audio buffers, and release notebook scrolled-out view references.
- Snapshot payload remains kilobytes and does not wait for non-critical assets before painting the first bird.

## 9. Security, privacy, and observability

Rate-limit magic-link requests per email without exposing account existence. Hash single-use tokens at rest, use short expiry, invalidate on use, and support per-device revocation. Restrict visitor credentials to read-only snapshot access; check revocation at each pull. Never record visitor sessions as host presence, listen-in, offers, or drift. Encrypt account email; only account service and mail delivery can access it. Account identifiers in logs are synthetic UUIDs. Protect export links with short expiry and verified-email delivery.

Interaction records are retained only for the owner’s simulation, never model training, recommendation, population-level bird analysis, third-party sharing, or the analytics warehouse. Aggregate operations data excludes account IDs and bird fields. Collect page/load/first-bird/frame timings, audio-context failures, API/tick latency and errors, synthetic browser checks from common geographies, and anonymized session-duration histograms. Do not log a user’s bird behavior, call content, or individual session timeline as an observability event. Account deletion hard-removes account-linked state after 30 days; make backups and export artifacts expire on a schedule compatible with that promise.

## 10. Delivery sequence and release gates

1. **Contracts and calibration foundations**: confirm mood enum, local-time handling, presence window, drift parameters, age thresholds, and visit notification channel. Define schemas, API contracts, event/idempotency rules, prose voice rules, and privacy data-flow boundaries. Build seeded simulation replay harness before product tuning.
2. **Account and canonical engine**: implement magic links, sessions, synthetic UUIDs, account/aviary creation, two stable starter birds, event log, minute tick, snapshots, event ordering, mood/weather and monotonic drift. Gate on replay/idempotency, presence conjunction, and two-device canonical-state checks.
3. **Scene and session interactions**: implement boot snapshot rendering, responsive 2D scene, return greeting, listen-in, offer panel/reactions/cooldowns, settle/undo, hidden-tab stop/resume, notebook generation/read path. Gate on “no announce” and “no in-scene chrome” review.
4. **Audio and access surfaces**: implement grammar synthesis, chorus and mix ramps, captions, narration, reduced motion, keyboard access, contrast, mute/fallback. Accessibility and audio acceptance happen before release, not in a later patch.
5. **Account/social operations**: implement visit issue/redeem/revoke/expiry/log, optional notification setting only after channel decision, settings, export, email change, session revocation, soft/hard deletion, matter-of-fact errors. Security review checks invite isolation and deletion coverage.
6. **Performance and limited launch**: synthetic runs and RUM verify all budgets at target devices/geographies. Begin with all new aviaries at two birds. Enable age-only later-bird opportunities gradually, first at the third-bird threshold, and validate recognizability/performance before each additional age cohort; cap remains seven. Ramp user traffic in cohorts behind rollback flags for scene/audio/social services. Use only aggregate health/performance signals to govern rollout, never per-account behavior or engagement scores.

## 11. Main risks and mitigations

- **Drift feels too fast, too slow, or guilt-inducing**: versioned bounded parameters, seeded week-three visual review, week-one numerical calibration, and explicit no-decrement absence invariant. Keep traits invisible outside export.
- **Presence is overstated or spoofed**: require visible + focused + recent pointer/key signal, short capped heartbeat credit, no backfill, interval union across devices, and lease expiry on hidden/closed sessions. Avoid collecting raw activity.
- **Concurrent devices lose state or double-apply events**: one server writer, sequenced append-only events, idempotency keys, per-aviary tick lease/CAS and atomic watermark commit; test overlapping device timelines and worker retries.
- **Ticks lag or go stale after outage**: elapsed-time recovery, sharded due queue, versioned snapshots, p99 alarm at five seconds, and load tests at seven birds/account scale. Keep tick compute bounded.
- **Calls sound uncanny or birds become indistinguishable**: constrain motif variation by species and stable bird seed; compare blind recognition across moods and two-to-seven-bird choruses; do not expand beyond seven without a separate v1 scope change.
- **Autoplay denial breaks the “already alive” feel**: never delay visual motion for audio; resume WebAudio on an allowed gesture and provide captions when audio cannot run. Validate current browser policies across supported versions.
- **Accessibility is treated as fallback work**: ship narration, reduced-motion, captioning, keyboard flow, and contrast gates with the initial release. Review prose and queue cadence with screen-reader users.
- **Visit feature leaks or affects host behavior**: invite-specific capability tokens, read-only endpoints, revocation checks on every snapshot, no visitor event writes, no discovery surfaces, and default-off notifications.
- **Privacy boundary erodes through logs/analytics**: enforce telemetry schemas that cannot accept bird/account event payloads; lint logs for email/token fields; verify export and 30-day hard deletion across backups and asynchronous jobs.
- **Sparse notebook becomes a feed or records user behavior**: derive only aviary facts, cap cadence, review every template, and block attendance/streak phrasing.
