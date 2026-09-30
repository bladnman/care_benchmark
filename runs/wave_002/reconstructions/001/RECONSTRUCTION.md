## System-level intent

1. Continuous private aliveness across time and devices. This shows up first in "feels continuous across visits and devices" and in the requirement that "mood, timing, weather and perch state continue when no browser is connected." The same intent returns in the minute tick, outage catch-up, sampled in-progress first paint, pull/reconcile rules and the final readiness phrase "continuous two-bird encounter" and "week-scale persistence."

2. Canonical server ownership over simulation and identity. The plan repeatedly separates durable truth from rendering: "canonical server simulation," "no client can overwrite personality," "the simulation worker is the sole role authorized," "the browser never chooses a mood," and "no client absolute state saves." Client work is presentation: interpolation, procedural synthesis, captions, ornaments and local descriptors.

3. Permanent identity with slow, positive-only personality. The plan carries "birds have permanent identities; personality changes slowly and only upward" through stable UUIDs, direct vector persistence, "no replacement on rename," no reseeding reads, additive nonnegative deltas, no database rollback of accepted effects and risk responses that say to "never fix by resetting birds."

4. Quiet, non-engagement product posture. The scope rejects "quests, points, scores, levels, streaks, visit-frequency surfaces," "badges," "unlock countdown," "welcome text," "toast" and notification badges. Later sections preserve the same posture through no absence pings, no visible timers, calm inline failures, sparse notebook entries, no user-facing counts and tests that reject banners, counters, badges and absence notices.

5. Naturalist voice rather than status UI. Product text is "lowercase, present-tense, particular observations." The observation library must use semantic scene facts, "specificity, present tense, lowercase and no announcements or behavioral surveillance." Captions, narration and notebook prose are grounded in actual scene facts rather than labels such as "mood: content" or "perch 2."

6. Privacy and minimization are design constraints, not afterthoughts. The plan uses encrypted contact data, blind indexes, private no-store snapshots, no trait values in snapshots, read-only visits, telemetry allowlists, no request-body logging, no session replay and no simulation warehouse access. It also says exported bytes stay out of telemetry and support diagnostics require explicit narrow access.

7. Accessibility is a first-class route to the same aliveness. "All accessibility surfaces" are in launch scope. Captions and focus are explicit exceptions to the no-inline-chrome scene rule, the scene has a separate semantic DOM, reduced motion has its own "aliveness review," and launch gates require keyboard, VoiceOver, NVDA, captions, focus outlines, audio-off users and reduced-motion users.

8. Performance must be honest and compatible with privacy. The plan treats the "500ms first-bird target" as end-to-end, including state fetch and first paint, and says to report warm and cold navigation separately. It accepts a quiet field fallback only when state is late and says missed targets are reported honestly, not hidden behind cache averages.

9. Recognition and gradual change are central to the product feeling alive. The plan emphasizes six species silhouettes/signatures, stable call seeds, "recognizable interval/timbre tendencies," gradual plumage/behavior mappings, week-one and week-three calibration, human recognition at seven birds and the closing statement that "aliveness is the central behavior being delivered."

10. Correctness comes from transactions, idempotency and versioning. The plan names PostgreSQL transactions as the correctness boundary; event sequences, row locks, idempotency keys, revision checks, transactional outbox messages, duplicate queue delivery handling, versioned engine/projection/configuration and migration fixtures all serve that intent.

11. Memory should be sparse, truthful and about birds, not attendance. The notebook and observation memory support claims like "first greeting change this week" while forbidding "user visit streaks." Notebook generation is sparse, stores final prose, preserves old observations on rename and avoids counts or adoption milestones.

12. Launch is gated by both technical tests and qualitative observation. The implementation sequence requires synthetic state prototypes, simulated 7/21/90-day timelines, assistive reviews, visual/audio reviews, human studies and staged rollout. The plan explicitly says "engineering completion does not substitute for the listening and observation reviews."

## Per-feature whys

**Delivery contract and scope**

1. Browser-based, single-account aviary: The plan's rationale is a private aviary that "feels continuous across visits and devices" while avoiding multi-aviary or shared-account complexity.

2. Two server-selected starter birds: The plan ties readiness to the "continuous two-bird encounter" and requires every cohort to get two starters.

3. Six coherent species: The plan connects species to distinguishable silhouettes/signatures, curated grammar families and human recognition at seven birds.

