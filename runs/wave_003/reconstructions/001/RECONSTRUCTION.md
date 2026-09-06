## System-level intent

- The plan treats principles as enforceable system constraints. This appears in "Invariants with teeth," where principles are "invariants with enforcement, not as tone guidance," and again in launch gates requiring every invariant to have an automated check.
- The plan wants buildable decisions, not open design drift. Its status says it "interprets the spec into buildable decisions," and "open questions are resolved, not deferred."
- Canonical behavior belongs to the server tick. The plan repeats that the tick is the "sole writer" of bird and aviary state, that "clients submit events, never values," and that there is no client path for trait, mood, perch, or timeline writes.
- The render boundary is philosophical as well as architectural: "The server decides what happens; the client decides how it looks and sounds." Canonical state, server-computed plans, derived rendering, and device-local choices are kept separate.
- Multi-device coherence is intentional. The plan says two devices should render "the same birds on the same perches in the same moods hearing the same phrases," with only micro-motion phase allowed to differ.
- Personality is meant to be hidden, behavioral, and monotone. "Personality numbers never reach a user surface," drift is "monotonic non-decreasing," and visible change is mapped through greet-first weight, perch propensity, call rate, callbacks, offer approach, head-tilt, and plumage step.
- Absence should read as quiet, not punishment. The plan explains "neglect reads as quiet" through the decaying attention reservoir: traits are "untouched," expressiveness falls, then recovers with presence.
- Presence must be honest rather than merely tab-open. It is defined as "visible and focused and recent pointer/key activity," unioned across devices, clamped server-side, and tested so it "never exceeds wall time."
- The product rejects announcement and engagement-loop surfaces. The plan forbids "toasts, banners, welcome text, streaks, counters, badges, calendars," excludes achievements and feeds from scope, and ends with "nothing, anywhere, announces."
- Privacy is a hard product boundary. "Per-account interaction data never leaves the simulation database"; RUM is "aggregate-only"; there is no analytics role; and the plan deliberately does not measure "engagement" or per-bird behavior.
- Email is exceptional PII and synthetic UUIDs are the normal identity. The plan says "Email is stored once, encrypted," every other reference is the account UUID, and blind indexes are "not an identifier."
- Bird identity is meant to be durable and recognizable. Bird UUIDs are "never reused, never rewritten," signatures are immutable, migrations cannot re-seed birds, and species changes never remove or remap existing birds.
- The aviary should feel continuous and alive immediately. The first frame is "the aviary, already moving"; loading is a "quiet field"; offline uses a local improviser; and the user should never see a frozen aviary for transient gaps.
- Accessibility is a v1 product surface, not a follow-up. Narration, captions, reduced motion, focus, keyboard, and WCAG AA are M1-M2 deliverables and must arrive "in the product voice, not a description of it."
- Audio is procedural, quiet, and individually recognizable. "No recorded audio" is an invariant; each bird has a stored signature; chorus avoids phase-lock; and beta identification gates the seven-bird ladder.
- Calibration is explicit and non-invasive. Launch defaults are tuned by a calibration harness, "felt" drift is checked through opt-in research and diary interviews, and the plan rejects production interaction data for calibration.

## Per-feature whys

### Scope

