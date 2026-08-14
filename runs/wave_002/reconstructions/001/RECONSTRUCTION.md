## System-level intent

- Quiet observational companionship over mechanics: The plan opens by calling Pocket Aviary "a quiet, browser-based virtual aviary" centered on "long-term, low-key observational companionship rather than game mechanics, task completion, or custodial burden." This intent recurs in the v1 scope forbidding streaks, levels, XP, badges, scores, health bars, mortality, neglect penalties, and other game or custody mechanics.

- "Feels alive, not robotic": The plan names this as a strict design principle and ties it to "mid-action loading," "continuous idle micro-motion," WebAudio calls that "never repeat identically," and "a server-side simulation that advances independently of whether a client is connected." It also shows up in first-frame loading, idle animation, audio randomization, and the autorun simulation tick.

- "Notice, never announce": Welcoming is supposed to occur "solely through procedurally varied bird behavior" such as head-tilts, glances, calls, and approaching perches. The same principle forbids greetings, toasts, banners, confetti, and counters, and later reappears in screen-reader narration that avoids "system-style state announcements."

- "Charm comes from specificity": The plan wants "naturalist, lowercase, present-tense observations" rather than generic milestones. This appears in the Field Notebook templates, screen-reader narration prose, and procedural call captions such as "a soft three-note rise."

- "Restraint over richness": The plan repeatedly prefers a "single fixed horizontal viewport," "no scrolling, panning, or zooming," "a calm, nature-derived palette," and "an aviary scene free of in-world UI chrome." The frontend rules reinforce this with "zero in-scene chrome" and a top bar that fades nearly away.

- Split product and system voice: The plan distinguishes "Naturalist voice for product, matter-of-fact for system." Product surfaces use lowercase present-tense naturalist prose; account management, authentication, sync errors, settings, and unsupported browser states use "direct, standard-capitalized, matter-of-fact technical prose."

- Authoritative server state over client authority: The architecture is "server-driven," with clients as "deterministic renderers and event-stream contributors." The multi-device model says clients never send absolute property values, avoiding state divergence and "Last-Write-Wins hazards."

- Slow additive growth without punishment: The plan uses "monotonic drift toward expressiveness," a "strictly monotonic" low-pass filter, and "traits never decay." Neglect produces no penalty; it simply results in "Delta = 0."

- Privacy through isolation and minimization: The plan uses synthetic UUIDs, encrypted emails, "Zero PII exposure," "Zero PII in Logs," "Zero Behavioral Tracking," and "STRICT ISOLATION" between production data and analytics or ML pipelines.

- Accessibility as a charm-preserving product surface: Accessibility is "designed as a first-class, charm-preserving surface rather than a compliance checklist." The same voice rules govern screen-reader narration, call captions, reduced motion, keyboard navigation, and contrast.

## Per-feature whys

### 1. Executive Summary & Scope

- Initial pair of birds in a single aviary: The plan presents this as the starting form of "long-term, low-key observational companionship."

- Maximum of seven birds unlocked strictly by aviary age: The rollout section says this preserves "per-bird vocal recognizability" and the product's "emotional relationship"; strict age milestones also keep expansion away from streaks, payment, or interaction counts.

- Single horizontal scene: This implements "restraint over richness" and later ensures "the entire aviary is visible without scrolling, panning, or zooming."

- No game mechanics, task completion, or custodial burden: The product purpose explicitly centers observation instead of those modes.

- Mid-action loading and first-frame motion: This supports "feels alive, not robotic"; the loading section says spinners "destroy the illusion of a self-sustaining habitat."

- Procedurally varied behavioral greetings with no welcome text: This is the "Notice, never announce" principle: welcoming happens through bird behavior, not toasts, banners, confetti, or counters.

- Naturalist, lowercase, present-tense product prose: This is how "charm comes from specificity," with examples like "pip greeted before wren today, first time this week."

