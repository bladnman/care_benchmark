## System-level intent

1. **The aviary should be encountered mid-life, not introduced by product chrome.** This shows up in the invariant that the "First frame is the aviary mid-motion; no spinner, no entry animation," the boot path where loading is the "quiet field," the rule that the greeting "is the welcome," and the repeated ban on "toasts, banners, welcome text, badges."

2. **The product should notice-never-announce.** The plan uses this phrase directly for arrivals: an unfamiliar bird "lingering at the scene's back edge," one possible notebook observation, and no modal, badge, or "NEW!" It also shows up in the top bar having "four slots and no extension point" and in "no UI inside the scene."

3. **Canonical simulation is slow and authoritative; performance is fast and expressive.** The "render-pipeline boundary" is named as "the most important line in the system": server owns personality, mood, perch, weather, arrivals, notebook, accounts, and invites; client owns rendering, idle micro-motion, call scheduling, interpolation, captions, and narration, with "zero authority."

4. **Users send intent, never state.** The upward boundary is the "interaction event log," append-only and idempotent. The plan says clients send "observations of user intent" and the server decides what they mean, with "additive deltas; never absolute values."

5. **Absence never punishes.** The plan makes "Drift is monotonic toward expressive; absence never penalized" a product invariant. In drift, "Negative drift does not exist in the codebase," and ambient behavior after neglect is mapped through "recent presence context," not trait decay.

6. **The bird should change visibly without being explained numerically.** Raw trait values live only in `birds`, snapshots carry "derived, quantized parameters," and "Personality numbers never rendered in any UI." Behavior bands make drift "visible without being told."

7. **Naturalist voice belongs to aviary surfaces; matter-of-fact voice belongs to system surfaces.** The plan calls the voice split "load-bearing product structure": naturalist for aviary, notebook, narration, captions, offer prompts, and arrival surface; matter-of-fact for sign-in, sessions, errors, account, settings, accessibility, export, and delete.

8. **Accessibility is a designed surface, not a fallback.** The plan says accessibility is "shipped with v1, not after" and that a reduced-motion mode landing post-launch is "a launch failure by definition." Narration, captions, reduced-motion, keyboard focus, and contrast are all release-gated.

9. **Procedural identity matters more than asset playback.** Calls are procedural with "no recorded audio anywhere, ever"; a fixed `signature_seed` makes "Pip's call stays Pip's"; the same call AST drives synthesis and captions. Bird art and ambient effects are also procedural or compactly generated.

10. **Privacy is pipeline architecture.** The plan says the privacy boundary is structural: per-account interaction state exists only in the simulation database, aggregate telemetry has "no per-account dimensions," no ML path exists, and population-level analysis of bird interaction "does not exist."

11. **The system should stay deliberately small until the product proves it needs more.** The plan chooses "one modular-monolith API service," "No WebSockets at v1," no game engine, no React in the scene, and no LLM in prose paths. The reasons are small state cadence, 2MB budget, determinism, auditability, and removing capabilities the product "must not use."

12. **Continuity is protected.** Stable bird identity appears in `birds.id`, `signature_seed`, rename behavior, and `sim_constants`: tuning applies prospectively, and "no migration ever rewrites existing trait vectors" because "the bird the user knows is never 'recalibrated' out from under them."

## Per-feature whys

### 1. Scope

- **One aviary per account:** NOT RECOVERABLE FROM PLAN

- **Two starter birds and cap of seven:** The cap is a hard PRD constraint "built into the engine"; later the plan ties seven to empirical recognizability and says the cap stays engine-enforced while arrivals stop scheduling.

- **Single horizontal scene, one screen, no pan/zoom/scroll:** The plan frames this as the product surface: one place where every bird stays in frame, with no arranging birds and no scene customization.

- **Day/night cycle on the user's local time:** The rationale is that the aviary is keyed to the user's day, with night "alive, just quiet"; most birds settle while the nightjar-like species keeps night voiced.

- **Rare ambient weather:** Weather is server-canonical so both devices and visitors see the same rain, and it remains "never assertive" so the place stays gentle.

- **Ambient leaf/feather drift:** The plan calls this "pure rendering ornament, no sim state," giving motion to the scene without adding canonical mechanics.

- **Hidden five-trait personality vector:** The rationale is enforceability of the no-numeric-exposure rule: the client receives derived behavior parameters, not raw traits.

