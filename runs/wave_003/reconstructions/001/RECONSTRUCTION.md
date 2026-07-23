## System-level intent

- **Build a place that continues without the viewer.** This appears in the executive summary and product one-liner: "build a place that continues without the viewer" and a "server-side simulation tick that advances whether or not anyone is watching." It also appears in the tick loop, catch-up behavior, offline mood evolution, and sync rule that "server is continuity; client is smoothness."

- **Measure attention, not clicks.** The plan says "measure attention, not clicks" and defines depth through "honest presence-time," listen-in duration, offers, and settle markers. Presence validity requires visible, focused, and recent activity, while still leaning long because "watching is stillness."

- **Never punish absence.** The one-liner says "never punish absence." The drift design says "Monotonic toward expressive," "δ ≥ 0 always," and "neglect does not decrease traits." Non-goals reject Tamagotchi mechanics such as "death, hunger, distress, decay-on-neglect happiness meters."

- **Notice, never announce.** The plan says "never announce" and later uses "Notice/never announce" as a design critique checklist. It shows up in no "Welcome back" banners, "figure is not a toast," notebook entries that "Never log user streaks or 'you visited,'" and listen-in that should "notice not announce."

- **Depth should be slow, characterized, and ungrindable.** The executive summary says depth lives in "slow personality drift." The drift goals calibrate "measurable instrument drift after ~7 days," "user-visible characterized change after ~21 days," and "Single session never loses visible 'I grinded boldness'." The risks name "too fast → Tamagotchi grind; too slow → screensaver."

- **Server-authored personality, append-only events, canonical state.** The client/server split says "Personality vector | Server only" and "Clients never send absolute trait values." The API says the POST handler "only appends; tick applies." Conflict prevention forbids "LWW wipe of traits" and "client offline cache becoming authoritative."

- **The product voice is sparse naturalist for the aviary and matter-of-fact for system surfaces.** Voice rules say "Product/notebook/narration/captions: naturalist lowercase present-tense specific," while "Auth/errors/settings/a11y settings chrome" are "matter-of-fact complete sentences capitalized." Error surfaces say "No naturalist cosplay for failures."

- **The scene should be quiet, single-screen, and low-chrome.** Scope and frontend sections require a "single-screen horizontal scene," "no pan/zoom/scroll," "No chrome inside scene proper," "Scene contains almost no text," and a top bar that fades after idle.

- **Accessibility is part of the core experience, not a lagging parity pass.** The plan says "Accessibility ships with v1, not as lagging parity." It asks for "Naturalist SR narration," "designed reduced-motion," call captions, full keyboard navigation, and success criteria where reduced-motion, SR, and captions are "charming naturalist surfaces, not dumps."

- **Privacy and analytics are deliberately constrained.** Scope says "aggregate-only telemetry." Data model and observability sections separate telemetry from simulation data, forbid analytics reads on birds/events, encrypt PII, and state a "Hard wall" that analytics DB role cannot read simulation tables.

- **Social must remain opt-in, read-only, and non-networked.** Scope says "Opt-in visit invites by email," "read-only ambient," and "default off." Non-goals reject "profiles, follows, public discovery, feeds, comments, co-presence, leaderboards, avatar visitors," and the risk section guards against "Scope fracture into social network."

- **Identity continuity protects emotional trust.** The plan says bird IDs are "stable identity forever," migrations "must never reissue bird UUIDs or reset vectors," and emotional trust breaks include "Reset birds, renumber IDs, restoring backup abnormalities." Mitigations say "never 'regenerate aviary'," soft delete recovery, and export.

## Per-feature whys

**Executive summary and scope**

- Browser-only single-aviary-per-account product: the plan ties this to "one canonical aviary per account," shared canonical records, and no merge UX; personality merges are unnecessary because absolute traits are never dual-writer.

- Two starter birds: NOT RECOVERABLE FROM PLAN

- No game loop: the plan's rationale is that depth should live in "slow personality drift" and honest presence rather than gamification.

- Small gestures: offers, listen-in, settle, and notebook are framed as ways the user "sits with them, listens in, offers small gestures, settles evenings" without creating a game loop.

- Modern web only: NOT RECOVERABLE FROM PLAN

