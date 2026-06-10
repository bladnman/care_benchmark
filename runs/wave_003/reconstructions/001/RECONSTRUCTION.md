# Pocket Aviary — phase-2A reconstruction from PLAN

## System-level intent

- **Canonical continuity, with no resets and no last-write-wins.** This intent shows up in the "state/performance boundary" where "Server owns state" and the client is only an "interpreter"; in the sync model where "sync is a property, not a feature"; in the statement that "two devices = two readers of one record"; and in the tick contract where catch-up folds and replays are exact. The product promise is that the aviary "kept running," a refresh never visibly "resets" anything, and the morning-laptop/lunch-phone overwrite scenario is "unreachable."

- **A quiet naturalist register instead of announcement, reward, or game framing.** This shows up in the explicit out-of-scope list for "streaks, badges, levels, counters," in the audio autoplay decision where there is "no 'enable sound' modal" and "no blocked-audio toast," in return-greeting where "no textual welcome surface of any kind exists," in offers where "a countdown is a meter — wrong register," and in launch where availability is "quiet" because "no growth mechanics exist."

- **Structural enforcement rather than editorial policy.** The plan repeatedly turns intent into schema, grants, lints, and absent components: "no per-account analytics dimensions exist," the personality vector "never leaves the server in raw named form," notebook generation has "no detector class that observes user behavior," the API service has "no UPDATE grant" on personality vectors, there is "no toast component," and there is "no welcome string class." The plan wants refusal paths to be physical in the system.

- **Privacy as infrastructure.** The plan names two "physically separate data paths" from day one: a simulation path containing per-bird, per-account interaction data and a telemetry path containing only "counts, latencies, error rates, anonymized histograms." This intent also appears in synthetic UUID identifiers, encrypted email cells, aggregate-only RUM, metric-definition lints, and the rule that calibration cannot use production user data because "the privacy rule is absolute."

- **Slow relationship, monotonic expression, and no resentment loop.** The bird engine is described as "monotonic-toward-expressive," with presence-time as the primary signal, daily caps, "asymptotic approach," and no negative trait deltas. The plan explicitly says neglect produces "no delta" and that quieter behavior comes from the expression layer, "not from trait decay." Risks frame the product as living in a "narrow band" between "Tamagotchi-feel" and "screensaver."

- **Procedural aliveness under hard budget and browser constraints.** The scene starts "mid-action," the renderer uses procedural skeletal birds, audio is client-side WebAudio with per-bird signatures, and loading is "literally the aviary's sky, never a spinner." The plan keeps this aliveness inside a bundle budget, first-bird timing, 60fps idle, zero memory growth, and browser autoplay policy.

- **Accessibility as a designed version of the same aviary.** The plan says accessibility surfaces "ship with v1, not after," are built by "the same engineers in the same milestones," and reduced-motion must be "the same aviary, different visual register." Narration, captions, keyboard focus, and reduced motion are not checklist add-ons; they carry the same naturalist voice and are gated in CI and beta.

- **Voice and sound require human craft loops.** The plan treats the writer and sound designer as "not nice-to-haves." Notebook, narration, captions, greetings, and call libraries are "the product's voice"; audio is "the affective spine"; and human voice review is "the one quality bar that stays human."

## Per-feature whys

### 1. Scope and v1 product surface

- **Single-user accounts and one canonical aviary per account** — The rationale is the single canonical record: there is "one canonical aviary per account," "two devices = two readers of one record," and "no merge, no LWW anywhere in the design."

- **Magic-link email sign-in as the account-entry method: NOT RECOVERABLE FROM PLAN**

- **Per-device revocable sessions as a v1 account surface: NOT RECOVERABLE FROM PLAN**

- **Email change with verification as a v1 account surface: NOT RECOVERABLE FROM PLAN**

- **Account export as JSON emailed by link** — The plan says export is async to keep "the API stateless" and avoid "long-poll generation requests"; it also says exported personality values are the named exception because "the user's numbers are theirs to take."

- **Soft-delete for 30 days, then hard-delete** — The plan ties the soft-delete window to recovery: "Recovery during the window restores by clearing deletion_requested_at." Hard deletion removes account, sessions, birds, vectors, events, notebook, invites, visit log, export artifacts, and issues a tombstone to purge account-UUID operational logs.

- **Browser support as last two major versions of Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN**

