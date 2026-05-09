## System-level intent

- Treat affect as an engineering constraint, not as decoration. The plan says the load-bearing rules are "`feels alive, not robotic`", "`notice never announce`", "`charm comes from specificity`", "`restraint over richness`", "presence as central idea", and "drift monotonic toward expressive"; it then gives those rules "concrete enforcement points" in architecture, schema, lint, CI, review, and rollout gates.

- Make the aviary continue without the viewer. This shows up in the client/server split ("the client is a render+capture surface only"), the server-side tick, the single canonical aviary, the serial event log, mood persistence, and the repeated statement that the server owns personality, mood, timeline, notebook entries, visit-invite state, and account state.

- Prefer "notice never announce" and "restraint over richness" across product surfaces. The plan refuses welcome banners, toasts, spinners, skeleton screens, milestone confetti, streak copy, and launch hype. The top bar fades instead of taking over the scene; new-bird offers are "unobtrusive"; notebook entries are sparse; the product is "meant to be found, not announced."

- Keep the voice split load-bearing. The plan divides surfaces into "Naturalist" and "Matter-of-fact": aviary, notebook, narration, captions, offer prompts, and return-greeting flavor use lowercase, present-tense, specific observational prose; auth, account, settings, errors, export, delete, unsupported-browser, and visit-revocation surfaces use capitalized, direct plain English.

- Encode refused features as absence, topology, and review gates. The plan repeatedly says gamification, Tamagotchi mechanics, social-network surfaces, push re-engagement, native concessions, recorded audio, and personality stat panels are refused "at the architectural level." It does this by not computing counters, not storing streak-shaped columns, isolating visit tables, separating analytics, blocking UI strings, and requiring rule-of-the-product review.

- Preserve one authoritative reality. The plan's "single canonical aviary" depends on server-owned simulation state, one source of truth per account, serial event consumption, no client personality writes, no last-write-wins model, and database role boundaries that prevent the API service from updating personality vectors.

- Make drift "monotonic toward expressive." The plan treats this as "the single most important design rule in the engine": traits can only increase or remain stable, ignored birds become "ambient" rather than distressed, and any change that subtracts would require rule-of-the-product review.

- Let change emerge over weeks, not minutes. The calibration commitments, drift test harness, age-based new-bird offers, sparse narrator cadence, slow day transitions, and closed-beta review all aim for instrument-detectable change after about one week and user-perceptible change after about three weeks without turning attention into a progress system.

- Protect privacy by design and by omission. Email is isolated, encrypted, and never used as an identifier. Telemetry is aggregate only. Analytics and simulation live in separate databases. Per-bird and per-account state are not reachable from the telemetry pipeline. The plan often relies on "physically impossible" joins rather than trust.

- Treat accessibility as a designed surface. The plan says accessibility is "not parity-by-checklist": screen-reader narration is naturalist prose, reduced-motion is "its own designed surface," call captions match the procedural audio, keyboard navigation is full, and WCAG AA contrast is tested across aviary states.

- Use procedural specificity rather than canned content. Bird calls are WebAudio, motif-based, parameterized, and signature-preserving; captions are derived from the same motif spec; visual motion uses named loops and mood-keyed transitions; the phrase library is curated so the voice remains invariant.

- Let performance protect the central conceit. The <500ms time-to-first-bird target, no-loader rule, 2MB bundle budget, 60fps idle target, 30-minute no-growth memory test, and synthetic monitor all serve the same reason: if the first experience reads as loading software, the aviary stops feeling alive.

- Keep sharing opt-in, bounded, and non-social. The visit feature is off by default, per-invite, read-only, revocable, expiring, cookie-less, and separated from the host event log. The plan says this prevents co-presence, visitor influence, "show-off" rendering, and social graph growth.

- Preserve the product with durable gates. The plan names lint checks, schema review, code-review checklists, editor-owned phrase libraries, accessibility QA, privacy review, synthetic performance tests, calibration tests, and rule-of-the-product review as defenses against small future changes that would erode the product rules.

## Per-feature whys

### Scope

- Single canonical aviary, magic-link auth, and multi-device sync: The rationale is to keep "one source of truth per account" so every client renders the same state and multi-device sync is "trivial."

