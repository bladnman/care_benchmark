## System-level intent

- The plan treats feel as architecture, not decoration. This appears in the opening frame: affective rules are "load-bearing," "feel constraints are the architecture," and product policies are translated into "invariants with enforcement." The repeated move is to enforce desired behavior at the database, role, schema, CI, design-system, and infrastructure layers.

- The plan prefers structural absence over policy. "Features that are not built" are considered safer than guidelines. This shows up in "No announcement surface exists," no toast/snackbar/badge primitive, the streak metric "not computed," no native app shaping, no social-network surfaces, and no recorded-audio pipeline.

- The aviary is server-authoritative and continuing. The server is the "sole writer" of personality and canonical mood/state; the tick runs whether or not a client is connected; the first paint should already be "the aviary," because it has "been continuing without the viewer."

- Growth is additive, monotonic, and non-punitive. Personality changes are "server-authored deltas applied in event-log order"; drift is "monotonic toward expressive"; neglect produces "no delta." The plan's explicit asymmetry is that birds may become "quieter, not warier," preserving "no Tamagotchi."

- Presence must be honest, not inferred from a tab being open. The presence rule is a "3-condition conjunction": visible, focused, and recent pointer/key activity. The plan repeats that "tab open" alone never emits and cannot inflate drift.

- The bird's interior state should be legible without numbers. The "personality vector is never exposed numerically," and mood is read from motion with "no label/tooltip/icon." Render and audio expose derived behavior, not raw traits.

- Privacy is enforced by topology, not best intentions. Email lives in "exactly one place, encrypted"; every other reference is a synthetic UUID. Telemetry has "no network path/credential to the simulation DB," and metric definitions exclude per-bird or per-account dimensions.

- Voice is split deliberately between naturalist and system surfaces. The plan uses "Naturalist voice everywhere except system surfaces"; sign-in, account, sync errors, unsupported-browser, and similar flows use "matter-of-fact voice," while notebook, narration, captions, and ambient surfaces use naturalist prose.

- Procedural variation protects the spell of living presence. Audio is "procedural-only," calls are "never identical twice yet always recognizably this bird," and a stable bird id is "never reused, regenerated, swapped, or reset." The plan treats canned repetition and identity reset as affective failures.

- Accessibility and performance are product constraints from v1, not polish. Accessibility is "a designed surface shipped with v1," "launch-blocking," and "not a fallback retrofitted later." Performance budgets are "enforced in CI" because first-bird speed, 60fps idle, and no memory growth support the product feeling alive.

## Per-feature whys

### Scope and non-goals

- Single-user accounts and account lifecycle: The plan keeps the product centered on one account and one aviary, avoiding social-network surfaces while still supporting sign-in, sessions, export, and deletion as system surfaces in matter-of-fact voice.

- Magic-link sign-in: The plan uses email because sign-in "inherently needs to find an account by email," but wraps that in a keyed HMAC lookup so plaintext email remains in one encrypted column and every other reference remains the UUID.

- Per-device revocable sessions: NOT RECOVERABLE FROM PLAN

- Email-change with verify: NOT RECOVERABLE FROM PLAN

- Account export: The plan frames export as "data portability" to the owner only, not a stats panel. Raw vectors are flagged as a defensible call because this may conflict with a maximal reading of "personality numbers never surfaced."

- Soft-delete followed by hard-delete: The plan gives a 30-day "I changed my mind" restore surface, then hard-deletes birds, vectors, notebook, events, telemetry linkage, and everything else, aligning account lifecycle with privacy and loss prevention.

- One canonical aviary per account: The plan makes sync "a property of server-authoritative state, not a feature." A single canonical aviary lets every device read the same continuing state rather than maintaining competing local worlds.

- Two starter birds, cap seven, third-plus by aviary age: The cap supports call recognizability, with the plan saying recognizability is "what makes 7 the cap." Age-based offers avoid tying new birds to "visit count / interaction score / payment."

- System-selected species for starter birds: NOT RECOVERABLE FROM PLAN

- User-named birds: NOT RECOVERABLE FROM PLAN

- Hidden personality vector: The rationale is that personality should shape behavior without becoming a numeric stat. The plan forbids raw trait values in API fields, DTOs, ARIA attributes, logs, and client-facing schemas.

