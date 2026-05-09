## System-level intent

- Server-canonical relationship, not client ownership. This shows up in Architecture, the client/server split, the API event model, the simulation engine, sync, frontend reconciliation, and the plan summary. The plan's own line is "anything that affects the canonical relationship the user has with their birds" runs server-side, while the client handles "presentation." It is repeated as "server-only personality writes," "clients never write personality directly," "server is the only canonical writer," and "no client-side sync engine."

- Canonical state vs presentation continuity. The render pipeline and frontend sections distinguish "presentational continuity" from speculation: the client may interpolate frames, run ambient ornaments, and map local day/night palette, but it "does not extrapolate canonical state." This principle also explains why snapshot delivery is push-on-change with pull fallback, and why the event endpoint returns "accepted" rather than "applied."

- Presence creates additive, monotonic drift; absence never punishes. The drift system is described as "presence-driven low-pass drift toward expressive (monotonic, never negative)." The simulation section names the "load-bearing asymmetry test": no presence for 14 days produces "zero drift in either direction." This also underlies the refusal of Tamagotchi mechanics, death, hunger, decaying happiness, distress states, and any negative drift on neglect.

- Refuse standard engagement mechanics at the architecture layer. The plan repeatedly refuses "streaks, achievements, badges, levels, scores," welcome-back surfaces, push/email/in-app summons, and "harmless" gamification. This is not only product copy: the schema has no `streak_days`, `xp`, `level`, `engagement_score`, or leaderboard table; lint rules forbid strings like "streak," "achievement," "unlocked," and "welcome back." The plan says "refusing these features at the data layer is more durable than refusing them at the UI layer."

- A quiet naturalist product voice, with matter-of-fact system exceptions. The aviary, field notebook, screen-reader narration, captions, and offers use "naturalist prose," "lowercase," "present-tense," "specific" language. In contrast, auth, errors, settings, unsupported-browser, visit revocation, and email copy use "matter-of-fact tone" and "system-clarity over naturalist warmth." The plan explicitly blocks backend developers from slipping naturalist phrasing into error responses.

- Accessibility is a designed surface, not a fallback or checklist. This shows up in the scope item "Accessibility as designed surface," the reduced-motion frontend section, the accessibility section's "first-class, not a checklist" stance, and the test plan. Reduced motion is "a different rendering of the same aviary" and should feel "calmer; not one that feels broken."

- Procedural, varied, deterministic audio instead of recorded or canned sound. The call grammar and audio sections keep audio procedural so calls vary, fit the bundle budget, and can produce a real chorus. The plan names the failure mode as "feels canned" and says silence plus captions is better than recorded fallback if WebAudio is unavailable.

- Privacy is an architectural boundary, not a policy promise. The plan says telemetry sinks "never receive per-account or per-bird state," the simulation database is separated from the analytics warehouse, analytics service accounts have no read access to simulation data, and metric labels containing `bird_id` or `account_uuid` are dropped. The privacy section repeats that per-bird interaction events are never aggregated, trained on, shared, or used for recommendation.

- Small, fast, browser-first implementation. The plan excludes native apps, targets a browser-rendered aviary, enforces a "<2MB initial JS bundle," and treats "time-to-first-bird <500ms" as an "affective-perf threshold." Architecture choices such as Solid.js, inlined snapshots, custom animation runtime, code splitting, and procedural audio are all justified by this budget.

- Calibration before certainty. Where numbers are not specified, the plan picks values as "calibration targets": 60-second tick cadence, 4-minute presence window, drift rates, 60-day third-bird eligibility, and 3 events per week weather. Rollout and risks repeatedly say values are tuned in beta, then locked at public launch.

- Future drift is forward-only. Migrations, drift calibration, rollback, and privacy audit sections all avoid retroactive reshaping. Personality-vector migrations must preserve existing values; post-launch drift-function changes are "forward-only"; rollback uses idempotent ticks and Postgres as source of truth.

## Per-feature whys

### 1. Scope

- Single horizontal browser-rendered aviary scene per account, sized to fit viewports: NOT RECOVERABLE FROM PLAN

- Two starter birds and hard cap of seven: the plan frames the product as "a small set of birds." Rollout starts with 2, raises to 3, and reaches 7 only after testing "chorus recognizability with real users," so the cap is treated as an affective/audio calibration issue rather than a scale-maximization issue.

