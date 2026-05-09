## System-level intent

1. **"Notice, never announce" as the product philosophy.** The plan states that V1 "focuses on the core observational relationship between a user and their birds" and is "strictly adhering to the principle of 'notice, never announce.'" This shows up again in the hidden "personality vector," low-frequency "Naturalist prose," and "Captions" written in a "naturalist voice."

2. **A quiet, non-coercive relationship rather than a game loop.** The "Non-Goals (Absolute)" reject "gamification (streaks, badges, quests)," "Tamagotchi-style neglect penalties," and "notifications." The "Drift Function" reinforces this with traits that "move up with presence, never down with neglect," while the "Drift Calibration" risk names the danger of a "gamified feel."

3. **Single-user continuity with limited, opt-in visitation.** The plan repeatedly frames the product as "single-user": "browser-based, single-user virtual aviary," "Multi-device sync (single-user)," and "no social network discovery/profiles." The "read-only visit invitation feature (opt-in)" fits this as a constrained exception rather than a social graph.

4. **Server-side canonical simulation with thin clients.** The "Client/Server Split" puts "canonical state, personality vectors, mood, drift calculations" on the server and leaves the client to rendering and "event-capturing." The "Sync Model" makes the "Server-side record" the "source of truth" and forbids "client-writes-state-directly."

5. **Ambient performance and procedural expression.** The rendering intent is a "Single horizontal scene with parallax, no-chrome view, procedural animation" and "client-side WebAudio synthesis." The plan pairs this with constraints like "60fps idle motion" and risk mitigation through "Strict adherence to procedural synthesis" so birds feel expressive without loops or chrome.

6. **Accessibility in the same naturalist voice, not as a separate mode of explanation.** The "Accessibility" section specifies "Naturalist running prose," "screen-reader optimized" narration, "Designed cross-fade sequences" for reduced motion, and call descriptions "written in naturalist voice."

## Per-feature whys

### Scope

- **2 to 7 birds per aviary:** NOT RECOVERABLE FROM PLAN

- **Multi-device sync (single-user):** The plan ties this to "single-user" continuity and later explains the mechanism as "Snapshot-based delivery to clients" from "Canonical State" where the server is the "source of truth."

- **Procedural call engine (WebAudio):** The why is to keep bird audio expressive without sounding looped or uncanny. The "Audio Uncanniness" risk says the mitigation is "Strict adherence to procedural synthesis," and the audio engine uses "Procedural grammar" with "chorus mixing."

- **Personality drift and daily mood system:** The plan's rationale is expressive change through observation without punishment. The "Drift Function" is "Monotonic toward expressive"; traits "move up with presence, never down with neglect." The risk is calibration between "gamified feel" and "static feel."

- **Read-only visit invitation feature (opt-in):** The rationale is constrained sharing while preserving "single-user" shape and avoiding "social network discovery/profiles." The plan's own limits are "read-only" and "opt-in."

- **Field notebook (naturalist, auto-generated):** The rationale is low-frequency "Naturalist prose" that fits "notice, never announce." The data model names "Notebook Entry" as "Naturalist prose, generated at low frequency."

- **Reduced-motion mode and call captioning:** The rationale is accessibility without leaving the product voice. Reduced motion uses "Designed cross-fade sequences instead of frame-by-frame animation," and captions are "Procedural call descriptions" in "naturalist voice."

### Architecture

- **Client/Server Split:** The rationale is to keep "canonical state, personality vectors, mood, drift calculations" server-side while the client remains responsible for "rendering and event-capturing." This supports the later "client-never-writes-state architecture."

- **Render Pipeline:** The rationale is the "no-chrome view" of a "Single horizontal scene with parallax," "procedural animation," and "client-side WebAudio synthesis," matching the observational aviary rather than a dashboard or game surface.

- **Auth Service: Magic-link email auth:** NOT RECOVERABLE FROM PLAN

- **Simulation Service: Periodic tick (~1min) processing event logs to update state:** The rationale is to process "event logs" into state changes on a server cadence, including drift and mood updates later described in the "Tick Engine."

- **Sync Service: Snapshot-based delivery to clients:** The rationale is to deliver server state snapshots that clients can render and interpolate, while preserving the server-side "source of truth."

### Data Model

- **Account: Synthetic UUID, magic-link auth, account settings:** NOT RECOVERABLE FROM PLAN

- **Bird stable internal ID:** NOT RECOVERABLE FROM PLAN

- **Bird user-assigned name:** NOT RECOVERABLE FROM PLAN

- **Bird species:** NOT RECOVERABLE FROM PLAN

- **Bird personality vector (hidden) and mood state (fast-timescale):** The rationale is to keep personality observable rather than announced. The vector is explicitly "hidden," while mood is "fast-timescale" and updated by the server simulation.

- **Presence: Conjunction of visibilityState, window focus, pointer/key activity:** The rationale is to ground drift in actual "presence logs." The "Tick Engine" "Computes drift from presence logs."

- **Interaction Log: Append-only events (listen-in, offer, settle):** The rationale is to let clients write interaction events without writing state directly. The "Simulation Service" processes "event logs," and "Updates" say the server applies deltas.

- **Notebook Entry: Naturalist prose, generated at low frequency:** The rationale is to provide "Naturalist prose" in the field-notebook mode without frequent announcements, matching the low-frequency accessibility narration.

### Simulation Engine

- **Tick Engine:** The rationale is server-side periodic processing: at "~1min cadence," it "Computes drift from presence logs" and "handles mood transitions."

- **Drift Function:** The rationale is expressive growth without neglect punishment. It is "Monotonic toward expressive"; traits "move up with presence, never down with neglect."

- **Audio Engine:** The rationale is procedural, mixed calls with an accessibility fallback. It uses "Procedural grammar," "synthesized client-side," "chorus mixing," and a "WebAudio fallback (silence + captions)," with the risk mitigation of avoiding looped or uncanny audio.

### Sync Model

- **Canonical State:** The rationale is explicit: the "Server-side record is the source of truth."

- **Updates:** The rationale is to avoid "Sync/State Divergence." The client writes "interaction events"; the server applies deltas; "No client-to-client or client-writes-state-directly" is allowed.

### Frontend Rendering

- **Client interpolates between snapshots:** The rationale is "smooth motion."

- **Initial JS bundle <2MB:** NOT RECOVERABLE FROM PLAN

- **First bird visible <500ms:** NOT RECOVERABLE FROM PLAN

- **60fps idle motion:** The rationale is tied to smooth ambient rendering and "idle motion," but the plan does not articulate a deeper product rationale beyond the rendering constraint.

- **Memory growth zero over 30min session:** NOT RECOVERABLE FROM PLAN

### Accessibility

- **Narration:** The rationale is screen-reader access in the product voice: "Naturalist running prose," "low-frequency updates," and "screen-reader optimized."

- **Reduced-Motion:** The rationale is to replace "frame-by-frame animation" with "Designed cross-fade sequences."

- **Captions:** The rationale is to describe procedural calls accessibly while keeping "naturalist voice."

- **Navigation:** The rationale is keyboard and visual focus access: "Keyboard-navigable focus" and "high-contrast outlines for birds."