- **Fast-timescale mood:** Mood makes current expression legible through motion, posture, calls, and perch behavior, while persisting across sessions and never snapping on tab-open.

- **Presence-driven monotonic drift:** The product wants weeks-scale change from presence, with "absence never penalized" and no Tamagotchi-style decay.

- **Procedural per-bird call signatures:** The plan uses procedural calls to avoid recorded audio, keep every call fresh, and preserve recognizable bird identity.

- **Bird-to-bird interaction:** The rationale is within-session liveness without giving clients authority: durable effects stay canonical, while call timing and head-tilts happen instantly on the client.

- **Return-greeting:** The greeting is the welcome, replacing all textual welcome surfaces, and must land in 1-2s faster than a server round trip.

- **Idle presence as a first-class interaction:** The plan says "sitting still and watching is the product," so presence credit uses a long activity window and does not require constant clicking.

- **Listen-in:** The mix is a "re-balance, not a mute," because the aviary is "a place, not soloable tracks."

- **Offer:** Offers provide quiet, mood-shaped stimuli from the top bar, not by grabbing birds; cooldowns are "functional, not punitive" so curiosity cannot saturate in one session.

- **Settle with 5-second undo:** Settle ends the presence window cleanly and quiets mood; undo exists as "accidental-click mercy," while closing the tab is engine-equivalent and never scolded.

- **Field notebook:** The notebook is "observations, not a feed," sparse, read-only, naturalist, and based on aviary-state inputs rather than observations of the user.

- **Email magic-link sign-in:** The route always returns `202` to avoid account enumeration, creates accounts lazily, and uses short-lived single-use links.

- **Per-device revocable sessions:** The rationale is session control from settings, with origin checks authoritative immediately and edge propagation within seconds.

- **Email change with verification:** NOT RECOVERABLE FROM PLAN

- **JSON export by emailed link:** The plan treats export as a data-ownership artifact in the system register, with a verified email and seven-day link.

- **Soft deletion then hard deletion:** Soft delete gives a restore window; hard purge removes birds, vectors, events, notebook, invites, visit logs, sessions, KV snapshots, and export files, then verifies deletion.

- **Server-side simulation tick as only writer:** This avoids client mutation and last-write-wins conflicts; multi-device coherence is "a property of the architecture."

- **Visit invitations:** Visits are opt-in, read-only, revocable, expiring, and off by default to avoid social-network surfaces and prevent visitor presence from producing drift.

- **Visit log:** The log is on-demand only and has "no badges anywhere," preserving the no-announcement posture.

- **Accessibility shipped with v1:** The plan says accessibility "not after" is an in-scope requirement and makes reduced-motion, narration, captions, contrast, and keyboard navigation launch work.

- **Performance launch gates:** The budgets are treated as launch gates because first-bird timing, 60fps, no memory growth, and tick latency are product requirements, not optimizations.

- **Browser support for last two major versions of Chrome, Safari, Firefox, Edge:** NOT RECOVERABLE FROM PLAN

- **Unsupported-browser surface:** The rationale is register consistency: older browsers get a "matter-of-fact unsupported-browser surface."

### 2. System architecture

- **Modular-monolith API service:** The plan calls the system "deliberately small"; module boundaries inside the monolith are future seams without v1 microservice cost.

- **Tick-worker fleet:** Tick workers own canonical simulation progress and write snapshots, keeping the server side as the single source of personality, mood, perch, and weather.

- **Postgres as canonical store:** Postgres holds canonical state, event log, notebook, and invites; the plan's sync model depends on "one canonical record."

- **Redis for scheduling, rate limits, and queues:** Redis supports tick scheduling, rate limiting, and job queues needed by the monolith and tick fleet.

- **Edge layer with snapshot inlining:** Edge delivery validates the session cookie locally and inlines a snapshot to hit the first-bird path without an origin round trip.

- **Render snapshot crossing downward:** The snapshot is small, cacheable, and carries positions, moods, derived behavior parameters, weather, transitions, and seeds so clients can perform without authority.

- **Interaction event log crossing upward:** Events are append-only, typed, and idempotent so clients express intent while the server interprets and folds additive deltas.

