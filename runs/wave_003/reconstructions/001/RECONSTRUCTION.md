## System-level intent

- Server-owned truth is the core architecture principle. The plan states this in "Planning stance" as "Canonical aviary state is server-owned end to end" and repeats it in "Client/server split," "Render pipeline boundary," and "Single canonical writer model": clients "never write personality, mood, or bird state directly," "never infer drift locally," and "never simulate ahead."

- Responsiveness is allowed only through server-authored state. The plan separates "immediate transient scene reactions for responsiveness" from a "slower once-per-minute durable tick for drift and mood evolution." This shows up again in "Immediate transient reactor," where accepted host events update short-lived scene changes "so the response feels immediate," while both layers still write through "server-owned canonical state."

- Snapshot delivery is pull-based and authoritative, with local interpolation limited to presentation. The plan says "Snapshot delivery is pull-based, not client-authoritative sync" and that clients poll "small authoritative snapshots and interpolate locally." The "Render pipeline boundary" makes the same split: "The server snapshot describes what is true" and "the client decides only how to draw that truth between snapshots."

- Product voice should be deterministic, private, and consistent. "Notebook prose, call captions, and screen-reader narration" come from "deterministic product logic, not an external LLM" to keep "voice consistent, latency low, and private bird history out of third-party systems." The same intent appears in "Notebook generation," "Caption coupling," "Screen-reader narration," and "Copy discipline."

- V1 favors a compact custom stack to protect bundle and first-bird budgets. The plan says "V1 should favor a compact, custom render/audio stack over heavyweight engines" and later says a heavy game engine is not justified because "the scene is small, the bundle budget is tight, and seven birds with modest layered motion do not justify a large rendering runtime."

- The product should keep "continuity, specificity, and restraint" instead of becoming a dashboard, a game, or a social feed. The scope excludes "scores, streaks, achievements, badges, counters," "Tamagotchi mechanics," "user-facing personality numbers," "feeds," and "leaderboards." The final execution guidance says any shortcut that turns the system into "a dashboard, a game, or a social feed" is a regression.

- Absence should not punish the user or reverse care. The plan excludes "hunger, decay, bird death, distress, punishment for absence." In the drift function, "Neglect never subtracts traits" and "The absence path is zero delta, not reversal."

- Privacy boundaries are product architecture, not only policy. The plan repeatedly says "aggregate-only operational telemetry," "privacy-preserving exports and deletion," "physically separated" metrics, "Never send per-bird personality, per-account interaction sequences, or notebook text into the metrics pipeline," and "Do not use production user-history aggregation."

- Accessibility is a first-class product surface in the same voice as the aviary. The scope calls out "Accessibility surfaces that are first-class product surfaces," and the QA section says to treat accessibility as "a release gate, not a post-launch patch." The mitigation for product voice says accessibility review is "a product review, not only a compliance review."

- Rollout should be conservative and operationally gated. The plan says to "Start conservatively," enable more birds only after "audio recognizability, tick stability, and snapshot size remain healthy," and keep the cap "server-configurable" so the team can "freeze expansion without migrations" if "chorus clarity or performance regresses."

## Per-feature whys

### Scope

- Web-only product for modern Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN

- Single-user account model with email magic-link auth: NOT RECOVERABLE FROM PLAN

- One canonical aviary per account: NOT RECOVERABLE FROM PLAN

- Two starter birds and age-based expansion up to a configurable max of seven: start conservatively so "audio recognizability, tick stability, and snapshot size remain healthy" before expanding cohorts from 3 to 5 to 7.

- Continuous server-side simulation with durable bird identity, personality drift, mood persistence, ambient weather, and time-of-day behavior: this supports the "aviary continues without the viewer" illusion and keeps state in canonical server-owned storage rather than client memory.

- Watch and presence behavior: presence is "the dominant drift input" and must remain "honest," so visible, focused, recently active attention becomes server-owned windows rather than trusted local state.

- Listen-in: listen-in duration contributes strongly to `social_warmth` and `vocal_frequency`, and the audio mix ramps the focused bird up while others remain audible so the aviary does not feel like "a track mixer."

