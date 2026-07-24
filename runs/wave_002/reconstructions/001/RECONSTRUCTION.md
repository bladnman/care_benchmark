## System-level intent

- Server-authored life, client-rendered presence. This shows up in the service shape ("Web client ... Owns no canonical state"), the client/server split ("Server owns" personality vectors, moods, canonical positions, drift), the simulation tick as "the only writer of personality state," and the sync model ("Single canonical state per account, server-only writes").

- Privacy boundary by architecture, not just policy. The plan repeats that personality vectors are hidden, the client "never derives behavior from raw trait values," render hints are coarsened to keep vectors "invisible on the wire," the simulation DB is "never read by analytics pipelines," and RUM has "No per-bird state, no per-account dimensions."

- Ambient, non-gamified, non-social product shape. The scope explicitly excludes "gamification of any kind," "Tamagotchi mechanics," "social-network surfaces," and "notifications pushed at the user." The risks section says scope creep into gamification/social is rejected by reference to non-goals, and observability avoids "anything reconstructing a user's relationship with their birds."

- Slow, monotonic expressiveness instead of obligation or decay. The bird engine includes "monotonic-toward-expressive drift"; the tick applies signals only upward while "absence applies zero (never negative) delta." Drift calibration is framed between "too fast -> Tamagotchi" and "too slow -> screensaver," with targets where instruments see change at about one week and users feel it at about three weeks.

- Procedural variation with recognizable individuals. The plan emphasizes "procedural calls," motif parameters, seeded RNG, and a fixed seed-derived timbre offset so each bird "stays recognizable across mood/drift." It also warns that repetitive-sounding procedural calls "break the spell as badly as loops."

- Naturalist, observation-voiced prose. The notebook entries "observe the aviary, never the user's behavior," captions use "naturalist voice," narration is "naturalist prose," and accessibility risk calls out narration collapsing into "state-list phrasing." The plan bans "you visited" phrasing "ever."

- Accessibility is part of v1, not a fallback. Scope says accessibility surfaces ship with v1; reduced motion is "a designed surface, not a stripped fallback"; and the accessibility ship gate says these surfaces are "launch-blocking, not v1.1."

- Immediate, quiet, continuously framed experience. The frontend requires "no spinner, no entry animation," a "quiet field" slow-connection state, "first bird visible <500ms," all birds "always in frame," a render loop that halts when hidden, and CI-enforced performance and memory budgets.

## Per-feature whys

### 1. Scope

- Web-only aviary: NOT RECOVERABLE FROM PLAN

- Two starter birds, cap of seven, and age-only unlocks: The rollout says the "ramp is time, not feature flags," and the scope excludes streaks, counters, badges, leaderboards, and other gamification, so the plan's articulated rationale is that bird growth comes from aviary age rather than game mechanics.

- Full bird engine: The plan ties hidden personality vectors, mood, procedural calls, and drift into a "full bird engine" so birds can change through server-authored moods, calls, perch positions, and notebook observations rather than static rendering.

- Hidden personality vectors: The rationale is the "never expose numbers" rule and privacy boundary. The render pipeline says the server pre-resolves behavior into snapshot fields and keeps personality vectors "invisible on the wire."

- Mood system: The tick uses a state machine over "wary, content, curious, drowsy, alert, settled" with recent interactions, time of day, weather, and personality modulation so mood persists across sessions and advances during absence.

- Procedural calls: Calls are generated from motif parameters, seeded jitter, and personality shaping so execution is varied and not a canned clip; the audio risk says repetition would "break the spell."

- Monotonic-toward-expressive drift: Drift is designed so signals only push up and absence applies "zero (never negative) delta," avoiding Tamagotchi-style decay while still letting instruments and users perceive change over time.

- Presence accounting with 3-signal conjunction: The risk section says any shortcut such as "tab open" would corrupt drift population-wide, so presence must be enforced in one tested client module with server-side sanity checks.

- Return-greeting: The plan uses absence buckets, boldness, and mood availability so the greeting intensity can vary from "glance" to "two-note call" to "approach + longer call," with parameters shipped in the snapshot for procedural execution.

- Listen-in: The audio pipeline says the focused bird ramps up while others ramp down to an ambient floor, "never silence," because it should feel "like listening, not soloing."

- Offer interaction: The tick gives offers drift meaning: accepted offers push curiosity, and offered-near signals push boldness. The offer feature is therefore part of the personality drift input surface.

- Per-bird offer cooldown: NOT RECOVERABLE FROM PLAN

