## System-level intent

1. **The aviary is one continuing place, not a session-bound app.** The plan names this as load-bearing: "The aviary continues without the viewer." It appears in the architecture and simulation sections as "server-side tick, canonical state, no entry animation, no spinner," in the cold-aviary read path where a returning user gets an aviary that "has been running for two weeks," in sync as "two readers of one row," and in the first-frame rule that the scene is "already in motion."

2. **Server authority protects the fiction and the data.** The plan's contract is: "clients send observations of the user; servers send observations of the aviary." This shows up in the client/server split, additive server-authored deltas, append-only event log, tick-only writes to traits, no client-submitted absolute values, no visitor POST verbs, and the rule that the client may extrapolate for rendering but never owns canonical state.

3. **Drift is monotonic toward expressive, never regression.** The plan calls this load-bearing and repeats that neglect produces "quietness, never regression." It is enforced in the drift function, the `sim.bird_traits` database trigger, calibration tests, and the risk correction path where a too-fast calibration is slowed going forward rather than rolling traits back.

4. **Notice, never announce.** The plan treats this as product voice and as a CI invariant: no toast, badge, streak, counter, welcome banner, friend-visited ping, assertive live region, sound prompt, spinner, or progress bar. It appears in the field notebook, greeting, failure surfaces, autoplay handling, third-bird offer, product-invariants suite, and the repeated choice to render observations rather than announcements.

5. **Personality is hidden and only expressed through consequences.** The plan says personality is "never computed, held, transmitted, or written by any client" and that raw traits are "not in a stats panel, not in a debug view." This drives the snapshot schema, derived and quantized render parameters, bundle checks that keep `sim/drift` off the client, no mood labels, no personality endpoint, and the single authorized export path to the data subject.

6. **Identity is stable; expression changes around it.** The call system states the "recognizability invariant": drift and mood "modulate expression; they never touch identity." This shows up in frozen bird ids, frozen `identity_seed`, additive species config, per-bird call signatures, spectral-fingerprint tests, and the bird-count ramp being gated on recognizability.

7. **The accessible surface is the actual product.** The plan says accessibility is not a variant and that the narration, captions, and visual renderer read the "same interpolated scene state object." This principle appears in the shadow accessibility tree, naturalist narration, captions generated from actual call parameters, full keyboard navigation, contrast gates, and reduced-motion as a "designed register" rather than a degraded fallback.

8. **Privacy boundaries are architectural, not procedural.** The plan repeatedly turns privacy claims into unrepresentable types, separated planes, revoked grants, no network path to `sim-db`, no per-account metrics, and shadow accounts for validation. It uses phrases like "structurally incapable" and says the telemetry plane is "physically separate."

9. **Sparsity and restraint are part of the product.** The plan treats quietness as a feature: the notebook is "deliberately stingy," the new-bird offer is quiet, weather is capped and "never assertive," transient failures show nothing, top-bar chrome fades, and the no-gamification list refuses metrics that would make ranking or streaks possible.

10. **Engineering choices are mechanized as tests, not reviewer discipline.** The plan says "Non-goals are not documentation; they are tests." This shows up in product invariants, deterministic-kernel tests, fast-forward equivalence, calibration lab, notebook lint, bundle scans, contrast checks, visitor-write-path proof, memory soaks, and alarms on impossible events.

## Per-feature whys

### 0. Reading this plan

- **Load-bearing callouts.** The plan calls out three requirements up front because "getting them wrong invalidates the rest of the build" and "no section of this plan is read in isolation from them."

### 1. Scope

- **Single-user account, email + magic-link sign-in, sessions, email change, export, delete.** The plan gives implementation and privacy details, but a specific product rationale for choosing these account features is NOT RECOVERABLE FROM PLAN.

- **One canonical aviary per account.** The rationale is sync simplicity and product coherence: "two readers of one row," "one aviary per account," and the settle open question says per-device settle "would make the aviary two aviaries."

- **2 starter birds at adoption.** NOT RECOVERABLE FROM PLAN.

- **Hard cap 7.** The cap exists "to protect the per-bird relationship"; if recognizability degrades, the plan would lower the cap rather than "ship 7 muddy birds."

- **Age-gated new-bird offers.** The plan says new birds are offered on "aviary age — not visits, not interactions, not a paid tier" and that the slow ramp gives time for recognizability studies before users hear larger choruses.

