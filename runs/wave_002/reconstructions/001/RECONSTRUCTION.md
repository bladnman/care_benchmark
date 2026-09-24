## System-level intent

- Quiet, sparse, opt-in presence is a governing product principle. It appears in the contract as "quiet, opt-in, read-only invitation flow," "default is silence," "No arrival toast or banner," and "no badge or push." It recurs in the notebook target of "roughly one entry every few days" and in the top bar as a "sparse bar requirement."
- The aviary should feel like naturalist observation, not a game dashboard. The plan excludes "progress counters, streaks, achievements, care meters," requires "lowercase, present-tense naturalist observation," and says notebook prose should not log "session starts, visit streaks, trait deltas, or generic action receipts."
- Personality is expressed through behavior, not displayed as statistics. The plan repeats "no raw vector," "never expose numerical traits," and says worker outputs are "presentation instructions, never displayed as statistics."
- The simulation is canonical, server-authored, and durable. The plan says "The database is authoritative," "The worker alone advances personality, mood, weather and behavior timelines," and "The client renders snapshots" with only "local, ephemeral interpolation and audio decisions."
- Growth is slow, bounded, and non-punitive. The plan requires "nonnegative" trait deltas, "no neglect subtraction," "absence of negative drift invariant," and states that "Two weeks away never decreases stored personality, resets birds, or generates distress."
- Aliveness comes from deterministic procedural variation under stable identity. The plan uses "deterministic seed protocol," "procedurally varied bird greeting," stable "call_signature_seed," and birds that vary while remaining recognizable, as in "Pip remains recognizable across mood and trait drift."
- Privacy and least privilege are cross-cutting constraints. The plan requires encrypted email fields, "aggregate-only operations pipeline," "no account UUID, email, bird ID, trait, offer/listen-in history, or per-bird state in analytics," and separate visitor scopes with "no event-write route."
- Accessibility is a first-class aviary experience, not a stripped fallback. The plan says "All supported sensory paths retain the aviary experience," names "naturalist narration, call-matched captions, and slow pose cross-fades," and declares "No launch waiver for accessibility."
- Returning accounts should see an already-running world. This shows up in "first frame depicts an already-running aviary," "never animate an entry for returning accounts," and "Arrival does not reset the world."
- The product avoids pressure loops and social mechanics. The plan keeps "public discovery, comments, feeds, co-presence," onboarding sharing prompts, payments, and aviary notifications out of scope; adoptions are "never usage" and "never based on visits or score."
- Failure and account surfaces should be plain and direct. The plan says "Identity, settings, access, and failure surfaces use direct matter-of-fact language" and calls for "plain system copy" in typed errors and "matter-of-fact unavailable state" on revoked visits.
- Operational correctness is part of the experience. The plan emphasizes monotonic revisions, idempotency, single-consumer tick lease, backup/restore drills, "tick p99 >5 s" alerts, and preserving bird IDs/vectors instead of reseeding.

## Per-feature whys

### 1. Product contract and decisions

