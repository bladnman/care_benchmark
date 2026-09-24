## System-level intent

- Keep the product "observational and quiet." This is stated directly in scope and then repeated through exclusions: no "scores, streaks, visit calendars, achievements, hunger or distress," no return toast, no welcome string, no elapsed-absence UI, and no user-facing progress indicator. The same intent shows up in offers as "a gesture, never sustenance," in absence behavior as no "illness, distress, decay, or punishment," and in risk control as "Never solve weak perceived response by rewarding clicks or penalizing absence."

- Preserve one server-owned canonical aviary. The plan says "Each account owns one canonical aviary" and "The server owns all canonical state," while the browser owns only "transient rendering state." This intent appears again in event contracts: clients send facts, "never state outcomes," and there is "no endpoint that sets a bird vector, mood, perch, or simulation timestamp."

- Treat the browser scene as a presentation projection, not a second model. The bootstrap snapshot is "a presentation projection, not a copy of the server's model," and the renderer must "consume immutable snapshot/presentation cues plus local interpolation time" and "produce a frame without mutating canonical state."

- Use naturalist, lowercase, present-tense product voice for aviary surfaces and matter-of-fact language for account surfaces. The plan explicitly assigns "naturalist, lowercase, present-tense prose" to bird, notebook, caption, and narration surfaces, while sign-in, account, settings, unsupported-browser, and error surfaces use "clear matter-of-fact language."

- Keep social access constrained, read-only, and revocable. The invite link becomes "a revocable, read-only visitor credential"; visitors receive only "a constrained render snapshot," have "no event-write permission," and "do not count visitor time as host presence." The plan also excludes public discovery, profiles, comments, and shared aviaries.

- Make privacy and minimization cross-cutting. The plan calls for synthetic account UUIDs, encrypted email fields, token hashes, no raw keystrokes or pointer paths, aggregate-only telemetry, redacted credentials and query tokens, and never using "an email address as a primary key, partition key, log field, or telemetry dimension."

- Make synchronization deterministic, ordered, and idempotent. This appears in row locks or optimistic version checks, server sequence as "the total order," idempotency UUIDs, transactional cursor advancement, deterministic tick seeds, and the requirement that retries cannot create "extra calls, drift, or notebook entries."

- Make change slow, calibrated, and never punitive. Personality uses a "low-pass filter over capped, positive daily inputs," small per-day deltas, deterministic cohorts, no trait decrement for neglect, and visible behavioral differences only after longer horizons such as "around week three." Bird count grows through "age gates, never visit-count rewards."

- Keep bird identity stable over time. Names and species stay attached to "stable bird identities"; birds have stable UUIDs and call-signature identifiers; "Renaming changes only the name"; and stable IDs and vectors survive simulation deployments and species-pool changes.

- Ship accessibility with v1 rather than treating it as a later layer. The plan says "Ship accessibility with v1" and makes narration, reduced motion, captions, keyboard paths, and contrast "release gates." Reduced motion must preserve "the same mood, calls, and state."

- Prioritize first-frame presence and quiet performance. The "first visual frame must not wait on audio, notebook, settings, or noncritical art"; returning users should see "current motion without an entry animation"; a slow snapshot may show a quiet field "never a spinner"; and release gates include first bird visible within 500 ms and idle motion at 60 fps.

## Per-feature whys

### 1. Scope and product invariants

- Browser-only, single-user Pocket Aviary: NOT RECOVERABLE FROM PLAN

- One canonical aviary per account: The plan frames this as a product invariant and supports it by making the server own all canonical state and by serializing ticks for "one aviary." The rationale carried by the plan is to avoid multiple competing copies of the aviary state.

- New aviary starts with two system-selected birds: NOT RECOVERABLE FROM PLAN

- Account can grow to seven as the aviary ages: The plan says these are "age gates, never visit-count rewards," and rollout says to "never use visits, offers, time spent, or payment to unlock birds." The why is to make growth time-based rather than a reward, payment, or engagement loop, while preserving performance and call recognizability.

