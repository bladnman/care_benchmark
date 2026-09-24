## System-level intent

- **A living aviary, not an engagement game.** This appears in the delivery contract's exclusions: "no score, visit streak, level, distress, feeding need, public discovery, co-presence, or user-editable scene." The review checklist must reject "UI or telemetry that turns attention into a counter," and later observability says product success is never "a visit-frequency leaderboard or account behavior warehouse."
- **Absence has no moral weight.** The plan says "a missing settle gesture and a long absence have no penalty," that "absence causes no harm," and that vectors are "never ... decremented on absence." Slow drift keeps absence from becoming "an event or hidden debt"; a two-week return may be "calmer but never distressed, mistrustful, or desaturated."
- **Continuity is canonical, server-owned, and cross-device.** The v1 release is complete only when a returning user on two devices sees "the same continuing birds." The client "never advances mood, drift, weather, adoption eligibility, or offer cooldowns," while row locks, revisions, and the tick worker preserve a single canonical scene.
- **Stable identity matters more than reconstruction.** Birds have "stable ID" values, IDs "survive rename, migrations, and species-library revisions," and migrations must preserve "bird UUIDs and perceptual continuity." The plan explicitly says to "never regenerate a bird from an event log."
- **Privacy is structural.** Email is encrypted and "never a partition key, telemetry dimension, or log identifier." Snapshots never expose raw vectors, metrics have "no account ID, bird ID, event payload, vector, invitee email, or per-account interaction history," and deletion destroys the per-account encryption key.
- **Observation must be truthful and sparse.** Notebook entries come from "persisted observation facts, not arbitrary session logs or raw vectors"; comparative claims require stored fact windows; entries must be "sparse, true, and naturalist" and never mention "user attendance, streaks, trait numbers, or fabricated facts."
- **The product voice is quiet, lowercase, and present-tense naturalist observation.** The delivery contract sets "lowercase, present-tense naturalist observation" for product copy, while "identity, errors, sync, and settings use direct matter-of-fact language." Return and visit states avoid toasts, banners, badges, and prompts.
- **Equal access is part of the aviary, not a later patch.** Screen-reader narration, reduced motion, captions, keyboard paths, contrast, no-audio behavior, and account controls are "launch requirements, not a later patch." Both motion modes use "the same bird/mood/drift state and notebook," and accessible mode must not lose "the aviary's character."
- **The first impression should feel like ongoing life, not software startup.** The plan asks for the first bird to render without an extra round trip, "no spinner, canned entry, or fade from a static bird," "phase-anchored motion," and a gate for first-bird performance. A listed risk is that the "first frame feels like software starting."
- **Visits are deliberate, revocable, read-only glimpses.** There is "no global sharing switch"; each invite is a "deliberate grant." Visitors see the actual canonical aviary but cannot write events, undo settle, access notebook/settings/export, or influence simulation.
- **Release decisions are gated by evidence.** Coefficients, timing, call grammar, adoption capacity, accessibility, and performance all require replay, fixtures, user studies, blinded comparisons, or release CI. The plan repeatedly uses "gates," "release signal," "stop-ship," and staged rollout language.

## Per-feature whys

### Delivery contract

