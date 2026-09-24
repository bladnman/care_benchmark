## System-level intent

- **Affective aliveness is an engineering invariant, not a mood board.** The plan says the product's value is "affective" and treats "feels alive", "notice, never announce", and "no gamification" as "engineering invariants with enforcement." This shows up again in the weekly "aliveness review", the "charm rubric", and launch gates that require the aviary to feel alive for screen-reader and reduced-motion cohorts.

- **Quiet noticing replaces announcement.** The plan repeatedly prefers behavior in the aviary over chrome or copy: "The greeting is the welcome", "no toasts, banners, badges, confetti", "no spinner", and "nothing" or "natural motion" for most sync and failure states. Product communication is meant to be noticed in birds' motion, calls, notebook prose, and greeting choreography rather than announced by system UI.

- **No gamification, no pressure, no visit-frequency surfaces.** The plan excludes "achievements, streaks, levels, scores, XP, badges" and says there will be "No growth experiments on engagement." The notebook input schema deliberately omits presence/session facts, telemetry omits engagement cohorts, and newcomer affordances have "no reminders, no expiry message."

- **Hidden personality should become legible only through living behavior.** The plan's rule is that "No trait value ever reaches a client"; snapshots carry "compiled presentation parameters" and the notebook makes drift legible "in prose rather than numbers." This intent also appears in copy bans on trait words and narration rules that describe "observable behavior and posture, never mood labels" or numbers.

- **The relationship must move only upward and never punish absence.** The plan's drift philosophy is "additive, monotonic, saturating" and "No negative path exists." Absence affects only "attunement", outside the personality vector, so returning after a long absence feels like "being noticed, not like repairing damage." Birds "don't die, starve, or show distress."

- **Honest attention is valuable, but engagement extraction is not.** Presence is a strict "visible and focused and recent activity" conjunction, server-clamped and unioned across devices. Daily saturation is explicitly there so "no single session moves a trait visibly" and a "mouse jiggler" cannot exceed the heavy persona band. At the same time, the plan refuses retention dashboards, engagement A/B tests, and per-account engagement metrics.

- **Determinism is a product trust mechanism.** The plan makes the simulation a pure, deterministic step function with fixed-point math and a counter-based PRNG. It emphasizes "exact equivalence" between warm and cold ticking, "bit-identical" cross-engine results, projection consistency, and forward-only engine versions so sync, performance, and calibration do not undermine the relationship.

- **Privacy is architectural, not merely policy.** The plan says "The privacy boundary is architectural": identity and aviary data live in separate databases, telemetry is in a separate network/account, emails are encrypted and blind-indexed, support cannot read bird state, and metric schemas exclude account, bird, species, mood, trait, and interaction-type dimensions.

- **The first frame must already be the aviary.** The plan insists on "First frame is the aviary", an edge-streamed response, an inline snapshot, a "first-bird" module, and "no spinner anywhere in the codebase." Slow paths use a quiet field or cached snapshot so loading never becomes the experience.

- **Accessibility is a designed register of the same product.** The plan says "Accessibility is a designed register, not a fallback" and later sets the bar that "every user gets the actual product." Reduced motion has its own renderer, screen-reader narration uses naturalist prose, captions match actual calls, and accessibility panels are launch gates rather than later fixes.

- **Voice is split, registered, and machine-checked.** The plan distinguishes `naturalist` and `system` registers, with lowercase naturalist prose for aviary surfaces and direct matter-of-fact system copy for identity/errors/settings. It enforces the split through a copy registry, banned vocabulary, terminology lint, and writer review.

- **Bird identity and accumulated state are sacred.** Stable identity appears as "never reset, regenerated, or swapped"; M3 data is "permanent from here on"; and risk SC-1 calls personality loss "the worst failure the product can have." The design treats resets, decreases, or bad migrations as potential loss of the relationship.

## Per-feature whys

### How to read this plan

- **Initial calibration values in versioned engine config:** The plan says tunable numbers live in versioned engine config, not code constants, so they can be tuned by calibration while hard PRD budgets remain gates.

- **Hard performance and reliability budgets:** The plan treats 500 ms, 2 MB, 60 fps, and p99 5 s as hard gates because they come from the PRD rather than from calibration.

### Executive summary and product guardrails

- **Server-authoritative deterministic simulation:** The server is the only ticker so clients render and append events but "never tick"; this supports single-writer state, deterministic replay, and consistent cross-device results.

- **Warm and cold tick tiers:** Cold aviaries run the same one-minute steps in batches so dormant aviaries are cheaper without changing state; read-time projection makes them look current without waiting for a batch.

- **Additive monotonic saturating drift:** Traits change only through non-negative deltas toward per-bird ceilings so attention can make birds more expressive without neglect ever lowering traits.

- **Attunement outside the personality vector:** This exists to satisfy "quieter after absence" without wariness, distress, fading color, or trait decrease; it changes only viewer-directed orientation.