- Magic-link sign-in: NOT RECOVERABLE FROM PLAN

- Persistent simulation: The plan says mood persists across sessions, opening a tab "never resets it," and birds "continue their server-side life and calls when no client is connected." The why is to make the aviary feel ongoing rather than session-bound.

- Multi-device snapshots: The plan grounds this in server-owned canonical state and projection snapshots. It pulls snapshots on initial load, visibility return, long frame gaps, and visible-tab keepalive so concurrent browser sessions share the server's current projection instead of writing old absolute state.

- Offers: The plan says an offer is "a gesture, never sustenance." The rationale is to allow user-initiated interaction without introducing hunger, distress, or a feeding mechanic, and to let mood and curiosity determine approach/reaction rather than user control.

- Listen-in: The plan lets the user focus a bird while lowering others "to an ambient floor, never to silence." The why is to support attentive listening without making the rest of the aviary disappear, while start/end facts can feed drift and reconnect recovery.

- Settle: The plan makes settle a "soft several-second evening shift" that lowers calls. It ends qualified presence but creates "neither a penalty or drift decrement," so the feature quiets the scene without punishing the account.

- Sparse read-only field notebook: The plan wants "meaningful, sufficiently rare" observations, not sessions, attendance frequency, trait numbers, or generic rows. The why is to provide a naturalist record that remains sparse, specific, and observational.

- Account export: The plan treats export as a "narrow exception" to the no-stats UI rule because the explicit user-requested JSON download must include current vectors. The why is account control without turning vectors into normal product UI.

- Account deletion: The plan places deletion with account controls and says deletion must be exercised through mail queues, grants, events, snapshots, and notebook records. The why is to make deletion comprehensive across stored account-owned state.

- Per-invite read-only visits: The plan constrains visits to a scoped render snapshot with no event-ingestion route. The why is to permit a visit without creating shared aviaries, public discovery, comments, or visitor influence on host presence.

- Screen-reader narration: The plan says accessibility ships with v1 and narration is driven from the same snapshot in "slow naturalist prose" at 30-60 second idle intervals. The why is access to the same scene state without flooding the speech queue or exposing raw state rows.

- Reduced-motion rendering: The plan says reduced motion substitutes still-pose cross-fades, removes leaf drift, and slows ambient color changes while preserving "the same mood, calls, and state." The why is accessibility without changing the underlying aviary.

- Call captions: Captions come from "the same seeded grammar event that creates the sound," so prose accurately describes the synthesized call. They also become the default fallback when audio is unavailable.

- Excluding scores, streaks, visit calendars, achievements, hunger, distress, public discovery, profiles, comments, shared aviaries, payments, native clients, scene customization, and user-controlled perch placement: The direct why is "Keep the product observational and quiet." The exclusions prevent reward loops, social/public surfaces, monetized unlocks, and direct manipulation from becoming the product.

- Keeping personality values out of product UI: The plan says values are "server-only" and "out of the product UI," with export as the narrow exception. The why is to maintain the no-stats UI rule and avoid numbers, badges, meters, hover labels, or progress indicators.

- Stable bird names and species: The plan says names and species remain attached to "stable bird identities" and "Renaming changes only the name." The why is continuity of identity even as simulation state changes.

- Single horizontal scene with no panning, zooming, or scrolling: The plan specifies this as an invariant and repeats that birds stay inside the viewport. The rationale articulated is to keep the aviary a bounded scene rather than a navigable map or placement tool.

- Social opt-in visit-notification setting: The plan calls it "the sole exception" to social defaults of no notifications, with a "matter-of-fact email" only when the host enables it. The why is to honor the explicit social file while avoiding default, marketing, or general aviary notifications.

- Redeeming an invite link once into a visitor credential: The plan says a forwarded or reused redemption link "cannot mint additional credentials," and unused grants expire after 30 days. The why is to keep invitation access revocable and bounded.

