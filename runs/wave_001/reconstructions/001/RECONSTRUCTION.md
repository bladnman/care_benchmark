## System-level intent

- Server-owned truth, client-owned presentation. The plan repeats a strict boundary: the server computes "what is true"; the client computes "what it looks and sounds like right now." This shows up in the architecture, sync model, event API, and simulation design: "Clients never tick," "Never owns canonical state," "Clients never submit absolute state," and "Simulation service - the only writer of personality/mood state."

- Single canonical state with additive, ordered changes. The plan's sync philosophy is "Single canonical state; server-only writes; additive deltas. There is nothing to merge, ever." It rejects "last-write-wins" and protects state through ordered event-log consumption, server-authored drift deltas, and DB/API constraints.

- A quiet relationship product, not a game, feed, pet-survival loop, or social network. The scope rules ban "gamification of any flavor," "Tamagotchi mechanics," and "social-network surfaces." This intent also appears in drift design ("monotonic toward expressive"), notebook sparsity ("so very active users don't get a feed"), and risks around "scope creep toward announcements."

- Privacy by architectural absence. The plan wants privacy enforced "at the pipeline level, not the policy level." Evidence appears in the Simulation DB rule that it is "never read by analytics or telemetry pipelines," telemetry that is "aggregate-only," email stored in "exactly one column, encrypted," and the principle "We don't compute stats we refuse to surface."

- Naturalist ambient voice split from matter-of-fact system voice. The plan assigns "naturalist" prose to notebook, captions, and screen-reader narration, while auth/session/server errors and settings use "matter-of-fact voice." It explicitly says the split is honored: "settings/error surfaces are matter-of-fact; everything else naturalist."

- Accessibility is a designed v1 surface, not a fallback. Reduced motion is called "a separate render path, not a fallback" and "not a later fix." Narration, captions, keyboard, focus visibility, and contrast all appear as first-class surfaces with review sign-off and CI checks.

- Aliveness should be persistent, calibrated, and low-pressure. The tick runs "whether or not any client is connected"; moods persist and evolve while no client is connected; the first frame is "already moving"; day/night, rare weather, greetings, calls, and notebook entries create aliveness. The calibration risk frames the product between "Tamagotchi" and "screensaver."

## Per-feature whys

### Scope

- Browser-only product: NOT RECOVERABLE FROM PLAN

- Single-user accounts with one canonical aviary per account: supports the "single canonical state" model, so "both devices read the same record" and a second device sees "the same birds in the same moods."

- Email magic-link auth: NOT RECOVERABLE FROM PLAN

- Two starter birds at adoption: NOT RECOVERABLE FROM PLAN

- Species chosen by the system from a pool of about six, with user-assigned names: NOT RECOVERABLE FROM PLAN

- Cap of seven birds unlocked by aviary age: the plan says the cap is "enforced engine-side" and later ties "the cap-of-7 invariant" to recognizability testing, but gives no product rationale for seven or for age unlocking. NOT RECOVERABLE FROM PLAN

- Server-side simulation tick: exists so the server owns "all canonical state" and mood/drift continue even when "no client is connected."

- Client rendering of snapshots with interpolation: follows the boundary where the server says what is true and the client decides "what it looks and sounds like right now," using interpolation for "motion continuity."

- Procedural WebAudio call synthesis: exists because "No recorded audio anywhere" is a hard rule; the audio section also says bundle budget and chorus correctness "both depend on this."

- Chorus mixing: preserves an "ambient mix" while listen-in rebalances attention; "Hard cuts are banned" because it is "a re-balance, not soloing."

- Return-greeting interaction: uses absence length, boldness, mood, and procedural variation so greeting kind comes from "absence-length band" and avoids "becoming canned."

- Listen-in interaction: gives a focused bird more gain while taking others down "never to silence," preserving the aviary rather than isolating a solo.

- Offer interaction with seed, song fragment, or still pool: NOT RECOVERABLE FROM PLAN

- Settle interaction with 5s undo: NOT RECOVERABLE FROM PLAN

- Field notebook: turns notable deltas into "naturalist, lowercase" prose that is "specific but non-repeating," with sparsity gates and weekly caps so it does not become "a feed."

- Presence accounting with three-signal conjunction: prevents inflated drift from weak signals; the risk section names the failure case as "tab open" silently inflating population drift.

- Day/night cycle anchored to user local time: drives time-of-day-appropriate moods, lighting phase, and palette, with client timezone used for "local-time anchoring."

