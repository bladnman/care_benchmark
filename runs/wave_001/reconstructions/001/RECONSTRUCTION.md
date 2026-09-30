## System-level intent

- **Private, single-user restraint.** The plan repeatedly frames Pocket Aviary as "one private aviary per single-user account" and excludes shared ownership, public discovery, profiles, follows, chat, comments, co-presence, rankings, and engagement notifications. This shows up in Product boundary, Visit API boundaries, observability limits, and rollout gates that are "never engagement/retention or population bird statistics."

- **Continuous place, not a resettable app screen.** The desired experience is a place that "starts already in motion," has "one bird noticing the returning owner," and resumes "mid-action" from persisted timestamps. Verification says V1 is ready only when "the place feels continuous in the first frame" and "the same birds persist across devices and absences."

- **Observe bird choices rather than command birds.** The plan says "The viewer observes bird choices; there is no placement editor," and later makes the server choose offer recipients "not a user bird-placement command." Scene slots, perch allocation, greeter choice, and call response are all server-authored or simulation-authored.

- **No dashboards, scores, or hidden engagement surfaces.** The plan bans "mood badges, numerical traits, user attendance history, or scene labels," says "No user-facing/debug trait display exists," and forbids hidden tables that compute "social rankings or attention scores for later display." Metrics also reject retention funnels, favorite birds, trait averages, and user interaction cadence.

- **Server canonical truth with strict responsibility boundaries.** The simulation worker is "the sole writer" of personality and canonical plans; the Owner API "never updates personality columns"; the client "never runs mood or drift simulation." Tick application makes cursor advancement and vector changes "inseparable."

- **Slow, non-punitive change from attention.** The plan says "Traits never decrease," absence trends toward "ordinary ambient behavior, never mistrust, loss of color, or distress," and attention changes birds "slowly without punishment." Presence is weighted strongly, while listen-in and offers are bounded so "interactions remain secondary."

- **Privacy minimization by design.** Email is encrypted and stored once, UUIDs are synthetic, raw events exist "only to drive that owner's simulation," metrics cannot query simulation/event tables, and deletion includes per-account crypto erasure. Logs redact token URLs, names, notebook text, and emails.

- **Accessibility as the same experience, not fallback.** The plan says "all accessibility modes together," "Accessibility is present now, not reserved for the end," and reduced motion is a "reviewed aesthetic" rather than paused animation. Narration, captions, visible poses, and notebook facts use the same semantic data so they cannot contradict each other.

- **Naturalist, quiet product voice.** Product copy is "lowercase, present-tense, specific to bird/action"; system copy is "normal capitalization and direct action." The plan rejects "generic happiness messages," "milestone prose," "gamification terms," and stock "Welcome back" strings.

- **Aliveness must not depend on audio or heavy assets.** First paint uses inline canonical birds before JavaScript; audio initialization cannot block paint; autoplay failure falls back to a visually/narratively alive scene with captions and a plain sound control. The first bird "cannot depend on downloading two megabytes over 4G."

- **Explicit, read-only social sharing.** Invitations are explicitly sent, one-time, revocable, visitor-only, and have "zero visitor drift." Visit logs are private host transparency, while visitor activity never enters owner events or simulation.

- **Verification over live behavioral analytics.** Calibration uses deterministic fixtures, synthetic accounts, consented laboratory review, and human listening/accessibility sessions. The plan says to "never calibrate by collecting average real-account drift or interaction histories" and to gate rollout on operational health and human quality review rather than engagement statistics.

## Per-feature whys

### 1. Product boundary and decisions

- **One private aviary per single-user account**: The plan uses this to preserve a private product boundary and avoid shared ownership, public discovery, co-presence, rankings, and attention-score surfaces.

- **Email magic-link authentication as the authentication choice**: NOT RECOVERABLE FROM PLAN

- **Two system-selected starter birds**: The plan ties this to the public onboarding baseline, avoids a species catalog or rarity surface, and reserves the quiet empty field plus soft fly-in for initial adoption only.

- **Optional age-based adoption up to seven**: The rationale is restraint: no urgency, catalog, badges, countdown, automatic adoption, visits/payment gate, or misleading opportunity before capacity is qualified.

- **Approximately six species as a count**: NOT RECOVERABLE FROM PLAN

- **Distinct silhouettes and procedural call signatures**: The plan wants species and individual birds to stay recognizable across mood, drift, and repeated species, with verified seven-bird listening sessions.

- **Persistent identity and personality**: Birds keep stable IDs, seeds, grammars, vectors, and notebook attribution so "the same birds persist across devices and absences."

- **Continuing server simulation**: The plan says birds continue having "mornings, weather, and moods while nobody watches"; freezing unattended aviaries is explicitly rejected.