- One browser aviary per email magic-link account: NOT RECOVERABLE FROM PLAN
- System-chosen birds at adoption: The plan later says the two starting species should have "contrasting signatures," then the user names them, so the recoverable rationale is recognizable contrast before user naming.
- Age-based growth to seven birds: The plan says later adoption is offered by "aviary age," "never usage," "never based on visits or score," and rollout only ramps to seven after "seven-voice chorus, responsive layout, accessibility and perf gates pass."
- One-screen horizontal scene with front, middle, and back perch zones: The plan wants the full scene visible on "narrow phones and wide screens, without scroll/pan/zoom," with depth carried by front/middle/back zones.
- Local-time light: The account time zone determines "visual day/night and daily mood boundaries," and visitors see the host's "world time."
- Rare mild weather: The weather schedule is "sparse" and deterministic, with "brief, gentle effects" and "no dramatic storms," fitting the quiet aviary tone.
- Procedural calls: The plan rejects "recorded call loops" and wants generated calls that "vary" while each bird remains recognizable.
- Mood-shaped motion: The plan maps mood and traits into bounded render behavior, such as wary birds scanning from back perches, content birds preening, curious birds investigating, and drowsy birds settling low.
- Listen-in: The plan lets the user focus one bird by smooth bus-gain changes while "other birds remain audible at an ambient floor."
- Specific seed/song-fragment/still-pool offer set: NOT RECOVERABLE FROM PLAN
- Optional settle: Settle "gradually quiets calls," warms evening light, ends presence "with no penalty," changes "current quietness, not personality," and "should not count itself as a drift reward."
- Sparse read-only field notebook: Notebook entries are for "noteworthy aviary events" and "specific detail," not "streaks," "trait deltas," or "generic action receipts."
- Quiet opt-in read-only invitation flow: Invitations require "explicit host action"; visits are read-only, have "no event-write route," and "Visitor activity produces zero host interaction events and zero drift input."
- Device sessions: The plan uses per-device session issue, session list/revoke, token rotation, and revocation so account access can be inspected and revoked by device.
- JSON export: Export is a "user-initiated data copy"; it includes numerical vectors only because "the PRD requires them," while keeping vectors out of product UI stats.
- Deletion and recovery: The lifecycle is an "immediate soft-delete state, 30-day recovery, scheduled hard deletion" with deletion of objects and related rows.
- Narration, captions, and reduced-motion rendering: These preserve the aviary across "supported sensory paths" through "naturalist narration, call-matched captions, and slow pose cross-fades."
- No arrival toast, banner, or text welcome: The plan replaces text welcome with a "procedurally varied bird greeting" and keeps the surface quiet.
- Product prose style: The plan specifies "lowercase, present-tense naturalist observation" for product prose and "direct matter-of-fact language" for identity, settings, access, and failures.
- The 4-minute recent-activity presence window: NOT RECOVERABLE FROM PLAN
- The 3-minute per-bird offer cooldown value: NOT RECOVERABLE FROM PLAN
- The 20-second live snapshot polling interval: NOT RECOVERABLE FROM PLAN
- The 30-day unused invitation token expiry value: NOT RECOVERABLE FROM PLAN
- First visible bird within 500 ms and already-running return frame: The plan treats first-bird timing as an acceptance gate and says returning accounts should not wait for a second trip or see an entry animation.

### 2. Architecture and ownership

- Edge document, bootstrap snapshot, authenticated API, relational store, simulation worker, mail sender, and aggregate-only operations pipeline: The split keeps the database authoritative, simulation server-owned, mail separate, and operations aggregate-only.
- Database authority, append-only input log, worker-only simulation, API validation, client rendering: The plan's rationale is that the API "never accepts client-authored trait, mood, or position values" and the client makes only "ephemeral interpolation and audio decisions."
- Shared versioned snapshot schema and deterministic seed protocol: The plan uses this between worker and client so snapshots, migrations, and multiple devices stay coherent.
- Public render snapshot without raw personality-vector numbers: The plan bars raw vector numbers from client, visitor, UI component, accessibility label, and telemetry event.
- Stable bird identity and grammar signature across migrations: The plan wants per-bird identity and call grammar continuity, saying not to regenerate IDs during rename or migration.
- Authenticated bootstrap snapshot with quiet sky fallback: The bootstrap avoids waiting for a "second round trip," while the quiet field avoids showing a cached bird state as canonical if authorization fails.
- Code splitting notebook, settings, invitations, and export: The rationale is first-frame performance; noncritical surfaces should not block the visible bird.
- Scene and chrome separation: The plan keeps the scene as "only birds/place" and puts account/settings, accessibility, notebook, and offer in a thin top bar to preserve the sparse aviary.
- Settle reachable through top bar or directly accessible top-bar action: The plan wants settle discoverable and accessible without placing chrome in the scene.

### 3. Persistent data and migrations

