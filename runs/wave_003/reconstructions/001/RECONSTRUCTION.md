## System-level intent

- **Invariants are the contract, not advisory guidance.** This appears first in §0 and §1: "§1 (invariants) is the contract," and "a change that violates an invariant is out of scope however small it looks." The plan repeatedly turns product principles into "Primary enforcement" and "Verification" through grants, triggers, lints, CI, e2e tests, audits, and launch gates.

- **The server simulation is canonical; clients are views.** INV-1, §4.2, §5.5, and §8.1 all carry this principle. The plan says the client "never receives" personality vectors, "never owns simulation state," and only renders snapshots and submits events. The tick is separate because it is "the sole writer."

- **Growth is monotonic, slow, and non-punitive.** INV-2 and §7.4 state that drift is "additive and non-negative" and that "Neglect never lowers a trait." The refusal of Tamagotchi mechanics in §2.2, the absence-free mood model in §7.6, and the risk language in R-1 and R-2 all point to birds changing over weeks without session-to-session pressure.

- **Birds must not become stats.** INV-3 says the personality vector is "never shown to the user," and D-2 explains that the exposure ban targets product surfaces that "turn birds into stats." This also appears in account settings, support tooling, ARIA names, notebook detectors, and the no-import export stance.

- **Bird identity is permanent and custodial.** INV-4 says a bird's identity lasts as long as the account, and D-26 refuses removal because "Bird identity is permanent" and removal is "custodial-flavored." Species rev pinning, migration checksums, hard-delete-only deletion, and "no code path replaces, regenerates, re-seeds, or resets a bird" enforce this.

- **Presence means active attention, credited conservatively.** INV-5 defines presence as the conjunction of visibility, focus, and recent pointer/key activity. D-7 says the PRD's failure mode is "over-counting," and the "strict reading can only under-count on phones, which is the safe direction." §7.3 makes overlapping devices "worth what one device is worth, never double."

- **The product should not announce, count, rank, or reach for the user.** INV-9 bans "welcome toasts or banners, badges, counters, confetti, spinners, push, or emails about the aviary." §2.2 refuses achievements, streaks, levels, scores, visit calendars, public discovery, leaderboards, and notifications. D-9 says "The aviary itself does the noticing."

- **Voice is split by surface.** §0 defines "Product surface" as naturalist voice and "System surface" as matter-of-fact voice. INV-12 says product surfaces are naturalist, system surfaces matter-of-fact, and the notebook and narration "describe the aviary, never the user's behavior." §11 enforces this with separate corpora, a system-string catalogue, lints, and copy review.

- **Accessibility is a first-class product surface, not a fallback.** INV-14 says narration, captions, reduced-motion rendering, keyboard, and accessibility settings ship "at full product quality, not as fallbacks." §9.9 calls reduced motion "the same aviary through a different renderer strategy," and §16.6 requires users to sign off that the experience "feels like the aviary."

- **Privacy is structural and aggregate-only.** INV-10 stores email once, encrypted, while INV-11 says telemetry is "aggregate-only" with no account, bird, session, or email dimension. §13.7 says "Nothing leaves" and "No route in"; §14.6 deliberately does not measure DAU, retention cohorts, visit frequency, interaction counts per account, or cross-account aggregates over `sim`.

- **First contact should feel alive, not loaded.** INV-13 requires the first frame to be "the aviary mid-motion, or the quiet field while state loads," with no spinner, entry animation, "ready" pop, or fade-from-static. §9.1 budgets the "First bird frame" and seeds birds, leaves, and micro-motion so "nothing starts from rest."

- **Procedural, deterministic craft is preferred over opaque runtime generation.** D-16 chooses an authored typed phrase grammar with "No LLM at runtime" because privacy, controlled voice, linting, determinism, and testability matter. D-18 simulates weather rather than using real-world weather. INV-8 bans recorded audio, and §10 synthesizes calls procedurally.

- **Individual birds should remain recognizable without repetition or convergence.** R-3 worries that birds might "all look alike," so the plan uses per-bird ceilings, immutable voice and look individuality, species rev pinning, voice-print distance, greeting signatures, ring buffers, and uniqueness tests. §7.10 says the same bird greets "the same way across visits" while different birds greet differently.

