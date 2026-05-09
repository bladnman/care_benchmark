## System-level intent

- Server-canonical, client-presentational product shape. This shows up wherever the plan says the "Server owns" personality vectors, mood, tick clock, event log, notebook entries, visit state, and canonical aviary state, while the "Client owns: nothing persistent." The split is repeated in "Client never writes personality state directly" and in the sync section: "The server is the only writer of state."

- Conflict-free multi-device sync by design. The plan carries "one canonical server record, no merge needed" from scope into the sync model: "No last-write-wins, no vector clocks, no merge. The append-only event log is the conflict-free data structure."

- Quiet aliveness instead of game pressure. The non-goals reject "streaks," "achievements," "levels," "badges," "green-dot calendars," "visit counters," and Tamagotchi mechanics where birds "die," have a "hunger meter," or show "distress." The first-frame strategy reinforces this by having the client "join the aviary mid-scene" with "No entry animation" and "No fade-from-black."

- Slow, monotonic, expressive change. The personality system is "monotonic-toward-expressive drift"; the drift function uses `max(0, delta)` and "never negative delta." The calibration target is change in instruments after about one week and perceptible change "after ~3 weeks," while the risk section warns against changing too fast into a "Tamagotchi feel" or too slow into a "screensaver feel."

- Naturalist voice across all prose surfaces. The field notebook uses "naturalist prose, sparse cadence"; notebook entries are "lowercase, present-tense"; screen-reader narration uses the "same register" as the notebook; captions are prose descriptions such as "a soft three-note rise." The risk section names the failure mode as narration and captions drifting until "a screen-reader user hears two products."

- Accessibility is part of the core experience, not a separate mode. Scope includes screen-reader narration, reduced motion, call captioning, WCAG AA contrast, and full keyboard navigation. Reduced motion is "cross-fade poses, not animations-off," and accessibility surfaces keep "call captions, narration, all semantic content."

- Privacy and observability boundaries are explicit. Scope says "Aggregate operational telemetry only; no per-bird state in any telemetry pipeline." Observability repeats that metrics contain "none" of "per-bird or per-account interaction data," and the warehouse never receives personality vector values, mood sequences, offer/listen-in history, or individual presence-session durations.

- Procedural, local audio rather than recorded media. Scope states "Procedural call grammar synthesized client-side via WebAudio" and "No recorded audio." The audio fallback proceeds "in silence" with captions rather than substituting recordings, and audio motif data are "small JSON descriptors, not audio files."

- Performance is framed around immediate presence. The plan sets "Time to first bird visible: <500ms," embeds the first snapshot in HTML, uses a CSS-only "quiet field" fallback, and targets "60fps idle motion on a 5-year-old laptop." The first frame goal is a bird at the correct position "in a mid-action pose within 500ms."

- Sharing stays opt-in, read-only, revocable, and non-social. Scope describes visit invitation as "opt-in, read-only, revocable, email-keyed, not co-presence." Non-goals reject "profiles," "follows," "discovery feed," "comments," and "public aviaries." Rollout says visit invites are enabled but "off-by-default" with "no invite prompt during onboarding."

## Per-feature whys

### Scope

- Two starter birds per account and growth to seven: the plan ties growth to time by saying availability is "age-gated, not interaction-gated," matching the anti-gamification intent and avoiding interaction pressure.

- Magic-link email auth: NOT RECOVERABLE FROM PLAN

- Single-user accounts and one aviary per account: NOT RECOVERABLE FROM PLAN

- Server-side simulation tick: the tick advances the canonical personality vectors and mood because the server owns simulation state and the client "never owns simulation state."

- Client snapshot rendering with smooth interpolation: the client renders snapshots and interpolates so it can present motion while persistent state remains server-owned.

- Multi-device sync: the why is "one canonical server record, no merge needed"; multiple devices read the same record and write only append-only events.

- Procedural call grammar synthesized client-side via WebAudio: this supports the "WebAudio-only" and "No recorded audio" boundary while allowing runtime variation and captions generated from what actually played.

- Mood system with daily-ish reset and interaction modulation: mood is shaped by time of day, recent interactions, personality, other birds, and weather, then reset in a way that is "not a hard cut."

- Personality vector with monotonic-toward-expressive drift: the rationale is slow expressive change without regression, calibrated so regular visits become measurable after about a week and perceptible after about three weeks.

- Return-greeting on tab open: NOT RECOVERABLE FROM PLAN