- Encrypted account email plus keyed nonreversible lookup index: The plan keeps email only on the encrypted record while preserving uniqueness and rate limiting without logging normalized email.
- Hashed sessions and magic links with transactional email-change verification: The plan supports one-time magic-link auth, replay failure, per-device sessions, and safe address replacement.
- Stable bird IDs, versioned trait values, and strict bounds: The plan says bird IDs are immutable and traits must be bounded and versioned so rename and migration do not reset identity.
- Append-only interaction events with validation and idempotency: The plan needs canonical host inputs in stable order and safe retries, with event type and bird ownership checked at ingress.
- Raw-input retention limits and presence segments: The plan records validated intervals rather than "unbounded ping rows" and purges raw inputs after simulation/audit need.
- Bounded recent snapshot window: The plan retains current plus recent snapshots for "debugging/recovery" while keeping canonical render JSON separate.
- Append-only notebook entries: The plan makes them immutable to users, chronologically queryable indefinitely, and unavailable for individual edit/delete.
- Invitations, visit grants, and visit sessions: The plan discloses visitor identity through invited email only, supplies a host-only visit log, and keeps visitor state out of the host simulation input log.
- Export and deletion jobs with short-lived encrypted links: The plan keeps these records minimal and makes export object storage short-lived and encrypted.
- Database constraints on aviary count, bird count, idempotency keys, revisions, and tick lease: The plan uses constraints to enforce one aviary/account, 2-7 birds after adoption, monotonic snapshots, and single-consumer ticking.
- Backups, restore, vector validation, and halt-on-corruption: The plan prefers halting mutation for an aviary over "silently reseeding" when vectors are missing or invalid.

### 4. API contract

- Host-session mutations with CSRF, ownership checks, request-size limits, and idempotency: The plan grounds these in security and retry behavior.
- Typed error envelope, plain system copy, API/schema versioning, UTC timestamps, and account time zone: The plan wants direct errors, versioned contracts, server UTC consistency, and local visual day/night.
- Magic-link request endpoint: Generic response avoids "account enumeration"; rate limits reduce abuse; the link expires in 15 minutes.
- Magic-link consume endpoint: Atomic consume plus per-device session issue makes the token one-time, and replay "fails clearly."
- Snapshot endpoint with ETag/revision, no raw vector, and access-checked 304: The plan keeps snapshots small and cache-efficient without leaking invalidated access or personality vectors.
- Batched host events endpoint: The endpoint validates cooldown and order, returns accepted event IDs, and never returns "speculative personality," preserving the canonical worker.
- Rename-only bird patch: Renaming "affects copy only" and must keep ID, vector, and mood.
- Adoption endpoint: Server checks age schedule, count, and idempotency; adoption is "never based on visits or score."
- Account settings, sessions, verified email-change workflow, export, deletion, and recovery: The plan groups these as account lifecycle and access-control surfaces with conflict checks and verified address workflows.
- Invitation issue/list/revoke endpoints: Invitations are one-time emailed tokens, expire if unused, require explicit host action, and revocation updates the grant immediately.
- Visit redeem/snapshot/end endpoints: They return the exact same render snapshot at the host's world time, recheck grants, and provide "no event-write route."
- Host-only visit log: The plan allows invited email, date, approximate duration, and outstanding invites, with "no badge or push."
- Scoped visitor token: The plan says it cannot use host actions, notebook, settings, or account endpoints and is invalidated on revocation.
- Optional visit notification mail job: Mail is sent only after settings opt-in, preserving "no default notifications or onboarding prompt."

### 5. Canonical tick, presence, mood and drift

- Per-minute canonical tick scheduled even when no browser is connected: The plan keeps the aviary world advancing on the worker, not in the browser.
- Worker lease, stable event order, atomic commit, and idempotent tick boundary: These keep revisions monotonic, prevent stale writes, and apply deltas once.
- Bounded deterministic catch-up after delay or outage: The plan replays missed boundaries without inventing presence and publishes only the newest state.
- Presence only while visible, focused, and recently active: The plan treats presence as validated attention evidence, not hidden-tab time.
- Presence segments ending on hidden, blur, settle, close, or activity expiry: The plan bounds durations and lets closing or settling end presence "with no penalty."
- Unioning duplicate devices for the same host: The plan prevents two open screens from double-counting attention.
- Presence pings as "input evidence, not an engagement metric": This protects the product from metric-like care loops.
- Slow nonnegative low-pass trait drift: The plan calibrates so one intensive session is invisible, regular weekly use is measurable, and about three weeks is perceptible.
- Presence, listen-in, offers, plumage, cooldowns, and daily caps: Presence is the "dominant shared positive input"; listen-in and offers are modest; repeated offers are ineffective as a "drift accelerator."
- Settle and stored personality: Settle changes "current quietness, not personality."
- Recency/absence factor separate from personality: The plan allows reduced greeting likelihood after absence while forbidding distress, punishment, or lowered stored traits.
- Persisted mood enum with bounded dwell times and conditioned transitions: The plan wants mood continuity across sessions and "no snap to neutral."
- Host arrival greeting: Selection is weighted by boldness, warmth, mood, absence, and greeting history, producing varied stateful greetings and avoiding cross-device duplicates.
- Notebook candidate observations: Rarity gates, spacing, templates, and duplicate suppression keep entries specific and non-repetitive.