- Email magic-link auth: the plan emphasizes 202 responses to avoid enumeration, single-use 15m tokens, revocable sessions, and matter-of-fact system voice.

- Synthetic account UUID: the rationale is privacy partitioning; UUIDs are the "only external id," logs are keyed by UUID, and FKs/partition keys use UUIDs "never email."

- Session list and revoke: the rationale is security and user control; sessions track devices and can be revoked.

- One canonical aviary per account: the rationale is sync correctness; devices are "dual readers + dual event writers" against one server row family, with no personality merges.

- Hard cap 7 birds: the plan links this to performance and audio intelligibility: chorus caps simultaneous voices for "≤7 birds," and launch success says "menagerie ≤7; maps not hang."

- Personality vector: the rationale is long-term characterized change; traits drift slowly, monotonically, and are never displayed as numbers in product UI.

- Mood enum: the rationale is expressive state continuity; mood persists across sessions, evolves while offline, and maps into motion, call, weather, and time-of-day behavior.

- Call grammar: the rationale is recognizable procedural bird identity without recorded-audio fallback; species motif family and per-bird seed bias create continuity.

- Idle motion signals: the rationale is that birds "never look paused when visible" and the place continues rather than becoming a frozen screen.

- Return-greeting: the rationale is a non-canned return experience that responds to absence and bird traits while remaining "not a toast."

- Presence accounting: the rationale is to measure "honest presence-time" and "attention, not clicks," while preventing tab-open inflation.

- Listen-in: the plan makes listen-in bird-specific attention; duration contributes to drift, and the audio mix gives a focus bird strong gain while others remain a low ambient floor.

- Offer: the rationale is a small gesture with mood/personality-shaped response and micro-impulses, constrained by cooldown to avoid grinding.

- Settle with 5s undo: the rationale is to end presence cleanly, quiet mood/lighting, provide a reversible evening gesture, and avoid penalty or nagging when tabs close.

- Field notebook read-only: the rationale is sparse, system-generated naturalist observation rather than user logging, streaks, or "you visited" records.

- Single-screen horizontal scene: the rationale is to let the user sit with one place; the plan forbids pan/zoom/scroll and keeps all birds on-screen.

- Three perch zones: the rationale is expressive placement; zone choice reflects boldness and mood, such as wary to back and bold/content to front.

- Local day/night: the rationale is continuity and mood correctness; local time shapes solar phase, drowsy/settled mood, lighting, and morning returns.

- Rare ambient weather: the rationale is environmental variation that can lower calling propensity, nudge mood, and trigger notebook entries.

- Top-bar chrome that fades: the rationale is a quiet scene with sparse controls; chrome should recede but remain keyboard/focus accessible.

- Server-only personality writes: the rationale is sync correctness and product integrity; clients never send trait absolutes and schema privileges restrict updates to the sim role.

- Client snapshot pull and event append: the rationale is canonical server state plus responsive client event capture; durable state rebounds to the next snapshot.

- Multi-device via shared canonical record: the rationale is to show the same birds and moods without merge UX while capping concurrent presence to protect calibration.

- Opt-in visit invites by email: the rationale is controlled, default-off social access without profiles, feeds, public discovery, or push by default.

- Read-only visitor mode: the rationale is to prevent visitor activity from affecting drift; visit_context is filtered out entirely.

- Visit log and revoke: the rationale is host control and audit without badges; revoke makes the next pull return "visit_unavailable."

- Naturalist screen-reader narration: the rationale is accessible parity in the product voice, fed from the same snapshot facts as visuals and not "mood: content" telegrams.

- Designed reduced-motion: the rationale is accessibility without losing the product; motion becomes still poses/cross-fades while audio, drift, notebook, and time-of-day remain.

- Call captions: the rationale is multimodal access and audio fallback; captions come from motif descriptor tokens and default on when audio is blocked or unavailable.

- Full keyboard navigation: the rationale is complete operability for listen-in, offers, settle, panels, and bird movement with visible focus rings.

- Initial JS under 2MB: the rationale is first-bird performance and release gating.

- First bird visible under 500ms: the rationale is launch feel on target device class; birds should appear before audio resume and the quiet field should paint immediately.

