## System-level intent

- The plan treats product meaning as enforceable engineering, not interpretation. This shows up in the opening claim that "every load-bearing rule is restated here as an engineering decision, an invariant, or a test," and in Product invariants, Testing, Definition of done, and the "summary fence for contributors."

- The product is intentionally made by refusal. Scope, non-goals, and "What this plan deliberately refuses to build" all frame the aviary through "what must never be built": no gamification, no Tamagotchi mechanics, no social-network surfaces, no recorded audio, no spinner, no numeric bird stats. The closing line says, "The product is what's left after these subtractions."

- The aviary should be quiet, observational, and non-announcing. This appears as "The aviary never announces," "No welcome toasts," "The bird greeting is the entire welcome," "matter-of-fact voice" for system surfaces, and notebook/narration rules banning announcement framing, exclamation, "you," streaks, and visit-count phrasing.

- The first visual impression must be continuous life, not startup ceremony. The plan repeats "The first frame is mid-action," "first paint is the aviary, already moving," "There is no 'ready' moment," and "A spinner is a launch-blocking defect." The quiet-field fallback is permitted because it is a field, not a loading indicator.

- The user should experience birds through behavior, not through exposed stats. Product invariants say "The user never sees a number about a bird." Snapshot design sends "derived rendering parameters" and quantized values rather than traits. The calibration harness measures "behavioral visibility, not raw numbers, because raw numbers are never shown."

- The core architecture is "clients render snapshots; the server simulates." This phrase is named as "the PRD's central rule." It drives the client/server split, the single-writer model, multi-device sync, fast ticks, snapshot reconciliation, and the rule that clients submit events but never write personality state.

- Care should accumulate without punishment. This appears in "Drift is monotonic toward expressive," "absence moves nothing down," "Neglect yields ambient quietness, never punishment," and the two-layer model in which traits stay intact while expressiveness can decay so ignored birds become quieter without becoming worse.

- Multi-device sync is meant to be the absence of divergence, not a merge feature. Sync Model says "Multi-device sync is not a feature; it is the absence of divergence," and ties that to one canonical record, one writer, state_version snapshots, idempotent event ingestion, and API schemas that accept observations but not trait values.

- Social access is ambient, read-only, revocable, and explicitly not a network. This shows up in visit invitations being "read-only ambient views," no profiles/follows/feeds/discovery/comments/co-presence, visitor tokens that cannot post events, visit revocation checked on every pull, and visitor presence not existing for the simulation.

- Privacy is structural rather than only policy. Email appears in "exactly one table"; telemetry has "no read credentials on the simulation database"; DB roles separate API writes from simulation writes; per-bird interaction data "never leaves the account context"; metric definitions are reviewed against banned fields.

- Liveliness should be procedural and unrepeatable. This appears in "Calls are procedural, always," "Motion is code, not footage," "The client renders it procedurally from the rig - never a canned animation," and audio tests requiring two renderings of the "same" call to differ while remaining the same motif.

- Accessibility is part of the product, not a remedial layer. Accessibility Surfaces says accessible surfaces deliver the "actual product," "designed for charm," "shipped at launch," and "not after." Reduced motion is "a designed surface, not a fallback"; release gates make "Accessibility ships with the feature."

- Engineering choices are deliberately boring and budget-driven. Architecture is "deliberately boring"; technology selections are "defensible defaults"; substitutions must re-prove the "2MB/500ms budgets and team scale"; CI budgets are "gates, not aspirations."

- The simulation must be deterministic, replayable, and calibratable. This shows up in fixed event order, exactly-once event claiming, ticks as "the observability spine," reconstructibility from initial state plus event log plus sim version, and the calibration harness running real tick code on synthetic personas.

- Growth and engagement mechanics must avoid metrics-chasing. Bird offers are "age-based, never engagement-based"; final pacing is tuned against "retention-safe judgment, not metrics-chasing"; the notebook governor is "explicitly anti-feed"; observability allows aggregate histograms but not per-user relationship reconstruction.

## Per-feature whys

### Scope

- Single-user accounts with email magic-link sign-in, with no passwords and no SSO: NOT RECOVERABLE FROM PLAN.

