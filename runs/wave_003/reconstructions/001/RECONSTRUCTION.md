## System-level intent

- **Felt aliveness is a protected product property.** The introduction says the work is ordered so "felt aliveness" is built first and is "hardest to erode later." It shows up in I10, where "the aviary's first rendered frame is the aviary in motion"; in the idle micro-motion rule that birds are "never paused-looking"; and in greetings, calls, weather, notebook observations, and narration being driven from the same living scene.

- **The server is the source of truth; the client is a renderer.** The plan repeatedly names a "server-authoritative simulation." I1 says only the server simulation tick writes personality vectors. §2.3 says the server decides "what is true" and the client decides "how it looks and sounds." §6.1 says there is "one canonical record per aviary, one writer (the tick), and many readers."

- **Personality maturation is monotonic, not punitive.** I2 requires drift to be "additive, non-negative, and bounded." §5.3 makes drift "monotonic by construction." §5.4 separates attunement from personality so neglected birds can greet less often without traits going down. The v1 exclusions also reject "hunger, death, distress, decaying happiness meters."

- **Presence must be honest and narrow.** I3 defines presence exactly as visible, focused, and recently active. §6.4 clamps intervals on the server and unions devices so "two devices watching at once count as one person watching." §7.2 frames this as "Idle attention is interaction" while keeping it invisible to the user.

- **Bird identity is permanent.** I4 says `bird.id`, voiceprint, and appearance seed "never change." The same intent appears in immutable columns, migration continuity tests, species version pinning, and the rule that existing birds stay pinned "forever" to their shipped species version.

- **Privacy and data minimization are design constraints, not add-ons.** I5 says personality values are never sent to clients or rendered. I8 keeps email in "exactly one place." I9 forbids per-bird and per-account interaction state from aggregate telemetry, analytics, or ML. §7.7 rejects a third-party LLM because it "would break the privacy commitment."

- **The product voice is quiet, matter-of-fact, and non-gamified.** I6 bans "toasts, banners, welcome text, streaks, counters, badges, confetti" and other announcement surfaces. System surfaces use "matter-of-fact voice"; bird-facing prose uses naturalist observations. New birds are "arrive" and "noticed, not announced," not "unlocked" or "available."

- **Accessibility is first-class and release-gating.** I11 says accessibility surfaces "ship with v1 and gate every release." §13.1 states "Accessibility is not a milestone. It's a gate on M2, M5, M6, and M7 exit." The accessibility section treats narration, captions, reduced motion, keyboard, contrast, and screen-reader passes as core surfaces.

- **Audio must be procedural, recognizable, and never file-backed.** I7 bans recorded audio. §8.3 makes each call procedural but anchored by a voiceprint and "signature syllable." §8.8 tests uniqueness and recognizability, while the risks section names "uncanny or synthy calls" as a major affective risk.

- **Performance and memory are part of the experience.** I10 bans spinners and requires the first frame to be motion. §11.1 budgets first bird visibility, frame time, audio CPU, and "no growth." §9.8 enforces pools and frame-loop allocation discipline so the aviary can stay open without degrading.

- **Interactions should be diegetic gestures, not control panels.** Listen-in has "no highlight, halo, or label." Offers are automatically placed so they remain "gestures, not targeting UIs." The top bar fades away. Tooltips appear "in the bar only," never inside the scene.

- **Observation is sparse and bird-centered.** The notebook writer uses a token bucket to preserve "sparsity." Its fact schema has "no user-behavior fields." Narration and notebook entries avoid state lists, mood labels, numbers, and second person; they describe what birds do.

- **Cross-device consistency falls out of shared snapshots.** §6.1 says multi-device sync "is not a feature to build" because devices read the same canonical row. Offers use overlays so "every device and visitor pulling a snapshot sees the same reaction." Weather and server-scheduled social events live in the snapshot for the same reason.

- **Calibration and rollout are conservative.** §5.10 builds the harness "before any UI." Dogfood must run long enough to cross the "3-week visible-drift horizon." `max_birds_enabled`, capacity gates, and kill switches are hidden operational safety nets, while users see no gate or announcement.

## Per-feature whys

### 0. Invariants and scope

- **Only the server simulation tick writes personality vectors:** The plan protects server authority and monotonicity by making personality writes impossible from clients and API handlers; it enforces this with role grants, code ownership, and an integration test.