- **Explicit out-of-scope surfaces** — Native apps, payments, shared accounts, gamification, Tamagotchi mechanics, social-network surfaces, push/email/ping notifications, recorded-audio fallback, user-visible personality numbers, user perch placement, and editable notebooks are structurally excluded so they cannot be "just exposed" later.

- **Server-side simulation tick with snapshot pulls** — The tick preserves the continuity requirement: "the aviary that has been running." Polling is enough because the tick is 1/min, so push adds "no perceptible freshness gain" and keeps the API stateless.

- **Hidden personality vector with derived performance parameters** — The plan says raw trait values in a payload would be "a dev-tools dashboard waiting to happen" and would invite third-party stat tooling. Server-side derivation makes the no-numbers rule "cheap to keep."

- **Monotonic-toward-expressive drift driven primarily by presence-time** — The plan wants slow expressive change without decay: presence-time dominates, headroom creates diminishing returns, daily caps prevent saturation, and neglect produces "no delta" rather than resentment.

- **Persistent mood states** — The semi-Markov model gives "persistent, non-twitchy moods that survive sessions naturally," so "drowsy at dusk yesterday → settled by morning" emerges from dwell expiry and time-of-day weights, not a tab-open reset.

- **Bird-to-bird interaction, call/response, mood contagion, and chorus** — The plan wants "real chorus" behavior from overlapping timers, response propensity, warmth, and chorus-join gates, rather than canned simultaneous audio.

- **Six-species pool as exactly six species: NOT RECOVERABLE FROM PLAN**

- **Stable bird identity for account lifetime** — The bird id is "never reissued, never replaced across rename/sync/migration," because a reset bird is framed as the worst silent sync/data-loss failure.

- **User naming and renaming: NOT RECOVERABLE FROM PLAN**

- **Adoption flow with two system-selected starters: NOT RECOVERABLE FROM PLAN**

- **Age-gated bird offers up to seven** — The schedule is chosen to match the rough rhythm that "a few months old offers a third" and "a year-old may have five or six"; the hard cap also supports the recognizability premise for 5-7 birds.

- **Presence accounting with the three-condition conjunction** — The activity window is biased long because "watching without moving is the actual product." Server validation, deduping, and caps protect drift integrity from inflated presence.

- **Return-greeting** — The rationale is relationship without announcement. Bolder birds tend to greet first, wary birds may not greet, and greeting is visual/audio grammar; the plan explicitly forbids a textual welcome surface.

- **Listen-in** — The feature gives focused attention without erasing the rest of the aviary: the focused bird ramps up, others ramp "down to ambient floor, never zero," and only the focused bird gets warmth and vocal-frequency credit.

- **Seed, song fragment, and still pool as the three offer types: NOT RECOVERABLE FROM PLAN**

- **Offers with per-bird cooldown** — Server-truth cooldowns prevent client manipulation, and the client shows a dimmed affordance with "no countdown timer" because "a countdown is a meter — wrong register."

- **Settle with 5-second undo** — Settle is a quieting surface, not a trait driver: engine-wise it "only closes the presence window cleanly" and has the same end-state as tab close.

- **Field notebook** — The notebook is sparse, read-only, and naturalist-voice so it remains an observation record; the hard line is that it can say "pip investigated the seed slowly" but never "you visited every day this week."

- **One-screen scene with no in-scene chrome and quiet-field loading** — The scene itself carries loading and empty states: the sky layer "is" the quiet field, "never a spinner," and no spinner asset exists.

- **Three perch zones: NOT RECOVERABLE FROM PLAN**

- **Responsive layout without cropping birds** — The solver keeps "all birds in frame at every viewport"; horizontal spacing changes, but "nothing crops, nothing pans."

- **Client-side procedural audio calls** — Procedural sound keeps the bundle inside budget, avoids a recorded-audio fallback, and lets each bird have a recognizable signature with "no two calls identical" and "no call off-signature."

- **Accessibility surfaces** — Accessibility is in scope as designed surfaces: narration, reduced motion, captions, and keyboard navigation are part of the same milestones and acceptance criteria as visual/audio work.

- **Read-only ambient social visits** — Visits are ambient only; visitors do not see notebook, offers, settings, or pending bird offers because "the notebook is the host's relationship record," and visit tokens have no event ingestion path.

- **Privacy via synthetic UUIDs and aggregate-only telemetry** — The rationale is that per-bird interaction state belongs only to the user's simulation path; telemetry has no bird, personality, mood, or notebook dimensions.

