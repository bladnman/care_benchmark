## System-level intent

- **Server-owned canonical state over client authority.** This shows up as an acceptance contract in "clients never advance simulation" and "the server is the sole writer of canonical vectors"; it recurs in "one canonical transaction boundary," "server UTC timestamps," the append-only event model, durable ticks, and the ready-to-ship requirement that "the server alone advances persistent identity, vectors, mood and calls."
- **A calm naturalist aviary, not a game or social network.** The plan asks for "naturalist, lowercase, present-tense copy" on bird-facing surfaces, "no textual welcome," a read-only "field notebook," "calm blues, greens, browns and ochres," and non-goals including "leaderboards, scores, achievements, streaks," feeds, comments, chat, public discovery, and co-presence.
- **Non-punitive care and absence.** The plan repeatedly says "birds never punish absence," "no trait moves downward because of neglect," "Verify no-attention periods do not decrease vectors or generate distress," and "Absence yields quieter ambience only, never distress."
- **Slow, monotonic, hidden personality drift.** The experience "accumulate[s] slow, monotonic personality drift"; the UI must "not expose personality values," while the simulation uses "bounded nonnegative deltas," "trait ceilings," and calibration targets of "instrument-level movement after about a week" and "perceivable but subtle expression by about three weeks."
- **Privacy and minimization as structural boundaries.** The plan uses "random synthetic account UUID," encrypted email only on the account/auth record, "aggregate only" telemetry, no traits/interactions/content in analytics or model-training pipelines, and deletion/export coverage for aviary data, vectors, notebook, visits, and telemetry.
- **Accessibility as a launch contract, not an afterthought.** Scope includes "screen-reader narration," "a designed reduced-motion surface," "call captions," "keyboard access," and "WCAG AA contrast at launch"; later sections call these "release criteria for the primary feature set."
- **Already-active, continuous one-screen presence.** The core scene is "a single horizontal, one-screen scene that appears already in motion"; first paint must avoid "spinner, fade-from-static, or wake-up sequence," and birds and ambient activity "should never read as paused."
- **Deterministic, idempotent correctness.** The plan emphasizes ordered events, "idempotency keys," "retry detection," tick workers "safe to retry," deterministic state machines, stable seeds, and fixtures for retry/crash/replay, local-time/DST, and drift calibration.
- **Opt-in, read-only, simulation-neutral visits.** Visits are "opt-in per invite and read-only"; a "visitor cannot influence host state," visitor snapshot access omits controls/notebook/settings, revocation is checked "at every pull," and notification preference is "off by default" and absent from onboarding.

## Per-feature whys

**Scope and product contracts**

- **Web-only delivery**: NOT RECOVERABLE FROM PLAN
- **Single-user-per-account and one canonical aviary**: The plan ties this to "multi-device snapshots," "one canonical transaction boundary," and one durable per-account simulation store so all devices show the same aviary.
- **Two starter birds**: NOT RECOVERABLE FROM PLAN
- **Age-paced additions up to seven**: The plan frames the schedule as "a calibration parameter, not a visit reward," and says seven is a "hard invariant" in service and database, keeping additions out of gamified progression.
- **Email magic-link authentication**: The rationale provided is security/correctness around verified account access: token hashes expire in 15 minutes, become invalid atomically on use, and device sessions are revocable.
- **Multi-device snapshots**: They exist so devices reconcile from a compact canonical snapshot after initial navigation, visibility return, suspension, or long frame gaps rather than forking state.
- **Server-owned simulation**: The stated reason is that clients never advance canonical progression or write vectors; persistent identity, moods, calls, and drift remain authoritative and retryable.
- **Presence accounting**: The plan uses presence as the primary input to drift while guarding against background-tab inflation and preserving "quiet watching" with a calibrated activity window.
- **Offers**: Offers let birds "respond to attention and offers" and provide canonical, mood/personality-dependent reactions that nudge curiosity and boldness without direct client vector writes.
- **Listen-in**: Listen-in is both a client audio mix choice and an attention signal; it chiefly influences the focused bird's "social warmth and vocal frequency."
- **Settle**: Settle "quiets current mood and ends presence without drift," changes lighting/call behavior for the session, and supports undo without making absence harmful.
- **Read-only field notebook**: The notebook is for sparse "noteworthy canonical facts," not raw events, visit frequency, streaks, or a feed.
- **Optional per-invite read-only visits**: The rationale is host control and isolation: visits are opt-in, revocable, expiring, and "simulation-neutral."
- **Screen-reader narration, reduced-motion, captions, keyboard access, and WCAG AA contrast**: These are launch acceptance contracts so the primary feature set works through alternate sensory and interaction modes.
- **Bird-facing naturalist copy and matter-of-fact system copy**: The plan separates an aviary voice from utility surfaces: bird-facing surfaces stay "naturalist, lowercase, present-tense," while sign-in, account, settings, accessibility-settings, and errors stay clear.
- **Explicit v1 non-goals**: The plan uses non-goals to keep scope, data/API surfaces, and acceptance review away from social, gamified, caretaker-punishment, and engagement mechanics.
- **Account state export**: The export is allowed as an account setting so vectors can be downloaded without becoming a product "stats surface."
- **Visit notification preference off by default and absent from onboarding**: The stated reason is that visits are opt-in and notification preference should not become an onboarding engagement prompt.