- **5-trait hidden personality vector.** The rationale is to drive long-term expressive consequences while keeping personality hidden: traits are never exposed numerically and only appear as motion, calls, plumage, greeting, and curiosity consequences.

- **Monotonic drift.** The plan's rationale is the asymmetry of neglect: birds stop becoming "more expressive but lose nothing"; no user sees a bird regress.

- **6-state mood FSM.** The rationale is that mood must be server state that keeps advancing during absence, with no "session start" reset, and must be read from motion rather than labels.

- **Bird-to-bird interaction.** The plan says call-and-response makes "the aviary a small social system rather than a row of independent NPCs."

- **Procedural call grammar.** The rationale is recognizability plus variation: a user should know Pip's call by ear, while 1,000 greetings should produce real variation rather than a rotation.

- **Mood-shaped idle motion.** The plan makes motion the delivery mechanism for mood: "Mood is legible from motion alone" and no label, tooltip, icon, or status indicator appears.

- **Server-side tick.** The rationale is that the aviary "continues without the viewer" and that a returning user gets a place that has been running, not one that resumes.

- **Append-only interaction event log.** The rationale is exact-once, replay-safe ingest and calibration auditability while keeping retention short enough for the privacy posture.

- **Server-authored additive deltas.** The rationale is to make stale client overwrites unreachable; clients submit observations and the server decides what they mean.

- **Return-greeting.** The plan says it is weighted because interactions warns against compressing it into "play arrival animation"; it should show the aviary continuing and noticing absence without announcing arrival.

- **Presence accounting.** The rationale is to credit actual continuous attention while preventing multi-device double-credit; the plan calls presence union "the single most consequential detail" and "the single most important line of sync code."

- **Listen-in.** The rationale is a re-balance, not a mute: unfocused birds are lowered "never to silence" so the aviary does not become "a set of soloable tracks."

- **Offer: seed / song fragment / still pool.** The plan specifies offers and cooldowns, but the specific rationale for these three offer types is NOT RECOVERABLE FROM PLAN.

- **Settle with +5s undo.** The plan says settle is "mood-quieting only" with no drift direction, but a specific rationale for the settle feature and 5-second undo is NOT RECOVERABLE FROM PLAN.

- **Field notebook.** The rationale is observational naturalist prose about the aviary, not the user; sparsity is the feature, with a target cadence that prevents the notebook from becoming a feed.

- **Single horizontal non-scrolling scene.** The rationale is to keep the aviary one place: birds stay fully in frame, are never cropped, and the scene is never panned, scrolled, or zoomed.

- **Three perch zones.** NOT RECOVERABLE FROM PLAN.

- **Local-time day/night.** The rationale is that the aviary should match local time while avoiding geolocation: local time is needed, "not location."

- **Ambient weather.** The rationale is device agreement for weather and quiet atmosphere: server-authored so devices agree, capped so it is "never assertive."

- **Ambient micro-motion.** The rationale is that the scene is already alive and never looped; no two birds are in step and no bird repeats a cycle.

- **Fading top bar.** The rationale is no in-scene chrome and restraint, while preserving accessibility: it honors "nearly transparent" but keeps glyph contrast at least 3:1.

- **No in-scene chrome.** The rationale is to keep controls above the aviary scene and avoid turning the place into an instrumented UI surface.

- **Visit invitations by email.** The plan gives behavior and abuse controls, but a specific rationale for email invitations is NOT RECOVERABLE FROM PLAN.

- **Read-only ambient visitor view.** The rationale is that visitors should see the same ambient aviary with "no show-off mode" and no code path into drift.

- **Revocation and 30-day invite expiry.** The rationale is abuse control and host control: immediate revocation at the visitor's next pull, scoped tokens, and expiry.

- **Visit log.** The plan includes approximate duration buckets, but a specific rationale for having the log itself is NOT RECOVERABLE FROM PLAN.

- **Off-by-default visit notification toggle.** The plan names the setting, but a specific rationale is NOT RECOVERABLE FROM PLAN.

- **Naturalist screen-reader narration.** The rationale is that the accessible surface is the actual product: running naturalist prose, not a state list, in the same lowercase present-tense voice as the notebook.