- Rare ambient weather: provides short, small, time-boxed ambient events such as rain and wind that can affect calls/moods and create "weather moments" for the notebook.

- Visit invitations with per-invite opt-in: supports host control over access in a product that otherwise bans social-network surfaces.

- Read-only ambient visits: ensure visitor sessions "set no presence events," "write no events," and expose "no interaction affordances," preserving the host's canonical aviary.

- Revocable visits: give the host control; active visitor sessions terminate at the next snapshot pull with a "visit no longer available" matter-of-fact surface.

- 30-day visit expiry: NOT RECOVERABLE FROM PLAN

- Silent visit log: the data-model note gives the rationale as "transparency to host," while the broader plan avoids visit-frequency surfaces.

- Visit notifications optional and off by default: keeps visits from becoming an announcement or engagement surface, in line with "no visit-frequency surfaces" and "deliberately not measured" engagement counters.

- Naturalist screen-reader narration: gives non-visual access to the same snapshot state as the visuals, written as "observation, not state transition."

- Designed reduced-motion mode: ships as v1 accessibility, "not a fallback" and "not a later fix."

- Procedural call captions: because text is derived from the same motif parameters as audio, "guaranteeing caption matches what played."

- WCAG AA contrast on chrome: supports accessibility and is CI-reviewed against "all chrome copy, captions, and any visually displayed narration."

- Full keyboard navigation: makes top bar, scene focus, listen-in, offer flow, and settle "fully keyboard-operable."

- Account export as JSON snapshot emailed as download link: NOT RECOVERABLE FROM PLAN

- Soft-delete for 30 days then hard-delete: NOT RECOVERABLE FROM PLAN

### Architecture

- Static SPA web client served from CDN: owns rendering, audio synthesis, presence detection, and interaction capture while "Never owns canonical state"; CDN and embedded snapshot support the first-bird budget.

- Stateless HTTP/JSON API service: NOT RECOVERABLE FROM PLAN

- Simulation service as only writer of personality and mood: protects ordered drift and prevents client-side canonical writes.

- Simulation DB separated from analytics and telemetry: enforces the privacy rule "at the pipeline level, not the policy level."

- Account DB with encrypted email: prevents "Email-as-identifier leak" by storing email once, encrypted.

- Telemetry store with aggregate-only metrics and hashed bucket only for rate-limiting: allows operational metrics without per-account interaction history.

- Synthetic account UUID as only account identifier: keeps email out of "DBs, logs, partition keys, and telemetry."

- Tick runs whether or not a client is connected: keeps the aviary's moods and simulation alive outside active sessions.

- UTC timestamps with client timezone offset: supports "local-time anchoring" for lighting, day/night, and time-of-day moods.

- Personality vectors server-only and never numerically exposed: honors the hard rule that vectors are "never exposed numerically on any surface, at any tier, ever."

- Visit-log asymmetry: shows who visited for "transparency to host," while no telemetry pipeline aggregates visit or interaction data across accounts.

### API surface

- Magic-link issuance and verification endpoints: NOT RECOVERABLE FROM PLAN

- Session listing and revocation: NOT RECOVERABLE FROM PLAN

- Snapshot endpoint with embedded initial HTML snapshot: keeps payloads small and supports "<500ms first bird."

- Event append endpoint: keeps writes one-way, lets the tick consume events, and ensures "Clients never submit absolute state."

- Notebook pagination with oldest-to-newest scrollback and no archiving: NOT RECOVERABLE FROM PLAN

- Bird adoption endpoint only when an age-gated offer is active: NOT RECOVERABLE FROM PLAN

- Bird rename endpoint with no other mutable field client-writable: protects canonical state by making personality and other fields unwritable by the client.

- Visit invite endpoint: NOT RECOVERABLE FROM PLAN

- Outstanding invites and visit log endpoint: supports the host transparency described by the visit-log asymmetry.

- Visit revoke endpoint: gives a matter-of-fact revocation surface and ends active visitor access at next snapshot pull.

- Visitor snapshot endpoint: keeps visits ambient and read-only by returning the host snapshot shape minus host-only fields and writing no events.

- Account settings endpoint: NOT RECOVERABLE FROM PLAN

- Email change with verify-new-before-commit: NOT RECOVERABLE FROM PLAN

- Account export endpoint: NOT RECOVERABLE FROM PLAN

- Delete and recover endpoints: support the 30-day soft-delete window; the plan gives no deeper rationale for the feature.

