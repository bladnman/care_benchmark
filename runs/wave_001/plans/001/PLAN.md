# Pocket Aviary v1 implementation plan

## 1. Delivery boundary and product invariants

Build Pocket Aviary as a modern-browser web application with one canonical, server-simulated aviary per signed-in account. The first release delivers two starter birds (and age-gated growth to a hard maximum of seven), email magic-link accounts, multi-device state convergence, the interactive aviary, the field notebook, optional quiet read-only visits, accessibility surfaces, and account/privacy controls. It is a relationship with a continuing place, not a progression system.

The following invariants are architectural acceptance criteria, not copy guidance:

- The server is the only writer of persistent bird personality, mood, and canonical scene state. Clients submit facts/events and render snapshots; they never calculate or upload an absolute personality value.
- Qualifying host presence requires visible document **and** focused window **and** pointer/key activity within the calibrated grace window. A visible open tab alone never creates presence time. Visitor activity never creates host presence.
- Personality changes only slowly and only toward expressive values. Absence cannot decrease personality traits or create sickness/distress. Mood may change normally while the aviary is unattended.
- A bird has a durable opaque ID independent of name/species. Renaming, login from another device, and later content changes must not replace that identity or its vector history.
- Calls are generated procedurally in the browser; recorded-call fallbacks and loop libraries are prohibited. The same grammar instance also supplies the call caption.
- The product view is naturalist, lowercase, specific, and observational. Account, auth, sync-error, and accessibility-settings UI is direct matter-of-fact language. There are no welcome toasts, streaks, badges, points, or engagement notifications.
- Per-account interaction data remains inside the simulation domain. Operational telemetry is aggregate only and cannot contain bird names, state, events, account IDs, raw email, or reconstructable relationship history.

### Explicit v1 scope

In scope: web-only responsive aviary; local-time day/night; restrained ambient weather; greeting, listen-in, offer, settle/undo; field notebook; keyboard, screen-reader, reduced-motion, and call-caption experiences; export, soft/hard deletion, revocable device sessions; email-specific visit invites and host visit log; browser support for the last two major Chrome/Safari/Firefox/Edge releases.

Out of scope: native clients, payments, multiple/shared aviaries, scene customization or user bird placement, public profiles/discovery/follows/comments/chat, co-presence, show-off rendering, leaderboards, achievement/streak/visit-frequency surfaces, push/email product notifications, hunger/death/decaying happiness, and numeric personality UI. Do not create latent analytics dimensions or UI models that make these features easy to add.

## 2. Service architecture and responsibilities

Use a TypeScript web client backed by a small modular service deployment. Keep the public API stateless; put all durable simulation work behind transactional storage and a scheduled worker.

| Component | Responsibilities | Cannot do |
| --- | --- | --- |
| Edge/document delivery | Serve a minimal app shell and an initial, authenticated snapshot envelope from a nearby edge; cache static assets. | Persist simulation state or expose a cache containing another account's state. |
| Identity/account service | Magic links, verified email encryption, account lifecycle, device sessions, settings, export/deletion workflows. | Use email as a foreign key, log identifier, telemetry dimension, or queue partition. |
| Aviary read API | Authorize host/visitor capability, return compact canonical snapshot plus ETag/version and delivery-time scene context. | Accept state replacement or calculate personality in the request path. |
| Interaction ingest API | Validate, deduplicate, sequence, and append host interaction/presence facts. | Mutate birds, generate drift, or accept visitor interaction facts. |
| Simulation worker | Claim due aviaries, consume ordered events, advance tick/mood/state/notebook eligibility atomically. | Call client devices or write aggregate analytics from per-bird data. |
| Visit service | Issue encrypted/email-bound single-use invitation capabilities; revoke and validate access; record a private host visit log. | Create co-presence, host notifications by default, or simulation events from a visitor. |
| Client renderer/audio runtime | Render/interpolate a snapshot, create procedural calls, present controls and accessible equivalents. | Become a second source of truth or keep mutation queues across logins. |

