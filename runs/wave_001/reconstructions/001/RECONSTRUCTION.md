## System-level intent

1. Ambient observation and attention, not game pressure

The plan frames Pocket Aviary as "an ambient, browser-based experience focused on observation and attention rather than gamification or custodial chore loops." This shows up in the In Scope/Out of Scope split: "No Gamification," "No Tamagotchi Mechanics," "No Push Notifications," and "The aviary only exists when visited." The same intent appears in drift calibration risk: if drift is too quick, "the experience becomes a game"; if too slow, "it feels like a static screensaver."

2. Gentle positive progression without punishment

The plan repeatedly makes progression monotonic and non-punitive: "Monotonic positive drift based on presence-time, listen-in, and offers," "Personality values only increase or stay constant," and "Neglect causes quietness, not suffering." Absence can produce "ambient quietness" but "never wariness or visual degradation."

3. Quiet naturalist product voice

The plan carries a restrained observational voice through "Naturalist return-greeting," "Field Notebook" entries that are "generated sparingly," "lowercase, present-tense," and screen-reader prose in "the naturalist voice." Social is explicitly "Optional & Quiet," and the WebAudio fallback uses "clean, non-obtrusive matter-of-fact styling."

4. Server-authoritative canonical state with append-only interactions

The architecture insists on one canonical simulation: the server is the "sole writer of the canonical state," the database is the "single source of truth," and clients "never send requests like UPDATE birds." Interactions are buffered as an "append-only event log" so multiple devices contribute without overwriting each other.

5. Snapshot parameters on the server, sensory interpolation on the client

The render boundary is explicit: the server dictates "what state" the aviary is in, while the client calculates "how to render" it. This principle appears across Canvas rendering, client-side micro-motion, local WebAudio synthesis, and snapshot responses that provide state parameters rather than rendered behavior.

6. Quiet social and privacy-preserving account surfaces

The plan allows read-only visit invitations but rejects "visitor presence, chat, comments, or shared cursors." Account and social surfaces use encrypted emails, revocable sessions, one-time links, instant revocation, soft deletion, and observability that exports "No individual bird names, presence history, or account-identifying telemetry values."

7. Accessibility as a parallel representation of the same mechanics

Accessibility is treated as a complete surface, not a separate simplified mode: "full access to its mechanics through structured alternative representations," ARIA narration uses "the exact same state models that feed the visual canvas render paths," call captions replace blocked audio signals, and the plan requires "Complete keyboard navigation."

8. Low-power, responsive, lazily evaluated implementation

Efficiency appears in the simulation and frontend: ticks run "lazily" to "maximize efficiency and prevent background polling issues," the Canvas loop is "optimized for low-power operation and responsive viewports," and budgets cover bundle size, first paint, frame rate, and memory growth.

9. Procedural life and variation without heavy assets

The plan favors procedural systems over recorded or bulky assets: "procedural client-side audio synthesis," "no recorded loops," "parameterized math formulas," "SVG outlines drawn programmatically," and jitter to ensure "no two synthesized calls sound identical."

## Per-feature whys

### 1. Scope

- Aviary Context: single horizontal responsive canvas fitting on one screen without panning, zoom, or scrolling
  - Rationale: The plan ties this to an "ambient, browser-based experience" and a responsive canvas that is immediately observable rather than navigated as a task space.

- Aviary Context: three perch zones
  - Rationale: The plan says the zones determine "bird placement based on mood/personality," and later maps moods to zones such as wary birds staying in the back perch zone and content birds staying in the middle or front.

- Aviary Context: day/night lighting cycle synced to the user's browser local time
  - NOT RECOVERABLE FROM PLAN

- Aviary Context: subtle ambient micro-motion
  - Rationale: The plan includes leaf drift, feather drift, wind, parallax, head bobbing, breathing, feather-fluffing, and tail-whips to keep the aviary alive while staying ambient and lightweight.

- Bird Engine: stable bird identity via unique persistent IDs
  - Rationale: Persistent IDs support stable bird identity in the canonical state, event log, per-bird personality vectors, moods, offers, listen-in, and snapshots across sessions/devices.