- Two starter birds at adoption: The plan frames the first encounter as the moment "the affective contract is made"; the birds are presented as "the birds that arrived," not as a catalog or avatar configuration.

- Six species pool: NOT RECOVERABLE FROM PLAN

- Cap of 7 birds per aviary: The rationale is partly auditory and affective. The plan says a user should be able to identify Pip's call from Wren's, and "this is what makes the cap of seven possible"; the cap is also enforced in service code and database constraints so growth does not become an endless system.

- Three perch zones: NOT RECOVERABLE FROM PLAN

- Stable internal bird ID: The rationale is bird identity. Renaming, syncing, or species-pool migrations "never replace this row," and there is no regenerate-bird path.

- Per-bird personality vector, mood, slow drift, mood persistence, and bird-to-bird interaction: The rationale is that the aviary "feels alive over weeks" and does not snap back to defaults when the client opens.

- Aviary scene with no in-aviary chrome and a fading top bar: The rationale is to keep controls available while preserving the aviary surface. The top bar is "never fully invisible" but fades because of "the small affordance cost of having controls on the surface."

- Return-greeting, listen-in, offer, settle, and read-only field notebook: The rationale is to provide small interaction surfaces without turning the product into a game, journal, or dashboard. Listen-in is attention, offer is gentle contact, settle quiets the scene, and the notebook is "an observer's record, not a journal."

- Server-side simulation tick at about 60s cadence: The rationale is to make "the aviary continues without the viewer" a real property while keeping cadence tunable without code changes.

- Procedural client-side call synthesis with no recorded fallback: The rationale is to avoid looped or canned audio, preserve call variation and signature, and avoid adding recorded assets or fallback paths that contradict procedural-only design.

- Accessibility surfaces: The rationale is that these are "designed surface" choices, not checklist parity. Screen-reader narration, reduced motion, captions, keyboard navigation, and contrast are part of the actual product experience.

- Visit-invitation feature: The rationale is bounded sharing without a social network. It is opt-in, off by default, read-only, revocable, expires after 30 days, and keeps visitor traffic out of host drift.

- Account hygiene: The rationale is user control and privacy: verified email changes, 30-day soft-delete restore, hard delete, export to verified email, and per-device session revocation.

- Aggregate-only operational telemetry: The rationale is to get operational signal while ensuring "per-bird interaction state never aggregated."

- Native iOS/Android refusal: The rationale is to avoid native code paths, native protocol changes, and abstractions left "in case."

- Gamification refusal: The rationale is that the product must not compute or store the counters that later become streaks, badges, levels, XP, "days visited," calendars, confetti, or activity logs.

- Tamagotchi-mechanics refusal: The rationale is that birds should not have hunger, death, distress, or negative drift; the drift function "physically cannot decrement personality traits."

- Social-network refusal: The rationale is to avoid profiles, follows, discovery, leaderboards, comments, co-presence, shared cursors, and friend notifications. The visit data model is "a small isolated table" that cannot become a social graph.

- Push notification and re-engagement email refusal: The rationale is to avoid announcement and re-engagement surfaces in the wrong product; email is limited to magic-link, export, and account events, with visit notification only if opted in.

- Native schema-concession refusal: The rationale is to protect server authority and the single canonical aviary by avoiding device-local personality state, client-authoritative writes, and multi-aviary accounts.

### Architecture

- Three independently deployable service boundaries: The rationale is that each boundary has "its own failure mode" and clear ownership: edge/web has no business logic, API handles account and event surfaces, and simulation owns canonical personality state.

- Edge/web tier with static SPA, SSR shell, CDN edge, and inline initial snapshot: The rationale is first-frame speed and no business logic at the edge.

- API tier as stateless HTTP service: The rationale is to handle auth, reads, event-log writes, notebook reads, visits, and account ops without owning simulation state.

- Simulation tier as sharded tick worker pool: The rationale is to own canonical personality state and advance the aviary independently of client connection.

- Mailer service: The rationale is to isolate allowed email surfaces: magic-link, account-export, deletion-confirmation, revocation, and opt-in visit-notification emails.

- Narrator job: The rationale is to decide notebook entries at low cadence, keep entries sparse, and preserve a hand-shaped invariant voice.

- Synthetic perf monitor: The rationale is to continuously test time-to-first-bird and first-call latency from common geographies so performance regressions do not silently break the affective contract.