Use a relational primary store with row-level transactional guarantees, an append-only event table, and a separate scheduled-work queue keyed only by synthetic `account_id`/`aviary_id`. Encrypt email and invite recipient email at rest with envelope encryption. Put a short-lived job lease around each aviary tick so overlapping workers cannot process the same event range. The worker update and `last_processed_event_seq` advance occur in one database transaction. A durable outbox is used only for permitted account mail and export/deletion jobs; no simulation data is copied to analytics.

### Client/server boundary

The client owns ephemeral presentation state: current render interpolation, audio nodes, top-bar visibility, local focus, whether a settle undo is available, and an idempotent submission queue. The server owns all account, invitation, canonical aviary, bird, mood, interaction-history, notebook, and visit-log records. A client may optimistically show the harmless visual consequence of an accepted interaction, but it must reconcile to the next canonical snapshot and never infer a trait delta locally.

Rendering uses a retained scene graph on `<canvas>`/WebGL where supported, with DOM only for the sparse top bar and accessible semantic surfaces. Feature-detect WebGL and retain a performant Canvas 2D path; neither changes the authoritative data model. The scene renderer receives a normalized snapshot, not database-shaped records.

## 3. Canonical data model

All primary identifiers are UUIDs/opaque random IDs. `account_id` is generated at account creation and is the only account reference outside the encrypted account table.

| Record | Key fields and rules |
| --- | --- |
| `accounts` | `account_id`, encrypted verified email, email-verification state, IANA `timezone`, lifecycle (`active`, `pending_deletion`, `deleted`), created/deletion dates. No email-derived IDs. |
| `device_sessions` | Session ID, account ID, issued/last-used/revoked timestamps, coarse device label. Session tokens are hashed at rest and revocable. |
| `aviaries` | `aviary_id`, account ID (unique), `state_version`, `next_tick_at`, `last_tick_at`, local-time configuration, current day/night/weather/settled state, seed/version for deterministic scene choices. |
| `birds` | Stable `bird_id`, aviary ID, species ID, current user name, creation/adoption timestamp, age-gate state, front/middle/back perch intent, persistent personality vector, current mood and timer, current visual/call signature seed. Traits are bounded normalized scalars and never serialized to normal product UI. |
| `bird_trait_history` | Server-only sampled calibration/audit history: bird ID, tick, deltas and algorithm version. Retention is inside the account domain and included in hard deletion; it is not an analytics source. |
| `interaction_events` | Append-only `event_seq`, aviary/account ID, bird ID where applicable, session ID, client idempotency key, server received time, client observed time, type/payload/schema, processed tick. Types include qualifying presence interval, listen-in start/end, offer request/result, settle, and session/visibility context. |
| `presence_windows` | Server-accepted intervals derived from client heartbeats, start/end, qualifying evidence timestamps, bounded duration, and event sequence. Store only what simulation needs; never expose a visit-frequency history to the host UI. |
| `scene_state` | Compact canonical positions, current action/transition, call scheduling seed/window, weather state, greeting eligibility/context. It is derived and replaceable from the current canonical state, unlike personality. |
| `notebook_entries` | Immutable entry ID, aviary ID, occurred range, naturalist prose, source event IDs, generation/rules version. Read-only, append-only, indefinitely scrollable until account deletion. |
| `settings` | Accessibility/reduced-motion/captions, account timezone, visit-notification opt-in (false by default), and non-product preferences. Do not store a gamification preference. |
| `visit_invites` | Invite ID, host account/aviary ID, encrypted recipient email, capability hash, created/used/expires/revoked timestamps, optional active session linkage. Each is a deliberate invite and expires unused after 30 days. |
| `visit_access_sessions` / `visit_log` | Read-only capability session and host-visible records of recipient email, time, and approximate duration. These never feed presence or the simulation. |
| `export_jobs` / `deletion_jobs` | Account-owned on-demand exports and 30-day soft-delete/hard-delete execution records. Export links are short-lived and emailed only to the verified address. |

