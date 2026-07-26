## System-level intent

- **Refuse the wrong product structurally, not by backlog discipline.** This shows up in "Out of scope for v1 - and structurally refused," where non-goals are made "harder to build later" through missing schema substrate, missing route paths, CI lints, and absent primitives. The plan repeatedly prefers "absence ... in the DDL, not in a policy doc."

- **Keep the server/client boundary simple and product-shaped: "the server owns discrete state; the client owns continuous presentation."** This principle appears in Architecture and is reused in the sync model, rendering, audio, visitor view, and API shape. The plan's test is whether "two devices showing the same aviary at the same moment" would disagree.

- **Make the aviary continuous in the strong sense through deterministic replay.** The "load-bearing systems trick" is that the tick is "logically continuous" and "physically tiered, with exact catch-up." The plan wants "the aviary that has been running" to be "true in the strong sense, not approximated."

- **Treat personality and bird identity as canonical, durable, and never client-authored.** This appears in the single-writer sim-worker, `personality_vectors` as the runtime read, `birds.id` as "STABLE FOREVER," fixed `voice_seed`, and the rule that no runbook contains "reset," "regenerate," or "swap."

- **Prefer mechanisms over style guides or code-review hopes.** The opening says affective rules become "a mechanism (a lint, a type, a CI gate)." The same pattern appears in voice lints, anti-announcement tests, schema CHECK constraints, Postgres grants, branded PII types, and tainting tests.

- **Make drift gentle, monotonic, and legible only over time.** The drift section is calibrated so day 7 is "instrument-measurable" but below the "perceptual threshold," while day 21 is where "you notice when you look back." The mood layer implements "quieter than they were, not sick."

- **Reject gamification, Tamagotchi mechanics, and announcement surfaces as product-voice violations.** The plan bans counters, "streak," "achievement," "welcome back," toasts, banners, confetti, and live-region announcements. It says "just a small toast" is the "single most likely charm-destroying change."

- **Use a naturalist voice for the aviary and a matter-of-fact voice for system surfaces.** This shows up in the chunk split (`aviary` vs `system`), the two string catalogs, the voice-kernel lexicon, and the lints that enforce lowercase naturalist prose, no second person, and system copy that "must state what happened and what to do."

- **Design accessibility as a first-class surface, not a fallback.** The plan says "Accessibility ships with v1 or v1 does not ship" and calls narration, reduced-motion, and captions "designed surfaces with their own product work, not fallbacks." It also extends those surfaces to visitors.

- **Make privacy a topology and type-system property, not just a promise.** The plan says the telemetry boundary is "implemented as network topology, not policy." Email lives in "exactly one place"; the telemetry stack has "no route" to sim data; metrics cannot carry account IDs.

- **Hide trait numbers even from devtools except for the one export exception.** The snapshot "carries behavior ... never traits," so "the user never sees the numbers" holds "with devtools open." The export section flags a genuine collision and makes it "the single documented exception."

- **Make the first frame feel already alive.** The frontend plan says "No spinner exists in the codebase," "No fade-in," "No entry animation," and "The first painted frame is a frame from the middle of a continuous animation."

- **Make recognizability empirical rather than aspirational.** Audio uses an "invariant/variant split" so a bird sounds like itself across mood and drift, and launch requires a recognizability study. "Seven is empirical" is treated as a gate and a configurable cap.

- **Keep social ambient, scoped, and read-only.** The visitor projection is derived by "deletion" from the host snapshot, "structurally forecloses show-off mode," and visitor activity produces "zero change in the host's drift ledger."

- **Keep the notebook sparse and observational, not a feed or user report.** The sparsity governor, detector refractory periods, and `subject_kind` constraint all enforce that entries are about a `bird` or `aviary`, never the user. The plan says this keeps the notebook from "becoming a feed."

- **Treat performance as affective, not merely technical.** The risk section says above 500ms "the product becomes an app that loads," and the budget section treats 2MB as a hard ceiling while targeting about 800KB because the "500ms budget is the one that actually matters."

- **Expect the most important failures to be silent.** Risks such as drift miscalibration, presence inflation, accessibility regression, catch-up divergence, and personality corruption are described as degrading the product "without throwing an error." Each has a detection mechanism.

- **Ship by measured gates, not vibe checks.** Launch gates are "measured, none is a vibe check," including drift thresholds, recognizability, first-bird timing, memory, anti-announcement, telemetry-boundary, visitor-no-drift, and restore-drill gates.

## Per-feature whys

### Scope

- **Single-user accounts** - NOT RECOVERABLE FROM PLAN.

- **One aviary per account** - NOT RECOVERABLE FROM PLAN.

- **Magic-link sign-in** - NOT RECOVERABLE FROM PLAN.

- **15-minute, single-use magic links** - The auth section frames these as security properties: 256-bit tokens, stored hashed, constant-time compare, purpose-bound, and consumed in the same transaction that mints the session.

- **Per-device revocable sessions** - The plan's rationale is account control and security: sessions are hashed at rest, revocable per device, and shown as a UA class rather than a raw UA string.

- **Email change with verification** - The plan sends verification to the new address and a matter-of-fact notice to the old address because omitting that would be "a security defect."