- One canonical aviary per account: the plan's rationale is that "one canonical record" prevents divergence. Sync is "read consistency, not merge," and laptop and phone pull the same state_version.

- Two starter birds at adoption: NOT RECOVERABLE FROM PLAN.

- Hard cap of seven birds: the plan ties the cap to engine and renderer boundaries. At "<= 7 birds plus ornaments," Canvas 2D can meet the 60fps/5-year-old-laptop budget, and the cap is enforced in the sim so "an 8th cannot be offered."

- Server-side simulation tick: the rationale is that "The tick is the product." The tick advances personality drift, mood, weather, notebook generation, and adoption offers "whether or not any client is connected," so the aviary has been living while the client was away.

- Multi-device sync: the why is "the absence of divergence." The server is "sole writer of canonical state"; clients render snapshots, so two devices never merge competing truths.

- Return-greeting: the plan makes the greeting the whole welcome surface. "The bird greeting is the entire welcome"; there are no toasts, banners, modals, or gone-X-days surfaces.

- Presence accounting with visible plus focused plus recent pointer/key activity: the rationale is honesty. The plan calls these "honesty guards," bounds stuck-tab and abuse cases, and treats pressure to count "tab open" as "Presence honesty erosion."

- Listen-in: the plan grounds it in focused attention without cutting off the aviary. The mixer ramps the focused bird to full and others to about 0.35, "never zero," with hard cuts forbidden; keyboard listen-in produces the identical mix ramp as pointer.

- Offer surface for seed, song fragment, and still pool: the rationale is naturalistic reaction without client prediction. The sim decides who reacts based on proximity, mood, and curiosity, and offer cooldowns guard against curiosity saturating in one session.

- Settle with 5-second undo: the rationale is a quieting affordance without punishment. Settle damps calls and trends birds drowsy; undo is server-validated; tab close without settle has "no penalty, no recovery surface, no notification."

- Field notebook: the rationale is observation, sparsity, and anti-feed behavior. It is "a generator, not a log formatter"; stale observations are "worse than none"; it may observe the aviary but "never observes the user's behavior."

- Visit invitations: the plan's why is ambient sharing without social-network mechanics. Visits are read-only, opt-in, revocable, expire after 30 days, generate only visit_sessions, and "visitor presence does not exist as far as the simulation is concerned."

- Accessibility surfaces at launch: the rationale is that accessible surfaces deliver the "actual product," not a degraded fallback. The plan makes screen-reader narration, reduced motion, captions, keyboard navigation, and contrast part of launch and definition of done.

- Account export: the plan says the user's own data is the one legitimate place where bird numbers leave the system. Personality vectors can be included in the JSON export because "the export goes to the user, not a surface."

- Soft-then-hard account deletion: the plan gives a recovery affordance during soft delete and requires a hard-delete job that removes every row, verified by a deletion audit test.

- Performance budgets: the plan treats them as "CI-enforced gates, not aspirations" because first-bird visibility, idle motion, memory flatness, payload size, and tick latency are product qualities users feel directly.

- No gamification: the rationale is to avoid turning attention into scoring. The plan bans streaks, scores, badges, levels, achievements, counters, green-dot calendars, XP, and tiers; the notebook may observe the aviary, not "you visited every day this week."

- No Tamagotchi mechanics: the rationale is no distress or punishment. Birds do not die, hunger, suffer, or display distress; neglect yields "ambient quietness, never punishment."

- No social-network surfaces: the rationale is that feeds, follows, discovery, comments, co-presence, and leaderboards would make a different product. The plan also refuses to compute the stats that would power leaderboards.

- No notifications or welcome surfaces: the rationale is "The aviary never announces." Transactional auth/export email is infrastructure, not a product notification surface; the greeting bird replaces welcome UI.

- No recorded audio fallback: the rationale is the invariant "Calls are procedural, always." If WebAudio is unavailable, the product falls back to silence plus captions, not recorded audio.

### Architecture and data model

- Three deployable units plus storage: the rationale is a "deliberately boring" architecture that is stateless where possible, horizontally scalable, and non-exotic for v1.

