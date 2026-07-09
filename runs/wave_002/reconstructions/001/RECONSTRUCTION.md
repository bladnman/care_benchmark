## System-level intent

- **Attentive observation as the primary interaction.** The plan opens by saying Pocket Aviary's "primary interaction is attentive observation." This shows up again in the refusal of "progress UI, streak, badge, notification loop, or generic event log," in the top bar that fades "almost transparent after cursor stillness," and in the rule that the user "cannot drag birds or issue movement commands."
- **Continuity rather than performance.** The plan names this principle directly. It means "the first useful frame is an already-living aviary," birds keep "stable identities," and "absence never creates punishment." It is carried through server-owned simulation state, the snapshot projection, minute ticks, bounded catch-up, and first-frame rules that avoid a "wake-up entrance."
- **Noticing rather than announcing.** The plan names this directly and grounds it in "no welcome toast, progress UI, streak, badge, notification loop, or generic event log." It also appears in the "quiet in-flow offer," the sparse notebook, no unread visit count, no scene labels, and product prose as "specific, lowercase naturalist observation."
- **A private relationship rather than an engagement system.** The plan names this directly. It shows up in hidden personality data, owner-only simulation-affecting events, visitors that are "read-only and cannot affect it," "aggregate operational telemetry only," and explicit exclusions of public discovery, follows, comments, chat, push/email engagement prompts, and gamification.
- **Server-owned canonical state.** The plan repeatedly separates server authority from browser rendering: "the server owns canonical simulation state," "the server simulation worker is the only actor allowed to mutate canonical bird personality or mood," and the client must "never derive or persist canonical personality/mood."
- **Snapshot projection is for rendering, never truth.** The plan distinguishes "canonical domain state" from "snapshot projection." The projection gives the client enough to paint birds "in media res," but is "never the source of truth"; the client may interpolate, then must request a fresh snapshot after visibility restoration or a long frame gap.
- **Positive, bounded drift without punishment.** The personality model uses "small positive delta," "non-decreasing drift deltas only," and "Neglect adds no negative delta." Absence can create quieter ambient behavior through lack of positive current signals, but may "never lower personality values, create distress, or trigger recovery debt."
- **Qualified presence is evidence, not raw time spent.** Presence exists only while the document is visible, focused, and recently active. The plan rejects background-tab, unfocused-window, idle-laptop, overlapping, and impossible intervals so attentive observation drives drift without visit-frequency reporting or hidden engagement scoring.
- **Accessibility is a designed representation of the same canonical state.** The plan says accessible output is "not a hidden list of engine values." Narration, captions, keyboard focus, reduced motion, and semantic controls are built from render projection and canonical event results, without exposing personality numbers, raw perch indexes, or pose-transition firehoses.
- **Separate naturalist product voice from matter-of-fact system surfaces.** Product prose is "specific, lowercase naturalist observation," while account, error, sync, settings, and accessibility surfaces are "direct and matter-of-fact." The content-template library enforces this split by rejecting gamification words and requiring system copy with clear subject/action/remedy.
- **Operational correctness and privacy without turning the product into a dashboard.** The closing sentence says the plan succeeds only if "operational correctness and privacy are achieved without turning the product into a dashboard." This intent links idempotent events, deterministic simulation, aggregate-only observability, hard deletion, negative-scope audits, and release gates.

## Per-feature whys

### 1. Delivery intent and product guardrails

