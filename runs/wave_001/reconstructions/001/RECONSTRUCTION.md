## System-level intent

- **The server owns the bird; the client renders it.** This is named as "the single organizing idea" and repeated in the client/server split, sync model, and simulation design. The plan carries this through as "server-authored, additive-delta" personality, a server-side tick, client event emission, and a hard ban on client-side personality writes or authoritative mood transitions.

- **Sync coherence comes from one canonical record, not reconciliation.** The plan says multi-device sync is "a property of the architecture, not a feature." It shows up in "one canonical aviary," "one canonical record, many readers," ordered `event_log.seq`, "no last-write-wins," and the lunch-phone-overwrites-morning-laptop failure being "unreachable."

- **The aviary continues without the viewer.** The plan grounds this in the tick that "must run for accounts with no connected client," mood persistence across sessions, day/night and weather advancement, snapshots on return, and the render rule that the app "loads with motion already in progress."

- **No Tamagotchi asymmetry: neglect contributes zero, never negative.** The plan names monotonic drift, "monotonic toward expressive," "no-punishment," "idle 14 days -> no trait decrease," and warns that adding symmetric decay "for realism" is "explicitly wrong here."

- **Architectural absence is a defense.** The out-of-scope section says rejected surfaces are "not 'later'" and that "the architectural absence is the defense." The data model deliberately lacks `streak`, `visit_count`, `level`, `score`, "negative-drift state," and cross-account aggregate tables; the risks section calls this the "harmless feature" leak.

- **Personality should be readable, never decoded as stats or controls.** This appears in hidden trait vectors, no numeric exposure, personality read through perch/pose/calls, "no label/tooltip/icon," and "the user never arranges birds; perch is signal, not control." The plan wants users to know a bird through behavior, not through a dashboard.

- **Naturalist charm belongs to bird surfaces; system surfaces stay matter-of-fact.** The plan uses "naturalist prose" for the field notebook, screen-reader narration, and call captions, but names a voice exception for magic-link replay, session timeout, server outage, account, settings, and errors: "No naturalist phrasing on system/identity/error/settings surfaces."

- **Accessibility is a first-class designed surface, not a checklist fallback.** The plan says accessibility is "designed for charm, not parity-by-checklist" and ships with v1. Reduced motion is "its own designed surface," screen-reader narration is prose rather than a state list, captions are runtime-generated, and keyboard navigation is part of launch.

- **Privacy is architectural, not policy.** This shows in service boundaries "drawn on the data-access line," synthetic UUIDs, encrypted email in one place, telemetry-svc with no simulation DB read grant, and CI checks that no metric carries per-account or per-bird dimensions.

- **Performance preserves the spell.** The plan treats <2MB, <500ms time-to-first-bird, 60fps idle, and no memory growth as product gates. It links this to the no-spinner first frame, procedural assets/audio, Web Worker offload, object pools, and "draw the first bird without waiting for non-critical assets."

- **The aviary is a small social system, not a row of NPCs.** The plan uses this exact contrast for bird-to-bird coupling. It shows up in call probability spread, wary mood spread, real-time chorus, small-bird-set coupling, and the cap of seven.

## Per-feature whys

### Scope

- Single-user accounts: Why: the plan frames v1 around one user's own aviary, one canonical record, and no shared/multi-profile aviaries, so account scope supports sync simplicity and the privacy boundary.

- Magic-link email sign-in: NOT RECOVERABLE FROM PLAN

- Per-device revocable sessions: NOT RECOVERABLE FROM PLAN

- Email change with verification: NOT RECOVERABLE FROM PLAN

- Account export: Why: export is treated as data portability for "the user's own record" and the one allowed context where current personality vectors can leave the server, only to the verified account owner.

- Soft-then-hard deletion with 30 days: NOT RECOVERABLE FROM PLAN

- One canonical aviary per account: Why: it makes multi-device sync "a property of the architecture," avoids client-to-client sync and merge, and keeps bird identity canonical.

- Two starter birds at adoption: Why: the plan wants new birds presented as "the birds that arrived," with coherent species chosen by the server rather than rarity weighting or catalog selection.

- Cap of seven birds: Why: the plan keeps the bird set cheap for per-aviary coupling and says raising the cap would require revisiting audio recognizability.

- Additional birds on aviary-age cadence: Why: growth is gated on aviary age, "never on visit count/score/payment," so it cannot become a gamified reward loop.

- Six-species pool size: NOT RECOVERABLE FROM PLAN

- Nightjar-like night-caller: Why: it supports the local day/night design where a nightjar species can be active at night.

- No catalog for birds: Why: new birds should arrive "in-flow" as a living aviary event, "not as a store."

