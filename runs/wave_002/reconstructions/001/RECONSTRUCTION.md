## System-level intent

- **Slow-timescale relationship over gamified tasks**: The plan states that V1 "focuses strictly on establishing a slow-timescale relationship" and that the core value is "idle attention (presence) rather than gamified custodial tasks." This shows up again in the explicit non-goals: "No streaks, counters, levels, XP," and "No hunger, sickness, or bird death."

- **Presence should be honest and ambient**: Presence tracking is described as "Honest tracking combining tab visibility, window focus, and user activity." The drift formulas use accumulated `presenceSeconds`, but the product framing keeps this as "idle attention (presence)" rather than a score or reward loop.

- **Neglect should never punish or distress the birds**: The plan says slow personality drift is "Monotonic progression toward expressiveness" where "traits never decrease on neglect" and "neglect makes birds quiet and ambient." The same intent appears in the non-goal rejecting hunger, sickness, and bird death.

- **Expressiveness should emerge slowly, measurably, and visibly**: The drift calibration target distinguishes "Measurable changes" after 1 week from "Visible changes" after 3 weeks. The risk section says drift that is too fast makes users "treat it as a game to optimize," while drift that is too slow makes the site "feel like a static wallpaper."

- **A single canonical server state should protect continuity across devices**: Accounts and sync require "multi-device sync via a canonical server-side simulation." The architecture says the server "Holds canonical authority," and the sync model says "conflict loops like 'Last-Write-Wins' are conceptually impossible."

- **The client should render and synthesize, not decide bird behavior**: The "Render Pipeline Boundary" says the client "does not simulate bird behavior, drift, or mood changes." It accepts a snapshot and renders it smoothly using CSS transitions and Canvas interpolation.

- **The aviary voice is naturalist, sparse, prose-like, and matter-of-fact**: The Field Notebook is "naturalist-narrated, sparse, read-only observations." Screen-reader narration uses "narrative prose" on a "slow queue-managed cadence," and network re-authorization is described as "matter-of-fact."

- **Audio should feel procedural but organic, not like stored media or bleeps**: The plan bypasses recorded files and uses "client-side procedural synthesis" with species motifs. The risk section warns that oscillators can sound like "retro video game bleeps rather than organic bird vocalizations" and mitigates that with filters, envelopes, and forest impulse responses.

- **Social should remain read-only, ambient, and low-pressure**: Visits are "Read-only ambient visits," "invite-only by email," "revocable," with "no co-presence" and visitor presence that "doesn't affect host drift." The explicit non-goals reject public directories, leaderboards, feeds, visitor avatars, chat, and host push notifications.

- **Accessibility is part of the core loop, not an add-on**: The plan says accessibility targets are "integrated directly into the core user experience loop." It includes narration, reduced motion, call captions, keyboard navigation, visible focus, and WCAG AA contrast.

- **Performance and observability must preserve the quiet product shape**: Budgets specify `< 2MB`, `< 500ms`, `60fps`, and `0MB` memory growth. Observability tracks aggregate operational metrics while analytics "never expose account UUIDs or individual bird names."

- **Privacy and PII isolation should be structural**: Accounts use synthetic UUIDs and encrypted email. The data model calls out "strict PII isolation," auth returns `200 OK` to prevent email enumeration, and telemetry payloads carry only generic OS/browser labels and performance numbers.

## Per-feature whys

### 1. Scope

- **Bird Adoption & Identity**: The plan frames this as part of the V1 focus on a "small flock of virtual birds" and a "slow-timescale relationship" between user and birds.

- **Start with 2 starter birds and expand up to a strict cap of 7**: The articulated rationale is to keep V1 focused on a "small flock" rather than an open-ended collection.

- **Species selected automatically from a pool of 6**: NOT RECOVERABLE FROM PLAN

- **Stable, unique internal IDs and user-changeable names**: The rationale is identity for a relationship: birds remain stable entities while the user can name them.

- **Presence Tracking**: It supports the core value of "idle attention (presence)" and feeds the simulation through "presence_ping" events and accumulated presence seconds.

- **Slow personality drift**: The why is "Monotonic progression toward expressiveness" without regression on neglect, preserving a slow relationship instead of custodial punishment.