- Telemetry collector separated from simulation database: The rationale is aggregate-only metrics ingest with hard separation from per-bird and per-account state.

- Client/server split: The rationale is explicit: it makes "the aviary continues without the viewer" true and makes multi-device sync trivial.

- Render pipeline boundary with authoritative time-stamped snapshots: The rationale is continuous client rendering without letting the client invent persistent birds, calls, moods, or state.

- Inline initial snapshot for first frame: The rationale is to bypass a network round-trip on warm cache and support the <500ms time-to-first-bird target.

- Go backend: The rationale is a single binary, mature concurrency primitives, predictable tick-loop latency, low ops cost, and avoiding unnecessary polyglot surface area.

- PostgreSQL 16: The rationale is v1 scale, team ops experience, partitioning where volume warrants, and avoiding premature Kafka, NATS, or CRDT infrastructure.

- Redis for cache and queue: The rationale is short-lived session state, rate limits, locks, and dispatch triggers while keeping canonical personality state out of Redis.

- S3-compatible bucket for outbound JSON exports: The rationale appears with account export: write a blob, email a 24-hour signed URL, and auto-delete it at expiry.

- CDN with edge HTML compute: The rationale is edge-served HTML with inline snapshot SSR for first-frame performance.

- TypeScript, React, and Canvas2D: The rationale is to let React own chrome and account surfaces while the Aviary Canvas2D component owns the scene; Canvas2D is considered sufficient for the motion budget.

- WebAudio API directly: The rationale is that the needed synthesis nodes are small, so no audio library belongs in the bundle.

- Vite, esbuild, bundle-size gate, service worker: The rationale is strict bundle budget enforcement, small build surface, snapshot caching, and offline-first cold-load behavior.

- Local, staging, and prod environments: The rationale given is that staging mirrors prod's data model and tick worker shape, so the synthetic perf monitor can run before and after deploys.

### Data Model

- Synthetic UUID identifiers: The rationale is that identifiers are "never derived from email or user input."

- Encrypted account email and HMAC email hash: The rationale is lookup without exposing plaintext email outside the auth path, with lazy re-encryption on key rotation.

- Rule that email never appears elsewhere: The rationale is privacy and isolation; logs, partitions, telemetry tags, queue keys, error messages, and filenames must not carry email.

- Bird row with stable ID: The rationale is the "load-bearing invariant" of bird identity across renaming, syncing, and species-pool migrations.

- Bird name as user-assigned and renameable: NOT RECOVERABLE FROM PLAN

- Bird `archived` field reserved and never used in v1: NOT RECOVERABLE FROM PLAN

- Personality vector as jsonb traits: The rationale is per-bird variance and render/audio behavior without exposing a numeric stat panel.

- Current mood and `mood_until`: The rationale is mood persistence and cooldowns so birds do not snap or churn unrealistically.

- Explicit absence of streak, XP, score, visit count, last visited, and days active columns: The rationale is to make gamification counters unavailable in a queryable shape.

- Personality history table: The rationale is engineering inspection during calibration and incident response, not product exposure or aggregation.

- Interaction event log with monotonic per-account sequence: The rationale is append-only event capture for tick consumption in order, supporting sync and replay without client authority.

- Visit-session events excluded from drift inputs: The rationale is that visits are for the visit log only and should never influence the host's birds.

- Aviary runtime state with weather, day phase, timezone, and next bird offer time: The rationale is to maintain canonical simulation state and make bird-offer timing age-based rather than interaction-based.

- `next_bird_offer_at`: The rationale is to have the only growth counter be aviary-age-based, not surfaced as count or progress, and consumed once when the offer becomes available.

- Notebook entries as read-only naturalist prose: The rationale is that the notebook observes the aviary; `source` supports internal narrator review and is never sent to the client.

- Magic links as opaque tokens and sessions as revocable rows: The rationale is secure magic-link sign-in and per-device session hygiene.

- Visit invites and visit sessions in small isolated tables: The rationale is to enforce no co-presence, no aggregated visitor influence, and no social graph by topology.

- Telemetry tables in a separate analytics database with no account or bird IDs: The rationale is aggregate-only observability and CI schema lint blocking per-user state.

### API Surface

- HTTPS-only JSON endpoints: NOT RECOVERABLE FROM PLAN

