## System-level intent

- The plan is meant to "interpret" rather than restate the product, and to make "defensible" calibration calls. This shows up in the opening note that defaults are starting calibration "unless overridden by measured user behavior in canary," then again in the "Calibration calls and defended choices" section.

- "Notice, never announce" is a load-bearing product philosophy. It appears in the hard rules, in the refusal of "Welcome back!" toasts, banners, modals, text greetings, achievements, notifications, and "friend visited!" surfaces, in the return-greeting stagger, in the no-prompt iOS audio unlock, and in the lint against banners or toast surfaces.

- The system must not punish absence. The plan calls this "Asymmetric drift": personality traits "move up on positive presence" and "never move down on neglect." The simulation section says if this fails, the product becomes "a Tamagotchi with extra steps" and user trust is gone.

- Canonical state belongs on the server. The plan repeats that the "simulation service is the spine," "server is the only writer of personality state," "the client renders. It does not simulate," and clients "only emit interaction events" to an append-only log.

- Privacy boundaries are architectural, not decorative. The service topology is justified as a "privacy and authority boundary" between per-account simulation state and aggregate telemetry. The plan also insists that telemetry "cannot read" simulation/events/accounts databases and that there is "no path for per-account simulation state to flow into aggregate dashboards."

- Identity is deliberately separated from email PII. The hard rule is "Synthetic UUID for account identity everywhere except the single encrypted email-of-record." The data model says email is never a partition key, log field, or join key.

- Calls are an affective surface and must be procedural. The plan calls procedural audio "non-negotiable," says every call is "produced fresh every time," and explicitly rejects recorded-audio fallback even when WebAudio is unavailable.

- Product voice is split by surface. The plan repeats "Naturalist voice for the product surface; matter-of-fact for system surfaces," applying naturalist voice to notebook, narration, captions, and aviary observations, while auth, errors, settings, sync conflict, and revocation stay plain.

- Accessibility is treated as part of the designed product, not a checklist. The plan says "Accessibility is a designed surface," "Reduced-motion mode is a designed surface, not a stripped fallback," and implements narration, captions, keyboard focus, contrast checks, and alternate rendering as first-class work.

- Performance budgets protect the aliveness conceit. The plan ties the 2MB bundle, sub-500ms time-to-first-bird, 60fps idle motion, and no-memory-growth tests to the product feeling alive. It says first-bird regression would make "the central aliveness conceit" collapse.

- The product should be discovered by sitting with it, not instructed into engagement. This appears in "There is no onboarding tutorial," "no tooltips explaining listen-in or offer," and "The product is meant to be discovered by sitting with it."

- Sync is not a bolted-on feature. The plan says multi-device sync is "a property of the architecture," with one canonical aviary, server-side personality state, deterministic event ordering, and derived client caches.

- Enforcement must be structural rather than aspirational. The "Architectural Invariants and Enforcement" section says the list is about what is enforced beyond "we will be careful": GRANTs, triggers, lint, CODEOWNERS, response inspectors, CI gates, network policy, IAM, and SDK rejection.

- Growth and reward pacing must not teach "more attention earns more stuff." This shows up in the deliberately slow bird-cap ramp, the refusal of streaks/XP/badges/achievements, and the note that faster bird pacing would teach the wrong lesson.

## Per-feature whys

### Scope and account foundation

- Magic-link sign-in and request-link behavior: The plan uses a 204 response with no enumeration so the surface never reveals whether an email is known; rate-limit overage gets the same "check your email" surface but sends no email.

- Per-device session management: NOT RECOVERABLE FROM PLAN

- Account export: The rationale is "the user's data is the user's"; export is framed as the one place personality numerics may appear because it is an explicit data handoff, not a UI surface.

- Soft-then-hard deletion and restore: NOT RECOVERABLE FROM PLAN

- One canonical aviary, two starter birds, cap of seven, and paced species offers: The cap and offer cadence support slow growth; the plan says faster pacing teaches "more attention earns more stuff."

- Starter-bird seeding and difference constraint: The two starters must differ by species and boldness so one greets first more often, which "gives the user something to read" from day one.

- Server-side simulation tick: It owns canonical mood, position, personality, drift, and call timing because the simulation is the product's spine and the only place that can decide that state changed.

