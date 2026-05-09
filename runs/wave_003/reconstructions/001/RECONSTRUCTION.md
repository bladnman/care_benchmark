## System-level intent

- The product's affective core is load-bearing. The plan says this directly in the opening: "the product's affective core is load-bearing," and keeps the procedural-call rule, server-only personality writer rule, asymmetric drift rule, no-numeric-personality rule, and "notice, never announce" rule visible across architecture, data model, API, simulation, sync, rendering, audio, accessibility, performance, rollout, and risks.
- The aviary is server-authoritative. The plan repeatedly gives the server ownership of "identity, personality vectors, mood, positions, calls in flight, day/night, weather, presence accounting, drift function, notebook prose, narration prose, visit invitations, the canonical clock." The client renders snapshots and never owns "any field that affects future drift."
- The client is a fast, small renderer, not a simulator. The client owns "rendering, audio synthesis, interpolation between snapshots, idle ambient ornaments," and input capture. It "does not run the drift function as a preview," does not project mood transitions, and does not predict call grammar.
- Personality is protected from numeric exposure and client writes. The plan says personality scalars are normalized internally but "never serialized to clients," with only a derived `plumage_render_token` crossing the boundary. It also says there is no `PUT /birds/:id/personality` and "there will not be one."
- Drift is asymmetric, slow, and silent. The plan calls the asymmetry rule "load-bearing": drift signals are non-negative, there is no "absence drift," no "decay constant," and no negative trait pressure. Notebook entries describe behaviors, not "Pip's social warmth went up."
- The product refuses engagement pressure. The plan excludes streaks, XP, levels, badges, scores, visit calendars, push/email nudges, DAU/MAU surfaces, and per-user retention metrics. It says adding a DAU dashboard would create internal pressure to optimize for it, "which is exactly the rotation the PRD spends three pages refusing."
- The voice is centralized and intentionally split by register. Naturalist prose for notebook, narration, and captions is server-generated and shared so "the voice cannot fork." Matter-of-fact strings are used for sign-in, sync errors, settings, and system clarity.
- The aviary should "notice, never announce." This appears in the ban on textual welcome surfaces, the return-greeting design where "the greeting is birds," the absence of tutorials/tooltips, and the field notebook's behavior-first prose.
- Audio is an affective spine and must be procedural. The plan says "audio is the affective spine" and "the procedural-call rule is non-negotiable." No recorded call audio is shipped or used; motifs are synthesis parameter sets, not recordings.
- Accessibility is a first-class designed surface. The plan says accessibility is "not parity-by-checklist," reduced-motion is "its own designed surface," and screen-reader narration should feel alive "in its register," not label the visual one.
- Performance budgets protect the central conceit. The plan treats <2 MB initial JS, <500 ms time-to-first-bird, 60fps idle, and no memory growth as hard constraints. It says above these thresholds "the central conceit cracks."
- Privacy and identity discipline are architectural. The "synthetic UUID rule is non-negotiable," email lives in exactly one place, analytics are aggregate-only, and the simulation DB and analytics DB have no shared credentials path.

## Per-feature whys

### Scope and non-goals

