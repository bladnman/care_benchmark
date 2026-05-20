## System-level intent

- **Relationship deepening over engagement pressure.** This shows up in "Bird Adoption & Population Scaling," where more birds unlock by "calendar age of the aviary account (relationship deepening)" and "rather than user interaction metrics." It returns in "Adoption and Population Milestones," which says population grows by "chronological age of the account" and avoids "gamified engagement triggers."

- **No punishment, no care anxiety, and quiet ambient consequences.** The plan explicitly excludes "Custodial/Tamagotchi Mechanics": birds cannot "die, fall ill, starve, or show distress." In the "Monotonic Drift Function," traits "never decay," fulfilling the "no Tamagotchi punishment" rule; ignored birds become "ambient" rather than "hostile."

- **A calm, non-gamified product surface.** The "No Gamification Elements" exclusion bans "achievements, streaks, XP, level systems, green-dot calendars, badges, or numeric metrics displayed to the user." This is reinforced by adoption timing being non-interaction-based and by the API hiding raw personality-vector values.

- **Server-authoritative continuity across devices.** The architecture uses a "strict Client-Server split" with the server as "the single source of truth." The sync model says the client "never submits absolute state values"; instead, an event queue and server tick dedupe and apply deltas to prevent "multi-device conflicts."

- **Privacy and hidden internal mechanics.** The data model uses synthetic `account_id` values and encrypted email "to prevent PII leakage." The API says no endpoint exposes raw floating-point personality values "to protect the core design intent." Observability excludes raw presence, bird names, motifs, direct interaction history, and PII.

- **Naturalist observer voice rather than dashboard voice.** The "Field Notebook" is a "sparse, auto-generated, read-only observer log" in a "naturalist field-notebook voice." Screen-reader narration and call captions also use "naturalist prose" and "naturalist descriptions," and the prose-fatigue mitigation preserves a descriptor vocabulary such as "fluffed," "ruffled," and "feather-soft."

- **Equivalent access through alternate sensory surfaces.** Accessibility is not only compliance: screen-reader narration is described as "an equivalent experience"; reduced motion replaces paths with "slow cross-fades"; call captioning displays descriptions of calling patterns; keyboard navigation reaches birds and listen-in. WebAudio failure also turns on call captioning automatically.

- **Procedural, smooth, living ambience with performance discipline.** The plan favors real-time procedural systems: WebAudio synthesis, call grammars, simulation ticks, interpolated rendering, canvas layers, and idle micro-motion where birds are "never statically frozen." The same intent is constrained by budgets for bundle size, "Time-to-First-Bird," 60fps, CPU load, and low-frequency snapshots without "snapping" or teleports.

## Per-feature whys

### Scope and Boundaries

- **Bird Adoption & Population Scaling / Adoption and Population Milestones:** The plan says the two starter birds, seven-bird cap, and day-based unlocks exist so population growth is paced by "calendar age," "relationship deepening," and "chronological age of the account," not "user interaction metrics" or "gamified engagement triggers."

- **Return-Greeting:** The plan's stated purpose is a "welcoming action when a session begins." Its staggered procedure varies by "absence length, bird boldness, and current mood" so the return is shaped by the aviary state instead of being a static greeting.

- **Presence Tracking / Presence Time Calculation:** The rationale is to count real attention precisely. The plan requires page visibility, focus, and recent pointer/key activity, then deduplicates concurrent devices and caps a tick at 60 seconds so multiple sessions cannot inflate presence.

- **Listen-In / Chorus Mixing and Listen-In Decay:** The plan says listen-in "brings one selected bird to the foreground" while others fade to "a low ambient level." The audio design keeps the focused bird at gain `1.0` and lowers other birds to `0.12`, preserving the chorus as ambience rather than muting the aviary.

- **Offers:** NOT RECOVERABLE FROM PLAN

- **Settle Gesture:** The plan calls this an "opt-in soft session-end action." Its why is to gently end a session by shifting light to evening, quieting calls, and giving a 5-second undo grace period; in the mood state machine, "Settle gesture" and "Settle" help wary or curious birds return to content.

- **Field Notebook / Notebook Entries / Notebook Endpoint:** The plan's rationale is an observer surface: a "sparse, auto-generated, read-only observer log" in a "naturalist field-notebook voice." The endpoint exposes those generated entries without turning them into editable user metrics.