- Fast-timescale mood: Mood carries short-term life in the aviary while personality changes slowly. The plan uses stored mood, hysteresis, time of day, weather, interaction, and bird-to-bird pressure so mood feels persistent rather than resetting on tab open.

- Monotonic-up drift: This is the engine-level way to avoid Tamagotchi mechanics. Positive presence and interaction can increase expressiveness, while neglect does not punish personality.

- Bird-to-bird interaction: The plan wants the aviary to behave "like a small social system, not independent NPCs," so wary mood can spread and high-vocal birds can affect chorus-join probability.

- Procedural call grammar: Procedural calls avoid canned repetition and support individual recognition. The species gives a signature, the bird gives a stable pitch/timbre offset, and the seed keeps calls varied but recognizable.

- Server-side simulation tick: The tick is the only writer of canonical personality, mood, perch intent, weather, greetings, and notebook generation. This makes "server-authoritative personality" enforceable, not just descriptive.

- Single horizontal scene: The plan rejects pan, scroll, and zoom so the aviary remains one quiet field where all birds stay visible.

- Three perch zones: NOT RECOVERABLE FROM PLAN

- Day/night by local time: The plan uses account timezone to derive day/night and mood time-of-day signals, making the aviary advance with the user's local rhythm rather than freezing at session boundaries.

- Rare ambient weather: Weather is ambient and "short and never assertive"; it creates mood pressure and atmosphere without becoming an announcement or game event.

- Parallax and ambient leaf/feather drift: These are "pure client-side" ornaments with "no sim state," keeping visual life present without polluting canonical state.

- First frame mid-action and quiet-field load state: The first paint is meant to express that "the aviary has been continuing without the viewer." A spinner or entry animation would make it feel started by the user's visit, so slow load uses a quiet field instead.

- Thin auto-fading top bar: The top bar keeps account, accessibility, notebook, and offer controls available while preserving "No UI chrome inside the scene." It fades to near-transparent so the aviary remains the primary surface.

- Return-greeting: The greeting is shaped by absence, boldness, and mood, but is staggered and never unison because unison would "announce." This lets the birds notice without creating a "welcome back" product surface.

- Listen-in: The plan makes listen-in a "re-balance, not a mute." Slow gain ramps make it feel like "listening," not "switching channels," and other birds drop but never go silent.

- Offer interaction: Offers create small, bounded signals into curiosity, boldness, and related traits without making interaction a score loop. Server-side cooldown keeps the interaction from becoming click-driven drift.

- Offer item set of seed, song-fragment, and still-pool: NOT RECOVERABLE FROM PLAN

- Settle interaction: Settle has "no directional drift" and is treated as a "clean window end only," avoiding a mechanic that rewards or punishes the bird.

- Five-second settle undo: NOT RECOVERABLE FROM PLAN

- Field notebook: The notebook is "sparse, read-only, naturalist" and records "observations of the aviary," never observations of the user's behavior. It avoids session timestamps, visit-frequency, and event-log dumps.

- Multi-device sync: Sync follows from server-authoritative state and idempotent event-log replay, so multiple devices resolve to the same additive result instead of last-write-wins conflict.

- Visit invitations: Visits are the "entire social surface." They are opt-in, revocable, expiring, and read-only so they remain ambient rather than becoming profiles, follows, feeds, comments, co-presence, or discovery.

- Visit log and off-by-default visit notifications: The visit log is reachable on demand with no badge or notification, and notifications default off, preserving "No announcement surface exists."

- Accessibility package: Narration, reduced motion, captions, and keyboard navigation are launch-blocking so users get the same product, not a degraded fallback.

- Performance package: Bundle size, first-bird time, 60fps idle, no memory growth, and procedural WebAudio are framed as necessary to make the aviary feel alive and quiet rather than heavy or stalled.

- Aggregate-only telemetry and synthetic checks: Operational observation exists, but only at aggregate level or through synthetic browsers. The plan deliberately does not define engagement loops, per-account dimensions, or visit-frequency metrics.

- No native app: The plan says protocols and data model are "not shaped for a future native client," keeping v1 decisions focused on the web product rather than speculative portability.

- No social-network surfaces: The rationale is to prevent visit scope-creep into leaderboards, discovery, co-presence, friend-of-friend, comments, follows, feeds, or profiles. The architecture makes those additions harder "by design."

