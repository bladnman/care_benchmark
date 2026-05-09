## System-level intent

- **The aviary must feel alive because it actually continues.** This shows up in the scope's "server-side simulation tick," in the architecture's insistence that "the aviary continues without the viewer," and in the tick engine being "the difference between 'feels alive over weeks' and 'screensaver.'" The plan treats continuation as a real server property, not a client illusion.

- **The product should notice without announcing.** The plan names "notice-never-announce" as a design principle and carries it through by refusing "Welcome back!"-style surfaces, avoiding forced chorus cues, skipping a "wake-up" animation on resume, and making return-greeting procedural rather than celebratory.

- **Charm comes from specificity, not abundance.** The plan names "charm-from-specificity" and applies it to notebook prose, narration, captions, and visual signals: phrases reference birds by name, perch position, mood verbs, and ambient cues; captions are generated from the "actual pitch shape and duration envelope."

- **Restraint is a product constraint, not just a visual style.** "Restraint-over-richness" appears in the non-goals, in sparse notebook cadence, subtle weather, capped rain intensity, no tutorial sequence, no stat panels, no recorded-audio fallback, and the refusal of engagement mechanics.

- **Voice is split by surface: naturalist for the aviary, matter-of-fact for system controls.** The plan names the "naturalist-vs-matter-of-fact voice split." Narration, notebook entries, and captions use naturalist prose; auth, errors, settings, unsupported-browser copy, naming UI, and account-state surfaces use matter-of-fact voice.

- **The engine must be true; the surface must feel true.** The closing line says "The engine is the thing that has to be true, and the surface is the thing that has to feel true." This shows up in the snapshot boundary: server thinks in personality and mood; client thinks in positions, opacities, and audio events.

- **Server authority protects identity, sync, and mood from client corruption.** The plan repeats that "the server is the sole writer of canonical state," events are append-only, and the `sim` service is the only writer of personality. This is presented as the reason last-write-wins cannot become a regression.

- **The system must refuse engagement features structurally.** Gamification, streak counters, Tamagotchi mechanics, social network surfaces, and per-account analytics are not merely omitted from the UI; the plan says the underlying counters, columns, code paths, and warehouse dimensions should not exist.

- **Raw personality must not leak into user-visible or analytics surfaces.** The plan repeatedly says no numeric exposure of personality vectors, no raw values in snapshots, no internal admin numeric view, no per-bird state in telemetry, and rendered visual signals only.

- **Accessibility ships with the product and has its own aesthetic.** Reduced motion is "its own designed surface," not a stripped fallback. Captions, screen-reader narration, keyboard navigation, focus indicators, contrast checks, and manual NVDA/VoiceOver review appear as first-class implementation surfaces.

- **Determinism and calibration are preferred over black-box magic.** The plan uses seeded RNG, deterministic ticks, named calibration constants, version history, beta tuning, grammar-driven narration, and no foundation model in v1 so behavior can be tested, replayed, and tuned.

- **Bird identity is durable.** Stable UUIDs, no regenerate endpoint, no personality reset tool, versioned personality vectors, migrations that never delete existing values, and the phrase "the bird the user adopted on day one is that bird" all carry this intent.

## Per-feature whys

### 1. Scope

- **Single-user accounts, magic-link sign-in, per-device session tokens, account export, soft-then-hard delete**: The plan grounds these in account-state control, transactional email, revocable devices, verified-address export, and a recoverable pending-delete state before hard deletion.

- **One canonical aviary per account**: The rationale is sync and aliveness. Two devices read from the same canonical state, and closing one device does not freeze the aviary.

- **Single horizontal scene**: NOT RECOVERABLE FROM PLAN

- **Two starter birds**: The plan seeds them with "complementary" vectors so the first encounter reads as two different birds, not two neutral engine instances.

- **Age-gated ramp toward seven max birds**: The plan says day 30 means "an aviary that has settled in," later gates use compounding intervals, and a year-old aviary reaches near-cap.

- **Six-species pool**: NOT RECOVERABLE FROM PLAN

- **Stable bird identities and user-assigned names**: Stable IDs preserve the bird across renames and migrations. Names let prose, captions, and notebook entries reference birds specifically; renames are logged so the user can recall naming history.