- **Designed reduced-motion mode.** The rationale is inclusion at launch: reduced motion is "a designed register with its own calm," not a degraded fallback or v1.1 fix.

- **Procedural call captions.** The rationale is accuracy and parity: captions are generated from the actual call parameters and mirror into `aria-describedby`.

- **Full keyboard navigation.** The rationale is that the full product surface must be keyboard reachable, with visual and DOM focus agreeing.

- **WCAG AA on all user copy.** The rationale is that user copy and non-text controls must remain readable across sky palettes, fade states, and motion modes.

- **Initial JS budget.** The rationale is that the 2 MB PRD figure is a ceiling, not a target; treating it as a target would miss time-to-first-bird on mid-tier devices.

- **Time-to-first-bird under 500 ms.** The rationale is the central conceit that the aviary is already in motion, with no load state.

- **60 fps idle.** The rationale is sustained smoothness for the living scene on a 5-year-old mid-range laptop.

- **Zero memory growth.** The rationale is long-running idle reliability; the plan makes it a "real test in CI, not a guideline."

- **Synthetic account UUID everywhere.** The rationale is data-plane separation and privacy: `iam.accounts.id` is the only cross-service identifier and emails are isolated to IAM.

- **Per-bird interaction data never aggregated.** The rationale is to prevent analytics from becoming trait dashboards, retention cohorts, or "the raw material of a streak counter."

- **Hard data-plane separation from analytics.** The rationale is that the telemetry plane must be unable to carry IDs or reach `sim-db`, making the privacy claim architectural.

- **Scope guards.** The rationale is durability: "A non-goal that only lives in a doc gets re-litigated in six months; a non-goal that fails the build does not."

### 2. Architecture

- **Five deployable services plus static/edge tier.** The rationale is "deliberately few": product complexity is in simulation and client, not topology.

- **TypeScript everywhere and Node 22 LTS.** The decisive reason is "one package, three consumers" for the simulation kernel; three implementations would become three drift functions.

- **Pure deterministic `@aviary/sim`.** The rationale is authoritative execution, calibration, and render-only extrapolation sharing one deterministic kernel.

- **Splitting `sim/core` and `sim/drift`.** The rationale is that the client needs mood, motion, and call scheduling for rendering but must be structurally unable to compute or hold trait values.

- **Snapshot boundary.** The rationale is smooth rendering for about 90 seconds while preserving server-side simulation authority.

- **Derived render parameters, not traits.** The rationale is to make "never exposed numerically" true even with DevTools open.

- **Client-side ornament.** The rationale is that leaves and feathers are not part of the aviary's identity, so two devices showing different leaves is "correct and intended."

### 3. Data model

- **Postgres schemas `sim` and `iam`.** NOT RECOVERABLE FROM PLAN.

- **UUIDv7 identifiers.** The rationale given is time-ordered, index-friendly ids.

- **Encrypted email and blind index.** The rationale is sign-in lookup without decrypting the corpus; email exists only in IAM and the pepper lives in KMS.

- **Session device labels without full UA strings.** The plan shows labels like "Safari on iPhone" and says no full UA string retained; the rationale is privacy minimization.

- **Separated `sim.bird_traits`.** The rationale is independent revocation of write privilege so only the tick can mutate personality.

- **Trait monotonicity database trigger.** The rationale is to fail loudly if any code path tries to decrement a trait, not rely on convention.

- **`applied_through_tick`.** The rationale is exactly-once consumption so a tick crash cannot double-apply presence on retry.

- **Mood and render state tables.** The rationale for server-persisted mood is that mood keeps advancing during absence and does not snap to default on tab open.

- **Greeting intents table.** The rationale is single-consumption across devices so a second device sees the aviary already continuing.

- **Presence stored as intervals.** The rationale is to compute union across devices, never sum, preventing double-credit.

- **Notebook entries immutable and read-only.** The rationale is that the field notebook is generated observation; client never authors or edits it, and history remains unbounded.

- **Visit sessions separated from interaction events.** The rationale is that visitor attention is structurally incapable of reaching the drift function.

- **Six species in versioned TypeScript config.** The rationale is the "about six" species pool, with versioned additive config so species changes do not re-identify existing birds.

- **Nightjar-like species.** The rationale is to support a night-active signature required by the late-hours aviary.

