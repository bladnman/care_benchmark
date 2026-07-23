## System-level intent

- Enforced constraints over aspirational guidance. The plan repeatedly uses "hard rule" for boundaries that must be "enforced in code review and CI, not just documented." This shows up in the non-goals, accessibility, server-only canonical state, no client personality mutation, no raw personality values in snapshots, no recorded audio assets, telemetry exclusions, and copy lint.

- Smallest correct architecture. The architecture is a "modular monolith" because it is "the smallest shape that satisfies the hard architectural rules" without "distributing a system that doesn't need distribution yet." The same philosophy shows in "no Kafka at v1 scale," polling instead of WebSockets, Canvas 2D instead of heavier scene approaches, and replaceable module boundaries.

- Server-only canonical life, client-only presentation. The central boundary is that the "server owns" personality, mood, perch decisions, call scheduling parameters, weather, notebook entries, account state, and invite state, while the "client owns pixels and audio." The sim-worker "runs with zero connected clients" so "the aviary genuinely continues."

- Hidden personality, visible expression. The plan carries a hidden 5-trait vector, "monotonic-toward-expressive drift," and the rule that "trait deltas are never negative." Neglect produces "ambient expression" and "quietness, never punishment," so personality is felt through behavior rather than exposed as numbers or decayed like a Tamagotchi.

- Notice, never announce. Product surfaces should be ambient and naturalist, not explicit status messaging. The plan bans any "welcome back" text, toast, banner, or modal; says the notebook "observes the aviary, never the user's behavior"; and names the erosion risk as "'Notice, never announce' erosion." System surfaces use a separate "matter-of-fact voice."

- Privacy by structure, not dashboard discipline. Email appears "exactly once" and never in logs, telemetry, partition keys, or error messages. Observability collects "aggregate only" data and "never" collects per-bird state, per-account interaction history, or anything reconstructing a user's relationship with their aviary. Drift calibration runs on "synthetic harness accounts," never production data.

- Accessibility as part of the product. The plan says "accessibility ships with v1, not after" and that an accessibility review is "a launch gate, not a follow-up." Reduced motion is "a designed surface," narration and captions come from shared state/prose, and accessible surfaces are described as "the actual product for those users."

- Procedural variation with durable bird identity. Calls use motif libraries, identity seeds, and render-plans so "Pip" remains "recognizable as Pip" while "every call is a fresh variation." The plan also makes `bird_id` and identity seed immutable, treating identity and traits as "immutable-key data."

- Performance as product feel. Budgets are "CI-enforced, not guidelines," and the boot path aims for an aviary "already in motion." The first frame draws birds mid-preen or mid-call, avoids spinners and static fades, and uses an inlined snapshot to hit "<500ms time-to-first-bird."

- Sync conflicts designed out. Multi-device sync is "a non-feature" because clients pull one canonical record. There is "no client-to-client sync, no merge, no CRDT, no eventual consistency," and "data conflicts as a category are designed out, not resolved."

- Social as opt-in ambient access, not a network. Visit invitations are email-based, default OFF, read-only, revocable, and visitor attention "never feeds the host's drift." The plan explicitly bans profiles, follows, feeds, discovery, comments, chat, leaderboards, and the aggregate stats that would make leaderboards computable later.

- Calibration through harnesses where the PRD is open. The introduction says open ranges and cadences are "calibration targets," not guesses. Drift, tick cadence, activity window, adoption timing, and bird cap are all tuned through synthetic personas, listening protocols, and CI gates rather than production-data mining.

## Per-feature whys

### Scope

- Web-only modern-browser platform: NOT RECOVERABLE FROM PLAN

- No native clients and no protocol concessions for hypothetical native clients: NOT RECOVERABLE FROM PLAN

- Two system-selected starter birds per new account: NOT RECOVERABLE FROM PLAN

- Species pool of about six: NOT RECOVERABLE FROM PLAN