- Email + magic-link sign-in and per-device session tokens: The plan grounds this in account clarity and privacy: magic-link requests return 202 with "no account-existence oracle," links expire and invalidate on use, and sessions are per-device and revocable.
- Single-user, single-aviary accounts: NOT RECOVERABLE FROM PLAN
- Browser-only delivery: NOT RECOVERABLE FROM PLAN
- Two starter birds at adoption: The plan says the starter pair is chosen to have "different silhouettes and call timbres" so the user "immediately experiences distinguishable calls."
- Cap of seven birds: The plan frames the product as "depth-not-breadth" and says cap-ramping is a calibration affordance while the user experience remains "the calm, two-starter, depth-not-breadth product."
- New-bird offers paced by aviary age: The plan explicitly says offers are "not interaction-count gated" and "not visit-count gated," preserving the refusal of engagement mechanics.
- Species pool with silhouette, plumage palette, and call grammar motif library: The plan ties species choice to distinguishable visual silhouettes and call timbres, and to procedural motif libraries rather than looped recordings.
- Server-authoritative simulation tick: The plan calls the simulation service "the heart of the product" because everything that makes the aviary "feel alive over weeks" is computed there.
- Client interactions as interaction events: Return-greeting, listen-in, offer, settle, and presence are captured as events because the simulation service consumes the append-only log and turns those events into server-authored drift, moods, calls, and notebook signals.
- Multi-device sync: The rationale is "single canonical aviary." Both clients read the same snapshots and stream the same deltas; neither client writes personality.
- Field notebook: The notebook is server-generated, sparse, read-only, and in naturalist prose so it can describe behaviors instead of exposing drift internals or numeric personality.
- Visit: The visit feature is opt-in and read-only because the plan refuses social-network surfaces, co-presence, visitor avatars, comments, and "show-off" rendering; it is a "single quiet visit."
- Visit 30-day invitation expiration: NOT RECOVERABLE FROM PLAN
- Visit notifications off by default: The plan says the aviary "lives where the user visits it"; the per-account visit-notification toggle is the named exception to the no push/email notification rule.
- Accessibility surfaces: The rationale is designed inclusion: screen-reader narration, captions, reduced motion, keyboard navigation, and contrast are first-class surfaces rather than retrofits.
- Performance budgets: The plan says the first bird must appear fast and the renderer must stay smooth because otherwise the "feels alive" premise turns into a loading or performance artifact.
- Account export to JSON by emailed link: NOT RECOVERABLE FROM PLAN
- Soft-delete with 30-day recovery then hard delete: NOT RECOVERABLE FROM PLAN
- Email change with verification: NOT RECOVERABLE FROM PLAN
- Aggregate-only operational telemetry and no ML on per-bird data: The plan uses this to enforce the privacy boundary and to avoid building analytics that can become engagement targets.
- Non-goal protections in the codebase: The plan says non-goals bind "to the codebase, not just the PR description," with CI lint blocking streak/xp/level/score/achievement/badge fields.
- Readonly client personality and no client mutator: This protects the server-only personality writer rule; only the simulation service has a writer.

### Architecture and storage

- Edge/web app embedding the first state snapshot: The rationale is time-to-first-bird; the CDN-edge tier serves the first snapshot with HTML so the first-bird path avoids an extra round trip.
- API gateway: The gateway is stateless and horizontally scaled, and enforces account-row scoping on every read and write, so public HTTPS access does not compromise the canonical account boundary.
- Auth service: It owns the email-to-account UUID mapping and encrypted email at rest because email lives in exactly one place and UUID is used everywhere else.
- Simulation service: It owns canonical aviary state and is "the only writer of personality state, anywhere," preserving server-authored drift.
- Notebook service: It writes on a "slow cadence" and is "sparsity-tuned," supporting sparse naturalist entries instead of noisy activity logs.
- Narration service: It runs at 30-60s idle cadence with priority bumps for user-initiated events so the accessibility register stays alive without bulldozing the experience.
- Visit service: It resolves visitor sessions to read-only snapshots and records host log entries, matching the quiet, read-only visit model.
- Email service: It is transactional only, with "no marketing, no engagement nudges," matching the refusal of notification-driven engagement.
- Event log as partitioned Postgres first, Kafka later if needed: The plan's decision is to start with partitioned Postgres keyed on account UUID and move only "if simulation throughput requires it."
- Physical simulation DB and analytics DB separation: The rationale is that privacy is a "network and credentials boundary, not just a policy"; no service has credentials to both.
- Synthetic UUID identity: The UUID is the only identifier in shard keys, messages, logs, telemetry, object storage, and WebSocket sessions so email does not spread into observability or services.
- Derived plumage render token: Plumage saturation is visible, but only as a derived token; the float never crosses the boundary, preserving no-numeric-personality.
- Mood enum finalized for v1: The plan says the small mood set is finalized so rendering and narration can build against it; widening it is cross-cutting.
- Append-only interaction event log: The log is never edited and is consumed in `(occurred_at, id)` order so ticks can be deterministic and idempotent.
- `cause_summary` on notebook entries: The plan says it is internal, used to rate-limit same-kind entries and debug entry generation, and is never exposed or exported.