- **Multi-device snapshots**: The why is one canonical state: two devices should read identical records and see the "same canonical state/version at equal server times."

- **Return greetings**: A primary bird noticing in one to two seconds supports felt aliveness without an arrival banner, "simultaneous canned greeting chorus," or displayed absence duration.

- **Listen-in**: It lets focus affect the local mix and qualified attention while keeping other birds on a "nonzero ambient floor"; listen-in cannot multiply drift across devices.

- **Three offers as the exact offer count**: NOT RECOVERABLE FROM PLAN

- **Offers to the aviary with server-selected recipients**: The server chooses eligible recipients by nearest/attentive bird and mood so offers are not placement commands and spam cannot dominate slow drift.

- **Settle**: Settle ends the owner's viewing-session presence and listen-in, adds quieting context, permits a short undo, gives no trait reward, and makes tab close drift-equivalent without the ceremonial palette overlay.

- **Sparse read-only notebook**: Notebook observations concern birds, never attendance; entries are rare, grounded, immutable, and not a routine feed, session log, CRUD surface, or praise loop.

- **Account controls**: Account controls support continuity and privacy operations: settings, sessions, email change, export, deletion, recovery, and revocation without resetting birds.

- **Explicitly invited read-only visits**: The feature allows sharing while preserving host timezone/state, forbidding visitor mutation, and ensuring visitor attention has "zero effect."

- **All accessibility modes together**: Accessibility is a V1 requirement and launch gate because alternate surfaces must preserve the aviary experience, not arrive as later polish.

- **Single responsive horizontal place with three perch depths**: This supports a consistent scene where viewers observe bird choices and where seven birds can remain visible without editable placement.

- **No mood badges, numerical traits, attendance history, or scene labels**: The plan wants to avoid dashboards, streaks, punishment language, and trait/status exposure.

- **Excluding engagement and social mechanics**: The why is product restraint: no scores, achievements, leaderboards, public discovery, follows, chat, comments, co-presence, or engagement notifications.

- **Single persisted IANA aviary timezone**: One timezone preserves canonical moods/light across devices and visitors; traveling owners can deliberately update it instead of snapshots switching it automatically.

- **Four-icon top bar with settle in the offer/action popover**: This keeps account, accessibility, notebook, and offer icons while making both offer and settle reachable "without a fifth icon."

- **Captions and focus outlines as no-scene-chrome exceptions**: The plan permits these as explicit accessibility exceptions while still forbidding ordinary bird labels, tooltips, and inline controls.

- **Focus and Enter listen-in semantics**: Keyboard focus engages listen-in; Enter ensures engagement without toggling it off, while Escape and pointer/touch rules provide predictable disengagement.

- **Opaque encrypted vector export**: Export preserves a state copy while the "absolute concepts rule wins": no plaintext traits, decoding key, numerical summaries, or debug endpoint.

- **Off-by-default visit email toggle**: The plan honors the social exception only through explicit consent, coalesced quiet email, and no push, scene announcement, badge, onboarding prompt, or automatic opt-in.

- **Prompt receipts before next tick**: Receipts make interactions feel responsive while preventing immediate client or API mutation of canonical personality; the next tick consumes immutable outcomes.

- **Absence quietness with monotonic vocal-frequency trait**: Daily mood, time-of-day, and decaying recent-attention context moderate realized behavior so absence becomes ordinary ambient behavior, never distress or trait loss.

### 2. Architecture and responsibility boundaries

- **TypeScript modular monolith as the implementation shape**: NOT RECOVERABLE FROM PLAN

- **PostgreSQL transactional source of truth with durable jobs**: The plan needs transactional state, scheduled ticks, email, exports, deletion, and exactly-once tick effects from at-least-once delivery.

- **Server-rendered HTML shell with inline snapshot and SVG poses**: This supports immediate canonical bird markup and first paint before JavaScript, WebAudio, account panels, or large assets.

- **Canvas 2D plus DOM semantics**: Canvas draws motion while DOM supplies focus targets, captions, settings, and accessibility semantics; the plan avoids a large 3D engine or AudioWorklet for the first bird.

- **Identity module with encrypted email and synthetic UUID references**: This isolates email and avoids using email as a cross-service identifier, partition key, telemetry dimension, or work-item handle.

- **Owner API append-only interaction events and receipts**: The API validates commands and reserves cooldowns without writing personality columns, preserving simulation authority.

- **Simulation worker as sole writer**: The rationale is durable canonical personality, mood, perch/motion/call plans, weather, and facts that are not inferred from historical logs.

