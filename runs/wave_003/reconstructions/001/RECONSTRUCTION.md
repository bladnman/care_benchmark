## System-level intent

1. **The aviary continues without the viewer.** This is stated directly in Architecture as the reason for the "thin-client, thick-server architecture": it enforces the "aviary continues without the viewer" rule. It also shows up in the "continuous tick" that computes drift and mood "regardless of client connectivity," in the "server is the absolute source of truth," and in the Initial Load goal of birds already "mid-motion" rather than waiting for the viewer to begin the scene.

2. **Care is non-punitive and deliberately not gamified.** The Scope excludes "Gamification (streaks, levels, points, leaderboards)" and "Tamagotchi mechanics (hunger, death, negative drift from neglect)." The Drift Function repeats that "Neglect results in zero drift, not negative drift," and the Drift Calibration risk says that if drift is "too fast" the product "feels like a Tamagotchi," while if it is "too slow" it "feels broken."

3. **State changes should be conflict-resistant and action-based.** Sync Model names a "Single Canonical State" and says "Clients never send absolute state." They "only send actions," and the server computes results through "Additive Deltas" to prevent "last-write-wins conflicts between devices." The Event Log is append-only, and the Event Push API accepts batches of interaction events.

4. **Bird life is procedural, stateful, and expressive.** The plan carries this through "personality_vector," "current_mood," species-specific "visual silhouette and motif library," "Procedural bird calls synthesized via WebAudio," and a Motif Library where runtime synthesis varies "pitch, timing, and sequence" based on mood and vocal frequency.

5. **The product voice is quiet, observational, and naturalist.** The Field Notebook stores "naturalist voice"; the Notebook Generator produces "specific, lowercase, present-tense observations"; Screen Reader Narration uses generated naturalist prose; Initial Load falls back to "a quiet empty field"; and the Top Bar "fades to near-transparent on cursor idle."

6. **Accessibility is a core surface, not an afterthought.** Scope lists "Screen-reader narration, reduced-motion mode (cross-fades), procedural call captions, full keyboard navigation, WCAG AA contrast." The Accessibility risk says narration should be treated as "a core writing task," and the audio fallback enables "textual call captions" when WebAudio fails or is disabled.

7. **Social connection stays bounded and read-only.** Scope includes "Read-only visits via one-time email invite" while excluding "Social networks (public discovery, chat, comments, co-presence, avatars)." The API includes invite generation and revoke, matching a narrow visit model rather than an open network.

8. **Measurement must respect a strict privacy boundary.** Observability names a "Strict privacy boundary" and forbids "per-bird or per-account interaction telemetry." The metrics that remain are aggregate or operational: bundle sizes, framerates, TTFB, simulation tick latency, WebAudio errors, and HTTP codes.

## Per-feature whys

### 1. Scope

- **Web-only browser-based client:** NOT RECOVERABLE FROM PLAN.
- **Single horizontal aviary scene with 3 perch zones:** NOT RECOVERABLE FROM PLAN.
- **Day/night cycle and ambient weather:** The plan ties these to mood and rendering: Mood Transitions use "local time of day" and "ambient weather," while Lighting & Weather derives "palette shifts from the local time of day."
- **2 starter birds, expanding up to 7 over time based on account age:** NOT RECOVERABLE FROM PLAN.
- **Procedural bird calls synthesized via WebAudio:** The Audio Pipeline says "No recorded loops"; procedural generation lets species motifs vary by "pitch, timing, and sequence" based on mood and vocal frequency, and lets simultaneous bird nodes avoid phase-cancellation.
- **Single-user accounts with magic-link email auth:** NOT RECOVERABLE FROM PLAN.
- **Cross-device sync via server-side simulation tick:** The architecture uses a server-side tick to keep the aviary computing "regardless of client connectivity" and uses server truth plus additive events to prevent sync conflicts.
- **Notice/Return-greeting:** NOT RECOVERABLE FROM PLAN.
- **Listen-in:** The Audio Pipeline explains that focusing a bird ramps up that bird's gain, decays other birds' gains, and returns to baseline on blur.
- **Offer (seed, song, pool):** The Offers API checks cooldowns, and the Drift Function names offers as one input for drift; Mood Transitions also evaluate recent events.
- **Settle:** NOT RECOVERABLE FROM PLAN.
- **Field notebook:** The Notebook Generator is triggered by notable state changes and creates "specific, lowercase, present-tense observations" in a "naturalist voice."
- **Read-only visits via one-time email invite:** The plan constrains social to one-time email invite and revoke while excluding public discovery, chat, comments, co-presence, and avatars.
- **Screen-reader narration:** The plan uses a live-region aria element to describe the scene in generated naturalist prose and rate-limits updates "to prevent queue flooding."
- **Reduced-motion mode:** The rendering engine swaps animation loops like "preening" and "flying" with "slow cross-fades between static poses" when `prefers-reduced-motion` is true.
- **Procedural call captions:** Captions provide textual descriptions such as "a soft three-note rise," and the fallback path enables textual call captions if WebAudio fails or is disabled.
- **Full keyboard navigation:** Keyboard Nav provides a high-contrast focus ring, Tab navigation through top bar and birds, Enter for listen-in, and Esc to cancel.
- **WCAG AA contrast:** Contrast is enforced for "all UI chrome and text overlays."