### API and user-visible surfaces

- Magic-link request/consume endpoints: They are rate-limited, return 202 regardless of account existence, expire after 15 minutes, and invalidate on first use to avoid account-existence or replay issues.
- Structured account-level errors: The plan says these are rendered in matter-of-fact voice and canonical strings; in error states "a user needs system clarity."
- Snapshot shape: Snapshots carry no personality scalars and no raw mood timers because the renderer only needs mood labels and derived render tokens.
- Interaction event endpoints: They append events rather than writing state so personality cannot be written by client methods.
- Offer cooldown rejection: NOT RECOVERABLE FROM PLAN
- Bird rename as the only mutable bird field: The plan permits only `display_name`; renames are events so the notebook can potentially reference them, and no other bird field is user-writable.
- Visitor entry: Visitors see a sign-in-less, read-only stream; visitor-issued events are rejected, preserving visit as observation rather than co-presence.
- Narration stream and call captions: Captions are pushed inline with call events so they are synchronous with the audible call, while narration uses server-generated prose.
- Server canonical wording and client catalog exception: The server owns every naturalist string to keep voice consistent; matter-of-fact client strings exist because offline or pre-response states need to render before the server can answer.

### Simulation engine

- Active and inactive tick cadence: The plan uses ~1/min for active aviaries and coarser drift-only ticks for inactive ones as "bandwidth," with instant reactivation on next event arrival.
- Deterministic ticks and seeded PRNG: The rationale is debugging and export-reproducibility; the same state, event stream, and wall clock replay identically.
- Idempotent watermark-and-events transaction: If a tick fails partway, replay from the last committed watermark produces the same output.
- Drift calibration targets: After about 7 days instruments should show clear dominant-trait delta, and after about 21 days a typical user can perceive change "without being told."
- Asymmetric drift: Every drift contribution is non-negative; a bird that is ignored "simply does not accumulate drift," avoiding neglect punishment.
- Drift saturation: Traits approach a soft cap exponentially so heavy users do not find birds "maxed out" within a couple of months.
- No client-visible drift event: Notebook entries describe behaviors while the engine computes underlying drift silently, preserving "notice, never announce."
- Mood finite-state machine: Mood is driven by interactions, time of day, ambient events, and personality damping so state persists across tab opens and does not reset on session start.
- Weather as transient influence: Rain dampens vocal frequency transiently and wind shifts mood probabilities; these are not personality drift changes.
- Procedural call grammar: Motifs combine with pitch, duration, pauses, repetition, and mood/personality parameters so two consecutive calls from the same bird are never identical.
- Chorus joining: High vocal-frequency birds can join another call within a short delay, producing "real-time chorus emergent behavior."
- Server schedules calls, client synthesizes them: The server decides when and what motif/shape params; the client synthesizes audio, so the server does not ship audio and the client does not invent call grammar.
- Return-greeting: The greeting is server-driven and can choose no greeter, because forcing a greet on every entry would be the "canned-cue failure" the plan warns against.
- Staggered greeting calls: If several birds would greet, the scheduler offsets calls by 200-900 ms so the aviary reads as noticing "one bird at a time," not as a chorus on cue.
- Ambient weather scheduler: Weather is low-probability, brief, and limited to rain or wind; the mix remains calm with "no sense of storm."
- Starter species selection: The pair is selected for different silhouettes and call timbres so the first experience has distinguishable birds.
- Declining new-bird offers by absence: The plan avoids a separate decline button; decline is the absence of accepting, matching the quiet surface.
- Tick not composing prose: The tick produces signals while notebook and narration services interpret signals into voice, preventing voice and simulation from being tangled.

### Sync model

- No last-write-wins: The plan says personality drift is additive, server-authored deltas, never client-submitted absolute values.
- Concurrent sessions: Events from two devices land in the append-only log and are processed in order; a per-account advisory lock prevents double writes.
- Client timestamp conflict handling: `client_ts` is informational, and the server reuses `ingested_at` as `occurred_at` on conflict for consistent ordering.
- Same-second listen-in conflict: The display may be briefly optimistic, but the plan accepts this because the case is rare and self-correcting at the next snapshot.
- Sync conflict surfaces: Errors use matter-of-fact strings and avoid birds or naturalist prose because system clarity matters in error states.

