## System-level intent

- **Mechanisms rather than conventions.** The plan says the product's important failures are "mostly invisible to tests" and therefore spends engineering budget making them "mechanically impossible" rather than "caught in review." This shows up in schema grants that make client-written personality unreachable, the monotonicity trigger, closed enums, closed JSON schemas, component lint against Toast/Banner/Snackbar/Notification, voice lint, token lint, metric-dimension allowlists, and import-boundary lint.

- **Feels alive, not robotic.** The plan repeats that the aviary must have been "continuing without the viewer." The mechanisms are server ticks on absent accounts, `motion_epoch`, first-frame pose phases, continuous noise fields "not animation cycles," runtime-composed calls, scheduled call seeds, and the frame-zero and loop-detection tests in the principles table.

- **Notice, never announce.** The plan treats announcement surfaces as antithetical to the product. It says the greeting is "the only session-start surface," errors render inline, there are no toast/banner/modal primitives, the loading state is a "quiet field" rather than a spinner, autoplay blocking gets "no prompt of any kind," arrivals have "no badge, no modal, no notification," and failed plumbing must not surface as system noise.

- **Charm comes from specificity.** The plan names specificity as the charm engine and implements it through one `voice-kit` package, slot-filled naturalist copy from realized aviary facts, a reviewed frame corpus, sparse notebook entries, captions generated from realized phrases, and writer-owned copy review. It rejects "generic-state phrasing" and event-log prose.

- **Restraint over richness.** The plan defends an engine-level cap of 7 birds, one non-scrolling scene, a frozen five-item top bar, no chrome inside the scene, no recorded audio assets, no scene-graph library, no notification infrastructure, no public discovery, and no gamification cluster. The refusal is active: "the architectural absence is what makes the feature hard to add later."

- **Naturalist product voice, matter-of-fact system voice.** The plan separates `voice/naturalist/*` from `voice/system/*`. Naturalist copy is lowercase, specific, and aviary-observing; system copy is direct account/auth/error information. The import-boundary lint exists so auth, account, error, sync, and settings surfaces cannot borrow the naturalist voice.

- **The server owns discrete facts; the client owns continuous ornament.** The client/server split is a product philosophy as much as an architecture. The server owns personality, mood, perch, call scheduling, weather, notebook generation, presence crediting, and time-of-day phase. The client owns frame-rate rendering, interpolation, flight paths, synthesis realization, ambient ornaments, captions, narration, and palette interpolation. Because ornament is never authoritative, reconciliation is "always a transition, never a merge."

- **One canonical aviary, no second writer.** The plan says multi-device sync is "the absence of a problem." One server-owned aviary per account, `aviary_api` without write grants on personality/mood, append-only events, and deterministic ticks all preserve the rule that there is no last-writer conflict over bird state.

- **Identity continuity.** Bird identity is protected by immutable `bird.id`, immutable `bird.seed`, frozen `species_version`, no internal path to replace a live bird, per-bird audio signatures for life, no session-start mood reset, monotonic drift, and stored notebook text that is "never regenerated." The plan frames resets and silent rewrites as quieter versions of the same identity failure.

- **Privacy by architectural boundary.** The plan treats privacy as a network, grant, schema, and telemetry shape. It forbids cross-account aggregation of bird state, keeps the simulation database out of analytics, gives the analytics role no grant on simulation tables, declares aggregate-only RUM, keeps email in exactly two ciphertext columns, and restricts calibration to synthetic cohorts plus consented staff accounts.

- **No user-behavior product surface.** The plan repeatedly draws the line between observations of the aviary and observations of the user. The notebook taxonomy has no user-behavior member, visit frequency is never computed for product surfaces, there are no visit counts or day counters, and no metric whose shape is "how often does this person come back."

- **Accessibility is the actual product.** The plan says accessible users get "the actual product, not a stripped variant." Narration, captions, reduced motion, keyboard navigation, focus rings, and visitor accessibility all use the same snapshot, renderer, and voice system; reduced motion is a `motionProfile` parameter, not a separate path.

- **Performance is part of the spell.** The plan connects load and runtime budgets to the product feeling alive: inline bootstrap snapshot avoids the gap where "a spinner would be invented," first bird appears under 500ms, audio and renderer steady states allocate nothing, and soaks run on real hardware for 30 minutes because sustained frame rate and memory are part of the experience.

- **Determinism makes continuity, deletion recovery, calibration, and testing possible.** The tick is a "pure, deterministic function" with counter-based randomness. That enables replayability, exact eager-vs-catch-up equality, bit-identical recovery after paused deletion, simulated years in seconds, and property tests that catch the failures conventional suites miss.