### 6. Bird behavior and audio

- Species pool with distinct silhouettes, muted palette, and compact motif grammar: The plan uses distinct visual and call signatures so birds are recognizable.
- Exact "about six species" pool size: NOT RECOVERABLE FROM PLAN
- Provisional later-adoption timing such as third bird after several months: NOT RECOVERABLE FROM PLAN
- Nightjar-like signature remaining possibly active at night: NOT RECOVERABLE FROM PLAN
- Local WebAudio procedural synthesis from seeded schedules: The plan avoids distributing audio files, keeps timing coherent across devices, and avoids "recorded-loop fallback."
- Motif grammar with immutable per-bird signature constraints: Calls can vary while the bird remains recognizable across mood and trait drift.
- Gain, headroom, soft limiter, chorus eligibility, response jitter, and seven-bird listening tests: The plan avoids clipping and synchronized arrival while preserving seven-bird recognizability.
- Listen-in controls by click/tap/Enter, second activation, another bird, empty-space click, focus leaving, or Escape: The plan makes focus and exit predictable across pointer and keyboard use.
- Offer song fragments through the same mixer: The plan keeps procedural motifs in the same audio system as calls.
- Audio node reuse, hidden-tab suspend, gesture unlock, and silent/caption fallback: The plan handles performance, browser autoplay rules, and truthfully avoids claiming audio where it cannot play.
- Call-derived captions and accessible transcript: Captions come from the actual motif, contour, timbre, and source location so muted or silent users still get meaningful call information without flooded live regions.

### 7. Frontend scene and accessibility

- Responsive full-viewport coordinate system with safe perch positions: The plan keeps every silhouette visible from two through seven birds on phones and wide screens without scroll, pan, or zoom.
- Quiet background, parallax, and local seeded ornaments: The plan makes leaves and feathers ambient visual detail, not server state.
- Scene graph chosen by measured mobile performance: The plan leaves Canvas/WebGL/SVG/DOM open so the renderer can meet mobile performance.
- Server-time interpolation, bounded correction, and no teleport: The plan preserves continuity while keeping the server canonical.
- No entry animation for returning accounts: This supports the "already-running aviary" return frame.
- Idle loops as phase-offset pose blends and stochastic actions keyed by mood: The plan avoids fixed loops while making mood visible through behavior.
- Settle animation with five-second reversal and keyboard equivalent: The plan makes quieting reversible and accessible without adding drift reward.
- Reduced-motion mode from the same snapshot and behavior events: The plan preserves the same aviary events through slow cross-fades, no leaf drift, and slower palette changes for vestibular comfort.
- Screen-reader narration cadence, priority queue, and on-demand current-scene description: The plan gives specific present-tense prose without turning the aviary into a noisy ARIA event stream or making users wait to inspect it.
- Keyboard order, focus visibility, WCAG AA, semantic names, and supported screen-reader tests: The plan requires every host interaction and settings surface to work by keyboard and screen reader.

### 8. Sync, failure behavior, security and privacy