- **Performance budgets** — The budget exists to keep the first impression alive on modest devices: first bird visible under 500ms, 60fps idle on older hardware, and no memory growth over a 30-minute session.

### 2. Architecture overview and defensible calls

- **Three deployables plus static edge layer** — The web client owns the "performance layer"; the API stays stateless; the tick service is the only personality writer; and the edge layer makes "time-to-first-bird achievable."

- **PostgreSQL as the single source of truth, with no Kafka or separate cache tier at v1** — The snapshot is "kilobytes and read-mostly," and fewer moving parts mean fewer ways to violate the single-writer rule.

- **Shared deterministic engine package** — Determinism lets server and client share drift math, mood transition, call grammar, and prose grammars while preserving exact catch-up folds, exact recovery replays, fast calibration, and no laptop/phone disagreement.

- **Simulation path separated from telemetry path** — The plan wants per-bird, per-account data never read by analytics; separate networks, roles, review checklist, and CI metric lints enforce that boundary.

- **HTTPS polling instead of WebSockets** — The plan says the PRD specifies the pull model, the tick is 1/min, push adds "no perceptible freshness gain," and polling keeps the API stateless.

- **Layered Canvas 2D instead of a game engine** — With at most seven birds and sparse ornaments, Canvas 2D should hit 60fps; a WebGL engine spends bundle budget and risk on capability the plan says v1 does not need.

- **Audio starting only after user gesture** — Browser autoplay policy makes first-frame audio unimplementable. The plan preserves "no entry ceremony" by making the scene visually alive, fading sound in on first input, using captions from frame one, and avoiding modals or nagging.

- **Adaptive tick cadence with fixed semantics** — Dormant aviaries can be advanced by catch-up folds because the fold is exact and bit-identical to minute-by-minute ticking; this preserves continuity while avoiding pure cost.

- **Raw traits never shipping to the client** — The rationale is to prevent a named `boldness: 0.62` from becoming a product surface through dev tools or third-party stat tooling.

- **Last-reported IANA timezone for canonical mood inputs** — With two devices in different zones, "last-reported wins" is called rare, low-stakes, and self-correcting on next use.

- **Four-minute presence activity window** — The plan picks a long starting value because it should count quiet watching; it remains config-calibrated in beta.

- **Server-scheduled weather and perch assignments** — Multi-device coherence is the reason: rain on laptop but not phone would break the "same aviary" promise.

- **Client-realized call scheduling inside server-set bands** — Exact cross-device audio sync would require sub-second server-clocked events "for no user-perceivable benefit"; the canonical layer is synced and the performance layer is local.

- **Semi-Markov mood model** — Dwell times and hysteresis create "persistent, non-twitchy moods" and avoid mood flapping.

- **Bird offer age-gate schedule** — The dates are chosen to match the plan's rough curve and stay "pure config."

- **Drift calibration without production user data** — The privacy commitment forbids population-level analysis of interaction patterns, so calibration uses a synthetic harness and explicit-consent internal/beta accounts.

- **Visitors not seeing the notebook** — The plan says the ambient visit is read-only and the notebook is "the host's"; it is not part of the ambient scene.

- **Async export** — The job-and-link model keeps the API stateless and avoids long-running generation requests.

### 4. Data model and 5. API surface

- **Encrypted email and email hash** — Email appears only in the encrypted account column and transient mailer payloads; the HMAC hash exists for uniqueness and lookup only.

- **Magic-link endpoint returning 202 and single-use token consume** — Returning 202 prevents an account-existence oracle, and atomic consume makes expired or replayed links matter-of-fact failures rather than security holes.

- **Personality vector table written only by the tick service role** — The grant structure makes the "no-last-write-wins rule" physical.

- **Drift ledger** — The ledger exists for "auditability, the export, invariant checks," and disaster recovery by re-folding vectors from ledger plus seed.

- **Append-only semantic interaction events** — Semantic payloads avoid raw coordinates or keystroke contents, and idempotency keys make retries safe.

- **Server-coalesced presence intervals and credit caps** — The tick reads intervals rather than raw pings; dedupe and daily caps prevent two devices or long idle sessions from inflating drift.

- **Server-enforced offer cooldowns** — Snapshot-truth `available_at` keeps the client honest while preserving quiet affordance state.