- No notification, push, or email-about-the-aviary surface: This supports "notice never announce" and prevents the product from turning bird behavior into re-engagement prompts.

- No Tamagotchi mechanics: The plan rejects death, hunger, distress, decaying happiness, and negative personality drift so absence does not become harm. Neglect can quiet recent behavior without making birds warier.

### Architecture, data, and API

- Edge/CDN layer with inlined initial snapshot: This makes the "<500ms first-bird budget" reachable by drawing from a short-TTL snapshot before full hydration.

- API service with no personality update grant: The API role can handle auth, reads, settings, visits, export, and events, but cannot update personality columns. A bad client write path fails at the database.

- Separate simulation tick worker: Splitting the tick into its own deployable with its own DB grants is described as the "cheapest way" to make server-sole-writer and event-ordered deltas unfalsifiable.

- Mailer: Mailer surfaces use `voice.system.*` because magic links, export links, visit invites, and optional visit notifications are system surfaces, not naturalist aviary prose.

- Separate telemetry plane: The separate account, VPC, and credentials enforce that aggregate metrics cannot reach the simulation DB or per-bird data.

- Server/client boundary: The server owns anything canonical and durable across devices or absence; the client owns latency-sensitive, ephemeral rendering, interpolation, exact waveform, and ambient ornaments.

- Client-local call scheduler with server parameters: The plan chooses this for latency and to avoid per-call server chatter, while preserving canonical personality and mood on the server through shared parameters and seed.

- Postgres plus append-only event log: Postgres is the v1-scale choice, "not Kafka," and the event log is the spine for ordered, additive, idempotent updates.

- Encrypted email plus HMAC lookup index: The HMAC index reconciles "email never an identifier" with sign-in by email. It is one-way, never logged, never used as an FK, partition key, shard key, or inter-service identifier.

- Coarse device label on sessions: The plan says the label is coarse and uses "no fingerprinting," aligning session management with the privacy posture.

- Stable bird id: The rationale is identity continuity. A bird id is immutable across rename, sync, species-pool changes, and migrations so the bird is not effectively reset or swapped.

- Per-bird call seed: The seed supports procedural call continuity and is "advanced, never reset," giving each bird recognizable variation over time.

- Recent-presence behavior term: This term separates short-term liveliness from personality, so absence can quiet greetings without reducing personality traits.

- Event log per-account sequence and event UUID dedup: Monotonic sequence gives event-log order, while event UUID dedup and tick watermarks make replay idempotent and avoid lost drift across devices.

- Visit session as render-only: Visitor sessions exist only for the log and "never feed drift," protecting host birds from being shaped by visitor attention.

- Raw presence pings aging out: The tick folds pings into bounded accumulators and recent-presence terms, then raw pings age out to keep the privacy surface small.

- Backups and PITR for personality vectors: The plan says losing a vector is "deleting the bird the user knows," so backups are configured specifically to protect personality state.

- Auth request-link always returns 200: This prevents account enumeration while still sending a link when appropriate.

- Auth consume is single-use with matter-of-fact replay handling: Token consumption checks expiry and consumption, issues a session once, and describes replay as "link may have expired" in system voice.

- Snapshot payload with derived render/audio parameters but no raw traits: The client receives mood, perch, saturation, vocal parameters, call seed, call phase, and greeting information, not numeric personality values.

- Snapshot pull triggers: Pulling on load, visible return, long render-frame gap, and low-frequency keepalive keeps the rendered aviary aligned with the server state that kept running.

- Rename event has no engine effect: Names update directly and do not affect personality, preserving the hidden engine from user-facing labels.

- Owner-only data export with raw-vector flag: The plan's why is data portability, but it flags PRD-owner confirmation because exporting raw traits may violate a strict reading of no personality numbers.

- Account deletion restore: The pending-deletion page gives a quiet "I changed my mind" path for 30 days before scheduled hard deletion.

### Simulation engine

- Tick loop sequence: The tick reads events in sequence, derives time, advances weather, mood, drift, perch intent, calls, greetings, notebook, tick sequence, and watermark so all canonical changes happen in one ordered pass.

- Presence detector: The detector emits only when the document is visible, focused, and recently active, ensuring "tab open" cannot count as presence.

- Activity window leaning long: The plan leans toward 3-4 minutes because "watching without moving is the actual product," while still requiring the three presence conditions.