- Aviary-age-gated adoption offers up to seven birds: The plan ties adoption pacing to aviary age and the hard cap to recognizability: "the cap follows recognizability, not the reverse," with a listening protocol before any pacing change.

- Hidden 5-trait personality vector: The plan keeps personality hidden because in-product numeric exposure is banned: "no stats panel, no debug toggle, no tier," and the snapshot "never contains raw personality values."

- Monotonic-toward-expressive drift driven primarily by presence: The plan wants regular presence to become measurable and visible over time while avoiding punishment; "neglect yields ambient quietness, never punishment" and "drift never moves a trait downward."

- Five-state mood with daily-ish cadence and persistence: The plan uses a "soft re-centering pull" so moods "feel fresh each day without snapping," and stores mood so "session boundaries never reset it."

- Procedural per-bird call grammar: The plan wants calls that are recognizable but non-repetitive: an immutable identity seed keeps a bird recognizable, while "no two calls render identically."

- Bird-to-bird call response, mood contagion, and emergent chorus: The plan describes chorus as "emergent, never scripted," with response probability shaped by social warmth, vocal frequency, and mood.

- Return-greeting: The plan replaces explicit welcome-back surfaces with bird behavior; greeters are scored by social warmth, boldness, absence-length factor, and mood suitability, and the hard rule is "never a simultaneous unison greeting."

- Listen-in gradual mix rebalance: The plan raises the focused bird but keeps others at an "ambient floor" that is "never silence," preserving the aviary while focusing attention.

- Offers of seed, song fragment, and still pool with per-bird cooldown: NOT RECOVERABLE FROM PLAN

- Settle with opt-in and 5-second undo: The plan says settle is "engine-equivalent to tab-close"; its drift role is only that "it ends the presence window cleanly."

- Field notebook, auto-generated, read-only, sparse, indefinite scrollback: The notebook "observes the aviary, never the user's behavior"; sparsity and read-only naturalist prose prevent it from becoming an event log, streak counter, or user-behavior surface.

- Presence accounting as strict three-signal conjunction: The plan calls this precision "load-bearing" and says lax checks risk inflated population drift; background-tab and unfocused-window scenarios must produce zero presence.

- Single horizontal scene with no pan, zoom, or scroll: NOT RECOVERABLE FROM PLAN

- Three perch zones chosen by birds, never by the user: NOT RECOVERABLE FROM PLAN

- Local-time day/night cycle with a nightjar-like nocturnal species: The plan uses account timezone as a mood input and keeps the nocturnal species active while most birds drift drowsy or settled at night.

- Rare ambient weather: Weather is server-scheduled so "all devices and visitors see the same sky," with brief, small effects on vocal dampening and alert/wary nudges.

- Ambient micro-motion as client-side ornaments only: The plan keeps leaves and feathers "pure client-side" and "never simulated, never synced" because ornaments are outside the canonical model.

- Thin top bar fading to near-transparent on cursor stillness: NOT RECOVERABLE FROM PLAN

- First frame renders mid-motion plus quiet-field loading: The plan wants the aviary "already in motion," with birds mid-preen or mid-call, "never a spinner," never fade-from-static, and no entry animation except adoption fly-in.

- Email-based visit invitations only, default off and read-only: The plan avoids social-network surfaces, keeps visits opt-in and revocable, and ensures "visitor attention never feeds the host's drift."

- Accessibility surfaces shipping with v1: The plan states accessibility is a launch gate because "accessible surfaces are the actual product for those users" and "retrofitting is the named failure mode."

### Architecture

- Modular monolith with api-server, sim-worker, and mailer: The plan calls this "the smallest shape" that satisfies server-only canonical state and ticks with no clients connected without unnecessary distribution.

- TypeScript end-to-end with shared `sim-core` and `protocol`: Sharing keeps narration, captions, notebook prose, and call render-plans from "one" generator, and supports shared-language velocity.

