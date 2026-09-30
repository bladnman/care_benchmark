## System-level intent

1. Server-owned canonical continuity. The plan opens with "continuity belongs to the birds, not to an open tab" and repeats that "the server is the only authority" for birds, personality, mood, weather, schedules, and notebook. This shows up again in the module boundary table, the single-writer simulation worker, the snapshot projector, row locks, canonical revisions, and the requirement that "every device reads one canonical record."

2. Quiet, non-gamified companionship. The exclusion list removes hunger, death, distress from absence, achievements, streaks, scores, badges, leaderboards, visit-frequency surfaces, and engagement notifications. Later sections reinforce "ordinary ambient life and no guilt cues," "no countdown, failure punishment, or scarcity rewards," "no welcome or gamification copy," and "the observer describes birds, not user effort, attendance, or compliance."

3. Watching matters, but unattended tabs do not. Product acceptance says "watching without clicking has a real effect over weeks, while unattended/background tabs do not." The monotonic drift function makes watching "sufficient and dominant," while presence correctness insists on visible, focused, recently active, not settled intervals and says it "must never credit unattended hours."

4. Bird identity is durable. The plan repeatedly protects stable bird UUIDs, immutable signature seeds, persisted vectors, and name-at-observation prose. It says identity and signature seeds survive "renames, migrations, software rollback, timezone changes, and offer responses," and rollout/rollback must "preserve all existing birds, vectors, signatures, notebook entries, and ownership."

5. Absence must never punish the birds or the owner. The acceptance criteria require return after absence with "ordinary ambient life and no guilt cues." The drift section says "no negative term exists," traits never decay, familiarity cannot "cause distress" or "make the bird progressively mistrust the user," and notebook prose must not create "neglect summaries or user-frequency judgments."

6. Accessibility parity is part of the core aviary, not an add-on. The delivery contract says "accessibility and performance are launch requirements," and audio-off, screen-reader, and reduced-motion users receive "the same specific, living aviary." This appears in shared semantic observations, narration, call captions, reduced-motion design, keyboard/focus behavior, contrast, zoom, and release gates.

7. Product voice separates naturalist prose from system facts. The notebook and accessibility sections call for "lowercase, present-tense, bird-specific prose," while account, auth, errors, sync, privacy, deletion, and accessibility settings use "ordinary capitalization" and "direct matter-of-fact messages." The plan explicitly says not to "disguise an outage as weather or an expired sign-in as a bird behavior."

8. Privacy minimization outranks engagement analytics. The plan forbids private interaction analytics, cross-account bird statistics, request-body logging, behavioral tracking, retention dashboards, and production private-history aggregation. It routes telemetry to "allowlisted timings, counts, error classes" and says private bird events are "for that user's simulation and nothing else."

9. Procedural, deterministic expressiveness replaces canned media. Return-greetings must not "replay fixed greeting clips," calls use symbolic motif grammars and deterministic descriptors, chorus windows "emerge" rather than coming from a precomposed track, and there is "no recorded-audio fallback." This supports recognizability without turning birds into static clips.

10. Aliveness is an affective performance requirement. The first normal-session frame must already be "mid-action," a bird must notice the owner within "one to two seconds," and the first-bird path must avoid spinners, placeholders, entry fades, and handoff jumps. The completion statement says a checklist is insufficient because release review must include "the actual first return" and "a few minutes of uneventful watching."

11. Social behavior is narrow, deliberate, and private. Invitations are "individually deliberate, revocable, and initially nonexistent." Visitors see identical canonical birds, lighting, weather, calls, and quality, but no greeting event, interaction endpoints, presence tracker, listen-in, offers, settle, private notebook, or co-presence overlay. Visit-email is an off-by-default, narrow exception.

12. Ambiguity is resolved through explicit, narrow exceptions and decision gates. The implementation decisions record conflicts around settle placement, Enter/focus behavior, export vectors, visit notifications, and audio autoplay. Rollout then uses health/quality gates, server-controlled feature flags, continuity checks, and rollback rules that preserve birds rather than reducing bird count or reseeding.

## Per-feature whys