- **Single-account-per-aviary experience:** The rationale is the "private relationship" model. The storage invariants say "One active aviary per account" and "Exactly one per account," keeping the product away from profiles, follows, public discovery, and multi-aviary account mechanics.
- **Browser-only:** NOT RECOVERABLE FROM PLAN
- **Two starter birds:** NOT RECOVERABLE FROM PLAN
- **Gradual age-gated additions up to seven:** The plan ties this to quiet growth rather than collection mechanics: eligibility comes "strictly from aviary age and capacity," offers are "quiet in-flow," species are chosen "without rarity/collection mechanics," and the seven-bird ceiling is protected by recognizability/performance tests and cautious rollout from 2 to 3, then toward 7.
- **Single unpannable horizontal aviary:** NOT RECOVERABLE FROM PLAN
- **Email magic-link accounts:** The plan's rationale is verified, revocable, non-enumerating access. Links have a 15-minute TTL, one successful consumption, safe rate limiting, and issue a revocable device session; only verified email may receive exports, invites, and magic links.
- **Server-side simulation:** The why is continuity: the aviary must "feel continuous even when no browser is open." The server owns canonical state, ticks the aviary with no active sessions, and prevents clients from writing mood, vector, perch, or species.
- **Multi-device snapshot sync:** The why is to avoid client-to-client state merging. Every visible client reads versioned snapshots, every mutation is an idempotent intent record, and events arrive in server order and are consumed once by the next tick.
- **Presence accounting:** The why is attentive observation without engagement scoring. Qualified presence is the dominant signal for drift, but it must be visible, focused, recently active, capped, clamped, deduplicated, and owner-only.
- **Listen-in:** The plan makes it an attention interaction. It focuses audio on a selected bird while leaving other birds ambient, and its signal contributes chiefly to that bird's social warmth and vocal frequency.
- **Three offers:** The why is bounded, quiet interaction. Only seed, song fragment, and still pool are allowed; they are server-cooled, canonical-response events with narrow trait effects and no catalog, rarity, score, or achievement path.
- **Settle:** The rationale is a reversible quieting of immediate scene state. Settle shifts toward evening, quiets calls, can be reversed by an aviary click in five seconds, terminates that device's qualified presence, and does not change trait score.
- **Read-only notebook:** The plan's why is sparse naturalist memory, not behavioral commentary. Entries come from explicit observation candidates, are persisted as rendered prose, are read-only, avoid duplicate semantic entries, and never write visit streaks.
- **Optional private email invitations:** The why is private sharing without social networking or visitor influence. Invites are per-email, one-time, expiring, revocable, default-off, and visitors get only read-only host snapshots.
- **Narration:** The why is an accessible representation of the same canonical state. A polite live region emits measured naturalist observations and priority bumps without becoming a status dump or interrupting speech excessively.
- **Call captions:** The rationale is accessibility tied to the actual call. Captions are generated from the same instantiated call grammar used for synthesis, synchronized to timing, and default on when audio is unavailable, denied, or failed.
- **Reduced motion:** The plan makes this a first-class rendering mode. It honors system preference, exposes a persistent override, replaces flight paths and leaf drift with designed still-pose cross-fades, and is "not an animation kill switch."
- **Explicit exclusions and negative scope:** The why is to protect continuity, noticing, and private relationship. The plan forbids native apps, payments, public discovery, profiles, follows, comments, chat, shared/co-present aviaries, gamification, caretaker mechanics, hidden counters, trait/status panels, and engagement prompts so the release surface does not become a dashboard.

### 2. Target architecture and service boundaries

- **Modular web application with a versioned HTTP API:** The plan's rationale is clear interfaces now with independent services deferred "until scale requires it." A single backend can contain modules behind boundaries while preserving runner-portable contracts.
- **Single deployable backend:** The why is staged complexity: service separation is explicitly "deferred until scale requires it."
- **Database as the authority:** The database owns account, aviary, bird, event, invitation, and notebook records so invariants and ordered transactions sit in one durable source.
- **Web client boundary:** The client exists to authenticate, fetch snapshots, draw, synthesize calls, collect qualified presence, submit intent events, and render accessible surfaces; it must not tick simulation, persist canonical mood, or write traits.
- **API/auth module:** Its rationale is secure account lifecycle without leaking email or product-tone copy into system surfaces: magic-link issuance/consumption, device sessions, account settings, export/deletion, authorization, and rate limits.
- **Aviary query module:** The why is compact, scoped render state. It builds owner/visitor snapshots, enforces invitation scopes, and returns stable versions/ETags without mutation or visitor presence recording.
- **Interaction ingestion module:** The rationale is durable, bounded owner evidence. It validates, de-duplicates, appends owner events, accepts bounded presence intervals, and makes receipt durable before acknowledgement without directly updating personality.
- **Simulation worker:** The why is canonical control. It processes the event log in order, runs one-minute ticks, updates mood/personality/state projections, creates sparse notebook candidates, and is the only simulation actor.
- **Invitation module:** The why is private, specific sharing. It issues, redeems, expires, revokes, and audits email invitations while excluding public listing, mutual access, chat, and default notification.
- **Observability pipeline:** The plan's rationale is operational health without private behavior. It collects aggregate performance/error metrics and synthetic probes, excluding per-bird state, raw event logs, email, and account-level histories.
- **Stateless API processes:** NOT RECOVERABLE FROM PLAN
- **Relational primary store:** The plan names its reason: "invariants and ordered transactions."
- **Durable job/queue mechanism:** The reason is reliable scheduling for ticks, mail, export generation, and notebook generation.
- **Short-lived encrypted object storage for exports:** The why is privacy and lifecycle control for generated exports.
- **CDN edge for cacheable shell and initial snapshot delivery:** The rationale is fast first delivery while keeping authenticated snapshot responses protected by cache controls and authorization.
- **Canonical domain state and snapshot projection as two forms of state:** The reason is to separate durable truth from a small render document. Canonical state stores identity, vector, mood, clocks, invitations, and settings; projection carries poses, seeds, light, weather, settle state, narration/caption inputs, and cursors.
- **Deterministic local animation seed:** The plan's why is that reconnect "does not present a visibly reset scene." The seed derives from opaque server-provided seeds and snapshot version.
- **Fresh snapshot after visibility restoration or long frame gap:** The rationale is to discard stale interpolation and return to canonical state.

