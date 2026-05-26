## System-level intent

- **The aviary should feel alive without becoming an obligation.** This shows up in "aviary continues without viewer," "already in motion conceit," "absence not punished," "no punitive negative drift," "no Tamagotchi mechanics," and "settle" being "opt-in" and "not required."

- **The product should prefer noticing over announcing.** The plan explicitly calls this "notice not announce" and encodes it through "no 'Welcome back!'" surfaces, "procedural, staggered" return greetings, a sparse auto-fading top bar, "sparse naturalist observations," and no toasts, modals, or banners.

- **Bird identity and personality are load-bearing, not decoration.** The opening frames "personality drift, presence signals, server-only personality writes, procedural chorus" as "load-bearing surfaces" whose failures are "silent and permanently damaging" to the "central promise." Stable bird identity, trait drift, mood, calls, and notebook observations all carry this.

- **Change should be gradual, measurable, and never punitive.** The drift design uses a "low-pass filter," bounded per-session input, calibration targets over 7 and 21 days, and a "monotonic guard" with "max(0, delta)" so lower presence only yields "near-zero drift."

- **Canonical state belongs on the server.** This appears as "canonical state always server," "server tick job ... the only writer of personality vectors and mood state," "no local simulation ownership," "no client-to-client merge," "no LWW," and "additive server deltas only."

- **Privacy boundaries are architectural, not policy hopes.** The plan says simulation data is used "only for owner's simulation," telemetry is "aggregate-only," email is encrypted "in one place only," analytics are "physically and logically separated" from simulation DB, and "no bridge" may leak vectors, names, or event sequences.

- **Social presence must not become social pressure.** The visit feature is "opt-in," "read-only," "defaults off," revocable, has "no co-presence," "no notifications," and "no effect on host drift." The risks section names the danger as "social surface pressure."

- **The voice boundary is part of the product.** Product surfaces use "naturalist lowercase present-tense specific field-notebook voice"; account, auth, error, settings, and unsupported states use "matter-of-fact" copy. The plan repeatedly separates "naturalist prose" from account surfaces.

- **Accessibility is a designed mode, not a fallback.** The plan calls accessibility "first-class, not checklist," makes reduced motion a "separate renderer path" and "separate aesthetic," and keeps narration, captions, keyboard, contrast, and screen-reader behavior in the same product voice system.

- **Procedural variation should preserve recognizable signatures.** Calls, chorus, captions, notebook text, idle motion, and mood transitions all use procedural systems. The audio goal is that every call is "unique" while a "recognizable signature persists across weeks of drift."

- **Performance is part of presence.** The plan ties bundle, first-bird, frame-rate, memory, tick, and Canvas choices to the "already in motion conceit"; performance failure would break the field-like illusion.

- **Refusals are treated as hard invariants.** The non-goals are described as "load-bearing architectural invariants," and the closing says constraints are "encoded into architecture, not left as policy hopes."

## Per-feature whys

### Scope - v1 inclusions and explicit exclusions

- **Two starter birds per new account:** The plan ties this to "stable bird identity across life of account" and to system selection "not user catalog," keeping the first experience inside the aviary rather than a catalog choice.

- **Maximum 7 birds per aviary and new birds offered on aviary-age schedule:** The plan says growth is "paced by relationship depth, not visit metrics" and later says new birds are "not per-user reward," with "no visible counter or 'unlock' narrative."

- **User-assignable names and renameable birds:** The plan articulates stable identity, but not a specific rationale for renameability: NOT RECOVERABLE FROM PLAN

- **5-trait personality vector:** The plan uses the vector as the substrate for drift, greetings, perch, motion, plumage, calls, and visible change. Its rationale is to make personality change measurable internally while never exposing vectors numerically.

- **Fast-timescale enumerated mood states:** The plan uses mood to shape idle micro-motion, calls, offers, daypart, weather, and settle. Mood gives short-term variation while personality remains slower drift.