- Node 22 LTS with Fastify or equivalent: The plan chooses it for "shared-language velocity" and because the sim-worker is "CPU-light at v1 scale."

- Postgres 16 as system of record and append-only interaction log: The plan keeps all canonical state in Postgres, avoids Kafka because it is not needed "at v1 scale," and lets the tick consume ordered events by watermarks.

- Vite, TypeScript, Canvas 2D, and small DOM chrome: The plan chooses this for the "<2MB gz" budget and "full control over the render loop"; "the scene is one canvas, not a component tree."

- Transactional mailer behind an internal interface: The plan asks for "idempotent send + delivery logging" for magic links, invites, and export links.

- CDN edge with inlined boot snapshot: The plan uses this to draw the first bird without waiting on the larger JS chunk graph and to support time-to-first-bird.

- Server/client split at the render pipeline boundary: This makes the PRD's "central architectural rule" concrete: server writes canonical state; client renders pixels and audio; no client path mutates personality.

- Shared render-plan and prose generation: "One plan, three expressions" guarantees the caption matches what was actually played and keeps notebook/narration phrasing aligned with state.

### Data model

- Synthetic `account_uuid` for every internal reference and encrypted email exactly once: The plan's rationale is privacy and containment: email never appears in logs, telemetry, partition keys, or error messages.

- Account settings for timezone, visit notifications, reduced motion, captions, and audio mute: NOT RECOVERABLE FROM PLAN

- Per-device revocable sessions: NOT RECOVERABLE FROM PLAN

- Magic links with 15-minute expiry, single use, and rate limits: The plan uses these to validate only unexpired unused links and to avoid account enumeration with a 202 response.

- Verified email change that commits only after new-address verification: The plan states the switch commits "only after new-address verification" and the old email works until then.

- JSON account export including birds, names, current personality vectors, moods, notebook entries, and settings: The plan treats export as account management and an "additional escape hatch" while keeping vectors numerically invisible in-product.

- Thirty-day soft deletion followed by hard deletion: NOT RECOVERABLE FROM PLAN

- Immutable `bird_id`: The plan says renames, migrations, and species-pool changes never replace a bird; migration tests assert continuity because "personality-vector loss or bird-identity break" is a worst invisible failure.

- Adoption offers generated from aviary-age thresholds: The plan uses age thresholds as the pure source of offers and caps total birds at seven so chorus recognizability remains plausible.

- Append-only `interaction_events` with `client_event_id` deduplication: The plan needs idempotent ingest against client retries and replayable ordered input for the tick.

- Materialized `presence_windows`: The plan makes this "the only presence representation the drift function reads," separating validated presence from raw pings.

- Notebook `generator_key`: The plan uses it for "sparsity budgeting and dedup" so entries stay novel and sparse.

- Internal-only `drift_observations`: The plan needs this for the calibration harness to verify "measurable after one week" while never rendering it, exposing it through APIs, or including it in export.

- Silent read-only visit log: The plan surfaces who, when, and approximate duration in settings without creating social surfaces or feeding visitor attention into drift.

### API surface

- Plain HTTPS + JSON with cookie-session auth: NOT RECOVERABLE FROM PLAN

- Polling transport instead of WebSockets: The plan says the canonical cadence is 60s, snapshots are kilobytes, and polling on visibility changes plus keepalive is "sufficient and radically simpler to make correct."

- Matter-of-fact API error surfaces: The plan reserves naturalist voice for product surfaces and uses matter-of-fact copy for sign-in, settings, sync errors, accessibility settings, and similar system surfaces.

- Magic-link issue and consume endpoints: The 202 response prevents account enumeration; consume validates single-use unexpired tokens and maps expired, used, or replayed links to stable matter-of-fact errors.

- Account settings endpoint with no aviary state: The plan keeps settings separate because the server owns aviary state and no settings patch may carry canonical bird/personality state.

