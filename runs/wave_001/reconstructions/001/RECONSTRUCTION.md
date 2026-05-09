## System-level intent

- **Executable specificity over restatement.** The plan says it "does not restate the PRD" but "commits to specific architectural shapes, schemas, protocols, calibrations, file layouts, error envelopes, and rollout steps." This shows up in the calibration table, concrete SQL schemas, API envelopes, tick pseudocode, budgets, and ship/no-ship criteria.

- **The server is the only writer of canonical state; the client is a renderer.** This is named directly in the client/server split, the sync model's one-sentence summary, the no-client-personality-write barrier, and the v1 no-ship rule that "the simulation runs anywhere on the client" must not happen.

- **Slow, sparse, non-twitchy aliveness.** The plan repeatedly favors slow cadence and felt continuity: 60s ticks are "slow enough for 'feels alive without being twitchy,'" drift is visible over weeks, notebook entries preserve "sparsity," the top bar fades, offers are gestures, and the first frame should feel like an "already-running aviary."

- **Presence can make birds more expressive, but absence must not punish.** The "monotonic-toward-expressive rule" is called the "load-bearing engine asymmetry": traits "never drift down on absence," so an ignored bird becomes ambient by "not gaining" rather than "learning to mistrust."

- **Anti-gamification and restraint are structural, not decorative.** The out-of-scope list excludes "achievements, streaks, levels, badges, XP" and similar mechanics. Later sections refuse engagement infrastructure: no per-account visit history table, no daily aggregation job, no "view per-user activity" admin tool, no user-visible streak or counter.

- **Privacy boundaries must be real, not policy.** The plan says this explicitly about telemetry. It appears in separate DBs, account UUID partitioning, email encryption, IAM separation, schema linters, aggregate-only telemetry, no per-account metrics, and audit checklists.

- **Naturalist voice belongs to the aviary; system/account surfaces stay matter-of-fact.** The plan reserves naturalist voice for "the aviary, the notebook, and the narration," while sign-in, sync, account, settings, and errors use a "matter-of-fact tone." The weekly voice review says "the voice is the product."

- **Procedural, deterministic, writer-owned generation is preferred over runtime invention.** Calls are procedural with seeded variation; there is "no recorded audio anywhere." Notebook, captions, and narration use checked-in templates/YAML, and the plan says no free-form runtime LLM because the voice is "too specific and load-bearing."

- **Accessibility is a designed surface, not a fallback.** Reduced motion is "not implemented as 'skip the rAF callbacks'" but has its own cross-fade aesthetic. Captions, narration, keyboard navigation, focus indicators, audio-off behavior, axe-core, and manual screen-reader tests are all part of definition-of-done.

- **Continuity and identity matter more than visible mechanics.** Stable `bird_id`, preserved `mood_started_at`, cross-device same-state sync, no missed-tick replay, 800ms cross-fades, and "bird identity slip" risk handling all protect the feeling that the aviary continues without snapping or resetting.

## Per-feature whys

### 1. Scope, service shape, and data model

- **Single-user accounts, magic-link auth, and per-device sessions.** The plan's rationale is separation: Auth Service "owns email + magic links + session tokens" and "produces no per-bird events." Account identity is carried by `account_uuid`; email is never a foreign key, logging dimension, or analytics dimension. Session tokens are 30-day sliding because that is "long-lived enough to match the rhythm of a relationship" while remaining revocable per-device.

- **One canonical aviary per account.** The plan ties this to canonical ownership: the Simulation Service is the "only writer of canonical aviary state," and the Snapshot Service is the only client read path.

- **Two starter birds at adoption.** The plan says the two birds are drawn with no user choice and biased toward "complementary call signatures," so the starting pair contrasts in sound. The first and second fly-ins are also "the only time the user sees a 'from-empty' transition."

- **Seven-bird cap.** NOT RECOVERABLE FROM PLAN

- **Six-species bird pool.** NOT RECOVERABLE FROM PLAN

- **User-assigned, renameable bird names.** NOT RECOVERABLE FROM PLAN

- **Stable bird identifier independent of name/species.** The plan makes this an identity-continuity risk: if `bird_id` is reassigned, the "user feels something is wrong without naming it." `bird_id` is generated once, never recomputed, and checked after migrations.

- **Server-side simulation tick owning personality, mood, perch, and call timing.** The plan calls the simulation engine "the spine of the product." The reason is canonical integrity: the server decides all future-affecting state so client rendering or multi-device use cannot corrupt drift, mood, or calls.