- **Presence as visible, focused, recent activity:** The conjunction aims at honest attention, while server union avoids double counting across devices and saturation bounds pathological cases.

- **Compiled snapshots instead of trait values:** Snapshots expose colors, call-rate parameters, perch assignments, and reaction plans rather than raw vectors to preserve the hidden-personality invariant.

- **First-bird edge boot and no spinner:** The plan wants birds mid-action on the first frame; slow paths are a quiet field or cached snapshot because loading chrome would violate "First frame is the aviary."

- **Canvas 2D cut-out renderer behind `DrawList`:** The stated reason is 60 fps on 2021-class integrated graphics while preserving an escape hatch to WebGL2 without changing scene code.

- **Procedural AudioWorklet audio:** Procedural calls avoid audio files, share grammar with captions, keep a fixed voice pool, and make CI able to enforce the no-recorded-audio rule.

- **Naturalist accessibility surfaces:** Screen-reader narration, reduced motion, captions, keyboard access, contrast, and visual narration ship in v1 because accessibility is part of the intended product, not a fallback.

- **Separate identity, aviary, and telemetry boundaries:** The plan uses separate databases, network paths, credentials, and schema allowlists so privacy commitments are enforced structurally.

- **Machine-checked voice split:** User-visible strings are tagged naturalist or system so lints can catch gamification, announcement language, terminology drift, and second-person naturalist prose.

### Product invariants and scope

- **No user-visible personality values:** The protocol has no trait fields, the client has no trait type, and support tools cannot read trait columns because the plan forbids any version or tier from showing values.

- **No toasts, banners, badges, confetti, or welcome-back UI:** The plan's rationale is "notice, never announce"; the greeting itself carries the welcome.

- **No gamification or visit-frequency surfaces in the notebook:** Detector input omits presence/session records so notebook entries cannot become streaks, visit facts, or user-behavior summaries.

- **Only `sim_writer` updates personality and aviary state:** This prevents last-write-wins behavior and makes drift an append-only, server-computed consequence of events.

- **Trait-decrease database trigger and nightly audit:** These provide multiple enforcement layers because monotonicity is considered essential to the relationship.

- **Visitors never generate presence:** Visitor bundles omit the event writer and server rejects visit credentials so observing someone else's aviary does not alter it.

- **Stable bird IDs and pinned species versions:** Bird identity must never reset, and new species versions must not change an existing bird's look or voice without an explicit migration.

- **Synthetic account UUID plus encrypted email:** The plan avoids using email as an identifier, stores it once encrypted, and keeps logs, metrics, and partition keys on UUIDs or blind indexes.

- **No recorded audio including fallback:** The product commitment is procedural calls only; silent-plus-captions is the fallback, not recorded samples.

- **Two starter birds, cap of seven, newcomers by age:** NOT RECOVERABLE FROM PLAN

- **No distress states or absence-to-mood path:** Birds should not die, starve, fade, or show distress, so absence has no mood or ease effect.

- **Magic-link auth, sessions, email change, deletion, and export in v1:** NOT RECOVERABLE FROM PLAN

- **One aviary per account in v1:** NOT RECOVERABLE FROM PLAN

- **Naming starter birds and renaming any time:** NOT RECOVERABLE FROM PLAN

- **New birds by aviary age only:** This avoids visit-frequency or achievement-like triggers and supports "no gamification."

- **Unsupported-browser surface:** NOT RECOVERABLE FROM PLAN

- **Privacy policy link and aggregate operational telemetry:** The policy names only aggregate operational categories and states that per-bird interaction data runs only the user's aviary.

- **No native apps, payments, household aviaries, customizable scenes, passwords, co-presence, or species picker in v1:** NOT RECOVERABLE FROM PLAN

- **No growth experiments on engagement:** The plan refuses tests whose metric is return frequency, session length, or retention to avoid optimizing the product around engagement.

- **No in-scene "enable sound" button:** The plan says that would be "Chrome inside the scene; announcing"; the muted glyph lives in the top-bar accessibility affordance instead.

- **No production debug panel with trait values:** Only synthetic lab tools may show them because production has no per-bird value surface.

- **No aggregate drift analysis across users:** Calibration uses simulation and consented staging studies because aggregate production drift would violate the privacy boundary.

- **No LLM-written notebook entries:** A third-party data flow would violate the privacy commitment, and a hand-authored grammar gives determinism and testability.

- **No retention dashboard:** Only aggregate operational health is allowed.

### Decisions log

- **Raw personality vectors in account export only:** The plan allows them only in the user-requested export file, never rendered in-product, and makes the setting reversible by config pending PM/privacy counsel.

- **Settle in the offer affordance's gesture popover:** This keeps the top bar to exactly four icons while still making settle accessible from the same gesture area.

