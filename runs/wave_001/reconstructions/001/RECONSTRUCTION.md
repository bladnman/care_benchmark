## System-level intent

- **Load-bearing absences, not deferred TODOs.** The plan treats non-goals as architectural constraints: "any gamification primitive," "Tamagotchi mechanics," and social-network surfaces beyond "the single visit affordance" are "load-bearing absences." This shows up in Scope and again in the data model, where no "streak/count/rank" fields exist on purpose so a later change must be "deliberate" and "reviewable."

- **Canonical server state over client guesses.** The plan repeats that "the client never computes canonical state," "there is no client-side simulation of any kind," and multi-device sync is "not a protocol at all" but a property of devices reading the same snapshot endpoint. The intent is to rule out "divergent-simulation failure," reconciliation, client-side merge logic, and stale-device clobbering.

- **Personality secrecy and privacy are structural, not cosmetic.** The plan treats "never exposed numerically" as a "wire-format constraint," using "presentation envelopes only" and no raw personality scalars in snapshots. The same philosophy appears in observability: aggregate-only RUM is "physically separated from the simulation database," with a deliberate negative space around engagement dashboards and per-account dimensions.

- **Drift is slow, additive, identity-preserving, and never punitive.** Bird identity is "permanent and never reused"; drift is "additive-only," has no negative signal, and is calibrated to be "measurable in ~1 week, visible in ~3 weeks." The plan ties this to the "perceived validity" of drift and to making neglect structurally unable to move traits down.

- **Accessibility is a first-class product surface with one shared voice.** Scope says narration, reduced motion, and captions ship "as first-class surfaces" in v1. Accessibility and notebook share an "ObservationGenerator" so a screen-reader user hears "the same product voice" in live narration and the notebook, rather than "two products glued together."

- **Naturalistic procedural liveness over loops or canned media.** The plan favors procedural call grammar, runtime variation, part-based bird rigs, noise-seeded idle motion, randomized greeting offsets, and "never identical twice" behavior. It also says "no audio files for calls, anywhere" and makes the first frame place birds "already in motion."

- **Performance is part of the feeling of the product.** The plan calls time-to-first-bird the "affective-perf bridge" and monitors it as a product metric. Bundle size, first paint, frame time, heap growth, and audio node lifecycle are treated as engineering requirements because they preserve the aviary's affective surface.

- **Sparse, quiet surfaces are preferred to feeds and dashboards.** The notebook is "sparse, naturalist, auto-generated, read-only," with under-suppression risking a "feed." Observability refuses "average drift across accounts" and per-species engagement breakdowns because they would require correlating behavior, "even in aggregate, even with good intentions."

- **Deliberate v1 simplicity with named evolution points.** The plan repeatedly chooses v1-appropriate architecture: focused services, Postgres append-only event table at v1 scale, no WebSockets, a single batch tick worker, and future migration or sharding only at known triggers. The reason given is to avoid infra that "would slow the team down on the bird engine" or be "wasted complexity at launch volume."

## Per-feature whys

- **Single-user accounts:** NOT RECOVERABLE FROM PLAN

- **Magic-link sign-in:** NOT RECOVERABLE FROM PLAN

- **One canonical aviary per account:** The plan uses this to make multi-device sync simple and canonical: devices share server state, clients never merge, and the aviary freezes instead of guessing forward when the network is down.

- **Two starter birds at adoption:** NOT RECOVERABLE FROM PLAN

- **Age-gated third-bird-and-beyond offers:** The rollout section says these ship for every account from day one because they are "a product mechanic, not a rollout ramp" and not something to "turn up gradually."

- **Seven-bird cap:** NOT RECOVERABLE FROM PLAN

- **Six-species pool:** NOT RECOVERABLE FROM PLAN

- **Field notebook:** The rationale is to keep observations "sparse" and "naturalist," not feed-like. It shares the ObservationGenerator with live narration so the voice of the notebook and the live aviary "never drift apart."

- **Presence accounting:** The plan uses presence as the drift-bearing attention signal, enforcing visible + focused + recent activity locally before heartbeats are sent. Heartbeats keep volume bounded while preserving the exact three-condition definition.

- **Visit-invitation feature:** The plan frames this as the only social affordance: "per-invite opt-in" and "read-only ambient visitor sessions." Visitor tokens have no POST route at all, preserving the core aviary while avoiding profiles, follows, discovery, comments, or other social-network surfaces.

- **Screen-reader narration:** The plan uses one ARIA live region with ambient and event lanes to avoid "separate competing regions." Narration is phrased as observations, not "state-transition announcements," and uses the shared ObservationGenerator for consistent product voice.

- **Reduced-motion mode:** The plan says reduced motion must ship with v1 because a late reduced-motion mode would "quietly" tell reduced-motion users "the product wasn't for them." It is threaded through the same animator and ambient systems rather than built as a separate stripped app.

