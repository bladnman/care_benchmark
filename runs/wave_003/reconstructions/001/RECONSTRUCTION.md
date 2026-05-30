## System-level intent

- **One canonical, server-authored aviary.** The plan repeatedly centers a "single canonical aviary" and "canonical server state." This shows up in Scope ("Multi-device sync via canonical server state"), Architecture ("Server owns state"), Simulation Service ("the canonical aviary state writer"), and Sync Model ("Single canonical source"). The intent is that personality, mood, presence, and drift come from the server, while clients render snapshots and write events.

- **Care without game mechanics.** The plan carries a quiet anti-gamification philosophy: "no streaks, no achievements, no badges, no leaderboards," "no hunger, no death, no visible distress, no decay," and "neglect results in ambient quietude, not regression." The Risks section names the danger as "Tamagotchi feel" and protects against it with slow drift and low-pass filtering.

- **Slow, low-pressure change.** Drift is meant to be gradual: "measurable drift in instruments after ~1 week," "User-visible drift after ~3 weeks," and "no single session shifts traits visibly." Bird count ramp also follows this principle: the third bird is based on "aviary age" and the year-old aviary may reach "five or six birds."

- **Naturalist voice instead of stats.** The plan's product voice is "naturalist prose, lowercase, present-tense, specific, observation-style." It rejects direct mechanical reporting: "Never: 'Pip is at perch 2' or 'Wren mood: content.'" This same intent appears in field notebook prose, screen-reader narration, call captions, and the rule that personality vectors are "never exposed numerically."

- **The aviary should feel already alive.** Rendering and performance choices point to an "already running" surface: "first frame shows birds mid-action," "no entry animation or spinner," "quiet field" if slow, idle motion that "runs regardless of user attention," and "Time to first bird visible <500ms" because below that the "aviary feels already running."

- **Procedural variation over recorded playback.** The audio system is designed around "procedural synthesis mandatory," "motif descriptors" rather than audio files, and "real chorus, not stacked loops." The plan grounds this in avoiding "looped audio," "phase-canceling artifact," and a recorded-audio bundle that would exceed the "2MB JS bundle."

- **Accessibility as a designed surface.** Accessibility is not framed as a fallback. Reduced motion is "not a stripped fallback; its own designed surface," and the risk section says accessible surfaces must not become "stripped fallback." Screen-reader narration, call captions, WCAG AA contrast, and keyboard navigation all ship with the product.

- **Privacy-limited observability.** The plan wants operational insight without behavioral exposure: "Aggregate operational telemetry only," "no per-bird state, no per-account interaction history in telemetry," "no per-bird interaction state for any aggregate purpose," and no pipeline that "could later expose per-account bird behavior."

- **Quiet, bounded social presence.** Visits are deliberately limited: "read-only ambient view, opt-in, revocable, email-based." The plan excludes "Social network surfaces," "co-presence," and later "chat, public discovery, leaderboards." It calls the refusal "structural, not policy."

## Per-feature whys

### Scope

- **Single canonical aviary per account.** The rationale is sync correctness and continuity: all clients read the same snapshot, with "no client-to-client sync" and "no state merging required."

- **Two-to-seven birds.** The plan gives two rationales: v1 starts with "Two birds at adoption" as the first encounter, and the "Seven bird cap" is an "empirical ceiling where call signatures remain individually recognizable."

- **Magic-link authentication.** NOT RECOVERABLE FROM PLAN

- **Single-user accounts.** NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick.** The tick exists so the "aviary runs on server," continues "regardless of client connection," consumes events in order, and writes the canonical snapshot.

- **Personality drift.** Drift is calibrated to avoid both "Tamagotchi feel" and "screensaver feel": measurable after about a week, user-visible after about three weeks, and filtered so no single session visibly shifts traits.

- **Mood system.** The plan uses mood to keep bird behavior continuous across sessions and shaped by state: it "persists across sessions," is "not reset on tab open," and is modulated by interactions, local time, weather, and personality.

- **Procedural call synthesis and per-bird call signatures.** The rationale is recognizable variation without recorded loops: a user should learn "Pip's call by ear," while procedural synthesis avoids "layered loops" and keeps the bundle within budget.

- **Multi-device sync via canonical server state.** The rationale is the same canonical-source rule: "All clients read the same snapshot," and personality is "additive server-authored delta, never client-submitted absolute value."

- **Field notebook.** The notebook records "notable moments" in the same naturalist voice as the rest of the product. The rationale articulated is voice continuity and observation: entries are "auto-generated" every few days per active aviary, not every tick.

