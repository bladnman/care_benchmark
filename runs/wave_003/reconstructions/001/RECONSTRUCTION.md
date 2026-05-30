## System-level intent

- Preserve a single, slow, canonical continuity. The plan repeatedly treats the server as the authority: "one canonical server-authored state," "slow tick," "simulation layer owns all personality and mood mutation," and "server is the single writer for personality, mood, and scene state." The intent is that the aviary continues over "days and weeks" and remains consistent across sessions and devices.

- Make the birds feel alive without becoming "a game, chore, or social network." This shows up in the executive summary, the explicit exclusions for "scores, streaks, badges," "hunger, decay, penalties for absence," and social-network surfaces, and in drift rules such as "Neglect does not apply negative trait deltas." The product is meant to be affective and continuous, not coercive.

- Treat presence as the honest dominant signal. The plan says "Presence is the dominant drift input" and "must be measured honestly from visibility, focus, and recent user activity together." The presence algorithm, guardrails, and risk section all reinforce this: "No credit when tab is hidden or window unfocused," "No retroactive reconstruction," and "Require all three signals for qualification."

- Keep privacy and tone boundaries hard. The plan states that "per-bird interaction history stays inside the simulation system," "telemetry excludes per-account bird-state analytics," and "system/error surfaces use matter-of-fact voice while product surfaces remain naturalist." This becomes a platform rule in scope, an API error-surface contract, aggregate-only observability, and the acceptance criterion that no telemetry path contains per-bird interaction history outside the simulation domain and user-requested export.

- Narrow scope to protect affective quality. The product is "web-only, single-account, single-aviary," "not implemented in v1 as a real-time multiplayer or native application," and the architecture "deliberately narrows scope to protect affective quality." The rollout also delays quiet social and optional surfaces until the core illusion, budgets, and accessibility are validated.

- Render semantic state locally rather than streaming raw animation. The render boundary says the server sends "semantic scene state plus timing anchors" and "never streams raw animations." The stated reasons are small payloads, "sub-500ms first bird," and allowing "reduced-motion/audio settings to vary per device without changing canonical state."

- Prefer honesty over fake continuity. In degraded connectivity the plan says "prefer honesty over faux continuity," continue ambient rendering only briefly from the last snapshot, and "do not allow extended offline simulation that later merges personality changes." Sync status belongs in top-bar/settings, "not in aviary scene."

- Make accessibility a first-class product surface, not a degraded fallback. Reduced motion has "its own QA coverage," narration and reduced-motion are "dedicated product surfaces with PM/design review," and acceptance requires screen-reader narration, reduced-motion, captions, and keyboard controls to "preserve the same product feel rather than acting as degraded fallbacks."

- Keep the interface quiet, naturalist, and sparse. The plan calls for a "quiet field" instead of a spinner, "continual low-amplitude motion, never UI-like flourish," a sparse top bar that "fades nearly transparent," and a notebook kept "sparse by policy." The same restraint governs "quiet invite-only social" and the instruction to keep non-scene interactions out of the aviary plane.

## Per-feature whys

### Executive summary and scope

- Web-only browser product for modern Chrome, Safari, Firefox, and Edge: the plan connects this to a "small frontier team," a "browser product," and the decision to narrow scope rather than build native apps or real-time multiplayer, in order to "protect affective quality."

- Single-account, single-aviary product with one aviary per account: the plan ties this to the core promise and to canonical continuity. One aviary per account avoids multiple divergent states and supports the "same aviary mood/state" on phone and laptop.

- Magic-link authentication: NOT RECOVERABLE FROM PLAN.

- Two starter birds at account creation: NOT RECOVERABLE FROM PLAN.

- Expansion over aviary age up to seven birds: the cap and conservative ramp are tied to audio and clarity risk. The plan calls for headroom "with up to seven birds," recognizability testing at "2, 5, and 7 birds," and says third-bird unlock can remain feature-flagged until "audio clarity is validated in real usage." The reason for age-based expansion itself is NOT RECOVERABLE FROM PLAN.

- Server-side simulation tick roughly once per minute: the rationale is canonical continuity and correctness. The tick owns personality drift, mood transitions, event consumption, notebook candidates, and canonical state writes while clients may be disconnected.