- **Call captioning:** Captions are generated from the same call-grammar invocation parameters used by synthesis, so a caption "can never describe a call that wasn't actually the one played."

- **Account export:** NOT RECOVERABLE FROM PLAN

- **Soft-then-hard account deletion:** NOT RECOVERABLE FROM PLAN

- **Internal split between `lighting_state: active|settled` and bird mood `roosting`:** The plan splits the overloaded "settle" term because the user gesture and the night-driven bird behavior have different inputs. Collapsing them would make tick logic ambiguous about which input caused which transition.

- **Aviary-scoped offer targeting with independent per-bird reactions:** The plan resolves the offer as aviary-scoped because the affordance is in the top bar, not on a bird. Independent reactions make the per-bird cooldown coherent and produce "a richer scene" than choosing one arbitrary receiving bird.

- **Small set of focused services:** The service shape is "not a monolith, not over-decomposed." The plan separates auth, simulation, event ingestion, snapshots, notebook, visits, account, and telemetry around ownership boundaries without adding excess decomposition.

- **Client renders snapshots and writes interaction events only:** The plan forbids client-side canonical simulation so reconnect never requires guessed-forward state to be reconciled against the server, avoiding exactly the divergent-simulation failure called out in the plan.

- **Presentation envelopes in snapshots:** The plan derives render parameters from traits server-side and serializes only those envelopes so raw personality numbers are absent from devtools, debug panels, and all response payloads.

- **Append-only event table at v1 scale:** The plan keeps events in Postgres for v1 because launch scale does not need Kafka/Kinesis and extra infra would slow the team down on the bird engine. Migration is named only if tick fan-out becomes a bottleneck.

- **CDN edge cache for the initial snapshot:** The plan uses this to support time-to-first-bird: first HTML can include the first snapshot, avoiding another round trip before the first paintable frame.

- **Visitor-token-authenticated read-only API:** The plan enforces read-only visitor sessions at the routing layer, "not just hidden client-side," so a visit token cannot mutate the aviary.

- **ETag-aware snapshot polling with no WebSockets:** The plan uses low-frequency pulls because the tick is slow, around 60 seconds, so push infrastructure "buys nothing" while costing bundle and ops budget.

- **Permanent `Bird.id`:** The plan calls this an "engine-level identity guarantee" and the foundation of drift's "perceived validity"; renames or migrations must never substitute one bird record for another.

- **Additive-only drift write path:** The plan removes any `set_personality` path and permits only `apply_delta`, enforcing "drift never moves down" at the simulation-service boundary rather than by convention.

- **No visit-count, days-active, streak, or display behavioral counter fields:** The plan omits these fields so future gamification cannot be a flag flip or hidden-column reuse; it must be a deliberate, reviewable schema change.

- **Scheduled server-side tick on a ~60s cadence:** The plan chooses 60 seconds as a cheap-to-retune config constant and avoids sharded workers until account volume requires them, since premature scaling would be wasted complexity.

- **Presence-time union across devices:** The plan unions segments instead of summing them so opening laptop and phone counts as one unit of attention, preventing silent drift inflation and the most likely subtle multi-device bug.

- **Leaky-integrator drift formula:** The formula gives natural saturation near the ceiling, guarantees monotonic non-decrease, and makes calibration a config change through per-signal constants rather than a code change.

- **Mood scoring with minimum margin or dwell time:** The plan uses margin and dwell requirements to avoid tick-to-tick flicker when candidate moods are borderline.

- **Ambient and bird-to-bird pass:** Weather, day/night, chorus, and wary-contagion are handled in the tick so ambient state can advance with zero user events. Chorus is "presentational only," and wary-contagion is transient so it cannot be mistaken for personality drift.

- **Perch-zone assignment with hysteresis and `perch_transition`:** The plan stages zone changes so the client animates flight rather than teleporting, and so borderline birds do not flicker zones tick over tick.

- **Return-greeting envelopes:** The plan treats greetings as important by computing eligibility after a real presence gap, selecting a weighted greeter, and attaching absence/intensity tiers. Client-side variation prevents the same greeting tier from playing identically twice and avoids "simultaneous chorus on cue."

- **Offer reaction fast path:** The plan splits immediate cosmetic reaction from authoritative drift because a 60s tick is too slow for the gesture to feel responsive, while trait-level drift is designed not to be visible within a single session anyway.

- **Settle undo window:** The plan delays sending settle to the server until the 5-second undo window passes, making undo simply "never send it."

- **Listen-in local audio ramp plus logged event:** The audio mix changes locally with no server round trip, while the event is logged in parallel only as the drift signal.

- **Presence heartbeats coalesced into segments:** Heartbeats every roughly 20 seconds bound event volume and avoid per-second chatter, while server coalescing preserves the exact presence definition for the tick.

- **Append-only sync model with no last-write-wins path:** Because clients append interaction events and never write absolute personality values, a stale device cannot clobber fresher drift.