- **Monotonic drift:** The plan says drift is "positive on presence," has "no punitive negative drift," and neglect only yields "near-zero drift," preserving "absence not punished."

- **Procedural call grammars per species:** The plan wants calls that are unique per instance yet retain "recognizable signature"; it also rejects recorded loops because of "bundle and aesthetic contract."

- **Mood-shaped idle micro-motion:** The plan uses this to make birds visibly different by mood/personality and to support the "already in motion" promise without entry tween.

- **Return-greeting:** The plan describes it as "procedural, staggered, absence-length + boldness + mood aware," using bird state to create a noticed return without "Welcome back!" announcements.

- **Listen-in:** The plan says listen-in is a "gradual re-mix" where "other birds never reach true 0 gain," preserving an ambient chorus rather than isolating one bird into full silence.

- **Offer with seed / song-fragment / still-pool:** The plan requires offer types, cooldowns, and mood-different reactions, but does not explain why these three offer forms specifically: NOT RECOVERABLE FROM PLAN

- **Per-bird offer cooldowns:** Cooldowns are "enforced server-side" so the snapshot remains authoritative and offers cannot be spammed into state drift.

- **Settle:** The plan calls settle an "opt-in soft goodbye," "not required," with "undo window" and lighting/call changes. Its why is to provide a gentle exit without obligation or punishment.

- **Field notebook:** The plan frames it as "auto-generated sparse naturalist observations" and an "observer record only," making it a naturalist record rather than editable creation or progress tracking.

- **Strict 3-signal presence accounting:** The plan uses visible state, focus, and recent pointer/key activity for drift. It also says never trust client cumulative time; the server reconstructs windows, protecting drift from false presence.

- **Single horizontal responsive aviary scene:** The plan says all birds must remain on-screen with "no cropping," and explicitly rejects panning, scrolling, zooming, and "multi-screen geography."

- **Three perch zones:** The plan names front, mid, and back perch zones but does not explain why three specifically: NOT RECOVERABLE FROM PLAN

- **Local-time day/night cycle:** The plan uses day/night for palette, call volume, mood priors, and time-of-day envelope, giving the aviary a field-like rhythm.

- **Rare ambient weather:** Weather creates rain/wind overlays, mood side-effects, noise layers, and notebook-worthy salience while remaining rare and ambient.

- **Subtle parallax and leaf/feather drift:** The plan includes these as ambient motion in the scene, but does not articulate a feature-specific why beyond the broader "already in motion" intent.

- **Top bar with sparse icons and auto-fade:** The top bar is limited and fading because chrome must not violate "no UI inside aviary" or "notice not announce."

- **Email magic-link sign-in:** The plan uses 15-minute, single-use, rate-limited magic links with no passwords in v1, supporting a matter-of-fact, low-account-friction auth surface.

- **Per-device revocable session tokens:** Sessions are revocable server-side so devices can be controlled and invalidated without client-owned authority.

- **Synthetic UUID account IDs and encrypted email in one place only:** This implements the "synthetic ID rule everywhere" and keeps PII out of logs, partition keys, telemetry, and simulation paths.

- **Account export:** Export emits deterministic JSON of birds, vectors, moods, notebook, and visit log. The plan does not state a product why beyond the account surface and privacy/data handling: NOT RECOVERABLE FROM PLAN

- **30-day soft-delete then hard-delete:** The plan specifies soft flag and cleanup after 30 days, but does not articulate why the period is 30 days: NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick:** The tick is the only writer of personality and mood, supports deterministic replay, ordered event consumption, and multi-device sync without client conflicts.

- **Clients pull snapshots and append interaction events:** The client stays a "thin state consumer" with transient render state only; this protects canonical personality and mood from local-clock or suspension errors.

- **Multi-device sync:** The plan makes sync a "property of architecture" with no client-to-client merge and no LWW so additive server deltas avoid personality conflicts.

