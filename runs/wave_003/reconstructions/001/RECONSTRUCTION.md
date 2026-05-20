## System-level intent

- Pocket Aviary is meant to be an observer-focused naturalist experience, not a game or caretaking loop. This shows up in the V1 scope as "Naturalist Experience" with "no quests, scores, or gamified mechanics," and in the non-goals excluding "streaks, XP, level counters, calendar green dots, stats dashboards, or badges." It also appears in the "Field Notebook" and screen-reader language as "naturalist prose rather than raw status lines."

- The product avoids punishment, distress, and obligation. The non-goals say birds "cannot die, fall ill, or show distress" and that neglect results only in "ambient quietness." The drift model is "monotonic" and the close-tab path says "No user penalties or warnings occur on raw tab-close."

- The simulation is server-canonical, while clients render, synthesize audio, and submit events. The architecture says "the server acts as the single source of truth" and the client is a "pure rendering and audio synthesis engine." The rendering boundary repeats that "The client never runs simulation physics or computes personality/mood," and sync says clients "NEVER send absolute values."

- Change should feel gradual, bounded, and calibrated rather than abrupt or farmable. The plan uses "monotonic drift," "soft saturation," a "per-tick capping factor," one-minute ticks, cooldown-bounded offers, age-based availability thresholds, and a calibration target where one week produces a "measurable instrument change" and three weeks a "visible user difference."

- Sharing is private, revocable, and non-invasive. The scope defines visits as "read-only guest sharing via private email invitation links," expiring after 30 days, with "No co-presence or guest-driven drift." The guest state endpoint rejects event submissions, and invitations have `expires_at` and `revoked_at`.

- Accessibility is part of the core product surface, not a later accommodation. The scope includes "running screen-reader narration, reduced-motion pose cross-fades, call captioning, keyboard navigation," and section 9 says "Accessibility is treated as a core product feature." The accessibility-drift mitigation says accessibility changes must merge "inside the same pull request as the visual update."

- The product voice is quiet, matter-of-fact, and descriptive. This appears in the auth response as "Matter-of-fact tone on error," in offers appearing as "gentle, non-obtrusive events," and in narration that says what is happening in the aviary rather than exposing "raw status lines."

- Privacy boundaries are systemic. The data model isolates and encrypts email "To prevent PII leakage into logs, caches, and telemetry," and observability says metrics "must never contain user email hashes, names of birds, or raw text observations."

## Per-feature whys

### Scope & Non-Goals

- **Horizontal Aviary View**: NOT RECOVERABLE FROM PLAN

- **Naturalist Experience**: The plan's rationale is that the user should be an observer, not a player. It says "Observer-focused interaction" and explicitly excludes "quests, scores, or gamified mechanics."

- **Return-Greeting**: The recoverable rationale is that greetings should reflect the living state of the aviary rather than be uniform. The plan says they are "randomly-staggered" and "mood-and-boldness-influenced."

- **Listen-in**: The rationale is audio focus without a hard cut. The scope calls it "Audio focus of a single bird using a slow mix ramp," and the audio section implements a 1.5 second cross-fade where the focused bird is centered and other birds fade down.

- **Offers**: The recoverable rationale is bounded, gentle interaction rather than unlimited input. The scope calls them "Cooldown-bounded gestures," and the simulation model uses offer events as mood inputs such as a seed offer increasing transition odds to `content`.

- **Settle**: The rationale is to provide an intentional close to a session. The scope calls it "an opt-in evening lighting shift to close a session with an undo window," and session-end handling distinguishes it from raw tab-close, which creates "No user penalties or warnings."

- **Field Notebook**: The rationale is to document aviary events in the product's naturalist voice. The scope says entries are "sparsely generated naturalist prose entries documenting events," and the tick loop generates entries only "If conditions met."

- **Server-Side Simulation**: The rationale is canonical, persistent state advancement. The scope says a background tick engine advances bird mood, applies drift, and persists "the canonical aviary state."

- **Accounts & Auth**: The rationale is passwordless sign-in with replay limits. The scope specifies "magic-link sign-in with 15-minute link expiration and automatic consumption invalidation," and the risk mitigation says tokens are "one-time-use" and deleted instantly.