- Snapshot-based client rendering with first bird visible within 500ms: the plan ties this to compact semantic payloads, performance budgets, and preserving the "already alive" illusion. It explicitly rejects spinner UI and uses the "quiet field" fallback.

- Return greeting: the plan says birds should "notice the user without announcing them." Greeting selection uses absence length, mood, boldness, and recent greeting history so the greeting feels responsive and avoids repetitive patterns.

- Listen-in: the plan uses listen-in to focus a bird without removing the rest of the aviary. Focused bird gain rises gradually, "other birds attenuate to ambient, never silence," and listen-in adds modest warmth and vocal-frequency drift for the focused bird.

- Offer: offers are interaction inputs that nudge "curiosity and, secondarily, boldness when approached." The plan also treats successful offers as priority narration events. A deeper product rationale for seed, song fragment, and still pool as the exact offer types is NOT RECOVERABLE FROM PLAN.

- Settle: the plan says settle is "optional ceremony only, never as a required state transition." It affects "immediate mood quieting only" and should not materially alter long-term drift, which keeps it from becoming a chore or progression requirement.

- Field notebook reading and notebook generation: notebook entries are meant to preserve sparse observations and "historical continuity." The plan says entries should come from "observation candidates, not raw events," be rate-limited so they average roughly one entry every few days, and store final prose so history remains stable if generators change.

- Multi-device sync through shared canonical server state: the plan says this prevents client divergence and makes "phone/laptop views inherently consistent."

- Quiet invite-only social via read-only visitor sessions: the feature gives social access while avoiding social-network surfaces. Visitor sessions "never write presence or interaction events into host drift inputs" and must be "clearly read-only."

- Accessibility surfaces: the rationale is preserving the same product feel for screen-reader narration, reduced-motion, keyboard support, call captions, and AA contrast rather than making them stripped fallbacks.

- Operational telemetry excluding per-account bird-state analytics: the rationale is to observe performance, tick correctness, and accessibility quality while preserving the privacy boundary around per-bird state and interaction history.

- Excluding native apps, gamification, Tamagotchi mechanics, social-network surfaces, shared aviaries, customization, and recorded-audio fallback: the plan ties these exclusions to avoiding a game, chore, or social network, keeping scope narrow, avoiding penalties for absence, and protecting privacy, tone, audio, and affective quality.

- Treat settle as optional ceremony only: the rationale is explicitly to avoid a required state transition and prevent settle from materially altering long-term drift.

- Single-screen scene with no panning or zooming: the plan states the product call but does not articulate a specific rationale beyond the broader quiet, narrow scope. NOT RECOVERABLE FROM PLAN.

- Sparse notebook policy: the rationale is that event volume should not create denser logging; entries should appear only when "something specifically noteworthy occurred."

- Voice split as a design-system and content-pipeline platform rule: the rationale is to keep matter-of-fact system/error surfaces separate from naturalist product surfaces.

### System architecture

- Three-tier architecture with web client, application API layer, and simulation/persistence layer: the plan separates rendering and event capture, authenticated API responsibilities, and simulation-owned state mutation so canonical state remains server-authored.

- Web client responsible for rendering, WebAudio, presence, event submission, listen-in, reduced motion, narration, and captions: the rationale is local conversion of semantic state into device-specific motion and audio while the server keeps canonical personality, mood, and scene state.

- API gateway for account, snapshots, ingestion, notebook, settings, export, invite, and visitor access: NOT RECOVERABLE FROM PLAN.

- Simulation worker for drift, mood, greeting seeds, notebook candidates, and bird-age unlocks: the rationale is that the simulation layer owns all personality and mood mutation and runs scheduled ticks from recent events.

- Notification worker for magic-link, invite, export, and optional visit emails: NOT RECOVERABLE FROM PLAN.

- Relational primary store plus object/blob store for exports if needed: the relational store supports canonical state, append-only events, row locks, versioning, and transactions. The specific rationale for object/blob storage beyond exports if needed is NOT RECOVERABLE FROM PLAN.

- Observability stack with aggregate metrics, tracing, and logs scrubbed of PII and per-bird state: the rationale is operational visibility without violating the privacy boundary.

- Static assets from CDN and stateless API behind load balancer: NOT RECOVERABLE FROM PLAN.