- Stateless API service: the plan's why is that any replica can serve any request and no API process holds simulation state beyond a short fast-tick wait.

- Simulation workers as sole writers: the rationale is to preserve the single-writer invariant. Workers alone have write credentials for birds, aviary state_version, notebook entries, and adoption offers; the split is enforced by database roles, "not by convention."

- Postgres as the main store with optional Redis: the plan says one Postgres cluster is sufficient at v1 scale, while Redis is only convenient; the system must degrade to Postgres-only operation if Redis is absent.

- Client/server split: the why is durability and consistency. "A client that crashes loses nothing"; two devices never diverge because neither owns state; snapshots are authoritative.

- TypeScript monorepo with shared schema, voice, and prose packages: the plan groups this with "defensible defaults" chosen for the 2MB/500ms budgets and team scale.

- REST-ish JSON rather than GraphQL: the plan says the surface is small and cache-friendly, so GraphQL is unnecessary.

- Custom Canvas 2D scene graph rather than a game engine or WebGL: the rationale is bundle and performance discipline. The bundle budget "forbids" a game engine, and Canvas 2D meets the 60fps/old-laptop budget with far less code at this scale.

- Hand-rolled WebAudio graph: the plan groups this with the same budget and team-scale technology selections, and the audio section emphasizes no framework, no allocation churn, and direct resource control.

- Synthetic UUIDs and email in exactly one account table: the rationale is the "most important boring detail." Email is never a log field, shard key, partition key, or telemetry dimension.

- Append-only interaction_events with consumed_tick_id: the rationale is an exactly-once ledger. A tick claims events in the same transaction as the state update so events and state move together.

- presence_daily rollup with caps and dedupe: the rationale is calibrated drift plus honesty. It feeds drift while capping creditable presence, bounding stuck tabs, and making presence per-person rather than per-tab.

- Immutable notebook_entries: NOT RECOVERABLE FROM PLAN.

- Offer cooldowns derived from interaction_events: the rationale is avoiding a separate table "to drift out of sync"; the server remains authoritative.

- visit_sessions separate from interaction_events: the why is that visitors cannot influence the sim. Visit sessions record host-visible logs, while visitors generate no presence, offers, greetings, listen-in, settle, or notebook writes.

- Export storage as encrypted JSON with a signed 7-day link: the plan's explicit rationale is that export is the legitimate place user-owned numbers leave the system; storage and email delivery are the mechanism described for that export.

### API surface

- Snapshot API returning derived rendering parameters instead of personality numbers: the rationale is that the API cannot leak trait values "even to a curious user reading their own traffic."

- Snapshot ETag, since cursor, and 45s polling: the plan says unchanged polls are tiny, ticks are usually no-ops, and visibility/gap-triggered pulls handle laptop-suspend recovery.

- Edge bootstrap with inlined snapshot: the why is first-bird under 500ms. HTML carries a signed, short-TTL snapshot so the renderer can paint birds before noncritical code and assets.

- Batched event ingest with client event_uid idempotency: the rationale is that retries from flaky mobile networks "never double-apply."

- Fast tick inline snapshot after reaction-relevant events: the why is responsiveness without client-side prediction. The API waits briefly for committed sim state so the client can render a reaction without an extra round trip; on timeout, polling catches up.

- Neutral magic-link request response: the rationale is "no account enumeration." Verification failures use the same matter-of-fact error surface.

- Account email-change flow: NOT RECOVERABLE FROM PLAN.

- Visit snapshot revocation check on every pull: the rationale is immediate revocation at the visitor's next poll with a matter-of-fact 410 surface.

- Scoped visitor route plus route-level guard plus DB-role guard: the plan calls this "defense in depth" so visit sessions cannot post interaction events.

### Simulation engine and sync

- Idle tick every about 60s with jitter: the plan ties this to the PRD's "~once per minute" anchor and to the idea that the server simulates even when no client is connected.

- Fast tick on reaction-relevant events: the rationale is "affective responsiveness" while preserving the single-writer invariant.

- Tick leases with crash-safe re-tick: the why is worker crash tolerance. Lease expiry permits re-tick, and idempotent event claiming makes a double-tick harmless.