- **JSON export** - The plan treats export as an explicit account surface and flags that including personality vectors collides with "never exposing numbers"; the rationale given for including them is that the export rule is "more specific and explicit."

- **Soft-delete for 30 days before hard delete** - The 30-day soft-delete window gives a matter-of-fact recovery affordance, while the sim continues to tick so "a recovered account has not lost 30 days of its aviary."

- **Two starter birds** - The plan's articulated reason is recognizability: starter selection is constrained so the two starters occupy "different call registers" because recognizability must work "from day one."

- **Cap of 7 birds** - The cap is treated as empirical: the recognizability study must pass at the shipping bird cap, and if it fails, "the ship cap drops to the largest passing count."

- **Three perch zones** - The plan makes perch a behavioral and sonic signal: front/middle/back affect visual position, call gain, reverb send, and probability functions, and "perch is a signal, not a layout."

- **Local-time day/night** - Local time supports circadian mood and light curves. The plan chooses timezone offset plus date only because "Location is PII we have no reason to hold."

- **Ambient weather** - Weather is deterministic so cold-account replay is exact, and `weather_next` lets the client "pre-warm" layers and cross-fade rather than popping them on.

- **Ambient micro-motion** - The rationale is aliveness: a "never-still test" fails if every bird's parameter vector does not change in every 500ms window.

- **Top-bar fade** - The fade keeps chrome quiet while preserving access: it is opacity only, never disables hit-testing, and never fades while focused, open, or screen-reader-active.

- **Five-trait personality vector** - The vector is canonical stored state and the plan keeps trait scalars off the wire so "the user never sees the numbers."

- **Monotonic drift** - The plan wants long-term change without punishment: deltas are clamped non-negative, absent birds hold traits, and neglected birds become "ambient, not distressed."

- **Mood state machine with cross-session persistence** - Persistence prevents a session-start reset; the plan asserts opening a session produces "zero mood writes."

- **Procedural call grammar** - The rationale is variation without recordings: calls are realized from motifs and transforms so there are no canned repeats and no recording phase-cancellation artifacts.

- **Mood-shaped idle motion** - Motion becomes expression: `drowsy`, `alert`, `wary`, and `curious` change rates, amplitudes, and gestures.

- **Bird-to-bird interaction** - Call-and-response, contagion, and chorus emergence make multi-bird behavior a "felt property" without scripting showpiece events.

- **Six-species pool** - The decision log chooses species for "distinct registers and rhythms" and says the nightjar-like species is required by night behavior.

- **Age-gated new-bird arrivals** - Age-only gates keep arrivals from becoming rewards. The plan says no interaction inputs feed the gate "by construction," and the arrival is "not a reward."

- **Return-greeting** - The greeting should feel immediate and continuous: it is precomputed in the bootstrap snapshot for "zero round trips" and selected from absence and bird state rather than client claims.

- **Listen-in** - Listen-in focuses a bird without turning the aviary into "soloable tracks"; other birds never go silent, with a hard -9 dB floor.

- **Offer: seed, song fragment, still pool** - Offers need immediate feel without client authority. The client reacts from server-supplied disposition, while the tick records the authoritative event.

- **Offer cooldown** - The plan chooses 4 minutes because with daily saturation at 3 investigated offers, "curiosity cannot be saturated by button-mashing."

- **Settle** - Settle closes the presence window cleanly, biases birds toward `drowsy`/`settled`, lengthens call intervals, and creates a quiet ramp rather than an exit announcement.

- **Five-second settle undo** - Undo is local and instantaneous because "the only way a 5-second mercy window feels like a mercy" is not to wait for confirmation.

- **Field notebook** - The notebook captures sparse observations, not a feed. Detectors, cooldowns, novelty gates, and subject constraints keep it about the aviary and birds, not user activity.

- **Presence accounting** - Presence is visible, focused, and recently active because only the client can observe those signals, but the server clamps them because the signal is forgeable and bug-prone.

- **Snapshot pull** - Full snapshots are small, and full-state replacement "removes an entire class of merge bugs."

- **Append-only client event log** - The event log makes writes idempotent, ordered, auditable, and recoverable while keeping clients from authoring personality state.

- **Social visit invitations** - Invitations are opt-in, scoped, expiring, and revocable so social remains read-only and ambient rather than a network surface.

- **Read-only ambient visitor view** - The visitor projection deletes private fields from the host snapshot and excludes greeting because "the visitor's arrival must not be noticed by the birds."

- **Silent visit log** - Visitor activity is kept out of `interaction_events` so a visit "produces zero change in the host's drift ledger."

- **Opt-in visit notification toggle default off** - The plan keeps visit notification optional and off by default to preserve the quiet surface and avoid turning visits into an announcement channel.

- **Screen-reader narration** - Narration is server-generated from the same state as the visual scene so it "cannot drift from the scene" and is identical across devices.

- **Reduced-motion mode** - Reduced motion is a separate `CrossfadePresenter` because the plan wants "a calmer aviary, not a broken one."

- **Call captions** - Captions are generated by the same function as audio so caption and sound "cannot diverge."

- **WCAG AA and keyboard navigation** - Accessibility is a launch condition; every pointer action must be reachable by keyboard and copy must remain readable in day and night states.

