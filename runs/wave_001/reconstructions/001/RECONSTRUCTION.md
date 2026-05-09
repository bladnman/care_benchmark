## System-level intent

1. **A low-key, observational relationship with affective depth.** The plan opens by saying Pocket Aviary v1 should establish a "low-key, observational relationship" with birds, with "high-fidelity affective depth and minimal UI chrome." This shows up again in the no-streaks/no-scores "No Gamification" boundary, the absence of "hunger" or "happiness" meters, the "No spinner" loading choice, idle micro-motion, and naturalist notebook/narration surfaces.

2. **Continuity and multi-device coherence through a thin-client, thick-server simulation.** The Architecture section states that the model exists "to ensure continuity and multi-device coherence." The same intent appears in "single canonical aviary per account," "multi-device sync," "The Server is the only writer," and clients that submit "events" rather than state.

3. **Canonical state over client-side ownership.** The client is a "view-only renderer of snapshots," while the server consumes an "append-only event log" and writes the "Canonical Snapshot." The Sync Model says this makes "last-write-wins irrelevant for bird traits."

4. **Gradual, expressive, non-punitive change.** Personality drift is "monotonic," "moving toward expressive," calibrated for visible change in "~3 weeks of regular presence," and "never negative." This aligns with the explicit non-goal that birds do not die, do not require maintenance, and have no custodial meters.

5. **Naturalist product voice.** The plan repeats "naturalist" across Field Notebook content, ARIA narration, captions, and keyboard focus indicators. Examples include "Naturalist prose string," "a small bird is preening on the front rail," "a low trill, pausing, low trill again," and "naturalist high-contrast."

6. **Accessibility as designed surfaces, not an afterthought.** Accessibility is in scope as "Designed surfaces for screen-reader narration, reduced-motion mode, and call captioning." Later sections specify ARIA-live narration, reduced-motion cross-fades, automatic call captions on WebAudio failure, and full keyboard navigation.

7. **Immediate calm presence with tight performance budgets.** The plan wants birds rendered in static "ready" poses "instantly," animation only after assets load, and "No spinner." Performance budgets reinforce this with "Time-to-First-Bird: <500ms," "Consistent 60fps," and "Zero growth over 30min session."

8. **Constrained, ambient social and privacy posture.** Social is limited to "One-to-one email-based visit invitations (read-only, ambient)," while "No Social Network" excludes profiles, discovery feeds, and public aviaries. Rollout instrumentation is "Aggregate RUM" with "no PII or bird-state telemetry."

## Per-feature whys

### 1. Scope

- **Modern web browsers / web-only v1:** The plan ties this to v1 scope control: "strictly limited to a browser-based experience" and the explicit non-goal "No Native Apps."

- **Procedural bird animation, call synthesis, and server-side simulation tick:** These carry the core "low-key, observational relationship" through animated birds, audible calls, and continuity from the server-side tick.

- **2 starter birds, capping at 7 based on aviary age:** NOT RECOVERABLE FROM PLAN

- **Return-greeting:** NOT RECOVERABLE FROM PLAN

- **Listen-in:** The Audio Pipeline explains that focus on Bird A triggers a gain ramp where `Gain_A` rises and `Gain_Others` drops, so the articulated why is focused listening.

- **Offer (seed, song, pool):** NOT RECOVERABLE FROM PLAN

- **Settle gesture:** NOT RECOVERABLE FROM PLAN

- **Field Notebook:** The Simulation Engine says notebook entries are generated for "noteworthy state changes," and the Data Model stores "Naturalist prose string."

- **Magic-link auth:** NOT RECOVERABLE FROM PLAN

- **Single canonical aviary per account and multi-device sync:** The plan's stated rationale is "continuity and multi-device coherence," with one canonical account-level aviary.

- **One-to-one email-based visit invitations:** The plan keeps visits "read-only, ambient" and excludes a broader "Social Network" with profiles, feeds, or public aviaries.

- **Screen-reader narration, reduced-motion mode, and call captioning:** These are named as "Designed surfaces" for Accessibility, then expanded into ARIA-live narration, slow cross-fades, and motif-triggered captions.

### 2. Architecture

- **Thin-client, thick-server simulation model:** The plan states the reason directly: "to ensure continuity and multi-device coherence."

- **React (TypeScript):** The plan assigns it to "UI chrome and layout."

- **WebAudio:** The plan assigns it to "procedural call synthesis" and later to species-specific motifs.

- **Canvas/WebGL:** The plan assigns it to "bird rendering and scene composition."

- **Client as view-only renderer of snapshots:** The rationale is canonical server ownership: the client pulls a JSON snapshot and does not write bird state.

- **Node.js Simulation Service running the tick:** The server tick consumes presence and interaction events, updates canonical state, and writes new snapshots.

- **Append-only event log:** The Sync Model explains that clients submit events rather than state, which makes "last-write-wins irrelevant for bird traits."

- **PostgreSQL:** The plan uses it for "persistent records" such as Account, Bird, Personality Vector, and Notebook.

- **Redis:** The plan uses it for "recent event-log buffering and active session snapshots."

- **JSON snapshot pull with state interpolation:** The plan says interpolation exists "to smooth transitions between snapshot positions."

- **Client-side local loops for preening and head-tilting:** The plan says micro-motions are "informed by the bird's current mood."

### 3. Data Model

- **Account `created_at`:** The plan gives the purpose as "used for bird-adoption pacing."

- **Account `settings`:** The plan stores "Accessibility preferences" and "notification toggles."