- Drift formula with EMA and diminishing returns: Low-pass accumulators prevent one session from moving traits visibly, and `k_t * acc_t * (1 - v_t)` creates slow movement toward a ceiling over weeks.

- Drift input weights: Presence-time is dominant across traits; listen-in strongly affects the focused bird's social warmth and vocal frequency; offers nudge curiosity and boldness. The plan keeps these as small, directional signals.

- Synthetic-agent calibration harness: Calibration uses synthetic agents and a consented internal cohort, never mined production per-bird data, because tuning from population drift curves would violate the privacy boundary.

- Mood transitions with hysteresis: Dwell floors and pressure margins prevent per-tick flapping, so mood reads as persistent rather than snapping.

- Mood persistence across sessions: Session start reads stored mood advanced by ticks during absence, avoiding "snap to neutral" on tab open.

- Idle motion as mood surface: Wary, content, curious, and drowsy states are visible through scanning, preening, head-tilt, or fluffed posture, with "no label/tooltip/icon."

- Notebook generation in the tick: Sparse generation from real moments keeps the notebook observational and rare, while the generator's lack of access to visit-frequency protects it from becoming user-behavior feedback.

### Frontend rendering

- Canvas2D renderer with parametric vector birds: Canvas2D is the "bundle-safe default"; parametric shapes let plumage saturation become a continuous render input and make reduced-motion pose cross-fades cheap.

- Continuous idle micro-motion: Procedural preen, scan, head-tilt, and weight-shuffle prevent the bird from reading as a "paused loop." The plan says looking paused is a bug against the headline principle.

- Reduced-motion mode: Reduced motion is not "animations off"; it gets slow cross-fades, removed leaf drift, slowed color shift, and full drift, mood, notebook, calls, and captions, making a "calmer" Pocket Aviary rather than a broken one.

- Responsive scene: The plan keeps one horizontal scene at every viewport, compressing without cropping any bird out so all birds remain visible.

### Audio pipeline

- Per-bird synth graph with reused nodes: Reused buffers and nodes support the no-memory-growth budget while keeping each bird separately mixable.

- WebAudio lookahead scheduler: Scheduling on the WebAudio clock avoids blocking on the main thread and keeps calls aligned with server parameters and call seed.

- Chorus from overlapping procedural calls: Chorus should be a "genuine chorus," not recorded loops phase-cancelling, so overlapping high-vocal birds use real per-call variation.

- Captions generated at synthesis time: Captions come from the same call-grammar parameters as the actual audio, so the displayed naturalist phrase matches what played.

- Silence plus captions fallback: If WebAudio is unavailable, the plan chooses graceful silence with captions because recorded fallback would break procedural variation and bundle constraints.

### Accessibility, performance, and rollout

- Screen-reader narration: Narration uses naturalist prose from the same state as the visual and avoids state-list language like "Pip mood: content."

- Dual aria-live regions: A polite ambient region and separate priority region let user-initiated events announce promptly without flooding the screen-reader queue.

- Keyboard navigation: Full keyboard paths make listen-in, offer, settle, bird focus, and exit reachable without a pointer, meeting the plan's v1 accessibility requirement.

- Focus indicators: The soft high-contrast outline is meant to stay legible against both bright and dim aviary states.

- WCAG AA on all user copy: The plan concentrates contrast work on top-bar labels, settings, account/error surfaces, captions, and visible narration because the scene itself carries little copy.

- Voice continuity between narration and notebook: Sharing `voice.naturalist.*` means a screen-reader user moving between surfaces hears "one product, not two."

- Bundle, first-bird, frame-rate, and memory gates: These are enforced in CI so budget breaches fail builds rather than becoming subjective review comments.

- Synthetic performance fleet and aggregate RUM: Synthetic browsers and aggregate metrics observe first-bird render, frame timings, audio-context errors, and tick latency without per-account or per-bird dimensions.

- Internal dogfood, closed beta, and GA sequence: Dogfood calibrates activity window, drift, and tick cadence; closed beta validates real-device budgets, recognizability, narration, and reduced-motion feel; GA waits for accessibility to ship with the default path.

- Feature flags for ramp pacing, tick cadence, and activity window: These allow safe tuning of age-based bird offers and simulation parameters while keeping the invariants intact.