Use a `state_version` monotonically incremented in the same transaction as each canonical update. Snapshots contain the version, server time, next relevant transition/tick, and a schema version. They may contain user-facing render parameters, not hidden numeric vectors. Account export is the narrowly authorized exception: it includes current vectors as specified, delivered to the verified owner only.

## 4. Public API and event contracts

Version the API (`/v1`) and JSON schemas from day one. All mutations require a host session plus a UUID idempotency key; retries return the original accepted event and do not duplicate presence, offers, or visit invitations. Validate payload size, schema version, membership, cooldown, and session revocation before append.

| Endpoint | Contract |
| --- | --- |
| `POST /auth/magic-links` | Request link for an email under per-email and IP abuse limits; always use non-enumerating response language. Link expires in 15 minutes. |
| `GET/POST /auth/magic-links/consume` | Atomically consume one unused link, issue a device session, and invalidate the link. |
| `GET /aviary/snapshot` | Host-authorized compact snapshot. Supports `If-None-Match`, returns `state_version`, server time, rendering/call schedule, active transition, and accessibility narration source data. |
| `POST /aviary/events` | Append validated host facts: qualifying presence heartbeat/close, listen-in begin/end, offer, settle, re-engage, and visibility return. Returns accepted sequence and cooldown/rejection facts, never trait values. |
| `GET /aviary/notebook?cursor=` | Paginated immutable entries in reverse chronological order, with no edit/delete endpoints. |
| `GET/PATCH /account/settings`, `GET /account/sessions`, `DELETE /account/sessions/:id` | Matter-of-fact account/accessibility settings and session revocation. |
| `POST /account/export`, `POST /account/delete`, `POST /account/delete/restore` | Start owner-authorized export, enter soft deletion, or restore within 30 days. |
| `POST /visits/invites`, `GET /visits/invites`, `DELETE /visits/invites/:id` | Create email-specific invite, list outstanding/active invites, and immediately revoke. Creation is never shown during onboarding. |
| `GET /visits/:capability/snapshot` | Validate non-revoked, unexpired visitor capability on every pull, return the same render snapshot in read-only mode, or a matter-of-fact unavailable surface. |
| `POST /visits/:capability/open` and `POST /visits/:capability/close` | Record only host visit-log facts and approximate duration; do not expose an interaction-event route to visitors. |

Snapshot delivery is pull-first: initial navigation, `visibilitychange` back to visible, recovery from a long render gap, and a low-frequency visible-tab keepalive. Optionally use a lightweight server-sent invalidation hint to prompt a pull, but never make persistent client state depend on a stream. The client sends `last_seen_state_version` so the API can return `304` or a compact delta envelope where safe; a full snapshot is always valid recovery.

## 5. Simulation engine

### Tick execution and determinism

Run a due-aviary tick about once per minute, with jittered scheduling to avoid a global thundering herd. The worker locks the aviary, reads unprocessed events ordered by server-assigned `event_seq`, resolves the IANA local time, applies bounded transitions, writes state and notebook candidates, marks the consumed range, increments `state_version`, and commits. Retry on serialization conflict; never skip or reorder an event. A missed scheduled tick is caught up with bounded elapsed-time segments, not one oversized jump, so time-of-day/mood transitions remain plausible.

Make scene randomness reproducible for a `(aviary_id, tick_number, simulation_algorithm_version)` seed. This permits replay of a reported state within the account domain without treating event history as a substitute for persisted personality. Use secure randomness only for identifiers/capabilities; use deterministic pseudorandom choices for simulation variation.

### Presence and drift

