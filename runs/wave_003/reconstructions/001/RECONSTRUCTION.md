## System-level intent

1. The aviary should feel "already alive" and quietly continuous. This shows up in the core product promise that it "continues without the viewer and notices the user without announcing them," in the frontend requirement that the "initial shell delivers a quiet field immediately," in "snapshot hydration" without "spinner-first framing," and in the performance risk that missing budgets breaks the "'already alive' illusion."

2. Canonical state belongs on the server, while the client is a disposable renderer and event emitter. The plan repeats this through "the server is the only writer," "clients never mutate bird state directly," the simulation service owning the "minute-scale tick," and sync rules where "client state is disposable and reconstructible from the latest snapshot plus local UI mode."

3. Relationship change is gentle, slow, and non-punitive. The plan blocks "Tamagotchi-style negative neglect mechanics, hunger, distress, or bird death," makes drift "monotonic upward" with "strong diminishing returns," says "neglect does not decrement personality values," and calibrates for "no visible single-session jumps."

4. Privacy and non-analytics boundaries are part of the product shape, not a later hardening pass. The plan hides "personality values" from product UI, retains presence only as needed for "simulation correctness," collects "only aggregate operational metrics," and excludes "per-account bird relationship data," "per-account interaction histories," and "population-level dashboards that summarize individual relationship dynamics."

5. Accessibility is first-class and must keep the charm of the main product. This appears in "first-class accessibility surfaces," identical notebook and interaction behavior under reduced motion, audio-off continuity through "captions and notebook/narration," and the mitigation that narration, reduced-motion, and captions are "launch-blocking first-class deliverables" rather than a "stripped fallback."

6. The product voice is observational, sparse, and naturalist rather than gamified or stat-based. The plan blocks "gamification, streaks, counters, achievements," asks notebook copy to use "naturalist lowercase prose," keeps narration "observational and naturalist, never stat-list based," and gives controls "matter-of-fact system voice where appropriate."

7. Birds should remain individually recognizable inside a bounded small flock. This appears in per-bird "personality vectors," "call signature seed," procedural call parameters by "bird-specific seeds, mood, and vocal frequency," chorus mixing that preserves "per-bird recognizability," and "mix caps at seven birds."

8. Social access is optional, quiet, read-only, and privacy-bound. The plan frames visits as "optional quiet visit invitations," keeps visits "default-off," backs them with "restricted capability token," prevents "notebook mutation" and "interaction posting," and launches them only after "host privacy, revocation, and read-only enforcement" are verified.

9. Performance budgets serve the product illusion. "First bird visible within 500ms," "first frame renders birds mid-action," "defer non-critical ornaments until after first-bird render," and CI/perf gate mitigations all support the plan's stated concern that performance failure breaks the "'already alive' illusion."

10. The system should create emergent-feeling behavior from bounded, calibrated rules. The plan uses "probabilistic within bounded rules" mood transitions, bird-to-bird "decay," "compatible call windows," rule-plus-template notebook generation, replayable simulation fixtures, and remotely configurable drift coefficients.

## Per-feature whys

1. Browser-only web client: NOT RECOVERABLE FROM PLAN

2. One canonical aviary: The plan's rationale is that the aviary needs one server-authored truth for simulation and sync. This is carried by "canonical server-side aviary simulation," "the server is the only writer," and conflict prevention that avoids merge UI because divergence should be "structurally prevented."

3. Single-user account: NOT RECOVERABLE FROM PLAN

4. Magic-link auth: NOT RECOVERABLE FROM PLAN

5. Account export, soft-delete, and recovery: NOT RECOVERABLE FROM PLAN

6. Two starter birds and gradual expansion up to seven birds: The rollout rationale is to keep early phases fixed while "drift and recognizability are tuned," then increase count while monitoring "audio recognizability, render stability, and notebook quality." Seven also appears as an audio mitigation: "mix caps at seven birds."

7. Server-authored simulation tick: The tick exists so mood, drift, notebook eligibility, weather windows, and call schedules continue as canonical state. The flow loads state and events, applies mood logic, computes drift, advances perch/pose/call intent, evaluates notebook entries, and persists a new canonical state.

8. Multi-device sync: The plan's rationale is to keep sessions coherent when devices open, resume, or act concurrently. Snapshots are versioned, stale responses are ignored, event logs serialize concurrent interactions, and suspended devices refresh before interaction surfaces that depend on cooldowns or settle state.