## Per-feature whys

### Scope, refusals, and decisions

- **One horizontal scene with no panning, scrolling, or zooming:** NOT RECOVERABLE FROM PLAN

- **Three perch zones and slot layout:** The plan uses front, middle, and back zones to support spatial behavior and readable layout. §7.7 says nine slots "comfortably hold seven birds plus a newcomer," and §9.5 tests every viewport so birds stay inside the scene and out of the top-bar band.

- **Day/night cycle on the aviary's local time:** D-11 chooses one canonical aviary timezone so "Mood and light stay coherent across devices and for visitors," and a brief phone check abroad does not flip the aviary. §7.9 ties both light and circadian mood to that curve.

- **Rare ambient weather:** D-18 avoids real weather because it would need location, "a privacy cost," and would not meet the "rare, never assertive" calibration. §7.9 says after-rain raises the chance a daily visitor sees weather "without anything happening for the viewer's benefit."

- **Ambient leaf and feather ornaments:** NOT RECOVERABLE FROM PLAN

- **Four-icon top bar and gestures menu:** D-1 uses a gestures menu so the top bar keeps "exactly four icons" while still containing offers and "settle the aviary." The plan groups settle and offer as "gestures toward the aviary" while settings and accessibility remain system surfaces.

- **Responsive layout from 320 px phone to ultrawide desktop:** §9.5 says the layout must "never crop a bird," keeps birds out of the top-bar band, avoids letterboxing phones, clamps ultrawide scale, and verifies every viewport from 320x480 to 3840x2160.

- **Six-species pool:** NOT RECOVERABLE FROM PLAN

- **Two starter birds, naming card, and starter arrival:** D-27 excludes the nightjar-like species because "a starter that sleeps all day would spoil the first encounter for daytime users." §7.13 uses a curated contrast table to maximize silhouette and call-register difference, and starter fly-in is the only entry animation allowed by INV-13.

- **Renaming birds any time:** NOT RECOVERABLE FROM PLAN

- **New birds by aviary age, capped at seven:** INV-7 and §2.2 make unlocks depend only on aviary age, not presence, offers, visits, or streaks, so the flow cannot become a reward system. §18.3 ramps the upper cap only after recognizability, performance, and layout gates pass.

- **Hidden personality vector:** INV-3 keeps the vector off product and system surfaces because the product should not turn birds into stats. D-2 permits it only in an out-of-band export for data portability and legal access, with no preview, parse, or import.

- **Mood model and daily-ish reset:** D-3 resolves reset as "circadian relaxation on the server," so opening a tab never touches mood and there is "no snapping." §7.6 also has no absence term, keeping mood from punishing time away.

- **Attunement state:** D-4 introduces attunement to model "less often is what's been observed" without lowering traits, making birds wary, dimming plumage, lowering call rate, or punishing absence. Its floor means birds never stop greeting altogether.

- **Procedural voice and greeting signature:** §7.10 says greeting signatures make the same bird greet recognizably across visits while different birds greet differently. §10.3 keeps voice identity bounded so "Pip stays recognizable across mood and years."

- **Server simulation tick, drift, mood, behavior plan, and notebook observer:** The plan makes the tick canonical so clients do not write simulation state, events apply exactly once, and every aviary "runs whether or not any client is connected." §7.1 also makes cadence a "cost knob, not a behavior knob."

- **Return-greeting:** §7.10 shapes greetings by absence continuously and always has exactly one bird notice, with no stored variants. D-25 adds refractory rules so rapid tab-flicking does not make greetings "a tic that reads as canned."

- **Listen-in:** The plan lets a user focus on one bird without muting the aviary: §10.5 says other birds are "never muted." Credit intersects listen-in with presence so a left-running listen-in stops counting when the person walks away.

- **Offers:** D-8 says offers go to the aviary, not a clicked bird, to keep the offer "a gesture rather than a button aimed at a bird" and keep the UI from feeling gamey. §7.11 uses canonical state so outcomes, cooldowns, mood, and plan overlays stay consistent.

