## System-level intent

- **One canonical relationship, not a platform.** The plan opens by naming Pocket Aviary v1 as a "web-only, single-user relationship with one canonical aviary" and repeats this through "one canonical aviary per account," the exclusion of "multi-aviary accounts," and the rule that visitor attention must not affect host presence, mood, or drift.
- **A glanceable scene, not a game shell.** The product surface is "one horizontal, glanceable scene with a sparse top bar, not a game shell." This shows up again in the bans on "scores, levels, XP, achievements, badges, streaks," the renderer rule that "the scene contains no inline buttons, badges, tooltips, or labels," and the delivery check for "no scene chrome."
- **Continuity without the viewer.** The central contract is explicit: "the aviary continues without the viewer," mood persists across sessions, ticks advance while no client is connected, and absence is "a behavior input, not a textual 'you were gone' announcement."
- **Recognition through bird behavior rather than system announcement.** The plan says the aviary "notices the viewer through a bird's behavior rather than a system announcement." Return greetings, listen-in ramps, offer reactions, settle, captions, and notebook observations all preserve that behavioral vocabulary.
- **Quiet change over weeks.** The product should "accumulate quiet change over weeks." The plan anchors this in server-authored monotonic drift, sparse notebook observations "roughly every few days," age-based bird additions, and calibration targets of "one week" for measurable instrument drift and "three weeks" for perceptual change.
- **Server authority over canonical life.** The client/server boundary is semantic: the server sends canonical state and owns "timestamps, event sequence/order, authorization, all drift/mood updates, and the canonical snapshot version"; the client derives presentation and "never submits an absolute personality vector, mood, position, or drift result."
- **One writer, deterministic ticks, no last-write-wins personality.** The architecture calls for a "clear single-writer simulation boundary," a simulation worker as "the only writer," per-aviary leases, deterministic ticks, append-only events, and the explicit rule that "Personality is never resolved by last-write-wins."
- **Calibration instead of hard-coded product promises.** Where values remain open, the plan requires a "configuration/feature flag with an explicit calibration owner." It names the inactivity window, tick cadence, mood constants, age schedule, offer cooldown, audio ramps, and weather frequency, while still requiring deterministic and testable defaults.
- **Privacy by minimization and separation.** Email is encrypted and "never an identifier"; telemetry is "aggregate-only"; operational metrics have "no per-bird/per-account dimensions"; exports are data portability artifacts; and hard deletion must purge account-tied state, including event history, vectors, notebook, sessions, invitations, logs, exports, and telemetry references.
- **Naturalist voice for the aviary, matter-of-fact voice for systems.** The plan separates "direct, matter-of-fact copy" for system errors from "naturalist and lowercase" observations. Notebook and narration use "lowercase, present-tense, specific naturalist prose"; auth, sync, settings, visit, and error surfaces are explicitly checked for the allowed voice.
- **Accessibility as a coequal surface.** The plan says to "treat accessibility as a parallel product surface that reads the same canonical state, not as labels attached after the visual build." Screen reader narration, captions, keyboard movement, focus treatment, contrast, reduced motion, and audio-off behavior are all primary paths.
- **Visits are explicit, read-only, and socially quiet.** Visits are "per-invite, revocable, read-only," "disabled by default," with a "silent visit log" and optional notification off by default. The rationale is carried by repeated rules that visitors cannot submit presence or interaction events and that visitor viewing must not become host presence or social pressure.

## Per-feature whys

### Product boundary and implementation posture

