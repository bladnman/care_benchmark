## System-level intent

1. Observational relationship-building instead of game progression. This shows up in the scope phrase "single-user virtual aviary focused on observational relationship-building" and in the non-goals: "No streaks, levels, achievements, or scores" and "No death, hunger, or distress." The intended relationship changes through presence and expressiveness, not through punishment or scores.

2. Procedural aliveness, calibrated slowly enough to feel alive rather than game-like. This appears in "Procedural bird behavior," "mood cycles," "procedural call grammar," the drift target of "measurable numerical change in ~1 week" and "visible user-perceived change in ~3 weeks," and the risk that drift "too fast" feels "like a game" while "too slow" feels "like a screensaver."

3. Server-side canonical state and simulation authority. The plan repeats this in "the server is the sole authority for state and simulation," "The tick is the only process allowed to mutate the aviary's canonical state," and "Single Source of Truth." The architecture is meant to avoid client-owned state and preserve a canonical aviary.

4. Signals instead of direct mutation. The sync model says clients "only send signals" such as `presence_event`, never absolute values such as `set_boldness(0.6)`. This shows the plan's intent to let the simulation engine compute deltas and prevent "last-write-wins" conflicts.

5. Naturalist voice as both experience and access surface. The same vocabulary appears in "Naturalist field notebook," "naturalist observation entry," "naturalist prose updates," and "lowercase, present-tense, naturalist observations." The plan carries the product voice into generated notebook entries, screen-reader narration, and call captions.

6. Ambient, opt-in, non-interactive social presence. The social surface is "Read-only ambient visits via email invitation," with "No profiles, follows, public discovery, or chat" and "Visitors cannot interact or be seen by the host." Social presence is intended to remain opt-in, private, and non-co-present.

7. Accessibility, performance, and privacy are designed boundaries. Accessibility is not a fallback: the risk section explicitly warns against "Treating accessibility as a 'fallback' rather than a designed surface." Performance is bounded by "<2MB initial bundle," "<500ms time-to-first-bird," and "60fps idle motion." Observability has a "Privacy Boundary" with "No per-user or per-bird data."

## Per-feature whys

### 1. Scope

- **Core Loop**: To make procedural bird behavior respond to "presence and specific interactions" such as "listen-in, offer, settle," supporting observational relationship-building.
- **Bird Engine**: To provide "Personality drift," "mood cycles," and "procedural call grammar" so birds change expressiveness, mood, and calls over time.
- **Naturalist field notebook**: To hold "generated observations" and "occasionally generate a new naturalist observation entry based on notable recent events."
- **Return-greeting**: To give "immediate priority" to user-initiated events in screen-reader narration, including "return-greeting, offer, settle."
- **Single horizontal scene**: NOT RECOVERABLE FROM PLAN
- **Three perch zones**: To organize the aviary into "three distinct Z-depth layers" of "front, middle, back."
- **Magic-link authentication**: NOT RECOVERABLE FROM PLAN
- **Single canonical aviary per account**: To keep one "canonical aviary state" owned by the account and served by the server-side source of truth.
- **Multi-device synchronization via server-side simulation**: To synchronize clients through an authoritative server simulation rather than client-owned state.
- **Read-only ambient visits via email invitation**: To allow opt-in social visiting while keeping visitors "read-only" and "non-interactive."
- **Naturalist prose narration for screen readers**: To make screen-reader updates part of the same "naturalist prose" product voice.
- **Reduced-motion mode**: To replace frame-by-frame animations with "slow, aesthetic cross-fades" while removing ambient drift and maintaining lighting shifts.
- **Call captioning**: To describe calls as "short, mood-driven prose descriptions" and to remain enabled during "graceful silence" if WebAudio fails.
- **Performance targets**: To keep the initial bundle, first bird, idle motion, runtime, and memory inside explicit budgets.
- **No streaks, levels, achievements, or scores**: To keep the aviary out of gamification and aligned with observational relationship-building.
- **No death, hunger, or distress**: To avoid Tamagotchi mechanics; "neglect only leads to reduced expressiveness."
- **No profiles, follows, public discovery, or chat**: To prevent social network surfaces and keep social features limited to opt-in visits.
- **Web-only for v1**: NOT RECOVERABLE FROM PLAN
- **Visitors cannot interact or be seen by the host**: To avoid co-presence while preserving read-only ambient visits.

### 2. Architecture

- **Client rendering, audio, and signal capture**: To keep the client responsible for "rendering the scene, synthesizing audio, and capturing user presence/interaction signals" while state mutation remains server-side.
- **Simulation Service as authoritative tick engine**: To execute the "tick" at a "~1-minute cadence," consume events, and produce updated snapshots.
- **Append-only event log**: To let the simulation consume interaction history such as "presence pings, offers, etc." without clients sending absolute state.
- **Clients pull snapshots and push interaction events**: To exchange "lightweight state snapshots" and append interaction events to the log.
- **Auth Service**: NOT RECOVERABLE FROM PLAN
- **Simulation Service**: To manage "aviary state, personality vectors, mood transitions, and the drift function."
- **Event Log Service**: To provide an "append-only store for user interactions."
- **Snapshot Service**: To serve "the current canonical aviary state to clients."
- **Social/Invite Service**: To manage "one-time visit links and revocation."