- **Server-canonical personality vectors**: The rationale is to keep personality out of client mutation paths and preserve drift, identity, and sync correctness.

- **Server-side simulation tick**: The plan says this is what makes "continues without the viewer" a real property and prevents frozen-client or multi-device merge problems.

- **Procedural call synthesis client-side with WebAudio**: The plan uses it to avoid recorded audio, keep calls varied, preserve the "no looped audio" invariant, and stay inside the bundle budget.

- **Chorus mixing**: The plan wants overlapping calls to happen statistically. Forcing chorus would "announce," while emergent overlap keeps it alive.

- **Listen-in mix**: The mix makes focused listening feel gradual and attentive, not like soloing tracks. Other birds rebalance lower but are "never silent."

- **Per-call captions**: Captions let the user read calls, especially when audio is unavailable or muted, while preserving naturalist voice through specific phrases.

- **Procedural visual scene with perch zones, day/night, weather, parallax, and drift**: Day/night ties the aviary to the user's actual day; weather and ornaments add subtle ambient life; bird animation remains the product priority.

- **Greeting on session start**: The plan makes greeting server-driven and keyed to absence length so the surface notices return without a generic announcement.

- **Offer interaction**: Offers provide a small, cooldown-bound nudge toward curiosity or boldness. The plan keeps them small so they do not become "click for change."

- **Settle interaction**: Settle ends the presence window and is server-tracked so settling on one device makes the aviary settled on another.

- **Field notebook**: The notebook is a sparse observational record in naturalist voice, generated only when something noteworthy happens or enough time has passed.

- **Top-bar access for offer, notebook, and settle**: The plan later says discoverability comes from the top bar's existence and natural exploration, not tutorial pushes.

- **Visit invitations**: Visit invitations are the allowed social surface: opt-in, revocable, expiring, render-only, and separated from social network mechanics.

- **Silent default with opt-in visit notification**: The rationale follows the non-goal against push notifications, email digests, and friend-visited notifications by default.

- **Multi-device sync**: The plan says the architecture itself is sync: both laptop and phone read the same canonical Postgres row and render the same aviary.

- **Screen-reader narration, captions, keyboard navigation, focus indicators, contrast, and reduced motion**: These make accessibility real in the shipped product, not a post-launch add-on.

- **Performance targets**: Time-to-first-bird, 60fps idle, and no memory growth protect the product's quiet first impression and long-running ambient use.

- **Browser support for last two major versions of Chrome, Safari, Firefox, and Edge**: NOT RECOVERABLE FROM PLAN

- **No native mobile apps**: The plan says the protocol should not include native-only fields or pre-shape the API for a future native client at the cost of clarity now.

- **No gamification**: The rationale is durability. The warehouse is forbidden from per-account dimensions so adding streaks later requires new infrastructure, not just a UI change.

- **No Tamagotchi mechanics**: Engine-level drift clamps to non-negative, so there is no negative-drift code path to flip on later.

- **No social network beyond visit invitations**: Visitor sessions are render-only, with no event emission, presence-time accounting, or visitor identity displayed inside the host aviary.

- **No marketing email, digests, or default notifications**: Email is reserved for transactional auth, export, deletion, opt-in visit notification, and account-state errors.

- **No stat panels, debug views, or numeric personality exposure**: The plan avoids showing raw personality values anywhere, including hidden UI, APIs, and internal admin.

### 2. Architecture

- **TypeScript everywhere**: The rationale is shared types across client and backend and simpler model parity at engineering velocity.

- **Preact chrome**: The plan chooses Preact for bundle size while retaining most of React's shape.

- **Custom Canvas2D renderer**: The scene stays a single composited surface, helping bundle size and keeping the canvas pure scene.

- **WebAudio custom DSP scheduler**: The rationale is procedural, varied call timing and synthesis without recorded audio.

- **Node 22 backend**: The plan chooses it for shared types with the client and simpler model parity at engineering velocity.

- **Postgres canonical state**: Relational integrity and a single store outweigh polyglot persistence at this size.