- **Snapshot projector hiding trait vectors**: It converts canonical internal state into display recipes while ensuring coordinates, colors, timing, and gain envelopes do not become a trait dashboard.

- **Visit API exposing only host scene projection**: The Visit API supports read-only sharing by refusing owner commands and never creating drift input.

- **Client interpolation and presentation only**: The client can synthesize calls, collect qualified intervals, and render accessibility prose, but never mood or drift simulation.

- **Operational metrics exporter without simulation/event-table permission**: Only operational counters and timings cross this boundary, preventing behavioral analysis and state leakage.

- **Private no-store personalized snapshots with auth checked before 304**: The plan prevents cross-user cache leaks and stale authorized views after authorization expires.

### 3. Persistent data model

- **Random UUID primary keys and encrypted private payloads**: UUIDs and encryption keep private identity, notebook, and names out of logs, metrics, partitions, and cross-service identifiers.

- **Account email stored once**: The plan says no duplicate email in simulation, metrics, or logs, preserving the identity boundary.

- **AccountSettings with visit-notification consent=false**: This encodes off-default social email consent while keeping local audio availability device-specific.

- **AuthChallenge with hashed token, short expiry, and encrypted pending contact data**: Short-lived, purgeable challenges limit exposure for registration and email-change data.

- **DeviceSession without detailed device fingerprints**: Sessions are revocable and user-readable without retaining detailed fingerprinting data.

- **Bird stable UUIDs, seeds, and immutable species/signature allocation**: Stability preserves identity, recognizability, names, notebook attribution, and continuity across rename and migration.

- **BirdPersonality non-null stored vectors with no reset defaults**: Missing data is corruption; default regeneration would hide vector loss behind an apparently functional UI.

- **BirdFastState persisted across sessions**: Mood, perch, action, and call context persist so the scene resumes rather than reopening at defaults.

- **InteractionEvent append-only with idempotency and sanitized payloads**: Append-only events support ordered tick consumption, retry safety, and bounded presentation outcomes until retention purge.

- **PresenceInterval union and no raw mouse paths/key contents**: The plan wants account-level drift truth without double-counting device overlaps or storing detailed behavior traces.

- **CommandReservation written with the event**: This prevents two devices from bypassing cooldowns while staying separate from personality state.

- **PresentationReceipt with bounded lifetime and replay idempotence**: Receipts provide short-lived reactions without becoming lasting optimistic state.

- **NotebookEntry immutable name-at-observation, prose, and facts**: Old entries remain browsable and grounded without owner behavior tallies or vector values.

- **AdoptionOpportunity unique thresholds with no scores or rarity**: Age-only opportunities avoid catalog/rarity and prevent accepting beyond seven.

- **VisitInvitation synthetic handles and hashed one-time token**: This avoids email-derived handles while supporting explicit invitation, expiry, revocation, and redemption.

- **VisitSession and VisitLog separate from owner events**: Visit accounting gives host transparency without bird-level attention fields or simulation input.

- **ExportJob and DeletionJob without email in queue messages**: Queue work can progress without copying email into operational messages.

- **Encrypted invitation-contact record**: The social feature can send and show recipient email without making it an account identifier or visit-log copy.

- **Encrypted identity lookup confined to identity service**: Per-email rate limits and lookup do not expose email or reusable email-derived IDs outside the identity boundary.

### 4. HTTP contracts and account flows

- **Versioned JSON API with typed schemas, allowlists, cookies, CSRF, CSP, and redaction**: These protect ownership, token URLs, bodies, names, notebook text, emails, and stable error behavior.

- **Magic-link generic response and rate limits**: The generic response avoids account-existence leakage, while token entropy, expiry, single use, and throttling address replay and abuse.

- **Magic-link consume via explicit continuation page**: Link scanners and browser prefetch must not consume the login token.

- **Device session list and revocation**: Revocation invalidates future pulls/commands, closes presence leases, and removes private client state.

- **Settings PATCH with If-Match**: Version checks prevent stale silent overwrites and return current settings on conflict.

- **Email change with verification before atomic swap**: The old address remains active until the new verified address can be uniquely installed.

- **Asynchronous export with private 24-hour download link**: The plan wants a transactionally consistent snapshot, status instead of aviary toast, and account-status rechecks before generation/download.

- **Deletion and recovery endpoints**: Mark deletion immediately, revoke visitors, stop ordinary operations/ticks, and allow 30-day recovery without resetting birds.

- **First successful sign-in atomic account/aviary/bird creation**: Atomic creation prevents duplicate aviaries or starter pairs; naming keeps adoption incomplete until confirmation.

- **Initial naming quiet empty field and one-time fly-in**: The empty field is permitted only when genuinely empty, and the fly-in never repeats on normal returns.

