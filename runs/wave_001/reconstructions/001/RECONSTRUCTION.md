## System-level intent

- **Architectural absences are product design.** The plan says the out-of-scope items are "not just deferred features" but "architectural absences," and that the data model and metrics pipeline should make gamification, Tamagotchi mechanics, social network surfaces, and similar features "harder to add later, not easier." This also shows up in the anti-feature checks, the forbidden strings, and the "engagement-feature creep" risk.

- **The server owns the aviary; the client renders it.** The load-bearing line is repeated in Architecture, Sync Model, and the API: "The client never owns canonical anything," clients submit "events, not state," and the simulation service is the only writer of personality, mood, perches, scheduled calls, and notebook entries.

- **Personality should be expressed, not exposed.** The plan keeps personality vectors server-only and says no personality vector values appear in snapshots. The protocol exposes categorical "render_hints" so a user sees "plumage_tier" rather than "plumage_saturation." The plan frames this as the implementation of "personality is never numeric to the user."

- **Drift is monotonic, slow, and central.** The drift function has an "asymmetry rule" where delta is always non-negative, traits asymptote instead of maxing out, and visible change is calibrated over weeks. The Risks section says wrong drift "invalidates the product's central claim."

- **Privacy is architecture, not an addendum.** The plan says "This is part of the architecture, not an addendum." Email lives in exactly one place; synthetic UUIDs are the only identifiers crossing service boundaries; there is no analytics warehouse ETL for per-bird or per-account state; metrics have dimension allowlists and denylist checks.

- **The product voice is calm, naturalist, and matter-of-fact.** This shows up in "naturalist prose" for notebook and screen-reader narration, in "matter_of_fact" error text rendered verbatim, and in out-of-band notes saying the naturalist voice is "load-bearing" and needs a writer-of-record review.

- **Sparsity and quietness are intentional.** The notebook section states "Sparsity is a feature, not a bug," caps very active users at about 3 entries/week, and forbids "you," "every day," "this week," "streak", and "visit" in templates. This is connected to the plan's refusal of feed and engagement-counter behavior.

- **Accessibility is a designed surface, not a fallback.** The plan says accessibility is "a designed surface of v1, shipped on day one," and reduced-motion mode is "not a degraded fallback" but "the same product, slower." Screen-reader narration, captions, keyboard navigation, contrast, and live settings are all treated as designed product surfaces.

- **Performance is part of the emotional contract.** The plan ties inline snapshots, first-frame rendering, bundle budgets, and synthetic perf fleets to the "first bird visible" and "first frame is alive" promises. It warns that a spinner during refactor would break that promise.

- **Determinism is a safety tool.** Tick output, call generation, screen-reader narration, ambient particles, and replay testing are seeded so a given state can be reproduced. The plan says this makes replay testing trivial, bug investigations safe, and migrations possible without ambiguity.

- **Recognizability matters more than expansion.** The 7-bird ceiling, motif limits, identifying motifs, and bird-count ramp are all justified by species recognizability and chorus sustainability. The plan says if recognizability collapses at 5, "We don't raise it."

- **Future pressure is expected, so defenses are concrete.** The plan anticipates "casual feature ask[s]," "harmless" engagement requests, and contributors adding quick client writes. It responds with schema absences, CI denylist checks, review checklist items, runtime invariants, and PR prompts like "what does this teach the user?"

## Per-feature whys

### Scope

- **Single-user accounts.** NOT RECOVERABLE FROM PLAN

- **Magic-link email auth.** The plan shapes this around privacy and enumeration resistance: `POST /auth/magic-link` returns "202 always (no enumeration)," and the auth service owns "the email-to-UUID mapping (the only place email is stored)."

- **Per-device sessions and session revocation.** NOT RECOVERABLE FROM PLAN

- **Account export.** NOT RECOVERABLE FROM PLAN

- **Soft-then-hard delete with undelete.** The plan's rationale is a 30-day recovery window: deletion "marks soft-delete" with "immediate UI lockdown but data preserved 30 days," and `POST /account/undelete` exists during that window.