- **Calibration ranges committed up front.** The plan says pinning values "prevents drift across surfaces" and gives the test harness "fixed targets." Recalibration is tied to named metrics or qualitative review rather than random engineering choices.

- **Six independently deployable services plus CDN edge.** The service split preserves ownership boundaries: Auth owns email, Aviary API validates and writes events, Simulation writes canonical state, Snapshot is read-only, Notebook owns entries, and Telemetry is separate.

- **CDN edge with first-snapshot blob.** The plan uses this so the aviary can render "with no round trip" and uphold the first-frame promise.

- **Aggregate telemetry pipeline separated from per-account event log.** The plan says the privacy boundary is "real, not policy"; the telemetry role cannot read Simulation DB or Notebook DB, and schemas reject `account_uuid`, `bird_id`, and denylisted fields.

- **Separate service-domain databases.** The rationale is domain ownership and privacy separation: Account DB, Simulation DB, Notebook DB, and Visit DB carry different responsibilities and permissions.

- **Splitting `aviaries` from `aviary_state`.** The plan says this keeps tick hot writes from touching cold rows that get backed up in account export.

- **Splitting `state_blob` from queryable bird rows.** The blob is the client snapshot and exists so Snapshot Service can serve "a read with no joins"; the `birds` table remains the canonical queryable record.

- **Five personality traits as explicit columns.** The plan gives two reasons: "cheaper indexed update by the tick" and "explicit visibility to a code reviewer" that no other table or service writes the traits.

- **Account export.** Personality numbers appear here because the user "explicitly asked for the data." The product UI never shows them; export is the only user-facing place for the numbers.

- **Browser support for latest two majors of Chrome, Safari, Firefox, and Edge.** NOT RECOVERABLE FROM PLAN

### 4. API surface

- **REST plus WebSocket snapshot channel.** REST is used for API surfaces, while WebSocket is only a best-effort "state changed" notification; the client still pulls the next snapshot so canonical read behavior stays centralized.

- **Auth rate limits.** NOT RECOVERABLE FROM PLAN

- **Snapshot read endpoint.** The endpoint returns server-composed canonical state so the client does not decide greetings, call timing, mood, or future state. `since`/304 avoids re-pulling identical state.

- **Greeting field only on the first snapshot of a session.** The plan says this avoids "the client deciding the greeting" and prevents repeated greetings until the next return.

- **Interaction endpoints return acknowledgments, not state deltas.** The rationale is that "the next snapshot pull surfaces the consequences"; event writes do not let the client own state.

- **Presence ping with visibility, focus, and last-input signals separated.** The server validates the conjunction itself "rather than trusting the client to compute presence," keeping "the drift-corruption prevention story honest."

- **Client event IDs and idempotency.** The plan says duplicates from retrying after a network blip are ignored, so "wrote it twice because the network blipped" is not a problem.

- **Offer cooldown.** The cooldown is "long enough that an offer is a gesture" and short enough that a 10-minute session can include more than one.

- **Three offer types: seed, song fragment, still pool.** NOT RECOVERABLE FROM PLAN

- **Settle gesture with 5s undo and tab-close equivalence.** NOT RECOVERABLE FROM PLAN

- **Notebook endpoint read-only.** The plan explicitly has no PUT/POST/DELETE because the notebook is a generated, read-only field record rather than a user-authored surface.

- **Visit host and visitor endpoints.** The plan keeps visits "per-invite opt-in" and read-only. Visitor snapshots omit interactive fields, visitors cannot send events, and visit pulls never write to the host event log.

- **Visit snapshot precision reduction.** The visitor sees the aviary "as-is," but `next_motion.started_at` precision is reduced so visitors do not get sub-tick precision that would reveal how the simulation works.

- **Consistent error envelope.** Codes are stable, and the client "never invents its own" message. Error surfaces use matter-of-fact tone.

### 5. Simulation engine design

- **Per-account 60s tick driver.** The plan says 60s is slow enough to avoid twitchiness and fast enough to land mood transitions before a snap. Jitter spreads load, and sleeping accounts reduce cadence to save CPU.

- **Single DB transaction per tick.** The reason is ordered retry safety: if a tick aborts, events remain unconsumed and the next tick retries with ordering preserved.

- **Drift low-pass function and 7-day time constant.** The plan ties this to "measurable drift in instruments at ~1 week" and visible change at about three weeks.

- **Presence-derived drift inputs.** Presence is computed from conjunction-validated pings, not just received events, so attention affects drift without letting noisy event counts become personality.