- **Additive-only species config.** The rationale is that changing species definition never re-identifies an existing bird.

### 4. API surface

- **REST/JSON over HTTP/2.** NOT RECOVERABLE FROM PLAN.

- **Private no-store authenticated responses and secure session cookie.** The security/privacy rationale is implied by account surfaces but a specific rationale is NOT RECOVERABLE FROM PLAN.

- **Polling, not WebSockets.** The rationale is that a 60-second tick, KB snapshots, and no co-presence make polling cheaper, graceful on flaky networks, and simpler operationally.

- **Magic-link constant-time request behavior.** The rationale is to avoid revealing whether an account exists.

- **Atomic magic-link consumption.** The rationale is replay resistance and matter-of-fact failure.

- **Revocable sessions.** The plan gives behavior but a specific rationale beyond account control is NOT RECOVERABLE FROM PLAN.

- **Snapshot in initial HTML.** The rationale is first render without a round trip.

- **Snapshot polling triggers.** The rationale is to refresh only when visible, after relevant interactions, or after render gaps; hidden tabs do not poll.

- **Events endpoint with idempotency.** The rationale is retries and multi-tab duplicates are free.

- **Server clamping event windows.** The rationale is that a suspended client cannot claim an hour of presence.

- **No trait-value endpoint.** The rationale is that future contributors must argue against a written API-design rule rather than fill an obvious gap.

- **Quantized `plumage_step`.** The rationale is to prevent it from acting as a high-resolution readout of `plumage_saturation`.

- **Notebook read endpoint with no write verbs.** The rationale is read-only notebook by construction.

- **Account export with raw traits.** The rationale is data subject access: the one authorized place raw traits leave the system, not a UI.

- **Soft delete and hard delete.** The plan gives a 30-day horizon and purge behavior, but a specific rationale for the 30 days is NOT RECOVERABLE FROM PLAN.

- **Visitor snapshot stripped of host-only fields.** The rationale is ambient read-only visitation with no greeting, offers, settled state, or account-scoped data.

- **Invite abuse controls.** The rationale is preventing spam and enumeration.

### 5. Simulation engine design

- **60-second logical tick.** The rationale is continuous wall-clock simulation independent of clients.

- **Fast-forward equivalence.** The rationale is sane cost without violating "runs whether or not a client is connected."

- **Hot/warm/cold physical tiering.** The rationale is that per-minute writes for every account would be "a large, mostly wasted bill."

- **Leases fenced by `tick_seq`.** The rationale is that two workers cannot tick one aviary.

- **Fixed per-tick pipeline order.** The plan says order matters and is fixed, but a specific rationale for the exact order is NOT RECOVERABLE FROM PLAN.

- **Low-pass signal in drift.** The rationale is slow, sustained attention: no single session produces visible change.

- **Presence credit caps.** The rationale is anti-farming and preventing heavy use from moving birds too quickly.

- **`plumage_saturation` driven by presence alone.** The rationale is that it is the "sustained attention" trait.

- **Saturating drift.** The rationale is that heavy users approach an asymptote instead of pinning.

- **Calibration targets.** The rationale is to make drift measurable at about one week and visible at about three weeks.

- **Rendered perceptual thresholds.** The rationale is that rendered consequence is what a user actually perceives.

- **Mood softmax and guards.** The rationale is smooth mood changes without flicker, damping, or invalid night states.

- **No mood names in UI.** The rationale is that mood is read from motion; rendering a label would break that contract.

- **Call motif library, identity parameters, and expressive modulation.** The rationale is preserving call identity while allowing mood and drift to change expression.

- **Server-issued call intent.** The rationale is agreement across devices about who calls when and matching captions to audio.

- **Density limiter and anti-unison.** The rationale is to keep chorus musical rather than muddy and avoid a simultaneous chorus "on cue."

- **Call-and-response.** The rationale is audible sociality: a small social system rather than independent NPCs.

- **Return-greeting absence buckets.** The rationale is that short absence is not arrival, while longer absence can be noticed in graduated, quiet forms.

- **One greeter, not all birds.** The rationale is that an all-bird cue would announce arrival in the wrong register.

- **Greeting variation test.** The rationale is that real variation should defeat pre-recorded or rotational variants.

- **Field notebook sparsity gate.** The rationale is one entry every 2-4 days and not a feed, especially for active users.

