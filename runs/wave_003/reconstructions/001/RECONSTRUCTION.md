## System-level intent

- **Correctness is affective, not just functional.** The plan opens by saying Pocket Aviary's correctness conditions are "mostly affective" and that a build can pass functional tests and still fail if it ships a `"Welcome back!" toast`. This shows up again in the first-frame discussion as "the most affectively important 500 milliseconds in the product," and in the definition of done where "without being told, they notice Pip comes forward now."

- **Make the wrong product structurally unrepresentable.** The organizing conviction is that hard rules should be "made structurally unrepresentable wherever possible, not merely documented." The plan spends design budget on types and missing paths: "a personality accumulator that has no decrement operation," "a notebook generator with no access to a user-fact type," "a narration generator that never receives a mood enum," and "a lint boundary that makes system-voice strings unimportable from product surfaces."

- **Notice, never announce.** The plan repeatedly rejects announcement surfaces: "No announcement surfaces," "The bird is the welcome," "no modal," "no celebration," "no onboarding tour," "no feature callouts," and "no 'here's how to listen in.'" This is also the reason resume does not animate eight hours of change as a montage, autoplay has no "click to enable sound" prompt, and launch opens with "a plain sign-up page and the product."

- **The aviary observes and changes on slow, natural timescales.** Drift is calibrated so "no single session shifts a trait visibly," "quietness is fast-timescale, drift is slow-timescale and one-directional," and three weeks can create visible change without telling the user. Mood, weather, greeting variation, notebook sparsity, and bird offers all use time as a naturalistic medium rather than an engagement loop.

- **Hidden personality is the product's central private state.** Personality vectors are "never exposed to any user surface," only the server tick writes them, updates are additive deltas, and an audit table exists because "losing a vector is the worst failure this product can suffer and would be invisible to monitoring." Support tools intentionally cannot see the birds' vectors.

- **No gamification and no resource mechanics.** The plan bans "streak, score, level, badge, XP, rank, visit count," avoids visible cooldowns, discards excess drift drive instead of banking it because banking would let a user "'save up' attention," and refuses visit-frequency metrics because "the query does not exist."

- **Naturalist voice for the aviary, matter-of-fact voice for system surfaces.** The copy system has two namespaces, `naturalist` and `system`, and the plan makes the split "an architectural property rather than an editorial one." Naturalist surfaces describe the aviary and ban second person; identity, errors, sync, settings, accessibility, and financial-like surfaces use matter-of-fact strings.

- **The user reads signals; the system does not explain numbers.** Perch position is "a signal the user reads, not a layout the user controls." Mood is "never labeled." Narration receives posture and behavior descriptors, not mood enums or trait values. Plumage is quantized enough to render but "too coarse to reconstruct a number."

- **One canonical aviary, presentation local where appropriate.** The sync model says "there is no sync algorithm" because clients read snapshots and write events. Canonical state lives on the server; pixel positions, listen-in mix, settled lighting, captions, viewport layout, and reduced-motion presentation are local presentation state where making them canonical would create conflicts or wrong cross-device behavior.

- **Privacy is infrastructure, not policy.** The simulation database sits in its own VPC and cloud account; there is "no CDC stream, no replica in the data platform, no nightly dump." Metrics use a typed dimension allowlist, per-bird interaction state is not aggregated, and the plan accepts lost product insight because the strict privacy reading is part of the product.

- **Accessibility is the same product, not a downgraded mode.** Reduced-motion is "a designed presenter, not disabled animation," with identical drift, mood, calls, notebook, and narration. Screen-reader narration is naturalist prose, never a state list. Accessibility work is scheduled inside the build because shipping it later would tell those users "the product was not for them."

- **Performance and loading states must preserve charm.** The first-bird path gets a stricter 90KB budget because the 500ms experience matters more than the 2MB application ceiling. Slow load and empty aviary are both "the quiet field"; there is "No entry animation. No fade-from-static. No spinner. No skeleton."

- **Emergence over scripted events.** Chorus is "emergent, not scripted"; greeting variation is combinatorial rather than "three pre-authored greeting animations in rotation"; idle motion comes from noise that "never loops and never lines up." The plan consistently prefers systems that produce recognizable life over authored rotations.

## Per-feature whys

### 0. How to read this plan

- **Invariant register and charm suite:** The plan explains that the product can fail affectively even if it passes ordinary functional tests, so it organizes around systems and mechanisms that keep the product from "drifting into the wrong product." The invariant register plus charm suite make violations hard rather than merely discouraged.

### 1. Invariant register

- **Presence requires visibility, focus, and recent pointer/key activity:** The rationale is to ensure presence only means real watching, never a subset of signals. The plan later says watching birds without moving is real product behavior, but an abandoned open laptop should stop counting.

- **Personality traits are monotonic non-decreasing:** This enforces the no-Tamagotchi rule. Neglect produces "nothing at all," and drift is "slow-timescale and one-directional."