- **Heavy-use saturation with `tanh` and 12-hour clamp.** The plan says heavy use should saturate rather than overshoot `[0, 1]`.

- **Monotonic-toward-expressive personality drift.** The rationale is explicitly affective: birds become "quieter than they were, not learning to mistrust."

- **Mood as fast-timescale state.** Mood may swing both directions because it is "not personality"; this lets alarms, weather, and time of day affect current behavior without violating the no-negative-drift rule.

- **Stochastic mood transitions.** Sampling rather than thresholding creates variation so "two days with the same nominal inputs do not produce identical mood paths."

- **Preserved `mood_started_at`.** The client renders the mood the bird has been in, not a fresh "just woke up" state; "mood snapping on tab open is forbidden."

- **Procedural call grammar.** Calls are generated from motif descriptors and WebAudio so there is no recorded audio, while per-call seeds keep repeated motifs from sounding identical.

- **Server-scheduled call timing.** The client never decides when a call happens; missed calls are rolled forward by the next tick so the client does not "make up" missed calls.

- **Bird-to-bird alarm spread.** The bounded one-alarm-per-tick rule prevents oscillation, and boldness dampens whether another bird absorbs the alarm.

- **Chorus emergence.** The tick aligns high-vocal-frequency calls to create an "audible chorus for the user" that is "procedurally re-emerging — not scripted."

- **Notebook noteworthy-moment detection.** The predicates pick moments such as greeter change, long quiet, visible mood shift, chorus, weather, plumage threshold, and bold step so entries come from observable, stateful changes.

- **Notebook sparsity gate.** The plan caps entries to preserve "sparsity even on heavy-usage days."

- **Notebook templated prose and no runtime LLM.** The plan says the PRD's voice is "too specific and load-bearing" for a drifting model; templates create stable prose and the writer owns them.

- **Return-greeting computation.** The greeting uses absence length, current personality, mood, and bird choice weights so the return is stateful but still server-owned.

- **Staggered multiple-bird greetings.** When a long-call response happens, calls are "staggered, never simultaneous," preserving a natural sequence rather than a pile-up.

- **Adoption with no species choice.** The rationale given is complementary call signatures, not user selection: the plan avoids two birds whose call grammars do not contrast.

- **Adding birds by aviary age.** The pacing is "age-only — never engagement-driven," which protects the anti-gamification rule.

- **Declining a bird offer and reappearance after 30 days.** NOT RECOVERABLE FROM PLAN

- **Seventh-bird cap reminder in quiet copy.** The plan's rationale is voice restraint: the reminder is "quiet copy in naturalist voice."

### 6. Sync model

- **Snapshot-pull pattern.** The plan repeats that clients receive state via snapshot pull only, reinforcing that the server is the canonical writer and the client does not replay missed ticks.

- **No client-side personality writes.** The plan says last-write-wins on personality would "silently delete drift," so the API lacks any personality write path and DB roles enforce single-writer access.

- **Append-only event log.** Events are consumed in server-side order, so submissions from two devices can interleave without corrupting drift.

- **Multi-device experience.** The user should feel, "I set down my laptop and picked up my phone; the aviary looks the same." Conflict resolution is a non-feature because only events are submitted.

- **Suspended-laptop detection and resync.** A long frame gap triggers a fresh snapshot; visible divergence cross-fades over 800ms so the client does not snap or replay missed ticks.

- **Email change with old sessions remaining valid.** The plan calls email change a "profile change, not a session-revoke event"; in assumptions it says the chosen path is friendlier for a low-stakes product.

### 7. Frontend rendering pipeline

- **React, Zustand, Vite, WebGL2 stack.** React is chosen because "team velocity > tech preference"; WebGL2 supports plumage saturation modulation, while the architectural shape does not depend on the framework.

- **Canvas/WebGL scene instead of React hot path.** The rendering hot path avoids React; React is only a thin mounting wrapper, supporting the 60fps budget.

- **Three-perch single-screen scene.** NOT RECOVERABLE FROM PLAN

- **Client-only ambient ornaments.** Leaves and feathers are "not part of simulation," so they can enrich the scene without affecting future canonical state.

- **Layered sprites with shader-modulated plumage.** This produces visible plumage drift over weeks "without re-shipping art."

- **Idle micro-motion.** Body bob, head tilt, pose cross-fades, and mood-shaped noise make birds move continuously while visible without creating new canonical state.

- **First-frame inline rendering.** The plan says the user "always sees birds first, never a spinner." The quiet-field fallback is acceptable; a spinner is forbidden elsewhere.

