## System-level intent

- Server-owned, single-writer aviary. The plan repeatedly makes the browser a "renderer and an event forwarder" and says the server is "the only writer of personality, mood, and notebook state." This shows up in the service split, the event-log interaction model, the sync model, and the implementation order's "central conceit (server-side tick + snapshot + renderer with motion already in progress)."

- Calm, ambient, anti-gamification product voice. The plan excludes "achievements, streaks, levels, scores, badges, counters" and any "ping surface," then reinforces the same intent through a "quiet field" loading state, "naturalist observations," and notebook templates that are "of the aviary, not of the user's behavior."

- Slow relationship and gradual expressiveness. The plan's personality system drifts "over days/weeks," is "monotonic-toward-expressive," and should be "measurable in instruments after ~1 week" but visible only after "~3 weeks." Neglect "produces no signal" rather than punishment.

- Hard privacy and PII boundary. The plan names a "PII boundary hard-enforced in pipelines," requires a "synthetic UUID" as the only downstream identifier, stores email once and encrypted, separates telemetry from the simulation database, and says some metrics are not computed so "the architectural absence makes the feature's reappearance harder."

- Accessibility as a first-class designed surface. Accessibility is in v1 scope as "first-class designed surfaces"; reduced motion is "a designed surface, not a fallback"; and the plan says these surfaces ship "Not v1.1. Not a retrofitted fix."

- Procedural naturalism over canned content. The plan favors "procedural call grammars," "procedural plumage silhouettes," WebAudio synthesis, per-bird PRNG seeds, and call captions generated from the actual grammar. It forbids "recorded audio" and "canned loops."

- Recognizable individual birds in a small social system. Stable bird identity, per-bird PRNG motion so "Pip's idle is recognizably Pip's," no regenerate path, and bird-bird effects are all meant to make the aviary feel like "a small social system rather than parallel independent birds."

- Immediate living scene. The first frame should be "the aviary mid-motion," with an inlined initial snapshot and a "<500ms time-to-first-bird" budget. There is "no spinner," "no entry animation," and no "ready" pop.

- Optional, bounded social presence. Visits are "default off," read-only, revocable, and visitor sessions have "no write permissions"; visitor presence is "never recorded." The plan also rules out profiles, follows, feeds, discovery, comments, and friend-of-friend chains.

- Explicit calibration through instruments. The plan flags build-time choices as `CALIBRATION`, proposes "defensible default" values, and ties them to staging ramps, synthetic weeks of presence, CI gates, and "instrumented from day one" rollout checks.

## Per-feature whys

### Scope and product boundaries

- Single-user accounts, one canonical aviary, and multi-device sync. Why: sync is "a property of this architecture"; devices read the same canonical record instead of negotiating competing state.

- Email magic-link authentication with no passwords and no SSO at v1. NOT RECOVERABLE FROM PLAN

- Two starter birds on adoption. NOT RECOVERABLE FROM PLAN

- Aviary cap of seven birds gated by aviary age. Why: the rollout section says intervals should be tuned against "retention and per-bird recognizability feedback," keeping growth slow enough for recognizability.

- Six-species bird pool. NOT RECOVERABLE FROM PLAN

- Procedural call grammars and procedural plumage silhouettes. Why: the plan avoids recorded or canned content and wants variation "within recognizability," with species config driving calls, silhouettes, palette, and trait seeds.

- Personality vector with five traits and monotonic drift. Why: it creates slow expressive change while ensuring "No input ever subtracts" and no neglect penalty; drift should matter without becoming "users move numbers by clicking."

- Fast-timescale mood system persisted across sessions. Why: mood continuity means "the mood at session-end is the mood at the next session-start," while the tick continues whether or not a client is attached.

- Server-side simulation tick around once per minute. Why: the server remains canonical, the client never simulates, and the aviary can keep transitioning mood while no client is attached.

- Return-greeting. Why: it makes session return responsive to absence length, mood, and personality; greetings are weighted and staggered so birds do not fire "in unison."

- Listen-in. Why: it gives focused attention to one bird while other birds "never go silent"; the mix is a "re-balance, not a mute," and listen duration feeds drift.