- **Notebook forbidden lexicon.** The rationale is that the notebook observes the aviary, never the user's behavior.

### 6. Sync model

- **Almost nothing to sync.** The rationale is that server canonical state and snapshot rendering make sync a property rather than a feature.

- **No CRDT, merge, vector clock, or LWW.** The rationale is that write shapes that would cause stale overwrite failures do not exist.

- **Ordered exact-once consumption.** The rationale is no lost drift, deterministic retry, and replay safety.

- **Presence union.** The rationale is to avoid double-crediting multi-device users.

- **Monotonic `tick_seq`.** The rationale is that out-of-order responses cannot make the aviary go backwards.

- **Client-side two-entry snapshot buffer and springs.** The rationale is that correction reads as the bird changing its mind, not a network artifact.

- **Large corrections rendered as flight.** The rationale is avoiding teleports after hidden-tab changes.

- **One-shot greeting, offer reaction, and settle acknowledgments.** The rationale is preventing duplicated greetings or reactions across devices and preserving the fiction of one place.

- **Matter-of-fact failure surfaces.** The rationale is product voice and keeping errors in chrome, not inside the aviary scene.

- **No transient error surfaces.** The rationale is that "a place does not display an error when a packet drops."

### 7. Frontend rendering pipeline

- **Canvas2D single canvas.** The rationale is enough budget for seven birds and ornament without WebGL bundle, complexity, or driver bugs for effects the plan does not want.

- **No scene framework.** The rationale is zero overhead on the critical path because the scene is not a component tree.

- **Preact for chrome/settings.** The rationale is that top bar, notebook, settings, account, and invites are ordinary UI and not on the first-bird path.

- **Vector path data to runtime-rasterized pose atlases.** The rationale is smaller assets and runtime plumage modulation.

- **Parallax layers.** The plan defines layers and small offsets, but a specific product rationale beyond scene composition is NOT RECOVERABLE FROM PLAN.

- **Procedural bird rig and feather overlay.** The rationale is mood, identity, and plumage expression without sprite sheets.

- **Synthesized idle motion.** The rationale is no loops, no synchronized birds, and mood legible through motion.

- **No timeline-based scene animation primitive.** The rationale is to prevent repeated cycles and keep motion synthesized.

- **No spinner, no fade-from-static, no entry animation.** The rationale is the "already in motion" conceit and the refusal of machine-like loading indicators.

- **Warm seed from IndexedDB.** The rationale is to hit first-frame speed on repeat visits while remaining render-only and superseded by the server.

- **Quiet field fallback.** The rationale is a slow-network fallback that is not a spinner, progress bar, or percentage.

- **Day/night without geolocation.** The rationale is local-time ambience without asking for location for a bird app.

- **Weather as rain or wind only.** The rationale is ambience that devices agree on and the user need not notice.

- **Fixed ornament pool.** The rationale is no per-frame allocation.

- **Four-icon top bar.** The plan specifies account/settings, accessibility settings, notebook, and offer only; a specific rationale for exactly these four icons is NOT RECOVERABLE FROM PLAN.

- **Top-bar fade suppression states.** The rationale is keyboard and reduced-motion accessibility.

- **Reduced-motion second sampler.** The rationale is identical scene state with only temporal sampling changed, preserving calls, drift, mood, notebook, and narration.

- **Hidden-tab lifecycle.** The rationale is no stale rendering, no wasted polling/audio, and resuming at current state with no entry animation.

### 8. Audio pipeline

- **Native WebAudio nodes, no AudioWorklet.** The rationale is avoiding cross-browser lifecycle bugs while node counts are within budget.

- **Pre-allocated voice pool.** The rationale is no per-call allocation and no memory growth.

- **Procedural synthesis.** The rationale is reproducible calls from identity, mood, and drift, enabling precise captions and avoiding recorded audio.

- **No recorded audio.** The rationale is unconditional: no audio files or playback fallback in any path.

- **Per-bird panning and reverb.** The rationale is chorus depth without stereo gimmickry.

- **Listen-in mix.** The rationale is distance and focus without muting the rest of the aviary or creating solo tracks.

- **Autoplay unlock by joining mid-phrase.** The rationale is resolving browser policy without an announcement; audio should feel like turning toward a room already making noise.