- **GET /aviary compact authorized snapshot**: The snapshot supplies rendering recipes and stable references while excluding vectors, event history, visit counts, other devices, and notebook contents.

- **Quiet field on snapshot cache miss**: The plan prefers failure honesty over fabricated/default birds.

- **POST /aviary/arrivals with nonce and optional signed proposal**: This enables a first greeting without waiting for a second round trip while preventing unconfirmed presence/drift and repeated greetings.

- **POST /aviary/events batched with server-derived session/device**: The server assigns order, validates intervals, calculates listen-in duration from leases, and rejects trusted client totals.

- **Terminal events and beacon validation**: Prompt blur/hide/settle/pagehide events improve accuracy, while lease expiry makes lost terminal events safe.

- **Offer payload limits**: Seed, validated song fragment, and still pool keep offers bounded; uploaded music is excluded.

- **Offer recipient choice and cooldowns**: Server choice and per-bird cooldowns prevent bird-placement commands, cross-device bypass, and spam acceleration.

- **Quiet explanation when all birds cool down**: The plan avoids countdown badges and punishment language while keeping feedback inside the already-open offer panel.

- **Settle five-second undo token**: Scene click can undo before listen-in or offer action; later scene interaction resumes deliberately.

- **Settle presence/session effects**: Settle ends this owner's viewing-session presence and listen-in, quiets mood slightly, and does not remove presence from other active devices.

- **Notebook pagination endpoint**: Stable cursor pagination keeps long immutable history accessible without event-counter summaries.

- **Versioned rename endpoint**: Rename conflicts are explicit and renaming never touches simulation state, IDs, seeds, grammars, vectors, or old notebook attribution.

- **Age-based adoption endpoints**: Opportunities are current, deferable indefinitely, one-at-a-time, and free of urgency, catalog, badges, countdown, automatic adoption, or visits/payment gates.

- **Invitation create/list/revoke and visit log**: Explicit send starts social sharing; private lists/logs support transparency without badges or success toasts.

- **Visitor redeem flow and visit-only cookie**: One-time redemption creates a scoped visit session without requiring visitor account creation or adoption.

- **GET /visit/aviary shared projection with reauthorization**: Rechecking every request and before 304 implements revocation at next pull and clears private state when unavailable.

- **Ten-second visit authorization freshness lease**: Offline or contact-lost clients cannot retain an indefinitely watchable cached visit.

- **Visitor local presentation controls only**: Audio, captions, narration, and reduced motion stay available because they affect only that browser, while visitor listen-in, offers, settle, rename, notebook, and drift are forbidden.

- **Host visit email**: If enabled, it is queued independently of simulation and never includes bird behavior.

### 5. Presence and concurrency correctness

- **Five-minute recent-activity window**: The plan credits recent actual activity without treating a still open tab as measurable continuous attention.

- **Eligible time as visible document, focused window, and pointermove or keypress**: This exact intersection prevents clicks, audio playback, tab-open duration, or visitor activity from substituting for the stipulated signals.

- **Touch and keyboard paths**: The plan requires testing actual mobile pointer-event behavior while keeping ordinary keyboard-accessible interaction, instead of weakening the definition silently.

- **Local monotonic qualification boundaries**: Boundary timestamps improve accuracy while storing only the latest pointer/key timestamp, never content or movement traces.

- **Still-watching cap after one movement**: Ten still minutes after one movement credits five, reflecting the plan's refusal to pretend continuous attention is measurable.

- **Server leases, 15-second clamps, and 20-second expiry**: The server bounds unprovable claims, rejects offline hours, and does not infer presence from keepalive alone.

- **Cross-device account interval union**: Overlapping devices cannot multiply drift.

- **Listen-in interval union by bird**: Multiple devices concentrating on a bird cannot multiply listen-in drift.

- **Idempotency scoped to aviary, session, and key**: Retries return the original outcome without another offer, presence interval, or greeting.

- **Late events applied to the next tick**: The plan avoids replaying events into old personality state.

- **Offline rendering without durable mutation queue**: Offline clients can show a bounded last-known scene, but commands need acknowledgement and cannot become new durable commands.

- **Version checks, locked reservations, and tick discipline for races**: These ensure conflicts never ask the user to choose between bird personalities because there is only one canonical personality.

### 6. Server simulation engine

- **60-second tick cadence as the exact cadence**: NOT RECOVERABLE FROM PLAN

- **Staggered UUID-derived tick buckets**: Staggering prevents all accounts ticking on the minute boundary.

- **No inactive-account exemption**: Birds continue having mornings, weather, and moods while nobody watches; deletion-pending is the explicit lifecycle exception.