- Six bird species with distinct silhouette, plumage palette, and call-grammar motif library: the plan uses species to make birds visually and sonically distinct. It also says exactly six were picked from "about six species," and that nightjar is required to satisfy "one species remains active into the night."

- Hidden five-dimensional personality vector: the rationale is that personality drives the relationship and drift while avoiding user-visible numbers. The plan repeats "user never sees personality vector values" and forbids numeric exposure in stats panels, debug views, toggles, or power-user surfaces.

- Five-state mood enum: the plan uses mood to make state persist and react to events, time of day, weather, and bird-to-bird interaction. The mood section says stickiness makes mood feel like a "state," not a per-tick guess.

- Presence-driven low-pass drift toward expressive: this is the product's central relationship model. Presence moves traits upward over weeks; absence creates "zero drift in either direction." The visible plumage ramp is the "headline drift signal."

- Server-side simulation tick: the tick is the sole writer of bird/personality state and preserves sync correctness. Rust is chosen because the tick is a "hot path," a memory-bounded long-lived workload, and GC pauses would show up as p99 tick-latency alarms.

- Append-only client-to-server interaction event log: the event log is the "source of truth for what the user did." It lets the simulation consume events in order, mark watermarks transactionally, replay for reconciliation, and avoid direct client mutation of personality.

- Multi-device sync as server-canonical model: the plan says sync "comes for free" from the server being the only canonical writer. There is no client-side merge layer, no CRDT, no eventual reconciliation, and no last-write-wins personality conflict.

- Email magic link sign-in: the plan gives the security and privacy shape rather than a password rationale. The endpoint always returns 202 so it does not leak whether an email is registered; tokens are single-use; sessions are revocable per device.

- Magic-link 15-minute expiry: NOT RECOVERABLE FROM PLAN

- Synthetic UUIDs and email stored once, encrypted at rest: this prevents PII spread. The privacy risk section warns against "email-as-identifier"; the accounts table is the only table with encrypted email, and a linter flags new email-like columns.

- Account export: the plan treats export as a data-portability surface. It explicitly says personality vectors belong in export because "they are the user's data," even though they are not product-surface numbers.

- Soft delete with 30-day grace and hard delete: the plan ties this to account deletion and cascading removal of birds, personality vectors, bird state, interaction events, notebook entries, sessions, and visits. The specific 30-day grace rationale is not otherwise articulated.

- Three perch zones: NOT RECOVERABLE FROM PLAN

- Local-time day/night cycle: the aviary uses "the user's local time, not server time." Local time drives palette and mood time-of-day priors; the cached timezone updates when the user signs in from a different timezone.

- Ambient weather: weather is server-canonical so the same aviary state appears across the user's devices and to visitors. Weather also modifies mood and call behavior.

- Ambient leaf/feather drift: the plan treats this as "pure rendering ornaments" with no state to sync.

- No in-scene chrome and fading top bar: the aviary scene is meant to be birds and place, not UI. The plan forbids UI chrome inside `<AviaryScene>` and uses a low-contrast top bar that fades on stillness while returning for pointer or keyboard activity.

- Return-greeting: the bird greeting is the entire welcome. The plan explicitly excludes "welcome back" toasts, banners, modals, or textual welcomes, so return-greeting carries the return moment without announcement-style product copy.

- Listen-in interaction: listen-in is focused attention on a bird. It changes the mix toward the focused bird, contributes small drift to that bird's social warmth and vocal frequency, and avoids a "selected" badge or toast.

- Three offer types: seed, song fragment, still pool: NOT RECOVERABLE FROM PLAN

- Per-bird offer cooldown: NOT RECOVERABLE FROM PLAN

- Settle gesture with 5-second undo: settle is treated as a clean session-end marker. It contributes no drift and is idempotent in multi-device sync.

- Field notebook: notebook entries are auto-generated "naturalist-prose observations" with sparsity guarantees. Server generation is justified because sparsity is a global property and entry quality depends on full state; read-only structure avoids turning it into user-editable notes or an event log.