- **Client autonomy on stale snapshots:** Because the performance layer elaborates the latest snapshot, a server blip becomes "slightly behind," not "frozen."

- **TypeScript end-to-end:** Shared `voice-grammar`, `call-grammar`, and boundary types remove drift bugs between server, notebook, captions, narration, and synthesis.

- **Canvas 2D for the scene:** The plan chooses Canvas 2D because the scene is at most seven birds and light particles, and a smaller kernel serves the 500ms budget.

- **Preact for chrome surfaces:** Preact keeps account, settings, notebook, and auth chrome code-split out of the boot kernel.

- **Provider-agnostic email adapter:** Email is behind one interface so Postmark or SES can serve the same auth, export, and visit flows.

- **No WebSockets at v1:** State changes at tick cadence; polling with jitter, visibility refetch, and frame-gap refetch meets behavior while staying edge-cacheable and avoiding a stateful fleet.

- **No game engine and no React in the scene:** An engine would spend the 2MB budget on physics, cameras, and tweening UI capabilities the product must not use.

- **No LLM in notebook/narration:** The rationale is deterministic, auditable, writer-owned output with zero marginal latency, zero marginal cost, and no third-party data flow.

### 3. Data model

- **Email stored once, encrypted, with blind index:** This enforces the synthetic ID rule: email is used solely for sign-in lookup and never becomes a reference, log field, or cross-system identifier.

- **Stable bird IDs:** The plan marks bird identity as stable for the life of the account, protecting continuity across rename, mood, and drift changes.

- **Server-only raw traits in `birds`:** Keeping traits only in `birds` and out of client payloads enforces the no-numeric-display rule at the data boundary.

- **`signature_seed` fixed at adoption:** The fixed seed makes call signature identity stable even as mood and drift modulate expression.

- **Append-only monthly partitioned events:** The log is used for tick consumption, not replay; old partitions drop after 90 days because vectors are canonical.

- **Notebook entries retained for account lifetime:** The user has unbounded scroll-back, and entries are immutable observations rather than an editable feed.

- **Arrivals table:** The table supports the age-driven, lingering/adopted/departed arrival mechanic without visit-, payment-, or interaction-driven unlocks.

- **Invite and visit-session tables:** These support revocable, expiring, read-only visits and an on-demand log with approximate duration.

- **Species catalog as versioned code config:** NOT RECOVERABLE FROM PLAN

- **Edge KV snapshots:** Tick writes snapshots to edge KV so the bootstrap path can inline state quickly.

- **Revoked-sessions KV set:** This gives seconds-level revocation propagation at the edge while origin enforcement remains strict.

### 4. API surface

- **Matter-of-fact error bodies with stable codes:** System surfaces use the matter-of-fact register and tell the user what happened and what to do.

- **Custom header on mutating routes:** The custom header is "CSRF belt-and-braces" alongside SameSite cookies.

- **Magic-link consume:** Atomic single-use consume prevents replay, and expired or used links resolve to a matter-of-fact error page.

- **Account settings:** Settings carry captions, reduced motion, narration, visit notifications, and sound preference; device-level reduced motion wins to respect the device signal.

- **Snapshot ETag equal to `tick_seq`:** The ETag makes 304 polling cheap at tick cadence.

- **Snapshot excludes raw traits:** The plan says this makes the no-numeric-exposure rule enforceable at the API boundary and blunts third-party "bird stats dashboard" tooling.

- **Snapshot `active_transition`:** This lets a freshly opened client render a bird mid-flight instead of teleported.

- **Batched idempotent event ingest:** Stable client event IDs allow at-least-once retry with exactly-once effect, including `sendBeacon` on pagehide.

- **Server-side event validation and caps:** Validation and saturation mean a tampered client can only distort its own aviary, and even then boundedly.

- **Rename any time:** Rename has no effect on personality, mood, or call, preserving bird identity and avoiding recalibration through naming.

- **Arrival adopt route:** The route implements the quiet adopt surface for the bird that arrived, rather than a species catalog.

- **Visit invite creation:** Invites are emailed one-time links and stay off by default so nothing in onboarding turns the product into a social surface.

- **Invite revocation:** Revocation is immediate, giving hosts control over read-only access.

- **Visit snapshot using host timezone:** The visitor sees the host's day/night cycle, preserving the host aviary as the viewed place.