- Simulation workers from durable queue or scheduler partitioned by account UUID: the rationale is correctness and avoiding concurrent processing for the same account.

- Row-level locking/versioning around simulation writes: the rationale is to protect canonical state from conflicting simulation writes.

- Regional deployment first with edge caching only for initial HTML/bootstrap payload: NOT RECOVERABLE FROM PLAN.

- Server sends semantic scene state plus timing anchors, never raw animations: the rationale is small payloads, sub-500ms first bird, and per-device reduced-motion/audio variation without changing canonical state.

### Data model

- Append-only event tables plus canonical state tables: the rationale is ordered event consumption, idempotent retries, one-transaction tick processing, and server-only canonical mutation.

- Account entity with encrypted email and hash lookup: the plan grounds this in privacy by saying email is stored only on account and invite records in encrypted form and never used as a join key elsewhere. The rationale for every individual account field is NOT RECOVERABLE FROM PLAN.

- Session entity: NOT RECOVERABLE FROM PLAN.

- Aviary entity with current state version, settled state, last snapshot time, and local timezone: state version supports snapshots and sync; local timezone drives time-of-day. Other field-level rationales are NOT RECOVERABLE FROM PLAN.

- Bird entity with stable `bird_id`: the rationale is that bird ids are "never replaced, even if names or species rendering rules evolve."

- BirdPersonality scalar normalized floats with guardrails and min/max bounds: the rationale is bounded server-authored drift and damped trait changes.

- BirdMoodState carrying mood and context: the rationale is that mood persists between sessions and is "never reset by client open."

- AviarySceneState and BirdSceneState: the rationale is canonical semantic scene state that can be rendered and interpolated locally.

- InteractionEvent: the rationale is append-only event ingestion for presence, listen-in, offers, settle, and visitor audit, with processed markers and tick batch ids for idempotent simulation consumption.

- `visitor_session_started` as audit only, never drift: the rationale is that visitor sessions cannot mutate host drift or presence.

- PresenceWindow as an optional derived table: the plan gives the rationale as tick efficiency and says it is derived from presence pings, "never directly client-authored," to preserve honest presence.

- NotebookEntry storing final prose: the rationale is to "preserve historical continuity if generators change later."

- Invite and VisitSession entities: the plan grounds them in read-only visitor sessions, revocation, visit log, and optional visit notifications. Field-level rationale is mostly NOT RECOVERABLE FROM PLAN.

- ExportRequest: NOT RECOVERABLE FROM PLAN beyond support for user-requested export.

- One aviary per account: the rationale is single-aviary scope and consistent canonical state.

- Personality vectors mutate only in server-side simulation transactions: the rationale is avoiding client divergence and maintaining the server as the only authority.

- Mood carries across sessions and is never reset by client open: the rationale is continuity.

- Visitor sessions never write presence or interaction events into host drift inputs: the rationale is read-only social that cannot affect host bird drift.

- Email not used as a join key elsewhere: the rationale is privacy.

### API surface

- HTTP JSON APIs rather than real-time streaming: the plan says real-time streaming is unnecessary because "the tick is slow and clients can poll lightly."

- Auth and account endpoints, including magic link, email change, export, delete, and delete cancel: magic-link rationale is NOT RECOVERABLE FROM PLAN; export/delete supports account flows and user-requested data/deletion, but deeper rationale is NOT RECOVERABLE FROM PLAN.

- `GET /aviary/bootstrap`: the rationale is to return the initial state snapshot, account rendering settings, notebook preview, and top-bar flags in one load path for first render.

- `GET /aviary/snapshot?since_version=<n>` with `304` or compact no-op: the rationale is latest canonical state with reduced bandwidth when unchanged.

- Read-only paginated notebook endpoint: the rationale is reading sparse notebook entries without making notebook an event stream.

- Batched `POST /aviary/events`: the endpoint lets clients submit interaction events while the server validates schema, ownership, cooldowns, and event semantics. The specific rationale for batching is NOT RECOVERABLE FROM PLAN.

- Presence event payload containing visibility, focus, and recent activity: the rationale is honest presence qualification.

- Offer catalog and offer convenience endpoint: server-side cooldown availability and validation protect event semantics. The specific rationale for a catalog endpoint is NOT RECOVERABLE FROM PLAN.