- Matter-of-fact technical prose for account, auth, sync, settings, and unsupported states: This preserves the split between product charm and system clarity.

- Modern web browsers, last two major versions: NOT RECOVERABLE FROM PLAN

- Starter birds assigned from a 6-species pool: NOT RECOVERABLE FROM PLAN

- Monotonic drift toward expressiveness and presence accumulation: The plan wants long-term change that is measurable and visible without becoming a game or a "numbers are manipulated" system.

- Mood state transitions: The plan uses fast-timescale mood states to create time-of-day, interaction, weather, and bird-to-bird variation while persisting mood between sessions.

- Seed, song, pool offers, and settle goodbye gestures: NOT RECOVERABLE FROM PLAN

- 1:1 email-based visit invitations and read-only ambient viewing: The social surface is deliberately narrow, excluding public directories, feeds, profiles, follows, chat, visitor avatars, comments, leaderboards, and co-presence.

- Instant invitation revocation: The API says revocation "instantly blocks active snapshot pulls using that token."

- Silent visit log: NOT RECOVERABLE FROM PLAN

- Passwordless email magic link authentication: NOT RECOVERABLE FROM PLAN

- Revocable per-device sessions: NOT RECOVERABLE FROM PLAN

- JSON export: NOT RECOVERABLE FROM PLAN

- 30-day soft delete: NOT RECOVERABLE FROM PLAN

- Real-time procedural WebAudio, silent fallback, and captions: The audio plan avoids recorded files to satisfy the "<2MB bundle budget" and "prevent audio fatigue," while captions preserve access when audio fails.

- Dynamic chorus and gradual listen-in focus: The chorus system avoids "unnatural acoustic phase cancellation" and clipping; listen-in raises one channel while other birds "remain distinctly audible" and are "never hard-muted."

- Naturalist screen-reader narration, reduced-motion cross-fades, call captions, keyboard navigation, and WCAG AA contrast: The plan treats accessibility as "first-class" and "charm-preserving," not a static or noisy fallback.

### 2. System Architecture & Service Boundaries

- Client-server architecture with authoritative, server-driven simulation: The plan says this eliminates "client-side state divergence across multiple devices."

- Zero-dependency Vanilla JS core under 2MB gzipped: The frontend service topology calls this out for "ultra-lean bundle size."

- High-performance Canvas 2D / WebGL rendering layer: This supports the performance budget for continuous 60 fps rendering.

- Live accessibility manager: It exists to manage "aria-live narration, call caption overlays, focus manager."

- Auth service translating session tokens into synthetic account IDs with encrypted email: This prevents email addresses from propagating into worker logs, metrics, or telemetry.

- Simulation worker fleet with a 60-second tick: This is the mechanism by which the simulation advances independently and remains the "sole authoritative writer of bird state."

- PostgreSQL 16 primary store: NOT RECOVERABLE FROM PLAN

- Redis state snapshot cache and broker: The plan connects Redis to "sub-millisecond reads" and later to sub-50ms snapshot retrieval for first-bird visibility.

- Aggregate Datadog telemetry with zero per-bird or per-account data: This draws the plan's "Observability vs. Privacy Enforcement Boundary."

### 3. Data Model & Database Schemas

- Synthetic UUIDv4 internal references and no email outside `accounts`: The plan says no table outside `accounts` stores or references user email addresses.

- Encrypted email plus HMAC email hash: The schema comment says the hash allows "lookup without decryption."

- Hidden normalized personality traits: The bird traits are "strictly hidden from UI," consistent with avoiding gamified number manipulation.

- Append-only interaction event log: The plan calls it the "Authoritative Input to Simulation Tick" and later uses it to avoid Last-Write-Wins overwrites.

- Field Notebook observation entries: These store the sparse naturalist observations that carry the "charm comes from specificity" voice.

- Visit invitations with encrypted visitor email, token hash, status, and expiry: This supports 1:1 email visit invitations with revocation and avoids plain visitor email storage.