- **One canonical aviary per account.** The rationale is the canonical-state model: the server owns canonical state, the client never owns canonical anything, and the architecture intentionally excludes multi-aviary and shared-aviary surfaces.

- **Two starter birds.** NOT RECOVERABLE FROM PLAN

- **Cap of 7 birds.** The plan ties this to recognizability and bounded rendering/audio complexity: motifs are limited "to keep recognizability high," the render loop has a fixed max of 7 birds, and the rollout will lower the cap if "recognizability collapses earlier."

- **Native apps, gamification, Tamagotchi mechanics, social network surfaces, push notifications, payments, multi-aviary accounts, shared aviaries, public discovery, recorded-audio fallback, and animated entry/loading transitions left out.** These are "architectural absences" so the data model and metrics pipeline make them harder to add later.

- **Forbidden strings for welcome-back, streak, levels, achievements, badges, and xp.** The plan uses these as concrete design-review and CI defenses against engagement-feature creep and the product's "affective contract" leaking.

- **No code path surfacing personality vectors to the client.** The rationale is that personality should not become numeric to the user; snapshots use categorical render hints and "No personality vector values appear in either" projection.

- **No code path writing personality vectors from the client.** The rationale is sync correctness and central authority: the simulation service is the only process that mutates personality vectors, and future client-side write paths are called out as a risk.

- **Aggregate-only operational telemetry.** The rationale is privacy: no per-bird or per-account state should enter analytics, metrics, third-party tools, or relationship-reconstructing dimensions.

- **Browser support for latest two majors of Chrome, Safari, Firefox, and Edge.** NOT RECOVERABLE FROM PLAN

- **Accessibility as designed surface.** The plan says these surfaces ship on day one and pairs design with engineering: naturalist screen-reader narration, reduced-motion mode, captions, full keyboard nav, and WCAG AA contrast.

- **Return-greeting interaction.** NOT RECOVERABLE FROM PLAN

- **Offer interactions: seed, song-fragment, still-pool.** NOT RECOVERABLE FROM PLAN

- **Settle interaction with 5-second undo.** NOT RECOVERABLE FROM PLAN

- **Field notebook.** The plan frames it as sparse, read-only, naturalist observation rather than a feed: it listens to canonical state changes, emits only salient observations with cooldowns, and never emits user-behavior observations.

### Architecture

- **Five logical services.** The plan separates "load-bearing" services from conventional ones: Edge/Web makes first-bird performance possible, while Simulation owns the tick and canonical mutation.

- **Edge/Web serving an initial state snapshot in HTML.** The inline snapshot is the trick that "makes the 500 ms first-bird budget reachable."

- **Auth service as the only email owner.** The rationale is privacy isolation: email-to-UUID mapping is owned in one place, with email encrypted and lookup via non-reversible HMAC.

- **Aviary API accepting interaction events into an append-only log.** The rationale is canonical-state sync: clients submit events, not state, and the API "Never" writes personality vectors.

- **Simulation service as pure consumer and canonical-state writer.** The plan says it owns the tick, is single writer per account, and is the only process mutating personality vectors.

- **Notebook service as read-only client surface.** It listens to canonical state and emits naturalist-prose entries on a "sparsity-controlled schedule," keeping clients read-only.

- **Client/server split.** The plan calls this "the load-bearing line": server owns personality, mood, simulation time, notebook, account records, sessions, invitations, and event log; client owns rendering, audio mix, focus, reduced-motion preference, input capture, and transient presence accumulation.

- **State snapshots instead of animation timelines.** The rationale is small protocol and stable rendering: snapshots are typically under 8 KB, two snapshots plus interpolation render any frame, and stale snapshots hold last-known state until resume.

- **Synthetic UUID v7 IDs.** The plan gives the rationale directly: v7 is "time-ordered for index locality," and email is never used as a key.