- Presence accounting: presence pings supply the `presence_seconds` input for drift while requiring visibility, focus, recent pointer/key activity, and a running interval so the server "cannot invent presence pings."

- Listen-in interaction: the plan frames this as "listening, not switching"; ambient birds decay gradually, the audio mix is the primary feedback, and hard cuts are avoided as jarring.

- Offer interaction: offers feed mood transitions and trait drift, with `offer_accepted` nudging content and curious and curiosity weighted by offer-accept events.

- Settle interaction: NOT RECOVERABLE FROM PLAN

- Field notebook: the notebook exists to carry "naturalist prose" at a "sparse cadence"; the risk section says too many entries dilute the prose and too few make the feature feel broken.

- Day/night cycle keyed to user local timezone: the cycle supplies local morning, dusk, and night inputs to mood transitions and day/night rendering.

- Rare ambient weather: weather affects mood and vocal behavior, with rain nudging birds toward drowsy/content and dampening vocal frequency; unusual weather can also make notebook events noteworthy.

- Ambient micro-motion: motion controllers, leaves, and feathers support the aviary feeling alive at idle, with complexity reduced if frame budget is exceeded.

- Visit invitation: the plan's why is quiet sharing without social-network surfaces: invites are opt-in, read-only, revocable, email-keyed, not co-presence, and off by default.

- Screen-reader narration: narration gives a screen-reader user the same naturalist register as the notebook, generated from the current snapshot on a slow cadence.

- Reduced-motion mode: reduced motion preserves semantic content while replacing motion with cross-fades and removing decorative particles.

- Call captioning: captions are generated at synthesis time so they "match what actually played," and they turn on automatically if audio is unavailable.

- WCAG AA contrast on user-copy text: the plan requires user-copy text to remain readable against both bright and dark aviary backgrounds.

- Full keyboard navigation: keyboard navigation makes the top bar, birds, listen-in, offer panel, and escape paths available without pointer input.

- Account export and soft/hard deletion: the plan provides export and a 30-day soft deletion/recovery window so account data can be exported, deleted, or recovered before hard deletion.

- Aggregate operational telemetry only: this preserves the boundary that no per-bird state or per-account interaction detail enters telemetry.

### Architecture

- Stateless API service: the service is "stateless, horizontally scalable" so auth, snapshot delivery, event ingestion, visit management, notebook reads, and account CRUD can scale around the canonical store.

- Simulation service: the simulation service centralizes tick running, drift computation, mood transitions, notebook writing, and weather generation as server-owned state work.

- Notification service for email only: it exists to deliver magic links, account export links, and visit invites; no other rationale is articulated.

- Canonical data store: the store holds account, session, bird, personality, mood, event, notebook, invite, visit, and weather records so the product has one source of canonical state.

- Append-only interaction event log: append-only events let clients submit interactions without mutating state directly, let the tick process ordered inputs, and support conflict-free sync.

- Snapshot cache: the tick writes snapshots to Redis or similar for "fast client reads."

- First snapshot embedded in HTML: this is explicitly for the "<500ms first-bird budget" and lets the first bird render before the full bundle parses.

- CDN-cached edge delivery for cold load: the plan ties this to cold-load snapshot delivery and the first-frame performance path.

### Data model

- Encrypted account email stored once and never used as a foreign key: this reflects the plan's PII posture for the account email.

- Account deletion timestamps: `deletion_requested_at` and `deletion_hard_at` support the 30-day soft deletion and recovery flow.

- Visit notifications default false: the plan keeps visit notifications as a quiet opt-in account setting rather than a default prompt.

- Session `device_hint`: the why is "browser UA, for session list display."

- Magic link 15-minute expiry and consumed timestamp: NOT RECOVERABLE FROM PLAN

- Aviary `created_at`: this timestamp is "used for age-gated bird availability."

- Stable bird id for life of account: NOT RECOVERABLE FROM PLAN

- Personality seed values from species-typical range with jitter: this gives adopted birds species-shaped variation before simulation drift begins.

- Mood `expires_at`: this carries the "next daily reset window."

- Interaction event `bird_id` nullable for aviary-level events: this lets settle and weather be recorded at the aviary level rather than falsely attaching them to a bird.

- Interaction events with "No updates. Only inserts.": this preserves the append-only log consumed by the simulation tick.

- Notebook entry prose as lowercase, present-tense naturalist prose: this preserves the notebook voice at the data boundary.

- Visit invite plaintext visitor email: the plan explains this as "the invitee's email, not ours; different PII posture."

- Visit log duration on session close: NOT RECOVERABLE FROM PLAN

