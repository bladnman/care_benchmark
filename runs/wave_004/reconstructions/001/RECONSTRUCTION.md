## System-level intent

1. Naturalist voice is the main product voice, with named matter-of-fact exceptions.
   This shows up in the field notebook as "naturalist observations", notebook prose as "naturalist, lowercase, present-tense", screen-reader narration as "naturalist prose, lowercase, present-tense", and call captions as "same naturalist voice". The plan also protects operational clarity by naming exceptions: "Account, error, sync, and accessibility-settings surfaces use matter-of-fact voice", with the visit log and sync conflict surfaces also matter-of-fact.

2. The aviary is designed to continue, not reset, when the user is away.
   The plan says "Server-side simulation tick ensures aviary continues without viewer", mood "persists across sessions", and the first frame has "birds mid-action". The risk section rejects a mood that "snaps to neutral on tab open" because the aviary should feel "continued".

3. User attention should shape the birds slowly without becoming stat management.
   The drift function uses a "low-pass filter over presence-and-interaction signals" with drift "measurable in instruments after ~1 week" and "visible to user after ~3 weeks". Traits move "monotonic toward expressive" and "never down on neglect"; personality vectors are "never exposed to user numerically" because "relationship becomes stat management".

4. Canonical state and sync correctness belong on the server.
   The render pipeline says the client "never owns canonical state"; the sync model says the server is "sole writer of personality vectors and mood". The plan repeatedly favors "append-only" events, "additive server-authored deltas", and "no last-write-wins on personality state" to prevent drift from being "lost silently" or simulations from becoming divergent.

5. Accessibility is a designed surface, not a fallback.
   The plan says reduced motion is "a designed surface, not a fallback" and later "not 'animations off'". The accessibility risk mitigation says "Accessibility as first-class design surface, not checklist"; screen-reader narration should not feel like "a checklist", and captions should not feel "tacked-on".

6. Audio and motion should feel procedural, varied, and real-time, not canned.
   The plan requires WebAudio, "procedural calls never looped", "always varied at runtime", "no recorded audio fallback", "gradual mix change (not hard cut)", and "Other birds drop in mix but never go silent". The audio risk names the danger directly: calls should not sound "canned" or like "stacked loops".

7. Social access is intentionally narrow, read-only, opt-in, and privacy-bounded.
   Visit invitations are "read-only, opt-in per invite"; rollout constraints exclude "public discovery or leaderboards"; non-goals exclude "profiles", "follows", "public feed", and "comments on visits". The visit risk mitigation calls for "Clear 'read-only ambient' language" and "no co-presence affordances".

8. Quiet performance and observability are part of the product shape.
   The plan names budgets such as "Time to first bird visible <500ms", "60fps idle motion", "No memory growth", and "Initial JS bundle <2MB". Observability starts with "Synthetic performance checks" and aggregate Real User Monitoring for page load, first-bird-render, render-frame, audio-context errors, and simulation-tick latencies.

## Per-feature whys

### V1 scope and explicit boundaries

- Two starter birds per aviary: NOT RECOVERABLE FROM PLAN

- Max seven birds per aviary: The cap is tied to an "empirical limit for call signature recognizability".

- New birds based on aviary age: The plan says new birds become available based on "aviary age (not visit count, not interaction score)", keeping growth away from "engagement metrics" such as streaks or visit counts.

- Single-user accounts: NOT RECOVERABLE FROM PLAN

- Email magic-link sign-in: NOT RECOVERABLE FROM PLAN

- Multi-device sync via server-side canonical state: The rationale is sync correctness: "All clients read from same canonical state", with "No client-to-client sync", "No client-side state to merge", and "No eventual consistency to reconcile".

- Field notebook with auto-generated naturalist observations: The notebook carries the naturalist voice through "Entry prose (naturalist, lowercase, present-tense)" and rare observations. Frequency is calibrated so entries are not "noise" and not so rare that the "user never sees them".

- Presence accounting for personality drift: Presence-time becomes a signal for "monotonic increase in expressive traits". The risk section says presence must avoid both "false drift" from a left-open tab and "no drift" for users who watch without moving, using visibility, focus, and recent activity.

- Listen-in interaction: Listen-in focuses "a single bird's audio" and feeds drift through "Listen-in duration -> social warmth, vocal frequency". Audio rationale is a "gradual mix change (not hard cut)" while other birds "never go silent".

- Offer interaction: Offers feed personality and mood changes: "Offer acceptance -> curiosity", "Offer attempt near bird -> boldness", and "offer accepted -> toward content".

- Settle gesture: The plan frames settle as a "soft session end with evening lighting". It also appears as a user-initiated event that can change narration cadence and as an event type in the append-only log.

- Visit invitations: Visits are "read-only, opt-in per invite" to avoid misuse where "users interpret read-only visit as co-presence", visitors expect to interact, or hosts expect notifications. Mitigations are "read-only ambient" language, "no co-presence affordances", opt-in visit notifications, and matter-of-fact expired/revoked surfaces.

- Screen-reader narration in naturalist prose: Narration is generated from the "same state as visual surface" and uses the "same voice as field notebook" so it does not feel like "a checklist". Slow cadence preserves the idle surface; faster cadence is reserved for user-initiated events.