4. Seven-bird ceiling: NOT RECOVERABLE FROM PLAN

5. Age-based additional adoption: The plan makes adoption "optional and based exclusively on aviary age" so more birds are not driven by engagement mechanics or visit frequency.

6. Specific adoption thresholds at 90, 180, 270, 365 and 540 days: The plan says six birds at one year and seven later matches the brief's "approximate pacing" and that earned opportunities cannot be revoked by later configuration.

7. Quiet adoption option in settings: The plan says not to send emails, add badges, count birds visibly or display an unlock countdown, keeping adoption out of the no-ping engagement loop.

8. Scope exclusions such as payments, native clients, passwords, SSO, public discovery, follows, comments, rankings and co-presence: The plan uses these exclusions to protect the single-account, quiet, private scope and explicitly says not to build dormant ranking schemas.

9. Product copy style: The plan treats copy as a product requirement: lowercase, present-tense, particular observations for the aviary, with direct standard English for identity, errors, privacy, sync and accessibility settings.

10. Three offer modalities as a fixed set: NOT RECOVERABLE FROM PLAN

11. Listen-in: The plan treats listen-in as deliberate attention, not proof of hearing; it raises the focused bird while keeping others at a nonzero ambient floor and contributes bounded per-bird influence.

12. Settle: The plan's rationale is a calm local quieting state that ends that device's presence, softly quiets calls and creates no missing-goodbye penalty.

13. Sparse permanent field notebook: The plan's rationale is durable, specific bird observation, not surveillance or counts; it stores final prose and bird IDs and preserves old prose after renames.

14. Optional private visits: The plan makes visits read-only and separate from simulation so a visitor can see the same ambient projection without notebook, settings, private history or interaction capabilities.

15. Account lifecycle controls: The plan uses sessions, email change, export, deletion, recovery and purge to support privacy, revocation, preservation and full account removal.

16. Accessibility surfaces: The plan makes accessibility part of launch scope and requires captions, focus, narration, reduced motion and actual screen-reader review as gates.

17. Opaque encrypted vector export: The plan reconciles complete account export with the ban on numerical trait exposure: the user keeps a complete state artifact "without a trait dashboard."

18. Visit notification opt-in: The plan says this narrowly fulfills an in-product notification while retaining the "global no-ping posture" and avoiding push, email, toast or badges.

19. Top bar with exactly account/settings, accessibility, notebook and offer icons: The plan keeps settle reachable from the top bar through the offer panel "without adding a fifth icon."

20. Captions and focus exceptions: The plan permits accessibility text near calls and keyboard focus outlines as exceptions to the no-inline-chrome rule while still banning mood labels, hover tooltips and permanent bird name tags.

21. Persisted account IANA timezone: The plan says one timezone avoids "simultaneous clients requesting incompatible mornings"; visits use the host timezone.

**System shape and ownership**

22. TypeScript web application and service: NOT RECOVERABLE FROM PLAN

23. PostgreSQL, durable queue, scheduler and transactional email delivery: The plan says PostgreSQL transactions are the main correctness boundary and queue delivery may repeat without repeating effects.

24. One deployable API/simulation service with a separate worker using the same versioned domain library: The plan uses this shape to keep API and simulation together initially while the worker owns scheduled canonical updates.

25. Canvas2D scene composition with procedural/SVG-derived geometry: The plan's rationale is a compact procedural scene shell that does not require bitmap download for the initial render.

26. Separate semantic DOM for focus, narration and controls: The plan uses it so the canvas scene can coexist with keyboard, screen-reader and control semantics.

27. Lazy-loaded settings, notebook history and invitation management: The plan defers these noncritical surfaces to protect the initial HTML and first draw.

28. Versioned behavior projection: The plan turns traits into bounded pose/call parameters as "descriptive rendering input," never a writable trait object.

29. Simulation worker as sole writer of personality and canonical mood: The plan uses sole ownership to prevent clients, account handlers or naming handlers from mutating vector columns.

30. Rendering planner with server-authored action descriptors: The plan lets fast reactions appear without waiting a full minute, while mood/drift effects wait for the next tick.

31. Render-only client work: The plan allows interpolation, procedural synthesis, captions and ornaments but says they are not simulation ticks and must not become canonical bird history.

32. Private edge snapshot delivery: The plan uses the edge path for speed while preventing one account's birds from being exposed to another through shared caching.

33. Versioned render snapshot beside canonical rows: The plan avoids reconstructing personality from events on reads and supports compact authenticated snapshots.