- **Tick row lock and missing-vector handling**: Missing vector data is corruption; the aviary stops ticking and restores from durable stored-state backup instead of creating defaults.

- **Frozen event sequence bound per tick**: Events committed after the bound wait for the next tick, preserving ordered deterministic consumption.

- **Stable seed per tick and engine version**: Retry produces the same result.

- **One transaction for vectors, fast state, filters, notebook, version, cursor, and tick times**: Cursor advancement and vector changes are inseparable, giving exactly-once effects under at-least-once jobs.

- **Outage catch-up in chronological substeps**: Stored state, elapsed time, and historical timezone context preserve ambient progress without multiplying the last heartbeat by outage length.

- **Nonnegative slow-drift algorithm**: Trait deltas are additive and monotonic, so one long absence cannot erase identity.

- **Presence as at least 80 percent of permitted stimulus**: The plan makes passive qualified presence primary and keeps explicit interactions secondary.

- **Listen-in and offers bounded to small stimulus shares**: Listen-in can help focused warmth/vocal traits and offers can react immediately, but spam and toggles cannot dominate slow drift.

- **No mute penalty**: Muting is a preference/context, not a reason to reduce traits or disadvantage audio-off users.

- **Filter, cap, and coefficients as initial hypothesis**: The algorithm is calibrated through synthetic fixtures and perception review, not live-account drift dashboards.

- **Mood enum and drowsy pose**: Sleeping/eyes-closed is represented as drowsy pose rather than another numerical status surface.

- **Mood dwell timers and dawn blending**: Timers prevent flicker, and local dawn blends transient bias rather than resetting to neutral on page open.

- **Quiet seeded weather**: Server-generated rain/wind gives canonical restrained variation without external weather/location API, thunder, or snow.

- **IANA local lighting and DST handling**: Host-zone day context is canonical while monotonic elapsed time governs durations.

- **Bounded bird-to-bird responses**: One-generation responses and refractory windows let chorus emerge without endless alarm loops.

- **Weighted, staggered greeting**: Bold/warm birds may greet first, but randomness and staggering avoid a user-arrival fanfare.

- **Canonical action descriptors and server perch allocation**: Server-side slot choice keeps all seven distinct and visible while the client only maps responsive projection.

- **Stable bird signatures**: Species grammar plus individual register, timbre, rhythm, and seed prevent drift from making one named bird sound like another.

- **Browser call expansion from recipes only**: The browser expands motifs and captions but does not run mood transitions, personality updates, or call-response simulation.

- **Caption generation from final expanded descriptor**: Caption prose matches the actual synthesized call instead of a fixed species string.

- **Deterministic notebook templates without LLM services**: Grounded authored templates protect privacy and avoid sending simulation history to third-party text services.

- **Notebook sparsity and deduplication**: Rare observations avoid a feed, routine session log, or fabricated multi-minute behavior from one instant.

### 7. Frontend rendering and session choreography

- **Server-rendered exact-phase SVG birds before JavaScript**: The first bird is canonical, not a placeholder, and does not wait for audio, panels, art downloads, or fonts.

- **Hydration swap to Canvas at matching frame**: This preserves continuity with no flash, duplicate bird, or pose restart.

- **Canvas-unavailable SVG renderer**: The compact SVG pose renderer still supports the designed slow cross-fade surface.

- **Server clock offset and persisted action timestamps**: Navigation resumes mid-preen rather than pose zero.

- **Skipping expired calls after suspend**: The plan avoids catch-up queues and stale replay.

- **Immediate pull on visibility regain or render gap**: The client adopts current timeline without a neutral mood reset.

- **Visible-state polling with jitter and command follow-up pulls**: Pulls keep the scene aligned to state versions without client-to-client sync.

- **Stopping RAF, ornaments, polls, audio, and presence when hidden**: Hidden documents stop local work and terminate presence while the server continues ticking.

- **Quiet sky field for slow/cold snapshot**: The plan rejects spinners, textual welcomes, fabricated birds, and reset recovery.

- **Responsive three-band composition with safe bounds**: Birds, captions, and wing extents stay visible across narrow, wide, zoomed, and short viewports.

- **No panning on unusually small viewports**: The plan reduces ornament density and compacts poses/spacing instead.

- **Top bar fade behavior**: Chrome recedes after inactivity but controls, focus outlines, captions, hit areas, and open menus remain stable and readable on interaction.

- **Listen-in local ramps and cross-ramps**: Audio focus changes feel immediate while acknowledgement controls slow-attention accounting and network errors roll back uncertain state.

- **Settle visual choreography**: Light shifts and calls reduce, but incidental movement does not re-engage presence and close remains allowed without ritual.