- Offer: offers are a "server-authored reaction surface" that can immediately produce approach, ignore, watch, song, drink, bathe, or stillness while folding lasting mood or drift implications into the next durable tick.

- Settle: settle closes open presence windows and "ends presence cleanly" without adding a direct positive or negative drift term.

- Notebook reading: notebook entries are deterministic, "naturalist, lowercase, present tense, bird-specific, observational" prose based on salience events, not metrics, counters, or hidden numeric state.

- Account/settings surfaces: NOT RECOVERABLE FROM PLAN

- Accessibility settings: they are part of "first-class product surfaces," including manual reduced-motion override, captions, narration, and "describe aviary now."

- Multi-device sync from a single authoritative aviary state: it prevents state divergence across devices by making snapshots monotonic and server-authored.

- Invite-by-email read-only visits, revocable and off by default: this is the "quiet social feature"; visitor endpoints are read-only, visitor attention is never written into host simulation inputs, and revocation is enforced on every visitor snapshot read.

- Privacy-preserving exports and deletion: export and deletion flows are handled by the API and durable queue, with deletion-window state treated as "matter-of-fact system errors" and privacy-preserving data handling.

- Aggregate-only operational telemetry and synthetic performance monitoring: operations can track timing, latency, errors, and failures without per-account drilldown or per-bird personality in the metrics pipeline.

### Architecture and data model

- `web-shell`: it serves the SSR HTML shell, inline initial snapshot payload, UI bundle, and static scene assets so the first bird can appear without a second round trip.

- `aviary-api`: NOT RECOVERABLE FROM PLAN

- `simulation-worker`: it owns canonical state mutation, immediate reactions after accepted events, and the durable per-aviary minute tick.

- `mail-worker`: NOT RECOVERABLE FROM PLAN

- `ops-metrics`: it is "physically separated from per-account simulation tables" to preserve the aggregate-only metrics boundary.

- PostgreSQL primary store: the plan says to "keep the persistent backend simple" for accounts, birds, state snapshots, invites, notebook entries, and append-only interaction events.

- Durable job queue: it is for per-aviary due ticks, email jobs, deletion deadlines, and export generation.

- Object storage only if needed for exported JSON bundles and no media CDN for bird audio: exported bundles may need storage, but bird audio does not because calls are "synthesized client-side."

- Client rendering from authoritative snapshots: clients render current scene truth, interpolate motion, and manage UI affordances without owning canonical bird state.

- Procedural audio from server-provided call descriptors: the client synthesizes sound locally from descriptors rather than receiving audio buffers.

- Presence signal events without local drift inference: clients detect presence and send events, but "never infer drift locally" because drift is server-owned.

- Render pipeline boundary: keeping "what is true" on the server and "how to draw that truth" on the client preserves the "aviary continues without the viewer" illusion while preventing state divergence.

- Decorative ambient leaf and feather drift: it is "client-only and disposable" because bird positions, perch choices, greetings, offer reactions, and settle state are authoritative server outputs.

- Stable bird identity: identity is "invariant across renames, sync, and future migrations."

- Hidden scalar personality traits: they support drift while avoiding "user-facing personality numbers," bird stats dashboards, or trait optimization surfaces.

- Append-only `interaction_events` with idempotency keys and limited retention: raw events are kept only for "tick correctness, replay, and support" and then compacted so indefinite raw logs are avoided.

- `presence_windows`: they union overlapping device attention and "prevent double-counting" when the same host has multiple visible sessions.

- `notebook_entries` read-only after write: NOT RECOVERABLE FROM PLAN

- Account-level `user_settings` plus device-local overrides: account defaults cover captions, narration, audio, reduced motion, visit notifications, and exports, while device-local overrides remain client-side where appropriate, "especially `prefers-reduced-motion`."

- Visitor snapshot shape: visitor snapshots use the same scene payload but exclude host settings, write affordances, and host-only account metadata.

### API surface and polling model

