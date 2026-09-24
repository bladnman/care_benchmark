## System-level intent

1. **Canonical simulation belongs on the server.** This shows up first in the invariants: "Server is the only writer of personality, mood, perch, and weather," "Clients write only interaction events," and later as "canonical simulation state (server) versus presentation realization (client)." The same intent appears in sync as "exactly one writer of simulation state" and "clients never hold authoritative state."

2. **Drift should be slow, additive, monotonic, and not punitive.** The plan repeats "Drift is monotonic non-decreasing, additive, and server-authored," "nothing goes down," and "a single session can never visibly move a trait." The separate "attunement" mechanism exists so birds may be "quieter" after absence without decreasing personality, wariness, or ease.

3. **Bird identity is durable.** The invariant "Stable bird identity" says `bird_id` is "minted once and never reissued, replaced, or regenerated." The same principle reappears in species definitions being "versioned and kept forever," visiting birds carrying the "same bird" into adoption, and migrations preserving every `bird_id`.

4. **Privacy is structural, not a policy afterthought.** The plan says "Email is PII and lives in exactly one place," stores only synthetic `account_id` references elsewhere, sends no raw pointer data, keeps "aggregate-only operational telemetry," excludes third-party analytics and session replay, and gives the analytics plane "no network route or credentials to the simulation database."

5. **The product must avoid announcement and gamification creep.** The invariant forbids "No announcement, gamification, or user-behavior surfaces," with "no toasts, no streaks, no counts, no 'welcome back.'" This shows up in top bar badges never existing, no cooldown messages, no reminders for visiting birds, no visit counts, and notebook entries that "observe the aviary, never the user."

6. **Voice is split between naturalist surfaces and matter-of-fact system surfaces.** Notebook, narration, captions, and onboarding share "lowercase, naturalist" grammar with no "you," digits, or exclamation marks. Errors, auth, deletion, unsupported browser, and revoked visits use "matter-of-fact" copy and sentence case.

7. **The aviary should feel ambient and alive, not demanding.** The plan uses "ambient, not punished," "not a Tamagotchi meter," "no failure state," "no visible decay," "no hunger or death or distress," and "quiet" flows. New birds "notice rather than announce," settle is a "per-session goodbye," and a missed snapshot shows a "quiet field" rather than a spinner.

8. **Accessibility is a first-class aesthetic and parity goal.** Accessibility is "all at launch," and reduced motion is a "designed surface" with its own pose library. Captions are generated from the "actual call grammar," narration uses the same snapshot as the renderer, and the manual pass criterion includes whether the aviary "feels alive" in each modality.

9. **Performance is part of the product feel.** The plan centers "first bird visible in <500 ms," "continuous from the first frame," "no entry animation exists anywhere," "no spinner, no text," and "no memory growth over 30 minutes." The internal JS budget is far below the PRD cap because "the headroom is deliberate."

10. **Determinism and calibration are load-bearing.** The engine is "pure and deterministic," with no `Date.now` or `Math.random`, versioned config, golden replays, persona calibration bands, and a physically separate `calibration` deployment. Ambiguous constants are "initial" and retuned with harnesses rather than production engagement data.

11. **Multi-device behavior should converge through snapshots and append-only events.** The plan avoids "client-to-client sync" and "merge logic," uses ETags and precomputed snapshots, makes event deltas additive, and unions presence intervals so simultaneous devices count once.

12. **Rollout is evidence-gated and forward-only.** The build sequence puts "engine, calibration harness, and privacy boundaries" first because they are "hardest to retrofit." Bird-count ramps depend on human recognizability panels, engine versions "apply forward only," and no kill switch may touch traits.

## Per-feature whys

1. **Single-user accounts with email magic-link sign-in, sessions, email change, export, and deletion.** Why: the plan frames account identity around synthetic `account_id`, encrypted email, matter-of-fact system emails, revocable sessions, data export, and deletion so account operations do not become aviary-state or engagement surfaces.

2. **Per-device revocable sessions.** Why: revocation is written to Redis immediately and checked at edge and origin, so a revoked session stops quickly across the system.

3. **Old-email behavior during email change**: NOT RECOVERABLE FROM PLAN

4. **JSON export and export-ready link.** Why: the export satisfies data portability, including PRD-mandated "current personality vectors," while keeping numerical traits out of product surfaces and requiring both the link token and a signed-in session.

5. **Soft-then-hard deletion with restore.** Why: soft delete permits "I changed my mind," suspended invites make visitors unavailable, continued ticking means "a restored aviary has continued living," and hard delete destroys the account DEK so older backups are unreadable.