- 60fps idle and no memory growth over 30 minutes: the rationale is avoiding jank and leaks, especially with audio and seven birds.

- Export JSON snapshot: the rationale is that a user can recover "emotional state offline" and meet data expectations.

- Soft delete 30d then hard delete: the rationale is recovery before cascade destroy and data subject expectations.

- Aggregate-only telemetry: the rationale is privacy; observability excludes account id, bird dimensions, per-bird interaction sequences, and warehouse joins.

**Architecture**

- Monolith-first backend: the rationale is to avoid inventing "a microservices mesh for v1" and split workers only when tick fan-out requires it.

- Edge/API gateway with auth, rate limits, CDN static, bootstrap snapshot: NOT RECOVERABLE FROM PLAN

- Auth service: NOT RECOVERABLE FROM PLAN

- Aviary API: NOT RECOVERABLE FROM PLAN

- Simulation worker: the rationale is a server-side tick that advances active aviary shards and consumes the event log.

- Postgres primary store: the plan's rationale is persistence for accounts, birds, vectors, moods, notebook, invites, and sessions; a deeper product reason for choosing Postgres specifically is not articulated.

- Redis optional for session cache, rate limits, and tick leases: NOT RECOVERABLE FROM PLAN

- Event log partitioned by account and time with tick cursor: the rationale is append-only ordering and simulation consumption without direct client trait writes.

- In-process tick scheduler first, leased worker pool later: the rationale is monolith-first delivery until account scale exceeds a process soft limit.

- Transactional email provider: the rationale is limited use for magic links, visit invites, and export links only.

- CDN static SPA and assets: the rationale is performance; authenticated snapshots must not sit on public CDN without signed short-TTL URLs.

- Render pipeline state layer: the rationale is to separate snapshot store, event outbox, presence, and focus/listen-in state from simulation writes.

- Simulation presentation layer: the rationale is expressive pose and greeting mapping without writing personality.

- Render/audio layer: the rationale is smooth scene, audio, captions, reduced-motion, and focus rings while durable state remains server-authored.

- Local prediction after interactions: the rationale is low-latency animation, while durable state still rebounds to the next snapshot.

- Suggested repo layout: NOT RECOVERABLE FROM PLAN

**Data model**

- Encrypted email ciphertext and keyed email hash: the rationale is PII confinement, lookup without using email as partition key, and audited access.

- Account timezone: the rationale is local day/night, solar phase, and offline mood path from validated IANA time.

- Account settings JSON: the rationale is to store a11y preferences, visit notification opt-in, and caption defaults.

- Aviary lighting_state: the rationale is day_cycle versus settled_override so settle can quiet evenings and snapshots can carry lighting truth.

- Aviary weather_state: the rationale is rare weather with expiry, mood/call nudges, and notebook candidate triggers.

- sim_cursor_event_id, sim_version, last_ticked_at: the rationale is ordered event consumption, versioned snapshots, and tick freshness.

- Bird stable id: the rationale is identity continuity; it is "stable identity forever."

- Bird species_id: the rationale is to select silhouette, plumage palette, call grammar, idle biases, and nocturnal flag.

- Bird display_name: NOT RECOVERABLE FROM PLAN

- Bird sort_index: NOT RECOVERABLE FROM PLAN

- Bird traits: the rationale is expressive drift for boldness, social warmth, vocal frequency, plumage saturation, and curiosity.

- Bird perch_zone, pose_phase, call_seed, cooldowns: the rationale is continuity of motion/audio identity and rate-limited offers.

- Append-only interaction events: the rationale is auditable, ordered input to the tick; visitor events may be stored but filtered out of drift inputs.

- Notebook salience and trigger fields: the rationale is a sparsity budget and internal reason codes that are "never shown."

- Visit invite token hash and expiry: the rationale is high-entropy, revocable, expiring access without exposing host PII.

- Visit sessions and visit log with no badges: the rationale is duration accounting and host visibility without gamification.

- Magic link tokens: the rationale is single-use, 15-minute authentication.

- Fixed mood set: NOT RECOVERABLE FROM PLAN

- Exactly six species at launch: NOT RECOVERABLE FROM PLAN

- No rarity in species selection: the rationale is to avoid rarity mechanics; new birds use uniform or complementarity selection.