- **Personality drift is additive, non-negative, and bounded:** The why is to make drift monotonic, avoid decreases, and prevent runaway maturation. The negative clamp is "belt-and-braces" and must stay at zero.

- **Presence equals visible, focused, and recently active:** The why is "honest presence." Background, unfocused, inactive, visitor, or revoked-session time must not inflate drift.

- **Bird identity permanence:** Permanent ids, voiceprints, and appearance seeds keep birds from being "regenerated, swapped, or re-seeded"; migration continuity protects the user's relationship with the same bird.

- **Personality values never go to clients or telemetry:** The plan treats trait secrecy as part of the privacy and product-voice commitment. Derived render parameters are allowed only because "no field maps one-to-one onto a trait."

- **No announcement surfaces:** The why is to avoid welcome surfaces, gamification, and product noise. The plan enforces this by having no toast/snackbar component, copy lint, and review checks.

- **Procedural client-side calls with no recorded audio:** The why is to satisfy the "no recorded audio" invariant while still providing distinctive, varied bird voices.

- **Email in encrypted account and invite records only:** The plan uses synthetic UUIDs everywhere else to prevent PII from leaking into logs, metrics, partitions, or joins.

- **No per-bird or per-account interaction state in aggregate telemetry:** The why is privacy. The plan explicitly avoids analytics and ML paths that could observe how individual birds or accounts are interacted with.

- **First rendered frame is the aviary in motion:** The plan makes this the antidote to spinners, entry animations, and static loading states; the first experience must be the living aviary.

- **Accessibility surfaces ship with v1:** The why is that narration, captions, reduced motion, keyboard navigation, and contrast are not follow-ups; they gate release.

- **Browser support for the last two major versions of Chrome, Safari, Firefox, and Edge:** NOT RECOVERABLE FROM PLAN

- **Native apps excluded from v1:** The plan's reason is that v1 is browser-only and "we also don't design protocols around native constraints."

- **No gamification:** The rationale is to prevent streaks, counters, scores, achievements, and visit calendars from becoming either UI or latent statistics "where they could be surfaced."

- **No Tamagotchi mechanics:** The plan wants neglect to produce "quietness and nothing else," not hunger, death, distress, or decaying meters.

- **No social-network surfaces:** The plan rejects profiles, feeds, follows, comments, chat, leaderboards, co-presence, and "show-off" rendering to keep visits and sharing from becoming social-network mechanics.

- **Unsupported-browser page instead of compatibility paths:** NOT RECOVERABLE FROM PLAN

### 2. Architecture

- **TypeScript everywhere:** The explicit reason is shared code. `sim-core`, lighting curves, prose, and species data are consumed by server and client, and "one implementation prevents drift between them."

- **Pure deterministic `sim-core`:** The why is testability, replay, and boundary clarity. It owns drift, mood, perch selection, weather, social events, greeting and offer planning, but never touches DB or network.

- **Tick workers as sole writer of aviary state:** This preserves the single-writer model and keeps client input from directly changing canonical simulation state.

- **Snapshot as the client/server boundary:** The plan says the server decides "what is true" and the client decides "how it looks and sounds." Anything requiring device agreement goes in the snapshot; local liveliness stays client-side.

- **Derived render parameters instead of traits:** The reason is I5. Parameters like call rate, scan rate, fluff, and plumage colors are quantized blends so no value exposes a trait directly.

- **Redis latest-snapshot cache:** Redis improves read speed, but the plan says it must "never hold the only copy of anything canonical"; the API can rebuild from Postgres.

- **Postgres not replicated to analytics warehouse:** The why is privacy isolation: the telemetry pipeline has "NO access" to simulation schemas and no network route to the simulation DB.

- **`prose` package with no third-party LLM:** The rationale is privacy and voice control. Sending per-bird facts out would break the privacy commitment, and a grammar can be linted.

- **Versioned `species` package:** The reason is identity permanence. Existing birds never get re-skinned; new versions affect new birds only.

- **Virtual clock in CI and staging:** The why is calibration and test speed: weeks can be compressed into minutes. In prod it is compiled out.

### 3. Data model

- **Synthetic account UUID and encrypted email:** The synthetic id is the only identifier used elsewhere; encrypted email plus lookup HMAC allows sign-in without exposing email.

