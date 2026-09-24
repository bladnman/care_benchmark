## System-level intent

- **Keep the product's affective promises true at the architecture level.** This is stated in the executive summary as the central engineering problem: "keeping the product's affective promises true at the architecture level." The plan turns product rules into "invariants" with "code, schema, database grant, CI gate, or review gate" enforcement, and repeats that privacy and anti-gamification are "architecture, not policy."
- **Canonical state belongs to the server; clients handle presentation.** The "How to read this plan" section defines "Canonical" as "server-authored state" and "Presentation" as client-derived state that "never persists, and never sends back as truth." This shows up again in "The server is the only writer of bird state," the writer-ownership matrix, and the sync principle that clients "read snapshots and append events."
- **The aviary should feel already alive, not launched by the user.** The plan says snapshots carry "where each bird is and will be" so the first frame can show birds "already mid-action." The first-frame invariant forbids "spinner, fade-from-static, or entry sequence," and the no-snapshot path has birds "emerge from the foliage edges" rather than fade in.
- **Presence is the relationship input, but it must be honest and counted once.** The summary says "Presence is measured honestly and counted once." The presence model uses visible, focused, recent trusted activity, server-side clamping, cross-device union, and daily concavity to prevent background tabs, multiple devices, kiosks, and automation from inflating drift.
- **Change is slow, monotonic, and expressive rather than need-based.** The plan repeatedly says hidden personalities drift "slowly and only upward," "no session moves a trait visibly," and "Relationship deepening, not saturating." It excludes Tamagotchi mechanics because "No state variable can represent need or suffering."
- **Birds must remain individual and continuous.** The plan protects identity with immutable `bird_id`, fixed `voice_seed` and `plumage_seed`, per-bird ceilings, no re-creating birds, no beta reset, and "Identity continuity." It says two heavily watched birds should converge to "different expressive characters" rather than all saturating.
- **Aliveness is procedural, not canned.** The executive summary says "Everything the user perceives as alive is generated, not stored." The plan applies this to AudioWorklet calls, mood-keyed procedural rigs, fresh greeting variation, captions, narration, notebook prose, "no audio files," and "no canned animations."
- **Quietness is a product value.** The plan bans "announcement surfaces," "welcome text, toasts, banners, badges," success messages, push/email about the aviary, and no-text loading for at least 8 seconds. Newcomers are framed as "a noticing, not a notification."
- **The product voice is split deliberately.** The plan defines "Two voice registers: naturalist for product surfaces, matter-of-fact for system surfaces." The surface inventory assigns every surface to a register, and the content system enforces separate naturalist and system catalogs.
- **Accessibility is part of the product, not a fallback.** Scope says accessibility "ships at v1, not after." The accessibility section says every user gets the "actual product in their register, not a stripped fallback," and reduced motion is "a designed register, not motion turned off."
- **Privacy boundaries constrain measurement and tuning.** The plan says the telemetry plane has "no route or credentials into Simulation DB or Identity DB," and the observability section lists many things it will "never" measure. Calibration therefore uses a harness, staging dogfood, qualitative channels, and consent-gated support access instead of production aggregates.
- **Determinism and testability are part of the product shape.** The shared core uses integer PRNG, fixed-point arithmetic, named seed streams, engine versions, and no DOM or Node APIs. The plan repeatedly relies on property tests, CI gates, visual regression, bundle checks, voice lint, contrast gates, and launch gates.
- **Operational changes should be conservative.** Parameters are server-side, versioned, and reviewed. Drift gains are set at the "slow edge of the calibration band" because "slow is fixable, fast is not," and engine changes after beta must respect monotonic drift and identity continuity.

## Per-feature whys

### How to read this plan

- **Invariants with enforcement mechanisms:** The plan uses them because "If an invariant can only be kept by people being careful, the plan treats that as a gap and adds a mechanism."
- **Decision log:** The plan uses it because where the PRD is "ambiguous or self-contradictory," it "makes a call and records it."
- **Tunable parameters:** Parameters live in reviewed server-side config so drift gains, cooldowns, cadence, and budgets can change without making unreviewed behavior changes.
- **Canonical vs. presentation split:** The rationale is to keep server-authored truth separate from local rendering, so presentation state is "never sent back as truth."

### Executive summary and invariants

- **Server-only writer of bird state:** Multi-device sync "falls out of this" because there is "nothing to merge," and last-write-wins on personality cannot happen.
- **Snapshot timeline rather than only state:** This lets any client compute "now" and paint birds "already mid-action," making "no entry animation" built rather than promised.
- **Procedural living surfaces:** The plan uses generated calls, motion, greetings, captions, narration, and notebook prose so there are "no audio files and no canned animations."
- **Presence measurement:** The three-signal conjunction plus clamping and cross-device union prevents background tabs and two open devices from double-counting.
- **Privacy and anti-gamification architecture:** Email in one encrypted table, synthetic UUIDs, telemetry isolation, allowlisted schemas, and no toast/badge/counter components make the promise structural.
- **No client-submitted trait values:** This prevents client code from writing personality and makes the tick the only path for personality change.
- **Non-decreasing traits:** The rationale is that personality is monotonic by construction; the trigger makes a reset fail loudly.
- **Trait values not shown or sent:** The plan keeps hidden personality hidden even from curious users reading payloads; export is handled as an opaque exception.
- **No announcement surfaces:** The plan prevents welcome text, toasts, badges, counters, streaks, and push/email from becoming product patterns.
- **First frame in motion:** It preserves the "already running" illusion and avoids spinner, fade-from-static, or entry sequence.
- **Stable bird identity:** `bird_id` is immutable so species-pool, rig, and grammar changes never re-create or swap birds.
- **Email only in one encrypted place:** This keeps email out of identifiers, keys, logs, and metric labels.
- **No aggregation of interaction state:** This preserves the privacy promise; telemetry and third parties never receive per-bird or per-account interaction state.
- **Procedural calls only:** The plan enforces no recorded-audio path; if WebAudio is unavailable, the fallback is silence plus captions.
- **Visitor read-only behavior:** Visitors cannot create presence or interaction events because they must never alter host state.
- **Two voice registers:** Naturalist copy belongs to product surfaces and matter-of-fact copy belongs to system surfaces, enforced by catalogs and lint.