- **One browser aviary per email magic-link account.** The rationale is continuity and private identity: the plan requires "multi-device continuity," one account owns one aviary, and the release definition requires the same continuing birds on two devices.
- **Stable bird IDs and user-chosen names.** Stable IDs preserve identity through rename, migration, and species-library revisions; names support naturalist notebook and narration templates that name birds without exposing traits.
- **Two system-chosen starter birds.** NOT RECOVERABLE FROM PLAN
- **Age-based invitations to adopt more birds up to seven.** The rationale is non-gamified growth: adoption remains based "solely on aviary age," with "no alert, badge, score, rarity catalog, or visit-based acceleration."
- **The exact adoption ages of 90, 180, 270, 365, and 540 days.** NOT RECOVERABLE FROM PLAN
- **A single, unscrolling scene.** NOT RECOVERABLE FROM PLAN
- **Return greetings.** They make returns responsive to "elapsed absence" and current bird state while avoiding return text, badges, banners, or synchronized performances.
- **Idle presence.** The plan wants "quiet watching" to count while preventing background inflation; presence is the dominant drift evidence without becoming a score or streak.
- **Listen-in.** It lets a user attend to one bird's call while other birds remain audible at a "nonzero ambient floor"; accepted listen-in events can contribute to drift, and audio-disabled users still get captions and the interaction.
- **Seed, song-fragment, and still-pool offers as the exact offer trio.** NOT RECOVERABLE FROM PLAN
- **Offer reactions.** Offers create immediate, mood-shaped bird cues; "ignoring is a valid bird reaction," cooldown rejections stay quiet, and offer spam cannot saturate traits.
- **Settle.** Settle makes a shared quiet evening presentation cue, ends the initiating device's presence interval, and creates no negative signal if missed, closed, or later cleared.
- **The five-second settle undo window.** NOT RECOVERABLE FROM PLAN
- **Sparse read-only field notebook.** The rationale is truth and calm: entries are immutable, fact-backed, "about one entry every few days," and never become entry-per-session behavior.
- **Multi-device continuity.** The plan unions overlapping attention once, applies ordered command streams once, shares persisted cues, and prevents stale devices from uploading absolute state.
- **Account export.** Raw vectors are hidden from the aviary UI but included in on-demand JSON as "user-owned data, outside the aviary experience."
- **Account deletion.** Deletion disables grants, jobs, sessions, records, object-store data, and account-correlated records, then destroys the per-account key so backups become unreadable.
- **Revocable read-only visit invitations.** The rationale is controlled sharing: a named invite is a deliberate grant, revocation is checked on every snapshot, and visitor activity cannot affect simulation.
- **Procedural-call captions.** Captions come from the same instantiated call event, so the visual text matches the actual audible contour and timing rather than a static species string.
- **Lowercase naturalist copy and matter-of-fact account/error copy.** The product voice separates living observation from identity, sync, settings, and failure states, which use direct language.

### System boundaries and deployment shape

- **Small edge-served HTML/JS shell with authenticated API, database, workers, scheduler, email worker, and private export storage.** This shape supports first-load speed, server-owned simulation, magic-link delivery, due ticks, and private short-lived exports.
- **One deployable API/worker codebase with separate process roles.** The rationale is to start simply while keeping simulation, API, scheduler, and worker responsibilities distinct.
- **Pure, versioned simulation module.** Determinism and versioning allow golden replay fixtures, restart safety, worker reassignment, migrations, and perceptual gates.
- **Relational database with row locking and transactions.** The plan chooses this "rather than cross-device client state merging" so concurrent devices cannot overwrite canonical state.
- **Append-only `interaction_event` input journal with current rows authoritative.** The journal supports ordered replay and idempotency, while current bird and aviary rows avoid rebuilding identity and personality from event logs.
- **Bounded consumed-event retention.** The rationale is an "operational replay window" followed by purge, preserving only current state, notebook, and minimal derived facts.
- **The exact 30-day consumed-event retention duration.** NOT RECOVERABLE FROM PLAN
- **Tick worker as the only path for personality writes.** This prevents API commands, retries, or stale clients from racing or directly changing vectors.
- **Client-owned presentation state only.** The client may handle interpolation, ornaments, audio, focus, and panels, but cannot advance mood, drift, weather, adoption eligibility, or cooldowns.
- **Persisted short-lived presentation cues.** Immediate command responses become visible to all connected host devices in snapshots, while the initiating client can render at once.
- **Initial authenticated HTML includes private scene projection and server timestamp.** This lets "the first bird render without an extra API round trip" and supports the first-bird budget.
- **`Cache-Control: private, no-store` for the authenticated projection.** The rationale is privacy: it is a compact private scene projection and must never be served from a shared cache.
- **Critical scene renderer and starter silhouettes on the initial path, with later UI and assets lazy-loaded.** The rationale is first-bird performance and a small critical path.
- **Quiet sky field recovery.** If the snapshot is unavailable, the scene still paints quietly; matter-of-fact retry/error UI appears only if recovery fails, with no spinner or fake entry.
- **Versioned presentation schema excluding raw personality values.** The schema gives clients enough to render while keeping personality vectors hidden from normal UI and snapshot APIs.
- **Fresh snapshot and quiet-field recovery on version mismatch.** The rationale is safe compatibility without resetting bird identity.

