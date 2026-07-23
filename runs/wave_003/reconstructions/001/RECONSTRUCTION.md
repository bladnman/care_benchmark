## System-level intent

- **Calm, quiet presence rather than engagement pressure.** This appears in the Executive Summary as a "calm, browser-based virtual aviary" shaped by "quiet presence and subtle interactions." It also shows up in the Scope and Non-Goals through "No Gamification," no "visit counters," and adoption paced by "aviary age" rather than "visit count or engagement metrics."
- **Personality should become more expressive slowly and never punish absence.** The plan says personalities "drift over weeks," with trait values moving "monotonically upward toward expressive limits" and "never decay upon absence." The non-goals reinforce this through "No Tamagotchi Mechanics" and "ambient quietness without negative numerical drift."
- **The product voice is naturalist, soft, and non-announcing.** This shows up in "non-announcing greeting," "read-only naturalist prose log," and the screen-reader tone: "Lowercase, present-tense, specific" while strictly refraining from "UI announcement phrasing or state dumps." Public errors use a "matter-of-fact register."
- **The server is the authoritative simulator and state writer.** The architecture states that the server is the "sole authoritative simulator and state writer." Synchronization repeats this as the "Canonical Single-Writer Model," with clients never submitting "absolute personality values."
- **State changes should be conflict-free across devices.** The plan carries this through "Conflict-Free Append-Only Logging," sequential processing by the tick worker, and the claim that this "entirely eliminates Last-Write-Wins (LWW) data loss."
- **Privacy boundaries are part of the product architecture.** The plan uses "synthetic account UUIDs," encrypted email PII, account export, a "30-day soft deletion window," and telemetry with "Zero collection or aggregation of PII, email addresses, individual bird vector states, or per-user interaction logs."
- **Accessibility is a primary surface, not an add-on.** The plan includes screen-reader narration, reduced-motion mode, procedural call captions, keyboard navigation, and "WCAG AA contrast" in scope, then gives each its own implementation surface.
- **Procedural freshness and graceful fallback preserve immersion.** The plan prefers procedural rendering and sound: "procedural ambient micro-motions," "procedural bird calls," "Micro-Variation" to guarantee "procedural freshness," and no recorded loops to prevent "repetitive audible patterns." Fallbacks are calm: an "empty sky field," "graceful silent mode," and captions enabled by default.

## Per-feature whys

### Scope & Non-Goals

- **Single Page Web Application targeting modern desktop and mobile web browsers:** The plan frames delivery as browser-only and later excludes "Native Apps" so there is no "iOS/Android client codebase or native protocol abstractions."
- **2 starter birds, a 6-species pool, age-based additions, and a strict cap of 7 birds:** The Executive Summary calls for "a small handful" of birds. Growth is paced by "aviary age," "never by visit count or engagement metrics," keeping capacity bounded and non-gamified.
- **Return-Greeting:** The plan's stated rationale is the tone and arrival behavior: it is "procedurally varied," "non-announcing," staggered when multiple birds respond, and "scaling with absence duration."
- **Listen-in:** The rationale is to "focus single bird" while rebalancing the audio mix "smoothly without muting background birds"; later the mix ramp says non-targeted birds are "never fully muted."
- **Offers:** NOT RECOVERABLE FROM PLAN
- **Settle:** The rationale given is a "user-initiated soft session end" with an "evening lighting transition" and a "5-second undo window."
- **Field Notebook:** The plan's rationale is a "sparse" and "read-only naturalist prose log," generated as "1 entry every few days" rather than a dense activity feed.
- **Presence Accounting:** The rationale is strict validation of real presence through concurrent `visibilityState === 'visible'`, window focus, and recent pointer/keyboard activity in a "3-minute activity window."
- **Optional read-only visit invitations:** The rationale is bounded social access: invitations are "Optional," "read-only," revocable, "auto-expiring in 30 days," and paired with "silent visit logging for host."
- **Magic-link email auth, one canonical aviary, JSON export, and soft deletion:** The plan ties these to account control: "single canonical aviary per account," "JSON account data export," and a "30-day soft deletion window."
- **Screen-reader naturalist prose narration:** The rationale is an accessibility surface that uses `aria-live="polite"` and naturalist prose rather than UI state dumps.
- **Reduced-motion mode:** The rationale is accessibility: reduced motion replaces "micro-motion/drift paths" with "slow cross-fades" while preserving daylight transitions and audio quality.
- **Procedural call captions:** The rationale is an accessibility surface for vocalizations, generated from call grammar and also enabled in silent fallback.
- **Keyboard navigation and WCAG AA contrast:** The rationale is complete keyboard traversal and focus/contrast across "light midday and dark night aviary palettes."
- **No Native Apps:** The plan's rationale is web-browser-only scope, with no "iOS/Android client codebase or native protocol abstractions."
- **No Gamification:** The rationale is the "absolute exclusion" of achievements, levels, scores, badges, streaks, calendars of visits, XP, ranks, and visit counters.
- **No Tamagotchi Mechanics:** The rationale is avoiding bird mortality, hunger, distress, happiness decay, and negative drift; neglect produces only "ambient quietness."
- **No Social Network Infrastructure:** The rationale is avoiding profiles, friend lists, feeds, discovery directories, leaderboards, co-presence, comments, and chat.

