## System-level intent

- Quiet, non-gamified ambient presence. This shows up in "No Gamification," "No Tamagotchi Mechanics," "No Push/Notification Surfaces," "No Social Network Features," and "No UI Chrome in Scene." The plan repeatedly excludes streaks, XP, death, distress, public directories, push notifications, and in-scene status surfaces.
- Gentle, non-punitive change over time. The plan says "Drift is monotonic toward expressiveness," "Neglect sets Signal = 0," "Traits never degrade," and neglect creates "ambient quietness, never hostility or sadness." The intent is that time and attention can make birds more expressive without creating decay.
- Server-authoritative state with no client personality writes. This appears in "server-side simulation tick" as "the single canonical author," "zero client-side personality mutations," "Single Writer Principle," and "Elimination of Last-Write-Wins." Client sessions append events; the backend simulation worker writes bird state.
- Privacy-preserving identity and observation. The plan uses "Synthetic UUIDs" for "zero PII leakage," encrypted email plus "HMAC for lookup without decrypting," raw scalar personality vectors that are "never exposed," and observability that strictly excludes per-bird trait values and user-bird relationship graphs.
- Naturalist product voice. The plan uses "naturalist prose," "lowercase, present-tense naturalist prose," "Matter-of-fact response," "sparse" notebook entries, and a risk mitigation against narration becoming "flooded or clinical."
- First-frame and ongoing aliveness without interruption. The plan asks for the aviary "already mid-motion," "quiet field fallback," "continuous micro-motion," "real-time day/night cycle," "ambient weather," "soft drifting leaves," and avoidance of "spinners or blank flashes."
- Accessibility and graceful fallback as normal surfaces. The plan includes screen-reader narration, reduced-motion mode, call captions, keyboard navigation, WCAG AA contrast indicators, and WebAudio fallback that enters silent mode and "automatically enables dynamic call captions."
- Optional, quiet social access under host control. This shows up in "Host-initiated email invitations," "read-only, ambient visitor access," "No co-presence," "no visitor drift impact," "instant revocation," "Silent visit logging," and no visitor push notifications by default.

## Per-feature whys

### Scope & Non-Goals

- Modern web browser platform with no native apps: NOT RECOVERABLE FROM PLAN
- Bird population starting with exactly 2 starter birds from a 6-species pool, expanding by aviary age up to 7 birds: NOT RECOVERABLE FROM PLAN
- Unannounced procedural return-greetings: The plan grounds these greetings in "absence length, boldness, and current mood," so the articulated rationale is that return behavior should be shaped by absence context, personality, and mood.
- Idle presence tracking: The plan later names the risk as "False Presence Inflation" and says backgrounded tabs can inflate presence hours; the three-factor conjunction of visible tab, window focus, and recent user input exists to avoid that.
- Listen-in interaction: The plan frames this as "focusing one bird" with "gradual dynamic mix rebalancing"; the mix raises the focused bird while other birds are "never full mute," preserving ambient balance.
- Offer gestures of seed, song fragment, and still pool: The simulation consumes "offers accepted," and a "Recent accepted offer" shifts mood toward "content" or "curious," so offers are interaction signals that can affect mood and drift.
- Per-bird cooldowns on offer gestures: NOT RECOVERABLE FROM PLAN
- Settle gesture: The plan calls it "opt-in" and gives a "5-second undo grace window," so the articulated rationale is a reversible, user-triggered soft evening lighting shift.
- Read-only Field Notebook: The plan says it generates "sparse, naturalist prose observations," evaluates "rare aviary occurrences," and emits at most 1 entry every "2-4 days," so its why is quiet observation rather than a feed of constant updates.
- Single horizontal screen with no scrolling, zooming, or panning: NOT RECOVERABLE FROM PLAN
- Three distinct perch zones: Birds choose front, middle, and back "autonomously by bird mood and personality"; later calibration says drift becomes perceptible through "perch proximity and feather richness."
- Real-time day/night cycle anchored to local timezone: The frontend sky gradient matches "local sun position," and the mood model weights dusk/night local time toward "drowsy."
- Ambient weather: Weather gives visual atmosphere and also affects mood; passing rain "nudges toward wary or drowsy."
- Continuous micro-motion: Breathing, weight shuffle, head tilts, and preening express idle aliveness, with motion parameterized by mood, curiosity traits, audio events, and content mood.
- Autohiding top-bar chrome: The plan ties this to stillness and interaction: the chrome "fades after stillness" and "restores on interaction," while the aviary scene itself excludes tooltips, tags, status bars, and handles.
- Initial load already mid-motion with quiet field fallback: The render pipeline draws birds "immediately in mid-pose" and uses a tranquil ambient sky if fetch exceeds 150ms, "avoiding spinners or blank flashes."
- Email magic-link accounts with 15-minute, single-use, per-device revocable sessions: NOT RECOVERABLE FROM PLAN
- Server-side simulation tick as canonical author: The plan says this is the "single canonical author of personality vectors and mood" and that only the backend simulation worker writes bird personality and mood.
- Append-only client interaction event log with zero client-side personality mutations: The plan uses this to avoid "last-write-wins conflicts"; multiple sessions simply append presence pings, then the server tick processes them chronologically.
- Synthetic UUIDs for internal identities: The stated reason is "to guarantee zero PII leakage."
- On-demand JSON snapshot export: NOT RECOVERABLE FROM PLAN
- 30-day soft deletion before hard purge: NOT RECOVERABLE FROM PLAN
- Host-initiated email invitations for read-only visitor access: The plan positions social as "Optional & Quiet" and "read-only, ambient visitor access."
- Visitor limits with no co-presence, no drift impact, no interactions, and instant revocation: The rationale in the plan is to keep visitor access read-only and prevent visitor sessions from changing aviary state.
- Silent visit logging and no visitor push notifications by default: The plan keeps visit records in account settings while avoiding unsolicited visitor notification surfaces.
- Screen-reader running narration: The plan provides an `aria-live="polite"` region with naturalist descriptions updated every 30-60s or on user actions; the risk mitigation caps updates to avoid a queue that is "flooded or clinical."
- Dedicated reduced-motion mode: The plan replaces micro-motion, hops, flights, and particle drift with static poses and cross-fades, making the same aviary available with reduced motion.
- Dynamic call captions: Captions are generated from procedural motifs, and if WebAudio is unavailable the app enters silent mode and "automatically enables dynamic call captions."
- Full keyboard navigation with WCAG AA contrast indicators: The plan routes focus through top bar items and birds, uses arrow keys and Enter/Escape for listen-in, and requires focus rings above 3:1 contrast.
- Performance budgets: Time-to-first-bird, 60fps idle motion, and a flat 30+ minute heap profile exist to keep the aviary responsive, moving, and free of memory leaks.
- No gamification: The plan excludes streaks, levels, XP, points, badges, adoption counters, visit calendars, and milestone toasts, keeping the aviary out of those reward structures.
- No Tamagotchi mechanics: The plan says birds never die, starve, decay, or exhibit distress; neglect creates "ambient quietness, never hostility or sadness."
- No social network features: The plan excludes public directory, discoverability, comments, chat, visitor avatars, leaderboards, and "show-off" rendering modes, keeping social access quiet and non-public.
- No push or notification surfaces: The plan excludes unsolicited emails and push notifications about aviary status.
- No UI chrome in scene: The plan excludes tooltips, tags, status bars, and drag-and-drop handles inside the aviary scene.

