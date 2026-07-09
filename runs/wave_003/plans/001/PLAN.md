# Pocket Aviary v1 implementation plan

## 1. Product contract and v1 scope

Build Pocket Aviary as a browser-only, single-user virtual aviary in which two starter birds share one calm, horizontal scene and evolve through slow server-side simulation. An account has one canonical aviary. The v1 ceiling is seven birds; additional birds become available by aviary age, never by visit count, interaction score, payment, or a visible progress system.

The shipped surface includes:

- email magic-link authentication, per-device sessions, account settings, export, and deletion;
- a single canonical aviary with two starter birds, stable bird identities, renameable user-assigned names, hidden personality vectors, moods, calls, bird-to-bird reactions, and age-based bird additions;
- a server-side simulation tick that continues without a connected client;
- the single-screen scene, day/night cycle, rare ambient weather, idle micro-motion, return-greeting, listen-in, offers, settle, presence accounting, and the read-only field notebook;
- multi-device snapshot sync, with the server as the only personality-state writer;
- screen-reader naturalist narration, reduced-motion rendering, procedural-call captions, keyboard access, and WCAG AA user-copy contrast;
- one quiet, opt-in, per-invite read-only visit flow with revocation, expiration, and a host visit log.

Explicitly exclude native apps, passwords/SSO, payments, shared or multi-aviary accounts, customizable scenes, public discovery, profiles, follows, comments, chat, co-presence, leaderboards, achievements, scores, streaks, visible visit-frequency metrics, push notifications, and any Tamagotchi mechanics such as hunger, death, distress, or decaying happiness. Do not add a generic welcome banner, toast, loading spinner, or “you have been away” announcement. The birds noticing the user is the welcome surface.

The principal acceptance criterion is affective as well as functional: a first frame must look like a place already in motion, a short return must feel different from a long absence, and a user should feel personality drift over weeks without ever being shown a stat to optimize.

## 2. Architectural shape

Use a modular web application with four hard boundaries:

1. **Browser client.** Authenticated clients pull immutable-ish state snapshots, submit append-only interaction events, and render/interpolate locally. The client owns transient presentation state (animation interpolation, audio graph, focus, caption visibility, top-bar opacity, local ambient leaf/feather ornaments) but never owns canonical mood, position, personality, presence totals, notebook truth, or bird identity.
2. **Application/API service.** Handles magic links, sessions, account settings, snapshot reads, event validation/append, notebook reads, exports/deletion requests, and host/visitor permissions. It authenticates account UUIDs and visitor capabilities separately and never accepts absolute personality values from a client.
3. **Simulation service.** A durable scheduler dispatches approximately one tick per minute per aviary, with a per-aviary lease or transactional lock. The worker consumes ordered events, applies elapsed-time and ambient changes, writes the canonical state, and advances a durable event cursor. It runs whether or not a client is connected.
4. **Persistence and delivery.** Store account/aviary state in a transactional database; keep the append-only event log and simulation cursor in the same consistency domain or behind an equivalent transactional boundary. Serve the initial HTML and a compact, authorized state snapshot from an edge-friendly path. Use a dedicated email provider for magic links and invitations. Keep operational telemetry in a separate aggregate-only pipeline that cannot read per-bird simulation records.

The v1 client can use pull-based synchronization rather than a real-time socket: pull on initial load, visibility restoration, long render-frame gaps/resume, and a low-frequency visible-tab keepalive. Snapshot payloads are kilobytes and include a monotonic snapshot version, server time, scene/day-night state, weather, per-bird presentation state, current mood, call timing, and active transitions. A future push transport may optimize freshness but must preserve this canonical protocol.

Use server time for ordering, tick elapsed time, expiration, and event cursors. Use the user’s stored timezone (or a validated browser timezone update) for local day/night and mood time-of-day signals. Visitor sessions receive the same host snapshot projection but no owner capabilities and no presence/interaction writes.

## 3. Data model and invariants