- Account export by queued request and emailed time-limited link: The plan uses queued export generation and mailer delivery for account data without turning export into a product surface.

- Account deletion and recovery endpoints: NOT RECOVERABLE FROM PLAN

- Snapshot endpoint and inlined initial snapshot: The plan uses the same canonical snapshot for pull sync and first paint so the client can draw immediately and recover staleness.

- Event ingest as the only interaction write path, batched every about 5 seconds and on pagehide: The plan makes interaction writes append-only and idempotent, with `sendBeacon` covering pagehide and retries covered by `client_event_id`.

- Notebook pagination by cursor: NOT RECOVERABLE FROM PLAN

- Offer catalog with seed, song fragments, and still pool: Song fragments are motif parameterizations rather than audio files, preserving the zero-recorded-audio rule.

- Adoption response and bird rename endpoints: NOT RECOVERABLE FROM PLAN

- Invite creation, revocation, visit tickets, and visit snapshots: The plan scopes tickets to `read:snapshot`, checks invite status on every pull, and makes revocation effective within one poll interval.

- Visitor clients post no events and gateway rejects visit-ticket writes: The plan ensures visitors write nothing, accrue nothing, and never feed the host's drift.

- Presence ping payload and server validation: The plan validates visibility, focus, and activity, rejects impossible presence, clamps overlaps, and ignores non-monotonic timestamps to protect drift precision.

- Snapshot schema with `plumage_render_tier` but no raw trait values: The plan allows bucketed render hints while keeping trait numbers only in the sim DB and the user's own export.

### Simulation engine design

- Pure deterministic `sim-core` with per-account seeded RNG: Replay invariance is "CI-tested" and is "the backbone of drift-calibration science."

- Sixty-second sim tick: The plan chooses a calibration-target cadence that runs with zero clients, folds events, advances state, and keeps p99 tick latency below 5 seconds.

- Tick writes all state in one transaction: The plan uses one transaction to consume events, advance simulation, generate notebook candidates, bump snapshot version, and preserve coherent canonical state.

- Drift as per-trait low-pass filter over signals: Presence is dominant, listen-in and offers contribute, and `drift_carry` keeps "slow-but-steady users" from being rounded down to zero.

- Per-day caps and marathon-persona gates: The plan wants regular visitors to show movement while a 4-hour marathon followed by nothing must not cross visible thresholds.

- Expression scaling for absence: The plan makes an ignored bird read quieter on return "with its traits intact," avoiding downward drift and Tamagotchi punishment.

- Mood as semi-Markov state with inputs from events, time, weather, ambient calls, and personality: The plan uses these inputs so moods evolve through absence and are not reset by sessions.

- Soft nightly mood re-centering: The plan says this makes moods feel fresh each day "without snapping."

- Motif libraries and immutable identity seeds: The plan uses motif variation for freshness and identity seeds so each bird remains recognizable across mood and drift.

- One render-plan driving audio, captions, and notebook phrasing: The plan says this guarantees "the caption matches what was actually played."

- Rolling-window call response and chorus: The plan lets other birds respond based on social warmth, vocal frequency, and mood, producing chorus as "emergent, never scripted."

- Wary and content contagion: The plan uses alarm-class calls and chorus to nudge mood transitions across nearby birds.

- Greeting arbitration: The plan scores candidate greeters and staggers other greetings so return-greeting never becomes simultaneous unison.

- Server-scheduled weather: The plan makes weather canonical so "all devices and visitors see the same sky."

- Notebook observation candidates from state diffs: The plan ranks candidates by novelty and discards weak ones silently to keep the notebook sparse and observational.

- Notebook prose templates with banned user-observing patterns: The plan limits prose to birds, weather, light, and time, never the user or trait numbers, so the notebook does not become a streak counter or event log.

### Sync model

- One canonical record in Postgres: The plan makes multi-device sync "a non-feature" because every signed-in client pulls the same snapshots.

