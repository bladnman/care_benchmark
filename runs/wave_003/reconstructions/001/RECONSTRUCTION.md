## System-level intent

1. Continuity, procedural variation, and restraint over breadth. The plan states this directly in "Product intent and v1 scope": the primary engineering goal is "not feature breadth; it is continuity, procedural variation, and restraint." This shows up in the "small set of birds," "living continuously," "changing slowly over weeks," the one-scene v1, the slow simulation tick, sparse notebook entries, and the repeated launch posture of "quality-first v1."

2. A strict canonical simulation boundary. The plan repeatedly treats the server-side tick as the authority: it is "the only writer of canonical bird personality, mood, position, active weather, and notebook-trigger state," while the browser "never computes or persists personality drift" and "never sends absolute personality, mood, or position values." In sync, conflicts are "prevented by authority boundaries rather than resolved by last-write-wins."

3. Internal traits should be felt, not shown as stats. The plan calls for "hidden per-bird personality vectors" visible only through "motion, calls, perch choice, color richness, greetings, and notebook observations." It also bans "any user-visible numeric trait/stat surface" and says not to "add user-visible stats to make drift legible."

4. The product voice is quiet, naturalist, and non-announcing. Product-facing prose is limited to "aviary observations, notebook entries, captions, and narration," while account/error/settings surfaces are "matter-of-fact." This shows up in "No text welcome," no "you have been away" copy, notebook entries as "naturalist prose, not event logs," and the final success condition that "a bird noticed them without the system announcing it."

5. Privacy and data minimization are part of the architecture, not an add-on. The plan uses encrypted email plus lookup hashes, says no log line or metric tag should use email, keeps interaction events as "private simulation inputs," forbids per-account or per-bird telemetry dimensions, and says "No simulation database reads from analytics jobs."

6. Variation must be deterministic enough to debug. The plan asks for "deterministic seeded procedural choices" so "variation is real but debuggable," and then applies that to greetings, idle beats, weather, calls, notebook candidates, tick idempotency, mood transitions, call scheduling seeds, and simulation fixtures.

7. Accessibility is a first-class product surface. The plan says accessibility "must ship in v1, not as a later patch," and that screen-reader and reduced-motion users should get "a designed aviary, not a static fallback." This intent appears in captions, narration, keyboard navigation, contrast tokens, reduced-motion renderer, and assistive-tech beta.

8. The build should defend the quiet product from gamification and social leakage. The plan explicitly excludes "leaderboards, achievements, scores, streaks," public discovery, profiles, chat, comments, hunger/death/distress mechanics, and push notifications. It warns that many exclusions are "cheap to add and expensive to undo," and the guardrails say that if a feature requires showing a number, it "probably violates the product."

## Per-feature whys

### 1. Product intent and v1 scope

1. Email magic-link accounts with one canonical aviary per account: Why: the v1 product is built around "one private, persistent aviary per account," with account creation leading directly to the starter aviary and no password surface.

2. Two starter birds and server-paced expansion up to seven birds based on aviary age: Why: v1 should begin with "a small set of birds," protect quality and audio recognizability, avoid "earn" language, and let the bird ramp hold until "audio quality catches up."

3. A single horizontal responsive aviary scene with three perch zones, local-time day/night, rare ambient weather, and continuous ambient motion: Why: one scene supports the feeling that birds have been "living continuously"; local time, weather, perch zones, and motion make continuity visible without broad feature scope.

4. Server-side simulation tick as the only writer of canonical state: Why: it preserves the authority boundary, keeps personality and mood canonical, and prevents client-side state writes from corrupting the core mechanic.

5. Hidden per-bird personality vectors: Why: the plan wants personality expressed through behavior, calls, perch choice, color richness, greetings, and notebook observations, not as a user-visible numeric surface.

6. Mood-shaped idle motion: Why: mood becomes observable through pose, call probability, perch preference, and micro-motion without exposing raw mood labels or stats.