Define versioned records with generated UUIDs and immutable creation identifiers. All mutable records carry `created_at`, `updated_at`, and an optimistic/version field where applicable.

### Account and access

- `Account`: synthetic UUID primary key, one encrypted email record, verified-email status, stored timezone, accessibility/audio settings, visit-notification preference (off by default), deletion status/deletion deadline, and timestamps. Never derive a key, partition, telemetry dimension, or log identifier from email.
- `Session`: random opaque per-device token hash, account UUID, device metadata safe for display, issued/last-seen/revoked timestamps. Revocation is immediate for API authorization.
- `MagicLink`: hashed one-time token, account/email target, issued and expiry timestamps (15 minutes), consumed timestamp, and rate-limit metadata. Consumption is transactional and invalidates the token immediately.

### Aviary and birds

- `Aviary`: stable UUID tied one-to-one to an account, creation timestamp/age epoch, current canonical snapshot version, tick cursor, last tick time, current local-time context, settled state, and active weather/ambient-event state.
- `Bird`: stable UUID, aviary UUID, species key, user name, adoption timestamp, and immutable identity metadata. Name changes never replace the UUID or alter engine state.
- `BirdPersonality`: one canonical server-side row per bird with normalized boldness, social warmth, vocal frequency, plumage saturation, curiosity, and a schema version. Values are bounded and only changed by the simulation service. They are not returned as ordinary UI state or exposed in any stats/settings surface.
- `BirdMood`: current enumerated mood, mood-entered time, transition seed/state, and any bounded timers. Mood persists across sessions and is changed by ticks, not reset on client open.
- `BirdPresentationState`: current/target perch zone (front/middle/back), pose/action, action phase/seed, last call timing, and transition metadata. Keep it separate from personality so rendering can interpolate without mutating engine truth.
- `SpeciesDefinition`: a small v1 pool of approximately six species, each with silhouette/palette, stable call-signature parameters, motif grammar, caption vocabulary, and calibration version. Rarity is not a product mechanic.

### Events, notebook, and social access

- `InteractionEvent`: UUID/idempotency key, account and aviary UUIDs, server receive time, event type (`presence`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `settle_undo`, and relevant session lifecycle events), actor (`owner` only for simulation inputs), bird UUID/item type/duration where applicable, client event time for diagnostics only, schema version, and consumed tick/version. Presence events are intervals or bounded heartbeats, not unbounded “tab open” flags.
- `NotebookEntry`: UUID, aviary UUID, server observation time, generated naturalist prose, source observation/type, involved bird UUIDs if needed for internal traceability, and generation/version metadata. Read-only to clients; sparse generation is enforced by a time/event budget.
- `Invite`: UUID, host account/aviary UUID, normalized encrypted recipient email, single-use token hash, issued/expiry (30 days unused), used/revoked timestamps, and status. It is per-recipient and never creates a global discoverability flag.
- `VisitSession`: invite UUID, opaque visitor capability, start/end/last-pull times, approximate duration counters, and revocation/expiration checks. It can read an authorized snapshot projection but cannot append owner interaction or presence events.

Enforce these invariants in the database and service layer: exactly one aviary per account; two starters before the initial scene is complete; no more than seven birds; all bird UUIDs stable; personality writes accepted only from the simulation service; event appends idempotent; event processing ordered and resumable; visitor capabilities cannot be upgraded to owner access; and no per-bird state crosses into aggregate telemetry.

The account export is an authenticated, on-demand JSON snapshot containing birds, names, current personality vectors, current moods, notebook entries, and account settings as required by the PRD. Treat the export as sensitive: generate asynchronously, email only to the verified account address, provide a short-lived download link, and audit access without copying its contents into analytics.

## 4. API and authorization surface

Keep API contracts explicit and versioned. Return server timestamps and snapshot versions so clients can reason about freshness without merging state.

### Authentication and account APIs