- **Bird `id`:** The plan names the purpose as "Stable identity."

- **Bird `personality_vector`:** The rationale is expressive trait modeling: vectors are updated by drift, influence call timing through `vocal_frequency`, vary pitch/vibrato through `social_warmth`, and include visible traits such as `plumage_saturation`.

- **Bird `current_mood`:** The plan uses mood for transitions based on local time, weather, and recent interactions, and for call timing and local micro-motion.

- **Bird `drift_accumulator`:** The plan gives the purpose as a "Buffer for pending drift calculations from the tick."

- **Field Notebook `content`:** The plan gives the content voice as "Naturalist prose string."

### 4. API Surface

- **`POST /auth/request-link` and `POST /auth/verify`:** These support magic-link auth by triggering a magic-link email and returning a session JWT.

- **`GET /aviary/snapshot`:** The endpoint returns the current state of birds, weather, time-of-day, and active events so the client can render snapshots.

- **`GET /aviary/notebook`:** The endpoint exposes paginated notebook entries for the Field Notebook surface.

- **`POST /aviary/events`:** The endpoint appends events to the log, matching the architecture where clients submit events rather than state.

- **`POST /social/invite`, `DELETE /social/invite/:id`, and `GET /social/visit/:token`:** These implement email visit links, invitation revocation, and an ambient visitor snapshot while keeping social read-only.

### 5. Simulation Engine Design

- **1-minute tick cadence:** NOT RECOVERABLE FROM PLAN

- **Presence-Time:** The plan says it validates pings against "visibility/focus rules."

- **Personality Vector updates:** The plan says vectors move "toward expressive" based on presence and interaction weights.

- **Mood transitions:** The plan says mood uses local time, ambient weather, and recent interactions.

- **Notebook entry generation:** The plan says entries are generated from "noteworthy state changes" such as "Pip greeted before Wren."

- **Canonical Snapshot writes:** The plan writes new snapshots to Redis/Postgres so the server remains the canonical source.

- **Low-pass drift function:** The plan uses it for gradual change, calibrated for visible change in "~3 weeks of regular presence."

- **Asymmetric drift:** The plan states the non-punitive rationale: "Neglect results in 0 change, never negative."

- **Species-specific motif sets:** The plan uses these for the Call-Grammar Runtime so calls are species-specific.

- **Timing modulated by `vocal_frequency` and `mood`:** The plan ties call interval to individual vocal tendency and current mood.

- **Pitch and vibrato varied by `social_warmth`:** The plan ties audible expression to the social warmth trait.

### 6. Sync Model

- **Server as only writer for personality and notebook state:** The stated rationale is "Conflict Prevention."

- **Clients submit events, not state:** The plan says this makes "last-write-wins irrelevant for bird traits."

- **Long-polling or low-frequency fetch:** The plan says a 30s pull model ensures "multi-device consistency without high overhead."

### 7. Frontend Rendering Pipeline

- **Three-plane parallax:** NOT RECOVERABLE FROM PLAN

- **Idle Micro-motion:** The plan uses procedural transforms for "breathing" and "scanning," supporting the observational affect of animated birds.

- **Reduced Motion:** The plan provides "slow cross-fades (3s) between still poses instead of frame-by-frame animation."

- **Critical Path snapshot embedding or immediate fetch:** The plan uses this to make current state available before the first frame.

- **First Frame ready poses and no spinner:** The plan says to render birds "instantly" in static "ready" poses and begin animation only after assets load.

### 8. Audio Pipeline

- **WebAudio oscillators and gain nodes:** The plan uses them to synthesize "species-specific motifs."

- **Central `AviaryMixer`:** The plan says it manages gain levels for all birds.

- **Listen-in Mix:** The plan uses a 2s linear ramp so Bird A rises and other birds drop, making the selected bird the focus.

- **Audio Fallback:** If WebAudio fails, the plan switches to "Silence + Automatic Call Captions enabled."

### 9. Accessibility Surfaces

- **Hidden ARIA-live narration:** The plan uses a polite live region updated by a naturalist prose generator at idle cadence.

- **Call captions:** The plan triggers a visual overlay from WebAudio motifs and uses "Naturalist phrasing."

- **Keyboard navigation:** The plan provides "Full Tab/Arrow/Enter navigation" and focus indicators designed for "naturalist high-contrast."

### 10. Performance Budgets

- **JS Bundle under 2MB gzipped:** The plan uses code-splitting for settings and notebook to stay within the bundle budget.

- **Time-to-First-Bird under 500ms on 4G:** The plan makes first visible bird presence a named performance budget.

- **Consistent 60fps on 2021-era mid-range hardware:** The plan sets smooth runtime rendering as the performance target.

- **Zero memory growth over 30min session:** The plan gives the implementation rationale as "strict buffer reuse."

### 11. Rollout & Risk

- **Server-side tweakable drift coefficients:** The plan uses this to mitigate the risk that birds drift too fast or too slow "without client updates."

- **Internal Phase 0 with simulated presence:** The plan uses a week of simulated presence to "verify feel."

- **Adaptive interpolation windows based on RTT:** The plan uses this to mitigate snapshot interpolation "pop" on high-latency links.

- **Invite-only alpha:** The plan uses 100 users to "calibrate drift and server load."

- **Public v1 launch with 2-bird start:** NOT RECOVERABLE FROM PLAN

- **Aggregate RUM instrumentation:** The plan uses RUM for frame-rate and bundle size while excluding "PII or bird-state telemetry."