- Versionless URL-level API with header versioning if needed: NOT RECOVERABLE FROM PLAN

- `POST /auth/magic-link` always returning 202: The rationale is to prevent email enumeration by making known and unknown email responses identical.

- Magic-link consumption as one-shot transaction: The rationale is race-safe session creation and token consumption.

- Matter-of-fact auth errors: The rationale is that a user trying to sign in needs information, "not flavor."

- `GET /aviary/state`: The rationale is to deliver the authoritative snapshot containing everything needed to render a frame.

- `GET /aviary/state/stream` with polling fallback: The rationale is tiny snapshot diffs while visible, with a 30-second visibility-aware fallback if SSE is unavailable.

- `POST /aviary/events` batching and canonical sequence assignment: The rationale is client deduplication, retry on overload, and server-side ordered tick consumption.

- Notebook GET-only endpoint: The rationale is architectural enforcement that the notebook is "an observer's record, not a journal."

- Visit endpoints and cookie-less visitor state: The rationale is that visitors cannot read or write other account data or append to the host event log.

- Visit revocation end surface: The rationale is matter-of-fact communication when the visit is no longer available.

- Account endpoints for email change, export, delete, restore, prefs, and rename: The rationale is account control, verified export, recoverable delete, and preference management.

- Account `display_name` patch field: NOT RECOVERABLE FROM PLAN

- Absence of streak, days-visited, and stats endpoints: The rationale is that the product does not compute the underlying counters.

- Account export containing personality vectors only in the export blob: The rationale is that "the user's relationship is theirs to take," while the UI never renders numbers as a stat panel.

- Stable error envelope with hand-written messages: The rationale is stable codes across versions and plain information without naturalist charm in system errors.

### Auth

- Magic-link token shape, storage, expiry, single-use consumption, and rate limiting: The rationale is secure, short-lived, one-shot sign-in with enumeration resistance.

- Matter-of-fact magic-link email body with no marketing copy: The rationale is to keep auth email plain and avoid marketing/re-engagement voice.

- HttpOnly, Secure, SameSite session cookie with rolling expiry: The rationale is durable sign-in with normal session protections.

- Per-device session list and revocation: The rationale is user-visible account hygiene and immediate revocation through Redis invalidation plus sessions table state.

- Suspicious-session signal shown only inside the device list page: The rationale is to avoid a notification that would be "an announcement surface in the wrong product."

- Email change keeps old email canonical until confirmation: The rationale is to avoid losing the working address if the new address is never confirmed.

- Soft delete and hard delete: The rationale is a 30-day restore window followed by idempotent full deletion and aggregate-only completion metrics.

### Simulation Engine

- Tick loop with per-account queueing and locks: The rationale is that only one worker ticks an account at a time, preserving "server is the only writer."

- Tunable 60-second nominal cadence and slower idle cadence: The rationale is to balance calibration with reduced idle work and avoid redeploys for cadence changes.

- Tick processing of events, birds, weather, call schedule, and narrator candidate signal: The rationale is that the aviary advances even with no clients connected.

- Personality vector traits: The rationale is to give each bird persistent behavioral and visual tendencies while keeping two birds of the same species from feeling identical.

- Persisted adoption seed: The rationale is that bird identity and personality are never re-rolled.

- Client receives plumage saturation only as render parameter: The rationale is to render visual richness without exposing the number as a labeled stat.

- Drift as bounded monotonic-up low-pass filter: The rationale is "drift monotonic toward expressive"; ignored birds become ambient, not wary, silent, or dull.

- `presence_drift` as dominant input: The rationale is that presence is the central idea and should be shaped by strict presence rather than tab-open approximation.

- Plumage saturation drifting only with sustained presence and visible perch positions: The rationale is to preserve the link between attention and visible richness.

- Drift clamp at 1.0: The rationale is to prevent runaway drift from breaking species recognizability.

- Calibration test harness: The rationale is to make drift targets named, defensible, changeable, and enforced in CI.

- Mood enum with cooldown, probabilities, local time, weather, interactions, and neighbor mood: The rationale is to avoid mood snap, preserve reproducible replay, and let birds feel alive in relation to context.

- Mood persistence across sessions: The rationale is that reopening the tab should not reset birds to a default mood.