- Audio autoplay handling: The plan notes browser autoplay rules may prevent audio before a trusted gesture. The why is to render the aviary immediately, resume WebAudio only when permitted, show captions when enabled, and never substitute recorded audio.

- Age-eligible bird adoption in a user-initiated aviary options flow: The plan says there should be "no badge or arrival notification" and the entry point "must not become another persistent scene control." The why is to keep adoption user-initiated and compatible with the sparse top bar.

- Bird age measured from aviary creation timestamp: The plan says thresholds are "age gates, never visit-count rewards" and can be tuned without a schema change. The why is to make eligibility time-based and configurable.

### 2. Architecture and ownership

- Web client, small authenticated application API, persistent relational store, and scheduled simulation worker: The plan's rationale is modularity for the first release, with clear account, aviary, simulation, notebook, and visit modules, and splitting services only "if measured load requires it."

- Transactional outbox for email delivery: NOT RECOVERABLE FROM PLAN

- Static versioned assets served from a CDN: NOT RECOVERABLE FROM PLAN

- Server-owned canonical state: The plan says the server owns identity, vectors, mood, simulation time, notebook records, invitation grants, and event ordering. The why is to keep authoritative state off clients and prevent old state writes.

- Browser-owned transient rendering state: The browser owns interpolated poses, audio nodes, focus, local accessibility preferences not yet synced, and an unsent event retry queue. The why is to allow rendering and local interaction while keeping canonical state server-side.

- Visitor constrained render snapshot: The plan says a visitor client receives only a constrained snapshot and "has no event-write permission." The why is read-only visiting.

- Personalized snapshot returned with the HTML: The plan says the first visual frame must not wait on audio, notebook, settings, or noncritical art. The why is fast first-frame rendering for authenticated returning visits.

- Private, non-cacheable personalized responses with only static assets cached: The risk section says personalized edge bootstrap can leak another account's scene if cached incorrectly. The why is cross-account cache isolation.

- Snapshot as presentation projection: The plan says to treat the snapshot as "a presentation projection, not a copy of the server's model." The why is to keep the client from becoming another model owner.

- Account partitioning by synthetic UUID: The plan says never use email as a key, log field, or telemetry dimension. The why is privacy and stable account partitioning without exposing email.

- Row lock or optimistic version check for ticks: The plan says this serializes ticks for one aviary. The why is to avoid concurrent simulation writers.

- Single-transaction tick commit: The plan says a tick applies events, updates bird state and notebook entries, advances cursor and last-tick time, and increments version in one transaction. The why is consistency between canonical state, processed events, and snapshot version.

- Idempotent retried worker jobs: The plan says retries must be idempotent and deterministic tick seeds must prevent extra calls, drift, or notebook entries. The why is recovery without double-applying simulation.

### 3. Data model

- Versioned schemas and server-generated UUIDs: The plan later says calibration changes roll out behind versioned parameters and stable IDs survive deployments. The why is evolution without destructive bird resets.

- Account encrypted verified email: The plan specifies encrypted email and says not to log link secrets or use email as telemetry/key material. The why is privacy.

- Per-device session records with revocation timestamps: The plan supports listing and revoking device sessions. The why is secure, revocable multi-device access.

- Magic-link token hashes, expiry, use time, and rate-limit state: The plan says links expire, are single-use, and secrets are never logged. The why is secure authentication without exposing link secrets.

- Aviary state version, last completed tick, next due tick, event cursor, and compact scene/weather state: The plan uses these fields to run ordered ticks, unchanged snapshot cursors, and projection rendering. The why is to coordinate simulation and snapshots.

- Bird stable UUID, species, user name, adopted timestamp, mood, perch/pose, call-signature identifier, and simulation schema version: The why is stable bird identity and recognizable behavior across sessions and simulation deployments.