- **Reduced motion before first paint**: OS preference is applied before first paint to prevent a default-motion flash.

- **Reduced motion as alternate aesthetic**: Cross-fades, palette/texture changes, and retained facts/calls/captions/greetings preserve the same experience without fast motion.

### 8. Procedural audio and listen-in mix

- **One WebAudio context with generated resources**: Calls are mathematically synthesized with no downloaded or recorded call files, and first paint is independent of audio initialization.

- **Small song-fragment library as note/contour data**: Song offers remain synthesized through the same graph rather than uploaded recordings.

- **Deterministic call expansion and lookahead scheduling**: Server-relative call times map to AudioContext time while late calls are skipped or resumed only at valid remaining envelopes.

- **Reusable capped voice pool and cleanup**: Capping voices, stealing low tails with ramps, and disconnecting nodes prevent clicks, overload, and memory growth.

- **Ambient chorus preserving signatures**: Staggered onsets, spatial panning, headroom, and non-phase-aligned oscillators keep individual signatures clear and comfortable.

- **Listen-in ramps with nonzero ambient floor**: Focused gain rises modestly while other birds remain audible, preserving the place instead of soloing a bird.

- **Autoplay-restricted sound handling**: The scene remains alive visually and through captions, with a plain Enable sound control rather than modal, banner, or canned substitution.

- **Unavailable or interrupted WebAudio behavior**: Captions turn on, aggregate error categories are emitted, and retries require explicit gesture so contexts do not multiply in a loop.

### 9. Accessibility, language, and interaction semantics

- **Shared projected semantic data**: Narration, captions, visible poses, and notebook facts use the same action/call data so accessibility surfaces do not contradict the scene.

- **Narration live region cadence and coalescing**: Slow polite prose gives current bird/action context without stale queue buildup, hidden-tab updates, or interruptive alerts.

- **Narration exclusions**: It never narrates numbered perches, raw mood labels, trait scores, or owner absence/attendance, preserving the no-dashboard/no-attendance rule.

- **Call captions derived from final procedural descriptor**: Captions match phrase count, contour, trill, pause, loudness, and perch instead of generic species text.

- **Caption overlap and bounds handling**: Shortening, staggering, backplates, and reflow keep captions truthful and readable across overlapping calls and scene states.

- **DOM semantic scene region with roving tabindex**: Keyboard and pointer focus share state, arrows move through stable bird order, and Canvas does not duplicate interactive semantics.

- **Visitor bird descriptions as noninteractive**: Visit scenes preserve accessibility descriptions without exposing listen-in buttons or owner controls.

- **Panel focus management and accessible forms**: Notebook, account, accessibility, and action popovers have headings, Escape close, focus restoration, load-more behavior, labels, and matter-of-fact errors.

- **Configurable Alt+Shift+O shortcut as the specific shortcut**: NOT RECOVERABLE FROM PLAN

- **Shortcut reachable offer/action menu**: The menu is optional and also reached through Tab, avoiding mouse-only affordances and browser-command overrides.

- **Contrast thresholds across states**: Palettes, captions, focus boundaries, and faded chrome remain compliant across morning, night, rain, zoom, and forced-colors states.

- **Product copy lower-case, present-tense, bird/action-specific**: The voice stays naturalist and concrete, e.g. "pip tilts toward the still pool," instead of generic happiness, milestone, or gamification language.

- **System copy normal capitalization and direct action**: System messages use matter-of-fact operational language like "Your session timed out."

- **Manual accessibility and charm review**: VoiceOver, NVDA, keyboard-only, captions, reduced motion, zoom, high contrast, and participant review are required because automated semantic checks are insufficient.

### 10. Performance budgets and operational observability

- **Initial JS ceiling treated as a ceiling, not target**: The first bird cannot depend on downloading two megabytes over 4G, so the working target is much smaller.

- **Inline HTML/CSS/pose/snapshot budgets**: Small critical bytes support a canonical first bird under reference network/device conditions.

- **First visible bird under reference profile**: The metric must be a painted canonical bird, not a loading silhouette or canvas-created mark.

- **Honest reference 4G/geography profile**: The plan says a 500ms budget across arbitrary slow connections is impossible, so slow regions use quiet-field behavior instead of fabricated birds.

- **Canvas DPR cap, pre-rendered foliage, pooled structures, lazy chunks**: These support 60fps, bounded frame work, and deferred heavy surfaces.

- **No per-frame component-tree reconciliation**: The rationale is stable frame time for seven birds over a 30-minute soak.

- **Memory CI with warm-up, 30-minute observation, and heap diffs**: The plan rejects retained growth, accumulating audio nodes/listeners, and hidden leak slopes.

