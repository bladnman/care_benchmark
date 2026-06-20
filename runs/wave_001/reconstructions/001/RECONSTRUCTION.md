## System-level intent

1. Relationship deepening, not attention metrics. This shows up in Scope through the explicit exclusion of "Gamification," "Tamagotchi-style mechanics," "Notification surfaces," and "Streak counters or visit-frequency tracking"; in Rollout where bird availability "matches relationship deepening, not attention metrics"; and in Risks under "Relationship safety," where "gamification creep" and being pushed "back to counter instead of birds" are red flags.

2. Server-owned canonical state. The Architecture says the server "Owns all mutable state," runs the tick, computes drift, and provides snapshots. Sync names this directly as "Canonical state sovereignty," with the "Server is only source of truth for personality vectors, moods, and drift." The same intent appears in conflict prevention: "No client-submitted personality deltas" and "Server-side tick only writer of personality vectors."

3. Slow, presence-shaped expressiveness. The Drift function is a "Low-pass filter over presence-time + interaction signals," with the dominant input being "presence-time." Calibration is intentionally slow: "measurable drift after ~1 week, visible to user after ~3 weeks." The Risks section frames the design space: too fast "turns product into Tamagotchi"; too slow "turns into screensaver."

4. Individual birds must remain recognizable. The plan repeats stable identity through "Internal UUID," "Identity continuity," "Call signature," "recognizable signatures per bird," and "Each bird maintains unique signature across drift." Audio risks use the same standard: a red flag is when "Users cannot identify their bird by ear after weeks of use."

5. Naturalist voice over interface voice. The Field notebook uses "Naturalist prose strings, lowercase, present-tense, sparse." Screen-reader narration "matches field notebook," captions use the "Same naturalist voice," and the aviary scene contains "no user copy except top bar." Accessibility risk rejects narration that feels like "system announcement rather than observation."

6. Accessibility is part of the affective core. Scope includes narration, reduced-motion, captions, keyboard navigation, and focus management. Rollout says "All accessibility surfaces ship with product." Reduced-motion is "Alternative rendering, not fallback," with the "same affective core"; the risk red flag is a screen reader user feeling "the product is dumbed down for them."

7. Privacy boundary between operations and relationship data. Data Model encrypts email and uses a "Synthetic UUID." Observability is "Aggregate-only telemetry (no per-bird state)," with "Interaction history" never included. The plan names the principle: "Privacy boundary separates operational health from user relationship data."

8. Performance is part of the product promise. Scope sets "<2MB JS bundle," "first bird visible <500ms," "60fps," and "no memory growth over 30 minutes." Risks name the user-facing failure modes: first bird rendering after 500ms breaks the "product opening promise," and it is a red flag if the "User notices load state on first page open."

## Per-feature whys

### Scope

- Single-user accounts with magic-link email authentication: NOT RECOVERABLE FROM PLAN
- Single canonical aviary per account: The plan ties this to "same account/aviary" sync, so devices render identical state from one canonical source instead of syncing state between devices.
- Two starter birds and gradual birds-per-aviary ramp: The rollout why is that pacing "matches relationship deepening, not attention metrics."
- Seven-bird cap: The audio why is that "above this chorus blurs into ambient."
- Multi-device sync with server-side canonical state: The why is sync correctness: both devices render identical state because they read from the "canonical source."
- Bird engine with personality vectors and drift mechanics: The why is slow expressiveness from "presence-time + interaction signals" without client-submitted absolute values.
- Mood system with reset and session persistence: Mood shapes "idle motion, call patterns, perch position" and "carries to next start."
- Procedural call synthesis with recognizable signatures per bird: The why is recognizability across "drift and mood changes" while preventing a "canned feel."
- Audio pipeline with listen-in, chorus mixing, and WebAudio fallback: The why is focused listening, a "real-time mix, not stacked loops," and graceful silence with captions if WebAudio is unavailable.
- Field notebook with naturalist observations: The plan uses it as the product voice source for narration and captions: "lowercase, present-tense, sparse."
- Visit invitations: The why is a read-only, opt-in visit surface without "profiles, follows, public feeds, discovery."
- Screen-reader narration with slow cadence prose: The why is observation rather than "system announcement," using naturalist prose at an idle cadence.
- Reduced-motion mode with cross-faded transitions: The why is "same affective core" through "Alternative rendering, not fallback."
- Call captions with runtime descriptions: The why is to describe procedural calls in sync with call timing, and to become default when WebAudio is unavailable.
- Keyboard navigation and focus management: The why is to make top bar, aviary, focused birds, listen-in, escape, and offer affordances reachable.
- Single horizontal scene with three perch zones: NOT RECOVERABLE FROM PLAN
- Day/night cycle based on user's local timezone: NOT RECOVERABLE FROM PLAN
- Ambient weather with mood effects: The why is that ambient events shape mood and call behavior, for example "rain dampens frequency."
- Ambient micro-motion: The why is that "Birds never still," with continuous animations and mood-shaped idle motion.