### Simulation engine design

- Idempotent ordered tick: consumes unconsumed event-log entries in server timestamp order so drift and moods are deterministic and not merged.

- Presence-ping duration summing: feeds presence-time into drift only when the three presence signals hold.

- Shared presence time with listen-in weighting per bird: lets general presence affect the aviary while focused listening can shape a specific bird.

- Low-pass drift calibration: targets change that instruments detect at "about 1 week" and users notice at "about 3 weeks."

- Monotonic drift toward expressive: avoids decaying meters; "absence of input leaves traits unchanged" and plumage saturation "never down."

- Monotonicity clamp and regression test: called out because "a symmetric-drift implementation is the single most likely engine bug."

- Mood transitions from interactions, time of day, ambient events, and personality: make mood persistent, contextual, and personality-modulated.

- Weighted-transition Markov-ish mood table with daily soft reset: creates a "daily-ish cadence" and a time-of-day-appropriate baseline.

- Perch and position selection from mood plus boldness: turns internal state into visible placement and motion.

- Chorus events from overlapping high-vocal-frequency call windows: creates bird-to-bird emergent behavior.

- Wary spread: lets one bird's alarm call affect another bird probabilistically.

- Rare weather scheduler: keeps rain and wind short, occasional, and with "effects small and time-boxed."

- Greeting decision on session_open: makes greetings sensitive to absence length, boldness, and mood.

- Staggered multiple greeters: prevents simultaneous greetings by using randomized offsets.

- Notebook generator from notable deltas: records "first-greeter changes, unusual quiet stretches, weather moments, offer reactions" rather than user-behavior stats.

- Notebook phrase bank seeded per account: makes entries "specific but non-repeating."

- Notebook sparsity gate and hard weekly cap: prevents very active users from getting "a feed."

- New-bird offers on age-based scheduler: NOT RECOVERABLE FROM PLAN

- Species motif library: gives each species pitch contours, syllables, and trill patterns for parametric calls.

- Per-bird call_grammar_seed: fixes a "recognizable signature" for each bird.

- Runtime call variation from vocal_frequency and mood: avoids repetition while keeping the signature recognizable.

- Server-scheduled call events with client synthesis: keeps canonical timing in snapshots and actual sound generation client-side.

- Caption generation from the same motif parameters: prevents "procedural-caption mismatch."

### Sync model

- Snapshot pulls on load, visibility return, render-frame gaps, and low-frequency keepalive: keeps the client reconciled after load, tab visibility changes, suspend gaps, and visible sessions.

- Mood cross-fades into mood-shaped idle behavior: keeps mood changes visually continuous.

- Matter-of-fact conflict surfaces: keeps errors in the plan's system voice.

- Multi-device reading the same record: makes "multi-device free" because devices see the same birds in the same moods.

- Visitor sessions as read-only with zero event writes: preserves the host's canonical state and allows revocation at next pull.

### Frontend rendering pipeline

- Canvas 2D, with WebGL only if profiling demands: chosen as a defensible stack for the scene under performance constraints.

- DOM for top bar, notebook, settings, captions, and narration live-region: keeps interface and accessibility surfaces in DOM while the scene renders separately.

- Procedurally assembled layered bird sprites: enables parametric plumage from plumage_seed and saturation while keeping birds renderable as parts.

- First frame with embedded snapshot and birds mid-pose: supports "No spinner, no fade-from-static, no entry animation" and the first-frame-already-moving intent.

- Quiet field slow-connection state: avoids a spinner and keeps the experience ambient with "soft sky, faint motion cues."

- Three perch zones and parallax planes: gives the scene depth and structured places for front/middle/back bird positions.

- Day/night palette keyed to user local time: makes local-time lighting visible with smooth gradient phases.

- Weather overlay when active: reflects active weather in the visual surface.

- Settle lighting transition over several seconds: NOT RECOVERABLE FROM PLAN

- Idle micro-motion selected by mood and parameterized by personality: creates aliveness through preen, scan, head-tilt, and weight-shuffle loops.

- Rendering halt when tab hidden: saves client rendering work while "simulation continues server-side."

- Client-only leaf and feather drift: adds ambient ornament without simulation state.

- Responsive scene with never-crop-bird invariant: protects bird visibility across viewport scaling.

- Top bar fade to near-transparent and no other chrome: keeps the scene visually quiet and free of in-scene interface.

- Reduced-motion render path: preserves calls, captions, notebook, and drift while replacing motion with still-pose and perch-transition cross-fades.

