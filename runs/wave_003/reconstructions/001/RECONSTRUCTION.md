## System-level intent

- Canonical server-authored life: The plan repeatedly makes the server "the only writer" and the simulation "canonical." This shows up in Architecture ("client input -> events -> server tick -> snapshot -> client interpolation"), Sync model ("one canonical state per account"), and Risks ("personality columns writable only by the sim role"). The intent is to make "multi-device sync," "feels alive without the viewer," and "no-last-write-wins" true by construction.
- Calm care without game pressure: Scope excludes "any gamification" and "Tamagotchi mechanics"; the Drift function is "monotonic toward expressive" and says absence creates "ambient quietness" but "never negative trait movement." The risks name the calibration failure as "Too fast -> Tamagotchi; too slow -> screensaver," so the product wants change without distress, punishment, or score-chasing.
- Naturalist voice, not dashboard voice: Notebook entries use "templated naturalist voice" and may reference "bird behavior and scene state only"; screen-reader narration is "written as observations" and "never state lists." Failure surfaces use "matter-of-fact error copy, never naturalist voice," and the risk checklist encodes "notice never announce."
- Designed accessibility as part of the feature, not a pass afterward: Scope includes "naturalist screen-reader narration," "designed reduced-motion mode," captions, contrast, and keyboard navigation. Rollout says "Accessibility surfaces ship with the features they describe -- not a later pass," and reduced motion is "a separate render register, not 'animations off.'"
- Privacy and restraint at the schema boundary: Data uses "synthetic UUIDs" and encrypted email; social visits are "per-invite opt-in read-only," "revocable," and invite-scoped. Observability is "aggregate-only RUM" with "no per-account, per-bird, or interaction dimensions," and the simulation DB is "never read by analytics." Audio also has "No audio files anywhere" and no recorded fallback.
- Performance-first living scene: The boot path targets "first bird painted before any non-critical asset loads," "no entry animation, no spinner," and a "<500ms first-bird budget." Performance budgets add 60fps idle and zero memory growth. The scene starts "already mid-motion" so the first experience is a quiet field, not loading chrome.
- Recognizability over scale: The plan caps birds at seven and says call signatures are "stable per bird across drift"; "recognizability is the design invariant behind the 7-bird cap." Rollout approaches the cap gradually while "listening for chorus-recognizability degradation."

## Per-feature whys

### 1. Scope