### Persistent data and invariants

- **UUID account identity with encrypted email and email lookup HMAC.** The plan keeps UUID as the sole internal account identifier so email is not a partition key, telemetry dimension, or log identifier.
- **Hashed one-use auth links and revocable per-device sessions.** Atomic consume makes replay fail, and session revocation takes effect on the next authenticated request.
- **Aviary timezone, lease owner, revision, due tick, and scene seed.** The timezone and lease keep all clients rendering the same host-time scene without device oscillation; revision changes only with server commits.
- **Bird rows with stable UUIDs, vectors, mood, perch, offer, expression, and UTC-day counters.** The rationale is durable identity and monotonic personality: vectors are persisted, server-written, and never decremented on absence.
- **Host-authored `interaction_event` rows with idempotency keys.** Unique keys prevent retry duplication, and visitors cannot inject events into simulation.
- **`presence_interval` with coverage ledger.** Tick uses newly covered UTC seconds from the union of overlapping qualified intervals so concurrent devices do not multiply attention.
- **`notebook_entry` and `observation_fact`.** Stored facts make truthful comparisons possible, while entries remain append-only and avoid visit frequency or numeric traits.
- **Invitation, visit session, and visit log records.** The rationale is scoped, revocable, read-only access with approximate visit logging and no global sharing switch.
- **`export_job` with encrypted object pointer and one-use token.** Export delivery stays private, short-lived, verified-email based, and deletion-aware.
- **Schema constraints for one aviary per account, max seven birds, unique event keys, and valid transitions.** The plan uses constraints and transactions to enforce invariants rather than trusting clients.
- **Per-account envelope encryption for email and private export/visit data.** This supports deletion by key destruction and keeps account/private sharing data protected.
- **Simulation database restricted from analytics at network and credential layers.** The rationale is to keep raw relationship, vector, and per-account history out of analytics.
- **Aggregate-only operational metrics from request and worker boundaries.** Observability is allowed only without account ID, bird ID, event payload, vector, invitee email, or per-account history.
- **Identical magic-link request responses for existing and new addresses.** The rationale is to avoid revealing account existence while rate limiting per email and IP.
- **Secure HTTP-only same-site cookies, CSRF protection, sign-in session rotation, and device revoke list.** These features protect authenticated sessions and give the user visible per-device control.
- **New-email verification before account email change.** The rationale is to verify the new identity field before switching it.
- **Plain-language privacy policy.** It names permitted aggregate operational metrics and excludes per-bird interaction data.
- **Recoverable deletion window followed by hard deletion and key destruction.** The user can recover while the account is marked recoverable; after the deadline, owned rows, objects, sessions, jobs, logs, and keys are removed.
- **Deletion test enumerating every owned table and object-store prefix.** The rationale is to prove deletion covers all account-owned data and storage.

### API contracts and command handling