- Snapshot pull on start, visible interval, visibility return, frame gaps, and accepted actions: The plan keeps clients converged by revision and discards older responses.
- Immediate reversible client reaction after an accepted event: The plan allows responsive feedback while waiting for canonical reconciliation.
- No last-write-wins path for personality and row-version conflicts for settings/name changes: The plan avoids silent overwrites.
- Offline mode with last known state, quiet connection status, bounded queue, and no local drift: The plan says not to "pretend the simulation advanced locally as canonical."
- Short-lived single-use hashed tokens, TLS, encrypted email, secure cookies, rotation/revocation, and separate scopes: The plan grounds these in account and invitation security.
- Grant recheck on every visit snapshot including conditional requests: Revocation becomes visible at the next poll and prevents stale read access.
- Threat modeling account enumeration, token replay, invite forwarding, CSRF, cache leakage, and privilege escalation: The plan identifies these as security risks to cover.
- Aggregate-only telemetry with no account, bird, trait, action, or per-bird state fields: The plan isolates operations from simulation data and private histories.
- Ephemeral request ID and no simulation-store read path into analytics: The plan prevents incident correlation tokens from joining to bird history.
- Export vectors only in private JSON with short-expiry one-time link: The plan satisfies the required data copy while keeping numerical vectors out of product UI and analytics.

### 9. Performance, tests and observability

- CI budgets for JS size, first bird, 60 fps, and memory slope: The plan makes performance a launch gate rather than a best effort.
- Separate warm signed-in, cold-auth, and adoption measurements: The plan says failures should not be hidden by combined timing.
- Critical scene inline, noncritical lazy-load, compact silhouettes, kilobyte snapshots, reuse pools/audio, and virtualized notebook: These support first-load and sustained-session performance.
- Synthetic geographies, aggregate RUM, and operational alerts without per-account bird analytics: The plan monitors health while preserving aggregate-only telemetry.
- Deterministic engine harness with fixed seeds and simulated weeks: The plan uses property tests to assert bounded nonnegative drift, mood continuity, idempotent replay, and no visitor drift.
- Delayed-tick concurrency tests: The plan proves two devices' deltas apply "exactly once."
- Presence tests for visibility, focus, idle, settle, duplicate devices, suspended tabs, and clock skew: These validate the attention-evidence rules.
- Visitor contract tests: The plan verifies visitors cannot write, cannot read notebook/account data, and lose access after revocation.
- Visual/audio review and blinded human review: The plan uses human judgment for "felt aliveness," greeting variety, seven-bird call recognizability, notebook specificity, and absence of canned loops.
- Accessibility tests and no launch waiver: Keyboard, screen-reader queue pacing, caption-to-sound matching, contrast, and reduced motion are release requirements.

### 10. Delivery sequence and rollout

- Building schemas, encrypted email handling, auth/session lifecycle, event ingestion, canonical tick, migrations, backups, and tests first: The plan wants durable foundations before client behavior depends on them.
- Building the two-bird scene, bootstrap, local-time world, mood motion, calls, presence, greeting, offer, listen-in, and settle next: The plan establishes the core aviary loop before broader account surfaces.
- Adding notebook, adoption schedule, settings/export/deletion, and quiet invitations after the core: The plan specifically says to keep visitor scopes separate from host routes.
- Completing narration, captions, reduced motion, keyboard coverage, contrast, fallback, and the 30-minute performance suite before external exposure: The plan treats accessibility and performance as gates.
- Dogfood, small opt-in cohort, and staged bird-count ramp through three, five, then seven: The plan ramps only after chorus, layout, accessibility, and perf gates pass.
- Age schedule and staged flag behavior: Users still gain birds slowly, and a staged cap "must never remove an already adopted bird."
- Rollback by versioned snapshot and grammar protocols: The plan permits rollback without "rewinding vectors or bird IDs."
- Calibration through synthetic trajectories and consensual qualitative research: The plan explicitly rejects population-level analysis of private per-bird interactions.
- Deletion completion and privacy-pipeline isolation as release gates: The plan makes privacy operations part of rollout readiness.

### 11. Main risks and mitigations

The risk section mainly restates feature rationales already recovered above: drift calibration and bounded inputs mitigate gameability; single writer, ordered event offset, idempotent retries, and restore drills mitigate divergent state; procedural grammar and long-session listening review mitigate canned audio; bootstrap and compact assets mitigate first-frame delay; narration, captions, and reduced motion mitigate accessibility fallback risk; encrypted storage and telemetry allowlists mitigate privacy leaks; partitioned always-on workers mitigate tick cost and outages; hashed one-time tokens and rechecked grants mitigate invitation leakage.