- Single-user accounts with email magic-link auth: NOT RECOVERABLE FROM PLAN
- No passwords and no SSO: NOT RECOVERABLE FROM PLAN
- One canonical aviary per account: The plan ties one canonical state to sync: "multi-device sync requires no merge logic -- both devices read one record."
- Two starter birds: NOT RECOVERABLE FROM PLAN
- Cap of seven birds: The cap protects recognizability; the call design says "recognizability is the design invariant behind the 7-bird cap," and rollout listens for "chorus-recognizability degradation."
- Further birds offered by aviary age: NOT RECOVERABLE FROM PLAN
- Server-side simulation tick: The tick "runs with zero clients connected" and, with the client/server split, makes "multi-device sync," "feels alive without the viewer," and "no-last-write-wins" all true at once.
- Mood, drift, and bird-to-bird behavior driven by the tick: The tick is where the worker "applies mood updates, drift deltas, bird-to-bird interactions," keeping these canonical and ordered.
- Multi-device sync as an architectural property: The plan says sync needs "no merge logic" because clients pull snapshots and "write events only"; conflicts are "prevented by construction."
- Return-greeting: NOT RECOVERABLE FROM PLAN
- Listen-in: Listen-in is a drift signal for a targeted bird, shaping "social_warmth" and "vocal_frequency," and the mix keeps other birds at an "ambient floor (never zero)" while focusing one bird.
- Offer of seed, song fragment, or still pool: Offers are used as drift signals; "accepted -> curiosity" and "nearby offer -> boldness." Offer outcomes can also become notebook candidates.
- Settle: Settle "ends the presence window cleanly -- no directional drift," has a lighting shift, and includes a "5-second undo."
- Field notebook: The notebook records sparse "candidate observations" in "templated naturalist voice" from real state, "never numeric" and "never user-behavior observations."
- Presence accounting: Presence-time is the "dominant" drift signal, and the presence-signal risk says a laxer definition like "tab-open" would "silently inflate drift population-wide."
- Three perch zones: NOT RECOVERABLE FROM PLAN
- Day/night cycle on user's local time: Local time feeds mood transitions and persistence, including "drowsy-at-dusk -> settled by morning."
- Ambient weather: Weather shapes mood, call probability through "weather dampening," scene state, and notebook observations.
- Ambient micro-motion: Micro-motion makes the scene continuously alive: idle behaviors include "preen, scan, head-tilt, weight-shift," and the boot path starts "already mid-motion."
- Top-bar chrome with fade: NOT RECOVERABLE FROM PLAN
- Procedural WebAudio call synthesis: The same call schedule and seeds let "the caption, the audio, and any narration all describe the same call"; the plan also requires "No audio files anywhere."
- Chorus mixing: Real-time mixing avoids fake repetition: chorus is "independently varied procedural calls -- never stacked loops."
- Silence plus captions as fallback: When WebAudio is unavailable or denied, the fallback is "graceful silence + captions enabled by default" with "No recorded fallback path."
- Naturalist screen-reader narration: Narration is observational, "naturalist voice," and paced with queue caps so "the reader is never flooded."
- Designed reduced-motion mode: Reduced motion is "a separate render register, not 'animations off,'" preserving a designed experience with pose cross-fades, removed leaf drift, and slowed color cycles.
- Runtime call captions: Captions are generated from "the same call-grammar parameters that produced the audio," so captions track the actual procedural call.
- WCAG AA contrast: The plan requires all user copy to meet "WCAG AA minimum" and be "verified in CI" across lighting states.
- Full keyboard navigation: Keyboard access makes the scene and interaction palette operable without pointer input: arrows move between birds, Enter listens in, Escape exits, and focus indicators stay visible.
- Per-invite opt-in read-only visits: Read-only visits preserve the canonical simulation: visitors emit "no events," the tick "ignores visitor sessions entirely," and social is "read-only consumers of existing snapshots."
- Revocable visits: Revocation has "immediate effect" and produces a matter-of-fact "visit no longer available" response.
- Visit log: NOT RECOVERABLE FROM PLAN
- Opt-in visit notifications off by default: NOT RECOVERABLE FROM PLAN
- 30-day invite expiration: NOT RECOVERABLE FROM PLAN
- Account export as JSON via email link: NOT RECOVERABLE FROM PLAN
- Soft and hard deletion with 30-day window: NOT RECOVERABLE FROM PLAN

### 2. Architecture

- Auth-service: NOT RECOVERABLE FROM PLAN
- Sim-service: The sim-service owns the "canonical simulation," including bird records, personality vectors, mood state, the tick, event consumption, snapshots, and notebook entries.
- Api-service: NOT RECOVERABLE FROM PLAN
- Web client: The web client owns "WebAudio + Canvas/WebGL rendering" while state writing remains server-side.
- Client/server split: The split prevents client-side drift, personality mutation, mood advancement, and merge conflicts; it is the reason multi-device sync and "feels alive without the viewer" hold together.
- CDN-inlined initial snapshot: Inlining snapshots with the HTML response is for the "<500ms first-bird budget."
- Worker fleet keyed by account UUID: This supports the per-account tick; the sync model also requires "one in-flight tick per account" enforced by lock or lease.

### 3. Data model

