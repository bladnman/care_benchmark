## System-level intent

- **The aviary is meant to be "observational and quiet."** This appears directly in "Product scope and invariants" and shapes the negative space of the product: "no score, streak, achievement, hunger, distress, punishment for absence, public discovery, native client, multi-aviary account, user-arranged perches, shared aviary, or push engagement loop." It also shows up in the client plan through a "quiet foreground/background," a "subdued palette," sparse top bar, no scene controls, no textual "welcome back" surface, no intrusive audio prompt, and no launch "progress counters or notification loops."

- **Canonical truth belongs to the server, while the client renders, interpolates, and emits events.** The plan states that "the server is the only writer of canonical bird personality and mood," that clients "submit events, never state replacements," and that the simulation worker is the "sole writer" of moods, personality vectors, positions, call schedules, ambient state, and notebook observations. This appears again in the reducer shape, the no "client-to-client replication or last-write-wins merge" rule, and the client pipeline where persistent decisions stay in the simulation service.

- **Host activity is carefully separated from visitor observation.** The plan repeatedly says only "the host's qualifying presence and interactions affect the host's simulation," while visit sessions are "read-only" and "never count as presence." Visit credentials cannot call host mutations, visitor attention cannot enter the host event stream, and optional visits are kept behind explicit one-time invitations, revocation, and read-only claims.

- **Presence should count quiet watching but not abandoned tabs.** This shows up in the invariant that presence requires "visible document, focused window, and recent pointer or key activity simultaneously." The risk section phrases the calibration goal as making "quiet watching counts but abandoned tabs do not." Server-derived presence windows, heartbeat validation, gap clamping, and activity timeout all support that intent.

- **Absence must not punish the user or harm the birds.** The plan says "absence creates no negative trait delta and no distress state," "closing without settle has no penalty," and "tab close and settle both end presence without penalty." The simulation section says "lack of events does not lower any value" and that "quieter after absence" must come from current time, mood, and lack of fresh greeting cues, "never a negative personality delta or distress mechanic."

- **Development is slow, monotonic, age-paced, and not engagement-gated.** The plan says personality changes are "slow and monotonic toward expressive on positive signals," that growth to seven happens through "age-paced offers," and that offers must never depend on "visit frequency, attention totals, or spending." The simulation calibration asks for measurable drift around one week and perceivable differences around three weeks, "without exposing numbers or introducing click-to-progress loops."

- **Bird identity is stable, while names and visible state are presentation.** The plan says "bird IDs are stable across rename, sync, and migrations" and "names are presentation fields." Snapshots expose only stable ID, name, species, visible pose/perch/mood representation, call schedule, and transition hints. This also motivates stale event handling: a stale event remains against a stable bird ID and cannot overwrite a newer vector.

- **Personality values are categorically hidden from interactive surfaces and export contracts.** The plan says to "keep personality values off all interactive client surfaces" and says numeric vector values are "categorically hidden." The data model says never expose the vector in snapshot or UI DTOs, and the export section follows the stronger "hidden-vector invariant" by excluding numeric personality values even though the export section has a conflict to resolve.

- **Privacy and data minimization are service-boundary requirements, not polish.** This appears in the synthetic UUID, encrypted email, "never put email in IDs, partitions, logs, traces, or analytics," private/no-store snapshots, aggregate-only telemetry, no account or bird IDs in metrics, no telemetry access to the simulation database, and the rule that per-account events and simulation state never enter "aggregate telemetry, recommendation, or training pipelines."

- **The product voice is split between naturalist observation and matter-of-fact account surfaces.** The plan asks for "naturalist lowercase, present-tense prose for bird observations" and "direct matter-of-fact copy for identity, settings, errors, accessibility settings, and sync." The same split appears in notebook templates, captions, screen-reader prose, expired/revoked invite copy, unsupported-browser surfaces, and account/error/sync copy.

- **Accessibility is part of v1, using the same facts as the scene.** The plan calls narration, captions, reduced-motion, keyboard navigation, contrast, WebAudio graceful silence, notebook, and account settings "launch requirements, not a later phase." Screen-reader prose, captions, and notebook copy all come from the same snapshot/event facts and motif parameters, not personality data, coordinates, or generic logs.