- **`email_lookup_hmac`:** The plan says it is needed because email is encrypted and auth still has to find the account at sign-in. Schema lint prevents any other reference to it.

- **No persisted IP addresses:** Rate limiting uses `HMAC(ip)` buckets with a one-hour TTL so IPs are not stored.

- **Account-wide `ON DELETE CASCADE`:** The reason is that hard delete can be "a single delete plus storage cleanup."

- **Immutable bird id, voiceprint, and appearance seed:** These protect bird identity and prevent migrations or edits from mutating the bird the user knows.

- **`bird_personality` written only by `tick_role`:** This enforces I1 and prevents API/client code paths from mutating traits.

- **`bird_personality_daily` backup:** The plan limits it to backup and integrity checks, with 90-day retention, so product code never reads it as a behavior source.

- **Append-only interaction events:** The why is to drive the user's own simulation, support idempotency, and give short-horizon incident debugging without becoming analytics.

- **Event retention deleted 14 days after consumption:** The plan says events exist for the user's simulation and short incident debugging "and nothing else."

- **Notebook entries immutable and writer-only:** The rationale is that the field notebook is read-only, auto-generated observation prose; the API role can select but not mutate.

- **Per-device mute and reduced-motion override in localStorage:** §15 D9 explains that captions, narration, and reduced-motion preference follow the user, while mute is "situational to a device."

### 4. API surface

- **Shared zod request validation:** NOT RECOVERABLE FROM PLAN

- **Errors as machine codes plus client-side matter-of-fact copy:** The matter-of-fact system voice is articulated, but the specific reason for keeping copy out of responses is NOT RECOVERABLE FROM PLAN.

- **Magic-link sign-in as the auth method:** NOT RECOVERABLE FROM PLAN

- **`POST /auth/link` always returns `202`:** The stated reason is "no enumeration" of whether an email exists.

- **Unknown email account created on link consumption, not request:** NOT RECOVERABLE FROM PLAN

- **`GET /auth/verify` never consumes the token:** The why is explicit: "email-scanner prefetches can't burn links."

- **Per-device session revocation:** The endpoint provides device control, and revocation takes effect within the edge session cache TTL.

- **Old email works until new email verifies:** NOT RECOVERABLE FROM PLAN

- **Account export:** §15 D6 says export is a "data-rights artifact" and data portability is a user right, even though product UI never renders vectors.

- **Soft delete with restore and 30-day hard delete:** §15 D17 says a restored aviary should have continued "like any other"; after 30 days data is hard-deleted and the DEK destroyed.

- **Deletion banner as a system surface:** The plan distinguishes it from an aviary announcement because it is account-status copy, not a product celebration or greeting.

- **Bird rename endpoint:** The why is that names "aren't personality," so API-side writes are allowed; optimistic concurrency handles two devices editing at once.

- **Snapshot route can attach a greeting and write a greeting event:** The greeting event lets the notebook observe order, for example "pip greeted first."

- **`/sync` event batching and idempotency:** The rationale is reliable client outbox flushing; `client_event_id` makes retries safe.

- **Offer planner is read-only on canonical state:** This keeps API request handling from writing mood or personality while still returning immediate choreography.

- **Offer reaction overlay in Redis:** The why is that "every device and visitor pulling a snapshot sees the same reaction" before the next tick applies consequences.

- **Offer cooldown invisible in UI:** The rationale is quiet product voice. Birds in cooldown glance or ignore; there is no error, timer, or disabled state.

- **Adoption through a `SECURITY DEFINER` function:** The function is the only non-tick insert path into `bird_personality`, preserving the tick-write invariant for updates.

- **Visit invite email with no aviary data:** The why is matter-of-fact privacy; the template contains no user-authored or aviary content.

- **Visit link binds to one browser on first use:** §15 D14 says this lets "one-time link" and "revoke active invites" both hold.

- **Visitor permission set is snapshot reads only:** The reason is that visitors are read-only; they cannot sync, offer, access notebook/settings, or instantiate presence.

- **Opt-in visit notification, off by default:** The plan keeps it as the only notification exception, debounced and matter-of-fact, to avoid announcement creep.

- **Trait-free snapshot under 6 KB gzipped:** The size supports first-frame performance; trait-free derived fields enforce I5.