- **Visit feature:** Visits are read-only, opt-in, revocable, expiring, without co-presence or notifications. The plan says this prevents "social surface pressure" and keeps visitor activity from mutating drift inputs.

- **Screen-reader running naturalist narration:** Narration gives slow-cadence state awareness in the same naturalist voice, prioritizing user events without duplicating every idle state.

- **Reduced-motion designed cross-fade renderer:** The plan says this is not "animations off" and not degradation; it is a designed surface with pose-to-pose cross-fades.

- **Call captions:** Captions derive from active grammar instances so audio has a naturalist prose equivalent and forced-on fallback when sound is unavailable.

- **Full keyboard navigation and visible focus ring:** Keyboard paths and high-contrast focus make the aviary operable without pointer input across all aviary states.

- **WCAG AA contrast on all text:** The plan uses contrast tokens so text surfaces remain readable while the scene itself avoids fine text inside aviary art.

- **Initial bundle <= 2 MB, first bird <= 500 ms, 60 fps, no memory growth:** These budgets protect the "already in motion" conceit on mid-tier mobile and older hardware.

- **Last two major versions of Chrome, Safari, Firefox, Edge:** The plan names browser support and graceful older-browser surface but does not explain this support range specifically: NOT RECOVERABLE FROM PLAN

- **Privacy rule for interaction data:** The plan says per-bird/per-account data is used only for the owner's simulation and never for training, recommendations, sharing, or individual analytics.

- **Aggregate operational telemetry only:** Telemetry is limited to request counts, latencies, histograms, and errors so observability does not expose bird names, vectors, or event sequences.

- **Observability and error surfaces in matter-of-fact voice:** The plan separates operational/account surfaces from naturalist product surfaces to maintain the voice boundary.

- **Rollout instrumentation from day one:** The plan needs early drift calibration, performance tracking, and alerts before launch, because load-bearing failures are silent.

- **Explicit exclusions:** The plan treats no native apps, no gamification, no Tamagotchi mechanics, no social network surfaces, no notifications, no numeric personality vectors, no panning/zooming/scrolling, no editable notebook, and no welcome announcements as architectural invariants rather than optional policy.

### Architecture

- **Thin browser client:** The client is a state consumer, renderer, and audio synthesizer so it can render richly without owning personality or mood state.

- **Aviary Simulation Service:** The service owns auth, event storage, ticks, snapshots, and visits so personality writes, mood, and causation stay centralized.

- **Append-only interaction event store:** The event log is "source of truth for causation" and enables replay, tick consumption, and corruption checks.

- **Notebook entries as immutable rows:** The plan uses immutable generated entries to preserve the notebook as observer record, not editable user content.

- **Presence aggregate windows:** The plan stores aggregates rather than raw mouse events, supporting drift while limiting privacy exposure.

- **Visit invites and visit log:** Invite state enables opt-in, expiry, revocation, and logs without co-presence or social feed behavior.

- **Static client assets on CDN plus containerized simulation workers:** This supports small static delivery for the browser and horizontally scaled server ticks.

- **Postgres or equivalent row-level/per-account isolation:** The plan wants isolation plus easy export/delete and no cross-account simulation queries.

- **Client/server render boundary:** Server owns authoritative state; client owns only interpolated positions, mix levels, clock, and animation timers so suspend/resume requires fresh snapshot.

- **TypeScript plus lightweight framework:** The plan chooses this to hit the bundle budget and avoid heavy UI frameworks.

- **Canvas 2D rather than WebGL:** The plan says Canvas is sufficient for 60 fps, smaller than three.js, and easier for accessibility integration and procedural drawing.

- **Compact vector paths or tiny sprites:** The plan wants no heavy textures and uses tinting to keep assets small while allowing plumage saturation.

- **Vite or esbuild:** The plan cites tree-shaking and code-splitting for the <2 MB target.

- **Go simulation core:** The plan says Go offers deterministic math, low memory, fast cold start, and predictable performance for ticks.

