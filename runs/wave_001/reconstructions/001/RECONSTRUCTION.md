## System-level intent

- Quiet, ambient presence over game pressure or announcement surfaces. This shows up in the hard boundary against "Gamification of any flavor" and "Any Tamagotchi-style mechanic," in "no assertive weather," in new bird offers that are "not a push notification, not a toast," and in the production-risk language "Notice, never announce."

- Server-authoritative, canonical state rather than client negotiation. The plan repeats that the server owns "Canonical aviary state," "Simulation tick computation," and mood/personality writes; the client "never" writes personality values, computes drift, generates mood transitions, or runs a tick. The sync model says conflict is "prevented (not resolved)" because there is "no client-side state to merge."

- Slow, expressive growth without punishment. Drift is "monotonic upward (toward expressive)," traits "never decrease," and "Zero drift on neglect" means birds are "not more wary, not less colorful, just growing more slowly." The calibration targets also protect against single-session changes that would make the aviary feel "Tamagotchi-like."

- Naturalist voice as the product voice. The field notebook is "naturalist prose"; notebook entries are "lowercase, present-tense"; call captions are in "naturalist voice"; screen-reader narration reads as "observer's present-tense notes" and "not a state list." The plan specifically rejects wording like "plumage increased" in favor of a naturalist observation.

- Privacy by architecture, not only policy. Email is encrypted and "never used as identifier"; visitor events are not recorded; telemetry is "aggregate only, no per-bird data"; the plan deliberately does not measure "any metric that could reconstruct a user's relationship with their birds." The analytics warehouse has "no read access" to Aviary DB as an "architectural rule enforced at the network/IAM level."

- The aviary should already feel alive on first paint. Performance budgets include "first bird visible <500ms"; first-frame rendering says "No spinner, no fade-in" and "The first paint includes the aviary." Bundle creep is framed as a risk because it breaks the "'aviary already in motion' conceit."

- Accessibility is the same product, not a stripped fallback. Reduced-motion mode is "cross-fade rendering, not stripped fallback," and later "a different renderer for the same state, not a different product." Screen-reader narration, captions, keyboard navigation, focus indicators, and WCAG AA contrast all carry the same naturalist/aviary state rather than a separate summary mode.

- Procedural generation with stable authored constraints. Calls are generated through WebAudio from species grammars, with "no recorded audio"; notebook and narration prose are template-driven, "not LLM-generated," to avoid "latency, cost, and voice drift." Deterministic seeds make procedural timing align across devices.

- Sharing is deliberate, bounded, and non-social. Visit invitations are "off by default," sent to a "specific email address," read-only, revocable, and visitor events do not affect the event log. This sits next to hard boundaries against "Social-network surfaces" and "Shared or multi-user aviaries."

## Per-feature whys

### 1. Scope

- Browser-based virtual aviary: NOT RECOVERABLE FROM PLAN

- Two starter birds, cap at seven birds, and age-unlocked slots: The plan ties growth to aviary age and "age-appropriate intervals," with starters fixed at two and later birds appearing only after thresholds. In rollout, the new bird offer is naturalist-voiced and "not a push notification, not a toast," keeping growth slow and ambient.

- Six species in the initial pool: NOT RECOVERABLE FROM PLAN

- System-chosen starter species: NOT RECOVERABLE FROM PLAN

- Email + magic-link authentication: NOT RECOVERABLE FROM PLAN

- Per-device session tokens and session revocation: Per-device records support listing sessions, showing a "device_label" that is "display only," tracking "last_seen_at," and setting "revoked_at." The account settings feature can revoke a single session rather than the whole account.

- Single canonical aviary per account and single-user accounts: The sync model says the server is the only writer and both clients pull from the same record, so there is "no synchronization protocol between clients," "no client-side state to merge," and "no eventual consistency window."

- Multi-device sync via server-authoritative state: The rationale is that personality vectors and moods are written only by the simulation tick. Event logs are append-only, so multiple devices can submit valid events without write conflicts.

- Server-side simulation tick: The tick is "not driven by client connections" and runs continuously. It centralizes drift, mood, weather, notebook generation, and tick timestamps so clients render snapshots rather than running the simulation.