- Settle gesture with 5-second undo: The rationale is to quiet the scene and move light toward dusk without making settle a drift input beyond closing the presence window.

- Visit-session filtering out of drift and mood pipelines: The rationale is that host birds drift from host presence only.

- Call grammar runtime with motif libraries and parameter seeds: The rationale is non-identical procedural calls that remain signature enough to identify a bird by ear.

- Mood-shaped call frequency: The rationale is that calls should express drowsy, settled, wary, content, curious, and alert states without replacing the underlying motif family.

- Chorus scheduling: The rationale is that overlapping calls should "fold into one another rather than colliding."

- Bird-to-bird interactions: The rationale is to make effects visible over a few minutes through small mood influences rather than a single tick shock.

- New-bird offers based on aviary age: The rationale is to make growth independent of interaction count, presence time, and notebook entries.

- Unobtrusive new-bird offer prompt: The rationale is no toast, no modal-on-load, and no progress indicator; the offer is gently noticed.

- Bird cap enforcement at service and database layers: The rationale is to make the hard cap real even if offer-generation logic fails.

- Day, night, and weather: The rationale is ambient continuity anchored to the user's local time, with rare weather and a night state that is not dead.

- Dropping signup IP geolocation after first use: The rationale is to get a coarse default latitude hint without retaining it over time.

- Nightjar-like nocturnal species: The rationale is that night should still have life and possible calls.

- Narrator job at low cadence: The rationale is sparse notebook prose about noteworthy aviary moments, not frequent logs.

- No LLM in v1 narrator: The rationale is invariant hand-shaped voice across millions of entries.

- Narrator forbidden from writing about "you" or user behavior: The rationale is the line between observations of the aviary and observations of the user's behavior.

### Sync Model

- Single canonical aviary: The rationale is one source of truth per account in the simulation database.

- Snapshot pull on visibility, render gap, keepalive, and SSE change: The rationale is to recover from visibility changes and suspend/resume while keeping snapshots small and account-specific.

- Snapshot `no-store`: The rationale is that snapshots are account-specific.

- Client event buffering and resend with backoff: The rationale is to preserve interaction events through network loss.

- Server sorting slightly out-of-order events by `occurred_at`: The rationale is order tolerance within a window before tick consumption.

- Presence definition requiring visible document, window focus, and recent pointermove or keypress: The rationale is to count presence rather than background tabs or abandoned laptops.

- Three-minute activity window: The rationale is that "watching birds without moving is the actual product," so the plan leans long.

- Presence seconds rather than ping count: The rationale is to model actual validated seconds and clip impossible bursts.

- Multi-device coherence through serial event log: The rationale is that simultaneous laptop and phone events do not need merging; the server reconstructs next state from sequence order.

- No last-write-wins: The rationale is to prevent one device from overwriting drift recorded from another session.

- Database role boundary for personality vector writes: The rationale is structural defense if a future endpoint accidentally exposes vector writes.

- Matter-of-fact auth conflict surfaces and no aviary-state conflict surface: The rationale is that the architecture should not produce aviary-state conflicts.

### Frontend Rendering Pipeline

- React for chrome and Canvas for scene: The rationale is to keep UI surfaces in React and the animation frame loop bound directly to snapshots rather than React rerenders.

- Inline snapshot first-frame strategy: The rationale is no blank state before first bird and the <500ms time-to-first-bird budget.

- Quiet field fallback: The rationale is that a spinner would "compromise the central conceit"; the quiet field reads as the aviary catching up.

- Layered scene composition: The rationale is visual depth and ambient life through sky, foliage, perches, birds, foreground branch, leaf drift, and weather overlays.

- Composed bird sprites with plumage saturation: The rationale is species-shaped identity plus visual drift/richness.

- Idle micro-motion loops keyed by mood: The rationale is specific living behavior without synchronized metronome motion.

- Perch-to-perch fly arcs shaped by boldness: The rationale is to make personality visible in movement.

- Smooth mood and day transitions: The rationale is no "mood snap" and continuous time rather than stepped fades.

- Settle visual ramp: The rationale is quieting the scene over 6-8 seconds in warm evening light.

- Top bar fade: The rationale is discoverable controls with low surface intrusion.

- Reduced-motion pipeline: The rationale is a designed surface with cross-fades, no animated paths, no leaf drift, gentle weather, and unchanged engine behavior.