- **Personality vectors are private:** The plan treats hidden personality as an internal expressive engine, not a stats surface. Values are not exposed in panels, debug views, ARIA, APIs, analytics, or exports except explicit account export.

- **Only the server simulation tick writes personality state:** The rationale is sync correctness and integrity: clients write events only, preventing conflicting personality writes and client inflation of the central mechanic.

- **No last-write-wins on personality:** Additive deltas in event-log order prevent an old read from overwriting newer state and avoid silent loss of drift.

- **Calls are procedurally synthesized:** Recorded audio is excluded because per-bird calls need stable identity plus variation, and because even fallbacks must not ship recorded audio assets.

- **No announcement surfaces:** The product's thesis is notice rather than announce; welcome toasts, banners, arrivals, absence copy, and celebrations put the user in the wrong register.

- **No gamification:** The plan bans counters, streaks, levels, badges, and visit-frequency surfaces because they convert the aviary into a game or engagement system.

- **Notebook observes the aviary, never the user:** This keeps notebook prose from becoming "you visited every day this week" and protects the line between aviary observation and user surveillance.

- **Voice split:** Naturalist surfaces and system surfaces need different registers; the lint boundary makes this split durable rather than editorial.

- **No spinner, skeleton, progress bar, or fade-from-static on the aviary:** The load state should read as "the quiet field," not as application loading chrome.

- **Bird identity is stable forever:** A reset, regenerated, swapped, or reseeded bird would break the user's long-term relationship and the hidden drift history.

- **Visitor sessions never contribute presence, interaction events, or drift:** A visitor looking at the host's aviary must not change the host's birds.

- **Per-bird/account state never enters telemetry:** Interaction state exists only to drive that user's simulation, not analytics, aggregates, training, or population analysis.

- **Reduced-motion is a designed presenter:** The same scene state should be available in cross-fades rather than disabled animation, so reduced-motion users get the same drift, mood, calls, notebook, and narration.

- **Screen-reader narration is naturalist prose:** A screen-reader user should receive the aviary as prose, not as a mood enum, number, or state list.

- **Perch position is not user controlled:** Perch location is a bird signal for the user to read, not a layout tool.

- **No push, ping, or re-engagement email:** The product should not summon the user back; the only exception is an opt-in visit notification about a person.

- **Email stored once and synthetic UUID elsewhere:** This prevents account references from becoming email-linked across the system.

- **New birds gated on aviary age only:** New birds should not depend on visits, interactions, drift, or payment, avoiding engagement and monetization mechanics.

### 2. Scope

- **Account and identity:** The plan lists sign-in, sessions, email change, export, and deletion as v1 scope. Specific rationale is articulated later: magic links avoid account enumeration; export is user-controlled access; deletion needs a recovery window and hard delete.

- **The aviary:** One canonical aviary, starter birds, cap, species, perch zones, day/night, weather, micro-motion, and thin top bar create the always-present scene with minimal chrome. The plan emphasizes that scene and mood must agree and no bird should be cropped or controlled as layout.

- **The bird engine:** Hidden traits, mood, procedural calls, chorus, idle motion, and perch choice are the "large hidden system" behind a small UI surface. The rationale is that birds should become expressive and recognizable without exposing numbers.

- **Interactions:** Greeting, listen-in, offers, settle, presence, and notebook are scoped because they give the user gentle gestures and long-term observations without announcements, counters, or game actions.

- **Sync:** Server-side simulation and append-only event logs exist so the aviary keeps living whether or not a client is connected, while clients remain presentation layers.

- **Social:** Invites are scoped as read-only ambient visitor views so a friend can look without becoming co-presence, chat, or interaction.

- **Accessibility:** Narration, reduced motion, captions, keyboard, focus, and contrast are v1 scope because accessibility is not a later fix and should deliver "the same three weeks."

- **Performance:** The budgets exist because a blank, slow, or memory-growing aviary damages the feeling of continuity.

- **Observability:** Aggregate RUM, synthetic checks, and tick latency alarms give operational health without per-account or per-bird analytics.

- **Native apps:** NOT RECOVERABLE FROM PLAN

- **Payments and tiers:** NOT RECOVERABLE FROM PLAN

- **Shared, team, or household aviaries:** NOT RECOVERABLE FROM PLAN

- **Multiple aviaries per account:** NOT RECOVERABLE FROM PLAN

- **Customizable or purchasable scenes:** NOT RECOVERABLE FROM PLAN

- **Public discovery, profiles, follows, feeds, comments, ratings, featuring:** NOT RECOVERABLE FROM PLAN

- **Leaderboards and aggregate metrics underlying them:** The plan excludes them as part of the no-gamification and no population-analysis stance.

- **Achievements, streaks, levels, scores, badges, XP, counters:** The rationale is explicit: these are gamification surfaces the product must not become.

- **Push notifications and re-engagement email:** These would ping the user about the aviary, contradicting notice-never-announce.

