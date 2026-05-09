## System-level intent

- **Long-term relationships through observation and idle attention.** This appears in `Scope and Strategy`: Pocket Aviary v1 is "focused on long-term relationships with procedural birds through observation and idle attention." It also shows up in the Field Notebook's "auto-generated naturalist prose" and in slow adoption pacing by "aviary age."
- **Persistent aliveness independent of the user's presence.** The Architecture section says the system uses a "thin-client, thick-server pattern to ensure aliveness independent of the user's presence." This is reinforced by the "server-side authoritative tick" and the Tick's "Idle Actions" for the next snapshot.
- **No punishment for absence.** The plan prohibits "No Custodial Pressure" with "no hunger, death, or distress meters" and "no punishment for absence." The Drift Function repeats this with "neglect results in ambient stagnation (no negative drift)."
- **No gamification.** The plan states an "absolute prohibition on streaks, levels, achievements, or scores." This product boundary aligns with the focus on observation rather than scoring.
- **Monotonic expressive drift over weeks.** The Core Engine includes "monotonic expressive drift," and the Drift Function says trait values move up on positive input. The Calibration note sets a long perception horizon: "Measurable in logs after 1 week; perceptible by users after ~3 weeks."
- **Ambient context shapes expression.** The Frontend includes "day/night and ambient weather cycles," while the Tick updates `mood_state` based on "time-of-day (Local TZ), ambient weather, and recent interaction density."
- **A focused but continuous aviary atmosphere.** "Listen-in" gives one bird focus while lowering others to "20% volume (never 0%)," keeping the wider aviary audible. The product also stays bounded by "One aviary per account" and a max of 7 birds.
- **Naturalist voice across product and accessibility.** The Field Notebook uses "auto-generated naturalist prose," screen-reader narration uses "naturalist prose descriptions," and call captions use "the naturalist voice."
- **Canonical server state with event-only client writes.** Sync and Privacy says "Clients are read-only for state; write-only for events," so there are no "Last-Write-Wins" conflicts on personality. The Persistence Layer has a "Canonical State Store" plus an "Append-only" Event Log.
- **Privacy and telemetry restraint.** The Privacy Mandate requires "PII Isolation," says email is "never used as an internal key," and forbids "aggregate training," "third-party sharing," and "population averages." Observability adds "Explicit exclusion of per-bird state from telemetry pipelines."
- **Accessibility and performance as designed surfaces.** Accessibility lists "Narration," "Reduced Motion," and "Captions" as "Designed Surfaces." Performance budgets define "<2MB gzipped initial payload," "<500ms on 4G," "60fps on 5-year-old hardware," and "Zero memory growth over 30-min sessions."

## Per-feature whys

### Scope and Strategy

- **Core Engine:** The personality vector model, fast-timescale mood transitions, and monotonic expressive drift support "long-term relationships with procedural birds" by giving birds durable traits, faster current moods, and slow expressive change over time.
- **Simulation / server-side authoritative tick:** The plan's rationale is "persistent aliveness" and, later, "aliveness independent of the user's presence."
- **Single-screen responsive horizontal scene:** NOT RECOVERABLE FROM PLAN
- **Three-perch depth model:** The plan uses `current_perch` values `(front, middle, back)` and "perch changes" as Idle Actions, tying the feature to bird positions, snapshots, and the "3-plane parallax" scene.
- **Day/night and ambient weather cycles:** These cycles matter because `mood_state` is updated from "time-of-day (Local TZ), ambient weather, and recent interaction density," and `server_time` aligns local day/night cycles.
- **Client-side WebAudio procedural call synthesis using motif grammars:** The rationale is expressive variation by species and state: motifs are "combined based on bird species" and modulated by `vocal_frequency` and `mood`; the risk section warns variation must be high enough to avoid "uncanny" audio fatigue.
- **Return-greeting:** NOT RECOVERABLE FROM PLAN
- **Listen-in (mix focus):** The feature focuses attention on a target bird using a "2-second gain ramp" while lowering other birds to "20% volume (never 0%)," so focus does not fully erase the rest of the aviary.
- **Offers (seed/song/pool):** Offers are part of the user interaction history in the Event Log. The Tick fetches events since the last tick and updates `personality_vector` through "Additive Drifts," so offers are recoverable as positive interaction input.
- **Settle gesture:** NOT RECOVERABLE FROM PLAN
- **Field Notebook:** The Notebook exists to persist "auto-generated naturalist prose" and "generated observation logs," matching the observation-centered product voice.
- **Magic-link auth:** NOT RECOVERABLE FROM PLAN
- **Multi-device sync:** The plan's rationale is conflict avoidance and cycle alignment: clients are "read-only for state; write-only for events," there are no "Last-Write-Wins" conflicts on personality, and snapshots include `server_time`.
- **One-time read-only visit invitations:** The feature is bounded by the Social and Non-Goals sections: visits are "one-time" and "read-only," while the plan excludes "public discovery, follows, profiles, or chat."
- **Screen-reader narration:** Narration provides an `aria-live` region with "naturalist prose descriptions of the scene every 45s."
- **Reduced-motion mode:** Reduced Motion replaces "path-based flight" with "1s opacity cross-fades" and "jittery micro-motion" with "slow pose-morphing."
- **Call captioning:** Captions provide on-screen text for procedural calls, using examples like "*a low trill*" in the "naturalist voice."

