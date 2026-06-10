## System-level intent

- Server-owned canonical simulation state. The plan repeatedly makes the server and tick worker the authority: "Server is the only writer of personality state" in the plan-wide invariants, clients "never send state - only events" in event ingest, and the ownership table says personality, mood, perch, E, weather, and settled are server-owned. The intent is to make client conflict and accidental state overwrite "unreachable," not merely discouraged.

- Monotonic personality and non-punitive neglect. The invariant "Drift is monotonic toward expressive" appears in the plan-wide invariants, personality drift, property tests, and the risk discussion. Neglect is handled by the expressiveness modulator, so an absent user returns to birds that are "quieter, present, and unresentful" rather than damaged or reset.

- Presence is narrow, observable attention. The plan defines presence as the "three-way conjunction" of visibility, focus, and recent activity, then repeats that no surface may use a laxer definition. At the same time, the default window is tuned to "lean long - watching without moving is the product," so the intent is strict accounting without turning still observation into absence.

- Privacy is an architectural boundary. The plan says "The privacy boundary is architectural," then implements it through physical separation, a metric registry, aggregate-only RUM, no read path from telemetry into the simulation DB, encrypted email, synthetic UUIDs, and CI rejection of metrics with account or bird dimensions. The intent is to make "one convenient join" impossible.

- No announcements, no gamification, no engagement loops. The invariant "No announcement surfaces, no gamification surfaces, ever" is carried through scope, copy lint, notebook grammar, top-bar design, telemetry exclusions, and risk R11. The plan wants "no toasts, banners, welcome text, streaks, counters, badges, levels," and even avoids computing underlying stats that could feed leaderboards.

- The aviary should feel alive whether watched or not. Tick scheduling, inline snapshot catch-up, long-gap fast-forward, service-worker warm starts, and risk R10 all serve the claim that the aviary is "continuing without you" and that user-visible staleness cannot occur. Every snapshot read is supposed to show a current aviary.

- Plain platform, careful product engineering. The architecture is a "deliberately boring modular monolith plus one worker fleet, one Postgres, one CDN." The plan explicitly says the interesting engineering lives in the simulation tick and client, while the surrounding platform stays "as plain as possible."

- Canonical policy, ephemeral performance. Calls, poses, ornaments, and some reactions are client-realized, while canonical state remains server policy. The decisions section says "calls are weather, not ledger," and the snapshot exposes call policies rather than call instances. The intent is to preserve character without over-ledgering every sensory event.

- Voice is infrastructure, not vibes. The voice section says "Voice is load-bearing, so it gets infrastructure," with a central registry, naturalist/system registers, lint, denylist, voice owner, and generated-prose coverage. This intent also appears in notebook, narration, captions, settings, auth, and errors.

- Accessibility is the actual product. The accessibility section says these are "Designed surfaces, shipped in v1 with the features they mirror," and risk R6 says "The accessible product must stay the actual product." Reduced motion, narration, captions, keyboard paths, focus treatment, and audits are all treated as launch gates.

- Calibration protects the central promise. The plan marks drift constants as harness-owned, calls the drift-calibration harness "the load-bearing test rig," and risk R1 says the "central promise lives in a narrow band." The intent is conservative launch, config-only adjustment, and beta validation rather than guessing.

- Performance budgets are part of felt aliveness. The plan ties TTFBird, bundle size, 60fps, memory growth, and snapshot size to "felt-aliveness numbers." The boot section says there is "no spinner anywhere in the product" and the first frame is "mid-action," making performance a product-voice requirement rather than a backend metric.

- Continuity is identity. Stable bird IDs, fixed lifetime signatures, persisted mood, additive drift, soft-delete restore behavior, and risk R15 all protect continuity. The plan says "Continuity is identity," and treats reset, frozen restore, or inconsistent multi-device state as threats to that identity.

## Per-feature whys

### Scope and product surface

- Single horizontal scene and three perch zones: NOT RECOVERABLE FROM PLAN

- Day/night by account-local time: The plan ties account-level IANA timezone to diurnal priors, lighting, and night gating so "visitors see the host's time of day" and the visitor sees "exactly what the host would see."

- Rare ambient weather: Weather is meant to create "its own moments" without becoming "assertive or random." The plan makes it server-canonical so devices and visitors agree, while droplets and leaves remain client-local ornaments.

- No in-scene chrome and fading top bar: This serves the anti-announcement invariant. The canvas contains "zero chrome, labels, tooltips, or overlays," while the top bar fades and contains only account/settings, accessibility settings, notebook, and offers.

