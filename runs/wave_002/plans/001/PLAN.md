# Pocket Aviary v1 implementation plan

## 1. Product boundary and implementation posture

Pocket Aviary v1 is a web-only, single-user relationship with one canonical aviary. A new account receives two starter birds; the aviary can grow to seven birds on age-based availability. The product surface is one horizontal, glanceable scene with a sparse top bar, not a game shell. The implementation must preserve the product's central contract: the aviary continues without the viewer, notices the viewer through a bird's behavior rather than a system announcement, and accumulates quiet change over weeks.

Ship in v1:

- Email magic-link authentication, per-device sessions, account settings, email change verification, export, and soft-then-hard deletion.
- One canonical aviary per account, two starter birds, stable bird identity, renameable names, a small coherent species pool, and age-based additions through a hard cap of seven.
- Server-authoritative mood and personality drift; presence accounting; return greetings; bird-to-bird responses; listen-in; seed, song-fragment, and still-pool offers; settle; a sparse read-only field notebook.
- Local-time day/night, quiet weather events, continuous ambient motion, responsive one-screen rendering, procedural calls, chorus mixing, call captions, screen-reader narration, keyboard access, and reduced-motion rendering.
- Per-invite, revocable, read-only visits, disabled by default, with a silent visit log and an optional host notification setting that is off by default.

Do not add native clients, shared/co-present aviaries, multi-aviary accounts, customization/catalog selection, hunger or health mechanics, death or distress, scores, levels, XP, achievements, badges, streaks, visit counters, public profiles, discovery, follows, chat, comments, leaderboards, push notifications, or marketing/show-off rendering. Do not expose personality numbers, even in settings or an export UI; the account export is a data portability artifact, not a product stats surface. Visitor attention must never affect host presence, mood, or drift.

Where the PRD leaves a value open, use a configuration/feature flag with an explicit calibration owner rather than hard-coding a product promise. In particular, isolate the presence inactivity window, tick cadence, mood transition constants, age-based bird schedule, offer cooldown, audio ramp times, and weather frequency so they can be calibrated without schema changes. The defaults must still be deterministic and testable.

## 2. Architecture

Use a small modular backend with a clear single-writer simulation boundary rather than a distributed set of independently authoritative services. The logical modules are:

1. **Web shell and renderer.** Serves the minimal HTML, initial CSS, renderer code, and a small bootstrap snapshot. It owns interpolation, local-time presentation, visual micro-motion, input handling, WebAudio, captions, and accessibility presentation. It never owns canonical bird personality or mood.
2. **Account/auth module.** Creates synthetic account UUIDs, sends and consumes magic links, issues/revokes per-device session tokens, handles email-change verification, export requests, and deletion lifecycle.
3. **Aviary API module.** Authenticated snapshot reads, append-only interaction-event ingestion, notebook reads, bird rename/adoption actions, and settings. It validates event shape and authorization but does not apply personality deltas.
4. **Simulation worker.** The only writer of canonical personality vectors, moods, bird positions/pose state, call scheduling state, weather state, and derived notebook observations. A scheduler enqueues due aviaries; a per-aviary lock/lease makes one tick authoritative.
5. **Visit module.** Creates one-time invitation links, resolves them to a read-only visit session, serves the same canonical snapshot, records only the host's requested visit-log facts, and enforces immediate revocation/expiry. Visit sessions cannot submit interaction or presence events.
6. **Operational telemetry pipeline.** Receives aggregate request, latency, render, audio-error, and tick-health measurements only. It has no read path to per-account simulation records and no per-bird/per-account dimensions.

The source of truth is a transactional relational store. Keep the event log and current state in the same consistency domain so a tick can claim events, apply a versioned update, advance its cursor, and publish a new snapshot version atomically. Use an object store or signed export worker only for on-demand account-export artifacts; do not put persistent simulation state in client storage. A cache/CDN may accelerate immutable web assets and a short-lived bootstrap response, but it cannot become a second state authority.

The client/server boundary is semantic and explicit:

- Server sends canonical state: bird stable IDs, names, species, normalized visual/mood inputs, perch zone, mood, current weather/day phase, call timing/seed metadata, active transition metadata, notebook cursor, snapshot version, and server time.
- Client derives presentation: interpolation between snapshots, local-time palette interpolation, idle pose phase, leaf/feather ornaments, canvas/SVG composition, audio nodes, caption placement, focus treatment, and reduced-motion presentation.
- Client submits facts about user actions and presence evidence; it never submits an absolute personality vector, mood, position, or drift result.
- Server owns timestamps, event sequence/order, authorization, all drift/mood updates, and the canonical snapshot version. A client timestamp is advisory and bounded by server receipt time.

The initial state request should be edge-friendly: authenticated bootstrap data is small and can be delivered with the HTML path where practical. If it is not ready, draw a quiet sky/field with faint ambient cues, never a spinner or entry animation. The first usable state should place birds at a non-zero pose phase so the first rendered frame looks mid-action.

## 3. Persistent data model

Use opaque UUIDs for all internal references. Email is encrypted on the account/invitation records and is never an identifier, partition key, log field, event payload key, or analytics dimension.

### Account and session records

- `accounts`: `account_id`, encrypted verified email, pending email and verification metadata, created/updated timestamps, deletion-requested-at, deletion-hard-at, settings, and a monotonically increasing state version.
- `device_sessions`: `session_id`, `account_id`, hashed revocable token, device label/created/last-seen timestamps, expiry/revocation timestamps, and auth audit metadata. Never persist raw tokens.
- `magic_links`: hashed one-time token, account reference, requested email, expiry (15 minutes), consumed-at, request-rate metadata, and creation time. Consumption is transactional and immediately invalidates the token.
- `account_settings`: accessibility settings (reduced motion override, captions), audio preference, optional visit-notification toggle, and other matter-of-fact system settings. Do not place engagement metrics here.

### Aviary and birds

- `aviaries`: one row per account with `aviary_id`, `account_id`, local-time-zone identifier, created-at, current tick cursor, next tick time, snapshot version, current day/night phase, weather state, and current canonical scene state/version.
- `birds`: `bird_id`, `aviary_id`, stable species ID, user-facing name, adoption timestamp, adoption sequence, hidden normalized personality vector, mood enum, mood-entered-at/expiry, perch zone, call signature/motif seed, and version timestamps. The stable ID is never regenerated by rename, sync, species-pool change, or migration.
- `bird_personality` can be a separate protected table if access control is clearer: boldness, social warmth, vocal frequency, plumage saturation, curiosity, plus schema/calibration version and last server-authored delta metadata. Keep it out of normal client DTOs. Values are clamped to a documented normalized range; no client write path exists.
- `aviary_snapshot_versions` or an equivalent immutable snapshot/audit record stores the last published canonical version and tick metadata needed for replay/debugging without making the client state authoritative. Retain only what privacy/deletion policy allows.

### Events and derived prose

- `interaction_events`: append-only `event_id`, `aviary_id`, `account_id`, authenticated session ID, server-received time, client-time bounded by server time, monotonic per-aviary sequence, event type, bird ID where applicable, sanitized payload, idempotency key, and consumed-at/tick version. Types include presence start/heartbeat/end, listen-in start/end, offer, settle, settle-undo, and re-engagement. Visitor sessions cannot insert rows.
- Presence payloads record the client's visibility/focus/activity evidence and interval, but the server accepts only intervals bounded by authenticated heartbeat cadence and rejects background-only keepalives. A presence interval ends on hidden/unfocused state, explicit settle, disconnect timeout, or inactivity threshold. The configured inactivity threshold is calibrated, with the initial implementation leaning long enough to permit quiet watching.
- `notebook_entries`: `entry_id`, `aviary_id`, observation timestamp/interval, generator rule/version, participating stable bird IDs, naturalist prose, and created-at. Entries are immutable, read-only, sparse, and cursor-paginated indefinitely. Do not include raw interaction counts or personality numbers in prose.
- `visit_invitations`: invitation ID, host/aviary ID, encrypted invitee email, hash of one-time token, created/used/expiry/revoked timestamps, and permission `read_only`. Unused invitations expire after 30 days and cannot be revived.
- `visit_sessions`/`visit_log`: opaque visit session, invitation reference, started/ended/last-pull, approximate duration, and revocation state. Store enough for the host's on-demand log; do not treat visitor viewing as host presence. Visitor email is displayed only in the authorized host settings surface.