- **Immutable notebook entries** — The API is read-only and rows are append-only so the notebook remains an observation record, with deletion only through hard account deletion.

- **Visitor email as a deliberate second encrypted PII cell** — The plan scopes this to the invite feature and to showing the address back to the host in the visit log.

- **Visit token class with no event ingestion path** — The visit snapshot is read-only by construction; visitor pulls write to visit log only and write nothing to interaction events or presence intervals.

- **Hard deletion issuing a log-pipeline tombstone** — This purges account-UUID-scoped operational log lines within the log retention window, extending deletion beyond primary tables.

- **Matter-of-fact API errors** — The voice rule applies to every error string: the surface states what happened and what to do without announcement framing.

- **Idempotency on mutating endpoints** — `client_event_id` and request keys make retries safe and keep duplicate events from changing simulation state.

- **Dormant snapshot catch-up before responding** — A snapshot request for a dormant aviary triggers synchronous catch-up so observable state is current while staying under the tick-latency p99 alarm.

- **Server-derived listen-in duration** — The server derives duration from start/end pairs and does not trust client-claimed durations beyond sanity bounds.

### 6. Simulation engine, 7. interactions, 8. sync, and 9. rendering/audio

- **Pure deterministic tick** — No wall-clock reads, unseeded randomness, or I/O inside the tick buys exact catch-up, exact replay, fast calibration, and impossible laptop/phone disagreement.

- **Workers claiming due aviaries with `SKIP LOCKED`** — The plan uses this for horizontal scaling "with no coordinator and no double-tick."

- **Drift headroom** — `(1 - trait)` gives asymptotic approach, no overshoot, and diminishing returns.

- **Daily per-trait cap** — The cap prevents "single-session saturation" and is a second defense against engine collapse from burst interaction.

- **Neglect producing no negative drift** — The plan wants absence to mean quieter expression, "not trait decay"; this protects the non-Tamagotchi promise.

- **Calibration targets for day 7 and day 21** — Named numeric targets make drift "instrument-measurable" by day 7 and visibly expressive by day 21.

- **Expression mapping from state to performance parameters** — This is the single place trait numbers become behavior, so visible timing and reverse-engineering risk are both controlled there.

- **Seeded return-greeting selector and choreography grammar** — The greeting should vary across sessions but remain stable within one, and variation is "real," not "N canned variants in rotation."

- **Greeting firing within 1-2 seconds** — It is part of the critical path budget and part of the first-render relationship impression.

- **No textual welcome surface** — This enforces the no-announcement register; the voice lint has no "welcome" string class.

- **Listen-in equal-power ramps and ambient floor** — Ramping keeps focus soft and musical, while "never zero" keeps the aviary ambient instead of soloing a bird into an instrument panel.

- **Offer outcomes reported from the realized client response** — The tick credits curiosity and boldness consistently with what the user actually saw.

- **Presence pings evaluated at send time with no retroactive buffer** — The plan prevents stale activity from being credited after focus or visibility is lost.

- **Single canonical sync model** — Clients append events; the tick folds both sessions in order; a newer snapshot reconciles by interpolation, never by merge conflict.

- **Stale-render interpolation and bounded movement** — Newer state moves birds smoothly to new perches or moods, and the tick bounds movement so birds do not teleport.

- **Render-gap detector after suspend/resume** — A reopened laptop pulls a snapshot before motion resumes, so it shows the aviary that kept running, not a frozen scene that jumps.

- **Preact or equivalent only for chrome surfaces** — Keeping the view layer tiny and code-splitting route surfaces protects the critical scene, engine-runtime, and audio bootstrap chunk.

- **Quiet field loading as the same sky component** — Loading is "literally the aviary's sky," so the first surface stays in product register and no spinner asset exists.

- **Procedural skeletal birds in layered Canvas** — The plan uses skeletons, pose systems, and dirty-rect compositing to keep motion expressive while meeting the 60fps and bundle constraints.

- **First-frame mid-pose rule** — Initializing pose clocks from `motion_anchor` creates motion-already-in-progress with no entry transition and no fade-from-static.

- **Halting rendering while hidden and pulling snapshot on resume** — This saves work and prevents stale scenes; audio also suspends to silence gracefully.

- **Layout solver for phone and desktop** — The rationale is "all birds in frame at every viewport" with no crop and no pan.

- **Per-bird voice signatures** — A stable hash of `bird_id` keeps a bird recognizable across mood and drift while mood and vocal-frequency modulate expression.