### Explicit out-of-scope boundaries

- Native mobile apps: NOT RECOVERABLE FROM PLAN
- Web-only deployment: NOT RECOVERABLE FROM PLAN
- Gamification exclusion: The why is relationship safety: no "achievements, streaks, levels, scores, badges" and no drift that turns the product into "Tamagotchi."
- Tamagotchi-style mechanics exclusion: The why is to avoid "death, hunger, happiness meters" and keep drift calibrated between Tamagotchi and screensaver.
- Social network surfaces exclusion: The why is to keep visits read-only and opt-in, with no "profiles, follows, public feeds, discovery."
- Notification surfaces exclusion: The risk section says visitor notifications could push the user "back to counter instead of birds."
- Streak counters or visit-frequency tracking exclusion: The why is "relationship deepening, not attention metrics."
- UI chrome inside aviary view limited to top bar: NOT RECOVERABLE FROM PLAN
- Customizable scenes or geography features: NOT RECOVERABLE FROM PLAN
- Multi-aviary accounts: NOT RECOVERABLE FROM PLAN
- Payments or monetization features: NOT RECOVERABLE FROM PLAN

### Deliberately limited v1 details

- Procedural bird naming with user renaming capability: NOT RECOVERABLE FROM PLAN
- Identity continuity with stable bird IDs across account changes: The why is continuity across account lifecycle and separation from email-derived identity.
- Accessibility surfaces designed for charm, not checklist parity: The why is that accessibility should share the naturalist voice and "same affective core," not feel "dumbed down."
- Performance budgets for bundle, first bird, framerate, and memory: The why is avoiding load notices, preserving the "product opening promise," and keeping long sessions stable.

### Architecture

- Monolithic simulation service with separate API layer: The why is that one server owns "all simulation logic" and "all mutable state."
- PostgreSQL plus event store for canonical state: The why is append-only event ordering and canonical state replay without "last-write-wins."
- Redis for session tokens and temporary state: NOT RECOVERABLE FROM PLAN
- CDN for static assets and initial state snapshots: The why is fast first-bird delivery from the "CDN edge."
- Server-side simulation service: The why is canonical ownership of mutable state, tick, drift, calls, interaction events, and snapshots.
- Browser client rendering and procedural audio: The why is that the client renders snapshots, interpolates state, handles input, and manages WebAudio without owning canonical state.
- Canvas 2D/WebGL render boundary: NOT RECOVERABLE FROM PLAN
- Client interaction events: The why is to submit offers, listen-in, settle, and presence pings as inputs to server drift and mood updates.
- Server state snapshots: The why is to give clients bird positions, moods, call timing, and animations from the canonical source.
- Server tick data flow: The why is ordered personality updates, mood transitions, and drift calculations outside the client.

### Data model and API surface

- Internal bird UUID: The why is stable bird identity "across account lifecycle."
- Server-side personality vector: The why is server sovereignty over drift and prevention of client-submitted personality state.
- Personality vector never exposed to user: NOT RECOVERABLE FROM PLAN
- Mood enum persistence: The why is that mood "persists across sessions" and carries to the next start.
- User-assigned mutable bird name: NOT RECOVERABLE FROM PLAN
- Species from v1 pool of six: NOT RECOVERABLE FROM PLAN
- Perch position derived from mood and personality: The why is that mood shapes position, idle motion, and call patterns.
- Call signature with personality-shaped timing and pitch: The why is per-bird recognizability and audio identity across drift.
- Aviary visit relationship with one-to-many read-only visitors: The why is a limited visit surface without broader social network surfaces.
- Account email encrypted storage: The why is privacy around account linkage.
- Synthetic account UUID never email-derived: The why is to prevent "email-based conflicts."
- InteractionEvent, PresenceEvent, and TickEvent: The why is ordered simulation inputs for interactions, presence, and server simulation steps.
- Field notebook entry sparsity, one entry every few days: NOT RECOVERABLE FROM PLAN
- GET state, notebook, and invites endpoints: The why is client access to current snapshot, notebook entries, and pending invitations.
- POST events and auth endpoints: The why is submitting interaction events and completing magic-link session flow.
- Invite expiry after 30 days and revocation: NOT RECOVERABLE FROM PLAN
- Session flow from magic link to snapshot, WebAudio, interactions, tick, interpolation, and settle/tab close: NOT RECOVERABLE FROM PLAN

