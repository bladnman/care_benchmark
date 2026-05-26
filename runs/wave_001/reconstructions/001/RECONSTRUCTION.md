## System-level intent

- **Anti-gamification, age-over-attention care.** The plan repeatedly protects the aviary from becoming a points or habit loop: v1 excludes "achievements, streaks, levels, scores, badges, XP," excludes "Tamagotchi mechanics," and makes new bird offers appear at fixed calendar intervals because "age--not attention--unlocks birds." The drift model also says "Neglect produces zero delta; traits never decrease," so absence is not punished.

- **Watching without moving is the product.** The presence window decision explicitly says "watching without moving is the product" and should not make "still observers lose presence." That intent carries into passive presence accounting, presence-time as the dominant drift input, and low-pressure interactions where the user can simply remain with the aviary.

- **Aviary as place, not app.** The risk section says one spinner, canned greeting, "Welcome back!" toast, achievement pop-up, or hard-cut transition can collapse the product from "place" to "app." This shows up in first-frame discipline ("No spinner," "No fade-in, no 'wake up' sequence"), in smooth lighting and audio transitions, and in review rules like "No toast greetings" and "No gamification language."

- **Server-owned inner life, thin renderers.** The plan's canonical-state principle is that the server holds the "only canonical copy" and clients are "thin renderers." Raw personality vectors are "never exposed to clients," only the simulation tick writes personality and mood, and the event log is the "only thing clients write." The stated why is to avoid a sync problem: "there is no client state to merge."

- **Slow, visible personality drift without decay.** Drift is a "low-pass filter" tuned so instruments detect change after about one week and a returning user can "feel a difference" after about three weeks. The risk section names the product boundary: too fast feels like a Tamagotchi; too slow feels like "a wallpaper."

- **Naturalist voice over app voice.** The field notebook, screen-reader narration, and call captions all use "naturalist" prose, often "lowercase" and present-tense. The plan separates "naturalist" voice components from "matter-of-fact" components, reserving matter-of-fact wording for error surfaces like "Your session timed out. Sign in again to keep watching."

- **Accessibility is a first-class surface.** The plan calls reduced motion a "designed surface, not a fallback," says it ships with v1, and names accessibility as a "launch blocker." This intent appears in screen-reader narration, call captions, keyboard navigation, reduced-motion cross-fades, and WCAG AA contrast.

- **Privacy-respecting observability.** Telemetry is aggregate-only and contains "zero per-bird state, zero per-account interaction history, zero personality vector values." The plan also says not to measure per-bird drift dashboards, per-account "engagement scores," individual offer preferences, or gamified A/B variants.

- **Rich aliveness inside strict performance budgets.** The plan ties rich client motion to small snapshots, <500ms first bird visible, 60 fps idle motion, <5 KB snapshots, no memory growth, and <2 MB initial JS. Performance is not generic polish; it protects the product's first impression and long ambient sessions.

## Per-feature whys

### 1. Scope

- **Single-user accounts, email magic-link sign-in, per-device revocable session tokens:** NOT RECOVERABLE FROM PLAN

- **One aviary per account:** NOT RECOVERABLE FROM PLAN

- **Two starter birds from a pool of about 6 species:** NOT RECOVERABLE FROM PLAN

- **Hard cap of 7 birds:** The rollout validates "7-bird cap audio recognizability" and uses staged bird caps to validate "audio recognizability and rendering performance at each density before the full cap."

- **Server-side simulation tick around once per minute:** The plan narrows "~once per minute" to "60 seconds" with "+/-5s jitter" to avoid a "thundering herd." The tick also keeps personality, mood, weather, and time-of-day server-authored.

- **Real-time client rendering of the aviary scene:** The render boundary keeps server snapshots "small (kilobytes)" while allowing "rich visual motion" at 60fps.

- **Return-greeting interaction:** NOT RECOVERABLE FROM PLAN

