## System-level intent

**Naturalist product voice, not game pressure.** The plan frames the product requirements as "naturalist product requirements" and names the journal a naturalist "Field Notebook." It carries that voice through "naturalist prose narration for screen readers" and "naturalist sentences." The same intent shows up in the boundaries: "Any form of gamification (streaks, levels, counters)" is out of scope, and "Custodial mechanics" are out of scope because "birds do not die or get hungry."

**A calm, bounded aviary.** The plan keeps the bird count deliberately bounded: "Initial adoption of 2 birds, capping at 7." Rollout repeats that shape: "Internal Alpha: 2 birds," "Beta: 5 birds," and "v1 Launch: 7 birds max." This presents the aviary as a small observed place rather than an unbounded collection.

**Server-authoritative, canonical simulation.** The architecture says the system uses a "Server-Authoritative Simulation" and that "Clients are thin rendering and interaction-capture shells." Sync repeats the same principle as "No Client Authorship": clients "never compute drift" and only report interaction duration/type. Multi-device state is grounded in "server-side canonical simulation," "State Snapshots," and "Snap-to-State."

**Bird change is slow, monotonic, and non-punitive.** Personality drift is described as "monotonic toward expressive" and later as "The 'Slow Current.'" The drift rule says "No signal = 0 change" and "Birds never become more wary." Calibration is meant to make visible change take "~3 weeks," and the risk section warns that "Too fast = Tamagotchi; too slow = Static."

**Mood is weather-like and time-aware.** The mood system is named "The 'Weather'" and is a "state machine: Alert -> Content -> Drowsy." Transitions are influenced by "Local_Time," "Time-of-Day," ambient events, and recent interactions, with examples like "Drowsy increases at sunset." The rendering pipeline mirrors this with "Day/Night" overlays tied to local `Date.now()`.

**Procedural life over static playback.** The plan says birds "don't play MP3s; they follow motifs." Calls use pitch/timing deltas and runtime "Variation," while the frontend uses procedural "breathing" and "scanning." The risk section treats procedural audio quality as important by calling out "Audio 'Uncanniness'" and mitigating "beep-boop" sounds.

**Social interaction is opt-in and read-only.** Social scope is limited to "Read-only visit invitations (per-invite opt-in)." Out-of-scope social features include "profiles, chat, public discovery." The API preserves that boundary with a host issuing an invitation and a visitor pulling a "read-only snapshot."

**Accessibility and performance are product constraints.** Accessibility appears in scope and gets dedicated surfaces: narration, captions, keyboard, and reduced motion. Performance has concrete budgets: bundle "<2MB Gzipped," first bird visible under "<500ms," and object pooling to prevent "GC pauses."

## Per-feature whys

### Scope and Boundaries

**Initial adoption of 2 birds, cap at 7, scaling based on aviary age:** NOT RECOVERABLE FROM PLAN. The plan states the bird counts and age-based scaling, then repeats the rollout counts, but it does not articulate why 2, 5, or 7 are the chosen numbers or why age should control scaling.

**Personality drift:** The plan's rationale is to let birds become more expressive over time through presence without punishing absence. It says drift is "monotonic toward expressive," presence increases "Boldness/Social Warmth," "No signal = 0 change," and "Birds never become more wary." The calibration target of "~3 weeks" is there to avoid drift being "Too fast" or "too slow."

**Fast-timescale mood:** The plan uses mood to make state respond to time and recent interaction. It names mood "The 'Weather'," transitions among "Alert -> Content -> Drowsy," and ties probability to "Local_Time" and "Recent_Offer," with drowsiness increasing at sunset.

**Procedural call synthesis:** The plan rejects static audio playback: "Birds don't play MP3s; they follow motifs." The why is variety and living texture through "Random jitter" on pitch and timing, while risk mitigation addresses procedural calls sounding too "beep-boop."

**Presence tracking:** Presence exists as the signal that drives slow personality change. The plan says "1 minute of validated presence" adds to "Boldness/Social Warmth," and the client reports "duration/type of interaction" rather than computing drift.

**Listen-in:** Listen-in exists to focus the audio mix on one bird. The plan describes "audio re-balancing," then specifies a focused bird's `GainNode` ramps to 1.0 while other birds are ducked to 0.1.

**Offers: seed, song, pool:** Offers exist as interaction events that can affect mood. The tick ingests "Offer" events, and mood transition probability is influenced by `Recent_Offer`. The plan does not articulate why the three offer types are specifically seed, song, and pool.

