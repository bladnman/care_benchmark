## System-level intent

- **The aviary should avoid gamification, obligation, and punishment.** This intent shows up in "Out of Scope (v1)" as "no streaks, achievements, levels, scores, badges, green-dot calendars," in "Non-Goals Honored" as "no negative drift on neglect" and "Settle and tab-close are equivalent," in "How We Ramp Birds Per Aviary" as "not visit count, not interaction score, not paid tier," and in rollout as "No engagement dashboards; no DAU/MAU targets."

- **The product should feel alive without becoming a Tamagotchi model.** The plan names the tick as "the heartbeat of the product's 'feels alive' claim," but also excludes "hunger/decay meters," "visible distress," and "death." Drift is calibrated so "no single session shifts a trait visibly," with visible drift only after "~3 weeks."

- **One canonical, persistent aviary is the core identity model.** The plan repeats that "Server is the only source of truth," there is "one canonical aviary state per account," mood "persists across sessions," and bird IDs are "stable, never recycled." The risk section says a reset or regeneration would make "the entire premise of weeks-long drift evaporates retroactively."

- **Multi-device sync should be prevented from diverging by architecture, not reconciled after the fact.** The plan says clients "render snapshots only," there is "No client-side simulation tick," personality drift is "additive, server-authored delta," and "No Last-Write-Wins for Personality." The risk mitigation says to "Enforce at the architecture layer, not policy."

- **Presence must mean active attention, not an open tab.** Presence is the conjunction of `visibilityState === 'visible'`, window focus, and recent pointer/key activity. The plan warns that a laxer definition like "tab is open" would "silently inflate drift signal" and let a laptop left open all night drift like active watching.

- **Change should be slow, expressive, and positive-only.** Drift is a "low-pass filter" where presence-time is "dominant," traits "move up on positive presence," and "never move down on neglect." Neglect produces "ambient quietness," not negative trait movement.

- **The voice should be observational, naturalist, and sparse.** Field notebook entries are "naturalist voice, lowercase, present-tense," "rare," and "not per-session." Screen-reader narration is "written as observations, not state transitions." Call captions use "the same voice as the field notebook." The risk section warns that "stock event-log entries" would break this voice.

- **Procedural variation should make birds recognizable as living individuals, not loops.** The plan specifies a "procedural bird engine," "procedural call synthesis," motif libraries, and "real chorus, not stacked loops." The audio risk says loop-based or low-variation audio would be "immediately identifiable as dead software."

- **Accessibility is a designed surface from day one.** The plan says reduced motion is "Not 'animations off'" and "Its own designed surface, not a fallback." The accessibility risk says these surfaces are "a designed experience, not a checklist" and must ship "with the rest of the product, not after."

- **Privacy boundaries are defined at metric design time.** Observability is "Aggregate-only Real User Monitoring," with "None of this includes per-bird state or per-account interaction history." The plan also says the telemetry pipeline "never touches" the per-account simulation database and rollout is based on "operational health metrics, not engagement metrics."

## Per-feature whys

### 1. Scope

- **Two starter birds per new aviary:** NOT RECOVERABLE FROM PLAN

- **Cap of seven birds total:** The audio system requires individual recognition: "Call signature must remain individually recognizable up to 7 birds (the cap)." The cap also bounds chorus mixing and scene complexity.

- **Species drawn from a pool of ~6 coherent species:** NOT RECOVERABLE FROM PLAN

- **Single-user accounts, one canonical aviary per account:** The plan's sync rationale is that "both devices pulling the same record = same aviary on both devices." One canonical aviary supports identity continuity and avoids client-to-client sync.

- **Email magic link auth:** NOT RECOVERABLE FROM PLAN

- **Multi-device sync via server-side canonical state:** The plan says clients "render snapshots only" because if a client ticks locally, "multi-device sync becomes divergent simulations" and "the product's central conceit (aviary continues without viewer) fails."

- **Procedural bird engine with personality vectors, mood state machine, and monotonic drift:** The why is expressive long-term change without visible numbers or penalties. Personality vectors are "never exposed numerically," drift is "monotonic toward expressive only," and mood shapes visible behavior like wary birds perching back or content birds preening.

- **Procedural call synthesis, chorus mixing, and listen-in mix rebalance:** The plan wants calls that are synthesized fresh, individually recognizable, and not "dead software." Chorus is "real-time mix, not stacked loops"; listen-in focuses one bird while others drop to "ambient level" and "never go fully silent."

- **Return-greeting:** NOT RECOVERABLE FROM PLAN

- **Listen-in interaction:** The plan gives it a strong per-bird role: "Listen-in -> strong signal for the listened-to bird (social warmth, vocal frequency)." The audio mix also makes listening focused through gradual gain changes.

- **Offer interaction: seed, song fragment, still pool:** Offers create "small drift toward curiosity (offered item) and boldness (offering near a bird)." An accepted offer gives a "content nudge."