- **Fast-timescale moods**: The plan separates "Daily-ish" mood cycles from slow personality drift so local time, weather, and recent interactions can affect current mood without changing the monotonic personality vector.

- **Procedural Calls**: The plan uses synthesized client-side calls based on species-specific motifs, avoiding recorded files and supporting mood-modified call grammar.

- **Return-greeting**: The rationale present in the plan is that greetings are "staggered, procedurally varied," matching the broader intent for natural, non-mechanical bird behavior.

- **Listen-in**: The plan articulates this as "gradual volume cross-fade focusing a single bird," with the audio pipeline centering and raising the focused bird while other birds stay audible as ambient background.

- **Offers**: Offers provide sparse user interaction inputs to the drift model; the tick loop maps offers into boldness and curiosity changes.

- **Per-bird cooldowns for offers**: NOT RECOVERABLE FROM PLAN

- **Settle**: The rationale in the plan is a "user-initiated soft session end" with a "5s undo grace period"; accessibility handling treats Settle as an explicit high-priority user event.

- **Field Notebook**: The plan makes it "naturalist-narrated, sparse, read-only observations," preserving the product voice while recording observations without turning them into tasks.

- **Accounts & Sync**: The rationale is single-user identity, "PII isolation," and "multi-device sync via a canonical server-side simulation."

- **Magic-link email auth**: The plan uses a 15-minute link, invalidate-on-use behavior, secure cookies, and a `200 OK` response that is "Always" returned "to prevent email enumeration."

- **Synthetic account UUIDs**: The explicit rationale is "PII isolation."

- **Accessibility surfaces**: The rationale is that accessibility is "integrated directly into the core user experience loop," with narration, reduced motion, captions, keyboard access, and contrast.

- **Social visits**: The why is read-only ambient visiting without social-network pressure: invite-only, expiring, revocable links, no co-presence, no visitor effect on host drift.

- **Web only**: The plan names native applications as out of scope; it gives no additional rationale beyond the V1 web scope.

- **No gamification**: The rationale is to protect "idle attention (presence)" from streaks, counters, levels, XP, milestones, badges, and optimization behavior.

- **No custodial/Tamagotchi mechanics**: The rationale is that neglect should make birds "quiet and ambient rather than distressed," not hungry, sick, or dead.

- **No social network layers**: The rationale is to keep visits ambient and avoid public directories, leaderboards, feeds, avatars, chat, or host push notifications.

### 2. Architecture

- **Browser Client / CDN Edge / Backend split**: The architecture separates rendering and local synthesis from canonical simulation and storage, letting the client pull snapshots, submit events, and render without owning drift or mood changes.

- **Server Role**: The server "Holds canonical authority," houses auth, receives raw interaction events, executes the tick runner, and persists accounts, birds, and notebook states.

- **Client Role**: The client is a "Thin client" that monitors visibility/activity, renders snapshots, schedules and synthesizes calls, manages accessibility queues, and forwards events.

- **Render Pipeline Boundary**: The rationale is to prevent the client from simulating "bird behavior, drift, or mood changes" while still allowing smooth visual interpolation between snapshot coordinates.

- **Static Assets & State Snippets at the Edge**: The plan uses edge-published snapshots to let clients pull state snapshots and meet time-to-first-bird goals.

### 3. Data Model

- **PostgreSQL schema**: The plan says PostgreSQL is used for "transactional consistency and strict PII isolation."

- **Accounts table**: The rationale is account identity with encrypted email and soft deletion; the `deleted_at` comment names a "30-day recovery window."

- **Sessions table**: The rationale present is active user session tracking through token hashes, expiry, and revocation.

- **Birds table**: It persists species, names, hidden personality vectors, and fast-timescale state so the canonical server-side simulation can update bird state.

- **Hidden Personality Vector**: The plan uses boldness, social warmth, vocal frequency, plumage saturation, and curiosity as monotonic expressiveness dimensions.

- **Fast-Timescale State**: The plan uses `current_mood` and `last_mood_updated_at` to separate fast mood from slow drift.

- **Append-Only Interaction Events**: The rationale is to ingest presence, listen-in, and offer events for processing during the next simulation tick and to avoid sync conflicts through append-only order.