- Settle interaction: The tick says settle has no directional drift and "closes presence window cleanly"; the frontend transition turns settle into a slow evening shift.

- 5s settle undo: NOT RECOVERABLE FROM PLAN

- Field notebook: Notebook generation exists to produce sparse aviary observations from candidate events, with a "sparsity governor," dedup keys, phrase variation, and the rule that entries observe the aviary rather than the user's behavior.

- Single-user accounts: The rationale present in the plan is by exclusion: social-network surfaces such as profiles, follows, feeds, discovery, co-presence, chat, avatars, and comments are explicitly out.

- Email magic-link auth: NOT RECOVERABLE FROM PLAN

- Per-device revocable sessions: The sync model says sessions can be revoked, tokens expired, and errors handled with matter-of-fact re-auth copy; the account API also exposes session listing and revocation.

- Email change with verification: NOT RECOVERABLE FROM PLAN

- Server-side simulation tick as the only writer of personality state: This is the basis for multi-device sync, additive ordered deltas, and the rule that the client must never write personality.

- Visit invitations: Visits are "per-invite opt-in," "read-only ambient," revocable, expiring, and logged silently; visitor snapshots reject event ingest so visitors cannot affect canonical state.

- Off-by-default visit notifications: This matches the explicit non-goal of "no notifications pushed at the user."

- Accessibility surfaces shipping with v1: The accessibility section says narration, keyboard, contrast, reduced motion, and captions are "launch-blocking, not v1.1."

- Screen-reader naturalist narration: Narration uses aria-live with naturalist prose from snapshot state, idle pacing of about 30-60s, and priority bumps that remain "observation-voiced."

- Designed reduced-motion mode: Reduced motion replaces frame animation with cross-fade pose sequences, removes ambient drift, and slows color transitions because it is "a designed surface, not a stripped fallback."

- Procedural call captioning: Captions derive from the same motif and parameter set as synthesized calls, giving small naturalist descriptions such as "a soft three-note rise."

- Keyboard navigation: Keyboard access covers the top bar, scene bird focus, listen-in, disengage, offer flow, and settle, with visible high-contrast focus indicators.

- WCAG AA chrome contrast: The rationale is the accessibility ship gate and CI verification on settings, error, and caption surfaces.

- Account export: NOT RECOVERABLE FROM PLAN

- Soft-delete for 30 days then hard-delete: NOT RECOVERABLE FROM PLAN

- Mood enum choice, including settled: The plan gives one explicit rationale: "settled covers night/settle-gesture states."

- Personality traits normalized to [0.0, 1.0] with per-species prior and jitter: NOT RECOVERABLE FROM PLAN

- Presence activity window of 4 minutes: The plan's only articulated rationale is "leaning long per PRD guidance."

- Snapshot keepalive cadence: The cadence keeps clients refreshed while visible and on visibility changes or long render gaps; the sync model adds that clients interpolate between snapshots for "no teleporting."

### 2. Architecture

- Static SPA web client served from CDN: The plan ties the client to rendering, WebAudio synthesis, presence detection, and interaction capture while owning no canonical state; the frontend first-frame plan also depends on CDN-edge inlining for first-bird speed.

- Stateless API service: NOT RECOVERABLE FROM PLAN

- Simulation service as cron-driven worker or queue-partitioned workers: The plan says it is "the tick," sharded by account UUID "when scale demands," so the rationale is central server-side simulation with a scaling path.

- Single relational datastore: NOT RECOVERABLE FROM PLAN

- Append-only event log table: The tick reads unprocessed interaction events in order and marks them consumed; sync relies on ordered additive deltas and no last-write-wins.

- Canonical state checkpoints: The data model says this is an "optional optimization" so snapshot reads do not recompute.

- Simulation database blocked from analytics pipelines: The rationale is an infra-level "privacy boundary" with no warehouse connection and separate credentials.

- Server pre-resolves behavior into snapshot fields: This keeps raw trait values off the wire and prevents the client from deriving behavior from personality vectors.

- Client synthesizes calls and captions from delivered parameters: The rationale is consistency: caption text comes from the same call parameters the client synthesizes, while canonical behavior remains server-delivered.

- Client ambient ornaments as pure client state: The plan says leaves and feathers are "pure client, no state," keeping ornaments outside simulation state.

### 3. Data model

- Account synthetic UUID as the only identifier in logs, telemetry, and sharding: The rationale is to avoid using email or another direct identifier for logs, telemetry, and sharding.

