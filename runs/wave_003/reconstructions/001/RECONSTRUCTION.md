## System-level intent

- The plan wants an aviary that is observed, not optimized. This shows up in the hard refusals of "Gamification," "Tamagotchi mechanics," and "Social-network surfaces"; in "no surface anywhere observes the user's behavior back at them"; and in the note that the "notebook observes the aviary only."

- The product should "notice, never announce." The plan names this in the risk table and carries it through "No textual welcome anywhere," "No badges, ever," no "enable sound" toast, no "welcome toast / streak / badge," and a top bar with "no badge capability."

- Voice is governed by two registers with "zero drift." The aviary, notebook, narration, captions, and offer prompts use "Naturalist" voice: "lowercase, present-tense, bird-named, specific, no exclamation, no 'you'." Identity, errors, settings, account, sync, and revocation surfaces use "Matter-of-fact" voice because the user engages the system "as a system."

- State truth belongs to the server; the client realizes it. The plan repeats that the "Server owns" personality, mood, drift, perch selection, weather, notebook, presence accounting, adoption offers, and persistence, while the "Client owns" rendering, interpolation, audio synthesis, micro-motion, captions/narration generation, and event batching. The boundary rule is: server sends "what is true"; client decides "how it looks and sounds this exact frame."

- Continuity is a core product promise. The plan protects "stable internal bird IDs for life," mood that is "never reset on session boundaries," per-bird "voice print," and persistent personality state because a reset or lost vector "deletes the bird the user knows."

- Care must be non-punitive and monotonic. The plan reconciles "traits never move down" with neglected birds becoming quieter through non-decreasing traits plus a decaying expression estimator, so absence yields quietness but "the bird itself never regresses." Settle and tab-close are also "neither penalized."

- Privacy is architectural, not a policy layer. The plan uses synthetic UUIDs, encrypts email in exactly one place, gives telemetry "no account dimension anywhere," denies telemetry a read path to the simulation database, and refuses "anything that could reconstruct a user's relationship with their aviary."

- The v1 system is "deliberately small" and "boring with teeth." The plan chooses one region, stateless HTTP, Postgres as canonical, Redis as ephemeral only, no websockets, and a small TypeScript monorepo so correctness is kept in simple primitives rather than extra infrastructure.

- The aviary should feel alive through procedural variation rather than canned content. The plan says return-greetings are "engineered, never canned," no two greetings are identical, recorded loops would break the spell, and synthesized calls use per-call jitter while staying inside a bird's voice-print bounds.

- Accessibility is a designed surface, not a checklist. The plan explicitly rejects the "cheap version" of ARIA lists, animations-off fallback, and fixed captions; reduced motion has "its own designed aesthetic," narration has cadence and dedupe rules, and accessibility ships "in v1, full stop."

- Performance is affective. The plan treats "first bird visible" as the metric because above 500ms "the user notices a load," while below it "the aviary was simply already there." First frame, quiet-field loading, no spinner, 60fps idle, and memory stability all serve that "already alive" feeling.

- Calibration replaces per-user behavioral analytics. The plan uses synthetic in-house accounts, named drift targets, qualitative sessions, CI gates, and config-tunable constants; it says the privacy boundary "forces the method, and the method is better anyway."

## Per-feature whys

**Scope**

- Aviary scene as a single horizontal one-screen scene: The rationale is that birds are "always in frame"; the scene should "never crop a bird, never scroll, never zoom," and the user should not experience the aviary as a navigable dashboard.

- Day/night cycle anchored to the user's local timezone: The plan says the time term is driven by the user's local timezone "so the aviary shares the user's day," with evening warming and quieting and night "not a dead state."

- Rare ambient weather: The weather is "rare, quiet, short" and includes "nothing the user must notice." Its purpose is ambient variation and mood color, not an event the product announces.

- Ambient leaf/feather drift: NOT RECOVERABLE FROM PLAN

- Subtle three-plane parallax: NOT RECOVERABLE FROM PLAN