- Visit invitation feature: the visit feature is the only social affordance the plan allows. It is email-based, one-time, read-only, revocable, default off, expiring, and visitor sessions do not write events, because visitors should not affect drift, mood, or simulation state.

- Optional opt-in visit notification: NOT RECOVERABLE FROM PLAN

- Procedural call synthesis and chorus mixing: procedural audio supports per-call variation, species motifs, deterministic cross-device calls, and real-time chorus instead of stacked loops. Recorded audio is refused because the bundle would not fit the needed variation and would risk "canned audio."

- Silent fallback with auto-captions if WebAudio is unavailable: silence is preferred over recorded fallback. Captions auto-enable so the call grammar remains perceptible when audio is unavailable or blocked by browser policy.

- Screen-reader narration, reduced motion, call captions, keyboard navigation, WCAG AA: these implement accessibility as a designed surface. The plan's why is not compliance alone: AT users should receive the same aviary, drift, calls, notebook, and product voice through another rendering.

- Performance budgets: the budgets protect the affective experience from slow regressions. "Time to first bird" is named as an affective-perf threshold, bundle size is CI-asserted, and memory/FPS limits protect long quiet sessions.

- Synthetic perf monitoring, aggregate-only RUM, tick-latency alarm: synthetic monitoring catches production regressions; aggregate-only RUM preserves the privacy boundary; p99 tick latency protects the server-canonical simulation loop.

- Browser support for last two majors and unsupported-browser surface: the matter-of-fact unsupported-browser tone is specified, but the exact "last two majors" browser-support cutoff is NOT RECOVERABLE FROM PLAN

- Per-bird interaction privacy rule: per-bird events are used only for simulation. The plan forbids aggregation, training, recommendation, sharing, and cross-account "popular interaction patterns" because that would violate the simulation/analytics boundary.

### 2. Architecture

- Five services with single responsibilities: the plan says "smaller is the goal" and resists merging services because simulation/sync correctness relies on clean boundaries.

- Edge / Web service: it serves static assets and inlines the initial state snapshot so the first bird renders without a second round trip, supporting the <500ms time-to-first-bird target.

- Auth service: it owns accounts, magic links, sessions, email changes, and deletion so authentication state stays isolated from simulation state.

- Simulation service in Rust: Rust is chosen for a "hot path," memory-bounded long-lived process handling hundreds of thousands of accounts and tick work without GC-pause risk.

- API Gateway: it concentrates public client API behavior, session validation, rate limiting, event-log writes, snapshot reads, visit lifecycle, export triggers, and settings so the browser surface stays narrow.

- Notebook Worker in Python: Python is chosen because entry generation is "text-template-heavy" and benefits from flexible templating, while performance is non-critical.

- Postgres for accounts, birds, state, events, notebook, visits, and sessions: the plan uses Postgres as the source of truth for durable state and rollback. The specific replica/read-replica rationale is NOT RECOVERABLE FROM PLAN

- Redis for magic-link tokens, session cache, and presence-window state: Redis is used for short-lived tokens, fast verification, and fast per-account presence-window reads during ticks.

- Object storage for static assets and account exports: object storage serves as CDN origin and holds export JSON behind signed, time-limited URLs.

- Telemetry sinks excluding per-account and per-bird state: this implements the architectural privacy boundary.

- Client/server split by canonical state vs presentation: personality, mood, call timing, weather, notebook, and greeting selection run server-side because they affect the canonical relationship; WebAudio rendering, micro-motion interpolation, local palette, and ambient ornaments run client-side because they are presentational or device-local.

- Push-on-change snapshot delivery with keepalive pull fallback: SSE makes state changes visible quickly while pull fallback covers unreliable SSE environments.

### 3. Data model

- Synthetic UUID identifiers and account-only email storage: this localizes PII to `accounts` and prevents email from becoming a shard/log/service identifier.

- Personality seed clipping at adoption: the ceiling leaves "headroom for drift-up over weeks"; the floor keeps a new bird from being "entirely flat at adoption."

- Plumage saturation starting low and drifting visibly: plumage is the most visibly drift-driven trait and the "headline drift signal," so it is calibrated to be legible over weeks.

- Mood x perch x action legality matrix: the simulation enforces plausible combinations to prevent "nonsensical states" like a wary bird preening unbothered, which would read as broken.