- Server-only normalized personality vector: The plan repeatedly says clients never submit or persist trait values and personality values are not exposed. The why is to keep traits authoritative and out of UI.

- Renaming changes only the name: The why is that names and species remain attached to stable bird identities and renaming must not alter traits or identity.

- Interaction event idempotency UUID and server sequence: The plan says a lost response can be retried without double-applying an offer or presence interval. The why is ordered, idempotent event ingestion.

- Rejecting client-supplied personality or mood changes: The plan says canonical vectors, not raw history, are authoritative. The why is server-owned simulation outcomes.

- Compacting/deleting processed event payloads after commit and retry window: The plan says canonical vectors, not raw history, are authoritative. The why is to minimize retained event detail once it is no longer needed.

- Presence as short-lived server-bounded intervals: The plan says retain only enough event detail to process a tick and prevent duplicate submissions. The why is to qualify presence without storing general browsing activity.

- No raw keystrokes, pointer paths, or general browsing activity in presence: The why is explicit privacy minimization.

- Notebook entry as append-only structured facts plus rendered prose: The plan says entries are indefinitely browsable until account deletion and there is no edit/delete API. The why is a stable observational record, not an editable log.

- Invitation encrypted invitee email and hashed one-time secret: The plan says not to write invitee email or tokens to logs. The why is invitation privacy.

- Distinct visitor grants and visit sessions: The plan says this makes revocation immediate and allows each session's approximate duration to be recorded. That rationale is explicit.

- Operational telemetry as aggregate counters/histograms separate from simulation store: The plan says no account, bird, or event identifiers and separate credentials. The why is privacy-preserving operations.

### 4. API and event contracts

- Secure, same-site, revocable per-device sessions: The plan pairs this with authorization at account and bird boundaries, request size/rate limits, and idempotent mutations. The why is secure authenticated access and safe mutation handling.

- Matter-of-fact errors: The plan places this under account, settings, unsupported-browser, and error surfaces. The why is consistency with clear matter-of-fact language outside the naturalist aviary surfaces.

- Requesting and consuming a magic link: NOT RECOVERABLE FROM PLAN

- Magic-link 15-minute expiry, single use, and per-email request limits without account revelation: The why is authentication safety without revealing whether an address has an account.

- Listing and revoking device sessions: The plan stores revocation timestamps and exposes session management. The why is account control over devices.

- Verifying a new address before switching it: The why is to ensure account email changes are verified before replacement.

- Bootstrap/snapshot API: The plan says it returns current projection, version, server time, tick time, bird names/species, presentation cues, transitions, and call descriptors. The why is to draw the current scene without exposing raw personality vectors.

- Snapshot pulls on initial load, visibility return, long frame gap, and visible-tab keepalive: The why is to recover from hidden tabs, frame gaps, and concurrent-device changes.

- Version/ETag cursor for unchanged snapshots: The why is to avoid unnecessary snapshot payloads when the projection has not changed.

- Event ingestion batch with client idempotency IDs: The why is retry safety and server-assigned total order.

- Server validation of bird ownership, offer cooldowns, durations, settle state, and payloads: The why is to keep authorization and simulation rules server-side.

- Clients send facts, never state outcomes: The plan says clients send facts such as "listen-in ended" or bounded presence, not outcomes. The why is to preserve server-owned simulation.

- Notebook entries paged by cursor, newest first, with no edit/delete API: The why is a read-only, browseable field notebook.

- Account timezone and accessibility preference updates: The plan uses timezone for local day/night and mood inputs and accessibility preferences for reduced motion/captions. The why is personalized presentation and local-time simulation.

- Bird renaming API: The plan says renaming changes only the name. The why is to allow naming without changing identity or traits.

- JSON export generation with short-lived single-use links sent to the verified address: The why is account export control with limited exposure of the generated download.

- Deletion request/recovery: The plan says to exercise deletion through mail queues, grants, events, snapshots, and notebook records. The why is complete account deletion with recovery handling.