- Three perch zones: The plan treats perch as legible behavior rather than placement control. Perch choice is server-driven, and "perch is a signal the user reads."

- No UI chrome inside the scene: The plan protects the aviary surface from dashboard framing. It sets a "no-chrome-in-scene review," keeps the top bar above the scene, and says "Nothing else."

- Two starter birds with species system-assigned, not user-picked: NOT RECOVERABLE FROM PLAN

- User-assigned renameable names: NOT RECOVERABLE FROM PLAN

- Cap of 7 birds: The cap keeps the scene and audio bounded. The plan says the scene is "<=7 articulated sprites" and that "bounded chorus size (7-cap exists for this)" prevents audio uncanniness.

- Stable internal bird IDs for life: The bird row is "THE bird; never regenerated." This protects continuity, because a reset or lost vector would delete "the bird the user knows."

- Age-based adoption offers after the starter birds: Offers are tied to "aviary age" and "never visit count, interaction score, or payment" so the mechanic "never teaches that attention earns stuff."

- Server-side simulation tick: The tick runs whether or not any client is connected, which makes "the aviary continues without the viewer" an architectural fact rather than a story.

- Five-trait personality vector: The traits drive perceptible behavior probabilities across boldness, warmth, vocal frequency, plumage, and curiosity. The plan calibrates them so instruments can measure change after a week and users can see behavior shifts after about three weeks.

- Monotonic-toward-expressive drift: The plan uses this to reconcile "traits never move down" with quieter neglected birds. Traits are non-decreasing; expression decays and returns, so birds become quieter without learning to mistrust the user.

- Enumerated mood states with cross-session persistence: Mood is persisted with `mood_entered_at` and "never reset on session boundaries" so the aviary does not visibly snap on open.

- Procedural per-species call grammars: Procedural calls avoid repetition and recorded-loop artifacts. The plan says the ear catches repetition, "the spell breaks permanently," and recorded variation cannot fit the bundle budget.

- Mood-shaped idle motion: Idle motion keeps birds from being "paused-still." Mood-weighted action tables make birds do something small every 8-30s, with motion "slow and personality-keyed."

- Bird-to-bird interaction: Call response, mood contagion, and chorus behavior create "a small social system, not a row of NPCs."

- Return-greeting: The greeting is "the anchor moment" and "the entire welcome surface." It replaces textual welcome with one bird noticing the user's return in a procedurally varied way.

- Listen-in: The mix is a "re-balance, never a mute" so the interaction feels like "listening, not switching channels." Other birds are eased down but "never to zero."

- Offer interaction: Offers are not request/response because the next snapshot carries the bird's reaction. The plan says this preserves "single-writer discipline" and makes offers feel "observed rather than transactional."

- Settle with 5-second undo: Settle is engine-equivalent to tab-close and "closes the presence window cleanly." Neither settle nor tab-close is penalized, and the evening ramp can be reversed by any click within 5s.

- Presence accounting with strict three-signal conjunction: The plan is guarding against the "tab open" being counted as attention and silently inflating drift. It names this as the "exact silent-corruption failure" and caps presence server-side as a belt-and-braces guard.

- Field notebook: The notebook is sparse, read-only, and naturalist-voice so it captures "noteworthy" aviary moments without becoming feed-noise or observing the user.

- Infinite scroll-back with no archive: NOT RECOVERABLE FROM PLAN

- Magic-link sign-in: Magic links provide the single auth path, with 15-minute TTL, single-use tokens, rate limits, and neutral responses to prevent account enumeration.

- Per-device revocable session tokens: Sessions are per-device so the account surface can list devices and "revoke any device."

- Verified email change: The new address must verify before commit, while the old email works until then, so identity changes do not strand the user mid-flow.

- JSON export emailed as a link: Export is the one place personality numbers are legible because it is "the user's own data, delivered to the user only"; no UI renders those numbers.

- Soft-delete then hard-delete: The account can recover for 30 days, then the hard purge removes every row tied to the account.