### Frontend rendering pipeline

- Preact + Signals: The plan says React is too heavy for the bundle budget and vanilla DOM is too verbose for the surface complexity.
- Tiny custom WebGL renderer with Canvas fallback: Three.js is rejected as too large, while Canvas fallback covers environments where WebGL is unavailable.
- Client-side idle ornament path: Leaf and feather drift are generated client-side because per-leaf state would balloon snapshots "for no benefit."
- One horizontal scene with no panning, scrolling, or zooming: NOT RECOVERABLE FROM PLAN
- First frame is not a load state: The plan rejects spinners because a spinner reads as "machine"; the quiet field reads as "the aviary catching up."
- Idle micro-motion: Every bird has subtle preening, scanning, head-tilt, or shuffle so the surface feels alive, with mood shaping frequency and amplitude.
- Soft transitions: Perch changes animate as smooth flight paths, mood changes cross-blend, and beak animation syncs to the call envelope so state changes do not snap.
- Reduced-motion rendering: It is a calmer visual register, not motion off; the same code branch and poses keep it current with feature changes.
- Top bar as only on-aviary chrome: The plan keeps chrome out of the scene itself: no badges, hover-tooltips, or inline labels.
- Top bar fade after cursor stillness: NOT RECOVERABLE FROM PLAN
- Focus indicators in the faded top bar: They must remain visible against the dim state so keyboard focus is never hidden by the chrome fade.
- Tab visibility handling: Hidden tabs halt rendering, close the WebSocket, and suspend audio for near-zero CPU; visible tabs resume with a fresh server snapshot.
- Empty-aviary state before first bird enters: NOT RECOVERABLE FROM PLAN

### Audio pipeline

- Custom WebAudio synthesis: The plan uses custom synthesis nodes and no third-party audio libraries because of bundle budget.
- Fresh per-call audio graph with pooling: Calls are synthesized from motifs and shape params, while graph reuse avoids hot-path allocation.
- Chorus mixing: Overlapping live synthesized calls produce a real chorus without phase-cancellation artifacts from layered loops.
- Listen-in mix: Ramping one bird up and others down is "leaning in to hear," not soloing a track; other birds never go fully silent because "the aviary is still the aviary."
- Ambient and weather audio: Procedural pink-noise textures, rain, and wind are calibrated to remain calm.
- Captions from actual shape params: Caption text reflects per-call variation, preserving the difference between the same motif rendered with different shape params.
- WebAudio fallback: If WebAudio is unavailable, the aviary plays in "graceful silence with captions on by default"; this is a real product surface, not recorded-audio fallback.

### Accessibility surfaces

- Screen-reader narration: It uses naturalist prose at a polite cadence so the screen-reader queue is not "bulldozed" and events remain observations, not state labels.
- Shared notebook/narration voice catalog: The narration service is implementation-paired with the notebook service because "the voice cannot fork between the two surfaces."
- Captions near the calling bird: The spatial cue that "this bird is calling now" is preserved for visual users and can feed screen-reader narration when audio is off.
- Keyboard navigation: Top bar, aviary surface, bird focus, listen-in, Escape, offer, and settle are keyboard reachable so full keyboard navigation is part of the designed surface.
- Settle undo focus behavior: The 5s undo is focusable and traps focus until it dismisses so the undo path remains reachable.
- WCAG AA contrast: All user-copy text passes AA, and the scene itself avoids chrome user copy except top bar and captions.
- Accessibility settings voice: Settings use matter-of-fact wording because this is the named-exception register, not naturalist prose.
- Audio-off mode: Muting auto-enables captions while the aviary remains full-quality otherwise.

### Performance budgets and observability