- Telemetry schema with no SELECT on birds/events: the rationale is the analytics privacy wall.

**API surface**

- Auth 202 always for magic-link request: the rationale is no enumeration.

- Consume magic link single-use with 15m TTL: the rationale is security against replay.

- Account export endpoint: the rationale is user recovery of offline emotional state and data access expectations.

- Account soft-delete and cancel: the rationale is 30-day recovery before hard delete.

- Canonical `/aviary` snapshot: the rationale is server source of truth for clients.

- `/aviary/stream` SSE or long-poll: the rationale is version bumps on tick or relevant event without requiring constant manual refresh.

- Batched `/aviary/events`: the rationale is append-only interaction capture with idempotency and server-side validation.

- `/aviary/bootstrap` for starter naming: NOT RECOVERABLE FROM PLAN

- Bird rename only PATCH: the rationale is preventing client writes to durable simulation state beyond allowed identity text.

- Server-picked adoption endpoint: the rationale is age-gated expansion without client choosing species for optimization.

- Snapshot derived rendering knobs instead of raw trait numbers: the rationale is no "optimize me" dashboard and no numerical personality exposure in product UI.

- Notebook GET with no user write APIs: the rationale is system-generated sparse naturalist entries.

- Visit endpoints separate from host endpoints: the rationale is read-only visitor snapshots, no host settings/PII, and no drift events.

- Event batching every 2-5s and flush on edges/hide: the rationale is reliable capture of presence, listen-in, offer, settle, and visibility changes.

- Client event idempotency keys: the rationale is duplicate-event prevention after retries or timeouts.

- Tick applies personality rather than POST handler: the rationale is server-authored personality and ordered simulation.

- Offer fast path mini-resolve: the rationale is low-latency animation while the tick remains source of truth for durable mood/drift.

- Rate limits: the rationale is abuse prevention for auth, events, invites, and exports.

**Simulation engine design**

- Tick loop every about 60 seconds: the rationale is the server-side simulation advancing whether or not anyone watches, with adaptive cadence under load.

- Lease and ordered event read: the rationale is to prevent duplicate workers and process events in authoritative server order.

- Filtering visitor and invalid events: the rationale is visitor activity "must not affect drift."

- Presence aggregation: the rationale is measuring honest presence windows, listen-in, offers, settle markers, and session_open absences.

- Environment advance: the rationale is local solar phase, rare weather, and settled override expiry independent of viewer presence.

- Catch-up after downtime: the rationale is avoiding frozen time while using stored events and time-of-day only, with "no invented presence."

- Drift function: the rationale is slow, measurable, visible, monotonic expressive change without grinding or neglect decay.

- Presence validity window: the rationale is to lean long because "watching is stillness" while excluding unfocused or hidden tabs.

- Attention recency envelope: the rationale is quieter greetings after absence without trait decay.

- Mood transitions with hysteresis: the rationale is personality-conditioned state that responds to time, weather, interactions, and neighbors without thrashing.

- Offline mood path: the rationale is that a morning return should not be "yesterday's dusk frozen incorrectly."

- Procedural call runtime: the rationale is species and per-bird recognizability through grammar, seeds, mood, and vocal frequency without audio samples.

- Chorus behavior: the rationale is antiphonal response when warmth and overlapping call windows make it plausible, while avoiding exact duplicates.

- Bird-to-bird response, wary contagion, and spacing: the rationale is social coupling and physical plausibility in a small aviary.

- Idle motion and perch selection server intents: the rationale is that mood and traits show through pose and depth while clients add smooth micro-variation.

- Return-greeting planner: the rationale is absence-styled, trait-weighted greeting that happens once, staggers secondary birds, and is "not a toast."

- Offers with cooldowns: the rationale is personality-shaped reaction and tiny drift impulse while preventing repeated grinding.

- Starter adoption soft fly-in: the rationale is a one-time introduction to the named birds.

- Later bird age gates: the rationale is account-age expansion over months without push notification or engagement pressure.

- Notebook generation: the rationale is sparse salience-based naturalist observation, not streak logging or visit logging.

- Identity continuity: the rationale is avoiding emotional trust breaks by never resetting UUIDs or vectors.

**Sync model**