- Two starters chosen by the system: The starters are sampled with a boldness gap so the first encounter has "a legible bolder-and-warier pair" and greet-first order is readable from day one.
- Six-species pool: NOT RECOVERABLE FROM PLAN
- One night-active species: The night-active bird keeps its normal call rate at night and gives the aviary night calls while non-night-active species mostly roost.
- Age-based arrivals up to seven birds: The ladder "ramps itself," gives dogfood and beta aviaries earlier canary counts, and lets audio identification studies gate larger groups before public aviaries reach them.
- Hidden five-trait personality vector: Personality is meant to become visible through behavior and render parameters, while "personality numbers never reach a user surface."
- Monotonic drift: The plan makes drift non-negative by construction so birds never get worse; absence reduces expressiveness through reservoir decay instead of moving traits down.
- Persisted mood: Mood is part of canonical state so two devices show the same moods, and "nothing about a snapshot request touches mood."
- Bird-to-bird callbacks: Warm birds and calls create ambient social response, with callback probability tied to warmth and responders staggered rather than simultaneous.
- Alarm spread: Wind and wary alarms add mood influences to nearby birds, making alarm a shared aviary event rather than an isolated animation.
- Chorus: Close overlapping calls by vocal birds become a deliberate `chorus` item so clients render intentional overlap and the notebook detector can fire.
- Stable bird identity: Bird rows are never deleted or re-seeded except hard delete, preserving identity, signatures, and recognizability across migrations.
- User-assigned and renameable names: Naming is "quiet" and optional, avoiding nagging or reward-like flows; prose can use given names while unnamed birds remain described by species.
- Return-greeting: The API computes it because absence length is known server-side, the notebook needs the greeter, and "clients never choose behaviour."
- Presence accounting: Presence exists to keep watching honest: visible, focused, recently active, unioned across devices, clamped so it never exceeds wall time.
- Listen-in with gradual mix re-balance: Listen-in focuses one bird while keeping others audible with a floor; credit is clipped to presence so it cannot accrue without real presence.
- Seed, song fragment, and still pool offers: Offers return immediate choreography while the tick recomputes the same plan later, preserving the single-writer rule.
- Per-bird cooldowns and aviary-level offer spacing: Cooldowns keep repeated offers from becoming spammy; birds in cooldown ignore and accrue no drift.
- Settle with a five-second undo: Settle is "a gesture the user gives the aviary"; undo is free, and only after the window does the server see a clean mood-quieting signal.
- Read-only, sparse field notebook: It records aviary observations, not user actions, and uses token-bucket sparsity to match "one entry every few days, more when something happens."
- One horizontal screen: NOT RECOVERABLE FROM PLAN
- Three perch zones and nine slots: Front, middle, and back zones make boldness, wariness, and curiosity legible through position and movement.
- Local-time day/night: The aviary uses the day of the active device so mood priors, roosting, lighting, and morning arrivals match the user's current local day.
- Rare rain and wind: Weather contributes mood influences, dampened calls, shared timeline items, and occasional notebook observations while the aviary continues.
- Ambient leaves and feathers: Leaf or feather moments can provide low-score filler only when nothing more notable has happened for days.
- Subtle parallax: The plan says parallax should "read as depth, not as an effect," so it follows slow ambient drift rather than the cursor.
- Thin fading top bar with exactly four icons: The fading bar keeps chrome quiet over the scene; exactly four icons force settle to live as a separated gesture inside the offer sheet.
- Quiet-field loading state: The quiet field avoids spinners and announcements while still giving a phase-correct, living first surface.
- Empty-aviary fly-in after adoption: NOT RECOVERABLE FROM PLAN
- Magic-link sign-in: Single-use, short-TTL links, no account enumeration, rate limits, and token hashes provide sign-in without exposing email as an identifier.
- Per-device revocable sessions: Sessions carry device labels and revocation so a user can remove a device; origin enforces revocation on the first API call.
- Verified email change: NOT RECOVERABLE FROM PLAN
- JSON export delivered as an emailed link: Export is the one named exception where the current personality vector appears, because it is "the user's own data" delivered as a file and not rendered in product.
- Thirty-day soft delete then hard purge: NOT RECOVERABLE FROM PLAN
- Synthetic UUID identity and encrypted email: Email is stored once and encrypted; every other reference is the synthetic account UUID to avoid PII spread.
- Server-side tick, snapshot pull, append-only event log: This sync model creates "multi-device coherence by construction" while keeping clients event-only.
- Named email invites: Invite emails are rate-limited so the product cannot become a "mail cannon," and the invite remains active and revocable.
- Read-only visitor ambient view: Visitors receive no greeting, offer state, settings, presence tracker, or event capture, so visiting cannot influence drift.
- Immediate visit revocation and expiry: Revocation takes effect at the next visitor pull with matter-of-fact copy, and expiry bounds the lifetime of visit access.
- Visit log in settings: NOT RECOVERABLE FROM PLAN
- Visit notification toggle off by default: The toggle exists, but caps and default-off behavior prevent notifications from becoming a feed.
- Naturalist screen-reader narration: Narration uses the same snapshot and timeline as the renderer, on a slow cadence, so screen-reader users receive the aviary in the product voice.
- Reduced-motion cross-fade renderer: Reduced motion is a designed surface with feature parity, not a stripped fallback.
- Runtime-generated call captions: Captions make calls legible when audio is off or locked and are generated from the same phrase the engine scheduled.
- Complete keyboard navigation and WCAG AA copy: These are required because accessibility surfaces ship in v1 and are launch-gated, not deferred.
- Last-two-major browser support: NOT RECOVERABLE FROM PLAN