- **The three-week claim is a real-time product constraint.** The plan stresses that accelerated harnesses prove math but not "felt experience." Drift, mood, and expression mappings freeze before long-soak staff accounts begin because "visible drift after about three weeks" must be validated in wall-clock time.

## Per-feature whys

### How to read this document

- **Concrete starting numbers and calibration harnesses.** The plan picks numbers where the source names ranges because engineering needs buildable decisions, but it names the harness that will calibrate them.

- **Judgment-call register.** The plan collects ambiguity resolutions so reviewers can audit the "interpretation layer" in one pass and product can sign off before milestones.

### Scope

- **Email and magic-link sign-in.** The plan uses magic links with scanner-safe POST consumption because GET links are followed by mail scanners and preview bots, which would make links look already used. `POST /v1/auth/link` always returns 202 to avoid an account-enumeration oracle over "people who use Pocket Aviary."

- **Fifteen-minute, single-use magic links.** The plan grounds them in security: tokens are hashed, single-use, consumed by POST, and rate-limited.

- **Per-device revocable sessions.** The plan gives sessions coarse labels and immediate revocation so account settings can remove a token without cached authorization lingering beyond the request.

- **Verified email change.** NOT RECOVERABLE FROM PLAN

- **On-demand JSON export.** The plan treats export as a data-portability artifact. It includes personality vectors despite the product-surface ban, with guards: never rendered in-product, no import path, session-gated download, 24-hour link, and a note that values are internal and not meaningful to compare.

- **Thirty-day soft delete then hard delete.** The plan makes deletion recoverable for 30 days while honoring the user's instruction immediately: tick pauses, snapshots are evicted, invites revoked, and hard deletion cascades later.

- **One aviary per account.** The plan uses one canonical server-held aviary so multi-device sync becomes "the absence of a problem," with two devices simply reading one row.

- **Two starter birds.** The plan presents adoption as birds that arrived rather than a catalog, preserving the product's feeling of encounter. Starter pairs avoid duplicate species and avoid two low-vocal species so the first session is not too quiet to read.

- **Engine cap of seven birds.** The cap protects restraint and call recognizability. If human recognizability testing fails at 7, the plan says to lower the cap rather than ship a blurred chorus.

- **Age-based arrival of birds three through seven.** Arrivals depend on `aviary.created_at` and nothing else so attention does not "earn stuff." Quiet discovery by looking at the aviary preserves the no-gamification stance.

- **Hidden five-trait personality vector.** The plan hides numeric traits because the product wants users to experience personality through behavior, sound, posture, and plumage, not through values that could become comparison or optimization surfaces.

- **Monotonic-toward-expressive drift.** Traits only move up or hold so there is no neglect penalty, no decay term, and no future "balance" change that turns absence into punishment. The trigger enforces this below the drift function.

- **Six-state mood with continuous-time transitions.** Mood gives the aviary daily, weather, event, neighbor, and personality-sensitive behavior without snapping on session start. The `roosting` name avoids collision with the aviary's `settled` lighting state.

- **Per-bird procedural call grammar with lifelong signature.** The plan wants "you know Pip by ear" to survive drift, so pitch center, timbre fingerprint, rhythm signature, and motif vocabulary are fixed by `bird.seed`, while mood and drift modulate tempo, dynamics, phrase length, ornamentation, and frequency.

- **Mood-shaped idle motion.** Continuous noise, mood targets, and event impulses prevent loops and make a bird's first frame land mid-life rather than at animation phase zero.

- **Bird-to-bird response and mood contagion.** The plan uses answering calls, staggered greetings, and neighbor-influenced mood so the aviary reads as several living birds in a shared place rather than isolated assets.

- **Server-side simulation independent of client connection.** The plan needs the aviary to continue without the viewer, gives the server sole authority over discrete facts, and prevents the client from owning personality or mood.

- **Append-only client event log.** Events are append-only and idempotent so at-least-once retries are safe, duplicate delivery is discarded, and no client can rewrite the historical input consumed by the tick.

- **One horizontal non-scrolling responsive scene.** The plan keeps the aviary as one place, not a browsable app surface, and avoids pointer-driven parallax because that would convert "a window into a toy."

- **Three perch zones.** Perch zones make position a readable signal of mood and boldness; no client command can set a perch because a placement affordance would erase the signal.

- **Local-time day/night cycle.** The aviary shares the user's day, including timezone changes, with smooth interpolation across DST rather than palette jumps.

