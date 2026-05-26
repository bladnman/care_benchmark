## System-level intent

- **A quiet aviary, not an engagement loop.** The plan repeatedly rejects engagement mechanics: "achievements, streaks, levels, scores, badges," "push notifications," "outbound engagement ping," and "show-off mode" are all out of scope. The first-frame strategy says "No spinner" and wants the quiet field to read as "the aviary is here" rather than "the app is loading." The top bar "fades to near-transparent after cursor stillness."
- **No custodial obligation or punishment.** The non-goals reject "death, hunger, distress, happiness decay, custodial obligation." The drift section reinforces this with "Traits never decrease" and "On neglect, the delta is simply zero." The birds can become more expressive, but the plan avoids decay pressure.
- **Slow, expressive change.** The bird engine calls for "monotonic-toward-expressive drift," with "one week" measurable in instruments, "three weeks" visible to the user, and "a single session" never visibly changing a bird. The drift risk frames the danger as feeling either "gamey" or like "nothing is happening after weeks."
- **Server-canonical simulation, client-side rendering.** The Simulation Service is "the canonical state owner" and the "only writer of personality vectors." Clients write events, not state mutations. The render pipeline "never makes simulation decisions" and "only visualizes what the server has already decided."
- **Additive facts instead of sync conflicts.** The sync model says events are facts like "user listened in to Pip for 3 minutes," not "set Pip's social_warmth to 0.62." The stated anti-pattern is last-write-wins silently discarding drift from another device.
- **Naturalist voice, observations, and sparsity.** Notebook and narration prose use "naturalist voice, lowercase, present-tense." Entries are "read-only" and "sparse." Narration and event prose are "observations, not announcements," and notebook templates are "finite, reviewed" so every entry has been approved for voice.
- **Procedural life over recorded repetition.** Calls are synthesized with WebAudio, motif variance means "no two calls are ever identical," and the plan makes the "no recorded audio" rule unconditional. Procedural variation is also treated as correct when two clients synthesize "different-but-similar calls."
- **Accessibility as a first-class surface.** Accessibility is "a first-class workstream from day 1, not a v1.1 fix." Reduced motion is a "designed cross-fade surface, not stripped fallback," and all other functionality is preserved: "calls play, moods change, drift continues, notebook updates."
- **Privacy boundary as architecture, not policy alone.** Telemetry is "aggregate-only"; the pipeline has a "hard pipeline boundary" and "never reads from the state store or event log." The plan explicitly avoids data that could "reconstruct a user's relationship with their aviary."
- **Performance protects the illusion of a living place.** Budgets include "time to first bird visible <500ms," "60fps," and "snapshot response time <100ms." The rationale column says the aviary should feel "already-running," and bundle limits keep "time-to-first-bird recoverable on mid-tier mobile over 4G."

## Per-feature whys

**Scope and account surface**

- **Browser-only client**: NOT RECOVERABLE FROM PLAN
- **Single-page responsive application**: The scene must work from "narrow phone to wide desktop" while birds remain "fully visible, never cropped"; on tall/narrow viewports it is "letterboxed" rather than cropped.
- **Single-user accounts**: NOT RECOVERABLE FROM PLAN
- **Email magic-link auth**: NOT RECOVERABLE FROM PLAN
- **Per-device session tokens**: NOT RECOVERABLE FROM PLAN
- **Synthetic UUID account IDs and encrypted email**: The plan keeps account identity separate from user-visible or operational state by using a synthetic UUID and storing `email_encrypted` in a "single storage location."
- **Soft-then-hard deletion with 30-day window**: The account API includes `/account/recover` to "Cancel pending deletion (within 30-day window)," so the soft window exists to make deletion reversible during that period.
- **Account export by JSON snapshot via email link**: NOT RECOVERABLE FROM PLAN
- **Email change with verification**: NOT RECOVERABLE FROM PLAN
- **Session revocation**: NOT RECOVERABLE FROM PLAN

**Aviary simulation and bird engine**