- **Supported browser and viewport set** - NOT RECOVERABLE FROM PLAN.

- **Initial bundle, first-bird, frame-rate, and memory budgets** - The rationale is affective continuity: above the first-bird budget the aviary becomes "an app that loads."

### Architecture

- **Seven deployable units** - The plan keeps services "small on purpose" because the "interesting complexity is in the sim, not in the topology."

- **`aviary-edge` with inlined bootstrap snapshot** - This is the hot path for first paint: the client needs "zero round-trips before drawing the first bird."

- **Edge timeout serving HTML without snapshot** - The edge "never blocks the document"; on timeout the client draws the quiet field and pulls normally.

- **`aviary-api` for read-heavy surfaces** - The rationale is cacheability: snapshots, notebook pagination, settings, renames, and export requests are read-heavy and cacheable.

- **Separate `events-api`** - Separating writes lets it be rate-limited and degraded independently; if it fails, "the aviary still renders and only drift accrual pauses."

- **`sim-worker` as the only writer of bird state and personality** - This enforces "clients never write personality state" through database roles, not code review.

- **`auth-svc`, `visits-api`, and `mailer-svc`** - NOT RECOVERABLE FROM PLAN.

- **Route-level chunks: `boot`, `aviary`, `system`** - Chunking protects first paint and makes the naturalist/matter-of-fact voice split mechanically lintable.

- **Preact plus signals for chrome** - Preact keeps top bar, notebook, and settings cheap against the 2MB budget; the plan rejects React for bundle size.

- **No framework inside the canvas** - The plan says DOM reconciliation "has no role in a canvas scene."

- **Canvas 2D with layered offscreen canvases** - Canvas 2D holds 60fps with 7 birds, costs almost no bundle, avoids WebGL GPU wake cost, and makes reduced-motion cross-fade easier than DOM/CSS animation.

- **Raw WebAudio** - Tone.js is rejected as too large and as fighting server-scheduled intents; the needed scheduling can be written in about 6KB.

- **TypeScript for API, edge, and sim-worker** - One language lets `voice-kernel`, `call-grammar`, and `drift` produce byte-identical results on server and client.

- **PostgreSQL 16 with partitioned event table** - Strong ordering, row-level locking, and a single-writer sim match Postgres's shape; Kafka is rejected as operational weight.

- **Redis for cache, leases, and rate limits** - The plan names Redis for snapshot cache, tick leases, and rate limits.

- **Tiered physical ticking with exact catch-up** - This preserves a logically continuous 60s tick while making 1M accounts affordable.

- **`stale: true` catch-up fallback** - Long catch-ups complete asynchronously while the request returns last-known state because "the aviary is never a load state."

### Data model

- **Email in exactly one column** - The privacy rule is "impossible to retrofit," so email is encrypted in one place and all other references use the synthetic account UUID.

- **`email_lookup_hash`** - HMAC lookup supports sign-in without storing reversible email or a join key.

- **`accounts.id` as the only identifier elsewhere** - This prevents email from appearing in foreign keys, service calls, logs, metrics, queue keys, sharding, or error reports.

- **`aviary_seed`** - The seed makes species selection, weather, and PRNG streams deterministic.

- **`max_birds` with CHECK 2 to 7** - The database enforces the bird ceiling rather than leaving it to UI policy.

- **`settings` JSONB** - NOT RECOVERABLE FROM PLAN.

- **`birds.id` stable forever** - Bird identity continuity is load-bearing; if state is unrecoverable, the incident procedure is PITR, "not to reseed."

- **Bird name renameable but never identifier** - This lets the user author a name while preserving durable identity by UUID.

- **Fixed `voice_seed`** - The plan says identity by ear is as stable as identity by id.

- **`retired_at` reserved but unused** - The plan states "birds do not die," aligning with refusal of Tamagotchi mechanics.

- **Stored `personality_vectors`** - Runtime reads always read a single stored row because personality is "never derived from session history at runtime."

- **`epoch_hi` idempotency** - It makes drift epoch application safe if a tick is retried.

- **`drift_deltas` ledger** - The ledger supports forensics and exact recovery from corruption while remaining "not a runtime dependency."

- **`bird_state` persistence** - Mood, gesture, phase, perch, and cooldown are stored so a returning client resumes mid-action and does not reset state.

- **Partitioned `interaction_events`** - Monthly partitions support append-only retention and deletion of raw behavior after 90 days.

- **`subject_kind` limited to `bird` or `aviary`** - This makes user-centered notebook entries unrepresentable and blocks gamified observations.

- **Realized notebook prose stored permanently** - A voice change must not rewrite history.

- **Session UA class instead of raw UA** - The device list remains useful without retaining raw user-agent detail.

- **Encrypted visitor email and hashed invite token** - Visitor email follows the same PII discipline as account email, and tokens are not stored raw.

- **No hunger, health, happiness, xp, levels, visit counts, streaks, achievements, or cross-account aggregates** - These absences are "load-bearing" and make gamification or Tamagotchi mechanics require a reviewable migration.

### API surface

- **Errors in matter-of-fact voice with stable codes** - System surfaces state what happened and give the client a code to map without naturalist prose.