- **Tamagotchi mechanics:** Hunger, death, distress, decaying happiness, and negative drift are excluded because neglect must produce no penalty.

- **Co-presence during visits:** NOT RECOVERABLE FROM PLAN

- **Editable notebooks:** The notebook is an immutable record of observations; editing would change the record.

- **User-controlled perch placement:** Perch position is a signal the user reads, not a layout control.

- **Numeric or debug view of personality:** Personality values must not become a user-facing stats/debug surface.

- **Recorded-audio fallback:** The plan forbids recorded audio in all paths.

- **Compatibility paths for older browsers:** NOT RECOVERABLE FROM PLAN

- **Small feature surface with large hidden system:** The plan accepts this risk because the bird engine, sync, audio, and accessibility carry more engineering weight than the UI footprint suggests.

- **Drift calibration floor under schedule:** The plan accepts that multi-week beta is required because the feel of drift cannot be compressed by adding people.

### 3. Architecture

- **Five services plus static edge tier:** The plan separates API, identity, visits, simulation, mailer, and edge because their ownership boundaries differ: stateless request handling, canonical tick work, transactional mail, and snapshot delivery.

- **TypeScript on Node 22 and TypeScript client:** The language choice exists so call grammar, captions, narration, and naturalist phrase engine can produce "byte-identical output on client and server."

- **No UI framework in the aviary render path:** NOT RECOVERABLE FROM PLAN

- **Simulation Postgres isolated in its own VPC/cloud account:** This makes INV-14 infrastructure rather than policy, preventing analytics or warehouse credentials from reading simulation state.

- **Server-owned snapshot state:** Mood, perch assignment, call schedule, greeting plan, weather, roster, and notebook are canonical because they are behavior and must agree across devices.

- **Client-owned presentation state:** Pixel positions, micro-motion, particles, top-bar fade, listen-in focus, settled lighting, captions, presenter choice, and viewport layout stay local because they are rendering and should not create sync conflicts.

- **Shared-deterministic audio synthesis:** The server owns the schedule while the client synthesizes from seed tuples. This makes chorus a simulation property while avoiding bandwidth and keeping captions matched to audio.

- **Render pipeline with StandardPresenter and ReducedMotionPresenter over the same state:** The rationale is that reduced motion has no branch inside simulation or behavior code, so parity falls out of the structure.

### 4. Data model

- **UUIDv7 IDs:** The plan says they are time-ordered and index-friendly.

- **Account synthetic id:** Synthetic UUID is the only external reference so email does not spread through the system.

- **Encrypted email plus lookup hash:** The blind index lets sign-in find an account without storing plaintext email anywhere but one encrypted column; KMS pepper prevents a database compromise alone from becoming an email-enumeration oracle.

- **Magic-link token hashing and conditional consumption:** Storing only hashes and consuming via conditional update makes replay impossible even under concurrency.

- **Rate-limited neutral auth response:** Returning the same neutral response prevents account-existence oracle behavior.

- **One aviary per account:** NOT RECOVERABLE FROM PLAN

- **Aviary timezone and candidate debounce fields:** These support day/night without travel whipsaw.

- **Dormant scheduling tier:** The tier is scheduling only, separating simulated cadence from execution cadence.

- **Bird seed plus accumulators:** Traits are stored as seed plus accumulator so the displayed value is a pure function, the accumulator has no decrement path, and the seed is write-once.

- **Immutable bird id:** Stable identity prevents resets or reseeding from breaking the bird.

- **Personality delta audit:** The audit provides a reconstruction path and nightly assertion because losing a vector would be the worst invisible failure.

- **Interaction events retained 90 days:** Events are tick inputs, not a user-facing history; shorter retention shrinks the blast radius of the sensitive table.

- **Notebook entries immutable:** Later grammar improvements should affect future entries only, because rewriting an entry would edit the user's record of what happened.

- **Visit table has no path into simulation:** This enforces that visitor sessions cannot affect host birds.

- **Species definitions in code:** Species data is versioned with renderer and synthesizer and must stay perfectly in sync with both.

### 5. API surface

- **HTTPS JSON with HttpOnly/Secure/SameSite cookies:** NOT RECOVERABLE FROM PLAN

- **Error bodies carry `surface_key` instead of prose:** Copy stays in the client string system where lint boundaries enforce matter-of-fact system voice.

- **Identity endpoints:** Request-link always returns 202 to avoid account-existence leakage; other identity endpoint rationales are described under auth.

- **Snapshot endpoint shape:** It strips traits while giving enough information to render birds, schedule calls, and narrate; greeting is in the request path so it can land within 1-2 seconds of tab open.

- **Quantized plumage saturation step:** It is enough to render but too coarse to reconstruct a trait value.

- **Server-generated narration in snapshot:** Notebook and narration share one voice engine and can be tuned without client deploy.

- **Batched event submission with idempotency:** Flush batching and `client_event_id` idempotency prevent duplicate `sendBeacon` delivery from double-counting presence.