- **Encrypted email plus HMAC lookup hash.** The rationale is that email is stored once, encrypted at rest, and the lookup hash is "not reversible."

- **Bird render hints persisted.** The plan says these are "derived, but persisted to keep snapshots cheap."

- **Personality fields as floats but never serialized.** The rationale is enforced projection: the schema keeps floats internally and the snapshot shape prevents numeric personality from reaching client surfaces.

- **Append-only event log.** The rationale is deterministic ordered simulation: ticks consume events in `(account_id, occurred_at, id)` order, idempotency uses `client_seq`, and a high-watermark marks processed events.

- **Event retention and pruning after 90 days.** The plan's call says 90 days "balances drift transparency with storage."

- **Notebook `trigger_kind`.** The rationale is QA only: it exists "so we can verify mix," not for client display.

- **Invitation and visit records.** The rationale is controlled read-only visiting: invite state tracks expiry, revocation, first consumption, and visitor sessions never write events to the host log.

- **Bucketed visit duration.** The plan explicitly calls this to "discourage exact stalking."

- **Postgres partitioned event log.** The rationale is operational simplicity: the plan considers Kafka plus materialized table but chooses Postgres until volume forces otherwise.

- **REST plus JSON with WebSocket optimization and HTTPS fallback.** The rationale is snapshot freshness without making WebSockets mandatory: "everything also reachable over plain HTTPS."

- **Bounded event batches.** The rationale is flood prevention: the server rejects batches beyond a small call of 32 "to prevent flooding."

- **Bird rename outside the event log.** The rationale is that rename has "no drift impact."

- **Snapshot projections with coarsened render hints.** The rationale is rendering without exposing floats: categorical buckets drive UI while hiding numeric trait values.

- **Visit snapshot projection.** The rationale is host privacy: visitor snapshots remove `host_session` and host-only fields.

- **Visitor revoke response.** The plan chooses a matter-of-fact 410 Gone with "visit no longer available," so revoke takes effect cleanly at the next snapshot pull.

- **JSON error envelope with `matter_of_fact`.** The rationale is product voice and instrumentation separation: the client renders the text verbatim; the code is for instrumentation only.

### Simulation Engine

- **60-second simulation tick.** The plan calls the tick "the heart of the product working at all" and requires deterministic, single-writer, bounded-cost execution.

- **Idle accounts dropping to 5-minute heartbeat ticks.** The rationale is bounded system load: active accounts times one per minute, not all accounts times one per minute.

- **Per-account-shard scheduler with one tick in flight.** The rationale is single-writer safety and no merge semantics.

- **Small, pure, unit-testable tick substeps.** The tick algorithm is decomposed so each substep is "small, pure, and individually unit-testable."

- **Non-negative drift delta.** The rationale is the asymmetry rule: the code should never contain a negative drift coefficient or "`-=`" in that function.

- **Saturating trait responsiveness curve.** The rationale is gradual expression: traits move fastest mid-range and "barely at all near 1.0," so birds asymptote instead of instantly maxing out.

- **Drift calibration over weeks.** The plan wants one week just-detectable in instruments and three weeks just-visible to a user, with synthetic harness fitting before launch.

- **Mood finite state machine with weighted transitions.** The rationale is personality-shaped mood behavior while keeping transition data in version-controlled JSON with calibration tests.

- **Mood persistence across sessions.** The rationale is that the server has no concept of "session"; end-of-tick mood is start-of-next-tick mood.

- **Server-scheduled call windows.** The server decides when calls happen so vocal-frequency trait, mood, weather, and time-of-day remain canonical.

- **Client-generated audio waveform.** The rationale is bundle and runtime split: the server does not ship audio, and the client does not decide when a bird calls.

- **Overlapping call windows as chorus.** The rationale is natural composition: chorus is a consequence of per-bird windows, not a separate code path.

- **Identifying motif per species.** The plan says the user's ear locks onto species through repeated re-anchoring of that motif.