- Procedural call grammar synthesized client-side via WebAudio API: The plan bans recorded audio and uses motif grammars so calls "vary procedurally" while remaining recognizably a bird's species. WebAudio also keeps audio out of bundled/fetched audio files.

- Presence accounting with visibility, focus, and recent pointer/key activity: The event model records active presence windows with `activity_confirmed`; the server validates windows and caps outliers. The why is to credit measured active presence, not merely an open tab.

- Return-greeting interaction: The plan grounds this through `greeting_received`, high-priority screen-reader announcement, and notebook triggers such as a bird greeting first for the first time in a week. The why is to let greeting become part of observed aviary behavior.

- Listen-in interaction: Listen-in changes the audio mix by raising the focused bird and lowering other birds to 0.15, "not 0, so the aviary remains a place with multiple things happening." It also feeds social warmth and vocal-frequency drift through listen-in duration.

- Offer interactions: Seed, song, and still-pool offers become event-log inputs for drift, especially boldness and curiosity. The server validates plausibility so an offer cannot target a bird outside the aviary.

- Settle interaction: Settle shifts lighting toward evening, reduces audio, and moves birds toward drowsy poses. The 5-second undo window makes the transition reversible before it becomes part of the quiet scene.

- Field notebook: The notebook is read-only, sparse, and naturalist-voiced. It records observations like long quiet periods, plumage bucket changes, weather, or first greetings without exposing internal labels or turning changes into user tasks.

- Day/night cycle keyed to local timezone: Time of day drives rendering and mood rules: early morning can become alert, near dusk becomes drowsy, and night becomes drowsy or settled. Local time lets the aviary light and behavior match the user's day.

- Ambient weather events: Weather is occasional and non-assertive. Rain and wind influence moods, and weather can become a notebook observation, but the plan keeps weather ambient rather than announced.

- Ambient scene micro-motion: Leaves, feathers, parallax, and idle animation make the aviary read as alive between server ticks. The client drives this locally so the scene can keep moving without additional server input.

- Visit invitation feature: The feature is specific, read-only, revocable, and expires after unused time. It allows a visitor to see an ambient view without creating social-network surfaces or letting visitor behavior affect host drift.

- Visit log and opt-in visit notification toggle: Visit notifications are opt-in and off by default, matching the hard boundary against push/email notifications about aviary state. The visit log gives account-setting visibility without turning visits into public or social surfaces.

- Account export: NOT RECOVERABLE FROM PLAN

- Soft-delete 30-day window and hard delete after window: NOT RECOVERABLE FROM PLAN

- Screen-reader narration: The narration is naturalist prose on a slow cadence, with prompt updates for user-initiated events. It avoids being "a state list" and shares sentence templates with the field notebook.

- Reduced-motion mode: Reduced motion uses slow cross-fades and still poses so the aviary "still reads as alive." It uses the same snapshot, mood, and perch-zone data as full motion, making it "a different renderer for the same state."

- Call captioning: Captions are generated from motif parameters at call time, in naturalist descriptions such as shape, pitch, and repetition. They also become the default fallback when WebAudio is unavailable.

- WCAG AA contrast: The plan requires all product text, captions, settings, notebook entries, and error surfaces to pass WCAG AA. Caption scrims are added when needed so the aviary background does not make text inaccessible.

- Full keyboard navigation and visible focus indicators: Keyboard navigation makes birds, listen-in, offer options, and exit paths reachable. Focus indicators are designed to remain visible against both morning and evening palettes.

- Performance targets: The budgets protect the first-bird-visible promise, smooth idle animation, long-session stability, and reliable server ticks. The risks section says bundle creep would break the "'aviary already in motion' conceit."

### 2. Architecture, data model, and API surface

- Auth Service, Aviary API, Simulation Worker, and Notification Worker split: The services separate magic-link/session work, state/event/notebook/visit APIs, periodic simulation, and async email dispatch for magic links, export links, and opt-in visit emails.

- Server-owned canonical state: The server owns personality vectors, moods, tick computation, event persistence, notebook generation, account/session records, and visit authorization so the simulation has a single source of truth.

- Client-owned rendering, presence batching, procedural audio, interpolation, and local time: The client owns continuous presentation work that can happen between server ticks, while the server still controls state boundaries.