- **Server-side tick at about one minute**: The tick is the place where events become canonical state: it consumes events, computes drift, transitions moods, schedules calls, writes state, invalidates cache, and pushes deltas. One minute is recommended because faster ticks "increase server cost without user-visible benefit."
- **Personality vector drift**: Drift exists so regular visits create change that is first "measurable in instruments" and later "visible to the user," while "a single session never produces visible drift."
- **Monotonic drift on neglect**: Traits are clamped so they "never decrease"; on neglect the bird "stays where it is, it doesn't regress." This carries the non-custodial intent.
- **Drift calibration harness**: Synthetic light, moderate, and heavy usage profiles verify that drift is neither "too fast" nor "too slow," and coefficients are tunable without client updates.
- **Mood transitions**: Moods combine recent interactions, time of day, ambient events, personality, and bird-to-bird effects. A minimum dwell time prevents "rapid oscillation," and probabilistic transitions keep identical inputs from always producing identical outputs.
- **Call scheduling and grammar**: The server decides "when" and "what motif class"; the client synthesizes the audio. This keeps simulation decisions canonical while allowing procedural calls to vary.
- **Bird-to-bird interactions**: Call response, mood contagion, and chorus emergence make birds react to each other through "social_warmth," nearby wary birds, and overlapping vocal schedules.
- **Day/night cycle**: The tick advances day phase using the account holder's local timezone, and the client renders a continuous palette shift from `day_phase` and `day_phase_progress`.
- **Ambient weather**: Weather creates occasional rain or wind that can affect mood weights, weather overlays, and notebook-worthy combinations like "rain during a long quiet period."
- **Two starter birds per account**: NOT RECOVERABLE FROM PLAN
- **Cap of 7 birds**: NOT RECOVERABLE FROM PLAN
- **About six species in the pool**: NOT RECOVERABLE FROM PLAN
- **Procedural personality vectors**: Seed values come from species-specific distributions, then drift through server-written values so birds can become more expressive without exposing raw stats.
- **Plumage saturation as a hint**: The snapshot exposes only `plumage_hint.saturation_level`; personality vector values are "never" included. The user sees a visual consequence, not a stats panel.
- **Renameable bird names**: NOT RECOVERABLE FROM PLAN
- **No personality vector numerical exposure**: The plan forbids a "stats panel," "debug view," or toggle because drift milestones should be described as behavioral consequences, not numbers.

**Interactions**

- **Return-greeting**: The greeting is "the welcome," especially for screen-reader users, so it is narrated within 1-2 seconds and varies procedurally with absence awareness.
- **Listen-in**: Listen-in lets the user focus on a bird by ramping that bird's gain up while other birds ramp down to about 30%; "other birds never go silent."
- **Offer seed / song fragment / still pool**: NOT RECOVERABLE FROM PLAN
- **Per-bird offer cooldown**: NOT RECOVERABLE FROM PLAN
- **Settle**: Settle is an "opt-in soft session-end" with evening lighting and audio ramp-down, but it remains reversible through a "5s undo" or any click.
- **Presence accounting by 3-signal conjunction**: The plan calibrates visibility, focus, and recent activity because "watching birds without moving is the actual product," while users who walk away should stop accumulating presence.
- **Presence pings stopping on signal loss**: When any of the three signals drops, the client stops pings so the tick "sees the gap and stops accumulating presence-time."
- **Tab open and tab close events**: NOT RECOVERABLE FROM PLAN

**Field notebook and product voice**

- **Field notebook**: The notebook turns noteworthy behavior into a scrollable naturalist history, while remaining "read-only" and sparse instead of becoming a task or feed.
- **Notebook noteworthy-event selection**: First-greeter changes, unusual moods, weather-behavior coincidences, offer patterns, and drift milestones are used because they are events worth observing in the aviary.
- **Notebook sparsity enforcement**: Cooldowns and noteworthiness thresholds prevent active users from getting "a flood of entries."
- **Template-based notebook prose**: Templates are curated and finite so the system maintains the "naturalist voice"; every possible entry has been "read by a human and approved for voice."
- **Read-only notebook UI**: NOT RECOVERABLE FROM PLAN
- **Scrollable, paginated notebook history**: Pagination and virtualization keep history available while only visible entries remain in the DOM, supporting the "No memory growth" budget.

**Audio pipeline**

- **Procedural call synthesis through WebAudio**: The client builds oscillator/gain graphs at call time so calls use no recorded audio and "no two calls are ever identical."
- **Species motif libraries**: Motifs define pitch, timing, intensity, envelope, and timbre so each species can have recognizable calls with enough variance to avoid repetition.
- **Chorus mixing**: Layered procedural calls create "a real chorus" when birds overlap, without "phase-canceling artifacts" or complex ducking.
- **Listen-in audio mix ramp**: Two-second ease-in-out ramps make focus gradual; disengage reverses the ramp back to ambient mix.
- **WebAudio silence plus captions fallback**: If AudioContext fails, the aviary still runs in silence and captions turn on, while the "no recorded audio" rule remains unconditional.
- **AudioContext created on first user gesture**: This follows browser autoplay policy; before that, the aviary still renders visually, and the first greeting may be visual-only.
- **AudioContext suspended on hidden tabs**: Suspension on tab hidden "saves battery" and resumes on visibility change.
- **Per-call oscillator cleanup**: Oscillator nodes are created per call and disconnected after playback so there is "No memory growth."

**Rendering and frontend behavior**