Protect personality, mood, event history, and notebook data with account-scoped authorization and audit access. Account hard deletion must cascade or tombstone-and-purge every account-tied record, including event history, vectors, notebook, sessions, invitations, visit logs, exports, and telemetry references. Operational aggregate metrics must be account-free and must not be reconstructible into per-bird histories.

## 4. API surface and protocol rules

Use versioned JSON endpoints with request IDs and an idempotency key for every mutating request. Return a snapshot version and server time on all state responses. System errors use direct, matter-of-fact copy; product observations remain naturalist and lowercase.

Core endpoints:

| Endpoint | Behavior |
| --- | --- |
| `POST /v1/auth/magic-link` | Request a link, rate-limited by normalized email/IP/device without exposing account existence. |
| `POST /v1/auth/magic-link/consume` | Atomically consume a 15-minute token, create/reuse account, issue one per-device session token, and create the two starter birds if this is a new account. |
| `GET /v1/aviary/bootstrap` | Return the current canonical snapshot, server time, accessibility/audio settings, capabilities, and a snapshot version. |
| `GET /v1/aviary/snapshot?since=` | Return a full or delta snapshot. Full snapshot is required after hidden-tab return, suspension/frame-gap detection, or version mismatch. |
| `POST /v1/aviary/events` | Append one or a bounded batch of interaction/presence events after validating session, bird ownership, cooldown, timestamp bounds, and idempotency. Return accepted sequence IDs, not derived personality. |
| `GET /v1/notebook?cursor=` | Read immutable sparse entries in reverse chronological order; never expose event-log internals. |
| `POST /v1/birds/:bird_id/rename` | Rename a stable bird after matter-of-fact validation; no personality/mood change. |
| `POST /v1/aviary/offers` | Convenience endpoint or event wrapper for seed/song-fragment/still-pool offers; server assigns the receiving bird and enforces per-bird cooldown if the UI has not already selected one. |
| `POST /v1/aviary/settle` and `POST /v1/aviary/reengage` | Append settle/re-engagement events; the tick owns resulting canonical mood/scene state. |
| `GET/PATCH /v1/account/settings` | Read/update system settings, including accessibility and optional visit-notification toggle; no streak or visit-frequency metric. |
| `POST /v1/account/export` | Queue a JSON snapshot export and email a signed, expiring download link to the verified address. Export generation reads canonical state under account authorization. |
| `POST /v1/account/delete` / `POST /v1/account/restore` | Start 30-day soft deletion or restore within the window; a scheduled purge hard-deletes after the deadline. |
| `POST /v1/visits/invitations` | Create a named, one-time, 30-day read-only invite and email its link; visits remain off unless this explicit action occurs. |
| `GET /v1/visits/:token` | Resolve a valid invite into a read-only visit session and return the host snapshot. Never issue host mutation capability. |
| `GET /v1/visits/:session/snapshot` | Pull the host's current snapshot. Revocation is observed on the next pull and returns a matter-of-fact unavailable surface. |
| `GET /v1/account/visits` / `POST /v1/account/visits/:id/revoke` | Show the authorized host's visit log/outstanding invites or revoke immediately. |

Polling is sufficient and preserves the canonical model: visible clients pull after bootstrap at a low cadence, on visibility return, and after long frame gaps. A short keepalive may reduce perceived latency but must not be interpreted as presence unless the client has all three signals: visible document, focused window, and recent pointer/key activity. Do not introduce WebSockets merely to broadcast client-owned state; if later used for freshness, it is only a snapshot transport.

For concurrency, every event batch carries the last observed snapshot version for diagnostics, but stale versions are not rejected as conflicts because events are facts that can be appended. The server assigns the ordering sequence. Mutations that should happen once (rename, adoption, invitation revoke) require idempotency and transactional compare-and-set. Personality is never resolved by last-write-wins.

## 5. Server-side simulation engine

### Tick lifecycle

Run a scheduler about once per minute per aviary, with a per-aviary lease and retry-safe transaction:

1. Claim an aviary whose `next_tick_at` is due. If another worker holds the lease, skip; never run two ticks concurrently for one aviary.
2. Read the last consumed event sequence, all newly appended events, elapsed server time, local-time phase, weather state, current birds, and calibration version.
3. Normalize/clamp event durations and deduplicate idempotency keys. Convert presence intervals into presence-time only when the visible/focused/recent-activity conjunction held. Ignore visit-session traffic.
4. Apply interaction effects to short-lived mood intents and bounded drift accumulators. Apply local-time and ambient-weather effects. Resolve bird-to-bird responses (calls can prompt calls; alarm/wary signals can spread) from the pre-tick state, so iteration order cannot create accidental bias.
5. Apply monotonic personality deltas server-side, clamp to range, persist the new vector, transition mood and mood timers, choose perch/pose/call schedule, advance weather, and compute any eligible notebook observation.
6. Publish a new snapshot version and consumed-event cursor atomically. Schedule the next tick. Emit only aggregate tick duration/error metrics.

The tick must be deterministic for a fixed prior state, ordered event set, elapsed time, calibration version, and RNG seed. Use a stable per-aviary/per-bird seed; random variation is reproducible for tests and debugging but not visible as a number to the user. If a tick retries, it must produce the same result or be rejected by version/lease checks rather than double-applying deltas.

### Drift model

Represent the low-pass filter explicitly, for example as a bounded, server-authored accumulator per trait. Presence is the dominant positive input; listen-in contributes primarily to social warmth and vocal frequency; offers contribute a small curiosity signal on acceptance and a small boldness signal when made near a bird; settle ends the presence window and quiets mood but has no positive/negative personality direction. The exact weights and learning rate are configuration values with a calibration version.

All personality deltas are non-negative in v1. Neglect produces no negative drift and no distress; at most it yields a quieter/less often observed presentation through current mood, time of day, or recent history. Clamp to the expressive ceiling. Do not derive the vector from the event log at read time, do not let a client submit absolute traits, and do not erase deltas when a device is stale.

Build a time-accelerated simulation harness that can run a typical regular-visit scenario and verify measurable instrument drift after approximately one week and user-visible-but-not-session-to-session change after approximately three weeks. Add scenarios for two weeks away, concurrent laptop/phone sessions, duplicate event delivery, hidden-tab presence, and seven-bird chorus. Review calibration with product/design using observed behavioral traces, not a visible stats surface.

### Mood and scene state

Use a small enum such as `wary`, `content`, `curious`, `drowsy`, and `alert`; keep the exact set in one versioned engine module. Mood transitions combine recent event impulses, local-time phase, weather, bird personality, and neighboring-bird signals. Mood persists across sessions and is advanced by ticks while no client is connected. Time-of-day changes should make a bird more likely to be drowsy near dusk/night and alert in early morning, not snap every bird to a default at login.

Map mood and personality to behavior, not numeric UI: perch-zone probabilities, approach/retreat timing, preen/scan/head-tilt pose families, call frequency, response latency, and offer reactions. Keep behavior distributions bounded so a high vocal-frequency aviary can still be quiet in rain or at night. At night most birds settle; the nightjar-like species may remain active.

### Calls, greetings, and notebook generation

Each bird has a stable call-signature seed and species motif library. The engine emits call intents with motif, pitch/rhythm parameters, mood modifier, intensity, and caption token data; it never stores or downloads a loop. Signature parameters stay recognizable as mood/personality changes. Chorus response is scheduled from bird-to-bird rules, with bounded polyphony and randomized, deterministic offsets.

On return, use absence duration, boldness, mood, and stable per-session seed to select one likely first greeter and a varied gesture/call. Stagger multiple likely responses by small randomized offsets; never run a synchronized arrival fanfare. The absence duration is a behavior input, not a textual “you were gone” announcement.

Generate notebook observations only when a sparse rule passes a per-aviary cooldown/novelty check (roughly every few days for regular users, with noteworthy moments allowed). Render lowercase, present-tense, specific naturalist prose from the same structured observation facts used for narration. Never generate event-log language, interaction counts, drift numbers, streaks, or user-behavior summaries. Store the generator version and source bird IDs for auditability while exposing only prose and date to the normal notebook reader.

## 6. Client rendering pipeline

### Bootstrap and scene composition