7. Return-greetings: Why: they let a bird notice the user through action rather than through "No text welcome" or "you have been away" copy.

8. Listen-in: Why: it focuses attention on one bird by ramping that bird up while other birds ramp down "but never to silence," and it contributes only small slow drift.

9. Offers: Why: they are "gestures from the top bar, not direct manipulation of birds"; cooldowns prevent "single-session trait saturation" and the copy must avoid feeding or caretaking implications.

10. Settle: Why: it creates a quiet state transition, ends the current presence window, quiets calls and lighting, allows a five-second undo, and makes closing without settle neutral.

11. Field notebook: Why: it gives sparse, stable, naturalist observations of meaningful state without becoming event logs, user-behavior summaries, counters, or a feed.

12. Procedural client-side WebAudio calls and captions from the same grammar: Why: call variation remains procedural and recognizable, while captions describe the "actual procedural call parameters" for accessibility and fallback.

13. Multi-device sync by server snapshot plus append-only client interaction events: Why: devices converge through canonical versions and ordered events, avoiding client-to-client merge and last-write-wins personality overwrites.

14. Read-only visit invitations that are off by default, revocable, and expire after 30 days: Why: visits allow sharing without visitor writes, host drift effects, public discovery, or social-network mechanics.

15. First-class accessibility surfaces: Why: the plan requires screen-reader narration, call captions, reduced-motion rendering, keyboard navigation, and WCAG AA contrast in v1 so accessibility is product work, not cleanup.

16. Account export, soft deletion, device session management, and privacy-respecting aggregate operational telemetry: Why: these account surfaces support user control and privacy while keeping operational telemetry aggregate and bounded to allowed fields.

### 2. Architecture

17. TypeScript web stack end to end: Why: simulation formulas, snapshot schemas, and client rendering contracts can "share types without sharing authority."

18. Relational primary store plus durable append-only interaction event table: Why: the plan wants canonical server state and ordered private simulation inputs "rather than client-sourced document sync."

19. Small server-rendered boot payload or edge-adjacent snapshot endpoint with account surfaces code-split away: Why: it is for "time-to-first-bird" and keeps non-aviary surfaces out of the initial aviary route.

20. Deterministic seeded procedural choices: Why: greetings, idle beats, weather, calls, and notebook candidates can vary while remaining "real but debuggable."

21. Four runtime surfaces: Why: the browser renders and records presence, the API service handles auth and privacy boundaries, the simulation service owns slow canonical updates, and background workers keep email/export/deletion/synthetic checks outside the user request path.

22. Strict browser/server boundary: Why: browser code may interpolate and synthesize, but not own drift or merges; server code may expose prose but must not expose numeric personality values on product surfaces.

23. Object storage for generated export files with short-lived signed URLs: Why: exports can be produced asynchronously and delivered through expiring download links.

24. CDN/edge caching for static assets and unauthenticated shell only: Why: static performance is allowed, but "authenticated snapshots are not publicly cached."

25. Internal admin and debug tooling can inspect numeric values only in protected environments: Why: debugging and calibration need internal visibility, but product surfaces and account export UI must not render values as stats.

### 3. Data model

26. Synthetic UUIDs plus encrypted email and keyed lookup hash: Why: "No log line, event partition, metric tag, or foreign key should use email as identity."

27. Account sessions with device management and coarse region only if needed: Why: session revocation and security review are supported while "high-resolution location" is not stored by default.

28. Aviary version, simulation cursor, tick timing, day phase, weather, and settled state fields: Why: these support canonical snapshots, ordered event processing, server-owned weather/day phase, and tick scheduling.

29. Bird stable identity, call seed, visual seed, perch, target perch, transition timing, and pose fields: Why: the client can render identity, procedural calls, visual continuity, and canonical transitions without owning personality or mood.

30. Bird personality vector table: Why: traits are normalized internal floats, clamped to calibrated ranges, and "only the simulation service may update this table."

31. Bird mood state table: Why: "Mood is persisted and not reset on page open."