1. Browser-only aviary: NOT RECOVERABLE FROM PLAN

2. One aviary per account: NOT RECOVERABLE FROM PLAN

3. Server authority for birds, personality, mood, weather, schedules, and notebook: the rationale is canonical continuity across devices and protection from client-generated persistent personality updates. The plan states that every device reads "one canonical record" and "no action can overwrite another device's personality drift."

4. Email magic-link accounts: the plan frames these through security and simplicity rather than passwords/SSO. Links use generic 202 responses to avoid address enumeration, single-use 15-minute tokens, explicit landing-page confirmation, and GET-safe handling so mail scanners do not consume sign-ins.

5. Account IANA timezone: the rationale is stable civil-time context for all devices and visitors. The plan says travel should not silently let devices "alternately redefine morning"; only a matter-of-fact account setting changes it.

6. Two server-selected starter birds: the articulated why is to start without species catalog browsing, rarity, or a selection economy. Adoption is idempotent, server-selected, and enters the scene after naming rather than presenting a catalog.

7. Coherent six-species pool: NOT RECOVERABLE FROM PLAN

8. Renameable birds with permanent identities: the rationale is continuity. Display names can change, but stable UUIDs, signature seeds, vectors, and notebook references survive renames, migrations, rollback, timezone changes, and offer responses; older notes keep observation-time names so they remain authentic.

9. Age-based optional adoption up to seven birds: the why is growth without attention rewards or achievement framing. Eligibility is age-based, persists without expiry, is "not activity-derived," has no progress meter or countdown, and is not earned by attention.

10. No bird reset/removal/regeneration routes in v1: the rationale is durable identity. The plan says stable identity and signature seeds survive major changes and that reset/removal/regeneration would undermine that continuity.

11. Five mood states: the reason given is to avoid turning sleeping or settled into extra needs or health states. "Sleeping/settled are poses and lighting states, not extra numerical needs or health states."

12. Top-bar settle action in the offer/actions popover: the rationale is to reconcile the four-primary-icon layout with the requirement for a top-bar settle control. It remains a top-bar action "without an extra scene control" and is keyboard reachable.

13. Focus engages listen-in, Enter ensures engagement, Escape disengages: the plan's why is accessibility consistency. This resolves the Enter wording "without making keyboard navigation inconsistent" and makes moving focus to another bird start that bird's listen-in.

14. Account-data-export exception for current personality vectors: the rationale is portability under an explicit export schema conflict. Vectors appear only in downloaded machine-readable JSON; no product surface, normal snapshot, ARIA label, debug toggle, or rendered export preview displays them.

15. Off-by-default visit-email opt-in: the rationale is a narrow exception to the no-notification rule. It is limited to account settings, initially off, with no onboarding prompts, badges, push, or duplicate visit mail.

16. Autoplay-safe procedural audio start: the why is browser compliance without sacrificing the living visual scene. The plan says to never circumvent autoplay restrictions; captions and visuals start immediately, then audio unlocks after explicit user gesture.

17. Typed architecture boundaries: the rationale is to keep each module from violating canonical ownership. The table says identity must not mutate vectors, the interaction gateway must not accept client moods, telemetry must not query simulation tables, and renderer code must not evolve birds.

18. Versioned contracts for snapshots, events, grammar descriptors, and observations: the why is consistency across server-rendered SVG, browser rendering, audio, captions, and simulation migrations. Shared geometry and grammar interpretation prevent mismatched first render and runtime behavior.

19. Packaging persistent simulation logic separately from renderer code: the articulated rationale is preventing accidental client bundling of personality evolution. The client may interpolate and present state, but cannot compute persistent drift.

20. Canvas2D scene renderer with inline SVG first birds: Canvas avoids "an expensive changing DOM tree," while inline SVG supplies first visible birds before the full renderer is ready. DOM buttons aligned to bird hit regions provide accessibility.

21. Scene and audio adapters: the rationale is that missing audio must never block rendering. The aviary starts visually and through captions even when WebAudio is unavailable.

22. Monotonically increasing aviary revision with transactional outbox: the why is committed state consistency. The database remains authority if edge cache or outbox is delayed, and publication can reject revision regressions.