- **Versioned JSON over HTTPS.** Versioning supports client/server schema compatibility, and HTTPS protects authenticated and visitor sessions.
- **Separate host and visitor credentials.** Host endpoints use account sessions; visitor endpoints use scoped read-only sessions so visitor credentials cannot reach write endpoints.
- **Server-derived identity from credentials.** The plan rejects account IDs submitted as authority, preventing a client-provided ID from choosing the account.
- **Structured error codes with matter-of-fact copy.** Expired auth, unavailable visits, version mismatches, and temporary sync failure stay direct and in the system voice.
- **Auth link request, redeem, and logout endpoints.** They request and atomically consume 15-minute links, create or revoke sessions, and rate limit without revealing account existence.
- **Snapshot endpoint.** It returns private presentation projection, clock anchor, and revision for first load, returns, keepalive, and recovery while never exposing vectors.
- **Arrival endpoint.** On foreground return, the server derives absence, chooses one primary greeter from boldness, warmth, mood, and absence, debounces focus flaps, and forbids visitor calls.
- **Events endpoint.** It batches presence, listen-in, offers, settle, and undo while validating ownership, timing, schema, idempotency, sequence, and rate limits.
- **Notebook endpoint.** Keyset pagination supports a stable immutable reverse-chronological notebook with no edit/delete endpoint.
- **Rename and adopt endpoints.** Rename is host-only; adoption checks age and seven-bird cap atomically, creates a new stable bird, and never replaces an existing bird.
- **Account settings, sessions, email-change, export, delete, and recover endpoints.** These expose matter-of-fact account/accessibility controls, verification, short-lived export delivery, device revocation, and deletion window handling.
- **Invitation endpoints.** They let the host create a named email invite, see outstanding grants and a quiet visit log, and revoke grants without default notifications.
- **Visitor redeem and snapshot endpoints.** Redemption proves possession of a bearer link, creates a scoped read-only session, rechecks revocation on every snapshot, and logs approximate duration without presence events.
- **Bearer visitor links with hashes, no referrers, and forwarded-link warning.** The rationale is that possession of the link is the capability, so links are hashed, redacted, protected by `Referrer-Policy: no-referrer`, and described plainly in settings.
- **One redeemed visit session per invite.** A redeemed link cannot create another visit session, limiting access from the one-use grant.
- **Same "visit no longer available" surface for revoked or expired invites and revoked active visits.** The plan keeps failure clear without leaking extra distinction.
- **Visitor snapshot intervals capped at 15 seconds.** The stated why is revocation latency: at most one pull interval plus request latency.
- **Visitor projection excludes notebook, settings, export, and interaction controls.** Visitors hear and see the actual aviary without host controls or a prettified state.
- **Quiet visit log, optional restrained visit email, no push or prompts.** The plan avoids badges, onboarding prompts, and default notifications; explicit opt-in email is rate limited.

### Presence, concurrency, and synchronization

- **Three-condition presence qualification.** Requiring visible document, focus, and recent trusted pointer/key activity lets quiet watching count while hidden, unfocused, synthetic, or idle tabs stop.
- **Thirty-second heartbeats and bounded server intervals.** The rationale is to count only received qualified presence up to the last heartbeat plus grace, not unbounded offline or suspended time.
- **`sendBeacon` as best-effort close path.** Missing close remains bounded by heartbeat expiry, so unload failure does not create unlimited presence.
- **No offline retroactive presence.** The plan prevents clients from claiming time the server did not observe as qualified.
- **Listen-in does not substitute for presence.** The interaction can persist for drift, but it does not bypass the three-condition attention test.
- **Union of overlapping host-device intervals.** One minute of overlapping attention counts once across multiple devices.
- **Ordered per-aviary command stream.** Commands from both devices are processed in increasing sequence so state changes are deterministic.
- **Atomic per-bird offer cooldown.** It prevents concurrent devices or retries from accepting the same bird's offer cooldown twice.
- **Transactional tick commit with consumed offset.** A crash before commit replays the uncommitted batch; a crash after commit cannot reapply it.
- **Due tick for every active account even when no browser is connected.** Server time, absence, weather, mood, and drift advance canonically; clients never perform catch-up.
- **UUID sharding, leases, and row locks for due work.** The rationale is to prevent double ticking.
- **Server-side catch-up and lag alerting.** If workers fall behind, the server catches up from the last committed tick and alerts instead of handing catch-up to clients.
- **Snapshot `revision` and `server_now`.** Clients estimate skew, interpolate within a bounded horizon, and request fresh state on returns, wakes, reauth, gaps, or keepalive.
- **Short bounded blend on revision jump.** Presentation remains smooth while preserving actual server mood and perch destination.
- **Retry mutations with the same idempotency key.** Lost responses can be retried without duplicating effects.
- **Matter-of-fact recovery only when state cannot be fetched.** The plan avoids letting stale devices upload absolute state or covering sync failure with ornamental loading UI.