- Invite creation, invite list, revocation, visit token consume, and visitor snapshot: the rationale is quiet read-only social with revocation and visitor authorization separated from host mutation.

- Accessibility settings endpoints: the rationale is user-toggleable reduced motion, captions, audio, and narration settings, with persisted account or device behavior.

- Error-surface contract with matter-of-fact copy keys and structured remediation hints: the rationale is to enforce tone boundaries and keep product-surface prose out of system endpoints.

### Simulation engine design

- Tick steps loading events, deriving presence, computing drift, mood, scene, notebook, and transactionally persisting state: the rationale is canonical server-authored continuity from recent events.

- Partitioning work by account and preventing two workers from processing the same account concurrently: the rationale is sync correctness and no duplicated or conflicting drift effects.

- Idempotent ticks via batch watermark and event processing markers: the rationale is safe retries without duplicate effects.

- Presence pings every 60-120 seconds only when visible, focused, and recently active: the rationale is strict honest presence and defense against over-crediting background tabs.

- Server discarding pings that do not satisfy all conditions, de-duplicating overlaps, and capping implausible continuity: the rationale is to prevent buggy clients from corrupting drift globally.

- Additive server-authored drift deltas: the rationale is measurable but gradual personality change controlled by the simulation.

- Presence-time as dominant drift input: the rationale is the plan's central "dominant drift input" principle.

- Listen-in drift adding warmth and vocal-frequency weight: the rationale is to make focused attention modestly shape the focused bird.

- Offers nudging curiosity and boldness: the rationale is to let approached offers influence traits without a large or punitive mechanic.

- Settle affecting immediate mood quieting only: the rationale is to keep settle ceremonial and prevent it from materially altering long-term drift.

- No negative trait deltas from neglect: the rationale is avoiding Tamagotchi-style absence penalties and chore pressure.

- Low-pass filtered drift with upper-bound damping: the rationale is no visible jumps session-to-session, measurable change after about one week, and user-visible change after about three weeks.

- Mood finite-state machine with probabilistic weighting: the rationale is mood that persists, changes legibly in motion and calls, and responds to recent interactions, local time, weather, neighboring calls, personality, and settled state.

- Bird-to-bird behavior through lightweight social coupling: the rationale is emergent response, wary spread, chorus windows, and greeting variety without turning into heavy simulation.

- Greeting seed from absence length, mood, boldness, and recent history: the rationale is responsive return greetings that avoid repetitive patterns.

- Time-of-day from account local timezone: the rationale is local day/night behavior.

- Rare weather from deterministic randomness seeded per aviary/week: the rationale is weather that feels coherent but "not synchronized globally."

- Weather affecting mood and call frequency temporarily but not permanent drift: the rationale is ambient variation without long-term personality mutation.

- Notebook entries generated from observation candidates, not raw events: the rationale is to capture noteworthy moments rather than log activity volume.

- Sparse notebook rate limiting: the rationale is sparse continuity and avoiding over-documentation even for active accounts.

- Deterministic template/rules only for notebook prose in v1: the plan recommends this for "consistency, latency, and privacy."

### Sync model

- Server-only writer for personality, mood, and scene state: the rationale is to prevent client divergence and keep phone/laptop views consistent.

- Clients only read snapshots, submit interaction events, and render/interpolate locally: the rationale is canonical sync with local presentation.

- Bootstrap snapshot, visibility-regain refresh, resume refresh, and low-frequency keepalive: the rationale is freshness after suspension and lightweight polling while visible.

- `state_version` and `simulated_at` in snapshots: the rationale is detecting unchanged state and grounding snapshot age.

- No last-write-wins on personality or mood: the rationale is avoiding overwritten drift or mood effects.

- Client event ids are idempotent: the rationale is retries without duplicate effects.

- Server-side offer cooldown validation: the rationale is authoritative interaction semantics.

- Visitor sessions with separate auth context and no access to host event ingestion: the rationale is read-only visitor access that cannot mutate host state.

- Brief local rendering during connectivity loss: the rationale is ambient feel from last snapshot without pretending to advance canonical personality.

- Short retry window for queued interaction events: the rationale is resilience for brief connectivity loss. The exact threshold is NOT RECOVERABLE FROM PLAN.

