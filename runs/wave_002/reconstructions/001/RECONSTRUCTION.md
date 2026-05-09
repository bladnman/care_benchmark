## System-level intent

1. **A quiet, ambient aviary rather than a game.** This shows up in Scope as a "Single-user, web-only virtual aviary," "Idle attention (Presence)," "Settle (soft session-end)," and in the explicit non-goals excluding "Gamification" and "Tamagotchi mechanics." It also appears in Frontend Rendering as "No spinner" and a "quiet field background," and in Rollout as protecting the "alive" feeling.

2. **A smart server with a dumb renderer.** Architecture states the boundary directly: "The client is a dumb renderer of a smart server." Sync Model repeats that "The server is the single source of truth," with "No Client-Side Resolution." The client "only appends to the event log" and has "No local state authority."

3. **Slow, gentle growth instead of punishment.** The Bird Engine includes "long-term personality drift," while the Simulation Engine uses "A slow low-pass filter" and makes drift "strictly monotonic toward expressive (never decreases)." Scope excludes "hunger, death, negative drift on neglect," and Rollout measures bird ramping in "weeks/months, not engagement."

4. **Naturalist product voice.** The plan repeatedly uses "naturalist" language: the Field Notebook contains "auto-generated naturalist observations," Notebook Entry text is "naturalist prose," and Screen-Reader Narration delivers "slow, naturalist prose updates." The Notebook Generator creates "a prose entry."

5. **Privacy-preserving, restrained sociality.** Scope is "Single-user" and the Social Visit feature is "opt-in" and "read-only." Non-goals exclude "profiles, public discovery, chat, co-presence, leaderboards, public aviaries." Observability is "Aggregate telemetry only" with "Strict Privacy" and "No per-bird state or per-user interaction history" logged to analytics.

6. **Accessible equivalents are part of the core surface.** Accessibility is in Scope and has dedicated surfaces: "Screen-reader narration," "reduced-motion mode," "call captioning," "full keyboard navigation," and "WCAG AA contrast." Audio fallback also keeps "silence with captions."

7. **The birds should feel alive immediately and continuously.** Frontend Rendering emphasizes interpolation, "Idle Micro-Motion," loading into motion with "No spinner," and motion "already in progress upon state fetch." Performance budgets include "Time-to-first-bird < 500ms," "60fps idle motion," and Rollout instrumentation protects the "alive" feeling.

8. **Procedural expression over static media.** Audio Pipeline says "Procedural Synthesis," "No static audio loops," runtime modulation by "pitch, timing," and "No recorded tracks." Simulation Engine keeps high-level call frequency and timing in the server snapshot so it dictates when a bird "should" call.

## Per-feature whys

### Scope

- **Single-user, web-only virtual aviary:** NOT RECOVERABLE FROM PLAN

- **Single horizontal scene without panning or scrolling:** NOT RECOVERABLE FROM PLAN

- **Two starter birds:** NOT RECOVERABLE FROM PLAN

- **Cap at seven birds:** The plan ties the cap to the chorus risk: procedural synthesis or the "chorus mechanic" could become "a cacophony with 7 birds."

- **Procedural calls:** The plan articulates that calls should be synthesized client-side with WebAudio, use "No static audio loops," and be modulated at runtime by species and personality.

- **Mood-shaped idle motion:** The plan says idle micro-motion runs client-side "based on the current mood enum," supporting continuous motion while keeping mood in the canonical snapshot.

- **Long-term personality drift:** The plan uses drift to let birds change through "presence" and interactions over time, with a "slow low-pass filter" so growth is slow and "strictly monotonic toward expressive."

- **Idle attention (Presence):** Presence is the primary input to personality drift: the Drift Function applies deltas "based primarily on accumulated `presence` time."

- **Return-greeting:** NOT RECOVERABLE FROM PLAN

- **Listen-in:** Listen-in gives focus without muting the aviary: the focused bird stays at "1.0," others ramp down to "ambient" and "never 0.0." It also secondarily increases `vocal_frequency`.

- **Offer seed:** Recent events including offers are evaluated for mood transitions, but the rationale for seed specifically is NOT RECOVERABLE FROM PLAN.

- **Offer song fragment:** Recent events including offers are evaluated for mood transitions, but the rationale for song fragment specifically is NOT RECOVERABLE FROM PLAN.

- **Offer still pool:** Recent events including offers are evaluated for mood transitions, but the rationale for still pool specifically is NOT RECOVERABLE FROM PLAN.

- **Settle:** The plan frames Settle as a "soft session-end" and exposes it as a gesture via `POST /api/aviary/settle`.

- **Server-side simulation tick:** The tick exists to advance canonical state, process the event log, update mood and personality drift, and generate Field Notebook entries at about one-minute intervals.

- **Single aviary per account:** NOT RECOVERABLE FROM PLAN