- **Field Notebook Entries**: The table supports sparse, naturalist-narrated observations generated by notebook evaluation.

- **Visits and visit logs**: The tables support invite-only ambient visits, revocation, expiry, and guest-session logging.

- **Encrypted visitor email**: The rationale is aligned with strict PII isolation.

- **Visitor IP hash**: The plan names a hash but gives no specific rationale beyond visit logging and privacy-shaped storage.

### 4. API Surface

- **API as ingestion gateway and distribution channel**: The plan says the API ingests interaction logs and distributes readonly snapshots.

- **`POST /api/auth/magic-link`**: The rationale includes rate limiting, secure short-lived token generation, invalidating existing tokens, and always returning `200 OK` to prevent email enumeration.

- **`POST /api/auth/verify`**: The why is to validate and immediately invalidate the token, then create a secure session token in an HTTP-only, secure, SameSite=Strict cookie.

- **`GET /api/aviary/snapshot`**: The rationale is to return the active aviary snapshot, with a restricted read-only subset for valid guest visits.

- **`POST /api/aviary/events`**: The rationale is to append incoming events to the database event log for execution during the next simulation tick.

- **`POST /api/visits/invite`**: The rationale is to create a 30-day visitor token and send it to the visitor for invite-only ambient visits.

- **`POST /api/visits/revoke`**: The rationale is revocability: it sets `revoked_at` and terminates active guest sessions immediately.

### 5. Simulation Engine Design

- **Server-side tick loop once per minute**: The plan uses this to process active accounts, consume unprocessed events, update bird state, evaluate notebook generation, and mark events processed.

- **Accumulated presence and interaction values**: The rationale is to convert presence pings, listen-in duration, and offers into simulation inputs.

- **Low-pass monotonic drift formulas**: The why is slow, clamped progression toward expressiveness, with values increasing toward `1.0` and never decreasing.

- **Drift calibration target**: The plan articulates measurable 1-week movement and visible 3-week movement, then ties calibration to the risk of becoming either "a game to optimize" or "a static wallpaper."

- **Backend evaluation tests for drift**: The rationale is to capture week-scale measurable change and assert target distribution curves before packaging releases.

- **Fast mood transition calculation**: The plan names the transition but does not articulate a specific rationale beyond maintaining fast-timescale state.

- **Periodic Field Notebook entry creation**: The plan names periodic evaluation but does not articulate a specific rationale beyond supporting notebook entries.

- **Call grammar motif definition**: The rationale is structured, species-specific procedural sound with notes, wave types, and mood modifiers.

- **Interval scheduling with exponential distribution**: The plan uses a random variable where vocal frequency shortens the mean gap, giving call timing a procedural cadence tied to personality.

- **Chorus behavior**: The why is a "natural staggered chorus" produced by response triggers based on social warmth and small randomized offsets.

### 6. Sync Model

- **Single Canonical State (Server-Authoritative)**: The rationale is that all drift calculations happen server-side from incoming logs, making Last-Write-Wins conflict loops "conceptually impossible."

- **Handshake on visibility or focus recovery**: The plan uses snapshot fetches so the client can align rendering parameters when a tab becomes visible or regains focus.

- **Local ingestion queue during connection failure**: The rationale is to preserve user events until reconnection, then flush them to the event endpoint.

- **Queue clearing on expired or revoked session**: The rationale is to avoid submitting events after auth is invalid and to present a "matter-of-fact re-authorization interface."

### 7. Frontend Rendering Pipeline

- **HTML5 Canvas with vector representation scaled to the page**: The plan uses this to render the aviary scene responsively.

- **Responsive width**: The rationale is to adapt boundaries and perch positions so "all birds remain visible."

- **Aspect-ratio constrained height**: The rationale is to prevent vertical scaling from "clipping the tree branches."

- **Three Perch Zones**: The rationale is visual depth through Y layers, scale, and saturation: back, middle, and front perch zones.

- **Quadratic bezier movement**: The plan explicitly says that when a bird changes perches, the client "does not teleport the canvas image."

- **Micro-motion**: The rationale is mood-mapped minor offsets in head tilt and tail feather flicking while perched.

- **Reduced-motion mode**: The rationale is accessibility: bypass particles, replace perch movement with cross-fades, and replace skeletal motion with static sprite pose cross-fades.