- Matter-of-fact sync status in top-bar/settings, not aviary scene: the rationale is tone split and preserving the aviary scene.

- No extended offline simulation that later merges personality changes: the rationale is "honesty over faux continuity" and server authority.

### Frontend rendering pipeline

- Canvas or WebGL/WebGPU scene with DOM for chrome and overlays: the rationale is smooth interpolation, efficient layering, subtle parallax, pose blending or sprite/SVG transitions, and deterministic reduced-motion rendering.

- Separate timing loop for scene rendering and audio scheduling: NOT RECOVERABLE FROM PLAN.

- Minimal HTML/CSS shell and critical JS: the rationale is fast initial load and first-bird budget.

- Quiet field while bootstrap snapshot is in flight: the rationale is "No spinner" and preserving the product feel before the first bird appears.

- First bird drawn already mid-action: the rationale is the "already alive" illusion.

- Lazy hydration of remaining controls: the rationale is first-bird and bundle budget.

- Scene layers of sky/foliage, perches/birds, foreground leaves, top bar, and captions: the plan names the composition but gives no specific rationale beyond the broader scene and readability goals. NOT RECOVERABLE FROM PLAN.

- Three perch zones and no camera panning: the plan names front, middle, back and "Camera never pans." Specific rationale is NOT RECOVERABLE FROM PLAN.

- Idle pose libraries, perch interpolation, micro-motion ornaments, and primitive-based greetings: the rationale is "continual low-amplitude motion" and avoiding "bespoke cinematic sequences" or "UI-like flourish."

- Reduced-motion parallel render mode: the rationale is a first-class experience that keeps lighting transitions but removes continuous motion, flight paths, and leaf/feather drift, with dedicated QA.

- Sparse top bar that fades after inactivity and returns on pointer or keyboard activity: the rationale is keeping non-scene interactions out of the aviary plane and preserving a quiet scene.

- Top bar containing settings/account, accessibility, notebook, offers, and settle: NOT RECOVERABLE FROM PLAN.

### Audio pipeline

- Client-side WebAudio synthesis from motif libraries and runtime parameters: the rationale is procedural calls, stable audible identity, and avoiding recorded audio fallback.

- Stable audible identity through fixed motif family, pitch neighborhood, rhythmic tendency, and timbral contour: the rationale is call recognizability across mood drift.

- Variation in timing, call length, spacing, ornamentation intensity, and chorus response likelihood: the rationale is to keep calls alive rather than canned while preserving recognizability.

- Listen-in mix with gradual gain rise and smooth ramp back: the rationale is focus without jarring transitions.

- Other birds attenuate to ambient, never silence: the rationale is to preserve the aviary as a living group even during focus.

- Distinct procedural calls instead of looping samples: the rationale is to avoid "phasey stacking."

- Soft headroom limits on simultaneous calls: the rationale is clarity with up to seven birds.

- Mostly centered mix with light depth cues unless spatial separation does not imply scrolling geography: the rationale is audio depth without contradicting the no-panning scene.

- Captions generated from runtime grammar events actually played: the rationale is avoiding mismatch between audio and caption and keeping captions "alive instead of canned."

- Silent fallback with captions on when WebAudio is unavailable or blocked: the rationale is preserving notebook, narration, and visual behavior without recorded-audio fallback.

### Accessibility surfaces

- Screen-reader narration from scene state in naturalist prose and low-frequency updates: the rationale is meaningful state without flooding assistive tech.

- Priority narration for greeting, successful offers, settle, and major mood-visible changes: the rationale is to surface meaningful product events rather than every micro-transition.

- ARIA live regions with priority separation: the rationale is urgent vs ambient narration management.

- Keyboard navigation into top-bar controls, aviary focus mode, bird switching, listen-in, offer, and settle: the rationale is keyboard-only session completion and pointer-free access to the product.

- Visible bird focus outline against bright and dark scenes: the rationale is focus visibility across scene conditions.

- User-toggleable captions near active caller: the rationale is call accessibility tied to the actual active audio event.

- Caption fade, AA contrast, and per-bird anchor zones: the rationale is readable captions that avoid overlapping important scene elements.

- Honor `prefers-reduced-motion` on first load with explicit override: the rationale is accessible defaults plus user control.