6. **One canonical aviary, starter pair, cap of 7, and new birds by aviary age only.** Why: one canonical aviary supports stable simulation and sync; age-only availability means presence and visit count never factor in; the cap is tied to recognizability, rendering, audio, and performance evidence.

7. **Server-side simulation tick.** Why: canonical personality, mood, perch, weather, drift, contagion, and notebook extraction stay server-authored and consistent across devices.

8. **Multi-device sync through snapshots and append-only events.** Why: every device reads the same precomputed snapshot, no client holds authoritative state, and additive events avoid personality last-write-wins.

9. **Visit invitations.** Why: visits are deliberate, opt-in, revocable, read-only, and do not create presence, interaction events, drift, co-presence, or rankings.

10. **Accessibility at launch.** Why: the plan treats screen-reader narration, call captions, reduced motion, keyboard navigation, and contrast as launch requirements and release-blocking gates, not follow-up work.

11. **Performance budgets.** Why: first-bird timing, idle frame rate, memory growth, JS size, tick latency, and offer resolution are product constraints enforced by CI, synthetic monitoring, and device-lab runs.

12. **Aggregate-only telemetry and synthetic monitoring.** Why: operational visibility is allowed, but per-account engagement, interaction history, trait distributions, and production calibration are deliberately excluded.

13. **Unsupported-browser support range, "last two majors"**: NOT RECOVERABLE FROM PLAN

14. **Shared TypeScript engine package.** Why: pure deterministic functions are shared where appropriate, while server-only drift, mood, and tick modules are excluded from the client bundle so invariant I1 is structural and trait semantics stay out of the browser.

15. **Postgres as system of record with logical shards.** Why: launch can run on one HA cluster, while 1024 logical shards let clusters be split later "without re-keying."

16. **Redis cache and encrypted object storage.** Why: Redis holds snapshots, rate limits, and revocations but is "never a source of truth"; object storage holds encrypted exports that expire.

17. **No event-streaming platform in v1.** Why: event volume per aviary is "tiny," Postgres gives ordered idempotent consumption and a single transaction with state writes, and there is "no second system to operate."

18. **Client/server render-pipeline boundary.** Why: the server owns canonical roster, mood, perch, weather, and scheduled events; the client owns micro-motion, lighting interpolation, audio mix, and captions so presentation can vary without mutating state.

19. **`greeting.observed` observation reports.** Why: they feed the notebook only, are treated as observations, and "never mutate traits."

20. **Separate `calibration` environment.** Why: staff dogfooding and time-compressed simulations need consented, isolated data; production interaction data never flows into calibration and calibration data never flows into analytics.

21. **Encrypted email plus blind index.** Why: the blind index allows sign-in lookup while preserving "email stored once, on the account record" and keeping it inside the auth module.

22. **Visitor emails encrypted under the host's DEK.** Why: visitor emails are "host-owned data" and are deleted with the host account.

23. **Retention windows for events, presence, observations, exports, and backups.** Why: raw interaction data and presence are short-lived; observations last long enough for "first time this week/month" comparisons; export files expire; DEK destruction makes deleted accounts unreadable in backups.

24. **Auth link endpoint always returns `202` with rate limits.** Why: "no enumeration" and bounded abuse.

25. **GET-to-POST confirmation for sign-in and visit links.** Why: email link scanners that prefetch GETs do not consume tokens.

26. **Snapshot projection with ETags and quantized `expr` values.** Why: clients can render personality from bounded expression parameters while raw trait values never appear in snapshots.

27. **Batched, idempotent event ingestion and client outbox.** Why: retries after network failures are no-ops, the client keeps events until acked, and pagehide can flush through keepalive or sendBeacon.

28. **Synchronous offer wrapper.** Why: offers "must react immediately, not on the next tick," with a target server time for the reaction script.

29. **Read-only notebook with no mutation endpoints**: NOT RECOVERABLE FROM PLAN

30. **Renaming constraints and "renaming affects nothing else"**: NOT RECOVERABLE FROM PLAN

31. **Visitor snapshot checks and visitor audience restrictions.** Why: `/v1/visit/snapshot` checks invite and session status on every call for revocation, and visitor tokens are rejected by every other endpoint so visitors cannot create presence or interaction events.

32. **Full 60-second tick cadence, including dormant aviaries.** Why: the PRD says the tick runs regardless; a dormant tier is only allowed later after variable-`dt` equivalence proves identical results.