23. Expedited server tick for user events: the rationale is prompt greetings, offers, and settle cues without a second mutation path. It uses the same simulation transaction and sole vector writer, coalesced and limited to avoid extra drift.

24. UUID-only internal references and encrypted email storage: the why is privacy and deletion isolation. Internal queues, storage keys, logs, invites, and sessions reference UUIDs, while email is restricted to identity/contact storage.

25. IdentityContact for invitees without accounts: the rationale is delivering mail and showing the host who was invited without creating unsolicited aviary accounts or putting email on invitation/event/log rows.

26. Stable Bird record with vector, filter state, mood, perch, pose, call sequence, and cooldown: the why is that personality, mood, position, and interaction timing are canonical server state rather than client reconstruction.

27. Database range checks and seven-bird cap locks: the rationale is preserving valid trait bounds and preventing concurrent adoption from exceeding the seven-bird cap.

28. Bounded event retention with compaction into rolling inputs and observation memory: the plan's why is to retain only what continuing simulation needs. Persisted vectors are never reconstructed from event retention or notebook history.

29. Same-origin HTTPS JSON endpoints with secure cookies, Origin, and CSRF checks: the rationale is owner authentication, revocable sessions, and mutation safety. Visitor cookies are distinct and cannot authorize owner endpoints.

30. Name validation and rendering names as text: the why is safe, inclusive naming. Names are bounded to 40 grapheme clusters, permit Unicode, trim unusable whitespace, and are rendered as text rather than HTML.

31. Adoption start/complete idempotency: the rationale is avoiding duplicate starter birds on retry. The first encounter can be committed once without additional starters.

32. Rename conflict handling with expected revision: the why is cross-device correctness. A stale concurrent rename returns 409 with the latest value rather than overwriting another device.

33. Snapshot endpoint contents and omissions: the rationale is a bounded render description that exposes living scene state without private internals. Snapshots omit vectors, filter states, presence totals, raw interactions, and numeric mood intensities.

34. Event endpoint accepting intents, not outcomes: the why is server authority. Cooldown, receiving bird, acceptance, and reactions are server decisions; client optimistic UI cannot claim acceptance or persist mood.

35. Owner arrival endpoint: the rationale is return-greeting separate from presence credit. Greeting can happen on navigation, visible return, or long-frame recovery without requiring recent pointer activity.

36. Read-only notebook endpoint: the why is preserving sparse observations as immutable field notes, not user-authored logs. There is no write API, and oldest entries remain addressable.

37. One-time visit links with 24-hour visitor sessions and 30-day unused expiry: the rationale is bounded, revocable visitor access. Revocation is checked before every response, including 304, and service workers must not serve authorized visits offline after lease expiry.

38. Stable plain error handling in system panels: the reason is product-voice boundary and recoverability. Errors are not naturalist prose or celebration UI; a failed optional interaction leaves birds present with a retry surface.

39. Minute tick regardless of connections: the why is that continuity belongs to birds and "ordinary unattended aviaries still advance every minute." It prevents lazy-on-open simulation from replacing ongoing ambient life.

40. Row locks and ordered committed event cursor: the rationale is exactly-once simulation effect and cross-device ordering. A database-generated sequence alone is not assumed to guarantee commit order.

41. Worker crash and outbox retry behavior: the why is observability of committed state only once. A crash before commit is unobservable; a crash after commit cannot reapply consumed events, and retries publish the same revision.

42. Scheduler catch-up from persisted state: the rationale is resilience, not open-tab laziness. The plan says never reseed or rebuild vectors, process bounded chronological chunks, and retain overdue priority until current.

43. Monotonic drift function: the why is slow, non-punitive personality growth. Watching is dominant, repeated offers cannot dominate, no negative term exists, muted users are not penalized, and a single session must not create observable trait change.

44. Synthetic calibration of drift constants: the rationale is tuning without private interaction analytics. The plan specifies seeded synthetic accounts and expected week/three-week numerical changes.