### Scope

- **Web-only aviary:** The plan excludes native apps because the API is "designed for browsers only" and avoids "store review cycles."
- **One aviary per account:** NOT RECOVERABLE FROM PLAN
- **Two starter birds:** NOT RECOVERABLE FROM PLAN
- **Pool of 6 species:** NOT RECOVERABLE FROM PLAN
- **Cap of 7 birds:** The plan ties the cap to bounded rendering, audio recognizability, snapshot size, and a ramp that validates each bird count before raising the adoption ceiling.
- **Newcomers by aviary age:** Aviary age avoids visit-count or catalog mechanics; D-18 says the offer should be "a noticing, not a notification."
- **No catalog for adoption:** The plan presents arrivals in-flow so adoption does not become a shopping or collection surface.
- **Naming and renaming birds:** Names are "the user's labels," while identity remains `bird_id`; rendering notebook names at read time avoids "who's pip?"
- **Server-side tick:** It advances personality, mood, perch placement, social events, weather, and notebook "whether or not any client is connected," preserving the aviary as ongoing.
- **Hidden personality vector:** It lets traits drive expression while keeping values out of product surfaces and client payloads.
- **Slow monotonic personality drift:** The rationale is relationship deepening that is never visible in a single session and never punishes absence.
- **Mood set with `roosting`:** `roosting` exists because the layout requires sleeping birds, and the internal name avoids collision with the settle gesture.
- **Procedural calls, chorus, call-and-response, and mood contagion:** The plan uses these to make birds feel alive and social without recorded audio or canned cues.
- **Single horizontal scene and three perch zones:** NOT RECOVERABLE FROM PLAN
- **Local-time day/night:** It anchors every device to "the same aviary in the same mood" without geolocation.
- **Rare weather:** Synthetic weather uses no location data and is "never assertive."
- **Responsive layout that never crops a bird:** The rationale is a safe band, layout solver tests, and preserving bird visibility across viewports.
- **Return greeting:** The plan calls greeting "the anchor moment"; it must honor boldness, mood, and absence length while never being canned.
- **Idle presence:** Presence feeds drift as the user's attention signal, but is clamped, unioned, and concave so it is counted honestly.
- **Listen-in:** It is "a strong signal of attention" for warmth and vocal drift, while the mix changes rather than the bird's behavior.
- **Offer with cooldown:** Offers need a visible reaction within a second; cooldown is "Functional, not punitive," so birds glance and no timer UI exists.
- **Seed, song fragment, and still pool offer kinds:** NOT RECOVERABLE FROM PLAN
- **Settle with 5 s undo:** Settle gives a quiet goodbye, closes the presence window cleanly, and undo prevents accidental commitment before the event is sent.
- **Field notebook:** The notebook records "observations of the aviary, never of the user," sparsely and read-only, so it does not become an activity log.
- **Email magic-link sign-in:** The plan chooses magic-link only; fragment tokens and POST consumption prevent scanner prefetch, silent token burn, and log leakage.
- **Unified sign-up/sign-in:** The rationale is "Magic-link only; no separate registration."
- **Per-device sessions and revocation:** The plan supports device-list accuracy and security, with immediate API revocation and edge propagation.
- **Verified email change:** The new address must confirm before swapping; the old address notice is "security hygiene."
- **JSON export by emailed download link:** The plan supports portability while keeping personality as an opaque, checksummed encoding, not readable numbers.
- **Soft delete then hard delete:** The 30-day window allows "I changed my mind"; hard delete removes account rows and crypto-shreds PII while backup residue expires.
- **Multi-device sync:** It is a property of server-owned canonical state: clients only read snapshots and append events.
- **Visit invitations:** Sharing is "deliberate, bounded," per-invite, one-time, read-only, revocable, and not a social network.
- **Visit log:** The rationale is "Host transparency" with "Visitor data minimized."
- **Opt-in visit notifications:** Email is used because there is no push in v1 and in-product badges are forbidden; notifications carry no aviary content.
- **Screen-reader narration:** It gives the actual product in naturalist prose with slow cadence instead of a stripped fallback.
- **Reduced-motion rendering:** It is "its own designed register," preserving aliveness through key poses and cross-fades.
- **Call captions:** Captions are generated from the same score as audio so they match what was actually heard.
- **Keyboard navigation and visible focus:** The plan makes canvas birds reachable through DOM focus targets and keeps focus visible across lighting states.
- **WCAG AA contrast:** Contrast gates ensure user copy and focus indicators remain legible in dawn, noon, dusk, night, settled, and rain states.
- **Initial JS cap:** The plan uses it to protect the first-bird performance budget.
- **First bird visible under 500 ms:** This is the moment the "already running" illusion matters most on a return visit.
- **60 fps idle and no 30-minute memory growth:** The rationale is long-running ambient use without leaks or degraded aliveness.
- **Aggregate-only RUM and synthetic monitoring:** Observability is limited by the telemetry boundary and privacy promise.
- **Supported browsers and unsupported surface:** Last-two-majors support lets the browser target stay safe; older browsers get matter-of-fact unsupported copy.

### Explicit exclusions

- **Native apps excluded:** The rationale is no native protocol concessions and no store review cycles.
- **Gamification excluded:** The plan removes schemas, APIs, copy, components, and checklist paths for achievements, streaks, levels, scores, badges, XP, counters, calendars, and milestones.
- **Tamagotchi mechanics excluded:** There is no need, suffering, death, hunger, distress, or decaying happiness; absence-only input cannot raise wary occupancy above baseline.
- **Social network surfaces excluded:** Visitor tokens can read only one host snapshot; there are no profiles, follows, feeds, comments, chat, avatars, mutual visits, or cross-account queries.
- **Push notifications and aviary-related email excluded:** The rationale is the no-announcement promise.
- **Payments, shared aviaries, multi-aviary accounts, customizable scenes excluded:** NOT RECOVERABLE FROM PLAN
- **Showing personality numbers excluded:** The rationale is INV-03; production support access never presents trait values in product UI.
- **Recorded audio excluded:** The rationale is procedural calls and no audio-asset path.
- **SSO and passwords excluded:** NOT RECOVERABLE FROM PLAN
- **User-controlled bird placement excluded:** Perch position is tick-owned and there is no placement API.
- **Localization excluded in v1:** The naturalist grammar is authored in English, though copy is externalized so localization remains possible later.