- No engagement columns or public-discovery flags: absence of `streak_days`, `total_visits`, `xp`, `level`, `engagement_score`, `share_publicly`, and `is_featured` makes gamification, engagement scoring, and discovery harder to accidentally construct.

- No mood-history analytics table and no average-drift view: the plan refuses replayable mood analytics and cross-account drift aggregation at the schema level to support the telemetry/privacy boundary.

### 4. API surface

- JSON-over-HTTPS plus SSE, not WebSocket: the plan considered WebSocket and rejected it because bidirectional streaming is unnecessary; clients POST events and reads are server-to-client, so SSE keeps the surface smaller.

- Magic-link issuance always returning 202: this avoids leaking whether an email is registered.

- First sign-in account creation and adoption: first sign-in creates the account and starter birds. The plan frames the birds as having arrived rather than showing a catalog.

- Adoption naming UI with default names and bird fly-in: the plan specifies the experience, but the rationale for the naming UI and fly-in sequence is NOT RECOVERABLE FROM PLAN

- Inlined initial snapshot: this exists to hit time-to-first-bird without a second network round trip.

- SSE event types: full snapshot, delta, greeting, notebook, keepalive: full snapshots support connect/reconnect, deltas push tick changes, greeting carries return choreography, notebook announces new entries, and keepalive prevents idle middlebox closure.

- Snapshot shape hiding personality vectors: devtools should not expose boldness, curiosity, social warmth, or vocal-frequency numbers. Only the visual plumage scalar is exposed because the render ramp needs it.

- Opaque plumage render hint: the scalar is a "hairline judgment call" that serves rendering while avoiding a named `plumage_saturation_trait` product surface.

- Batched interaction events returning "accepted" only: this enforces server-authoritative state. The client must wait for the next snapshot to learn what the simulation made of its events.

- Notebook API read-only with no write/edit/delete: the notebook is generated by the system as sparse naturalist observation, not maintained by the user.

- Visit invitations and read-only visitor sessions: visitors can see the ambient aviary, but read-only tokens reject event writes and the server does not record visitor presence, preserving the host's simulation state.

- Account export worker and signed URL: export generation is async and delivered through a signed URL because it may include a full JSON snapshot, including user's data such as personality vectors.

- Settings for audio, captions, reduced motion, visit notifications, sessions, export, and delete: settings are the matter-of-fact control surface for accessibility, account, and privacy operations.

- Voice-of-API matter-of-fact responses: system-side messages should not use naturalist warmth; a string-style guide and lint rule protect that boundary.

### 5. Simulation engine design

- 60-second tick cadence: the plan resolves "~once per minute" to 60 seconds with a 30-90 second calibration range. The why is operational: bounded tick work and calibration against drift targets.

- Tick idempotency: re-running the same tick should produce the same state, using `last_tick_at` and consumed-event watermarks, so rollback and worker replacement do not corrupt state.

- Tick computation order: events are consumed before presence rollup, drift, mood, calls, bird-to-bird interaction, weather, perch decisions, state writes, and SSE delta. This order makes user activity feed drift/mood before rendering new snapshots.

- Drift function: it low-pass-filters presence and interaction into non-negative trait deltas, with saturation so traits do not bunch at 1.0.

- Drift calibration targets: week 1 should be measurable in instruments; week 3 should be visible through plumage and idle-motion repertoire; no presence for 14 days should create zero drift.

- Rejecting client-submitted personality state: the input schema rejects trait-like fields and only the simulation worker writes `personality_vectors`, protecting the core relationship model.

- Bounded drift history: a compact 12-week summary lets the notebook write lines like plumage changing "over the last fortnight" without storing raw event history forever.

- Mood transition priors and modifiers: time of day, recent events, weather, and personality shape mood so birds react to context while remaining server-persistent.

- Mood stickiness: a 90-second minimum prevents mood from feeling like a per-tick random guess.

- Species assignment at adoption with no visible catalog: the plan says "the two birds arrived; that's the framing," so adoption avoids catalog-shopping language.

- Presence accounting requiring visible tab, focus, and recent pointer/key activity: this prevents background tabs and unattended laptops from creating drift, while choosing a 4-minute window because "watching birds without moving is the real product."