- No achievements, streaks, scores, badges, levels, XP, or leaderboards: Why: the plan treats these as product-integrity failures and says no underlying cross-account metrics should exist to make them cheap to add.

- No user-behavior surface such as visit counts, calendars, or "you've been gone X days": Why: the risks section says these surfaces "announce, gamify, or surface user behavior" and each "breaks the product."

- No client-side simulation: Why: if the client ticked, "two devices would diverge and any merge corrupts drift."

### Architecture

- Five backend services plus a static client: Why: the services are "deliberately kept small," and the privacy boundary dictates datastore access, so boundaries are drawn on the data-access line.

- auth-svc: Why: it owns the account record, keeps email in the only place email lives, and issues the synthetic account UUID.

- aviary-svc: Why: it records events and serves snapshots while explicitly not computing drift, preserving sim-svc as the sole writer.

- sim-svc: Why: it is the only writer of `bird.personality_vector` and `bird.mood`, with no public HTTP surface, so canonical personality and mood cannot be client-authored.

- visit-svc: Why: it provides read-only visitor snapshots and never records presence or events from visitors, preserving the ambient visit boundary.

- telemetry-svc: Why: it is physically isolated from the simulation datastore to enforce the privacy commitment.

- Edge/CDN with an inlined first-state snapshot: Why: it supports time-to-first-bird and lets the render path draw the first bird before non-critical assets load.

- Snapshot as render boundary: Why: the client renders forward by interpolation and local procedural motion without inventing personality or drift.

- Thin reactive view layer: Why: the plan chooses a Preact/Solid-class layer for bundle size against the 2MB cap.

- Canvas2D or lightweight WebGL: Why: the plan defers the exact choice to the rendering spike and ties the choice to the scene renderer and bundle/performance budget.

- Web Worker for audio scheduling and interpolation math: Why: it protects the main-thread 60fps budget.

- Single server language/runtime: Why: the plan names staffing simplicity.

- Postgres for the simulation DB: Why: the plan wants row-level ownership, ordered event consumption, and transactional delta application.

- Durable queue for sim-svc ticks: Why: it drives scheduled ticks and absorbs backpressure.

### Data model

- Synthetic UUIDs everywhere: Why: email must never appear as a key, partition, shard, or log field.

- Email encrypted in one account record: Why: the plan calls this "the single most important boring detail" and uses it to enforce the privacy boundary.

- Account settings JSON: NOT RECOVERABLE FROM PLAN

- Session `device_label`: NOT RECOVERABLE FROM PLAN

- Aviary timezone: Why: it drives day/night and offers, and the risk section says canonical timezone keeps all devices agreeing through DST and travel.

- Stable `bird_id`: Why: it is "stable forever" and never reissued on rename, sync, or migration, preserving the bird the user knows.

- Bird `name` independent of id: Why: renaming should not affect stable identity.

- `personality_seed`: Why: it keeps drift "from baseline" auditable and supports an audit trail without making recomputation the read source of truth.

- Persisted `personality_vector`: Why: the plan says the vector is "persisted, never derived from history at runtime"; the event log drives the tick, but the canonical persisted vector remains source of truth.

- `last_offer_at` per offer type: NOT RECOVERABLE FROM PLAN

- Append-only `event_log` with per-aviary `seq`: Why: it defines processing order, prevents last-write-wins, and lets the tick apply additive server-authored deltas.

- Event log payloads never carrying absolute personality values: Why: the client can only say "user listened in," never "set boldness=0.62."

- `consumed_by_tick_seq`: Why: it makes event consumption idempotent.

- Read-only `notebook_entry`: Why: the field notebook is server-generated, sparse, immutable naturalist prose rather than an edit/delete/annotate surface.

- `visit_invite` with one-time token and revocation: Why: social access is per-invite, opt-in, revocable, and read-only.

- 30-day invite expiry: NOT RECOVERABLE FROM PLAN

- Silent `visit_log_entry`: Why: the plan allows host visibility into visits while keeping visits ambient and without co-presence, chat, avatars, or comments.

- Deliberately absent `streak`, `visit_count`, `last_visit`, `days_active`, `level`, `score`, `happiness`, `hunger`, `health`, negative drift, and cross-account aggregate tables: Why: absence enforces the non-goals and creates friction against later gamification or punishment.

### Simulation engine

- Advisory lock per aviary during a tick: Why: two workers should never tick the same aviary concurrently.

- Tick running for accounts with no connected client: Why: this is the "continues without the viewer" property.

- Idle aviary ticks with zero drift: Why: mood, time-of-day, and weather still advance while neglect contributes zero and never negative drift.