- Invitation create/list/revoke/redeem/visit snapshot APIs: The why is controlled read-only visiting with host visibility into outstanding grants and visit history.

- Rechecking grant status on every visit snapshot: The plan says revoked or expired grants return the same unavailable surface. The why is immediate revocation and non-revealing failure behavior.

- Visit log with approximate session start/end from heartbeats and last-seen time: The plan says not to trust arbitrary client-reported duration. The why is bounded, approximate visit history.

- Excluding visitor time from host presence: The plan states this directly. The why is to prevent visitors from affecting the host aviary's presence-driven drift.

- No endpoint that sets bird vector, mood, perch, or simulation timestamp: The plan says the event API rejects old state writes "by design." The why is to prevent clients from overwriting canonical simulation state.

### 5. Server simulation and drift

- Due-aviary scheduler approximately once per minute: NOT RECOVERABLE FROM PLAN

- Worker lease and committed cursor reads: The plan uses these to claim an aviary and read events after the committed cursor. The why is one worker advancing one aviary from a known point.

- Deterministic elapsed-time integration after worker delay: The plan says to integrate elapsed time rather than running "unbounded catch-up loops." The why is bounded recovery after delayed workers.

- UTC persisted times and account IANA timezone for local day/night and mood inputs: The plan explicitly includes daylight-saving changes. The why is correct local-time behavior without ambiguous persisted times.

- Folding events into qualified presence, listen-in duration, accepted offers, and terminal settle state: The why is to turn ordered raw events into bounded simulation signals.

- Deduplicating by event ID and applying cooldowns server-side: The why is retry safety and rule enforcement independent of clients.

- Mood update from persisted mood, recent events, local time, weather, and nearby calls/alarms: The plan says personality can bias probabilities but "does not replace mood." The why is layered behavior rather than personality-only behavior.

- Persisted mood across sessions: The plan says opening a tab "never resets it." The why is continuity.

- Personality drift through low-pass filter over capped, positive daily inputs: The plan says to apply small per-day deltas, clamp, and never decrement for neglect. The why is gradual, non-punitive adaptation.

- Presence as dominant drift signal; listen-in, offers, proximity, and sustained presence as trait inputs: The plan ties these inputs to social warmth, vocal frequency, curiosity, boldness, and plumage. The why is to let regular presence and gentle interactions bias behavior without exposing trait values.

- Deterministic cohorts detecting small change after roughly a week and visible differences around week three: The why is calibration of imperceptible-versus-too-fast drift.

- Greeting likelihood, perch choice, idle pose, call timing, and bird-to-bird response derived from mood, species, and vector: The why is to make visible behavior reflect canonical bird state.

- Days since owner presence used only to make greetings less frequent or more exploratory after long absence: The why is to avoid illness, distress, decay, or punishment.

- Birds continue server-side life and calls when no client is connected: The why is persistent simulation independent of visits.

- Short, rare weather events and ambient scene state: NOT RECOVERABLE FROM PLAN

- Client-side leaf/feather drift as rendering ornaments with no per-particle server state: The why is to avoid unnecessary canonical particle state.

- Notebook observation only for meaningful, sufficiently rare aviary events: The why is sparsity and naturalist specificity.

- Structured facts and prose templates for notebook observations: The why is varied naturalist prose grounded in simulation facts.

- Excluding sessions, attendance frequency, trait numbers, and generic event rows from notebook entries: The why is to avoid turning the notebook into activity tracking or stats UI.

- Atomic save of canonical state, notebook entries, processed cursor, tick timestamp, and version before publishing/cache invalidation: The why is that snapshots should only reflect committed simulation state.

- Versioned simulation configuration with seeded replay fixtures: The why is to roll out calibration changes behind versioned parameters and test deterministic behavior.

- Stable IDs and vectors surviving simulation deployments and species-pool changes: The why is continuity without destructive bird resets.