32. Interaction events: Why: presence, listen-in, offer, settle, rename, and accessibility changes are private simulation inputs with client idempotency keys, not analytics exports.

33. Presence windows: Why: the simulation service consolidates pings into bounded windows so valid attention can drive drift while stale tabs cannot create runaway presence.

34. Offer cooldowns: Why: per-bird cooldowns keep offers "gesture-like" and avoid repeated offers saturating traits in one session.

35. Notebook entries: Why: entries are "read-only and sparse," naturalist prose rather than event logs, and stored as final prose so the record remains stable.

36. Visit invitations and visit sessions: Why: visits are revocable and expiring, and "do not generate host presence events and cannot write interaction events."

37. Telemetry events as allowed aggregate fields, not a general behavioral table: Why: operational telemetry should go to metrics directly with privacy-safe fields only.

38. privacy_policy_version_acknowledged: NOT RECOVERABLE FROM PLAN

39. name_updated_at: NOT RECOVERABLE FROM PLAN

40. ordinal_in_aviary: NOT RECOVERABLE FROM PLAN

41. last_visit_started_at: NOT RECOVERABLE FROM PLAN

42. approximate_duration_seconds: NOT RECOVERABLE FROM PLAN

### 4. API surface

43. Matter-of-fact error messages and limited product-facing prose: Why: account, settings, and error surfaces should stay direct, while prose belongs only to aviary observations, notebook entries, captions, and narration.

44. Magic-link request and consume endpoints: Why: links are single-use, expire after 15 minutes, are rate-limited by email hash and IP bucket, and the request response is non-enumerating.

45. Account and starter aviary creation on first consume: Why: a new email immediately gets the canonical account and starter aviary needed for the first usable experience.

46. Account GET/PATCH surfaces: Why: account settings, sessions, exports, deletion status, accessibility settings, visit notifications, timezone, and device labels are kept in matter-of-fact account surfaces rather than inside the aviary scene.

47. Email change verification: Why: the plan says to "Commit only after new address verifies."

48. Export, soft delete, and recover endpoints: Why: they queue export delivery, mark soft deletion, and allow restoration within the 30-day window before hard deletion.

49. Host snapshot and bootstrap endpoints: Why: the snapshot provides canonical state without numeric personality values, and bootstrap carries the smallest state needed to draw the first bird within 500ms.

50. Notebook and narration endpoints: Why: they provide read-only observations and slow-cadence naturalist narration without event-log language or user-behavior summaries.

51. Interaction ingestion endpoint: Why: it validates ownership, timestamps, cooldown eligibility, and bird membership, returns accepted/rejected statuses, and "does not update personality directly."

52. Presence event semantics: Why: pings are accepted only when the client asserts visible, focused, and recent activity, and the server closes windows after heartbeat thresholds.

53. Bird renaming: NOT RECOVERABLE FROM PLAN

54. Rename-only bird API restriction: Why: the endpoint must not allow species, personality, perch, or mood updates, preserving the no-direct-state-write boundary.

55. Age-paced adoption offer and adopt endpoints: Why: the plan avoids "count-progress copy," "earn" language, and catalog browsing; the server selects species and the data model supports adoption from day one.

56. Visit invitation and visitor snapshot APIs: Why: visitors get the same rendering snapshot with visitor permissions, no notebook mutation, no listen-in, no offer, no settle, and no event submission bundle.

### 5. Simulation engine design

57. Tick scheduling with per-aviary lease/lock, event cursor, and atomic update: Why: duplicate scheduling is tolerated, failed mid-transaction ticks commit no partial state, and retries are idempotent.

58. Presence consolidation: Why: presence-time is the "dominant drift input" but must come only from visible, focused, recently active host sessions and must not become a displayed counter.

59. Slow personality drift: Why: drift should be measurable after about a week, user-visible after about three weeks, never jump from one session, and "never apply negative deltas for neglect."