- **Postgres 16+ with normalized vector columns where useful:** The plan prefers normalized columns for drift history queries during calibration while allowing JSONB if needed.

- **REST over HTTPS and no GraphQL:** REST is chosen for versioned simplicity and v1 size/perf discipline.

- **No persistent WebSocket required:** Polling or optional long-poll/SSE is enough because low-frequency snapshots meet the spec.

- **Jest, Playwright, Go tests, synthetic browser fleet:** The plan uses these for client behavior, simulation correctness, performance, and drift regression.

### Data model

- **Account entity:** Encrypted email, synthetic ID, deletion flag, and settings support privacy, deletion, and multi-device accessibility preferences.

- **Bird entity:** Stable bird_id, species, name, vector, mood, and perch zone keep bird identity persistent and snapshot-consistent.

- **Personality never reset except hard-delete:** This preserves stable identity and avoids personality loss except when the account is actually deleted.

- **Species static table or struct:** Species carries silhouettes, palettes, call motifs, and vocal base rate so visuals and audio share a recognizable signature.

- **InteractionEvent:** Append-only events normalize client/server time and record the tick that consumed or ignored them, supporting replay and ordered causation.

- **Presence pings throttle:** Presence pings are lightweight and throttled so active presence can be tracked without raw continuous input logging.

- **NotebookEntry:** Immutable naturalist prose with related birds keeps notebook output sparse, generated, and tied to bird state.

- **VisitInvite and VisitEvent:** These support revocation, expiry, read-only access, and a visit log without affecting host drift.

- **Session:** Token hashes and revocation allow per-device session control.

- **Derived presence windows:** Derived windows avoid raw mouse event storage while still feeding drift.

- **Mood timers and weather influence windows:** These let tick advance state across sessions without client authority.

- **Personality delta history:** The plan says this is optional and internal, for calibration analysis only and never surfaced.

- **Export job:** Export walks normalized rows to produce deterministic JSON. Its specific user-facing rationale is not articulated beyond the account/export surface: NOT RECOVERABLE FROM PLAN

- **Deletion cleanup job:** Hard-drops rows after the soft-delete period; the plan gives mechanics, but not why the exact retention design is chosen: NOT RECOVERABLE FROM PLAN

### API surface

- **POST request magic link:** Issues a signed single-use token and emails it; rate limiting and TTL support secure passwordless auth.

- **GET verify magic link:** Validates token and issues session pair so login state remains server-controlled.

- **Matter-of-fact auth rejection:** The plan keeps expired/invalid auth errors outside naturalist voice.

- **GET aviary snapshot:** Provides small authoritative state, server time, weather, birds, hints, and presence summary so clients can render without owning simulation.

- **POST aviary events:** Batchable, idempotent, rate-limited events let the client append interactions while the server normalizes time.

- **GET notebook:** Paginated prose entries keep the notebook readable and sparse.

- **Presence pings as events:** The server reconstructs windows and does not trust the client for cumulative time.

- **Offer cooldowns server-side:** Server enforcement makes cooldowns authoritative; client remaining time is approximate until next snapshot.

- **Visit invite/guest/revoke endpoints:** These keep visits opt-in, read-only, expirable, and immediately revocable.

- **Export, session revoke, settings, delete account surfaces:** These are account functions in matter-of-fact voice; the plan explicitly says no naturalist prose on these surfaces.

- **Unsupported and sync/auth errors:** Prescriptive matter-of-fact copy gives recovery actions such as "request new link" or "reload aviary."

### Simulation engine design

- **60-second configurable tick cadence:** The plan chooses a nominal minute tick with 45-75 second tuning for calibration, balancing steady state advancement with server cost.

- **Idempotent tick jobs:** Idempotency protects state from duplicate scheduler runs.

- **Tick inputs and outputs:** Recent events, time-of-day, personality, mood, weather, and timers produce vectors, mood, call timers, notebook entries, perch changes, and snapshots.