- Visit rendering constrained mode: The rationale is read-only sharing with no offers, listen-in, settle, notebook, or event submission.

- Empty aviary quiet field and first bird fly-in: The rationale is to avoid "your aviary is being prepared" processing language and keep the first encounter alive.

- Unsupported-browser surface: The rationale is matter-of-fact compatibility guidance without shims.

- Audio-unavailable graceful silence with captions: The rationale is to keep the aviary usable and informative when audio cannot play.

- Forbidden frontend patterns: The rationale is to make loaders, welcome copy, celebration, streak language, personality numbers, and counters code-review failures.

### Audio Pipeline

- Single shared AudioContext on first user interaction: The rationale is browser autoplay policy compliance without blocking first bird visibility.

- Per-bird procedural voice graph: The rationale is bird-specific synthesized calls through small WebAudio nodes.

- Procedural-only audio: The rationale is no recorded audio bundle and no recorded fallback; unavailable WebAudio becomes graceful silence with captions.

- Listen-in mix: The rationale is to focus attention on one bird while other birds "do not go silent."

- Minimum gain floor for other birds: The rationale is to enforce that listen-in changes the mix without erasing the ambient aviary.

- Chorus mixing with reverb and limiter: The rationale is shared acoustic space and avoidance of transient peaks or loop artifacts.

- Caption generation from motif spec: The rationale is that captions match what was actually played and preserve naturalist voice continuity.

- Caption live-region dedupe: The rationale is to avoid overwhelming screen-reader users.

- Audio memory discipline: The rationale is the 30-minute zero net heap growth rule.

- Audio edge-case handling: The rationale is graceful silence, no hidden resume when tab-hidden, and aggregate error logging only.

### Accessibility Surfaces

- Screen-reader narration from server-derived prose: The rationale is naturalist access to the aviary, not a state list like "Pip mood: content."

- User-initiated narration priority: The rationale is to narrate return-greeting, offer, and settle promptly without breaking the naturalist surface.

- Captions: The rationale is access to bird calls and fallback when audio is unavailable.

- Keyboard navigation: The rationale is full keyboard access to top bar, aviary scene, birds, listen-in, settle, notebook, and settings.

- Focus indicator: The rationale is visible bird focus with contrast against any aviary background.

- WCAG AA contrast verification: The rationale is readable user copy across dawn, midday, dusk, night, and weather states.

- Accessibility settings: The rationale is user control over reduced motion, captions, top bar visibility, and narration cadence.

- Matter-of-fact accessibility setting labels: The rationale is that settings are system surfaces, not naturalist prose.

### Performance Budgets and Observability

- Initial JS bundle budget: The rationale is to keep the first experience under the load and time-to-first-bird budget.

- Code-splitting account, settings, visits, notebook, export, and sign-in flows: The rationale is to keep the initial bundle focused on the aviary.

- Ban on large general-purpose libraries: The rationale is bundle control.

- Time-to-first-bird inputs: The rationale is first bird visible without an extra round-trip, blocking audio setup, or network-fetched sprites.

- Idle 60fps target: The rationale is sustained ambient life on a 5-year-old laptop reference rig.

- Animation-loop constraints and pooled particles: The rationale is to avoid frame drops and layout reflows during idle motion.

- 30-minute memory budget: The rationale is zero net heap growth during long watching sessions.

- Tick latency budget and backpressure: The rationale is to keep simulation timely and slacken cadence before user-facing degradation.

- Aggregate observability: The rationale is RED/USE operational insight without per-bird, per-account, or per-interaction telemetry.

- Browser support for last two major versions of Chrome, Safari, Firefox, and Edge: The rationale given is that older browsers receive a matter-of-fact unsupported-browser page and the team will not maintain compatibility shims.

### Voice and Content Discipline

- Two voice surfaces: The rationale is to keep naturalist prose for aviary surfaces and matter-of-fact prose for system/account/error surfaces.

- Central editor-owned phrase library: The rationale is that engineering does not write product prose ad hoc and naturalist voice remains consistent.

- Forbidden welcome, achievement, user-behavior, stat, and marketing surfaces: The rationale is to protect "notice, never announce" and the gamification refusals.