### Architecture and data model

- Preact for chrome: The plan keeps a "small UI library for chrome only" and keeps UI framework code out of the render loop.
- Custom Canvas 2D renderer: Seven birds "do not need WebGL," and Canvas 2D has the "least driver variance on old laptops"; the interface leaves WebGL possible later.
- WebAudio synthesis engine: Procedural audio satisfies the no-recorded-audio invariant and avoids audio files in the bundle or on the wire.
- DOM accessibility layer: Screen-reader narration, captions, and focus controls live outside the canvas so the scene can remain visual while access stays semantic.
- Service worker app shell: Repeat opens can draw the first bird from cache in well under 200 ms and then pull fresh state.
- Edge worker and CDN with inlined boot snapshot: Edge verification and cached snapshots avoid an origin round trip for first paint and serve the first bird within the hard budget.
- Stateless Node/TypeScript API: TypeScript lets the backend share `@aviary/engine`; the API can assemble snapshots and plans while never writing bird state.
- Tick workers with leased partitions: Leases, row locks, and compare-and-set preserve the sole-writer model and let fleet size be driven by tick lag.
- Shared `@aviary/engine`: A single pure implementation of drift, mood, grammar, and choreography avoids server/client divergence; golden vectors prove parity.
- Shared `@aviary/prose`: Prose centralizes template grammar, anti-repetition, and voice lint across notebook, narration, captions, descriptors, and adoption copy.
- Postgres, Redis, object storage, and KMS: Postgres is canonical state, Redis covers leases/rate limits/cache, object storage holds exports, and KMS protects email and blind-index keys.
- Single primary region topology: NOT RECOVERABLE FROM PLAN
- Separate observability account: The metrics/logs/traces account has no VPC peering or database credential, keeping product health analytics outside simulation data.
- Render-pipeline boundary: Canonical state, server-computed request plans, client-derived rendering, and device-local choices are separated so canonical state is not computed on the client.
- Device-local listen-in focus, settled lighting, audio state, top-bar fade, motion mode, and captions: These are local because they affect experience, not canonical bird state.
- Trust and privacy roles: The `api` role cannot update traits, reservoirs, mood, perch, or timeline; the `tick` role cannot read email ciphertext; "analytics" does not exist.
- UUIDv7 identifiers and stored IANA timezone: Identifiers are time-ordered and random, while local-time reasoning is based on the aviary's stored timezone.
- Email blind index: It enables magic-link lookup and uniqueness "without decrypting every row" and is never logged, joined, or exported.
- Immutable bird signature: A stored signature lets a bird stay recognizable in pitch, tempo, brightness, motif weights, ornaments, and rhythm.
- Event retention after consumed tick plus seven days: Raw interaction data exists only to drive the simulation, with a week covering tick replay and debugging.
- Species records in the engine package rather than the database: NOT RECOVERABLE FROM PLAN

### API surface