- **Web-only, single-user relationship with one canonical aviary** - The plan uses this as the product boundary: a "single-user relationship" centered on "one canonical aviary," with native clients, shared/co-present aviaries, and multi-aviary accounts excluded.
- **One horizontal, glanceable scene with a sparse top bar** - The why is to keep the surface "not a game shell" and to preserve a quiet, one-screen scene instead of a menu-heavy product surface.
- **Two starter birds** - NOT RECOVERABLE FROM PLAN
- **Age-based additions through a hard cap of seven** - The plan ties growth to "quiet change over weeks" and later says to ramp later birds "conservatively by aviary age toward the seven-bird cap" only after engine/audio recognizability and stability gates.
- **Email magic-link authentication** - NOT RECOVERABLE FROM PLAN
- **Per-device sessions** - NOT RECOVERABLE FROM PLAN
- **Account settings** - The plan limits settings to matter-of-fact system settings such as accessibility, captions, audio preference, and optional visit notification, and says not to place engagement metrics there.
- **Email change verification** - NOT RECOVERABLE FROM PLAN
- **Account export** - The plan gives the why directly: the export is a "data portability artifact, not a product stats surface," and generation reads canonical state under account authorization.
- **Soft-then-hard deletion** - The why is account recovery during the soft-deletion window and privacy completion after the deadline: restore is allowed within 30 days, then a scheduled purge hard-deletes account-tied records.
- **Stable bird identity** - Stable IDs protect the bird as an ongoing relationship object: the ID is "never regenerated by rename, sync, species-pool change, or migration."
- **Renameable bird names** - Renaming changes the user-facing name but causes "no personality/mood change," preserving stable identity and avoiding hidden simulation effects from a matter-of-fact edit.
- **Small coherent species pool** - NOT RECOVERABLE FROM PLAN
- **Server-authoritative mood and personality drift** - The why is the central canonical contract: the server owns drift and mood updates, clients never submit absolute traits, and mood advances even while no client is connected.
- **Presence accounting** - Presence is the "dominant positive input" for drift, but the plan protects it from corruption by requiring visible, focused, recent activity and rejecting background-only keepalives.
- **Return greetings** - Return greetings enact the rule that the aviary notices the viewer through bird behavior; absence duration selects a likely greeter and gesture/call without a textual announcement.
- **Bird-to-bird responses** - The plan uses responses so calls can prompt calls and signals can spread, while resolving from pre-tick state so iteration order does not create "accidental bias."
- **Listen-in** - Listen-in is meant to read as attention rather than UI switching: gain and visual focus ramp gradually, the server records intent/duration, and neighbors remain ambient rather than silent.
- **Seed, song-fragment, and still-pool offers** - Offers are an interaction path that can produce small curiosity or boldness signals on acceptance or proximity, while server cooldowns and assignment keep durable effects canonical.
- **Settle** - Settle ends the presence window, quiets mood, warms/dims the scene, and has "no positive/negative personality direction," so leaving does not punish or reward the relationship.
- **Sparse read-only field notebook** - The notebook turns selected simulation observations into quiet, immutable naturalist prose while avoiding "raw interaction counts or personality numbers."
- **Local-time day/night** - Local time is used to make behavior feel continuous and situated: birds may become drowsy near dusk/night or alert in early morning, without snapping to a login default.
- **Quiet weather events** - Weather affects scene ornaments and mood through canonical state, with bounded behavior so even high vocal-frequency birds can be quiet in rain or at night.
- **Continuous ambient motion** - Ambient motion supports the sense that the aviary is already alive; the first frame should be non-zero pose phase and "mid-action," with no spinner or ready transition.
- **Responsive one-screen rendering** - The rationale is to preserve the horizontal, glanceable scene: all birds remain visible, spacing compresses or widens, and the renderer must never pan, zoom, or crop a bird.
- **Procedural calls** - Procedural calls keep signatures recognizable while avoiding downloaded loops, recorded fallback, and canned repetition.
- **Chorus mixing** - Chorus mixing supports bird-to-bird aliveness while bounding polyphony and CPU, and the plan says a focused bird's neighbors must never become silent.
- **Call captions** - Captions preserve call meaning when audio is unavailable or muted and must match the same call intent/motif parameters sent to the synthesizer.
- **Screen-reader narration** - Narration is a primary accessibility surface using the same semantic snapshot, so nonvisual users receive naturalist observations rather than raw coordinates or internal state.
- **Keyboard access** - Keyboard access makes the sparse top bar, bird focus, listen-in, offers, and settle fully reachable while keeping reading order stable as birds move visually.
- **Reduced-motion rendering** - Reduced motion is first-class: high-motion paths become slow cross-fades while mood, calls, captions, day/night, offers, settle, and notebook remain recognizable as the same aviary.
- **Per-invite, revocable, read-only visits disabled by default** - The why is visitor privacy and host-state protection: visits require explicit action, cannot submit interaction or presence events, and revocation is observed immediately on pull.
- **Silent visit log and optional host notification setting off by default** - The plan keeps visits socially quiet: the host can view requested log facts, but notification pressure is optional and off by default.
- **Configuration and feature flags for open values** - The plan avoids hard-coded product promises by assigning calibration owners to values such as inactivity, tick cadence, mood constants, age schedule, cooldowns, audio ramps, and weather frequency.

### Architecture and state authority