- **Performance and bounded scale are release gates.** The plan makes budgets "release gates," caps supported calls at seven signatures, asks to test at seven birds, bounds audio nodes and client objects, and says to pause population expansion if budgets regress. This appears in the architecture, render pipeline, audio graph, observability, rollout, and risks.

## Per-feature whys

### Product scope and invariants

- **Browser-only, single-user aviary per account:** The plan connects this scope to keeping the experience "observational and quiet" and avoiding "public discovery," "native client," "multi-aviary account," "shared aviary," and a "push engagement loop."

- **Two system-selected starting birds:** The rollout rationale is to "tune stability and recognizability at two birds" before broader v1 and population growth.

- **User naming of the starting birds:** NOT RECOVERABLE FROM PLAN

- **Growth to seven birds through age-paced offers:** The rationale is to avoid growth through "visit frequency, attention totals, or spending," avoid "click-to-progress loops," and only expand toward the seven-bird cap after "call intelligibility, render, audio-mix, and memory budgets hold at each count."

- **No score, streak, achievement, hunger, distress, punishment for absence, public discovery, native client, multi-aviary account, user-arranged perches, shared aviary, or push engagement loop:** These exclusions preserve the "observational and quiet" experience and keep absence from becoming a penalty.

- **Server-only canonical bird personality and mood:** The rationale is to keep clients from replacing state: "Clients submit events, never state replacements," and the simulation worker is the "sole writer" of canonical moods, personality vectors, positions, call schedules, ambient state, and notebook observations.

- **Host-only qualifying presence and interactions:** The rationale is to make only the host's qualified activity affect the host simulation. Visit sessions are "read-only and never count as presence."

- **Presence requiring visible document, focused window, and recent pointer or key activity:** The rationale is that "tab-open time alone does not count" and the risk section says calibration should make "quiet watching counts but abandoned tabs do not."

- **Slow monotonic personality changes toward expressive:** The rationale is that positive signals should accumulate slowly while "absence creates no negative trait delta and no distress state."

- **Stable bird IDs across rename, sync, and migrations:** The rationale is that names are only "presentation fields," stale events still target stable bird IDs, and migrations must "preserve vector and bird IDs across every version."

- **Maximum of seven distinct call signatures:** The rationale appears in supported scale and release gates: the audio mix, call intelligibility, render, and memory budgets must hold from two through seven voices.

- **No recorded-call fallback assets:** The plan's articulated rationale is to preserve the procedural call system: calls are synthesized from motif libraries, captions come from the same motif parameters, and poor signatures must not be solved "with recordings."

- **No per-account events or simulation state in telemetry, recommendation, or training pipelines:** The rationale is privacy and telemetry isolation; metrics must be aggregate-only and cannot query the simulation database.

- **Naturalist lowercase, present-tense bird observations:** The rationale is product voice for observations, notebooks, captions, and screen-reader updates.

- **Matter-of-fact identity, settings, errors, accessibility settings, and sync copy:** The rationale is to keep account, error, sync, expired/revoked invite, and unsupported-browser surfaces direct rather than naturalist.

### Architecture and ownership

- **Small web client, authenticated application API, and server-side simulation service:** The rationale is separated ownership: the web client renders/interpolates and reports events, the API validates and serializes snapshots, and the worker owns canonical simulation state.

- **Transactional relational store plus durable per-aviary event queue/log:** The rationale is ordered event consumption, atomic state plus cursor commits, safe retries, and no last-write-wins replacement.

- **Minute tick scheduler for every aviary, including inactive accounts:** The rationale is that mood and ambient state continue while "no client is connected" and are not reset by tab opens.

- **Per-aviary write serialization and synthetic account/aviary partitioning:** The rationale is to serialize writes per aviary and avoid duplicate or lost events across devices.

- **UTC times plus account IANA timezone:** The rationale is local day/night and mood inputs, and the risks call out daylight-saving changes and device timezone changes.

- **Identity/account service boundaries:** The rationale is privacy and authorization: synthetic UUIDs, encrypted email, magic links, device sessions, email verification/change, deletion lifecycle, and export authorization stay in one service.