- Per-account setting with local override fallback: the plan recommends this but says it depends on privacy/product choice. Specific rationale for per-account over per-device is NOT RECOVERABLE FROM PLAN.

- AA contrast and matter-of-fact unsupported-browser/auth/sync errors: the rationale is accessible system surfaces that preserve the tone boundary.

- Accessibility QA for VoiceOver, NVDA, TalkBack/VoiceOver mobile, keyboard-only, reduced-motion, and captions with audio disabled: the rationale is preventing accessibility modes from technically functioning while losing the product's charm.

### Performance budgets and observability

- Initial JS under 2MB gzipped, first bird under 500ms, 60fps idle, no memory growth, tick p99 under 5 seconds: the rationale is preserving the "already alive" illusion and ensuring the slow canonical tick remains healthy.

- Code splitting settings, notebook history, invite/account flows: the rationale is to keep first render and bundle budgets intact.

- Compact semantic snapshots: the rationale is bandwidth, first-bird time, and local rendering.

- Reusing audio buffers and object pools: the rationale is performance and memory stability.

- Bounding in-memory scene history and avoiding unbounded DOM nodes: the rationale is no memory growth over a 30-minute session.

- Aggregate-only telemetry for bootstrap, first-bird render, snapshots, frames, WebAudio failures, tick backlog, and API errors: the rationale is operational observability without per-account bird-state analytics.

- Excluding raw personality vectors, per-account interaction histories, bird names, and notebook text from metrics: the rationale is the privacy boundary.

- Synthetic browser runs, load tests, soak tests, and deterministic simulation tests: the rationale is validating production-like performance, event ingestion, tick scheduling, memory stability, drift, and mood transitions.

### Rollout plan, risks, and build order

- Phase 0 foundation: the rationale is schema/API alignment, voice/content rules, and early validation of first-bird and bundle budgets.

- Phase 1 single-account aviary core: the rationale is to validate first-bird and 60fps budgets before optional surfaces.

- Phase 2 interaction and continuity: the rationale is to add offers, settle, mood persistence, notebook, sync validation, settings, export/delete, and then "lock down privacy boundary and telemetry schema."

- Phase 3 accessibility and polish: the rationale is tuning greeting variety, notebook sparsity, mood readability, audio recognizability, and accessible surfaces before release.

- Phase 4 quiet social: the rationale is to add invite and visitor mode only after proving visitor sessions cannot mutate host drift or presence.

- Employee/internal dogfood, limited beta, and capped account ramp: the rationale is controlled release and validation before scale.

- Conservative bird unlock ramp and feature-flagged third-bird unlock: the rationale is audio clarity validation in real usage.

- Drift tuning harnesses with one-week and three-week targets: the rationale is avoiding drift that feels either game-like or static.

- Presence sanity checks, caps, and tests: the rationale is avoiding over-crediting background tabs and corrupting drift globally.

- Append-only idempotent ingestion, server-only writes, locks, and resume snapshot tests: the rationale is avoiding overwrite or duplicate drift effects across devices.

- Procedural motif prototype and recognizability testing: the rationale is avoiding calls that are canned, repetitive, or muddy.

- Accessibility-specific experiential QA and launch block for poor reduced-motion/narration: the rationale is preserving charm in accessible modes.

- CI performance budgets and deferred non-critical routes: the rationale is keeping first-bird under target and protecting the "already alive" illusion.

- UUIDs, log scrubbing, separated datastores, and privacy review for telemetry fields: the rationale is preventing per-bird state from leaking into logs, analytics, or service identifiers.

- Four parallel workstreams after schema/API alignment: the plan names simulation/data, rendering/audio, account/auth/settings/invite, and accessibility/QA/observability. Specific rationale beyond parallel delivery after alignment is NOT RECOVERABLE FROM PLAN.

- Weekly integration checkpoints for snapshot contract, tone boundary, perf trend, and drift trend: the rationale is keeping shared cross-cutting qualities visible during delivery.

- Build order beginning with schema/auth/contracts, simulation tests, renderer, calls, presence, interactions, accessibility, invites, and privacy hardening: the plan says this order "protects the central illusion first: continuity, presence, and live-feeling birds before optional surfaces."