45. Recent owner familiarity as transient only: the why is expressive greetings without trait punishment. It may affect greeting frequency and session expressiveness, but cannot lower traits, cause distress, reduce plumage, or make birds mistrust the user.

46. Bounded probabilistic mood state machine: the rationale is ambient life with personality-shaped behavior, not needs or page-load reset. Old session influence dissipates over local-day cycles without forcing all birds neutral at midnight.

47. Weighted perch selection with reserved slots: the why is personality expression and visual correctness. Boldness, wariness, warmth, and drowsiness shape zones while atomic reservations prevent clipping and users cannot send birds to perches.

48. Seeded rain and soft wind: the rationale is ambient behavior without real-weather dependency or urgency. Rain briefly attenuates call activity, wind shifts mood slightly, and there is no thunder or urgent alert framing.

49. Bird-to-bird call responses and chorus windows: the why is emergent social sound. Replies use warmth/vocal weighting, chorus emerges from overlapping calls, and alarm propagation is capped so the aviary cannot escalate indefinitely.

50. Return-greeting primary response: the rationale is prompt recognition without banners, guilt, or canned clips. Exactly one primary bird notices within one to two seconds, parameterized by identity, mood, history, and absence, with accessibility narration describing the noticing.

51. Staggered non-primary return responses: the why is naturalism without synchronized fanfare. Other birds may respond later at randomized offsets, not as an arrival chorus.

52. Three offers from the top bar: the rationale is small aviary-level interaction, not feeding a bird by clicking it. Seed, still pool, and synthesized song fragment prompt server-authored approach, inspection, drinking, bathing, watching, joining, quieting, or counter-calling.

53. Offer cooldown and request-time reservation: the why is preventing offer mashing from becoming boldness input or bypassing cooldown on concurrent devices. Disabled copy remains calm, with no countdown, punishment, or scarcity reward.

54. Listen-in targeted attention: the rationale is focused attention within eligible presence only. It changes the local mix and supplies capped targeted input without allowing two simultaneous devices to double drift.

55. Settle and undo: the why is deliberate quieting and presence end without a ritual bonus. Settle ends that device's presence, ramps a quiet overlay, gives a shared small mood cue, ignores passive pointer motion, and undo resumes only prospectively.

56. Presence eligibility predicate: the rationale is honest watching without invasive tracking. Visibility, focus, recent pointer/key activity, and not settled are required; clicks, load, audio playback, and open tabs alone do not count.

57. Fifteen-second heartbeat segments with lease validation: the why is preventing unattended-hour credit. The plan accepts losing a few seconds during disconnects but says it must never backfill unverifiable attention.

58. Union presence across devices: the rationale is canonical account-time accounting. Presence counts once per interval, listen-in is a subset, and offer spam, duplicates, visits, background audio, and multiple tabs cannot increase counted presence.

59. Snapshot pull and resume rules: the why is synchronization without client simulation. Clients discard expired local schedules, ignore stale revisions, interpolate only within issued horizons, and never invent greetings, weather, offers, or personality changes.

60. Small in-memory pending event queue: the rationale is retry safety without a long offline action queue. Same UUID retries are accepted as duplicates, and stale behavioral actions do not overwrite state.

61. Authenticated private snapshot caching: the why is first-frame speed without account leakage. Cache keys include aviary, role, schema, and revision; revocation checks happen before cached projections.

62. Deterministic naturalist language module: the rationale is factual, private, bird-specific prose without external language-model dependency. Claims like "first time this week" require compact observation memory support.

63. Sparse notebook cadence: the why is meaningful observation rather than a feed. Ordinary entries are roughly one per three active days, with gaps and caps; inactive aviaries avoid neglect summaries and user-frequency judgments.

64. Notebook prose style: the rationale is to describe birds, not effort. It avoids generic messages and numeric trait narration, uses lowercase present-tense named observations, and distinguishes quietness from absence judgment.

65. Lazy-loaded, virtualized notebook panel: the why is performance and read-only browsing. Offscreen DOM/listeners are removed, page cache is bounded, and older pages can be re-fetched indefinitely.