### Surface inventory and voice register

- **Aviary scene with no text:** The rationale is no labels, tooltips, buttons, badges, or UI inside the scene.
- **Return-greeting as visual/audio/narration only:** It keeps greeting as behavior rather than a welcome surface.
- **Offer tray in naturalist voice:** Item names like "a seed," "a song fragment," and "a still pool" keep offers in product voice.
- **Settle label with system-style accessible hint:** The visible label stays naturalist, while the undo affordance is stated plainly for accessibility.
- **Field Notebook panel:** The proper UI label is allowed, while entries are lowercase naturalist prose.
- **Loading quiet field:** No text appears for at least 8 seconds; matter-of-fact copy appears only on real failure.
- **Account, visits, accessibility, and auth surfaces in matter-of-fact voice:** These are system surfaces rather than aviary behavior.
- **Error/sync/offline lines near the top bar:** Errors need clarity without using toast or announcement patterns.
- **All emails in matter-of-fact voice:** The plan forbids aviary content in email.

### Architecture

- **Edge worker:** It streams HTML with critical CSS and an inline snapshot without a DB call so the first frame can be fast and already populated.
- **API service:** It is stateless and owns auth, events, offers, notebook reads, naming, adoption, visits, export, and deletion because these are writes or account operations outside the tick.
- **Tick workers:** They own state progression because personality, mood, perches, weather, attunement, candidates, and snapshots must remain canonical.
- **Notebook writer:** It separates sparse prose selection from the minute tick while still rendering through the shared grammar.
- **Adoption scheduler:** It creates newcomer windows from aviary age so adoption is not driven by engagement metrics.
- **Mailer:** It is the only component with decrypt rights because email must live in exactly one encrypted place.
- **Export worker:** It builds an encrypted downloadable artifact with a 7-day lifecycle and sends the link through the mailer.
- **Deletion worker:** It hard-deletes after the soft-delete window and crypto-shreds per-account keys.
- **Telemetry plane:** Its separate network segment and credentials preserve the "no route" privacy boundary.
- **Writer-ownership matrix:** It makes conflicts unreachable by assigning each state category one writer.
- **TypeScript end to end:** Server and client share deterministic modules such as planner, caption generator, grammar engine, timeline evaluator, PRNG, and species catalog.
- **PostgreSQL system of record:** Separate identity and simulation storage supports privacy isolation and reliable canonical state.
- **Redis and Edge KV snapshots:** They support hot snapshots, rate limits, cooldowns, edge rendering, and revocation checks.
- **No framework in the scene path:** The scene path stays small and fast; Preact is limited to top bar and lazy panels.
- **Self-hosted error collection with scrubbing:** The plan avoids SaaS error tracking that receives user payloads.
- **`aviary-core`:** A pure shared package keeps canonical decisions reproducible across tick workers, API, notebook writer, and client.
- **Integer PRNG and fixed-point arithmetic:** The reason is cross-engine determinism; `Math.random` and transcendental functions can differ across engines.
- **Named seed streams:** Adding a stream never perturbs existing streams.
- **Server-only drift subpath:** It keeps drift math out of the browser.
- **Snapshot schema versioning:** Clients accepting N and N-1 makes rolling deploys safe.
- **Staging simulated clock:** It tests newcomers, saturation, and 7-bird scenes months before real aviaries reach them.
- **Production without simulated clock or debug surfaces:** This protects production promises from research and debug-only affordances.

### Data model

- **UUID primary keys and `account_id` link:** Synthetic IDs keep identity separate from simulation and telemetry.
- **Fixed-point trait and engine values:** They make drift arithmetic exact, deterministic, and monotonic-checkable.
- **Email normalization without plus/dot stripping:** The plan calls stripping provider-specific and error-prone.
- **Account preferences without usage history:** Prefs hold accessibility, sound, and visit-notification settings, not behavior history.
- **Simulation fields for `created_at`:** Aviary age drives adoption and never visit count.
- **Append-only interaction events:** They provide idempotent inputs without letting clients write state directly.
- **Presence intervals:** They normalize clamped client events for server-side cross-device union.
- **Greeting history:** It enables notebook observations like "first time this week."
- **Observation candidates:** They keep notebook material as typed aviary facts before sparse rendering.
- **Adoption offers:** They support lingering, welcomed, and departed newcomer windows without counters or penalties.
- **Visit invitations and sessions:** They bind one-time links, support revocation, and approximate visit duration for host transparency.
- **Species catalog:** It keeps common names, rig parameters, palettes, perch preferences, circadian profile, call grammar references, and trait bases versioned and additive so birds are never remapped.
- **Call grammars:** They define motifs for procedural audio rather than audio files.
- **Prose grammar:** It keeps notebook, narration, captions, greeting narration, and offer-reaction narration in the authored voice.
- **Offer library:** Song fragments are motif scores, not audio, preserving procedural sound.
- **Retention for personality, mood, birds, notebook:** These last for the life of the account because they are "canonical relationship state."
- **Retention for interaction events and presence intervals:** They are kept only to drive the user's own simulation, notebook lookbacks, and replay window.
- **Retention for personality ledger and checkpoints:** They exist for integrity audit, repair, and point-in-time recovery, not runtime use.
- **Retention for visit invitations and log:** The rationale is host transparency with minimized visitor data.
- **Retention for logs and backups:** It bounds residual UUID mentions and backup copies after hard delete.
- **Personality vector defenses:** The plan treats vector loss as "the worst possible failure" and layers single writer, trigger, CAS, ledger, checkpoints, nightly integrity, migration discipline, and identity continuity.