34. Quiet sky field fallback: The plan prefers subtle empty-state cues over invented placeholder birds when state is late.

35. Established accounts beginning at sampled in-progress action phase: The plan uses this to preserve continuity and avoid an entry animation on every visit.

**Durable model**

36. UUID primary keys and UTC timestamps: NOT RECOVERABLE FROM PLAN

37. Encrypted email contact data and keyed blind-index lookup: The plan avoids raw email in URLs, logs, traces, analytics and service identifiers while still supporting lookup and uniqueness.

38. Device sessions with hashed opaque credentials and minimal browser labels: The plan allows one token per browser sign-in without fingerprinting and supports settings revocation.

39. Auth challenges with one-use random tokens: The plan supports expiring magic links, scanner-safe explicit consumption and removal of unknown-email delivery envelopes.

40. Aviary record with revision, tick state, seed, weather, local-day phase, recent-attention envelope and scene plan: The plan stores canonical continuity in one durable owner of the aviary state.

41. Bird record with stable UUID, traits, accumulators, mood, perch state, action descriptors and call signature seed: The plan preserves identity, behavior memory and recognizable calls; rename does not replace a bird.

42. Interaction event log with idempotency key and server sequence: The plan enables append-only accepted gestures, duplicate detection and ordered tick processing until retention expiry.

43. Presence interval record: The plan stores only validated visibility/focus/recent-input evidence as retention-limited simulation data, not analytics.

44. Notebook entry record: The plan keeps server-authored final prose tied to stable bird IDs, with no user edit/delete routes and historical prose preserved on rename.

45. Observation memory: The plan enables true sparse claims such as "first greeting change this week" while explicitly avoiding user visit streaks.

46. Invitation record: The plan supports named private visits through encrypted visitor contact and hashed one-use secrets without creating a social profile.

47. Visit session/log record: The plan records approximate duration for owner transparency while keeping visits out of simulation presence and bird events.

48. Export job record: The plan supports consistent encrypted exports while keeping exported bytes out of telemetry.

49. Adoption opportunity record: The plan enforces unique threshold acceptance and the seven-bird transactional cap.

50. Direct vector persistence with filter memory: The plan says events can be compacted without affecting identity and that reads must not fill missing vectors with seeds.

51. Encrypted backup, point-in-time restore and migration checks: The plan's rationale is preserving bird IDs, traits, accumulators, notebook entries and call identity, with corrupt state quarantined instead of regenerated.

**APIs and security contracts**

52. Owner-route session, ownership, deletion and CSRF enforcement: The plan protects owner data and mutation routes through live session, account ownership, deletion status, CSRF, origin checks and CSP.

53. Magic-link authentication as the chosen auth form: NOT RECOVERABLE FROM PLAN

54. Generic magic-link request response, throttles and one-use link: The plan avoids address-existence disclosure and abuse while permitting retries after the window.

55. Explicit magic-link consumption route: The plan prevents email scanners from consuming links with GET and makes replay fail cleanly.

56. Aviary snapshot API: The plan returns enough for rendering while excluding trait values, attention totals and visitor identities; ETags support repeat pulls.

57. Interaction API: The plan accepts bounded idempotent event batches and rejects absolute personality or arbitrary mood payloads.

58. Notebook API: The plan uses stable keyset pages so all history remains accessible without a mutation endpoint.

59. Adoption start API: The plan makes starter creation idempotent so repeated navigation cannot duplicate birds.

60. Bird rename API: The plan allows naming while preserving identity and behavior.

61. Adoption opportunity accept API: The plan locks the aviary, verifies age and cap and consumes the opportunity once before returning the new bird identity.

62. Settings API: The plan uses revision checks so stale patches get 409 with current settings, avoiding silent whole-record overwrite.

63. Session list and revoke API: The plan lets owners revoke sessions immediately, including signing out the current browser.

64. Email change flow: The plan verifies the new encrypted address with a purpose-bound challenge before atomic replacement, keeping the old address valid until then.

65. Account export API: The plan enqueues a transactionally consistent snapshot and treats the email as an explicit-request system email.

66. Delete and recover APIs: The plan soft-deletes for 30 days, keeps recovery available and suspends ordinary writes and simulation until recovered.

67. Invitation creation API: The plan requires an explicit named recipient and one-time link, with no automatic invitations.