- **Offer endpoint returns immediately:** Immediate client-side shared behavior makes the offer feel instantaneous while canonical outcome still comes from the server.

- **Visitor token scopes as hard capability:** Visitor tokens lack `write:events`, so event ingest has no branch reasoning about visitors; this enforces visitor non-impact.

- **Visitor polling every 15 seconds:** This bounds revocation delay honestly without adding a WebSocket tier for a rarely used feature.

- **No API for traits, drift history, visit counts, session counts, days visited, or user behavior aggregates:** These omissions prevent stats surfaces, gamification, and analytics leakage.

- **No perch assignment, mood override, absolute personality write, or admin personality vector endpoint:** These omissions preserve perch as signal, mood as hidden state, additive server-owned drift, and private vectors.

### 6. Simulation engine

- **Tick as pure function:** Purity allows batching, retry, replay, and acceleration without changing user experience.

- **60-second tick cadence:** NOT RECOVERABLE FROM PLAN

- **Active and dormant aviary scheduling:** The product claim is every aviary ticks whether connected or not, but one transaction per account per minute is too costly for unwatched accounts; batching preserves state equivalence while controlling cost.

- **Synchronous catch-up before serving dormant snapshots:** The user is never served a stale aviary, and the state does not depend on their arrival.

- **Hash sharding by aviary id:** NOT RECOVERABLE FROM PLAN

- **Hyperbolic drift saturation:** Exponential saturation would pin older birds at the ceiling; hyperbolic keeps a long tail so two years remains more expressive than one year without exceeding 1.0.

- **Six-hour low-pass filter:** Drift smooths across session boundaries and into the next day so one long session does not immediately convert to visible trait movement.

- **`max(0, drive)` guard:** It is a second lock on monotonicity against future signed signals.

- **Presence weights by trait:** Plumage is highest because sustained attention explicitly ties to it; warm reflects greeting-first behavior; bold is the slowest legible coming-forward change; curious is mostly driven by offers. Vocal has no specific rationale in the table.

- **Listen-in drift to focused bird:** The reward for listening shows up over weeks as warm and vocal drift, not immediate performance.

- **Offer accepted/made drift impulses:** Curious is driven by accepted offers; making an offer gives all birds a small bold impulse. Further rationale is not articulated.

- **Settle has no drift contribution:** It ends presence and nudges mood only; there is no penalty or different drift path.

- **Per-session and per-day drift budgets:** Unbudgeted marathon sessions would visibly move traits in one sitting; caps prevent attention from becoming a resource mechanic.

- **Discarding excess drive:** Banking excess would let the user "save up" attention, which is the wrong product.

- **Measurable vs visible drift definitions:** The user perceives behavior, not values, so visible drift requires both numerical movement and a behavior-threshold crossing.

- **Neglect produces no drift:** Quietness after absence is a mood/call-density effect and recovers when the user returns; personality does not decay.

- **Mood FSM states including roosting:** Roosting finalizes the night state so night is represented as settled or sleeping.

- **Minimum dwell time:** Mood flickering at tick resolution would make birds twitch between postures and read as a machine sampling a distribution.

- **Daily dawn re-derivation:** This honors daily-ish reset without snapping to a default on tab open.

- **Damped contagion:** A single startle should not cascade the entire aviary into wary, because that would read as a scripted event.

- **Mood never labeled:** The user gets posture, captions, and behavior, never mood icons, tooltips, enum names, or color coding.

- **Call schedule as Poisson process modulated by vocality, mood, weather, and time:** NOT RECOVERABLE FROM PLAN

- **Listen-in does not change call schedule:** Listen-in changes mix only, preserving reward on the slow drift timescale rather than immediate performance.

- **Chorus emergent, not scripted:** If chorus rate is wrong, tune rates and join probability rather than adding a "chorus event," preserving emergence.

- **Call as seed tuple:** One deterministic value produces audio parameters and caption prose, so caption and audio cannot drift apart.

- **Identity signature vs variation:** Pip remains recognizably Pip while no two calls are identical; this is the testable definition of recognizability.

- **Per-bird individuation within species:** Same-species birds in one aviary must be distinguishable.

- **Six-species pool designed as a set:** All six must be mutually distinguishable and any two should make a good starter pair.

- **Nightjar species:** It keeps night from being a dead state.

- **Nightjar excluded from starter pairs:** A new user whose first day is mostly a quiet dusk/night bird would read the aviary as broken.

- **Starter selection for different contour and timbre:** The first pair a user hears should be maximally easy to tell apart.

- **User does not choose starter birds:** NOT RECOVERABLE FROM PLAN

- **Timezone travel debounce:** A day trip should not whipsaw the aviary; a move should update it within a day.

- **Solar times from timezone latitude table:** The product needs evening to feel like evening without holding user location; astronomical accuracy is unnecessary.

- **Scene follows aviary timezone:** Scene and mood must never disagree, because that small wrongness reads as fake.

- **Rare, small weather:** Weather feeds mood and agrees across devices, but "must never be the most interesting thing on screen."