9. Procedural audio calls: The plan ties this to compact species motif libraries, runtime synthesis, bird-specific seeds, mood, and vocal frequency. It also rejects "poor recorded substitutes" by preferring silent-caption fallback if WebAudio is unavailable.

10. Field notebook entries: Notebook entries exist to capture "noteworthy state changes and rare combinations, not from every session." The plan uses "naturalist lowercase prose" and a sparsity gate of "one entry every few days" except for "genuinely distinctive moments."

11. Optional quiet visit invitations: The rationale is quiet sharing without changing the host aviary. Visit sessions are read-only, visitor activity "never writes host drift inputs," host metadata is limited to what is needed for "transparency," and visits launch only after privacy, revocation, and read-only enforcement are verified.

12. First-class accessibility surfaces: The plan says audio-off users still need a "coherent aviary experience" and warns that accessibility must not become a "stripped fallback that loses the product's charm." That is why captions, narration, reduced motion, keyboard traversal, and acceptance tests are launch-level requirements.

13. Aggregate operational observability: The rationale is to monitor load, frame timing, audio failures, API errors, tick latency, and backlog depth while excluding "per-bird state," "per-account interaction histories," and relationship-dynamics dashboards.

14. Three-surface architecture: The split lets the web client render, capture presence, synthesize audio, and present UI while the Application API handles auth/read models/invites/account surfaces and the Simulation service owns canonical tick progression.

15. Relational primary store: NOT RECOVERABLE FROM PLAN

16. Append-only event log: The event log supports presence and interaction events, server ordering, idempotency, deterministic replay, concurrent device serialization, and simulation ticks that consume validated events.

17. Short-lived cache: The stated purpose is "snapshot delivery and invite/session validation."

18. Email delivery integration: The stated purpose is "magic links, exports, and visit invitations."

19. Server-only authority for personality vectors, mood state, perch state, weather state, notebook generation records, and call schedule seeds: The rationale is to prevent client mutation of bird state and keep drift, mood, and snapshots canonical.

20. Client-owned ornamental motion: The rationale is that clients may add "leaf drift and subtle parallax" only when it is "explicitly non-canonical," preserving visual life without changing canonical state.

21. Visit sessions backed by the same snapshot read path: The rationale is read-only rendering through a "restricted capability token" with no notebook mutation, interaction posting, or unnecessary host metadata.

22. Snapshot data for first-frame rendering: The plan includes perch zone, pose family, interpolation target, call intent, mood, scene time-of-day, weather, and settle state so birds can be placed "mid-action on first paint."

23. Smoothing late or dropped snapshots: The rationale is to avoid "visual snapping" by treating the latest snapshot as truth and smoothing toward it.

24. Hidden personality values: The plan's rationale is that personality values stay out of product UI and accessible surfaces; only internal services and staff tooling can inspect them.

25. Additive drift deltas: The plan says drift must be "history-safe deltas, not replacement values from clients," aligning with server-only state and replayable simulation.

26. Presence raw event expiry and bounded aggregates: The rationale is to retain raw presence only for "simulation correctness" and avoid long-term data that can "reconstruct user behavior."

27. Immutable notebook entries: NOT RECOVERABLE FROM PLAN

28. Visitor activity never writing host drift inputs: The rationale is that visits remain read-only and cannot affect the host's bird relationships or canonical drift.

29. Aviary snapshot read model: The rationale is to provide a canonical scene snapshot with bird state, scene state, notebook unread count, capability flags, visit mode, and "reduced data needed for first-frame rendering."

30. Idempotent interaction events: The rationale is to handle duplicated client submissions through client-generated event ids.

31. Coarse-grained presence heartbeats: The rationale is to avoid "per-motion spam" while still reconstructing validated presence-time increments for simulation.

32. Server-side offer cooldown: The rationale is to enforce per-bird cooldowns centrally, including when multiple devices may emit interactions.

33. Settle cancel/re-engage undo window: The plan's rationale is an explicit "five-second undo path" and endpoint support for cancel/re-engage inside that undo window.

34. Visit token exchange into a short-lived read-only visit session: The rationale is first-entry control plus constrained ongoing read-only access.

35. Drift function: The plan's rationale is to make regular presence, listen-in, and offers slowly shape expressive traits. Presence is the "dominant driver," listen-in targets social warmth and vocal frequency, offers bias curiosity and boldness, and diminishing returns prevent jumps.

36. No negative neglect decrement: The rationale is to preserve the no-punishment product contract. "Neglect does not decrement personality values"; greeting frequency and visible expressiveness instead emerge from mood and current state.