- No-op fast path: the rationale is cheap polling and meaningful observability. If nothing changed, the tick writes nothing and does not bump state_version.

- Transactional tick pipeline: the plan says the transaction boundary guarantees events and state move together and makes the "no-last-write-wins rule" structural.

- Monotonic drift slow integrator: the why is to make sustained attention matter without allowing single-session manipulation or neglect punishment. Absence produces zero input and flat traits.

- Separate expressiveness layer: the plan explicitly says this resolves the simultaneous demands that traits never move down and ignored birds greet less often.

- Server-time event order: the rationale is that "client clocks lie." Fixed order makes ticks deterministic and replayable for calibration and incident forensics.

- Mood machine with time priors and hysteresis: the rationale is mood continuity without flicker. "Daily-ish reset" emerges from strong priors, not a hard reset, and minimum dwell prevents tick-to-tick flicker.

- Renames flowing through events and tick: the why is keeping the single-writer invariant absolute, with no "just this one field" exception for future contributors.

- Bird-to-bird call response, wary spread, and chorus flags: the plan's rationale is natural social behavior with controlled ensemble timing. Alarm spreads because "calm is not contagious; alarm is," and chorus members are staggered, never unison.

- Weather generator: the rationale is rare, short-lived atmosphere that never demands attention. The plan bans thunderstorms, snow, and anything attention-demanding.

- Notebook candidate detectors, sparsity governor, and prose writer: the why is naturalist observation without feed dynamics. Very active users get the same cap; candidates that lose are dropped because stale observations are worse than none.

- Settle persistence and re-engagement clearing: the plan's rationale is that settle is a state of quieting until the user re-engages or a new presence window clears it, not an auto-resetting morning mode.

- state_version cursor and full snapshot replacement instead of deltas: the rationale is that snapshots are kilobytes and delta infrastructure is unjustified complexity at v1.

- No realtime push in v1: the plan says polling at 45s is indistinguishable for a product whose fastest canonical change cadence is about one minute; fast ticks surface inline.

### Frontend rendering pipeline and audio

- Scene composition in painter's order: the rationale is a calm visual register where sky, foliage, perches, birds, weather, ornaments, and lighting cooperate without ornaments crossing bird faces.

- Viewport rules with no pan, zoom, or scroll: the why is that all birds are always in frame and never crop; sky and foliage extend instead of letterboxing.

- Bird rigs and idle micro-motion as code: the rationale is that motion never stops and the first frame joins an existing activity phase. "Paused" is banned, and activity_phase lets the client resume mid-cycle.

- Boot path with quiet field fallback and no spinner: the rationale is the "first frame is mid-action" invariant. The quiet field is acceptable because it is a field, while a spinner is a launch-blocking defect.

- Snapshot reconciliation with interpolation, blending, and phase continuity: the plan's why is preventing motion pops, jumps, and snapping after polling gaps or full-snapshot replacement.

- Reduced-motion mode: the rationale is that it remains "the aviary" with calls, captions, notebook, drift, and mood unchanged, and it is tested alongside full-motion so it cannot rot.

- Thin top bar and no chrome inside the scene: the plan's why is preserving the scene as an ambient aviary, with no tooltips, badges, labels, hover affordances, or in-scene chrome.

- Offers never triggered by clicking a bird: NOT RECOVERABLE FROM PLAN.

- Procedural audio motif library and per-bird voiceprint: the rationale is recognizability with variation. "Pip is always Pip by ear," while two renderings of the same call are never identical.

- Captions generated from the same parameter set as synthesis: the why is guaranteeing that "the caption matches what played."

- Listen-in mixer ramps and non-focused floor: the rationale is avoiding hard cuts while preserving ambient presence. Others drop to about 0.35, "never zero."

- Client-side call scheduler for unobserved calling: the plan says the tab "audibly lives" even when the user is not interacting, subject to autoplay.

- Audio resource discipline and node pooling: the rationale is no allocation churn and flat memory across the 30-minute audio soak.

- WebAudio fallback to silence plus captions: the why is "procedural or nothing." Recorded audio never ships, so captions carry the product when audio cannot.