- Synthetic UUID identifiers: The plan says all identifiers are synthetic UUIDs and email appears only once encrypted; the articulated boundary is privacy-oriented identity separation.
- Encrypted account email: Email appears "once, encrypted, on the account record," and visitor emails are also encrypted.
- Account settings for visit notifications, reduced motion, and captions: NOT RECOVERABLE FROM PLAN
- Session records with device label and revoked flag: NOT RECOVERABLE FROM PLAN
- Bird personality vector: The personality vector is the target of long-term drift, with traits normalized and changed by server-authored deltas.
- Bird drift history for instruments only, never surfaced: This supports calibration and telemetry while keeping drift hidden from the user-facing product.
- Species assets, palettes, and motif libraries: Species include silhouettes/SVG assets, palette, and call grammar motifs so rendering and call synthesis can be species-shaped.
- One nightjar-like nocturnal signature: NOT RECOVERABLE FROM PLAN
- Append-only InteractionEvent log: The log lets clients append events without writing state; events are "ordered per account" and "consumed by the tick in order."
- Client-supplied event timestamps and server timestamps: NOT RECOVERABLE FROM PLAN
- Snapshot: The snapshot is "derived" canonical state at tick N and is what clients render and interpolate.
- NotebookEntry generation context internal only: Keeping generation context internal supports notebook entries that reference behavior and scene state, not numeric or user-behavior data.
- VisitInvite token, expiration, revoked flag, and consumed flag: The invite model supports invite-scoped visitor access, 30-day expiration, revocation, and single consumption.
- VisitLogEntry: NOT RECOVERABLE FROM PLAN

### 4. API surface

- Per-account scoping everywhere: Per-account scoping aligns with one canonical state per account and invite-scoped visitor access.
- Visitor invite-scoped tokens: Visitor tokens constrain visitors to the read-only snapshot endpoint.
- `POST /auth/magic-link`: NOT RECOVERABLE FROM PLAN
- Magic-link rate limiting, 15-minute expiry, and single-use: NOT RECOVERABLE FROM PLAN
- `POST /auth/consume`: NOT RECOVERABLE FROM PLAN
- `GET /state/snapshot?since=N`: Snapshot pulls support load, visibility changes, long frame gaps, keepalive, and stale-client recovery without client state writes.
- `POST /events`: Event ingestion lets clients record interactions idempotently while never carrying "absolute personality values."
- `GET /notebook?cursor=`: NOT RECOVERABLE FROM PLAN
- Paged notebook entries oldest scrollable indefinitely: NOT RECOVERABLE FROM PLAN
- Bird rename and bird list endpoints: NOT RECOVERABLE FROM PLAN
- Visit invite creation, revocation, and log endpoints: Revocation is immediate, and the log endpoint exposes the host's visit log; no deeper rationale is articulated.
- `GET /visit/{token}/snapshot`: The visitor snapshot endpoint gives read-only access and returns matter-of-fact unavailable copy when revoked or expired.
- Account export, delete, and recover endpoints: NOT RECOVERABLE FROM PLAN
- Server-gated adoption: Adoption is "gated server-side on aviary-age offers," keeping bird growth canonical.
- Visitors emit no events and do not affect the tick: This keeps visits read-only and prevents visitor sessions from changing simulation state.

### 5. Simulation engine design

- Tick cadence every about 60 seconds: NOT RECOVERABLE FROM PLAN
- Tick runs with zero clients connected: This is the mechanism for the aviary to "feel alive without the viewer."
- Tick p99 latency alarm at 5 seconds: NOT RECOVERABLE FROM PLAN
- Drift low-pass filter over presence-and-interaction signals: The low-pass filter lets traits change gradually from presence, listen-in, and offers.
- Presence-time as dominant drift signal: Presence-time is the strongest signal, and the risk section says honest presence prevents tab-open inflation.
- Listen-in drift weights: Listen-in affects the target bird's "social_warmth" and "vocal_frequency."
- Offer drift weights: Accepted offers increase "curiosity"; nearby offers increase "boldness."
- Settle as no directional drift: Settle only "ends the presence window cleanly."
- Monotonic toward expressive traits: "No trait ever decreases," preventing negative trait movement and supporting non-Tamagotchi care.
- Absence as ambient quietness: Absence changes expression by lowering greeting frequency, "never negative trait movement."
- One-week instrument drift and three-week user-visible drift targets: The targets calibrate drift between "Too fast -> Tamagotchi" and "too slow -> screensaver."
- Time-compressed simulation harness: The harness tunes weights against the one-week and three-week targets before launch.
- Mood enum: NOT RECOVERABLE FROM PLAN
- Mood transition inputs: Mood uses recent session events, local time, weather, and personality so state responds to both context and individual birds.
- Mood persists across sessions: Persistence prevents mood from resetting on connect and allows absence evolution like "drowsy-at-dusk -> settled by morning."
- Species motif library: Motifs provide timing patterns, pitch contours, and syllable primitives for procedural call grammar.
- Tick-time call scheduling: Calls are shaped by vocal_frequency, mood, chorus opportunities, weather dampening, and night/nocturnal exceptions.
- Deterministic client synthesis from motif, seed, mood, and personality: Determinism makes caption, audio, and narration describe the same call.
- Stable call signatures per bird: Stability keeps birds recognizable across drift and underwrites the seven-bird cap.
- Bird-to-bird response checks: Warmth-weighted responses, local wary spread, and overlapping call windows create choruses without an agent framework.
- Notebook candidate observations: Candidates come from greeting order changes, quiet stretches, weather reactions, and offer outcomes so entries reflect real behavior and scene state.
- Notebook sparsity gate: The gate keeps entries around "1 entry per few days" while letting noteworthy events break through.
- Notebook data access hard rule: Entries can never reference visit frequency or user stats, preserving the naturalist voice and privacy boundary.