60. Drift formula versioning: Why: formula changes can migrate without resetting birds, and production personality state is updated forward rather than recomputed from raw event history.

61. Mood transitions: Why: mood should persist, be probabilistic but bounded, and translate time, weather, events, personality, and bird-to-bird influence into pose, perch, call, and offer behavior.

62. Return-greeting generation: Why: greetings are based on current state plus absence length, with one primary greeter, staggered responses, and no welcome announcement text.

63. Offer reactions: Why: offers are validated by the server, reactions come from mood and personality, fast mood can change, slow drift remains small, and cooldowns prevent saturation.

64. Settle flow: Why: settle quiets lighting and calls, records the transition, ends presence, supports undo, and makes closing without settle penalty-free.

65. Weather and day/night: Why: weather that affects mood must be canonical, rain is per-aviary rather than global, timezone stays current without being noisy, and night is "quiet, not dead."

66. Notebook generation: Why: entries come from meaningful state changes or unusual combinations, pass sparsity gates, use controlled templates, and are stored as stable prose for tone, privacy, localization, and repeatability.

### 6. Sync model

67. Server canonical state plus append-only events with no client-to-client merge: Why: the plan prevents conflicts by authority boundaries.

68. Client snapshot pulling on load, visibility return, frame gaps, and sleep resume: Why: stale or suspended clients refresh to the canonical server state while optional SSE/WebSocket is limited by "complexity and battery budgets."

69. Optimistic rendering only for ephemeral UI/audio transitions: Why: the server can confirm or correct them later "without changing personality."

70. Versioned snapshots, server time, idempotent event ingestion, and received-order cursor processing: Why: duplicate events are not double-applied and stale concurrent sessions do not overwrite personality history.

### 7. Frontend rendering pipeline

71. Route-level code splitting and 2MB gzipped bundle gate: Why: the aviary route stays minimal and account/settings/notebook/visit management surfaces are loaded away from first render.

72. Thin Canvas 2D or WebGL renderer after prototype measurement, avoiding a heavy game engine: Why: renderer choice depends on asset complexity and mobile performance.

73. Quiet field loading state, no spinner, and never showing an empty aviary as a normal return state: Why: the first frame should feel already alive and preserve the continuity of an aviary that has appeared before.

74. Motion state machine with canonical state and client-only micro-variation: Why: bird identity, mood, perch, weather, settled state, and day/night stay canonical, while breathing, feather shifts, and tiny weight changes add local life.

75. Minimal top bar, no UI chrome inside the scene, no labels/status icons/badges/tooltips over birds: Why: the scene stays quiet and focused on birds rather than interface pressure.

76. Offer UI with seed, song fragment, and still pool in naturalist voice: Why: offers should not imply feeding or caretaking, and cooldown copy should not feel punitive.

77. Notebook UI without a persistent badge unless needed inside the opened notebook: Why: the plan wants to avoid "notification pressure."

78. Keyboard model and visible focus outlines: Why: top bar, scene bird focus, listen-in, offer, settle, and notebook must be fully keyboard navigable across bright, evening, and night palettes.

79. Reduced-motion renderer: Why: it uses the same snapshot state but different presentation, keeping calls, captions, notebook, mood, drift, and interactions while replacing motion loops and flights with cross-fades.

### 8. Audio pipeline

80. Procedural WebAudio calls with no recorded audio fallback: Why: the product relies on procedural calls and, when WebAudio is unavailable, chooses silent audio plus captions by default rather than a recorded fallback.

81. Species call grammar and per-bird call signature seeds: Why: calls remain recognizable and varied while the server can transmit derived call parameters without exposing raw personality values.

82. Audio mixing, listen-in ramps, and settle ramps: Why: birds have spatial placement, chorus avoids loop artifacts, focus changes use gradual ramps, other birds never cut to silence, and settle quiets gently.

83. Captions generated from call parameters: Why: captions match what the procedural grammar produced, such as "a soft three-note rise," and provide the fallback when audio is unavailable.