- Synthetic UUID as the only internal identifier: The plan uses the synthetic account UUID for all internal references and stores email exactly once, encrypted at rest, to keep PII out of the internal model.

- Server-canonical sync: There is "one canonical record, many readers, one writer," so clients have nothing to merge, nothing to conflict, and no last-write-wins path.

- Append-only client event log: Events give the server an ordered input stream for additive deltas, idempotent ticks, and redundant reconstruction after personality-loss risk.

- Email visit invitations and read-only ambient visitor view: Social is constrained to opt-in email invites and visit mode so there are no profiles, feeds, comments, co-presence, or visitor input into the host's simulation.

- Silent visit log and host notifications off by default: The visit log is "on-demand only, no badges," and notifications default off so visits are not turned into an announcement surface.

- Screen-reader naturalist narration: Narration uses the same snapshots as the visuals so screen-reader users receive observations of the aviary, not a state-list. User-initiated events are written as observations, "never state transitions."

- Designed reduced-motion mode: Reduced motion is a "render strategy switch," not an asset swap or "animations off," so the aviary remains "the aviary in a calmer register."

- Runtime-generated call captions: Captions are generated from the same motif sequence as the emitted phrase, so captions "always match what actually played."

- WCAG AA contrast and full keyboard navigation: The plan treats keyboard paths, visible focus indicators, semantic HTML, and contrast as release-gated support so "the plain parts stay plain" and remain usable.

- Performance budgets: Bundle size, first bird, fps, and memory budgets are "enforced, not aspirational" because slow first bird, frame drops, or memory growth break the already-alive aviary.

- WebAudio with graceful silence and captions fallback: If WebAudio is unavailable, the plan chooses "graceful silence + captions on by default" because "Silence with captions beats canned audio."

- Voice governance: The two-register rule exists so naturalist surfaces stay lowercase, present-tense, and aviary-focused, while system surfaces drop out of that register for identity, settings, and errors.

- Copy-lint in CI: Copy-lint blocks "welcome," "congrat," "streak," "level," "score," "you visited," exclamation marks, and announcement framing because one leak would reframe the product as trying.

**Architecture**

- TypeScript monorepo: The plan uses one language across client, server, and shared modules so voice/prose generation and call-grammar descriptors "can never diverge in vocabulary."

- Stateless HTTP API: The API is horizontally scalable behind CDN/edge, and keeping it stateless makes multi-device sync correctness a pure function of the database.

- Partitioned sim workers: Logical shards keyed by account UUID let each shard run a sequential tick loop, and short leases let a dead worker's shards be re-claimed within one tick period.

- Postgres as canonical store: Canonical state lives in Postgres with point-in-time recovery and nightly logical dumps because personality loss is one of the worst failures.

- Redis as ephemeral only: Redis holds rate limits, token hashes, idempotency keys, and cache entries, but "nothing canonical lives here"; losing Redis is "a performance event, never a data event."

- Transactional email service: Email is auth-critical, so provider failure equals sign-in failure. The plan calls for deliverability monitoring and a runbook from day one.

- Static edge with inlined initial snapshot: The initial snapshot is inlined into edge-served HTML to support first bird visible under 500ms and make the aviary feel already there.

- Client/server render boundary: The server never sends frame data, and the client never sends state. This keeps persistence and simulation authoritative while letting rendering and sound vary frame by frame.

- No websockets at v1: The plan says the canonical cadence is about one snapshot per minute; 30s keepalive polling plus pull-on-visible covers sync with stateless HTTP, edge caching, and no connection inventory.

**Data model and API**

- Email exactly once and telemetry in a separate database: Email appears once, encrypted on the account row, and telemetry has no per-account dimension so PII and relationship history do not leak into analytics.

- Account timezone: The IANA timezone drives day/night and mood time-of-day so the aviary shares the user's day.

- Personality not serialized to clients as labeled trait numbers: The plan calls this a deliberate "non-statistics surface"; render-relevant consequences are opaque visual bands, not trait UI.