- **Captions while audio is locked.** The rationale is preserving the audible surface's information until audio can resume.

- **Graceful silence when WebAudio unavailable.** The rationale is accessibility and restraint: captions default on, no recorded fallback, no scene error.

### 9. Accessibility surfaces

- **Single-source accessibility architecture.** The rationale is preventing visual, narration, and caption surfaces from drifting apart over time.

- **Canvas `aria-hidden` plus shadow accessibility tree.** The rationale is real DOM semantics for focus, hit-testing, and assistive technology while canvas remains paint.

- **Naturalist prose narration cadence.** The rationale is product voice: narration is running observation, not a state list.

- **No `aria-live="assertive"`.** The rationale is that assertive interruption is the auditory equivalent of a toast.

- **Priority event observations.** The rationale is still "observations," not event labels.

- **Call captions from synthesis parameters.** The rationale is captions that describe precisely what was played.

- **DOM captions near birds and mirrored to `aria-describedby`.** The rationale is visual readability and inline screen-reader access.

- **Keyboard navigation spec.** The rationale is complete access to scene, listen-in, offer, and settle without a pointer.

- **Dual focus indication.** The rationale is browser/AT focus tracking and visual scene agreement across bright and dark skies.

- **Contrast checks across sky palettes and fade states.** The rationale is that fading and time-of-day ambience must not break readability.

### 10. Performance budgets and observability

- **Aggressive code-splitting.** The rationale is keeping non-critical UI off the first-bird path.

- **Edge-computed document and inline bootstrap.** The rationale is first bird within 500 ms despite per-account HTML.

- **Deferred atlas/audio/chrome work.** The rationale is first bird before non-critical work.

- **Frame-budget choices.** The rationale is keeping motion sampling O(birds), avoiding layout reads, and staying inside 60 fps.

- **Disposable scene lifecycle resources.** The rationale is preventing memory growth over long sessions.

- **Synthetic monitoring.** The rationale is measuring first-bird timing, frame-time, audio-context success, sign-in, and snapshot latency before users report regressions.

- **Aggregate-only RUM.** The rationale is observability without account or bird dimensions.

- **Tick lag metric.** The rationale is detecting "the aviary stopped running," which tick latency alone would miss.

- **No per-account, per-bird, retention, visit-frequency, or leaderboard-ready metrics.** The rationale is that not computing them prevents forbidden features from being "just exposed" later.

- **Shadow accounts.** The rationale is validating drift health without inspecting real users' birds.

### 11. Rollout

- **Parallel workstreams.** The plan says engine/backend, client/scene, audio, accessibility, and infra/edge run in parallel; a specific rationale beyond execution sequencing is NOT RECOVERABLE FROM PLAN.

- **Accessibility workstream from M0.** The rationale is that reduced motion as a v1.1 fix would tell those users "the product wasn't for them."

- **Four-week beta minimum.** The rationale is that a two-week beta cannot validate a three-week calibration target.

- **Qualitative beta question.** The rationale is to measure perceived drift without naming personality, drift, or traits.

- **Bird ramp delayed by aviary age.** The rationale is slack for recognizability studies before real users hear larger choruses.

- **Config flag for new-bird offer.** The rationale is to keep the offer off until recognizability passes.

- **Instrumentation from day one.** The rationale is that silent failures like tick lag, presence-union regressions, trait-write provenance, and monotonicity violations must be visible immediately.

- **Launch gate.** The rationale is to require budget, accessibility, calibration, privacy, security, visitor isolation, no-recorded-audio, and invariants to be green before public v1.

### 12. Testing strategy

- **Kernel property tests.** The rationale is to make determinism, fast-forward equivalence, monotonicity, presence union, idempotency, and bounds merge-blocking.

- **Sync integration tests.** The rationale is to catch no double-credit, no lost drift, no duplicate greeting, and monotone `tick_seq` across two clients.

- **Calibration lab.** The rationale is to collapse a three-week feedback loop to seconds and give drift tuning a purpose-built instrument.

- **Automated and human recognizability tests.** The rationale is protecting "you know Pip by ear" and gating the bird-count ramp.

- **Audio variation tests.** The rationale is rejecting rotation-of-N variants.

- **No recorded audio scans.** The rationale is enforcing the no-recorded-audio product rule.