### 3. Domain model and storage design

- **Synthetic UUIDs, opaque client IDs, and encrypted email isolation:** The why is privacy. Email is stored only on the encrypted account record and never in logs, telemetry dimensions, event keys, queue partition keys, or URLs.
- **Account record and lifecycle:** The rationale is the one-aviary-per-account invariant and verified-email boundary for exports, invites, and magic links.
- **Revocable device sessions:** The why is session control: "A revoked session is rejected on every authenticated API."
- **Magic link record:** The rationale is one-time, short-lived access: 15-minute TTL, one successful consumption, and safe rate limiting without exposing email.
- **Aviary record ticking without active sessions:** The why is continuity, since the server tick progresses the aviary even with no active sessions.
- **Bird record with stable identity and hard capacity:** The plan ties this to continuity: identity survives rename, sync, migrations, and species-pool changes; count starts at 2 and never exceeds 7.
- **Personality vector as server-only bounded values:** The rationale is hidden, non-leaking personality drift. Values are bounded, normalized, versioned, server-only, and non-decreasing.
- **Append-only interaction_event log:** The why is durable simulation input and replay/explainability. Only authorized owner sessions generate simulation-affecting events.
- **Presence_interval records:** The plan's reason is qualified owner presence with server-side caps, evidence, deduplication, and merging; visitors never create one.
- **Simulation_run record:** The rationale is retry safety: one committed result per aviary/tick input range, with code/config version and resulting state version.
- **Notebook_entry record:** The why is sparse, read-only observation history retained until account hard deletion.
- **Invite record:** The rationale is per-invite opt-in sharing with opaque token digests, expiry after 30 days, and revocation.
- **Visit_log record:** The plan keeps it host-visible only on demand, with no badge or default notification, preserving the no-announcement principle.
- **Time-zone preference and last confirmed local zone:** The why is coherent day/night mapping across devices without making raw location a requirement.
- **Caption, reduced-motion, audio, and accessible UI settings:** The rationale is persistent user control over accessibility and fallback behavior.
- **Visit-notification opt-in defaulting false:** The why is to avoid notification loops by default while allowing an explicit preference.
- **Client mutation UUID and per-session monotonic sequence:** The rationale is idempotency: ingestion enforces uniqueness and returns the original receipt for retries.
- **Deliberately narrow event types:** The plan limits events to presence, listen-in, offers, settle, and server-authored explainability/replay records so client inputs stay bounded.
- **Client timestamps advisory and clamped:** The why is to prevent impossible intervals; server sequence and receipt time order simulation processing.
- **No event payloads routed into analytics:** The rationale is privacy retention only, not behavior analytics.

### 4. Public API and authorization surface