- Autoplay handling without "click to enable sound" banner: the rationale is that such a banner would be an announcement. Captions render while suspended, and the first gesture resumes audio quietly.

### Accessibility, performance, testing, rollout, and risks

- Screen-reader naturalist narration: the rationale is delivering the actual product in one voice. It uses the same snapshot fields as visuals, avoids state-list narration and numbers, and only narrates significant observations.

- Keyboard navigation: the plan's why is parity with pointer behavior. Focus order reaches the scene and birds, Enter toggles listen-in, Escape restores focus, and keyboard listen-in uses the identical mix ramp.

- Procedural call captions: the rationale is audio-equivalent specificity and reliability. Captions come from call parameters, auto-enable when audio is unavailable, merge chorus captions, and avoid overlap with bird focus outlines.

- WCAG AA contrast and matter-of-fact settings/error surfaces: the rationale is accessibility plus voice separation. Chrome text is tested against real palettes, and system surfaces avoid naturalist idioms.

- Accessibility regression prevention: the why is preventing launch promises from decaying under schedule pressure. axe, keyboard walkthroughs, narration tests, reduced-motion visuals, and caption goldens block release.

- Performance and observability budgets: the plan makes them gates because first paint, frame timing, memory, snapshot size, and tick latency are directly tied to whether the aviary feels alive.

- First-bird under 500ms strategy: the rationale is preserving immediate life. Edge snapshots, one small blocking bundle, system fonts, lazy detail layers, and CDN assets all serve that budget.

- Aggregate-only synthetic/RUM/sim telemetry: the rationale is privacy-bounded insight. The tick is the product's heart, but dashboards must not reconstruct a user's relationship with their birds.

- Drift calibration harness: the plan's why is avoiding silent product failure. Too fast becomes "Tamagotchi-by-accident"; too slow becomes "screensaver"; the harness checks behavior-threshold visibility rather than raw numbers.

- Golden and voice tests for notebook, captions, and narration: the rationale is preventing banned phrases, announcement frames, exclamation, repetition, cadence creep, and voice drift.

- Definition of done per feature: the why is to make accessibility, voice, performance, privacy review, and feature flags part of every feature rather than cleanup work.

- Independently demoable milestones: the plan's rationale is embodied in exit criteria: each milestone proves an integrated product property, such as identical snapshots, calibration bands, first-bird timing, or deletion completeness.

- Birds-per-aviary age ramp: the why is growth without reward mechanics. Offers appear by aviary age only, "never engagement-based," quietly framed, and never with a count.

- Exact adoption pacing constants at 3, 6, 9, 12, and 15-18 months: NOT RECOVERABLE FROM PLAN.

- Instrumentation from day one: the rationale is operational visibility while staying aggregate-only: tick latency, no-op ratios, first-bird timings, frame timings, audio errors, snapshot sizes, auth success, invite counts, notebook counts, and harness results.

- PITR backups, cross-region replica, and restore drill: the why is that "personality loss is the worst failure"; restore rehearsal is an M7 exit criterion.

- Rate limits for magic links, event ingest, and invite sends: the plan's rationale is anti-flood and deliverability protection.

- Feature flags for user-facing surfaces and defaults: the plan uses flags to control launch, accessibility defaults, and fast-tick wait time without changing core invariants.

- Sim-worker incident posture: the rationale is graceful degradation. Clients keep rendering the last snapshot so the aviary remains visually alive while drift pauses and tick staleness alerts.

- Drift calibration risk mitigations: the plan says no test might fail if drift feels wrong, so calibration harnesses, input caps, persona trajectories, and forward-only retuning are used.

- Audio uncanniness mitigations: the rationale is avoiding synthetic-flat, uncanny-valley, and phase-cancel chorus failures. Voiceprints must cluster by bird, chorus schedules decorrelate, and a playtest asks whether Pip can be picked out.

- Timezone and DST handling: the why is local-time anchoring. The plan stores IANA zones, never offsets, and tests DST transitions so mood priors follow account-local time.

- Privacy boundary leak mitigations: the rationale is preventing well-meaning dashboards from joining sim data. The plan relies on credential separation, banned-field lint, and the rule that if a dashboard reconstructs a user's relationship with birds, "it does not exist."