- **Snapshot payload `render` palette instead of trait value** - The endpoint carries behavior and palette buckets, never traits, so trait numbers are unavailable even in devtools.

- **Snapshot `window_ms` forward window** - Forward calls and light curves mean the client always has material to render and never stalls on a poll.

- **Snapshot `greeting`** - Precomputing it at snapshot time makes the return-greeting fire within 1-2s with no extra round trip.

- **`since` optimization with no partial diffs** - Full-state snapshots avoid merge bugs; `since` can return `204` but never partial state.

- **Batched event post** - Batching every 15s, on visibility change, and via `sendBeacon` preserves events without chatty writes.

- **Event idempotency on `(account_id, cid)`** - Duplicate client events are counted and dropped.

- **Server-side timestamp clamp** - Clamping to `[now - 300s, now + 5s]` prevents suspended devices from dumping backdated presence or future events.

- **Ignoring client `offer.outcome` for drift** - The sim recomputes the authoritative outcome so client-observed presentation cannot forge drift.

- **Auth request always returning 202** - This prevents account enumeration.

- **Magic-link rate limits** - The plan uses per-email, per-IP, and global circuit breaker limits to reduce abuse and protect mail delivery.

- **Notebook pagination, newest first** - NOT RECOVERABLE FROM PLAN.

- **Read-only notebook with no PATCH/DELETE** - Entries are stable observations and history is not rewritten or curated.

- **Bird rename endpoint accepts name only** - Trait fields are absent from the request schema so clients cannot author or inspect traits.

- **Account settings endpoint** - NOT RECOVERABLE FROM PLAN.

- **Async export emailed as signed 24h link** - The signed short-lived link limits exposure of exported account data.

- **Soft delete and undelete endpoints** - Undelete is available on any signed-in page during the 30-day window, preserving recovery.

- **Visit invitation endpoints with max 20 outstanding and 30-day expiry** - The expiry and cap constrain the scope of sharing.

- **Visitor projection derived by deletion** - This reuses the same snapshot generator, preserves accessibility parity, and structurally forecloses "show-off mode."

- **Visitor client reduced control set** - Read-only visitors do not ship listen-in, offer, or settle handlers, making writes unavailable in the client as well as the API.

- **Visitor writes return `403 visit_read_only`** - The visitor session is ambient and read-only by construction.

- **Visitor activity not written to interaction events** - Visits cannot affect the host's drift ledger.

- **Invite revocation returning `410 visit_unavailable`** - Revoked visits stop working on the next poll while using a matter-of-fact surface and no host confirmation.

### Simulation engine design

- **Pure `packages/sim` tick** - Purity makes deterministic replay, golden fixtures, time-warp, catch-up equivalence, and safe tiered ticking possible.

- **Per-account Redis lease and Postgres advisory lock** - These enforce single-writer ticking per account.

- **Events consumed in sequence and transactionally** - The tick either commits entirely or not at all; `consumed_by_tick` prevents reprocessing.

- **Drift epoch as one local day ending at 04:00** - The decision log says a late-night session belongs to the day it began.

- **Presence/listen/offer drift signals** - Signals tie long-term changes to presence, listening, and investigated offers while letting settle contribute nothing.

- **Low-pass drift update with diminishing returns** - `(1 - t)` means a bird "retains reserve indefinitely" and no user maxes out a trait.

- **Daily `DRIFT_SESSION_CEILING`** - A marathon session cannot move any trait visibly in one day.

- **Instrument vs perceptual drift thresholds** - The plan makes drift measurable at 7 days but noticeable only around 21 days, matching "you notice when you look back."

- **Behavior mapping from traits** - Perch probability, call rate, and quantized plumage steps create the user-visible feel without exposing trait numbers.

- **Drift observatory** - Synthetic profiles in CI catch calibration shifts without using production data.

- **Monotonicity enforced by type, database, and property test** - The plan wants no negative-drift code path to compile or persist.

- **Six mood states** - The decision log adds `settled` because night behavior requires a distinct state from `drowsy`.

- **Mood hysteresis and 90s minimum** - This prevents rapid mood flapping and creates a settled quality.

- **Circadian mood curves** - Local time shapes alert, content, drowsy, and settled patterns so the aviary follows day/night.

- **Weather mood influence** - Rain and wind influence moods mildly, making weather behavioral as well as visual.

- **Mood contagion double-buffered** - Reading previous-tick neighbor moods prevents oscillation and makes evaluation order irrelevant.

- **Personality bias in mood** - Boldness, curiosity, and species make the same context produce bird-specific behavior.

- **Perch evaluation at most every 8 ticks** - Birds feel settled rather than constantly hopping.

- **User cannot influence perch directly** - No API field exists for perch, enforcing "perch is a signal, not a layout."

- **Slots avoiding collisions and warmth adjacency** - This makes birds share space legibly while high-warmth birds bias toward adjacency.

- **Gesture scheduling by mood** - Gestures communicate state: preen, scan, tilt, fluff, shuffle, and low-sit all reflect mood.

- **Gesture and phase in snapshots** - Opening mid-preen renders mid-preen, supporting the "already in motion" premise.

- **Call-and-response** - Replies make birds respond to one another from warmth and vocal traits, not from scripted scenes.