- **Listen-in mix tests.** The rationale is verifying ramp shape and unfocused buses never reaching silence.

- **Accessibility tests.** The rationale is making narration, captions, keyboard paths, reduced motion, and contrast release-quality and merge-blocking.

- **Product-invariants suite.** The rationale is that the temptation to add "just one" forbidden surface is strong; build failures are more durable than documents.

- **Performance tests.** The rationale is enforcing bundle size, first-bird timing, frame time, memory slope, snapshot payload, and tick tiering before launch.

### 13. Risks

- **Drift calibration risk.** The rationale for mitigation is that too fast becomes Tamagotchi and too slow becomes screensaver; correction must preserve monotonicity.

- **Silent sync corruption risk.** The rationale is that drift errors are invisible to users and logs unless specifically detected.

- **Audio uncanniness risk.** The rationale is that procedural calls and chorus quality are qualitative and need sound design plus listening gates.

- **Accessibility regression risk.** The rationale is that narration and captions are easy to let rot because many contributors do not experience them.

- **Autoplay policy risk.** The rationale is that missing audio on first session weakens the affective spine, so the unlock path must avoid banners.

- **500 ms versus per-account HTML risk.** The rationale is that per-account documents cannot be CDN-cached, so edge compute and warm seed must carry first paint.

- **Tick cost risk.** The rationale is that naive per-minute ticking for every account is too expensive, so fast-forward tiering must hold.

- **Notebook prose risk.** The rationale is that event-log-like prose would make the product voice feel like performance.

- **Gamification creep risk.** The rationale is that it will arrive as reasonable-sounding pitches, so invariants and PR questions defend the line.

- **Privacy-boundary erosion risk.** The rationale is that future dashboard requests need a compliant answer already available through shadow accounts.

- **Visit-flow abuse risk.** The rationale is preventing spam and enumeration with rate limits, token design, expiry, and non-disclosure.

### 14. Decisions and assumptions

- **Presence activity window: 4 minutes.** The rationale is that watching birds without moving is the actual product, so "a few minutes" leans longer.

- **Tick cadence and tiering.** The rationale is continuous running at sane cost.

- **Mood set includes roosting.** The rationale is that the plan needs a distinct full-night state.

- **Trait range and seeds.** The rationale is headroom for weeks of monotonic drift without pinning.

- **Drift constants.** The rationale is the 1-week and 3-week targets, owned by the calibration lab.

- **Day/night source.** The rationale is local time without the poor trade of asking location for a bird app.

- **Offer cooldown.** The rationale is keeping all three offers reachable in a session while respecting "a few minutes."

- **Notebook sparsity.** The rationale is one entry every few days and protection of sparsity for active users.

- **Bird-count ramp.** The rationale is "a few months" for the third bird and protecting recognizability.

- **Species pool.** The rationale is "about six" and one night-active signature.

- **Transport.** The rationale is 60-second tick, KB payloads, no co-presence, and cheaper operation.

- **Renderer.** The rationale is small object count and not a parallax-heavy showpiece.

- **Server language.** The rationale is one shared simulation kernel.

- **Audio unlock.** The rationale is consistency with browser policy and "notice, never announce."

- **Visit duration granularity.** The rationale is that approximate buckets resist inference about visitor habits.

- **Export delivery.** The rationale is keeping trait values off UI surfaces while delivering to the verified address.

- **Event-log retention.** The rationale is enough for calibration audit while traits remain canonical.

- **Top-bar fade floor.** The rationale is honoring near-transparency without failing contrast.

- **Snapshot content.** The rationale is making "never exposed numerically" true against DevTools.

- **Warm-seed rendering.** The rationale is first-frame speed on repeat visits without violating server authority.

### 15. Open questions for the product owner

- **Third-bird offer presentation.** The rationale for the default is quiet in-aviary naturalist voice; the plan notes an even quieter arrival-with-notebook approach may better fit "notice, never announce."

- **Settle persistence across devices.** The rationale for canonical settle is that per-device settle would make the aviary "two aviaries."

- **Bird naming at adoption.** The plan names default suggestions and inline renaming, but a specific rationale for that default is NOT RECOVERABLE FROM PLAN.

- **Beta length.** The rationale is that four weeks is the floor for observing three-week drift; if compressed, the plan would plainly say validation came from simulation rather than the wild.