- Offer interaction with seed, song fragment, or still pool. Why: offers produce immediate sub-tick feedback while the server remains authoritative for cooldown and later drift reconciliation.

- Settle interaction. Why: settle "ends the presence window cleanly," quiets calls globally, and changes lighting without pushing personality drift.

- Presence accounting. Why: only visible, focused, recently active time counts, preventing "tab is open" from inflating drift.

- Field notebook. Why: entries are "sparse," "read-only," and naturalist-style; frequency is tuned to "preserve sparsity," and templates observe the aviary rather than the user.

- Single horizontal scene and three perch zones. NOT RECOVERABLE FROM PLAN

- Local-time day/night cycle. Why: it supports ambient palette changes without a server round-trip, and mood transitions use time-of-day buckets.

- Ambient weather. Why: weather provides rare, short-lived scene input and mood input while remaining snapshot-driven so the renderer does not own weather state.

- Ambient micro-motion. Why: "Birds are never still"; motion expresses pose and mood through head-tilt, weight-shift, preen cycles, and scan sweeps.

- Top-bar chrome that fades. NOT RECOVERABLE FROM PLAN

- Visit-invitation social feature. Why: it allows a read-only ambient view without becoming a social network; invitations are revocable, default off, and have no notifications by default.

- Visit log. NOT RECOVERABLE FROM PLAN

- Account export as JSON. NOT RECOVERABLE FROM PLAN

- Soft-delete with a 30-day window and recovery. Why: deletion is recoverable during the soft-delete window before hard purge.

- Performance budgets. Why: the product depends on fast first bird, 60fps idle motion, fixed memory behavior, and small snapshot/bundle sizes, all enforced by CI or alarms.

- Synthetic account UUID, email stored once and encrypted, and PII boundary. Why: email is "never referenced as an identifier" downstream and even a database dump should not expose PII.

- No gamification. Why: the plan treats gamification creep as a risk and rejects even "harmless" engagement features so observations remain of the aviary.

- No Tamagotchi mechanics. Why: neglect should produce no signal, no visible suffering, and no decay pressure.

- No social network. Why: visits must not grow into profiles, follows, feeds, discovery, leaderboards, comments, or chains.

- No push notifications or system-driven pings. Why: the product should not reach out to the user except through the host-controlled opt-in visit notification.

### Architecture and storage

- Edge/CDN serving static assets, HTML shell, and inlined initial snapshot. Why: it supports the time-to-first-bird budget while keeping "No business logic" at the edge.

- Stateless API service. Why: it handles public REST/SSE surfaces while remaining "horizontally scalable."

- Simulation service as the only writer of personality and mood. Why: single-writer state avoids divergent personality, mood, and sync behavior.

- Async email service behind a queue. Why: the API "never blocks on SMTP."

- Operational telemetry service separated from simulation storage. Why: aggregate metrics must stay outside the simulation database and privacy boundary.

- Postgres as the v1 event log default. Why: it is "simpler operationally" and the plan says v1 is not at the scale where Kafka pays for itself.

- Object storage for exports and large immutable assets. Why: signed-URL delivery and large immutable assets do not belong in primary rows.

- Redis as optional ephemeral storage. Why: rate limits, idempotency keys, and magic-link attempt tracking are temporary; "No persistent state lives here."

- Snapshot schema as the renderer boundary. Why: anything needed for the next frame is in the snapshot, derivable from it, or static config in the bundle.

### Data model

- Accounts table with encrypted email and blind index. Why: the blind index enables uniqueness and lookup without reversible email use.

- Bird records with immutable `bird_id`. Why: stable identity must survive renames, sync, species-pool changes, and migrations.

- Personality vectors with hidden raw traits. Why: numeric traits are "never exposed" and the snapshot serializer must assert that no numeric trait field leaks.

- Mood state table. Why: mood persistence lets session start inherit the latest tick-computed mood.

- Append-only interaction events. Why: this is "the substrate the simulation tick consumes"; crashes can restart from the cursor and events are processed in `server_ts` order.

- Notebook entries as old, read-only rows. Why: the notebook is scrollable indefinitely and not an editable user document.