- **Chorus emergence** - Scheduler nudges overlapping calls so choruses happen "naturally and often enough to be a felt property."

- **Pre-generated deterministic weather schedule** - It supports exact replay and lets the client cross-fade weather in.

- **Return-greeting absence computed server-side** - The plan never trusts absence from the client.

- **Greeter weighted by boldness, warmth, mood, and recency** - Greeter identity varies in a way that tracks drift and makes notebook observations true rather than coincidence.

- **Greeting absence buckets** - Short absences get a glance; long absences get approach, so greeting scale follows time away.

- **Greeting variation test** - The test enforces "real, not a rotation of variants" by requiring distinct transform stacks and timings.

- **Greeting staggering** - Enforced spacing makes simultaneous greeting unreachable.

- **Offer server disposition mirrored to client** - Immediate reaction feels responsive while server and client cannot disagree.

- **Song offer through same audio engine** - This keeps offered song fragments sonically part of the same aviary.

- **Pool offer reflected with Canvas 2D** - The plan explains the implementation, but its product rationale is NOT RECOVERABLE FROM PLAN.

- **Settle event light ramp and longer call intervals** - Settle turns the aviary toward evening and quiet.

- **Settle undo correlation by `cid`** - The tick can treat settle plus undo as a no-op even if the settle already reached the server.

- **Presence close on settle, tab close, or ping stop** - The engine gets a clean boundary even if the browser misses `sendBeacon`.

- **Age gates at 90/210/365/545/730 days** - The plan says these match "a few months -> third bird; a year -> five or six" using age only.

- **New bird appears wary on back perch with quiet naming affordance** - The arrival is intentionally not a modal, reward, celebration, or confetti moment.

- **Species excluding species already present** - This keeps new arrivals distinct until all six are used.

### Sync model

- **No client-side authoritative state** - With one canonical record, "there is no merge."

- **Database grants for single-writer discipline** - APIs refuse to boot if over-granted, ensuring only the sim can update personality and bird state.

- **Closed event schema with no trait fields** - A last-write-wins personality bug is "unrepresentable."

- **Presence credited with per-ping clamp** - Overlaps or replays cannot credit extra presence.

- **Presence capped at 6 hours/day** - This never binds for real users because drift saturates earlier, but caps client-bug blast radius.

- **Presence `vis`, `focus`, and `act_age_ms` echo** - The server can reject pings where the client dropped one of the three conditions.

- **Activity window of 4 minutes** - It is long because "watching birds without moving is the product," and finite because an empty room must not count.

- **Presence-minutes distribution monitor** - Presence inflation has "no user-visible symptom and no failing test," so the population distribution is watched.

- **Snapshot pull on visible, resume, keepalive, and interaction** - The client updates when the user can see the aviary or after server-adjudicated interactions.

- **No polling while hidden** - There is nothing to see, and hidden tabs should not spend battery.

- **Discard older snapshot versions** - Monotonic `v` makes out-of-order responses harmless.

- **No WebSockets in v1** - A 60s tick and 90s forward window make 30s polling sufficient; sockets add failure modes and buy nothing.

- **Last-write-wins timezone** - Timezone is a hint about where the user is now, so the newest device is the best answer.

- **Last-write-wins bird name** - The name is user-authored, low-stakes, and immediately visible.

- **Per-key last-write-wins settings** - Per-key JSONB merge prevents a stale phone from reverting another setting.

- **Server-time clock offset estimate** - Scheduled calls land within about 50ms across devices, so host and visitor hear the same aviary.

### Frontend rendering pipeline

- **Four canvas layers** - Separate redraw rates keep `sky` cheap, `mid` and `fore` smooth, and weather/parallax composited efficiently.

- **Frame-loop `dt` clamp** - A suspend cannot produce a giant catch-up step.

- **Loop stops on hidden tabs** - This saves battery and matches "there is nothing to see."

- **Resume advances phases by elapsed wall time** - The resume frame proves the bird kept preening while the tab was away.

- **Small skeletal rig per bird** - Compact geometry supports several animated sub-parts with little bundle cost.

- **SVG authored and compiled to Path2D commands** - This keeps species assets small while allowing runtime palette application.

- **Runtime plumage palette** - Plumage drift is "free at the asset layer" because geometry need not change.

- **Seeded continuous micro-motion** - A bird's fidget signature is its own and identical across devices.

- **Jittered gesture blends** - Blends prevent snapping and keep gestures from feeling canned.

- **Perch flight interpolation** - New perches are reached by flight rather than teleport.

- **Mid-flight retargeting** - Retargeting from current position and velocity guarantees "no teleport, ever."

- **Ambient ornaments as client-side pooled objects** - Ornaments require no server agreement and allocate nothing after warmup.

- **Subtle parallax** - The plan says it is "subtle by mandate," with small smoothed offsets and disabled reduced-motion behavior.

- **Responsive virtual coordinate space** - The tested invariant is that no bird is cropped at any viewport size.

- **Reduced-motion presenter interface** - A second presenter prevents reduced motion from becoming "`if (reducedMotion) return;`."

- **Cross-fade still-pose sets** - The pose sets are authored so vestibular users get a calmer designed aviary.

- **Top bar with exactly four items** - A test forces any addition to reopen the conversation.