### API Surface

- Auth request returns 202 always and is rate-limited per email: NOT RECOVERABLE FROM PLAN

- Session list and session revoke endpoints: the session list exposes `device_hint` and `last_seen_at`, giving users a way to see and revoke sessions.

- Aviary snapshot endpoint: snapshots are small representations of settled state, weather, day fraction, and bird display state so clients can render without owning simulation.

- Interaction event endpoint: clients batch presence pings and submit events so the server can compute deltas from the append-only log.

- Notebook endpoint: pagination and most-recent-first ordering support sparse, growing naturalist entries without limiting how far back the client can scroll.

- Visit invite, revoke, invite list, log, and token endpoints: these implement opt-in, revocable, read-only visiting, with expired or revoked invites returning a matter-of-fact 410.

- Account endpoints for notifications, export, deletion, recovery, and email change: these support quiet visit notification settings, account export, a 30-day recovery path, and verified email changes.

- Bird rename endpoint only: limiting client mutation to rename only preserves the rule that clients do not write personality or mood state.

### Simulation Engine Design

- Tick runner cadence around 60 seconds: the tick is the server's regular place to read new events, update mood and drift, recompute positions, advance weather, write notebook entries, cache snapshots, and persist canonical state.

- Idempotent tick window with `last_processed_event_id`: this prevents drift deltas from being double-applied after restart.

- Drift low-pass filter and calibration constants: the low-pass filter slows interaction inputs into gradual drift, with constants calibrated before launch rather than guessed.

- Monotonic drift clamp: `max(0, delta)` makes traits only move toward expressiveness and never reverse.

- Drift calibration scenarios: synthetic seven-day and thirty-day presence scenarios validate that changes are neither too fast nor too slow.

- Trait-specific input weights: NOT RECOVERABLE FROM PLAN

- Probabilistic mood transitions: the plan says the same inputs should not always produce the same next state; weights are shaped by personality, time, events, nearby moods, and weather.

- Randomized daily reset window: the reset recomputes mood from scratch while keeping the prior mood as a starting point, so it is daily-ish but "not a hard cut."

- Runtime call grammar motifs and variation: motif selection, pitch contour, random variation, and onset jitter produce call events shaped by vocal frequency and mood.

- Client-held grammar state with server `call_timing_offset`: grammar state stays presentation-only, and the offset lets the client "join mid-call rather than starting fresh."

- Bird-to-bird chorus events: high `social_warmth` and content/curious mood can produce response calls, giving social behavior a simulation expression.

### Sync Model

- Single canonical record for personality and mood: this is the reason conflicts do not need resolution.

- Client state consumption on tab open: fetch snapshot, render immediately, send `return`, and subscribe to SSE so the client joins current state instead of reconstructing it locally.

- SSE lightweight diff events: diff events update mood, perch, weather, and notebook changes without resending full snapshots for every change.

- Hidden/visible lifecycle handling: disconnecting when hidden and fetching a fresh snapshot when visible handles laptop-lid-close gaps.

- Presence-time additivity across devices: simultaneous pings are both recorded and summed because presence-time is additive.

### Frontend Rendering Pipeline

- Canvas/WebGL scene: the plan chooses it for "procedural motion, parallax layers, and per-bird rendering at 60fps."

- CSS top bar, overlays, notebook, and settings panels: NOT RECOVERABLE FROM PLAN

- Aggressive code splitting: this keeps the aviary core synchronous while settings, account, visit-invite, and notebook panels load on demand for the bundle budget.

- Layered scene composition: NOT RECOVERABLE FROM PLAN

- Inline snapshot, critical CSS, preload, and lightweight init script: these exist to place birds in current poses before the full bundle executes and meet first-frame timing.

- CSS-only quiet-field fallback: it renders before JS and is also used on reconnect to avoid showing a stale snapshot.

- No entry animation and no fade-from-black: the rationale is that "the client joins the aviary mid-scene."

- Mood-driven idle micro-motion: bird behaviors such as preening, scanning, head-tilts, slow blink, and alert posture make mood visible in the scene.

- Spring physics for pose transitions: the plan says this gives an "organic feel."

- Reduced-motion rendering: cross-fades, particle removal, and slower day/night color transitions reduce motion while keeping captions, narration, and semantic content.

- Perch transition animations: normal mode uses a short flight arc; reduced-motion cross-fades between poses so perch changes remain legible without forced motion.

- Listen-in visual treatment: the vignette and lack of harsh ring keep visual focus subtle because the audio mix change is the primary feedback.

