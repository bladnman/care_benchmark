## System-level intent

1. Naturalist design philosophy: The plan explicitly says the implementation adheres to a naturalist design philosophy: "feels alive," "notice never announce," "specificity," and "restraint." This shows up again in "naturalist narration," "Recognizable per-bird motifs," "Gradual mix re-balancing (not solo/mute)," and the risk framing of drift calibration as '"felt alive" vs "Tamagotchi".'

2. Restrained, non-game product shape: The scope excludes "gamification (streaks, badges, scores)," "Tamagotchi mechanics (no decay/punishment)," and "social network surfaces." The simulation echoes this with drift that is "monotonic toward expressive" and "non-symmetric (neglect does not cause negative drift)."

3. Canonical continuity over client-side reconciliation: The plan emphasizes "Per-account, persisted server-side" canonical state, clients pulling "snapshots of canonical aviary state," "Server-side simulation," and "No client-side merge or conflict resolution." The risk section reinforces that sync correctness "must preserve drift history."

4. Single-user, web-only focus: The plan lists a "Browser-based aviary," excludes "Native apps," and later states "Shipping: Single-user, web-only." The frontend also keeps the aviary "Horizontal," "single-screen," and "no scrolling."

5. First-class accessibility and motion restraint: Accessibility is included in scope as "narration, captions," then restated as "First-class accessibility" with "naturalist narration," "designed reduced-motion mode," and "call captioning."

6. Operational observability without bird-level telemetry: The plan calls for "Aggregate operational telemetry only" covering "latency, error rates, frame times" and explicitly says "No per-bird state in telemetry." Alerts are limited to "p99 simulation-tick latency > 5s."

## Per-feature whys

### 1. Scope

- Browser-based aviary: The plan keeps shipping "single-user, web-only" and excludes "Native apps," so the browser aviary is the delivery surface for that scoped product.
- Bird simulation (personality drift, mood): This supports the naturalist goal that the aviary "feels alive" through "personality vectors" and "mood transitions."
- Procedural audio (WebAudio): The audio is meant to provide "Recognizable per-bird motifs" while being "Synthesized client-side (no recorded audio)."
- Magic-link auth: NOT RECOVERABLE FROM PLAN
- Multi-device sync: Sync exists so clients pull "the same snapshot (canonical state)" with "No client-side merge or conflict resolution."
- Read-only visitor feature: NOT RECOVERABLE FROM PLAN
- Field notebook: NOT RECOVERABLE FROM PLAN
- Accessibility (narration, captions): The plan treats this as "First-class accessibility," specifically through "naturalist narration" and "call captioning."
- Reduced-motion mode: The rationale is accessibility; it is a "designed reduced-motion mode" under "First-class accessibility."
- Native apps exclusion: This follows from the "web-only" shipping boundary.
- Gamification exclusion (streaks, badges, scores): This follows the plan's "restraint" philosophy and keeps the product outside streak/badge/score mechanics.
- Tamagotchi mechanics exclusion (no decay/punishment): This is echoed in drift being "non-symmetric" so "neglect does not cause negative drift."
- Social network surfaces exclusion (discovery, profiles, public feeds): This aligns with the "Single-user" rollout and avoids discovery, profiles, and public feeds.

### 2. Architecture

- React + TypeScript: NOT RECOVERABLE FROM PLAN
- Canvas API for rendering (aviary): The plan assigns Canvas to "rendering (aviary)."
- Procedural audio via WebAudio API: WebAudio is the browser mechanism for "Procedural Calls" and client-side synthesis.
- Node.js (Express): NOT RECOVERABLE FROM PLAN
- Simulation engine (server-side tick): The server-side tick computes "drift (personality vectors) and mood transitions" and consumes the event log.
- Postgres (or similar SQL DB) for accounts, birds, and interaction history: This supports "Per-account, persisted server-side" canonical state and persisted "interaction history."
- Synthetic UUIDs for all account/bird references: NOT RECOVERABLE FROM PLAN
- Render Boundary: The client pulls "snapshots of canonical aviary state" and performs only "interpolation," preserving the server as the canonical source.

### 3. Data Model & Sync

- Canonical State: It is "Per-account" and "persisted server-side" so the aviary has a durable canonical state.
- Sync: "Server-side simulation ensures clients pull the same snapshot (canonical state)" and avoids "client-side merge or conflict resolution."
- Event Log: The "Append-only event log" records client interactions such as "offer" and "listen-in" so the "Server consumes log via simulation tick."

### 4. Simulation Engine

- Tick: The tick exists to compute "drift (personality vectors) and mood transitions."
- ~1 minute cadence: NOT RECOVERABLE FROM PLAN
- Drift: Drift is "Presence-time based," "monotonic toward expressive," and "non-symmetric" so neglect "does not cause negative drift."
- Mood: Mood handles a "Fast-timescale, daily-ish reset" and is influenced by "time-of-day, recent interactions, ambient events."

### 5. Frontend Pipeline

- Horizontal scene, single-screen, no scrolling, three perch zones: NOT RECOVERABLE FROM PLAN
- Performance targets (<2MB JS bundle, <500ms TTFB, 60fps idle motion, no memory growth): These are tied to "time-to-first-bird," "60fps idle motion," "leak-proof" behavior, and the risk of meeting "500ms TTFB" on mobile.
- Accessibility: The frontend repeats the same first-class accessibility rationale: "naturalist narration," "designed reduced-motion mode," and "call captioning."

### 6. Audio Pipeline

- Procedural Calls: Calls are "Synthesized client-side (no recorded audio)" while remaining "Recognizable per-bird motifs."
- Listen-in: Listen-in uses "Gradual mix re-balancing (not solo/mute)," which carries the plan's restrained interaction style.

### 7. Rollout & Observability

- Shipping: "Single-user, web-only" defines the rollout boundary.
- Instrumentation: The plan wants "Aggregate operational telemetry only" for "latency, error rates, frame times" and explicitly avoids "per-bird state in telemetry."
- Alerts: The alert exists so "p99 simulation-tick latency > 5s" alerts engineering when simulation latency crosses that threshold.

### 8. Risks

- Drift calibration: The risk is balancing '"felt alive" vs "Tamagotchi".'
- Audio synthesis variability: The plan calls out variability "across browser versions."
- Sync correctness: The stated reason is that sync "must preserve drift history."
- Performance targets: The stated risk is meeting "500ms TTFB" on mobile.
