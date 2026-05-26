## System-level intent

- Alive over weeks, not robotic. The plan opens by anchoring every decision in "feels alive, not robotic" and repeats this through "procedural everything where aliveness matters," "feels alive over weeks," "no animation start from rest," and first-paint timing fast enough that motion feels "always-already-running."

- Notice never announce. The plan carries "notice never announce" into sparse notebook entries, "no spinner," "no hover pop-outs," "no inline labels in scene," top-bar fade, and field-note/narration cadence that avoids flooding.

- Charm from specificity. The plan insists on "charm from specificity" through bird names, "specific verbs," species motifs, mood-specific timbre, absence-length and boldness aware greetings, and the "snippet test" requiring bird-name specificity and no numbers.

- Restraint over richness. The plan makes restraint a product rule: "No gamification of any kind," "No Tamagotchi mechanics," "no hidden 'enable streaks' toggle," no social-network surfaces, no panning/scroll/zoom, no catalog pick, no marketing push, and feature toggles only for infrastructure.

- Naturalist voice for product, matter-of-fact for system. The plan separates product prose from system prose: notebook, narration, and captions use "naturalist register," while auth, errors, account ops, support, and revoked visits use "matter-of-fact voice" with "zero naturalist vocabulary."

- Server-owned canon. The plan repeatedly states that the "server alone owns time, drift, mood, and persistence," that the "server is the sole author of personality state and canonical aviary," and that there is "no LWW ever." Clients render snapshots and append events only.

- Accessibility and performance are first-class surfaces, not fallbacks. The plan says the product "must feel fully alive in reduced-motion, screen-reader, and audio-off paths." It pairs this with hard budgets, automated and manual accessibility testing, caption fallback, reduced-motion cross-fades, and first-bird paint targets.

- Privacy boundary and user ownership. The plan limits telemetry to "aggregate-only," forbids per-bird/per-account product analytics, keeps personality values out of client APIs except export, and preserves account export, revocation, and deletion paths.

## Per-feature whys

### Document Conventions & Defensible Decisions

- Presence activity window of 180s: The plan gives the rationale directly: "longer than quick glance, shorter than 'left laptop open'."

- Tick cadence of 60s nominal: NOT RECOVERABLE FROM PLAN

- Bird cap of 7: NOT RECOVERABLE FROM PLAN

- Starting with exactly 2 birds and adding birds by aviary age: The plan explains that new birds are "gated purely by aviary age" and "never visit- or interaction-count gated," preserving the anti-gamification and anti-Tamagotchi boundary.

- No feature flags that reintroduce forbidden surfaces: The plan frames this as preventing hidden streaks or similar forbidden mechanics from returning through toggles.

- Ambiguity rule: The plan says ambiguous choices should preserve "feels alive over weeks," minimize "announcement surfaces," and keep the bundle under the gzipped ceiling.

### Scope

- Single-user accounts via email magic-link: NOT RECOVERABLE FROM PLAN

- Server-side simulation tick: The plan says it advances personality drift, mood, notebook, and time-of-day "regardless of clients," supporting server-owned life continuing even when no client is open.

- Multi-device sync via canonical server state: The plan rejects "last-write-wins on vectors" so devices receive the same canonical state rather than reconciling personality locally.

- Return-greeting: The plan makes it procedural and "absence-length + boldness aware" so greeting behavior reflects specificity instead of canned entry behavior.

- Listen-in: The plan describes "mix rebalance with slow ramps," matching the intent to focus on one bird while keeping ambient life present.

- Offer types and cooldowns: NOT RECOVERABLE FROM PLAN

- Settle: The plan calls it an "opt-in soft end-gesture" with "5s undo," giving the rationale as a deliberate, reversible end action rather than a decay or punishment mechanic.

- Field notebook: The plan calls for "sparse auto naturalist prose" and read-only presentation so observations notice changes without turning them into scores, numbers, or user-authored tasks.

- Presence accounting: The plan requires a "strict 3-signal conjunction" and later says pointer/keyboard alone does not trigger greeting; the reason is to credit real presence, not idle laptops or accidental input.

- Aviary layout: The plan uses a "single horizontal responsive scene," no panning/scroll/zoom, no inner chrome, and always keeping birds in frame to preserve the aviary as one ambient scene rather than a navigated interface.

- Visit invitations: The plan makes them "opt-in, per-invite, revocable, read-only ambient, defaults OFF" and bars discovery/leaderboards/social surfaces, keeping visits private and non-social-networked.

- Accessibility surfaces: The plan says accessibility is a "first-class surface" and that reduced-motion, screen-reader, and audio-off paths must still feel fully alive.

- Account settings: The plan groups re-linking, session revocation, export, soft-delete, privacy policy, accessibility prefs, visit log, and revocation under matter-of-fact account surfaces, supporting user ownership and system clarity.