### Audio Pipeline

- Per-bird gain nodes and listen-in ramps: these let the focused bird rise while others fall gradually, preserving the "listening, not switching" feel.

- Master gain normalization or compressor: this prevents loudness spikes when two birds call at once.

- Motif descriptor library: descriptors allow procedural synthesis with pitch, contour, formant, duration, and variation rather than stored audio files.

- Client grammar FSM shaped by vocal frequency and mood: this sequences motifs with personality-shaped gaps and mood-shaped selection weights.

- WebAudio fallback to silence with captions on: this preserves the no-recorded-audio boundary and keeps the experience accessible when `AudioContext` is unavailable.

- Caption generation from motif and variation: the rationale is one caption per actual synthesized call, matching what played.

### Accessibility Surfaces

- `aria-live="polite"` narration cadence: slow polite updates describe the scene without interrupting current screen-reader activity.

- Prompt narration on user-initiated events: offer accepted, settle, and return greeting updates are prompt so user actions receive timely feedback.

- Shared narration and notebook voice: the plan wants a screen-reader user moving from aviary to notebook to hear "the same register."

- Keyboard tab order and bird navigation: this gives predictable access to top bar controls, birds, listen-in, and modal escape behavior.

- Modal focus trap in offer panel: NOT RECOVERABLE FROM PLAN

- Focus indicators with day/night-aware colors: two ring colors are used so focus passes WCAG AA against both bright midday and dark night backgrounds.

- Contrast targets for user-copy text: the plan sets WCAG AA ratios for normal and large text, while excluding non-text aviary scene imagery.

- Synchronous reduced-motion detection before first paint: this avoids a "flash of motion before detection."

### Performance Budgets and Observability

- Initial JS bundle under 2MB gzipped: bundle analysis, async panels, SVG assets, and JSON audio descriptors enforce fast loading.

- First bird visible under 500ms: embedded snapshots, inline CSS, preload, and a tiny init script exist to make the bird appear quickly in the right pose.

- 60fps idle motion on a 5-year-old laptop: frame-time monitoring reduces particle and bird-motion complexity when p95 exceeds budget.

- No memory growth over 30 minutes: heap tests, buffer reuse, virtual notebook scrolling, and a single shared `AudioContext` prevent long-session growth.

- Aggregate operational metrics: latency, first-bird-visible, audio errors, render frames, ingest errors, and magic-link failures provide operations alarms without per-bird fields.

- Synthetic monitoring from three or more geographies: this checks first-bird-visible, snapshot fetch time, and audio context initialization every five minutes.

- Real User Monitoring as anonymized session-level aggregates: this captures real client performance while preserving the no per-bird/per-account telemetry line.

### Rollout

- Launch with two birds, cap at seven, and age-based unlocks: this repeats the age-gated growth model and avoids interaction-gated expansion.

- Visit invite feature enabled but off by default: this keeps sharing available while avoiding onboarding pressure.

- Pre-launch synthetic calibration runs: these validate drift constants against seven-day and thirty-day scenarios before users encounter them.

- Closed beta around 100 accounts: this monitors tick latency, presence volume, first-bird-visible RUM, and notebook cadence under invited use.

- Soft open: removing the invite gate adds monitoring for audio-context error rate and memory growth under broader use.

- v1 GA: the launch condition is that "all perf budgets hold under real traffic."

- Day-one instrumentation: metrics start with the first account so latency, first-bird-visible, audio initialization, aggregate presence volume, notebook generation rate, and invite issuance are visible immediately.

### Risks

- Server-side `DRIFT_ALPHA` config: this allows drift calibration recovery without a client deploy.

- Fresh snapshot on reconnect rather than stale in-memory render: this prevents a visible jump after long gaps such as laptop lid close.

- Audio design and listening tests: procedural calls need human-ear tuning to avoid "MIDI" or "video-game" uncanniness that breaks the aliveness promise.

- Shared `VoiceGenerator` module: shared code is the mitigation for narration, caption, and notebook voice drift.

- Presence pings only when all conditions are met: this makes power-saving modes under-report rather than over-report presence, which is acceptable because monotonic drift is robust to under-reporting.

- Visit revocation SSE behavior: sending `{ type: "visit_revoked" }` and terminating the stream is the target behavior so revocation is matter-of-fact and immediate after receipt.

- Probabilistic notebook-entry gating: a base rate near one entry per three active-aviary-days plus noteworthy-event multipliers prevents both prose dilution and a broken-feeling empty notebook.