- PresenceTracker tests and red-team trace: these exist because presence is the dominant drift input and unattended background time must assert zero drift.

- Server-side call grammar decisions with client-side synthesis: the server controls when and which motifs happen; the client uses the seed to reproduce the same audio across devices and sessions.

- Bird-to-bird interaction: chorus-join, wary-spread, and curious-attention make the aviary feel like "a small social system rather than independent NPCs."

- Weather simulation: weather is calibrated to "a few times per week," lasts long enough to affect mood/calls, and is shared by visitors because it is server-canonical.

- Third-bird offer cadence by aviary age: the plan explicitly refuses visit-count or score-based acceleration. The cadence is "wall-clock" so no one is rewarded for visiting every day.

- Simulation forbidden behaviors: no client trait writes, no negative drift, no personality reset, no bird replacement, no streak signals, and no cross-account aggregation. These preserve the load-bearing product rules.

### 6. Sync model

- No client-side sync engine: both devices connect to the same stream, write the same event log, and receive the same snapshots because the server is canonical.

- No last-write-wins for personality: personality is additive from event-log deltas, so clients never submit conflicting personality states.

- Event-log conflict resolution: simultaneous device events are ordered by `occurred_at`; later listen-in is "in effect" while earlier listen-in still contributes observed-duration drift.

- Simultaneous settle idempotency: settle is not stored as a conflicting boolean, so simultaneous settle events do not create a data conflict.

- Rejecting CRDT/eventual/client reconciliation models: the plan says there is no client-side state that needs to merge and snapshots are consistent at every tick.

- Multi-device simultaneous listen-in without device locking: attention from two devices is still attention, and locking would add a "switch active device" UI that the plan calls announcement-style.

- Visitor stream as fan-out: visitors receive host snapshot deltas but cannot write events, so visitor activity does not enter simulation.

- Visitor token validation and revocation surface: invite tokens are checked for revocation and expiry; invalidation produces a matter-of-fact `visit:revoked` event.

### 7. Frontend rendering pipeline

- Solid.js: chosen over React for smaller runtime size and fine-grained reactivity, matching a "signal in, signal out" state flow without virtual DOM diff work per tick.

- Hybrid SVG and Canvas rendering: SVG gives sharp scene/background/perches/accessibility; Canvas handles per-frame procedural bird micro-motion without thrashing SVG.

- Local UI state in Solid signals: local state is limited to presentation controls such as listen-in state and top-bar visibility, while canonical state comes from SSE.

- Custom animation runtime: chosen instead of GreenSock or Framer for bundle budget, deterministic seeded jitter, and mood-keyed easing.

- Vite/Rollup code splitting: account settings, accessibility settings, and visits load separately to stay within the initial bundle budget.

- Bootstrap drawing first bird before hydration: the first bird should not wait for JS hydration; a small script can draw a static-pose bird from the inlined snapshot to hit the affective performance target.

- Snapshot reconciliation with no extrapolation: deltas update render state, but stale connections keep last-known animation rather than inventing state; reconnect cross-fades to canonical full snapshot.

- Idle micro-motion generators: computed motion avoids looped frame animation and allows mood/action/personality-keyed variation.

- Day/night palette and weather overlays: local time drives sky variables; weather and particles are bounded to preserve 60fps headroom.

- Reduced-motion renderer: cross-fades, still poses, removed leaf drift, and slower palette changes make a "different rendering" that is intentional and calmer rather than broken.

- Top-bar icons and fade behavior: low-contrast chrome recedes from the scene, while hover/focus states restore accessibility and keyboard focus keeps the bar visible.

- Top-bar slot placement, with notebook/offer/settle left and settings right: NOT RECOVERABLE FROM PLAN

- No UI chrome inside `<AviaryScene>`: a linter enforces this so future tooltips, modals, and toasts cannot creep into the aviary scene.

### 8. Audio pipeline

- WebAudio procedural synthesis per bird: motif parameters produce varied calls without recorded assets and allow every bird/species to have distinct call grammar.

- Motif libraries per species: these support species identity and per-call variation while staying within the bundle.

- Same-call determinism: the server's `motif_seed` lets the same snapshot sound the same on multiple devices.

- Chorus mixing: per-bird gain, master bus, and compressor allow simultaneous calls to form an actual chorus without clipping.