- Privacy boundary: The plan says per-account interaction events are used "only for that user's simulation/tick" and telemetry is "aggregate ops" with "no per-bird in analytics."

- Graduated rollout: The plan uses birds-per-aviary over time, age thresholds, and no marketing push to preserve the same restraint as the product experience.

### Architecture

- Thin API + thick server-owned simulation + rich browser client: The plan says clients render snapshots and emit events while the server owns time, drift, mood, and persistence.

- Lightweight SPA at the edge: The plan sets a tiny bootstrap and lazy-loaded secondary surfaces so the first load supports fast aliveness and keeps account/settings flows out of the critical path.

- Core API service: The plan says it handles auth, snapshots, events, and visit flows but "never mutates personality vectors directly," preserving the simulation boundary.

- Simulation Engine Service: The plan says it owns the tick and uses leader election because the sim-worker requires "no dual writer" discipline.

- PostgreSQL persistence: The plan calls it the primary store, mentions future row-level security patterns, and says "no ORM surprises" to keep schema and state behavior explicit.

- Magic-link email instead of SSO/passkeys/passwords: NOT RECOVERABLE FROM PLAN

- Aggregate-only telemetry and synthetic probes: The plan uses operational metrics and browser probes while keeping analytics out of per-account/per-bird data.

- Blue/green or canary deploys with sim-worker discipline: The plan ties this to avoiding dual writers during deployment.

- Client/server split boundary: The plan keeps personality vectors, mood, notebook, and event logs on the server while the client owns only ephemeral render/audio/focus state.

- Snapshot payload without raw personality numbers: The plan says snapshots carry pose, mood, seeds, and timing data, not raw personality numbers, preserving the rule that users never see numeric personality.

- Canvas 2D render pipeline: The plan says Canvas wins for "60fps five-year-old laptop guarantee" and "aliveness micro-control."

- Parallel accessibility narration and captions from the same snapshot interpreter: The plan says this prevents divergence between visual/audio and accessibility output paths.

- Hidden-tab behavior: The plan halts rAF and suspends WebAudio for battery while the server tick continues, then refreshes on visibility/focus restoration.

### Data Model

- Synthetic account id and encrypted email: The plan comments that the id is "never email," supporting the privacy boundary.

- Personality stored in personality_json and hidden from client APIs: The plan says it is never exposed except protected export, aligning with "Any numeric personality exposure to users (never)."

- personality_drift_history: The plan describes it as "append-only audit for debug; not for computation," supporting replay/debug without becoming live state.

- interaction_events append-only feed: The plan says this feed is consumed by the tick, preserving append-only client input and server-owned mutation.

- notebook_entries prose field: The plan specifies "naturalist lowercase present-tense," tying storage directly to the product voice.

- visit_invitations with token_hash and revocation fields: The plan supports per-invite, revocable visit links without public discovery.

- Event log retention policy of 13 months: NOT RECOVERABLE FROM PLAN

- UTC timestamps and refusing client clock for simulation: The plan says all timestamps are UTC and "Never trust client clock for simulation logic," protecting canonical server time.

### API Surface

- GET /aviary/snapshot with ETag: The plan calls it idempotent and cheap, and uses 304 responses when there is no change.

- POST /aviary/events: The plan accepts events immediately as append-only input and processes them on the next tick, avoiding direct client mutation.

- Presence pings with proof of three signals: The plan requires visibility, focus, and activity proof before the server credits presence.

- GET /account/export: The plan describes a user-owned full dump delivered through a signed URL, preserving export without exposing vectors through ordinary APIs.

- Session list and revocation: The plan includes "revocable per-device sessions," giving users direct control over active sessions.

- Visitor-side visit mode: The plan limits visits to a read-only snapshot, captions default-on, no controls, disabled settle, and presence never credited, keeping visits ambient and non-mutating.

- Revoked visit response: The plan returns a 410-like matter-of-fact message, preserving revocability and system voice.

- Error contract: The plan requires matter-of-fact voice and request-id, and explicitly bars naturalist phrasing on errors or login screens.

- Rate limits and abuse detection: The plan rate-limits event velocity while avoiding hard per-bird write quotas besides engine cooldowns, keeping abuse control out of user-visible counters.

### Simulation Engine Design

- Tick lifecycle with serializable transaction: The plan loads ordered events, computes presence, applies drift/mood/notebook changes, and saves atomically so the tick remains the single writer.

- Monotonic drift: The plan says monotonic increase only, matching "no decay, no distress" and preventing Tamagotchi-style loss.

- Drift input weights: The plan maps presence to expressive weight, listen-in to social_warmth and vocal_frequency, offers to boldness, and accepted offers to curiosity, making interactions affect traits through plan-defined meaning.

