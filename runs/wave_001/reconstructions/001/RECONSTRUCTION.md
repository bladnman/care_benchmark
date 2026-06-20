## System-level intent

- **Respect the non-goals as product boundaries.** The plan repeatedly protects Pocket Aviary from becoming a game, Tamagotchi, social network, or notification product. This shows up in Scope as "no achievements, streaks, scores," "birds don't die or get hungry," "no profiles, follows, public feeds," "no notifications or push communications," and again in Rollout as "No marketing push or social promotion."

- **Make the aviary feel alive over weeks, not session by session.** The Drift Calibration Risk names the core promise as "feels alive over weeks." The drift function is calibrated for "Visible drift after ~3 weeks" and "measurable instruments after ~1 week," with "Low-pass filter" behavior and monotonic movement "toward expressive." The birds-per-aviary ramp also says "Automatic progression to maintain relationship depth."

- **Keep personality and aviary state server-authored and canonical.** Architecture says the "Server owns" personality state, event log, and canonical aviary state. Sync Model says the server is the "single source of truth," personality vectors are "only written by server simulation tick," and Conflict Prevention rejects "last-write-wins" in favor of "additive server-authored deltas only."

- **Treat accessibility as a designed surface, not a fallback.** The Render Pipeline Boundary calls reduced-motion mode "a first-class designed surface." Accessibility Regression Risk names the mitigation as "Accessibility included from day one; designed surfaces instead of fallbacks." Reduced-motion keeps the "Core simulation unchanged," and call captions/screen-reader narration are integrated with the same simulation state and grammar.

- **Use naturalist prose as a product voice.** The Field Notebook is "Auto-generated naturalist observations." Screen-reader narration is "Running naturalist prose" and uses the "Same voice as field notebook (continuous prose, not bullet points)." Call captioning uses "Short prose descriptions" and a "naturalist voice consistent with rest of product."

- **Use procedural, personality- and mood-shaped generation.** The plan specifies "procedural calls," "Procedural bird animation generation," "WebAudio API for real-time call synthesis (no recorded audio)," "Procedural motif combination," and personality/mood-shaped timing, pitch, idle motion, and call variation.

- **Keep the first experience fast and stable on modest devices.** Performance intent appears as "Critical rendering path optimized for <500ms first bird visibility," "Initial JS bundle <2MB," "60fps idle motion on 5-year-old laptops," and "No memory growth over 30-minute sessions." Performance Budget Risk says exceeding thresholds causes "Poor user experience, slow adoption."

- **Keep telemetry aggregate-only and privacy-bounded.** The data model uses "synthetic UUID account IDs" and "encrypted email." Observability is "Aggregate-only Real User Monitoring (no per-bird data)," "Session duration histograms (anonymized)," and "No per-account interaction history in telemetry." Privacy Compliance Risk identifies leakage of "Per-bird interaction data" as a "User trust violation."

- **Allow bounded, opt-in visiting without social network surfaces.** The plan includes a "Visit-invite feature" where a host can invite friends for "read-only viewing," while Scope excludes "profiles, follows, public feeds." Rollout keeps visit invitations "opt-in only (disabled by default)."

## Per-feature whys

### Scope / What's in v1

- **Single-user accounts with email magic-link authentication:** NOT RECOVERABLE FROM PLAN

- **Two starter birds per aviary, capped at seven birds total:** The plan ties the cap to "Bird recognizability maintained up to 7 birds in chorus." The starter count and later ramp are also connected to "Automatic progression to maintain relationship depth."

- **Core bird engine with personality vectors, mood system, and procedural calls:** The articulated why is the "feels alive over weeks" promise: personality drift, persistent mood, and procedural calls let birds change through accumulated inputs and express state through motion and call characteristics.

- **Multi-device sync through server-side simulation tick:** The plan's why is sync correctness: avoid "Multi-device sessions corrupt personality state" and inconsistent birds by using "Server-only personality writing," shared server state, and canonical simulation.

