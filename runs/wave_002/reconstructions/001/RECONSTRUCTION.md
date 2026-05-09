## System-level intent

- **Quiet, non-gamified aviary care with no punitive drift.** This shows up in Scope through "gamification (no streaks, levels, or achievements)" and "Tamagotchi mechanics (no hunger, death, or negative drift)." It also shows up in the Drift Function, where personality drift is "monotonically upwards," and in Risks, where drift that is "too aggressive" would make the product "feel like a Tamagotchi."
- **A single canonical server state, not client-owned state.** The Architecture and Sync Model repeatedly make the server "the authoritative source of truth" and say the server owns "all personality and mood state." The plan explicitly wants to avoid "last-write-wins conflicts" and ensure "deterministic state progression."
- **Semantic state drives rich local experience.** The Render Pipeline Boundary says the server sends "semantic state" such as "current mood" and "perch zone," while the client handles "interpolation, micro-motion, and procedural variation." The same principle appears in Screen-Reader Narration, which is "sourced from semantic aviary state."
- **Slow cadence and separated timescales.** The plan distinguishes "Personality Vector (Slow Timescale)" from "Mood (Fast Timescale)." It uses a server tick "at a slow cadence," screen-reader updates at "slow-cadence," a listen-in "slow, gradual volume ramp," and reduced-motion "slow, calming cross-fades."
- **Naturalist field-observation product voice.** The plan carries a naturalist voice through the "read-only field notebook," "naturalist prose string," "naturalist field notebook entries," and screen-reader prose like "a warbler perches on the high branch..." The vocabulary is observational rather than game-like.
- **Privacy boundary and non-social design.** Scope excludes "social network surfaces" such as "public directories, leaderboards, or co-presence." Account email is stored encrypted "strictly for authentication and account exports." Observability uses "Aggregate telemetry only" and includes "Absolutely no per-bird state or interaction logs."
- **Accessibility as a designed surface, not only a fallback.** The plan includes "Screen-Reader Narration," "Captions," "Full keyboard navigability," "high-contrast focus indicators," "WCAG AA contrast," and a "specifically designed aesthetic" for Reduced-Motion Mode.
- **Performance and observability are product constraints.** The plan sets budgets for "Initial JS bundle," "Time-to-first-bird visible," "60fps idle motion," and "Zero memory growth," then says "RUM and synthetic performance checks" are deployed "from day one."
- **Per-bird audio recognizability and careful audio tuning.** Rollout says the aviary scales to a maximum of 7 "to preserve per-bird audio recognizability." Audio Pipeline and Risks emphasize "species-specific motifs," avoiding "phase-cancellation," and preventing "robotic-sounding motifs or jarring chorus overlaps."

## Per-feature whys

### Scope

- **Web-only virtual aviary:** NOT RECOVERABLE FROM PLAN
- **Starting with exactly 2 birds and ramping to a maximum of 7 based on aviary age:** The Rollout says additional birds are offered "based purely on aviary age" and capped at 7 "to preserve per-bird audio recognizability." Starting with exactly 2 is repeated in V1 Launch.
- **Single-user accounts via magic-link sign-in:** NOT RECOVERABLE FROM PLAN
- **Server-side simulation ticking:** The server is the "authoritative source of truth" and executes a regular background tick to "process events and update canonical state."
- **Multi-device sync via a single canonical server state:** The Sync Model says devices pull the "same canonical snapshot" and the server processes event logs sequentially, ensuring "deterministic state progression regardless of which device submitted the event."
- **Presence accounting:** The Drift Function is based heavily on "presence-time and interactions." The Risks section explains the need for integrity: if presence is "over-counted," population-wide drift will "artificially accelerate."
- **Read-only field notebook:** NOT RECOVERABLE FROM PLAN
- **Procedural audio synthesis:** The Audio Pipeline ties calls to "species-specific motifs" and says timing and pitch are modulated by "personality (vocal frequency) and mood."
- **Screen-reader narration:** The Accessibility Surfaces section says narration is sourced from "semantic aviary state" and delivers "slow-cadence, naturalist prose updates," with user-initiated events receiving "priority queuing."
- **Reduced-motion mode:** The Frontend Rendering Pipeline says this is a "specifically designed aesthetic" that replaces frame-by-frame motion and particle drift with "slow, calming cross-fades between poses."
- **Call captioning:** The plan uses captions as "text descriptions of audio calls" near the calling bird and as the graceful fallback when WebAudio is unavailable.
- **Read-only opt-in visit invitation system:** The plan pairs this with the exclusion of "social network surfaces" and specifies no "public directories, leaderboards, or co-presence." The API includes create, revoke, and visit log surfaces rather than public discovery.

### Architecture

- **Thick client:** The client handles "rendering, WebAudio procedural synthesis, and capturing user interactions," while the server keeps the simulation authoritative.
- **Render pipeline boundary:** The server does not dictate "frame-by-frame layout," only "semantic state"; the client handles visual interpolation, micro-motion, audio motifs, and ambient particle drift.

### Data Model