- Horizontal scene, perch zones, day/night cycle, weather, and micro-motion: These are the affective rendering surface, meant to make the aviary feel alive while staying within the single-scene performance budget.

- Procedural client-side WebAudio calls: The plan's rationale is that calls are "procedural, non-negotiable" and must not become recorded rotations or a fallback path.

- Presence as visibility plus focus plus recent pointer/key activity: The conjunction prevents background tabs from accumulating presence while still allowing a user to sit and watch without constant input.

- Return-greeting: The rationale is to make the user feel noticed without announcing arrival; the plan explicitly rejects system-driven welcome text and uses sampled bird behavior instead.

- Listen-in: The plan makes listen-in the strongest per-second drift signal so "the user feels recognized for paying attention to a specific bird"; the audio mix rebalance keeps other birds audible.

- Five-second settle undo: NOT RECOVERABLE FROM PLAN

- Settle: Settle is a "mood-quieting signal" and "goodbye gesture," not a personality-growth input, and it affects all clients because it would be weird for phone and laptop to disagree.

- Field notebook: Notebook entries are read-only and sparse because the rule is "observations that matter, not a feed"; prose must not become generic event-log voice.

- Visit affordance: Visits are opt-in, read-only, revocable, no co-presence, and no notification by default to keep the social surface narrow and avoid profiles, follows, chat, discovery, or friend-of-friend chains.

- Last-two-major-versions browser support and unsupported-browser surface: NOT RECOVERABLE FROM PLAN

### Architecture, data model, and API

- Small service topology: The plan says it is "not building microservices for taste"; the service split draws a privacy and authority boundary between simulation state and telemetry.

- Authority gradient: Each service can write only its own state so clients cannot PATCH personality, telemetry cannot read simulation data, and notebook prose comes from simulation-event subscriptions.

- Client/server split: Client state is "purely derived and ephemeral," so losing a device cache never loses simulation state.

- Render pipeline boundary: NOT RECOVERABLE FROM PLAN

- Edge bootstrap plus regional simulation: Edge delivery and inlined snapshots serve time-to-first-bird, while regional simulation preserves cheap, consistent per-tick read/write without chasing per-region replication.

- PostgreSQL and partitioned events table: The plan chooses a relational store as system-of-record and event log; per-account monotonic event ordering is made "trivially clear."

- Opaque UUIDv7 identifiers: Account, bird, invite, visit, and session identifiers avoid using email in service messages, partition keys, log lines, telemetry records, or joins.

- Encrypted email and lookup hash: Raw email stays encrypted; a peppered HMAC supports sign-in lookup without making email the primary identifier.

- Per-account event sequence and client event UUID: The sequence gives deterministic event order; `client_event_uuid` gives idempotency so retries are safe.

- Account settings defaults: NOT RECOVERABLE FROM PLAN

- Species pool as configuration with code review: New species cannot be added by reaching into the database; they ship through configuration updates and review.

- Derived simulation snapshot and small in-memory cache: Snapshots are derived rather than persisted, with a per-account cache to avoid recomputing within a tick.

- Signed account-export URL: Export is generated to a signed expiring URL emailed to the verified address so it "never sits in a public bucket."

- Personality seeding: Beta-distributed starts avoid identical birds, leave room for upward drift, and make plumage saturation growth legible.

- API versioning through the `Accept` header: NOT RECOVERABLE FROM PLAN

- Snapshot ETag and pull cadence: ETags avoid unnecessary payloads, and server-suggested cadence lets the scene poll slower when nothing notable is pending and faster during offer/weather activity.

- Bootstrap snapshot on initial navigation: The snapshot is embedded to "remove a network round-trip" from the time-to-first-bird budget.

- Event push batching with immediate sends for sensitive events: Presence pings batch to reduce chatter, while `offer_made`, `listen_in_start`, and `settle_triggered` push immediately because they are latency-sensitive.

- Bird naming and rename endpoint: NOT RECOVERABLE FROM PLAN

- Field notebook indefinite pagination and no archiving: NOT RECOVERABLE FROM PLAN

- Visitor sessions and visit snapshots: Visitor sessions are not account sessions, grant only read access, disallow event push, and re-check revocation or expiry so the visitor cannot affect the host aviary.

- Narration API: The endpoint returns naturalist-voice prose from the same canonical snapshot the renderer reads, giving screen-reader users the same state surface at a slow cadence.