**Architecture and ownership boundaries**

- **Browser client, authenticated application/API layer, durable simulation store, and background tick workers**: This separation lets the browser present while durable server components own identity, ordering, simulation, and ticks.
- **Split modules for account/auth, aviary state and event ingestion, simulation, notebook, and visit authorization**: The plan says to split by deployment scale while keeping one canonical transaction boundary for aviary state.
- **Relational store as practical default**: The reason given is durable transactions and constraints for account ownership, ordered append-only events, snapshots, invitations, and deletion.
- **Queued or leased due aviaries for tick workers**: The purpose is retry-safe tick execution and scalable background simulation.
- **No client-to-client replication**: The plan grounds this in server-owned canonical snapshots and durable simulation rather than peer state exchange.
- **Server ownership of account identity, bird identity, vectors, moods, timestamps, notebook entries, and invitation status**: This enforces canonical progression and prevents normal clients from authoring hidden state.
- **Client ownership of interpolation, focus, panels, listen-in selection, local audio nodes, and ornaments**: These are ephemeral presentation state that can be discarded or reconciled from a fresh snapshot.
- **Compact render snapshot boundary**: The snapshot gives stable render data, cues, call tokens, environment state, and transitions while withholding numeric personality traits from normal clients.
- **Scene drawing independent from API and audio implementations**: The reason is that scene, API, and audio can all consume the same snapshot without exposing or serializing hidden vectors.
- **Server UTC plus account IANA time zone**: UTC supports ordering, expiration, and durable history; the IANA zone supports local day/night and daily mood timing.
- **Deterministic local-time boundary handling**: The plan calls this out to make daylight-saving changes and ambiguous local times testable and non-authoritative from client wall-clock time.

**Data model**

- **Random synthetic account UUID and encrypted email only on the account/auth record**: The reason is to avoid deriving identifiers, logs, partition keys, or analytics dimensions from email.
- **Device sessions and magic-link token hashes**: These support revocation, limited device labeling, one-time token use, 15-minute expiry, and rate limiting without raw email in logs.
- **Aviary revision, tick cursor, last tick time, presence time, settled state, and local environment state**: These preserve canonical tick progress, snapshot consistency, session presentation, and seeded day/weather transitions.
- **Bird identity, names, species, vectors, mood, perch, pose, call seed, and offer timing**: The model preserves stable identity across renames/migrations, enforces the seven-bird limit, and keeps personality vectors server-only.
- **Append-only interaction events**: The plan uses events for canonical ordering, idempotent retries, bounded payload validation, processing status, and rejecting client-provided vector deltas.
- **Notebook entry records**: Stable IDs, source facts, template versions, and deduplication support immutable, sparse observations instead of a raw event log.
- **Invitation and visit records**: Invite status, expiry, revocation, one-time secrets, and visit duration support opt-in read-only access plus a host settings log without simulation capability.
- **Snapshot/revision recovery state**: Canonical revision and cursors allow consistent pulls and retry detection, while checkpoint state prevents vector history from being reconstructed by clients or lossy analytics logs.
- **30-day account deletion window**: The account becomes inaccessible immediately, can be recovered while signed in, and then hard deletes aviary data, vectors, notebook, visits, and associated telemetry at expiry.