### Out-of-Scope / Non-Goals

- **No Native Apps / Web-only v1:** NOT RECOVERABLE FROM PLAN
- **No Gamification:** The plan makes this an "absolute prohibition" against "streaks, levels, achievements, or scores," preserving the non-scored observation model.
- **No Custodial Pressure:** The rationale is explicitly non-punitive care: "No hunger, death, or distress meters; no punishment for absence."
- **No Social Network:** The feature boundary prevents "public discovery, follows, profiles, or chat," keeping social access limited to the one-time read-only visit model.
- **No Multi-Aviary / one aviary per account:** The plan states "One aviary per account," keeping the product bounded around a single long-lived aviary.

### Architecture

- **Thin-client, thick-server pattern:** The stated rationale is to "ensure aliveness independent of the user's presence."
- **React/TypeScript client technology choice:** NOT RECOVERABLE FROM PLAN
- **State-driven rendering engine:** The client "interpolates between server snapshots" and handles local rendering around canonical server state.
- **WebAudio synthesis on the client:** The client handles "WebAudio synthesis," matching the Audio section's client-side procedural call synthesis.
- **Local presence detection:** The client handles "local presence detection," and the data model defines `presence_ping` from `visibilityState: visible`, window focus, and recent input.
- **Simulation Service:** The service periodically processes the Tick, computes "personality drift and mood transitions," and writes canonical state.
- **API Gateway:** NOT RECOVERABLE FROM PLAN
- **Canonical State Store:** This store holds "Current aviary state (bird positions, moods, vectors)," supporting canonical state delivery and sync.
- **Event Log:** The Append-only log persists "User interaction history (offers, presence, listen-ins)" so the Tick can fetch all events since the last tick.
- **Notebook Store:** This store makes generated observation logs "Persistent."
- **Layered Canvas or SVG-based rendering:** The Render Pipeline uses this for "Scene Composition" with "3-plane parallax."
- **Snapshot interpolation buffer:** The client keeps a small buffer of snapshots to "smoothly animate bird transitions between perches."
- **Micro-motion:** Procedural animations such as "preening" and "head-tilting" run locally and are "modulated by the current mood state."

### Data Model

- **Synthetic UUID account id:** The rationale is "PII isolation"; email is not used as an internal key.
- **Encrypted email:** The plan stores `email_encrypted` as "Encrypted email for auth," tying it to authentication while preserving the privacy mandate.
- **Aviary age anchor / `created_at`:** `created_at` is an "Aviary age anchor for bird adoption pacing."
- **Immutable `bird_id`:** NOT RECOVERABLE FROM PLAN
- **`species_id`:** This links to the "species pool (visuals/motifs)," connecting species identity to rendering and call motifs.
- **User-defined bird name:** NOT RECOVERABLE FROM PLAN
- **`personality_vector`:** The vector provides durable traits (`boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`) that drift through positive input and modulate expression such as calls and plumage.
- **`mood_state`:** Mood is the current state (`wary`, `content`, `curious`, `drowsy`, `alert`) updated from local time, weather, and recent interaction density, and it modulates micro-motion and call grammar.
- **`current_perch`:** The perch field supports the three-perch depth model and Idle Actions such as "perch changes."
- **`presence_ping`:** The plan defines presence as a conjunction of `visibilityState: visible`, window focus, and recent input, supporting local presence detection and event ingestion.
- **`event_entry`:** Event entries carry type, timestamp, and metadata such as `OFFER_ACCEPTED`, giving the Tick interaction history for drift and mood updates.