- Host-visible visit logs with masked visitor email and duration: NOT RECOVERABLE FROM PLAN

### 4. API Surface & Protocols

- Matter-of-fact error responses and system endpoints: This implements the system side of "Naturalist voice for product, matter-of-fact for system."

- Magic-link rate limit, 15-minute token, and single-use sign-in: NOT RECOVERABLE FROM PLAN

- `HttpOnly; Secure; SameSite=Strict` session cookie on verify: NOT RECOVERABLE FROM PLAN

- Account export sent as a short-lived download link to verified email: NOT RECOVERABLE FROM PLAN

- Account deletion scheduled for 30 days with cancellation by sign-in: NOT RECOVERABLE FROM PLAN

- Full aviary state snapshot on initial load, tab visibility restoration, or network recovery: The purpose is explicitly "Full snapshot retrieval" for those recovery points.

- Batch append-only event ingestion rejecting read-only visitors: This keeps visitors read-only and preserves host-only event authority.

- Visit invitation create, revoke, and read-only state endpoints: These implement 1:1 ambient viewing and make revoked visits return "This visit is no longer available."

### 5. Simulation Engine & Mathematical Models

- Presence accounting only when visible, focused, and recently active: The plan says this "prevents background tabs or forgotten overnight sessions from falsely saturating drift."

- Strictly monotonic low-pass personality drift: This creates "monotonic drift toward expressiveness" while ensuring neglect gives "Delta = 0" and "traits never decay."

- Trait weight parameters for boldness, social warmth, vocal frequency, plumage saturation, and curiosity: NOT RECOVERABLE FROM PLAN

- Drift calibration targets for one week and three weeks of visits: The plan wants one week to be "measurable in instrument tests" and three weeks "clearly noticeable" without saturation or decay.

- Fast-timescale mood state machine with persisted session continuity: Mood gives solar, interaction, weather, and alarm-call variation, and on tab open the bird shows the "exact stored mood" updated by interim ticks.

- Sparse Field Notebook cadence: The plan triggers entries every 2 to 4 active days or at "significant inflection points," matching the specificity principle without constant announcements.

- Deterministic naturalist template grammar: This generates lowercase observations such as greeter shifts, calm idle notes, and weather events.

### 6. Multi-Device Sync & Conflict Model

- Zero client authority on state: Clients never send absolute values; they only submit validated interaction events.

- Elimination of Last-Write-Wins hazards: Because the server applies additive deltas, concurrent sessions "cannot overwrite or erase accumulated drift history."

- Client interpolation and flight glide: This prevents teleportation; relocated birds glide naturally rather than snapping.

- Disconnection and re-sync on visibility restore or sleep resume: The client fetches current state and "smoothly merges" rendering with the updated server state.

### 7. Frontend Rendering Pipeline

- Single horizontal viewport: The whole aviary remains visible with no scrolling, panning, or zooming.

- Responsive 16:9 aspect scaling with dynamic padding: The plan says this ensures "no bird is ever cropped out of frame" from mobile to 4K.

- Zero in-scene chrome: No health bars, name tags, hover tooltips, or interaction icons appear in the aviary canvas, preserving restraint and habitat illusion.

- Bootstrap quiet field instead of spinners or blank screens: The reason is explicit: "Spinners destroy the illusion of a self-sustaining habitat."

- Mid-motion initialization with randomized phase offsets: Birds appear "mid-preen or mid-scan on the very first frame painted."

- Procedural idle micro-motion engine: Preening, scanning, weight shuffle, and breathing provide the "continuous idle micro-motion" required for feeling alive.

- Designed reduced-motion mode: It replaces skeletal animation with cross-fades, disables particles, and slows lighting transitions instead of using an "un-designed reduced-motion disabling."

- Top-bar chrome fade after inactivity: The top bar becomes nearly invisible after inactivity while restoring instantly on movement, keeping the scene free of persistent UI chrome.