- **Rare passing weather.** Weather adds short-lived atmospheric and behavioral variation without creating events the user must notice; rain, wind, and aftermath change mood/calls quietly.

- **Ambient leaf and feather drift.** Ambient ornaments give place and motion, but in fixed buffers and removable quality tiers because birds remain the product.

- **Five-item top bar that fades on stillness.** The frozen inventory defends restraint. The fade clears chrome from the scene while staying in the accessibility tree and tab order; focus forces full opacity.

- **No chrome inside the scene.** The plan avoids buttons, badges, tooltips, overlay icons, and labels inside the aviary so the scene remains a watched place. Focus indicators and captions are bounded accessibility exceptions.

- **Return-greeting.** The greeting is the whole welcome and the anchor moment of a session. It varies by absence length, boldness, and mood; it is server-decided because those canonical inputs live there; and it is staggered so the aviary notices one bird at a time rather than announcing the user's arrival.

- **Listen-in.** Slow mix ramps make the interaction feel like "leaning in," not switching channels. Other birds never reach silence because the aviary should remain a place where several things are happening at once.

- **Three offer types.** Seed, pool, and song offers are placed into the scene, not aimed, so each bird can react from its own mood, distance, curiosity, and cooldown.

- **Offer cooldowns.** Cooldowns prevent curiosity input from saturating in one session and keep the drift model from collapsing into a clicker. The plan hides timers and disabled-tooltip states because those would be game surfaces.

- **Settle with 5-second undo.** Settle quiets lighting, calls, and mood without creating drift. The undo window allows reversal, and nothing records whether a session ended with settle so it cannot become attendance meaning.

- **Presence accounting with visible/focused/recent activity.** Presence is strict enough to reject background tabs and hidden windows, but the activity window is long because "watching birds without moving is the product."

- **Touch and wheel activity for presence.** The plan adds touch and pointerdown because otherwise presence on phones is "structurally near-zero"; a tap is treated as at least as strong as a mouse twitch.

- **Presence clamps and no backfill.** Server clamps prevent forged long windows, replayed pings, and two devices double-counting one human. Hidden-tab gaps are not reconstructed because unobserved presence is not credited.

- **Field notebook.** The notebook is a sparse observer's record, not a journal, feed, or log. It is read-only, immutable, infinitely scrollable back, and generated from aviary facts so entries feel specific and trustworthy.

- **Visits.** Visits are per-invite, opt-in, expiring, revocable, read-only, and unable to write interaction events because a visitor watching must not drift the host's birds. The visit log is a transparency surface, not a measurement surface.

- **Visit notifications off by default.** The plan keeps visit notifications out of onboarding and restricts email templates so notification infrastructure does not grow into a general product surface.

- **Prose screen-reader narration.** Narration gives screen-reader users the same specificity as sighted users, using prose from the same `voice-kit` instead of state lists.

- **Reduced motion as a designed render mode.** Reduced motion is a calmer Pocket Aviary, not a broken version. It keeps calls, drift, mood, and notebook unchanged while parameterizing motion inside the same renderer.

- **Runtime-generated call captions.** Captions derive from realized phrases so caption users get the same non-repetition as listening users. They are a bounded in-scene text exception because the alternative would ration core sensory content by hearing.

- **Full keyboard navigation.** Keyboard users can reach the top bar, birds, listen-in, dialogs, settle undo, and focus restoration; focus rings are designed for contrast against every aviary state.

- **WCAG AA on user copy.** The plan computes contrast against brightest midday and darkest night because the background luminance changes by design.

- **Initial JS, first-bird, frame-rate, and memory budgets.** The budgets defend the product's immediacy and sustained aliveness: no spinner gap, no degraded bird fidelity, no GC churn, no memory growth over a 30-minute watch.

### Explicitly out of scope and non-goals

- **Native apps.** The plan says the data model is not shaped for them.

- **Payments and tiers.** NOT RECOVERABLE FROM PLAN

- **Shared, team, or household aviaries.** NOT RECOVERABLE FROM PLAN

- **Multiple aviaries per account.** The plan's sync and identity model depend on one canonical aviary per account.

- **Customizable or purchasable scenes.** The plan's restraint and single-scene design refuse richness that would turn the aviary into a configurable product surface.

- **Public discovery, directories, feeds, profiles, follows, comments, ratings, leaderboards.** The plan refuses cross-account aggregation and any metric or surface that could become comparison, ranking, or social pressure.

- **Push notifications and marketing email.** The plan keeps outbound email behind a closed template allowlist so no scheduled job can announce the aviary.