### 6. Sync model

- Single canonical state per account: This gives multi-device sync "no merge logic."
- Clients pull snapshots and interpolate: This keeps clients as renderers, not state writers.
- Clients write events only: Events allow interactions without client mutation of bird state.
- Additive server-authored personality deltas: Additive deltas applied in event-log order prevent absolute trait conflicts.
- Event idempotency and per-account ordering: Deduping replayed events and ordering per account make event ingestion safe.
- Single-writer tick per account: The lock or lease ensures one in-flight tick per account.
- `since=N` staleness handling: Clients recover from stale snapshots by pulling canonical state.
- Matter-of-fact failure copy: Errors avoid naturalist voice when the system is explaining magic-link replay, timeouts, or outage.

### 7. Frontend rendering pipeline

- Canvas/WebGL scene with 2D fallback: NOT RECOVERABLE FROM PLAN
- DOM top bar and settings surfaces: NOT RECOVERABLE FROM PLAN
- Procedural or compact bird assets: NOT RECOVERABLE FROM PLAN
- Single horizontal scene with middle-plane birds and subtle parallax: NOT RECOVERABLE FROM PLAN
- Responsive scale-and-respace: The scene "never crops a bird at any viewport."
- Minimal boot JS and inlined first snapshot: This gets "first bird painted before any non-critical asset loads."
- No entry animation or spinner: The cold-load fallback is the "quiet-field state," keeping the scene calm from first paint.
- Code-split settings, visits, and notebook: These load on demand to support the boot budget.
- Idle micro-motion from snapshot state and local animation clock: This creates continuous mood-shaped behaviors while preserving canonical server state.
- Client interpolation between snapshots: Interpolation makes perch moves smooth without advancing simulation on the client.
- Pause rendering when tab hidden and re-pull on resume: This saves render work and avoids "catch-up snapping" while the server simulation continues.
- Slow-ramp transitions: Greeting, listen-in focus, offer reactions, settle shift, undo, and top-bar fade are "all slow ramps, no hard cuts."
- Reduced-motion render register: This preserves a full designed motion-reduced version rather than turning animations off.

### 8. Audio pipeline

- WebAudio per-bird synth voices: Per-bird voices turn species motif primitives into procedural calls with no audio files.
- Seeded per-call variation: Variation keeps calls from becoming repetitive and supports captions matching actual audio.
- Per-bird gain and pan buses: Buses let chorus and listen-in adjust individual birds.
- Ambient master: NOT RECOVERABLE FROM PLAN
- Chorus as real-time mixing: It is "never stacked loops," protecting the spell from repetitive or phase-locked calls.
- Listen-in bus ramps: The focused bird ramps up and others ramp down "never zero," preserving focus while keeping the aviary present.
- Symmetric disengage ramp: NOT RECOVERABLE FROM PLAN
- WebAudio fallback to silence and captions: This is a graceful fallback with captions default-on and no recorded audio path.

### 9. Accessibility surfaces