Use a lightweight scene renderer (Canvas 2D or a similarly small retained layer) behind a semantic DOM interaction layer. Keep the renderer API independent of transport: `SnapshotInterpolator` supplies pose/mood/scene values, while `SceneRenderer` draws background, subtle parallax, three perch zones, birds, and restrained foreground elements. Top-bar controls, captions, focus proxies, narration, and settings remain normal DOM so they are keyboard and screen-reader addressable. The scene contains no inline buttons, badges, tooltips, or labels.

The renderer should:

- Draw immediately from bootstrap state with a non-zero pose/call phase and no fade-in, spinner, or “ready” transition.
- Interpolate between canonical snapshots; use server time to compensate for client clock differences and avoid teleports after normal polling.
- Keep all birds visible across responsive aspect ratios. Compress spacing on narrow screens and widen it on desktop; never pan, zoom, or crop a bird.
- Derive local-time palette and light from the account time zone, with a gentle sunrise/day/evening/night curve. Weather affects scene ornaments and mood through canonical state, while leaves/feathers are client-only ornaments not included in simulation state.
- Keep top-bar fade separate from the scene: fade toward transparent after cursor stillness, restore on pointer/keyboard activity, and preserve keyboard focus visibility. Do not use top-bar fade to hide focused controls.
- Stop expensive rendering when the document is hidden, keep audio/canonical simulation semantics separate, and fetch a fresh snapshot on visibility return or a long frame gap.

The UI should not expose mood labels or personality values as normal controls. Interaction is through the top-bar offer affordance, notebook, account/accessibility settings, settle, and bird focus/listen-in targets. Bird targets may be transparent semantic focus proxies aligned with the rendered bird; their focus ring is drawn as a soft high-contrast outline so the scene stays free of chrome.

### Interaction transitions

Listen-in, settle, offer reactions, and perch changes must use motion curves and durations tuned to read as attention rather than UI state switching. Listen-in gain and visual focus ramp gradually; settle warms/dims the scene over several seconds and quiets calls; an aviary click within five seconds reverses settle. Accidental clicks do not create extra system announcements. The server remains authoritative for durable effects while the client may optimistically show a reversible transition, then reconcile from the next snapshot.

Reduced-motion is a first-class renderer mode. Replace frame-by-frame micro-motion and flight paths with slow cross-fades between still poses; remove leaf drift; retain slowed day/night color changes, canonical moods, calls, captions, and notebook behavior. Honor `prefers-reduced-motion` before first render and allow a matter-of-fact settings override. Test transitions at setting changes so a user is never caught in a high-motion animation.

## 7. Audio pipeline

Create one bounded WebAudio context per visible client, initialized/resumed from an allowed user gesture where autoplay policy requires it. Load motif metadata, not recorded calls. A call synthesizer maps a bird's stable signature, species motif, mood, and server-emitted variation into short oscillator/noise/envelope nodes or reusable worklet buffers. Pool/reuse nodes and release scheduled resources so a 30-minute session does not grow memory.

Route each bird through a per-bird gain bus into an ambient/chorus master. Normal playback leaves all active birds audible. The chorus scheduler allows overlapping calls with small timing and pan/level differences and enforces a bounded polyphony/CPU budget. Never layer recorded loops and never make a focused bird's neighbors silent.

On listen-in, ramp the focused bird's gain up and other birds' gains down toward ambient over a slow configurable interval; on disengage, reverse the same ramp. Clicking the focused bird, focusing another bird, clicking empty scene space, or moving keyboard focus away ends listen-in. The server records the interaction intent/duration; the client owns only the ephemeral mix curve.

Caption generation must use the same call intent/motif parameters that reach the synthesizer, producing short naturalist descriptions such as “a soft three-note rise.” Place captions near the calling bird as DOM text, animate/fade them accessibly, and ensure they remain readable against every day/night state.

If WebAudio is unavailable, blocked, or errors, fail into graceful silence with captions enabled by default and a clear matter-of-fact audio setting/error affordance. Do not ship a recorded-audio fallback. Mute/permission states must not stop simulation, visual behavior, captions, or screen-reader narration.

## 8. Accessibility surfaces

Treat accessibility as a parallel product surface that reads the same canonical state, not as labels attached after the visual build.