### Simulation Engine Design

- **60-second Tick interval:** NOT RECOVERABLE FROM PLAN
- **Fetch all events since the last tick:** Events are the input for updating the account's simulation state each Tick.
- **Low-pass filter / Additive Drifts:** The plan uses this to update `personality_vector` gradually, with measurable logs after 1 week and user-perceptible effects after about 3 weeks.
- **Mood update from local time, weather, and recent interaction density:** This keeps `mood_state` tied to ambient context and recent interaction rather than only fixed personality.
- **Idle Actions:** Perch changes and call triggers are computed for the next snapshot, supporting persistent aliveness between user interactions.
- **Monotonicity:** Trait values "move up on positive input," while neglect creates "ambient stagnation (no negative drift)," matching the no-punishment rule.
- **Calibration:** The one-week log and three-week user perception targets define how slow the drift should feel and why CI "dry-run" simulation testing is needed.

### Audio Pipeline

- **Motif-based oscillators/samplers:** Motifs provide procedural calls without a recorded loop fallback.
- **Call grammar:** Motifs are combined by bird species and modulated in pitch/timing by `vocal_frequency` and `mood`, so calls express both species and state.
- **Listen-in mixing:** The target bird gains focus over 2 seconds while the rest of the aviary remains audible at 20% volume.
- **Silence plus visual captions fallback:** If WebAudio is unavailable, the fallback is "Silence + visual captions," preserving accessibility without introducing "recorded loop" audio.

### Sync and Privacy

- **Read-only clients for state / write-only clients for events:** This prevents client writes to personality and avoids "Last-Write-Wins" conflicts.
- **Snapshot `server_time`:** Server time aligns "local day/night cycles" across devices.
- **PII Isolation:** Email is "never used as an internal key."
- **Data Boundary:** Interaction logs are "strictly for the user's simulation," with no "aggregate training," "third-party sharing," or "population averages."

### Accessibility

- **`aria-live` narration cadence:** The plan updates scene narration every 45 seconds with naturalist prose descriptions.
- **Reduced-motion cross-fade rendering:** Path-based flight becomes "1s opacity cross-fades," and jittery micro-motion becomes slow pose-morphing.
- **Procedural call captions:** Captions render call descriptions as text in naturalist voice.

### Performance and Observability

- **Bundle size budget:** The "<2MB gzipped initial payload" budget protects initial load.
- **Time to First Bird:** The "<500ms on 4G" target centers the first visible bird as the key first-load experience.
- **Runtime budget:** "60fps on 5-year-old hardware" and "Zero memory growth over 30-min sessions" support stable observation sessions.
- **Simulation Tick p99 latency:** Instrumenting p99 latency makes the authoritative Tick observable.
- **Aggregate RUM:** RUM tracks "Time-to-first-bird" and "render-frame drops" without per-bird state.
- **Per-bird telemetry exclusion:** Excluding per-bird state from telemetry enforces the Privacy Mandate.

### Rollout and Risks

- **Phase 1: 2 starter birds, max 7:** NOT RECOVERABLE FROM PLAN
- **New bird adoption slots unlocked by aviary age:** The pacing uses aviary age milestones such as "Month 1, Month 3, Year 1," reinforcing long-term use.
- **Dry-run simulation testing in CI:** Needed because the Drift Function may feel "static or too rapid."
- **Robust client-side smoothing:** Needed because Snapshot interpolation might "jitter on poor connections."
- **Diverse motif library per species:** Needed because procedural motifs may become "uncanny" if variation is too low.
- **Template-based naturalist prose with high variance:** Needed because automation of naturalist prose for narration is complex and risks accessibility regressions.