The client listens for focus, blur, visibility, pointer movement, and keypress. It sends a periodic heartbeat only while all three qualifying conditions hold; the server caps interval length, deduplicates, rejects impossible clock skew, and closes a presence window on lost qualification, settle, explicit unload beacon, session expiry, or a missing-heartbeat timeout. The grace interval is a feature flag calibrated toward several minutes, so a quiet watcher is not discarded after a short still spell. Heartbeats are facts, not an invitation to count raw tab-open time.

At each tick, aggregate newly processed qualifying presence duration and eligible interaction facts into a decayed per-bird signal. For trait `t`, use a bounded, slow low-pass delta such as:

`delta_t = remaining_headroom(t) * base_rate_t * smooth(quality_presence) * weighted_interaction_signal_t * elapsed_normalizer`

where `quality_presence` is capped qualifying minutes over a rolling window and interaction signals are capped per bird/session. Listen-in primarily contributes warmth/vocal-frequency; an accepted offer gives a small curiosity signal and making an offer nearby gives a small boldness signal. Settle influences the short-lived mood/scene state only. Clamp each delta to a small per-tick and daily maximum, and clamp each trait to its normalized range. No branch produces a negative trait delta from absence, neglect, cancellation, or an ignored offer. The headroom term naturally slows a mature bird rather than allowing repeated activity to saturate it in a session.

Calibration fixtures should simulate sparse, typical, and high-but-human presence patterns. Tune rates until normal regular use produces instrument-detectable change around one week and perceptible behavioral/visual differences around three weeks, while no single session yields a noticeable jump. Keep calibration coefficients/versioned rules server-side and rollout-gated; do not expose them in product responses.

### Mood, scene, greeting, and notebook rules

Model mood as a finite state machine/probabilistic transition matrix over `wary`, `content`, `curious`, `drowsy`, and `alert`, retaining a minimum dwell interval to prevent flicker. Transition weights combine current mood, local time, local weather, recent accepted interactions, adjacent bird call/alarm context, and personality. Examples: dusk raises drowsy probability, rain dampens vocal behavior, an accepted offer nudges content, and high boldness reduces wary probability. Mood persists in storage across sessions and ticks; opening a tab cannot reset it.

Each tick also selects perch intent/action, weather opportunity, and call-window scheduling. Wary birds bias toward back/scanning, content birds toward preening, curious birds toward sound/leaf attention, and drowsy birds toward low/fluffed poses. Weather remains rare and soft: short rain a few times weekly and occasional wind, with temporary small effects. Ambient leaves and feathers are client-only ornaments, never per-leaf server entities.

On an eligible host return, derive absence from the last qualifying-presence end, choose a greeting bird weighted by boldness/social warmth/mood, and create a short greeting cue with a deterministic variation seed. It must execute within the first one or two seconds after a usable snapshot, be one bird first, and stagger any later response. It must never be a text banner or synchronized chorus. A settle event enters a server-visible settled lighting/audio state; any valid re-engage/click event in the next five seconds cancels it. Closing without settle only ends presence.

Generate notebook candidates from durable meaningful changes (unusual greeting ordering, sustained quiet, weather/mood combinations, a distinctive accepted reaction), then apply a per-aviary sparsity gate. Target roughly one entry every few days for regular use and permit more only for real noteworthy moments. Templates compose named, lower-case, present-tense observations from current state; they never report trait numbers, session timestamps, streaks, or user behavior. Persist the final prose and source event IDs once, not a live template that can change historical observations.

### Call grammar runtime contract

The simulation supplies each bird's species motif family, stable timbre/signature seed, current mood/personality-derived modifiers, and time window. The browser's audio engine chooses/varies motif sequence, pitch contour, timing, envelope, and small noise components from those parameters. A signature fingerprint remains identifiable as timing/frequency changes. Bird-to-bird call responses and chorus windows are scheduled with offset randomness, not identical loop starts. Limit active call voices, use gain staging/compression, and enforce the seven-bird cap in server validation and renderer allocation.

## 6. Sync, integrity, and lifecycle