- **Recorded audio fallback.** The plan refuses recorded audio because runtime synthesis is central to non-repetition, bundle weight, and the rule that no audio assets enter the bundle.

- **Co-presence, chat, avatars, or visitor representation.** The plan keeps visits read-only ambient rendering with no visitor drift contribution and no visitor-specific representation.

- **Achievements, badges, levels, XP, ranks, tiers, scores, streaks, day counters, visit calendars, green dots, counters, milestone celebrations, exportable visit logs.** The plan refuses the gamification cluster "in every disguise" and backs the refusal with closed taxonomies, API contract tests, no aggregation, and no notification surface.

- **No negative drift.** The plan explicitly refuses neglect, decay, and wariness accumulation so ignored birds hold rather than worsen.

- **No cross-account aggregation of bird state.** The plan refuses the pipeline that would compute leaderboards or drift dashboards, making the absence architectural.

- **No notification surface.** The plan refuses push infrastructure, notification services, and broad email templates so announcements cannot be added casually.

### Architecture

- **Five deployable units.** The split follows the boundaries of who may write personality state and what may touch per-account interaction data, rather than feature lines.

- **Edge with inline bootstrap snapshot.** The edge worker inlines a snapshot so the first bird can be drawn without a round trip, avoiding the gap where a spinner would appear.

- **Stateless API with limited grants.** The API handles auth, reads, events, settings, invites, and visits, but cannot write personality vectors; this expresses the "no last-write-wins for personality" rule as infrastructure.

- **Simulation workers as sole writers.** The `sim` service alone writes personality and mood so the tick owns drift and mood transitions.

- **Redis for snapshots, presence buckets, rate limits, and leases.** The plan uses Redis for hot coordination and cache roles that do not need the simulation database to become an analytics store.

- **Mail with closed template allowlist.** The allowlist enforces the no-notification surface by allowing only the few transactional templates named in the plan.

- **Ops telemetry in a separate aggregate-only store.** The plan separates ops telemetry from simulation data so observability cannot see or aggregate per-bird state.

- **Pure renderer boundary.** The renderer takes `(snapshot, localTime, motionProfile, audioState)` and no network/storage access, making reduced motion a parameter, visitor rendering the same code, and headless renderer tests practical.

- **Preact and signals.** The plan chooses a small client framework because the interactive surface is a five-item top bar and a few panels; React's weight "buys nothing" against the critical-path budget.

- **WebGL2 primary renderer with Canvas2D fallback and decision gate.** The plan expects GPU to handle grading, weather, parallax, and saturation more cheaply, but measures Canvas2D on reference hardware and will take it if it sustains the full load.

- **WebAudio with AudioWorklet.** Worklets give allocation-free steady state and the f0 curves needed for the call grammar.

- **TypeScript API and Rust simulation.** The API shares `voice-kit` and schema types; the simulation is a numeric loop where Rust avoids GC tail latency.

- **Postgres and row-level grants.** The plan chooses Postgres for grants, transactional tick batches, and ordinary scale needs rather than exotic storage.

- **Single region plus CDN.** The tick is 60 seconds and does not need regional replicas; the latency budget is met by edge-delivered HTML and bootstrap snapshot.

### Data model

- **Synthetic UUIDv7 identifiers.** Time-ordered synthetic ids index well and carry no PII.

- **Email ciphertext plus HMAC blind indexes.** The plan confines email to two encrypted columns while still allowing lookup.

- **Closed versioned account settings.** Settings are not a bag so adding a key requires schema friction against "just add a toggle."

- **Immutable bird id and seed.** Identity continuity requires no path, including internal tooling, that can replace one bird with another.

- **Frozen species version.** Pool updates never re-skin live birds, preserving identity continuity.

- **Database bird cap.** The cap is enforced in the database, not only application code, so restraint survives future code paths.

- **Personality monotonicity trigger.** The trigger makes negative drift raise in production and tests, absorbing float noise without admitting decay.

- **`bird_mood` without reset column or default-on-connect path.** Mood can only be written by the tick, preventing session-start mood resets.

- **Interaction event idempotency key and server sequence.** Client-generated ids make retries safe, while server sequence gives the tick canonical order.

- **Presence minute aggregate.** The plan aggregates pings because the tick needs sums, not every 15-second row.

- **Notebook realized text.** Storing text preserves what the user read; regenerating would let corpus revisions rewrite history.

- **Visit duration rounded to the minute.** Minute precision serves transparency without turning the visit log into measurement.

- **Versioned species pool in shared code.** Loading species definitions in both `sim` and client from a shared package prevents disagreement about what a species is.

### API surface