- Slow time constant: The plan says one effective hour changes traits by about +0.008 and "No visible change <1 week regular use," supporting change that is felt over weeks.

- Mood transitions with hysteresis and probabilistic edges: The plan uses timers and a table-driven matrix to make moods tunable and less mechanical than hardcoded ifs.

- Local time-of-day mood bias: The plan uses local 22:00 and 06:00 biases so drowsy and alert states track the user's day/night context.

- Call grammar runtime: The plan uses motif templates, personality perturbation, mood modulation, and jitter, supporting procedural calls instead of canned audio loops.

- Bird-to-bird chorus overlap: The plan uses nearby high vocal_frequency calls within an 800ms window to create a "natural chorus overlap without scheduling."

- Notebook cooldowns and sparse triggers: The plan targets max 1 entry per 36h for active users, aligning with notice-not-announce and avoiding a feed-like surface.

- Adoption and age gating: The plan starts with two low-permutation birds, no catalog pick, neutral-average personality, and age-based additions so adoption is not choice-optimization or visit-count gated.

### Sync Model

- Canonical identical snapshots: The plan says every authenticated client receives identical snapshot contents at the same wall time modulo transport, preserving one source of truth.

- Conflict prevention without LWW: The plan uses tick-only mutation, idempotent event correlation keys, server ordering, and session expiry handling because personalities are never auto-merged.

- Offline/suspend/resume buffering: The plan buffers a limited number of events and drops stale ones over 10 minutes with a non-error notice only if critical, preventing stale activity from distorting state.

### Frontend Rendering Pipeline

- Boot path without spinner: The plan uses critical CSS, fast state fetch, and a "quiet field" background on slow fetch so the aviary feels alive before data arrives.

- Initial render from current idle action: The plan computes current local phase and interpolated positions with "no animation start from rest," preserving always-already-running motion.

- Shared bird silhouettes and plumage tint: The plan avoids "per-bird raster bloat" while still letting personality affect visual tint.

- Perch zones by y-band and z-sort: NOT RECOVERABLE FROM PLAN

- Idle micro-motion engine: The plan uses preen, scan, weight shift, and head-tilt behaviors driven by mood, personality speed, and seeded randomness to create procedural aliveness.

- Ambient ornament object pools and weather wind: The plan uses object pool reuse to avoid hot-loop allocation, with wind affecting trajectories for weather specificity.

- Lighting palette shifts: The plan ties palettes to time-of-day phase and weather, making the scene respond to local time and ambient conditions.

- Listen-in, settle, and fly transitions plus reduced-motion substitution: The plan uses tuned ease curves and reduced-motion cross-fades so motion has polish without excluding reduced-motion users.

- No hover pop-outs or inline labels: The plan keeps the scene from becoming an explanatory interface, matching "notice never announce."

- Pointer/keyboard alone does not trigger greeting: The plan says greeting must satisfy presence, preventing accidental input from becoming a social signal.

- Drag forbidden on birds: The plan says the gesture is "reserved for future deliberate non-use," preserving restraint around manipulation.

- Always preserving all 7 birds in frame: The plan tests 320-1440px and auto-spaces/squashes on narrow screens so the aviary remains whole without panning or zoom.

### Audio Pipeline

- Web Audio as sole synthesis path: The plan bans MP3/WAV fetches and recorded fallback, matching procedural calls and "no canned audio loops."

- Per-bird oscillator voice with shared richness: The plan uses oscillators, envelopes, filters, and shared reverb for richer procedural sound without audio assets.

- Chorus mixer and listen-in ramps: The plan raises the target and lowers others but "Never zero for ambient," keeping the focused bird prominent without making the aviary silent.

- Client-local call scheduler: The plan keeps real firing client-local "to stay live during brief disconnects."

- Mood-to-timbre mapping: The plan maps alert to brighter harmonics and drowsy to slower attack/detune, making mood audible.

- WebAudio failure fallback: The plan forces captions on and plays silence, treating audio-off as a valid accessibility path without recorded fallback.

- Global mute and per-bird listen-in for captions users: The plan lets caption users listen-in independently of global mute, preserving interaction meaning without audio.

### Accessibility Surfaces

- Screen-reader narration: The plan uses an ARIA live log and a pure function from snapshot/time/tz so narration matches the same naturalist register as the notebook.

- Narration queue limits: The plan avoids flooding by allowing max one queued item and a slow cadence.

- Captions: The plan places captions near the calling bird or at fixed bottom and emits text from the same audio synthesis source so captions match actual calls.

- Reduced-motion mode: The plan replaces procedural idle animations with pose cross-fades while keeping calls, captions, narration, drift, and day-cycle color unchanged, making it "distinct" rather than stripped.