### Simulation engine and sync model

- Server-side tick at about one-minute cadence: The why is to consume the event log in order, update moods and drift, and run "regardless of client connection."
- Low-pass drift function: The why is slow calibration, with measurable drift after about a week and visible change after about three weeks.
- Monotonic drift toward expressive traits: The why is that traits "only increase, never decrease" toward expressiveness.
- Presence-time as dominant drift input: The why is that presence-time, not attention metrics, is the main signal for relationship deepening.
- Three-condition presence conjunction: NOT RECOVERABLE FROM PLAN
- Listen-in, offers, and settle as drift inputs: The why is targeted bird signal, small curiosity boost, and a "mood-quieting signal."
- Additive server-authored deltas: The why is to avoid client-submitted absolute values and prevent conflict.
- Mood transitions from interaction, time of day, ambient events, and personality: The why is to connect behavior to recent events, dusk, rain, and traits such as boldness.
- Call grammar runtime: The why is recognizable motifs, real-time chorus, and audible calls across front-to-back perch zones.
- Append-only event log and ordered consumption: The why is to prevent race conditions and avoid "last-write-wins for personality state."
- Multi-device propagation: The why is that valid sessions on any device get the current snapshot and render identical state.
- Offline handling: The why is that server simulation continues and the client resumes with the latest snapshot, batching pending events on reconnect.

### Frontend rendering, audio, accessibility, and rollout

- Layered canvas scene with background, sky, birds, and occasional foreground: NOT RECOVERABLE FROM PLAN
- Fixed perch zones with parallax: NOT RECOVERABLE FROM PLAN
- Bird sprites with species silhouette and plumage saturation effects: NOT RECOVERABLE FROM PLAN
- Idle mood micro-motion: The why is continuous life in the aviary, with wary, content, curious, drowsy, and alert motions.
- Motion independent of user attention: NOT RECOVERABLE FROM PLAN
- Settle transition to evening: NOT RECOVERABLE FROM PLAN
- Day/night palette shifts: NOT RECOVERABLE FROM PLAN
- Reduced-motion rendering details: The why is cross-faded still poses, slower color shifts, and no leaf drift while birds, calls, and notebook still continue.
- Frontend performance optimizations: The why is 60fps idle motion, no unreleased per-call allocation, and no retained notebook references after scroll-out.
- WebAudio nodes and motif libraries: The why is procedural variation, personality-shaped timing, and avoiding recorded audio in the bundle.
- Listen-in mix: The why is a gradual focus-bird rise while other birds remain ambient "but never silent."
- Listen-in disengagement triggers: NOT RECOVERABLE FROM PLAN
- WebAudio fallback: The why is "graceful silence with captions on by default" and continued visual-only presentation.
- Screen-reader narration queue: The why is naturalist prose every 30-60 seconds at idle, with user-initiated events getting a priority bump.
- Call captions generated from procedural grammar: The why is runtime descriptions such as call shape, timed to fade in and out with calls.
- Keyboard controls: The why is focus traversal, arrow-key bird focus, Enter listen-in, Escape exit, top-bar offer, and visible focus indicators.
- Contrast rules: The why is WCAG AA user-copy contrast while keeping the aviary scene free of user copy except top bar.
- Bundle size budget: The why is small initial load, supported by code-splitting, procedural audio, optimized bird assets, and CI measurement.
- Time-to-first-bird budget: The why is first bird visible under 500ms by using edge snapshots and skipping non-critical assets.
- Runtime performance budget: The why is 60fps and no memory growth over 30 minutes.
- Aggregate-only observability: The why is operational health without per-bird state or interaction history.
- Simulation-tick p99 alarm and synthetic checks: The why is to catch latency and performance regressions from automated browsers.
- All accessibility surfaces enabled by default in v1: The why is accessibility "from day one, not retrofitted."
- No feature flags: NOT RECOVERABLE FROM PLAN
- Instrumentation from day one: The why is monitoring bundle, first-bird timing, FPS, memory, audio errors, tick latency, revisit rate, and drift calibration.
- Retention measured by revisit rate: The why is measuring return behavior while preserving "no streaks."
- Monitoring priorities: The why is to detect drift calibration, sync correctness, audio uncanniness, accessibility regressions, and performance failures.