### API surface

- **Cookie-based host and visitor auth:** Separate `__Host-pa_session`, edge, and visitor tokens let host and visitor capabilities remain distinct.
- **CSRF protections:** SameSite, client header, and Origin checks guard state-changing requests.
- **Client-generated idempotency IDs:** Retries return the original result and cannot duplicate writes.
- **Error codes mapped on the client:** Server prose never reaches product surfaces, preserving the register split.
- **Rate limits:** They constrain magic-link and invitation abuse.
- **Snapshot caching and ETags:** They keep snapshots private and let polling avoid unnecessary payloads.
- **Endpoint set without trait, mood, or perch writes:** This enforces that clients cannot submit canonical bird state.
- **No host visit history endpoint:** The plan avoids visit-frequency displays and streak-like metrics.
- **Snapshot target size and tick compilation:** The snapshot is small, fast, and canonical rather than assembled per request.
- **Host-only greeting fields:** `last_presence_end_at` and `greeting_plan` support greetings without exposing them to visitors.
- **Quantized `profile` and `look`:** These render behavior without leaking trait values or providing an invertible mapping.
- **Visitor snapshot strips host-private fields:** Visitors see the same visual state but cannot act on host state.
- **`session_start` event:** It records greeting outcome using the canonical `plan_id`, not the client claim.
- **`presence_interval` event:** Server clamping prevents old or inflated client intervals.
- **Listen-in events:** They are paired per session and count only where they intersect with presence.
- **Settle event:** It is sent after undo or pagehide and closes the presence window.
- **Synchronous offer resolution:** The API resolves visible reactions within a second, while the tick applies slower consequences later.
- **Offer item immediate presentation:** Seed, pool, or song fragment appears immediately to mask the round trip.
- **Offer response included in next snapshot:** A second device sees the same item and birds at it.
- **Offer failure behavior:** The item fades normally and only a quiet system line appears after repeated failures.
- **Visit token in URL fragment:** Tokens never reach server logs, the edge, or `Referer` headers, and GET-only link scanners cannot consume them.
- **Invite email identifies host by email:** There are no profiles or display names; the email is verified and no free-text message prevents spam or phishing text.
- **Visitor account not required:** Sharing remains read-only and independent from any host session.
- **Magic links independently single-use:** Requesting a new link does not invalidate outstanding ones; each expires and consumes atomically.
- **Session timeout discards queued events:** Losing minutes is harmless under slow drift and avoids cross-session attribution.
- **Deletion copy on signed-in pages:** It presents a matter-of-fact recovery action during the soft-delete window.
- **Complete email template set:** CI fails if any unlisted template is registered so no aviary-content email appears later.

### Simulation engine

- **Every-aviary 60 s tick:** The PRD states the tick runs regardless of connection, and pure `step()` allows equivalent batching later.
- **Bucketed work units and leased workers:** This spreads load and lets workers claim batches safely.
- **CAS tick writes:** They prevent double application and stale worker commits.
- **Snapshot publication after commit:** Publishing is idempotent because the snapshot is a pure function of committed state and tick index.
- **`step()` as pure function:** No I/O and deterministic seeds make simulation testable, replayable, and safe for catch-up.
- **Absence as normal case:** With no events, the aviary still advances reservoirs, attunement decay, mood hazards, weather, and timeline.
- **Separate sub-model modules:** Property tests can target presence, drift, attunement, mood, perch, social, weather, greeting, adoption, and candidates.
- **Presence chunks every 30 s:** They capture continuous presence while bounding event volume.
- **Cross-device presence union:** "Attention is per person, not per device"; summing would inflate drift.
- **Audible milliseconds only for vocal modifier:** This honors mute without punishing mute or making it negative.
- **Daily concavity:** It bounds all-day activity, jigglers, and automation so calibration holds for every usage pattern.
- **Reservoir-and-release drift:** It directly expresses low-pass filtering, continuing during absence, and monotonic construction.
- **No visible single session:** Concave inflow, two-day release, and daily cap keep changes below the perceptibility threshold.
- **Per-bird ceilings:** Individuality survives years because heavily watched birds converge to different expressive characters.
- **Launch drift values at slow edge:** Drift cannot be undone; "slow is fixable, fast is not."
- **Attunement:** It reconciles "quieter after absence" with traits never moving down; it affects expression only and cannot produce distress.
- **Mood Markov process:** Personality, circadian context, recent interactions, rain, wind, and alarm impulses shape mood without direct user punishment.
- **Daily-ish dawn relaxation:** It honors "Mood resets daily-ish" and "never snaps."
- **Perch placement:** Front/back/drowsy/adjacent weights make boldness, attunement, wary, roosting, and warmth visible in space.
- **180 s timeline:** It lets clients render current and near-future action and makes missed polls invisible.
- **Conflict-free slots:** Reservation prevents two birds targeting the same slot.
- **Call-and-response:** Warmth and mood make bird-to-bird response likely but not scripted.
- **Chorus events:** They emerge from multiple calling birds and are scheduled as clusters, never one stacked cue.
- **Alarm calls:** They are rare, ambient, and never tied to user behavior or absence.
- **Synthetic local day/night:** It gives every device the same light without geolocation.
- **Timezone easing:** Travel changes do not snap the sun curve or moods.
- **Synthetic weather:** It avoids location data and assertive weather.
- **Server greeting planner:** It is canonical so the notebook can truthfully describe greeting order.
- **Client greeting realization:** Fresh session randomness keeps greetings from being canned while the server keeps order canonical.
- **At least one greeter:** Even at low attunement, one bird always greets.
- **No visitor greeting:** Visitors must never generate presence or interaction events.
- **Starter adoption species selection:** Two distinct diurnal species are chosen for acoustic contrast; the nocturnal species is not a starter because "a first encounter with a sleeping bird is wrong."
- **Starter fly-in only once:** It is the only entrance because later loads must not have entry animation.
- **Newcomer lingering:** It turns adoption into "Arrival as a noticing" with no toast, badge, or prompt.
- **Newcomer not welcomed:** Departure creates no penalty, counter, or missed state.
- **Notebook candidates:** They are typed aviary facts so entries do not observe the user.
- **Notebook allowlist:** It rules out visits, session counts, durations, return frequency, days away, and "you."
- **Notebook sparsity budget:** Time-based cadence prevents heavy users from getting more entries.
- **Notebook during absence:** The writer keeps running because the aviary kept going, but never frames absence.
- **Read-only notebook:** No edit, delete, or annotate endpoints exist, keeping it an observation record.
- **Server/client audio division:** The server owns canonical multi-bird timing; the client owns sound realization, keeping audio procedural.
- **Calibration harness:** Production drift and mood data cannot be aggregated, so calibration is done offline and in staging.
- **Staging dogfood cohort:** It provides real-time drift and fatigue evidence in a consented research environment that never migrates to production.