- **Settle:** D-14 makes settled lighting local because "Settle is a goodbye from one person at one screen," while the mood effect is canonical. D-20 adds keyboard undo for parity.

- **Field notebook:** INV-12 and §7.14 make the notebook describe aviary-internal facts, never the user's behavior. The sparsity governor, novelty penalty, and forbidden detectors avoid streaks, fatigue, and visit-frequency narratives.

- **Presence accounting:** INV-5 and §7.3 prevent the "tab open all night" failure, double-crediting overlapping devices, and visitor traffic. D-7 explicitly chooses the safe direction of under-counting over over-counting.

- **Procedural audio in an AudioWorklet:** INV-8 refuses recorded audio. §10.1 chooses one AudioContext and one worklet for precise scheduling, bounded queues, and no per-call node allocation, satisfying the memory rule.

- **Graceful silence with captions fallback:** §10.7 keeps the aviary usable when WebAudio fails while preserving INV-8: there is "no recorded-audio path under any condition." Captions turn on by default for that device, and the only mention is in accessibility settings.

- **Magic-link sign-in:** §13.1 uses one sign-up/sign-in flow and identical responses to avoid account enumeration. Tokens live in fragments and are POSTed so mail scanners that prefetch GET links cannot burn them.

- **Per-device sessions with revocation:** §13.2 sets a long session lifetime so "leave for two weeks and come back" does not hit a sign-in wall. The session list uses day-granular activity, "just enough to spot an unfamiliar device, never a visit history."

- **Email change with verification:** D-21 keeps the old email working until the new one is verified because the new address may be read on another device, and this avoids lockout.

- **JSON export delivered as an emailed link:** D-2 and §13.5 frame export as data portability and legal access. The product never previews or imports it so the raw vectors do not become a surface and clients never gain a way to write personality.

- **Deletion that is soft for 30 days, then hard:** §13.6 gives the user a restore window through "I changed my mind," keeps the tick running so a restored aviary has carried on, then verifies zero rows and no tombstone after hard delete.

- **Multi-device sync:** §8.1 says sync comes from the canonical server and write-ownership matrix. §8.3 avoids last-write-wins because devices post events and the tick adds deltas in sequence order; no device writes the vector.

- **Visits through email invitations, visitor passes, revocation, expiry, and visit log:** D-10 combines a single-use link with a 14-day browser-bound pass to avoid a permanent visitor list. §6.6 uses expiry, revocation, and a log for control and transparency while visitors remain read-only.

- **Visit notifications opt-in and off by default:** The plan keeps notifications matter-of-fact, limited to at most one per invite per day, and off by default so the aviary does not reach for the host. This aligns with INV-9 and the email allowlist.

- **Accessibility surfaces in v1:** INV-14 says these ship at full product quality. §16.6 gates launch on screen-reader, keyboard, deaf/HoH, and vestibular testing so narration, captions, keyboard, and reduced motion feel like the aviary rather than fallbacks.

- **Unsupported-browser page:** §2.1 and §9.10 make unsupported browsers a matter-of-fact system surface. Missing WebAudio is not unsupported because the intended fallback is silence plus captions.

- **Refusal of gamification:** §2.2 removes achievements, streaks, levels, scores, badges, visit calendars, and counters, then backs the refusal with no counting columns, no badge prop, age-only newcomer unlocks, and lexicon lints.

- **Refusal of Tamagotchi mechanics:** §2.2 refuses death, hunger, distress, and decaying meters. The rationale is expressed structurally: monotonic drift, no negative-valence absence input, no distress/sick/starving poses, offers without sustenance state, and no bird removal.

- **Refusal of social network, public discovery, leaderboards, and show-off mode:** §2.2 prevents listing or looking up aviaries, keeps aviary IDs out of URLs, uses opaque visit tokens, and has no cross-account `sim` pipeline, making leaderboards costly to add.

- **Refusal of native apps:** §2.2 says v1 avoids "native-driven protocol compromises": web cookie auth, no deep links, and no push tokens.