- **Visitor mode drops greeting but keeps names:** §15 D10 says visitors see "exactly as it is," and names are needed because narration uses them.

### 5. Simulation engine

- **One-minute tick cadence with fixed phase per aviary:** The fixed phase spreads load evenly across the minute.

- **4,096 shard leases:** The why is scalable worker ownership with automatic rebalancing.

- **`tick_seq` fencing token:** The reason is to stop a stalled worker that lost its lease from committing.

- **`dt`-aware catch-up and overnight step:** The aviary can recover after outages "correct, just coarser," without resetting state.

- **Tick SLOs and quarantine:** Quarantine alerts on poison aviaries but "never resets state."

- **Personality seeds in the 0.20-0.45 range:** The seeds define starting character while leaving "headroom for expression."

- **Per-bird ceilings:** The plan says ceilings keep birds distinct as they mature and avoid the "homogenization risk."

- **Two-stage low-pass drift:** The drive decays slowly so drift can continue after the user leaves, matching the idea that personality drifts during absence from prior inputs.

- **Headroom saturation:** The rationale is that early drift feels responsive and later drift slows; a year-old bird keeps maturing without capping out quickly.

- **Drift calibration table and JND targets:** The why is to make 1-day changes below perception, 3-week changes visible, and heavy use bounded.

- **Persisting vectors by deltas rather than recomputing from log:** The plan keeps product behavior out of the event log and reserves daily copies for recovery.

- **Drift repair by pause and fix-forward:** The reason is that writing lower values would itself be negative drift, which violates the invariant.

- **Attunement separate from personality:** §15 D3 says it is "the only way to satisfy both statements" that traits never decrease and neglected birds greet less often.

- **Attunement floor and quick recovery:** The why is to prevent neglect from reading as punishment; birds "always keep calling and always still notice the user."

- **Mood target distribution from time, weather, interactions, social contagion, and personality:** The plan uses these inputs so visible behavior reads as living context rather than exposed state labels.

- **Soft dawn re-anchor for mood:** §15 D4 says it satisfies "daily-ish" reset without breaking persistence or causing a visible snap.

- **Alarm cap:** The stated reason is to prevent contagion loops.

- **Ten perch slots for at most seven birds plus one wanderer:** The plan says this means "there's always somewhere to go."

- **Perch changes scheduled at least five seconds in the future:** This lets every client play the movement instead of disagreeing or teleporting.

- **Ninety-second choreography horizon with no revision of started actions:** The why is continuity; actions already playing keep playing.

- **Weather in the snapshot:** The reason is that every device and visitor sees the same rain.

- **Server chorus windows, client call-and-response:** Chorus windows need shared agreement; individual call-and-response stays local because it "doesn't need cross-device agreement" and feels immediate.

- **Six species as one coherent habitat:** The plan states this directly as the design rationale for the species pool.

- **Starter pair with far-apart registers and no nightjar:** The rationale is first-day clarity: voiceprints are distinct, and excluding the nightjar avoids making the first encounter quieter than it should be.

- **Individual voiceprint separation:** The why is recognizability, especially when two birds of the same species are in one aviary.

- **New birds available by aviary age only:** The rationale is I6, "no gamification"; no input except `created_at` affects availability.

- **Wanderer arrival and adoption through focusing the bird:** The plan says birds "arrive" rather than being acquired, and the user notices the bird instead of receiving a badge, prompt, or indicator.

- **Simulation harness built before UI:** The rationale is calibration. It asserts drift, mood, greeting, notebook sparsity, I2, and I3 before product surfaces can obscure errors.

### 6. Sync model

- **One canonical record, one writer, many readers:** The why is that multi-device sync "falls out" of both devices reading the same row.

- **Per-aviary serialized sequence allocator:** The plan explains that plain sequence gaps could let the tick skip late-committing events; row locking makes commit order equal sequence order.

- **IndexedDB client outbox:** The rationale is durable retry across sync intervals, transitions, and pagehide.

- **Dropping very late intervals and limiting stale effects to drift:** A stale offer should not make a bird content now, but older events can still count toward drift.

- **Presence interval server clamps:** The why is to reject impossible or inflated presence, visitor intervals, and revoked sessions.

- **Presence union across devices:** The plan states that two devices watching at once count as one person watching.