- **Visitor cookies with no event-ingest capability:** The routing deny is structural: visitor watching creates no presence, no drift, and no events.

### 5. Simulation engine

- **Transactional tick:** A tick reads state and events, folds accumulators, advances processes, writes canonical state, and pushes a snapshot atomically so retries are deterministic.

- **Event consumption cursor:** Cursor-based consumption means a crashed tick re-reads the same events and produces the same deltas.

- **Redis sorted-set scheduling:** Workers claim due aviaries while hash sharding keeps per-aviary ordering.

- **Adaptive cadence:** Continuous-time drift, mood, and weather let dormant accounts advance without burning CPU every minute.

- **Wake-on-read:** Any snapshot request or event ingest enqueues an immediate tick so a returning user's first pull reflects a just-ticked aviary.

- **Dormant notebook moments:** Sparse dormant observations let the user see that the aviary continued, which the plan calls "the product's central conceit doing work while nobody watches."

- **Versioned `sim_constants`:** Constants can be tuned prospectively while preserving identity continuity and avoiding rewrites of existing trait vectors.

- **Presence ping conditions:** Visibility, focus, and recent activity are all required so a laptop left open accrues nothing while still honoring quiet watching.

- **Five-minute activity window:** The plan leans long because "sitting still and watching is the product."

- **Union-bucketed presence:** Multiple devices in the same wall-clock bucket accrue presence at 1x, closing multi-device drift inflation.

- **Gap merging:** Gaps under 90s are bridged so dropped requests do not chop presence.

- **Hard presence cap:** Credited presence cannot exceed elapsed wall-clock, structurally limiting tampering and double credit.

- **Exponential monotonic drift formula:** Traits approach 1 without decreasing, making expression grow while never punishing absence.

- **Daily presence saturation:** Presence saturates around the design calibration so camping a legitimate tab cannot exceed the intended curve.

- **Listen-in drift credit:** Listen-in is a "strong attention signal" for social warmth and vocal frequency, capped per bird per day.

- **Offer drift credit cooldown:** The cooldown lets reactions still happen while preventing curiosity from saturating in one session.

- **Settle with zero drift direction:** Settle ends presence and quiets mood without changing trait direction.

- **Plumage saturation drift:** Plumage becomes richer through sustained attention and never moves down.

- **Behavior bands with hysteresis:** Bands make drift visible without numbers and avoid flicker at boundaries.

- **Per-session trait clamp:** No single session can shift a trait visibly, preserving weeks-scale change.

- **Calibration harness and dogfood only:** Calibration avoids production aggregates because the privacy commitment forbids population-level drift analysis.

- **Continuous-time mood machine:** Minimum dwell and hazard integration prevent flapping and preserve mood continuity across session boundaries.

- **Local-time mood modulation:** Dusk, morning, and night shape mood so the aviary follows the user's day without snapping open.

- **Weather schedule:** Rare rain and wind are gentle, canonical, and shared across devices and visitors.

- **Perch selection:** Perch is a signal the user reads, not a control; boldness, mood, social warmth, and inertia make it expressive.

- **Greeting selection:** Derived `greet_weight`, absence tiers, deterministic seeding, and staggered greeters make the return beat fast, varied, and not a unison chorus.

- **`greeting_performed` event:** The server records what happened for notebook history, but greeting gives no drift because it is system behavior, not user input.

- **Bird-to-bird split:** Canonical mood/proximity effects stay on the tick, while response timing and head-tilts stay instant on the client.

- **Age-driven arrivals:** New birds arrive by aviary age only, not visits, interactions, or payment, and ignored arrivals depart without penalty.

- **Lingering unfamiliar bird arrival UX:** The plan chooses in-scene lingering plus a quiet naturalist surface to honor "notice-never-announce."

- **Nightjar bias by arrival 3-4:** The soft bias "gives night a voice" if the aviary lacks the night-active species.

- **Starter bird naming:** Starters are "the birds that arrived"; the user names them, and rename remains separate from personality, mood, call, and id.

- **Notebook detectors:** Detectors use aviary-state inputs only because the line is "observations of the aviary, never of the user."

- **Notebook sparsity controller:** Token buckets and detector cooldowns keep the notebook as observations, not a feed.

- **Notebook prose generation:** Shared voice grammar gives lowercase, present-tense, specific, bird-named prose with no trait numbers or gamification lexicon.