- **Mute recorded as `audio_state` and vocal credit at 0.3x:** The plan includes mute as a drift input without allowing any negative drift.

- **Six moods:** NOT RECOVERABLE FROM PLAN

- **Code mood name `roosting`:** The plan uses it to avoid collision with the user's `settle` gesture while prose may still say "settled for the night."

- **Five-minute presence activity window:** The plan chooses this because the PRD says "a few minutes, lean long", with calibration range 3-8 minutes.

- **Activity signals as pointermove, pointerdown, keydown, wheel:** `keypress` is deprecated, touch may not fire pointermove, and wheel counts because scrolling is deliberate attention while the aviary itself never scrolls.

- **One-time invite link converted to browser-bound visit pass:** The link is consumed on first open so later viewing depends on a revocable pass rather than reusing the secret link.

- **Ninety-day active visit-pass lapse:** NOT RECOVERABLE FROM PLAN

- **Visit notifications by email only:** There is no push surface in the product, and wording remains matter-of-fact.

- **Notebook entries render current names:** This keeps identity continuous after rename while freezing wording and variation at write time.

- **Offer cooldown with lingering items:** Birds in cooldown can glance but earn no drift; lingering items let a cooldown-ending bird still come, and a 20 s client pace prevents mashing.

- **Settle lighting local but settle event canonical:** Other devices see calmer birds, not another device's evening light.

- **Hidden-tab audio fade and suspend:** The plan names battery, background-timer throttling, and the rule that hidden tabs do not count presence.

- **Autoplay behavior with no prompt:** The plan tries boot resume when browsers allow it, otherwise stays silent until the first activation so the scene does not announce audio setup.

- **Six species with one nocturnal species and non-nocturnal starters:** Starters avoid the nocturnal species so a midnight sign-up still gets an awake first meeting; distinct call registers support recognizability.

- **Newcomer eligibility days and jitter:** The plan maps "a few months" and "a year" to default dates, but the specific thresholds are NOT RECOVERABLE FROM PLAN.

- **Timezone auto-follows most recent session start with manual override:** This resolves "the user's local time" across devices, and the 20-minute ramp prevents abrupt lighting changes.

- **No gendered pronouns in generated bird prose:** The plan avoids assigning pronouns and uses names, "the bird", or species epithets.

- **Pending-deletion account page instead of aviary:** The plan lets the aviary keep ticking while signed-in pages show "I changed my mind"; invites and passes are suspended and restorable.

- **Email-change link lifetime of 24 h:** NOT RECOVERABLE FROM PLAN

- **Old-address notice on email change:** The plan labels this as security.

- **Session lifetime of 60-day sliding idle and one-year absolute:** NOT RECOVERABLE FROM PLAN

- **Captions off by default, on when WebAudio unavailable:** The on-by-default fallback follows the PRD so silent mode still has call information.

- **Visual narration display:** The plan treats it as implied by WCAG and renders the latest utterance as a quiet bottom strip.

- **Visit-log retention of 12 months:** Visitor emails are PII, so log entries are deleted after that period.

- **Magic-link GET never consumes token:** Link-scanning proxies cannot burn links because only the POST consumes the token.

- **Lowercase bird names in naturalist prose:** This matches the notebook voice samples while settings/system surfaces keep typed casing.

- **Bird-size floor before other layout compromises:** The floor supports "never crop a bird" on 320 px viewports and makes the seven-bird cap a design-review gate.

### Architecture

- **Home region for primaries, serving regions for API/cache/replicas, global edge:** The plan keeps writes off the first-frame path and uses regional serving for the 150 ms snapshot budget.

- **Writes to home primary off the critical path:** Events are asynchronous and offers execute from precomputed plans, so cross-ocean write latency does not block visible reactions.

- **EU shard escape hatch:** The aviary shard key supports an EU home shard if counsel requires EU residency.

- **Edge workers stream HTML and send Early Hints:** The purpose is first-bird speed; the module fetch begins before the HTML is complete.

- **API never writes personality or aviary state:** This preserves the service split in which only simulation workers own canonical updates.

- **Sim-workers have no identity-db credentials:** This keeps simulation away from emails and reinforces the privacy boundary.

- **Telemetry account has no database reachability:** Metrics are pushed out and nothing pulls in, making per-account or bird-state access unavailable to telemetry.

- **TypeScript everywhere:** The engine, voice, and prose code can be shared between server and client, and same-language determinism is testable.

- **Fixed-point Q16.16 and Philox PRNG:** The plan avoids JS-engine differences in `Math.exp` and `Math.sin` and makes random draws addressable and replayable.

- **PostgreSQL for identity and aviary clusters:** The stated rationale is transactional single-writer semantics, triggers for invariants, and `SKIP LOCKED` leasing.

- **Valkey regional cache:** The plan uses it for snapshots, auth cache, and rate limits.

- **pg-boss job queue:** It avoids new infrastructure and supports transactional enqueue.