- No last-write-wins path for personality: The plan removes trait-carrying write messages and keeps updates in the tick's additive event-log consumption path.

- Designed-out data conflicts: The plan says user-visible conflicts are limited to auth/session failures and that data conflicts are "designed out, not resolved."

- Snapshot pull on load, visibility change, suspend gaps, and keepalive: The plan uses schema and monotonic state version so clients can detect staleness and re-pull.

- Visitor revocation at next pull: The plan uses revocable tickets and poll-interval checks so access ends within one poll interval.

### Frontend rendering pipeline

- Canvas 2D retained layer graph: The plan uses layered sky, foliage, perch zones, birds, ornaments, and weather to support subtle parallax and controlled rendering.

- Bird vector sprite parts with plumage detail tiers: The plan keeps assets small for the 2MB budget while species silhouettes make birds "distinguishable at a glance."

- DOM top bar, dialogs, and settings: The plan uses DOM because it gives "real focus" and "real semantics."

- Top bar fading to about 5 percent opacity after cursor stillness: NOT RECOVERABLE FROM PLAN

- Boot with critical CSS, tiny boot script, and inlined snapshot: The plan uses this to render from current state immediately.

- Backdated activity clocks at first paint: The plan wants birds to appear mid-preen and mid-call, preserving the "already in motion" feel.

- Quiet field fallback: The plan uses a soft local-time sky and faint motion cues when a snapshot is slow, "never a spinner" or static fade.

- Return-greeting within the first one to two seconds: The plan ties this to the snapshot `greeting` block so the greeting is state-driven.

- Mood-keyed idle state machines: The plan maps wary, content, curious, and drowsy to visible behavior so mood is expressed through posture, scanning, preening, head tilts, and slow blinks.

- Procedural blink and fidget jitter: The plan uses it so "no two idle loops look alike."

- Server-decided perch moves with client choreography: The plan keeps the server authoritative about that a bird moves while the client controls how it animates, smoothing snapshots so birds "never teleport."

- Hidden-tab render suspension with server-side simulation continuing: The plan saves client work while keeping the aviary continuing through absence.

- Responsive scaling with all birds in frame: NOT RECOVERABLE FROM PLAN

- Reduced-motion cross-fade driver: The plan treats reduced motion as an alternate `MotionDriver` where audio, drift, mood, notebook, captions, and narration are unchanged.

### Audio pipeline

- Client-side WebAudio synthesis for all calls: The plan enforces "zero recorded audio assets" so the soundscape is procedural and not loop-based.

- Per-bird voice chain with formant filtering and identity seed: The plan keeps per-bird signatures recognizable while varying note-level pitch, duration, and level.

- Pooled buffers and nodes: The plan uses pooling because of the no-growth memory budget.

- Chorus bus compression: The plan keeps a seven-bird chorus legible and preserves per-bird signatures, which is the empirical basis of the seven-bird cap.

- Listen-in mix ramps: The plan focuses one bird while keeping the rest of the aviary audible at an ambient floor that is "never silence."

- Client call scheduling from hints with local jitter: The plan lets audio render locally while canonical state remains independent of whether calls actually render.

- Autoplay-blocked audio as silent start plus small speaker affordance: The plan treats this as a platform constraint "surfaced quietly," not an announcement surface.

- WebAudio failure as graceful silence plus captions default ON: The plan preserves accessibility and avoids any recorded fallback path.

### Accessibility surfaces

- Screen-reader narration through an aria-live polite region: The plan generates naturalist prose from the same snapshot state the canvas renders.

- Narration queue fed by snapshot interpolation: The plan wants narration to describe "what is on screen, not raw state."

- Call captions from the call render-plan: The plan derives captions and sound from one plan so they "always agree."

- Keyboard navigation and focus for top bar, scene birds, listen-in, offers, and settle: The plan makes the full experience keyboard-operable with real roles, labels, visible focus, and an off-DOM list for canvas birds.