- **Settle gesture:** Settle is included to quiet the mood without teaching penalty or reward. It is a "small mood-quieting signal; no directional drift," and "Settle and tab-close are equivalent at the engine level."

- **Field notebook:** The notebook exists to preserve the "naturalist voice" in sparse observations rather than event logs. Entries are "rare" and require a "prose-quality bar" because stock entries like "session started at 7:43" would break the voice.

- **Single horizontal visual scene with three perch zones:** The plan uses perch zone as a derived display signal, for example "perch position choice," and mood expression: "A wary bird perches further back." Specific rationale for exactly three zones is NOT RECOVERABLE FROM PLAN

- **Day/night cycle anchored to user's local time:** Time of day modulates mood: birds become "drowsy near dusk" and "alert in early morning." Local time anchors that modulation to the user's aviary experience.

- **Ambient weather:** Weather feeds mood and audio behavior. Rain "dampens vocal frequency," and ambient events can shift birds, such as an "alarm call" moving nearby birds toward wary.

- **Ambient micro-motion:** Micro-motion is how the aviary stays visibly alive at idle. The plan names "preening, scanning, head-tilting, body-shuffle" as "continuous, mood-shaped" behaviors.

- **Top bar with account/settings, accessibility, notebook, and offer affordance:** NOT RECOVERABLE FROM PLAN

- **Top bar fading nearly transparent after cursor stillness:** NOT RECOVERABLE FROM PLAN

- **Visit invitations:** The plan frames visits as opt-in and controlled: "visits off by default," the visitor gets a "read-only ambient view," and read-only visit snapshots have "no interaction event submission." This keeps visits out of social-network mechanics.

- **Accessibility surfaces:** The why is equal product experience, not retrofit compliance. Screen-reader narration follows the slow visual rhythm because "high-frequency narration would overwhelm the screen reader queue"; reduced motion is "its own designed surface"; captions keep audio information in the same naturalist voice.

- **Performance budgets:** The budgets support immediacy and aliveness: "<500ms time-to-first-bird," "60fps idle," and "no memory growth over 30 min." The loading state avoids a spinner so the first experience is "quiet field" and birds appear "mid-action."

### 2. Architecture

- **Server as the only writer of personality state:** This prevents divergent simulations and preserves the product premise. Clients "never mutate personality directly," and the sync risk says direct client writes would make divergence possible.

- **Clients owning rendering and audio synthesis only:** Clients pull snapshots, interpolate, and synthesize calls so they can present smooth motion and real-time audio while the server remains canonical.

- **Append-only interaction event log:** The event log gives the tick ordered inputs for server-authored drift. Events are "immutable once written" and consumed "in order," preventing concurrent writers to personality state.

- **No client-side simulation tick:** The plan's explicit why is that local ticking would make "multi-device sync" into "divergent simulations" and break the idea that the "aviary continues without viewer."

- **Server-side tick about once per minute:** The tick is "the heartbeat" of the feels-alive claim. It runs whether or not a client is connected, reads events, updates personality and moods, advances timers, and writes canonical state.

### 3. Data Model

- **Stable bird IDs:** IDs are "stable, never recycled" so identity continuity survives long-term drift and migrations. The risk section says reset or regeneration would make weeks-long drift evaporate.

- **Renameable bird names:** NOT RECOVERABLE FROM PLAN

- **Raw personality vectors not sent to client:** The why is to keep numbers hidden and avoid turning personality into a manipulable score. The client receives "only derived display signals."

- **Mood persisting across sessions:** Persistence supports continuity: "No reset on tab open" and at session start the bird "resumes whatever mood it had at last tick."

- **Presence events recorded only when all three conditions hold:** The reason is to avoid "presence signal inflation" from tab-open behavior and make presence reflect active viewing.

- **Notebook entries read-only to user:** NOT RECOVERABLE FROM PLAN

- **Encrypted account email and synthetic account UUID:** The plan says account ID is "synthetic, never email" and email is "stored once, encrypted." The privacy rationale is implicit in this data shape and the broader metric/privacy boundary.

- **Soft delete with 30-day window:** NOT RECOVERABLE FROM PLAN

### 4. API Surface

- **State snapshot API:** The snapshot is small, "kilobytes," and clients interpolate between snapshots for "smooth per-bird motion." It supports rendering without client-side authority over simulation.

- **Batched interaction event submission:** Batching lets the client "buffers and flushes on visibility change / session end" while preserving append-only event order for the tick.

- **Visit invitation one-time link and read-only snapshot stream:** The read-only stream prevents visitor interactions from entering the event log; host revocation and off-by-default visits preserve control.

- **Account export endpoint:** NOT RECOVERABLE FROM PLAN

- **Session list and revocation endpoints:** NOT RECOVERABLE FROM PLAN

### 5. Simulation Engine Design

