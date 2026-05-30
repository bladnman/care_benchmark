## System-level intent

1. **Ambient aliveness without game pressure**

The plan repeatedly frames Pocket Aviary as a quiet, ambient experience rather than a game loop. This shows up in the "Core Experience" as a "single-scene horizontal aviary," in "Social" as "ambient Visits," in the "Loading Sequence" as the "'Quiet Field' approach" that should "Avoid spinners," and in the exclusions: "No scores, streaks, levels, badges, or achievements" and "No mortality, hunger, or distress; no negative drift on neglect." The risk language also defines the desired feel by avoiding both "'Tamagotchi' (too fast)" and "'Screensaver' (too slow)."

2. **Server-authoritative canonical state with stateless clients**

The architecture centers on a canonical server state. The client "acts as a stateless viewer of the server-side canonical state," while the server is the "authoritative source of truth." This principle appears again in the API surface: "Clients do not write state; they append events," and in the sync mitigation: "Server-authoritative model; clients are strictly read-only for personality state."

3. **Slow, expressive change through presence and interactions**

The simulation is meant to create gradual behavioral change rather than immediate rewards or punishments. The tick "Calculates Personality Drift" through "low-pass filters" based on "presence-time and interaction-weighting," with drift "monotonic toward expressive." Presence is carefully gated by visible, focused, recently active use, and calibration targets "measurable drift" after "1 week" and "visible behavioral change" after "~3 weeks."

4. **Naturalist voice as a primary surface**

The product voice is naturalist prose. The plan names "Field Notebook" as an "Auto-generated naturalist observation log," describes notebook generation as synthesizing "naturalist prose," and defines accessibility narration as "naturalist prose describing the aviary state." The risk mitigation explicitly rejects treating accessibility as a fallback by saying to "Design accessibility as a primary, naturalist-voiced surface from day one."

5. **Graceful, lightweight, web-first delivery**

The system is constrained to V1 "Web-only," with concrete budgets: "<2MB JS bundle," "<500ms time-to-first-bird," and "60fps idle motion." The frontend should render a "soft-colored, lightly-animated scene immediately," use reduced motion with "slow cross-fades," and fall back to "Graceful Silence" if WebAudio fails.

6. **Constrained, opt-in social presence**

Social intent is deliberately narrow. "Visits" are "Opt-in, read-only, ambient" and are created "via email invitation." Non-goals exclude "profiles, follows, public discovery, or chat," and "Visitors are read-only and do not interact or influence the host's birds."

7. **Felt aliveness through procedural behavior, mood, and variation**

The plan tries to make birds feel alive through "Personality vectors," "fast-timescale Mood," "procedural call synthesis," "mood-shaped micro-motions," and "timing/pitch variation." Rollout includes beta verification of "felt aliveness," while risks call out "Audio Uncanniness" and mitigate it with "High-quality motif libraries" and variation.

## Per-feature whys

### 1. Scope & Non-Goals

**Core Experience:** NOT RECOVERABLE FROM PLAN

**Bird Engine:** The plan's rationale is to support aliveness through individual differences and fast-changing state: "Personality vectors," "fast-timescale Mood system," and "procedural call synthesis" are later connected to "felt aliveness," "mood-shaped micro-motions," and call variation.

**Simulation:** The rationale is canonical continuity. The server-side tick advances "canonical state," consumes events, updates drift and mood, generates notebook entries, and persists state so clients remain viewers rather than independent simulations.

**Interactions:** NOT RECOVERABLE FROM PLAN

**Accounts & Sync:** The rationale is "multi-device sync via canonical server state" and "synthetic UUID-based identity." Account fields also carry "Accessibility, Social Opt-in, Notification Opt-in" settings, but the plan gives no further why for magic-link authentication.

**Social:** The rationale is ambient, constrained visiting: "Opt-in, read-only, ambient 'Visits'" allow invitation by email while avoiding "Social Network Surfaces" and "Co-presence."

**Field Notebook:** The rationale is to produce a naturalist observation surface from state changes. The tick "Generates Notebook Entries" based on "significant events or time" and synthesizes "naturalist prose based on recent state changes."

**Accessibility:** The rationale is primary access in the same product voice, not a fallback. The plan calls for "Screen-reader narration (naturalist prose)," "reduced-motion mode," "call captioning," and "WCAG AA contrast," with the mitigation to "Design accessibility as a primary, naturalist-voiced surface from day one."

**Performance:** The rationale is immediate, smooth access to the aviary: "<500ms time-to-first-bird," "60fps idle motion," and small bundle constraints support the web-first experience.

**Gamification exclusion:** The rationale is to keep the product from becoming a game loop: "No scores, streaks, levels, badges, or achievements."

**Tamagotchi Mechanics exclusion:** The rationale is to avoid punishment or distress: "No mortality, hunger, or distress; no negative drift on neglect."

**Native Apps exclusion:** NOT RECOVERABLE FROM PLAN

**Social Network Surfaces exclusion:** The rationale is to keep social presence constrained: "No profiles, follows, public discovery, or chat."

**Co-presence exclusion:** The rationale is that "Visitors are read-only and do not interact or influence the host's birds."

### 2. Architecture & Data Model

**Client frontend:** The rationale is to render and interact locally while staying a "stateless viewer of the server-side canonical state." It handles scene rendering, WebAudio synthesis, interactions, and local presence signals without owning truth.

**Server backend:** The rationale is authoritative continuity. It is the "authoritative source of truth," hosts the simulation engine, manages the "append-only interaction event log," persists canonical state, and runs the tick.

**Account:** NOT RECOVERABLE FROM PLAN