- Quiet landing/sign-in page: The plan places auth, identity, errors, and settings in the system register, with sign-in emails "system-voice, no marketing." The quiet landing/sign-in page follows that register boundary.

- Unsupported-browser surface: NOT RECOVERABLE FROM PLAN

### Birds

- Two starters, system-selected and user-named: The starters are chosen for contrast in silhouette and call register, with deliberate seed-trait differentiation so "the first week already shows legible character contrast." Naming frames them as "the birds that arrived."

- Seven-bird cap: The plan ties the cap to audio recognizability and chorus quality. It tests worst-case "7-bird flock" signature distance and treats recognizability as a cap on the product.

- Roughly six-species pool exact size: NOT RECOVERABLE FROM PLAN

- Stable bird identity: Bird IDs are "never reused or regenerated," and each bird's audio signature is "fixed for the bird's lifetime." This protects continuity and keeps a bird knowable across moods and months.

- Hidden personality vector: Raw traits never leave the server; snapshots expose derived render parameters only. The plan resolves "never expose" as never rendering personality in any UI, with export as a machine-readable portability exception.

- Mood machine: Mood gives visible variation without snaps. Dawn re-anchoring creates "fresh mornings" while preserving persistence across sessions, and sticky dwell prevents flicker.

- Age-gated new-bird offers: The schedule is self-ramping, giving the team months of real chorus data between cap stages. The surface is a quiet "visitation" with no badge, modal, countdown, or urgency mechanics.

- Adoption and naming sheet: The client presents starters as "the birds that arrived," with curated suggestions and rename later. The empty-aviary quiet field and first bird soft fly-in make adoption part of the aviary surface rather than an announcement.

### System architecture and data model

- Modular monolith plus worker fleet, Postgres, and CDN: The plan wants the platform "as plain as possible" because the interesting engineering is in the simulation tick and client, not distributed infrastructure.

- TypeScript monorepo, shared schema, and pure sim-core: Shared zod schemas are a "single source of truth," and pure sim-core enables the tick worker, calibration harness, time-compressed QA, and deterministic tests to run the same math.

- Custom canvas renderer plus Preact chrome: The scene is only "7 birds + ornaments on one screen," so a game engine would spend bundle budget on capabilities the product does not use.

- HTML from origin with embedded state and CDN static assets: Serving HTML from origin lets the page inline the current snapshot for one-round-trip boot, while CDN handles immutable static assets.

- Encrypted email, HMAC lookup hash, and hashed tokens: These support the privacy and security boundary: no email in logs/metrics/queues, no raw token at rest, and no account enumeration.

- Partitioned append-only event log: The event log provides server order, idempotency, retention as user history, and watermark-based tick consumption so event loss or ordering bugs are detectable.

- Static versioned species and template content: NOT RECOVERABLE FROM PLAN

### API surface and sync

- Magic-link auth: The plan uses uniform 202 responses for no account enumeration, hashed single-use tokens, short TTLs, and system-voice emails with "no marketing." Risk R9 frames auth as "single-path by design."

- Per-device revocable sessions: Sessions are the "source of truth," with httpOnly cookies, device labels, last-seen, and server-side revocation so a device can be revoked directly.

- Email change with verification: NOT RECOVERABLE FROM PLAN

- Snapshot pull with `known_tick`: The snapshot is the read path. `known_tick` enables a cheap "304-style" response when nothing changed, keeping polling lightweight.

- Inline catch-up on snapshot read: The API runs bounded catch-up before serving stale snapshots, so "every served snapshot is current" and hosts or visitors never see a lagged aviary.

- Event ingest batching and idempotency: Batches are capped and deduped by client idempotency keys to produce at-most-once effect. Server timestamps and `seq` ordering make client clocks advisory only.

- Clients submit events only: This enforces I1. The plan says clients send "user listened in on Pip" and never "set warmth to x," making state conflicts and last-write personality loss unreachable.

- JSON export: Export exists for data portability, including raw personality vectors as machine-readable data rather than a rendered surface.

- Soft-delete, hard-delete, and restore: Soft-delete suspends ticks because the user "asked to leave"; restore performs catch-up so birds are quieter, not frozen or reset. Hard-delete includes an audit asserting zero rows remain.

- Visitor routes and render-only mode: Visitors have no event-ingest route, generate no presence, and cannot feed drift. The visit page uses the same renderer in render-only mode so visitors see the same ambient aviary without affecting it.

- Invite expiry, revocation, and single-consumption browser-bound links: The raw visit link dies after first use; revisits work only from the bound browser while valid. Revocation and expiry are checked on every poll.

- Visit log with email, date, and approximate duration: NOT RECOVERABLE FROM PLAN