- Error envelope and code catalog: Errors use a single matter-of-fact shape and enumerated codes so no surprise codes leak into responses.

### Simulation engine

- Per-account tick cadence: Default 60s matches the product expectation, active 20s makes mood transitions responsive, and deep-idle 5min keeps time-of-day flowing without burning resources.

- Per-account scheduled jobs instead of one global tick: The plan says this keeps tick latency bounded per account and allows alarms on per-account tick lag rather than only aggregate lag.

- Transactional tick application: If a tick fails, events are not consumed and the next tick reprocesses them; deterministic event ordering makes this safe.

- Drift function: Drift is a low-pass filter over presence and interactions so change is measurable in instruments after about a week and visible to users after about three weeks.

- Listen-in drift coefficients: Listen-in has the strongest per-second drift because it should recognize attention to a specific bird.

- Settle having no drift contribution: Settle quiets mood and terminates a presence window; it does not push personality growth.

- Asymmetry rule enforcement: Code and database trigger both block negative drift because "personality only moves up" is the most affectively important invariant and protects trust.

- Mood enum of six states: `alert` and `settled` are added because fewer states would muddle day/night and post-settle distinctions, while more would dilute mood-readability.

- Mood sampling rather than fixed argmax: Sampling keeps transitions organic and avoids patterns like one bird always being content at noon.

- Mood refractory period: Transition damping prevents mood-flicker after a bird enters a mood.

- Settle ramp and stagger: Birds move toward `settled` over a slow, slightly staggered sequence so they settle in sequence rather than in unison.

- Perch offsets: Offsets avoid overlapping birds and are persisted so a bird "stays" between snapshots.

- Ambient weather probabilities and cooldown: The cooldown keeps weather "rare," only a few times a week.

- Alarm spread one-pass rule: The plan applies one retroactive re-pass and avoids iterative cascade because it does not want feedback loops.

- Chorus scheduling and pitch alignment: Overlapping calls are aligned so chorus reads as birds together, while gain reduction keeps loudness comparable to a single call.

- Notebook seed generation and rate limits: Seeds become sparse entries only when noteworthy; the stated rule is "observations that matter, not a feed."

- Return-greeting deterministic variation: The greeting bird, motif, and timing are sampled so each return is fresh but reproducible for debugging.

- Return-greeting stagger: Simultaneous chorus on cue would announce arrival; stagger preserves "noticed, one bird at a time."

- Drift calibration harness: Synthetic users and accelerated ticks make the three-week drift target testable in CI before launch.

### Frontend rendering pipeline

- Inlined snapshot and quiet-field loading: The inlined snapshot buys time-to-first-bird; the quiet-field state avoids a spinner and keeps the first surface in-product.

- Canvas2D renderer: WebGL is "overkill" and too costly for the 2MB bundle; SVG is too DOM-heavy for per-frame idle motion.

- Runtime graceful degradation: If frame budget burns out, idle micro-motion drops to 30fps so birds preen more slowly rather than the scene stuttering.

- Snapshot interpolation and no teleporting: Perch moves use flight paths and rebeziering so birds never teleport between snapshots.

- Reduced-motion renderer strategy: It is a separate module so reduced motion can be styled and tested as "its own designed surface" rather than a stripped branch.

- Top bar as HTML/CSS and code-split modules: HTML gives accessible focus management, ARIA, keyboard navigation, and standard click targets; code splitting keeps the base aviary bundle smaller.

- Visibility-hidden pause and visible resume: The renderer pauses when hidden, then resumes from the new snapshot because "the aviary continues without the viewer."

- Adoption with no tutorial or first-time tooltips: The plan says the product is meant to be discovered by sitting with it, and tooltips would solve a problem the product refuses to have.

### Audio pipeline

- Procedural motif grammar: Every call is generated fresh from species motifs, bird identity, mood, and drift; there is "no single recording" and no pre-baked rotation.

- Per-bird identifiability: Stable pitch offset, ornament preference, and envelope tendency let each bird remain recognizable across mood and drift.

- Fixed WebAudio graph topology: Nodes are reused and created once so the 30-minute no-memory-growth rule can hold.

- Chorus mixing: Pitch and timing alignment should read as "two birds together," not "two stacked tracks," with gain scaled by participant count.