### Simulation engine and content generation

- **Deterministic pure reducer.** Given prior state, elapsed time, ordered events, and engine version, the same tick result supports restart, worker reassignment, and golden replay fixtures.
- **Persisted seeds, last tick, active weather, and engine version.** Persisting these values makes the reducer deterministic across deployments and migrations.
- **Version migrations that transform current vectors in place.** The why is stable UUIDs and perceptual continuity without regenerating birds from an event log.
- **Hidden personality traits normalized to `[0,1]`.** The plan uses versioned mappings, seed distributions, calibration, and export-only raw vectors while hiding trait names and values from the aviary UI.
- **Presence-dominant slow drift.** The plan makes presence the dominant component of every trait so regular quiet attention gradually shapes birds.
- **Daily evidence caps and small coefficients.** A single session stays below perceptible change, regular visits become detectable over time, and long sessions or offer spam cannot saturate traits.
- **Nondecreasing traits and no neglect decay.** Tests must show all traits are nondecreasing, neglect leaves vectors unchanged, and absence never lowers personality.
- **UTC-day evidence counters.** UTC prevents travel or timezone changes from resetting the daily cap.
- **Bounded recent-expression signal separate from vectors.** It can make a long-return scene calmer without making absence a debt or reducing personality.
- **Mood enum with seeded transitions, hysteresis, and dwell times.** The rationale is living variation without one-minute flicker or tab-open resets.
- **Perch-zone preference from mood and boldness.** It gives visible behavioral expression: wary scans from back, content preens, curious investigates, drowsy sits low, and alert responds.
- **Rare deterministic weather windows.** Weather adds local-time variation and affects mood/calls, while rarity keeps it from dominating the scene.
- **Bird-to-bird replies and chorus with timing offsets.** The plan wants relationship-like calls that do not perform in lockstep.
- **Greeting resolution from absence, boldness, warmth, mood, and fresh seed.** Returns feel responsive and varied without delaying first render or adding return text.
- **Offer resolution from mood, curiosity, boldness, and offer type.** The resulting approach, hesitate, ignore, join, quiet, call-against, drink, bathe, or watch cue is a bird reaction rather than a success/failure score.
- **Song fragments as bundled symbolic motif library.** The rationale is to synthesize them through the same audio engine as calls, not audio files.
- **Species motif grammar and per-bird stable signature seed.** Calls remain species- and individual-recognizable while allowing bounded variation.
- **Roughly six species as the exact species count.** NOT RECOVERABLE FROM PLAN
- **Scheduled symbolic call events.** The same call event drives synthesis and captions, so captions are generated from actual motif tokens rather than static text.
- **Perceptual tests for call recognition and repetition.** The plan requires users to distinguish individual birds at two through seven birds and flags uncanny or repetitive calls before rollout.
- **Notebook generation from observation facts.** It prevents fabricated facts and arbitrary session-log narration, supports audit through evidence/template versions, and keeps prose sparse and naturalist.
- **Screen-reader narration from the same fact/prose layer.** The rationale is parity with different cadence: current-state descriptions rather than notebook spam.

### Browser scene, audio, and accessibility