**Settle gesture:** NOT RECOVERABLE FROM PLAN. The plan includes `SETTLE` as an interaction event but gives no specific rationale for the gesture.

**Auto-generated Field Notebook:** The notebook carries the naturalist product voice into journaling. The plan calls it a naturalist "Field Notebook" and stores `entry_text` with a `timestamp`, but it does not describe the exact generation trigger or editorial rules.

**Single-user magic-link auth:** NOT RECOVERABLE FROM PLAN. The plan specifies "Single-user magic-link auth," `/auth/request-link`, `/auth/verify`, JWT return, and `email_hash`, but does not explain why magic links are the chosen account mechanism.

**Multi-device state sync:** The rationale is consistency through a server-side canonical simulation. The plan says multi-device sync is via "server-side canonical simulation"; clients get the latest snapshot, return from background by requesting a new snapshot, and avoid "time travel" through hard sync to server time.

**Read-only visit invitations:** The rationale is to allow visits without creating a social network or allowing visitor authorship. The plan says invitations are "read-only," "per-invite opt-in," and excludes "profiles, chat, public discovery." Visitors pull a read-only snapshot.

**Naturalist prose narration for screen readers:** The rationale is accessible narration in the same product voice. The plan converts `StateSnapshot` into "naturalist sentences" and injects them into an `aria-live="polite"` region.

**Reduced-motion cross-fade mode:** The rationale is to preserve the experience while disabling path animations when `prefers-reduced-motion` is true. The plan swaps animation-frame updates for "2-second cross-fades between keyframe poses."

**Call captions:** The rationale is to represent bird calls as text tied to birds' positions. The plan displays call motifs like "a low trill" in positioned elements that track bird coordinates.

**No gamification:** The rationale is a non-game boundary for the experience. The plan explicitly excludes "streaks, levels, counters," and the drift risk says too-fast change would become "Tamagotchi."

**No custodial mechanics:** The rationale is that the aviary should not make birds die or get hungry. The plan states this directly: "birds do not die or get hungry."

**No social network features:** The rationale is to keep social limited to controlled visits. The plan excludes "profiles, chat, public discovery" while keeping read-only visit invitations.

### Architecture

**Server-Authoritative Simulation:** The rationale is canonical state and consistency across devices. The plan says clients are thin shells, the server advances "aviary state" for all accounts, and clients "never compute drift."

**Simulation Service with a Universal Tick:** The rationale is to advance all account state centrally: "drift, mood, position." The tick also lets the system collect events, evaluate drift, transition moods, generate snapshots, and broadcast low-latency state.

**API Gateway:** The rationale is to separate auth, snapshot delivery, and event ingestion. The plan says it handles "magic-link auth, state snapshot delivery, and append-only event ingestion."

**React web client:** The rationale is client-side rendering and interaction capture while leaving simulation authorship to the server. The client handles "WebGL rendering, WebAudio synthesis, and presence/interaction logic" as a thin shell.

**Event Log ingestion:** The rationale is append-only interaction capture before simulation evaluation. The tick collects "Offer, Listen-in, Presence Pings" from the Event Log, and Redis stores recent interactions that are "drained by the Ticker."

**Low-pass drift evaluation:** The rationale is damped, gradual personality change. The plan uses a low-pass filter for the drift function and log-based dampening so visible change takes "~3 weeks."

**Static JSON state snapshots:** The rationale is to give clients a canonical renderable state. The plan's "Snap" step generates bird perches, mood states, and call motifs, while `GET /aviary/state` returns the latest snapshot for initial load and re-sync.

**Distributed cache for snapshots:** The rationale is "low-latency client retrieval" and "immediate delivery to clients." The plan stores the latest canonical state in Redis.

### Data Model

**Accounts:** The plan gives account fields for identity, settings, and auth linkage, but the specific rationale for `email_hash` and `settings` is NOT RECOVERABLE FROM PLAN.

**Aviaries:** The plan stores `age_days` and `last_tick_at`, supporting age-based scaling and tick timing. It does not further explain why those fields are sufficient.

**Birds:** The plan stores `personality_vector`, `current_mood`, and `last_interaction_at` because drift, mood, and interaction recency are core simulation state.

**Field Notebook records:** The plan stores `entry_text` and `timestamp` to persist generated notebook entries in time.

**Visit Invites:** The plan stores host, visitor, token, status, and expiry to support per-invite read-only visits.

### API Surface