- **Refusal of payments, shared or multi-aviary accounts, and customizable scenes:** NOT RECOVERABLE FROM PLAN

- **English-only v1:** D-17 says the naturalist grammar is "language-specific craft."

- **SSO and passwords deferred:** NOT RECOVERABLE FROM PLAN

- **Background listening deferred:** D-12 says hidden-tab audio would be hurt by background timer throttling, battery cost, and the product stance of "nothing to see."

- **Canvas 2D instead of WebGL:** D-22 says around 200 draw calls for seven birds is easy at 60 fps on old laptops, with a smaller bundle, no context-loss handling, and one code path.

- **HTTP pull instead of WebSocket or SSE:** D-23 says the pull model is enough because action plans cover gaps between pulls, and it has "far less operational surface."

- **Releasing a bird deferred:** D-26 refuses removal in v1 because bird identity is permanent and removal is "custodial-flavored."

### Architecture, data, and API

- **Modular monolith plus separate tick and jobs:** §4.1 says a modular monolith fits a team of about 12. The tick is separate because it has a different scaling profile and is the sole writer.

- **TypeScript everywhere and shared packages:** §4.4 uses TypeScript for shared contracts, PRNG, plan evaluator, and voice grammar across server and client.

- **Vite, manual chunking, and `size-limit`:** §4.4 and §14.2 use code-splitting and CI size checks to keep the boot path under budget.

- **Preact with signals for chrome:** §4.4 chooses it because the runtime is about 5 KB and adequate for accessible widgets.

- **PostgreSQL with primary and replicas:** §4.4 uses transactions for exactly-once tick application and column grants for INV-1.

- **pg-boss for jobs:** §4.4 says job volume is low and this avoids adding another system.

- **S3 exports with SSE-KMS and lifecycle deletion:** §4.4 and §13.5 use S3 for export links and automatic deletion after seven days.

- **Transactional email with open and click tracking disabled:** §4.4 says tracking pixels are behavioral telemetry and click rewriting breaks single-use links.

- **Telemetry collector in a separate host and project:** §4.1 and §13.7 prevent telemetry systems from having credentials or network access to the simulation database.

- **Schemas split into `acct`, `sim`, and `social`:** §5.1 keeps identity and PII in `acct`, simulation and private interaction data in `sim`, and visits in `social`, with `sim` and `social` using UUIDs and no email.

- **Database roles and write grants:** §5.1 and §5.5 enforce sole writer rules and prevent conflict by construction. The plan explicitly says no field has two writers of different kinds, so there is no last-write-wins race on personality.

- **Personality seeding and per-bird ceilings:** §5.3 starts traits low enough to leave "real room for expressive growth," and uses per-bird ceilings so long-lived birds do not all converge on the same maximum.

- **Retention windows for events, logs, facts, exports, tokens, and passes:** D-24 and §5.6 keep enough history for outage catch-up and debugging while minimizing private history. The event log is not used to rebuild the vector.

- **API conventions: JSON, cookie scopes, CSRF, idempotency, mapped errors, and rate limits:** §6.1 makes host and visitor scopes distinct, avoids server-sent user-facing prose for system errors, and keys rate limits without logging raw emails.

- **Edge boot and inline snapshot:** §6.3 and §9.1 stream the head and snapshot to avoid an extra round trip. If the regional call is slow, the quiet field covers the gap without a spinner.

- **Snapshot contract with no trait fields:** §6.3 includes only derived, quantized render parameters and demeanor hints. Contract tests fail if any trait key appears, upholding INV-3.

- **Event submission with per-account sequencing:** §6.4 and §8.2 assign sequence numbers under a row lock so commit order equals sequence order and duplicates are idempotent.

- **Synchronous offer resolution:** §6.5 leaves time for p95 round trips while the client starts local placement immediately. The server writes the offer event, cooldowns, and overlay in one transaction for consistency.

- **Visit tokens in URL fragments:** §6.6 says fragments keep tokens out of server logs and Referer headers, and link scanners that do not run JavaScript cannot consume them.