- Client never writes personality, computes drift, generates mood transitions, or runs a tick: This prevents hidden personality state from becoming user- or client-controlled and preserves the server-side simulation as the only writer.

- Snapshot-plus-interpolation render boundary: The server sends snapshots at tick boundaries; the client interpolates between them, synthesizes calls, and runs idle animation locally. This gives smooth motion without moving simulation authority to the client.

- Encrypted email stored once and never used as identifier: This supports the privacy principle that account identity should not be built around raw email.

- Session records with display-only device labels, last-seen timestamps, and revocation timestamps: These fields support account settings where sessions can be listed and revoked without turning the device label into an identity key.

- Aviary record unique per account: The unique account-to-aviary link enforces "one aviary per account," which also supports the single canonical sync model.

- Bird stable identity that is never reused or reassigned: The renderer uses `bird.id` as a deterministic seed for stable perch placement across loads without storing per-bird pixel coordinates server-side.

- Bird name mutable but never affects personality: The plan separates user naming from simulation personality so renaming does not change the bird's behavior or drift.

- Personality fields never returned in client-facing API responses: The plan keeps raw vectors internal to the simulation service and exposes only bucketed or rendered effects.

- Append-only host event log: Append-only events avoid write conflicts, preserve inputs for the next tick, and keep visitor sessions from affecting host simulation because visitor events are not recorded.

- Presence pings with validated windows and capped outliers: The server accepts network delay as normal, capping outliers rather than rejecting them, while rejecting impossible windows such as `ended_at < started_at`.

- Notebook entries with triggers, rate limits, and indefinite scrollback: Triggers create sparse observations; rate limits prevent overproduction; reverse chronological fetch with no pagination cap lets the user "scroll back indefinitely."

- Magic-link token one-time 15-minute TTL: NOT RECOVERABLE FROM PLAN

- Visit invitation visitor email not requiring registered user: NOT RECOVERABLE FROM PLAN

- Visit invitation token, expiry, and revocation: The token enables a bounded read-only visit session, the expiry limits unused invitations, and revocation cuts off access at snapshot-pull time.

- Snapshot response omits personality values and buckets plumage: The plan says `plumage_level` is the only personality-derived client field and is bucketed "specifically to avoid communicating numerical precision."

- Interaction event validation that logs and skips invalid events without rejecting the whole batch: This preserves valid batched events even when one event is implausible, such as a target bird not in the aviary.

- Visit snapshot subset and pull-time revocation check: Visitor snapshots omit settings and next slot unlocks, preserving a read-only ambient view. Pull-time revocation returns a matter-of-fact `visit_revoked` response.

- Account email update requiring re-verification: NOT RECOVERABLE FROM PLAN

- Account restore within the deletion window: Restore exists because deletion is soft for 30 days before hard delete, giving the pending deletion a reversible state.

### 5. Simulation engine design and sync model

- Durable scheduled tick with per-aviary advisory lock: The tick runs independent of client connections and uses locking/concurrency controls so one aviary is not ticked concurrently.

- Tick reads event log since the last tick and computes presence time: The tick consumes only recent events, sums validated presence windows, and turns interaction history into the next canonical state.

- Additive drift deltas, clamping, and no client-submitted absolute trait values: The server applies small deltas and clamps traits to `[0.0, 1.0]`, avoiding any client path that can set personality directly.

- Monotonic upward drift and zero drift on neglect: The plan's why is explicit: fewer inputs mean birds grow more slowly, "not more wary, not less colorful." This avoids a decay-on-neglect loop.

- Drift calibration targets for 7 days, 21 days, and single-session limits: The targets prevent drift from being too fast or too slow. A single session must not move a bucketed output, while regular visits should become measurable and then visible over weeks.

- Mood transitions from time of day, recent interactions, personality, and weather: These inputs let mood reflect the aviary's current conditions without requiring the client to run transition logic.

- Probabilistic mood persistence: Applying the target mood probabilistically "avoids snap transitions" and lets mood change feel gradual.

- Species call grammars with motifs: Motif parameter sets keep each species recognizable while allowing procedural combinations, pitch curves, envelopes, and repetition patterns.