### 6. Sync model

- **One canonical record:** Both devices and visitors read the same Postgres-backed aviary; the plan says "there is nothing to sync."

- **Single writer enforcement:** Code structure, DB trigger, role-based access, and audits keep personality, mood, perch, and weather owned by the tick.

- **No last-write-wins:** Events fold in sequence order, so laptop and phone events combine instead of overwriting.

- **Idempotent ingest:** Stable `(aviary_id, client_event_id)` uniqueness gives exactly-once effect under retry.

- **Snapshot freshness refetches:** Visibility, frame-gap, and jittered polling bound staleness to tick cadence plus one poll interval.

- **Presentation-only reconciliation:** Birds glide, cross-fade, or blend toward snapshots without preserving client simulation state.

- **Server-relative clocks:** Snapshot server time keeps greeting absence and sim-relative choices from trusting local wall-clock.

- **Matter-of-fact conflict surfaces:** Auth races, expired links, revoked sessions, and outages stay out of aviary chrome and use system copy.

### 7. Frontend rendering pipeline

- **Edge-inlined bootstrap snapshot:** This buys the "first bird visible <500ms" path by avoiding an origin round trip before first paint.

- **Boot kernel under 150KB target:** The kernel contains only scene, renderer, puppet, pose decode, and day/night LUT so first paint is small.

- **Prebaked compact pose atlas:** The atlas lets first paint show birds in snapshot poses without waiting on procedural art.

- **Greeting after first paint:** The first 1-2s "noticed you" beat happens as the greeting, not as text or animation chrome.

- **Lazy audio, voice, caption, and chrome loading:** Nonessential systems load after first paint or first intent to protect the boot budget.

- **Quiet field degraded path:** KV miss or slow origin gives a quiet scene with no spinner or progress text, preserving the no-announcement invariant.

- **Layered Canvas 2D renderer:** Separate canvases, caching, dirty-region skips, and capped DPR support the 60fps and memory gates.

- **Graceful degradation ladder:** Under jank, particles and parallax drop before bird motion; "Bird motion degrades last."

- **Hidden-tab behavior:** Rendering, audio, and presence naturally stop when hidden while the server simulation continues.

- **Responsive layout solver:** The solver keeps all perch zones and every bird in frame, with no cropped bird.

- **Parametric bird puppet:** Vector descriptors and palette interpolation make plumage drift visible without image sets.

- **Idle micro-motion generators:** Mood is legible from motion alone; the plan explicitly forbids mood labels, tooltips, and status icons.

- **Never-still awake birds:** Generator amplitude never reaches zero so the scene does not read as paused.

- **Perch transitions from `active_transition`:** All clients render the same hop or flight instead of inventing divergent moves.

- **Day/night render system:** Palettes and activity changes make local time visible while keeping night alive and quiet.

- **Weather render:** Rain and wind are light visual/audio modifiers; reduced-motion turns weather into slowed palette shifts and audio only.

- **Top bar only chrome:** Four fixed slots and no extension point enforce "nothing else, ever" and prevent product-surface creep.

- **Top bar fade:** Fading to low opacity reduces chrome presence while focus and keyboard reachability remain intact.

- **Empty-aviary state:** The first bird's soft fly-in appears after adoption and is never reused once the aviary exists.

- **Reduced-motion register:** Reduced motion keeps calls, captions, narration, notebook, drift, and mood unchanged while changing the motion driver.

- **Listen-in interaction:** Focus a bird to rebalance audio with only a subtle attention cue, avoiding gamey highlight language and behavior.

- **Offer interaction:** Offers originate from the top bar and use soft-disable pacing without countdown UI or scolding.

- **Settle interaction:** Settle quiets the scene until tab-close or re-engagement, with undo on any aviary click within five seconds.

- **Notebook UI:** The notebook is a code-split, read-only, virtualized panel to preserve the memory rule.

### 8. Audio pipeline

- **AudioWorklet synthesis:** Synthesis runs off the main thread so the render loop is unaffected and no audio files are needed.

- **Pre-allocated voice pool:** The pool gives zero per-call allocation, supporting the no-memory-growth rule.

- **Procedural ambience bed:** The low air/leaf layer gives "quieting to ambient" a real floor instead of silence.