- **Screen reader:** maintain a polite live narration region with naturalist prose generated from the current semantic snapshot. At idle, update roughly every 30–60 seconds and only when the observation materially changes; prioritize a return greeting, accepted offer, and settle without flooding the queue. Use lowercase, present tense, specific observations. Do not announce raw coordinates, mood enums, personality numbers, event IDs, or every interpolation frame.
- **Captions:** offer a setting to enable captions, with captions on by default when audio is unavailable. Descriptions correspond to the actual procedural call and its current bird/mood context, not a fixed per-bird string.
- **Keyboard:** tab through the sparse top bar; enter the scene's first bird; arrow keys move between birds; Enter starts/ends listen-in; Escape exits it. Make offers and settle fully reachable with visible focus. Keep the reading order stable while birds move visually.
- **Focus and hit targets:** use semantic controls with accessible names derived from bird names/species without exposing internal stats. Draw a high-contrast focus treatment against bright and dim scenes, and do not rely on hover or audio for state comprehension.
- **Visual system:** all user-copy, top-bar labels, settings, account/error text, narration when displayed, and captions meet WCAG AA contrast. System/auth/sync/accessibility-settings errors use direct matter-of-fact copy; the aviary and notebook use the naturalist voice.
- **Reduced motion/audio off:** preserve mood, calls-as-captions, day/night, offers, settle, and notebook behavior when motion is reduced or audio is muted. Verify the result is still recognizable as the same aviary.

Run automated accessibility checks plus manual VoiceOver/NVDA/keyboard passes on fresh, returning, settled, nighttime, rain, reduced-motion, audio-denied, visitor, and error states. Include screen-reader queue length and caption timing in acceptance tests.

## 9. Performance and observability

Treat the following as release gates on supported last-two-major Chrome, Safari, Firefox, and Edge versions:

- Initial JavaScript bundle under 2 MB gzipped at first paint. Code-split settings, notebook history, accessibility settings, and visit invitation flow; keep the initial scene, snapshot adapter, and essential renderer in the critical path.
- First bird visible under 500 ms on a mid-tier mobile device over 4G, with initial state delivered as a small bootstrap payload or fetched without waiting on non-critical assets.
- 60 fps idle motion on a five-year-old mid-range laptop for a 30-minute session; measure frame-time percentiles under two and seven birds, weather, captions, and listen-in.
- No client memory growth over 30 minutes. Test pooled audio nodes, timers/listeners, snapshot interpolation buffers, caption cleanup, notebook scroll/unmount, worker lifetimes, and hidden/visible transitions.
- Simulation tick p99 under 5 seconds, with alerting at the stated threshold. Snapshot payloads remain kilobytes-scale and polling backoff must not create a thundering herd.

Collect aggregate-only metrics: request/error/latency counts, tick duration and failures, snapshot size/version lag, first-bird timing, render-frame timing, audio-context error counts, browser capability rates, and anonymized session-duration histograms. Synthetic browsers should run from common geographies. Do not attach account ID, bird ID, email, interaction payload, per-bird mood, personality, notebook text, or visit identity to telemetry. Keep the simulation database out of analytics read paths. Add privacy-boundary tests that fail CI if forbidden dimensions enter metric schemas.

## 10. Delivery sequence and verification

1. **Foundations and contracts:** define versioned domain types, account/session authorization, migrations, event schemas, snapshot schema, calibration configuration, and privacy/retention policy. Add contract tests before UI work.
2. **Canonical engine:** implement deterministic tick, event cursor/leases, monotonic drift, mood transitions, local-time/weather effects, stable IDs, call intents, return greeting, and sparse notebook rules. Build the accelerated-time calibration harness and property tests for no negative drift, idempotent retries, and no visitor influence.
3. **Auth and state API:** implement magic-link lifecycle, device revocation, bootstrap/delta snapshots, event ingestion, rename/adoption, settings, export, deletion, and matter-of-fact error responses. Exercise two simultaneous devices against one account.
4. **Renderer and interaction layer:** implement immediate scene bootstrap, three zones, responsive layout, interpolation, local day/night, weather, ambient ornaments, top-bar fade, bird focus proxies, offers, settle/undo, and notebook shell. Verify no scene chrome or loading spinner leaks into the experience.
5. **Audio and accessibility:** add procedural synth, chorus buses, gradual listen-in mix, captions, WebAudio failure mode, semantic narration, keyboard movement, visible focus, and reduced-motion renderer. Test audio-off and reduced-motion as primary paths.
6. **Visits and account surfaces:** add named invitation creation, one-time resolution, read-only snapshot pulls, revocation/expiry, silent host log, optional notification setting, export/download, and deletion/recovery. Verify visitors cannot append events or receive mutation tokens.
7. **Performance/security hardening:** run bundle, first-bird, frame-time, memory, tick-load, browser-matrix, accessibility, authorization, rate-limit, token, deletion, and privacy-boundary suites. Load-test the scheduler with due-aviary contention and retry/failure injection.
8. **Canary and ramp:** release behind flags to internal synthetic accounts, then a small real-account canary. Start all accounts at two birds. Enable age-based third-bird offers only after engine/audio recognizability and stability gates; ramp later birds conservatively by aviary age toward the seven-bird cap. Watch aggregate performance and error metrics, not per-user behavioral optimization metrics.