- **OpenAPI contract, strict JSON schemas, error codes, and versioned formats:** The why is recoverability; the client can request a full snapshot after schema/version mismatch.
- **Matter-of-fact system errors with accessible remediation:** The rationale is the content-domain split: system surfaces use direct copy rather than product tone.
- **POST /v1/auth/magic-links:** The reason is non-enumerating magic-link issuance with rate limits and a 15-minute link.
- **POST /v1/auth/magic-links/consume:** The why is atomic one-time consumption and revocable device-session issuance.
- **GET /v1/aviary/snapshot and delta snapshot:** The rationale is compact current render projection with stateVersion, server time, ETag, notebook cursor, and conditional/delta support.
- **POST /v1/aviary/events:** The why is batched, idempotent, validated owner intent with schema, ownership, cooldown, timestamp checks, accepted sequence numbers, and state hints.
- **GET /v1/notebook:** The rationale is immutable, newest-first, cursor-paginated scrollback retained indefinitely.
- **PATCH /v1/birds/{birdId}:** The plan allows rename only so personality, mood, perch, and species never become exposed or editable client fields.
- **POST /v1/aviary/adoptions/accept:** The why is age-gated growth without catalog, rarity, or score mechanics; the offer must be server-issued and capacity-bounded.
- **GET/PATCH /v1/account/settings:** The rationale is explicit control of captions, reduced motion, audio, visit notification preference, and other settings.
- **GET and DELETE account sessions:** The plan's why is visible session management and revocation.
- **POST /v1/account/export:** The rationale is account data access through a current JSON export and a short-lived verified-address download link.
- **Account deletion and cancellation endpoints:** The why is a 30-day soft delete with restore before deadline, followed by hard deletion lifecycle.
- **Invite create/list/delete endpoints:** The rationale is named email invites, outstanding/revoked/active state, and immediate revocation.
- **Visit consume and visitor snapshot endpoints:** The why is constrained read-only access with terminal matter-of-fact availability when an invite is revoked or expired.
- **GET /v1/visits/log:** The rationale is host-visible, on-demand visit history and outstanding invitation state with no unread count.
- **Separate least-privilege visitor token:** The plan's why is to deny all owner event and account endpoints and terminate active visitors no later than the next snapshot poll after revocation/expiry.
- **Server-side visit duration from visitor snapshot activity:** The rationale is that visits never enter host presence or simulation events.

### 5. Simulation engine

- **Account-partitioned roughly one-minute tick:** The why is continuous canonical simulation with retryable missed work.
- **Per-aviary lease/advisory lock:** The rationale is that only one worker commits a tick for an aviary at a time.
- **Tick algorithm over cursor, ordered events, bird state, time/weather, effects, and projection:** The why is one coherent canonical update that advances cursor/state version and appends derived events atomically.
- **Deterministic pure tick calculation:** The plan's rationale is retry safety and fixture replay over prior state, ordered event range, tick instant, config version, and seeded ambient randomness.
- **Incremental cursor instead of unbounded replay:** The why is that durable vector/mood state is the baseline and the cursor makes calculation incremental.
- **Bounded catch-up windows after late scheduling:** The rationale is avoiding "a burst of visibly separate changes"; the client sees one final coherent state.
- **Tick p99 alert, retry/dead-letter, and protected replay:** The reason is operational correctness when jobs fail or exceed five seconds.
- **Personality vector in [0, 1] with calibration version:** The why is bounded, configurable drift rather than code constants.
- **Positive delta from decayed evidence:** The plan's rationale is gradual additive drift from presence, listen-in, offers, and ambient/social modifiers, clamped to never decrease.
- **Qualified presence conditions:** The why is to make presence real evidence: document visible, window focused, and recent pointer/key activity inside a calibrated window.
- **Periodic heartbeats and server caps/clamps:** The rationale is to avoid a single end-of-session total and reject overlapping impossible sequences.
- **Narrow interaction signal mapping:** The why is trait-specific meaning: listen-in affects social warmth/vocal frequency, offer nearness affects boldness, relevant acceptance affects curiosity, sustained presence increases expressive traits, and plumage follows sustained positive presence.
- **Settle not affecting trait score and neglect adding no negative delta:** The rationale is no punishment, distress, recovery debt, or caretaker mechanic.
- **Conservative drift coefficients with week and three-week targets:** The why is an instrument-detectable but not user-visible movement after about a week, and a perceptible cumulative change after about three weeks.
- **Deterministic simulation fixtures and synthetic test aviaries only:** The rationale is calibration and correctness without aggregating real users' interaction histories.
- **Mood as enumerated state machine:** The why is controlled behavior with versioned future additions.
- **Hysteresis, minimum dwell time, and persisted mood:** The plan's rationale is that birds do not visibly flip mood on successive snapshots and do not reset when a browser session begins.
- **Time, weather, vector, and nearby-bird transition rules:** The why is coherent local behavior: morning alert, evening drowsy/settled, rain reducing calls, wind adding alert/wary bias, vectors shaping response probability, and calls propagating or chorusing.
- **Seeded rare weather schedule:** The rationale is soft aviary-level ambience, "not a user-facing weather system or per-leaf state."
- **Server-authored perch selection:** The why is mood/boldness/occupancy intention from canonical state while the client supplies only continuous paths or reduced-motion transitions.
- **Procedural motif library and stable call-signature seed:** The plan's rationale is recognizable variation without storing or downloading recordings.
- **Server-side return greetings after meaningful owner gap:** The why is a specific owner return moment chosen by bird traits, mood, seed, and absence-duration context; visitor pulls do not create greetings, and longer absence changes form, not tone.
- **Notebook generation from explicit observation candidates and sparsity policy:** The rationale is rare, meaningful, stable naturalist prose that avoids duplicate semantic entries and user-behavior commentary.
- **Third-through-seventh-bird eligibility and adoption:** The why is quiet, age/capacity-based growth with idempotent server-issued offers, no countdown, no score, no achievement, no rarity, stable bird IDs, and rename-only user naming.