- Deterministic mood and behavior for a given tick seed: The plan says retries cannot create extra calls, drift, or notebook entries. The why is idempotent simulation.

### 6. Interaction implementation

- Return greeting selected on bootstrap: The plan selects one likely greeter using boldness, mood, and absence duration. The why is to make greeting a state-derived behavior.

- Staggering any second greeting: NOT RECOVERABLE FROM PLAN

- No welcome string, elapsed-absence UI, or toast for return greeting: The why is the quiet, observational product voice.

- Listen-in through pointer, touch, and keyboard focus: The why is access across input modes.

- Listen-in audio mix that raises the focused bird and lowers others to an ambient floor: The why is focus without silence.

- Listen-in disengage and focus transfer rules: NOT RECOVERABLE FROM PLAN

- Listen-in start/end facts: The plan says they are sent for drift and recovery after reconnect. That is the explicit why.

- Offer top-bar control rather than clicking a bird: The plan states this and later says the top bar is sparse. The why is to keep offers in a deliberate control flow rather than direct bird manipulation.

- Intended recipient selection in offer flow: NOT RECOVERABLE FROM PLAN

- Per-bird offer cooldown on the server: The why is server-side enforcement of interaction pacing.

- Offer reactions from mood and curiosity, with song fragment using vocal frequency and pool evoking drinking, bathing, or watching: The why is to make reactions simulation-derived.

- Offer as gesture, never sustenance: The why is to avoid hunger or distress mechanics.

- Settle soft evening shift and lower calls: The why is to quiet the scene.

- Five-second click-anywhere undo kept local: The plan says to send the resulting settle or settle-undo event idempotently. The why is to avoid committing until the undo window resolves.

- Settle or loss of qualified presence ending presence: NOT RECOVERABLE FROM PLAN

- Later trusted interaction resuming normal scene and presence: The why is that settle is reversible through later engagement and carries no penalty.

- Field notebook on-demand from top bar: The plan makes entries sparse and read-only. The why is to keep the notebook available without persistent scene chrome.

### 7. Rendering pipeline

- Renderer strict boundary consuming immutable snapshot cues and local interpolation time: The why is to produce frames without mutating canonical state.

- Single responsive SVG/canvas scene with compact vector bird art and DOM semantic/focus targets: The why is rendering plus accessible focus/semantics in one bounded scene.

- Keeping all birds inside the viewport at supported aspect ratios: The why is to maintain the no-panning, no-zooming horizontal scene.

- Separate background, perch planes, bird sprites, foreground, and ornaments: The plan says this lets palette/time/weather transitions happen without rebuilding bird behavior. That is the explicit why.

- No scene panning, zoom, or user placement: The why is the single bounded scene and avoidance of user-controlled perch placement/customization.

- Embedding or fetching first snapshot with the authenticated shell and preloading minimal assets: The why is to draw a frame already in progress and meet first-frame requirements.

- Returning users seeing current motion without an entry animation: The why is persistent, ongoing aviary presence rather than a re-entry ceremony.

- Initial adoption briefly showing quiet empty field before two birds arrive with a soft fly-in: NOT RECOVERABLE FROM PLAN

- Slow snapshot showing quiet field with faint ambient cue, never spinner: The why is quiet presentation even under latency.

- Thin top-bar chrome that fades after stillness and restores on input/focus: The why is sparse chrome that remains accessible when needed.

- requestAnimationFrame interpolation between snapshots: The why is continuous visible motion from discrete server projections.

- Clamping animation deltas after suspend, stopping rendering in hidden tabs, and pulling fresh snapshot on visibility restoration: The why is to recover cleanly from hidden tabs and avoid stale or runaway animation.

- Idle movement continuous in visible tabs within 60 fps budget: The why is alive motion while meeting performance gates.

- Day/night palette following local time: The why is local-time presentation tied to account timezone.

- Night remaining alive through the nightjar-like species: The explicit why is that night should "remain alive."

