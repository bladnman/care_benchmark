## System-level intent

- Calm, companion-first scope. The plan repeatedly frames Pocket Aviary as a "calm, single-aviary, browser-based companion product" and protects that calm through "no gamification whatsoever," no Tamagotchi mechanics, no push notifications, no "welcome back" announcements, and a single "quiet visit affordance."

- Honest, slow expressiveness instead of optimization loops. Personality vectors "drift monotonically toward expressiveness over weeks of honest presence," with drift that is "presence dominant, listen-in secondary, offer marginal." The plan wants "1 week regular use" to be measurable but "3 weeks" to be the user-visible character shift, and it forbids numerical trait exposure.

- Server-authoritative simulation. The product contract depends on clients being "pure renderers + event appenders" while the server owns "all personality and mood state." The architecture rationale says the "server tick + additive deltas + synthetic IDs" eliminate "two clients diverge" and "PII sprawl" before client code exists.

- Privacy wall by architecture, not policy copy. The plan says "per-bird events never leave the simulation service for any aggregate/ML purpose," only "operational aggregate telemetry" is collected, and the simulation database is physically/network-isolated from analytics. Visit mode also records "no presence" from visitors.

- Accessibility is a first-class product surface. Accessibility appears in the v1 scope, in a dedicated section that says it must "ship concurrent with core," and in ship criteria requiring blind tester review and reduced-motion review by vestibular-sensitive users. Reduced motion must be "calmer, not broken," not an "animations off" fallback.

- Procedural naturalism over canned assets. The plan couples procedural call grammars, naturalist notebook prose, call captions from grammar metadata, and "no recorded audio loops." It names "dead-loop artifacts," bundle size, and "uncanny-valley" prevention as reasons for procedural audio.

- Product voice separation. Naturalist prose belongs in the notebook, narration, and call captions; system failure paths must use "matter-of-fact voice." The plan explicitly says "No naturalist prose on system failure paths."

- Scope discipline against social and game creep. The plan lists explicit refusals, includes "no-discover surface review gate," and calls out failure modes such as "Just a little comment box" or discovery feed. The social surface is constrained to invite/revoke/read-only visit flows.

## Per-feature whys

### Executive Summary and Scope

- Browser-only web product: NOT RECOVERABLE FROM PLAN

- Single-aviary companion surface: The plan uses "single-aviary" and rejects profiles, discovery, leaderboards, comments, public aviaries, and social network surfaces so the product remains a calm companion rather than a network.

- Two starter birds at account creation: NOT RECOVERABLE FROM PLAN

- Cap at seven total birds: NOT RECOVERABLE FROM PLAN

- Birds added solely by aviary age: The plan ties adoption to "the birds that arrived" and explicitly rejects "catalog selection of birds," preserving non-catalog, non-shopping discovery.

- Personality 5-tuple: The five scalar traits are the substrate for weeks-long expressive drift, mood bias, call behavior, and visual character while keeping numbers hidden from users.

- Server-only persisted personality: The plan says personality is "persisted only on server" and "No client ever mutates personality" so clients cannot diverge or last-write-win the aviary state.

- Mood enum and transition matrix: Mood gives fast-timescale state on top of slow personality, with transitions modulated by personality, time of day, recent events, and ambient drivers.

- Procedural call grammars per species: The plan uses grammar and WebAudio so calls can vary by species, mood, phase, and personality without stored audio loops.

- Server tick around one minute: The tick applies drift, transitions moods, writes snapshots, and keeps the simulation moving "even with zero connected clients."

- Strict presence definition: Presence is the main input to drift, so the plan defines it as visibility plus focus plus recent pointer/key activity and adds a harness for the "three-conjunct logic."

- Field notebook: The notebook exists to produce "sparse, naturalist prose" from state changes and worth-observing moments, not per-session summaries.