- `X-Aviary-Api: 1` version header: NOT RECOVERABLE FROM PLAN
- System copy from `copy/system.ts`: Error and account copy use the "matter-of-fact voice" with normal capitalization and no bird verbs.
- HttpOnly/Secure/SameSite cookies, Origin checks, and UUID/IP-hash rate limits: These protect state-changing requests without using email as a rate-limit identity.
- `POST /api/auth/magic-link` always returning 200: This avoids account enumeration while enqueueing link email under rate limits.
- `GET /auth/consume` creating the account, aviary, and starters for new users: NOT RECOVERABLE FROM PLAN
- Snapshot endpoint with greeting and timezone update: It keeps local day current and lets the server compute eligible greetings against current state.
- Batched event ingestion with idempotent event ids: Retried batches are safe because event ids are primary keys and the tick consumes append-only events.
- Offer endpoint with reaction plan and `available_at`: The client can render immediately, while 429 spacing keeps offers from violating aviary-level timing.
- Adoption and starter-name endpoints: NOT RECOVERABLE FROM PLAN
- Visitor endpoints with read-only snapshot and 410 on revoked or expired access: The visitor path cannot append events and uses matter-of-fact failure copy.
- No endpoint for trait, mood, perch, or timeline writes: This enforces that clients submit events and "never values."

### Simulation engine

- Every aviary ticks on the server regardless of connected clients: "The aviary always continues server-side," so time, weather, drift reservoirs, and arrivals continue without clients.
- Sixty-second cadence with flag-gated idle coarsening and sub-steps: Coarsening is a cost lever, and sub-steps make the 300-second path "provably equivalent."
- Lease, row lock, compare-and-set, and single transaction: These prevent double drift, lost events, and partial writes if workers race or die.
- Snapshot cache warming and immediate tick for coarsened aviaries: The returning user's state and greeting are fresh even after idle coarsening.
- Server-side presence clamp, union, and clip: Timestamp clamping defuses clock skew; union across sessions prevents double-counting; clipping keeps presence within the tick window.
- Listen-in credit clipped to presence and visitor sessions contributing nothing: Listen-in cannot accrue without presence, and visitors produce "zero presence, zero drift."
- Reservoirs and drift equation: Presence, listen-in, accepted offers, and near offers become non-negative signals that make drift measurable and monotone.
- Daily drift cap: Heavy use stays bounded, with tests ensuring per-day delta never exceeds the cap.
- Expressiveness `E`: Decaying attention explains "quieter after absence" while traits remain unchanged and can become expressive again after return.
- Mood set with `roosting` separate from settled lighting: "Settled" is reserved for the lighting state; `roosting` is the bird's night state.
- Mood priors, influences, decay, and dwell: Local phase and short-lived influences create persistence and a "daily-ish reset" without snapshot requests changing mood.
- Behaviour timeline with seeds: The tick emits a 90-second ordered list so every device agrees on calls, movement, weather, and activities from the same seeds.
- Perch movement from front-perch propensity and mood: Position makes boldness, curiosity, drowsiness, and wariness visible without labels.
- Calls, callbacks, chorus, alarm, weather, and activities: These create an ambient social soundscape, deliberate overlaps, shared weather, and notebook-detectable events.
- Return-greeting buckets, lottery, stagger, and variation: Tab switches become only a glance, longer absences reorient birds, bolder warmer birds usually greet first, and responders are never simultaneous.
- Offer reaction mapping and recomputation: The API plan gives immediate choreography, while the tick later applies the same reaction plan to mood and drift.
- Field notebook detectors, token bucket, filler, and voice lint: The notebook stays sparse, observation-only, lowercase, present-tense, non-numeric, and free of user-action or achievement language.
- Adoption ladder, morning arrivals, and quiet naming: Arrival timing matches "a few months," gives the notebook and user the same moment, starts full bird identity immediately, and avoids nagging.
- Determinism and PRNG: Pure engine functions plus versioned golden vectors keep server and client behavior byte-equal or fall back safely on version skew.
- Calibration harness T1-T10: Synthetic profiles tune launch defaults and guard measurable drift, honest presence, neglect quietness, notebook sparsity, calls, and greetings.
- Engine test suite: Unit, property, integration, sub-step, timezone, ladder, and notebook tests make invariants and edge cases fail in CI.