- **Snapshot transitions and cross-fades.** Mood and state changes cross-fade so visible changes do not snap; large divergence also cross-fades rather than animating every missed transition.

- **Reduced-motion mode.** The plan says it is "its own designed surface" with cross-fades, not a broken renderer that merely skips rAF callbacks.

- **Top-bar fade.** NOT RECOVERABLE FROM PLAN

- **Empty-aviary quiet field.** The quiet field avoids a spinner or text and is used only during loading or the brief first adoption transition.

- **Responsive layout.** The plan gives practical rationale in the constraints: on phones "nothing crops"; all viewports preserve bird aspect ratio with "no stretching."

- **60fps frame budget.** The budget assigns milliseconds to state, bird animation, draw, and audio scheduling so idle motion can stay below p99 frame-time targets.

- **No memory growth over 30 minutes.** The plan guards against "un-released allocations" with bounded pools, dropped off-screen entries, bounded workers, and fixed-size sprite cache.

### 8. Audio pipeline

- **Client-side WebAudio synthesis.** Calls are synthesized in the browser from motif descriptors, supporting the "no recorded audio anywhere" boundary while keeping the server out of audio rendering.

- **Motif library and grammar in YAML.** The checked-in library gives writer/audio-team-owned motif parameters while weighted seeded selection creates variation.

- **Listen-in mix ramps.** The 1.5s smoothed exponential ramp avoids the "switching" feel; other birds are lowered but "never 0," preserving the "never silent" rule.

- **Chorus mixing.** A small master gain rise gives the chorus "presence" without creating a separate "chorus mode."

- **Seeded per-call variation.** Pitch, envelope, vibrato, and loudness jitter make calls "never identical," while seeding means replaying a snapshot sounds the same.

- **Captions.** Captions are deterministic mappings from descriptor and mood; templates are the "authoritative writer-owned voice," with no runtime generation.

- **Caption placement and contrast.** Captions avoid overlapping the bird sprite and choose foreground color by measured background luminance to pass AA.

- **WebAudio fallback to silence plus captions.** The plan says there is no recorded-audio fallback. Captions turn on so the product remains complete, and the one toast is matter-of-fact because the user is dealing with the browser as a system.

- **Audio context lifecycle.** The context is created lazily because of autoplay policy, suspended on tab hide, resumed on show, and closed on sign-out.

- **Volume control in accessibility settings, with no top-bar mute.** The plan says muting should be a deliberate choice, "not a thoughtless app-level reflex."

### 9. Accessibility surfaces

- **Screen-reader narration.** The narrator uses the same canonical state and naturalist voice as the visual scene, with full-sentence updates so screen readers do not read partial state.

- **Keyboard navigation and shortcuts.** The plan makes top bar, birds, listen-in, escape, and shortcuts keyboard reachable so the aviary scene is navigable without pointer input.

- **Focus indicators.** The renderer chooses high-contrast color based on local background luminance so the outline remains visible over the scene.

- **WCAG AA contrast checks.** Chrome text, captions, and displayed narration must pass AA; `axe-core` verifies rendered chrome in CI.

- **Audio-off captions.** When audio is off, captions auto-enable so "the product remains complete."

- **Settings surfaces in matter-of-fact voice.** Settings, account, and sign-in avoid naturalist prose because naturalist voice is reserved for the aviary, notebook, and narration.

### 10. Performance budgets and observability

- **Bundle budgets and code-splitting.** Settings, account, visit-host UI, and export are lazy-loaded so the initial bundle stays within the first-bird performance budget.

- **Time-to-first-bird measurement.** The p95 ≤ 500ms target is tied to the affective threshold and is measured from the inline render moment with aggregate-only RUM.

- **Idle 60fps and memory tests.** These keep the ambient surface from becoming visibly janky or leaking over long sessions.

- **Simulation tick latency.** p99 < 5s protects event-to-state freshness and triggers alarms if the simulation falls behind.

- **Aggregate-only metrics.** The plan measures request counts, latency, errors, tick latency, sessions, page load, frame-time, audio errors, magic-link counts, and visit pulls without per-account dimensions.

- **Deliberately unmeasured per-account and per-bird state.** The plan refuses "average drift," "average mood," and per-account visit histograms because these would convert interaction into a "data product."

- **Structured logs and traces.** Logs use `account_uuid` only and never email; traces are sampled and never include event payloads, preserving privacy boundaries.