37. Mood system: The rationale is to create persistent, varied behavior from daypart, weather, interactions, and personality. Transitions are "probabilistic within bounded rules," personality "shapes transition probabilities," and mood is "never reset on tab open."

38. Bird-to-bird behavior: The rationale is emergent local social behavior: alarm or wary signals propagate with decay, chorus can emerge from compatible call windows, and social warmth affects who greets first or perches nearer others.

39. Notebook rule-plus-template pipeline: The rationale is to detect first greeter changes, quiet stretches, mood/weather interactions, repeated attention, and milestones, then turn them into sparse "observed specifics" rather than generic session logging.

40. Canonical state propagation: The rationale is freshness after open, visibility regain, long suspension gaps, and visible periods; versioned snapshots let clients ignore stale responses.

41. Conflict prevention: The rationale is to make divergence structurally impossible: no client bird-state writes, append-only events with server ordering, and simulation tick transactions for personality and mood.

42. Multi-device owner-only contribution: The rationale is that concurrent owner devices can serialize interactions through the event log, while "only the account owner's authenticated sessions can contribute" presence.

43. Quiet field and no spinner-first frontend shell: The rationale is to put the aviary into a live scene immediately, consistent with "continues without the viewer" and the "'already alive' illusion."

44. Sparse top bar overlay: The plan ties this to quiet presentation and "accessibility affordances," with fade behavior rather than heavy persistent controls.

45. Responsive scene layout: The rationale is to "preserve all birds in frame on narrow viewports."

46. Client-side weather overlays and ornamental drift within strict limits: The rationale is to allow non-canonical atmosphere while keeping canonical weather in the snapshot and deferring non-critical ornaments until after first-bird render.

47. Reduced-motion path: The rationale is to reduce motion while keeping the same product behavior: still poses replace frame-by-frame motion, ornaments are removed, daypart transitions slow down, and "all interaction affordances and notebook behavior" remain identical.

48. Procedural synthesis runtime: The rationale is compact runtime synthesis from motif libraries with parameters from seeds, mood, and vocal frequency, rather than shipping recorded audio.

49. Audio mix model: The rationale is a recognizable multi-bird ambience. The mix is "always multi-bird," listen-in attenuates others "to ambient, not silence," chorus preserves "per-bird recognizability," and daypart/settle affect ambience and call density.

50. Captions and silent fallback: The rationale is that captions match "the actual synthesized call events" and audio-blocked users still get silent mode with captions by default and no recorded-audio fallback.

51. Screen-reader narration: The rationale is an accessible channel that comes from the same state graph as rendering and notebook logic, stays "observational and naturalist," and uses slow cadence, prioritization, rate limits, and user-triggered preemption to avoid queue flooding.

52. Input accessibility: The rationale is full keyboard traversal across top bar, birds, offers, notebook, and settings, with focus treatment that survives both bright and dim scene states.

53. Visual and auditory accessibility settings: The rationale is cross-device consistency and coherent non-audio experience: reduced motion and call captions are persisted per account, and audio-off users still receive captions plus notebook/narration continuity.

54. Performance budgets: The rationale is product quality and simulation headroom: first bird timing, 60fps idle rendering, no memory growth, and simulation tick p99 under five seconds all protect the live-scene experience.

55. Engineering tactics: The rationale is to stay inside budgets by code-splitting settings/export/visit surfaces, keeping snapshots small, reusing audio nodes and buffers, bounding allocations, and deferring ornaments until after first-bird render.

56. Rollout phases: The rationale is staged validation: dogfood checks presence, tick correctness, and audio reliability; beta adds lifecycle, notebook, accessibility, and invites behind flags; public launch keeps visits default-off and uses staged bird unlocks.

57. Ramping strategy: The rationale is to tune drift and recognizability before new bird unlocks, then monitor recognizability, render stability, and notebook quality as bird count increases, and launch visits after privacy and read-only enforcement are verified.

58. Day-one instrumentation: The rationale is operational validation of presence qualification, tick backlog, snapshot freshness, audio failure rates, and aggregate accessibility mode usage without user-behavior profiling.

59. Testing and validation strategy: The rationale is to verify specific plan guarantees: monotonic drift and no-negative-neglect, deterministic replay from ordered logs, multi-device ordering and invite revocation, first-frame snapshots, accessible narration/keyboard/captions/reduced-motion, long-session stability, and graceful unsupported-browser handling.

60. Early implementation decisions: The rationale is to resolve heartbeat timing, trait ranges, notebook thresholds, snapshot transport, and rendering technology in early engineering design reviews "without changing the PRD-level product contract above."