- **Listen-in interaction.** The rationale is focused attention without breaking the ambient aviary: the focused bird "rises gradually in mix," while other birds drop to ambient and are "never silent"; transitions are a "slow mix-level ramp."

- **Offer interaction as simulation input.** The plan treats offers as a secondary drift input, alongside listen-in duration and settle gestures, and as recent events that can modulate mood.

- **Offer types and per-bird cooldown.** NOT RECOVERABLE FROM PLAN

- **Settle gesture.** The rationale is an "opt-in session-end" that changes the aviary softly: "slow lighting shift to evening," calls quiet, and "Settle undo" within five seconds reverses it.

- **Day/night cycle anchored to user's local timezone.** The rationale is that time of day shapes mood, palette, and calls: morning has warming light, evening has quieter calls, and night has most birds settled.

- **Ambient weather.** Weather adds small, non-assertive environmental variation: rain is rare, wind occasional, and both affect mood in small ways, such as briefly dampening vocal frequency or making some birds alert/wary.

- **Ambient micro-motion.** The rationale is continuous liveliness. Birds preen, scan, tilt, and shuffle, with motion shaped by mood; the scene "runs regardless of user attention."

- **Visit invitations.** The rationale is quiet sharing without a social network: read-only ambient visits are "opt-in" and "revocable," and the risk mitigation rejects co-presence, chat, public discovery, and leaderboards.

- **Screen-reader narration.** The rationale is to expose the same aviary state through "naturalist prose" rather than mechanical labels, maintaining "Voice continuity with field notebook."

- **Reduced-motion mode.** The plan explicitly says reduced motion is "not a stripped fallback" but "its own designed surface," using cross-fades and slowed ambient shifts while preserving calls, drift, and notebook notices.

- **Call captions.** Captions make procedural calls available in the same product voice: they are generated from the call grammar and described as short prose such as "a soft three-note rise."

- **WCAG AA contrast and full keyboard navigation.** The rationale is accessible operation of all user-copy and controls: contrast applies to top bar, settings, account, error, caption, and narration surfaces; keyboard focus moves through birds and interactions.

- **Account export.** NOT RECOVERABLE FROM PLAN

- **Soft-delete with 30-day recovery.** NOT RECOVERABLE FROM PLAN

- **Aggregate operational telemetry only.** The rationale is operational monitoring within a privacy boundary: page-load timing, first-bird-render, frame timing, audio-context errors, and tick latency are collected without per-bird or per-account interaction history.

### Architecture

- **Auth Service.** NOT RECOVERABLE FROM PLAN

- **Simulation Service as canonical writer.** The rationale is state integrity: it owns personality vectors, mood states, presence accumulation, drift computation, and is "the only service that writes personality state."

- **Client Application served from CDN.** The rationale is fast render and a clean split: the client renders, synthesizes audio, handles input, writes events, and benefits from CDN delivery for the "Time to first bird visible <500ms" target.

- **Client/server split.** The split exists so the server owns durable state while the client owns "visual animation, audio synthesis, user input handling, local interpolation between snapshots."

- **Render pipeline boundary.** The plan wants tiny snapshots and instant life: the client receives "kilobytes," renders immediately, interpolates between snapshots, and shows birds "mid-action."

### Data Model

- **Bird stable UUID.** NOT RECOVERABLE FROM PLAN

- **User-assigned mutable bird name.** NOT RECOVERABLE FROM PLAN

- **Personality vector persistence.** The rationale is durable, incremental server truth: vectors are stored server-side, "never recomputed from event logs," never sent as absolutes, and drift is an "additive delta from previous tick."

- **Personality vector non-exposure.** The plan gives an explicit affective rationale: exposing numerical values would make the product "collapse into stat management," so values are never exposed "not even in debug views."

- **Presence event with all three conditions.** The rationale is to prevent "Presence signal inflation" and avoid a tab-open being counted as care; presence requires visibility, focus, and recent input simultaneously.

- **Append-only interaction event log.** The rationale is ordered, conflict-free simulation: clients write events, the tick consumes them in order, and there is no concurrent write path to personality.

- **Notebook entry object.** The rationale is sparse observation, not constant reporting: entries are generated by the Simulation Service on "notable moments" every few days per active aviary.

- **Visit invitation object.** The rationale is controlled visiting: email is encrypted at rest, links are one-time, and status supports pending, accepted, revoked, and expired states.

### API Surface