### Sync model

- One canonical record: The aviary row and birds are the state, avoiding client-to-client paths, merge conflicts, and client-held simulation state.
- Pull triggers and animated reconciliation: Pulls on visibility, focus, frame gaps, exhaustion, and keepalive keep state fresh, while hops and cross-fades ensure "nothing snaps."
- Single writer and no last-write-wins for simulation: Additive event deltas plus CAS make double application impossible; last-write-wins is reserved for names and settings because they "have no drift to lose."
- Multi-device timezone, greetings, settle, and cooldowns: Recency resolves timezone, greeting rate limits prevent duplicate greetings, settled lighting stays local, and offer cooldowns are aviary-level.
- Offline local improviser: It provides visual and audio continuity with no canonical effect, and a real snapshot wins on reconnect.
- Migration and bird identity policy: Migrations cannot rewrite IDs, re-seed traits, or regenerate signatures; any trait semantic change must be monotone and audited.
- Failure-mode recovery: Worker death, backlog, stale cache, clock skew, duplicate batches, and magic-link replay are handled without partial state or dramatic user-facing copy.

### Frontend rendering pipeline

- Vite chunking, no third-party scripts, system fonts, and vector species art: These choices serve privacy, bundle size, and the first-bird budget.
- Inline boot path: The boot script draws sky, perches, and silhouettes before external JS so the first frame is the aviary with no spinner or entry animation.
- Pixel-diff continuity between boot and renderer: The first animated frame must match the static pose, preventing a visible discontinuity.
- Scene layer composition and layout solver: Ordered depth layers, bounded slots, DPR caps, and viewport tests keep birds inside the scene from 320 to 2560 px.
- Bird rig and procedural idle motion: Seeded noise and part rigs make each bird's manner persist across devices without being identical frame for frame.
- Mood through posture: Alert, curious, content, wary, drowsy, and roosting are read from pose and movement; there is "no label, tooltip, or icon."
- Transitions and choreography: Perch moves, greetings, offers, pool actions, and arrivals are procedural families rather than clips, so repeated actions vary.
- Settle choreography: Lighting, audio, and posture change together; any click, tap, Enter, or Space within five seconds reverses the local effect and cancels the event.
- Timeline executor and interpolation: Server-time scheduling, fixed simulation steps, dropped stale items, and snapshot reconciliation keep motion coherent through pauses and resumes.
- Reduced-motion renderer: `StillsRenderer` shares the same interface and keeps calls, captions, greetings, offers, notebook, narration, and drift.
- Top bar and sheets: The scene remains primary; sheets are anchored under the four-icon top bar and are "the only place chrome exists."
- Offer sheet with settle visually separated: Settle is reachable from the top bar while still framed as a fourth, different gesture.
- Client presence tracker: The tracker implements the visible/focused/recent-activity definition and flushes bounded intervals; visitor mode never instantiates it.
- UI lint, component allowlist, forbidden identifiers, and copy import boundaries: These prevent announcement components, reward vocabulary, trait fields, and voice leakage from creeping in.
- Memory and frame discipline: Pools, object reuse, windowed notebook rows, stable listeners, and zero per-frame allocation guard the 30-minute no-growth budget.
- Loading, unsupported, error, and audio-locked states: Loading is quiet field; unsupported and errors use matter-of-fact sheets; the scene keeps last known state where possible.

### Audio pipeline