### 8. Audio Pipeline

- **Client-side procedural synthesis**: The plan bypasses recorded files entirely and synthesizes on the fly with WebAudio API nodes, also helping bundle budgets by avoiding sound files.

- **Natural exponential envelope decay**: The rationale is to shape oscillator notes into bird-call-like envelopes rather than raw tones.

- **Stereo Spatialization**: The rationale is to map each bird's static horizontal perch position to a matching panner position.

- **Listen-In Re-balance**: The rationale is a focus state: the focused bird is centered and louder, while other birds drop but remain "audible as ambient background."

- **WebAudio Fallback**: The rationale is graceful handling when `AudioContext` cannot resume: stop synthesis and auto-activate captions representing procedural sounds.

### 9. Accessibility Surfaces

- **Screen-Reader Narration**: The rationale is narrative prose in an `aria-live="polite"` region that describes the current bird array and environment at a slow cadence.

- **45-second narration cadence**: The risk section explains that frequent updates would clutter the screen-reader queue; standard narration changes are limited to 45-second intervals.

- **Event-driven narration overrides**: The rationale is that high-priority user events, such as Settle, should be announced immediately.

- **Call Captions**: The rationale is prose text corresponding to the active synthesized motif, placed adjacent to the bird and synchronized with audio envelopes.

- **Focus and Navigation**: The rationale is keyboard navigation through the header, canvas, and birds sorted left to right.

- **Dual-border visible focus outline**: The rationale is high contrast on both sunny and nighttime background frames.

- **Arrow / Enter / Escape keybinds**: The plan maps these to moving focus, triggering listen-in, and disengaging focus to return audio to ambient levels.

### 10. Performance Budgets and Observability

- **Initial Gzipped JS Bundle `< 2MB`**: The plan mitigates bundle size through code-splitting settings dialogs, excluding heavy asset packs, and synthesizing audio instead of storing sound files.

- **Time-to-First-Bird `< 500ms`**: The rationale is to embed the first state snapshot JSON directly in server-rendered HTML.

- **Animation Rate `60fps`**: The rationale is to use simple 2D canvas draws rather than complex DOM-node translations.

- **Memory Growth `0MB` over 30 minutes**: The rationale is to recycle AudioContext nodes and clear inactive canvas references on teardown.

- **Server telemetry**: The rationale is operational observability for simulation tick duration and PostgreSQL latency, with a DevOps alert if p99 tick execution exceeds 5 seconds.

- **Client telemetry**: The rationale is observing paint frame rate and WebAudio initialization failure rates.

- **PII isolation in analytics**: The rationale is that dashboards should never expose account UUIDs or bird names, only generic OS/browser labels and performance numbers.

### 11. Rollout

- **Controlled rollout phases**: The plan says the launch sequence is split into controlled phases "To ensure stability."

- **Phase 1: Architecture & Scaffolding**: The rationale is to set up databases, UUID generators, and magic-link email pathways before later simulation work.

- **Phase 2: Local Simulation & Synthesizer Prototyping**: The rationale is to build the baseline WebAudio species-timbre engine before accounts and beta rollout.

- **Standardize Wren and Pipit as first 2 starter species**: NOT RECOVERABLE FROM PLAN

- **Phase 3: Multi-Device Sync & Accessibility Verification**: The rationale is to deploy server-side simulation, execute screen-reader assertions, and verify LWW prevention on cross-tab usage.

- **Phase 4: Release & Scaling**: The rationale is production deployment while monitoring TTFB and bundle limits through edge-routed CDNs.

### 12. Risks and Mitigations

- **Drift Calibration Over-acceleration mitigation**: The why is to avoid making users optimize a game while also avoiding a static wallpaper.

- **Audio Timbre Uncanniness mitigation**: The why is to make procedural sounds feel like "organic bird vocalizations" rather than "retro video game bleeps."

- **Sync Drift on Multi-Device Session Collision mitigation**: The rationale is to process append-only event logs sequentially with database transactions to prevent race conditions.

- **Accessibility Screen-Reader Fatigue mitigation**: The rationale is to avoid cluttering the client's screen-reader queue while still allowing explicit actions to trigger immediate updates.