- **System notices in the top-bar region:** §6.7 keeps errors matter-of-fact, persistent until resolved or dismissed, never toasts, and never triggered by arrival.

### Simulation engine

- **Tick scheduling with phase offsets and partition leases:** §7.1 spreads load so ticks do not bunch at :00, and fencing prevents a zombie worker from committing after lease expiry.

- **Write-on-change and closed-form decay:** §7.1 still evaluates every aviary every minute but writes only changed rows, reducing cost without changing behavior.

- **dt-invariance:** §7.1 makes 30 s, 60 s, 120 s ticks or catch-up after outage give the same trajectories within tolerance, so cadence is a cost knob.

- **Presence union across devices:** §7.3 makes laptop and phone together worth one device for presence, never double, matching the active-attention principle.

- **No strong anti-cheat against faked presence:** §7.3 says fake presence affects only the user's own birds, and with no leaderboard there is no reason to over-police it.

- **Drift saturation and low-pass filter:** §7.4 uses diminishing returns, a 4-day EMA, per-day caps, and slowing near ceilings to prevent "Tamagotchi-speed drift" for heavy users while still making three-week change visible.

- **Visible trait mappings and JND studies:** §7.4 defines visibility through behavioral mappings and 2-alternative forced-choice studies, so "visible" is calibrated rather than guessed.

- **Mood components and transitions:** §7.6 blends circadian prior, personality, recent interaction, ambient weather, and contagion so birds feel alive while absence never enters mood logits.

- **Alarm sources ambient only:** §7.6 makes wind gusts or passing shadows the source of alarm, never the user, preserving the non-punitive relationship.

- **Committed action queue:** §7.7 keeps already committed actions immutable and only amends actions at least five seconds in the future, so devices agree about the present and birds do not visibly jump.

- **Call scheduling and onset-collision avoidance:** §7.7 delays ordinary overlapping calls because real birds avoid acoustic overlap and it preserves recognizability.

- **Bird-to-bird interaction and affinity:** §7.8 gives birds internal social behavior while affinity is bird-to-bird only, never user-derived, and never exposed.

- **After-rain phase:** §7.9 gives daily short visitors a chance to notice weather roughly every three weeks without making weather happen for the viewer's benefit.

- **Greeting resolver sampling:** §7.10 samples rather than always choosing the top bird so the bolder bird usually greets first "but not always," avoiding canned behavior while always ensuring one bird notices.

- **Offer resolver response types and cooldowns:** §7.11 lets mood, curiosity, boldness, distance, and cooldown shape reactions, while invisible cooldowns prevent timers or disabled game-like UI.

- **Server schedules call descriptors, not audio:** §7.12 makes every device and visitor hear the same bird at the same moment while variation comes from fresh seeds and client synthesis.

- **Newcomer lifecycle in the scene:** D-9 and §7.13 make the wild bird appear on the back-edge perch and add "make room for the thrush" to the menu, so the aviary itself notices, declining costs nothing, and there is no announcement surface.

- **Notebook observer inputs and sparsity governor:** §7.14 uses aviary-internal facts only, with a token bucket and novelty penalty, so active users do not get more entries than regular users and the notebook does not become a behavior log.

- **Determinism, shadow mode, and species revs:** §7.15 makes ticks reproducible, tests new sim versions as aggregate histograms with no account dimension, and keeps species revisions immutable unless recognizability and visual identity checks pass.

### Sync, frontend, audio, and voice

- **Never-go-backwards snapshots:** §8.4 has clients ignore older `tick` or `overlayVersion`, preventing lagging replicas from making birds jump back.

- **Local overlays for offers and greetings:** §8.4 keeps read-your-writes behavior until a snapshot catches up, so local reactions do not disappear because of replica lag.

- **Plan continuity and reconciliation:** §8.5 keeps the next five seconds from the existing plan and reconciles with hops, flights, turns, or reduced-motion cross-fades, so visible birds never teleport.

- **Offline outbox and idle-only continuation:** §8.7 tolerates disconnection while avoiding invented macro actions, greetings, or perch changes. Fresh state reconciles naturally when it returns.