- **Presence-time as dominant drift input:** The plan makes presence dominant so regular, active attention slowly shapes expression without clicker-game mechanics.

- **Low-pass drift function:** The low-pass filter ensures "no single session shifts a trait visibly" and calibrates between "Tamagotchi model" and "screensaver."

- **No downward drift on neglect:** The why is to avoid punishment. Neglect produces "ambient quietness," while traits "never move down."

- **Mood transitions from interactions, time of day, ambient events, and personality:** The rationale is expressive behavior: offers nudge content, dusk makes drowsy, rain dampens calls, and high-boldness birds are less likely to become wary.

- **Sparse notebook generation by simulation or narration service:** Sparsity preserves the slow naturalist voice: entries are "~one every few days," "not per-session," and must pass voice review.

- **Species motif libraries:** Motif libraries create varied procedural calls from "small sets of melodic fragments" while keeping call signatures recognizable.

### 6. Sync Model

- **Single canonical aviary:** The why is straightforward sameness across devices: "Both devices pulling the same record = same aviary on both devices."

- **No last-write-wins for personality:** Personality is not a client-submitted absolute value because "client never sends 'set boldness to 0.62'"; it sends observed events like listening for three minutes. This makes divergent simulations "impossible."

- **Serialized single-writer tick per account:** The plan uses single-writer serialization and append-only events so there are "no concurrent writers to personality state."

### 7. Frontend Rendering Pipeline

- **Hybrid Canvas plus SVG/CSS rendering:** NOT RECOVERABLE FROM PLAN

- **Mood-shaped idle micro-motion:** The rationale is to display mood through behavior: wary birds scan, content birds preen, curious birds tilt toward sounds, and drowsy birds sit low.

- **State interpolation:** The plan explicitly rejects teleporting: birds move "smoothly" between perches across snapshots.

- **Reduced-motion mode:** The why is to provide a designed alternative, "not 'animations off.'" Cross-fades preserve the aviary's visual language while reducing motion.

- **Quiet field loading state:** The plan avoids a spinner and wake-up animation so the aviary "appears with motion already in progress."

- **Responsive scene:** The rationale is that "all birds" remain "visible at all times," with narrow viewports compressing without cropping and wide viewports gaining space.

### 8. Audio Pipeline

- **WebAudio procedural synthesis:** The plan uses WebAudio so calls are "never looped audio" and are "synthesized fresh each time."

- **Chorus mixing:** Real-time mix avoids "phase-canceling artifacts from stacked loops" and supports multiple birds calling simultaneously as a "real chorus."

- **Listen-in mix:** The why is focus without erasing the rest of the aviary: the focused bird rises, others drop gradually, and "never go fully silent."

- **Graceful silence with captions if WebAudio is unavailable:** The fallback keeps the product usable without recorded audio. Captions are on by default, and there is "no recorded-audio fallback path."

- **Call captions:** Captions translate procedural calls into "short naturalist-prose descriptions" near the bird and use the field notebook voice.

### 9. Accessibility Surfaces

- **Screen-reader narration:** Narration gives the same aviary state as the visual surface in "lowercase, present-tense, specific" naturalist prose. Its slow cadence avoids overwhelming the queue and matches "the slow rhythm of the visual aviary."

- **Keyboard navigation:** The plan makes the main interactions reachable by keyboard: moving focus between birds, triggering listen-in, exiting with Escape, opening offers, and reaching settle from the top bar.

- **Focus indicators:** The rationale is visibility across "both bright and dim aviary states" with a "soft, high-contrast outline."

- **WCAG AA contrast:** The plan requires all user-copy text to pass contrast across top bar labels, settings, account surfaces, errors, captions, and visual narration.

### 10. Performance Budgets and Observability

- **Synthetic performance checks and aggregate RUM:** These measure operational health: load timings, first-bird-render timings, frame timings, audio-context errors, and tick latency. The metric boundary keeps out "per-bird state or per-account interaction history."

- **Unsupported-browser surface:** NOT RECOVERABLE FROM PLAN

### 11. Rollout

- **Soft launch to small cohort:** The plan ramps based on "operational health metrics, not engagement metrics," which aligns with no engagement dashboards.

- **New birds based on aviary age:** The explicit why is that the mechanic "refuses to teach the user that more attention earns more birds."

### 12. Risks

- **Drift calibration target:** The why is to avoid both extremes: too fast becomes "Tamagotchi model," too slow becomes "screensaver." The named target makes tuning testable.

- **Audio variation and chorus quality:** The plan treats looped or low-variation audio as a product risk because it is "immediately identifiable as dead software."

- **Accessibility planned from day one:** The why is that retrofitted reduced motion or generic narration would tell users "the product wasn't designed for them."

- **Presence-signal testing:** The conjunction must be tested rigorously because inflated presence would make passive open tabs count like active watching.

- **Field notebook voice review:** Voice review exists because generic event-log prose would "break the naturalist voice across the entire product."