### 8. WebAudio Procedural Synthesis Pipeline

- Zero recorded audio files and real-time WebAudio synthesis: This satisfies the "<2MB bundle budget" and prevents "audio fatigue."

- Species motif grammar with runtime parameter modulation: Randomized motifs based on vocal frequency and mood help calls avoid repeating identically and express bird state.

- Dynamic chorus staggering and compression: Staggered triggers avoid "unnatural acoustic phase cancellation"; compression prevents clipping and harmonizes chorus output.

- Listen-in mix curves: The focus bird rises gradually, ambient birds decay gradually, and other birds are "never hard-muted."

- WebAudio fallback to silence with call captions: This degrades gracefully when audio cannot initialize while respecting the strict non-goal of no looped audio fallback.

### 9. Accessibility Surfaces

- Screen-reader narration through `aria-live`: Naturalist prose at a 30-60s cadence makes accessibility part of the same charm-preserving voice rather than a system dump.

- Interaction-priority narration: User actions preempt the queue with observation-style prose while "strictly avoiding system-style state announcements."

- Real-time procedural call captions: Captions are high-contrast and unobtrusive, derived from the procedural motif, and synchronized with the audio envelope.

- Keyboard navigation and focus flow: Tab, arrow keys, Enter / Space, and Escape provide full keyboard navigation across top controls and birds.

- Double-ring high-contrast focus indicators: The plan says they remain visible across bright morning and dim evening sky palettes.

- 4.8:1 contrast minimum: This surpasses the WCAG AA 4.5:1 requirement.

### 10. Performance Budgets, Telemetry & Privacy

- Initial JS bundle under 2MB gzipped: The implementation strategy is zero large frameworks, Vanilla JS, SVGs, procedural WebAudio, and dynamic modal imports.

- Time to first bird visible under 500ms on 4G mobile: The strategy is an edge-cached shell, pre-rendered canvas background, and sub-50ms Redis snapshot retrieval.

- Continuous 60 fps runtime: The plan uses lightweight canvas draw routines, no DOM recreation in the main loop, and particle object pooling.

- Zero heap growth over 30 minutes: The plan uses reusable WebAudio node pools, pre-allocated particle arrays, and recycled notebook list elements.

- Simulation tick p99 under 5 seconds: The plan uses batched PostgreSQL queries, indexed event logs, and an asynchronous worker processing pool.

- Strict isolation of authoritative database contents from analytics, BI, and ML: This enforces privacy for birds, personality vectors, moods, notebook entries, and presence pings.

- Aggregate telemetry boundary: The plan permits request rates, status codes, latencies, tick runtimes, FPS percentiles, and WebAudio errors while forbidding user IDs, synthetic IDs, bird names, traits, moods, interaction frequencies, and per-user session durations.

- Zero PII in logs: Synthetic UUIDs are used between services, and emails are encrypted and prohibited from log lines and partition keys.

- Zero behavioral tracking: The plan explicitly forbids telemetry that tracks individual bird preferences or user habits.

### 11. Rollout & Population Ramp Plan

- Aviary age population milestones: The plan says age-only expansion preserves "per-bird vocal recognizability" and the product's "emotional relationship."

- Engineering execution phases from core engine through hardening: NOT RECOVERABLE FROM PLAN

### 12. Engineering Risks & Mitigations

- Drift simulation tests over 1,000 synthetic days: The mitigation keeps drift from becoming a game where "numbers are manipulated" or a "screensaver," while validating measurable and visible changes without saturation or decay.

- Dual-oscillator FM synthesis, formant filtering, and micro-randomized vibrato: This mitigates WebAudio sounding "sterile, metallic, or robotic" and reduces repetition fatigue.

- Overlap grouping and 60-second cap for multi-device presence: This prevents simultaneous tabs or devices from "artificially accelerating drift."

- Fixed pre-allocated WebAudio node pool: This mitigates garbage collection churn and memory bloat during extended sessions.