- **Aviary API authorization, snapshot serialization, event validation, invitations, notebook, and settings:** The rationale is role-specific enforcement, especially so "visit credentials cannot invoke host mutations."

- **Simulation worker as sole writer of canonical moods, vectors, positions, call schedules, ambient state, and notebook observations:** The rationale is to keep persistent world decisions canonical, deterministic, and outside the client.

- **Web client limited to snapshots, render/interpolation, ephemeral audio/render state, and qualifying events:** The rationale is that it "does not simulate the persistent world or compute drift."

- **No bird-state payloads in shared public caches:** The rationale is privacy; authenticated state is delivered through a private/no-store bootstrap while only static assets are publicly cached.

- **Private/no-store bootstrap snapshot or personalized HTML response:** The rationale is to meet first-render goals without public caching of bird state.

- **Quiet field instead of spinner when no authenticated snapshot is quickly available:** The rationale is to preserve the quiet scene and then place birds into "already-progressing poses" once the snapshot arrives.

### Data model

- **Account with synthetic UUID, encrypted verified email, timezone, deletion timestamps, and preferences:** The rationale is stable private ownership with email kept out of IDs, partitions, logs, traces, and analytics.

- **One active aviary per account:** NOT RECOVERABLE FROM PLAN

- **Magic-link authentication:** NOT RECOVERABLE FROM PLAN

- **Fifteen-minute, single-use magic links:** The plan states the behavior but does not articulate why fifteen minutes specifically. NOT RECOVERABLE FROM PLAN

- **Per-device sessions with individual revocation:** The rationale is account/session control and explicit session revocation.

- **Pending email changes until new-address verification:** The rationale is verified email control before changing the account email.

- **Aviary record with canonical version, last tick time, local-time configuration, next offer threshold, and event cursor:** The rationale is versioned snapshot sync, local-time simulation, age-based offers, and ordered event processing.

- **Bird record with stable ID, species, user name, adoption time, server-only normalized personality vector, persisted mood, perch/pose, motif-library key, call schedule/seed, and last update:** The rationale is to separate stable identity and visible presentation from server-only personality and deterministic call identity.

- **No personality vector in snapshot or UI DTOs:** The rationale is the "hidden-vector invariant" and keeping personality values off interactive client surfaces.

- **Immutable InteractionEvent with idempotency key and monotonic per-aviary sequence:** The rationale is safe retries, ordered simulation, and prevention of client trait deltas.

- **Server receive time as canonical and client event time for diagnostics only:** The rationale is that "client timestamps never order canonical writes."

- **Presence windows derived by the server:** The rationale is to prevent "offline/backdated accumulation," clamp gaps, and close presence on timeout, settle, revocation, or tab close without penalty.

- **NotebookEntry generated by simulation from noteworthy observations:** The rationale is that notebook entries are naturalist observations, not visit-frequency entries or user streaks.

- **Read-only, indefinitely scrollable notebook while the account exists:** The plan explains read-only generation by simulation; the rationale for indefinite scrollability itself is not separately articulated. NOT RECOVERABLE FROM PLAN

- **VisitInvitation and VisitSession records:** The rationale is explicit visitor access with encrypted invited email, hashed opaque token, expiry/revocation, first-use tracking, and a host on-demand visit log.

- **Unused invites expiring after 30 days:** The plan states the duration but does not articulate why 30 days specifically. NOT RECOVERABLE FROM PLAN

- **No visitor attention recorded as host presence:** The rationale is that visitors are read-only and never affect host simulation.

- **Idempotency and outbox cursor retention:** The rationale is to "make retries safe" and allow compaction without an "indefinitely growing analytics copy of interaction events."

- **Export excluding numeric personality values:** The rationale is the stronger "hidden-vector invariant" and the conflict between export asks and hidden vectors.

- **Export including bird identity, names, species, visible mood/state, notebook, and settings:** The rationale is to provide an account/aviary snapshot while keeping numeric personality hidden.

- **Deletion with immediate mark, signed-in recovery for 30 days, then record removal:** The rationale is reversible deletion followed by removal of account, bird, event, notebook, session, invitation, and telemetry-linked records.