### Sync model

- **One canonical record per aviary:** It makes the simulation DB authoritative and bird-state conflicts unreachable.
- **Clients read snapshots and append events:** There is no client-to-client channel, no client-side persistence of bird state, and nothing to merge.
- **Edge-inlined navigation snapshot:** No fetch is needed for the first frame.
- **Visible polling with jitter:** It keeps snapshots fresh without every client polling at the same instant.
- **Fetch on visibility and long frame gaps:** It handles return, suspend, and OS sleep, then reconciles naturally.
- **Hold mode after stale snapshots:** Birds continue perching, idling, and calling; "the aviary never freezes."
- **In-memory event queue:** Persisting the queue would add storage and privacy surface; losing minutes is harmless under slow drift.
- **Flush on listen-in, settle, session start, hidden, and pagehide:** These preserve important session boundaries.
- **Retry with idempotency:** Network retries cannot duplicate events.
- **Conflict-prevention catalogue:** Scenarios such as two devices, duplicate events, zombie workers, concurrent offers, renames, and magic-link replay are made safe by unioning, append-only events, unique IDs, CAS, row locks, and optimistic concurrency.
- **Clock offset from `server_now`:** Timeline evaluation uses server time instead of trusting the client clock.
- **Canonical timezone:** Every device shows "the same aviary in the same mood."
- **Suspend/resume reconciliation:** Birds fly or cross-fade to canonical perches and "never teleport."
- **Offline and failure behavior:** The plan keeps the scene quiet, retries, shows matter-of-fact copy only on real failures, and never shows success toasts.

### Frontend rendering pipeline

- **Boot HTML with critical CSS and inline snapshot:** Even pre-JS pixels are the quiet field, and first bird rendering does not wait on a fetch.
- **Small `boot.js`:** It contains only the pieces needed to draw birds mid-action in the first frame.
- **Hydrating `aviary.js` after paint:** Audio, interactions, presence, poller, narration, and captions do not block first bird visibility.
- **Audio starts when allowed:** If audible start is allowed, fade-in reads as ambient sound already in progress rather than an entrance.
- **Lazy chunks:** Notebook, settings, visits, adoption, and errors load after the first experience is already alive.
- **No-snapshot quiet field:** It avoids spinner and fade-from-nothing, then birds emerge as if "just out of view."
- **Logical scene and safe band:** They let birds stay in frame across narrow, portrait, and wide viewports.
- **Subtle parallax:** It avoids a "parallax-heavy feel" and pointer-motion coupling.
- **Calm naturalist palette:** The visual language uses soft blues, greens, warm browns, and muted ochre while engineering supplies contrast tests.
- **Canvas DPR cap and debounced resize:** They balance sharpness, performance, and stable repositioning.
- **Procedural 2D rig:** It makes individual birds of the same species distinct through seeds, cached patterns, and species-specific movement.
- **Idle micro-motion aliveness rules:** Breathing never stops, loops are not periodic, and birds are never motionless except for breathing and blinks.
- **Allocation discipline:** Preallocated arrays and no per-frame object creation support the 30-minute memory test.
- **Timeline reconciliation:** Absolute server time and catch-up flights make snapshot updates smooth.
- **Mood transitions:** Behavior weights blend so posture never snaps.
- **Listen-in interaction:** Clicking, tapping, or keyboard activation changes the mix and attention without visual chrome.
- **Offer tray:** It stays small, keyboard navigable, naturalist, and only adds "welcome the newcomer" when present.
- **Offer choreography:** The server supplies reaction class and seed; the client makes procedural paths, hops, and calls.
- **Settle control:** It creates a local evening palette, audio duck, drowsy posture bias, and one soft acknowledgement call as goodbye.
- **Re-engage rule:** Pointer movement alone does not count because the user may be reaching for the close button.
- **Presence tracker:** It implements visible, focused, recent trusted activity, plus touch-friendly `pointerdown`, to avoid background and device bias.
- **Exactly five top-bar controls:** D-01 keeps settle while preserving sparse chrome and no badges, dots, counts, avatars, or status indicators.
- **Top-bar fade:** It keeps the scene visually quiet but stays visible for focus, panels, touch, and accessibility preference.
- **Panels over running scene:** Birds and calls continue, preserving the aviary while system UI is open.
- **Virtualized notebook list:** It supports unlimited scrollback without DOM growth.
- **Reduced-motion rendering:** Key poses and cross-fades preserve the product register while removing parallax, drifting particles, and rain streaks.
- **Frame-budget governor:** It degrades DPR, particles, parallax, and grading before bird rig fidelity or call timing.
- **Visitor mode:** It uses the same renderer but removes greeting, presence, listen-in, offers, settle, notebook, and account panel so visits are read-only and not show-off mode.

### Audio pipeline

