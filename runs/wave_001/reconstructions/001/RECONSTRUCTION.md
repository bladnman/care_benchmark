## System-level intent

- **Slow-timescale bird evolution with low-key user presence.** The plan states this at the top as "slow-timescale bird evolution and low-key user presence." It appears again in server ticks at "~1 minute," drift calibration at "1 week" and "3 weeks," and rollout pacing through "Age-based bird unlocking."
- **Presence should change birds without punishing absence.** The strongest expression is "Monotonic Expressivity": traits "only increment (or stay flat)" and "Neglect results in zero delta, causing the bird to feel 'ambient' but not 'wary.'" This also connects to "No Custodial Mechanics" and the absence of death, starvation, distress, and happiness meters.
- **No gamification, scoring, or social pressure.** The plan explicitly says "No Gamification" with "No streaks, achievements, levels, or scores," and "No Social Network" with no "profiles, discovery feeds, chat, or public ranking." This supports the "low-key" product posture.
- **Server-owned canonical state, client-owned high-fidelity feeling.** The architecture says the "Server" is the "Canonical state owner" and the "Client" is a "State consumer and renderer." The server runs tick, event logs, auth, snapshots, and naturalist prose; the client handles "high-fidelity animation," "procedural audio synthesis," interpolation, and interaction event emission.
- **Naturalist prose rather than state logging.** The plan repeatedly uses "naturalist prose" for the notebook and accessibility narration. The accessibility risk says narration must remain "naturalist" and not "devolve into state-logging."
- **Bounded visiting, not co-presence or a network.** Social is limited to "Visit invitations" with "one-time link, read-only, no co-presence." The visit route accepts "no interaction events," and the non-goal rules out profiles, feeds, chat, and ranking.
- **Accessibility surfaces should carry the same world, not a separate substitute.** Screen-reader narration uses "naturalist prose," reduced motion uses "slow cross-fades between key poses," and call captions are "generated from the same motif metadata used by the audio engine."

## Per-feature whys

### 1. Scope and Constraints

- **2 starter birds per account:** NOT RECOVERABLE FROM PLAN
- **7 total bird cap:** NOT RECOVERABLE FROM PLAN
- **Aviary-age-based bird growth:** The plan ties bird growth to time by saying the cap is "based on aviary age" and later gives "Bird 3 at 1 month, Bird 4 at 3 months." This supports the "slow-timescale" intent.
- **Server-side simulation ticks:** The plan makes the server the "Canonical state owner" and has tick input combine "Last state + interaction events since last tick + wall clock time." The why is consistent, server-side evolution from state, events, and time.
- **Personality drift, monotonic:** The rationale is explicit: presence and interaction create expressivity while neglect causes "zero delta," so a bird feels "ambient" but not "wary." The risk language adds that drift must feel "earned" but not "fast."
- **Mood transitions:** Moods exist to respond to "time-of-day bias, weather effects, and recent interaction 'nudges.'" They connect the scene, weather, time, and interactions to bird behavior.
- **Presence accounting:** Presence is recorded because accumulated `PresenceLog` duration is an input to drift. Rollout instrumentation also monitors "presence accounting accuracy."
- **Return-greeting:** NOT RECOVERABLE FROM PLAN
- **Listen-in:** The audio section explains that listen-in "focuses one gain node while attenuating others (never 0%)." The why is focused listening without erasing the chorus.
- **Offer:** Offers are interaction events that feed the tick as "recent interaction 'nudges'" and "specific interaction weights." The plan recovers that offers affect moods and drift.
- **Seed, song fragment, still pool as offer choices:** NOT RECOVERABLE FROM PLAN
- **Settle:** The plan describes Settle as a "soft session-end." The recoverable why is to end a session softly within the low-key presence model.
- **Single horizontal scene:** NOT RECOVERABLE FROM PLAN
- **Three perch zones:** NOT RECOVERABLE FROM PLAN
- **Day/night cycle:** Time of day is a mood input through "time-of-day bias" and appears in narration such as "morning light." `time_offset` also appears in the aviary model.
- **Ambient weather:** Weather is stored as `weather_state` and used as "weather effects" in mood updates, so it gives the scene and simulation an ambient condition.
- **Field notebook:** The tick can "Generate Observations" and create naturalist prose entries. The API fetches "paginated observation history."
- **Email magic-link auth:** NOT RECOVERABLE FROM PLAN
- **Multi-device sync:** The plan's stated risk is "Preventing 'stale state' glitches when switching devices." Server-side canonical state and snapshots support that.
- **Server-side state snapshots:** Snapshots are committed as canonical state and sent to the client as JSON containing positions, moods, and active transitions. The client consumes snapshots for rendering and interpolation.
- **Visit invitations:** The plan limits visits to "one-time" URLs, "read-only" access, "no co-presence," and "no interaction events accepted." This supports visiting without becoming a social network.
- **Screen-reader narration:** The rationale is accessibility through "naturalist prose" that describes the scene, for example a bird on a perch "preening softly in the morning light."
- **Reduced-motion mode:** The rationale is to "Replace all frame-based animations with slow cross-fades between key poses."
- **Call captioning:** Captions are generated from "the same motif metadata used by the audio engine," so the audio information has a caption surface.
- **Web-only / no native apps:** The product is described as "browser-based," and the non-goal says "No Native Apps: Web-only."
- **No gamification:** The plan excludes "streaks, achievements, levels, or scores," preserving the low-key and non-scored experience.
- **No custodial mechanics:** The plan says birds "do not die, starve, or show distress" and there are "No happiness meters." This aligns with drift that avoids birds becoming "wary" from neglect.
- **No social network:** The plan excludes "profiles, discovery feeds, chat, or public ranking," keeping social limited to read-only invitations.