- Reduced motion cross-fades, no leaf drift, slower color changes while preserving mood/calls/state: The why is reduced motion without changing canonical experience.

- No personality numbers, badges, meters, or hover labels: The why is to keep personality values out of product UI.

### 8. Procedural audio

- Client audio module from species motif libraries and synthesis primitives: The why is species identity with generated variation instead of downloaded recordings.

- Prefer AudioWorklet where supported: NOT RECOVERABLE FROM PLAN

- Stable bird signature plus per-event seed for scheduled calls: The plan says pitch contour, rhythm, and timbre vary while remaining recognizable. That is the why.

- Mood shaping call contour/articulation and vocal frequency shaping cadence/chorus participation: The why is for audio behavior to reflect canonical mood and trait-derived cadence without exposing values.

- Per-bird channels, chorus headroom, reusable buffers/nodes, and hard cap on simultaneous voices: The why is bounded audio performance and controlled chorus density.

- Avoiding audio loops and downloaded recordings: The plan ties this to procedural motif synthesis and later says recorded-audio fallback is not allowed. The why is generated, recognizable calls rather than canned playback.

- Call captions generated from the same seeded grammar event as sound: The why is that prose "accurately describes the audio actually synthesized."

- Captions fading with the call and placed near the calling bird: The why is to associate caption with the call without persistent UI chrome.

- WebAudio failure or denied permission switching to silence with captions on by default: The why is graceful no-audio behavior and accessibility.

- No recorded-audio fallback: The why is to avoid substituting recordings for the procedural audio system.

- Resuming audio context only on browser-permitted gesture: The why is compliance with browser autoplay policy.

- Listen-in using slow gain ramps and keeping other birds at ambient floor: The why is smooth focus without silencing the aviary.

- Bounded audio allocations and context close/reuse across hidden-tab and teardown transitions: The why is no memory growth and resource control.

### 9. Accessibility, privacy, and performance

- Polite screen-reader narration region from the same snapshot: The why is accessible narration of the same scene state.

- Narration at roughly 30-60 second idle intervals: The plan says to avoid "flooding the speech queue." That is the why.

- Narration priority for user-initiated greeting, offer reaction, and settle observations: The why is to prioritize user-initiated, salient events.

- Not announcing raw state rows or personality values: The why is the no-stats UI and naturalist narration intent.

- Complete tab/arrow/Enter/Escape navigation: The why is keyboard access to the scene and controls.

- Visible focus working in day and night palettes plus WCAG AA contrast: The why is accessibility across palette changes.

- Respecting prefers-reduced-motion and account preference: The why is to honor both system and account accessibility settings.

- Captions user-configurable and default-on if audio unavailable: The why is access to call information when audio is unavailable.

- Initial JavaScript below 2 MB gzipped: NOT RECOVERABLE FROM PLAN

- First bird visible within 500 ms on mid-tier mobile over 4G: The why is first-frame performance for authenticated returning visits.

- Idle motion at 60 fps on a five-year-old mid-range laptop: The why is smooth continuous visible motion.

- No client memory growth across a 30-minute session: The why is long-session stability, with reuse/dispose/release called out for audio, workers/contexts, and notebook rows.

- Supporting last two major versions of Chrome, Safari, Firefox, and Edge with unsupported-browser surface for older browsers: The why is clear handling for unsupported clients.

- Synthetic browser checks from common geographies and aggregate-only RUM: The why is to monitor page load, first bird, frame timing, audio-context failures, request errors, and tick latency without personal identifiers.

- Alerting when tick p99 exceeds five seconds: The why is operational detection of simulation latency.

- Excluding account UUID, email, bird ID, event payload, and per-account session history from metrics: The why is privacy-preserving telemetry.

- Disconnected analytics and simulation storage plus redacted access logs: The why is to protect credentials, query tokens, and simulation data from operational metrics/log leakage.