- Initial bundle code-splitting: Settings, visits, notebook, export/delete, and secondary surfaces are lazy-loaded so the initial bundle is "just the aviary surface."
- Species motif and asset splitting: Adopted species assets are initial; the full species pool loads only during adoption to protect the first load.
- CI bundle-size hard fail: The 2 MB budget is "a build-time constraint, not an aspiration."
- Time-to-first-bird path: Inline snapshot, preloaded critical bundle, and no await on audio or auxiliary chrome make first bird visible within 500 ms.
- Synthetic time-to-first-bird checks: p95 over 500 ms alarms because first-bird latency is a product contract.
- 60fps idle budget: The renderer targets <8 ms per frame and samples idle motion less often above the hard cap to protect smoothness.
- No memory growth over 30 minutes: Audio pools, notebook teardown, and snapshot-history clearing protect long-session aliveness.
- Operational telemetry: Request counts, latencies, tick latency, WebSocket duration histograms, audio errors, render timing, bundle size, synthetic checks, and email delivery measure whether the product is up and working.
- Explicitly unmeasured engagement and drift analytics: The plan refuses per-account duration, per-bird counts, offer frequency, mood distribution, and drift-curve analytics because the warehouse cannot join simulation state.
- Browser support limits: Older browsers get an unsupported-browser surface because compatibility paths would add bundle bloat that "isn't justified."

### Rollout and operability

- Closed alpha: The plan uses about 100 invited users, feature-complete bird engine surfaces, cap held at 3, and visits off to focus tuning on drift calibration.
- Open beta: Registration opens with a soft cap, visit feature on, notebook frequency tuned, and reduced-motion in production for opt-in users.
- v1 launch: Bird cap reaches 7 and new-bird offers use the long schedule from the adoption pacing section.
- Cap ramping via server-side config: The plan says ramping is calibration, not a product feature; the user experience is unchanged at any cap.
- Drift instrumentation during alpha/beta: It is internal, per-deployment, discarded afterward, and not a per-user analytic because it exists only to validate calibration targets.
- No DAU/MAU or retention dashboards: The plan says success is whether users "keep their birds," and engagement dashboards would shift design priorities.
- Adoption-flow wording: The exact lowercase wording is a "seam-handling style" between matter-of-fact system voice and naturalist aviary voice.
- No onboarding tutorial: The product "teaches itself by being present"; discovering listen-in is part of the relationship.
- Snapshot serializer versioning and reload prompt: Older clients reject new snapshot shapes gracefully and prompt reload in matter-of-fact voice.
- Simulation canary/deploy pattern: Ticks pause during a cycle and replay from watermark because ticks are idempotent.
- Sharded simulation workers and advisory locks: Workers shard by account ID and lock per account so ticks stay serialized.
- Backpressure handling: If logs grow faster than ticks consume them, the next tick processes the full window; "no event is dropped."

### Risks and intentionally open seams

- Drift weights as tunables: Too fast becomes Tamagotchi, too slow becomes screensaver; explicit calibration targets and tunable weights protect the "feels alive over weeks" premise.
- Sound designer in the loop: Procedural calls can sound too synthetic, so motif libraries require audio-design collaboration and alpha listen tests.
- Accessibility regression lint and shared ownership: Changes to simulation, mood, perch logic, or call grammar must be paired with prose generator updates so accessible surfaces do not become stripped variants.
- PII scan against email use: CI rejects new code reading account email outside email service or audit-logged display surfaces because email as a convenient identifier is a leakage risk.
- Engagement-feature creep review: Non-goal constants, review triggers, required PRD reading, and team lead ownership keep harmless-looking counters, calendars, or notifications from changing the product.
- Long-session soak tests: The risk is slow memory leak; the mitigation is 30-minute CI soak with hard memory-growth threshold and audited pool sizes.
- Embedded snapshot as hard requirement: The first-bird path is not an optimization; a missing snapshot would turn the quiet field into the spinner-failure mode without the spinner.
- Shared prose module and rejection criteria: Notebook, narration, captions, matter-of-fact strings, and seam strings are reviewed centrally to avoid "naturalist-ish" fragmentation.
- Exact normalized personality ranges: The plan intentionally leaves this as implementation detail in the simulation service.
- Exact filter gains and mood probabilities: The plan fixes calibration targets and transition graph but leaves gains and probabilities to alpha tuning.
- Concrete species pool, contrast ratios, typography, and palette tokens: The plan leaves these to visual/audio designers and the design system while preserving the commitments above.