All device interactions are append-only facts. A phone and laptop may submit events concurrently; ordering is determined by durable server sequence, and simulation consumes each once. `idempotency_key` protects retries; a per-session sequence can diagnose gaps but never determines personality order. The read API exposes version and server clock so clients discard stale snapshots and interpolate only forward. After a rejected event (cooldown, revoked session, expired invite), the client removes optimistic presentation and shows system wording only when user action needs explanation.

Do not use last-write-wins for any bird field. Names/settings use account-authorized PATCH with version/ETag preconditions and a clear refresh/retry surface; personality/mood/scene have no PATCH endpoint at all. If an old client wakes after suspension, it drops stale interpolation, pulls a new snapshot, and resumes from canonical state. A client cache is encrypted/ephemeral enough to render an offline quiet field but must not queue arbitrary state mutations across logout or use cached bird state as current.

For deletion, revoke sessions and visit capabilities at soft-delete start, remove the aviary from normal reads, and schedule hard deletion exactly after 30 days. Restore cancels the job only before irreversible work. The hard-delete job erases account-owned primary records, encrypted email material, exports/capabilities, event/history rows, and operational identifiers associated with the account while retaining only non-identifying aggregate counters. Build a deletion ledger that proves completion without retaining bird or account content.

## 7. Frontend scene and interaction pipeline

1. Serve minimal HTML, critical CSS, and enough authenticated snapshot data to draw the quiet field/first bird without waiting for settings, notebook, invite, or account bundles. If data is late, show the calm quiet field with faint ambient cues; never show a spinner, static app shell, fade-in, or wake-up sequence.
2. Normalize the snapshot into scene nodes: background/sky, three perch zones, birds, weather, foreground ornament, and a separate top-bar layer. Render the first frame with all existing birds mid-action and call scheduling already live.
3. Use time-based interpolation from server timestamps and action envelopes, not frame-count animation. On a fresh snapshot, reconcile position/action targets without teleporting; on a long frame gap/visibility return, snap only where interpolation history is invalid, then resume gentle motion.
4. Run mood-specific idle micro-motion continuously while visible: preen, scan, head tilt, weight shift. Stop animation work when hidden, release nonessential draw loops, and pull a new snapshot before rendering on return. The authoritative simulation continues regardless.
5. Map pointer/touch/keyboard focus to birds. Clicking/tapping/focusing a bird begins listen-in; clicking again, empty scene, another bird, or moving focus away ends/rebalances it. Offers are reachable only through the top-bar affordance; users cannot direct-manipulate perch placement.
6. Implement settle as a several-second lighting/call envelope with an in-scene click undo window of five seconds. It is an optional gesture, not a blocking leave flow.

The responsive layout keeps every bird in frame: horizontally compress spacing on narrow phones, expand gaps on wide desktops, preserve scene aspect behavior, and never introduce pan, zoom, scroll, or crop. The top bar has only account/settings, accessibility, notebook, and offer affordances. It fades almost transparent after cursor stillness and returns on pointer/keyboard activity; keyboard focus prevents a focused control from becoming visually unavailable. The scene contains no badges, inline labels, hover-only tooltips, or app chrome.

Code-split settings, notebook history, export/deletion, and invitation management. Avoid large framework UI libraries, raster/audio asset packs, and per-bird DOM animation trees in the initial route. Treat the visual palette and contrast tokens as a design-system input, with calm natural colors and AA-compliant user copy at every day/night state.

## 8. Audio and accessibility surfaces

### Audio

Initialize WebAudio after the first browser-permitted user gesture while allowing the visual aviary to begin immediately. Implement each bird as a bounded voice graph (oscillator/noise/sample-free synthesis, envelope/filter, panner/gain), reuse AudioWorklet/node resources, and schedule only a short lookahead window. A species motif library is code/data, not recorded media. On listen-in, ramp the focused gain up and other birds down over a defined gentle envelope; other birds remain audible and no track is muted. Release/close nodes on teardown and enforce voice caps so a 30-minute session cannot leak buffers or contexts.