- **Aviary snapshot API.** The rationale is small canonical state transfer: clients fetch all birds, moods, positions, weather, and time-of-day as a kilobyte-scale snapshot and interpolate locally.

- **Aviary events catch-up API.** The rationale is stated directly as "catch-up after offline."

- **Interaction events API.** The rationale is write-only client input into the simulation: events are appended and the Simulation Service consumes them.

- **Notebook entries API.** The client rationale is read-only access to generated notebook prose.

- **Visit APIs.** The rationale is invitation control and read-only visitor access: create a one-time link, list outstanding invitations and recent visits, revoke invitations, and fetch a visitor snapshot by token.

- **Account APIs.** NOT RECOVERABLE FROM PLAN

### Simulation Engine Design

- **Tick cadence of about once per minute.** The rationale is a slow, lightweight simulation that can continue on the server and avoid backlog; server-load mitigation calls the cadence "slow (~1/min)" and computation "lightweight."

- **Drift inputs.** Presence-time is primary because regular presence is the care signal; listen-in duration, offers, and settle gestures are secondary interaction signals.

- **Monotonic toward expressive traits.** The rationale is anti-punitive care: traits only increase, and "neglect results in ambient quietude, not regression."

- **Low-pass filter.** The rationale is to keep drift from becoming immediately game-like: "no single session shifts traits visibly."

- **Mood transitions.** The rationale is continuity and contextual behavior: moods reset on a daily-ish cadence but are modulated by recent events, local time, weather, and personality, and they persist across sessions.

- **Call grammar runtime.** The rationale is procedural identity: the server sends motif descriptors for pitch contours, rhythm, ranges, and ornamentation, while the client synthesizes actual audio.

- **Bird-to-bird interaction.** The rationale is emergent aviary behavior: calls can prompt responses, wary mood can spread, and chorus events emerge when multiple high-vocal-frequency birds call in the same window.

### Sync Model

- **Single canonical source.** The rationale is to remove merging and last-write-wins failure modes: the Simulation Service writes personality, all clients read the same snapshot, and clients submit events rather than state.

- **Client snapshot pull strategy.** The rationale is recovery from visible/resume states without chatty sync: pull on visibility change, long render-frame gap, and low-frequency keepalive while visible.

- **Conflict prevention.** The rationale is direct: only server writes personality, clients write append-only events, and ordered tick processing means "no concurrent writes to personality."

### Frontend Rendering Pipeline

- **Single horizontal scene.** The rationale is a contained, glanceable aviary: it fits one viewport with "no panning/scrolling."

- **Three perch zones and mood/personality perch choice.** The rationale is expressive proximity without user control: front, middle, and back signal relation to the viewer, and perch choice is driven by mood and personality.

- **Responsive behavior.** The rationale is preserving the aviary composition: the scene compresses or widens while keeping all birds visible, with birds never cropped or offscreen.

- **Idle micro-motion.** The rationale is mood-readable life at rest: wary birds scan from farther back, content birds preen, curious birds tilt toward sounds, and drowsy birds sit low and fluffed.

- **Day/night rendering.** The rationale is time-shaped ambience: palette and calls shift from morning to midday to evening to night, with nightjar-like species as the exception to quietness.

- **Ambient leaf/feather drift.** The rationale is visual ornament without simulation burden: it is "pure rendering," client-side, idle cadence, and has "no per-leaf state."

- **Loading state.** The rationale is continuity rather than app loading chrome: a slow snapshot shows a "quiet field" with faint motion, not a spinner, and the first frame has ambient drift already running.

- **Listen-in and settle transitions.** The rationale is softness and reversibility: listen-in uses slow mix ramps, settle shifts evening light over seconds, and undo within five seconds reverses it.

- **Top bar fade and no chrome inside the aviary scene.** NOT RECOVERABLE FROM PLAN

- **Reduced-motion rendering details.** The rationale is an intentionally designed alternate surface: cross-fades replace frame animation and flight paths, while ambient color shifts remain slowed.

- **Empty-aviary state.** NOT RECOVERABLE FROM PLAN

### Audio Pipeline

- **Procedural call synthesis.** The rationale is variation, identity, and budget: per-species motifs and personality-shaped timing produce recognizable bird signatures without recorded audio.

- **Chorus mixing.** The rationale is to make simultaneous calls feel real: two birds produce "real chorus," with per-call variation, not stacked loops or phase-canceling artifacts.

- **Listen-in mix.** The rationale is focus without silence: the selected bird rises gradually, other birds fall to ambient, and no hard cuts occur.

