## System-level intent

1. Ambient, evolving relationships between users and birds

The plan frames Pocket Aviary V1 as "a browser-based virtual aviary focused on ambient, evolving relationships between users and birds." This intent shows up again in the procedural bird engine, "mood/personality drift," "presence-time and interaction signals," "idle micro-motion," and naturalist narration.

2. Non-game, non-punishing, non-social product boundaries

The plan explicitly excludes "Games/gamification (streaks, scores)," "Tamagotchi mechanics (punishing neglect)," and "social networks (profiles, discovery)." These non-goals make the relationship model ambient rather than score-driven, punitive, or discovery/profile-driven.

3. Server-authored canonical state

The architecture and sync model repeatedly center server authorship: the server is the "Canonical state manager," "Canonical state lives on the server," clients "consume snapshots," and there is "no last-write-wins for personality (server-only authorship)." Sync correctness is also called a risk with "hard dependencies on server tick integrity."

4. Procedural change from presence, interaction, time, personality, and events

The plan ties bird change to procedural signals: "Low-pass filter over presence-time and interaction signals," mood "modulated by time of day, personality, and events," and procedural rendering/audio. The vocabulary of "drift," "tick," "current mood," and "personality vector" carries the evolving-system intent.

5. Smooth snapshot-based presentation

The client is "Stateless; pulls snapshots from the server," and the render boundary says the "Client renders snapshots" and "interpolates states smoothly between snapshots." The frontend rendering pipeline reinforces this with "60fps idle micro-motion," parallax, and procedural animation.

6. Accessible parallel surfaces

Accessibility is not a single add-on in the plan. It includes "Naturalist prose" via screen reader, "Reduced-Motion" cross-fade, "Procedural prose captions for calls," and "Full reachability" by keyboard.

7. Operationally limited analytics and account data

The data and rollout sections keep account/telemetry language constrained: "Synthetic UUID, encrypted email," "Operational health only," and "No per-bird state in analytics."

## Per-feature whys

### 1. Scope

- Two starter birds (cap at 7): NOT RECOVERABLE FROM PLAN

- Magic-link authentication: NOT RECOVERABLE FROM PLAN

- Multi-device sync: The plan's rationale is canonical server state: "Canonical state lives on the server," clients consume snapshots, and there is "No client-to-client sync" or "last-write-wins for personality."

- Procedural bird engine (mood/personality drift): The rationale is the product's "ambient, evolving relationships." The plan explains the mechanism as "mood/personality drift," a "Low-pass filter over presence-time and interaction signals," and mood modulated by time of day, personality, and events.

- Field notebook: NOT RECOVERABLE FROM PLAN

- Presence-based interaction tracking: The plan defines presence as a conjunction of `visibilityState`, `window focus`, and recent `pointermove/keypress`, then uses "presence-time and interaction signals" for drift.

- Read-only visitor feature: NOT RECOVERABLE FROM PLAN

### 2. Architecture

- Server / Node.js simulation engine: The server exists as the "Canonical state manager" and handles "authentication, event log processing, and simulation ticks."

- Client / React (TypeScript) + WebAudio + Canvas/SVG-based rendering: The client is "Stateless" and "pulls snapshots from the server," making rendering and audio presentation follow canonical server snapshots.

- Render Pipeline Boundary: The rationale is smooth snapshot presentation: the "Client renders snapshots" and "interpolates states smoothly between snapshots."

### 3. Data Model

- Account with synthetic UUID and encrypted email: NOT RECOVERABLE FROM PLAN

- Bird with stable internal ID, personality vector, and current mood: The rationale is to support mood/personality drift. Personality dimensions such as "boldness," "social warmth," and "curiosity" can modulate mood and events; mood is the fast-timescale state.

- Presence: Presence is the signal surface for "presence-time" in drift, combining visibility state, window focus, and recent pointer/keypress activity.

- Event Log: The event log captures user interactions such as "offer, listen-in, settle" so the server can perform "event log processing" and feed interaction signals into drift and mood.

### 4. API Surface

- `GET /snapshot`: The rationale is snapshot consumption: clients pull the "current canonical state" from the server.

- `POST /events`: The rationale is to submit interaction events, including "presence ping, listen-in, offer," into the event log and simulation inputs.

- `POST /auth/magic-link`: NOT RECOVERABLE FROM PLAN

### 5. Simulation Engine Design

- Tick: The rationale is server-side simulation of canonical state. The plan says the tick runs "once per minute" and later names "server tick integrity" as a sync correctness dependency.

- Drift: The rationale is slow relationship change: a "Low-pass filter over presence-time and interaction signals," "Monotonic toward expressive," with calibration risk if it feels "too slow or too fast."

- Mood: The rationale is fast-timescale variation: mood resets on a "daily-ish cadence" and is "modulated by time of day, personality, and events."

### 6. Sync Model

- Server-side canonical state: The rationale is consistency across devices and personality authorship: canonical state lives on the server and personality has "server-only authorship."

- Snapshot-consuming clients: The rationale is avoiding client-to-client sync and last-write-wins behavior; clients consume server snapshots instead.

### 7. Frontend Rendering Pipeline

- 60fps idle micro-motion: The rationale is smooth ambient presentation, reinforced by the runtime budget of "60fps on 5-year-old hardware."

- Parallax background/foreground: NOT RECOVERABLE FROM PLAN

- Procedural animation with cross-fading still poses for reduced-motion: The rationale is to keep procedural animation while providing a reduced-motion cross-fade sequence.

### 8. Audio Pipeline

- Procedural call grammar using WebAudio: The rationale is bird calls generated procedurally while avoiding "Audio uncanniness"; the risk section says the grammar "must avoid repetition."

- Chorus mixing through procedural combination: NOT RECOVERABLE FROM PLAN

- Slow ramp for listen-in mix balance: The rationale appears in the feature phrase itself: managing the "listen-in mix balance" with a slow ramp.

### 9. Accessibility Surfaces

- Narration: The rationale is screen reader access through "Naturalist prose" updated every "30-60s."

- Reduced-Motion: The rationale is an alternate motion surface: a "Designed cross-fade sequence for motion."

- Captioning: The rationale is making calls available as "Procedural prose captions."

- Keyboard: The rationale is "Full reachability."

### 10. Performance Budgets

- Bundle under 2MB gzipped: NOT RECOVERABLE FROM PLAN

- First bird under 500ms: NOT RECOVERABLE FROM PLAN

- Runtime at 60fps on 5-year-old hardware with no memory growth over 30 mins: The rationale is sustained smooth runtime for the ambient rendering experience.

### 11. Rollout

- Web-only v1: NOT RECOVERABLE FROM PLAN

- Telemetry for operational health only: The rationale is limiting analytics to "Operational health only" with "No per-bird state in analytics."