66. Critical first-render path with inline snapshot/SVG/bootstrap: the rationale is showing real birds before hydration, settings chunks, audio context, fonts, or notebook queries. Birds are drawn at scheduled phase, already mid-action.

67. SVG-to-Canvas handoff without fade or jump: the why is continuity of identity and motion. Identical geometry, pose, phase, and seeds are preserved across the handoff.

68. First-adoption empty-scene exception: the rationale is only first encounter onboarding. After naming, starters enter with a gentle fly-in or reduced-motion cross-fade, and completion is recorded so ordinary visits do not restart empty onboarding.

69. Restrained scene composition without scene chrome: the why is keeping birds visible, recognizable, and non-gamified. The plan excludes scene buttons, labels, badges, mood icons, tooltips, and drag handles, except captions and focus outlines.

70. Responsive normalized scene geometry: the rationale is preserving all birds across devices. Portrait compresses or letterboxes instead of cropping or scrolling the scene, and seven birds must be valid at 320px width.

71. Server-computed personality expression and mood-shaped idle poses: the why is persistent personality without client evolution. The server computes expressive parameters; the client uses bounded phase curves and ornaments only for presentation.

72. Top-bar controls that fade only when safe: the rationale is a quiet scene without inaccessible controls. Controls remain visible during focus, popovers, modals, keyboard use, and touch use; no badges signal notebook, eligibility, or visits.

73. Reduced-motion renderer: the rationale is accessibility parity without changing simulation state. It swaps motion for slow pose/perch fades, removes moving ornaments/parallax, avoids flashes, and keeps calls, captions, moods, drift, notebook, and outcomes full quality.

74. Symbolic motif grammar per species and signature seed per bird: the why is recognizable procedural audio. Register and timbral anchors stay stable enough to identify the same bird across moods, weeks, and same-species pairs.

75. Server-selected call descriptors: the rationale is deterministic canonical behavior with client synthesis only. Descriptors carry call IDs, grammar versions, seeds, note/envelope/semantic parameters, and captions derive from the exact descriptor.

76. One AudioContext with bounded voices, buses, limiter, and cleanup: the why is comfortable, leak-free audio. The plan caps simultaneous voices, stops oscillators, bounds buffers/graphs, and drops quiet ornament voices rather than distorting foreground calls.

77. Audio scheduling by server-time mapping and resume skipping: the rationale is temporal continuity. Elapsed calls are skipped rather than rushed, and mid-call snapshots never blast a call from the start to catch up.

78. Listen-in mix envelope: the why is prominence without isolation or volume spike. The target ramps up gently, other birds remain audible at reduced ambient gain, and switching/disengagement crossfades.

79. Captions near birds: the rationale is audio parity and descriptor fidelity. Captions are contrast-safe, avoid covering heads, allow a few simultaneous captions or combined chorus descriptions, and are automatic when audio cannot run.

80. WebAudio failure fallback: the why is graceful silence rather than fake sound. Captions remain enabled, settings explain matter-of-factly, user gesture can retry, and no recorded loop pretends to be procedural.

81. Shared semantic observation stream: the rationale is agreement between visual cues, captions, narration, and prose. Return-greetings, offers, settle, idle narration, and current observations all draw from the same source.

82. Screen-reader narration cadence and queue limits: the why is useful presence without speech overload. Idle updates occur about every 45 seconds, priority observations supersede stale prose, and settings/notebook can pause ambient narration.

83. Roving-tabindex bird controls with decorative Canvas: the rationale is accessible interaction without duplicate representations. Bird controls carry name/species and observational phrases, focus engages listen-in, and Canvas stays decorative to assistive technology.

84. Restricted offer and settle keyboard shortcuts: the why is keyboard efficiency without interfering with text inputs or assistive contexts. Visitor mode omits owner shortcuts and listen-in controls.

85. Touch target, contrast, focus, zoom, and forced-colors requirements: the rationale is making the aviary usable in all light states and accessibility modes. Captions, controls, and focus treatments have contrast gates.

86. Account lifecycle creation on first verified sign-in: the why is transactional identity/aviary setup. Adoption reservations are idempotent, lookup uses keyed digest only within identity, and other references use UUIDs.