- **Operational metrics from day one**: Request health, first-bird timings, frame distributions, audio failures, queue/tick freshness, email/export/deletion success help operators run the service.

- **Metrics allowlists and stripped payloads**: Names, email, notebook, token URLs, payloads, and arbitrary exception attachments are kept out of metrics.

- **Forbidden engagement/behavior analytics**: Retention funnels, streaks, offers-per-user, favorite birds, trait averages, interaction cadence, visitor rankings, session replay, and real-aviary screenshots are excluded to protect product restraint and privacy.

- **Synthetic performance account isolation**: Diagnostics from synthetic state cannot be confused with real-account telemetry.

- **Operator alerting only**: Alerting is for operators, never a bird-related user notification.

### 11. Privacy, durability, and security operations

- **Raw owner event retention for seven days after consumption**: Events exist only to drive that owner's simulation and idempotency/correctness recovery, then are deleted after cursor advancement.

- **Current personality as stored durable state**: The plan keeps long-term vectors and compact summaries as current state, not reconstructed history.

- **No simulation payloads to outside providers**: Email, crash vendors, LLM providers, recommendation systems, and training pipelines do not receive calls, offers, traits, or presence.

- **Synchronous durable vector commit, replication, backups, and PITR**: These protect stable bird IDs and vectors from loss and prevent replay from inventing personality.

- **Migration and rollback preservation**: Scripts cannot truncate personality or reseed birds; rollback changes code/config, not an account's evolved personality.

- **Deletion pending flow**: It revokes visitors, hides the aviary behind recovery controls, pauses simulation, cancels exports/invites, and permits recovery without recreating birds.

- **Hard deletion and per-account key destruction**: Purging data, destroying keys, and applying restore tombstones ensure deleted encrypted payloads and backups cannot resurrect readable account data.

- **Deletion audits through synthetic fixtures**: The plan verifies all stores, queues, artifacts, and restoration paths without copying real PII into audit tools.

- **Export consistent snapshot with opaque vector payload**: Export includes birds, names, moods, settings, notebook, and sealed current vector state without plaintext traits or owner presence histories.

- **Private signed export authorization and 24-hour artifact deletion**: Artifacts use random handles, account encryption, and short retention rather than mailed attachments containing traits/private state.

- **Abuse controls**: Token entropy, replay, CSRF, cross-account access, forged durations, giant payloads, invalid names, invitation flooding, and visitor-cookie mutation are explicitly defended.

- **Scoped cookies for owner and visit sessions**: A browser can view a friend's aviary without confusing that access with its owner session.

- **Rate limiting as operational protection**: The plan says rate limiting is not user interaction punishment.

### 12. Verification strategy and acceptance cases

- **Tests around product invariants and failure cases**: Verification focuses on invariants rather than duplicating implementation internals.

- **Test-only private fixture inspector excluded from production**: It supports deterministic verification without creating any user-facing or debug trait display.

- **Vector integrity cases**: Rename, sign-in, second device, deployment, migration, restart, and restore must preserve IDs/seeds/vectors; missing vectors fail closed.

- **Drift calibration cases**: Regular presence, absence, spam, listen toggles, muted/caption-only use, and long sessions verify monotonic traits and secondary interactions.

- **Presence truth table**: All visible/focused/recent-activity combinations, expiry, blur/hide/pagehide/settle, missed beacons, mobile behavior, and two-device unions protect attention correctness.

- **Tick atomicity cases**: Crash and concurrency tests ensure exactly one delta per input and cursor/vector commit together.

- **Sync cases**: Cross-device offers, stale conflicts, retries, expired sessions, ETags, and receipts verify one canonical state/version.

- **Session feel cases**: Mid-action first frame, primary greeting timing, procedural variation, no banner/chorus, and initial-adoption-only fly-in protect aliveness.

- **Human listening and accessibility launch gates**: Automated pass alone cannot establish uncanniness, recognizable signatures, felt aliveness, or participant accessibility quality.

- **Drift perception day 0/7/21 review**: The plan wants day-seven instrumentability and day-21 perceptibility without obvious one-session jumps or live-account drift dashboards.

- **Load tests for never-connect accounts**: Capacity must include approximately N/60 jobs per second with zero online accounts, so scale is not saved by freezing unattended aviaries.

### 13. Delivery sequence, dependencies, and rollout

- **Small complete slices with testable exits**: Each stage has exit criteria so foundations, vertical slice, interactions, account operations, sharing, and release qualification can be verified.

- **Foundations and specification assets first**: Schemas, event contracts, UUID/privacy boundaries, seeds/versioning, authorization roles, species motifs, palette, focus tokens, fixtures, and metric allowlists prevent unsupported design-system dependency.