- **Listen-in interaction:** The audio mix gives the focused bird a gain ramp while other birds fade but are "never silent," preserving ambient presence instead of isolating a single sound.

- **Offer interaction (seed / song fragment / still pool):** The plan says offer acceptance gives "+content" and contributes a "small" input to drift, but a separate product rationale for the offer feature is NOT RECOVERABLE FROM PLAN

- **Settle interaction:** The plan defines settle as causing "+settled," a lighting shift to an evening palette, and quieter calls, but a separate product rationale for the settle feature is NOT RECOVERABLE FROM PLAN

- **Passive presence accounting:** Presence is the dominant drift input because "watching without moving is the product."

- **Field notebook:** The notebook uses auto-generated, read-only "naturalist prose" and sparse entries, keeping the surface observational rather than editable user content.

- **Visit invitations:** Visits are "opt-in," "per-invite," "read-only," revocable, expiring, and silent by default. This keeps visits from becoming the excluded social-network surfaces: no profiles, public discovery, comments, leaderboards, rankings, or "show-off" visitor mode.

- **Accessibility surface:** Screen-reader narration, reduced-motion cross-fade rendering, call captioning, keyboard navigation, and WCAG AA chrome are included because accessibility is not a v1.1 feature; it is a launch blocker and first-class rendering path.

- **Multi-device sync by architecture:** The plan's why is explicit: a single canonical server state means "no client-to-client sync" and "no merge logic."

- **Fixed calendar bird adoption pacing:** New bird offers appear at fixed intervals because "age--not attention--unlocks birds," aligning with the "anti-gamification stance."

- **Six mood states (wary, content, curious, drowsy, alert, settled):** The rationale is to cover the stated examples and add "settled" as an explicit evening/night state.

- **Three-minute presence pointer/key activity window:** The window leans longer so still observers do not lose presence; the plan says "watching without moving is the product."

- **Weather frequency of 2-4 rain events and 1-2 wind events per week:** The why is that weather should be "rare" and "soft," "noticeable but never dominant."

### 2. Architecture

- **Client/server split with server canonical state:** The server stores, updates, and protects personality and mood while the client reads snapshots and interpolates. The why is to keep clients as renderers and prevent client-side merge problems.

- **State snapshots instead of streamed animation frames:** Snapshots of bird positions, moods, animation states, and weather flags stay small while the client interpolates "for 60fps motion."

- **Personality vectors hidden from clients:** Raw values are protected on the server; API responses contain only derived rendering hints such as motion style and call timing.

- **Client-side WebAudio calls from motif grammar:** The server stores the grammar seed "not audio," keeping audio procedural and lightweight.

- **Event log as the only client write path:** Clients emit events; the server appends them and the tick consumes them in order. This supports ordered, additive simulation updates instead of direct client mutation.

- **Notebook entries generated server-side:** Entries come from simulation history through the tick or notebook worker and remain read-only, reinforcing the naturalist observation surface.

### 3. Data Model

- **Account email encryption:** NOT RECOVERABLE FROM PLAN

- **Account settings for reduced motion, call captions, and visit notifications:** Reduced motion and captions support the accessibility surfaces. Visit notifications default false because visit logs are silent by default and event emails are excluded from v1.

- **Session device description and revocation:** NOT RECOVERABLE FROM PLAN

- **Stable bird identity:** `bird_id` is "immutable across renames, syncs, and migrations," so a bird remains the same entity even when its name or surrounding state changes.

- **Server-owned personality vector and mood:** Only the simulation tick writes them, preserving the server-authored inner life and preventing clients from sending absolute values.

- **Derived render hints in snapshots:** Render hints let clients show personality-shaped behavior without exposing raw vector values.

- **Append-only event log:** The log is processed in `server_ingested_at` order and is central to conflict prevention and vector integrity.

- **Presence derived from contiguous pings rather than a separate table:** NOT RECOVERABLE FROM PLAN