- **Bird-offer thresholds by aviary age:** The thresholds fit the plan's target that a few months can bring a third bird and a year-old aviary may have five or six.

- **New bird presentation without modal/celebration:** A bird arrives as a bird, at the back perch, avoiding "you unlocked" language and preserving quiet discovery.

- **Declining by inaction:** This allows refusal without a dialog.

### 7. Interactions

- **Return-greeting in snapshot request path:** It must land within 1-2 seconds of tab open and the tick is minute-wide.

- **Absence bands:** Greeting intensity should reflect absence length while continuity under two minutes has no greeting so tab-focus does not feel canned.

- **Greeter selection by bold/warm/mood availability:** Bolder, warmer, available birds greet more often; wary birds may not greet. Anti-repetition makes variation true and notebook-worthy.

- **Greeting variation across five axes:** Combinatorial and continuous variation prevents the feature from becoming a rotation among stored variants.

- **Staggered multi-bird greetings:** Unison would announce the user's arrival, which is the wrong register.

- **No textual welcome:** The bird is the welcome.

- **Listen-in engagement/disengagement controls:** NOT RECOVERABLE FROM PLAN

- **Equal-power mix ramp:** No hard cut; a linear crossfade would have an audible dip. Other birds never silence so the aviary remains a place with several things happening.

- **Listen-in changes mix only:** It should not make a bird perform; the reward appears over weeks through warm and vocal drift.

- **Listen-in decay after absent presence and settle:** A forgotten listen-in should not leave the aviary permanently unbalanced.

- **Listen-in mix is session-local, events canonical:** The local mix should not change another device, but listening events still drive drift.

- **Offers from top bar, not bird click:** Bird click is reserved for listen-in.

- **Offer made to the aviary, not a bird:** Birds decide independently, supporting naturalistic responses.

- **Seed offer:** Response depends on curiosity and mood. Further rationale is not articulated.

- **Song fragment offer:** Responses are musically related through the call grammar, so a bird answers that motif rather than making a random call.

- **Still pool offer:** NOT RECOVERABLE FROM PLAN

- **Invisible offer cooldown:** A visible timer would convert a gesture into a game action; if birds ignore an offer, it reads as natural disinterest.

- **Settle ritual:** It gives people who want one a quiet end-of-session gesture without changing drift rewards or creating a penalty.

- **Settle undo by click or Escape:** Reversal without toast/button/confirmation preserves the no-announcement register; Escape provides keyboard parity.

- **Settle emits same presence termination as tab-close/timeout:** There is no "you didn't settle" surface or drift difference.

- **Settled lighting session-scoped:** Settling on one device should not make another device look broken; settle is a gesture for that session, not a world change.

- **Presence activity window of five minutes:** It is long enough for genuine watching without movement and short enough for an abandoned laptop to stop counting.

- **Server-side presence integrity:** Presence is the dominant drift input, so client timestamps and modified clients must not inflate it.

- **Concurrent presence union:** Two devices open should not silently double the drift rate.

- **BroadcastChannel leader election:** NOT RECOVERABLE FROM PLAN

- **Hidden tabs stop rendering while simulation continues:** Background tabs should cost approximately nothing, and return should pull fresh state.

### 8. Accounts, sync, and social

- **No sync algorithm:** One canonical server record, snapshots, and events avoid CRDTs, merges, vector clocks, and diverging offline queues.

- **Snapshot KV delivery:** The hot path is KV rather than database, making the inlined-snapshot first-frame path fast enough.

- **Freshness bound of one tick plus KV propagation:** This is the intended model because the client should interpolate from the last tick.

- **Client interpolation:** Birds ease and fly to canonical targets instead of teleporting.

- **Resume after long gap as cold start:** Animating hours of change would be a "here's what you missed" montage, which is both an announcement and a lie about continuity.

- **Magic links:** Hashing, expiry, conditional consumption, and neutral responses protect against replay and account enumeration.

- **Per-device sessions:** Sessions can be listed and individually revoked; matter-of-fact labels keep this in system voice.

- **Session token rotation with overlap:** NOT RECOVERABLE FROM PLAN

- **Email change verification and notices:** New-address verification prevents silent moves, and notice to the old address protects against compromised sessions.

- **Account export:** It is the one correct place for trait values to leave the system because the user explicitly asks for their data; JSON file, not a product stats view.

- **Soft delete then hard delete:** Soft delete allows recovery; hard delete removes all account-related data and asserts no orphaned rows.

- **Visits as one-time emailed invite links:** This keeps visitors bounded and deliberately re-issued rather than creating a permanent visitor list.

- **Visitor sees the same aviary without show-off mode:** Prettification would betray the point of letting a friend look.

- **Visitor interaction surfaces omitted, not disabled:** Disabled-with-tooltip advertises things the visitor cannot have.

- **Visitor invisible in aviary:** No avatar, cursor, marker, cue, chat, or comments; visitor attention contributes nothing.