- **Per-species motif library:** Authored motif atoms give the sound designer control over stylized call character.

- **Phrase grammar:** Fresh generated ASTs avoid loops, fixed stored sequences, and phase-cancel artifacts.

- **Stable signature seed:** The seed keeps the signature core recognizable across mood, drift, and renames.

- **Shared call AST for synthesis and captioning:** Captions describe the call that actually played "by construction."

- **Client call scheduler:** Poisson-ish onsets, refractory periods, and snapshot modulation make calls feel alive while respecting time of day and weather.

- **Emergent chorus:** Chorus is overlapping independent procedural voices, which the plan says is "what makes it a chorus."

- **Deterministic scheduler seed:** Same-account devices stay in approximate macro-agreement.

- **Listen-in mix:** Focused birds rise while others fall toward, but never to, the ambient floor because the place should not become soloable tracks.

- **Audio gesture-gating:** The plan reconciles browser policy with design intent through visual-first aliveness, fade-in on first gesture, and no "click to enable sound" banner.

- **Captions default before audio unlock:** Captions make calls perceivable while audio is unavailable or still gesture-locked.

- **WebAudio-unavailable path:** Silence plus captions is chosen because recorded-audio fallback is forbidden and "silence with captions beats canned audio."

### 9. Accessibility surfaces

- **Screen-reader narration:** Narration is running naturalist prose from the same state visuals render, making the same product available "in the ears as in the eyes."

- **Single polite ARIA live region:** Swap-replace avoids queue flooding while still providing current observations.

- **Narration cadence:** Idle updates every 45s plus user-event jumps satisfy the 30-60s intent without becoming announcements.

- **Semantic chrome surfaces:** Settings, notebook, auth, and top bar use conventional landmarks, headings, and labels in the appropriate register.

- **Reduced-motion activation:** OS `prefers-reduced-motion` wins; account opt-in covers users whose OS setting is unavailable and syncs across devices.

- **Call captions:** Runtime AST-derived captions match the actual call and provide authored variation instead of fixed strings.

- **Caption scrim chip:** The chip is a bounded exception to no scene chrome because AA contrast over arbitrary scene pixels is otherwise unmeetable.

- **Keyboard navigation:** Tab, arrows, Enter, Escape, focus trap, and top-bar reachability implement the PRD interaction path without pointer dependence.

- **Focus indicator:** High-contrast outline plus halo is tested across bright, dim, and settled palettes.

- **Contrast and CI:** Automated AA checks, axe-core, and manual screen-reader matrix make accessibility a release gate.

### 10. Voice and copy system

- **Register field per surface:** The mandatory register keeps naturalist and matter-of-fact voice separated by whether the user engages the aviary or the system as a system.

- **Naturalist register:** Lowercase, present-tense, specific, bird-verbed copy prevents "you"-announcements and supports the observational product voice.

- **Matter-of-fact register:** Identity, errors, settings, export, delete, and revoked visits state what happened and what to do.

- **Shared `voice-grammar`:** One package generates notebook, narration, and captions so voice continuity is testable across surfaces.

- **Writer-authored corpus:** Writer ownership, reviewable files, and seeded golden tests make phrasing changes deliberate.

- **Anti-repetition memory:** Surface-form hashes enforce no verbatim repeats within an aviary.

- **Copy lint:** CI fails on gamification lexicon, second person in notebook/narration, exclamation marks, uppercase naturalist starts, and unregistered copy strings.

- **Append-only banned lexicon:** Writer ownership and append-only growth prevent well-meaning contributors from adding forbidden product shapes.

- **Vocabulary in code:** Identifiers use `call`, `listen_in`, `offer`, `settle`, `presence`, and `visit` because "teams build the product their identifiers describe."

### 11. Security and privacy engineering

- **Synthetic ID rule:** UUID-only references, encrypted email, blind index, log scrubbing, and canary tests prevent email from becoming a system identifier.

- **Magic-link security:** High-entropy tokens, hashes at rest, 15-minute TTL, atomic consume, and rate limits prevent replay and enumeration.

- **Revocable sessions:** Hashed tokens and an edge-verifiable wrapper support fast bootstrap plus user-controlled revocation.

- **Visitor token namespace:** Separate visitor credentials and routing ensure read-only visits cannot call account APIs or event ingest.