- **Weather as global server-driven event log input.** The rationale is consistency: weather goes through the same log so the tick treats it like any other input.

- **Notebook salience, threshold, and cooldown.** The rationale is sparse observation rather than feed behavior; entries only emit when salience clears a threshold and cooldowns allow it.

- **Notebook hand-written lowercase templates with light variation.** The rationale is naturalist voice without sameness, and templates are parameterized by observed patterns rather than user behavior.

- **Notebook denylist against user-behavior language.** The rationale is to avoid "you," "every day," "this week," "streak", "visit", and similar engagement framing.

- **Hard cap of about 3 notebook entries/week for very active users.** The plan says this prevents the notebook from becoming a feed.

- **Deterministic tick replay.** The rationale is replay testing, safe bug investigations, and migrations without "what would the bird have done" ambiguity.

### Sync Model

- **Single canonical writer.** The rationale is no merge conflict over personality, mood, perches, calls, or notebooks; row locks or leases enforce one writer per account.

- **Inline-on-HTML initial snapshot.** The rationale is frame-zero data before JS bundle parsing, making the 500 ms first-bird budget reachable.

- **Pull and push after initialization.** The rationale is freshness with fallback: WebSockets push on meaningful state changes, while polling covers disconnected WebSocket cases.

- **No last-write-wins.** The rationale is stated directly: clients submit events, servers apply deltas, and the schema lacks a "set personality_vector" path.

- **Multi-device identical snapshots.** The rationale is consistency: every foregrounded device receives identical canonical snapshots and interpolates locally.

- **Presence dedupe across sessions.** The rationale is avoiding double-counted attention when two devices are foregrounded at once; exact multi-device attention is not required.

- **Visibility and suspend refresh.** The rationale is stale-snapshot recovery: on resume or wake, the client pulls fresh state before continuing.

- **Server-time trust when clocks diverge.** The rationale is correctness under client-clock drift: if divergence exceeds about 5 seconds, the client trusts `server_time`.

### Frontend Rendering Pipeline

- **TypeScript, Vite, esbuild, and bundle-size CI.** The rationale is strict bundle-size control and production build discipline.

- **Canvas aviary with React chrome.** The rationale is performance: render-hot paths stay out of React's reconciler to maintain 60 fps idle on a 5-year-old laptop.

- **Single full-bleed canvas plus thin React overlay.** The rationale is to keep the aviary rendering unified while chrome and modals defer into UI chunks.

- **Background, mid, and foreground scene layers.** The rationale is bounded visual work: background mostly stays static, mid layer holds birds, foreground particles use deterministic seeds without per-leaf state.

- **Skeleton-and-pose bird renderer.** The rationale is compact expressive motion using named poses, micro-deformations, and species-specific assets.

- **SVG poses for v1.** The plan's call says SVG keeps the bundle small and allows procedural color recoloring for plumage tier.

- **Idle micro-motion.** The rationale is mood expression through subtle procedural animation while the client renders mood rather than deciding it.

- **Perch transition paths.** The rationale is visible perch changes without dominating the scene: paths are short, under 800 ms.

- **Reduced-motion perch cross-fade.** The rationale is equivalent continuity without animated paths.

- **Tiny inline first-frame script.** The rationale is first bird visible before the main bundle parses, with continuity when the main renderer hydrates.

- **Quiet field loading state.** The rationale is avoiding spinner/animated loading transitions while preserving the calm aviary surface.

- **Reduced-motion render mode.** The rationale is "the same product, slower": no animated paths, no micro-motion deformation, no leaf drift, no top-bar fade animation, with calls and audio unchanged.

- **Top-bar fade after pointer stillness.** NOT RECOVERABLE FROM PLAN

### Audio Pipeline

- **Per-bird WebAudio subgraph.** The rationale is live procedural motif voicing parameterized by species, mood, and personality.

- **Shared reverb.** The plan says the small reverb gives the aviary "its sense of space."

- **Weather ambient layer.** The rationale is audio reflecting weather state through wind/rain ambience.