### Architecture & Service Boundaries

- **Decoupled client-server architecture:** The rationale is a split where the server is the "sole authoritative simulator and state writer" and the client is a "render-and-synthesis engine."
- **Authentication & Identity service:** The rationale is issuing magic links, verifying tokens, generating "synthetic account UUIDs," and managing "encrypted PII storage."
- **Authoritative Simulation Engine:** The rationale is processing event logs, updating personality vectors, transitioning moods, advancing daylight/weather, and writing "canonical state snapshots."
- **API & Data Access Layer:** The rationale is to serve "lightweight aviary snapshots," append interaction events, handle exports, and manage invitations.
- **Client Render Engine:** The rationale is "60fps interpolation between server state snapshots" plus procedural ambient micro-motions.
- **Client WebAudio Audio Runtime:** The rationale is real-time synthesis of procedural calls and listen-in mix modulation.
- **Presence & Accessibility Runtime:** The rationale is monitoring presence conjunction signals and driving the live region plus captions.

### Data Model & Schema Design

- **Synthetic `account_id` references and encrypted email PII stored only in `accounts`:** The rationale is separating account identity from PII, with email encrypted at rest and email hash used for lookup.
- **`accounts` status and `deleted_at`:** The rationale is support for `ACTIVE`, `PENDING_DELETION`, and the "30-day soft deletion window."
- **`birds` table:** NOT RECOVERABLE FROM PLAN
- **`personality_vectors` table with bounded traits:** The rationale is server-authoritative personality drift across boldness, social warmth, vocal frequency, plumage saturation, and curiosity, each constrained between 0.0 and 1.0.
- **`bird_moods` table:** NOT RECOVERABLE FROM PLAN
- **Append-only `interaction_events` log:** The rationale is storing presence pings, listen-in windows, offers, and settle events for sequential server processing without overwriting vectors.
- **`notebook_entries`:** The rationale is storing field notebook "naturalist prose" entries tied to an account and optionally a bird.
- **`visit_invitations`:** The rationale is revocable and expiring social invitations using hashed visitor email and hashed invite token.
- **`visit_logs`:** The rationale is "silent visit logging for host" while storing `visitor_email_masked`.

### API Surface & Protocols

- **HTTPS with JSON payloads and matter-of-fact public errors:** The rationale is a public-surface register that is "matter-of-fact."
- **`POST /api/v1/auth/magic-link`:** The rationale is sending the same `{ "status": "sent" }` response "regardless of email existence to prevent enumeration."
- **`POST /api/v1/auth/verify`:** NOT RECOVERABLE FROM PLAN
- **`GET /api/v1/account/export`:** The rationale is account data export through a signed URL for JSON download.
- **`DELETE /api/v1/account`:** The rationale is marking the account `PENDING_DELETION` to initiate the "30-day recovery window."
- **`GET /api/v1/aviary/snapshot`:** The rationale is returning the current "canonical aviary state" for the client to render.
- **`POST /api/v1/aviary/events`:** The rationale is accepting client interaction events for presence pings, offer actions, listen-in windows, and settle triggers.
- **`POST /api/v1/invitations`:** The rationale is allowing a host to issue an invitation to a visitor email.
- **`DELETE /api/v1/invitations/:invite_id`:** The rationale is immediate host revocation.
- **`GET /api/v1/visits/:token`:** The rationale is resolving a token to a read-only aviary snapshot and returning matter-of-fact errors for expired or revoked tokens.

### Simulation Engine & Drift Runtime

- **Server simulation tick every approximately 60 seconds:** The rationale is asynchronous processing per active aviary of presence/events, drift, mood transitions, daylight/weather, and snapshots.
- **Presence time calculated from validated `PRESENCE_PING` events:** The rationale is strict presence accounting from validated events only.
- **Exponential low-pass personality drift:** The rationale is slow, monotonic movement toward expressive limits, calibrated for "measurable numerical drift" at 7 days and "user-perceivable drift" at 21 days.
- **No negative drift when absent:** The rationale is "If P = 0, Delta T = 0" and "No negative drift."
- **Mood state machine:** The rationale is a "fast-timescale state" that responds to time-of-day, interactions, ambient events, and personality modulators while persisting across sessions.
- **Call-Grammar Engine:** The rationale is species-specific procedural calls with bird seed, trait/mood-scaled tempo, and micro-variation for "procedural freshness" and to "avoid phase cancellation."