- **Return-greeting:** NOT RECOVERABLE FROM PLAN

- **Listen-in:** The plan explains listen-in as focused attention without hard separation: the focused bird ramps up, other birds "drop to ambient but remain audible," and there is "No hard channel switching."

- **Offer:** Offers are an input to the drift function and event log; the plan says the server updates personality vectors "based on accumulated inputs."

- **Settle:** Settle gestures are another drift input and mood/user interaction input, supporting personality drift and mood transitions from accumulated interaction events.

- **Field notebook:** The plan positions it as the read-only naturalist prose surface: "Auto-generated naturalist observations" and the voice anchor for screen-reader narration.

- **Screen-reader narration:** The why is accessibility through state-aware naturalist prose: it is generated from "server-side state," updated every "30-60 seconds," and gives "User-initiated events" priority.

- **Reduced-motion mode:** The why is accessible motion without changing the product's core behavior: motion becomes cross-fades and simplified ambient effects while "Core simulation (birds, calls, drift) unchanged."

- **Call captioning:** The why is making procedural calls accessible: captions are "generated from procedural grammar," synchronized with calls, and available even when WebAudio becomes "complete silence with captions."

- **Visit-invite feature:** The why is bounded, host-controlled viewing: a host can invite friends for "read-only viewing," invitations are "opt-in only," and the product still avoids public feeds and social network surfaces.

- **Account export functionality:** NOT RECOVERABLE FROM PLAN

### Architecture

- **Single-page webapp with WebAudio client-side synthesis:** The plan's why is to keep rendering and real-time call synthesis on the client, with WebAudio used for "real-time call generation."

- **Server-side simulation tick:** The why is persistence and continuity independent of client presence: it runs "regardless of client presence," carries mood across sessions, and writes canonical aviary state.

- **Relational database with synthetic UUID account IDs:** The synthetic UUID aspect is tied to privacy boundaries and internal references; Privacy Compliance Risk names "synthetic UUIDs for all internal references" as mitigation.

- **REST APIs for state snapshots and event submission:** NOT RECOVERABLE FROM PLAN

- **CDN for static assets:** NOT RECOVERABLE FROM PLAN

- **Service workers for offline capabilities:** The why is stated directly as "offline capabilities."

- **Server owns personality state, event log, and canonical aviary state:** The why is to enforce canonical state, conflict prevention, and server-authored personality changes.

- **Client owns rendering pipeline, audio synthesis, and user input handling:** The plan separates these from server-owned personality/canonical state so the client can handle rendering, real-time audio, and input while the server controls state.

- **State snapshots on change:** The plan ties snapshots to multi-device sync and shared server state, so clients receive the current canonical aviary state rather than authoring their own.

- **Critical rendering path optimized for under 500ms first bird visibility:** The why is first-experience performance; the same target appears as a success and risk budget for avoiding poor user experience.

- **Procedural bird animation generation client-side:** NOT RECOVERABLE FROM PLAN

- **WebAudio real-time call synthesis with no recorded audio:** NOT RECOVERABLE FROM PLAN

- **Reduced-motion mode as first-class designed surface:** The why is accessibility quality: the risk mitigation explicitly says "designed surfaces instead of fallbacks."

### Data Model and Relationships

- **Account with synthetic UUID, encrypted email, and single aviary reference:** Synthetic UUIDs and encrypted email support the strict privacy boundary; the single aviary reference supports the one Account to one Aviary model.

- **Bird with stable ID, name, species reference, personality vector, and current mood:** NOT RECOVERABLE FROM PLAN

- **Personality Vector:** The why is to make individual birds respond and express differences; its traits shape vocal frequency, plumage saturation, curiosity, timing, and drift.

- **Mood:** The why is session-persistent expression: mood "carries across sessions" and is visible through idle motion and call characteristics.