- Listen-in gain ramps: Ramps are logarithmic so the effect feels like a re-balance rather than a fade, and other birds are attenuated but never silenced.

- Audio scheduler: It uses `audioContext.currentTime` for sample accuracy and avoids scheduling calls too far ahead because the simulation may revise the schedule.

- Captions: Captions come from the call schedule and procedural grammar so silent or captioned play still carries naturalist information about calls.

- Audio off and WebAudio fallback: Silence with forced captions is a designed surface and parity path; no recorded fallback is loaded.

- iOS Safari audio unlock: The aviary does not show an enable-audio prompt; audio "simply joins in" after the first gesture, matching noticing-not-announcing.

### Accessibility surfaces

- Screen-reader narration regions: Polite prose gives ongoing state, assertive regions handle user-initiated acknowledgements, and redundant prose is suppressed so the queue stays meaningful.

- Focus model and mood label: The page has no focus trap; arrow keys move among birds, and mood is named in the label because a screen-reader user cannot see it from idle motion.

- Contrast checks: Chrome, captions, and focus outlines are tested against actual morning and night backgrounds, with regressions breaking CI.

- Auth, settings, and error surfaces: They stay accessible HTML forms in matter-of-fact voice rather than naturalist prose.

### Performance, observability, and calibration

- Initial bundle budget and CI gate: The budget protects the 2MB constraint; overruns require written justification in perf debt.

- Time-to-first-bird synthetic monitoring: The bootstrap path is load-bearing, and p95 crossing 500ms pages on-call because it threatens the first aliveness moment.

- Aggregate RUM and operational telemetry: Metrics track latency, frame timing, audio errors, and tick behavior while explicitly excluding account, bird, and per-bird state.

- What the product does not measure: The plan refuses engagement-style metrics because this is "a privacy stance and a product stance," not a product where engagement is optimized.

- Presence recent-input window calibration: Three minutes is long enough for sitting and watching, short enough to avoid background accumulation; canary usage may move it toward five.

- Notebook entry-rate calibration: The rate target is about 5-10 entries in the first month for a reasonably engaged user, weighted toward early observations.

- Snapshot inlining cap: The cap prevents the critical-path HTML from bloating while still usually preserving the no-extra-round-trip path.

- Visit-session lifetime: Four hours balances letting a visitor sit for a while with revocation taking effect quickly enough.

- Light-state cross-fade durations: Fades are slow enough not to be noticed within a session but fast enough that returning after hours reflects the real shift.

### Architectural invariants, sync, and rollout

- Invariant enforcement through GRANTs, triggers, lint, CODEOWNERS, response inspection, CI, network policy, IAM, and SDK checks: The rationale is to enforce product principles beyond "we will be careful."

- Telemetry and simulation database isolation: Network policy, IAM, and no ETL job ensure per-account simulation state cannot flow into aggregate dashboards.

- Append-only event log: Events allow INSERT and SELECT, not UPDATE or DELETE; simulation tracks highest applied event separately, preserving the event history until hard delete.

- No client-side personality cache: The local cache holds only derived state and pending events so wiping it cannot erase anything that matters.

- Event-log ordering for sync: Server `received_at` and event_id order make simultaneous multi-device events deterministic for the same event sequence.

- Listen-in as per-device state: Each device can focus a different bird because the audio mix is local UI state; events only feed drift.

- Settle as account-wide state: Settle is a goodbye gesture, so all clients should show the same settled aviary.

- Conflict and stale-snapshot surface: Failed pushes retry idempotently; stale active snapshots show a quiet matter-of-fact status, not a banner.

- No account import in v1: Import would create an unclear personality-drift carryover problem and introduce an alternate write path for personality state.

- Rollout stages: Internal alpha checks instruments and affective feel; closed beta validates drift and performance; open beta shakes out scaling and observability; public launch opens signup.

- Bird-cap ramp: The deliberately slow schedule keeps the aviary from teaching that attention earns more birds.

- Configuration without live-tuning admin UI: Tunables go through code review because live tuning would be too easy to misuse and invariants are not live-tunable.

- Post-launch calibration and privacy audit: Week 1 watches perf/tick baselines, week 4 compares drift against real aggregate presence distributions, and quarter 1 checks privacy isolation.
