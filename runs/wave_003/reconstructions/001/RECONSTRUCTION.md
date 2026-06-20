## System-level intent

- **Server-authored continuity, not client ownership.** This shows up in Architecture and Sync Model: the "server-side simulation tick" is the "single source of truth," clients "pull state snapshots and interpolate," and there is "no client-side personality ownership." The same intent appears in conflict handling: "No last-write-wins" and "additive server-authored deltas processed in event-log order."

- **Slow, cumulative aliveness.** The plan repeatedly frames personality as something that drifts over time, not instantly: "daily-ish deltas," "monotonic tendency (never decreases)," "visible drift expected after 3 weeks of regular use," and KPIs around "drift visibility timeline" and "user-reported feeling of aliveness." The Risks section names the failure mode as birds feeling "static or overly reactive."

- **A quiet aviary, not a game or Tamagotchi.** Scope excludes "scores, achievements, streaks, levels," "bird death, hunger, distress meters," and other gamification/Tamagotchi mechanics. This is reinforced by the naturalist-facing surfaces: "field notebook," "naturalist observations," "ambient playback," and prose descriptions of calls.

- **Naturalist voice instead of a dashboard state list.** The Field Notebook uses "naturalist prose, lowercase, present tense." Screen-reader support is "running narration (not state list)," and captions use "same voice as notebook." Error surfaces are "matter-of-fact," keeping even failure copy within the same restrained product voice.

- **Accessibility ships with the main product and keeps the affective core.** Accessibility appears in Scope, Frontend Rendering Pipeline, Accessibility Surfaces, Performance, and Risks. The plan says reduced motion must have "maintained charm" and "same state transition logic," and it explicitly warns against "Accessible surfaces losing affective core" by ensuring accessibility work "ships with main product, not as afterthought."

- **Presence and interaction should be observable, additive, and conservative.** Presence is "comprehensive, conjunction-based" and requires document visibility, focus, and recent input signals. Interactions go to an "append-only server log," while personality deltas come from "presence-time, listen-in duration, offers" and are processed in event-log order.

- **Privacy boundary around bird state and interaction history.** Analytics are "Aggregate-only monitoring (no per-bird state)." The Performance Budgets and Observability section states "No per-bird state in aggregate telemetry," the simulation database is "isolated from analytics warehouse," and telemetry "can never infer interaction history."

- **Performance is part of product feel and rollout safety.** The plan sets concrete budgets: bundle size "≤2MB," "first bird visible <500ms," "60fps idle," no memory growth, and simulation tick "p99 latency ≤5 seconds." Rollout monitoring watches for "performance regressions," with strict CI checks enforced at merge.

## Per-feature whys

### Scope

- **Two starter birds per aviary (expandable to seven based on aviary age):** NOT RECOVERABLE FROM PLAN

- **Single-user accounts:** The plan ties this to one "single server-side record per account" and excludes "shared/collaborative aviaries" and "social network surfaces," keeping the aviary account model non-collaborative.

- **Magic-link sign-in:** NOT RECOVERABLE FROM PLAN

- **Multi-device sync via server-side simulation tick:** The plan's reason is consistency: the server is the "single source of truth," devices read "identical snapshots," clients never write personality directly, and conflicts avoid "last-write-wins."

- **Field notebook:** It exists to surface "auto-generated naturalist observations" tied to "observable events" in "naturalist prose, lowercase, present tense."

- **Read-only field notebook:** NOT RECOVERABLE FROM PLAN

- **Presence accounting (comprehensive, conjunction-based):** Presence is a core drift signal: "presence-time" has "major weight" in personality deltas, and detection uses conjunction across visibility, focus, and recent input.

- **Visit invitations (off-by-default):** NOT RECOVERABLE FROM PLAN

- **Read-only guest viewing:** The plan presents visits as "read-only, ambient playback," aligning with exclusions of "shared/collaborative aviaries" and social-network surfaces.

- **Audio system with procedural calls:** Procedural calls let species motifs be shaped by "personality-driven timing," "mood-appropriate motifs," and drift while "recognizability" is maintained.

- **Chorus mix:** The mix supports focus without breaking ambient presence: "focused bird rises in mix," while others become "quiet to ambient but never silent."

- **Screen-reader narration:** The reason is affective accessibility: narration is "naturalist prose," a "running narration (not state list)," and user-initiated events get "queue priority."