- **Immediate pull triggers on visibility, bfcache, online, and long-frame gap:** The rationale is catch-up after hidden tabs, offline periods, and laptop suspend.

- **Client reconciliation without teleporting:** New snapshots merge schedules and blend mood changes so birds move normally even after unseen changes.

- **Lighting computed locally from clock plus timezone:** The plan says lighting never waits for the network.

- **Timezone hysteresis:** The reason is travel support without flip-flopping between devices in different zones; both devices show the same light.

- **Conflict handling rules:** Each rule is there to prevent drift corruption: stale devices never push state, tick races fence, offers re-check cooldown in sequence order, and name edits use optimistic concurrency.

- **System strip for session/offline/load errors:** The strip is matter-of-fact, persistent until the condition clears, and "not a toast."

### 7. Interactions

- **Presence activity events include pointermove, pointerdown, and keydown:** §15 D2 says this stays faithful to the intent while making touch devices workable.

- **Presence window `W` defaults to four minutes:** The plan says it leans longer because the PRD asks; mobile under-counting is a known risk to calibrate.

- **Visitor mode does not instantiate presence:** The reason is that visitors never affect host drift or state.

- **Settle ends and suppresses presence:** The plan keeps "settle = presence ends" true until explicit re-engagement.

- **Idle attention as interaction:** The rationale is that watching should affect boldness, attunement, curiosity, and drift without counters or indicators.

- **Return-greeting after session start or return from hidden ≥15 seconds:** Shorter hides do not greet because "every alt-tab would produce a greeting" and constant greetings are canned.

- **Absence-banded greeting magnitude:** The why is proportionality: a short absence can be a glance; a long absence can be re-orientation or a longer call.

- **Greeter selection by boldness, warmth, mood, attunement, and proximity:** The plan wants bolder or warmer birds to tend to go first, wary birds to greet later or not, and night silence to be honest.

- **Stable greeting style with procedural variation:** The same bird greets the same way across visits, but timing, gaze, call motif, and hop distance vary so instances are not identical.

- **No return text:** The greeting is the only welcome surface; "there is no text anywhere on return."

- **Listen-in auto-ends on presence loss:** The reason is that someone who has walked away should not keep inflating listen-in.

- **Listen-in visual response is subtle and diegetic:** The plan explicitly rejects highlight, halo, and label so focus stays inside the aviary.

- **Offer menu with seed, song fragment, and still pool:** The song fragment choices play softly on focus and hover "since there is no text catalog."

- **Automatic offer placement:** §15 D5 and D11 say this keeps offers as gestures, not targeting UIs or manual arrangement.

- **Local offer visual plus server reaction choreography:** The item appears immediately, while server planning preserves authoritative shared reactions; local "noticing" covers high RTT.

- **Per-bird offer cooldown invisible:** The plan keeps the UI from showing timers, disabled states, or errors; a bird in cooldown still gives a low-interest reaction.

- **Notebook observation extraction:** The facts are about birds and moments, never direct trait values or user behavior.

- **Notebook sparsity controls:** Token bucket, spacing, novelty, and weekly caps preserve sparse observation even for very active users.

- **Notebook fact schema excludes user-behavior fields:** The plan says writing that the user was present every day is "structurally impossible."

- **Notebook UI as read-only sheet with no edit, delete, or share:** NOT RECOVERABLE FROM PLAN

- **Prose engine as hand-authored constraint grammar:** The rationale is privacy, total control over voice, deterministic seeding, phrase-memory, and lint validation.

- **Prose output lint:** The why is to preserve lowercase naturalist voice and ban product/gamification language such as "welcome," "streak," "unlock," "badge," and percentages.

### 8. Audio pipeline

- **Single AudioWorklet rendering all voices:** The reason is no per-call node allocation, no GC churn, reused buffers, and no memory growth.

- **Procedural node-graph fallback:** It keeps audio procedural if AudioWorklet fails.

- **Silence with captions when WebAudio is unavailable:** The rationale is accessibility without violating the no-recorded-audio invariant.

- **No "tap to enable sound" overlay:** The plan says an overlay would announce the product; instead the aviary becomes audible mid-motion after a gesture.

- **Species motif libraries and call grammar:** The why is varied, bird-like procedural calls grounded in whistle, chirp, trill, coo, and churr syllables.