- **Export generated on demand and sent as a time-limited download link to verified email:** The rationale is authorized, verified delivery of export data.

### API surface and authorization

- **Versioned JSON APIs over TLS with short-lived session credentials, schema validation, rate limits, and stable error codes:** The rationale is authenticated, validated, rate-limited API behavior with predictable errors.

- **Idempotency keys for all mutation endpoints:** The rationale is retry safety and duplicate acknowledgement.

- **Snapshot and notebook cursors carrying canonical version/cursor values:** The rationale is canonical sync; "client timestamps never order canonical writes."

- **Uniform response to magic-link requests:** The rationale is "to avoid account enumeration."

- **Atomic magic-link consume creating a per-device session:** The rationale is single-use consumption and session creation as one operation.

- **Account sessions, email-change, verification, export, deletion, and recovery endpoints:** The rationale is account/session management, verified email change, on-demand export, reversible 30-day deletion, and signed-in recovery.

- **Aviary snapshot endpoint with version, server time, timezone-derived phase, weather, scene transitions, visible bird state, motif parameters, and transition hints:** The rationale is to give the client current renderable state without personality numbers or internal event history.

- **Snapshot pulls on open, visibility return, long frame gap, and low-frequency visible keepalive:** The rationale is to reconcile client view with canonical state while preserving low-frequency sync.

- **Aviary events endpoint with typed events and unique IDs:** The rationale is event validation/append, ordered worker application, and duplicate IDs returning the original acknowledgement.

- **Offer cooldown, one active listen-in target, five-second settle/undo window, and presence-condition validation:** The rationale is to enforce service-side interaction rules before simulation.

- **API acknowledgement meaning accepted, not already simulated:** The rationale is separation between event acceptance and worker application.

- **Notebook pagination in reverse chronological pages:** The rationale is sparse, read-only observation retrieval.

- **Bird rename changing only display name:** The rationale is stable bird identity: names are presentation fields.

- **Server-generated bird adoption offers capped at seven:** The rationale is age-based population growth and the seven-bird support cap.

- **Visit invitations requiring a specific visitor email and one-time opaque link:** The rationale is explicit invite-only access, with default off because no invite exists until sent.

- **Host visit list and visit log with visitor email, date, and approximate duration:** The rationale given is host on-demand visibility into outstanding invitations and visits.

- **Revocation of unused or active invites:** The rationale is terminating visit access, with revocation checked on every snapshot pull.

- **Read-only visit snapshot endpoint:** The rationale is that visit authorization cannot call events, settle, notebook mutation, or host account APIs.

- **No client-to-client replication or last-write-wins merge:** The rationale is that server-applied events against stable bird IDs cannot overwrite newer vectors.

- **Matter-of-fact expired-session and invalid-target errors:** The rationale is product voice for errors and stale clients.

### Simulation engine

- **Deterministic reducer from canonical state, ordered events, tick time, and seeded RNG:** The rationale is replayable, ordered simulation with a consumed cursor.

- **Simulation version and RNG seed storage:** The rationale is to replay fixtures and migrations "without changing bird identity."

- **Bounded work per tick and capped deterministic outage catch-up:** The rationale is to avoid "a huge catch-up jump" after outages.

- **Presence heartbeats only while visible, focused, and recently active:** The rationale is to count qualified watching and reject tab-open time alone.

- **Server-configurable activity window and heartbeat cadence:** The rationale is calibration, including a "few minutes" window where quiet watching counts but abandoned tabs do not.

- **Presence duration as dominant drift input:** The rationale is that qualifying host presence should be the main positive signal.

- **Listen-in as a smaller focused-bird signal:** The rationale is to nudge the focused bird's warmth/vocal frequency without making listen-in the dominant input.

- **Offers as small trait-specific signals:** The rationale is to nudge curiosity and related traits without creating click-to-progress loops.

- **Settle ending presence and quieting mood without a trait direction:** The rationale is that settle is a quieting interaction, not personality progress.

- **Per-trait low-pass accumulated evidence and small bounded positive deltas:** The rationale is slow, nonnegative, monotonic movement that prevents one session from producing a perceptible step.