68. Invitation and visit owner APIs: The plan keeps outstanding invites and quiet visit logs owner-only.

69. Invitation revoke API: The plan atomically stops outstanding or active access without a success toast.

70. Visit consume API: The plan exchanges a one-use invite secret for a separate read-only cookie without requiring an account.

71. Visit aviary API: The plan checks revocation every request and returns the ambient scene without notebook, settings, private attention/history or interaction capabilities.

72. Visit keepalive/end APIs: The plan updates approximate visit-log duration only and prevents visit presence or simulation events.

73. Initial 30-day absolute device-session lifetime: NOT RECOVERABLE FROM PLAN

74. Active visit lifetime and revocation rules: The plan enforces at-most-24-hour active access, 30-day unused expiry, next-pull revocation and a shared unavailable surface for revoked or expired links.

75. Historical visit records after revocation: The plan keeps transparency about past access while hiding revoked outstanding access from the active list.

**Simulation and calibration**

76. Minute tick transaction: The plan runs every non-deleted aviary independent of connections so canonical state advances even when nobody is connected.

77. Duplicate tick and crash behavior: The plan compares scheduled time with last tick, commits atomically and makes crash-before-commit no-op and crash-after-commit harmless duplicate.

78. Outage catch-up with historical timestamps: The plan prevents the returning browser's present time from fabricating presence or skipping trait changes.

79. Stale snapshot handling: The plan permits stale rendering with a system error while insisting the API not masquerade stale state as freshly simulated.

80. Five-minute presence activity window: The plan counts attention only with visible document, focus and recent pointer or keyboard input; initial navigation alone does not count.

81. Presence interval chunking and terminal beacons: The plan counts only observed elapsed eligibility and says a missing beacon does not grant a grace lease.

82. Offline presence exclusion: The plan says "preserving honest drift outweighs crediting unverifiable time."

83. Multi-device owner interval union: The plan prevents laptop and phone attention from doubling the aviary budget.

84. Listen-in interval union and split budget: The plan caps per-bird listen influence and prevents simultaneous listen-in from doubling attention.

85. Separate visit session/event boundary: The plan prevents visits from affecting simulation presence or bird events.

86. Positive-only low-pass drift: The plan allows the accumulator to decline during absence but says "the personality never does."

87. Presence-dominant trait influence: The plan keeps presence at least 75% of influence and secondary signals at most 25% so offers are not "a cheap trait farming path."

88. Week-scale calibration: The plan targets small instrument-resolvable changes around seven days and noticeable mapped behavior/appearance shifts around 21 days, not exposed numbers.

89. Absence behavior: The plan lets recent-attention decay reduce greeting probability but forbids distress, mistrust, punitive mood or loss of stored traits.

90. Five mood states and weather: The plan uses dwell times, local-day baseline, rain and wind to create ambient/context changes that persist and do not reset on navigation.

91. Persisted weather schedule: The plan ensures every browser sees the same weather and handles DST through IANA conversion.

92. Server behavior planning for perches, actions and calls: The plan gives mood and boldness visible effects while preserving collision-free slots and bounded chorus response.

93. Greeting on owner return/navigation: The plan provides a first response within 1-2 seconds using server-authored absence/personality/mood context, not prerecorded variants.

94. Greeting debounce and visitor exclusion: The plan avoids synchronized arrival choruses, focus-flapping duplication and visitor-triggered greetings.

95. Offer reservation and cooldown: The plan prevents competing devices from bypassing per-bird cooldowns.

96. Offer targeting near a recipient/group rather than direct click-on-bird: The plan keeps offers as scene gestures and lets mood/curiosity produce investigate, wait or ignore.

97. Calm no-ready response for offers: The plan avoids visible timers and punitive alerts, using a calm naturalist observation instead.

**Synchronization and continuity**

98. Visible-page pull cadence and ETags: The plan keeps visible clients current after navigation, resume and render gaps while avoiding out-of-order state replacement.

99. Stricter visit pull cadence: The plan uses at-most-10-second visitor pulls so revocation is enforced on the next request.

100. Interaction retry with same event UUID: The plan returns the original acceptance and descriptor after a lost response rather than creating a second offer.

101. Bounded in-memory retry queue: The plan tolerates short disruption but avoids a long offline backlog that could manufacture retroactive mood or presence.

102. Local acknowledged action descriptors: The plan makes gestures feel immediate while keeping canonical mood and traits unchanged until the server tick.