- Settle: The plan describes settle as opt-in and as a "global warm-dim + calls -> ambient wind-down," so its articulated why is a calm wind-down surface.

- Listen-in: Listen-in is "mix rebalance, not mute"; it focuses one bird while every other bird remains audible and the visual cue is "purely visual affordance, no state change."

- Offers: Offers are selected interactions, but the plan keeps them marginal: "presence dominant, listen-in secondary, offer marginal," with offer acceptance also able to influence mood.

- Per-bird cooldowns on offers: NOT RECOVERABLE FROM PLAN

- Magic-link email auth: The plan keeps auth thin and email-based, with session issuance/revocation and email change verification separated from aviary state.

- Synthetic account UUID: The architecture rationale says synthetic IDs help prevent "PII sprawl"; account identity uses synthetic UUID primary and encrypted email secondary.

- Encrypted email at rest: The why is privacy minimization around account identity, reinforced by the separate auth store and synthetic UUID foreign key.

- Per-device revocable tokens: Revocable tokens support session control and the plan's explicit revoke-session endpoint.

- Multi-device pure renderer model: Multi-device works because clients append events and pull snapshots while the server owns personality and mood, avoiding divergent vectors.

- One-time visit invitations: The visit feature is the only social affordance and is constrained to one-time email links, revocation, expiry, and read-only ambient access.

- Visit notifications opt-in and off by default: The plan keeps visit awareness quiet and avoids default notification pressure.

- No visitor presence recording: The plan states visitors emit no events and record no presence, preserving the host aviary's honest-presence signal and privacy wall.

- Running screen-reader narration: Narration is part of the first-class accessibility surface and must provide naturalist prose at a slow cadence.

- Designed reduced-motion mode: Reduced motion must be a separate cross-fade surface, not an "animations off" toggle, and must be reviewed as "calmer, not broken."

- Call captions: Captions come from the same grammar as audio calls so the accessible surface matches what the procedural audio produced.

- Full keyboard navigation and WCAG AA: The plan names keyboard nav, roving tabindex, focus rings, contrast tokens, and no hue-only information as accessibility requirements.

- Performance budgets: The budgets are hard gates because the main path must reach "time-to-first-bird" quickly, hold 60 fps idle, and avoid memory growth.

- Privacy wall telemetry: The plan collects only operational aggregates and deliberately avoids per-bird histograms, exact individual session duration, and click counts.

### High-Level Architecture

- Frontend SPA as pure rendering, interaction capture, and WebAudio context: The frontend has "Zero ownership of persistent state" so it cannot become authoritative for personality or mood.

- Code-splitting settings, notebook, and visit flows: The rationale is the hard bundle and first-bird performance budget; the critical path stays small.

- Thin Auth Service: Auth owns identity tables only and is "Never authoritative for aviary state," limiting both responsibility and PII spread.

- Simulation Service as core owner: It is the canonical owner of aviary records, personality, moods, and notebook, matching the product's server-authoritative state principle.

- Tick running for active aviaries with zero connected clients: The aviary remains a continuing companion rather than a client-local animation that stops when closed.

- Asset/CDN layer for small static SVGs, palettes, and motif data: The plan keeps assets small while procedural generators ship as code.

- Transactional-only Email Delivery: The email surface is constrained to account and invite flows, with "no marketing."

- Single database logical boundary with separate auth store: The simulation primary store is the "source of truth," while auth has only the foreign synthetic UUID.

- Render/simulation boundary snapshots: The snapshot includes enough positions, velocities, moods, call phases, and timing for 10-60 seconds of interpolation without client-side stochastic bird behavior.

- Client-only ornament leaves and feathers: Ornament can be non-authoritative because it does not affect bird behavior, personality, mood, or sync.

### Data Model

- Account settings: Settings carry visit notification default false, reduced-motion override, and captions enabled so privacy and accessibility choices are persistent.

- Bird stable internal UUID: The bird id "never changes even on rename/species migration," preserving continuity.