- **Two-bird canonical vertical slice**: This proves durable vectors, client-independent ticks, private bootstrap, mid-action rendering, calls/captions, greeting, presence, keyboard, narration, and accessibility early.

- **Owner interactions and observations stage**: Listen-in, offers, settle, notebook, naming, and age-only adoption are paired with drift fixtures to ensure spam cannot accelerate drift and no numbers/streak/counters leak.

- **Accounts and continuity operations stage**: Lifecycle, backup, restore, idempotency, deletion, and outage recovery must pass so migrations/restarts do not reset birds.

- **Read-only sharing stage**: Invitations, visitor projection, authorization leases, private logs, revoke, and off-default notification are gated by forbidden visitor actions and zero drift.

- **Release qualification stage**: Six-species review, seven-bird narrow-view/chorus/load tests, browser/accessibility checks, soak, geography checks, and privacy negatives must pass with deploy/restore rollback.

- **Internal synthetic build, consented closed pilot, staged public rollout**: Expansion depends on operational health and human quality review, never engagement/retention or population bird statistics.

- **Full local day/night observation per rollout step**: This prevents judging slow drift from one afternoon and exercises relevant zones.

- **Two-bird public onboarding baseline with two-to-four-to-seven qualification**: The plan avoids automatically adding birds to public accounts for rollout testing and exposes no opportunity until count is qualified.

- **Feature flags preserving state and ownership**: Flags may control projection, rollout, adoption, and invitations but cannot reset state, change ownership, or resimulate drift from history.

- **Operational rollback preserving records**: Rollback can disable new invitations/adoptions or swap compatible renderer while preserving existing records and revocation rules.

- **Simulation correctness failure surface**: If simulation is suspect, affected writes stop and durable state/events are preserved; the plan rejects pretending progress by reseeding birds.

- **Day-one runbooks with UUID/error/timing diagnostics**: Operators diagnose tick, snapshot, audio, magic-link, vector, visit, export, deletion, and restore issues without dumping state into analytics/logging.

### 14. Risks and mitigations

- **Drift speed risk mitigation**: Low-pass filter, presence weighting, caps, and versioned calibration are used; release is delayed if three-week perception is wrong.

- **Vector loss risk mitigation**: Restricted writer, non-null records, atomic cursor commits, durable replication, and restore tests make missing data fail closed.

- **Multi-device double-credit/cooldown race mitigation**: Account interval union, idempotent keys, row-locked reservations/ticks, and versioned metadata avoid last-write-wins personality paths.

- **One-minute latency mitigation**: Server-authored receipts and bounded overlays give responsiveness without making optimistic client state a second simulation.

- **Procedural audio quality mitigation**: Distinct signatures, bounded variation, headroom, ramps, nonzero background, and capped response chains protect recognizability and comfort.

- **Autoplay restriction mitigation**: Visual/narrated aliveness, captions, and deliberate enable-sound controls prevent audio limits from blocking paint or drift.

- **Presence measurement risk mitigation**: Exact signal intersection, bounded leases, unions, and explicit observation limits avoid inferring attention from open sessions.

- **Accessibility fallback risk mitigation**: Authored prose, priority queue, captions, cross-fades, and launch-alongside-default treatment make failures launch blockers.

- **Mobile crowding risk mitigation**: Safe slots, minimal ornaments, stable hit regions, caption collision handling, and no crop/pan/tiny copy protect seven-bird narrow views.

- **First-bird budget risk mitigation**: Inline canonical SVG/snapshot, small critical bytes, private regional delivery, deferred UI/audio, and quiet-field failure avoid false first-bird metrics.

- **Tick cost risk mitigation**: Staggered scheduling, UUID sharding, bounded transactions, and worker capacity support unattended aviaries without freezing them.

- **Invitation leak/drift risk mitigation**: Pull authorization before 304, scoped cookies, freshness leases, no visit owner-event route, and separate visit accounting protect revocation and drift.

- **Product restraint erosion mitigation**: Static route/string review and explicit forbidden surfaces reject celebratory toasts, counters, trait/attendance/ranking APIs, and other conventional UI creep.

- **Export/notification contradiction mitigation**: Sealed vector payload and explicitly consented social email keep implementers from adding implicit trait dashboards or default notifications.

- **Telemetry/deletion promise mitigation**: Metrics role isolation, allowlisted payloads, no third-party state analytics, crypto erasure, restore tombstones, and deletion drills make hard-delete a tested workflow.

- **V1 readiness standard**: The plan's final why is that the place must feel continuous, birds persist, attention changes them slowly without punishment, and accessible surfaces preserve that experience within runtime/privacy budgets.