- **Render pipeline boundary**: Client rendering takes snapshots and interpolates between them; it "never makes simulation decisions."
- **Canvas 2D or lightweight WebGL**: Canvas 2D is recommended because the scene complexity is low enough for "7 birds + ambient effects at 60fps"; WebGL is reserved for visual-fidelity or profiling needs.
- **Bird sprites and plumage rendering**: SVG sprites or compact bitmaps keep assets small, while plumage saturation uses `plumage_hint.saturation_level` rather than raw personality values.
- **Idle micro-motion**: Mood-weighted idle states let the user read mood from motion; transitions like preening to scanning carry the change instead of explicit status effects.
- **Perch change animations**: When the server moves a bird, the client animates a short flight or hop over 1-2 seconds so snapshot changes are visually smooth.
- **Mood changes without explicit visual transition**: The plan says the user reads mood changes "from the motion, not from a visual effect."
- **Day/night palette interpolation**: The client smoothly shifts gradient, lighting warmth, and ambient brightness from the day-phase fields.
- **Ambient leaf and feather drift**: NOT RECOVERABLE FROM PLAN
- **Weather overlay**: Rain particles and wind-ripple effects render only when `ambient_weather` is active in the snapshot, keeping weather tied to server-authored state.
- **Top-bar auto-fade**: NOT RECOVERABLE FROM PLAN
- **Quiet-field first frame**: The quiet field avoids a loading state; with no spinner, it should read as "the aviary is here."
- **Responsive layout and letterboxing**: Narrow, medium, and wide layouts keep perch spacing appropriate, and letterboxing prevents cropping on tall/narrow screens.

**Accessibility surfaces**

- **Screen-reader narration**: Narration generates slow naturalist prose from the aviary state so non-visual users receive the same observed aviary experience.
- **Narration cadence control**: Minimum intervals and batching prevent the screen reader's queue from being "overwhelmed."
- **User-initiated narration priority**: Greetings, offer reactions, and settle jump the queue because they are user-facing moments, but they remain written as observations.
- **ARIA live region with `aria-live="polite"`**: The prose is visually hidden but present in the DOM so the screen reader can announce it "at its own pace."
- **Call captions**: Captions are generated from motif parameters so each caption "matches what was actually synthesized"; they also become the default when WebAudio is unavailable.
- **Keyboard navigation**: Tab, arrows, Enter, and Escape make the scene operable through birds and top-bar controls without pointer input.
- **Focus indicator**: The focused bird needs a soft high-contrast outline visible against both "midday" and "night" states.
- **WCAG AA contrast**: Top bar text, settings, captions, and any visual narration need contrast against changing scene backgrounds.
- **Reduced-motion mode**: Motion becomes cross-fades and simplified overlays, but "calls play, moods change, drift continues, notebook updates."

**Sync, social, and observability**

- **Server-canonical multi-device sync**: Two devices read the same canonical state, and both can submit events without overwriting each other.
- **Append-only event log and additive deltas**: Events are ordered facts; the additive model prevents last-write-wins from discarding one session's drift.
- **Snapshot delivery on tab open, visibility change, gaps, and keepalive**: Pulling snapshots lets clients catch up after hidden tabs, navigation, or suspend gaps.
- **WebSocket stream as optional live update path**: WebSocket is "an optimization over polling," while polling remains the fallback for restrictive proxies.
- **Visit invitations and read-only ambient view**: Visitor clients render the host aviary but disable interaction endpoints, keeping visits ambient and outside social-network surfaces.
- **Visit invitation revocation and 30-day expiry**: NOT RECOVERABLE FROM PLAN
- **Silent visit log**: NOT RECOVERABLE FROM PLAN
- **Aggregate-only telemetry**: Metrics cover request counts, latency, errors, render timing, and audio errors for operational health while avoiding per-account behavior.
- **Telemetry pipeline hard boundary**: Separate data stores, access controls, IAM, and network policy make a privacy-boundary violation "a clean cut."
- **Synthetic monitoring**: Automated browsers measure time-to-first-bird, snapshot latency, audio initialization, and fps so regressions alert against budgets.
- **Bundle-size CI gate**: The build fails above 2 MB gzipped because bundle creep would blow the "time-to-first-bird" budget.
- **RUM first-bird and frame timing**: Client-side timing verifies the perceived experience: first bird visible, frame histograms, and audio failures.

**Rollout and launch**

- **Internal alpha**: Team-only accounts focus on "simulation tick correctness, drift calibration, audio pipeline stability" before broader exposure.
- **Closed beta**: About 100 users test whether drift feels right, calls feel "alive or annoying," and notebook entries feel "charming or spammy."
- **Open launch**: NOT RECOVERABLE FROM PLAN
- **Birds-per-aviary ramp**: NOT RECOVERABLE FROM PLAN
- **New bird offer prompt**: The prompt uses naturalist voice, "a new species has been spotted near the aviary," and the offer remains available indefinitely if ignored.
- **Day-one drift calibration dashboard**: Aggregated vector distributions answer whether birds are drifting at the expected rate or saturating too fast or slow.
- **Day-one audio health instrumentation**: AudioContext success, synthesis errors, and listen-in ramp smoothness track whether the audio pipeline is stable.
- **Day-one notebook quality instrumentation**: Entry frequency distribution checks whether entries are too frequent for active users or too sparse for light users.
- **Day-one accessibility instrumentation**: Narration latency, caption rendering errors, and reduced-motion activation rate track whether the shipped accessibility surfaces work.