- Pooled WebAudio graph: Nodes are created once and calls are envelopes, removing per-call allocation and start/stop clicks.
- Motif recipes: Whistles, trills, chips, burrs, purrs, doubles, and churrs provide procedural bird-like vocabulary for species grammars.
- Per-bird signatures and distance rule: Pitch, tempo, brightness, motif weights, ornaments, and rhythm make birds recognizable, especially as the aviary approaches seven birds.
- Symbolic phrase generation: The same seed yields the same phrase on every device, while seeded variation and no-duplicate tests avoid "the same call twice."
- Mood and personality shaping in audio: Mood shapes phrase length, pitch, attack, volume, and brightness; personality only reaches audio through server-computed call rate and callbacks.
- Lookahead scheduling and stale-item drops: Calls stay aligned to server time, and a resumed laptop never plays a burst of catch-up calls.
- Mixing, listen-in, chorus, night, settle, rain, and wind: The mix targets "a quiet room," keeps a listen-in floor rather than muting others, and uses detune/micro-timing to avoid phase-lock.
- Background tabs keeping ambient audio: The plan says "a tab left open for the sound is a window left open," while presence remains unaffected because visibility fails.
- Autoplay lock and WebAudio fallback: Silence plus captions is the fallback, consistent with no recorded audio and no "enable sound" announcement.
- Song-fragment offer timbre: NOT RECOVERABLE FROM PLAN
- Audio quality tests and beta listening panel: Offline renders, CPU/node stability, spectral-flatness checks, and identification studies guard robotic sound, comb filtering, and recognizability.

### Accessibility surfaces

- Screen-reader narration live region: A polite, single-slot, slow-cadence region gives scene and bird lines from the same snapshot without becoming too chatty.
- Focus and keyboard: Roving buttons over birds, arrow navigation, Enter/Space listen-in, Escape, focus traps, and return focus make the canvas operable.
- Captions near birds: Captions are phrase-derived, positioned near the bird's head, stacked when close, and backed by a fixed-lightness scrim for AA.
- Reduced motion as accessibility: It carries the same calls, captions, greetings, offers, notebook, narration, and drift and gets design-system screenshots.
- Contrast and focus ring: Token checks across lighting phases protect copy, icon glyphs, and focus outlines against evening and night palettes.
- Accessibility settings surface: Captions, motion, narration text, narration pace, and audio settings persist on the account and apply on every device.
- Accessibility testing: Screen-reader matrices, keyboard walkthroughs, axe-core, visual regression, voice lint, and beta research are launch-gating.

### Performance, observability, rollout, and operations

- Hard performance budgets and CI gates: Bundle size, first-bird time, idle motion, memory, audio CPU, tick latency, API latency, mail latency, and availability are made testable rather than aspirational.
- Bundle, first-bird, frame, memory, and tick techniques: Vector parts, procedural audio, inlined snapshots, cached layers, bounded pools, no allocation, and bounded queries are the stated ways to meet budgets.
- Server observability and alerts: Tick lag, CAS conflicts, backlogs, API errors, cache misses, mail depth, export/purge jobs, and replication lag are monitored so operations can act.
- Client telemetry privacy boundary: RUM is aggregate-only and closed-schema, with no cookie, session, account id, or bird data.
- Deliberate non-measurement: The plan rejects visit frequency per account, retention cohorts keyed to aviary state, drift distributions, offer/listen counts per account, per-bird anything, and "engagement."
- Workstreams and milestones: The schedule sequences foundations, vertical slice, complete feature set, private beta, and launch with exit criteria tied to tests and launch gates.
- Server-side flags and kill switches: Flags can pause visits, weather, ladder, notebook, background audio, and tuning values, but "no flag exposes or alters the invariants."
- Launch gates: Hard budgets, harness tests, accessibility, voice lint, privacy, security, export/delete, kill switches, and runbooks must be green or explicitly waived.
- Ramping birds per aviary: The age-based ladder creates public, beta, and dogfood canaries, and `ladder.paused` can hold arrivals with nothing announced.
- Instrumented from day one: Operational metrics, aggregate feature-use counts, mail funnel, invite counts, export/delete outcomes, and log-scanner hits are present from launch.
- Operations runbooks: Runbooks cover tick lag, CAS spikes, cache misses, mail outage, failover, key rotation, abuse, RUM regression, job failure, and incident communication in matter-of-fact voice with no in-product announcement.