- `POST /auth/magic-link`: validate an email, rate-limit per email and request source, create/send a 15-minute one-time link, and return a matter-of-fact confirmation.
- `POST /auth/magic-link/consume`: atomically consume the token and issue a per-device session. Replay/expiry returns a clear system error.
- `GET /me`, `GET/PATCH /me/settings`, `GET/POST/DELETE /me/sessions`: expose only needed account/settings/session data. `PATCH` rejects engine fields and supports accessibility/audio/timezone/notification preferences.
- `POST /me/export`, `POST /me/delete`, `POST /me/delete/cancel`: queue export; mark deletion immediately with 30-day recovery; hard-delete all account-tied records after the window.

### Aviary and interaction APIs

- `GET /aviary/snapshot`: authorize the owner session, return the current canonical snapshot plus server time, snapshot version, and any owner-only affordance metadata. It must not trigger a simulation tick or synthesize drift on demand.
- `POST /aviary/events`: accept a typed batch of idempotent events. Validate actor, bird membership, per-bird offer cooldown, settle/undo timing, duration bounds, and event sequence without accepting personality/mood writes. Owner presence is accepted only from the presence protocol described below.
- `GET /aviary/notebook`: paginated, chronological read-only entries with no visit-frequency statistics.
- `POST /aviary/birds/:id/name`: rename a stable bird identity; do not expose vector fields.

### Visit APIs

- `POST /visits/invites`, `GET /visits/invites`, `POST /visits/invites/:id/revoke`: host-only create/list/revoke, with exact recipient and 30-day unused expiration.
- `GET /visit/:token/bootstrap` and `GET /visit/:token/snapshot`: validate a single-use/active invite capability and return a read-only projection. Every pull rechecks revocation and expiration; a revoked active visit receives the matter-of-fact “visit no longer available” surface.
- `GET /visits/log`: host-only, on-demand list of recipient email, date, approximate duration, and outstanding invites. Do not badge or push this log.

Use CSRF protection for cookie-backed mutations, strict origin/token handling, rate limits on link and invite issuance, short-lived visitor tokens, encrypted email at rest, and audit records that use synthetic UUIDs. API error strings for account, sync, accessibility settings, unsupported browsers, and revoked visits are direct system language, never naturalist euphemisms.

## 5. Presence protocol and simulation engine

### Honest presence capture

The client tracks the three required signals locally: `document.visibilityState === "visible"`, window focus, and a recent pointermove or keypress. Only while all three hold does it send bounded owner presence heartbeats. Choose the exact activity window as a versioned calibration parameter (start with a few minutes, with a longer grace interval so quiet watching remains presence); send no presence when hidden, unfocused, settled, or inactive beyond the window. Settle and tab close end the presence window but neither is required for a valid session.

The server treats heartbeats as an idempotent bounded interval for the authenticated owner, deduplicates overlaps, caps impossible durations, and never counts visitor sessions. A client cannot claim presence by setting a duration or by keeping a tab open. Record enough event detail to reconstruct engine inputs, but do not send those details to analytics.

### Tick algorithm

Run a tick at approximately one-minute cadence, with a durable per-aviary lease and catch-up path for delayed workers. Each tick:

1. Acquire the aviary lock/version and read the last processed event cursor, current state, and elapsed server time.
2. Consume owner events in server receive order exactly once. Collapse duplicate heartbeats, normalize event durations, and preserve event IDs for audit/replay.
3. Advance mood timers, time-of-day, weather, ambient events, bird-to-bird call effects, and presentation targets over the elapsed interval. Mood is a small enumerated fast state (wary, content, curious, drowsy, alert, plus any finalized implementation states), persists across ticks, and is influenced by recent session interactions, local time, weather, nearby alarms, and personality.
4. Compute additive drift deltas from presence-time (dominant), listen-in attention, offers, and the settle boundary. Apply a slow low-pass filter with configurable, versioned coefficients; clamp every trait to its permitted range; allow positive movement toward expressive traits but never negative movement caused by neglect. The implementation must be calibrated so instruments detect change after about a week of regular visits and users feel it after about three weeks, without visible session-by-session jumps.
5. Update canonical personality rows and mood/presentation state in one transaction with the consumed cursor and incremented snapshot version. If a tick retries, the cursor/idempotency boundary makes it safe; if it cannot commit, it releases/lets the lease expire and retries without partial drift.
6. Generate a notebook observation only when a sparse cadence budget or genuinely noteworthy event permits it. Generate naturalist lowercase, present-tense, bird-specific prose; never emit raw event logs, trait numbers, visit counts, or achievement language.