33. **Shard leases, CAS, and one-transaction commits.** Why: leases distribute work, CAS resolves split-brain workers, and state, traits, observations, notebook writes, and event cursor advancement commit together.

34. **Pure deterministic step and PRNG seeding.** Why: deterministic replay, property tests, catch-up, and golden files require no wall-clock randomness inside the engine.

35. **Variable-`dt` catch-up.** Why: after an outage, one catch-up step should produce the same continuous state as minute-by-minute ticking, while deterministic schedules handle discrete stochastic events.

36. **Two-stage low-pass drift reservoirs.** Why: reservoirs keep draining after the user leaves, making "personality drifts during absence based on inputs from before they left" literally true.

37. **Monotonic drift, headroom, and daily cap.** Why: monotonicity ensures "nothing goes down"; headroom lets long-lived aviaries keep drifting slowly for years; the daily cap keeps a single session from visibly moving a trait.

38. **Presence saturation, offer cooldowns, and credited-offer caps.** Why: the plan states these stop curiosity from saturating in one session, which is the PRD's stated reason for the cooldown.

39. **Captions count as audible for vocal drift.** Why: deaf and hard-of-hearing users attend to calls through captions and "must not get a slower-drifting aviary."

40. **Mood latent state, labels, hysteresis, and impulses.** Why: latent arousal/ease/interest supports continuous mood while labels avoid rapid flipping; the exception lets an alarm cross a threshold and make a bird wary immediately.

41. **Overnight roost attractor instead of a mood reset.** Why: it creates the "daily-ish cadence" while mood persists continuously and "never snaps on tab open."

42. **Hidden attunement scalar.** Why: it is the mechanism for birds becoming quieter and greeting less after absence without lowering traits, lowering ease, showing a meter, or creating a failure state.

43. **Perch utility, dwell rules, timeline moves, and fixed slots.** Why: utility links perch choice to personality, mood, weather, and attunement; timeline offsets let clients animate at the right moment instead of teleporting; slots make seven birds fit.

44. **Expression projection.** Why: quantized bounded `expr` values let the client render personality without learning raw trait values.

45. **Offer reaction resolver and stored reaction script.** Why: immediate pure resolution gives the client a script now, stores the same outcome for other devices, and lets the tick later apply drift and mood without re-rolling.

46. **Invisible offer cooldown and latency-hiding notice beat.** Why: "No cooldown message ever appears" preserves the no-announcement rule, while the notice beat gives immediate bird attention even before the server script returns.

47. **Timezone rules, glide, and fixed civil day curve.** Why: the two-hour rule avoids timezone flapping, the 20-minute glide avoids jumps, and the fixed IANA day curve avoids collecting location.

48. **Canonical weather and scheduled alarm calls.** Why: every device and visitor agrees on weather, and scheduled alarms let every client play the alarm while contagion stays consistent.

49. **Starter-pair constraints and onboarding arrival.** Why: seeded boldness separation makes one bird reliably greet first and greeting order legible; voice-print distance protects recognizability; the empty-to-fly-in sequence is the only empty aviary moment.

50. **Age-based visiting birds and adoption.** Why: thresholds line up with "a few months" and "a year," presence and visits never factor in, the flow has no announcement or reminders, and the visiting bird's identity carries into adoption.

51. **Notebook observation extraction and sparsity controller.** Why: entries come from noteworthy aviary happenings, but token buckets and hard limits keep writing sparse "regardless of how active the user is."

52. **Notebook grammar, no LLM, and hard content rule.** Why: no LLM protects privacy and gives voice control, determinism, and testability; the extractor has no user-behavior aggregates so it cannot write visit or streak language.

53. **Drift calibration harness.** Why: it operationalizes "visible" drift, proves one-session movement stays below the threshold, checks captions-only equals audible, and tunes constants without production-user calibration.

54. **Single canonical sync state and additive event log.** Why: devices cannot disagree beyond pull gaps, there is no merge logic, retries are idempotent, and events carry no absolute personality values.

55. **Server presence clamping and union across devices.** Why: client timestamps are advisory, spoofing and offline outboxes are bounded, and simultaneous devices count once.

56. **Snapshot propagation, interpolation, and staleness handling.** Why: clients play server-timed timeline entries, schedule natural flights rather than teleporting after contradictions, and keep system errors as a small top-bar line rather than an in-scene overlay.

57. **Names and settings conflict handling.** Why: names and settings are "user intent, not simulation state"; a single renamed field does not need a conflict dialog, and settings can be per-field last-write-wins.