- **Revocation checked on every visitor snapshot:** The visitor gets a matter-of-fact surface within one poll.

- **Visit notification exception:** It is allowed because it is about a person, not the aviary's state; it is off by default, email only, and never push.

- **Permitted outbound email enum:** Adding a ninth template requires changing an enum, making it a visible decision rather than a quiet one.

### 9. Audio pipeline

- **AudioWorklet graph:** Worklets keep synthesis off the main thread for 60fps and avoid per-call allocations for the 30-minute memory test.

- **Generated reverb impulse response:** It avoids 100-300KB of audio asset cost and avoids anything arguably recorded audio.

- **Per-note synthesis:** NOT RECOVERABLE FROM PLAN

- **Per-call jitter:** Guarantees no two calls are identical.

- **Identity signature in synthesis:** Guarantees calls remain recognizably the same bird.

- **Ambient bed, perch-zone ducking, and soft limiter:** Perch-zone audio reinforces visual proximity; limiter prevents chorus clipping. Ambient bed rationale is not specifically articulated.

- **Equal-power listen-in ramp:** Avoids the audible dip of a linear crossfade.

- **Runtime call captions from synthesis params:** Caption and audio are two projections of one value, so they match.

- **Caption scrim:** Legibility and WCAG AA win over no-chrome tension, kept as soft and small as contrast permits.

- **Captions `aria-hidden`:** Screen-reader users already receive narration; double-speaking calls would be noise.

- **Autoplay handling without prompt:** Browser policy may silence the first greeting, but an enable-sound overlay would be an announcement and the first thing a new user saw.

- **Greeter calls again on first gesture if needed:** A second natural call is better than a replayed cue.

- **Top-bar audio-state icon:** A sanctioned chrome location gives users somewhere to look if they wonder about silence.

- **WebAudio unavailable silence with captions:** No recorded fallback is allowed; system explanation appears only if the user looks in settings.

### 10. Voice, notebook, and narration

- **Typed naturalist/system copy namespaces:** They make voice split architectural.

- **Banned lexicon:** It catches copy that reliably leaks the wrong register, including welcome, streak, unlocked, score, progress, and comeback language.

- **Second person banned in naturalist surfaces:** Naturalist voice describes the aviary, never addresses the user.

- **Field notebook generation from AviaryFacts:** Facts let the notebook write about birds and events in the aviary.

- **Notebook scorer by novelty, contrast, and coincidence:** These criteria identify observations that matter rather than routine entries.

- **Notebook sparsity limiter:** An observation per session would dilute meaningful ones; a scorer alone would not hold the line for heavy users.

- **Notebook template grammar, not runtime LLM:** Runtime LLM latency, cost, and unpredictability are wrong where voice is load-bearing; LLMs can help offline under human review.

- **Notebook no UserFact dependency:** The system cannot construct user-observation sentences because the required type does not exist.

- **Notebook read-only, uneditable, unannotatable, infinite scrollback:** Entries are an immutable record. Specific rationale for unannotatable and never archived is not articulated beyond preserving the record.

- **Screen-reader narration:** A listener deserves prose describing what a person watching would say, not mood names or state transitions.

- **Narration cadence:** Idle updates are slow, faster only for user-initiated events, so narration does not become interruption.

- **Polite live regions, never assertive:** Assertive interrupts, and interrupting is announcing.

- **Canvas `aria-hidden` with parallel DOM buttons:** Canvas is not semantic; buttons are accepted because listen-in is actionable and keyboard/screen-reader users need to know it.

- **Keyboard navigation:** It makes top bar, scene birds, offers, listen-in, and settle usable from the keyboard. Further rationale is not articulated.

- **Dual-stroke focus indicator:** It reads against bright midday and dim night scenes; focus visibility is never suppressed.

- **Contrast suite:** Captions and top bar sit over changing luminance, so they must be tested against palette extremes.

- **Reduced-motion cross-fade presenter:** It should be calmer, not wronger; the mid-air pose avoids teleportation.

- **Reduced-motion override:** Some users want reduced motion without OS setting, and some want full motion despite it.

### 11. Frontend rendering pipeline

- **Canvas2D single path:** The decisive rationale is the 500ms first-bird budget; WebGL initialization and shader compilation cost too much for headroom the scene does not need. A single path also reduces baselines and profiles.

- **Preact plus small hand-rolled component set for chrome:** NOT RECOVERABLE FROM PLAN

- **Absent component primitives:** Toasts, snackbars, banners, modal alerts, spinners, skeletons, progress bars, badges, and counters must be created deliberately rather than imported.

- **Bird vector path geometry and pose atlas:** Geometry stays compact and renderer-linked; per-session atlases are fresh because plumage drifts weekly.

- **Rasterize only first-frame poses synchronously:** Protects the first-bird budget; the rest can fill during idle time.

- **Parameterized poses:** Behavior such as preening becomes continuous parameter trajectories rather than canned clips.