- One canonical prop: the rationale is no personality merge conflicts because absolute traits are never dual-writer.

- Quiet field instead of spinner while loading: the rationale is a calm load metaphor consistent with the place; "Spinner as primary load metaphor" is forbidden.

- Mid-action start from snapshot poses/phases: the rationale is that birds should feel already alive, not reset on load.

- Presence monitor and audio after gesture: the rationale is browser autoplay compliance while preserving presence if watching.

- SSE or polling plus resume refetch: the rationale is freshness after ticks, visibility changes, and sleep gaps.

- Interpolation or reduced-motion cross-fade on snapshot apply: the rationale is smooth continuity without violating reduced-motion.

- Aggressive flush on hide: the rationale is preserving interaction evidence before page lifecycle loss.

- Conflict prevention rules: the rationale is to block trait wipe, offline branching, duplicate events, replayed links, visitor drift, and clock skew.

- Settled lighting override with undo: the rationale is reversible evening state and clean presence ending with no penalty.

- Multi-device presence cap: the rationale is to "avoid gaming via two devices left open" and protect drift calibration.

- System voice errors: the rationale is "No naturalist cosplay for failures."

**Frontend rendering pipeline**

- Scene layer stack: the rationale is depth, parallax, bird/offer placement, captions, focus rings, and sparse DOM controls outside the scene.

- No badges, bird tooltips, or inline labels: the rationale is no chrome inside the scene and no gamification surfaces.

- Responsive fit with three readable zones: the rationale is to maintain all birds on-screen on phones and desktop.

- First paint strategy: the rationale is first-bird speed through critical CSS, compact silhouettes, cached snapshot, and CSS-only quiet field.

- Empty pre-adopt exception with soft fly-in: the rationale is one-time starter introduction after bootstrap.

- Idle micro-motion: the rationale is birds never becoming fully still when emotive motion is allowed.

- Hidden-tab rAF/audio suspension: the rationale is performance while not claiming the simulation stopped.

- Perch, day/night, settle, and listen-in transitions: the rationale is continuous, subtle state change and "notice not announce."

- Top bar fade behavior: the rationale is sparse chrome that reappears on activity and remains focus-accessible.

- Calm naturalist color system: the rationale is no electric accents and AA text-on-chrome.

- Procedural/silhouette asset strategy: the rationale is performance and trait-modulated plumage without unique textures per trait step.

- Code-split settings, notebook, and visit admin: the rationale is initial bundle and first paint performance.

**Audio pipeline**

- WebAudio call synth graph: the rationale is procedural calls, per-bird gain, listen-in bus, subtle atmosphere, and no competing music bed.

- Motif graph synthesis: the rationale is recognizability from species motif families and per-bird seed biases.

- No PCM chirp sample loop: the rationale is avoiding looping artifacts and preserving procedural design.

- Listen-in mix: the rationale is focus without erasing the rest of the aviary; other birds remain floor "≠ 0."

- Chorus mixing with jitter and polyphony cap: the rationale is avoiding phase-locked duplicates and maintaining intelligibility up to seven birds.

- Captions from motif descriptors: the rationale is naturalist voice parity with the audio being generated.

- AudioContext blocked handling: the rationale is graceful silent operation with captions default on until unlock.

- WebAudio unavailable handling: the rationale is graceful silence plus captions and "No recorded-audio pack."

- Audio memory pooling and heap diff CI: the rationale is no memory growth over a 30-minute call stress.

- Mute not reducing vocal-frequency drift: the rationale is that muting is "accessibility/environment, not neglect."

**Accessibility surfaces**

- Slow screen-reader live region: the rationale is naturalist paragraphs from visual facts without flooding or state dumps.

- Priority narration on greeting, offer reaction, and settle: the rationale is making meaningful changes accessible while collapsing micro-events.

- Reduced motion replacement rules: the rationale is preserving the aviary's state and atmosphere while removing continuous motion and leaf drift.

- Captions opt-in and default on when audio fails: the rationale is access to calls without sound.

- Keyboard map: the rationale is fully keyboard-operable birds, listen-in, offers, settle, and panels.

- Contrast and semantic requirements: the rationale is AA chrome/captions and mutually reinforcing SR narration, captions, and visuals.