**API surface and flows**

- **Authenticated HTTPS or typed RPC endpoints with strict account scoping**: The stated reason is validation, idempotency, versioned response contracts, and cross-account containment.
- **Auth/session/account endpoints**: These manage magic-link request/consume, per-device sessions, revocation, email changes, export, recovery, and deletion under verified account control.
- **Owner aviary snapshot endpoint**: It returns render state, timestamp, revision, permissions, transitions, and notebook cursor without vector numbers, supporting resume, keepalive, and conditional unchanged responses.
- **Aviary event batch endpoint**: It lets the server assign canonical sequence/time, validate ownership and cooldown, deduplicate retries, and pass accepted events to the tick in order.
- **Presence packets as capped intervals**: The plan uses interval packets only when all required attention signals are present, caps length, and rejects stale, future, or replayed times to prevent arbitrary self-reported hours.
- **Notebook pagination endpoint**: Pagination gives immutable observations while excluding interaction history and visit-frequency surfaces.
- **Visit invitation and read-only snapshot endpoints**: These support named opt-in invitations, one-time exchange, short-lived read-only tokens, revocation/expiry checks, and an unavailable surface on the next pull.
- **Offer submission by seed, song fragment, or still pool rather than bird click**: NOT RECOVERABLE FROM PLAN
- **Server-side per-bird offer cooldown and reaction choice**: The reason is to keep offer pacing calibrated and make reactions come from canonical mood/personality.
- **Listen-in start/end events**: The plan records them as attention inputs while keeping listen-in itself a client audio mix choice.
- **Settle event, tab close timeout, and five-second undo**: These end presence without a required goodbye or penalty and allow a quick scene-click reversal.

**Simulation engine and calibration**

- **Deterministic, idempotent one-minute tick**: The tick makes offline and online aviary progression durable, ordered, retry-safe, and independent of whether a client is open.
- **Bounded catch-up after downtime**: Bounded elapsed-time steps or closed-form transitions prevent a week-old aviary from requiring "a burst of unbounded minute jobs."
- **Three-signal presence collection**: Visible document, focused window, and recent pointer/key activity distinguish real owner attention from unattended open tabs.
- **Server clipping of gaps, overlaps, and simultaneous devices**: The rationale is to count owner attention once while retaining event source attribution for processing.
- **Low-pass personality drift**: Drift is slow, primarily weighted by valid owner presence, nonnegative under absence, and avoids visibly attributable per-session change.
- **Trait effects from listen-in and offers**: The plan links listen-in to social warmth and vocal frequency, accepted offers to curiosity, and making offers near birds to boldness.
- **Drift calibration fixtures**: Deterministic fixtures verify measurable movement after about a week, subtle expression after about three weeks, and no decreases or distress during no-attention periods.
- **Persistent mood enum and seeded mood transitions**: Persisted mood/timers avoid a default-mood snap on open and allow recent interactions, local time, weather, calls, and personality biases to shape state.
- **Bird-to-bird response, wary influence, and chorus emergence**: These create inter-bird behavior without turning birds into independent NPC UI states.
- **Local time, rain, wind, night behavior, and nightjar-like exception**: These produce gradual palette/mood/call changes and "small mood changes, not dramatic events."
- **Procedural call grammar over species motifs**: The rationale is recognizable individual signatures with bounded variation, mood/personality modulation, and no downloaded audio loops.
- **Call captions from exact generated motif/timing**: Captions must describe the actual generated call rather than use a fixed per-bird caption.
- **Owner-return greeting selection**: The plan chooses one likely greeter and staggers others to avoid synchronized greeting cues; quick absences get a glance and long absences may get re-orientation.
- **Notebook generation rule layer**: Rules, deduplication, and a sparsity budget keep observations specific, supported by state, lowercase/present-tense, and not a feed or engagement mechanism.

**Frontend scene and interaction pipeline**