- **Screen-reader narration:** The plan says this provides "an equivalent experience." Snapshot data is translated into "cohesive prose" in a hidden `aria-live` region, with slow idle cadence and immediate updates after user actions.

- **Reduced-motion mode / Reduced-motion specification:** The why is to replace smooth paths, parallax, and micro-animations when `prefers-reduced-motion` is detected. The plan substitutes "slow cross-fades," disables parallax, and removes ambient drift animations.

- **Dynamic call captioning:** The plan uses captions to display "naturalist descriptions of calling patterns." It also becomes the automatic fallback when WebAudio fails or permissions are denied.

- **Full keyboard navigation and WCAG AA contrast compliance:** The rationale is operability and readable alternatives: the keyboard path creates a logical ring through controls and birds, while captions use high-contrast containers exceeding WCAG AA.

- **Identity, Auth, and Sync / Magic-link authentication:** The plan uses "single-user accounts" and 15-minute email magic links to sign in and sync "a single canonical aviary state across multiple devices."

- **No Native Clients:** The plan keeps v1 "Web-only" so data structures and API boundaries can be optimized for "standard web protocols (HTTP, WebAudio)."

- **No Gamification Elements:** The rationale is to keep achievements, streaks, XP, levels, green-dot calendars, badges, and displayed numeric metrics out of the experience.

- **No Custodial/Tamagotchi Mechanics:** The why is to avoid distress and punishment. Birds cannot die, fall ill, starve, or show distress; neglect becomes "quiet ambient behavior."

- **No Social Network Features / Visit Invitations:** The plan excludes public discovery, leaderboards, shared or mutual aviaries, comments, chat, avatars, and co-presence indicators. The only social surface is opt-in visit links, constrained to read-only snapshots.

- **No Default Push Notifications:** The rationale is non-intrusion: the system will not "push, ping, or email users" unless a host explicitly opts into email notifications when a friend uses a visit link.

### System Architecture, Data Model, and API Surface

- **Client-Server split:** The plan's why is explicit: the server holds "the single source of truth" and the client is an "interactive rendering and audio synthesis layer."

- **Client Browser SPA:** The client receives JSON snapshots, interpolates between them, synthesizes audio in real time, and monitors attention. Its role is to make the aviary feel smooth and responsive while the server remains authoritative.

- **Server Backend / API Layer / Simulation Engine:** The server centralizes magic-link auth, event queue management, and periodic simulation ticks so state updates happen through controlled transactions.

- **Database Layer / relational store:** The database persists account credentials, bird records, "persistent personality vectors," current moods, field notebook entries, and visit invitations so the canonical aviary survives across devices and sessions.

- **Synthetic `account_id` and encrypted email:** The plan states the rationale directly: "To prevent PII leakage and ensure data integrity," tables reference synthetic UUIDs and store email encrypted in one account column.

- **Sessions:** The plan ties sessions to magic-link authentication and HttpOnly session cookies; session tokens are stored as hashes with expiry and revocation so account access can be controlled.

