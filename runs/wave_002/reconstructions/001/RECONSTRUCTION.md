## System-level intent

- **Quiet containment over game or social pressure.** The plan repeatedly frames Pocket Aviary as "quiet" and "Optional & Quiet", with "No Gamification", "No Tamagotchi Mechanics", and "No Social Network Features" as explicit non-goals. This intent also appears in the "quiet sky field fallback", the "Minimalist overlay", and the ban on "stock loading spinners".

- **Server-authored consistency and presence honesty.** The architecture "separates client rendering from server-side state simulation" to ensure "multi-device consistency, presence honesty, and zero client-owned personality states." This shows up again in the "Unidirectional Event Pipeline", the "Server-Authored State Engine", "canonical snapshots", and the rule that clients never submit absolute trait values.

- **Gentle, monotonic expression instead of neglect punishment.** The personality model is "monotonic toward expressive"; traits "shift UP on presence" and "NEVER decay on neglect." The non-goals reinforce this by excluding "bird death", "hunger meters", "distress states", "happiness meters", and "negative personality drift on neglect."

- **Naturalist product voice instead of dashboards.** The plan uses "Naturalist prose observation in lowercase present tense", "Naturalist field notes", and "screen-reader naturalist prose narration." It explicitly forbids "State lists" such as "Bird 1: Perch 2", so the interface voice is observational rather than numerical.

- **Procedural aliveness instead of canned assets.** The audio plan requires "Procedural WebAudio" and "species call grammars" with "zero static audio files"; the risk matrix calls out "Audio Uncanniness / Looped Sound Perception" and mitigates it with "pitch/timing perturbation." The loading experience similarly forbids "traditional loading spinners" in favor of a "quiet sky field".

- **Private identity and data isolation.** Accounts use "synthetic UUID primary keys"; PII is "strictly isolated and encrypted at rest"; telemetry uses "strict PII/per-bird data isolation"; and social access is limited to "read-only host-invited visits" with "private visit audit logging."

- **Accessibility as a first-class rendering path.** The plan pairs visual/audio features with "Reduced-Motion Mode", "call captions", a dedicated `aria-live="polite"` region, "WCAG AA compliance", and keyboard navigation. When audio is unavailable, captions are "enabled automatically."

- **Budgeted reliability across devices and browsers.** The plan turns quality into enforcement targets: "< 2.0 MB" initial JS, "< 500 ms" time-to-first-bird render, "60 fps", "0 bytes" uncollected memory growth, p99 simulation tick latency under "5.0 seconds", and a Playwright matrix for the last two major versions of Chrome, Safari, Firefox, and Edge.

## Per-feature whys

### 1. Scope and Boundaries

- **Modern web-only single-page application:** NOT RECOVERABLE FROM PLAN

- **Email magic-link authentication with 15-minute token expiry:** NOT RECOVERABLE FROM PLAN

- **Synthetic UUID-based account models:** The plan ties this to isolation: synthetic account identifiers are "used in all internal systems", and the risk matrix says synthetic UUIDs are used "exclusively in downstream tables, logs, and telemetry" to mitigate "PII Leakage in Telemetry & Logs."

- **Single-user canonical aviary per account:** The plan's rationale is canonical state: "single-user canonical aviary per account" belongs with "server-authored" snapshots and the separation that ensures "multi-device consistency."

- **Multi-device sync:** The architecture explicitly exists to ensure "multi-device consistency", and Phase 3 validates "server-authored vector delta sync across multi-device client sessions."

- **JSON account export:** NOT RECOVERABLE FROM PLAN

- **Soft/hard account deletion with 30-day soft window:** NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick:** The plan says server-side simulation keeps personality state canonical: the background worker processes event logs, calculates deltas, updates mood states, generates Field Notebook entries, and writes "canonical snapshots."