- **Event Log:** The why is deterministic, conflict-safe simulation: it is "append-only," processed "in chronological order," and "never overwritten."

- **Visit host-to-visitor invitation relationships:** The why is controlled read-only visiting rather than open social surfaces.

- **One Account to One Aviary:** NOT RECOVERABLE FROM PLAN

- **Birds interact with each other through ambient calls:** NOT RECOVERABLE FROM PLAN

- **Birds interact with user through personality drift and mood transitions:** The why is the long-horizon alive feeling: user interactions accumulate into drift and mood rather than scores, hunger, or other game mechanics.

### API Surface and Access Control

- **Current state snapshot API:** The why is client synchronization to canonical state.

- **User interaction event submission API:** The why is to feed offers, listen-in, settle gestures, and other interactions into the append-only event log and server simulation.

- **Aviary state export API:** NOT RECOVERABLE FROM PLAN

- **Account deletion API:** NOT RECOVERABLE FROM PLAN

- **Create visit invitation API:** The why is host-created, opt-in visiting.

- **Check visit status API:** NOT RECOVERABLE FROM PLAN

- **Revoke visit invitation API:** The why is host control over invitation relationships.

- **Magic-link request API:** NOT RECOVERABLE FROM PLAN

- **Magic-link verify and session establishment API:** NOT RECOVERABLE FROM PLAN

- **List active sessions and revoke session APIs:** The why is explicit in Conflict Prevention: "Revocable session tokens prevent unauthorized writes."

- **Valid session token requirement:** The why is server-side access control over all state operations.

- **Server-side account ownership enforcement:** The why is preventing non-owners from reading or writing account state.

- **Visit invitations with one-time links and email verification:** The why is access control for visitor entry.

### Simulation Engine Design

- **Server-side tick:** The why is continuous state evolution: it runs about once per minute, processes events chronologically, updates personality and mood, and writes canonical state.

- **Drift function:** The why is calibrated, slow personality change: "Visible drift after ~3 weeks" and "measurable instruments after ~1 week."

- **Presence-time, listen-in, offers, and settle gestures as drift inputs:** The why is to base personality updates on accumulated user presence and interactions instead of explicit game mechanics.

- **Monotonic drift toward expressive:** The plan states the direction as "traits only move up, never down," giving drift an expressive direction.

- **Low-pass filter over accumulated event inputs:** The why is calibration: avoid birds changing too fast while still becoming measurable and visible over the target windows.

- **Call Grammar Runtime:** The why is procedural bird calls shaped by species, personality, and mood rather than fixed recordings.

- **Mood Transitions:** The why is responsiveness and persistence: moods are triggered by time, interactions, ambient events, and personality traits, carry across sessions, and are expressed in idle motion and calls.

### Sync Model

- **Canonical State:** The why is a "single source of truth for aviary state" shared across devices.

- **Personality vectors only written by server simulation tick:** The why is conflict prevention and protection from client-authored personality corruption.

- **No last-write-wins; additive server-authored deltas only:** The why is to prevent multi-device conflicts from corrupting personality state.

- **Events processed in chronological order:** The why is ordered, deterministic processing from the append-only event log.

- **Pull-based client synchronization:** NOT RECOVERABLE FROM PLAN

- **Efficient snapshot format:** The why is bandwidth efficiency: snapshots should be "kilobytes, not megabytes."

- **Interpolation between ticks:** The why is "smooth bird motion between ticks."

- **Presence-aware state pull:** NOT RECOVERABLE FROM PLAN

### Frontend Rendering Pipeline

- **Single horizontal scene optimized for one screen:** The why is a one-screen aviary composition.

- **Three perch zones:** The why is "proximity signaling."

- **Local timezone-anchored day/night cycles and rare subtle weather:** NOT RECOVERABLE FROM PLAN

- **Idle micro-motion:** The why is visible bird life and mood expression: preening, scanning, head-tilting, and mood-shaped behavior.