- Bird Engine: starting population of exactly 2 birds
  - NOT RECOVERABLE FROM PLAN

- Bird Engine: progression cap of exactly 7 birds
  - Rationale: The rollout plan says maximum bird caps ramp from 2 to 7 "progressively based on user cohort feedback."

- Bird Engine: hidden slowly-drifting numerical personality vector
  - Rationale: The vector lets presence, listen-in, and offers create "measurable trait delta" and later "visible shift in perching behaviors, coloration, and vocalisation frequencies" without turning the experience into overt game stats.

- Bird Engine: monotonic positive drift
  - Rationale: The plan guarantees "Personality values only increase or stay constant," making attention positive while preserving the principle that neglect causes quietness rather than harm.

- Bird Engine: fast-timescale mood state
  - Rationale: Mood transitions supply immediate behavior changes from time of day, weather, recent events, and personality while the personality vector remains slow-timescale.

- Bird Engine: procedural client-side audio synthesis
  - Rationale: Procedural synthesis keeps calls unique and species-voiced without "recorded loops"; the plan mitigates "Robotic Procedural Audio" with jitter so no two calls sound identical.

- Interactions: naturalist return-greeting staggered offset
  - Rationale: The greeting varies by "boldness, mood, and absence duration," reinforcing bird personality and the naturalist voice rather than a generic notification.

- Interactions: listen-in
  - Rationale: Listen-in lets focusing a bird bring "its call up in the audio mix and pan-centers it while dampening others," giving attention a sensory effect and nudging social_warmth and vocal_frequency.

- Interactions: offers
  - Rationale: Offers are accepted events that nudge "curiosity and boldness," with cooldowns to keep them from becoming a rapid chore loop.

- Interactions: settle
  - Rationale: Settle creates a "soft close," shifts lighting to evening, and ends presence while remaining "reversable within 5s."

- Interactions: presence tracking
  - Rationale: Conjunct tracking of visible state, focus, and mouse/key activity defines "valid presence" so drift is based on actual attentive presence rather than background time.

- Interactions: Field Notebook
  - Rationale: The notebook provides "read-only naturalist observations" generated "sparingly" in lowercase present-tense, supporting quiet observation instead of active task tracking.

- System & Account Surfaces: single-user accounts via email magic link
  - Rationale: The plan specifies short-lived, single-use magic links and encrypted emails, suggesting low-friction access with account protection.

- System & Account Surfaces: revocable per-device session tokens
  - Rationale: Revocation supports account/session control across devices through session listing and deletion.

- System & Account Surfaces: JSON state snapshot export
  - NOT RECOVERABLE FROM PLAN

- System & Account Surfaces: soft-deletion for 30 days, hard-deletion thereafter
  - NOT RECOVERABLE FROM PLAN

- Social: read-only visit invitation
  - Rationale: The invitation is "Optional & Quiet" and read-only so social access can exist without "visitor presence, chat, comments, or shared cursors."

- Accessibility: ARIA live narration in naturalist prose
  - Rationale: ARIA narration gives screen-reader users descriptive access in the same "naturalist voice" as the rest of the product.

- Accessibility: reduced-motion rendering
  - Rationale: Reduced motion bypasses flight Bezier movement, cross-fades poses, and deactivates particles for users with `prefers-reduced-motion`.

- Accessibility: call captions
  - Rationale: Captions ensure users do not miss audio interaction signals, especially when WebAudio fails or is blocked.

- Accessibility: WebAudio fallback
  - Rationale: If AudioContext fails, the app suppresses synthesis and turns captions on by default so the interaction remains available without sound.

- Accessibility: WCAG AA contrast matching
  - NOT RECOVERABLE FROM PLAN

- Accessibility: complete keyboard navigation
  - Rationale: Keyboard focus and commands give full access to birds, buttons, settings, listen-in, options, and views.

- Out of Scope: web-only, no native app
  - NOT RECOVERABLE FROM PLAN