- **Retained Canvas 2D scene graph with DOM controls.** Canvas renders sky, perch planes, birds, and foreground detail; DOM remains for actual controls, captions, narration, and focus.
- **Normalized content coordinates and responsive layout solver.** The scene keeps all two to seven birds and focus targets visible on narrow phones and wide desktops.
- **Separate bird hit regions from artwork.** Touch and keyboard targets remain usable even when silhouettes are small.
- **Compact SVG/shape species assets.** The rationale is a small asset footprint for the browser scene.
- **First frame sampled from persisted motion phase at `server_now`.** Birds resume in mid-motion and never start at frame zero, preserving continuity.
- **Client-only leaves and feathers.** They provide bounded local ornament without entering canonical state.
- **Top bar limited to account, accessibility, notebook, and offer controls.** NOT RECOVERABLE FROM PLAN
- **Top bar fade with visible focus retained.** Chrome recedes during pointer stillness, but keyboard focus indicators stay visible and usable.
- **Hidden-tab pause, resource disposal, and fresh state on return.** The rationale is performance, memory stability, and canonical state refresh while the server continues ticking.
- **Reduced-motion mode with still-pose cross-fades.** It removes micro-motion, flights, parallax, and drifting ornaments without presenting a frozen or empty substitute.
- **One bounded WebAudio context with voice pool, per-bird controls, and master limiter.** The rationale is predictable resource use, polyphony control, and bird-specific gain/pan/filter.
- **Procedural calls from symbolic grammar.** The ambient chorus is independent procedural voices, not stacked loops or recorded fallback.
- **Listen-in gain ramp.** Focused bird gain rises while other birds ramp down only to a nonzero ambient floor, keeping the aviary alive.
- **Autoplay-blocked and WebAudio-failed paths.** The scene renders immediately, captions are enabled while silent, audio unlocks on gesture, and there is no fake playback or blocking modal.
- **Mute setting that does not stop mood or call scheduling.** Silence changes presentation, not canonical bird behavior.
- **Navigable DOM and keyboard model.** Tab order, arrow movement among birds, Enter listen-in, Escape exit, offer and notebook focus restoration make the scene operable without pointing.
- **AA contrast tokens and fixtures.** All copy, captions, and overlays must meet WCAG AA across day, night, and weather states.
- **Slowly refreshed screen-reader scene paragraph.** It gives naturalist current-state descriptions while coalescing updates so `aria-live` never floods the queue.
- **Higher-priority narration for greeting, offer, or settle.** These interruptions are phrased as observations rather than raw state changes.
- **On-demand current description review.** Users can review the scene without exposing mood codes or numeric vectors.
- **Captions near the actual calling bird.** Placement, contour, and timing make captions equivalent to the instantiated call event.

### Budgets, instrumentation, and gates

- **Initial gzipped JS and critical scene budgets.** The rationale is to keep the first bird visible quickly on the agreed mid-tier 4G mobile profile.
- **First bird visible within 500 ms for returning authenticated users.** The plan ties this to the living-continuity goal and says to profile edge delivery, critical JS, and first draw before adding a loading flourish.
- **Sustained 60 fps idle and no retained-heap upward trend.** Long quiet sessions must remain smooth and memory-stable.
- **Frame-time, snapshot, audio, notebook, and hidden-loop resource caps.** These keep rendering, network, audio, virtualized notebook, and background behavior within bounds.
- **CI profiles for cold/warm, two/seven birds, day/night, rain, notebook, audio, and reduced motion.** The rationale is release coverage across the scene states the plan names.
- **Aggregate-only operational observability.** Metrics cover health and budgets but exclude account/bird dimensions, event payloads, vectors, email, and per-account histories.
- **Synthetic probes and deterministic simulation fixtures.** They provide performance and calibration evidence without reading production relationships.
- **Alerts on tick p99, due-tick lag, snapshot errors, and performance regression.** These are the operational failure modes the plan wants surfaced.
- **Consented qualitative research and synthetic traces for product success.** The rationale is to avoid visit-frequency leaderboards and account behavior warehouses.
- **Engine acceptance suite.** It proves bounded drift, idempotency, presence dominance, no neglect decay, mood persistence, stable identity, weather rarity, cooldown, and no eighth adoption.
- **Sync/security acceptance suite.** It proves two host devices plus visitor behavior, overlap unioning, retry once, stale-device rejection, visitor forbiddance, revocation, auth replay failure, verified email changes, and deletion.
- **Experience acceptance suite.** It proves varied greetings, mid-motion first frame, seven-bird call distinction, listen-in mix, and sparse true naturalist notebook entries.
- **Access/performance acceptance suite.** It proves contrast, keyboard, assistive-technology, reduced motion, WebAudio-denied captions, soak, 4G first-bird, and bundle checks.