- Bird species enum: NOT RECOVERABLE FROM PLAN

- Personality last_drift_applied_tick_id: The plan says this makes deltas "replayable for repair."

- Mood last transition timestamp: The timestamp supports decay timers and continuity in the mood state machine.

- PresenceEvent source audit: Presence stores the "visibility+focus+activity conjunction record" for audit of the strict presence rule.

- InteractionEvent append-only log sharded by account: The tick consumes ordered events later, supporting replay and preventing direct client mutation.

- NotebookEntry created with sparse policy: Sparse creation prevents notebook prose from becoming a per-session feed and keeps it tied to observations.

- VisitInvite token, expiry, revoke, use fields: The fields implement one-time, revocable, 30-day visit links.

- Hashed visit token: NOT RECOVERABLE FROM PLAN

- VisitLogEntry host-only append: The plan constrains visit logging to host-side records rather than a public social surface.

### API Surface

- Snapshot Pull tiny JSON: The response is intentionally "tiny" and supports fast first render with current bird state, mood, call phase, and notebook head.

- ETag, last_seen_ts, and short edge cache: These exist to skip unchanged snapshots and cache for only 30-60 seconds per account without giving up freshness.

- Personality fingerprint hash: It supports drift change detection while exposing "never numbers."

- Event Append endpoint: Events are batched and appended with idempotency; the tick consumes them later, preserving canonical server mutation.

- Presence pings as lightweight heartbeats: They exist inside the window rule so presence can be tracked without treating every client action as authoritative state.

- Magic-link auth endpoints: These implement the thin auth model for request, consume, revoke, and email change flows.

- Double opt-in email change: NOT RECOVERABLE FROM PLAN

- Notebook cursor pagination: NOT RECOVERABLE FROM PLAN

- Visitor read-only projection: The visitor endpoint returns a snapshot-shaped projection flagged read_only and visitor_mode, and "emits no events."

- Settings/account endpoints: Export, delete, and settings patching belong in matter-of-fact account control rather than naturalist prose.

- Error surfaces: The plan requires matter-of-fact voice and no naturalist prose on failures so system problems do not pretend to be aviary observations.

### Simulation Engine Design

- Pure, testable tick implementation: The tick is outlined as a deterministic sequence so drift, mood, notebook, and snapshot writes can be tested and replayed.

- Drift deltas applied before mood transitions: The plan orders presence-driven personality changes first, then mood transitions using updated personality.

- Additive expressive-only drift: Every component goes up or stays flat, matching "monotonic expressive" drift and avoiding negative Tamagotchi-style consequences.

- Presence dominant, listen-in secondary, offer marginal: This hierarchy makes "honest presence" the main driver rather than rewarding clicks.

- Slow week-one and week-three drift targets: The plan wants week one measurable in instrumentation and week three visible to users without exposing numbers.

- Sparse notebook generation from worth_observing: Notebook entries appear when events, personality deltas, or time-of-day shifts are worth observing.

- Calibration harness before real-user ship: Synthetic accounts and scripted presence patterns are required so drift targets are known before real accounts.

- Call grammar deterministic function: Determinism from seed, phase, mood, and personality lets server and client share grammar outputs and keeps captions aligned.

- Mood input order: Personality bias comes before time-of-day, recent interaction history, and rare weather so mood reflects bird character first.

- No-snapping mood transitions: Drowsy should drift into evening settled rather than "reset randomly," preserving continuity.

### Sync Model and Conflict Prevention

- One canonical writer plus append-only log: The plan says this means "Never reconcile divergent vectors."

- Arbitrarily stale clients pulling fresh snapshot on visibility restoration: Suspended laptops are safe because the client does not own state.

- Magic-link replay conflict surface: The matter-of-fact "link expired" path keeps auth failure clear and non-naturalist.

- Session timeout mid-write conflict surface: The response is re-auth, keeping write authority tied to current sessions.