- **Privacy boundary as pipeline architecture:** The analytics store has no network connection to the sim DB, telemetry has no per-account dimensions, and no ML path receives per-bird fields.

- **No leaderboard-able data:** The plan structurally forecloses leaderboards by never computing cross-account bird or interaction aggregates.

- **Export job:** Export sends a JSON file to the verified email because it is a data-ownership artifact, not a product display surface.

- **Deletion lifecycle:** Hard purge includes canonical state, logs, visits, sessions, KV, and export files, then records verification.

- **Strict CSP with no third-party scripts:** Product privacy and surface integrity are protected by keeping third-party scripts out entirely.

- **Least-privilege DB roles:** The trait-write role belongs to the tick alone, reinforcing the single-writer boundary.

### 12. Performance budgets and observability

- **Initial JS bundle budget:** The <2MB hard budget and boot-kernel target protect the first-bird path.

- **First bird visible budget:** The 500ms gate exists because the first frame must be the aviary mid-motion, not loading chrome.

- **Idle frame-rate budget:** Sustained 60fps for the worst scene keeps bird motion alive over a 30-minute session.

- **Client memory budget:** The no-growth gate drives pooling, virtualized notebook, and allocation discipline.

- **Tick latency alarm:** p99 at 5s and tick-lag alarms catch server delay before users notice.

- **Snapshot payload cap:** Small snapshots keep polling and edge bootstrap cheap.

- **Bundle discipline:** Procedural art/audio, code-splitting, Preact, and no third-party UI/analytics SDKs buy the budgets.

- **Synthetic monitoring:** Scripted browsers measure cold load, first-bird render, frames, audio errors, and snapshot latency from several geographies.

- **Aggregate-only RUM:** RUM stays operational and sampled, with no per-account dimensions.

- **Server observability:** Request health, tick health, event validity, email success, queues, and anonymized session-duration histograms support operations.

- **Deliberately unmeasured analytics:** Retention cohorts, engagement funnels, visit frequency, per-bird analytics, drift distributions, and A/B engagement experiments are excluded as a product and privacy decision.

### 13. Testing and quality strategy

- **Auth, ingest, visits, and lifecycle tests:** These cover expiry, replay, dedupe, authz, revocation, export, restore, and purge because they are security and data-boundary flows.

- **Simulation property tests:** The plan calls them "the engine's spine," proving monotonic drift, trait bounds, clamps, idempotent re-tick, presence caps, cadence equivalence, mood dwell, and no mood snaps.

- **Calibration harness:** The harness is the only calibration instrument besides consenting dogfood accounts, satisfying privacy while making week-scale drift testable.

- **Performance CI:** Bundle gates, throttled cold load, frame tests, and memory soaks enforce the launch budgets.

- **Cold-load visual regression:** The first frame must contain birds mid-pose and quiet-field fallback must contain no spinner pixels, directly testing product invariants.

- **Audio tests and listening protocols:** Offline render tests catch technical errors, while uncanniness and recognizability panels test the affective spine and the empirical seven-bird cap.

- **Voice and accessibility tests:** Golden outputs, lints, axe, contrast, keyboard tests, manual SR matrix, and reduced-motion regression keep the designed surfaces intact.

- **Security/privacy tests:** PII canary, authz matrices, log-scrub verification, telemetry denylist, and deletion E2E enforce the privacy architecture.

- **Chaos/sync tests:** Two-device storms, retries, out-of-order ingest, and tick-fleet kill/resume prove no lost deltas, duplicate effects, or single-writer erosion.

### 14. Rollout

- **M0 foundations and spikes:** Audio uncanniness and 500ms boot are "two scary spikes" and go/no-go gates before architecture hardens.

- **M1 vertical slice:** A dev sits with one bird for ten minutes to test whether it "feels alive" before accounts exist.

- **M2 accounts and sync:** Accounts, multi-device, offers, settle, starters, species, perch, day/night, and weather come once the vertical slice exists.

- **M3 voice and accessibility:** Voice, notebook, narration, captions, reduced-motion, keyboard, and SR pass land pre-beta because accessibility is launch work.

- **M4 visits and lifecycle:** Visits, export, deletion, unsupported browser, security review, and privacy audit close the social and data-lifecycle surfaces.