### 6. Sync, client session, and conflict model

- **Canonical server record plus append-only owner event log:** The rationale is eliminating client-to-client state merging and last-write-wins on bird vector/mood/perch.
- **Initial shell plus compact snapshot with deferred nonessential UI:** The why is immediate bird rendering; settings, notebook history, invitation management, and nonessential icons can wait.
- **Low-frequency visible-tab conditional pull with ETag/version:** The rationale is current snapshots without client-owned canonical state.
- **Fresh full/delta snapshot on visibility, focus, reconnect, suspend, or accepted event:** The why is to rebase interpolation and never locally merge canonical state.
- **Small durable client outbox with idempotency keys:** The plan's rationale is reliable retry with backoff while the session remains valid.
- **Matter-of-fact non-blocking sync error only when requested action could not be accepted:** The why is direct remediation without generic announcement UI.
- **Stop mutation on expiration/revocation or unrecoverable mismatch:** The rationale is to discard invalid projection and direct the user to sign-in/reload.
- **Simultaneous phone/laptop server ordering:** The why is events consumed once by the next tick.
- **Listen-in as device-local audio focus:** The rationale is that it may log attention evidence but must not globally mute another device's mix.
- **Settle as local scene presentation:** The why is that it ends that device's qualified presence but is not an aviary-wide lockout.
- **Name/settings version preconditions and field-specific conflicts:** The rationale is plain choices only for true user-owned editable conflicts; canonical simulation fields never expose conflict UI because clients cannot write them.

### 7. Frontend scene and rendering pipeline

- **Renderer separated from DOM product controls:** The why is semantic HTML for top bar, dialogs, settings, notebook, account/auth, keyboard focus, live narration, captions, and errors while canvas/WebGL handles the scene.
- **Canvas/WebGL scene layer with tested canvas fallback:** The rationale is rendering birds, perches, foliage, parallax, lighting, weather, and ornament within supported browsers.
- **Pure render-state adapter:** The plan's why is to map snapshots to scene models without calculating personality, server time progression, or eligibility in the client.
- **requestAnimationFrame only while visible:** The rationale is performance and lifecycle cleanup while server simulation remains independent.
- **Procedural/SVG/compact assets, under-2-MB initial bundle, lazy chunks, and starter-bird preload:** The why is first bird paint and initial scene budget.
- **Quiet-field placeholder instead of spinner or skeleton dashboard:** The rationale is preserving the alive illusion if the snapshot is slow.
- **In-progress pose/call/motion phase, nonzero ambient offset, and no wake-up entrance:** The why is to make a valid snapshot feel already living and "in media res."
- **Empty scene only during post-adoption/pre-first-bird handoff:** The rationale is that an established aviary never becomes empty during loading.
- **Three logical perch zones with responsive transforms:** The why is every bird in frame on narrow and wide layouts while preserving one-screen/no-pan/no-zoom.
- **No bird dragging or movement commands:** The rationale is attentive observation and server-authored perch intention rather than user-controlled placement.
- **Bounded pose state machines with sampled variation:** The plan's why is micro-motion that does not become "a static looping cycle."
- **Gentle ambient drift, parallax, local-time palette, sparse ornament leaves/feathers, and soft weather ambience:** The rationale is subtle atmosphere; weather must never become an intrusive event.
- **Thin top bar with limited affordances and fade on stillness:** The why is a quiet scene that remains discoverable and keyboard reachable.
- **No labels, badges, or hover tooltips in the aviary scene:** The rationale is noticing rather than announcing.
- **Naturalist prose in product affordances and direct copy in system/settings surfaces:** The why is the plan's content-domain split.