- **REST, JSON, cookie-bearer sessions, and closed schemas.** Closed schemas make the no-trait-values contract test possible.

- **Snapshot as entire aviary read surface.** A compact snapshot keeps reads simple, cacheable, and closed against accidental trait exposure.

- **Expression vectors instead of trait values.** Quantized, non-injective render/voice/posture values let the client render personality while keeping numbers unavailable even in devtools.

- **Calls scheduled ahead.** Publishing the next call window lets calls fire on time despite network jitter, keeps devices coherent, and avoids request-time audio.

- **Server-decided greeting.** `greeting` is present only when owed so the client does not infer welcome behavior from a flag.

- **Batched idempotent event ingest.** Batching reduces overhead; idempotency supports offline and retry behavior.

- **Silent protocol-level rejections.** Failed presence plumbing must not announce itself in a quiet product.

- **Read-only notebook API.** No edit endpoint exists because editing would turn an observer's record into a curated journal.

- **Bird rename only.** Naming writes `bird.name` and nothing else; no profanity filter or uniqueness requirement because naming the first bird "should not be an argument."

- **Arrival accept and decline endpoints.** The plan allows free, non-final decline and later re-offer without badges or modal pressure.

- **Session-gated export download.** The inbox alone is not the intended security boundary for a file containing full interaction history.

- **Visitor route tree and DB role.** A separate SELECT-only path means a visitor cannot create interaction events or influence drift.

- **Greeting stripped from visitor snapshots.** A greeting is for the host, so visitor rendering omits it while keeping captions and narration.

- **Matter-of-fact visitor revocation.** Pull-time `410` is enough because visits poll every 25 seconds and no push channel is needed.

- **System-voice API errors.** Account, auth, error, sync, and settings copy stay matter-of-fact and cannot import naturalist phrasing.

- **Transport errors during a session are not surfaced.** The client keeps rendering from the current snapshot and queues events because the aviary continues server-side and an offline indicator would announce plumbing.

### Simulation engine

- **Sixty-second canonical tick.** The tick cadence matches "roughly once per minute" while staying cheap and deterministic.

- **Counter-based randomness.** Removing `rand()`, wall-clock entropy, and ambient state gives replayability, catch-up equality, and fast test harnesses.

- **Transactional tick batches with compare-and-set.** The plan prevents double-processing because double-applied drift would be a silent data-quality failure.

- **Hot, warm, and cold account tiering.** Tiering saves cost while replaying the same skipped tick indices, not an approximation.

- **Snapshot catch-up before response.** Returning users see an aviary that has genuinely advanced during absence.

- **Four-minute presence activity window.** The window is long because quiet watching should count; beta calibration later locks it between 3 and 6 minutes.

- **Drift formula with asymptotic `(1 - v_t)`.** Diminishing returns keep long-tenured aviaries from saturating into identical maximally expressive birds.

- **Trait-specific drift sources.** Presence moves all traits, listen-in moves social warmth and vocal frequency, offers move curiosity or boldness, and settle moves none, preserving each interaction's meaning.

- **Perceptual mappings from hidden traits.** Drift matters only if it changes visible and audible surfaces, so the plan maps traits to perch weights, greetings, calls, plumage, head tilts, posture, and responsiveness.

- **Visual quantization with hysteresis.** Hysteresis prevents shimmer while making three-week changes visible in retrospect.

- **Continuous-time mood Markov chain.** Intensities combine time, weather, recent events, neighbors, and personality so the same input can read differently for different birds.

- **Dawn mood re-draw.** Daily cadence is anchored to the aviary clock, not tab open, avoiding session-start snaps.

- **Perch softmax with stickiness.** Stickiness keeps the scene calm; boldness creates a long-term resting distribution while mood dominates short timescales.

- **Greeting selection by absence, boldness, warmth, and mood.** The plan wants the greeting to be noticed as a bird behavior, not a replayed welcome animation.

- **Parameterized greeting behaviors.** No greeting clips exist, so there is nothing to rotate identically.

- **Offers placed into the scene.** Offers are environmental facts each bird evaluates independently, not direct commands to a bird.

- **Settle duration and re-engagement.** The aviary remains settled until the user actively comes back, with closing the tab equally valid.

- **Weather process.** Low-frequency rain and wind make the aviary variable without thunder, snow, or events the user must notice.

- **Soft-deleted accounts stop ticking.** The plan honors deletion immediately while deterministic catch-up preserves continuity on recovery.

- **Hard deletion cascade and orphan audit.** The plan verifies no data remains referenced after deletion.