- **Local-time sky before network:** The first paint is already the aviary's sky, avoiding a blank or white frame.

- **Inlined edge snapshot:** Removes a round trip so birds can be placed as soon as the renderer boots.

- **No entry/loading animation:** Birds appear already in motion at simulated positions.

- **Slow-network quiet field:** It reads as the aviary catching up, not product loading.

- **Failure state over quiet field with matter-of-fact message:** Avoids white flash and error-page chrome while using system voice.

- **Layered scene composition:** NOT RECOVERABLE FROM PLAN

- **Subtle non-pointer parallax:** Pointer-driven parallax would convert the window into an interactive toy.

- **Responsive mapping by widening gaps, not scaling/cropping:** No bird is ever cropped, and front/back reading is preserved.

- **Stable slot assignment during resize:** Birds should not shuffle on resize.

- **Ambient leaves and feathers as client-only ornaments:** They cheaply signal continuing life while keeping snapshots small and ticks cheap.

- **Top bar with four icons and no counts/badges:** It keeps chrome minimal and avoids counters, notification dots, and status surfaces beyond audio state.

- **Top-bar fade never fully invisible and opaque on focus:** A control the user cannot find is worse than a faint one, and keyboard focus must keep it available.

- **Fixed 30Hz behavior loop with rAF interpolation:** NOT RECOVERABLE FROM PLAN

- **Noise-based idle micro-motion:** It never loops and never lines up between birds.

- **Hidden tab suspension:** Background tab cost should be approximately nothing.

### 12. Performance and observability

- **Total initial JS budget:** NOT RECOVERABLE FROM PLAN

- **First-bird critical path budget:** 500ms over 4G allows only roughly 60-90KB on the critical path, so this sub-budget matters more than the whole-app 2MB cap.

- **Time to first bird measurement:** It verifies the affectively important first frame under realistic mobile and network constraints.

- **60fps sustained budget:** Smooth idle motion is part of the aviary feeling alive; the allocation leaves headroom "where reality goes."

- **Memory growth test:** Structural allocation choices need a real CI soak because optional manual memory tests stop being run.

- **Tick latency alarm:** NOT RECOVERABLE FROM PLAN

- **Snapshot payload cap:** NOT RECOVERABLE FROM PLAN

- **Code-splitting settings, notebook, offer panel, and invite flow:** Keeps only aviary and top bar in the initial graph to protect first bird.

- **`first_bird_drawn` aggregate RUM:** Measures the real user first-bird path without per-account dimensions.

- **Synthetic checks:** They are stable and carry no user data, making them primary regression signals.

- **Memory structural choices:** One-time worklets, object pools, bounded atlases/particles, virtualized notebook rows, last-two snapshot cache, and disposers prevent retained growth.

- **Operational metrics:** They provide request, tick, cache, mail, auth, invite, and health visibility. Specific rationale is general operational health without sensitive dimensions.

- **No DAU/WAU/MAU, retention cohorts, session counts, visit frequency, per-bird/per-account metric dimensions, drift aggregates, or event-type breakdown:** The plan accepts lost product insight because strict privacy forbids population analysis of bird interactions.

- **Metric definitions in one typed registry:** Adding a dimension becomes reviewable.

### 13. Privacy and the data boundary

- **Per-bird interaction events only drive that user's simulation:** The plan explicitly says they are never aggregated, used for training, shared, cross-user features, or population analysis.

- **Network and IAM isolation:** Strongest enforcement because no analytics/warehouse/ML credential can reach simulation data.

- **Terraform CI check:** Prevents infrastructure drift that would create an analytics path.

- **Metric dimension allowlist:** Keeps telemetry aggregate-only and dimension-controlled.

- **Encryption:** Email encryption and identifier discipline address linkage risk.

- **Retention:** Interaction events are temporary inputs; notebook and vectors last account lifetime; deletion removes everything after the recovery window.

- **Log redaction:** Prevents email and personality fields from leaking into logs.

- **Plain-text privacy policy in account settings:** It names collected aggregate categories and excludes per-bird interaction state in matter-of-fact voice.

### 14. Rollout

- **Milestones M0-M7:** The ordering builds foundations, tick, engine, renderer, audio, interactions, accessibility, and social/account lifecycle with exit criteria tied to invariants and performance gates.

- **Accessibility scheduled inside the build:** Shipping reduced motion as a v1.1 fix would tell those users the product was not for them.

- **Team shape:** The writer is not optional because notebook and narration are the product's voice; the audio-capable engineer is specialist because DSP and call grammar are hard.

- **Accelerated drift harness:** It validates math quickly by running the pure tick core at high speed across archetypes.

- **Real-time internal alpha:** It validates the feel of drift, which no simulation can settle; wall-clock time is irreducible.

- **Internal debug client showing vectors:** It is limited to alpha because vectors are otherwise private, but calibration needs instrumentation.

- **Closed beta interviews:** The product learns about drift by asking people rather than instrumenting them like a normal beta.