- **Field notebook trigger types:** The plan names drift milestones, notable moods, weather, offer reactions, and daily observations as entry triggers, tying prose to simulation history.

- **Active visits pulling host snapshot read-only:** The guest view remains ambient and read-only rather than social or editable.

### 4. API Surface

- **Magic-link request endpoint:** The plan says it generates and emails a link and is rate-limited per email, but a feature rationale is NOT RECOVERABLE FROM PLAN

- **Magic-link verify endpoint:** NOT RECOVERABLE FROM PLAN

- **Session revoke and session list endpoints:** NOT RECOVERABLE FROM PLAN

- **Snapshot fetch triggers:** Initial load gets the scene started; visibility changes and long frame gaps pull fresh state because "the aviary has been ticking while away" and laptop sleep/resume can leave the client stale. Visible/focused keepalive preserves current state.

- **Interaction event endpoint:** It appends events for the tick to consume, keeping the client on the allowed event-write path.

- **Presence endpoint:** It provides the lightweight pings used to compute presence-time for drift.

- **Notebook pagination:** The endpoint supports sparse, indefinite scrollback, but a separate rationale for pagination itself is NOT RECOVERABLE FROM PLAN

- **Account email change, export, soft delete, and delete cancel endpoints:** NOT RECOVERABLE FROM PLAN

- **Visit invite, revoke, log, and guest endpoints:** The endpoints implement opt-in, revocable, email-based ambient visits and let hosts manage the silent visit log.

- **Bird rename-only endpoint:** Names are "user-facing cosmetic metadata with no simulation impact," so last-write-wins is acceptable and no other mutable fields are exposed.

### 5. Simulation Engine Design

- **Stateless horizontally scalable tick worker:** Stateless workers can be containerized, sharded by account hash, and scaled while each account's events remain ordered.

- **Idempotent ticks:** Given the same previous state and event batch, the tick produces the same next state, reducing sync and corruption risk.

- **Tick inputs from previous state, unprocessed events, actual time, and user timezone:** These inputs let mood, vectors, weather timers, and time-of-day nudges advance from canonical state rather than client guesses.

- **Tick outputs of updated state, zero or one notebook entry, and processed events:** The outputs make the tick the single place where simulation, prose generation, and event consumption move forward.

- **Drift as a low-pass filter:** The plan uses low-pass drift so presence and interactions accumulate slowly rather than producing a game-like meter.

- **One-week and three-week drift calibration:** The why is testability: instruments should detect change after a week, while after about three weeks a returning user can "feel a difference" without being told.

- **Presence-time as dominant drift input:** This reinforces the plan's intent that watching counts and stillness is valid participation.

- **Monotonic drift rule:** Neglect produces zero delta and traits never decrease, avoiding punishment or Tamagotchi-style decay.

- **Mood transition state machine:** Mood responds to time of day, weather, recent interaction, personality, and nearby birds, giving the aviary ambient continuity and local reactions.

- **Mood persistence across sessions:** Mood from the last tick starts the next tick and there is "No reset on session open," so opening the app does not restart the birds.

- **Call-grammar runtime:** Motif sequences plus personality and mood variation make calls procedural, shaped by the bird, and not pre-baked loops.

- **`call_timing_next_ms` hint:** The hint lets the client schedule calls locally while the server remains the source of derived timing.

- **Chorus mixing:** Simultaneous calls are mixed as "two distinct sounds, not a layered loop," with panning and compression to avoid hard-left/right placement and clipping.

- **Perch selection:** Mood, boldness, and recent offers decide target zones, so where a bird sits reflects its current state; the server emits only target zones while the client animates transitions.

- **Weather generation:** Low-probability rolls and one active weather type keep weather within the "rare" and "soft" pacing.

### 6. Sync Model

- **Canonical server state principle:** "There is no sync problem because there is no client state to merge."