- **Personality vector persistence:** The vector is a persistent hidden trait model used by drift, mood, and position. The plan grounds it in traits such as `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, and `curiosity`, then uses those traits for "Personality Vector Buffers" and "current_perch_zone."

- **Low-pass filter drift model:** The rationale is gradual, non-punitive expression: drift is "monotonic toward expressive", calibrated so "measurable instrument drift" occurs around one week and "visible user drift" around three weeks, while traits "never decrease."

- **Mood state machine:** The plan makes mood a "Fast-timescale state" driven by local time, recent events, personality buffers, and session continuity, so the aviary can respond dynamically without altering the slower personality vector.

- **Local-time day/night cycles:** The plan uses local time to shape behavior: `drowsy` probability increases near dusk/night and `alert` spikes at local morning.

- **Rare ambient weather events:** NOT RECOVERABLE FROM PLAN

- **Return-greeting with absence-weighted randomized stagger:** NOT RECOVERABLE FROM PLAN

- **Idle attention presence accounting:** The rationale is "presence honesty." Presence only counts when visibility, focus, and recent pointer/key activity are all true, and the risk matrix names "Presence Spoofing / Background Tab Drift Inflation" as the risk being mitigated.

- **Listen-in audio focus re-balance:** The plan frames listen-in as focus without isolation: target bird gain ramps up, non-targeted birds scale down to an "ambient floor", and "Muting non-targeted birds entirely is strictly prohibited."

- **Offer gestures for seed, song fragment, and still pool:** The plan gives behavioral effects: accepted offers boost `curiosity` and `boldness`, and mood transitions map seed to `content` and song fragment to `curiosity`.

- **Per-bird cooldowns on offers:** NOT RECOVERABLE FROM PLAN

- **Opt-in settle evening gesture:** NOT RECOVERABLE FROM PLAN

- **5-second undo for settle:** NOT RECOVERABLE FROM PLAN

- **Read-only auto-generated Field Notebook:** The plan positions it as a sparse naturalist record: the simulation evaluates a "Sparse Notebook Generator" and generates an entry "if milestone hit"; notebook content is "Naturalist prose observation in lowercase present tense."

- **Responsive single horizontal viewport:** NOT RECOVERABLE FROM PLAN

- **Three perch zones:** The plan says perch position is a signal derived from mood and `boldness`; birds select front, middle, or back "autonomously based on current mood and `boldness`."

- **Mid-action load state and quiet sky field fallback:** The plan's rationale is continuity without stock loading UI: the scene "initializes immediately displaying birds mid-action", and slow initial state fetches render "a quiet sky field" instead of "a loading spinner."

- **Ambient leaf/feather drift:** NOT RECOVERABLE FROM PLAN

- **Auto-fading top bar chrome:** The plan calls the top bar a "Minimalist overlay" and has it auto-fade after inactivity, restoring on mouse movement or keypress, keeping controls available while quieting chrome.

- **Reduced-motion mode:** The plan replaces continuous animation with "slow cross-fades" between static pose keyframes, removes parallax drift, and keeps "color palette shifts", preserving the scene while reducing motion.

- **Procedural WebAudio call synthesis:** The plan bans canned audio and mitigates "Looped Sound Perception" through "100% procedural WebAudio call synthesis", species grammars, and pitch/timing perturbation.

- **Dynamic chorus mixing:** The plan uses spatial panning and randomized motif offsets to make chorus behavior procedural rather than looped.

- **Listen-in mix scaling:** The plan gives exact mix behavior: target bird gain ramps to `1.0`, non-targeted birds scale to `0.15`, and full muting is prohibited.

- **Graceful silent fallback with automated captions:** The plan says that if WebAudio is unavailable or blocked, the application "runs silently with call captions enabled automatically."

- **Read-only host-invited visits:** The rationale is quiet social access without social-network mechanics: visits are "Optional & Quiet", "read-only", and "host-invited", while public discovery, feeds, co-presence, comments, and similar features are non-goals.

- **One-time email visit link with 30-day expiry:** NOT RECOVERABLE FROM PLAN

- **Host revocation affordances:** NOT RECOVERABLE FROM PLAN

- **Private visit audit logging:** NOT RECOVERABLE FROM PLAN

- **Screen-reader naturalist prose narration:** The plan requires narration to use "Naturalist field notes" and explicitly forbids mechanical state lists, preserving the product voice for screen-reader users.

- **Call captions derived from runtime motifs:** Captions are tied to actual synthesis parameters, such as "a soft three-note rise", so the caption describes the currently generated call rather than a static label.

- **WCAG AA compliance:** NOT RECOVERABLE FROM PLAN

- **Keyboard navigation:** NOT RECOVERABLE FROM PLAN

- **Aggregate operational telemetry with strict PII/per-bird isolation:** The plan wants operational telemetry while preserving isolation: telemetry is aggregate, and PII/per-bird data isolation is strict.

### 2. System Architecture & Component Topology

- **Client rendering separated from server-side simulation:** The plan states the reason directly: to ensure "multi-device consistency, presence honesty, and zero client-owned personality states."

- **Unidirectional Event Pipeline:** Clients submit interaction events to an append-only event log so simulation inputs are events rather than client-owned trait values.

- **Server-Authored State Engine:** The background worker processes event logs, applies vector deltas, updates moods, generates notebook entries, and writes "canonical snapshots", making the server the state author.

- **Snapshot Pull & Interpolation:** Clients fetch lightweight JSON snapshots on load, visibility change, resume, and keepalive intervals, then "smoothly interpolate bird positions and states between snapshots."

- **Read-Only Snapshot CDN / Cache:** The plan pairs read-only snapshots with initial load and hydration targets, using lightweight snapshots as the client-facing state surface.

- **API Gateway & Auth:** NOT RECOVERABLE FROM PLAN

- **Relational Storage with PostgreSQL / Redis:** NOT RECOVERABLE FROM PLAN

### 3. Data Schema & Persistence Model

- **Encrypted account email:** The plan says PII email is "strictly isolated and encrypted at rest", and `email_encrypted` is used for magic link authentication.

- **Account settings for reduced motion, captions, and visit notifications:** The settings object persists user-level controls for `reduced_motion_override`, `captions_enabled`, and `visit_notifications_enabled`.

- **Aviary `created_at` for age-based adoption eligibility:** The plan states `created_at` is "used to compute aviary age for bird adoption eligibility."

- **Aviary `settled_at`:** NOT RECOVERABLE FROM PLAN

- **Stable internal bird ID:** The plan says the bird ID "persists across renames and syncs", giving birds stable identity independent of editable names and devices.

- **Species ID:** The plan uses `species_id` as the reference to "species grammar/visual definition", tying bird records to audio and visual behavior.

- **Editable bird name:** NOT RECOVERABLE FROM PLAN

- **Hidden personality traits:** The plan calls the vector "Hidden numerical traits" and uses those traits to influence drift, mood probabilities, perch zone, listen-in effects, and offer effects.

- **Current mood:** The mood field carries "Fast-timescale state" that can react to time of day, events, personality buffers, and absences.

- **Current perch zone:** The plan describes it as a "Position signal derived from mood & boldness."

- **Interaction event payloads:** The event log stores offer type, presence duration, listen-in toggles, settle commands, and related payloads so the simulation tick can process user interactions.

- **Notebook entry content in lowercase present tense:** The plan specifies "Naturalist prose observation in lowercase present tense", matching the broader naturalist voice.

- **Visit invite token hash, status, and expiry:** The visit schema supports one-time access, revocation, expiration, and pending/active state for the quiet invited-visit flow.

- **Visit `last_visited_at`:** NOT RECOVERABLE FROM PLAN

### 4. Simulation Engine & Personality Drift Math

- **Active-aviary 60-second tick cadence:** NOT RECOVERABLE FROM PLAN

- **Fetching unprocessed `interaction_events`:** The simulation tick consumes the append-only interaction log as its input before computing state changes.

- **Presence-time and attentive interactions delta:** The delta calculation is the bridge from honest presence and interactions into trait drift.

- **Monotonic low-pass drift filter:** The plan says traits "shift UP on presence" and "NEVER decay on neglect"; if inputs are zero, delta is zero.

- **Sparse notebook generator on milestones:** The flow generates a notebook entry only "if milestone hit", keeping the notebook sparse rather than continuous.

- **Drift alpha calibration:** The plan calibrates alpha so internal measurable drift appears around one week / 100 presence-minutes and visible drift around three weeks / 300 presence-minutes.

- **Listen-in multiplier on target bird traits:** Listen-in applies a `1.5x` multiplier to the target bird's `social_warmth` and `vocal_frequency`.

- **Accepted offers boosting curiosity and boldness:** Accepted `offer` events boost `curiosity` and `boldness`, connecting gestures to expressive personality movement.

- **Mood transition from local time of day:** The plan maps dusk/night toward `drowsy` and local morning toward `alert`.

- **Mood transition from recent events:** Recent events map to mood changes: seed accepted leads to `content`, song fragment to `curiosity`, and alarm call to `wary`.

- **Mood transition from personality vector buffers:** High `boldness` reduces `wary` transition probability by `50%`.

- **Session continuity for mood:** Mood at session start equals the last snapshot modified by time-decay toward base mood during long absences, preserving continuity without freezing state.

### 5. Client Rendering, Audio & Animation Pipeline

- **Canvas/WebGL full-bleed rendering with DPR auto-scaling:** The plan uses a single full-bleed container and DPR auto-scaling to support the responsive visual scene.

- **Autonomous perch allocation with no drag-and-drop:** Birds choose perches based on mood and `boldness`, and "No user drag-and-drop" keeps movement autonomous.

- **Initial scene pre-roll with birds mid-action:** The plan initializes immediately with birds mid-action, making the aviary feel already alive before all state fetches complete.

- **Top bar contents for Settings, Accessibility, Field Notebook, and Offer icons:** NOT RECOVERABLE FROM PLAN

- **Standard micro-motion:** The plan uses skeletal/mesh deformation or frame interpolation for "preening, scanning, weight shifting, and head tilts" at 60fps.

- **Reduced-motion cross-fades and parallax removal:** Reduced motion swaps continuous bone animation for 1.5-second static-pose cross-fades and removes leaf/feather parallax drift.

- **Motif Synthesizer with WebAudio nodes:** The synthesizer uses `OscillatorNode`, `BiquadFilterNode`, and `GainNode` envelopes from species grammars so no MP3/WAV loops are downloaded.

- **Chorus spatial panning and randomized motif offsets:** Standard chorus uses spatial panning and randomized offsets to keep calls mixed and procedural.

- **Listen-in target gain, ambient floor, and no full mute:** The mix focuses attention by ramping one bird and preserving the rest at an ambient floor, because muting non-target birds is "strictly prohibited."

- **WebAudio blocked/unavailable fallback:** The silent fallback keeps the application usable with call captions enabled automatically.

### 6. Accessibility & Interaction Specifications

- **Dedicated `aria-live="polite"` narration region:** The plan delivers naturalist narration through a polite live region so screen-reader updates are part of the interface.

- **Idle narration cadence and immediate action dispatch:** Idle updates are throttled to 30-60 seconds, while user actions dispatch immediately, balancing calm cadence with responsiveness.

- **Naturalist prose instead of state lists:** The plan forbids state lists and requires naturalist field-note phrasing, matching the product voice.

- **Call captions near the calling bird with fade transitions:** Captions appear near the active bird and fade in/out, tying visible text to the calling bird and the generated motif.

- **Presence pings requiring visibility, focus, and recent input activity:** The contract emits a ping only when all three conditions hold and stops immediately otherwise, matching the "Presence Spoofing / Background Tab Drift Inflation" mitigation.

### 7. Performance Budgets & Technical Constraints

- **Initial JS bundle under 2.0 MB:** The plan enforces this with a "CI build threshold failure check."

- **Time-to-first-bird render under 500 ms:** The plan uses "Edge snapshot hydration & asset pre-compilation" to enforce fast first render on mid-tier 4G mobile.

- **Idle frame rate at 60 fps:** The plan uses "WebGL batching & requestAnimationFrame throttling" to preserve idle frame rate on a 5-year-old laptop.

- **Zero uncollected memory growth over 30 minutes:** The plan enforces this with "Automated CI Puppeteer heap snapshot testing."

- **Simulation tick latency p99 under 5 seconds:** The plan uses Datadog/CloudWatch service alarms to enforce the bound.

- **Last-two-major browser compatibility:** The plan uses an "Automated Playwright browser test matrix" for Chrome, Safari, Firefox, and Edge.

### 8. Release Strategy & Rollout Plan

- **Phase 1 Core Engine & Audio Proof-of-Concept:** The plan starts with WebAudio motif synthesis, species grammar definitions, basic rendering, server simulation, schema, and auth as the proof-of-concept foundation.

- **Phase 2 Interaction & Accessibility Integration:** The plan groups return-greeting, presence, listen-in, offers, narration, captions, and reduced-motion together as integration work.

- **Phase 3 Multi-Device Sync & Social Invites:** The plan validates server-authored vector sync across devices before building read-only invitations and visit logging.

- **Phase 4 Optimization, Auditing & Launch:** The plan holds bundle/render benchmarks and security audit work for launch hardening.

- **Day 1 adoption of two starter birds:** NOT RECOVERABLE FROM PLAN

- **Third bird at 60 days and additional birds at multi-month intervals:** NOT RECOVERABLE FROM PLAN

- **Hard cap of seven birds:** NOT RECOVERABLE FROM PLAN

### 9. Risk Matrix & Mitigation Strategies

- **Strict presence contract with server-side sanity validation:** This mitigates "Presence Spoofing / Background Tab Drift Inflation."

- **Clients never submit absolute trait values:** This mitigates "Personality Vector Sync Corruption" by ensuring only server simulation ticks execute additive delta updates.

- **Pitch/timing perturbation and zero static audio files:** This mitigates "Audio Uncanniness / Looped Sound Perception."

- **Automated PR reviews against counters, streak logic, badges, and toast banners:** This mitigates "Gamification Leakage / Product Identity Shift."

- **Synthetic UUIDs used exclusively downstream:** This mitigates "PII Leakage in Telemetry & Logs."