- Visit invitations with states, expiration, revocation, and encrypted visitor email. Why: visits are bounded, revocable, expiring access rather than an open social graph.

- Device sessions with revocation and coarse user-agent hint. Why: users can revoke sessions while storing only "coarse, non-PII" device hints.

- Magic-link token hashes with first-use consumption. Why: the secret is not stored raw, expires after 15 minutes, and further uses are rejected.

- Static species config mirrored server-side and shipped in the bundle. Why: both snapshot derivation and rendering/audio need the same species pool.

### API surface

- Magic-link request always returns 202. Why: it must "never reveal whether an account exists."

- Magic-link request rate limits. Why: the plan sets per-email and per-IP limits to control attempts.

- Snapshot endpoint. Why: it returns the small canonical state the renderer needs, cacheable only in a device-scoped private way.

- Server-Sent Events stream. Why: attached clients receive live deltas and can reconnect using `Last-Event-ID`.

- Batched event append endpoint. Why: client writes are append-only, idempotent, and routed through the event log.

- Interactions routed through `POST /v1/aviary/events`. Why: this "keeps the client dumb and the writer single."

- Listen-in endpoint. Why: listen-in has a synchronous audio flavor but still writes log events for drift.

- Offer endpoint. Why: offer needs server-authoritative cooldown and immediate deterministic reaction descriptor, with the tick reconciling later drift.

- Settle endpoint with undo. Why: the client can send `undo:true` in the 5-second window and the log marks the settle event reverted.

- Visitor resolve and read-only visitor snapshot/stream. Why: visitors can see the ambient view without write permissions or recorded presence.

### Snapshot schema

- Snapshot size in kilobytes. Why: the snapshot must support fast first render and stay below the serializer size budget.

- `*_render_hint` fields. Why: they are "the minimum the renderer needs" while avoiding raw personality trait exposure.

- Monotonic `snapshot_version`. Why: stale writes can be detected without dropping append-only events, and immediate reactions can be recomputed if needed.

- Visitor snapshot minus host-only fields. Why: visitors should see the aviary scene, not host settings, visit log, or anything beyond it.

### Simulation engine

- Per-account worker lease. Why: the system can shard workers while ensuring "never two workers on the same account simultaneously."

- Tick consumption in `server_ts` order. Why: server-received time is authoritative for additive deltas and conflict handling.

- Low-pass drift function with per-tick cap. Why: drift should be calibrated over weeks and a "single heavy session" must not move a trait too far.

- Monotonic-toward-expressive clamp. Why: "No input ever subtracts," so neglect cannot make a bird decay or visibly suffer.

- Mood finite-state machine. Why: time of day, weather, recent events, other birds, and personality make mood feel situated while still persisting across sessions.

- Wary spreading and chorus join. Why: bird-bird effects make the aviary feel like "a small social system."

- Presence ping payload duration. Why: the server can sum presence-time accurately "even across ping jitter."

- Return-greeting absence buckets. Why: greeting variants respond to absence length from less than two minutes through more than one day.

- Notebook generation in the simulation service. Why: it keeps voice consistent across devices and keeps the client bundle small.

- Stable identity unit test. Why: there must be no write path that creates a new `bird_id` for an existing bird.

### Sync model and error voice

- Single canonical state. Why: multi-device sync comes from both devices reading the same server record.

- No last-write-wins. Why: clients submit events, never absolute personality values, so there is no `set boldness to X` path.

- Append-only conflict surface. Why: simultaneous device writes both succeed and are ordered by `server_ts`.

- Snapshot-version stale hint for immediate reactions. Why: old client state can be handled gracefully without rejecting the event log append.

- Matter-of-fact sync/auth/account error voice. Why: operational errors stay separate from naturalist prose, with separate i18n namespaces so voices do not "accidentally bleed."

### Frontend rendering pipeline

- Modern web framework with streaming HTML and code-splitting. Why: the exact framework is secondary to small initial bundle and edge-rendered initial snapshot support.

- Custom 2D canvas compositor. Why: canvas gives the "frame budget for 60fps idle motion" and batched redraws; DOM is reserved for chrome and settings.