- Listen-in mix ramp: gradual 600ms gain changes avoid a "switching channels" feel.

- No recorded audio fallback: the plan says recorded audio would not fit the variation requirement and "canned audio" is worse than silence.

- Audio context activation on user gesture: this exists because browser autoplay policies require it; before activation, captions cover scheduled calls.

- Captioning from call grammar: captions match what the audio actually played because they are generated from the same motif and intensity parameters.

- Listen-in controls and subtle visual signal: click/tap/keyboard controls make the interaction accessible; subtle outline avoids badges and toasts.

- Audio bundle budget: motif library, synthesis, and captions must fit within the 2MB envelope.

### 9. Accessibility surfaces

- Screen-reader narration: a live region reads the same snapshot as the visual renderer and emits sparse naturalist updates, so the AT experience is not a state-list or announcement surface.

- Assertive return-greeting narration only: return-greeting is prioritized so it lands promptly; ongoing observations remain polite.

- Reduced-motion mode: the rationale is parity through a calmer rendering, not a "lite" version; calls, drift, and notebook still happen.

- Captions: captions appear automatically when WebAudio is unavailable or audio is off and can be forced on, making procedural calls accessible.

- Keyboard navigation: full keyboard access lets users operate top bar, birds, listen-in, offer, settle, notebook, and escape flows without pointer dependence.

- Contrast requirements: all user copy must meet WCAG AA against any aviary state, especially top bar labels, captions, settings, errors, and account UI.

- Accessibility settings surface: audio, captions, and reduced-motion preferences are explicit matter-of-fact controls.

- Accessibility test plan: axe-core catches mechanical regressions, while real VoiceOver/NVDA/TalkBack testing is required because narration quality cannot be validated by automation alone.

### 10. Performance budgets and observability

- Initial bundle, first bird, FPS, memory, tick latency, and snapshot-delivery budgets: these protect core feel, quiet long sessions, and server-canonical freshness.

- CI bundle assertion and synthetic Lighthouse/perf fleet: these prevent slow regressions such as bundle creep and time-to-first-bird drift.

- Frame-time histograms and RUM: aggregate client telemetry lets the team catch dropped frames and slow sessions without per-account tracking.

- Observability server metrics: request latency, tick duration, worker queue depth, SSE connection health, and DB latency are operational signals for the distributed architecture.

- Deliberately not measuring per-account session duration, per-bird mood histograms, per-account interaction counts, or engagement metrics: these would violate the product intent and privacy boundary.

- Telemetry denylist for `bird_id` and `account_uuid`: the metric library and linter make the analytics boundary hard rather than policy-only.

- Synthetic monitoring fleet across four geographies: production checks assert TTFB, time-to-first-bird, SSE, audio activation, and frame rate with structured diagnostics.

- Error budgets: render errors, SSE disconnects, tick latency, and audio activation are bounded so regressions page or fall back to captions.

### 11. Privacy architecture

- Encrypted email plus HMAC lookup: email can be looked up without becoming reversible identifiers elsewhere; the HMAC key stays inside auth lookup.

- Per-bird interaction events retained for account lifetime: they are needed by simulation and deleted on hard account delete; the plan does not let them become analytics material.

- Notebook entries retained for account lifetime: they are user-visible generated observations and deleted on hard delete.

- Aggregate operational telemetry with 90-day retention: counts, latencies, errors, and anonymized histograms support operations without per-account dimensions.

- Logs with synthetic account UUID only: logs avoid emails, long-lived IP details, and bird-state details; lint flags bird-state logging.

- Not aggregating, training on, sharing, or computing patterns from per-bird data: this preserves the privacy architecture and avoids recommendation/engagement pipelines.

- Account hard delete cascade: removing the account drops birds, personality vectors, state, events, notebook entries, sessions, and visits, making deletion complete for simulation data.

- Quarterly privacy audit: this confirms the simulation/analytics separation has not drifted, linter rules still exist, and new paths have not violated the boundary.

### 12. Internationalization and localization

- English-only v1: the plan says the PRD does not call out i18n, so localization is not scoped for v1.

- Local-time anchoring from day one: day/night palette and time-of-day mood priors must use the user's local time, not server time.