- **Account with synthetic UUID and encrypted email:** Email is stored encrypted "strictly for authentication and account exports," which matches the plan's privacy boundary.
- **Per-device session tokens:** NOT RECOVERABLE FROM PLAN
- **Bird with stable internal ID, user-assigned name, and species:** NOT RECOVERABLE FROM PLAN
- **Personality Vector:** The plan labels it "Slow Timescale" and connects it to the Drift Function, where personality changes through a low-pass filter based on presence-time and interactions. Audio also uses "vocal frequency."
- **Mood:** The plan labels mood "Fast Timescale" and uses it across the simulation and experience: mood is updated from local time, weather, and recent interactions, and it shapes micro-motion and audio.
- **Interaction Event Log:** It is "append-only" so the server tick can process events sequentially and clients avoid writing absolute personality or mood values.
- **Notebook Entry:** NOT RECOVERABLE FROM PLAN
- **Visit Invitation with token, visitor email, host UUID, and expiration timestamp:** The expiration timestamp and revoke endpoint support an opt-in visit model with bounded access.

### API Surface

- **Auth endpoints `POST /api/auth/magic-link` and `POST /api/auth/verify`:** NOT RECOVERABLE FROM PLAN
- **State endpoint `GET /api/aviary/snapshot`:** It returns bird positions, moods, and recent events as the "lightweight state snapshots" that the client renders and interpolates.
- **Interactions endpoint `POST /api/events`:** It appends presence, listen-in, offers, and settle events to the server log so the tick can process interaction history rather than accepting client-owned state.
- **Invite endpoints `POST /api/invites`, `DELETE /api/invites/:id`, and `GET /api/invites`:** The create, revoke, and visit log endpoints fit the "read-only opt-in visit invitation system" rather than a public social surface.
- **Account export generation:** Account email is stored encrypted for "account exports," making export generation one of the explicit reasons account email is retained.
- **Soft-deletion endpoints:** NOT RECOVERABLE FROM PLAN

### Simulation Engine Design

- **The Tick:** The tick runs server-side at a "slow cadence" and processes the append-only event log, which keeps the simulation tied to canonical server state.
- **Drift Function:** The low-pass filter makes personality drift "monotonically upwards" based on presence-time and interactions, aligning with "no hunger, death, or negative drift." Risks say the calibration must avoid feeling like a Tamagotchi or feeling unresponsive.
- **Presence Definition:** The strict conjunction of visible tab, focus, and recent input protects "Presence Signal Integrity" because over-counting presence would make drift "artificially accelerate."
- **Mood updates from local time, weather, and recent interactions:** The plan makes mood responsive to time, weather, and interaction history rather than static bird state.
- **Procedural notebook generation:** The tick occasionally generates "naturalist field notebook entries" using "procedural prose templates," carrying the field-observation voice into a read-only surface.

### Sync Model

- **Canonical State:** The server owns all personality and mood state; clients never write absolute state values, preventing "last-write-wins conflicts."
- **Passive multi-device sync:** Devices pull the same canonical snapshot, and sequential server tick processing ensures "deterministic state progression."

### Frontend Rendering Pipeline

- **Single responsive horizontal scene with three depth planes and subtle parallax:** NOT RECOVERABLE FROM PLAN
- **Mood-shaped idle micro-motion:** The plan uses continuous animations such as "preening" and "scanning" as "mood-shaped" expression of bird state.
- **No entry/loading animations:** NOT RECOVERABLE FROM PLAN
- **Reduced-motion rendering:** The plan replaces frame-by-frame motion and ambient particle drift with "slow, calming cross-fades between poses."

### Audio Pipeline

- **Procedural call synthesis with species-specific motifs:** The plan makes call timing and pitch reflect the bird's "personality (vocal frequency) and mood."
- **Chorus mixing:** Real-time mixing is specified to prevent "phase-cancellation."
- **Listen-in interaction:** Listen-in uses a "slow, gradual volume ramp" to focus on a specific bird while others fade to "ambient levels."
- **Graceful silence with call captions:** If WebAudio is unavailable, the fallback is silence plus captions, preserving access to call information.

### Accessibility Surfaces

- **Semantic screen-reader narration:** Narration is sourced from semantic aviary state, uses "slow-cadence, naturalist prose updates," and prioritizes user-initiated events.
- **Captions near the calling bird:** Captions are procedural text descriptions of audio calls and appear near the bird, tying audio information to visual location.
- **Keyboard navigation, focus indicators, and WCAG AA text:** The plan places Tab, Arrow keys, Enter, Escape, high-contrast focus indicators, and WCAG AA contrast under Accessibility Surfaces.

### Performance Budgets, Observability, Rollout, and Risks

- **Aggregate telemetry only:** The rationale is the "Privacy Boundary": no per-bird state or interaction logs are included in aggregate telemetry.
- **p99 simulation-tick latency alarm:** The alarm is tied to simulation-tick latency exceeding 5 seconds, keeping the background simulation observable.
- **RUM and synthetic performance checks from day one:** Instrumentation exists to monitor the 500ms Time-to-first-bird and 60fps runtime budgets.
- **Drift calibration testing:** Risks say calibration is necessary because an aggressive low-pass filter feels like a Tamagotchi, while a slow one feels unresponsive.
- **Presence signal integrity work:** Risks say browser focus and visibility heuristics vary across platforms, and over-counted presence accelerates drift.
- **Audio tuning:** Risks say WebAudio mixing must avoid "robotic-sounding motifs" and "jarring chorus overlaps."
- **Narration queue throttling:** Risks say screen-reader queues must be strictly throttled so they do not become overwhelming and force users to mute narration.