- **Fixed audio graph:** A bounded node count prevents per-call allocation and protects performance.
- **Procedural reverb impulse:** It gives open-air space without audio files.
- **AudioWorklet sinusoidal voice:** The plan says sinusoidal modeling fits birdsong, which is dominated by near-pure tones with rapid modulation.
- **Score messages to preallocated syllable slots:** Worklet processing does no allocation.
- **Soft clipping, ceilings, and reset on bad values:** They prevent NaN, clipping, and runaway audio.
- **Per-species call grammars:** They supply distinctive motifs for wren-like, warbler-like, finch-like, thrush-like, dove-like, and nightjar-like calls.
- **Per-bird voice signature:** Fixed register, signature motif, rhythm, and timbre keep the bird recognizable for life.
- **Fresh variation every call:** Pitch, timing, syllable count, and motif order vary so the same call never plays exactly twice.
- **Mood and drift change delivery only:** Tempo, loudness, phrase length, register spread, and density change, but not signature motif or timbre fingerprint.
- **Recognizability targets:** Listening panels and embedding margins verify identification at 7 birds and with same-species pairs.
- **Idle call scheduling:** Local Poisson calls make ambient sound ongoing while server-scheduled social calls remain canonical.
- **Beak sync:** Lookahead links score syllable onsets to rendering within the timing tolerance.
- **Listen-in mix:** The focused bird rises, others lower but never go silent; panning remains untouched because this is listening, not channel switching.
- **Autoplay handling:** The plan maximizes immediate starts where allowed and resumes on first activation with no "tap to enable sound" prompt because a prompt would be an announcement.
- **WebAudio fallback:** Silence plus captions preserves accessibility without adding recorded-audio fallback.
- **Caption generation from score:** Captions describe exactly what was heard and merge chorus windows.
- **Audio QA:** Offline render safety, loudness, recognizability proxy, human listening panels, fatigue tests, and cross-browser worklet tests guard uncanniness and regressions.

### Accessibility surfaces

- **Aviary ARIA structure:** The canvas is hidden from assistive tech while narration and bird focus targets expose the product.
- **Screen-reader narration from rendered state:** It describes what the canvas actually shows without reading trait values.
- **Narration cadence and priority queue:** It keeps updates slow, avoids repetition, and delivers important events within about 1.5 seconds without `assertive`.
- **Naturalist narration voice:** Lowercase present-tense prose with bird names and no "you" preserves product voice.
- **Visible narration option:** It reuses the same text for users who want narration text.
- **Caption placement and backing:** Auto-placement and translucent backing protect legibility without overlapping birds and top bar.
- **Caption nodes `aria-hidden`:** Narration serves screen readers so captions are not double-announced.
- **Keyboard bird focus targets:** Transparent buttons make moving canvas birds keyboard and screen-reader accessible.
- **Roving spatial navigation:** Arrows, Home/End, Enter/Space, Escape, and `o` give full keyboard operation.
- **Focus ring:** The double ring follows the bird and meets contrast requirements in every lighting state.
- **Hit targets:** 44 by 44 CSS pixels support touch and exceed the WCAG 2.2 AA minimum.
- **Accessibility settings:** Captions, reduced motion, visible narration, top-bar visibility, sound, volume, and shortcuts expose expected controls.
- **Synced settings with device-resolved system motion:** A captions user wants captions everywhere, while OS motion setting is a device fact.
- **Accessibility audit and research:** Axe, keyboard, contrast, external audit, usability sessions, and accessibility lead veto make accessibility a release blocker.

### Voice and content system

- **Two copy catalogs:** Components declare a register so surfaces stay naturalist or system as assigned.
- **Naturalist lint:** Lowercase, present tense, no exclamation, no second person, no digits, and no trait names preserve the naturalist voice.
- **System lint:** Sentence case and direct wording keep errors and settings matter-of-fact.
- **Global banned list:** It blocks welcome-back, streak, reward, loneliness, hunger, sickness, and similar framing.
- **Typed weighted prose grammar:** It gives deterministic, testable prose from state slots while preventing recent-template repetition.
- **No LLM in v1:** The plan cites privacy, determinism, voice control, and cost; sending per-bird state to a third-party model would breach INV-09.
- **Large authoring volume:** Hundreds of notebook, narration, caption, and greeting fragments reduce repetition over months.
- **Content review gate:** Content-designer approval and sample reviews protect voice quality.

### Privacy and security engineering

- **Three data planes:** Identity DB, simulation DB, and telemetry have separate credentials and network segments so analytics cannot reach simulation data.
- **CI guard on telemetry connectors:** It enforces the plane boundary in infrastructure.
- **Encrypted email with blind index:** Lookup works without using email as an identifier, and only the mailer decrypts to send.
- **Redacting `Email` type and canary scans:** These catch accidental email leakage in logs or traces.
- **Telemetry label allowlist:** Operational metrics cannot carry account, aviary, bird, session, or email identifiers.
- **RUM page-view IDs only:** Client telemetry does not link to accounts.
- **Logs with limited account IDs:** Operational debugging is allowed for 14 days, but logs exclude payloads, traits, moods, notebook text, names, and email.
- **No third-party analytics or scripts:** This avoids session replay, ad pixels, external fonts, and third-party payload exposure.
- **Magic-link hardening:** Fragment tokens, POST consumption, single use, expiry, and rate limits defend sign-in.
- **`__Host-` cookies and CSRF:** Cookie and request rules protect sessions.
- **Trusted Types, SRI, and CSP:** Static assets and scripts are constrained.
- **Visitor token binding and revocation:** Visit access is read-only, browser-bound, and revoked on next poll.
- **Account enumeration resistance:** Identical responses and timing-equalized handlers hide account existence.
- **Support access consent grant:** Staff-only tooling requires a user-initiated, time-boxed, audited grant and never feeds aggregates.
- **Privacy policy:** It discloses telemetry categories, excluded interaction state, retention, backup expiry, and processors.

### Performance budgets and observability