- **Multi-Device Sync**: The rationale is consistent state across browser sessions. The scope says "Server-canonical state synchronization," and the sync section says conflicts are "structurally avoided by centralizing logic on the server."

- **Visits**: The rationale is limited sharing without changing the host's aviary. The scope says "Revocable, read-only guest sharing" with "No co-presence or guest-driven drift," and the guest state endpoint says visitor event submissions are rejected.

- **Accessibility & Performance**: The rationale is to make the experience usable and responsive as a baseline. The scope names narration, reduced motion, captions, keyboard navigation, a "<2MB bundle budget," and "<500ms time-to-first-bird."

- **Native Applications**: NOT RECOVERABLE FROM PLAN

- **Custodial tamagotchi-like mechanics**: The rationale is to avoid distress and obligation. The plan says birds "cannot die, fall ill, or show distress," and neglect becomes "ambient quietness" with "monotonic positive drift towards expressiveness only."

- **Gamification**: The rationale is to preserve the naturalist, observer-focused character. The plan excludes "streaks, XP, level counters, calendar green dots, stats dashboards, or badges" after defining the product as having "no quests, scores, or gamified mechanics."

- **Social Networks**: NOT RECOVERABLE FROM PLAN

- **Monetization/Billing**: NOT RECOVERABLE FROM PLAN

- **Customization**: NOT RECOVERABLE FROM PLAN

### Architecture & Service Shape

- **Decoupled client-server architecture**: The rationale is a clean split between canonical simulation and presentation. The server is the "single source of truth," while the client is a "pure rendering and audio synthesis engine."

- **Append-Only Event Log**: The rationale is to submit interactions as events that can be consumed by the simulation tick. The diagram routes "Submit Events" into the "Append-Only Event Log," and the tick reads events before updating state.

- **REST API Server**: The plan says it handles authentication, profile management, magic links, event ingestion, and invitations, but gives no separate product rationale beyond the component split. NOT RECOVERABLE FROM PLAN

- **Simulation Engine worker**: The rationale is to periodically consume the event log and recalculate state. It ticks every 60 seconds, "recalculates personality vectors and mood transitions," updates the DB, and refreshes cache.

- **PostgreSQL**: The rationale is transactional persistence. The storage layer calls it the "Primary transactional database" for user records, bird configurations, notebook entries, and invitations.

- **Redis**: The rationale is fast state delivery. The storage layer says Redis "Caches the pre-computed, compressed JSON state snapshot for each active aviary."

- **Frontend SPA**: The rationale is to keep sensory work on the client. The audio pipeline "Synthesizes procedural bird calls" and the render engine draws "three parallax layers at 60fps."

- **Rendering Boundary**: The rationale is to prevent client-side divergence from canonical simulation. The client reads a "sequence-numbered snapshot" and "never runs simulation physics or computes personality/mood."

- **Hidden document behavior**: The rationale is resource saving on the client without stopping the aviary simulation. When hidden, the client pauses RAF and WebAudio, but "does not stop the server from continuing its 60-second tick cycle."

### Data Model

- **PII Isolation and Encryption**: The rationale is explicit: "To prevent PII leakage into logs, caches, and telemetry." Email is encrypted, lookup uses an HMAC hash, and internal references use synthetic UUIDs.

- **Synthetic `account_id` foreign keys**: The rationale is to keep internal tables away from email. The plan says all internal tables reference "a synthetic UUIDv4 `account_id`."

- **Account settings JSON**: NOT RECOVERABLE FROM PLAN

- **Bird personality vector, mood state, and layout positional state**: NOT RECOVERABLE FROM PLAN

- **Interaction events table**: The rationale is to preserve session interactions for server processing. It is labeled "Append-Only Interaction Event Log" and stores presence, offers, listen-in, settle, targets, duration, and timestamps.

- **Notebook entries table**: The rationale is to persist the field notebook surface. It stores `entry_text` with account and creation time, matching the sparse naturalist prose entries.