- Timezone cache updates on new devices or travel: the plan avoids clever travel detection and simply uses the most recent observed timezone.

- Tagging future localization surfaces: notebook, narration, offer prompts, and call captions are tagged so a future i18n pass has clean boundaries.

### 13. Rollout

- Phase 0 internal prototype: simulation is built end-to-end with one species and two birds first so the team can verify drift over a simulated week before frontend work.

- Phase 1 closed alpha: frontend, two species, core interactions, notebook, and SSE are used with about 20 internal users to validate "does the aviary feel alive?" rather than scale.

- Phase 2 invite-only beta: all six species, sync, accessibility, visits, and real users validate drift calibration over 4-6 weeks, sync correctness across devices, and AT surfaces.

- Phase 3 public launch: sign-up opens only after calibration values are frozen and bundle budget is held.

- Birds-per-aviary ramp: the cap is feature-flagged and raised only after chorus recognizability is tested with real users.

- Drift instrumentation: simulated users and population-level trait distributions verify presence-time produces calibrated shifts without exposing per-account values.

- Sync instrumentation: a multi-device harness asserts clients see identical snapshots within the SSE delivery window.

- Audio instrumentation: a daily synthetic chorus spectrum check catches regressions that subtly alter procedural audio.

- Accessibility instrumentation: axe-core plus manual AT walkthroughs catch both mechanical and quality regressions.

- Calibration windows: beta tuning adjusts rates so the plan's "measurable in instruments at week 1, visible to user at week 3" target is reached, then locked.

- Server-controlled feature flags: visits, third-bird offers, bird cap, and screen-reader narration can ramp or be disabled without client overrides.

- Forward-only migrations: production avoids down-migrations, and any future personality trait change must preserve existing values and document the change so drift history is not invalidated.

- Rollback via worker-version swap and idempotent ticks: the simulation worker is the fragile component, so rollback swaps workers while Postgres remains source of truth.

- Day-1 launch checklist: performance, accessibility, drift calibration, privacy audit, email delivery, export, hard delete, visit revocation, and WebAudio fallback are all launch gates.

### 14. Risks

- Drift speed setting refusal: the plan will not ship a "drift speed" setting because drift speed is a designed property, and making it tunable would turn it into a number the user manages.

- Sync correctness mitigation: consumed-event watermarks, same-transaction personality updates, replay reconciliation, and property-based tests protect against silent skipped or double-counted interactions.

- Audio uncanniness mitigation: sound-designer involvement, alpha/beta listening research, real-bird reference comparison, and phase-correlation analysis address the "feels canned" failure mode.

- Accessibility regression mitigation: CI, manual AT testing, reduced-motion visual diffing, and AT-user beta gating protect surfaces most engineers do not routinely use.

- Harmless gamification mitigation: PR templates, absent schema columns, migration review, and string linting make small engagement features visible before they change the product.

- Notebook voice-drift mitigation: constrained templates, copywriter review, generated-entry review, forbidden-pattern checks, and hard sparsity caps preserve naturalist prose.

- Performance-regression mitigation: CI bundle checks, synthetic time-to-first-bird monitoring, and monthly frame-time review catch slow regressions.

- PII leakage mitigation: synthetic UUID review, email-column linter, and quarterly log grep prevent email from spreading.

- Visitor privacy mitigation: the visit log shows only visitor email and timing; visitor tokens are scoped per invitation; visitor IPs are scrubbed after 30 days.

- Simulation failure mitigation: account partition fan-out, backfill on restart, last-known snapshots, and a matter-of-fact stale-state banner preserve delayed-but-correct behavior.

### 15. Open questions

- Plumage-saturation visual treatment: the plan exposes an opaque normalized scalar but leaves exact visual mapping for the visual designer.

- Motif libraries: sound designer delivers them and voice-of-product approves.

- Notebook-entry template library: copywriter delivers and voice-of-product approves.

- Reduced-motion pose set: visual designer delivers it.

- Default name suggestions per species: copywriter delivers them; the rationale for the default-name set itself is NOT RECOVERABLE FROM PLAN

- Magic-link email copy: copywriter approves matter-of-fact tone.

- Unsupported-browser surface: copywriter delivers a matter-of-fact surface.