- Server-provided call seed, timing offset, and derived timing modifier without raw vocal-frequency exposure: The server shapes call frequency but does not expose `pv_vocal_frequency` directly; the client receives only small derived values.

- Bird-to-bird chorus triggers from deterministic seed: Cross-bird triggers create chorus behavior, and shared deterministic seed means devices for the same account produce the same chorus timing.

- Notebook generation worker separate from the simulation tick with shared lock: Separate generation keeps prose work out of the tick while the shared account-level lock prevents concurrent tick/notebook writes.

- Template-driven notebook prose, not LLM-generated: The plan says this avoids "latency, cost, and voice drift"; specificity comes from trigger-specific templates rather than long automatic generation.

- Conflict prevented rather than resolved: Personality and mood writes have no client path; event logs are append-only; notebook entries have a single writer. This removes most conflict surfaces by construction.

- Concurrent presence deduplication and cap: Overlapping sessions should not inflate presence. The tick credits overlapping duration once and caps total credited presence per tick cycle.

- Magic-link compare-and-swap on `consumed_at`: Concurrent link clicks are handled at the database level so only one consume succeeds and the other gets "link already used."

- Snapshot freshness pulls on visible tab, long render gaps, and keepalive: These triggers refresh the snapshot after visibility changes, laptop suspension, or normal tick cadence.

- Per-account CDN-cached snapshots with short TTL: Snapshots are small and cacheable so both devices can get a fresh response quickly without hitting the database on every pull.

### 7. Frontend rendering pipeline, audio pipeline, and accessibility surfaces

- React or Preact with canvas/WebGL, with SVG/CSS alternative benchmarked: The plan evaluates Three.js, PixiJS, and SVG/CSS against the 2MB bundle cap, so technology choice is constrained by first-load budget.

- Code splitting for account settings, accessibility settings, notebook, and visit invitation flow: These are lazy-loaded because only the aviary scene belongs in the initial bundle.

- SVG-based bird sprites and bucketed HSL plumage shifts: SVG/JSON assets keep bundle weight down, and `plumage_level` drives richness without exposing raw `pv_plumage_sat`.

- Horizontal scene with front, middle, and back perch zones plus layers: The layered scene supports depth through background, mid-plane, and foreground planes; responsive compression keeps "all birds in frame."

- Deterministic bird placement seeded by `bird.id`: This maintains consistent perch positions across loads without storing server-side pixel coordinates.

- First-frame rendering with inline snapshot, mid-action poses, and no spinner or fade-in: The plan wants birds visible immediately and already mid-action, so the first paint includes the aviary rather than a loading state.

- Quiet field fallback before JS runs: The fallback gives visual continuity with muted sky and subtle leaf drift while avoiding the word "loading."

- Idle micro-motion playlists parameterized by mood and species: Weighted playlists give "non-deterministic variation within the mood's affective register" without server input between ticks.

- Cross-fade between snapshot mood or perch-zone changes: Cross-fading prevents the bird from snapping when the server snapshot changes.

- Top-bar fade to 15% opacity after cursor stillness: NOT RECOVERABLE FROM PLAN

- Settle UI lighting, audio, and drowsy transition with undo: The transition moves the aviary toward evening and quieter audio, while any click/tap within 5 seconds can reverse it.

- "Undo settle" temporary label: The label appears only during the 5-second window and is "not permanent UI chrome," keeping the affordance small and time-bound.

- WebAudio node graph with oscillator, gain, filter, panner, and per-bird mix: The graph synthesizes calls, shapes species formants, positions sound by bird location, and supports listen-in mixing.

- Pitch variation within motif ranges: Slight random deviation is "the source of procedural variation" so repeated calls do not sound identical.

- `AudioContext.currentTime` scheduling and call timing offsets: Current-time scheduling avoids jitter, and offsets prevent every bird from starting calls simultaneously.

- Per-bird motif scheduler state machine: The scheduler selects motif sequences, fires calls at the right time, and scales intervals with the personality-derived timing modifier.

- Chorus mixing for simultaneous calling birds: Because calls are procedural rather than looped files, simultaneous streams avoid phase-cancellation artifacts.

- Listen-in gain ramp: The focused bird rises to 1.0 while others ramp to 0.15, preserving the sense that the aviary has multiple things happening.