- **Redis hot caches and SSE fanout**: The plan ties Redis to ephemeral hot caches and pubsub fanout for streamed snapshot and narration updates.

- **Edge-hosted HTML shell and initial snapshot**: The edge path lets first paint happen without an origin round-trip and supports the time-to-first-bird budget.

- **One HTTP API gateway with colocated services in a monorepo**: The plan uses this for a small set of services sharing a common types package.

- **Five logical services: edge, api, sim, narrator, mailer**: The split separates rendering delivery, client API, simulation authority, prose generation, and transactional email.

- **Database role and column-level grant enforcement**: The rationale is a backstop: even a bug in `api` cannot write personality if the database refuses it.

- **`sim` as sole writer of personality state**: This forecloses last-write-wins corruption and makes all personality changes flow through the tick.

- **`narrator` separate from `sim`**: Narrator can generate notebook and narration prose while having no personality writes.

- **Client/server split at the snapshot boundary**: Above the snapshot, the server owns personality and mood; below it, the client owns positions, opacities, audio, and presentation.

- **Client events append-only, never state mutations**: This lets two devices emit events without overwriting each other.

- **Immutable server-to-client snapshots**: The plan uses immutable snapshots so the client applies rendered signals, not raw mutable personality state.

- **Rendered visual signals instead of raw vectors**: This prevents a future debug build or log file from resurrecting numeric personality state.

- **HTTP/JSON plus SSE, not WebSockets**: SSE is chosen because there is no high-rate client-to-server push and it is simpler behind the CDN.

- **Snapshot schema versioning**: Clients can refuse unknown major versions, preserving forward compatibility.

- **Visit route as separate frontend bundle**: The visitor bundle is smaller and lacks offer, settle, notebook, and settings code paths.

- **Visitor cookie separated from host cookie**: This avoids accidental host-state writes from a visitor frame in the same browser.

### 3. Data model

- **Synthetic UUIDs**: The plan uses synthetic IDs so email is not a foreign key or repeated identifier.

- **Email stored only on `accounts.email_encrypted`**: This limits where the user's address exists and keeps all other references on account UUID or lookup hash.

- **`email_lookup_hash`**: The hash supports sign-in lookup without storing plaintext email in other tables.

- **`local_timezone` on account**: This drives day/night and daily-ish cadence in the user's local day.

- **`aviaries.last_tick_at`**: The tick window uses it to decide which events are processed next.

- **`birds.personality_vector`**: The vector is the engine's canonical basis for drift, rendered signals, and durable bird identity.

- **`events` as one append-only log**: One event log ordered by `server_at` makes tick processing deterministic and avoids competing mutation channels.

- **30-day event retention**: The plan says this bounds storage because tick-relevant aggregation has already been applied to canonical state.

- **Notebook entries append-only from the user's perspective**: This preserves the user's observational record, except rare operations deletion for voice violations.

- **Visit sessions outside the events table**: Visitor activity cannot feed drift; the simulation never reads `visit_sessions`.

- **Per-device session tokens**: Device rows support device labels, last seen, and revocation from settings.

- **15-minute single-use magic links**: The plan uses short expiry and one-time use for replay protection and limited blast radius.

- **Complementary starter personality seeds**: The plan explicitly says this makes "two birds" read as two different birds from the start.

- **Presence computed by intersecting visibility, focus, and activity**: Keeping the conjunction on the server makes the signal honest and prevents a modified client from inflating presence with a single boolean.

- **Named calibration constants**: Constants are named so they can be tuned, versioned, and mapped to measurable drift targets.

- **Calibration changes behind feature gates**: The plan wants tuning experiments on cohorts rather than untracked production retuning.

### 4. API surface

- **`POST /api/auth/magic-link` always returns 204**: This avoids PII leaks through response timing or observable account existence.

- **`GET /api/auth/verify` redirects to matter-of-fact success or expired surfaces**: The rationale is voice consistency for auth and errors.

- **Account profile returns sanitized profile, devices, invitations, and recent visits**: The plan uses this for settings and account-state management without exposing raw sensitive state.

- **Email change keeps old address working until verified**: The rationale is account continuity during verification.

- **Account delete and recover endpoints**: The delete flow is recoverable during `pending_delete`; hard delete waits 30 days.