- **Small modular backend with a single-writer simulation boundary** - The why is to avoid "a distributed set of independently authoritative services" and keep canonical simulation changes in one authoritative path.
- **Web shell and renderer owning presentation but not canonical personality or mood** - The module owns interpolation, local-time presentation, WebAudio, captions, and accessibility, but "never owns canonical bird personality or mood" because those are server state.
- **Aviary API validating events without applying personality deltas** - The API ingests append-only interaction events and validates authorization/shape, but does not mutate personality so the simulation worker remains the only writer.
- **Simulation worker as only writer of canonical state** - The worker owns personality vectors, moods, pose/position state, call scheduling, weather, and notebook observations so each tick has one authoritative result.
- **Operational telemetry pipeline with aggregate measurements only** - The why is privacy: it receives request, latency, render, audio-error, and tick-health measurements, but has "no read path to per-account simulation records" and no per-bird/per-account dimensions.
- **Transactional relational source of truth** - Event log and current state stay in "the same consistency domain" so a tick can claim events, apply a versioned update, advance its cursor, and publish a snapshot atomically.
- **Object store or signed export worker only for on-demand exports** - The plan allows object storage for export artifacts but says not to put persistent simulation state in client storage or outside the canonical store.
- **Cache/CDN for immutable assets and short-lived bootstrap only** - The cache may accelerate delivery but "cannot become a second state authority."
- **Explicit semantic client/server boundary** - The server sends canonical IDs, names, species, mood inputs, weather, call metadata, notebook cursor, version, and server time; the client derives interpolation, palette, ornaments, audio, captions, focus, and reduced motion.
- **Edge-friendly bootstrap with quiet sky/field fallback** - If authenticated bootstrap is not ready, the client draws faint ambient cues "never a spinner or entry animation," preserving the quiet product contract.
- **First usable state with non-zero pose phase** - The why is that the first rendered frame should look "mid-action," not newly spawned or waiting for the app.
- **Opaque UUIDs and encrypted email** - The plan uses this to prevent email from becoming an identifier, partition key, log field, event payload key, or analytics dimension.
- **Hashing raw session and magic-link tokens** - NOT RECOVERABLE FROM PLAN
- **Protected bird personality table or protected access path** - The rationale is access control and product boundary: personality is kept out of normal client DTOs, clamped in a documented range, and has no client write path.
- **Append-only interaction events with idempotency and consumed cursors** - This gives the tick ordered facts it can claim, deduplicate, consume once, and connect to a tick version.
- **Presence intervals ending on hidden, unfocused, settle, disconnect timeout, or inactivity** - The why is to keep presence tied to actual visible/focused/recent activity and avoid background tabs inflating drift.
- **Immutable, cursor-paginated notebook entries** - Entries are read-only and sparse so the notebook remains naturalist prose rather than an event-log or stats surface.
- **Unused visit invitations expiring after 30 days and not being revived** - NOT RECOVERABLE FROM PLAN
- **Hard deletion cascading or purging account-tied records** - The why is privacy completeness: account deletion must remove or purge records across simulation, sessions, invitations, logs, exports, and telemetry references.

### API surface and protocol rules

- **Versioned JSON endpoints with request IDs and idempotency keys** - The plan uses these for diagnostics and to make every mutating request safe to retry or apply once.
- **Snapshot version and server time on state responses** - These allow clients to reconcile snapshots, compensate for client clock differences, and diagnose stale state.
- **Matter-of-fact system errors and naturalist product observations** - The why is product voice separation: errors stay direct while aviary observations remain lowercase and naturalist.
- **Bootstrap and snapshot endpoints with full refresh after hidden-tab return, suspension, frame gaps, or version mismatch** - Full snapshots repair stale client presentation without making the client authoritative.
- **Event ingestion returning accepted sequence IDs, not derived personality** - The plan lets the server acknowledge ordered facts while keeping derived personality hidden and server-owned.
- **Bird rename endpoint with no personality or mood change** - Rename is validated as a matter-of-fact identity edit, not a simulation input.
- **Offer endpoint or event wrapper with server assignment and cooldown enforcement** - Server assignment and cooldowns keep offer effects canonical when the UI has not already selected a receiving bird.
- **Settle and re-engagement endpoints appending events while the tick owns results** - The rationale is that durable mood and scene state remain simulation outcomes, not immediate client writes.
- **Account settings endpoint without streak or visit-frequency metrics** - Settings are system settings; the plan explicitly bars engagement metrics from this surface.
- **Export endpoint emailing a signed, expiring link to the verified address** - Export remains authorized data portability and not an in-product stats view.
- **Visit token resolution that never issues host mutation capability** - The why is read-only visitation: visitors can pull the host snapshot but cannot mutate host state.
- **Polling instead of WebSockets for freshness in v1** - Polling "preserves the canonical model"; WebSockets are rejected if their purpose would be broadcasting client-owned state.
- **Keepalive not counted as presence unless visible, focused, and recently active** - This protects drift from background tabs and passive connection traffic.
- **Stale snapshot versions not rejected for event append** - The plan treats events as appendable facts; the server assigns ordering even if the client observed an older snapshot.
- **Transactional compare-and-set for once-only mutations and no last-write-wins personality** - This prevents rename, adoption, or revocation from duplicating and prevents stale devices from overwriting drift.