- **Fast tick target:** p99 limits prevent tick backlog and protect simulation freshness.

- **Seeded PRNG:** Deterministic PRNG by account and tick enables reproducible test replays.

- **Drift low-pass filter:** The filter makes changes gradual rather than jumpy, with presence as primary weight and listen/offer as secondary/tertiary.

- **Bounded per-session input:** Bounding prevents single-session jumps in personality.

- **Drift calibration targets:** The plan uses internal 7-day and 21-day targets so changes become measurable and then noticeable without prompting users.

- **Neglect modeling:** Lower presence produces "near-zero drift" and only ambient quieting, avoiding neglect punishment.

- **Mood FSM:** Time, interaction, inter-bird coupling, and personality create short-term state while avoiding forced neutral.

- **Settle bias:** Settle injects a "settled" bias that decays over hours, making the goodbye soft and temporary.

- **Call grammar runtime:** Server advances timing and client synthesizes sound, splitting canonical readiness from local audio rendering.

- **Bird-to-bird social:** Call-response, safety-in-numbers, wary propagation, and chorus make birds relate to each other without social-network features.

- **Starter species draw from six species:** The plan says no catalog UI and system draw, but does not explain why six species specifically: NOT RECOVERABLE FROM PLAN

- **Additional birds age-gated:** Stored thresholds make new birds automatic aviary growth rather than rewards or unlocks.

- **Notebook generator with templates and slot filling:** Templates avoid LLM cost, unpredictability, and privacy concerns while still varying prose.

- **Sparse notebook cadence:** At most one entry every 2-3 days plus high-salience events keeps the notebook from becoming an announcing feed.

### Sync model and conflict handling

- **Simulation DB as canonical state:** This prevents clients from mutating personality or mood.

- **Event log as source of truth:** Event causation can be replayed and tested.

- **Snapshot versioning:** Monotonic versions let the client detect lag.

- **Matter-of-fact conflict surfaces:** Auth, replay, and outage problems get explicit recovery actions instead of naturalist ambiguity.

- **No reconciliation UI in normal path:** Additive deltas and ordered consumption "preclude personality conflicts."

- **Replay property tests:** Replaying an event log against empty state must converge, verifying state integrity.

### Frontend rendering pipeline

- **Canvas scene layers:** Sky, foliage, perches, birds, branches, leaves, and weather provide depth and atmosphere within one scene.

- **Responsive all-birds-on-screen layout:** Horizontal spacing and perch depth compression prevent cropping and avoid panning/zooming.

- **First-frame mid-action placement:** The plan avoids entry tween so the aviary appears already alive on load/resume.

- **Per-bird animation state machine:** Preen, scan, weight-shift, and head-tilt express personality and mood through speed and amplitude.

- **Slow eased perch changes:** The plan says no teleport; duration is personality-shaped to preserve natural movement.

- **Continuous day/night palette and call envelope:** The aviary changes gradually with local time instead of abrupt mode switches.

- **Weather overlays with mood side-effects:** Weather becomes ambient state, audio texture, and possible notebook salience.

- **Independent leaf/feather PRNG spawners:** These add ambient motion without requiring server simulation state.

- **Reduced-motion renderer path:** Cross-fades, static poses, removed ambient drift, and slower color shifts preserve accessibility without turning the product off.

- **Quiet-field loading background:** First load/resume avoids spinner and matches the palette while awaiting snapshot.

- **Settle transition and undo:** A 3-5 second lighting wrap and call envelope down make settle soft; any interaction within 5 seconds can revert.

- **Top-bar opacity fade:** Cursor silence causes chrome to recede, keeping the aviary primary.

- **Pointer bird hit regions:** Hit-testing gives direct bird interaction inside Canvas.

- **Keyboard roving and bird cycling:** The plan provides a complete non-pointer route, with Enter for listen-in and Escape exit.