- Hidden five-trait personality vector: Why: the bird should express personality through behavior, not expose stats numerically.

- Exporting vectors only as opaque "expressiveness markers": Why: the plan resolves the tension between data portability and "never exposed numerically" by keeping export outside in-product display.

- Presence computed from visibility, focus, and recent activity: Why: it gives the "laptop open all night != watching" guarantee.

- Three-minute presence activity window: Why: the plan biases long because "watching birds without moving is the actual product."

- Twenty-second keepalive and sixty-second canonical tick: Why: the plan wants three pings per tick window and a canonical tick cadence.

- Server-side presence clamping: Why: a burst of events cannot inflate presence time.

- Visitor sessions never emitting presence: Why: ambient visits must not affect the host's simulation.

- Slow low-pass drift over presence and interaction signals: Why: a regular visitor becomes instrument-detectable in about a week and user-perceptible in about three weeks, while no single session visibly moves a trait.

- Monotonic drift clamp: Why: neglect contributes zero, never negative, and plumage saturation only ever rises.

- Perceptibility quantization: Why: expression changes only when a band boundary is crossed, making within-session invisibility structural.

- Greeting frequency dropping via mood/recency rather than trait decay: Why: the aviary can feel less responsive after absence without punishing the bird or decreasing traits.

- Exact mood enum `wary, content, curious, drowsy, alert, settled`: NOT RECOVERABLE FROM PLAN

- Mood persistence across sessions: Why: mood should not "snap to neutral" on tab open; the client renders the canonical mood in the snapshot.

- Mood timers: Why: they prevent flicker.

- Stochastic, personality-weighted mood transitions: Why: mood should never become a hard state machine the user can decode.

- Perch and pose derived from mood plus personality: Why: the user reads mood off motion with no label, tooltip, or icon, and perch remains signal rather than control.

- Call motif library with variation and preserved signature: Why: a call is "never identical twice," while each bird stays recognizable across mood and drift.

- Server call timing parameters with client-side synthesis: Why: the server owns canonical parameters, while local synthesis makes real-time chorus possible.

- Captions generated from the same call parameters: Why: the caption matches what was actually played.

- Bird-to-bird coupling: Why: it makes the aviary "a small social system, not a row of NPCs."

- Adoption with coherent server-selected species: Why: new birds should feel like "the birds that arrived," not a rarity-weighted choice.

- Growth offers appearing in-flow: Why: a new bird arriving should not feel like a store, and declining should be fine and re-offered later.

- Offer types of seed, song-fragment, and still-pool: NOT RECOVERABLE FROM PLAN

- Settle with 5s undo: NOT RECOVERABLE FROM PLAN

### Sync and error surfaces

- Multi-device clients reading the same snapshots: Why: there is no client-to-client sync, no client state to merge, and no eventual consistency to reconcile.

- No last-write-wins: Why: the lunch-phone-overwrites-morning-laptop failure is unreachable when clients cannot write personality vectors.

- Snapshot pulls on load, visibility return, resume gap, and visible keepalive: Why: returning clients resume from canonical state after hidden tabs or laptop suspend.

- Interpolating between snapshots: Why: birds move smoothly instead of teleporting.

- Matter-of-fact conflict and error copy: Why: system, identity, error, and settings surfaces are the named voice exception.

- Copy lint for naturalist tokens in system strings: Why: it enforces the matter-of-fact voice boundary.

### Frontend rendering pipeline

- Single horizontal three-perch scene: Why: the scene has no panning, scrolling, or zoom, and supports readable front/middle/back perch signal.

- Subtle parallax, sky, soft foliage, and occasional branch: NOT RECOVERABLE FROM PLAN

- Responsive scene that never crops a bird: Why: every viewport must preserve the bird as the primary subject.

- First frame with motion already in progress and no spinner: Why: this is a "graded affective contract"; a spinner-then-fade is an explicit failure.

- Quiet-field loading and empty state: Why: slow snapshots should still feel like an aviary field, never an app spinner; after first bird entry the user never sees an empty aviary again.

- Idle micro-motion: Why: birds remain continuously alive while the tab is visible, shaped by mood and pose hints.

- Pausing rendering when hidden: Why: it saves battery while the server keeps ticking.

- Reduced-motion cross-fade rendering: Why: it is a different "register," not "animations off," and retains calls, drift, mood, and notebook.

- Thin top bar with account/settings, accessibility settings, field notebook, and offer affordance: Why: the scene itself should have no inline labels, badges, tooltips, or overlay icons.

- Top-bar fade after cursor stillness: Why: chrome recedes so the aviary remains the surface.