- **Faded bar remains hit-testable and keyboard-reachable** - Fade changes opacity only; access is not reduced.

- **No spinner component** - The absence prevents accidental loading states and supports the "first frame from the middle" premise.

- **Inlined first-paint path** - Critical CSS, snapshot, and boot module are sized so first bird can appear under 500ms.

- **Quiet field fallback** - When no snapshot is inlined, the field reads as "the aviary catching up" rather than as a spinner or error.

- **Empty aviary fly-in only after adoption** - This is the single fly-in-from-offscreen; after adoption, the user never sees an empty aviary again.

- **Client never ticks or infers mood/personality** - This preserves the server-owned discrete state boundary.

### Audio pipeline

- **Schroeder-style algorithmic reverb** - It gives an "outdoor space" cue at negligible size without convolution impulse responses.

- **Ambient bed** - NOT RECOVERABLE FROM PLAN.

- **Call grammar motifs and transforms** - Fresh realizations create variation from a structured grammar rather than recordings.

- **Pure `realize` function shared across devices** - Same seeds produce the same call on every device and in captions.

- **Fixed identity cues at adoption** - Motif library, pitch centre, timbre, breathiness, vibrato, and attack shape make a bird recognizable.

- **Mood/personality modulation of expression cues** - Phrase count, gaps, tempo, ornament density, intensity, and interval let a bird express state while still sounding like itself.

- **Server absolute call times plus 200ms lookahead** - WebAudio can schedule sample-accurate starts without high-frequency timers.

- **Zone gain, reverb, and high-shelf cut** - Distance is audible, so a bold bird coming forward sounds as well as looks closer.

- **Chorus mixing from genuine overlapping realized calls** - Since there are no recordings, simultaneous birds cannot phase-cancel like duplicate samples.

- **Voice budget dropping quietest back-zone voice** - The plan bounds concurrent syllable voices while minimizing audible loss.

- **Listen-in gain and reverb change** - The focused bird moves "forward in the space, not just up in volume."

- **Non-focused bird -9 dB floor** - Others never go silent because silence would make the aviary "soloable tracks."

- **Listen-in disengage triggers** - Multiple exits let pointer and keyboard users leave focus naturally.

- **Recognizability launch gate** - The cap only ships if participants identify birds at the threshold; otherwise cap drops to the passing count.

- **Autoplay suspended handling with no prompt** - Browser policy is handled "without announcement"; first interaction fades audio in mid-call.

- **Captions during silent period** - Captions cover the autoplay gap without adding an unmute surface.

- **Accessibility-settings-only audio explanation** - The explanation lives where a user solves a problem, "not on the aviary surface."

- **No recorded-audio fallback** - With WebAudio unavailable, captions are forced on and the product plays in silence; there is no audio file loader.

- **WebAudio memory pooling** - `PeriodicWave`, noise buffers, envelope arrays, and auxiliaries are cached or pooled because audio is the likely leak source.

### Voice kernel - notebook, narration, captions

- **Shared `voice-kernel`** - The same lexicon, template graph, anti-repetition memory, and catalogs make notebook, narration, and captions sound like "one observer."

- **Lexicon-limited generators** - Words outside the allowed observer vocabulary cannot be emitted.

- **Template graph with seeded PRNG** - Multiple realizations create variation while remaining deterministic.

- **Anti-repetition memory** - The last 40 phrase-template ids prevent the notebook from reading like "a mail merge after two months."

- **Naturalist and system string catalogs** - They enforce the naturalist/matter-of-fact split by surface.

- **Voice lints** - Lints prevent second person, announcements, gamification, trait numbers, and catalog-boundary leaks.

- **Server-side screen-reader narration** - Narration is generated from the same state as visuals and therefore cannot drift from the scene.

- **45s idle narration cadence** - The decision log picks the center of the 30-60s band.

- **Prioritized narration updates** - Greeting, offer, settle, listen-in, and notebook events are timely without waiting for idle cadence.

- **Narration coalescing queue** - The middle update is dropped if three arrive so the user is not forced through backlog.

- **20s idle suppression after priority update** - This is an explicit rule against talking over the user.

- **Narration composition by salience** - It mentions the most salient bird, then another notable bird, then aviary condition, making prose instead of a list.

- **Single polite live region** - `polite`, never `assertive`, avoids interrupting the user's own work.

- **Visitor narration** - Accessibility parity "does not stop at the host's own account."

- **Notebook observation detectors** - Detectors look for greeter changes, firsts, quiet intervals, chorus, weather moments, unusual perches, proximity, and night calling so entries are eventful observations.

- **Notebook sparsity governor** - Cooldowns, novelty, and caps keep the notebook from "becoming a feed."

- **Heavy-profile notebook test** - It proves active users get more moments but not proportionally more.

- **Notebook subject constraint** - `subject_kind` prevents entries about the user, so "you visited every day this week" is not expressible.

- **Permanent notebook prose** - Entries are never regenerated because "a voice change must not rewrite history."

- **Caption generation from `realizeCall`** - Caption and audio share source data and cannot diverge.

- **Caption position near calling bird** - Captions associate text with the bird and avoid overlap with the bird, other captions, and viewport edges.