- **WebAudio fallback.** The rationale is graceful degradation: if WebAudio is unavailable, the product uses "graceful silence with captions on by default."

- **No recorded audio fallback.** The rationale is explicit: procedural synthesis is mandatory because recorded audio at the needed variation exceeds the bundle budget and risks loop artifacts.

- **Bundle budget alignment and species pool.** The rationale is keeping initial JavaScript under 2MB while still supporting about six species with distinct call signatures.

### Accessibility Surfaces

- **Screen-reader narration cadence and voice.** The rationale is observation-style parity with the visual surface: narration updates at idle cadence, faster on user events, and never exposes mechanical perch or mood labels.

- **Call captions placement and grammar.** The rationale is to attach generated call descriptions to the calling bird in the same naturalist voice, fading with the call.

- **Reduced-motion accessibility surface.** The rationale is preventing accessibility regression: reduced motion keeps calls, drift, notebook noticing, and designed cross-fade aesthetics.

- **WCAG AA contrast.** The rationale is readable user-copy across bright and dim states; the plan clarifies that contrast applies to chrome because the aviary scene itself has no user copy.

- **Keyboard navigation.** The rationale is complete non-pointer operation: top bar traversal, bird focus, listen-in, exit, offer, settle, and visible high-contrast focus indicators.

### Performance Budgets and Observability

- **Initial JS bundle under 2MB.** The rationale is to force procedural audio, generated visual assets, and code-splitting of secondary surfaces.

- **Time to first bird visible under 500ms.** The rationale is product feel: "Above 500ms: user notices load; below: aviary feels already running."

- **60fps idle motion on a 5-year-old mid-range laptop.** The rationale is sustained ambience for a "30-minute session," not just first-minute smoothness.

- **No memory growth over 30 minutes.** The rationale is long-session stability, enforced by reused audio buffers, bounded workers/audio contexts, no retained notebook references, and CI testing.

- **Performance observability.** The rationale is operational reliability within the privacy boundary: synthetic checks and aggregate-only RUM watch load, render, audio errors, and tick latency.

- **Tick latency p99 alarm at 5 seconds.** The rationale is an explicit error budget for server tick health.

- **Browser support: last two major versions of Chrome, Safari, Firefox, Edge.** NOT RECOVERABLE FROM PLAN

- **Unsupported browser requirements surface.** NOT RECOVERABLE FROM PLAN

### Rollout

- **v1 two-bird adoption and user naming.** The rationale is first encounter: "birds that arrived, not birds user chose," with the system selecting species rather than offering a catalog pick.

- **Visit invitations off by default.** The rationale follows the opt-in visit philosophy: sharing exists, but it starts quiet and disabled.

- **Bird count ramp by aviary age.** The rationale is to avoid tying growth to "visit count," "interaction score," or "paid tier" and keep pacing slow over months and a year.

- **Day-one instrumentation.** The rationale is operational readiness at launch: aggregate telemetry, tick latency, synthetic checks, bundle tracking, and first-bird-render timing start immediately.

- **Deliberately not instrumented behavior.** The rationale is preventing future exposure of behavior: no per-bird interaction state, no population drift analytics, no user-facing visit-frequency metrics, and no pipeline that could later expose per-account bird behavior.

### Risks

- **Drift calibration mitigation.** The rationale is balancing two named failures: too fast creates "Tamagotchi feel," too slow creates "screensaver feel."

- **Presence signal enforcement.** The rationale is to prevent corrupted drift from tab-open counting; all three presence signals are enforced server-side.

- **Sync correctness mitigation.** The rationale is avoiding "silent data loss" from client-submitted personality absolute values and last-write-wins behavior.

- **Audio uncanniness mitigation.** The rationale is preserving the spell: looped audio or chorus artifacts would break the experience, so procedural synthesis is mandatory.

- **Accessibility regression mitigation.** The rationale is that accessibility features are part of the product, not post-ship fallback work.

- **Personality vector exposure mitigation.** The rationale is the "affective contract": numerical values would turn birds into stat management.

- **Server tick load mitigation.** The rationale is preventing backlog through slow cadence, lightweight computation, and p99 tick-latency alarms.

- **Visit feature scope mitigation.** The rationale is keeping visits read-only and ambient; scope refusal is "structural, not policy."

- **Magic-link security mitigation.** The rationale is protection from token replay and email enumeration through invalidated links and rate limiting.

- **Synthetic performance regression mitigation.** The rationale is guarding bundle size, render performance, and memory growth with CI, synthetic monitoring, and alarms.