- Magic-link, session, email-change, export, delete, and settings endpoints: NOT RECOVERABLE FROM PLAN

- `GET /aviary/snapshot`: it returns a compact authoritative snapshot or `304` and is used for initial load, visibility resume, long frame gap recovery, after write acknowledgements, and visible polling.

- `GET /aviary/notebook?cursor=...`: NOT RECOVERABLE FROM PLAN

- `POST /aviary/events`: it is the only host write surface for aviary behavior; clients send batched events with idempotency keys and "never send absolute bird values."

- Visit flow endpoints: they preserve read-only visitor behavior and do not accept presence or interaction events.

- Host visible-tab polling every 15 seconds: it stays faithful to the pull-based model while keeping cross-device freshness acceptable.

- Visitor visible-tab polling every 30 seconds: it keeps revocation latency low while staying lightweight.

- Hidden tabs stop polling: hidden tabs stop except for session expiry handling, and resumed clients refetch instead of trusting stale local state.

### Simulation engine design

- Immediate transient reactor: it updates greeting queues, listen-in focus, offer reactions, settle transitions, and other short-lived scene changes so the response "feels immediate."

- Durable minute tick: it applies personality drift, mood evolution, perch/action target updates, weather progress, notebook generation, and age-based bird offer eligibility at the slower cadence.

- Presence accounting with visibility, focus, recent activity, clamping, and device-window union: presence is "the dominant drift input" and "must remain honest," so the server does not trust the client blindly.

- Visitor sessions never create presence windows: visitor attention is never written into host simulation inputs.

- Trait-specific additive drift deltas: presence, listen-in, and offers can nudge traits, while "Neglect never subtracts traits" and "The absence path is zero delta, not reversal."

- Low-pass filter, caps, and diminishing returns within 24 hours: one long session should not create "visible same-day jumps," and grinding should not overpower the intended "three-week visible-drift cadence."

- Synthetic calibration harness: synthetic accounts and controlled internal dogfood scenarios tune the target of "one week measurable / three weeks visible" without production user-history aggregation.

- Mood finite-state machine with hysteresis: hysteresis prevents "one-minute oscillation" between states without a meaningful new impulse.

- Mood persistence: mood persists across sessions because it lives in canonical state, not client memory.

- Perch choice and visible action selection: target actions and perch zones express boldness, wary mood, content mood, curious mood, and drowsy mood through server-emitted pose/action windows.

- Greeting logic on session resume or visibility return: a meaningful absence can produce a primary greeting bird, longer calls, closer perches, and secondary responders, while history and cooldowns prevent one bird from monopolizing every return unless personality warrants it.

- Offer reactions: per-bird cooldowns and server-selected receiving birds keep offers ordered by proximity, curiosity, and mood, with transient reaction plans emitted immediately.

- Weather: soft wind and short rain occur only a few times per week, briefly alter mood and call propensities, and "never dominate the scene."

- Call grammar runtime: compact motif libraries and per-bird seeds create stable call identity without sending audio buffers.

- Notebook generation: salience inputs, minimum spacing, dedupe, and refusal to mention counters, metrics, or hidden numeric state keep entries sparse and observational.

- Single canonical writer model: only the server simulation mutates canonical bird state; host clients write append-only events and visitor clients never write to the host aviary.

- Idempotency keys, per-aviary serialization, presence union, and stale-precondition refresh: these prevent conflicts, double-counted presence, unordered transient state, and writes against obsolete state.

- Resume and suspension handling: resumed clients refetch immediately, stop local rendering and audio when hidden, and discard stale transient local animation in favor of a fresh authoritative snapshot.

- Revocation and expiry handling: visitor snapshot reads enforce revocation every time, and expired/revoked/deletion-window states use matter-of-fact system errors rather than naturalist copy.

### Frontend rendering pipeline

- Layered frontend with SSR HTML shell, React/TypeScript, and compact Canvas 2D scene renderer: this gives the quiet field, settings/notebook/accessibility surfaces, focus management, and small custom scene runtime.