- Out of Scope: no gamification
  - Rationale: The plan rejects "streaks, achievements, green-dot calendars, badges, XP, levels, or rank lists" because the product is focused on observation and attention rather than gamification.

- Out of Scope: no Tamagotchi mechanics
  - Rationale: The plan rejects deaths, hunger meters, and custodial duty indicators because neglect causes "quietness, not suffering."

- Out of Scope: no social network features
  - Rationale: The plan rejects comments, mutual visits, public discovery lists, profiles, shared cursors, and leaderboards to keep social "Optional & Quiet."

- Out of Scope: no push notifications
  - Rationale: The plan says "The aviary only exists when visited" and sends no emails/push notices about neglect or bird states.

### 2. Architecture

- Frontend Single Page Application
  - Rationale: The SPA serves static assets from a CDN edge, runs "completely in client memory," and owns WebAudio and Canvas execution for local rendering and sound.

- Server API Service
  - Rationale: The service centralizes authentication, session management, event log writes, and snapshot reads.

- Simulation Tick Engine
  - Rationale: The tick engine updates canonical state by processing the event log and writing database changes.

- Server Core Responsibilities: storing and securing client records
  - Rationale: The plan calls for synthetic UUIDs and encrypted emails to secure account records.

- Server Core Responsibilities: sole writer of canonical state
  - Rationale: Keeping birds, personality vectors, and moods server-authored protects the canonical simulation and sync model.

- Server Core Responsibilities: Field Notebook entries
  - Rationale: Server-generated entries use simulation state, keeping notebook observations grounded in canonical bird behavior.

- Server Core Responsibilities: social token verification and host-visitor isolation
  - Rationale: Verification and isolation preserve read-only visits without visitor interaction or shared presence.

- Client Core Responsibilities: rendering visual frames from snapshots
  - Rationale: The client turns state snapshots into frames, leaving canonical state on the server and sensory rendering local.

- Client Core Responsibilities: client-side animations and cross-fades
  - Rationale: Local interpolation, particles, skeletal micro-movements, and cross-fades implement the "how to render" side of the boundary.

- Client Core Responsibilities: procedural audio locally
  - Rationale: The client generates sound from server parameters so calls remain procedural and locally responsive.

- Client Core Responsibilities: detecting inputs and focus changes
  - Rationale: Client detection provides presence status based on visibility, window focus, and mouse/key activity.

- Client Core Responsibilities: buffering and posting events
  - Rationale: Buffered user interaction events feed the server-side append-only event log.

- Render Pipeline Boundary
  - Rationale: The boundary separates server "what state" from client "how to render," allowing canonical state with fluid local animation and synthesis.

### 3. Data Model

- Accounts table
  - Rationale: The table stores synthetic account IDs, encrypted email, creation time, and deletion time for secured account identity and soft deletion.

- Sessions table
  - Rationale: Hashed tokens, user agents, expiration, and revocation support revocable per-device sessions.

- Birds table
  - Rationale: Stores each bird's persistent ID, species, hidden traits, current mood, perch zone, and tick time as canonical simulation state.

- Interaction / Presence Event Log table
  - Rationale: The append-only table records presence, offers, listen-in, and settle events so the server can process interaction trails sequentially.

- Field Notebook Entries table
  - Rationale: Stores generated prose observations tied to the account.

- Visit Invitations table
  - Rationale: Stores one-time invitation state, encrypted visitor email, hash lookup, status, expiration, and host link for read-only visits and revocation.

- Visit Log table
  - Rationale: Records visits and duration for invitation visits.

### 4. API Surface

- `POST /api/auth/magic-link`
  - Rationale: Encrypts email, checks or creates account, and sends a 15-minute verification link for single-user account access.

- `POST /api/auth/verify`
  - Rationale: Verifies the magic token and sets an httpOnly, Secure, SameSite=Strict session cookie.

- `GET /api/account/sessions`
  - Rationale: Lists sessions with current-session indication so users can inspect device sessions.

- `DELETE /api/account/sessions/:id`
  - Rationale: Revokes a session as part of per-device session control.