- **Preact plus framework-free scene:** Preact is tiny, and keeping the scene framework-free preserves the critical rendering path.

- **Draw-list renderer boundary:** It allows a WebGL2 backend to replace Canvas 2D without touching scene code.

- **Transactional email subdomain with SPF/DKIM/DMARC:** Magic-link deliverability is on the sign-in critical path.

- **Render-pipeline boundary of state in, pixels and sound out, events out:** The same renderer can support visitors with a narrower snapshot and no event writer.

### Data model

- **UUIDv4 account, aviary, and bird IDs:** They are random and leak no creation time.

- **UUIDv7 event IDs:** They provide ordering and idempotency for append-only ingest.

- **Branded TypeScript IDs:** The plan says an email or name cannot be passed where an ID is expected.

- **Integer micro-units and no persisted floats for engine state:** This supports deterministic fixed-point simulation.

- **Per-account DEKs, KMS wrapping, and blind indexes:** The plan encrypts email, bird names, and visitor emails while still allowing lookup by normalized email.

- **Sim-workers never hold DEKs:** Simulation cannot decrypt identity or bird names.

- **UTC timestamps plus stored local dates:** Local daily buckets remain explicit where they matter.

- **`aviary_state.state_blob`:** Fast-changing state sits in one row so a tick is one state read and write plus personality rows.

- **Bird names encrypted in `birds`:** Names are treated as account data under the account DEK.

- **No app DELETE grant on birds:** This supports stable identity except hard account deletion.

- **Bird-personality trigger rejecting decreases and ceiling changes:** This enforces monotonic drift and immutable ceilings at the database layer.

- **`bird_trait_daily`:** It gives the integrity audit and disaster-recovery second line.

- **`bird_daily_behavior`:** It stores long-horizon bird facts for the notebook, not user-behavior facts.

- **Append-only `interaction_events` with short retention:** Events feed simulation and then partitions are dropped after consumption and age, limiting retained interaction data.

- **Notebook entries as template/facts/seed:** This keeps wording deterministic and lets current names render at read time.

- **Versioned species catalog with pinned bird species versions:** Existing birds keep their look and voice unless an explicit reviewed migration passes listening and visual review.

- **Retention and deletion matrix:** Different data classes age out or crypto-shred according to privacy and hard-delete requirements.

- **Deletion ledger re-applied after restore:** This prevents deleted UUIDs from reappearing after PITR restore.

### API surface

- **Host and visitor cookies separated:** Host cookies are ignored on visitor routes and visitor cookies only work on visit routes, which keeps visit access narrow.

- **CSRF protection with JSON content type, custom header, Origin check, and SameSite:** The custom header forces preflight and the checks restrict state-changing requests.

- **Protocol version support for N and N-1:** Deploys can tolerate stale clients, and reload waits for visibility change so it never happens mid-view.

- **Matter-of-fact error registry:** Error codes map to system-register copy so failures are direct and actionable rather than naturalist or emotional.

- **Random support `ref`:** Support can trace a report in logs without deriving the ref from any identifier.

- **Uniform auth-link response:** Returning `202 {}` always prevents account enumeration.

- **Atomic auth-link consume:** A link can be used once and is invalidated immediately.

- **First sign-in creates account, aviary, starter birds, and schedule in one saga:** This makes the initial relationship consistent and retryable.

- **Session list with buckets but no IP or location:** The plan gives device awareness without storing or showing precise location.

- **Email change keeps old address active until verification:** The change is safe against losing sign-in access before the new address is confirmed.

- **Snapshot cache, replica, and projection serving algorithm:** It gives current-looking snapshots even when the committed simulation is behind, while preserving engine-version safety.

- **Visitor snapshot strips host-only fields:** Visitors see the same ambient aviary but cannot get offer plans, greeting dispositions, or host presence state.

- **Snapshot version monotonicity:** Clients never apply an older snapshot, preventing a bird from moving back.

- **`GET /api/notebook` text rendered at read time:** Current names appear after rename while stored observations and variation remain stable.

- **Event rejection never surfaces to the user:** The plan records aggregate rejection reasons without type labels and keeps the experience quiet.

- **Offer plans resolved client-side but validated server-side:** The reaction starts on the gesture frame and works through brief network loss, while the server re-derives outcomes so clients cannot claim different credit.

- **Plans expire after three minutes:** Stale plans are disabled and require a fresh connection before offers continue.

- **Visit flow with one-time redemption and browser-bound pass:** The secret token leaves the URL path after redemption and access remains revocable.

- **No visitor heartbeats or visitor events:** Visitors update only the visit log and never alter the aviary.

- **Visit duration buckets:** The plan reports approximate durations without fine-grained telemetry.

- **Visit notification debounce:** At most one email per visitor per 24 h keeps notifications matter-of-fact and bounded.