- **Multi-device sync by reading the same server state:** The plan's rationale is consistency through a canonical snapshot: phone and laptop both poll `GET /api/aviary/state`, then both pull updated mood after the next server tick.

- **Magic-link email sign-in:** NOT RECOVERABLE FROM PLAN

- **Read-only Field Notebook:** The plan says the notebook provides "auto-generated naturalist observations" and is generated from simulation conditions into "naturalist prose."

- **Social Visit feature:** The plan makes social access "opt-in" and "read-only" via email link, matching the non-goal of avoiding social network surfaces such as chat, co-presence, and public aviaries.

- **Screen-reader narration:** The rationale is to describe the scene through "slow, naturalist prose updates," prioritized by user interactions.

- **Reduced-motion mode:** The rationale is to replace "frame animations" and movement paths with "cross-fades" or "slow cross-fades between static poses."

- **Call captioning:** The plan uses captions to make procedural calls available as text overlays and as the fallback when WebAudio is denied or fails.

- **Full keyboard navigation:** The plan says keyboard support must cover "the top bar and birds" with high-contrast focus rings.

- **WCAG AA contrast:** The plan requires WCAG AA as the minimum for "all UI copy."

### Explicit Non-Goals

- **Native mobile apps:** NOT RECOVERABLE FROM PLAN

- **Gamification:** NOT RECOVERABLE FROM PLAN

- **Tamagotchi mechanics:** The plan excludes "hunger, death, negative drift on neglect" in order to keep drift from decreasing; the Simulation Engine says drift is "never decreases."

- **Social network surfaces:** The plan excludes profiles, discovery, chat, co-presence, leaderboards, and public aviaries while keeping Social Visit "opt-in" and "read-only."

- **Multi-aviary accounts:** NOT RECOVERABLE FROM PLAN

- **Customizable scenes:** NOT RECOVERABLE FROM PLAN

- **Payments:** NOT RECOVERABLE FROM PLAN

- **Real-time client-to-client sync:** The plan avoids this because the server is the "single source of truth" and clients "do not sync with each other."

- **Client-authored personality state:** The plan excludes this because the client "never mutates personality or mood directly" and only appends to the event log.

### Architecture

- **Browser client as rendering and audio synthesis engine:** The client pulls snapshots, interpolates motion, synthesizes WebAudio, and submits interaction events while retaining "No local state authority."

- **Server as authoritative state machine:** The server owns canonical state for mood, personality drift, and Field Notebook generation.

- **Auth Service:** NOT RECOVERABLE FROM PLAN

- **Simulation Engine Tick Worker:** The worker periodically processes the append-only event log for each active aviary to update mood, personality drift, and Field Notebook entries.

- **API Server:** The API server serves state snapshots and receives interaction events so clients can render state while only pushing events.

- **Database:** The plan uses the database to store accounts, UUID mappings, aviary state, bird states, and Field Notebook entries; no deeper rationale is articulated.

- **Append-only event log boundary:** The boundary exists so the client appends interaction events but never directly mutates personality or mood.

### Data Model

- **Account with synthetic UUID and encrypted email:** The plan stores identity through `account_id`, encrypted email, and session tokens; the specific rationale for synthetic UUID mappings is NOT RECOVERABLE FROM PLAN.

- **Aviary state:** The aviary stores `weather_state` and `time_of_day_offset` so weather and time-of-day can participate in mood transitions and snapshots.

- **Bird stable identity:** The bird stores a stable `bird_id`, species, name, and adoption time; the rationale for stable identity is NOT RECOVERABLE FROM PLAN.

- **Hidden Personality Vector:** Hidden personality scalars drive long-term drift and expression through boldness, social warmth, vocal frequency, plumage saturation, and curiosity.

- **Mood State:** Mood persists across sessions and is used for mood-shaped idle motion, call timing, and transitions based on recent events, time-of-day, weather, and base personality.

- **Event Log:** The append-only event log is the server's input for ticks, drift, mood transitions, and multi-device consistency.

- **Notebook Entry:** Notebook entries store timestamped "naturalist prose" generated by the tick.

- **Social Invite:** Social invites track email-linked read-only visits and revocation/expiration.

### API Surface

- **`POST /auth/magic-link`:** NOT RECOVERABLE FROM PLAN

- **`POST /auth/verify`:** NOT RECOVERABLE FROM PLAN

- **`GET /api/aviary/state`:** The endpoint returns the canonical snapshot for load, visibility changes, and polling so clients read the same server state.

- **`POST /api/aviary/events`:** The endpoint batches interaction events so the server can process presence, offers, and listen-in on the tick.

- **`GET /api/aviary/notebook`:** The endpoint exposes read-only paginated Field Notebook entries.

- **`POST /api/aviary/settle`:** The endpoint triggers the settle gesture; the plan's stated rationale is "soft session-end."