103. Revision-checked names/settings: The plan keeps conflict handling clear and separate from additive simulation events.

104. Settle as local presentation plus bounded quieting input: The plan keeps other devices' canonical scene intact and lets a quick linked undo reverse lighting/mix and quieting.

**Rendering and interaction surface**

105. Normalized horizontal scene layout: The plan preserves silhouettes and all seven birds across viewport widths through safe margins, collision radii and remapped perch slots.

106. No scene pan, scroll or zoom: The plan keeps the scene itself stable while settings and notebook panels may scroll.

107. First-paint action sampling: The plan starts birds mid-action when appropriate so returning accounts do not replay artificial entry animations.

108. Bounded low-frequency motion and ornaments: The plan creates breathing, weight shifts and environmental aliveness without regular identical loops or catch-up animation bursts.

109. Trait-reflective feather detail and saturation: The plan maps gradual traits through published render parameters without showing numerical personality.

110. Hidden-page rendering and audio pause: The plan stops animation, ornament timers and audible scheduling when hidden to avoid missed-call replay and waste.

111. Visible-but-unfocused rendering with stopped presence: The plan separates looking at the scene from earning attention.

112. Top-bar fade behavior: The plan keeps the scene quiet after pointer stillness while protecting keyboard focus, open panels and assistive navigation from fading.

113. Bird listen-in pointer and keyboard controls: The plan gives stable ID-based focus, roving tabindex, arrow navigation, Enter/Escape controls and focus-retaining updates.

114. Offer panel accessibility and focus restoration: The plan requires operable choices, accessible help and focus restoration.

115. Reduced-motion renderer: The plan preserves aliveness with designed still poses, slow cross-fades, slower day/evening shifts and retained greeting/call distinction instead of a blanket animation-disable switch.

**Procedural audio and matching captions**

116. Species audio grammars and per-bird signature seeds: The plan makes birds recognizable through interval/timbre tendencies and avoids species-wide identical sound.

117. Versioned grammar anchors: The plan preserves signature identity across grammar updates.

118. Server-published call windows and bounded grammar parameters: The plan keeps behavioral calling on the server while the browser handles presentation scheduling.

119. One descriptor driving synthesis, pose and caption text: The plan ensures captions describe the actual procedural phrase.

120. Bounded WebAudio node path: The plan minimizes support risk, caps simultaneous voices and total gain and avoids repetitive recorded loops.

121. Listen-in gain automation: The plan avoids hard cuts by ramping from current gain and keeps nonfocused birds at a nonzero ambient floor.

122. Recording listen intervals while muted: The plan says listen-in represents deliberate attention, not proof of hearing, and mute causes no negative drift.

123. Settle audio quieting: The plan softly quiets all calls over several seconds without permanently silencing birds.

124. Autoplay/WebAudio failure path: The plan renders birds and captions immediately, avoids permission modals or nags and continues offering, notebook and mood in silence.

**Narration, notebook and access quality**

125. Curated observation library from semantic scene facts: The plan avoids an LLM service and supports grammar-safe, specific, present-tense naturalist prose.

126. Narration live region cadence: The plan keeps updates polite and bounded, replacing obsolete queued observations instead of accumulating them.

127. Prompt but calm priority observations: The plan lets greetings, offers and settle receive priority without breaking the calm prose queue.

128. Stable actionable bird labels plus naturalist prose: The plan avoids hundreds of changing ARIA labels while keeping birds operable by name/species.

129. Optional visual captions near calling birds: The plan supports call access even when synthesis is unavailable, with contrast, safe placement and chorus merging.

130. Notebook tick generation from sparse candidates: The plan creates observations from meaningful bird/weather/gesture facts rather than raw log dumps.

131. Notebook rate limits: The plan keeps history sparse with one entry per two to four days, at most one per day initially and a rolling cap of three per seven days.

132. Notebook preservation and pagination: The plan stores final prose, avoids rewriting old entries and uses keyset pagination plus bounded rendering for indefinite history.

133. Keyboard/screen-reader review: The plan requires the complete adoption-to-offer-to-notebook-to-settings journey and actual VoiceOver/NVDA review.

134. WCAG AA, focus and caption checks: The plan makes accessibility visible across brightest/dimmest palettes and protects focus outlines from top-bar fading.

**Privacy, retention and account operations**

135. Service-owned simulation database isolated from analytics: The plan prevents operational analytics from querying simulation rows and events.