### 2. Architecture

- **Thin-client, thick-server architecture:** The stated why is to enforce the "aviary continues without the viewer" rule and prevent sync conflicts.
- **Client rendering and event role:** The client renders state snapshots, interpolates motion, synthesizes audio, and sends interaction events; this keeps canonical state computation outside the client.
- **Backend services for Auth API, Snapshot API, and Event Intake API:** These services separate authentication, state pull, and event intake around the server-owned state model.
- **Simulation Engine background worker:** It runs a continuous tick "across all active aviaries" so drift and mood transitions happen even when the client is disconnected.
- **Database plus event store:** Relational data stores accounts, birds, and personality vectors; the event store exists for processing client interaction events.

### 3. Data Model

- **`account_id` synthetic UUID:** It is the primary key "across all systems."
- **Encrypted `email`:** It is "used strictly for auth and export/visit invites."
- **`created_at`:** It supports account-age behavior used by Ramping, where a cron job evaluates account age.
- **Stable `bird_id`:** It gives each bird a stable UUID identity.
- **User-assigned `name`:** NOT RECOVERABLE FROM PLAN.
- **`species` enum:** It defines the bird's "visual silhouette and motif library."
- **Server-side `personality_vector`:** It stores Boldness, Social Warmth, Vocal Frequency, Plumage Saturation, and Curiosity, and is used by drift, mood, and audio while being "Persisted server-side only."
- **`current_mood`:** It is the value Mood Transitions determine during the tick and Audio uses to vary calls.
- **Append-only Event Log:** It records user interactions so the tick can drain events and compute additive deltas; offline recovery also aims for eventual delivery into this log.
- **Field Notebook data:** Entries preserve timestamped `prose_content` in "naturalist voice."

### 4. API Surface

- **Auth endpoints:** `/auth/request-link` and `/auth/verify` issue "expiring session tokens."
- **State Pull snapshot:** `/api/aviary/snapshot` returns layout, positions, moods, and active animations, and is polled on visibility change, render gaps, and slow keepalive.
- **Event Push:** `/api/aviary/events` accepts batches of interaction events for the action-based state model.
- **Offers endpoint:** `/api/aviary/offer` submits seed, song, or pool offers while "checking cooldowns."
- **Social invite and revoke:** `/api/visits/invite` generates email and `/api/visits/revoke` supports bounded, revocable visits.
- **Notebook entries endpoint:** NOT RECOVERABLE FROM PLAN.

### 5. Simulation Engine Design

- **The Tick:** It drains the event log, computes additive deltas to the personality vector, and determines mood.
- **Presence Accounting:** Presence requires visible tab, document focus, and pointer/key activity, so presence pings only count when all three hold.
- **Drift Function:** The low-pass filter makes traits drift "towards expressiveness"; neglect creates "zero drift, not negative drift," supporting the non-Tamagotchi intent.
- **Mood Transitions:** Mood is evaluated from recent log events, local time of day, ambient weather, and the current personality vector.
- **Notebook Generator:** It triggers only on notable state changes and generates "specific, lowercase, present-tense observations."