- **`POST /api/social/invite`:** The endpoint generates a visit link for an email to support opt-in Social Visit.

- **`DELETE /api/social/invite/:id`:** The endpoint revokes an invite so Social Visit remains controllable.

- **`GET /api/visit/:invite_id`:** The endpoint lets a visitor pull read-only state.

### Simulation Engine Design

- **Server-side tick:** The tick reads the event log since the last tick and periodically updates active aviaries.

- **Drift Function:** The drift function keeps personality change slow, presence-led, and "monotonic toward expressive."

- **Mood Transitions:** Mood transitions use recent events, time-of-day, ambient weather, and base personality to create short-term mood that "persists across sessions."

- **Notebook Generator:** The generator watches for conditions such as "first time bird X greeted before bird Y" and sporadically creates prose entries.

- **Call-Grammar Runtime:** The server determines high-level call frequency and timing so the snapshot dictates when a bird "should" call, even though audio is synthesized client-side.

### Sync Model

- **Canonical Server State:** The rationale is to make the server the "single source of truth."

- **No Client-Side Resolution:** The plan avoids client conflict resolution because clients only push events and never sync with each other.

- **Multi-Device:** Multi-device behavior is handled by polling the same canonical state so an event from one device updates both devices after the next tick and snapshot request.

### Frontend Rendering Pipeline

- **Minimal HTML/CSS top bar:** NOT RECOVERABLE FROM PLAN

- **Canvas or WebGL scene rendering:** The plan uses Canvas or WebGL to handle smooth parallax and sprite animations within the "2MB bundle budget."

- **Interpolation:** Interpolation creates continuous movement from server-provided absolute positions and states.

- **Idle Micro-Motion:** Idle micro-motion keeps the scene alive continuously while reflecting the current mood enum.

- **Reduced-Motion Mode:** Reduced motion swaps frame-by-frame animations and movement paths for slow cross-fades between static poses.

- **Loading with no spinner:** The plan says loading should start from a "quiet field background" with motion already in progress, avoiding load states that break the "entry illusion."

### Audio Pipeline

- **Procedural Synthesis:** Procedural synthesis uses WebAudio and avoids "static audio loops."

- **Motif Library:** The motif library keeps base samples/oscillators small and modulates pitch and timing by bird species and personality.

- **Listen-In Mix Decay:** Mix decay focuses one bird while leaving the others audible at ambient level, "never 0.0."

- **WebAudio Fallback:** If audio fails, the experience falls back to silence with captions and uses "No recorded tracks."

### Accessibility Surfaces

- **Screen-Reader Narration:** A visually hidden ARIA live region provides "slow, naturalist prose updates" every 30-60 seconds and prioritizes user interactions.

- **Captions:** Captions are opt-in overlays near calling birds and dynamically match the procedural call.

- **Focus & Keyboard:** Focus and keyboard support allow tab navigation for the top bar and birds with high-contrast focus rings.

- **Contrast:** WCAG AA contrast applies to all UI copy.

### Performance Budgets and Observability

- **Initial JS bundle under 2MB:** The budget is tied to avoiding performance and bundle-size failures that break the entry illusion.

- **Time-to-first-bird under 500ms:** The budget protects immediate arrival into the aviary and the "alive" feeling.

- **60fps idle motion:** The budget protects continuous idle motion on a "5-year-old laptop."

- **Zero memory leaks over 30 minutes:** The plan gives the target but no specific rationale beyond session stability; deeper rationale is NOT RECOVERABLE FROM PLAN.

- **Aggregate telemetry only:** Telemetry tracks load, render-frame timings, tick latency, and audio-context errors while preserving "Strict Privacy."

- **Simulation tick latency alarm at p99 over 5s:** The alarm exists to monitor tick latency so the engine scales without degrading the "alive" feeling.

- **No per-bird state or per-user interaction history in analytics:** The rationale is "Strict Privacy."

### Rollout

- **Web-only launch:** NOT RECOVERABLE FROM PLAN

- **Two birds per new account at launch:** NOT RECOVERABLE FROM PLAN

- **Ramping birds as an aviary ages:** Ramping is based on "weeks/months, not engagement" and unlocks up to five additional birds without using engagement mechanics.

- **Instrumentation from Day One:** Instrumentation is meant to ensure the engine scales without degrading the "alive" feeling.

### Risks

- **Drift Calibration:** The risk rationale is that loose presence tracking or aggressive filtering could make birds change too fast and break the "illusion of slow growth."

- **Sync Correctness:** The risk rationale is that dropped events or out-of-order processing could stall personality drift.

- **Audio Uncanniness:** The risk rationale is that procedural synthesis might not sound organic or could become a "cacophony with 7 birds."

- **Performance / Bundle Size:** The risk rationale is that exceeding the 2MB budget could cause load states that break the "entry illusion."