- Outage during tick window: The client poll model and eventual-consistency message keep users oriented while preserving the server tick model.

- Export and hard-delete coordination: These are the only account-level operations spanning auth and simulation because they touch both identity and aviary state.

### Frontend Rendering Pipeline

- Scene layer z-order: NOT RECOVERABLE FROM PLAN

- Idle micro-motion driven by mood and personality: The birds' visual behavior reflects mood and personality through blend weights such as wary scanning, content preening, and curious head tilts.

- Never rely on CSS for personality: The plan requires authored idle loops or spline curves so personality is not an accidental styling effect.

- Pause rendering when document.hidden: This protects performance while "simulation marches on" server-side.

- Initial fly-in path: NOT RECOVERABLE FROM PLAN

- Perch change as smooth arc or hop, personality-modulated: The motion itself carries personality rather than being a generic transition.

- Listen-in visual cue: The gentle scale or lighting lift identifies the focused bird while remaining "purely visual" with no state change.

- Settle warm-dim transition: The transition turns calls into ambient wind-down for the settle surface.

- Undo settle fast reverse: NOT RECOVERABLE FROM PLAN

- Reduced-motion separate render paths: The plan rejects toggled filters and requires a full reduced surface with canonical posture cross-fades.

- Leaf ornaments removed in reduced motion: This supports the vestibular-safe reduced surface while keeping static foliage.

- First-frame contract: Snapshot data must render frame N immediately, with "No spinner ever" and only a quiet-field placeholder on very slow pull.

### Audio Pipeline

- WebAudio graph as mandatory primary path: The graph gives procedural per-bird sound with small footprint and controllable mixing.

- Light stereo pan by perch zone: The plan allows spatial placement but says it must be "never distracting."

- Listen-in gain ramps and sidechain ducking: The rationale is focus through mix balance, never muting any bird to 0.0.

- Phase-aware chorus blending: Procedural variation and phase awareness eliminate "dead-loop artifacts" from layered tracks.

- Client call scheduling between snapshots: Server seeds and offsets allow local timing extrapolation while the authoritative snapshot remains server-provided.

- Silence plus forced captions when WebAudio is unavailable: This keeps the accessible surface available without shipping fallback recorded voices.

- No audio file assets shipped for voices: The plan calls this a "hard bundle + uncanny-valley preventative."

### Accessibility Surfaces

- Polite live-region screen-reader narration: The plan provides running narration without interruptive urgency.

- Narration cadence of 40-60 seconds with action bump: The narration stays slow in idle but becomes more responsive to user actions.

- Narration text surviving visual and pure-narration contexts: The same voice must work without rephrasing when visual context is absent.

- Call captions from audio metadata: Captions match the same seed and grammar as the sound, keeping captions and calls synchronized.

- Caption fade timing matching audio duration: The visual text follows the call it represents.

- Roving tabindex and key bindings: Birds and top-bar items remain keyboard reachable, with Enter for listen-in and Escape to disengage.

- High-contrast soft halo focus ring: Focus remains visible against any palette state.

- Contrast tokens and no hue-only information: Text and information surfaces meet WCAG AA and do not rely on color alone.

### Performance Budgets and Observability

- Hard performance gates blocking v1: The plan treats bundle size, first-bird time, 60 fps, memory, audio allocation, and tick latency as ship blockers.

- Synthetic browser fleet and RUM: These instruments track operational paint, frame, audio-error, and tick-consumption health across geographies.

- Aggregate-only telemetry: Observability is allowed only when it does not expose per-account personality.

- Deliberately unmeasured per-bird interaction histograms: The plan avoids data that could "reconstruct relationships."

- Deliberately unmeasured exact individual session duration and click counts: The plan says "presence is the metric," not click counting or exact individual duration.

- Explicit SLOs in runbooks: Operational health has written error budgets rather than informal monitoring.