- **Rate-limit keys as blind indexes or UUIDs:** Raw emails are never used in rate-limit keys.

### Simulation engine

- **Pure `step()` function:** No I/O and no clock reads make simulation deterministic and testable.

- **Counter-based PRNG keyed by `rng_key`:** Every draw is addressable and replayable across server and browser.

- **Versioned forward-only engine config:** Personality is stored and canonical; production code has no replay-to-rebuild path.

- **Warm/cold scheduler with exact equivalence:** Cold batching cuts load while preserving the claim that the aviary continues without the viewer.

- **Read-time projection:** It keeps cold aviaries current to readers without synchronous writes.

- **Projection mismatch metric:** Production samples must show zero mismatch because projection and committed tick should be the same step function.

- **Load shedding by stretching cold cadence:** Unobserved aviaries lose nothing because projection covers reads.

- **Presence coverage union:** Two devices attended at once earn one minute per minute, not two.

- **Fifteen-minute stale heartbeat rejection:** Brief offline gaps can count but day-old backfills cannot inflate presence.

- **Daily saturation curve:** Long sessions and synthetic activity are bounded so drift remains slow and no single session visibly changes traits.

- **Per-bird trait ceilings:** Long-loved birds do not converge into the same maxed-out character.

- **Leaky pressure integrator:** Session effects are smeared over about 1.5 days, making in-session change imperceptible while drift can continue gently after leaving.

- **Perception study and JND tuning:** The plan does not guess visible thresholds; it measures look-back discrimination on staging accounts.

- **Attunement rising fast and decaying slowly:** One unhurried session restores most of the gap after absence, while two weeks away lowers viewer orientation without changing mood or traits.

- **Mood as continuous affect plus categorical label:** Continuous affect lets idle motion blend smoothly with no snap; labels drive narration, captions, grammar, and detectors.

- **Circadian nightly roosting instead of tab-open reset:** Birds persist mood through sessions and move naturally through daily rhythms.

- **Coarse activity hint:** The client can bias visible behavior so notebook statements match what a viewer would have seen.

- **Eleven perch slots for seven birds:** The plan says this leaves room for personal space.

- **Canonical user-caused perch moves:** Other devices and visitors converge on offer and long-absence greeting approaches.

- **Deterministic weather process:** Pre-drawn event times keep weather identical for projections, visitors, and devices.

- **No storms, thunder, or snow:** The plan attributes this to `aviary_layout`.

- **Voice identity fixed at adoption:** Recognizability must survive mood and drift.

- **Voice-distance redraw at adoption:** Same-species or similar birds still need to be distinguishable in a chorus.

- **Calling parameters quantized and mixed:** Vocal-frequency-driven outputs avoid exposing raw traits.

- **Offer reaction planning from snapshot:** The plan gets instant local response and validated server credit without trusting the client.

- **Greeting input compilation:** The bolder, warmer bird usually greets first, but wary or roosting birds can greet later or subtly.

- **Field-notebook detectors from aviary facts only:** User presence is not in the type, so entries cannot frame behavior as a visit log.

- **Notebook sparsity budget:** Entries stay sparse, noteworthy, and independent of how often the user visits.

- **Notebook entries during absence without "while you were away":** The aviary continues, but the prose avoids turning absence into a notification surface.

- **Newcomer visiting window:** The wild bird appears gently at the far edge, has no bird ID or personality yet, and does not pressure the user.

- **Newcomer ignored behavior:** No reminders and no expiry message preserve the "no pressure" intent.

- **Engine verification suite:** Property tests, absence invariance, golden hashes, persona harnesses, fuzzing, and designer review all exist because the invariants are product-critical.

### Sync model

- **Single writer, exactly-once event consumption, no lost drift, monotonic versions, projection consistency, convergence:** The plan frames multi-device sync as an architecture property rather than a separate feature.

- **Pull strategy instead of SSE/WebSocket in v1:** Canonical state changes only about once per minute, simultaneous attended devices are rare, and SSE would add connection state and fan-out for little perceptible benefit.

- **Clock-offset estimation from server time:** Device wall-clock errors never affect animation timing.

- **No pulls while hidden:** There is nothing to see, and hidden tabs should not waste battery or count presence.

- **Never teleport a visible bird:** Reconciliation synthesizes a flight or hop so state catch-up reads as natural motion.

- **Weather ramp and stale-rain taper:** Weather never cuts on or off abruptly.

- **Stale state on return reconciled with greeting:** Movement reads as the aviary noticing the user rather than catching up.

- **IndexedDB-mirrored event queue:** A crash or brief offline period does not immediately lose recent events.

- **Dropping oldest presence first when queue is full:** Presence loss is least visible at drift scale and cannot safely be backfilled indefinitely.

- **Clearing event queue on auth failure:** Presence cannot be attributed safely after the session is expired or revoked.

- **Multi-tab audio leader election:** Only one tab plays audio to avoid doubled, phasey choruses.