- **Chorus mix from overlapping live calls.** The rationale is natural layering without phase-canceling and without a separate chorus path.

- **Listen-in gain re-balance.** The rationale is focus without collapsing the world: other birds never reach zero because "the aviary remains a place, not a bird-soloing UI."

- **Procedural call generation from motif data.** The rationale is deterministic realization from grammar seed, mood, and personality bucket, useful for QA replay.

- **No recorded-audio fallback.** The rationale is bundle and product boundary: if audio fails or is off, playback is bypassed and captions auto-enable.

- **Captions auto-enable on audio failure.** The rationale is accessibility continuity: the aviary continues unchanged and gives a quiet first-time notice.

- **Audio node pooling and deterministic teardown.** The rationale is memory hygiene and avoiding GC churn over long sessions.

### Accessibility Surfaces

- **Server-side screen-reader narration in snapshots.** The rationale is calm, deterministic narration in the same naturalist prose as the notebook.

- **ARIA-live polite idle narration.** The plan chooses polite so calm narration "doesn't barge into the SR queue."

- **Separate assertive region for user-initiated events.** The rationale is priority: offers, settle, and return-greeting get a single short prose bump, then quiet again.

- **Call captions generated from the same seed as audio.** The rationale is consistency between what is heard and what is described.

- **Captions near the calling bird.** The rationale is spatial association with the calling bird's screen position.

- **Caption contrast scrim.** The rationale is WCAG AA contrast against changing aviary backgrounds.

- **Keyboard navigation.** The rationale is full keyboard access to top-bar items, bird focus, listen-in, offer, and settle/undo.

- **Canvas-rendered focus ring.** The rationale is visibility above the scene without leaking through React.

- **Contrast-token verifier.** The rationale is build-time enforcement that chrome text passes WCAG AA.

- **High-contrast mode.** The rationale is to strengthen chrome contrast and bird-perch silhouettes without changing the palette character.

- **Live accessibility settings with no save button.** The plan says changes apply live because save buttons would "over-engineer a low-stakes surface."

### Performance Budgets and Observability

- **Initial JS bundle at or below 2 MB gzipped.** The rationale is first-load performance; budgets are measured on every PR and fail builds.

- **Deferred chrome and modal chunks.** The rationale is keeping the aviary initial chunk small while account, settings, notebook, accessibility, and visit flows load on demand.

- **Motif library as data, not code.** The rationale is bundle accounting and small combined species footprint.

- **Time-to-first-bird at or below 500 ms p75.** The rationale is the first-bird promise: inline snapshot, tiny first-frame renderer, async full bundle, and pre-warmed account snapshot cache.

- **Synthetic perf checks from multiple geographies.** The rationale is continuous detection of p75 first-bird breaches under mobile/4G conditions.

- **60 fps idle on a 5-year-old laptop.** The rationale is calm scene quality with bounded per-frame work for a maximum of 7 birds.

- **Headless scene benchmark with 7 birds.** The rationale is regression detection: p99 frame time over 12 ms fails the build.

- **30-minute memory CI test.** The rationale is enforcing flat-memory sessions and catching AudioBufferSourceNode, list-view, and worker leaks.

- **Aggregate observability.** The rationale is operational health without reconstructing personal state: request counts, latencies, error rates, tick latency, audio errors, and anonymized frame histograms.

- **Metrics forbidden dimensions.** The rationale is privacy invariants at emission and ingestion, blocking `bird_id`, `account_email`, and similar dimensions.

### Privacy / Telemetry Boundary

- **Email in exactly one place.** The rationale is minimizing PII spread: no other table, log, queue, metric, or telemetry event references email.

- **Synthetic UUIDs across service boundaries.** The rationale is identifier minimization and avoiding email as a key.

- **No per-bird or per-account state to analytics, ML, or third-party tools.** The rationale is structural privacy: the plan says no ETL job or ML pipeline exists at v1 and third-party tools must not receive the state.