### 6. Sync Model

- **Single Canonical State:** The server is "the absolute source of truth."
- **Additive Deltas:** Clients send actions, not absolute state, to prevent "last-write-wins conflicts between devices."
- **Interpolation:** When bird state changes between snapshots, the client interpolates movement "rather than snapping."

### 7. Frontend Rendering Pipeline

- **Initial Load with no spinners, mid-motion birds, and quiet empty field fallback:** NOT RECOVERABLE FROM PLAN.
- **Scene Composition with 3 depth planes and minimal parallax:** NOT RECOVERABLE FROM PLAN.
- **Lighting & Weather palette shifts:** The client derives palette shifts from local time of day, aligning rendering with the day/night cycle.
- **Ambient leaves/feathers particle system:** It is client-side and independent of the server tick, keeping ambient weather effects out of canonical simulation.
- **Top Bar fading on cursor idle:** NOT RECOVERABLE FROM PLAN.

### 8. Audio Pipeline

- **WebAudio synthesis with no recorded loops:** Procedural synthesis supports generated variation instead of fixed loops.
- **Species Motif Library:** Each species has an abstract grammar so calls can vary by mood and vocal frequency trait.
- **Chorus Mechanic:** Independent procedural bird audio nodes can run simultaneously "without phase-cancellation."
- **Listen-in Mix:** Focus increases one bird's audio gain and decays others, then returns to baseline on blur.
- **Audio Fallback:** If WebAudio fails or is disabled, the system fails to silence and enables textual call captions.

### 9. Accessibility Surfaces

- **Screen Reader Narration live region:** It periodically receives generated prose describing the scene and is rate-limited to avoid flooding the queue.
- **Captions:** They render procedural call descriptions near the bird, making calls legible as text.
- **Keyboard Nav:** It provides a high-contrast focus ring and a complete keyboard path for listen-in and cancel.
- **Contrast:** WCAG AA compliance is enforced for UI chrome and text overlays.

### 10. Performance Budgets & Observability

- **Initial JS Bundle < 2MB with code-splitting:** Code-splitting is required for settings and visit logs to keep the initial bundle within budget.
- **Time-to-first-bird < 500ms on mid-tier mobile 4G:** The rollout instrumentation monitors the 500ms budget from day one.
- **60fps idle motion on 5yo hardware:** The framerate budget keeps idle motion smooth on older hardware.
- **Flat memory footprint over 30 minutes:** The plan calls for reusing audio buffers and cleaning up unseen DOM nodes.
- **Strict privacy boundary:** Observability avoids per-bird and per-account interaction telemetry.
- **Operational metrics:** Bundle sizes, render framerates, TTFB, simulation tick latency, WebAudio error rates, and API HTTP codes let the team monitor performance and reliability without per-account telemetry.
- **Simulation tick p99 alarm:** The tick latency alarm fires at p99 greater than 5 seconds.

### 11. Rollout

- **V1 Launch with 2 birds out of the gate:** NOT RECOVERABLE FROM PLAN.
- **Ramping by account age with new species offers up to 7 birds:** NOT RECOVERABLE FROM PLAN.
- **Instrumentation from day one:** Synthetic browser tests and aggregate RUM monitor the 500ms TTFB and 60fps budgets.

### 12. Risks

- **Drift Calibration:** The why is explicit: too fast feels like a Tamagotchi, too slow feels broken; mitigation is internal dogfooding with time-accelerated simulation to tune low-pass coefficients.
- **Audio Uncanniness:** Procedural generation may sound robotic if the grammar is too simple; mitigation is heavy frontend time in motif synthesis and randomized timing offsets.
- **Sync Correctness:** Dropped events from offline clients are mitigated through robust local queueing and eventual delivery to the append-only log so the server eventually factors them into the tick.
- **Accessibility narration quality:** Screen reader narration may sound robotic, so the plan treats narration generation as a core writing task and shares logic with the Field Notebook.