If WebAudio is unavailable, denied, or fails health checks, render graceful silence and automatically enable generated captions. Never download fallback recordings. Audio preference changes must be reflected without changing simulation/call schedules.

### Accessibility

Build a semantic interaction layer synchronized to the canvas scene:

- The top bar uses ordinary labeled controls. Tab reaches it; entering the aviary focuses the first bird; arrows move among birds; Enter toggles listen-in; Escape exits it; offer and settle are keyboard-complete. Focus indicators meet AA contrast across dawn, day, dusk, and night.
- A dedicated, politely managed live-region narration service generates naturalist state summaries from the same normalized snapshot as the renderer. At idle, publish one concise update every 30–60 seconds; coalesce superseded updates. Greeting, accepted-offer reaction, and settle descriptions get a controlled priority bump without becoming announcement spam.
- Narration describes observable place/bird state in lower-case present-tense prose, never raw perch numbers, mood labels, or personality values. Accessibility/account error UI uses the allowed matter-of-fact voice.
- Reduced-motion activates from `prefers-reduced-motion` and an explicit setting. Replace continuous micro-motion and flight paths with slow pose/perch cross-fades; remove leaf drift; retain slow day/evening color shifts, calls/captions, state changes, notebook, and all interaction meaning.
- Generate captions at call synthesis time from the actual grammar result (motif/pitch/rhythm/mood), anchor near the caller, and fade with it. Captions are concise naturalist descriptions, not a fixed species string. Let the user opt in; enable by default in audio-failure mode.

Test screen-reader queue behavior with VoiceOver, NVDA, and major browser combinations, keyboard-only use on desktop and mobile hardware keyboards, contrast in all light states, and reduced-motion behavior with animation instrumentation—not merely snapshot tests.

## 9. Performance, reliability, privacy, and observability

Make these release gates:

- Initial JS is under 2 MB gzip; route budgets are enforced by CI bundle analysis.
- On a defined mid-tier mobile/4G synthetic profile, the first bird is visible in under 500 ms. The initial render must not wait for noncritical chunks or audio permission.
- Idle visual motion sustains 60 fps on a five-year-old mid-range laptop for a 30-minute run; frame time and long-task budgets fail performance tests.
- The 30-minute soak test shows no client heap growth trend, audio-node/buffer count growth, leaked RAF loop, or retained offscreen notebook node.
- Simulation tick p99 latency alarms above five seconds, with backlog, lease-conflict, error-rate, and snapshot freshness alerts. Availability degradation serves a clear retry path rather than stale personality writes.

Instrument only aggregate operational measures: route/error counts, latency histograms, initial/first-bird timing, frame-time distribution, audio context error counts, synthetic regional checks, tick duration/backlog, and anonymous session-duration bins. Metric schema validation must reject bird IDs, account IDs, emails, names, interaction type/payload, state version tied to an account, notebook prose, and any high-cardinality per-user tags. Separate observability credentials/network access from the simulation database; prohibit warehouse reads from account/simulation tables. Review logging middleware to ensure request bodies, magic links, capabilities, and emails are redacted before export.

Security reviews cover magic-link single use/expiry, capability entropy and hashing, recipient email authorization, CSRF/session fixation, rate limits, encrypted PII access, export-link TTL, deletion race conditions, and immediate visit/session revocation. Visitor snapshot responses use private/no-store caching and validate capability on every snapshot pull, so revocation reaches the next polling interval.

## 10. Implementation sequence and rollout

### Build sequence