- **On-load, visible, long-gap, and visible keepalive snapshot behavior:** The client refreshes when it first starts, when it returns, when sleep/resume is detected, and while active so it stays aligned with the server-ticked aviary.

- **Fire-and-forget interaction events with retry:** Events go to the server immediately and retry on network or 5xx failure, preserving the append-only event path.

- **Stopping rendering, presence pings, and pulls while hidden:** The plan says simulation continues server-side, but a separate feature rationale for stopping these client behaviors is NOT RECOVERABLE FROM PLAN

- **Conflict prevention for personality, mood, bird names, account settings, and notebook entries:** Personality and mood stay server-authored; names and settings can be last-write-wins because names are cosmetic and settings are small and independent; notebook entries are append-only and server-generated.

- **Multi-device coherence through shared snapshots:** Devices may differ only in interpolation or event timing; ordering uses `server_ingested_at`, not client timestamp.

- **Outage and offline behavior:** Rendering from the last snapshot, queueing events locally, and pulling fresh state on reconnect keep the aviary usable without client catch-up simulation.

- **Matter-of-fact session timeout error:** The wording keeps error surfaces plain: "Your session timed out. Sign in again to keep watching."

### 7. Frontend Rendering Pipeline

- **Single full-viewport canvas:** The scene is the primary surface; top bar chrome stays in HTML "for accessibility and interaction."

- **2D Canvas API preferred for v1:** NOT RECOVERABLE FROM PLAN

- **Layered scene composition:** Back-to-front layers let lighting, weather, perch depth, birds, and chrome coexist while keeping accessibility chrome out of canvas.

- **First frame with birds in mid-action:** The first seen state must feel already alive. If data is slow, the loading state is a "quiet field" and "No spinner."

- **No fade-in or wake-up sequence:** Birds appear at their current positions and motion states so motion "continues as if it had always been running."

- **Idle micro-motion by mood:** Wary, content, curious, drowsy, alert, and settled states each have distinct procedural idle motion, making mood visible without meters.

- **Procedural motion seeded by personality:** High boldness and other traits shape poses and motion, making inner state visible as behavior.

- **Smooth perch, mood, lighting, listen-in, and settle transitions:** The plan avoids hard cuts through bezier flights, gradual blends, continuous gradients, volume ramps, and quieting calls.

- **Settle undoable within 5 seconds by any click:** NOT RECOVERABLE FROM PLAN

- **Reduced-motion mode:** Animation becomes calm cross-fades, ambient drift is removed, and calls still play or caption; this is "a designed surface, not a fallback."

- **Responsive layout:** The scene preserves aspect ratio, compresses or spreads perch zones, keeps birds visible, and supports a 320px minimum viewport so birds never crop offscreen.

### 8. Audio Pipeline

- **Compact species motif libraries:** Each species library is under 20KB and all six must fit within the 2MB JS bundle budget, which is why procedural synthesis is required.

- **Personality and mood variation in calls:** Vocal frequency, mood, and boldness shape gaps, pitch, attack, and gain so calls reflect the bird's state.

- **Separate audio graph branches for simultaneous calls:** Multiple calls remain distinct sounds; compression prevents clipping during chorus events.

- **Listen-in mix:** Focused gain rises, other birds fade but are "never silent," and ambient sound remains audible at low level.

- **WebAudio fallback to silence plus captions:** If audio is unavailable, silence is preferred "to canned audio," while captions keep the aviary functional.

- **Pooled audio buffers:** Buffers are reused so there is "No memory growth" and no new `AudioBuffer` per call.

- **Lazy audio context initialization on first user gesture:** This exists because of browser autoplay policy.

### 9. Accessibility Surfaces

- **Screen-reader live narration:** A polite live region receives naturalist prose updates so nonvisual users get the same aviary voice.

- **Narration cadence and dropped updates:** Idle updates every 30-60s and dropped queued updates prevent screen-reader backlog.