87. Session listing, revocation, token rotation, and 30-day cap: the rationale is control over devices and compromised sessions. Revoking a current device also clears its cookie and sessions contain no bird data.

88. Export job generation and expiring emailed download: the why is account portability with privacy boundaries. It reads one committed revision, includes current vectors only under the exception, excludes raw logs/secrets/visitor emails, encrypts output, and expires the one-time token.

89. Deletion with 30-day recovery and hard purge: the rationale is immediate access removal plus reversible mistake window, followed by irreversible data removal. Recovery retains bird IDs/vectors and does not invent absence presence or reissue old visits.

90. Backup cryptographic erasure and tombstones: the why is preventing purged accounts from returning through restore. Backup retention must support the hard-deletion promise.

91. Telemetry isolation from simulation data: the rationale is private data remaining private. Metrics cover health/performance, not bird IDs, names, species, moods, traits, presence seconds, offer/listen behavior, recipients, emails, or interaction histories.

92. Individual visit invitations: the why is opt-in, private social access without discovery or co-presence. Invitations are deliberate, revocable, absent by default, and visitor authorization blocks forged owner API requests.

93. Private visit log with approximate duration: the rationale is host transparency, not personality input or engagement analytics. Duration uses snapshot pulls, excludes long gaps, and stays in account settings only.

94. Performance budgets for first bird, greeting, idle 60fps, memory, tick p99, and browsers: the why is measured aliveness and launch quality. The plan says the 2MB JS limit is a ceiling, first-bird measurement uses real pixels, and supported browsers without audio still use captions.

95. Lazy-loading noncritical panels and small snapshots: the rationale is the sub-500ms affective path. Initial birds depend on inline authenticated state and minimal critical bytes, not loading a near-2MB framework.

96. Pausing hidden work and bounded per-tab resources: the why is resource control. Hidden tabs stop animation, ornaments, audio scheduling, and presence; logout/visit termination disposes contexts and listeners.

97. Synthetic probes and fixture accounts: the rationale is calibration and health monitoring without production private-history analysis. Dashboards show health/performance, not engagement or mean personality drift.

98. Milestone A aliveness prototype: the why is testing affect early so infrastructure does not lock in a "robotic renderer." Exit criteria require first frame moving, no handoff jump, grammar/caption agreement, and screen-reader focus validation.

99. Milestone B persistence, tick, and honest presence: the rationale is proving one committed vector across concurrent devices and no invented input. Exit requires server ticking while clients are absent and no reset-from-log recovery.

100. Milestone C full owner experience and accounts: the why is a complete quiet session across mouse, touch, keyboard, and no-audio paths, with no welcome or gamification copy and consistent cross-device state.

101. Milestone D visits and launch hardening: the rationale is proving visitors cannot produce simulation input and that default visits remain silent to the host while reduced-motion, captioned, and narrated surfaces are release-complete.

102. Required verification matrix: the why is to test properties, not only implementation checkboxes. It covers simulation monotonicity, presence truth tables, transactions, continuity, audio, first frame, accessibility, performance, and privacy/security.

103. Staged rollout by health and quality gates: the rationale is avoiding engagement-conversion gating and preserving quality. Social is default-deny until acceptance passes; public v1 includes visits and all accessibility surfaces.

104. Gradual adoption ceiling rollout with accelerated fixtures: the why is validating seven-bird audio, layout, memory, and concurrency before public eligibility matures. The plan says not to auto-add birds, charge for them, expose count rewards, or tie eligibility to visits.

105. Operational rollback by disabling new invites/adoptions or faulty grammar versions: the rationale is preserving continuity. Rollback must never reduce existing bird count or reseed a bird; audio failure falls back to captions rather than making the aviary unusable.

106. Risk gates for drift, absence, retries, vector loss, presence, cue latency, calls, captions, render, caching, privacy, tick cost, and scope creep: the why is that known failures threaten the core intent. The mitigations map directly to synthetic cohorts, no negative drift, row locks, backups, truth tables, expedited passes, recognition gates, semantic queues, inline snapshots, auth cache keys, canaries, scalable tick queues, and documented decisions.