**Aviary:** NOT RECOVERABLE FROM PLAN

**Bird:** The rationale is stable individuality and current behavioral state: "UUID (Stable ID)," "Species ID," name, personality, mood, perch, and last interaction timestamp. The plan does not articulate a deeper rationale for each field.

**Interaction Event Log:** The rationale is append-only state change input. The server consumes pending interaction events "in chronological order," and clients "do not write state; they append events."

**Field Notebook Entry:** The rationale is to persist generated "Naturalist" prose observations with timestamped entries.

**Visit:** The rationale is to support invited, revocable, read-only social access through host account, visitor email, status, and last access timestamp.

### 3. Simulation Engine Design

**Server-side tick:** The rationale is to periodically advance canonical aviary state at an "approx. 60s" cadence by consuming events, calculating drift, updating mood, generating notebook entries, and persisting state.

**Consumes the Event Log:** The rationale is ordered processing of all pending account interactions "in chronological order."

**Calculates Personality Drift:** The rationale is gradual expressive change from "presence-time and interaction-weighting," using "low-pass filters" with drift "monotonic toward expressive."

**Updates Mood:** The rationale is to make mood responsive to "Recent interactions," "Time of day," "Ambient events," and "Personality constraints."

**Generates Notebook Entries:** The rationale is to create periodic naturalist prose from "significant events or time" and "recent state changes."

**Persists State:** The rationale is to write the "updated canonical aviary state to the database."

**Presence Signal:** The rationale is to count presence only when the user is visibly and actively present: `visibilityState === 'visible'`, focus is true, and recent pointer or keyboard activity is detected.

**Drift Function:** The rationale is calibrated pacing: telemetry should show "measurable drift" after "1 week," while users should see "visible behavioral change" after "~3 weeks."

### 4. API Surface & Sync Model

**Client-to-server interaction events:** The rationale is that "Clients do not write state; they append events."

**Presence endpoint:** The rationale is to send "Presence heartbeats" into the event flow for presence accounting and drift.

**Interaction endpoint:** The rationale is to append offer, listen-in, and settle events without direct client state writes.

**Snapshot endpoint:** The rationale is to return "the current canonical state" for client consumption: positions, moods, call timing, notebook, and related state.

**Snapshot pull triggers:** The rationale is to resync on "visibility change," "long render-frame gaps," or "low-frequency keepalive."

**Interpolation:** The rationale is "smooth motion/audio transitions" between snapshot state `N` and `N+1`.

**Visit Flow:** The rationale is to allow email-invited, authenticated "read-only session" access where the server filters `GET /snapshot` "for read-only mode."

### 5. Frontend Rendering & Audio Pipelines

**Scene Composition:** The rationale is a "single horizontal viewport" with "foreground/middle/background" layers and "subtle parallax." The plan does not articulate a further why.

**Idle Motion:** The rationale is mood expression through "procedural, mood-shaped micro-motions" such as "preening, scanning, head-tilting."

**Reduced-Motion Mode:** The rationale is accessible motion reduction by replacing "frame-by-frame animation with slow cross-fades between still poses."

**Loading Sequence:** The rationale is the "'Quiet Field' approach": "Avoid spinners" and render a soft scene immediately while fetching the first snapshot.

**Procedural Call Synthesis:** The rationale is to avoid audio files and synthesize motifs from "Vocal Frequency" and "Mood." Risk mitigation further emphasizes avoiding robotic sound through motif quality and "timing/pitch variation."

**The Chorus Mixer:** The rationale is to preserve ambient chorus while highlighting focus: all birds remain at "low, baseline volume," while "Listen-in" ramps up the focused bird and reduces others.

**Graceful Silence fallback:** The rationale is to handle WebAudio failure by entering "Graceful Silence" with "call captions enabled by default."

### 6. Accessibility & Performance

**Naturalist Narration:** The rationale is screen-reader access in product voice: an ARIA-live or similar mechanism reads "periodic, naturalist prose describing the aviary state."

**Call Captioning:** The rationale is to provide "short, mood-aware prose captions" near the bird.

**Keyboard/Focus:** The rationale is full navigation through the top bar and bird-to-bird focus using "standard keyboard patterns."

**Bundle Size:** The rationale is the performance budget of initial JS "< 2MB (gzipped)." The plan gives no further why.

**Time-to-First-Bird:** The rationale is fast first render: "< 500ms on mid-tier mobile/4G."

**Runtime:** The rationale is "Consistent 60fps idle motion on 5-year-old laptops."

**Memory:** The rationale is "Zero growth over a 30-minute session" verified through CI.

### 7. Rollout & Risks

**Alpha:** The rationale is "Internal testing of simulation accuracy and drift calibration."

**Beta:** The rationale is to "verify multi-device sync and 'felt aliveness.'"

**V1 Launch:** The rationale is to "Ramp up birds-per-aviary (starting at 2)" while monitoring "performance/stability."

**Drift Calibration risk:** The rationale is to avoid the feel becoming "'Tamagotchi' (too fast)" or "'Screensaver' (too slow)," mitigated by "Strict telemetry monitoring of personality deltas."

**Sync Correctness risk:** The rationale is to prevent "divergent simulations," mitigated by the "Server-authoritative model" and read-only client personality state.

**Audio Uncanniness risk:** The rationale is to avoid procedural calls "sounding robotic," mitigated through "High-quality motif libraries" and "timing/pitch variation."

**Accessibility Regressions risk:** The rationale is to avoid "treating accessibility as a fallback," mitigated by designing accessibility as "a primary, naturalist-voiced surface from day one."