- **Budget table:** Hard PRD caps and stricter internal targets keep bundle, first-bird, frame, memory, snapshot, and tick latency measurable.
- **Reference profile definition:** The plan gates the return visit because that is when the "already running" illusion matters most, while cold first visit is tracked separately.
- **500 ms strategy:** Edge TTFB, cached boot, snapshot parse, first draw, and frame presentation add to about 410 ms with headroom.
- **Runtime frame budget:** It leaves headroom for GC and browser work while audio runs on the rendering thread.
- **Memory rules and soak:** Preallocation, fixed nodes, pooled DOM, bounded queues, pruning, atlas release, and listener cleanup support no-growth testing.
- **Server metrics:** They cover operational health without per-account labels.
- **Client RUM:** It records aggregate timings, frame histograms, audio start state, errors, and bucketed session duration with no account link.
- **Synthetic monitoring:** Dedicated synthetic accounts check first-bird, frame timing, audio startup, freshness, and sign-in from regions and browsers.
- **Deliberately unmeasured data:** DAU, retention, drift distributions, funnels, click maps, A/B tests, accessibility-setting adoption, notebook reads, and extra visitor behavior are "never" instrumented because the boundary is the privacy promise.
- **Alarms and SLOs:** Tick latency, backlog, publish lag, integrity, API 5xx, magic-link consumption, first-bird regression, and audio unavailability are the earliest allowable operational signals.

### Testing strategy

- **`aviary-core` unit and property tests:** They prove monotonicity, absence invariants, determinism, batch equivalence, presence union, greeting, and offer cooldown behavior.
- **Calibration harness tests:** They replace production aggregates and verify persona bands, mood occupancy, and notebook cadence.
- **API contract tests:** They block trait fields, enforce visitor write rejection, and prove idempotency.
- **DB tests:** They enforce grants, monotonic trigger, and concurrent CAS behavior.
- **Chaos and failure tests:** They rehearse worker death, stale workers, failover, Redis loss, edge lag, and outage catch-up.
- **Browser E2E:** It verifies sign-in, adoption, first frame, greeting, listen-in, offers, settle, notebook, visits, deletion, and recovery in real browser flows.
- **Presence tests:** Real OS visibility, focus, activity, idle, and touch tests guard honest presence.
- **Anti-announcement DOM audit:** It catches text nodes, alert/status roles, and forbidden components that would break quietness.
- **Voice lint, accessibility, performance, audio, privacy, load, and security tests:** These turn cross-cutting promises into release gates.

### Delivery plan

- **Team shape:** Workstreams map to the bespoke systems: engine, client scene, audio, platform, product UI, accessibility, content, design, and quality.
- **M0 foundations:** Early CI, planes, core, schemas, design tokens, voice guide, and species shortlist set the invariant scaffolding.
- **M1 vertical slice:** A staff member must see two birds already in motion under 500 ms, hear procedural calls, and prove presence-driven drift.
- **M2 feature complete:** Full species, mood, social, weather, day/night, interactions, notebook, accessibility, accounts, and starters come together before hardening.
- **M3 hardening:** Visits, newcomers, governor, soak, accessibility audit, pen test, privacy review, load test, and DR drill prove readiness.
- **M4 dogfood:** Real-time drift must be felt before beta; diaries validate "looking back" changes rather than session-to-session changes.
- **M5 private beta and GA:** Production runs through SLO burn-in and capacity before open sign-up.
- **Snapshot schema critical path:** Freezing v1 after week 6 lets platform, engine, and boot renderer move safely with additive changes.
- **Dogfood clock critical path:** Visible drift is a real-time phenomenon, so late cohort start slips beta.
- **Species delivered in pairs:** Audio grammars and rigs must be co-designed, with starter-eligible pairs first.
- **Cuttable items:** Reverb, SharedArrayBuffer, some governor refinements, sixth species, extra fragments, visible narration band, and tz polish can slip without violating the PRD.
- **Never-cut items:** Tick, presence precision, monotonic drift protections, procedural calls, first-frame no entry, greeting, accessibility, privacy planes, anti-announcement guardrails, and performance gates are core promises.

### Rollout

- **Internal alpha:** Staging validates function and harness-versus-felt behavior.
- **Dogfood cohort:** Consenting staff and friends calibrate real-time drift, notebook sparsity, audio fatigue, and accessibility diaries.
- **Private beta:** Capped production tests real networks, devices, deliverability, and SLO burn-in.
- **No beta reset:** Production beta accounts are real and permanent because reset would violate identity and persistence.
- **Bird-count ramp:** Age-based growth means production is two-bird for months, giving time to validate larger aviaries.
- **Global adoption ceiling:** It defers, not skips, newcomer windows until performance, recognizability, narration, layout, and snapshot size pass for each count.
- **Instrumentation from day one:** Allowed metrics, integrity jobs, mail dashboards, audio-start shares, and edge freshness are live before beta.
- **Launch gates:** Harness, performance, accessibility, privacy, security, integrity, content, and anti-announcement audits all must pass.
- **Parameter changes:** Harness diffs, sign-off, staged rollout, and engine-version records protect tuning changes.
- **Kill switches:** Offers, visits, notebook, newcomer scheduler, tick cadence, and edge inline snapshot can be controlled without adding replacement chrome.
- **Runbooks:** Tick backlog, integrity violation, deliverability, edge KV outage, audio regression, and session compromise procedures are prepared because these are named operational risks.

### Risks