- **Visit invitations table**: The rationale is private, revocable guest access. It stores visitor email hash, token hash, expiration, and revocation time.

### API Surface

- **`POST /api/auth/magic-link`**: The rationale is account-safe, matter-of-fact login initiation. The response says "If the email matches an account, we sent a link," which avoids exposing a direct account-existence result.

- **`POST /api/auth/verify`**: The rationale is exchanging a magic-link token for a session token, with later mitigation that token use is one-time and prevents replay attacks.

- **`GET /api/aviary/state`**: The rationale is delivering the cached state snapshot that the rendering boundary depends on. It returns time, weather, settled state, birds, mood, perch, call seed, and animation pose.

- **`POST /api/aviary/events`**: The rationale is batched event ingestion instead of client-side state setting. It accepts a session id and events such as presence and listen-in, returning a processed count.

- **`POST /api/visits/invite`**: The rationale is creating private guest access by visitor email. It generates a sharing link for the guest email.

- **`POST /api/visits/revoke`**: The rationale is immediate revocation of an active invitation. It takes an invite id and returns success.

- **`GET /api/visits/:token/state`**: The rationale is read-only viewing. It returns the same visual snapshot as aviary state, while "Event submissions via the visitor session are rejected."

### Simulation Engine Design

- **Tick Loop**: The rationale is periodic server-side advancement of active aviaries. Each tick retrieves events, computes presence, applies monotonic drift, transitions mood, possibly generates notebook prose, writes PostgreSQL, and flushes Redis cache.

- **Active account filtering**: The rationale is to process accounts that are recently active or still need transition checks. The plan limits the tick to accounts with presence in the last 15 minutes or "those undergoing transition checks."

- **Drift Function (Low-Pass Filter)**: The rationale is gradual, monotonic, bounded personality change. Values "never decrease," per-tick caps prevent saturation, and `(1 - P_i(t-1))` gives "smooth asymptotic growth."

- **Calibration Target Values**: The rationale is to make repeated visits perceptible on a designed timeline. The plan targets one week of daily 15-minute visits for a "measurable instrument change" and three weeks for a "visible user difference."

- **Listen-in drift multiplier**: The rationale is that focused listening affects the focused bird more strongly. The plan gives a "Listen-in weight multiplier" on focused birds.

- **Mood Transition Model**: The rationale is mood as a probabilistic state affected by context. Transition probability uses time of day, weather, interaction spikes, and bird boldness.

- **Boldness dampening negative transitions**: The rationale is that personality should shape mood resilience. High boldness "dampens the transition to `wary` on rain or user absence."

- **Call-Grammar Runtime Rules**: The rationale is repeatable but state-sensitive calls. A deterministic `call_seed` makes a call "recognizable and repeatable for the same state," while mood changes pitch or duration.

### Sync & Consistency Model

- **Centralized sync**: The rationale is structural conflict avoidance. The plan says "Multi-device conflicts are structurally avoided by centralizing logic on the server."

- **Event Flow vs State Setting**: The rationale is to prevent clients from overwriting simulation results. Clients "NEVER send absolute values" and only append events for sequential server processing.

- **Concurrent Devices**: The rationale is to avoid double-counting while allowing multiple sessions. The server computes total presence as "the union of active ranges."

- **Polling or HTTP SSE**: The rationale is synchronized state pull after server updates. Both clients fetch the updated snapshot via "polling or HTTP SSE."

- **Settle vs. Close**: The rationale is to separate intentional session closure from ordinary tab exit. Settle sends an event and starts an evening fade; tab close uses `sendBeacon` best effort and times out after missing presence pings, with no penalties.

### Frontend Rendering Pipeline

- **Single canvas renderer**: The rationale is the 60fps budget. The plan says Pocket Aviary renders on a single canvas element to "satisfy the 60fps budget."

- **Visual Scene Layers**: NOT RECOVERABLE FROM PLAN

- **Idle Micro-Motion**: NOT RECOVERABLE FROM PLAN

- **Fly-in Bezier transitions**: NOT RECOVERABLE FROM PLAN