- **Signature syllable in at least 70% of calls:** The plan calls this the recognizability anchor.

- **Call uniqueness hash:** The reason is to make "calls never repeat exactly" checkable.

- **Recognizability invariants under mood or drift:** Base pitch, signature syllable, and timbre class keep a bird identifiable; `vocal` changes how often it calls, not what it sounds like.

- **Poisson call scheduler with local response timing:** The plan uses rate parameters from the snapshot while keeping immediate local call-and-response within `responseP`.

- **Captions generated from the resolved call object:** The rationale is that captions always match what was actually synthesized.

- **Listen-in mix and decay:** The focused bird comes closer while other birds are never silent, and disengage "never" returns as a cut.

- **Chorus mixing with pan, depth, compressor, and limiter:** The why is spatial separation and preventing chorus build-up from pumping.

- **Quiet loudness target:** The full aviary is intentionally quiet at about -24 LUFS short-term.

- **Tab hidden fade and suspend:** §15 D8 says this saves battery and avoids timer-throttling glitches; presence does not count while hidden anyway.

- **Offline audio QA tooling:** The plan uses automated renders, loudness checks, uniqueness checks, spectral distance, MFCC classification, and human panels as proxies for quality and recognizability.

### 9. Frontend rendering pipeline

- **Inline boot chunk and inline snapshot:** The rationale is the "< 500 ms path" to first bird, with the current snapshot available immediately.

- **First frame places birds mid-activity:** The plan wants the first visible frame to be the aviary already alive, not a staged intro.

- **Service worker cache with two-minute snapshot limit:** The reason is repeat-visit speed without presenting a stale snapshot as current.

- **No spinner, ever:** The quiet field avoids indeterminate-progress semantics and preserves the non-announcement product voice.

- **Empty-aviary starter fly-in:** NOT RECOVERABLE FROM PLAN

- **Canvas2D renderer:** The plan says the scene is within Canvas2D's budget, Canvas2D has universal support, avoids WebGL blocklist risk, and keeps the boot chunk tiny.

- **Renderer interface escape hatch:** A WebGL2 backend can be added only if performance gates fail.

- **Layered rendering with DOM captions:** DOM captions stay crisp and screen-reader-invisible because narration covers them.

- **Gentle parallax disabled in reduced motion and touch:** The plan grounds the disablement in reduced-motion behavior; no separate product rationale is given for touch.

- **Responsive safe-rect layout:** The reason is explicit: "never cropping a bird" across phones, ultrawides, and flight paths.

- **Idle micro-motion system:** The rationale is felt aliveness; every bird always has breathing and micro-noise and no fully static idle state.

- **Behavior selector with mood-shaped weights:** Behavior conveys mood through scanning, preening, tilting, fluffing, resting, and looking, rather than labels.

- **Anti-strobe rule and eased transitions:** The why is visual comfort and avoiding hard-cut pose changes.

- **Mood, day/night, and settle transitions blended:** The plan avoids visible snaps by easing mood parameters, lighting, and settled presentation.

- **Settle undo window with held outbox event:** The reason is that an undone settle "never reaches the log."

- **Settle as device-local presentation plus server drowsy nudge:** Another device does not dim, but canonical moods can still reflect the drowsy nudge.

- **Top bar item set including settle:** §15 D1 says settle needs a keyboard-reachable home and belongs in the top bar rather than the offer menu.

- **Top bar fade rules:** The rationale is quiet chrome that does not hide focus, popovers, or system conditions.

- **Reduced-motion stillness renderer:** The plan treats it as a "designed surface," not a follow-up, preserving calls, captions, drift, mood, and notebook while removing motion paths and particles.

- **Memory discipline:** Object pools, reused offscreen canvases, no frame-loop allocations, virtualized notebook rows, and bounded caches enforce the "no growth" budget.

### 10. Accessibility surfaces

- **Screen-reader narration from snapshot plus scene events:** The why is immediacy for greetings and reactions without an extra fetch.

- **Narration cadence and subject choice:** The plan prevents narration from becoming a state list; it rotates salient birds and avoids enumerating every bird.

- **Narration avoids mood labels and perch numbers:** Behavior conveys mood, preserving the naturalist voice.

- **Priority narration throttling:** The reason is to keep the screen reader queue from flooding.