- **No lowering values when events are absent:** The rationale is absence without punishment, negative trait deltas, or distress.

- **Offline/replay calibration for one-week instrument drift and three-week perceivable differences:** The rationale is to make drift neither inert nor too fast while avoiding exposed numbers and click-to-progress loops.

- **Quieter after absence derived from time, mood, and lack of fresh greeting cues:** The rationale is to avoid negative personality deltas or distress mechanics.

- **Persisted bird mood with transitions and timers:** The rationale is mood that survives sessions and advances while no client is connected.

- **Minute tick considering recent host events, local time, rare weather, neighboring calls/alarm events, and personality bias:** The rationale is ambient behavior driven by canonical facts rather than client-open state.

- **Daily-ish reconciliation/reset as bounded mood transition:** The rationale is that mood should not reset "just because a tab opens."

- **Night settling most birds while a nightjar-like species may remain active:** The plan states the behavior but does not articulate why that species distinction is needed. NOT RECOVERABLE FROM PLAN

- **Rare, soft, short-lived weather:** The rationale is ambient variety that briefly damps calls or shifts alertness without becoming a dominant mechanic.

- **Aviary age alone scheduling additional bird offers:** The rationale is no visit-count or interaction threshold.

- **Roughly six coherent species:** NOT RECOVERABLE FROM PLAN

- **Motif libraries and stable signatures:** The rationale is recognizable call identity.

- **Server-provided timing, mood, and trait-shaped motif parameters and call scheduling:** The rationale is that client synthesis can vary pitch, envelope, spacing, and motif combinations while preserving recognizable identity.

- **Notebook observations only for meaningful moments:** The rationale is to avoid generic event logs, numerical drift, user activity counts, and unconstrained generated claims.

- **Reviewed naturalist templates tied to facts:** The rationale is grounded notebook copy about facts such as which bird greeted first, weather, perch, and calls.

### Client render and interaction pipeline

- **Single responsive horizontal scene with stable framing for all birds:** The rationale is a consistent quiet scene that can support every bird count without panning, zoom, or dragging.

- **Three perch zones:** NOT RECOVERABLE FROM PLAN

- **Quiet foreground/background, subdued palette, and local-time day/night color:** The rationale is the observational and quiet field feeling.

- **Rare ambient leaf/feather motion:** NOT RECOVERABLE FROM PLAN

- **No panning, zoom, bird dragging, badges, hover labels, or controls over the scene:** The rationale is to keep the scene observational and not turn birds into direct-manipulation or progress surfaces.

- **Sparse top bar above the scene:** The rationale is to keep account/settings, accessibility settings, notebook, and offer controls available without placing controls over the scene.

- **Top bar fading after cursor stillness and returning on pointer or keyboard activity:** The rationale is to keep the field quiet while preserving access when the user becomes active.

- **Private bootstrap snapshot, current pose/time initialization, first bird before noncritical assets, interpolation, and version reconciliation:** The rationale is to make the aviary appear already alive and keep the client aligned to canonical snapshots.

- **Immutable view snapshots plus ephemeral interpolation state in the renderer:** The rationale is that persistent decisions and transitions belong in the simulation service.

- **Compact SVG/vector or procedural bird assets and efficient Canvas/SVG composition:** The rationale is performance: cap draw work, reuse objects/buffers, stop rendering when hidden, and still let server ticking continue.

- **Avoiding fade-from-static or spinner:** The rationale is that the opening should not feel static; if needed, draw the quiet field and then place birds into already-progressing poses.

- **Soft fly-in for the first adopted bird:** The plan states the behavior but does not articulate why the first adopted bird specifically flies in. NOT RECOVERABLE FROM PLAN

- **Mood-shaped idle motion:** The rationale is to show wary, content, curious, and drowsy states through observation rather than labels.

- **Return greeting selected from boldness, mood, and absence with staggered second response:** The rationale is procedural variation without a textual "welcome back" surface.

- **Listen-in as gradual mix rebalance, never silencing other birds:** The rationale is focused listening while preserving the broader aviary sound.