84. Audio performance bounds: Why: reused nodes, bounded simultaneous calls, hidden-tab suspension, and a 30-minute memory-growth test prevent accumulating audio allocations while simulation continues.

### 9. Accessibility surfaces

85. Screen-reader narration: Why: a slow coalesced narration channel gives naturalist observations without flooding, raw mood labels, personality numbers, perch numbers, or event-log phrasing.

86. Semantic model: Why: controls and errors use matter-of-fact names, while bird focus targets use naturalist labels or names without numeric stats.

87. Caption placement and contrast: Why: captions should sit near the calling bird without obscuring the scene, respect reduced motion, and meet WCAG AA contrast.

88. Visual accessibility and touch targets: Why: offer state, selected listen-in state, and errors cannot rely on color alone, and phone viewports need touch-sized targets.

89. Accessibility testing: Why: axe checks, keyboard tests, screen-reader QA, and reduced-motion visual regression ensure the accessible modes remain a designed aviary.

### 10. Privacy, security, and data boundaries

90. Privacy rules for email, per-bird events, analytics, model training, and visitors: Why: per-bird interaction events are only for that user's simulation, analytics cannot read simulation data, and visitors never affect host drift.

91. Security controls for magic links, token hashes, rate limits, CSRF, sessions, visits, exports, and deletion: Why: tokens remain short-lived and hashed, writes are protected, sessions and visits are revocable, export links go only to the verified address, and soft-deleted accounts stop normal snapshots and ticks.

92. Logging redaction: Why: logs should not contain email, bird names, notebook prose, interaction payloads, or raw snapshots by default.

### 11. Performance budgets and observability

93. Performance budgets: Why: the product depends on first bird under 500ms, 60fps idle motion, no 30-minute memory growth, and simulation tick p99 under the alarm threshold.

94. Implementation tactics for performance: Why: code splitting, bootstrap snapshots, first-bird-before-effects rendering, compact assets, bounded renderer state, hidden-tab pause, and cautious service worker use protect speed without stale-state complexity.

95. Observability and CI gates: Why: synthetic checks, aggregate RUM, server metrics, privacy-safe dashboards, bundle gates, performance smoke tests, memory tests, deterministic simulation tests, and accessibility tests catch regressions without account/bird/event/notebook dimensions.

### 12. Rollout plan, test strategy, risks, and guardrails

96. Milestone 0 technical spikes: Why: renderer, audio, simulation, presence, bundle, and first-bird timing are proven before deeper product build.

97. Milestone 1 account and canonical aviary foundation: Why: the first foundation exit is that a user can sign in, receive two starter birds, open on two devices, and see the same canonical state.

98. Milestone 2 bird engine v1: Why: no client writes personality, one week of fixture presence produces measurable drift, one session does not visibly jump traits, and closing without settle is neutral.

99. Milestone 3 rendering and audio quality: Why: first frame appears already in motion, there is no spinner, 60fps passes, and WebAudio-unavailable sessions get captions by default.

100. Milestone 4 notebook and accessibility: Why: screen-reader and reduced-motion users get a designed aviary, and notebook entries remain naturalist observations rather than event logs.

101. Milestone 5 visits, export, deletion, and privacy hardening: Why: visitors cannot write events or generate presence, revoked visits end at next snapshot, and analytics cannot read per-bird simulation data.

102. Milestone 6 private beta, calibration, bird ramp, and launch gate: Why: the rollout starts with two birds, tunes drift and interaction calibration through fixtures and consenting internal test accounts, and only expands bird counts when quality and recognizability hold.

103. Copy and voice tests: Why: product surfaces must use naturalist voice, system/error/settings surfaces must be matter-of-fact, and no welcome toast, streak, achievement, public discovery, profile, stats, or raw personality copy should appear.

104. Implementation guardrails: Why: the plan treats numbers, client canonical writes, welcome copy, per-bird/per-account metrics, flattened accessible modes, visit state changes, absence punishment, and indistinguishable calls as violations of the product.