- **Quiet bird arrival.** New birds appear in the scene and are accepted on focus because discovery should happen by looking at the aviary.

### Sync model

- **HTTP polling instead of sockets.** With a 60-second tick, sockets add connection state and scaling cost for one meaningful message per minute.

- **IndexedDB outbox.** The outbox lets clients write at-least-once while offline or flaky without losing user-initiated events.

- **Outbox bounds and stale presence eviction.** Presence pings older than two minutes are never replayed because stale attention would inflate drift.

- **Presentation-state reconciliation.** Every snapshot difference becomes a natural transition: flights, cross-blends, ramps, merged calls, weather fades, and greeting offsets.

- **Server-time offset smoothing.** Calls and greetings schedule against server time so skewed client clocks do not desynchronize the aviary.

- **Timezone refresh on session open.** The aviary follows the user's local day after travel while keeping interpolation smooth.

### Frontend rendering pipeline

- **Layered scene.** Sky, foliage, birds/perches, ornaments, and foreground create depth while preserving a single non-scrolling place.

- **No static fallback.** A static aviary is "not this product," so unsupported rendering gets graceful handling rather than a fake aviary.

- **Bird rigs from atlas quads.** Small rigs and posed parts support expressive animation without a scene-graph library or general-purpose engine.

- **Noise fields instead of cycles.** No cycle length means there is nothing to notice looping.

- **Event impulses in pose.** Calls, nearby calls, and landings affect the same pose system, making reactions part of one body language.

- **Flight between perches.** Birds fly rather than slide or fade so perch changes remain embodied signals; reduced motion is the named exception.

- **Inline bootstrap snapshot and quiet field.** First-frame delivery avoids a spinner, and the quiet field covers slow snapshot fetches without reading as machinery.

- **Motion epoch.** Anchoring micro-motion makes cold loads and two devices look like the aviary was already in motion.

- **Hidden-tab lifecycle.** Hidden tabs stop rendering, audio, and presence, while the server simulation continues where it belongs.

- **Adaptive quality that never degrades birds.** Ornament density, blur, and parallax can step down, but bird fidelity is protected because birds are the product.

- **Top bar accessibility fade rules.** Opacity-only fade keeps controls reachable and avoids trading keyboard/screen-reader access for aesthetics.

- **Adoption fly-in only for genuinely new birds.** The entry animation belongs to adoption and later arrivals, never session start, so it cannot masquerade as continuity.

- **Responsive no-cropping invariant.** Every bird remains visible and recognizable at every viewport because cropped or offscreen birds would break the aviary as a watched scene.

### Audio pipeline

- **Runtime synthesis and zero audio assets.** Synthesis prevents looped recordings, phase-cancellation artifacts, bundle weight, and recorded fallback drift.

- **Species motif library and phrase grammar.** Motifs and stochastic productions give each species a call character while allowing mood-scaled variation.

- **Per-bird signature.** Fixed audio identity makes individual birds recognizable by ear across moods and drift.

- **Pooled AudioWorklet voices.** Pre-allocation directly answers the 30-minute no-memory-growth rule.

- **Feedback-delay reverb.** The plan avoids convolution impulse responses because they would be recorded audio assets with weight and rule problems.

- **Lookahead scheduler.** AudioContext-based scheduling keeps note onsets precise instead of relying on `setTimeout`.

- **Chorus onset jitter and compressor.** Birds answer rather than fire in unison, and a multi-bird chorus stays bounded in loudness.

- **Ambient bed.** Quiet, varying background sound makes silence between calls "a place rather than a gap."

- **Listen-in gain floor.** The other birds never go silent because silence would turn the aviary into soloable tracks.

- **Procedural song offer.** Song fragments use the same sound world rather than sounding like samples dropped into it.

- **Degradation ladder to graceful silence with captions.** The plan prefers silence with captions over canned audio and keeps the rest of the aviary unchanged.

- **Autoplay silence with no prompt.** Browser policy is accepted as a first-session silent visual aviary until a gesture; prompts would announce the system at the wrong moment.

- **Audio joining mid-phrase.** When audio resumes, joining an in-flight call makes the sound read as already there rather than switched on.

### Voice system

- **Single `voice-kit`.** Sharing notebook, narration, and captions means users hear one product rather than separate surfaces written in similar style.

- **Hand-written frame corpus with slot filling.** The plan rejects a hosted language model for privacy, voice stability, and determinism; self-hosting would add cost and eval burden for roughly two sentences per week.

- **Sparse notebook generation.** About two entries per week and a 36-hour floor keep entries meaningful; an entry per session would dilute the notebook into noise.