### 7. Interactions and motion states

- **Click/tap/focus bird selection for listen-in:** The rationale is a direct attention interaction with keyboard parity.
- **Keyboard Tab, arrows, Enter, and Escape for the scene:** The why is an equivalent semantic control model for keyboard and screen-reader users.
- **High-contrast visible bird focus indicator:** The plan's rationale is focus that works against day/night palettes and remains semantic even if canvas owns visible pixels.
- **Listen-in gain ramps instead of hard cuts:** The why is to focus the selected bird while leaving nonfocused birds audible at ambient gain.
- **Offer from top-bar affordance with canonical response animation:** The rationale is bounded interaction whose result comes from the next snapshot/event result, not client invention.
- **Server-side per-bird cooldown and unavailable state without shaming language:** The why is to enforce limits while preserving the product voice.
- **Settle toward evening, quiet calls, and reversible five-second click:** The rationale is a calm, reversible local presentation; closing a tab is equivalent at simulation/presence level.

### 8. Client audio and caption pipeline

- **One managed WebAudio graph per visible session:** The why is controlled master, ambient, focused, voice, limiter/compressor, and lifecycle cleanup.
- **Gesture-gated audible playback with alive scene and captions anyway:** The rationale is that browser policy may defer sound, but the aviary must still behave alive.
- **No pre-recorded call assets:** The why is avoiding canned-audio substitutes in every path.
- **Client-side synthesis from motif, seed, snapshot seed, and mood:** The plan's rationale is recognizable bird signatures with deterministic variation in phrase length, pitch contour, spacing, and timbre.
- **Lookahead scheduling, bounded voice pools, reused nodes, staggered chorus, and depth-aware gain:** The why is timing, memory, collision control, and natural mixing.
- **Suspend on hidden and reconstruct scheduling from fresh snapshot on resume:** The rationale is safe browser lifecycle behavior and continuity.
- **AudioContext monitoring without content, bird IDs, or account IDs:** The why is operational visibility without private identifiers.
- **Caption text from the same instantiated call grammar:** The rationale is consistency between synthesis and accessible text.
- **Graceful silence and captions by default on WebAudio failure:** The why is an accessible fallback without downloading canned audio.

### 9. Accessibility and content-system plan

- **Polite live narration region with scheduler:** The rationale is measured accessible observation: idle narration at most every 30-60 seconds, redundant updates suppressed, and priority bumps that do not interrupt excessively.
- **Narration from render projection and canonical event result:** The why is the same canonical state in "lowercase, present tense, named/visible particulars," without personality numbers, raw perch indexes, generic status codes, or pose-transition firehose.
- **Reduced-motion renderer:** The rationale is a tested rendering mode with slow cross-fades, removed leaf drift, retained palette changes, calls, captions, mood, and notebook activity.
- **Call-caption control and fallback default-on:** The why is user control plus accessibility when audio is denied or unavailable.
- **Caption positioning and WCAG AA contrast:** The rationale is captions and UI text that do not obscure controls and remain readable across light phases.
- **Keyboard, dialog focus, touch target, semantic name, heading, landmark, and bird focus requirements:** The why is that every control and canvas interaction has an equivalent accessible model.
- **VoiceOver/Safari, NVDA/Firefox, Chrome screen-reader, reduced motion, forced colors, keyboard-only, no-audio, denied WebAudio, narrow layout, and zoom/reflow tests:** The rationale is release-quality accessibility coverage.
- **Content-template library with tone tests:** The why is to reject exclamation, gamification words, user-behavior scoring, and generic status copy while requiring clear system subject/action/remedy.