### 10. Delivery sequence and rollout

- Locking API/event schemas, privacy boundaries, visual state projection, and simulation parameter versioning first: The plan says to validate design/accessibility contracts before styling all states. That is the why.

- Shipping authenticated web shell, magic-link flow, account/session controls, initial two-bird records, snapshot bootstrap, and responsive moving scene before secondary surfaces: The why is to meet first-frame requirements first.

- Adding scheduled server tick, ordered event ingestion, presence qualification, mood transitions, drift, bird-to-bird behaviors, local day/night, and weather together: The why is to establish the core simulation before interaction/audio surfaces.

- Tuning drift against week-one and week-three fixtures using synthetic accounts: The why is to calibrate small detectable changes and later visible behavioral differences.

- Adding WebAudio synthesis, captions, listen-in, offers/cooldowns, settle/undo, and notebook generation together: The plan says to validate audio identity and caption matching across species and moods. That is the why.

- Adding export/deletion, verified email change, accessibility settings/narration, invite grants, visit logs, revocation, and notification preference before release: The plan says to exercise deletion through mail queues, grants, events, snapshots, and notebook records. The why is account/privacy completeness before rollout.

- Internal release, small canary, then measured expansion: The plan says expansion continues only while privacy, sync, audio, accessibility, and performance gates hold. That is the why.

- Disabling invitations first if authorization/revocation health degrades: The plan says "core aviary remains available." The why is to degrade the social surface before the core product.

- Bird count ramp through age-based eligibility only with seven-bird cap in API and database invariants: The why is to keep two at creation, enforce the cap, and avoid visit, offer, time-spent, or payment unlocks.

- Changing age thresholds from configuration after observing performance and call recognizability: The why is tuning without schema changes or reward mechanics.

- Pre-launch checks for concurrent sessions, duplicate/reordered retries, hidden-tab resume, delayed ticks, worker restarts, DST, revoked visitors, link expiry/replay, export/delete recovery, reduced motion, no-audio, and keyboard-only paths: The plan says these are "release checks to build into the engineering work, not instrumentation of personal behavior." The why is verification without tracking personal behavior.

### 11. Principal risks and controls

- Drift controls: The plan says drift may be "imperceptible or too fast," so inputs are capped, fixtures are long-horizon, coefficients adjust slowly, and weak perceived response must not be solved by rewarding clicks or penalizing absence.

- Sync controls: The plan says sync may lose or double-apply a delta, so it uses one server writer, ordered events, idempotency IDs, transactional cursor advancement, replay/recovery, and no client absolute state writes.

- Audio controls: The plan says audio may sound uncanny or repetitive, or browser policy may block playback, so species identity, motif variation, caption comparison, chorus caps, audio-independent first frame, and silence plus captions are the controls.

- Accessibility controls: The plan says accessibility can regress as scene motion evolves, so narration, reduced motion, captions, keyboard paths, and contrast are release gates reviewed with each animation or control.

- Personalized edge bootstrap controls: The plan says incorrect caching can leak another account's scene, so personalized responses are private/no-store, account auth is isolated at the edge, and cross-account cache-isolation checks are required.

- Presence controls: The plan says presence can overcount idle tabs or be spoofed, so presence requires visible + focused + recent pointer/key activity, server-capped heartbeat duration, visitor exclusion, and no raw input retention.

- Invitation controls: The plan says links can be forwarded or linger after revocation, so secrets are hashed, visitor credentials are scoped to one grant, revocation is checked on every snapshot, unused grants expire after 30 days, and hosts see grants and access history.

- Cross-file policy seam controls for visit emails and vector export: The plan says to keep the email exception explicit opt-in only, keep vectors out of normal UI, and get product review before general release.

- Sparse top-bar controls: The plan says the top bar must house adoption, offers, notebook, and settings while remaining sparse, so the adoption path should be prototyped early and any badge, streak, or unsolicited prompt removed.