- **Failure modes mostly invisible:** Snapshot failures keep rendering, tick backlog is hidden by projection, stale replicas are ignored, and only clear account/session/visit failures get system copy.

### Frontend rendering pipeline

- **Feature probe before boot:** Unsupported browsers route to a matter-of-fact page; WebAudio is not required because silent-plus-captions exists.

- **HTML, inline snapshot, and first-bird budgets:** The plan keeps them inside the initial congestion window and targets first bird within 500 ms.

- **Quiet field fallback:** When no snapshot/cache is available, the user sees sky, foliage, and faint motion cues rather than text or a spinner.

- **Atmospheric clearing reveal from quiet field:** This is the one sanctioned reveal because the quiet field is the only state without birds.

- **No service worker in v1:** A service-worker shell could serve stale HTML without the fresh inline snapshot.

- **Layout solver excluding top bar band:** Birds never sit under chrome.

- **Narrow viewport scaling and no cropping:** The solver prioritizes keeping every bird inside the viewport.

- **Portrait procedural fill and no letterbox bars:** The scene adapts without turning into a framed media object.

- **Ultra-wide spacing cap:** Birds should not feel scattered across wide screens.

- **Timezone-coordinate lighting table:** The plan gets seasonal day length and hemispheres without asking for location.

- **Parallax not tied to pointer or device tilt:** Motion should be subtle, not showy.

- **Per-bird OffscreenCanvas atlases:** Rasterizing parts once per bird keeps per-frame draw calls within budget.

- **Plumage compiled as OKLCH regions and detail level:** Trait expression changes appearance without exposing the trait on the wire.

- **Atlas cross-blend over 10 seconds:** Visual drift remains invisible at the moment it changes.

- **Idle micro-motion in base, attention, and action layers:** Motion is continuous and alive without canned clips or repeated parameter instances.

- **Randomized jitter on periodic components:** The plan avoids mechanical periodicity.

- **WCAG flash guard:** No element flashes more than three times a second.

- **Return greetings after attended return:** The aviary notices arrival through gaze, calls, steps, and re-orientation rather than a welcome message.

- **Fresh entropy and variation hash for greetings:** Greetings must never be identical twice.

- **Roosting nighttime greeting forms:** Night returns remain quiet and biologically aligned.

- **Settle ramp to evening, quieter audio, and undo window:** The event is only sent after undo elapses; why the undo window exists is NOT RECOVERABLE FROM PLAN.

- **Top bar fade:** NOT RECOVERABLE FROM PLAN

- **Reduced-motion renderer:** It is separately designed so reduced-motion users still get the product; the motion registry prevents animations from shipping without counterparts.

- **Top bar exactly four buttons and no badges:** The plan preserves quiet chrome and the no-announcement rule.

- **Gesture popover offer items and settle:** It centralizes offering/settling while keeping scene chrome out of the canvas.

- **Notebook sheet with aviary alive behind it:** NOT RECOVERABLE FROM PLAN

- **No chrome inside the scene:** No tooltips, bird hover states, or labels preserve the scene as an aviary rather than a UI overlay.

- **Adaptive quality ladder reducing particles before bird motion:** Bird motion quality is protected as more important than atmospheric effects.

- **No allocations in hot frame functions:** This supports 60 fps and no memory growth.

- **AttentionMonitor hidden behavior:** Cancelling rAF, fading audio, suspending, and flushing events saves battery and matches no-presence hidden state.

### Audio pipeline

- **Audio graph built once with persistent nodes:** Constant node count supports allocation-free audio and soak-test stability.

- **Procedural reverb IR:** It gives room feel while remaining non-recorded audio.

- **`latencyHint: playback`:** Larger buffers mean fewer glitches and better battery for modest interactivity needs.

- **Sixteen-voice preallocated synth pool:** The audio thread never allocates, and voice stealing is deterministic by oldest quietest voice.

- **Two-source syrinx-inspired model:** The plan uses it to make calls more bird-like and less toy-like.

- **Comfort-bounded pitch registers:** High species avoid piercing 7-9 kHz content, and the low/nocturnal species stay comfortable on common speakers.

- **Caption descriptors from `CallPlan`:** Captions describe exactly what was actually played.

- **Signature motif in most calls:** Bird identity remains recognizable across variation.

- **No exact repeat guard:** The product should not reveal loops during long listening.

- **Ambient calls deterministic from corrected time:** Two devices or a visitor hear the same calls at the same moment.

- **Damped call-and-response chains:** Exchanges end naturally rather than escalating forever.

- **Song-fragment offer with human-whistle timbre:** The fragment stays deliberately distinct from any bird.

- **Listen-in mix with non-focused floor:** Other birds are narrowed but "never silent", so listen-in does not become channel switching.

- **No hard cuts on focus switching:** Cross-ramps prevent the interaction from reading as "switching channels."