- **Ambient leaf and feather drift plus subtle parallax:** NOT RECOVERABLE FROM PLAN

- **Listen-In Mix:** The why is focused listening while preserving the chorus: gradual level transitions, ambient audibility, smooth ramping, and "No hard channel switching."

- **Reduced-motion cross-fades and simplified ambient elements:** The why is accessibility while leaving birds, calls, and drift unchanged.

- **Frontend performance optimizations:** The why is to meet the plan's first-bird, bundle-size, frame-rate, memory, and procedural-audio budgets.

### Audio Pipeline

- **Procedural Call Synthesis:** The why is real-time, species-, personality-, and mood-shaped calls.

- **Chorus Mixing:** The why is per-bird balance and proximity-sensitive listening, with high vocal-frequency birds joining more readily.

- **Listen-In Mix Decay:** The why is a complete smooth transition back to chorus level "over a few seconds."

- **Audio accessibility integration:** The why is access to call meaning even without WebAudio: screen readers, text captions, and silence with captions if needed.

### Accessibility Surfaces

- **Screen-Reader Narration:** The why is continuous access to state in "naturalist prose," generated from server state and prioritized around user-initiated events.

- **Keyboard Navigation:** The why is "Full keyboard accessibility for all interactive surfaces."

### Performance Budgets and Observability

- **Bundle size, code-splitting, and tree-shaking:** The why is protecting the critical path that includes bird rendering, audio engine, and state management.

- **Rendering performance budgets:** The why is first-bird visibility, 60fps idle motion, tab-hidden skipping, and memory usage caps.

- **Audio performance budgets:** The why is audio stability and memory control: user-gesture context creation, buffer reuse, tick timeout protection, and audio-context failure monitoring.

- **Aggregate-only Real User Monitoring:** The why is observability without per-bird data.

- **Synthetic performance tests from multiple geographic locations:** NOT RECOVERABLE FROM PLAN

- **Anonymized session duration histograms:** The why is metrics collection without per-account interaction history.

- **Error budgets:** The why is controlling bundle size, render timing, and audio stability.

- **Bird recognizability target up to seven birds in chorus:** The why is preserving recognizability at the maximum aviary size.

- **Drift calibration target:** The why is ensuring drift is measurable after one week and visible after three weeks.

- **Presence accuracy target:** The why is reliable detection of "proper user engagement scenarios."

- **Accessibility compliance target:** The why is keeping accessibility "maintained throughout development."

### Rollout Strategy

- **Magic-link authentication only at launch:** NOT RECOVERABLE FROM PLAN

- **Two starter birds at launch:** The why is connected to the later birds-per-aviary ramp and maintaining relationship depth.

- **Visit invitations opt-in only and disabled by default:** The why is bounded visiting rather than default social exposure.

- **Limited user feedback collection with session metrics only:** The why is consistency with aggregate-only observability and no per-account interaction history.

- **No marketing push or social promotion:** NOT RECOVERABLE FROM PLAN

- **Birds-per-aviary ramp by aviary age:** The why is "Automatic progression to maintain relationship depth."

- **Species variety increases with aviary age:** The why is part of the same automatic progression for relationship depth.

- **Session start/end tracking:** NOT RECOVERABLE FROM PLAN

- **Presence event recording and accuracy monitoring:** The why is the "Presence accuracy >95%" calibration target.

- **Audio pipeline error rates:** The why is audio stability observability.

- **Render frame timing histograms:** The why is performance observability against render timing budgets.

- **Bird interaction frequency:** The why is to observe "offer, listen-in usage patterns."

- **User retention success metric:** The why is measuring whether sessions show visible drift after three weeks.

- **Performance success metrics:** The why is measuring bundle size, time-to-first-bird, and audio continuity.

- **Accessibility success metrics:** The why is measuring screen reader task completion rates.

- **Drift calibration success metrics:** The why is measuring personality vector changes in instrumented accounts.