- **Call captions:** Captions are generated from the motif sequence "actually played," not pre-written strings, so they describe the real call in naturalist lowercase.

- **Keyboard navigation:** Keys cover top bar, bird focus, listen-in, offer, settle, and escape behavior, making the core scene keyboard-operable.

- **Focus indicator:** The outline color and shadow are specified to ensure visibility "against all aviary backgrounds."

- **WCAG AA contrast on chrome:** Contrast applies to top bar, settings, captions, and error surfaces because those are the user-copy text surfaces.

### 10. Performance Budgets and Observability

- **Initial JS bundle budget with CI failure:** The <2 MB budget is enforced and code-splitting is used so the first aviary experience stays fast.

- **Time to first bird visible budget:** The <500ms target protects first-frame discipline; synthetic monitoring alarms above budget.

- **60fps idle motion and no memory growth over 30 minutes:** These budgets protect long ambient sessions and avoid degradation over time.

- **Snapshot payload budget:** Keeping snapshots under 5KB supports frequent pulls and small server-client state transfer.

- **Audio context init latency budget:** The first user gesture should not make audio feel laggy.

- **Procedural assets and lazy species/UI bundles:** Procedural SVGs and compact drawing routines avoid large bitmap atlases; lazy loading keeps non-core flows out of the initial bundle.

- **Synthetic checks:** Automated browsers measure TTFB, first-bird render, frame rate, and audio init from three geographies to catch budget regressions.

- **Aggregate-only RUM:** The plan gathers timings and error counts while preserving the privacy boundary.

- **Server operational metrics:** Tick latency, event ingest, backlog depth, snapshot latency, and DB saturation are measured to keep the simulation reliable.

- **Deliberately unmeasured analytics:** The plan avoids per-bird drift dashboards, engagement scores, individual preferences, and gamified A/B variants to protect the product stance.

### 11. Rollout

- **Alpha:** Internal team and friends validate tick calibration, audio mix, and first accessibility regressions.

- **Closed beta:** 500 waitlist users measure drift at real-world scale, tune presence window, fix sync edge cases, and validate the 500ms budget.

- **Open beta:** Public invite-only access load-tests tick worker scaling, validates 7-bird audio recognizability, and monitors memory growth.

- **General availability:** Full release follows the staged validation.

- **Birds-per-aviary ramp:** Caps of 3, then 5, then 7 birds let the team validate audio recognizability and rendering performance at each density.

- **Instrumentation from day one:** Synthetic monitoring, aggregate RUM, error alerting, and accessibility audits are active before users so regressions are caught early.

- **Success metrics not user-facing:** Retention and session duration are observed but not surfaced to users, preserving the no-gamification stance.

### 12. Risks

- **Drift calibration mitigation:** A fast-forward harness, measurable simulated change, qualitative drift review, and server-side `global_drift_rate` tuning address the risk that drift feels too fast like Tamagotchi or too slow like wallpaper.

- **Sync correctness mitigation:** Append-only logs, monotonic ingest order, delta audit logs, vector integrity checks, and a small soft launch address invisible personality corruption.

- **Audio uncanniness mitigation:** Audio-engineer ownership, blind listening tests, mobile performance tests, and reducing active voice count address loops, lost recognizability, muddy chorus, and WebAudio degradation.

- **Accessibility regression mitigation:** CI axe-core tests, manual VoiceOver/NVDA passes, and synthetic testing of reduced-motion and keyboard paths support the launch-blocker stance.

- **"Feels alive" review and lint mitigation:** Checklists, product sign-off, banned app/gamification strings, and voice-register declarations protect the aviary from becoming "app" instead of "place."

- **Performance at scale mitigation:** Sharding by account, p99 tick alarms, batch-tick safety valves, and separate read replicas address event spikes and worker backlog.

- **Privacy boundary mitigation:** Separate telemetry schemas, privacy sign-off, no simulation DB credentials for analytics, and privacy audits address accidental telemetry overreach.