- **Snapshot endpoint cached at edge for 1 second**: This absorbs burst loads from one device opening multiple tabs.

- **SSE aviary stream**: It pushes tick-cadence and event-driven snapshot deltas while keeping operation simpler than WebSockets.

- **Batched event ingestion with server-stamped `server_at`**: Batching controls request overhead; server timestamps establish ordering.

- **Bird rename endpoint audited**: The plan says this supports the user's ability to recall renaming history.

- **Offer endpoint checked by server cooldown**: The next tick computes reaction, keeping the server authoritative.

- **Visitor snapshot filtered through visitor-render boundary**: Visitors see the host aviary render but no notebook, offers, settle, settings, account info, or events.

- **Narration SSE stream**: The stream delivers naturalist prose phrases for screen readers and captions when audio is off.

- **Export job emailed as signed download link**: The plan keeps export asynchronous and tied to the verified address.

### 5. Simulation engine design

- **Tick jobs partitioned by `aviary_id`**: The queue guarantee prevents two workers from ticking the same aviary simultaneously.

- **Transactional tick function**: If a worker crashes mid-tick, rollback and retry process the same window safely.

- **Seeded deterministic RNG**: Determinism is required for testing and safe retries.

- **Low-pass personality drift**: The plan uses slow accumulation so attention becomes visible over weeks instead of immediate change.

- **Monotonic-toward-expressive drift**: The `max(0, ...)` clamp enforces no negative drift and blocks Tamagotchi-style deterioration.

- **Trait saturation at 1.0**: A saturated bird stays saturated, making saturation a graceful end-state rather than a runaway.

- **Mood finite state machine**: Mood integrates time of day, weather, recent interactions, and personality while persisting unless a stochastic draw crosses a threshold.

- **Cross-bird mood coupling from start-of-tick state**: This prevents order-of-evaluation effects and makes wary mood spread slowly across ticks.

- **Per-species motif library**: The motif library is the species fingerprint; variation keeps calls alive while preserving recognizability.

- **Server schedules call windows; client synthesizes audio**: This keeps canonical timing in snapshots while leaving synthesis to the renderer.

- **Statistical chorus rather than forced chorus**: Forced chorus would announce; statistical overlap preserves the product principle.

- **Hourly narrator, not tick-bound notebook generation**: The plan wants sparse, jittered notebook entries rather than mechanical pacing.

- **Phrase-fragment grammar for notebook prose**: The grammar supports lowercase, specific naturalist voice without foundation-model drift.

- **No foundation model for prose in v1**: The plan cites determinism, voice consistency, privacy, and performance.

- **Day/night from user timezone and simple solar approximation**: This ties the aviary to the user's actual day with a slow yearly drift.

- **Weather as hidden Markov chain**: Weather is subtle, uncorrelated across aviaries, and reproducible by seed.

- **Cold-aviary 10-minute tick interval**: This still updates ambient time and mood while reducing compute for long-tail inactive aviaries.

- **Versioned personality-vector migration**: New traits can be backfilled without deleting existing values, preserving the day-one bird.

- **No regenerate or admin reset endpoint**: The plan avoids any path that would break durable bird identity.

### 6. Sync model

- **No client-to-client sync, CRDT, or eventual consistency layer**: The canonical Postgres row is the sync mechanism.

- **Snapshot refresh on visibility, suspend gaps, SSE reconnect, and heartbeat**: A device idle for hours does not keep a stale snapshot.

- **Names last-write-wins only because names are benign**: Personality and mood remain server-owned; name changes can be logged for user awareness.

- **Server-side settle flag and undo window**: Settle intentionally crosses devices, and undo can work from either device.

- **Server-side offer cooldown**: An offer from one device prevents another during the cooldown.

- **Snapshot ordering and SSE sequence numbers**: Older snapshots are dropped; sequence gaps trigger full resync.

### 7. Frontend rendering pipeline

- **Small Preact chrome plus single canvas**: The canvas stays pure scene and the chrome remains testable in isolation.

- **Top-bar chrome fading toward transparent on cursor stillness**: The rationale is to keep chrome quiet around the scene.