- JSX content lint, schema lint, and rule-of-the-product review: The rationale is enforcement beyond taste or memory.

- Lowercase present-tense naturalist voice: The rationale is specific observation rather than state-transition phrasing.

### Privacy, Compliance, and Engineering Boundaries

- Email isolation: The rationale is that email stays inside the auth boundary and never appears in operational identifiers or telemetry.

- Logging filter, code-review checklist, and quarterly telemetry audits: The rationale is enforcement of email isolation.

- Separate simulation and analytics databases: The rationale is topology that makes per-bird state in telemetry impossible.

- Schema-locked analytics ingest allowlist: The rationale is to prevent analytics from reading simulation data directly.

- Account export job with JSON blob, signed URL, and 24-hour expiry: The rationale is data portability without building an in-product numeric preview.

- Hard delete and backup rollover: The rationale is to honor the privacy commitment after the 30-day soft-delete window.

- Rule-of-the-product review: The rationale is an explicit defense against "just one streak counter" and similar footholds.

### Rollout Plan

- Build phase milestones ending in M7 before any users: The rationale is that the product should not ship until calibration, performance, audio, accessibility, account, visit, and privacy gates are green.

- Refusal to ship a public beta with broken audio: The rationale is that audio is "the affective spine" and broken audio risks the "feels alive" core.

- Closed beta with about 50 personal invitations: The rationale is controlled review rather than public sign-up.

- Accelerated new-bird offer pacing for beta accounts: The rationale is to surface the offer experience for review within beta timing.

- Nightly calibration band checks against beta data: The rationale is to validate drift at aggregate level while respecting PII isolation.

- Weekly editor review of narrator notebook entries: The rationale is voice consistency.

- Public launch with small announcement: The rationale is that the product is not built for a hype cycle.

- Visit feature flag: The rationale is to keep visits off until invite, revocation, and read-only leak checks are robust.

- Server-side calibration knobs: The rationale is tuning without redeploy.

- Day-one instrumentation list: The rationale is to monitor request/error/latency, time-to-first-bird, tick health, audio errors, email sends, and calibration bands.

- Explicit non-instrumentation list: The rationale is to avoid collecting per-account presence, per-bird drift, visit frequency, session counts, days active, or anything that could later become a streak.

### Risks, Explicit Calls, and Definition of Done

- Drift band alerting: The rationale is that drift calibration failure is silent, so aggregate distributions need monthly review and alert thresholds.

- API role write test for personality vectors: The rationale is to catch future sync regressions that would let non-tick services overwrite drift.

- Personality history discontinuity checks: The rationale is to flag deltas whose source is not a recognized tick run.

- Audio designer review and synthetic listening test: The rationale is to prevent procedural calls from sounding mechanical or repetitive.

- Accessibility PR review and automated tests: The rationale is to catch later visual changes that silently break narration, captions, keyboard focus, or reduced motion.

- Bundle-size diff and source-map audits: The rationale is to prevent dependency creep from hurting time-to-first-bird.

- Telemetry schema lint and metric-tag audits: The rationale is to prevent a future debugging tag from leaking per-account behavior into analytics.

- Visit feature growth review: The rationale is to prevent invitations from becoming leaderboards, discovery, friend feeds, or a social graph.

- Tick worker outage behavior: The rationale is that the API can serve the last canonical snapshot and accept events so the user sees a stale aviary rather than an error.

- Adoption-flow wording and first bird fly-in: The rationale is to make first encounter feel like arrival, not avatar setup or processing.

- Backend stack open call: The rationale is already given in stack choices: Go/Postgres/Redis/S3 match concurrency, latency, ops, and export needs.

- Frontend stack open call: The rationale is already given in stack choices: React for chrome, Canvas2D for sufficient motion budget, WebAudio for small procedural synthesis, and no animation library.

- Default name list per species silhouette: NOT RECOVERABLE FROM PLAN

- Visit notification opt-in as email only: The rationale is consistent with no push notifications and opt-in visit notification surfaces.

- Simultaneous v1 definition-of-done gates: The rationale is that the product only ships when calibration, performance, accessibility, privacy, sync, visits, account export/delete, and audio fallback are all true at once.

- Quiet shipment: The rationale repeats the product philosophy: "Not with a launch event, not with a marketing push. The product is meant to be found, not announced."