### 10. Performance, reliability, security, and observability

- **Initial JavaScript under 2 MB gzip:** The plan's rationale is initial scene budget verified by CI bundle analyzer and production artifact checks.
- **First bird visible under 500 ms:** The why is first actual bird pixels on mid-tier mobile over representative 4G.
- **Idle rendering at 60 fps:** The rationale is sustained smoothness in normal and two-to-seven-bird scenarios.
- **No upward memory trend over 30 minutes:** The why is bounded audio voices, scene objects, notebook virtualization, and worker lifecycle.
- **Tick latency p99 below five seconds:** The plan uses this as the operational gate for timely server simulation.
- **Low-kilobyte ordinary snapshot size:** The rationale is compact projection verified by contract-size CI and CDN/API timing tests.
- **Aggregate-only RUM:** The why is navigation/request timing, first-bird render, long frames, AudioContext errors, snapshot failures, and fallback rates without account or bird behavior linkage.
- **Synthetic browser checks:** The rationale is availability, auth handoff, first render, snapshot freshness, audio fallback, and revocation behavior from representative geographies.
- **No bird IDs, event payloads, email, notebook text, trait values, or per-account timelines in logs/traces/metrics:** The why is the privacy boundary.
- **Standard web protections:** The rationale is secure transport, session/token handling, CSRF defense, CSP, rate limits, secret rotation, signed opaque tokens stored as digests, encrypted email/export data, authorization checks, and audit events.
- **Verified asynchronous hard deletion and no shadow simulation archive:** The why is complete purge across DB, queues, objects, and operational data within the stated lifecycle.

### 11. Delivery sequence and rollout

- **Foundations and invariants milestone:** The rationale is to establish schemas, opaque IDs, auth/session lifecycle, event idempotency, simulation cursor/lock, API contracts, configuration versioning, privacy-safe logging, and deterministic fixtures before feature layers.
- **Canonical aviary vertical slice milestone:** The why is to prove two starter birds, server tick, projection, renderer, presence, day/night/mood, no-spinner first frame, and replay fixtures under canonical multi-device and no-false-presence tests.
- **Affective interaction layer milestone:** The rationale is to add greetings, calls, listen-in, offers, settle, bird-to-bird responses, notebook sparsity, content system, and adoption behind tone/content and long-timescale calibration fixtures.
- **Accounts and private social milestone:** The why is to complete magic links, sessions, export/deletion, invitations, visitor scope, silent visit log, and matter-of-fact failures behind permission and revocation tests.
- **Accessibility/performance hardening milestone:** The plan states these are "release requirements, not post-launch work," covering narration, captions, reduced motion, keyboard semantics, fallback, bundle splitting, soaks, and support-matrix tests.
- **Controlled release:** The rationale is internal/synthetic first, then invite-only external with two birds, observing aggregate operational health and qualitative opt-in feedback "without inspecting private interaction histories."
- **Cautious age-gated bird ramp:** The why is correctness and performance gates before increasing eligible count from 2 to 3, then toward 7.
- **Feature/config flags only for rollout and calibration:** The rationale is operational control over tick cadence, drift coefficients, weather frequency, notebook sparsity, species availability, and maximum eligible count, while forbidding flags for gamification, alerts, visitor influence, or social mode.
- **Versioned/audited configuration changes against synthetic fixtures:** The why is safe calibration without retroactively recomputing stored personality history.
- **Release criteria end-to-end tests:** The rationale is to prove magic-link expiry/replay, simultaneous devices, worker retry/idempotency, reconnect after suspension, visitor enforcement/revocation, deletion purge, audio fallback, and every accessibility interaction.
- **Product review and negative-scope audit:** The why is to confirm first-frame behavior, greeting/no-announcement behavior, notebook/narration/caption specificity, system-copy clarity, and no scores, streaks, badges, push loops, trait display, public discovery, or tamagotchi state on the release surface.