### Architecture & System Topology

- Server responsibility split: The server is responsible for the authoritative personality vectors, base mood timers, append-only ingestion, background tick execution, notebook prose, magic-link auth, and visitor authorization.
- Client responsibility split: The client renders the Canvas/WebGL/SVG scene, procedural micro-motion, WebAudio calls, precise presence detection, narration queueing, caption rendering, and keyboard focus routing using server-provided mood and personality parameters.
- Realtime Gateway snapshot stream: The plan gives it "Snapshot Stream," "Ephemeral state," and "Push state diffs," so its articulated role is propagating snapshot changes.
- Redis cache and queue with tick lock, recent events, and magic link OTPs: NOT RECOVERABLE FROM PLAN

### Data Model

- `accounts.encrypted_email` and `accounts.email_hash`: The plan explicitly says the hash is "HMAC for lookup without decrypting."
- Account settings for visit notifications, reduced motion, and captions: The defaults encode quiet visitor notifications and user-controlled accessibility settings.
- Bird personality vector fields: The plan describes them as normalized scalar floats in the range 0.0 to 1.0 and enforces a check constraint, so their why is bounded, comparable trait state.
- Bird fast-timescale dynamic state: `current_mood`, `current_perch`, `mood_updated_at`, and `last_call_at` support mood, perch, and call behavior that can change faster than long-term traits.
- `interaction_events`: The table exists for append-only event ingestion, and the tick loop consumes presence pings, listen-in durations, accepted offers, and settle triggers from it.
- `field_notebook_entries`: The table stores `entry_prose` and `noteworthy_event_type`, matching the notebook generator's rare naturalist observations.
- `visit_invitations`: Expiration, revocation, last visit time, and total visit duration support read-only visitor access, host revocation, and silent visit logging.

### API Surface & Protocols

- Magic-link request voice: The plan specifies a "Matter-of-fact response" and a single-use token valid for 15 minutes, tying auth copy to the product voice.
- Complete aviary state snapshot: The response includes weather, birds, mood, perch, visual params, audio params, and interaction delta so the client can render and synthesize from canonical state.
- Hiding raw scalar personality vectors from `GET /api/v1/aviary/state`: The stated note is that raw `trait_boldness` and related vectors are "never exposed."
- Batch event ingestion through `POST /api/v1/aviary/events`: The endpoint appends client events directly to `interaction_events`, supporting the append-only model.
- Notebook pagination from newest to oldest: NOT RECOVERABLE FROM PLAN
- Visit state endpoint rejecting event submissions: The plan says visitor sessions are read-only and event submission endpoints are rejected.
- Host-only visit revocation endpoint: The plan ties revocation to host control and "instant revocation."

### Simulation Engine & Drift Runtime