- Polling-only sync: The plan says visible keepalive plus read-time catch-up meets every latency requirement, and "real-time push buys nothing" while websockets add infrastructure and reconnect-state bugs.

### Simulation engine

- Pure tick function: Purity makes calibration, time-compressed QA, and deterministic tests possible because `tick(state, events, interval, config, rng)` can run outside production I/O.

- Active and dormant tick cadence: Tiering honors "runs whether or not anyone is watching" at about "1/15 the dormant cost," while inline catch-up makes cadence differences unobservable.

- Long-gap fast-forward: For gaps over 48 hours, the plan samples distributions rather than stepping every interval, preserving the sense that the aviary continued without wasting work or flooding the notebook.

- Personality drift formula: Saturating drift is "fast-feeling early, asymptotic later, never overflowing." It keeps all deltas additive and supports the plan's measurable-at-one-week, visible-at-three-weeks contract.

- Daily drift cap: The rolling cap enforces "no single session moves a trait visibly," protecting the product from marathon-session jumps.

- Expressiveness modulator E: E is how absence becomes ambient. It makes birds quieter after a lapse while guaranteeing the aviary is "never dead" and allowing "a few good sessions" to restore expressiveness.

- Presence accounting: The conjunction prevents laxer shortcuts, while account-wide interval union ensures two devices cannot double-count. Settle and tab-close are identical to the engine, with no penalty surface.

- Dawn-anchored mood chain: The mood chain uses sticky state, diurnal priors, weather, interaction nudges, personality bias, and contagion. Dawn re-anchoring gives daily freshness "without visible snap."

- Return-greeting planner: The greeting is computed on the snapshot response to fit the 1-2s arrival window. Absence tiers separate a coffee break from arrival, and anti-streak variation makes greeting order "worth writing."

- Weather generation: Per-aviary server-canonical weather lets devices and visitors agree, while mood effects make weather influence the birds without turning weather into a user-facing event surface.

- Offers engine-side cooldown and response: The engine chooses the responding bird and validates cooldowns so the offer stays "a gesture, not a vending machine." Rejections produce no user-facing error.

- Settle, undo, and re-engage: Settle creates a canonical quiet state visible across devices and visitors. A five-second undo reverses cleanly, and re-engage returns gently through explicit interaction.

- Seed personalities: Species priors and deliberate contrast make early character legible, while different call registers and silhouettes prevent the two starters from blending.

- Notebook generation: The notebook is server-only because it must never claim client-only specifics. Sparse emissions and adaptive thresholds target "1 entry per 2-4 days," and the grammar has "no grammatical slot for the user."

- Shared naturalist text engine: Using one fragment library for notebook, narration, and captions creates "one voice" across visual, screen-reader, and caption experiences.

### Sync model and state ownership

- Ownership table and database grants: The ownership table narrows each state writer, and Postgres grants make wrong writes fail at the DB. The plan wants invariants enforced by schema, not review convention.

- Multi-device coherence and presence union: Both devices append events and the tick consumes them in server order; unioned presence prevents simultaneous devices from inflating drift.

### Client architecture and rendering

- Inline boot and micro-paint: The first frame should be the aviary "mid-action" with "no spinner anywhere." Inline snapshot plus a tiny pose-compatible painter aims at first bird in under 500ms.

- Quiet field loading state: If snapshot boot fails, the fallback is a designed "quiet field" with subtle motion cues, not a spinner. It also serves the pre-adoption empty-aviary moment.

- Service-worker warm path: Caching the shell and last snapshot lets a returning user see the aviary instantly, then refresh and reconcile, which the plan calls "the truest implementation of the conceit."

- Layered renderer and parametric birds: Cached layers, parametric Path2D birds, object pools, adaptive resolution, and no unnecessary bitmaps serve 60fps and bundle limits while keeping all birds in frame.

- Behavior controller and local reactions: Client reactions make offers and calls feel immediate, while canonical mood remains server-owned. The plan says the one-minute seam is invisible because reactions and mood-idle differences are continuous.

- Visibility loop: When hidden, the client stops rendering and audio because "the server is the simulation." On visible, it pulls a snapshot and reconciles to truth.

- Reduced-motion presenter: Reduced motion is "a register of rendering," not a feature-flag sprinkle. It preserves calls, captions, drift, mood, notebook, and e2e parity with a calmer aesthetic.

- Reconcile rules: Short gaps ease through natural transitions; longer gaps render canonical state directly because "things moved while you were away." This avoids snapping while preserving truth.

- Top bar and lazy secondary surfaces: The bar is thin, contains only named surfaces, lazy-loads secondary UI, and never fades while focused. This keeps the aviary canvas free of chrome while preserving access.