- **Anti-repetition rules.** Frame and `(frame, bird)` reuse windows protect the notebook from sounding mechanical.

- **Closed notebook observation taxonomy.** The taxonomy can observe greeting order, perch, mood read, weather, chorus, bird-to-bird, offers, plumage, quiet stretches, arrival, and time of day, but not the user.

- **`quiet_stretch` observations.** The plan lets unwatched aviaries be observed truthfully without reproach or absence language.

- **Voice lint.** Lint catches second person, exclamation marks, gamification terms, generic phrasing, numeric trait references, and namespace drift before users see them.

- **Narration cadence bounds.** Slow, jittered prose and an 8-second minimum prevent the live region from outrunning the reader.

- **Captions from realized phrase shape.** Caption variability matches call variability, avoiding a text version of the looped-audio problem.

### Accessibility surfaces

- **Two polite live regions.** Priority prose can jump the idle queue without interrupting the reader mid-sentence; assertive regions would be rude for an ambient product.

- **Bird descriptions as prose.** Screen-reader users get naturalist specificity rather than "perch 2, mood content."

- **Mood never exposed as a label.** Mood is described through behavior, not named as state text or tooltip-like metadata.

- **`aria-pressed` for listen-in.** Listen-in is a genuine toggle, so the accessibility tree needs to expose whether it is active.

- **Roving keyboard focus over birds.** The keyboard map lets users navigate birds by scene position and operate listen-in without extra chrome.

- **Dual-stroke focus ring.** A dark inner and light outer stroke ensure contrast across midday sky and full night.

- **Reduced-motion parameter.** A shared renderer path keeps reduced motion from rotting and makes it testable alongside full motion.

- **Caption scrim and stacking.** Captions must remain readable against every aviary state and avoid overlapping bird silhouettes.

- **Visitor accessibility.** Captions, narration, keyboard navigation, and reduced motion apply to visitors because accessibility does not stop at the account boundary.

### Performance and observability

- **Per-chunk size limits.** The plan sets a working target below the PRD ceiling so the ceiling is not negotiated away later.

- **Real-device first-bird timing.** The load budget is tested on a mid-tier Android over throttled 4G because the first-bird promise is a lived mobile experience.

- **Nightly frame and memory soaks.** Sustained 30-minute performance is verified on real hardware because short PR tests and containers miss the relevant failures.

- **Audio, notebook, renderer, and outbox memory constraints.** These are the places the no-memory-growth rule bites, so each uses pooling, virtualization, fixed buffers, pre-allocated arrays, or bounded queues.

- **Synthetic monitoring.** Synthetic browsers measure load, frame timing, and audio init without privacy load.

- **Aggregate-only RUM.** RUM is limited to timings, histograms, error counts, versions, and browser/OS distributions so telemetry supports operations without user or bird dimensions.

- **Metric-definition allowlist.** Metrics with undeclared dimensions fail the build, and the telemetry client cannot accept account ids.

- **Alarms and invariant jobs.** Tick latency, backlog, snapshot latency, ingest errors, mail failures, audio-context deviations, and daily monotonicity checks catch operational failures without tracking individual users.

### Security and privacy engineering

- **Hashed session and token storage.** The plan stores hashes, not bearer tokens, so database compromise does not expose usable tokens.

- **CSRF protection.** SameSite plus required custom headers protect mutating requests.

- **PII containment.** Email is envelope-encrypted in two places only; logs redact email-like strings; schema grep prevents accidental email columns.

- **No simulation-to-analytics path.** There is no replication, CDC, export, or ETL path for per-bird state, making aggregation structurally unavailable.

- **Calibration accounts.** Consented staff accounts flagged `is_calibration` are the only real accounts whose per-bird state can be read for drift analysis.

- **Export and deletion handling.** Export is asynchronous and session-gated; deletion pauses ticks, evicts snapshots, revokes invites, and later purges storage and mail logs.

- **Visitor isolation.** Separate route tree, serializer, and SELECT-only DB role prevent visitors from writing events or influencing drift.

- **Invite abuse limits.** Outstanding and weekly invite caps plus host identification limit using invites to email strangers.

- **Presence forgery bounds.** Clamps bound forged presence, and without scores there is little to inflate.

### Testing and calibration

- **Test strategy around invisible failures.** The plan organizes tests around drift constants, event-log prose, greeting repetition, and vector resets because those failures often do not throw.

- **Unit/property/contract/integration/render/audio/accessibility/performance/voice layers.** Each layer targets a class of product failure: determinism, monotonicity, schema closure, visitor isolation, non-cropping, recognizability, live-region flooding, soaks, and voice erosion.