- **Reduced-Motion Mode**: The rationale is accessibility for `prefers-reduced-motion`. The plan disables frame-by-frame animations, replaces fly-ins with a "1.5-second cross-fade," disables particles, and disables parallax.

### Audio Pipeline

- **Procedural WebAudio engine**: The rationale is to synthesize naturalistic sounds. The plan says the engine uses WebAudio "to synthesize naturalistic sounds."

- **FM synthesizer architecture**: The rationale is a "clean bird call." The plan instantiates a mini FM synthesizer network of oscillator, gain, envelope, filter, panner, and master mix.

- **Ambient environmental sound**: NOT RECOVERABLE FROM PLAN

- **Chorus Mixing & Listen-In Decay**: The rationale is focused listening without muting the world. The selected bird centers, remains at gain 1.0, and other birds/background fade to 0.1 over 1.5 seconds.

- **WebAudio Fallback**: The rationale is continued access when audio is unsupported or blocked. The client silently catches initialization errors, shows mute status, enables captions, and renders call descriptions visually.

### Accessibility Surfaces

- **Screen-Reader Narration**: The rationale is naturalist prose that communicates the scene. The server compiles coordinates, weather, and time of day into narration "rather than raw status lines."

- **Narration cadence**: The rationale is steady but non-noisy updates. Narration updates every 45 seconds during idle and immediately on interactions.

- **Call Captioning**: The rationale is visual access to calls. Captions appear near birds and transition with CSS opacity "matching the WebAudio synthesis envelope."

- **Keyboard Map**: The rationale is keyboard operation of settings, notebook, offers, birds, listen-in, and cancellation.

- **High-contrast focus outlines**: The rationale is visibility across backgrounds. Outlines are custom, glowing, and visible against "day, night, and evening background textures."

### Performance Budgets & Observability

- **Initial JS Bundle Size**: The rationale is staying under the `<2MB (gzipped)` budget. The plan says this requires dynamic import code-splitting for non-essential Settings, Notebook content, and Visit management.

- **Time-to-First-Bird**: The rationale is reaching `<500ms`. The plan requires server injection of initial state, minimal synchronous head scripts, and critical asset preloading.

- **Runtime Frames**: The rationale is steady `60fps` on "5-year-old mid-range laptops."

- **Memory Footprint**: The rationale is zero memory leak policy. CI verifies by running Chrome Headless for 30 minutes and measuring heap increments.

- **Anonymized performance statistics**: The rationale is observability without content or identity leakage. Metrics include load time, first bird render, WebAudio errors, and frame-rate drops.

- **Telemetry Boundaries**: The rationale is privacy. Metrics must never include email hashes, bird names, or raw observations.

- **Simulation tick queue latency alert**: The rationale is operational responsiveness. If p99 tick latency exceeds 5 seconds, alerts fire to pager.

### Rollout & Tuning

- **Starter state adopts exactly 2 birds**: NOT RECOVERABLE FROM PLAN

- **Age-Based Ramping**: The rationale is paced expansion up to seven birds. The plan unlocks slots by account age, and new species are introduced as "gentle, non-obtrusive events in the field notebook."

- **Calibration Testing**: The rationale is validating drift and mood parameters before launch. The harness simulates "10,000 parallel accounts" and asserts bounded personality variables and drift curves matching design specs.

### Risks & Mitigations

- **Calibration Divergence mitigation**: The rationale is preventing tab-open or headless-script behavior from accelerating drift "to saturation instantly." The mitigation uses per-tick drift caps and the union model for overlapping presence.

- **Audio Uncanniness mitigation**: The rationale is avoiding sound that is "artificial, shrill, or like a broken retro synthesizer." The mitigation calibrates low-pass filter decay and uses pitch jitter for organic vocalization fluctuations.

- **Sync and Link Replay mitigation**: The rationale is preventing replayed magic links from corrupting state. Tokens are one-time-use and deleted instantly, while sync comes from central Redis state rather than client edits.

- **Accessibility Drift mitigation**: The rationale is preventing narration and caption tracks from falling out of sync with sighted features. State-to-narrator compile rules are tested against mock states, and accessibility changes merge with the visual update.