- Live-region screen-reader prose from snapshot state: Using the same snapshot keeps narration aligned with real scene state.
- Idle narration cadence and priority bump: Cadence avoids flooding, while user-initiated events can be surfaced promptly.
- Narration queue cap: Queue caps ensure "the reader is never flooded."
- Per-call captions near the calling bird: Captions fade with the call and use the same parameters as audio, tying text to actual call events.
- Captions opt-in and default-on without WebAudio: NOT RECOVERABLE FROM PLAN
- Reduced-motion surface: It is a v1 surface and parallel render register, not a stripped fallback.
- Keyboard path through top bar and scene: Keyboard support makes birds, listen-in, offer palette, and settle reachable.
- Visible high-contrast focus indicators: Focus indicators remain visible "against all lighting states."
- Contrast CI per lighting state: Automated checks verify WCAG AA on chrome, settings, captions, and errors.

### 10. Performance budgets and observability

- Initial JS under 2MB gzipped: NOT RECOVERABLE FROM PLAN
- First bird visible under 500ms: The boot path and CDN-inlined snapshot are organized around the first-bird budget.
- 60fps idle on a five-year-old laptop for 30 minutes: NOT RECOVERABLE FROM PLAN
- Zero memory growth over 30 minutes: The plan names CI heap snapshots, audio buffer pooling, and no retained notebook DOM as mitigation.
- Synthetic browser fleet from common geographies: NOT RECOVERABLE FROM PLAN
- Aggregate-only RUM: The privacy boundary says telemetry has no per-account, per-bird, or interaction dimensions.
- Tick latency instrumentation and p99 alarm: This watches canonical simulation health.
- Simulation DB never read by analytics: This enforces the privacy boundary at the metric schema level.
- Last-two-majors browser support: NOT RECOVERABLE FROM PLAN
- Matter-of-fact unsupported-browser surface: Unsupported browser errors use the same matter-of-fact voice split as other failure surfaces.

### 11. Rollout

- Foundation first: Auth, UUID account model, event log, and tick skeleton establish the canonical architecture before richer behavior.
- Scene first: The boot path and three-perch scene are validated against the "<500ms budget early."
- Engine after scene: Drift, mood, and calls are built behind the time-compressed calibration harness to tune the core simulation before launch.
- Notebook generator with sparsity gate: This keeps naturalist entries rare and tied to real state.
- Audio after engine: Synth voices, chorus, listen-in ramps, captions, and fallback depend on call scheduling and accessibility alignment.
- Accessibility surfaces shipped with features: The plan says they are "not a later pass."
- Social last: Visits are "read-only consumers of existing snapshots," so they come after the core canonical state exists.
- Internal accounts to invite-only beta with two birds: NOT RECOVERABLE FROM PLAN
- Opening third-bird age offers after drift calibration: This waits until drift calibration is "confirmed in production instruments."
- Approaching seven-bird cap gradually: The reason is to listen for "chorus-recognizability degradation."
- Instrumented from day one: Performance budgets, tick latency, and error rates are tracked "all aggregate-only."

### 12. Risks and mitigations

- Drift calibration harness and adjustable server-side weights: This mitigates the silent failure between "Tamagotchi" and "screensaver."
- Shared presence module with unit tests: This mitigates laxer tab-open presence inflating drift population-wide.
- DB role restriction on personality columns: This enforces the no-client-state-write rule behind sync correctness.
- Event ingestion schema with no trait fields: This prevents clients from submitting trait values.
- Single-tick-per-account lease under failover: This protects the single-writer tick model.
- Seeded per-call variation and anti-repetition windows: This mitigates repetitive or phase-locked calls.
- Staggered multi-bird greetings: This protects audio naturalness and recognizability.
- Listening tests against the seven-bird recognizability ceiling: This validates the cap and chorus complexity.
- Narration soak tests and queue caps: This mitigates screen-reader flooding.
- Reduced-motion design review: This prevents reduced motion from becoming a stripped fallback.
- Axe CI on all copy surfaces: This catches accessibility regressions.
- Copy linting and content review checklist: This blocks voice drift such as "a toast, a streak, a numeric stat, or an event-log-style notebook entry."
- Bundle-size CI gate at 2MB and first-bird synthetic check: These mitigate performance regressions from audio or asset additions.