The tick must be deterministic for a fixed prior state, ordered event set, server time, random seed, and engine version. Store engine/config versions with state transitions so calibration changes do not silently rewrite history. Add replay tooling for test fixtures and production incident diagnosis without exposing per-account records to analytics.

Implement age-based bird availability as a server decision derived from aviary creation age. Present an offer when the age threshold is reached, without a counter or progress UI. Adoption creates the next stable bird record from the species pool and never resets existing vectors or moods. At seven birds, no further offer is generated.

## 6. Frontend rendering and interaction pipeline

Split the client into a critical scene path and lazy secondary surfaces. The critical path receives HTML, compact initial snapshot data, species render definitions, and enough procedural audio grammar to draw the scene. Lazy-load account/settings, accessibility settings, notebook history, invitation management, export, and deletion UI. Do not block first bird rendering on non-critical assets or audio permission.

The scene renderer should:

- compose sky/background, subtle foreground/background separation, three perch zones, birds, and occasional client-only leaf/feather ornaments in a single horizontal responsive viewport;
- preserve every bird in frame on narrow phones and wide desktops; never pan, scroll, zoom, crop, or allow user placement;
- place birds from snapshot state with motion already in progress, including pose/action phase and seed, instead of an entry animation or spinner; use the quiet field only while initial state is unavailable and use the genuine empty state only during adoption;
- interpolate between snapshot positions/actions so a bird does not teleport when the one-minute tick changes its perch; pause unnecessary rendering while hidden and resynchronize on visibility return or a long frame gap;
- derive idle micro-motion from mood/personality presentation inputs: preen, scan, head-tilt, weight-shift, and mood-appropriate perch choices. Ambient ornaments are local rendering effects, not server-simulated per-leaf records;
- implement local-time day/night palette and weather state, with quiet evening settling and a nightjar-like active species path without treating night as a dead state.

Keep all chrome in a thin top bar: account/settings, accessibility, notebook, and offer. Fade it nearly transparent after cursor stillness and restore it on pointer/keyboard activity. Keep buttons, badges, hover tooltips, inline labels, and overlays out of the scene. Settle is in the top bar, has slow lighting/call ramps, and has a five-second click-anywhere-in-scene undo; ordinary tab close remains valid.

Implement interactions as explicit state machines rather than scattered event handlers:

- **Return-greeting:** on initial/visible return, choose one eligible bird using boldness, mood, absence length, and a seeded variation; stagger any follow-on greeting by a small randomized offset. Never render a textual welcome or absence-duration announcement.
- **Listen-in:** click/tap/focus a bird to ramp its gain up and all other bird gains down toward ambient, never to silence. Clicking again, empty space, another bird, or focus leaving the scene disengages and ramps back. Emit start/end events with bounded duration.
- **Offer:** open from the top bar; choose seed, song fragment, or still pool; target/resolve through server-approved bird and cooldown rules; animate reaction from snapshot/event acknowledgement. A curious/content bird may approach, wary may wait, and drowsy may ignore. The song fragment is a procedural/soft motif, not a downloaded recording.
- **Settle:** submit the event, ramp lighting over seconds, quiet calls, mark the client settled, and allow the five-second undo. A new owner interaction can re-engage. Presence ends regardless of whether settle was used.
- **Notebook:** fetch and render sparse, read-only entries; never turn them into a session/event feed or user-behavior report.