- Day/night anchored to local time: Why: lighting should follow the user's local time while canonical timezone keeps devices coherent.

- Rare, non-assertive server-driven weather: Why: weather should be coherent across devices without becoming the point of the product.

- Leaf and feather drift as client-local ornament: Why: it is pure render and carries no server state.

- Object pools, no per-frame allocation, requestAnimationFrame, and hidden-tab pause: Why: they enforce 60fps and no memory growth.

### Audio pipeline

- Procedural WebAudio synthesis: Why: recorded loops are "the audible signature of dead software" and break the spell once heard twice identically.

- Per-call variation with preserved bird signature: Why: the user should know a bird by ear across mood and drift.

- Real-time chorus: Why: stacked recorded loops phase-cancel audibly, while independent synthesized voices can mix into a true chorus.

- Listen-in gradual mix ramps: Why: focus should re-balance the mix, not hard-cut or fully silence the other birds.

- Graceful silence with captions when WebAudio is unavailable: Why: the plan has an unconditional no recorded-audio fallback path, so captions carry the call information.

- Reused audio buffers/nodes and bounded audio contexts: Why: they feed the "no memory growth over 30 min" CI test.

### Accessibility surfaces

- Slow naturalist screen-reader narration: Why: the plan wants charm rather than state-list announcements such as "Pip mood: content."

- Narration cadence cap: Why: it protects the screen-reader queue while allowing faster updates for user-initiated events.

- Runtime call captions: Why: captions are generated from the same grammar parameters as synthesis and therefore match the actual call.

- Keyboard navigation through top bar, scene birds, listen-in, offer, and settle: Why: the main interactions must be fully keyboard-navigable.

- Soft high-contrast focus indicator: Why: it must remain legible against bright and dim aviary states.

- WCAG AA contrast on all user copy: Why: top-bar labels, settings, account/error surfaces, captions, and displayed narration must remain readable.

### Social

- Read-only ambient visits off by default: Why: visitors should receive snapshots without recording presence or events, preserving privacy and the host simulation.

- Per-invite email opt-in: Why: visit access is explicit and tied to a specific invited email, with the same encryption discipline.

- Revocable visit invites: Why: host control is part of the visit boundary.

- Optional visit-notification toggle off by default: Why: notifications stay opt-in and do not become a default behavior surface.

### Privacy, performance budgets, and observability

- Telemetry isolation with no simulation DB read grant: Why: per-bird and per-account interaction events must drive only that user's own simulation, never training, recommendations, third parties, or population analysis.

- Aggregate telemetry categories: Why: request counts, latencies, error rates, session-duration histograms, frame timing, and audio-context errors are allowed only without per-account dimensions.

- Privacy policy surface in account settings: Why: it names allowed telemetry categories and explicitly excludes per-bird state.

- Initial JS bundle under 2MB gzipped: Why: the first paint and first-bird target require a small bundle and code-splitting.

- Time to first bird under 500ms: Why: the aviary must draw the first bird before non-critical assets load.

- 60fps idle on a five-year-old laptop: Why: the plan treats sustained runtime smoothness as a budget, not just launch performance.

- Zero memory growth over 30 minutes: Why: pooled audio, released notebook rows, and bounded workers/contexts must be CI-enforced.

- Synthetic perf checks plus aggregate-only RUM: Why: scheduled headless browsers and aggregate timings catch load, frame, audio, and tick problems without per-bird inspection.

- Sim-tick p99 latency alarm over 5s: Why: it catches degradation before users feel "slow."

- Browser support for last two majors of Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN

### Rollout, calibration, and build sequence

- Single coordinated v1 launch with accessibility and reduced-motion included: Why: these are launch gates, not v1.1 work.

- Conservative birds-per-aviary ramp: Why: cap stays at seven, offers remain age-based, and aggregate health metrics are the only observation channel.

- Day-one instrumentation: Why: bundle size, first-bird timing, frame timing, tick latency, audio errors, and memory growth are product gates from launch.

- Synthetic-account calibration loop: Why: drift calibration is observed in staging through simulated presence schedules, never by reading real users' birds.

- Week-0 drift-calibration spike: Why: drift miss is the highest engine risk, and "everything else hangs off it."

- Week-0 audio spike: Why: procedural calls and chorus need listening tests for uncanniness, signature recognizability, and real-time mixing.

- Week-0 render spike: Why: the "loads-in-motion" and first-bird-under-500ms path are hard affective and performance contracts.

- Observability and budgets in CI before launch: Why: bundle gate, memory test, first-bird synthetic timing, tick alarm, voice lints, sole-writer checks, and privacy lints enforce the plan's boundaries.