- **Bird records and personality vectors:** The birds table stores stable traits such as `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, and `curiosity`, plus mood and perch zone, because later simulation, rendering, and audio modulation depend on persistent bird state.

- **Interaction Events Log / Event queue:** The rationale is transactionality and conflict control. Interaction events are appended, grouped by account, processed by the tick, and later marked processed instead of letting clients write final state directly.

- **API transaction boundary / no raw personality vectors:** The plan says this protects "the core design intent" by ensuring no endpoint exposes raw floating-point personality values to the client.

- **Aviary snapshot endpoint:** The endpoint gives the client renderable state such as light, weather, mood, perch zone, and plumage saturation while keeping the raw personality model internal.

- **Interaction events submission endpoint:** This endpoint accepts presence, offers, listen-in, and settle events as events rather than state writes, supporting the event queue and server-authoritative update model.

- **Social invite endpoint:** The invite endpoint exists as the opt-in social path: it generates an invitation link and sends it to the visitor rather than creating a public or mutual social network.

- **Social visit token endpoint:** The plan's why is read-only visiting. The token retrieves the host aviary snapshot but rejects POST interaction calls from that token.

- **Invite revocation endpoint:** Revocation exists so a host can instantly mark an invitation as revoked; active visitor sessions fail on their next snapshot pull.

### Simulation Engine and Sync

- **Server-side tick:** The once-per-minute serialized tick is the point where unresolved events become canonical state. It deduplicates presence, applies drift, updates mood, generates ambient events, writes notebook entries, and marks events processed.

- **Monotonic Drift Function:** The plan uses a low-pass filter calibrated around "3 weeks of regular visits." Its key rationale is no punishment: because delta is never negative, traits "never decay."

- **Mood Transition State Machine:** The rationale is a fast-timescale mood layer, separate from slow personality drift. Time of day, weather, offers, settle, and personality dampeners change mood without rewriting long-term traits.

- **Call-Grammar Runtime:** The plan uses a recursive formal grammar so bird calls are generated procedurally and then modulated by species grammar, `vocal_frequency`, `current_mood`, and `boldness`.

- **Server-Authoritative Delta Writes:** The stated reason is to "prevent multi-device conflicts." The plan rejects last-write-wins absolute values and instead applies deduped event deltas in the server tick.

- **Interpolated Rendering Pipeline:** Because snapshots are pulled at a low frequency, interpolation makes perch movement smooth. If a snapshot is missed, birds stay in the current perch zone's idle loop to prevent "snapping" or abrupt teleports.

### Frontend Rendering, Audio, Accessibility, and Observability

- **HTML5 Canvas rendering:** The plan uses a single canvas "to maintain a high frame rate and allow custom rendering controls."

- **Layer Composition:** NOT RECOVERABLE FROM PLAN

- **Idle Micro-Motion Equations:** The rationale is stated directly: birds are "never statically frozen." Respiration and head tilting keep the scene alive, especially when mood is `curious` or `alert`.

- **Procedural WebAudio Pipeline:** The plan says all sounds are "synthesized procedural audio, avoiding static samples."

- **Tonal synthesis, breathy texture, and envelope shaping:** Tonal synthesis uses LFO vibrato "to simulate natural avian vibrato"; breathy texture mixes filtered white noise with the oscillator; attack times of 10-30ms "prevent clicking."

- **Graceful WebAudio fallback:** The rationale is continuity when audio cannot run. If initialization fails or permissions are denied, audio goes silent and the client turns on call captioning.

- **Call-caption contrast styling:** The plan uses white text on dark charcoal with a ratio greater than 7:1, "well exceeding WCAG AA standards," so captions remain readable.

- **Keyboard navigation path:** The plan creates a predictable route through controls, canvas birds, listen-in, and exit behavior so the core interaction is keyboard-operable.

- **Initial JS Bundle Size:** The 1.5MB gzipped cap and exclusion of heavy framework libraries exist to support the performance budget, especially because rendering is done with vanilla Canvas API calls.

- **Time-to-First-Bird:** The plan targets under 500ms on mid-tier mobile. The backend embeds the initial snapshot so the client can draw the first frame "immediately without waiting for a separate fetch call."

- **CPU and Frame Budget:** The target is "Steady 60fps at <15% CPU load on a 5-year-old laptop," preserving the smooth ambient scene without excessive device cost.

- **Observability aggregate metrics:** The plan collects load durations, first-paint timings, API latency, JS errors, WebAudio failures, and tick durations so system health can be monitored.

- **Observability exclusions:** The rationale is privacy boundary protection. Raw presence durations tied to emails, bird names, chosen motifs, direct interaction history, and PII are excluded from telemetry streams.

- **System Integrity verification stage:** The plan runs simulated 30-day cron scripts to verify the low-pass filter updates traits "smoothly and monotonically."

- **Telemetry Validation stage:** The plan audits metrics pipelines to confirm no PII or raw interaction logs cross into analytics databases.

- **Robotic/Uncanny Synthesized Sounds mitigation:** The plan identifies artificial calls as an immersion risk and mitigates it with FM modulation and low-frequency random perturbations to simulate "minor larynx variations."

- **Sync Race Conditions mitigation:** The plan identifies state regression across laptop and phone as a risk and mitigates it with transactional Redis event logs and a lock on `account_id` to ensure ordering.

- **Drift Calibration Saturation mitigation:** The plan identifies birds reaching maximum warmth or boldness too quickly as a risk and mitigates it by simulation harnesses that tune the low-pass filter time constant.

- **Accessibility Prose Fatigue mitigation:** The plan identifies repetitive narration as a risk and mitigates it with a randomized naturalist vocabulary matrix.