- **High-contrast focus ring:** Double stroke or luminance inversion keeps focus visible across changing palettes.

### Audio pipeline

- **Per-species motif library:** Motifs give each species a recognizable vocal base.

- **Stochastic call concatenation and pitch/time variation:** The plan wants every call instance unique while preserving signature through seeded variation.

- **Real overlapping chorus:** Independent call instances avoid phase artifacts of looping.

- **Listen-in gain automation and filters:** Slow ramps focus one source while low-pass/distance filtering keeps other birds present.

- **Ambient floor for non-focused birds:** Other birds never reach true zero, preserving the aviary as a place rather than a solo track.

- **Master bus, reverb, time-of-day filtering, weather noise:** These create environmental place and night/weather envelopes.

- **Silent WebAudio fallback with captions forced on:** If audio is unavailable, the product degrades gracefully without loading recorded audio.

- **Runtime caption generation from grammar:** Captions reflect the actual active call, not hard-coded strings, in naturalist voice.

- **Shared AudioContext and buffer reuse:** These prevent per-call allocation leaks and support precise scheduling.

### Accessibility surfaces

- **ARIA live narration:** Slow, polite, atomic narration summarizes state and prioritizes user events without flooding idle state.

- **Visible caption option:** The plan allows sighted users to access the same narration/caption information.

- **Captions toggle:** Toggle is a matter-of-fact accessibility setting; captions also support audio fallback.

- **Reduced motion responding to media query:** The design respects both global preference and per-session override.

- **Keyboard order and modal trapping only in settings:** This keeps normal aviary navigation complete while avoiding unnecessary traps.

- **Contrast design tokens:** Tokens enforce 4.5:1+ on text and avoid unreadable text inside aviary art.

- **Screen-reader idle restraint:** The plan says not to duplicate every idle state via ARIA, preventing noisy assistive output.

- **Accessible names for hidden affordances:** Hidden controls remain named for assistive technology.

- **Audio opt-out with captions defaulting on when historically selected:** This preserves user preference and audio accessibility across sessions.

- **Server-side accessibility settings:** Stored settings give multi-device consistency and apply at render bootstrap.

### Performance budgets, observability, and CI gates

- **Bundle size gate:** The <=2 MB initial JS gate enforces asset and framework discipline.

- **Time-to-first-bird gate:** The 500 ms target makes the bird visible and interactive quickly enough for the "already in motion" conceit.

- **60 fps idle and memory budget:** Long idle stability keeps the aviary from degrading while open.

- **Tick p99 alarms:** Slow ticks threaten state freshness and simulation reliability.

- **Synthetic fleet:** Rotating browser accounts measure load, first-bird, idle frame times, audio readiness, and narration emissions under controlled conditions.

- **Aggregate-only RUM:** RUM measures user-facing performance without per-account fields.

- **Simulation histograms and backlog metrics:** These detect tick duration, event backlog, drift cost, and DB query issues.

- **Drift calibration metrics:** Cohort-level deltas let the team tune drift without individual bird analytics.

- **Telemetry separation from simulation DB:** The plan calls this "absolute" to avoid leaking vectors, names, or event sequences.

- **CI regression gates:** Bundle, perf, drift monotonicity, accessibility, keyboard, and narrative voice gates prevent load-bearing regressions.

### Rollout strategy

- **Internal dogfood, closed alpha, limited beta, public gradual ramp:** Phasing supports qualitative learning, drift calibration, and infrastructure ramp without thundering herd.

- **No feature flag for birds:** The number constraint is hardcoded because bird count is a product invariant, not a rollout trick.

- **Automatic bird ramp:** New accounts start exactly two birds and later birds appear by aviary age, not visible unlocks.

- **Alpha feedback surface:** Optional, matter-of-fact, reachable but non-interrupting feedback preserves the non-announcing posture.

- **Canary accounts and cohort feature flags for server changes:** These support safe server rollout without changing the bird-count invariant.