- Reduced-motion mode for accessibility: Reduced motion is a "designed surface, not a fallback": it cross-fades between still poses, removes ambient leaf drift, keeps ambient color shifts slowed, and keeps calls "at full quality".

- Call captioning: Captions are accessibility content in the "same naturalist voice"; they are "Runtime-generated per call (not fixed strings)", appear near the calling bird, and become default when WebAudio falls back to graceful silence.

- Day/night cycle tied to user's local time: The time-of-day signal drives mood and atmosphere: birds become "drowsy near dusk" and "alert morning", with visual day/night transitions.

- Ambient weather with mood effects: Weather directly shapes mood and calls: "rain dampens vocal frequency" and "wind increases alertness". Weather also provides notebook context such as "after rain" and transitions softly with "no thunderstorms".

- Native mobile apps out of scope, web-only: NOT RECOVERABLE FROM PLAN

- No gamification: The plan keeps growth away from "visit count" and "interaction score", excludes "engagement metrics", and avoids surfaces such as achievements, streaks, levels, scores, badges, and "green dots".

- No Tamagotchi mechanics: The articulated rationale is that birds do not punish neglect: traits are "monotonic toward expressive" and "never down on neglect"; the scope excludes death, hunger, distress, and "happiness meter decay".

- No social network surfaces: The rationale is consistent with read-only ambient visits: no profiles, follows, public feed, comments, public discovery, or leaderboards, and "no co-presence affordances".

- No notifications about aviary activity: The plan constrains rollout with "No push notifications" and "No email about aviary activity"; for visits, host expectations are managed with opt-in visit notifications and matter-of-fact visit surfaces.

- Named exceptions to naturalist voice: NOT RECOVERABLE FROM PLAN

### Architecture, data, and API surface

- Single-page web application: NOT RECOVERABLE FROM PLAN

- Stateful Node.js service: The service owns canonical state, simulation tick, append-only event log, session token management, and email operations; this supports the server-side simulation and sync model.

- REST API for state snapshots and event submission: The API shape supports clients pulling canonical snapshots and submitting events without owning canonical state.

- Server-side simulation tick once per minute: The tick reads events, computes presence-time, applies drift, updates mood, advances weather, generates notebook entries, writes canonical state, and ensures the aviary "continues without viewer".

- Event log append-only store: Events are appended and consumed by the tick in order, enabling "additive server-authored deltas" and preventing personality drift from being overwritten by client absolute values.

- Session token management: NOT RECOVERABLE FROM PLAN

- Email service for magic links and account operations: NOT RECOVERABLE FROM PLAN

- Client rendering of aviary scene, birds, perches, and ambient motion: Client rendering enables smooth visible behavior while canonical bird, mood, and personality state remain server-owned.

- Client interpolation between snapshots: The plan states interpolation is for "smooth motion".

- Client procedural audio synthesis via WebAudio: The plan avoids downloaded or pre-recorded audio, supports real-time calls, keeps bundle size down, and avoids "canned audio concerns".

- Client presence detection and pings: Presence detection uses visibilityState, focus, pointer, and keypress to accumulate presence-time while mitigating left-open tabs and watch-without-moving sessions.

- Client reduced-motion rendering: The client handles reduced motion as a "designed surface", cross-fading still poses instead of turning animation into a degraded fallback.

- Account data with synthetic UUID and encrypted email stored once: The privacy rationale is reinforced by the risk mitigation: synthetic UUIDs are internal identifiers, email is encrypted and stored once, and telemetry avoids PII.

- Soft-delete flag, restore window, export, email change, session listing, and session revocation: NOT RECOVERABLE FROM PLAN

- Bird stable internal UUID, user-assigned name, species, adoption timestamp: The rationale stated for the UUID is stable "identity, never reused".

- Bird personality vector: Personality drives drift, perch position, call timing, pitch richness, and mood. The plan hides the numbers so personality is "felt by watching bird" rather than managed as stats.

- Bird mood and perch position: Mood makes behavior legible: wary birds move to the back perch and scan, content birds preen, curious birds investigate, and drowsy birds sit low and fluffed.

- Presence model: The plan uses session start, last activity, visibilityState, focus, and presence-time so drift is based on observed presence rather than simple tab-open time.

- Notebook entries with context: Context such as "morning", "evening", and "after rain" grounds naturalist prose in time and ambient conditions.

- Visit invitation token, expiry, revocation, and log: Tokens are one-time use and expire after 30 days; revocation and host-only logs support opt-in, read-only visit control.

- State retrieval endpoint: It returns canonical birds, positions, moods, personality vectors, call timing, weather, day/night phase, notebook entries, and settings so clients render from server truth.

- Event submission endpoint: It appends interaction events and gives no per-event response, matching the append-only event log model consumed by the tick.

- Presence endpoint: The endpoint records presence only when "all three conditions" are met, supporting calibrated presence-time.

- Account operation endpoints: NOT RECOVERABLE FROM PLAN

- Visit invitation endpoints: They issue, list, revoke, log, and return read-only visitor state snapshots, matching opt-in read-only visits and matter-of-fact visit management.