- Encrypted account email: NOT RECOVERABLE FROM PLAN

- Account settings JSON for reduced motion, captions, visit notifications, and audio: Settings persist server-side, and accessibility settings honor media query by default.

- Session device label and revocation: The rationale is per-device session management and revocation through the account sessions API.

- Stable bird UUID that is never regenerated: NOT RECOVERABLE FROM PLAN

- Bird personality vector with server-only write access: This protects hidden personality state and supports the tick-only write path.

- InteractionEvent as append-only and never aggregated cross-account: Append-only events allow ordered tick consumption; "Never aggregated cross-account" supports the privacy boundary.

- NotebookEntry generation key: The plan says it is a dedup key so "the same observation isn't written twice."

- VisitInvite token hash, expiry, and revocation: The rationale is invite lifecycle control: 30-day expiry, revocation, and token-hash storage.

- VisitLogEntry with approximate duration: NOT RECOVERABLE FROM PLAN

### 4. API surface

- Magic-link issue endpoint with 15-minute expiry, single use, and per-email rate limit: The articulated rationale is bounded, rate-limited magic-link authentication.

- Magic-link consume endpoint: It issues a session token and invalidates the link, matching the single-use auth shape.

- Aviary snapshot endpoint: The snapshot is small and can be inlined into the initial HTML payload for "first-bird <500ms."

- Event ingest endpoint: It validates, appends, returns 202, and "Never accepts personality fields," preserving server-only personality writes.

- Notebook pagination: NOT RECOVERABLE FROM PLAN

- Bird rename endpoint, name only: The rationale present is narrow mutation: rename changes name only, not personality or behavior fields.

- Adoption respond endpoint for age-unlocked species offers: This supports the age-based bird ramp where unlocks are time-based rather than feature-flag based.

- Visit invite create, revoke, and log endpoints: These implement the opt-in, revocable invitation lifecycle and silent visit log.

- Visitor snapshot endpoint: It is "render-only," rejects event ingest, and logs visit duration silently, preserving read-only ambient visits.

- Account settings endpoint: It persists settings such as reduced motion, captions, visit notifications, and audio.

- Account email-change endpoint: NOT RECOVERABLE FROM PLAN

- Account export endpoint: NOT RECOVERABLE FROM PLAN

- Account delete and recover endpoints: NOT RECOVERABLE FROM PLAN

- Account sessions list and revoke endpoints: These support per-device revocable sessions.

### 5. Simulation engine design

- Tick every about 60 seconds across all accounts: The plan makes the tick the server-side simulation pass that reads events, advances moods and drift, emits canonical state, and writes checkpoints.

- Sharding by account UUID: The plan says this happens "when scale demands."

- Reading events since the last consumed watermark in order: This supports ordered additive personality deltas and idempotent retries.

- Folding presence pings into presence-time per session-window: Presence-time is the dominant aviary-wide drift input, so it must be computed from validated pings.

- Attributing listen-in and offer events to birds: This lets listen-in affect social warmth and vocal frequency per bird, while offers affect curiosity or boldness.

- Low-pass drift update: The plan tunes drift so instruments see change at about one week of regular visits and users feel it at about three weeks.

- Absence applying zero drift delta: This avoids negative decay and Tamagotchi mechanics.

- Drift input weighting: Presence-time is "dominant"; listen-in maps to social warmth and vocal frequency; offers map to curiosity and boldness; settle has no directional drift.

- Mood state machine: Mood responds to recent-session interactions, local time-of-day bands, weather, and personality modulation, giving moods that persist and evolve through absence.

- Time-of-day mood effects: Drowsy near dusk, alert early morning, overnight drowsy to settled to morning alert/content give the aviary day-part behavior.

- Ambient weather effects: Rain dampens vocal frequency aviary-wide, and wind creates an alert/wary split by boldness.

- Bird-to-bird coupling: Response scheduling, wary spread with short decay, and chorus emergence make birds affect each other rather than act independently.

- Next-call scheduling and perch choice: The tick chooses timestamps, motif parameters, seeds, and perch positions from mood and personality so snapshots contain resolved behavior.

- Weather state machine: Rare rain and occasional wind give the snapshot subtle environmental variation.

- Notebook emission from the tick: Notebook entries are occasional and generated from observed candidate events, not direct user-behavior phrasing.

- Checkpoint write and event consumption: Writing canonical state plus a checkpoint and marking events consumed makes retries safe with watermarks and unique event ids.