- **Visitor narration identical except greetings absent:** The visitor sees the shared aviary state but does not get host return greetings.

- **Captions opt-in and forced on when WebAudio is unavailable:** The rationale is access to call information when sound is absent or unavailable.

- **Caption placement near calling bird:** Captions tie text to the actual bird call while avoiding overlaps.

- **Caption contrast sampling:** The plan samples rendered background luminance so caption text stays at or above 4.5:1.

- **Top-bar and focus contrast tokens:** The two-tone focus ring is verified against every light keyframe.

- **Keyboard focus group with transparent bird buttons:** The rationale is making the canvas accessible while tracking rendered bird positions.

- **Naturalist accessible names:** Names like "pip, a wren, on the front perch" avoid mood labels and numbers.

- **Disable-able single-key shortcuts:** The reason is WCAG 2.1.4 compliance.

- **English-only v1:** §15 D13 says this keeps v1 scope in check because naturalist grammar is language-specific work.

### 11. Performance budgets and observability

- **CI-enforced budgets:** The rationale is to make PRD limits and product feel enforceable: first bird, bundle size, frame time, audio CPU, memory, API latency, and tick latency.

- **RUM metrics with only low-cardinality dimensions:** The why is aggregate-only observability without account, bird, or session identifiers.

- **No product analytics SDKs, replay, heatmaps, retention cohorts, or interaction breakdowns:** The plan says these would amount to observing user or bird interaction patterns.

- **Telemetry registry and scrubbers:** The reason is enforcement: unknown labels, UUID-like values, emails, request bodies, bird names, and personality-shaped fields are rejected or redacted.

- **Network isolation from warehouse to simulation DB:** The rationale is that analytics must not have credentials or a network path to canonical simulation data.

- **Drift calibration without production aggregation:** Because production drift cannot be aggregated, calibration uses the offline harness, consented staff dogfood accounts, and synthetic prod accounts only.

### 12. Testing and quality strategy

- **`sim-core` property tests and calibration harness:** These enforce monotonicity, ceilings, dt-invariance, union presence, attunement/mood separation, cooldowns, and calibrated drift.

- **Sync Jepsen-lite tests:** The reason is exactly-once event consumption and proving that personality writes never happen outside the tick.

- **Migration continuity tests:** They protect bird identity and vectors from invisible catastrophic changes.

- **Contract denylist for traits and email:** The why is to enforce trait privacy and PII boundaries in API and snapshot schemas.

- **E2E first-bird with no spinner:** The test directly enforces the first-frame and no-busy-indicator invariants.

- **Accessibility automated and manual tests:** The plan checks axe, keyboard journeys, live-region cadence, reduced-motion thresholds, and screen-reader passes because accessibility gates release.

- **Copy lint:** The rationale is preventing announcement creep, gamification language, mood labels, and system/naturalist voice mixing.

- **Audio offline render suite:** The why is clipping, loudness, uniqueness, and recognizability before human listening panels.

- **Call recognizability panel:** It gates birds-per-aviary ramp because users must recognize birds in 3-, 5-, and 7-bird mixes.

- **Drift perception study:** The reason is to ensure three-week differences are noticed while one-week changes mostly are not.

- **Uncanniness listening panel:** The plan uses it to iterate until calls are natural and pleasant enough for launch.

### 13. Rollout

- **Milestone order:** The introduction says the most important properties are built first. M1 builds sim and harness, M2 first frame and reduced motion, M3 audio, and later milestones layer interactions, voice, visits, and hardening.

- **Accessibility as a gate, not a milestone:** The rationale is that accessibility must be present across renderer, interactions, voice, and visits rather than deferred.

- **Dogfood alpha lasting at least five weeks:** The reason is to cross the three-week visible-drift horizon and tune presence, cadence, drift constants, sparsity, and greeting feel.

- **Time-warped staff aviaries:** They exercise 3- to 7-bird aviaries early while being unavailable in the user path.

- **Closed beta with visits behind a flag after abuse review:** The reason is invite email abuse risk.

- **Public launch capacity gate:** New signups are adjusted from tick headroom metrics.

- **`max_birds_enabled` operational flag:** The plan calls it a hidden safety net; if recognizability or perf is not ready, eligible aviaries simply do not spawn a wanderer yet and nothing is surfaced.