Treat snapshot acknowledgement as the boundary between optimistic presentation and canonical state. An offer can show a provisional response animation, but the next snapshot is authoritative. Never optimistically mutate a personality vector, mood truth, bird identity, or notebook history.

## 7. Audio and caption pipeline

Build a client-side WebAudio graph with reusable oscillator/noise/filter/envelope nodes or pooled buffers. For each bird, load a stable species/personality call signature and synthesize a fresh call from a small motif grammar with mood-shaped pitch/timing and vocal-frequency-shaped cadence. Seed runtime variation so calls are not identical while preserving recognition across mood and drift. Schedule calls against the render/audio clock, reuse resources, and dispose bounded transient nodes.

Use a master ambient bus and per-bird buses feeding a chorus mix. Listen-in changes bus gains with a slow ramp: the focused bird rises and others fall to ambient but never zero. Ambient weather, song-fragment offers, and staggered return-greetings use the same mixer without hard cuts. Keep the call grammar small and code/data compact enough for the initial bundle; do not ship recorded call loops.

Have the grammar return a caption descriptor at the same time as the scheduled call, such as “a soft three-note rise” or “a low trill, paused, low trill again.” Render it near the calling bird for the call’s lifetime with a gentle fade, and expose it through the narration queue when appropriate. Captions are runtime-generated from the actual motif, not a static string per bird.

If WebAudio is unavailable, denied, or fails initialization, continue visual simulation in graceful silence and enable captions by default. Do not introduce a recorded-audio fallback. Provide a clear audio setting and matter-of-fact error/permission treatment while keeping the rest of the aviary functional.

## 8. Accessibility and inclusive surfaces

Ship the accessible surface in the same milestone as the visual scene.

- Maintain a polite, bounded screen-reader narration queue generated from the same snapshot/observation model as the scene. Produce lowercase, present-tense naturalist prose roughly every 30–60 seconds at idle; prioritize a return-greeting, successful offer, and settle without flooding the queue. Do not expose “mood: content,” perch IDs, personality numbers, or raw state transitions as the primary experience.
- Honor both `prefers-reduced-motion` and an explicit accessibility setting. Replace micro-motion with slow cross-fades between still poses, replace flight paths with perch cross-fades, remove leaf drift, and retain slowed day/evening color changes, calls/captions, mood, drift, and notebook behavior.
- Provide a captions toggle in accessibility settings; if WebAudio fails, default it on. Ensure captions match the scheduled procedural call and meet contrast requirements.
- Implement a complete keyboard map: top bar in logical tab order; Tab into the scene focuses the first bird; arrow keys move between birds; Enter engages listen-in; Escape exits; offer menu and settle are reachable and operable without a pointer. Focus outlines must remain visible over bright and dim scene states.
- Test semantic names and roles for birds/actions without exposing hidden numeric traits. Ensure no focus trap in notebook/settings/invite dialogs and that returning from a lazy surface restores the prior scene focus.
- Validate all user-copy text, captions, narration shown visually, settings, account, error, and unsupported-browser surfaces at WCAG AA contrast. Keep naturalist voice only on the product surface; use direct matter-of-fact language for system surfaces.

## 9. Performance, observability, and privacy boundaries

Define performance tests as release gates:

- initial JavaScript bundle under 2 MB gzipped;
- first bird visible under 500 ms on the agreed mid-tier mobile/4G profile;
- idle scene at 60 fps on a five-year-old mid-range laptop;
- no client memory growth over a 30-minute session;
- snapshot payloads in kilobytes and no long-frame resync loop;
- simulation tick p99 below the five-second alarm threshold.

Use synthetic browsers from common geographies for navigation/first-bird/audio/accessibility checks and aggregate-only RUM for page load, first-bird-render, frame timing, audio-context errors, snapshot/tick latency, and anonymized session-duration histograms. Never attach account UUID, bird UUID, personality, mood, offer/listen/presence history, email, or reconstructed per-bird state to metrics. Keep operational logs on the synthetic UUID boundary and scrub tokens/emails.