### 2. Architecture

- **Server as canonical state owner:** The server owns truth because it "Runs the simulation engine," "processes event logs," "manages auth," and generates naturalist prose for notebook and narration.
- **Client as state consumer and renderer:** The client exists to render the server state with "high-fidelity animation," "procedural audio synthesis," and interaction event emission.
- **Snapshot handoff from server to client:** The client receives JSON with "bird positions, moods, and active transitions," so rendering is driven by canonical state.
- **Client interpolation at 60fps:** Interpolation between snapshots is used to sustain smooth rendering at the stated runtime target of "Consistent 60fps."
- **Client-side procedural micro-motion and call variation:** These are computed from "snapshot parameters," letting the client add variation while staying tied to server state.

### 3. Data Model

- **Account:** NOT RECOVERABLE FROM PLAN
- **Aviary `weather_state` and `time_offset`:** These support ambient weather and day/night behavior, both of which feed mood and scene narration.
- **Bird model:** `personality_vector`, `current_mood`, `current_perch`, and `drift_history` support the plan's drift, mood transitions, perches, and state snapshots.
- **PersonalityVector traits:** `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, and `curiosity` give concrete dimensions for behavioral, vocal, and visual expressivity.
- **PresenceLog:** It records session duration so drift can be based on "accumulated `PresenceLog` duration."
- **InteractionEvent:** It records offers, listen-in, settle, and presence-ping events so the tick can process "interaction events since last tick."
- **NotebookEntry:** It persists `prose_content` and timestamp for the field notebook's observation history.

### 4. Simulation Engine Design

- **Tick frequency of approximately 1 minute:** NOT RECOVERABLE FROM PLAN
- **Update Moods:** Mood updates exist to apply "time-of-day bias, weather effects, and recent interaction 'nudges.'"
- **Apply Drift:** Drift updates personality vectors from accumulated presence and "specific interaction weights."
- **Generate Observations:** Observation generation occasionally triggers naturalist prose for the Field Notebook.
- **Save Snapshot:** Saving the snapshot commits the "new canonical state to the database."
- **Drift calibration target:** Numerical drift at "1 week" and visible behavioral change at "3 weeks" are used so drift feels "earned" but not "fast."

### 5. API Surface

- **`POST /auth/magic-link`:** NOT RECOVERABLE FROM PLAN
- **`GET /aviary/state`:** The endpoint pulls the "current snapshot" for the client renderer.
- **`POST /aviary/event`:** The endpoint appends interactions and presence-pings so the server tick can process events.
- **`GET /notebook`:** The endpoint fetches "paginated observation history."
- **`POST /invite`:** The endpoint creates a "one-time visit URL for a specific email."
- **`GET /visit/:token`:** The route allows public read-only visiting with "no auth required" and "no interaction events accepted."
- **`DELETE /invite/:id`:** The endpoint revokes access.

### 6. Frontend Rendering & Audio

- **Parallax background foliage, middle-plane perches/birds, foreground framing elements:** NOT RECOVERABLE FROM PLAN
- **Mood-keyed idle poses:** Poses such as "preening" and "scanning" make bird mood visible in the rendered scene.
- **Soft fly-ins for new birds:** NOT RECOVERABLE FROM PLAN
- **Initial snapshot delivered with HTML:** This exists "to achieve <500ms time-to-first-bird."
- **WebAudio procedural call generation:** Calls are shaped by "species-specific motifs and current bird mood," so audio expresses species and mood.
- **Chorus mixing:** "Staggered start times" create a chorus, and listen-in focuses one gain node while attenuating others "never 0%."

### 7. Accessibility Surfaces

- **Narration engine:** It periodically generates a "naturalist description of the scene," keeping accessibility narration in the same prose voice as the notebook.
- **Keyboard navigation:** It provides "Full Tab/Arrow/Enter/Escape support with visible focus rings."

### 8. Performance Budgets

- **Initial payload under 2MB gzipped:** The recoverable why is that it is part of the stated "Performance Budgets."
- **TTFB under 500ms on 4G:** This supports the loading goal of "<500ms time-to-first-bird."
- **Runtime at consistent 60fps with zero memory leak over 30min sessions:** This protects long, smooth sessions in the scene.
- **Observability for render frame drops and snapshot latency:** These metrics track rendering health and snapshot freshness, with snapshot latency budgeted at "p99 < 5s."

### 9. Rollout & Risks

- **Instrumentation for drift velocity and presence accounting accuracy:** This exists to monitor whether drift and presence are calibrated correctly.
- **Drift calibration risk:** The rationale is explicitly "Ensuring drift feels 'earned' but not 'fast.'"
- **Sync correctness risk:** The rationale is preventing "'stale state' glitches when switching devices."
- **Audio uncanniness risk:** The rationale is that procedural calls must avoid "phase-canceling or repetitive artifacts."
- **Accessibility regression risk:** The rationale is ensuring prose narration remains "naturalist" and does not become "state-logging."