- Avoiding a heavyweight game engine: the scene is small, the bundle budget is tight, and seven birds with modest layered motion do not justify a large rendering runtime.

- First-frame strategy with server-rendered field, inline first snapshot, and deferred non-critical bundles: this hits the "already in motion" illusion without a spinner.

- Bird rendering as species-specific rigs and palette parameters rather than large sprite sheets: this keeps assets compact while allowing snapshot-selected pose families, target perches, action timing, and saturation.

- Scene layering draw order: NOT RECOVERABLE FROM PLAN

- Reduced-motion rendering: slow cross-fades, cross-fade repositioning, and removal of leaf/feather drift preserve mood, palette, calls, captions, and notebook so the scene still feels intentional, not disabled.

- Top bar behavior: a thin top bar fades to near-transparent after inactivity but restores on pointer or keyboard activity and maintains keyboard visibility and clear focus styling.

### Audio pipeline

- WebAudio with `AudioWorklet`: it supports deterministic scheduling and low-GC synthesis.

- One synthesis voice per active bird and shared chorus bus: NOT RECOVERABLE FROM PLAN

- Calls scheduled from server-provided descriptor windows: they avoid "random local guesses."

- Bird call synthesis from oscillator/noise combinations, envelopes, filters, motif timing descriptors, and timbre seeds: this implements mood and vocal-frequency changes through cadence density, attack softness, pitch drift range, and chorus join likelihood.

- Mix behavior: ambient mode keeps birds audible at low levels, listen-in changes levels gradually, others never mute fully, and subtle depth cues avoid making the aviary feel like "a track mixer."

- Caption coupling: captions come from the same descriptor object used by audio scheduling, so near-bird overlays and assistive surfaces share the same text.

- Audio fallback: if WebAudio or AudioWorklet is unavailable, the aviary runs silently, captions are automatically enabled, and recorded audio fallbacks are not shipped.

### Accessibility surfaces

- Screen-reader narration: an aria-live narration region uses the same state as the visual scene and stays in naturalist voice, with idle cadence and priority cadence for greetings, offers, settle, and major scene changes.

- Manual "describe aviary now" control: it lets screen-reader users request an immediate summary without waiting for cadence.

- Keyboard navigation: keyboard operation is a first-class accessibility surface, with Tab, arrow keys, Enter, and Escape covering top-bar items, birds, listen-in, offers, settle, and transient overlays.

- Captions: captions are optional, quickly toggleable, short, prose-like, near-bird overlays with AA contrast that must not obscure birds or become a marquee.

- Reduced motion: `prefers-reduced-motion` is honored by default, manual override is allowed, and the reduced-motion scene must still feel intentional.

- Copy discipline: aviary scene, notebook, captions, and narration use naturalist voice; auth, errors, sync failures, settings, unsupported browser, and revoked/expired invite surfaces use matter-of-fact voice.

- Accessibility QA: VoiceOver, NVDA, keyboard-only flows, reduced-motion regression, and caption overlap checks are a release gate rather than a post-launch patch.

### Performance budgets, observability, and rollout

- Tactics to hit budgets: inline initial snapshot, code splitting, custom renderer, compact assets, audio-node reuse, and pausing the render loop when hidden all exist to protect bundle, first-bird, frame-rate, memory, and tick budgets.

- Observability metrics: timing, latency, error rates, frame-time distributions, audio failures, tick durations, invite sends, and session-duration histograms provide operations coverage while excluding per-bird personality, per-account interaction sequences, and notebook text.

- Synthetic monitoring and calibration: browser probes check load/render/audio from common geographies, and synthetic aviary accounts calibrate drift and notebook behavior without production user histories.

- Delivery phases: NOT RECOVERABLE FROM PLAN

- Birds-per-aviary ramp: cap increases are cohort-based and server-configurable so expansion can pause if chorus clarity or performance regresses.

- Day-one instrumentation: auth, first-bird timing, snapshot polls, event ingestion, ticks, audio init, reduced-motion/caption usage, and invite issuance/revocation are instrumented from day one, while "average user drift" and "most offered bird" are explicitly not instrumented.