Instrument engine-specific diagnostics only inside the simulation service’s restricted operational store: tick duration, lease contention, event backlog size, replay/version failures, and drift-calibration fixtures. The dashboard may aggregate service health, not user relationships. Set alerts for tick p99 >5 seconds, repeated failed commits, snapshot staleness, auth-link failure spikes, WebAudio failure spikes, and performance-budget regressions.

For memory safety, pool/reuse audio resources, cap active scheduled calls, release scrolled notebook DOM/data references, bound worker/audio contexts, and take heap snapshots before/after a 30-minute scripted session. For rendering, profile hidden-tab handling, low-power mobile, seven-bird chorus, reduced motion, captions, and rapid visibility changes.

## 10. Delivery sequence and rollout

Deliver in vertical slices while preserving the final contract at each slice:

1. **Foundations and contracts:** define versioned schemas, account/aviary/bird invariants, synthetic UUID boundary, auth/session threat model, API error taxonomy, engine configuration, and deterministic replay fixtures. Build the transactional event cursor and per-aviary tick lease before adding UI polish.
2. **Engine slice:** implement two starter birds, stable identity, hidden vectors, mood transitions, server tick, additive monotonic drift, age-based bird eligibility, bird-to-bird effects, sparse notebook generation, and snapshot serialization. Exercise catch-up, retry, concurrent-device, and visitor-exclusion cases.
3. **Critical scene slice:** render the initial snapshot as already-in-progress, add three perch zones, responsive one-screen composition, local day/night/weather, idle micro-motion, top-bar fade, return-greeting, visibility resync, and reduced-motion architecture. Gate first bird visibility and bundle size immediately.
4. **Interaction/audio slice:** add presence heartbeats, listen-in ramps, offer cooldowns/reactions, settle/undo, procedural WebAudio grammar, chorus mixing, caption descriptors, and the silence-with-captions fallback. Validate that no action directly changes canonical personality.
5. **Account and notebook slice:** complete magic links, settings, session revocation, rename, notebook pagination, export, soft/hard deletion, and direct system errors. Add keyboard/screen-reader narration, focus handling, contrast checks, and accessible reduced-motion behavior before considering the slice complete.
6. **Quiet visits slice:** add explicit per-invite email links, read-only host projection, no visitor presence/events, visit log, 30-day expiration, immediate revocation on next pull, and off-by-default notification preference. Test concurrent host/visitor snapshot pulls and revoked/expired tokens.
7. **Hardening:** run browser matrix (last two Chrome/Safari/Firefox/Edge), mobile/desktop performance profiles, seven-bird chorus and memory soak, screen-reader/manual keyboard passes, security/privacy review, deletion/export verification, and fault injection for tick retries, stale snapshots, auth replay, email delay, and WebAudio denial.

Launch with two birds for every account and progressive, server-controlled age eligibility for the third through seventh. Release behind operational flags for the tick engine, audio, offers, notebook, and visits so a faulty optional surface can be disabled without changing canonical state. Ramp from internal fixtures to a small authenticated beta, then broaden only after first-bird, tick p99, memory, accessibility, and privacy gates hold. Do not introduce engagement prompts, push campaigns, visible metrics, or a public directory during ramp.

## 11. Test strategy and acceptance gates

### Engine and persistence

- Property-test drift monotonicity, clamping, low-pass behavior, no change from neglect alone, and no single-session visible jump.
- Replay fixed event streams and assert deterministic state/snapshot/version results across retries, delayed ticks, catch-up windows, and concurrent devices.
- Assert only the simulation service can write personality; reject client absolute-vector mutations at API and database boundaries.
- Test mood persistence over session boundaries and local-time/weather transitions, bird-to-bird effects, stable IDs after rename, age-based additions, seven-bird cap, and sparse notebook generation.
- Verify exact-once event cursor advancement, idempotent presence overlap handling, per-bird offer cooldowns, settle/undo bounds, and visitor events excluded from owner drift.