- **Caption scrim** - A tuned subtle backdrop gives WCAG AA at every light level without a hard box.

- **Captions `aria-hidden` when narration is active** - This prevents a screen-reader user hearing each call twice.

- **Captions default on when WebAudio unavailable** - Silence is compensated by an accessibility surface rather than an audio fallback.

### Accessibility surfaces

- **No automated state `aria-label`s** - State reaches screen readers as prose through the voice kernel or not at all, avoiding mechanical state dumps.

- **Visitors get accessibility surfaces** - A read-only view is "still a view."

- **Scene as composite keyboard widget** - A keyboard user is not forced through seven tab stops before reaching settings.

- **Roving bird focus** - Arrow navigation maps to visual order and perch zones, making the scene keyboard-operable.

- **Offer popover focus trap and restore** - Pointer actions must have keyboard equivalents.

- **Settle undo triggered by any keypress** - Keyboard users get the same mercy window as pointer users.

- **Shortcuts documented in accessibility settings, not on the surface** - The plan keeps shortcut text off the aviary surface while still documenting it where accessibility controls live.

- **Dual-stroke focus ring** - Light and dark strokes remain visible against bright noon and dim night backgrounds.

- **Focus forcing top bar to full opacity** - The fade never suppresses focus visibility.

- **Automated contrast checks in light and night states** - Contrast depends on the aviary background, so it is checked in both states and for captions at multiple light levels.

- **`prefers-reduced-motion` honored before first paint** - This avoids a motion flash.

- **`prefers-contrast: more` and reduced transparency** - These settings strengthen chrome/focus or disable the fade to respect user preferences.

- **Accessibility settings contents** - Reduced motion, captions, narration detail, and audio belong in a matter-of-fact settings surface.

- **Accessibility settings prefetched on first top-bar focus** - A keyboard user should not wait for a lazy chunk.

### Performance budgets and observability

- **Initial JS budget under 2MB and critical path under 60KB** - The hard ceiling protects first-bird timing; the working target leaves headroom for art and reduced-motion pose sets.

- **First bird under 500ms** - The plan treats first-bird time as the primary affective budget.

- **60fps idle on a 2019 laptop** - Seven birds must remain calm and alive without dropped frames.

- **No memory growth over 30 minutes** - Long calm sessions should not slowly degrade, with audio singled out as a likely leak source.

- **Snapshot latency and tick latency budgets** - These keep reads and sim updates timely.

- **Tick freshness SLI** - Freshness corresponds to the user's experience of a live aviary more directly than latency.

- **Chunk allocation under about 708KB** - Coming far under 2MB is deliberate because the 500ms budget "actually matters."

- **Server metrics** - Operational health covers endpoint latency, tick freshness, backlog, event ingestion, cache hit rate, DB saturation, and mailer health.

- **Aggregate-only client RUM** - The plan collects performance and error shape without per-account dimensions.

- **Synthetic browser fleet** - Synthetic sessions assert first-bird time, frame rate, audio output, and no surfaced error from several geographies.

- **Product health metrics as aggregate only** - Session-duration histograms and sign-in/account rates are allowed, but not per-user cohorts.

- **Synthetic-only calibration** - Drift trajectories are measured from observatory profiles, "never from real accounts."

- **No per-account or per-bird metric dimensions** - The plan avoids creating a data product or a substrate for streaks and retention cohorts.

- **No notebook/narration content in telemetry** - Observed prose remains outside logs and metrics.

- **Presence-minutes alarm** - Presence inflation is silent, so distribution shifts get urgent investigation.

### Privacy and security engineering

- **Envelope-encrypted email ciphertext** - Email storage is isolated and encrypted.

- **HMAC email lookup hash with KMS pepper** - Sign-in lookup is possible while the value is not reversible or useful as a join key.

- **Log scrubber with counter** - Redaction hits are treated as bugs to fix, not success.

- **CI grep banning email in logs/metrics/spans** - This makes PII leakage reviewable at build time.

- **Branded `Pii<string>` type** - Logging and metrics cannot accept PII because no overload exists.

- **Visitor email same discipline** - Invitations do not become a weaker PII path.

- **Purpose-bound tokens** - Sign-in and email-change tokens are not interchangeable.

- **No account enumeration** - Auth request responses are identical for known and unknown emails.

- **CSRF and CSP controls** - SameSite cookies, origin checks, and nonce-tagged JSON bootstrap reduce web attack surface.

- **Old-address notice after email change** - The plan distinguishes account-security mail from product notification and calls omission a security defect.

- **Telemetry account/VPC with no route to simulation DB** - Privacy is enforced by network topology.

- **Closed `AllowedDims` union for metrics** - Per-account metric dimensions are unrepresentable without changing shared types.

- **Analytics role denied sim tables** - No ETL can read personality, state, events, or notebook entries.

- **No ML access to per-bird fields** - Because the pipeline cannot read them, future ML cannot quietly consume them.

- **Hard delete across all account-keyed rows** - Deletion removes the account's rows from all named tables and queued mail.

- **Hard-delete audit UUID and timestamp only** - The audit record proves deletion without retaining content.

- **Backups retained 30 days** - Hard-deleted data ages out of backups within the same promise window.