- **Thin top bar plus full single-screen responsive scene**: This keeps controls in the top bar and preserves a clean scene with "no buttons, badges, labels, tooltips, or overlays within the scene."
- **Top bar fading toward transparency**: The rationale is to restore controls on pointer/keyboard activity while letting the aviary scene stay visually primary during stillness.
- **Calm color palette with WCAG AA copy/focus**: The palette supports the calm aviary voice while contrast remains an acceptance requirement in bright and dim scenes.
- **All birds and three perch zones in frame**: The plan requires every supported viewport to show the whole aviary context; users cannot drag/place birds because mood/personality determine perch, pose, and motion.
- **First-paint quiet field, immediate snapshot, and first bird under 500 ms**: The reason is an already-active scene without spinner, static wake-up, or delayed motion on mid-tier mobile over 4G.
- **True new aviary empty state before starter birds enter**: The plan allows this only for a true new aviary, so the brief quiet state represents creation rather than normal loading.
- **Bounded render loop with interpolation from snapshot N to N+1**: The plan uses stable objects, reusable buffers, capped updates, fresh resume snapshots, and interpolation to avoid teleporting and memory growth.
- **Client-side ambient ornaments**: Leaf/feather drift and subtle parallax are independent of tick and not persisted because they are render-only presentation.
- **Continuous idle motion**: Mood-keyed preening, scanning, head tilts, and weight shifts prevent birds and ambient activity from reading as paused.
- **Designed reduced-motion renderer**: Cross-fades, removed drifting leaves, slower color shifts, and preserved calls/captions/mood/drift provide an alternate surface instead of merely stripping animation.
- **Listen-in focus interactions**: Mouse, touch, and keyboard focus start listen-in; disengagement rules and smooth gain changes keep the focused bird raised while others remain audible at ambient level.
- **Offer top-bar flow**: Keyboard-navigable offer types, mood/personality reactions, and cooldowns keep offers accessible and paced.
- **Settle transition and undo**: Settle moves to evening over seconds, quiets calls, persists until re-engagement, and gives a five-second scene-click undo.
- **Notebook behind an icon with paginated scroll**: This keeps the notebook available but read-only and separate from the scene, with indefinite scrolling through paginated fetches.
- **Keyboard model for scene and controls**: Tab, arrows, Enter, Escape, visible focus, and no hover-only affordances make the whole scene and top-bar flows reachable.

**Audio pipeline**

- **WebAudio procedural calls and reusable motif library**: The plan uses oscillators/noise/envelopes and controlled variation to produce bird calls without recorded loops.
- **Per-bird gain nodes feeding master/ambient mix**: Listen-in can ramp focus while non-focused birds remain audible, preserving ambient chorus.
- **Live generated chorus timing**: Multiple generated calls avoid stacked recorded loops and keep chorus tied to vocal frequency and scheduling.
- **Bounded voices, near-future scheduling, and disposal**: These prevent leaks and runaway audio work during hidden state, unmount, account/session changes, and revoked visits.
- **Browser audio activation policy handling**: Creating/resuming context after a permitted gesture keeps audio compliant without delaying the first visual bird.
- **Offer song fragments through the same pipeline**: Reusing procedural or compact motif instructions keeps offers sonically consistent with calls.
- **Settle and night/weather mix influence**: These factors quiet or reshape scheduling/mix in the same way mood and environment affect the aviary.
- **Captions near the calling bird**: Captions fade with sound, derive from the generated grammar, and remain available when audio is disabled.
- **Graceful silence when WebAudio is missing or denied**: The plan prefers silence plus default captions over recorded fallback audio.
- **Aggregate audio-context error reporting**: The rationale is operational visibility without bird/account state in telemetry.

**Accessibility and voice**

- **Semantic scene region and screen-reader running narration**: The narration gives naturalist prose from canonical/render state, not coordinates, trait values, or status labels.
- **Narration cadence and priority**: Idle updates every 30-60 seconds and prioritized greetings, offer reactions, and settle observations avoid flooding the announcement queue.
- **Consistent notebook and narration voice**: The plan keeps both in the same naturalist voice so read-only observations and screen-reader updates match.
- **Accessible names in conventional system voice**: Settings/actions receive clear accessible names, while bird-facing prose stays naturalist.
- **Screen-reader, reduced-motion, keyboard, and contrast testing**: These checks are required because accessibility surfaces are part of the primary feature set and release criteria.