### Synchronization & Conflict Prevention

- **Canonical Single-Writer Model:** The rationale is multi-device consistency from a single server canonical record, with clients never computing or submitting absolute personality values.
- **Conflict-Free Append-Only Logging:** The rationale is eliminating "Last-Write-Wins (LWW) data loss" because phone and laptop sessions append events without overwriting vectors.

### Frontend Rendering Pipeline

- **Single Horizontal Viewport:** NOT RECOVERABLE FROM PLAN
- **Three Perch Zones:** The rationale is proximity and trait expression: Front Perch for bold/curious birds, Middle Perch as default, Back Perch for wary or drowsy birds.
- **Automated Perch Selection:** The rationale is that placement is driven by bird mood and boldness trait; the user cannot manually drag or place birds.
- **Loaded Mid-Motion:** NOT RECOVERABLE FROM PLAN
- **Loading Fallback:** The rationale is a "calm empty sky field" with "no loading spinners or app frame graphics."
- **Chrome Auto-Fade:** NOT RECOVERABLE FROM PLAN
- **Reduced-Motion Rendering:** The rationale is replacing frame-by-frame animation and interpolation with slow cross-fades while maintaining daylight color transitions and WebAudio quality.

### Audio Pipeline & WebAudio Runtime

- **Procedural Synthesis with no recorded audio:** The rationale is preserving bundle size and preventing "repetitive audible patterns."
- **Chorus Rebalance:** The rationale is dynamically mixing up to 7 birds into ambient space.
- **Listen-In Mix Ramp:** The rationale is raising the target bird smoothly through a 1.5-second logarithmic gain ramp while non-target birds decay to ambient background gain and are "never fully muted."
- **WebAudio Fallback Path:** The rationale is graceful silent mode when WebAudio is unsupported, blocked, or fails, with call captions automatically enabled and no recorded fallback attempted.

### Accessibility Surfaces

- **Screen-Reader Narration Engine:** The rationale is polite naturalist prose every 30-60 seconds or after user-triggered events, with "lowercase, present-tense, specific" tone and no UI announcement phrasing.
- **Real-Time Call Captioning:** The rationale is captions near the calling bird during vocalizations, procedurally generated from call grammar parameters.
- **Keyboard Navigation:** The rationale is full traversal through top bar controls, aviary scene, and birds, with Enter for listen-in and Esc for cancel or undo settle.
- **Contrast and focus rings:** The rationale is high-contrast focus rings for both light and dark palettes and text passing WCAG AA thresholds.

### Performance Budgets & Observability

- **Initial JS Bundle Size <= 2.0 MB gzipped:** The rationale is achieved through WebAudio synthesis instead of audio samples, compact SVG assets, and route code-splitting.
- **Time-to-First-Bird Visible < 500 ms:** NOT RECOVERABLE FROM PLAN
- **Consistent 60fps idle micro-motion:** The rationale is smooth rendering on a "5-year-old laptop."
- **Zero memory leak growth over 30-minute sessions:** The rationale is verification through CI automated browser heap tests.
- **Permitted Telemetry:** The rationale is aggregate operational metrics only, limited to latency histograms, tick processing duration, error rates, and WebAudio context error counts.
- **P99 Alarm Threshold:** The rationale is alerting when simulation tick p99 latency exceeds 5 seconds.
- **Strict Privacy Rule for telemetry:** The rationale is zero collection or aggregation of PII, email addresses, individual bird vector states, or per-user interaction logs in telemetry warehouses.

### Rollout & Aviary Growth Pacing

- **V1 Release Target:** NOT RECOVERABLE FROM PLAN
- **Bird Capacity Growth Pacing:** The rationale is unlocking birds strictly by "aviary age" and "never by visit count or engagement metrics," from Day 1 starter birds to Month 2 and Month 6+ additions up to 7.

### Risk Matrix & Mitigations

- **Drift Calibration Imbalance mitigation:** The rationale is avoiding drift that is too fast and "feels like Tamagotchi" or too slow and "feels static."
- **Audio Uncanniness / Synthetic Tone mitigation:** The rationale is avoiding harsh or robotic calls that "breaks immersion."
- **Multi-Device State Divergence mitigation:** The rationale is avoiding LWW race conditions that overwrite vector drift.
- **Accessibility Surface Regressions mitigation:** The rationale is preventing screen-reader narration from dropping into "announcement/UI register" by enforcing lowercase naturalist grammar.