### Rollout and Ramp

- Internal dogfood with synthetic "bird watchers": The plan starts with a small internal and synthetic group to exercise the system before alpha.

- Invite-only 50 alpha accounts: The alpha limits risk before expanding to closed beta.

- 500 closed beta: The plan uses staged growth before gradual public release.

- Gradual public ramp with kill-switch and rollback: New accounts increase in controlled steps with a rollback plan.

- Bird count age thresholds at 60, 180, and 270 days: NOT RECOVERABLE FROM PLAN

- Day-one notebook health check: The plan explicitly asks whether any notebook entry was written each day so the sparse notebook is still alive.

- Drift velocity synthetic alert: The alert protects calibration by detecting median movement beyond the target.

- Presence definition regression harness: The plan says failure is a regression "in every future account."

- Narrated experience review by accessibility specialist and blind tester: This validates the screen-reader surface before ship.

- Reduced-motion review by vestibular-sensitive users: This validates the reduced surface as "calmer, not broken."

- Zero gamification strings or counters in shipped bundle: This enforces the no-gamification boundary at ship.

- Visit flows exercised with no presence leak: The rationale is proving invite/revoke/visit while preserving visitor non-presence.

### Risks and Mitigations

- Drift calibration sprint and offline Markov model tuning: The mitigation exists because one week could otherwise produce too much visible shift or no perceptible change.

- Deterministic replay tests using captured real sequences: These protect against personality reset or last-write-wins during two-device handoff.

- Call repeat test and alpha A/B: These mitigate "audio uncanny / canned feel" and repeated motifs in early sessions.

- Accessibility owner embedded in every vertical slice: This reduces the risk that narration or reduced-motion regress late.

- Early performance sprints with real Moto G device: This mitigates bundle bloat and time-to-first-bird misses on target hardware.

- DLP, query log scanning, and analytics isolation: These mitigate accidental per-bird event rows entering the analytics warehouse.

- Non-goal review and no-discover surface gate: These mitigate social feature scope creep such as comments or discovery.

### Work Breakdown, Documentation, and Open Decisions

- Sprint 0 foundation and calibration: The first sprint builds data model, tick skeleton, replay harness, presence verifier, drift math, and auth because calibration and authority are prerequisites.

- Sprint 1 core alive surface: The plan gets two birds, one call grammar, snapshot/event pipeline, ambient, and first notebook generator working as an alive product core.

- Sprint 2 visual core, listen-in, offers, and settle: This adds the main interaction and visual behavior once core state exists.

- Sprint 3 accounts, sync polish, and multi-device dogfood: This verifies session revokes, conflict surfaces, and cross-device drift continuity.

- Sprint 4 audio production and polish: Full grammar, WebAudio, transitions, and silence/caption fallback come after the core has one species path.

- Sprint 5 accessibility complete and reduced-motion: The plan finishes narration, reduced motion, keyboard, and captions before social and beta hardening.

- Sprint 6 social minimal and hard privacy gates: Visit work is delayed until the core privacy model can enforce read-only projection and no presence leak.

- Sprint 7-8 performance, calibration validation, and beta: Bundle/runtime gates, 30 synthetic weeks, and alpha-to-beta ramp validate the fragile product constraints.

- Sprint 9 hardening and ship: Final sign-offs, rollback, monitoring, and runbooks close the release.

- Module "why this protects the product contract" notes: The plan says every module must carry this note so teams do not need to re-ask "why."

- ADRs for presence, monotonic drift, procedural audio, reduced motion, and privacy wall: ADRs preserve the locked architectural and affective constraints.

- Tick p99 runbook: The runbook gives operators a path when tick latency exceeds the 3 second alarm.

- Calibration notebook: It is a living doc that "survives the team that produced it," preserving drift calibration knowledge.

- Open implementation decisions: The plan leaves only narrow implementation choices open and says "all major affective and architectural constraints" are already locked.