- **Snapshot interpolation**: The renderer makes server snapshots feel continuous between tick updates.

- **Idle micro-motion from oscillators**: Smooth bird motion is cheap and avoids per-frame physics.

- **Particle free-list for leaves and feathers**: This avoids allocations and supports the no-memory-growth invariant.

- **Bird animation prioritized over particles and palette updates**: The plan says birds are the product.

- **Renderer stops while hidden**: This follows the PRD while relying on server-side simulation to continue.

- **No wake-up animation on resume**: The first resumed frame should look as if the aviary had been running all along.

- **Quiet field load state instead of spinner**: The first frame is already part of the product mood and resolves into the aviary.

- **Reduced-motion cross-fade render path**: It gives reduced-motion users a calmer aviary, not a degraded one.

- **Compact SVG paths converted to render commands**: This supports species poses and palette swaps within the visual asset budget.

- **Greeting keyed to audio onset**: The visual and audio greeting stay in sync.

- **Greeting variation by absence length**: The plan uses absence length to decide glance, soft call, step forward, or longer call.

- **Empty-aviary quiet field and one-time fly-in**: The first adoption moment gets a soft onboarding animation; later loads place birds at perches.

### 8. Audio pipeline

- **One AudioContext per session**: Audio can suspend and resume with tab visibility as a single session resource.

- **Master bus, per-bird buses, and per-call voices**: This supports independent birds, listen-in mix, and chorus summing.

- **Voice pool of 24 voices**: Pooling prevents allocate-on-call and supports the no-memory-growth CI invariant.

- **Scheduling with `audioContext.currentTime`**: The plan uses sample-accurate timing instead of `setTimeout`.

- **Seeded synthesis-time jitter**: The same tick can sound the same across two devices while still varying tick-to-tick.

- **Listen-in gain ramps**: Gradual ramps avoid a hard-cut UI and preserve aviary ambience.

- **Chorus through independent buses**: Real-time variation makes a chorus a chorus, not stacked loops.

- **Phase-coherence dev check**: The check catches phase-cancelling artifacts in simultaneous calls.

- **Captions near the calling bird**: Captions are spatial, specific, and tied to the actual call.

- **WebAudio fallback to silence with captions**: This is graceful access while respecting the unconditional "no recorded audio" rule.

- **Volume and mute in accessibility settings**: Audio controls belong in matter-of-fact accessibility UI; muted aviaries default captions on to protect access.

- **Greeting skew check**: The plan verifies audio and visual onset stay within the named tolerance.

- **Audio test rig**: Spectral fingerprinting, same-call-twice detection, and recognizability review protect species identity and avoid looped audio.

### 9. Accessibility surfaces

- **Screen-reader narration as separate live prose stream**: The plan refuses automated ARIA state lists and uses naturalist prose.

- **Matter-of-fact ARIA labels for controls**: Controls are system surfaces, not aviary narration.

- **Canvas `role="img"` plus live narration region**: The canvas gets a short summary while the stream carries full prose.

- **Keyboard navigation through chrome and birds**: The plan provides non-pointer access to top bar, bird focus, listen-in, offer, and settle.

- **Canvas focus indicator computed against local pixels**: This keeps focus visible in bright and dim aviary states.

- **Captions toggle and WebAudio-driven captions default**: The rationale is access to calls when sound is unavailable or undesired.

- **Automatic and manual reduced motion**: The plan combines standard `prefers-reduced-motion` behavior with user agency.

- **WCAG AA contrast and automated assertions**: CI fails on contrast regression in chrome and text surfaces.

- **axe-core, Lighthouse, and manual screen-reader review**: The plan uses automated and manual checks to prevent regressions.

- **Prominent accessibility settings page**: The page explains toggles plainly because the user is configuring the system, not being noticed by the aviary.

### 10. Performance budgets and observability

- **Initial JS bundle at or below 2MB gzipped**: The plan ties this to first paint and keeps deliberate headroom for real build overhead.

- **Code-split settings, notebook, visits, export, and sign-in**: Non-initial surfaces load on demand to protect the initial bundle.

- **Time-to-first-bird at or below 500ms**: Edge shell, inline snapshot, critical CSS, quiet field, and parallel bundle loading all serve this target.