- **Error reporter PII scrubbing and dimension allowlist.** The rationale is preventing accidental PII leakage in operational tooling.

- **Plain-text privacy policy linked from settings.** The rationale is explicit user-facing disclosure of aggregate categories and exclusion of per-bird state.

### Rollout Plan

- **Foundations first with CI budgets from day one.** The rationale is that schema, auth, edge HTML, empty aviary, and bundle/perf budgets are prerequisites for later work rather than hardening afterthoughts.

- **Simulation before full render and audio.** The rationale is centrality of the tick: drift, mood, event log, snapshot endpoint, and single-bird behavior come before richer species/audio work.

- **Accessibility parallelized before launch.** The rationale is day-one designed accessibility, including screen-reader narration, reduced-motion mode, captions, keyboard nav, focus rings, and high contrast.

- **Visits late in staging.** The rationale is that invites, visitor sessions, projected snapshots, revocation, logs, and expiration depend on stable account/snapshot boundaries.

- **Bird-count ramp with cap at 7 from alpha.** The rationale is early evidence: the team wants to surface chorus issues early and lower the cap if beta shows recognizability collapse.

- **Drift calibration with synthetic populations.** The rationale is meeting the one-week instrument and three-week visible-change targets before launch without per-account telemetry.

- **Post-launch aggregate drift monitoring.** The rationale is tuning the shape of trait distributions without retroactive per-account changes.

- **Calibration version on every bird record.** The rationale is interpreting historical drift correctly after tuning constants.

- **Launch accessibility audit with real assistive-tech users.** The rationale is preventing accessibility being merely synthetic or theoretical.

- **Privacy review with warehouse schema export.** The rationale is confirming no per-account telemetry dimension in the actual data environment.

- **Disaster recovery drill.** The rationale is confirming birds and personality vectors round-trip after restoring Postgres snapshots.

- **Static-analysis CI for negative drift coefficients.** The rationale is enforcing monotonic drift at code level.

### Risks, Mitigations, and Out-of-Band Notes

- **Drift calibration mitigation.** The rationale is protecting the product's central claim from too-fast volatility or too-slow invisibility.

- **Personality vector loss mitigation.** The rationale is avoiding the "worst possible failure" where the user only feels the bird is off.

- **Sync correctness mitigation.** The rationale is preventing future contributors from adding quick client-side personality writes.

- **Audio uncanniness mitigation.** The rationale is making calls feel like birds instead of uncanny UI sounds, using audio design, alpha panels, and tweakable data files.

- **Accessibility regression mitigation.** The rationale is protecting narration timing and focus visibility from later visual/audio polish.

- **Engagement-feature creep mitigation.** The rationale is preserving the product's "affective contract" against reasonable-looking stakeholder requests.

- **Privacy / PII leakage mitigation.** The rationale is preventing email or per-bird state from entering logs, reporters, metrics, or warehouses.

- **Performance creep mitigation.** The rationale is stopping small per-release regressions before the first-bird budget is no longer met.

- **Visit-feature abuse mitigation.** The rationale is reducing forwarded-link, activity-harvesting, revocation-race, and stalking utility risks with short sessions, rate limits, revocation, bucketed duration, and single-bind tokens.

- **Single-writer simulation bottleneck mitigation.** The rationale is horizontal sharding, idle tick dropoff, bounded per-tick work, and launch load tests.

- **Open/deferred decisions.** The rationale is that these are calibrations or details with owners and measurement gates, and "None of these defaults block v1 design."

- **PR description line: "what does this teach the user?"** The rationale is catching features that teach engagement-counter behavior even if technically correct.

- **Writer-of-record review for naturalist voice.** The rationale is that narration and notebook voice is "load-bearing" style, not just code.

- **Timezone regression tests.** The rationale is that day/night bugs are likely, so every discovered timezone bug becomes a test.

- **Manual visual verification of cold-cache first frame.** The rationale is protecting the "first frame is alive" promise from dropped optimizations or spinner regressions.