- Sixty-second server-side simulation tick cadence: NOT RECOVERABLE FROM PLAN
- Event consumption in the tick loop: The tick aggregates validated presence minutes, listen-in durations, accepted offers, and settle triggers so those inputs can drive drift and mood.
- Asymmetrical, monotonic leaky accumulator drift: The monotonicity constraint makes trait change non-negative; neglect produces "zero downward drift," and calibration makes drift measurable after 7 days and perceptible after 3 weeks.
- Mood transition Markov model: The model makes mood respond to local dusk/night, recent accepted offers, passing weather, and trait boldness.
- Notebook observation generator: It watches for rare events such as greeting order changes or content mood through rain, limits entries to at most every 2-4 days, and writes lowercase present-tense naturalist prose.

### Sync Model & State Propagation

- Single Writer Principle: Only the backend simulation worker writes birds and updates personality/mood, eliminating client-side trait conflict.
- Snapshot consumption on tab open, visibility return, and every 60s while visible: The client refreshes from canonical snapshots when the tab opens, returns, or remains visible.
- Cubic Hermite spline interpolation for perch transitions: The stated reason is "preventing teleportation."
- Chronological server processing of multi-device events: Because clients never send trait updates, concurrent devices append presence pings and the server tick processes them chronologically.

### Frontend Rendering Pipeline & Scene Composition

- Canvas 2D or SVG responsive rendering target: NOT RECOVERABLE FROM PLAN
- Background, middle ground, and foreground planes: The plan separates sky, silhouettes, parallax, perches, birds, foliage, leaves, and feather drift into scene layers.
- First-frame aliveness: Birds are drawn in mid-pose from snapshot timestamps, and the fallback background appears after 150ms to avoid spinners or blank flashes.
- Breathing oscillation, weight shuffle, head tilts, and preening: These procedural kinematics make idle birds visibly alive and tie behaviors to audio events, curiosity traits, and content mood.
- Reduced-motion rendering details: In reduced-motion mode, loops are disabled, perch hops and flights become 600ms opacity cross-fades, particles are disabled, and day/night transitions use gentle 5-second cross-fades.

### Procedural Audio Pipeline

- WebAudio graph architecture: The graph takes motif grammar through oscillator/formant shaping, bandpass filtering, ADSR envelope, per-bird pan/gain, listen-in attenuation, and master chorus.
- Dual oscillator FM and formant synthesis: The plan says this reproduces "natural avian syrinx harmonics."
- Call cadence tied to `trait_vocal_frequency` and weather: Inter-call intervals are driven by vocal personality and ambient weather dampening.
- Listen-in mix levels: The focused bird ramps up +4dB while others ramp down to -14dB, "never full mute," then all restore to ambient balance.
- Graceful WebAudio fallback: If `AudioContext` fails, the system enters silent mode, enables captions, and loads "No canned or looped audio files."

### Accessibility Surfaces

- `aria-live="polite"` screen-reader narration: The plan uses rolling naturalist descriptions at a relaxed pace, with immediate updates for offer or settle actions.
- Call caption bubbles: Captions float near the calling bird, derive text from the active motif, and fade out over 2 seconds.
- Keyboard and focus navigation: Tab, Shift+Tab, arrow keys, Enter, and Escape make top bar items and birds navigable, with WCAG AA visible focus rings.

### Performance Budgets & Observability

- Initial bundle, time-to-first-bird, frame, and memory budgets: The plan budgets load size, 4G first bird, rock-solid 60fps, and flat heap over 30+ minute sessions.
- Aggregate observability: API latencies, simulation tick time, render FPS drops, WebAudio error rates, and anonymized session duration histograms are measured.
- Strictly excluded observability: The plan excludes logging or aggregation of per-bird trait values, individual interaction frequencies, and user-bird relationship graphs.

### Rollout & Aviary Scaling Plan

- Phase 1 Alpha Foundation: The plan calls this "Alpha Foundation" and includes the core WebAudio engine, 2 starter species, server tick, append-only ingestion, and presence conjunction verification.
- Phase 2 Beta Experience: The plan calls this "Beta Experience" and adds the full 6-species pool, notebook prose, naturalist narration, reduced-motion mode, and call captioning verification.
- Phase 3 General Availability: The plan places aviary age-based scaling unlocks and read-only visit invitations in "General Availability."

### Risk Analysis & Mitigation Strategies

- Drift calibration simulation tests: The risk is traits saturating too quickly or failing to move noticeably; the mitigation validates drift envelopes for 1, 7, 21, and 60 virtual days.
- False presence mitigation: The risk is backgrounded tabs inflating presence hours; the mitigation requires visible document state, window focus, and recent user input.
- Audio uncanniness mitigation: The risk is calls sounding "robotic or grating"; the mitigation uses multi-oscillator formant synthesis, micro-randomized pitch, attack envelopes, and ambient silence pauses.
- Multi-device clock skew mitigation: The risk is client timestamp inconsistencies; server-authoritative ingestion timestamps are assigned on HTTP receipt, while client timestamps are only for relative sequencing.
- Accessibility queue mitigation: The risk is narration becoming "flooded or clinical"; the mitigation caps announcements and enforces naturalist copy templates in automated linting.