- **Presence activity window beta validation:** Uses consenting beta users only and discards data after analysis.

- **Ramping birds-per-aviary with synthetic aviaries and panels:** The organic age gate is too slow to validate seven birds before GA; recognizability determines whether the cap holds.

- **Backdated beta accounts:** Let real people live with five-to-seven-bird aviaries before GA.

- **Cap enforced in engine:** The cap remains server-side and lowerable without a deploy.

- **Plain launch and no onboarding tour:** A product whose thesis is "notice, never announce" cannot open with a tour.

- **Adoption naming exception:** Naming two birds is functional and brief, dropping directly into naturalist voice.

- **Feature flags and kill switches:** They allow weather, notebook, visits, or bird offers to be disabled without deploy.

- **Rollback with expand/contract schemas and tick replay:** Bad tick versions can be rolled back and replayed from audit data.

### 15. Testing strategy

- **Simulation property tests:** They protect monotonicity, batched equivalence, purity, and idempotency: the core mathematical invariants behind drift and sync.

- **Calibration tests:** They enforce week-one measurable and week-three visible bands while preventing marathon-session budget violations.

- **Simulation fuzzing:** Malformed, replayed, out-of-order, and future events must not crash, create negative drift, or inflate presence.

- **Presence truth-table and hostile-client tests:** They enforce the three-signal conjunction and server-side cap because presence dominates drift.

- **Sync tests:** Concurrent clients must not write personality, double-count presence, or produce interleaving-dependent state.

- **Visitor tests:** Visitor events must be rejected and revocation must occur within one poll.

- **Rendering regression:** Cold load, quiet field, empty aviary, resume, time, weather, reduced motion, breakpoints, and focus states protect the visual and interaction surfaces most likely to regress.

- **No-bird-cropped assertion:** Preserves the layout rule that all birds remain fully in frame.

- **Audio determinism:** Same seed tuple must produce identical parameters and caption so audio and captions stay aligned.

- **Audio distinctness:** Generated calls should have near-zero collisions so no two calls are identical.

- **Audio identity preservation:** Classifier and panel tests proxy the recognizability requirement across moods and drift.

- **Mix tests:** Listen-in must not silence other birds and chorus must not clip.

- **Charm suite:** It exists because charm rules are likely to erode quietly; it turns document-only rules into merge-blocking checks.

- **Accessibility tests:** Automation covers axe, focus, contrast, cadence, and live-region politeness, while manual and external sessions validate qualitative naturalist narration.

### 16. Risks

- **Drift calibration risk:** Too fast becomes Tamagotchi, too slow becomes screensaver; harnesses, alpha, config weights, and replay mitigate it.

- **Silent sync correctness failures:** Lost deltas can make birds drift slightly wrong without alerts; audit, assertions, tick-lag alerts, and transactional apply treat vector anomalies as Sev-1.

- **Procedural calls uncanny valley:** Bad synthesis is worse than silence; mitigation is real DSP time, early prototype, panels, and if necessary fewer, sparser, quieter calls rather than recorded audio.

- **Accessibility regression:** Unusual surfaces can degrade quietly; parity tests, owner, PR question, and recurring sessions protect them.

- **500ms first-bird failure:** The quiet field is preferable to spinner compromise; mitigations protect the critical path.

- **Charm erosion:** Reasonable pull requests can add the toast, streak, cooldown timer, or unlock celebration; absent primitives, type impossibilities, and charm suite create visible friction.

- **Autoplay silence:** Mitigated without sound prompts.

- **Tick cost at scale:** Active/dormant split and batched equivalence allow rescheduling without behavior changes.

- **Seven birds blur chorus:** Individuation, mix ducking, and configurable cap keep recognizability empirical.

- **Naturalist corpus staleness:** Anti-repetition, corpus metric, quarterly authoring, and immutable entries preserve freshness over time.

- **Support cannot debug vectors:** Slower support is accepted so the no-vector-surface rule holds; user export is the route for actual vector access.

### 17. Calls made where the PRD is silent

- **Mood set, daily reset, presence window, device-union presence, settled lighting, Escape undo, listen-in decay, invisible cooldown, bird thresholds, new-bird presentation, nightjar starter exclusion, timezone debounce, solar anchoring, aviary timezone, Canvas2D, template grammar, bird buttons, visitor sessions, visitor polling, visit notification exception, no event-type telemetry breakdown, first-bird 90KB budget, event retention, export vector exception, internal debug client:** The plan states each call is defensible from the PRD's stated intent and cheap to revisit. Individual rationales are included in the relevant feature entries above.

### 18. Definition of done for v1

- **End-to-end v1 experience:** The rationale is the product promise compressed into one scenario: a user signs in, names two birds, sees motion within half a second, notices different behavior over days and weeks, can interact gently, reads notebook sentences about the aviary and "nothing about them," and receives the same three weeks via screen reader or reduced motion. The final standard is that "Nothing in the product has ever counted anything at them."