### Audio pipeline

- Voice pool and no steady-state allocation: Pre-built synth voices, reused buffers, and AudioParam automation support the zero-growth memory budget and avoid rebuilding WebAudio graphs.

- Motifs, grammar, and phrase generation: Parametric motifs and grammar tables produce calls from policy rather than recordings, supporting procedural variety and small bundle size.

- Per-bird audio signature: The signature transform is fixed for the bird's lifetime so "Pip stays knowable by ear across moods and months."

- Anti-repetition memory: Phrase memories and minimum parameter distance make "hearing the same call twice, exactly" structurally impossible, with a regression test rendering 500 phrases.

- Bird-to-bird responses and chorus: Chorus is "real polyphony, no loop stacking by construction." Risk R3 calls audio "the affective spine," so response timing, limiter design, and density gates protect the feeling.

- Call-space allocation for new birds: Species selection weights toward unoccupied call space, directly defending "knowable by ear at 7."

- Listen-in mix: The focused bird rises while others drop only to an ambient floor, "never to zero." Organic ramps preserve the aviary bed, and listen-in events feed drift.

- Autoplay visual start and fade-in: Because browsers gate audio, the plan accepts a silent visual start and fades audio in on first qualifying gesture, with "no prompt" and no click-to-start modal.

- WebAudio unavailable fallback: Graceful silence turns captions on by default for that session because otherwise the product would be "silently absent." The note lives inside accessibility settings, not a toast.

### Accessibility, voice, privacy, performance, testing, and rollout

- Screen-reader narration: The narration engine ships in the core bundle so SR users get it on first load. It reads "the same aviary the eyes would" and uses cadence/coalescing to avoid spam.

- Call captions: Captions are generated from realized phrase parameters so they match what actually played. They are `aria-hidden` because narration owns the spoken channel.

- Keyboard navigation, focus, and contrast: Focus proxies, roving tabindex, Enter/Escape behavior, and high-contrast focus tokens support full keyboard sessions and hold contrast across lighting states.

- Voice and copy registry: Central tagging, lint, denylist, and voice-owner approval make voice enforceable. The denylist blocks announcement and gamification language across both registers.

- Telemetry boundary and metric registry: The metric registry and allowed aggregate set keep RUM aggregate-only, exclude account and bird dimensions, and deliberately avoid data that could support leaderboards or engagement loops.

- Performance budgets: Initial JS, first bird, frame rate, memory, tick latency, and snapshot payload are CI/fleet-enforced because the plan treats them as "felt-aliveness numbers."

- Memory discipline: Pools, recycling, virtualized notebook scroll-back, and heap tests make "zero net growth over 30 min" real rather than aspirational.

- Synthetic fleet and RUM: The scripted fleet asserts cold load, idle, listen-in, offer, settle, reload-warm, autoplay, and fallback paths from multiple geographies; RUM alarms stay within the aggregate-only boundary.

- Security controls: Magic-link, session, CSRF, CORS, CSP, visit-token, validation, PII, dependency, and external review controls target account takeover, invite token leakage, event-flood abuse, and privacy leakage.

- Drift-calibration harness: The harness is the "load-bearing test rig" for the central promise. It validates one-week measurement, three-week behavior visibility, no single-session visibility, and E-floor lapse behavior.

- Audio quality gates: Signature metrics, blind listening, anti-repeat rendering, and cross-browser conformance exist because audio is the "affective spine" and must remain recognizable across browsers and flock densities.

- Accessibility tests: axe, keyboard e2e, narration cadence, register lint, manual SR matrix, reduced-motion parity, and external audit prevent accessibility from landing after v1.

- Milestone gates: Gates are "hard" and "failing-capable from day one." Each milestone blocks on the product risks most relevant to that stage: alive feel, a11y, voice, privacy, security, calibration, and runbooks.

- Private beta: Beta lasts at least four weeks because "the product's core claim needs >=3 weeks to be perceivable." Instrumentation is disclosed and deleted after calibration.

- Birds-per-aviary ramp and stage gates: Age-gating gives months of real chorus data before higher densities. A failing gate delays additions through server config "without ever revoking a bird."

- Instrumented from day one: The plan adds specific aggregate metrics and alarms early so boot, audio, email, snapshot, reduced-motion, and caption behavior are observable without violating the privacy registry.

- Deliberately deferred calibration-time constants: Exact W, final cadence, drift constants, notebook theta, palette tokens, focus treatment, species art, motif content, and provider selection are deferred where the plan names them as harness-, designer-, sound-designer-, or ops-owned rather than engineering guesses.