### API, security, and sync

- Exercise magic-link expiry/replay/rate limits, session revocation, email-change verification, export access, soft deletion/recovery/hard deletion, CSRF, authorization, and token leakage checks.
- Run contract tests for snapshot versions, stale/resume pulls, event batches, matter-of-fact error surfaces, invitation expiration/revocation, and read-only visitor enforcement.
- Run two-device scenarios in which events arrive interleaved and prove additive server deltas preserve both sessions without last-write-wins personality overwrites.

### Browser and sensory surfaces

- Browser-test first frame with no spinner/entry animation, one-screen responsive layout, no crop at seven birds, top-bar fade, return-greeting variation, smooth listen-in ramps, offer reactions, settle undo, notebook read-only behavior, and weather/day-night transitions.
- Test WebAudio success, denied permission, unavailable API, pooled resource cleanup, caption-to-call correspondence, chorus recognizability fixtures, and no recorded-audio request.
- Test screen-reader narration cadence/priority, reduced-motion cross-fades, caption toggle, keyboard order/shortcuts/focus restoration, contrast, unsupported-browser messaging, and no accidental announcement surfaces.
- Run performance and heap soak gates on representative devices plus synthetic geographies; retain only aggregate measurements.

The release is complete only when the product works functionally and preserves the stated tone: no stat-management UI, no guilt for absence, no notification-driven social loop, no visitor influence on the host’s simulation, and no client-owned canonical state.

## 12. Risks and mitigations

- **Drift too fast or too slow.** Keep coefficients/config versioned, ship deterministic calibration fixtures, compare instrument-visible one-week and user-visible three-week targets, and adjust server parameters without exposing numbers or adding engagement metrics.
- **Presence inflation from background tabs or quiet watching.** Require visibility + focus + recent pointer/key conjunction, use server-deduplicated bounded intervals, calibrate a generous inactivity grace window, and test laptop/background/suspended cases explicitly.
- **Sync corruption or lost drift.** Make the server tick the sole personality writer, process append-only events by cursor, lock per aviary, use additive deltas and idempotent retries, and never merge client snapshots.
- **Catch-up or scheduler failure makes the aviary stale.** Persist last tick/cursor, alert on p99 and staleness, replay elapsed time deterministically, and expose only a direct system error if a snapshot cannot be produced.
- **Audio sounds repetitive, uncanny, or overwhelms at seven birds.** Separate recognizable signature from runtime variation, tune per-species grammars, use chorus/voice budgets, test with listeners, ramp gains rather than hard-cut, and fall back to captions/silence rather than recordings.
- **Initial-load performance breaks the already-running conceit.** Keep the critical path small, inline/edge-deliver the compact snapshot, lazy-load secondary surfaces, draw immediately from current pose/action phase, and enforce the 2 MB/500 ms gates in CI and synthetic monitoring.
- **Accessibility becomes a degraded afterthought.** Build narration, captions, reduced-motion presentation, keyboard flow, contrast, and screen-reader tests alongside the scene; treat them as launch blockers, not a later fallback.
- **Visits expand into a social network or influence simulation.** Keep invite capability narrow and per-recipient, default off, read-only, host-controlled, and explicitly exclude visitor events, chat, profiles, discovery, badges, and notifications by schema and authorization.
- **Privacy leakage through convenience identifiers or telemetry.** Use synthetic UUIDs everywhere outside the encrypted account email record, isolate the simulation store from analytics, scrub logs, and test export/deletion/telemetry payloads for per-bird fields.
- **Announcement/gamification drift.** Review every new surface against the voice and non-goal rules; reject welcome banners, streaks, visit counts, progress counters, achievements, and “helpful” engagement nudges in product and analytics schemas.