1. Establish schemas/migrations, synthetic IDs and encrypted email handling, magic-link/session lifecycle, soft/hard deletion skeleton, static app shell, and privacy-safe logging rules.
2. Implement aviary/bird/event/state models, transactional tick worker/lease, deterministic simulator fixtures, canonical snapshot endpoint, and property tests for monotonic drift, event ordering, stable identity, and no-client-vector mutation path.
3. Deliver the first two-bird renderer, day/night/weather/three-zone composition, quiet-field loading, snapshot interpolation, basic return cue, visibility recovery, and performance harness before feature-heavy surfaces.
4. Add presence qualification/heartbeats, listen-in, offers/cooldown, settle/undo, mood/action integration, notebook sparsity pipeline, and procedural audio/caption contract.
5. Build accessibility surfaces in parallel with each interaction, then add account settings/export/session management and deliberate visit invitation/read-only/revocation/log flows.
6. Run end-to-end multi-device, privacy, accessibility, resilience, and soak/performance validation; conduct content/voice review for all product copy and error-state copy.

### Controlled release

Start with an internal test cohort using synthetic accounts and fixture aviaries, then a small opt-in external cohort with two birds only. Enable the tick and notebook rules behind versioned configuration and compare only aggregate health/performance/error metrics plus consented qualitative research; do not aggregate personal interaction behavior to tune drift. Use simulator fixtures and direct user interviews for calibration. Increase from two starter birds only through age-based server eligibility, initially cap public rollout at two/three birds, then progressively enable the existing hard cap of seven once chorus recognizability, mobile frame time, and audio failure rates meet gates. There is no engagement-based unlock, prompt, notification, or growth metric.

Rollback strategy: retain backward-compatible snapshot/event schemas, feature-flag new simulation algorithm versions, and retain pre-change canonical records only inside the protected account store for a defined recovery window. A rollback restores the prior server algorithm/read model without replacing bird IDs or erasing drift. Never roll back by regenerating birds from logs or accepting a client cache as truth.

## 11. Validation matrix and principal risks

| Risk | Detection and mitigation |
| --- | --- |
| Drift feels like a counter or changes too fast | Deterministic multi-week fixtures plus rate caps/headroom; human review at one- and three-week equivalence points; keep trait values hidden. |
| Neglect accidentally produces negative drift or guilt UI | Property tests assert nonnegative trait deltas for absence; copy/UI review rejects decay, streak, and missed-session surfaces. |
| Concurrent devices lose drift | Transactional event sequencing, tick lease, idempotency fuzzing, and two-device suspend/resume tests; no absolute personality endpoint exists. |
| Presence is inflated by an idle/background tab | Browser lifecycle tests require all three qualifying signals; server heartbeat caps/timeouts; test overnight background scenarios. |
| Simulation state diverges after outage/missed ticks | Segmented catch-up, deterministic seed replay, state-version reconciliation, stale-client recovery tests, and worker backlog alarms. |
| Calls sound repetitive/uncanny or chorus collapses | Species signature listening tests, deterministic novelty checks, gain/voice budgets, seven-bird stress mixes, and no recorded-loop code path. |
| Audio fails or leaks in long sessions | Audio health fallback to captions, AudioWorklet/node soak tests, heap/voice counters, and 30-minute CI profiles. |
| Accessibility becomes a static fallback | Acceptance tests for naturalist narration cadence, actual generated captions, reduced-motion cross-fades, keyboard flows, and AT user review before launch. |
| First render exposes a loading-app feel | Synthetic 4G first-bird gate; inline critical snapshot/quiet field; visual review prohibits spinners, fades, and entry animations. |
| Invitation leak or revocation delay | High-entropy hashed capabilities, no-store responses, capability validation per pull, revocation integration test, 30-day expiry, and recipient-bound flow. |
| Private relationship data reaches telemetry | Compile-time metric allowlist, redaction tests, warehouse network isolation, routine log sampling, and deletion/export audits. |
| Scope creeps toward a social/game surface | Architecture/API review rejects profile/feed/chat/counter primitives; release checklist checks every visible surface against the non-goals. |

The v1 exit decision requires all release gates, no unresolved privacy/accessibility critical findings, and qualitative confirmation that the core session is a continuing aviary rather than a loading screen, dashboard, or obligation loop.