- `GET /api/aviary/snapshot`
  - Rationale: Provides current aviary and bird state parameters for client rendering from canonical server state.

- `POST /api/aviary/events`
  - Rationale: Accepts presence and interaction events into the event log for simulation processing.

- `GET /api/notebook`
  - Rationale: Returns stored naturalist prose observations.

- `POST /api/social/invite`
  - Rationale: Creates a read-only visit invitation with email and expiration.

- `POST /api/social/invite/:id/revoke`
  - Rationale: Provides instant revocation for visit invitations.

- `GET /api/social/visit/:invite_id`
  - Rationale: Returns a read-only aviary snapshot with interactions disabled, preserving host-visitor isolation.

### 5. Simulation Engine Design

- Server-side lazy tick architecture
  - Rationale: Lazy ticks "maximize efficiency and prevent background polling issues" while catching up state when a client requests snapshots or posts events.

- Fast-forward tick chain
  - Rationale: The chain catches up elapsed time from `last_tick_time`, applying deltas before returning a snapshot.

- Drift function
  - Rationale: Drift is "monotonic, slow, and incremental" so traits only increase or stay constant and attention produces gradual measurable change.

- Presence time drift
  - Rationale: Presence time is valid accumulated attention and "nudges all traits."

- Listen-in time drift
  - Rationale: Focused duration nudges `social_warmth` and `vocal_frequency` for the focused bird.

- Offers drift
  - Rationale: Accepted offers nudge `curiosity` and `boldness`.

- Drift calibration
  - Rationale: The calibration aims for 7 hours to be measurable in instruments and 21 hours to show visible shifts in perching, coloration, and vocalisation frequencies.

- Mood transition engine
  - Rationale: Fast-timescale moods react to time of day, weather, recent events, and personality, giving birds changing observable behavior.

- Personality bias in mood transitions
  - Rationale: Boldness and social warmth shape transitions, such as reducing wary probability or increasing curiosity when another bird calls.

- Mood: wary
  - Rationale: Wary birds stay in the back perch zone and quiet, expressing mood through placement and sound.

- Mood: content
  - Rationale: Content birds preen and stay in middle or front zones, expressing calm presence.

- Mood: curious
  - Rationale: Curious birds focus on cursor movements and head-tilting, making attention visible.

- Mood: drowsy
  - Rationale: Drowsiness triggers near dusk and expresses evening settling.

- Mood: alert
  - Rationale: Alertness triggers at morning or during wind/rain and produces high vocal frequency and looking around.

- Call-grammar runtime
  - Rationale: Server seed parameters and client assembly keep vocalizations "procedurally unique" while preserving distinct "species voice."

### 6. Sync Model

- Multi-device state synchronization
  - Rationale: The server database is the "single source of truth," allowing a phone and laptop to render the "exact same scene."

- Clients do not store or update state values locally
  - Rationale: Clients retrieve snapshots and push events so state remains canonical on the server.

- Conflict prevention: no last-write-wins
  - Rationale: The plan prohibits direct mutations and processes event trails sequentially so concurrent devices do not overwrite each other's state changes.

- Deterministic execution order
  - Rationale: Sequential event consumption ensures concurrent logins "merely contribute to the same append-only log."

### 7. Frontend Rendering Pipeline

- HTML5 Canvas drawing loop
  - Rationale: Canvas supports low-power rendering and responsive viewports.

- Logical coordinates frame of 1920 x 1080
  - NOT RECOVERABLE FROM PLAN

- Maintained 16:9 aspect ratio
  - Rationale: CSS constraints keep the single canvas contained within the viewport without panning, zoom, or scrolling.

- Three depth layers
  - Rationale: Background, middle, and foreground layers provide scene composition and depth ordering for sky, perches, birds, leaves, branches, and droplets.

- Idle micro-motion
  - Rationale: Client-side sine waves and noise maps create head bobbing, breathing, feather-fluffing, and tail-whips while keeping assets "extremely lightweight."

- Flight interpolation
  - Rationale: Quadratic Bezier movement smooths bird perch changes over 1.5 seconds.