- **Fast drift risk:** The mitigation is slow launch values, daily caps, reservoir spread, and reviewed changes because over-drift cannot be rolled back.
- **Slow drift risk:** Harness day-21 visibility and visible perceptibility mappings keep the product from reading as a screensaver.
- **No production drift analytics risk:** The constraint is accepted; calibration uses harness, dogfood, support, interviews, and consent-gated support view.
- **Presence inflation or deflation risk:** Three-signal presence, `isTrusted`, touch `pointerdown`, cross-device union, and concavity address it.
- **Personality vector loss risk:** Seven-layer defense, DR drills, two-person migrations, and monotonic trigger address the worst failure.
- **Sync and tick incorrectness risk:** Pure `step()`, CAS, idempotent events, batch-equivalence, DST/travel personas, and chaos tests address it.
- **Audio uncanniness risk:** Sound design, variation, density caps, loudness ceilings, recognizability gates, night quietness, and compression address it.
- **Autoplay risk:** Resume on activation with slow fade, no nagging prompt, captions, and RUM audio-start-state address it.
- **Accessibility regression risk:** A11y lead veto, DOM overlays, cadence, contrast, audits, and disabled-participant research address it.
- **Performance risk:** Edge-inline snapshot, tiny boot, service-worker caching, governor, soak tests, and honest measurement address it.
- **Announcement and gamification creep risk:** No components, banned copy lint, DOM audit, CODEOWNERS, and contributor invariants address it.
- **Privacy leakage risk:** Data-plane isolation, `Email` type, label allowlist, no third-party SDKs, support consent, and 14-day logs address it.
- **Magic-link deliverability and scanner risk:** Provider setup, SPF/DKIM/DMARC, warmed subdomain, fragment tokens, POST consumption, and re-request path address it.
- **Tick cost risk:** Bucketed bulk batches, HOT updates, sharding plan, and dormant-batching equivalence address it.
- **Export/personality conflict risk:** Opaque encoding and legal review address it.
- **Newcomer under-discovery risk:** Narration, notebook observation, offer-tray item, recurring windows, and no pressure address it.
- **Content repetition risk:** Large template volume, recent-use memory, and ongoing writer investment address it.
- **Visit abuse risk:** Host limits, no free-text message, single-browser binding, revocation, and expiries address it.
- **Schedule risk:** Vertical slice by week 8, cut lines, and species pairs address it.
- **Cross-engine determinism risk:** Integer PRNG and fixed-point canonical paths address it.

### Decision log

- **Settle as fifth top-bar control:** Two files require settle in the top bar; a fifth quiet control keeps the bar sparse.
- **Opaque personality export:** It satisfies portability without creating a numeric surface.
- **Quantized snapshot presentation levels:** They keep trait values from leaving the server.
- **Presence activity events:** `keydown` replaces deprecated `keypress`, and `pointerdown` avoids biasing against touch users.
- **Unioned multi-device presence:** Attention is per person, not per device.
- **Attunement for quieter returns:** It is the only way to satisfy quieter after absence and no trait decreases.
- **Reservoir-and-release filter:** It expresses low-pass drift and absence release while remaining monotonic.
- **Audibility effect only on vocal inflow:** It honors mute without punishing mute.
- **Offer cooldown behavior:** The offer works visually, cooled-down birds glance, and there is no game-like cooldown display.
- **Offer drop point:** It gives "near" a concrete, user-intelligible meaning.
- **Server greeter, client variation:** It is canonical for notebook truth and still never canned.
- **Hidden vs. unfocused behavior:** Hidden saves battery; visible but unfocused keeps the aviary alive without recording presence.
- **Autoplay fallback without prompt:** A prompt would be an announcement.
- **Visit link binding and duration:** Sharing is deliberate and bounded, with no permanent visitor list.
- **Host email in invitation:** No display names exist, the email is verified, and no free-text message avoids spam.
- **Visit notification channel:** Email only because there is no push and badges are forbidden.
- **Current notebook names after rename:** Names are user labels and identity is `bird_id`.
- **Newcomer flow:** It makes the offer a noticing, not a notification.
- **No bird removal:** It conflicts with identity continuity and "birds do not die."
- **`roosting`:** The layout requires sleeping birds and avoids collision with the settle gesture.
- **Accessibility preference scope:** Captions sync everywhere; OS motion setting is device-specific.
- **Canonical account timezone:** Both devices must show "the same aviary in the same mood."
- **English only:** The naturalist grammar is authored per language.
- **Keyboard listen-in controls:** Browsing by keyboard should not thrash the mix, and transfer mirrors click-switching.
- **Offer shortcut:** It satisfies WCAG 2.1.4 while keeping the shortcut.
- **90-day sliding session:** It supports a calm, low-friction product.
- **Every-minute dormant tick in v1:** The PRD says ticks run regardless of connection.
- **Authored prose grammar, no LLM:** It preserves privacy, determinism, and voice control.
- **No DAU/retention metrics:** The plan follows aggregate categories and an anti-engagement stance.
- **Synthetic weather:** It avoids location data and keeps weather "never assertive."
- **Stylized sun without location:** It gives local-time anchoring without geolocation.
- **Canonical social calls only:** Notebook-visible events must be canonical; solo idle calls are presentation.
- **Birds after slow snapshot load:** Emerging from foliage avoids fade-from-nothing and keeps the aviary continuing.
- **Error placement:** Errors need matter-of-fact clarity without announcement patterns.
- **Dawn mood relaxation:** It honors daily-ish reset and never snaps.
- **Magic-link scanner defense:** Fragment tokens and POST consumption prevent silent token burn and log leakage.
- **Unified sign-up:** Magic-link only means no separate registration.
- **Calibration data source:** Harness and consenting staging cohort preserve INV-09.
- **Backup residue:** Crypto-shredding PII and expiring simulation backups is the bounded, disclosed approach.
- **Rename concurrency:** Optimistic concurrency handles user-authored scalar conflicts; the no-LWW rule targets personality.
- **Time-to-first-bird measurement:** Return visit is the gated illusion; cold first visit is reported separately and honestly.

### Open items and parameters

- **Legal open items:** Export encoding, host email disclosure, metric set, backup residue language, and visitor retention need specialist confirmation, but defaults apply.
- **Design open items:** Top-bar iconography, settle glyph, focus-ring colors, caption backing, and reduced-motion poses need specialist input.
- **Sound-design open items:** Species grammars and signature-separation thresholds need sound-design finalization.
- **Content open items:** Suggested names, song-fragment names, and grammar corpus need content review.
- **Engine/design open items:** Presence window and drift gains are finalized after dogfood.
- **Server-configured parameters:** The rationale is controlled, versioned, reviewed tuning with audit history.
- **PR guardrail checklist:** It makes each change confront text register, scene UI, gamification, notifications, bird-state writes, metrics, dependencies, motion, audio, and performance budgets before merge.