- Snapshot `state_version` and ETags: Clients discard older state and get 304s when unchanged, which makes polling cheap and prevents stale snapshots from winning.

- Immutable events: Corrections are new events, such as `settle_undo`, so the event log remains append-only and ordered.

- Neutral `POST /auth/magic-link` response: The endpoint always returns 202 with neutral copy to avoid account enumeration.

- Expired or used magic-link page: The plan uses matter-of-fact copy with the next step, "request a new link," because auth errors are system surfaces.

- Account settings toggles: Settings expose call audio, captions, reduced motion, and visit notifications because these are user-facing accessibility, audio, and social-control surfaces; visit notifications default false.

- Aviary snapshot payload: The snapshot carries canonical truth, not frames: lighting, weather, settled flag, per-bird pose and call hints, and greeting directives. The payload target is under 8KB so pulls stay cheap.

- Adoption offer species pre-selected by server: The server pre-selects species so adoption remains age-gated and quiet, not a choice surface that teaches optimization.

- Event batching via keepalive and `sendBeacon`: Events are batched with idempotency keys, and final `presence_end` uses `sendBeacon` on pagehide so the append-only log receives clean closures.

- Offer reactions through later snapshots: A cooled-down offer is accepted as an event but produces ambient acknowledgment rather than an error toast, preserving the observed, non-transactional feel.

- Invite revocation on next pull: Every visitor snapshot validates invite status, and revoked visits return a matter-of-fact `visit_unavailable` surface so revocation has immediate effect.

- Visitor notebook exclusion and heartbeat-only visit tracking: The notebook is excluded because "the notebook is the host's"; heartbeats feed only the host's log and carry no behavioral payload.

**Simulation engine**

- Idempotent tick loop: The cursor and monotonic version make re-running after a crash safe, giving an at-least-once worker loop "exactly-once effect."

- Tick latency budget and p99 alarm: A tick should be a few ms of math; a p99 alarm at 5s catches degradation before users feel the aviary "run slow."

- Client presence activity window: The initial 5-minute window can calibrate longer because "watching birds without moving is the product."

- Server-side presence cap: Presence is capped at 16h/day to guard against client bugs inflating drift across the population.

- Expression estimator `E`: The estimator gives "honest expression decay" while keeping traits non-decreasing, so leaving for two weeks returns to quieter birds that have not learned mistrust.

- Drift calibration targets: The plan names testable one-week and three-week targets so drift is neither too fast like Tamagotchi nor too slow like a screensaver.

- Mood scoring with softmax, hysteresis, and minimum dwell: These keep transitions gradual, prevent flapping, and prevent mood snap across session boundaries.

- Perch selection not user-placeable: Perch probability comes from boldness, expression, and mood; the plan says user placement is disallowed because "perch is a signal the user reads."

- Return-greeting single greeter with stagger: The plan selects one greeter and staggers possible second responses so "the aviary notices one bird at a time, never a unison chorus-on-cue."

- Notebook writer sparsity governor: The writer uses noteworthy detectors, daily caps, and uniqueness dedupe to avoid "feed-noise even for very active users."

- Per-bird voice print: A stable pitch/timbre center lets Pip sound like Pip across moods and drift levels while still allowing bounded procedural variation.

**Sync model**

- Snapshot pull propagation: Clients pull on load, visibility return, render-frame gaps, and a visible keepalive so multi-device state converges without push infrastructure.

- Client interpolation: Interpolation between snapshots makes perch transitions render as flight or transition, "never a teleport."

- Structural conflict prevention: Clients cannot submit absolute personality values, events are idempotent, server timestamps order events, and the cursor lives in the same transaction as the state write.

**Frontend rendering pipeline**

- Hand-rolled Canvas 2D renderer: Canvas 2D is enough for "<=7 articulated sprites + 3 parallax planes + weather"; a framework would burn bundle budget "for no product value."