- Runtime call captions not stored: Captions are generated fresh from envelope, pitch, and repetition at the moment of each call, keeping text tied to the actual synthesized sound.

- WebAudio unavailable fallback: The aviary remains fully visual, captions turn on automatically, a one-time matter-of-fact notice appears, and no recorded-audio fallback is attempted.

- Screen-reader live regions: Idle narration updates politely every 30-60 seconds, while high-priority return-greeting events use a separate assertive live region for prompt announcement.

- Visually hidden narration region: Off-screen CSS lets screen readers announce prose while sighted users do not see a text block.

- Keyboard navigation through top bar, aviary, birds, listen-in, escape, and offer panel: The plan makes bird focus and offer options reachable without pointer input.

- Focus indicators as SVG overlay on the canvas: A DOM focus ring would not render over the canvas surface, so the outline is rendered as an overlay and tuned for contrast.

- Caption and text contrast with scrim if needed: Text must pass WCAG AA against changing aviary palettes; the scrim is the plan's way to preserve readability over visual backgrounds.

- Accessibility settings persisted server-side: Reduced motion, captions, narration, and text size follow the user across devices instead of living only in localStorage.

### 10. Performance budgets, observability, rollout, and risk controls

- CI budget enforcement for bundle size, first bird visible, frame rate, memory, and tick latency: The plan blocks PRs or alerts on violations so performance targets remain build constraints, not wishes.

- Bundle size strategy using lazy loading, no recorded audio, SVG/JSON assets, tree-shaking, and dependency audits: This protects the initial aviary load and the <2MB gzipped budget.

- Aggregate-only measurement: Request latency, tick duration, magic-link rates, render timing, WebAudio errors, and visit pulls are measured without account-level or per-bird attribution.

- Deliberately unmeasured telemetry categories: The plan refuses per-account interaction history, per-bird trait values, attributable visit behavior, streak-like visit frequency, and relationship-reconstructing metrics.

- Analytics warehouse network/IAM isolation from Aviary DB: The why is to make privacy an "architectural rule" rather than relying on policy alone.

- Synthetic browser runners with dedicated test accounts: Synthetic flows measure first bird visible, frame rate, memory, and offer submission every 15 minutes without affecting user drift.

- Pre-launch internal sequence: The sequence validates species/call grammars, drift calibration, performance budgets, keyboard/screen-reader behavior, and reduced motion before any user accounts exist.

- Soft launch closed beta: Invite-only beta gives 100-500 accounts for monitoring first-bird render p95, tick p99, WebAudio error rate, anonymized drift calibration, and blocking broader launch on p0 issues.

- Open launch bird-slot and visit-invite posture: Even after signup opens, bird slots remain at two starters and visit invitation still requires a deliberate host invite, preserving the slow-growth and bounded-sharing model.

- Open launch remove invite gate: NOT RECOVERABLE FROM PLAN

- Bird slot unlock offer as naturalist-voiced surface: A new bird appears near the aviary "watching from the tree line," avoiding push notifications and toasts while making growth part of the scene.

- Day-one instrumentation checklist: The checklist makes tick latency, magic-link errors, first-bird rendering, WebAudio errors, drift calibration, memory growth, and bundle size visible from launch.

- Personality vector backups, single write path, and decrease alerts: These protect the silent-feeling identity of a bird; any unexpected decrease violates the monotonic invariant and becomes a bug signal.

- Sound-designer-authored motif parameters and natural imperfections: This mitigates "audio uncanniness" by tuning calls by ear and adding pitch wobble, timing variation, and false starts instead of recorded audio fallback.

- Shared narration/notebook templates, reduced-motion hardware QA, axe-core scan, and focus screenshot comparison: These controls keep accessibility surfaces from diverging or regressing across releases.

- Product-review checklist blocking toasts, badges, streaks, and announcement surfaces: The plan treats such additions as "collectively fatal to the product's affective model" and blocks them pending explicit product-owner sign-off.

- Bundle-size PR comment and dependency justification: These prevent incremental dependency creep from crossing the 2MB limit and breaking time-to-first-bird.

- WebAudio synthetic browser coverage, smoke tests, versioned motif parameters, and browser-specific overrides: These controls mitigate browser behavior changes that could alter call timing or character.