- **Canvas2D aviary scene:** The plan uses Canvas2D because the scene animates continuously at 60fps and a DOM-diffing framework would fight that render loop.

- **Lazy DOM chrome with React/Preact:** The top bar, settings, account flows, and notebook are low-frequency, high-declarative surfaces, so they live in lazy DOM chunks above the canvas instead of the canvas render path.

- **Layered part-based bird rig:** The plan uses head/body/wing rigs rather than full skeletal animation or large sprite sheets to stay light against the 2MB budget while still allowing procedural, non-looping motion.

- **Snapshot interpolation:** The client interpolates from the currently rendered state to new snapshots to avoid popping or snapping to freshly fetched authoritative positions.

- **Initial HTML with inlined first snapshot and no spinner asset:** The plan wants the first rendered frame to place birds in their actual current positions, "mid-action." No spinner exists on this path because that would break the "already in motion" conceit.

- **Quiet field fallback:** When a snapshot is genuinely unavailable, the plan uses a shared "quiet field" component, also used before first-bird fly-in, preserving a soft sky/ambient state instead of inventing a separate loading experience.

- **Reduced-motion implementation inside the same animator:** Continuous offsets become slow cross-faded pose changes, flight becomes perch-to-perch crossfade, and ambient drift is removed. The why is to preserve the same product surface while honoring motion preferences across devices.

- **Responsive perch anchors with minimum-spacing solver:** The plan keeps birds inside the visible frame on narrow viewports and recalculates anchors without resetting each bird's animation phase.

- **Top bar fade separate from presence accounting:** The fade timer is deliberately separate because coupling a cosmetic chrome timer to the drift-bearing presence signal would create accidental, hard-to-notice breakage.

- **Client-side WebAudio call grammar with no audio files:** The plan treats procedural synthesis as unconditional. No recorded-audio fallback exists because "silence with captions is a better fallback than canned audio."

- **Pooled audio nodes or once-instantiated AudioWorklets:** The plan uses pooling to satisfy the no per-call allocation and no-memory-growth performance requirement.

- **True concurrent chorus mixing:** Each calling bird has its own synthesis voice and gain node into a shared bus, avoiding pre-mixed loops and the phase-cancellation artifact the plan names.

- **Listen-in mix control:** A single `setListenTarget(birdId | null)` funnels target gain, ambient floor, and all disengage triggers through one place so listen-in is a re-balance, not a mute.

- **Captions from synthesis parameters:** The same deterministic templating layer that knows motif and mood feeds both the synth engine and caption renderer, preserving exact alignment between sound and text.

- **WebAudio fallback to silence with captions on:** If synthesis cannot run, the plan no-ops audio and defaults captions on because canned audio is explicitly rejected as a fallback.

- **ARIA live region with ambient and event lanes:** The plan uses lanes feeding one queue so ambient updates and event observations can have different timing without competing screen-reader regions.

- **Invisible DOM overlays for canvas birds:** Because birds rendered on canvas are not natively focusable or assistive-technology-addressable, the plan adds positioned DOM buttons with accessible names and visible focus rings.

- **Initial bundle-size gate and lazy chunks:** The plan enforces the <2MB gzip budget on the first-paint chunk and lazily loads settings, accessibility settings, and visits so they do not inflate the gated bundle.

- **Time-to-first-bird monitoring:** The plan monitors this continuously because it is the "affective-perf bridge," achieved through inlined snapshots, prioritized renderer preload, and deferring audio/chrome work.

- **60fps idle motion and no 30-minute memory growth harness:** The plan makes frame time and heap growth real CI assertions, with object pooling as an engineering requirement rather than a guideline.

- **Aggregate-only observability pipeline:** The plan permits load timing, first-bird timing, frame timing, audio errors, and tick latency while refusing per-account or per-bird dimensions to preserve the privacy boundary.

- **Phase 0 dogfood:** The plan uses a small internal cohort with full engine and accessibility from day one to catch audio uncanniness and drift-feel issues that automated checks will not surface.

- **Phase 1 private beta:** The plan makes drift-rate distribution a go/no-go gate because miscalibrated drift would create either a "Tamagotchi-speed product" or a "screensaver-speed one," both core-experience failures.

- **Phase 2 public v1:** The plan launches starter birds, age-gated offers, and visit invitations together because offers are a product mechanic and visit invitations are live though default-off per account.

- **Instrumented from day one:** The plan instruments tick latency, heartbeat volume/coalescing, drift distribution, notebook rate, audio init, narration latency, bundle size, and first-bird timing to validate calibration, sparsity, and reliability without inspecting individuals.

- **Tick fleet pause safety valve:** The plan allows ticking to pause without data loss: the aviary freezes, events stay durable, and the log replays cleanly when ticking resumes.

- **Visit-invite creation safety valve:** The plan allows invite creation to be disabled independently so an invite-email incident can be isolated without taking down sign-in or the core aviary.