**`POST /auth/request-link`:** The plan's rationale is to trigger magic link email for auth.

**`POST /auth/verify`:** The plan's rationale is to consume the link and return both a JWT and "initial state snapshot," joining auth and first render state.

**`GET /aviary/state`:** The rationale is initial load and re-sync from the latest canonical snapshot.

**`POST /aviary/events`:** The rationale is append-only reporting of `PRESENCE`, `LISTEN_IN`, `OFFER`, and `SETTLE` for the server tick to process.

**`POST /social/invite`:** The rationale is host-issued invitations rather than public discovery.

**`GET /social/visit/:token`:** The rationale is token-based access to a read-only snapshot.

### Simulation Engine Design

**Presence signal:** The rationale is to convert validated presence into slow expressive change: "1 minute of validated presence" adds to "Boldness/Social Warmth."

**Neglect rule:** The rationale is non-punitive absence. "No signal = 0 change" and "Birds never become more wary."

**Drift calibration:** The rationale is pacing. Visible change should take "~3 weeks" so the experience is neither "Tamagotchi" nor "Static."

**Mood state machine:** The rationale is lightweight state transitions among "Alert," "Content," and "Drowsy" driven by local time and recent offers.

**Call motifs:** The rationale is procedural calls built from "a sequence of pitch/timing deltas" instead of MP3 playback.

**Runtime call variation:** The rationale is variation through pitch jitter "±5%" and timing jitter "±10%."

### Sync and Consistency

**No Client Authorship:** The rationale is authoritative drift and consistency. Clients "never compute drift" and only report interaction duration/type.

**Spring-based interpolation:** The rationale is smooth movement: "ensuring 60fps movement even if the server tick is slow."

**Snap-to-State on return from background:** The rationale is to prevent "time travel" by hard-syncing to current server time.

### Frontend Rendering Pipeline

**Single HTML5 Canvas with WebGL context:** NOT RECOVERABLE FROM PLAN. The plan names the scene composition approach but does not articulate why this rendering surface was chosen.

**Idle micro-motions:** The rationale is procedural visible life: "breathing" through scale pulsing and "scanning" through head rotation, driven by per-mood shaders.

**Day/Night overlay:** The rationale is local time-of-day visual atmosphere: night uses `mix-blend-mode: multiply`, golden hour uses `soft-light`, and both are tied to local `Date.now()`.

**Reduced-motion rendering behavior:** The rationale is to disable path animations and replace them with cross-fades when `prefers-reduced-motion` is true.

### Audio Pipeline

**Oscillator/Gain synthesis:** The rationale is to "simulate bird calls" through WebAudio rather than use MP3s.

**Spatialization:** The rationale is positional audio: `PannerNode` maps birds to left, center, or right based on horizontal scene position.

**Listen-In Mix:** The rationale is focused listening, with a "2-second linear ramp" on the focused bird and ducking other birds.

### Accessibility Surfaces

**Narration Engine:** The rationale is screen-reader access to the latest state in naturalist prose, using `StateSnapshot` templates and `aria-live="polite"`.

**Captions over the canvas:** The rationale is visible text for call motifs, placed over the canvas and tracking bird coordinates.

**Keyboard bird interaction:** The rationale is keyboard access to birds: tab-index management makes birds reachable, and `Space/Enter` triggers `LISTEN_IN`.

### Performance Budgets

**Bundle under 2MB gzipped:** The rationale is lightweight delivery. The plan says to "Avoid heavy frameworks" and use "standard Web APIs."

**Initial snapshot embedded in first HTML response:** The rationale is to hit the "<500ms first-bird-visible target."

**Object pooling for audio nodes and render particles:** The rationale is to prevent "GC pauses."

### Rollout and Risks

**Internal Alpha, Beta, and v1 Launch phases:** NOT RECOVERABLE FROM PLAN. The plan names the staged feature sets but does not articulate why those exact capabilities are assigned to each phase.

**Sequential event-log processing for sync races:** The rationale is deterministic conflict handling when a user interacts on two devices simultaneously. The mitigation says the server processes the event log sequentially and the "last-processed event defines the current mood."

**High-pass filters and subtle noise injection:** The rationale is to reduce audio "Uncanniness" and avoid procedural calls sounding too "beep-boop."

**Server-side drift-weight toggle:** The rationale is calibration without client deployment. The plan says drift that is too fast becomes "Tamagotchi" and too slow becomes "Static," so server-side weights must be adjustable.