### Build sequence and rollout

- **Foundation milestone.** It exits only when a return visit paints existing birds within budget without entry animation or spinner, matching the continuing-life intent.
- **Canonical engine milestone.** It gates on replay/idempotency tests and synthetic traces whose vectors never fall on absence.
- **Expressive surface milestone.** Species, chorus, listen-in, offers, settle, greetings, notebook, and day/night are tuned through repeated-call listening and drift-calibration studies before production histories accumulate.
- **Equal access and account controls milestone.** Narration, captions, reduced motion, keyboard/focus/contrast, export, deletion, device revocation, and invitations are launch requirements.
- **Private beta beginning with two birds.** It starts production aviaries small while synthetic aged accounts exercise birds three through seven.
- **Staged rollout of adoption capacity.** Third, then five, then seven birds roll out only after distinct-call recognition, layout, audio CPU, and accessibility checks pass at each count.
- **Rollback switches for audio grammars, presentation versions, and adoption capacity.** The plan wants reversible rollout for presentation risk while never rolling back persisted bird IDs or vectors.
- **Explicit tested vector transforms for calibration changes.** If calibration changes, vectors migrate intentionally rather than resetting personality.

### Risk register and release definition

- **Drift calibration controls.** Versioned coefficients, daily caps, synthetic traces, monotonic tests, and blinded comparisons mitigate drift that is too fast, too slow, or punishing absence.
- **Presence inflation controls.** Three browser signals, heartbeat bounds, server clipping, and UTC union mitigate background tabs and concurrent devices.
- **Personality-loss controls.** Server-only writes, ordered offsets, atomic commits, stable UUIDs, restore drills, migration checksums, and reversible deploys make "any vector reset" a stop-ship incident.
- **Call distinctiveness controls.** Stable signatures, bounded variation, collision analysis, listening tests, and chorus/polyphony tuning mitigate canned or indistinct calls by bird seven.
- **First-frame controls.** Inline projection, tiny critical path, phase-anchored motion, quiet-field recovery, and synthetic first-bird gates mitigate the "software starting" feeling.
- **Browser audio permission controls.** Gesture unlock, captions while blocked, unobtrusive settings, and browser policy tests mitigate blocked immediate sound.
- **Accessible-character controls.** Designing narration, captions, and reduced motion alongside the base renderer, testing combined preferences, and requiring parity studies mitigate accessible mode losing the aviary's character.
- **Invitation safety controls.** Hashed one-use tokens, encrypted recipient data, read-only credentials, per-pull revocation, no visitor event ingestion, quiet log, and explicit notifications mitigate access leakage and host-state influence.
- **Analytics privacy controls.** Separate credentials/network paths, allowlisted aggregate metrics, CI checks, deletion inventory, and security review mitigate erosion of the privacy boundary.
- **V1 completeness definition.** Release is complete only when two devices see the same continuing birds, a visitor sees only the canonical scene, absence causes no harm, and every sensory path offers a living aviary within budget.