- Tick p99 latency alarm at 5s: The rationale is operational detection when the simulation pass exceeds the stated latency budget.

- Idempotent tick per account: Watermark plus unique event ids make retries safe.

- Species motif library: Parametric tone envelopes allow species-specific procedural calls without shipped audio files.

- Motif choice weighted by mood: This makes call style reflect mood.

- Parameter jitter with seeded RNG: This supplies variation while remaining reproducible from shipped parameters.

- Personality shaping of calls: Vocal frequency affects rate, and warmth affects response likelihood, connecting call behavior to personality without exposing raw numbers.

- Fixed seed-derived timbre offset: The plan says this gives each bird call signature stability so it stays recognizable across mood and drift.

- Return-greeting greeter selection: Boldness, mood availability, absence length, and staggered offsets produce greeting behavior that varies by bird and absence bucket.

- Notebook candidate detector: Greeting-order novelty, quiet stretches, unusual perches, chorus events, and offer firsts feed entries that observe the aviary.

- Notebook phrase banks and clause composition: The rationale is "strong variation" and avoidance of generic entries that "leak system-ness."

- Notebook sparsity governor: Max about one entry per 2-3 days with a weekly cap keeps the field notebook sparse.

### 6. Sync model

- Single canonical state per account: Multi-device coherence comes from every device reading the same record, with "nothing to merge."

- Snapshot pulls on load, visibilitychange, long frame gaps, and 30s keepalive: This keeps visible clients synchronized with canonical state.

- Interpolating positions between snapshots: The rationale is "no teleporting."

- Clients append events only: This keeps personality changes server-authored and applied in event-log order.

- No last-write-wins anywhere: The plan uses additive server-authored deltas and ordered event consumption to avoid merge conflicts.

- Matter-of-fact auth/session error copy and re-auth: The plan says token, replay, and revocation conflicts are handled with "matter-of-fact error copy and re-auth."

- Two devices interacting simultaneously: The plan says this is safe because events are ordered server-side and the tick serializes consumption per account.

- Visit revocation at snapshot-pull time: The next pull returns the revoked surface, enforcing revocation without event merge complexity.

### 7. Frontend rendering pipeline

- Single horizontal canvas scene: The plan uses one scene with background, middle, and foreground parallax planes, and says all birds are always in frame.

- WebGL or 2D canvas by capability probe: NOT RECOVERABLE FROM PLAN

- SVG/DOM fallback only for unsupported-browser surface: The plan says "we do not maintain old-browser paths."

- Responsive compression and expansion of perch spacing: This keeps all birds in frame with a fixed aspect envelope.

- Snapshot inlined with HTML payload at CDN edge: This is for first-bird visibility under 500ms.

- Birds placed mid-action with phase-offset idle loops: The plan avoids a spinner or entry animation and makes the first frame already alive.

- Quiet field slow-connection loading state: The rationale is to show soft sky and faint motion cues instead of a spinner.

- Initial JS under 2MB and code-splitting settings, visit, and notebook surfaces: This supports the first-frame performance budget.

- Mood-keyed idle micro-motion: Preen, scan, head-tilt, and weight-shuffle use mood and personality-modulated frequency so birds do not feel synchronized.

- Phase-randomized per-bird animation: The plan says this keeps "nothing" from syncing up.

- Render loop halts when hidden: Simulation continues server-side while the client avoids hidden rendering work.

- Ambient ornaments: Leaves and feathers drift client-side at slow random cadence, with no simulation state.

- Day/night palette keyframes: They are driven by local time with gradual transitions and a night state.

- One nocturnal species active at night: NOT RECOVERABLE FROM PLAN

- Weather overlays: They are subtle rain and wind layers keyed from the snapshot, matching the simulation weather state.

- Thin top bar above scene: It contains account, accessibility, notebook, and offer controls while keeping "No UI inside the scene."

- Top bar fade on cursor stillness and return on movement/keyboard: NOT RECOVERABLE FROM PLAN

- Reduced-motion frontend behavior: Cross-fade pose sequences, perch cross-fades, removed ambient drift, and slowed color transitions make reduced motion designed rather than stripped.

- Settle transition: The plan turns settle into a slow evening shift over about 4s with any-click undo.

- Listen-in transition: Slight scale/light on the focused bird plus audio mix ramp provides "gentle camera-agnostic emphasis."

### 8. Audio pipeline

- WebAudio synthesis per bird: Oscillator/noise sources shaped by motif envelopes allow procedural calls with no audio files shipped.