**Privacy, security, and correctness**

- **Owner API account enforcement**: Every owner query checks account ownership to prevent cross-account state access.
- **Hashed one-time auth and visit secrets with expiry/revocation**: Hashing, atomic consumption, 15-minute magic links, 30-day invites, and revocable sessions keep temporary credentials bounded.
- **Event ingestion protections**: Retry, spoofed interval, oversized batch, stale timestamp, cooldown bypass, and cross-account bird ID defenses protect canonical simulation.
- **Visitor token restrictions**: Visitors get only short-lived read-only snapshot access, never controls, notebook, settings, owner presence, or interaction capability.
- **Analytics isolation and aggregate operational telemetry**: The plan allows request counts, latencies, tick health, anonymized session-duration histograms, first-bird/render timing, and audio errors while excluding traits, moods, interactions, IDs, emails, and content.
- **Deletion/export scope and verified email delivery**: Notebook content and visit records are included in deletion/export scope; export is generated on demand and delivered by expiring download link to verified email.

**Performance budgets and observability**

- **Initial JavaScript, first-bird, frame-rate, and memory gates**: These budgets ensure the primary scene is fast, smooth, and stable on mid-tier mobile and older laptops.
- **Kilobyte snapshots, efficient initial HTML/snapshot, lazy-loaded secondary surfaces, and compact assets**: These keep the first scene path light while deferring settings, accessibility settings, invitations, and notebook code when possible.
- **Synthetic browser performance checks and aggregate-only RUM**: The plan measures geography, first-bird timings, frame distribution, memory, latency, audio failures, auth/visit errors, accessibility/performance gates, and queue lag without per-account simulation analytics.
- **Tick latency p99 alarm, canary health, and queue lag**: These expose background simulation health before canonical state freshness falls behind.

**Rollout and delivery sequence**

- **Data ownership, schema/versioning, UUID discipline, event ordering/idempotency, privacy, deletion, and deterministic fixtures before client behavior**: The rationale is that client features depend on correct canonical state and privacy boundaries.
- **Magic-link auth and server-authoritative two-bird aviary before richer scene work**: The plan verifies stable IDs, vector persistence, offline continuity, and no client state writes early.
- **Responsive scene, first-frame path, renderer, idle motion, resume handling, and reduced motion before interaction depth**: The reason is to measure bundle, first bird, frame rate, and memory early.
- **Procedural calls, captions/narration, listen-in, offers, settle/undo, and presence capture after scene foundations**: These features depend on the renderer, audio grammar, and precise presence capture, then feed drift calibration.
- **Notebook, export/deletion/account settings, then invitations/visitor tokens**: The sequence places sparse observations and account data controls before isolated read-only visits and confirms no visit influences host simulation.
- **Keyboard/screen-reader/contrast, browser support, security/privacy review, dashboards, and canary rollout as final gates**: The plan treats these as readiness checks rather than optional polish.

**Risks and mitigations reflected in the plan**

- **Versioned adjustable drift coefficients and deterministic attendance traces**: These mitigate drift that is "too fast, slow, or misleading" without inspecting private interaction analytics.
- **Presence hidden/blur/sleep/wake/keyboard-only/multiple-device tests**: These mitigate presence overcount and undercount while preserving quiet watching.
- **Retry/crash/replay fixtures and recovery checkpoints**: These mitigate lost or duplicated interactions across devices.
- **Persisted moods, transition timers, stable seeds, and local-time/DST validation**: These mitigate mood or offline catch-up snaps.
- **Qualitative listening across the seven-bird ceiling**: This mitigates calls that sound synthetic, repetitive, or lose identity without adding looped recordings.
- **Assistive-technology and reduced-motion review each release**: This mitigates accessibility regressions from animation or copy changes.
- **Acceptance review for non-goals and no unused social/achievement metrics**: This mitigates scope expansion toward game or social mechanics.