- **Listen-in disengaging on refocus, second bird, empty scene click, or keyboard focus departure:** The rationale is to keep one active listen-in target and make focus changes end the mode.

- **Offers originating from the top-bar affordance:** The rationale is to avoid scene controls.

- **Seed/song-fragment/still-pool offer choices:** NOT RECOVERABLE FROM PLAN

- **Mood/personality-shaped offer reactions:** The rationale is to keep reactions tied to canonical mood and personality while personality values remain hidden.

- **Settle warming evening light and quieting calls:** The rationale is to end presence and gently quiet mood without adding a trait direction.

- **Scene click within five seconds undoing settle:** The rationale is an undo window matching the service validation.

- **Tab close and settle ending presence without penalty:** The rationale is no punishment for absence or closing.

- **Reduced-motion still-pose cross-fade mode:** The rationale is to honor prefers-reduced-motion and in-product settings while retaining the same moods, calls/captions, interactions, and notebook.

- **State changes and narration independent of frame rate:** The rationale is accessibility and simulation consistency even when motion is reduced or rendering differs.

### Audio and captions

- **One bounded WebAudio graph per active browser session:** The rationale is bounded client resource use and clean session disposal.

- **Client-side synthesis from compact motif libraries:** The rationale is procedural audio with no recordings or fallback assets.

- **Deterministic per-bird seeds plus variation per call:** The rationale is that signatures persist while exact calls do not repeat.

- **Timing and pitch shaped by server schedule, mood, and personality:** The rationale is to align audible behavior with canonical simulation state.

- **At most seven voices with a chorus bus and soft ambient bed:** The rationale is the seven-bird support cap and bounded audio mix.

- **Slow listen-in gain ramp and reverse curve on disengage:** The rationale is smooth focus without silencing other birds.

- **Settle gradually reducing call activity and master level:** The rationale is quieting the aviary gently.

- **Reusing oscillators/buffers, disposing scheduled nodes, and closing contexts:** The rationale is no client memory growth and bounded audio resources.

- **Captions generated from the same motif parameters actually synthesized:** The rationale is caption accuracy: captions describe the actual procedural motif.

- **Captions positioned near the calling bird and faded with the call:** The rationale is tying text to the audible/visual calling bird.

- **Captions enabled by default when WebAudio is unavailable or denied:** The rationale is graceful silence and browser autoplay/audio policy handling.

- **Resume WebAudio on first permitted interaction or explicit audio preference action:** The rationale is to handle browser autoplay policies while avoiding a welcome/toast announcement.

- **Making browser-policy limitation an acceptance and design calibration item:** The rationale is to avoid silently shipping repeated audio failures.

### Accessibility, account surfaces, and voice

- **Semantic scene model alongside the visual renderer:** The rationale is accessible representation of the same scene facts.

- **Keyboard order of top bar, first bird, arrow navigation, Enter listen-in, Escape exit, and operable controls:** The rationale is keyboard-alone operation for scene and controls.

- **Visible high-contrast focus outlines and WCAG AA contrast across day/night backgrounds:** The rationale is accessible visibility across changing backgrounds.

- **Screen-reader prose from snapshot/event facts rather than coordinates, status labels, or personality data:** The rationale is naturalist narration grounded in the same facts as scene and notebook while keeping personality hidden.

- **Naturalist updates at roughly 30-60 second idle cadence:** The rationale is restrained narration that does not overwhelm assistive technology.

- **Priority updates for user-initiated greeting, offer reaction, and settle:** The rationale is to surface user-initiated changes without making every event noisy.

- **Queueing and coalescing screen-reader updates:** The rationale is preventing bursts from overwhelming assistive technology.

- **Call captions in the same naturalist voice as generated audio parameters:** The rationale is consistency between procedural calls, captions, and voice.

- **Matter-of-fact account, error, sync, accessibility-setting, invite, and unsupported-browser copy:** The rationale is the product-voice split between bird observations and account surfaces.

### Performance, observability, and privacy

- **Initial JS under 2 MB gzipped, first bird under 500 ms, 60 fps idle, no memory growth, and tick latency p99 below alert threshold:** The rationale is that these budgets are "release gates."