- Reduced-motion override in rendering
  - Rationale: Bypasses Bezier motion, uses fade transitions, and disables particles for reduced-motion users.

### 8. Audio Pipeline

- WebAudio node pipeline
  - Rationale: Oscillator, filter, gain, panner, and destination nodes generate procedural sound on the client.

- Frequency modulation
  - Rationale: A fast LFO adds "natural vibrato" to create an organic bird-like whistle.

- Motif definition
  - Rationale: Frequency and gain sequences define call shapes.

- Parameter sweeps
  - Rationale: `linearRampToValueAtTime` connects nodes smoothly.

- Listen-in focused bird volume
  - Rationale: Ramping the focused bird from normal to focus volume makes attention audible.

- Listen-in other birds volume
  - Rationale: Ramping other birds down to ambient level lets the focused call come forward without muting the aviary.

- Listen-in panning
  - Rationale: Centering the focused bird and moving others left/right creates a "spacious background soundscape."

- WebAudio fallback
  - Rationale: When AudioContext fails or is blocked, audio is suppressed and captions are turned on so users do not miss interaction signals.

### 9. Accessibility Surfaces

- Screen-reader narration
  - Rationale: The `aria-live="polite"` container updates with naturalist prose so screen-reader users receive the scene state.

- 45-second narration cadence
  - Rationale: The cadence gives periodic descriptive paragraphs without constant interruption.

- Immediate live-region updates for successful offers and settle
  - Rationale: Interaction events are announced promptly through the same narration surface.

- Keyboard focus order
  - Rationale: Birds, top-bar buttons, and settings fields are reachable through focus order.

- `Tab` keyboard behavior
  - Rationale: Cycles focus through top bar and perch-zone birds.

- `Enter` keyboard behavior
  - Rationale: Engages listen-in on the focused bird.

- `Escape` keyboard behavior
  - Rationale: Exits listen-in and closes panels or notebook view.

- `Space` keyboard behavior
  - Rationale: Activates the selected option or action.

- High-contrast focus ring
  - Rationale: Provides visible focus around birds or buttons through CSS outlines with canvas fallback.

### 10. Performance Budgets and Observability

- JS bundle size limit
  - NOT RECOVERABLE FROM PLAN

- First paint target
  - Rationale: The plan targets fast mobile first paint and names inline critical CSS, initial state payload injection, and deferred settings/notebook imports as the strategy.

- Frame rate budget
  - Rationale: Stable 60 fps is required during active rendering on older hardware and major browsers.

- Memory growth target
  - Rationale: Zero leak over 30 minutes of idle operation is supported by reusing WebAudio synth nodes and clearing old canvas buffers.

- Observability metrics
  - Rationale: Aggregate metrics track load, first bird visibility, tick latency, and WebAudio errors for operational health.

- Anonymized telemetry
  - Rationale: No bird names, presence history, or account-identifying telemetry are exported, preserving privacy.

### 11. Rollout Plan

- Phase 1: Local Testing & Calibration
  - Rationale: Tests verify lazy catch-up ticks match sequential ticks and drift remains monotonic.

- Phase 2: Private Beta
  - Rationale: A small beta with magic-link verification lets the team observe aggregate performance logs, including p99 tick computation time.

- Phase 3: Public Release
  - Rationale: Production rollout ramps bird caps from 2 to 7 based on user cohort feedback.

### 12. Risks and Mitigations

- Drift Calibration Imbalance mitigation
  - Rationale: Centralized adjustable calibration and integration tests keep the experience between game-like overprogression and static screensaver.

- Multi-device Session Race Conditions mitigation
  - Rationale: Sequential append-only event logs and PostgreSQL `SERIALIZABLE` isolation mitigate concurrent active-tab races.

- Robotic Procedural Audio mitigation
  - Rationale: Frequency and envelope jitter make procedural calls less cold or mechanical.

- Accessibility Degradation mitigation
  - Rationale: ARIA narration uses the same state models as the canvas render paths to avoid drift between visual and screen-reader descriptions.