- **`sim-lab`.** The harness drives the real tick over synthetic cohorts so calibration tests the shipping math, not a reimplementation.

- **Presence-window calibration.** Aggregate inter-activity histograms lock the activity window before GA because changing it later changes everyone's drift rate.

- **Physical device lab.** Real laptops, phones, and browsers cover WebGL, sustained frame rate, first-bird timing, and Safari audio behavior.

- **Human call recognizability evaluation.** If listeners cannot identify birds by call at seven birds, the cap comes down because recognizability is the purpose of the cap.

- **Weekly aliveness review.** Watching a live aviary is the only reliable detector for canned, mechanical, or repeated behavior.

- **Weekly voice review.** The writer reviews notebook, narration, and captions because voice erosion is gradual and invisible to code authors.

### Rollout and milestones

- **Engine freeze at M4.** Drift, mood, and expression mappings freeze early so real accounts can drift for weeks on the shipping engine.

- **Overlapping milestones.** Engine, audio, and client tracks run in parallel after foundations so the long-lead risks get enough calendar time.

- **Writer and accessibility specialist from week one.** The plan says both fail when retrofitted and are cheap to build in but expensive to bolt on.

- **Invitation-wave GA ramp.** Capacity is not the main issue; the ramp gives aliveness and voice reviews time across more timezones, locales, and low-end hardware.

- **Time-shifted arrival validation.** No launch user reaches day 90 during ramp, so arrival must be tested before GA rather than discovered live months later.

- **Day-one instrumentation.** Operational metrics, alarms, synthetic aviaries, screenshot diffs, and monotonicity jobs are live from the first GA account without user-dimensioned telemetry.

- **Post-launch string rule.** Every user-facing string goes through the writer and voice lint so product voice does not erode.

- **Post-launch temperament rule.** Changes to `α`, trait weights, mood intensities, or expression mappings regenerate calibration charts and require simulation-owner review because they are "the product's temperament."

### Risks

- **Drift calibration risk.** Too fast becomes a Tamagotchi; too slow becomes a screensaver. The plan mitigates with `sim-lab`, long-soak accounts, and perceptual-step mappings.

- **Personality vector loss or reset risk.** The plan treats this as the worst failure, mitigated by server-only writes, additive deltas, append-only events, monotonicity, invariant jobs, PITR, and rehearsed restore.

- **Audio uncanniness risk.** Procedural audio has no recorded fallback, so audio starts early, gets human review, and uses recognizability harnesses.

- **Accessibility regression risk.** Shared renderer parameters, shared `voice-kit`, PR gates, screenshot tests, and specialist involvement through GA reduce rot.

- **Autoplay first-session risk.** The plan accepts some silent first sessions rather than showing a modal that would break the more important quietness.

- **Notebook event-log risk.** Voice lint, taxonomy closure, writer review, and an M5 corpus deadline protect the most concentrated voice surface.

- **Scanner-consumed links risk.** POST-only consumption and corporate-mail-filter tests prevent magic and invite links from appearing flaky.

- **Sustained 60fps risk.** Allocation-free loops, fixed buffers, adaptive quality, and nightly soaks address long-session performance.

- **Timezone and DST risk.** Absolute-time interpolation against sunrise/sunset and transition tests keep roosting and palette phases local and smooth.

- **Engagement-surface risk.** Component lint, closed schemas, closed taxonomy, no notifications, and no aggregation stop "harmless" engagement ideas before review.

- **Simulation cost risk.** Tiering, change-only writes, and batched transactions keep per-minute ticking ordinary at scale.

- **Bundle creep risk.** Per-chunk budgets and aggressive splitting preserve headroom under the PRD ceiling.

### Appendices

- **Coherent temperate garden/woodland species pool.** The species set gives visual and call variety while staying in one world.

- **Species seed biases.** Biases shape adoption starting points but do not constrain drift, so timid species can become bold individuals.

- **Nightjar active at night.** The nightjar keeps night from becoming a dead state while most birds are `roosting`.

- **Mood reference.** The table translates moods into idle read, perch bias, call rate, entry sources, and notes so implementation and QA share a behavioral vocabulary.

- **Naturalist copy samples.** Samples demonstrate sparse, lower-case, aviary-subject prose and in-voice arrival acceptance.

- **System copy set.** The complete matter-of-fact copy set prevents ad hoc error/auth language from drifting into warmth or naturalist phrasing.

- **Constant reference.** Tunables live in one place with owner and calibration source so product temperament is reviewable rather than hidden in code.