- **Quiet field boot path:** §9.1 gives a calm visual while state loads, never a spinner, and after 15 seconds uses a system notice. The rejected server-rendered SVG first frame would risk a visible discontinuity at canvas hand-off.

- **Scene composition and ambient parallax:** §9.2 uses layered canvas composition. Parallax is ambient breathing rather than pointer-driven because pointer-driven parallax "reads as a UI trick."

- **No per-frame allocation and adaptive render scale:** §9.2 protects long-session memory and frame-time budgets, with only aggregate counters for scale changes.

- **Bird rigs and individual markings:** §9.3 keeps same-species birds visually distinct, which matters because seven birds from six species means at least one repeat.

- **No fixed-period micro-motion loops:** §9.3 uses seeded distributions and noise to avoid "strobing micro-animation," with flash limits for WCAG.

- **Macro transitions for hops, flights, weather, light, and settle:** §9.4 gives continuous transitions, clamps arcs to the viewport, and keeps adopted birds in frame.

- **Top bar fade and non-fade conditions:** §9.6 lets chrome recede but keeps it visible when focus, menus, panels, notices, or "Keep the top bar visible" require it.

- **Panels and field notebook drawer:** §9.7 keeps the aviary rendering behind panels and drops to 30 fps rather than pausing when a phone panel covers most of the scene, preserving the living aviary.

- **Pointer and touch interaction:** §9.8 expands bird hit areas to 44x44 px and uses audible listen-in feedback rather than chrome, keeping interaction direct and quiet.

- **Reduced-motion renderer:** §9.9 is a different renderer strategy over the same plan. It removes drift, particles, falling rain, and motion arcs while keeping calls, captions, narration, drift, mood, and notebook identical.

- **Audio graph spatial cues:** §10.1 uses pan, gain, low-pass, and reverb so the ear can read perch zones.

- **Synthesis from motif templates:** §10.2 uses sinusoidal and FM models with noise components because bird vocalizations are well served by them, and sound design auditions against descriptions rather than shipped recordings.

- **Call grammar voice print:** §10.3 fixes base pitch, timbre, repertoire, syntax, signature motif, rhythm, and interval signature so each bird is recognizable by ear.

- **Bounded mood transforms:** §10.3 allows mood color while keeping identity features fixed; vocal-frequency drift changes how often a bird calls, "never its identity."

- **Never-twice call guard:** §10.3 resamples on recent call-parameter collisions to prove calls do not repeat, even though continuous jitter makes collisions rare.

- **Chorus mixing:** §10.4 keeps calls individually shaped, prevents clipping, and uses register spacing so overlapping chorus calls remain separable rather than becoming a loop.

- **Listen-in mix:** §10.5 brings one bird closer while others soften slowly and are never muted, avoiding hard cuts and preserving the aviary.

- **Autoplay handling without a prompt:** D-13 and §10.6 avoid a "tap for sound" prompt because it would be announcement chrome; audio fades in on first activation when needed.

- **Hidden-tab audio fade and suspend:** D-12 and §10.6 fade out, suspend, and reschedule from the current plan because background timer throttling would break call scheduling and suspension saves battery.

- **Sound off setting:** §10.6 mutes the master bus but keeps synthesis running so captions stay accurate and perceivability can be recorded.

- **Typed phrase grammar:** D-16 and §11.1 use authored, deterministic corpora because per-bird state cannot go to a third party, voice must be controlled and lint-checked, and outputs should be reproducible.

- **System string catalogue:** §11.3 separates sign-in, errors, settings, sessions, visits, export, deletion, and accessibility copy into plain English with normal capitalization, preserving the surface split.

- **Voice, lexicon, notebook, and system lints:** §11.4 prevents naturalist copy from saying "you," using digits or exclamation, invoking streak/level/badge/reward language, or letting code identifiers drift into banned concepts.

### Accessibility, privacy, observability, and rollout

- **Accessible structure and overlay buttons:** §12.1 gives the canvas semantic access through one transparent button per bird, short naturalist names, roving tabindex, and no mood or trait exposure.