- **Spatial layout by perch depth and pan:** The chorus has a real layout that helps recognizability.

- **Compression and limiter:** Chorus build-up should not clip.

- **Autoplay-suspended handling without prompt:** Visuals carry the first seconds, and sound fades in on first real activation.

- **Mute in accessibility panel and shortcut:** It supports comfort and accessibility while still emitting `audio_state` for the drift rule.

- **Captions aria-hidden:** Screen-reader users get calls through narration and avoid double speech.

- **Dense chorus caption collapse:** At most three captions remain readable; many calls become a single aggregate caption.

- **WebAudio silent mode:** Captions, beak motion, and narration keep the aviary socially alive without any recorded fallback.

- **Listening gates:** MOS, recognizability, identity across moods, non-repetition, and comfort gates protect the affective spine.

### Accessibility surfaces

- **Polite live region present in initial HTML:** Assistive technology registers it before first update, and `assertive` is avoided.

- **Narration candidate scoring and cadence:** The plan balances scene change, attention rotation, and calm timing so narration is naturalist but not chatty.

- **Narration content rules:** It describes observable posture and behavior, never traits, digits, perch indexes, or mood labels, keeping hidden state hidden.

- **Visual narration strip:** It offers the same prose as an opt-in visual surface with contrast protection.

- **Transparent bird proxy buttons:** Canvas birds become keyboard and assistive-technology targets with stable names and 44 px hit areas.

- **Roving keyboard model for birds:** Keyboard users can listen in and move spatially through the aviary.

- **Single-key shortcut toggle:** Shortcuts can be turned off to satisfy WCAG 2.1.4.

- **Visitor accessibility button only:** Visitors keep narration, captions, and reduced motion without host controls.

- **Dynamic scrims for captions and narration strip:** Text maintains WCAG contrast across lighting and weather.

- **Two-tone focus ring around bird hulls:** Focus remains visible against any adjacent canvas colors.

- **Accessibility QA with lived-experience panels:** The plan measures both task success and whether the aviary felt alive.

### Voice and copy system

- **Naturalist register:** Lowercase, present-tense, specific prose with bird verbs keeps aviary surfaces observational and calm.

- **No second person in notebook, narration, or captions:** The plan prevents these surfaces from becoming about the user's behavior.

- **System register:** Identity, errors, settings, visits, and email surfaces are direct and actionable, with "no warmth standing in for information."

- **Copy registry:** Every static string has an owner and register so lints and writer review can enforce voice.

- **Banned gamification and re-engagement vocabulary:** The plan blocks achievements, streaks, "welcome back", "missed you", and similar pressure language.

- **Terminology lint:** The product says bird, call, and listen in rather than pet-like or UI-like terms.

- **Prose grammar recency memory and large sample review:** The plan aims to avoid repetition while preserving deterministic renders.

- **Notebook grammar volume targets:** NOT RECOVERABLE FROM PLAN

- **Adoption sheet copy "two birds have come to the aviary":** The wording introduces birds as arrivals rather than rewards or unlocks.

- **Live bird vignettes in adoption sheet:** The sheet shows the birds alive before naming, consistent with first-frame aliveness.

- **Song fragment naturalist names:** These keep offers in the naturalist register instead of UI labels.

### Accounts, security, and privacy mechanics

- **Magic-link token hashing and single atomic consume:** Tokens work once and are invalidated immediately.

- **Scanner-safe sign-in landing:** GET never consumes, so security scanners cannot burn links.

- **Magic-link email with UA/time and ignore-if-not-requested copy:** The email is transactional, system-register, and security-oriented.

- **First sign-in saga with idempotency:** Account and aviary creation can retry without duplicating state.

- **Starter species deterministic and distinct:** The first two birds avoid the nocturnal species and have different call registers for an awake, recognizable first meeting.

- **Second device before naming shows same sheet:** Naming remains a single consistent account state.

- **Session token rotation and revocation propagation:** Revocation takes effect across regional auth caches in under five seconds.

- **Email change revokes other sessions:** The plan calls this a security precaution.

- **Export encrypted in object storage with seven-day lifecycle:** Export is user-requested, time-limited, and not rendered in-product.

- **Export `about` string:** It explains that raw values are stored simulation state.

- **Complete inventory of emails:** There are no marketing, digest, re-engagement, or aviary-content emails because the product is not a notification surface.

- **Soft delete keeps sessions and ticks aviary:** The user can recover through the "I changed my mind" page while the aviary relationship is not frozen.

- **Hard delete with crypto-shred:** Destroying the DEK makes encrypted fields unreadable even in backups.

- **Visitor data-subject deletion by blind index:** Support can delete visitor email data without raw-email indexing.

- **Nonce CSP, Trusted Types, textContent rendering, and no third-party scripts:** These protect against XSS and supply-chain privacy leakage.