136. Telemetry allowlist: The plan excludes bird IDs, names, vectors, grammar seeds, event payloads, offer/listen counts and per-account interaction series.

137. Request and session-capture restrictions: The plan disables request body logging, query/credential capture, session replay and screenshot capture.

138. Raw event retention for 30 days: The plan keeps consumed interaction/presence events only for bounded correctness repair, then removes them after cursor/accumulator verification.

139. Indefinite notebook retention while account exists: The plan treats notebook as account memory while minimizing transient invite delivery data.

140. Export object encryption and 24-hour one-use link: The plan avoids public object URLs and cacheable endpoints and deletes export objects after expiry.

141. Account deletion invalidations: The plan immediately cancels export jobs and invalidates links, device sessions and invitations.

142. Hard-delete worker and envelope encryption: The plan removes account-owned data and destroys keys so retained backup ciphertext becomes unreadable.

143. Hard-delete tombstones outside restorable account data: The plan re-applies deletion after disaster restoration.

144. Aggregate irreversible operational histograms: The plan allows operations data only without account dimension and without recoverable relationship data.

**Performance budgets and observability**

145. 500ms first-bird metric: The plan defines it end-to-end on a mid-tier phone over fixed 4G so performance is measured as the user experiences it.

146. Separate cold and warm navigation reporting: The plan says not to hide cold failure behind cache averages.

147. Critical JS and snapshot size budgets: The plan uses under-300KB working JS target, 2MB hard cap, small HTML/state and deferred panels to protect first draw.

148. 60fps seven-bird session target: The plan preserves mood, identity, poses and calls while lowering decorative detail on slow visible devices.

149. Memory growth definition and soak tests: The plan focuses on retained-memory slope after warmup and explicitly tests notebook scrolling, listen switches, resize, hidden/resume and audio failures.

150. Operational metrics and tick alarms: The plan observes route latency/errors, tick compute, queue lag, oldest snapshot age, deadletters and first-bird timing without account dimensions.

151. Synthetic browsers and aggregate-only RUM: The plan measures across geographies with stripped identifiers and no real-user relationship aggregation for calibration.

152. Scoped support diagnostics: The plan allows support access only with explicit narrow scope, audit and no collection into product analytics.

**Implementation sequence, tests and risks**

153. Contracts and quality prototype: The plan uses schemas, invariant tests, species silhouettes/signatures, voice library and synthetic rendering/audio before production implementation to gate aliveness.

154. Durable canonical core milestone: The plan gates on no negative traits, no resets, no duplicate delta, exact owner-presence union and no visitor simulation influence.

155. Owner vertical slice: The plan combines magic links, two-bird adoption, initial snapshot, greeting and procedural audio to test first-bird performance and greeting latency.

156. Complete quiet interactions and access milestone: The plan integrates offer reactions, mix ramps, settle/undo, notebook, controls, captions, narration and reduced motion before later launch gates.

157. Lifecycle and optional visits milestone: The plan validates visitor denial, privacy schema tests, hard deletion and encrypted backup restoration preserving deletions.

158. Launch qualification: The plan requires browser/device coverage, memory/frame soak, seven-bird rain/chorus and assistive reviews before broader availability.

159. Staged rollout: The plan starts with staff synthetic accounts, then small consented beta, then 5%, 25% and full signup after technical and qualitative gates pass.

160. Feature flags for admission and presentation assets: The plan says flags must never erase identity or recalculate personality.

161. Synthetic timeline acceleration: The plan verifies months/year adoption before those ages occur in real life.

162. Calibration human studies: The plan uses dedicated synthetic birds and explicit observations instead of hidden real-account aggregate analysis.

163. Engine version freeze and reviewed tuning: The plan preserves existing state on migration and tunes only between reviewed versions.

164. Failure response to quality problems: The plan stops new enrollment or disables unstable cosmetic rendering while continuing canonical ticks, retaining identities and showing system error guidance.

165. Forward-compatible migrations and forward fixes: The plan says a database rollback must not roll back accepted event effects.

166. Test inventory tied to failure modes: The plan grounds tests in trait bounds, presence accounting, fault tolerance, privacy isolation, visual/audio review and access/performance behavior.

167. Risk operating responses: The plan maps known risks to prevention, including calibrated drift, union intervals, durable scheduler catch-up, curated audio grammars, accessibility launch gates, compact private snapshots, migration audits, metric allowlists and tests against engagement mechanics.