- **60fps idle on a five-year-old laptop**: Cheap oscillators, object pools, and no per-frame allocation protect ambient performance.

- **30-minute no-memory-growth invariant**: Voice pools, particle pools, virtualization, bounded workers, and small snapshot history prevent long-session growth.

- **Aggregate operational telemetry only**: The plan measures health without per-account, per-bird, or streak-style dimensions.

- **No individual retention curves or per-bird interaction dashboards**: The plan refuses the engagement features such metrics would create.

- **Logger PII redaction and no per-account labels**: Observability keeps operational visibility without turning traces or logs into identity stores.

- **Synthetic browsers for major flows**: Synthetic tests exercise cold load, sign-in, snapshot, listen-in, offer, settle, notebook, settings, and sign-out.

- **WebAudio error spike as ticket, not page**: Silence-with-captions is graceful, so audio failure is not treated like auth or API failure.

### 11. Privacy, security, and content audit

- **Tiered data classification**: The plan separates highly sensitive email/tokens, sensitive bird and notebook state, and operational aggregate metrics.

- **Tier 1 never exfiltrated to analytics**: Personality, mood, position, notebook, and event log remain out of analytics infrastructure.

- **Email decryption limited to three points**: Outbound email, export, and email-change verification are the only stated decryption cases.

- **Magic-link replay protection**: Atomic `consumed_at`, HMAC signing, and short expiry limit stolen-link risk.

- **Session cookie protections**: HttpOnly, Secure, SameSite=Lax, revocation, and CSRF tokens address stolen cookie and CSRF threats.

- **No foundation model for narration/notebook**: Determinism, voice consistency, privacy, and performance are the articulated reasons.

- **Notebook content audit**: Sampling checks voice quality and informs grammar tuning without rewriting entries post-hoc.

- **Visitor isolation**: Visitor cookies, endpoints, and bundles prevent visitors from reading host private surfaces or writing drift events.

- **Account export contents**: The export includes the user's account, bird records, notebook entries, and settings, while excluding event logs and tier-0 secrets.

- **30-day soft delete before hard delete**: The pending-delete state gives a recovery option; hard delete removes account-owned rows.

- **No public aviaries or explore endpoint**: The plan avoids mass scraping by keeping visitor access tokenized and individual.

- **No browser fingerprinting for shared visit links**: The plan rejects that mitigation because it would require new tracking infrastructure.

### 12. Rollout

- **Internal alpha**: The rationale is tick correctness, snapshot accuracy, drift-calibration smoke tests, and synthetic browsers.

- **Closed beta**: The plan uses invite-only cohorts, compressed third-bird timing, daily voice review, and drift tuning.

- **Open beta gradual rollout**: Feature gates allow 10% to 100% rollout while production bird cadence is restored.

- **General availability**: Feature gates are removed and the seven-bird cap plus age-gate schedule become the stable shape.

- **Bird-per-aviary ramp**: The plan says the compounding interval shape is stable while exact days can flex after beta observation.

- **Feature gates in API config**: Client respects gates in snapshot response, enabling controlled rollout by surface.

- **Blue/green and rolling deploys**: The plan uses deploy patterns and queue absorption to keep services safe during release.

- **Backwards-compatible migrations for one deploy cycle**: Previous app versions may still be running, so migrations must tolerate overlap.

- **First-time sign-in flow with two birds and naming**: The empty field, fly-in, calls, and naming surface introduce the aviary without tutorializing it.

- **No tutorial sequence or pushed tooltips**: Discoverability comes from the top bar and natural exploration.

- **Release-day WebAudio, edge snapshot, sim capacity, and email deliverability risks**: The plan names these because they threaten core access, first paint, aliveness, and sign-in.

### 13. Risks and mitigations

- **Drift instruments**: Aggregate trait-delta histograms reveal whether the population is drifting in expected ranges without per-account dimensions.

- **Drift calibration controls**: Tunable config, alpha/beta cohorts, explicit consent for A/B-able alpha values, and hard ceilings mitigate too-fast or too-slow drift.

- **Sync consistency checks**: Replaying event logs against canonical state catches duplicate event processing or direct personality writes.