### Server-side simulation engine

- **Scheduler claiming due aviaries with a per-aviary lease** - The why is single authority: "never run two ticks concurrently for one aviary."
- **Event normalization, clamping, and idempotency deduplication** - These keep event durations bounded, duplicate delivery harmless, and tick inputs reproducible.
- **Bird-to-bird responses resolved from pre-tick state** - The plan states the why directly: iteration order must not create "accidental bias."
- **Deterministic tick with stable per-aviary/per-bird seed** - Determinism supports tests and debugging and makes retry produce the same result or fail by version/lease checks rather than double-applying.
- **Explicit bounded low-pass drift accumulator** - The plan uses this to make trait changes server-authored, bounded, and gradual rather than read-time derivations from raw event logs.
- **Presence, listen-in, offers, and settle as drift or mood inputs** - Presence is dominant positive input, listen-in affects social warmth/vocal frequency, offers add small curiosity or boldness, and settle quiets mood without trait direction.
- **Non-negative personality deltas and no negative neglect drift** - The why is to avoid distress or punitive absence: neglect creates no negative drift and "no distress."
- **Time-accelerated simulation harness** - The harness verifies one-week measurable instrument drift, three-week user-visible change, two-week absence, concurrent devices, duplicate delivery, hidden-tab presence, and seven-bird chorus.
- **Small versioned mood enum and signal-based transitions** - Mood combines recent events, local time, weather, personality, and neighbors, persists across sessions, and advances without clients.
- **Mapping mood and personality to behavior instead of numeric UI** - The plan hides numbers by expressing traits through perch probabilities, approach/retreat, pose families, call frequency, latency, and offer reactions.
- **Stable procedural call signature and motif intents** - Each bird remains recognizable as mood/personality change, while the engine emits parameters rather than storing or downloading loops.
- **Return greeting with one likely first greeter and staggered responses** - The plan avoids a "synchronized arrival fanfare" and uses absence as behavior input rather than announcement text.
- **Notebook generation behind cooldown and novelty checks** - Sparse generation keeps entries "roughly every few days" for regular users, allows noteworthy moments, and avoids event-log language, counts, drift numbers, streaks, or user-behavior summaries.

### Client rendering pipeline

- **Lightweight scene renderer behind a semantic DOM interaction layer** - The renderer can draw the scene while DOM controls, captions, focus proxies, narration, and settings stay keyboard and screen-reader addressable.
- **SnapshotInterpolator and SceneRenderer separation** - The plan keeps the renderer API independent of transport: one component supplies pose/mood/scene values while the other draws.
- **No inline scene buttons, badges, tooltips, or labels** - This keeps the scene free of chrome and aligned with the sparse top-bar, glanceable-scene intent.
- **Immediate draw from bootstrap with no fade-in, spinner, or ready transition** - The why is to make the aviary appear already in progress and quiet rather than app-like.
- **Interpolation using server time** - Server time compensates for client clock differences and avoids teleports after normal polling.
- **All birds visible across responsive aspect ratios** - The plan preserves the one-screen scene by compressing or widening spacing while never panning, zooming, or cropping a bird.
- **Local-time palette and weather ornaments** - Palette derives from account time zone while weather comes from canonical state; client-only leaves/feathers remain ornaments outside simulation state.
- **Top-bar fade that preserves keyboard focus visibility** - The bar can fade toward transparent after cursor stillness, but it must restore on activity and never hide focused controls.
- **Hidden-document render pause and fresh snapshot on return** - This protects performance and repairs canonical presentation after hidden tabs or long frame gaps.
- **No mood labels or personality values as normal controls** - The plan keeps mood and personality expressed behaviorally rather than as visible stats.
- **Transparent semantic bird focus proxies with soft outline** - Focus proxies make birds reachable by keyboard and screen reader while keeping the visual scene free of chrome.
- **Listen-in, settle, offer reactions, and perch changes using attention-like motion curves** - The plan says these transitions should read as attention, not "UI state switching."
- **Settle reversal on aviary click within five seconds** - The why is forgiving accidental clicks while avoiding extra system announcements.
- **Optimistic reversible client transitions with server reconciliation** - The client may show reversible transition feedback, but the server remains authoritative for durable effects.
- **Reduced-motion renderer mode** - The mode replaces micro-motion and flight paths with slow cross-fades, removes leaf drift, and still preserves the same canonical aviary behavior.