Definition of done includes reproducible engine tests, API contract documentation, migration rollback/forward strategy, accessibility sign-off on all primary states, performance budgets met on representative devices, privacy review of schemas and telemetry, and a launch checklist that explicitly confirms no gamification, notification, public-discovery, co-presence, or client-authored personality path exists.

## 11. Risks and mitigations

| Risk | Failure mode | Mitigation and release signal |
| --- | --- | --- |
| Drift calibrates too quickly | Birds visibly change session-to-session or absence is accidentally punitive | Accelerated time simulations, non-negative-delta property tests, feature-flagged learning rate, one-week instrument/three-week perceptual targets, two-week absence scenario. |
| Drift calibrates too slowly | Presence feels irrelevant after weeks | Instrument-only probes and longitudinal canary review; tune weights without exposing stats or adding engagement counters. |
| Presence is overcounted | Background tabs inflate drift and corrupt the relationship | Require visible + focused + recent activity, terminate on hidden/unfocused/timeout, reject visitor pings, cap intervals, test browser visibility/focus permutations. |
| Concurrent devices lose history | Stale phone overwrites laptop drift | Append-only ordered events, server-only additive deltas, per-aviary tick lease, atomic cursor/version transaction, no last-write-wins personality write. |
| Tick retry double-applies changes | Personality/mood jumps after worker failure | Idempotency keys, consumed cursor in same transaction, deterministic seed, lease fencing, fault-injection tests. |
| Audio sounds canned or uncanny | Repeated motifs break aliveness; seven-bird chorus muddies | Stable recognizable signatures plus bounded variation, motif listening tests, procedural-only policy, polyphony limits, listen-in gain curves, staged bird-count ramp. |
| Autoplay/WebAudio failures | User hears silence without understanding, or fallback becomes recorded loops | Treat silence/captions as designed fallback, resume on gesture, caption by default when unavailable, monitor aggregate audio errors. |
| Accessibility becomes a degraded mode | Screen-reader narration is a state dump; reduced motion looks broken; focus disappears | Shared prose generator, manual assistive-tech review, designed cross-fade mode, semantic focus proxies, caption/narration queue tests, WCAG AA gates. |
| Renderer performance regresses | First bird misses 500 ms or idle motion drops below 60 fps | Critical-path budget, asset/code splitting, device lab, frame/memory soak tests, hide-tab pause, no per-call allocation. |
| Visitor privacy expands accidentally | Visitor changes host state or host gets unsolicited social pressure | Read-only scoped token, no event/presence writes, immediate revocation on pull, visits off by default, no public discovery, notification toggle off by default. |
| Product voice leaks into system errors or gamification | Toasts, streaks, badges, or whimsical auth errors reframe the product | Copy review fixtures and route-level lint/checklist; explicitly test return, sync, settings, visit, and error surfaces for the allowed voice and prohibited concepts. |
| Deletion/privacy boundary is incomplete | Per-bird history survives hard deletion or enters analytics | Account-scoped purge job with verification, export/deletion integration tests, telemetry schema denylist, no warehouse access to simulation tables. |

The launch decision is based on the above behavioral and operational gates, not on adding more surface area. Stop before evaluation once the two requested phase-1 artifacts are written; product implementation belongs to the engineering phase that consumes this plan.