- **M5 calibration and hardening:** Drift needs weeks of wall-clock, so calibration starts early and continues through dogfood, beta, perf gates, and listening tests.

- **Ramping birds-per-aviary:** Age-driven arrivals make load grow naturally from two-bird aviaries toward five to seven over the first year.

- **Closed beta posture:** Invitation-only beta, on-call, runbooks, and no product marketing surfaces keep launch matter-of-fact.

- **Instrumentation from day one:** Observability is live from M1 because retrofitting it after calibration starts would leave drift work blind.

### 15. Risks

- **Drift calibration mitigation:** Harness assertions, behavior bands, consented dogfood, conservative-slow shipping, and prospective constants address the narrow band between Tamagotchi and screensaver.

- **Procedural-call mitigation:** The M0 sound-designer spike, stylized direction, uncanniness panels, recognizability protocol, and low-end perf floor address synthetic call risk.

- **Autoplay mitigation:** Visual-first aliveness, first-gesture fade-in, captions default, and no banner address browser policy without violating no-announcement surfaces.

- **First-bird performance mitigation:** Edge snapshots, edge-local cookie validation, small kernel, pose atlas, CI, and device lab address the 500ms risk.

- **Sync correctness mitigation:** Single-writer layers, cursor-ordered ingest, chaos tests, and monotonicity audit guard against silent lost or doubled drift.

- **Presence-inflation mitigation:** Three-condition pings, union bucketing, gap rules, hard caps, saturation, and self-harm-only tampering protect calibration.

- **Memory-growth mitigation:** Pools, virtualized notebook, and CI heap assertions target the hard 30-minute no-growth gate.

- **Accessibility regression mitigation:** Pre-beta milestone placement, shared voice engine, golden tests, SR matrix, and reduced-motion visual suite prevent checklist-only parity.

- **Tick fleet cost mitigation:** Continuous-time formulation, adaptive cadence, wake-on-read, and tick-lag alarms avoid waste without stopping simulation.

- **PII leakage mitigation:** UUID-only references, blind index, log scrubbing, CI canary, and telemetry denylist address email and trait leaks.

- **Voice/gamification creep mitigation:** No toast primitive, no announce pipeline, banned lexicon, register-typed copy, writer ownership, and design checklist block "just a small toast."

- **Notebook repetition mitigation:** Large corpus, anti-repetition memory, sparsity, writer-reviewed goldens, and cheap corpus growth reduce templated prose over months.

### 16. Decision ledger

- **Canvas 2D over WebGL/engine:** The plan says the scene fits Canvas 2D, and the smaller kernel serves the 500ms budget.

- **Polling, no WebSockets:** Tick-cadence state and next-pull revocation do not need sub-tick push.

- **Audio gesture-gating:** Platform reality constrains "calls already audible"; the closest conforming behavior is visual aliveness, fade-in, captions, and no banner.

- **Arrival UX for birds 3-7:** In-scene lingering plus one notebook observation and quiet adoption honors "notice-never-announce" and no-catalog adoption.

- **Authored grammar, not LLM:** Determinism, auditability, writer ownership, zero latency/cost, and no third-party data flow justify the corpus work.

- **Raw traits never serialized:** Derived behavior parameters buy API-level enforcement of no numeric exposure.

- **Export includes personality vectors:** The plan resolves the tension by treating export as a data-ownership artifact in the system register, flagged for product sign-off.

- **No production drift monitoring:** The privacy commitment outranks calibration convenience.

- **Visit link semantics:** Single consumption binds one browser, reconciling "one-time" with revocable active invites and a multi-visit log.

- **Caption scrim chip:** Accessibility wins the tie because AA contrast over arbitrary scene pixels is otherwise unmeetable.

- **Low ambience bed:** The quiet procedural bed gives listen-in's ambient floor something real and is trivially removable.

- **Numeric defaults:** Defaults are all `sim_constants`-tunable unless the plan names them as hard PRD numbers.

### 17. Team and sequencing

- **Seven engineers plus part-time craft roles:** Staffing is mapped to architecture seams: rendering, audio, simulation, accounts/lifecycle/visits/edge, accessibility/design systems/voice, visual design, sound design, and writing.

- **Critical path:** Audio uncanniness, 500ms boot, felt-aliveness, wall-clock drift validation, and listening-test iteration start early because drift validation and listening iteration "cannot be compressed."