### Audio pipeline

- **One bounded WebAudio context per visible client** - The plan bounds audio resources and initializes/resumes only from an allowed gesture where autoplay policy requires it.
- **Motif metadata and procedural synthesis instead of recorded calls** - This avoids recorded loops and supports stable signatures with bounded variation.
- **Pooling and releasing audio nodes/resources** - The stated reason is that a 30-minute session should not grow memory.
- **Per-bird gain buses into an ambient/chorus master** - This keeps all active birds audible in normal playback and prevents listen-in from making neighbors silent.
- **Bounded chorus polyphony and CPU budget** - The plan uses this to keep overlapping calls alive without letting a seven-bird chorus become muddy or expensive.
- **Gradual listen-in gain ramp and clear disengage conditions** - The ramp makes focus feel gradual and reversible; the client owns only the ephemeral mix curve while the server records intent/duration.
- **Captions generated from the same call intent as synthesis** - Captions describe the actual procedural call in its current bird/mood context, not a fixed per-bird string.
- **Graceful silence when WebAudio is unavailable, blocked, or errors** - The plan treats captions as the designed fallback, rejects recorded-audio fallback, and keeps simulation, visuals, captions, and narration running.

### Accessibility surfaces

- **Accessibility as a parallel product surface** - The plan states that accessibility reads the same canonical state rather than being labels added after the visual build.
- **Polite screen-reader live narration region** - Narration updates only when the observation materially changes, prioritizes return greeting, accepted offer, and settle, and avoids flooding the queue.
- **Captions on by default when audio is unavailable** - Captions preserve call information when the audio path is blocked or muted.
- **Keyboard traversal through top bar and birds** - The keyboard model makes offers, settle, bird focus, and listen-in reachable while keeping reading order stable as visuals move.
- **Accessible focus and hit targets without internal stats** - Names derive from bird names/species, focus treatment is high contrast, and the plan avoids hover/audio as the only state channel.
- **WCAG AA contrast and copy split** - User copy, top-bar labels, settings, account/error text, displayed narration, and captions must meet contrast, while system copy stays matter-of-fact and aviary/notebook copy stays naturalist.
- **Reduced motion and audio off preserving the same aviary** - The rationale is that mood, calls-as-captions, day/night, offers, settle, and notebook behavior must remain recognizable without motion or audio.
- **Automated checks plus VoiceOver/NVDA/keyboard passes across primary states** - The plan makes accessibility acceptance state-based, including fresh, returning, settled, nighttime, rain, reduced-motion, audio-denied, visitor, and error states.

### Performance, observability, and delivery

- **Initial JavaScript bundle under 2 MB gzipped with code splitting** - The why is first-paint discipline: settings, notebook history, accessibility settings, and visit invitation flow are split away from the critical scene path.
- **First bird visible under 500 ms on mid-tier mobile over 4G** - This release gate keeps the first visible relationship moment fast and avoids waiting on non-critical assets.
- **60 fps idle motion and no client memory growth over 30 minutes** - The plan treats smooth ambient life and long-session stability as release gates, testing two/seven birds, weather, captions, listen-in, audio nodes, timers, buffers, and hidden/visible transitions.
- **Simulation tick p99 under 5 seconds with kilobytes-scale snapshots and polling backoff** - The rationale is operational health without thundering-herd behavior.
- **Aggregate-only metrics and privacy-boundary CI tests** - Metrics are limited to request/error/latency, tick, snapshot, render, audio, capability, and anonymized duration signals; denylist tests fail CI if forbidden dimensions enter schemas.
- **Foundations and contracts before UI work** - The delivery sequence starts with versioned domain types, authorization, migrations, event schemas, snapshot schema, calibration, privacy/retention policy, and contract tests so later surfaces consume stable contracts.
- **Canary and conservative bird-count ramp** - Release starts behind flags with internal synthetic accounts and a small real-account canary; all accounts start at two birds, and later age-based birds wait for engine/audio recognizability and stability gates.
- **Definition of done confirming prohibited paths are absent** - Launch requires explicit confirmation of no gamification, notification, public discovery, co-presence, or client-authored personality path, keeping the v1 boundary intact.