- **Chaos tests with overlapping clients**: These prove canonical state stays correct under competing device event orderings.

- **Audio listening reviews and same-call tests**: These guard against mechanical, samey, or off-pitch calls that would break the spell.

- **Accessibility regression ownership**: Reduced motion is owned by the same team as standard mode, and screen-reader checks happen before launch and at chrome changes.

- **Voice linter and string checklist**: The plan blocks announcement phrases, streak language, achievements, badges, levels, and scores from entering user-visible strings.

- **Privacy regression tests**: Logger redaction, ETL validation, access audits, and threat-model exercises protect against per-bird and email leakage.

- **Sim worker autoscaling and cold-aviary interval**: These protect tick timeliness during population spikes.

- **Mailer deliverability setup and monitoring**: Dedicated subdomain, SPF/DKIM/DMARC, provider choice, help link, and domain monitoring protect sign-in.

- **Personality-vector migration testing, backups, and delta alarms**: These protect against vector corruption or reset.

- **Revoked visitor link checks on every snapshot pull**: Revocation takes immediate database effect and no caching window preserves access.

### 14. Cross-cutting concerns

- **Monorepo structure split by apps and packages**: Shared packages support types, audio DSP, rendering, narration grammar, and calibration across services.

- **Engine and surface product teams split at the snapshot boundary**: Engine owns above the snapshot; surface owns below it.

- **Audio specialist, designer, and embedded accessibility reviewer**: These roles match the plan's load-bearing audio, voice, visual, and accessibility requirements.

- **Unit, integration, end-to-end, performance, accessibility, audio, and voice tests**: The test strategy mirrors deterministic engine behavior and user-facing surfaces.

- **Config package for calibration and voice thresholds**: Values are versioned, reviewed, and changed by deploy rather than runtime hot config.

- **English-only v1**: Naturalist voice is load-bearing, and machine translation would not preserve it.

- **Chrome matter-of-fact localization path**: System surfaces use i18n keys so v2 localization can happen without architectural change.

- **Unsupported older-browser surface and no old compatibility paths**: NOT RECOVERABLE FROM PLAN

- **Mobile web no-cropping layout**: The plan preserves every bird in frame on narrow viewports.

- **iOS WebAudio user-gesture handling**: The first-frame silence-with-captions path covers the moment before any tap.

- **Data retention windows**: Events are operational and short-lived; notebook entries are the user's record; aggregate telemetry rolls for 13 months.

- **Backups and quarterly restore drills**: These verify accounts can sign in and birds are intact after restoration.

- **Internal admin after support request only**: It helps engineering inspect operational state while avoiding raw personality numbers.

- **Support link without in-product chat or AI support agent**: NOT RECOVERABLE FROM PLAN

### 15. Open questions and defensible calls

- **Frontend framework call**: Preact and Canvas2D are chosen for bundle budget and a single composited scene surface.

- **Personality-vector JSONB call**: The trait set is small, evolution is slow, and schema versioning is cheaper than a separate table.

- **Postgres-row partition queue call**: It can be replaced later, but starting with Postgres avoids early infrastructure.

- **SSE call**: It is simpler than WebSockets through the CDN and bandwidth is small.

- **Mood reset at 04:00 local time**: The plan calls it a daily-ish cadence at a low-activity hour.

- **Notebook frequency of one entry every 2-4 days**: The plan says this is rare enough to feel observational.

- **Visitor cookie scope `/visit/*`**: This prevents accidental host-state writes.

- **Audio voice pool size of 24**: Six birds times four overlap windows is the stated worst case, and the pool prevents allocation.

- **Inline initial snapshot for visitors**: This serves the time-to-first-bird budget.

- **Event log retention of 30 days**: It is operationally sufficient because personality is already in canonical state.

- **WebAudio fallback of silence with captions**: This preserves the no-recorded-audio constraint and the bundle budget.

- **No raw personality numbers in internal admin**: The plan says this avoids resurfacing risk.

- **First-time naming UI in matter-of-fact voice**: Naming is configuring, not noticing.

- **Reduced-motion default honoring `prefers-reduced-motion`**: The plan combines standard practice with user agency.