### Simulation engine and sync model

- Drift function: The low-pass filter calibrates change so it is "measurable" after about a week and "visible" after about three weeks, preventing both too-fast session jumps and too-slow user indifference.

- Monotonic expressive drift: Traits move upward on positive presence and "never down on neglect", matching the no-Tamagotchi boundary and avoiding decay mechanics.

- Mood transitions: Mood is "Fast-timescale, resets daily-ish" and shaped by time of day, weather, recent interactions, and personality, giving birds stateful behavior without exposing numeric personality.

- Mood persistence across sessions: The plan explicitly mitigates the risk that the aviary feels "reset, not continued" by persisting mood and avoiding a "wake up" animation.

- Ambient weather advancement: Weather passes through rain and wind events and feeds mood, vocal frequency, alertness, notebook context, and soft scene transitions.

- Notebook entry generation: Entries are rare, about "one every few days for active users", with risk mitigation for too-frequent noise and too-rare invisibility.

- Event log window clearing after tick: The plan says to "keep only recent for debugging".

- Server as sole writer of personality vectors and mood: This prevents drift from being lost silently by last-write-wins and avoids divergent client simulations.

- Additive server-authored deltas: The plan uses deltas rather than client-submitted absolute values to prevent conflicts and preserve ordered drift.

- Client pulls fresh snapshots on visibility change, long render-frame gaps, and low-frequency keepalive: The rationale is to keep client rendering aligned with canonical state after visibility changes, laptop resume, and ongoing visible sessions.

### Frontend rendering pipeline

- Single horizontal scene, no panning or scrolling: NOT RECOVERABLE FROM PLAN

- Three perch zones: Perches make mood and personality visible through front, middle, and back positions, including wary birds on the back perch and drowsy birds on the low perch.

- Ambient background and foreground elements: NOT RECOVERABLE FROM PLAN

- Subtle parallax, not parallax-heavy: NOT RECOVERABLE FROM PLAN

- Continuous micro-motion regardless of user attention: The plan uses idle motion to support an aviary that continues whether or not the user is actively interacting.

- Mood-shaped idle motion: Mood is visible through behavior: wary scans, content preens, curious investigates, and drowsy sits low.

- Day/night, settle, listen-in, and weather transitions: Transitions are gradual and soft: settle moves to evening lighting over seconds, listen-in rises and drops gradually, and weather uses soft transitions with no thunderstorms.

- Loading sequence with birds mid-action: The rationale is continuity: no entry animation, no fade-from-static, and the first frame has birds mid-action because the server-side tick keeps the aviary going without a viewer.

- Quiet field instead of spinner when snapshot is delayed: NOT RECOVERABLE FROM PLAN

- Initial bundle, first-bird, frame-rate, and memory budgets: Budgets enforce quick visibility, 60fps idle motion, long-session stability, and no recorded audio.

### Audio pipeline

- Procedural call synthesis: WebAudio, motif libraries, personality-shaped timing and pitch, mood-shaped variation, and no looped calls are used so calls remain varied at runtime and avoid sounding canned.

- Chorus mixing: Real-time mixing and gradual listen-in changes avoid hard cuts; other birds drop in mix but "never go silent" so the aviary remains a chorus.

- WebAudio fallback: If WebAudio is unavailable, the plan chooses "graceful silence" with captions on by default, avoiding recorded fallback because of "bundle budget and canned audio concerns".

### Accessibility surfaces

- Screen-reader narration cadence: Slow idle updates and faster user-initiated updates keep narration aligned with the aviary's pace while still responding to return-greetings, offers, and settle.

- Reduced-motion preference and setting: `prefers-reduced-motion` plus opt-in settings let the reduced-motion surface be explicitly available while preserving calls and narration.

- Call captions near calling bird: Captions appear near the calling bird and fade with the call, keeping caption timing and placement tied to runtime calls.

- WCAG AA contrast and focus indicators: Text must pass WCAG AA, focus indicators must remain visible against the aviary background, and keyboard focus gets a high-contrast outline.

- Keyboard navigation: Keyboard access reaches top bar items, bird focus, listen-in, offer, and settle so the core interactions are not pointer-only.

### Performance, observability, rollout, and risks

- Synthetic performance checks and aggregate Real User Monitoring: These measure page load, first-bird render, render-frame timings, audio-context errors, and simulation-tick latencies against named budgets.

- Simulation-tick p99 alarm above five seconds: The alarm protects the once-per-minute server tick from excessive latency.

- Last-two-major browser support: The plan limits support to recent Chrome, Safari, Firefox, and Edge; older browsers receive matter-of-fact unsupported surfaces.

- Instrumentation from day one: Presence-time, personality drift, session duration histograms, render timing, tick latency, audio errors, and aggregate visit usage are needed for drift calibration, performance, and usage without per-account PII.

- Versioned data model and migrations: These mitigate backward-compatibility risks where data model changes break accounts or tick changes corrupt drift history.

- Telemetry privacy boundary: Telemetry never touches the simulation database, ML training never receives per-bird fields, and per-account dimensions are avoided for anonymized session duration histograms.