- Empty aviary quiet field and first bird soft fly-in: NOT RECOVERABLE FROM PLAN

### Audio pipeline

- Oscillator/filter/noise-envelope synthesis from motif fragments: implements the WebAudio-only call grammar.

- Buffer reuse and freed allocations: supports the "Zero memory growth" and memory-budget test.

- Per-bird gain nodes into a master bus: enables chorus mixing and listen-in rebalancing.

- Listen-in ramps over about one to two seconds: avoids hard cuts and keeps the change a re-balance.

- Captions on enablement or WebAudio failure: ensures silence has captions by default when audio fails.

- Caption placement near the calling bird and fading with the call: connects the textual description to the visual caller and the audio event.

- Night behavior with quiet birds and a nightjar-like late caller: keeps night from becoming a "silent-by-default dead state."

- Recorded-audio ban: preserves bundle budget and chorus correctness.

### Accessibility surfaces

- Narration live-region from the same snapshot state as visuals: keeps screen-reader narration aligned with the scene.

- Narration cadence and rate-limited queue: ensures the system does not "overwhelm the screen reader."

- Priority bump for user-initiated events: lets greeting, offer reaction, and settle surface promptly while remaining observation.

- Reduced motion as designed surface: prevents reduced motion from becoming a "stripped fallback."

- Keyboard path through top bar and scene: makes birds, listen-in, offer, and settle reachable without pointer input.

- Visible high-contrast focus indicators: keeps focus visible against "both bright and dim scene states."

- Contrast minimum on chrome copy, captions, and visually displayed narration: makes displayed text meet WCAG AA.

- No copy in the scene itself: keeps the scene from carrying text that would need contrast treatment.

- Voice split for settings/errors versus everything else: keeps system surfaces matter-of-fact and ambient surfaces naturalist.

### Performance budgets and observability

- Initial JS bundle under 2MB gzipped with code-splitting: protects first paint and keeps settings/visit/notebook surfaces out of the initial path.

- Time-to-first-bird under 500ms: achieved through "embedded snapshot + CDN edge + render path that doesn't await non-critical assets."

- 60fps idle motion for 30 minutes: ensures ambient motion is sustainable on "a 5-year-old mid-range laptop."

- Zero memory growth over 30 minutes: prevents long-session degradation and is automated in CI.

- Synthetic fleet checks and RUM: provide aggregate operational visibility into load, first-bird, frame timing, audio-context errors, and tick latency.

- Simulation tick p99 alarm at 5s: protects server tick latency.

- Never collecting per-bird state, per-account interaction history, or relationship reconstruction data: preserves the privacy boundary.

- Deliberately not measuring visit frequency, streaks, or engagement counters: ensures "the architectural absence makes reappearance harder."

### Rollout

- Phase A engine plus skeleton: starts with simulation service, event log, snapshot API, magic-link auth, and minimal static-pose renderer for internal-only validation.

- Phase B aliveness: adds procedural audio, chorus, idle micro-motion, day/night, greetings, and first-frame-already-moving.

- Phase C relationship: calibrates drift, notebook, offers, settle, and listen-in around the "1-week-instrument / 3-week-visible targets."

- Phase D access plus social: adds narration, reduced motion, captions, keyboard, and visit invitations.

- Limited beta before GA: used for calibration tuning of presence activity window, tick cadence, offer cooldown, and notebook sparsity.

### Risks and mitigations

- Drift calibration harness and dashboards: mitigate "Too fast -> Tamagotchi; too slow -> screensaver" and silent failures.

- Three-signal presence module and tests: mitigate corruption from laxer signals like "tab open."

- API schema, code-review rule, and DB constraint for personality fields: prevent convenient client writes from eroding additive-delta discipline.

- Audio jitter, motif combination space, listening tests, and recognizability testing: mitigate ear-detected repetition, phase cancellation, and loss of individual bird identity.

- Single parameter object for audio and captions: ensures caption text describes "what actually played."

- Accessibility review sign-off and CI: prevent reduced-motion or narration from becoming a "stripped fallback."

- Procedural greeting variation and no-repeat test: prevent greetings from becoming canned.

- Notebook sparsity gate, weekly cap, and prose-quality samples: prevent notebook dilution into a feed.

- PR checklist naming banned surfaces: prevents "toasts, streaks, welcome text" and other scope creep toward announcements.

- Email schema and lint rule: prevent email appearing in log statements and partition keys.