- **Call scheduler and chorus emergence** — Poisson-ish timers, response propensity, and chorus-join gates make call/response and chorus emerge instead of being fixed tracks.

- **Pooled buffers, particles, and no per-call allocations** — This directly feeds the "no-memory-growth gate."

- **Caption hook from each synthesis plan** — Captions describe the call that actually played because they consume the same realized plan as the audio.

- **Graceful silence fallback with captions auto-enabled** — If AudioContext is unavailable or denied, the product stays usable without adding a recorded-audio path.

- **Single light-state controller for day/night and settle** — Sky, palette tinting, and audio ambience share one controller so visual and audio "evening" agree.

### 10. Notebook and 11. accessibility

- **Notebook detectors observing aviary state transitions** — Entries come from actual bird, weather, chorus, offer, perch, arrival, or plumage state, making prose specific rather than generic.

- **Sparsity governor** — Dropping below-threshold candidates rather than queuing them ensures "sparsity holds even for very active users."

- **Naturalist prose realizer and voice lint** — Templates, variation slots, and linting keep notebook entries lowercase, present-tense, bird-named, and observational.

- **No user-behavior detector input schema** — Structural absence keeps the notebook from becoming a diary of the user's visits or habits.

- **Accessibility surfaces built in the same milestones** — The plan says "nothing here is post-launch," and each renderer/audio work item carries narration, captions, and reduced-motion acceptance criteria.

- **Screen-reader narration from realized client state** — Narration uses the same state the visuals draw from, so it describes the aviary as observations and the product sounds like "one product across surfaces."

- **Copy registry with `naturalist` and `system` register tags** — This enforces lowercase present-tense naturalist copy, plain system copy, and a blocklist for gamification and announcement language.

- **Runtime captions near calling birds** — Captions map contour, tempo, intensity, and source perch from the actual synthesis plan, and must keep WCAG AA contrast in bright and dim scenes.

- **Reduced-motion as a second renderer** — The feature is not a broken low-motion mode; it must read as "calmer," with calls, captions, drift, mood, and notebook unchanged.

- **Keyboard and focus model** — Full keyboard operation, focus traps, and high-contrast focus indicators make the scene and chrome accessible in both bright and dim states.

- **Session cookie flags and rotating token: NOT RECOVERABLE FROM PLAN**

### 12. Accounts/privacy, 13. performance, 14. rollout, 15. risks, 16. testing, and 17. team

- **Export containing current personality vectors** — This is the named exception to hidden numbers because "the user's numbers are theirs to take," but the values stay out of product surfaces.

- **PII audit gate in CI** — Schema migrations, log statements, and metrics are scanned to enforce the sanctioned email columns and telemetry boundary.

- **Plain-text privacy policy page** — It names aggregate telemetry categories and explicitly excludes per-bird interaction state, matching the infrastructure boundary.

- **Bundle budget split into critical, deferred, and route chunks** — Procedural audio and bird art make the budget fit; recorded audio and sprite sheets would spend too much budget.

- **Cached snapshot and edge-cached HTML for first bird under 500ms** — Returning users see the last IndexedDB snapshot immediately while fresh state races; cold sessions use a small nearest-region snapshot and the quiet field covers the gap.

- **Runtime degradation ladder** — The renderer drops parallax, ornament cadence, and offscreen resolution before ever dropping bird motion.

- **Thirty-minute memory soak as CI** — The plan calls zero growth a "CI test, not a guideline," backed by heap and AudioContext node sampling.

- **Synthetic fleet and aggregate RUM** — Observability measures first-bird-time, frame rate, audio init, and latency while keeping "no per-account dimensions."

- **M0 foundations before product data** — Repo/CI, budget gates, voice lint, PII lint, schema grants, auth, deploy, and telemetry split are stood up "before any product data exists."

- **M1 engine vertical slice gate** — The engine harness is required before affective work because calibration targets must be demonstrable in simulation.

- **M2 renderer gate** — First-frame-mid-action, presence, quiet field, and reduced-motion ship together, gated by first-bird timing and 60fps.

- **M3 sound and interaction gate** — Calls, chorus, listen-in, offers, settle, greetings, captions, narration, and keyboard nav are gated by ABX recognizability and screen-reader pass.

- **M4 relationship layer gate** — Notebook sparsity, voice lint, adoption, naming, offers, export, deletion, and email change are validated on a 60-day simulated aviary.