58. **Settle across devices.** Why: settled lighting is "device-local and session-scoped," while the small mood-quieting impulse is canonical; waiting 5 seconds before sending avoids accidental settles reaching the server.

59. **Visit invite, claim, and lifetime rules.** Why: one-time links mint browser-bound visitor sessions, keep visits deliberate, and avoid a self-updating visitor list.

60. **Visitor render-only mode.** Why: visitors see "exactly what the host would see" but cannot create presence, listen-in, offer, settle, greet, or affect host drift.

61. **Visit log, optional notifications, and no aggregates.** Why: logs are silent and unbadged, notifications are default off and capped, and no cross-account visit statistics exist that could later become leaderboards.

62. **Frontend stack, custom renderer, and bundle budgets.** Why: Preact is small for UI, the custom WebGL2/Canvas2D renderer avoids a general-purpose game engine for "bundle and control," and the internal budget keeps deliberate headroom below the PRD cap.

63. **Procedural bird rendering and responsive layout solver.** Why: parametric rigs avoid bird bitmaps, plumage can be shader-driven and quantized, and layout property tests keep all birds and flight corridors inside the safe area.

64. **Behavior layer and first-frame motion.** Why: utility-selected micro-actions make birds vary by mood and expression, and wall-clock pose sampling ensures frame 1 is mid-action, never a neutral T-pose.

65. **Scene composition, quiet loading field, and warm start.** Why: ambient ornaments carry no state, cold loads use a quiet field with no spinner or text, birds appear at current perches with no wake-up, and old cached snapshots are rejected to avoid stale mood snapping.

66. **Top bar icons, fade, and no badges.** Why: controls remain sparse and icon-based, fade avoids constant chrome, focus or open panels keep it visible, and badges/dots/counts are forbidden.

67. **Reduced-motion presenter.** Why: reduced motion is authored as a separate aesthetic with pose holds, cross-fades, no particles, slower lighting, unchanged calls/captions/drift, and review by vestibular-sensitive testers.

68. **Return-greeting.** Why: account-wide absence prevents quick tab switches from re-greeting; greeter selection and procedural variation make style stable but details varied; no DOM welcome text preserves the no-announcement rule.

69. **Listen-in.** Why: listen-in changes the mix, not bird behavior; it has no visual chrome beyond keyboard focus and uses server-measured capped duration for drift.

70. **Offer menu and song fragments.** Why: top-bar offers go "to the aviary" while focused listen-in gives a natural primary recipient; song fragments are synthesized, not samples; item limits prevent duplicate clutter.

71. **Settle interaction.** Why: settle is a quiet goodbye with local lighting and mix changes, no undo text, a 5-second undo before server commit, and tab close equivalent at the engine level.

72. **Audio unlock and WebAudio fallback.** Why: browser autoplay policy is handled by a silent-running scheduler that fades in mid-phrase with no enable-sound prompt, and WebAudio failure becomes graceful silence with captions defaulted on.

73. **Synthesis engine, fixed voice print, and call grammar.** Why: a single AudioWorklet and fixed voice pool avoid node churn and GC; fixed voice prints make "pip sound like pip"; grammar variation changes mood and rate without losing signature.

74. **Mixing and listen-in floor.** Why: listen-in creates a "leaning in" feel while the floor is "never muted," so other birds remain ambient.

75. **Captions and recognizability.** Why: captions derive from the actual phrase object so text matches sound; recognizability is "load-bearing" and gates voice-print minting, CI, human panels, and `max_birds` ramp.

76. **Screen-reader narration, keyboard model, contrast, and voice linter.** Why: nonvisual users receive polite naturalist observations from the same state as the renderer, keyboard users can traverse birds and panels, contrast is verified across lighting states, and generated prose is linted to avoid stats, second person, and announcement grammar.

77. **Performance observability and deliberate non-measurement.** Why: RUM and server metrics cover timing, frame health, audio failures, errors, and invariants, while engagement, funnels, trait distributions, session replay, and affective A/B tests are not measured by design.

78. **System emails and copy registry.** Why: allowed emails are transactional and never about aviary state; system copy is centralized for content review and separated from naturalist grammar.

79. **Rollout, max-birds ramp, flags, and migrations.** Why: foundational engine/privacy work comes first because it is hardest to retrofit; drift dogfood starts early because it takes weeks to feel; birds ramp only after evidence; flags must be presentation-safe; migrations preserve identity and traits.

80. **Test strategy and English-only v1.** Why: tests cover engine, sync, client, audio, accessibility, performance, privacy, and voice; English-only v1 exists because naturalist voice "doesn't machine-translate."