- No audio files shipped ever: This follows the explicit non-goal of no recorded audio and the procedural call design.

- Per-bird timbre offset in audio graph: This supports call signature stability.

- Chorus mixing through per-bird gain nodes and master bus with soft limiting: The plan supports overlapping calls and chorus while controlling the mix.

- Synthesized ambient bed: It stays "very quiet" underneath and remains procedural.

- Listen-in audio mix: Focused bird gain ramps up, others ramp down to ambient floor, and the plan states the reason: "Feels like listening, not soloing."

- Autoplay policy: The AudioContext resumes on first user gesture; until then the aviary plays in "graceful silence" with captions auto-on.

- WebAudio-unavailable fallback: The plan uses captions and silence, because "no recorded-audio path exists."

- Call captions near the calling bird: Captions are generated from the same motif and parameters as the synthesized call, maintaining consistency.

- Preallocated buffer pool and bounded AudioContext: The rationale is avoiding memory growth; CI covers the no-memory-growth test.

### 9. Accessibility surfaces

- aria-live narration region: It gives screen-reader users naturalist observations from snapshot state without turning into a state list.

- Narration update cadence: About one update per 30-60s at idle keeps narration sparse while allowing priority bumps for greetings, offers, and settle.

- Keyboard scene navigation: Tab, arrows, Enter, and Escape make bird focus, listen-in, disengage, offer, and settle keyboard navigable.

- High-contrast focus indicators: They must remain visible against bright and dim scenes.

- Automated contrast checks in CI: The plan verifies WCAG AA on settings, error, and caption surfaces.

- Reduced-motion and captions settings persistence: Settings persist server-side and honor media query by default.

- Accessibility ship gate: The plan makes accessibility surfaces "launch-blocking, not v1.1."

### 10. Performance budgets and observability

- Initial JS budget under 2MB gzipped: This supports first-frame and first-bird speed.

- First bird under 500ms: The plan tests this on a synthetic fleet, mid-tier mobile profile, 4G throttle, and multiple geographies so the quiet scene appears quickly.

- 60fps idle for 30 minutes: This ensures sustained aviary presence on the reference laptop.

- Zero memory growth over 30 minutes: Heap snapshots in CI make long-session memory leaks a hard gate.

- Tick p99 under 5s alarm: This monitors simulation latency.

- Aggregate-only RUM: The rationale is useful timing and error observability without per-bird state or per-account dimensions.

- Metric definition privacy review: Metrics are reviewed against the privacy boundary before shipping.

- Deliberately not measuring relationship reconstruction: The plan refuses per-account drift dashboards, population interaction analysis, and leaderboard substrate to avoid reconstructing a user's relationship with their birds.

### 11. Rollout

- Internal dogfood behind flag: Dogfood calibrates drift k, mood transitions, greeting variation, and notebook sparsity against the one-week instrument and three-week user targets.

- Simulated presence schedules: They assert instrument-visible drift at one week, supporting drift calibration before broader release.

- Private beta: Invite-only accounts watch tick latency, snapshot payload size, audio-context error rates, caption quality, and presence activity window tuning.

- Public v1: It ships the full surface including visits, export, deletion, and accessibility suite.

- Birds-per-aviary ramp: It starts at two and enables age-based offers from day one, with third bird around three months and up toward seven over about a year.

- Day-one instrumentation: Synthetic perf fleet, aggregate RUM, tick latency alarms, error rates, and aggregate sign-in funnel health exist from launch.

### 12. Risks

- Drift calibration controls: The plan calls drift the "Biggest silent-failure risk" and mitigates it with simulation harnesses, beta tuning, and a kill-switch config.

- Presence definition enforcement: The risk is population-wide drift corruption, so the feature needs a well-tested 3-signal conjunction plus server sanity checks.

- Sync correctness enforcement: API schema, code review lint rule, and data-layer tick-only write path exist so the client never writes personality.

- Audio variation tests: Seeded variation, chorus phase randomness, dogfood listening tests, and timbre stability tests protect the product spell and recognizability.

- Accessibility regression gates: Copy review, screen-reader dogfooding, and snapshot tests keep narration and captions in the intended voice and keep reduced motion from becoming "animations off."

- Notebook voice coverage tests: Phrase-bank coverage and human review prevent generic entries from leaking "system-ness."

- Long-session memory leak gate: Audio buffers and notebook DOM are guarded by a CI heap-growth hard gate.

- Gamification and social design-review checklist: The plan rejects every "harmless" streak, toast, or notification proposal by reference to non-goals.