- **Reduced-motion mode:** It preserves the same product state without frame-by-frame motion: "Cross-fade rendering replaces frame-by-frame animations," with "maintained charm," "same state transition logic," and calls/captions at "full quality."

- **Call captioning:** Captions translate calls into "proximate prose" such as "soft three-note rise," use the "same voice as notebook," and remain available when WebAudio falls back to silence.

- **Keyboard navigation support:** The plan frames it as accessibility reachability: users can tab through the interface, use arrow keys for focus, Enter for listen-in, Escape to exit, and "keyboard reachability" is tested.

- **Matter-of-fact error surfaces:** NOT RECOVERABLE FROM PLAN

### Architecture

- **Client-server model:** The purpose is to keep personality, mood, and state canonical on the server while clients "pull state snapshots and interpolate for smooth rendering."

- **Simulation Service:** It exists to "run continuous tick," update "personality drift," and transition moods independent of client connectivity.

- **API Gateway:** NOT RECOVERABLE FROM PLAN

- **Identity Service:** NOT RECOVERABLE FROM PLAN

- **Asset Service:** NOT RECOVERABLE FROM PLAN

- **Analytics/Telemetry:** Its reason is operational visibility without bird-state exposure: "Aggregate-only monitoring (no per-bird state)."

### Data Model

- **Bird UUID:** NOT RECOVERABLE FROM PLAN

- **User-editable bird name:** NOT RECOVERABLE FROM PLAN

- **Species from 6-member pool:** NOT RECOVERABLE FROM PLAN

- **Personality vector:** The vector supplies the traits that drive drift, mood, calls, and appearance: "boldness, social warmth, vocal frequency, plumage saturation, curiosity."

- **Mood:** Mood is a fast-timescale state used to shape behavior and rendering, computed from "personality, recent interactions, time-of-day, ambient events."

- **Drift history:** It records cumulative personality movement, matching the plan's "cumulative" and "monotonic tendency" drift model.

- **Interactions events:** The event set is the input stream for server-authored change; recent interaction events drive personality deltas and mood updates.

- **Notebook entries:** The plan's reason is to bind prose observations to evidence: entries have timestamps and are "tied to observable events."

- **Personality Drift:** Drift is how sustained use becomes visible: server-computed deltas come from presence-time, listen-in, and offers, with visible drift after regular use.

- **Mood:** Mood provides the fast-timescale layer over slower personality drift, updating from personality, recent interactions, time of day, and ambient events.

### API Surface

- **Authentication endpoints:** NOT RECOVERABLE FROM PLAN

- **State snapshot and events endpoints:** They support the server snapshot model: clients fetch the aviary state and event history rather than owning personality state locally.

- **Listen-in event:** It supports focused engagement with one bird; the audio plan raises the focused bird in the mix and uses gradual level changes.

- **Offer event:** Offers are one of the inputs to personality deltas, with "minor" weight.

- **Settled event:** NOT RECOVERABLE FROM PLAN

- **Presence ping:** It supports presence-time accounting, which has "major weight" in personality deltas.

- **Account settings:** NOT RECOVERABLE FROM PLAN

- **Invitation endpoints:** They support the off-by-default visit invitation surface and read-only visit access.

- **Visit access:** The reason given is "read-only, ambient playback."

- **Export:** NOT RECOVERABLE FROM PLAN

- **JSON with metadata:** NOT RECOVERABLE FROM PLAN

### Simulation Engine Design

- **Tick Engine:** It exists so the simulation advances "server-side every ~60 seconds, regardless of client connectivity," reading recent events and writing canonical personality and mood state.

- **Drift Function:** The low-pass filter prevents instant reactivity while still allowing measurable change: "visible drift expected after 3 weeks" and instruments detect after "~1 week."

- **Presence Detection:** It requires a conjunction of visibility, focus, and recent input so presence signals are comprehensive and grounded in actual active presence.

- **Mood Transitions:** They let personality, time zone, weather-like ambient events, and other birds affect moment-to-moment state; examples include rain dampening vocal frequency and wary responses to other birds.

- **Call Grammar:** It keeps calls recognizable while letting species, personality, timing, pitch, mood, and drift shape procedural sound.

### Sync Model

- **Canonical State:** The reason is explicit: "Single server-side record per account; no client-side personality ownership."

- **Multi-device sync:** It is "transparent via server as single source of truth," so devices read "identical snapshots."