### 3. Data Model

- **Account encrypted email**: NOT RECOVERABLE FROM PLAN
- **Account settings**: To store "Accessibility, Social opt-in, etc." as account-level settings.
- **Aviary owned by account**: To model the "single canonical aviary per account."
- **Aviary bird_list**: To hold the list of Bird entities in the owned aviary.
- **Aviary notebook_entries**: To store generated naturalist observations.
- **Aviary last_tick_timestamp**: To support fetching "all new events since the last tick."
- **Bird stable UUID**: NOT RECOVERABLE FROM PLAN
- **Bird species_id**: NOT RECOVERABLE FROM PLAN
- **Bird user-assigned name**: NOT RECOVERABLE FROM PLAN
- **Bird personality_vector**: To hold traits the drift function can move toward expressive: "boldness," "social_warmth," "vocal_frequency," "plumage_saturation," and "curiosity."
- **Bird current_mood and last_mood_update**: To support mood transitions and mood-keyed behavior.
- **Event Log Entry**: To record interaction signals by `event_type`, optional `bird_id`, account, and timestamp for later tick processing.

### 4. API Surface

- **POST /events**: To append an interaction event to the log.
- **GET /snapshot**: To return the current aviary state, including birds, ambient mood, time, weather, and recent notebook entries.
- **POST /invites**: To let the host initiate an invite by email.
- **GET /visit/{token}**: To let a visitor pull a "read-only, non-interactive snapshot."
- **DELETE /invites/{email}**: To let the host revoke an invitation.

### 5. Simulation Engine Design

- **The Tick (~1 minute cadence)**: To make one process the only mutator of canonical state.
- **Fetch Log**: To retrieve new events since the last tick.
- **Presence Processing**: To calculate total presence-time from presence-pings.
- **Drift Computation**: To apply a low-pass filter to personality vectors using presence and interaction weights.
- **Presence > Listen-in > Offers weighting**: To make presence the strongest interaction weight in drift computation.
- **Mood Transition**: To update moods using recent interactions, local time-of-day, ambient weather/events, and personality-driven probabilities.
- **Notebook Generation**: To create a naturalist observation when notable recent events happen.
- **State Commit**: To save the new canonical state and clear processed events.
- **Drift calibration target**: To make change numerically measurable in about a week and visible to the user in about three weeks.
- **Monotonic drift constraint**: To ensure neglect "does not lower traits" and "simply fails to raise them."

### 6. Sync Model

- **Single Source of Truth**: To make server-side simulation the only authority.
- **No Client Ownership**: To prevent clients from sending absolute values and limit them to signals.
- **Additive Deltas**: To let the simulation engine compute deltas and prevent "last-write-wins" conflicts between devices.
- **Snapshot Interpolation**: To ensure smoothness by bridging the gap between ticks for position and pitch.

### 7. Frontend Rendering Pipeline

- **HTML5 Canvas or WebGL visual layer**: NOT RECOVERABLE FROM PLAN
- **Shader-based day/night transitions and ambient weather effects**: To create "Atmosphere."
- **Micro-motion**: To show continuous idle loops such as "preening, scanning, head-tilting" that are "mood-keyed."
- **Reduced-motion cross-fade posing**: To replace frame-by-frame animation while preserving ambient lighting shifts.
- **WebAudio API engine**: NOT RECOVERABLE FROM PLAN
- **Procedural Synthesis**: To generate calls through "motif libraries" and real-time pitch/timing modulation.
- **The Chorus**: To mix multiple procedural streams in real time.
- **Listen-in Mix**: To ramp gain for the focused bird while attenuating others.
- **Graceful silence fallback**: To keep the system usable with call captions if WebAudio fails.

### 8. Accessibility Surfaces

- **Hidden live region (`aria-live="polite"`)**: To receive naturalist prose updates for screen readers.
- **Screen-reader narration cadence**: To provide idle updates every "~30-60s" and immediate priority for user-initiated events.
- **Lowercase, present-tense naturalist voice**: To keep narration in the plan's example style: "a small grey bird is perched on the front rail..."
- **Call caption overlays**: To place small fading mood-driven descriptions near the calling bird.
- **Keyboard navigation**: To make the top bar tabbable, bird focus navigable by arrow keys, and listen-in/exit available through `Enter` and `Escape`.

### 9. Performance Budgets and Observability

- **JS Bundle <2MB (gzipped)**: To keep the initial bundle inside the stated budget.
- **Time to First Bird <500ms**: To make the first bird appear quickly on mid-tier mobile/4G.
- **Runtime 60fps**: To keep idle motion smooth on 5-year-old hardware.
- **Memory zero growth over 30 minutes**: To prevent session-long memory growth.
- **Synthetic Monitoring**: To test load and render timings globally with automated browser fleets.
- **RUM aggregate-only metrics**: To observe latencies, error rates, and frame drops without per-user data.
- **Privacy Boundary**: To avoid sending per-user or per-bird data to analytics.

### 10. Rollout

- **Phase 1 internal alpha**: To test stability of the simulation tick and drift calibration.
- **Phase 2 limited beta**: To calibrate the "feels alive" threshold and presence/drift balance.
- **Phase 3 public v1 launch**: NOT RECOVERABLE FROM PLAN