- **Instrumented from day one:** Synthetic accounts and DR drills begin early so performance, privacy, and personality integrity are tested before and after beta.

### 14. Risks and mitigations

- **Drift too fast:** The plan uses JND limits, headroom saturation, daily soft caps, dogfood perception, and `drift_apply` to avoid Tamagotchi-like change.

- **Drift too slow:** The plan uses the 21-day visible band, A/B dogfood tests, and configurable constants so "nothing seems to matter" can be corrected.

- **Homogenization:** Per-bird ceilings, species sensitivities, individual seeds, and voiceprint-driven expression keep loved birds from converging.

- **Presence inflation:** One owned `PresenceMonitor`, server clamps, cross-device union, and daily soft caps prevent laxer checks from creeping in.

- **Neglect reading as punishment:** Attunement does not touch mood valence, has a floor, recovers quickly, and gets writer review after long absence.

- **Audio uncanniness, repetition, and lost recognizability:** DSP ownership, motif design, micro-variation, uniqueness guards, voiceprint separation, classifiers, human panels, and bird-count gates mitigate the affective risk.

- **Autoplay policy:** The plan accepts the platform constraint and uses visual call cues, captions, and gesture fade-in rather than an enable-sound overlay.

- **Narration spam or flattened state lists:** Cadence tests, screen-reader passes, writer ownership, lint, and enumeration rejection keep narration observational.

- **Canvas accessibility:** DOM focus proxies and e2e bounds checks prevent focus buttons from drifting away from rendered birds.

- **Cold high-RTT 500 ms risk:** The plan interprets the gate as warm DNS, then uses HTTP/3, edge, inline snapshot, SW cache, and a small boot chunk while tracking cold-start separately.

- **Announcement creep:** No toast primitive, copy lint, product-principles review, and enum-driven system strips keep "just a small toast" from appearing.

- **PII leakage:** Registry, scrubbers, log scans, crypto-shredding, and email confinement enforce privacy.

### 15. Decisions on ambiguities

- **Settle in the top bar:** The rationale is that settle is specified as top-bar in two places and needs a keyboard-reachable home; putting it inside offer would misclassify it.

- **Presence activity includes pointerdown but excludes wheel, scroll, orientation, and audio:** The decision keeps touch workable while staying faithful to the definition.

- **Separate attunement variable:** The rationale is satisfying both monotonic traits and "greet less often" after neglect.

- **Soft server-side dawn mood reset:** It keeps session persistence while satisfying daily-ish mood reset without visible snap.

- **Offer target automatic and server-chosen respondents:** The rationale is keeping offers as gestures rather than targeting UIs.

- **Export includes vectors in `simulation_state`:** The reason is data portability; the export is "not a product surface."

- **Audio audible only when browser allows:** The plan treats autoplay as a platform constraint and rejects overlays because they violate "notice, never announce."

- **Hidden-tab audio fade and suspend:** The rationale is battery and avoiding throttling glitches.

- **Account vs device settings split:** User needs follow them; mute is situational to a device.

- **Visitors see names:** The plan says the host chose to share and visitors see "exactly as it is."

- **No manual offer placement:** The rationale is that the PRD rejects user arrangement of the scene.

- **Offline offering queues event but withholds bird reactions:** This keeps the server authoritative while avoiding a broken affordance.

- **English-only v1:** The reason is scope and language-specific naturalist grammar.

- **Visit link binds once but grants ongoing access:** The decision preserves both one-time link semantics and revocable active access.

- **Wanderer instead of new-bird announcement:** The rationale is that birds "arrive" and are noticed, not acquired or unlocked.

- **Every aviary ticks at the same cadence:** The plan says the tick runs whether or not anyone is connected and cost is manageable.

- **Soft-deleted aviaries keep ticking:** A restored aviary should have continued without the viewer, like any other aviary.

### 16. Definition of done

- **Automated enforcement of invariants:** The rationale is that invariants are "non-negotiables every PR is checked against," not guidelines.

- **Dogfood perception, first-bird, soak, 60 fps, recognizability, screen-reader, reduced-motion, and privacy signoffs:** These define done as validated product feel, accessibility, performance, recognizability, and privacy rather than only feature completion.

- **No toast, spinner, counter, badge, or streak anywhere:** The final grep/lint audit enforces the quiet, non-gamified product voice across the codebase.