- **Synthetic probes.** Headless browsers verify sign-in, snapshot, first-bird render, offer acknowledgment, tick landing, and sign-out so failures page on-call.

### 11. Rollout

- **Internal alpha.** The plan uses ~30 internal users for daily review of errors, tick latency, narrations, and notebook entries to catch voice drift early.

- **Closed beta.** The plan uses ~500 invited users to monitor drift calibration, audio uncanniness, and accessibility regressions before public open.

- **Public open ramp.** The sign-up cap ramps aviary count over four weeks, controlling operational exposure.

- **Server-only feature flags.** Flags are operational and not user-visible; they tune bird cap, tick cadence, drift, notebook rate, and WebAudio fallback behavior without exposing "experimental" toggles.

- **Fresh DB and migration discipline.** V1 has no migration, but future personality-vector schema changes require review for preserving "drift identity."

- **Day-one instrumentation.** Synthetic probes, aggregate RUM, counters, tick dashboards, bundle gates, memory tests, and axe-core start immediately so the product is observable without per-user activity tools.

- **No per-user activity admin tool.** The rationale is anti-engagement and privacy restraint: internal debugging requires privileged justification and audited reads.

- **Privacy launch checklist.** The checklist verifies email join paths, telemetry fields, log scrubbing, legal review, and KMS audit before public open.

- **Weekly voice-drift review.** The product writer reviews notebook entries and narrations because "the voice is the product."

- **No A/B testing on affective core calibration.** The plan forbids A/B testing here because it would "shift the relationship the product is selling."

### 12. Risks

- **Drift too fast.** The risk is that birds changing session-to-session makes Pocket Aviary a "stat-management exercise without ever showing stats."

- **Drift too slow.** The risk is failing the "feels alive over weeks" promise.

- **Sync corruption.** The plan protects against personality being "lost or doubled" with DB-role enforcement, tick uniqueness, and replay property tests.

- **Audio uncanniness.** The risk is robotic, samey, or ugly chorus artifacts; the plan treats naturalist-feeling calls as central enough that failure may require re-evaluating procedural synthesis.

- **Engagement-pressure leak.** The plan says the first "harmless" engagement feature cracks the rule; mitigation is cultural and structural, including refusing infrastructure that could feed streaks.

- **Privacy regression.** Separate IAM, separate DBs, schema linting, KMS review, and audits make accidental leakage hard.

- **Accessibility regression.** Accessibility is in definition-of-done, with PR checks for narration, reduced motion, and keyboard navigation.

- **First-frame failure.** A spinner is forbidden; the quiet field is the only acceptable degraded state.

- **Bird identity slip.** Stable `bird_id` protects an unnamed but felt continuity; migrations must preserve it.

- **Visit feature corrupting host drift.** The visit endpoint lacks `INSERT` on `event_log`, so visitor attention cannot change host simulation.

- **Notebook prose drifting off-voice.** Writer review and PR review of YAML templates prevent notebook text from becoming "event log" prose.

- **Multi-device magic-link replay.** Single-use magic links reject replay, while legitimate two-device sessions are safe because the event log absorbs interleaving.

- **Email-change race.** A single pending change stored on the account row and atomic verification avoid an indeterminate state.

### 13-15. Work breakdown, assumptions, and ship criteria

- **Sequenced engineering work.** The ordering starts with account UUIDs, encrypted email, DB schemas, and CI scaffolding, then builds simulation, snapshots, calls, rendering, interactions, notebook, accessibility, visits, account management, hardening, and phased release so foundational invariants exist before surfaces depend on them.

- **Stack choices as assumptions.** React/Zustand/Vite are chosen for team familiarity; WebGL2 is chosen for plumage modulation; Postgres UUIDs avoid a separate UUID service at v1 scale.

- **Mute reachability as a flagged design call.** The plan explicitly says this is "against the obvious" and needs design-lead sign-off because it expresses restraint over richness.

- **Bird-to-bird coupling tunables.** The plan admits earshot radius and chorus window are educated guesses subject to phase 2 tuning.

- **English-only notebook templates for v1.** The plan states i18n is a v2 problem; v1 templates are English YAML.

- **Visitor per-token rate limit.** The plan says it is "not a security feature; a politeness one."

- **Definition of done.** Ship requires functional acceptance, first-bird, tick latency, memory, accessibility, bundle, probes, privacy, voice review, and calibration. No-ship criteria encode load-bearing principles: no streak/counter, no return-greeting toast, no state-list narration, no recorded fallback, no client simulation, and no per-account/per-bird warehouse metrics.