- Keyboard navigation: The plan includes roving tabIndex, arrow navigation, offer/notebook shortcuts, listen-in toggle, escape behavior, and visible focus, making all interaction paths keyboard-accessible.

- Panels without focus trap: The plan says no trap-focus is required because the aviary must remain visible.

- Contrast and color tokens: The plan requires WCAG AA, AAA for body text, and immutable tokens to keep contrast reliable across palettes.

- Accessibility testing: The plan requires axe-core and manual VoiceOver/JAWS/NVDA, keyboard-only, and reduced-motion coverage before beta.

### Performance Budgets and Observability

- Hard client budgets: The plan ties first-bird paint to motion feeling "always-already-running," and enforces JS, frame, and memory budgets in CI and RUM.

- Server budgets: The plan sets tick, snapshot, and cold-start targets so simulation and first paint stay fast enough for live-feeling state.

- Day-0 aggregate instrumentation: The plan measures timing, frame-time, audio errors, presence fidelity, and synthetic dwell scans while keeping identifiers out.

- Deliberately uninstrumented data: The plan forbids per-bird/product analytics, notebook content logs, personality values outside export/debug, and session replays, preserving the privacy boundary.

- Bundle strategy: The plan code-splits secondary flows, avoids heavy date libs, inlines motif/vector data, and uses system or small local fonts to stay within the bundle ceiling.

### Rollout

- Internal launch with manual bird ramp: The plan uses founder/test accounts and manual 2-to-7 ramping for drift calibration before broader exposure.

- Limited beta and GA age thresholds: The plan keeps beta to 2 birds and GA additions by age threshold with "no marketing push," preserving restraint.

- Feature toggles only for infrastructure: The plan allows maintenance toggles but never gamification or social surfaces, protecting non-goals during rollout.

- Instrumentation from hour zero: The plan wires build-time assertions and runtime synthetic alerts so budgets are enforced immediately.

- Support surfaces: The plan uses mailto, a plain KB, export, and 30-day recoverable delete in matter-of-fact voice, keeping support out of naturalist framing.

- Recoverable delete: The plan gives users a 30-day restore path before hard-delete.

### Risks & Mitigations

- Drift calibration harness: The plan calls this "most existential" because too-fast change feels like Tamagotchi and too-slow change feels static; mitigation is shadow accounts, coefficients, killswitch, and delaying GA if needed.

- Sync/personality corruption mitigation: The plan uses serializable transactions, advisory lock, idempotency, and reconciliation audits to avoid two writers, lost events, and double-crediting.

- Audio prototype feedback: The plan treats uncanny or samey sound as an "aliveness killer" and mitigates it with early non-team listener loops and capped motifs.

- Accessibility regression mitigation: The plan calls out narration lag, less charming reduced-motion, and invisible focus as risks, then assigns sprint reviews, linters, and a reduced-motion pilot cohort.

- Performance creep mitigation: The plan uses size limits, perf PR blockers, and an owner pair so later surfaces do not gate motion-aliveness.

- Privacy leakage mitigation: The plan separates DB users and adds code review labeling for data access so telemetry pipes do not cross the privacy boundary.

- Email deliverability mitigation: The plan says spam/friction kills "calm tone," so it calls for reputable delivery, warm-up, SPF/DKIM/DMARC, and plain-text matter-of-fact emails.

- Tick latency mitigation: The plan uses p99 alarms, auto-pause, batching older accounts first, and indexed pure CPU queries to avoid load or contention breaking the simulation.

### Additional Execution Notes

- Snippet test: The plan uses blind reads of five random outputs to verify lowercase, present-tense, no "you," no achievement language, specificity, and no numbers.

- Zero side-effect without presence: The plan says every interaction must have zero side-effect when the presence definition is not satisfied, protecting drift from accidental or uncredited activity.

- Matter-of-fact surface audit: The plan requires every matter-of-fact surface to be audited for "zero naturalist vocabulary."

- Feature addition PRs citing non-goals: The plan requires later feature PRs to cite which non-goal they would violate, making restraint a review gate.

- Three thin vertical slices: The plan starts implementation with snapshot render, tick round-trip, and WebAudio motif/listen-in, proving aliveness, server mutation, and audio before layering behavior.

- Fourteen-day qualitative success criterion: The plan wants a new user to report "my birds feel different now - Pip more forward," showing that change is perceptible as specificity, not as metrics.

- Zero streak/gamification visual review: The plan makes absence of gamification surfaces a GA success criterion.

- Screen-reader cohort criterion: The plan requires screen-reader users to describe bird mood from narration alone, proving the accessibility path is fully expressive.

- Deletion hard-delete criterion: The plan requires all per-account data tables to clear with no dangling events or vectors, proving account deletion respects the privacy boundary.