- **Code-splitting account/accessibility settings, invitation, and notebook routes:** The rationale is to load scene-critical assets first and lazy-load noncritical UI.

- **Benchmarking signed-in snapshot path:** The rationale is to measure the real private snapshot path, "not just a public cache path."

- **Synthetic browser runs from common geographies and aggregate-only RUM:** The rationale is to observe navigation, first-bird render, frame times, audio-context errors, snapshot/tick latency, API errors, and memory trend without per-account detail.

- **Stripping account IDs, bird IDs, per-bird payloads, presence intervals, and interaction detail before metrics leave product services:** The rationale is telemetry privacy and isolation from simulation data.

- **Never dimensioning operational telemetry by account:** The rationale is privacy; telemetry pipelines cannot query the simulation database.

- **Avoiding product analytics for visits, drift, session frequency, or bird behavior:** The rationale is to avoid engagement analytics and protect per-account simulation behavior.

- **Operational logs with request correlation IDs, bounded retention, and no email or bird prose:** The rationale is operational debugging without personal or generated-content leakage.

### Rollout and release sequence

- **Foundation first:** The rationale is to establish schema and identity boundaries, private snapshot path, sequencing/idempotency, deletion/export lifecycle, replay fixtures, telemetry redaction, and service/privacy review before real accounts.

- **Two-bird private alpha:** The rationale is to tune stability and recognizability at two birds and verify client/server state boundaries.

- **Accessible v1 surface in the same launch cohort:** The rationale is that narration, captions, reduced-motion, keyboard navigation, contrast, WebAudio graceful silence, notebook, and account settings are "launch requirements, not a later phase."

- **Broader v1 enabling offers, settle, sparse notebook generation, invitations/visit log, and revocation:** The rationale is to validate read-only authorization and ensure visitor sessions cannot alter host state.

- **Age-based population ramp:** The rationale is to start all accounts with two and expand toward seven only after call intelligibility, render, audio-mix, and memory budgets hold at each count.

- **Offer schedule keyed only on aviary age:** The rationale is to avoid engagement gates and "telemetry-derived user segments."

- **Server flags and rollback for tick/model versions and optional social entry points:** The rationale is controlled rollout and rollback.

- **Backward-readable migrations preserving vectors and bird IDs:** The rationale is stable identity across versions.

- **Day-one operational measures as aggregate service health only:** The rationale is privacy and avoiding account-level telemetry.

- **No user-facing progress counters or notification loops during launch:** The rationale is to keep launch aligned with the observational, quiet experience.

### Main risks and mitigations

- **Drift calibration with deterministic multi-week replay scenarios and blinded qualitative review:** The rationale is to keep drift from feeling inert or too fast while protecting monotonicity and no-negative-absence behavior.

- **Per-aviary ordering, idempotency keys, committed cursors, retries, and replayable tick inputs:** The rationale is to avoid duplicate or lost events across devices without client absolute-state writes.

- **Explicit timezone update semantics:** The rationale is to avoid mood or local-time discontinuities across daylight-saving and device timezone changes.

- **Auditioning mixes from two through seven voices:** The rationale is to keep calls from sounding synthetic, repetitive, or indistinguishable across supported bird counts.

- **Captions and first-gesture resume for autoplay restrictions:** The rationale is to make opening silence expected and recoverable without an intrusive prompt.

- **Screen-reader and keyboard reviews for every interaction:** The rationale is to keep accessible surfaces from regressing or becoming noisy.

- **Aggregate technical heartbeat validity instrumentation:** The rationale is to calibrate presence without account-level attention metrics.

- **Hashed opaque invite tokens, read-only claims, expiry, revocation checks, and rate-limited consumption:** The rationale is to reduce visitor token leakage or lingering access.

- **Resolving the vector-export conflict before freezing export contract:** The rationale is privacy and consistency with the hidden-vector invariant.

- **Keeping social visited-notification off by default and silent unless explicitly opted in:** The rationale is to preserve the default quiet/social-off posture while honoring the specific social exception only with explicit opt-in.

- **Testing at seven birds, bounding audio nodes and client objects, and pausing population expansion on budget regression:** The rationale is to prevent performance degradation with growth.