- **Conflict prevention:** It avoids "last-write-wins" by using additive server-authored deltas in event-log order.

- **Capture scenarios:** Idempotency keys, retries, and matter-of-fact error pages exist to handle magic-link replay, session timeout mid-write, failed ticks, and sync errors.

### Frontend Rendering Pipeline

- **Three-zone layout:** NOT RECOVERABLE FROM PLAN

- **Single horizontal scene with no panning/scrolling:** NOT RECOVERABLE FROM PLAN

- **Micro-motion:** Mood-shaped idle animations and ambient drift make state visible through "preening, scanning, head-tilting," leaf/feather drift, and cross-fades.

- **Window focus recovery:** It loads a fresh state snapshot so the client resumes from canonical state.

- **Procedurally staggered greet animations:** NOT RECOVERABLE FROM PLAN

- **Listen-in slow-ramp mix changes:** The audio rationale is smooth engagement: "gradual mix level changes," "smooth engagement/disengagement," and "no hard cuts."

- **Performance budgets:** These protect load, smoothness, and stability: small bundle, fast first bird, 60fps idle, and no memory growth over 30 minutes.

### Audio Pipeline

- **Synthesis:** Procedural WebAudio expresses species motif, personality timing, vocal frequency, and mood-appropriate motifs.

- **Chorus mixing:** It preserves ambience during focus by raising the focused bird and making other birds "quiet to ambient but never silent."

- **Listen-in:** It avoids abruptness through "gradual mix level changes" and "no hard cuts."

- **WebAudio fallback:** The plan's reason is graceful degradation: WebAudio unavailable means "graceful silence with captions on by default," with no recorded fallback.

### Accessibility Surfaces

- **Screen-reader cadence:** The cadence of "~30-60s" supports a "running narration" rather than a static state list.

- **Screen-reader queue priority:** User-initiated events get priority so direct actions are narrated promptly.

- **Keyboard focus order and commands:** The plan uses this to ensure the top bar, aviary, and birds are reachable and operable from the keyboard.

- **Reduced-motion accessibility:** It replaces animation with cross-fades while keeping charm, transition logic, and full-quality calls/captions.

- **Captions fade with call:** NOT RECOVERABLE FROM PLAN

- **WCAG AA text, focus indicators, and keyboard reachability:** These ensure copy contrast, visible focus against aviary backgrounds, and tested keyboard access.

### Performance Budgets and Observability

- **Load performance budget:** The plan connects this to first-visible experience: "first bird visible <500ms on mid-tier 4G."

- **Runtime budget:** The reason is stable long-session feel: "60fps idle motion on 5-year laptop" and "no memory growth over 30-minute sessions."

- **Simulation tick p99 latency alert:** It monitors the server tick so tick latency breaches are detected.

- **Observability metrics:** Metrics cover request health, errors, sessions, frame timing, and audio-context errors for "operational health only."

- **Privacy boundary:** The rationale is preventing telemetry from exposing bird lives: no per-bird state, isolated simulation database, and telemetry that "can never infer interaction history."

### Rollout

- **V1 ship strategy:** The plan uses V1 to launch the "core bird engine," basic interactions, accounts and sync, and field notebook first.

- **Birds-per-aviary ramp:** NOT RECOVERABLE FROM PLAN

- **No catalog selection for new birds:** NOT RECOVERABLE FROM PLAN

- **Instrumentation KPIs:** These measure whether the product works as intended: retention, "drift visibility timeline," "user-reported feeling of aliveness," error rate, and performance budgets.

- **Telemetry focus on operational health only:** This preserves the privacy boundary while still watching health.

- **Internal staging to canary to full release:** The plan uses phased rollout to catch "performance regressions" and allows immediate rollback on "drift correctness issues."

### Risks

- **Drift calibration mitigation:** A/B testing and phased rollout exist because drift that is over- or under-sensitive makes birds feel "static or overly reactive."

- **Sync correctness mitigation:** Single source of truth and event-log ordering prevent state divergence from server tick lag.

- **Audio voice testing:** It mitigates procedural calls sounding "too synthetic or robotic" through "extensive voice testing" and personality-based variation.

- **Accessibility integration:** It exists to prevent accessible surfaces from "losing affective core" by shipping accessibility with the main product.

- **Strict CI performance checks:** These prevent bundle-size or render-latency breaches by enforcing budgets at merge.