- **Rollback compatibility and dual-write window:** Migration discipline keeps rollback possible.

- **Alerts on presence corruption, tick backlog, narration queue, audio dropouts, and first-bird regression:** These are the load-bearing failure modes the plan wants caught early.

### Testing strategy

- **Drift, mood FSM, and call grammar unit/property tests:** These verify monotonicity, invariants, and deterministic grammar behavior.

- **Event-log replay tests:** Identical inputs must produce identical personality vectors across tick versions.

- **End-to-end Playwright scenarios:** Full-session flows check greet variation, listen-in, offers, settle/undo, notebook sparseness, 3-signal presence, multi-device, visits, and deletion.

- **Chaos tests:** Suspend, revoke, replayed magic links, and network interruption test outage and edge-case recovery.

- **Audio spectrogram regression:** Species calls must remain recognizable across moods.

- **Notebook prose regression:** Template output stays within the intended prose system.

### Security and privacy engineering

- **Email PII encrypted only in account row:** This keeps email out of logs and partition keys.

- **Magic links single-use, short TTL, rate limited:** These bound login-token risk.

- **Session token revocation and constant-time comparison:** These protect active sessions and token validation.

- **Telemetry aggregate-only by construction:** Privacy is enforced in data shape, not only policy.

- **Deletion cascade measured and logged by count, not content:** Compliance evidence avoids content exposure.

- **No cross-account visibility or inference surfaces:** This supports the privacy and no-social-network boundaries.

- **Export and visit revocation audited in visit log only:** The plan tracks account actions without broad social telemetry.

### Risks, mitigations, and calibration

- **Drift calibration risk:** The plan worries too-fast drift feels "Tamagotchi" and too-slow drift "feels dead," so it uses A/A cohorts, numeric targets, server config dials, and slow documented recalibration.

- **Sync/event ordering/personality corruption risk:** Append-only log, tick versioning, no client canonical writes, replay harness, and property tests protect personality state.

- **Procedural audio artifact risk:** Studio tuning across many generated calls and variation knobs fight uncanny repetition; fallback remains silent+caption, never recorded loops.

- **Accessibility abandonment risk:** Co-design with screen-reader users, narration voice tests, and reduced-motion as aesthetic avoid broken-feeling access surfaces.

- **Performance slip risk:** CI gates, Canvas 2D, tight assets, code splitting, and daily first-bird tracking preserve "already in motion."

- **Privacy incident risk:** Synthetic ID rule, separate DBs, redaction audit, and pre-merge searches defend against email or event leakage.

- **Top-bar/chrome violation risk:** Design reviews against exact language and a four-icon/fade rule protect "notice not announce" and "no UI inside aviary."

- **Visit feature social-pressure risk:** Read-only, no presence recorded, no host notifications by default, and a "would this require mutating the drift inputs from visitor?" test keep visits from becoming social pressure.

- **Email deliverability:** The plan says use a reputable provider from day one.

- **DB growth on event logs:** Partition and archive to cold storage after N days for export only.

- **Tick leader election:** The plan recommends simple lease or job queue with single consumer for starters.

### Open questions and calls made

- **Exact drift constants:** The plan leaves constants to first-month calibration; the rationale is that targets are documented but live cohort data must tune the weights.

- **Precise species count and initial names:** Species count is set to six, but exact names/visuals/motifs are delegated to a companion design asset spec: NOT RECOVERABLE FROM PLAN

- **Tick cadence exact value:** 60 seconds nominal and tunable +/-15 seconds, because calibration may need adjustment.

- **Presence activity window:** Three minutes nominal is chosen to be "generous watching-without-mousing."

- **Notebook generator with no ML:** Template and grammar expansion are chosen for "cost/predictability/privacy."

- **Canvas vs DOM:** Canvas is chosen for motion, hit-testing precision, and single-draw-call discipline.

- **Language/framework:** TypeScript plus minimal reactive and Go simulation service are chosen for size and performance.