- WebWorker for audio. Why: audio scheduling should never block on main-thread jank.

- Scene layer ordering. Why: birds are drawn at perch-zone depth and ordered by `y`; captions and focus indicators remain DOM overlays.

- Idle micro-motion from snapshot pose, mood hints, and PRNG. Why: it makes each bird expressive, continuous, and recognizably itself without looping fixed animation.

- Snapshot interpolation. Why: birds move smoothly between perches instead of "teleporting."

- Reduced-motion renderer. Why: reduced motion remains aesthetic and complete: cross-fades replace motion, calls stay full quality, birds still drift, and mood still changes.

- Loading quiet field. Why: it reads as "the aviary catching up," not "the product loading."

- Empty-aviary quiet field and first soft fly-in. Why: after adoption, the user "never sees an empty aviary."

- Responsive scene. Why: narrow viewports compress without cropping birds, while wide viewports add perch space and keep all birds visible.

- Continuous day/night palette. Why: local-time palette changes should be continuous, not stepped, and avoid a server round-trip.

- Snapshot-driven ambient weather. Why: weather is rendered client-side but not stored by the renderer or owned by the tick.

- Ambient leaf and feather drift as client-side ornaments. Why: they are idle visual ornaments, not part of canonical simulation state.

### Audio pipeline

- WebAudio procedural call synthesis. Why: the same bird should not produce an identical call twice, and calls should vary by trait, mood, seed, and chorus context.

- Call scheduling from `call_render_hint`. Why: the audio worker can schedule the next calls from the snapshot rather than inventing state.

- Chorus bus mixing. Why: simultaneous procedural calls produce a "real chorus" without phase-canceling artifacts from layered loops.

- Listen-in gain ramp. Why: focus should feel like attention, not channel-switching; other birds "never go silent."

- Graceful silence if WebAudio is unavailable. Why: the plan unconditionally rejects a recorded fallback path.

- Settle audio. NOT RECOVERABLE FROM PLAN

- Fixed audio buffer/node pool. Why: audio must show "No memory growth" over a 30-minute synthetic session.

### Accessibility surfaces

- Screen-reader narration. Why: narration should be naturalist prose from the same snapshot, "not state-list," and use `aria-live="polite"` at a controlled cadence.

- Call captions. Why: captions are generated from the procedural call grammar so they match "what was actually played."

- Keyboard navigation. Why: listen-in, offers, settle, and bird focus must be reachable without a pointer.

- Focus indicators. Why: they must remain visible against both bright and dim aviary states.

- WCAG AA contrast. Why: all user-copy text, including captions and top bar text, must pass AA.

- Accessibility in v1. Why: the plan explicitly rejects treating it as v1.1 or retrofitted work.

### Performance, observability, and privacy

- CI and alarm-enforced budgets. Why: bundle size, snapshot size, memory growth, frame rate, first-bird timing, and tick latency are product constraints, not aspirations.

- Synthetic performance checks. Why: automated browsers from common geographies catch load, first-bird, frame, and audio-context problems.

- Aggregate-only RUM. Why: operational health can be measured without per-bird state or per-account interaction history.

- Deliberately unmeasured engagement-style stats. Why: not computing "most-visited aviaries" or "most birds" makes those features harder to reintroduce.

- Privacy policy link in account settings. Why: it names aggregate categories in plain text and explicitly excludes per-bird interaction state.

### Rollout and implementation order

- Closed beta for dozens of users. Why: the plan wants aggregate perf-budget checks and drift calibration sanity before broader exposure.

- Open beta for hundreds of users. Why: the plan watches for multi-device sync correctness, audio uncanniness, accessibility regressions, and memory-growth anomalies.

- Bird-count ramp during rollout. Why: availability intervals are tuned against "retention and per-bird recognizability feedback."

- Instrumented from day one. Why: drift, presence, memory, numeric-trait leakage, and perf behavior need tests before general availability.

- Foundation-first implementation order. Why: account model, event log, snapshot serializer, tick, and renderer establish the central conceit before peripheral surfaces.

- Accessibility before social and account surfaces in the suggested order. Why: the plan treats accessibility as part of launch, not a later fix.