- **Event retention after 90 days** - Raw events are dropped because drift has already been applied and longer retention would hold a behavioral record.

- **Export current vectors despite trait-number rule** - The plan calls this a genuine tension, makes it a singular exception, and recommends explicit product review.

### Testing strategy

- **Golden replay corpus** - Byte-comparing vectors, moods, perches, calls, and notebook prose catches behavior changes as fixture diffs in review.

- **Replay-equivalence test** - It proves lazy catch-up and minute-by-minute ticking are bit-identical, making the cost optimization safe.

- **Property tests for drift and events** - They enforce monotonicity, bounds, idempotency, presence clamping, ordering, and mood hysteresis under hostile inputs.

- **Time-warp harness** - It makes months-long drift, sparsity, age gates, and manual QA testable in seconds.

- **Frame-time capture** - The 60fps budget is enforced with throttled headless rendering.

- **Never-still render test** - It encodes the aliveness requirement as a parameter-change assertion.

- **Viewport invariant test** - It enforces "Never crop a bird."

- **Reduced-motion render test** - It proves motion is calmer while calls, drift, and mood still occur.

- **Audio variation test** - The same test encodes "never canned" and "always recognizable" by checking variant and invariant parameters.

- **Offline audio render check** - It catches clipping, silence, and spectral outliers in a 7-bird chorus.

- **30-minute memory test** - It targets leaks across offers, listen-in, notebook, tab hide/show, AudioNodes, and ornament pools.

- **Anti-announcement suite** - It prevents likely friendly regressions: status regions, welcome text, extra top-bar items, toast components, second person, modal arrivals, and trait leaks.

- **Accessibility CI and manual scripts** - These catch regressions invisible to sighted or motion-tolerant developers.

- **Load tests with cold-account thundering herd** - The catch-up path must degrade to `stale: true`, not timeouts.

### Rollout

- **Seven-person team shape** - NOT RECOVERABLE FROM PLAN.

- **Accessibility work inside each milestone** - The plan says accessibility is "never after."

- **M0 skeleton with voice lints and anti-announcement suite from day one** - These are live before there is anything to violate.

- **M1 one bird alive** - The first milestone proves a server-ticked bird can look alive, call procedurally, and pass golden replays.

- **M2 seven-bird aviary** - This milestone proves chorus, distinguishability, listen-in, day/night, weather, and 60fps together.

- **M3 interactions and voice** - Return-greeting, offers, settle, notebook, narration, captions, and reduced motion are grouped because they make the product feel and speak correctly.

- **M4 accounts and sync** - Multi-device, export, delete, sessions, visits, and telemetry boundary prove one aviary, read-only visitors, and privacy topology.

- **M5 calibration and hardening** - Drift calibration, performance, memory, load, and beta all become launch gates.

- **Internal beta starting week 18** - The plan starts it six weeks before launch because "the calibration feedback loop is inherently three weeks long."

- **Closed beta with accessibility over-sampling** - The product explicitly seeks screen-reader, reduced-motion, and audio-off users because those surfaces are launch-critical.

- **Feedback by direct interview, not in-product prompts** - An in-product survey would be "an announcement surface."

- **`MAX_BIRDS_CEILING` launched at 3 and ramped** - Ramping is honest because seven is empirical and must be proven by study and real traffic.

- **Age gates implemented early with time-warp tests** - No rushed work is needed when the first accounts become old enough.

- **Backdated internal accounts** - Mature aviaries are exercised in production from launch day rather than discovered at day 730.

- **Instrumentation from day one** - The plan says observability ships with M0 or with the feature, "not retrofitted."

- **Post-launch daily and weekly watch** - The first 30 days monitor silent failure modes: tick freshness, presence inflation, first-bird RUM, audio failures, mail delivery, drift trajectories, and notebook sparsity.

### Risks, decisions, and appendices

- **Drift miscalibration mitigation** - The plan uses a closed-form function, observatory, and three-week beta because too fast becomes Tamagotchi and too slow becomes screensaver.

- **Personality vector restore drill** - Vector loss is the worst named failure, so restoring from PITR plus ledger is a launch gate.

- **Daily ledger-vs-canonical consistency job** - It catches silent vector corruption that ordinary unit tests would miss.

- **Presence synthetic hidden-tab client** - Hidden tabs must accrue zero presence, catching presence inflation regressions.

- **No toast/snackbar/banner/confetti primitive** - The design system omits these because announcement creep is expected from well-meaning contributors.

- **Migration review for per-account counters** - Adding a counter column becomes a visible event, blocking gamification creep.

- **Synthetic sign-in probe through a real mailbox** - Magic-link deliverability matters because there is "no password fallback."

- **Weekly notebook sample review** - Notebook genericness would discredit the voice, so beta includes sample review and template-distribution checks.

- **`packages/sim`, `packages/call-grammar`, and `packages/voice-kernel` dependency rule** - "No I/O, no clock access, no unseeded randomness" is what makes replay, time-warp, captions, and catch-up work.

- **Development environment with fast tick and time-warp** - This supports local testing of the sim and long-term behavior.

- **Staging synthetic only** - Full topology can be tested without production copies.

- **Production data never copied lower** - Debugging uses ledger and structured logs by UUID rather than cloning account data.