- WebGL sprite path as contingency: WebGL is budgeted only if profiling misses 60fps, not built speculatively.

- Responsive letterboxed scene: Scale-to-fit layout keeps every bird in frame across narrow and wide viewports, with no crop, scroll, or zoom.

- Weather constraints: The plan forbids thunderstorms, snow, and weather the user must notice, keeping weather rare and quiet rather than dramatic.

- Procedural bird sprite atlases: Startup-generated atlases support species silhouettes, pose sets, and plumage bands while keeping rendering lightweight.

- First frame and quiet-field loading: The initial snapshot places birds mid-pose with calls scheduled, with no entry animation, spinner, or fade-from-static; slow loading is a "quiet field."

- Thin fading top bar: The top bar contains account/settings, accessibility settings, notebook, and offer affordance only, then fades after cursor stillness to keep chrome from competing with the scene.

- No badges in top bar: The plan forbids badges including notebook or friend-visit counts so the UI cannot become a counter surface.

- Reduced-motion render strategy: Cross-fades, removed drift particles, and slowed color shifts preserve state and voice while changing how motion is rendered.

**Audio pipeline**

- Procedural audio unconditionally: Recorded loops would produce phase artifacts and repetition, while full recorded variation cannot fit the bundle budget.

- Motif library and phrase composer: Motifs, jitter, and voice-print bounds make calls varied enough to avoid repetition while preserving identity.

- Autoplay unlock without an enable-sound toast: The app resumes audio on first pointer/key event and stays silent before unlock because announcing sound would violate the no-announcement surface.

- Chorus and listen-in mixing: Overlapping birds are simultaneous synth voices, and listen-in ramps maintain other birds at a low ambient level so the experience remains a mix, not channel switching.

- Captions from the same descriptor: Captions stay synchronized with audio because they are generated from the phrase descriptor that produced the sound.

**Accessibility surfaces**

- Screen-reader cadence and throttle: Idle narration is jittered every 30-60s and deduped so it does not become a metronome or flood that users must silence.

- Shared prose generator: One isomorphic generator for notebook, narration, and captions keeps voice continuity across all naturalist surfaces.

- Keyboard navigation and visible focus: Standard key paths and soft high-contrast focus indicators make the scene and plain account surfaces operable without removing outlines.

- Unsupported-browser page: Older browsers get matter-of-fact unsupported-browser copy so the product avoids "compatibility bloat."

**Performance and observability**

- First-bird implementation path: Edge HTML, inlined snapshot, critical CSS, deferred JS, first rAF paint, and startup atlas generation are all in service of the "first bird visible" affective metric.

- Runtime allocation discipline: Object pools, pre-rendered atlases, pooled audio, virtualized notebook rows, and hidden-tab render halt enforce 60fps and no 30-minute memory growth.

- Aggregate-only observability: Synthetic fleet, RUM, sim health, and email deliverability are measured without account dimensions so operational health can be seen without relationship analytics.

- Deliberately not measured telemetry: The plan refuses per-bird state, per-account interaction history, visit-frequency per user, and account-dimension retention funnels because those could reconstruct the user's relationship with the aviary.

**Rollout and quality gates**

- Feature flags for drift, activity window, tick cadence, offer cooldown, and notebook sparsity: These are config-tunable without deploys so calibration can move without changing code.

- Private alpha, closed beta, waitlist ramp, GA: The sequence uses "in-house synthetic households" and explicit opt-in qualitative feedback, consistent with the telemetry boundary.

- Birds-per-aviary ramp intervals: The concrete schedule from third bird at about three months to a hard cap of 7 matches "a few months" and keeps adoption keyed to age, not attention or payment.

- Calibration harness: Synthetic accounts validate drift, neglect expression decay, presence caps, and mood continuity "without per-account production telemetry."

- CI quality gates: Bundle size, axe, copy-lint, keyboard e2e, reduced-motion visual diffs, narration cadence, caption matching, memory growth, tick fault injection, and snapshot contracts make the plan's constraints enforceable at every merge.