- **Polite screen-reader narration:** §12.2 narrates salient light, weather, birds, and happenings as observations, with cadences and queues that prevent flooding and avoid talking over dialogs.

- **Call captions:** §12.3 generates captions from what was actually realized and played, so they match the audio. Placement, stacking, contrast, and chorus folding keep them readable.

- **Keyboard model:** §12.4 gives keyboard equivalents for bird focus, listen-in, settle undo, menus, notebook, and gestures while allowing single-key shortcuts to be disabled for WCAG 2.1.4.

- **Focus and contrast system:** §12.5 checks focus rings, text, plates, light phases, weather states, forced colors, and target size so accessibility survives the visual variability of the aviary.

- **Email encryption, HMAC lookup, and synthetic IDs:** §13.4 stores email only as ciphertext and lookup HMAC on the account row, while every other reference is the account UUID, containing PII.

- **No third-party scripts or per-bird data leaving:** §13.7 blocks analytics, ads, session replay, tag managers, warehouses, recommenders, models, and ETL from per-bird interaction state.

- **Support tooling limits:** §13.8 lets support see account status, sessions, error codes, and deletion state, but never personality values, mood history, notebook text, or events.

- **Outbound email allowlist:** §13.9 makes re-engagement, digests, "your bird misses you," and aviary-state emails impossible without a reviewed schema change.

- **Performance budgets:** §14.1 turns first-bird, memory, frame rate, tick, API, and delivery expectations into targets, hard limits, and enforcement, directly tying the experience to measurable gates.

- **Synthetic checks:** §14.3 uses dedicated synthetic accounts with no real user data to monitor geography, devices, warm and cold cache, and first-bird performance.

- **Aggregate-only RUM:** §14.4 folds client measurements into fixed histogram buckets without cookies, identifiers, account, session, device ID, or page-view ID.

- **Things deliberately not measured:** §14.6 avoids engagement analytics, accessibility-preference adoption, engine behavior, and cross-account `sim` aggregates because those would violate the privacy boundary. Calibration uses synthetic harnesses and lab studies instead.

- **Security controls:** §15 gives CSP, HSTS, token hashing, fragment links, host cookies, rate limits, visitor scopes, dependency audits, KMS rotation, and launch-gate security review to protect auth, visits, CSRF, and deletion completeness.

- **Verification strategy:** §16 turns invariants into property tests, contract tests, chaos tests, accessibility research, lints, scanners, and first-frame e2e so the principles remain enforceable over implementation changes.

- **Delivery milestones:** §17 sequences guard lints, grants, schemas, tick, calibration, living slice, engine/audio, voice/accessibility, lifecycle, hardening, alpha, beta, and GA so the load-bearing constraints exist before polish and scale.

- **Staff alpha and closed beta:** §18.1 uses lived staff aviaries and an 8-week beta so people experience the one-week and three-week drift horizons. Feedback comes only through user-started channels because in-product surveys and feedback emails would be "the product reaching for the user."

- **Bird-cap ramp:** §18.3 gates three, five, and seven birds on recognizability, performance, and layout before real aviaries reach those ages. Production aviaries are never aged artificially because that would break the age rule.

- **Feature flags, drift multiplier, and drift freeze:** §18.4 lets the team slow or freeze drift without reversing it, losing events, or moving traits backward, preserving monotonicity during calibration bugs.

- **Production instrumentation from day one:** §18.5 makes latency, lag, integrity, first-bird, audio failure, email delivery, and PII scanners live before real users so operational risks surface early.

- **Touch presence default in open questions:** §20 defaults to no `pointerdown` counting because D-7's default treats over-counting as the worse failure, while leaving a beta-reviewed one-line change open.

- **Visitor notebook default in open questions:** §20 defaults to no visitor notebook access, consistent with D-10's statement that "The notebook is the host's record."

- **Business metrics open question:** §20 says leadership should accept that v1 reports only total account count and aggregate load, "as a direct result of the privacy boundary."

- **Species working set confirmation:** NOT RECOVERABLE FROM PLAN