- Visit mode accessibility: the rationale is same narration in read-only mode with no offer controls exposed.

**Performance budgets and observability**

- Release-gated budgets: the rationale is preventing asset bloat, jank, long first-bird delays, heap leaks, large snapshots, and slow ticks.

- Draw birds before WebAudio resume: the rationale is first-bird feel despite browser audio restrictions.

- Aggregate RUM and synthetic fleet: the rationale is operational visibility without account id or bird dimensions.

- Deliberately unmeasured per-bird sequences and personality distributions: the rationale is avoiding content ML, streak funnels, and cross-account ranking.

- Logging lint against email fields: the rationale is preventing PII leakage outside auth redaction paths.

**Rollout**

- M0 through M6 delivery phases: the rationale is to build foundations, canonical tick, render/audio, interactions, visits, a11y/perf, and soft launch in a dependency-aware sequence.

- Birds-per-aviary ramp: the rationale is everyone starts with two and grows over months without push notification.

- Internal max-birds override: the rationale is dogfood only, not production engagement pressure.

- Day-one instrumentation: the rationale is to monitor auth, tick, snapshot, first-bird, audio, event, visit, and JS health in aggregate.

- Synthetic drift calibration dashboard: the rationale is calibration on synthetic accounts, not production user birds piped into analytics.

- Feature flags: the rationale is controlled rollout for visits, nightjar species, notebook salience, and tick cadence.

- Continuous backup and PITR: the rationale is recovery of aviary DB state and emotional continuity.

- Export verification: the rationale is offline recovery of emotional state.

- Hard delete job cascades and email suppression: the rationale is completing deletion after the 30-day window.

- Support help blurb and contact email: the rationale is matter-of-fact support without in-aviary chatbots.

**Risks, standards, and launch criteria**

- Drift calibration tests and review gates: the rationale is avoiding Tamagotchi grind, screensaver slowness, negative drift fixes, and tab-open inflation.

- Sync contract tests and schema privileges: the rationale is preventing trait APIs, authoritative offline cache, visitor drift leakage, and ordering bugs.

- Professional motif design and listen tests: the rationale is avoiding audio uncanniness such as loops, repetition, phase cancellation, resume latency, and mobile quirks.

- A11y acceptance journeys: the rationale is preventing live-region spam, reduced-motion "as off," keyboard gaps, invisible night focus, and narration dumps.

- Non-goals checklist and codeowners on UI shell: the rationale is preventing "helpful" engagement creep such as welcome toasts, streaks, push defaults, and badges.

- Performance CI budgets and adaptive quality: the rationale is avoiding asset bloat, audio leaks, seven-bird jank, and main-thread synth blocking before dropping bird recognizability.

- Privacy tests and separate analytics role: the rationale is preventing email keys, bird stories in logs, and warehouse joins.

- Tick p99 alarms and backlog catch-up: the rationale is preventing users returning to "frozen" time after worker stalls.

- Identity continuity runbooks: the rationale is avoiding emotional trust breaks from resets, renumbering, or backup abnormalities.

- Social optional refusal tests: the rationale is preventing visits from expanding into chat, feed, or broader social network surfaces.

- Voice system content rules: the rationale is keeping aviary prose naturalist lowercase while system chrome remains capitalized and matter-of-fact.

- Testing strategy: the rationale is coverage across sim, contracts, visuals, audio, E2E, perf, and privacy for the plan's highest-risk behaviors.

- Security baseline: the rationale is session/token safety, CSRF handling, invite enumeration resistance, and no public aviary id listing.

- Legal/compliance baseline: the rationale is a privacy policy that names aggregate categories and excludes per-bird state, plus export/delete expectations.

- Named component checklist: the rationale is build order inside modules from presence monitor through sim, API, scene, audio, a11y, notebook, visits, hard delete, and export.

- V1 launch success criteria: the rationale is proving the product experience end to end: two named birds, procedural calls, listen-in, offer, settle undo, notebook possibility, return continuity, multi-device, read-only visit, accessibility, no gamification, bounded menagerie, drift calibration, and telemetry privacy.

- "What implementers must not cleverly add": the rationale is that anything conflicting with "notice-never-announce, monotonic expressive drift, or server-authored personality" is "out of product—not a backlog item."