- **M5 visits and hardening gate** — Invite/revoke/expiry/log, settings, unsupported-browser, memory soak, security review, load test, and recovery replay must pass before launch.

- **Dogfood and closed beta with consenting cohorts** — Presence-window calibration, drift tuning, greeting/notebook tone, audio panel, accessibility sign-off, and perf gates use consent because production interaction analytics are off-limits.

- **Open launch as quiet availability** — The plan says there are no growth mechanics to ramp, so launch verifies serving and operations rather than adding engagement loops.

- **Post-launch bird population milestones** — The first day-60 third-bird offers are called out because chorus density rises then, requiring audio-recognizability monitoring to re-run.

- **Operational kill-switches** — Config-only switches avoid deploys and preserve visible continuity; drift freeze is safe because monotonic drift means freezing never visibly regresses a bird.

- **Drift mis-calibration mitigation** — The plan treats this as real because the product sits between Tamagotchi-feel and screensaver; the mitigation is deterministic harness, numeric targets, consenting validation, config constants, drift freeze, and separate expression mapping.

- **Sync/data-loss mitigation** — The risk is "the worst, silent failure," so the plan uses DB grants, append-only events, idempotency, drift ledger re-folding, PITR backups, and alerts on any non-tick vector write.

- **Audio recognizability mitigation** — Because audio is "the affective spine," the plan uses sound-design work, ABX recognizability gates, spectral spacing, milestone re-runs, and config reduction of chorus-join rather than recordings.

- **Autoplay-policy mitigation** — Since browsers block pre-gesture sound, the plan uses visual aliveness, mid-call fade-in, captions, and no blocked-audio nagging.

- **Presence distortion mitigation** — Drift integrity depends on presence honesty, so the plan uses the exact conjunction, server re-validation, dedupe, daily caps, a long window, dogfood tuning, and anomalous-ping counters.

- **Accessibility regression mitigation** — The risk is deadline erosion toward checklist mode; mitigations put accessibility acceptance criteria in each definition of done, plus CI, scripted screen-reader runs, paid testing, and reduced-motion design review.

- **Tick fleet scaling mitigation** — The 1-minute cadence times account count is the main scale axis; exact catch-up, `SKIP LOCKED`, backlog alarms, and 10x load tests reduce that risk.

- **Magic-link deliverability mitigation** — Magic link is "the only door into the product," so the plan uses transactional ESP, domain warmup, DMARC/SPF/DKIM, monitoring, and failover behind an interface.

- **Notebook tone mitigation** — Because template smell compounds, the plan uses writer-owned grammar, voice lint, sparse volume, beta sample review, and a minimum number of realized-prose variants per detector.

- **Timezone/DST mitigation** — The plan accepts that mood and lighting can drift slightly for one session; standard IANA handling, tests, and last-reported-wins make it self-healing.

- **Scope-creep mitigation** — The plan expects "one harmless toast" pressure and blocks it with lints, no toast component, no welcome string class, no streak-capable schema, no per-account analytics dimensions, and a PR checklist.

- **Engine unit and property tests** — Determinism, monotonic drift, fold equivalence, mood dwell/hysteresis, and presence caps verify the core invariants.

- **Calibration harness** — Simulated regular, sporadic, intense, and absent-then-returning users assert numeric drift targets and "quieter-not-resentful" return behavior.

- **Sync and integration tests** — Multi-device interleavings, idempotent retries, tick crash/replay, and attempted visit-token event posts prove that unsafe scenarios are unreachable.

- **Audio tests** — ABX recognizability, no-two-identical-calls, mix ramps, and fallback silence plus captions test the affective and accessibility spine.

- **Performance CI** — Bundle budgets, first-bird synthetic gates, 30-minute memory soak, and frame-time regression make performance a release gate.

- **Accessibility CI and manual testing** — Axe, voice lint, NVDA/VoiceOver scripts, keyboard tests, and contrast checks verify the designed accessibility surfaces.

- **Privacy and PII gates** — Schema, log, metric lints and quarterly access audit keep the telemetry/simulation boundary intact.

- **Human voice review** — Generated prose samples for notebook, narration, captions, and greetings keep the voice quality bar human.

- **Team assumptions for writer and sound designer** — The plan says they are "not nice-to-haves" because notebook, narration, captions, and call libraries are the product's voice; the engine calibration harness is the long pole for affective work.