- WCAG AA contrast: The plan sets AA as the minimum for all user copy across chrome, dialogs, captions, narration if displayed, settings, and errors.

- Naturalist versus matter-of-fact voice split: The plan enforces product surfaces as naturalist and system surfaces as matter-of-fact through copy deck and review.

### Performance budgets and observability

- Initial JS budget below 2MB gzipped: The plan enforces it with a bundle-size CI gate and code-splitting for non-first-paint flows.

- Time to first bird below 500ms: The plan uses synthetic release runs, edge-inlined snapshots, and a render path that does not block on non-critical assets.

- Sustained 60fps idle motion and no memory growth over 30 minutes: The plan enforces long-session frame timing and heap-delta tests, including pooled audio buffers and notebook virtualization.

- Sim tick p99 below 5 seconds: The plan uses server metrics and alarms because staleness would break the continuing-without-you conceit.

- Aggregate-only operational measurement: The plan collects request and timing data while refusing per-account dimensions or per-bird state.

- Never-collect telemetry exclusions: The plan bans anything reconstructing the user's relationship with the aviary and isolates telemetry with explicit allowlists and no path to the simulation database.

- Drift calibration on synthetic harness accounts: The plan reconciles calibration with privacy by never using production data for drift tuning.

### Rollout

- Foundations before full product buildout: The plan starts with repo, protocol, schema, auth, ingest, snapshots, deploy skeleton, and CI gates so later work rests on the hard architectural rules.

- Engine core overlapping foundations: The plan brings up drift, mood, weather, call scheduling, tick worker, and synthetic-time calibration early so calibration risk is addressed during implementation.

- Accessibility in parallel, not trailing: The plan makes accessibility a launch-gated phase running alongside product work rather than a post-launch pass.

- Closed beta before open web launch: The plan waits for performance gates, synthetic fleet, and error budgets before opening the web launch.

- Bird-count ramp: The plan says the seven-bird cap is in the engine, but adoption unlocks by aviary age and any pacing change must pass chorus-recognizability listening.

- Day-one synthetic fleet, aggregate RUM, tick alarms, audio counters, drift harness, kill switches, and dark-launchable visits: The plan uses these to catch performance, calibration, and operational failures before users feel them, while keeping visits default-off forever.

### Risks

- Drift miscalibration mitigations: The plan says the product "lives in the narrow band," so it uses synthetic-persona CI, per-day delta caps, expression scaling, and harness-only tuning.

- Presence-signal corruption mitigations: The plan calls precision load-bearing, so it centralizes the three-signal conjunction, validates on the server, biases the activity window long, and tests background/unfocused zero presence.

- Sync correctness regression mitigations: The plan prevents future trait writes through protocol shape, schema rejection, architecture review, and replay-invariance tests.

- Audio uncanniness mitigations: The plan says one repeated-sounding call "breaks the spell permanently," so it budgets motif breadth, enforces distinct render-plans, and tests recognizability.

- Accessibility regression mitigations: The plan treats accessible surfaces as the product for those users and uses launch gates, snapshot tests, visual diffs, keyboard e2e, and contrast checks.

- Notice-never-announce erosion mitigations: The plan bans welcome, streak, and visit-frequency copy and uses copy-deck review plus voice checklists.

- Personality-vector loss or bird-identity break mitigations: The plan treats reset birds as an invisible failure and relies on point-in-time recovery, migration continuity tests, export, and immutable identity data.

- Tick staleness mitigations: The plan uses p99 alarms, shard lag metrics, and worker autoscale triggers because slow ticks break the continuing-without-you conceit.

- Scope creep toward gamification/social mitigations: The plan quotes non-goals in PR templates and prevents leaderboard-computable stats from existing.

- Notebook voice collapse mitigations: The plan uses template lint, sparsity budget, and editorial review so notebook prose does not expose itself as an event log.