- **Referrer policy no-referrer:** Visit and link URLs do not leak through Referer.

- **Privacy policy statement on per-bird interactions:** Offers, listen-ins, and presence are used only to run the user's own aviary and deleted within about seven days of processing.

### Performance budgets and observability

- **Initial JS target well below 2 MB:** The internal target keeps the first-bird path fast while the PRD hard limit remains enforced.

- **Synthetic first-bird fleet:** The plan measures the actual first-bird budget across devices, regions, and visit states.

- **Frame-rate and memory device-lab sessions:** Long sessions with seven birds, weather, offers, dawn, reduced motion, and settle verify the product does not degrade over time.

- **Telemetry beacon without cookies, IDs, or credentials:** RUM remains aggregate and privacy-bounded.

- **Client-side histogram bucketing:** The client sends buckets rather than raw per-session traces.

- **No per-account engagement, trait, mood, drift, or interaction-type analytics:** The data is deliberately not computed so engagement pressure and leaderboards cannot "just be exposed."

- **No session replay, heatmaps, third-party analytics, or ads:** CSP and code review ban these because they conflict with privacy and the quiet product model.

- **Integrity alerts page on any non-zero trait decrease/reset signal:** The plan treats these as "potential loss of the relationship."

### Test strategy, rollout, ownership, and risks

- **Tests on every invariant:** The plan turns invariants into CI, property tests, DB grants, visual tests, accessibility audits, privacy tests, and surface audits because "we'll be careful" is not enough.

- **M0 spikes:** Canvas, edge first frame, synth, and determinism spikes determine whether the launch plan is feasible.

- **M3 data permanent:** Beta aviaries carry into GA so the product honors stable identity once real users enter.

- **Operational `ops.max_birds`:** The product cap is seven, but the rollout cap rises only after recognizability, performance, tick-cost, and layout gates pass.

- **Silent deferral if bird-cap gate slips:** Eligible aviaries simply do not get a newcomer yet, avoiding pressure or bursts of birds.

- **Forward-only calibration changes:** Vectors are never recomputed or back-filled.

- **Fourteen-day glide paths for behavior-affecting parameter changes:** Birds should not "snap" into a new personality.

- **Calibration rollout as infrastructure, not experiment:** Hash cohorts watch operational metrics only; there is no outcome comparison.

- **Scaling triggers with pre-agreed actions:** Sharding, SSE, WebGL2, EU home shards, or Rust/WASM are tied to concrete capacity and performance signals.

- **Critical path emphasis on audio naturalness, first-bird performance, calibration studies, and reduced-motion pose libraries:** These are named because delays there threaten the affective product more than ordinary feature slippage.

- **Weekly aliveness review:** Watching the aviary on a real device for 15 minutes keeps the team judging the same experiential target.

- **Per-bird ceilings and headroom growth as drift-risk mitigation:** They protect long-term individuality so every bird does not become maximally bold.

- **Strict presence despite under-count risk:** The plan says this is the PRD's intended strictness and must not be relaxed unilaterally.

- **Daily saturation against automation:** There is no competitive incentive, and the curve caps any day's contribution.

- **Attunement guardrails against hidden happiness meter:** It affects only viewer-directed orientation, never mood, color, or bird-to-bird calls, and long-absence greetings are bigger re-orientations.

- **Personality-loss runbook:** The plan freezes deploys and restores only upward because personality loss is catastrophic.

- **Autoplay risk accepted:** Visuals carry first impressions, and no in-scene prompt is added without design review.

- **Principle-erosion mitigations:** No components exist for announcements/gamification, lints block vocabulary, and milestone audits plus decisions-log sign-off make creep expensive.

- **Pressure-for-engagement-metrics mitigation:** The plan says the data does not exist at aggregate level, making reappearance expensive by design.

- **Invite-spam mitigation:** Fixed templates, rate limits, and complaint-driven host suspension keep visit invites from becoming arbitrary messages.

### Open questions and appendices

- **D-01 export vectors remains open:** The default is yes, but counsel and PM may reverse it by config.

- **Background-tab audio after launch:** The default is no in v1; rationale beyond existing battery/timer/presence reasons is NOT RECOVERABLE FROM PLAN.

- **Seasonal scene variation:** Deferred because it is not in the PRD, though timezone coordinates already support it.

- **Additional languages for naturalist prose:** Deferred; the grammar engine is language-parameterized.

- **Per-bird pronoun preference:** Deferred after choosing no generated gendered pronouns.

- **Snapshot example compiled presentation only:** It demonstrates that host snapshots include presentation state and plans but no trait fields.

- **Visitor audience snapshot differences:** Visitor snapshots remove host-only fields so visits remain read-only ambient views.

- **Prose samples and never-generated examples:** They illustrate the naturalist voice and the lint-rejected boundaries: no "Welcome back", no visit streaks, no trait increases, no achievements, no mood labels, and no session timestamps.
